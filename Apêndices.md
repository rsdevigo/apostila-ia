# Apêndices

## Inteligência Artificial e Ilusão de Inteligência — Curso Superior de Tecnologia em Jogos Digitais

> **Natureza deste material.** Os apêndices são parte integrante da apostila e servem como **material permanente de consulta**. Eles não introduzem conteúdo novo: **consolidam, organizam e facilitam** o acesso ao que foi ensinado nas Partes I a VII. Sempre que um assunto exigir aprofundamento, o apêndice remete ao capítulo correspondente. Conforme o Planejamento Editorial, esta seção reúne quatro apêndices:
>
> - **Apêndice A — Glossário de termos técnicos**
> - **Apêndice B — Guia rápido de ferramentas Unity de IA**
> - **Apêndice C — Tabela-decisão e tabelas de consulta**
> - **Apêndice D — Referências e leituras recomendadas**

---

# Apêndice A — Glossário de Termos Técnicos

Este glossário reúne os termos técnicos empregados ao longo da apostila. Cada verbete traz uma **definição objetiva**, o **contexto de uso em IA para jogos** e a **referência ao capítulo** onde o assunto é apresentado ou aprofundado. Os termos estão em ordem alfabética; quando um verbete depende de outro, há remissão cruzada com "→".

Para uma leitura orientada, o glossário pode ser cruzado com a **tabela-decisão** (Apêndice C) e com o **guia de ferramentas** (Apêndice B).

## A

**A\*** — Algoritmo de busca informada de menor caminho em grafo que combina o custo real acumulado `g(n)` com uma heurística `h(n)` na função de avaliação `f(n) = g(n) + h(n)`, expandindo sempre o nó de menor `f`. *Contexto:* é o padrão de fato do pathfinding em jogos; roda por baixo dos sistemas de navegação de Unity e Unreal. *Ver:* Cap. 8.

**Admissível (heurística)** — → Heurística admissível.

**Agente** — Entidade que percebe seu ambiente por meio de sensores e age sobre ele por meio de atuadores, seguindo um ciclo de decisão. *Contexto:* todo NPC, inimigo ou companheiro controlado por IA é modelado como um agente. *Ver:* Cap. 1.

**Agente racional** — Agente que escolhe, a cada instante, a ação que maximiza seu desempenho esperado dado o que percebe e sabe. *Contexto:* na IA de jogos, a racionalidade é subordinada à experiência do jogador — o agente deve parecer competente, não ser ótimo. *Ver:* Cap. 1.

**Algoritmo genético (AG)** — Metaheurística de busca inspirada na evolução natural que faz evoluir uma população de soluções candidatas por seleção, cruzamento e mutação, guiada por uma função de aptidão. *Contexto:* usado para ajuste de parâmetros de IA, geração de conteúdo e evolução de comportamento. *Ver:* Cap. 13.

**Ambiente** — Tudo aquilo sobre o qual o agente atua e que ele percebe; no jogo, o nível, os objetos e os demais agentes. *Contexto:* a definição do ambiente (observável ou não, determinístico ou estocástico) condiciona a escolha da técnica. *Ver:* Cap. 1; ambientes adversariais em Cap. 11.

**Aprendizagem por reforço (RL)** — Paradigma em que um agente aprende uma política de ação por tentativa e erro, maximizando uma recompensa acumulada obtida da interação com o ambiente. *Contexto:* base da "IA adaptativa"; conecta-se ao Unity ML-Agents. *Ver:* Cap. 12.

**Aptidão (fitness)** — Medida numérica da qualidade de um indivíduo de uma população evolutiva, usada para orientar a seleção. *Contexto:* projetar bem a função de aptidão é o passo mais crítico e mais difícil de um AG. *Ver:* Cap. 13.

**Aresta** — Ligação entre dois vértices de um grafo, podendo ter peso e direção. *Contexto:* representa a conexão navegável entre duas posições (células, waypoints ou polígonos de NavMesh). *Ver:* Cap. 7.

**Árvore de comportamento (Behavior Tree)** — Estrutura hierárquica de nós que, avaliada periodicamente (tick), decide a ação de um agente combinando nós compostos (sequência, seletor, paralelo), decoradores e folhas de ação/condição, com retornos de sucesso, falha ou execução. *Contexto:* arquitetura de decisão dominante na indústria; técnica de ênfase da Parte II. *Ver:* Cap. 6.

**Árvore de decisão** — Estrutura em árvore em que nós internos testam condições e as folhas indicam ações ou classes; percorrida da raiz à folha para escolher uma ação. *Contexto:* ponte conceitual para a árvore de comportamento, com a qual não deve ser confundida. *Ver:* Cap. 5.

**Árvore de jogo** — Representação de um jogo em que nós são estados e arestas são jogadas legais, alternando camadas de decisão dos jogadores. *Contexto:* estrutura sobre a qual operam o Minimax e a poda alfa-beta. *Ver:* Cap. 11.

## B

**Blackboard** — Área de memória compartilhada onde nós de uma árvore de comportamento (ou outros sistemas) leem e escrevem dados, desacoplando percepção e decisão. *Contexto:* evita que os nós conheçam uns aos outros diretamente; presente no pacote Unity Behavior. *Ver:* Cap. 6.

**Busca informada** — Busca que usa conhecimento sobre o domínio (uma heurística) para priorizar a exploração rumo ao objetivo. *Contexto:* torna o pathfinding viável em tempo real; o A\* é o exemplo central. *Ver:* Cap. 8.

**Busca não-informada** — Busca que explora o grafo sem estimar a distância ao objetivo, como o Dijkstra. *Contexto:* garante otimalidade, mas expande muitos nós desnecessários. *Ver:* Cap. 8.

## C

**Camada de influência** — Mapa escalar dedicado a um aspecto tático (presença inimiga, perigo, controle de território), que pode ser combinado com outras camadas. *Contexto:* somar/subtrair camadas produz mapas derivados (tensão, vulnerabilidade). *Ver:* Cap. 10.

**Camada MAX / camada MIN** — Níveis alternados da árvore de jogo no Minimax: MAX busca maximizar a utilidade (o agente); MIN busca minimizá-la (o oponente). *Ver:* Cap. 11.

**Ciclo Sentir–Pensar–Agir** — Laço fundamental do agente: perceber o ambiente (sentir), decidir (pensar) e executar a ação (agir), repetido a cada quadro ou intervalo. *Contexto:* organiza toda a apostila; cada família de técnicas atua sobre uma fase do ciclo. *Ver:* Cap. 1.

**Comportamento deliberativo** — Comportamento que planeja com antecedência, raciocinando sobre estados futuros antes de agir. *Contexto:* mais flexível e mais caro; exemplos: GOAP, planejamento, Minimax. *Ver:* Cap. 1.

**Comportamento emergente** — Padrão complexo e não programado explicitamente que surge da interação de regras simples. *Contexto:* fonte poderosa da "ilusão de inteligência" (ex.: personalidades dos fantasmas de Pac-Man). *Ver:* Cap. 1; Cap. 2; Cap. 15.

**Comportamento reativo** — Comportamento que responde diretamente ao estímulo presente, sem planejar o futuro. *Contexto:* barato e previsível; base das FSMs e de árvores de comportamento simples. *Ver:* Cap. 1.

**Computação evolutiva** — Família de metaheurísticas inspiradas na evolução biológica, da qual os algoritmos genéticos são o principal representante. *Ver:* Cap. 13.

**Consistente (heurística)** — → Heurística consistente.

**Cromossomo** — Representação codificada de uma solução candidata em um algoritmo genético, composta por genes. *Contexto:* a forma de codificar o cromossomo determina o espaço de busca. *Ver:* Cap. 13.

**Cruzamento (crossover)** — Operador genético que combina genes de dois indivíduos-pais para gerar descendentes. *Contexto:* mecanismo de recombinação que mistura soluções promissoras. *Ver:* Cap. 13.

**Curva de utilidade** — Função que mapeia uma variável do jogo (fome, distância, munição) em um valor de utilidade normalizado, usada pela IA de utilidade para pontuar opções. *Ver:* Cap. 6.

**Custo g** — → Função f, custo g, heurística h.

## D

**Decaimento** — Redução gradual do valor de influência com a distância à fonte, durante a propagação em um mapa de influência. *Ver:* Cap. 10.

**Decorador** — Nó de uma árvore de comportamento que envolve um único filho e modifica seu resultado ou execução (inverter, repetir, condicionar, limitar por tempo). *Ver:* Cap. 6.

**Deep Reinforcement Learning (Deep RL)** — Aprendizagem por reforço em que a política ou a função valor é representada por redes neurais profundas, permitindo lidar com estados de alta dimensão. *Contexto:* conteúdo de aprofundamento; base de marcos como AlphaGo e agentes de StarCraft II. *Ver:* Cap. 12 (seção de aprofundamento).

**Diretor de IA (AI Director)** — Sistema de alto nível que ajusta o ritmo do jogo em tempo real (intensidade, geração de inimigos, recursos) para moldar a experiência. *Contexto:* exemplo canônico em Left 4 Dead. *Ver:* Cap. 15.

**Dijkstra** — Algoritmo clássico de menor caminho que expande nós por ordem de custo acumulado, sem heurística. *Contexto:* caso particular do A\* com `h = 0`; fundamento histórico do pathfinding. *Ver:* Cap. 8.

**Distância de Manhattan / Euclidiana / Chebyshev** — Heurísticas de estimativa de distância em grade: Manhattan (movimentos ortogonais, conectividade-4); Euclidiana (linha reta, movimento livre); Chebyshev/octile (conectividade-8 com diagonais). *Contexto:* a heurística deve casar com a conectividade da grade para preservar otimalidade. *Ver:* Cap. 8.

## E

**Efeito** — Em GOAP, a mudança de estado que uma ação produz no mundo quando executada. *Ver:* Cap. 6.

**Elitismo** — Estratégia de um AG que preserva os melhores indivíduos de uma geração para a seguinte, evitando perder boas soluções. *Ver:* Cap. 13.

**Engenharia reversa (de IA)** — Método de inferir, pela observação sistemática do comportamento, qual(is) técnica(s) de IA um jogo provavelmente emprega. *Contexto:* competência integradora da apostila; conclui-se em inferências fundamentadas, não em fatos documentados. *Ver:* Cap. 14; Cap. 15.

**Estado** — Configuração de comportamento em que um agente permanece até que um evento ou condição provoque uma transição (ex.: Patrulhar, Perseguir, Atacar). *Ver:* Cap. 3.

**Estado de histórico** — Mecanismo de uma HFSM que memoriza o último subestado ativo de um superestado, permitindo retomá-lo ao reentrar. *Contexto:* dá "memória" à hierarquia (origem formal nos statecharts de Harel). *Ver:* Cap. 4.

**Evento** — Ocorrência que dispara a avaliação de transições em uma FSM (ex.: "viu o jogador", "vida baixa"). *Contexto:* FSMs por eventos reagem a estes gatilhos; FSMs por polling os verificam a cada quadro. *Ver:* Cap. 3.

**Exploração versus explotação** — Dilema central da aprendizagem por reforço: explorar ações novas para descobrir recompensas ou explotar o que já se sabe ser bom. *Ver:* Cap. 12.

## F

**Folha** — Nó terminal de uma árvore. Em árvore de decisão, indica a ação/classe escolhida; em árvore de comportamento, é um nó de ação ou de condição. *Ver:* Cap. 5; Cap. 6.

**Frame budget (orçamento de quadro)** — Fração do tempo de um quadro (ex.: alguns milissegundos dentro de 16,7 ms a 60 fps) que a IA pode consumir sem prejudicar a taxa de quadros. *Contexto:* restrição prática que domina toda decisão de projeto de IA de jogo. *Ver:* Cap. 1.

**Função de avaliação** — Função heurística que estima a qualidade de um estado não terminal em um jogo adversarial, permitindo interromper a busca antes do fim. *Contexto:* preenche o "horizonte" do Minimax. *Ver:* Cap. 11.

**Função f, custo g, heurística h** — Componentes do A\*: `g(n)` é o custo real do início até `n`; `h(n)` é a estimativa de `n` até o objetivo; `f(n) = g(n) + h(n)` orienta a expansão. *Ver:* Cap. 8.

**Função valor** — Em RL, função que estima o retorno esperado a partir de um estado (ou par estado-ação), como a tabela Q. *Ver:* Cap. 12.

## G

**Gene** — Unidade elementar de um cromossomo em um AG, correspondente a um parâmetro ou traço da solução. *Ver:* Cap. 13.

**GOAP (Goal-Oriented Action Planning)** — Arquitetura de decisão que, a partir de um objetivo, encadeia ações com pré-condições e efeitos para montar dinamicamente um plano. *Contexto:* conteúdo de aprofundamento; caso emblemático em F.E.A.R. *Ver:* Cap. 6; Cap. 15.

**Grade (grid)** — Representação do espaço em células regulares, cada uma um vértice de um grafo implícito. *Contexto:* comum em jogos 2D, de estratégia e tower defense; base do JPS+. *Ver:* Cap. 7.

**Grafo** — Estrutura formada por vértices e arestas que modela relações; em jogos, representa o espaço navegável. *Contexto:* fundamento matemático de A\*, JPS+ e mapas de influência. *Ver:* Cap. 7.

**Guarda de transição** — Condição que precisa ser verdadeira para que uma transição entre estados ocorra. *Ver:* Cap. 3.

## H

**Heurística** — Estimativa ou regra prática que orienta a busca sem garantir a solução ótima. *Contexto:* aparece no A\* (Cap. 8), nas funções de avaliação do Minimax (Cap. 11) e como guarda-chuva dos algoritmos genéticos (Cap. 13). *Ver:* Cap. 8.

**Heurística admissível** — Heurística que nunca superestima o custo real restante (é otimista); condição que garante a otimalidade do A\*. *Ver:* Cap. 8.

**Heurística consistente** — Heurística que respeita a desigualdade triangular; garante que um nó, uma vez fechado, não precisa ser reaberto. *Ver:* Cap. 8.

**Horizonte** — Profundidade máxima até onde a busca adversarial consegue enxergar; além dela, recorre-se à função de avaliação. *Contexto:* o "efeito horizonte" ocorre quando um evento decisivo fica logo além do limite de busca. *Ver:* Cap. 11.

**HFSM (Máquina de estado hierárquica finita)** — → Máquina de estado hierárquica.

## I

**IA de utilidade (Utility AI)** — Arquitetura que pontua cada ação possível por meio de curvas de utilidade e escolhe (ou sorteia proporcionalmente) a de maior valor. *Contexto:* aprofundamento; caso clássico em The Sims. *Ver:* Cap. 6; Cap. 15.

**Ilusão de inteligência** — Tese central da disciplina: a IA de jogos busca produzir comportamento **convincente** e divertido, não inteligência "verdadeira". *Contexto:* critério para julgar as técnicas pelos valores da indústria (credibilidade, diversão, custo, controle do designer). *Ver:* Cap. 1.

## J

**Jogo de soma zero** — Jogo em que o ganho de um jogador equivale exatamente à perda do outro. *Contexto:* premissa do Minimax. *Ver:* Cap. 11.

**JPS / JPS+ (Jump Point Search)** — Otimização do A\* para grades uniformes que "salta" sobre nós simétricos, reduzindo drasticamente a expansão; o JPS+ pré-computa saltos para acelerar ainda mais. *Contexto:* ganho expressivo de desempenho ao custo de restrições (grade uniforme, pré-processamento estático). *Ver:* Cap. 9.

## L

**Lista aberta** — Fronteira de nós ainda por expandir no A\*, tipicamente implementada como fila de prioridade ordenada por `f`. *Ver:* Cap. 8.

**Lista fechada** — Conjunto de nós já expandidos (resolvidos) no A\*, evitando reprocessamento. *Ver:* Cap. 8.

## M

**Malha de navegação (NavMesh)** — Representação do espaço navegável como um conjunto de polígonos convexos conectados, sobre os quais se faz pathfinding. *Contexto:* padrão em jogos 3D; gerada e usada nativamente na Unity. *Ver:* Cap. 7.

**Mapa de influência** — Campo escalar sobreposto ao mapa que registra a intensidade de algum fator tático (presença, perigo, controle) em cada posição, atualizado por propagação e decaimento. *Contexto:* responde "onde agir?", complementando o "como chegar?" do pathfinding. *Ver:* Cap. 10.

**Máquina de estado finita (FSM)** — Modelo de decisão em que o agente está sempre em exatamente um estado e transita entre estados por eventos/condições. *Contexto:* técnica de decisão mais intuitiva; mapeia ao Animator da Unity. *Ver:* Cap. 3.

**Máquina de estado hierárquica (HFSM)** — Extensão da FSM que agrupa estados em superestados, com herança de transições e estado de histórico, combatendo a explosão de transições. *Contexto:* corresponde às sub-state machines do Animator. *Ver:* Cap. 4.

**MDP (Processo de decisão de Markov)** — Modelo matemático de decisão sequencial definido por estados, ações, transições e recompensas, em que o futuro depende apenas do estado atual. *Contexto:* formalismo que fundamenta a aprendizagem por reforço. *Ver:* Cap. 12.

**Metaheurística** — Estratégia de busca de alto nível, independente de domínio, que guia heurísticas para explorar grandes espaços de solução (ex.: algoritmos genéticos). *Ver:* Cap. 13.

**Minimax** — Algoritmo de decisão para jogos adversariais de soma zero que propaga, pela árvore de jogo, valores de utilidade assumindo jogo ótimo de ambos os lados. *Ver:* Cap. 11.

**ML-Agents** — Kit oficial da Unity para treinar agentes por aprendizagem por reforço (e imitação) integrando a engine a bibliotecas de aprendizado. *Ver:* Cap. 12; Apêndice B.

**Monte Carlo Tree Search (MCTS)** — Algoritmo de busca que estima o valor das jogadas por simulações aleatórias (rollouts), equilibrando exploração e explotação; estado da arte em jogos como Go. *Contexto:* conteúdo de aprofundamento. *Ver:* Cap. 11 (seção de aprofundamento).

**Mutação** — Operador genético que altera aleatoriamente genes de um indivíduo, mantendo diversidade e evitando convergência prematura. *Ver:* Cap. 13.

## N

**Nó composto** — Nó de uma árvore de comportamento que orquestra filhos: sequência (executa em ordem até uma falha), seletor (tenta até um sucesso) ou paralelo (executa simultaneamente). *Ver:* Cap. 6.

**Nó de decisão** — Nó interno de uma árvore de decisão que testa uma condição e escolhe o ramo a seguir. *Ver:* Cap. 5.

## O

**Objeto inteligente (smart object)** — Objeto do mundo que anuncia aos agentes as ações que oferece e seus efeitos, descentralizando a lógica de comportamento. *Contexto:* mecanismo central da IA de utilidade de The Sims. *Ver:* Cap. 15.

## P

**Pathfinding hierárquico** — Técnica que divide o mapa em regiões e resolve o caminho em dois níveis (entre regiões e dentro delas), reduzindo o custo em mapas grandes. *Ver:* Cap. 9.

**Peso** — Valor associado a uma aresta que representa o custo de percorrê-la (distância, dificuldade de terreno). *Ver:* Cap. 7; usado como custo em `g` no Cap. 8.

**Percepção / sensoriamento** — Processo pelo qual o agente obtém informação do ambiente (visão, audição, memória), definindo o que ele "sabe". *Contexto:* boa parte da ilusão de inteligência vem de modelar percepção limitada (o inimigo que "não viu" o jogador). *Ver:* Cap. 1; Cap. 15 (Alien Isolation).

**Poda alfa-beta** — Otimização do Minimax que elimina ramos que não podem influenciar a decisão, sem alterar o resultado, permitindo buscar mais fundo. *Ver:* Cap. 11.

**Política** — Em RL, o mapeamento de estados para ações que define o comportamento do agente; o objetivo do aprendizado é encontrar a política ótima. *Ver:* Cap. 12.

**Pré-condição** — Em GOAP, a condição do mundo que precisa ser satisfeita para que uma ação possa ser executada. *Ver:* Cap. 6.

**Propagação** — Difusão do valor de uma fonte de influência para as células vizinhas de um mapa de influência, combinada com decaimento. *Ver:* Cap. 10.

## Q

**Q-Learning** — Algoritmo de RL sem modelo que aprende a função valor-ação `Q(s,a)` por atualização iterativa, convergindo para a política ótima. *Contexto:* base didática do aprendizado por reforço tabular. *Ver:* Cap. 12.

## R

**Recompensa** — Sinal numérico que o ambiente devolve ao agente de RL, indicando o quão desejável foi o resultado de uma ação; guia todo o aprendizado. *Ver:* Cap. 12.

## S

**Seleção** — Operador genético que escolhe indivíduos para reprodução com probabilidade proporcional (ou relacionada) à aptidão. *Ver:* Cap. 13.

**Simetria de caminhos** — Fenômeno em grades uniformes em que muitos caminhos de mesmo custo são equivalentes; explorá-los todos é desperdício que o JPS elimina. *Ver:* Cap. 9.

**Subestado** — Estado contido dentro de um superestado em uma HFSM. *Ver:* Cap. 4.

**Suavização de caminho (path smoothing)** — Pós-processamento que remove "degraus" de um caminho em grade, aproximando-o de uma trajetória natural por linha de visão. *Ver:* Cap. 9.

**Superestado** — Estado que agrupa um conjunto de subestados em uma HFSM, permitindo transições e regras comuns escritas uma única vez. *Ver:* Cap. 4.

## T

**Transição** — Passagem de um estado a outro em uma FSM, disparada por um evento ou condição (guarda). *Ver:* Cap. 3.

## U

**Utility AI** — → IA de utilidade.

## V

**Vértice** — Elemento fundamental de um grafo, ligado a outros por arestas; em jogos, uma posição navegável (célula, waypoint, polígono). *Ver:* Cap. 7.

## W

**Waypoint** — Ponto de rota predefinido no cenário; conjuntos de waypoints conectados formam um grafo de navegação esparso. *Ver:* Cap. 7.

---

# Apêndice B — Guia Rápido de Ferramentas Unity de IA

Este apêndice consolida, em formato de consulta, **como cada conceito da apostila se materializa em ferramentas**. O objetivo é o mesmo dos capítulos: **contextualizar**, não ensinar menus. Para o aprofundamento de cada técnica, consulte o capítulo indicado. As ferramentas de terceiros e as equivalências na Unreal Engine aparecem apenas como referência comparativa.

> **Atenção**
> A apostila não é um manual de Unity. As ferramentas evoluem e mudam de nome entre versões (por exemplo, o pacote **Unity Behavior** substituiu, na prática, o antigo Bolt/Behavior de terceiros para grafos oficiais). Trate a coluna "ferramenta oficial" como o ponto de partida atual, sempre confirmando na documentação da versão em uso.

## B.1 Relação conceito → ferramenta

A tabela relaciona cada conceito estudado à ferramenta **oficial** da Unity (quando existe), a uma alternativa **de terceiros** relevante e ao **capítulo** de origem. Quando a Unity não oferece sistema nativo dedicado, indica-se a construção "sobre o básico" e a comparação com a Unreal.

| Conceito | Ferramenta oficial Unity | Ferramenta de terceiros / comparação | Cap. |
|---|---|---|---|
| Máquina de estado (FSM) | Animator Controller (estados e transições); Visual Scripting (State Graphs); pacote **Unity Behavior** | Panda BT; comparação: Unreal State Machine (Anim Blueprints) | 3 |
| Máquina de estado hierárquica (HFSM) | Animator **sub-state machines** e Animator Layers; hierarquias no pacote **Unity Behavior** | Comparação: Unreal **State Tree** (HFSM nativa) | 4 |
| Árvore de decisão | Visual Scripting (lógica condicional); C# | NodeCanvas; conceito também presente em Behavior Designer | 5 |
| Árvore de comportamento (Behavior Tree) | Pacote **Unity Behavior** (Behavior Trees + Blackboard) | Behavior Designer (Opsive); NodeCanvas; Panda BT; comparação: Unreal Behavior Tree | 6 |
| GOAP / IA de utilidade *(aprofundamento)* | (sem sistema nativo dedicado; construído em C# sobre o Blackboard) | Bibliotecas GOAP open source; assets de Utility AI da Asset Store | 6 |
| Grafos e representação do espaço | AI Navigation (geração de NavMesh, NavMesh Surface/Modifier) | A\* Pathfinding Project (grades e grafos de pontos) | 7 |
| Busca de caminho A\* | **NavMesh Agent** (A\* embutido sobre o grafo de polígonos) | A\* Pathfinding Project (Aron Granberg); Recast & Detour (open source) | 8 |
| Otimização JPS+ e pathfinding hierárquico | (não exposto nativamente; NavMesh já otimiza internamente) | A\* Pathfinding Project (variantes hierárquicas, multithreading) | 9 |
| Mapa de influência | (sem sistema nativo; construído sobre grid/NavMesh) | Assets de influence map da Asset Store; comparação: Unreal **EQS** | 10 |
| Minimax e busca adversarial | (conceitual; implementado em C# sobre a lógica do jogo) | Bibliotecas open source de busca em árvore de jogo | 11 |
| Aprendizagem por reforço | **ML-Agents** (treino por RL e imitação) | Comparação: Unreal **Learning Agents** | 12 |
| Algoritmos genéticos / heurísticas | (conceitual; implementado em C#) | Bibliotecas de GA em C#; frameworks de neuroevolução | 13 |
| Engenharia reversa / estudos de caso | (todas as anteriores, aplicadas em análise) | — | 14, 15 |

> **Na Prática**
> A maior parte da IA "de gameplay" em jogos comerciais Unity é montada com **três blocos**: NavMesh Agent para movimento, um sistema de decisão (Animator/Unity Behavior ou um asset de Behavior Tree) para escolher ações, e código C# próprio para percepção e regras específicas. O ML-Agents é a exceção avançada, mais comum em pesquisa, QA automatizado e protótipos do que em produção.

## B.2 Recursos online comentados

Lista de referência de documentações oficiais e projetos, priorizando os materiais usados na construção da apostila. Os endereços podem mudar entre versões da engine; em caso de link quebrado, busque pelo nome do sistema na documentação atual.

**Documentação oficial Unity — sistemas de IA nativos**

- **AI Navigation / NavMesh (Manual):** `https://docs.unity3d.com/6000.3/Documentation/Manual/com.unity.ai.navigation.html` — geração e uso de malhas de navegação; base dos Caps. 7 e 8.
- **Building a NavMesh:** `https://docs.unity3d.com/Manual/nav-BuildingNavMesh.html` — como o espaço navegável é gerado. *Cap. 7.*
- **Animator Controller:** `https://docs.unity3d.com/Manual/AnimatorControllers.html` — materialização de FSM/HFSM. *Caps. 3 e 4.*
- **Animator — State Machine Transitions:** `https://docs.unity3d.com/Manual/StateMachineTransitions.html` — transições e condições. *Cap. 3.*
- **Unity Behavior (pacote):** `https://docs.unity3d.com/Packages/com.unity.behavior@1.0/manual/index.html` — árvores/grafos de comportamento oficiais. *Cap. 6 (e Caps. 3–4).*
- **Unity Visual Scripting — State Graphs:** `https://docs.unity3d.com/Packages/com.unity.visualscripting@1.8/manual/vs-graph-machine-types.html` — FSM nativa independente de animação. *Cap. 3.*
- **ML-Agents (documentação):** `https://unity-technologies.github.io/ml-agents/` — treino de agentes por RL. *Cap. 12.*
- **ML-Agents (repositório):** `https://github.com/Unity-Technologies/ml-agents` — código, exemplos e ambientes. *Cap. 12.*
- **Unity Learn:** `https://learn.unity.com/` — cursos e tutoriais oficiais; útil para exercitar NavMesh, Animator e ML-Agents na prática.

**Documentação oficial Unreal Engine — comparações**

- **State Tree (HFSM nativa):** `https://dev.epicgames.com/documentation/unreal-engine/overview-of-state-tree-in-unreal-engine` — comparação no Cap. 4.
- **Behavior Trees (com Blackboard):** `https://dev.epicgames.com/documentation/en-us/unreal-engine/behavior-trees-in-unreal-engine` — comparação no Cap. 6.
- **Navigation / Pathfinding (A\*):** `https://dev.epicgames.com/documentation/unreal-engine/basic-navigation-in-unreal-engine` — comparação no Cap. 8.
- **Environment Query System (EQS):** `https://dev.epicgames.com/documentation/en-us/unreal-engine/environment-query-system-in-unreal-engine` — o equivalente mais próximo de mapas de influência; comparação no Cap. 10.
- **Learning Agents (RL):** `https://dev.epicgames.com/documentation/en-us/unreal-engine/API/PluginIndex/LearningAgents` — comparação no Cap. 12.

**Projetos e bibliotecas de terceiros**

- **A\* Pathfinding Project (Aron Granberg):** biblioteca de pathfinding para Unity sobre grades, grafos de pontos e malhas, com otimizações modernas. *Caps. 8 e 9.*
- **Recast & Detour (open source):** ferramenta de geração de NavMesh (Recast) e busca sobre a malha (Detour), usada por engines próprias. *Caps. 7 e 8.*
- **Behavior Designer (Opsive) e NodeCanvas:** soluções maduras de árvore de comportamento na Asset Store. *Cap. 6.*

---

# Apêndice C — Tabela-Decisão e Tabelas de Consulta

Este apêndice reúne o material de consulta rápida da apostila: uma **tabela-decisão** que ajuda a escolher a técnica a partir do problema de jogo; **tabelas consolidadas** que comparam as arquiteturas e algoritmos estudados; uma **linha do tempo** da IA em jogos; e um **checklist para o professor**. Nenhuma dessas peças substitui os capítulos — todas remetem a eles para o aprofundamento.

## C.1 Tabela-decisão — "dado este problema de jogo, qual técnica usar?"

Ferramenta de revisão geral. Cada linha parte de um **problema típico** e sugere a **técnica recomendada**, **alternativas** e o **capítulo** de referência. As recomendações seguem a filosofia da apostila: preferir a técnica mais simples que resolve o problema dentro do orçamento de quadro e que preserva o controle do designer.

| Problema típico de jogo | Técnica recomendada | Alternativas | Cap. |
|---|---|---|---|
| Alternar poucos comportamentos discretos (patrulhar, perseguir, atacar, fugir) | Máquina de estado finita (FSM) | Árvore de comportamento; árvore de decisão | 3 |
| A FSM cresceu e virou uma "teia" de transições | Máquina de estado hierárquica (HFSM) | Árvore de comportamento | 4 |
| Comportamento complexo, modular e reutilizável entre personagens | Árvore de comportamento | HFSM; GOAP | 6 |
| Escolher uma ação a partir de muitas condições encadeadas | Árvore de decisão | Árvore de comportamento; IA de utilidade | 5 |
| Inimigo que "improvisa" um plano para atingir um objetivo | GOAP *(aprofundamento)* | Árvore de comportamento com muitos ramos | 6 |
| Agente que equilibra vários desejos concorrentes (fome, diversão, higiene) | IA de utilidade *(aprofundamento)* | Árvore de comportamento | 6 |
| Mover um NPC 3D de A a B desviando de obstáculos | A\* sobre NavMesh | Grafo de waypoints | 7, 8 |
| Menor caminho em grade grande, uniforme e estática | JPS+ | A\* ponderado; pathfinding hierárquico | 9 |
| Muitos agentes pedindo caminho ao mesmo tempo (RTS) | Pathfinding hierárquico + *time-slicing* | JPS+; flow fields; cache de caminhos | 8, 9 |
| Decidir **onde** se posicionar (cobertura, avanço, zona de perigo) | Mapa de influência | Consultas espaciais (estilo EQS) | 10 |
| Jogar bem um jogo de tabuleiro por turnos (xadrez, damas) | Minimax + poda alfa-beta | MCTS | 11 |
| Jogo com fator de ramificação gigantesco (Go) | MCTS *(aprofundamento)* | Alfa-beta com boa função de avaliação | 11 |
| Comportamento que a equipe não consegue programar à mão | Aprendizagem por reforço | Heurísticas/scripts ajustados manualmente | 12 |
| Otimizar parâmetros de IA num espaço de busca enorme | Algoritmo genético | Busca local; ajuste manual | 13 |
| Gerar conteúdo ou comportamento variado automaticamente | Algoritmo genético / PCG | Aprendizagem por reforço | 13 |
| Descobrir qual técnica um jogo comercial provavelmente usa | Metodologia de engenharia reversa | — | 14, 15 |

> **Boa Prática**
> Leia esta tabela de cima para baixo: as primeiras linhas trazem técnicas mais simples e baratas. Só desça para RL e algoritmos genéticos quando as técnicas determinísticas realmente não derem conta — na indústria, a solução mais simples que convence é quase sempre a vencedora.

## C.2 Tabela consolidada de arquiteturas e algoritmos

Visão comparativa das principais técnicas da apostila. O **custo computacional** é indicado de forma qualitativa (baixo/médio/alto) por ser o que mais importa na prática de jogos; os detalhes formais estão nos capítulos.

| Técnica | Família | Vantagens | Limitações | Custo | Aplicações típicas | Cap. |
|---|---|---|---|---|---|---|
| FSM | Decisão baseada em regras | Simples, previsível, fácil de depurar | Explosão de transições; pouca reutilização | Baixo | NPCs simples, inimigos clássicos | 3 |
| HFSM | Decisão baseada em regras | Reduz transições; organiza por camadas; memória de histórico | Ainda rígida em cenários muito dinâmicos | Baixo | Combate em camadas, IA de personagem | 4 |
| Árvore de decisão | Decisão baseada em regras | Leitura clara; mapeia condições a ações | Não guarda estado; cresce com as condições | Baixo | Seleção de ação por condições | 5 |
| Árvore de comportamento | Decisão baseada em regras | Modular, reutilizável, escalável; prioridades explícitas | Curva de projeto; blackboard exige disciplina | Baixo–médio | Guardas, combate, IA de ação (Halo) | 6 |
| GOAP | Planejamento | Improvisa planos; comportamento emergente | Custo de busca; difícil de depurar | Médio–alto | Esquadrões táticos (F.E.A.R.) | 6 |
| IA de utilidade | Decisão por pontuação | Lida com desejos concorrentes; comportamento fluido | Ajuste fino das curvas é trabalhoso | Baixo–médio | Simulação de vida (The Sims) | 6 |
| A\* | Busca informada | Ótimo e completo; geral; sintonizável | Memória por nó; caro com muitos agentes | Médio | Pathfinding em geral (NavMesh) | 8 |
| JPS+ | Busca informada (otimização) | Muito mais rápido que A\* em grades | Só grade uniforme; pré-processamento estático | Baixo (em execução) | RTS, 2D, tower defense | 9 |
| Mapa de influência | Raciocínio espacial/tático | Responde "onde agir"; combina camadas | Custo de atualização; depende da resolução | Médio | Estratégia (Age of Empires, Civilization) | 10 |
| Minimax + alfa-beta | Busca adversarial | Jogo forte em soma zero; alfa-beta corta ramos | Explosão combinatória; exige boa avaliação | Alto | Xadrez, damas, jogos de tabuleiro | 11 |
| MCTS | Busca adversarial (estatística) | Escala a ramificações enormes; não precisa de avaliação forte | Custo de simulações; variância | Alto | Go, jogos de ramificação alta | 11 |
| Aprendizagem por reforço | Aprendizado | Descobre comportamentos não programáveis à mão | Custo de treino; imprevisibilidade; pouco controle | Alto (treino) | IA adaptativa, QA, pesquisa | 12 |
| Algoritmo genético | Metaheurística/evolutiva | Explora espaços enormes; não exige gradiente | Sem garantia de ótimo; ajuste da aptidão | Médio–alto | Otimização de parâmetros, PCG | 13 |

## C.3 Linha do tempo da IA aplicada a jogos digitais

Síntese cronológica dos marcos discutidos, sobretudo no Capítulo 2 e retomados nos estudos de caso (Cap. 15). Inclui tanto marcos **técnicos/acadêmicos** que fundamentam as técnicas quanto **jogos-marco** que as popularizaram.

| Ano | Marco | Relevância para a IA de jogos | Cap. |
|---|---|---|---|
| 1950 | Shannon — "Programming a Computer for Playing Chess" | Fundamento da busca adversarial e do Minimax | 11 |
| 1959 | Dijkstra — algoritmo de menor caminho; Samuel — damas com autoaprendizado | Bases do pathfinding e do aprendizado | 8, 12 |
| 1968 | Hart, Nilsson & Raphael — algoritmo **A\*** | Nasce a busca informada de caminhos | 8 |
| 1980 | **Pac-Man** — fantasmas com "personalidades" | Ilusão de inteligência a partir de regras simples (FSM/perseguição) | 2, 15 |
| 1987 | Harel — **statecharts** | Formalização da hierarquia e do estado de histórico (raiz da HFSM) | 4 |
| 1997 | **Age of Empires**; Deep Blue vence Kasparov | Mapas de influência e pathfinding em RTS; auge do Minimax/alfa-beta | 10, 11 |
| 2000 | **The Sims** | IA de utilidade e objetos inteligentes | 6, 15 |
| 2001 | **Black & White** | Aprendizado da criatura (RL/reforço aplicado a jogo comercial) | 12, 15 |
| 2004 | **Halo 2** (Isla) | Popularização das **árvores de comportamento** | 6, 15 |
| 2005 | **F.E.A.R.** (Orkin) | **GOAP** e a ilusão de esquadrão inteligente | 6, 15 |
| 2008 | **Left 4 Dead** | **Diretor de IA** e ritmo adaptativo | 15 |
| 2011 | Harabor & Grastien — **JPS** | Otimização de pathfinding em grades | 9 |
| 2013 | **The Last of Us** | IA de companheiro e furtividade | 15 |
| 2014 | **Alien Isolation** | Sistema de "dois cérebros" e sensoriamento | 15 |
| 2015 | **JPS+** (Rabin & Sturtevant); DQN joga Atari (Mnih et al.) | Pré-processamento de saltos; ascensão do Deep RL | 9, 12 |
| 2016 | **AlphaGo** vence o Go | MCTS + aprendizado profundo derrotam humanos no Go | 11, 12 |
| 2017 | Lançamento do **Unity ML-Agents** | RL acessível dentro de uma engine comercial | 12 |
| 2018–2019 | **AlphaZero**; **AlphaStar** (StarCraft II) | Autoaprendizado geral; RL em jogo comercial complexo | 11, 12 |

> **Curiosidade**
> A distância entre a teoria e o uso comercial é enorme na linha do tempo: o A\* é de 1968, mas só se tornou onipresente nos jogos décadas depois, quando o hardware permitiu recalculá-lo dezenas de vezes por segundo para muitos agentes. A história da IA de jogos é, em boa medida, a história do hardware que passou a caber no orçamento de quadro.

## C.4 Checklist para o professor

Guia de apoio didático, organizado por Parte. Para cada uma, reúne os **conceitos essenciais**, as **dificuldades** e **erros conceituais** mais comuns dos estudantes, e **sugestões de demonstração prática**. A última linha aponta os **capítulos que exigem maior atenção**.

**Parte I — Fundamentos e ilusão de inteligência**

- *Conceitos essenciais:* agente e ciclo Sentir–Pensar–Agir; ilusão de inteligência; IA de jogos × IA acadêmica; frame budget; critérios de qualidade (credibilidade, diversão, controle).
- *Dificuldades comuns:* aceitar que "não ótimo" pode ser melhor para o jogo; entender que a IA serve à experiência, não à competição.
- *Erros conceituais recorrentes:* achar que IA de jogo é "IA de verdade"; confundir dificuldade com inteligência.
- *Demonstração sugerida:* comparar dois inimigos — um "burro" divertido e um "perfeito" frustrante — para discutir o ponto ideal de dificuldade.

**Parte II — Decisão baseada em regras (FSM, HFSM, árvores de decisão e de comportamento)**

- *Conceitos essenciais:* estado, transição, evento, guarda; hierarquia e histórico; nós compostos, decoradores, blackboard; tick e retornos success/failure/running.
- *Dificuldades comuns:* distinguir polling de eventos; entender o fluxo de execução de uma árvore de comportamento.
- *Erros conceituais recorrentes:* **confundir árvore de decisão com árvore de comportamento** (o erro mais frequente da Parte); tratar a árvore de comportamento como se tivesse "estado" implícito.
- *Demonstração sugerida:* o mesmo inimigo guarda modelado como FSM (Cap. 3), HFSM (Cap. 4) e árvore de comportamento (Cap. 6), mostrando a mesma lógica em três arquiteturas; no Animator, exibir a explosão de transições.

**Parte III — Movimento e pathfinding (grafos, A\*, JPS+)**

- *Conceitos essenciais:* grafo, vértice, aresta, peso; `f = g + h`; admissibilidade e consistência; listas aberta/fechada; simetria de caminhos.
- *Dificuldades comuns:* casar a heurística com a conectividade da grade; entender por que o A\* é ótimo.
- *Erros conceituais recorrentes:* usar Manhattan em grade com diagonais; achar que uma heurística "melhor" é sempre a que dá o valor maior (superestimar quebra a otimalidade).
- *Demonstração sugerida:* traçar A\* passo a passo em uma grade no quadro; comparar visualmente nós expandidos por A\* e por JPS; gerar uma NavMesh na Unity e mover um agente.

**Parte IV — Raciocínio espacial e tático (mapas de influência)**

- *Conceitos essenciais:* campo escalar; fonte, propagação, decaimento; combinação de camadas.
- *Dificuldades comuns:* separar "como chegar" (pathfinding) de "onde ir" (influência); calibrar decaimento e resolução.
- *Erros conceituais recorrentes:* confundir mapa de influência com mapa de calor de pathfinding; atualizar o mapa todo quadro sem necessidade.
- *Demonstração sugerida:* heatmap de influência sobre um mapa de RTS, alternando camadas (presença, perigo) e mostrando o mapa derivado.

**Parte V — IA adversarial (Minimax e alfa-beta)**

- *Conceitos essenciais:* jogo de soma zero; árvore de jogo; camadas MAX/MIN; função de avaliação; poda alfa-beta; horizonte.
- *Dificuldades comuns:* propagar valores corretamente na árvore; entender por que a poda não altera o resultado.
- *Erros conceituais recorrentes:* achar que Minimax "resolve" qualquer jogo (ignorar a explosão combinatória); esquecer que a função de avaliação é heurística.
- *Demonstração sugerida:* jogo da velha completo no quadro; contar nós avaliados com e sem alfa-beta para evidenciar o ganho.

**Parte VI — Aprendizado e adaptação (RL e algoritmos genéticos)**

- *Conceitos essenciais:* agente–ambiente–recompensa; MDP; política e função valor; exploração × explotação; Q-Learning; população, aptidão, seleção, crossover, mutação, elitismo.
- *Dificuldades comuns:* projetar a função de recompensa/aptidão; aceitar a imprevisibilidade e o custo de treino.
- *Erros conceituais recorrentes:* superestimar o uso comercial de RL; confundir treino (offline, caro) com execução (em jogo); tratar o AG como se garantisse o ótimo.
- *Demonstração sugerida:* um exemplo tabular de Q-Learning (grade simples) e um AG evoluindo um parâmetro com o fitness subindo por geração; se possível, um ambiente do ML-Agents.

**Parte VII — Engenharia reversa e estudos de caso**

- *Conceitos essenciais:* observação sistemática; estímulo–resposta; sinais de cada técnica; distinção entre fato e inferência; ética.
- *Dificuldades comuns:* formular hipóteses testáveis; não "ver" a técnica favorita em todo lugar (viés de confirmação).
- *Erros conceituais recorrentes:* afirmar como fato o que é inferência; concluir a partir de uma única observação.
- *Demonstração sugerida:* analisar em conjunto um jogo simples e preencher a tabela "comportamento observado → hipótese → sinal-chave" (Cap. 15) ao vivo.

**Capítulos que exigem maior atenção:** Cap. 6 (arquiteturas de decisão e a distinção árvore de decisão × comportamento), Cap. 8 (formalização do A\*, admissibilidade e heurísticas) e Cap. 12 (RL, o tema com maior distância entre teoria acadêmica e uso comercial). Reserve tempo extra e mais demonstrações nesses três.

---

# Apêndice D — Referências e Leituras Recomendadas

Este apêndice unifica toda a bibliografia da apostila (D.1), organiza sugestões de aprofundamento por tema (D.2) e fecha com os **índices da obra** — figuras, diagramas e tabelas (D.3). As entradas foram padronizadas e as duplicações eliminadas.

## D.1 Referências bibliográficas consolidadas

Bibliografia única de toda a apostila, em ordem alfabética por sobrenome do primeiro autor e no padrão adotado ao longo dos capítulos.

**Livros-base do projeto**

- BONDY, John Adrian; MURTY, U. S. R. *Graph Theory*. New York: Springer, 2008. (Graduate Texts in Mathematics, v. 244.)
- BOURG, David M.; SEEMANN, Glenn. *AI for Game Developers: Creating Intelligent Behavior in Games*. Sebastopol, CA: O'Reilly Media, 2004.
- CORMEN, Thomas H.; LEISERSON, Charles E.; RIVEST, Ronald L.; STEIN, Clifford. *Algoritmos: Teoria e Prática*. 3. ed. Rio de Janeiro: Elsevier, 2012.
- MILLINGTON, Ian. *AI for Games*. 3. ed. Boca Raton: CRC Press, 2019.
- RABIN, Steve (org.). *Game AI Pro 3: Collected Wisdom of Game AI Professionals*. Boca Raton: A K Peters/CRC Press, 2017.
- RUSSELL, Stuart J.; NORVIG, Peter. *Inteligência Artificial*. 3. ed. Rio de Janeiro: Elsevier, 2013.

**Livros e coletâneas complementares**

- BELLMAN, Richard. *Dynamic Programming*. Princeton: Princeton University Press, 1957.
- GOLDBERG, David E. *Genetic Algorithms in Search, Optimization, and Machine Learning*. Boston: Addison-Wesley, 1989.
- HOLLAND, John H. *Adaptation in Natural and Artificial Systems*. Ann Arbor: University of Michigan Press, 1975.
- HOPCROFT, John E.; MOTWANI, Rajeev; ULLMAN, Jeffrey D. *Introduction to Automata Theory, Languages, and Computation*. Boston: Addison-Wesley.
- RABIN, Steve (org.). *Game AI Pro: Collected Wisdom of Game AI Professionals*. Boca Raton: A K Peters/CRC Press, 2013.
- RABIN, Steve (org.). *Game AI Pro 2*. Boca Raton: A K Peters/CRC Press, 2015.
- SHAKER, Noor; TOGELIUS, Julian; NELSON, Mark J. *Procedural Content Generation in Games*. Cham: Springer, 2016.
- SUTTON, Richard S.; BARTO, Andrew G. *Reinforcement Learning: An Introduction*. 2. ed. Cambridge: MIT Press, 2018.
- YANNAKAKIS, Georgios N.; TOGELIUS, Julian. *Artificial Intelligence and Games*. Cham: Springer, 2018.

**Artigos científicos e palestras (GDC)**

- BOTEA, Adi; MÜLLER, Martin; SCHAEFFER, Jonathan. Near Optimal Hierarchical Path-Finding (HPA\*). *Journal of Game Development*, v. 1, n. 1, 2004.
- BROWNE, Cameron et al. A Survey of Monte Carlo Tree Search Methods. *IEEE Transactions on Computational Intelligence and AI in Games*, v. 4, n. 1, 2012.
- CAMPBELL, Murray; HOANE, A. Joseph; HSU, Feng-hsiung. Deep Blue. *Artificial Intelligence*, v. 134, n. 1-2, p. 57-83, 2002.
- DIJKSTRA, Edsger W. A Note on Two Problems in Connexion with Graphs. *Numerische Mathematik*, v. 1, p. 269-271, 1959.
- HARABOR, Daniel; GRASTIEN, Alban. Online Graph Pruning for Pathfinding on Grid Maps. In: *Proceedings of the AAAI Conference on Artificial Intelligence*, 2011.
- HAREL, David. Statecharts: A Visual Formalism for Complex Systems. *Science of Computer Programming*, v. 8, n. 3, p. 231-274, 1987.
- HART, Peter E.; NILSSON, Nils J.; RAPHAEL, Bertram. A Formal Basis for the Heuristic Determination of Minimum Cost Paths. *IEEE Transactions on Systems Science and Cybernetics*, v. 4, n. 2, p. 100-107, 1968.
- ISLA, Damian. Handling Complexity in the Halo 2 AI. In: *Game Developers Conference (GDC)*, 2005.
- JULIANI, Arthur et al. Unity: A General Platform for Intelligent Agents. *arXiv:1809.02627*, 2018.
- MNIH, Volodymyr et al. Human-level control through deep reinforcement learning. *Nature*, v. 518, n. 7540, p. 529-533, 2015.
- ORKIN, Jeff. Three States and a Plan: The A.I. of F.E.A.R. In: *Game Developers Conference (GDC)*, 2006.
- QUINLAN, J. Ross. Induction of Decision Trees. *Machine Learning*, v. 1, n. 1, p. 81-106, 1986.
- RABIN, Steve; STURTEVANT, Nathan R. JPS+: An Extreme A\* Speed Optimization for Static Uniform Cost Grids. In: RABIN, Steve (org.). *Game AI Pro 2*. Boca Raton: A K Peters/CRC Press, 2015.
- SCHAEFFER, Jonathan et al. Checkers is Solved. *Science*, v. 317, n. 5844, p. 1518-1522, 2007.
- SHANNON, Claude E. Programming a Computer for Playing Chess. *Philosophical Magazine*, v. 41, n. 314, 1950.
- SILVER, David et al. Mastering the game of Go with deep neural networks and tree search. *Nature*, v. 529, p. 484-489, 2016.
- SILVER, David et al. A general reinforcement learning algorithm that masters chess, shogi, and Go through self-play. *Science*, v. 362, n. 6419, p. 1140-1144, 2018.
- STANLEY, Kenneth O.; MIIKKULAINEN, Risto. Evolving Neural Networks through Augmenting Topologies (NEAT). *Evolutionary Computation*, v. 10, n. 2, p. 99-127, 2002.
- STANLEY, Kenneth O.; BRYANT, Bobby D.; MIIKKULAINEN, Risto. Real-Time Neuroevolution in the NERO Video Game. *IEEE Transactions on Evolutionary Computation*, v. 9, n. 6, p. 653-668, 2005.
- TOGELIUS, Julian et al. Search-Based Procedural Content Generation: A Taxonomy and Survey. *IEEE Transactions on Computational Intelligence and AI in Games*, v. 3, n. 3, p. 172-186, 2011.
- VINYALS, Oriol et al. Grandmaster level in StarCraft II using multi-agent reinforcement learning. *Nature*, v. 575, p. 350-354, 2019.
- WATKINS, Christopher J. C. H.; DAYAN, Peter. Q-learning. *Machine Learning*, v. 8, n. 3-4, p. 279-292, 1992.

**Documentação oficial e ferramentas**

- EPIC GAMES. *Unreal Engine Documentation: Behavior Trees, Blackboard, State Tree, Navigation e Environment Query System (EQS), Learning Agents.* Disponível na documentação oficial da Unreal Engine.
- GRANBERG, Aron. *A\* Pathfinding Project — Documentação.*
- MONONEN, Mikko. *Recast & Detour: Navigation Mesh Toolset.* Documentação do projeto de código aberto.
- UNITY TECHNOLOGIES. *Documentação oficial da Unity: AI Navigation (NavMesh), Animator Controller e sub-state machines, Visual Scripting (State Graphs), pacote Unity Behavior e ML-Agents.* Disponível em: docs.unity3d.com.

## D.2 Leituras complementares por tema

Sugestões de aprofundamento, organizadas por tema e priorizando os materiais usados na construção da apostila. Endereços de sites externos podem mudar; use-os como ponto de partida.

**Fundamentos e agentes (Caps. 1–2).** RUSSELL & NORVIG, *Inteligência Artificial* (cap. sobre agentes racionais); MILLINGTON, *AI for Games* (cap. introdutório); YANNAKAKIS & TOGELIUS, *Artificial Intelligence and Games* (visão contemporânea e metodológica).

**Decisão baseada em regras (Caps. 3–6).** MILLINGTON (máquinas de estado, árvores de decisão e de comportamento); HAREL (1987) para a base formal de hierarquia e histórico; ISLA (GDC 2005) para árvores de comportamento em Halo 2; ORKIN (GDC 2006) para GOAP em F.E.A.R.; *Game AI Pro 3* para boas práticas de implementação.

**Movimento e pathfinding (Caps. 7–9).** HART, NILSSON & RAPHAEL (1968) para o A\* original; DIJKSTRA (1959); CORMEN et al. para os fundamentos algorítmicos; BONDY & MURTY para teoria dos grafos; HARABOR & GRASTIEN (2011) e RABIN & STURTEVANT (2015) para JPS/JPS+; BOTEA et al. (2004) para pathfinding hierárquico.

**Raciocínio espacial e tático (Cap. 10).** MILLINGTON (mapas de influência e táticas espaciais); *Game AI Pro 3* (artigos sobre influence maps e análise tática); documentação do EQS da Unreal para comparação.

**IA adversarial (Cap. 11).** SHANNON (1950); CAMPBELL, HOANE & HSU (2002, Deep Blue); SCHAEFFER et al. (2007, damas resolvidas); BROWNE et al. (2012, survey de MCTS); SILVER et al. (2016, 2018, AlphaGo/AlphaZero).

**Aprendizado e adaptação (Caps. 12–13).** SUTTON & BARTO, *Reinforcement Learning* (referência canônica); WATKINS & DAYAN (1992, Q-Learning); MNIH et al. (2015, Deep RL); VINYALS et al. (2019, StarCraft II); HOLLAND (1975) e GOLDBERG (1989) para algoritmos genéticos; STANLEY & MIIKKULAINEN (NEAT) e NERO para neuroevolução em jogos; SHAKER, TOGELIUS & NELSON e TOGELIUS et al. para geração procedural.

**Engenharia reversa e estudos de caso (Caps. 14–15).** YANNAKAKIS & TOGELIUS (discussão metodológica); ISLA e ORKIN (fontes documentais de Halo e F.E.A.R.); *Game AI Pro 3* (relatos de profissionais que sustentam as inferências dos estudos de caso).

**Cursos, documentações e canais técnicos (recursos online).**

- **Unity Learn** (`learn.unity.com`) — cursos oficiais; ótimo para praticar NavMesh, Animator e ML-Agents.
- **ML-Agents** (repositório e documentação oficiais) — ambientes de exemplo para RL.
- **Game AI Pro** (`gameaipro.com`) — capítulos das coletâneas disponibilizados gratuitamente pelos autores.
- **GDC Vault** — palestras técnicas de estúdios (incluindo as de Isla e Orkin citadas).
- **Red Blob Games** (`redblobgames.com`) — tutoriais interativos clássicos sobre A\*, heurísticas e grids.
- **Canais técnicos de referência** — séries de vídeo bem estabelecidas sobre pathfinding A\* e IA de jogos (por exemplo, os tutoriais de A\* de Sebastian Lague), úteis como apoio visual às demonstrações do professor.

> **Atenção**
> Materiais externos (vídeos, blogs, wikis) são excelentes para intuição e prática, mas variam em rigor. Para definições e afirmações técnicas, prefira sempre as fontes-base do projeto e a documentação oficial.

## D.3 Índices da obra

Índices consolidados dos recursos visuais e das tabelas da apostila. A numeração segue o capítulo e a ordem de aparição. Todas as imagens estão marcadas no texto como `[IMAGEM NECESSÁRIA]` (a serem produzidas), e os diagramas descritos em quadros `[DIAGRAMA]`.

### D.3.1 Índice de figuras

| Nº | Título | Cap. |
|---|---|---|
| 1.1 | Os múltiplos papéis da IA no game design | 1 |
| 1.2 | Mapa geral das famílias de técnicas da apostila | 1 |
| 2.1 | As quatro regras de mira dos fantasmas de Pac-Man | 2 |
| 2.2 | Linha do tempo da IA em jogos | 2 |
| 3.1 | Máquina de estados do inimigo guarda | 3 |
| 4.1 | HFSM de combate em camadas do inimigo | 4 |
| 6.1 | Árvore de comportamento completa do inimigo guarda | 6 |
| 7.1 | NavMesh gerada sobre um cenário 3D no editor da Unity | 7 |
| 9.1 | Gráfico comparativo de desempenho — A\* × JPS × JPS+ | 9 |
| 10.1 | Duas perguntas espaciais — "como chegar?" versus "onde ir?" | 10 |
| 10.2 | Mapa de calor de influência sobreposto a um mapa de RTS | 10 |
| 11.1 | O efeito horizonte no xadrez | 11 |
| 15.1 | Estados e alvos dos fantasmas de Pac-Man | 15 |
| 15.2 | Prioridades de comportamento de um inimigo de Halo | 15 |
| 15.3 | Percepção do Xenomorfo × posição real do jogador | 15 |

### D.3.2 Índice de diagramas

| Nº | Título | Cap. | Finalidade didática |
|---|---|---|---|
| 1.1 | A ilusão de inteligência — mecanismo simples, percepção rica | 1 | Mostrar como regras simples geram percepção de inteligência |
| 1.2 | O canal de fluxo e o ponto ideal de dificuldade | 1 | Relacionar desafio e habilidade ao engajamento |
| 1.3 | O ciclo Sentir → Pensar → Agir de um agente de jogo | 1 | Apresentar o laço fundamental do agente |
| 2.1 | A coevolução de hardware e técnicas de IA de jogos | 2 | Ligar avanços de hardware às técnicas dominantes |
| 3.1 | Anatomia de uma máquina de estados finita | 3 | Definir estados, transições, eventos e guardas |
| 3.2 | Polling versus eventos na avaliação de transições | 3 | Contrastar as duas formas de disparar transições |
| 3.3 | A explosão de transições | 3 | Ilustrar a principal limitação da FSM |
| 4.1 | Anatomia de uma máquina de estados hierárquica | 4 | Mostrar superestados, subestados e histórico |
| 5.1 | Estrutura de uma árvore de decisão | 5 | Apresentar nós de decisão, ramos e folhas |
| 6.1 | Os três nós compostos fundamentais | 6 | Explicar sequência, seletor e paralelo |
| 6.2 | Árvore de comportamento e blackboard | 6 | Relacionar fluxo de execução e memória compartilhada |
| 6.3 | Planejamento GOAP — do objetivo ao plano | 6 | Mostrar encadeamento de ações por pré-condições/efeitos |
| 6.4 | Decisão por utilidade — pontuar e escolher | 6 | Ilustrar pontuação por curvas de utilidade |
| 7.1 | Anatomia de um grafo de navegação | 7 | Definir vértices, arestas e pesos no espaço |
| 7.2 | Grade de navegação e sua conversão em grafo | 7 | Mostrar a grade como grafo implícito |
| 7.3 | Grafo de waypoints sobre um cenário | 7 | Representar navegação esparsa por pontos de rota |
| 7.4 | Malha de navegação (NavMesh) e seu grafo de polígonos | 7 | Mostrar a NavMesh como grafo de polígonos |
| 8.1 | A função de avaliação f = g + h | 8 | Decompor a função de avaliação do A\* |
| 8.2 | O ciclo de expansão do A\* com listas aberta e fechada | 8 | Detalhar o funcionamento passo a passo |
| 8.3 | Comparação de heurísticas em uma grade | 8 | Contrastar Manhattan, Euclidiana e Chebyshev |
| 8.4 | Traço de execução do A\* contornando uma parede | 8 | Exemplificar a busca em um obstáculo |
| 9.1 | Vizinhos naturais, podados e forçados no JPS | 9 | Explicar a poda de simetria do JPS |
| 9.2 | Nós expandidos — A\* versus JPS numa grade aberta | 9 | Evidenciar o ganho de desempenho |
| 9.3 | Suavização de caminho por linha de visão | 9 | Mostrar o pós-processamento do caminho |
| 10.1 | Fonte, propagação e decaimento em uma grade | 10 | Explicar a construção de um mapa de influência |
| 10.2 | Combinação de camadas de influência | 10 | Mostrar mapas derivados por soma de camadas |
| 11.1 | Três tipos de ambiente para a IA de jogo | 11 | Situar o ambiente adversarial |
| 11.2 | Anatomia de uma árvore de jogo | 11 | Definir estados, jogadas e camadas MAX/MIN |
| 11.3 | Propagação de valores no Minimax (árvore de 2 plies) | 11 | Demonstrar a propagação de utilidades |
| 11.4 | Como a função de avaliação preenche o horizonte | 11 | Ligar avaliação heurística ao limite de busca |
| 11.5 | Minimax completo versus alfa-beta (árvore podada) | 11 | Visualizar o efeito da poda alfa-beta |
| 11.6 | As quatro fases do MCTS | 11 | Apresentar seleção, expansão, simulação e retropropagação |
| 12.1 | Vocabulário fundamental da Aprendizagem por Reforço | 12 | Fixar agente, ambiente, estado, ação e recompensa |
| 12.2 | O ciclo de aprendizagem por reforço | 12 | Mostrar o laço de interação e recompensa |
| 12.3 | Estrutura de um Processo de Decisão de Markov (MDP) | 12 | Formalizar o problema de decisão sequencial |
| 12.4 | Atualização de uma célula da tabela Q | 12 | Detalhar a atualização do Q-Learning |
| 13.1 | Anatomia de uma população genética | 13 | Definir população, cromossomo, gene e aptidão |
| 13.2 | O ciclo completo do Algoritmo Genético | 13 | Apresentar o laço evolutivo |
| 13.3 | Os operadores genéticos em ação | 13 | Ilustrar seleção, crossover, mutação e elitismo |
| 14.1 | O problema da caixa-preta na IA de jogos | 14 | Enquadrar o desafio da engenharia reversa |
| 14.2 | Fluxo da observação sistemática | 14 | Apresentar o método de observação |
| 14.3 | Árvore de decisão para identificação de técnicas | 14 | Guiar a inferência da técnica provável |
| 14.4 | Roteiro reutilizável de engenharia reversa (seis etapas) | 14 | Consolidar o roteiro de análise |
| — | Estrutura do relatório final do Projeto Integrador | Encerr. VII | Padronizar o produto final e separar fato, hipótese e indeterminação |

### D.3.3 Índice de tabelas

| Título | Cap. | Finalidade |
|---|---|---|
| IA acadêmica × IA de jogos | 1 | Contrastar os dois campos e seus critérios |
| IA forte × IA fraca | 1 | Situar a "ilusão de inteligência" filosoficamente |
| Tipos de agente (reativo, deliberativo, híbrido) | 1 | Comparar formas de decidir e suas trocas |
| Famílias de técnicas por Parte | 1 | Mapear a apostila às perguntas que cada família responde |
| Marcos cronológicos da IA em jogos | 2 | Reunir hardware, técnica e jogos por período |
| Estados do inimigo guarda (enter/update/transições) | 3 | Especificar a FSM do exemplo condutor |
| Hierarquia completa da HFSM do inimigo | 4 | Organizar níveis, superestados e transições |
| Árvore de decisão × árvore de comportamento | 5 | Fixar a distinção conceitual essencial |
| Seis arquiteturas de decisão comparadas | 6 | Consolidar FSM, HFSM, decisão, comportamento, GOAP e utilidade |
| Jogos × técnica × nível de confiança | 15 | Resumir os estudos de caso e a confiabilidade da análise |
| Comportamento observado → hipótese → sinal-chave | 15 | Tabela operacional de engenharia reversa |
| Tipos de evidência na análise de IA | 15 | Distinguir o que cada evidência revela e não revela |

---

> **Encerramento dos apêndices.** Estes quatro apêndices fecham a apostila como material de consulta permanente. Para qualquer aprofundamento, retorne ao capítulo indicado em cada verbete, tabela ou índice — os apêndices apontam o caminho; os capítulos trazem a explicação completa.
