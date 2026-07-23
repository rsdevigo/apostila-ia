# PARTE III — Movimento e Busca de Caminhos

> **Apostila:** *Inteligência Artificial e Ilusão de Inteligência*
> **Curso:** Superior de Tecnologia em Jogos Digitais — IFMS
> **Parte III de VII** — Capítulos 7, 8 e 9

---

## Apresentação da Parte III

Nas Partes I e II construímos as fundações da IA de jogos e estudamos, em profundidade, a fase **Pensar** do ciclo do agente pela família da **tomada de decisão baseada em regras**. Aprendemos como um NPC decide *o que* fazer: patrulhar, perseguir, atacar, procurar cobertura. Mas quase todas essas decisões terminam com um verbo que ainda não sabemos executar — **mover-se**. O guarda decide *perseguir*; o soldado decide *avançar até aquela cobertura*; a unidade de estratégia recebe a ordem de *ir até ali*. Em todos esses casos, a decisão produz um **destino**, e resta o problema que organiza toda esta Parte: **como chegar até lá?**

Essa pergunta parece trivial para um ser humano — olhamos para o mundo e simplesmente andamos até o ponto desejado, desviando de paredes e obstáculos sem esforço consciente. Para um agente de software, porém, ela é um dos problemas mais estudados e computacionalmente sensíveis de toda a IA de jogos. O personagem não "vê" o cenário como uma cena tridimensional contínua; ele precisa de uma **representação computacional do espaço navegável** e de um **algoritmo de busca** capaz de encontrar, nessa representação, um caminho que seja curto, que não atravesse paredes e — decisivo para um jogo — que seja calculado em uma fração de milissegundo, dezenas ou centenas de vezes por segundo, dentro do severo orçamento de quadro que estabelecemos na Parte I.

O fio condutor desta Parte é **construtivo e cumulativo**: cada capítulo fornece o alicerce exato de que o seguinte precisa, sem jamais apresentar um conceito antes de seus pré-requisitos. A progressão é rígida e proposital:

1. O **Capítulo 7 — Grafos e Representação do Espaço** é o capítulo de **fundamentação**. Ele responde à pergunta anterior a todas as outras: *como o mundo do jogo é convertido em algo sobre o qual um computador pode buscar?* Aqui apresentamos a **teoria dos grafos** aplicada a jogos — vértices, arestas, pesos, conectividade — e as três grandes formas de representar o espaço: **grades (grids)**, **grafos de waypoints** e **malhas de navegação (NavMesh)**. Ao final, o leitor enxergará qualquer cenário de jogo como um grafo, pronto para ser percorrido.

2. O **Capítulo 8 — Algoritmo A\*** é o **coração da Parte**. Sobre o grafo do Capítulo 7, construímos o algoritmo de busca de caminhos mais utilizado da história dos jogos. Partimos da busca não-informada de **Dijkstra**, introduzimos a ideia de **heurística** e de **busca informada** e chegamos, passo a passo, à função de avaliação `f = g + h`, às **listas aberta e fechada**, à **reconstrução do caminho** e às propriedades formais — **admissibilidade** e **consistência** — que explicam *por que* o A\* funciona e quando ele garante o caminho ótimo.

3. O **Capítulo 9 — JPS+ e Otimizações** mostra a **fronteira de desempenho**. O A\* é excelente, mas pode ser caro em grades grandes e uniformes. Estudamos o **Jump Point Search** e sua variante pré-processada **JPS+**, que exploram a **simetria de caminhos** para acelerar a busca em ordens de magnitude, além de outras técnicas maduras da indústria: **pathfinding hierárquico**, **pré-processamento** e **suavização de caminho (path smoothing)**. Igualmente importante, o capítulo ensina a reconhecer *quando* essas otimizações valem a pena — e quando não são apropriadas.

> **Boa Prática**
> Leia esta Parte com uma bússola conceitual sempre à mão: *o problema de busca de caminhos tem sempre duas metades — a representação (como o espaço vira grafo) e a busca (como se acha o caminho no grafo).* A maioria das confusões de estudantes nasce de misturar as duas. O Capítulo 7 cuida da primeira metade; os Capítulos 8 e 9, da segunda. Nenhum algoritmo de busca é melhor do que a representação sobre a qual roda.

Ao concluir a Parte III, você será capaz de compreender como o espaço de um jogo é representado computacionalmente, de aplicar a teoria dos grafos a problemas concretos de navegação, de explicar e traçar a execução do algoritmo A\*, de entender o papel das heurísticas na busca informada, de descrever o funcionamento do Jump Point Search e de **comparar criticamente** as diferentes técnicas de representação e de busca — reconhecendo, para cada problema de jogo, qual abordagem é a mais adequada e por quê. Essa base espacial, além de sustentar o movimento, será reaproveitada na Parte IV, quando os **mapas de influência** transformarem o mesmo grafo do mundo em um raciocínio *tático* sobre onde agir.
