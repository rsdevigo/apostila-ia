# Capítulo 8 — Busca de Caminhos com A\*

## Introdução

Se o Capítulo 7 nos ensinou a **enxergar o mundo como um grafo**, este capítulo nos ensina a **percorrê-lo com inteligência**. O problema agora é preciso: dado um grafo de nós, arestas e custos, uma posição de **origem** e uma de **destino**, como encontrar o **caminho de menor custo** entre elas — e fazê-lo rápido o bastante para caber no orçamento de um quadro de jogo? A resposta que a indústria adotou, de forma quase universal, tem um nome e uma pronúncia próprios: **A\***, lido "A estrela" (do inglês *A-star*).

Poucos algoritmos têm uma presença tão dominante em sua área quanto o A\* tem na busca de caminhos de jogos. Publicado em 1968 por Peter Hart, Nils Nilsson e Bertram Raphael, no contexto da robótica (o robô Shakey, do Stanford Research Institute, precisava planejar rotas pelo laboratório), o A\* atravessou meio século sem ser destronado. Ele está por trás do movimento dos aldeões de *Age of Empires*, dos soldados de qualquer jogo de tiro tático, das unidades de praticamente todo RTS e — por baixo do **NavMesh Agent** da Unity — de uma vasta fração dos jogos 3D já lançados. Compreendê-lo a fundo é, para um profissional de IA de jogos, um conhecimento tão fundamental quanto a própria noção de estado da Parte II.

Este capítulo o constrói **por camadas**, jamais introduzindo um conceito antes de seus pré-requisitos. Partimos do **problema** do menor caminho em tempo real; recuamos até o algoritmo de **Dijkstra** para entender a **busca não-informada** e o que lhe falta; introduzimos a ideia de **heurística** e de **busca informada**, chegando à célebre função de avaliação `f = g + h`; detalhamos o **funcionamento** passo a passo, com as **listas aberta e fechada**, a **reconstrução do caminho** e um **traço de execução** completo; formalizamos as propriedades que explicam *por que* o A\* funciona — **admissibilidade** e **consistência** da heurística — e o impacto de diferentes heurísticas; e, por fim, aterrissamos a teoria nos **jogos conhecidos**, nas **ferramentas** da Unity (NavMesh) e de terceiros (A\* Pathfinding Project) e no **mercado**. Ao final, ficará claro não apenas *como* o A\* funciona, mas *por que* ele se tornou o algoritmo de busca de caminhos mais utilizado da história dos jogos.

> 🕰️ **Contexto Histórico**
> O A\* nasceu para um robô, não para um jogo. Em 1966–1972, o Stanford Research Institute desenvolvia **Shakey**, o primeiro robô móvel capaz de raciocinar sobre as próprias ações. Shakey precisava planejar como se deslocar por um ambiente com obstáculos, e a busca de Dijkstra — embora correta — era lenta demais, pois explorava o espaço em todas as direções sem noção de "para onde fica o objetivo". Hart, Nilsson e Raphael perceberam que, se o algoritmo pudesse **estimar** a distância que ainda faltava até o alvo, ele poderia **priorizar** a exploração na direção certa. Essa estimativa é a **heurística**, e a ideia de combiná-la com o custo já percorrido é toda a genialidade do A\*. Décadas depois, a mesma matemática que guiava um robô desajeitado por um laboratório move milhões de personagens por mundos virtuais.

---

## 8.1 O problema: menor caminho em tempo real

No Capítulo 7, reduzimos o cenário a um grafo. Agora precisamos de um procedimento que, dado esse grafo, uma origem `s` e um destino `t`, devolva a **sequência de nós** que leva de `s` a `t` com o **menor custo total** — respeitando os obstáculos (que já não fazem parte do grafo, pois só há arestas entre lugares realmente conectados) e, idealmente, evitando terrenos caros (via os pesos das arestas).

A dificuldade não está em *existir* uma solução — em um grafo conexo, basta seguir arestas até chegar. A dificuldade está em três exigências simultâneas, que puxam em direções opostas:

1. **Otimalidade (ou quase):** o caminho deve ser o mais curto/barato possível, ou muito próximo disso. Um NPC que faz um desvio absurdo para chegar à porta ao lado destrói a ilusão de inteligência.
2. **Velocidade:** o cálculo deve caber no **orçamento de quadro**. Num jogo a 60 quadros por segundo, há cerca de 16 milissegundos para *tudo* — física, renderização, som, lógica de jogo e IA. A busca de caminho disputa uma fatia mínima desse tempo, muitas vezes para **dezenas de agentes ao mesmo tempo**.
3. **Escalabilidade:** o mesmo algoritmo precisa funcionar num grafo de algumas centenas de nós (um nível pequeno) e num de centenas de milhares (um mundo aberto ou uma grade fina).

> 🎮 **Na Prática**
> A tensão entre **otimalidade** e **velocidade** é o eixo de todo o pathfinding de jogos, e vale internalizá-la agora. Em muitos jogos, um caminho **ligeiramente subótimo calculado em tempo hábil** é infinitamente preferível a um caminho **perfeito que trava o jogo por meio segundo**. Essa é a mesma lógica pragmática da Parte I: a IA de jogos busca o *convincente e barato*, não o *perfeito e caro*. O A\* é celebrado justamente por oferecer o melhor equilíbrio conhecido entre as duas exigências — e por permitir, quando preciso, **trocar deliberadamente** um pouco de otimalidade por muito mais velocidade, como veremos ao discutir heurísticas.

A abordagem ingênua — testar todos os caminhos possíveis e escolher o mais barato — é inviável: o número de caminhos cresce **exponencialmente** com o tamanho do grafo. Precisamos de um método que encontre o melhor caminho **sem** enumerar todos. A família de algoritmos que faz isso é a da **busca em grafos**, e para entender o A\* precisamos primeiro entender seu antecessor direto.

---

## 8.2 Do Dijkstra ao A\*: busca não-informada versus informada

### Busca não-informada: o algoritmo de Dijkstra

Em 1959, Edsger Dijkstra publicou um algoritmo que resolve o problema do caminho de menor custo em grafos com pesos não negativos — e o faz de forma **garantidamente ótima**. A ideia é uma expansão sistemática e cuidadosa: partindo da origem, o algoritmo vai "descobrindo" nós em ordem **crescente de custo acumulado** desde a origem. Ele mantém, para cada nó já alcançado, o **menor custo conhecido** para chegar até ali, e sempre expande em seguida o nó de **menor custo acumulado** ainda não processado. Como os pesos são não negativos, quando um nó é finalmente escolhido para expansão, o custo com que ele foi alcançado é, comprovadamente, o **menor possível** — e nunca precisará ser revisto.

O comportamento visual de Dijkstra é o de uma **mancha que se espalha uniformemente** a partir da origem, como ondas concêntricas em um lago, crescendo em todas as direções até tocar o destino. E aqui está sua fraqueza para jogos: **Dijkstra não sabe onde fica o destino**. Ele explora com o mesmo entusiasmo o nó que fica na direção do alvo e o que fica exatamente na direção oposta. É uma busca **não-informada** (ou "cega"): usa apenas a informação do custo *já percorrido*, nada sobre o que *ainda falta*. Num mapa grande, isso significa examinar uma quantidade enorme de nós irrelevantes antes de alcançar o objetivo — desperdício que um jogo em tempo real não pode pagar.

> ⚠️ **Atenção**
> Dijkstra **não é** um algoritmo ruim — ele é ótimo e continua sendo a escolha certa em situações específicas: quando **não há um destino único** (por exemplo, "qual a distância deste ponto a *todos* os outros?", útil em mapas de influência da Parte IV), ou quando não existe uma boa maneira de estimar a distância ao alvo. O ponto não é que Dijkstra seja inferior, mas que, **quando conhecemos o destino**, estamos jogando fora uma informação preciosa ao não usá-la. É exatamente essa informação desperdiçada que o A\* recupera.

### A ideia da heurística e a busca informada

Suponha que, além do custo já percorrido, o algoritmo pudesse **estimar** quanto ainda falta de cada nó até o destino. Se soubéssemos (mesmo aproximadamente) que o nó X está "perto" do alvo e o nó Y está "longe", faria todo o sentido **explorar X primeiro**. Essa estimativa do custo restante é chamada de **heurística**, tradicionalmente denotada por `h(n)` — uma função que, dado um nó `n`, devolve um "palpite" do custo de `n` até o destino.

Uma busca que usa uma heurística para se orientar é chamada de **busca informada**. A heurística é o que transforma a mancha uniforme de Dijkstra em um **feixe direcionado** que se estica preferencialmente rumo ao objetivo, examinando muito menos nós. O termo "heurística", que reencontraremos em Minimax (Parte V) e em algoritmos genéticos (Parte VI), significa exatamente isto: um **conhecimento aproximado que guia a busca**, sem garantia de ser exato, mas útil o bastante para economizar trabalho.

Um exemplo torna a ideia concreta. Num mapa em grade, uma heurística natural para o custo restante de um nó até o destino é a **distância em linha reta** (ou a distância em número de células) entre eles, **ignorando os obstáculos**. Essa estimativa é fácil de calcular (pura geometria) e, embora possa **subestimar** o custo real (o caminho de fato terá de contornar paredes), ela nunca o **superestima** — propriedade que, como veremos na seção 8.2.2, é a chave de tudo.

> ❌ **Erro Comum**
> Achar que a heurística "decide o caminho" ou que precisa ser exata. A heurística **não** escolhe o caminho; ela apenas **ordena a exploração**, dizendo ao algoritmo qual nó parece mais promissor examinar em seguida. Uma heurística imperfeita (que subestima) ainda leva o A\* ao caminho ótimo — apenas o faz examinando mais ou menos nós conforme sua qualidade. Confundir "guiar a busca" com "determinar a resposta" é um mal-entendido frequente que atrapalha a compreensão do algoritmo.

### 8.2.1 Custo g, heurística h e função f

O A\* é, em essência, a fusão das duas ideias anteriores: ele combina o **custo já percorrido** (a força de Dijkstra, que garante otimalidade) com a **estimativa do custo restante** (a heurística, que dá direção). Formalmente, para cada nó `n` durante a busca, o A\* mantém três valores:

- **`g(n)` — custo acumulado:** o custo do **melhor caminho conhecido** da origem até `n`. É um valor **exato e já pago** — a soma real dos pesos das arestas percorridas para chegar a `n`. É a herança direta de Dijkstra.
- **`h(n)` — heurística (custo estimado restante):** o **palpite** do custo de `n` até o destino. É um valor **estimado e ainda não pago**. É a novidade da busca informada.
- **`f(n)` — função de avaliação:** a soma `f(n) = g(n) + h(n)`. Ela representa a **estimativa do custo total** de um caminho que passe por `n`: o que já se pagou para chegar até `n`, mais o que se estima faltar dali ao destino.

A regra de ouro do A\* é então de uma simplicidade notável: **a cada passo, expanda o nó com o menor valor de `f`.** Ao priorizar o menor `f`, o algoritmo prefere nós que combinam "já cheguei barato até aqui" (`g` pequeno) com "e daqui parece faltar pouco" (`h` pequeno). Essa única regra é o que equilibra, automaticamente, a **cautela** de Dijkstra (não desprezar o custo já pago) com a **ambição** da busca informada (correr na direção do alvo).

Vale observar os dois casos-limite, que iluminam o lugar do A\* no espectro de algoritmos:

- Se `h(n) = 0` para todo nó (nenhuma informação sobre o destino), então `f = g`, e o A\* **degenera exatamente em Dijkstra** — busca ótima, porém cega.
- Se ignorássemos `g` e usássemos `f = h` (só a heurística), teríamos a **Busca Gulosa pelo Melhor Primeiro** (*Greedy Best-First Search*): rapidíssima, corre direto para o alvo, mas **sem garantia de otimalidade** — pode mergulhar num caminho que *parece* curto e revelar-se um beco caro, porque ignora quanto já gastou.

O A\* vive no ponto exato entre esses dois extremos, colhendo o melhor de cada um: a **otimalidade** de Dijkstra e a **direção** da busca gulosa.

[DIAGRAMA]
Título: A função de avaliação f = g + h
Objetivo pedagógico: Fixar visualmente o significado dos três valores do A\* e mostrar que f é uma estimativa do custo total do caminho que passa por um nó.
Descrição detalhada: Desenhar uma linha horizontal representando um caminho, com três marcos: a Origem (à esquerda), um nó intermediário n (no meio) e o Destino (à direita). Do trecho Origem→n, uma chave/colchete inferior rotulada "g(n): custo já percorrido (exato, pago)", desenhada como linha sólida. Do trecho n→Destino, outra chave rotulada "h(n): custo estimado restante (palpite, não pago)", desenhada como linha tracejada para indicar estimativa. Acima de todo o percurso, uma chave abrangente rotulada "f(n) = g(n) + h(n): estimativa do custo total". Ao lado, uma pequena caixa de texto com a regra: "A* expande sempre o nó de MENOR f". Incluir, em rodapé, os dois casos-limite: "h = 0 → Dijkstra (cego, ótimo)" e "ignora g → Guloso (rápido, não ótimo)".
Elementos obrigatórios: os três marcos (origem, n, destino); g como trecho sólido; h como trecho tracejado; f como soma abrangente; a regra do menor f; os dois casos-limite.
[/DIAGRAMA]

### 8.2.2 Admissibilidade e consistência da heurística

O A\* só garante encontrar o caminho **ótimo** se a heurística satisfizer certas propriedades. Compreendê-las é o que separa quem "sabe usar A\*" de quem realmente **entende** o algoritmo.

**Admissibilidade.** Uma heurística é **admissível** se ela **nunca superestima** o custo real restante — ou seja, se `h(n)` é sempre **menor ou igual** ao custo verdadeiro do caminho ótimo de `n` até o destino. Uma heurística admissível é, por isso, chamada de **otimista**: ela pode achar que falta menos do que realmente falta, mas nunca que falta mais. O teorema central do A\*, demonstrado por Hart, Nilsson e Raphael, garante: **se a heurística é admissível, o A\* encontra o caminho de menor custo.** A intuição é que, sendo otimista, a heurística nunca faz o algoritmo *descartar cedo demais* um caminho que na verdade era bom — ela pode enganar para menos, adiando a descoberta, mas nunca para mais, o que faria perder a solução ótima.

O exemplo canônico de heurística admissível é a **distância em linha reta** (ou a distância de grade correspondente) ignorando obstáculos: como os obstáculos só podem **aumentar** o custo real (obrigando desvios), a linha reta é sempre um limite **inferior** do custo verdadeiro. Ela subestima — é otimista — e, portanto, é admissível.

**Consistência (ou monotonicidade).** Uma propriedade mais forte é a **consistência**: a heurística é consistente se, para toda aresta de `n` para um vizinho `m` com custo `c(n, m)`, vale `h(n) ≤ c(n, m) + h(m)`. Em palavras: a estimativa de `n` não pode ser maior que o custo de dar um passo até `m` mais a estimativa de `m` — uma espécie de "desigualdade triangular" aplicada à heurística. A consistência garante que, ao longo de qualquer caminho, o valor de `f` **nunca diminui** (é monotônico). Sua consequência prática é valiosa: **com uma heurística consistente, um nó nunca precisa ser reprocessado** depois de fechado — a primeira vez que o A\* fecha um nó já é com seu custo `g` ótimo. Isso simplifica a implementação e melhora o desempenho.

Toda heurística consistente é admissível, mas nem toda admissível é consistente. Felizmente, as heurísticas geométricas usadas em jogos (Manhattan, Euclidiana, Chebyshev, que veremos na seção 8.3.2) são, na prática, **consistentes** — o que explica por que implementações de A\* em jogos raramente se preocupam em reabrir nós fechados.

> ⚠️ **Atenção**
> A distinção entre admissível e consistente costuma parecer abstrata, mas tem uma consequência concreta de implementação. Se sua heurística é apenas admissível (não consistente), o A\* pode, em raras situações, encontrar um caminho melhor para um nó **já fechado** e precisará **reabri-lo** para propagar a melhoria — caso contrário, o resultado pode não ser ótimo. Com heurística **consistente**, isso nunca acontece, e você pode ignorar com segurança qualquer nó já na lista fechada. Como as heurísticas geométricas comuns são consistentes, a maioria dos tutoriais de jogos omite o reprocessamento — o que é correto *naquele contexto*, mas não em geral.

> ✅ **Boa Prática**
> Ao projetar uma heurística, mantenha-a **admissível** para preservar a otimalidade e o mais **próxima possível** do custo real para maximizar a eficiência (menos nós expandidos). Há uma tensão elegante aqui: a heurística "perfeita" seria o próprio custo ótimo restante — mas calculá-la seria tão caro quanto resolver o problema. A arte está em encontrar uma estimativa **barata de calcular** e ao mesmo tempo **informativa**. Toda a seção 8.3.2 é sobre esse equilíbrio.

---

## 8.3 Funcionamento passo a passo de A\*

Com os conceitos de `g`, `h`, `f` e admissibilidade no lugar, podemos descrever a mecânica do algoritmo. O A\* organiza a busca em torno de duas coleções de nós — as **listas aberta e fechada** — e de um registro de "de onde vim" que permitirá reconstruir o caminho ao final.

### 8.3.1 Listas aberta e fechada

A **lista aberta** (*open list*) contém os nós **descobertos, mas ainda não expandidos** — a "fronteira" da busca, os candidatos a serem examinados. A cada iteração, o algoritmo retira da lista aberta o nó de **menor `f`**. Por isso, a lista aberta é implementada, na prática, como uma **fila de prioridade** (frequentemente um *heap binário*), estrutura que devolve eficientemente o elemento de menor valor — um detalhe de implementação que impacta diretamente o desempenho, discutido em Cormen et al.

A **lista fechada** (*closed list*) contém os nós **já expandidos** — aqueles cujo custo ótimo já foi determinado (sob heurística consistente) e que não precisam ser reexaminados. Sua função é evitar o **reprocessamento** e impedir **ciclos** (voltar a um nó de onde já se partiu).

Para cada nó, o algoritmo guarda ainda um ponteiro para seu **predecessor** — o nó a partir do qual ele foi alcançado pelo melhor caminho conhecido. É esse encadeamento de "de onde vim" que permitirá reconstruir a rota (seção 8.3.3).

O laço principal do A\* pode ser descrito em pseudocódigo enxuto — e aqui vale a regra da apostila: o código aparece apenas para **esclarecer o conceito**, não como implementação a ser copiada.

```
função A*(origem, destino):
    listaAberta ← { origem }          // fila de prioridade por f
    listaFechada ← { }
    g(origem) ← 0
    f(origem) ← h(origem)

    enquanto listaAberta não estiver vazia:
        atual ← nó de menor f na listaAberta
        se atual = destino:
            retornar reconstruirCaminho(atual)   // achou!

        mover atual de listaAberta para listaFechada

        para cada vizinho de atual:
            se vizinho está em listaFechada:
                continuar               // já resolvido
            custoTentativa ← g(atual) + custoAresta(atual, vizinho)
            se custoTentativa < g(vizinho):        // caminho melhor até vizinho
                predecessor(vizinho) ← atual
                g(vizinho) ← custoTentativa
                f(vizinho) ← g(vizinho) + h(vizinho)
                se vizinho não está em listaAberta:
                    adicionar vizinho à listaAberta

    retornar falha                     // destino inalcançável
```

Note o coração do algoritmo: para cada vizinho do nó atual, calcula-se o custo de alcançá-lo **passando pelo atual** (`custoTentativa`); se esse custo for **melhor** do que o melhor conhecido até então para aquele vizinho, atualiza-se seu `g`, seu `f` e seu predecessor. Essa operação — chamada de **relaxamento** da aresta, no vocabulário de Cormen et al. — é a mesma de Dijkstra; a única diferença do A\* é que a **ordem de expansão** é ditada por `f = g + h`, e não apenas por `g`.

> 🎮 **Na Prática**
> A eficiência do A\* depende criticamente da estrutura de dados da lista aberta. Uma implementação ingênua que percorre toda a lista para achar o menor `f` a cada passo custa caro; uma **fila de prioridade** (heap binário) reduz drasticamente esse custo e é o padrão. Em motores profissionais, otimizam-se ainda mais essas estruturas (heaps especializados, buckets de custo), porque o pathfinding é chamado tantas vezes por segundo que cada microssegundo economizado no gerenciamento da lista aberta se multiplica por milhares.

[DIAGRAMA]
Título: O ciclo de expansão do A\* com listas aberta e fechada
Objetivo pedagógico: Ilustrar o laço central do algoritmo — retirar o menor f da lista aberta, expandir vizinhos, relaxar arestas e mover para a lista fechada.
Descrição detalhada: Desenhar um fluxograma vertical. Caixa 1: "Lista aberta vazia? → se sim, FALHA (destino inalcançável)". Caixa 2: "Retirar da lista aberta o nó ATUAL de menor f". Losango de decisão: "ATUAL é o destino? → se sim, RECONSTRUIR CAMINHO e terminar". Caixa 3: "Mover ATUAL para a lista fechada". Caixa 4: "Para cada vizinho de ATUAL não fechado: calcular custoTentativa = g(atual) + custo(atual,vizinho)". Losango: "custoTentativa < g(vizinho)? → se sim: atualizar g, f e predecessor; inserir vizinho na lista aberta". Seta de retorno da última caixa de volta à Caixa 1, formando o laço. Ao lado do fluxograma, duas colunas visuais representando o estado das listas (Aberta = fronteira; Fechada = resolvidos) com alguns nós de exemplo.
Elementos obrigatórios: laço fechado; retirada do menor f; teste de destino; movimentação para a fechada; relaxamento condicional dos vizinhos; representação lateral das duas listas.
[/DIAGRAMA]

### 8.3.2 Heurísticas comuns (Manhattan, Euclidiana, Chebyshev)

A qualidade da heurística determina quantos nós o A\* expande — e, portanto, sua velocidade. Em grades, três heurísticas geométricas dominam, e a escolha entre elas depende de **quais movimentos são permitidos** (conectividade-4, conectividade-8). Todas medem a distância ao destino **ignorando obstáculos**, o que as torna admissíveis.

**Distância de Manhattan.** Soma das diferenças absolutas nas coordenadas: `h = |x₁ − x₂| + |y₁ − y₂|`. É a distância percorrida por quem só pode andar em passos **ortogonais** (norte/sul/leste/oeste), como um táxi no quadriculado das ruas de Manhattan — daí o nome. É a heurística correta para grades de **conectividade-4**, nas quais não há movimento diagonal. Usá-la em conectividade-8, porém, a torna **inadmissível** (superestima), pois ignora que a diagonal encurta caminho.

**Distância Euclidiana.** A distância em **linha reta** clássica: `h = √((x₁ − x₂)² + (y₁ − y₂)²)`. É admissível em qualquer conectividade, pois nenhum caminho por arestas pode ser mais curto que a reta. É a heurística natural quando o movimento é **livre em qualquer ângulo** (típico de NavMesh e de espaços contínuos). Sua desvantagem é o custo da raiz quadrada e o fato de, em grades de 8 direções, **subestimar demais** (é otimista em excesso), fazendo o A\* expandir mais nós que o necessário.

**Distância de Chebyshev (e a "octile").** Para grades de **conectividade-8** com custo diagonal igual ao ortogonal, a distância de Chebyshev — `h = máx(|x₁ − x₂|, |y₁ − y₂|)` — é a medida exata (o número de passos, já que a diagonal cobre avanço em ambos os eixos de uma vez). Quando o custo diagonal é o correto `√2`, usa-se a variante **octile**, que pondera adequadamente os trechos diagonais e ortogonais e é a heurística **exata e admissível** para movimento em 8 direções — a escolha ideal para a maioria das grades de jogo.

> ❌ **Erro Comum**
> **Casar a heurística errada com a conectividade da grade.** Usar Manhattan numa grade de 8 direções superestima o custo restante, tornando a heurística **inadmissível** — e o A\* pode devolver um caminho **não ótimo**. Usar Euclidiana numa grade de 8 direções é admissível, mas **subestima**, fazendo o algoritmo trabalhar mais do que precisa. A regra prática: **conectividade-4 → Manhattan; conectividade-8 → octile (Chebyshev ponderada); movimento livre/NavMesh → Euclidiana.** Errar esse casamento é uma das causas mais comuns de pathfinding lento ou de caminhos estranhos.

**O impacto da heurística — e o truque do "peso".** Quanto mais a heurística se aproxima do custo real (sem superestimá-lo), menos nós o A\* expande e mais rápido ele termina. No extremo, uma heurística nula (`h = 0`) reduz o A\* a Dijkstra, expandindo o máximo de nós. Há ainda um recurso pragmático importantíssimo na indústria: **multiplicar a heurística por um peso** maior que 1 (o chamado **A\* ponderado**, ou *weighted A\**). Isso a torna **deliberadamente inadmissível** — ela passa a superestimar — e o algoritmo fica **muito mais rápido**, correndo com mais agressividade rumo ao alvo, ao custo de **abrir mão da garantia de otimalidade** (o caminho pode ficar um pouco mais longo). Em muitos jogos, esse é um negócio excelente: um caminho 5% mais longo calculado na metade do tempo é, frequentemente, a escolha profissional correta. É a materialização, no nível do algoritmo, da tese de que a IA de jogos prefere o *convincente e barato* ao *perfeito e caro*.

> 🏭 **Na Indústria**
> O A\* ponderado (*weighted A\**) é um dos segredos abertos do pathfinding comercial. Estúdios ajustam o peso da heurística como um **dial** entre qualidade e desempenho: quando há muitos agentes buscando ao mesmo tempo e o orçamento de quadro aperta, aumenta-se o peso para acelerar; quando a qualidade do caminho é crítica e há folga de tempo, aproxima-se de 1 para recuperar a otimalidade. Essa sintonia fina, invisível ao jogador, é parte do ofício de um programador de IA — e um bom exemplo de como um parâmetro matemático simples vira uma alavanca de design.

[DIAGRAMA]
Título: Comparação de heurísticas em uma grade
Objetivo pedagógico: Mostrar visualmente como Manhattan, Euclidiana e Chebyshev medem a distância entre dois pontos de uma grade e como cada uma corresponde a um conjunto de movimentos permitidos.
Descrição detalhada: Desenhar três cópias de uma mesma grade pequena (8×8), cada uma com a célula de origem (verde) no canto inferior esquerdo e a de destino (vermelha) deslocada 5 células à direita e 3 para cima. Painel 1 — Manhattan: traçar o caminho em "escada" só com passos ortogonais e anotar h = 5 + 3 = 8; legenda "movimento em 4 direções". Painel 2 — Chebyshev/octile: traçar 3 passos diagonais + 2 ortogonais e anotar h = máx(5,3) = 5 passos (ou 3·1,41 + 2·1,0 na variante octile); legenda "movimento em 8 direções". Painel 3 — Euclidiana: traçar uma linha reta diagonal atravessando as células e anotar h = √(5² + 3²) ≈ 5,83; legenda "movimento livre em qualquer ângulo". Sob os três painéis, uma faixa-resumo: "cada heurística é exata para o seu conjunto de movimentos; usar a errada torna o A* inadmissível ou lento".
Elementos obrigatórios: três grades com mesma origem e destino; caminho e valor de h de cada heurística; associação de cada uma ao seu conjunto de movimentos; faixa-resumo.
[/DIAGRAMA]

### 8.3.3 Reconstrução do caminho

Quando o A\* retira o **destino** da lista aberta, a busca termina — mas o resultado imediato não é o caminho, e sim o registro de **predecessores** construído ao longo do processo. Para obter a rota, o algoritmo faz a **reconstrução do caminho**: parte do nó destino, segue o ponteiro de predecessor até o nó anterior, dali para o anterior deste, e assim por diante, até chegar à origem (cujo predecessor é nulo). Isso produz a sequência de nós **em ordem reversa** (do destino à origem); basta **invertê-la** para obter o caminho da origem ao destino.

Essa etapa é barata — percorre apenas os nós do caminho final, não todos os explorados — e elegante: toda a informação necessária já foi registrada como efeito colateral do relaxamento das arestas. É por isso que guardar o predecessor a cada melhoria de `g` (na seção 8.3.1) não era um detalhe, mas parte essencial do projeto do algoritmo.

> ✅ **Boa Prática**
> Separe mentalmente **três produtos** de uma busca A\*: (1) o **conjunto de nós explorados** (aberta + fechada), que mede o *esforço* gasto; (2) o **encadeamento de predecessores**, que é a *memória* de como cada nó foi alcançado; e (3) o **caminho final reconstruído**, que é a *resposta* entregue ao agente. Confundir "nós explorados" com "caminho" é um erro conceitual comum — o A\* costuma explorar muito mais nós do que os que aparecem no caminho devolvido, e a diferença entre esses dois números é justamente o que as otimizações do Capítulo 9 buscam reduzir.

---

## 8.4 Exemplos e traços de execução

Nada fixa o A\* como acompanhar sua execução célula a célula. Considere uma grade de conectividade-4 (movimento ortogonal, custo 1 por passo), com heurística de **Manhattan**, uma origem `S`, um destino `T` e uma parede vertical entre eles, com uma única passagem. Acompanhemos o raciocínio do algoritmo em linhas gerais.

No início, apenas `S` está na lista aberta, com `g(S) = 0` e `f(S) = h(S)` (a distância de Manhattan de `S` a `T`). O A\* expande `S`, calcula `g = 1` para cada vizinho livre e, para cada um, `f = 1 + h`. Os vizinhos que ficam **na direção de `T`** recebem `f` menor (porque seu `h` é menor) e, portanto, são preferidos nas próximas expansões. Enquanto o caminho reto está livre, a busca avança **quase em linha** rumo à parede, expandindo pouquíssimos nós fora da rota — é a heurística fazendo seu trabalho de direcionar.

Ao encontrar a **parede**, algo revelador acontece: os nós logo atrás dela têm `h` baixo (estão geometricamente perto de `T`), mas todo caminho real até eles é bloqueado. O A\* tenta insistir naquela direção — afinal, o `h` os faz parecer promissores —, mas cada tentativa esbarra no obstáculo. Aos poucos, os nós ao longo da parede vão sendo expandidos e seu `g` cresce (o desvio custa passos), até que o algoritmo "percebe", pela aritmética de `f`, que **contornar pela passagem** tem custo total menor do que continuar espremendo contra a parede. A busca então se **espalha lateralmente** até a abertura, atravessa-a e retoma o avanço direto até `T`. Quando `T` sai da lista aberta, a reconstrução devolve o caminho que sobe reto, desvia até a passagem, cruza e segue reto ao alvo — exatamente a rota que um humano traçaria.

Esse traço ilustra o comportamento característico do A\*: **eficiente e direto em terreno aberto** (a heurística poda quase tudo), **mais trabalhoso perto de obstáculos** (onde a heurística "mente" ao sugerir direções bloqueadas), mas sempre **correto e ótimo** (sob heurística admissível). Também antecipa a motivação do Capítulo 9: em grades grandes e abertas, o A\* ainda expande **muitos nós redundantes** — vizinhos equivalentes que levam ao mesmo lugar —, e é essa redundância que o Jump Point Search atacará.

> 🎲 **Curiosidade**
> Se você já jogou um RTS e mandou uma unidade para o outro lado do mapa, deve ter notado que, às vezes, ela hesita ou faz um pequeno desvio ao contornar uma construção. O que você viu foi, muito provavelmente, o A\* "descobrindo" em tempo real que a direção mais curta em linha reta estava bloqueada e recalculando o contorno. Esse comportamento não é um defeito — é a assinatura visível de uma busca informada lidando com a diferença entre a **distância geométrica** (o que a heurística estima) e a **distância real navegável** (o que o grafo impõe).

[DIAGRAMA]
Título: Traço de execução do A\* contornando uma parede
Objetivo pedagógico: Visualizar como o A\* expande poucos nós em terreno aberto e mais nós perto do obstáculo, e como o caminho ótimo emerge.
Descrição detalhada: Desenhar uma grade 10×10. Origem S no lado esquerdo, destino T no lado direito, e uma parede vertical (células bloqueadas em cinza-escuro) entre eles, deixando uma única passagem na parte inferior. Colorir as células conforme o papel na busca: em tom claro, as células da LISTA ABERTA no momento retratado (fronteira); em tom médio, as da LISTA FECHADA (já expandidas); deixar em branco as nunca tocadas. Mostrar a concentração de células fechadas formando um "leque" que avança reto até a parede e depois se espalha lateralmente rumo à passagem. Sobrepor, em linha destacada (por exemplo, amarela), o CAMINHO FINAL reconstruído de S a T, subindo/descendo até a passagem, cruzando e seguindo a T. Anotar em três ou quatro células o par de valores (g, h) para ilustrar o cálculo de f. Incluir legenda das cores (aberta, fechada, não tocada, caminho final).
Elementos obrigatórios: grade com parede e passagem única; células coloridas por lista (aberta/fechada/não tocada); leque de expansão; caminho final destacado; valores (g,h) em algumas células; legenda.
[/DIAGRAMA]

---

## 8.5 Vantagens e limitações

**Vantagens.** O A\* reúne um conjunto raro de qualidades que explicam seu domínio. É **ótimo** sob heurística admissível — devolve comprovadamente o caminho de menor custo. É **completo** — se existe um caminho, ele o encontra; se não existe, ele reporta a falha. É **geral** — funciona sobre *qualquer* grafo (grade, waypoints, NavMesh), pois nada em sua mecânica depende da representação, apenas de "nós, vizinhos e custos". É **eficiente** — entre os algoritmos que garantem otimalidade com uma dada heurística, o A\* é, num sentido preciso demonstrado na literatura, o que expande o menor número de nós (resultado que vale para heurística **consistente** e a menos de critérios de desempate entre nós de mesmo `f`). E é **sintonizável** — via a escolha e o peso da heurística, permite negociar otimalidade por velocidade conforme a necessidade do jogo.

**Limitações.** O A\* guarda `g`, `f` e predecessor para **cada nó tocado**, então seu **consumo de memória** cresce com o número de nós explorados — um problema real em grades muito finas ou mundos enormes. Seu **tempo de execução**, embora muito melhor que o de Dijkstra, ainda pode ser proibitivo quando **muitos agentes** buscam simultaneamente em grafos grandes. Em grades uniformes e abertas, ele desperdiça trabalho explorando **nós simétricos** (rotas equivalentes que chegam ao mesmo ponto) — a ineficiência que motiva o Capítulo 9. O A\* clássico calcula um caminho **estático**: se o mundo muda (uma ponte desaba, um obstáculo surge), é preciso **recalcular** — o que originou variantes dinâmicas como o **D\*** e o **D\* Lite**, usadas em robótica e em alguns jogos. E o caminho que ele devolve, especialmente em grades, tende a ter aparência **"quadriculada"** e a "colar" nas quinas, exigindo a **suavização** que estudaremos no Capítulo 9.

> ⚠️ **Atenção**
> Uma limitação frequentemente subestimada é o **custo de memória por agente**. Em um RTS com centenas de unidades pedindo caminho ao mesmo tempo, não é o tempo de uma única busca que preocupa, mas a **soma** de todas elas e a memória temporária que cada uma consome. Por isso, motores comerciais raramente deixam cada agente rodar um A\* completo e independente a cada quadro: eles **enfileiram** requisições, **compartilham** resultados entre agentes com destinos próximos, **fatiam** buscas longas em vários quadros (*time-slicing*) e **reaproveitam** caminhos. O A\* é o núcleo, mas ao redor dele há toda uma engenharia de orçamento — tema que o Capítulo 9 aprofunda.

---

## 8.6 Jogos conhecidos e aplicações

O A\* é, provavelmente, o algoritmo de IA mais onipresente da história dos jogos, ainda que quase sempre invisível. Alguns campos de aplicação tornam sua presença concreta:

Nos **jogos de estratégia em tempo real (RTS)** — a família de *Age of Empires*, *StarCraft*, *Command & Conquer* —, o A\* (ou variantes otimizadas dele) move cada unidade que o jogador clica para uma posição. É aqui que suas limitações de **escala** mais aparecem: dezenas ou centenas de unidades pedindo caminho ao mesmo tempo, em mapas grandes, tornaram esses jogos os principais motores de inovação em pathfinding — muitas otimizações do Capítulo 9 nasceram justamente da necessidade de fazer exércitos inteiros se moverem sem travar o jogo.

Nos **jogos de ação e tiro** — de clássicos como *F.E.A.R.* aos títulos 3D contemporâneos —, o A\* roda sobre a NavMesh para conduzir inimigos que perseguem, flanqueiam e recuam. A qualidade tática desse movimento (o inimigo que aparece pela lateral) combina o A\* com os **custos de aresta** e o raciocínio espacial que veremos na Parte IV.

Em praticamente **todo jogo 3D com NPCs que se deslocam** — incluindo os grandes títulos de mundo aberto e narrativos, como *The Last of Us* —, é um A\* sobre NavMesh, encapsulado no equivalente ao **NavMesh Agent**, que garante que companheiros e inimigos cheguem aonde precisam sem atravessar paredes.

> 🏭 **Na Indústria**
> Um ponto de honestidade profissional, alinhado à cautela da apostila: raramente um estúdio publica "usamos A\* com heurística octile e peso 1,3". O que sabemos é que o A\* é o **padrão de fato** e que os sistemas oficiais de navegação (Unity, Unreal) o implementam por baixo. Quando atribuímos A\* a um jogo específico, fazemos uma **inferência técnica fundamentada** na estrutura observável do movimento (contorno de obstáculos, caminhos próximos do ótimo, comportamento em recálculo), não uma afirmação documentada — a mesma distinção entre *fato* e *análise* que praticaremos na engenharia reversa da Parte VII.

---

## 8.7 Ferramentas: NavMesh (Unity) e A\* Pathfinding Project (terceiros)

Como no Capítulo 7, o objetivo aqui é **conceitual**, não um passo a passo. Na Unity, o A\* não é exposto como um algoritmo que o desenvolvedor implementa, mas como um serviço **embutido no NavMesh Agent**: ao definir um destino (a propriedade de destino do agente), internamente o sistema executa uma busca da família A\* sobre o **grafo de polígonos** da NavMesh gerada no Capítulo 7, obtém a sequência de polígonos e depois refina o trajeto dentro deles. Os **custos por área** (NavMesh Areas/Modifiers) que estudamos são exatamente os **pesos de aresta** que o A\* soma em `g`; os **off-mesh links** são arestas adicionais no grafo que o A\* atravessa como qualquer outra. Ou seja: tudo o que este capítulo ensinou já está rodando quando o estudante arrasta um NavMesh Agent para a cena — a diferença é que agora ele **entende o que acontece por baixo**.

Quando é preciso **controle direto** sobre o A\* — tipicamente em jogos baseados em **grade** (estratégia, 2D, tower defense) ou que exigem **atualização dinâmica** intensa —, a biblioteca de terceiros mais usada no ecossistema Unity é o **A\* Pathfinding Project**, de Aron Granberg. Ela expõe o algoritmo sobre grades, grafos de pontos e malhas, permite ajustar heurística e conectividade, e implementa otimizações modernas (incluindo variantes hierárquicas e multithreading) — sendo a escolha natural de projetos cujo mundo é melhor descrito por uma grade do que por uma NavMesh. No código aberto, o par **Recast & Detour** oferece o **Detour** como o componente que realiza a busca (um A\* sobre a malha do Recast), usado diretamente por equipes com tecnologia própria.

> ✅ **Boa Prática**
> Antes de escrever seu próprio A\*, pergunte-se se o problema **realmente** exige isso. Para movimento 3D padrão, o NavMesh Agent já entrega um A\* robusto e testado; reinventá-lo raramente compensa. Implementar o A\* "na mão" faz sentido em três situações: (1) para **aprender** — e todo estudante de IA de jogos deveria fazê-lo ao menos uma vez, para desmistificar o algoritmo; (2) quando a representação é uma **grade** que o sistema nativo não cobre bem; (3) quando se precisa de **controle fino** (heurística customizada, custos dinâmicos complexos, *time-slicing* sob medida). Fora desses casos, usar a implementação madura da ferramenta é a decisão profissional.

---

## Resumo

Este capítulo construiu, do fundamento à aplicação, o algoritmo **A\***, o coração da busca de caminhos em jogos. Partimos do **problema** do menor caminho em tempo real e de sua tensão essencial — **otimalidade × velocidade × escala** — dentro do orçamento de quadro. Recuamos ao algoritmo de **Dijkstra**, uma **busca não-informada** que garante o caminho ótimo, mas explora o grafo **cegamente**, em todas as direções, por ignorar onde fica o destino.

Introduzimos a **heurística** `h(n)` — a estimativa do custo restante — que transforma a busca cega em **busca informada**, e chegamos à ideia central do A\*: combinar o **custo acumulado** `g(n)` (exato, herdado de Dijkstra) com a heurística `h(n)` (estimada) na **função de avaliação** `f(n) = g(n) + h(n)`, expandindo sempre o nó de **menor `f`**. Vimos que o A\* é o ponto de equilíbrio entre dois extremos: com `h = 0` ele vira **Dijkstra** (ótimo, cego); usando só `h` ele vira a **Busca Gulosa** (rápida, não ótima).

Formalizamos as propriedades que garantem a otimalidade: a **admissibilidade** (a heurística nunca superestima — é otimista), condição do teorema que assegura o caminho ótimo, e a **consistência** (desigualdade triangular), que dispensa o reprocessamento de nós fechados. Detalhamos o **funcionamento** com a **lista aberta** (fronteira, implementada como fila de prioridade por `f`), a **lista fechada** (nós resolvidos), o **relaxamento** de arestas e a **reconstrução do caminho** via ponteiros de predecessor. Estudamos as três **heurísticas** de grade — **Manhattan** (conectividade-4), **octile/Chebyshev** (conectividade-8) e **Euclidiana** (movimento livre/NavMesh) — e a importância de **casar** a heurística com a conectividade, além do recurso pragmático do **A\* ponderado**, que troca otimalidade por velocidade.

Percorremos um **traço de execução** contornando uma parede, vimos as **vantagens** (ótimo, completo, geral, eficiente, sintonizável) e as **limitações** (memória por nó, custo com muitos agentes, exploração de nós simétricos, recálculo em mundos dinâmicos, caminhos "quadriculados"), e ancoramos tudo nos **jogos** (RTS, ação, mundo aberto) e nas **ferramentas** (NavMesh Agent da Unity; A\* Pathfinding Project e Recast/Detour de terceiros). O leitor sai apto a explicar, traçar e sintonizar o A\* — e preparado para o Capítulo 9, que atacará justamente a redundância de nós simétricos que o A\* clássico ainda deixa sobre a mesa.

## Exercícios de Fixação

1. Explique a diferença entre **busca não-informada** e **busca informada**, usando Dijkstra e A\* como exemplos. Que informação o A\* usa que Dijkstra ignora?
2. Defina `g(n)`, `h(n)` e `f(n)` com precisão. Qual dos três é **exato** e qual é **estimado**? Por que o A\* expande sempre o nó de **menor `f`**?
3. Mostre que, com `h(n) = 0` para todo nó, o A\* se comporta **exatamente** como Dijkstra. O que se ganha e o que se perde ao aumentar a qualidade da heurística?
4. Defina **heurística admissível**. Enuncie o teorema que relaciona admissibilidade à otimalidade do A\*. Por que a **distância em linha reta** ignorando obstáculos é sempre admissível?
5. Distinga **admissibilidade** de **consistência**. Qual consequência prática de implementação a consistência traz (pense na lista fechada)?
6. Para uma grade de **conectividade-8** com custo diagonal `√2`, qual heurística é a exata e admissível? O que acontece com a otimalidade do A\* se, nessa grade, usarmos a distância de **Manhattan**? Justifique.
7. Descreva o papel das **listas aberta e fechada**. Por que a lista aberta é implementada como uma **fila de prioridade**, e não como uma lista comum?
8. Explique, passo a passo, a **reconstrução do caminho** ao final da busca. Por que guardar o **predecessor** de cada nó, durante o relaxamento, é indispensável para essa etapa?
9. O que é o **A\* ponderado** (*weighted A\**)? Ele preserva a otimalidade? Em que situação de jogo trocar otimalidade por velocidade é uma boa decisão? Relacione com a filosofia da Parte I.
10. Um jogo de estratégia trava por meio segundo quando o jogador seleciona 200 unidades e as move ao mesmo tempo. Explique por que o problema não é o A\* de uma única unidade, e cite ao menos **três** estratégias de engenharia (fora do algoritmo em si) para caber no orçamento de quadro.

## Referências

HART, Peter E.; NILSSON, Nils J.; RAPHAEL, Bertram. A Formal Basis for the Heuristic Determination of Minimum Cost Paths. *IEEE Transactions on Systems Science and Cybernetics*, v. 4, n. 2, p. 100–107, 1968.

DIJKSTRA, Edsger W. A Note on Two Problems in Connexion with Graphs. *Numerische Mathematik*, v. 1, p. 269–271, 1959.

CORMEN, Thomas H.; LEISERSON, Charles E.; RIVEST, Ronald L.; STEIN, Clifford. *Algoritmos: Teoria e Prática.* 3. ed. Rio de Janeiro: Elsevier, 2012.

RUSSELL, Stuart J.; NORVIG, Peter. *Inteligência Artificial.* 3. ed. Rio de Janeiro: Elsevier, 2013.

MILLINGTON, Ian. *AI for Games.* 3. ed. Boca Raton: CRC Press, 2019.

BOURG, David M.; SEEMANN, Glenn. *AI for Game Developers.* Sebastopol, CA: O'Reilly Media, 2004.

RABIN, Steve (org.). *Game AI Pro 3: Collected Wisdom of Game AI Professionals.* Boca Raton: A K Peters/CRC Press, 2017.

UNITY TECHNOLOGIES. *Documentação oficial: AI Navigation (NavMesh Agent, NavMesh Areas).*

GRANBERG, Aron. *A\* Pathfinding Project — Documentação.*
