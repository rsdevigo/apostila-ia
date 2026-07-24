# Capítulo 9 — Otimização de Pathfinding: JPS+ e Técnicas Avançadas

## Introdução

O Capítulo 8 nos deu o A\*, o melhor algoritmo de busca de caminhos que garante otimalidade com uma dada heurística. Poderíamos parar por aí — e, por décadas, muitos jogos pararam. Mas há uma pergunta que qualquer profissional de IA de jogos acaba enfrentando: *e quando o A\*, mesmo bem implementado, ainda é lento demais?* Grades cada vez maiores, mundos abertos, exércitos de centenas de unidades pedindo caminho ao mesmo tempo — em algum ponto, o custo do A\* clássico deixa de caber no orçamento de quadro, e é preciso ir além.

Este capítulo é sobre **ir além do A\* sem abandoná-lo**. Nenhuma das técnicas que veremos substitui o A\* como fundamento; todas o **aceleram**, explorando estruturas especiais do problema — a regularidade de uma grade, a estabilidade de um cenário estático, a hierarquia natural de um mapa. A estrela do capítulo é o **Jump Point Search (JPS)** e sua variante pré-processada **JPS+**, que atacam um desperdício específico do A\* em grades uniformes: a exploração de **caminhos simétricos**, rotas equivalentes que levam ao mesmo lugar e que o A\* examina uma a uma, sem perceber que são redundantes. Ao lado dele, veremos um arsenal de otimizações maduras da indústria: **pathfinding hierárquico**, **pré-processamento**, busca **em blocos** e **suavização de caminho (path smoothing)**.

Mas há uma lição tão importante quanto os algoritmos, e a apostila insiste nela desde já: **otimização não é gratuita e nem sempre é apropriada**. Cada técnica deste capítulo compra velocidade com alguma moeda — restrição de aplicabilidade, custo de pré-processamento, memória adicional, tempo de construção, dificuldade de atualização dinâmica. O profissional maduro não é quem aplica a otimização mais sofisticada, mas quem sabe **quando ela rende ganhos reais e quando é esforço desperdiçado**. Fiel à estrutura da apostila, partimos do **problema** (o A\* é caro em grades grandes), construímos os **fundamentos** do JPS e do JPS+, comparamos **desempenho** com o A\*, apresentamos as **demais otimizações**, pesamos **vantagens e limitações**, e aterrissamos nas **aplicações**, **jogos** e **ferramentas**. Ao final, o leitor terá não só mais algoritmos no repertório, mas o **critério** para escolher entre eles.

> 🕰️ **Contexto Histórico**
> O Jump Point Search é surpreendentemente recente para um tema tão maduro quanto o pathfinding. Foi apresentado em 2011 por **Daniel Harabor** e **Alban Grastien**, pesquisadores australianos, num artigo que mostrava como podar simetrias em grades uniformes e alcançar acelerações de uma ordem de magnitude sobre o A\* — **sem** abrir mão da otimalidade. Poucos anos depois, **Steve Rabin** e **Nathan Sturtevant** popularizaram na indústria a variante **JPS+**, com pré-processamento, e a combinaram com *Goal Bounding* para ganhos ainda maiores, documentando-a na série *Game AI Pro*. É um raro exemplo de avanço acadêmico que migrou rapidamente para a prática comercial — e uma prova de que até algoritmos "resolvidos" como a busca em grade ainda guardavam ganhos expressivos a descobrir.

---

## 9.1 O problema: A\* é caro em grades grandes e uniformes

O A\* é ótimo e eficiente — no sentido de expandir o menor número de nós **para uma dada heurística**. Mas "o menor número de nós possível com aquela heurística" ainda pode ser um número **enorme**. E há um cenário em que esse número explode de forma particularmente frustrante: a **grade uniforme, grande e aberta**.

O problema tem um nome: **simetria de caminhos**. Numa grade sem obstáculos, existem **incontáveis caminhos de mesmo custo** entre dois pontos. Para ir de um ponto a outro deslocado 3 células à direita e 3 para cima (em conectividade-8), tanto faz alternar "diagonal, diagonal, diagonal" quanto "direita, cima, diagonal, direita, cima" e dezenas de outras combinações — **todas custam o mesmo**. O A\* não sabe disso. Ele trata cada uma dessas rotas equivalentes como uma possibilidade distinta a ser explorada, expandindo uma multidão de nós que são, para efeitos práticos, **intercambiáveis**. É como se, para escolher entre dois caminhos idênticos, alguém insistisse em percorrer os dois inteiros antes de decidir que dava no mesmo.

Esse desperdício é invisível em grades pequenas, mas cresce com a área. Numa grade de 1000×1000 células (um milhão de nós) — nada absurdo para um jogo de estratégia ou um mundo aberto —, uma única busca do A\* pode expandir **dezenas ou centenas de milhares** de nós, a maioria deles explorando simetrias inúteis. Multiplique isso por dezenas de agentes buscando por quadro, e o orçamento de 16 milissegundos evapora.

> 🎮 **Na Prática**
> A simetria de caminhos é fácil de ver com um experimento mental. Imagine o A\* buscando num campo totalmente aberto, sem nenhum obstáculo, do canto ao centro. O caminho ótimo é trivial — uma reta diagonal —, mas o A\* não "sabe" que é trivial: ele expande um **leque** gordo de nós em torno da diagonal, porque todos têm `f` empatado ou quase. Todo esse trabalho produz uma resposta que poderíamos ter dado de imediato. O JPS nasce exatamente desta observação: *em regiões abertas e uniformes, a maior parte da exploração do A\* é redundante e poderia ser "pulada".*

O ponto crucial — e que orienta todo o capítulo — é que esse desperdício **só existe por causa da regularidade da grade**. Em um grafo irregular (waypoints, NavMesh), não há essa profusão de caminhos idênticos, e o A\* já é enxuto. Ou seja: a mesma regularidade que, no Capítulo 7, apontamos como **vantagem** da grade (grafo implícito, fácil atualização) é também a **origem** de sua ineficiência na busca — e, como veremos, a **chave** para otimizá-la. As técnicas deste capítulo exploram essa regularidade a seu favor.

---

## 9.2 Fundamentos do Jump Point Search

A ideia do Jump Point Search é elegante e pode ser resumida em uma frase: **em vez de expandir vizinho por vizinho, "pule" em linha reta por sequências de nós que não oferecem nenhuma decisão relevante, parando apenas nos poucos pontos onde uma decisão real precisa ser tomada.** Esses poucos pontos de decisão são os **jump points** (pontos de salto), e são os únicos nós que o JPS efetivamente coloca na lista aberta. Todo o resto — a longa fila de células intermediárias que o A\* teria expandido uma a uma — é **atravessada de um pulo só**, sem entrar nas listas.

Para entender o que faz de um nó um "ponto de decisão", precisamos dos dois conceitos que sustentam o método: a **poda de simetria** e os **vizinhos forçados**.

### 9.2.1 Simetria de caminhos e "pulos"

O JPS impõe uma **regra de canonização**: entre todos os caminhos simétricos (de mesmo custo) que levam a um nó, ele escolhe considerar apenas **um** representante canônico — por convenção, aquele que faz os movimentos diagonais **o mais cedo possível** e os ortogonais depois. Com essa regra, a maioria dos vizinhos de um nó torna-se **desnecessária de examinar**: se para chegar a eles existe um caminho canônico que **não passa** pelo nó atual, então explorá-los a partir daqui seria redundante — eles serão (ou já foram) alcançados por sua rota canônica. Esses vizinhos são **podados** (descartados).

O que resta, depois da poda, são os **vizinhos naturais** (a continuação óbvia do movimento na mesma direção) e, ocasionalmente, os **vizinhos forçados** — e é aqui que está o coração do método.

Um **vizinho forçado** surge quando um **obstáculo** quebra a simetria. Considere um agente movendo-se para a direita ao longo de uma parede. Enquanto a parede continua, seguir em frente é a única continuação sensata — não há decisão a tomar. Mas se, de repente, a parede **termina** (abre-se uma passagem), surge uma nova opção: **desviar** para a célula que o obstáculo antes bloqueava. Essa célula recém-liberada é um **vizinho forçado**: o obstáculo "força" a existência de uma decisão que não existiria em terreno aberto. **Um nó que possui vizinhos forçados é um jump point** — um ponto onde o caminho pode legitimamente ramificar, e que, por isso, merece entrar na lista aberta.

O procedimento de **"pulo"** (*jumping*) formaliza isso. A partir de um nó, numa dada direção, o JPS avança célula a célula **sem parar**, enquanto: (a) não encontra o destino, (b) não encontra um nó com vizinhos forçados, e (c) — no caso de movimento diagonal — não encontra um ponto de onde partem "pulos" ortogonais produtivos. Ao encontrar qualquer uma dessas condições, ele para: aquele é o próximo jump point, e só ele é inserido na lista aberta, com o `g` correto (o custo acumulado ao longo de todo o trecho pulado). Todas as células intermediárias foram **atravessadas sem serem expandidas** — nunca entraram nas listas, nunca custaram gerenciamento de fila de prioridade.

> ⚠️ **Atenção**
> O JPS **não é uma heurística nem um algoritmo diferente do A\*** — é o **mesmo A\***, com a **mesma função `f = g + h`** e as **mesmas garantias de otimalidade**, apenas com uma **geração de sucessores** radicalmente mais esperta. Onde o A\* clássico pergunta "quais são os 8 vizinhos desta célula?", o JPS pergunta "quais são os próximos **pontos de decisão** alcançáveis a partir daqui, e a que custo?". A resposta a essa segunda pergunta contém muito menos nós, e é isso — e só isso — que acelera a busca. Compreender que JPS é "A\* com sucessores podados" evita o erro de tratá-lo como uma técnica exótica e desconexa.

[DIAGRAMA]
Título: Vizinhos naturais, podados e forçados no JPS
Objetivo pedagógico: Mostrar como a poda de simetria elimina a maioria dos vizinhos e como um obstáculo cria um vizinho forçado, definindo um jump point.
Descrição detalhada: Dois painéis lado a lado, ambos sobre uma pequena grade. Painel A (terreno aberto): um nó central com uma seta de chegada horizontal (vindo da esquerda). Marcar em verde o único "vizinho natural" (a célula à direita, continuação do movimento) e riscar em cinza-claro (podados) os demais vizinhos, com a legenda "podados: alcançáveis por caminho canônico que não passa aqui". Painel B (com obstáculo): mesma situação, mas com uma célula bloqueada (cinza-escuro) logo acima-à-esquerda do nó; ao passar do bloqueio, a célula acima-à-direita fica liberada e é marcada em laranja como "vizinho FORÇADO". Rotular o nó central do painel B como "JUMP POINT (tem vizinho forçado)". Entre os dois painéis, uma seta longa ilustrando um "pulo": uma sequência de células atravessadas em linha (marcadas com um traço fino, não coloridas como expandidas) da origem até o jump point, com a legenda "células intermediárias são atravessadas, não expandidas".
Elementos obrigatórios: vizinho natural (verde); vizinhos podados (cinza, riscados); vizinho forçado criado por obstáculo (laranja); rótulo de jump point; ilustração do pulo com células intermediárias não expandidas.
[/DIAGRAMA]

### 9.2.2 JPS+ com pré-processamento e a tabela de saltos

O JPS clássico calcula os pulos **em tempo de busca**: a cada consulta de caminho, ele varre as células em linha reta procurando jump points. Isso já é muito mais rápido que o A\*, mas ainda faz trabalho repetido — se o mapa não muda, as distâncias de salto de cada célula em cada direção são **sempre as mesmas** e estão sendo recalculadas a cada busca. O **JPS+** elimina esse desperdício com a estratégia clássica de otimização: **pré-processamento**.

Antes de o jogo começar (ou ao carregar o nível), o JPS+ percorre a grade **uma única vez** e calcula, para **cada célula** e **cada uma das oito direções**, a distância até o próximo jump point (ou até a parede, ou até o destino em potencial) naquela direção. Esses valores são guardados numa estrutura por célula — a **tabela de saltos** (*jump distance table*, ou "distâncias de salto"). Durante a busca, o algoritmo não varre mais nada: ele **consulta a tabela** e salta direto para o próximo ponto relevante em **tempo constante**. A busca deixa de "procurar" os pulos e passa a "ler" os pulos já calculados.

O ganho é expressivo — o JPS+ é, tipicamente, **várias vezes mais rápido** que o JPS clássico e **ordens de magnitude** mais rápido que o A\* em grades grandes e abertas. Frequentemente, o JPS+ é combinado com o **Goal Bounding** (limites de objetivo): um pré-processamento adicional que, para cada aresta, guarda a "caixa" de destinos que ela pode alcançar num caminho ótimo, permitindo **descartar** de imediato saltos que jamais levariam ao destino atual. A combinação **JPS+ com Goal Bounding** está entre os algoritmos de busca em grade mais rápidos conhecidos.

Mas o pré-processamento tem um **preço**, e ele define exatamente onde o JPS+ é ou não apropriado:

- **Custo de memória:** a tabela de saltos armazena vários valores por célula, aumentando o consumo de memória proporcionalmente ao tamanho da grade.
- **Custo de construção:** o pré-processamento leva tempo (feito no carregamento do nível, não deve pesar no jogo em execução — mas existe).
- **Rigidez perante mudanças:** e aqui está a limitação decisiva — **se o mapa muda, a tabela fica inválida** e precisa ser recalculada. O JPS+ pressupõe um mundo **estático**. Num cenário onde o jogador constrói e destrói estruturas o tempo todo, ou onde obstáculos surgem e somem dinamicamente, o custo de reprocessar a tabela pode anular todo o ganho — ou inviabilizar a técnica.

> ❌ **Erro Comum**
> Adotar JPS+ num jogo de mundo **dinâmico** (construção/destruição frequente, terreno mutável) e depois descobrir que o recálculo da tabela de saltos, disparado a cada mudança do mapa, custa mais do que se economizou na busca. O JPS+ brilha em mapas **estáticos e uniformes**; num mundo que muda a todo instante, o **JPS clássico** (sem pré-processamento) ou um **A\* bem otimizado** costumam ser escolhas mais seguras. Casar a técnica com a **dinâmica do mundo** é tão importante quanto casar a heurística com a conectividade (Capítulo 8).

> 🏭 **Na Indústria**
> A escolha entre JPS, JPS+ e A\* é um exemplo perfeito da engenharia de *trade-offs* que caracteriza o desenvolvimento de jogos. Não há um "melhor algoritmo" universal: há o melhor algoritmo **para este mapa, esta dinâmica e este orçamento**. Equipes que trabalham com grades grandes e estáticas (muitos jogos de estratégia por turnos, tower defense, roguelikes de mapa fixo) colhem enormes ganhos do JPS+. Equipes com mundos 3D irregulares usam NavMesh + A\* e sequer têm uma grade onde aplicar JPS. Reconhecer a qual desses mundos o seu jogo pertence é a primeira — e mais importante — decisão de otimização.

---

## 9.3 Funcionamento e comparação de desempenho com A\*

Vale consolidar **por que** o JPS/JPS+ ganha do A\*, para que o ganho não pareça mágica. O A\* e o JPS produzem **o mesmo caminho ótimo** e usam a **mesma heurística**; a diferença está inteiramente no **número de nós que entram na lista aberta** e na **quantidade de operações de fila de prioridade** — que é o gargalo real do A\*.

O A\* insere na lista aberta praticamente **toda célula na fronteira de exploração**. Cada inserção e cada remoção do menor `f` custam operações de heap, e é a **soma** dessas operações que domina o tempo. O JPS, ao pular sequências inteiras de células e inserir na lista aberta **apenas os jump points**, reduz drasticamente o número dessas operações caras: em uma região aberta onde o A\* inseriria centenas de nós, o JPS pode inserir **um punhado**. O JPS+ vai além ao eliminar até o custo de *procurar* os saltos, trocando varredura por consulta de tabela.

O resultado prático, documentado por Harabor & Grastien e por Rabin & Sturtevant:

- Em grades **grandes e abertas**, JPS costuma ser **uma ordem de magnitude** (aproximadamente 10×) mais rápido que o A\*, e JPS+ ainda mais.
- O ganho **diminui** à medida que o mapa fica mais **labiríntico** (cheio de obstáculos e corredores estreitos): com muitos obstáculos, surgem muitos jump points, os pulos ficam curtos, e o JPS se aproxima do A\* — a simetria que ele explora simplesmente existe menos.
- Em grades **pequenas**, a vantagem é marginal e pode não compensar a complexidade adicional de implementação.

[IMAGEM NECESSÁRIA]
Título: Gráfico comparativo de desempenho — A\* × JPS × JPS+
Objetivo didático: Dar ao estudante uma noção quantitativa concreta da diferença de desempenho entre os três algoritmos, reforçando com números a afirmação de "ordens de magnitude" e mostrando como o ganho varia com a densidade de obstáculos do mapa.
Descrição: Gráfico de barras (ou de linhas) comparando A\*, JPS e JPS+ em dois eixos possíveis: (a) tempo médio de busca por consulta, e (b) número de nós expandidos, medidos em mapas de mesma dimensão com densidades crescentes de obstáculos (aberto, moderado, labiríntico). O gráfico deve evidenciar que JPS/JPS+ vencem por larga margem em mapas abertos e que a vantagem encolhe em mapas labirínticos.
Tipo: Gráfico de dados (benchmark), idealmente gerado a partir de medições reais ou reproduzido de fonte confiável.
Como produzir: Executar os três algoritmos sobre um conjunto de mapas de teste padronizados (por exemplo, os mapas de benchmark de pathfinding em grade usados na literatura), registrar tempo e nós expandidos, e plotar os resultados. Alternativamente, reproduzir com a devida citação um gráfico da fonte original (Harabor & Grastien, 2011; Rabin & Sturtevant, Game AI Pro 2).
Legenda sugerida: "Em grades grandes e abertas, JPS e JPS+ superam o A\* em ordens de magnitude; à medida que o mapa fica mais labiríntico, a vantagem diminui, pois surgem mais jump points e os pulos encurtam."
[/IMAGEM NECESSÁRIA]

> ✅ **Boa Prática**
> Antes de investir em JPS+, **meça**. Otimização sem medição é adivinhação. Perfile o pathfinding do seu jogo: quantas buscas por segundo, em que tamanho de grade, com que densidade de obstáculos, com o mapa mudando com que frequência? Se o perfil revela **grades grandes, abertas e estáticas** com o pathfinding pesando no quadro, o JPS+ pode ser transformador. Se revela **mapas pequenos**, **muito obstruídos** ou **muito dinâmicos**, o esforço de implementação pode não se pagar — e um A\* com melhor gerenciamento de fila, *time-slicing* ou cache de caminhos pode render mais. A regra é a mesma de toda otimização: primeiro medir, depois otimizar o que realmente pesa.

[DIAGRAMA]
Título: Nós expandidos — A\* versus JPS numa grade aberta
Objetivo pedagógico: Contrastar visualmente a quantidade de nós que cada algoritmo coloca na lista aberta para o mesmo caminho, evidenciando a fonte do ganho de desempenho.
Descrição detalhada: Duas cópias lado a lado de uma mesma grade grande (por exemplo 16×16) com poucos obstáculos, mesma origem S e mesmo destino T. Painel esquerdo (A\*): colorir um leque largo e denso de células como "expandidas / na lista aberta", cobrindo boa parte da área entre S e T. Painel direito (JPS): colorir apenas alguns poucos jump points (pontos destacados) e desenhar linhas retas (os pulos) conectando-os de S a T, com a vasta maioria das células deixadas em branco. Sobrepor, em ambos, o MESMO caminho final (idêntico) em cor de destaque, deixando claro que a resposta é a mesma. Abaixo, uma barra comparativa simples: "A\*: N nós na lista aberta" contra "JPS: n ≪ N nós na lista aberta", com N muito maior que n.
Elementos obrigatórios: mesma grade, origem e destino nos dois painéis; leque denso (A\*) contra poucos jump points e pulos retos (JPS); caminho final idêntico destacado; barra comparativa do número de nós expandidos.
[/DIAGRAMA]

---

## 9.4 Outras otimizações: hierarquia de caminhos, pathfinding em blocos, suavização de caminho

O JPS+ ataca a simetria em grades. Mas a caixa de ferramentas da otimização de pathfinding é maior, e cada técnica ataca um aspecto diferente do custo. As três a seguir são as mais usadas na indústria e complementam — não substituem — o que já vimos.

**Pathfinding hierárquico.** A ideia é dividir o mapa em **regiões** (setores, salas, blocos de células) e construir um **grafo de alto nível** cujos nós são essas regiões e cujas arestas são as conexões entre regiões vizinhas (por exemplo, as portas). A busca passa a ter **dois níveis**: primeiro, um A\* barato no grafo abstrato de regiões encontra a **sequência de regiões** a atravessar (sala A → corredor → sala D); depois, buscas locais e curtas resolvem o caminho **dentro** de cada região. A analogia humana é exata: para ir de uma cidade a outra, você primeiro planeja "que estradas pegar" (nível alto) e só depois se preocupa com "que ruas fazer dentro de cada cidade" (nível baixo) — você não planeja a viagem inteira rua por rua. O ganho é enorme em mapas grandes, porque o problema global vira uma cadeia de problemas pequenos. O custo é a **construção e manutenção da hierarquia** e o fato de o caminho resultante poder ser **ligeiramente subótimo** (a divisão em regiões impõe passar pelas conexões pré-definidas). A abordagem clássica dessa família é o **HPA\*** (*Hierarchical Pathfinding A\**).

**Pathfinding em blocos e agrupamento de agentes.** Quando **muitos agentes** compartilham origem ou destino próximos — o exército de um RTS marchando para o mesmo ponto —, é desperdício cada um rodar sua própria busca completa. Técnicas de **flow field** (campo de fluxo) invertem o problema: em vez de calcular um caminho por agente, calcula-se **uma única vez** um "campo" que, para toda célula do mapa, indica a direção a seguir rumo ao destino comum (essencialmente, um Dijkstra a partir do destino). Todos os agentes então apenas "leem" a direção da célula em que estão e fluem para o alvo. É a técnica por trás do movimento de grandes multidões em vários jogos de estratégia modernos: o custo é pago **uma vez por destino**, não uma vez por unidade — uma economia gigantesca quando as unidades se contam às centenas.

**Suavização de caminho (path smoothing).** Os caminhos que o A\*/JPS devolvem sobre uma grade têm aparência **"quadriculada"** e "colam" nas quinas, com viradas de 45° ou 90° que nenhum personagem real faria. A **suavização** é um pós-processamento que aproxima o caminho de algo natural. A técnica mais comum é o **teste de linha de visão** (*string pulling* / *funnel algorithm* em NavMeshes): percorre-se o caminho tentando "esticar um barbante" — se há linha de visão livre entre um ponto e um ponto mais adiante, os pontos intermediários entre eles são **descartados**, encurtando e retificando a rota. O resultado é um caminho com menos vértices e traçado mais direto, que o steering (a locomoção local, cujo conceito foi apresentado no Capítulo 7) percorre com curvas suaves. A suavização não altera *qual* rota global se toma — apenas **melhora sua aparência e naturalidade**, fechando a distância entre o caminho matemático e o movimento crível.

> 🎮 **Na Prática**
> A suavização é o passo que costuma faltar em implementações amadoras e que mais impacta a **percepção de qualidade** da IA. Um caminho ótimo mas quadriculado faz o personagem parecer um robô seguindo trilhos; o mesmo caminho suavizado faz parecer que ele "decidiu" andar até ali. Como estamos no negócio da **ilusão de inteligência** (Parte I), esse ajuste estético não é secundário — é parte central de fazer o movimento convencer. Vale lembrar a divisão de trabalho: o **pathfinding** acha a rota, a **suavização** a refina, e o **steering** a percorre com um corpo que acelera, freia e curva de modo natural.

[DIAGRAMA]
Título: Suavização de caminho por linha de visão
Objetivo pedagógico: Mostrar como o pós-processamento de suavização transforma um caminho quadriculado em uma rota direta e natural, descartando pontos intermediários desnecessários.
Descrição detalhada: Uma grade com origem S e destino T e um obstáculo em L entre eles. Em cinza, desenhar o caminho BRUTO devolvido pelo A\*/JPS: uma sequência "escadinha" de segmentos ortogonais/diagonais ligando os centros das células, com muitos vértices. Sobrepor, em cor de destaque, o caminho SUAVIZADO: poucos segmentos longos e retos que contornam a quina do obstáculo (por exemplo, S direto até o vértice externo do L, e deste direto até T). Desenhar linhas tracejadas de "teste de visão" entre S e vários pontos adiante, com um "✓" onde a visão é livre (ponto intermediário descartado) e um "✗" onde a visão é bloqueada pelo obstáculo (ponto mantido). Legenda: "se há linha de visão livre entre A e C, o ponto B intermediário é descartado".
Elementos obrigatórios: caminho bruto quadriculado (cinza); caminho suavizado direto (destaque); testes de linha de visão com ✓/✗; obstáculo forçando a manutenção de um vértice de contorno; legenda da regra.
[/DIAGRAMA]

---

## 9.5 Vantagens e limitações

Consolidando os *trade-offs* deste capítulo:

**Vantagens (JPS/JPS+ e otimizações em geral).** Aceleração de **ordens de magnitude** sobre o A\* em grades grandes e abertas (JPS/JPS+), **preservando a otimalidade** — não se troca qualidade por velocidade, como no A\* ponderado. O pré-processamento do JPS+ move o custo para **fora do tempo de jogo** (carregamento), aliviando o quadro. O pathfinding **hierárquico** e os **flow fields** tornam viáveis mapas enormes e multidões de agentes que seriam impensáveis com A\* por unidade. A **suavização** eleva a qualidade percebida do movimento a baixo custo.

**Limitações.** O JPS e o JPS+ **exigem grade uniforme** — não se aplicam a NavMeshes nem a grafos irregulares de waypoints. O JPS+ pressupõe mapa **estático**; mundos dinâmicos invalidam a tabela de saltos. O ganho **desaparece em mapas labirínticos** (muitos obstáculos, poucos pulos longos). O pré-processamento custa **memória e tempo de construção**. O pathfinding hierárquico e os flow fields podem devolver caminhos **levemente subótimos** e adicionam **complexidade de implementação e manutenção** (construir e atualizar a hierarquia/o campo). E toda otimização acrescenta **código para manter e depurar** — um custo de engenharia que só se justifica quando o problema real o exige.

> ⚠️ **Atenção**
> A tentação de "otimizar por otimizar" é um erro clássico de engenharia, e o pathfinding é um campo fértil para ele. Implementar JPS+ num jogo cujo mapa é pequeno, ou cujo gargalo real está na física e não na busca, adiciona complexidade sem retorno — e complexidade é dívida técnica. A pergunta correta nunca é "qual é o algoritmo mais rápido?", mas "**o pathfinding é, comprovadamente por medição, um gargalo neste jogo — e, se é, qual otimização ataca a causa medida?**". Otimização é uma resposta a um problema medido, não um troféu a colecionar.

---

## 9.6 Aplicações e jogos conhecidos

As otimizações deste capítulo encontram seu lar natural nos **jogos de estratégia**, onde a combinação de mapas grandes, grades regulares e multidões de unidades leva o A\* clássico ao limite. Títulos com grandes exércitos e movimento de massa — a linhagem dos RTS e dos jogos de estratégia em geral — são os que mais dependem de **flow fields** para mover centenas de unidades a um destino comum sem colapsar o quadro, e de **pathfinding hierárquico** para planejar rotas longas por mapas extensos. O JPS+ encontra aplicação em jogos de grade **estática** — estratégia por turnos, *tower defense*, roguelikes de mapa fixo, muitos jogos móveis e web — onde seus pressupostos (grade uniforme, mundo estável) são plenamente satisfeitos e o ganho é máximo.

A **suavização de caminho**, por sua vez, é praticamente **universal**: qualquer jogo 3D com personagens que se movem — dos grandes títulos de ação e mundo aberto aos jogos independentes — aplica alguma forma de *string pulling* ou *funnel* para converter o caminho bruto da NavMesh em movimento crível. É a otimização que o jogador **nunca vê**, mas cuja ausência ele **sempre percebe** na forma de personagens que andam como robôs.

> 🏭 **Na Indústria**
> A GDC (Game Developers Conference) e a série de livros *Game AI Pro*, organizada por Steve Rabin, são as fontes onde a indústria documenta abertamente essas técnicas — incluindo a apresentação original do JPS+ com Goal Bounding e relatos de uso de flow fields e pathfinding hierárquico em jogos comerciais. Para o estudante que quiser ver esses algoritmos "na prática dos estúdios", e não apenas na teoria, essas coletâneas são a ponte direta entre a academia e o mercado — exatamente o tipo de fonte primária que retomaremos nos estudos de caso da Parte VII.

---

## 9.7 Ferramentas na Unity e de terceiros

Na Unity, a **suavização de caminho** já está embutida no fluxo do **NavMesh Agent**: a busca sobre o grafo de polígonos é seguida de um algoritmo de *funnel* (*string pulling*) que retifica o trajeto dentro dos polígonos, entregando um caminho já suavizado — mais uma razão pela qual, para movimento 3D padrão, reimplementar o pathfinding raramente compensa. O sistema oficial, porém, **não oferece JPS/JPS+ nativamente**, porque sua representação primária é a NavMesh (irregular), e não a grade uniforme que o JPS exige.

Para **grades**, onde as técnicas deste capítulo mais rendem, recorre-se a terceiros. O **A\* Pathfinding Project** (Aron Granberg) suporta grades e oferece variantes otimizadas, incluindo **multithreading**, cache de caminhos e abordagens hierárquicas, sendo a escolha frequente de jogos de estratégia no ecossistema Unity. Implementações abertas de **JPS/JPS+** em C# estão disponíveis para integração direta em projetos baseados em grade, e a técnica de **flow field** é comumente implementada sob medida por estúdios de RTS, dada sua dependência das particularidades de cada jogo. No campo do código aberto para 3D, o **Detour** (par do Recast) já incorpora o *funnel* para suavização, novamente evidenciando que essas otimizações são parte madura e padronizada do ferramental de navegação.

> ✅ **Boa Prática**
> Ao avaliar uma ferramenta de pathfinding, olhe além do "tem A\*?". Pergunte: **qual representação ela suporta** (grade, waypoints, NavMesh)? **Ela suaviza** os caminhos? **Oferece atualização dinâmica** eficiente e, se sim, a que custo? **Suporta multithreading ou time-slicing** para muitos agentes? **Traz otimizações** (hierarquia, JPS, flow field) que casam com o meu tipo de mapa? Essas perguntas — e não a mera presença do A\* — é que determinam se a ferramenta serve ao seu jogo. Elas são, no fundo, a aplicação prática de tudo o que esta Parte III ensinou.

---

## Resumo

Este capítulo mostrou como **ir além do A\*** sem abandoná-lo, e — tão importante quanto — **quando** vale a pena fazê-lo. Partimos do **problema** que motiva toda a otimização de busca em grade: a **simetria de caminhos**. Em grades grandes, uniformes e abertas, existem incontáveis rotas de mesmo custo entre dois pontos, e o A\* clássico desperdiça esforço explorando essas alternativas redundantes, expandindo uma multidão de nós intercambiáveis.

Apresentamos o **Jump Point Search (JPS)** como a resposta direta a esse desperdício: em vez de expandir vizinho por vizinho, o JPS **pula** por sequências de células sem decisão e para apenas nos **jump points** — os poucos nós que possuem **vizinhos forçados**, criados quando um obstáculo quebra a simetria. Enfatizamos que o JPS **é o mesmo A\***, com a mesma `f = g + h` e a mesma otimalidade, mudando apenas a **geração de sucessores**. O **JPS+** acrescenta **pré-processamento**: calcula uma vez a **tabela de saltos** (distâncias de salto por célula e direção), trocando varredura por consulta em tempo constante e, combinado com **Goal Bounding**, alcançando desempenho de ponta — ao custo de **memória**, **tempo de construção** e, decisivamente, da exigência de um mapa **estático**.

Comparamos o **desempenho**: JPS/JPS+ são até **uma ordem de magnitude** mais rápidos que o A\* em grades abertas, com ganho que **diminui** em mapas labirínticos e é **marginal** em grades pequenas — reforçando a regra de **medir antes de otimizar**. Percorremos as demais otimizações da indústria: o **pathfinding hierárquico** (HPA\*), que planeja em dois níveis (regiões e depois interior de cada região); os **flow fields**, que calculam o caminho **uma vez por destino** para mover multidões de agentes; e a **suavização de caminho** (*string pulling* / *funnel*), que converte o caminho bruto e quadriculado em movimento natural e crível, essencial para a ilusão de inteligência.

Pesamos **vantagens** (aceleração preservando otimalidade; viabilização de mapas enormes e multidões; qualidade de movimento) e **limitações** (JPS exige grade uniforme; JPS+ exige mundo estático; ganho some em labirintos; custo de memória, construção e complexidade), e ancoramos tudo nas **aplicações** (estratégia, mundos grandes, todo jogo 3D com suavização), nos **jogos** e nas **ferramentas** (suavização nativa do NavMesh Agent; grades e otimizações via A\* Pathfinding Project; JPS/JPS+ e flow fields por terceiros ou sob medida). A lição central, que fecha a Parte III, é de engenharia: **não existe o melhor algoritmo em abstrato — existe o melhor algoritmo para este mapa, esta dinâmica e este orçamento**, e o profissional maduro é aquele que sabe medir o problema antes de escolher a solução.

## Exercícios de Fixação

1. O que é a **simetria de caminhos** em uma grade uniforme? Por que ela faz o A\* clássico desperdiçar trabalho, e por que esse desperdício não ocorre (ou é muito menor) em grafos irregulares como NavMeshes?
2. Explique, com suas palavras, o que é um **jump point** e o que é um **vizinho forçado**. Qual o papel de um **obstáculo** na criação de um vizinho forçado?
3. Por que se afirma que o **JPS é "o mesmo A\* com geração de sucessores diferente"**? O que ele **mantém** do A\* (função de avaliação, otimalidade) e o que ele **muda**?
4. O que o **JPS+** pré-calcula e armazena na **tabela de saltos**? Como isso o torna mais rápido que o JPS clássico durante a busca?
5. Cite os **três custos** do pré-processamento do JPS+. Qual deles torna o JPS+ **inadequado** para jogos de mundo dinâmico, e por quê?
6. Em que tipo de mapa o ganho do JPS sobre o A\* é **máximo**, e em que tipo ele **quase desaparece**? Explique a razão em termos de quantidade de jump points e comprimento dos pulos.
7. Explique o **pathfinding hierárquico** usando a analogia da viagem entre cidades. Qual é o ganho de desempenho e qual é o preço pago em otimalidade?
8. O que é um **flow field** e por que ele é vantajoso quando **muitos agentes** têm o mesmo destino? Compare o custo "uma busca por agente" com "uma busca por destino".
9. O que é a **suavização de caminho** e por que ela é decisiva para a **ilusão de inteligência**, mesmo sem alterar a rota global escolhida? Descreva o teste de **linha de visão** (*string pulling*).
10. Um estúdio quer "deixar o pathfinding mais rápido" e propõe implementar JPS+. Que **perguntas de medição** você faria antes de aprovar essa decisão? Descreva ao menos um cenário em que JPS+ seria a escolha certa e um em que seria um erro.

## Referências

HARABOR, Daniel; GRASTIEN, Alban. Online Graph Pruning for Pathfinding on Grid Maps. In: *Proceedings of the AAAI Conference on Artificial Intelligence*, 2011.

RABIN, Steve; STURTEVANT, Nathan R. JPS+: An Extreme A\* Speed Optimization for Static Uniform Cost Grids. In: RABIN, Steve (org.). *Game AI Pro 2*. Boca Raton: A K Peters/CRC Press, 2015.

RABIN, Steve (org.). *Game AI Pro 3: Collected Wisdom of Game AI Professionals.* Boca Raton: A K Peters/CRC Press, 2017.

BOTEA, Adi; MÜLLER, Martin; SCHAEFFER, Jonathan. Near Optimal Hierarchical Path-Finding (HPA\*). *Journal of Game Development*, v. 1, n. 1, 2004.

MILLINGTON, Ian. *AI for Games.* 3. ed. Boca Raton: CRC Press, 2019.

CORMEN, Thomas H.; LEISERSON, Charles E.; RIVEST, Ronald L.; STEIN, Clifford. *Algoritmos: Teoria e Prática.* 3. ed. Rio de Janeiro: Elsevier, 2012.

UNITY TECHNOLOGIES. *Documentação oficial: AI Navigation (NavMesh Agent, cálculo e suavização de caminho).*

GRANBERG, Aron. *A\* Pathfinding Project — Documentação.*

MONONEN, Mikko. *Recast & Detour: Navigation Mesh Toolset.* Documentação do projeto de código aberto.
