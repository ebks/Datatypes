---

# CAPÍTULO 2

# ESPECIFICAÇÃO ALGÉBRICA DE TIPOS ABSTRATOS DE DADOS

---

Após a introdução aos conceitos fundamentais de abstração de dados e a problemática da corretude de software no capítulo anterior, este capítulo mergulha profundamente na primeira e principal técnica formal que utilizaremos ao longo desta obra: a **especificação algébrica de Tipos Abstratos de Dados (TADs)**. Esta abordagem, também conhecida como especificação axiomática ou equacional, oferece um formalismo matemático robusto e elegante para definir o comportamento de tipos de dados de maneira abstrata, isto é, independentemente de qualquer representação ou implementação concreta. Iniciaremos nossa jornada estabelecendo os fundamentos da lógica equacional e das álgebras multissortidas, que formam a base teórica da especificação algébrica. Discutiremos os conceitos de *sorts* (os nomes dos tipos de dados), *operações* (as funções que atuam sobre esses tipos) e *assinaturas* (a coleção de sorts e operações com seus perfis). Em seguida, exploraremos a noção de *termos* sobre uma assinatura, que são as expressões sintaticamente válidas que podemos construir usando as operações e variáveis, distinguindo entre termos com variáveis e termos fundamentais (sem variáveis). A seguir, introduziremos o cerne da especificação algébrica: os *axiomas equacionais*. Mostraremos como um conjunto de axiomas, expressos como equações entre termos, define as propriedades semânticas das operações e as relações entre elas, culminando na definição de uma *especificação algébrica* como um par consistindo de uma assinatura e um conjunto de axiomas. A discussão então se voltará para a semântica formal das especificações algébricas, introduzindo o conceito de *$\Sigma$-álgebras* como as estruturas matemáticas que servem de modelo para uma dada assinatura, e como uma equação é satisfeita em uma álgebra. Definiremos a noção de *modelos de uma especificação* – as álgebras que satisfazem todos os seus axiomas – e tocaremos brevemente nos conceitos importantes de modelos iniciais e finais, que capturam diferentes interpretações semânticas de uma especificação. Distinguiremos entre diferentes papéis que as operações podem desempenhar em uma especificação, como *construtores* (usados para gerar todos os valores do tipo), *observadores* (usados para inspecionar valores) e operações derivadas. Finalmente, para solidificar a compreensão, apresentaremos exemplos fundamentais de especificações algébricas, focando nos TADs `Boolean`, `Natural` (baseado nos axiomas de Peano) e `Integer`, ilustrando como os conceitos teóricos se aplicam na prática para definir tipos de dados básicos de forma rigorosa. Cada seção será acompanhada de um exercício para reforçar o aprendizado dos conceitos apresentados.

---

## 2.1 Introdução à Lógica Equacional e Álgebras Multissortidas

A especificação algébrica de Tipos Abstratos de Dados (TADs) é uma abordagem formal que se baseia em conceitos da álgebra universal e da lógica matemática, mais especificamente, da lógica equacional. A ideia central é definir um TAD especificando seus diferentes conjuntos de valores (chamados *sorts*), as operações que podem ser realizadas sobre esses valores (com seus tipos de entrada e saída) e um conjunto de axiomas (geralmente equações) que descrevem as propriedades dessas operações e como elas interagem. Esta seção introduzirá os blocos de construção fundamentais desta abordagem: sorts, operações, assinaturas, e a noção de termos construídos sobre uma assinatura, que são as expressões sintáticas que formam a base para os axiomas.

O termo "multissortido" (many-sorted) refere-se ao fato de que, em geral, um TAD pode envolver múltiplos tipos de dados distintos que interagem entre si. Por exemplo, uma especificação de `NaturalList` (lista de números naturais) envolve o sort `NaturalList` em si, o sort `Natural` para os elementos da lista, e o sort `Boolean` para o resultado de operações como `isEmpty`. Uma álgebra multissortida é, portanto, uma estrutura matemática que consiste em múltiplos conjuntos portadores (um para cada sort) e múltiplas funções (uma para cada operação da assinatura) que operam entre esses conjuntos de acordo com os perfis especificados.

A lógica equacional é o sistema lógico usado para expressar e raciocinar sobre as propriedades dos TADs. Ela se concentra em equações da forma $t_1 = t_2$, onde $t_1$ e $t_2$ são termos (expressões) construídos a partir das operações do TAD e de variáveis. Um conjunto de tais equações (os axiomas) define o comportamento do TAD, e a lógica equacional fornece regras de inferência para derivar novas equações (teoremas) a partir dos axiomas.

### 2.1.1 Sorts, Operações e Assinaturas ($\Sigma$-Signatures)

No formalismo da especificação algébrica, os componentes básicos de um TAD são definidos da seguinte maneira:

1.  **Sorts (Tipos):**
    Um *sort* é simplesmente um nome simbólico para um tipo de dados, ou, mais formalmente, um nome para um conjunto de valores. Por exemplo, em uma especificação, poderíamos ter sorts como `Natural`, `Boolean`, `Integer`, `Character`, `NaturalList`, `Stack`, `Queue`, etc. O conjunto de todos os sorts relevantes para uma determinada especificação é frequentemente denotado por $S$. Cada sort $s \in S$ corresponde, em qualquer modelo (implementação) do TAD, a um conjunto não vazio $M_s$, chamado de *conjunto portador* (carrier set) do sort $s$. Este conjunto contém todos os possíveis valores concretos daquele tipo. Por exemplo, para o sort `Boolean`, o conjunto portador seria tipicamente $\{ \text{verdadeiro}, \text{falso} \}$. Para o sort `Natural`, seria $\{0, 1, 2, \ldots \}$.

2.  **Operações (Funções):**
    Uma *operação* (ou função) define uma maneira de combinar ou transformar valores dos sorts. Cada operação é caracterizada por:
    *   Um **nome** (ou símbolo de operação), por exemplo, `zero`, `succ`, `add`, `true`, `not`, `and`, `nil`, `cons`, `head`.
    *   Um **perfil** (ou tipo funcional, ou aridade e tipagem), que especifica os sorts dos seus argumentos de entrada e o sort do seu resultado. Se uma operação $f$ toma $k$ argumentos, onde o primeiro argumento é do sort $s_1$, o segundo do sort $s_2, \ldots$, e o k-ésimo do sort $s_k$, e retorna um resultado do sort $s_{k+1}$ (que pode ser um dos $s_1, \ldots, s_k$ ou um sort diferente), então seu perfil é escrito como:
        $f: s_1 \times s_2 \times \ldots \times s_k \rightarrow s_{k+1}$
        O produto cartesiano $s_1 \times \ldots \times s_k$ é chamado de *domínio* (ou aridade) da operação, e $s_{k+1}$ é chamado de *contradomínio* (ou sort de resultado ou co-aridade).
        *   Se $k=0$ (nenhum argumento), a operação é uma **constante** do sort $s_{k+1}$. Seu perfil é escrito como $f: \rightarrow s_{k+1}$. Constantes são fundamentais, pois fornecem os "blocos de construção" iniciais para os valores de um sort. Exemplos: `zero: \rightarrow Natural`, `true: \rightarrow Boolean`, `nil: \rightarrow NaturalList`.
        *   Se $k=1$, a operação é **unária**. Exemplo: `succ: Natural \rightarrow Natural`, `not: Boolean \rightarrow Boolean`.
        *   Se $k=2$, a operação é **binária**. Exemplo: `add: Natural \times Natural \rightarrow Natural`, `cons: Natural \times NaturalList \rightarrow NaturalList`.
    É importante notar que o mesmo nome de operação pode, em algumas teorias mais ricas, ser *sobrecarregado* (overloaded), significando que pode ter múltiplos perfis (e.g., um `+` para inteiros e um `+` para floats). No entanto, na forma mais simples de especificação algébrica que consideraremos inicialmente, cada símbolo de operação tem um único perfil.

3.  **Assinaturas ($\Sigma$-Signatures):**
    Uma **assinatura** (frequentemente denotada pela letra grega $\Sigma$, e por isso chamada de $\Sigma$-assinatura) é o que formalmente define a estrutura sintática de um TAD. Uma assinatura $\Sigma$ consiste em duas partes:
    *   Um conjunto $S$ de nomes de sorts.
    *   Um conjunto de símbolos de operação, onde cada símbolo de operação $f$ está associado a um perfil único da forma $f: s_1 \times \ldots \times s_k \rightarrow s_{k+1}$, onde $s_1, \ldots, s_k, s_{k+1}$ são sorts em $S$.

    A assinatura estabelece o vocabulário básico para falar sobre o TAD. Ela nos diz quais tipos de dados existem (os sorts) e quais operações podemos usar para construir ou manipular valores desses tipos (e quais são os tipos de entrada e saída dessas operações). A assinatura, por si só, não atribui qualquer significado ou comportamento às operações; ela apenas define sua sintaxe.

    **Exemplo de Assinatura para `Boolean`:**
    Seja $S_{Bool} = \{ \text{Boolean} \}$.
    A assinatura $\Sigma_{Bool}$ poderia ser:
    *   Sorts: `Boolean`
    *   Operações:
        *   `true: \rightarrow \text{Boolean}`
        *   `false: \rightarrow \text{Boolean}`
        *   `not: \text{Boolean} \rightarrow \text{Boolean}`
        *   `and: \text{Boolean} \times \text{Boolean} \rightarrow \text{Boolean}`
        *   `or: \text{Boolean} \times \text{Boolean} \rightarrow \text{Boolean}`

    **Exemplo de Assinatura para `Natural` (baseado em Peano):**
    Seja $S_{Nat} = \{ \text{Natural}, \text{Boolean} \}$ (assumindo `Boolean` já definido ou como um sort primitivo).
    A assinatura $\Sigma_{Nat}$ poderia ser:
    *   Sorts: `Natural`, `Boolean`
    *   Operações:
        *   `zero: \rightarrow \text{Natural}`
        *   `succ: \text{Natural} \rightarrow \text{Natural}`
        *   `add: \text{Natural} \times \text{Natural} \rightarrow \text{Natural}`
        *   `isZero: \text{Natural} \rightarrow \text{Boolean}`

    Uma assinatura é, portanto, o primeiro passo para a formalização de um TAD. Ela fornece o esqueleto sintático sobre o qual a semântica (através dos axiomas) será construída. A escolha dos sorts e das operações, e seus perfis, é uma decisão de modelagem crucial. Uma boa assinatura deve ser intuitiva, suficientemente expressiva para capturar as funcionalidades essenciais do TAD, e, idealmente, levar a uma especificação axiomática concisa e compreensível. Operações podem ser classificadas em *construtores* (que são suficientes para gerar todos os valores de um sort) e *observadores* ou *operações derivadas* (que consultam ou transformam valores, mas podem ser definidas em termos dos construtores). Esta distinção será explorada mais adiante (Seção 2.4).

**Exercício**
Considere um TAD simples para representar um "Ponto" em um plano 2D com coordenadas inteiras.
a) Quais seriam os sorts relevantes para este TAD?
b) Sugira pelo menos três operações para este TAD `Ponto`, incluindo uma constante (construtor), uma operação para acessar uma coordenada, e uma operação para calcular a distância (ao quadrado, para manter em inteiros) até a origem (0,0). Forneça os perfis dessas operações.
c) Escreva a assinatura $\Sigma_{Ponto}$ completa para este TAD, com base em suas respostas para (a) e (b). (Assuma que o sort `Integer` já existe).

**Resolução:**
a) **Sorts Relevantes para `Ponto`:**
    *   `Ponto`: O sort principal, representando um ponto no plano 2D.
    *   `Integer`: Um sort auxiliar (assumido como existente) para representar as coordenadas inteiras e o resultado da distância ao quadrado.

b) **Operações Sugeridas para `Ponto`:**
    1.  **Construtor/Constante (origem):**
        *   Nome: `origin`
        *   Perfil: `origin: --> Ponto` (cria o ponto na origem (0,0))
        Alternativamente, um construtor mais geral:
        *   Nome: `makePoint`
        *   Perfil: `makePoint: Integer x Integer --> Ponto` (cria um ponto com coordenadas x, y dadas)
    2.  **Acessor de Coordenada:**
        *   Nome: `getX`
        *   Perfil: `getX: Ponto --> Integer` (retorna a coordenada x do ponto)
        (Similarmente, poderíamos ter `getY: Ponto --> Integer`)
    3.  **Distância ao Quadrado até a Origem:**
        *   Nome: `distSqToOrigin`
        *   Perfil: `distSqToOrigin: Ponto --> Integer` (retorna $x^2 + y^2$)

c) **Assinatura $\Sigma_{Ponto}$ Completa:**
    (Usando o construtor `makePoint` para maior generalidade, e incluindo `getY`)

    *   Sorts: `Ponto`, `Integer`
    *   Operações:
        *   `makePoint: Integer \times Integer \rightarrow Ponto`
        *   `getX: Ponto \rightarrow Integer`
        *   `getY: Ponto \rightarrow Integer`
        *   `distSqToOrigin: Ponto \rightarrow Integer`

    (Se fôssemos incluir `origin` como uma constante e as outras operações, seria:
    *   Sorts: `Ponto`, `Integer`
    *   Operações:
        *   `origin: \rightarrow Ponto`
        *   `makePointFromOriginOffset: Integer \times Integer \rightarrow Ponto` (poderia ser um construtor alternativo, ou `makePoint` ser o único)
        *   `getX: Ponto \rightarrow Integer`
        *   `getY: Ponto \rightarrow Integer`
        *   `distSqToOrigin: Ponto \rightarrow Integer`
    Para este exercício, a primeira versão com `makePoint` é mais concisa e geral como o construtor principal.)
□

### 2.1.2 Termos sobre uma Assinatura ($T_{\Sigma}(X)$). Variáveis e Termos Fundamentais (`Ground Terms`)

Uma vez que temos uma assinatura $\Sigma = (S, \text{Op})$ (onde $\text{Op}$ é o conjunto de símbolos de operação com seus perfis), podemos construir expressões sintaticamente válidas usando as operações e, possivelmente, variáveis. Essas expressões são chamadas de **termos**. Os termos são fundamentais porque são os objetos sobre os quais os axiomas (equações) são escritos. A semântica de um TAD é definida especificando quais termos são considerados "iguais".

Formalmente, para uma dada assinatura $\Sigma$, e um conjunto $X$ de *variáveis* (onde cada variável $x \in X$ também tem um sort associado, $sort(x) \in S$), o conjunto de todos os **$\Sigma$-termos com variáveis em $X$**, denotado por $T_{\Sigma}(X)$, é definido indutivamente da seguinte maneira:

1.  **Caso Base (Variáveis e Constantes):**
    *   Toda variável $x \in X$ do sort $s$ é um termo do sort $s$.
    *   Toda constante $c: \rightarrow s$ na assinatura $\Sigma$ é um termo do sort $s$.

2.  **Passo Indutivo (Aplicação de Operações):**
    *   Se $f: s_1 \times s_2 \times \ldots \times s_k \rightarrow s$ é um símbolo de operação em $\Sigma$ (com $k \ge 1$), e $t_1, t_2, \ldots, t_k$ são termos (já construídos) tais que $sort(t_1) = s_1, sort(t_2) = s_2, \ldots, sort(t_k) = s_k$, então a expressão $f(t_1, t_2, \ldots, t_k)$ é um termo do sort $s$.

O conjunto $T_{\Sigma}(X)$ é o menor conjunto que satisfaz essas duas condições. Cada termo $t \in T_{\Sigma}(X)$ tem um sort bem definido, $sort(t) \in S$. Se usarmos a notação $T_{\Sigma,s}(X)$ para denotar o conjunto de todos os $\Sigma$-termos do sort $s$ com variáveis em $X$, então $T_{\Sigma}(X) = \bigcup_{s \in S} T_{\Sigma,s}(X)$.

**Variáveis:**
As variáveis em $X$ atuam como "placeholders" para valores dos seus respectivos sorts. Quando escrevemos axiomas, as variáveis são geralmente entendidas como sendo *universalmente quantificadas*. Por exemplo, um axioma `add(n, zero) = n` significa que esta equação deve valer para *qualquer* valor natural que a variável `n` possa assumir. É comum ter um conjunto infinito de variáveis disponíveis para cada sort, e.g., $X_s = \{x_1^s, x_2^s, \ldots \}$ para cada $s \in S$.

**Termos Fundamentais (`Ground Terms`):**
Um caso especial e muito importante é quando o conjunto de variáveis $X$ é vazio ($X = \emptyset$). Os termos construídos sem usar nenhuma variável são chamados de **termos fundamentais** (ou *ground terms*). O conjunto de todos os $\Sigma$-termos fundamentais é denotado por $T_{\Sigma}$ (ou $T_{\Sigma}(\emptyset)$).
Os termos fundamentais de um sort $s$, $T_{\Sigma,s}$, representam todas as expressões que podem ser construídas usando apenas as constantes e as operações da assinatura para produzir um valor do sort $s$. Em muitas semânticas de TADs (como a semântica inicial, que veremos), os termos fundamentais (ou suas classes de equivalência sob os axiomas) são considerados como os próprios *valores* do tipo de dados.

**Exemplos de Termos:**
Usando a assinatura $\Sigma_{Nat}$ da Seção 2.1.1 (com sorts `Natural`, `Boolean` e operações `zero`, `succ`, `add`, `isZero`):
Seja $X_{Nat} = \{n, m\}$ um conjunto de variáveis do sort `Natural`, e $X_{Bool} = \{b\}$ uma variável do sort `Boolean`.

*   **Termos Fundamentais ($T_{\Sigma_{Nat}}$):**
    *   `zero` (sort `Natural`)
    *   `succ(zero)` (sort `Natural`)
    *   `succ(succ(zero))` (sort `Natural`)
    *   `add(succ(zero), zero)` (sort `Natural`)
    *   `isZero(zero)` (sort `Boolean`)
    *   `isZero(add(succ(zero), succ(succ(zero))))` (sort `Boolean`)
    (Assumindo que `true` e `false` não estão em $\Sigma_{Nat}$ diretamente, mas são resultados em `Boolean`; se `Boolean` tivesse seus próprios construtores como `true_const: -> Boolean`, então `true_const` seria um termo fundamental de sort `Boolean`).

*   **Termos com Variáveis ($T_{\Sigma_{Nat}}(X_{Nat} \cup X_{Bool})$):**
    *   `n` (sort `Natural`)
    *   `m` (sort `Natural`)
    *   `succ(n)` (sort `Natural`)
    *   `add(n, m)` (sort `Natural`)
    *   `add(succ(n), zero)` (sort `Natural`)
    *   `isZero(m)` (sort `Boolean`)
    *   `add(n, add(m, zero))` (sort `Natural`)

Note que cada termo é construído seguindo estritamente os perfis das operações. Uma expressão como `succ(isZero(zero))` seria malformada (ou mal tipada), porque `isZero(zero)` é do sort `Boolean`, mas `succ` espera um argumento do sort `Natural`.

**Representação de Termos como Árvores:**
Os termos podem ser naturalmente visualizados como árvores, onde:
*   As folhas são as constantes ou as variáveis.
*   Os nós internos são os símbolos de operação.
*   Os filhos de um nó de operação são os subtermos que são os argumentos daquela operação.
Por exemplo, o termo `add(succ(n), zero)` pode ser representado como:
```
      add
     /   \
   succ  zero
    |
    n
```
Esta estrutura em árvore é fundamental para muitas operações sobre termos, como substituição, unificação e reescrita, que serão discutidas posteriormente.

A capacidade de construir termos é o primeiro passo para dar significado a uma assinatura. Os axiomas de uma especificação algébrica serão equações entre termos, como `add(n, zero) = n`, onde ambos os lados da equação são termos (neste caso, com a variável `n`).

**Exercício**
Usando a assinatura $\Sigma_{Nat}$ (com `zero`, `succ`, `add`, `isZero`) e a assinatura $\Sigma_{Bool}$ (com `true`, `false`, `not`, `and`) da Seção 2.1.1:
a) Forneça três exemplos de termos fundamentais do sort `Natural` que usem a operação `add`.
b) Forneça um exemplo de um termo do sort `Boolean` que use variáveis `n: Natural` e `b: Boolean`, e as operações `isZero`, `not`, e `and`.
c) Desenhe a representação em árvore para o termo `add(succ(zero), add(zero, succ(zero)))`.

**Resolução:**
a) **Três exemplos de termos fundamentais do sort `Natural` usando `add`:**
    1.  `add(zero, zero)`
    2.  `add(succ(zero), zero)`
    3.  `add(succ(zero), succ(succ(zero)))`
    (Outros exemplos: `succ(add(zero, succ(zero)))`, `add(add(zero,zero), zero)`)

b) **Exemplo de termo do sort `Boolean` com variáveis `n: Natural`, `b: Boolean` e operações `isZero`, `not`, `and`:**
    `and(not(isZero(n)), b)`

    (Explicação: `isZero(n)` é um termo do sort `Boolean`. `not(isZero(n))` também é do sort `Boolean`. `b` é uma variável do sort `Boolean`. Portanto, `and(not(isZero(n)), b)` é um termo bem formado do sort `Boolean`.)

c) **Representação em árvore para o termo `add(succ(zero), add(zero, succ(zero)))`:**
```
           add
          /   \
       succ    add
        |     /   \
      zero  zero  succ
                  |
                zero
```
□

### 2.1.3 Substituições e Instanciação

Os termos com variáveis, como `add(n, m)`, representam padrões ou esquemas. Para dar-lhes um significado mais concreto ou para aplicar axiomas que são geralmente escritos com variáveis, precisamos do conceito de **substituição** e **instanciação**.

Uma **substituição** é uma função (ou mapeamento) que associa variáveis a termos. Mais formalmente, dado um conjunto $S$ de sorts e um conjunto $X = \bigcup_{s \in S} X_s$ de variáveis (onde $X_s$ são as variáveis do sort $s$), uma **$\Sigma$-substituição** (ou simplesmente substituição, se $\Sigma$ estiver claro pelo contexto) é uma coleção de mapeamentos $\sigma = \{ \sigma_s : X_s \rightarrow T_{\Sigma,s}(X) \}_{s \in S}$. Ou seja, para cada sort $s$, $\sigma_s$ mapeia cada variável $x \in X_s$ para um $\Sigma$-termo $\sigma_s(x)$ do mesmo sort $s$ (possivelmente contendo outras variáveis de $X$). Na prática, uma substituição é frequentemente definida apenas para um subconjunto finito de variáveis, assumindo-se que ela é a identidade para as variáveis não mencionadas. Uma substituição é tipicamente escrita como $\{x_1 \mapsto t_1, x_2 \mapsto t_2, \ldots, x_k \mapsto t_k\}$, onde cada $x_i$ é uma variável e $t_i$ é um termo do mesmo sort que $x_i$, e $x_i \neq x_j$ para $i \neq j$.

A **aplicação de uma substituição $\sigma$ a um termo $t$**, denotada por $t\sigma$ (ou $\sigma(t)$ ou $t[\sigma]$), resulta em um novo termo obtido substituindo simultaneamente cada ocorrência de cada variável $x$ em $t$ pelo termo $\sigma(x)$ ao qual $x$ é mapeada por $\sigma$. A aplicação é definida indutivamente sobre a estrutura do termo $t$:
1.  **Se $t$ é uma variável $x$**:
    *   $x\sigma = \sigma(x)$ (se $x$ está no domínio de $\sigma$).
    *   $x\sigma = x$ (se $x$ não está no domínio de $\sigma$, ou seja, $\sigma$ é a identidade para $x$).
2.  **Se $t$ é uma constante $c: \rightarrow s$**:
    *   $c\sigma = c$ (constantes não são afetadas por substituições de variáveis).
3.  **Se $t$ é da forma $f(t_1, t_2, \ldots, t_k)$**:
    *   $(f(t_1, t_2, \ldots, t_k))\sigma = f(t_1\sigma, t_2\sigma, \ldots, t_k\sigma)$.
    (A substituição é aplicada recursivamente aos subtermos).

O termo $t\sigma$ é chamado de **instância** do termo $t$ (obtida pela substituição $\sigma$). Se todos os termos $t_i$ na substituição $\sigma = \{x_1 \mapsto t_1, \ldots, x_k \mapsto t_k\}$ são termos fundamentais (ground terms), então $t\sigma$ também será um termo fundamental (assumindo que todas as variáveis em $t$ estão no domínio de $\sigma$). Nesse caso, $t\sigma$ é chamado de **instância fundamental** de $t$.

**Exemplo de Substituição e Instanciação:**
Usando a assinatura $\Sigma_{Nat}$ e o termo $t = \text{add}(\text{succ}(n), m)$, onde $n, m$ são variáveis do sort `Natural`.
Seja a substituição $\sigma = \{n \mapsto \text{zero}, m \mapsto \text{succ(zero)}\}$.
Então, $t\sigma = (\text{add}(\text{succ}(n), m))\sigma$
$= \text{add}((\text{succ}(n))\sigma, m\sigma)$ (pela regra 3)
$= \text{add}(\text{succ}(n\sigma), m\sigma)$ (pela regra 3 para `succ`)
$= \text{add}(\text{succ}(\text{zero}), \text{succ(zero)})$ (pela regra 1, substituindo $n$ e $m$)

O termo resultante, `add(succ(zero), succ(zero))`, é uma instância fundamental do termo original `add(succ(n), m)`.

As substituições são cruciais para a aplicação de axiomas. Um axioma como `add(n, zero) = n` é uma "regra" que se aplica a qualquer instância deste padrão. Por exemplo, se quisermos aplicar este axioma ao termo `add(succ(zero), zero)`, podemos ver que este termo é uma instância do lado esquerdo do axioma com a substituição $\{n \mapsto \text{succ(zero)}\}$. Portanto, podemos "reescrever" `add(succ(zero), zero)` para `succ(zero)` (o lado direito do axioma instanciado com a mesma substituição). Este processo de reescrita, que será formalizado no Capítulo 3, é a base para a "computação" ou "avaliação" de termos em uma especificação algébrica.

**Composição de Substituições:**
Se $\sigma_1$ e $\sigma_2$ são duas substituições, sua **composição**, denotada por $\sigma_1\sigma_2$ (ou $\sigma_2 \circ \sigma_1$, dependendo da convenção de aplicação), é uma nova substituição tal que para qualquer variável $x$, $x(\sigma_1\sigma_2) = (x\sigma_1)\sigma_2$. Ou seja, primeiro aplica-se $\sigma_1$ a $x$, resultando em um termo $t_x = x\sigma_1$, e depois aplica-se $\sigma_2$ ao termo $t_x$. A composição é associativa: $(\sigma_1\sigma_2)\sigma_3 = \sigma_1(\sigma_2\sigma_3)$. Existe uma substituição identidade $\epsilon$ (que mapeia cada variável para si mesma) tal que $\sigma\epsilon = \epsilon\sigma = \sigma$.

O conceito de substituição é fundamental não apenas para a lógica equacional, mas também para áreas como programação lógica (onde a unificação, que busca encontrar uma substituição que torna dois termos iguais, é central) e sistemas de prova de teoremas.

**Exercício**
Dado o termo $t = \text{add}(n, \text{succ}(m))$ do sort `Natural` (onde `n, m` são variáveis do sort `Natural`) e a especificação de `Natural` da Seção 1.1.2.
Considere as seguintes substituições:
$\sigma_1 = \{n \mapsto \text{succ(zero)}, m \mapsto \text{zero}\}$
$\sigma_2 = \{n \mapsto m, m \mapsto n\}$ (renomeando variáveis)
$\sigma_3 = \{m \mapsto \text{add(k, zero)}\}$ (onde `k` é outra variável do sort `Natural`)

a) Calcule $t\sigma_1$. O resultado é um termo fundamental?
b) Calcule $t\sigma_2$.
c) Calcule $t\sigma_3$.

**Resolução:**
O termo é $t = \text{add}(n, \text{succ}(m))$.

a) **Cálculo de $t\sigma_1$ com $\sigma_1 = \{n \mapsto \text{succ(zero)}, m \mapsto \text{zero}\}$:**
   $t\sigma_1 = (\text{add}(n, \text{succ}(m)))\sigma_1$
   $= \text{add}(n\sigma_1, (\text{succ}(m))\sigma_1)$
   $= \text{add}(n\sigma_1, \text{succ}(m\sigma_1))$
   Substituindo $n$ por `succ(zero)` e $m$ por `zero`:
   $= \text{add}(\text{succ(zero)}, \text{succ(zero)})$
   Sim, o resultado `add(succ(zero), succ(zero))` é um termo fundamental, pois não contém variáveis.

b) **Cálculo de $t\sigma_2$ com $\sigma_2 = \{n \mapsto m, m \mapsto n\}$:**
   $t\sigma_2 = (\text{add}(n, \text{succ}(m)))\sigma_2$
   $= \text{add}(n\sigma_2, (\text{succ}(m))\sigma_2)$
   $= \text{add}(n\sigma_2, \text{succ}(m\sigma_2))$
   Substituindo $n$ por `m` e $m$ por `n` (assumindo que `m` e `n` na substituição são as *novas* variáveis):
   $= \text{add}(m, \text{succ}(n))$
   Este termo ainda contém as variáveis `m` e `n` (embora trocadas de posição em relação ao original).

c) **Cálculo de $t\sigma_3$ com $\sigma_3 = \{m \mapsto \text{add(k, zero)}\}$ (onde `k` é uma variável do sort `Natural`):**
   $t\sigma_3 = (\text{add}(n, \text{succ}(m)))\sigma_3$
   $= \text{add}(n\sigma_3, (\text{succ}(m))\sigma_3)$
   $= \text{add}(n\sigma_3, \text{succ}(m\sigma_3))$
   A variável `n` não está no domínio de $\sigma_3$, então $n\sigma_3 = n$.
   A variável `m` é mapeada para `add(k, zero)`.
   $= \text{add}(n, \text{succ(add(k, zero))})$
   Este termo contém as variáveis `n` e `k`.
□

---
*O Capítulo 2, Seção 2.1, está agora com uma boa extensão. Continuarei com as próximas seções (2.2, 2.3, etc.) para atingir a meta de palavras, mantendo um exercício por seção de nível 2 e as demais diretrizes.*
---
(Continuando a expansão do Capítulo 2)

## 2.2 Axiomas Equacionais e Especificações Algébricas

Com os conceitos de assinatura e termos (com e sem variáveis) estabelecidos, estamos prontos para abordar o cerne da especificação algébrica: os **axiomas equacionais**. São esses axiomas que conferem semântica às operações definidas na assinatura, especificando como elas devem se comportar e interagir. Uma especificação algébrica completa é então definida como um par consistindo de uma assinatura e um conjunto desses axiomas. Adicionalmente, a lógica equacional fornece um sistema de dedução para derivar novas igualdades (teoremas) a partir dos axiomas dados, permitindo-nos raciocinar sobre as propriedades do Tipo Abstrato de Dados especificado.

### 2.2.1 Definição de Axiomas como Equações entre Termos

Na abordagem de especificação algébrica mais comum e que adotaremos predominantemente, os **axiomas** são **equações** da forma:
$t_L = t_R$
onde $t_L$ (o termo esquerdo, *left-hand side*) e $t_R$ (o termo direito, *right-hand side*) são $\Sigma$-termos do mesmo sort, possivelmente contendo variáveis de um conjunto $X$. Ou seja, se $X_S = \{x_1:s_1, \ldots, x_k:s_k\}$ é o conjunto de variáveis que podem aparecer em $t_L$ ou $t_R$, então para um dado sort $s \in S$, tanto $t_L$ quanto $t_R$ devem pertencer a $T_{\Sigma,s}(X_S)$.

A interpretação de tal equação axiomática é que ela deve ser válida para **todas as possíveis instanciações das variáveis** por termos fundamentais (ou, mais abstratamente, por valores nos conjuntos portadores de qualquer álgebra que seja um modelo da especificação). Essa quantificação universal sobre as variáveis é geralmente implícita, mas às vezes é tornada explícita pela notação "for all $x_1:s_1, \ldots, x_k:s_k, t_L = t_R$".

**Exemplos de Axiomas Equacionais:**
Revisitando a especificação do TAD `Natural` da Seção 1.1.2, os axiomas eram:
*   (Axiom N1): `isZero(zero) = true` (Aqui, `true` seria uma constante do sort `Boolean`. Este axioma envolve termos de sort `Boolean`.)
*   (Axiom N2): `isZero(succ(n)) = false` (Envolve a variável `n: Natural`. `false` é outra constante `Boolean`.)
*   (Axiom N3): `add(n, zero) = n` (Envolve `n: Natural`. Ambos os lados são do sort `Natural`.)
*   (Axiom N4): `add(n, succ(m)) = succ(add(n, m))` (Envolve `n, m: Natural`. Ambos os lados são do sort `Natural`.)

E para o TAD `NaturalList` da Seção 1.2.1:
*   (Axiom NL1): `isEmpty(nil) = true`
*   (Axiom NL3): `head(cons(n, l)) = n` (Envolve `n: Natural, l: NaturalList`. Ambos os lados são do sort `Natural`.)
*   (Axiom NL6): `append(cons(n, l1), l2) = cons(n, append(l1, l2))` (Envolve `n: Natural, l1, l2: NaturalList`. Ambos os lados são do sort `NaturalList`.)

**O Papel dos Axiomas:**
Os axiomas servem para:
1.  **Definir o Comportamento das Operações:** Especialmente para operações não construtoras (como `add`, `isEmpty`, `head`), os axiomas especificam como elas se comportam quando aplicadas a termos construídos com os construtores (como `zero`, `succ`, `nil`, `cons`).
2.  **Estabelecer Relações entre Operações:** Podem mostrar como uma operação pode ser definida em termos de outras (e.g., o axioma de De Morgan `or(b1,b2) = not(and(not(b1),not(b2)))` para `Boolean` define `or` em termos de `and` e `not`).
3.  **Identificar Termos Equivalentes:** A relação de igualdade definida pelos axiomas particiona o conjunto de todos os termos (especialmente os termos fundamentais) em classes de equivalência. Todos os termos em uma mesma classe de equivalência são considerados como representando o mesmo "valor abstrato". Por exemplo, para `Natural`, os termos `add(succ(zero), zero)` e `succ(zero)` seriam equivalentes (ambos representam o número 1) devido ao Axioma N3.

**Axiomas Condicionais (Breve Menção):**
Embora este livro se concentre primariamente em axiomas puramente equacionais, é importante notar que algumas extensões da especificação algébrica permitem **axiomas condicionais**. Um axioma condicional tem a forma:
$u_1 = v_1 \land u_2 = v_2 \land \ldots \land u_k = v_k \implies t_L = t_R$
onde $u_i, v_i, t_L, t_R$ são termos. Isso significa que a equação $t_L = t_R$ só é válida se todas as condições $u_i = v_i$ forem satisfeitas. Axiomas condicionais aumentam o poder expressivo, mas também a complexidade da teoria e das ferramentas de análise. Por exemplo, a operação `head` para `NaturalList` que é parcial (só definida para listas não vazias) poderia ser especificada com um axioma condicional (embora existam outras técnicas):
`not(isEmpty(l)) = true implies head(cons(first_element_of_l_if_it_exists, tail_of_l_if_it_exists_when_l_is_cons_form)) = first_element_of_l_if_it_exists`
No entanto, a forma mais comum e simples, como `head(cons(n,l)) = n`, lida com isso implicitamente, definindo `head` apenas para termos que casam com o padrão `cons(n,l)`. A questão de como tratar operações parciais e erros é um tópico importante na especificação algébrica, que será abordado com mais detalhes em contextos específicos mais adiante (e.g., ao discutir completeza e consistência no Capítulo 4).

A escolha de um bom conjunto de axiomas é uma arte. Idealmente, os axiomas devem ser:
*   **Consistentes:** Não devem permitir a derivação de contradições (e.g., `true = false`, ou `zero = succ(zero)` se isso for indesejado).
*   **Suficientemente Completos:** Devem definir o comportamento das operações para todas as entradas "relevantes" (e.g., para todas as combinações de construtores).
*   **Intuitivos e Legíveis:** Devem refletir claramente as propriedades intencionadas do TAD.
*   **Independentes (Opcional, mas Desejável):** Nenhum axioma deve ser derivável dos outros (para minimalidade), embora redundâncias possam às vezes ser incluídas para clareza.

### 2.2.2 Uma Especificação Algébrica SP = ($\Sigma$, E) como um par Assinatura-Axiomas

Com os conceitos de assinatura e axiomas equacionais definidos, podemos agora formalizar o que é uma **especificação algébrica**.

Uma **especificação algébrica** (ou simplesmente **especificação**), denotada por $SP$, é um par:
$SP = (\Sigma, E)$
onde:
*   $\Sigma = (S, \text{Op})$ é uma **assinatura**, consistindo de um conjunto $S$ de sorts e um conjunto $\text{Op}$ de símbolos de operação com seus perfis sobre $S$.
*   $E$ é um **conjunto de axiomas**, onde cada axioma em $E$ é uma $\Sigma$-equação (uma equação entre $\Sigma$-termos do mesmo sort, possivelmente com variáveis universalmente quantificadas).

Esta definição é central. Ela encapsula a ideia de que um TAD é definido sintaticamente por sua assinatura $\Sigma$ e semanticamente pelas propriedades impostas por seus axiomas $E$.

**Exemplo de Especificação Algébrica Completa para `Boolean`:**

$SP_{Bool} = (\Sigma_{Bool}, E_{Bool})$
onde:
*   $\Sigma_{Bool} = (S_{Bool}, \text{Op}_{Bool})$ com:
    *   $S_{Bool} = \{\text{Boolean}\}$
    *   $\text{Op}_{Bool} = \{$
        `true: \rightarrow \text{Boolean}`,
        `false: \rightarrow \text{Boolean}`,
        `not: \text{Boolean} \rightarrow \text{Boolean}`,
        `and: \text{Boolean} \times \text{Boolean} \rightarrow \text{Boolean}`,
        `or: \text{Boolean} \times \text{Boolean} \rightarrow \text{Boolean}`
        $\}$
*   $E_{Bool} = \{$
    1.  `not(true) = false`
    2.  `not(false) = true`
    3.  `and(true, b) = b`
    4.  `and(false, b) = false`
    5.  `or(true, b) = true`
    6.  `or(false, b) = b`
    (onde `b` é uma variável do sort `Boolean`. Outros axiomas como comutatividade, associatividade, ou leis de De Morgan poderiam ser adicionados ou derivados destes, dependendo da escolha do conjunto "base" de axiomas). Por exemplo, poderíamos ter optado por:
    3'. `and(b, true) = b` (para completar a identidade)
    4'. `and(b, false) = false`
    5'. `or(b, true) = true`
    6'. `or(b, false) = b`
    7. `and(b1,b2) = and(b2,b1)` (Comutatividade de and)
    8. `or(b1,b2) = or(b2,b1)` (Comutatividade de or)
    9. `or(b1,b2) = not(and(not(b1),not(b2)))` (De Morgan, se `or` for definido em termos de `and` e `not`)
    $\}$
    (Na cláusula `for all` implícita ou explícita: `for all b, b1, b2: Boolean`)

Esta especificação $SP_{Bool}$ define formalmente o Tipo Abstrato de Dados `Boolean`. Qualquer implementação correta de `Boolean` (seja em hardware ou software) deve fornecer um conjunto com (pelo menos) dois valores distintos correspondendo a `true` e `false`, e funções para `not`, `and`, `or` que satisfaçam todas essas equações.

O objetivo de uma especificação algébrica não é apenas definir um TAD, mas também fornecer uma base para:
*   **Análise Formal:** Estudar propriedades da especificação como consistência, completeza, e a relação entre diferentes especificações (e.g., refinamento, implementação).
*   **Verificação de Implementações:** Provar que uma implementação concreta satisfaz a especificação.
*   **Geração de Testes:** Derivar casos de teste (especialmente testes baseados em propriedades) a partir dos axiomas.
*   **Execução Simbólica/Reescrita:** Usar os axiomas como regras de reescrita para "calcular" o valor de termos.

A teoria das especificações algébricas é rica e possui muitas variantes e extensões (e.g., para lidar com operações parciais, erros, polimorfismo, especificações parametrizadas, aspectos de ordem superior, etc.). Nosso foco inicial será na forma mais básica e fundamental, baseada em lógica equacional e álgebras totais.

**Exercício**
Dada a especificação $SP_{Nat} = (\Sigma_{Nat}, E_{Nat})$ para o TAD `Natural` (com `zero`, `succ`, `add`, `isZero` e os Axiomas N1-N4 da Seção 1.1.2).
a) Escreva a assinatura $\Sigma_{Nat}$ completa.
b) Escreva o conjunto de axiomas $E_{Nat}$ completo.
c) Usando a variável `x: Natural`, qual dos seguintes é um axioma válido em $E_{Nat}$ (ou uma instância direta de um)?
    i) `add(x, zero) = zero`
    ii) `isZero(succ(x)) = false`
    iii) `add(zero, succ(x)) = succ(x)`

**Resolução:**
a) **Assinatura $\Sigma_{Nat}$:**
    *   Sorts $S_{Nat} = \{\text{Natural}, \text{Boolean}\}$
    *   Operações $\text{Op}_{Nat} = \{$
        `zero: \rightarrow \text{Natural}`,
        `succ: \text{Natural} \rightarrow \text{Natural}`,
        `add: \text{Natural} \times \text{Natural} \rightarrow \text{Natural}`,
        `isZero: \text{Natural} \rightarrow \text{Boolean}`
        $\}$
    (Assumindo que `Boolean` e suas constantes `true`/`false` são dadas ou especificadas separadamente).

b) **Conjunto de Axiomas $E_{Nat}$:**
    (Com variáveis `n, m: Natural` implícitamente ou explicitamente quantificadas universalmente)
    1.  (Axiom N1): `isZero(zero) = true`
    2.  (Axiom N2): `isZero(succ(n)) = false`
    3.  (Axiom N3): `add(n, zero) = n`
    4.  (Axiom N4): `add(n, succ(m)) = succ(add(n, m))`

c) **Validando Axiomas com `x: Natural`:**
    i) `add(x, zero) = zero`:
       O Axioma N3 é `add(n, zero) = n`. Se substituirmos `n` por `x`, obtemos `add(x, zero) = x`.
       Portanto, `add(x, zero) = zero` **não é** um axioma em $E_{Nat}$ nem uma instância direta, a menos que `x` seja `zero` (nesse caso `add(zero,zero)=zero` que é uma instância de N3). A forma geral `add(x, zero) = zero` é falsa para `x != zero`.

    ii) `isZero(succ(x)) = false`:
        O Axioma N2 é `isZero(succ(n)) = false`. Se substituirmos `n` por `x`, obtemos `isZero(succ(x)) = false`.
        Portanto, `isZero(succ(x)) = false` **é** uma instância direta de um axioma em $E_{Nat}$ (Axioma N2).

    iii) `add(zero, succ(x)) = succ(x)`:
         Este não é diretamente um dos axiomas N1-N4. No entanto, pode ser um *teorema* derivável.
         Vamos tentar usar os axiomas:
         O Axioma N4 é `add(n, succ(m)) = succ(add(n, m))`.
         Se instanciarmos com $n \mapsto \text{zero}$ e $m \mapsto x$, obtemos:
         `add(zero, succ(x)) = succ(add(zero, x))`
         Agora, precisamos avaliar `add(zero, x)`. O Axioma N3 é `add(n, zero) = n`. Este não se aplica diretamente a `add(zero, x)` (a menos que x seja zero). Precisaríamos de um axioma como `add(zero, n) = n` ou provar a comutatividade da adição primeiro.
         Se tivéssemos `add(zero, n) = n` como um axioma (ou teorema):
         Então `succ(add(zero, x))` se tornaria `succ(x)`.
         Portanto, `add(zero, succ(x)) = succ(x)` seria um teorema se `add(zero, n) = n` fosse verdadeiro.
         Sem essa propriedade adicional, `add(zero, succ(x)) = succ(x)` **não é** um axioma em $E_{Nat}$ nem uma instância direta imediata. (Será um teorema se a especificação for "boa", mas a questão é sobre ser um axioma ou instância direta).
□

### 2.2.3 Consequência Lógica e Derivação (Regras de Inferência da Lógica Equacional)

Uma especificação algébrica $SP = (\Sigma, E)$ define um conjunto base de axiomas $E$. No entanto, o poder de uma especificação formal reside também na capacidade de **derivar novas verdades** (teoremas) a partir desses axiomas. A noção de "verdade" aqui é a igualdade entre termos. A lógica equacional fornece um conjunto de regras de inferência que nos permitem deduzir novas equações a partir de um conjunto dado de equações (os axiomas).

Se uma equação $t_L = t_R$ pode ser derivada dos axiomas $E$ usando as regras de inferência da lógica equacional, dizemos que $t_L = t_R$ é uma **consequência lógica** de $E$, ou que $E$ **implica logicamente** $t_L = t_R$. Isso é frequentemente denotado por $E \models t_L = t_R$. O conjunto de todas as equações que são consequências lógicas de $E$ forma a **teoria equacional** apresentada por $SP$.

As regras de inferência fundamentais da lógica equacional (para igualdade) são:

1.  **Reflexividade:** Para qualquer termo $t$, a equação $t = t$ é derivável.
    $\overline{t = t}$ (Refl)

2.  **Simetria:** Se a equação $t_1 = t_2$ é derivável, então a equação $t_2 = t_1$ também é derivável.
    $\frac{t_1 = t_2}{t_2 = t_1}$ (Symm)

3.  **Transitividade:** Se as equações $t_1 = t_2$ e $t_2 = t_3$ são deriváveis, então a equação $t_1 = t_3$ também é derivável.
    $\frac{t_1 = t_2, \quad t_2 = t_3}{t_1 = t_3}$ (Trans)

4.  **Congruência (ou Substitutividade):** Se $f: s_1 \times \ldots \times s_k \rightarrow s$ é um símbolo de operação em $\Sigma$, e temos equações deriváveis $t_1 = u_1, t_2 = u_2, \ldots, t_k = u_k$ (onde $sort(t_i) = sort(u_i) = s_i$), então a equação $f(t_1, \ldots, t_k) = f(u_1, \ldots, u_k)$ também é derivável. Isso significa que podemos substituir "iguais por iguais" dentro de um contexto de operação.
    $\frac{t_1 = u_1, \quad \ldots, \quad t_k = u_k}{f(t_1, \ldots, t_k) = f(u_1, \ldots, u_k)}$ (Cong)
    (para cada $f$ k-ário em $\Sigma$)

5.  **Instanciação (ou Especialização):** Se uma equação $t_L = t_R$ é um axioma em $E$ (ou já foi derivada), e $\sigma$ é uma substituição qualquer que mapeia as variáveis em $t_L$ e $t_R$ para termos (respeitando os sorts), então a equação instanciada $t_L\sigma = t_R\sigma$ também é derivável.
    $\frac{t_L = t_R \quad (\text{onde } t_L=t_R \text{ é um axioma ou teorema com variáveis } X)}{t_L\sigma = t_R\sigma \quad (\text{para qualquer substituição } \sigma: X \rightarrow T_\Sigma(Y))}$ (Inst)

Uma **derivação** (ou prova) de uma equação $e$ a partir de um conjunto de axiomas $E$ é uma sequência finita de equações $e_1, e_2, \ldots, e_p$ tal que $e_p = e$, e cada $e_i$ na sequência é:
*   Ou um axioma em $E$.
*   Ou é obtida a partir de equações anteriores na sequência pela aplicação de uma das regras de inferência (Reflexividade, Simetria, Transitividade, Congruência, Instanciação).

Se existe tal derivação, escrevemos $E \vdash t_L = t_R$, significando que $t_L = t_R$ é *dedutível* ou *provável* a partir de $E$. O Teorema da Completude de Birkhoff para a lógica equacional (que não provaremos aqui) estabelece que o sistema de dedução é *sólido* ($E \vdash e \implies E \models e$) e *completo* ($E \models e \implies E \vdash e$). Ou seja, uma equação é uma consequência lógica dos axiomas se, e somente se, ela pode ser provada usando essas regras de inferência.

**Exemplo de Derivação (informal):**
Vamos tentar provar o teorema `add(zero, n) = n` a partir dos axiomas N3: `add(x, zero) = x` e N4: `add(x, succ(y)) = succ(add(x, y))` da especificação `Natural`. (Isso requereria indução sobre `n`, que a lógica equacional pura não suporta diretamente sem axiomas de indução ou um sistema de prova mais rico. No entanto, podemos mostrar passos que usam as regras).

Se quiséssemos provar, por exemplo, `add(succ(zero), zero) = succ(zero)`:
1.  `add(n, zero) = n` (Axioma N3)
2.  Seja $\sigma = \{n \mapsto \text{succ(zero)}\}$.
3.  `add(succ(zero), zero) = succ(zero)` (Por Instanciação de 1 com $\sigma$). Esta é uma prova de 1 passo.

Provar `add(zero, n) = n` é mais complexo e geralmente requer um esquema de indução ou uma semântica de modelo inicial. No entanto, a lógica equacional permite derivar muitas igualdades úteis. Por exemplo, a comutatividade da adição (`add(n,m) = add(m,n)`) pode ser provada por indução usando os axiomas de Peano, e cada passo indutivo usaria as regras da lógica equacional.

O processo de usar os axiomas como regras de reescrita direcionais (e.g., sempre reescrever o lado esquerdo de um axioma para o lado direito) é uma forma de "computar" com especificações algébricas e será o tópico do Capítulo 3 (Sistemas de Reescrita de Termos).

**Exercício**
Dada a especificação `NaturalList` com os axiomas:
*   (NL5): `append(nil, l2) = l2`
*   (NL6): `append(cons(n, l1), l2) = cons(n, append(l1, l2))`
E assumindo que temos um axioma para `length` (como NL7, NL8) e para `add` (como N3, N4).
Mostre os passos principais (informalmente, indicando quais axiomas ou regras seriam usados) para argumentar que `length(append(cons(a, nil), cons(b, nil)))` é igual a `succ(succ(zero))` (ou seja, 2), onde `a,b` são `Natural`s e `zero, succ` são do TAD `Natural`.

**Resolução:**
Queremos mostrar que `length(append(cons(a, nil), cons(b, nil))) = succ(succ(zero))`.

Seja $L_1 = \text{cons}(a, \text{nil})$ e $L_2 = \text{cons}(b, \text{nil})$.
O termo é `length(append(L1, L2))`.

1.  **Aplicar Axioma NL6 a `append(L1, L2)`:**
    `append(cons(a, nil), L2) = cons(a, append(nil, L2))`
    (Usando NL6 com $n \mapsto a, l1 \mapsto \text{nil}, l2 \mapsto L_2$)

2.  **Aplicar Axioma NL5 a `append(nil, L2)` (o subtermo interno):**
    `append(nil, L2) = L2`
    (Usando NL5 com $l2 \mapsto L_2$)

3.  **Substituir o resultado de (2) em (1):**
    O termo original `append(L1, L2)` é igual a `cons(a, L2)`.
    Lembrando que $L_2 = \text{cons}(b, \text{nil})$, então `append(L1, L2) = cons(a, cons(b, nil))`.

4.  **Agora, calcular o `length` deste resultado:**
    O termo se torna `length(cons(a, cons(b, nil)))`.
    Aplicar Axioma NL8 (`length(cons(n, l)) = succ(length(l))`):
    `length(cons(a, cons(b, nil))) = succ(length(cons(b, nil)))`
    (Usando NL8 com $n \mapsto a, l \mapsto \text{cons}(b, \text{nil})$)

5.  **Aplicar Axioma NL8 novamente ao subtermo `length(cons(b, nil))`:**
    `length(cons(b, nil)) = succ(length(nil))`
    (Usando NL8 com $n \mapsto b, l \mapsto \text{nil}$)

6.  **Aplicar Axioma NL7 (`length(nil) = zero`):**
    `length(nil) = zero`

7.  **Substituir os resultados de volta:**
    De (6) em (5): `length(cons(b, nil)) = succ(zero)`
    De (este resultado) em (4): `length(cons(a, cons(b, nil))) = succ(succ(zero))`

Portanto, `length(append(cons(a, nil), cons(b, nil))) = succ(succ(zero))`.

Este processo usou principalmente instanciação de axiomas e substituição de termos iguais por termos iguais (que é uma consequência das regras de congruência e transitividade). Uma prova formal completa seria mais longa, listando cada aplicação de regra de inferência.
□

---
*O capítulo 2 está agora com uma boa base nas seções 2.1 e 2.2, aproximando-se de 4000 palavras. Continuarei com as seções 2.3, 2.4 e 2.5 para atingir a meta de 8000 palavras.*
---
(Continuando a expansão do Capítulo 2)

## 2.3 Semântica das Especificações Algébricas: $\Sigma$-Álgebras

Até agora, definimos uma especificação algébrica $SP = (\Sigma, E)$ como um par sintático: uma assinatura $\Sigma$ que nos dá os nomes dos sorts e das operações, e um conjunto $E$ de axiomas equacionais que restringem o comportamento dessas operações. Mas qual é o "significado" ou a "semântica" de tal especificação? A resposta reside no conceito de **$\Sigma$-álgebra**. Uma $\Sigma$-álgebra é uma estrutura matemática concreta que fornece uma interpretação para os sorts e as operações da assinatura $\Sigma$. A semântica de uma especificação $SP$ é então dada pela classe de todas as $\Sigma$-álgebras que não apenas fornecem uma interpretação para $\Sigma$, mas também *satisfazem* todos os axiomas em $E$.

Esta seção explorará a definição de uma $\Sigma$-álgebra, como as equações são satisfeitas em uma álgebra, o que significa para uma álgebra ser um modelo de uma especificação, e introduzirá brevemente os conceitos importantes de modelos iniciais e finais, que fornecem semânticas canônicas para especificações.

### 2.3.1 Definição de uma $\Sigma$-Álgebra: Domínios Portadores e Interpretação de Operações

Dada uma assinatura $\Sigma = (S, \text{Op})$, uma **$\Sigma$-álgebra** (ou simplesmente uma **álgebra sobre $\Sigma$**), denotada por $\mathcal{A}$, consiste em:

1.  **Uma família de conjuntos portadores (carrier sets):** Para cada nome de sort $s \in S$, a álgebra $\mathcal{A}$ atribui um conjunto não vazio, denotado por $s^{\mathcal{A}}$ (ou $A_s$, ou $\mathcal{A}_s$). Este conjunto $s^{\mathcal{A}}$ contém os "valores" concretos do tipo $s$ na interpretação fornecida por $\mathcal{A}$. Por exemplo, se $S = \{\text{Boolean}, \text{Natural}\}$, uma $\Sigma$-álgebra $\mathcal{A}$ teria um conjunto $\text{Boolean}^{\mathcal{A}}$ (e.g., $\{\texttt{true}_{\mathcal{A}}, \texttt{false}_{\mathcal{A}}\}$) e um conjunto $\text{Natural}^{\mathcal{A}}$ (e.g., $\{0_{\mathcal{A}}, 1_{\mathcal{A}}, 2_{\mathcal{A}}, \ldots\}$).

2.  **Uma família de funções (interpretação das operações):** Para cada símbolo de operação $f \in \text{Op}$ com perfil $f: s_1 \times s_2 \times \ldots \times s_k \rightarrow s_{k+1}$ (onde $s_1, \ldots, s_{k+1} \in S$), a álgebra $\mathcal{A}$ atribui uma função concreta (ou operação), denotada por $f^{\mathcal{A}}$ (ou $A_f$, ou $\mathcal{A}_f$). Esta função $f^{\mathcal{A}}$ deve ter o tipo correspondente nos conjuntos portadores:
    $f^{\mathcal{A}}: s_1^{\mathcal{A}} \times s_2^{\mathcal{A}} \times \ldots \times s_k^{\mathcal{A}} \rightarrow s_{k+1}^{\mathcal{A}}$
    *   Se $f$ é uma constante $c: \rightarrow s$, então $c^{\mathcal{A}}$ é um elemento específico (um valor) escolhido do conjunto portador $s^{\mathcal{A}}$. Ou seja, $c^{\mathcal{A}} \in s^{\mathcal{A}}$.

Em resumo, uma $\Sigma$-álgebra $\mathcal{A}$ dá uma interpretação concreta para todos os símbolos da assinatura $\Sigma$: mapeia cada sort para um conjunto e cada símbolo de operação para uma função sobre esses conjuntos que respeita os perfis.

**Exemplo de uma $\Sigma_{Bool}$-Álgebra:**
Considere a assinatura $\Sigma_{Bool}$ (da Seção 2.2.2) com sort `Boolean` e operações `true`, `false`, `not`, `and`, `or`.
Uma possível $\Sigma_{Bool}$-álgebra, chamemos de $\mathcal{B}_{std}$ (a álgebra booleana padrão), seria:
*   Conjunto portador para `Boolean`: $\text{Boolean}^{\mathcal{B}_{std}} = \{ \mathbf{T}, \mathbf{F} \}$ (onde $\mathbf{T}$ representa "verdadeiro" e $\mathbf{F}$ representa "falso").
*   Interpretação das operações:
    *   $\text{true}^{\mathcal{B}_{std}} = \mathbf{T}$ (a constante `true` é interpretada como o valor $\mathbf{T}$)
    *   $\text{false}^{\mathcal{B}_{std}} = \mathbf{F}$
    *   $\text{not}^{\mathcal{B}_{std}}: \{\mathbf{T}, \mathbf{F}\} \rightarrow \{\mathbf{T}, \mathbf{F}\}$ definida por:
        $\text{not}^{\mathcal{B}_{std}}(\mathbf{T}) = \mathbf{F}$
        $\text{not}^{\mathcal{B}_{std}}(\mathbf{F}) = \mathbf{T}$
    *   $\text{and}^{\mathcal{B}_{std}}: \{\mathbf{T}, \mathbf{F}\} \times \{\mathbf{T}, \mathbf{F}\} \rightarrow \{\mathbf{T}, \mathbf{F}\}$ definida pela tabela verdade usual da conjunção:
        $\text{and}^{\mathcal{B}_{std}}(\mathbf{T}, \mathbf{T}) = \mathbf{T}$
        $\text{and}^{\mathcal{B}_{std}}(\mathbf{T}, \mathbf{F}) = \mathbf{F}$
        $\text{and}^{\mathcal{B}_{std}}(\mathbf{F}, \mathbf{T}) = \mathbf{F}$
        $\text{and}^{\mathcal{B}_{std}}(\mathbf{F}, \mathbf{F}) = \mathbf{F}$
    *   $\text{or}^{\mathcal{B}_{std}}$ seria definida similarmente pela tabela verdade da disjunção.

Esta álgebra $\mathcal{B}_{std}$ é a interpretação "intencionada" da assinatura booleana. No entanto, existem muitas outras $\Sigma_{Bool}$-álgebras possíveis. Por exemplo, poderíamos ter uma álgebra "degenerada" $\mathcal{B}_{deg}$ onde $\text{Boolean}^{\mathcal{B}_{deg}} = \{ \text{unico_valor} \}$ e todas as operações retornam $\text{unico_valor}$. Tal álgebra ainda é uma $\Sigma_{Bool}$-álgebra (pois respeita os perfis), mas provavelmente não satisfará os axiomas que esperamos para booleanos (e.g., `not(true) = false`).

**Avaliação de Termos em uma $\Sigma$-Álgebra:**
Uma vez que temos uma $\Sigma$-álgebra $\mathcal{A}$, podemos dar significado (avaliar) qualquer termo fundamental $t \in T_{\Sigma,s}$ (um termo do sort $s$ sem variáveis) nesta álgebra. A avaliação de $t$ em $\mathcal{A}$, denotada $t^{\mathcal{A}}$ (ou $\text{eval}_{\mathcal{A}}(t)$), resulta em um valor no conjunto portador $s^{\mathcal{A}}$. A avaliação é definida indutivamente:
1.  Se $t$ é uma constante $c: \rightarrow s$, então $t^{\mathcal{A}} = c^{\mathcal{A}} \in s^{\mathcal{A}}$.
2.  Se $t$ é da forma $f(t_1, \ldots, t_k)$ com $f: s_1 \times \ldots \times s_k \rightarrow s$, então $t^{\mathcal{A}} = f^{\mathcal{A}}(t_1^{\mathcal{A}}, \ldots, t_k^{\mathcal{A}})$. O resultado é obtido aplicando a função $f^{\mathcal{A}}$ aos valores resultantes da avaliação dos subtermos $t_1, \ldots, t_k$ em $\mathcal{A}$.

Por exemplo, na álgebra $\mathcal{B}_{std}$, o termo fundamental `and(not(true), false)` seria avaliado como:
$(\text{and}(\text{not(true)}, \text{false}))^{\mathcal{B}_{std}}$
$= \text{and}^{\mathcal{B}_{std}}((\text{not(true)})^{\mathcal{B}_{std}}, \text{false}^{\mathcal{B}_{std}})$
$= \text{and}^{\mathcal{B}_{std}}(\text{not}^{\mathcal{B}_{std}}(\text{true}^{\mathcal{B}_{std}}), \mathbf{F})$
$= \text{and}^{\mathcal{B}_{std}}(\text{not}^{\mathcal{B}_{std}}(\mathbf{T}), \mathbf{F})$
$= \text{and}^{\mathcal{B}_{std}}(\mathbf{F}, \mathbf{F})$
$= \mathbf{F}$

Se um termo $t$ contém variáveis $X = \{x_1:s_1, \ldots, x_k:s_k\}$, sua avaliação em $\mathcal{A}$ depende de uma **atribuição de valores** (ou valoração) $\alpha: X \rightarrow \bigcup_{s \in S} s^{\mathcal{A}}$ que mapeia cada variável $x_i$ para um valor $\alpha(x_i)$ no conjunto portador correspondente $s_i^{\mathcal{A}}$. Dada uma atribuição $\alpha$, a avaliação $t^{\mathcal{A}}[\alpha]$ é definida similarmente, onde se $t$ é uma variável $x$, $t^{\mathcal{A}}[\alpha] = \alpha(x)$.

### 2.3.2 Satisfação de Equações em uma Álgebra. Modelos de uma Especificação

Agora podemos definir o que significa para uma $\Sigma$-álgebra $\mathcal{A}$ **satisfazer** uma $\Sigma$-equação $t_L = t_R$ (onde $t_L, t_R$ são termos do mesmo sort, possivelmente com variáveis $X$).

A álgebra $\mathcal{A}$ **satisfaz** a equação $t_L = t_R$ (denotado $\mathcal{A} \models t_L = t_R$) se, para **toda** possível atribuição de valores $\alpha$ das variáveis $X$ em $t_L, t_R$ para elementos dos conjuntos portadores correspondentes de $\mathcal{A}$, a avaliação de $t_L$ em $\mathcal{A}$ sob $\alpha$ é igual à avaliação de $t_R$ em $\mathcal{A}$ sob $\alpha$. Ou seja:
$\mathcal{A} \models t_L = t_R \quad \text{iff} \quad \forall \alpha: X \rightarrow \mathcal{A}, \quad t_L^{\mathcal{A}}[\alpha] = t_R^{\mathcal{A}}[\alpha]$.

Se a equação não contém variáveis (i.e., $t_L, t_R$ são termos fundamentais), então ela é satisfeita por $\mathcal{A}$ se $t_L^{\mathcal{A}} = t_R^{\mathcal{A}}$.

Agora, podemos definir um **modelo** de uma especificação algébrica:
Dada uma especificação algébrica $SP = (\Sigma, E)$, uma $\Sigma$-álgebra $\mathcal{A}$ é chamada de **modelo** de $SP$ se $\mathcal{A}$ satisfaz **todos** os axiomas em $E$.
Ou seja, $\mathcal{A}$ é um modelo de $SP \iff \forall (t_L = t_R) \in E, \mathcal{A} \models t_L = t_R$.

**Exemplo de Satisfação e Modelo:**
Considere a assinatura $\Sigma_{Bool}$ e a álgebra $\mathcal{B}_{std}$ definidas acima.
Seja o axioma (Axioma B1 de $E_{Bool}$): `not(true) = false`.
Para verificar se $\mathcal{B}_{std} \models \text{not(true) = false}$:
*   $(\text{not(true)})^{\mathcal{B}_{std}} = \text{not}^{\mathcal{B}_{std}}(\text{true}^{\mathcal{B}_{std}}) = \text{not}^{\mathcal{B}_{std}}(\mathbf{T}) = \mathbf{F}$.
*   $(\text{false})^{\mathcal{B}_{std}} = \mathbf{F}$.
Como $\mathbf{F} = \mathbf{F}$, a álgebra $\mathcal{B}_{std}$ satisfaz este axioma.

Seja o axioma (Axioma B3 de $E_{Bool}$): `and(true, b) = b` (onde `b` é uma variável do sort `Boolean`).
Para verificar se $\mathcal{B}_{std} \models \text{and(true, b) = b}$:
Precisamos verificar para todas as atribuições de `b` aos valores em $\text{Boolean}^{\mathcal{B}_{std}} = \{\mathbf{T}, \mathbf{F}\}$.
1.  Atribuição $\alpha_1 = \{b \mapsto \mathbf{T}\}$:
    *   $(\text{and(true, b)})^{\mathcal{B}_{std}}[\alpha_1] = \text{and}^{\mathcal{B}_{std}}(\text{true}^{\mathcal{B}_{std}}, \alpha_1(b)) = \text{and}^{\mathcal{B}_{std}}(\mathbf{T}, \mathbf{T}) = \mathbf{T}$.
    *   $(b)^{\mathcal{B}_{std}}[\alpha_1] = \alpha_1(b) = \mathbf{T}$.
    Como $\mathbf{T} = \mathbf{T}$, a equação vale para esta atribuição.
2.  Atribuição $\alpha_2 = \{b \mapsto \mathbf{F}\}$:
    *   $(\text{and(true, b)})^{\mathcal{B}_{std}}[\alpha_2] = \text{and}^{\mathcal{B}_{std}}(\text{true}^{\mathcal{B}_{std}}, \alpha_2(b)) = \text{and}^{\mathcal{B}_{std}}(\mathbf{T}, \mathbf{F}) = \mathbf{F}$.
    *   $(b)^{\mathcal{B}_{std}}[\alpha_2] = \alpha_2(b) = \mathbf{F}$.
    Como $\mathbf{F} = \mathbf{F}$, a equação vale para esta atribuição.
Como a equação vale para todas as atribuições possíveis da variável `b`, a álgebra $\mathcal{B}_{std}$ satisfaz o axioma `and(true, b) = b`.

Se $\mathcal{B}_{std}$ satisfizer todos os axiomas em $E_{Bool}$ (o que ela faz, pela definição usual de lógica booleana), então $\mathcal{B}_{std}$ é um modelo da especificação $SP_{Bool}$.

A álgebra "degenerada" $\mathcal{B}_{deg}$ mencionada antes, com $\text{Boolean}^{\mathcal{B}_{deg}} = \{ \text{unico_valor} \}$, onde, por exemplo, $\text{true}^{\mathcal{B}_{deg}} = \text{unico_valor}$ e $\text{false}^{\mathcal{B}_{deg}} = \text{unico_valor}$, não seria um modelo de $SP_{Bool}$, pois o axioma `not(true) = false` implicaria $\text{not}^{\mathcal{B}_{deg}}(\text{unico_valor}) = \text{unico_valor}$, mas se `true` e `false` são o mesmo valor, a distinção pretendida se perde. Mais crucialmente, se `true = false` é derivável nesta álgebra, e os axiomas implicam `true != false` na semântica intencionada, então esta álgebra não é um modelo "interessante". Um dos objetivos da especificação é excluir modelos indesejados.

### 2.3.3 A Classe de Todas as $\Sigma$-Álgebras que Satisfazem E (Alg(SP)). Modelos Iniciais e Finais.

Para uma dada especificação $SP = (\Sigma, E)$, a classe de todas as $\Sigma$-álgebras que são modelos de $SP$ é denotada por $\mathbf{Alg}(SP)$.
$\mathbf{Alg}(SP) = \{ \mathcal{A} \mid \mathcal{A} \text{ é uma } \Sigma\text{-álgebra e } \mathcal{A} \models E \}$

Esta classe $\mathbf{Alg}(SP)$ representa a **semântica de modelo-teórica** da especificação $SP$. Qualquer álgebra nesta classe é uma instanciação concreta válida do Tipo Abstrato de Dados definido por $SP$.

No entanto, a classe $\mathbf{Alg}(SP)$ pode ser muito grande e conter modelos que são bastante diferentes entre si, incluindo alguns que podem não capturar totalmente a "intenção" do especificador. Por exemplo, para o TAD `Natural` com `zero` e `succ`, além do modelo padrão dos números naturais $\{0, 1, 2, \ldots\}$, poderíamos ter modelos que incluem "lixo" (elementos não alcançáveis a partir de `zero` por aplicações de `succ`) ou modelos que identificam termos que gostaríamos de manter distintos (e.g., um modelo onde `succ(succ(zero)) = zero`, como aritmética módulo 2, se os axiomas não forem fortes o suficiente para proibir isso).

Para refinar a semântica e selecionar modelos "canônicos" ou "mais representativos", a teoria da especificação algébrica introduz os conceitos de **álgebra inicial** e **álgebra final** (entre outros tipos de modelos). Estes são modelos dentro de $\mathbf{Alg}(SP)$ que possuem propriedades universais especiais (definidas em termos de homomorfismos entre álgebras, um conceito da teoria de categorias que veremos brevemente na Parte V e que não detalharemos aqui).

*   **Álgebra Inicial ($\mathcal{I}$):**
    Intuitivamente, a álgebra inicial de uma especificação $SP$ é o modelo "menos confuso" ou "mais abstrato" que satisfaz os axiomas.
    *   **"No junk":** Todos os elementos em seus conjuntos portadores são representados por (ou são identificáveis com) termos fundamentais da assinatura $\Sigma$. Não há "lixo" ou elementos espúrios que não podem ser construídos pelas operações da assinatura.
    *   **"No confusion":** Dois termos fundamentais $t_1, t_2$ são interpretados como o mesmo elemento na álgebra inicial se, e somente se, sua igualdade $t_1 = t_2$ é uma consequência lógica dos axiomas $E$ (i.e., $E \models t_1 = t_2$). Ela não introduz nenhuma igualdade adicional além daquelas que são estritamente exigidas pelos axiomas.
    A álgebra inicial é frequentemente construída como a **álgebra de termos fundamentais módulo as equações deriváveis de E** (a álgebra $T_{\Sigma} / \equiv_E$, onde $\equiv_E$ é a relação de congruência induzida por $E$).
    A semântica inicial é muito popular porque parece capturar a ideia de que os valores de um TAD são exatamente aqueles que podem ser construídos e distinguidos usando as operações e axiomas fornecidos, nada mais, nada menos.

*   **Álgebra Final ($\mathcal{Z}$):**
    Intuitivamente, a álgebra final de uma especificação $SP$ é o modelo "mais confuso" (consistente com os axiomas) ou "mais concreto" no sentido de que ela identifica o máximo possível de termos sem violar as inequalidades impostas (implícita ou explicitamente) pelos axiomas. Ela maximiza as igualdades.
    *   Na semântica final, dois termos são considerados iguais se eles não puderem ser distinguidos por nenhuma "observação" permitida pelas operações da especificação (especialmente operações que retornam valores em tipos de dados "observáveis" já bem definidos, como `Boolean`).
    A semântica final é útil para modelar tipos de dados com comportamento de estado ou quando se quer abstrair representações internas (ocultação de informação).

A existência e unicidade (a menos de isomorfismo) das álgebras inicial e final dependem de certas propriedades da especificação (e.g., para a inicial, a consistência dos axiomas; para ambas, certas formas dos axiomas).

Para muitas especificações de TADs comuns, como `Boolean`, `Natural` (Peano), `List`, a semântica inicial é a mais natural e corresponde à nossa intuição. Por exemplo, a álgebra inicial da especificação `Natural` (com `zero`, `succ` e axiomas de Peano apropriados) é isomorfa aos números naturais padrão $\{0, 1, 2, \ldots\}$, onde cada número é distinto e não há "ciclos" ou elementos extras.

A discussão detalhada sobre homomorfismos, categorias de álgebras, e as construções formais de álgebras iniciais e finais está além do escopo desta seção introdutória, mas é importante estar ciente de que a semântica de uma especificação algébrica pode ser definida de maneiras mais precisas do que apenas "qualquer álgebra que satisfaça os axiomas". Para a maior parte deste livro, a compreensão intuitiva da semântica inicial (termos fundamentais módulo as equações) será suficiente. Os Capítulos 3 e 4, ao discutirem reescrita de termos e propriedades como consistência e completeza, revisitarão implicitamente aspectos da semântica inicial.

**Exercício**
Considere uma assinatura $\Sigma_{SimpleNat}$ com sort `SNat` e operações `z: --> SNat` e `s: SNat --> SNat`.
Considere dois conjuntos de axiomas:
1.  $E_1 = \emptyset$ (nenhum axioma)
2.  $E_2 = \{ s(s(s(x))) = x \}$ (onde `x` é uma variável do sort `SNat`)

Descreva intuitivamente como seria a álgebra inicial para $SP_1 = (\Sigma_{SimpleNat}, E_1)$ e para $SP_2 = (\Sigma_{SimpleNat}, E_2)$.

**Resolução:**
Assinatura $\Sigma_{SimpleNat}$: sort `SNat`, operações `z: --> SNat`, `s: SNat --> SNat`.

1.  **Especificação $SP_1 = (\Sigma_{SimpleNat}, E_1)$ com $E_1 = \emptyset$ (sem axiomas):**
    *   **Álgebra Inicial $\mathcal{I}_1$:**
        Na ausência de axiomas, não há equações que forcem quaisquer termos fundamentais a serem iguais, exceto por igualdade trivial (um termo é igual a si mesmo).
        Os termos fundamentais são: `z`, `s(z)`, `s(s(z))`, `s(s(s(z)))`, ...
        A álgebra inicial $\mathcal{I}_1$ terá como conjunto portador $\text{SNat}^{\mathcal{I}_1}$ um conjunto isomorfo ao conjunto dos números naturais $\{0, 1, 2, 3, \ldots\}$.
        *   $z^{\mathcal{I}_1}$ corresponderia a $0$.
        *   $s^{\mathcal{I}_1}$ seria a função sucessor (adicionar 1).
        *   $(s(z))^{\mathcal{I}_1}$ corresponderia a $1$.
        *   $(s(s(z)))^{\mathcal{I}_1}$ corresponderia a $2$.
        E assim por diante. Todos os termos da forma $s^k(z)$ (aplicação de $s$, $k$ vezes a $z$) são distintos na álgebra inicial. É o modelo "mais livre" ou "mais abstrato" dos números naturais de Peano (sem outras operações ou axiomas sobre elas). Ela tem a propriedade "no junk" (todos os elementos são construíveis) e "no confusion" (termos só são iguais se forem sintaticamente idênticos, já que não há axiomas para impor outras igualdades).

2.  **Especificação $SP_2 = (\Sigma_{SimpleNat}, E_2)$ com $E_2 = \{ s(s(s(x))) = x \}$:**
    *   **Álgebra Inicial $\mathcal{I}_2$:**
        O axioma $s(s(s(x))) = x$ impõe uma restrição cíclica. Ele significa que aplicar o sucessor três vezes a qualquer elemento $x$ retorna ao próprio $x$.
        Vamos ver o que acontece com os termos fundamentais:
        *   `z`
        *   `s(z)`
        *   `s(s(z))`
        *   `s(s(s(z)))`: Pelo axioma (instanciando $x \mapsto z$), este é igual a `z`.
        *   `s(s(s(s(z))))`: Isto é `s(z)` (pois `s(s(s(z)))` é `z`, então `s` disso é `s(z)`).
        *   `s(s(s(s(s(z)))))`: Isto é `s(s(z))`.
        *   `s(s(s(s(s(s(z))))))`: Isto é `s(s(s(z)))` que é `z`.
        A álgebra inicial $\mathcal{I}_2$ terá um conjunto portador $\text{SNat}^{\mathcal{I}_2}$ com apenas três elementos distintos, que podemos identificar com os termos fundamentais (módulo a equivalência imposta pelo axioma):
        $\text{SNat}^{\mathcal{I}_2} = \{ [z], [s(z)], [s(s(z))] \}$
        onde $[t]$ denota a classe de equivalência do termo $t$.
        *   $z^{\mathcal{I}_2}$ corresponderia a $[z]$.
        *   $s^{\mathcal{I}_2}$ seria uma função que cicla através desses três elementos:
            *   $s^{\mathcal{I}_2}([z]) = [s(z)]$
            *   $s^{\mathcal{I}_2}([s(z)]) = [s(s(z))]$
            *   $s^{\mathcal{I}_2}([s(s(z))]) = [s(s(s(z)))] = [z]$ (devido ao axioma)
        Esta álgebra é isomorfa à aritmética módulo 3 (os inteiros $\{0, 1, 2\}$ com a operação de adicionar 1 módulo 3). O axioma "confundiu" (identificou) termos que eram distintos na álgebra inicial de $SP_1$.
□

---
*O capítulo 2 está agora com mais de 6000 palavras. Vou continuar com as seções 2.4 e 2.5 para adicionar mais conteúdo e atingir a meta de 8000 palavras.*
---
(Continuando a expansão do Capítulo 2)

## 2.4 Construtores, Observadores e Operações Derivadas

Dentro da assinatura de um Tipo Abstrato de Dados, as operações podem desempenhar papéis diferentes em relação à forma como os valores do TAD são gerados e inspecionados. Uma distinção conceitual importante, embora nem sempre formalmente exigida pela teoria básica da especificação algébrica (mas crucial para certas análises como a de completeza suficiente), é entre **construtores**, **observadores** e, por vezes, **operações derivadas**.

1.  **Construtores (Constructors):**
    Os construtores de um sort $s$ são um subconjunto das operações da assinatura (cujo contradomínio pode ser $s$ ou outros sorts usados para construir $s$) que são suficientes para gerar *todos* os valores possíveis do sort $s$. Mais formalmente, na semântica inicial, todo termo fundamental de sort $s$ que representa um valor distinto pode ser construído (ou é igual, módulo os axiomas) a um termo formado exclusivamente por aplicações de operações construtoras.
    *   **Exemplo para `Natural` (com `zero`, `succ`):**
        As operações `zero: --> Natural` e `succ: Natural --> Natural` são tipicamente consideradas os construtores do sort `Natural`. Qualquer número natural pode ser representado por um termo como `zero`, `succ(zero)`, `succ(succ(zero))`, etc., usando apenas esses dois construtores.
    *   **Exemplo para `NaturalList` (com `nil`, `cons`):**
        As operações `nil: --> NaturalList` e `cons: Natural x NaturalList --> NaturalList` são os construtores canônicos para listas. Qualquer lista finita de naturais pode ser construída por uma sequência de aplicações de `cons` terminando em `nil`, e.g., `cons(n1, cons(n2, nil))`.
    *   **Características:**
        *   **Geradores de Valores:** Seu propósito principal é criar instâncias do tipo.
        *   **Base para Indução Estrutural:** Os construtores formam a base para definir funções recursivas sobre o tipo e para provar propriedades por indução estrutural sobre os termos do tipo. Por exemplo, para provar uma propriedade $P(l)$ para todas as `NaturalList`s `l`, provamos $P(\text{nil})$ (caso base) e provamos que se $P(l')$ vale, então $P(\text{cons}(n, l'))$ também vale (passo indutivo).
        *   **"Free-ness" (Liberdade):** Em uma especificação "bem comportada" com um conjunto de construtores $C$, espera-se frequentemente que termos diferentes formados apenas por construtores representem valores diferentes, a menos que os axiomas imponham igualdades específicas entre eles (e.g., axiomas de comutatividade para construtores de um tipo `Set`). Isso é relacionado à propriedade "no confusion" da álgebra inicial.

    A escolha de quais operações designar como construtores é uma parte importante do design da especificação. Idealmente, o conjunto de construtores deve ser mínimo e completo para gerar todos os valores "distintos" do tipo.

2.  **Observadores (Observers ou Accessors):**
    Os observadores são operações que retornam informações sobre os valores de um TAD, geralmente mapeando valores do sort principal para valores de outros sorts (frequentemente sorts mais "primitivos" ou já bem compreendidos, como `Boolean` ou `Natural`). Eles não modificam o valor do TAD que estão observando (no sentido de que, se o TAD fosse um objeto mutável, eles não teriam efeitos colaterais; em especificações algébricas puras, todas as operações são funcionais e não têm efeitos colaterais de qualquer forma).
    *   **Exemplo para `Natural`:**
        `isZero: Natural --> Boolean` é um observador. Ele "observa" um `Natural` e retorna um `Boolean`.
        `isEqual: Natural x Natural --> Boolean` observa dois `Natural`s.
    *   **Exemplo para `NaturalList`:**
        `isEmpty: NaturalList --> Boolean` é um observador.
        `head: NaturalList --> Natural` é um observador (retorna o primeiro elemento, um `Natural`).
        `tail: NaturalList --> NaturalList` pode ser visto como um observador que retorna uma parte da lista original, mas seu contradomínio é o mesmo sort `NaturalList`. Às vezes, operações como `tail` que decompõem uma estrutura são chamadas de *seletores* ou *decompositores*, e estão intimamente ligadas aos construtores (e.g., `head` e `tail` são os seletores para o construtor `cons`).
        `length: NaturalList --> Natural` é um observador.
    *   **Características:**
        *   **Extração de Informação:** Seu propósito é extrair propriedades ou componentes de um valor do TAD.
        *   **Definição em Termos de Construtores:** O comportamento dos observadores é tipicamente definido por axiomas que especificam como eles atuam sobre cada um dos construtores do TAD. Por exemplo, `isEmpty(nil) = true` e `isEmpty(cons(n,l)) = false` definem `isEmpty` para ambos os construtores de `NaturalList`. Esta é a base para a análise de *completeza suficiente* (ver Capítulo 4).

3.  **Operações Derivadas (ou Modificadores, Transformadores – em um sentido funcional):**
    Estas são operações que tomam um ou mais valores do TAD (e possivelmente de outros sorts) e produzem um novo valor do TAD principal, mas não são consideradas construtores primários. Elas podem frequentemente ser definidas em termos dos construtores e observadores, ou de outras operações derivadas.
    *   **Exemplo para `Natural`:**
        `add: Natural x Natural --> Natural` é uma operação derivada (ou um "modificador funcional"). Ela pode ser definida recursivamente usando `zero` e `succ` (os construtores), como vimos nos axiomas N3 e N4.
        `multiply: Natural x Natural --> Natural` também é derivada, definida em termos de `add`, `zero` e `succ`.
    *   **Exemplo para `NaturalList`:**
        `append: NaturalList x NaturalList --> NaturalList` é uma operação derivada, definida recursivamente usando `nil`, `cons` e `append` (axiomas NL5 e NL6).
        Uma operação `reverse: NaturalList --> NaturalList` também seria uma operação derivada.
    *   **Características:**
        *   **Computação de Novos Valores:** Produzem novos valores do TAD a partir de existentes, mas não são essenciais para *gerar* todos os valores básicos do tipo (os construtores já fazem isso).
        *   **Definição Axiomática:** Seus axiomas geralmente mostram como elas podem ser "reduzidas" ou definidas em termos de aplicações a formas mais simples (construídas com construtores) ou em termos de outras operações.

A distinção entre construtores, observadores e operações derivadas não é sempre rígida ou única; às vezes uma operação pode ter características de mais de uma categoria, ou a classificação pode depender da perspectiva do especificador. No entanto, pensar nesses papéis é muito útil durante o processo de design de uma especificação:
*   **Comece pelos Construtores:** Identifique o conjunto mínimo de operações necessárias para construir todas as instâncias possíveis do tipo de dados.
*   **Defina Observadores:** Adicione operações que permitem inspecionar ou extrair informações relevantes dos valores construídos. Defina o comportamento dos observadores para cada construtor.
*   **Adicione Operações Derivadas:** Introduza outras operações úteis e defina seu comportamento, idealmente em termos dos construtores, observadores ou de forma recursiva.

Essa abordagem estruturada ajuda a garantir que a especificação seja bem organizada e que as propriedades como completeza (todas as operações estão definidas para todas as entradas relevantes) possam ser analisadas mais facilmente. No Capítulo 4, quando discutirmos a *completeza suficiente*, a distinção entre construtores e outras operações (que precisam ser definidas para todos os padrões de construtores) será formalmente importante.

Por exemplo, na especificação de `Natural` com `zero` e `succ` como construtores, a operação `add` é definida por axiomas que mostram como `add(n, m)` se comporta quando `m` é `zero` (Axioma N3: `add(n, zero) = n`) e quando `m` é `succ(m')` (Axioma N4: `add(n, succ(m')) = succ(add(n, m'))`). Esses dois axiomas cobrem todos os casos possíveis para o segundo argumento de `add`, dado que qualquer natural é ou `zero` ou o sucessor de algum outro natural. Isso é um exemplo de como uma operação derivada (`add`) é definida em relação aos construtores (`zero`, `succ`).

**Exercício**
Para o TAD `NaturalList` (com construtores `nil` e `cons`), considere uma operação `sum: NaturalList --> Natural` que calcula a soma de todos os números naturais em uma lista.
a) Classifique `sum` como construtor, observador ou operação derivada. Justifique.
b) Escreva os axiomas equacionais que definiriam o comportamento de `sum` em termos dos construtores `nil` e `cons`, e da operação `add` do TAD `Natural`.

**Resolução:**
a) **Classificação de `sum: NaturalList --> Natural`:**
   A operação `sum` é um **observador** ou, mais especificamente, uma **operação derivada com característica de observação**.
   *   **Não é um construtor** de `NaturalList` porque seu contradomínio é `Natural`, não `NaturalList`. Ela não cria listas.
   *   **É um observador** porque "observa" uma `NaturalList` e retorna uma propriedade dela (a soma de seus elementos) em um sort diferente (`Natural`). Ela extrai informação da lista.
   *   Pode também ser vista como uma **operação derivada** porque seu comportamento pode ser completamente definido em termos dos construtores da lista (`nil`, `cons`) e de uma operação do tipo dos elementos (`add` para `Natural`). Ela não introduz nenhuma nova forma fundamental de valor de lista.

b) **Axiomas Equacionais para `sum`:**
   Precisamos definir o comportamento de `sum` para cada construtor de `NaturalList`: `nil` e `cons(n, l)`.
   (Assumindo que `add: Natural x Natural --> Natural` e `zero: --> Natural` são do TAD `Natural`)

   *   (Axiom SUM_NIL): `sum(nil) = zero`
   *   (Axiom SUM_CONS): `sum(cons(n, l)) = add(n, sum(l))` (para todo `n: Natural`, `l: NaturalList`)

   **Explicação dos Axiomas:**
   *   (Axiom SUM_NIL): O caso base. A soma dos elementos de uma lista vazia (`nil`) é `zero` (o elemento neutro da adição de naturais).
   *   (Axiom SUM_CONS): O passo recursivo. A soma dos elementos de uma lista não vazia `cons(n, l)` (que tem `n` como primeiro elemento e `l` como o resto da lista) é obtida somando o primeiro elemento `n` com a soma dos elementos do restante da lista, `sum(l)`. A operação `add` do TAD `Natural` é usada para esta soma.

Estes dois axiomas definem completamente a operação `sum` para todas as possíveis `NaturalList`s construídas a partir de `nil` e `cons`.
□

## 2.5 Exemplos Fundamentais de Especificações Algébricas

Para consolidar os conceitos de sorts, operações, assinaturas e axiomas, e para ilustrar como especificações algébricas completas são construídas, apresentaremos agora as especificações formais para alguns Tipos Abstratos de Dados fundamentais: `Boolean`, `Natural` (baseado nos axiomas de Peano) e `Integer`. Estes TADs servem não apenas como exemplos didáticos, mas também como blocos de construção essenciais que serão frequentemente utilizados ou pressupostos em especificações de TADs mais complexos ao longo do livro.

Observaremos o formato `SPEC ADT ... END SPEC` introduzido, lembrando que ele encapsula uma assinatura $\Sigma = (S, \text{Op})$ e um conjunto de axiomas $E$. As variáveis nos axiomas são implicitamente quantificadas universalmente sobre seus respectivos sorts, a menos que indicado de outra forma (embora a cláusula `for all` torne isso explícito).

### 2.5.1 Especificação de Boolean

O tipo de dados `Boolean` é fundamental em qualquer sistema lógico ou computacional, representando os valores de verdade "verdadeiro" e "falso" e as operações lógicas básicas.

**SPEC ADT** Boolean

**sorts:**
*   Boolean

**operations:**
*   true: --> Boolean
*   false: --> Boolean
*   not: Boolean --> Boolean
*   and: Boolean x Boolean --> Boolean
*   or: Boolean x Boolean --> Boolean
*   implies: Boolean x Boolean --> Boolean
*   isEqualB: Boolean x Boolean --> Boolean

**axioms:**
*   (Axiom B1): not(true) = false
*   (Axiom B2): not(false) = true
*   (Axiom B3): and(true, b) = b
*   (Axiom B4): and(false, b) = false
*   (Axiom B5): or(true, b) = true
*   (Axiom B6): or(false, b) = b
*   (Axiom B7): implies(b1, b2) = or(not(b1), b2)
*   (Axiom B8): isEqualB(b1, b1) = true
*   (Axiom B9): isEqualB(true, false) = false
*   (Axiom B10): isEqualB(false, true) = false

for all b, b1, b2: Boolean

**END SPEC**

Esta especificação define o TAD `Boolean`.
*   **Sorts:** Há um único sort, `Boolean`.
*   **Operações:**
    *   `true` e `false` são constantes (construtores) do sort `Boolean`. Elas representam os dois únicos valores distintos deste tipo na semântica intencionada (inicial).
    *   `not` é a negação unária.
    *   `and` é a conjunção lógica (E).
    *   `or` é a disjunção lógica (OU inclusivo).
    *   `implies` é a implicação lógica.
    *   `isEqualB` é a operação de igualdade para booleanos (nomeada `isEqualB` para evitar conflito com uma possível operação de igualdade genérica ou polimórfica, que não estamos tratando ainda).
*   **Axiomas:**
    *   (B1) e (B2) definem o comportamento de `not` em relação aos construtores `true` e `false`.
    *   (B3) e (B4) definem `and` quando o primeiro argumento é `true` ou `false`. Para uma definição completa de `and` (que também seria comutativa), precisaríamos de axiomas para `and(b, true)` e `and(b, false)`, ou um axioma de comutatividade como `and(b1,b2) = and(b2,b1)`. A especificação aqui é suficiente para definir o resultado de `and(x,y)` por casos sobre `x`.
    *   (B5) e (B6) definem `or` similarmente, por casos sobre o primeiro argumento.
    *   (B7) define `implies` em termos de `or` e `not`, que é uma definição padrão da implicação. Isso mostra como uma operação pode ser especificada em termos de outras já definidas.
    *   (B8), (B9), e (B10) definem a igualdade booleana. `isEqualB(b1,b1) = true` (reflexividade), e os outros dois axiomas cobrem os casos de desigualdade. A partir destes, `isEqualB(true,true) = true` e `isEqualB(false,false) = true` podem ser inferidos.

Este conjunto de axiomas é suficiente para determinar o resultado de qualquer expressão booleana fundamental (sem variáveis). Por exemplo, `or(not(true), false)` seria igual a `or(false, false)` por (B1), que por sua vez é igual a `false` por (B6). A especificação também é *consistente* (não permite derivar `true = false`) e *completa* no sentido de que todas as operações estão definidas para todas as combinações dos construtores `true` e `false` (este é um aspecto de completeza suficiente que veremos no Capítulo 4).

### 2.5.2 Especificação de Natural (Números Naturais segundo Peano)

O tipo de dados dos números naturais ($0, 1, 2, \ldots$) é outro pilar da matemática e da computação. A especificação algébrica a seguir é baseada na construção de Peano, usando `zero` e `succ` (sucessor) como construtores.

**SPEC ADT** Natural

**sorts:**
*   Natural
*   Boolean

**operations:**
*   zero: --> Natural
*   succ: Natural --> Natural
*   isZero: Natural --> Boolean
*   add: Natural x Natural --> Natural
*   multiply: Natural x Natural --> Natural
*   isEqualN: Natural x Natural --> Boolean
*   pred: Natural --> Natural

**axioms:**
*   (Axiom N1): isZero(zero) = true
*   (Axiom N2): isZero(succ(n)) = false
*   (Axiom N3): add(n, zero) = n
*   (Axiom N4): add(n, succ(m)) = succ(add(n, m))
*   (Axiom N5): multiply(n, zero) = zero
*   (Axiom N6): multiply(n, succ(m)) = add(multiply(n, m), n)
*   (Axiom N7): isEqualN(zero, zero) = true
*   (Axiom N8): isEqualN(succ(n), zero) = false
*   (Axiom N9): isEqualN(zero, succ(m)) = false
*   (Axiom N10): isEqualN(succ(n), succ(m)) = isEqualN(n, m)
*   (Axiom N11): pred(zero) = zero
*   (Axiom N12): pred(succ(n)) = n

for all n, m: Natural

**END SPEC**

Esta especificação define o TAD `Natural`.
*   **Sorts:** `Natural` (o tipo principal) e `Boolean` (para resultados de `isZero` e `isEqualN`).
*   **Operações:**
    *   `zero: --> Natural` e `succ: Natural --> Natural` são os construtores. Todo natural é ou `zero` ou o sucessor de algum outro natural.
    *   `isZero: Natural --> Boolean` é um observador que testa se um natural é zero.
    *   `add: Natural x Natural --> Natural` é a operação de adição.
    *   `multiply: Natural x Natural --> Natural` é a operação de multiplicação.
    *   `isEqualN: Natural x Natural --> Boolean` testa a igualdade entre dois naturais.
    *   `pred: Natural --> Natural` é a operação predecessor (com `pred(zero)` definido como `zero` para manter o fechamento no sort `Natural`).
*   **Axiomas:**
    *   (N1) e (N2) definem `isZero` em termos dos construtores.
    *   (N3) e (N4) definem `add` recursivamente sobre a estrutura do segundo argumento. (N3) é o caso base (adicionar zero). (N4) é o passo recursivo (adicionar $n$ a $m+1$ é o mesmo que adicionar $n$ a $m$ e depois tomar o sucessor).
    *   (N5) e (N6) definem `multiply` recursivamente. (N5) é o caso base (multiplicar por zero). (N6) define $n \times (m+1)$ como $(n \times m) + n$.
    *   (N7) a (N10) definem `isEqualN` recursivamente. Dois naturais são iguais se ambos são `zero`, ou se ambos são sucessores e os números dos quais eles são sucessores são iguais. Se um é `zero` e o outro não, eles não são iguais.
    *   (N11) e (N12) definem `pred` recursivamente. O predecessor de `zero` é `zero`. O predecessor de `succ(n)` é `n`.

Esta especificação, na semântica inicial, define os números naturais padrão. Os axiomas são consistentes e suficientemente completos (no sentido que será definido no Capítulo 4). Por exemplo, a partir destes axiomas, podemos derivar (provar) outras propriedades, como a comutatividade e associatividade da adição e multiplicação, embora essas provas geralmente requeiram o princípio da indução, que não é uma regra de inferência da lógica equacional pura, mas pode ser adicionado como um esquema de axioma ou tratado na metateoria.

### 2.5.3 Especificação de Integer

O tipo de dados `Integer` (números inteiros: $\ldots, -2, -1, 0, 1, 2, \ldots$) estende os naturais para incluir números negativos. Há várias maneiras de especificar `Integer` algebricamente. Uma abordagem comum é construí-los a partir de dois cópias dos naturais (para positivos e negativos, mais o zero), ou usando construtores como `zeroI` (zero inteiro), `succI` (sucessor inteiro) e `predI` (predecessor inteiro, que agora é fundamental e não apenas um derivado como em `Natural`).

Vamos apresentar uma especificação que usa `zeroI`, `posSucc` (para construir positivos a partir de um natural que representa a magnitude) e `negSucc` (para construir negativos a partir de um natural que representa a magnitude da parte negativa). Esta é uma abordagem um pouco mais complexa, mas ilustra como múltiplos construtores podem interagir. (Uma abordagem mais simples com `zeroI`, `succI`, `predI` seria mais direta, mas esta é mais explícita sobre a estrutura).

**SPEC ADT** Integer

**sorts:**
*   Integer
*   Natural  // Assumido como especificado em 2.5.2
*   Boolean  // Assumido como especificado em 2.5.1

**operations:**
*   zeroInt: --> Integer                       // O inteiro 0
*   pos: Natural --> Integer                   // Constrói um inteiro positivo ou zero a partir de um Natural
                                             // pos(zero) é 0, pos(succ(n)) é n+1 positivo
*   neg: Natural --> Integer                   // Constrói um inteiro negativo a partir de um Natural > 0
                                             // neg(succ(n)) é -(n+1)
*   addI: Integer x Integer --> Integer        // Adição de inteiros
*   negateI: Integer --> Integer              // Negação de um inteiro (-i)
*   isEqualI: Integer x Integer --> Boolean   // Igualdade de inteiros
*   isZeroI: Integer --> Boolean

**axioms:**
// Relação entre construtores e normalização:
*   (Axiom I1): pos(zero) = zeroInt                            // O positivo de 0 natural é 0 inteiro.
*   (Axiom I2): neg(zero) = zeroInt                             // O negativo de 0 natural (não canônico) também é 0 inteiro.

// Definição de isZeroI:
*   (Axiom I3): isZeroI(zeroInt) = true
*   (Axiom I4): isZeroI(pos(succ(n))) = false
*   (Axiom I5): isZeroI(neg(succ(n))) = false

// Definição de negateI:
*   (Axiom I6): negateI(zeroInt) = zeroInt
*   (Axiom I7): negateI(pos(n)) = neg(n)
*   (Axiom I8): negateI(neg(n)) = pos(n)

// Definição de addI (simplificada - apenas alguns casos para ilustração):
// Adicionar zero:
*   (Axiom I9): addI(i, zeroInt) = i
*   (Axiom I10): addI(zeroInt, i) = i
// Adicionar dois positivos:
*   (Axiom I11): addI(pos(n), pos(m)) = pos(add(n,m))  // Reusa 'add' de Natural
// Adicionar dois negativos:
*   (Axiom I12): addI(neg(n), neg(m)) = neg(add(n,m))  // Reusa 'add' de Natural
// Adicionar um positivo e um negativo (pos(n) + neg(m)):
// Se n >= m (Natural): pos(n) + neg(m) = pos(subtract(n,m)) onde subtract é a subtração de Naturais
// Se m > n (Natural): pos(n) + neg(m) = neg(subtract(m,n))
// Isso requer uma operação 'subtract' e 'isGreaterThanOrEqual' em Naturais.
// Para simplificar aqui, não vamos detalhar todos os axiomas de addI.
// Uma especificação completa de addI seria mais extensa.
// Exemplo de um caso mais complexo para addI (requer 'greaterThan' e 'subtract' em Natural):
// (Axiom I13_cond): greaterThan(n,m) = true implies addI(pos(n), neg(m)) = pos(subtractNat(n,m))
// (Axiom I14_cond): greaterThan(m,n) = true implies addI(pos(n), neg(m)) = neg(subtractNat(m,n))
// (Axiom I15_cond): isEqualN(n,m) = true implies addI(pos(n), neg(m)) = zeroInt

// Definição de isEqualI (simplificada):
*   (Axiom I16): isEqualI(zeroInt, zeroInt) = true
*   (Axiom I17): isEqualI(pos(n), pos(m)) = isEqualN(n,m)
*   (Axiom I18): isEqualI(neg(n), neg(m)) = isEqualN(n,m) // (n,m devem ser > 0 para neg canônico)
*   (Axiom I19): isEqualI(pos(succ(n)), zeroInt) = false
*   (Axiom I20): isEqualI(neg(succ(n)), zeroInt) = false
*   (Axiom I21): isEqualI(pos(succ(n)), neg(succ(m))) = false // Positivo nunca é igual a negativo

for all n, m: Natural, i: Integer

**END SPEC**

Esta especificação para `Integer` é mais complexa e ilustra alguns desafios:
*   **Múltiplos Construtores e Canonicidade:** Usamos `zeroInt`, `pos(n)` e `neg(n)` como "quase-construtores". No entanto, `pos(zero)` e `neg(zero)` ambos representam o mesmo inteiro `zeroInt`. Os axiomas (I1) e (I2) tentam normalizar isso. Uma escolha mais canônica de construtores poderia ser `zeroInt`, `posSuccFromZero: Natural --> Integer` (onde `posSuccFromZero(n)` é $n+1$) e `negSuccFromZero: Natural --> Integer` (onde `negSuccFromZero(n)` é $-(n+1)$). A escolha dos construtores afeta a forma dos axiomas para as outras operações.
*   **Dependência de Outros TADs:** Esta especificação depende fortemente do TAD `Natural` e suas operações (`zero`, `succ`, `add`, `isEqualN`, e implicitamente precisaria de `subtractNat` e `greaterThan` para uma adição completa). Isso é comum e desejável (reuso de especificações).
*   **Complexidade dos Axiomas:** Definir operações como `addI` para todos os casos de sinais combinados (+/+, +/-, -/+, -/-) pode levar a um número maior de axiomas, possivelmente condicionais, se não for feita uma escolha inteligente de representação ou operações auxiliares. Uma abordagem alternativa comum é usar um único sucessor `succI` e um único predecessor `predI` como construtores (junto com `zeroInt`), e então definir `pos(n)` e `neg(n)` como operações derivadas.
*   **Operações Parciais/Condicionais:** O axioma (I18) para `isEqualI(neg(n), neg(m))` implicitamente assume que `n` e `m` são os argumentos para `neg` que o tornam canônico (i.e., $n, m > 0$ se `neg` for interpretado como $x \mapsto -x$). Uma especificação mais robusta lidaria com isso mais explicitamente.

Esta breve incursão na especificação de `Integer` mostra que, à medida que os TADs se tornam mais complexos, suas especificações algébricas também podem crescer em complexidade. A clareza, consistência e completeza dos axiomas tornam-se ainda mais cruciais e desafiadoras de alcançar. Os capítulos seguintes fornecerão ferramentas e técnicas para analisar essas propriedades.

**Exercício**
Usando a especificação `Integer` acima (com `zeroInt`, `pos(n)`, `neg(n)`), e assumindo que os axiomas para `negateI` são (I6-I8):
*   (Axiom I6): `negateI(zeroInt) = zeroInt`
*   (Axiom I7): `negateI(pos(n)) = neg(n)`
*   (Axiom I8): `negateI(neg(n)) = pos(n)`
Mostre como o termo `negateI(negateI(pos(succ(zero))))` seria simplificado para `pos(succ(zero))` usando esses axiomas. (Lembre-se que `succ(zero)` é um `Natural`).

**Resolução:**
Queremos simplificar o termo $T = \text{negateI}(\text{negateI}(\text{pos}(\text{succ(zero)}))))$.
Seja $N_1 = \text{succ(zero)}$ (um `Natural` que representa 1). O termo interno é $\text{pos}(N_1)$.

1.  **Simplificar o termo mais interno `negateI(pos(succ(zero)))`:**
    Este termo casa com o lado esquerdo do Axioma I7: `negateI(pos(n)) = neg(n)`.
    A substituição é $\{n \mapsto \text{succ(zero)}\}$.
    Aplicando o axioma, `negateI(pos(succ(zero)))` se torna `neg(succ(zero))`.

2.  **Substituir este resultado de volta no termo original:**
    Agora o termo é $T = \text{negateI}(\text{neg(succ(zero))})$.

3.  **Simplificar `negateI(neg(succ(zero)))`:**
    Este termo casa com o lado esquerdo do Axioma I8: `negateI(neg(n)) = pos(n)`.
    A substituição é $\{n \mapsto \text{succ(zero)}\}$.
    Aplicando o axioma, `negateI(neg(succ(zero)))` se torna `pos(succ(zero))`.

Portanto, `negateI(negateI(pos(succ(zero)))) = pos(succ(zero))`.
Isso demonstra a propriedade de que a dupla negação de um inteiro positivo (ou qualquer inteiro) retorna o próprio inteiro, o que é esperado.
□

---
Este capítulo forneceu uma introdução detalhada aos componentes sintáticos e semânticos da especificação algébrica. Estabelecemos a noção de assinatura (sorts e operações), termos (com e sem variáveis), substituições, e o papel central dos axiomas equacionais na definição do comportamento de um TAD. Vimos como uma especificação algébrica é um par $(\Sigma, E)$ e como as $\Sigma$-álgebras servem de modelos para tais especificações, com a semântica inicial fornecendo uma interpretação canônica. A distinção entre construtores, observadores e operações derivadas foi discutida como uma ferramenta conceitual útil no design de especificações. Finalmente, exemplos fundamentais para `Boolean`, `Natural` e `Integer` ilustraram a aplicação prática desses conceitos. O próximo capítulo se aprofundará na semântica operacional das especificações algébricas, introduzindo os Sistemas de Reescrita de Termos, que nos permitirão "computar" com os axiomas e analisar propriedades como terminação e confluência.

---
## REFERÊNCIAS BIBLIOGRÁFICAS

BERGSTRA, J. A.; HEERING, J.; KLINT, P. (Eds.). *Algebraic Specification*. ACM Press Frontier Series. New York, NY: Addison-Wesley, 1989.
*   *Resumo: Uma coletânea de artigos seminais que cobrem diversos aspectos da especificação algébrica, incluindo fundamentos teóricos, linguagens de especificação e aplicações. Oferece uma perspectiva histórica e aprofundada sobre o desenvolvimento da área.*

EHRIG, H.; MAHR, B. *Fundamentals of Algebraic Specification 1: Equations and Initial Semantics*. EATCS Monographs on Theoretical Computer Science, vol 6. Berlin, Heidelberg: Springer-Verlag, 1985.
*   *Resumo: O primeiro volume de uma obra clássica e abrangente sobre a teoria matemática das especificações algébricas. Foca em detalhes na lógica equacional, álgebras iniciais, e nas construções teóricas que fundamentam a semântica inicial. Altamente recomendado para um estudo rigoroso.*

GOGUEN, J. A.; THATCHER, J. W.; WAGNER, E. G. An initial algebra approach to the specification, correctness, and implementation of abstract data types. In: YEH, R. T. (Ed.). *Current Trends in Programming Methodology, Vol. IV: Data Structuring*. Englewood Cliffs, NJ: Prentice-Hall, 1978, p. 80-149.
*   *Resumo: Um dos artigos fundadores da abordagem de semântica inicial para especificações algébricas, escrito pelo grupo ADJ. Introduz muitos dos conceitos chave, como $\Sigma$-álgebras, homomorfismos, e a construção da álgebra inicial como uma álgebra de termos módulo uma congruência.*

GUTTAG, J. V. Abstract data types and the development of data structures. *Communications of the ACM*, v. 20, n. 6, p. 396-404, Jun. 1977.
*   *Resumo: Um artigo influente e acessível que argumenta a favor do uso de tipos abstratos de dados no processo de desenvolvimento de software. Discute como a especificação formal (incluindo abordagens axiomáticas) pode melhorar a clareza e a confiabilidade, e como ela se relaciona com a implementação de estruturas de dados.*

KLINT, P. A meta-language for interactive experiments with algebraic specifications. *The Computer Journal*, v. 30, n.4, p.344-353, 1987. (Referenciando ASF+SDF que evoluiu para ferramentas como Rascal MPL)
*   *Resumo: Embora focado em uma metalinguagem (ASF+SDF), este artigo e trabalhos subsequentes de Klint e outros no CWI demonstram a aplicação prática da especificação algébrica e da reescrita de termos na construção de ferramentas para análise e transformação de linguagens e software. Relevante para a conexão com a semântica operacional.*

WIRSING, M. Algebraic specification. In: VAN LEEUWEN, J. (Ed.). *Handbook of Theoretical Computer Science, Volume B: Formal Models and Semantics*. Amsterdam: Elsevier/MIT Press, 1990, p. 675-788.
*   *Resumo: Um capítulo de manual abrangente que serve como uma excelente pesquisa sobre o campo da especificação algébrica. Cobre fundamentos, diferentes tipos de semântica (inicial, final, solta), especificações estruturadas, implementação e aspectos de prova. Uma referência valiosa para uma visão geral e aprofundada.*
