# Capítulo 3 — Máquinas de Estado Finitas (FSM)

## Introdução

Se existe uma técnica que merece o título de "linguagem materna" da IA de jogos, é a **máquina de estados finita** (do inglês *Finite State Machine*, ou FSM). Ela é, provavelmente, a primeira arquitetura de decisão que qualquer desenvolvedor de jogos aprende, a mais usada na história do meio e, ainda hoje, a mais presente nos títulos comerciais — muitas vezes escondida dentro de arquiteturas mais sofisticadas. Um estudante que já mexeu no **Animator Controller** da Unity, ainda que para animar personagens, já construiu uma máquina de estados sem necessariamente saber o nome do que fazia.

A razão dessa onipresença é simples: a FSM captura, de forma quase óbvia, um modo de pensar que os seres humanos já usam naturalmente para descrever comportamento. Quando dizemos "o guarda está *patrulhando*", "agora ele está *perseguindo* o jogador", "ele entrou em *combate*", estamos, sem perceber, descrevendo **estados** e as **transições** entre eles. A FSM apenas formaliza essa intuição em uma estrutura precisa, implementável e — o que é decisivo para a indústria — **visualizável e depurável**.

Este capítulo abre a Parte II seguindo a filosofia da apostila: começa pelo **problema** que a FSM resolve, constrói seus **fundamentos** a partir da teoria dos autômatos, detalha seu **funcionamento** (incluindo as duas grandes variantes de implementação e o ciclo enter/update/exit), ilustra com **exemplos** clássicos de comportamento de NPC, pesa **vantagens e limitações** — com destaque para o problema que dará origem ao próximo capítulo, a *explosão de transições* — e, por fim, aterrissa a teoria nas **ferramentas da Unity e de terceiros** e no **mercado**. Ao final, a FSM não será apenas mais uma técnica: será a base conceitual sobre a qual todas as arquiteturas seguintes desta Parte serão construídas e comparadas.

> **Contexto Histórico**
> As máquinas de estado não nasceram nos jogos. Elas são um dos objetos matemáticos mais antigos e estudados da Ciência da Computação, formalizados na primeira metade do século XX no contexto da teoria da computação e dos autômatos — os mesmos autômatos finitos que descrevem circuitos digitais, analisadores léxicos de compiladores e protocolos de comunicação. Quando os primeiros desenvolvedores de jogos precisaram organizar o comportamento de inimigos em máquinas de recursos minúsculos (os arcades e consoles de 8 bits), a máquina de estados era a ferramenta perfeita: exigia pouquíssima memória, era rápida de executar e fácil de raciocinar. A IA de jogos, portanto, não *inventou* a FSM — ela **adotou** uma ferramenta madura e a colocou a serviço da ilusão de inteligência.

---

## 3.1 O problema: alternar comportamentos de forma organizada

Retomemos o ciclo **Sentir → Pensar → Agir** da Parte I. Um agente percebe o mundo e precisa, a cada quadro, decidir o que fazer. A forma mais ingênua de programar essa decisão é escrever uma longa cadeia de condicionais dentro do código que roda a cada quadro:

```
a cada quadro:
    se vejo o jogador e estou perto: atacar
    senão se vejo o jogador e estou longe: perseguir
    senão se ouvi um barulho: investigar
    senão se levei dano recentemente: procurar cobertura
    senão: patrulhar
```

Para um comportamento minúsculo, isso funciona. Mas essa abordagem tem um defeito conceitual grave, que se manifesta assim que o comportamento cresce: **ela não tem memória de contexto**. A cada quadro, o agente reavalia tudo do zero, como se tivesse acabado de nascer. Não há noção de "o que eu estava fazendo" nem de "como cheguei aqui". Isso gera dois problemas concretos.

Primeiro, **comportamentos que dependem de continuidade ficam difíceis de expressar**. Imagine que, ao perder o jogador de vista, o guarda deva *investigar por alguns segundos a última posição conhecida* antes de voltar a patrulhar. Com puros condicionais reavaliados a cada quadro, não existe um lugar natural para guardar "estou investigando há quanto tempo?". O programador acaba criando variáveis avulsas — `estouInvestigando`, `tempoDeInvestigacao`, `jaViOJogador` — que se multiplicam e se contradizem.

Segundo, **a lógica vira uma teia impossível de manter**. Cada novo comportamento exige revisitar toda a cadeia de condicionais para garantir que as prioridades continuem corretas. Uma mudança em um ponto quebra silenciosamente outro. O designer não consegue enxergar o comportamento como um todo, e a equipe perde a capacidade de responder à pergunta mais básica de depuração: *por que o inimigo está fazendo isso agora?*

O problema real, portanto, não é "como escolher uma ação" — condicionais fazem isso. O problema é **como organizar comportamentos distintos e mutuamente exclusivos ao longo do tempo, de modo que o agente tenha uma noção clara de "em que modo está" e de "sob quais condições muda de modo"**. É exatamente esse problema que a máquina de estados resolve.

> **Na Prática**
> Pense em um inimigo de qualquer jogo de ação. Em quase todos os momentos, ele está fazendo *uma coisa por vez*: patrulhando, ou perseguindo, ou atacando, ou recuando — nunca "meio patrulhando e meio atacando". Essa característica — comportamentos **mutuamente exclusivos**, um ativo por vez — é a assinatura do problema que a FSM foi feita para resolver. Sempre que você conseguir descrever o comportamento de um agente como "ele está num destes N modos, e passa de um para outro quando acontece tal coisa", a FSM é uma candidata natural.

---

## 3.2 Fundamentos: estados, transições, eventos e ações

Uma máquina de estados finita organiza o comportamento em torno de quatro conceitos elementares. Dominá-los com precisão é indispensável, pois eles reaparecerão, com nomes ligeiramente diferentes, em todas as arquiteturas seguintes.

**Estado.** Um estado é um **modo de comportamento** em que o agente pode se encontrar. Em cada instante, o agente está em exatamente **um** estado — essa exclusividade é a essência da FSM. Exemplos de estados de um inimigo: *Patrulhar*, *Perseguir*, *Atacar*, *Fugir*, *Investigar*, *Morto*. Cada estado encapsula um comportamento coeso: enquanto o agente está no estado *Patrulhar*, ele executa a lógica de patrulha e nada mais.

**Transição.** Uma transição é uma **passagem permitida de um estado para outro**. Ela conecta um estado de origem a um estado de destino e define *sob que condição* a passagem ocorre. Nem toda mudança de estado é permitida: a FSM declara explicitamente quais transições existem. Por exemplo, pode haver uma transição de *Patrulhar* para *Perseguir*, mas não de *Morto* para *Patrulhar*.

**Evento (ou condição de transição).** É o **gatilho** que dispara uma transição. Pode ser um evento discreto ("o jogador entrou no campo de visão", "a vida chegou a zero") ou uma condição contínua avaliada periodicamente ("a distância ao jogador é menor que 2 metros"). A cada transição associa-se uma condição; quando ela se torna verdadeira, a transição é executada.

**Ação.** É o **comportamento executado** — o que o agente efetivamente faz. Como veremos na seção 3.3.2, ações podem estar associadas ao *momento de entrar* em um estado, à *permanência* nele (executada a cada quadro) ou ao *momento de sair* dele.

Reunindo os quatro conceitos: **uma FSM é um conjunto de estados, conectados por transições, disparadas por eventos, que executam ações.** Uma definição compacta e poderosa.

[DIAGRAMA]
Título: Anatomia de uma máquina de estados finita
Objetivo pedagógico: Fixar visualmente os quatro conceitos elementares (estado, transição, evento, ação) em um único diagrama de referência.
Descrição detalhada: Um diagrama de grafo com três estados desenhados como círculos rotulados "Patrulhar", "Perseguir" e "Atacar". Setas dirigidas ligam os estados representando transições: de Patrulhar para Perseguir (rótulo do evento: "vê o jogador"); de Perseguir para Atacar (rótulo: "distância < 2 m"); de Atacar para Perseguir (rótulo: "distância > 2 m"); de Perseguir para Patrulhar (rótulo: "perdeu o jogador de vista"). Dentro de cada círculo, uma pequena legenda indica a ação de permanência (por exemplo, dentro de "Patrulhar": "seguir rota de pontos"). Um dos círculos deve ter uma seta de entrada solta, sem origem, marcando o "estado inicial". Uma legenda lateral associa cada elemento gráfico ao seu nome: círculo = estado; seta = transição; texto sobre a seta = evento/condição; texto dentro do círculo = ação.
Elementos obrigatórios: três estados como círculos; setas dirigidas rotuladas com eventos; marcação do estado inicial; ações dentro dos estados; legenda dos quatro conceitos.
[/DIAGRAMA]

### 3.2.1 Autômatos finitos e a raiz teórica

A FSM de jogos é uma aplicação direta de um objeto matemático clássico: o **autômato finito**. Compreender essa raiz não é preciosismo acadêmico — ela explica tanto o poder quanto os limites da técnica.

Formalmente, um autômato finito determinístico é definido por uma quíntupla: um **conjunto finito de estados**, um **alfabeto de entradas** (os eventos possíveis), uma **função de transição** que, dado um estado e uma entrada, determina o próximo estado, um **estado inicial** e um conjunto de **estados de aceitação**. Na teoria da computação, esse objeto serve para *reconhecer* linguagens: dada uma sequência de símbolos, o autômato termina em um estado de aceitação se, e somente se, a sequência pertence à linguagem.

Nos jogos, reaproveitamos a **estrutura** do autômato, mas mudamos o **propósito**. Não estamos reconhecendo linguagens; estamos **gerando comportamento**. Os "estados de aceitação" perdem o sentido; o que nos interessa é que, em cada estado, o agente *faz algo*, e que a "função de transição" nos diz para qual comportamento migrar quando um evento acontece. Ainda assim, a herança teórica traz consequências importantes.

A mais importante delas é o **caráter finito e determinístico**. "Finito" significa que o número de estados é fixo e conhecido de antemão — o agente nunca inventa um estado novo em tempo de execução. "Determinístico", na formulação clássica, significa que, dado o estado atual e o evento, o próximo estado é *univocamente* determinado. Essas duas propriedades são exatamente o que torna a FSM tão querida pela indústria: um comportamento finito e determinístico é **previsível, testável e depurável**. O designer pode enumerar todos os estados, revisar todas as transições e ter certeza de que não há comportamento "escondido". Como vimos na Parte I, previsibilidade e controle são critérios de primeira ordem na IA de jogos — e a FSM os oferece quase de graça, por sua própria natureza matemática.

> **Curiosidade**
> A teoria dos autômatos distingue autômatos *determinísticos* (AFD) de *não determinísticos* (AFN), nos quais um mesmo evento poderia levar a vários estados. Um belo resultado clássico mostra que os dois têm o mesmo poder de reconhecimento — todo AFN pode ser convertido em um AFD equivalente. Na IA de jogos, quase sempre trabalhamos com máquinas *determinísticas*, justamente porque queremos comportamento previsível. Quando um jogo precisa de imprevisibilidade, ela costuma ser introduzida de forma controlada — por exemplo, uma transição que ocorre com certa probabilidade — e não pela ambiguidade do autômato.

### 3.2.2 Estados, guardas de transição e condições

Na prática de jogos, o conceito de "evento" que dispara uma transição é frequentemente implementado como uma **guarda de transição** (*transition guard*, ou simplesmente *guard condition*): uma **condição booleana** associada à transição, avaliada para decidir se a passagem deve ocorrer. A transição só "dispara" quando sua guarda é verdadeira.

Essa modelagem por guardas é importante por três motivos. Primeiro, ela **desacopla o gatilho do comportamento**: a lógica de "quando mudar de estado" fica separada da lógica de "o que fazer no estado". Segundo, ela permite **condições compostas**: uma guarda pode combinar vários testes ("vejo o jogador *e* minha vida está acima de 30%"). Terceiro, ela dá lugar à noção de **prioridade entre transições**, essencial quando várias guardas de um mesmo estado são verdadeiras ao mesmo tempo.

Esse último ponto merece atenção, pois é uma fonte comum de bugs. Suponha que o agente esteja no estado *Perseguir* e que, num dado quadro, *duas* guardas se tornem verdadeiras simultaneamente: "distância < 2 m" (que levaria a *Atacar*) e "vida < 10%" (que levaria a *Fugir*). Qual transição vence? A FSM precisa de uma **regra de desempate** — em geral, uma ordem de prioridade fixa entre as transições de saída de cada estado. O projetista deve decidir conscientemente essa ordem; deixá-la ao acaso produz comportamento errático e difícil de reproduzir.

> **Erro Comum**
> Esquecer de definir a **prioridade** entre transições concorrentes. Quando duas guardas de saída de um mesmo estado podem ser verdadeiras ao mesmo tempo, a máquina precisa saber qual delas prevalece. Se a implementação simplesmente "pega a primeira que encontrar", o comportamento passa a depender da ordem em que as transições foram cadastradas — uma dependência frágil e invisível, que produz bugs difíceis de reproduzir. Boa prática: ordenar explicitamente as transições de cada estado por prioridade (por exemplo, sobrevivência antes de agressão).

Vale também distinguir dois modelos teóricos de FSM que influenciam onde as ações "moram", conhecidos pelos nomes de seus criadores. Nas **máquinas de Moore**, a ação (a saída) depende apenas do *estado atual*: estar no estado *Atacar* produz sempre o comportamento de ataque, independentemente de como se chegou lá. Nas **máquinas de Mealy**, a saída depende do estado *e da transição* que levou a ele. Na IA de jogos, o modelo predominante é próximo do de Moore — o comportamento é definido pelo estado —, mas o ciclo enter/update/exit que veremos a seguir incorpora um toque de Mealy, ao permitir ações específicas *no momento da transição* (as ações de entrada e saída).

---

## 3.3 Funcionamento

Vistos os fundamentos, examinemos como uma FSM efetivamente *roda* dentro do laço de jogo. Há duas decisões de projeto centrais: **quando as transições são avaliadas** (a distinção entre polling e eventos, seção 3.3.1) e **como as ações se distribuem no tempo de vida de um estado** (o ciclo enter/update/exit, seção 3.3.2).

O esqueleto de execução de uma FSM, a cada quadro, é o seguinte pseudocódigo:

```
a cada quadro (para o estado atual S):
    1. avaliar as guardas das transições que saem de S,
       em ordem de prioridade
    2. se alguma guarda for verdadeira:
        - executar a ação de SAÍDA do estado atual
        - trocar o estado atual para o estado de destino
        - executar a ação de ENTRADA do novo estado
    3. senão:
        - executar a ação de PERMANÊNCIA (update) do estado atual
```

Esse laço, repetido a cada quadro para cada agente, é toda a "máquina" da máquina de estados. Sua simplicidade é justamente sua força: cabe em pouquíssimas linhas, custa quase nada e é trivial de depurar (basta registrar o estado atual e as transições disparadas).

### 3.3.1 FSM baseada em polling versus baseada em eventos

Existe uma distinção prática fundamental sobre *como* as guardas de transição são verificadas, com impacto direto no custo computacional — e, portanto, no orçamento de quadro.

Na **FSM baseada em polling** (ou "por sondagem"), o agente **verifica ativamente todas as suas guardas de transição a cada quadro**, exatamente como no pseudocódigo acima. É a abordagem mais direta e a mais comum em implementações didáticas. Sua vantagem é a simplicidade e a robustez: como as condições são checadas o tempo todo, o agente nunca "perde" uma mudança de situação. Sua desvantagem é o custo: se cada guarda envolve cálculos caros (por exemplo, um teste de linha de visão que consulta a física do mundo), fazê-lo a cada quadro, para cada agente, pode pesar no orçamento.

Na **FSM baseada em eventos**, o agente **não fica verificando condições**; em vez disso, ele *reage* a notificações enviadas por outros sistemas do jogo. Quando o sistema de percepção detecta o jogador, ele *envia um evento* "jogador avistado" ao agente, que então dispara a transição correspondente. As guardas só são avaliadas quando um evento relevante chega. A vantagem é a eficiência: nada é recalculado sem necessidade. A desvantagem é a complexidade de infraestrutura — é preciso um sistema de mensagens/eventos bem construído — e o risco de "estados perdidos" se um evento deixar de ser emitido.

Na prática, jogos comerciais frequentemente **combinam as duas abordagens**: usam eventos para mudanças que outros sistemas já detectam naturalmente (dano recebido, colisão, jogador avistado pelo sistema de percepção) e polling para condições que só o próprio agente pode avaliar continuamente (distância a um alvo, tempo decorrido em um estado).

> **Boa Prática**
> Prefira **eventos** para condições que já são detectadas por outros sistemas (o motor de física já sabe quando houve uma colisão; o sistema de percepção já sabe quando avistou o jogador) e reserve o **polling** para o que precisa de verificação contínua e barata. Verificar por polling, a cada quadro, uma condição cara (como um *raycast* de linha de visão) para dezenas de agentes é uma das causas mais comuns de estouro do orçamento de IA. Quando o polling for inevitável, considere reduzir sua frequência (avaliar a cada poucos quadros em vez de todo quadro) para agentes distantes ou menos importantes — um uso do "LOD de IA" visto na Parte I.

[DIAGRAMA]
Título: Polling versus eventos na avaliação de transições
Objetivo pedagógico: Contrastar visualmente as duas estratégias de avaliação de guardas, evidenciando onde cada uma gasta processamento.
Descrição detalhada: Diagrama dividido em duas colunas. Coluna esquerda, "Polling (por sondagem)": uma linha do tempo com quadros sucessivos (quadro 1, 2, 3, ...); em cada quadro, um ícone de "lupa" sobre todas as guardas ("vê jogador?", "distância < 2 m?", "vida < 10%?"), indicando que todas são verificadas sempre. Coluna direita, "Eventos": a mesma linha do tempo, mas as guardas só são verificadas nos quadros em que chega uma "mensagem" (representada por um envelope), como "evento: jogador avistado" no quadro 3; nos demais quadros, o agente permanece ocioso quanto a transições. Um rodapé compara: "Polling: custo constante, simples, robusto" × "Eventos: custo sob demanda, eficiente, exige infraestrutura de mensagens".
Elementos obrigatórios: duas colunas rotuladas; linha do tempo de quadros; ícones de verificação contínua (polling) versus verificação sob evento (envelopes); rodapé comparativo de custos.
[/DIAGRAMA]

### 3.3.2 Ações de entrada, permanência e saída (enter/update/exit)

Um refinamento essencial, presente em praticamente toda implementação profissional de FSM, é a divisão das ações de cada estado em **três momentos** distintos do seu ciclo de vida:

**Ação de entrada (*enter* / *on-enter*).** Executada **uma única vez**, no instante em que o agente entra no estado. Serve para *inicializar* o comportamento: tocar uma animação, emitir um alerta ("Inimigo à vista!"), escolher um ponto de destino, zerar um cronômetro. É o "preparar-se para fazer".

**Ação de permanência (*update* / *on-update*).** Executada **a cada quadro**, enquanto o agente permanece no estado. É o comportamento contínuo propriamente dito: seguir o caminho, mirar e disparar, procurar cobertura. É o "fazer".

**Ação de saída (*exit* / *on-exit*).** Executada **uma única vez**, no instante em que o agente deixa o estado. Serve para *limpar* e *encerrar*: parar uma animação de laço, guardar a última posição conhecida do jogador, liberar um recurso reservado. É o "arrumar antes de ir".

Essa tripartição resolve, de forma elegante, o problema de "memória de contexto" que atormentava a abordagem por condicionais. Considere novamente o guarda que investiga a última posição vista antes de voltar a patrulhar. Com o ciclo enter/update/exit, a solução é limpa: ao *entrar* no estado *Investigar*, o guarda registra a última posição conhecida e zera um cronômetro; na *permanência*, ele caminha até lá e olha em volta enquanto o cronômetro corre; a guarda de saída "cronômetro esgotado" o leva de volta a *Patrulhar*. Cada pedaço de estado vive no lugar certo.

> **Na Prática**
> O ciclo enter/update/exit não é apenas uma conveniência de programação — ele é o que torna a FSM *legível para o designer*. Ao abrir o estado *Atacar* de um inimigo, a equipe encontra, organizadamente: "ao entrar, grita e assume postura de combate"; "enquanto ataca, mira e dispara a cada X segundos"; "ao sair, guarda a última posição do alvo". Essa organização espelha como um roteirista descreveria a cena — e é justamente por isso que designers, não apenas programadores, conseguem trabalhar com FSMs.

> **Atenção**
> Não confunda a ação de **permanência** (roda a cada quadro) com as de **entrada/saída** (rodam uma vez). Colocar no *update* algo que deveria estar no *enter* — por exemplo, "tocar o som de alerta" — faz o som se repetir a cada quadro, produzindo um erro audível e clássico de iniciante. A pergunta orientadora é sempre: *isto deve acontecer uma vez, ao mudar de modo, ou continuamente, enquanto estou neste modo?*

---

## 3.4 Exemplos: patrulha, perseguição, ataque, fuga

Consolidemos os conceitos com o exemplo canônico da IA de jogos: o **inimigo guarda** de um jogo de ação ou furtividade. Este é, provavelmente, o comportamento mais reimplementado da história do meio, e serve de referência para toda a Parte II — voltaremos a ele em cada arquitetura para comparar como cada uma o expressa.

Nosso guarda terá cinco estados: *Patrulhar*, *Investigar*, *Perseguir*, *Atacar* e *Fugir*. A tabela a seguir descreve cada estado por seu ciclo enter/update/exit e suas transições de saída, em ordem de prioridade.

| Estado | Ao entrar (enter) | Enquanto permanece (update) | Transições de saída (por prioridade) |
|---|---|---|---|
| **Patrulhar** | Escolhe o próximo ponto de rota | Caminha pela rota; varre o ambiente com o olhar | vida < 20% → *Fugir*; vê o jogador → *Perseguir*; ouve ruído → *Investigar* |
| **Investigar** | Registra a origem do ruído; zera cronômetro | Vai até o local; olha em volta | vida < 20% → *Fugir*; vê o jogador → *Perseguir*; cronômetro esgotado → *Patrulhar* |
| **Perseguir** | Emite alerta ("Ali está ele!") | Corre em direção à última posição vista (usando busca de caminho) | vida < 20% → *Fugir*; distância < 2 m → *Atacar*; perdeu de vista por 3 s → *Investigar* |
| **Atacar** | Assume postura de combate | Mira e dispara em intervalos | vida < 20% → *Fugir*; distância > 2 m → *Perseguir* |
| **Fugir** | Grita por reforços | Corre para longe do jogador, em direção a aliados | jogador fora de alcance e vida recuperada → *Patrulhar* |

**Análise interpretativa.** Observe como a tabela expressa um comportamento rico — patrulha, curiosidade, perseguição, combate, autopreservação — sem uma única linha de condicionais aninhados. Cada estado é uma "caixa" isolada e compreensível; cada transição é uma regra explícita; a prioridade de sobrevivência (a guarda "vida < 20% → *Fugir*" aparece em quase todos os estados) fica visível e auditável. Note também um traço de credibilidade deliberadamente construído: o estado *Investigar* existe unicamente para *fabricar a ilusão de inteligência*. Um agente que fosse instantaneamente de "vê o jogador" para "não vê mais o jogador → volta a patrulhar" pareceria bobo e desatento; o guarda que caminha até o último ruído e "procura um pouco" antes de desistir parece *curioso e atento* — embora, por dentro, seja apenas mais um estado com um cronômetro.

[IMAGEM NECESSÁRIA]
Título: Máquina de estados do inimigo guarda
Objetivo didático: Oferecer ao aluno a representação visual completa da FSM de cinco estados descrita na tabela, para que ele associe a descrição textual ao grafo de estados.
Descrição: Diagrama de estados profissional (grafo dirigido) com cinco nós rotulados — Patrulhar, Investigar, Perseguir, Atacar, Fugir — dispostos de forma clara. Cada seta de transição rotulada com sua condição (vê o jogador, ouve ruído, distância < 2 m, distância > 2 m, perdeu de vista por 3 s, vida < 20%, etc.). O estado Fugir com setas de entrada vindas de todos os outros (destacando a prioridade de sobrevivência). Marcar o estado inicial (Patrulhar) com uma seta de entrada solta. Usar cores para agrupar estados "pacíficos" (Patrulhar, Investigar) e "de combate" (Perseguir, Atacar, Fugir).
Tipo: Diagrama de estados (ilustração vetorial)
Como produzir: Ferramenta de diagramação (draw.io, Figma, yEd ou Mermaid exportado), com estética limpa e legendas legíveis; sem uso de arte protegida.
Legenda sugerida: Figura 3.1 — FSM completa de um inimigo guarda: cinco estados e as transições que os conectam, com a prioridade de sobrevivência (Fugir) alcançável de qualquer estado.
[/IMAGEM NECESSÁRIA]

Um segundo exemplo, muito diferente em domínio mas idêntico em estrutura, mostra a generalidade da técnica: o **ciclo de vida de uma unidade coletora** em um jogo de estratégia (um aldeão de *Age of Empires*, por exemplo). Estados: *Ocioso*, *Indo coletar*, *Coletando*, *Voltando ao depósito*, *Depositando*. As transições formam um ciclo quase circular: de *Indo coletar* para *Coletando* (chegou ao recurso), de *Coletando* para *Voltando* (carga cheia), de *Voltando* para *Depositando* (chegou ao depósito), de *Depositando* de volta para *Indo coletar* (carga entregue). Um comportamento econômico complexo, essencial ao gênero, cabe numa FSM de cinco estados — evidência de que a técnica serve tanto ao combate quanto à simulação.

> **Curiosidade**
> Muitos jogos de estratégia em tempo real modelam *cada unidade* como uma pequena FSM, e há centenas de unidades em tela. Isso ilustra por que o **baixo custo** da FSM é tão valioso: multiplicado por centenas de agentes, a diferença entre uma decisão barata e uma cara determina se o jogo roda ou engasga. Uma arquitetura de decisão elegante mas custosa seria inviável nesse cenário — a FSM, por ser quase gratuita, prospera.

---

## 3.5 Vantagens e limitações

Reunimos agora, de forma sistemática, os pontos fortes e fracos da FSM. Este balanço é o que justificará, nos capítulos seguintes, o surgimento das arquiteturas sucessoras.

**Vantagens.**

A FSM é **simples e intuitiva**: mapeia diretamente o modo humano de descrever comportamento ("ele está fazendo X; quando acontece Y, passa a fazer Z"). É **barata**, tanto em memória quanto em processamento, o que a torna viável para centenas de agentes simultâneos e para plataformas modestas. É **previsível e determinística**, propriedade que a Parte I identificou como critério de primeira ordem: o comportamento pode ser enumerado, testado e reproduzido. É **depurável**: a qualquer instante, o agente está em um estado identificável, e a pergunta "por que ele faz isso agora?" tem resposta imediata (o estado atual e a última transição). É **visualizável**: presta-se naturalmente a editores gráficos, o que dá autoria ao designer, como veremos na seção 3.7. E é **modular no pequeno**: cada estado é uma unidade coesa e isolada.

**Limitações.**

A FSM sofre de um conjunto de problemas que se agravam com a escala. O **acoplamento entre estados** cresce descontroladamente: cada estado precisa "saber" para quais outros pode transitar, o que espalha o conhecimento das transições por toda a máquina. A **reutilização é fraca**: um comportamento como "procurar cobertura", útil em vários contextos, tende a ser reescrito em cada estado que o necessita, pois não há um mecanismo natural de reaproveitamento. A FSM lida mal com **comportamentos concorrentes ou em camadas**: como o agente está em *um* estado por vez, expressar "atirar *enquanto* recua *enquanto* recarrega" exige ou multiplicar estados combinados (*RecuarAtirando*, *RecuarRecarregando*...) ou recorrer a truques. E, acima de tudo, ela sofre da **explosão de transições**, o problema estrutural que trataremos a seguir e que motiva o Capítulo 4.

### 3.5.1 A explosão de transições

O calcanhar de Aquiles da FSM plana (não hierárquica) é o crescimento **quadrático** do número potencial de transições em relação ao número de estados. A raiz do problema é geométrica: em uma máquina com *n* estados, cada estado pode, no pior caso, precisar de uma transição para cada um dos outros *n − 1* estados. Isso dá, no limite, *n × (n − 1)* transições possíveis — um crescimento de ordem *n²*.

Traduzindo em números concretos: uma máquina de **5 estados** admite até 20 transições — perfeitamente gerenciável, como no nosso guarda. Mas uma de **10 estados** já admite até 90; uma de **20 estados**, até 380; uma de **50 estados**, quase **2.500**. Muito antes de chegar a esses extremos, a máquina se torna um "prato de espaguete" de setas cruzadas, impossível de ler, editar ou depurar. Pior: cada novo estado adicionado exige revisar as transições de *todos* os estados existentes para decidir se algum deve poder transitar para o novo — um custo de manutenção que também cresce de forma não linear.

[DIAGRAMA]
Título: A explosão de transições
Objetivo pedagógico: Tornar visceral, de forma visual, por que a FSM plana não escala — mostrando o crescimento quadrático das transições.
Descrição detalhada: Três painéis lado a lado. Painel A: uma FSM de 3 estados, com poucas setas, rotulado "3 estados — até 6 transições — legível". Painel B: uma FSM de 6 estados, já com um emaranhado moderado de setas, rotulado "6 estados — até 30 transições — difícil". Painel C: uma FSM de 12 estados totalmente interconectada, um "novelo" ilegível de setas cruzadas, rotulado "12 estados — até 132 transições — impraticável". Abaixo dos três painéis, um pequeno gráfico de curva mostrando o número de transições crescendo com o quadrado do número de estados (curva n²), evidenciando a aceleração.
Elementos obrigatórios: três FSMs de complexidade crescente (legível → difícil → impraticável); contagem de transições em cada uma; curva n² ao fundo evidenciando o crescimento quadrático.
[/DIAGRAMA]

Esse não é um problema meramente estético. Ele ataca justamente as vantagens que tornam a FSM valiosa: a máquina deixa de ser **depurável** (ninguém consegue mais raciocinar sobre o novelo), deixa de ser **editável** pelo designer (a ferramenta visual vira um caos) e deixa de ser **modular** (tudo se conecta a tudo). É a percepção desse limite — vivido na prática pela indústria à medida que os jogos cresciam nos anos 1990 e 2000 — que motivou a busca por arquiteturas que **domassem a complexidade** sem abandonar as virtudes da FSM. A primeira dessas respostas, a **máquina de estados hierárquica**, é o tema do próximo capítulo.

> **Erro Comum**
> Tentar resolver todo problema de comportamento adicionando *mais estados* à mesma FSM plana. Existe um ponto — em geral em torno de uma dúzia de estados — a partir do qual acrescentar estados *piora* a manutenibilidade em vez de melhorar o comportamento. Reconhecer esse ponto e migrar para uma arquitetura hierárquica (Capítulo 4) ou para árvores de comportamento (Capítulo 6) é sinal de maturidade de engenharia, não de fracasso da FSM.

---

## 3.6 Aplicações e jogos conhecidos

A FSM está presente, de uma forma ou de outra, em praticamente todo jogo já feito. Destacamos aplicações representativas, sempre distinguindo o que é **documentado** do que é **análise técnica fundamentada**.

**Pac-Man (1980).** É o exemplo didático por excelência, e felizmente bem documentado por análises técnicas consagradas do jogo. Cada fantasma alterna entre modos de comportamento que correspondem a estados de uma FSM: *Perseguir* (*chase*), *Espalhar* (*scatter*, recuar para o próprio canto) e *Assustado* (*frightened*, fugir do Pac-Man após ele comer uma pílula de poder). A transição entre *chase* e *scatter* é governada por temporizadores globais; a transição para *frightened* é disparada por um evento (o jogador comeu a pílula). O que torna *Pac-Man* um caso tão elegante é que a *personalidade* de cada fantasma emerge não de estados diferentes, mas de **alvos de perseguição diferentes** dentro do mesmo estado *chase* — um detalhe que já discutimos na Parte I como exemplo de comportamento emergente. A estrutura de estados, porém, é uma FSM clássica.

**Inimigos de jogos de ação e plataforma (era 8/16 bits e além).** A imensa maioria dos inimigos de jogos das décadas de 1980 e 1990 é modelada como FSMs pequenas: *Parado → Andar → Atacar → Atordoado → Morto*. Aqui a atribuição a "FSM" é uma **análise técnica** segura, ainda que raramente documentada título a título, pela própria natureza do hardware e das práticas da época.

**Unidades de jogos de estratégia (RTS).** Como vimos, o ciclo econômico de unidades coletoras e o ciclo militar de unidades de combate (*Parado → Mover → Engajar → Atacar → Recuar*) são naturalmente FSMs. Isso vale para clássicos do gênero como *Age of Empires* e incontáveis sucessores — novamente, uma inferência arquitetural sólida, ancorada na estrutura observável do comportamento.

**Sistemas de animação de personagens.** Um uso ubíquo e frequentemente esquecido: o **controle de animação** é quase sempre uma máquina de estados (*Parado → Andando → Correndo → Pulando → Caindo*). Na Unity, isso é *literalmente* o que o Animator Controller implementa, como veremos a seguir. Aqui não há inferência: a documentação oficial descreve o Animator como uma máquina de estados.

> **Na Indústria**
> Um padrão recorrente na indústria é usar **duas FSMs em camadas** para um mesmo personagem: uma FSM de *decisão* (patrulhar/perseguir/atacar) e uma FSM de *animação* (parado/andando/atacando), sincronizadas. A FSM de decisão diz *o que* fazer; a de animação cuida de *mostrar* isso de forma fluida. Essa separação entre "cérebro" e "corpo" é uma boa prática que reaparecerá quando estudarmos árvores de comportamento acionando um sistema de animação por baixo.

---

## 3.7 Ferramentas na Unity

Fiel à filosofia da apostila, o objetivo aqui **não** é ensinar menus, e sim mostrar *onde* o conceito de FSM se materializa nas ferramentas que o aluno usará, para que ele reconheça a teoria na prática.

**Animator Controller (Mecanim).** A ferramenta mais imediatamente reconhecível como FSM na Unity é o **Animator Controller**. Embora tenha sido concebido para controlar *animações*, ele é, na sua essência, um editor visual de máquina de estados: cada "estado de animação" é um estado da FSM; as "transições" entre eles têm **condições** baseadas em *parâmetros* (booleanos, gatilhos, floats), que são exatamente as guardas de transição que estudamos. O conceito de prioridade aparece na ordenação das transições; o conceito de ação de entrada aparece nos eventos de início de animação. Um estudante que compreendeu a seção 3.2 já entende, conceitualmente, o Animator — a diferença é apenas de vocabulário e de propósito (animação em vez de decisão). Justamente por isso, é comum — embora nem sempre recomendável — usar o Animator para a *lógica de decisão* do agente, não só para animação.

**Visual Scripting (grafos de estado).** O pacote de **Visual Scripting** da Unity oferece grafos que também podem expressar máquinas de estado de forma visual, permitindo que designers montem lógica sem escrever C#. É outra materialização do mesmo conceito, agora voltada explicitamente à lógica de jogo.

**O pacote Unity Behavior.** Mais recentemente, a Unity passou a oferecer o pacote **Unity Behavior**, uma ferramenta voltada à autoria de comportamento de agentes por meio de grafos. Embora seu foco principal sejam as *árvores de comportamento* (que estudaremos no Capítulo 6), ele também acomoda a lógica de estados e representa a direção oficial da engine para autoria visual de IA. Voltaremos a ele em profundidade no Capítulo 6; aqui basta registrar que a Unity oferece um caminho oficial e visual para construir a decisão dos agentes.

**FSM "na mão" em C#.** Fora das ferramentas visuais, é perfeitamente comum — e muitas vezes preferível — implementar a FSM diretamente em código C#, seja com um simples `enum` de estados e um `switch`, seja com o **padrão de projeto State** (cada estado como uma classe com métodos `Enter`, `Update` e `Exit`). Essa abordagem dá controle total e é a que melhor espelha os conceitos deste capítulo. Reforçamos, no entanto, o princípio da apostila: o objetivo é *compreender a estrutura*, não decorar uma implementação específica.

> **Atenção**
> Usar o **Animator Controller para toda a lógica de IA** (e não apenas para animação) é uma prática difundida, mas controversa. A ferramenta foi otimizada para animação, e sobrecarregá-la com decisões complexas mistura duas responsabilidades ("cérebro" e "corpo") que a boa arquitetura recomenda separar. Para comportamento simples, funciona bem e economiza código; para comportamento complexo, tende a reproduzir — dentro do editor de animação — a mesma explosão de transições da seção 3.5.1. A decisão é de projeto: pese simplicidade imediata contra manutenibilidade futura.

---

## 3.8 Ferramentas de terceiros

O ecossistema em torno da Unity e das demais engines oferece diversas soluções de terceiros para construir FSMs, geralmente com editores visuais mais poderosos que os nativos ou com ênfase em desempenho. Na Asset Store e em repositórios abertos, encontram-se pacotes dedicados a máquinas de estado (muitos com nomes que incluem "State Machine" ou "FSM"), bem como ferramentas mais amplas de IA — como **NodeCanvas** e **Behavior Designer** — que, embora conhecidas por árvores de comportamento, também oferecem editores de FSM/HFSM integrados. Voltaremos a essas ferramentas no Capítulo 6, pois elas cobrem várias arquiteturas de decisão sob um mesmo guarda-chuva.

Vale ainda mencionar, a título de comparação entre engines (sem sair do plano conceitual), que a **Unreal Engine** oferece o **State Tree**, uma abordagem que combina hierarquia de estados com seleção em árvore — uma evolução que dialoga tanto com este capítulo quanto com o Capítulo 4. O ponto a reter é que **toda engine profissional oferece meios visuais de construir máquinas de estado**, porque a FSM continua sendo um alicerce indispensável da IA de jogos.

> **Boa Prática**
> Ao avaliar uma ferramenta de terceiros para FSM, os critérios que importam são os mesmos que a Parte I estabeleceu para a própria IA: ela oferece **visualização e depuração** claras do estado atual? Permite **autoria pelo designer** sem programar? Tem **bom desempenho** com muitos agentes? Integra-se bem ao restante do projeto? A sofisticação do editor é secundária diante dessas perguntas práticas.

---

## 3.9 Resumo, Exercícios de fixação e Referências

### Resumo do Capítulo 3

Este capítulo apresentou a **máquina de estados finita (FSM)**, a arquitetura de decisão mais fundamental e difundida da IA de jogos. Partimos do **problema** que ela resolve — organizar comportamentos mutuamente exclusivos ao longo do tempo, dando ao agente uma noção clara de "em que modo está" e "quando muda de modo", algo que uma cadeia de condicionais reavaliada a cada quadro não oferece. Construímos seus **quatro conceitos elementares** — estado, transição, evento e ação — e vimos sua **raiz teórica** nos autômatos finitos, cuja natureza *finita e determinística* é a origem das virtudes de previsibilidade e controle tão valorizadas pela indústria. Detalhamos o **funcionamento**: o laço de avaliação por quadro, a distinção entre FSM por **polling** e por **eventos** (e seu impacto no orçamento de quadro), e o ciclo **enter/update/exit**, que resolve o problema da memória de contexto e torna a máquina legível para o designer. Ilustramos com o **inimigo guarda** de cinco estados e com o ciclo econômico de uma unidade de RTS. Pesamos **vantagens** (simplicidade, baixo custo, previsibilidade, depurabilidade, autoria visual) contra **limitações**, culminando na **explosão de transições** — o crescimento quadrático que inviabiliza a FSM plana em escala e que motivará o Capítulo 4. Por fim, aterrissamos a teoria nas ferramentas — o **Animator Controller**, o **Visual Scripting**, o pacote **Unity Behavior** e o padrão *State* em C# — e no mercado, de *Pac-Man* às unidades de RTS. A FSM é o alicerce sobre o qual as próximas arquiteturas serão erguidas e comparadas.

### Exercícios de fixação

1. Explique, com suas palavras, por que uma cadeia de condicionais reavaliada a cada quadro é insuficiente para comportamentos que dependem de **continuidade**. Que problema a FSM resolve que os condicionais não resolvem?
2. Defina, com precisão, os quatro conceitos elementares da FSM: **estado, transição, evento e ação**. Dê um exemplo de cada no contexto de um NPC.
3. O que significa dizer que a FSM é **finita e determinística**? Relacione essas duas propriedades com os critérios de qualidade da IA de jogos vistos na Parte I.
4. Diferencie **FSM por polling** e **FSM por eventos**. Dê um exemplo de condição que convém avaliar por polling e outro que convém tratar por evento, justificando com base no orçamento de quadro.
5. Explique o ciclo **enter / update / exit**. Para cada momento, dê um exemplo concreto de ação apropriada no estado *Perseguir* de um inimigo. Por que colocar um som de alerta no *update* é um erro?
6. O que é a **explosão de transições**? Mostre, com números, quantas transições, no pior caso, admitem uma FSM de 5, de 10 e de 20 estados, e explique por que esse crescimento ataca as vantagens da técnica.
7. Desenhe (ou descreva textualmente) a FSM de um **inimigo de um jogo de plataforma** com pelo menos quatro estados, especificando as ações enter/update/exit de cada estado e as guardas de transição, incluindo suas prioridades.
8. Explique por que, no caso de *Pac-Man*, a *personalidade* dos fantasmas **não** decorre de estados diferentes. Onde, então, ela reside? Relacione com o conceito de comportamento emergente.
9. O **Animator Controller** da Unity é frequentemente descrito como uma máquina de estados. Justifique essa afirmação mapeando os conceitos deste capítulo (estado, transição, guarda) aos elementos do Animator. Em seguida, discuta os prós e contras de usá-lo para a *lógica de decisão*, e não apenas para animação.
10. Dado um inimigo com os estados *Patrulhar*, *Perseguir*, *Atacar* e *Fugir*, suponha que, em um quadro, as guardas "distância < 2 m" (→ *Atacar*) e "vida < 10%" (→ *Fugir*) sejam ambas verdadeiras estando o agente em *Perseguir*. Qual deveria vencer, e por quê? Que mecanismo da FSM garante que o desempate seja consciente e não acidental?

### Referências

BOURG, David M.; SEEMANN, Glenn. *AI for Game Developers: Creating Intelligent Behavior in Games.* Sebastopol, CA: O'Reilly Media, 2004. (Capítulos sobre máquinas de estado aplicadas ao comportamento de agentes.)

MILLINGTON, Ian. *AI for Games.* 3. ed. Boca Raton: CRC Press, 2019. (Referência principal para máquinas de estado, incluindo FSMs por polling e por eventos, e a discussão de suas limitações.)

RABIN, Steve (org.). *Game AI Pro 3: Collected Wisdom of Game AI Professionals.* Boca Raton: A K Peters/CRC Press, 2017. (Artigos sobre arquiteturas de decisão e boas práticas de implementação na indústria.)

RUSSELL, Stuart J.; NORVIG, Peter. *Inteligência Artificial.* 3. ed. Rio de Janeiro: Elsevier, 2013. (Fundamentos de agentes reativos e da relação entre percepção, estado interno e ação.)

HOPCROFT, John E.; MOTWANI, Rajeev; ULLMAN, Jeffrey D. *Introduction to Automata Theory, Languages, and Computation.* (Referência conceitual para a teoria dos autômatos finitos, raiz matemática da FSM.)

UNITY TECHNOLOGIES. *Documentação oficial da Unity: Animator Controller, Visual Scripting e pacote Unity Behavior.* (Referência para a materialização do conceito de FSM nas ferramentas da engine.)

> **Nota sobre fontes.** A descrição da IA de *Pac-Man* baseia-se em análises técnicas públicas e amplamente reconhecidas do jogo. As atribuições de FSM a inimigos de jogos de ação/plataforma e a unidades de RTS constituem **análise técnica fundamentada** na estrutura observável do comportamento e nas práticas de desenvolvimento da época, e não documentação oficial título a título, salvo quando explicitamente indicado.
