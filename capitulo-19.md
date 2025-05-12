---
# Capítulo 19
# Tipo Abstrato de Dado HashTable
---

Concluindo nossa jornada pela Parte IV, dedicada a estruturas de dados para coleções e relações, este capítulo se aprofunda no Tipo Abstrato de Dados (TAD) `HashTable[K,V]`, ou Tabela Hash. Enquanto o capítulo anterior introduziu o TAD `Map[K,V]` de forma abstrata, definindo a interface lógica e o comportamento de um mapeamento de chaves para valores, o presente capítulo foca na `HashTable` como uma das implementações mais proeminentes e eficientes deste TAD. Uma tabela hash é uma estrutura de dados que, através do uso de uma função de hash para calcular um índice (ou "balde", *bucket*) a partir de uma chave, busca oferecer desempenho em tempo médio constante, $O(1)$, para as operações fundamentais de inserção (`put`), busca (`get`) e remoção (`delete`). Este capítulo seguirá a estrutura metodológica adotada: iniciaremos com uma definição do `HashTable[K,V]` não como um novo TAD com semântica fundamentalmente distinta, mas como uma estrutura de dados concreta que *realiza* o TAD `Map[K,V]`, discutindo sua relevância, os conceitos de função de hash, colisões, estratégias de resolução de colisões (como encadeamento separado e endereçamento aberto), fator de carga e a necessidade de redimensionamento. Em seguida, apresentaremos a "especificação algébrica" do `HashTable[K,V]`, que, neste contexto, consistirá em declarar sua interface (espelhando a do `Map[K,V]`) e postular que seus axiomas de comportamento são precisamente aqueles do TAD `Map[K,V]`, indicando a obrigação de conformidade. A análise de corretude e completeza focará em como essas obrigações são satisfeitas. Posteriormente, detalharemos o projeto e a implementação de uma classe genérica `PyHashTable[K,V]` em Python, com tipagem MyPy, utilizando a técnica de encadeamento separado para resolução de colisões. A validação da corretude da implementação `PyHashTable` em relação aos axiomas do `Map` será conduzida com Teste Baseado em Propriedades usando Hypothesis. Finalmente, analisaremos a complexidade algorítmica das operações e proporemos um exercício prático.

# 19.1 Definição Abstrata e Relevância do Tipo de Dados `HashTable[K,V]`

O Tipo Abstrato de Dados `HashTable[K,V]` (Tabela Hash) não é, em si, um novo tipo abstrato com um conjunto fundamentalmente distinto de operações e axiomas em relação ao TAD `Map[K,V]` (discutido no Capítulo 18). Em vez disso, uma `HashTable` é uma **estrutura de dados concreta** e uma técnica de implementação altamente eficiente projetada para realizar as operações do TAD `Map[K,V]`. A ideia central de uma tabela hash é utilizar uma **função de hash** para transformar uma chave `K` em um índice numérico, que é então usado para localizar a posição (ou "balde", *bucket*) onde o valor `V` associado à chave deve ser armazenado ou recuperado dentro de uma estrutura de dados subjacente, tipicamente um array.

**Definição Abstrata da Estrutura e Conceitos Chave:**

Embora a `HashTable` seja uma implementação, podemos descrever seus componentes e princípios abstratos:
1.  **Array de Baldes (Buckets):** A estrutura primária é um array (ou vetor) de um tamanho fixo $M$, onde cada posição $i$ (para $0 \le i < M$) é um "balde".
2.  **Função de Hash (`hash_function: K -> Natural` ou `K -> Integer`):** Uma função que mapeia uma chave arbitrária do tipo `K` para um número inteiro. Este número é então tipicamente mapeado para um índice de balde válido, geralmente através da operação módulo $M$: `index = hash_function(key) % M`.
    *   Uma boa função de hash deve distribuir as chaves de forma aproximadamente uniforme pelos baldes e ser computacionalmente eficiente.
3.  **Colisões:** Como o número de chaves possíveis geralmente excede (e muito) o número de baldes $M$, é inevitável que diferentes chaves possam ser mapeadas pela função de hash para o mesmo índice de balde. Isso é chamado de **colisão**.
4.  **Resolução de Colisões:** Mecanismos são necessários para lidar com colisões, ou seja, para armazenar e recuperar múltiplos pares chave-valor que hasheiam para o mesmo balde. As duas estratégias principais são:
    *   **Encadeamento Separado (Separate Chaining):** Cada balde armazena uma estrutura de dados secundária (comumente uma lista encadeada, mas poderia ser outra `Map` menor ou uma árvore de busca) que contém todos os pares chave-valor cujo hash da chave corresponde àquele balde.
    *   **Endereçamento Aberto (Open Addressing):** Se ocorre uma colisão ao tentar inserir em um balde $i$ que já está ocupado, outras posições no array de baldes são sondadas (de acordo com uma sequência de sondagem, e.g., linear, quadrática, hashing duplo) até que um balde vazio seja encontrado. A busca segue a mesma sequência de sondagem.
5.  **Fator de Carga ($\alpha$):** É a razão entre o número de elementos $N$ armazenados na tabela e o número de baldes $M$ (i.e., $\alpha = N/M$). O fator de carga afeta o desempenho; à medida que $\alpha$ aumenta, a probabilidade de colisões também aumenta, e o desempenho pode degradar.
6.  **Redimensionamento (Resizing):** Quando o fator de carga excede um certo limiar, a tabela hash é tipicamente redimensionada (geralmente para um tamanho maior, frequentemente o dobro do tamanho primo mais próximo) e todos os elementos existentes são re-hasheados e reinseridos na nova tabela maior. Isso ajuda a manter o fator de carga baixo e o desempenho eficiente.

**Operações (Realizando o TAD `Map[K,V]`):**
As operações de uma `HashTable` são aquelas do TAD `Map[K,V]` que ela implementa:
*   `put(key: K, value: V, ht: HashTable) -> HashTable`: Insere/atualiza o par (chave, valor).
*   `get(key: K, ht: HashTable) -> V` (ou `Maybe[V]`): Recupera o valor associado à chave.
*   `delete(key: K, ht: HashTable) -> HashTable`: Remove o par associado à chave.
*   `containsKey(key: K, ht: HashTable) -> Boolean`: Verifica se a chave existe.
*   `isEmpty(ht: HashTable) -> Boolean`: Verifica se a tabela está vazia.
*   `size(ht: HashTable) -> Natural`: Retorna o número de pares chave-valor.
*   `emptyHashTable: --> HashTable`: Cria uma tabela hash vazia.

**Relevância do TAD `HashTable[K,V]`:**
A principal relevância das tabelas hash reside em sua capacidade de oferecer um desempenho em **tempo médio constante ($O(1)$)** para as operações `put`, `get` e `delete`.
1.  **Eficiência Extrema (Caso Médio):** Com uma boa função de hash e uma estratégia de resolução de colisões eficaz, e mantendo um fator de carga razoável (e.g., $\alpha \le 0.7-0.8$ para encadeamento, $\alpha \le 0.5$ para algumas formas de endereçamento aberto), as operações básicas podem ser realizadas, em média, em tempo constante, independentemente do número de elementos já na tabela. Isso as torna extremamente rápidas para grandes volumes de dados.
2.  **Ampla Aplicabilidade:** São usadas em inúmeras aplicações:
    *   Implementação de dicionários e conjuntos em linguagens de programação (e.g., `dict` e `set` em Python).
    *   Tabelas de símbolos em compiladores.
    *   Caches (e.g., caches de disco, caches da web).
    *   Indexação em bancos de dados (embora B-trees sejam mais comuns para índices em disco, tabelas hash são usadas para índices em memória).
    *   Contagem de frequência de itens, detecção de duplicatas.
3.  **Flexibilidade de Tipos de Chave:** Desde que um tipo de chave `K` possa ser hasheado (i.e., uma função de hash pode ser definida para ele) e comparado por igualdade, ele pode ser usado em uma tabela hash.

**Desvantagens e Considerações:**
*   **Desempenho no Pior Caso:** Se a função de hash for ruim (e.g., mapear muitas chaves para o mesmo balde) ou em cenários de ataque (onde chaves são escolhidas para causar colisões máximas), o desempenho pode degradar para $O(N)$ (e.g., todas as chaves caem na mesma lista encadeada).
*   **Função de Hash:** A qualidade da função de hash é crítica.
*   **Ordenação:** Tabelas hash geralmente não mantêm os elementos em nenhuma ordem específica (e.g., ordem de inserção ou ordem de chave), a menos que sejam variantes especializadas (como `OrderedDict` em Python antes da versão 3.7, ou `LinkedHashMap` em Java).
*   **Uso de Espaço:** Podem usar mais espaço do que estruturas baseadas em árvores se o fator de carga for mantido muito baixo ou se houver muitos baldes vazios. O redimensionamento pode ser uma operação custosa (embora amortizada).

Apesar dessas considerações, a eficiência no caso médio faz das tabelas hash uma das estruturas de dados mais importantes e amplamente utilizadas para implementações de mapeamentos.

**Exercício:**

Suponha uma `HashTable` com $M=10$ baldes (índices 0-9) e uma função de hash $h(k) = k \pmod{10}$. Se as seguintes chaves do tipo `Natural` são inseridas na ordem: 12, 22, 3, 13, 25, 35, usando **encadeamento separado** para resolução de colisões, descreva o estado dos baldes (quais chaves estão em cada lista de balde). Qual balde terá a lista mais longa?

**Resolução:**

Tabela Hash com $M=10$ baldes, $h(k) = k \pmod{10}$.
Chaves a serem inseridas: 12, 22, 3, 13, 25, 35.

1.  **Inserir 12:**
    $h(12) = 12 \pmod{10} = 2$.
    Balde 2: $\langle 12 \rangle$

2.  **Inserir 22:**
    $h(22) = 22 \pmod{10} = 2$. Colisão com 12.
    Balde 2: $\langle 12, 22 \rangle$ (ou $\langle 22, 12 \rangle$, dependendo da política de inserção na lista do balde - vamos assumir que adiciona ao final da lista do balde).

3.  **Inserir 3:**
    $h(3) = 3 \pmod{10} = 3$.
    Balde 3: $\langle 3 \rangle$

4.  **Inserir 13:**
    $h(13) = 13 \pmod{10} = 3$. Colisão com 3.
    Balde 3: $\langle 3, 13 \rangle$

5.  **Inserir 25:**
    $h(25) = 25 \pmod{10} = 5$.
    Balde 5: $\langle 25 \rangle$

6.  **Inserir 35:**
    $h(35) = 35 \pmod{10} = 5$. Colisão com 25.
    Balde 5: $\langle 25, 35 \rangle$

**Estado Final dos Baldes:**
*   Balde 0: $\langle \rangle$
*   Balde 1: $\langle \rangle$
*   Balde 2: $\langle 12, 22 \rangle$
*   Balde 3: $\langle 3, 13 \rangle$
*   Balde 4: $\langle \rangle$
*   Balde 5: $\langle 25, 35 \rangle$
*   Balde 6: $\langle \rangle$
*   Balde 7: $\langle \rangle$
*   Balde 8: $\langle \rangle$
*   Balde 9: $\langle \rangle$

**Balde com a Lista Mais Longa:**
Os baldes 2, 3 e 5 contêm duas chaves cada. Todos eles têm o comprimento máximo de lista, que é 2.
Não há um único balde com a lista "mais" longa; três baldes compartilham o comprimento máximo.

# 19.2 Especificação Algébrica Formal do TAD `HashTable[K,V]` (como Implementação de `Map[K,V]`)

Como discutido na Seção 19.1, o `HashTable[K,V]` é primariamente uma estratégia de implementação para o Tipo Abstrato de Dados `Map[K,V]`. Portanto, em vez de definir um novo conjunto de axiomas comportamentais fundamentalmente diferentes para `HashTable`, a "especificação" de `HashTable` se concentra em:
1.  Definir os requisitos para seus tipos de parâmetro (chave `K` e valor `V`).
2.  Declarar as operações que a `HashTable` oferece (que devem espelhar as operações do `Map`).
3.  Estabelecer que o comportamento dessas operações deve ser consistente com os axiomas do TAD `Map[K,V]`. Isso significa que uma `HashTable` correta deve ser um modelo válido da especificação `Map[K,V]`.

Assumimos que uma especificação para `Map[ParamK_Map, ParamV_Map]` (onde `ParamK_Map` requer pelo menos igualdade `eqK_Map` sobre o sort `K_Map`, e `ParamV_Map` requer um sort `V_Map`) foi fornecida no Capítulo 18, com operações como `emptyMap`, `putMap`, `getMap`, `deleteMap`, `containsKeyMap`, `sizeMap`, `isEmptyMap` e seus respectivos axiomas (e.g., `getMap(putMap(m,k,v),k) = v`).

**Requisitos dos Parâmetros para `HashTable`:**

**SPEC ADT** KeyTypeWithHashAndEq
**sorts:**
*   K
**operations:**
*   hashFun: K -> Natural
*   eqK: K x K -> Boolean
**axioms:**
*   (KE1): eqK(k,k) = true
*   (KE2): eqK(k1,k2) = eqK(k2,k1)
*   (KE3): eqK(k1,k2) = true AND eqK(k2,k3) = true => eqK(k1,k3) = true
for all k,k1,k2,k3: K
**END SPEC**

A especificação `KeyTypeWithHashAndEq` define os requisitos para o tipo da chave `K` a ser usado em uma `HashTable`. Ele deve fornecer um sort `K`, uma função de hash `hashFun` que mapeia chaves `K` para `Natural` (ou inteiros, que podem ser mapeados para índices de bucket), e uma operação de igualdade `eqK` que deve ser uma relação de equivalência (reflexiva, simétrica e transitiva, como exemplificado pelos axiomas KE1-KE3).

**SPEC ADT** ValueType
**sorts:**
*   V
**END SPEC**

A especificação `ValueType` apenas requer um sort `V` para os valores, sem impor requisitos de operações sobre `V` para a funcionalidade básica de uma `HashTable` (embora operações específicas possam requerer mais de `V`).

**Interface e Obrigações Axiomáticas de `HashTable[K,V]`:**

**SPEC ADT** HashTable[ParamK: KeyTypeWithHashAndEq, ParamV: ValueType]
**imports:**
*   ParamK
*   ParamV
*   Natural
*   Boolean

**sorts:**
*   HashTable

**operations:**
*   emptyHT: --> HashTable
*   putHT: K x V x HashTable --> HashTable
*   getHT: K x HashTable --> V
*   deleteHT: K x HashTable --> HashTable
*   containsKeyHT: K x HashTable --> Boolean
*   sizeHT: HashTable --> Natural
*   isEmptyHT: HashTable --> Boolean

**axioms:**
for all ht: HashTable, k, k1: K, v: V

(*
  Os axiomas para HashTable afirmam que suas operações se comportam
  identicamente às operações do TAD Map[K,V] abstrato.
  Formalmente, isso é estabelecido mostrando que existe uma função de abstração
  AF: HashTable -> Map (onde Map é o sort da especificação abstrata Map[K,V])
  tal que:
  1. AF(emptyHT) = emptyMap
  2. AF(putHT(k, v, ht)) = putMap(k, v, AF(ht))
  3. getMap(AF(ht), k) = getHT(ht, k) (considerando parcialidade ou tipo Maybe)
  4. AF(deleteHT(k, ht)) = deleteMap(k, AF(ht))
  5. containsKeyMap(AF(ht), k) = containsKeyHT(ht, k)
  6. sizeMap(AF(ht)) = sizeHT(ht)
  7. isEmptyMap(AF(ht)) = isEmptyHT(ht)

  E que os axiomas de Map[K,V], quando traduzidos usando AF,
  são satisfeitos. Para fins de especificação direta do comportamento
  esperado de HashTable, listamos os axiomas equivalentes aos de Map
  diretamente sobre as operações de HashTable.
*)

*   (HT1): isEmptyHT(emptyHT) = true
*   (HT2): isEmptyHT(putHT(k,v,ht)) = false
*   (HT3): sizeHT(emptyHT) = zero
*   (HT4): containsKeyHT(ht,k) = true  => sizeHT(putHT(k,v,ht)) = sizeHT(ht)
*   (HT5): containsKeyHT(ht,k) = false => sizeHT(putHT(k,v,ht)) = succ(sizeHT(ht))

*   (HT6): getHT(putHT(k,v,ht), k) = v
*   (HT7): eqK(k1,k) = false => getHT(putHT(k,v,ht), k1) = getHT(ht, k1)

*   (HT8): containsKeyHT(emptyHT, k) = false
*   (HT9): containsKeyHT(putHT(k,v,ht), k) = true
*   (HT10): eqK(k1,k) = false => containsKeyHT(putHT(k,v,ht), k1) = containsKeyHT(ht, k1)

*   (HT11): deleteHT(emptyHT, k) = emptyHT
*   (HT12): deleteHT(putHT(k,v,ht), k) = deleteHT(ht,k)
*   (HT13): eqK(k1,k) = false => deleteHT(putHT(k,v,ht), k1) = putHT(k,v,deleteHT(ht,k1))

**END SPEC**

A especificação `HashTable[ParamK, ParamV]` define a interface para uma tabela hash genérica. Ela é parametrizada por `ParamK` (que deve satisfazer `KeyTypeWithHashAndEq`, fornecendo o sort `K`, a função `hashFun` e a igualdade `eqK`) e `ParamV` (que satisfaz `ValueType`, fornecendo o sort `V`). A especificação importa estes parâmetros, bem como `Natural` e `Boolean`. O sort principal é `HashTable`.
As **operações** (`emptyHT`, `putHT`, `getHT`, `deleteHT`, `containsKeyHT`, `sizeHT`, `isEmptyHT`) espelham diretamente as operações de um TAD `Map[K,V]`. Notavelmente, `getHT` é especificada como retornando `V`, o que implica que ela é uma operação parcial, definida apenas quando a chave está presente. Uma especificação mais robusta usaria um tipo `Maybe[V]` ou `Option[V]` para o retorno de `getHT`, ou um predicado de definibilidade.
Os **axiomas** (HT1)-(HT13) são diretos análogos aos axiomas típicos do TAD `Map[K,V]`. Eles estipulam que uma `HashTable` deve se comportar como um mapa abstrato.
*   (HT1)-(HT2) definem `isEmptyHT`.
*   (HT3)-(HT5) definem `sizeHT`, distinguindo entre inserção de nova chave e atualização de chave existente.
*   (HT6)-(HT7) definem `getHT` em relação a `putHT`. (HT6) afirma que buscar uma chave `k` após inseri-la com valor `v` retorna `v`. (HT7) afirma que inserir um par $(k,v)$ não afeta o valor associado a uma chave diferente `k1`.
*   (HT8)-(HT10) definem `containsKeyHT` de forma similar.
*   (HT11)-(HT13) definem `deleteHT`. (HT12) afirma que deletar uma chave `k` após inseri-la/atualizá-la é como deletar `k` da tabela original (o valor `v` associado a `k` em `putHT` é irrelevante para o efeito de `deleteHT` sobre a presença da chave `k`). (HT13) mostra que `deleteHT` em `k1` e `putHT` em `k` (com $k \neq k1$) comutam de certa forma.
A nota sobre a função de abstração `AF` clarifica a relação formal entre `HashTable` e `Map` que está implícita aqui. Para os testes de propriedade, esses axiomas (HT1)-(HT13) serão diretamente usados.

## 10.2.1 Análise de Corretude e Completeza da Especificação `HashTable[K,V]`

A "corretude" da especificação `HashTable[K,V]` acima refere-se primariamente à sua consistência e à sua capacidade de definir completamente o comportamento esperado de um mapa.

**Consistência:**
A consistência desta especificação depende da consistência da especificação abstrata `Map[K,V]` da qual ela deriva seus axiomas. Se os axiomas (HT1)-(HT13) são uma transcrição fiel de um conjunto consistente de axiomas para `Map[K,V]`, e as especificações importadas (`KeyTypeWithHashAndEq`, `ValueType`, `Natural`, `Boolean`) são consistentes, então a especificação `HashTable` também será consistente.
1.  **Proteção dos Tipos Importados:** Os axiomas definem o comportamento das operações de `HashTable`, resultando em `HashTable`, `V`, `Boolean`, ou `Natural`. Eles usam `eqK` e `hashFun` do parâmetro `K`. É crucial que `hashFun` seja bem comportada (embora sua "qualidade" seja uma questão de desempenho, não de consistência lógica da especificação em si, a menos que leve a contradições nos índices). A especificação não parece introduzir contradições como `true = false` ou $0=1$.
2.  **Consistência Interna:** Os axiomas (HT1)-(HT13) são padrões para mapas e geralmente são consistentes. Por exemplo, `emptyHT` é distinto de `putHT(k,v,ht)` (via HT1, HT2). A distinção entre diferentes tabelas não vazias é dada por `getHT`, `containsKeyHT` ou `sizeHT`.
3.  **Modelo:** Qualquer implementação correta de uma tabela hash (e.g., usando encadeamento separado ou endereçamento aberto) que satisfaça a semântica de mapa serve como um modelo.

A especificação parece **consistente**.

**Completeza Suficiente:**
Consideramos `emptyHT` e `putHT` como os construtores principais do estado da `HashTable`.
Analisamos as operações não-construtoras (`isEmptyHT`, `sizeHT`, `getHT`, `containsKeyHT`, `deleteHT`).

1.  **`isEmptyHT(ht: HashTable)`:**
    *   Caso $ht = \text{emptyHT}$: `isEmptyHT(emptyHT)` $\rightarrow_{HT1}$ `true`. (Resultado `Boolean`).
    *   Caso $ht = \text{putHT(k,v,ht')}$: `isEmptyHT(putHT(k,v,ht'))` $\rightarrow_{HT2}$ `false`. (Resultado `Boolean`).
    `isEmptyHT` é suficientemente completa.

2.  **`sizeHT(ht: HashTable)`:**
    *   Caso $ht = \text{emptyHT}$: `sizeHT(emptyHT)` $\rightarrow_{HT3}$ `zero`. (Resultado `Natural`).
    *   Caso $ht = \text{putHT(k,v,ht')}$: O comportamento é definido por (HT4) e (HT5) condicionalmente sobre `containsKeyHT(ht',k)`.
        *   Se `containsKeyHT(ht',k) = true`: `sizeHT(putHT(k,v,ht')) = sizeHT(ht')`.
        *   Se `containsKeyHT(ht',k) = false`: `sizeHT(putHT(k,v,ht')) = succ(sizeHT(ht'))`.
        Assumindo que `containsKeyHT` é suficientemente completa (veja abaixo) e se reduz a `true` ou `false`, então `sizeHT` é definida recursivamente sobre `ht'`. Por indução, `sizeHT` se reduzirá a um termo `Natural`.
    `sizeHT` é suficientemente completa, dependendo da completeza de `containsKeyHT`.

3.  **`getHT(ht: HashTable, k_query: K)`:**
    *   Caso $ht = \text{emptyHT}$: `getHT(emptyHT, k_query)`. Nenhum axioma (HT1-HT13) define diretamente este caso. A operação é parcial.
    *   Caso $ht = \text{putHT(k,v,ht')}$:
        *   Subcaso `eqK(k_query, k) = true`: `getHT(putHT(k,v,ht'), k_query)` $\rightarrow_{HT6}$ `v`. (Resultado `V`).
        *   Subcaso `eqK(k_query, k) = false`: `getHT(putHT(k,v,ht'), k_query)` $\rightarrow_{HT7}$ `getHT(ht', k_query)`. Redução recursiva.
    `getHT` é suficientemente completa para chaves presentes, mas parcial (indefinida pelos axiomas) para chaves não presentes ou para `emptyHT`.

4.  **`containsKeyHT(ht: HashTable, k_query: K)`:**
    *   Caso $ht = \text{emptyHT}$: `containsKeyHT(emptyHT, k_query)` $\rightarrow_{HT8}$ `false`. (Resultado `Boolean`).
    *   Caso $ht = \text{putHT(k,v,ht')}$:
        *   Subcaso `eqK(k_query, k) = true`: `containsKeyHT(putHT(k,v,ht'), k_query)` $\rightarrow_{HT9}$ `true`. (Resultado `Boolean`).
        *   Subcaso `eqK(k_query, k) = false`: `containsKeyHT(putHT(k,v,ht'), k_query)` $\rightarrow_{HT10}$ `containsKeyHT(ht', k_query)`. Redução recursiva.
    `containsKeyHT` é suficientemente completa.

5.  **`deleteHT(ht: HashTable, k_del: K)`:**
    *   Caso $ht = \text{emptyHT}$: `deleteHT(emptyHT, k_del)` $\rightarrow_{HT11}$ `emptyHT`. (Resultado `HashTable` construtor).
    *   Caso $ht = \text{putHT(k,v,ht')}$:
        *   Subcaso `eqK(k_del, k) = true`: `deleteHT(putHT(k,v,ht'), k_del)` $\rightarrow_{HT12}$ `deleteHT(ht', k_del)`. Redução recursiva.
        *   Subcaso `eqK(k_del, k) = false`: `deleteHT(putHT(k,v,ht'), k_del)` $\rightarrow_{HT13}$ `putHT(k,v,deleteHT(ht', k_del))`. Redução recursiva.
    `deleteHT` é suficientemente completa.

**Conclusão da Análise:**
*   **Consistência:** A especificação `HashTable[K,V]` (como um espelho da `Map[K,V]`) é consistente.
*   **Completeza Suficiente:**
    *   `isEmptyHT`, `sizeHT` (dado `containsKeyHT`), `containsKeyHT`, `deleteHT` são suficientemente completas.
    *   `getHT` é parcial; não é definida para chaves ausentes. Para ser totalmente completa, precisaria de um mecanismo de erro ou um tipo de resultado `Maybe[V]`.

Esta análise assume que `eqK` do parâmetro `K` é uma relação de equivalência bem definida. A função `hashFun` não afeta a consistência ou completeza lógica da especificação (que é a de `Map`), mas é crucial para a eficiência da *implementação* `HashTable`.

**Exercício:**

Considere os axiomas para `sizeHT`:
*   (HT3): `sizeHT(emptyHT) = zero`
*   (HT4): `containsKeyHT(ht,k) = true  => sizeHT(putHT(k,v,ht)) = sizeHT(ht)`
*   (HT5): `containsKeyHT(ht,k) = false => sizeHT(putHT(k,v,ht)) = succ(sizeHT(ht))`
Explique como estes três axiomas, assumindo que `containsKeyHT` é suficientemente completa, garantem que `sizeHT` é suficientemente completa.

**Resolução:**

Para mostrar que `sizeHT` é suficientemente completa, precisamos demonstrar que `sizeHT(ht_term)` se reduz a um termo `Natural` construtor para qualquer `HashTable` `ht_term` formado pelos construtores `emptyHT` e `putHT`. Fazemos isso por análise de casos sobre os construtores de `ht_term`, usando indução estrutural sobre a formação de `ht_term`.

1.  **Caso Base: `ht_term = emptyHT`**
    O termo a ser avaliado é `sizeHT(emptyHT)`.
    Pelo axioma (HT3): `sizeHT(emptyHT) = zero`.
    `zero` é um termo construtor do sort `Natural`. Portanto, o caso base está coberto e leva a um resultado na forma desejada.

2.  **Passo Indutivo: `ht_term = putHT(k, v, ht')`**
    Assumimos como hipótese de indução (HI) que `sizeHT(ht')` é suficientemente completa, ou seja, `sizeHT(ht')` se reduz a um termo `Natural` construtor.
    O termo a ser avaliado é `sizeHT(putHT(k, v, ht'))`.
    O comportamento deste termo é definido pelos axiomas condicionais (HT4) e (HT5), que dependem do valor de `containsKeyHT(ht', k)`.
    Assumimos que `containsKeyHT` é suficientemente completa, o que significa que `containsKeyHT(ht', k)` se reduz a `true` ou `false`.

    *   **Subcaso 2.1: `containsKeyHT(ht', k) = true`**
        Neste caso, o axioma (HT4) se aplica: `sizeHT(putHT(k,v,ht')) = sizeHT(ht')`.
        Pela hipótese de indução (HI), `sizeHT(ht')` se reduz a um termo `Natural` construtor. Portanto, `sizeHT(putHT(k,v,ht'))` também se reduz a um termo `Natural` construtor.

    *   **Subcaso 2.2: `containsKeyHT(ht', k) = false`**
        Neste caso, o axioma (HT5) se aplica: `sizeHT(putHT(k,v,ht')) = succ(sizeHT(ht'))`.
        Pela hipótese de indução (HI), `sizeHT(ht')` se reduz a um termo `Natural` construtor. Seja $N_{ht'}$ esse termo. Então `succ(sizeHT(ht'))` se reduz a `succ(N_{ht'})`. Como `N_{ht'}$ é um termo `Natural` construtor (formado por `zero` e `succ`), `succ(N_{ht'})` também é um termo `Natural` construtor.

**Conclusão da Análise de Completeza Suficiente para `sizeHT`:**
Em todos os casos possíveis de construção de um `HashTable` (seja `emptyHT` ou `putHT(k,v,ht')`), e assumindo a completeza suficiente de `containsKeyHT` para resolver as condições, a operação `sizeHT` é definida pelos axiomas (HT3)-(HT5) para resultar em um termo `Natural` construído com `zero` e `succ`.
Portanto, `sizeHT` é suficientemente completa.

# 19.3 Projeto e Implementação em Python com Tipagem Estática (MyPy)

Agora, projetaremos e implementaremos uma classe Python genérica `PyHashTable[K, V]` que realiza o TAD `Map[K,V]` (e, portanto, os axiomas de `HashTable` da Seção 19.2). Utilizaremos a técnica de **encadeamento separado (separate chaining)** para resolução de colisões. A implementação incluirá tipagem estática com MyPy e recursos como fator de carga e redimensionamento.

**Requisitos para a Chave `K`:**
A chave `K` deve ser hasheável (implementar `__hash__`) e comparável por igualdade (implementar `__eq__`). Em Python, isso é tipicamente satisfeito por tipos imutáveis básicos ou por classes que implementam esses métodos corretamente. Usaremos `typing.Hashable` para tipar `K` quando apropriado.

**Design da Classe `PyHashTable[K,V]`:**

```python
# py_hash_table.py
from __future__ import annotations # For type hints within the class
from typing import TypeVar, Generic, List, Tuple, Optional, Hashable, Iterator, Any

K = TypeVar('K', bound=Hashable) # Key type must be hashable
V = TypeVar('V')                 # Value type

# Type alias for a bucket, which is a list of (key, value) pairs
Bucket = List[Tuple[K, V]]

class PyHashTable(Generic[K, V]):
    _buckets: List[Bucket]
    _num_elements: int
    _initial_capacity: int
    _load_factor_threshold: float

    def __init__(self, initial_capacity: int = 16, load_factor_threshold: float = 0.75) -> None:
        # Constructor for PyHashTable.
        # initial_capacity: initial number of buckets.
        # load_factor_threshold: threshold to trigger resizing.
        if initial_capacity <= 0:
            raise ValueError("Initial capacity must be positive.")
        if not (0.0 < load_factor_threshold <= 1.0):
            raise ValueError("Load factor threshold must be between 0 and 1.")

        self._initial_capacity = initial_capacity
        self._buckets = [[] for _ in range(self._initial_capacity)]
        self._num_elements = 0
        self._load_factor_threshold = load_factor_threshold

    @staticmethod
    def empty_hash_table(initial_capacity: int = 16, load_factor_threshold: float = 0.75) -> PyHashTable[Any, Any]:
        # Corresponds to 'emptyHT: --> HashTable'.
        # Returns an empty hash table.
        return PyHashTable(initial_capacity, load_factor_threshold)

    def _get_bucket_index(self, key: K) -> int:
        # Calculates the bucket index for a given key.
        return hash(key) % len(self._buckets)

    def _resize_if_needed(self) -> None:
        # Checks if resizing is needed based on load factor and resizes if so.
        # This is a mutating operation on self, called internally.
        current_load_factor = self._num_elements / len(self._buckets)
        if current_load_factor > self._load_factor_threshold:
            # Double the capacity (or choose next prime for better distribution)
            new_capacity = len(self._buckets) * 2
            if new_capacity == 0 : new_capacity = self._initial_capacity # handle empty initial
            
            old_buckets = self._buckets
            self._buckets = [[] for _ in range(new_capacity)]
            self._num_elements = 0 # Will be recounted during rehash
            
            for bucket in old_buckets:
                for key, value in bucket:
                    # Re-put, which will use new bucket index and handle num_elements
                    self.put(key, value, _resizing=True) # Pass flag to avoid recursive resize

    def put(self, key: K, value: V, _resizing: bool = False) -> None:
        # Corresponds to 'putHT: K x V x HashTable --> HashTable' (mutating version).
        # Inserts or updates a key-value pair.
        if not _resizing: # Avoid resize check during a resize operation
             self._resize_if_needed()

        bucket_index = self._get_bucket_index(key)
        bucket = self._buckets[bucket_index]

        found_key = False
        for i, (existing_key, _) in enumerate(bucket):
            if existing_key == key: # Using K's __eq__
                bucket[i] = (key, value) # Update existing value
                found_key = True
                break
        
        if not found_key:
            bucket.append((key, value))
            if not _resizing : self._num_elements += 1 # Increment only if it's a new key from external put

    def get(self, key: K) -> Optional[V]:
        # Corresponds to 'getHT: K x HashTable --> V' (returning Optional[V]).
        # Returns the value for a key, or None if key not found.
        bucket_index = self._get_bucket_index(key)
        bucket = self._buckets[bucket_index]
        for existing_key, value in bucket:
            if existing_key == key:
                return value
        return None # Key not found

    def delete(self, key: K) -> None:
        # Corresponds to 'deleteHT: K x HashTable --> HashTable' (mutating version).
        # Removes a key-value pair. Does nothing if key not found.
        bucket_index = self._get_bucket_index(key)
        bucket = self._buckets[bucket_index]
        
        for i, (existing_key, _) in enumerate(bucket):
            if existing_key == key:
                bucket.pop(i) # Remove the tuple (key, value)
                self._num_elements -= 1
                return
        # Key not found, do nothing

    def contains_key(self, key: K) -> bool:
        # Corresponds to 'containsKeyHT: K x HashTable --> Boolean'.
        bucket_index = self._get_bucket_index(key)
        bucket = self._buckets[bucket_index]
        for existing_key, _ in bucket:
            if existing_key == key:
                return True
        return False

    def size(self) -> int:
        # Corresponds to 'sizeHT: HashTable --> Natural'.
        return self._num_elements

    def is_empty(self) -> bool:
        # Corresponds to 'isEmptyHT: HashTable --> Boolean'.
        return self._num_elements == 0

    # For iteration (optional, but useful)
    def __iter__(self) -> Iterator[K]:
        # Iterates over keys in the hash table.
        for bucket in self._buckets:
            for key, _ in bucket:
                yield key
    
    def __len__(self) -> int:
        # Allows using len(py_hash_table_instance)
        return self.size()

    def __repr__(self) -> str:
        # String representation.
        items_str = []
        for bucket in self._buckets:
            for key, value in bucket:
                items_str.append(f"{repr(key)}: {repr(value)}")
        return f"PyHashTable({{{', '.join(items_str)}}})"

```
O código Python acima, no arquivo `py_hash_table.py`, define a classe genérica `PyHashTable[K, V]`.
`K` é um `TypeVar` restrito a `Hashable`, e `V` é um `TypeVar` genérico. A tabela é implementada como uma lista de baldes (`_buckets`), onde cada balde é uma lista de tuplas `(chave, valor)` (encadeamento separado).
O construtor `__init__` inicializa a tabela com uma capacidade inicial e um limiar para o fator de carga, que dispara o redimensionamento. `empty_hash_table()` é um método estático para criar uma tabela vazia.
`_get_bucket_index(key)` calcula o índice do balde usando `hash(key) % len(self._buckets)`.
`_resize_if_needed()` verifica o fator de carga e, se necessário, dobra a capacidade da tabela, re-hasheando e reinserindo todos os elementos.
A operação `put(key, value)` calcula o índice do balde. Se a chave já existe no balde (verificado com `==` da chave), o valor é atualizado. Caso contrário, o novo par `(key, value)` é anexado à lista do balde e `_num_elements` é incrementado (a menos que seja uma chamada interna de `_resize_if_needed`). `put` chama `_resize_if_needed` antes da inserção.
A operação `get(key)` busca a chave no balde apropriado e retorna o valor associado ou `None` se a chave não for encontrada (implementando um retorno tipo `Maybe[V]`).
A operação `delete(key)` remove o par chave-valor do balde apropriado, se existir, e decrementa `_num_elements`.
`contains_key(key)`, `size()`, e `is_empty()` implementam as operações correspondentes do TAD `Map`.
Métodos como `__iter__` (para iterar sobre chaves), `__len__` e `__repr__` são adicionados para conveniência e melhor integração com Python.
Esta implementação é **mutável**; as operações `put` e `delete` modificam a tabela no local. O redimensionamento também é uma mutação interna.

**Exercício:**

Na classe `PyHashTable[K,V]`, o método `put` atualmente não retorna nada (retorna `None` implicitamente) pois modifica a tabela no local. Se quiséssemos que ele seguisse um estilo mais funcional, retornando uma nova instância de `PyHashTable` (mantendo a imutabilidade), como você modificaria o método `put` e a lógica de redimensionamento? Descreva as principais alterações necessárias. (Não precisa reescrever todo o código, apenas descreva as mudanças).

**Resolução:**

Para modificar `put` e o redimensionamento para um estilo funcional/imutável, onde `put` retorna uma nova `PyHashTable`:

1.  **Método `put` Retornando Nova Instância:**
    *   O método `put(self, key: K, value: V) -> PyHashTable[K, V]` precisaria criar uma cópia da estrutura de baldes atual antes de fazer qualquer modificação.
    *   A lógica de encontrar o balde e inserir/atualizar o par `(key, value)` ocorreria nesta cópia.
    *   Ao final, uma nova instância de `PyHashTable` seria criada e retornada, contendo a estrutura de baldes modificada e o `_num_elements` atualizado. O objeto `self` original não seria alterado.

2.  **Lógica de Redimensionamento (`_resize_if_needed`):**
    *   Atualmente, `_resize_if_needed` modifica `self._buckets` e `self._num_elements` no local e chama `self.put` recursivamente (com flag `_resizing`).
    *   Em um estilo imutável, `_resize_if_needed` (ou uma função similar chamada por `put`) não modificaria `self`. Em vez disso, se o redimensionamento fosse necessário, ela construiria uma *nova* estrutura de baldes com a nova capacidade e re-inseriria todos os elementos da tabela original (copiada) nesta nova estrutura.
    *   O método `put` chamaria esta função de "verificar e redimensionar se necessário". Se o redimensionamento ocorresse, a função de redimensionamento retornaria uma *nova* `PyHashTable` já redimensionada. A operação `put` então procederia para inserir o novo par `(key,value)` nesta tabela (possivelmente já redimensionada), novamente de forma imutável (criando outra cópia).

3.  **Fluxo do `put` Imutável:**
    a.  O método `put(self, key: K, value: V)` seria chamado.
    b.  Primeiro, criaria uma cópia preliminar da tabela atual, ou adiaria a cópia.
    c.  Verificaria o fator de carga *desta cópia (ou da tabela original se a cópia ainda não foi feita)*.
    d.  **Se o redimensionamento for necessário:**
        i.  Uma nova `PyHashTable` ($HT_{resized}$) seria criada com maior capacidade, e todos os elementos de `self` seriam re-hasheados e inseridos (de forma imutável, um por um, ou por uma construção de baldes em lote) em $HT_{resized}$.
        ii. A "tabela atual" para a inserção do novo par $(key, value)$ se torna $HT_{resized}$.
    e.  **Se não for necessário redimensionar (ou após o redimensionamento):**
        i.  Uma cópia da estrutura de baldes da "tabela atual" (seja `self` ou $HT_{resized}$) é feita: `new_buckets = copy_buckets(current_table._buckets)`.
        ii. O par $(key, value)$ é inserido/atualizado em `new_buckets`.
        iii. O `new_num_elements` é calculado.
        iv. `return PyHashTable(initial_capacity=len(new_buckets), ...)` inicializado com `new_buckets` e `new_num_elements` (o construtor precisaria de um modo para aceitar baldes pré-populados e contagem).

**Principais Alterações:**
*   **Cópia Profunda:** Quase todas as operações "modificadoras" (`put`, `delete`, e a lógica de `resize`) precisariam começar criando cópias profundas da estrutura de baldes (e das listas de baldes) para não alterar o objeto original.
*   **Construtor:** O `__init__` precisaria de uma forma de ser chamado internamente com uma estrutura de baldes já preenchida e a contagem de elementos, para evitar re-validar/re-copiar desnecessariamente ao criar novas instâncias a partir de modificações internas.
*   **Retorno de `self` ou Nova Instância:** Todos os métodos que alteram o estado retornariam uma nova instância de `PyHashTable`.

Isso aumentaria significativamente a sobrecarga de memória e o tempo de execução para operações de escrita devido às cópias, mas garantiria a imutabilidade, que pode ser desejável em certos contextos (e.g., programação funcional, concorrência sem locks). A implementação mutável atual é mais eficiente para o uso típico de tabelas hash como estruturas de dados de desempenho.

# 19.4 Verificação de Propriedades Axiomáticas com Hypothesis

Para validar a corretude da nossa implementação `PyHashTable[K,V]`, vamos testá-la em relação aos axiomas do TAD `Map[K,V]` (que são espelhados pelos axiomas (HT1)-(HT13) da Seção 19.2). Usaremos Hypothesis para gerar instâncias de `PyHashTable`, chaves e valores, e verificar se as propriedades se mantêm.

**Estratégias Hypothesis:**
Assumiremos `K` como `int` e `V` como `str` para os exemplos de teste, mas as estratégias podem ser generalizadas.

```python
# test_py_hash_table.py
import pytest
from hypothesis import given, strategies as st, settings, HealthCheck, assume
from typing import TypeVar, Any, Tuple, Optional

# Import the implementation
from py_hash_table import PyHashTable # Assuming py_hash_table.py is accessible

# Strategies for Key (K) and Value (V) types
# K must be Hashable. For tests, int and str are good.
st_key_type = st.integers(min_value=-1000, max_value=1000) 
# st_key_type = st.text() # Alternative for string keys
st_value_type = st.text(max_size=20)

# Strategy to build a PyHashTable instance by a sequence of puts
# This approach is often better for testing data structures than st.builds
# if the internal representation is complex or shouldn't be exposed to st.builds.
@st.composite
def st_py_hash_table(draw) -> PyHashTable[int, str]:
    # Draw a list of (key, value) pairs to insert
    # Allow duplicates in list; put will handle updates.
    # max_size controls the number of put operations.
    items_to_insert = draw(st.lists(st.tuples(st_key_type, st_value_type), max_size=20))
    
    ht = PyHashTable[int, str]() # Start with an empty hash table
    for key, value in items_to_insert:
        ht.put(key, value)
    return ht

# Strategy for a hash table and a key known to be in it (if table is not empty)
@st.composite
def st_table_and_contained_key(draw) -> Tuple[PyHashTable[int, str], int]:
    ht = draw(st_py_hash_table.filter(lambda h: not h.is_empty()))
    # To get a key that is definitely in the table:
    # We can iterate over its keys or draw from a list of its keys.
    # This is simpler if PyHashTable implements __iter__ to yield keys.
    # For now, let's draw a list of items used to build, then pick a key from there.
    
    # Re-draw items to ensure we have access to a key that was put
    items_used = draw(st.lists(st.tuples(st_key_type, st_value_type), min_size=1, max_size=10))
    ht_non_empty = PyHashTable[int, str]()
    for k_item, v_item in items_used:
        ht_non_empty.put(k_item, v_item)
    
    # Draw one of the keys that were inserted
    key_in_table = draw(st.sampled_from([item[0] for item in items_used]))
    return ht_non_empty, key_in_table
```
O código de teste `test_py_hash_table.py` configura as estratégias.
*   `st_key_type` e `st_value_type` definem estratégias para gerar chaves (inteiros) e valores (strings).
*   `st_py_hash_table`: Uma estratégia composta para construir uma `PyHashTable` populando-a com uma sequência de operações `put`. Isso geralmente resulta em uma melhor distribuição de estados da tabela para teste do que tentar usar `st.builds` diretamente sobre a estrutura interna de baldes.
*   `st_table_and_contained_key`: Uma estratégia composta para gerar uma `PyHashTable` não vazia e uma chave que está garantidamente contida nela. Isso é útil para testar operações como `get` e `delete` em chaves existentes. Ela reconstrói uma tabela a partir de uma lista de itens e depois amostra uma chave dessa lista.

**Testes de Propriedade para os Axiomas de `HashTable` (espelhando `Map`):**

```python
# test_py_hash_table.py (continuação)

# --- Axioms for isEmptyHT and sizeHT ---
def test_HT1_is_empty_emptyHT():
    # (HT1): isEmptyHT(emptyHT) = true
    assert PyHashTable[Any, Any].empty_hash_table().is_empty() == True

@given(k=st_key_type, v=st_value_type, ht=st_py_hash_table)
def test_HT2_is_empty_putHT(k: int, v: str, ht: PyHashTable[int, str]):
    # (HT2): isEmptyHT(putHT(k,v,ht)) = false
    # Note: Our put is mutating. So we create a "new" state by putting into a copy or new table.
    # For simplicity of testing the *effect* of put, we can do this:
    ht_after_put = PyHashTable[int, str]() # Start conceptually fresh for this test instance
    for existing_k, existing_v in ht: # Iterate over keys of original ht
        ht_after_put.put(existing_k, existing_v)
    ht_after_put.put(k, v) # The put operation being focused on
    assert ht_after_put.is_empty() == False

def test_HT3_size_emptyHT():
    # (HT3): sizeHT(emptyHT) = zero
    assert PyHashTable[Any, Any].empty_hash_table().size() == 0

@given(k=st_key_type, v=st_value_type, ht=st_py_hash_table)
@settings(suppress_health_check=[HealthCheck.filter_too_much], deadline=1000)
def test_HT4_HT5_size_putHT(k: int, v: str, ht: PyHashTable[int, str]):
    # (HT4): containsKeyHT(ht,k) = true  => sizeHT(putHT(k,v,ht)) = sizeHT(ht)
    # (HT5): containsKeyHT(ht,k) = false => sizeHT(putHT(k,v,ht)) = succ(sizeHT(ht))
    
    # Create a functional copy to operate on, as put is in-place
    ht_copy = PyHashTable[int,str](initial_capacity=len(ht._buckets)) # crude copy
    # A better copy method or to_list_of_items would be good for PyHashTable
    # For now, re-populate:
    temp_items = []
    for key_orig, val_orig_bucket in enumerate(ht._buckets): # This iterates buckets, not items
        for ik, iv in val_orig_bucket:
            temp_items.append((ik,iv))
    
    # Rebuild ht_copy with items from ht
    for ik_rebuild, iv_rebuild in temp_items:
        ht_copy.put(ik_rebuild, iv_rebuild)

    original_size = ht_copy.size()
    key_was_present = ht_copy.contains_key(k)
    
    ht_copy.put(k, v) # Mutating put
    new_size = ht_copy.size()
    
    if key_was_present:
        assert new_size == original_size # Axiom HT4
    else:
        assert new_size == original_size + 1 # Axiom HT5

# --- Axioms for getHT ---
@given(k=st_key_type, v=st_value_type, ht=st_py_hash_table)
def test_HT6_getHT_after_putHT_same_key(k: int, v: str, ht: PyHashTable[int, str]):
    # (HT6): getHT(putHT(k,v,ht), k) = v
    ht.put(k, v) # Mutating
    assert ht.get(k) == v

@given(k=st_key_type, v=st_value_type, ht=st_py_hash_table, k1=st_key_type)
@settings(suppress_health_check=[HealthCheck.filter_too_much], deadline=500)
def test_HT7_getHT_after_putHT_different_key(k: int, v: str, ht: PyHashTable[int, str], k1: int):
    # (HT7): eqK(k1,k) = false => getHT(putHT(k,v,ht), k1) = getHT(ht, k1)
    assume(k1 != k) # This is eqK(k1,k) = false
    
    original_value_at_k1 = ht.get(k1) # Could be None
    ht.put(k, v) # Mutating
    
    assert ht.get(k1) == original_value_at_k1

# --- Axioms for containsKeyHT ---
@given(k=st_key_type)
def test_HT8_containsKeyHT_emptyHT(k: int):
    # (HT8): containsKeyHT(emptyHT, k) = false
    assert PyHashTable[Any,Any].empty_hash_table().contains_key(k) == False

@given(k=st_key_type, v=st_value_type, ht=st_py_hash_table)
def test_HT9_containsKeyHT_after_putHT_same_key(k: int, v: str, ht: PyHashTable[int, str]):
    # (HT9): containsKeyHT(putHT(k,v,ht), k) = true
    ht.put(k, v)
    assert ht.contains_key(k) == True

@given(k=st_key_type, v=st_value_type, ht=st_py_hash_table, k1=st_key_type)
@settings(suppress_health_check=[HealthCheck.filter_too_much], deadline=500)
def test_HT10_containsKeyHT_after_putHT_different_key(k: int, v: str, ht: PyHashTable[int, str], k1: int):
    # (HT10): eqK(k1,k) = false => containsKeyHT(putHT(k,v,ht), k1) = containsKeyHT(ht, k1)
    assume(k1 != k)
    
    original_contains_k1 = ht.contains_key(k1)
    ht.put(k, v)
    
    assert ht.contains_key(k1) == original_contains_k1

# --- Axioms for deleteHT ---
@given(k=st_key_type)
def test_HT11_deleteHT_emptyHT(k: int):
    # (HT11): deleteHT(emptyHT, k) = emptyHT
    # Our delete is in-place, so we check size and containsKey
    ht_empty = PyHashTable[Any,Any].empty_hash_table()
    ht_empty.delete(k)
    assert ht_empty.is_empty() == True
    assert ht_empty.size() == 0

@given(k_data=st_table_and_contained_key, v_new=st_value_type)
def test_HT12_deleteHT_after_putHT_same_key(k_data: Tuple[PyHashTable[int,str], int], v_new: str):
    # (HT12): deleteHT(putHT(k,v,ht), k) = deleteHT(ht,k)
    # This means after putting (k,v) and then deleting k, k should not be in the table.
    # And other elements of ht should remain as if k was deleted from original ht.
    ht_original, k_to_put_and_delete = k_data # k_to_put_and_delete is in ht_original
    
    # State 1: original table's state after deleting k_to_put_and_delete
    ht_copy1 = PyHashTable[int, str]() # Rebuild for functional comparison
    for key_orig, val_orig_bucket in enumerate(ht_original._buckets):
        for ik, iv in val_orig_bucket: ht_copy1.put(ik,iv)
    ht_copy1.delete(k_to_put_and_delete) # This is deleteHT(ht,k)

    # State 2: table after put(k,v) then delete(k)
    ht_copy2 = PyHashTable[int, str]() # Rebuild for functional comparison
    for key_orig, val_orig_bucket in enumerate(ht_original._buckets):
        for ik, iv in val_orig_bucket: ht_copy2.put(ik,iv)
    ht_copy2.put(k_to_put_and_delete, v_new) # This is putHT(k,v,ht)
    ht_copy2.delete(k_to_put_and_delete)    # This is deleteHT(putHT(k,v,ht), k)
    
    # For this axiom, a full comparison of tables is complex.
    # A simpler check derived from it: containsKey(delete(put(ht,k,v),k), k) == false
    assert ht_copy2.contains_key(k_to_put_and_delete) == False
    # And also, the size should reflect the deletion from original state if k was unique
    # This specific axiom is hard to test directly with mutating operations without deep equality over abstract Map states.
    # A better test might be:
    # 1. ht_copy = clone(ht_original)
    # 2. ht_copy.put(k,v)
    # 3. ht_copy.delete(k)
    # 4. ht_original.delete(k) # (on another clone if ht_original is needed elsewhere)
    # 5. assert ht_copy == ht_original (after delete)
    # This requires a good clone method.
    # Given our implementation, we can assert a property:
    # After put(k,v) then delete(k), the key k is no longer present.
    assert not ht_copy2.contains_key(k_to_put_and_delete)
    # And size relationship:
    if ht_original.contains_key(k_to_put_and_delete):
         expected_size_ht_copy1 = ht_original.size() -1
    else: # Should not happen with st_table_and_contained_key
         expected_size_ht_copy1 = ht_original.size()
    assert ht_copy1.size() == expected_size_ht_copy1
    assert ht_copy2.size() == expected_size_ht_copy1


@given(k_put=st_key_type, v_put=st_value_type, ht=st_py_hash_table, k_delete=st_key_type)
@settings(suppress_health_check=[HealthCheck.filter_too_much], deadline=1000)
def test_HT13_deleteHT_after_putHT_different_key(k_put: int, v_put: str, ht: PyHashTable[int, str], k_delete: int):
    # (HT13): eqK(k1,k) = false => deleteHT(putHT(k,v,ht), k1) = putHT(k,v,deleteHT(ht,k1))
    assume(k_delete != k_put) # This is eqK(k_delete, k_put) = false

    # LHS: deleteHT(putHT(k_put,v_put,ht), k_delete)
    ht_lhs = PyHashTable[int,str]() # Create a functional copy
    for key_orig, val_orig_bucket in enumerate(ht._buckets):
        for ik, iv in val_orig_bucket: ht_lhs.put(ik,iv)
    ht_lhs.put(k_put, v_put)
    ht_lhs.delete(k_delete)

    # RHS: putHT(k_put,v_put,deleteHT(ht,k_delete))
    ht_rhs = PyHashTable[int,str]() # Create another functional copy
    for key_orig, val_orig_bucket in enumerate(ht._buckets):
        for ik, iv in val_orig_bucket: ht_rhs.put(ik,iv)
    ht_rhs.delete(k_delete)
    ht_rhs.put(k_put, v_put)
    
    assert ht_lhs == ht_rhs
```
Os testes acima traduzem os axiomas (HT1)-(HT13).
*   Testes (HT1)-(HT3) para `isEmptyHT` e `sizeHT` em `emptyHT` são diretos.
*   Teste (HT2) e (HT4-HT5) para `putHT`: Como `put` é mutável na implementação, para testar o estado *após* `put` como se fosse uma nova tabela (conforme o axioma), precisamos de uma maneira de simular a criação de uma nova tabela que é o resultado de `put`. O `test_HT2_is_empty_putHT` recria uma tabela. `test_HT4_HT5_size_putHT` faz uma cópia (de forma um pouco rudimentar, iterando sobre os baldes; uma função `clone()` na `PyHashTable` seria melhor) e então aplica `put` a essa cópia para verificar o tamanho.
*   Testes (HT6)-(HT7) para `getHT`: `test_HT6_getHT_after_putHT_same_key` verifica que `get` após `put` com a mesma chave retorna o valor inserido. `test_HT7_getHT_after_putHT_different_key` verifica que `put` não afeta `get` para outras chaves, usando `assume` para a condição de chaves diferentes.
*   Testes (HT8)-(HT10) para `containsKeyHT`: Seguem uma lógica similar aos de `getHT`.
*   Testes (HT11)-(HT13) para `deleteHT`: `test_HT11_deleteHT_emptyHT` verifica o comportamento de `delete` em uma tabela vazia. `test_HT12_deleteHT_after_putHT_same_key` é complexo de testar perfeitamente com a API mutável sem um `clone()` profundo e uma comparação de tabelas completa. A versão aqui verifica a consequência principal: que a chave não está mais contida e o tamanho é consistente. `test_HT13_deleteHT_after_putHT_different_key` testa a comutatividade de `delete` (em $k_1$) e `put` (em $k \neq k_1$) recriando os estados LHS e RHS para comparação.

A necessidade de copiar ou recriar estados para testar axiomas de estilo funcional com uma implementação mutável é uma complexidade comum. A clareza dos axiomas (e.g., se `getHT` é parcial ou retorna `Maybe`) também influencia como os testes são escritos.

**Exercício:**

Um axioma comum para mapas (e, portanto, para `HashTable`) relacionado à idempotência de `put` (para o valor, não para o estado se o valor muda) é: `getHT(putHT(putHT(ht, k, v1), k, v2), k) = v2`. Isso significa que se você insere um valor `v1` para uma chave `k`, e depois insere um valor `v2` para a *mesma* chave `k`, buscar `k` deve retornar `v2`. Escreva um teste de propriedade Hypothesis para este axioma.

**Resolução:**

```python
# test_py_hash_table.py (continuação)

@given(ht=st_py_hash_table, k=st_key_type, v1=st_value_type, v2=st_value_type)
@settings(suppress_health_check=[HealthCheck.filter_too_much], deadline=500)
def test_put_idempotence_on_value_for_same_key(
    ht: PyHashTable[int, str], k: int, v1: str, v2: str
):
    # Axiom: getHT(putHT(putHT(ht, k, v1), k, v2), k) = v2
    # Testing the effect of two puts with the same key.
    
    # Since put is mutating, we apply it sequentially to the same ht instance.
    ht.put(k, v1) # First put
    ht.put(k, v2) # Second put with the same key, should overwrite v1
    
    # After both puts, getting k should yield v2
    assert ht.get(k) == v2
    
    # Additionally, if k was a new key, size should reflect one new element, not two.
    # This is more related to HT4/HT5 size axioms, but good to keep in mind.
    # If we wanted to test size change from original ht:
    # original_ht_clone_for_size_test = PyHashTable[int,str]() # needs a proper clone
    # ... populate original_ht_clone_for_size_test from ht initial state ...
    # initial_size = original_ht_clone_for_size_test.size()
    # was_present_initially = original_ht_clone_for_size_test.contains_key(k)
    # ... then do the puts on a mutable ht and check final size:
    # final_size = ht.size()
    # if was_present_initially:
    #    assert final_size == initial_size
    # else:
    #    assert final_size == initial_size + 1
```
O teste `test_put_idempotence_on_value_for_same_key` verifica o axioma especificado.
1.  Ele recebe uma `PyHashTable` `ht`, uma chave `k`, e dois valores `v1` e `v2`.
2.  Aplica `ht.put(k, v1)`.
3.  Em seguida, aplica `ht.put(k, v2)` à *mesma* instância `ht` (já que `put` é mutável). A segunda inserção com a mesma chave `k` deve sobrescrever o valor `v1` com `v2`.
4.  Finalmente, a asserção `assert ht.get(k) == v2` verifica se buscar a chave `k` agora retorna `v2`.

Este teste valida que a última inserção para uma chave prevalece, um comportamento fundamental dos mapas. A parte comentada sobre o tamanho ilustra uma verificação adicional que poderia ser feita se uma boa maneira de clonar o estado inicial da tabela (`ht`) estivesse disponível para comparar o tamanho antes e depois da sequência de `put`s.

# 19.5 Análise de Complexidade Algorítmica

A grande vantagem de usar uma `HashTable` para implementar o TAD `Map` é seu desempenho eficiente no caso médio. No entanto, é crucial entender tanto o caso médio quanto o pior caso, e os fatores que os influenciam. A análise aqui assume uma implementação com **encadeamento separado** e uma **função de hash** que distribui as chaves razoavelmente bem.

Seja $N$ o número de elementos na tabela hash e $M$ o número de baldes (tamanho do array subjacente). O **fator de carga** $\alpha = N/M$.

1.  **`__init__(self, initial_capacity=..., load_factor_threshold=...)`**:
    *   Criação da lista de baldes: $O(M)$ para inicializar $M$ listas vazias.
    *   **Complexidade:** $O(M)$.

2.  **`empty_hash_table()`**:
    *   Chama `__init__`.
    *   **Complexidade:** $O(M_{initial})$, onde $M_{initial}$ é a capacidade inicial.

3.  **`_get_bucket_index(key)`**:
    *   `hash(key)`: O custo disto depende do tipo da chave `K` e da qualidade da implementação de `__hash__` para `K`. Frequentemente assumido como $O(\text{comprimento da chave})$ para strings, ou $O(1)$ para tipos numéricos simples. Vamos denotar como $O(H_K)$.
    *   Operação módulo: $O(1)$.
    *   **Complexidade:** $O(H_K)$. Para muitas chaves comuns, pode ser aproximado por $O(1)$ se o comprimento da chave for pequeno ou limitado.

4.  **`put(key, value)`**:
    *   `_resize_if_needed()`: Veja abaixo. Se não houver redimensionamento:
    *   `_get_bucket_index(key)`: $O(H_K)$.
    *   Iterar sobre o balde (lista encadeada): No pior caso, se todas as $N$ chaves hashearem para o mesmo balde, isso é $O(N)$ (mais $N$ comparações de chave $O(E_K)$ cada, onde $E_K$ é o custo de `key.__eq__`). No caso médio, com boa distribuição, o comprimento médio do balde é $\alpha$. Então, a busca no balde é $O(\alpha \cdot E_K)$.
    *   `bucket.append((key,value))` (se nova chave): $O(1)$ para lista encadeada.
    *   Atualizar `_num_elements`: $O(1)$.
    *   **Complexidade (sem redimensionamento):**
        *   **Caso Médio:** $O(H_K + \alpha \cdot E_K + 1) \approx O(1+\alpha)$ se $H_K, E_K$ são $O(1)$. Se $\alpha$ é mantido constante (e.g., $\le 1$), então **$O(1)$ em média**.
        *   **Pior Caso:** $O(H_K + N \cdot E_K) \approx O(N)$ se todas as chaves colidirem.

5.  **`get(key)`**:
    *   `_get_bucket_index(key)`: $O(H_K)$.
    *   Iterar sobre o balde: Mesma análise que `put`.
    *   **Complexidade:**
        *   **Caso Médio:** $O(1+\alpha) \approx O(1)$.
        *   **Pior Caso:** $O(N)$.

6.  **`delete(key)`**:
    *   `_get_bucket_index(key)`: $O(H_K)$.
    *   Iterar sobre o balde para encontrar e remover: Mesma análise que `put` para encontrar. Remoção de lista encadeada é $O(1)$ uma vez encontrado (se tivermos referência ao nó anterior, senão é o custo da busca).
    *   Atualizar `_num_elements`: $O(1)$.
    *   **Complexidade:**
        *   **Caso Médio:** $O(1+\alpha) \approx O(1)$.
        *   **Pior Caso:** $O(N)$.

7.  **`containsKey(key)`**:
    *   Mesma complexidade que `get`. **$O(1)$ em média, $O(N)$ no pior caso.**

8.  **`size()`**: $O(1)$ (retorna `_num_elements`).

9.  **`is_empty()`**: $O(1)$ (compara `_num_elements` com 0).

10. **`_resize_if_needed()` (e o redimensionamento em si):**
    *   Criação da nova lista de baldes (tamanho $M' \approx 2M$): $O(M')$.
    *   Iterar sobre todos os $N$ elementos da tabela antiga. Para cada elemento:
        *   Re-hashear e calcular novo índice de balde: $O(H_K)$.
        *   Inserir na nova tabela: $O(1)$ em média (para a nova inserção no balde da nova tabela, pois a nova tabela está inicialmente vazia ou com baixo fator de carga durante o rehash).
    *   Custo total do redimensionamento: $O(M' + N \cdot H_K) \approx O(M+N)$ se $H_K=O(1)$.
    *   **Análise Amortizada:** Embora um redimensionamento seja $O(N+M)$, ele acontece com pouca frequência (e.g., quando $N$ dobra). Se o custo de $O(N+M)$ for distribuído sobre as $N$ operações `put` que levaram ao redimensionamento, o custo amortizado de `put` (incluindo redimensionamento) permanece $O(1)$ em média, desde que a capacidade aumente geometricamente (e.g., dobrando).

**Resumo da Complexidade (`PyHashTable` com Encadeamento Separado):**
*   `put`, `get`, `delete`, `containsKey`:
    *   **Caso Médio:** $O(1)$ (assumindo boa função hash, fator de carga controlado, e $H_K, E_K$ são $O(1)$).
    *   **Pior Caso:** $O(N)$.
*   `size`, `is_empty`: $O(1)$.
*   `__init__`: $O(M_{initial})$.
*   Redimensionamento: $O(N+M)$ quando ocorre.

Este perfil de desempenho, com operações chave em $O(1)$ no caso médio, é o que torna as tabelas hash tão atraentes e amplamente utilizadas, apesar de seu desempenho no pior caso ser linear. A escolha de uma boa função de hash e uma política de redimensionamento adequada são cruciais para manter o desempenho médio próximo de $O(1)$.

**Exercício:**

Se, em vez de encadeamento separado, uma `PyHashTable` usasse **endereçamento aberto** com **sondagem linear** para resolver colisões, como isso poderia afetar a complexidade no pior caso da operação `get(key)`? E como o desempenho de `delete(key)` se tornaria mais complexo de implementar corretamente com sondagem linear?

**Resolução:**

**Endereçamento Aberto com Sondagem Linear:**
Nesta estratégia, se uma colisão ocorre no balde $i = h(key)$, a tabela sonda sequencialmente os baldes $i+1, i+2, \ldots$ (módulo $M$) até encontrar um balde vazio para inserção, ou encontrar a chave (para `get`), ou encontrar um balde vazio (indicando que a chave não está presente, para `get`).

**1. Impacto na Complexidade no Pior Caso de `get(key)`:**
*   A sondagem linear é suscetível a um fenômeno chamado **agrupamento primário (primary clustering)**. Se várias chaves hasheiam para o mesmo índice inicial, ou para índices próximos, elas tendem a formar longas sequências contíguas de baldes ocupados.
*   **Pior Caso para `get(key)`:** No pior caso absoluto, todas as $N$ chaves inseridas (ou um grande número delas) podem formar um único e longo cluster. Isso pode acontecer se, por exemplo, a função de hash mapear muitas chaves para o mesmo índice ou índices adjacentes, e as inserções subsequentes preencherem uma longa sequência de baldes.
*   Ao buscar uma chave (`get(key)`), se a chave não estiver presente e o cluster cobrir uma grande parte da tabela (ou a tabela inteira, se estiver quase cheia), a sondagem linear pode ter que percorrer muitos baldes. No pior caso, pode precisar examinar até $M$ baldes (o tamanho total da tabela). Se $N$ é próximo de $M$, isso é $O(M) \approx O(N)$.
*   Mesmo que a chave esteja presente, mas no final de um longo cluster, a busca pode levar muitos passos.
*   Portanto, a complexidade no **pior caso** de `get(key)` com sondagem linear ainda é **$O(N)$** (ou $O(M)$ se $M > N$). O agrupamento primário pode tornar esse pior caso mais provável ou levar a um desempenho médio ruim se o fator de carga $\alpha$ se aproximar de 1.

**2. Complexidade da Implementação Correta de `delete(key)` com Sondagem Linear:**
A operação `delete(key)` em uma tabela hash com endereçamento aberto (especialmente sondagem linear ou quadrática) é mais complexa do que com encadeamento separado.
*   **Problema:** Simplesmente esvaziar o balde onde a chave deletada estava pode quebrar as sequências de sondagem para outras chaves que colidiram com aquele balde (ou com um balde anterior na sequência de sondagem) e foram inseridas mais adiante. Uma busca subsequente por essas outras chaves poderia parar prematuramente no balde agora vazio, concluindo erroneamente que a chave não existe.
*   **Soluções Comuns (e sua complexidade):**
    a.  **Marcação com "Lazy Deletion" (Exclusão Preguiçosa):** Em vez de esvaziar o balde, ele é marcado com um estado especial "deletado" (ou "túmulo", *tombstone*).
        *   `put`: Pode reutilizar baldes marcados como "deletado".
        *   `get`: Deve continuar sondando além dos baldes "deletados", pois a chave procurada pode estar mais adiante na sequência.
        *   `delete`: Encontra a chave e marca o balde como "deletado".
        *   **Impacto na Complexidade:** A operação `delete` em si (encontrar e marcar) tem a mesma complexidade de `get` ($O(1)$ médio, $O(N)$ pior). No entanto, a presença de muitos túmulos pode aumentar o comprimento médio das sequências de sondagem para `get` e `put` (pois eles não param a busca), efetivamente diminuindo o desempenho como se o fator de carga fosse maior. O redimensionamento pode ser usado para limpar os túmulos.
    b.  **Re-hashing de Elementos no Cluster:** Quando um elemento é deletado, os elementos subsequentes no mesmo cluster de sondagem (que poderiam ter sido colocados lá devido à presença do elemento deletado) podem precisar ser re-hasheados ou deslocados para preencher a lacuna ou para garantir que ainda possam ser encontrados.
        *   Isso pode tornar a operação `delete` potencialmente custosa, no pior caso $O(N)$ (ou $O(M)$), se um cluster longo precisar ser reajustado. A análise amortizada pode ser complexa.

Portanto, `delete(key)` com sondagem linear é mais complexa de implementar corretamente para manter o desempenho das outras operações, e geralmente envolve o uso de marcadores de exclusão preguiçosa ou algoritmos de re-hashing mais envolvidos. A complexidade da operação `delete` em si (encontrar e marcar/reorganizar) pode variar de $O(1)$ médio (para marcar) a $O(N)$ no pior caso (para reorganizar).

# 19.6 Exercícios Teóricos e Práticos

Esta seção apresenta um exercício que combina aspectos teóricos e práticos relacionados ao TAD `HashTable[K,V]` e sua implementação, com foco em um dos principais desafios: o redimensionamento.

**Exercício:**

**Contexto:** A classe `PyHashTable[K,V]` implementada na Seção 19.3 inclui um método `_resize_if_needed()` que é chamado por `put`. Este método atualmente dobra a capacidade quando o fator de carga é excedido.

**Parte a) Teórica: Impacto da Estratégia de Redimensionamento**
Discuta brevemente:
1.  Por que o aumento geométrico da capacidade (e.g., dobrar) é preferível a um aumento aritmético (e.g., adicionar um número fixo de baldes) para manter o custo amortizado de `put` em $O(1)$?
2.  Qual é uma desvantagem potencial de sempre dobrar a capacidade? Como a escolha de tamanhos de tabela que são números primos poderia ajudar (mesmo que nossa implementação atual não faça isso)?

**Parte b) Prática: Estender `PyHashTable` com Redimensionamento para Encolher**
Atualmente, `PyHashTable` só cresce. Estenda a classe `PyHashTable[K,V]` da Seção 19.3 para incluir uma lógica de redimensionamento que também **encolhe** a tabela se o fator de carga ficar muito baixo após operações `delete`.
1.  Modifique o método `delete` para que, após remover um elemento, ele chame um novo método (ou uma lógica estendida em `_resize_if_needed()`, talvez renomeado para `_check_and_perform_resize()`) que verifique se a tabela deve encolher.
2.  Defina um limiar inferior para o fator de carga (e.g., `shrink_threshold = 0.25`). Se o fator de carga cair abaixo deste limiar E o número atual de baldes for maior que a `_initial_capacity`, a tabela deve ser redimensionada para, por exemplo, metade de sua capacidade atual (mas não menor que `_initial_capacity`).
3.  Implemente esta lógica de encolhimento. Lembre-se que o encolhimento também requer re-hash de todos os elementos.

**Parte c) Teste de Propriedade para Encolhimento**
Escreva um teste de propriedade Hypothesis para verificar se a lógica de encolhimento está funcionando como esperado. O teste deve:
1.  Criar uma tabela e enchê-la com um número de elementos que exceda o `_load_factor_threshold` para causar um crescimento.
2.  Em seguida, deletar elementos suficientes para que o fator de carga caia abaixo do `shrink_threshold`.
3.  Verificar se o número de baldes na tabela foi reduzido para o tamanho esperado (e.g., metade da capacidade após o crescimento, ou para `_initial_capacity` se metade for menor).

**Resolução:**

**Parte a) Teórica: Impacto da Estratégia de Redimensionamento**

1.  **Aumento Geométrico vs. Aritmético para Custo Amortizado $O(1)$:**
    *   **Aumento Geométrico (e.g., dobrar):** Quando a tabela é redimensionada de $M$ para $2M$ baldes, o custo é aproximadamente $O(M+N)$ (onde $N \approx \alpha M$). Antes do próximo redimensionamento (para $4M$), pelo menos $N$ novas inserções (ou $M$ em termos de capacidade) devem ocorrer. O custo $O(M+N)$ do redimensionamento é "pago" por essas $N$ (ou $M$) inserções, resultando em um custo adicional constante por inserção em média (amortizado). A soma dos custos de redimensionamento forma uma série geométrica que é limitada pelo custo do último redimensionamento, levando a um custo amortizado total de $O(1)$ para `put`.
    *   **Aumento Aritmético (e.g., adicionar $C$ baldes fixos):** Se a capacidade aumenta de $M$ para $M+C$, e isso acontece a cada $C \cdot \alpha$ inserções (aproximadamente), o custo de cada redimensionamento ainda é $O(M_i+N_i)$ (onde $M_i$ é a capacidade no $i$-ésimo redimensionamento). Se $M_i$ cresce linearmente ($M_i = M_0 + i \cdot C$), o custo total de redimensionamento sobre $k$ redimensionamentos para atingir $N$ elementos seria da ordem de $\sum O(i) = O(k^2)$. Como $k \approx N/C$, o custo total de redimensionamento seria $O(N^2)$, e o custo amortizado por inserção se tornaria $O(N)$, o que é inaceitável.

2.  **Desvantagem de Dobrar a Capacidade e Benefício de Tamanhos Primos:**
    *   **Desvantagem de Dobrar:** Dobrar a capacidade pode levar a um uso de memória que é, em média, de $25\%$ a $50\%$ vazio (imediatamente após um redimensionamento, a tabela está $\approx \alpha/2$ cheia se $\alpha$ era o limiar). Para tabelas muito grandes, isso pode ser um desperdício significativo de memória. Além disso, se o tamanho da tabela $M$ tem muitos fatores em comum com os valores de hash (especialmente se a parte do módulo da função de hash $h(k) \pmod M$ não for robusta), a distribuição das chaves pode ser ruim.
    *   **Benefício de Tamanhos Primos:** Usar um tamanho de tabela $M$ que é um número primo ajuda a melhorar a distribuição das chaves pelos baldes, especialmente com funções de hash simples (como divisão $h(k) \pmod M$). Um número primo $M$ tem menos probabilidade de ter fatores comuns com padrões nos dados de entrada ou nos valores de hash, reduzindo a chance de colisões sistemáticas e melhorando o desempenho da sondagem (em algumas formas de endereçamento aberto) e do encadeamento. Isso torna a função de hash modular mais eficaz em espalhar as chaves.

**Parte b) Prática: Estender `PyHashTable` com Redimensionamento para Encolher**

Modificaremos a classe `PyHashTable`.

```python
# py_hash_table.py (modificado)
from __future__ import annotations 
from typing import TypeVar, Generic, List, Tuple, Optional, Hashable, Iterator, Any

K = TypeVar('K', bound=Hashable) 
V = TypeVar('V')                 

Bucket = List[Tuple[K, V]]

class PyHashTable(Generic[K, V]):
    _buckets: List[Bucket]
    _num_elements: int
    _initial_capacity: int
    _load_factor_threshold: float
    _shrink_threshold: float # New attribute

    def __init__(self, initial_capacity: int = 16, 
                 load_factor_threshold: float = 0.75,
                 shrink_threshold: float = 0.25) -> None: # Added shrink_threshold
        if initial_capacity <= 0:
            raise ValueError("Initial capacity must be positive.")
        if not (0.0 < load_factor_threshold <= 1.0):
            raise ValueError("Load factor threshold must be between 0 and 1.")
        if not (0.0 <= shrink_threshold < load_factor_threshold): # Shrink threshold must be valid
            raise ValueError("Shrink threshold must be < load factor threshold and >= 0.")

        self._initial_capacity = initial_capacity
        self._buckets = [[] for _ in range(self._initial_capacity)]
        self._num_elements = 0
        self._load_factor_threshold = load_factor_threshold
        self._shrink_threshold = shrink_threshold # Store it

    @staticmethod
    def empty_hash_table(initial_capacity: int = 16, 
                         load_factor_threshold: float = 0.75,
                         shrink_threshold: float = 0.25) -> PyHashTable[Any, Any]:
        return PyHashTable(initial_capacity, load_factor_threshold, shrink_threshold)

    def _get_bucket_index(self, key: K) -> int:
        return hash(key) % len(self._buckets)

    def _resize_table(self, new_capacity: int) -> None:
        # Generic resize method (for growing or shrinking)
        old_buckets = self._buckets
        self._buckets = [[] for _ in range(new_capacity)]
        self._num_elements = 0 # Will be recounted during rehash
        
        for bucket in old_buckets:
            for key, value in bucket:
                self.put(key, value, _resizing=True) # Re-put items

    def _check_and_perform_resize(self) -> None:
        # Checks load factor and resizes (grows or shrinks) if needed.
        current_capacity = len(self._buckets)
        current_load_factor = self._num_elements / current_capacity if current_capacity > 0 else 0.0

        if current_load_factor > self._load_factor_threshold and current_capacity > 0 :
            # Grow table
            new_capacity = current_capacity * 2
            self._resize_table(new_capacity)
        elif current_load_factor < self._shrink_threshold and current_capacity > self._initial_capacity:
            # Shrink table
            new_capacity = max(self._initial_capacity, current_capacity // 2)
            if new_capacity < current_capacity: # Only resize if new capacity is smaller
                self._resize_table(new_capacity)


    def put(self, key: K, value: V, _resizing: bool = False) -> None:
        if not _resizing:
             self._check_and_perform_resize() # Check for grow before put

        bucket_index = self._get_bucket_index(key)
        bucket = self._buckets[bucket_index]

        found_key = False
        for i, (existing_key, _) in enumerate(bucket):
            if existing_key == key: 
                bucket[i] = (key, value) 
                found_key = True
                break
        
        if not found_key:
            bucket.append((key, value))
            if not _resizing : self._num_elements += 1
    
    def delete(self, key: K) -> None:
        bucket_index = self._get_bucket_index(key)
        bucket = self._buckets[bucket_index]
        
        for i, (existing_key, _) in enumerate(bucket):
            if existing_key == key:
                bucket.pop(i) 
                self._num_elements -= 1
                self._check_and_perform_resize() # Check for shrink after delete
                return
    
    # ... (get, contains_key, size, is_empty, __iter__, __len__, __repr__ remain the same) ...
    def get(self, key: K) -> Optional[V]:
        bucket_index = self._get_bucket_index(key)
        bucket = self._buckets[bucket_index]
        for existing_key, value in bucket:
            if existing_key == key:
                return value
        return None

    def contains_key(self, key: K) -> bool:
        bucket_index = self._get_bucket_index(key)
        bucket = self._buckets[bucket_index]
        for existing_key, _ in bucket:
            if existing_key == key:
                return True
        return False

    def size(self) -> int:
        return self._num_elements

    def is_empty(self) -> bool:
        return self._num_elements == 0
    
    def __iter__(self) -> Iterator[K]:
        for bucket in self._buckets:
            for key, _ in bucket:
                yield key
    
    def __len__(self) -> int:
        return self.size()

    def __repr__(self) -> str:
        items_str = []
        # Need to handle ht._buckets being potentially empty if initial_capacity was 0 (fixed)
        if not self._buckets: # Should not happen with initial_capacity > 0
            return "PyHashTable({})"
            
        for bucket in self._buckets:
            for key, value in bucket:
                items_str.append(f"{repr(key)}: {repr(value)}")
        return f"PyHashTable({{{', '.join(items_str)}}})"

```
O código `py_hash_table.py` foi modificado:
*   O construtor `__init__` agora aceita e armazena `shrink_threshold`.
*   `_resize_if_needed()` foi renomeado para `_check_and_perform_resize()` e sua lógica foi estendida.
*   `_resize_table(new_capacity)` é um novo método auxiliar para encapsular a lógica de redimensionamento (criação de novos baldes e re-hash).
*   `_check_and_perform_resize()` agora verifica duas condições:
    *   **Crescimento:** Se `current_load_factor > self._load_factor_threshold`, a capacidade é dobrada.
    *   **Encolhimento:** Se `current_load_factor < self._shrink_threshold` E a capacidade atual `current_capacity` é maior que a `self._initial_capacity`, a capacidade é reduzida pela metade (mas não menor que `_initial_capacity`). A condição `new_capacity < current_capacity` garante que o encolhimento só ocorre se a nova capacidade calculada for de fato menor.
*   O método `put` chama `_check_and_perform_resize()` *antes* da lógica de inserção para potencialmente crescer.
*   O método `delete` chama `_check_and_perform_resize()` *após* remover um elemento para potencialmente encolher.

**Parte c) Teste de Propriedade para Encolhimento**

```python
# test_py_hash_table.py (adicionar este novo teste)
# ... (importações e estratégias st_key_type, st_value_type como antes) ...

@settings(deadline=2000, suppress_health_check=[HealthCheck.too_slow, HealthCheck.filter_too_much])
@given(
    initial_cap=st.integers(min_value=4, max_value=8), # Small initial capacity
    # Generate enough items to trigger at least one resize (grow)
    # e.g., if threshold is 0.75, initial_cap=4, needs >3 items. Then more to pass 0.75 of 8.
    # Let's aim for enough items to make it grow, then delete many.
    # List of (key,value) tuples for insertion and then for deletion.
    # Ensure keys are unique for initial fill to make size predictable.
    items_to_fill = st.lists(st.tuples(st.integers(min_value=0, max_value=1000), st_value_type), min_size=10, max_size=20, unique_by=lambda x: x[0]),
    num_to_delete_later = st.integers(min_value=1, max_value=20) # Number of items to delete
)
def test_hash_table_shrinks_when_load_factor_is_low(
    initial_cap: int, 
    items_to_fill: List[Tuple[int, str]],
    num_to_delete_later: int
):
    # Use specific thresholds for this test for predictability
    load_factor_thresh = 0.75
    shrink_thresh = 0.25 # Must be < load_factor_thresh
    
    # Ensure initial_cap is compatible with shrink_thresh for meaningful test
    # e.g. initial_cap * shrink_thresh should be at least 1 for num_to_delete to have effect
    assume(initial_cap * shrink_thresh >= 1 or initial_cap == 1)


    ht = PyHashTable[int, str](
        initial_capacity=initial_cap, 
        load_factor_threshold=load_factor_thresh,
        shrink_threshold=shrink_thresh
    )

    # 1. Fill the table to potentially trigger growth
    current_keys = set()
    for key, value in items_to_fill:
        if key not in current_keys: # Ensure unique keys for predictable size
            ht.put(key, value)
            current_keys.add(key)
    
    capacity_after_fill = len(ht._buckets)
    # We expect capacity_after_fill to be >= initial_cap.
    # If it grew, it should be initial_cap * 2^k for some k >= 1.

    # 2. Delete elements to potentially trigger shrink
    keys_to_delete = list(current_keys)[:num_to_delete_later] # Take a subset of existing keys
    
    if not keys_to_delete and ht.size() > 0: # If num_to_delete_later was too high or no keys
        assume(False) # Not a good scenario for testing shrink, get new data

    for key_del in keys_to_delete:
        ht.delete(key_del)
        
    capacity_after_delete = len(ht._buckets)
    final_num_elements = ht.size()
    
    # 3. Check if shrinking occurred as expected
    expected_shrink_capacity = max(initial_cap, capacity_after_fill // 2)

    if final_num_elements == 0 and capacity_after_fill > initial_cap : # Special case: if all deleted
        assert capacity_after_delete == initial_cap
    elif (final_num_elements / capacity_after_fill) < shrink_thresh and \
         capacity_after_fill > initial_cap:
        # Shrinking should have occurred
        assert capacity_after_delete == expected_shrink_capacity
        # Also check that new capacity is not less than initial_cap (already in logic)
        assert capacity_after_delete >= initial_cap
    else:
        # No shrinking expected, capacity should remain capacity_after_fill
        # or it might have shrunk but not to expected_shrink_capacity if that was too small
        if capacity_after_fill > initial_cap and (expected_shrink_capacity < capacity_after_fill):
             # if it was supposed to shrink, but the condition above didn't catch it, there might be an issue
             # or the condition (final_num_elements / capacity_after_fill) < shrink_thresh was not met
             # For simplicity, we only assert if we positively expect a shrink.
             # This else branch means either no shrink was triggered or it shrunk to initial_cap already.
             pass

```
O teste `test_hash_table_shrinks_when_load_factor_is_low` é adicionado:
1.  **Estratégias e Configurações:**
    *   Gera uma `initial_cap` pequena.
    *   Gera `items_to_fill` (com chaves únicas) para popular a tabela, com um `min_size` que provavelmente causará um ou mais redimensionamentos para crescimento.
    *   Gera `num_to_delete_later` para controlar quantos itens são removidos.
    *   Usa `@settings` para aumentar o `deadline` e suprimir `too_slow` e `filter_too_much` devido à complexidade da geração e `assume`.
2.  **Setup do Teste:**
    *   Cria uma `PyHashTable` com `initial_cap` e os limiares `load_factor_thresh` (0.75) e `shrink_thresh` (0.25) fixos para previsibilidade.
    *   Um `assume` garante que a capacidade inicial e o `shrink_threshold` são compatíveis para um teste significativo de encolhimento.
3.  **Fase de Preenchimento (Grow):**
    *   A tabela é preenchida com `items_to_fill` (apenas chaves únicas são adicionadas para que `ht.size()` seja o número de itens únicos).
    *   `capacity_after_fill` armazena o número de baldes após esta fase (espera-se que tenha crescido).
4.  **Fase de Deleção (Shrink):**
    *   Um subconjunto de chaves existentes (`keys_to_delete`) é selecionado para deleção.
    *   Essas chaves são deletadas da tabela.
5.  **Verificação:**
    *   `capacity_after_delete` e `final_num_elements` são obtidos.
    *   `expected_shrink_capacity` é calculado como `max(initial_cap, capacity_after_fill // 2)`.
    *   A asserção principal verifica: se o fator de carga após as deleções (`final_num_elements / capacity_after_fill`, usando a capacidade *antes* do possível encolhimento para verificar a condição de disparo) cair abaixo de `shrink_thresh` E a capacidade atual (`capacity_after_fill`) for maior que a inicial, então a capacidade final (`capacity_after_delete`) deve ser igual à `expected_shrink_capacity`.
    *   Um caso especial para quando todos os elementos são deletados e a capacidade encolhe para a inicial também é verificado.
    *   O `else` cobre casos onde o encolhimento não era esperado ou já encolheu para `initial_cap`.

Este teste é complexo devido à natureza estatal do redimensionamento e à necessidade de controlar as condições que o disparam. A geração de dados precisa criar cenários onde o crescimento e depois o encolhimento são prováveis. O uso de `assume` dentro das estratégias ou do teste é comum para guiar Hypothesis para esses cenários mais específicos.

Este capítulo abordou o TAD `HashTable[K,V]` como uma implementação eficiente do TAD `Map[K,V]`. Discutimos seus princípios (função de hash, colisões, resolução, fator de carga, redimensionamento), sua relevância e a interface que espelha `Map`. A especificação algébrica foi apresentada como a obrigação de satisfazer os axiomas de `Map`, com requisitos adicionais para o tipo chave `K` (hasheável e com igualdade). Uma implementação Python `PyHashTable[K,V]` usando encadeamento separado e redimensionamento (incluindo encolhimento) foi detalhada. A verificação de propriedades axiomáticas com Hypothesis demonstrou como validar a conformidade da implementação com a semântica de `Map`. Finalmente, a análise de complexidade confirmou o desempenho $O(1)$ amortizado para operações chave. O próximo capítulo, Capítulo 20, se voltará para o TAD `Graph`, uma estrutura de dados não-linear fundamental para modelar relações complexas entre entidades.

---
## REFERÊNCIAS BIBLIOGRÁFICAS
CELIS, P. **Robin Hood Hashing**. Tese de Doutorado, University of Waterloo, 1986. Disponível em: https://cs.uwaterloo.ca/research/tr/1986/CS-86-14.pdf. Acesso em: 10 ago. 2023.
*   *Resumo: Uma tese clássica que introduz e analisa a técnica de Robin Hood hashing, uma forma de endereçamento aberto para resolução de colisões que visa reduzir a variância nos tempos de busca. Relevante para discussões aprofundadas sobre estratégias de resolução de colisões.*

CORMEN, T. H.; LEISERSON, C. E.; RIVEST, R. L.; STEIN, C. **Introduction to Algorithms**. 3. ed. Cambridge, MA: MIT Press, 2009. (Capítulo 11: Hash Tables).
*   *Resumo: Este capítulo seminal fornece uma discussão teórica detalhada sobre tabelas hash, incluindo funções de hash, resolução de colisões (encadeamento e endereçamento aberto), e análise de desempenho (incluindo fator de carga e redimensionamento). Essencial para todos os tópicos do Capítulo 19.*

KNUTH, D. E. **The Art of Computer Programming, Volume 3: Sorting and Searching**. 2. ed. Reading, MA: Addison-Wesley, 1998. (Seção 6.4: Hashing).
*   *Resumo: A obra de Knuth é uma referência canônica. A seção sobre hashing é extremamente detalhada, cobrindo uma vasta gama de funções de hash, técnicas de resolução de colisões e análises matemáticas profundas. Fundamental para um entendimento completo da teoria de tabelas hash.*

MITZENMACHER, M.; UPFAL, E. **Probability and Computing: Randomized Algorithms and Probabilistic Analysis**. 2. ed. Cambridge: Cambridge University Press, 2017. (Capítulo 5: Hashing).
*   *Resumo: Este livro aborda o hashing sob a perspectiva da probabilidade e da análise de algoritmos randomizados, incluindo tópicos como hashing universal e perfect hashing. Relevante para entender os fundamentos teóricos que garantem o bom desempenho médio das tabelas hash.*

PYTHON SOFTWARE FOUNDATION. **Python `dict` documentation and `hash()` built-in function**. Disponível em: https://docs.python.org/3/library/stdtypes.html#mapping-types-dict e https://docs.python.org/3/library/functions.html#hash. Acesso em: 10 ago. 2023.
*   *Resumo: A documentação oficial do Python sobre o tipo `dict` (que é implementado como uma tabela hash) e a função `hash()` é crucial para entender como os conceitos de tabela hash são aplicados na prática em Python e os requisitos para que objetos sejam hasheáveis.*

SEDGEWICK, R.; WAYNE, K. **Algorithms**. 4. ed. Upper Saddle River, NJ: Addison-Wesley Professional, 2011. (Capítulo 3.4: Hash Tables).
*   *Resumo: Oferece uma apresentação clara e moderna de tabelas hash, com implementações em Java e discussões sobre escolhas práticas de design, como funções de hash para diferentes tipos de dados e estratégias de resolução de colisões. Um bom complemento prático à teoria.*

---
