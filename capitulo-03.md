---
# Capítulo 3
# Sistemas de Reescrita de Termos e Semântica Operacional de Especificações Algébricas
---

Após a fundamentação teórica da especificação algébrica, que estabelece a sintaxe dos Tipos Abstratos de Dados (TADs) através de assinaturas e sua semântica comportamental por meio de axiomas equacionais, este capítulo se volta para uma perspectiva mais dinâmica e computacional: a semântica operacional dessas especificações, investigada através da lente dos Sistemas de Reescrita de Termos (SRTs, ou TRSs – Term Rewriting Systems). A reescrita de termos oferece um mecanismo formal para interpretar os axiomas equacionais não apenas como declarações de igualdade, mas como regras direcionais de transformação, que permitem "calcular" ou "simplificar" termos complexos em termos mais simples ou canônicos. Esta abordagem estabelece uma ponte conceitual entre a natureza declarativa das especificações algébricas e a natureza operacional da computação. O capítulo iniciará explorando a motivação para a reescrita de termos, elucidando como os axiomas podem ser orientados para formar regras de reescrita e como essa direcionalidade se relaciona com o processo computacional de avaliação de expressões. Em seguida, serão apresentadas as definições formais essenciais no âmbito dos SRTs, incluindo a derivação de regras de reescrita, a relação de reescrita em um passo e em múltiplos passos, o conceito crucial de formas normais (termos irredutíveis) e a noção de estratégias de reescrita. Uma parte significativa do capítulo será dedicada à análise das propriedades fundamentais dos SRTs, como terminação (garantia de que toda sequência de reescrita eventualmente termina), confluência (garantia de que a ordem de aplicação das regras não afeta o resultado final, levando a formas normais únicas quando existentes) e canonicidade (convergência, que combina terminação e confluência). A aplicação da teoria de reescrita na análise de especificações algébricas será discutida, mostrando como essas propriedades dos SRTs podem informar sobre a consistência e outras características desejáveis das especificações. Finalmente, para ilustrar concretamente os conceitos, um exemplo detalhado de reescrita será desenvolvido, focando na especificação do TAD `Natural`, demonstrando a derivação de regras, a execução de sequências de reescrita e a discussão sobre as propriedades de terminação e confluência para o sistema resultante.

# 3.1 Introdução à Reescrita de Termos (`Term Rewriting`)

A especificação algébrica, como explorada no Capítulo 2, define a semântica de um Tipo Abstrato de Dados (TAD) por meio de um conjunto de axiomas equacionais. Esses axiomas, na forma $t_1 = t_2$, declaram que os termos $t_1$ e $t_2$ são equivalentes, ou seja, denotam o mesmo valor abstrato. Embora essa perspectiva declarativa seja poderosa para o raciocínio lógico e para a definição formal do comportamento, ela não prescreve diretamente um mecanismo de "cálculo" ou "execução". Os Sistemas de Reescrita de Termos (SRTs) oferecem uma abordagem para dotar as especificações algébricas de uma semântica operacional, interpretando os axiomas equacionais como regras direcionais de transformação. Essa mudança de perspectiva – de uma igualdade bidirecional para uma transformação unidirecional – é fundamental para conectar a teoria das especificações com a prática da computação. A reescrita de termos não apenas formaliza a noção de simplificação de expressões, mas também fornece um modelo abstrato de computação que tem profundas implicações para a análise de programas, otimização, verificação de propriedades e implementação de linguagens de programação funcionais e lógicas. Esta seção introdutória explorará a motivação subjacente à reescrita de termos, a ideia de direcionalidade e a intrínseca relação entre reescrita e computação.

**Motivação: Axiomas como Regras de Transformação**

Os axiomas em uma especificação algébrica, como $add(n, zero) = n$ para o TAD `Natural`, estabelecem uma igualdade entre dois termos. Na lógica equacional, essa igualdade é simétrica: se $add(n, zero)$ é igual a $n$, então $n$ também é igual a $add(n, zero)$. No entanto, do ponto de vista computacional, frequentemente interpretamos tais equações de forma direcional. A equação $add(n, zero) = n$ sugere naturalmente uma "simplificação" ou "cálculo": a expressão $add(n, zero)$ pode ser *substituída* ou *reescrita* pela expressão mais simples $n$. Raramente pensaríamos em substituir $n$ por $add(n, zero)$ como um passo de cálculo progressivo, embora seja uma substituição válida na lógica equacional.

A principal motivação para a reescrita de termos é formalizar essa noção intuitiva de simplificação ou avaliação direcionada. Ao transformar axiomas equacionais $t_1 = t_2$ em **regras de reescrita** $t_1 \rightarrow t_2$, impomos uma direção. A regra $t_1 \rightarrow t_2$ significa que uma ocorrência do padrão (ou redex, de "reducible expression") $t_1$ em um termo maior pode ser substituída por sua contraparte $t_2$ (o contráctil). O objetivo é, através de uma sequência de tais reescritas, transformar um termo inicial em um termo "mais simples" ou em uma "forma normal" que não pode mais ser reescrito.

Essa transformação de equações em regras direcionais nem sempre é trivial. Uma equação $t_1 = t_2$ pode, em princípio, dar origem a duas regras: $t_1 \rightarrow t_2$ ou $t_2 \rightarrow t_1$. A escolha da direção (orientação) é crucial e deve ser guiada por critérios que garantam propriedades desejáveis do sistema de reescrita, como a terminação (toda sequência de reescritas termina) e a confluência (o resultado final é independente da ordem de aplicação das regras, quando múltiplas regras são aplicáveis).

Por exemplo, o axioma de comutatividade da adição, $add(x,y) = add(y,x)$, se transformado diretamente em uma regra como $add(x,y) \rightarrow add(y,x)$, poderia levar a uma reescrita não terminante: $add(a,b) \rightarrow add(b,a) \rightarrow add(a,b) \rightarrow \ldots$. Para lidar com tais axiomas, podem ser necessárias técnicas mais sofisticadas, como a reescrita sob teorias equacionais (onde a comutatividade é tratada separadamente) ou a imposição de uma ordem nos termos para que a regra só seja aplicada se o lado esquerdo for "maior" que o lado direito segundo essa ordem.

A interpretação de axiomas como regras de transformação permite:
1.  **Definir uma semântica operacional:** A reescrita fornece um modelo de como os termos são "avaliados" ou "executados".
2.  **Provar a equivalência de termos:** Dois termos $u$ e $v$ podem ser considerados equivalentes se ambos podem ser reescritos para um mesmo termo $w$ (ou seja, $u \rightarrow^* w$ e $v \rightarrow^* w$, onde $\rightarrow^*$ denota múltiplos passos de reescrita). Se o sistema de reescrita for canônico (terminante e confluente), então $u$ e $v$ são equivalentes se e somente se suas formas normais únicas são idênticas.
3.  **Analisar propriedades da especificação:** Propriedades como consistência e completude de uma especificação algébrica podem ser investigadas através das propriedades do sistema de reescrita associado (e.g., confluência e terminação).

A transformação de um conjunto de axiomas equacionais em um conjunto de regras de reescrita é o primeiro passo para utilizar o poderoso ferramental da teoria de reescrita de termos na análise e na implementação de Tipos Abstratos de Dados.

**A Direcionalidade da Reescrita: De Termos Complexos a Termos Mais Simples**

Um conceito central na reescita de termos é a **direcionalidade** imposta às equações quando são transformadas em regras. Uma regra de reescrita $l \rightarrow r$ (onde $l$ é o lado esquerdo, *left-hand side*, e $r$ é o lado direito, *right-hand side*) significa que uma instância de $l$ pode ser substituída por (ou reescrita para) a correspondente instância de $r$. Esta seta ($\rightarrow$) não é simétrica; a reescrita de $r$ para $l$ não é permitida pela regra $l \rightarrow r$ (a menos que exista uma regra separada $r \rightarrow l$, o que geralmente é evitado para garantir terminação).

A intenção por trás dessa direcionalidade é, na maioria das aplicações, mover de termos "mais complexos" para termos "mais simples" ou "mais definidos" ou "mais canônicos". A noção exata de "simplicidade" ou "ordem" pode ser formalizada através de **ordenações de redução** (reduction orderings), que são relações de ordem bem fundadas sobre o conjunto de termos. Uma ordenação de redução $>_{ord}$ é tal que, para cada regra $l \rightarrow r$ do sistema, deve-se ter $l >_{ord} r$ (após qualquer substituição de variáveis por termos). Se tal ordenação existe e todas as regras a respeitam, o sistema de reescrita é garantidamente terminante, pois cada passo de reescrita leva a um termo estritamente "menor" na ordenação, e não pode haver uma sequência infinita decrescente em uma ordem bem fundada.

Exemplos de como a direcionalidade reflete uma noção de simplificação:
*   **Axioma:** `add(n, zero) = n`
    *   **Regra:** `add(n, zero) $\rightarrow$ n`
    Aqui, o lado direito `n` é claramente mais simples (e.g., menor em tamanho, menor número de símbolos de operação) do que o lado esquerdo `add(n, zero)`.
*   **Axioma:** `add(n, succ(m)) = succ(add(n, m))`
    *   **Regra:** `add(n, succ(m)) $\rightarrow$ succ(add(n, m))`
    Neste caso, a "complexidade" da operação `add` é reduzida no lado direito, pois o segundo argumento de `add` passa de `succ(m)` para `m` (que é estruturalmente menor). A operação `succ` é movida para fora. Esta regra é fundamental para a definição recursiva da adição.
*   **Axioma (para `Boolean`):** `and(true, b) = b`
    *   **Regra:** `and(true, b) $\rightarrow$ b`
    A constante `true` é eliminada, e a expressão `and(true, b)` é simplificada para `b`.

A orientação dos axiomas em regras de reescrita é uma etapa crítica. Uma orientação inadequada pode levar a sistemas não terminantes ou não confluentes. Por exemplo, orientar $n \rightarrow add(n, zero)$ seria desastroso para a terminação. Para axiomas como a comutatividade ($x+y = y+x$) ou associatividade ($(x+y)+z = x+(y+z)$), que não têm uma direção "natural" de simplificação, técnicas especiais são necessárias, como:
1.  **Reescrita Condicional:** Aplicar a regra apenas sob certas condições.
2.  **Ordenações Lexicográficas ou de Caminho:** Utilizar ordenações sofisticadas sobre termos que podem justificar uma direção mesmo quando a contagem de símbolos não diminui.
3.  **Reescrita AC (Associativa-Comutativa):** Tratar os axiomas AC separadamente, considerando termos equivalentes sob AC como uma classe de equivalência, e reescrever essas classes.
4.  **Completamento (Knuth-Bendix):** Um algoritmo que tenta transformar um conjunto de equações em um sistema de reescrita canônico (terminante e confluente), às vezes adicionando novas equações (lemas críticos) deduzidas.

A direcionalidade da reescrita é, portanto, um esforço para impor uma noção de progresso computacional ou de simplificação aos axiomas declarativos, permitindo que eles sejam usados de forma efetiva como um motor de cálculo.

**Relação entre Reescrita e Computação**

A reescrita de termos estabelece uma relação profunda e fundamental com o conceito de computação. De fato, os sistemas de reescrita de termos podem ser vistos como um **modelo de computação** abstrato, no mesmo patamar de outros modelos como máquinas de Turing, cálculo lambda, ou funções recursivas.

1.  **Programas como Sistemas de Reescrita:** Muitas linguagens de programação, especialmente as funcionais (como Haskell, Lisp, ML) e algumas lógicas, têm uma semântica operacional que pode ser naturalmente expressa ou aproximada por sistemas de reescrita de termos.
    *   Em linguagens funcionais, a definição de uma função pode ser vista como um conjunto de regras de reescrita. Por exemplo, a definição de fatorial:
        `fact(0) = 1`
        `fact(n) = n * fact(n-1)` (para $n>0$)
        corresponde às regras:
        `fact(0) $\rightarrow$ 1`
        `fact(n) $\rightarrow$ mult(n, fact(pred(n)))` (assumindo `pred` e condições)
    A avaliação de uma expressão funcional como `fact(3)` corresponde a uma sequência de reescritas:
    `fact(3) $\rightarrow$ mult(3, fact(2)) $\rightarrow$ mult(3, mult(2, fact(1))) $\rightarrow$ mult(3, mult(2, mult(1, fact(0)))) $\rightarrow$ mult(3, mult(2, mult(1, 1))) $\rightarrow$ mult(3, mult(2, 1)) $\rightarrow$ mult(3, 2) $\rightarrow$ 6`.

2.  **Computação como Redução a uma Forma Normal:** Em um sistema de reescrita, um cálculo ou computação é uma sequência de aplicações de regras de reescrita a um termo inicial. O objetivo é atingir um termo que não pode mais ser reescrito por nenhuma regra do sistema. Tal termo é chamado de **forma normal** (ou termo irredutível).
    *   Se um sistema de reescrita é **terminante** (toda sequência de reescritas termina), então todo termo possui pelo menos uma forma normal.
    *   Se o sistema é também **confluente** (o resultado final é independente da ordem de aplicação das regras), então a forma normal de um termo, se existir, é **única**.
    Um sistema terminante e confluente é dito **canônico** (ou convergente). Em um sistema canônico, a forma normal única de um termo pode ser considerada o "valor" ou o "resultado da computação" para esse termo.

3.  **Equivalência Computacional e Equivalência Axiomática:** A teoria da reescrita de termos conecta a equivalência computacional (dois termos são computacionalmente equivalentes se são reescritos para a mesma forma normal) com a equivalência axiomática (dois termos são iguais segundo os axiomas da lógica equacional). Se um sistema de reescrita $R$ é derivado de um conjunto de equações $E$ de forma a ser canônico, então dois termos $t_1$ e $t_2$ são iguais segundo $E$ ($E \models t_1 = t_2$) se e somente se suas formas normais em $R$ são idênticas. Isso fornece um **procedimento de decisão** para a teoria equacional $E$.

4.  **Completude de Turing:** Sistemas de reescrita de termos são Turing-completos, o que significa que eles podem simular qualquer computação que uma máquina de Turing pode realizar. Isso os estabelece como um modelo de computação de propósito geral.

5.  **Aplicações Práticas:**
    *   **Demonstração de Teoremas:** Usados em provadores automáticos de teoremas para simplificar expressões e verificar igualdades.
    *   **Análise de Protocolos:** Modelagem e verificação de protocolos de segurança.
    *   **Compiladores e Otimização de Código:** Regras de reescrita podem representar transformações de otimização de código.
    *   **Execução Simbólica:** Para explorar o comportamento de programas com entradas simbólicas.
    *   **Implementação de Linguagens Algébricas:** Linguagens de especificação como OBJ, Maude, e CASL usam reescrita como sua semântica operacional.

Em resumo, a reescrita de termos fornece uma ponte vital entre a especificação declarativa (axiomas equacionais) e a computação operacional. Ao orientar axiomas em regras e aplicar essas regras para reduzir termos a formas normais, podemos simular a avaliação de expressões, provar propriedades e, em última análise, entender melhor o poder computacional e o comportamento dos Tipos Abstratos de Dados que especificamos.

**Exercício:**

Considere o TAD `Boolean` com as operações `true`, `false`, `not`, `and` e os seguintes axiomas como possíveis regras de reescrita:
1.  `not(true) $\rightarrow$ false`
2.  `not(false) $\rightarrow$ true`
3.  `and(true, b) $\rightarrow$ b` (onde `b` é uma variável `Boolean`)
4.  `and(false, b) $\rightarrow$ false` (onde `b` é uma variável `Boolean`)

Mostre uma sequência de reescrita para o termo `and(not(false), true)` até atingir uma forma normal. Indique qual regra é aplicada em cada passo.

**Resolução:**

O termo inicial é `and(not(false), true)`.
Vamos aplicar as regras de reescrita fornecidas:
Regras:
(R1): `not(true) $\rightarrow$ false`
(R2): `not(false) $\rightarrow$ true`
(R3): `and(true, b) $\rightarrow$ b`
(R4): `and(false, b) $\rightarrow$ false`

Sequência de reescrita:

1.  **Termo atual:** `and(not(false), true)`
    *   O subtermo `not(false)` corresponde ao lado esquerdo da Regra (R2).
    *   Aplicando (R2) a `not(false)` (onde `not(false)` é substituído por `true`):
    **Termo resultante:** `and(true, true)`
    (Passo: `and(not(false), true) $\rightarrow_{R2}$ and(true, true)`)

2.  **Termo atual:** `and(true, true)`
    *   Este termo corresponde ao lado esquerdo da Regra (R3), onde a variável `b` é instanciada com `true`.
    *   Aplicando (R3) a `and(true, true)` (onde `and(true, true)` é substituído por `true`):
    **Termo resultante:** `true`
    (Passo: `and(true, true) $\rightarrow_{R3}$ true`)

3.  **Termo atual:** `true`
    *   O termo `true` não corresponde ao lado esquerdo de nenhuma das regras (R1-R4). Portanto, ele não pode mais ser reescrito.
    *   Este é uma forma normal.

A sequência completa de reescrita é:
`and(not(false), true)`
$\rightarrow_{R2}$ `and(true, true)`
$\rightarrow_{R3}$ `true`

A forma normal do termo `and(not(false), true)` é `true`.

# 3.2 Definições Formais em Sistemas de Reescrita de Termos (SRTs / TRSs)

Para estudar e aplicar a reescrita de termos de forma rigorosa, é essencial estabelecer um conjunto de definições formais. Estas definições formam a base da teoria dos Sistemas de Reescrita de Termos (SRTs), também conhecidos na literatura internacional como TRSs (Term Rewriting Systems). Um SRT é essencialmente uma coleção de regras de reescrita definidas sobre um conjunto de termos gerados por uma assinatura. As definições cruciais incluem a noção de regra de reescrita derivada de axiomas, a relação de reescrita em um passo (que descreve uma única aplicação de regra), a relação de reescrita em múltiplos passos (o fecho reflexivo e transitivo da reescrita em um passo), o conceito de formas normais (termos que não podem mais ser reescritos) e as diferentes estratégias que podem ser empregadas para selecionar qual regra aplicar e onde aplicá-la quando múltiplas opções existem. Esta seção se dedicará a formalizar esses conceitos fundamentais.

**Regras de Reescrita ($l \rightarrow r$) derivadas de Axiomas Equacionais**

Um **Sistema de Reescrita de Termos (SRT)** $R$ sobre uma assinatura $\Sigma$ (e um conjunto $X$ de variáveis) é um conjunto de **regras de reescrita**. Cada regra de reescrita é um par ordenado de termos $(l, r)$, escrito como $l \rightarrow r$, tal que:
1.  $l$ (o lado esquerdo, *left-hand side* ou LHS) e $r$ (o lado direito, *right-hand side* ou RHS) são termos construídos sobre $\Sigma$ e $X$, ou seja, $l, r \in T_{\Sigma}(X)$.
2.  $l$ e $r$ são do mesmo sort.
3.  $l$ não é uma variável isolada. (Esta condição é usual para evitar trivialidades e problemas com terminação, embora algumas teorias possam relaxá-la sob certas condições).
4.  Toda variável que ocorre em $r$ também deve ocorrer em $l$. (Esta condição, chamada de restrição de variáveis, é importante para garantir que a reescrita não introduza variáveis "do nada". Ela assegura que, se reescrevemos um termo fundamental, o resultado também será um termo fundamental).

A derivação de regras de reescrita a partir de um conjunto $E$ de axiomas equacionais é um processo de **orientação** das equações. Para cada equação $t_1 = t_2 \in E$:
*   Podemos tentar orientá-la como uma regra $t_1 \rightarrow t_2$ ou como $t_2 \rightarrow t_1$.
*   A escolha da orientação deve ser feita com cuidado. O objetivo é geralmente criar um sistema de reescrita que seja **terminante** (toda sequência de reescritas termina). Isso muitas vezes envolve garantir que o lado direito seja "mais simples" ou "menor" que o lado esquerdo de acordo com alguma **ordenação de redução** bem fundada $>_{ord}$ (i.e., para cada regra $l \rightarrow r$, deve valer $l\sigma >_{ord} r\sigma$ para toda substituição $\sigma$).

**Exemplo de Derivação de Regras:**
Dada a especificação `Natural` com os axiomas:
(N1): `isZero(zero) = true`
(N2): `isZero(succ(n)) = false`
(N3): `add(n, zero) = n`
(N4): `add(n, succ(m)) = succ(add(n, m))`

Podemos orientá-los para obter o seguinte conjunto de regras $R_{Natural}$:
(rN1): `isZero(zero) $\rightarrow$ true`
(rN2): `isZero(succ(n)) $\rightarrow$ false`
(rN3): `add(n, zero) $\rightarrow$ n`
(rN4): `add(n, succ(m)) $\rightarrow$ succ(add(n, m))`

Para cada regra:
*   Os lados esquerdo e direito são do mesmo sort (`Boolean` para rN1, rN2; `Natural` para rN3, rN4).
*   Os lados esquerdos não são variáveis isoladas.
*   Todas as variáveis no lado direito aparecem no lado esquerdo (e.g., em rN4, $n, m$ em `succ(add(n,m))` também estão em `add(n, succ(m))`).
*   Intuitivamente, os lados direitos parecem "mais simples" ou representam um passo de cálculo (e.g., `add(n, zero)` é mais complexo que `n`; em `add(n, succ(m))`, o segundo argumento de `add` no lado direito, `m`, é estruturalmente menor que `succ(m)`).

Nem todas as equações são facilmente orientáveis. Axiomas como comutatividade ($x+y=y+x$) ou associatividade ($(x+y)+z=x+(y+z)$) não têm uma direção de simplificação óbvia e, se orientados ingenuamente, podem levar a não-terminação. Tais equações são frequentemente tratadas por técnicas especiais, como reescrita módulo uma teoria equacional (e.g., reescrita AC para axiomas associativos e comutativos) ou usando ordenações mais sofisticadas. Por ora, focaremos em equações que podem ser orientadas de forma mais direta.

Um sistema de reescrita de termos é, portanto, a peça central para dar uma semântica operacional a uma especificação algébrica. Ele nos diz *como* transformar termos, enquanto os axiomas apenas nos dizem *quais* termos são equivalentes.

**A Relação de Reescrita em Um Passo ($\rightarrow_R$)**

Uma vez que temos um Sistema de Reescrita de Termos $R$ (um conjunto de regras de reescrita), podemos definir a **relação de reescrita em um passo**, denotada por $\rightarrow_R$ (ou simplesmente $\rightarrow$ quando $R$ está claro no contexto). Esta relação descreve como um termo $s$ pode ser transformado em um termo $t$ pela aplicação de uma única regra de $R$ a um subtermo de $s$.

Formalmente, dizemos que um termo $s$ reescreve em um passo para um termo $t$, escrito $s \rightarrow_R t$, se existem:
1.  Uma regra $l \rightarrow r$ em $R$.
2.  Uma **posição** (ou ocorrência de subtermo) $p$ em $s$. (Posições podem ser representadas como sequências de inteiros que navegam da raiz até o subtermo. Por exemplo, em $f(g(a), b)$, $g(a)$ está na posição 1, $a$ na posição 1.1, $b$ na posição 2).
3.  Uma **substituição** $\sigma$ (um mapeamento de variáveis para termos, que preserva sorts).

tais que:
*   O subtermo de $s$ na posição $p$, denotado $s|_p$, é igual à instância $l\sigma$ do lado esquerdo da regra (i.e., $s|_p = l\sigma$). Este subtermo $s|_p$ é chamado de **redex** (reducible expression).
*   O termo $t$ é obtido substituindo o subtermo $s|_p$ em $s$ pela instância correspondente $r\sigma$ do lado direito da regra. Isso é escrito como $t = s[r\sigma]_p$.

Em outras palavras, $s \rightarrow_R t$ se $s$ contém uma subexpressão que "casa" (matches) com o lado esquerdo de uma regra (após substituição das variáveis da regra), e $t$ é o resultado de trocar essa subexpressão pela correspondente instância do lado direito da regra.

**Exemplo:**
Seja $R_{Natural}$ o conjunto de regras da seção anterior, incluindo (rN4): `add(n, succ(m)) $\rightarrow$ succ(add(n, m))`.
Considere o termo $s = \text{succ(add(succ(zero), succ(zero)))}$.

*   **Posição:** $p=1$ (o subtermo `add(succ(zero), succ(zero))`).
*   **Subtermo $s|_p$:** `add(succ(zero), succ(zero))`.
*   **Regra:** (rN4) `add(n, succ(m)) $\rightarrow$ succ(add(n, m))`.
    Lado esquerdo $l = \text{add(n, succ(m))}$.
*   **Substituição $\sigma$:**
    $\sigma(n) = \text{succ(zero)}$
    $\sigma(m) = \text{zero}$
    Com esta substituição, $l\sigma = \text{add(succ(zero), succ(zero))}$, que é igual a $s|_p$.
    O lado direito $r = \text{succ(add(n, m))}$.
    Então, $r\sigma = \text{succ(add(succ(zero), zero))}$.
*   **Termo $t$:** Obtido substituindo $s|_p$ por $r\sigma$ em $s$:
    $t = s[r\sigma]_p = \text{succ(succ(add(succ(zero), zero)))}$.

Portanto, temos a reescrita em um passo:
$\text{succ(add(succ(zero), succ(zero)))} \rightarrow_{R_{Natural}} \text{succ(succ(add(succ(zero), zero)))}$

A escolha da regra, da posição e da substituição não é necessariamente única. Um termo pode ter múltiplos redexes, levando a diferentes possíveis reescritas em um passo.

**Notação:**
*   $s \rightarrow_R t$: $s$ reescreve para $t$ em um passo usando uma regra de $R$.
*   $s \leftarrow_R t$: $t$ reescreve para $s$ em um passo (relação inversa).
*   $s \leftrightarrow_R t$: $s \rightarrow_R t$ ou $s \leftarrow_R t$.

A relação $\rightarrow_R$ é a unidade fundamental da computação em um SRT. Sequências dessas reescritas em um passo formam computações mais longas.

**A Relação de Reescrita em Múltiplos Passos ($\rightarrow_R^*$ - Fecho Reflexivo e Transitivo)**

A relação de reescrita em um passo, $\rightarrow_R$, descreve uma única transformação. Para capturar a ideia de uma computação completa ou uma sequência de simplificações, precisamos da noção de reescrita em múltiplos passos.

A **relação de reescrita em múltiplos passos**, denotada por $\rightarrow_R^*$ (ou $\xrightarrow{*}$, ou $\rightarrow^*$ quando $R$ está claro), é o **fecho reflexivo e transitivo** da relação de reescrita em um passo $\rightarrow_R$.
Isto significa que $s \rightarrow_R^* t$ se:
*   $s=t$ (reflexividade: zero passos de reescrita), OU
*   Existe uma sequência finita de termos $u_0, u_1, \ldots, u_k$ (com $k \ge 1$) tal que:
    $s = u_0 \rightarrow_R u_1 \rightarrow_R u_2 \rightarrow_R \ldots \rightarrow_R u_k = t$.
    (Transitividade: um ou mais passos de reescrita).

Formalmente:
*   $\rightarrow_R^0$ é a relação de identidade (i.e., $s \rightarrow_R^0 s$ para todo $s$).
*   $\rightarrow_R^{k+1} = \rightarrow_R \circ \rightarrow_R^k$ (composição de relações: $s \rightarrow_R^{k+1} t$ se existe $u$ tal que $s \rightarrow_R u$ e $u \rightarrow_R^k t$).
*   $\rightarrow_R^* = \bigcup_{k \ge 0} \rightarrow_R^k$. (A união de reescritas em $0, 1, 2, \ldots$ passos).

Alternativamente, $\rightarrow_R^*$ é a menor relação que satisfaz:
1.  Se $s=t$, então $s \rightarrow_R^* t$ (Reflexividade).
2.  Se $s \rightarrow_R t$, então $s \rightarrow_R^* t$ (Inclusão da relação de um passo).
3.  Se $s \rightarrow_R^* u$ e $u \rightarrow_R^* t$, então $s \rightarrow_R^* t$ (Transitividade).

Outras notações comuns relacionadas:
*   $\rightarrow_R^+$: O **fecho transitivo** de $\rightarrow_R$. Ou seja, $s \rightarrow_R^+ t$ se $s$ reescreve para $t$ em *um ou mais* passos ($s \rightarrow_R^* t$ e $s \neq t$).
*   $s \downarrow_R t$: Significa que $s$ e $t$ têm um **descendente comum**, i.e., existe um termo $u$ tal que $s \rightarrow_R^* u$ e $t \rightarrow_R^* u$. Esta notação é usada para definir confluência.
*   $s \leftrightarrow_R^* t$: O **fecho reflexivo, simétrico e transitivo** de $\rightarrow_R$. Esta é a relação de equivalência gerada por $R$, onde $s \leftrightarrow_R^* t$ significa que $s$ e $t$ são "conectados" por uma sequência de passos de reescrita para frente ou para trás. Se $R$ é derivado de um conjunto de equações $E$, então $s \leftrightarrow_R^* t$ implica $E \models s=t$. Se $R$ é também canônico, a recíproca vale.

**Exemplo de $\rightarrow_R^*$:**
Usando $R_{Natural}$ e o exemplo anterior, onde tínhamos:
$s_0 = \text{succ(add(succ(zero), succ(zero)))}$
$s_1 = \text{succ(succ(add(succ(zero), zero)))}$
e $s_0 \rightarrow_{R_{Natural}} s_1$.

Agora, vamos continuar a partir de $s_1$. O subtermo `add(succ(zero), zero)` em $s_1$ é um redex para a regra (rN3): `add(n, zero) $\rightarrow$ n`.
Com a substituição $\sigma'(n) = \text{succ(zero)}$, o lado esquerdo $l\sigma' = \text{add(succ(zero), zero)}$.
O lado direito $r\sigma' = \text{succ(zero)}$.
Então, $s_1 \rightarrow_{R_{Natural}} s_2 = \text{succ(succ(succ(zero)))}$.

Assim, temos a sequência:
$s_0 \rightarrow_{R_{Natural}} s_1 \rightarrow_{R_{Natural}} s_2$.
Portanto, $s_0 \rightarrow_{R_{Natural}}^* s_2$, ou seja:
$\text{succ(add(succ(zero), succ(zero)))} \rightarrow_{R_{Natural}}^* \text{succ(succ(succ(zero)))}$.
O termo $s_2 = \text{succ(succ(succ(zero)))}$ (representando 3) é uma forma normal neste sistema, pois não contém nenhum subtermo que case com o lado esquerdo de alguma regra em $R_{Natural}$.

A relação $\rightarrow_R^*$ é fundamental porque ela define todas as possíveis "computações" ou "derivações" a partir de um termo inicial. A meta de um sistema de reescrita é frequentemente encontrar a(s) forma(s) normal(is) de um termo através de $\rightarrow_R^*$.

**Formas Normais (Termos Irredutíveis)**

Em um Sistema de Reescrita de Termos $R$, um termo $t$ é dito estar em **forma normal** (ou ser uma **forma normal**) se ele não pode ser reescrito por nenhuma regra em $R$. Ou seja, não existe nenhum termo $u$ tal que $t \rightarrow_R u$. Um termo em forma normal é também chamado de **irredutível**.

Se um termo $s$ pode ser reescrito para uma forma normal $t$ (i.e., $s \rightarrow_R^* t$ e $t$ é uma forma normal), então $t$ é dita uma **forma normal de $s$**.

**Propriedades das Formas Normais:**
*   Um termo pode ter zero, uma ou múltiplas formas normais.
    *   **Zero formas normais:** Se todas as sequências de reescrita a partir de $s$ são infinitas (o sistema não é terminante para $s$).
    *   **Múltiplas formas normais:** Se o sistema não é confluente (ver Seção 3.3.2), um termo pode ser reescrito para diferentes formas normais irredutíveis dependendo da escolha das regras e posições.
    *   **Uma forma normal única:** Se o sistema é **confluente** (ou possui a propriedade de Church-Rosser), então qualquer termo tem no máximo uma forma normal. Se o sistema é também **terminante**, então todo termo tem exatamente uma forma normal única. Tal sistema é dito **canônico** (ou convergente).

**Notação:**
*   Se $t$ é uma forma normal, escreve-se $t \in NF_R$ (onde $NF_R$ é o conjunto de todas as formas normais em $R$).
*   $t!_R$ (ou $t \downarrow_R$ em alguns contextos) pode denotar a forma normal única de $t$, se existir.

**Importância das Formas Normais:**
*   **Resultado da Computação:** Em sistemas canônicos, a forma normal única de um termo é considerada o "valor" ou o resultado da computação representada pelo termo inicial. Por exemplo, na aritmética, $2+2 \rightarrow^* 4$, e $4$ é uma forma normal.
*   **Decisão de Equivalência:** Se um SRT $R$ é canônico e é derivado de um conjunto de equações $E$, então $E \models t_1 = t_2$ se e somente se $t_1$ e $t_2$ têm a mesma forma normal em $R$. Isso fornece um algoritmo para decidir a igualdade na teoria equacional $E$: reescreva $t_1$ e $t_2$ para suas formas normais e compare-as sintaticamente.
*   **Simplificação:** Formas normais representam a forma "mais simples" de um termo com respeito ao sistema de reescrita.

**Exemplo de Formas Normais:**
Considerando o sistema $R_{Natural}$ com regras:
(rN1): `isZero(zero) $\rightarrow$ true`
(rN2): `isZero(succ(n)) $\rightarrow$ false`
(rN3): `add(n, zero) $\rightarrow$ n`
(rN4): `add(n, succ(m)) $\rightarrow$ succ(add(n, m))`
E as constantes `true`, `false` do TAD `Boolean`.

*   Termos em forma normal:
    *   `zero`, `succ(zero)`, `succ(succ(zero))`, ... (representações canônicas dos números naturais)
    *   `true`, `false` (valores booleanos)
    *   `add(n, m)` *não* é uma forma normal se $m$ for `zero` ou `succ(...)`, pois (rN3) ou (rN4) se aplicariam. Só seria forma normal se $n$ e $m$ fossem variáveis e não pudessem ser instanciadas para permitir uma reescrita. No entanto, se $n$ e $m$ são interpretados como "qualquer natural já em forma normal", então o termo `add(forma_normal1, forma_normal2)` é um redex.
    *   `isZero(n)`: não é forma normal se $n$ for `zero` ou `succ(...)`. Só é forma normal se $n$ for uma variável.
*   Exemplo de computação para forma normal:
    `add(succ(zero), succ(zero))`
    $\rightarrow_{rN4}$ `succ(add(succ(zero), zero))` (com $n \mapsto \text{succ(zero)}, m \mapsto \text{zero}$)
    $\rightarrow_{rN3}$ `succ(succ(zero))` (com $n \mapsto \text{succ(zero)}$ no subtermo)
    O termo `succ(succ(zero))` é uma forma normal.

A existência e unicidade de formas normais são propriedades cruciais de um sistema de reescrita, determinando sua utilidade como um mecanismo de cálculo e decisão.

**Reduções e Estratégias de Reescrita (e.g., `innermost`, `outermost`)**

Uma **redução** é simplesmente uma sequência de reescritas $t_0 \rightarrow_R t_1 \rightarrow_R \ldots \rightarrow_R t_k$. Quando um termo $s$ contém múltiplos redexes (subtermos que casam com o lado esquerdo de alguma regra), ou quando um único redex pode ser reescrito por diferentes regras (se houver regras sobrepostas, o que geralmente é evitado em sistemas bem comportados), surge a questão de qual redex escolher e qual regra aplicar. Uma **estratégia de reescrita** (ou estratégia de redução) é um algoritmo ou um conjunto de critérios que especifica como selecionar o próximo redex a ser reescrito em cada passo de uma redução.

A escolha da estratégia de reescrita pode afetar:
1.  **Terminação:** Algumas estratégias podem levar a sequências de reescrita não terminantes, mesmo que o sistema de reescrita seja fracamente terminante (i.e., existe pelo menos uma sequência terminante para cada termo).
2.  **Eficiência:** Diferentes estratégias podem levar à forma normal (se existir) por caminhos de diferentes comprimentos, impactando a eficiência da computação.
3.  **Convergência para uma Forma Normal Específica:** Em sistemas não confluentes, diferentes estratégias podem levar a diferentes formas normais.

Algumas estratégias de reescrita comuns incluem:

*   **Reescrita `innermost` (mais interna, ou `call-by-value` em programação funcional):**
    Um redex é `innermost` se ele não contém nenhum outro redex como subtermo próprio. A estratégia `innermost` consiste em sempre escolher um redex `innermost` para reescrever.
    *   **Vantagem:** Se um sistema de reescrita é terminante, a estratégia `innermost` também é terminante. Além disso, se os argumentos de uma função são reescritos para suas formas normais antes da função ser aplicada, isso pode evitar recomputações de argumentos (similar à avaliação eager).
    *   **Desvantagem:** Pode realizar reescritas desnecessárias em subtermos que seriam descartados por uma regra mais externa.

*   **Reescrita `outermost` (mais externa, ou `call-by-name` / `lazy evaluation` em programação funcional):**
    Um redex é `outermost` se ele não é um subtermo próprio de nenhum outro redex. A estratégia `outermost` consiste em sempre escolher um redex `outermost` para reescrever. Se houver múltiplos redexes `outermost` (e.g., em $f(g(x), h(y))$, $g(x)$ e $h(y)$ poderiam ser redexes outermost se $f$ não for um redex), uma regra adicional (e.g., da esquerda para a direita) pode ser necessária para desambiguar.
    *   **Vantagem:** Pode evitar reescrever subtermos que não são necessários para o resultado final (e.g., se uma regra $l \rightarrow r$ descarta alguns dos subtermos de $l$). Se uma forma normal existe, a estratégia `outermost` (com uma ressalva para argumentos de funções construtoras em alguns contextos) frequentemente a encontra (Teorema da Padronização para sistemas ortogonais).
    *   **Desvantagem:** Pode reescrever o mesmo subtermo (que não foi reduzido a forma normal inicialmente) múltiplas vezes se ele for duplicado pelo lado direito de uma regra.

*   **Reescrita Paralela:** Reeescrever múltiplos redexes disjuntos (que não se sobrepõem) simultaneamente em um único passo.

*   **Estratégias Específicas da Linguagem:** Muitas linguagens funcionais usam variações de `call-by-need` (avaliação preguiçosa com memoização/compartilhamento) que é uma otimização da estratégia `outermost`.

**Exemplo com Estratégias:**
Considere as regras:
(R_F): `F(x) $\rightarrow$ C` (F descarta seu argumento)
(R_G): `G(A) $\rightarrow$ B` (G opera sobre A)
Termo: `F(G(A))`

1.  **Estratégia `innermost`:**
    *   O único redex `innermost` é `G(A)`.
    *   `F(G(A)) $\rightarrow_{R_G}$ F(B)`
    *   Agora, `F(B)` é um redex `outermost` para (R_F).
    *   `F(B) $\rightarrow_{R_F}$ C`
    Sequência: `F(G(A)) $\rightarrow$ F(B) $\rightarrow$ C`. (2 passos)

2.  **Estratégia `outermost`:**
    *   O redex `outermost` é `F(G(A))` (assumindo que $F$ é a função mais externa a ser reduzida).
    *   `F(G(A)) $\rightarrow_{R_F}$ C` (o argumento `G(A)` é descartado)
    Sequência: `F(G(A)) $\rightarrow$ C`. (1 passo)
    Neste caso, `outermost` foi mais eficiente.

A escolha da estratégia é importante para a implementação de sistemas de reescrita. Para sistemas canônicos (terminantes e confluentes), qualquer estratégia que seja completa (i.e., sempre reescreve um redex se um existe até que uma forma normal seja alcançada) levará à mesma forma normal única. No entanto, a eficiência pode variar. Para sistemas que são apenas confluentes mas não necessariamente terminantes, a estratégia `outermost` tem maior probabilidade de encontrar uma forma normal se ela existir.

**Exercício:**

Considere o TAD `Natural` com as regras:
(rN3): `add(n, zero) $\rightarrow$ n`
(rN4): `add(n, succ(m)) $\rightarrow$ succ(add(n, m))`
E uma operação adicional `double(k) $\rightarrow$ add(k, k)`.
Para o termo $t = \text{double}(\text{add(succ(zero), zero))}$.

Mostre uma sequência de reescrita para $t$ usando a estratégia `innermost`.
Mostre uma sequência de reescrita para $t$ usando a estratégia `outermost`.
Compare os resultados e o número de passos.

**Resolução:**

Regras:
(R_double): `double(k) $\rightarrow$ add(k, k)`
(rN3): `add(n, zero) $\rightarrow$ n`
(rN4): `add(n, succ(m)) $\rightarrow$ succ(add(n, m))`

Termo inicial: $t = \text{double}(\text{add(succ(zero), zero))}$

**Estratégia `innermost`:**
Um redex é `innermost` se não contém outros redexes.
No termo $t = \text{double}(\text{add(succ(zero), zero))}$:
*   O subtermo `add(succ(zero), zero)` é um redex para (rN3) com $n \mapsto \text{succ(zero)}$.
*   Este é o único redex `innermost`.

1.  $t_0 = \text{double}(\text{add(succ(zero), zero))}$
    Aplicando (rN3) ao subtermo `add(succ(zero), zero)`:
    `add(succ(zero), zero) $\rightarrow_{rN3}$ succ(zero)`
    Substituindo no termo original:
    $t_1 = \text{double}(\text{succ(zero))}$
    (Passo: `double(add(succ(zero), zero)) $\rightarrow_{rN3}$ double(succ(zero))`)

2.  Termo atual: $t_1 = \text{double}(\text{succ(zero))}$
    *   O único redex é o termo inteiro `double(succ(zero))`, que casa com (R_double) com $k \mapsto \text{succ(zero)}$.
    Aplicando (R_double):
    `double(succ(zero)) $\rightarrow_{R\_double}$ add(succ(zero), succ(zero))`
    $t_2 = \text{add(succ(zero), succ(zero))}$
    (Passo: `double(succ(zero)) $\rightarrow_{R\_double}$ add(succ(zero), succ(zero))`)

3.  Termo atual: $t_2 = \text{add(succ(zero), succ(zero))}$
    *   O único redex é o termo inteiro, que casa com (rN4) com $n \mapsto \text{succ(zero)}$ e $m \mapsto \text{zero}$.
    Aplicando (rN4):
    `add(succ(zero), succ(zero)) $\rightarrow_{rN4}$ succ(add(succ(zero), zero))`
    $t_3 = \text{succ(add(succ(zero), zero))}$
    (Passo: `add(succ(zero), succ(zero)) $\rightarrow_{rN4}$ succ(add(succ(zero), zero))`)

4.  Termo atual: $t_3 = \text{succ(add(succ(zero), zero))}$
    *   O subtermo `add(succ(zero), zero)` é um redex `innermost` para (rN3).
    Aplicando (rN3):
    `add(succ(zero), zero) $\rightarrow_{rN3}$ succ(zero)`
    Substituindo:
    $t_4 = \text{succ(succ(zero))}$
    (Passo: `succ(add(succ(zero), zero)) $\rightarrow_{rN3}$ succ(succ(zero))`)

O termo $t_4 = \text{succ(succ(zero))}$ é uma forma normal.
Sequência `innermost`: $\text{double}(\text{add(succ(zero), zero)}) \rightarrow \text{double}(\text{succ(zero)}) \rightarrow \text{add(succ(zero), succ(zero))} \rightarrow \text{succ(add(succ(zero), zero))} \rightarrow \text{succ(succ(zero))}$.
Total de **4 passos**. Forma normal: `succ(succ(zero))`.

**Estratégia `outermost`:**
Um redex é `outermost` se não é subtermo de outro redex.
No termo $t = \text{double}(\text{add(succ(zero), zero))}$:
*   O redex `outermost` é o termo inteiro, que casa com (R_double) com $k \mapsto \text{add(succ(zero), zero)}$.

1.  $t_0 = \text{double}(\text{add(succ(zero), zero))}$
    Aplicando (R_double):
    `double(add(succ(zero), zero)) $\rightarrow_{R\_double}$ add(add(succ(zero), zero), add(succ(zero), zero))`
    $t'_1 = \text{add(add(succ(zero), zero), add(succ(zero), zero))}$
    (Passo: `double(add(succ(zero), zero)) $\rightarrow_{R\_double}$ add(add(succ(zero), zero), add(succ(zero), zero))`)

2.  Termo atual: $t'_1 = \text{add(add(succ(zero), zero), add(succ(zero), zero))}$
    *   Precisamos de uma regra para desambiguar qual `add` reescrever. Assumimos uma ordem da esquerda para a direita para redexes `outermost` no mesmo nível.
    *   O primeiro subtermo `add(succ(zero), zero)` (argumento esquerdo do `add` principal) é um redex `outermost` dentro desse argumento.
    Aplicando (rN3) a ele:
    `add(succ(zero), zero) $\rightarrow_{rN3}$ succ(zero)`
    Substituindo:
    $t'_2 = \text{add(succ(zero), add(succ(zero), zero))}$
    (Passo: `add(add(succ(zero), zero), add(succ(zero), zero)) $\rightarrow_{rN3}$ add(succ(zero), add(succ(zero), zero))`)

3.  Termo atual: $t'_2 = \text{add(succ(zero), add(succ(zero), zero))}$
    *   O subtermo `add(succ(zero), zero)` (argumento direito do `add` principal) é um redex `outermost` dentro desse argumento.
    Aplicando (rN3) a ele:
    `add(succ(zero), zero) $\rightarrow_{rN3}$ succ(zero)`
    Substituindo:
    $t'_3 = \text{add(succ(zero), succ(zero))}$
    (Passo: `add(succ(zero), add(succ(zero), zero)) $\rightarrow_{rN3}$ add(succ(zero), succ(zero))`)
    (Nota: Este é o mesmo termo $t_2$ da estratégia `innermost`)

4.  Termo atual: $t'_3 = \text{add(succ(zero), succ(zero))}$
    *   Redex `outermost` é o termo inteiro. Aplicando (rN4) com $n \mapsto \text{succ(zero)}, m \mapsto \text{zero}$:
    $t'_4 = \text{succ(add(succ(zero), zero))}$
    (Passo: `add(succ(zero), succ(zero)) $\rightarrow_{rN4}$ succ(add(succ(zero), zero))`)

5.  Termo atual: $t'_4 = \text{succ(add(succ(zero), zero))}$
    *   Redex `outermost` (e `innermost`) é `add(succ(zero), zero)`. Aplicando (rN3):
    $t'_5 = \text{succ(succ(zero))}$
    (Passo: `succ(add(succ(zero), zero)) $\rightarrow_{rN3}$ succ(succ(zero))`)

O termo $t'_5 = \text{succ(succ(zero))}$ é uma forma normal.
Sequência `outermost`: $\text{double}(\text{add(succ(zero), zero)}) \rightarrow \text{add(add(succ(zero), zero), add(succ(zero), zero))} \rightarrow \text{add(succ(zero), add(succ(zero), zero))} \rightarrow \text{add(succ(zero), succ(zero))} \rightarrow \text{succ(add(succ(zero), zero))} \rightarrow \text{succ(succ(zero))}$.
Total de **5 passos**. Forma normal: `succ(succ(zero))`.

**Comparação:**
*   Ambas as estratégias levaram à mesma forma normal `succ(succ(zero))`. Isso é esperado se o sistema for confluente.
*   A estratégia `innermost` levou 4 passos.
*   A estratégia `outermost` levou 5 passos.
Neste caso particular, a estratégia `innermost` foi ligeiramente mais eficiente em termos de número de passos. A estratégia `outermost` duplicou o subtermo `add(succ(zero), zero)` no primeiro passo, o que levou a reescrevê-lo duas vezes. Se esse subtermo fosse muito custoso para reduzir, `innermost` seria vantajosa. Contudo, se a regra `double` descartasse seu argumento (o que não é o caso aqui), `outermost` poderia ter evitado reduzir o argumento desnecessariamente.

# 3.3 Propriedades Fundamentais de Sistemas de Reescrita de Termos

Para que um Sistema de Reescrita de Termos (SRT) seja útil como um modelo de computação ou como uma ferramenta para decidir a equivalência em uma teoria equacional, ele idealmente deve possuir certas propriedades fundamentais. As três propriedades mais importantes são: **terminação** (ou noetherianidade), que garante que toda computação (sequência de reescritas) eventualmente para; **confluência** (ou a propriedade de Church-Rosser), que garante que a ordem de aplicação das regras não afeta o resultado final (se ele existir e for único); e **canonicidade** (ou convergência), que é a combinação de terminação e confluência, implicando que todo termo tem uma forma normal única que pode ser alcançada independentemente da estratégia de reescrita (contanto que a estratégia seja completa). Estas propriedades são cruciais para a confiabilidade e a previsibilidade dos sistemas de reescrita. Esta seção explorará cada uma dessas propriedades em detalhe, incluindo o importante Lema de Newman que relaciona a confluência local com a confluência global sob a condição de terminação.

**Terminação (Noetherianity)**

Um Sistema de Reescrita de Termos $R$ é dito **terminante** (ou **noetheriano**, ou **fortemente normalizador**, *strongly normalizing* - SN) se não existe nenhuma sequência infinita de reescritas. Ou seja, para qualquer termo inicial $t_0$, toda sequência de reescritas $t_0 \rightarrow_R t_1 \rightarrow_R t_2 \rightarrow_R \ldots$ deve ser finita.

Se um SRT é terminante, então todo termo possui pelo menos uma forma normal (um termo irredutível que não pode mais ser reescrito). Isso ocorre porque, se um termo não estivesse em forma normal, ele poderia ser reescrito; se todas as sequências de reescrita terminam, elas devem terminar em uma forma normal.

**Importância da Terminação:**
*   **Garantia de Resultado:** A terminação garante que o processo de "cálculo" por reescrita sempre produzirá um resultado (uma forma normal) em um tempo finito.
*   **Base para Outras Propriedades:** A terminação é frequentemente um pré-requisito para provar outras propriedades importantes, como a confluência (via Lema de Newman).
*   **Procedimentos de Decisão:** Se um SRT é terminante e confluente, ele fornece um procedimento de decisão para a teoria equacional correspondente: para verificar se $t_1 = t_2$, basta reescrever ambos para suas formas normais únicas e compará-las. Sem terminação, este procedimento não seria garantido de parar.

**Provar a Terminação:**
Provar a terminação de um SRT pode ser uma tarefa desafiadora. A técnica mais comum é encontrar uma **ordenação de redução** $>_{ord}$, que é uma relação de ordem sobre o conjunto de termos $T_{\Sigma}(X)$ tal que:
1.  $>_{ord}$ é **bem fundada** (well-founded): não existe nenhuma sequência infinita decrescente $t_0 >_{ord} t_1 >_{ord} t_2 >_{ord} \ldots$.
2.  $>_{ord}$ é **compatível com a estrutura do SRT** (monotonicidade com operações): se $s >_{ord} t$, então $C[s]\sigma >_{ord} C[t]\sigma$ para qualquer contexto $C[\_]$ (um termo com uma lacuna) e qualquer substituição $\sigma$. (Contexto e substituição significam que a ordenação se mantém se os termos são partes de termos maiores ou se variáveis são instanciadas).
3.  $>_{ord}$ é **estável sob substituição**: se $s >_{ord} t$, então $s\sigma >_{ord} t\sigma$ para qualquer substituição $\sigma$.
4.  $>_{ord}$ **contém as regras de reescrita**: para toda regra $l \rightarrow r$ em $R$ e toda substituição $\sigma$ para as variáveis em $l$ e $r$, deve-se ter $l\sigma >_{ord} r\sigma$.

Se tal ordenação de redução existe, então o SRT $R$ é terminante. Encontrar uma ordenação de redução adequada pode ser difícil. Algumas ordenações comuns usadas para provar terminação incluem:
*   **Ordenações Polinomiais:** Mapeiam termos para polinômios sobre os números naturais e comparam os valores dos polinômios.
*   **Ordenações de Caminho Recursivas (RPO - Recursive Path Ordering):** Comparam termos com base em sua estrutura de árvore, lexicograficamente. Variações incluem LPO (Lexicographic Path Ordering) e MPO (Multiset Path Ordering).
*   **Método de Knuth-Bendix (KBO - Knuth-Bendix Ordering):** Associa pesos a símbolos de função e usa uma precedência entre eles.

**Exemplo (Não-Terminação):**
Se tivéssemos uma regra $x \rightarrow f(x)$, o sistema seria não terminante: $a \rightarrow f(a) \rightarrow f(f(a)) \rightarrow \ldots$.
A regra $add(x,y) \rightarrow add(y,x)$ derivada do axioma de comutatividade também levaria à não-terminação, como visto antes.

A terminação é uma propriedade global do sistema de reescrita (todas as sequências devem terminar). Um sistema pode ser **fracamente terminante** (ou fracamente normalizador - WN) se para todo termo $t$, *existe pelo menos uma* sequência de reescrita finita que leva a uma forma normal, mesmo que outras sequências possam ser infinitas. A terminação forte (SN) é geralmente a propriedade desejada.

**Confluência (Propriedade de Church-Rosser) e o Lema de Newman**

Um Sistema de Reescrita de Termos $R$ é dito **confluente** (ou possuir a **propriedade de Church-Rosser**) se, para quaisquer termos $s, t_1, t_2$ tais que $s \rightarrow_R^* t_1$ e $s \rightarrow_R^* t_2$ (ou seja, $t_1$ e $t_2$ são ambos deriváveis de $s$ por zero ou mais passos de reescrita), então existe um termo $u$ tal que $t_1 \rightarrow_R^* u$ e $t_2 \rightarrow_R^* u$.
Isso pode ser visualizado com um diagrama:
```
      s
     / \
    /   \
   *↓   *↓  (onde *↓ denota  →_R^*)
  /       \
 t_1     t_2
  \       /
   *↓   *↓
    \   /
     \ /
      u
```
A propriedade de confluência significa que, não importa como escolhamos aplicar as regras de reescrita a um termo (se houver múltiplas opções), se as sequências de reescrita divergem, elas eventualmente "reconvergem" para um descendente comum.

**Importância da Confluência:**
*   **Unicidade da Forma Normal:** Se um SRT é confluente, então todo termo tem **no máximo uma** forma normal. Se um termo $s$ tem duas formas normais $nf_1$ e $nf_2$, então $s \rightarrow_R^* nf_1$ e $s \rightarrow_R^* nf_2$. Pela confluência, deve existir um $u$ tal que $nf_1 \rightarrow_R^* u$ e $nf_2 \rightarrow_R^* u$. Mas como $nf_1$ e $nf_2$ são formas normais (irredutíveis), as únicas reescritas possíveis são $nf_1 \rightarrow_R^* nf_1$ (0 passos) e $nf_2 \rightarrow_R^* nf_2$ (0 passos). Isso implica $u=nf_1$ e $u=nf_2$, logo $nf_1 = nf_2$.
*   **Independência da Estratégia (para alcançar a forma normal):** Se um sistema é confluente e terminante, então qualquer estratégia de reescrita que seja completa (reduz até uma forma normal) levará à mesma forma normal única.

**Confluência Local (Weak Confluence ou Weak Church-Rosser Property):**
Provar a confluência diretamente pode ser difícil. Uma propriedade mais fraca, chamada **confluência local**, é muitas vezes mais fácil de verificar.
Um SRT $R$ é **localmente confluente** se, para quaisquer termos $s, t_1, t_2$ tais que $s \rightarrow_R t_1$ (um passo) e $s \rightarrow_R t_2$ (um passo), então existe um termo $u$ tal que $t_1 \rightarrow_R^* u$ e $t_2 \rightarrow_R^* u$.
```
      s
     / \
    /   \
   ↓     ↓  (onde ↓ denota  →_R, um passo)
  /       \
 t_1     t_2
  \       /
   *↓   *↓
    \   /
     \ /
      u
```
A confluência local considera apenas divergências que ocorrem em um único passo a partir de $s$.

**Lema de Newman:**
O Lema de Newman é um resultado fundamental que conecta a confluência local com a confluência (global) sob a condição de terminação:
**Lema de Newman:** Um Sistema de Reescrita de Termos **terminante** é confluente se e somente se ele é localmente confluente.

Este lema é extremamente útil porque:
1.  A terminação pode, às vezes, ser provada usando ordenações de redução.
2.  A confluência local pode ser verificada analisando **pares críticos**. Um par crítico surge quando dois lados esquerdos de regras (ou o mesmo lado esquerdo em diferentes posições) se sobrepõem em um termo. Se todos os pares críticos de um sistema são "juntáveis" (i.e., seus componentes podem ser reescritos para um termo comum), então o sistema é localmente confluente (este é o critério de Knuth-Bendix para confluência local).

**Exemplo (Não-Confluência):**
Considere as regras:
(R1): `a $\rightarrow$ b`
(R2): `a $\rightarrow$ c`
(R3): `b $\rightarrow$ d`
Assuma que `c` e `d` são formas normais e $c \neq d$.
O termo `a` pode ser reescrito:
*   `a $\rightarrow_{R1}$ b $\rightarrow_{R3}$ d`
*   `a $\rightarrow_{R2}$ c`
Temos $a \rightarrow^* d$ e $a \rightarrow^* c$. Como $d$ e $c$ são formas normais distintas e não podem ser mais reescritas, não existe um descendente comum $u$. Logo, este sistema não é confluente. O termo `a` tem duas formas normais, `d` e `c`.

A confluência é essencial para que um SRT defina uma função única (de termos para suas formas normais) e para que a noção de "valor" de um termo seja bem definida.

**Canonicidade (Convergência)**

Um Sistema de Reescrita de Termos $R$ é dito **canônico** (ou **convergente**, ou *complete* em alguma literatura mais antiga, embora "complete" agora tenha outros significados) se ele é simultaneamente:
1.  **Terminante** (Noetheriano): Toda sequência de reescritas é finita.
2.  **Confluente** (Propriedade de Church-Rosser): Se $s \rightarrow_R^* t_1$ e $s \rightarrow_R^* t_2$, então existe $u$ tal que $t_1 \rightarrow_R^* u$ e $t_2 \rightarrow_R^* u$.

**Importância da Canonicidade:**
*   **Existência e Unicidade de Formas Normais:** Se um SRT $R$ é canônico, então todo termo $t$ possui **exatamente uma** forma normal única, frequentemente denotada como $t \downarrow_R$ ou $NF_R(t)$.
*   **Procedimento de Decisão para Equivalência:** A canonicidade fornece um algoritmo efetivo para decidir a equivalência de dois termos $t_1$ e $t_2$ com respeito à teoria equacional $E$ da qual $R$ foi derivada (assumindo que $R$ é "correto e completo" para $E$). Para verificar se $E \models t_1 = t_2$, basta calcular as formas normais $NF_R(t_1)$ e $NF_R(t_2)$ e verificar se elas são sintaticamente idênticas. Como o sistema é terminante, o cálculo das formas normais termina. Como é confluente, as formas normais são únicas, então a comparação é definitiva.
*   **Semântica Bem Definida:** Um SRT canônico define uma semântica operacional clara e não ambígua para uma especificação algébrica. A forma normal de um termo pode ser considerada seu "valor" ou sua interpretação canônica.

**Obtendo Canonicidade:**
O processo de transformar um conjunto de equações $E$ em um SRT canônico $R$ é o objetivo do **algoritmo de completamento de Knuth-Bendix**. Este algoritmo tenta orientar as equações em regras e, em seguida, analisa os pares críticos. Se um par crítico $(p, q)$ não é juntável (i.e., $p$ e $q$ não podem ser reescritos para um termo comum), o algoritmo tenta adicionar uma nova equação $p'=q'$ (onde $p \rightarrow^* p'$ e $q \rightarrow^* q'$, e $p', q'$ são formas normais parciais) ao sistema e reorientá-la como uma nova regra. Este processo pode:
1.  Terminar com sucesso, produzindo um SRT canônico.
2.  Terminar com falha (e.g., se uma equação não pode ser orientada sem violar a ordenação de redução que garante a terminação).
3.  Não terminar (gerando infinitas regras).

**Exemplo de Sistema Canônico:**
O sistema $R_{Natural}$ com regras:
(rN1): `isZero(zero) $\rightarrow$ true`
(rN2): `isZero(succ(n)) $\rightarrow$ false`
(rN3): `add(n, zero) $\rightarrow$ n`
(rN4): `add(n, succ(m)) $\rightarrow$ succ(add(n, m))`
É geralmente considerado canônico.
*   **Terminação:** Pode ser provada usando uma ordenação de caminho recursiva (RPO) ou uma ordenação polinomial, onde `succ(n)` é maior que `n`, e `add(n, succ(m))` é maior que `succ(add(n,m))` porque o segundo argumento de `add` diminui estruturalmente.
*   **Confluência:** Pode ser provada mostrando que todos os seus pares críticos são juntáveis (ou usando o Lema de Newman, já que é terminante, e provando confluência local). Para este sistema, não há sobreposições críticas entre os lados esquerdos das regras que levem a divergências não juntáveis.

Portanto, para $R_{Natural}$, todo termo como `add(succ(zero), succ(zero))` tem uma forma normal única (`succ(succ(zero))`) que é alcançada independentemente da estratégia de reescrita (desde que seja completa).

A canonicidade é a "medalha de ouro" para sistemas de reescrita, pois ela garante as melhores propriedades computacionais e de decisão. Muitas das ferramentas e técnicas em especificação algébrica e programação funcional dependem, implícita ou explicitamente, da existência (ou da busca por) sistemas de reescrita canônicos.

**Exercício:**

Considere o seguinte Sistema de Reescrita de Termos (SRT) $R_{ex}$:
Regras:
(R1): `f(g(x)) $\rightarrow$ x`
(R2): `g(a) $\rightarrow$ c`
(R3): `f(c) $\rightarrow$ d`
Onde `a, c, d` são constantes e `x` é uma variável.

a) O termo `f(g(a))` pode ser reescrito de duas maneiras diferentes em um passo. Mostre essas duas reescritas.
b) Investigue se este sistema é localmente confluente para o termo `f(g(a))`. Ou seja, os dois termos resultantes do item (a) podem ser reescritos para um descendente comum?
c) Com base na sua análise, o que você pode inferir sobre a confluência global deste sistema (sem necessariamente prová-la ou a terminação)?

**Resolução:**

Termo inicial: $s = f(g(a))$

**a) Duas maneiras diferentes de reescrever $s = f(g(a))$ em um passo:**

**Reescrita 1 (Aplicando R1 primeiro, se possível, ou R2 no subtermo interno):**
*   O subtermo $g(a)$ em $f(g(a))$ é um redex para a regra (R2): `g(a) $\rightarrow$ c`.
    Aplicando (R2) ao subtermo $g(a)$:
    $f(g(a)) \rightarrow_{R2} f(c)$
    Chamaremos este resultado $t_1 = f(c)$.

**Reescrita 2 (Aplicando R1 ao termo inteiro, se $g(a)$ for substituível por $x$):**
*   O termo inteiro $f(g(a))$ casa com o lado esquerdo da regra (R1): `f(g(x)) $\rightarrow$ x`, com a substituição $\sigma(x) = a$.
    Aplicando (R1) com $\sigma(x) = a$:
    $f(g(a)) \rightarrow_{R1} a$
    Chamaremos este resultado $t_2 = a$.

Portanto, as duas reescritas em um passo a partir de $s = f(g(a))$ são:
1.  $f(g(a)) \rightarrow_{R2} f(c)$
2.  $f(g(a)) \rightarrow_{R1} a$

**b) Investigando a confluência local para $s = f(g(a))$:**
Temos $s \rightarrow t_1 = f(c)$ e $s \rightarrow t_2 = a$.
Para que o sistema seja localmente confluente *neste ponto de divergência*, $t_1$ e $t_2$ devem ser reescritos para um descendente comum $u$.

*   Considerando $t_1 = f(c)$:
    $f(c)$ é um redex para a regra (R3): `f(c) $\rightarrow$ d`.
    Então, $f(c) \rightarrow_{R3} d$.
    O termo $d$ é uma constante, presumivelmente uma forma normal (não casa com o lado esquerdo de nenhuma regra).
    Assim, $t_1 \rightarrow^* d$.

*   Considerando $t_2 = a$:
    O termo $a$ é uma constante. Não há regras em $R_{ex}$ cujo lado esquerdo seja `a` ou contenha `a` de forma que `a` seja um redex por si só.
    Portanto, $a$ é uma forma normal.
    Assim, $t_2 \rightarrow^* a$.

Agora temos:
$t_1 \rightarrow^* d$
$t_2 \rightarrow^* a$

Para que haja um descendente comum $u$, precisaríamos que $d \rightarrow^* u$ e $a \rightarrow^* u$.
Como $d$ e $a$ são formas normais (e assumimos $d \neq a$, pois são constantes distintas), o único modo de terem um descendente comum é se $d=a$. Se $d \neq a$, então não há descendente comum além deles mesmos (e eles são diferentes).

Assumindo que $a, c, d$ são constantes distintas, então $d \neq a$.
Portanto, $t_1 = f(c)$ e $t_2 = a$ não têm um descendente comum (além do próprio $f(g(a))$ na relação de equivalência, mas aqui buscamos na direção da reescrita).
A situação é:
```
      f(g(a))
     /       \
    /         \
   ↓R2         ↓R1
  /             \
f(c)             a  (forma normal)
 ↓R3
 d (forma normal)
```
Como $d \neq a$, o sistema **não é localmente confluente** para o termo $f(g(a))$, pois a divergência $f(c)$ vs $a$ não converge.

**c) Inferência sobre a confluência global:**

Se um sistema de reescrita não é localmente confluente, ele **não pode ser globalmente confluente**. A confluência global implica a confluência local (se $s \rightarrow t_1$ e $s \rightarrow t_2$, então, como $\rightarrow$ é um caso especial de $\rightarrow^*$, pela confluência global $t_1$ e $t_2$ devem ter um descendente comum).
Como demonstramos que $R_{ex}$ não é localmente confluente a partir do termo $f(g(a))$, podemos inferir que o sistema $R_{ex}$ **não é globalmente confluente**.

O termo $f(g(a))$ pode ser reduzido a duas formas normais distintas: $d$ e $a$. Isso é uma característica de sistemas não confluentes. A unicidade da forma normal não é garantida.

# 3.4 Aplicação da Reescrita na Análise de Especificações Algébricas

A teoria dos Sistemas de Reescrita de Termos (SRTs) não é apenas um modelo abstrato de computação; ela também oferece ferramentas analíticas poderosas quando aplicada a especificações algébricas de Tipos Abstratos de Dados (TADs). Ao orientar os axiomas equacionais de uma especificação $SP = (\Sigma, E)$ em um conjunto de regras de reescrita $R$, podemos usar as propriedades do SRT $R$ (como terminação, confluência e canonicidade) para inferir características importantes sobre a própria especificação $SP$. Esta conexão é fundamental porque muitas propriedades desejáveis de uma especificação, como consistência e certos tipos de completude, podem ser mais facilmente analisadas ou mesmo decididas através da análise do comportamento dinâmico do sistema de reescrita associado. Por exemplo, um sistema de reescrita canônico derivado de uma especificação pode fornecer um procedimento de decisão para a igualdade de termos na teoria equacional definida pelos axiomas, e a natureza das formas normais pode revelar insights sobre a estrutura dos valores do TAD. Esta seção explorará como a reescrita de termos é aplicada na análise de especificações algébricas, focando em como as propriedades dos SRTs se traduzem em informações valiosas sobre a qualidade e o comportamento da especificação original.

**Análise de Consistência via Confluência e Terminação:**
Uma das primeiras preocupações ao definir uma especificação algébrica $SP = (\Sigma, E)$ é sua **consistência**. Informalmente, uma especificação é consistente se ela não permite a derivação de contradições, como, por exemplo, `true = false` no TAD `Boolean`. Se tal contradição puder ser derivada, a especificação é trivial e inútil, pois todos os termos de um determinado sort se tornariam equivalentes.

A reescrita de termos oferece uma maneira de investigar a consistência. Se pudermos transformar o conjunto de axiomas $E$ em um Sistema de Reescrita de Termos $R$ que seja **canônico** (terminante e confluente), então temos uma ferramenta poderosa:
*   **Procedimento de Decisão para Igualdade:** Em um SRT canônico, dois termos $t_1$ e $t_2$ são iguais na teoria equacional definida por $E$ (i.e., $E \models t_1 = t_2$) se e somente se suas formas normais únicas em $R$ são idênticas ($NF_R(t_1) \equiv NF_R(t_2)$).
*   **Verificação de Consistência:** Para verificar a consistência, podemos calcular as formas normais de termos que não deveriam ser iguais. Por exemplo, para o TAD `Boolean`, calculamos $NF_R(\text{true})$ e $NF_R(\text{false})$. Se o sistema $R$ é canônico e derivado corretamente dos axiomas de `Boolean`, esperamos que $NF_R(\text{true})$ seja (ou seja isomorfo a) `true` e $NF_R(\text{false})$ seja `false`, e que `true` $\not\equiv$ `false`. Se, ao contrário, $NF_R(\text{true}) \equiv NF_R(\text{false})$, então a especificação original $E$ é inconsistente, pois implica `true = false`.

Mesmo que não se consiga provar a canonicidade completa, a simples existência de um modelo (uma Σ-álgebra que satisfaz todos os axiomas) já é uma forte indicação de consistência. A construção de um sistema de reescrita terminante a partir dos axiomas e a análise de suas formas normais podem ajudar a construir ou a validar tal modelo (e.g., o modelo de termos em forma normal).

**Análise de Completude Suficiente:**
Outra propriedade desejável de especificações, especialmente aquelas que definem TADs indutivos (como `Natural` ou `NaturalList`), é a **completude suficiente** (sufficient completeness ou constructor completeness). Informalmente, uma especificação é suficientemente completa se toda operação (especialmente observadores e operações derivadas) aplicada a termos construídos puramente a partir de construtores pode ser "reduzida" a um termo que não envolve essa operação, mas sim construtores de sorts de resultado (como `true`/`false` para `Boolean`, ou termos numéricos para `Natural`). Essencialmente, isso significa que o comportamento das operações é definido para todas as entradas "válidas" (construídas).

A reescrita de termos é fundamental aqui:
*   Se um SRT $R$ derivado da especificação $SP$ é **terminante**, então todo termo fundamental $t$ (formado por construtores e a operação em questão) pode ser reduzido a uma forma normal $NF_R(t)$.
*   A especificação é suficientemente completa com respeito a uma operação $f$ se, para todos os termos $t$ da forma $f(c_1, \ldots, c_k)$ onde os $c_i$ são termos construtores (ou formas normais de outros sorts), a forma normal $NF_R(t)$ não contém $f$ e, idealmente, é um termo construtor do sort de resultado de $f$ (ou um termo fundamental de um sort base como `Boolean` ou `Natural`).
*   Por exemplo, para `isZero(n)` no TAD `Natural`, a completude suficiente exige que para qualquer $n$ construído por `zero` ou `succ`, `isZero(n)` se reduza a `true` ou `false`. As regras `isZero(zero) $\rightarrow$ true` e `isZero(succ(m)) $\rightarrow$ false` garantem isso.

A análise de completude utilizando técnicas de reescrita (como a verificação de "cobertura" dos construtores pelos lados esquerdos das regras para operações definidas) é um tópico importante na teoria de especificações algébricas.

**Derivação de Implementações e Geração de Código:**
Se um sistema de reescrita $R$ é canônico, ele não apenas fornece um procedimento de decisão, mas também pode ser visto como um **protótipo de implementação** do TAD. As regras de reescrita podem ser diretamente traduzidas em código de uma linguagem de programação funcional. A estratégia de reescrita (e.g., `innermost` ou `outermost`) corresponderia à estratégia de avaliação da linguagem (e.g., `call-by-value` ou `call-by-name`/`lazy evaluation`).
*   Linguagens de programação baseadas em reescrita, como Maude, usam diretamente SRTs como seu modelo de execução.
*   Mesmo para linguagens convencionais, um SRT canônico pode guiar a implementação correta das operações do TAD.

**Verificação de Propriedades Adicionais (Teoremas):**
Uma vez que temos um conjunto de axiomas $E$ (e um SRT canônico $R$ associado), podemos querer provar que outras equações $t_1 = t_2$ (que não são axiomas) também são válidas na teoria. Essas são os **teoremas** da especificação.
*   Se $R$ é canônico, provar $E \models t_1 = t_2$ se reduz a mostrar que $NF_R(t_1) \equiv NF_R(t_2)$.
*   Técnicas como **prova por indução sobre a estrutura dos termos** (indução de Noether, se o sistema é terminante) ou **indução sobre a estrutura dos construtores** (structural induction) são frequentemente usadas em conjunto com a reescrita para provar teoremas.

**Geração de Casos de Teste:**
Os axiomas equacionais $l=r$ de uma especificação, quando orientados como regras $l \rightarrow r$, podem ser usados para gerar casos de teste para uma implementação do TAD. Se uma implementação $I$ é correta, então para qualquer instância $l\sigma$ do lado esquerdo de um axioma, o resultado da computação de $l\sigma$ em $I$ deve ser igual ao resultado da computação de $r\sigma$ em $I$. Isso é a base do teste baseado em propriedades, onde os axiomas são as propriedades a serem verificadas (Capítulo 6).

**Limitações e Desafios:**
*   **Orientação de Equações:** Nem todos os conjuntos de equações podem ser facilmente orientados em um sistema de reescrita canônico. Axiomas como comutatividade e associatividade são problemáticos e requerem tratamento especial (e.g., reescrita AC, completamento).
*   **Provar Terminação e Confluência:** Estas são, em geral, propriedades indecidíveis. Embora existam muitas técnicas heurísticas e métodos poderosos (como ordenações e análise de pares críticos), não há um algoritmo universal.
*   **Complexidade:** Mesmo que um SRT canônico exista, o cálculo de formas normais pode ser computacionalmente caro.

Apesar desses desafios, a aplicação da reescrita de termos na análise de especificações algébricas tem sido uma área frutífera de pesquisa e aplicação, fornecendo insights profundos sobre a estrutura e o comportamento dos Tipos Abstratos de Dados e servindo como uma ponte entre a especificação formal e a computação efetiva.

**Exercício:**

Considere a especificação do TAD `Boolean` com as operações `true`, `false`, `not` e `and`, e os axiomas/regras:
(R1): `not(true) $\rightarrow$ false`
(R2): `not(false) $\rightarrow$ true`
(R3): `and(true, b) $\rightarrow$ b`
(R4): `and(false, b) $\rightarrow$ false`
(R5): `and(b, true) $\rightarrow$ b` (Suponha que este também é uma regra)
(R6): `and(b, false) $\rightarrow$ false` (Suponha que este também é uma regra)

Suponha que este sistema de reescita $R_{Bool}$ é canônico (terminante e confluente). Como você usaria este sistema $R_{Bool}$ para verificar se a propriedade (teorema) `and(true, false) = and(false, true)` é válida na especificação `Boolean`? Descreva os passos.

**Resolução:**

Se o sistema de reescrita $R_{Bool}$ (composto pelas regras R1-R6) é canônico, então para verificar se uma equação $t_1 = t_2$ é válida na teoria equacional definida pelos axiomas originais de `Boolean`, basta calcular as formas normais únicas de $t_1$ e $t_2$ usando $R_{Bool}$ e verificar se essas formas normais são sintaticamente idênticas.

A propriedade a ser verificada é: `and(true, false) = and(false, true)`.
Seja $t_1 = \text{and(true, false)}$ e $t_2 = \text{and(false, true)}$.

**Passos:**

1.  **Calcular a forma normal de $t_1 = \text{and(true, false)}$:**
    *   O termo `and(true, false)` casa com o lado esquerdo da regra (R3): `and(true, b) $\rightarrow$ b`, com a substituição $\sigma(b) = \text{false}$.
    *   Aplicando (R3): `and(true, false) $\rightarrow_{R3}$ false`.
    *   O termo `false` é uma constante e não casa com o lado esquerdo de nenhuma das regras (R1-R6) de forma a ser reescrito. Portanto, `false` é uma forma normal.
    *   Assim, $NF_{R_{Bool}}(\text{and(true, false)}) \equiv \text{false}$.

2.  **Calcular a forma normal de $t_2 = \text{and(false, true)}$:**
    *   O termo `and(false, true)` casa com o lado esquerdo da regra (R4): `and(false, b) $\rightarrow$ false`, com a substituição $\sigma(b) = \text{true}$.
    *   Aplicando (R4): `and(false, true) $\rightarrow_{R4}$ false`.
    *   O termo `false` é uma forma normal.
    *   Assim, $NF_{R_{Bool}}(\text{and(false, true)}) \equiv \text{false}$.

3.  **Comparar as formas normais:**
    Temos $NF_{R_{Bool}}(t_1) \equiv \text{false}$ e $NF_{R_{Bool}}(t_2) \equiv \text{false}$.
    Como as formas normais são sintaticamente idênticas (`false` $\equiv$ `false`), concluímos que a equação original `and(true, false) = and(false, true)` é válida na especificação `Boolean` (assumindo que $R_{Bool}$ é canônico e corretamente derivado dos axiomas que definem `Boolean`).

Este processo ilustra como um sistema de reescrita canônico fornece um procedimento de decisão para a teoria equacional: reduzir ambos os lados da equação à forma normal e comparar. Se as formas normais são as mesmas, a equação é uma consequência dos axiomas.

# 3.5 Exemplo Detalhado: Reescrita na Especificação de `Natural`

Para solidificar a compreensão dos conceitos de sistemas de reescrita de termos, regras, sequências de reescrita, formas normais e as propriedades de terminação e confluência, esta seção apresentará um exemplo detalhado focado na especificação do Tipo Abstrato de Dados `Natural` (números naturais de Peano). Utilizaremos a especificação `Natural` introduzida no Capítulo 2, que inclui as operações `zero` (constante), `succ` (sucessor), `add` (adição) e `isZero` (predicado para verificar se um natural é zero). Primeiramente, derivaremos um conjunto de regras de reescrita a partir dos axiomas de `add` e `isZero`. Em seguida, demonstraremos algumas sequências de reescrita para termos específicos, ilustrando como eles são reduzidos a formas normais. Finalmente, discutiremos informalmente (pois uma prova formal está além do escopo desta seção introdutória) as propriedades de terminação e confluência para este sistema de reescrita particular, destacando por que ele é geralmente bem-comportado e adequado para definir a semântica operacional dos números naturais e da adição.

**Derivando Regras de Reescrita dos Axiomas de `add` e `isZero`**

Relembremos os axiomas relevantes da especificação `Natural` (do Capítulo 2, seção 2.5), que também importa `Boolean` com suas constantes `true` e `false`:

Axiomas para `Natural`:
*   (N1): `isZero(zero) = true`
*   (N2): `isZero(succ(n)) = false`
*   (N3): `add(n, zero) = n`
*   (N4): `add(n, succ(m)) = succ(add(n, m))`
(for all n, m: Natural)

A partir desses axiomas, podemos derivar (orientar) o seguinte conjunto de regras de reescrita $R_{Natural}$:

1.  **Regra (R1) de (N1):**
    `isZero(zero) $\rightarrow$ true`
    *   **Justificativa da Orientação:** O lado direito `true` é uma constante do sort `Boolean`, considerada uma forma normal para esse tipo. O lado esquerdo envolve a aplicação da operação `isZero`. Esta orientação representa uma simplificação ou avaliação direta.

2.  **Regra (R2) de (N2):**
    `isZero(succ(n)) $\rightarrow$ false`
    *   **Justificativa da Orientação:** Similarmente, `false` é uma forma normal booleana. A regra define o comportamento de `isZero` para qualquer número natural não-zero (que deve ser da forma `succ(n)` se `zero` e `succ` são os únicos construtores de `Natural`). A variável `n` no lado esquerdo também aparece (implicitamente, como não sendo necessária) no lado direito, mas como o lado direito é uma constante, a restrição de variáveis é satisfeita.

3.  **Regra (R3) de (N3):**
    `add(n, zero) $\rightarrow$ n`
    *   **Justificativa da Orientação:** O lado direito `n` é um subtermo do lado esquerdo `add(n, zero)` (se considerarmos `n` como uma forma mais simples). Reduz o número de símbolos de operação. Define o caso base para a adição.

4.  **Regra (R4) de (N4):**
    `add(n, succ(m)) $\rightarrow$ succ(add(n, m))`
    *   **Justificativa da Orientação:** Esta regra é crucial para a definição recursiva da adição. O lado direito é considerado "mais simples" ou "mais próximo de um caso base" porque o segundo argumento da operação `add` interna, `m`, é estruturalmente menor que `succ(m)` no lado esquerdo (se $m$ for uma variável representando um natural). A operação `succ` é "movida para fora" da chamada recursiva a `add`. Esta orientação é padrão para definições recursivas e geralmente leva à terminação quando combinada com um caso base como (R3).

O conjunto de regras de reescrita $R_{Natural}$ é então:
*   (R1): `isZero(zero) $\rightarrow$ true`
*   (R2): `isZero(succ(n)) $\rightarrow$ false`
*   (R3): `add(n, zero) $\rightarrow$ n`
*   (R4): `add(n, succ(m)) $\rightarrow$ succ(add(n, m))`

Estas regras serão usadas para demonstrar sequências de reescrita.

**Demonstração de Sequências de Reescrita**

Vamos demonstrar algumas sequências de reescrita usando o sistema $R_{Natural}$.
Para clareza, usaremos $0_N$ para `zero`, $S_N$ para `succ`, $+_N$ para `add`, $?0_N$ para `isZero`, $T_B$ para `true`, e $F_B$ para `false`.

**Exemplo 1: Reescrita de `isZero(succ(zero))`**
Termo inicial: $?0_N(S_N(0_N))$

1.  $s_0 = ?0_N(S_N(0_N))$
    *   O termo $S_N(0_N)$ casa com o padrão $S_N(n)$ na Regra (R2): `isZero(succ(n)) $\rightarrow$ false`, com a substituição $\sigma(n) = 0_N$.
    *   O lado esquerdo da regra (R2), $?0_N(S_N(n))$, com $\sigma(n) = 0_N$, torna-se $?0_N(S_N(0_N))$.
    *   O lado direito da regra (R2) é $F_B$.
    *   Aplicando (R2):
    $?0_N(S_N(0_N)) \rightarrow_{R2} F_B$

O termo $F_B$ é uma forma normal (uma constante booleana).
Sequência: $?0_N(S_N(0_N)) \rightarrow F_B$.

**Exemplo 2: Reescrita de `add(succ(zero), succ(zero))`**
(Isso representa $1 + 1$)
Termo inicial: $+_N(S_N(0_N), S_N(0_N))$

1.  $s_0 = +_N(S_N(0_N), S_N(0_N))$
    *   Este termo casa com o lado esquerdo da Regra (R4): `add(n, succ(m)) $\rightarrow$ succ(add(n, m))`.
    *   Substituição $\sigma_1$: $\sigma_1(n) = S_N(0_N)$ e $\sigma_1(m) = 0_N$.
    *   LHS $\sigma_1$: $+_N(S_N(0_N), S_N(0_N))$.
    *   RHS $\sigma_1$: $S_N(+_N(S_N(0_N), 0_N))$.
    *   Aplicando (R4):
    $+_N(S_N(0_N), S_N(0_N)) \rightarrow_{R4} S_N(+_N(S_N(0_N), 0_N))$
    $s_1 = S_N(+_N(S_N(0_N), 0_N))$

2.  $s_1 = S_N(+_N(S_N(0_N), 0_N))$
    *   O subtermo $+_N(S_N(0_N), 0_N)$ casa com o lado esquerdo da Regra (R3): `add(n, zero) $\rightarrow$ n`.
    *   Substituição $\sigma_2$: $\sigma_2(n) = S_N(0_N)$.
    *   LHS $\sigma_2$: $+_N(S_N(0_N), 0_N)$.
    *   RHS $\sigma_2$: $S_N(0_N)$.
    *   Aplicando (R3) ao subtermo:
    $S_N(+_N(S_N(0_N), 0_N)) \rightarrow_{R3} S_N(S_N(0_N))$
    $s_2 = S_N(S_N(0_N))$

O termo $s_2 = S_N(S_N(0_N))$ (representando o número 2) é uma forma normal, pois não pode mais ser reescrito pelas regras (R1)-(R4).
Sequência: $+_N(S_N(0_N), S_N(0_N)) \rightarrow S_N(+_N(S_N(0_N), 0_N)) \rightarrow S_N(S_N(0_N))$.

**Exemplo 3: Reescrita de `isZero(add(zero, zero))`**
Termo inicial: $?0_N(+_N(0_N, 0_N))$

1.  $s_0 = ?0_N(+_N(0_N, 0_N))$
    *   Estratégia `innermost`: O subtermo $+_N(0_N, 0_N)$ é um redex `innermost`.
    *   Ele casa com a Regra (R3): `add(n, zero) $\rightarrow$ n`.
    *   Substituição $\sigma_3$: $\sigma_3(n) = 0_N$.
    *   LHS $\sigma_3$: $+_N(0_N, 0_N)$.
    *   RHS $\sigma_3$: $0_N$.
    *   Aplicando (R3) ao subtermo:
    $?0_N(+_N(0_N, 0_N)) \rightarrow_{R3} ?0_N(0_N)$
    $s_1 = ?0_N(0_N)$

2.  $s_1 = ?0_N(0_N)$
    *   Este termo casa com o lado esquerdo da Regra (R1): `isZero(zero) $\rightarrow$ true`.
    *   Aplicando (R1):
    $?0_N(0_N) \rightarrow_{R1} T_B$
    $s_2 = T_B$

O termo $T_B$ é uma forma normal.
Sequência: $?0_N(+_N(0_N, 0_N)) \rightarrow ?0_N(0_N) \rightarrow T_B$.

Estas demonstrações ilustram como o processo de reescrita, guiado pelas regras derivadas dos axiomas, pode sistematicamente simplificar termos até que uma forma irredutível (forma normal) seja alcançada. Esta forma normal pode ser considerada o "valor" do termo inicial.

**Discussão sobre Terminação e Confluência para o sistema de exemplo**

O Sistema de Reescrita de Termos $R_{Natural}$ composto pelas regras:
(R1): `isZero(zero) $\rightarrow$ true`
(R2): `isZero(succ(n)) $\rightarrow$ false`
(R3): `add(n, zero) $\rightarrow$ n`
(R4): `add(n, succ(m)) $\rightarrow$ succ(add(n, m))`

É um sistema bem conhecido e possui boas propriedades.

**Terminação:**
O sistema $R_{Natural}$ é **terminante**. Uma prova formal de terminação para este sistema geralmente envolve a definição de uma ordenação de redução bem fundada $>_{ord}$ tal que $l\sigma >_{ord} r\sigma$ para cada regra $l \rightarrow r$ e toda substituição $\sigma$ por termos fundamentais.
*   **Para (R1) e (R2):** `isZero(zero)` e `isZero(succ(n))` podem ser considerados "maiores" que `true` e `false` respectivamente, pois os lados esquerdos envolvem a aplicação de uma operação a um termo `Natural`, enquanto os lados direitos são constantes booleanas simples.
*   **Para (R3):** `add(n, zero)` é "maior" que `n` porque remove a operação `add` e o argumento `zero`. Medidas como o número de símbolos de operação ou o tamanho total do termo diminuem.
*   **Para (R4):** `add(n, succ(m))` é "maior" que `succ(add(n, m))`. Isso pode ser justificado por uma ordenação de caminho lexicográfica (LPO) ou recursiva (RPO). Intuitivamente, a complexidade da adição é reduzida porque o segundo argumento de `add` no lado direito, `m`, é um subtermo direto de `succ(m)` no lado esquerdo. A reescrita "consome" um `succ` do segundo argumento e o move para fora, aproximando-se do caso base (R3) onde o segundo argumento é `zero`. Cada aplicação de (R4) reduz o tamanho do segundo argumento da chamada recursiva a `add`. Como os números naturais (representados por `zero` e `succ`) são bem fundados (não se pode aplicar `succ` "para baixo" infinitamente), a parte da adição terminará.

Como cada regra leva a um termo "menor" em alguma ordenação bem fundada, nenhuma sequência infinita de reescritas pode existir.

**Confluência:**
O sistema $R_{Natural}$ também é **confluente**.
Como ele é terminante, pelo Lema de Newman, basta mostrar que ele é localmente confluente. A confluência local é frequentemente verificada analisando os **pares críticos**. Um par crítico ocorre quando há uma sobreposição (overlap) entre o lado esquerdo de uma regra e um subtermo (não variável) do lado esquerdo de outra regra (ou da mesma regra).
No sistema $R_{Natural}$:
*   Os lados esquerdos são: `isZero(zero)`, `isZero(succ(n))`, `add(n, zero)`, `add(n, succ(m))`.
*   Não há sobreposições entre esses lados esquerdos que criem ambiguidades críticas. Por exemplo, `isZero(T)` só pode casar com (R1) se $T$ for `zero`, ou com (R2) se $T$ for `succ(n)`. Não há termo que case com ambos simultaneamente de maneiras que levem a redexes sobrepostos não triviais. Similarmente para `add(T1, T2)`. Os padrões dos lados esquerdos são suficientemente distintos e não se "aninhando" de forma problemática.

Uma análise formal de pares críticos confirmaria que todos eles são juntáveis (i.e., se $s \rightarrow t_1$ e $s \rightarrow t_2$ devido a uma sobreposição, então $t_1$ e $t_2$ podem ser reescritos para um termo comum $u$). Como $R_{Natural}$ é terminante e (pode-se mostrar) localmente confluente, ele é globalmente confluente.

**Canonicidade:**
Sendo terminante e confluente, o sistema $R_{Natural}$ é **canônico** (ou convergente). Isso implica que:
1.  Todo termo construído sobre a assinatura de `Natural` (e `Boolean`) tem uma **forma normal única**.
2.  Esta forma normal única pode ser alcançada independentemente da estratégia de reescrita utilizada (desde que a estratégia seja completa, como `innermost` ou `outermost`).
Por exemplo, `add(succ(zero), succ(zero))` sempre será reescrito para a forma normal única `succ(succ(zero))`.
`isZero(add(zero,zero))` sempre será reescrito para a forma normal única `true`.

Esta propriedade de canonicidade é o que torna o sistema $R_{Natural}$ um modelo computacional efetivo para a aritmética de Peano e para os predicados definidos. A forma normal única de um termo pode ser considerada seu "valor".

**Exercício:**

Considere o sistema $R_{Natural}$ (regras R1-R4). Para o termo $s = \text{add}(\text{succ(zero)}, \text{isZero(zero))}$.
a) Este termo está bem tipado de acordo com a assinatura de `Natural` (onde `add` espera dois `Natural`s)? Se não, explique por quê.
b) Se o termo fosse (hipoteticamente, ignorando o erro de tipo por um momento, ou assumindo uma linguagem com coerção implícita) `add(succ(zero), M)` onde `M` é um termo que se reescreve para `succ(zero)`, mostre a sequência de reescrita para `add(succ(zero), succ(zero))` até a forma normal.
c) Se um sistema de reescrita não é terminante, o que isso implica sobre a possibilidade de sempre encontrar uma forma normal para qualquer termo?

**Resolução:**

a) **Tipagem do termo $s = \text{add}(\text{succ(zero)}, \text{isZero(zero))}$:**
A assinatura de `add` é `add: Natural x Natural --> Natural`. Isso significa que ambos os argumentos de `add` devem ser do sort `Natural`.
*   O primeiro argumento de `add` em $s$ é `succ(zero)`. `zero` é `Natural`, e `succ: Natural --> Natural`, então `succ(zero)` é do sort `Natural`.
*   O segundo argumento de `add` em $s$ é `isZero(zero)`. `zero` é `Natural`, e `isZero: Natural --> Boolean`. Portanto, o termo `isZero(zero)` é do sort `Boolean`.

Como o segundo argumento `isZero(zero)` é do sort `Boolean` e não `Natural`, o termo $s = \text{add}(\text{succ(zero)}, \text{isZero(zero))}$ **não está bem tipado** de acordo com a assinatura da operação `add`. Não podemos aplicar `add` a um `Natural` e um `Boolean`.

b) **Sequência de reescrita para `add(succ(zero), succ(zero))`:**
Este é o Exemplo 2 da seção 3.5.2. A sequência de reescrita é:
1.  `add(succ(zero), succ(zero))`
    Aplica-se (R4): `add(n, succ(m)) $\rightarrow$ succ(add(n, m))`
    Com substituição $n \mapsto \text{succ(zero)}$, $m \mapsto \text{zero}$.
    $\rightarrow_{R4}$ `succ(add(succ(zero), zero))`

2.  `succ(add(succ(zero), zero))`
    O subtermo `add(succ(zero), zero)` casa com (R3): `add(n, zero) $\rightarrow$ n`.
    Com substituição $n \mapsto \text{succ(zero)}$.
    $\rightarrow_{R3}$ `succ(succ(zero))`

A forma normal é `succ(succ(zero))`.

c) **Implicações da não-terminação:**
Se um sistema de reescrita não é terminante, significa que existe pelo menos um termo $t_0$ a partir do qual existe pelo menos uma sequência de reescrita infinita: $t_0 \rightarrow t_1 \rightarrow t_2 \rightarrow \ldots$.
Isso implica que:
*   **Nem todo termo garante ter uma forma normal.** O termo $t_0$ (ou qualquer termo em uma sequência infinita de reescrita) pode não ter uma forma normal se *todas* as sequências de reescrita a partir dele forem infinitas.
*   Mesmo que um termo $t_0$ *possa* ter uma forma normal (ou seja, existe *alguma* sequência finita de reescrita que leva a uma forma normal – o sistema seria fracamente terminante, mas não fortemente terminante), a existência de uma sequência infinita significa que um procedimento de cálculo que segue essa sequência infinita nunca pararia e, portanto, nunca "encontraria" a forma normal por esse caminho.
*   Se um termo não tem forma normal (porque todas as sequências de reescrita são infinitas), então não se pode usá-lo para decidir a igualdade na teoria equacional de forma algorítmica, pois o "cálculo do valor" não termina.

Em resumo, a não-terminação compromete a capacidade do sistema de reescrita de servir como um procedimento de cálculo universalmente garantido para encontrar "valores" (formas normais) para todos os termos.

Este capítulo explorou os Sistemas de Reescrita de Termos (SRTs) como uma forma de prover uma semântica operacional para especificações algébricas. Iniciamos com a motivação de tratar axiomas como regras de transformação direcionadas, movendo de termos complexos para mais simples, e estabelecemos a relação intrínseca entre reescrita e computação. Formalizamos conceitos chave como regras de reescrita, as relações de reescrita em um e múltiplos passos, e a noção de formas normais como termos irredutíveis. Discutimos as propriedades cruciais de terminação (garantia de fim da computação), confluência (unicidade do resultado independente da ordem de aplicação das regras, via Lema de Newman) e canonicidade (terminação mais confluência, garantindo formas normais únicas). A aplicação da reescita na análise de especificações, como consistência e completude, foi abordada, seguida por um exemplo detalhado com o TAD `Natural`. O próximo capítulo, "Análise de Especificações Algébricas: Corretude e Completeza", aprofundará as propriedades desejáveis das próprias especificações, como consistência e completude suficiente, e como as técnicas de reescrita, entre outras, podem ser usadas para verificá-las.

---
## REFERÊNCIAS BIBLIOGRÁFICAS
BAADER, F.; NIPKOW, T. **Term Rewriting and All That**. Cambridge: Cambridge University Press, 1998.
*   *Resumo: Um texto clássico e abrangente sobre a teoria de reescrita de termos. Cobre em profundidade as definições formais de SRTs, propriedades como terminação e confluência, o Lema de Newman, pares críticos e o algoritmo de Knuth-Bendix, sendo uma referência essencial para todos os tópicos do Capítulo 3.*

DERSHOWITZ, N.; JOUANNAUD, J.-P. Rewrite Systems. In: VAN LEEUWEN, J. (Ed.). **Handbook of Theoretical Computer Science, Volume B: Formal Models and Semantics**. Amsterdam: Elsevier; Cambridge, MA: MIT Press, 1990. p. 243-320.
*   *Resumo: Este capítulo do Handbook é uma pesquisa seminal e altamente citada sobre sistemas de reescrita. Fornece uma visão geral concisa, porém profunda, dos principais conceitos, resultados e técnicas na área, incluindo aqueles discutidos neste capítulo como regras, relações de reescrita e propriedades fundamentais.*

HUET, G. Confluent Reductions: Abstract Properties and Applications to Term Rewriting Systems. **Journal of the ACM**, v. 27, n. 4, p. 797-821, out. 1980.
*   *Resumo: Um artigo fundamental que estabeleceu muitos dos resultados teóricos importantes sobre confluência em sistemas de redução abstratos, com aplicações diretas a sistemas de reescrita de termos. Discute a propriedade de Church-Rosser e sua relação com a confluência local (Lema de Newman em um contexto mais geral).*

KLOP, J. W. Term Rewriting Systems. In: ABRAMSKY, S.; GABBAY, D. M.; MAIBAUM, T. S. E. (Eds.). **Handbook of Logic in Computer Science, Vol. 2: Background: Computational Structures**. Oxford: Oxford University Press, 1992. p. 1-116.
*   *Resumo: Outro extenso capítulo de handbook que serve como uma excelente introdução e referência à teoria de reescrita de termos. Cobre os fundamentos, incluindo as definições de SRTs, terminação, confluência, e discute várias extensões e aplicações, alinhado com o conteúdo do Capítulo 3.*

KNUTH, D. E.; BENDIX, P. B. Simple word problems in universal algebras. In: LEECH, J. (Ed.). **Computational Problems in Abstract Algebra**. Oxford: Pergamon Press, 1970.
. p. 263-297.
*   *Resumo: Este é o artigo original que introduziu o algoritmo de Knuth-Bendix para tentar completar um conjunto de equações em um sistema de reescrita canônico (convergente). É um marco na área, diretamente relacionado à discussão sobre como obter sistemas com boas propriedades (terminação e confluência).*

TERESE. **Term Rewriting Systems**. Cambridge: Cambridge University Press, 2003. (Cambridge Tracts in Theoretical Computer Science, 55).
*   *Resumo: Uma monografia moderna e exaustiva sobre sistemas de reescrita de termos, cobrindo desde os fundamentos até tópicos de pesquisa avançados. É uma excelente referência para um estudo aprofundado de todas as propriedades (terminação, confluência, etc.) e técnicas mencionadas no Capítulo 3.*

---
