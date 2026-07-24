# Encerramento da Parte II

## Resumo Geral da Parte II

A Parte II percorreu a primeira e mais fundamental família de técnicas da fase **Pensar** do ciclo do agente: a **tomada de decisão baseada em regras**. Seu fio condutor foi **evolutivo** — cada arquitetura nasceu para resolver uma limitação concreta da anterior —, e o percurso deixou claro que essas técnicas não são alternativas soltas, mas uma **linhagem de engenharia** movida por dores reais de produção.

O **Capítulo 3** apresentou a **máquina de estados finita (FSM)**, o alicerce de toda a IA de decisão. Vimos seus quatro conceitos elementares (estado, transição, evento, ação), sua raiz nos autômatos finitos — cuja natureza *finita e determinística* é a origem de suas virtudes de previsibilidade e controle —, seu funcionamento (polling × eventos; ciclo enter/update/exit) e, sobretudo, sua limitação estrutural: a **explosão de transições**, o crescimento quadrático que inviabiliza a FSM plana em escala.

O **Capítulo 4** respondeu a essa limitação com a **máquina de estados hierárquica (HFSM)**, que organiza estados relacionados sob **superestados** e permite, via **herança de transições**, escrever regras comuns uma única vez. Acrescentou o **estado de histórico** (memória de configuração) e mostrou como a hierarquia dá organização e manutenibilidade — sem, contudo, eliminar o acoplamento e a rigidez de fundo da família FSM.

O **Capítulo 5**, de **apoio**, apresentou a **árvore de decisão** como uma terceira via — classificar uma situação em uma ação por testes encadeados — e cumpriu seu papel de **ponte**, distinguindo com precisão a árvore de decisão (um *classificador* instantâneo) da árvore de comportamento (um *executor* temporal). Suas limitações — ausência de tempo, sequência, reutilização e composição — anteciparam exatamente o que a arquitetura seguinte viria oferecer.

O **Capítulo 6**, o de **ênfase**, apresentou a **árvore de comportamento (Behavior Tree)**, a síntese madura e o padrão de fato da indústria. Vimos como ela desacopla comportamento de controle de fluxo por meio de **nós compostos** (sequência, seletor, paralelo), **decoradores**, **folhas** e **blackboard**, e como o mecanismo de **tick** com os três **estados de retorno** (sucesso, falha e *em execução*) lhe confere a noção de duração e a reatividade que faltavam a tudo o que veio antes. Como **aprofundamentos**, conhecemos o **GOAP** (o agente *planeja* sua sequência de ações) e a **IA de utilidade** (o agente *pontua* opções por curvas), duas fronteiras modernas que complementam — sem substituir — o núcleo de árvores de comportamento.

A síntese da Parte é uma tese: **a decisão em jogos é um espectro de arquiteturas combináveis, escolhidas conforme o problema**, e não uma escada em que cada degrau torna obsoleto o anterior. Um jogo comercial típico combina várias delas — uma BT no comando, uma FSM animando por baixo, uma árvore de decisão numa folha, e, em casos sofisticados, GOAP ou utilidade num ramo específico.

## Principais Conceitos Aprendidos

- **Tomada de decisão baseada em regras** — família de técnicas que responde à pergunta "qual comportamento adotar agora?", preenchendo a fase *Pensar* do ciclo do agente.
- **Máquina de estados finita (FSM)** — estados mutuamente exclusivos conectados por transições disparadas por eventos; um modo ativo por vez.
- **Estado, transição, evento (guarda) e ação** — os quatro elementos elementares da FSM; a guarda é a condição booleana que dispara a transição.
- **Ciclo enter / update / exit** — divisão das ações de um estado em entrada (uma vez), permanência (a cada quadro) e saída (uma vez); resolve a memória de contexto.
- **Polling × eventos** — avaliar guardas continuamente (simples, custoso) versus reagir a notificações (eficiente, exige infraestrutura); impacta o orçamento de quadro.
- **Explosão de transições** — crescimento quadrático (ordem *n²*) das transições com o número de estados; o limite estrutural da FSM plana.
- **Máquina de estados hierárquica (HFSM)** — estados agrupados em **superestados** contendo submáquinas; combate a explosão de transições pela organização em níveis.
- **Herança de transições** — regras comuns escritas uma vez na borda do superestado, válidas para todos os subestados; elimina redundância.
- **Estado de histórico (raso/profundo)** — memória que faz o superestado retomar o subestado em que estava, preservando a continuidade do comportamento.
- **Árvore de decisão** — classificador que desce da raiz à folha por testes encadeados; decisão instantânea, sem estado persistente.
- **Árvore de decisão × árvore de comportamento** — mesma forma, propósitos distintos: classificar uma ação versus orquestrar comportamento ao longo do tempo.
- **Árvore de comportamento (Behavior Tree)** — arquitetura em que nós de controle de fluxo orquestram tarefas desacopladas; padrão dominante da indústria.
- **Nós compostos** — sequência (E; aborta na falha), seletor/fallback (OU e prioridade; aborta no sucesso), paralelo (concorrência por política).
- **Decoradores, folhas e blackboard** — modificadores de um filho; ações/condições que fazem o trabalho; memória compartilhada que desacopla os nós.
- **Tick e estados de retorno (sucesso/falha/em execução)** — travessia da árvore por quadro; o estado *em execução* confere a noção de duração.
- **Reatividade** — capacidade de abortar ramos em execução quando surge algo mais prioritário (reavaliação e decoradores de interrupção).
- **GOAP (aprofundamento)** — planejamento por objetivos: o agente monta a sequência de ações (pré-condições/efeitos) que atinge um objetivo, por busca.
- **IA de utilidade (aprofundamento)** — pontuação de opções por curvas de utilidade sobre considerações contínuas; escolha da ação de maior utilidade.
- **Espectro combinável** — as arquiteturas se acumulam e se combinam; a maturidade está em escolher e integrar, não em aplicar sempre a mais sofisticada.

## Tabela Comparativa Consolidada

A tabela a seguir consolida as **seis arquiteturas** da Parte II segundo os critérios pedidos: objetivo, estrutura, vantagens, limitações, escalabilidade, aplicações e complexidade. É a principal ferramenta de revisão da Parte.

| Critério | **FSM** | **HFSM** | **Árvore de Decisão** | **Árvore de Comportamento** | **GOAP** *(aprof.)* | **IA de Utilidade** *(aprof.)* |
|---|---|---|---|---|---|---|
| **Objetivo** | Alternar entre modos de comportamento | Organizar FSMs grandes em níveis | Escolher uma ação a partir de condições | Orquestrar comportamento sequenciado e reutilizável | Montar planos para atingir objetivos | Escolher a ação mais desejável por pontuação |
| **Estrutura** | Estados + transições (grafo) | Superestados aninhados com submáquinas | Nós de teste + ramos + folhas (árvore) | Compostos + decoradores + folhas + blackboard (árvore) | Ações (pré-condições/efeitos) + planejador (busca) | Considerações + curvas de utilidade + seleção |
| **Como decide** | Transita de estado sob guardas | Avaliação em cascata, do superestado ao subestado | Desce da raiz à folha classificando | Tick da raiz; sucesso/falha/em execução | Busca a sequência de menor custo até o objetivo | Pontua cada ação; escolhe a maior (ou sorteia) |
| **Vantagens** | Simples, barata, previsível, depurável | Reduz redundância; organiza; memória | Transparente, intuitiva, barata | Modular, reutilizável, legível, reativa, autoria visual | Adaptativa; improvisa; menos codificação manual | Lida com muitos fatores contínuos; "orgânica"; variável |
| **Limitações** | Explosão de transições; sem concorrência | Não elimina acoplamento/rigidez | Sem tempo, sequência, reutilização | Prescritiva; decisão booleana/ordinal | Cara; difícil de depurar/controlar | Difícil de prever/depurar; tunagem trabalhosa |
| **Escalabilidade** | Baixa (degrada rápido) | Média (hierarquia adia o problema) | Baixa–média (árvores profundas repetem) | Alta (modularidade contém a complexidade) | Média (custo de busca cresce com ações) | Média (muitas curvas ficam difíceis de balancear) |
| **Aplicações típicas** | Inimigos simples, unidades de RTS, animação | NPCs com combate em camadas; animação hierárquica | Decisões pontuais: reação, diálogo, alvo | Espinha dorsal de NPCs de médio/grande porte | Esquadrões táticos; agentes que "improvisam" | Simulação com necessidades; escolhas multifator |
| **Complexidade de implementação** | Muito baixa | Baixa–média | Muito baixa | Média | Alta | Média–alta |

**Análise interpretativa.** Lida da esquerda para a direita, a tabela reconta a **progressão da Parte**: da FSM (simples, mas de baixa escalabilidade) à HFSM (que adia o problema pela hierarquia), passando pela árvore de decisão (transparente, mas sem tempo nem reúso), até a árvore de comportamento — a única cuja **escalabilidade de manutenção é alta**, razão de seu domínio na indústria. As duas colunas de aprofundamento (GOAP e utilidade) trocam parte da **controlabilidade** por poder adaptativo, o que explica sua adoção mais seletiva: elas rendem comportamento impressionante, mas atritam com o critério de autoria da Parte I. Repare, por fim, que a linha "aplicações típicas" mostra que **as arquiteturas coexistem** — animação (FSM), decisão central (BT), microdecisões (árvore de decisão) e casos especiais (GOAP/utilidade) podem estar todos no *mesmo* agente. A escolha profissional não é "qual é a melhor?", mas "qual é a melhor *para cada parte* do problema, e como combiná-las?".

## Questões de Revisão

As questões a seguir cobrem os conceitos centrais da Parte II. Recomenda-se respondê-las por escrito, com as próprias palavras, antes de avançar para a Parte III.

1. Descreva a **progressão evolutiva** da Parte II (FSM → HFSM → árvore de decisão → árvore de comportamento → GOAP → utilidade). Para cada passo, indique *qual limitação da técnica anterior* motivou a seguinte.
2. Explique por que a natureza **finita e determinística** da FSM é, ao mesmo tempo, a fonte de suas virtudes (previsibilidade, controle) e de seus limites (rigidez).
3. O que é a **explosão de transições** e como a **HFSM** a combate? A hierarquia *elimina* ou apenas *administra* o problema? Justifique.
4. Distinga, com precisão técnica, **árvore de decisão** e **árvore de comportamento**, usando os conceitos de *estados de retorno* e *nós de controle de fluxo*.
5. Explique o papel do estado de retorno **em execução** nas árvores de comportamento. Que capacidade ele confere que nenhuma arquitetura anterior tinha?
6. Compare como a **prioridade entre comportamentos** é expressa em cada arquitetura: na FSM (transições), na HFSM (herança), na árvore de decisão (ordem dos testes) e na BT (ordem dos filhos do seletor). Qual é mais fácil de reordenar, e por quê?
7. *(Aprofundamento)* Em que sentido o **GOAP** inverte a lógica das arquiteturas prescritivas? O que significa dizer que montar um plano GOAP é um problema de **busca em grafo**?
8. *(Aprofundamento)* Por que a **IA de utilidade** é mais adequada que a árvore de comportamento para decisões com muitos fatores **contínuos e concorrentes**? Qual o preço pago em termos de depuração e controle?
9. Relacione cada arquitetura da Parte com o critério de **controle do designer/autoria** da Parte I. Por que a árvore de comportamento é considerada um bom equilíbrio entre poder e controle?
10. Explique o padrão "**BT decide, FSM anima**" e por que ele exemplifica a tese de que as técnicas *se acumulam em camadas, não se substituem*.

## Exercícios de Integração da Parte II

Os exercícios a seguir exigem **relacionar** os quatro capítulos entre si, não apenas recordar cada um isoladamente. São exercícios de aplicação e análise.

**Exercício 1 — A mesma IA, quatro arquiteturas.** Escolha um NPC de comportamento moderado (por exemplo, um guarda de patrulha com combate simples). Modele o *mesmo* comportamento como (a) FSM, (b) HFSM, (c) árvore de decisão e (d) árvore de comportamento — em diagramas ou descrições estruturadas. Depois, escreva um parágrafo comparando **quão fácil** foi expressar o comportamento em cada uma e **quão fácil** seria adicionar um novo comportamento (por exemplo, "pedir reforços") em cada versão.

**Exercício 2 — Diagnóstico de arquitetura.** Para cada situação abaixo, escolha a arquitetura mais adequada da Parte II e **justifique**, citando vantagens e limitações: (a) o controlador de animação de um personagem; (b) a decisão instantânea de qual linha de diálogo um NPC diz; (c) o comportamento completo de um chefe com muitas fases e táticas; (d) um esquadrão que deve improvisar planos de flanqueamento; (e) um personagem de simulação com necessidades (fome, sono, diversão).

**Exercício 3 — Refatorando uma FSM que explodiu.** Descreve-se um inimigo com 14 estados numa FSM plana, difícil de manter, em que a regra "se em chamas, rolar no chão" precisa aparecer em quase todos os estados. Proponha: (a) como uma **HFSM** reorganizaria esses 14 estados em superestados e onde a regra "se em chamas" seria ancorada; (b) como uma **árvore de comportamento** expressaria o mesmo, e onde a regra "se em chamas" entraria no seletor-raiz. Discuta qual solução você prefere e por quê.

**Exercício 4 — Distinguindo fato de análise.** Para os três estudos de caso do Capítulo 6 (*Halo 2*/BT, *F.E.A.R.*/GOAP, *The Sims*/utilidade), escreva, para cada um, uma frase do que é **documentado** pelos desenvolvedores e uma do que seria **análise técnica** sua. Reflita, em um parágrafo, sobre por que essa distinção é uma competência profissional (antecipando a Parte VII).

**Exercício 5 — Combinando arquiteturas.** Projete, em alto nível, a IA de um único inimigo "de elite" que **combine** pelo menos três arquiteturas da Parte II (por exemplo: BT para a decisão central, FSM para a animação, e utilidade para escolher entre variações de ataque). Descreva qual arquitetura cuida de qual parte, como elas se comunicam (por exemplo, via blackboard) e por que essa divisão é melhor do que usar uma única arquitetura para tudo.

**Exercício 6 — Da limitação à inovação (síntese histórica).** Escreva um texto curto (uma página) narrando a Parte II como uma **história de engenharia**: comece pela FSM e explique, passo a passo, que problema de produção levou à HFSM, à árvore de decisão, à árvore de comportamento e, por fim, às fronteiras de GOAP e utilidade. Conecte essa narrativa à lição da Parte I de que "as técnicas se acumulam em camadas, não se substituem em bloco".

## Leituras Complementares

- **MILLINGTON, I.** *AI for Games.* 3. ed. — a referência mais completa e acessível para toda a Parte II: máquinas de estado, hierarquias, árvores de decisão, árvores de comportamento, GOAP e IA de utilidade, com boa articulação entre teoria e prática de jogos. Leitura recomendada dos capítulos sobre *decision making*.
- **RABIN, S. (org.).** *Game AI Pro 3* (e a série *Game AI Pro* em geral) — coletâneas de artigos escritos por profissionais da indústria, com relatos concretos de implementação de árvores de comportamento, planejamento e utilidade em jogos reais. Excelente para conectar os conceitos da Parte ao mercado.
- **BOURG, D.; SEEMANN, G.** *AI for Game Developers.* — panorama introdutório e prático das técnicas de comportamento baseadas em estados, útil como reforço dos Capítulos 3 e 4.
- Palestras técnicas da **GDC (Game Developers Conference)** sobre a IA de *Halo 2* (árvores de comportamento) e *F.E.A.R.* (GOAP) — fontes primárias e documentadas dos estudos de caso desta Parte, a serem retomadas na Parte VII.
- **HAREL, D.** *Statecharts: A Visual Formalism for Complex Systems* (1987) — para o aluno que quiser a origem formal da hierarquia e do estado de histórico usados no Capítulo 4.
- Documentação oficial do pacote **Unity Behavior** e das **Behavior Trees da Unreal Engine** — para ver os conceitos do Capítulo 6 materializados em ferramentas profissionais.

> ✅ **Boa Prática**
> Ao consultar as leituras, mantenha em mente o **quadro comparativo consolidado** deste Encerramento. Cada fonte tende a apresentar as arquiteturas de forma isolada; usar a tabela como mapa ajuda a enxergá-las como uma linhagem conectada, em vez de uma lista de técnicas independentes. Pergunte-se sempre: *que problema esta técnica resolve que a anterior não resolvia, e a que custo?*

## Referências Bibliográficas da Parte II

BOURG, David M.; SEEMANN, Glenn. *AI for Game Developers: Creating Intelligent Behavior in Games.* Sebastopol, CA: O'Reilly Media, 2004.

HAREL, David. Statecharts: A Visual Formalism for Complex Systems. *Science of Computer Programming*, v. 8, n. 3, p. 231–274, 1987.

HOPCROFT, John E.; MOTWANI, Rajeev; ULLMAN, Jeffrey D. *Introduction to Automata Theory, Languages, and Computation.* Boston: Addison-Wesley.

ISLA, Damián. Handling Complexity in the Halo 2 AI. In: *Game Developers Conference (GDC)*, 2005.

MILLINGTON, Ian. *AI for Games.* 3. ed. Boca Raton: CRC Press, 2019.

ORKIN, Jeff. Three States and a Plan: The A.I. of F.E.A.R. In: *Game Developers Conference (GDC)*, 2006.

QUINLAN, J. Ross. Induction of Decision Trees. *Machine Learning*, v. 1, n. 1, p. 81–106, 1986.

RABIN, Steve (org.). *Game AI Pro 3: Collected Wisdom of Game AI Professionals.* Boca Raton: A K Peters/CRC Press, 2017.

RUSSELL, Stuart J.; NORVIG, Peter. *Inteligência Artificial.* 3. ed. Rio de Janeiro: Elsevier, 2013.

UNITY TECHNOLOGIES. *Documentação oficial da Unity: Animator Controller, Visual Scripting e pacote Unity Behavior.*

EPIC GAMES. *Documentação oficial da Unreal Engine: Behavior Trees, Blackboard e State Tree.*

> **Nota sobre fontes dos estudos de caso.** As descrições da IA de *Pac-Man*, *Halo 2*, *F.E.A.R.* e *The Sims* baseiam-se em documentação pública dos desenvolvedores (palestras e artigos técnicos) quando indicado como documentado, e em análise técnica fundamentada nos demais casos. As atribuições de FSM/HFSM/árvore de decisão a outros títulos comerciais constituem análise técnica fundamentada na estrutura observável do comportamento. As referências primárias completas dos estudos de caso serão consolidadas na Parte VII (Capítulos 14 e 15).

---

## Encerramento

Com a Parte II, o leitor domina a primeira grande família da fase *Pensar*: a **tomada de decisão baseada em regras**. Aprendeu não apenas cada arquitetura — FSM, HFSM, árvore de decisão, árvore de comportamento — e suas fronteiras modernas — GOAP e utilidade —, mas, sobretudo, a **raciocinar sobre trade-offs**: poder de expressão contra custo, sofisticação contra controle, elegância contra depurabilidade. Essa é a competência que distingue um profissional de IA de jogos de alguém que apenas conhece algoritmos.

A **Parte III — Movimento e Busca de Caminhos** muda o foco da pergunta "*qual* comportamento adotar?" para a pergunta "*como chegar* até onde quero agir?". Deixaremos o terreno das regras e entraremos no da **busca em grafos**: como o NPC representa o espaço navegável e como encontra caminhos nele, culminando no algoritmo A\*. As decisões desta Parte II continuarão presentes — afinal, é uma árvore de comportamento ou uma FSM que *decide* mover-se —, mas o *como* do movimento exigirá um novo ferramental, mais matemático, que começaremos a construir a seguir.
