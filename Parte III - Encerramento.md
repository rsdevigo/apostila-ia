# Encerramento da Parte III

## Resumo Geral da Parte III

A Parte III mudou o foco da pergunta que organizou a Parte II — "*qual* comportamento adotar?" — para uma nova pergunta, igualmente fundamental: "*como chegar* até onde quero agir?". Deixamos o terreno das regras de decisão e entramos no da **busca em grafos**, percorrendo, em progressão rígida e cumulativa, as duas metades do problema de movimento: a **representação** do espaço e a **busca** de caminhos sobre ela.

O **Capítulo 7** forneceu a **fundação espacial**. Estabeleceu que a busca de caminhos não opera sobre a geometria bruta do jogo, mas sobre uma abstração discreta — um **grafo** de nós, arestas e custos —, e distinguiu com cuidado a **busca de caminho** (rota global) da **direção/locomoção** (movimento local). Construiu a **teoria dos grafos** aplicada a jogos (vértices, arestas, pesos, grafos direcionados, conectividade, custo de caminho, representação esparsa) e apresentou as três grandes formas de representar o espaço — **grades**, **waypoints** e **malhas de navegação (NavMesh)** —, cada uma com seu perfil de precisão, memória, custo de construção e manutenção. O capítulo deixou o leitor apto a enxergar qualquer cenário como um grafo pronto para ser percorrido.

O **Capítulo 8** construiu, sobre esse grafo, o **algoritmo A\***, o coração da busca de caminhos em jogos. Partiu de **Dijkstra** (busca não-informada, ótima, mas cega), introduziu a **heurística** e a **busca informada**, e chegou à função de avaliação `f = g + h` — a fusão do custo já pago (`g`, exato) com o custo estimado restante (`h`, palpite). Formalizou as propriedades que garantem a otimalidade — **admissibilidade** e **consistência** —, detalhou o funcionamento (**listas aberta e fechada**, **relaxamento**, **reconstrução do caminho**), estudou as **heurísticas de grade** (Manhattan, octile/Chebyshev, Euclidiana) e a importância de casá-las com a conectividade, e apresentou o recurso pragmático do **A\* ponderado**. Ficou claro *por que* o A\* se tornou o padrão universal: ele equilibra a otimalidade de Dijkstra com a direção da busca gulosa.

O **Capítulo 9** mostrou a **fronteira de desempenho**. Diagnosticou o desperdício do A\* em grades grandes e uniformes — a **simetria de caminhos** — e apresentou o **Jump Point Search (JPS)** e sua variante pré-processada **JPS+** como a resposta: pular sequências de células sem decisão, parando apenas nos **jump points** definidos por **vizinhos forçados**, mantendo a otimalidade do A\* e ganhando ordens de magnitude em velocidade. Complementou com o arsenal da indústria — **pathfinding hierárquico**, **flow fields** e **suavização de caminho** — e, sobretudo, ensinou o **critério** que fecha a Parte: não existe o melhor algoritmo em abstrato, existe o melhor algoritmo **para este mapa, esta dinâmica e este orçamento**, e otimização é resposta a um problema **medido**, não um troféu.

A síntese da Parte é uma tese em duas partes. Primeiro: **todo movimento inteligente em jogos repousa sobre um grafo**, e a qualidade do resultado depende tanto da representação escolhida quanto do algoritmo de busca — "nenhum algoritmo é melhor do que a representação sobre a qual roda". Segundo: **a busca de caminhos é um exercício permanente de *trade-offs*** — otimalidade contra velocidade, generalidade contra especialização, simplicidade contra desempenho —, e dominar o pathfinding é dominar a arte de escolher, para cada problema concreto, o ponto certo nesse espaço de compromissos.

## Principais Conceitos Aprendidos

- **Busca de caminho × direção (pathfinding × steering)** — a rota global (contornar obstáculos grandes e fixos) versus o movimento local suave (desviar de obstáculos pequenos e dinâmicos); problemas distintos e complementares, aplicados em camadas.
- **Grafo de navegação** — abstração do espaço navegável como vértices (lugares), arestas (conexões diretas) e pesos (custos); a estrutura sobre a qual toda busca ocorre.
- **Vértice, aresta, peso e direção** — os quatro elementos do grafo; os pesos permitem "caminhos convenientes" (não só curtos), e as arestas direcionadas capturam movimentos irreversíveis.
- **Custo de caminho e conectividade** — o custo total é a soma dos pesos das arestas; a conectividade determina o que é sequer alcançável (componentes desconexos).
- **Grade (grid)** — cobertura regular do espaço; grafo implícito, dinâmica fácil, mas custosa em alta resolução; conectividade-4 (Manhattan) versus conectividade-8 (custo diagonal ≈ 1,41).
- **Waypoints** — pontos de rota conectados por linha de visão; grafo esparso, busca rápida, controle autoral, porém manutenção manual e movimento "amarrado a trilhos".
- **Malha de navegação (NavMesh)** — polígonos convexos cobrindo áreas caminháveis; representa áreas (não linhas), adapta a resolução, é gerada automaticamente; padrão da indústria 3D.
- **Dijkstra (busca não-informada)** — expansão por custo acumulado crescente; ótima, mas cega — explora em todas as direções por ignorar o destino.
- **Heurística e busca informada** — estimativa `h(n)` do custo restante que direciona a busca; transforma a mancha de Dijkstra num feixe rumo ao alvo.
- **A\* e a função f = g + h** — combina o custo exato já pago (`g`) com a estimativa restante (`h`); expande sempre o menor `f`; o equilíbrio entre Dijkstra e a busca gulosa.
- **Admissibilidade e consistência** — a heurística nunca superestima (otimista → garante otimalidade); a desigualdade triangular (consistente → dispensa reprocessar nós fechados).
- **Listas aberta e fechada, relaxamento e reconstrução** — a fronteira (fila de prioridade por `f`), os nós resolvidos, a atualização de `g`/predecessor, e a montagem do caminho por predecessores.
- **Heurísticas de grade (Manhattan, octile/Chebyshev, Euclidiana)** — cada uma exata para um conjunto de movimentos; casá-la à conectividade preserva otimalidade e eficiência.
- **A\* ponderado (weighted A\*)** — multiplicar a heurística por um peso > 1; troca deliberadamente otimalidade por velocidade — um "dial" de desempenho.
- **Simetria de caminhos** — profusão de rotas de mesmo custo em grades uniformes; a fonte do desperdício do A\* que o JPS ataca.
- **Jump Point Search (JPS) e jump points** — pular células sem decisão e parar só nos pontos com **vizinhos forçados**; o mesmo A\*, com geração de sucessores podada.
- **JPS+ e tabela de saltos** — pré-processamento das distâncias de salto por célula/direção; velocíssimo em mapas **estáticos e uniformes**, inadequado a mundos dinâmicos.
- **Pathfinding hierárquico (HPA\*)** — planejar por regiões (alto nível) e depois no interior de cada uma (baixo nível); viabiliza mapas enormes ao custo de leve subotimalidade.
- **Flow fields** — calcular o caminho **uma vez por destino** (não por agente) para mover multidões; economia decisiva em RTS.
- **Suavização de caminho (string pulling / funnel)** — pós-processamento por linha de visão que converte o caminho quadriculado em movimento natural; central para a ilusão de inteligência.
- **Trade-off como método** — não há melhor algoritmo em abstrato; medir o problema (tamanho, dinâmica, densidade, orçamento) antecede escolher a solução.

## Tabela Comparativa 1 — Representações do Espaço

A tabela consolida as três representações do Capítulo 7 mais a comparação entre **grafo genérico**, **grade**, **waypoints** e **NavMesh** pedida ao final da Parte. É a principal ferramenta de revisão sobre *representação*.

| Critério | **Grafo (abstração)** | **Grade (grid)** | **Waypoints** | **NavMesh** |
|---|---|---|---|---|
| **O que é um nó** | Um lugar qualquer (conceito) | Uma célula (quadrada/hexagonal) | Um ponto de rota | Um polígono convexo (área) |
| **O que é uma aresta** | Conexão direta genérica | Adjacência entre células vizinhas | Linha de visão livre entre pontos | Borda compartilhada entre polígonos |
| **Como o grafo é obtido** | — (modelo teórico) | Implícito (vizinhos por aritmética) | Manual (designer posiciona) | Gerado automaticamente (baking/Recast) |
| **Cobertura do espaço** | — | Total, uniforme | Esparsa (só rotas previstas) | Total, adaptativa (áreas) |
| **Custo de memória** | — | Alto em alta resolução | Muito baixo | Médio (adapta à geometria) |
| **Atualização dinâmica** | — | Trivial (mudar uma célula) | Trabalhosa (revisar rede) | Cara (recozimento local/carving) |
| **Qualidade do movimento** | — | "Quadriculada" (exige suavização) | Presa a trilhos | Fluida e tática |
| **Custo de autoria** | — | Baixo | **Alto** (manual) | Baixo (automático) |
| **Aplicações típicas** | Base conceitual de todas | 2D, estratégia por turnos, tower defense, roguelikes | Jogos 3D clássicos (anos 2000); hoje, pontos táticos | Padrão dos jogos 3D modernos (Unity/Unreal) |

**Análise interpretativa.** Lida da esquerda para a direita, a tabela conta a **história evolutiva** da representação espacial em jogos. O **grafo** é o conceito unificador — tudo o mais é uma forma concreta de instanciá-lo. A **grade** foi a primeira resposta prática: barata de raciocinar e de atualizar, ideal quando o próprio mundo já é quadriculado, mas presa à sua regularidade e cara em alta resolução. Os **waypoints** trocaram cobertura por velocidade — um grafo minúsculo e rapidíssimo —, mas pagaram com autoria manual e movimento engessado, o que os tornou obsoletos como representação primária. A **NavMesh** venceu na era 3D porque resolveu simultaneamente os dois maiores problemas das alternativas: eliminou a explosão de memória da grade fina (adaptando a resolução) e o custo de autoria dos waypoints (gerando a malha automaticamente), entregando ainda movimento fluido por representar **áreas**. Repare, contudo, que a NavMesh **não tornou as grades obsoletas**: em jogos 2D, de estratégia por turnos ou de mundo dinâmico e quadriculado, a grade continua superior — e é a única sobre a qual o JPS+ do Capítulo 9 se aplica. A lição, coerente com toda a apostila, é que **a escolha da representação depende do problema**, não de uma hierarquia fixa de "melhor para pior".

## Tabela Comparativa 2 — Algoritmos de Busca

A segunda tabela consolida os algoritmos de busca da Parte, comparando **Dijkstra**, **Busca Gulosa (Greedy Best-First)**, **A\*** e **JPS+** segundo os critérios pedidos. É a principal ferramenta de revisão sobre *busca*.

| Critério | **Dijkstra** | **Greedy Best-First** | **A\*** | **JPS+** |
|---|---|---|---|---|
| **Função que guia a busca** | `f = g` (só custo pago) | `f = h` (só estimativa) | `f = g + h` | `f = g + h` (sucessores podados) |
| **Usa heurística?** | Não (cega) | Sim (só ela) | Sim (com `g`) | Sim (com `g`) |
| **Ótimo?** | Sim | **Não** | Sim (se `h` admissível) | Sim (grade uniforme) |
| **Custo computacional** | Alto (explora tudo) | Baixo (corre ao alvo) | Médio | **Muito baixo** (mapas abertos) |
| **Qualidade do caminho** | Ótima | Pode ser ruim/torta | Ótima | Ótima |
| **Memória** | Alta | Baixa–média | Média (por nó tocado) | Média + **tabela de saltos** (pré-processada) |
| **Requisitos** | Grafo com pesos ≥ 0 | Heurística | Heurística admissível | **Grade uniforme e estática** |
| **Aplicações típicas** | Muitos destinos; mapas de influência (Parte IV) | Raro puro; quando velocidade ≫ qualidade | Padrão universal de pathfinding | Grades grandes, abertas e estáticas (estratégia) |
| **Limitações** | Lento (não sabe onde é o alvo) | Não garante o melhor caminho | Nós simétricos; custo com muitos agentes | Só grade; invalida-se se o mapa muda |

**Análise interpretativa.** As quatro colunas formam um **espectro no eixo otimalidade × velocidade**, e compreendê-lo é compreender toda a busca de caminhos. Nos extremos estão os dois algoritmos "puros": **Dijkstra**, que só olha para trás (o custo já pago) e por isso é ótimo mas lento e cego; e a **Busca Gulosa**, que só olha para frente (a estimativa) e por isso é rápida mas não confiável — pode se lançar por um caminho que parece curto e revelar-se péssimo. O **A\*** é a síntese: ao usar `g + h`, herda a otimalidade de Dijkstra **e** a direção da busca gulosa, ocupando o ponto de equilíbrio que o consagrou como padrão universal. O **JPS+** não é um concorrente do A\*, mas uma **especialização** dele: o mesmo `g + h`, a mesma otimalidade, com uma geração de sucessores que explora a regularidade da grade para eliminar o trabalho redundante — pagando por isso com restrições fortes (só grade, só mundo estático) e memória extra. A leitura profissional da tabela é que a escolha não é "qual é o melhor?", mas "**qual casa com o meu problema?**": Dijkstra quando há muitos destinos (ou nenhuma boa heurística); A\* como padrão seguro e geral; JPS+ quando — e somente quando — o mapa é uma grade grande, aberta e estável; e a Busca Gulosa quase nunca sozinha, mas viva no espírito do **A\* ponderado**, que a "mistura" ao A\* para comprar velocidade quando o orçamento aperta. É o mesmo pragmatismo da Parte I: escolher, para cada contexto, o ponto certo entre o *perfeito* e o *barato*.

## Questões de Revisão

As questões cobrem os conceitos centrais da Parte III. Recomenda-se respondê-las por escrito, com as próprias palavras, antes de avançar para a Parte IV.

1. Explique a afirmação "**o problema de busca de caminhos tem duas metades: a representação e a busca**". Por que "nenhum algoritmo de busca é melhor do que a representação sobre a qual roda"?
2. Distinga **pathfinding** de **steering**. Em que ordem atuam e por que um sistema robusto precisa dos dois?
3. Compare **grade**, **waypoints** e **NavMesh** quanto ao que cada nó representa, como o grafo é obtido e como cada uma lida com atualização dinâmica. Para que tipo de jogo cada uma é a escolha natural?
4. Deduza por que o A\* com `h(n) = 0` se comporta exatamente como **Dijkstra** e por que, ignorando `g`, ele vira a **Busca Gulosa**. O que o A\* colhe de cada extremo?
5. Defina **admissibilidade** e **consistência** de uma heurística. Qual garante a otimalidade e qual dispensa reprocessar nós fechados?
6. Por que a heurística deve **casar** com a conectividade da grade? Dê o exemplo do erro de usar Manhattan em conectividade-8 e sua consequência sobre a otimalidade.
7. Explique a **simetria de caminhos** e como o **JPS** a explora. O que é um **vizinho forçado** e por que ele define um **jump point**?
8. Contraste **JPS** e **JPS+**. O que o pré-processamento economiza e qual pressuposto sobre o mundo ele exige? Por que isso o inviabiliza em jogos dinâmicos?
9. Descreva **pathfinding hierárquico** e **flow fields**. Que problema de escala cada um resolve, e a que custo (otimalidade, complexidade)?
10. Explique por que a **suavização de caminho** é essencial para a **ilusão de inteligência**, mesmo sem mudar a rota global. Descreva o teste de linha de visão (*string pulling*).

## Exercícios Conceituais

1. **Do mundo ao grafo.** Escolha um cenário concreto (por exemplo, uma casa com cômodos e móveis) e descreva como você o representaria como (a) grade, (b) waypoints e (c) NavMesh. Para cada uma, diga o que seria um nó, o que seria uma aresta e onde estariam os custos. Aponte qual escolheria e por quê.
2. **Modelagem por custos.** Um mapa tem estrada (rápida), grama (média), pântano (lenta) e lava (mortal). Atribua pesos de aresta a cada terreno de modo que os NPCs prefiram a estrada, tolerem a grama, evitem o pântano e nunca entrem na lava — **sem** escrever nenhuma regra explícita de comportamento. Explique como o comportamento "inteligente" emerge apenas dos custos.
3. **Traço de A\*.** Em uma grade pequena (por exemplo 6×6) desenhada por você, com uma parede e uma passagem, escolha origem e destino e trace, passo a passo, algumas iterações do A\* (conectividade-4, Manhattan): mostre os valores `g`, `h`, `f` de alguns nós e a ordem de expansão. Ao final, reconstrua o caminho pelos predecessores.
4. **Escolha da heurística.** Para cada situação, indique a heurística correta e justifique: (a) grade de 4 direções; (b) grade de 8 direções com custo diagonal `√2`; (c) NavMesh com movimento livre em qualquer ângulo. Depois, explique o que aconteceria em (b) se usasse Manhattan.
5. **O dial do peso.** Explique, com um exemplo de RTS sob pressão de quadro, como e por que você usaria o **A\* ponderado**. O que você ganha e o que aceita perder? Como isso se relaciona com a filosofia da Parte I?
6. **Diagnóstico de otimização.** Um jogo tem um mapa 2000×2000 em grade, quase todo aberto, **estático**, com 300 unidades marchando ao mesmo destino e travando o quadro. Proponha uma combinação de técnicas do Capítulo 9 para resolver, justificando cada escolha (JPS+? flow field? hierarquia? suavização?). Depois, mude **um** pressuposto (o mapa passa a ser dinâmico) e reavalie sua proposta.

## Exercícios de Integração da Parte III

Os exercícios a seguir exigem **relacionar** os três capítulos entre si — e conectá-los às Partes anteriores —, não apenas recordar cada um isoladamente.

**Exercício 1 — Da decisão ao destino ao caminho.** Retome uma arquitetura de decisão da Parte II (uma FSM ou uma árvore de comportamento de um guarda). Mostre, ponta a ponta, o fluxo completo: (a) a **decisão** que produz um destino ("investigar o último ruído"); (b) a **representação** do espaço em que esse destino existe (que representação você escolheria para o nível e por quê); (c) a **busca** que encontra o caminho (qual algoritmo); e (d) a **suavização/steering** que faz o guarda percorrê-lo de forma crível. Explique como as três Partes da apostila colaboram nesse único comportamento.

**Exercício 2 — A mesma busca, três representações.** Para um mesmo par origem–destino, descreva (em diagramas ou texto estruturado) como o A\* se comportaria sobre (a) uma grade, (b) um grafo de waypoints e (c) uma NavMesh do mesmo cenário. Compare quantos nós cada representação provavelmente faria o A\* expandir e como seria a **qualidade** do caminho antes da suavização. O que isso demonstra sobre a frase "a representação importa tanto quanto o algoritmo"?

**Exercício 3 — Escolhendo o algoritmo certo.** Para cada jogo abaixo, escolha entre **Dijkstra, Busca Gulosa, A\*, A\* ponderado e JPS+**, justificando com base em mapa, dinâmica e orçamento: (a) um *tower defense* de grade fixa com centenas de inimigos pelo mesmo caminho; (b) um jogo 3D de ação com inimigos sobre NavMesh; (c) um RTS de mundo aberto com construção/destruição de estruturas; (d) o cálculo da distância de um ponto a **todos** os outros para um futuro mapa de influência (Parte IV). Discuta os *trade-offs* de cada escolha.

**Exercício 4 — Otimização orientada por medição.** Um estúdio afirma: "nosso jogo está lento, vamos implementar JPS+". Escreva um roteiro de **diagnóstico** (que medir, que perguntas fazer sobre o mapa e a dinâmica) que **confirme ou refute** essa decisão. Descreva um cenário em que a proposta é acertada e outro em que ela pioraria as coisas — e o que você faria no lugar.

**Exercício 5 — Custos como ferramenta de design (ponte com a Parte IV).** Explique como os **pesos de aresta** aprendidos no Capítulo 7 permitem que um caminho "seguro" ou "furtivo" emerja naturalmente da busca, sem regras explícitas. Depois, especule: se, em vez de custos fixos por terreno, os custos refletissem o **perigo dinâmico** (proximidade de inimigos, linha de tiro), que tipo de comportamento tático emergiria? Essa reflexão antecipa exatamente o tema da Parte IV — os **mapas de influência**.

**Exercício 6 — A busca de caminhos como história de engenharia.** Escreva um texto curto (cerca de uma página) narrando a Parte III como uma **história de trade-offs**: comece pela necessidade de representar o espaço, passe pela escolha entre grade, waypoints e NavMesh, chegue ao A\* como equilíbrio entre Dijkstra e a busca gulosa, e termine nas otimizações do Capítulo 9 e na lição de "medir antes de otimizar". Conecte a narrativa à tese da apostila de que a IA de jogos busca o *convincente e barato*, não o *perfeito e caro*.

## Leituras Complementares

- **MILLINGTON, I.** *AI for Games.* 3. ed. — a referência mais completa e acessível para toda a Parte III: representação do mundo, grafos, A\* e suas variantes, otimizações de pathfinding e locomoção/steering, com boa articulação entre teoria e prática de jogos. Leitura recomendada dos capítulos sobre *movement* e *pathfinding*.
- **RUSSELL, S.; NORVIG, P.** *Inteligência Artificial.* 3. ed. — para o embasamento formal de busca não-informada e informada (Dijkstra, busca gulosa, A\*, admissibilidade e consistência) fora do contexto específico de jogos; o tratamento clássico do tema.
- **CORMEN, T. et al.** *Algoritmos: Teoria e Prática.* — para o rigor sobre grafos, representações em memória, filas de prioridade e o algoritmo de Dijkstra (relaxamento de arestas), a base algorítmica sob o A\*.
- **BONDY, J. A.; MURTY, U. S. R.** *Graph Theory.* — para o aluno que quiser aprofundar a teoria dos grafos além do necessário ao pathfinding (conectividade, componentes, caminhos), com todo o rigor matemático.
- **RABIN, S. (org.).** *Game AI Pro* (séries 1 a 3) — coletâneas com artigos de profissionais sobre pathfinding hierárquico, JPS+ com Goal Bounding, flow fields e suavização em jogos reais; a ponte direta entre a teoria desta Parte e o mercado.
- **HARABOR, D.; GRASTIEN, A.** *Online Graph Pruning for Pathfinding on Grid Maps* (2011) — o artigo original do Jump Point Search, para quem quiser a fonte primária do Capítulo 9.
- Documentação oficial do sistema **AI Navigation / NavMesh** da Unity e da navegação da **Unreal Engine** — para ver os conceitos materializados nas ferramentas profissionais, e a biblioteca aberta **Recast & Detour** para entender o que há "por baixo do botão de bake".

> **Boa Prática**
> Ao consultar as leituras, mantenha as **duas tabelas comparativas** deste Encerramento como mapa. As fontes tendem a apresentar cada representação e cada algoritmo isoladamente; usar as tabelas ajuda a enxergá-los como um **espaço de trade-offs** conectado, e não como uma lista de técnicas soltas. Pergunte-se sempre, diante de cada técnica: *que problema ela resolve, que pressupostos ela exige, e a que custo?*

## Referências Bibliográficas da Parte III

BONDY, John Adrian; MURTY, U. S. R. *Graph Theory.* Nova York: Springer, 2008. (Graduate Texts in Mathematics, v. 244.)

BOTEA, Adi; MÜLLER, Martin; SCHAEFFER, Jonathan. Near Optimal Hierarchical Path-Finding (HPA\*). *Journal of Game Development*, v. 1, n. 1, 2004.

CORMEN, Thomas H.; LEISERSON, Charles E.; RIVEST, Ronald L.; STEIN, Clifford. *Algoritmos: Teoria e Prática.* 3. ed. Rio de Janeiro: Elsevier, 2012.

DIJKSTRA, Edsger W. A Note on Two Problems in Connexion with Graphs. *Numerische Mathematik*, v. 1, p. 269–271, 1959.

HARABOR, Daniel; GRASTIEN, Alban. Online Graph Pruning for Pathfinding on Grid Maps. In: *Proceedings of the AAAI Conference on Artificial Intelligence*, 2011.

HART, Peter E.; NILSSON, Nils J.; RAPHAEL, Bertram. A Formal Basis for the Heuristic Determination of Minimum Cost Paths. *IEEE Transactions on Systems Science and Cybernetics*, v. 4, n. 2, p. 100–107, 1968.

MILLINGTON, Ian. *AI for Games.* 3. ed. Boca Raton: CRC Press, 2019.

BOURG, David M.; SEEMANN, Glenn. *AI for Game Developers.* Sebastopol, CA: O'Reilly Media, 2004.

RABIN, Steve; STURTEVANT, Nathan R. JPS+: An Extreme A\* Speed Optimization for Static Uniform Cost Grids. In: RABIN, Steve (org.). *Game AI Pro 2*. Boca Raton: A K Peters/CRC Press, 2015.

RABIN, Steve (org.). *Game AI Pro 3: Collected Wisdom of Game AI Professionals.* Boca Raton: A K Peters/CRC Press, 2017.

RUSSELL, Stuart J.; NORVIG, Peter. *Inteligência Artificial.* 3. ed. Rio de Janeiro: Elsevier, 2013.

MONONEN, Mikko. *Recast & Detour: Navigation Mesh Toolset.* Documentação do projeto de código aberto.

UNITY TECHNOLOGIES. *Documentação oficial: AI Navigation (NavMesh Surface, NavMesh Agent, NavMesh Modifier, Off-Mesh Links).*

GRANBERG, Aron. *A\* Pathfinding Project — Documentação.*

> **Nota sobre fontes e atribuições.** As atribuições de técnicas de pathfinding (A\*, JPS+, flow fields) a jogos e gêneros específicos constituem **análise técnica fundamentada** na estrutura observável do movimento e no que a indústria documenta em fontes como a série *Game AI Pro* e as palestras da GDC, não afirmações confirmadas caso a caso pelos estúdios. A distinção entre *fato documentado* e *análise técnica* será exercitada formalmente na engenharia reversa da Parte VII.

---

## Encerramento

Com a Parte III, o leitor domina a segunda grande família da IA de jogos: o **movimento e a busca de caminhos**. Aprendeu a converter o mundo em um **grafo** (grade, waypoints ou NavMesh), a percorrê-lo com o **A\*** e a compreender *por que* ele funciona (heurística, admissibilidade, `f = g + h`), e a acelerá-lo com **JPS+** e as demais otimizações — sempre sob a disciplina de **medir antes de otimizar** e de escolher a técnica que casa com o problema. Mais do que algoritmos, adquiriu um modo de pensar: o movimento inteligente é um exercício permanente de **trade-offs** entre otimalidade, velocidade e recursos.

A **Parte IV — Raciocínio Espacial e Tático** dará o próximo passo natural. O mesmo grafo do mundo que aqui usamos para *mover* agentes será reaproveitado para uma pergunta mais sofisticada: não apenas "*como* chegar até um ponto?", mas "*onde*, no mapa, é vantajoso agir?". Os **mapas de influência** transformarão a representação espacial desta Parte em um raciocínio tático sobre território, perigo e oportunidade — e o conceito de **custo** que aqui guiou a busca reaparecerá, agora como campo de influência que molda decisões. A base espacial que acabamos de construir não se encerra aqui: ela é o solo sobre o qual a IA de jogos ergue seu raciocínio sobre o espaço.
