---

# CAPÍTULO 2

# ESPECIFICAÇÃO ALGÉBRICA DE TIPOS ABSTRATOS DE DADOS

---

No capítulo anterior, estabelecemos a importância da abstração de dados e a necessidade de especificações precisas para garantir a corretude e a robustez do software. Introduzimos os Tipos Abstratos de Dados (TADs, ou ADTs – Abstract Data Types) como modelos matemáticos que se concentram no "o quê" um tipo de dado faz, em vez do "como" ele é implementado. Agora, mergulharemos em uma das técnicas formais mais poderosas e elegantes para definir ADTs: a **especificação algébrica**. Esta abordagem utiliza os conceitos da álgebra universal e da lógica equacional para descrever os sorts (tipos), as operações e, crucialmente, o comportamento dos ADTs através de um conjunto de axiomas na forma de equações. A especificação algébrica nos permite definir ADTs de maneira precisa, não ambígua e independente de qualquer representação ou linguagem de programação particular. Ela fornece uma base sólida para o raciocínio formal sobre as propriedades dos dados, para a verificação de implementações e para a derivação de testes. Neste capítulo, exploraremos os fundamentos das álgebras multissortidas, assinaturas, termos e axiomas equacionais. Investigaremos a semântica dessas especificações através do conceito de Σ-álgebras e modelos, e distinguiremos os papéis cruciais dos construtores e observadores. Finalmente, ilustraremos esses conceitos com especificações algébricas para ADTs fundamentais como `Boolean`, `Natural` (números naturais) e `Integer` (números inteiros), preparando o terreno para análises mais aprofundadas e técnicas de verificação nos capítulos subsequentes.

---

## 2.1 Introdução à Lógica Equacional e Álgebras Multissortidas

A especificação algébrica de ADTs é fundamentada na lógica equacional e na teoria das álgebras universais, adaptada para um contexto multissortido (ou heterogêneo), o que significa que lidamos com múltiplos tipos (sorts) de dados simultaneamente, em vez de um único conjunto universal. Esta abordagem nos permite modelar sistemas onde diferentes tipos de entidades interagem.

A lógica equacional é um sistema formal para raciocinar sobre igualdade. Em nosso contexto, ela será usada para expressar as propriedades comportamentais das operações de um ADT. Uma equação, como `$x + y = y + x$`, afirma que dois termos (expressões) denotam o mesmo valor sob certas condições. As álgebras multissortidas fornecem a estrutura semântica, ou seja, os "mundos" matemáticos onde essas equações podem ser interpretadas e sua validade verificada.

### 2.1.1 Sorts, Operações e Assinaturas (Σ-Signatures)

No cerne de uma especificação algébrica está a **assinatura**, denotada frequentemente por $\Sigma$ (Sigma maiúsculo). Uma assinatura define o vocabulário básico do ADT: os nomes dos tipos de dados envolvidos e os nomes e perfis das operações que podem ser realizadas sobre eles.

```definition
**Definição 2.1 (Assinatura Multissortida Σ)**
Uma **assinatura multissortida** (ou simplesmente **assinatura**) $\Sigma$ é um par $\Sigma = (S, \Omega)$, onde:
1.  $S$ é um conjunto de **sorts**. Um sort é um nome que representa um tipo de dado ou um domínio de valores (e.g., `Nat`, `Bool`, `List`).
2.  $\Omega$ é um conjunto de **símbolos de operação** (ou nomes de operação). Cada símbolo de operação $op \in \Omega$ é associado a um **perfil** (ou aridade e tipo) da forma $s_1 \times s_2 \times \dots \times s_n \rightarrow s$, onde $s_1, \dots, s_n \in S$ são os sorts dos argumentos (domínio da operação) e $s \in S$ é o sort do resultado (contradomínio da operação). O inteiro $n \ge 0$ é a aridade da operação.
    *   Se $n=0$, a operação $op: \rightarrow s$ é chamada de **constante** (ou operação de aridade 0) do sort $s$. Ela não recebe argumentos e produz um valor do sort $s$.
```

**Exemplo:**
Considere uma assinatura simples para uma parte do ADT `Natural`:
*   $S = \{\text{Nat, Bool}\}$
*   $\Omega = \{$
    *   `zero: $\rightarrow \text{Nat}$` (constante)
    *   `succ: $\text{Nat} \rightarrow \text{Nat}$` (operação unária)
    *   `isZero: $\text{Nat} \rightarrow \text{Bool}$` (operação unária)
    *   `true: $\rightarrow \text{Bool}$` (constante)
    *   `false: $\rightarrow \text{Bool}$` (constante)
    $\}$

Neste exemplo, `Nat` e `Bool` são os sorts. `zero` é uma constante de sort `Nat`. `succ` é uma operação que toma um `Nat` e retorna um `Nat`. `isZero` toma um `Nat` e retorna um `Bool`. `true` e `false` são constantes de sort `Bool`.

A assinatura estabelece a "sintaxe" permitida para formar expressões (termos) sobre o ADT. Ela não diz nada sobre o *significado* ou comportamento dessas operações; isso será definido pelos axiomas. A escolha dos nomes dos sorts e das operações deve ser intuitiva e refletir o conceito que se está modelando.

**Exercício 2.A**
Defina uma assinatura $\Sigma = (S, \Omega)$ para um ADT `Point` que representa pontos bidimensionais com coordenadas inteiras. O ADT deve incluir operações para:
1.  Criar um ponto a partir de duas coordenadas inteiras.
2.  Obter a coordenada x de um ponto.
3.  Obter a coordenada y de um ponto.
Assuma que um sort `Int` (para inteiros) já existe e faz parte de $S$.

**Resolução:**
Para o ADT `Point`:

*   $S = \{\text{Point, Int}\}$
    (Assumimos que `Int` é um sort que representa os números inteiros. Se `Int` também estivesse sendo definido, ele teria suas próprias operações, como constantes numéricas, adição, etc., mas aqui o foco é apenas na estrutura do `Point`.)

*   $\Omega = \{$
    *   `create_point: $\text{Int, Int} \rightarrow \text{Point}$`
        (Operação binária que toma dois argumentos do sort `Int` e retorna um valor do sort `Point`.)
    *   `get_x: $\text{Point} \rightarrow \text{Int}$`
        (Operação unária que toma um argumento do sort `Point` e retorna sua coordenada x, do sort `Int`.)
    *   `get_y: $\text{Point} \rightarrow \text{Int}$`
        (Operação unária que toma um argumento do sort `Point` e retorna sua coordenada y, do sort `Int`.)
    $\}$

Esta assinatura $\Sigma = (S, \Omega)$ define os tipos `Point` e `Int` e as três operações `create_point`, `get_x` e `get_y` com seus respectivos perfis, estabelecendo a sintaxe para interagir com o ADT `Point`.

### 2.1.2 Termos sobre uma Assinatura ($\mathcal{T}_{\Sigma}(X)$). Variáveis e Termos Fundamentais (`Ground Terms`)

Dada uma assinatura $\Sigma = (S, \Omega)$, podemos construir **termos**. Termos são expressões formadas recursivamente usando os símbolos de operação de $\Omega$ e, opcionalmente, variáveis. Eles representam os "cálculos" ou "valores" que podem ser expressos dentro do ADT.

Para definir termos formalmente, primeiro precisamos de um conjunto de variáveis. É comum usar um conjunto $S$-indexado de variáveis, $X = \{X_s\}_{s \in S}$, onde $X_s$ é um conjunto de variáveis do sort $s$. Cada variável $x \in X_s$ tem um sort associado.

```definition
**Definição 2.2 (Termos sobre Σ e X - $\mathcal{T}_{\Sigma}(X)$)**
Dado uma assinatura $\Sigma = (S, \Omega)$ e um conjunto $S$-indexado de variáveis $X = \{X_s\}_{s \in S}$, o conjunto $S$-indexado dos **termos sobre $\Sigma$ com variáveis em $X$**, denotado por $\mathcal{T}_{\Sigma}(X) = \{\mathcal{T}_{\Sigma,s}(X)\}_{s \in S}$, é o menor conjunto $S$-indexado que satisfaz as seguintes regras:
1.  **Variáveis:** Se $x \in X_s$ (i.e., $x$ é uma variável de sort $s$), então $x \in \mathcal{T}_{\Sigma,s}(X)$ (toda variável de sort $s$ é um termo de sort $s$).
2.  **Operações Aplicadas:** Se $op: s_1 \times \dots \times s_n \rightarrow s$ é um símbolo de operação em $\Omega$ (com $n \ge 0$), e $t_1 \in \mathcal{T}_{\Sigma,s_1}(X), \dots, t_n \in \mathcal{T}_{\Sigma,s_n}(X)$ são termos dos sorts requeridos pelos argumentos de $op$, então a expressão $op(t_1, \dots, t_n)$ é um termo de sort $s$, i.e., $op(t_1, \dots, t_n) \in \mathcal{T}_{\Sigma,s}(X)$.
    *   No caso de uma constante $c: \rightarrow s$ (onde $n=0$), $c$ (ou $c()$) é um termo de sort $s$, i.e., $c \in \mathcal{T}_{\Sigma,s}(X)$.
```

Essencialmente, um termo é uma variável ou uma aplicação de um símbolo de operação a termos dos sorts corretos.

*   **Termos Fundamentais (`Ground Terms`):** Se o conjunto de variáveis $X$ é vazio ($X = \emptyset$), os termos resultantes são chamados de **termos fundamentais** (ou `ground terms`). Eles não contêm variáveis. O conjunto de todos os termos fundamentais de sort $s$ é denotado por $\mathcal{T}_{\Sigma,s}$ ou $\mathcal{T}_{\Sigma,s}(\emptyset)$. Termos fundamentais representam valores concretos que podem ser construídos no ADT.
    *   Exemplo (usando a assinatura de `Natural` da seção 2.1.1):
        *   `zero` é um termo fundamental de sort `Nat`.
        *   `succ(zero)` é um termo fundamental de sort `Nat`.
        *   `succ(succ(zero))` é um termo fundamental de sort `Nat`.
        *   `isZero(succ(zero))` é um termo fundamental de sort `Bool`.

*   **Termos com Variáveis:** Se $X$ não é vazio, os termos podem conter variáveis.
    *   Exemplo (se $n_1, n_2$ são variáveis de sort `Nat`):
        *   `$n_1$` é um termo de sort `Nat`.
        *   `succ($n_1$)` é um termo de sort `Nat`.
        *   `isZero($n_2$)` é um termo de sort `Bool`.

A estrutura dos termos é arbórea. Por exemplo, o termo `succ(add(zero, $n_1$))` pode ser visualizado como uma árvore com `succ` na raiz, tendo `add` como único filho, que por sua vez tem `zero` e `$n_1$` como filhos.

A notação $\mathcal{T}_{\Sigma}(X)$ representa a coleção de todos os termos bem formados de todos os sorts, enquanto $\mathcal{T}_{\Sigma,s}(X)$ refere-se especificamente aos termos de sort $s$.

### 2.1.3 Substituições e Instanciação

Variáveis em termos atuam como espaços reservados (`placeholders`). Uma **substituição** é uma função que mapeia variáveis para termos. Aplicar uma substituição a um termo significa substituir cada ocorrência de uma variável no termo pelo termo correspondente mapeado pela substituição.

```definition
**Definição 2.3 (Substituição)**
Uma **substituição** $\sigma: X \rightarrow \mathcal{T}_{\Sigma}(Y)$ é um mapeamento $S$-sortido de um conjunto de variáveis $X$ para termos construídos sobre um conjunto de variáveis $Y$ (frequentemente $Y=X$ ou $Y=\emptyset$). Para cada variável $x \in X_s$, a substituição $\sigma$ atribui um termo $\sigma(x) \in \mathcal{T}_{\Sigma,s}(Y)$ do mesmo sort $s$.
A aplicação de uma substituição $\sigma$ a um termo $t \in \mathcal{T}_{\Sigma}(X)$, denotada por $t\sigma$ ou $\sigma(t)$, é definida recursivamente:
1.  Se $t = x$ (uma variável $x \in X_s$), então $t\sigma = \sigma(x)$.
2.  Se $t = op(t_1, \dots, t_n)$, então $t\sigma = op(t_1\sigma, \dots, t_n\sigma)$. (A substituição é aplicada recursivamente aos subtermos).
```

O termo $t\sigma$ é chamado de **instância** do termo $t$ (obtida por $\sigma$). Se uma substituição mapeia variáveis para termos fundamentais (i.e., $\sigma: X \rightarrow \mathcal{T}_{\Sigma}$), então $t\sigma$ será um termo fundamental, desde que $t$ não contenha outras variáveis além daquelas em $X$.

**Exemplo:**
Seja $\Sigma$ a assinatura de `Natural` e $X = \{m, n\}$ variáveis de sort `Nat`.
Considere o termo $t = \text{add}(\text{succ}(m), n)$.
Considere a substituição $\sigma = \{ m \mapsto \text{zero}, n \mapsto \text{succ(zero)} \}$.
Então, $t\sigma = \text{add}(\text{succ}(m), n)\sigma = \text{add}(\text{succ}(m\sigma), n\sigma) = \text{add}(\text{succ}(\text{zero}), \text{succ(zero)})$.
O resultado $t\sigma$ é um termo fundamental.

As substituições são fundamentais para definir a validade de axiomas (que são equações entre termos com variáveis) e para o processo de derivação na lógica equacional. Quando um axioma é afirmado como universalmente verdadeiro para todas as variáveis, uma substituição nos permite criar uma instância específica desse axioma.

## 2.2 Axiomas Equacionais e Especificações Algébricas

Até agora, definimos a sintaxe de um ADT através de sua assinatura $\Sigma$ e a capacidade de formar termos $\mathcal{T}_{\Sigma}(X)$. Para especificar o comportamento (a semântica) das operações, introduzimos **axiomas equacionais**.

### 2.2.1 Definição de Axiomas como Equações entre Termos

Um axioma equacional é uma afirmação de que dois termos são iguais, sob uma quantificação universal implícita sobre todas as variáveis que aparecem nos termos.

```definition
**Definição 2.4 (Axioma Equacional)**
Dado uma assinatura $\Sigma = (S, \Omega)$ e um conjunto $S$-indexado de variáveis $X$, um **axioma equacional** (ou simplesmente **equação**) é uma fórmula da forma:
$\forall X_0. l = r$
onde $X_0 \subseteq X$ é o conjunto de variáveis que ocorrem em $l$ ou $r$ (ou ambos), e $l, r \in \mathcal{T}_{\Sigma,s}(X_0)$ são termos do mesmo sort $s \in S$.
Por convenção, o quantificador universal $\forall X_0$ é frequentemente omitido, e todas as variáveis que aparecem em uma equação são consideradas universalmente quantificadas. Assim, um axioma é tipicamente escrito como:
$l = r$
```

**Exemplo:**
Na especificação do ADT `Natural`, o axioma `add(zero, n) = n` é uma abreviação para `$\forall n \in \text{Nat. add(zero, } n) = n$`. Ele afirma que para qualquer número natural `n`, adicionar `zero` a `n` resulta no próprio `n`.
Outro axioma, `add(succ(m), n) = succ(add(m, n))`, significa `$\forall m \in \text{Nat}, \forall n \in \text{Nat. add(succ(}m), n) = \text{succ(add(}m, n))$`.

Esses axiomas não são arbitrários; eles são cuidadosamente escolhidos para capturar as propriedades desejadas do ADT. Eles definem as inter-relações entre as operações. Por exemplo, os axiomas para `add` no ADT `Natural` definem recursivamente a operação de adição em termos do construtor `succ` e da constante `zero`.

### 2.2.2 Uma Especificação Algébrica SP = (Σ, E) como um par Assinatura-Axiomas

Com os conceitos de assinatura e axiomas equacionais, podemos agora definir formalmente uma especificação algébrica.

```definition
**Definição 2.5 (Especificação Algébrica)**
Uma **especificação algébrica** $SP$ é um par $SP = (\Sigma, E)$, onde:
1.  $\Sigma = (S, \Omega)$ é uma assinatura multissortida.
2.  $E$ é um conjunto de axiomas equacionais sobre $\Sigma$ e um conjunto adequado de variáveis $X$. (As variáveis são implicitamente quantificadas universalmente sobre cada axioma).
```

Uma especificação algébrica $SP$ é, portanto, uma teoria formal que descreve uma classe de ADTs. $\Sigma$ fornece o vocabulário (sorts e operações), e $E$ fornece as leis (comportamento) que esses sorts e operações devem obedecer.

**Exemplo:**
A especificação completa para `Natural` (excluindo `Boolean` por enquanto para simplificar $S$) seria:
$SP_{Natural} = (\Sigma_{Natural}, E_{Natural})$
onde:
*   $\Sigma_{Natural} = (S, \Omega)$ com
    *   $S = \{\text{Nat}\}$
    *   $\Omega = \{\text{zero: } \rightarrow \text{Nat, succ: Nat} \rightarrow \text{Nat, add: Nat, Nat} \rightarrow \text{Nat}\}$
*   $E_{Natural} = \{$
    *   `add(zero, n) = n`
    *   `add(succ(m), n) = succ(add(m, n))`
    $\}$
(Assumindo variáveis $m, n$ de sort `Nat`.)

Esta especificação $SP_{Natural}$ define o que significa ser um "número natural com adição" no estilo de Peano, do ponto de vista algébrico.

### 2.2.3 Consequência Lógica e Derivação (Regras de Inferência da Lógica Equacional)

Dado uma especificação $SP = (\Sigma, E)$, estamos interessados em quais outras equações são consequências lógicas dos axiomas em $E$. Uma equação $l=r$ é uma **consequência lógica** de $E$, denotado $E \models l=r$, se $l=r$ é verdadeira em todas as álgebras que satisfazem os axiomas em $E$ (veremos a semântica na Seção 2.3).

Alternativamente, podemos usar um sistema de **derivação** (ou cálculo) para provar que uma equação $l=r$ decorre de $E$. Denotamos isso por $E \vdash l=r$. A lógica equacional possui um conjunto de regras de inferência que nos permitem derivar novas equações a partir de um conjunto de axiomas. Um sistema de prova para a lógica equacional é **sólido** se tudo o que é derivável é uma consequência lógica ($E \vdash l=r \implies E \models l=r$), e **completo** se toda consequência lógica é derivável ($E \models l=r \implies E \vdash l=r$). A lógica equacional padrão é sólida e completa.

As regras de inferência fundamentais da lógica equacional são:

1.  **Reflexividade:** Para qualquer termo $t$, a equação $t=t$ é derivável.
    $E \vdash t=t$

2.  **Simetria:** Se $E \vdash l=r$, então $E \vdash r=l$.
    $\frac{E \vdash l=r}{E \vdash r=l}$

3.  **Transitividade:** Se $E \vdash l=r$ e $E \vdash r=u$, então $E \vdash l=u$.
    $\frac{E \vdash l=r \quad E \vdash r=u}{E \vdash l=u}$

4.  **Substituição (ou Instanciação de Axioma):** Se $l_0=r_0$ é um axioma em $E$ (onde as variáveis em $l_0, r_0$ são $X_0$), e $\sigma: X_0 \rightarrow \mathcal{T}_{\Sigma}(X)$ é uma substituição, então $E \vdash l_0\sigma = r_0\sigma$.
    (Basicamente, podemos substituir as variáveis em um axioma por quaisquer termos dos sorts apropriados.)

5.  **Congruência:** Para qualquer símbolo de operação $op: s_1 \times \dots \times s_n \rightarrow s$, se temos $E \vdash t_i=u_i$ para cada $i=1, \dots, n$ (onde $t_i, u_i$ são termos de sort $s_i$), então $E \vdash op(t_1, \dots, t_n) = op(u_1, \dots, u_n)$.
    $\frac{E \vdash t_1=u_1 \quad \dots \quad E \vdash t_n=u_n}{E \vdash op(t_1, \dots, t_n) = op(u_1, \dots, u_n)}$
    (Se os argumentos de uma operação são iguais, então aplicar a operação a eles resulta em termos iguais. Isso significa que "iguais podem ser substituídos por iguais em qualquer contexto de termo".)

Uma derivação é uma sequência finita de equações, onde cada equação na sequência é ou um axioma instanciado (regra 4) ou é obtida de equações anteriores na sequência pela aplicação das regras de reflexividade, simetria, transitividade ou congruência.

**Exercício 2.B**
Usando a especificação $SP_{Natural}$ (com $\Sigma_{Natural}$ e $E_{Natural}$ como definidos acima) e as regras de inferência da lógica equacional, mostre uma derivação para $E_{Natural} \vdash \text{add(succ(zero), zero)} = \text{succ(zero)}$. Indique qual regra ou axioma é usado em cada passo.

**Resolução:**
Queremos derivar $\text{add(succ(zero), zero)} = \text{succ(zero)}$ a partir de $E_{Natural} = \{ \text{A1: add(zero, n) = n, A2: add(succ(m), n) = succ(add(m, n))}\}$.

1.  $ \text{add(succ(m), n) = succ(add(m, n))}$
    (Axioma A2 de $E_{Natural}$)
2.  $ \text{add(succ(zero), zero) = succ(add(zero, zero))}$
    (Da linha 1, por Substituição, aplicando $\sigma = \{m \mapsto \text{zero}, n \mapsto \text{zero}\}$ ao axioma A2. Note que `zero` é um termo fundamental de sort `Nat`.)
3.  $ \text{add(zero, n) = n}$
    (Axioma A1 de $E_{Natural}$)
4.  $ \text{add(zero, zero) = zero}$
    (Da linha 3, por Substituição, aplicando $\sigma' = \{n \mapsto \text{zero}\}$ ao axioma A1.)
5.  $ \text{succ(add(zero, zero)) = succ(zero)}$
    (Da linha 4, por Congruência, aplicando o operador `succ` a ambos os lados da equação `add(zero, zero) = zero`.)
6.  $ \text{add(succ(zero), zero) = succ(zero)}$
    (Das linhas 2 e 5, usando Transitividade, pois temos $LHS_2 = RHS_2$ e $RHS_2 = RHS_5$, então $LHS_2 = RHS_5$. Explicitamente: $LHS_2$ é `add(succ(zero), zero)`. $RHS_2$ é `succ(add(zero, zero))`. $RHS_5$ é `succ(zero)`. Então `add(succ(zero), zero) = succ(zero)`.)

Assim, demonstramos que $E_{Natural} \vdash \text{add(succ(zero), zero)} = \text{succ(zero)}$ é derivável.

## 2.3 Semântica das Especificações Algébricas: Σ-Álgebras

Uma especificação algébrica $SP = (\Sigma, E)$ define uma teoria sintática. Para dar significado (semântica) a essa teoria, precisamos de estruturas matemáticas chamadas **Σ-álgebras** (ou álgebras sobre $\Sigma$). Uma Σ-álgebra fornece uma interpretação concreta para os sorts e símbolos de operação da assinatura $\Sigma$.

### 2.3.1 Definição de uma Σ-Álgebra: Domínios Portadores e Interpretação de Operações

```definition
**Definição 2.6 (Σ-Álgebra)**
Dada uma assinatura multissortida $\Sigma = (S, \Omega)$, uma **Σ-álgebra** (ou **álgebra para $\Sigma$**), denotada por $\mathcal{A}$, consiste em:
1.  Para cada sort $s \in S$, um conjunto não-vazio $A_s$, chamado **domínio portador** (ou conjunto suporte, `carrier set`) do sort $s$ na álgebra $\mathcal{A}$.
2.  Para cada símbolo de operação $op: s_1 \times \dots \times s_n \rightarrow s$ em $\Omega$, uma função concreta (ou operação) $op^{\mathcal{A}}: A_{s_1} \times \dots \times A_{s_n} \rightarrow A_s$ na álgebra $\mathcal{A}$, que mapeia tuplas de elementos dos domínios portadores dos argumentos para um elemento do domínio portador do resultado.
    *   Em particular, para uma constante $c: \rightarrow s$, sua interpretação $c^{\mathcal{A}}$ é um elemento específico do conjunto $A_s$.
```

Uma Σ-álgebra $\mathcal{A}$ é, portanto, uma coleção de conjuntos (os $A_s$) juntamente com funções concretas ($op^{\mathcal{A}}$) que operam sobre esses conjuntos, respeitando os perfis definidos na assinatura $\Sigma$.

**Exemplo:**
Considere a assinatura $\Sigma_{NatBool}$ da Seção 2.1.1: $S = \{\text{Nat, Bool}\}$, $\Omega = \{\text{zero, succ, isZero, true, false}\}$.
Uma possível Σ-álgebra $\mathcal{A}_{std}$ (a "álgebra padrão") para $\Sigma_{NatBool}$ seria:
1.  Domínios Portadores:
    *   $A_{Nat} = \{0, 1, 2, \dots \}$ (o conjunto dos números naturais usuais).
    *   $A_{Bool} = \{\textbf{true}, \textbf{false}\}$ (o conjunto dos valores booleanos usuais).
2.  Interpretação das Operações:
    *   $zero^{\mathcal{A}_{std}} = 0 \in A_{Nat}$.
    *   $succ^{\mathcal{A}_{std}}: A_{Nat} \rightarrow A_{Nat}$, definida por $succ^{\mathcal{A}_{std}}(k) = k+1$ para todo $k \in A_{Nat}$.
    *   $isZero^{\mathcal{A}_{std}}: A_{Nat} \rightarrow A_{Bool}$, definida por $isZero^{\mathcal{A}_{std}}(k) = \textbf{true}$ se $k=0$, e $\textbf{false}$ caso contrário.
    *   $true^{\mathcal{A}_{std}} = \textbf{true} \in A_{Bool}$.
    *   $false^{\mathcal{A}_{std}} = \textbf{false} \in A_{Bool}$.

Esta álgebra $\mathcal{A}_{std}$ fornece uma interpretação concreta e intuitiva para os símbolos da assinatura. No entanto, outras álgebras podem existir para a mesma assinatura. Por exemplo, poderíamos ter uma álgebra "módulo 3" para `Nat`:
$\mathcal{A}_{mod3}$:
*   $A'_{Nat} = \{0, 1, 2\}$
*   $A'_{Bool} = \{\textbf{T}, \textbf{F}\}$
*   $zero^{\mathcal{A}_{mod3}} = 0$
*   $succ^{\mathcal{A}_{mod3}}(k) = (k+1) \pmod 3$
*   $isZero^{\mathcal{A}_{mod3}}(k) = \textbf{T}$ se $k=0$, $\textbf{F}$ caso contrário.
*   $true^{\mathcal{A}_{mod3}} = \textbf{T}$
*   $false^{\mathcal{A}_{mod3}} = \textbf{F}$

### 2.3.2 Satisfação de Equações em uma Álgebra. Modelos de uma Especificação

Uma Σ-álgebra dá significado aos termos. Para avaliar um termo $t \in \mathcal{T}_{\Sigma}(X)$ em uma álgebra $\mathcal{A}$, precisamos primeiro de uma **atribuição de variáveis** (ou valoração) $\nu: X \rightarrow \bigcup_{s \in S} A_s$, que mapeia cada variável $x \in X_s$ a um elemento $\nu(x) \in A_s$ do domínio portador correspondente em $\mathcal{A}$.

A **avaliação de um termo $t$ em $\mathcal{A}$ sob $\nu$**, denotada $\text{eval}_{\mathcal{A},\nu}(t)$ (ou $[[t]]_{\mathcal{A},\nu}$), é definida recursivamente:
1.  Se $t = x$ (uma variável), então $\text{eval}_{\mathcal{A},\nu}(x) = \nu(x)$.
2.  Se $t = op(t_1, \dots, t_n)$, então $\text{eval}_{\mathcal{A},\nu}(op(t_1, \dots, t_n)) = op^{\mathcal{A}}(\text{eval}_{\mathcal{A},\nu}(t_1), \dots, \text{eval}_{\mathcal{A},\nu}(t_n))$.
(Para uma constante $c$, $\text{eval}_{\mathcal{A},\nu}(c) = c^{\mathcal{A}}$.)

Agora podemos definir quando uma Σ-álgebra satisfaz uma equação:

```definition
**Definição 2.7 (Satisfação de uma Equação em uma Álgebra)**
Uma Σ-álgebra $\mathcal{A}$ **satisfaz** um axioma equacional $\forall X_0. l=r$ (ou simplesmente $l=r$), escrito $\mathcal{A} \models l=r$, se para toda atribuição de variáveis $\nu: X_0 \rightarrow \bigcup A_s$, a avaliação de $l$ em $\mathcal{A}$ sob $\nu$ é igual à avaliação de $r$ em $\mathcal{A}$ sob $\nu$. Ou seja:
$\text{eval}_{\mathcal{A},\nu}(l) = \text{eval}_{\mathcal{A},\nu}(r)$ para toda $\nu$.
```

Se uma Σ-álgebra $\mathcal{A}$ satisfaz *todos* os axiomas $E$ de uma especificação algébrica $SP = (\Sigma, E)$, então $\mathcal{A}$ é chamada de **modelo** de $SP$.

**Exemplo:**
Considere o axioma `isZero(succ(m)) = false` do ADT `Natural` (onde $m$ é uma variável de sort `Nat`).
A álgebra $\mathcal{A}_{std}$ (definida acima) satisfaz este axioma:
$\mathcal{A}_{std} \models \text{isZero(succ(m))} = \text{false}$.
Para qualquer atribuição $\nu: \{m\} \rightarrow A_{Nat}$, seja $\nu(m) = k \in \{0,1,2,...\}$.
$\text{eval}_{\mathcal{A}_{std},\nu}(\text{isZero(succ(m))}) = isZero^{\mathcal{A}_{std}}(succ^{\mathcal{A}_{std}}(\nu(m))) = isZero^{\mathcal{A}_{std}}(succ^{\mathcal{A}_{std}}(k)) = isZero^{\mathcal{A}_{std}}(k+1)$.
Como $k+1$ nunca é $0$ para $k \ge 0$, $isZero^{\mathcal{A}_{std}}(k+1) = \textbf{false}$.
E $\text{eval}_{\mathcal{A}_{std},\nu}(\text{false}) = false^{\mathcal{A}_{std}} = \textbf{false}$.
Como $\textbf{false} = \textbf{false}$, a equação é satisfeita.

A álgebra $\mathcal{A}_{mod3}$ também satisfaz `isZero(succ(m)) = false`?
Seja $\nu(m) = 2$.
$\text{eval}_{\mathcal{A}_{mod3},\nu}(\text{isZero(succ(m))}) = isZero^{\mathcal{A}_{mod3}}(succ^{\mathcal{A}_{mod3}}(2)) = isZero^{\mathcal{A}_{mod3}}((2+1)\pmod 3) = isZero^{\mathcal{A}_{mod3}}(0) = \textbf{T}$.
$\text{eval}_{\mathcal{A}_{mod3},\nu}(\text{false}) = false^{\mathcal{A}_{mod3}} = \textbf{F}$.
Como $\textbf{T} \neq \textbf{F}$, $\mathcal{A}_{mod3}$ *não* satisfaz este axioma. Portanto, $\mathcal{A}_{mod3}$ não é um modelo da especificação completa de `Natural` (que incluiria este axioma).

### 2.3.3 A Classe de Todas as Σ-Álgebras que Satisfazem E (Alg(SP)). Modelos Iniciais e Finais.

Dada uma especificação $SP = (\Sigma, E)$, a classe de todas as Σ-álgebras que são modelos de $SP$ é denotada por $\text{Alg}(SP)$.
$\text{Alg}(SP) = \{\mathcal{A} \mid \mathcal{A} \text{ é uma } \Sigma\text{-álgebra e } \mathcal{A} \models E \}$

Esta classe $\text{Alg}(SP)$ representa o universo semântico da especificação. Qualquer álgebra nesta classe é uma interpretação válida do ADT conforme especificado.

Dentro de $\text{Alg}(SP)$, certos modelos têm propriedades especiais e são de particular interesse:

1.  **Modelo Inicial ($I_{SP}$):**
    Um modelo $I_{SP} \in \text{Alg}(SP)$ é dito **inicial** se, para qualquer outro modelo $\mathcal{A} \in \text{Alg}(SP)$, existe um *único* Σ-homomorfismo $h: I_{SP} \rightarrow \mathcal{A}$. (Um Σ-homomorfismo é um mapeamento entre álgebras que preserva a estrutura das operações).
    *   **Propriedades Intuitivas:**
        *   **"No junk" (Sem lixo):** Todo elemento no domínio portador de um sort em $I_{SP}$ é a interpretação de algum termo fundamental (ground term) da especificação. Não há elementos "extras" que não possam ser construídos pelas operações.
        *   **"No confusion" (Sem confusão):** Dois termos fundamentais $t_1, t_2$ são interpretados como o mesmo elemento em $I_{SP}$ ($[[t_1]]_{I_{SP}} = [[t_2]]_{I_{SP}}$) se, e somente se, a equação $t_1=t_2$ for derivável (provável) a partir dos axiomas $E$ ($E \vdash t_1=t_2$). O modelo inicial identifica apenas o que *deve* ser igual.
    *   O modelo inicial é frequentemente construído como a **álgebra de termos fundamentais módulo as equações prováveis**, $\mathcal{T}_{\Sigma} / \equiv_E$. É considerado o modelo "padrão" ou "mínimo" da especificação. Para o ADT `Natural`, o modelo inicial corresponde aos números naturais usuais $\{0, 1, 2, \dots\}$ com as operações padrão.

2.  **Modelo Final ($Z_{SP}$):**
    Um modelo $Z_{SP} \in \text{Alg}(SP)$ é dito **final** se, para qualquer outro modelo $\mathcal{A} \in \text{Alg}(SP)$, existe um *único* Σ-homomorfismo $h: \mathcal{A} \rightarrow Z_{SP}$.
    *   **Propriedades Intuitivas:** O modelo final tende a identificar o máximo possível de elementos, ou seja, dois termos são iguais no modelo final se eles não puderem ser distinguidos por nenhuma "observação" permitida pelas operações da assinatura que resultam em sorts "visíveis" (como `Bool`). Está relacionado à noção de **equivalência observacional**.

A semântica inicial é frequentemente a mais intuitiva para ADTs definidos por construtores, como `Natural` ou `List`. Ela capta a ideia de que os valores do ADT são exatamente aqueles que podem ser construídos e que duas construções representam o mesmo valor abstrato apenas se os axiomas o exigirem. A existência e unicidade (a menos de isomorfismo) de modelos iniciais e finais são resultados importantes na teoria das especificações algébricas, mas sua construção detalhada está além do escopo introdutório deste capítulo. O importante é reconhecer que uma especificação algébrica define uma classe de modelos, e dentro dessa classe, o modelo inicial é frequentemente o que se tem em mente como a "verdadeira" semântica do ADT.

## 2.4 Construtores, Observadores e Operações Derivadas

Ao projetar uma especificação algébrica para um ADT, é útil classificar as operações com base em seu papel na definição e uso do ADT. Três categorias principais são frequentemente distinguidas: construtores, observadores e operações derivadas (ou transformadores).

1.  **Construtores (`Constructors`):**
    *   **Definição:** São operações cuja principal finalidade é criar ou gerar instâncias do sort (tipo) de interesse que está sendo definido pelo ADT.
    *   **Características:**
        *   O contradomínio (sort do resultado) de um construtor é o sort de interesse (e.g., `Nat` para o ADT `Natural`, `List` para o ADT `List`).
        *   Idealmente, um conjunto de construtores deve ser **suficientemente completo**, significando que todo valor do sort de interesse pode ser representado por um termo fundamental construído apenas com operações construtoras. (Este conceito de "completeza suficiente" será detalhado no Capítulo 4).
    *   **Exemplos:**
        *   No ADT `Natural`: `zero: \rightarrow Nat` e `succ: Nat \rightarrow Nat` são os construtores. Qualquer número natural pode ser construído por aplicações repetidas de `succ` a `zero` (e.g., `succ(succ(zero))` para o número 2).
        *   No ADT `List[Element]`: `nil: \rightarrow List` e `cons: Element, List \rightarrow List` são os construtores. Qualquer lista finita pode ser construída por aplicações de `cons` terminando com `nil` (e.g., `cons(e1, cons(e2, nil))`).
    *   Construtores são a espinha dorsal da definição indutiva de um ADT.

2.  **Observadores (`Observers`):**
    *   **Definição:** São operações que tomam uma instância do sort de interesse como argumento (possivelmente entre outros argumentos) e retornam informações sobre ela, geralmente resultando em um valor de um sort diferente e já bem compreendido (como `Bool`, ou um sort de parâmetro como `Element`, ou mesmo `Nat` se estiver sendo usado como um tipo de "dados" como em `length`).
    *   **Características:**
        *   Eles não modificam o ADT (na semântica algébrica, eles não retornam o sort de interesse). Eles "olham para dentro" do valor do ADT.
        *   São frequentemente usados nos lados esquerdos de equações axiomáticas para definir o comportamento de operações mais complexas, ou para distinguir diferentes estados do ADT.
    *   **Exemplos:**
        *   No ADT `Natural`: `isZero: Nat \rightarrow Bool`.
        *   No ADT `List[Element]`: `isEmpty: List \rightarrow Bool`, `head: List \rightarrow Element`, `tail: List \rightarrow List` (este último é um caso especial, pois retorna o sort de interesse, mas remove parte da estrutura; alguns o classificariam como um "deconstrutor" ou um tipo específico de transformador. Se a lista é imutável, `tail` produz uma *nova* lista), `length: List \rightarrow Nat`.

3.  **Operações Derivadas (ou Transformadores/Funções Auxiliares):**
    *   **Definição:** São operações adicionais que podem ser definidas em termos dos construtores e observadores (ou outras operações já definidas). Elas geralmente realizam cálculos ou transformações mais complexas sobre as instâncias do ADT.
    *   **Características:**
        *   Seu comportamento é tipicamente definido por axiomas onde o lado esquerdo da equação envolve a aplicação da operação derivada a termos construídos com construtores, e o lado direito expressa o resultado em termos de outras operações.
        *   No paradigma funcional puro das especificações algébricas, um transformador que modifica uma instância do ADT de interesse tipicamente retorna uma *nova* instância do ADT de interesse com a modificação.
    *   **Exemplos:**
        *   No ADT `Natural`: `add: Nat, Nat \rightarrow Nat`. Seus axiomas (`add(zero, n) = n`, `add(succ(m), n) = succ(add(m, n))`) mostram como `add` é definida recursivamente sobre os construtores `zero` e `succ`.
        *   No ADT `List[Element]`: Uma operação `append: List, List \rightarrow List` (para concatenar duas listas) seria uma operação derivada, definida recursivamente usando `nil` e `cons`. Uma operação `contains: List, Element \rightarrow Bool` também seria derivada.

A distinção entre construtores e outras operações é fundamental para várias técnicas de análise de especificações, como a prova de **completeza suficiente** (que verifica se todas as operações estão bem definidas para todas as combinações de construtores) e para a **indução estrutural** sobre os termos do ADT. Os observadores são cruciais para definir o que significa para duas instâncias de um ADT serem "diferentes" ou "iguais" de um ponto de vista externo.

**Exercício 2.C**
Para o ADT `Point` (com assinatura da Resolução do Exercício 2.A: $S = \{\text{Point, Int}\}$, $\Omega = \{\text{create_point: Int, Int} \rightarrow \text{Point, get_x: Point} \rightarrow \text{Int, get_y: Point} \rightarrow \text{Int}\}$), classifique cada operação como construtor, observador ou operação derivada. Justifique sua classificação.

**Resolução:**
Classificação das operações do ADT `Point`:

1.  `create_point: $\text{Int, Int} \rightarrow \text{Point}$`
    *   **Classificação:** Construtor.
    *   **Justificativa:** Esta operação é a única forma de criar instâncias do sort `Point`. Seu contradomínio é `Point`, o sort de interesse. Presumivelmente, todos os valores possíveis do ADT `Point` são gerados por esta operação (assumindo que `Int` fornece todos os valores inteiros necessários para as coordenadas).

2.  `get_x: $\text{Point} \rightarrow \text{Int}$`
    *   **Classificação:** Observador.
    *   **Justificativa:** Esta operação toma uma instância de `Point` como argumento e retorna um valor (`Int`) que é de um sort diferente do sort de interesse (`Point`). Ela "observa" ou extrai uma propriedade (a coordenada x) de um ponto, sem criar um novo `Point` ou modificar o sort `Point`.

3.  `get_y: $\text{Point} \rightarrow \text{Int}$`
    *   **Classificação:** Observador.
    *   **Justificativa:** Similar a `get_x`, esta operação toma uma instância de `Point` e retorna um `Int` (a coordenada y). Ela observa uma propriedade do ponto.

Neste exemplo simples, não temos operações derivadas explicitamente, mas poderíamos adicionar uma, por exemplo, `distance_to_origin: Point \rightarrow Real` (assumindo um sort `Real`), que seria uma operação derivada (e também um observador em relação a `Point`). Se tivéssemos uma operação como `translate: Point, Int, Int \rightarrow Point` (para transladar um ponto), ela seria classificada como uma operação derivada (ou transformadora, pois retorna um `Point`).

## 2.5 Exemplos Fundamentais de Especificações Algébricas

Para solidificar os conceitos apresentados, vamos agora detalhar as especificações algébricas para três ADTs fundamentais: `Boolean`, `Natural` (números naturais segundo Peano) e `Integer` (números inteiros). Usaremos o formato de especificação que estabelecemos.

### 2.5.1 Especificação de `Boolean`

O ADT `Boolean` representa os valores lógicos verdadeiro e falso, e as operações lógicas básicas.

```markdown
## ADT: `Boolean`
```
```text
spec Boolean
    // Defines the standard boolean values and logical operations.

sorts
    Bool    // The sort for boolean truth values

opns
    // -- Constructors (Constants)
    true: $\rightarrow Bool$                // The boolean constant true
    false: $\rightarrow Bool$               // The boolean constant false

    // -- Logical Operations
    not: $Bool \rightarrow Bool$            // Logical negation
    and: $Bool, Bool \rightarrow Bool$      // Logical conjunction (AND)
    or: $Bool, Bool \rightarrow Bool$       // Logical disjunction (OR)
    // implies: $Bool, Bool \rightarrow Bool$ // Logical implication (can be derived)
    // eq: $Bool, Bool \rightarrow Bool$      // Logical equality (can be derived)

vars
    b, b1, b2: $Bool$ // Variables of sort Bool for use in axioms

axioms
    // -- Axioms for 'not'
    1. $not(true) = false$
    2. $not(false) = true$

    // -- Axioms for 'and'
    3. $and(true, b) = b$
    4. $and(false, b) = false$
    // Note: $and(b, true) = b$ and $and(b, false) = false$ can be derived
    // or added for completeness if a specific rewriting strategy is in mind.

    // -- Axioms for 'or'
    5. $or(true, b) = true$
    6. $or(false, b) = b$
    // Note: $or(b, true) = true$ and $or(b, false) = b$ can be derived
    // or added.

    // -- Example of a derivable property (demonstrating equality)
    // For instance, to show De Morgan's Law $not(and(b1, b2)) = or(not(b1), not(b2))$
    // would require proving it from these axioms using the rules of equational logic.
    // This is not an axiom itself here, but a property.

end Boolean
```
**Discussão:**
*   **Sorts:** Apenas um sort `Bool`.
*   **Operações:**
    *   `true` e `false` são os construtores (constantes) do ADT `Boolean`. Eles representam os dois únicos valores fundamentais deste tipo.
    *   `not`, `and`, `or` são operações derivadas (ou transformadoras) que combinam ou modificam valores `Bool`.
*   **Axiomas:** Os axiomas definem o comportamento padrão da negação, conjunção e disjunção, como esperado da lógica booleana. Por exemplo, `and(true, b) = b` significa que "verdadeiro E b" é equivalente a "b".
*   A especificação é simples, mas fundamental, pois muitos outros ADTs usarão `Boolean` como um sort auxiliar para observadores ou condições.

### 2.5.2 Especificação de `Natural` (Números Naturais segundo Peano)

Este ADT formaliza os números naturais $(0, 1, 2, \dots)$ usando a abordagem de Peano, baseada em `zero` e a função `sucessor`.

```markdown
## ADT: `Natural`
```
```text
spec Natural
    // Defines natural numbers based on Peano-like axioms (0, 1, 2, ...).
    // Uses the Boolean ADT for observer operations.

uses
    Boolean // For operations like isZero, eq_nat (equality on Nat)

sorts
    Nat     // The sort for natural numbers

opns
    // -- Constructors
    zero: $\rightarrow Nat$                 // The natural number 0
    succ: $Nat \rightarrow Nat$             // The successor function (n -> n+1)

    // -- Observers
    isZero: $Nat \rightarrow Bool$         // Checks if a natural number is zero

    // -- Derived Operations
    add: $Nat, Nat \rightarrow Nat$         // Addition
    mult: $Nat, Nat \rightarrow Nat$        // Multiplication
    // eq_nat: $Nat, Nat \rightarrow Bool$   // Equality for Naturals (can be defined)
    // leq_nat: $Nat, Nat \rightarrow Bool$  // Less than or equal for Naturals (can be defined)

vars
    m, n: $Nat$ // Variables of sort Nat for use in axioms

axioms
    // -- Axioms for 'isZero' (defined with respect to constructors)
    1. $isZero(zero) = true$
    2. $isZero(succ(m)) = false$

    // -- Axioms for 'add' (recursive definition)
    3. $add(zero, n) = n$
    4. $add(succ(m), n) = succ(add(m, n))$

    // -- Axioms for 'mult' (recursive definition, using 'add')
    5. $mult(zero, n) = zero$
    6. $mult(succ(m), n) = add(n, mult(m, n))$ 
       // or $mult(succ(m), n) = add(mult(m,n), n)$ depending on preferred definition style

    // -- Optional: Axioms for 'eq_nat' (equality on Naturals)
    // 7. $eq_nat(zero, zero) = true$
    // 8. $eq_nat(succ(m), zero) = false$
    // 9. $eq_nat(zero, succ(n)) = false$
    // 10. $eq_nat(succ(m), succ(n)) = eq_nat(m, n)$

end Natural
```
**Discussão:**
*   **Construtores:** `zero` e `succ` são os construtores canônicos. Todo número natural é ou `zero` ou o `sucessor` de algum outro número natural.
*   **Observadores:** `isZero` é um observador fundamental. `eq_nat` (se incluído) também seria um observador.
*   **Operações Derivadas:** `add` e `mult` são definidas recursivamente. Note que a definição de `mult` utiliza `add`.
*   Esta especificação é um exemplo clássico de como ADTs recursivos são definidos. Os axiomas para `add` e `mult` mostram como essas operações são "reduzidas" a casos mais simples envolvendo os construtores.

### 2.5.3 Especificação de `Integer`

O ADT `Integer` representa os números inteiros $(\dots, -2, -1, 0, 1, 2, \dots)$. Existem várias maneiras de especificar `Integer` algebricamente. Uma abordagem é construí-lo a partir de `Natural`, por exemplo, representando inteiros como pares de naturais (e.g., $(a,b)$ representando $a-b$) ou usando um construtor de sinal. Outra é definir construtores diretos para inteiros. Aqui, optaremos por uma abordagem com `zero_int`, `succ_int` (sucessor inteiro) e `pred_int` (predecessor inteiro).

```markdown
## ADT: `Integer`
```
```text
spec Integer
    // Defines integers (..., -1, 0, 1, ...).
    // Uses Boolean ADT and potentially Natural ADT (e.g., for absolute value).

uses
    Boolean
    // Natural // Could be used for operations like 'abs_val'

sorts
    Int     // The sort for integer numbers

opns
    // -- Core Constructors/Operations
    zero_int: $\rightarrow Int$             // The integer 0
    succ_int: $Int \rightarrow Int$         // Successor (i -> i+1)
    pred_int: $Int \rightarrow Int$         // Predecessor (i -> i-1)

    // -- Observers
    is_zero_int: $Int \rightarrow Bool$     // Checks if an integer is zero
    // is_pos: $Int \rightarrow Bool$       // Checks if an integer is positive
    // is_neg: $Int \rightarrow Bool$       // Checks if an integer is negative

    // -- Derived Operations
    add_int: $Int, Int \rightarrow Int$     // Integer addition
    negate_int: $Int \rightarrow Int$       // Integer negation (-i)
    // sub_int: $Int, Int \rightarrow Int$    // Integer subtraction (i - j), can be $add_int(i, negate_int(j))$
    // mult_int: $Int, Int \rightarrow Int$   // Integer multiplication

vars
    i, j: $Int$ // Variables of sort Int for use in axioms

axioms
    // -- Axioms relating succ_int and pred_int
    1. $pred_int(succ_int(i)) = i$
    2. $succ_int(pred_int(i)) = i$

    // -- Axioms for 'is_zero_int'
    // This requires a way to define zero without relying on a single 'zero' constructor
    // if we want to show properties like is_zero_int(add_int(i, negate_int(i))) = true.
    // For simplicity with succ/pred, we can define it relative to 'zero_int' and its properties.
    // If 'zero_int' is the ONLY way to make zero, then:
    3. $is_zero_int(zero_int) = true$
    4. $is_zero_int(succ_int(i)) = false \quad \text{if } is_zero_int(i) = false \land \text{ (i is not -1, more complex logic needed here or different axioms)}$
       // Axioms for is_zero_int are trickier with only succ/pred if we don't want to assume 'zero_int' is canonical form.
       // A common way is to define equality first. Let's assume 'eq_int' for now.
       // Alternative for is_zero_int if eq_int is defined: $is_zero_int(i) = eq_int(i, zero_int)$

    // -- Axioms for 'negate_int'
    5. $negate_int(zero_int) = zero_int$
    6. $negate_int(succ_int(i)) = pred_int(negate_int(i))$
    // 7. $negate_int(pred_int(i)) = succ_int(negate_int(i))$ (can be derived from 1,2,6)

    // -- Axioms for 'add_int'
    8. $add_int(i, zero_int) = i$
    9. $add_int(i, succ_int(j)) = succ_int(add_int(i, j))$
    // 10. $add_int(i, pred_int(j)) = pred_int(add_int(i, j))$ (can be derived if succ/pred are true inverses)

    // Note: A full specification of Integer ensuring all desired properties
    // (like commutativity, associativity of add_int, distributivity)
    // can become quite extensive and may involve more sophisticated axiom sets
    // or building Integers from Naturals (e.g., as pairs or with signs).
    // This specification provides a basic structure.

end Integer
```
**Discussão:**
*   **Sorte e Operações Fundamentais:** O sort `Int` é introduzido. `zero_int`, `succ_int`, e `pred_int` são as operações fundamentais. Diferentemente de `Natural` onde `zero` e `succ` eram suficientes como construtores para gerar todos os valores não-negativos, para `Integer` a combinação de `zero_int` com `succ_int` e `pred_int` permite construir todos os inteiros. `succ_int(zero_int)` seria 1, `pred_int(zero_int)` seria -1.
*   **Axiomas Fundamentais:** Os axiomas $pred_int(succ_int(i)) = i$ e $succ_int(pred_int(i)) = i$ são cruciais, estabelecendo que `succ_int` e `pred_int` são inversos um do outro.
*   **Operações Derivadas:** `negate_int` (negação) e `add_int` (adição) são definidas recursivamente. A negação é definida observando que negar o sucessor de `i` é o mesmo que o predecessor da negação de `i`. A adição é definida de forma similar à adição de `Natural`, mas precisaria de um axioma para `pred_int` ou ser tratada de forma mais simétrica para cobrir números negativos de forma elegante.
*   **Complexidade:** Especificar `Integer` completamente, garantindo todas as propriedades aritméticas usuais (anéis comutativos, etc.) apenas com axiomas equacionais sobre `succ_int`/`pred_int` pode ser mais complexo do que parece, especialmente para observadores como `is_zero_int` sem uma forma canônica clara baseada apenas nesses construtores. Por exemplo, `add_int(succ_int(zero_int), pred_int(zero_int))` deve ser `zero_int`. Provar que `is_zero_int` disso é `true` requer um sistema de axiomas robusto. Frequentemente, a especificação de `Integer` é feita construindo-o a partir de dois `Natural`s (um para a parte positiva, outro para a negativa, ou um par representando uma diferença) ou usando um sort `Sign`. A abordagem aqui é uma tentativa de definir `Integer` de forma "autônoma" com sucessor e predecessor.

Estes exemplos demonstram a flexibilidade e o poder da especificação algébrica. Eles mostram como ADTs complexos podem ser construídos a partir de componentes mais simples e como o comportamento recursivo é naturalmente expresso. Nos próximos capítulos, exploraremos como analisar essas especificações em busca de propriedades como consistência e completeza, e como elas podem ser usadas para guiar a implementação e o teste.

---
## REFERÊNCIAS BIBLIOGRÁFICAS

BERGSTRA, Jan A.; HEERING, Jan; KLINT, Paul (Eds.). **Algebraic Specification**. ACM Press Frontier Series. New York, NY: Addison-Wesley, 1989. (Um clássico que compila trabalhos importantes na área)
*   *Resumo: Esta coletânea oferece uma visão aprofundada de vários aspectos da especificação algébrica, incluindo teoria, linguagens de especificação (como ASF+SDF) e aplicações. Cobre temas como semântica inicial, reescrita de termos e modularidade.*

EHRIG, Hartmut; MAHR, Bernd. **Fundamentals of Algebraic Specification 1: Equations and Initial Semantics**. EATCS Monographs on Theoretical Computer Science, Vol. 6. Berlin, Heidelberg: Springer-Verlag, 1985.
*   *Resumo: O primeiro volume de uma obra fundamental e detalhada sobre a teoria das especificações algébricas. Foca na lógica equacional, álgebras, homomorfismos, semântica inicial e na construção de tipos de dados abstratos como álgebras iniciais.*

GOGUEN, Joseph A.; THATCHER, James W.; WAGNER, Eric G. An initial algebra approach to the specification, correctness, and implementation of abstract data types. In: YEH, Raymond T. (Ed.). **Current Trends in Programming Methodology, Vol. IV: Data Structuring**. Englewood Cliffs, NJ: Prentice-Hall, 1978. p. 80-149. (Artigo seminal do grupo ADJ)
*   *Resumo: Este é um dos artigos mais influentes que estabeleceram a abordagem de álgebra inicial para a especificação de ADTs. Introduz conceitos chave como assinaturas, Σ-álgebras, homomorfismos e a ideia de que a semântica de um ADT é capturada por uma álgebra inicial.*

SANNELLA, Donald; TARLECKI, Andrzej. **Foundations of Algebraic Specification and Formal Software Development**. EATCS Monographs in Theoretical Computer Science. Berlin, Heidelberg: Springer-Verlag, 2012.
*   *Resumo: Um texto moderno e abrangente que cobre os fundamentos teóricos da especificação algébrica e suas aplicações no desenvolvimento formal de software. Aborda desde conceitos básicos até tópicos avançados como especificações estruturadas, refinamento e desenvolvimento institucional.*

WECHLER, Wolfgang. **Universal Algebra for Computer Scientists**. EATCS Monographs on Theoretical Computer Science, Vol. 25. Berlin, Heidelberg: Springer-Verlag, 1992.
*   *Resumo: Este livro oferece uma introdução à álgebra universal especificamente direcionada a cientistas da computação. Cobre tópicos como álgebras, homomorfismos, congruências, produtos, álgebras livres e variedades, com aplicações em semântica de linguagens de programação e especificações.*

WIRSING, Martin. Algebraic Specification. In: VAN LEEUWEN, Jan (Ed.). **Handbook of Theoretical Computer Science, Volume B: Formal Models and Semantics**. Amsterdam: Elsevier Science Publishers B.V. and Cambridge, MA: The MIT Press, 1990. p. 675-788.
*   *Resumo: Um capítulo de manual altamente conceituado que fornece uma visão geral abrangente e rigorosa da especificação algébrica. Cobre a teoria básica, diferentes abordagens semânticas (inicial, final, solta), especificações estruturadas, implementação e aspectos de linguagens de especificação.*

---
