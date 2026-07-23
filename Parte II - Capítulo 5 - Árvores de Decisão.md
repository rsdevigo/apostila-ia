# Capítulo 5 — Árvores de Decisão

> **Natureza deste capítulo.** Este é um capítulo de **apoio**, deliberadamente mais enxuto que os demais desta Parte. Seu papel é duplo: (1) apresentar a **árvore de decisão clássica** como uma terceira forma de organizar a seleção de ação — complementar à FSM e à HFSM — e (2) servir de **ponte conceitual** para o Capítulo 6, preparando o terreno e, sobretudo, desfazendo uma confusão de nomes muito comum no meio dos jogos: *árvore de decisão* **não** é o mesmo que *árvore de comportamento*. O peso principal da Parte II recai sobre o Capítulo 6; aqui construímos o degrau que leva até ele.

## Introdução

Os dois capítulos anteriores organizaram o comportamento em torno de **estados**: o agente está sempre "em algum modo" e transita entre modos. Há, porém, uma forma diferente e igualmente antiga de organizar a decisão, que não pensa em modos, mas em **perguntas encadeadas**. Em vez de "em que estado estou e para qual devo transitar?", ela pergunta: "dadas as condições atuais, qual ação devo escolher agora?" — e responde percorrendo uma sequência de testes, como quem desce por um fluxograma de "sim/não" até chegar a uma decisão.

Essa é a **árvore de decisão**. Ela é, provavelmente, a estrutura de decisão mais intuitiva que existe: qualquer pessoa que já seguiu um fluxograma de "se... então... senão..." já usou uma árvore de decisão sem saber o nome. Sua simplicidade a torna útil em jogos para decisões pontuais e rápidas, e sua estrutura em árvore — nós que se ramificam até folhas — é justamente o que prepara o leitor para a arquitetura de ênfase da Parte, as árvores de comportamento, que reaproveitam a *forma* da árvore para um propósito mais poderoso.

Seguimos, de forma condensada, o roteiro da apostila: **problema**, **fundamentos** (com a distinção essencial frente à árvore de comportamento), **funcionamento**, **exemplo**, **vantagens e limitações** (por que os jogos migraram para árvores de comportamento), **jogos e aplicações**, **ferramentas** e o fechamento.

> **Contexto Histórico**
> As árvores de decisão têm dupla origem. Como estrutura de *lógica condicional*, são tão antigas quanto os fluxogramas e a própria programação. Como modelo de *aprendizado de máquina*, ganharam destaque a partir dos anos 1980 com algoritmos que **constroem** a árvore automaticamente a partir de dados (o mais célebre deles, o ID3, e seus sucessores). Na IA de jogos, quase sempre usamos a primeira acepção — árvores *escritas à mão* pelo designer —, mas vale conhecer a segunda, pois ela reaparece quando se fala em IA que aprende (Parte VI).

---

## 5.1 O problema: escolher uma ação a partir de muitas condições

O problema que a árvore de decisão ataca é sutilmente diferente do da FSM. A FSM resolve "como alternar entre comportamentos contínuos ao longo do tempo, com memória de modo". A árvore de decisão resolve algo mais imediato: **dado um conjunto de condições, qual ação escolher *agora*, neste instante?** É um problema de *classificação de situação em ação*, sem necessidade intrínseca de memória de estado.

Considere um NPC que, a cada avaliação, precisa escolher uma entre várias ações possíveis com base em várias condições: vê o inimigo? está armado? tem vida suficiente? há cobertura por perto? Escrever isso como uma FSM seria desajeitado — não há realmente "modos" persistentes aqui, apenas uma decisão que depende da combinação atual de fatores. Escrever como uma pilha de `if/else if` aninhados funciona, mas rapidamente vira um bloco ilegível de código, difícil de visualizar e de ajustar pelo designer.

A árvore de decisão oferece uma terceira via: **estruturar essa cadeia de condições como uma árvore**, em que cada nó interno faz *uma pergunta* e cada ramo corresponde a *uma resposta*, conduzindo a subperguntas cada vez mais específicas, até chegar a uma **folha** que representa a ação escolhida. A mesma lógica dos `if/else` aninhados, mas agora **visualizável, ordenável e editável** como um diagrama — recuperando, para a lógica condicional, a autoria visual que a FSM oferecia para os modos.

> **Na Prática**
> Um bom teste para saber se um problema pede uma árvore de decisão (e não uma FSM) é perguntar: *"o comportamento tem 'modos' que persistem no tempo, ou é uma escolha recalculada do zero a cada avaliação?"* Se o agente precisa "lembrar que estava perseguindo", pense em FSM. Se ele só precisa "olhar a situação atual e escolher a melhor ação agora", a árvore de decisão é mais direta. Na prática, as duas coexistem: é comum uma FSM usar, *dentro* de um estado, uma pequena árvore de decisão para escolher qual variação de ação executar.

---

## 5.2 Fundamentos: nós de decisão, ramos e folhas

Uma árvore de decisão é composta por três tipos de elemento, dispostos em uma estrutura de árvore (um nó raiz, do qual tudo parte, sem ciclos):

**Nós de decisão (nós internos).** Cada nó interno contém um **teste** — uma pergunta sobre o estado do mundo ou do agente ("vejo o inimigo?", "minha vida é maior que 50%?", "a distância é menor que 10 m?"). O teste pode ser binário (sim/não) ou de múltiplas vias (por exemplo, um teste sobre a distância que se ramifica em "perto / média / longe").

**Ramos.** Cada ramo que sai de um nó de decisão corresponde a um **resultado possível do teste**. Seguir um ramo significa ter obtido aquela resposta e prosseguir para o próximo nó abaixo dele.

**Folhas (nós terminais).** As folhas são os **resultados finais** da árvore — no caso de jogos, as **ações** a executar ("atacar", "recarregar", "fugir", "procurar cobertura"). Ao chegar a uma folha, a decisão está tomada.

**Avaliar** uma árvore de decisão consiste em partir da raiz e descer, nó a nó: em cada nó de decisão, executa-se o teste, segue-se o ramo correspondente à resposta e repete-se, até alcançar uma folha. O caminho da raiz até a folha é uma sequência de condições que, juntas, justificam a ação escolhida. Note a diferença essencial em relação à FSM: **não há transições nem estados persistentes**; cada avaliação é independente e recomeça da raiz.

[DIAGRAMA]
Título: Estrutura de uma árvore de decisão
Objetivo pedagógico: Fixar os três elementos (nó de decisão, ramo, folha) e o processo de descida da raiz até a ação.
Descrição detalhada: Árvore desenhada de cima para baixo. Raiz (nó de decisão): "Vejo o inimigo?". Dois ramos: "Não" leva a uma folha "Patrulhar"; "Sim" leva a outro nó de decisão "Minha vida > 30%?". Deste, o ramo "Não" leva à folha "Fugir"; o ramo "Sim" leva a um nó "Distância < 2 m?". Deste, "Sim" leva à folha "Atacar corpo a corpo"; "Não" leva à folha "Atirar". Os nós de decisão desenhados como losangos (ou retângulos), as folhas como retângulos arredondados/ovais com cor distinta, e os ramos rotulados com as respostas (Sim/Não). Uma seta lateral ilustra um "caminho de avaliação" destacado (por exemplo, Sim → Sim → Não → Atirar).
Elementos obrigatórios: nó raiz; nós de decisão internos com testes; ramos rotulados com respostas; folhas de ação com cor distinta; um caminho de avaliação destacado da raiz à folha.
[/DIAGRAMA]

### 5.2.1 Árvore de decisão versus árvore de comportamento (distinção essencial)

Esta é a subseção mais importante do capítulo, e a razão pela qual ele existe como ponte. No jargão dos jogos, "árvore de decisão" e "árvore de comportamento" (*Behavior Tree*) são frequentemente confundidas — inclusive porque a ementa da disciplina lista "árvores de decisão", e a resposta editorial da apostila foi atender esse item **principalmente** pela árvore de comportamento (Capítulo 6). É crucial, portanto, que o aluno saia deste capítulo com a distinção cristalina.

As duas compartilham a **forma** (uma estrutura em árvore, com nós que se ramificam até folhas), mas diferem profundamente no **propósito** e no **funcionamento**:

| Aspecto | Árvore de **Decisão** | Árvore de **Comportamento** (Cap. 6) |
|---|---|---|
| **Nós internos representam** | Testes/condições (perguntas sim/não) | Formas de *controlar o fluxo* de tarefas (sequência, seleção, paralelo) |
| **Folhas representam** | A ação escolhida (o resultado da classificação) | Tarefas executáveis (ações e condições) |
| **O que a avaliação produz** | *Uma* decisão instantânea (classifica situação → ação) | *Execução* de comportamento ao longo do tempo, com estados de retorno |
| **Noção de tempo / duração** | Ausente: decide num instante, recomeça do zero | Central: tarefas podem estar *em execução* (running) por vários quadros |
| **Estados de retorno dos nós** | Não há (chega-se a uma folha e pronto) | Sucesso / Falha / Em execução, propagados árvore acima |
| **Objetivo típico** | Selecionar rapidamente uma ação por condições | Orquestrar sequências e alternativas de comportamento complexo, reutilizável |

**Análise interpretativa.** A distância entre as duas é a mesma que separa "escolher uma ação" de "orquestrar um comportamento ao longo do tempo". A árvore de decisão é um **classificador**: entra a situação, sai *uma* ação, e acabou. A árvore de comportamento é um **executor**: seus nós internos não perguntam "qual é a situação?", mas comandam "faça isto; se der certo, faça aquilo; se falhar, tente esta alternativa", e acompanham cada tarefa ao longo de vários quadros por meio dos estados de retorno *sucesso/falha/em execução*. Em outras palavras: a árvore de decisão descreve **como decidir**; a árvore de comportamento descreve **como agir de forma estruturada**. Guardar essa diferença agora tornará o Capítulo 6 muito mais claro.

> **Erro Comum**
> Tratar "árvore de decisão" e "árvore de comportamento" como sinônimos porque ambas são "árvores". São arquiteturas distintas, com propósitos distintos. A confusão é agravada pelo fato de que uma árvore de comportamento *pode conter* nós de condição que lembram os testes de uma árvore de decisão — mas o que caracteriza a árvore de comportamento são seus **nós compostos de controle de fluxo** (sequência, seletor, paralelo) e seus **estados de retorno**, ausentes na árvore de decisão clássica. Volte a este quadro comparativo quando estudar o Capítulo 6.

---

## 5.3 Funcionamento

O funcionamento de uma árvore de decisão *escrita à mão* é direto e já foi antecipado: avalia-se da raiz às folhas. Vale, porém, aprofundar dois pontos que afetam desempenho e qualidade da decisão — a ordenação dos testes e a profundidade — e registrar, brevemente, a variante *aprendida*.

### 5.3.1 Avaliação, ordenação de testes e profundidade

A eficiência de uma árvore de decisão depende fortemente de **quais testes ficam perto da raiz**. Como a avaliação abandona ramos inteiros ao seguir uma resposta, colocar perto da raiz os testes mais **decisivos** e mais **baratos** faz a árvore chegar à ação certa com menos perguntas. Um teste caro (por exemplo, um cálculo de linha de visão) posicionado no topo será executado em *toda* avaliação; posicionado mais abaixo, só é executado quando os testes anteriores já restringiram a situação. Da mesma forma, um teste que "separa bem" as situações (que descarta muitas ações de uma vez) merece estar no topo.

A **profundidade** da árvore — o número de testes no caminho mais longo — determina o custo do pior caso e a legibilidade. Árvores rasas são rápidas e fáceis de ler, mas expressam decisões simples; árvores muito profundas expressam decisões refinadas, mas ficam difíceis de manter e podem repetir subárvores idênticas em ramos diferentes (uma redundância análoga à que vimos na FSM). Equilibrar profundidade e clareza é a arte de projetar boas árvores de decisão.

> **Boa Prática**
> Ao ordenar os testes de uma árvore de decisão feita à mão, aplique dois critérios combinados: coloque no topo os testes que (1) são **mais determinantes** para a ação (separam o maior número de casos) e (2) são **mais baratos** de avaliar. Testes caros e pouco decisivos devem ficar o mais fundo possível, para serem evitados na maioria das avaliações. Esse princípio — testar cedo o que é barato e decisivo — reduz o custo médio da árvore sem alterar seu resultado.

### 5.3.2 Árvores de decisão aprendidas (breve nota sobre ID3/entropia)

Há uma vertente inteiramente diferente, oriunda do aprendizado de máquina, em que a árvore **não é escrita por um humano, mas construída automaticamente a partir de dados**. Algoritmos como o **ID3** (e seus sucessores) recebem um conjunto de exemplos rotulados (situações e a ação/classe "correta" de cada uma) e *aprendem* a árvore que melhor separa esses exemplos. A ideia central é escolher, para cada nó, o teste que produz a maior **redução de incerteza** — medida por uma grandeza chamada **entropia** (o "ganho de informação"): o teste que melhor "organiza" os exemplos em grupos mais homogêneos vai para o topo.

Registramos essa vertente apenas para contexto, por dois motivos. Primeiro, para que o aluno não confunda a **árvore de decisão escrita à mão** (o uso típico em jogos, tema deste capítulo) com a **árvore de decisão aprendida** (uma técnica de aprendizado supervisionado). Segundo, porque a ideia de "aprender uma árvore a partir de dados" reaparece na Parte VI, quando tratarmos de IA que aprende. Para os fins da Parte II — tomada de decisão baseada em **regras** —, o que importa é a árvore *autoral*, projetada pelo designer.

> **Atenção**
> Não confunda os dois usos da mesma expressão. A "árvore de decisão" da IA de jogos deste capítulo é uma estrutura de **regras escritas por um humano**, previsível e controlável — coerente com os critérios da Parte I. A "árvore de decisão" do aprendizado de máquina é um **modelo estatístico induzido de dados**, e traz consigo as dificuldades de controle e depuração já discutidas para técnicas aprendidas. São a mesma estrutura de dados a serviço de filosofias opostas.

---

## 5.4 Exemplos: seleção de ação de um NPC

Retomemos, de forma condensada, o mesmo inimigo dos capítulos anteriores — agora encarando a *escolha instantânea de ação* como uma árvore de decisão. A árvore da figura da seção 5.2 já expressa o essencial: parte de "Vejo o inimigo?" e desce por testes de vida e distância até uma folha de ação (patrulhar, fugir, atacar corpo a corpo ou atirar).

Um segundo exemplo, típico de jogos de esporte ou de estratégia, é a **decisão de passe** de um jogador controlado pela IA no futebol: "Estou sob pressão? → Sim: há companheiro desmarcado? → Sim: passar; → Não: chutar para frente. → Não: tenho ângulo de chute a gol? → Sim: chutar; → Não: conduzir a bola." Trata-se de uma decisão pontual, recalculada a cada oportunidade, sem "modos" persistentes — o caso de uso natural da árvore de decisão.

**Análise interpretativa.** Em ambos os exemplos, a árvore de decisão brilha exatamente onde a FSM seria desajeitada: não há comportamento contínuo a manter, apenas uma **escolha condicional recalculada**. E note como a árvore se combina naturalmente com as arquiteturas anteriores: nada impede que a folha "Atirar" da nossa árvore seja, na verdade, a *entrada* no estado *Atacar* de uma FSM. Na prática, árvores de decisão frequentemente vivem **dentro** de estados de uma FSM/HFSM, cuidando das microdecisões locais enquanto a máquina de estados cuida dos modos de longo prazo.

> **Na Indústria**
> Árvores de decisão feitas à mão são muito usadas para decisões **rápidas e localizadas**: qual animação de reação tocar, qual linha de diálogo dizer, qual alvo priorizar, qual item usar. Raramente uma árvore de decisão sozinha governa *todo* o comportamento de um NPC complexo — para isso, a indústria preferiu as árvores de comportamento. A árvore de decisão é mais uma "ferramenta de bolso" para escolhas pontuais do que a espinha dorsal de uma IA inteira.

---

## 5.5 Vantagens e limitações (por que jogos migraram para árvores de comportamento)

**Vantagens.** A árvore de decisão é **extremamente simples e intuitiva**, mapeando diretamente o raciocínio "se... então...". É **barata** de avaliar (uma descida por poucos testes) e **transparente**: o caminho da raiz à folha *explica* a decisão, o que a torna fácil de depurar ("ele atirou porque via o inimigo, tinha vida e estava longe"). É **visualizável e editável** por designers. E é **modular no pequeno**: subárvores podem ser raciocinadas isoladamente.

**Limitações.** As fraquezas explicam por que a árvore de decisão, sozinha, não se tornou a espinha dorsal da IA de decisão em jogos:

Primeiro, ela **não tem noção de tempo nem de duração**. Cada avaliação é instantânea e recomeça do zero; não há como expressar "faça isto por três segundos" ou "continue a tarefa iniciada no quadro anterior" sem recorrer a estado externo — exatamente a memória que a FSM oferecia e que a árvore de decisão pura não tem.

Segundo, ela **não sequencia comportamentos** de forma nativa. Dizer "vá até a porta, *depois* abra-a, *depois* atravesse" não é o que uma árvore de decisão faz — ela escolhe *uma* ação, não uma *sequência* de ações com dependência de sucesso entre elas.

Terceiro, ela **reutiliza mal**. Subárvores comuns tendem a ser copiadas em vários ramos (a mesma redundância da FSM plana), e não há um mecanismo limpo de compor comportamentos a partir de blocos reaproveitáveis.

Quarto, ela **cresce mal em profundidade**: decisões refinadas exigem árvores profundas e repetitivas, difíceis de manter.

É a soma dessas limitações — ausência de tempo, de sequenciamento, de reutilização e de composição — que levou a indústria, a partir de meados dos anos 2000, a adotar as **árvores de comportamento**. Elas preservam a forma de árvore e a legibilidade, mas acrescentam justamente o que faltava: **nós de controle de fluxo** que sequenciam e selecionam comportamentos, **estados de retorno** que dão noção de duração e sucesso/falha, e **modularidade** que permite montar comportamentos complexos a partir de blocos reutilizáveis. O Capítulo 6 desenvolve tudo isso — e agora o leitor sabe *exatamente qual problema* essa arquitetura veio resolver.

> **Erro Comum**
> Tentar expressar **sequências de ações com dependência** ("faça A; só se A der certo, faça B") como uma árvore de decisão. A árvore de decisão escolhe *uma* ação; ela não foi feita para orquestrar passos sucessivos que dependem do resultado uns dos outros. Forçar isso leva a árvores contorcidas e a variáveis de estado espalhadas pelo código. Esse é precisamente o território das árvores de comportamento (Capítulo 6).

---

## 5.6 Jogos conhecidos e aplicações

Por ser um bloco de construção pequeno e ubíquo, a árvore de decisão raramente é anunciada como "a IA" de um jogo — ela costuma ser um componente interno. Ainda assim, suas aplicações típicas são reconhecíveis:

**Seleção de ações táticas e de diálogo.** Escolher qual reação, fala ou animação disparar diante de uma situação é um caso clássico de árvore de decisão embutida, presente em incontáveis jogos de ação, RPGs e aventuras.

**Decisões esportivas e de estratégia.** Jogos de esporte e de estratégia usam árvores de decisão para escolhas pontuais (passar/chutar/driblar; atacar/defender/recuar), muitas vezes dentro de uma arquitetura maior.

**Variante aprendida em bastidores.** Árvores de decisão *aprendidas* aparecem em usos de bastidor — por exemplo, na análise de dados de jogadores ou em sistemas de balanceamento —, coerentes com o papel restrito do aprendizado na prática comercial descrito na Parte I. No jogo em si, a variante autoral predomina.

A atribuição de "árvore de decisão" a um título específico é, na maioria dos casos, **análise técnica fundamentada**, dada a natureza interna e pouco documentada desse componente.

> **Curiosidade**
> A árvore de decisão é tão fundamental que aparece disfarçada em ferramentas que não a chamam por esse nome: sistemas de *diálogo ramificado* de RPGs, editores de *quests* condicionais e sistemas de regras de eventos são, estruturalmente, árvores de decisão. Sempre que você vir um fluxograma de "se o jogador fez X, então Y; senão Z", está diante de uma árvore de decisão, tenha ela esse rótulo ou não.

---

## 5.7 Ferramentas na Unity e de terceiros

Coerente com o caráter de apoio deste capítulo, a nota sobre ferramentas é breve.

A Unity **não** possui uma ferramenta oficial dedicada exclusivamente a "árvores de decisão" no sentido deste capítulo — o que faz sentido, pois a lógica condicional simples é facilmente escrita em C# ou montada no **Visual Scripting**, cujos grafos expressam naturalmente cadeias de decisão. Para comportamento estruturado de agentes, a direção oficial da engine é o pacote **Unity Behavior**, voltado a *árvores de comportamento* (Capítulo 6), que incorporam nós de condição capazes de cumprir o papel dos testes de uma árvore de decisão.

Entre terceiros, as ferramentas de IA já citadas — **NodeCanvas**, **Behavior Designer** — oferecem, além de árvores de comportamento e FSMs, meios de montar lógica condicional visual. Bibliotecas de aprendizado de máquina (para a variante *aprendida*) fogem ao escopo da Parte II e pertencem ao ferramental da Parte VI.

O ponto a reter é que, na prática de jogos, a árvore de decisão *autoral* raramente exige uma ferramenta própria: ela vive dentro de código, de grafos de visual scripting ou, mais frequentemente, **embutida como nós de condição dentro de uma árvore de comportamento** — o que nos conduz diretamente ao próximo capítulo.

> **Boa Prática**
> Se o seu comportamento é uma **decisão pontual** por poucas condições, uma árvore de decisão em código ou em visual scripting é suficiente e não justifica adotar uma ferramenta pesada. Reserve as ferramentas de árvore de comportamento para quando precisar **sequenciar, priorizar e reutilizar** comportamentos ao longo do tempo. Escolher a ferramenta proporcional ao problema é, também, um critério de boa engenharia.

---

## 5.8 Resumo, Exercícios de fixação e Referências

### Resumo do Capítulo 5

Este capítulo de **apoio** apresentou a **árvore de decisão** como uma terceira forma de organizar a seleção de ação, distinta das arquiteturas baseadas em estado dos Capítulos 3 e 4. Vimos que ela ataca um problema específico — **escolher uma ação, agora, a partir de um conjunto de condições** — organizando testes encadeados em uma estrutura de árvore de **nós de decisão**, **ramos** e **folhas** de ação, avaliada da raiz até a folha, sem estados persistentes nem memória de modo. Dedicamos atenção especial à **distinção essencial entre árvore de decisão e árvore de comportamento**: ambas têm a forma de árvore, mas a primeira é um *classificador* que produz uma decisão instantânea, enquanto a segunda (Capítulo 6) é um *executor* que orquestra comportamento ao longo do tempo, com nós de controle de fluxo e estados de retorno. Discutimos o funcionamento — a importância de **ordenar os testes** (baratos e decisivos no topo) e o efeito da **profundidade** — e registramos, para contexto, a variante **aprendida** (ID3/entropia), pertencente ao território do aprendizado de máquina da Parte VI. Pesamos vantagens (simplicidade, baixo custo, transparência, autoria visual) e limitações — ausência de tempo, de sequenciamento, de reutilização e de composição — que explicam **por que a indústria migrou para as árvores de comportamento**. Encerramos posicionando a árvore de decisão como um bloco de construção ubíquo, frequentemente **embutido dentro** de outras arquiteturas, e preparamos, assim, a chegada ao capítulo de ênfase da Parte.

### Exercícios de fixação

1. Qual problema a **árvore de decisão** resolve, e como ele difere do problema que a FSM resolve? Dê um exemplo de comportamento mais bem modelado por árvore de decisão e outro mais bem modelado por FSM.
2. Defina **nó de decisão**, **ramo** e **folha**. Descreva o processo de **avaliação** de uma árvore de decisão, da raiz à ação.
3. Explique, com um quadro ou lista, **três diferenças** entre uma árvore de decisão e uma árvore de comportamento. Por que confundi-las é um erro comum?
4. Por que a **ordem dos testes** afeta o custo de avaliação de uma árvore de decisão, mas não o seu resultado? Que critérios devem guiar a colocação de um teste perto da raiz?
5. O que distingue uma árvore de decisão **escrita à mão** de uma árvore de decisão **aprendida** (ID3)? Por que, na Parte II, nos interessa sobretudo a primeira?
6. Cite e explique **três limitações** da árvore de decisão que motivaram a adoção das árvores de comportamento pela indústria.
7. Projete uma árvore de decisão (diagrama ou texto) para um NPC comerciante que decide se **abaixa o preço**, **mantém o preço** ou **recusa a venda**, com base em pelo menos três condições (reputação do jogador, estoque, relação de oferta e procura).
8. Explique por que árvores de decisão frequentemente vivem **dentro** de estados de uma FSM ou de nós de uma árvore de comportamento, em vez de governarem sozinhas todo o comportamento de um NPC.
9. Dê um exemplo de comportamento que **não** pode ser bem expresso por uma árvore de decisão pura (uma sequência de ações com dependência de sucesso) e explique por que a árvore de comportamento é a arquitetura adequada para ele.
10. Um estudante afirma: "árvore de decisão e árvore de comportamento são a mesma coisa, só muda o nome". Refute essa afirmação com precisão técnica, usando os conceitos de *estados de retorno* e *nós de controle de fluxo*.

### Referências

MILLINGTON, Ian. *AI for Games.* 3. ed. Boca Raton: CRC Press, 2019. (Referência para árvores de decisão em jogos e sua relação com máquinas de estado e árvores de comportamento.)

RUSSELL, Stuart J.; NORVIG, Peter. *Inteligência Artificial.* 3. ed. Rio de Janeiro: Elsevier, 2013. (Fundamentos das árvores de decisão como classificadores e da variante aprendida — ganho de informação e entropia.)

RABIN, Steve (org.). *Game AI Pro 3: Collected Wisdom of Game AI Professionals.* Boca Raton: A K Peters/CRC Press, 2017. (Uso de lógica de decisão e sua integração com outras arquiteturas na prática da indústria.)

QUINLAN, J. Ross. Induction of Decision Trees. *Machine Learning*, v. 1, n. 1, p. 81–106, 1986. (Referência conceitual para o algoritmo ID3 e a construção de árvores de decisão a partir de dados.)

UNITY TECHNOLOGIES. *Documentação oficial da Unity: Visual Scripting e pacote Unity Behavior.* (Materialização de lógica condicional e de comportamento estruturado na engine.)

> **Nota sobre fontes.** O uso de árvores de decisão como componente interno de NPCs em jogos comerciais é, na maioria dos casos, **análise técnica fundamentada** na estrutura observável do comportamento, dada a natureza pouco documentada desse componente. A descrição do ID3 e do ganho de informação segue a literatura consagrada de aprendizado de máquina.
