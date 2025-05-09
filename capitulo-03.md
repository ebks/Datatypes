
---

# CAPÍTULO 3

# SISTEMAS DE REESCRITA DE TERMOS E SEMÂNTICA OPERACIONAL DE ESPECIFICAÇÕES ALGÉBRICAS

---

![Visualização de uma regra de reescrita transformando um termo](tipo.png)

Nos capítulos anteriores, estabelecemos a base para a especificação algébrica de Tipos Abstratos de Dados (TADs), focando em suas definições sintáticas através de assinaturas e semânticas através de axiomas equacionais. Este capítulo introduz uma perspectiva mais dinâmica e computacional sobre essas especificações: os Sistemas de Reescrita de Termos (SRTs). Veremos como os axiomas equacionais, quando orientados, podem ser interpretados como regras de transformação que simplificam termos complexos em formas mais canônicas ou "normais". Esta abordagem não apenas fornece uma semântica operacional para as especificações, conectando-as mais diretamente à noção de computação, mas também estabelece um formalismo crucial para analisar propriedades importantes das especificações, como terminação e confluência. Exploraremos as definições formais dos SRTs, incluindo regras de reescrita, relações de reescrita em um ou múltiplos passos, e o conceito de formas normais. Discutiremos propriedades fundamentais como terminação, confluência (propriedade de Church-Rosser) e canonicidade, que são essenciais para garantir que o processo de reescrita seja bem-comportado e produza resultados únicos. Finalmente, ilustraremos esses conceitos com um exemplo detalhado, aplicando a reescrita à especificação de `Natural` e analisando as propriedades do sistema resultante.

---

## 3.1 Introdução à Reescrita de Termos (`Term Rewriting`)

A reescrita de termos é um modelo de computação simples, porém poderoso, que se baseia na aplicação sucessiva de regras de simplificação (ou transformação) a expressões simbólicas chamadas termos. No contexto das especificações algébricas, os Sistemas de Reescrita de Termos (SRTs), também conhecidos como TRSs (do inglês, *Term Rewriting Systems*), fornecem uma maneira de dar uma interpretação computacional ou operacional aos axiomas equacionais. Em vez de tratar os axiomas apenas como declarações de igualdade, passamos a vê-los como instruções direcionadas para transformar termos.

**Motivação: Axiomas como Regras de Transformação**

Considere um axioma equacional típico de uma especificação algébrica, como `add(n, zero) = n` para o TAD `Natural`. Na lógica equacional, esta é uma afirmação simétrica: `add(n, zero)` é igual a `n`, e `n` é igual a `add(n, zero)`. No entanto, do ponto de vista computacional, frequentemente queremos usar essa igualdade de uma forma direcionada: para simplificar uma expressão como `add(suc(zero), zero)` para `suc(zero)`. Ou seja, interpretamos o axioma como uma regra de reescrita $l \rightarrow r$, onde o termo $l$ (o lado esquerdo) pode ser substituído pelo termo $r$ (o lado direito).

Essa orientação dos axiomas é a ideia central por trás da reescrita de termos. As regras de reescrita são usadas para "calcular" ou "reduzir" um termo, aplicando-as repetidamente até que nenhuma regra mais possa ser aplicada. O termo resultante, se existir, é chamado de forma normal e pode ser considerado o "resultado" da computação.

**A Direcionalidade da Reescrita: De Termos Complexos a Termos Mais Simples**

A escolha da direção em uma regra de reescrita $l \rightarrow r$ não é arbitrária. Geralmente, o objetivo é que o lado direito $r$ seja, de alguma forma, "mais simples" ou "mais próximo de uma forma canônica" do que o lado esquerdo $l$. Essa noção de simplicidade pode ser formalizada de várias maneiras (e.g., através de uma ordem de redução bem-fundada sobre os termos), mas a intuição é que a reescrita deve progredir em direção a um estado final.

Por exemplo, para a adição em `Natural`:
*   `add(n, zero) $\rightarrow$ n` (simplifica removendo `add` e `zero`)
*   `add(n, suc(m)) $\rightarrow$ suc(add(n, m))` (reduz a complexidade do segundo argumento de `add`)

A reescrita é um processo que, idealmente, termina. Se um termo pode ser reescrito indefinidamente sem atingir uma forma que não pode mais ser reescrita (uma forma normal), então o sistema de reescrita não é terminante para esse termo, o que corresponde a um loop infinito em um programa convencional.

**Relação entre Reescrita e Computação**

A reescrita de termos tem uma forte conexão com modelos de computação, especialmente com a programação funcional e a avaliação de expressões.

*   **Programação Funcional:** Muitas linguagens funcionais (como Haskell, Lisp, ML) podem ser vistas, em certo nível, como implementações de sistemas de reescrita de termos. A definição de uma função frequentemente corresponde a um conjunto de regras de reescrita. Por exemplo, a definição de uma função fatorial:
    `fact(0) = 1`
    `fact(n) = n * fact(n-1)` se $n > 0$
    pode ser vista como as regras de reescrita:
    `fact(0) $\rightarrow$ 1`
    `fact(n) $\rightarrow$ n * fact(n-1)` (com uma condição lateral $n > 0$)
    A avaliação de uma chamada de função como `fact(3)` corresponde a uma sequência de passos de reescrita.

*   **Cálculo Lambda:** O cálculo lambda, um dos fundamentos teóricos da programação funcional, é ele próprio um sistema de reescrita de termos com regras como a $\beta$-redução: $(\lambda x.M)N \rightarrow M[x:=N]$ (onde $M[x:=N]$ é o termo $M$ com todas as ocorrências livres de $x$ substituídas por $N$).

*   **Semântica Operacional:** A reescrita de termos fornece uma maneira natural de definir a semântica operacional de linguagens de programação ou especificações formais. Ela descreve *como* os termos são transformados, passo a passo, para produzir um resultado.

Ao transformar os axiomas de uma especificação algébrica em um conjunto de regras de reescita direcionadas, obtemos um modelo computacional para o TAD. Isso nos permite não apenas definir o que as operações significam (semanticamente, através de modelos algébricos), mas também como calcular com elas.

## 3.2 Definições Formais em Sistemas de Reescrita de Termos (SRTs / TRSs)

Para formalizar a noção de reescrita, precisamos definir o que é uma regra de reescrita e como ela induz uma relação de reescrita sobre os termos.

**Regras de Reescrita ($l \rightarrow r$) derivadas de Axiomas Equacionais**

Um **Sistema de Reescrita de Termos (SRT)**, denotado por $R$, sobre uma assinatura Σ é um conjunto de **regras de reescrita**. Cada regra de reescrita é um par de termos $(l, r)$, escrito $l \rightarrow r$, tal que:
1.  $l$ e $r$ são termos sobre Σ (geralmente com variáveis, $l, r \in T_{\Sigma}(X)$) do mesmo sort.
2.  $l$ não é uma variável isolada. (Isso evita regras como $x \rightarrow t$, que podem levar a problemas de terminação ou não-confluência).
3.  Todas as variáveis que ocorrem em $r$ também devem ocorrer em $l$. (Isso garante que a reescrita não introduza variáveis "do nada" e é importante para a propriedade de substituição fechada).

Quando derivamos um SRT a partir de uma especificação algébrica $(\Sigma, E)$, cada equação $e \in E$ da forma $t_L = t_R$ é orientada para se tornar uma ou mais regras de reescrita. A orientação $t_L \rightarrow t_R$ é escolhida se $t_R$ é considerado "mais simples" que $t_L$. Às vezes, uma equação pode ser usada em ambas as direções se a noção de "simplicidade" não for clara ou se a igualdade for usada para normalização simétrica (como na completação de Knuth-Bendix, um tópico mais avançado). No entanto, para a semântica operacional básica, buscamos uma direção preferencial.

Por exemplo, os axiomas de `Stack`:
`isEmpty(new) = true` $\implies$ `isEmpty(new) $\rightarrow$ true`
`isEmpty(push(e, s)) = false` $\implies$ `isEmpty(push(e, s)) $\rightarrow$ false`
`top(push(e, s)) = e` $\implies$ `top(push(e, s)) $\rightarrow$ e`
`pop(push(e, s)) = s` $\implies$ `pop(push(e, s)) $\rightarrow$ s`

**A Relação de Reescrita em Um Passo ($\rightarrow_R$)**

Um SRT $R$ induz uma **relação de reescrita em um passo** sobre o conjunto de todos os termos $T_{\Sigma}(Y)$ (onde $Y$ pode ser um conjunto diferente de variáveis, ou vazio para termos fundamentais), denotada por $\rightarrow_R$. Dizemos que um termo $t$ reescreve para um termo $t'$ em um passo, escrito $t \rightarrow_R t'$, se:
1.  Existe uma regra $l \rightarrow r$ em $R$.
2.  Existe uma subexpressão (ou subtermo) $u$ de $t$ em alguma posição $p$ (denotado $t|_p = u$).
3.  Existe uma substituição $\sigma$ tal que $u = l\sigma$ (ou seja, $u$ é uma instância do lado esquerdo $l$ da regra).
4.  $t'$ é obtido de $t$ substituindo a subexpressão $u$ em $t|_p$ por $r\sigma$ (a instância correspondente do lado direito $r$). Escrevemos $t' = t[r\sigma]_p$.

Em outras palavras, $t \rightarrow_R t'$ se $t'$ pode ser obtido de $t$ aplicando uma regra de $R$ a uma subexpressão de $t$ que "casa" com o lado esquerdo da regra.

Exemplo: Seja $R = \{\text{`add(x, zero)`} \rightarrow x \}$.
O termo $t = \text{suc(add(suc(zero), zero))}$ pode ser reescrito.
A subexpressão $u = \text{add(suc(zero), zero)}$ em $t$ casa com `add(x, zero)` com a substituição $\sigma = \{x \mapsto \text{suc(zero)}\}$.
O lado direito instanciado é $x\sigma = \text{suc(zero)}$.
Então, $t \rightarrow_R \text{suc(suc(zero))}$.

**A Relação de Reescrita em Múltiplos Passos ($\rightarrow_R^*$ - Fecho Reflexivo e Transitivo)**

A relação de reescrita em um passo $\rightarrow_R$ pode ser estendida para múltiplos passos:
*   $\rightarrow_R^0$ é a relação de identidade ($t \rightarrow_R^0 t$ para todo $t$).
*   $\rightarrow_R^{k+1} = \rightarrow_R^k \circ \rightarrow_R$ (composição de relações: $t \rightarrow_R^{k+1} u$ se existe $v$ tal que $t \rightarrow_R^k v$ e $v \rightarrow_R u$).
*   $\rightarrow_R^+$ (fecho transitivo): $t \rightarrow_R^+ u$ se $t \rightarrow_R^k u$ para algum $k \ge 1$.
*   $\rightarrow_R^*$ (fecho reflexivo e transitivo): $t \rightarrow_R^* u$ se $t \rightarrow_R^k u$ para algum $k \ge 0$.
    Isto significa que $u$ pode ser obtido de $t$ por zero ou mais aplicações de regras de $R$.
*   $\leftrightarrow_R^*$ (fecho simétrico, reflexivo e transitivo): Esta é a menor relação de equivalência contendo $\rightarrow_R$. Se o SRT $R$ foi derivado de um conjunto de equações $E$, então $t \leftrightarrow_R^* u$ se e somente se $E \vdash t=u$.

**Formas Normais (Termos Irredutíveis)**

Um termo $t$ é dito **irredutível** (ou em **forma normal**) com respeito a um SRT $R$ se não existe nenhum termo $t'$ tal que $t \rightarrow_R t'$. Ou seja, nenhuma regra de $R$ pode ser aplicada a $t$ ou a qualquer uma de suas subexpressões.
Se $t \rightarrow_R^* u$ e $u$ é irredutível, então $u$ é chamado de uma **forma normal de $t$**.

Um termo pode ter zero, uma ou múltiplas formas normais.
*   Se um termo não tem forma normal, significa que ele pode ser reescrito indefinidamente (o sistema não é terminante para esse termo).
*   Se um termo tem múltiplas formas normais distintas, o sistema não é confluente (discutido abaixo).

**Reduções e Estratégias de Reescrita (e.g., `innermost`, `outermost`)**

Uma **redução** (ou sequência de reescrita) de um termo $t_0$ é uma sequência finita ou infinita $t_0 \rightarrow_R t_1 \rightarrow_R t_2 \rightarrow_R \ldots$.
Se um termo tem múltiplas subexpressões que podem ser reescritas (chamadas *redexes*, de *reducible expressions*), a escolha de qual redex reescrever primeiro é determinada por uma **estratégia de reescrita**. Algumas estratégias comuns incluem:

*   **Innermost (Mais Interna):** Sempre reescreve um redex que não contém outros redexes como subtermos. Corresponde à avaliação por valor (call-by-value) em programação funcional.
*   **Outermost (Mais Externa):** Sempre reescreve um redex que não é uma subexpressão de nenhum outro redex. Corresponde à avaliação por nome (call-by-name) ou avaliação preguiçosa (lazy evaluation).

A escolha da estratégia pode afetar se uma forma normal é encontrada (em sistemas não terminantes) e a eficiência da computação, mas não afeta a forma normal se o sistema for confluente (ver próxima seção).

## 3.3 Propriedades Fundamentais de Sistemas de Reescrita de Termos

Para que um SRT seja útil como modelo computacional ou para provar propriedades de uma especificação, ele deve idealmente possuir certas propriedades. As mais importantes são terminação e confluência.

**Terminação (Noetherianity)**

Um SRT $R$ é **terminante** (ou **Noetheriano**, ou **fortemente normalizador**) se não existem sequências de reescrita infinitas: $t_0 \rightarrow_R t_1 \rightarrow_R t_2 \rightarrow_R \ldots$.
Se um sistema é terminante, toda redução eventualmente para, e todo termo possui pelo menos uma forma normal.

Provar a terminação pode ser complexo. Técnicas comuns incluem o uso de **ordens de redução bem-fundadas** (well-founded reduction orderings). Uma ordem $>$ sobre termos é uma ordem de redução se ela é compatível com as operações da assinatura (i.e., $t > u \implies C[t] > C[u]$ para qualquer contexto $C$) e é fechada sob substituição (i.e., $t > u \implies t\sigma > u\sigma$). Se, para toda regra $l \rightarrow r$ em $R$, temos $l > r$ para alguma ordem de redução bem-fundada $>$, então $R$ é terminante. Exemplos incluem ordens polinomiais e a ordem de caminho recursiva (RPO).

A terminação é crucial, pois garante que o processo de "cálculo" por reescrita sempre termina.

**Confluência (Propriedade de Church-Rosser) e o Lema de Newman**

Um SRT $R$ é **confluente** (ou possui a **propriedade de Church-Rosser**) se, sempre que um termo $t$ pode ser reescrito para dois termos $u_1$ e $u_2$ (i.e., $t \rightarrow_R^* u_1$ e $t \rightarrow_R^* u_2$), então existe um termo $v$ tal que $u_1 \rightarrow_R^* v$ e $u_2 \rightarrow_R^* v$.
Intuïtivamente, a confluência significa que a ordem de aplicação das regras não importa para o resultado final (se ele existir). Se diferentes caminhos de reescrita divergem, eles eventualmente "reconvergem" para um termo comum.

Uma propriedade relacionada, mais fácil de verificar localmente, é a **confluência local** (ou **propriedade do diamante fraco**): $R$ é localmente confluente se, sempre que $t \rightarrow_R u_1$ e $t \rightarrow_R u_2$ (em um passo), então existe $v$ tal que $u_1 \rightarrow_R^* v$ e $u_2 \rightarrow_R^* v$.

O **Lema de Newman** é um resultado fundamental que conecta terminação e confluência local:
> **Lema 3.1: Lema de Newman**
>
> Um Sistema de Reescrita de Termos terminante é confluente se e somente se ele é localmente confluente.

Este lema é muito útil: para provar a confluência de um sistema terminante, basta verificar a confluência local, o que envolve analisar apenas os "picos críticos" que surgem de superposições entre lados esquerdos de regras.

Se um SRT é confluente, então cada termo tem no máximo uma forma normal. Se um termo tem uma forma normal, ela é única. Isso garante que o "resultado" da computação por reescita é unicamente definido, independentemente da estratégia de reescrita utilizada (desde que a estratégia encontre uma forma normal).

**Canonicidade (Convergência)**

Um SRT $R$ é **canônico** (ou **convergente**) se ele é ao mesmo tempo **terminante** e **confluente**.
Em um sistema canônico:
1.  Todo termo tem uma única forma normal.
2.  Qualquer sequência de reescrita a partir de um termo $t$ eventualmente termina nessa forma normal única.

A canonicidade é uma propriedade altamente desejável. Se um SRT derivado de uma especificação algébrica $SP = (\Sigma, E)$ é canônico, então a igualdade de dois termos $t_1, t_2$ (i.e., $E \vdash t_1 = t_2$) pode ser decidida reescrevendo $t_1$ e $t_2$ para suas formas normais únicas, $nf(t_1)$ e $nf(t_2)$, e verificando se $nf(t_1)$ e $nf(t_2)$ são sintaticamente idênticos. Isso fornece um procedimento de decisão para a teoria equacional definida por $E$.

## 3.4 Aplicação da Reescrita na Análise de Especificações Algébricas

A transformação de uma especificação algébrica $SP = (\Sigma, E)$ em um SRT $R$ canônico (se possível) tem implicações significativas:

1.  **Semântica Operacional:** $R$ fornece um modelo de como "calcular" com o TAD. A forma normal de um termo fundamental pode ser considerada o valor que ele representa.
2.  **Decidibilidade da Igualdade de Termos:** Se $R$ é canônico, a igualdade de termos $t_1 = t_2$ na teoria $E$ é decidível: basta computar suas formas normais $nf(t_1)$ e $nf(t_2)$ usando $R$ e verificar se são idênticas. Isso é crucial para verificar se uma implementação está correta em relação à especificação, ou para provar outras propriedades.
3.  **Verificação de Consistência:** Se, por exemplo, conseguirmos reescrever `true` e `false` (da especificação `Boolean`) para a mesma forma normal, isso indicaria uma inconsistência na especificação (e.g., $true=false$ é uma consequência dos axiomas). Se `true` e `false` têm formas normais distintas e irredutíveis, isso é uma evidência de consistência.
4.  **Prova de Teoremas:** Para provar uma equação $t_L = t_R$ que não está nos axiomas, podemos tentar mostrar que $nf(t_L)$ é idêntico a $nf(t_R)$.

O processo de transformar um conjunto de equações $E$ em um SRT $R$ canônico é o objetivo do algoritmo de **completação de Knuth-Bendix**. Este algoritmo tenta orientar equações, adicionar novas equações (deduzidas de picos críticos) e simplificar o sistema até que um sistema canônico seja obtido. No entanto, o algoritmo não tem garantia de terminar ou ter sucesso para todos os conjuntos de equações.

Nos capítulos seguintes, especialmente ao discutir a análise de especificações (Capítulo 4), veremos como as propriedades de terminação e confluência de um SRT associado são usadas para argumentar sobre a consistência e completeza da especificação algébrica original.

---
**Exercício 3.1:**
Considere o seguinte SRT sobre uma assinatura com uma constante `a` e uma operação unária `f`:
$R = \{ f(f(x)) \rightarrow x, f(a) \rightarrow a \}$
(a) O sistema é terminante? Justifique.
(b) O sistema é confluente? Se não, mostre um exemplo de divergência.

**Resolução do Exercício 3.1:**

(a) Terminação:
    Sim, o sistema é terminante.
    Seja $size(t)$ o número de ocorrências de `f` em $t$.
    Para a regra (R1) $f(f(x)) \rightarrow x$:
    $size(f(f(x))) = size(x) + 2$.
    $size(x) = size(x)$.
    Como $size(x) + 2 > size(x)$, a regra R1 diminui o tamanho do termo.
    Para a regra (R2) $f(a) \rightarrow a$:
    $size(f(a)) = 1$.
    $size(a) = 0$.
    Como $1 > 0$, a regra R2 diminui o tamanho do termo.
    Como cada aplicação de regra estritamente diminui o número de símbolos `f`, e o número de símbolos `f` é um número natural (que é bem-fundado), o sistema é terminante. Nenhuma sequência de reescrita pode ser infinita.

(b) Confluência:
    Sim, o sistema é confluente.
    Como o sistema é terminante (provado em (a)), podemos usar o Lema de Newman e verificar a confluência local analisando os picos críticos.
    As regras são:
    (R1) $f(f(x)) \rightarrow x$
    (R2) $f(a) \rightarrow a$

    Consideramos a superposição do lado esquerdo de R2, $f(a)$, na subexpressão $f(x)$ do lado esquerdo de R1, $f(f(x))$. Isso ocorre quando $x$ é instanciado como $a$, resultando no termo $f(f(a))$.
    1.  Reescrevendo $f(f(a))$ usando R1 (com $x \mapsto a$):
        $f(f(a)) \rightarrow_R a$.
    2.  Reescrevendo $f(f(a))$ aplicando R2 à subexpressão interna $f(a)$:
        $f(f(\underline{a})) \leftarrow f(\underline{f(a)}) \rightarrow_R f(a)$.
        Agora, o termo resultante $f(a)$ pode ser reescrito usando R2:
        $f(a) \rightarrow_R a$.

    O "pico" é $(a, f(a))$, ambos originados de $f(f(a))$.
    $f(f(a)) \rightarrow_R a$
    $f(f(a)) \rightarrow_R f(a)$
    Como $f(a) \rightarrow_R a$, ambos os caminhos convergem para $a$.
    Não há outras superposições que gerem picos críticos distintos. A superposição de R1 em si mesma (e.g., em $f(f(f(f(y))))$) também converge.
    $f(f(\underline{f(f(y))})) \rightarrow_R f(f(y)) \rightarrow_R y$.
    $\underline{f(f(f(f(y))))} \rightarrow_R f(f(y)) \rightarrow_R y$.
    Como o sistema é terminante e localmente confluente (todos os picos críticos convergem), pelo Lema de Newman, ele é confluente.
∎
---
**Exercício 3.2:**
Dado o SRT $R = \{ g(x,y) \rightarrow x, g(x,y) \rightarrow y \}$.
(a) O sistema é terminante?
(b) O sistema é confluente? Justifique.

**Resolução do Exercício 3.2:**

(a) Terminação:
    Sim, o sistema $R = \{ g(x,y) \rightarrow x, g(x,y) \rightarrow y \}$ é terminante.
    Ambas as regras, $g(x,y) \rightarrow x$ e $g(x,y) \rightarrow y$, reduzem estritamente o número de ocorrências do símbolo de função `g` no termo. A cada passo de reescrita, um `g` é eliminado. Como um termo possui um número finito de símbolos `g`, ele só pode ser reescrito um número finito de vezes.

(b) Confluência:
    Não, o sistema não é confluente.
    Considere o termo $t = g(a,b)$, onde `a` e `b` são constantes distintas (ou variáveis que não são redexes).
    1.  Aplicando a primeira regra $g(x,y) \rightarrow x$ com a substituição $\sigma_1 = \{x \mapsto a, y \mapsto b\}$:
        $g(a,b) \rightarrow_R a$.
    2.  Aplicando a segunda regra $g(x,y) \rightarrow y$ com a substituição $\sigma_2 = \{x \mapsto a, y \mapsto b\}$:
        $g(a,b) \rightarrow_R b$.

    Temos $t \rightarrow_R a$ e $t \rightarrow_R b$. Os termos $a$ e $b$ são irredutíveis (formas normais) se não houver outras regras para `a` ou `b`.
    Se $a \neq b$, então não existe um termo $v$ tal que $a \rightarrow_R^* v$ e $b \rightarrow_R^* v$, a menos que $v=a$ e $v=b$, o que é impossível se $a \neq b$.
    Como os caminhos de reescrita de $g(a,b)$ levam a formas normais distintas ($a$ e $b$) e não podem convergir para um termo comum, o sistema não é confluente.
∎
---

## 3.5 Exemplo Detalhado: Reescrita na Especificação de `Natural`

Vamos aplicar os conceitos de reescrita de termos à especificação `Natural` com as operações `zero`, `suc` e `add`, e os axiomas (N3) e (N4) para `add`.
Lembre-se dos axiomas para `add`:
*   (N3) `add(n, zero) = n`
*   (N4) `add(n, suc(m)) = suc(add(n, m))`

**Derivando Regras de Reescrita dos Axiomas de `add`**

Orientamos esses axiomas para formar as seguintes regras de reescita para um SRT $R_{Nat}$:
*   (R_N3) `add(x, zero) $\rightarrow$ x`
*   (R_N4) `add(x, suc(y)) $\rightarrow$ suc(add(x, y))`
(Usamos variáveis $x, y$ nas regras para distingui-las das variáveis $n, m$ nos axiomas, embora representem o mesmo papel).

**Demonstração de Sequências de Reescrita**

Vamos calcular a forma normal do termo `add(suc(zero), suc(zero))`, que representa $1+1$.
Termo inicial: $t_0 = \text{`add(suc(zero), suc(zero))`}$

1.  $t_0$ casa com o lado esquerdo de (R_N4), `add(x, suc(y))`, com a substituição $\sigma_1 = \{x \mapsto \text{`suc(zero)`}, y \mapsto \text{`zero`}\}$.
    O lado direito de (R_N4) instanciado é `suc(add(x,y))`$\sigma_1$ = `suc(add(suc(zero), zero))`.
    Então, $t_0 \rightarrow_{R_{Nat}} \text{`suc(add(suc(zero), zero))`}$ (vamos chamar este $t_1$).

2.  Agora consideramos $t_1 = \text{`suc(add(suc(zero), zero))`}$.
    A subexpressão `add(suc(zero), zero)` casa com o lado esquerdo de (R_N3), `add(x, zero)`, com a substituição $\sigma_2 = \{x \mapsto \text{`suc(zero)`}\}$.
    O lado direito de (R_N3) instanciado é `x`$\sigma_2$ = `suc(zero)`.
    Então, $t_1 \rightarrow_{R_{Nat}} \text{`suc(suc(zero))`}$ (vamos chamar este $t_2$).

3.  O termo $t_2 = \text{`suc(suc(zero))`}$ é irredutível com respeito a $R_{Nat}$, pois não contém nenhuma subexpressão que case com o lado esquerdo de (R_N3) ou (R_N4). As únicas operações são `suc` e `zero`, e as regras são para `add`.
    Portanto, `suc(suc(zero))` é a forma normal de `add(suc(zero), suc(zero))`.

Esta sequência de reescrita $t_0 \rightarrow_{R_{Nat}} t_1 \rightarrow_{R_{Nat}} t_2$ demonstra como a "soma" $1+1$ é "calculada" para resultar em $2$ (representado por `suc(suc(zero))`).

**Discussão sobre Terminação e Confluência para o sistema de exemplo**

Para o sistema $R_{Nat} = \{ \text{`add(x, zero)`} \rightarrow x, \text{`add(x, suc(y))`} \rightarrow \text{`suc(add(x, y))`} \}$:

*   **Terminação:**
    Este sistema é terminante. Uma maneira de ver isso é considerar uma ordem de redução onde `add(x, suc(y))` é maior que `suc(add(x, y))` porque o segundo argumento de `add` (ou seja, `y`) é estruturalmente menor que o segundo argumento original (`suc(y)`), e a recursão acontece nesse argumento. Formalmente, ordens como LPO (Lexicographic Path Ordering) podem provar isso.

*   **Confluência:**
    O sistema $R_{Nat}$ também é confluente. Como ele é terminante, podemos usar o Lema de Newman e verificar a confluência local analisando picos críticos.
    Não há sobreposição entre os lados esquerdos das duas regras `add(x,zero)` e `add(x,suc(y))` de forma a criar um pico crítico não trivial (eles diferem no segundo argumento fundamentalmente, um é `zero`, outro é `suc(y)`).
    As regras não se sobrepõem de maneira a criar divergências que não possam ser unidas. Portanto, o sistema é localmente confluente.
    Sendo terminante e localmente confluente, $R_{Nat}$ é confluente (e, portanto, canônico).

Isso significa que toda expressão envolvendo `add`, `suc` e `zero` tem uma forma normal única (um termo construído apenas com `suc` e `zero`), e qualquer estratégia de reescrita chegará a ela.


---
## REFERÊNCIAS BIBLIOGRÁFICAS

1.  BAADER, Franz; NIPKOW, Tobias. **Term Rewriting and All That.** Cambridge University Press, 1998. (ISBN 978-0521779203)
    *   *Resumo: Um livro texto abrangente e clássico sobre sistemas de reescrita de termos. Cobre todos os fundamentos, incluindo terminação, confluência, completação, estratégias de reescrita e aplicações em lógica e programação.*

2.  TERESE. **Term Rewriting Systems.** Cambridge University Press, 2003. (Cambridge Tracts in Theoretical Computer Science, Vol. 55) (ISBN 978-0521391153)
    *   *Resumo: Conhecido como o "livro do Terese" (um pseudônimo para uma equipe de autores), é uma referência enciclopédica e detalhada sobre praticamente todos os aspectos da teoria de reescrita de termos. Mais avançado, mas extremamente completo.*

3.  KLOP, Jan Willem. **Term Rewriting Systems.** In: Abramsky, S., Gabbay, D. M., Maibaum, T. S. E. (Eds.) *Handbook of Logic in Computer Science, Volume 2: Background: Computational Structures*. Oxford University Press, 1992, pp. 1-116. (ISBN 978-0198537618)
    *   *Resumo: Um capítulo de manual seminal que fornece uma introdução rigorosa e uma visão geral da reescrita de termos, com foco em suas propriedades e conexões com o cálculo lambda e outros modelos computacionais. Uma referência clássica.*

4.  DERSHOWITZ, Nachum; JOUANNAUD, Jean-Pierre. **Rewrite Systems.** In: van Leeuwen, J. (Ed.) *Handbook of Theoretical Computer Science, Volume B: Formal Models and Semantics*. Elsevier/MIT Press, 1990, pp. 243-320. (ISBN 978-0262220392)
    *   *Resumo: Outro capítulo de manual fundamental que cobre os aspectos teóricos dos sistemas de reescrita, incluindo ordens de terminação, prova de confluência (Lema de Newman, pares críticos) e completação de Knuth-Bendix. Altamente influente.*

5.  HUET, Gérard; OPPEN, Derek C. **Equations and Rewrite Rules: A Survey.** In: Book, R. (Ed.) *Formal Language Theory: Perspectives and Open Problems*. Academic Press, 1980, pp. 349-405.
    *   *Resumo: Um dos primeiros levantamentos abrangentes da área, este artigo clássico introduziu muitos conceitos e estabeleceu a terminologia. Embora mais antigo, ainda é valioso pelos insights fundamentais sobre a relação entre equações e regras de reescrita.*

6.  DERSHOWITZ, Nachum. **Termination of Rewriting.** *Journal of Symbolic Computation*, vol. 3, no. 1-2, 1987, pp. 69-116.
    *   *Resumo: Um artigo de pesquisa seminal focado especificamente no problema da terminação de sistemas de reescrita. Apresenta e analisa várias técnicas e ordens de terminação, incluindo ordens de caminho recursivas. Fundamental para quem quer se aprofundar em provas de terminação.*

---
As correções na formatação dos `backticks` nos axiomas e nas resoluções dos exercícios foram aplicadas. O Capítulo 3 está agora regenerado com essas mudanças.
