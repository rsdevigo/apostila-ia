# Capítulo 7 — Grafos e Representação do Espaço

## Introdução

Todo personagem que se move em um jogo enfrenta, em silêncio, um problema que os seres humanos resolvem sem perceber: **como ir de um lugar a outro sem atravessar paredes**. Quando um Sim caminha da cozinha até a cama, quando um aldeão de *Age of Empires* contorna uma montanha para chegar à floresta, ou quando um inimigo de *The Last of Us* rodeia um balcão para flanquear o jogador, há, por trás da naturalidade do movimento, um cálculo intenso e invisível. Esse cálculo é a **busca de caminhos** (do inglês *pathfinding*), e ela é o assunto de toda esta Parte III.

Antes, porém, de aprender *como* se busca um caminho — o que faremos no Capítulo 8, com o algoritmo A\* —, precisamos responder a uma pergunta mais básica e frequentemente ignorada: **sobre o quê, exatamente, essa busca acontece?** Um algoritmo de busca de caminhos não opera sobre a geometria bruta do cenário, com seus milhões de polígonos, texturas e detalhes visuais. Ele opera sobre uma **representação simplificada e abstrata** do espaço navegável — uma estrutura de dados que responde a duas perguntas elementares: *quais lugares eu posso ocupar?* e *de quais lugares eu posso ir diretamente para quais outros?*. Essa estrutura, em praticamente todos os jogos já feitos, é um **grafo**.

Este é, portanto, um **capítulo de fundamentação**. Seu objetivo não é ensinar um algoritmo, mas construir, do zero, todo o vocabulário matemático e conceitual necessário para que os capítulos seguintes façam sentido. Seguindo a filosofia da apostila, partimos do **problema** — como o NPC "enxerga" o mundo navegável —, edificamos os **fundamentos** com a teoria dos grafos, detalhamos o **funcionamento** das três grandes formas de representação espacial (grades, waypoints e malhas de navegação), aterrissamos nas **ferramentas** oficiais da Unity (o sistema *AI Navigation* / NavMesh) e de terceiros, e discutimos como a **indústria** decide, na prática, qual representação usar. Ao terminar este capítulo, você deverá enxergar qualquer cenário de jogo como aquilo que ele realmente é para a IA: um conjunto de **nós** conectados por **arestas** com **custos** — um grafo pronto para ser percorrido.

> **Contexto Histórico**
> A teoria dos grafos é muito mais antiga do que os computadores. Ela nasceu em 1736, quando o matemático suíço **Leonhard Euler** resolveu o famoso problema das **sete pontes de Königsberg**: seria possível passear pela cidade cruzando cada uma de suas sete pontes exatamente uma vez? Euler percebeu que a resposta não dependia da geografia detalhada, mas apenas de *como as regiões estavam conectadas*. Ao abstrair as porções de terra como pontos e as pontes como linhas ligando esses pontos, ele criou o primeiro grafo da história e provou que o passeio era impossível. Essa ideia — descartar o detalhe irrelevante e reter apenas a **estrutura de conexões** — é exatamente o que um motor de jogos faz quando transforma um cenário rico em um grafo de navegação. Quase três séculos depois, a intuição de Euler move cada NPC que dá um passo na tela.

---

## 7.1 O problema: como o NPC "enxerga" o mundo navegável

Retomemos o ciclo **Sentir → Pensar → Agir** da Parte I e as arquiteturas de decisão da Parte II. Ao final de qualquer processo de decisão, o agente frequentemente chega a uma conclusão que envolve deslocamento: *"devo ir até a última posição conhecida do jogador"*, *"devo recuar até aquela cobertura"*, *"devo levar este recurso até o depósito"*. A decisão produz um **destino**. Mas conhecer o destino não é o mesmo que saber **como chegar até ele**.

Considere o modo mais ingênuo de mover um personagem até um alvo: a cada quadro, calcular a direção em linha reta até o destino e dar um passo nessa direção. Essa técnica — que estudaremos formalmente como *seek*, um comportamento de direção (*steering*) — funciona perfeitamente em um campo aberto e vazio. O problema é que **o mundo dos jogos não é vazio**: há paredes, penhascos, rios, mesas, caixas, outros personagens. Assim que existe um único obstáculo entre o agente e o alvo, o movimento em linha reta falha: o personagem caminha até encostar na parede e ali fica, empurrando-a inutilmente, ou pior, atravessa-a, quebrando por completo a ilusão de inteligência que tanto trabalho custou a construir nas Partes anteriores.

> **Erro Comum**
> Confundir **busca de caminho** (*pathfinding*) com **direção/locomoção** (*steering*). São dois problemas distintos e complementares. A busca de caminho decide a **rota global** — a sequência de pontos que leva da origem ao destino contornando os obstáculos grandes e fixos do cenário. A direção cuida do **movimento local** — como o corpo do personagem segue essa rota de forma suave, desviando de obstáculos pequenos e dinâmicos (outro NPC que cruza a frente, uma caixa que rolou). Esta Parte trata da primeira; a segunda é um tema de locomoção e animação. Um sistema robusto usa as duas em camadas: o pathfinding traça o caminho, o steering o percorre.

O problema real, portanto, não é "mover em direção ao alvo" — isso é fácil. O problema é **encontrar uma sequência de deslocamentos que respeite a geometria do cenário**, contornando os obstáculos intransponíveis, de preferência pelo trajeto mais curto (ou mais barato) possível, e fazê-lo rápido o bastante para caber no orçamento de quadro. E aqui surge a dificuldade central: um algoritmo não consegue "raciocinar" sobre uma geometria contínua de polígonos. Ele precisa de um **modelo discreto e finito** do espaço — um conjunto bem definido de posições possíveis e de conexões entre elas. É preciso, em outras palavras, **converter o mundo em um grafo**.

> **Na Prática**
> Pense em como você mesmo daria a alguém instruções para atravessar um shopping: você não descreveria cada centímetro do piso. Você diria algo como "vá até a praça de alimentação, vire à direita no corredor das lojas de roupa, suba a escada rolante, e a loja fica ao lado do cinema". Você naturalmente reduziu o espaço contínuo a um punhado de **lugares relevantes** (nós) e **conexões diretas** entre eles (arestas). Essa redução é precisamente o que uma representação espacial faz para o NPC. O algoritmo de busca, no capítulo seguinte, é apenas quem escolhe *qual* sequência dessas conexões seguir.

Este capítulo constrói, então, a ponte entre o mundo geométrico do jogo e o mundo abstrato dos algoritmos. Começaremos pela matemática que dá nome a essa ponte — a teoria dos grafos — e depois veremos as três maneiras concretas de erguê-la.

---

## 7.2 Fundamentos de teoria dos grafos aplicados a jogos

Um **grafo** é uma das estruturas mais simples e poderosas da Ciência da Computação. Informalmente, um grafo é apenas um conjunto de **pontos** ligados por **linhas**. Formalmente, um grafo `G` é um par ordenado `G = (V, E)`, em que `V` é um conjunto de **vértices** (do inglês *vertices*, também chamados de **nós**, *nodes*) e `E` é um conjunto de **arestas** (do inglês *edges*), cada aresta sendo um par de vértices que indica uma conexão entre eles. Essa definição, com pequenas variações, é a mesma que se encontra em Bondy & Murty (*Graph Theory*) e em Cormen et al. (*Algoritmos*); sua elegância está em ser genérica o bastante para modelar redes sociais, mapas rodoviários, circuitos elétricos e — o que nos interessa — o espaço navegável de um jogo.

A tradução para jogos é direta e vale a pena fixá-la desde já, pois será usada em todos os capítulos seguintes:

- cada **vértice/nó** representa **um lugar que o agente pode ocupar** (uma célula de uma grade, um ponto de rota, um polígono de uma malha de navegação);
- cada **aresta** representa a possibilidade de **ir diretamente de um lugar a outro** sem passar por um terceiro (um passo de uma célula para a vizinha, uma linha de visão livre entre dois waypoints);
- o **caminho** que buscamos é uma **sequência de arestas** que liga o nó de origem ao nó de destino.

> **Atenção**
> Um ponto de terminologia que confunde iniciantes: os "nós" de um grafo de navegação **não são necessariamente pontos isolados no espaço**. Dependendo da representação, um nó pode ser uma **célula quadrada** (grade), um **ponto** (waypoint) ou até um **polígono inteiro** (NavMesh). O que os unifica é o papel abstrato: no grafo, cada um é *um vértice*, um lugar entre o qual e seus vizinhos existe uma conexão. Manter clara a distinção entre o *objeto geométrico* (célula, ponto, polígono) e seu *papel no grafo* (vértice) evita a maior parte da confusão deste capítulo.

### 7.2.1 Vértices, arestas, pesos e grafos direcionados

Vamos refinar os elementos do grafo com os atributos de que a busca de caminhos precisa.

**Vértices (nós).** São os lugares. Um nó pode carregar informações associadas: sua posição no mundo, se está ocupado, o tipo de terreno que representa (grama, água, lava), e — durante a busca — valores temporários de custo que o algoritmo do Capítulo 8 irá calcular. Do ponto de vista do grafo puro, porém, um vértice é apenas uma identidade: "o lugar A", "o lugar B".

**Arestas.** São as conexões. Uma aresta liga dois vértices e declara que existe uma passagem **direta** entre eles. É crucial entender que a existência de uma aresta é uma afirmação sobre *adjacência local*, não sobre *alcance global*: dizer que há uma aresta entre A e B não diz nada sobre como chegar de A a um vértice distante Z — isso é justamente o que o algoritmo de busca terá de descobrir, encadeando arestas.

**Pesos (custos).** Nem toda conexão é igualmente "barata". Atravessar um pântano deve custar mais do que atravessar uma estrada; subir uma ladeira, mais do que descer. Para modelar isso, associamos a cada aresta um **peso** (ou **custo**), um número que representa o "esforço" de percorrê-la. Um grafo com pesos nas arestas é chamado de **grafo ponderado** (*weighted graph*). Quando todas as arestas têm o mesmo custo (por exemplo, custo 1 para todo passo), o grafo é dito **não ponderado** — um caso particular em que buscar o caminho mais curto equivale a buscar o caminho com **menos arestas**. O conceito de custo é o que permite ao pathfinding ir além de "o caminho com menos passos" e alcançar "o caminho mais **conveniente**", desviando de terrenos perigosos ou lentos mesmo que isso signifique dar mais passos.

> **Na Indústria**
> Os pesos de aresta são uma das ferramentas mais expressivas — e mais subutilizadas — do design de IA. Ao atribuir custo alto a células próximas de precipícios, de lava ou da linha de tiro do jogador, o designer faz os NPCs **preferirem naturalmente** rotas seguras, sem escrever uma única regra explícita de "evite o perigo". O caminho seguro simplesmente *emerge* como o mais barato. Esse é um exemplo perfeito da filosofia da apostila: a "inteligência" observada (o inimigo que se esgueira pela lateral em vez de atravessar o campo aberto) é, na verdade, o subproduto de uma boa modelagem de custos, não de um raciocínio sofisticado.

**Grafos direcionados e não direcionados.** Uma aresta pode ser de mão dupla ou de mão única. Em um **grafo não direcionado**, a aresta entre A e B permite ir de A para B *e* de B para A, com o mesmo custo — o caso comum de terreno plano. Em um **grafo direcionado** (*directed graph* ou *dígrafo*), as conexões têm sentido: pode existir uma aresta de A para B sem existir a de B para A, ou os custos podem diferir em cada sentido. Isso modela situações reais de jogo: uma plataforma da qual se pode **pular para baixo**, mas não subir; uma ladeira barata de descer e cara de subir; um portão de mão única. A capacidade de representar assimetrias é uma das razões pelas quais o grafo é a estrutura escolhida — ela captura, com naturalidade, a irreversibilidade de muitos movimentos.

[DIAGRAMA]
Título: Anatomia de um grafo de navegação
Objetivo pedagógico: Fixar visualmente os quatro elementos fundamentais — vértice, aresta, peso e direção — em um único diagrama de referência.
Descrição detalhada: Desenhar de cinco a seis vértices como círculos rotulados (A, B, C, D, E, F) espalhados como um pequeno mapa. Ligá-los por arestas: a maioria como linhas simples de mão dupla, com um número sobre cada linha indicando o peso (por exemplo, A–B custo 1, B–C custo 4 atravessando um "pântano" sombreado, C–D custo 1). Incluir pelo menos uma aresta direcionada, desenhada como seta de sentido único (por exemplo, E→F, rotulada "queda: só descida"). Sombrear a região entre B e C para justificar visualmente o custo maior. Uma legenda lateral deve associar: círculo = vértice/nó (lugar); linha = aresta (conexão direta); número sobre a linha = peso/custo; seta = aresta direcionada (mão única).
Elementos obrigatórios: vértices rotulados; arestas com pesos numéricos; pelo menos uma aresta direcionada com seta; uma região de custo elevado destacada; legenda dos quatro conceitos.
[/DIAGRAMA]

### 7.2.2 Custo, conectividade e representação em memória

Três conceitos derivados completam o ferramental de grafos de que precisaremos.

**Custo de um caminho.** Se uma aresta tem um custo, um **caminho** — sequência de arestas — tem um custo total, que é simplesmente a **soma dos custos das arestas** que o compõem. O objetivo do pathfinding pode agora ser enunciado com precisão: dado um nó de origem e um nó de destino, encontrar o caminho de **menor custo total** entre eles (o chamado *caminho ótimo* ou *caminho mínimo*). Note que "menor custo" nem sempre é "menor distância geométrica": num grafo ponderado, o caminho mais curto em metros pode ser mais caro do que um desvio por terreno favorável. Essa distinção é o coração da modelagem por custos.

**Conectividade.** Um grafo é **conexo** se existe pelo menos um caminho entre qualquer par de vértices. Em jogos, essa propriedade tem consequência prática direta: se o grafo de navegação estiver dividido em **componentes desconexos** — por exemplo, uma ilha sem ponte ligando ao continente —, nenhum algoritmo do mundo encontrará um caminho entre eles, porque ele simplesmente **não existe**. Reconhecer a conectividade é vital: muitos jogos pré-calculam, para cada nó, a qual "ilha" (componente conexo) ele pertence, de modo a **responder instantaneamente** que dois pontos são inalcançáveis, sem desperdiçar processamento numa busca fadada ao fracasso.

> **Boa Prática**
> Antes de disparar uma busca de caminho potencialmente cara, verifique se origem e destino estão no **mesmo componente conexo**. Essa checagem, feita com uma marcação pré-calculada de "ilhas" de navegação, custa quase nada e evita o pior caso do A\*: uma busca que explora *todo* o mapa alcançável apenas para concluir, ao final, que o destino era inalcançável. Em jogos de mundo aberto, essa única otimização economiza uma quantidade enorme de processamento desperdiçado.

**Representação em memória.** Um grafo precisa ser armazenado em alguma estrutura de dados, e a escolha afeta o desempenho da busca. As duas formas clássicas, apresentadas em Cormen et al., são:

- **Lista de adjacência:** para cada vértice, guarda-se uma lista dos vértices vizinhos (e dos custos das arestas correspondentes). É econômica em memória para grafos **esparsos** — aqueles em que cada nó tem poucos vizinhos —, que é exatamente o caso da maioria dos grafos de navegação (uma célula de grade tem no máximo oito vizinhos; um waypoint, um punhado). É a representação dominante em pathfinding.
- **Matriz de adjacência:** uma tabela `n × n` em que a posição `(i, j)` indica se (e com que custo) existe aresta do vértice `i` para o `j`. Consulta a existência de uma aresta em tempo constante, mas consome memória proporcional a `n²`, o que a torna proibitiva para os grafos grandes e esparsos típicos de jogos.

Há ainda um ponto sutil e importante: em muitas representações espaciais, o grafo é **implícito**, não armazenado explicitamente. Numa grade, por exemplo, não é preciso guardar uma lista de arestas — os vizinhos de uma célula `(x, y)` são calculados na hora, por aritmética simples (`(x±1, y)`, `(x, y±1)` etc.). Essa geração de vizinhos "sob demanda" economiza memória e é uma das razões da popularidade das grades. Guardaremos essa observação para a seção 7.3.1.

---

## 7.3 Representações espaciais

Munidos da teoria dos grafos, podemos agora enfrentar a questão central do capítulo: **como transformar o cenário concreto de um jogo em um grafo navegável?** Não há uma resposta única. A indústria consolidou três grandes famílias de representação, cada uma com um perfil distinto de precisão, custo de memória, custo de construção e facilidade de manutenção: **grades**, **waypoints** e **malhas de navegação**. Estudaremos as três, sempre lembrando que todas produzem, ao final, a mesma coisa abstrata — um grafo `(V, E)` com custos — e que a escolha entre elas é uma das decisões de arquitetura mais consequentes de um projeto.

### 7.3.1 Grades (grid) e grafos de células

A representação mais intuitiva do espaço é a **grade** (*grid*): sobrepõe-se ao mapa uma malha regular de células, tipicamente quadradas, e classifica-se cada célula como **navegável** (livre) ou **bloqueada** (obstáculo). O grafo emerge de forma natural: cada célula livre é um **nó**, e há uma **aresta** entre duas células livres adjacentes. Jogos de tabuleiro digitais, roguelikes, muitos jogos de estratégia por turnos e incontáveis títulos 2D usam grades como representação primária, porque a estrutura do mundo já é, ela própria, quadriculada.

A primeira decisão de projeto numa grade é a **conectividade das células**, isto é, quais vizinhas contam como adjacentes:

- **Conectividade-4 (vizinhança de von Neumann):** cada célula liga-se apenas às quatro vizinhas ortogonais (norte, sul, leste, oeste). O movimento é "em cruz"; não há diagonais. Simples e sem ambiguidades, mas produz caminhos com aparência "escadinha" e superestima distâncias na diagonal.
- **Conectividade-8 (vizinhança de Moore):** cada célula liga-se às oito vizinhas, incluindo as quatro diagonais. Produz caminhos mais naturais e curtos, mas introduz duas sutilezas: o custo de um passo diagonal deve ser maior que o de um ortogonal (a diagonal de um quadrado de lado 1 mede `√2 ≈ 1,41`, não 1), e é preciso decidir se um agente pode "cortar" a quina entre dois obstáculos diagonais — o chamado problema do *corner cutting*.

> **Atenção**
> O detalhe do **custo diagonal** parece pedante, mas ignorá-lo produz um erro visível: se um passo diagonal custar o mesmo que um ortogonal (custo 1 para ambos), o algoritmo passa a acreditar que andar na diagonal é tão barato quanto andar reto e produz caminhos em ziguezague antinaturais. O valor correto é aproximadamente **1,41** (`√2`) para a diagonal contra **1,0** para a ortogonal. Muitos motores usam a aproximação inteira de custo **14** para diagonais e **10** para ortogonais, evitando aritmética de ponto flutuante sem perder a proporção. Esse é um "erro comum" clássico de quem implementa pathfinding em grade pela primeira vez.

[DIAGRAMA]
Título: Grade de navegação e sua conversão em grafo
Objetivo pedagógico: Mostrar que uma grade de células livres/bloqueadas é, na verdade, um grafo implícito, e ilustrar a diferença entre conectividade-4 e conectividade-8.
Descrição detalhada: Desenhar uma grade 6×6 de células quadradas. Marcar cerca de seis células como bloqueadas (preenchidas de cinza-escuro, formando uma pequena parede em L). Numa célula livre central, desenhar setas para os vizinhos: no painel da esquerda, apenas quatro setas ortogonais (conectividade-4); no painel da direita, oito setas incluindo diagonais (conectividade-8), com as setas diagonais rotuladas com custo ≈1,41 e as ortogonais com custo 1,0. Destacar, no painel da direita, um caso de "corner cutting": uma diagonal que passaria entre duas células bloqueadas em quina, marcada com um "X" vermelho indicando que costuma ser proibida. Sobrepor, em transparência, os nós (pontos no centro de cada célula livre) e as arestas (linhas ligando centros de células adjacentes) para evidenciar o grafo implícito.
Elementos obrigatórios: grade com células livres e bloqueadas; painel-4 e painel-8 lado a lado; custos ortogonal (1,0) e diagonal (1,41); marcação de corner cutting proibido; sobreposição do grafo implícito (nós e arestas).
[/DIAGRAMA]

**Vantagens das grades.** São **simples de implementar e de entender**; o grafo é **implícito** (os vizinhos são calculados por aritmética, dispensando armazenamento de arestas); atualizações dinâmicas são triviais (bloquear ou liberar uma célula é mudar um único valor, útil quando o jogador constrói ou destrói estruturas em tempo real); e a regularidade abre caminho para as otimizações poderosas que veremos no Capítulo 9 (o Jump Point Search **exige** uma grade uniforme). Além disso, a grade é uma representação que serve a vários propósitos ao mesmo tempo: a mesma estrutura usada para pathfinding alimenta, na Parte IV, os **mapas de influência**.

**Limitações das grades.** O custo de memória e de busca cresce com a **resolução**: uma grade fina o suficiente para representar bem espaços apertados pode ter milhões de células, tornando a busca cara — problema que motiva diretamente o Capítulo 9. Grades quadradas representam mal geometrias **oblíquas ou curvas** (uma parede diagonal vira uma escadinha de blocos). E há o já mencionado viés direcional: mesmo com conectividade-8, os caminhos tendem a ter uma aparência artificialmente "quadriculada" antes da suavização.

> **Curiosidade**
> Nem toda grade é quadrada. Alguns jogos usam grades **hexagonais** (populares em estratégia, como em vários títulos da série *Civilization* e em wargames), nas quais cada célula tem seis vizinhos **equidistantes** — o que elimina o problema do custo diagonal, já que todos os passos custam o mesmo. Outros usam **grades triangulares**. A escolha da forma da célula é uma decisão de design que afeta tanto a estética do movimento quanto a matemática da busca. O grafo subjacente, no entanto, continua sendo apenas nós e arestas — a forma da célula muda quem são os vizinhos, não a natureza abstrata da representação.

### 7.3.2 Waypoints e grafos de pontos de rota

Se a grade cobre o espaço inteiro com células, a abordagem de **waypoints** (pontos de rota) faz o oposto: espalha pelo cenário um número **pequeno** de pontos estratégicos, tipicamente posicionados **à mão** pelos designers de nível, e conecta por arestas os pares de pontos entre os quais existe **linha de visão livre** (isto é, um trajeto reto sem obstáculos). O grafo resultante é **esparso** — poucos nós, poucas arestas — e, por isso, a busca sobre ele é extremamente **rápida**. Essa foi a representação dominante nos jogos 3D do final dos anos 1990 e início dos 2000, antes da popularização das malhas de navegação.

O funcionamento em jogo tem três passos. Primeiro, o agente encontra o waypoint mais próximo de sua posição atual e o mais próximo do destino (os pontos de "entrada" e "saída" da rede). Segundo, roda-se a busca de caminho sobre o grafo de waypoints, obtendo uma sequência de pontos a visitar. Terceiro, o agente caminha em linha reta de waypoint em waypoint. Como as arestas foram construídas garantindo linha de visão livre, cada trecho reto é, por definição, transitável.

> **Contexto Histórico**
> Os grafos de waypoints foram a espinha dorsal da IA de movimento em clássicos do tiro em primeira pessoa e da ação 3D da virada do milênio. Marcavam-se manualmente, no editor de níveis, os pontos por onde os inimigos deveriam poder passar — corredores, batentes de porta, cantos de sala — e o motor conectava os que se enxergavam. Essa herança explica por que, em jogos daquela era, os inimigos às vezes pareciam "correr sobre trilhos invisíveis": eles literalmente seguiam de waypoint em waypoint. A técnica era barata e eficaz, mas dependia intensamente do trabalho manual dos designers, o que se tornaria seu calcanhar de Aquiles.

**Vantagens dos waypoints.** O grafo é **minúsculo**, então a busca é **muito rápida** e consome pouquíssima memória. O designer tem **controle autoral** direto: pode induzir os NPCs a preferir certos corredores, a cobrir posições táticas específicas, a evitar zonas — basta posicionar (ou não) waypoints ali. Essa controlabilidade dialoga diretamente com o critério de autoria da Parte I.

**Limitações dos waypoints.** A principal é o **custo de autoria e manutenção**: os pontos são colocados à mão, e cada alteração no nível exige revisá-los. Como a rede cobre apenas os trajetos previstos, o movimento fica **"amarrado aos trilhos"**: um agente que precise se afastar de um waypoint (para desviar de algo, para se posicionar num ponto não previsto) sai da rede e fica sem orientação. A representação **não descreve áreas**, apenas linhas: o NPC sabe ir do ponto 5 ao ponto 6, mas não tem noção do *espaço livre ao redor* desse trajeto, o que dificulta movimentação tática fluida (esgueirar-se, espalhar-se, flanquear por qualquer ponto de uma sala aberta). Foi justamente para superar essa limitação — descrever **áreas navegáveis** em vez de apenas **rotas** — que surgiu a terceira família.

[DIAGRAMA]
Título: Grafo de waypoints sobre um cenário
Objetivo pedagógico: Ilustrar como poucos pontos de rota, conectados por linha de visão, formam um grafo esparso, e contrastar com a cobertura total de uma grade.
Descrição detalhada: Desenhar a planta baixa de um pequeno cenário com duas salas ligadas por um corredor e alguns obstáculos (mesas, colunas) representados como blocos. Espalhar de seis a oito waypoints (pontos circulares numerados) em posições estratégicas: cantos de sala, batentes de porta, meio do corredor. Ligar por linhas (arestas) apenas os pares de waypoints com linha de visão livre — deixar claro que dois pontos separados por uma coluna NÃO são ligados diretamente. Traçar, em destaque, um caminho de exemplo do waypoint 1 (numa sala) ao waypoint 7 (na outra), passando pelo corredor. Ao lado, em miniatura, mostrar o mesmo cenário coberto por uma grade densa, para contraste visual entre "poucos pontos" (waypoints) e "cobertura total" (grade).
Elementos obrigatórios: planta com duas salas, corredor e obstáculos; waypoints numerados; arestas apenas entre pontos com linha de visão; um caminho de exemplo destacado; miniatura comparativa da grade.
[/DIAGRAMA]

### 7.3.3 Malhas de navegação (NavMesh)

A **malha de navegação** (*navigation mesh*, universalmente abreviada **NavMesh**) é a representação dominante nos jogos 3D modernos e a que sustenta os sistemas oficiais de navegação da Unity e da Unreal. Sua ideia central resolve a limitação dos waypoints de forma elegante: em vez de representar o espaço navegável como **pontos** ou como **células regulares**, a NavMesh o representa como um conjunto de **polígonos convexos** (tipicamente triângulos ou quadriláteros) que **recobrem toda a superfície caminhável** do cenário. Cada polígono cobre uma **área** — uma região de chão plano o suficiente para ser atravessada livremente em linha reta em qualquer direção.

O grafo, novamente, emerge naturalmente, mas com uma virada conceitual importante: **cada polígono é um nó**, e há uma **aresta** entre dois polígonos que **compartilham uma borda** (por onde o agente pode cruzar de um para o outro). Repare na inversão em relação à grade: lá, o nó era uma célula minúscula de tamanho fixo; aqui, o nó é um polígono que pode ser **grande** onde o espaço é aberto e **pequeno** onde o espaço é recortado. Um salão vazio pode ser um único triângulo enorme; um corredor cheio de colunas, uma colcha de retalhos de triângulos pequenos. Essa **adaptatividade** é a grande virtude da NavMesh: ela gasta detalhe (e, portanto, nós) apenas onde a geometria exige, mantendo o grafo pequeno em áreas abertas.

> **Na Prática**
> A propriedade de "cobrir áreas, não linhas" é o que dá à NavMesh sua fluidez característica. Como cada polígono representa uma região *inteira* onde o movimento livre é garantido, o agente não fica preso a trilhos: dentro de um polígono, ele pode ir em linha reta para qualquer ponto; a busca só precisa decidir **por quais polígonos passar**, e o refinamento fino do trajeto (por onde exatamente cruzar cada borda) é resolvido depois, na fase de suavização (Capítulo 9). É essa combinação — busca grossa por polígonos + ajuste fino dentro deles — que produz o movimento natural dos NPCs em jogos como *The Last of Us* ou qualquer título 3D contemporâneo.

**Como a NavMesh é construída.** Aqui reside outra vantagem decisiva sobre os waypoints: a malha é, em geral, **gerada automaticamente** por um processo de *baking* ("cozimento"). O motor analisa a geometria estática do cenário e os parâmetros do agente — seu **raio** (largura), sua **altura**, a **inclinação máxima** que ele consegue subir e a **altura de degrau** que consegue vencer — e calcula quais superfícies são caminháveis para *aquele* agente, recuando as bordas da malha para longe das paredes pela medida do raio (de modo que o centro do agente nunca leve seu corpo a atravessar um obstáculo). O resultado é uma malha sob medida, produzida em segundos ou minutos, sem a marcação manual ponto a ponto que atormentava os waypoints. A biblioteca de código aberto **Recast**, de Mikko Mononen, tornou-se o padrão de fato para essa geração e está por trás dos sistemas de NavMesh de vários motores comerciais.

> **Erro Comum**
> Esquecer que uma NavMesh é construída **para um agente específico**. A malha calculada para um humano de raio pequeno e passo baixo **não serve** para um veículo largo ou para um monstro gigante que não passa pelas mesmas portas nem sobe as mesmas rampas. Quando um jogo tem agentes de tamanhos muito diferentes, é preciso gerar **várias NavMeshes**, uma por perfil de agente. Assumir que "uma malha serve para todos" produz o bug clássico do inimigo enorme que tenta atravessar uma passagem estreita porque a malha foi feita pensando em personagens pequenos.

[DIAGRAMA]
Título: Malha de navegação (NavMesh) e seu grafo de polígonos
Objetivo pedagógico: Mostrar como a superfície caminhável é recoberta por polígonos convexos de tamanhos variados e como o grafo se forma pela adjacência entre polígonos, com nós no centro de cada polígono.
Descrição detalhada: Desenhar a planta de um cenário 3D visto de cima, com um salão amplo, um corredor estreito e uma varanda, além de obstáculos (colunas, um fosso). Recobrir a superfície caminhável com polígonos convexos (triângulos e quadriláteros): grandes no salão aberto, pequenos e numerosos ao redor das colunas e no corredor. Deixar sem cobertura (em branco/hachurado) as áreas não caminháveis: paredes, o fosso, o interior das colunas. Mostrar a borda da malha recuada das paredes por uma pequena margem, rotulada "recuo = raio do agente". Sobrepor o grafo: um ponto (nó) no centro de cada polígono e linhas (arestas) ligando polígonos que compartilham uma borda. Destacar um caminho de exemplo atravessando uma sequência de polígonos do salão até a varanda.
Elementos obrigatórios: polígonos convexos de tamanhos variados cobrindo o caminhável; áreas não caminháveis destacadas; margem de recuo rotulada; grafo sobreposto (nó por polígono, aresta por borda compartilhada); caminho de exemplo por polígonos.
[/DIAGRAMA]

**Vantagens da NavMesh.** Representa **áreas**, permitindo movimento fluido e tático; **adapta a resolução** à geometria (poucos nós em áreas abertas, muitos onde há detalhe), o que costuma resultar em grafos **menores** que uma grade fina equivalente; é **gerada automaticamente**, eliminando a autoria manual dos waypoints; e integra-se naturalmente a **modificadores de custo por área** (marcar uma região como "água" ou "perigo" com custo elevado). É a razão de ser o padrão da indústria 3D.

**Limitações da NavMesh.** A **geração é complexa** e o resultado, sensível aos parâmetros — uma malha mal configurada deixa buracos, sobe onde não deveria ou ignora passagens estreitas. Atualizações **dinâmicas** (reconstruir a malha quando o cenário muda em tempo real) são **mais caras** do que numa grade, exigindo técnicas específicas (recozimento local, *carving* de obstáculos). E, por representar apenas o **chão caminhável**, movimentos que saem da superfície — pular, escalar, usar tirolesa — precisam de complementos, os chamados **off-mesh links** (ligações fora da malha), que reconectam manualmente polígonos separados por um vão. Ainda assim, para a esmagadora maioria dos jogos 3D, a NavMesh é a escolha padrão, e é ela que a Unity oferece como sistema oficial.

---

## 7.4 Ferramentas: AI Navigation / NavMesh na Unity

Fiel à filosofia da apostila, esta seção **não é um tutorial de menus**: não ensinaremos onde clicar. O objetivo é mostrar como os **conceitos** deste capítulo — grafo, nós, arestas, custos, áreas, recuo pelo raio do agente — aparecem, com esses mesmos nomes, na ferramenta oficial que o estudante usará na prática.

A Unity oferece a navegação por NavMesh através do pacote **AI Navigation** (o sistema moderno, baseado em componentes, que substituiu o antigo NavMesh embutido no editor). Seus componentes principais correspondem quase um a um aos conceitos que estudamos:

- O **NavMesh Surface** define uma superfície e dispara o processo de **baking** que gera a malha. Ao "cozinhar", a Unity roda internamente um algoritmo da família Recast, exatamente o processo de geração automática descrito na seção 7.3.3. Os parâmetros de baking — **Agent Radius**, **Agent Height**, **Max Slope**, **Step Height** — são precisamente o raio, a altura, a inclinação máxima e a altura de degrau que definem para qual agente a malha está sendo construída.
- O **NavMesh Agent** é o componente que se anexa ao personagem para que ele *use* a malha: dado um destino, ele encontra o polígono de origem e de destino, executa a busca de caminho (um A\* sobre o grafo de polígonos, assunto do Capítulo 8) e conduz o movimento ao longo do caminho, já com um steering básico de desvio de outros agentes.
- Os **NavMesh Modifiers** e as **áreas de navegação** (*NavMesh Areas*) implementam os **custos por área**: pode-se marcar uma região como "Água" ou "Lama" e atribuir-lhe um custo maior, fazendo os agentes preferirem naturalmente as rotas baratas — a materialização direta dos **pesos de aresta** da seção 7.2.1.
- Os **Off-Mesh Links** (ou *NavMesh Links*) reconectam trechos de malha separados por vãos, modelando pulos, escadas ou teletransportes — a solução para a limitação de "só chão caminhável" discutida acima. No grafo abstrato, cada link é simplesmente **mais uma aresta**, muitas vezes **direcionada** (pular para baixo, mas não para cima), reencontrando o conceito de grafo direcionado da seção 7.2.1.

[IMAGEM NECESSÁRIA]
Título: NavMesh gerada sobre um cenário 3D no editor da Unity
Objetivo didático: Mostrar ao estudante como a abstração de "polígonos que recobrem o chão caminhável" aparece concretamente na ferramenta oficial, ligando o conceito da seção 7.3.3 à prática que ele verá na tela.
Descrição: Captura de tela do editor da Unity com um cenário 3D simples (salas, corredor, obstáculos) e a NavMesh gerada visível como uma sobreposição azul semitransparente cobrindo apenas o piso caminhável, recuada das paredes e contornando os obstáculos. De preferência, com o painel do componente NavMesh Surface visível, mostrando os parâmetros Agent Radius, Agent Height, Max Slope e Step Height.
Tipo: Screenshot de software (editor da Unity).
Como produzir: Montar um cenário mínimo na Unity com o pacote AI Navigation, adicionar um NavMesh Surface, executar o Bake e capturar a Scene View com a sobreposição da malha ativada, junto ao Inspector do NavMesh Surface.
Legenda sugerida: "A NavMesh (em azul) recobre apenas a superfície caminhável, recuada das paredes pela medida do raio do agente. Cada polígono dessa malha é um nó do grafo de navegação."
[/IMAGEM NECESSÁRIA]

> **Na Indústria**
> É instrutivo notar que a Unreal Engine oferece um sistema conceitualmente equivalente — sua navegação também é baseada em NavMesh gerada por Recast — e que motores proprietários de grandes estúdios seguem a mesma filosofia. Isso não é coincidência: a NavMesh venceu como padrão da indústria 3D porque resolve, de uma vez, os dois maiores problemas das alternativas (a explosão de memória das grades finas e o custo de autoria dos waypoints). Ao aprender os **conceitos**, e não os menus, o estudante transfere o conhecimento sem esforço de um motor a outro — que é exatamente o que a apostila pretende.

Para **grades**, a Unity não traz um sistema nativo dedicado de pathfinding em grid (o NavMesh é a via oficial), mas grades são triviais de implementar sobre a lógica do jogo quando o design pede — e são a base de assets de terceiros voltados a jogos 2D e de estratégia. Para **waypoints**, embora hoje raramente sejam a representação primária de movimento, o conceito sobrevive como **pontos de interesse** táticos (posições de cobertura, pontos de patrulha) que decoram uma NavMesh, combinando as duas ideias.

---

## 7.5 Ferramentas de terceiros

Quando o sistema oficial não atende — por exigir grades, por precisar de reconstrução dinâmica muito frequente, por buscar controle fino sobre o algoritmo — a comunidade oferece alternativas maduras.

A mais conhecida no ecossistema Unity é o **A\* Pathfinding Project**, de Aron Granberg, uma biblioteca comercial (com versão gratuita) que suporta **múltiplas representações** — grades quadradas, grades hexagonais, grafos de pontos (waypoints) e malhas de navegação geradas de forma independente do sistema nativo. Sua flexibilidade o torna a escolha frequente de jogos de estratégia e de projetos que precisam de **grid-based pathfinding** com atualização dinâmica rápida, cenário em que ele costuma superar o NavMesh nativo. Voltaremos a ele no Capítulo 8, ao discutir ferramentas de A\*.

No campo do código aberto, a já mencionada biblioteca **Recast & Detour** (Recast para gerar a malha, Detour para buscar e seguir caminhos nela) é o motor por trás de boa parte da navegação 3D da indústria e pode ser integrada diretamente por equipes que desenvolvem tecnologia própria. Conhecer sua existência ajuda o estudante a entender que o "botão de bake" da Unity não é mágica, mas a interface amigável de um algoritmo público e bem documentado.

> **Boa Prática**
> A escolha da ferramenta deve seguir a escolha da **representação**, e não o contrário. Primeiro pergunte: *meu mundo é melhor descrito por uma grade, por waypoints ou por áreas (NavMesh)?* — decisão que depende de o jogo ser 2D quadriculado, 3D com espaço aberto, dinâmico ou estático, com um ou muitos tipos de agente. Só então escolha a ferramenta que melhor implementa aquela representação. Escolher a ferramenta antes de entender o problema é uma inversão que costuma custar caro em retrabalho.

---

## Resumo

Este capítulo estabeleceu a **fundação espacial** de toda a Parte III. Vimos que a busca de caminhos não opera sobre a geometria bruta do jogo, mas sobre uma **representação abstrata e discreta** do espaço navegável — invariavelmente, um **grafo**. Distinguimos, com cuidado, a **busca de caminho** (rota global, tema desta Parte) da **direção/locomoção** (movimento local suave), um par de conceitos frequentemente confundido.

Construímos os **fundamentos de teoria dos grafos** aplicados a jogos: **vértices/nós** (lugares que o agente pode ocupar), **arestas** (conexões diretas entre lugares), **pesos/custos** (o esforço de percorrer cada conexão) e a distinção entre grafos **direcionados** e **não direcionados**, que captura assimetrias reais de movimento. Vimos que o **custo de um caminho** é a soma dos custos de suas arestas, que a **conectividade** determina o que é sequer alcançável, e que grafos de navegação são tipicamente **esparsos**, favorecendo a **lista de adjacência** — muitas vezes de forma **implícita**, com vizinhos calculados sob demanda.

Estudamos as **três grandes representações espaciais** e seus compromissos: as **grades**, simples, dinâmicas e de grafo implícito, mas custosas em alta resolução e presas à sua própria regularidade; os **waypoints**, de grafo esparso e busca velocíssima, porém dependentes de autoria manual e "amarrados a trilhos"; e as **malhas de navegação (NavMesh)**, que representam **áreas** em vez de linhas, adaptam a resolução à geometria, são geradas automaticamente e se tornaram o **padrão da indústria 3D**. Por fim, ancoramos esses conceitos na ferramenta oficial da Unity — o pacote **AI Navigation** (NavMesh Surface, NavMesh Agent, áreas de custo, off-mesh links) — e em alternativas de terceiros como o **A\* Pathfinding Project** e o **Recast & Detour**, sempre reforçando que a escolha da ferramenta deve seguir a escolha da representação.

O leitor termina este capítulo com aquilo que os próximos exigem: a capacidade de enxergar **qualquer cenário como um grafo de nós, arestas e custos**. Sobre essa base, o Capítulo 8 construirá o algoritmo que percorre esse grafo em busca do melhor caminho — o A\*.

## Exercícios de Fixação

1. Defina, com suas palavras, os quatro elementos de um grafo de navegação (**vértice, aresta, peso, direção**) e dê, para cada um, um exemplo concreto tirado de um jogo.
2. Explique a diferença entre **busca de caminho** (*pathfinding*) e **direção/locomoção** (*steering*). Por que um sistema robusto precisa dos dois, e em que ordem eles atuam?
3. Um grafo de navegação tem dois **componentes desconexos** (uma ilha e um continente sem ponte). O que acontece se um NPC na ilha receber a ordem de ir até um ponto no continente? Como um jogo pode **detectar isso antes** de rodar uma busca cara? Justifique.
4. Numa grade com **conectividade-8**, por que atribuir custo **1,0** ao passo diagonal (igual ao ortogonal) é um erro? Qual é o custo correto e por quê? O que se observa no caminho resultante quando esse erro é cometido?
5. Compare **grades**, **waypoints** e **NavMesh** quanto a: (a) o que cada nó representa; (b) como o grafo é obtido (manual, implícito, gerado); (c) custo de memória; (d) facilidade de atualização dinâmica. Organize sua resposta em forma de tabela.
6. Explique por que a NavMesh "representa **áreas**, não **linhas**" e por que essa propriedade produz movimento mais fluido e tático do que os waypoints.
7. Uma NavMesh foi gerada para um personagem humano de raio pequeno. O jogo vai adicionar um chefe gigante que não cabe em corredores estreitos. Que problema surgirá e como resolvê-lo? Relacione sua resposta ao parâmetro **Agent Radius**.
8. Dê um exemplo de movimento em jogo que **não** pode ser representado por uma malha de chão caminhável comum (por exemplo, um pulo entre plataformas) e explique como um **off-mesh link** o resolve. Por que, no grafo abstrato, esse link costuma ser uma aresta **direcionada**?
9. Um designer quer que os inimigos **evitem naturalmente** um campo aberto exposto ao fogo do jogador, sem escrever regras explícitas de "fuja do perigo". Explique como fazê-lo usando **pesos de aresta**. Relacione com o conceito de "ilusão de inteligência" da Parte I.
10. Justifique a afirmação: *"nenhum algoritmo de busca é melhor do que a representação sobre a qual roda"*. Dê um exemplo em que uma boa escolha de representação resolve um problema que nenhum ajuste de algoritmo resolveria.

## Referências

BONDY, John Adrian; MURTY, U. S. R. *Graph Theory.* Nova York: Springer, 2008. (Graduate Texts in Mathematics, v. 244.)

CORMEN, Thomas H.; LEISERSON, Charles E.; RIVEST, Ronald L.; STEIN, Clifford. *Algoritmos: Teoria e Prática.* 3. ed. Rio de Janeiro: Elsevier, 2012.

MILLINGTON, Ian. *AI for Games.* 3. ed. Boca Raton: CRC Press, 2019.

RABIN, Steve (org.). *Game AI Pro 3: Collected Wisdom of Game AI Professionals.* Boca Raton: A K Peters/CRC Press, 2017.

RUSSELL, Stuart J.; NORVIG, Peter. *Inteligência Artificial.* 3. ed. Rio de Janeiro: Elsevier, 2013.

MONONEN, Mikko. *Recast & Detour: Navigation Mesh Toolset.* Documentação do projeto de código aberto.

UNITY TECHNOLOGIES. *Documentação oficial: AI Navigation (NavMesh Surface, NavMesh Agent, NavMesh Modifier, Off-Mesh Links).*

GRANBERG, Aron. *A\* Pathfinding Project — Documentação.*
