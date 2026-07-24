# Capítulo 2 — História e Evolução da IA em Jogos

## Introdução

O Capítulo 1 nos deu o vocabulário e os critérios para pensar a IA de jogos. Este capítulo acrescenta a dimensão que falta: **o tempo**. Nenhuma técnica de IA de jogos surgiu no vácuo. Cada uma nasceu como resposta a um problema concreto, dentro dos limites do hardware de sua época, e foi substituída ou complementada quando esses limites mudaram. Compreender essa evolução histórica é indispensável, porque explica **por que** cada técnica existe e **por que** os capítulos seguintes estão organizados como estão.

A história da IA de jogos é, em grande medida, a história de uma **negociação permanente com a escassez de recursos**. Nos primeiros arcades, cada byte de memória e cada ciclo de processamento eram preciosíssimos; a "inteligência" precisava caber em restrições que hoje pareceriam absurdas. À medida que o hardware evoluiu, técnicas antes impensáveis tornaram-se viáveis, e novos problemas — antes irrelevantes — passaram a merecer solução. Essa relação íntima entre **hardware e técnica** é o fio condutor deste capítulo.

Seguindo a orientação do projeto, adotamos uma abordagem **cronológica** e usamos **jogos reais** como marcos. Sempre que uma informação sobre a IA de um jogo não for oficialmente confirmada por seus desenvolvedores, isso será dito de forma explícita: trataremos como **análise técnica fundamentada**, não como fato documentado. Essa cautela é essencial em um campo onde a documentação técnica raramente é publicada pelos estúdios.

> ⚠️ **Atenção**
> Muitas afirmações populares sobre "como funciona a IA de tal jogo" circulam pela internet como se fossem fatos, quando são, na verdade, inferências de jogadores ou análises não confirmadas. Nesta apostila, distinguimos rigorosamente o que foi **documentado pelos próprios desenvolvedores** (por exemplo, em palestras da GDC ou artigos técnicos) do que é **análise provável**. Adote você também essa disciplina: é a base da engenharia reversa responsável, tema da Parte VII.

---

## 2.1 O problema: como a IA evoluiu junto com o hardware

Para entender a evolução da IA de jogos, é preciso primeiro entender a **restrição** que a moldou. Diferentemente da IA acadêmica, que ao longo das décadas pôde contar com computadores de pesquisa cada vez mais potentes e com tempo de processamento praticamente ilimitado para muitos problemas, a IA de jogos sempre operou sob a **tirania do tempo real** e do **hardware de consumo**.

Três restrições históricas moldaram cada geração de técnicas:

- **Memória.** Os primeiros arcades e consoles tinham quantidades ínfimas de memória — kilobytes, não gigabytes. Não havia espaço para armazenar mapas elaborados, históricos de estado ou modelos de mundo. A IA precisava ser *compacta*.
- **Processamento.** O poder de cálculo era minúsculo, e a IA disputava-o com tudo o mais. Algoritmos que exigissem muitos cálculos por quadro eram simplesmente inviáveis.
- **Tempo real.** Diferentemente de um programa de xadrez acadêmico, que podia "pensar" por minutos, a IA de jogos sempre precisou decidir em uma fração de segundo, quadro após quadro.

Essas restrições explicam por que a IA de jogos, durante décadas, privilegiou técnicas **determinísticas, compactas e baratas** — padrões de movimento, máquinas de estado, scripts — em vez das técnicas mais "ambiciosas" da pesquisa acadêmica. Não era ignorância: era **engenharia sob restrição**. À medida que memória e processamento cresceram (seguindo, por décadas, a tendência descrita pela Lei de Moore), o teto do que era viável subiu, e a IA de jogos pôde incorporar técnicas progressivamente mais sofisticadas — sempre, porém, dentro do orçamento de quadro de sua época.

[DIAGRAMA]
Título: A coevolução de hardware e técnicas de IA de jogos
Objetivo pedagógico: Mostrar que cada salto de capacidade do hardware habilitou novas famílias de técnicas de IA, evidenciando a relação de causa e efeito.
Descrição detalhada: Um gráfico de duas faixas paralelas ao longo de um eixo temporal horizontal (dos anos 1970 aos anos 2020). Faixa superior "HARDWARE": marcos como "arcades (KB de memória)", "consoles 8/16 bits", "PCs anos 1990", "consoles 3D anos 2000", "hardware multinúcleo e GPU", "computação em nuvem / GPUs para ML". Faixa inferior "TÉCNICAS DE IA DE JOGOS", alinhada temporalmente: "padrões de movimento", "máquinas de estado e scripts", "pathfinding A*", "árvores de comportamento e GOAP", "aprendizado de máquina e IA procedural orientada a dados". Setas verticais ligam cada avanço de hardware às técnicas que ele habilitou.
Elementos que devem aparecer: eixo temporal; duas faixas paralelas (hardware e técnicas); setas verticais de habilitação; marcos rotulados em ambas as faixas.
[/DIAGRAMA]

> 🎲 **Curiosidade**
> O primeiro jogo eletrônico com um oponente controlado por computador amplamente reconhecido é frequentemente associado a *Pong* (Atari, 1972), cuja "IA" era simplesmente uma raquete que seguia a bola — mal se poderia chamar de inteligência. Ainda assim, já continha o germe de toda IA de jogos: um comportamento automático que dá ao jogador algo com que se medir. De uma raquete que segue a bola aos sistemas atuais, o salto é imenso — mas a pergunta de fundo permaneceu a mesma: *como criar um adversário convincente dentro dos recursos disponíveis?*

---

## 2.2 Primeiros jogos: perseguição, padrões e a IA dos fantasmas

Nos anos 1970 e início dos 1980, a era de ouro dos arcades, a IA de jogos deu seus primeiros passos memoráveis. Com memória e processamento extremamente limitados, os desenvolvedores recorreram a soluções engenhosas baseadas em **padrões predefinidos** e **regras simples de perseguição**.

Em *Space Invaders* (Taito, 1978), os alienígenas não tinham propriamente "inteligência": moviam-se em um padrão coreografado, descendo e acelerando conforme eram eliminados. A aceleração — na verdade, um efeito colateral do hardware, que processava menos invasores mais rápido — foi percebida pelos jogadores como uma *escalada de tensão* proposital. Um belo exemplo, ainda que acidental, de como o comportamento observável (e não o mecanismo) define a experiência.

Foi, porém, com *Pac-Man* (Namco, 1980) que a IA de jogos produziu seu primeiro caso verdadeiramente célebre e didático — a ponto de ser estudado até hoje. Por isso ele merece subseção própria.

### 2.2.1 Pac-Man e as personalidades dos fantasmas

*Pac-Man* apresenta quatro fantasmas que perseguem o jogador pelo labirinto: Blinky (vermelho), Pinky (rosa), Inky (ciano) e Clyde (laranja). O que torna esse jogo um marco da IA não é a complexidade técnica — é modestíssima — mas a **eficácia com que produz a ilusão de quatro personalidades distintas a partir de regras simples**. Aqui, a informação é razoavelmente **documentada**: o design da IA dos fantasmas foi analisado em detalhe e é bem compreendido pela comunidade técnica.

Cada fantasma alterna entre modos (estados) — essencialmente uma **máquina de estados** simples, tema do Capítulo 3 — e, no modo de perseguição, cada um usa uma **regra de mira (targeting) diferente**:

- **Blinky (vermelho)** persegue diretamente a posição atual de Pac-Man. É o mais agressivo, o que o jogador percebe como "o caçador".
- **Pinky (rosa)** mira alguns quadros à *frente* da direção de Pac-Man, tentando emboscá-lo. É percebido como "o que tenta cortar o caminho".
- **Inky (ciano)** usa um cálculo mais complexo que combina a posição de Pac-Man com a posição de Blinky, resultando em movimento imprevisível. É percebido como "o errático".
- **Clyde (laranja)** persegue Pac-Man quando está longe, mas foge para seu canto quando chega perto. É percebido como "o tímido/covarde".

**Análise interpretativa.** O brilho do design está em três decisões que ecoam tudo o que vimos no Capítulo 1. Primeiro, **personalidade emergente a partir de regras simples**: nenhum fantasma tem "personalidade" programada como tal; ela *emerge* da regra de mira e é *lida* pelo jogador como caráter — a ilusão de inteligência em estado puro. Segundo, **previsibilidade com margem**: os comportamentos são determinísticos o bastante para que jogadores experientes aprendam padrões e rotas de fuga (a tal "previsibilidade que permite estratégia" da seção 1.2.2), mas a combinação dos quatro gera situações variadas. Terceiro, os fantasmas alternam periodicamente entre modos de **perseguição (chase)** e **dispersão (scatter)**, em que recuam para seus cantos — um alívio rítmico deliberado que impede a frustração constante, exatamente o tipo de gestão do "ponto ideal" discutido em 1.2.3.

> 🎮 **Na Prática**
> *Pac-Man* é, provavelmente, o melhor exemplo introdutório de IA de jogos que existe: com quatro regras simples e uma máquina de estados mínima, produz-se a ilusão de quatro adversários com personalidades e intenções. Ele demonstra, num único jogo, quase todos os princípios do Capítulo 1 — ilusão de inteligência, comportamento emergente, previsibilidade estratégica e gestão da dificuldade. Voltaremos a ele no Capítulo 3 (como exemplo de FSM) e no Capítulo 15 (como estudo de caso completo).

> 🎲 **Curiosidade**
> Os quatro fantasmas têm apelidos além dos nomes: na versão original japonesa, seus comportamentos eram descritos por características como "perseguidor", "emboscador", "caprichoso" e "tolo". O design foi intencionalmente pensado para que os fantasmas *não* atacassem todos da mesma forma — o criador queria evitar que o jogo fosse tenso e implacável o tempo todo, buscando justamente o equilíbrio de ritmo que hoje reconhecemos como gestão do fluxo.

[IMAGEM NECESSÁRIA]
Título: As quatro regras de mira dos fantasmas de Pac-Man
Objetivo didático: Ilustrar como cada fantasma usa uma regra de alvo (targeting) diferente, produzindo "personalidades" distintas a partir de lógica simples.
Descrição: Um esquema (não uma captura do jogo original, para evitar violação de direitos autorais) representando um labirinto estilizado com Pac-Man e os quatro fantasmas. Setas coloridas mostram o alvo de cada fantasma: Blinky mira a posição atual de Pac-Man; Pinky mira uma posição à frente; Inky mira um ponto calculado a partir de Blinky e de Pac-Man; Clyde alterna entre perseguir e recuar para o canto. Uma legenda associa cada cor à sua "personalidade percebida".
Tipo: Ilustração / esquema (recriação estilizada, não screenshot do original)
Como produzir: Ilustração vetorial original com labirinto e personagens genéricos (evitar a arte protegida da Namco); setas e anotações explicativas sobre as regras de mira.
Legenda sugerida: Figura 2.1 — Quatro regras de mira simples produzem quatro "personalidades" distintas: o cerne da ilusão de inteligência.
[/IMAGEM NECESSÁRIA]

---

## 2.3 A era das máquinas de estado e dos scripts

Ao longo dos anos 1980 e, sobretudo, 1990, com consoles de 8 e 16 bits e os primeiros PCs domésticos, a IA de jogos consolidou duas ferramentas que dominariam a indústria por décadas e que permanecem centrais até hoje: as **máquinas de estado finitas (FSM)** e os **scripts**.

A **máquina de estados finita** organiza o comportamento de um personagem em um conjunto de **estados** (patrulhar, perseguir, atacar, fugir) e **transições** entre eles, disparadas por eventos ou condições (avistar o jogador, perder de vista, ficar sem vida). É uma formalização natural do agente reativo do Capítulo 1, e tornou-se onipresente por reunir todas as virtudes que a indústria valoriza: é **compacta** (cabia na memória da época), **barata** (pouquíssimo processamento), **previsível**, **depurável** e **controlável pelo designer**. Dedicaremos a ela todo o Capítulo 3.

Os **scripts**, por sua vez, permitiam aos designers definir sequências de comportamento predeterminadas — o inimigo que sempre aparece naquele ponto, dispara naquela hora e recua para aquele local. Jogos de tiro e ação dos anos 1990 dependiam fortemente de scripts para criar sequências dramáticas e controladas. A vantagem era o **controle autoral absoluto**; a desvantagem, a **rigidez**: comportamentos roteirizados não se adaptam ao imprevisto e perdem o efeito na repetição.

> 🕰️ **Contexto Histórico**
> Muitos jogos marcantes dessa era combinavam FSMs para o comportamento de combate com scripts para os momentos roteirizados. Essa combinação — uma base reativa (FSM) sob uma camada de eventos autorais (scripts) — é um exemplo inicial da arquitetura **híbrida** discutida em 1.3.2, e prenuncia a forma como jogos modernos ainda organizam sua IA.

Um marco frequentemente citado dessa era é *Half-Life* (Valve, 1998), elogiado por seus inimigos soldados que pareciam **coordenar-se em esquadrão** — flanqueando, recuando, lançando granadas para forçar o jogador a sair da cobertura. É importante o enquadramento correto: embora o resultado *pareça* coordenação tática sofisticada, boa parte do efeito vinha de FSMs bem ajustadas combinadas com **falas contextuais** ("*Cobertura!*", "*Ele está atrás daquilo!*") que comunicavam intenção ao jogador. É o princípio da seção 1.3.1 em ação: a comunicação do estado interno (por voz e animação) faz o jogador *ler* como inteligência algo que, internamente, é relativamente simples.

> ❌ **Erro Comum**
> Concluir que, se um jogo antigo "parecia inteligente", ele devia usar técnicas avançadas. Frequentemente ocorria o oposto: a impressão de inteligência vinha de **boa comunicação** (falas, animações, timing) sobre uma lógica simples. Confundir o comportamento observável com o mecanismo interno é justamente o erro que a ideia de "ilusão de inteligência" nos ensina a evitar.

---

## 2.4 A virada dos anos 2000: árvores de comportamento, GOAP e planejamento

Na passagem para os anos 2000, com consoles capazes de gráficos 3D e muito mais memória e processamento, os jogos tornaram-se mais complexos — mundos maiores, mais personagens, situações menos roteirizáveis. As FSMs, embora continuassem úteis, começaram a mostrar um limite estrutural conhecido como **explosão de estados e transições**: à medida que o número de comportamentos cresce, o número de transições entre eles cresce muito mais rápido, tornando a máquina um emaranhado impossível de manter (problema que detalharemos no Capítulo 3 e cuja solução hierárquica veremos no Capítulo 4).

Duas respostas a esse limite marcaram a década e definiram boa parte da IA de jogos moderna: as **árvores de comportamento** e o **planejamento orientado a objetivos (GOAP)**.

As **árvores de comportamento (Behavior Trees)** organizam o comportamento como uma árvore de tarefas — compostas por nós de sequência, seleção e decoração — que é reavaliada periodicamente. Elas oferecem a **modularidade** e a **reutilização** que faltavam às FSMs, permitindo montar comportamentos complexos a partir de peças simples e combináveis, e são muito amigáveis a **edição visual por designers**. Tornaram-se, desde então, uma das arquiteturas dominantes na indústria. Um marco é *Halo 2* (Bungie, 2004): a equipe da Bungie documentou publicamente (em palestras técnicas) o uso de árvores de comportamento para orquestrar o comportamento dos inimigos — um dos casos de adoção documentada mais influentes. Dedicaremos à árvore de comportamento o Capítulo 6, o capítulo de ênfase da Parte II.

O **GOAP (Goal-Oriented Action Planning)** representa uma abordagem diferente: em vez de o designer especificar *como* o agente deve se comportar, ele especifica **objetivos** e um conjunto de **ações** com **pré-condições** e **efeitos**; o agente então *planeja*, em tempo de execução, uma sequência de ações que atinge o objetivo. É um exemplo claro de agente **deliberativo** (seção 1.3.2), que raciocina sobre o futuro. O caso célebre e **documentado** é *F.E.A.R.* (Monolith, 2005), cujos inimigos foram amplamente elogiados por parecerem taticamente brilhantes — flanqueando, buscando cobertura, coordenando-se. O desenvolvedor Jeff Orkin apresentou publicamente o sistema GOAP de *F.E.A.R.*, tornando-o uma das referências mais estudadas da IA de jogos. Estudaremos o GOAP como conteúdo de **aprofundamento** no Capítulo 6 e retomaremos *F.E.A.R.* no Capítulo 15.

> 🏭 **Na Indústria**
> A IA de *F.E.A.R.* é um dos exemplos mais instrutivos da tese desta apostila. Análises técnicas do próprio criador apontam que grande parte da impressão de "esquadrão genial" vinha não só do planejamento GOAP, mas também de **falas coordenadas** e de comportamentos que *comunicavam* a tática ao jogador. Novamente: parte da inteligência percebida é **ilusão bem construída**. Mesmo com um planejador real por trás, é a comunicação que converte o cálculo em experiência.

> 🎲 **Curiosidade**
> Ainda nessa era surgiu, em *The Sims* (Maxis, 2000), uma abordagem distinta: a **IA de utilidade** combinada com **objetos inteligentes** (*smart objects*). Em vez de a IA do personagem conter toda a lógica, os próprios objetos do mundo "anunciam" as ações que oferecem e o quanto satisfazem as necessidades do Sim (uma geladeira anuncia "reduzo a fome"). O Sim apenas escolhe a ação de maior utilidade no momento. É uma forma elegante de distribuir a inteligência pelo ambiente — que estudaremos como aprofundamento no Capítulo 6 e como estudo de caso no Capítulo 15.

---

## 2.5 A era dos dados: aprendizado de máquina e IA procedural

A partir dos anos 2010, dois movimentos convergentes abriram uma nova fase: o crescimento explosivo do poder de processamento (em especial das **GPUs**) e a disponibilidade de **grandes volumes de dados** de jogadores. Juntos, tornaram viáveis abordagens antes restritas à pesquisa: o **aprendizado de máquina** aplicado a jogos e uma geração mais ambiciosa de **conteúdo procedural** e sistemas orientados a dados.

É preciso, aqui, um enquadramento honesto e cuidadoso — o mesmo que o Planejamento Editorial recomenda. O aprendizado de máquina obteve resultados espetaculares em **pesquisa** e em **jogos como domínio de teste para IA**: sistemas que aprenderam a jogar Atari, Go (AlphaGo, 2016), xadrez e jogos complexos de estratégia em tempo real superaram campeões humanos. Esses feitos, porém, pertencem majoritariamente ao campo da **pesquisa em IA**, que usa jogos como *ambiente de treino*, e não ao campo da **IA comercial de entretenimento**, que precisa de controle, previsibilidade e baixo custo.

Na prática comercial, o aprendizado de máquina entrou de forma mais **cautelosa e localizada**, justamente pelas razões vistas no Capítulo 1: é difícil de controlar, de depurar e de ajustar à visão de design, além de caro de treinar. Seus usos mais consolidados na indústria tendem a ser **auxiliares**: teste automatizado e balanceamento de jogos (agentes que jogam milhares de partidas para encontrar exploits), animação orientada a dados, geração de conteúdo, personalização e detecção de trapaça, entre outros. O comportamento dos NPCs no jogo final, na maioria dos títulos, continua governado por técnicas determinísticas.

> ⚠️ **Atenção**
> É tentador — e comum na imprensa — anunciar que "a IA dos jogos agora aprende sozinha". Trate essas afirmações com ceticismo técnico. Na maioria dos jogos comerciais, o comportamento dos personagens **não** usa aprendizado de máquina em tempo de jogo; usa FSMs, árvores de comportamento e busca. O aprendizado, quando presente, costuma atuar **nos bastidores** (produção, teste, balanceamento) ou em usos muito específicos. Estudaremos a aprendizagem por reforço com esse enquadramento realista na Parte VI.

Um exemplo pioneiro e frequentemente citado de aprendizado *dentro* do jogo é *Black & White* (Lionhead, 2001): a "Criatura" do jogador aprendia por reforço e imitação, ajustando seu comportamento conforme era recompensada ou repreendida. É um caso raro e influente de aprendizado como *mecânica central de jogo*, e não como ferramenta de bastidor — razão pela qual será nosso estudo de caso de aprendizado no Capítulo 12 e no Capítulo 15.

Paralelamente, a **geração procedural de conteúdo** (PCG) amadureceu, criando mundos, fases, itens e até narrativas por algoritmo. Embora nem toda PCG seja "IA" no sentido de tomada de decisão de agentes, ela se entrelaça com o tema — por exemplo, na geração de comportamentos ou no ajuste dinâmico de dificuldade (o *AI Director* de *Left 4 Dead*, já citado, é um sistema orientado a dados que molda a experiência em tempo real).

> 🏭 **Na Indústria**
> A Unity oferece o **ML-Agents**, um conjunto de ferramentas que permite treinar agentes com aprendizagem por reforço dentro da engine. É a ferramenta que usaremos como referência ao estudar aprendizado (Parte VI). Vale notar, porém, que o ML-Agents é hoje mais empregado em **pesquisa, prototipagem e teste** do que na IA final de jogos comerciais — coerente com o quadro realista descrito nesta seção.

---

## 2.6 Linha do tempo consolidada

Reunimos a seguir os marcos discutidos numa visão cronológica única. A tabela relaciona cada era ao **hardware que a habilitou**, à **técnica dominante** e a **jogos de referência**. Para cada jogo, indicamos se a informação sobre sua IA é majoritariamente **documentada** (D) pelos desenvolvedores ou **analisada/provável** (A).

| Período | Hardware / contexto | Técnica dominante de IA | Jogos de referência | Fonte |
|---|---|---|---|---|
| Anos 1970 | Arcades, memória em KB | Padrões predefinidos e regras de perseguição | *Pong* (1972), *Space Invaders* (1978) | A |
| ~1980 | Arcades | Máquina de estados simples + regras de mira | *Pac-Man* (1980) | D |
| Anos 1980–1990 | Consoles 8/16 bits, PCs | Máquinas de estado finitas (FSM) e scripts | Jogos de ação/plataforma da era; *Half-Life* (1998) | A |
| Início dos anos 2000 | Consoles 3D, mais memória/CPU | Árvores de comportamento; IA de utilidade | *Halo 2* (2004), *The Sims* (2000) | D |
| Meados dos anos 2000 | Consoles 3D | Planejamento (GOAP) | *F.E.A.R.* (2005) | D |
| Fins dos anos 2000 | Multinúcleo | Sistemas diretores / IA adaptativa por regras | *Left 4 Dead* (2008) | D |
| Anos 2010 em diante | GPUs, dados em larga escala | Aprendizado de máquina (majoritariamente pesquisa e bastidores); PCG | *Black & White* (2001, pioneiro); *AlphaGo* (2016, pesquisa) | D |

**Análise interpretativa.** A leitura da tabela confirma o fio condutor do capítulo: **cada nova família de técnicas surgiu quando o hardware a tornou viável e um problema de design a tornou necessária**. As FSMs não foram "superadas" pelas árvores de comportamento — elas continuam vivas e são a base do Capítulo 3; as árvores de comportamento surgiram para resolver um limite específico (a explosão de estados) que só se tornou crítico quando os jogos ficaram grandes o bastante. Da mesma forma, o aprendizado de máquina não "substituiu" as técnicas clássicas na prática comercial; ocupou nichos onde suas vantagens compensam suas dificuldades de controle. Essa é a lição histórica mais importante para o restante da apostila: **as técnicas se acumulam em camadas, não se substituem em bloco**. Um jogo moderno pode usar, simultaneamente, uma FSM dos anos 1980, um A\* dos anos 1960–1990 e um sistema orientado a dados dos anos 2010 — cada um resolvendo a parte do problema para a qual é a melhor ferramenta.

> ✅ **Boa Prática**
> Ao estudar cada técnica nos próximos capítulos, retorne mentalmente a esta linha do tempo e pergunte: *que problema histórico fez essa técnica surgir? que restrição de hardware ela respeitava? o que ela resolveu que a anterior não resolvia?* Essa perspectiva histórica transforma uma lista de algoritmos em uma **narrativa de engenharia** — muito mais fácil de compreender e de reter.

[IMAGEM NECESSÁRIA]
Título: Linha do tempo da IA em jogos
Objetivo didático: Consolidar visualmente a evolução cronológica das técnicas de IA de jogos e sua relação com marcos de jogos e de hardware.
Descrição: Uma linha do tempo horizontal, dos anos 1970 aos anos 2020, com marcadores para cada jogo de referência (Pong, Space Invaders, Pac-Man, Half-Life, The Sims, Halo 2, F.E.A.R., Left 4 Dead, Black & White, AlphaGo). Acima da linha, faixas indicando a técnica dominante de cada era; abaixo, ícones de hardware representativos da época. Marcadores de jogos "documentados" e "analisados" com símbolos distintos (por exemplo, ✓ para documentado, ~ para análise provável).
Tipo: Linha do tempo (timeline)
Como produzir: Infográfico vetorial de linha do tempo (Illustrator, Figma ou ferramenta de timeline), com ícones originais e sem uso de arte protegida dos jogos; incluir legenda dos símbolos documentado/analisado.
Legenda sugerida: Figura 2.2 — Meio século de IA em jogos: técnicas dominantes, jogos-marco e a base de hardware que os habilitou.
[/IMAGEM NECESSÁRIA]

---

## Resumo do Capítulo 2

Este capítulo percorreu a **evolução histórica** da IA de jogos, sempre sob a chave de leitura da **coevolução com o hardware**. Vimos que a IA de jogos foi moldada pela **tirania do tempo real** e pela escassez de memória e processamento, o que a levou, por décadas, a privilegiar técnicas **determinísticas, compactas e baratas**. Percorremos os **primeiros jogos** (padrões de movimento e perseguição), com destaque para *Pac-Man* e suas quatro "personalidades" emergentes de regras simples — um caso documentado e exemplar da ilusão de inteligência. Estudamos a **era das FSMs e scripts** (anos 1980–1990), base controlável e barata que domina até hoje, e o exemplo de *Half-Life*, onde a inteligência percebida vinha em boa parte da comunicação (falas e timing). Vimos a **virada dos anos 2000**, com as **árvores de comportamento** (*Halo 2*) e o **planejamento GOAP** (*F.E.A.R.*) respondendo ao limite da explosão de estados, além da IA de utilidade de *The Sims*. Por fim, examinamos a **era dos dados** (anos 2010+), com aprendizado de máquina e PCG, distinguindo com cuidado o uso do aprendizado em **pesquisa e bastidores** de seu uso, ainda restrito, na IA comercial em tempo de jogo. A **linha do tempo consolidada** sintetizou tudo e revelou a lição central: **as técnicas se acumulam em camadas, não se substituem em bloco**. Cada capítulo seguinte estudará, em profundidade, uma dessas camadas.
