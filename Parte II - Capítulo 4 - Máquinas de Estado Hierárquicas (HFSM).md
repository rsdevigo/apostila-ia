# Capítulo 4 — Máquinas de Estado Hierárquicas (HFSM)

## Introdução

O Capítulo 3 terminou com um problema em aberto. A máquina de estados finita é simples, barata, previsível e depurável — mas **não escala**. À medida que o comportamento cresce, o número de transições explode em ritmo quadrático, e a máquina degenera em um novelo de setas cruzadas que ninguém consegue ler nem manter. A pergunta que abre este capítulo é direta: **como preservar as virtudes da FSM e, ao mesmo tempo, domar sua complexidade?**

A resposta que a indústria consolidou é uma ideia que aparece repetidamente na engenharia sempre que a complexidade ameaça o controle: a **hierarquia**. Assim como um programa grande se organiza em funções e módulos, e uma empresa se organiza em departamentos e equipes, uma máquina de estados grande pode se organizar em **níveis**. Estados relacionados são agrupados sob um "superestado" comum, formando uma **máquina de estados hierárquica** (do inglês *Hierarchical Finite State Machine*, HFSM). A hierarquia não adiciona poder expressivo novo — tudo o que uma HFSM faz, uma FSM plana suficientemente grande também faria —, mas adiciona **organização**, e organização é exatamente o que faltava.

Este capítulo é a continuação natural do anterior e assume tudo o que foi visto nele. Seguimos a mesma filosofia: partimos do **problema** (a complexidade das FSMs grandes), construímos os **fundamentos** da hierarquia (superestados, subestados e histórico), detalhamos o **funcionamento** (herança de transições e memória de estado), ilustramos com um **exemplo** de combate em camadas, pesamos **vantagens e limitações** — apontando o que a HFSM ainda *não* resolve, o que motivará os capítulos seguintes — e aterrissamos nas **ferramentas** (as sub-state machines do Animator, o pacote Unity Behavior, o State Tree da Unreal) e no **mercado**.

> 🕰️ **Contexto Histórico**
> A ideia de hierarquizar máquinas de estado não é exclusiva dos jogos: ela foi formalizada de modo influente por David Harel em 1987, com os **statecharts**, uma extensão das máquinas de estado que introduziu hierarquia, concorrência e memória. Os statecharts tornaram-se um padrão na engenharia de sistemas reativos e influenciaram notações como a UML. A IA de jogos, mais uma vez, adotou uma ferramenta madura de outra área: quando os jogos dos anos 1990 e 2000 cresceram a ponto de suas FSMs se tornarem ingerenciáveis, a hierarquização — já bem compreendida em engenharia de software — foi a resposta natural.

---

## 4.1 O problema: reduzir a complexidade de FSMs grandes

Retomemos concretamente o ponto onde o Capítulo 3 parou. Nosso inimigo guarda tinha cinco estados e funcionava bem. Mas jogos reais pedem mais. Suponha que o designer queira enriquecer o comportamento de combate: o inimigo agora deve poder *atirar à distância*, *avançar em cobertura*, *recarregar*, *lançar granada* e *corpo a corpo*. E que o comportamento pacífico também cresça: *patrulhar rota fixa*, *vagar aleatoriamente*, *conversar com outro guarda*, *investigar ruído*, *reportar à base*. De repente, temos uma dúzia de estados — e o problema da explosão de transições se instala.

O ponto crucial, porém, não é apenas o número de transições. É que **muitas transições se repetem de forma idêntica em vários estados**. Observe: a regra "se a vida cair abaixo de 20%, fugir" precisa valer estando o inimigo atirando, avançando, recarregando, lançando granada *ou* em corpo a corpo. Na FSM plana, isso significa **replicar a mesma transição em cinco estados diferentes**. Pior: se o designer decidir mudar o limiar de 20% para 30%, precisará caçar e alterar essa transição em todos os cinco lugares — e basta esquecer um para introduzir um bug sutil, em que o inimigo "esquece" de fugir em uma situação específica.

Há aqui uma redundância estrutural gritante. Os cinco estados de combate — atirar, avançar, recarregar, granada, corpo a corpo — **compartilham** um conjunto de transições comuns ("se vida baixa → fugir", "se perdeu o alvo → investigar", "se sem munição e sem alvo → recuar"). Eles formam, intuitivamente, uma *família*: "estados de combate". O que falta à FSM plana é um modo de **expressar essa família** e de **dizer, uma única vez, o que é comum a todos os seus membros**.

Esse é o problema que a HFSM resolve. Ela permite agrupar os estados de combate sob um **superestado** "Combate" e associar a esse superestado as transições comuns. Cada subestado (atirar, avançar...) herda automaticamente as transições do superestado. A regra "se vida baixa → fugir" passa a ser escrita **uma vez**, no superestado, e vale para todos os subestados. A redundância desaparece; a manutenção se simplifica; a máquina volta a ser legível.

> 🎮 **Na Prática**
> A pergunta que revela a necessidade de hierarquia é: *"há um conjunto de estados que compartilham as mesmas transições de saída?"* Se a resposta for sim — e em comportamentos de jogo ela quase sempre é —, esses estados formam uma família que pede um superestado. "Enquanto estou em qualquer modo de combate, se levar dano grave eu fujo" é uma frase que descreve, ao mesmo tempo, um superestado (combate) e uma transição herdada (dano grave → fugir).

---

## 4.2 Fundamentos: superestados, subestados e histórico

A HFSM introduz três conceitos que estendem o vocabulário do Capítulo 3. Todos os conceitos da FSM plana continuam valendo; a hierarquia apenas os organiza em níveis.

**Superestado (estado composto).** Um superestado é um estado que **contém, dentro de si, outra máquina de estados**. Ele não representa um comportamento atômico, mas uma *categoria* de comportamentos. Nosso superestado "Combate" contém a submáquina com os estados atirar, avançar, recarregar, etc. Quando dizemos que o agente "está em Combate", queremos dizer que ele está em *algum* dos subestados de Combate. O superestado funciona como uma "pasta" que agrupa estados relacionados e à qual se podem anexar transições comuns.

**Subestado.** Um subestado é um estado *dentro* de um superestado. Ele pode ser um comportamento atômico (uma folha, como "recarregar") ou, por sua vez, outro superestado — a hierarquia pode ter vários níveis de profundidade. A qualquer instante, o agente está em uma **cadeia** de estados: por exemplo, "Combate → Atirar" significa que ele está no superestado Combate e, dentro dele, no subestado Atirar. Essa cadeia é chamada de **configuração ativa** ou *pilha de estados ativos*.

**Estado inicial de cada nível.** Cada superestado precisa declarar qual de seus subestados é o **inicial** — o comportamento assumido ao se entrar naquele superestado pela primeira vez. Ao entrar em Combate, por exemplo, o agente pode começar em "Atirar". Isso equivale ao estado inicial da FSM plana, agora replicado em cada nível da hierarquia.

**Estado de histórico (memória).** É o conceito mais distintivo da HFSM, e merecerá a seção 4.3.2. Um **estado de histórico** faz o superestado *lembrar* em qual subestado ele estava quando foi abandonado, de modo que, ao retornar, o agente retome de onde parou em vez de recomeçar do subestado inicial. Se o inimigo estava "Combate → Recarregando" quando foi forçado a "Fugir", ao voltar ao Combate um superestado com histórico o recolocaria em "Recarregando" (ou no subestado mais apropriado), preservando a continuidade.

[DIAGRAMA]
Título: Anatomia de uma máquina de estados hierárquica
Objetivo pedagógico: Mostrar visualmente a organização em níveis — superestados contendo submáquinas — e como as transições comuns migram para o superestado.
Descrição detalhada: Diagrama com dois grandes retângulos arredondados representando superestados, dispostos lado a lado: "PACÍFICO" e "COMBATE". Dentro de "Pacífico", uma submáquina com os subestados Patrulhar, Vagar e Investigar, com suas transições internas. Dentro de "Combate", uma submáquina com os subestados Atirar, Avançar, Recarregar e Corpo a corpo, com transições internas. Fora e acima dos dois superestados, um estado simples "Fugir". Setas de nível superior: de "Pacífico" para "Combate" (rótulo: "vê o jogador"); de "Combate" para "Pacífico" (rótulo: "perdeu o alvo"); e — crucial — uma seta partindo da *borda* do superestado "Combate" (não de um subestado específico) para "Fugir", rotulada "vida < 20% (transição herdada por todos os subestados de Combate)". Marcar o subestado inicial de cada superestado com uma seta de entrada solta. Uma legenda destaca: "a transição na borda do superestado vale para todos os seus subestados".
Elementos obrigatórios: dois superestados como caixas contendo submáquinas; subestados internos com transições; transição comum ancorada na borda do superestado; estado externo Fugir; marcação dos subestados iniciais; legenda explicando a herança.
[/DIAGRAMA]

---

## 4.3 Funcionamento

O funcionamento da HFSM estende o laço de execução da FSM plana com duas ideias-chave: a **avaliação em cascata pela hierarquia** (que dá origem à herança de transições) e a **memória de configuração** (o estado de histórico). Vejamos cada uma.

O princípio geral é que, a cada quadro, as transições são avaliadas **do nível mais externo para o mais interno** (ou, em algumas formulações, do mais interno para o mais externo — a ordem é uma decisão de projeto com consequências, como veremos). Considere o agente na configuração "Combate → Atirar". A máquina avalia, em cascata:

```
a cada quadro, para a configuração ativa "Combate → Atirar":
    1. avaliar as transições do superestado Combate
       (ex.: vida < 20% → Fugir ; perdeu o alvo → Pacífico)
    2. se nenhuma disparar, avaliar as transições do subestado Atirar
       (ex.: sem munição → Recarregar ; alvo colado → Corpo a corpo)
    3. se nenhuma disparar, executar a ação de permanência do
       subestado ativo (Atirar): mirar e disparar
```

Repare que as transições do superestado são checadas **antes** das do subestado. Isso implementa a prioridade natural "regras gerais de sobrevivência vêm antes de detalhes táticos": não importa qual subestado de combate esteja ativo, a regra "vida baixa → fugir" tem a primeira palavra. É essa avaliação em cascata que dá sentido operacional à herança de transições.

### 4.3.1 Herança de transições e transições entre níveis

A **herança de transições** é o mecanismo central da HFSM e a origem direta de sua economia. Uma transição ancorada em um superestado é, na prática, **compartilhada por todos os seus subestados** — como se estivesse replicada em cada um deles, mas escrita uma única vez. Escrever "Combate → Fugir quando vida < 20%" na borda do superestado equivale a escrever essa transição em atirar, avançar, recarregar e corpo a corpo simultaneamente. Muda-se o limiar em um lugar, e a mudança se propaga a todos.

Isso resolve exatamente a redundância diagnosticada na seção 4.1. O número de transições que o projetista precisa *escrever e manter* deixa de crescer de forma quadrática: transições comuns sobem para o superestado e são declaradas uma vez; apenas as transições genuinamente específicas de cada subestado permanecem no nível interno. A complexidade percebida despenca, embora a máquina *equivalente plana* continuasse tendo todas aquelas transições.

Precisamos, contudo, ser cuidadosos com dois tipos de transição entre níveis:

**Transições de saída de superestado.** Quando uma transição herdada dispara — digamos, "vida < 20% → Fugir" —, o agente **sai de toda a cadeia** Combate → Atirar de uma vez. As ações de saída (*exit*) devem ser executadas **de dentro para fora**: primeiro o *exit* de Atirar, depois o *exit* de Combate. Simetricamente, ao entrar em um superestado, executam-se as ações de entrada **de fora para dentro**: primeiro o *enter* de Combate, depois o *enter* do subestado inicial (ou do subestado lembrado pelo histórico). Manter essa ordem é essencial para que inicializações e limpezas ocorram na sequência correta.

**Transições que entram diretamente em um subestado profundo.** Às vezes queremos transitar não para o subestado inicial de um superestado, mas diretamente para um subestado específico e profundo (por exemplo, "ao ser emboscado, entrar direto em Combate → Corpo a corpo"). A HFSM permite isso, mas o projetista deve garantir que todas as ações de entrada da cadeia sejam disparadas na ordem correta (enter de Combate, depois enter de Corpo a corpo).

> ❌ **Erro Comum**
> Esquecer de disparar as ações **enter/exit dos níveis intermediários** ao transitar entre hierarquias. Ao pular de "Pacífico → Patrulhar" para "Combate → Corpo a corpo", uma implementação descuidada executa o *exit* de Patrulhar e o *enter* de Corpo a corpo, mas **esquece** o *exit* de Pacífico e o *enter* de Combate. O resultado são inicializações perdidas (o superestado Combate não "se preparou") e vazamentos de estado (o superestado Pacífico não "se encerrou"). Regra de ouro: ao mudar de configuração, execute os *exit* da cadeia antiga de dentro para fora e os *enter* da cadeia nova de fora para dentro, até o ponto em que as duas cadeias divergem.

### 4.3.2 Estado de histórico (memória)

O **estado de histórico** é o que confere à HFSM uma forma de **memória** que a FSM plana não tem naturalmente. Sem histórico, sempre que o agente entra num superestado, ele começa pelo subestado inicial. Com histórico, o superestado *lembra* onde estava e retoma dali.

O exemplo torna a utilidade evidente. Nosso inimigo está em "Combate → Recarregando" (a meio caminho de recarregar a arma) quando leva um dano crítico e a transição herdada "vida < 20% → Fugir" dispara. Ele foge, recupera-se e a situação o traz de volta ao Combate. **O que deve acontecer?** Sem histórico, ele reentra em Combate pelo subestado inicial (digamos, Atirar) — e tenta atirar com a arma ainda descarregada, um comportamento incoerente que quebra a ilusão de inteligência. Com histórico, o superestado Combate lembra que o agente estava recarregando e o recoloca em Recarregando (ou em um subestado apropriado à situação), preservando a continuidade do comportamento.

A literatura de statecharts distingue dois tipos de histórico: o **histórico raso** (*shallow history*), que lembra apenas o subestado imediato do nível em questão, e o **histórico profundo** (*deep history*), que lembra a configuração completa de toda a subárvore, em todos os níveis. Para a maioria dos comportamentos de jogo, o histórico raso já é suficiente; o profundo é útil quando há várias camadas de aninhamento cuja configuração completa importa.

> 🎮 **Na Prática**
> O estado de histórico é uma das ferramentas mais eficazes para *fabricar credibilidade* de forma barata. Um NPC que, ao ser interrompido, **retoma exatamente o que fazia** — volta a recarregar, retoma a conversa no ponto em que parou, continua a patrulha do trecho onde estava — parece ter *memória* e *intenção persistente*. É a diferença entre um personagem que "vive" no mundo e um que "renasce" a cada mudança de estado. E, como quase tudo na ilusão de inteligência, o mecanismo por baixo é modesto: apenas guardar qual subestado estava ativo.

> ⚠️ **Atenção**
> Histórico é poderoso, mas nem sempre é o que se quer. Em algumas situações, *reiniciar* pelo subestado inicial é o comportamento correto — por exemplo, ao reentrar em "Patrulha" o guarda talvez deva recomeçar a rota do começo, não do meio. Decida, para cada superestado, se ele deve ter memória (histórico) ou recomeçar do zero (subestado inicial). Aplicar histórico indiscriminadamente produz comportamentos "grudentos" e às vezes ilógicos.

---

## 4.4 Exemplos: comportamento de combate em camadas

Voltemos ao inimigo do Capítulo 3, agora enriquecido, para ver a HFSM em ação. Organizaremos seu comportamento em **dois superestados** de alto nível — *Pacífico* e *Combate* — mais um estado simples de alto nível, *Fugir*, e o estado terminal *Morto*.

O superestado **Pacífico** contém a submáquina do comportamento fora de combate: os subestados *Patrulhar*, *Vagar*, *Investigar* e *Conversar*. As transições internas a Pacífico governam a vida rotineira do guarda (de Patrulhar para Investigar ao ouvir um ruído, etc.).

O superestado **Combate** contém a submáquina tática: *Atirar*, *Avançar em cobertura*, *Recarregar*, *Lançar granada* e *Corpo a corpo*. As transições internas governam as escolhas táticas (de Atirar para Recarregar quando a munição acaba; para Corpo a corpo quando o alvo se aproxima demais; etc.).

No **nível superior**, três transições organizam a vida do agente: de *Pacífico* para *Combate* ("vê o jogador"); de *Combate* de volta para *Pacífico* ("perdeu o alvo por tempo suficiente"); e, ancoradas na **borda do superestado Combate** (herdadas por todos os seus subestados), as transições de sobrevivência: "vida < 20% → Fugir". O estado *Fugir*, ao recuperar a segurança, devolve o agente a *Combate* (com histórico) ou a *Pacífico*, conforme a situação.

A tabela a seguir organiza a hierarquia completa.

| Nível | Estado / Superestado | Contém / Comportamento | Transições próprias (do nível) |
|---|---|---|---|
| Superior | **Pacífico** (superestado) | Patrulhar, Vagar, Investigar, Conversar | vê o jogador → Combate |
| Superior | **Combate** (superestado) | Atirar, Avançar, Recarregar, Granada, Corpo a corpo | perdeu o alvo → Pacífico; **vida < 20% → Fugir (herdada por todos os subestados)** |
| Superior | **Fugir** (estado simples) | Corre para aliados, pede reforço | seguro e vida recuperada → Combate (histórico) ou Pacífico |
| Superior | **Morto** (estado simples) | Ativa animação de morte; desativa IA | (terminal) |
| Interno (Combate) | Atirar → Recarregar | — | sem munição → Recarregar |
| Interno (Combate) | Atirar → Corpo a corpo | — | alvo a menos de 1,5 m → Corpo a corpo |

**Análise interpretativa.** Compare esta organização com o que seria uma FSM plana equivalente. A regra de sobrevivência "vida < 20% → Fugir", que na FSM plana precisaria ser replicada nos cinco subestados de combate (e revisada nos cinco a cada ajuste), aqui é escrita **uma única vez** na borda do superestado Combate. As transições internas de cada submáquina não "enxergam" as da outra — o projetista raciocina sobre *Combate* e *Pacífico* separadamente, como dois problemas menores. A máquina completa, que teria dezenas de transições cruzadas se fosse plana, torna-se um punhado de submáquinas pequenas e compreensíveis. **Esse é o ganho central da HFSM: ela não faz o agente mais inteligente; faz a máquina mais gerenciável** — e, com isso, permite comportamentos maiores sem perder o controle.

[IMAGEM NECESSÁRIA]
Título: HFSM de combate em camadas do inimigo
Objetivo didático: Permitir que o aluno visualize a hierarquia completa descrita na tabela, percebendo como submáquinas isoladas se conectam por transições de alto nível e por transições herdadas.
Descrição: Diagrama hierárquico com dois grandes contêineres (Pacífico e Combate), cada um englobando seus subestados e transições internas; os estados de alto nível Fugir e Morto fora dos contêineres. Transições de nível superior entre os contêineres e para Fugir/Morto, com destaque visual (cor ou espessura) para a transição herdada "vida < 20% → Fugir" ancorada na borda de Combate. Indicar o subestado inicial de cada contêiner e um pequeno ícone de "histórico" (H) no superestado Combate, simbolizando a memória.
Tipo: Diagrama de estados hierárquico (ilustração vetorial)
Como produzir: Ferramenta de diagramação com suporte a estados aninhados (draw.io, yEd, PlantUML/statechart); estética limpa, com contêineres claramente delimitados e legenda dos símbolos (subestado inicial, histórico, transição herdada).
Legenda sugerida: Figura 4.1 — HFSM do inimigo: os superestados Pacífico e Combate agrupam submáquinas coesas; a regra de sobrevivência, escrita uma única vez na borda de Combate, é herdada por todos os subestados de combate.
[/IMAGEM NECESSÁRIA]

> 🏭 **Na Indústria**
> A organização "comportamento de alto nível → submáquinas táticas" é um padrão consagrado em jogos de tiro e de ação. Um NPC costuma ter um nível estratégico (pacífico, alerta, combate, retirada) e, dentro de cada um, submáquinas de detalhe. Essa separação em camadas também facilita a divisão de trabalho na equipe: um designer pode ajustar a submáquina de combate sem tocar na de patrulha, e vice-versa — uma vantagem de *produção*, não apenas de *código*, coerente com o critério de autoria da Parte I.

---

## 4.5 Vantagens e limitações

**Vantagens.**

A HFSM herda todas as virtudes da FSM — simplicidade conceitual, baixo custo, previsibilidade, depurabilidade — e acrescenta as que resolvem o problema de escala. A principal é a **redução drástica da redundância**, via herança de transições: o que é comum a uma família de estados é dito uma vez. Daí decorre a **manutenibilidade**: mudanças em regras gerais se propagam automaticamente. A hierarquia traz **organização e legibilidade**: comportamentos grandes se decompõem em submáquinas pequenas e coesas, raciocináveis isoladamente — um caso da estratégia de "dividir para conquistar". O **estado de histórico** adiciona memória de contexto, que fortalece a credibilidade. E há um ganho de **reutilização** moderado: uma submáquina bem isolada (por exemplo, "comportamento de cobertura") pode, com cuidado, ser reaproveitada em diferentes agentes.

**Limitações.**

A HFSM, entretanto, **não elimina** os problemas de fundo da família FSM — apenas os adia e atenua. Continua havendo **acoplamento**: os subestados de uma submáquina ainda conhecem uns aos outros por transições explícitas, e a reutilização de comportamentos *entre* submáquinas diferentes continua trabalhosa. A **rigidez estrutural** permanece: adicionar um novo comportamento que se relacione com muitos outros ainda exige costurar transições manualmente, e reorganizar prioridades pode significar reescrever conexões. A hierarquia **precisa ser projetada com acerto**: se o agrupamento de estados em superestados for mal escolhido, a HFSM pode ficar tão confusa quanto uma FSM plana — a hierarquia certa não é óbvia e depende de boa análise do domínio. E há um limite intrínseco: a HFSM continua sendo, no fundo, uma máquina de **estados e transições explícitas**; ela é excelente para comportamento *modal* (o agente está sempre em um modo bem definido), mas desajeitada quando o comportamento precisa ser **sequenciado, priorizado e reordenado com frequência** — quando queremos dizer "tente A; se falhar, tente B; se falhar, tente C", em vez de "esteja no modo A e transite para o modo B sob tal condição".

É precisamente essa última limitação que abre caminho para as próximas arquiteturas. As **árvores de decisão** (Capítulo 5) reorganizam a lógica em torno de *testes encadeados* em vez de estados, servindo de ponte conceitual; e as **árvores de comportamento** (Capítulo 6) reorganizam-na em torno de *tarefas compostas e reutilizáveis*, atacando de frente o acoplamento e a rigidez que a HFSM apenas atenua. A HFSM, note-se, não é tornada obsoleta — ela continua sendo a melhor escolha para comportamento genuinamente modal —, mas deixa de ser a resposta única.

> ❌ **Erro Comum**
> Acreditar que a HFSM "resolve de vez" os problemas da FSM. Ela os *administra*, não os elimina. Uma HFSM mal hierarquizada — com superestados escolhidos sem critério, ou com transições entre níveis emaranhadas — pode ser tão difícil de manter quanto a FSM plana que pretendia substituir. A hierarquia é uma ferramenta de organização; como toda organização, só ajuda se a divisão em grupos refletir a estrutura real do problema.

---

## 4.6 Jogos conhecidos e aplicações

A hierarquização de estados é onipresente em jogos de porte médio e grande, ainda que raramente rotulada como "HFSM" nos materiais de divulgação. Apresentamos aplicações representativas, sempre distinguindo o documentado da análise técnica.

**Personagens de jogos de ação e tiro.** NPCs com repertório rico de combate e comportamento fora de combate são, estruturalmente, candidatos naturais à HFSM: um nível estratégico (patrulha/alerta/combate/retirada) com submáquinas táticas dentro de cada modo. A atribuição específica de "HFSM" a um título comercial dado é, na maioria dos casos, **análise técnica fundamentada** na estrutura observável do comportamento — a documentação pública costuma descrever a IA em termos gerais, sem detalhar a arquitetura interna.

**Sistemas de animação em camadas.** Um uso *documentado* e explícito de hierarquia está nos sistemas de animação: o Animator da Unity oferece **sub-state machines** e **camadas de animação** precisamente para organizar animações complexas em grupos (por exemplo, um superestado "Locomoção" contendo parado/andar/correr, e outro "Ações" contendo atacar/pular). Aqui a hierarquia é parte oficial da ferramenta, não inferência.

**Jogos de simulação e gerenciamento.** Personagens com muitas atividades (trabalhar, socializar, cuidar de necessidades) frequentemente organizam esses comportamentos em camadas. Ainda que jogos de simulação sofisticados tendam a usar IA de utilidade (Capítulo 6), a estruturação hierárquica de comportamentos modais aparece com frequência como componente.

> 🎲 **Curiosidade**
> Vários motores e frameworks de IA de jogos oferecem HFSM "de fábrica", mas muitos estúdios, ao crescerem, migraram de HFSMs para **árvores de comportamento** justamente pelas limitações da seção 4.5 — a dificuldade de reutilizar e reordenar comportamento. Essa migração histórica, ocorrida sobretudo a partir de meados dos anos 2000, é o pano de fundo do Capítulo 6 e um exemplo concreto da lição da Parte I: as técnicas se acumulam e se sucedem conforme os problemas de produção evoluem.

---

## 4.7 Ferramentas na Unity

Como sempre, o foco é reconhecer o conceito nas ferramentas, não decorar menus.

**Sub-state machines no Animator.** A materialização mais direta da HFSM na Unity são as **sub-state machines** (submáquinas de estado) do Animator Controller. Elas permitem agrupar estados de animação relacionados dentro de um estado composto, exatamente como um superestado agrupa subestados. As transições podem ser definidas tanto dentro da submáquina quanto para "sair" dela em direção a outros estados do nível superior, reproduzindo a herança e as transições entre níveis discutidas na seção 4.3. As **camadas de animação** (*Animator Layers*) acrescentam ainda uma forma de *concorrência* (várias submáquinas ativas em camadas diferentes) — um recurso que endereça, no domínio da animação, a limitação da FSM quanto a comportamentos concorrentes.

**O pacote Unity Behavior.** O pacote oficial **Unity Behavior**, voltado à autoria de comportamento de agentes, acomoda hierarquias na organização do comportamento, e sua orientação principal a *árvores de comportamento* o coloca já no território do Capítulo 6. Para o propósito deste capítulo, registre-se que ele oferece um caminho oficial e visual para estruturar comportamento em níveis, alternativo ao uso do Animator para lógica de decisão.

**HFSM em C#.** Também é comum implementar HFSMs diretamente em código, estendendo o padrão *State* do Capítulo 3 para que estados possam conter submáquinas (um estado que, em seu `Update`, delega para a submáquina interna, e cujas transições de nível têm prioridade sobre as internas). É a abordagem que melhor espelha os conceitos deste capítulo e que oferece controle total sobre a ordem de avaliação e sobre o histórico.

> ⚠️ **Atenção**
> Repetimos a ressalva do Capítulo 3, agora agravada: usar o **Animator (com sub-state machines) para toda a lógica de decisão** de um agente complexo mistura as responsabilidades de "cérebro" e "corpo" e pode reproduzir, dentro do editor de animação, os problemas de manutenção que a hierarquia deveria evitar. Sub-state machines são excelentes para organizar *animação*; para *decisão* complexa, avalie ferramentas dedicadas (Unity Behavior, soluções de terceiros) ou uma implementação em código.

---

## 4.8 Ferramentas de terceiros

No ecossistema Unity, as ferramentas de terceiros mais conhecidas para IA — **NodeCanvas** e **Behavior Designer** — oferecem editores de **FSM e HFSM** além de árvores de comportamento, permitindo montar hierarquias de estado visualmente, com suporte a estados aninhados e a transições entre níveis. Há também pacotes especializados em máquinas de estado hierárquicas na Asset Store e em repositórios abertos, muitos inspirados nos *statecharts* de Harel.

Como comparação entre engines, vale destacar novamente o **State Tree** da Unreal Engine, que combina a hierarquia das máquinas de estado com a seleção estruturada em árvore, situando-se num ponto intermediário entre a HFSM deste capítulo e as árvores de comportamento do próximo — uma evidência de que a fronteira entre "estados hierárquicos" e "árvores de tarefas" é mais um espectro do que uma divisão rígida.

> ✅ **Boa Prática**
> Ao escolher uma ferramenta de HFSM, verifique especialmente três pontos que a teoria deste capítulo mostrou serem críticos: (1) ela deixa **visível e correta** a ordem de avaliação entre níveis (superestado antes ou depois do subestado)? (2) ela suporta **estado de histórico**, para retomar comportamentos interrompidos? (3) ela dispara corretamente as ações **enter/exit de todos os níveis** intermediários nas transições entre hierarquias? Uma ferramenta que erra nesses pontos produz bugs sutis de continuidade e inicialização.

---

## 4.9 Resumo, Exercícios de fixação e Referências

### Resumo do Capítulo 4

Este capítulo apresentou a **máquina de estados hierárquica (HFSM)** como a primeira resposta da engenharia de IA de jogos ao problema deixado em aberto no Capítulo 3: a **explosão de transições** e a redundância das FSMs planas grandes. A ideia central é a **hierarquia** — agrupar estados relacionados sob **superestados**, cada um contendo sua própria submáquina de **subestados**. Construímos os fundamentos (superestado, subestado, estado inicial de cada nível e **estado de histórico**) e detalhamos o funcionamento: a **avaliação em cascata** das transições, do nível externo ao interno, que dá origem à **herança de transições** — o mecanismo que permite escrever uma regra comum (como "vida baixa → fugir") **uma única vez**, na borda do superestado, valendo para todos os seus subestados. Vimos os cuidados com as transições entre níveis (a ordem correta de disparo das ações enter/exit) e o papel do **histórico** em preservar a continuidade do comportamento — uma forma barata de fabricar credibilidade. Ilustramos com o inimigo de **combate em camadas**, organizado nos superestados *Pacífico* e *Combate*, evidenciando a redução de redundância frente à FSM plana. Pesamos vantagens (redundância reduzida, manutenibilidade, organização, memória) e limitações — e destacamos que a HFSM **atenua, mas não elimina** o acoplamento e a rigidez da família FSM, sendo excelente para comportamento *modal* mas desajeitada quando o comportamento precisa ser sequenciado e reordenado com frequência. É essa limitação que motiva os capítulos seguintes: as **árvores de decisão** (Cap. 5), como ponte conceitual, e as **árvores de comportamento** (Cap. 6), como resposta madura. Por fim, reconhecemos o conceito nas **sub-state machines** do Animator, no pacote **Unity Behavior**, no **State Tree** da Unreal e em ferramentas de terceiros.

### Exercícios de fixação

1. Explique, retomando o Capítulo 3, qual problema específico das FSMs planas a HFSM foi criada para resolver. Por que a simples adição de estados não bastava?
2. Defina **superestado**, **subestado** e **configuração ativa** (pilha de estados ativos). Dê um exemplo de configuração ativa de dois níveis para um inimigo.
3. O que é **herança de transições**? Usando a regra "vida < 20% → Fugir", mostre concretamente quanta redundância ela elimina em comparação com uma FSM plana equivalente de cinco estados de combate.
4. Descreva a **avaliação em cascata** das transições numa HFSM. Por que, em geral, convém avaliar as transições do superestado *antes* das do subestado? Que prioridade de comportamento isso implementa?
5. Explique o **estado de histórico** e diferencie histórico raso de histórico profundo. Dê um exemplo em que o histórico é indispensável para a credibilidade e outro em que *não* usar histórico é o comportamento correto.
6. Ao transitar de "Pacífico → Patrulhar" para "Combate → Corpo a corpo", quais ações **enter** e **exit**, e em que ordem, devem ser executadas? Que bug surge se as ações dos níveis intermediários forem esquecidas?
7. Projete (em tabela ou diagrama) uma HFSM para um NPC **companheiro** do jogador (ao estilo de um aliado que segue, cobre e revive o jogador), com pelo menos dois superestados e quatro subestados no total, incluindo ao menos uma transição herdada.
8. A HFSM "elimina" os problemas da FSM? Justifique, citando pelo menos duas limitações que a HFSM **não** resolve e explicando por que elas motivam as arquiteturas dos Capítulos 5 e 6.
9. As **sub-state machines** do Animator da Unity implementam o conceito deste capítulo. Mapeie os elementos da ferramenta (submáquina, transições internas, transições de saída, camadas) aos conceitos de superestado, subestado, transição entre níveis e concorrência.
10. Um estúdio percebe que sua HFSM de inimigos está ficando ingerenciável: comportamentos precisam ser constantemente reordenados e reutilizados entre agentes diferentes. Que sinais indicam que talvez seja hora de migrar para uma **árvore de comportamento**? Relacione com as limitações da seção 4.5.

### Referências

MILLINGTON, Ian. *AI for Games.* 3. ed. Boca Raton: CRC Press, 2019. (Referência principal para máquinas de estado hierárquicas, herança de transições e comportamento em camadas.)

RABIN, Steve (org.). *Game AI Pro 3: Collected Wisdom of Game AI Professionals.* Boca Raton: A K Peters/CRC Press, 2017. (Artigos sobre arquiteturas de decisão hierárquicas e sua aplicação prática.)

BOURG, David M.; SEEMANN, Glenn. *AI for Game Developers: Creating Intelligent Behavior in Games.* Sebastopol, CA: O'Reilly Media, 2004. (Panorama de técnicas de comportamento baseadas em estados.)

HAREL, David. Statecharts: A Visual Formalism for Complex Systems. *Science of Computer Programming*, v. 8, n. 3, p. 231–274, 1987. (Referência conceitual para hierarquia, concorrência e memória em máquinas de estado — origem formal do estado de histórico.)

UNITY TECHNOLOGIES. *Documentação oficial da Unity: Animator sub-state machines, Animator Layers e pacote Unity Behavior.* (Materialização das HFSMs na engine.)

EPIC GAMES. *Documentação oficial da Unreal Engine: State Tree.* (Comparação entre engines para estados hierárquicos e seleção em árvore.)

> **Nota sobre fontes.** O uso de sub-state machines e camadas de animação no Animator da Unity é **documentado** pela engine. As atribuições de HFSM a NPCs de jogos comerciais específicos constituem **análise técnica fundamentada** na estrutura observável do comportamento, e não documentação oficial de arquitetura interna, salvo indicação em contrário.
