
---

# CAPÍTULO 2

# Especificação Algébrica de Tipos Abstratos de Dados

---

![imagem](tipo.png)

Após a introdução aos Tipos Abstratos de Dados (TADs) e sua importância para a abstração e corretude de software no capítulo anterior, este capítulo aprofunda-se na principal técnica formal que utilizaremos para sua definição: a Especificação Algébrica. Exploraremos os fundamentos da lógica equacional e das álgebras multissortidas, que fornecem o arcabouço matemático para esta abordagem. Serão introduzidos os conceitos de sorts, operações e assinaturas, seguidos pela definição de termos sobre uma assinatura, distinguindo entre variáveis e termos fundamentais. Avançaremos para a construção de especificações algébricas através da formulação de axiomas equacionais, culminando na definição de uma especificação como um par composto por uma assinatura e um conjunto de axiomas. Investigaremos a semântica dessas especificações por meio das Σ-álgebras, discutindo a propriedade de satisfatibilidade de equações por uma álgebra e o conceito de modelos, incluindo a importância dos modelos iniciais e finais. Diferenciaremos entre construtores, observadores e operações derivadas, elucidando o papel de cada um na construção e na observação do comportamento de um TAD. Finalmente, solidificaremos o aprendizado com exemplos fundamentais, detalhando as especificações algébricas para os TADs `Boolean`, `Natural` (segundo os axiomas de Peano) e `Integer`. Exercícios e suas resoluções serão apresentados ao longo do capítulo para reforçar a compreensão dos conceitos e técnicas matemáticas envolvidas.

---

## 2.1 Introdução à Lógica Equacional e Álgebras Multissortidas

A especificação algébrica de Tipos Abstratos de Dados (TADs) é uma abordagem formal que utiliza conceitos da álgebra universal e da lógica equacional para definir o comportamento dos dados de maneira abstrata e precisa. Nesta abordagem, um TAD é visto como uma (ou uma classe de) estrutura algébrica, onde os "tipos" de dados são interpretados como conjuntos (chamados *sorts* ou *portadores*), e as "operações" sobre os dados são interpretadas como funções entre esses conjuntos. As propriedades dessas operações e suas inter-relações são descritas por *axiomas*, que são tipicamente equações.

A beleza da especificação algébrica reside em sua capacidade de definir TADs sem fazer qualquer referência a uma representação concreta ou a um modelo de implementação. Ela foca exclusivamente no comportamento observável "de fora", ou seja, nas propriedades que devem ser válidas independentemente de como o TAD é implementado.

**Lógica Equacional**

A lógica equacional é um ramo da lógica matemática que lida com a igualdade. Ela fornece as regras formais para raciocinar sobre identidades (equações) entre termos. Um sistema de lógica equacional consiste em:

1.  Um conjunto de *símbolos de função* (operações), cada um com uma aridade (número de argumentos) associada.
2.  Um conjunto de *variáveis*.
3.  Regras para formar *termos* a partir de símbolos de função e variáveis.
4.  Um conjunto de *axiomas equacionais*, que são pares de termos declarados como iguais.
5.  Um conjunto de *regras de inferência* que permitem derivar novas equações a partir dos axiomas. As regras padrão incluem reflexividade ($t = t$), simetria (se $t_1 = t_2$, então $t_2 = t_1$), transitividade (se $t_1 = t_2$ e $t_2 = t_3$, então $t_1 = t_3$), e a regra de substituição (se $t_1 = t_2$, então $\sigma(t_1) = \sigma(t_2)$ para qualquer substituição $\sigma$) e a regra de congruência (se $t_1 = u_1, \dots, t_n = u_n$, então $f(t_1, \dots, t_n) = f(u_1, \dots, u_n)$ para qualquer símbolo de função $f$ de aridade $n$).

**Álgebras Multissortidas**

Em muitas aplicações em Ciência da Computação, lidamos com diferentes "tipos" de objetos simultaneamente. Por exemplo, em uma pilha de inteiros, temos o tipo "Pilha" e o tipo "Inteiro". Uma *álgebra multissortida* (ou heterogênea) generaliza o conceito de álgebra (que opera sobre um único conjunto) para lidar com múltiplos conjuntos subjacentes, chamados *sorts*.

> **Definição 2.1: Álgebra Multissortida (Intuitiva)**
>
> Uma **álgebra multissortida** consiste em uma família de conjuntos (os *sorts* ou *domínios portadores*), um para cada "tipo" de dado, e uma família de funções (as *operações*) definidas sobre esses conjuntos. Cada operação tem um perfil que especifica os sorts de seus argumentos e o sort de seu resultado.

A combinação da lógica equacional com o formalismo de álgebras multissortidas fornece a base para a especificação algébrica de TADs.

**Sorts, Operações e Assinaturas (Σ-Signatures)**

O primeiro passo na especificação algébrica de um TAD é definir sua *assinatura*. Uma assinatura estabelece o vocabulário do TAD: os nomes dos tipos de dados envolvidos e os nomes e perfis das operações.

> **Definição 2.2: Assinatura Multissortida (Σ-Assinatura)**
>
> Uma **assinatura multissortida** Σ é um par $(S, F)$, onde:
> 1.  $S$ é um conjunto de **sorts** (nomes de tipos).
> 2.  $F$ é um conjunto de **símbolos de operação** (ou nomes de função). Para cada $f \in F$, é associado um *perfil* (ou *tipo da operação*) da forma $s_1 \times s_2 \times \dots \times s_n \rightarrow s$, onde $s_1, \dots, s_n \in S$ são os *sorts dos argumentos* (domínio) e $s \in S$ é o *sort do resultado* (contradomínio). O número $n \ge 0$ é a *aridade* da operação. Se $n=0$, a operação $f: \rightarrow s$ é chamada de **constante** do sort $s$.

**Notação:**
Escrevemos $f: s_1 \times \dots \times s_n \rightarrow s$ para denotar uma operação $f \in F$ com o perfil indicado. O conjunto $S$ de sorts é frequentemente denotado por $Sorts(\Sigma)$ e o conjunto $F$ de operações por $Ops(\Sigma)$. Uma assinatura é muitas vezes referida simplesmente como $\Sigma$.

**Exemplo:**
Considere um TAD simples para Números Naturais com zero e sucessor.
$S = \{\text{Nat}\}$
$F = \{ \text{zero}: \rightarrow \text{Nat}, \text{suc}: \text{Nat} \rightarrow \text{Nat} \}$
Aqui, `Nat` é o único sort. `zero` é uma constante do sort `Nat`, e `suc` é uma operação unária que toma um `Nat` e retorna um `Nat`.
A assinatura completa seria $\Sigma_{\text{NatPeano}} = ( \{\text{Nat}\}, \{\text{zero}: \rightarrow \text{Nat}, \text{suc}: \text{Nat} \rightarrow \text{Nat} \} )$.

Na prática, quando especificamos um TAD, primeiro listamos os sorts e depois as operações da assinatura, indicando claramente o perfil de cada operação, como fizemos no esboço do TAD `Stack[Item]` no Capítulo 1. A assinatura define a sintaxe das operações; a semântica será dada pelos axiomas.

**Termos sobre uma Assinatura (T<sub>Σ</sub>(X)). Variáveis e Termos Fundamentais**

Dada uma assinatura $\Sigma = (S, F)$, podemos construir *termos* que representam expressões ou "cálculos" usando as operações de $\Sigma$. Se quisermos termos com variáveis, precisamos de um conjunto $S$-indexado de variáveis, $X = \{X_s\}_{s \in S}$, onde $X_s$ é um conjunto de variáveis do sort $s$. Assumimos que $X_s \cap F = \emptyset$ para todo $s$.

> **Definição 2.3: Termos sobre uma Assinatura (T<sub>Σ</sub>(X))**
>
> Seja $\Sigma = (S, F)$ uma assinatura e $X = \{X_s\}_{s \in S}$ uma família $S$-indexada de conjuntos de variáveis. Para cada sort $s \in S$, o conjunto de **termos de sort s sobre Σ com variáveis em X**, denotado por $T_{\Sigma}(X)_s$, é definido indutivamente da seguinte forma:
> 1.  **Base (Variáveis):** Se $x \in X_s$, então $x \in T_{\Sigma}(X)_s$. (Toda variável de sort $s$ é um termo de sort $s$).
> 2.  **Base (Constantes):** Se $f: \rightarrow s$ é uma constante em $F$ (i.e., $f \in Ops(\Sigma)$ com aridade 0 e sort de resultado $s$), então $f \in T_{\Sigma}(X)_s$. (Toda constante de sort $s$ é um termo de sort $s$).
> 3.  **Passo Indutivo (Aplicações de Operações):** Se $f: s_1 \times \dots \times s_n \rightarrow s$ é uma operação em $F$ (com $n > 0$), e $t_1 \in T_{\Sigma}(X)_{s_1}, \dots, t_n \in T_{\Sigma}(X)_{s_n}$ são termos dos sorts correspondentes, então a expressão $f(t_1, \dots, t_n)$ é um termo de sort $s$, i.e., $f(t_1, \dots, t_n) \in T_{\Sigma}(X)_s$.
>
> O conjunto de todos os termos sobre $\Sigma$ com variáveis em $X$ é $T_{\Sigma}(X) = \bigcup_{s \in S} T_{\Sigma}(X)_s$.

Se o conjunto de variáveis $X$ é vazio ($X_s = \emptyset$ para todo $s$), os termos resultantes são chamados de **termos fundamentais** (ou *ground terms*). O conjunto de termos fundamentais de sort $s$ é denotado por $T_{\Sigma,s}$ ou $T_{\Sigma}(\emptyset)_s$. Termos fundamentais são aqueles construídos apenas a partir de constantes e outras operações, sem variáveis. Eles representam os "valores concretos" que podem ser construídos no TAD.

**Exemplo (continuando $\Sigma_{\text{NatPeano}}$):**
Seja $X_{\text{Nat}} = \{n, m\}$ um conjunto de variáveis do sort `Nat`.
*   `zero` é um termo fundamental de sort `Nat`.
*   `n` é um termo (não fundamental) de sort `Nat`.
*   `suc(zero)` é um termo fundamental de sort `Nat`.
*   `suc(n)` é um termo (não fundamental) de sort `Nat`.
*   `suc(suc(zero))` é um termo fundamental de sort `Nat`.
*   `suc(suc(m))` é um termo (não fundamental) de sort `Nat`.

Expressões como `suc(zero, n)` não seriam termos válidos, pois `suc` tem aridade 1, e `zero(n)` não seria válido, pois `zero` é uma constante (aridade 0). A formação de termos deve respeitar os perfis das operações na assinatura.

**Substituições e Instanciação**

Uma *substituição* é uma função que mapeia variáveis a termos dos mesmos sorts. Dada uma assinatura $\Sigma$ e uma família $S$-indexada de variáveis $X$, uma substituição $\sigma: X \rightarrow T_{\Sigma}(Y)$ é uma família de mapeamentos $\{\sigma_s: X_s \rightarrow T_{\Sigma}(Y)_s\}_{s \in S}$, onde $Y$ é outra (possivelmente a mesma) família de variáveis.

A aplicação de uma substituição $\sigma$ a um termo $t \in T_{\Sigma}(X)$, denotada por $\sigma(t)$ ou $t\sigma$, resulta em um novo termo onde cada ocorrência de uma variável $x$ em $t$ é substituída pelo termo $\sigma(x)$. Isso é chamado de *instanciação*.

> **Definição 2.4: Aplicação de Substituição (Instanciação)**
>
> Seja $t \in T_{\Sigma}(X)$ um termo e $\sigma: X \rightarrow T_{\Sigma}(Y)$ uma substituição. A instância de $t$ por $\sigma$, denotada $\sigma(t)$, é definida indutivamente:
> 1.  Se $t = x$ para $x \in X_s$, então $\sigma(t) = \sigma_s(x)$.
> 2.  Se $t = c$ para uma constante $c: \rightarrow s$, então $\sigma(t) = c$.
> 3.  Se $t = f(t_1, \dots, t_n)$ para $f: s_1 \times \dots \times s_n \rightarrow s$, então $\sigma(t) = f(\sigma(t_1), \dots, \sigma(t_n))$.

**Exemplo:**
Considere $\Sigma_{\text{NatPeano}}$ e $X_{\text{Nat}} = \{n\}$. Seja o termo $t = \text{suc(suc(n))}$.
Seja a substituição $\sigma: \{n\} \rightarrow T_{\Sigma_{\text{NatPeano}}}$ definida por $\sigma(n) = \text{zero}$.
Então, $\sigma(t) = \text{suc(suc(zero))}$.

Seja a substituição $\theta: \{n\} \rightarrow T_{\Sigma_{\text{NatPeano}}}(\{m\})$ definida por $\theta(n) = \text{suc(m)}$.
Então, $\theta(t) = \text{suc(suc(suc(m)))}$.

As substituições são fundamentais para definir a semântica das equações que envolvem variáveis, como veremos a seguir. Uma equação com variáveis é entendida como uma afirmação universal sobre todas as possíveis instanciações dessas variáveis por termos fundamentais (ou termos em geral, dependendo da semântica escolhida).

**Exercício 2.1:**
Dada a seguinte assinatura para um TAD `SimpleList` de inteiros (assuma que `Int` e `Boolean` são sorts pré-definidos):
$S = \{\text{List}, \text{Int}, \text{Boolean}\}$
$Ops = \{$
  `nil: $\rightarrow$ List`,
  `cons: Int $\times$ List $\rightarrow$ List`,
  `isEmpty: List $\rightarrow$ Boolean`,
  `head: List $\rightarrow$ Int`
$\}$
Seja $X_{\text{List}} = \{l\}$, $X_{\text{Int}} = \{i, j\}$.
Identifique quais das seguintes são termos válidos e, para os válidos, indique seu sort e se são termos fundamentais.
(a) `cons(i, nil)`
(b) `head(nil)`
(c) `isEmpty(cons(i, l))`
(d) `cons(nil, i)`
(e) `head(cons(5, nil))` (assuma que `5` é uma constante do sort `Int`)

**Resolução 2.1:**
(a) `cons(i, nil)`:
    *   `i` é uma variável de sort `Int`.
    *   `nil` é um termo fundamental de sort `List`.
    *   `cons` espera `Int $\times$ List`. Os argumentos correspondem.
    *   **Válido.** Sort: `List`. Não é fundamental (contém a variável `i`).
(b) `head(nil)`:
    *   `nil` é um termo fundamental de sort `List`.
    *   `head` espera `List`. O argumento corresponde.
    *   **Válido.** Sort: `Int`. É fundamental (não contém variáveis). (Nota: este termo pode representar uma condição de erro se a semântica de `head` não for definida para listas vazias, mas sintaticamente é um termo válido).
(c) `isEmpty(cons(i, l))`:
    *   `i` é uma variável de sort `Int`.
    *   `l` é uma variável de sort `List`.
    *   `cons(i, l)` é um termo de sort `List`.
    *   `isEmpty` espera `List`. O argumento corresponde.
    *   **Válido.** Sort: `Boolean`. Não é fundamental (contém `i` e `l`).
(d) `cons(nil, i)`:
    *   `nil` é um termo de sort `List`.
    *   `i` é um termo de sort `Int`.
    *   `cons` espera `Int $\times$ List`. Os sorts dos argumentos estão trocados.
    *   **Inválido.**
(e) `head(cons(5, nil))`:
    *   Assumindo `5` é uma constante (termo fundamental) de sort `Int`.
    *   `nil` é um termo fundamental de sort `List`.
    *   `cons(5, nil)` é um termo fundamental de sort `List`.
    *   `head` espera `List`. O argumento corresponde.
    *   **Válido.** Sort: `Int`. É fundamental. ■

---

## 2.2 Axiomas Equacionais e Especificações Algébricas

Uma vez definida a assinatura $\Sigma$ de um TAD, que estabelece os sorts e as operações, o próximo passo é especificar a *semântica* dessas operações, ou seja, seu comportamento e como elas se relacionam. Na especificação algébrica, isso é feito através de um conjunto de *axiomas*, que são tipicamente equações entre termos.

**Definição de Axiomas como Equações entre Termos**

Um axioma equacional sobre uma assinatura $\Sigma$ e um conjunto de variáveis $X$ é uma fórmula da forma $t_1 = t_2$, onde $t_1$ e $t_2$ são termos do mesmo sort em $T_{\Sigma}(X)$. Ou seja, $t_1, t_2 \in T_{\Sigma}(X)_s$ para algum sort $s \in S$.

> **Definição 2.5: Axioma Equacional**
>
> Dado uma assinatura $\Sigma = (S,F)$ e uma família $S$-indexada de variáveis $X$, um **axioma equacional** (ou simplesmente **equação**) é uma expressão da forma:
> $\forall V . (t_1 = t_2)$
> onde $V \subseteq X$ é o conjunto de variáveis que ocorrem em $t_1$ ou $t_2$ (as variáveis quantificadas universalmente), e $t_1, t_2 \in T_{\Sigma}(V)_s$ para algum $s \in S$.
> Por convenção, o quantificador universal $\forall V$ é frequentemente omitido e entendido implicitamente. Assim, um axioma é escrito simplesmente como $t_1 = t_2$.

Essa equação postula que, para quaisquer valores (termos fundamentais) que as variáveis em $V$ possam assumir, os termos $t_1$ e $t_2$, quando instanciados com esses valores, devem ser "iguais" ou "equivalentes" no contexto do TAD. O significado preciso de "igualdade" será dado pela semântica das Σ-álgebras (Seção 2.3).

**Exemplo (TAD `Stack[Item]`):**
Assinatura $\Sigma_{\text{Stack}}$ (como no Capítulo 1).
Variáveis: $s: \text{Stack}$, $x: \text{Item}$.
Axiomas (equações):
1.  `isEmpty(create) = true`
2.  `isEmpty(push(s, x)) = false`
3.  `pop(push(s, x)) = s`
4.  `top(push(s, x)) = x`

Cada uma dessas equações estabelece uma propriedade fundamental do TAD `Stack`. Por exemplo, o `(Axioma 3)` `pop(push(s, x)) = s` afirma que desempilhar após empilhar um item $x$ em uma pilha $s$ resulta na pilha original $s$.

**Uma Especificação Algébrica SP = (Σ, E) como um par Assinatura-Axiomas**

Com os conceitos de assinatura e axiomas equacionais em mãos, podemos agora definir formalmente o que é uma especificação algébrica.

> **Definição 2.6: Especificação Algébrica**
>
> Uma **especificação algébrica** $SP$ é um par $(\Sigma, E)$, onde:
> 1.  $\Sigma = (S, F)$ é uma assinatura multissortida.
> 2.  $E$ é um conjunto de axiomas equacionais sobre $\Sigma$ (e um conjunto implícito de variáveis $X$ cujos sorts estão em $S$). Cada axioma em $E$ é da forma $t_1 = t_2$, onde $t_1$ e $t_2$ são termos do mesmo sort em $T_{\Sigma}(X)$.

A especificação $SP = (\Sigma, E)$ define um Tipo Abstrato de Dado. A assinatura $\Sigma$ define a "sintaxe" do TAD (seus tipos e operações), enquanto o conjunto de axiomas $E$ define sua "semântica" (o comportamento e as propriedades dessas operações).

**Exemplo (Especificação Algébrica Completa para `Boolean`):**
> **SPEC ADT `Boolean`**
>
> **Sorts:** `Boolean`
>
> **Signature (Σ<sub>Boolean</sub>):**
> *   `true:` $\rightarrow$ `Boolean`
> *   `false:` $\rightarrow$ `Boolean`
> *   `not:` `Boolean` $\rightarrow$ `Boolean`
> *   `and:` `Boolean` $\times$ `Boolean` $\rightarrow$ `Boolean`
>
> **Axioms (E<sub>Boolean</sub>):**
>
> `For all b: Boolean:`
>
> *   (Axiom B1) `not(true) = false`
> *   (Axiom B2) `not(false) = true`
> *   (Axiom B3) `and(true, b) = b`
> *   (Axiom B4) `and(false, b) = false`

**Consequência Lógica e Derivação**

Dado uma especificação algébrica $SP = (\Sigma, E)$, o conjunto de axiomas $E$ postula certas igualdades como verdadeiras. A lógica equacional fornece um sistema de dedução para derivar novas equações que são *consequências lógicas* dos axiomas em $E$. Denotamos $E \vdash t_1 = t_2$ para indicar que a equação $t_1 = t_2$ pode ser derivada de $E$ usando as regras de inferência da lógica equacional.

As regras de inferência básicas para a lógica equacional são:

1.  **Reflexividade:** Para qualquer termo $t \in T_{\Sigma}(X)$, $E \vdash t = t$.
2.  **Simetria:** Se $E \vdash t_1 = t_2$, então $E \vdash t_2 = t_1$.
3.  **Transitividade:** Se $E \vdash t_1 = t_2$ e $E \vdash t_2 = t_3$, então $E \vdash t_1 = t_3$.
4.  **Congruência:** Para uma operação $f: s_1 \times \dots \times s_n \rightarrow s$ em $\Sigma$, se $E \vdash t_1 = u_1, \dots, E \vdash t_n = u_n$, então $E \vdash f(t_1, \dots, t_n) = f(u_1, \dots, u_n)$.
5.  **Substituição (ou Instanciação):** Se $E \vdash t_1 = t_2$, e $\sigma: X \rightarrow T_{\Sigma}(Y)$ é uma substituição, então $E \vdash \sigma(t_1) = \sigma(t_2)$.

Uma equação $t_1 = t_2$ é uma *consequência lógica* de $E$ se puder ser provada a partir dos axiomas em $E$ através de um número finito de aplicações das regras de inferência.

**Exemplo de Derivação:**
Usando a especificação $SP_{\text{Boolean}}$ e seus axiomas:
Queremos provar `not(not(b)) = b` para `b: Boolean`.

1.  `not(true) = false` (Axiom B1)
2.  `not(false) = true` (Axiom B2)

   Prova para `not(not(true)) = true`:
   `not(not(true))`
   $= \text{not(false)}$ (por (1) e congruência)
   $= \text{true}$ (por (2))
   Logo, `not(not(true)) = true`. (Equação 2.1)

   Prova para `not(not(false)) = false`:
   `not(not(false))`
   $= \text{not(true)}$ (por (2) e congruência)
   $= \text{false}$ (por (1))
   Logo, `not(not(false)) = false`. (Equação 2.2)

Como `b` pode ser instanciado por `true` ou `false`, e a propriedade `not(not(b)) = b` vale para ambas as instanciações (conforme (Equação 2.1) e (Equação 2.2)), podemos concluir que $E_{\text{Boolean}} \vdash \text{not(not(b))} = b$.

---

## 2.3 Semântica das Especificações Algébricas: Σ-Álgebras

Até agora, definimos a sintaxe de uma especificação algébrica $SP = (\Sigma, E)$. Agora, precisamos definir sua *semântica*, ou seja, o que ela "significa". A semântica de uma especificação algébrica é dada em termos de **Σ-álgebras**, que são as estruturas matemáticas que podem servir como "modelos" para a especificação.

**Definição de uma Σ-Álgebra: Domínios Portadores e Interpretação de Operações**

Dada uma assinatura $\Sigma = (S, F)$, uma Σ-álgebra fornece uma interpretação concreta para os sorts e os símbolos de operação em $\Sigma$.

> **Definição 2.7: Σ-Álgebra**
>
> Seja $\Sigma = (S, F)$ uma assinatura multissortida. Uma **Σ-álgebra** $A$ consiste em:
> 1.  Para cada sort $s \in S$, um conjunto não vazio $A_s$, chamado **domínio portador** do sort $s$ em $A$.
> 2.  Para cada símbolo de operação $f: s_1 \times \dots \times s_n \rightarrow s$ em $F$ (com $n \ge 0$), uma função (concreta) $f_A: A_{s_1} \times \dots \times A_{s_n} \rightarrow A_s$.
>     *   Se $f: \rightarrow s$ é uma constante ($n=0$), então $f_A$ é um elemento específico de $A_s$.

**Exemplo de Σ-Álgebra para $\Sigma_{\text{Boolean}}$:**
Seja $\Sigma_{\text{Boolean}} = (S_B, F_B)$ onde $S_B = \{\text{Boolean}\}$ e $F_B = \{\text{true}, \text{false}, \text{not}, \text{and}\}$.
Podemos definir uma Σ-álgebra $A_{\text{std_bool}}$ da seguinte forma:
1.  Domínio portador: $A_{\text{Boolean}} = \{\mathbf{T}, \mathbf{F}\}$.
2.  Interpretação das operações:
    *   $\text{true}_{A}: \mathbf{T}$
    *   $\text{false}_{A}: \mathbf{F}$
    *   $\text{not}_{A}(\mathbf{T}) = \mathbf{F}$, $\text{not}_{A}(\mathbf{F}) = \mathbf{T}$
    *   $\text{and}_{A}(\mathbf{T}, \mathbf{T}) = \mathbf{T}$, $\text{and}_{A}(\mathbf{T}, \mathbf{F}) = \mathbf{F}$, etc.

**Satisfatibilidade de Equações em uma Álgebra e o Conceito de Modelo**

Dada uma Σ-álgebra $A$, podemos avaliar termos $T_{\Sigma}(X)$ nela. Para isso, precisamos de uma *atribuição de variáveis* $\alpha: X \rightarrow A$. A *avaliação* de um termo $t \in T_{\Sigma}(X)_s$ em $A$ sob $\alpha$, denotada $\text{eval}_A(t, \alpha)$, é um valor em $A_s$.
Agora podemos definir quando uma Σ-álgebra possui a propriedade de satisfatibilidade para um axioma equacional (ou, de forma mais simples, quando ela *satisfaz* um axioma equacional).

> **Definição 2.8: Satisfatibilidade de uma Equação em uma Σ-Álgebra**
>
> Uma Σ-álgebra $A$ **satisfaz** uma equação $t_1 = t_2$ (ou, diz-se que a equação é **satisfatível** em $A$) se, para *toda* atribuição de variáveis $\alpha: X \rightarrow A$, a avaliação de $t_1$ em $A$ sob $\alpha$ é igual à avaliação de $t_2$ em $A$ sob $\alpha$. Ou seja:
> $A \models t_1 = t_2 \iff \forall \alpha: X \rightarrow A . (\text{eval}_A(t_1, \alpha) = \text{eval}_A(t_2, \alpha))$
> Se a equação não contém variáveis (i.e., $t_1, t_2$ são termos fundamentais), então ela é satisfeita em $A$ se $\text{eval}_A(t_1) = \text{eval}_A(t_2)$.

Uma Σ-álgebra $A$ é um **modelo** de uma especificação algébrica $SP = (\Sigma, E)$ se $A$ satisfaz *todos* os axiomas em $E$.

> **Definição 2.9: Modelo de uma Especificação Algébrica**
>
> Uma Σ-álgebra $A$ é um **modelo** de uma especificação $SP = (\Sigma, E)$, denotado $A \models SP$, se $A \models e$ para todo axioma $e \in E$.

**Exemplo (Verificação de Satisfatibilidade):**
Considere $A_{\text{std_bool}}$ e o axioma `(Axiom B3) and(true, b) = b`.
Para $\alpha_1(b) = \mathbf{T}$: $\text{eval}_{A_{\text{std_bool}}}(\text{and(true, b)}, \alpha_1) = \mathbf{T}$ e $\text{eval}_{A_{\text{std_bool}}}(\text{b}, \alpha_1) = \mathbf{T}$. Satisfeito.
Para $\alpha_2(b) = \mathbf{F}$: $\text{eval}_{A_{\text{std_bool}}}(\text{and(true, b)}, \alpha_2) = \mathbf{F}$ e $\text{eval}_{A_{\text{std_bool}}}(\text{b}, \alpha_2) = \mathbf{F}$. Satisfeito.
Logo, $A_{\text{std_bool}}$ satisfaz `and(true, b) = b`.

**A Classe de Todas as Σ-Álgebras que Satisfazem E. Modelos Iniciais e Finais.**

Para uma dada especificação $SP = (\Sigma, E)$, a classe de todas as Σ-álgebras que são modelos de $SP$ é denotada por $Alg(SP)$.
Dentro de $Alg(SP)$, destacam-se:

1.  **Álgebra Inicial ($T_{SP}$ ou $I_{SP}$):**
    *   Propriedades "no junk" (sem lixo) e "no confusion" (sem identificações desnecessárias).
    *   $T_{SP} \models t_1 = t_2 \iff E \vdash t_1 = t_2$ para termos fundamentais $t_1, t_2$.
    *   É o modelo mais abstrato e canônico, frequentemente construído como $T_{\Sigma} / \equiv_E$.

2.  **Álgebra Final ($Z_{SP}$):**
    *   Tende a identificar o máximo de termos possível sem violar distinções observáveis.

Este livro geralmente assumirá uma semântica baseada em modelos iniciais.

**Exercício 2.2:**
Considere a especificação $SP_{\text{NatPeano}} = (\Sigma_{\text{NatPeano}}, E_{\text{NatPeano}})$ onde $\Sigma_{\text{NatPeano}}$ tem sort `Nat` e operações `zero: $\rightarrow$ Nat` e `suc: Nat $\rightarrow$ Nat`. Suponha que o conjunto de axiomas $E_{\text{NatPeano}}$ está vazio.
Descreva duas Σ-álgebras diferentes, $A_1$ e $A_2$, para $\Sigma_{\text{NatPeano}}$.

**Resolução 2.2:**
1.  **Álgebra $A_1$ (Modelo Padrão dos Naturais):**
    *   $A_{1, \text{Nat}} = \mathbb{N}_0 = \{0, 1, 2, \dots\}$.
    *   $\text{zero}_{A_1}: 0$.
    *   $\text{suc}_{A_1}(n) = n+1$.

2.  **Álgebra $A_2$ (Modelo Módulo 2):**
    *   $A_{2, \text{Nat}} = \mathbb{Z}_2 = \{ \mathbf{par}, \mathbf{ímpar} \}$.
    *   $\text{zero}_{A_2}: \mathbf{par}$.
    *   $\text{suc}_{A_2}(\mathbf{par}) = \mathbf{ímpar}$, $\text{suc}_{A_2}(\mathbf{ímpar}) = \mathbf{par}$.
Ambas são modelos válidos se $E_{\text{NatPeano}} = \emptyset$. Nelas, termos como `suc(suc(zero))` teriam interpretações diferentes: $2$ em $A_1$ e $\mathbf{par}$ em $A_2$. ■

---

## 2.4 Construtores, Observadores e Operações Derivadas

Dentro da assinatura $\Sigma$ de um TAD, as operações podem ser classificadas com base em seu papel:

1.  **Construtores (`Constructors`):**
    *   **Papel:** Usados para construir todos os valores possíveis do sort principal do TAD.
    *   **Exemplo (TAD `Stack`):** `create` e `push`.
    *   **Exemplo (TAD `Natural` de Peano):** `zero` e `suc`.

2.  **Observadores (`Observers`):**
    *   **Papel:** Retornam informações sobre um valor do TAD, geralmente para um sort "primitivo" (e.g., `Boolean`).
    *   **Exemplo (TAD `Stack`):** `top` e `isEmpty`.
    *   **Exemplo (TAD `Natural`):** `isZero`.

3.  **Operações Derivadas (ou Modificadores):**
    *   **Papel:** Operações cujos resultados poderiam ser expressos usando construtores e observadores, ou que "modificam" um valor (retornando um novo valor em especificações funcionais).
    *   **Exemplo (TAD `Stack`):** `pop`.
    *   **Exemplo (TAD `Natural`):** `add`.

A distinção é conceitualmente útil para projetar especificações, especialmente para definir axiomas por análise de casos sobre os construtores e para analisar a completeza da especificação.

**Exemplo (Axiomas de `isEmpty` para `Stack`):**
Construtores de `Stack`: `create` e `push(s,x)`.
*   `isEmpty(create) = true` `(Axioma S1)`
*   `isEmpty(push(s,x)) = false` `(Axioma S2)`
Estes axiomas cobrem todos os casos de como uma pilha pode ser construída.

---

## 2.5 Exemplos Fundamentais de Especificações Algébricas

Vamos detalhar as especificações algébricas para `Boolean`, `Natural` e `Integer`.

**Especificação de `Boolean`**

O TAD `Boolean` representa os valores de verdade lógicos.

> **SPEC ADT `Boolean`**
>
> **Sorts:**
> *   `Boolean`
>
> **Signature (Σ<sub>Boolean</sub>):**
> *   `true:` $\rightarrow$ `Boolean` (Construtor)
> *   `false:` $\rightarrow$ `Boolean` (Construtor)
> *   `not:` `Boolean` $\rightarrow$ `Boolean`
> *   `and:` `Boolean` $\times$ `Boolean` $\rightarrow$ `Boolean`
> *   `or:` `Boolean` $\times$ `Boolean` $\rightarrow$ `Boolean`
> *   `eqB:` `Boolean` $\times$ `Boolean` $\rightarrow$ `Boolean`
>
> **Axioms (E<sub>Boolean</sub>):**
>
> `For all b, b1, b2: Boolean:`
>
> *   (Axiom B1) `not(true) = false`
> *   (Axiom B2) `not(false) = true`
> *   (Axiom B3) `and(true, b) = b`
> *   (Axiom B4) `and(false, b) = false`
> *   (Axiom B5) `or(true, b) = true`
> *   (Axiom B6) `or(false, b) = b`
> *   (Axiom B7) `eqB(true, true) = true`
> *   (Axiom B8) `eqB(false, false) = true`
> *   (Axiom B9) `eqB(true, false) = false`
> *   (Axiom B10) `eqB(false, true) = false`

**Especificação de `Natural` (Números Naturais segundo Peano)**

O TAD `Natural` representa os números naturais $\{0, 1, 2, \dots\}$.

> **SPEC ADT `Natural`**
>
> **Sorts:** `Natural`, `Boolean` (importado).
>
> **Signature (Σ<sub>Natural</sub>):**
> *   `zero:` $\rightarrow$ `Natural` (Construtor)
> *   `suc:` `Natural` $\rightarrow$ `Natural` (Construtor)
> *   `isZero:` `Natural` $\rightarrow$ `Boolean` (Observador)
> *   `add:` `Natural` $\times$ `Natural` $\rightarrow$ `Natural` (Derivada)
> *   `mult:` `Natural` $\times$ `Natural` $\rightarrow$ `Natural` (Derivada)
> *   `eqN:` `Natural` $\times$ `Natural` $\rightarrow$ `Boolean` (Observador)
>
> **Axioms (E<sub>Natural</sub>):**
>
> `For all n, m: Natural:`
>
> *   (Axiom N1) `isZero(zero) = true`
> *   (Axiom N2) `isZero(suc(n)) = false`
> *   (Axiom N3) `add(zero, m) = m`
> *   (Axiom N4) `add(suc(n), m) = suc(add(n, m))`
> *   (Axiom N5) `mult(zero, m) = zero`
> *   (Axiom N6) `mult(suc(n), m) = add(m, mult(n, m))`
> *   (Axiom N7) `eqN(zero, zero) = true`
> *   (Axiom N8) `eqN(suc(n), zero) = false`
> *   (Axiom N9) `eqN(zero, suc(m)) = false`
> *   (Axiom N10) `eqN(suc(n), suc(m)) = eqN(n, m)`

**Especificação de `Integer` (Inteiros)**

O TAD `Integer` representa os números inteiros $\{\dots, -2, -1, 0, 1, 2, \dots\}$.

> **SPEC ADT `Integer`**
>
> **Sorts:** `Integer`, `Boolean` (importado).
>
> **Signature (Σ<sub>Integer</sub>):**
> *   `zeroI:` $\rightarrow$ `Integer` (Construtor)
> *   `sucI:` `Integer` $\rightarrow$ `Integer` (Construtor)
> *   `predI:` `Integer` $\rightarrow$ `Integer` (Construtor)
> *   `addI:` `Integer` $\times$ `Integer` $\rightarrow$ `Integer`
> *   `negI:` `Integer` $\rightarrow$ `Integer`
> *   `eqI:` `Integer` $\times$ `Integer` $\rightarrow$ `Boolean`
>
> **Axioms (E<sub>Integer</sub>):**
>
> `For all i, j: Integer:`
>
> *   (Axiom I1) `sucI(predI(i)) = i`
> *   (Axiom I2) `predI(sucI(i)) = i`
> *   (Axiom I3) `addI(i, zeroI) = i`
> *   (Axiom I4) `addI(i, sucI(j)) = sucI(addI(i, j))`
> *   (Axiom I5) `addI(i, predI(j)) = predI(addI(i, j))`
> *   (Axiom I6) `negI(zeroI) = zeroI`
> *   (Axiom I7) `negI(sucI(i)) = predI(negI(i))`
> *   (Axiom I8) `eqI(zeroI, zeroI) = true`
> *   (Axiom I9) `eqI(sucI(i), zeroI) = eqI(i, predI(zeroI))`
> *   (Axiom I10) `eqI(predI(i), zeroI) = eqI(i, sucI(zeroI))`
> *   (Axiom I11) `eqI(sucI(i), sucI(j)) = eqI(i, j)`
> *   (Axiom I12) `eqI(predI(i), predI(j)) = eqI(i, j)`
> *   (Axiom I13) `eqI(sucI(i), predI(j)) = eqI(i, predI(predI(j)))`
>
> *Nota: A especificação completa e consistente de `eqI` para inteiros com `sucI`/`predI` pode ser mais sutil e requerer axiomas adicionais ou uma abordagem baseada em formas normais, especialmente para garantir propriedades como terminação em sistemas de reescrita. Os axiomas `I9`, `I10`, `I13` são tentativas de cobrir casos mistos e podem precisar de refinamento para robustez total.*

**Exercício 2.3:**
Usando os axiomas `(Axiom N3) add(zero, m) = m` e `(Axiom N4) add(suc(n), m) = suc(add(n, m))` do TAD `Natural`, mostre a derivação de `add(suc(suc(zero)), suc(zero))`. O resultado deve ser um termo construído apenas com `suc` e `zero`.

**Resolução 2.3:**
Queremos calcular `add(suc(suc(zero)), suc(zero))`.
1.  `add(suc(suc(zero)), suc(zero))`
    *   Aplicando `(Axiom N4)` com $n = \text{suc(zero)}$ e $m = \text{suc(zero)}$:
    *   $= \text{suc(add(suc(zero), suc(zero)))}$ (Equação 2.3)
2.  Cálculo de `add(suc(zero), suc(zero))`:
    *   Aplicando `(Axiom N4)` com $n = \text{zero}$ e $m = \text{suc(zero)}$:
    *   `add(suc(zero), suc(zero)) = suc(add(zero, suc(zero)))` (Equação 2.4)
3.  Cálculo de `add(zero, suc(zero))`:
    *   Aplicando `(Axiom N3)` com $m = \text{suc(zero)}$:
    *   `add(zero, suc(zero)) = suc(zero)` (Equação 2.5)
4.  Substituindo (Equação 2.5) em (Equação 2.4):
    *   `add(suc(zero), suc(zero)) = suc(suc(zero))` (Equação 2.6)
5.  Substituindo (Equação 2.6) em (Equação 2.3):
    *   `add(suc(suc(zero)), suc(zero)) = suc(suc(suc(zero)))`
Portanto, `add(suc(suc(zero)), suc(zero)) = suc(suc(suc(zero)))`. ■

---

Este capítulo forneceu uma introdução detalhada à especificação algébrica de Tipos Abstratos de Dados. Cobrimos os blocos de construção fundamentais: sorts, assinaturas, termos, axiomas equacionais e o conceito de Σ-álgebras como modelos semânticos. A distinção entre construtores, observadores e operações derivadas foi discutida, e ilustramos a teoria com especificações para `Boolean`, `Natural` e `Integer`. A especificação algébrica nos oferece uma ferramenta poderosa para definir TADs com precisão matemática, focando em seu comportamento abstrato. No próximo capítulo, exploraremos como os Sistemas de Reescrita de Termos podem ser usados para dar uma semântica operacional a essas especificações, tratando os axiomas como regras de computação.

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
