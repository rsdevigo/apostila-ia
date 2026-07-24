# Capítulo 13 — Heurísticas e Algoritmos Genéticos

## Introdução

O capítulo anterior tratou de **aprender a agir**: um agente que, pela interação e pela recompensa, descobre como se comportar. Este capítulo trata de uma pergunta aparentada, mas distinta: **como encontrar uma boa solução quando o espaço de possibilidades é gigantesco demais para examinar por inteiro?**. Não se trata mais de um agente aprendendo um comportamento ao longo do tempo, e sim de um processo de **busca e otimização** que vasculha um imenso conjunto de candidatos à procura do melhor — ou, mais realisticamente, de um **suficientemente bom**.

A resposta que este capítulo desenvolve vem de uma ideia bela e improvável: **imitar a evolução biológica**. Assim como a seleção natural, ao longo de gerações, "descobre" organismos extraordinariamente bem adaptados sem que ninguém os projete, um **Algoritmo Genético** faz evoluir, ao longo de gerações artificiais, soluções cada vez melhores para um problema — mantendo uma **população** de candidatos, favorecendo os mais **aptos**, **cruzando-os** para gerar descendentes e introduzindo **mutações** para explorar o novo. É uma das técnicas mais elegantes da Inteligência Artificial, e pertence a uma família maior — as **metaheurísticas** e a **computação evolutiva** — dedicada a otimizar o inatingível.

Fiéis à estrutura da apostila, partimos do **problema** (espaços de busca astronômicos e a ideia de otimização heurística), construímos os **fundamentos** (evolução, população, indivíduo, cromossomo, gene, aptidão), detalhamos o **funcionamento** do algoritmo genético e de seus **operadores** (seleção, cruzamento, mutação, elitismo), discutimos a peça mais delicada — a **função de aptidão** —, e aterrissamos em **exemplos**, **aplicações**, **ferramentas**, **vantagens e limitações** e **estudos de caso**. Ao longo de todo o caminho, uma comparação estará sempre presente: **em que os algoritmos genéticos diferem da Aprendizagem por Reforço, e quando cada um é a ferramenta certa?**

> **Contexto Histórico**
> A ideia de simular a evolução em um computador é quase tão antiga quanto a própria computação. Nos anos 1950 e 1960, vários pesquisadores exploraram "estratégias evolutivas" e "programação evolutiva". Mas foi **John Holland**, na Universidade de Michigan, quem formalizou os **Algoritmos Genéticos** em sua forma moderna, culminando no livro *Adaptation in Natural and Artificial Systems* (1975) — obra que introduziu os conceitos de população, cromossomo, cruzamento e mutação como os conhecemos, e o famoso "teorema dos esquemas". Seu aluno **David Goldberg** popularizou a técnica na engenharia com *Genetic Algorithms in Search, Optimization, and Machine Learning* (1989). Os algoritmos genéticos fazem parte de um guarda-chuva maior, a **computação evolutiva**, que inclui também programação genética, estratégias evolutivas e evolução diferencial. A inspiração declarada é, evidentemente, a teoria da evolução por seleção natural de **Charles Darwin** (1859) — uma das raras vezes em que a biologia do século XIX fecunda diretamente a engenharia do século XXI.

---

## 13.1 O problema: otimizar e gerar comportamentos em espaços enormes

Como sempre, comecemos pelo problema concreto — e por reconhecer que ele é de uma natureza diferente de tudo o que vimos até agora.

### Quando "experimentar tudo" é impossível

Muitos problemas de jogos podem ser formulados assim: "entre todas as configurações possíveis de X, encontre a melhor". Por exemplo: entre todas as combinações de parâmetros da IA inimiga (agressividade, precisão, tempo de reação, distância de engajamento...), qual produz o oponente mais divertido e balanceado? Entre todos os layouts possíveis de uma fase gerada proceduralmente, qual é o mais interessante e jogável? Entre todas as estratégias possíveis para um time de NPCs, qual vence com mais frequência?

A tentação óbvia é **testar todas as possibilidades** e escolher a melhor — a chamada **busca exaustiva** ou "força bruta". Para problemas pequenos, funciona. Mas o número de combinações **explode** de forma assustadora. Suponha apenas 10 parâmetros da IA, cada um com 10 valores possíveis: são 10¹⁰ = **dez bilhões** de combinações. Testar cada uma, se cada teste exigisse simular uma partida de poucos segundos, levaria **séculos**. Aumente para 20 parâmetros e o número ultrapassa a idade do universo em qualquer computador imaginável. Este é o fenômeno da **explosão combinatória** — o mesmo inimigo que enfrentamos no Minimax (Capítulo 11), agora em um contexto de otimização.

O **espaço de busca** — o conjunto de todas as soluções candidatas — é simplesmente **grande demais** para ser examinado por completo. E, diferentemente do pathfinding (onde A\* encontra o caminho **ótimo** com eficiência), aqui muitas vezes **não existe** algoritmo eficiente que garanta a solução ótima: são problemas da classe que a Ciência da Computação chama de "difíceis" (NP-difíceis e afins), para os quais nenhum método conhecido encontra o ótimo em tempo razoável.

### Otimização heurística: "bom o suficiente" no lugar do "ótimo"

Diante do impossível, muda-se a pergunta. Em vez de exigir **a melhor** solução (inatingível), passamos a buscar uma solução **boa o suficiente**, encontrada em **tempo aceitável**. Essa é a essência da **otimização heurística**: usar estratégias inteligentes de busca que, sem garantir o ótimo, encontram soluções de **alta qualidade** explorando apenas uma fração minúscula do espaço.

A palavra **heurística** já nos é familiar. No Capítulo 8, uma heurística era uma **estimativa** que guiava o A\* em direção ao objetivo (a distância em linha reta). O sentido aqui é o mesmo, generalizado: uma heurística é uma **regra prática de busca** que orienta a procura para regiões promissoras do espaço, sem examinar tudo e sem garantia de perfeição. Uma **metaheurística** é uma heurística de alto nível, genérica, aplicável a muitos problemas diferentes — e os Algoritmos Genéticos são justamente uma das metaheurísticas mais conhecidas.

> **Boa Prática**
> Aceitar o "bom o suficiente" não é preguiça de engenharia — é **realismo matemático**. Para a maioria dos problemas de otimização de jogos, a diferença entre a solução ótima (que ninguém consegue encontrar) e uma ótima solução heurística (que encontramos em minutos) é irrelevante para o jogador. Um balanceamento "95% ideal" obtido automaticamente é infinitamente mais útil do que um balanceamento "100% ideal" que jamais terminaria de calcular. Saber quando parar de perseguir o ótimo é uma marca de maturidade profissional.

### Quando o "bom o suficiente" é o que importa — exemplos em jogos

Vários problemas de jogos têm exatamente esse perfil (voltaremos a eles na Seção 13.7):

- **Balanceamento automático.** Ajustar dezenas de parâmetros de unidades, armas ou economia para que nenhuma estratégia domine. O espaço de combinações é gigantesco; qualquer configuração "bem balanceada" serve, não é preciso a perfeita.
- **Geração procedural de conteúdo.** Evoluir fases, mapas, dungeons, missões ou até regras de jogo que satisfaçam critérios (jogável, interessante, difícil na medida certa). Há infinitas fases possíveis; queremos boas fases, não *a* fase ótima.
- **Evolução de comportamentos.** Fazer emergir estratégias ou controladores de NPCs por evolução, quando não sabemos programá-los à mão.
- **Ajuste de parâmetros de IA.** Calibrar os muitos números de uma FSM, behavior tree ou função de avaliação para maximizar desempenho ou diversão.

Em todos, a estrutura é a mesma: **um espaço enorme de candidatos, um critério de qualidade, e a necessidade de encontrar um bom candidato sem examinar todos.** É o problema que os Algoritmos Genéticos resolvem.

> **Atenção — otimização não é aprendizado**
> Marque desde já a distinção que atravessa todo o capítulo e que a Parte inteira quer que você domine. No Capítulo 12, um **agente** aprendia a **agir** ao longo do tempo, pela interação com um ambiente, ajustando um comportamento (a política). Aqui, um algoritmo **busca**, num espaço de candidatos, aquele que **maximiza um critério** (a aptidão) — não há necessariamente um "agente" que "age" nem interação contínua com um ambiente ao longo de um episódio. RL responde "**como devo me comportar?**"; um Algoritmo Genético responde "**qual é a melhor configuração/solução?**". São ferramentas para perguntas diferentes, ainda que ambas "melhorem com o tempo" e ambas sejam frequentemente confundidas.

---

## 13.2 Fundamentos: evolução, população, indivíduo, cromossomo, gene e aptidão

Antes do algoritmo, o vocabulário. Os Algoritmos Genéticos tomam emprestada a terminologia da biologia evolutiva — e, para usá-la com precisão sem nos perdermos em biologia, definiremos cada termo **no sentido computacional** que ele assume aqui, mencionando a analogia natural apenas na medida em que ela ajuda a compreender.

### A metáfora da evolução

A ideia central vem da **evolução por seleção natural**: em uma população de organismos que variam entre si, os mais **adaptados** ao ambiente tendem a **sobreviver e se reproduzir**, passando suas características aos descendentes; ao longo de muitas gerações, com variação (mutação) e recombinação (reprodução sexual), a população como um todo se torna cada vez mais adaptada. Nenhum "projetista" desenha os organismos — a adaptação **emerge** do ciclo repetido de variação e seleção.

O Algoritmo Genético reproduz esse ciclo **artificialmente** para resolver um problema: cada "organismo" é uma **solução candidata**; o "quão adaptado" é dado por uma função que mede a **qualidade** da solução; e as "gerações" são iterações em que boas soluções se combinam e mutam para produzir soluções ainda melhores. Vejamos cada peça.

### Indivíduo e cromossomo

Um **indivíduo** é **uma solução candidata** para o problema — uma resposta possível, boa ou ruim. Se o problema é "ajustar os parâmetros da IA inimiga", um indivíduo é **um conjunto específico** de valores para esses parâmetros. Se o problema é "gerar uma fase", um indivíduo é **uma fase concreta**.

O **cromossomo** é a **representação codificada** desse indivíduo — a forma como a solução é escrita para que o algoritmo possa manipulá-la. Tipicamente, o cromossomo é uma **sequência** (uma lista, um vetor, uma cadeia de bits) que descreve completamente a solução. Por exemplo, o cromossomo `[0.8, 0.3, 12, 5.0]` poderia codificar uma IA com agressividade 0,8, cautela 0,3, distância de engajamento 12 e tempo de reação 5,0. Em muitas formulações clássicas, o cromossomo é uma **cadeia binária** (`10110100...`), mas pode ser qualquer estrutura adequada ao problema.

### Gene

Um **gene** é **cada unidade** do cromossomo — cada posição que carrega parte da informação da solução. No cromossomo `[0.8, 0.3, 12, 5.0]`, cada um dos quatro valores é um gene. Em um cromossomo binário, cada bit (ou cada grupo de bits com significado) é um gene. O **valor** que um gene assume é às vezes chamado de **alelo** (emprestando de novo o termo biológico). Os genes são as "peças" que a mutação altera e que o cruzamento embaralha entre pais.

### População

A **população** é o **conjunto de indivíduos** que o algoritmo mantém e faz evoluir **simultaneamente** a cada geração — tipicamente dezenas, centenas ou milhares de soluções candidatas coexistindo. Este é um ponto conceitual importante e distintivo: o Algoritmo Genético **não** trabalha com uma única solução que ele vai melhorando (como faz uma subida de encosta simples); ele mantém **muitas** soluções ao mesmo tempo, explorando **várias regiões** do espaço de busca em paralelo. Essa diversidade é a fonte de sua robustez — enquanto uma solução isolada pode ficar presa em um beco ruim, uma população diversa continua explorando alternativas.

### Aptidão (fitness)

A **aptidão** (ou *fitness*) é um **número** que mede **quão boa é uma solução** — o equivalente artificial ao "quão adaptado ao ambiente" de um organismo. Uma solução com aptidão alta é boa; com aptidão baixa, ruim. É a aptidão que determina quem tem mais chance de "sobreviver e se reproduzir": indivíduos mais aptos são preferidos na seleção, passando suas características adiante.

A aptidão é calculada por uma **função de aptidão** (*fitness function*), definida pelo projetista, que recebe um indivíduo (uma solução candidata) e devolve seu valor de qualidade. Para o problema de balanceamento, a função de aptidão poderia simular partidas com aquela configuração e medir "quão equilibrado foi o resultado". A função de aptidão é a peça mais crítica de todo o algoritmo, e a Seção 13.5 é inteiramente dedicada a ela.

### Seleção natural (artificial)

A **seleção natural**, no contexto do algoritmo, é o **princípio** de que indivíduos mais **aptos** têm **maior probabilidade** de serem escolhidos para reproduzir e, portanto, de transmitir seus genes à geração seguinte. É o motor que empurra a população em direção a soluções melhores: geração após geração, as boas características se acumulam e se combinam, e as ruins tendem a desaparecer. Note a palavra **probabilidade** — a seleção não é determinística (não pega **só** os melhores); ela **favorece** os melhores, mas dá alguma chance aos demais, preservando a diversidade que a exploração exige.

> **Curiosidade**
> A correspondência entre os termos biológicos e computacionais é notavelmente direta, mas há uma simplificação enorme: a "genética" dos Algoritmos Genéticos é uma caricatura da biologia real. Não há genes dominantes e recessivos, não há a bioquímica do DNA, não há a complexidade do desenvolvimento embrionário. O algoritmo captura apenas o **esqueleto lógico** da evolução — variação hereditária + seleção diferencial = adaptação —, que se revela suficiente para resolver problemas de otimização. É um belo exemplo de como uma abstração simplificada de um fenômeno natural pode se tornar uma ferramenta de engenharia poderosa.

[DIAGRAMA]
Título: Anatomia de uma população genética
Objetivo pedagógico: Fixar a relação hierárquica entre população, indivíduo, cromossomo, gene e aptidão em uma única imagem de referência.
Descrição detalhada: Uma caixa grande rotulada "POPULAÇÃO (geração t)" contendo quatro "indivíduos" empilhados. Cada indivíduo é uma fileira de células (o cromossomo), com cada célula rotulada como um "gene" e contendo um valor (ex.: indivíduo A = [0.8 | 0.3 | 12 | 5.0]). À direita de cada indivíduo, uma barra horizontal de tamanho proporcional rotulada "aptidão (fitness)", com um número (ex.: A → 87, B → 42, C → 95, D → 12), mostrando que C é o mais apto e D o menos apto. Uma legenda destacando: "cromossomo = a solução codificada", "gene = uma posição/parâmetro", "aptidão = qualidade da solução". Setas laterais indicando que os indivíduos de maior aptidão (C, A) terão mais chance de reproduzir.
Elementos obrigatórios: a população contendo vários indivíduos; cada indivíduo mostrado como cromossomo dividido em genes com valores; a barra de aptidão de cada um; a indicação de que maior aptidão = maior chance de reprodução.
[/DIAGRAMA]

---

## 13.3 Estrutura Geral do Algoritmo

Com o vocabulário no lugar, podemos montar o **ciclo completo** do Algoritmo Genético. Ele é, na sua essência, um laço que repete a evolução geração após geração. Vejamos cada etapa na ordem em que ocorre.

### O ciclo evolutivo, etapa por etapa

**1. Inicialização.** Cria-se uma **população inicial**, geralmente **aleatória** — um conjunto de soluções candidatas geradas ao acaso. No início, essas soluções são quase todas ruins, e isso não é problema: o algoritmo vai melhorá-las. A aleatoriedade inicial garante **diversidade**, espalhando a busca por muitas regiões do espaço.

**2. Avaliação.** Calcula-se a **aptidão** de **cada indivíduo** da população, aplicando a função de aptidão. Ao final deste passo, sabemos quão boa é cada solução — quem são os candidatos promissores e quem são os fracos.

**3. Seleção.** Escolhem-se, com base na aptidão, os indivíduos que servirão de **pais** da próxima geração. Os mais aptos têm mais chance de serem escolhidos (seleção natural artificial), mas a escolha é probabilística para preservar diversidade.

**4. Reprodução (cruzamento).** Os pais selecionados são **combinados** para gerar **descendentes** (filhos). O **cruzamento** (*crossover*) mistura os genes de dois pais, produzindo filhos que herdam características de ambos — a esperança é que, combinando partes boas de pais bons, surjam filhos ainda melhores.

**5. Mutação.** Nos descendentes, com **pequena probabilidade**, alteram-se aleatoriamente alguns genes. A mutação introduz **novidade** — variações que não estavam em nenhum dos pais —, essencial para explorar o espaço e não estagnar.

**6. Elitismo (opcional, mas comum).** Preservam-se, intactas, as **melhores** soluções da geração atual, copiando-as diretamente para a próxima. Isso garante que o algoritmo **nunca perca** a melhor solução já encontrada.

**7. Nova geração.** Os descendentes (mais os "eleitos" pelo elitismo) formam a **nova população**, que **substitui** a anterior. O algoritmo volta ao passo 2 (avaliação) com essa nova geração.

**8. Critério de parada.** O ciclo se repete até que uma **condição de parada** seja atingida — por exemplo: um número máximo de gerações; uma aptidão "boa o suficiente" alcançada; ou a população parar de melhorar (estagnação). Ao parar, retorna-se a **melhor solução encontrada** ao longo de toda a evolução.

O efeito acumulado desse ciclo é notável: geração após geração, a aptidão média da população **sobe**, e a melhor solução vai se refinando. De um punhado inicial de soluções aleatórias e ruins, emerge — sem que ninguém a tenha projetado — uma solução de alta qualidade.

> **Na Prática**
> Um erro conceitual frequente é imaginar que o Algoritmo Genético "converge" suavemente rumo ao ótimo como uma linha reta. Na realidade, a curva de evolução da melhor aptidão costuma ter **saltos** (quando um cruzamento afortunado descobre uma combinação nova e muito melhor) intercalados com **longos platôs** (gerações em que nada de novo aparece). Acompanhar o gráfico da melhor aptidão ao longo das gerações é a principal ferramenta de diagnóstico: platôs muito longos podem indicar convergência prematura (falta de diversidade) e a necessidade de mais mutação.

[DIAGRAMA]
Título: O ciclo completo do Algoritmo Genético
Objetivo pedagógico: Apresentar o laço evolutivo como um fluxo cíclico, deixando claro o ponto de entrada, o de decisão (parada) e o de saída.
Descrição detalhada: Um fluxograma cíclico. No topo, uma caixa de início "1. Inicialização (população aleatória)". Uma seta desce para "2. Avaliação (calcular aptidão de cada indivíduo)". Dessa caixa, uma seta vai a um losango de decisão "Critério de parada atingido?". Do losango, a saída "SIM" leva a uma caixa final "Retornar a melhor solução". A saída "NÃO" continua o ciclo, descendo para "3. Seleção (escolher pais por aptidão)" → "4. Cruzamento (combinar pais → filhos)" → "5. Mutação (alterar genes ao acaso)" → "6. Elitismo (preservar os melhores)" → "7. Nova geração substitui a anterior", e uma seta longa retorna dessa última caixa de volta para "2. Avaliação", fechando o laço. Destacar visualmente que os passos 3–6 (seleção, cruzamento, mutação, elitismo) são os "operadores genéticos".
Elementos obrigatórios: as oito etapas nomeadas; o laço fechado voltando à avaliação; o losango de critério de parada com as saídas SIM/NÃO; o destaque para o bloco de operadores genéticos.
[/DIAGRAMA]

---

## 13.4 Representação do Problema

Antes de qualquer operador, uma decisão de projeto governa o sucesso de tudo: **como codificar as soluções candidatas como cromossomos?**. A **representação** (também chamada de **codificação**) é a ponte entre o problema do mundo real e a maquinaria do algoritmo — e escolhê-la mal condena a busca, por melhores que sejam os operadores.

### Por que a representação importa

O Algoritmo Genético não "entende" o problema; ele manipula cromossomos cegamente, cruzando e mutando genes. Para que essa manipulação produza soluções válidas e melhoráveis, o cromossomo precisa **codificar a solução de tal forma que operações genéticas façam sentido**. Se cruzar dois cromossomos válidos frequentemente produzir filhos **inválidos** (soluções que nem sequer são possíveis no problema), o algoritmo desperdiça esforço, e a representação está mal escolhida.

### Formas comuns de representação

- **Representação binária.** O cromossomo é uma cadeia de bits (`1011010...`). É a codificação clássica de Holland, geral e simples, adequada quando a solução pode ser vista como um conjunto de decisões sim/não ou como números codificados em binário. Cruzamento e mutação sobre bits são triviais.

- **Representação por valores reais.** Cada gene é um número real, ideal quando a solução é um conjunto de **parâmetros contínuos** (agressividade 0,8; velocidade 3,2...). É a mais natural para o **ajuste de parâmetros de IA**.

- **Representação por permutação.** O cromossomo é uma **ordenação** de elementos, usada em problemas onde a solução é uma **sequência** — como a ordem de visita de pontos (o clássico "problema do caixeiro-viajante") ou a ordem de salas em uma dungeon. Aqui, operadores especiais são necessários para que o cruzamento não gere sequências inválidas (com elementos repetidos ou faltando).

- **Representações estruturadas / em árvore.** Para problemas onde a solução é uma **estrutura** (uma regra, um programa, uma árvore de comportamento), o cromossomo pode ser uma árvore — território da **programação genética**, um parente do algoritmo genético.

### Exemplos simples aplicados a jogos

- **Ajuste da IA de um inimigo.** Cromossomo de valores reais: `[agressividade, precisão, tempo_reação, distância_engajamento, taxa_recuo]`. Cada gene é um parâmetro; a aptidão mede quão desafiador e justo o inimigo resultante é.

- **Geração de uma fase em grade.** Cromossomo binário ou inteiro codificando a presença/tipo de elemento em cada célula da grade (0 = vazio, 1 = parede, 2 = inimigo, 3 = item...). A aptidão avalia jogabilidade (existe caminho até a saída? a dificuldade é adequada?).

- **Ordem de ondas em um jogo de defesa.** Cromossomo de permutação definindo a sequência de tipos de inimigos; a aptidão mede a curva de dificuldade resultante.

> **Erro Comum**
> Escolher uma representação em que **a maioria dos cromossomos possíveis é inválida**. Por exemplo, codificar uma fase de labirinto como bits soltos por célula frequentemente gera labirintos sem caminho da entrada à saída — soluções que a função de aptidão precisa punir ou consertar, desperdiçando busca. Boas representações fazem com que **operações genéticas sobre soluções válidas tendam a produzir soluções válidas** (ou usam operadores/reparos que garantem validade). Pensar na representação **antes** dos operadores é meio caminho andado.

---

## 13.5 Função Fitness

Se há uma única peça que decide o destino de um Algoritmo Genético, é a **função de aptidão**. Ela é a única forma pela qual o problema "conversa" com o algoritmo — o algoritmo não sabe nada sobre o jogo, sobre balanceamento ou sobre diversão; ele sabe apenas os **números** que a função de aptidão lhe dá. Uma função de aptidão bem projetada guia a evolução ao sucesso; uma mal projetada leva, com toda a competência do algoritmo, ao lugar errado.

### O objetivo da função de aptidão

A função de aptidão traduz "o que queremos" em um **número a ser maximizado** (ou minimizado). Ela recebe um indivíduo — uma solução candidata — e devolve sua qualidade. Todo o resto do algoritmo (seleção, cruzamento, mutação) existe apenas para **subir** esse número ao longo das gerações. Portanto, **a função de aptidão define, na prática, o que o algoritmo vai buscar**. Se ela mede a coisa errada, o algoritmo encontrará, com maestria, a solução errada.

Note o **paralelo profundo com a recompensa do Capítulo 12**: assim como a recompensa é a única forma de dizer ao agente de RL o que se deseja, a função de aptidão é a única forma de dizer ao Algoritmo Genético o que se deseja. Ambas sofrem do mesmo perigo — especificar mal o objetivo produz otimização perfeita do alvo errado.

### Critérios de avaliação

Projetar a função de aptidão significa decidir **quais critérios** compõem a qualidade e **como pesá-los**. Frequentemente, a qualidade é **multiobjetivo** — uma boa fase deve ser jogável **e** interessante **e** difícil na medida certa. Isso leva a funções de aptidão que **combinam** vários critérios, tipicamente por uma **soma ponderada** (cada critério multiplicado por um peso que reflete sua importância) — exatamente como as funções de avaliação do Minimax (Capítulo 11) e os mapas de influência (Capítulo 10) combinavam fatores. Definir esses pesos é, ele próprio, um problema de ajuste delicado.

### Influência na convergência

A **forma** da função de aptidão afeta drasticamente a **velocidade e a qualidade** da convergência. Uma função que dá **gradações finas** de qualidade (soluções ligeiramente melhores recebem aptidão ligeiramente maior) fornece um "caminho" claro para a evolução subir — a busca tem para onde ir. Já uma função **binária** ou **plana** (que dá a mesma aptidão a quase todas as soluções e só distingue as pouquíssimas perfeitas) deixa o algoritmo **cego**: sem gradação, a seleção não tem como preferir os "quase bons", e a busca vira um passeio aleatório. O ideal é uma função de aptidão que **recompense o progresso parcial**, guiando suavemente a população rumo às boas soluções.

### Problemas de funções mal projetadas

- **Aptidão enganosa (*deceptive*).** A função aponta para uma direção que parece boa mas leva a um beco — soluções de aptidão média que **não** são passos rumo às ótimas. O algoritmo é atraído para o lugar errado.
- **Objetivo mal especificado.** A função mede um substituto conveniente em vez do que realmente se quer, e o algoritmo **explora a brecha** — o análogo do *reward hacking* do RL. Se você recompensa "número de inimigos na fase" achando que isso mede dificuldade, o algoritmo entope a fase de inimigos de forma injogável.
- **Custo de avaliação proibitivo.** Se avaliar um único indivíduo exige simular muitas partidas longas, e a população tem centenas de indivíduos por muitas gerações, o custo total pode ser inviável. A **eficiência** da função de aptidão é uma preocupação real, pois ela é executada milhares ou milhões de vezes.

> **Atenção**
> A qualidade da função de aptidão é, quase sempre, o **fator determinante** do sucesso ou fracasso de um Algoritmo Genético — mais do que os detalhes dos operadores ou o tamanho da população. Equipes iniciantes gastam tempo ajustando taxas de mutação e métodos de seleção quando o verdadeiro problema está numa função de aptidão que mede a coisa errada ou não dá gradação suficiente. Ao depurar um GA que "não funciona", **suspeite primeiro da função de aptidão**.

---

## 13.6 Operadores Genéticos

Os **operadores genéticos** são as operações que transformam uma geração na seguinte: **seleção**, **cruzamento**, **mutação** e **elitismo**. Cada um tem um papel distinto no equilíbrio entre **explorar** o espaço (buscar novidade) e **explotar** o que já é bom (refinar) — o mesmo dilema exploração/explotação que encontramos no RL, aqui em outra roupagem.

### Seleção

A **seleção** escolhe quais indivíduos serão pais da próxima geração, favorecendo os mais aptos. Existem várias estratégias, cada uma com um perfil próprio:

- **Seleção por roleta (proporcional à aptidão).** Cada indivíduo recebe uma "fatia" de uma roleta proporcional à sua aptidão; gira-se a roleta para escolher cada pai. Indivíduos mais aptos ocupam fatias maiores e são escolhidos com mais frequência. **Vantagem:** simples e intuitiva; dá chance a todos. **Limitação:** se um indivíduo tem aptidão muito superior aos demais, ele domina a roleta e a população perde diversidade rapidamente (convergência prematura); e não funciona bem se as aptidões forem todas parecidas (a roleta fica quase uniforme, perdendo pressão seletiva).

- **Seleção por torneio.** Sorteiam-se *k* indivíduos ao acaso e escolhe-se o mais apto entre eles como pai; repete-se para cada pai necessário. **Vantagem:** simples, eficiente, e a "pressão seletiva" é facilmente ajustável pelo tamanho do torneio *k* (torneios maiores favorecem mais os fortes); não sofre com escalas de aptidão. É a estratégia **mais usada na prática**. **Limitação:** com *k* muito grande, os fracos nunca vencem e a diversidade despenca.

- **Seleção por ranqueamento (rank).** Ordenam-se os indivíduos por aptidão e a probabilidade de seleção depende da **posição no ranking**, não do valor absoluto da aptidão. **Vantagem:** evita que um "super-indivíduo" domine (só importa a ordem, não o quanto ele é melhor) e mantém pressão seletiva mesmo com aptidões parecidas. **Limitação:** ignora a **magnitude** das diferenças de aptidão, o que pode desacelerar a convergência.

O ponto comum a todas: a seleção não deve ser **gananciosa demais** (pegar só os melhores mata a diversidade e trava a busca em soluções medianas) nem **frouxa demais** (não favorecer os bons faz a busca virar acaso). Calibrar a **pressão seletiva** é a arte da seleção.

### Crossover

O **cruzamento** (*crossover*) combina os genes de **dois pais** para gerar um ou mais **filhos**, na esperança de que juntar partes boas de bons pais produza filhos melhores. É o operador que faz a **explotação** — recombinar o material genético promissor já existente. As principais variantes:

- **Crossover de um ponto.** Escolhe-se um **ponto de corte** aleatório ao longo do cromossomo; o filho herda os genes de um pai **antes** do corte e do outro pai **depois** do corte. Simples e clássico. Exemplo: pais `AAAA|AAAA` e `BBBB|BBBB` com corte no meio geram filhos `AAAABBBB` e `BBBBAAAA`.

- **Crossover de múltiplos pontos.** Vários pontos de corte, alternando-se os trechos herdados de cada pai. Mistura mais os genes, quebrando dependências entre posições distantes no cromossomo.

- **Crossover uniforme.** Para **cada gene**, sorteia-se independentemente de qual pai o filho o herdará (como jogar uma moeda por posição). Produz a mistura mais completa, sem preservar blocos contíguos de genes. Útil quando não há razão para manter posições vizinhas juntas.

Qual usar? Depende do problema e, sobretudo, da **representação**. Se genes vizinhos no cromossomo formam "blocos" que fazem sentido juntos (esquemas), o crossover de um ou poucos pontos os preserva melhor; se os genes são independentes, o uniforme mistura mais e pode explorar melhor. Em representações por permutação, nenhum desses serve diretamente (gerariam sequências inválidas), exigindo operadores especializados.

> **Curiosidade**
> O poder do crossover é o cerne da chamada "hipótese dos blocos construtivos" (*building blocks*), de Holland: a ideia de que o algoritmo genético funciona porque **combina pequenos blocos de genes bons** (esquemas de alta aptidão) de pais diferentes em filhos que os reúnem. É o análogo computacional da ideia de que a reprodução sexual acelera a evolução ao recombinar boas adaptações que surgiram em linhagens separadas — algo que a mera mutação, sozinha, levaria muito mais tempo para juntar.

### Mutação

A **mutação** altera **aleatoriamente** um ou poucos genes de um filho, com **pequena probabilidade** por gene (a **taxa de mutação**). É o operador da **exploração** — a fonte de **novidade genética**, capaz de introduzir valores que **não existiam em nenhum pai**.

- **Objetivo.** Manter a **diversidade** da população e permitir escapar de becos. Sem mutação, a população só pode recombinar o material genético inicial; se uma característica boa nunca esteve presente e nunca surge por acaso, ela **jamais** aparecerá. A mutação é o que garante que qualquer ponto do espaço de busca permaneça, em princípio, alcançável.

- **Taxa de mutação.** Um parâmetro crítico. **Baixa demais** (perto de zero): a população estagna, perde diversidade e converge prematuramente para uma solução mediana. **Alta demais:** o algoritmo vira uma busca praticamente **aleatória**, destruindo as boas soluções mais rápido do que a seleção consegue preservá-las — perde-se o "hereditário" da evolução. Valores típicos são **pequenos** (por exemplo, alterar cada gene com probabilidade de ~0,1% a ~5%), justamente para que a mutação **complemente** o crossover sem sabotá-lo.

- **Importância contra a convergência prematura.** A **convergência prematura** é o fracasso mais comum dos GAs: a população inteira fica parecida (todos os indivíduos quase idênticos), a diversidade morre, o crossover não gera nada novo (cruzar iguais dá iguais) e a evolução **congela** num ótimo local medíocre. A mutação é a principal defesa contra isso — ela reinjeta variação e mantém a busca viva.

### Elitismo

O **elitismo** copia, sem alterações, as **melhores soluções** da geração atual diretamente para a próxima. É uma salvaguarda simples e poderosa.

- **Preservação das melhores soluções.** Sem elitismo, é perfeitamente possível que a **melhor** solução de uma geração seja **destruída** na seguinte — o crossover e a mutação são aleatórios e podem não reproduzir o melhor indivíduo, ou podem estragá-lo. O elitismo **garante** que a melhor aptidão encontrada **nunca regrida**: a qualidade da melhor solução só pode subir ou ficar igual ao longo das gerações, nunca piorar.

- **Impacto na convergência.** O elitismo **acelera** a convergência e a torna **monótona** (a melhor aptidão é uma curva que só sobe). Mas há um preço: elitismo **excessivo** (preservar muitos indivíduos) reduz a diversidade e pode **acelerar demais** a convergência prematura, pois os "eleitos" dominam a reprodução. A prática comum é preservar **poucos** dos melhores (por exemplo, 1 a 5% da população) — o suficiente para não perder o topo, sem sufocar a exploração.

> **Boa Prática**
> Pense nos quatro operadores como um sistema de **equilíbrio entre exploração e explotação**. Crossover e elitismo **explotam** (refinam e preservam o bom já encontrado); mutação **explora** (busca o novo); seleção **regula a pressão** entre os dois. Um GA que converge cedo demais e trava (convergência prematura) geralmente precisa de **mais exploração**: aumentar a mutação, reduzir o elitismo ou a pressão seletiva. Um GA que "não converge" e vagueia geralmente precisa de **mais explotação**: aumentar a pressão seletiva ou o elitismo. Ajustar um GA é, no fundo, calibrar essa balança.

[DIAGRAMA]
Título: Os operadores genéticos em ação — seleção, crossover, mutação e elitismo
Objetivo pedagógico: Ilustrar visualmente como dois pais selecionados geram filhos por crossover e mutação, e o papel do elitismo.
Descrição detalhada: Quatro painéis em sequência. Painel 1 ("Seleção"): uma população de vários cromossomos com barras de aptidão; dois deles (de alta aptidão) destacados e marcados como "Pai 1" e "Pai 2". Painel 2 ("Crossover de um ponto"): os dois pais desenhados como fileiras de genes coloridos, com uma linha vertical tracejada marcando o "ponto de corte"; abaixo, dois filhos resultantes, cada um com a porção esquerda de um pai e a direita do outro (cores trocadas no ponto de corte). Painel 3 ("Mutação"): um dos filhos com um único gene destacado (piscando/em cor de alerta) que teve seu valor alterado ao acaso, com a legenda "taxa de mutação pequena". Painel 4 ("Elitismo"): a seta mostrando o melhor indivíduo da geração anterior sendo copiado intacto direto para a nova população, com a legenda "o melhor nunca se perde". Uma seta geral no rodapé indicando "geração t → geração t+1".
Elementos obrigatórios: os dois pais selecionados por aptidão; o ponto de corte do crossover e os filhos resultantes; a mutação de um gene em um filho; a cópia do elite para a nova geração; a indicação da transição entre gerações.
[/DIAGRAMA]

---

## 13.7 Aplicações em Jogos

Vejamos como os Algoritmos Genéticos — e a otimização evolutiva em geral — aparecem em jogos, sempre contextualizando o exemplo. Como no capítulo anterior, distinguiremos, na Seção 13.10, fatos documentados de análises técnicas.

### Balanceamento automático

- **O problema.** Um jogo com muitas unidades, cartas, armas ou parâmetros econômicos precisa que **nenhuma estratégia domine** — mas o espaço de configurações é vasto demais para o designer testar à mão.
- **A abordagem evolutiva.** Cada indivíduo é um **conjunto de valores** (dano, custo, alcance, cadência...); a aptidão mede o **equilíbrio** resultante (por exemplo, simulando muitas partidas entre IAs e medindo se as taxas de vitória das diferentes estratégias ficam próximas). O GA evolui configurações cada vez mais balanceadas.
- **O valor.** Automatiza um trabalho tedioso e sugere pontos de partida que o designer refina — o GA como **assistente de balanceamento**, não como substituto do julgamento humano.

### Geração procedural de conteúdo (PCG)

- **O problema.** Gerar fases, mapas, dungeons, missões ou até armas que satisfaçam critérios de qualidade (jogável, interessante, difícil na medida certa).
- **A abordagem evolutiva.** Conhecida como **"PCG baseada em busca"** (*search-based PCG*): cada indivíduo é **um conteúdo** (uma fase); a aptidão mede sua qualidade segundo critérios formalizados (existe caminho? há variedade? a dificuldade cresce bem?). O GA evolui conteúdos que maximizam esses critérios.
- **O valor.** Produz conteúdo virtualmente infinito e ajustável, e é uma das aplicações **mais estudadas** de GAs em jogos na academia.

### Evolução de comportamentos e estratégias

- **O problema.** Fazer emergir comportamentos de NPCs ou estratégias que não sabemos programar à mão.
- **A abordagem evolutiva.** Cada indivíduo codifica um **controlador** (um conjunto de regras, os parâmetros de uma FSM, ou os pesos de uma rede neural — técnica chamada **neuroevolução**); a aptidão mede o desempenho do comportamento em jogo. O GA evolui comportamentos cada vez mais competentes.
- **O valor.** Descobre estratégias não óbvias e serve tanto para criar oponentes quanto para **testar** o jogo (agentes evoluídos que encontram exploits).

### Ajuste de parâmetros de IA

- **O problema.** Uma behavior tree, uma FSM ou uma função de avaliação (Minimax) têm muitos parâmetros numéricos cujo ajuste manual é trabalhoso e frágil.
- **A abordagem evolutiva.** Cada indivíduo é um **conjunto de parâmetros**; a aptidão mede o desempenho da IA com aqueles valores. O GA **calibra** os parâmetros automaticamente.
- **O valor.** Note a sinergia com as Partes anteriores: o GA **não substitui** a FSM ou a behavior tree — ele **afina** os números delas. As técnicas convivem: a arquitetura é escrita à mão (determinística, controlável), e o GA otimiza seus parâmetros. É frequentemente o uso **mais realista** de GAs em produção.

### Design procedural (níveis, criaturas, itens)

- **O problema.** Gerar automaticamente ativos de design (formas de criaturas, estatísticas de itens, disposições de nível) que atendam a restrições estéticas e funcionais.
- **A abordagem evolutiva.** Cada indivíduo é um **artefato de design**; a aptidão codifica as restrições e objetivos. Uma variante interessante é a **evolução interativa**, em que o **humano** atua como função de aptidão, escolhendo a cada geração os artefatos de que mais gosta — usada para gerar criaturas, texturas e conteúdo estético onde "qualidade" é subjetiva.

> **Na Indústria**
> A aplicação mais famosa de "evolução" em um jogo comercial não é bem um GA de otimização, mas ilustra o espírito: em ***Spore*** (Maxx/EA, 2008), as criaturas criadas pelos jogadores povoam o universo de outros jogadores, e o jogo tem forte tema evolutivo. Já a evolução como **ferramenta de otimização** (balanceamento, PCG, ajuste) tende a ser usada nos **bastidores do desenvolvimento**, não como recurso anunciado ao jogador — o que a torna menos visível, porém real, sobretudo em estúdios com equipes de pesquisa e nas ferramentas internas.

---

## 13.8 Ferramentas

Seguindo a filosofia da apostila, esta seção é **contextualização**, não tutorial. Diferentemente do RL (que tem o ML-Agents como ferramenta oficial destacada na Unity), os Algoritmos Genéticos são conceitualmente **simples de implementar** — o ciclo da Seção 13.3 cabe em poucas dezenas de linhas —, então o cenário de ferramentas é mais aberto.

### Implementação própria em C#

Por serem simples, GAs são frequentemente **implementados à mão** no próprio projeto. O esqueleto é direto: uma classe que representa o indivíduo (o cromossomo), uma função de aptidão específica do problema, e o laço de gerações aplicando seleção, crossover, mutação e elitismo. A parte que exige juízo **não** é o código do algoritmo (que é padrão), e sim as **decisões de projeto** — representação, função de aptidão, operadores e parâmetros —, que dependem inteiramente de compreender os fundamentos deste capítulo. Para fixar apenas a estrutura (não como tutorial), o esqueleto conceitual de um laço genético em C# seria:

```csharp
// Contextualização — não é um tutorial de implementação.
Populacao pop = InicializarAleatoria(tamanho);
while (!CriterioDeParada(pop))
{
    AvaliarAptidao(pop);                 // função de aptidão do problema
    var elite = SelecionarMelhores(pop); // elitismo
    var nova  = new Populacao(elite);
    while (nova.Count < tamanho)
    {
        var (pai1, pai2) = Selecionar(pop);   // torneio, roleta...
        var filho = Crossover(pai1, pai2);
        Mutar(filho, taxaMutacao);
        nova.Add(filho);
    }
    pop = nova;
}
return MelhorIndividuo(pop);
```

O ponto pedagógico é o mesmo do Capítulo 11: **não há mágica de ferramenta**. O algoritmo genético é este laço; dominá-lo é dominar as decisões que ele encapsula.

### Bibliotecas e Asset Store

Para quem não quer reimplementar, existem **bibliotecas de código aberto** de algoritmos genéticos em C# e .NET (por exemplo, a **GeneticSharp**, uma biblioteca popular e madura de GA para .NET, utilizável na Unity), além de pacotes na **Asset Store** que oferecem GAs, neuroevolução e otimização prontos para integrar. Essas bibliotecas fornecem os operadores clássicos e a infraestrutura do laço, deixando ao desenvolvedor apenas a definição da representação e da função de aptidão.

### Integração com a Unity e neuroevolução

Os GAs integram-se naturalmente à Unity porque a própria **cena do jogo** pode servir de "ambiente de avaliação": simula-se cada indivíduo (uma configuração de IA, uma fase gerada) dentro da Unity e mede-se sua aptidão pelo comportamento observado. Uma combinação notável é a **neuroevolução** — usar um GA para evoluir os **pesos de redes neurais** que controlam agentes —, popularizada por frameworks como o **NEAT** (*NeuroEvolution of Augmenting Topologies*), que tem implementações em C#/Unity. É um ponto de contato fascinante entre este capítulo e o anterior: a neuroevolução usa a **otimização evolutiva** (Cap. 13) para produzir os **controladores neurais** que, no RL (Cap. 12), seriam treinados por gradiente — dois caminhos diferentes para um fim parecido.

> **Atenção**
> Como no capítulo anterior, nenhuma biblioteca substitui a compreensão conceitual. A GeneticSharp ou o NEAT automatizam a **mecânica** (os operadores, o laço), mas as decisões que **determinam o sucesso** — como codificar a solução e, sobretudo, **como medir a aptidão** — continuam sendo do projetista, e dependem inteiramente dos fundamentos das Seções 13.4 a 13.6.

---

## 13.9 Vantagens e Limitações

Avaliação crítica, comparando — quando apropriado — com métodos tradicionais de otimização e com a Aprendizagem por Reforço.

### Vantagens

- **Generalidade.** GAs aplicam-se a quase qualquer problema em que se consiga (a) representar uma solução como cromossomo e (b) medir sua qualidade com uma função de aptidão. Não exigem que o problema seja contínuo, diferenciável ou bem-comportado — funcionam onde métodos clássicos de otimização (como os baseados em gradiente) **falham**.
- **Exploração global e robustez a ótimos locais.** Por manter uma **população diversa** explorando várias regiões ao mesmo tempo, o GA tem boa chance de **escapar de ótimos locais** que aprisionariam uma busca local simples (como a subida de encosta). Ele busca globalmente, não só na vizinhança de um ponto.
- **Não exige conhecimento do problema.** O GA trata a função de aptidão como **caixa-preta** — não precisa de fórmula, derivada nem modelo interno; basta poder **avaliar** soluções. Isso o torna aplicável a problemas onde só se sabe simular, não resolver analiticamente.
- **Paralelizável.** A avaliação dos indivíduos é independente e pode ser distribuída, aproveitando bem múltiplos núcleos ou máquinas.
- **Produz soluções criativas.** Por não seguir a intuição humana, frequentemente encontra soluções **inesperadas e não óbvias** — um valor real em design e PCG.

### Limitações

- **Não garante o ótimo.** É uma **heurística**: entrega soluções boas, mas **sem garantia** de que sejam as melhores possíveis. Onde existe um algoritmo exato eficiente (como A\* para caminho mínimo, ou métodos analíticos para problemas convexos), esse algoritmo é **preferível** — usar GA nesses casos é trocar uma garantia por uma aproximação.
- **Custo computacional.** Avaliar centenas de indivíduos por muitas gerações pode ser **caro**, sobretudo quando cada avaliação exige simular partidas. O custo total = (tamanho da população) × (nº de gerações) × (custo de uma avaliação), e esse produto cresce rápido.
- **Dependência crítica da função de aptidão.** Como discutido, uma função de aptidão mal projetada arruína tudo — e projetá-la bem é difícil. Esta é a limitação prática mais séria e a mais subestimada.
- **Muitos parâmetros a ajustar.** Tamanho da população, taxa de mutação, tipo de seleção e crossover, grau de elitismo, critério de parada — todos afetam o resultado e exigem calibração (às vezes, ironicamente, resolvida com **outra** busca de parâmetros).
- **Convergência prematura.** O risco constante de a população perder diversidade e travar num ótimo local medíocre, exigindo vigilância sobre mutação e pressão seletiva.
- **Velocidade de convergência imprevisível.** Pode encontrar boa solução rápido ou levar gerações intermináveis, sem garantia de tempo.

### Comparação com métodos tradicionais e com RL

Frente a **métodos tradicionais de otimização**, o GA ganha em **generalidade e robustez** (funciona onde eles não se aplicam) mas perde em **garantia e eficiência** (onde eles se aplicam, são melhores). A regra é a mesma do resto da apostila: **use a ferramenta mais específica que resolve o problema**; o GA é a opção **genérica** para quando as específicas não servem.

Frente à **Aprendizagem por Reforço**, a diferença é de **natureza do problema**, não só de desempenho — e é o cerne da Parte VI, que a próxima seção e o encerramento aprofundam: RL aprende um **comportamento** por interação ao longo do tempo (um agente que age); o GA **otimiza** uma solução por busca populacional (uma configuração que se avalia). Curiosamente, os dois podem atacar problemas parecidos por caminhos opostos — a neuroevolução, por exemplo, treina redes de controle por GA, ali onde o RL as treinaria por gradiente.

> **Erro Comum**
> Usar um Algoritmo Genético onde um método exato e eficiente existe. GA é sedutor por sua elegância e generalidade, mas se o problema é "achar o caminho mais curto" (use A\*), "resolver um sistema linear" (use álgebra) ou qualquer problema com solução analítica ou exata rápida, o GA é a **ferramenta errada** — mais lento e sem garantia. Reserve-o para os problemas genuinamente difíceis, de espaço enorme e sem método exato viável, que motivaram o capítulo.

---

## 13.10 Estudos de Caso

Casos reais, sempre distinguindo **fato documentado** de **análise técnica fundamentada**.

### PCG baseada em busca na pesquisa acadêmica — fato documentado

A geração procedural de conteúdo por métodos evolutivos é um campo de pesquisa **bem documentado**. Trabalhos acadêmicos (reunidos, por exemplo, no livro *Procedural Content Generation in Games*, de Shaker, Togelius e Nelson, 2016) mostram GAs evoluindo fases de plataforma, mapas de estratégia, dungeons, pistas de corrida, armas e regras. São resultados **verificáveis e reproduzíveis**, com funções de aptidão publicadas — a base mais sólida para afirmar que GAs geram conteúdo de jogo de qualidade.

### Competições de IA de jogos por evolução — fato documentado

Diversas competições acadêmicas (como as ligadas à *IEEE Conference on Games*) usaram evolução e neuroevolução para criar agentes de jogos como *Super Mario Bros.* (a *Mario AI Competition*), corridas (*TORCS*) e o *General Video Game AI*. Há documentação de agentes competitivos **evoluídos** por GA/neuroevolução, incluindo o uso de **NEAT** para evoluir controladores neurais que jogam. São exemplos concretos e publicados de comportamento de jogo produzido por otimização evolutiva.

### Balanceamento e ajuste de parâmetros na indústria — análise técnica fundamentada

Aqui a documentação é mais escassa e entramos em **inferência cautelosa**. É **plausível e frequentemente relatado** (em coletâneas como *Game AI Pro* e em palestras da GDC) que estúdios usem otimização — incluindo métodos evolutivos — nos **bastidores** para balanceamento e ajuste de parâmetros. Contudo, salvo quando um estúdio o documenta explicitamente, atribuir "este jogo usou GA para balancear" a um título específico é **hipótese**, não fato. A prática existe e é razoável; a atribuição a jogos comerciais fechados exige a mesma cautela que exercemos com o RL.

### Neuroevolução em jogos experimentais — fato documentado

Jogos e demos **experimentais/acadêmicos** usaram neuroevolução de forma central e documentada — o exemplo mais citado é ***NERO*** (*NeuroEvolving Robotic Operatives*), um jogo de pesquisa em que o **jogador treina**, por evolução (via NEAT), um exército de agentes neurais que aprendem táticas ao longo da partida. É um caso raro e documentado de evolução como **mecânica de jogo**, não apenas ferramenta de bastidor.

> **Atenção**
> Repare no padrão que se repete desde o Capítulo 12: os casos **mais sólidos** de RL e de GAs em jogos vêm da **pesquisa acadêmica** e de **ferramentas de desenvolvimento**, não de recursos anunciados em grandes lançamentos comerciais. Isso não diminui as técnicas — apenas situa corretamente **onde** elas efetivamente brilham hoje. Manter essa honestidade sobre a fronteira entre pesquisa e produção é uma competência central de um profissional de IA de jogos, e é exatamente o que a Parte VII (engenharia reversa) levará ao limite.

---

## 13.11 Encerramento do Capítulo

### Resumo

Este capítulo apresentou as **heurísticas de otimização** e, em particular, os **Algoritmos Genéticos**, a técnica que resolve problemas de **busca em espaços gigantescos** imitando a **evolução biológica**. Partimos do **problema** — a **explosão combinatória** que torna a busca exaustiva impossível e a inexistência, para muitos problemas, de algoritmo exato eficiente — e da ideia de **otimização heurística**: buscar uma solução **boa o suficiente** em tempo aceitável, em vez do ótimo inatingível. Demarcamos, desde o início, a distinção que atravessa a Parte VI: **otimizar** (buscar a melhor configuração num espaço de candidatos) é diferente de **aprender a agir** (ajustar um comportamento por interação), separando o GA da Aprendizagem por Reforço do Capítulo 12.

Construímos os **fundamentos** com a terminologia evolutiva no sentido computacional: **indivíduo** (uma solução candidata), **cromossomo** (sua codificação), **gene** (cada unidade do cromossomo), **população** (o conjunto de candidatos que evolui em paralelo), **aptidão/fitness** (o número que mede a qualidade) e **seleção natural** (a preferência probabilística pelos mais aptos). Detalhamos o **ciclo completo** do algoritmo — inicialização, avaliação, seleção, cruzamento, mutação, elitismo, nova geração e critério de parada — e a importância decisiva da **representação** (codificar a solução de modo que operações genéticas façam sentido e gerem soluções válidas) e da **função de aptidão** (a única forma de dizer ao algoritmo o que se quer, com o mesmo perigo de má especificação que a recompensa do RL).

Dissecamos os **operadores genéticos**: **seleção** (roleta, torneio, ranqueamento — calibrando a pressão seletiva), **crossover** (um ponto, múltiplos pontos, uniforme — a explotação que recombina bons blocos), **mutação** (a exploração que injeta novidade e evita a **convergência prematura**, governada pela taxa de mutação) e **elitismo** (preservar os melhores, garantindo que a melhor aptidão nunca regrida). Aterrissamos em **aplicações** (balanceamento automático, geração procedural, evolução de comportamentos, ajuste de parâmetros, design procedural), **ferramentas** (implementação própria em C#, bibliotecas como GeneticSharp, neuroevolução com NEAT), uma discussão crítica de **vantagens e limitações** (generalidade e robustez a ótimos locais versus falta de garantia, custo e dependência da função de aptidão) e **estudos de caso** documentados (PCG acadêmica, competições de IA, NERO), sempre separando fato de análise e reforçando **quando usar** — e quando **não** usar — um GA frente a métodos exatos e ao RL.

### Questões de Revisão

1. O que é a **explosão combinatória** e por que ela inviabiliza a busca exaustiva? Relacione com o problema já visto no Minimax (Capítulo 11).
2. O que significa **otimização heurística**? Por que "bom o suficiente" é, muitas vezes, a meta realista? Como o sentido de "heurística" aqui se conecta ao do A\* (Capítulo 8)?
3. Defina, no sentido computacional: indivíduo, cromossomo, gene, população e aptidão. Dê um exemplo de cada para o problema de ajustar os parâmetros de uma IA inimiga.
4. Descreva as **oito etapas** do ciclo de um Algoritmo Genético, na ordem. Qual delas é o "momento do aprendizado" análogo à atualização no Q-Learning?
5. Por que a **representação** do problema é uma decisão crítica? O que caracteriza uma representação mal escolhida?
6. Qual o papel da **função de aptidão**? Explique o paralelo entre função de aptidão e **recompensa** do RL, incluindo o perigo comum a ambas.
7. Compare **seleção por roleta**, **por torneio** e **por ranqueamento**, com uma vantagem e uma limitação de cada.
8. Explique os três tipos de **crossover** (um ponto, múltiplos pontos, uniforme). Como a escolha se relaciona com a representação?
9. Qual o objetivo da **mutação**? O que acontece com taxas de mutação **baixas demais** e **altas demais**? O que é **convergência prematura**?
10. O que é **elitismo** e por que ele garante que a melhor aptidão nunca piore? Qual o risco de elitismo excessivo?

### Exercícios Conceituais

1. **Projeto de representação e aptidão.** Você quer usar um GA para gerar fases de um jogo de plataforma em grade. Proponha uma **representação** (cromossomo) e uma **função de aptidão** com pelo menos três critérios ponderados. Explique como evitar que o GA gere fases invencíveis.
2. **Diagnóstico de convergência prematura.** Um GA de balanceamento estagna após poucas gerações, com toda a população quase idêntica e uma aptidão medíocre. Liste três causas prováveis e, para cada uma, uma correção (relacione com mutação, elitismo e pressão seletiva).
3. **Aptidão enganosa.** Dê um exemplo, em um jogo à sua escolha, de uma função de aptidão que **parece** medir o que se quer mas leva o GA a uma solução indesejada (análogo ao *reward hacking*). Proponha uma correção.
4. **Traçado de crossover.** Dados os pais `[A A A A A A]` e `[B B B B B B]`, mostre os filhos resultantes de: (a) crossover de um ponto com corte após a 2ª posição; (b) crossover de dois pontos com cortes após a 2ª e a 4ª posições; (c) crossover uniforme com a sequência de sorteios "pai1, pai2, pai2, pai1, pai1, pai2".
5. **GA ou método exato?** Para cada problema, decida se usaria um GA ou um método exato/específico, e justifique: (a) caminho mais curto entre dois pontos de um mapa; (b) balancear 30 parâmetros de unidades para que nenhuma domine; (c) ordenar uma lista de inimigos por vida; (d) gerar dungeons variadas e jogáveis.
6. **Otimização × aprendizado.** Explique, com um exemplo concreto de jogo, por que "evoluir os parâmetros de uma behavior tree com um GA" é **otimização** e "um agente aprender a jogar por Q-Learning" é **aprendizado**. Poderiam os dois atacar o **mesmo** problema? Como?

### Leituras Complementares

- **Millington, I.** *AI for Games*. 3ª ed. — Capítulo sobre técnicas de aprendizado e otimização em jogos: algoritmos genéticos, neuroevolução e a perspectiva prática de uso na indústria.
- **Bourg, D. M.; Seemann, G.** *AI for Game Developers*. — Capítulo dedicado a algoritmos genéticos em jogos, com exemplos aplicados e implementação — leitura acessível e orientada ao desenvolvedor.
- **Shaker, N.; Togelius, J.; Nelson, M. J.** *Procedural Content Generation in Games*. — A referência sobre geração procedural, incluindo a abordagem baseada em busca (evolução) com funções de aptidão detalhadas.
- **Goldberg, D. E.** *Genetic Algorithms in Search, Optimization, and Machine Learning*. — O clássico que popularizou os GAs na engenharia; tratamento completo dos operadores e da teoria dos esquemas.
- **Russell, S.; Norvig, P.** *Inteligência Artificial*. 3ª ed. — Seção sobre algoritmos genéticos e busca local, situando os GAs dentro do panorama mais amplo da busca em IA.

### Referências

- RUSSELL, S.; NORVIG, P. *Inteligência Artificial*. 3ª ed. Rio de Janeiro: Elsevier, 2013.
- MILLINGTON, I. *AI for Games*. 3rd ed. Boca Raton: CRC Press, 2019.
- BOURG, D. M.; SEEMANN, G. *AI for Game Developers*. Sebastopol: O'Reilly, 2004.
- RABIN, S. (Ed.). *Game AI Pro 3: Collected Wisdom of Game AI Professionals*. Boca Raton: CRC Press, 2017.
- HOLLAND, J. H. *Adaptation in Natural and Artificial Systems*. Ann Arbor: University of Michigan Press, 1975.
- GOLDBERG, D. E. *Genetic Algorithms in Search, Optimization, and Machine Learning*. Boston: Addison-Wesley, 1989.
- SHAKER, N.; TOGELIUS, J.; NELSON, M. J. *Procedural Content Generation in Games*. Cham: Springer, 2016.
- STANLEY, K. O.; MIIKKULAINEN, R. Evolving Neural Networks through Augmenting Topologies (NEAT). *Evolutionary Computation*, v. 10, n. 2, p. 99-127, 2002.
- STANLEY, K. O.; BRYANT, B. D.; MIIKKULAINEN, R. Real-Time Neuroevolution in the NERO Video Game. *IEEE Transactions on Evolutionary Computation*, v. 9, n. 6, p. 653-668, 2005.
- TOGELIUS, J. et al. Search-Based Procedural Content Generation: A Taxonomy and Survey. *IEEE Transactions on Computational Intelligence and AI in Games*, v. 3, n. 3, p. 172-186, 2011.
