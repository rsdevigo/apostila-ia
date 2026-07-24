# Capítulo 14 — Metodologia de Engenharia Reversa de IA

## Introdução

Todo o percurso desta apostila foi, em certo sentido, um treino para este capítulo. Aprendemos a construir inteligências: sabemos escrever uma máquina de estados, desenhar uma árvore de comportamento, calcular um caminho com A\*, propagar um mapa de influência, podar uma árvore de jogo, treinar um agente por reforço. Agora vamos aprender a **desconstruí-las** — não a partir do código, que raramente teremos, mas a partir da única coisa que sempre está disponível: **o comportamento que o jogo mostra na tela**.

A pergunta que organiza o capítulo é enganosamente simples: *diante de um NPC que se comporta de determinada maneira, como descobrir qual técnica de IA provavelmente o anima?* Responder a ela com seriedade exige mais do que intuição. Exige um **método** — uma forma disciplinada de observar, de registrar, de levantar hipóteses e de julgá-las. Este capítulo constrói esse método. Ele é, por natureza, diferente de todos os anteriores: não apresenta um algoritmo novo, e sim uma **competência investigativa** que recruta, ao mesmo tempo, todos os algoritmos já estudados. Por isso não há código aqui. O produto do capítulo é uma **disciplina de análise**, e o instrumento que a materializa é um **roteiro reutilizável** que você aplicará, no Capítulo 15, a nove jogos reais — e, depois, a qualquer jogo que despertar sua curiosidade.

Antes de começar, vale fixar o espírito da coisa com uma imagem. O engenheiro reverso de IA se parece menos com um programador e mais com um **naturalista** — alguém que observa um animal na natureza e, sem poder dissecá-lo, deduz seus hábitos, seus sentidos e sua estratégia de caça a partir de como ele se comporta. O naturalista não abre o bicho; ele o **observa com paciência, repete a observação, controla as condições e conclui com humildade**. É exatamente essa a postura que o capítulo quer formar.

---

## Contexto histórico

A engenharia reversa é tão antiga quanto a engenharia. Desmontar um mecanismo para entender como ele funciona — um relógio, um motor, um circuito — sempre foi uma forma legítima e fértil de aprender. No software, a prática ganhou contornos próprios: como o código-fonte costuma ser fechado, analistas passaram a estudar programas **de fora**, observando entradas, saídas e comportamento, sem acesso ao que estava escrito por dentro. Essa tradição — a análise **comportamental** de um sistema opaco, tratado como uma **caixa-preta** — é a raiz direta do que faremos com a IA de jogos.

No campo específico dos jogos, a história tem dois momentos. O primeiro é o da **comunidade de jogadores**. Muito antes de existir uma disciplina acadêmica, jogadores dedicados já faziam engenharia reversa da IA por conta própria, movidos pelo desejo de vencer. O exemplo canônico — que estudaremos em detalhe no Capítulo 15 — é o de **Pac-Man** (1980): ao longo de décadas, jogadores mapearam com precisão os padrões de perseguição de cada um dos quatro fantasmas, batizaram seus comportamentos e descobriram "rotas seguras" que exploravam a lógica determinística do jogo — tudo isso **sem jamais ver uma linha do código original**. Foi engenharia reversa comportamental pura, feita por observação sistemática e repetição exaustiva. Comunidades de *speedrun*, de *fighting games* (que estudam o comportamento da CPU quadro a quadro) e de RPGs (que datam tabelas de "IA de chefe") aperfeiçoaram essa arte por conta própria.

O segundo momento é o da **profissionalização e da abertura parcial**. A partir dos anos 2000, um movimento notável transformou a IA de jogos de segredo industrial em conhecimento compartilhado. A **Game Developers Conference (GDC)** tornou-se o palco onde desenvolvedores passaram a **descrever suas próprias arquiteturas** em palestras técnicas; surgiram **postmortems** detalhados (na revista *Game Developer* e no site Gamasutra, renomeado *Game Developer* em 2021), livros como a série ***Game AI Pro*** (organizada por Steve Rabin) e ***AI for Games*** (Ian Millington), e a comunidade acadêmica de IA para jogos se consolidou. Palestras como a de **Jeff Orkin** sobre o GOAP de *F.E.A.R.* (2006) e a de **Damian Isla** sobre a arquitetura de *Halo 2* (2005) tornaram-se referências fundadoras. Esse movimento não elimina a engenharia reversa — pelo contrário, dá a ela um **gabarito**: agora podemos **comparar** nossas hipóteses, levantadas por observação, com o que os próprios criadores revelaram. É precisamente essa combinação — observar de fora e, quando existir, confrontar com documentação oficial — que define a metodologia madura deste capítulo.

> 🕰️ **Contexto Histórico**
> A expressão "engenharia reversa" carrega, na indústria de software em geral, uma conotação às vezes controversa, ligada à quebra de proteções e à cópia de produtos. No contexto **acadêmico e didático** desta apostila, o sentido é outro e muito mais antigo: o de **estudar um sistema observável para compreendê-lo**, como um cientista estuda a natureza. Voltaremos a essa distinção, que é sobretudo ética, na seção 14.4.

---

## 14.1 O problema: como descobrir qual técnica um jogo utiliza

### Por que os desenvolvedores raramente publicam os detalhes da IA

Se todos os estúdios documentassem publicamente a IA de seus jogos, este capítulo seria desnecessário: bastaria consultar o manual. Não é o que acontece, e há razões concretas para isso.

A primeira razão é **comercial e competitiva**. A arquitetura de IA de um jogo pode ser um diferencial de mercado e o resultado de anos de investimento. Um estúdio que desenvolveu um sistema de comportamento inovador tem poucos incentivos para entregá-lo pronto aos concorrentes. Embora as **ideias** (FSM, GOAP, behavior trees) sejam de domínio público, os **detalhes de implementação**, os parâmetros ajustados a mão e os truques que fazem a diferença entre uma IA convincente e uma medíocre costumam permanecer fechados.

A segunda razão é a **proteção da ilusão**. Este é um ponto que atravessa toda a apostila e reaparece aqui com força total. A IA de jogos existe para **parecer inteligente**, não para ser inteligente no sentido acadêmico. Quando o público sabe exatamente como o "truque" funciona, a mágica corre o risco de se desfazer — como um número de ilusionismo explicado. Um estúdio pode preferir que os jogadores acreditem que os inimigos "pensam" e "se coordenam", em vez de revelar que, por trás disso, há uma máquina de estados e alguns gatilhos bem colocados. Manter a IA como uma caixa-preta é, muitas vezes, **preservar deliberadamente a experiência**.

A terceira razão é simplesmente **prática e cultural**. Documentar a fundo um sistema complexo dá trabalho, e nem toda equipe tem tempo ou incentivo para transformar seu código em uma palestra de GDC. Muitos sistemas de IA, além disso, são **emaranhados** — cresceram organicamente ao longo da produção, com camadas de gambiarras e casos especiais que ninguém descreveria com orgulho. O que existe de documentação, quando existe, tende a ser uma **reconstrução idealizada e simplificada** feita depois do lançamento, e não um retrato fiel do código que rodou.

Uma quarta razão, mais sutil, é que **frequentemente não há uma "técnica" única a revelar**. A IA de um jogo comercial raramente é "uma FSM" ou "uma behavior tree" pura. É quase sempre um **híbrido**: uma behavior tree que aciona um planejador, que consulta um mapa de influência, que pede uma rota a um A\*, tudo orquestrado por camadas de código específicas daquele jogo. Reduzir isso a um único nome seria falso. Parte do trabalho da engenharia reversa é justamente reconhecer que se observa um **sistema composto**, e identificar cada peça provável.

> 🎮 **Na Prática**
> Mesmo quando um jogo tem documentação oficial excelente — como *F.E.A.R.* ou *Left 4 Dead* —, ela descreve o sistema **do ponto de vista de quem o construiu**, com o vocabulário e a organização que fizeram sentido para aquela equipe. A engenharia reversa continua útil por um motivo pedagógico: ela treina você a **chegar sozinho** a hipóteses que depois pode confrontar com a fonte oficial. Acertar a hipótese antes de ler a palestra é o melhor exercício de aprendizado que esta Parte oferece.

### Por que a engenharia reversa se tornou uma habilidade importante

Se os detalhes são fechados, por que investir em aprender a analisá-los de fora? Porque essa habilidade é valiosa para três perfis profissionais distintos, e por razões diferentes em cada caso.

Para o **designer de jogos**, a engenharia reversa é uma forma de **estudar referências**. Quando um designer joga um título de sucesso e percebe que os inimigos "parecem inteligentes de um jeito especial", saber *por que* — que mecanismo produz aquela sensação — é o que permite reproduzir a qualidade, e não apenas copiar a aparência. Analisar como *Halo* faz seus inimigos recuarem, ou como *The Last of Us* coordena aliados, é aprender **princípios de design de comportamento** que se transferem para o próprio projeto.

Para o **programador de IA**, é uma forma de **expandir o repertório de soluções** e de **depurar por analogia**. Reconhecer que um comportamento observado num jogo comercial "cheira a GOAP" ou "tem jeito de utility AI" ajuda o programador a escolher a arquitetura certa para um problema semelhante, e a entender os *trade-offs* que outra equipe já enfrentou. É também uma escola de humildade técnica: perceber que um efeito impressionante foi obtido com uma técnica simples ensina a **não superdimensionar** as próprias soluções.

Para o **pesquisador e o estudante**, é o método por excelência de **transformar observação em conhecimento estruturado**. É aqui que esta apostila se situa. A engenharia reversa é a ponte entre a teoria abstrata — que você estudou nas seis Partes anteriores — e os artefatos concretos do mundo real. Ela obriga a **integrar** todo o conhecimento: não se reconhece uma técnica isoladamente, reconhece-se por **contraste** com todas as outras que ela não é. Por isso este é o capítulo integrador da apostila: para dizer "provavelmente é uma FSM", é preciso saber, simultaneamente, o que **não** é uma behavior tree, o que **não** é um planejador e o que **não** é aprendizado.

[DIAGRAMA]
Título: O problema da caixa-preta na IA de jogos
Objetivo pedagógico: Fixar visualmente que a engenharia reversa parte do comportamento observável (saída) para inferir o mecanismo interno (oculto).
Descrição detalhada: Uma caixa retangular central, opaca e cinza, rotulada "IA DO JOGO (código fechado)". À esquerda, entrando na caixa, setas rotuladas "ENTRADAS observáveis" — estado do jogo, ações do jogador, posição, ruídos, tempo. À direita, saindo da caixa, setas rotuladas "SAÍDAS observáveis" — comportamento do NPC, animações, escolhas, movimentos, falas. Acima da caixa, um ponto de interrogação grande e a legenda "O que há aqui dentro? FSM? BT? GOAP? Utility? Pathfinding?". Abaixo, uma seta curva ligando as saídas de volta a um bloco rotulado "OBSERVADOR (engenheiro reverso): observa, registra, isola variáveis, formula hipóteses". Uma seta tracejada vai do OBSERVADOR de volta à caixa, rotulada "hipótese (nunca certeza)".
Elementos obrigatórios: caixa-preta central rotulada; setas de entrada e de saída observáveis; ponto de interrogação sobre o interior; bloco do observador; seta tracejada de hipótese indicando incerteza.
[/DIAGRAMA]

---

## 14.2 Fundamentos

A engenharia reversa de IA repousa sobre um único princípio, herdado do **método científico**: *não temos acesso ao mecanismo interno, mas temos acesso ao comportamento que ele produz; portanto, estudamos o comportamento de forma controlada para inferir o mecanismo*. Tudo o que se segue são consequências disciplinadas desse princípio. Esta seção o desdobra em três fundamentos: a **observação sistemática** (como olhar), o raciocínio por **estímulo e resposta** (como provocar o sistema para que revele sua lógica) e o repertório de **sinais** que permitem **identificar técnicas** (o que procurar).

### 14.2.1 Observação sistemática

Observar não é o mesmo que assistir. Assistir é passivo; observar é uma atividade **metódica**, guiada por regras que separam a evidência da impressão. Quatro princípios estruturam a observação sistemática.

**Observação controlada.** A primeira regra é **controlar as condições** em que se observa. Um comportamento visto uma única vez, em meio ao caos de uma partida normal, quase nada informa: não sabemos o que o causou. A observação útil acontece em condições que o observador **escolhe e conhece** — de preferência as mais simples possíveis. Em vez de tirar conclusões de um tiroteio confuso com seis inimigos, o engenheiro reverso monta uma cena mínima: **um** inimigo, um ambiente conhecido, um estímulo único. Quanto mais simples a cena, mais clara a relação entre causa e efeito. Muitos jogos oferecem recursos que ajudam nisso — modos sandbox, fases de teste, dificuldade ajustável, códigos de invulnerabilidade que permitem observar sem morrer. Onde existirem, são o laboratório do analista.

**Repetição de experimentos.** A segunda regra é **repetir**. Uma observação isolada pode ser um acidente, um caso de borda, um bug. Só a repetição distingue o **padrão** do **acaso**. Se, ao ser avistado, o inimigo procura cobertura, a pergunta é: *ele faz isso sempre? Faz da mesma forma?* Repetir o mesmo estímulo dezenas de vezes revela a diferença crucial entre um comportamento **determinístico** (sempre a mesma resposta) e um **estocástico ou variável** (respostas diferentes a cada vez) — e essa diferença, como veremos, é um dos sinais mais informativos sobre a arquitetura por trás. A comunidade de Pac-Man só conseguiu mapear os fantasmas porque **repetiu** os mesmos movimentos milhares de vezes até que os padrões emergissem.

**Isolamento de variáveis.** A terceira regra, emprestada diretamente do experimento científico, é **mudar uma coisa de cada vez**. Se quero saber o que dispara o alerta de um inimigo, não posso, na mesma tentativa, correr, atirar e acender a lanterna — se ele reagir, não saberei a qual dos três estímulos. O método é **isolar**: numa tentativa, só faço barulho; noutra, só entro no campo de visão; noutra, só aceno uma luz. Comparando as respostas, atribuo cada reação a uma causa. Esse isolamento é o que permite mapear a **percepção** do agente: seu alcance de visão, seu ângulo, sua sensibilidade a som, se enxerga no escuro, se tem memória de onde viu o jogador pela última vez.

**Coleta de evidências.** A quarta regra é **registrar**. A memória humana é péssima testemunha: ela confunde, generaliza e inventa padrões que não existem. A observação séria produz um **registro** — anotações, tabelas, capturas de tela, gravações de vídeo que possam ser revistas quadro a quadro. A evidência registrada é o que separa uma hipótese fundamentada de uma impressão vaga, e é o que permite que **outra pessoa** verifique a análise. Uma boa evidência é **específica e datável**: "no minuto 3:42, ao ouvir o tiro a ~15 metros atrás de uma parede, o guarda virou-se e caminhou até a origem do som, parando na parede" vale infinitamente mais do que "os guardas escutam tiros".

> ✅ **Boa Prática**
> Grave suas sessões de observação. Uma captura de vídeo permite avançar quadro a quadro e flagrar detalhes invisíveis em tempo real — o instante exato em que um inimigo muda de postura, a fração de segundo entre o estímulo e a reação (que sugere tempos de "percepção"), a repetição idêntica de uma animação (que sugere um número finito de estados). Muitas descobertas de engenharia reversa só aparecem no *replay* lento, nunca no jogo ao vivo.

[DIAGRAMA]
Título: Fluxo da observação sistemática
Objetivo pedagógico: Apresentar a observação como um laço metódico, e não como um olhar passivo.
Descrição detalhada: Um ciclo fechado com cinco caixas ligadas por setas em sentido horário. (1) "Montar cena controlada" — ambiente simples, um NPC, condições conhecidas. (2) "Aplicar estímulo isolado" — uma única variável por vez. (3) "Observar e registrar resposta" — anotações, vídeo, captura. (4) "Repetir" — várias vezes o mesmo estímulo, com uma seta curta que retorna ao passo 2. (5) "Comparar respostas" — determinístico × variável; atribuir efeito à causa. Do passo 5 sai uma seta rotulada "→ evidência consolidada" que aponta para fora do ciclo, em direção à seção 14.3 (formular hipóteses). Uma nota lateral: "Mudar UMA variável de cada vez".
Elementos obrigatórios: as cinco etapas nomeadas; o laço de repetição destacado; a saída para "evidência consolidada"; a nota sobre isolamento de variáveis.
[/DIAGRAMA]

### 14.2.2 Estímulo e resposta

O coração da engenharia reversa comportamental é tratar o NPC como um sistema de **estímulo e resposta**: aplico uma entrada conhecida, observo a saída e, do padrão que liga as duas, infiro a lógica interna. Levantar boas hipóteses depende de saber **onde olhar** — e há cinco eixos de observação especialmente reveladores, cada um deles diretamente ligado a um conceito estudado na apostila.

**Percepção do NPC.** Antes de decidir, o agente precisa **sentir** (o "Sentir" do ciclo Sentir–Pensar–Agir da Parte I). Sondar a percepção é frequentemente o primeiro passo, porque ela delimita tudo o que vem depois. As perguntas são concretas: *A que distância o inimigo me detecta? Ele tem um cone de visão, ou enxerga em 360°? Percebe som? Enxerga no escuro? Reage a luz? Perde-me de vista quando me escondo — e, se sim, por quanto tempo continua procurando onde me viu por último?* As respostas revelam o **modelo sensorial** do agente, e já eliminam muitas hipóteses: um inimigo que reage instantaneamente a você atrás dele, sem linha de visão, provavelmente **trapaceia** (lê seu estado diretamente) em vez de perceber; um que demora, procura na direção errada e desiste tem um modelo de percepção mais rico e honesto.

**Tomada de decisão.** Uma vez que percebe, o agente **escolhe**. Aqui a observação foca no *como* da escolha: *As decisões são sempre as mesmas nas mesmas condições (determinístico) ou variam (aleatoriedade/ponderação)? Há uma ordem de prioridade fixa — sempre atacar antes de recarregar? A escolha parece binária (faz ou não faz) ou graduada (às vezes recua, às vezes avança, conforme a situação)?* O caráter da decisão é um sinal forte: prioridades rígidas e fixas sugerem regras (FSM, árvores); escolhas graduadas e sensíveis a múltiplos fatores sugerem ponderação (utility AI); sequências que "montam um plano" sob medida sugerem planejamento (GOAP).

**Mudança de estados.** Muitos comportamentos se organizam em **modos** discretos e observáveis — patrulhar, alertar, perseguir, atacar, recuar, procurar. Detectar esses modos, e sobretudo as **transições** entre eles, é revelador. As perguntas: *Quais modos distintos consigo identificar? O que dispara cada transição? As transições são abruptas (troca instantânea de postura, música, animação) ou suaves? Há modos que só aparecem "dentro" de outros (por exemplo, um sub-comportamento de recarga que só ocorre durante o combate)?* Transições abruptas e disparadas por eventos claros são a assinatura das **máquinas de estado** (Parte II); modos-dentro-de-modos sugerem **hierarquia** (HFSM).

**Planejamento.** Alguns agentes não apenas reagem: eles parecem **encadear ações em direção a um objetivo**, de formas diferentes conforme a situação. O sinal é a **variedade estruturada**: o mesmo inimigo, diante do mesmo objetivo (chegar até você), ora chuta uma porta, ora pula uma janela, ora dá a volta — escolhendo a sequência conforme o ambiente, e não repetindo um roteiro fixo. Quando as **ações intermediárias** parecem existir para **satisfazer pré-condições** de uma ação final (empurrar uma mesa para criar cobertura, e só então atirar), há forte indício de **planejamento orientado a objetivos** (GOAP).

**Comportamento emergente.** Por fim, alguns dos efeitos mais impressionantes não estão programados em nenhum agente individual: **emergem** da interação de regras simples entre muitos agentes, ou entre agentes e ambiente. Bandos que parecem coordenados sem um "líder", situações que nunca se repetem igual, dinâmicas de grupo que surpreendem até os desenvolvedores — tudo isso é sinal de **emergência** (um conceito central desde a Parte I). O engenheiro reverso precisa de cuidado redobrado aqui, porque a emergência é a maior **fonte de ilusão**: ela faz parecer que há um mecanismo sofisticado de coordenação onde, muitas vezes, há apenas regras locais simples se somando. Reconhecer emergência é reconhecer que "parece coordenado" **não** implica "há um coordenador".

> ❌ **Erro Comum**
> Confundir "parece inteligente" com "é complexo". O erro mais frequente do analista iniciante é **superestimar** o mecanismo: ao ver um esquadrão que flanqueia de forma convincente, conclui que há um planejador tático central sofisticado, quando muitas vezes o efeito vem de regras individuais simples ("se um aliado está atirando, eu procuro outro ângulo") que **se somam** em coordenação aparente. É o caso célebre de *F.E.A.R.*, que estudaremos: a "coordenação de esquadrão" tão elogiada é, em boa parte, **ilusão emergente** produzida por agentes que planejam **individualmente** e **anunciam** suas ações em voz alta. A regra de ouro: **prefira sempre a hipótese mais simples que explique a evidência** (é a navalha de Occam aplicada à IA de jogos).

### 14.2.3 Identificando técnicas

Reunindo os fundamentos anteriores, podemos agora catalogar os **sinais característicos** de cada família de técnicas estudada na apostila. Este é o núcleo prático do capítulo: uma lista de "sintomas" que, observados, elevam a probabilidade de determinada hipótese. Um aviso, porém, precede tudo o que se segue e vale para cada item da lista: **nenhum sinal é prova**. Os sinais são **indícios** que tornam uma hipótese mais ou menos provável; várias técnicas produzem comportamentos parecidos, e os jogos reais quase sempre **combinam** técnicas. O objetivo é formular hipóteses **fundamentadas e graduadas em confiança**, jamais afirmar com certeza absoluta qual algoritmo foi usado.

**Sinais de FSM — Máquina de Estados Finita (Parte II, Cap. 3).**
Procure por um **pequeno número de modos discretos** e bem definidos, com **transições abruptas** entre eles disparadas por **eventos claros**. O NPC alterna entre estados visíveis ("patrulha", "alerta", "combate", "fuga"), e a troca costuma vir acompanhada de sinais evidentes — mudança de animação, de música, de postura, de expressão. Comportamento **determinístico** dentro de cada estado e **repetitivo** ao longo do tempo reforça a hipótese. Se, com poucas horas de observação, você consegue **desenhar um diagrama de estados** que prevê o comportamento, é quase certo que há uma FSM (ou algo equivalente) por baixo.

**Sinais de HFSM — Máquina de Estados Hierárquica (Parte II, Cap. 4).**
Os mesmos sinais da FSM, acrescidos de **aninhamento**: comportamentos que só ocorrem **dentro** de outros. Você identifica um estado "Combate" que, internamente, alterna entre "atirar", "recarregar" e "buscar cobertura" — e percebe que **sair** do combate (por exemplo, ao ser surpreendido por uma granada) **interrompe todo esse submodo de uma vez**, retornando a um estado de nível superior. Essa "herança" de transições — um evento de alto nível cancela toda a atividade de baixo nível — é a assinatura da hierarquia.

**Sinais de Behavior Trees — Árvores de Comportamento (Parte II, Cap. 6).**
Procure por comportamento **modular, em camadas de prioridade**, com **reavaliação frequente**. Diferentemente da FSM, aqui não há tantas "transições" explícitas: o agente parece, a cada instante, **percorrer uma lista de prioridades de cima para baixo** e executar a primeira opção viável ("se há uma ameaça grave, reagir a ela; senão, se há um alvo, atacá-lo; senão, patrulhar"). O sinal típico é a **retomada suave**: interrompido por algo urgente, o NPC resolve o urgente e **volta naturalmente** à atividade anterior, sem "resetar". Comportamentos que se compõem de **blocos reutilizáveis** e claramente **priorizados**, com um NPC que reage a mudanças reavaliando constantemente suas opções, sugerem behavior tree — hoje a arquitetura mais comum na indústria para NPCs de ação.

**Sinais de Árvores de Decisão (Parte II, Cap. 5).**
Procure por decisões que parecem resultado de uma **cadeia de perguntas sim/não** encadeadas ("o jogador está visível? → está no alcance? → tenho munição? → atacar"). O comportamento é **classificatório** e **determinístico**: dadas as mesmas condições, a mesma folha é sempre alcançada. Diferentemente da behavior tree, a árvore de decisão **não executa ações duradouras nem tem noção de "continuar" uma tarefa** — ela apenas **escolhe** uma ação a cada avaliação. Decisões rápidas, nítidas e perfeitamente previsíveis a partir de condições observáveis são o seu indício.

**Sinais de GOAP — Planejamento Orientado a Objetivos (Parte II, Cap. 6; Parte V).**
Procure por **sequências variáveis de ações que convergem para um mesmo objetivo**, adaptadas ao contexto. O mesmo inimigo, para chegar até você, ora vira uma mesa e se agacha atrás dela, ora rodeia por outra sala, ora arromba uma porta — **escolhendo os passos conforme o ambiente**, e não repetindo um roteiro. O sinal decisivo é a presença de **ações-meio** que só fazem sentido como **pré-condição** de uma ação-fim (mover-se até a cobertura *para* poder atirar protegido). Quando o comportamento dá a impressão de que o agente "montou um plano sob medida para esta situação", GOAP é uma hipótese forte.

**Sinais de Utility AI — IA de Utilidade (Parte II, Cap. 6).**
Procure por decisões **graduadas e sensíveis a múltiplos fatores simultâneos**, em vez de gatilhos binários. O agente não "liga" um comportamento ao cruzar um limiar: ele parece **ponderar** o tempo todo, escolhendo a ação que no momento é a "mais valiosa" dado um conjunto de necessidades ou pressões que variam continuamente. O sinal clássico está em agentes com **múltiplas necessidades concorrentes** que se equilibram — como os personagens de *The Sims*, que decidem entre comer, dormir, socializar e trabalhar conforme níveis internos que sobem e descem. Comportamento que muda **suavemente** com o contexto, difícil de reduzir a um punhado de regras binárias, sugere ponderação por utilidade.

**Sinais de Pathfinding — Busca de Caminhos (Parte III).**
Este é um dos sinais mais fáceis de reconhecer, porque quase todo jogo com movimento o usa. Procure por agentes que **navegam de forma inteligente por ambientes complexos**: contornam obstáculos, encontram rotas não óbvias, atravessam portas e corredores em direção a um alvo. Sinais de **A\*** e afins incluem trajetos que parecem "ótimos" ou quase, recálculo de rota quando o caminho é bloqueado, e — reveladoramente — **artefatos** típicos: movimento levemente "quadriculado" ou em ziguezague (grade), ou trajetos que "colam" em bordas de polígonos (NavMesh). Já falhas de pathfinding — inimigos que ficam presos em cantos, que não atravessam um vão óbvio, ou que fazem rotas absurdamente longas — confirmam que **há** um sistema de busca (e revelam seus limites).

**Sinais de Mapas de Influência (Parte IV).**
Procure, sobretudo em **jogos de estratégia**, por decisões que dependem da **distribuição espacial de forças** no mapa. Sinais: a IA "sente" regiões perigosas e as evita ou reforça; concentra ataques em pontos fracos da frente inimiga; recua de áreas dominadas pelo oponente; posiciona unidades de forma tática em relação ao controle de território. Quando o comportamento estratégico parece guiado por um "mapa de calor" invisível de ameaça e oportunidade — decidindo *onde* agir com base em quem domina o quê no espaço —, mapas de influência são a hipótese natural.

**Sinais de Minimax e Busca Adversarial (Parte V).**
Procure, em **jogos de tabuleiro e de turnos** (xadrez, damas, jogos de estratégia por turnos), por um oponente que **antecipa jogadas** e joga de forma **forte e consistente**. Sinais: a IA evita armadilhas de várias jogadas de profundidade, faz sacrifícios que só compensam lances depois, e tem uma força que escala com o "tempo de pensamento" ou com o nível de dificuldade (mais profundidade de busca). Um oponente que joga **perfeitamente** em posições simples mas comete erros de "horizonte" em posições muito profundas é a assinatura clássica de uma busca com profundidade limitada e função de avaliação.

**Sinais de Aprendizagem por Reforço e adaptação (Parte VI).**
Este é o sinal mais **raro** e o que exige **mais cautela**, justamente porque é o mais alegado sem fundamento. O indício genuíno é a **mudança de comportamento ao longo do tempo em resposta à experiência**: a IA que, de fato, joga diferente na décima partida por causa do que "aprendeu" nas nove anteriores. Cuidado: a imensa maioria dos jogos que "parecem aprender" **não usa** aprendizado real — usa **adaptação por regras** (dificuldade dinâmica, gatilhos condicionais, roteiros que mudam conforme o desempenho do jogador), que produz uma ilusão de aprendizado com técnicas determinísticas. Só se deve levantar a hipótese de aprendizado verdadeiro quando há **evidência de melhoria persistente e não-scriptada**, idealmente confirmada por documentação — e mesmo então, com reservas.

> **Observação de Campo**
> Um atalho útil para começar qualquer análise: **conte os estados**. Se você consegue, em pouco tempo, enumerar um punhado de "modos" claros e prever as transições, está diante de algo próximo de uma FSM/BT — a esmagadora maioria dos NPCs de ação. Se o comportamento **resiste** a essa contagem — porque é graduado (utility), porque monta planos sob medida (GOAP), porque depende do mapa inteiro (influência) ou porque muda com o tempo (aprendizado) —, a análise fica mais interessante, e é aí que o repertório completo da apostila se torna indispensável.

[DIAGRAMA]
Título: Árvore de decisão para identificação de técnicas
Objetivo pedagógico: Oferecer um roteiro de triagem que conduz o analista, por perguntas, à família de técnicas mais provável.
Descrição detalhada: Um fluxograma em forma de árvore, lido de cima para baixo, com perguntas nos nós e famílias de técnicas nas folhas. Nó raiz: "O comportamento muda de forma persistente com a experiência ao longo de várias partidas?" — SIM → folha "Aprendizado real (Parte VI) — raro; exigir evidência forte"; NÃO → próximo nó. Nó 2: "É um jogo de tabuleiro/turnos com oponente que antecipa jogadas?" — SIM → folha "Minimax / busca adversarial (Parte V)"; NÃO → próximo nó. Nó 3: "As decisões dependem da distribuição de forças no mapa (estratégia espacial)?" — SIM → folha "Mapas de influência (Parte IV)"; NÃO → próximo nó. Nó 4: "O agente monta sequências variáveis de ações com pré-condições para atingir um objetivo?" — SIM → folha "GOAP (planejamento)"; NÃO → próximo nó. Nó 5: "As decisões são graduadas, ponderando várias necessidades concorrentes simultâneas?" — SIM → folha "Utility AI"; NÃO → próximo nó. Nó 6: "Dá para enumerar poucos modos discretos com transições por eventos claros?" — SIM, e há modos aninhados → folha "HFSM"; SIM, sem aninhamento → folha "FSM / Behavior Tree"; NÃO → folha "Sistema híbrido/indeterminado — reobservar". Nota lateral em todas as folhas: "Pathfinding (Parte III) quase sempre coexiste, se há navegação." Rodapé do diagrama: "Toda folha é uma HIPÓTESE, não uma certeza."
Elementos obrigatórios: nó raiz sobre aprendizado; a cadeia de perguntas na ordem indicada; folhas nomeadas por família de técnica; nota sobre pathfinding coexistente; rodapé reforçando o caráter hipotético.
[/DIAGRAMA]

---

## 14.3 Funcionamento: roteiro de análise de uma IA

Os fundamentos da seção anterior tornam-se operacionais quando organizados em um **roteiro** — uma sequência de etapas que qualquer estudante pode seguir para analisar a IA de um jogo de forma disciplinada e produzir um documento defensável. O roteiro a seguir é **reutilizável**: ele funciona como um formulário que você preencherá, no Capítulo 15, para cada estudo de caso, e que deverá aplicar no Projeto Integrador ao final desta Parte. São **seis etapas**.

### Etapa 1 — Definição do problema

Toda análise começa delimitando **o que se quer descobrir**. "Analisar a IA de *Halo*" é vago demais e leva a lugar nenhum. "Descobrir como os inimigos de *Halo* decidem quando recuar e quando avançar" é uma pergunta analisável. Nesta etapa, você define o **escopo** (qual agente, qual comportamento, em que situação) e formula uma **pergunta de pesquisa** concreta. Um bom escopo é **estreito**: é melhor entender profundamente **um** comportamento do que superficialmente todos. Registre também o que **já se sabe de antemão** — há documentação oficial? Palestras? Postmortems? Isso define, desde já, quais conclusões poderão ser "documentadas" e quais serão "inferência".

### Etapa 2 — Coleta de evidências

Aqui se aplica, na prática, a **observação sistemática** da seção 14.2.1. Você monta cenas controladas, aplica estímulos isolados, repete cada experimento e **grava tudo**. O produto desta etapa é um **corpo de evidências brutas**: vídeos, capturas, medições de tempo e distância, contagens de repetição. A disciplina é não interpretar ainda — apenas **coletar** com o máximo de controle e o mínimo de suposição. Uma lista de estímulos-padrão a testar sempre: aproximar-se lentamente (mapear o alcance de detecção); fazer barulho fora do campo de visão (testar audição); entrar e sair da linha de visão (testar memória/persistência); atacar e recuar (testar transições de combate); bloquear o caminho do agente (testar pathfinding); colocá-lo diante de escolhas (testar prioridades).

### Etapa 3 — Registro das observações

As evidências brutas precisam ser **organizadas** em observações estruturadas. Aqui você transforma "assisti ao vídeo" em anotações padronizadas: uma **tabela de estímulo → resposta**, uma **lista dos modos/estados** identificados com seus gatilhos aparentes, medições consolidadas (alcance de visão em metros, tempo entre estímulo e reação, número de comportamentos distintos observados). O registro deve ser **específico, datável e verificável** — de modo que outra pessoa, lendo suas anotações, entenda exatamente o que foi observado e possa reproduzir. Esta etapa é o que separa a análise séria da impressão de sofá.

### Etapa 4 — Formulação de hipóteses

Com as observações organizadas, você **levanta hipóteses** sobre o mecanismo interno, usando o repertório de sinais da seção 14.2.3 e a árvore de triagem do diagrama anterior. O ponto essencial — que a apostila repete sem cansar — é formular **várias hipóteses concorrentes** e atribuir a cada uma um **nível de confiança**, em vez de se casar com a primeira. Cada hipótese deve vir **amarrada à evidência que a sustenta** ("hipótese: FSM, porque observei apenas quatro modos discretos com transições abruptas por eventos claros — confiança alta") e, idealmente, à evidência que a **enfraqueceria** se aparecesse. Aplique deliberadamente a **navalha de Occam**: entre duas hipóteses que explicam igualmente bem os dados, prefira a mais simples.

### Etapa 5 — Validação das hipóteses

Uma hipótese só vale o quanto resiste a **tentativas de refutá-la**. Nesta etapa, você projeta **novos experimentos** especificamente desenhados para **testar** — e de preferência **quebrar** — cada hipótese. Se a hipótese é "o inimigo tem um cone de visão de ~90° e não me vê pelas costas", então o teste é aproximar-se por trás repetidamente: se ele reagir, a hipótese cai. Se a hipótese é "há apenas quatro estados", o teste é procurar exaustivamente um quinto. A validação é o passo mais científico e o mais negligenciado: é fácil coletar evidências que **confirmam** o que já achamos (viés de confirmação); o analista honesto busca ativamente as que **desmentem**. Quando existe documentação oficial, esta é também a etapa de **confrontar** a hipótese com a fonte — e registrar honestamente se ela foi confirmada, refutada ou apenas não contradita.

### Etapa 6 — Documentação da análise

Por fim, a análise se consolida em um **documento** comunicável, que **distingue explicitamente** três camadas: (a) os **fatos observados** (o que se viu, com evidência); (b) as **hipóteses** levantadas, cada uma com seu nível de confiança e a evidência que a sustenta; e (c) o que permaneceu **indeterminado**. Quando houver documentação oficial usada, ela deve ser **citada** e claramente separada das inferências próprias. Um bom documento de engenharia reversa nunca apresenta suposição como fato — e é justamente essa honestidade que o torna útil e confiável. O Capítulo 15 inteiro é uma coleção de documentos escritos nesse formato; use-os como modelo.

[DIAGRAMA]
Título: Roteiro reutilizável de engenharia reversa de IA (seis etapas)
Objetivo pedagógico: Fornecer o processo completo em um único quadro, servindo de checklist para o Projeto Integrador.
Descrição detalhada: Um fluxo linear com seis blocos numerados, ligados por setas da esquerda para a direita (ou de cima para baixo), com um laço de retorno importante. (1) DEFINIÇÃO DO PROBLEMA — escopo estreito + pergunta de pesquisa + levantamento de fontes existentes. (2) COLETA DE EVIDÊNCIAS — cenas controladas, estímulos isolados, repetição, gravação. (3) REGISTRO DAS OBSERVAÇÕES — tabelas estímulo→resposta, lista de modos, medições. (4) FORMULAÇÃO DE HIPÓTESES — várias hipóteses concorrentes, cada uma com nível de confiança e evidência; navalha de Occam. (5) VALIDAÇÃO — experimentos que tentam REFUTAR cada hipótese; confronto com documentação oficial. (6) DOCUMENTAÇÃO — separar fatos × hipóteses × indeterminado; citar fontes. Uma seta de retorno tracejada vai do bloco 5 de volta ao bloco 2, rotulada "se a hipótese cai, coletar novas evidências". Faixa inferior atravessando todos os blocos: "Princípio constante: hipótese ≠ confirmação; registrar sempre o nível de confiança".
Elementos obrigatórios: os seis blocos nomeados e numerados; o laço de retorno da validação para a coleta; a faixa inferior com o princípio da incerteza; destaque na etapa 5 para "tentar refutar".
[/DIAGRAMA]

> 🎮 **Na Prática**
> O roteiro parece longo, mas na prática ele se torna um hábito rápido. Depois de aplicá-lo a dois ou três jogos, você passa a executar as etapas 1 a 4 quase automaticamente durante o próprio jogo, reservando as etapas 5 e 6 (validação e documentação) para quando quiser produzir uma análise formal. O valor do roteiro não é a burocracia — é garantir que você **não pule** as duas partes que os iniciantes sempre pulam: **tentar refutar** a própria hipótese e **separar** o que viu do que supôs.

---

## 14.4 Ética e limites da engenharia reversa

Nenhum método é neutro, e a engenharia reversa levanta questões éticas e epistemológicas que um curso universitário deve enfrentar de frente. Esta seção trata de quatro delas.

### Engenharia reversa acadêmica versus violação de propriedade intelectual

Há uma diferença fundamental — jurídica e ética — entre **observar o comportamento** de um jogo e **apropriar-se de sua propriedade intelectual**. Tudo o que esta apostila ensina situa-se firmemente no primeiro campo: **análise comportamental de fora**, tratando o jogo como uma caixa-preta que se observa jogando normalmente, como qualquer usuário legítimo faria. Isso é análise, é estudo, é crítica técnica — atividades protegidas e legítimas, tão antigas quanto o hábito de estudar as obras que admiramos para aprender com elas.

Muito diferente é **descompilar** o código de um jogo, contornar proteções técnicas, extrair *assets* protegidos ou **copiar** a implementação de um sistema para um produto concorrente. Essas ações entram no terreno da **violação de direitos autorais, de contratos de licença (EULA) e, potencialmente, de leis de proteção**, e **não** são o que se ensina aqui. A distinção prática é clara: **observar o que o jogo mostra a quem joga é legítimo; abrir o que o jogo esconde de quem joga é outra coisa**. Reconhecer que um inimigo "provavelmente usa GOAP" a partir de como ele se comporta é análise; obter o código do planejador e reutilizá-lo é apropriação. A engenharia reversa desta apostila é, e deve permanecer, **estritamente comportamental**.

> ⚠️ **Atenção**
> Ideias e técnicas — FSM, GOAP, A\*, behavior trees — **não** são propriedade de ninguém: são conhecimento científico e de engenharia, de domínio público, que esta apostila inteira ensina livremente. O que é protegido é a **implementação específica** de um estúdio (seu código, seus *assets*, sua arte). Aprender que um problema pode ser resolvido com GOAP porque *F.E.A.R.* o fez é uso perfeitamente legítimo do conhecimento. Copiar o código de *F.E.A.R.* não é. Mantenha essa fronteira sempre nítida.

### Uso responsável da documentação

Quando um jogo tem documentação oficial — uma palestra de GDC, um postmortem, um artigo de seus criadores —, o uso responsável dela tem duas faces. A primeira é **creditar corretamente**: uma afirmação que vem de uma fonte oficial deve ser atribuída a ela ("segundo Jeff Orkin, em sua palestra de 2006, *F.E.A.R.* usa GOAP"), e não apresentada como descoberta própria. A segunda é **ler criticamente**: a documentação oficial é valiosíssima, mas descreve o sistema **como seus autores escolheram descrevê-lo** — em geral simplificado, idealizado e posterior ao lançamento. Ela não é a "verdade absoluta" do código que rodou; é o **relato dos criadores**, sujeito a simplificação e a memória. O analista responsável usa a documentação como a **melhor evidência disponível**, sem tratá-la como oráculo infalível, e sempre a distingue de suas próprias inferências.

### Limitações da observação externa

É preciso ser honesto sobre o que a observação de fora **não** consegue fazer. A engenharia reversa comportamental tem limites intransponíveis:

Ela **não distingue implementações comportamentalmente equivalentes**. Um mesmo comportamento observável pode ser produzido por uma FSM ou por uma behavior tree cuidadosamente montada para imitá-la; de fora, pode ser impossível decidir qual. A observação revela o **quê**, raramente o **como exato**.

Ela **não vê o que não é exercido**. Comportamentos raros, que só aparecem em condições específicas, podem passar despercebidos por horas de observação — levando a subestimar a riqueza do sistema. A ausência de evidência não é evidência de ausência.

Ela **confunde-se com a aleatoriedade e a emergência**. Como vimos, efeitos emergentes de regras simples podem parecer mecanismos complexos, e variações aleatórias podem parecer "decisões". Distinguir os dois exige muita repetição e, ainda assim, nem sempre é conclusivo.

Ela **é sensível à versão e ao contexto**. A IA de um jogo muda entre versões, patches e níveis de dificuldade; uma análise feita numa versão pode não valer para outra.

Reconhecer esses limites não enfraquece o método — ao contrário, é o que o torna honesto. A engenharia reversa produz **hipóteses bem fundamentadas**, não certezas, e sua utilidade está exatamente em saber **até onde** ela pode ir.

### A diferença entre hipótese e confirmação

Chegamos ao princípio que é a espinha dorsal de toda esta Parte. Uma **hipótese** é uma explicação provável, sustentada por evidências, mas passível de estar errada. Uma **confirmação** é uma verificação que estabelece a hipótese como fato — e, em engenharia reversa de IA, ela quase sempre depende de uma **fonte externa à observação**: o código-fonte, ou uma declaração oficial dos criadores. Sem uma dessas, permanecemos, por mais confiantes que estejamos, no terreno da hipótese.

Isso tem uma consequência direta na forma como escrevemos. Frases como "*F.E.A.R.* usa GOAP" só são apropriadas quando há documentação que as confirme (e, nesse caso, há — a palestra de Orkin). Frases como "os inimigos de *Halo* provavelmente reavaliam prioridades numa árvore de comportamento" devem manter o "provavelmente", porque descrevem uma **inferência**. A disciplina de escrever com esse cuidado — qualificar cada afirmação pelo seu nível de certeza — não é um preciosismo acadêmico: é o que distingue a análise técnica séria da especulação de fórum. Todo o Capítulo 15 é construído sobre essa distinção.

> ✅ **Boa Prática**
> Adote três rótulos ao escrever qualquer análise, e use-os explicitamente: **[Documentado]** para o que uma fonte oficial confirma; **[Inferência]** para o que você deduz da observação com boa base; **[Especulação]** para hipóteses plausíveis mas frágeis. Marcar cada afirmação com um desses rótulos — como faremos nos estudos de caso — força a honestidade intelectual e torna a análise imediatamente confiável para quem a lê.

**Nota sobre os estudos de caso desta apostila.** Fica aqui o compromisso que vale para todo o Capítulo 15: salvo quando explicitamente indicado como **documentado** por fonte oficial, tudo o que a apostila afirma sobre a implementação interna dos jogos analisados constitui **análise técnica fundamentada** — inferência a partir de comportamento observável e do repertório teórico —, e **não** um relato verificado do código. Essa ressalva não é uma formalidade: é a aplicação honesta da própria metodologia que este capítulo ensina.

---

## 14.5 Encerramento do Capítulo 14

### Resumo

Este capítulo construiu o **método** da Parte VII. Partiu do **problema** — os desenvolvedores raramente publicam os detalhes da IA que usam, por razões comerciais, de proteção da ilusão, culturais e porque a IA real costuma ser um **híbrido** sem "técnica única" a revelar —, e mostrou por que **analisá-la de fora** é uma competência valiosa para designers (estudar referências), programadores (expandir repertório) e pesquisadores/estudantes (integrar teoria e prática).

O núcleo do capítulo é uma **metodologia**. Seus **fundamentos** são a **observação sistemática** (observação controlada, repetição, isolamento de variáveis, coleta de evidências), o raciocínio por **estímulo e resposta** (percepção, decisão, mudança de estados, planejamento, emergência) e um repertório de **sinais** que permitem **identificar** cada família de técnicas — FSM, HFSM, behavior trees, árvores de decisão, GOAP, utility AI, pathfinding, mapas de influência, minimax e aprendizado —, sempre lembrando que sinais são **indícios**, não provas. Esses fundamentos se operacionalizam num **roteiro reutilizável de seis etapas**: definição do problema, coleta de evidências, registro das observações, formulação de hipóteses, validação (tentando **refutar**) e documentação (separando fato de inferência).

O capítulo fechou com a **ética e os limites**: a fronteira entre a engenharia reversa **comportamental e legítima** e a **violação de propriedade intelectual**; o uso responsável e crítico da documentação oficial; as **limitações reais** da observação externa; e o princípio que atravessa toda a Parte — **hipótese não é confirmação** —, materializado nos rótulos [Documentado], [Inferência] e [Especulação] que guiarão os estudos de caso.

### Questões de Revisão

1. Cite quatro razões pelas quais os estúdios raramente publicam os detalhes internos da IA de seus jogos. Qual delas está mais diretamente ligada ao conceito de "ilusão de inteligência"?
2. Explique, com um exemplo próprio, a diferença entre **observação controlada** e simplesmente **jogar e reparar**. Por que a repetição e o isolamento de variáveis são indispensáveis?
3. Um NPC alterna entre poucos modos claros ("patrulha", "alerta", "combate") com transições abruptas por eventos. Que hipótese isso sugere? Que observação adicional distinguiria uma FSM de uma HFSM?
4. Por que "parece coordenado" **não** implica "há um coordenador"? Relacione sua resposta ao conceito de comportamento emergente e ao caso de *F.E.A.R.*
5. Descreva as seis etapas do roteiro de engenharia reversa. Qual etapa os iniciantes mais negligenciam, e por quê?
6. Diferencie **hipótese** de **confirmação** no contexto da IA de jogos. O que, na prática, permite passar de uma à outra?

### Exercícios

1. **Sinais e técnicas.** Para cada comportamento observado a seguir, indique a família de técnica **mais provável**, justificando com o sinal correspondente e atribuindo um nível de confiança (alto/médio/baixo): (a) um oponente de xadrez que sacrifica uma peça e ganha vantagem cinco lances depois; (b) um personagem que decide sozinho entre comer, dormir e trabalhar conforme "medidores" internos; (c) um inimigo que, para te alcançar, ora arromba a porta, ora pula a janela, ora dá a volta; (d) uma IA de RTS que concentra ataques exatamente no ponto mais fraco da sua linha de defesa.
2. **Projeto de experimento.** Você quer descobrir o **modelo de percepção** de um guarda em um jogo furtivo: alcance de visão, ângulo, se ouve som e se tem memória de onde te viu. Escreva um plano de experimentos que **isole uma variável de cada vez**, indicando o que observar e registrar em cada teste, e como a repetição entraria no plano.
3. **Refutação de hipótese.** Sua hipótese é: "este inimigo tem exatamente três estados e é totalmente determinístico". Descreva dois experimentos **desenhados para refutar** essa hipótese (não para confirmá-la). O que cada resultado possível diria?
4. **Ética aplicada.** Classifique cada atividade como **legítima** (engenharia reversa comportamental) ou **problemática** (violação de PI), justificando: (a) gravar vídeos do jogo e mapear os padrões dos inimigos; (b) descompilar o executável para ler o código da IA; (c) escrever um artigo comparando o comportamento observado com a palestra de GDC dos criadores; (d) extrair e reutilizar os arquivos de configuração da IA num jogo próprio.
5. **Escrita disciplinada.** Reescreva as três afirmações a seguir aplicando os rótulos [Documentado], [Inferência] ou [Especulação] e ajustando a linguagem ao nível de certeza adequado: (a) "Os fantasmas de Pac-Man usam quatro personalidades diferentes"; (b) "O jogo aprende com você e fica mais difícil de propósito"; (c) "Segundo os criadores, o Diretor de IA de *Left 4 Dead* ajusta o ritmo do jogo dinamicamente".

### Referências

- ORKIN, Jeff. *Three States and a Plan: The A.I. of F.E.A.R.* Game Developers Conference (GDC), 2006.
- ISLA, Damian. *Handling Complexity in the Halo 2 AI*. Game Developers Conference (GDC), 2005.
- MILLINGTON, Ian. *AI for Games*. 3ª ed. Boca Raton: CRC Press, 2019.
- RABIN, Steve (org.). *Game AI Pro 3: Collected Wisdom of Game AI Professionals*. Boca Raton: CRC Press, 2017.
- BOURG, David M.; SEEMANN, Glenn. *AI for Game Developers*. Sebastopol: O'Reilly, 2004.
- RUSSELL, Stuart; NORVIG, Peter. *Inteligência Artificial*. 3ª ed. Rio de Janeiro: Elsevier, 2013. (Método científico, agentes e percepção.)
- YANNAKAKIS, Georgios N.; TOGELIUS, Julian. *Artificial Intelligence and Games*. Springer, 2018. (Discussão metodológica sobre IA em jogos.)
