# Capítulo 11 — Minimax e Busca Adversarial

## Introdução

Todas as técnicas que estudamos até aqui compartilham uma suposição que agora precisamos abandonar: a de que o mundo, por mais complexo que seja, **não tem a intenção de nos derrotar**. Uma máquina de estados decide o que um guarda faz; um algoritmo A\* encontra a rota até um destino; um mapa de influência aponta a melhor posição tática. Em nenhum desses casos o ambiente **raciocina de volta** — as paredes não se movem para bloquear a rota, o terreno não reconfigura o perigo para invalidar a decisão do agente. Mas há uma classe inteira de jogos em que existe, do outro lado do tabuleiro, uma mente que persegue exatamente o objetivo oposto ao nosso e que, a cada instante, escolhe deliberadamente a jogada que mais nos prejudica. É o mundo do **xadrez**, das **damas**, do **jogo da velha**, do **Go** — o mundo dos **jogos adversariais**.

Jogar bem contra um oponente racional é um problema de natureza diferente de tudo o que vimos. Não basta escolher a melhor ação para si: é preciso escolher a melhor ação **supondo que o adversário responderá com a melhor ação para ele**, que por sua vez supõe a nossa melhor resposta, e assim por diante. A decisão deixa de ser um ato isolado e vira o topo de uma **cadeia de antecipações mútuas** — "eu jogo isto, ele responde aquilo, então eu faço isto outro...". Formalizar essa cadeia, torná-la calculável e extrair dela a jogada correta é a tarefa da **busca adversarial**, e o algoritmo que a resolve de forma exata é o **Minimax**, o assunto central deste capítulo.

Fiel à estrutura da apostila, partimos do **problema** de design (jogar contra uma inteligência que planeja contra nós), construímos os **conceitos** que tornam o jogo um objeto matemático (soma zero, turnos, árvore de jogo), detalhamos o **funcionamento** do Minimax e das funções de avaliação, apresentamos a otimização indispensável — a **poda alfa-beta** — e, como conteúdo de aprofundamento, o **Monte Carlo Tree Search (MCTS)**, a técnica que redefiniu a fronteira dos jogos modernos. Aterrissamos em **exemplos** progressivos (jogo da velha, damas, xadrez), em **estudos de caso** históricos (com destaque para o **Deep Blue**), e nas **ferramentas** de implementação. Ao final, o leitor terá no repertório o pilar teórico da IA competitiva — e verá que a **função de avaliação heurística**, ideia que já encontramos no A\* (Capítulo 8) e nos mapas de influência (Capítulo 10), reaparece aqui como o coração pragmático de todo jogador artificial.

> 🕰️ **Contexto Histórico**
> O Minimax não nasceu nos jogos digitais: suas raízes estão na **teoria dos jogos** matemática. O princípio *minimax* foi formalizado por **John von Neumann** em 1928, num teorema que se tornou a pedra fundamental da análise de jogos de soma zero. Na computação, foi **Claude Shannon** quem, em 1950, no artigo seminal *"Programming a Computer for Playing Chess"*, descreveu como uma máquina poderia jogar xadrez explorando uma árvore de possibilidades e avaliando posições com uma função heurística — o texto que fundou toda a IA de jogos de tabuleiro. Pouco depois, **Alan Turing** escreveu (em papel, sem um computador capaz de executá-lo) um dos primeiros programas de xadrez, o *Turochamp*. A poda alfa-beta foi descoberta e redescoberta por vários pesquisadores ao longo das décadas de 1950 e 1960, sendo **John McCarthy** — o mesmo que cunhou o termo "Inteligência Artificial" — uma das figuras centrais em sua formalização. Essa linhagem intelectual explica por que o Minimax ocupa um lugar tão especial na história da IA: por décadas, "fazer um computador jogar xadrez bem" foi considerado um teste definitivo de inteligência de máquina — uma expectativa que culminaria, em 1997, na vitória do **Deep Blue** sobre o campeão mundial Garry Kasparov.

---

## 11.1 O problema: jogar bem contra um oponente racional

Comecemos, como sempre, pelo problema concreto. Imagine que você precisa programar a IA de um jogo de **xadrez**, ou de **damas**, ou mesmo do humilde **jogo da velha**. O NPC precisa, a cada vez que é sua vez, escolher uma jogada. Como ele decide?

A primeira tentação é aplicar o que já sabemos. Poderíamos escrever uma máquina de estados ("se o rei está em xeque, defenda; senão, ataque"), ou uma árvore de comportamento com regras táticas, ou até uma função de avaliação estilo mapa de influência que pontuasse cada casa do tabuleiro. Todas essas abordagens produzem *alguma* jogada — mas todas partilham um mesmo defeito fatal quando o adversário é competente: elas **avaliam a jogada apenas pelo que ela faz agora**, e não pelo que o oponente fará em resposta.

Considere o exemplo clássico. No xadrez, você pode capturar a dama adversária com um lance que parece magnífico — a avaliação imediata dispara, você "ganhou" a peça mais poderosa do jogo. Mas se essa captura deixa seu rei exposto a um xeque-mate na jogada seguinte, o lance é, na verdade, o **pior** possível. Uma IA que decide olhando apenas o presente cai nessa armadilha todas as vezes. O oponente racional **conta com isso**: ele oferece a dama justamente para provocar a captura que o levará à vitória. É a diferença entre jogar contra o tabuleiro e jogar contra uma **mente**.

Aqui está o cerne do problema. Nas Partes anteriores, o "melhor" de uma decisão podia ser julgado localmente: a rota mais curta é a mais curta independentemente do que o mundo faça; a cobertura mais segura é a mais segura agora. Em um jogo adversarial, **o valor de uma jogada depende inteiramente da resposta do adversário** — e essa resposta, por sua vez, é escolhida por alguém que quer o oposto do que você quer. Não existe "a melhor jogada em si"; existe apenas "a melhor jogada **dado que o outro jogará da melhor forma contra ela**".

### Tomada de decisão adversarial

Chamamos de **tomada de decisão adversarial** o processo de escolher ações em um ambiente em que outro agente, com **objetivos diretamente opostos aos nossos**, também escolhe ações que afetam o resultado. A palavra-chave é *opostos*: o adversário não é meramente imprevisível ou dinâmico — ele é **antagônico**. Cada ganho nosso é uma perda dele, e vice-versa. Isso muda a matemática da decisão de forma profunda.

Em um problema de decisão comum (como os das Partes anteriores), buscamos **maximizar** um valor: escolhemos a ação que leva ao melhor resultado. Em um problema adversarial, não podemos simplesmente maximizar, porque **metade das jogadas não é nossa**. Após nossa ação, o adversário jogará — e ele **minimizará** o nosso valor (equivalente a maximizar o dele). Portanto, para escolher bem, precisamos **antecipar essa minimização**: escolher a jogada cujo pior desdobramento (a melhor resposta do adversário) seja o menos ruim possível. Essa lógica de "maximizar supondo que o outro minimizará" é exatamente o que dá nome ao algoritmo **Mini-Max**.

> ⚠️ **Atenção**
> Um erro conceitual comum é imaginar que a IA adversarial tenta "prever o que o oponente vai fazer" no sentido psicológico — modelar seus hábitos, seus erros, sua personalidade. O Minimax **não** faz isso. Ele adota a suposição mais segura e pessimista possível: a de que o oponente jogará **perfeitamente** contra nós, sempre escolhendo a melhor resposta disponível. Essa suposição pode parecer excessivamente conservadora (o oponente real talvez erre), mas ela garante uma propriedade valiosa: a jogada escolhida é a que produz o **melhor resultado garantido no pior caso**. Contra um adversário que erra, tende a ir ainda melhor. Modelar as fraquezas específicas de um oponente é um tema avançado (*opponent modeling*), mas o alicerce é sempre o jogo contra o adversário perfeito.

### Ambientes cooperativos, reativos e competitivos

Para situar com precisão onde a busca adversarial se aplica, é útil distinguir três grandes tipos de ambiente em que uma IA de jogo pode operar. A distinção não é meramente acadêmica: ela determina **qual família de técnicas** faz sentido usar.

**Ambiente cooperativo.** Outros agentes existem, mas têm objetivos **alinhados** aos do nosso agente — ou pelo menos não conflitantes. O companheiro controlado pela IA em um jogo de ação (como Ellie em *The Last of Us*, que veremos na Parte VII) trabalha *a favor* do jogador. Aqui, o raciocínio é de coordenação e apoio, não de antecipação hostil. Técnicas de decisão (Parte II) e de movimento (Parte III) bastam.

**Ambiente reativo (neutro/hostil, mas não estratégico).** O ambiente pode ser perigoso e mutável, mas **não planeja contra o agente de forma estratégica**. Um inimigo de um jogo de ação que patrulha, persegue e atira é hostil, porém suas decisões são governadas por regras reativas (uma FSM, uma behavior tree), não por uma antecipação da árvore completa de jogadas do jogador. É o território das Partes II a IV: o agente reage a estímulos e avalia o estado presente, mas não constrói uma cadeia de "eu faço, ele responde, eu respondo".

**Ambiente competitivo (adversarial).** Existe um oponente com objetivos **diametralmente opostos**, que **escolhe suas ações antecipando as nossas** para nos prejudicar ao máximo. É o xadrez, as damas, o Go, o jogo da velha — e é o único ambiente que exige, genuinamente, busca adversarial. Aqui, e somente aqui, o Minimax é a ferramenta certa.

> 🎮 **Na Prática**
> A maioria dos jogos comerciais de ação, tiro e RPG vive no ambiente **reativo**, não no competitivo — e é por isso que o Minimax raramente aparece neles. Um inimigo de *Halo* ou de *F.E.A.R.* não precisa construir uma árvore de jogo; ele precisa *parecer* inteligente reagindo bem ao jogador, o que se resolve com behavior trees, GOAP e mapas de influência. O Minimax brilha em um nicho específico mas importante: jogos de **tabuleiro e estratégia por turnos**, com regras discretas, jogadas alternadas e resultado claro (vitória/derrota). Reconhecer se o seu jogo é reativo ou competitivo é o primeiro passo — aplicar Minimax a um jogo de ação em tempo real é usar a ferramenta errada.

> ❌ **Erro Comum**
> Confundir "ambiente difícil" com "ambiente adversarial". Um jogo de plataforma com física traiçoeira, inimigos numerosos e armadilhas é *difícil*, mas não é *adversarial* no sentido técnico: nada ali constrói uma estratégia antecipando suas jogadas futuras. A busca adversarial só se justifica quando há, de fato, um **jogador oponente racional** alternando decisões com o agente — não quando o jogo é apenas exigente.

[DIAGRAMA]
Título: Três tipos de ambiente para a IA de jogo
Objetivo pedagógico: Fixar a distinção entre ambientes cooperativo, reativo e competitivo, deixando claro que só o último exige busca adversarial.
Descrição detalhada: Três painéis lado a lado. Painel 1 ("Cooperativo"): dois agentes (um "IA" e um "jogador") com setas de objetivo apontando para o MESMO alvo; legenda "objetivos alinhados — coordenação". Painel 2 ("Reativo"): um agente "IA" com uma seta que sai de um sensor (olho) e volta para uma ação, representando o laço sentir→pensar→agir; o "mundo" é mostrado como engrenagens neutras que mudam, mas sem uma seta de "planejamento contra a IA"; legenda "mundo hostil, mas não estratégico — regras reativas (FSM/BT)". Painel 3 ("Competitivo"): dois agentes frente a frente com setas de objetivo apontando em SENTIDOS OPOSTOS, e uma cadeia de balões "eu jogo → ele responde → eu respondo" entre eles; legenda "objetivos opostos — busca adversarial (Minimax)". Destacar visualmente o painel 3 como o foco do capítulo.
Elementos obrigatórios: os três rótulos; setas de objetivo alinhadas (painel 1), laço reativo (painel 2) e setas opostas + cadeia de antecipação (painel 3); indicação de qual técnica cabe em cada caso.
[/DIAGRAMA]

---

## 11.2 Fundamentos

Antes de apresentar o algoritmo, precisamos construir o vocabulário e os objetos matemáticos que ele manipula. O Minimax não opera sobre "o jogo" no sentido informal; ele opera sobre uma **modelagem formal** do jogo. Esta seção constrói, peça por peça, essa modelagem — e nenhum conceito das seções seguintes será usado antes de ser definido aqui.

### Jogos de soma zero

O primeiro conceito é a razão pela qual "eu ganho ⟺ o outro perde" é matematicamente verdadeiro. Um **jogo de soma zero** é aquele em que o ganho de um jogador é **exatamente** a perda do outro: se somarmos os resultados dos dois lados, o total é sempre o mesmo (convencionalmente, zero). Não há como ambos ganharem, nem como ambos perderem; o que é bom para um é, na mesma medida, ruim para o outro.

O xadrez, as damas, o jogo da velha e o Go são jogos de soma zero: cada partida termina com uma vitória (para um), uma derrota (para o outro) ou um empate (neutro para ambos). Se atribuirmos +1 à vitória de um jogador, ele terá +1 e o adversário −1; a soma é zero. Essa propriedade é o que autoriza a simplificação central do Minimax: **basta uma única função de valor**, medida do ponto de vista de um dos jogadores. Como o ganho de um é a perda do outro, aquilo que um jogador tenta **maximizar**, o outro tenta **minimizar** — não precisamos de dois números, apenas de um, e de dois objetivos opostos sobre ele.

> ⚠️ **Atenção**
> Nem todo jogo é de soma zero. Jogos em que os jogadores podem cooperar para um ganho mútuo (soma não-zero, como muitos jogos econômicos ou de negociação) exigem outras abordagens da teoria dos jogos e **não** se encaixam diretamente no Minimax clássico. O Minimax, na forma que estudaremos, pressupõe soma zero e dois jogadores. Extensões para mais jogadores ou soma não-zero existem, mas são conteúdo avançado; aqui, mantemo-nos no caso clássico, que cobre a imensa maioria dos jogos de tabuleiro tradicionais.

### Jogos por turnos

O segundo pilar é a estrutura **por turnos** (ou *sequencial*): os jogadores **alternam** decisões, e cada um enxerga o resultado da jogada do outro antes de decidir a sua. Primeiro joga A, depois B, depois A novamente, e assim por diante. Essa alternância é o que permite representar o jogo como uma sequência ordenada de decisões — e, portanto, como uma **árvore** (que veremos a seguir).

Além de ser por turnos, o Minimax clássico pressupõe que o jogo seja de **informação perfeita**: ambos os jogadores conhecem completamente o estado do jogo a todo momento — o tabuleiro está aberto, não há cartas escondidas nem dados a lançar. O xadrez é de informação perfeita; o pôquer não é (há cartas ocultas e acaso). Jogos com informação oculta ou elementos aleatórios exigem variantes (como o *Expectiminimax*, que adiciona nós de "chance"), mencionadas adiante, mas o algoritmo base assume informação perfeita e determinismo.

> 🎲 **Curiosidade**
> A combinação "dois jogadores, soma zero, informação perfeita, determinístico, por turnos" descreve uma família de jogos que a teoria chama de **jogos combinatórios**. É uma família matematicamente elegante: para qualquer jogo desse tipo, existe, em teoria, uma **estratégia ótima** perfeitamente determinada. O jogo da velha, por exemplo, está completamente "resolvido" — com jogo perfeito dos dois lados, sempre termina em empate. Damas também foi resolvido (em 2007, após 18 anos de computação): com jogo perfeito, é empate. O xadrez e o Go, embora igualmente determinados em teoria, são grandes demais para serem resolvidos por força bruta — e é justamente essa intratabilidade que torna a heurística indispensável, como veremos.

### A árvore de jogo

Chegamos à estrutura central de todo o capítulo. Uma **árvore de jogo** (*game tree*) é a representação de **todas as sequências possíveis de jogadas** a partir de uma posição, organizadas hierarquicamente. É o objeto sobre o qual o Minimax opera — assim como o A\* opera sobre o grafo de navegação, o Minimax opera sobre a árvore de jogo.

A construção é natural e decorre da estrutura por turnos:

- A **raiz** da árvore é o **estado atual** do jogo (a posição de que partimos, de quem é a decisão).
- Cada **ramo** (aresta) saindo de um nó representa uma **jogada legal** disponível naquele estado.
- Cada **nó filho** é o **novo estado** resultante de aplicar aquela jogada.
- Os **níveis** da árvore alternam entre os dois jogadores: se a raiz é a vez do jogador A, o primeiro nível de filhos representa as respostas de B, o nível seguinte as réplicas de A, e assim por diante.
- As **folhas** (nós terminais) são os estados em que o jogo **acaba** — vitória, derrota ou empate.

Vamos aos elementos que compõem essa árvore, um a um, pois cada um será um parâmetro do algoritmo.

**Estados.** Um **estado** é uma configuração completa do jogo em um dado momento — no xadrez, a posição exata de todas as peças no tabuleiro, de quem é a vez, e informações auxiliares (direitos de roque, etc.). Cada nó da árvore é um estado. O estado da raiz é o presente; os estados das folhas são finais.

**Ações (jogadas).** Uma **ação** é uma jogada legal que transforma um estado em outro. As ações são os ramos da árvore. O conjunto de ações legais em um estado define quantos filhos aquele nó terá. Uma função de *sucessores* (ou *movimentos legais*) gera, dado um estado, todos os estados-filho alcançáveis.

**Profundidade.** A **profundidade** (*depth*) de um nó é o número de jogadas (ramos) que o separam da raiz. A raiz tem profundidade 0; seus filhos, profundidade 1; e assim por diante. Cada nível de profundidade corresponde a **meia-rodada** — uma jogada de um único jogador. Na literatura de jogos, uma jogada de um jogador é chamada de ***ply***: profundidade 2 significa "dois plies", isto é, uma jogada minha e a resposta do adversário. A profundidade é crucial porque, como veremos, quase nunca poderemos explorar a árvore até as folhas — teremos de **parar** em uma profundidade limite.

**Fator de ramificação.** O **fator de ramificação** (*branching factor*), geralmente denotado por *b*, é o número **médio** de jogadas legais disponíveis por estado — ou seja, quantos filhos cada nó tem, em média. É o parâmetro que governa a **explosão** do tamanho da árvore. No jogo da velha, *b* começa em 9 e diminui; nas damas, *b* ≈ 8; no xadrez, *b* ≈ 35; no Go, *b* ≈ 250. O número de nós numa árvore de profundidade *d* cresce aproximadamente como *b^d* — e é essa função exponencial que decidirá tudo sobre a viabilidade do Minimax.

> 🎮 **Na Prática**
> Vale sentir a magnitude do problema com números. No xadrez, com *b* ≈ 35, olhar apenas **4 plies** à frente (duas jogadas de cada lado) já significa examinar cerca de 35⁴ ≈ 1,5 milhão de posições. Olhar 8 plies — ainda muito pouco para o xadrez de alto nível — significa 35⁸ ≈ 2,25 **trilhões** de posições. O número total de partidas de xadrez possíveis foi estimado por Shannon em cerca de 10¹²⁰ — um número maior que a quantidade de átomos no universo observável, conhecido como o **número de Shannon**. Explorar a árvore completa é, e sempre será, impossível. Toda a engenharia do Minimax prático gira em torno de **como decidir bem sem explorar tudo**.

**Utilidade.** A **utilidade** (*utility*, ou *payoff*) é o valor numérico atribuído a um estado **terminal** (folha), do ponto de vista de um dos jogadores (convencionalmente, o jogador MAX). É o que o jogo "vale" quando termina. A convenção mais comum é simples: **+1** para uma vitória de MAX, **−1** para uma derrota de MAX (vitória de MIN), **0** para empate. Em jogos onde a margem importa, a utilidade pode assumir outros valores, mas a ideia é sempre a mesma: a utilidade mede, objetivamente, **quão bom é um final de jogo**. É a partir das utilidades das folhas que o Minimax deriva o valor de todos os nós internos.

**Função de avaliação.** Aqui está o conceito que faz a ponte entre a teoria e a prática — e que reencontraremos como protagonista na Seção 11.3.4. Como acabamos de ver, é impossível explorar a árvore até as folhas em jogos reais. Então paramos em uma profundidade limite, em nós que **não são terminais** — o jogo ainda não acabou ali. Mas o Minimax precisa de um número para esses nós, assim como precisa da utilidade para as folhas. Esse número é fornecido pela **função de avaliação** (*evaluation function*, ou **função heurística de avaliação**): uma estimativa de **quão favorável** é uma posição não-terminal para o jogador MAX, sem saber quem de fato venceria. No xadrez, uma função de avaliação típica soma o valor material das peças (dama = 9, torre = 5, bispo/cavalo = 3, peão = 1), ajustado por fatores posicionais (controle do centro, segurança do rei, estrutura de peões). É, em essência, a mesma ideia de **heurística** que vimos no A\* (uma estimativa que substitui um cálculo exato inviável) e nos mapas de influência (um número que resume a qualidade de uma posição) — agora aplicada a decidir *quão boa é esta posição de jogo*.

> ✅ **Boa Prática**
> Guarde a distinção entre **utilidade** e **função de avaliação**, porque confundi-las é fonte de muitos erros. A **utilidade** é *exata* e se aplica a estados **terminais**: o jogo acabou, sabemos o resultado com certeza. A **função de avaliação** é *estimada* e se aplica a estados **não-terminais**: o jogo continua, e apenas *chutamos* quem está melhor. Um Minimax que explorasse até o fim usaria só utilidades e jogaria perfeitamente; um Minimax prático, limitado em profundidade, depende crucialmente da qualidade de sua função de avaliação — e é por isso que, em jogos reais, **a heurística importa tanto quanto o algoritmo**.

[DIAGRAMA]
Título: Anatomia de uma árvore de jogo
Objetivo pedagógico: Consolidar visualmente todos os elementos definidos na Seção 11.2 (estado, ação, profundidade, ramificação, folha, utilidade) em uma única figura de referência, usando um exemplo pequeno.
Descrição detalhada: Uma árvore de três níveis desenhada de cima para baixo, com um jogo simples e abstrato. No topo, um único nó rotulado "RAIZ — estado atual (vez de MAX), profundidade 0". Dele saem 3 ramos rotulados "ações (jogadas legais)", levando a 3 nós de profundidade 1 (rotulados "vez de MIN"). De cada um desses, saem 2 ramos até nós de profundidade 2, alguns marcados como folhas terminais com valores de utilidade (+1, 0, −1) e outros marcados com um símbolo de "?" indicando nós não-terminais que precisariam de função de avaliação se a busca parasse ali. Anotações laterais: uma chave indicando "fator de ramificação b ≈ 3 no topo, 2 no nível seguinte"; uma régua vertical à esquerda marcando "profundidade 0, 1, 2 (plies)"; destaque nas folhas: "utilidade: +1 vitória MAX, 0 empate, −1 derrota MAX".
Elementos obrigatórios: raiz identificada como estado atual; ramos rotulados como ações; níveis alternando MAX/MIN; folhas terminais com utilidade; pelo menos um nó não-terminal marcado com "?"; escala de profundidade/plies; indicação do fator de ramificação.
[/DIAGRAMA]

Com esses fundamentos — soma zero, turnos, árvore de jogo, estados, ações, profundidade, ramificação, utilidade e função de avaliação — temos todo o vocabulário necessário. Estamos prontos para o algoritmo que dá nome ao capítulo.

---

## 11.3 Funcionamento do Minimax

### O princípio do algoritmo

O Minimax responde a uma única pergunta: **dada a posição atual, qual jogada devo fazer para obter o melhor resultado garantido, supondo que o adversário jogará da melhor forma possível contra mim?**

A ideia é de uma simplicidade elegante. Se eu pudesse construir a árvore de jogo inteira até as folhas, eu saberia o resultado (a utilidade) de cada final possível. O problema é que os finais estão lá embaixo, na base da árvore, e a decisão que preciso tomar está aqui em cima, na raiz. O Minimax é o mecanismo que **transporta a informação das folhas para a raiz** — ele "propaga" os valores de baixo para cima, e ao chegar à raiz, sei exatamente qual jogada leva ao melhor desfecho garantido.

A propagação obedece a uma regra que reflete diretamente a natureza adversarial do jogo. Como estamos em soma zero, medimos tudo do ponto de vista de **um** jogador — chamado **MAX**, porque ele quer **maximizar** o valor. O adversário é chamado **MIN**, porque, ao maximizar o *seu* próprio ganho, ele necessariamente **minimiza** o valor de MAX. A regra de propagação é:

- Em um nó onde é a vez de **MAX**, o valor do nó é o **maior** valor entre seus filhos (MAX escolherá a melhor jogada para si).
- Em um nó onde é a vez de **MIN**, o valor do nó é o **menor** valor entre seus filhos (MIN escolherá a melhor jogada para si, que é a pior para MAX).

Ou seja: cada jogador, em seu nível, escolhe o filho que lhe é mais favorável. MAX puxa o valor para cima; MIN puxa para baixo. Alternando essas duas operações camada por camada, dos nós terminais até a raiz, o algoritmo calcula o **valor minimax** de cada nó — e o valor da raiz é o melhor resultado que MAX pode garantir contra um MIN perfeito.

### Jogadores MAX e MIN

Fixemos bem os dois papéis, porque toda a mecânica gira em torno deles. **MAX** é o jogador para quem estamos decidindo — o "nós" da situação, aquele cujo interesse o algoritmo defende. Ele quer o **maior** valor de utilidade possível. **MIN** é o adversário — aquele que quer o **menor** valor de utilidade de MAX (o que, em soma zero, equivale a querer o maior valor para si mesmo).

Os dois jogadores se **alternam** por profundidade na árvore. Se a raiz é a vez de MAX (profundidade 0, um nó MAX), então a profundidade 1 é composta de nós MIN, a profundidade 2 de nós MAX, e assim por diante. Chamamos essas alternâncias de **camadas MAX e MIN**, tema da Subseção 11.3.1.

É crucial entender que ambos jogam **da melhor forma possível dentro do modelo**. O Minimax não supõe que MIN erre; ao contrário, supõe que MIN é tão competente quanto MAX. Essa é a suposição pessimista que garante a solidez da decisão: se a jogada é boa contra o melhor MIN possível, ela é ao menos tão boa contra qualquer MIN real.

### Propagação dos valores na árvore

Vejamos a propagação passo a passo, com um exemplo pequeno e concreto. Considere uma árvore de profundidade 2 (dois plies): a raiz é MAX, tem três jogadas possíveis (A, B, C); cada uma leva a um nó MIN; e cada nó MIN tem dois filhos terminais com utilidades conhecidas.

Suponha estas utilidades nas folhas (da esquerda para a direita):

- Após jogada **A**: folhas com valores **3** e **5**.
- Após jogada **B**: folhas com valores **6** e **2**.
- Após jogada **C**: folhas com valores **1** e **8**.

O cálculo, sempre **de baixo para cima**:

1. **Nível MIN (profundidade 1).** Cada nó MIN escolhe o **menor** de seus filhos:
   - Nó MIN após A = min(3, 5) = **3**.
   - Nó MIN após B = min(6, 2) = **2**.
   - Nó MIN após C = min(1, 8) = **1**.
2. **Nível MAX (raiz, profundidade 0).** A raiz MAX escolhe o **maior** entre os valores dos nós MIN:
   - Raiz = max(3, 2, 1) = **3**, obtido pela jogada **A**.

A conclusão do Minimax é: **jogue A**, que garante um resultado de pelo menos 3. Repare no raciocínio adversarial embutido. A jogada B leva a uma folha de valor 6 — o **melhor** valor da árvore inteira! Um algoritmo ingênuo, olhando o "melhor caso", escolheria B. Mas o Minimax **não** escolhe B, porque sabe que, após B, é a vez de MIN — e MIN jamais permitiria o 6; ele escolheria o 2. A jogada B, na verdade, **garante apenas 2**. A jogada C garante apenas 1 (MIN escolhe o 1, não o tentador 8). Só a jogada A garante 3. O Minimax escolhe A não por ser a mais brilhante no melhor caso, mas por ter o **melhor pior caso** — a essência do pensamento adversarial.

> ❌ **Erro Comum**
> O engano mais frequente ao aprender Minimax é olhar as folhas e escolher a jogada que leva ao **maior valor absoluto** da árvore. Isso ignora que, entre você e aquela folha tentadora, há uma ou mais jogadas do adversário, que **nunca** o deixarão chegar lá se isso o beneficiar. Sempre pergunte, para cada jogada sua: "qual é o **pior** que pode me acontecer depois dela, se o oponente jogar perfeitamente?". A resposta a essa pergunta é o valor real da jogada — e é isso que o Minimax calcula.

### Escolha ótima e construção da árvore de busca

O resultado do Minimax é duplamente informativo. Ele produz não apenas o **valor** da posição (o melhor resultado garantido — no exemplo, 3), mas também a **jogada ótima** que o realiza (jogar A). Na prática, é essa jogada que interessa: o algoritmo é chamado uma vez por turno, calcula o valor minimax de cada jogada legal a partir da posição atual, e o agente executa aquela de maior valor.

A **construção da árvore de busca** é feita, na implementação usual, de forma **recursiva e sob demanda**. O algoritmo não constrói a árvore inteira na memória primeiro para só depois avaliá-la — isso seria proibitivo. Em vez disso, ele explora a árvore em **profundidade** (uma busca em profundidade, *depth-first*): desce por um ramo até uma folha (ou até o limite de profundidade), obtém seu valor, volta, desce pelo próximo ramo, e assim por diante, combinando os valores (com max ou min conforme a camada) à medida que retorna. Isso mantém na memória apenas o **caminho atual** da raiz até o ponto explorado, e não a árvore toda — uma economia de memória decisiva, à qual voltaremos ao discutir vantagens e limitações.

O pseudocódigo a seguir expressa o algoritmo completo. Note que ele é notavelmente curto — a elegância do Minimax está justamente na concisão com que captura um raciocínio tão profundo:

```
função minimax(estado, profundidade, éMAX):
    // Casos-base: fim do jogo ou limite de profundidade atingido
    se estado é terminal:
        retornar utilidade(estado)
    se profundidade == 0:
        retornar avaliação(estado)      // função heurística (ver 11.3.4)

    se éMAX:
        valor = −infinito
        para cada jogada em ações(estado):
            filho = aplicar(estado, jogada)
            valor = max(valor, minimax(filho, profundidade−1, falso))
        retornar valor
    senão:  // é MIN
        valor = +infinito
        para cada jogada em ações(estado):
            filho = aplicar(estado, jogada)
            valor = min(valor, minimax(filho, profundidade−1, verdadeiro))
        retornar valor
```

Para obter a **jogada** (e não só o valor) na raiz, guarda-se qual jogada produziu o melhor valor no laço do topo. Observe os três ingredientes que já conhecemos aparecendo no código: os **casos-base** usam a **utilidade** (folha terminal) ou a **função de avaliação** (limite de profundidade); a alternância **MAX/MIN** aparece como o par max/min; e a **construção sob demanda** é a própria recursão `aplicar(...)` seguida da chamada recursiva.

[DIAGRAMA]
Título: Propagação de valores no Minimax (árvore de 2 plies)
Objetivo pedagógico: Mostrar, com o exemplo numérico da seção, como os valores sobem das folhas até a raiz, alternando min e max, e por que a jogada A é escolhida apesar de B conter a maior folha.
Descrição detalhada: Árvore de três níveis. Topo: nó MAX (triângulo apontando para cima) rotulado "raiz — MAX". Três ramos rotulados A, B, C descem para três nós MIN (triângulos apontando para baixo). Cada nó MIN tem dois filhos-folha: sob A as folhas 3 e 5; sob B as folhas 6 e 2; sob C as folhas 1 e 8. Anotar em cada nó MIN o valor calculado (A→3, B→2, C→1) com uma seta indicando qual folha foi escolhida (a menor). No topo, anotar a raiz = 3, com a seta destacando o ramo A como a jogada escolhida. Destacar visualmente (por exemplo, com um X ou sombreado) a folha 6 sob B, com a legenda "MIN nunca deixa MAX chegar aqui". Usar cores distintas para camadas MAX (azul) e MIN (vermelho).
Elementos obrigatórios: triângulos MAX/MIN distinguindo as camadas; as seis folhas com os valores dados; os valores propagados nos nós MIN e na raiz; a jogada A destacada; a folha 6 marcada como inalcançável.
[/DIAGRAMA]

### 11.3.1 Camadas MAX e MIN

Detalhemos a estrutura em camadas, pois é ela que organiza a alternância de objetivos. Uma **camada** (ou *ply*) é um nível de profundidade da árvore, associado a **um único jogador**. As camadas se alternam rigidamente: MAX, MIN, MAX, MIN, ... A cada camada, o jogador daquele nível aplica sua operação característica (máximo para MAX, mínimo para MIN) sobre os valores vindos da camada de baixo.

Essa alternância traduz, de forma mecânica, o diálogo adversarial. A camada MAX pergunta "qual a **melhor** coisa que **eu** posso fazer?"; a camada MIN logo abaixo pergunta "e qual a **pior** coisa que o **oponente** pode me fazer em resposta?". Uma decisão de qualidade só emerge quando as duas perguntas são consideradas em conjunto, camada após camada — o que é exatamente o que a propagação alternada realiza.

Um ponto prático importante: uma **rodada completa** do jogo (uma jogada de cada lado) corresponde a **duas** camadas — um ply de MAX e um ply de MIN. Quando dizemos que um programa de xadrez "pensa 10 jogadas à frente", em geral se referem a 10 plies, isto é, 5 rodadas completas. A confusão entre "jogadas" (rodadas) e "plies" (meias-rodadas) é frequente; adotaremos sempre **ply = camada = uma jogada de um jogador**, coerente com a Seção 11.2.

> ✅ **Boa Prática**
> Ao desenhar ou traçar manualmente uma árvore de jogo, marque cada camada com o jogador correspondente (MAX ou MIN) **antes** de começar a propagar valores. A causa número um de erros em exercícios de Minimax é aplicar max onde deveria ser min (ou vice-versa) porque se perdeu a conta de qual camada é de quem. Uma convenção visual simples — triângulos para cima (MAX) e para baixo (MIN), ou cores distintas — previne quase todos esses erros.

### 11.3.2 Profundidade da busca

Como já antecipamos, explorar a árvore até as folhas terminais é inviável em quase todos os jogos interessantes. A solução prática é impor um **limite de profundidade**: o algoritmo desce apenas até uma profundidade máxima *d* (medida em plies) e, ao atingi-la, **para** — mesmo que o jogo não tenha acabado ali. Nesses nós de parada, usa-se a **função de avaliação** (11.3.4) para estimar o valor da posição, no lugar da utilidade exata que teríamos numa folha real.

A profundidade da busca é o **principal controle de qualidade e custo** do Minimax. Quanto maior *d*, mais longe o algoritmo "enxerga" e melhores tendem a ser suas decisões — mas o custo cresce exponencialmente (aproximadamente *b^d*). Dobrar a profundidade não dobra o trabalho: multiplica-o por um fator enorme. Escolher *d* é, portanto, um **trade-off** direto entre força de jogo e tempo disponível por jogada — a mesma lógica de orçamento de quadro que atravessa toda a apostila. Em um jogo de xadrez por turnos, pode-se dar segundos ou minutos por lance; em um jogo em tempo real, o orçamento é de milissegundos, o que limita drasticamente *d*.

Um refinamento essencial usado na prática é o **aprofundamento iterativo** (*iterative deepening*): em vez de fixar *d* de antemão, executa-se o Minimax repetidamente com profundidades crescentes (1, 2, 3, ...) até esgotar o tempo disponível, guardando sempre o melhor resultado da última profundidade completada. Isso garante que o programa sempre tenha uma jogada pronta quando o relógio acabar, e — surpreendentemente — costuma ser **mais** eficiente, porque os resultados das buscas rasas ajudam a **ordenar as jogadas** das buscas profundas (assunto da poda alfa-beta, Seção 11.4).

### 11.3.3 Horizonte de busca

O limite de profundidade cria um fenômeno importante e às vezes traiçoeiro: o **horizonte de busca**. O horizonte é a fronteira além da qual o algoritmo **não enxerga** — tudo o que aconteceria depois da profundidade *d* é invisível para ele. E aqui mora um perigo sutil, conhecido como **efeito horizonte** (*horizon effect*).

O efeito horizonte ocorre quando algo decisivo acontece **logo além** do limite de profundidade, e o algoritmo, por não vê-lo, toma uma decisão ruim ou se ilude. O exemplo clássico no xadrez: a IA está prestes a perder uma peça de forma inevitável. Se essa perda ocorreria dentro do horizonte, o algoritmo a "vê" e tenta evitá-la. Mas se ela está a um lance **além** do horizonte, o algoritmo pode fazer lances inúteis que apenas **empurram** a perda para depois do horizonte — como dar xeques desesperados que atrasam o inevitável. Do ponto de vista do algoritmo, a perda "desapareceu" (saiu do campo de visão), quando na verdade apenas foi adiada para uma casa que ele não olha. Ele troca uma posição ruim por uma pior, iludido pela sua própria miopia.

> ⚠️ **Atenção**
> O efeito horizonte é uma limitação **fundamental** de qualquer busca com profundidade limitada — não um bug que se conserte com código mais esperto, mas uma consequência inevitável de não poder olhar até o fim. Mitiga-se, não se elimina. As principais mitigações são: a **busca de quietude** (*quiescence search*), que estende a busca em posições "instáveis" (com capturas ou xeques pendentes) até chegar a uma posição "calma" antes de avaliar; e o próprio **aprofundamento iterativo**, que aumenta o horizonte quando há tempo. Reconhecer o efeito horizonte é importante porque ele explica muitos comportamentos aparentemente irracionais de IAs de jogo de tabuleiro mais simples.

[IMAGEM NECESSÁRIA]
Título: O efeito horizonte no xadrez
Objetivo didático: Ilustrar, de forma intuitiva, por que uma busca com profundidade limitada pode ser "iludida" por eventos que ocorrem logo além do seu horizonte, motivando a busca de quietude.
Descrição: Uma faixa de tempo/profundidade horizontal representando os plies da busca (1, 2, 3, ... até d). Dentro do horizonte, jogadas normais. Exatamente na fronteira "d", uma linha tracejada vermelha rotulada "HORIZONTE". Logo depois da linha, um ícone de peça capturada (uma dama tombada) rotulado "perda inevitável — invisível para a busca". Setas mostrando a IA fazendo "lances de adiamento" (pequenos xeques) que empurram a captura para além da linha, com a legenda "a IA acredita ter evitado a perda, mas apenas a escondeu além do horizonte".
Tipo: Ilustração/esquema conceitual (pode combinar um mini-diagrama de tabuleiro com a linha do tempo de busca).
Como produzir: Diagrama vetorial simples com uma régua de profundidade, uma linha de horizonte destacada e ícones de xadrez; opcionalmente sobrepor a um pequeno diagrama de posição de xadrez real que exemplifique o adiamento.
Legenda sugerida: "Efeito horizonte: o que a busca não alcança, ela não evita — apenas empurra para fora do próprio campo de visão."
[/IMAGEM NECESSÁRIA]

### 11.3.4 Funções heurísticas de avaliação

Chegamos ao componente que, na prática, **decide a força de um jogador artificial** tanto quanto o algoritmo de busca. A **função heurística de avaliação** é a peça que preenche a lacuna deixada pela profundidade limitada: quando a busca para em um nó não-terminal, é ela que responde "quão boa é esta posição para MAX?", produzindo o número que sobe pela árvore no lugar da utilidade.

Já vimos a ideia de heurística duas vezes na apostila. No **A\*** (Capítulo 8), a heurística `h(n)` estimava o custo restante até o destino, substituindo um cálculo exato inviável. Nos **mapas de influência** (Capítulo 10), um campo de valores resumia a qualidade tática de cada região. A função de avaliação do Minimax é a mesma ideia aplicada a um novo domínio: **estimar o valor de uma posição de jogo sem calcular seu desfecho exato**. É a heurística reaparecendo, mais uma vez, como a ferramenta que torna tratável o intratável.

Como se constrói uma função de avaliação? Ela é tipicamente uma **soma ponderada de características** (*features*) da posição — quantidades mensuráveis que se acredita correlacionarem-se com a probabilidade de vitória. No xadrez, o exemplo canônico:

- **Material:** soma dos valores das peças de MAX menos as de MIN, com os pesos clássicos (peão = 1, cavalo = 3, bispo = 3, torre = 5, dama = 9). É de longe a característica mais importante.
- **Posição:** controle das casas centrais, atividade das peças, segurança do rei, estrutura de peões (peões dobrados, isolados, passados), mobilidade (número de lances legais).

A função combina essas características em um único número: `avaliação = w₁·material + w₂·centro + w₃·segurançaDoRei + ...`, onde os pesos *wᵢ* refletem a importância relativa de cada fator. Uma posição com avaliação alta é favorável a MAX; baixa (ou negativa), favorável a MIN.

A **qualidade da heurística influencia diretamente o desempenho** — e este é o ponto central da subseção. Duas IAs com o **mesmo** algoritmo de busca e a **mesma** profundidade podem jogar em níveis completamente diferentes, dependendo apenas de suas funções de avaliação. Uma heurística que só conta material joga razoavelmente, mas cai em armadilhas posicionais; uma que pondera bem material *e* posição joga muito mais forte. De forma geral:

- Uma **heurística melhor** permite jogar bem com **menos profundidade** — ela extrai mais informação de cada posição avaliada, compensando o horizonte curto.
- Uma **heurística ruim** desperdiça a busca: mesmo olhando fundo, o algoritmo propaga estimativas erradas e escolhe mal.

Há aqui um **trade-off** interno delicioso. Uma heurística mais sofisticada avalia melhor cada posição, mas **custa mais** por avaliação — e, como o Minimax avalia milhões de posições, uma heurística lenta reduz a profundidade alcançável no tempo disponível. Assim, a decisão de projeto não é "a heurística mais precisa possível", mas "a melhor relação entre **precisão** e **custo**": às vezes uma heurística mais simples e rápida, avaliando mais posições, joga melhor que uma sofisticada e lenta. É o mesmo dilema precisão-versus-custo que encontramos em toda a apostila.

> 🏭 **Na Indústria**
> A busca por boas funções de avaliação foi, por décadas, uma arte manual: mestres de xadrez e programadores ajustavam pesos à mão, testando exaustivamente. A virada moderna foi **aprender** a função de avaliação a partir de dados, em vez de programá-la. Programas de xadrez de ponta como o **Stockfish** passaram a usar redes neurais (a chamada arquitetura **NNUE**, *Efficiently Updatable Neural Network*) como função de avaliação, treinadas sobre milhões de posições — mas mantendo, note bem, a **busca alfa-beta** por baixo. O **AlphaZero**, do outro lado, aprendeu tanto a avaliação quanto a política de jogadas por auto-jogo com redes neurais profundas, combinadas com **MCTS** (Seção 11.5). Esses sistemas mostram que "algoritmo de busca" e "função de avaliação" são componentes **separáveis**: pode-se manter o Minimax/alfa-beta e trocar apenas a avaliação por uma aprendida — uma ponte direta com as técnicas de aprendizado da Parte VI.

[DIAGRAMA]
Título: Como a função de avaliação preenche o horizonte
Objetivo pedagógico: Mostrar que, na busca limitada, os nós da fronteira de profundidade recebem valores da função heurística (não da utilidade), e que esses valores são o que sobe pela árvore.
Descrição detalhada: Uma árvore de jogo desenhada até a profundidade d=3. Os nós dos níveis 0 a 2 são internos (MAX/MIN alternando). Na profundidade 3 (a fronteira), em vez de folhas terminais com utilidade, mostram-se nós rotulados "não-terminal — jogo continua", cada um recebendo um valor de uma "caixa" lateral rotulada "FUNÇÃO DE AVALIAÇÃO" que lista features (material, centro, segurança do rei) sendo somadas com pesos. Setas dessa caixa apontando para os nós da fronteira, atribuindo-lhes valores numéricos (ex.: +2, −1, +3...). Depois, setas de propagação subindo (min/max) até a raiz. Contraste: em um canto, um pequeno nó marcado "folha REAL (xeque-mate) → utilidade +∞", para distinguir avaliação de utilidade.
Elementos obrigatórios: fronteira de profundidade d recebendo valores da função de avaliação; a caixa de features com soma ponderada; propagação min/max até a raiz; distinção visual entre "avaliação" (fronteira) e "utilidade" (folha terminal real).
[/DIAGRAMA]

---

## 11.4 Poda Alfa-Beta

### Motivação

O Minimax é correto, mas caro. Como vimos, ele examina aproximadamente *b^d* nós — uma explosão exponencial que limita severamente a profundidade alcançável. A pergunta natural é: **precisamos mesmo examinar todos os nós?** A resposta, felizmente, é **não** — e a **poda alfa-beta** (*alpha-beta pruning*) é a técnica que identifica e descarta os ramos que não podem, sob nenhuma circunstância, afetar a decisão final.

A intuição é simples e vale a pena internalizá-la antes da mecânica. Imagine que você está avaliando suas jogadas uma a uma. Já analisou a jogada A e sabe que ela **garante** um resultado de pelo menos 3 (é o seu "melhor até agora"). Agora começa a analisar a jogada B e, logo na primeira resposta do adversário, descobre que B permite ao adversário forçar um resultado de **2**. Nesse instante, você já pode **parar de analisar B**: como o adversário escolherá a *pior* resposta para você, e ele já tem à disposição uma que dá 2 (pior que os 3 garantidos por A), a jogada B **nunca** será melhor que A, não importa o que haja nos outros ramos de B. Analisar o resto de B é trabalho desperdiçado. **Você "poda" esse ramo.**

Essa é toda a ideia: **se já sabemos que um ramo não pode superar uma alternativa já encontrada, não precisamos explorá-lo até o fim**. A poda alfa-beta formaliza esse raciocínio e o aplica sistematicamente durante a busca.

### Funcionamento

A formalização usa dois valores que dão nome à técnica, carregados ao longo da busca em profundidade:

- **α (alfa):** o **melhor valor já garantido para MAX** até o momento, ao longo do caminho da raiz até o nó atual. É um **limite inferior** — MAX já sabe que consegue *pelo menos* α. Começa em −∞ e só cresce.
- **β (beta):** o **melhor valor já garantido para MIN** até o momento, ao longo do mesmo caminho. É um **limite superior** — MIN já sabe que consegue limitar MAX a *no máximo* β. Começa em +∞ e só diminui.

À medida que a busca desce e sobe pela árvore, α e β vão se **estreitando**, formando uma "janela" [α, β] de valores ainda relevantes. A regra de poda é uma única condição:

> **Sempre que α ≥ β, pode-se parar de explorar os filhos restantes do nó atual (poda).**

Por que essa condição funciona? Se α ≥ β, significa que MAX já tem, em outro ramo, uma opção que garante α, enquanto neste ramo MIN já pode forçar algo ≤ β ≤ α. Logo, o jogador do nível acima **nunca escolherá este ramo** — ele já tem algo tão bom ou melhor em outro lugar. Explorar os filhos restantes não mudaria a decisão. Poda-se.

Concretamente, distinguem-se dois casos, com nomes tradicionais:

- **Corte beta (β-cutoff):** ocorre em um nó **MAX**. Se, ao avaliar os filhos, MAX encontra um valor ≥ β, ele para. Razão: MIN, no nível acima, já tem uma opção que limita o resultado a β; se este nó MAX pode alcançar algo ≥ β, MIN simplesmente não permitirá que se chegue aqui. Os demais filhos deste nó MAX são irrelevantes.
- **Corte alfa (α-cutoff):** ocorre em um nó **MIN**. Se, ao avaliar os filhos, MIN encontra um valor ≤ α, ele para. Razão: MAX, no nível acima, já garante α por outro caminho; se este nó MIN pode forçar algo ≤ α, MAX nunca escolherá vir para cá. Os demais filhos deste nó MIN são irrelevantes.

> ⚠️ **Atenção**
> A *nomenclatura* dos cortes varia entre as fontes. Aqui, "corte beta" nomeia a poda que ocorre em um nó **MAX** (quando o valor alcança β) e "corte alfa" a que ocorre em um nó **MIN** (quando o valor cai a α). Alguns textos (incluindo Russell & Norvig) descrevem o mesmo mecanismo em termos de *quem atualiza qual limite* — MAX eleva α, MIN reduz β —, o que pode inverter a impressão de qual letra "pertence" a cada nó. O algoritmo é idêntico; apenas a rotulagem difere. Ao resolver um exercício, verifique a convenção adotada pela fonte.

### Eliminação de ramos: um exemplo

Retomemos o exemplo numérico da Seção 11.3.3 (raiz MAX; jogadas A, B, C; cada uma levando a um nó MIN com dois filhos): A→(3,5), B→(6,2), C→(1,8).

A busca alfa-beta, explorando da esquerda para a direita:

1. **Ramo A.** Desce ao nó MIN sob A. Avalia a folha 3 e a folha 5; MIN escolhe min(3,5) = **3**. Volta à raiz: MAX agora tem **α = 3** garantido.
2. **Ramo B.** Desce ao nó MIN sob B, carregando α = 3, β = +∞. Avalia a **primeira** folha: **6**. MIN buscará o mínimo, então o valor deste nó será **≤ 6** — ainda não conclui nada. Avalia a **segunda** folha: **2**. Agora o nó MIN sob B vale no máximo min(6, 2) = **2**. Mas 2 ≤ α (=3): **corte alfa!** Se houvesse mais folhas sob B, elas seriam **podadas** — MAX já tem 3 garantido por A, e B não pode dar mais que 2. (Neste exemplo pequeno, a folha 2 era a última; em árvores maiores, tudo à direita dela seria descartado sem avaliação.)
3. **Ramo C.** Desce ao nó MIN sob C, α = 3. Avalia a **primeira** folha: **1**. O nó MIN sob C valerá no máximo 1 ≤ α (=3): **corte alfa!** A segunda folha de C — o tentador **8** — é **podada sem ser sequer olhada**. MAX já sabe que C não supera A.
4. **Raiz.** Resultado: max(3, 2, 1) = **3**, jogada **A** — exatamente o mesmo do Minimax puro.

O ganho: a folha **8** (e, em árvores realistas, subárvores inteiras) **nunca foi examinada**. E aqui está o ponto crucial:

### Corretude da poda

A poda alfa-beta **não altera a decisão do Minimax**. Isso não é um detalhe — é a propriedade que a torna aceitável. Os ramos podados são, por construção, ramos que **comprovadamente não influenciariam** o valor da raiz: o jogador do nível superior já tinha uma alternativa ao menos tão boa. Portanto, a jogada escolhida e o valor calculado pela alfa-beta são **idênticos** aos do Minimax completo. Ela é uma otimização **exata**, não uma aproximação: economiza trabalho **sem** sacrificar qualidade. Isso a distingue de heurísticas de corte "arriscadas" (como a poda de profundidade ou de largura, que descartam ramos possivelmente relevantes para ganhar velocidade). A alfa-beta é uma poda **segura**.

> ⚠️ **Atenção**
> Este é o ponto conceitual mais importante da seção, então vale repetir: **alfa-beta e Minimax produzem sempre a mesma jogada**. A alfa-beta é apenas uma forma mais **inteligente** de calcular o mesmo resultado, pulando trabalho comprovadamente inútil. Se um exercício pede "o valor minimax da raiz", ele é o mesmo com ou sem poda — a poda só muda **quantos nós** você precisou visitar para chegar a ele. Nunca pense na alfa-beta como "um algoritmo diferente que joga diferente"; pense nela como "o Minimax, sem desperdício".

### Impacto na complexidade

Qual o tamanho da economia? No **pior caso** (quando as jogadas são examinadas na pior ordem possível), a alfa-beta não poda nada e mantém a complexidade do Minimax, **O(b^d)**. Mas no **melhor caso** (ordem ótima de jogadas), a complexidade cai para **O(b^{d/2})** — a raiz quadrada do número de nós do Minimax. O que isso significa na prática é espetacular: com o mesmo tempo de computação, a alfa-beta permite buscar aproximadamente **o dobro da profundidade** que o Minimax puro alcançaria. Dobrar a profundidade em um jogo como o xadrez é uma diferença gigantesca de força — a diferença entre um amador e um mestre.

Dito de outra forma: se o Minimax puro consegue olhar 6 plies à frente em um segundo, a alfa-beta com boa ordenação consegue olhar cerca de 12 plies **no mesmo segundo**, examinando o mesmo número de nós porém cobrindo o dobro do horizonte. É por isso que **nenhum** programa sério de jogos de tabuleiro usa Minimax puro: a alfa-beta é considerada o padrão mínimo, o ponto de partida obrigatório.

> 🎲 **Curiosidade**
> A economia da alfa-beta depende crucialmente de examinar as jogadas em boa ordem — e isso nos leva a um resultado quase paradoxal. Para podar bem, gostaríamos de examinar **primeiro** as melhores jogadas; mas se já soubéssemos quais são as melhores, não precisaríamos buscar! A saída prática é **estimar** a ordem das jogadas com heurísticas baratas (ver 11.4.1), aproximando-se do melhor caso sem conhecê-lo de antemão. É um exemplo elegante de como uma boa aproximação inicial paga dividendos enormes ao longo de toda a busca.

### 11.4.1 Ordenação de jogadas

Já ficou claro que a eficácia da poda depende da **ordem** em que os filhos de cada nó são examinados. A **ordenação de jogadas** (*move ordering*) é a técnica de examinar primeiro as jogadas **mais promissoras**, para provocar cortes o mais cedo possível.

Por que a ordem importa tanto? Reveja o corte alfa: ele acontece quando MAX encontra, cedo, uma jogada tão boa que torna o restante irrelevante. Se a melhor jogada é a **primeira** examinada, o valor de α (ou β) sobe (ou desce) imediatamente para um patamar forte, e **todos** os ramos seguintes são podados rapidamente. Se, ao contrário, a melhor jogada é a **última**, o algoritmo perambula por ramos ruins antes de encontrar o bom limite, e poda pouco. A diferença entre a melhor e a pior ordenação é exatamente a diferença entre O(b^{d/2}) e O(b^d) — entre dobrar a profundidade e não ganhar nada.

Como estimar quais jogadas são promissoras, sem já ter feito a busca? As heurísticas de ordenação mais comuns:

- **Capturas e promoções primeiro** (em xadrez/damas): jogadas que ganham material tendem a ser fortes; examiná-las cedo costuma provocar cortes. Uma variante popular é a heurística **MVV-LVA** (*Most Valuable Victim – Least Valuable Attacker*): capturar a peça mais valiosa com a peça menos valiosa.
- **Jogada da tabela de transposição:** guarda-se, de buscas anteriores, qual jogada foi melhor em cada posição já vista (numa *transposition table*, tipicamente uma tabela hash); ao reencontrar a posição, examina-se essa jogada primeiro.
- **Aprofundamento iterativo (revisitado):** ao buscar em profundidade *d*, usa-se a melhor jogada encontrada na busca de profundidade *d−1* (mais barata) como primeira candidata. Aqui está a razão, prometida na Seção 11.3.2, de o aprofundamento iterativo ser frequentemente **mais rápido** que uma busca direta: o custo das buscas rasas é compensado com folga pela ordenação superior que elas fornecem às buscas profundas.
- **Heurísticas "killer" e histórico:** jogadas que causaram cortes em posições irmãs (*killer moves*) ou que historicamente foram boas (*history heuristic*) são priorizadas.

> ✅ **Boa Prática**
> Sempre compare mentalmente com o Minimax tradicional ao raciocinar sobre alfa-beta e ordenação. O Minimax puro é indiferente à ordem — ele visita todos os nós de qualquer jeito, então a ordenação não muda nada nele. É **apenas** quando se adiciona a poda que a ordenação passa a valer ouro. Essa é a razão de a ordenação de jogadas quase não ser discutida junto com o Minimax básico, mas ser um dos tópicos mais estudados em programação de xadrez: ela é o multiplicador de força da poda alfa-beta.

[DIAGRAMA]
Título: Minimax completo versus alfa-beta (árvore podada)
Objetivo pedagógico: Comparar lado a lado a árvore que o Minimax puro examina inteiramente e a mesma árvore com os ramos que a poda alfa-beta descarta, tornando visível a economia.
Descrição detalhada: Duas cópias da mesma árvore de jogo (3 níveis, com o exemplo A/B/C e folhas 3,5,6,2,1,8), lado a lado. À esquerda, rótulo "MINIMAX — examina TODOS os nós": todas as folhas e ramos desenhados em cor cheia. À direita, rótulo "ALFA-BETA — mesma decisão, menos trabalho": a mesma árvore, mas com a segunda folha de C (o valor 8) e quaisquer ramos à direita dos cortes desenhados apagados/tracejados em cinza, marcados com uma tesoura ✂ e a etiqueta "PODADO — não examinado". Anotar em cada nó os valores de α e β no momento do corte (ex.: no nó MIN sob C, "α=3, valor parcial=1 ≤ α ⇒ corte"). Embaixo, uma barra comparando "nós examinados: 6 (Minimax) × 5 (alfa-beta)" com nota "em árvores reais, a economia chega a b^{d/2}".
Elementos obrigatórios: as duas árvores idênticas; ramos podados destacados (tracejado + tesoura) na versão alfa-beta; valores de α/β anotados nos pontos de corte; a mesma raiz = 3 e jogada A em ambas; indicação da economia de nós.
[/DIAGRAMA]

---

## 11.5 Monte Carlo Tree Search (MCTS)

> **Nota sobre este tópico.** Esta seção é conteúdo de **aprofundamento**. O Minimax (com poda alfa-beta) permanece o **conteúdo central** deste capítulo e da Parte V. O MCTS é apresentado porque é impossível entender a IA de jogos moderna sem ele — foi a técnica por trás da revolução do Go —, mas ele **complementa**, não substitui, o entendimento do Minimax. Leia esta seção depois de dominar bem as anteriores.

### Motivação: onde o Minimax falha

O Minimax com alfa-beta domina jogos como xadrez e damas. Mas há uma classe de jogos em que ele **encalha**, e o exemplo canônico é o **Go**. Dois obstáculos, ambos já anunciados nos fundamentos, tornam o Minimax impraticável no Go:

1. **Fator de ramificação gigantesco.** No Go, *b* ≈ 250 (contra ≈ 35 do xadrez). A explosão *b^d* é tão brutal que mesmo a poda alfa-beta não permite profundidade útil. A árvore é larga demais.
2. **Ausência de uma boa função de avaliação.** No xadrez, contar material dá uma estimativa decente de quem está ganhando. No Go, **não existe** um análogo simples e confiável: avaliar quem está à frente numa posição de Go é notoriamente difícil, e por décadas ninguém conseguiu escrever uma função de avaliação heurística boa o suficiente. Sem boa avaliação, o Minimax limitado em profundidade propaga estimativas ruins e joga mal.

Foi essa dupla dificuldade que manteve os computadores em nível amador no Go por muito tempo, mesmo depois de dominarem o xadrez. A saída veio de uma ideia radicalmente diferente: **e se, em vez de avaliar posições com uma heurística, nós estimássemos o valor de uma jogada simulando partidas aleatórias até o fim e vendo quem ganha mais?**

### Funcionamento geral

O **Monte Carlo Tree Search** (busca em árvore de Monte Carlo) troca a **avaliação heurística** do Minimax por **amostragem estatística**. A ideia central: para estimar quão boa é uma jogada, jogue a partir dela **muitas partidas até o fim, com jogadas rápidas/aleatórias**, e conte a proporção de vitórias. Jogadas que levam a muitas vitórias nessas simulações são consideradas promissoras. Não se avalia a posição com conhecimento especializado; **avalia-se pela estatística de desfechos simulados**.

O nome "Monte Carlo" vem justamente desse uso de **amostragem aleatória** para estimar um valor difícil de calcular exatamente — a mesma família de métodos usada em física e finanças, aqui aplicada à decisão em jogos. O MCTS constrói a árvore de busca de forma **incremental e assimétrica**: em vez de expandir todos os ramos uniformemente até uma profundidade fixa (como o Minimax), ele **concentra o esforço** nos ramos que se mostram mais promissores, aprofundando-os mais e ignorando os ruins. A árvore cresce mais fundo onde importa.

Cada iteração do MCTS executa **quatro fases**, repetidas milhares de vezes dentro do orçamento de tempo:

**1. Seleção.** Partindo da raiz, o algoritmo desce pela árvore já construída, escolhendo a cada nó o filho mais "interessante", até chegar a um nó ainda não totalmente expandido. A escolha equilibra dois impulsos conflitantes: **exploração** (experimentar jogadas pouco testadas, que podem ser boas surpresas) e ***exploitation*** (aprofundar jogadas que já se mostraram boas). Esse equilíbrio é governado por uma fórmula chamada **UCT** (*Upper Confidence Bound applied to Trees*), que combina a taxa de vitórias observada de um nó com um termo que favorece nós pouco visitados. É a mesma matemática do problema clássico do "caça-níqueis de múltiplos braços" (*multi-armed bandit*): como distribuir tentativas entre opções incertas para maximizar o ganho.

**2. Expansão.** Ao chegar a um nó com jogadas ainda não representadas na árvore, adiciona-se um (ou mais) **novo nó filho** — expandindo a árvore em direção a uma posição ainda não explorada.

**3. Simulação (rollout/playout).** A partir do novo nó, joga-se uma **partida completa até o fim**, usando uma política de jogadas rápida (no MCTS clássico, **aleatória**; em versões modernas, guiada por uma rede neural). Não se avalia a posição — joga-se até haver um vencedor. O resultado (vitória ou derrota) é o "sinal" que essa simulação fornece.

**4. Retropropagação (backpropagation).** O resultado da simulação é **propagado de volta** pelo caminho percorrido, da folha até a raiz, atualizando em cada nó visitado duas estatísticas: o número de visitas e o número de vitórias. Assim, cada nó acumula, ao longo de milhares de iterações, uma estimativa cada vez mais confiável de sua taxa de vitórias.

Após esgotar o tempo (milhares ou milhões de iterações), a jogada escolhida na raiz é, tipicamente, a **mais visitada** — aquela em que o algoritmo investiu mais confiança. Note o contraste fundamental com o Minimax: o Minimax **calcula** o valor exato de uma árvore limitada; o MCTS **estima** valores por amostragem em uma árvore que ele mesmo faz crescer seletivamente.

> 🎲 **Curiosidade**
> A grande virtude do MCTS clássico é ser **agnóstico de domínio**: ele não precisa de uma função de avaliação especializada, apenas das regras do jogo (para simular até o fim) e de saber quem venceu. Por isso conquistou não só o Go, mas também jogos gerais (*General Game Playing*) e jogos com muita ramificação. Sua fraqueza espelha essa virtude: sem conhecimento do domínio, as simulações aleatórias podem ser pouco informativas em jogos com táticas agudas (como o xadrez, onde uma única jogada errada arruína a posição) — e é por isso que, curiosamente, o xadrez continuou sendo território do alfa-beta mesmo depois que o Go caiu para o MCTS.

### Por que o MCTS se tornou popular

O marco histórico foi **2016**, quando o **AlphaGo**, da DeepMind, derrotou **Lee Sedol**, um dos maiores jogadores de Go do mundo — um feito que especialistas acreditavam estar a **décadas** de distância. O AlphaGo combinou MCTS com **redes neurais profundas**: uma rede "de política" para sugerir jogadas promissoras (guiando a seleção e substituindo as simulações puramente aleatórias) e uma rede "de valor" para avaliar posições (a função de avaliação que faltava ao Go). O sucessor **AlphaZero** (2017) generalizou a abordagem, aprendendo **do zero, apenas por auto-jogo**, a jogar Go, xadrez e shogi em nível sobre-humano — sempre com MCTS no núcleo da busca.

Foi esse conjunto de resultados que consolidou o MCTS como uma das técnicas mais importantes da IA de jogos moderna. Ele se tornou popular porque: (a) lida bem com **fator de ramificação alto**; (b) **dispensa** uma função de avaliação manual (crucial onde ela é difícil de construir); (c) é **anytime** — pode ser interrompido a qualquer momento e devolver a melhor jogada encontrada até ali, gastando mais tempo apenas se houver; e (d) combina-se naturalmente com **aprendizado de máquina**, formando a ponte mais direta entre a busca clássica (esta Parte) e a IA que aprende (Parte VI).

> ⚠️ **Atenção**
> Apesar de toda a fama, **não** conclua que "MCTS tornou o Minimax obsoleto". Não tornou. Em jogos com fator de ramificação moderado e boa função de avaliação disponível — xadrez, damas, muitos jogos de tabuleiro digitais e a maioria dos jogos de estratégia por turnos comerciais — o **alfa-beta continua sendo mais forte e mais eficiente**. O melhor programa de xadrez do mundo (Stockfish) usa alfa-beta, não MCTS. A regra prática: **MCTS** quando a ramificação é enorme e/ou não há boa avaliação (Go, jogos gerais); **alfa-beta** quando há boa avaliação e ramificação administrável (xadrez, damas). São ferramentas complementares, cada uma soberana em seu domínio — exatamente o tipo de discriminação que a tabela comparativa do encerramento desta Parte consolidará.

[DIAGRAMA]
Título: As quatro fases do MCTS
Objetivo pedagógico: Mostrar o ciclo seleção → expansão → simulação → retropropagação de uma iteração do MCTS, evidenciando o crescimento assimétrico da árvore.
Descrição detalhada: Quatro quadros em sequência, cada um mostrando a mesma árvore parcial de MCTS em um estágio. Quadro 1 ("Seleção"): a partir da raiz, uma seta desce por nós já existentes seguindo os de maior valor UCT, até um nó de fronteira; nós anotados com estatísticas "vitórias/visitas" (ex.: 7/10, 3/8). Quadro 2 ("Expansão"): um novo nó-filho (destacado em verde) é adicionado à fronteira. Quadro 3 ("Simulação"): a partir do novo nó, uma linha ondulada/tracejada rápida desce até um símbolo de fim de jogo (bandeira) com resultado "VITÓRIA" ou "DERROTA", rotulada "playout aleatório até o fim". Quadro 4 ("Retropropagação"): setas subindo do nó novo até a raiz, com as estatísticas "vitórias/visitas" sendo atualizadas em cada nó do caminho (ex.: 7/10 → 8/11). Abaixo dos quatro quadros, a legenda "repetir milhares de vezes; ao final, jogar o filho da raiz mais visitado". Contrastar, num cantinho, com um selo "sem função de avaliação — só regras + estatística".
Elementos obrigatórios: os quatro quadros rotulados; estatísticas vitórias/visitas nos nós; o playout até um estado terminal; a atualização retropropagada; a nota de repetição e escolha final; o crescimento assimétrico (a árvore mais funda no ramo promissor).
[/DIAGRAMA]

---

## 11.6 Exemplos

Consolidemos tudo com três exemplos de dificuldade crescente. Em cada um, **primeiro** analisamos a árvore de decisão (o objeto sobre o qual o algoritmo opera) e **depois** a execução do Minimax — a ordem que a apostila adota sempre: a ideia antes do mecanismo.

### Jogo da velha — o Minimax completo e exato

O jogo da velha é o exemplo perfeito para começar, porque é **pequeno o bastante para ser resolvido por completo**, sem heurística. Suas características: tabuleiro 3×3, dois jogadores (X e O) alternando, soma zero, informação perfeita.

**A árvore de decisão.** Na posição inicial (tabuleiro vazio), o jogador X tem **9** jogadas possíveis. Após a jogada de X, O tem 8; depois X tem 7, e assim por diante. O fator de ramificação começa em 9 e cai a cada jogada, pois as casas se esgotam. A árvore completa tem, no máximo, 9! = 362.880 folhas se contássemos todas as ordens de preenchimento — e bem menos considerando que muitas partidas terminam antes de o tabuleiro encher (por vitória) e descontando simetrias. Esse número é **pequeno para um computador**: dá para explorar a árvore **inteira**, até as folhas terminais, sem parar em profundidade nenhuma.

**A execução do Minimax.** Como chegamos às folhas reais, não precisamos de função de avaliação — usamos apenas a **utilidade**: +1 se X vence, −1 se O vence, 0 se empata (medindo do ponto de vista de X = MAX). O Minimax propaga esses valores de baixo para cima: nos níveis de X (MAX), toma o máximo; nos de O (MIN), o mínimo. O resultado é conhecido e instrutivo: **com jogo perfeito dos dois lados, o jogo da velha sempre empata** (valor minimax da raiz = 0). Uma IA Minimax de jogo da velha é, portanto, **imbatível**: ela nunca perde. Se o oponente jogar perfeitamente, empata; se errar, ela vence. É o exemplo mínimo, completo e satisfatório do Minimax em sua forma pura — sem horizonte, sem heurística, sem poda necessária (embora a poda funcione aqui também).

> 🎮 **Na Prática**
> O jogo da velha é o "olá, mundo" da busca adversarial, e por boa razão: é o único dos nossos três exemplos em que o Minimax puro roda até o fim em tempo trivial, permitindo ver o algoritmo funcionando **exatamente** como a teoria descreve, sem as aproximações que os jogos maiores impõem. Se você for implementar Minimax uma vez na vida para entendê-lo, comece por ele — a árvore é pequena o suficiente para você traçar ramos à mão e conferir o que o código faz.

### Damas — quando a heurística e a poda se tornam necessárias

As damas elevam a escala. Tabuleiro 8×8 (com 32 casas jogáveis), fator de ramificação médio *b* ≈ 8 (mais quando há capturas obrigatórias e múltiplas), e um número de posições na casa de **10²⁰** — grande demais para explorar até o fim em tempo de jogo.

**A árvore de decisão.** A árvore de damas já **não pode** ser explorada até as folhas. Aparecem, aqui, todos os conceitos da Seção 11.2 em pleno vigor: precisamos de um **limite de profundidade**, de uma **função de avaliação** para os nós de fronteira, e da **poda alfa-beta** para alcançar profundidade útil. Uma peculiaridade das damas — as **capturas obrigatórias e em cadeia** — cria posições "instáveis" que ilustram bem o **efeito horizonte** e a necessidade de estender a busca até posições calmas (busca de quietude).

**A execução.** Uma função de avaliação típica de damas conta o **material** (peças simples versus damas coroadas, com pesos diferentes), a **posição** (peças avançadas, controle do centro, peças na última fileira que impedem a coroação adversária) e a **mobilidade**. O Minimax com alfa-beta, aprofundando o quanto o tempo permitir, propaga essas avaliações e escolhe a jogada. As damas são historicamente importantes porque foram o palco de **Chinook** (o programa de Jonathan Schaeffer) e, mais tarde, da **resolução completa** do jogo — voltaremos a isso nos estudos de caso.

### Xadrez — a busca adversarial em escala plena

O xadrez é o exemplo culminante, onde todos os desafios se manifestam na forma mais aguda. Fator de ramificação *b* ≈ 35, profundidade de uma partida típica em torno de 80 plies, e o já citado **número de Shannon** (~10¹²⁰ partidas possíveis).

**A árvore de decisão.** Absolutamente impossível de explorar por completo. Tudo o que estudamos torna-se **obrigatório**: profundidade limitada, função de avaliação sofisticada, poda alfa-beta, ordenação de jogadas, tabelas de transposição, aprofundamento iterativo, busca de quietude. O xadrez é, historicamente, o problema que **forçou o desenvolvimento** de quase todas essas técnicas — cada refinamento do Minimax prático nasceu, em boa parte, da corrida para jogar xadrez melhor.

**A execução.** Um motor de xadrez clássico executa, a cada lance, uma busca alfa-beta com aprofundamento iterativo até onde o tempo permite (frequentemente 15–25 plies em hardware moderno, muito além do que o Minimax puro alcançaria), usando uma função de avaliação que pesa material e dezenas de fatores posicionais, com todas as otimizações de ordenação. É o Minimax da Seção 11.3, turbinado pela poda da Seção 11.4 e por uma heurística da Subseção 11.3.4 refinada ao longo de décadas. O ápice dessa linhagem — o confronto **Deep Blue × Kasparov** — é o tema central da próxima seção.

> ✅ **Boa Prática**
> Perceba a progressão pedagógica destes três exemplos, porque ela resume o capítulo inteiro: **jogo da velha** mostra o Minimax **puro e exato** (sem horizonte nem heurística); **damas** introduzem a **necessidade** de horizonte, heurística e poda; **xadrez** leva tudo ao **limite**, exigindo o arsenal completo. Se você entende por que cada exemplo precisa de mais maquinaria que o anterior, entendeu por que o Minimax teórico e o Minimax prático são, na verdade, o mesmo algoritmo em graus diferentes de exigência.

---

## 11.7 Vantagens e Limitações

Avaliemos criticamente a busca adversarial, separando o que a torna poderosa do que a limita.

**Vantagens.** A principal força do Minimax é ser **teoricamente correto e ótimo**: contra um adversário perfeito, ele joga da melhor forma possível, com garantia matemática. É **completo** (encontra a jogada em árvores finitas), **determinístico** (mesma entrada, mesma saída — fácil de depurar e testar) e **conceitualmente transparente** (podemos inspecionar exatamente por que escolheu cada jogada, ao contrário de métodos aprendidos que funcionam como caixas-pretas). A poda alfa-beta preserva todas essas garantias enquanto reduz drasticamente o custo. E a separação limpa entre **busca** e **avaliação** permite melhorar cada parte independentemente. Para jogos de tabuleiro determinísticos, de informação perfeita e com boa função de avaliação disponível, é difícil superá-lo.

**Limitações.** As limitações são, quase todas, faces de um único problema — a **explosão combinatória** — e merecem discussão individual:

- **Explosão combinatória.** O número de nós cresce como *b^d*. É a limitação-mãe: torna a exploração completa impossível em qualquer jogo não-trivial e força todas as outras concessões. Nenhuma otimização a elimina; ela apenas é **empurrada** (a alfa-beta a reduz à raiz quadrada, mas continua exponencial).
- **Fator de ramificação.** Quanto maior *b*, mais rasa a busca possível no mesmo tempo. Jogos de *b* alto (Go, com *b* ≈ 250) simplesmente **quebram** o Minimax, como vimos — foi o que motivou o MCTS.
- **Profundidade limitada e efeito horizonte.** Como não se chega às folhas, a busca para num horizonte artificial e pode ser iludida por eventos logo além dele (Seção 11.3.3). Mitigável (quietude, aprofundamento), nunca eliminável.
- **Dependência da heurística.** A qualidade do jogo depende criticamente da função de avaliação (11.3.4). Onde não existe boa heurística (Go clássico), o Minimax joga mal por melhor que seja o algoritmo. Construir boas heurísticas é difícil e específico de cada jogo.
- **Custo computacional.** Avaliar milhões de posições por jogada consome muito processamento — inviável dentro do orçamento de milissegundos de um jogo de ação em tempo real.
- **Memória.** Embora a busca em profundidade mantenha na memória apenas o caminho atual (custo de memória **linear** na profundidade, O(d), o que é modesto), as **tabelas de transposição** que aceleram a busca podem consumir muita memória, e há um trade-off entre velocidade e espaço.

**Quando o Minimax deixa de ser viável.** Reunindo as limitações, o Minimax **deixa de ser a escolha certa** quando: (a) o fator de ramificação é grande demais (Go, jogos com muitíssimas ações); (b) não há função de avaliação confiável; (c) o jogo é em **tempo real** com orçamento de milissegundos (jogos de ação); (d) o jogo tem **acaso** ou **informação oculta** (pôquer, jogos com dados/cartas), exigindo variantes como Expectiminimax ou abordagens de teoria dos jogos com incerteza; ou (e) há **mais de dois jogadores** em alianças mutáveis, onde a suposição de soma zero entre dois lados não se aplica. Reconhecer esses limites é tão importante quanto conhecer o algoritmo: aplicá-lo fora de seu domínio é a receita para uma IA lenta e fraca.

> ❌ **Erro Comum**
> Um equívoco de projeto recorrente é tentar usar Minimax em um jogo de **ação em tempo real** ("vou fazer o inimigo do meu FPS calcular uma árvore de jogo"). Isso quase sempre fracassa: o orçamento de milissegundos por quadro não comporta a busca, e o ambiente não é o de "turnos alternados de informação perfeita" que o Minimax pressupõe. Para IA de jogos de ação, as ferramentas certas são as das Partes II a IV (FSM, behavior trees, GOAP, mapas de influência). Minimax é para o **nicho** de jogos de tabuleiro e estratégia por turnos — poderosíssimo ali, inadequado fora.

---

## 11.8 Estudos de Caso

Examinemos como a busca adversarial apareceu em sistemas reais. Seguindo a orientação editorial, **distinguimos com rigor** o que é **fato documentado** do que é **análise técnica fundamentada** (inferência plausível, não confirmada oficialmente).

### Deep Blue × Kasparov (1997) — fato documentado

O caso mais célebre da história da IA de jogos. Em maio de 1997, o computador **Deep Blue**, da IBM, derrotou o campeão mundial de xadrez **Garry Kasparov** em uma partida de seis jogos (2 vitórias, 1 derrota e 3 empates para a máquina — placar 3½–2½). Foi a primeira vez que um computador venceu um campeão mundial reinante em condições de torneio, um marco simbólico comparável, para a IA, à chegada à Lua.

O que está **documentado** sobre o Deep Blue: ele era um sistema **massivamente paralelo**, com **hardware dedicado** (chips VLSI projetados especificamente para gerar e avaliar lances de xadrez), capaz de examinar cerca de **200 milhões de posições por segundo**. Sua base algorítmica era **exatamente o que estudamos neste capítulo**: **busca alfa-beta** com aprofundamento iterativo, uma **função de avaliação** com milhares de características ajustadas com auxílio de grandes mestres, busca de quietude, tabelas de transposição e extensa **ordenação de jogadas**. Complementava a busca com **bibliotecas de aberturas** e **bases de finais** (tablebases) — posições pré-computadas. O Deep Blue não "aprendeu" nem usou redes neurais; foi o triunfo da **força bruta bem-engenheirada** sobre a árvore de jogo, o ápice da linhagem Shannon–alfa-beta.

> 🕰️ **Contexto Histórico**
> A vitória do Deep Blue encerrou um debate de quase meio século. Desde Shannon e Turing, o xadrez fora considerado o "*teste da mosca-drosófila*" da IA — o organismo-modelo em que se estudaria a inteligência de máquina. Curiosamente, o desfecho foi ambíguo: o Deep Blue mostrou que o xadrez de campeonato podia ser vencido **sem** nada que se parecesse com o pensamento humano — sem intuição, sem compreensão, apenas busca e avaliação em escala colossal. Para muitos, isso não provou que a máquina "pensava", mas sim que o xadrez era mais suscetível à força de cálculo do que se imaginava. O verdadeiro salto para algo mais próximo do "aprendizado" viria só com AlphaGo/AlphaZero, vinte anos depois.

### Chinook e a resolução das damas — fato documentado

O programa **Chinook**, desenvolvido por **Jonathan Schaeffer** e sua equipe na Universidade de Alberta, tornou-se em 1994 o primeiro programa a conquistar um título mundial de damas contra humanos. Baseava-se, novamente, em **busca alfa-beta** com função de avaliação e enormes bases de finais. Em **2007**, a mesma equipe anunciou que havia **resolvido completamente** o jogo de damas (*checkers is solved*): provaram, com 18 anos de computação, que com jogo perfeito dos dois lados o resultado é **empate**. É o maior jogo já resolvido por completo até então — um feito que ilustra tanto o poder da busca adversarial exaustiva quanto o custo astronômico de resolver mesmo um jogo de ramificação modesta.

### Jogos de tabuleiro e estratégia por turnos digitais — análise técnica fundamentada

Aqui entramos no terreno da **inferência**: para a maioria dos jogos comerciais, os desenvolvedores não publicam detalhes de implementação, de modo que o que segue é **análise técnica plausível, não confirmação oficial**.

É **razoável inferir** que implementações digitais de **xadrez, damas, Othello/Reversi, Gomoku e Connect Four** usem Minimax com poda alfa-beta e funções de avaliação específicas de cada jogo — é a abordagem padrão, documentada em incontáveis fontes técnicas, e a mais natural para esses domínios. Muitos jogos de **estratégia por turnos** e de **tabuleiro** com IA de nível ajustável provavelmente usam buscas adversariais de profundidade variável (profundidade maior = dificuldade maior) como um dos componentes de sua IA, frequentemente combinadas com heurísticas de domínio. Reversi/Othello, em particular, é um caso didático clássico de alfa-beta, com função de avaliação baseada em estabilidade de peças e controle de cantos.

> ⚠️ **Atenção**
> Trate essas atribuições com o rigor devido: dizer "este jogo *provavelmente* usa Minimax com alfa-beta" é uma **hipótese fundamentada** na natureza do jogo e na prática da indústria, **não** um fato verificado. Onde há documentação oficial (Deep Blue, Chinook, AlphaGo), afirmamos com segurança; onde não há, sinalizamos a inferência. Essa disciplina — separar o que se sabe do que se supõe — é exatamente a competência que a Parte VII (engenharia reversa) desenvolverá em profundidade.

### AlphaGo / AlphaZero — fato documentado (fronteira MCTS)

Já citados na Seção 11.5, fecham o arco histórico. O **AlphaGo** (DeepMind) derrotou o profissional **Lee Sedol** no Go em 2016, e o **AlphaZero** (2017) alcançou nível sobre-humano em Go, xadrez e shogi aprendendo **apenas por auto-jogo**. Ambos usam **MCTS** (não alfa-beta) guiado por redes neurais. São o contraponto ao Deep Blue: onde este foi o ápice da busca clássica com heurística programada, aqueles inauguraram a era da busca combinada com **avaliação aprendida** — a transição que nos conduz naturalmente à Parte VI.

---

## 11.9 Ferramentas

Seguindo a filosofia da apostila, esta seção é **contextualização**, não tutorial. O objetivo é situar onde e como a busca adversarial aparece nas ferramentas de desenvolvimento, sem ensinar menus nem passo a passo.

**Na Unity.** É importante deixar claro de saída: a Unity **não** oferece um sistema nativo dedicado à busca adversarial, do modo como oferece o **NavMesh** para pathfinding (Parte III) ou o **Animator/Unity Behavior** para máquinas de estado e árvores de comportamento (Parte II). A razão é coerente com tudo o que discutimos: o Minimax é a ferramenta de um **nicho** (jogos de tabuleiro e estratégia por turnos), e a Unity é um motor de propósito geral voltado majoritariamente a jogos de ação e tempo real, onde a busca adversarial raramente cabe. Portanto, quando um jogo feito na Unity precisa de Minimax — digamos, um xadrez, um jogo de damas ou um jogo de tabuleiro por turnos —, o desenvolvedor **implementa o algoritmo em C#** sobre a própria lógica do jogo. Isso é natural e nada exótico: o Minimax é conceitualmente simples (reveja o pseudocódigo da Seção 11.3), e traduzi-lo para uma classe C# que opere sobre a representação do estado do jogo é um exercício direto. A "ferramenta", aqui, é a linguagem e a estrutura de dados do próprio jogo, não um pacote pronto.

**Bibliotecas e soluções de terceiros.** Para não reinventar a roda, existem bibliotecas de código aberto de **busca em árvore de jogo** — implementações genéricas de Minimax, alfa-beta e MCTS que recebem, por injeção, as regras do jogo específico (função de sucessores, teste de término, utilidade e avaliação) e cuidam da mecânica da busca. Na Asset Store e em repositórios abertos há também implementações prontas para jogos específicos (motores de xadrez em C#, por exemplo) e frameworks de MCTS. Para o xadrez em particular, motores maduros de código aberto (como o **Stockfish**, que usa alfa-beta com avaliação NNUE) podem ser integrados via comunicação por protocolo (UCI), delegando a "inteligência" a um motor especializado enquanto a Unity cuida da apresentação.

**Exemplos em C#.** Quando se implementa na mão, o esqueleto segue exatamente a estrutura recursiva já apresentada. Para fixar a **contextualização** (não como tutorial), eis o formato mínimo de uma assinatura de método Minimax em C#, apenas para evidenciar a correspondência direta com o pseudocódigo:

```csharp
// Contextualização — não é um tutorial de implementação.
// A assinatura evidencia os mesmos ingredientes do pseudocódigo da Seção 11.3.
int Minimax(EstadoJogo estado, int profundidade, bool ehMax)
{
    if (estado.Terminal())        return estado.Utilidade();   // folha real
    if (profundidade == 0)        return estado.Avaliacao();   // heurística
    // ... laço sobre estado.JogadasLegais(), alternando Max/Min ...
}
```

O ponto pedagógico é reconhecer que **não há mágica de ferramenta**: a busca adversarial em um jogo Unity é o algoritmo deste capítulo escrito em C#, operando sobre a sua própria modelagem de estado. Dominar o **conceito** — árvore de jogo, propagação MAX/MIN, poda alfa-beta, função de avaliação — é o que capacita o desenvolvedor a implementá-lo, ajustá-lo e depurá-lo, com ou sem biblioteca de apoio.

> 🏭 **Na Indústria**
> Uma escolha de engenharia comum em jogos de tabuleiro comerciais é **separar** a IA da apresentação: um módulo de busca (Minimax/alfa-beta ou MCTS) roda de forma independente do motor gráfico, muitas vezes em uma *thread* separada ou de forma incremental (aproveitando a natureza *anytime* do aprofundamento iterativo e do MCTS) para não travar o quadro. A dificuldade da IA costuma ser ajustada variando a **profundidade de busca** ou o **número de iterações de MCTS** — uma alavanca simples e eficaz para oferecer níveis "fácil/médio/difícil" a partir do mesmo algoritmo.

---

## 11.10 Encerramento do Capítulo

### Resumo

Este capítulo apresentou a **busca adversarial**, a família de técnicas para decidir bem contra um **oponente racional** — o único ambiente, entre os que a apostila estuda, em que o "mundo" planeja ativamente contra o agente. Partimos do **problema** (jogar contra uma mente que quer o oposto do que queremos) e da distinção entre ambientes **cooperativos, reativos e competitivos**, situando o Minimax como ferramenta específica do último.

Construímos os **fundamentos** — jogos de **soma zero**, **turnos**, **informação perfeita** e a **árvore de jogo** com seus elementos (estados, ações, profundidade, fator de ramificação, utilidade e função de avaliação). Sobre eles, detalhamos o **Minimax**: os jogadores **MAX** e **MIN**, a **propagação** alternada de valores das folhas à raiz, a **escolha ótima** e a construção recursiva da busca. Aprofundamos as **camadas MAX/MIN**, a **profundidade** limitada, o **horizonte** de busca (e o traiçoeiro **efeito horizonte**) e o papel decisivo das **funções heurísticas de avaliação**, que tornam o Minimax viável em jogos reais e cuja qualidade influencia o desempenho tanto quanto o próprio algoritmo.

Apresentamos a **poda alfa-beta** como a otimização **exata** do Minimax — mesma decisão, muito menos trabalho —, com seus limites **α** e **β**, os cortes, a **corretude** (não muda a jogada) e o **impacto na complexidade** (de O(b^d) para O(b^{d/2}) no melhor caso, dobrando a profundidade alcançável), destacando a **ordenação de jogadas** como o multiplicador dessa economia. Como **aprofundamento**, vimos o **MCTS** — seleção, expansão, simulação e retropropagação —, a técnica que, dispensando função de avaliação e lidando com ramificação enorme, conquistou o Go e fundou, com AlphaGo/AlphaZero, a ponte para o aprendizado de máquina. Fechamos com **exemplos** progressivos (jogo da velha, damas, xadrez), **estudos de caso** documentados (Deep Blue, Chinook, AlphaGo) e a **contextualização** de ferramentas, reforçando que, na Unity, o Minimax é implementado em C# sobre a lógica do próprio jogo.

### Questões de Revisão

1. Explique, com suas palavras, por que jogar contra um oponente racional exige uma abordagem diferente das técnicas das Partes II a IV. O que muda na natureza da decisão?
2. Diferencie ambientes cooperativos, reativos e competitivos. Dê um exemplo de jogo (ou situação de jogo) para cada um e indique qual família de técnicas de IA é adequada.
3. O que é um jogo de soma zero? Por que essa propriedade permite ao Minimax usar uma única função de valor em vez de duas?
4. Defina, na árvore de jogo: estado, ação, profundidade, fator de ramificação, utilidade e função de avaliação. Qual a diferença essencial entre **utilidade** e **função de avaliação**?
5. Descreva a regra de propagação do Minimax nos nós MAX e nos nós MIN. Por que MAX "maximiza" e MIN "minimiza" sobre a **mesma** medida de valor?
6. O que é o efeito horizonte? Por que ele é uma limitação fundamental (e não um simples bug), e como a busca de quietude o mitiga?
7. Enuncie a condição de poda alfa-beta (α ≥ β) e explique por que podar não altera a decisão final do Minimax.
8. Por que a **ordenação de jogadas** é irrelevante para o Minimax puro, mas decisiva para a alfa-beta? Relacione com a diferença entre O(b^d) e O(b^{d/2}).
9. Liste as quatro fases do MCTS e explique o que cada uma faz. Em que dois aspectos o MCTS difere fundamentalmente do Minimax?
10. Em quais situações o Minimax **deixa de ser viável**? Para cada uma, indique por quê e qual abordagem alternativa seria mais adequada.

### Exercícios de Fixação

1. **Traçado de Minimax.** Considere uma raiz MAX com três jogadas. A primeira leva a um nó MIN com folhas (7, 4); a segunda a um nó MIN com folhas (5, 9); a terceira a um nó MIN com folhas (6, 6). Calcule o valor minimax de cada nó MIN, o valor da raiz e indique a jogada escolhida. Explique por que a folha de valor 9 não determina a escolha.
2. **Traçado de alfa-beta.** Usando a mesma árvore do exercício anterior, execute a poda alfa-beta da esquerda para a direita. Anote os valores de α e β ao longo da busca e identifique se algum ramo é podado. Confirme que a jogada escolhida é a mesma do Minimax puro.
3. **Efeito da ordenação.** Reordene as três jogadas do exercício 1 de modo a **maximizar** a poda (mais cortes) e depois de modo a **minimizá-la** (nenhum corte). Explique, a partir dos dois arranjos, por que a ordenação de jogadas vale tanto para a alfa-beta.
4. **Escolha de profundidade.** Um jogo tem *b* ≈ 20. Estime a ordem de grandeza do número de nós examinados pelo Minimax puro a profundidades 3, 4 e 5. Depois estime o efeito da alfa-beta no melhor caso. Discuta o impacto no tempo de resposta de um jogo por turnos versus um em tempo real.
5. **Projeto de função de avaliação.** Para o jogo Connect Four (Lig-4), proponha três características (features) que uma função de avaliação poderia medir e justifique o peso relativo de cada uma. Explique como uma heurística melhor poderia compensar uma profundidade de busca menor.
6. **Classificação de jogos.** Para cada jogo — pôquer, Go, xadrez, jogo da velha e um MOBA em tempo real —, indique se o Minimax clássico é adequado e por quê (considere ramificação, informação perfeita/oculta, acaso, tempo real e disponibilidade de boa heurística).

### Leituras Complementares

- **Russell, S.; Norvig, P.** *Inteligência Artificial*. 3ª ed. — Capítulo sobre "Busca Competitiva/Jogos": a referência canônica para Minimax, alfa-beta e Expectiminimax, com o tratamento formal completo.
- **Millington, I.** *AI for Games*. 3ª ed. — Capítulo sobre *Board Game AI*: Minimax, poda alfa-beta, tabelas de transposição, funções de avaliação e MCTS na perspectiva prática de desenvolvimento de jogos.
- **Rabin, S. (org.).** *Game AI Pro* (séries) — Artigos sobre implementações de MCTS e otimizações de busca em jogos comerciais.
- **Shannon, C. E.** *"Programming a Computer for Playing Chess"* (1950) — O artigo fundador; curto, histórico e surpreendentemente atual na descrição da árvore de busca e da função de avaliação.
- **Schaeffer, J.** *One Jump Ahead* — Relato em primeira pessoa do desenvolvimento do Chinook e da resolução das damas; leitura acessível sobre busca adversarial na prática.

### Referências

- RUSSELL, S.; NORVIG, P. *Inteligência Artificial*. 3ª ed. Rio de Janeiro: Elsevier, 2013.
- MILLINGTON, I. *AI for Games*. 3rd ed. Boca Raton: CRC Press, 2019.
- BOURG, D. M.; SEEMANN, G. *AI for Game Developers*. Sebastopol: O'Reilly, 2004.
- RABIN, S. (Ed.). *Game AI Pro 3: Collected Wisdom of Game AI Professionals*. Boca Raton: CRC Press, 2017.
- CORMEN, T. H.; LEISERSON, C. E.; RIVEST, R. L.; STEIN, C. *Algoritmos: Teoria e Prática*. 3ª ed. Rio de Janeiro: Elsevier, 2012.
- SHANNON, C. E. Programming a Computer for Playing Chess. *Philosophical Magazine*, v. 41, n. 314, 1950.
- CAMPBELL, M.; HOANE, A. J.; HSU, F. Deep Blue. *Artificial Intelligence*, v. 134, n. 1-2, p. 57-83, 2002.
- SCHAEFFER, J. et al. Checkers is Solved. *Science*, v. 317, n. 5844, p. 1518-1522, 2007.
- SILVER, D. et al. Mastering the game of Go with deep neural networks and tree search. *Nature*, v. 529, 2016.
- SILVER, D. et al. A general reinforcement learning algorithm that masters chess, shogi, and Go through self-play. *Science*, v. 362, n. 6419, 2018.
- BROWNE, C. et al. A Survey of Monte Carlo Tree Search Methods. *IEEE Transactions on Computational Intelligence and AI in Games*, v. 4, n. 1, 2012.
