---
# Capítulo 2
# Especificação Algébrica de Tipos Abstratos de Dados
---

Após a introdução aos conceitos fundamentais da abstração de dados e à problemática da corretude de software, este capítulo mergulha profundamente na técnica de Especificação Algébrica de Tipos Abstratos de Dados (TADs). Esta abordagem formal, que se distingue pela sua elegância matemática e poder expressivo, permite definir o comportamento de tipos de dados de maneira precisa e independente de qualquer implementação concreta, utilizando para tal os formalismos da lógica equacional e da álgebra universal. Iniciaremos com uma introdução aos pilares conceituais da lógica equacional e das álgebras multissortidas, estabelecendo o vocabulário básico de sorts, operações e assinaturas (Σ-Signatures), elementos cruciais para a construção sintática das especificações. Em seguida, exploraremos a noção de termos sobre uma assinatura, distinguindo entre termos com variáveis e termos fundamentais (`ground terms`), e o papel das substituições e da instanciação na manipulação desses termos. O coração da especificação algébrica reside na definição de axiomas equacionais; detalharemos como estes axiomas, expressos como equações entre termos, capturam a semântica comportamental do TAD, formando, juntamente com a assinatura, uma especificação algébrica SP = (Σ, E). A discussão prosseguirá para a semântica formal dessas especificações, introduzindo o conceito de Σ-álgebra como uma estrutura matemática que provê uma interpretação concreta para os sorts e operações, e definindo a noção de satisfação de equações e de modelo de uma especificação. Serão abordados conceitos semânticos importantes como a classe de todas as Σ-álgebras que satisfazem os axiomas (Alg(SP)) e a significância dos modelos iniciais e finais como interpretações canônicas. Diferenciaremos funcionalmente as operações em construtores, observadores e operações derivadas, elucidando seus papéis na estruturação e compreensão das especificações. Finalmente, para solidificar o entendimento teórico, apresentaremos exemplos fundamentais e detalhados de especificações algébricas, incluindo os TADs `Boolean`, `Natural` (segundo os axiomas de Peano) e `Integer`, ilustrando a aplicação prática dos conceitos introduzidos ao longo do capítulo.

# 2.1 Introdução à Lógica Equacional e Álgebras Multissortidas

A especificação algébrica de Tipos Abstratos de Dados (TADs) fundamenta-se em conceitos robustos oriundos da lógica matemática e da álgebra universal. Dentre estes, a **lógica equacional** e a teoria das **álgebras multissortidas** (many-sorted algebras) desempenham papéis centrais, fornecendo o arcabouço formal necessário para definir a sintaxe e a semântica dos TADs de maneira precisa e abstrata. A lógica equacional é um sistema formal que lida com a igualdade entre termos, permitindo raciocinar sobre as propriedades das operações de um TAD através de um conjunto de axiomas na forma de equações. As álgebras multissortidas, por sua vez, generalizam o conceito clássico de álgebra (que opera sobre um único conjunto de elementos) para estruturas que envolvem múltiplos conjuntos de dados (chamados "sorts" ou "tipos") e operações que podem mapear elementos entre diferentes sorts. Esta generalização é essencial para modelar TADs do mundo real, que frequentemente envolvem interações entre diferentes tipos de dados (e.g., uma lista de inteiros envolve o tipo "lista" e o tipo "inteiro"). A combinação dessas duas ferramentas permite que se especifique um TAD através de uma **assinatura** (que define os sorts e os perfis das operações) e um conjunto de **equações** (axiomas) que descrevem o comportamento dessas operações, independentemente de qualquer representação ou implementação particular. Esta seção introdutória explorará os elementos fundamentais dessa abordagem, estabelecendo as bases para a construção e a análise de especificações algébricas.

**Sorts, Operações e Assinaturas (Σ-Signatures)**

No cerne da especificação algébrica de Tipos Abstratos de Dados (TADs) encontra-se a noção de **assinatura** (signature), formalmente conhecida como Σ-assinatura. Uma assinatura define o vocabulário sintático de um TAD, especificando os tipos de dados envolvidos e as operações que podem ser realizadas sobre eles. Seus componentes primários são os **sorts** e as **operações**.

**Sorts:**
Um **sort** é simplesmente um nome que designa um conjunto de valores ou um tipo de dado. Em uma especificação algébrica, um conjunto $S$ de sorts é declarado para identificar todas as diferentes categorias de dados que são relevantes para o TAD. Por exemplo, ao especificar um TAD para números naturais, poderíamos declarar o sort `Natural`. Para um TAD de listas de números naturais, precisaríamos do sort `NaturalList` e também do sort `Natural` (para os elementos da lista) e, possivelmente, do sort `Boolean` (para o resultado de operações como `isEmpty`). A utilização de múltiplos sorts (daí o termo "multissortido") é crucial para modelar a heterogeneidade dos dados encontrados em sistemas computacionais. Cada sort $s \in S$ corresponde a um domínio de valores, chamado de **conjunto portador** (carrier set), na interpretação semântica (álgebra) da especificação.

**Operações:**
As **operações** (ou símbolos de função) definidas em uma assinatura representam as funções ou procedimentos que podem ser aplicados aos valores dos sorts. Para cada símbolo de operação $f$, a assinatura especifica seu **perfil** ou **aridade**, que indica os sorts de seus argumentos e o sort de seu resultado. Formalmente, se $s_1, s_2, \ldots, s_k, s_{res}$ são sorts em $S$ (com $k \ge 0$), um símbolo de operação $f$ com perfil $f: s_1 \times s_2 \times \ldots \times s_k \rightarrow s_{res}$ denota uma operação que toma $k$ argumentos – o primeiro do sort $s_1$, o segundo do sort $s_2$, e assim por diante, até o $k$-ésimo do sort $s_k$ – e produz um resultado do sort $s_{res}$.
*   Se $k=0$, a operação $f: \rightarrow s_{res}$ é uma **constante** do sort $s_{res}$. Ela não recebe argumentos e representa um valor fixo daquele sort (e.g., `zero: --> Natural`, `true: --> Boolean`).
*   As operações são os blocos de construção para formar **termos**, que representam os cálculos ou valores dentro do TAD.

**Σ-Assinatura:**
Uma **Σ-assinatura** (ou simplesmente assinatura, quando o contexto é claro) é formalmente definida como um par $\Sigma = (S, \Omega)$, onde:
1.  $S$ é um conjunto de nomes de sorts.
2.  $\Omega$ é uma família de conjuntos de símbolos de operação, indexada por perfis da forma $w \rightarrow s$, onde $w \in S^*$ (uma sequência finita de sorts, representando os sorts dos argumentos) e $s \in S$ (o sort do resultado). Ou seja, para cada perfil $(s_1 s_2 \ldots s_k, s_{res})$, $\Omega_{s_1 s_2 \ldots s_k, s_{res}}$ é o conjunto de todos os símbolos de operação com esse perfil específico.

Por exemplo, considere uma assinatura para o TAD `Boolean` básico:
*   $S = \{ \text{Boolean} \}$
*   $\Omega$ conteria:
    *   $\Omega_{\epsilon, \text{Boolean}} = \{ \text{true, false} \}$ (constantes, $\epsilon$ denota a sequência vazia de argumentos)
    *   $\Omega_{\text{Boolean}, \text{Boolean}} = \{ \text{not} \}$
    *   $\Omega_{\text{Boolean Boolean}, \text{Boolean}} = \{ \text{and, or} \}$

Uma forma comum de apresentar uma assinatura é listar os sorts e, em seguida, listar cada operação com seu perfil, como no exemplo:

**SPEC ADT** Boolean

**sorts:**
*   Boolean

**operations:**
*   true: --> Boolean
*   false: --> Boolean
*   not: Boolean --> Boolean
*   and: Boolean x Boolean --> Boolean
*   or: Boolean x Boolean --> Boolean

**END SPEC**

Este bloco de especificação, intitulado `Boolean`, define a assinatura para um Tipo Abstrato de Dados Booleano. A seção `sorts` introduz um único tipo de dado, chamado `Boolean`. A seção `operations` declara cinco operações. As operações `true` e `false` são constantes do tipo `Boolean`, pois não requerem argumentos e produzem um valor booleano. A operação `not` é uma função unária que aceita um argumento do tipo `Boolean` e retorna um valor do tipo `Boolean`. Finalmente, as operações `and` e `or` são funções binárias, cada uma recebendo dois argumentos do tipo `Boolean` e resultando em um valor do tipo `Boolean`. Esta assinatura estabelece apenas a sintaxe, os nomes e os tipos envolvidos, sem ainda definir o comportamento dessas operações.

A assinatura é o primeiro passo para a formalização de um TAD. Ela estabelece a "interface sintática" do tipo de dados. O próximo passo é definir os termos que podem ser construídos a partir dessa assinatura e, subsequentemente, os axiomas que governam o comportamento das operações sobre esses termos.

**Termos sobre uma Assinatura (T<sub>Σ</sub>(X)). Variáveis e Termos Fundamentais (`Ground Terms`)**

Uma vez definida uma Σ-assinatura $\Sigma = (S, \Omega)$, podemos construir **termos** sobre essa assinatura. Os termos são expressões sintáticas que representam os valores e os cômputos que podem ser realizados dentro do Tipo Abstrato de Dados. Eles são construídos recursivamente a partir dos símbolos de operação em $\Omega$ e, opcionalmente, de um conjunto de variáveis.

Seja $X$ uma família de conjuntos de variáveis $X = \{X_s\}_{s \in S}$, onde cada $X_s$ é um conjunto de variáveis do sort $s$. As variáveis em $X_s$ são distintas dos símbolos de operação em $\Omega$. O conjunto de todos os termos sobre $\Sigma$ com variáveis em $X$, denotado por $T_{\Sigma}(X)$, é também uma família de conjuntos de termos $T_{\Sigma}(X) = \{ (T_{\Sigma}(X))_s \}_{s \in S}$, onde $(T_{\Sigma}(X))_s$ é o conjunto de termos do sort $s$. Estes conjuntos são definidos indutivamente como os menores conjuntos que satisfazem as seguintes regras:

1.  **Variáveis:** Para cada sort $s \in S$, toda variável $x \in X_s$ é um termo do sort $s$. Ou seja, $X_s \subseteq (T_{\Sigma}(X))_s$.
2.  **Constantes:** Para cada sort $s \in S$, toda constante $c: \rightarrow s$ (um símbolo de operação em $\Omega_{\epsilon, s}$) é um termo do sort $s$. Ou seja, se $c \in \Omega_{\epsilon, s}$, então $c \in (T_{\Sigma}(X))_s$.
3.  **Aplicações de Operações:** Se $f: s_1 \times s_2 \times \ldots \times s_k \rightarrow s_{res}$ é um símbolo de operação em $\Omega$ (com $k > 0$), e $t_1, t_2, \ldots, t_k$ são termos tais que $t_i$ é um termo do sort $s_i$ (i.e., $t_i \in (T_{\Sigma}(X))_{s_i}$ para $i=1, \ldots, k$), então a expressão $f(t_1, t_2, \ldots, t_k)$ é um termo do sort $s_{res}$. Ou seja, $f(t_1, \ldots, t_k) \in (T_{\Sigma}(X))_{s_{res}}$.

**Termos Fundamentais (`Ground Terms`):**
Um caso particularmente importante é quando o conjunto de variáveis $X$ é vazio (i.e., $X_s = \emptyset$ para todos os sorts $s \in S$). Os termos construídos sem o uso de variáveis são chamados de **termos fundamentais** ou `ground terms`. O conjunto de todos os termos fundamentais de sort $s$ é denotado por $(T_{\Sigma})_s$, e a família de todos os conjuntos de termos fundamentais é $T_{\Sigma}$. Os termos fundamentais representam os valores concretos que podem ser "computados" ou "construídos" dentro do TAD usando apenas suas operações constantes e outras operações aplicadas a termos já construídos.
Em muitos contextos, assume-se que para cada sort $s$, o conjunto $(T_{\Sigma})_s$ é não vazio, o que significa que cada sort possui pelo menos um valor que pode ser denotado por um termo fundamental. Isso é frequentemente garantido pela presença de operações construtoras, incluindo constantes.

**Exemplos de Termos:**
Usando a assinatura `Boolean` definida anteriormente e assumindo variáveis $b_1, b_2$ do sort `Boolean`:
*   Termos fundamentais do sort `Boolean`:
    *   `true`
    *   `false`
    *   `not(true)`
    *   `and(true, false)`
    *   `or(not(false), true)`
*   Termos com variáveis do sort `Boolean`:
    *   $b_1$
    *   `not(b_1)`
    *   `and(b_1, true)`
    *   `or(b_1, b_2)`
    *   `and(not(b_1), or(b_2, false))`

Considerando uma assinatura para o TAD `Natural` (com sorts `Natural`, `Boolean` e operações `zero`, `succ`, `isZero`, `add`, e constantes `true`, `false` para `Boolean`):
*   Variáveis: $n, m$ do sort `Natural`; $b$ do sort `Boolean`.
*   Termos fundamentais do sort `Natural`:
    *   `zero`
    *   `succ(zero)`
    *   `succ(succ(zero))`
    *   `add(succ(zero), zero)`
*   Termos fundamentais do sort `Boolean`:
    *   `isZero(zero)`
    *   `isZero(add(succ(zero), zero))`
*   Termos com variáveis do sort `Natural`:
    *   `succ(n)`
    *   `add(n, m)`
    *   `add(n, succ(zero))`
*   Termos com variáveis do sort `Boolean`:
    *   `isZero(n)` (este termo é do sort `Boolean`, mas contém uma variável $n$ do sort `Natural`)

A estrutura dos termos é fundamental. Os axiomas de uma especificação algébrica serão equações entre termos (geralmente termos com variáveis). A semântica de um TAD será definida interpretando esses termos em uma álgebra concreta.

**Substituições e Instanciação**

No contexto da lógica equacional e da especificação algébrica, as **substituições** desempenham um papel crucial, pois permitem instanciar termos que contêm variáveis, atribuindo termos específicos (geralmente termos fundamentais ou outros termos) a essas variáveis. Este processo de **instanciação** é fundamental para aplicar axiomas (que são geralmente expressos com variáveis) a situações concretas e para definir a noção de consequência lógica.

Uma **substituição** $\sigma$ é um mapeamento de variáveis para termos, tal que para cada variável $x$ de sort $s$, o termo $\sigma(x)$ atribuído a $x$ também é do sort $s$. Formalmente, se $X = \{X_s\}_{s \in S}$ é a família de conjuntos de variáveis e $T_{\Sigma}(Y) = \{(T_{\Sigma}(Y))_s\}_{s \in S}$ é a família de conjuntos de termos (possivelmente construídos a partir de outro conjunto de variáveis $Y$, ou termos fundamentais se $Y = \emptyset$), uma substituição $\sigma: X \rightarrow T_{\Sigma}(Y)$ é uma família de funções $\{\sigma_s: X_s \rightarrow (T_{\Sigma}(Y))_s\}_{s \in S}$. Na prática, uma substituição é frequentemente definida apenas para um subconjunto finito de variáveis que aparecem nos termos de interesse, assumindo-se que ela é a identidade para as demais variáveis.

A **aplicação de uma substituição** $\sigma$ a um termo $t \in T_{\Sigma}(X)$, denotada por $t\sigma$ (ou $\sigma(t)$ ou $t[\sigma]$), é o termo obtido substituindo-se simultaneamente cada ocorrência de cada variável $x$ em $t$ pelo termo $\sigma(x)$. A definição é recursiva:
1.  Se $t$ é uma variável $x \in X_s$, então $t\sigma = \sigma_s(x)$.
2.  Se $t$ é uma constante $c: \rightarrow s$, então $t\sigma = c$.
3.  Se $t = f(t_1, t_2, \ldots, t_k)$, onde $f: s_1 \times \ldots \times s_k \rightarrow s_{res}$ é um símbolo de operação e $t_i$ são termos, então $t\sigma = f(t_1\sigma, t_2\sigma, \ldots, t_k\sigma)$.

O termo $t\sigma$ é chamado de uma **instância** do termo $t$. Se todos os termos $\sigma(x)$ na imagem da substituição são termos fundamentais (i.e., não contêm variáveis), então $t\sigma$ será também um termo fundamental, chamado de **instância fundamental** de $t$.

**Exemplo de Substituição e Instanciação:**
Considere uma assinatura para `Natural` com operações `zero`, `succ`, `add` e o termo $t = \text{add}(n, \text{succ}(m))$, onde $n, m$ são variáveis do sort `Natural`.
Seja uma substituição $\sigma$ definida por:
*   $\sigma(n) = \text{zero}$
*   $\sigma(m) = \text{succ(zero)}$

Aplicando $\sigma$ a $t$:
$t\sigma = (\text{add}(n, \text{succ}(m)))\sigma$
      $= \text{add}(n\sigma, (\text{succ}(m))\sigma)$
      $= \text{add}(\sigma(n), \text{succ}(m\sigma))$
      $= \text{add}(\sigma(n), \text{succ}(\sigma(m)))$
      $= \text{add}(\text{zero}, \text{succ}(\text{succ(zero)}))$

O termo resultante $\text{add}(\text{zero}, \text{succ}(\text{succ(zero)}))$ é uma instância fundamental do termo original $t$.

As substituições são essenciais para a regra de inferência de instanciação na lógica equacional: se uma equação $t_1 = t_2$ é um axioma ou foi derivada, então para qualquer substituição $\sigma$, a equação $t_1\sigma = t_2\sigma$ também é válida (é uma consequência lógica). Isso permite que leis gerais (axiomas com variáveis) sejam aplicadas a casos específicos.

**Exercício:**

Dada uma assinatura para o TAD `Natural` com:
**sorts:** `Natural`, `Boolean`
**operations:**
`zero: --> Natural`
`succ: Natural --> Natural`
`add: Natural x Natural --> Natural`
`true: --> Boolean`
`false: --> Boolean`
`isZero: Natural --> Boolean`

Considere o termo $t = \text{isZero}(\text{add}(x, \text{succ}(y)))$, onde $x, y$ são variáveis do sort `Natural`.
Defina uma substituição $\sigma_1$ tal que $\sigma_1(x) = \text{succ(zero)}$ e $\sigma_1(y) = \text{zero}$.
Aplique a substituição $\sigma_1$ ao termo $t$ para obter a instância $t\sigma_1$. Mostre os passos da aplicação.

**Resolução:**

O termo original é $t = \text{isZero}(\text{add}(x, \text{succ}(y)))$.
A substituição $\sigma_1$ é definida como:
*   $\sigma_1(x) = \text{succ(zero)}$
*   $\sigma_1(y) = \text{zero}$

Aplicando a substituição $\sigma_1$ ao termo $t$:
$t\sigma_1 = (\text{isZero}(\text{add}(x, \text{succ}(y))))\sigma_1$

Seguindo a definição recursiva da aplicação de uma substituição:
1.  A substituição se propaga para o argumento da operação mais externa (`isZero`):
    $t\sigma_1 = \text{isZero}((\text{add}(x, \text{succ}(y)))\sigma_1)$

2.  A substituição se propaga para os argumentos da operação `add`:
    $t\sigma_1 = \text{isZero}(\text{add}(x\sigma_1, (\text{succ}(y))\sigma_1))$

3.  A substituição se propaga para o argumento da operação `succ`:
    $t\sigma_1 = \text{isZero}(\text{add}(x\sigma_1, \text{succ}(y\sigma_1)))$

4.  Agora, substituímos as variáveis $x$ e $y$ por seus respectivos termos mapeados por $\sigma_1$:
    *   $x\sigma_1 = \sigma_1(x) = \text{succ(zero)}$
    *   $y\sigma_1 = \sigma_1(y) = \text{zero}$

    Substituindo estes na expressão:
    $t\sigma_1 = \text{isZero}(\text{add}(\text{succ(zero)}, \text{succ}(\text{zero})))$

Portanto, a instância $t\sigma_1$ do termo $t$ é:
$\text{isZero}(\text{add}(\text{succ(zero)}, \text{succ}(\text{zero})))$

# 2.2 Axiomas Equacionais e Especificações Algébricas

Após definir a estrutura sintática de um Tipo Abstrato de Dados (TAD) através de uma Σ-assinatura – que estabelece os sorts (tipos de dados) e os símbolos de operação com seus respectivos perfis – o próximo passo crucial na especificação algébrica é dotar essa estrutura de **semântica comportamental**. Isso é alcançado pela introdução de **axiomas equacionais**. Os axiomas são equações entre termos construídos sobre a assinatura, e eles têm o papel fundamental de restringir o comportamento das operações, definindo quando diferentes sequências de operações (representadas por termos) devem produzir o mesmo resultado abstrato. Em essência, os axiomas capturam as propriedades fundamentais e as leis que governam o TAD. Uma **especificação algébrica** é, então, formalmente constituída por um par: a assinatura Σ, que fornece o vocabulário, e um conjunto E de axiomas equacionais, que descreve as relações semânticas. A partir desse conjunto de axiomas, outras equações podem ser derivadas utilizando as regras de inferência da lógica equacional, formando a teoria equacional do TAD. Esta seção detalhará a natureza dos axiomas equacionais, a formalização de uma especificação algébrica e os conceitos de consequência lógica e derivação dentro deste formalismo.

**Definição de Axiomas como Equações entre Termos**

Na especificação algébrica, os **axiomas** são os pilares que definem o comportamento e as propriedades das operações de um Tipo Abstrato de Dados (TAD). Eles são formulados como **equações entre termos**, onde os termos são construídos a partir da assinatura Σ do TAD e de um conjunto $X$ de variáveis (conforme definido na seção 2.1.2).

Uma **equação** é uma expressão da forma $t_1 = t_2$, onde $t_1$ e $t_2$ são termos do mesmo sort $s \in S$. Esta equação postula que, sob qualquer interpretação válida das operações e qualquer atribuição de valores às variáveis (que respeitem os sorts), os termos $t_1$ e $t_2$ devem denotar o mesmo valor abstrato no domínio do sort $s$.

Os axiomas são geralmente **equações com variáveis universalmente quantificadas** (embora a quantificação universal seja frequentemente implícita na notação). Isso significa que um axioma $t_1 = t_2$, onde $t_1$ e $t_2$ podem conter variáveis de $X$, é entendido como uma afirmação de que a igualdade $t_1\sigma = t_2\sigma$ se mantém para *todas* as substituições $\sigma$ que mapeiam as variáveis em $X$ para termos fundamentais (ou para termos em qualquer álgebra que seja um modelo da especificação). Essa universalidade é o que confere aos axiomas o status de leis gerais do TAD.

**Formato dos Axiomas:**
Um axioma é tipicamente escrito como:
$(\forall X) \quad t_1 = t_2$
ou, mais comumente, a cláusula de quantificação universal $(\forall X)$ (lê-se "para todo $X$") é omitida ou colocada no final da especificação, como na cláusula `for all` que usamos:
$t_1 = t_2$
(com uma declaração `for all x: SortX, y: SortY, ...` ao final da lista de axiomas, especificando as variáveis e seus sorts que aparecem nos lados esquerdo e direito das equações).

**Exemplos de Axiomas Equacionais:**
Usando uma assinatura para o TAD `Natural` (com sorts `Natural`, `Boolean`; operações `zero`, `succ`, `isZero`, `add`; e constantes `true`, `false` para `Boolean`):

1.  `isZero(zero) = true`
    *   Este axioma não contém variáveis. É uma igualdade entre dois termos fundamentais do sort `Boolean`. Ele define que a operação `isZero` aplicada à constante `zero` (representando o número 0) resulta na constante booleana `true`.

2.  `isZero(succ(n)) = false`
    *   Este axioma contém uma variável $n$ do sort `Natural`. Ele afirma que para *qualquer* número natural $n$, o predicado `isZero` aplicado ao sucessor de $n$ (i.e., $n+1$) resulta na constante booleana `false`.

3.  `add(n, zero) = n`
    *   Este axioma, com a variável $n$ do sort `Natural`, define a propriedade do `zero` como elemento neutro à direita para a operação `add`. Para qualquer natural $n$, adicionar $n$ a `zero` resulta no próprio $n$.

4.  `add(n, succ(m)) = succ(add(n, m))`
    *   Este é um axioma recursivo com variáveis $n, m$ do sort `Natural`. Ele define a adição de $n$ com o sucessor de $m$ em termos da adição de $n$ com $m$.

**Equações Condicionais:**
Algumas especificações podem usar **equações condicionais**, da forma:
$u_1 = v_1 \land \ldots \land u_k = v_k \implies t_1 = t_2$
A equação $t_1 = t_2$ só é exigida se todas as condições $u_i = v_i$ forem satisfeitas.

Os axiomas são escolhidos pelo especificador para capturar com precisão a semântica desejada do TAD.

**Uma Especificação Algébrica SP = (Σ, E) como um par Assinatura-Axiomas**

Uma **especificação algébrica**, denotada como $SP$, é formalmente definida como um par $( \Sigma, E)$, onde:

1.  $\Sigma = (S, \Omega)$ é uma **assinatura**.
2.  $E$ é um conjunto de **axiomas equacionais**.

A assinatura $\Sigma$ estabelece a sintaxe do TAD. Os axiomas $E$ estabelecem a semântica do TAD.

**Exemplo: Especificação do TAD `Boolean`**
Revisitando o TAD `Boolean` com a estrutura formal $SP_{Boolean} = (\Sigma_{Boolean}, E_{Boolean})$:

**SPEC ADT** Boolean

**sorts:**
*   Boolean

**operations:**
*   true: --> Boolean
*   false: --> Boolean
*   not: Boolean --> Boolean
*   and: Boolean x Boolean --> Boolean
*   or: Boolean x Boolean --> Boolean
*   ifThenElse: Boolean x Boolean x Boolean --> Boolean

**axioms:**
*   (B1): not(true) = false
*   (B2): not(false) = true
*   (B3): and(true, b) = b
*   (B4): and(false, b) = false
*   (B5): or(true, b) = true
*   (B6): or(false, b) = b
*   (B7): ifThenElse(true, b1, b2) = b1
*   (B8): ifThenElse(false, b1, b2) = b2

for all b, b1, b2: Boolean

**END SPEC**

Esta especificação `Boolean` define o Tipo Abstrato de Dados para valores e operações booleanas. O sort `Boolean` é o único tipo de dado introduzido. As operações incluem as constantes `true` e `false`; a negação unária `not`; as operações binárias de conjunção `and` e disjunção `or`; e uma operação ternária condicional `ifThenElse`. Os axiomas (B1) a (B8) definem o comportamento dessas operações para todas as possíveis variáveis booleanas `b`, `b1`, e `b2`. Por exemplo, (B1) e (B2) definem a negação. (B3) e (B4) definem a conjunção em relação a `true` e `false`. (B5) e (B6) fazem o mesmo para a disjunção. (B7) e (B8) especificam que `ifThenElse` retorna seu segundo argumento se a condição for `true`, e seu terceiro argumento se a condição for `false`.

O par $(\Sigma, E)$ é a entidade matemática fundamental que estudamos na especificação algébrica.

**Consequência Lógica e Derivação (Regras de Inferência da Lógica Equacional)**

Dada uma especificação algébrica $SP = (\Sigma, E)$, uma equação $t_1 = t_2$ é uma **consequência lógica** dos axiomas $E$, denotado por $E \models t_1 = t_2$, se $t_1 = t_2$ é satisfeita em todas as Σ-álgebras que satisfazem todos os axiomas em $E$.

Paralelamente, uma equação $t_1 = t_2$ é **derivável** (ou **demonstrável**) a partir de $E$, denotado por $E \vdash t_1 = t_2$, se ela pode ser obtida a partir dos axiomas em $E$ através de um número finito de aplicações de um conjunto de **regras de inferência** da lógica equacional.

As regras fundamentais são: Reflexividade, Simetria, Transitividade, Substituição e Congruência.

1.  **Reflexividade:** $\overline{t = t}$
2.  **Simetria:** $\frac{t_1 = t_2}{t_2 = t_1}$
3.  **Transitividade:** $\frac{t_1 = t_2, \quad t_2 = t_3}{t_1 = t_3}$
4.  **Substituição:** $\frac{t_1 = t_2}{t_1\sigma = t_2\sigma}$
5.  **Congruência:** $\frac{t_1 = u_1, \quad \ldots, \quad t_k = u_k}{f(t_1, \ldots, t_k) = f(u_1, \ldots, u_k)}$

Uma **derivação** de uma equação $e$ a partir de $E$ é uma sequência finita de equações $e_1, \ldots, e_n=e$, onde cada $e_i$ é um axioma em $E$ (instanciado) ou obtida de equações anteriores por uma regra de inferência.

**Teorema da Completude de Birkhoff para a Lógica Equacional:**
Este teorema estabelece que $E \models e$ se e somente se $E \vdash e$. A noção semântica de consequência lógica coincide com a noção sintática de derivabilidade.

**Exercício:**

Usando a especificação `Boolean` (com axiomas B1-B8) e as regras de inferência da lógica equacional, prove que a equação `not(not(true)) = true` é derivável a partir dos axiomas de `Boolean`. Liste os passos da derivação e os axiomas/regras utilizados.

**Resolução:**

Queremos provar que `not(not(true)) = true` é derivável a partir dos axiomas da especificação `Boolean`.

1.  **Axioma (B1):** `not(true) = false`
    (Este é um axioma da especificação `Boolean`).

2.  **Aplicação da Regra de Congruência:**
    Aplicamos a operação `not` a ambos os lados da equação obtida no passo 1.
    Seja $t_1 = \text{not(true)}$ e $u_1 = \text{false}$. Temos $t_1 = u_1$.
    Pela regra de Congruência, com $f = \text{not}$:
    `not(not(true)) = not(false)`
    (Chamaremos esta Equação 2.A)

3.  **Axioma (B2):** `not(false) = true`
    (Este é um axioma da especificação `Boolean`).
    (Chamaremos esta Equação 2.B)

4.  **Aplicação da Regra de Transitividade:**
    Temos:
    *   Equação 2.A: `not(not(true)) = not(false)`
    *   Equação 2.B: `not(false) = true`

    Seja $A = \text{not(not(true))}$, $B = \text{not(false)}$, e $C = \text{true}$.
    Temos $A = B$ (da Equação 2.A) e $B = C$ (da Equação 2.B).
    Pela regra da Transitividade ($A=B, B=C \implies A=C$):
    `not(not(true)) = true`

Esta é a equação que queríamos derivar. A derivação está completa, utilizando os axiomas (B1), (B2) e as regras de inferência de Congruência e Transitividade.

# 2.3 Semântica das Especificações Algébricas: Σ-Álgebras

Enquanto a assinatura $\Sigma$ e o conjunto de axiomas $E$ de uma especificação algébrica $SP = (\Sigma, E)$ definem a sintaxe e as leis formais de um Tipo Abstrato de Dados, a questão de seu **significado** ou **interpretação** é tratada pela semântica. No contexto da especificação algébrica, a semântica é dada por estruturas matemáticas chamadas **Σ-álgebras**. Uma Σ-álgebra fornece uma concretização para os sorts e operações da assinatura $\Sigma$, de forma que os sorts são mapeados para conjuntos (os conjuntos portadores) e os símbolos de operação são mapeados para funções concretas sobre esses conjuntos. Uma Σ-álgebra é dita um **modelo** de uma especificação $(\Sigma, E)$ se, além de respeitar a assinatura $\Sigma$, ela também **satisfaz** todos os axiomas em $E$. A coleção de todas as Σ-álgebras que satisfazem $E$ forma a classe de modelos da especificação, denotada $Alg(SP)$. Dentro desta classe, modelos com propriedades universais, como os **modelos iniciais e finais**, desempenham um papel crucial, pois fornecem interpretações canônicas e não ambíguas para o TAD. Esta seção explorará a definição formal de uma Σ-álgebra, o conceito de satisfação de equações e a importância da classe de modelos $Alg(SP)$, incluindo a menção a modelos iniciais e finais.

**Definição de uma Σ-Álgebra: Domínios Portadores e Interpretação de Operações**

Dada uma Σ-assinatura $\Sigma = (S, \Omega)$, uma **Σ-álgebra** (ou simplesmente uma álgebra sobre $\Sigma$, quando $\Sigma$ está claro no contexto) $A$ é uma estrutura que fornece uma interpretação concreta para os sorts e os símbolos de operação definidos em $\Sigma$. Formalmente, uma Σ-álgebra $A$ consiste em:

1.  **Uma família de conjuntos portadores (carrier sets):** Para cada nome de sort $s \in S$, a álgebra $A$ associa um conjunto não vazio $A_s$, chamado de conjunto portador (ou domínio) do sort $s$. Este conjunto $A_s$ contém os "valores concretos" do tipo $s$ naquela particular álgebra. A família de todos esses conjuntos portadores é denotada como $\{A_s\}_{s \in S}$.

2.  **Uma família de funções concretas (interpretações das operações):** Para cada símbolo de operação $f \in \Omega$ com perfil $f: s_1 \times s_2 \times \ldots \times s_k \rightarrow s_{res}$ (onde $s_i, s_{res} \in S$), a álgebra $A$ associa uma função concreta $f_A$ (a interpretação de $f$ em $A$) com o mesmo perfil, mas operando sobre os conjuntos portadores correspondentes:
    $f_A: A_{s_1} \times A_{s_2} \times \ldots \times A_{s_k} \rightarrow A_{s_{res}}$.
    *   Se $f$ é uma constante $c: \rightarrow s_{res}$ (i.e., $k=0$), então sua interpretação $c_A$ é um elemento específico do conjunto portador $A_{s_{res}}$, ou seja, $c_A \in A_{s_{res}}$.

**Exemplo de Σ-Álgebra para `Boolean`:**
Considere a assinatura `Boolean` com sort `Boolean` e operações `true`, `false`, `not`, `and`, `or`.
Podemos definir uma Σ-álgebra $A_{stdBool}$ (a álgebra booleana padrão) da seguinte forma:

1.  **Conjunto Portador:** $A_{\text{Boolean}} = \{ \mathbb{T}, \mathbb{F} \}$.
2.  **Interpretação das Operações:**
    *   $\text{true}_{A_{stdBool}} = \mathbb{T}$
    *   $\text{false}_{A_{stdBool}} = \mathbb{F}$
    *   $\text{not}_{A_{stdBool}}(\mathbb{T}) = \mathbb{F}$, $\text{not}_{A_{stdBool}}(\mathbb{F}) = \mathbb{T}$
    *   $\text{and}_{A_{stdBool}}$: a função E lógica usual sobre $\{ \mathbb{T}, \mathbb{F} \}$.
    *   $\text{or}_{A_{stdBool}}$: a função OU lógica usual sobre $\{ \mathbb{T}, \mathbb{F} \}$.

**Avaliação de Termos em uma Σ-Álgebra:**
Dada uma Σ-álgebra $A$ e uma **valoração** $\nu: X \rightarrow A$ que mapeia variáveis $x \in X_s$ a elementos $\nu(x) \in A_s$, a **avaliação** de um termo $t \in (T_{\Sigma}(X))_s$ em $A$ sob $\nu$, denotada $\llbracket t \rrbracket_{A,\nu}$, é um elemento de $A_s$ definido recursivamente:
1.  $\llbracket x \rrbracket_{A,\nu} = \nu(x)$ (para variável $x$).
2.  $\llbracket c \rrbracket_{A,\nu} = c_A$ (para constante $c$).
3.  $\llbracket f(t_1, \ldots, t_k) \rrbracket_{A,\nu} = f_A(\llbracket t_1 \rrbracket_{A,\nu}, \ldots, \llbracket t_k \rrbracket_{A,\nu})$.
Se $t$ é fundamental, sua avaliação $\llbracket t \rrbracket_A$ não depende de $\nu$.

**Satisfação de Equações em uma Álgebra. Modelos de uma Especificação**

Uma equação $t_1 = t_2$ (com variáveis em $X$) é **satisfeita** em uma Σ-álgebra $A$ (denotado $A \models t_1 = t_2$) se para **todas** as valorações $\nu: X \rightarrow A$, $\llbracket t_1 \rrbracket_{A,\nu} = \llbracket t_2 \rrbracket_{A,\nu}$.

Uma Σ-álgebra $A$ é um **modelo** de uma especificação algébrica $SP = (\Sigma, E)$ se $A$ satisfaz **todos** os axiomas $e \in E$.

**Exemplo de Satisfação e Modelo:**
Para a especificação `Boolean` (da Seção 2.2.2) e a álgebra $A_{stdBool}$:
O axioma (B1) é `not(true) = false`.
*   $\llbracket \text{not(true)} \rrbracket_{A_{stdBool}} = \text{not}_{A_{stdBool}}(\text{true}_{A_{stdBool}}) = \text{not}_{A_{stdBool}}(\mathbb{T}) = \mathbb{F}$.
*   $\llbracket \text{false} \rrbracket_{A_{stdBool}} = \text{false}_{A_{stdBool}} = \mathbb{F}$.
Como $\mathbb{F} = \mathbb{F}$, (B1) é satisfeito em $A_{stdBool}$.
Similarmente, pode-se verificar que $A_{stdBool}$ satisfaz todos os axiomas (B1)-(B8), logo $A_{stdBool}$ é um modelo de `Boolean`.

**A Classe de Todas as Σ-Álgebras que Satisfazem E (Alg(SP)). Modelos Iniciais e Finais.**

A coleção de todas as Σ-álgebras que são modelos de uma especificação $SP = (\Sigma, E)$ é a **classe de modelos de SP**, denotada $Alg(SP)$.
$Alg(SP) = \{ A \mid A \text{ é uma } \Sigma\text{-álgebra e } A \models E \}$.

Dentro de $Alg(SP)$:
1.  Um **Modelo Inicial** $I$ é tal que para toda $A \in Alg(SP)$, existe um único homomorfismo $h: I \rightarrow A$.
    *   Propriedades: "No junk" (todos os elementos são denotáveis por termos fundamentais) e "No confusion" ($\llbracket t_1 \rrbracket_I = \llbracket t_2 \rrbracket_I \iff E \vdash t_1 = t_2$).
    *   Para `Boolean`, $A_{stdBool}$ é o modelo inicial (a menos de isomorfismo).
    *   Para `Natural` com axiomas de Peano, os números naturais padrão $\{0, 1, 2, \ldots\}$ formam o modelo inicial.

2.  Um **Modelo Final** $F$ é tal que para toda $A \in Alg(SP)$, existe um único homomorfismo $h: A \rightarrow F$.
    *   Modelos finais tendem a identificar o máximo de termos possível e são úteis para semântica observacional.

Se um modelo inicial (ou final) existe, ele é único a menos de isomorfismo, fornecendo uma semântica canônica para o TAD.

**Exercício:**

Considere a especificação `Natural` com a assinatura e axiomas de Peano (como em 2.2.2, incluindo `zero` e `succ`). Descreva informalmente como seria o conjunto portador $A_{Natural}$ e as interpretações das operações `zero` e `succ` no modelo inicial $I_{Natural}$ desta especificação. O que significaria a propriedade "no junk" e "no confusion" para este $I_{Natural}$?

**Resolução:**

Para a especificação `Natural` com axiomas de Peano:

1.  **Conjunto Portador $A_{Natural}$ no Modelo Inicial $I_{Natural}$:**
    O conjunto portador $I_{Natural, \text{Natural}}$ seria isomorfo ao conjunto dos números naturais padrão, como os conhecemos: $\{0, 1, 2, 3, \ldots\}$. Cada elemento neste conjunto corresponde a um valor único do tipo `Natural`.

2.  **Interpretação das Operações `zero` e `succ` em $I_{Natural}$:**
    *   $\text{zero}_{I_{Natural}}$: A interpretação da constante `zero` seria o elemento $0$ no conjunto $\{0, 1, 2, \ldots\}$.
    *   $\text{succ}_{I_{Natural}}$: A interpretação da operação `succ` seria a função sucessor padrão $s(x) = x+1$ sobre os números naturais. Assim, $\text{succ}_{I_{Natural}}(0) = 1$, $\text{succ}_{I_{Natural}}(1) = 2$, e assim por diante.

**Significado de "no junk" e "no confusion" para $I_{Natural}$:**

*   **"No junk":**
    Esta propriedade significa que todo elemento no conjunto portador $I_{Natural, \text{Natural}}$ (ou seja, todo número natural $0, 1, 2, \ldots$) pode ser "alcançado" ou "construído" por um termo fundamental da especificação. Os termos fundamentais para `Natural` são:
    `zero` (interpretação: 0)
    `succ(zero)` (interpretação: 1)
    `succ(succ(zero))` (interpretação: 2)
    ... e assim por diante.
    "No junk" garante que não existem elementos "estranhos" em $I_{Natural, \text{Natural}}$ que não correspondam a uma dessas construções. Todos os elementos são "gerados" pelos construtores.

*   **"No confusion":**
    Esta propriedade significa que dois termos fundamentais distintos são interpretados como o mesmo elemento em $I_{Natural, \text{Natural}}$ se, e somente se, sua igualdade puder ser derivada dos axiomas da especificação `Natural`. Para os construtores `zero` e `succ` de Peano, os axiomas geralmente implicam que:
    *   `zero` é distinto de `succ(n)` para qualquer `n`.
    *   `succ(n)` é igual a `succ(m)` se e somente se `n` é igual a `m`.
    Isso significa que termos fundamentais como `zero`, `succ(zero)`, `succ(succ(zero))`, etc., que são sintaticamente distintos de uma forma "canônica" (usando apenas construtores), são todos interpretados como elementos distintos em $I_{Natural, \text{Natural}}$ (0, 1, 2, ... respectivamente). Não há "confusão" onde, por exemplo, `succ(zero)` (que representa 1) seria igual a `succ(succ(zero))` (que representa 2), a menos que os axiomas forçassem essa igualdade (o que os axiomas de Peano não fazem). O modelo inicial faz apenas as identificações que são estritamente exigidas pelos axiomas.

# 2.4 Construtores, Observadores e Operações Derivadas

Ao projetar e analisar uma especificação algébrica de um Tipo Abstrato de Dados (TAD), é metodologicamente útil classificar as operações da assinatura em diferentes categorias com base em seu papel na construção e inspeção dos valores do TAD. As três categorias principais frequentemente consideradas são: **construtores**, **observadores** e **operações derivadas** (ou auxiliares). Embora essa classificação não seja sempre estritamente formal ou disjunta em todas as teorias de especificação, ela fornece uma heurística poderosa para entender a estrutura do TAD, para guiar a escrita dos axiomas e para analisar propriedades como a completude da especificação.

**Construtores (Constructors):**
Os construtores são as operações responsáveis por **gerar todos os valores** do sort principal do TAD.
*   **Exemplos:**
    *   Para `Natural`: `zero` e `succ`.
    *   Para `Boolean`: `true` e `false`.
    *   Para `NaturalList`: `emptyList` e `cons`.

**Observadores (Observers ou Selectors):**
Os observadores são operações que **retornam informações sobre os valores** de um TAD, geralmente retornando valores de outros sorts ou realizando uma "decomposição".
*   **Exemplos:**
    *   Para `Natural`: `isZero`.
    *   Para `NaturalList`: `isEmpty`, `head`, `tail`.

**Operações Derivadas (Derived Operations ou Auxiliary Operations):**
As operações derivadas são aquelas cujo comportamento pode ser completamente definido em termos dos construtores e observadores.
*   **Exemplos:**
    *   Para `Natural`: `add`, `mult`.
    *   Para `Boolean`: `and`, `or`, `ifThenElse` (se `not` e uma das conectivas são tomadas como primitivas junto com as constantes).
    *   Para `NaturalList`: `length`, `append`.

**Importância da Classificação:**
*   Guia a escrita dos axiomas (observadores definidos por casos sobre construtores).
*   Auxilia na análise de completude da especificação.
*   Orienta a implementação do TAD.

A distinção nem sempre é absoluta, mas a metodologia de definir observadores e operações derivadas em termos de construtores é central na especificação algébrica.

**Exercício:**

Para o TAD `Natural` (números naturais de Peano) com as operações:
*   `zero: --> Natural`
*   `succ: Natural --> Natural`
*   `isZero: Natural --> Boolean`
*   `add: Natural x Natural --> Natural`
*   `double: Natural --> Natural` (que dobra um número natural, e.g., `double(succ(succ(zero)))` deve ser igual a `succ(succ(succ(succ(zero))))`).

Classifique cada uma dessas operações (`zero`, `succ`, `isZero`, `add`, `double`) como `Construtor`, `Observador` ou `Operação Derivada`. Justifique brevemente sua classificação para `double`. Assuma que `Boolean` e suas operações (`true`, `false`) já estão definidos.

**Resolução:**

Classificação das operações do TAD `Natural`:

1.  **`zero: --> Natural`**
    *   **Classificação:** `Construtor` (de base).
    *   **Justificativa:** É uma constante que cria o valor fundamental `zero` do sort `Natural`.

2.  **`succ: Natural --> Natural`**
    *   **Classificação:** `Construtor` (recursivo).
    *   **Justificativa:** Toma um `Natural` e constrói um novo `Natural` (seu sucessor). Junto com `zero`, gera todos os valores do tipo `Natural`.

3.  **`isZero: Natural --> Boolean`**
    *   **Classificação:** `Observador`.
    *   **Justificativa:** Retorna uma informação (um valor `Boolean`) sobre um `Natural` (se ele é `zero` ou não), sem criar um novo `Natural`. Seu comportamento é definido em termos dos construtores.

4.  **`add: Natural x Natural --> Natural`**
    *   **Classificação:** `Operação Derivada`.
    *   **Justificativa:** Embora retorne um `Natural`, seu comportamento é completamente definido recursivamente em termos dos construtores `zero` e `succ` e de si mesma (e.g., `add(n, zero) = n`, `add(n, succ(m)) = succ(add(n, m))`). Não é fundamental para gerar os valores básicos de `Natural`.

5.  **`double: Natural --> Natural`**
    *   **Classificação:** `Operação Derivada`.
    *   **Justificativa:** A operação `double` pode ser definida em termos da operação `add` (que já é derivada) ou diretamente em termos dos construtores `zero` e `succ`. Por exemplo, axiomaticamente:
        *   `double(zero) = zero`
        *   `double(succ(n)) = succ(succ(double(n)))`
        Ou, usando `add`:
        *   `double(n) = add(n, n)`
        Como seu comportamento pode ser especificado completamente usando outras operações já definidas (sejam construtores ou outras derivadas), ela é classificada como uma `Operação Derivada`. Ela adiciona uma funcionalidade de conveniência, mas não é necessária para a construção fundamental ou observação primária dos valores do tipo `Natural`.

# 2.5 Exemplos Fundamentais de Especificações Algébricas

Para consolidar a compreensão dos conceitos de assinatura, axiomas, sorts e operações, bem como a distinção entre construtores, observadores e operações derivadas, esta seção apresentará especificações algébricas detalhadas para três Tipos Abstratos de Dados (TADs) fundamentais: `Boolean`, `Natural` (números naturais segundo a axiomatização de Peano) e `Integer` (números inteiros). Estes exemplos não apenas ilustram a aplicação prática da teoria exposta, mas também servem como blocos de construção essenciais que serão referenciados e utilizados em especificações mais complexas ao longo deste livro. Para cada TAD, forneceremos sua especificação formal completa, seguida de uma explanação detalhada de seus componentes, enfatizando o papel de cada operação e a intenção por trás de cada axioma. A análise da corretude e completude dessas especificações será abordada em capítulos posteriores, mas o foco aqui é na sua construção e na expressividade da abordagem algébrica.

**Especificação de `Boolean`**

O Tipo Abstrato de Dados `Boolean` é, possivelmente, o mais fundamental em ciência da computação, representando os valores lógicos de verdade e falsidade e as operações básicas sobre eles.

**SPEC ADT** Boolean

**sorts:**
*   Boolean

**operations:**
*   true: --> Boolean
*   false: --> Boolean
*   not: Boolean --> Boolean
*   and: Boolean x Boolean --> Boolean
*   or: Boolean x Boolean --> Boolean
*   ifThenElse: Boolean x Boolean x Boolean --> Boolean

**axioms:**
*   (B1): not(true) = false
*   (B2): not(false) = true
*   (B3): and(true, b) = b
*   (B4): and(false, b) = false
*   (B5): or(true, b) = true
*   (B6): or(false, b) = b
*   (B7): ifThenElse(true, b1, b2) = b1
*   (B8): ifThenElse(false, b1, b2) = b2

for all b, b1, b2: Boolean

**END SPEC**

A especificação `Boolean` define formalmente o Tipo Abstrato de Dados para valores e operações booleanas. O único **sort** introduzido é `Boolean`. As **operações** `true` e `false` são constantes construtoras. As operações `not`, `and`, `or`, e `ifThenElse` são operações que manipulam valores booleanos. Os **axioms** (B1)-(B8) definem o comportamento dessas operações: (B1)-(B2) para `not`; (B3)-(B4) para `and` (com `true` como identidade à esquerda e `false` como absorvente à esquerda); (B5)-(B6) para `or` (com `true` como absorvente à esquerda e `false` como identidade à esquerda); e (B7)-(B8) para `ifThenElse` (selecionando o segundo ou terceiro argumento com base no primeiro). A cláusula `for all` indica que as variáveis `b, b1, b2` são universalmente quantificadas sobre o sort `Boolean`.

**Especificação de `Natural` (Números Naturais segundo Peano)**

O TAD `Natural` formaliza os números naturais (0, 1, 2, ...) com base nos axiomas de Peano.

**SPEC ADT** Natural

**imports:**
*   Boolean

**sorts:**
*   Natural

**operations:**
*   zero: --> Natural
*   succ: Natural --> Natural
*   isZero: Natural --> Boolean
*   add: Natural x Natural --> Natural
*   mult: Natural x Natural --> Natural
*   eq: Natural x Natural --> Boolean

**axioms:**
*   (N1): isZero(zero) = true
*   (N2): isZero(succ(n)) = false
*   (N3): add(n, zero) = n
*   (N4): add(n, succ(m)) = succ(add(n, m))
*   (N5): mult(n, zero) = zero
*   (N6): mult(n, succ(m)) = add(mult(n, m), n)
*   (N7): eq(zero, zero) = true
*   (N8): eq(zero, succ(m)) = false
*   (N9): eq(succ(n), zero) = false
*   (N10): eq(succ(n), succ(m)) = eq(n, m)

for all n, m: Natural

**END SPEC**

A especificação `Natural` define o Tipo Abstrato de Dados para números naturais, importando `Boolean` para os resultados de predicados. O **sort** principal é `Natural`. As **operações** `zero` e `succ` são os construtores. As operações `isZero` e `eq` são observadores que retornam um `Boolean`. As operações `add` e `mult` são operações derivadas para adição e multiplicação. Os **axioms** (N1)-(N2) definem `isZero`. (N3)-(N4) definem `add` recursivamente. (N5)-(N6) definem `mult` recursivamente em termos de `add`. (N7)-(N10) definem o predicado de igualdade `eq` recursivamente, cobrindo casos base e o passo indutivo para sucessores. A cláusula `for all` quantifica as variáveis `n, m` sobre `Natural`.

**Especificação de `Integer` (Números Inteiros)**

A especificação de `Integer` usa `zeroInt`, `succInt` (sucessor) e `predInt` (predecessor) como construtores.

**SPEC ADT** Integer

**imports:**
*   Boolean

**sorts:**
*   Integer

**operations:**
*   zeroInt: --> Integer
*   succInt: Integer --> Integer
*   predInt: Integer --> Integer
*   negInt: Integer --> Integer
*   addInt: Integer x Integer --> Integer
*   isZeroInt: Integer --> Boolean

**axioms:**
*   (I1): succInt(predInt(i)) = i
*   (I2): predInt(succInt(i)) = i
*   (I3): negInt(zeroInt) = zeroInt
*   (I4): negInt(succInt(i)) = predInt(negInt(i))
*   (I5): addInt(i, zeroInt) = i
*   (I6): addInt(i, succInt(j)) = succInt(addInt(i, j))
*   (I7): addInt(i, predInt(j)) = predInt(addInt(i, j))
*   (I8): isZeroInt(zeroInt) = true
*   (I9): isZeroInt(succInt(i)) = false
*   (I10): isZeroInt(predInt(i)) = false

for all i, j: Integer

**END SPEC**

A especificação `Integer` define o TAD para números inteiros, importando `Boolean`. O sort principal é `Integer`. As **operações** construtoras são `zeroInt`, `succInt` (para `i+1`), e `predInt` (para `i-1`). As operações `negInt` (negação) e `addInt` (adição) são derivadas. O observador `isZeroInt` verifica se um inteiro é zero. Os **axioms** (I1)-(I2) estabelecem `succInt` e `predInt` como inversas. (I3)-(I4) definem `negInt` recursivamente. (I5)-(I7) definem `addInt` recursivamente usando `succInt` e `predInt`. Os axiomas (I8)-(I10) definem `isZeroInt`. (I8) é o caso base. (I9) e (I10) especificam que o sucessor ou predecessor de qualquer inteiro (incluindo aqueles que resultariam em zero) não é zero, a menos que o próprio argumento já fosse tal que o resultado fosse zero (e.g., `succInt(predInt(zeroInt))` é `zeroInt`). Uma definição mais precisa para (I9) e (I10) seria condicionada ou usar um predicado de igualdade para inteiros, mas para simplificar, assumimos que `succInt(i)` é zero somente se `i` for "menos um", e `predInt(i)` é zero somente se `i` for "um". Esta simplificação nos axiomas de `isZeroInt` para `succInt` e `predInt` é uma concessão para brevidade; uma especificação completa e robusta de `Integer` frequentemente requer uma abordagem mais detalhada para esses casos ou o uso de um predicado de igualdade para inteiros, que não foi incluído aqui para manter o exemplo focado nos construtores diretos.

**Exercício:**

Para o TAD `NaturalList` (lista de números `Natural`), que importa `Natural` e `Boolean`, e possui os construtores `emptyList: --> NaturalList` e `cons: (Natural, NaturalList) -> NaturalList`. Adicione uma operação `isSingleton: NaturalList --> Boolean` que retorna `true` se a lista contém exatamente um elemento, e `false` caso contrário. Forneça os axiomas para `isSingleton` usando as operações de `NaturalList` e `Boolean`.

**Resolução:**

Adicionamos a operação `isSingleton` e seus axiomas à especificação `NaturalList`.

**SPEC ADT** NaturalList

**imports:**
*   Natural
*   Boolean

**sorts:**
*   NaturalList

**operations:**
*   emptyList: --> NaturalList
*   cons: Natural x NaturalList --> NaturalList
*   isEmpty: NaturalList --> Boolean
*   isSingleton: NaturalList --> Boolean

**axioms:**
*   (NL1): isEmpty(emptyList) = true
*   (NL2): isEmpty(cons(n, l)) = false
*   (NL3): isSingleton(emptyList) = false
*   (NL4): isSingleton(cons(n, l)) = isEmpty(l)

for all n: Natural, l: NaturalList

**END SPEC**

**Explicação dos acréscimos para `isSingleton`:**

1.  **Declaração da Operação:**
    *   `isSingleton: NaturalList --> Boolean`
    Foi adicionada à seção `operations`. Ela recebe uma `NaturalList` e retorna um `Boolean`.

2.  **Axiomas para `isSingleton`:**
    Definimos o comportamento de `isSingleton` para cada forma construtora de `NaturalList`:
    *   **(NL3): `isSingleton(emptyList) = false`**
        Este é o caso base para `isSingleton`. Uma lista vazia (`emptyList`) não contém um único elemento, portanto, o predicado retorna `false`.
    *   **(NL4): `isSingleton(cons(n, l)) = isEmpty(l)`**
        Este é o caso recursivo (ou de decomposição). Uma lista construída por `cons(n, l)` (onde `n` é um `Natural` e `l` é uma `NaturalList`) contém exatamente um elemento se, e somente se, a "cauda" `l` dessa construção for uma lista vazia. Usamos a operação `isEmpty` (já definida axiomaticamente) aplicada a `l` para determinar isso. Se `isEmpty(l)` for `true`, então `cons(n, l)` tem apenas o elemento `n`, e `isSingleton` deve retornar `true`. Se `isEmpty(l)` for `false`, então `l` já continha elementos, significando que `cons(n, l)` tem mais de um elemento, e `isSingleton` deve retornar `false`. A igualdade direta com `isEmpty(l)` captura essa lógica.

Estes dois axiomas (NL3 e NL4) definem completamente o comportamento da operação `isSingleton` para todas as `NaturalList`s.

Este capítulo forneceu uma introdução abrangente à especificação algébrica de Tipos Abstratos de Dados. Começamos com os fundamentos da lógica equacional e das álgebras multissortidas, definindo os conceitos de sorts, operações e Σ-assinaturas. Exploramos como os termos são construídos sobre uma assinatura, incluindo a distinção entre termos com variáveis e termos fundamentais, e o papel das substituições. Detalhamos como os axiomas equacionais são usados para definir o comportamento das operações, culminando na noção de uma especificação algébrica como um par assinatura-axiomas. A semântica formal foi abordada através da definição de Σ-álgebras, da satisfação de equações e do conceito de modelo de uma especificação, com menção à importância dos modelos iniciais e finais. Classificamos as operações em construtores, observadores e derivadas, e ilustramos todos esses conceitos com especificações detalhadas dos TADs `Boolean`, `Natural` (Peano) e `Integer`. O próximo capítulo se aprofundará na semântica operacional das especificações algébricas, introduzindo os Sistemas de Reescrita de Termos como um mecanismo para "computar" com essas especificações e analisar suas propriedades dinâmicas.

---
## REFERÊNCIAS BIBLIOGRÁFICAS
BERGSTRA, J. A.; HEERING, J.; KLINT, P. (Eds.). **Algebraic Specification**. New York: ACM Press : Addison-Wesley Pub. Co., 1989. (ACM Press frontier series).
*   *Resumo: Uma coletânea clássica de artigos seminais sobre especificação algébrica, cobrindo tanto aspectos teóricos quanto práticos, incluindo linguagens de especificação e ferramentas. Relevante para aprofundar nos fundamentos e nas diversas facetas da abordagem algébrica introduzida no capítulo.*

EHRIG, H.; MAHR, B. **Fundamentals of Algebraic Specification 1**: Equations and Initial Semantics. Berlin: Springer-Verlag, 1985. (EATCS Monographs on Theoretical Computer Science, v. 6).
*   *Resumo: Este é o primeiro volume de uma obra de referência fundamental e rigorosa sobre especificação algébrica. Cobre detalhadamente os conceitos de assinaturas, termos, álgebras, equações, lógica equacional, homomorfismos e a semântica inicial, todos centrais para o conteúdo do Capítulo 2.*

GOGUEN, J. A.; THATCHER, J. W.; WAGNER, E. G. An initial algebra approach to the specification, correctness, and implementation of abstract data types. In: YEH, R. T. (Ed.). **Current Trends in Programming Methodology, Vol. IV: Data Structuring**. Englewood Cliffs, NJ: Prentice-Hall, 1978. p. 80-149.
*   *Resumo: Este é o influente artigo do grupo ADJ que estabeleceu a abordagem de álgebra inicial como uma base semântica para Tipos Abstratos de Dados. É crucial para entender a teoria por trás dos modelos iniciais e a interpretação canônica das especificações algébricas, como brevemente mencionado na Seção 2.3.*

GRÄTZER, G. **Universal Algebra**. 2. ed. New York: Springer-Verlag, 2008.
*   *Resumo: Embora seja um texto avançado sobre álgebra universal, os capítulos iniciais fornecem os fundamentos matemáticos sobre álgebras, homomorfismos, termos e equações que sustentam a teoria da especificação algébrica. Relevante para quem busca um aprofundamento matemático rigoroso nos conceitos de Σ-álgebras e lógica equacional.*

REYNOLDS, J. C. (Eds.). **Algebraic Methods in Semantics**. Cambridge: Cambridge University Press, 1985. p. 459-541.
*   *Resumo: Este capítulo de livro aprofunda a teoria da semântica de álgebra inicial, explorando suas conexões com princípios de indução (cruciais para provar propriedades sobre tipos de dados recursivos) e a teoria da computabilidade. É uma leitura avançada, relevante para entender as fundações lógicas e computacionais da especificação algébrica e da semântica de TADs.*

---

