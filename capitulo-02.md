
---

# CAPÍTULO 2

# Especificação Algébrica de Tipos Abstratos de Dados

---

![imagem](tipo.png)

Após a introdução aos Tipos Abstratos de Dados (TADs) e sua importância para a abstração e corretude de software no capítulo anterior, este capítulo aprofunda-se na principal técnica formal que utilizaremos para sua definição: a Especificação Algébrica. Exploraremos os fundamentos da lógica equacional e das álgebras multissortidas, que fornecem o arcabouço matemático para esta abordagem. Serão introduzidos os conceitos de sorts, operações e assinaturas, seguidos pela definição de termos sobre uma assinatura, distinguindo entre variáveis e termos fundamentais. Avançaremos para a construção de especificações algébricas através da formulação de axiomas equacionais, culminando na definição de uma especificação como um par composto por uma assinatura e um conjunto de axiomas. Investigaremos a semântica dessas especificações por meio das Σ-álgebras, discutindo a propriedade de satisfatibilidade de equações por uma álgebra e o conceito de modelos, incluindo a importância dos modelos iniciais e finais. Diferenciaremos entre construtores, observadores e operações derivadas, elucidando o papel de cada um na construção e na observação do comportamento de um TAD. Finalmente, solidificaremos o aprendizado com exemplos fundamentais, detalhando as especificações algébricas para os TADs Boolean, Natural (segundo os axiomas de Peano) e Integer. Exercícios e suas resoluções serão apresentados ao longo do capítulo para reforçar a compreensão dos conceitos e técnicas matemáticas envolvidas.

---

## 2.1 Introdução à Lógica Equacional e Álgebras Multissortidas

A especificação algébrica de Tipos Abstratos de Dados (TADs) é uma abordagem formal que utiliza conceitos da álgebra universal e da lógica equacional para definir o comportamento dos dados de maneira abstrata e precisa. Nesta abordagem, um TAD é visto como uma (ou uma classe de) estrutura algébrica, onde os "tipos" de dados são interpretados como conjuntos (chamados *sorts* ou *portadores*), e as "operações" sobre os dados são interpretadas como funções entre esses conjuntos. As propriedades dessas operações e suas inter-relações são descritas por *axiomas*, que são tipicamente equações.

A beleza da especificação algébrica reside em sua capacidade de definir TADs sem fazer qualquer referência a uma representação concreta ou a um modelo de implementação. Ela foca exclusivamente no comportamento observável "de fora", ou seja, nas propriedades que devem ser válidas independentemente de como o TAD é implementado.

**Lógica Equacional**

A lógica equacional é um ramo da lógica matemática que lida com a igualdade. Ela fornece as regras formais para raciocinar sobre identidades (equações) entre termos.

**Álgebras Multissortidas**

Em muitas aplicações em Ciência da Computação, lidamos com diferentes "tipos" de objetos simultaneamente. Uma *álgebra multissortida* (ou heterogênea) generaliza o conceito de álgebra para lidar com múltiplos conjuntos subjacentes, chamados *sorts*.

> **Definição 2.1: Álgebra Multissortida (Intuitiva)**
>
> Uma **álgebra multissortida** consiste em uma família de conjuntos (os *sorts* ou *domínios portadores*), um para cada "tipo" de dado, e uma família de funções (as *operações*) definidas sobre esses conjuntos. Cada operação tem um perfil que especifica os sorts de seus argumentos e o sort de seu resultado.

**Sorts, Operações e Assinaturas (Σ-Signatures)**

O primeiro passo na especificação algébrica de um TAD é definir sua *assinatura*.

> **Definição 2.2: Assinatura Multissortida (Σ-Assinatura)**
>
> Uma **assinatura multissortida** Σ é um par $(S, F)$, onde:
> 1.  $S$ é um conjunto de **sorts**.
> 2.  $F$ é um conjunto de **símbolos de operação**. Para cada $f \in F$, é associado um *perfil* $s_1 \times s_2 \times \dots \times s_n \rightarrow s$. Se $n=0$, $f: \rightarrow s$ é uma **constante**.

**Termos sobre uma Assinatura (T<sub>Σ</sub>(X)). Variáveis e Termos Fundamentais**

Dada uma assinatura $\Sigma = (S, F)$ e um conjunto $S$-indexado de variáveis $X$, podemos construir *termos*.

> **Definição 2.3: Termos sobre uma Assinatura (T<sub>Σ</sub>(X))**
>
> Para cada sort $s \in S$, o conjunto de **termos de sort s sobre Σ com variáveis em X**, $T_{\Sigma}(X)_s$, é definido indutivamente:
> 1.  **Base (Variáveis):** Se $x \in X_s$, então $x \in T_{\Sigma}(X)_s$.
> 2.  **Base (Constantes):** Se $f: \rightarrow s$ é uma constante em $F$, então $f \in T_{\Sigma}(X)_s$.
> 3.  **Passo Indutivo (Aplicações de Operações):** Se $f: s_1 \times \dots \times s_n \rightarrow s$ ($n > 0$), e $t_i \in T_{\Sigma}(X)_{s_i}$, então $f(t_1, \dots, t_n) \in T_{\Sigma}(X)_s$.
>
> Termos sem variáveis são **termos fundamentais** ($T_{\Sigma,s}$).

**Substituições e Instanciação**

Uma *substituição* $\sigma: X \rightarrow T_{\Sigma}(Y)$ mapeia variáveis a termos. A aplicação de $\sigma$ a um termo $t \in T_{\Sigma}(X)$, $\sigma(t)$, é a *instanciação* de $t$.

> **Definição 2.4: Aplicação de Substituição (Instanciação)**
>
> A instância $\sigma(t)$ é definida indutivamente:
> 1.  Se $t = x$, $\sigma(t) = \sigma_s(x)$.
> 2.  Se $t = c$ (constante), $\sigma(t) = c$.
> 3.  Se $t = f(t_1, \dots, t_n)$, $\sigma(t) = f(\sigma(t_1), \dots, \sigma(t_n))$.

**Exercício 2.1:**
Dada a seguinte assinatura para um TAD SimpleList de inteiros (assuma que Int e Boolean são sorts pré-definidos):
$S = \{\text{List}, \text{Int}, \text{Boolean}\}$
$Ops = \{$
  nil: $\rightarrow$ List,
  cons: Int $\times$ List $\rightarrow$ List,
  isEmpty: List $\rightarrow$ Boolean,
  head: List $\rightarrow$ Int
$\}$
Seja $X_{\text{List}} = \{l\}$, $X_{\text{Int}} = \{i, j\}$.
Identifique quais das seguintes são termos válidos e, para os válidos, indique seu sort e se são termos fundamentais.
(a) cons(i, nil)
(b) head(nil)
(c) isEmpty(cons(i, l))
(d) cons(nil, i)
(e) head(cons(5, nil)) (assuma que 5 é uma constante do sort Int)

**Resolução 2.1:**
(a) cons(i, nil): **Válido.** Sort: List. Não é fundamental.
(b) head(nil): **Válido.** Sort: Int. É fundamental.
(c) isEmpty(cons(i, l)): **Válido.** Sort: Boolean. Não é fundamental.
(d) cons(nil, i): **Inválido.** (Tipos dos argumentos trocados para cons).
(e) head(cons(5, nil)): **Válido.** Sort: Int. É fundamental. ■

**Exercício 2.1.A:**
Considere a seguinte assinatura para um TAD ParOrdenado de Naturais:
$S = \{\text{Pair}, \text{Natural}\}$ (assuma Natural e suas operações como previamente definidos, incluindo constantes como 0, 1, 2...)
$Ops = \{$
  mkPair: Natural $\times$ Natural $\rightarrow$ Pair,
  first: Pair $\rightarrow$ Natural,
  second: Pair $\rightarrow$ Natural
$\}$
Sejam $X_{\text{Natural}} = \{n, m\}$, $X_{\text{Pair}} = \{p\}$.
Construa:
(a) Dois termos fundamentais distintos de sort Pair.
(b) Um termo não fundamental de sort Natural usando operações desta assinatura.
(c) Um termo inválido e explique porquê.

**Resolução 2.1.A:**
(a) Dois termos fundamentais distintos de sort Pair:
    1.  mkPair(0, 1)
    2.  mkPair(suc(zero), zero) (assumindo zero e suc de Natural) ou mkPair(1,0)
(b) Um termo não fundamental de sort Natural:
    1.  first(p) (onde p é uma variável de sort Pair)
    2.  second(mkPair(n, 0)) (onde n é uma variável de sort Natural)
(c) Um termo inválido e explicação:
    1.  mkPair(p, n): Inválido porque mkPair espera dois argumentos do sort Natural, mas p é do sort Pair.
    2.  first(n): Inválido porque first espera um argumento do sort Pair, mas n é do sort Natural. ■

**Exercício 2.1.B:**
Usando a assinatura do TAD SimpleList do Exercício 2.1 e as variáveis $X_{\text{List}} = \{l\}$, $X_{\text{Int}} = \{i, j\}$.
Considere o termo $t = $ cons(i, cons(j, l)).
Seja a substituição $\sigma: \{i \mapsto \text{0}, j \mapsto \text{suc(0)}, l \mapsto \text{nil}\}$.
Calcule $\sigma(t)$. O resultado é um termo fundamental?

**Resolução 2.1.B:**
$t = $ cons(i, cons(j, l))
$\sigma = \{i \mapsto \text{0}, j \mapsto \text{suc(0)}, l \mapsto \text{nil}\}$

$\sigma(t) = \text{cons}(\sigma(\text{i}), \sigma(\text{cons(j, l)}))$
$= \text{cons}(\text{0}, \text{cons}(\sigma(\text{j}), \sigma(\text{l})))$
$= \text{cons}(\text{0}, \text{cons}(\text{suc(0)}, \text{nil}))$

Sim, o resultado $\sigma(t) = $ cons(0, cons(suc(0), nil)) é um termo fundamental. ■

---

## 2.2 Axiomas Equacionais e Especificações Algébricas

Uma vez definida a assinatura $\Sigma$, a semântica das operações é especificada por *axiomas equacionais*.

**Definição de Axiomas como Equações entre Termos**

> **Definição 2.5: Axioma Equacional**
>
> Um **axioma equacional** é uma expressão $\forall V . (t_1 = t_2)$ (ou $t_1 = t_2$ com quantificação implícita), onde $V$ são variáveis e $t_1, t_2 \in T_{\Sigma}(V)_s$ para algum sort $s$.

**Uma Especificação Algébrica SP = (Σ, E) como um par Assinatura-Axiomas**

> **Definição 2.6: Especificação Algébrica**
>
> Uma **especificação algébrica** $SP$ é um par $(\Sigma, E)$, onde $\Sigma$ é uma assinatura e $E$ é um conjunto de axiomas equacionais sobre $\Sigma$.

**Consequência Lógica e Derivação**

$E \vdash t_1 = t_2$ indica que $t_1 = t_2$ pode ser derivada de $E$ usando as regras da lógica equacional (reflexividade, simetria, transitividade, congruência, substituição).

**Exercício 2.2.A:**
Para o TAD Natural (com operações zero, suc, add como definidas anteriormente), proponha axiomas equacionais para uma nova operação max: Natural $\times$ Natural $\rightarrow$ Natural, que retorna o maior entre dois números naturais. Defina por recursão no primeiro argumento.

**Resolução 2.2.A:**
Assinatura adicional:
max: Natural $\times$ Natural $\rightarrow$ Natural

Axiomas para max:
`For all n, m: Natural:`
*   (Axiom M1) max(zero, m) = m
*   (Axiom M2) max(suc(n), zero) = suc(n)
*   (Axiom M3) max(suc(n), suc(m)) = suc(max(n, m)) ■

**Exercício 2.2.B:**
Considere a especificação de Stack com os axiomas:
(Axiom S1) isEmpty(create) = true
(Axiom S2) isEmpty(push(s, x)) = false
(Axiom S3) pop(push(s, x)) = s
(Axiom S4) top(push(s, x)) = x
Para as seguintes equações, indique se elas são (i) uma instância direta de um axioma (e qual), (ii) uma consequência lógica que necessitaria de derivação (não precisa derivar), ou (iii) não relacionada/inválida.
(a) isEmpty(create) = true
(b) pop(push(pop(push(create, itemA)), itemB)) = pop(push(create, itemA))
(c) top(create) = itemErro
(d) isEmpty(push(s,x)) = not(isEmpty(create)) (assumindo not e o TAD Boolean)

**Resolução 2.2.B:**
(a) isEmpty(create) = true: (i) Instância direta do (Axiom S1).
(b) pop(push(pop(push(create, itemA)), itemB)) = pop(push(create, itemA)): (i) Instância direta do (Axiom S3), com $s = \text{pop(push(create, itemA))}$.
(c) top(create) = itemErro: (iii) Não relacionada/inválida.
(d) isEmpty(push(s,x)) = not(isEmpty(create)): (ii) Consequência lógica. (Ambos os lados avaliam para false). ■

---

## 2.3 Semântica das Especificações Algébricas: Σ-Álgebras

A semântica de $SP = (\Sigma, E)$ é dada em termos de **Σ-álgebras**.

**Definição de uma Σ-Álgebra: Domínios Portadores e Interpretação de Operações**

> **Definição 2.7: Σ-Álgebra**
>
> Uma **Σ-álgebra** $A$ consiste em:
> 1.  Para cada sort $s \in S$, um conjunto não vazio $A_s$ (domínio portador).
> 2.  Para cada $f: s_1 \times \dots \times s_n \rightarrow s$ em $F$, uma função $f_A: A_{s_1} \times \dots \times A_{s_n} \rightarrow A_s$.

**Satisfatibilidade de Equações em uma Álgebra e o Conceito de Modelo**

A avaliação de um termo $t$ em $A$ sob uma atribuição de variáveis $\alpha$ é $\text{eval}_A(t, \alpha)$.

> **Definição 2.8: Satisfatibilidade de uma Equação em uma Σ-Álgebra**
>
> Uma Σ-álgebra $A$ **satisfaz** $t_1 = t_2$ (a equação é **satisfatível** em $A$) se $\forall \alpha . (\text{eval}_A(t_1, \alpha) = \text{eval}_A(t_2, \alpha))$. Notação: $A \models t_1 = t_2$.

> **Definição 2.9: Modelo de uma Especificação Algébrica**
>
> $A$ é um **modelo** de $SP = (\Sigma, E)$ ($A \models SP$) se $A \models e$ para todo $e \in E$.

**A Classe de Todas as Σ-Álgebras que Satisfazem E. Modelos Iniciais e Finais.**

$Alg(SP) = \{ A \mid A \models SP \}$.
**Álgebra Inicial ($T_{SP}$):** "no junk", "no confusion".
**Álgebra Final ($Z_{SP}$):** Identifica o máximo possível.

**Exercício 2.2:** (Original, renumerado para contexto)
Considere a especificação $SP_{\text{NatPeano}} = (\Sigma_{\text{NatPeano}}, E_{\text{NatPeano}})$ onde $\Sigma_{\text{NatPeano}}$ tem sort Nat e operações zero: $\rightarrow$ Nat e suc: Nat $\rightarrow$ Nat. Suponha $E_{\text{NatPeano}} = \emptyset$. Descreva duas Σ-álgebras $A_1, A_2$ para $\Sigma_{\text{NatPeano}}$.

**Resolução 2.2:**
1.  **Álgebra $A_1$ (Padrão):** $A_{1, \text{Nat}} = \mathbb{N}_0$; $\text{zero}_{A_1}: 0$; $\text{suc}_{A_1}(n) = n+1$.
2.  **Álgebra $A_2$ (Módulo 2):** $A_{2, \text{Nat}} = \{ \mathbf{par}, \mathbf{ímpar} \}$; $\text{zero}_{A_2}: \mathbf{par}$; $\text{suc}_{A_2}(\mathbf{par}) = \mathbf{ímpar}$, $\text{suc}_{A_2}(\mathbf{ímpar}) = \mathbf{par}$. ■

**Exercício 2.3.A:**
Considere $\Sigma_{\text{Interruptor}}$ com sort `Estado` e ops `ligado`, `desligado`, `inverter`.
Axiomas $E_{\text{Interruptor}}$: (AxI1) inverter(ligado) = desligado; (AxI2) inverter(desligado) = ligado.
Proponha uma Σ-álgebra $A_{\text{quebrada}}$ para $\Sigma_{\text{Interruptor}}$ que *não* seja um modelo de $SP_{\text{Interruptor}}$ e explique qual axioma ela viola.

**Resolução 2.3.A:**
$A_{\text{quebrada}}$: $A_{\text{Estado}} = \{\text{ON}, \text{OFF}\}$. $\text{ligado}_{A}: \text{ON}$, $\text{desligado}_{A}: \text{OFF}$.
$\text{inverter}_{A}(\text{ON}) = \text{ON}$, $\text{inverter}_{A}(\text{OFF}) = \text{OFF}$.
Para (AxI1) inverter(ligado) = desligado:
LHS avalia para $\text{inverter}_{A}(\text{ON}) = \text{ON}$. RHS avalia para $\text{OFF}$. ON $\neq$ OFF. Viola (AxI1).
(Similarmente, viola (AxI2)). ■

**Exercício 2.3.B:**
Usando $A_2$ (Modelo Módulo 2) do Exercício 2.2.
Verifique se $A_2 \models E_1$, onde $E_1$: suc(suc(zero)) = zero.
Verifique se $A_2 \models E_2$, onde $E_2$: suc(n) = n.

**Resolução 2.3.B:**
Para $E_1$: suc(suc(zero)) = zero
LHS em $A_2$: $\text{suc}_{A_2}(\text{suc}_{A_2}(\mathbf{par})) = \text{suc}_{A_2}(\mathbf{ímpar}) = \mathbf{par}$.
RHS em $A_2$: $\mathbf{par}$.
Como $\mathbf{par} = \mathbf{par}$, $A_2 \models E_1$.

Para $E_2$: suc(n) = n
Seja $\alpha(n) = \mathbf{par}$.
LHS: $\text{suc}_{A_2}(\mathbf{par}) = \mathbf{ímpar}$. RHS: $\mathbf{par}$. $\mathbf{ímpar} \neq \mathbf{par}$.
$A_2 \not\models E_2$. ■

---

## 2.4 Construtores, Observadores e Operações Derivadas

Dentro da assinatura $\Sigma$ de um TAD, as operações podem ser classificadas com base em seu papel:
1.  **Construtores:** Usados para construir todos os valores do sort principal.
2.  **Observadores:** Retornam informações sobre um valor do TAD.
3.  **Operações Derivadas (ou Modificadores):** Outras operações.

**Exercício 2.4.A:**
Classifique as operações de Queue[Item]: newQ, enqueue, dequeue, front, isEmptyQ.

**Resolução 2.4.A:**
*   newQ: **Construtor.**
*   enqueue: **Construtor.**
*   dequeue: **Operação Derivada/Modificadora.**
*   front: **Observador.**
*   isEmptyQ: **Observador.** ■

---

## 2.5 Exemplos Fundamentais de Especificações Algébricas

Para solidificar os conceitos apresentados, vamos detalhar as especificações algébricas para três Tipos Abstratos de Dados fundamentais: Boolean, Natural (números naturais baseados nos axiomas de Peano) e Integer (números inteiros). Essas especificações não apenas ilustram a teoria, mas também servem como blocos de construção para TADs mais complexos que encontraremos nos capítulos subsequentes. Prestar atenção à escolha dos construtores e à forma como os axiomas definem outras operações em termos destes é crucial.

### Especificação de `Boolean`

O TAD Boolean é, talvez, o mais fundamental em qualquer sistema lógico ou computacional, representando os valores de verdade. Sua especificação algébrica é concisa e serve como um excelente primeiro exemplo.

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
> *   (Axiom B1) `not(true) = false`
> *   (Axiom B2) `not(false) = true`
> *   (Axiom B3) `and(true, b) = b`
> *   (Axiom B4) `and(false, b) = false`
> *   (Axiom B5) `and(b, true) = b` (*Adicionado para completeza de `and`*)
> *   (Axiom B6) `and(b, false) = false` (*Adicionado para completeza de `and`*)
> *   (Axiom B7) `or(true, b) = true`
> *   (Axiom B8) `or(false, b) = b`
> *   (Axiom B9) `or(b, true) = true` (*Adicionado para completeza de `or`*)
> *   (Axiom B10) `or(b, false) = b` (*Adicionado para completeza de `or`*)
> *   (Axiom B11) `eqB(true, true) = true`
> *   (Axiom B12) `eqB(false, false) = true`
> *   (Axiom B13) `eqB(true, false) = false`
> *   (Axiom B14) `eqB(false, true) = false`

**Comentários sobre os Axiomas de `Boolean`:**
As operações true e false são designadas como construtores. Elas representam os únicos valores atômicos do sort Boolean.
*   Os axiomas (Axiom B1) e (Axiom B2) definem a operação not por análise de casos sobre os dois construtores. not(true) resulta em false, e not(false) resulta em true, como esperado.
*   Os axiomas para and, (Axiom B3) a (Axiom B6), definem a conjunção. Originalmente, (Axiom B3) e (Axiom B4) definiam and com base no primeiro argumento ser true ou false. Adicionamos (Axiom B5) e (Axiom B6) para cobrir os casos em que o segundo argumento é fixo. Em um sistema de reescrita, apenas um conjunto (e.g., B3 e B4) seria tipicamente usado como regras, e a comutatividade (se desejada) seria um teorema ou uma regra adicional tratada com cuidado. No entanto, para uma especificação equacional completa, listar todas essas propriedades pode ser útil. Por exemplo, and(true, b) = b e and(b, true) = b.
*   Similarmente, os axiomas para or, (Axiom B7) a (Axiom B10), definem a disjunção. or(true, b) é sempre true, e or(false, b) é b.
*   Finalmente, os axiomas para eqB, (Axiom B11) a (Axiom B14), definem a igualdade entre booleanos. eqB(b1, b2) resulta em true se b1 e b2 são o mesmo construtor (ambos true ou ambos false), e false caso contrário.
É importante notar que, na semântica inicial, assume-se que true $\neq$ false, pois não há axioma que os iguale e eles são construtores distintos. As operações not, and, or, e eqB são totalmente definidas em termos dos construtores.

### Especificação de `Natural` (Números Naturais segundo Peano)

O TAD Natural, representando os números naturais $\{0, 1, 2, \dots\}$, é classicamente especificado usando os axiomas de Peano, que se traduzem elegantemente para uma especificação algébrica.

> **SPEC ADT `Natural`**
>
> **Sorts:**
> *   `Natural` - o sort dos números naturais.
> *   `Boolean` - (importado da especificação Boolean).
>
> **Signature (Σ<sub>Natural</sub>):**
> *   `zero:` $\rightarrow$ `Natural`
>     - O número natural zero. (Construtor)
> *   `suc:` `Natural` $\rightarrow$ `Natural`
>     - A função sucessor. (Construtor)
> *   `isZero:` `Natural` $\rightarrow$ `Boolean`
>     - Verifica se um natural é zero. (Observador)
> *   `add:` `Natural` $\times$ `Natural` $\rightarrow$ `Natural`
>     - A operação de adição. (Derivada)
> *   `mult:` `Natural` $\times$ `Natural` $\rightarrow$ `Natural`
>     - A operação de multiplicação. (Derivada)
> *   `eqN:` `Natural` $\times$ `Natural` $\rightarrow$ `Boolean`
>     - A operação de igualdade entre naturais. (Observador)
>
> **Axioms (E<sub>Natural</sub>):**
>
> `For all n, m: Natural:`
>
> *   (Axiom N1) `isZero(zero) = true`
> *   (Axiom N2) `isZero(suc(n)) = false`
>
> *   (Axiom N3) `add(zero, m) = m`
> *   (Axiom N4) `add(suc(n), m) = suc(add(n, m))`
>
> *   (Axiom N5) `mult(zero, m) = zero`
> *   (Axiom N6) `mult(suc(n), m) = add(m, mult(n, m))`
>
> *   (Axiom N7) `eqN(zero, zero) = true`
> *   (Axiom N8) `eqN(suc(n), zero) = false`
> *   (Axiom N9) `eqN(zero, suc(m)) = false`
> *   (Axiom N10) `eqN(suc(n), suc(m)) = eqN(n, m)`

**Comentários sobre os Axiomas de `Natural`:**
Os construtores são zero e suc. Qualquer número natural pode ser construído aplicando suc repetidamente a zero (e.g., 2 é suc(suc(zero))).
*   (Axiom N1) e (Axiom N2) definem o observador isZero. isZero(zero) é true. Para qualquer natural construído com suc (ou seja, qualquer natural não-zero), isZero(suc(n)) é false. Estes dois axiomas cobrem todos os casos possíveis de números naturais devido à natureza dos construtores.
*   (Axiom N3) e (Axiom N4) definem a adição recursivamente sobre a estrutura do primeiro argumento. (Axiom N3) é o caso base: adicionar zero a qualquer número m resulta em m. (Axiom N4) é o passo recursivo: adicionar o sucessor de n a m é o mesmo que tomar o sucessor da adição de n a m. Por exemplo, add(suc(n), m) = (n+1)+m = 1+(n+m) = suc(add(n,m)).
*   (Axiom N5) e (Axiom N6) definem a multiplicação, também recursivamente sobre o primeiro argumento. (Axiom N5) é o caso base: multiplicar zero por m resulta em zero. (Axiom N6) define a multiplicação por suc(n) em termos de adição e multiplicação por n: mult(suc(n), m) = (n+1)*m = m + n*m = add(m, mult(n,m)).
*   (Axiom N7) a (Axiom N10) definem a igualdade eqN. (Axiom N7) estabelece que zero é igual a si mesmo. (Axiom N8) e (Axiom N9) afirmam que zero não é igual a nenhum número construído com suc (e vice-versa), o que é uma propriedade fundamental de Peano (zero não é sucessor de nenhum número). (Axiom N10) define a igualdade de dois números construídos com suc recursivamente: suc(n) é igual a suc(m) se e somente se n é igual a m. Isso efetivamente "cancela" o suc de ambos os lados.

Esta especificação captura a essência indutiva dos números naturais.

### Especificação de `Integer` (Inteiros)

Especificar o TAD Integer é um pouco mais complexo do que Natural, pois precisamos representar números negativos. Uma abordagem comum usa três construtores: um para zero, um para construir inteiros positivos a partir de zero (sucessor), e um para construir inteiros negativos a partir de zero (predecessor).

> **SPEC ADT `Integer`**
>
> **Sorts:**
> *   `Integer` - o sort dos números inteiros.
> *   `Boolean` - (importado).
>
> **Signature (Σ<sub>Integer</sub>):**
> *   `zeroI:` $\rightarrow$ `Integer`
>     - O inteiro zero. (Construtor)
> *   `sucI:` `Integer` $\rightarrow$ `Integer`
>     - A função sucessor para inteiros (e.g., $i+1$). (Construtor)
> *   `predI:` `Integer` $\rightarrow$ `Integer`
>     - A função predecessor para inteiros (e.g., $i-1$). (Construtor)
> *   `addI:` `Integer` $\times$ `Integer` $\rightarrow$ `Integer`
>     - Adição de inteiros.
> *   `negI:` `Integer` $\rightarrow$ `Integer`
>     - Negação (inverso aditivo) de um inteiro.
> *   `eqI:` `Integer` $\times$ `Integer` $\rightarrow$ `Boolean`
>     - Igualdade entre inteiros.
>
> **Axioms (E<sub>Integer</sub>):**
>
> `For all i, j: Integer:`
>
> *   (Axiom I1) `sucI(predI(i)) = i`
> *   (Axiom I2) `predI(sucI(i)) = i`
>
> *   (Axiom I3) `addI(i, zeroI) = i`
> *   (Axiom I4) `addI(i, sucI(j)) = sucI(addI(i, j))`
> *   (Axiom I5) `addI(i, predI(j)) = predI(addI(i, j))`
>
> *   (Axiom I6) `negI(zeroI) = zeroI`
> *   (Axiom I7) `negI(sucI(i)) = predI(negI(i))`
> *   (Axiom I8) `negI(predI(i)) = sucI(negI(i))` (*Adicionado para simetria de `negI`*)
>
> *   (Axiom I9) `eqI(zeroI, zeroI) = true`
> *   (Axiom I10) `eqI(sucI(i), zeroI) = eqI(i, predI(zeroI))`
> *   (Axiom I11) `eqI(predI(i), zeroI) = eqI(i, sucI(zeroI))`
> *   (Axiom I12) `eqI(sucI(i), sucI(j)) = eqI(i, j)`
> *   (Axiom I13) `eqI(predI(i), predI(j)) = eqI(i, j)`
> *   (Axiom I14) `eqI(sucI(i), predI(j)) = eqI(i, predI(predI(j)))`
> *   (Axiom I15) `eqI(predI(i), sucI(j)) = eqI(i, sucI(sucI(j)))` (*Adicionado para simetria de `eqI`*)
>
> *(Opcional, se eqI for difícil de completar apenas com recursão):*
> *   (Axiom I16) `eqI(i,j) = if isZeroI(subI(i,j)) then true else false` (se `subI` e `isZeroI` forem definidas)

**Comentários sobre os Axiomas de `Integer`:**
Os construtores aqui são zeroI, sucI, e predI. A ideia é que qualquer inteiro pode ser alcançado a partir de zeroI aplicando uma sequência de sucI (para positivos) ou predI (para negativos).
*   Os axiomas (Axiom I1) e (Axiom I2) são cruciais: eles estabelecem que sucI e predI são inversos um do outro. sucI(predI(i)) = i significa que se você toma o predecessor e depois o sucessor, você volta ao inteiro original. Isso evita que, por exemplo, predI(zeroI) seja um "novo tipo" de -1, distinto do que seria obtido por outras combinações.
*   Os axiomas para addI, (Axiom I3) a (Axiom I5), definem a adição recursivamente sobre a estrutura do segundo argumento, usando seus construtores (zeroI, sucI(j), predI(j)). addI(i, zeroI) é i. Adicionar sucI(j) a i é o mesmo que tomar o sucessor de addI(i, j). Adicionar predI(j) a i é o mesmo que tomar o predecessor de addI(i, j).
*   Os axiomas para negI, (Axiom I6) a (Axiom I8), definem a negação. negI(zeroI) é zeroI. A negação de um sucessor, negI(sucI(i)), é o predecessor da negação de i (e.g., - (i+1) = (-i) - 1). Similarmente para negI(predI(i)).
*   Os axiomas para eqI, (Axiom I9) a (Axiom I15), tentam definir a igualdade. (Axiom I9) é o caso base. Os outros tentam reduzir a complexidade dos argumentos "cancelando" sucI ou predI, ou movendo-os. Definir a igualdade para inteiros desta forma puramente recursiva pode ser bastante complicado para garantir todas as propriedades desejadas (como terminação e confluência se usado como regras de reescrita). Por exemplo, (Axiom I10) eqI(sucI(i), zeroI) = eqI(i, predI(zeroI)) reduz o problema de comparar $i+1$ com $0$ a comparar $i$ com $-1$. Isso continua até que um dos argumentos seja zeroI. Os axiomas (Axiom I14) e (Axiom I15) lidam com comparações entre um sucessor e um predecessor. Uma especificação completa e robusta de eqI pode requerer uma abordagem mais sofisticada ou a introdução de outras operações como subtração (subI) e um teste de zero (isZeroI), e então definir eqI(i,j) como isZeroI(subI(i,j)).

A especificação de Integer ilustra que, à medida que os TADs se tornam mais complexos, garantir que os axiomas sejam consistentes, completos e capturem a semântica desejada torna-se um desafio maior.

**Exercício 2.3:** (Original, renumerado para contexto)
Usando os axiomas (Axiom N3) add(zero, m) = m e (Axiom N4) add(suc(n), m) = suc(add(n, m)) do TAD Natural, mostre a derivação de add(suc(suc(zero)), suc(zero)).

**Resolução 2.3:**
add(suc(suc(zero)), suc(zero))
$= \text{suc(add(suc(zero), suc(zero)))}$ (por (Axiom N4))
$= \text{suc(suc(add(zero, suc(zero))))}$ (por (Axiom N4))
$= \text{suc(suc(suc(zero)))}$ (por (Axiom N3)) ■

---

Este capítulo forneceu uma introdução detalhada à especificação algébrica de Tipos Abstratos de Dados. Cobrimos os blocos de construção fundamentais: sorts, assinaturas, termos, axiomas equacionais e o conceito de Σ-álgebras como modelos semânticos. A distinção entre construtores, observadores e operações derivadas foi discutida, e ilustramos a teoria com especificações para Boolean, Natural e Integer. A especificação algébrica nos oferece uma ferramenta poderosa para definir TADs com precisão matemática, focando em seu comportamento abstrato. No próximo capítulo, exploraremos como os Sistemas de Reescrita de Termos podem ser usados para dar uma semântica operacional a essas especificações, tratando os axiomas como regras de computação.

---
## REFERÊNCIAS BIBLIOGRÁFICAS

1.  BAADER, Franz; NIPKOW, Tobias. **Term Rewriting and All That.** Cambridge: Cambridge University Press, 1998.
    *   *Resumo: Um livro clássico e abrangente sobre sistemas de reescrita de termos, que são intimamente relacionados à semântica operacional de especificações algébricas. Cobre os fundamentos teóricos, incluindo confluência e terminação.*

2.  EHRIG, Hartmut; MAHR, Bernd. **Fundamentals of Algebraic Specification 1: Equations and Initial Semantics.** Berlin: Springer-Verlag, 1985. (EATCS Monographs on Theoretical Computer Science, Vol. 6).
    *   *Resumo: Este é um volume fundamental e um dos textos clássicos sobre especificação algébrica. Ele detalha a teoria das assinaturas, termos, axiomas, Σ-álgebras e foca extensivamente na semântica inicial e suas propriedades.*

3.  GRÄTZER, George. **Universal Algebra.** 2nd ed. New York: Springer-Verlag, 2008.
    *   *Resumo: Uma referência matemática abrangente sobre álgebra universal, que fornece o embasamento teórico para a especificação algébrica. Embora denso, é uma fonte autoritativa para conceitos como álgebras, homomorfismos, e construções como produtos e quocientes.*

4.  MESSEGUER, José; GOGUEN, Joseph A. Order-Sorted Algebra Solves the Constructor-Selector, Multiple Representation, and Coercion Problems. **Information and Computation**, v. 101, n. 1, p. 114-158, 1992.
    *   *Resumo: Este artigo de pesquisa aborda problemas comuns em especificações algébricas, como a interação entre construtores e seletores (observadores que "desconstroem" termos), e propõe a álgebra ordenada por sorts como uma solução elegante, estendendo a expressividade das especificações.*

5.  SANNELLA, Donald; TARLECKI, Andrzej. **Foundations of Algebraic Specification and Formal Software Development.** Berlin: Springer-Verlag, 2012. (EATCS Monographs in Theoretical Computer Science).
    *   *Resumo: Um texto moderno e abrangente que cobre os fundamentos da especificação algébrica e seu papel no desenvolvimento formal de software. Discute teoria de instituições, estruturação de especificações e refinamento.*

6.  WIRSING, Martin. Algebraic Specification. In: VAN LEEUWEN, Jan (Ed.). **Handbook of Theoretical Computer Science, Volume B: Formal Models and Semantics.** Amsterdam: Elsevier e Cambridge, MA: MIT Press, 1990. p. 675-788.
    *   *Resumo: Um capítulo de manual altamente influente que fornece uma visão geral concisa e profunda da especificação algébrica, cobrindo seus principais conceitos, técnicas, variações (como especificações parciais e de ordem superior) e aplicações.*

---
