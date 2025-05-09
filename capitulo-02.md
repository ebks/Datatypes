
---

# CAPÍTULO 2

# ESPECIFICAÇÃO ALGÉBRICA DE TIPOS ABSTRATOS DE DADOS

---

![Diagrama conceitual de uma Σ-Álgebra](tipo.png)

Após estabelecermos a importância da abstração de dados e a necessidade de especificações formais no capítulo anterior, este capítulo mergulha profundamente no coração da abordagem algébrica para a definição de Tipos Abstratos de Dados (TADs). Exploraremos os fundamentos da lógica equacional e das álgebras multissortidas, que fornecem o arcabouço matemático para esta técnica. Detalharemos os componentes cruciais de uma especificação algébrica: sorts, que definem os tipos de dados; assinaturas, que declaram as operações; e axiomas equacionais, que prescrevem o comportamento dessas operações. Investigaremos a semântica formal dessas especificações através do conceito de Σ-álgebras, que são as estruturas matemáticas que servem como modelos para um TAD. Distinguiremos entre diferentes tipos de operações, como construtores e observadores, e ilustraremos a teoria com exemplos fundamentais, incluindo a especificação de `Boolean`, `Natural` (baseado nos axiomas de Peano) e `Integer`. Ao final deste capítulo, o leitor terá uma compreensão sólida dos mecanismos formais para definir TADs algebricamente, preparando o caminho para análises mais avançadas de corretude, completeza e para a aplicação desses conceitos na construção e verificação de software.

---

## 2.1 Introdução à Lógica Equacional e Álgebras Multissortidas

A especificação algébrica de Tipos Abstratos de Dados (TADs) é fundamentada na lógica equacional e na teoria das álgebras universais, mais especificamente, álgebras multissortidas. A lógica equacional é um sistema formal para raciocinar sobre igualdade. Uma álgebra multissortida, por sua vez, é uma estrutura matemática que consiste em múltiplos conjuntos de elementos (chamados *sorts* ou *portadores*) e um conjunto de operações definidas sobre esses sorts. Esta combinação nos permite definir TADs de uma maneira puramente sintática (através de assinaturas e axiomas) e, subsequentemente, interpretar essa definição semanticamente através de estruturas algébricas que satisfazem os axiomas.

**Sorts, Operações e Assinaturas (Σ-Signatures)**

No contexto da especificação algébrica, um **sort** (plural: sorts) é um nome que representa um tipo de dado ou um domínio de valores. Por exemplo, em uma especificação de números naturais, `Natural` seria um sort; em uma especificação de pilhas, `Stack` e `Elem` (para os elementos da pilha) seriam sorts. A utilização de múltiplos sorts (daí o termo "multissortida") é essencial para modelar TADs que envolvem diferentes tipos de dados interagindo entre si.

Uma **operação** (ou função símbolo) é definida em termos dos sorts de seus argumentos (seu *domínio* ou *aridade*) e o sort de seu resultado (seu *contradomínio* ou *tipo de resultado*). Uma operação que não possui argumentos é chamada de **constante**. Por exemplo, a operação `zero` que representa o número natural 0 pode ser definida como uma constante do sort `Natural`. A operação `suc` que pega um `Natural` e retorna o próximo `Natural` é uma operação unária. A operação `add` que pega dois `Naturais` e retorna um `Natural` é uma operação binária.

Uma **assinatura multissortida (Σ-Signature)**, denotada por Σ, é um par $(S, \Omega)$ onde $S$ é um conjunto de nomes de sorts e $\Omega$ é uma família de conjuntos de símbolos de operações indexados por seus perfis de aridade e tipo. Formalmente, para cada $s_1, \ldots, s_n \in S$ (com $n \ge 0$) e $s \in S$, $\Omega_{s_1 \ldots s_n, s}$ é o conjunto de símbolos de operações que tomam $n$ argumentos dos sorts $s_1, \ldots, s_n$ respectivamente, e produzem um resultado do sort $s$. Se $n=0$, a operação é uma constante do sort $s$.

Por exemplo, uma assinatura para números naturais (Peano), que também utilizaria o sort `Boolean` para algumas operações observadoras, poderia ser parcialmente descrita da seguinte forma:

> **Signature (Exemplo para `Natural` e `Boolean`)**
>
> **Sorts (parcial):**
> *   `Natural`
> *   `Boolean`
>
> **Operations (parcial - Σ<sub>Natural_Example</sub>):**
> *   `zero:` $\rightarrow$ `Natural`
>     - Representa o número natural zero.
> *   `suc:` `Natural` $\rightarrow$ `Natural`
>     - Representa a função sucessor.
> *   `add:` `Natural` $\times$ `Natural` $\rightarrow$ `Natural`
>     - Representa a adição de números naturais.
> *   `isZero:` `Natural` $\rightarrow$ `Boolean`
>     - Verifica se um número natural é zero.
> *   (*...e operações para `Boolean` como `true`, `false`, `not`, etc.*)

Esta estrutura formal de sorts e operações definidas por uma assinatura Σ estabelece o vocabulário sintático básico para construir termos e, subsequentemente, axiomas.

**Termos sobre uma Assinatura (T<sub>Σ</sub>(X)). Variáveis e Termos Fundamentais (`Ground Terms`)**

Dado uma Σ-assinatura e um conjunto $X$ de variáveis (onde cada variável $x \in X$ tem um sort associado $s \in S$), podemos definir o conjunto de **termos sobre Σ com variáveis em X**, denotado $T_{\Sigma}(X)$. Um termo é uma expressão construída recursivamente usando os símbolos de operação da assinatura e as variáveis:

1.  Toda variável $x \in X$ do sort $s$ é um termo do sort $s$.
2.  Se $f: s_1 \times \ldots \times s_n \rightarrow s$ é um símbolo de operação em Σ (se $n=0$, $f$ é uma constante) e $t_1, \ldots, t_n$ são termos dos sorts $s_1, \ldots, s_n$ respectivamente, então $f(t_1, \ldots, t_n)$ é um termo do sort $s$.

Se o conjunto de variáveis $X$ é vazio ($X = \emptyset$), os termos resultantes são chamados de **termos fundamentais (`ground terms`)** ou termos fechados, denotados $T_{\Sigma}$. Estes são termos que não contêm variáveis; eles representam todos os valores concretos que podem ser construídos usando apenas as operações da assinatura.

Por exemplo, usando a assinatura de `Natural` simplificada com `zero` e `suc`:
*   `zero` é um termo fundamental do sort `Natural`.
*   `suc(zero)` é um termo fundamental do sort `Natural`.
*   `suc(suc(zero))` é um termo fundamental do sort `Natural`.
Se tivermos uma variável $n$ do sort `Natural`, então:
*   $n$ é um termo (não fundamental) do sort `Natural`.
*   `suc(n)` é um termo (não fundamental) do sort `Natural`.
Se `add` está na assinatura, então o termo `add(suc(zero), n)` é um termo (não fundamental) do sort `Natural`.

Os termos são as "expressões" ou "programas" que podemos escrever na linguagem definida pela assinatura. Os termos fundamentais são particularmente importantes, pois representam os valores canônicos do tipo de dado que está sendo especificado, especialmente quando consideramos modelos iniciais (discutidos adiante).

**Substituições e Instanciação**

Uma **substituição** é um mapeamento de variáveis para termos, respeitando os sorts. Se $\sigma$ é uma substituição e $x$ é uma variável do sort $s$, então $\sigma(x)$ (o termo que substitui $x$) deve também ser do sort $s$. A aplicação de uma substituição $\sigma$ a um termo $t$, denotada $t\sigma$ ou $\sigma(t)$, resulta em um novo termo onde cada ocorrência de uma variável $x$ em $t$ (que está no domínio de $\sigma$) é substituída por $\sigma(x)$. Este processo é chamado de **instanciação**.

Por exemplo, se $t$ é o termo `add(x, suc(y))` e a substituição $\sigma = \{x \mapsto \text{`zero`}, y \mapsto \text{`suc(zero)`}\}$, então $t\sigma$ é o termo `add(zero, suc(suc(zero)))`.
Substituições são fundamentais para definir a semântica dos axiomas, que geralmente são equações entre termos contendo variáveis universalmente quantificadas.

---

## 2.2 Axiomas Equacionais e Especificações Algébricas

Uma vez que temos a sintaxe (sorts, operações, termos), precisamos definir a semântica, ou seja, o comportamento das operações. Na especificação algébrica, isso é feito através de **axiomas equacionais**.

**Definição de Axiomas como Equações entre Termos**

Um **axioma equacional** (ou simplesmente uma **equação**) é uma expressão da forma $t_L = t_R$, onde $t_L$ e $t_R$ são termos do mesmo sort, possivelmente contendo variáveis. As variáveis em uma equação são implicitamente (ou explicitamente, com $\forall$) universalmente quantificadas sobre seus respectivos sorts. Uma equação $t_L = t_R$ postula que, para qualquer atribuição de valores (termos fundamentais de sorts apropriados) às variáveis presentes em $t_L$ e $t_R$, os termos resultantes $t_L'$ e $t_R'$ denotam o mesmo valor na álgebra que interpreta a especificação.

Por exemplo, para o TAD `Stack` (com variáveis $s: \text{Stack}, e: \text{Elem}$), o axioma `pop(push(e, s)) = s` afirma que, para qualquer elemento `e` e qualquer pilha `s`, aplicar `pop` ao resultado de `push(e, s)` é o mesmo que a pilha original `s`.

Outro exemplo, para `Natural` (com variáveis $m, n: \text{Natural}$), temos os axiomas de Peano para a adição:
*   `add(m, zero) = m`
*   `add(m, suc(n)) = suc(add(m, n))`

**Uma Especificação Algébrica SP = (Σ, E) como um par Assinatura-Axiomas**

Formalmente, uma **especificação algébrica** (ou Teoria Equacional Apresentada) é um par $SP = (\Sigma, E)$, onde:
*   $\Sigma$ é uma assinatura multissortida (definindo os sorts $S$ e as operações $\Omega$).
*   $E$ é um conjunto de $\Sigma$-equações (os axiomas), onde os termos em cada equação são construídos a partir de $\Sigma$ e de um conjunto $X$ de variáveis (cujos sorts estão em $S$).

> **Definição 2.1: Especificação Algébrica**
>
> Uma Especificação Algébrica $SP$ é um par $(\Sigma, E)$, onde $\Sigma$ é uma assinatura multissortida e $E$ é um conjunto de $\Sigma$-equações.
> Os sorts, operações e variáveis em $E$ devem estar de acordo com $\Sigma$.

Esta definição é central. Ela captura a ideia de que um TAD é definido puramente por sua sintaxe (o que pode ser escrito) e pelas leis (axiomas) que governam a igualdade dessas expressões sintáticas.

**Consequência Lógica e Derivação (Regras de Inferência da Lógica Equacional)**

Dado uma especificação $SP = (\Sigma, E)$, uma equação $t_L = t_R$ é uma **consequência lógica** de $E$, denotado $E \models t_L = t_R$, se $t_L = t_R$ é válida em todas as álgebras que satisfazem todas as equações em $E$ (a noção de "satisfação em uma álgebra" será definida na próxima seção).

Alternativamente, podemos definir um sistema de **derivação** formal para a lógica equacional. Uma equação $t_L = t_R$ é derivável de $E$, denotado $E \vdash t_L = t_R$, se ela pode ser obtida a partir dos axiomas em $E$ e das regras de inferência da lógica equacional. As regras de inferência básicas são:

1.  **Reflexividade:** Para qualquer termo $t$, $E \vdash t = t$.
2.  **Simetria:** Se $E \vdash t_1 = t_2$, então $E \vdash t_2 = t_1$.
3.  **Transitividade:** Se $E \vdash t_1 = t_2$ e $E \vdash t_2 = t_3$, então $E \vdash t_1 = t_3$.
4.  **Congruência (Substitutividade para operações):** Se $f: s_1 \times \ldots \times s_n \rightarrow s$ é uma operação em Σ, e $E \vdash t_i = t'_i$ para $i=1, \ldots, n$ (onde $t_i, t'_i$ são termos do sort $s_i$), então $E \vdash f(t_1, \ldots, t_n) = f(t'_1, \ldots, t'_n)$.
5.  **Substituição (Instanciação de axiomas):** Se $(u = v) \in E$ é um axioma (com variáveis $X_E$) e $\sigma$ é uma substituição que mapeia as variáveis em $X_E$ para termos sobre Σ (com variáveis $X$), então $E \vdash u\sigma = v\sigma$.

Um resultado fundamental da lógica equacional (Teorema da Completude de Birkhoff para Lógica Equacional) estabelece que o sistema de derivação é **sólido** ($E \vdash eq \implies E \models eq$) e **completo** ($E \models eq \implies E \vdash eq$). Isso significa que podemos usar provas formais (derivações) para estabelecer todas as consequências lógicas dos axiomas.

---
**Exercício 2.1:**
Dada a especificação `Stack` com axiomas (S1)-(S4) e um novo axioma (S5) `top(new) = errorElem` (assumindo `errorElem` é uma constante de `Elem`), mostre uma derivação informal de `top(pop(push(e1, push(e2, new)))) = e2`, listando os axiomas usados em cada passo.

**Resolução do Exercício 2.1:**

Queremos derivar `top(pop(push(e1, push(e2, new)))) = e2`.

1.  Considere o termo interno $pop(push(e1, push(e2, new)))$.
    Seja $s' = push(e2, new)$. Então o termo é $pop(push(e1, s'))$.
    Pelo Axioma (S4), `pop(push(x, y)) = y`.
    Instanciando com $x \mapsto e1$ e $y \mapsto s'$, temos:
    $pop(push(e1, push(e2, new))) = push(e2, new)$ (Usando S4)

2.  Agora, substituímos isso na expressão original:
    $top(pop(push(e1, push(e2, new))))$ se torna $top(push(e2, new))$ (Pelo passo 1 e regra de congruência para `top`)

3.  Considere o termo $top(push(e2, new))$.
    Pelo Axioma (S3), `top(push(x, y)) = x`.
    Instanciando com $x \mapsto e2$ e $y \mapsto new$, temos:
    $top(push(e2, new)) = e2$ (Usando S3)

Portanto, por transitividade da igualdade, combinando os passos 2 e 3:
`top(pop(push(e1, push(e2, new)))) = e2`.
∎

(Nota: O axioma (S5) não foi necessário para esta derivação específica, pois `pop` e `top` nunca foram aplicados diretamente a `new` de forma que gerasse erro neste caminho.)

---

## 2.3 Semântica das Especificações Algébricas: Σ-Álgebras

Até agora, lidamos com a sintaxe das especificações algébricas. Para dar significado (semântica) a uma especificação $SP = (\Sigma, E)$, introduzimos o conceito de **Σ-álgebra**. Uma Σ-álgebra é uma estrutura matemática concreta que fornece uma interpretação para os sorts e símbolos de operação da assinatura Σ.

**Definição de uma Σ-Álgebra: Domínios Portadores e Interpretação de Operações**

Dado uma assinatura multissortida $\Sigma = (S, \Omega)$, uma **Σ-álgebra** $A$ consiste em:

1.  Para cada nome de sort $s \in S$, um conjunto não vazio $A_s$, chamado **domínio portador** (ou carrier set) do sort $s$ na álgebra $A$. Este conjunto contém os "valores" concretos do tipo $s$ no modelo $A$.
2.  Para cada símbolo de operação $f \in \Omega_{s_1 \ldots s_n, s}$ (com $n \ge 0$), uma função (ou operação concreta) $f_A: A_{s_1} \times \ldots \times A_{s_n} \rightarrow A_s$. Esta função $f_A$ é a interpretação do símbolo de operação $f$ na álgebra $A$. Se $f$ é uma constante ($n=0$), $f_A$ é um elemento específico do portador $A_s$.

> **Definição 2.2: Σ-Álgebra**
>
> Seja $\Sigma = (S, \Omega)$ uma assinatura multissortida. Uma Σ-álgebra $A$ é uma família de conjuntos $\{A_s\}_{s \in S}$ (os domínios portadores) e uma família de funções $\{f_A\}_{f \in \Omega}$ (as interpretações das operações) tal que se $f \in \Omega_{s_1 \ldots s_n, s}$, então $f_A: A_{s_1} \times \ldots \times A_{s_n} \rightarrow A_s$.

Por exemplo, para uma assinatura de `Boolean` com sorts $S = \{\text{Boolean}\}$ e operações `true`: $\rightarrow \text{Boolean}$, `false`: $\rightarrow \text{Boolean}$, `not`: $\text{Boolean} \rightarrow \text{Boolean}$:
Uma Σ-álgebra $A_{std}$ (a álgebra padrão dos booleanos) seria:
*   $A_{std, \text{Boolean}} = \{\mathbf{T}, \mathbf{F}\}$ (dois valores distintos para verdade e falsidade)
*   $\text{true}_{A_{std}} = \mathbf{T}$
*   $\text{false}_{A_{std}} = \mathbf{F}$
*   $\text{not}_{A_{std}}(\mathbf{T}) = \mathbf{F}$, $\text{not}_{A_{std}}(\mathbf{F}) = \mathbf{T}$

Qualquer termo $t \in T_{\Sigma}(X)$ pode ser interpretado em uma Σ-álgebra $A$ dado uma **valoração** (ou atribuição) $\alpha: X \rightarrow A$ que mapeia cada variável $x$ de sort $s$ em $X$ para um valor $\alpha(x)$ no portador $A_s$. A interpretação de $t$ em $A$ sob $\alpha$, denotada $eval_A(t, \alpha)$ ou $t_{A,\alpha}$, é definida recursivamente:
*   Se $t = x$ (uma variável), $eval_A(x, \alpha) = \alpha(x)$.
*   Se $t = f(t_1, \ldots, t_n)$, $eval_A(f(t_1, \ldots, t_n), \alpha) = f_A(eval_A(t_1, \alpha), \ldots, eval_A(t_n, \alpha))$.
Se $t$ é um termo fundamental (sem variáveis), sua interpretação $t_A$ não depende de $\alpha$.

**Satisfação de Equações em uma Álgebra. Modelos de uma Especificação**

Uma Σ-álgebra $A$ **satisfaz** uma Σ-equação $(X, t_L = t_R)$ (onde $X$ é o conjunto de variáveis na equação), denotado $A \models t_L = t_R$, se para toda valoração $\alpha: X \rightarrow A$, temos $eval_A(t_L, \alpha) = eval_A(t_R, \alpha)$. Ou seja, a equação se torna uma identidade na álgebra $A$ quando as variáveis são substituídas por quaisquer valores dos portadores apropriados.

Uma Σ-álgebra $A$ é um **modelo** de uma especificação algébrica $SP = (\Sigma, E)$ se $A$ satisfaz todas as equações (axiomas) em $E$. Escrevemos $A \models SP$.

Por exemplo, a álgebra $A_{std}$ dos booleanos satisfaz os axiomas usuais como `not(true) = false`. Se uma álgebra interpretasse $not(true)$ como $true$, ela não seria um modelo da especificação padrão de `Boolean`.

**A Classe de Todas as Σ-Álgebras que Satisfazem E (Alg(SP)). Modelos Iniciais e Finais.**

Para uma dada especificação $SP = (\Sigma, E)$, a classe de todas as Σ-álgebras que são modelos de $SP$ é denotada por $Alg(SP)$. Esta classe pode conter muitas álgebras diferentes, algumas mais "abstratas" ou "concretas" que outras.

Dentro de $Alg(SP)$, dois tipos de modelos são de particular interesse:

1.  **Modelo Inicial (ou Álgebra Inicial):** Uma álgebra $I \in Alg(SP)$ é inicial se, para qualquer outra álgebra $A \in Alg(SP)$, existe um único homomorfismo de Σ-álgebras $h: I \rightarrow A$. (Um homomorfismo preserva a estrutura das operações). Intuïtivamente, a álgebra inicial é o modelo "mais abstrato" ou "menos confuso". Seus elementos são essencialmente os termos fundamentais da assinatura, quocientados apenas pelas igualdades que são prováveis a partir dos axiomas, e nada mais (sem "confusão" de elementos distintos a menos que os axiomas forcem). Frequentemente, $I$ é isomorfa a $T_{\Sigma} / \equiv_E$, a álgebra de termos fundamentais onde $\equiv_E$ é a relação de equivalência induzida pelos axiomas de $E$.
    A álgebra inicial capta a ideia de "não há lixo" (todos os elementos são representáveis por termos) e "não há confusão" (dois termos são iguais se e somente se sua igualdade é uma consequência dos axiomas).

2.  **Modelo Final (ou Álgebra Final):** Uma álgebra $Z \in Alg(SP)$ é final se, para qualquer outra álgebra $A \in Alg(SP)$, existe um único homomorfismo $h: A \rightarrow Z$. Intuïtivamente, a álgebra final é o modelo "mais concreto" ou "mais confuso". Ela identifica o máximo de termos possível sem violar os axiomas. Modelos finais são frequentemente usados para definir semântica observacional, onde os valores são distinguidos apenas por como se comportam sob operações "observadoras".

A escolha entre semântica inicial ou final (ou outras) depende do que se quer enfatizar na especificação. A semântica inicial é muito comum para TADs, pois foca nas propriedades dedutíveis dos axiomas.

## 2.4 Construtores, Observadores e Operações Derivadas

Ao projetar uma especificação algébrica, é útil classificar as operações com base em seu papel na definição do TAD. As categorias mais comuns são construtores, observadores e, às vezes, operações derivadas ou modificadores.

*   **Construtores (`Constructors`):** São as operações usadas para gerar todos os valores do sort principal do TAD. Um conjunto de construtores é escolhido de forma que todo termo fundamental do sort de interesse possa ser expresso (ou é igual, via axiomas) a um termo construído apenas com essas operações construtoras.
    *   Para `Stack`: As operações `new` e `push` são tipicamente os construtores. Qualquer pilha pode ser representada como um termo da forma `push(e_n, ... push(e_1, new)...)`.
    *   Para `Natural` (Peano): As operações `zero` e `suc` são os construtores.
    A escolha de construtores é crucial para analisar propriedades como completeza (discutida no Capítulo 4).

*   **Observadores (`Observers`):** São operações que retornam valores de sorts "mais primitivos" ou já definidos (como `Boolean`, `Integer`, ou `Elem` no caso do `Stack`), fornecendo informações sobre o estado de um valor do TAD sem modificá-lo no sentido de retornar um novo valor do mesmo TAD.
    *   Para `Stack`: As operações `top` (retorna `Elem`) e `isEmpty` (retorna `Boolean`) são observadores.
    *   Para `Natural`: A operação `isZero` (retorna `Boolean`) é um observador.

*   **Operações Derivadas (ou Modificadores que não são construtores puros):** São operações cujo comportamento pode ser definido em termos dos construtores e observadores, ou que transformam um valor do TAD em outro valor do mesmo TAD, mas não são essenciais para gerar todos os valores.
    *   Para `Stack`: A operação `pop` é um exemplo. Ele retorna um `Stack`, mas seu resultado pode ser definido pelos axiomas em termos de `push` e `new`. Por exemplo, o axioma `pop(push(e,s)) = s`.
    *   Para `Natural`: A operação `add` (com termo genérico `add(m,n)`) é uma operação derivada. Seus axiomas a definem recursivamente usando `zero` e `suc`.

A distinção entre construtores e outras operações é fundamental para várias técnicas de análise de especificações, como a prova de completeza suficiente, onde se verifica se todas as operações não construtoras estão definidas para todos os casos de aplicação dos construtores.

Idealmente, os axiomas de um TAD devem definir o comportamento das operações observadoras e derivadas quando aplicadas a termos construídos a partir dos construtores. Por exemplo, os axiomas do `Stack` definem las operações `isEmpty`, `top` e `pop` em termos das operações `new` e `push`.

## 2.5 Exemplos Fundamentais de Especificações Algébricas

Para solidificar os conceitos apresentados, vamos detalhar as especificações algébricas de alguns TADs fundamentais.

**Especificação de `Boolean`**

Este é o TAD mais básico, representando os valores de verdade.

> **SPEC ADT `Boolean`**
>
> **Sorts:**
> *   `Boolean` - o sort dos valores de verdade.
>
> **Signature (Σ<sub>Boolean</sub>):**
> *   `true:` $\rightarrow$ `Boolean`
>     - A constante booleana representando verdade. (Construtor)
> *   `false:` $\rightarrow$ `Boolean`
>     - A constante booleana representando falsidade. (Construtor)
> *   `not:` `Boolean` $\rightarrow$ `Boolean`
>     - A operação de negação lógica.
> *   `and:` `Boolean` $\times$ `Boolean` $\rightarrow$ `Boolean`
>     - A operação de conjunção lógica (E).
> *   `or:` `Boolean` $\times$ `Boolean` $\rightarrow$ `Boolean`
>     - A operação de disjunção lógica (OU).
> *   `eqB:` `Boolean` $\times$ `Boolean` $\rightarrow$ `Boolean`
>     - A operação de igualdade booleana.
>
> **Axioms (E<sub>Boolean</sub>):**
>
> `For all b, b1, b2: Boolean:`
>
> *   (B1) `not(true) = false`
> *   (B2) `not(false) = true`
> *   (B3) `and(true, b) = b`
> *   (B4) `and(false, b) = false`
> *   (B5) `or(true, b) = true`
> *   (B6) `or(false, b) = b`
> *   (B7) `eqB(true, true) = true`
> *   (B8) `eqB(false, false) = true`
> *   (B9) `eqB(true, false) = false`
> *   (B10) `eqB(false, true) = false`
>
> Nota: As operações `true` e `false` são os construtores. É implicitamente assumido na semântica inicial que $true \neq false$, pois não há axioma que os iguale. Os axiomas para `not`, `and`, `or` e `eqB` definem seu comportamento quando aplicados aos construtores.

**Especificação de `Natural` (Números Naturais segundo Peano)**

Esta especificação formaliza os números naturais ($0, 1, 2, \ldots$) usando os axiomas de Peano.

> **SPEC ADT `Natural`**
>
> **Uses:** `Boolean` (assumimos que o TAD `Boolean` e suas operações já foram especificados)
>
> **Sorts:**
> *   `Natural` - o sort dos números naturais.
>
> **Signature (Σ<sub>Natural</sub>):**
> *   `zero:` $\rightarrow$ `Natural`
>     - A constante zero. (Construtor)
> *   `suc:` `Natural` $\rightarrow$ `Natural`
>     - A operação sucessor (n+1). (Construtor)
> *   `add:` `Natural` $\times$ `Natural` $\rightarrow$ `Natural`
>     - A operação de adição.
> *   `mult:` `Natural` $\times$ `Natural` $\rightarrow$ `Natural`
>     - A operação de multiplicação.
> *   `isZero:` `Natural` $\rightarrow$ `Boolean`
>     - Verifica se um natural é zero. (Observador)
> *   `eqN:` `Natural` $\times$ `Natural` $\rightarrow$ `Boolean`
>     - Verifica a igualdade entre dois naturais. (Observador)
>
> **Axioms (E<sub>Natural</sub>):**
>
> `For all n, m: Natural:`
>
> *   (N1) `isZero(zero) = true`
> *   (N2) `isZero(suc(n)) = false`
>
> *   (N3) `add(n, zero) = n`
> *   (N4) `add(n, suc(m)) = suc(add(n, m))`
>
> *   (N5) `mult(n, zero) = zero`
> *   (N6) `mult(n, suc(m)) = add(mult(n, m), n)`
>
> *   (N7) `eqN(zero, zero) = true`
> *   (N8) `eqN(suc(n), zero) = false`
> *   (N9) `eqN(zero, suc(m)) = false`
> *   (N10) `eqN(suc(n), suc(m)) = eqN(n, m)`
>
> Nota: As operações `zero` e `suc` são os construtores. O axioma de Peano que afirma que o termo `suc(n)` nunca é `zero` é capturado por (N2) e (N8)/(N9). O axioma de que `suc` é injetivo (ou seja, se $suc(n) = suc(m)$ então $n = m$) é capturado por (N10) através da operação `eqN`. A indução não é diretamente expressa como um axioma de primeira ordem aqui, mas é uma propriedade da álgebra inicial desta especificação.

**Especificação de `Integer` (Inteiros)**

A especificação de inteiros pode ser feita de várias maneiras. Aqui, apresentaremos uma abordagem que usa os construtores `zeroI`, `sucI` (sucessor) e `predI` (predecessor).

> **SPEC ADT `Integer`**
>
> **Uses:** `Boolean`
>
> **Sorts:**
> *   `Integer` - o sort dos números inteiros.
>
> **Signature (Σ<sub>Integer</sub>):**
> *   `zeroI:` $\rightarrow$ `Integer`
>     - A constante zero inteiro. (Construtor)
> *   `sucI:` `Integer` $\rightarrow$ `Integer`
>     - A operação sucessor inteiro (i+1). (Construtor)
> *   `predI:` `Integer` $\rightarrow$ `Integer`
>     - A operação predecessor inteiro (i-1). (Construtor)
> *   `addI:` `Integer` $\times$ `Integer` $\rightarrow$ `Integer`
>     - A operação de adição inteira.
> *   `negI:` `Integer` $\rightarrow$ `Integer`
>     - A operação de negação (oposto aditivo).
> *   `isZeroI:` `Integer` $\rightarrow$ `Boolean`
>     - Verifica se um inteiro é zero.
> *   `eqI:` `Integer` $\times$ `Integer` $\rightarrow$ `Boolean`
>      - Verifica a igualdade entre dois inteiros.
>
> **Axioms (E<sub>Integer</sub>):**
>
> `For all i, j: Integer:`
>
> *   (I1) `sucI(predI(i)) = i`
> *   (I2) `predI(sucI(i)) = i`
>
> *   (I3) `addI(i, zeroI) = i`
> *   (I4) `addI(i, sucI(j)) = sucI(addI(i, j))`
> *   (I5) `addI(i, predI(j)) = predI(addI(i, j))`
>
> *   (I6) `negI(zeroI) = zeroI`
> *   (I7) `negI(sucI(i)) = predI(negI(i))`
> *   (I8) `negI(predI(i)) = sucI(negI(i))`
>
> *   (I9) `isZeroI(zeroI) = true`
> *   (I10) `isZeroI(sucI(i)) = eqI(sucI(i), zeroI)`
> *   (I11) `isZeroI(predI(i)) = eqI(predI(i), zeroI)`
>
> *   (I12) `eqI(zeroI, zeroI) = true`
> *   (I13) `eqI(sucI(i), zeroI) = isZeroI(predI(sucI(i)))`
>
> Nota: As operações `zeroI`, `sucI`, e `predI` podem ser considerados construtores, embora haja redundância (e.g., `sucI(predI(i)) = i`). Axiomas (I1) e (I2) estabelecem a relação inversa entre sucessor e predecessor. A especificação completa de `eqI` e `isZeroI` de forma não circular e completa requer mais cuidado e é omitida aqui para brevidade, mas envolveria casos para `sucI` e `predI` em ambos os argumentos ou seria baseada na subtração.

Estes exemplos ilustram como a estrutura de assinatura e axiomas pode ser usada para definir formalmente TADs bem conhecidos. A precisão e a natureza abstrata dessas definições são as principais vantagens da abordagem algébrica. Nos próximos capítulos, exploraremos como analisar essas especificações e como usá-las para guiar a implementação e a verificação.

---
**Exercício 2.2:**
Usando a especificação `Natural` (N1-N10), prove informalmente que `add(suc(zero), suc(zero)) = suc(suc(zero))`. Indique os axiomas usados.

**Resolução do Exercício 2.2:**

Queremos provar `add(suc(zero), suc(zero)) = suc(suc(zero))`.

1.  Seja o termo $add(suc(zero), suc(zero))$.
    Este termo corresponde ao formato do lado esquerdo do axioma (N4): `add(n, suc(m)) = suc(add(n, m))`.
    Podemos instanciar $n \mapsto suc(zero)$ e $m \mapsto zero$.
    Aplicando (N4):
    $add(suc(zero), suc(zero)) = suc(add(suc(zero), zero))$

2.  Agora, consideramos o termo interno $add(suc(zero), zero)$.
    Este termo corresponde ao formato do lado esquerdo do axioma (N3): `add(n, zero) = n`.
    Podemos instanciar $n \mapsto suc(zero)$.
    Aplicando (N3):
    $add(suc(zero), zero) = suc(zero)$

3.  Substituindo o resultado do passo 2 no resultado do passo 1:
    $suc(add(suc(zero), zero))$ se torna $suc(suc(zero))$

Portanto, por transitividade da igualdade, temos:
`add(suc(zero), suc(zero)) = suc(suc(zero))`.
∎

---
---
## REFERÊNCIAS BIBLIOGRÁFICAS

1.  EHRIG, Hartmut; MAHR, Bernd. **Fundamentals of Algebraic Specification 1: Equations and Initial Semantics.** Springer-Verlag, 1985. (EATCS Monographs on Theoretical Computer Science, Vol. 6) (ISBN 978-3540137182)
    *   *Resumo: Um texto clássico e fundamental que estabelece as bases teóricas da especificação algébrica, com foco em lógica equacional e semântica inicial. Embora antigo, é uma referência seminal para a teoria profunda da área.*

2.  WIRSING, Martin. **Algebraic Specification.** In: van Leeuwen, J. (Ed.) *Handbook of Theoretical Computer Science, Volume B: Formal Models and Semantics*. Elsevier/MIT Press, 1990, pp. 675-788. (ISBN 978-0262220392)
    *   *Resumo: Um capítulo de manual abrangente que serve como uma excelente pesquisa sobre o campo da especificação algébrica. Cobre desde os fundamentos até tópicos avançados como especificações estruturadas, implementações e semântica. Referência clássica e altamente citada.*

3.  MESEGUER, José; GOGUEN, Joseph. **Order-Sorted Algebra Solves the Constructor-Selector, Multiple Representation and Coercion Problems.** *Information and Computation*, vol. 103, no. 1, 1993, pp. 114-158.
    *   *Resumo: Este artigo seminal introduz e formaliza a Álgebra Ordenada por Sorts (Order-Sorted Algebra - OSA), uma extensão poderosa da especificação algébrica multissortida que lida elegantemente com subtipos, herança e tratamento de erros. Relevante para entender extensões da abordagem básica.*

4.  ASTESIANO, Egidio; BIDOIT, Michel; LESCANNE, Pierre; PAREDAENS, Jan; TARLECKI, Andrzej; WIRSING, Martin (Eds.). **Recent Trends in Data Type Specification: 17th International Workshop, WADT 2004, Barcelona, Spain, March 2004, Revised Selected Papers.** Springer, 2005. (Lecture Notes in Computer Science, Vol. 3423) (ISBN 978-3540261291)
    *   *Resumo: Uma coletânea de artigos de um workshop proeminente na área (WADT - Workshop on Algebraic Development Techniques). Embora específico de 2004, os workshops WADT (agora ADT) são uma fonte contínua de pesquisa atual em especificações algébricas e suas aplicações. Serve como exemplo do tipo de publicação onde avanços são discutidos.*

5.  SANNELLA, Donald; TARLECKI, Andrzej. **Foundations of Algebraic Specification and Formal Software Development.** Springer, 2012. (EATCS Monographs in Theoretical Computer Science) (ISBN 978-3642173356)
    *   *Resumo: Um livro mais moderno que fornece uma cobertura abrangente e rigorosa dos fundamentos da especificação algébrica e seu papel no desenvolvimento formal de software. Conecta a teoria clássica com desenvolvimentos mais recentes e aplicações práticas.*

6.  PAREIGIS, Bodo. **Categories and Functors.** Academic Press, 1970. (Pure and Applied Mathematics, Vol. 39) (ISBN 978-0125451505)
    *   *Resumo: Embora seja um livro sobre Teoria das Categorias, ele é incluído aqui porque os fundamentos das Σ-Álgebras estão profundamente conectados à Teoria das Categorias (álgebras para um funtor, por exemplo). Entender categorias pode fornecer uma perspectiva mais profunda sobre as estruturas algébricas, preparando para a Parte V do livro.*

---
