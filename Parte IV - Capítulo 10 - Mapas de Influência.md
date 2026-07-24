# Capítulo 10 — Mapas de Influência

## Introdução

Toda a Parte III respondeu a uma única pergunta: **como chegar** de um ponto a outro do mapa. Grafos (Capítulo 7), A\* (Capítulo 8) e as otimizações de busca (Capítulo 9) formam, juntos, uma máquina poderosa de traçar rotas — dado um destino, ela encontra o melhor caminho até ele. Mas repare no que essa máquina **pressupõe**: que alguém já decidiu **para onde ir**. O pathfinding recebe um destino e trabalha; ele nunca pergunta se aquele é um bom destino. E, para uma quantidade enorme de decisões que uma IA de jogo precisa tomar, essa é justamente a pergunta que importa. Um atirador não precisa saber apenas como chegar a uma cobertura — precisa saber **qual** cobertura escolher. Um general de RTS não precisa apenas mover seu exército — precisa decidir **em que direção** avançar. Um NPC ferido não quer o caminho mais curto: quer o caminho para **longe do perigo**.

Este capítulo trata da técnica clássica que responde a essa segunda pergunta — não "como chegar?", mas "**onde devo ir?**". O **mapa de influência** (*influence map*) é uma das ideias mais elegantes e duradouras da IA de jogos: em vez de raciocinar sobre posições isoladas, ele transforma o **espaço inteiro** do jogo em um **campo de valores** — um mapa de calor invisível que, para cada região, responde perguntas como "quão perigoso é aqui?", "quem controla este território?", "quão exposto eu fico se ficar parado neste ponto?". Com esse campo em mãos, decisões espaciais que seriam extremamente difíceis de programar caso a caso reduzem-se a algo simples: **olhar o mapa e escolher a célula com o melhor valor**.

Fiel à estrutura da apostila, partimos do **problema** de design (uma IA que precisa decidir *onde* agir), construímos o **conceito** de campo escalar sobre o mapa, detalhamos o **funcionamento** (fontes, propagação, decaimento, combinação de camadas e atualização em tempo real), aterrissamos em **exemplos** concretos e nos **jogos** que os popularizaram, pesamos **vantagens e limitações**, e fechamos com as **ferramentas** — incluindo a comparação esclarecedora com o **EQS** da Unreal e a discussão de por que a Unity não oferece um sistema nativo dedicado. Ao final, o leitor terá no repertório a ferramenta central do **raciocínio espacial e tático**, e verá que ela é a ponte natural entre a Parte III (movimento) e a Parte V (decisão adversarial): o mapa de influência é, no fundo, uma **função de avaliação espacial**, e essa ideia reaparecerá com força no Minimax do Capítulo 11.

> 🕰️ **Contexto Histórico**
> A ideia de sobrepor um "campo de valores" ao terreno para orientar decisões é antiga e tem raízes fora dos jogos — na análise militar e na cartografia tática, onde mapas de controle territorial e de ameaça são usados há muito tempo. Nos jogos digitais, os mapas de influência ganharam corpo com os jogos de estratégia em tempo real dos anos 1990, quando a IA precisava, pela primeira vez, avaliar mapas grandes e decidir onde atacar, defender e expandir. O termo consolidou-se na literatura técnica com autores como **Steve Rabin** (série *Game AI Pro*) e **Ian Millington** (*AI for Games*), que sistematizaram o método e o apresentaram como a ferramenta padrão de raciocínio tático — o complemento espacial do pathfinding. É um daqueles conceitos que, uma vez compreendido, o estudante passa a reconhecer em praticamente todo jogo de estratégia que já jogou.

---

## 10.1 O problema: decidir *onde* agir

Comecemos, como sempre, por situações concretas de desenvolvimento — problemas reais que um programador de IA enfrenta e para os quais o pathfinding, sozinho, **não tem resposta**.

**Situação 1 — o atirador que precisa de cobertura.** Em um jogo de tiro tático, um inimigo sob fogo precisa se proteger. Há seis obstáculos por perto que poderiam servir de cobertura. Qual escolher? A mais próxima pode estar exposta ao jogador por outro ângulo. A mais protegida pode estar do lado errado do combate, longe demais dos aliados. A "melhor" cobertura depende de onde está o inimigo, de onde estão os aliados, de quais posições dão linha de tiro e quais estão protegidas. O pathfinding sabe **como chegar** a qualquer uma das seis — mas quem decide **qual das seis**?

**Situação 2 — o exército que precisa avançar.** Em um RTS, a IA controla um exército e precisa decidir para onde marchar. Avançar pela direita a leva a um ponto de recursos valioso, mas próximo à base inimiga. Avançar pelo centro é seguro, mas inútil. Avançar pela esquerda flanqueia o inimigo, mas expõe a retaguarda. Essa é uma decisão sobre **regiões do mapa**, não sobre um único destino — e envolve avaliar, simultaneamente, oportunidade, risco e controle territorial ao longo de todo o terreno.

**Situação 3 — o NPC que precisa fugir.** Um personagem ferido quer recuar. O caminho mais curto para trás pode levá-lo direto para uma emboscada. O que ele quer não é o destino mais próximo, e sim a **direção mais segura** — aquela que maximiza a distância a todas as ameaças ao mesmo tempo. Não existe um "destino" óbvio a informar ao pathfinding; o destino precisa ser **descoberto** a partir da configuração espacial do perigo.

O que essas três situações têm em comum? Todas são **decisões espaciais**: a resposta não é uma ação abstrata ("atacar", "fugir"), mas uma **posição** ou uma **região** do mapa. E todas dependem não de um único fator, mas da **combinação** de múltiplos fatores distribuídos pelo espaço — a posição de aliados e inimigos, a geometria das coberturas, o valor dos recursos, a proximidade do perigo. Programar cada uma dessas decisões "à mão", com uma cascata de `if`s comparando distâncias e ângulos par a par, é possível para casos pequenos, mas **não escala**: cada nova fonte de consideração multiplica a complexidade do código, e o resultado é frágil, difícil de ajustar e específico demais para reaproveitar.

> 🎮 **Na Prática**
> O sintoma clássico de que um problema pede mapa de influência é a frase "a IA precisa escolher a melhor posição considerando *várias coisas ao mesmo tempo*". Se você se pega escrevendo laços aninhados que, para cada candidato de posição, comparam distância ao inimigo, distância ao aliado, exposição, valor do terreno e mais meia dúzia de fatores — e depois precisa **ponderar** tudo isso num único número —, você está reimplementando, de forma ad hoc, aquilo que o mapa de influência resolve de maneira estruturada e reaproveitável. Reconhecer o padrão cedo evita meses de código tático emaranhado.

A raiz do problema é conceitual: o pathfinding raciocina sobre **conexões** (como os lugares se ligam), mas essas decisões exigem raciocinar sobre **qualidades do espaço** (o que cada lugar *vale*, taticamente). São dois tipos diferentes de conhecimento espacial. O Capítulo 7 nos deu a representação das **conexões** — o grafo navegável. Falta uma representação das **qualidades** — e é exatamente isso que o mapa de influência fornece.

[IMAGEM NECESSÁRIA]
Título: Duas perguntas espaciais — "como chegar?" versus "onde ir?"
Objetivo didático: Fixar, logo na abertura do capítulo, a distinção conceitual central entre pathfinding (Parte III) e mapa de influência (Capítulo 10), mostrando que são ferramentas complementares que respondem a perguntas diferentes.
Descrição: Painel dividido em dois. À esquerda, o mesmo mapa de jogo com um NPC, um destino marcado e uma rota traçada de um ao outro, rotulado "PATHFINDING — como chegar?". À direita, o mesmo mapa sem destino definido, mas coberto por um mapa de calor (zonas verdes seguras, zonas vermelhas perigosas) e o NPC "olhando" para a zona verde mais próxima, rotulado "MAPA DE INFLUÊNCIA — onde ir?". Uma seta ligando os dois painéis indica que, na prática, o mapa de influência escolhe o destino que o pathfinding depois alcança.
Tipo: Ilustração conceitual (montagem sobre screenshots de jogo ou arte esquemática).
Como produzir: Capturar ou desenhar um mesmo cenário de jogo em dois estados; no segundo, sobrepor um gradiente de cores (heatmap) representando segurança/perigo. Ferramenta sugerida: editor de imagem com camadas (por exemplo, sobreposição de gradiente semitransparente sobre screenshot).
Legenda sugerida: "Pathfinding e mapa de influência são complementares: um encontra a rota, o outro escolhe o destino que vale a pena alcançar."
[/IMAGEM NECESSÁRIA]

---

## 10.2 Fundamentos: campos escalares sobre o mapa

A ideia central do mapa de influência pode ser enunciada em uma frase: **atribuir a cada região do mapa um número que resume alguma qualidade tática daquela região**. Esse "número por região" é o que a matemática chama de **campo escalar** — uma função que associa, a cada ponto do espaço, um único valor (um escalar). Onde o pathfinding vê o mapa como um **grafo de conexões**, o mapa de influência vê o mapa como uma **paisagem de valores**: um terreno com picos e vales, onde os picos são as regiões de alto valor (muito perigo, muito controle, muita oportunidade) e os vales são as de baixo valor.

O nome "mapa de influência" vem do uso original mais comum: medir a **influência militar** de cada lado sobre o território. Cada unidade "irradia" influência ao seu redor — forte perto dela, enfraquecendo com a distância. Somando a influência de todas as unidades aliadas e subtraindo a de todas as inimigas, obtém-se, para cada região, um valor: positivo onde os aliados dominam, negativo onde os inimigos dominam, próximo de zero na **fronteira disputada**. Esse campo responde, de relance, a perguntas que seriam custosas de calcular caso a caso: onde está a linha de frente? Onde estou seguro? Onde o inimigo está vulnerável? Onde vale a pena atacar?

Mas "influência militar" é apenas **um** uso. O poder da técnica está em sua **generalidade**: o mesmo mecanismo — atribuir valores ao espaço e deixá-los se espalhar — serve para representar **qualquer** qualidade espacial. Um campo pode medir perigo, outro pode medir visibilidade, outro pode medir proximidade de recursos, outro pode medir o "cheiro" deixado pelo jogador que passou por ali. Cada um desses é uma **camada** (ou **canal**) de influência, e o raciocínio tático sofisticado nasce de **combinar** várias camadas — assunto da Seção 10.2.2.

> ⚠️ **Atenção**
> É comum confundir o mapa de influência com o mapa de navegação (grade/NavMesh do Capítulo 7). Eles frequentemente **compartilham a mesma grade** como suporte, mas guardam informações de natureza distinta. A grade de navegação responde "esta célula é caminhável e a quais outras se conecta?" — informação **topológica**, sobre movimento. O mapa de influência responde "quanto vale, taticamente, estar nesta célula?" — informação **avaliativa**, sobre decisão. Uma célula perfeitamente caminhável (ótima para a navegação) pode ter valor tático péssimo (bem no meio da linha de fogo inimiga). Manter clara essa separação — **onde posso ir** versus **onde vale a pena ir** — é a chave para não misturar os dois sistemas.

### 10.2.1 Fontes de influência, propagação e decaimento

Três conceitos sustentam qualquer mapa de influência: **fonte**, **propagação** e **decaimento**. Vamos a cada um.

**Fonte de influência.** Uma fonte é qualquer entidade do jogo que "emite" influência para o campo: uma unidade militar, uma torre, um recurso, uma explosão, o próprio jogador. Cada fonte tem uma **posição** (onde ela está no mapa) e uma **intensidade** (quão forte é a influência que emite). Um tanque emite mais influência militar que um soldado; uma explosão recente emite mais "perigo" que uma antiga. A intensidade é o valor que a fonte deposita na célula onde ela se encontra — o **pico** local do campo.

**Propagação.** Se cada fonte afetasse apenas a própria célula, o mapa seria um punhado de pontos isolados, inútil para raciocinar sobre regiões. O que torna o campo contínuo e informativo é a **propagação** (também chamada *spreading* ou *bleeding*): a influência de cada fonte **se espalha** para as células vizinhas, e destas para as vizinhas seguintes, cobrindo uma área ao redor da fonte. É a propagação que transforma pontos isolados em **zonas** — é ela que faz surgir a "área de controle" de uma unidade, o "raio de perigo" de uma ameaça, o "território" de um jogador.

**Decaimento.** A propagação não pode ser uniforme, ou toda fonte dominaria o mapa inteiro por igual. A influência deve **enfraquecer com a distância** à fonte — forte no centro, cada vez mais fraca nas bordas, até se anular. Esse enfraquecimento é o **decaimento** (*decay* ou *falloff*). Ele modela uma intuição simples e quase universal: coisas próximas importam mais que coisas distantes. Uma unidade inimiga a dois passos é uma ameaça imediata; a mesma unidade a cinquenta passos é quase irrelevante. O decaimento captura essa noção de "alcance" da influência.

Existem duas grandes formas de calcular o valor de uma célula a partir das fontes, e vale conhecer ambas:

A primeira é a **fórmula direta**: para cada célula, soma-se a contribuição de cada fonte, calculada aplicando o decaimento à intensidade da fonte em função da distância. Uma forma comum de decaimento é **linear** (a influência cai proporcionalmente à distância até zerar num raio máximo); outra é **exponencial** (a influência cai por um fator a cada passo de distância, `valor = intensidade × decaimento^distância`, com decaimento entre 0 e 1). A fórmula direta é precisa, mas custosa: para cada célula, é preciso considerar cada fonte.

A segunda é a **propagação iterativa** (por vizinhança): parte-se das células-fonte com sua intensidade máxima e, em passos sucessivos, cada célula "vaza" para as vizinhas uma fração do próprio valor (multiplicada pelo fator de decaimento), repetindo até o campo estabilizar ou por um número fixo de iterações. Essa forma é mais barata e naturalmente respeita **obstáculos** — se a propagação segue as conexões da grade navegável, a influência **contorna paredes** em vez de atravessá-las, produzindo um campo que reflete a **distância real de caminhada**, não a distância em linha reta. Essa é uma vantagem tática importante: o perigo de um inimigo do outro lado de um muro é (corretamente) baixo, mesmo que ele esteja perto em linha reta.

> ⚠️ **Atenção**
> Decaimento em linha reta versus decaimento pela grade navegável é uma distinção que costuma confundir. Se a influência decai pela **distância euclidiana** (linha reta), ela "atravessa" paredes: um inimigo do outro lado de um muro parece perigoso, embora não possa te alcançar. Se decai pela **distância de caminhada** (propagando célula a célula pelas conexões navegáveis, como no Capítulo 7), a influência **contorna** obstáculos e reflete a ameaça real. Para raciocínio tático fiel, quase sempre se quer o segundo comportamento — e é por isso que a propagação iterativa sobre a grade navegável é tão popular: ela herda "de graça" a topologia do mundo.

O pseudocódigo a seguir ajuda a fixar o mecanismo da propagação iterativa:

```
// Propagação iterativa de uma camada de influência sobre uma grade navegável.
// Executada a cada atualização do mapa (ver Seção 10.3).

para cada célula c da grade:
    novaInfluência[c] = 0

// 1. Depositar a intensidade das fontes
para cada fonte f:
    novaInfluência[célula(f)] += intensidade(f)

// 2. Espalhar por N iterações, enfraquecendo com o decaimento
repetir N vezes:
    para cada célula c navegável:
        máximoVizinho = maior valor de influência entre os vizinhos navegáveis de c
        // cada célula recebe uma fração decaída do vizinho mais influente
        candidato = máximoVizinho * decaimento        // decaimento entre 0 e 1
        novaInfluência[c] = max(novaInfluência[c], candidato)

influência = novaInfluência
```

Note que a propagação **respeita as conexões navegáveis**: só se espalha entre células que o Capítulo 7 consideraria conectadas. É assim que o campo "aprende", sem custo extra, a contornar paredes.

[DIAGRAMA]
Título: Fonte, propagação e decaimento em uma grade
Objetivo pedagógico: Mostrar visualmente como uma única fonte gera um campo de influência que decai com a distância e contorna um obstáculo ao se propagar pela grade navegável.
Descrição detalhada: Uma grade retangular (por exemplo 9×9) vista de cima. No centro-esquerda, uma célula-fonte marcada com o valor máximo (por exemplo 100) e cor intensa. Ao redor dela, anéis concêntricos de células com valores decrescentes (por exemplo 100 → 70 → 49 → 34 …, ilustrando decaimento por fator 0,7), representados por cores cada vez mais claras — um mapa de calor. Inserir uma pequena parede (duas ou três células bloqueadas, em cinza-escuro) a leste da fonte; mostrar que a influência NÃO atravessa a parede: as células logo atrás dela têm valor baixo, e o valor "dá a volta" pela passagem livre, evidenciando a propagação pela grade navegável (distância de caminhada). Legenda destacando: "a influência decai com a distância de caminhada e contorna obstáculos".
Elementos obrigatórios: célula-fonte com valor máximo; anéis de decaimento com valores numéricos; escala de cores (heatmap); parede que bloqueia a propagação; influência contornando a parede.
[/DIAGRAMA]

### 10.2.2 Combinação de camadas

Um único campo raramente basta para uma decisão tática rica. A força real do método aparece quando se mantêm **várias camadas** de influência simultaneamente — cada uma medindo uma qualidade diferente — e se **combinam** essas camadas para produzir campos derivados, mais informativos. Cada camada é um mapa de influência independente, calculado com suas próprias fontes e seu próprio decaimento; a combinação é feita célula a célula, com operações aritméticas simples.

As camadas e combinações mais úteis na prática são:

**Influência (controle).** A camada básica: influência aliada como valores positivos, inimiga como negativos, somadas célula a célula. O resultado é um campo de **controle territorial** — positivo onde os aliados dominam, negativo onde os inimigos dominam. É o campo que responde "de quem é este pedaço de mapa?".

**Tensão.** Calculada como a **soma dos valores absolutos** das influências aliada e inimiga (`|aliada| + |inimiga|`). A tensão é alta onde **ambos** os lados têm presença forte — ou seja, onde há **conflito**. Um ponto de alta tensão é uma zona quente de batalha; um ponto de baixa tensão é uma região tranquila (dominada por um lado só ou vazia). A tensão localiza **onde a ação está acontecendo**.

**Vulnerabilidade (fronteira).** Calculada a partir do **controle**: as regiões onde o controle está próximo de **zero** mas a tensão é **alta** são a **linha de frente** — o território contestado, onde uma pequena vantagem decide quem avança. É onde reforços fazem mais diferença. Uma variação mede a vulnerabilidade **do inimigo**: regiões que o inimigo controla fracamente e que estão ao alcance dos aliados são alvos de ataque promissores.

**Ameaça.** Uma camada dedicada ao **perigo** para o agente: fontes são as unidades inimigas (e seus alcances de ataque, explosões, zonas de fogo), propagadas de modo a marcar as células onde ficar é arriscado. É a camada que o NPC ferido da Situação 3 consulta para fugir.

**Segurança.** Frequentemente definida como o **inverso** ou o **complemento** da ameaça: as células de menor ameaça são as mais seguras. Pode incorporar também a proximidade de aliados e de coberturas. É o campo que o agente maximiza quando quer se proteger.

A combinação dessas camadas é o que produz o raciocínio tático de fato. Um exemplo típico de "campo de decisão" para escolher **onde atacar** poderia ser: *procurar a célula que maximiza a vulnerabilidade inimiga (alvo fraco) e ao mesmo tempo minimiza a ameaça a mim (não é uma armadilha) e está próxima dos meus reforços (posso ser apoiado)*. Cada um desses três critérios é uma camada; o campo final é uma combinação ponderada delas, e a decisão vira, de novo, "escolher a célula de maior valor no campo combinado".

> 🎮 **Na Prática**
> A combinação de camadas costuma ser uma **soma ponderada**: `decisão = pesoA × camadaA + pesoB × camadaB − pesoC × camadaC …`, onde os pesos codificam a "personalidade" ou a "doutrina" da IA. Uma IA agressiva dá peso alto à vulnerabilidade inimiga e baixo à própria segurança; uma IA cautelosa faz o oposto. O belo desse arranjo é que **ajustar o comportamento tático da IA vira ajustar números**, não reescrever lógica — os designers podem afinar a agressividade sem tocar no código, exatamente o tipo de **controle de autoria** que valorizamos desde o Capítulo 1. Essa mesma ideia de "somar considerações ponderadas para escolher a melhor opção" é o coração da **IA de utilidade** que vimos no Capítulo 6: o mapa de influência é, em boa medida, IA de utilidade aplicada ao **espaço**.

[DIAGRAMA]
Título: Combinação de camadas de influência
Objetivo pedagógico: Mostrar como camadas independentes (influência aliada, influência inimiga) se combinam célula a célula para produzir campos derivados de significado tático (controle, tensão, fronteira).
Descrição detalhada: Uma pilha de grades sobrepostas, estilo "camadas do Photoshop", cada uma rotulada. Camada 1 — "Influência aliada": heatmap azul concentrado à esquerda do mapa. Camada 2 — "Influência inimiga": heatmap vermelho concentrado à direita. Abaixo, setas apontando para três grades-resultado: (a) "Controle = aliada − inimiga": azul à esquerda, vermelho à direita, faixa neutra no meio; (b) "Tensão = |aliada| + |inimiga|": faixa quente (amarela/laranja) exatamente na região central onde os dois se encontram; (c) "Fronteira": destaque fino na linha onde o controle ≈ 0 e a tensão é alta. Legenda: "camadas simples, combinadas por aritmética célula a célula, revelam controle territorial, zonas de conflito e a linha de frente".
Elementos obrigatórios: camada aliada (azul); camada inimiga (vermelho); campo de controle (subtração); campo de tensão (soma de absolutos); destaque da fronteira; indicação de que a operação é célula a célula.
[/DIAGRAMA]

---

## 10.3 Funcionamento e atualização em tempo real

Já sabemos **o que** é um mapa de influência e **de que** ele é feito. Falta o aspecto que mais determina sua viabilidade prática: **quando e como atualizá-lo**. Um mapa de influência não é calculado uma vez e esquecido — o jogo muda a cada instante (unidades se movem, morrem, nascem; ameaças surgem e somem), e o campo precisa refletir o estado atual para ser útil. Mas recalcular o campo inteiro, a cada quadro, para um mapa grande, é **caro demais**. A engenharia do mapa de influência é, em boa parte, a engenharia desse compromisso entre **atualidade** e **custo**.

O ciclo de funcionamento típico tem três passos, executados a cada atualização:

**1. Coletar as fontes.** Percorre-se o estado do jogo e registram-se as fontes ativas e suas intensidades e posições — quais unidades existem, onde estão, quão fortes são.

**2. Recalcular (ou atualizar) o campo.** Aplica-se a propagação com decaimento — pela fórmula direta ou, mais comumente, pela propagação iterativa da Seção 10.2.1 — obtendo o novo valor de cada célula.

**3. Combinar as camadas.** Se houver múltiplas camadas, calculam-se os campos derivados (controle, tensão, ameaça etc.) de que as decisões precisam.

O ponto crítico está no passo 2, e as técnicas para torná-lo viável são o verdadeiro ofício aqui:

**Atualização em baixa frequência.** A observação-chave é que o raciocínio tático **não precisa ser instantâneo**. Diferentemente da física ou da animação, que exigem 60 atualizações por segundo, uma decisão tática de "para onde avançar o exército" pode ser reavaliada a cada meio segundo ou a cada segundo sem prejuízo perceptível — o mundo tático não muda tão rápido assim. Por isso, mapas de influência são tipicamente atualizados em **baixa frequência** (algumas vezes por segundo, não a cada quadro), aliviando drasticamente o custo. Essa é a decisão de engenharia mais importante do capítulo.

**Atualização incremental.** Em vez de recalcular tudo, atualiza-se **apenas o que mudou**. Se uma unidade se moveu de uma célula para outra, subtrai-se sua contribuição antiga e soma-se a nova, propagando a correção apenas na vizinhança afetada. Regiões do mapa onde nada aconteceu não são recalculadas.

**Fatiamento no tempo (*time-slicing*).** Distribui-se o cálculo ao longo de **vários quadros**: atualiza-se um pedaço do mapa por quadro, de modo que o campo inteiro se renova a cada N quadros sem que nenhum quadro isolado carregue o custo total. É a mesma ideia de *time-slicing* aplicada ao pathfinding de muitos agentes (Capítulo 9), agora aplicada ao campo.

**Resolução reduzida.** Nem sempre o mapa de influência precisa da mesma resolução da grade de navegação. Um campo tático pode usar uma grade **mais grossa** (cada célula de influência cobrindo várias células de navegação), reduzindo o número de células a atualizar por um fator quadrático. A perda de precisão costuma ser aceitável — decisões táticas raramente dependem da célula exata, e sim da **região**.

> ✅ **Boa Prática**
> Case a **frequência de atualização** e a **resolução** do mapa de influência com a **velocidade e a granularidade das decisões** que ele alimenta. Um RTS que decide estratégia a cada segundo não ganha nada atualizando o campo 60 vezes por segundo — só desperdiça CPU. Um mapa de perigo para esquiva em um jogo de ação rápido pode precisar de atualização mais frequente, mas provavelmente cobre uma área pequena. A pergunta orientadora é sempre a mesma: *qual é a decisão mais rápida e mais fina que este campo precisa suportar?* — e dimensione o campo para isso, nem mais, nem menos. Superdimensionar o mapa de influência é um dos desperdícios de CPU mais comuns em IA tática.

> ❌ **Erro Comum**
> Recalcular o mapa de influência inteiro, do zero, a cada quadro. É a implementação ingênua que "funciona" no protótipo com dez unidades e num mapa pequeno — e que **derrete o desempenho** assim que o mapa cresce ou as unidades se multiplicam, exatamente quando o mapa de influência seria mais útil. O custo de atualização cresce com o número de células **e** com a frequência; ignorar isso é assinar um cheque que o orçamento de quadro não pode pagar. As três defesas — baixa frequência, atualização incremental e resolução adequada — não são "otimizações avançadas opcionais": são parte do **design correto** da técnica desde o início.

[IMAGEM NECESSÁRIA]
Título: Mapa de calor de influência sobreposto a um mapa de RTS
Objetivo didático: Dar ao estudante uma imagem mental concreta de como um mapa de influência "se parece" quando visualizado sobre um jogo real de estratégia, conectando a abstração do campo escalar à experiência de jogo.
Descrição: Screenshot (ou mock-up) de um mapa de jogo de estratégia visto de cima, com unidades de dois exércitos, sobreposto por um mapa de calor semitransparente: tons azuis nas áreas controladas por um lado, tons vermelhos nas do outro, faixa amarela/laranja de alta tensão na linha de frente entre eles, e o restante neutro/transparente. Marcar com um rótulo a "linha de frente" (fronteira) e uma "zona de vulnerabilidade inimiga".
Tipo: Screenshot com sobreposição de dados (data overlay) ou mock-up esquemático.
Como produzir: Usar uma ferramenta de RTS com depuração de IA que exponha o mapa de influência (muitos engines de estratégia têm um modo de visualização de debug), ou montar sobre um screenshot uma camada de gradiente que represente os valores de controle e tensão. Ferramenta: editor de imagem com camadas, ou o próprio visualizador de debug do engine.
Legenda sugerida: "Visualização de debug de um mapa de influência: azul e vermelho indicam controle de cada lado; a faixa quente marca a linha de frente disputada."
[/IMAGEM NECESSÁRIA]

---

## 10.4 Exemplos

Vejamos a técnica em ação, retomando as situações da Seção 10.1 e ampliando o repertório. Em todos os casos, o padrão é o mesmo: **construir um campo (uma combinação de camadas) e escolher a célula de melhor valor** — mas o que muda é *qual* campo e *qual* critério de escolha.

**Seleção de cobertura.** O atirador da Situação 1 consulta um campo de **segurança** construído a partir da camada de **ameaça** dos inimigos. As posições de cobertura candidatas (obstáculos, muros, quinas) são avaliadas pelo valor do campo de segurança em cada uma: a melhor cobertura é a que combina **baixa ameaça** (protegida das linhas de tiro inimigas) com **proximidade razoável** (não é preciso atravessar o mapa) e, muitas vezes, **proximidade de aliados**. Repare como o mapa de influência **resolve de uma vez** o que exigiria comparar cada cobertura contra cada inimigo: a ameaça de todos os inimigos já está **somada** no campo; basta ler o valor da célula.

**Avanço de exército.** O general de RTS da Situação 2 combina três camadas: **oportunidade** (proximidade de recursos e de alvos fracos do inimigo — a vulnerabilidade inimiga), **risco** (a camada de ameaça / influência inimiga) e **apoio** (proximidade das próprias forças). O campo de decisão pondera essas três camadas segundo a "doutrina" da IA, e o exército marcha para a região de maior valor — tipicamente uma zona onde há algo a ganhar, o inimigo está fraco e os reforços estão perto. Muda-se a agressividade da IA mexendo nos pesos, sem reescrever lógica.

**Mapa de perigo.** O NPC ferido da Situação 3 mantém uma camada de **perigo** (ameaça) e simplesmente busca a direção do **gradiente descendente** desse campo — isto é, move-se, passo a passo, para a célula vizinha de **menor** perigo. Isso o afasta de **todas** as ameaças ao mesmo tempo, de forma emergente, sem que ninguém tenha programado "fuja da ameaça A, depois da B". O campo já integrou todas as ameaças; seguir o gradiente é seguir o caminho mais seguro. Perceba a beleza: um comportamento de fuga inteligente e coordenado emerge de uma regra local trivial ("vá para a célula vizinha mais fria") aplicada sobre um campo global.

**Controle territorial.** Em jogos de estratégia, o campo de **controle** (Seção 10.2.2) alimenta decisões de alto nível: onde construir a próxima base (em território seguro, longe da fronteira), onde posicionar defesas (na fronteira, onde o controle está disputado), quando recuar (quando o controle de uma região vira negativo). O mesmo campo, combinado de formas diferentes, sustenta decisões econômicas, defensivas e ofensivas — um único sistema alimentando toda a "consciência espacial" da IA.

> 🎮 **Na Prática**
> Uma técnica poderosa e barata é a **descida (ou subida) de gradiente** sobre o campo. Em vez de rodar um pathfinding completo para "fugir do perigo" ou "avançar para a oportunidade", o agente apenas olha as células vizinhas e caminha na direção que mais melhora seu valor no campo — subindo rumo às oportunidades ou descendo rumo à segurança. É um movimento **reativo, local e extremamente barato**, que produz trajetórias sensatas sem qualquer busca global. A limitação clássica é ficar preso em **mínimos/máximos locais** (um "vale" cercado por perigo, do qual todo passo imediato piora a situação) — um problema que veremos ecoar em outras técnicas de otimização, e que costuma ser mitigado combinando o gradiente com o pathfinding tradicional para saídas de longo alcance.

> 🏭 **Na Indústria**
> Os mapas de influência raramente decidem **sozinhos** — eles **alimentam** as outras técnicas da apostila. Numa arquitetura típica de IA de jogo, o mapa de influência fornece a **avaliação espacial** (onde é bom/ruim), uma **árvore de comportamento** ou **FSM** (Parte II) decide **o que fazer** com base nessa avaliação ("se minha posição é perigosa demais, buscar cobertura"), e o **pathfinding** (Parte III) executa o deslocamento até a célula escolhida. Os sistemas não competem: eles se **encaixam**, cada um respondendo à pergunta que sabe responder. Reconhecer essa divisão de trabalho — avaliar, decidir, mover — é um dos aprendizados mais transferíveis desta apostila.

---

## 10.5 Vantagens e limitações

**Vantagens.** O mapa de influência oferece um punhado de benefícios que explicam sua longevidade. Primeiro, **generalidade**: um mesmo mecanismo (fontes + propagação + decaimento + combinação) modela perigo, controle, oportunidade, "cheiro", visibilidade — qualquer qualidade espacial. Segundo, **integração de muitos fatores**: o campo **soma** naturalmente a contribuição de dezenas de fontes; ler o valor de uma célula já considera todos os inimigos, aliados e recursos de uma vez, sem laços aninhados. Terceiro, **comportamento emergente e crível**: regras locais triviais (seguir o gradiente) produzem táticas sofisticadas — fuga coordenada, avanço por flancos, ocupação de fronteiras — sem que cada tática seja programada explicitamente. Quarto, **controle de autoria**: ajustar pesos e decaimentos afina a "personalidade" tática da IA sem reescrever código, dando aos designers um painel de controle intuitivo. Quinto, **reaproveitamento da base espacial**: o campo assenta sobre a mesma grade/NavMesh do pathfinding (Capítulo 7), sem exigir uma nova representação do mundo.

**Limitações.** Os custos são igualmente concretos e definem onde a técnica cabe.

**Custo de atualização.** Como vimos na Seção 10.3, manter o campo atualizado é a principal despesa. Cresce com o número de células e com a frequência de atualização, e exige as defesas discutidas (baixa frequência, incremental, *time-slicing*) para caber no orçamento de quadro.

**Resolução.** É um dilema permanente: grade **fina** dá decisões precisas, mas custa muito a atualizar e a armazenar; grade **grossa** é barata, mas pode "borrar" detalhes táticos importantes (uma passagem estreita, uma cobertura específica). Escolher a resolução certa é um julgamento de engenharia sem resposta universal.

**Memória.** Cada camada é uma grade inteira de valores; manter várias camadas (influência aliada, inimiga, ameaça, perigo, visibilidade…) para um mapa grande consome memória proporcional a `número de células × número de camadas`. Em plataformas limitadas ou mapas enormes, isso pesa.

**Escalabilidade.** As três limitações acima se **multiplicam**: um mapa grande, com muitas camadas, atualizado com frequência, pode facilmente estourar tanto CPU quanto memória. A escalabilidade do mapa de influência é, portanto, um equilíbrio delicado entre **tamanho do mapa**, **número de camadas**, **resolução** e **frequência** — e é raro poder maximizar os quatro.

> ⚠️ **Atenção**
> Uma limitação **conceitual**, e não apenas de custo: o mapa de influência é excelente para decisões **espaciais** e **agregadas** ("qual região é melhor?"), mas **não** substitui o raciocínio sobre **entidades individuais** e **sequências de ações**. Ele diz "esta área é perigosa", não "aquele atirador específico vai recarregar em 2 segundos, então avance agora". Para lógica individual, temporal ou sequencial, continuam necessárias as FSMs, árvores de comportamento e planejamento da Parte II. O mapa de influência é uma **camada de percepção tática do espaço**, não um cérebro completo — e tratá-lo como se fosse leva a IAs que "entendem" o mapa mas não sabem o que fazer momento a momento.

> ✅ **Boa Prática**
> Comece **simples e barato**, e só refine sob medição. Uma única camada de influência, em grade grossa, atualizada duas vezes por segundo, já resolve uma quantidade surpreendente de problemas táticos — e é fácil de depurar. Adicione camadas, aumente a resolução ou a frequência **apenas** quando um comportamento observável exigir e o perfil de desempenho permitir. Como em toda a Parte III, a regra é **medir antes de otimizar** (e antes de sofisticar): a complexidade do mapa de influência deve ser puxada pela necessidade real do jogo, não empurrada pela vontade de usar todos os recursos da técnica.

---

## 10.6 Jogos conhecidos

Como em toda a apostila, a ressalva vale: salvo quando a equipe documentou publicamente sua IA (em GDC, *Game AI Pro* ou postmortems), descrevemos a técnica **provável**, inferida do comportamento observado e das práticas conhecidas da indústria — não uma implementação confirmada.

**Age of Empires** e a linhagem dos **RTS clássicos** são o lar por excelência do mapa de influência. A IA precisa decidir, sobre um mapa grande, onde expandir, onde atacar, onde defender e por onde marchar exércitos — decisões espaciais agregadas que são exatamente o que o campo de influência resolve. O comportamento típico dessas IAs — concentrar ataques em pontos fracos, evitar avançar para dentro de forças superiores, defender a fronteira, recuar quando o controle se perde — é a assinatura de um raciocínio baseado em controle territorial e ameaça. É difícil imaginar uma IA de RTS competente **sem** alguma forma de mapa de influência por baixo.

**Civilization** e os jogos de estratégia **por turnos** usam a mesma ideia, com uma vantagem: como o jogo é por turnos, o campo pode ser recalculado **entre turnos**, sem pressão de tempo real, permitindo mapas de influência mais ricos e finos. A IA da série avalia o mapa para decidir onde fundar cidades (território seguro e valioso), onde posicionar unidades (fronteiras, pontos estratégicos) e para onde expandir — decisões que casam perfeitamente com camadas de controle, ameaça e oportunidade sobre a grade de casas do mapa.

**StarCraft** e os RTS competitivos modernos levam a técnica a um alto grau de sofisticação. A necessidade de gerenciar controle de mapa, "linhas de visão", posicionamento de exércitos e timing de ataques faz do mapa de influência (em múltiplas camadas) uma ferramenta natural — e a comunidade de IA que desenvolve bots para esses jogos documenta abertamente o uso de mapas de influência como componente central de percepção espacial.

De modo geral, **qualquer jogo de estratégia** com decisões sobre território — de tower defenses (onde posicionar torres para cobrir caminhos) a *4X* (explorar, expandir, explorar, exterminar) — é candidato natural. E o uso não se restringe à estratégia: jogos de **ação tática** e **tiro** usam mapas de perigo e de cobertura (versões locais e menores da mesma ideia) para posicionar inimigos de forma crível — assunto que reencontraremos, sob a ótica de sensoriamento e posicionamento, nos estudos de caso da Parte VII.

> 🎲 **Curiosidade**
> A ideia de "campo que se propaga pelo espaço navegável e guia o movimento" reaparece, com outro nome, na técnica de **flow field** (campo de fluxo) que vimos no Capítulo 9 para mover multidões. Não é coincidência: ambos são campos escalares/vetoriais sobre a grade, calculados por propagação a partir de fontes/destinos. Um mapa de perigo cujo **gradiente** aponta para a segurança é, essencialmente, um flow field de fuga. Reconhecer que técnicas de capítulos diferentes compartilham a mesma raiz matemática — propagar valores por um grafo — é um sinal de maturidade conceitual, e explica por que engines que já têm flow fields conseguem reaproveitar boa parte da infraestrutura para mapas de influência.

---

## 10.7 Ferramentas

Aqui é preciso ser franco sobre uma diferença importante entre as engines.

**Unity: sem sistema nativo dedicado.** A Unity **não** oferece um sistema oficial de mapas de influência, da mesma forma que oferece o NavMesh para navegação (Capítulo 7) ou o pacote **Unity Behavior** para árvores de comportamento (Parte II). O mapa de influência, na Unity, é **construído pelo desenvolvedor**. Isso não é uma deficiência acidental — reflete a natureza da técnica: um mapa de influência é fortemente **específico do jogo** (que camadas? que fontes? que resolução? que frequência?), e um sistema genérico "de prateleira" raramente serviria bem a jogos tão diferentes quanto um RTS e um jogo de tiro tático.

Na prática, o desenvolvedor da Unity constrói o mapa de influência **sobre a infraestrutura que já existe**:

- A **grade** (grid): a forma mais direta é manter uma matriz de valores (um `float[,]` ou estrutura equivalente) cobrindo o mapa, com cada célula guardando a influência. A propagação percorre essa matriz. Para desempenho, a Unity oferece o **Job System** e **Burst** (processamento paralelo e compilação otimizada), que se encaixam bem no cálculo célula a célula, e estruturas de dados como `NativeArray` para as camadas.
- O **NavMesh** e o sistema de **AI Navigation**: o mapa de influência pode assentar sobre a mesma topologia navegável, garantindo que a propagação (e o decaimento) siga a **distância de caminhada**, contornando obstáculos (Seção 10.2.1). É comum usar o NavMesh para o movimento (chegar à célula escolhida) e o campo de influência para a **escolha** dessa célula — a divisão "onde ir" versus "como chegar" concretizada em dois subsistemas.
- **Sistemas próprios em C#**: a lógica de fontes, propagação, decaimento, combinação de camadas e atualização (com baixa frequência e *time-slicing*) é escrita pelo time, tipicamente como um serviço central que os agentes consultam. É exatamente o tipo de sistema que esta apostila ensina a **conceber**, não a copiar de um menu.

**Unreal: o EQS como termo de comparação.** A Unreal Engine oferece o **EQS — Environment Query System** (Sistema de Consulta ao Ambiente), e a comparação com ele é iluminadora. O EQS **não é** exatamente um mapa de influência, mas resolve o **mesmo problema de fundo** — "qual é a melhor posição/ator no ambiente para este objetivo?" — de uma forma complementar. Enquanto o mapa de influência **pré-calcula um campo denso** sobre todo o mapa e depois consulta células, o EQS faz o inverso: no **momento da decisão**, gera um conjunto de **pontos candidatos** (numa grade ao redor do agente, ao redor de um alvo, sobre atores etc.) e os **pontua** aplicando uma cadeia de **testes** (distância ao inimigo, linha de visão, exposição, proximidade de cobertura…), escolhendo o candidato de maior nota. É uma avaliação **sob demanda e localizada**, em vez de um campo **persistente e global**.

A distinção é conceitualmente rica e vale internalizar: o **mapa de influência** é "empurrado" (as fontes empurram influência para o campo, que fica pronto para consulta) e brilha em decisões **globais e agregadas** (controle territorial, avanço de exércitos), amortizando o custo entre muitos consumidores; o **EQS** é "puxado" (o agente puxa uma avaliação pontual quando precisa) e brilha em decisões **locais e situacionais** (qual das coberturas ao meu redor agora?), pagando o custo só quando há uma pergunta a responder. Muitos jogos usam **ambas** as abordagens, cada uma onde é mais eficiente. Compreender essa dualidade — campo persistente versus consulta pontual — dá ao estudante o critério para escolher a ferramenta certa mesmo em engines que rotulam as coisas de forma diferente.

**Soluções de terceiros.** No ecossistema Unity, há *assets* da **Asset Store** que oferecem implementações de mapas de influência e de sistemas de consulta ambiental inspirados no EQS, poupando a escrita da infraestrutura básica. Eles são úteis como ponto de partida, mas — coerente com a filosofia da apostila — devem ser avaliados pelas mesmas perguntas de sempre: que **representação** suportam, que **camadas** e **combinações** permitem, como fazem a **atualização** (frequência, incremental, *time-slicing*) e a que **custo**. Nenhum deles é uma "solução oficial", e a compreensão conceitual do capítulo é o que permite julgá-los.

> 🏭 **Na Indústria**
> A ausência de um sistema nativo de mapas de influência na Unity **não** significa que a técnica seja marginal — significa que ela é, por natureza, **artesanal e específica do jogo**. Estúdios que fazem estratégia constroem seus próprios sistemas de influência sob medida, afinados para o seu mapa e o seu ritmo, e tratam esse sistema como um **diferencial** da IA do jogo. A lição de carreira é clara: dominar o **conceito** (este capítulo) vale mais do que dominar qualquer botão de qualquer engine, porque é o conceito que você levará de um projeto a outro, de uma ferramenta a outra — inclusive para engines que ainda nem existem.

> ✅ **Boa Prática**
> Ao construir um mapa de influência na Unity, **separe** desde o início três responsabilidades: (1) a **representação** do campo (a estrutura de dados das camadas, idealmente amigável ao Job System/Burst para escalar); (2) a **atualização** (o serviço que coleta fontes, propaga, decai e combina, com sua política de frequência e *time-slicing*); e (3) o **consumo** (como os agentes leem o campo para decidir). Manter esses três blocos desacoplados torna o sistema testável (você pode visualizar o campo isoladamente, como no debug da Seção 10.3), ajustável (designers mexem nos pesos sem tocar na propagação) e reaproveitável entre projetos — exatamente as qualidades de engenharia que distinguem um sistema tático profissional de um protótipo emaranhado.

---

## Resumo

Este capítulo abriu a **Parte IV — Raciocínio Espacial e Tático** apresentando a técnica que responde à pergunta que o pathfinding **não** responde. A Parte III nos ensinou **como chegar** a um destino; o mapa de influência ensina a decidir **onde ir** — a escolher, entre todas as posições e regiões do mapa, aquela que vale a pena. Partimos de três **problemas** de design concretos (escolher uma cobertura, avançar um exército, fugir do perigo) para mostrar que muitas decisões de IA são **espaciais** e **multifatoriais**, e que programá-las caso a caso não escala.

Construímos o **conceito**: o mapa de influência é um **campo escalar** sobre o mapa — um valor por região que resume uma qualidade tática (perigo, controle, oportunidade). Detalhamos os três pilares do **funcionamento** — **fontes** (entidades que emitem influência), **propagação** (o espalhamento pela vizinhança, de preferência pela grade navegável, contornando obstáculos) e **decaimento** (o enfraquecimento com a distância) — e mostramos como **combinar camadas** (influência, tensão, vulnerabilidade, ameaça, segurança) por aritmética simples célula a célula para produzir campos de decisão ricos, ajustáveis por **pesos** que codificam a personalidade tática da IA. Vimos que a viabilidade prática depende inteiramente da **atualização em tempo real** — resolvida com **baixa frequência**, **atualização incremental**, ***time-slicing*** e **resolução adequada** —, e que a alternativa ingênua (recalcular tudo a cada quadro) é o erro que derruba o desempenho.

Nos **exemplos**, o padrão se repetiu — construir um campo e escolher a melhor célula — na seleção de cobertura, no avanço de exército, no mapa de perigo (via descida de gradiente) e no controle territorial, sempre com o mapa de influência **alimentando** as demais técnicas (FSMs, árvores de comportamento, pathfinding), não substituindo-as. Pesamos **vantagens** (generalidade, integração de muitos fatores, emergência crível, controle de autoria, reaproveitamento da base espacial) e **limitações** (custo de atualização, dilema de resolução, memória por camada, escalabilidade, e o limite conceitual de não raciocinar sobre entidades individuais e sequências). Vimos os **jogos** (Age of Empires, Civilization, StarCraft e a estratégia em geral) e, nas **ferramentas**, enfrentamos a decisão editorial de frente: a Unity **não** tem sistema nativo — o mapa de influência é construído sobre grid, NavMesh e C# (com Job System/Burst para escalar) — enquanto a Unreal oferece o **EQS**, cuja comparação (campo persistente e global versus consulta pontual e local) dá ao estudante o critério para escolher a abordagem certa em qualquer engine.

O fio que amarra o capítulo, e que se estenderá adiante, é este: o mapa de influência é uma **função de avaliação espacial** — ele atribui um valor a cada posição para orientar a escolha. Essa mesma ideia de "avaliar posições com um número para decidir a melhor jogada" reaparece, no próximo capítulo, como a **função de avaliação** do **Minimax** (Capítulo 11), agora aplicada a estados de um jogo adversarial em vez de células de um mapa. E a capacidade de reconhecer, no comportamento de um jogo, que "esta IA claramente avalia o território" será uma das ferramentas da **engenharia reversa** da Parte VII (Capítulo 14). Raciocinar sobre o espaço com campos de valor é, assim, não um tópico isolado, mas uma peça central do modo como a IA de jogos "pensa" onde agir.

## Exercícios de Fixação

1. Explique, com suas palavras e um exemplo próprio, a diferença entre as perguntas "**como chegar?**" (pathfinding) e "**onde ir?**" (mapa de influência). Por que o pathfinding, sozinho, não resolve a escolha de uma cobertura?

2. O que é um **campo escalar** aplicado ao mapa de um jogo? Como ele difere da grade de **navegação** do Capítulo 7, mesmo quando ambos usam a mesma grade como suporte?

3. Defina **fonte**, **propagação** e **decaimento**. Qual o papel de cada um na transformação de pontos isolados em zonas de influência úteis para decisão?

4. Compare o decaimento por **distância euclidiana** (linha reta) com o decaimento por **distância de caminhada** (propagação pela grade navegável). Em que situação a diferença entre eles é decisiva para o comportamento tático? Dê um exemplo.

5. A partir de duas camadas básicas (influência aliada e inimiga), mostre como se calculam os campos de **controle**, **tensão** e **fronteira**. O que cada um desses campos derivados "significa" taticamente?

6. Por que o mapa de influência **não** precisa ser recalculado a cada quadro? Descreva as quatro técnicas que tornam a atualização viável (baixa frequência, incremental, *time-slicing*, resolução reduzida) e explique o que cada uma economiza.

7. Explique o comportamento de **fuga** de um NPC via **descida de gradiente** sobre um mapa de perigo. Por que isso o afasta de todas as ameaças ao mesmo tempo sem que ninguém programe isso explicitamente? Qual é a limitação clássica dessa abordagem?

8. Liste três **vantagens** e três **limitações** do mapa de influência. Explique por que resolução, memória e custo de atualização se **multiplicam** ao afetar a escalabilidade.

9. Por que a Unity **não** oferece um sistema nativo dedicado de mapas de influência? Sobre quais recursos existentes (grid, NavMesh, C#, Job System/Burst) o desenvolvedor constrói o seu?

10. Compare o **mapa de influência** com o **EQS** da Unreal em termos de "campo persistente e global" versus "consulta pontual e local". Descreva um problema de jogo em que cada abordagem seria a mais eficiente, e justifique.

## Referências

MILLINGTON, Ian. *AI for Games.* 3. ed. Boca Raton: CRC Press, 2019.

RABIN, Steve (org.). *Game AI Pro: Collected Wisdom of Game AI Professionals.* Boca Raton: A K Peters/CRC Press, 2013.

RABIN, Steve (org.). *Game AI Pro 3: Collected Wisdom of Game AI Professionals.* Boca Raton: A K Peters/CRC Press, 2017.

RUSSELL, Stuart; NORVIG, Peter. *Inteligência Artificial.* 3. ed. Rio de Janeiro: Elsevier, 2013.

BOURG, David M.; SEEMANN, Glenn. *AI for Game Developers.* Sebastopol: O'Reilly Media, 2004.

UNITY TECHNOLOGIES. *Documentação oficial: AI Navigation (NavMesh), C# Job System e Burst Compiler.*

EPIC GAMES. *Unreal Engine Documentation: Environment Query System (EQS).*
