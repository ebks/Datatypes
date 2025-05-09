---

# CAPÍTULO 3

# Sistemas de Reescrita de Termos e Semântica Operacional de Especificações Algébricas

---

![imagem](tipo.png)

Nos capítulos anteriores, estabelecemos a importância da abstração de dados e introduzimos a especificação algébrica como um formalismo para definir Tipos Abstratos de Dados (TADs) através de assinaturas e axiomas equacionais. Enquanto as Σ-álgebras fornecem uma semântica denotacional (baseada em modelos matemáticos) para essas especificações, este capítulo explora uma perspectiva complementar e mais operacional: os Sistemas de Reescrita de Termos (SRTs), também conhecidos como *Term Rewriting Systems* (TRSs). Investigaremos como os axiomas equacionais de uma especificação podem ser interpretados como regras direcionais de transformação, permitindo "computar" ou "simplificar" termos até que uma forma mais fundamental, ou normal, seja alcançada. Discutiremos a relação intrínseca entre a reescrita de termos e o processo de computação. Serão apresentadas as definições formais de regras de reescrita, a relação de reescrita em um e múltiplos passos, e o conceito crucial de formas normais. Exploraremos propriedades fundamentais dos SRTs, como terminação (garantia de que toda sequência de reescrita termina) e confluência (garantia de que a ordem de aplicação das regras não afeta o resultado final, desde que se chegue a uma forma normal), culminando no conceito de canonicidade. Analisaremos como a reescrita de termos pode ser aplicada na análise de especificações algébricas, fornecendo uma maneira de "executar" a especificação. Por fim, um exemplo detalhado de reescrita sobre a especificação de Natural ilustrará a derivação de regras e a demonstração de sequências de reescrita, juntamente com uma discussão sobre terminação e confluência para o sistema de exemplo.

---

## 3.1 Introdução à Reescrita de Termos

A especificação algébrica, como vista no Capítulo 2, define um TAD através de um conjunto de equações. Essas equações, da forma $t_1 = t_2$, afirmam uma igualdade simétrica entre os termos $t_1$ e $t_2$. No entanto, para fins computacionais, muitas vezes é útil interpretar essas equações de forma direcional, como regras que permitem transformar um termo em outro, geralmente "mais simples" ou "mais canônico". Essa é a ideia central dos Sistemas de Reescrita de Termos (SRTs).

Um SRT consiste em um conjunto de *regras de reescrita*, onde cada regra $l \rightarrow r$ especifica que um termo que casa com o padrão $l$ pode ser substituído pelo termo $r$ instanciado apropriadamente.

**Motivação: Axiomas como Regras de Transformação**

Considere os axiomas da adição para o TAD Natural (do Capítulo 2):
*   (Axiom N3) add(zero, m) = m
*   (Axiom N4) add(suc(n), m) = suc(add(n, m))

Interpretados como regras:
*   Regra R1: add(zero, m) $\rightarrow$ m
*   Regra R2: add(suc(n), m) $\rightarrow$ suc(add(n, m))

Exemplo de cálculo: add(suc(zero), suc(zero))
add(suc(zero), suc(zero))
$\rightarrow_{\text{R2}}$ suc(add(zero, suc(zero)))
$\rightarrow_{\text{R1}}$ suc(suc(zero)) (forma normal)

**A Direcionalidade da Reescrita: De Termos Complexos a Termos Mais Simples**

A reescrita é direcional ($l \rightarrow r$, não $r \rightarrow l$). O objetivo é que $r$ seja "mais simples" que $l$. Equações como a comutatividade add(n, m) = add(m, n) são problemáticas para orientação simples.

**Relação entre Reescrita e Computação**

A reescrita de termos é um modelo fundamental de computação, ligado ao cálculo lambda e linguagens funcionais. Pode ser usada para provar igualdades em teorias equacionais.

---

## 3.2 Definições Formais em Sistemas de Reescrita de Termos (SRTs / TRSs)

**Regras de Reescrita ($l \rightarrow r$) derivadas de Axiomas Equacionais**

> **Definição 3.1: Regra de Reescrita**
>
> Uma **regra de reescrita** é $l \rightarrow r$, onde $l, r \in T_{\Sigma}(X)_s$, $l \notin X$, e $Vars(r) \subseteq Vars(l)$.

Um **Sistema de Reescrita de Termos (SRT)** é $(\Sigma, R)$, com $R$ um conjunto de regras.

**A Relação de Reescrita em Um Passo ($\rightarrow_R$)**

> **Definição 3.2: Relação de Reescrita em Um Passo ($\rightarrow_R$)**
>
> $t_1 \rightarrow_R t_2$ se existe $l \rightarrow r \in R$, posição $p$ em $t_1$, e substituição $\sigma$ tal que $t_1|_p = \sigma(l)$ e $t_2 = t_1[\sigma(r)]_p$.
> $t_1|_p$ é um *redex*.

**Exercício 3.1.A:**
Considere a assinatura $\Sigma$ com uma constante `k` e uma operação unária `h`.
Seja o SRT $(\Sigma, R)$ com as seguintes regras:
R1: h(h(x)) $\rightarrow$ x
R2: h(k) $\rightarrow$ k
Identifique todos os redexes no termo t = h(h(h(k))). Para cada redex, mostre o termo resultante após a reescita em um passo.

**Resolução 3.1.A:**
Termo $t = $ h(h(h(k))).
1.  Redex na raiz (correspondendo a R1): O subtermo $t|_{\epsilon} = $ h(h(h(k))) casa com h(h(x)) se $x = $ h(k).
    *   Substituição $\sigma_1 = \{x \mapsto \text{h(k)}\}$.
    *   $t \rightarrow_{\text{R1}} \sigma_1(x) = $ h(k).

2.  Redex no subtermo h(k) (correspondendo a R2): O subtermo $t|_{h(h(.))} = $ h(k) (isto é, o `h(k)` mais interno).
    *   Substituição $\sigma_2$ é vazia.
    *   h(h(h(k))) $\rightarrow_{\text{R2}}$ h(h(k)).

Portanto, temos duas possíveis reescritas em um passo a partir de h(h(h(k))):
*   h(h(h(k))) $\rightarrow_R$ h(k) (usando R1 na raiz)
*   h(h(h(k))) $\rightarrow_R$ h(h(k)) (usando R2 no subtermo h(k)) ■

**A Relação de Reescrita em Múltiplos Passos ($\rightarrow_R^*$ - Fecho Reflexivo e Transitivo)**

$\rightarrow_R^*$ é o fecho reflexivo e transitivo de $\rightarrow_R$.

**Formas Normais (Termos Irredutíveis)**

> **Definição 3.3: Termo Irredutível (Forma Normal)**
>
> $t$ é **irredutível** se não existe $t'$ tal que $t \rightarrow_R t'$.
> Se $t \rightarrow_R^* t^*$ e $t^*$ é irredutível, $t^*$ é uma **forma normal de $t$**.

**Reduções e Estratégias de Reescrita**

Uma *sequência de redução* é $t_0 \rightarrow_R t_1 \rightarrow_R \dots$.
*Estratégias de reescrita* (e.g., innermost, outermost) escolhem qual redex reescrever.

**Exercício 3.1:** (Original, renumerado para contexto)
SRT com regras: R1: f(g(x)) $\rightarrow$ x; R2: g(a) $\rightarrow$ b; R3: c $\rightarrow$ g(a).
Reescreva f(c) até uma forma normal.

**Resolução 3.1:**
f(c) $\rightarrow_{\text{R3}}$ f(g(a)) $\rightarrow_{\text{R1}}$ a. Forma normal: a. ■

**Exercício 3.2.A:**
Usando o SRT do Exercício 3.1.A (R1: h(h(x)) $\rightarrow$ x; R2: h(k) $\rightarrow$ k):
(a) Encontre a(s) forma(s) normal(is) do termo t = h(h(h(k))) usando uma estratégia *innermost*.
(b) Encontre a(s) forma(s) normal(is) do termo t = h(h(h(k))) usando uma estratégia *outermost*.
(c) O resultado é o mesmo?

**Resolução 3.2.A:**
Termo t = h(h(h(k))).
(a) Estratégia Innermost:
    O redex mais interno é h(k) (subtermo).
    1. h(h(h(k))) $\rightarrow_{\text{R2}}$ h(h(k)) (aplicando R2 ao h(k) mais interno).
    Agora o termo é h(h(k)). O único redex é h(h(k)) na raiz (para R1) ou h(k) como subtermo (para R2).
    Se considerarmos h(k) como o redex innermost (se não houver outros redexes dentro dele):
    2. h(h(k)) $\rightarrow_{\text{R2}}$ h(k) (aplicando R2 ao h(k) como subtermo).
    Agora o termo é h(k). O único redex é h(k) (para R2).
    3. h(k) $\rightarrow_{\text{R2}}$ k.
    k é forma normal.
    Sequência Innermost 1: h(h(h(k))) $\rightarrow$ h(h(k)) $\rightarrow$ h(k) $\rightarrow$ k.

    Alternativa no passo 2 (se R1 for considerado innermost no h(h(k)) porque R2 já foi aplicado "mais dentro"):
    2'. h(h(k)) $\rightarrow_{\text{R1}}$ k (aplicando R1 à raiz h(h(k)) com $x \mapsto k$).
    k é forma normal.
    Sequência Innermost 2: h(h(h(k))) $\rightarrow$ h(h(k)) $\rightarrow$ k.

    Ambas as sequências innermost levam a k.

(b) Estratégia Outermost:
    O redex outermost em h(h(h(k))) é h(h(h(k))) ele mesmo, que casa com h(h(x)) de R1, com $x \mapsto $ h(k).
    1. h(h(h(k))) $\rightarrow_{\text{R1}}$ h(k).
    Agora o termo é h(k). O redex outermost é h(k) (para R2).
    2. h(k) $\rightarrow_{\text{R2}}$ k.
    k é forma normal.
    Sequência Outermost: h(h(h(k))) $\rightarrow$ h(k) $\rightarrow$ k.

(c) Sim, o resultado (forma normal k) é o mesmo para ambas as estratégias neste caso. Isso sugere que o sistema pode ser confluente. ■

---

## 3.3 Propriedades Fundamentais de Sistemas de Reescrita de Termos

**Terminação (Noetherianity)**

> **Definição 3.4: Terminação (Noetherianity)**
>
> Um SRT é **terminante** se não existe sequência infinita $t_0 \rightarrow_R t_1 \rightarrow_R \dots$.

**Exercício 3.3.A:**
Para cada um dos seguintes SRTs (com uma única regra), determine se ele é terminante ou não. Justifique brevemente.
(a) R: f(x,y) $\rightarrow$ f(y,x)
(b) R: g(s(x)) $\rightarrow$ x (onde s é um construtor como 'sucessor')
(c) R: a $\rightarrow$ b; b $\rightarrow$ c
(d) R: k(x) $\rightarrow$ k(k(x))

**Resolução 3.3.A:**
(a) R: f(x,y) $\rightarrow$ f(y,x)
    *   **Não terminante.** f(a,b) $\rightarrow$ f(b,a) $\rightarrow$ f(a,b) $\rightarrow \dots$ (loop).
(b) R: g(s(x)) $\rightarrow$ x
    *   **Terminante.** O lado direito x é um subtermo próprio do argumento s(x) no lado esquerdo. A cada passo, um par g(s(...)) é removido, e o termo "diminui".
(c) R: a $\rightarrow$ b; b $\rightarrow$ c
    *   **Terminante.** Uma sequência de reescrita só pode ser a $\rightarrow$ b $\rightarrow$ c. O comprimento máximo é 2.
(d) R: k(x) $\rightarrow$ k(k(x))
    *   **Não terminante.** k(a) $\rightarrow$ k(k(a)) $\rightarrow$ k(k(k(a))) $\rightarrow \dots$ (crescimento infinito). ■

**Confluência (Propriedade de Church-Rosser) e o Lema de Newman**

> **Definição 3.5: Confluência (Propriedade de Church-Rosser)**
>
> Um SRT é **confluente** se para $t, t_1, t_2$ com $t \rightarrow_R^* t_1$ e $t \rightarrow_R^* t_2$, existe $t_3$ tal que $t_1 \rightarrow_R^* t_3$ e $t_2 \rightarrow_R^* t_3$.

Se confluente, formas normais são únicas.
**Confluência Local**:

> **Definição 3.6: Confluência Local**
>
> Se $t \rightarrow_R t_1$ e $t \rightarrow_R t_2$, existe $t_3$ tal que $t_1 \rightarrow_R^* t_3$ e $t_2 \rightarrow_R^* t_3$.

> **Teorema 3.1: Lema de Newman**
>
> SRT terminante é confluente $\iff$ é localmente confluente.

**Canonicidade (Convergência)**

> **Definição 3.7: Sistema de Reescrita Canônico (Convergente)**
>
> Um SRT é **canônico** se é terminante e confluente.

**Exercício 3.3.B:**
Considere o SRT com regras:
R1: f(x,x) $\rightarrow$ a
R2: c $\rightarrow$ d
R3: f(c,d) $\rightarrow$ b
(Assuma que a, b, c, d são constantes distintas e x é uma variável).
(a) O sistema é terminante?
(b) Analise o termo f(c,c). Mostre duas sequências de reescrita diferentes partindo dele.
(c) O sistema é confluente? Justifique.

**Resolução 3.3.B:**
(a) **Terminação:** Sim.
    *   R1: f(x,x) $\rightarrow$ a. O lado direito 'a' é mais simples (e.g., menor em tamanho ou profundidade) que f(x,x).
    *   R2: c $\rightarrow$ d. 'd' é mais simples que 'c' (ou mesma complexidade, mas é uma substituição finita).
    *   R3: f(c,d) $\rightarrow$ b. 'b' é mais simples que f(c,d).
    Não há como criar termos infinitamente crescentes ou ciclos óbvios.

(b) Análise de f(c,c):
    Sequência 1 (aplicando R2 primeiro, innermost):
    1. f(c,c) $\rightarrow_{\text{R2 no 1º arg}}$ f(d,c)
    2. f(d,c) $\rightarrow_{\text{R2 no 2º arg}}$ f(d,d)
    3. f(d,d) $\rightarrow_{\text{R1 com } x \mapsto d}$ a.
    Forma normal: a.

    Sequência 2 (aplicando R1 primeiro, outermost):
    1. f(c,c) $\rightarrow_{\text{R1 com } x \mapsto c}$ a.
    Forma normal: a.

(c) **Confluência:**
    Vamos analisar os "picos" ou divergências.
    Do termo f(c,c):
    *   f(c,c) $\rightarrow^*$ a (Sequência 2)
    *   f(c,c) $\rightarrow^*$ a (Sequência 1)
    Ambos levam a 'a'. Este caso particular converge.

    Consideremos o termo f(c,d). Ele só pode ser reescrito por R3 para b.
    Consideremos o termo f(x,y) onde x e y são distintos de c e d. Ele não pode ser reescrito.

    Vamos verificar a confluência local examinando pares críticos, que surgem de superposições.
    *   Não há superposição entre R1 e R2, R1 e R3, R2 e R3 nos lados esquerdos de forma que uma regra se aplique a um subtermo do LHS de outra.
    *   O único ponto onde um termo pode ser reescrito de múltiplas formas é quando há múltiplos redexes.
    Exemplo: t = f(f(c,c), c)
    Pico 1: f(f(c,c), c) $\rightarrow_{\text{R1 em f(c,c)}}$ f(a,c) (forma normal, assumindo 'a' e 'c' não formam redex com f)
    Pico 2: f(f(c,c), c) $\rightarrow_{\text{R2 em c}}$ f(f(c,c), d)
        $\rightarrow_{\text{R1 em f(c,c)}}$ f(a,d) (forma normal)
        $\rightarrow_{\text{R3 em f(c,d) se fosse o caso, mas aqui é f(a,d)}}$
    Como f(a,c) e f(a,d) são formas normais distintas que vieram do mesmo termo f(f(c,c),c), o sistema **não é confluente**.
    Mesmo que f(c,c) sozinho leve à mesma forma normal, a interação das regras em termos maiores pode levar a formas normais diferentes.

    A análise mais rigorosa de pares críticos:
    O LHS de R3 é f(c,d). O LHS de R1 é f(x,x).
    Podemos unificar f(c,d) com f(x,x)? Não, pois exigiria c=x e d=x, implicando c=d, o que contradiz a suposição de que são constantes distintas.
    Se as regras não se sobrepõem de maneira "crítica" e o sistema é terminante, a confluência local (e, portanto, a confluência) geralmente se mantém para sistemas simples como este. No entanto, meu exemplo acima com f(f(c,c), c) mostrou uma divergência.

    Revisando a análise do exemplo f(f(c,c), c):
    $t_0 = $ f(f(c,c), c)
    1. Reescrevendo o f(c,c) interno (usando R1 com $x \mapsto c$):
       $t_0 \rightarrow $ f(a, c). Esta é uma forma normal, pois 'a' e 'c' são constantes, e f(a,c) não casa com f(x,x) nem com f(c,d).
    2. Reescrevendo o 'c' externo (usando R2):
       $t_0 \rightarrow $ f(f(c,c), d).
       Agora, o f(c,c) interno pode ser reescrito (usando R1 com $x \mapsto c$):
       f(f(c,c), d) $\rightarrow $ f(a,d). Esta também é uma forma normal.

    Como $t_0$ pode ser reescrito para duas formas normais distintas (f(a,c) e f(a,d)), o sistema **não é confluente**. ■

---

## 3.4 Aplicação da Reescrita na Análise de Especificações Algébricas

SRTs dão semântica operacional a $SP = (\Sigma, E)$ orientando axiomas em regras $R_E$. Se $(\Sigma, R_E)$ for canônico:
1.  **Computação:** Termos são avaliados para formas normais.
2.  **Decisão de Igualdade:** $E \vdash t_1 = t_2$ sse $t_1\downarrow_{R_E} = t_2\downarrow_{R_E}$.
3.  **Análise de Propriedades:** Consistência, Completeza Suficiente.

**Exercício 3.2:** (Original, renumerado para contexto)
SRT para Point com newPoint, getX, getY e regras:
R1: getX(newPoint(x, y)) $\rightarrow$ x
R2: getY(newPoint(x, y)) $\rightarrow$ y
(a) Terminante? (b) Localmente confluente? (c) Canônico?

**Resolução 3.2:**
(a) Terminante: Sim.
(b) Localmente confluente: Sim.
(c) Canônico: Sim. ■

**Exercício 3.4.A:**
Considere a especificação algébrica de Boolean com as regras:
R_B1: not(true) $\rightarrow$ false
R_B2: not(false) $\rightarrow$ true
R_B3: and(true, b) $\rightarrow$ b
R_B4: and(false, b) $\rightarrow$ false
(a) O termo and(not(true), true) é redutível? Se sim, qual sua forma normal?
(b) O termo and(b, false) é redutível por estas regras? Se não, como isso se relaciona com a "completeza" da definição de 'and' por estas regras?

**Resolução 3.4.A:**
(a) Sim, and(not(true), true) é redutível.
    1. and(not(true), true) $\rightarrow_{\text{R_B1 no subtermo not(true)}}$ and(false, true)
    2. and(false, true) $\rightarrow_{\text{R_B4 com } b \mapsto true}$ false
    Forma normal: false.

(b) O termo and(b, false) não é redutível pelas regras R_B1 a R_B4. Nenhuma regra tem and(b, false) ou um padrão mais geral que case com ele como lado esquerdo (R_B3 e R_B4 casam no primeiro argumento de 'and').
    Isso se relaciona com a completeza da definição de 'and'. As regras R_B3 e R_B4 definem 'and' com base no valor do *primeiro* argumento. Elas não especificam diretamente o que acontece se o primeiro argumento for uma variável e o segundo for 'false' ou 'true'. Para ter uma definição mais completa que possa ser usada para simplificar termos como and(b, false) para false, precisaríamos de axiomas/regras adicionais (como and(b, false) $\rightarrow$ false) ou provar a comutatividade de 'and' e depois aplicar and(false,b) $\rightarrow$ false. ■

---

## 3.5 Exemplo Detalhado: Reescrita na Especificação de `Natural`

Regras para Natural:
R_isZero1: isZero(zero) $\rightarrow$ true
R_isZero2: isZero(suc(n)) $\rightarrow$ false
R_add1: add(zero, m) $\rightarrow$ m
R_add2: add(suc(n), m) $\rightarrow$ suc(add(n, m))

**Demonstração de Sequências de Reescrita**

1.  isZero(suc(zero)) $\rightarrow_{\text{R_isZero2}}$ false.
2.  add(suc(suc(zero)), suc(zero)) $\rightarrow_R \dots \rightarrow_R$ suc(suc(suc(zero))).

**Discussão sobre Terminação e Confluência**
O sistema {R_isZero1, R_isZero2, R_add1, R_add2} é terminante e confluente (canônico).

**Exercício 3.5.A:**
Usando as regras R_add1 e R_add2 para Natural, mostre todos os passos da reescrita do termo add(zero, suc(suc(zero))).

**Resolução 3.5.A:**
Termo inicial: add(zero, suc(suc(zero)))
1.  add(zero, suc(suc(zero)))
    $\rightarrow_{\text{R_add1 com } m \mapsto suc(suc(zero))}$ suc(suc(zero)).
A forma normal é suc(suc(zero)). A reescrita levou um passo. ■

**Exercício 3.5.B:**
Considere a adição da seguinte regra (incorreta) ao sistema de Natural:
R_err: add(n,m) $\rightarrow$ suc(n)
(a) O sistema continua terminante com R_err?
(b) Mostre uma sequência de reescrita para add(zero,zero) usando {R_add1, R_add2, R_err}.
(c) O sistema com R_err é confluente?

**Resolução 3.5.B:**
(a) **Terminação com R_err:**
    A regra R_err: add(n,m) $\rightarrow$ suc(n) pode levar a não-terminação se 'n' puder crescer ou se houver ciclos.
    Se aplicarmos R_err a add(zero,zero) $\rightarrow$ suc(zero). Isso termina.
    Se aplicarmos a add(suc(n),m) $\rightarrow$ suc(suc(n)). Isso parece terminar.
    No entanto, a regra R_err é muito genérica e pode interagir mal. Por exemplo, se tivéssemos uma regra como $x \rightarrow \text{add(x, zero)}$, então $a \rightarrow \text{add(a,zero)} \rightarrow_{\text{R_err}} \text{suc(a)}$, que pode ou não terminar dependendo de outras regras para 'suc'.
    Considerando apenas R_add1, R_add2 e R_err:
    add(suc(n), m) $\rightarrow_{\text{R_add2}}$ suc(add(n,m))
    add(suc(n), m) $\rightarrow_{\text{R_err}}$ suc(suc(n))
    Se suc(add(n,m)) puder ser reescrito para algo contendo add(n',m'), pode haver não-terminação.
    Por si só, add(n,m) $\rightarrow$ suc(n) parece terminante, pois 'add' é removido. O lado direito suc(n) é geralmente considerado mais simples que add(n,m) em ordens de redução típicas. Sim, parece terminante.

(b) Sequência para add(zero,zero) usando {R_add1, R_err} (R_add2 não se aplica):
    Opção 1 (usando R_add1):
    add(zero,zero) $\rightarrow_{\text{R_add1 com } m \mapsto zero}$ zero. (Forma normal)

    Opção 2 (usando R_err):
    add(zero,zero) $\rightarrow_{\text{R_err com } n \mapsto zero, m \mapsto zero}$ suc(zero). (Forma normal)

(c) **Confluência com R_err:**
    Do item (b), o termo add(zero,zero) pode ser reescrito para duas formas normais distintas: zero e suc(zero).
    Como $t_0 = $ add(zero,zero) $\rightarrow_R^*$ zero e $t_0 \rightarrow_R^*$ suc(zero), e zero $\neq$ suc(zero) (são formas normais distintas), o sistema **não é confluente**. ■

---

Este capítulo introduziu os Sistemas de Reescrita de Termos como uma forma de dar uma interpretação computacional às especificações algébricas. Definimos regras de reescrita, a relação de reescrita, e formas normais. Discutimos as propriedades cruciais de terminação e confluência, que juntas levam à canonicidade, garantindo que a reescrita sempre termina com um resultado único. A aplicação da reescrita na análise de especificações e um exemplo detalhado com o TAD Natural ilustraram esses conceitos. A reescrita de termos será uma ferramenta implícita em nossa compreensão de como as especificações definem o comportamento computável dos TADs. No próximo capítulo, nos aprofundaremos na análise das próprias especificações algébricas, investigando propriedades desejáveis como consistência e completeza.

---
## REFERÊNCIAS BIBLIOGRÁFICAS

1.  BAADER, Franz; NIPKOW, Tobias. **Term Rewriting and All That.** Cambridge: Cambridge University Press, 1998.
    *   *Resumo: Um texto fundamental e abrangente sobre sistemas de reescrita de termos. Cobre os fundamentos teóricos como ordens de reescrita, terminação, confluência (Lema de Newman, pares críticos), e completion (procedimento de Knuth-Bendix).*

2.  DERSHOWITZ, Nachum; JOUANNAUD, Jean-Pierre. Rewrite Systems. In: VAN LEEUWEN, Jan (Ed.). **Handbook of Theoretical Computer Science, Volume B: Formal Models and Semantics.** Amsterdam: Elsevier e Cambridge, MA: MIT Press, 1990. p. 243-320.
    *   *Resumo: Um capítulo de manual seminal que oferece uma visão geral concisa e profunda da teoria dos sistemas de reescrita de termos, incluindo suas propriedades e aplicações em ciência da computação, como prova de teoremas e semântica de linguagens de programação.*

3.  KLOP, Jan Willem. Term Rewriting Systems. In: ABRAMSKY, S.; GABBAI, D. M.; MAIBAUM, T. S. E. (Eds.). **Handbook of Logic in Computer Science, Volume 2: Background: Computational Structures.** Oxford: Oxford University Press, 1992. p. 1-116.
    *   *Resumo: Outra referência clássica e detalhada sobre SRTs, cobrindo uma vasta gama de tópicos, desde os fundamentos até sistemas mais avançados como reescrita de ordem superior e reescrita condicional. Conhecido por sua profundidade e rigor.*

4.  HUET, Gérard; OPPEN, Derek C. Equations and Rewrite Rules: A Survey. In: BOOK, Ronald V. (Ed.). **Formal Language Theory: Perspectives and Open Problems.** New York: Academic Press, 1980. p. 349-405.
    *   *Resumo: Um survey influente e um dos primeiros a consolidar a teoria dos sistemas de reescrita e sua relação com a decisão de teorias equacionais. Introduziu muitos conceitos chave e estabeleceu a área.*

5.  TERESE. **Term Rewriting Systems.** Cambridge: Cambridge University Press, 2003. (Cambridge Tracts in Theoretical Computer Science, Vol. 55).
    *   *Resumo: Um livro mais moderno e extremamente completo, escrito por um coletivo de especialistas ("Terese" é um pseudônimo). Cobre praticamente todos os aspectos da reescrita de termos, desde o básico até tópicos de pesquisa avançados, com muitos exemplos e exercícios.*

6.  VAN OOSTEROM, Vincent; VAN RAAMSDONK, Femke. **Visualising Combinatory Logic and Term Rewriting with Term Tapper.** Electronic Notes in Theoretical Computer Science, v. 352, p. 145-151, 2020.
    *   *Resumo: Um artigo mais recente que, embora focado em uma ferramenta de visualização, ilustra a relevância contínua da reescrita de termos e sua aplicação em contextos educacionais e de pesquisa, mostrando como a visualização pode ajudar na compreensão de sequências de reescrita.*

---
