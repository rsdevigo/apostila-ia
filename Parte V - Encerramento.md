# Encerramento da Parte V

## Resumo Geral da Parte V

A Parte V introduziu uma ruptura conceitual em relação a tudo o que a viera antes. Nas Partes I a IV, o agente de IA operava em um mundo que podia ser complexo, dinâmico e até hostil, mas que **não raciocinava contra ele**: a máquina de estados reagia a estímulos, o pathfinding traçava rotas sobre um mapa passivo, o mapa de influência avaliava um terreno que não replanejava para anular essa avaliação. A Parte V colocou, pela primeira vez, um **oponente racional** do outro lado — uma entidade que persegue exatamente o objetivo oposto e que, a cada jogada, escolhe deliberadamente a resposta que mais nos prejudica. Essa mudança — da decisão em um mundo neutro para a decisão contra uma mente antagônica — é o território da **busca adversarial**.

O **Capítulo 11** desenvolveu esse território a partir do **problema**: por que jogar bem contra um oponente racional exige antecipar sua melhor resposta, e não apenas maximizar o próprio ganho imediato. Distinguiu os ambientes **cooperativo**, **reativo** e **competitivo**, situando o Minimax como ferramenta específica — e apenas — do último. Construiu os **fundamentos** (soma zero, turnos, informação perfeita, árvore de jogo, e seus elementos: estados, ações, profundidade, fator de ramificação, utilidade e função de avaliação) e, sobre eles, o algoritmo **Minimax**: os jogadores **MAX** e **MIN**, a propagação alternada de valores das folhas à raiz, a escolha da jogada com o **melhor pior caso**. Aprofundou as **camadas MAX/MIN**, a **profundidade** limitada, o **horizonte** de busca e o **efeito horizonte**, e o papel das **funções heurísticas de avaliação** — a mesma ideia de heurística que atravessa a apostila (A\*, mapas de influência), agora decidindo *quão boa é uma posição de jogo*.

Em seguida, a **poda alfa-beta** foi apresentada como a otimização **exata** e indispensável do Minimax: com os limites **α** e **β**, ela descarta ramos comprovadamente irrelevantes **sem alterar a decisão final**, reduzindo a complexidade de O(b^d) para O(b^{d/2}) no melhor caso — o que, na prática, **dobra a profundidade** alcançável. A **ordenação de jogadas** revelou-se o multiplicador dessa economia. Como conteúdo de **aprofundamento**, o **MCTS** mostrou uma filosofia alternativa — estimar valores por **simulação estatística** em vez de heurística —, que, dispensando função de avaliação e absorvendo fatores de ramificação enormes, conquistou o **Go** e inaugurou, com AlphaGo/AlphaZero, a era da busca combinada com aprendizado. Os **exemplos** (jogo da velha, damas, xadrez) e os **estudos de caso** (Deep Blue, Chinook) aterrissaram a teoria na história real, sempre separando fato documentado de análise fundamentada.

A síntese da Parte é uma tese em duas partes. Primeiro: **decidir contra um adversário é decidir contra a sua melhor resposta** — nunca supor que o oponente errará é o que dá solidez à jogada escolhida. Segundo: **toda busca adversarial prática é um exercício de trade-offs contra a explosão combinatória** — profundidade contra tempo, precisão da heurística contra custo por avaliação, exatidão do alfa-beta contra a estatística do MCTS —, e dominar a Parte V é dominar a arte de escolher, para cada tipo de jogo, o ponto certo nesse espaço de compromissos.

## Principais Conceitos Aprendidos

- **Busca adversarial** — decidir ações em um ambiente com um oponente de objetivos opostos que antecipa nossas jogadas; distinta da busca cooperativa e da reativa.
- **Ambientes cooperativo, reativo e competitivo** — objetivos alinhados (coordenação), mundo hostil mas não estratégico (regras reativas) e oponente antagônico que planeja contra nós (busca adversarial); só o último exige Minimax.
- **Jogo de soma zero** — o ganho de um é exatamente a perda do outro; autoriza uma única função de valor e dois objetivos opostos sobre ela.
- **Árvore de jogo** — representação hierárquica de todas as sequências de jogadas; raiz = estado atual, ramos = ações, folhas = fins de jogo; o objeto sobre o qual o Minimax opera.
- **Estado, ação, profundidade, fator de ramificação** — a configuração do jogo, a jogada legal, a distância em plies à raiz e o número médio de jogadas por estado (o motor da explosão *b^d*).
- **Utilidade × função de avaliação** — valor *exato* de um estado terminal versus *estimativa* de um estado não-terminal; a distinção que separa o Minimax teórico do prático.
- **Minimax** — propagação alternada de valores (máximo nos nós MAX, mínimo nos MIN) das folhas à raiz; escolhe a jogada com o melhor resultado garantido no pior caso.
- **Jogadores MAX e MIN e camadas** — os dois papéis opostos que se alternam por nível de profundidade; marcar quem é quem antes de propagar evita a maioria dos erros.
- **Profundidade e horizonte de busca** — o limite até onde a busca enxerga; o **efeito horizonte** é a ilusão de empurrar um evento ruim para além desse limite, mitigado pela busca de quietude.
- **Função heurística de avaliação** — soma ponderada de características da posição (material, posição...); sua qualidade influencia o desempenho tanto quanto o algoritmo.
- **Poda alfa-beta** — otimização *exata* que descarta ramos irrelevantes usando os limites α (melhor garantido a MAX) e β (melhor garantido a MIN); poda quando α ≥ β.
- **Corretude e complexidade da poda** — não altera a jogada do Minimax; reduz de O(b^d) para O(b^{d/2}) no melhor caso, dobrando a profundidade viável.
- **Ordenação de jogadas** — examinar primeiro as jogadas promissoras para provocar cortes cedo; irrelevante no Minimax puro, decisiva na alfa-beta.
- **MCTS (aprofundamento)** — seleção, expansão, simulação e retropropagação; estima valores por amostragem estatística, dispensa heurística e absorve ramificação enorme (Go, AlphaGo/AlphaZero).
- **Limites de viabilidade do Minimax** — falha com ramificação enorme, sem boa heurística, em tempo real, com acaso/informação oculta ou com mais de dois jogadores.

## Tabela Comparativa — Minimax × Alfa-Beta × MCTS

A tabela consolida as três abordagens da Parte segundo os critérios pedidos. É a principal ferramenta de revisão da Parte V e o instrumento para escolher a técnica certa para cada jogo.

| Critério | **Minimax (puro)** | **Minimax + Poda Alfa-Beta** | **Monte Carlo Tree Search (MCTS)** |
|---|---|---|---|
| **Objetivo** | Calcular a jogada ótima contra um oponente perfeito | O mesmo, sem trabalho desperdiçado | Escolher uma jogada forte por amostragem estatística de desfechos |
| **Tipo de busca** | Exaustiva, em profundidade, até um limite fixo | Exaustiva com poda de ramos irrelevantes (exata) | Seletiva e incremental; árvore assimétrica guiada por estatística |
| **Como avalia posições** | Função de avaliação heurística (nós de fronteira) | Idem Minimax | Simulações aleatórias até o fim (rollouts); sem heurística obrigatória |
| **Custo computacional** | Alto — O(b^d) | Menor — O(b^{d/2}) no melhor caso (≈ dobra a profundidade) | Ajustável — proporcional ao nº de iterações; *anytime* |
| **Consumo de memória** | Linear na profundidade, O(d) (busca em profundidade) | Idem, O(d) (+ tabelas de transposição opcionais) | Cresce com a árvore construída; guarda estatísticas por nó |
| **Dependência da heurística** | **Crítica** — joga tão bem quanto a função de avaliação | **Crítica** — igual ao Minimax | **Baixa** no clássico (rollouts aleatórios); alta se guiado por rede neural |
| **Escalabilidade (ramificação)** | Ruim — *b* alto inviabiliza a busca | Melhor, mas ainda exponencial; sofre com *b* muito alto | Excelente — lida com *b* enorme (Go, *b* ≈ 250) |
| **Aplicações típicas** | Didática; jogos pequenos (jogo da velha) | Xadrez, damas, Othello, Connect Four, estratégia por turnos | Go, jogos gerais, jogos sem boa heurística; base de AlphaGo/AlphaZero |
| **Vantagens** | Ótimo e correto; simples; transparente | Ótimo e correto **e** eficiente; padrão da indústria de tabuleiro | Dispensa heurística; escala à ramificação; *anytime*; casa com ML |
| **Limitações** | Explosão combinatória; efeito horizonte; caro | Ainda exponencial; depende de boa ordenação e heurística | Estatístico (não exato); fraco em táticas agudas; muitas iterações |

**Análise interpretativa.** Lida da esquerda para a direita, a tabela conta a evolução da IA competitiva. O **Minimax puro** é o alicerce conceitual — correto, ótimo e transparente, mas caro demais para uso direto além de jogos triviais; seu valor é sobretudo pedagógico e como fundamento teórico. O **alfa-beta** é o Minimax **utilizável**: preserva integralmente a corretude e a otimalidade, mas paga muito menos por elas, dobrando a profundidade alcançável — por isso é o **padrão da indústria** em jogos de tabuleiro com boa função de avaliação (xadrez, damas, Othello). Não é uma técnica "diferente" do Minimax, e sim o Minimax **sem desperdício**: essa é a leitura correta das duas primeiras colunas. O **MCTS**, por sua vez, não é um "alfa-beta melhor" — é uma filosofia distinta, que troca a *exatidão* do cálculo pela *estatística* da amostragem. Essa troca é perdedora onde há boa heurística e ramificação moderada (por isso o xadrez de ponta continua com alfa-beta), mas **vencedora** onde a heurística inexiste ou a ramificação explode (por isso o Go caiu para o MCTS). A lição central, coerente com toda a apostila, é que **não há técnica universalmente superior**: a escolha depende do fator de ramificação, da existência de uma boa função de avaliação, do orçamento de tempo e da natureza do jogo — e reconhecer, para cada caso, qual coluna da tabela se aplica é a competência que a Parte V forma.

## Questões de Revisão

1. Em uma frase, o que distingue a busca adversarial de todas as técnicas das Partes I a IV? Por que essa distinção é essencial?
2. Por que o alfa-beta e o Minimax puro sempre escolhem a **mesma** jogada? O que, então, a poda muda?
3. Explique a afirmação "uma heurística melhor permite jogar bem com menos profundidade". Como isso se relaciona com o efeito horizonte?
4. Por que o MCTS teve sucesso no Go onde o Minimax havia fracassado? Cite os dois obstáculos específicos do Go.
5. Dê um critério prático de decisão: diante de um novo jogo, que perguntas você faria para escolher entre alfa-beta e MCTS?

## Exercícios Conceituais

1. Um estudante afirma: "o Minimax escolhe sempre a jogada que leva ao maior valor possível na árvore". Corrija a afirmação e explique o erro conceitual por trás dela.
2. Explique por que a ordenação de jogadas não tem efeito sobre o Minimax puro, mas pode fazer a diferença entre O(b^d) e O(b^{d/2}) na alfa-beta.
3. Argumente a favor e contra a afirmação "o Deep Blue provou que máquinas podem pensar". Apresente as duas perspectivas de forma equilibrada, com base no que foi documentado sobre sua arquitetura.

## Exercícios de Integração da Parte V

Estes exercícios exigem análise, comparação e aplicação conceitual, relacionando os temas da Parte entre si e com o restante da apostila.

1. **Da função de avaliação ao mapa de influência.** A Subseção 11.3.4 afirma que a função de avaliação do Minimax é "a mesma ideia de heurística" dos mapas de influência (Capítulo 10) e do A\* (Capítulo 8). Escreva uma análise comparando os três usos de heurística: o que cada um estima, sobre qual estrutura opera (árvore de jogo, grade/campo, grafo) e por que, nos três casos, a heurística substitui um cálculo exato inviável. Aponte uma semelhança essencial e uma diferença essencial entre a "função de avaliação de posição de jogo" e o "valor tático de uma célula" de um mapa de influência.

2. **Escolhendo a arquitetura de IA para um jogo.** Você é o programador de IA de um estúdio e recebe três projetos: (a) um jogo de **xadrez** para celular com níveis de dificuldade; (b) um **RTS** com dezenas de unidades navegando e escolhendo posições; (c) um jogo de **tabuleiro abstrato original** com fator de ramificação estimado em ~200 e sem nenhuma heurística de avaliação conhecida. Para cada projeto, indique qual(is) família(s) de técnicas da apostila você usaria (FSM/BT da Parte II, pathfinding da Parte III, mapas de influência da Parte IV, Minimax/alfa-beta ou MCTS da Parte V) e justifique com base na natureza do problema (tempo real × turnos, ramificação, disponibilidade de heurística, adversarial × reativo).

3. **Profundidade, horizonte e orçamento.** Um jogo de damas por turnos precisa responder em no máximo 1 segundo por lance, em um hardware que avalia ~1 milhão de posições por segundo. Com *b* ≈ 8, estime a profundidade máxima que o Minimax puro alcança nesse orçamento e, depois, a que o alfa-beta (melhor caso) alcança. Discuta como o **efeito horizonte** poderia afetar as decisões na profundidade obtida e que técnica do Capítulo 11 você usaria para mitigá-lo. Relacione a discussão com o conceito de **orçamento de quadro** introduzido na Parte I.

4. **Adversarial × reativo na prática.** Escolha um jogo comercial de estratégia por turnos que você conheça (por exemplo, um da série *Civilization* ou um jogo de tabuleiro digital). Analisando seu comportamento observável, discuta quais decisões da IA **provavelmente** usam busca adversarial (Minimax/alfa-beta) e quais **provavelmente** usam técnicas reativas ou de avaliação tática (mapas de influência, heurísticas de regras). Seguindo a disciplina da Seção 11.8, distinga claramente o que você **infere** do que seria **fato documentado**, e explique em que sinais observáveis você baseia cada inferência — uma preparação direta para a engenharia reversa da Parte VII.

5. **Da busca clássica ao aprendizado.** A Parte V termina apontando para a Parte VI: AlphaZero substituiu a função de avaliação **programada** por uma **aprendida**, mantendo a busca (MCTS) por baixo. Escreva um parágrafo argumentando por que "algoritmo de busca" e "função de avaliação" são componentes **separáveis**, e como essa separação abre a porta para as técnicas de aprendizado que virão a seguir. Que vantagens e que riscos uma avaliação aprendida traz frente a uma programada à mão?

## Leituras Complementares

- **Russell, S.; Norvig, P.** *Inteligência Artificial*, 3ª ed. — capítulo de jogos: tratamento formal completo de Minimax, alfa-beta, Expectiminimax e busca com tempo limitado.
- **Millington, I.** *AI for Games*, 3ª ed. — capítulo de *Board Game AI*: a perspectiva de desenvolvimento, com MCTS, tabelas de transposição e ordenação de jogadas.
- **Rabin, S. (org.)** *Game AI Pro* — artigos práticos sobre MCTS e otimizações de busca.
- **Schaeffer, J.** *One Jump Ahead* — a história do Chinook e da resolução das damas.
- **Silver, D. et al.** artigos do AlphaGo (*Nature*, 2016) e AlphaZero (*Science*, 2018) — a fronteira da busca combinada com aprendizado profundo.

## Referências Bibliográficas

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
