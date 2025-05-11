---
# Capítulo 18
# Tipo Abstrato de Dado Map
---

Iniciando a Parte IV deste livro, que se dedica às estruturas de dados para coleções e relações, este capítulo introduz o Tipo Abstrato de Dados (TAD) `Map[K,V]`, também conhecido como dicionário, tabela associativa, ou mapa associativo. O TAD `Map[K,V]` modela uma coleção de pares chave-valor, onde cada chave do tipo `K` é única e está associada a um valor do tipo `V`. Esta estrutura é fundamental para inúmeras aplicações que requerem armazenamento e recuperação eficientes de dados baseados em chaves únicas, como tabelas de símbolos em compiladores, caches, contadores de frequência, ou a representação de objetos com atributos nomeados. Seguindo a metodologia estabelecida, começaremos com uma definição abstrata do TAD `Map[K,V]`, destacando suas operações essenciais como criação de um mapa vazio, inserção ou atualização de um par chave-valor (`put`), recuperação de um valor associado a uma chave (`get`), remoção de um par chave-valor (`remove`), e verificação de existência de uma chave (`containsKey`). Em seguida, desenvolveremos uma especificação algébrica formal para `Map[K,V]`, parametrizada pelos tipos da chave `K` e do valor `V`, detalhando seus sorts, operações e axiomas. A especificação do parâmetro `K` exigirá uma operação de igualdade. Analisaremos a consistência e a completeza suficiente desta especificação. Posteriormente, projetaremos e implementaremos uma classe genérica `PyMap[K,V]` em Python, utilizando dicionários Python como estrutura de dados subjacente e anotações de tipo MyPy. A validação da corretude da implementação `PyMap[K,V]` será conduzida através do Teste Baseado em Propriedades com Hypothesis. Concluiremos com uma análise da complexidade algorítmica das operações e um exercício prático.

# 18.1 Definição Abstrata e Relevância do Tipo de Dados `Map[K,V]`

O Tipo Abstrato de Dados `Map[K,V]`, também conhecido como dicionário, array associativo ou tabela de símbolos, representa uma coleção de pares $(k, v)$, onde $k$ é uma **chave** de um tipo `K` e $v$ é um **valor** associado de um tipo `V`. A característica fundamental de um mapa é que cada chave $k$ dentro do mapa é **única**; ou seja, uma chave pode estar associada a no máximo um valor. Mapas são estruturas de dados não lineares (no sentido de que não impõem uma ordem sequencial primária aos seus elementos como listas ou vetores, embora algumas implementações possam oferecer iteração ordenada por chaves) e são otimizados para busca, inserção e remoção eficientes de valores baseados em suas chaves.

**Definição Abstrata:**
Um `Map[K,V]` é um TAD que modela uma função parcial finita de chaves do tipo `K` para valores do tipo `V`. "Parcial" porque nem toda chave possível do tipo `K` precisa estar no mapa, e "finita" porque o mapa contém um número finito de associações chave-valor. As operações essenciais incluem:

1.  **Criação:**
    *   `emptyMap: --> Map[K,V]`: Cria um mapa vazio, sem nenhuma associação chave-valor.
2.  **Modificação/Inserção:**
    *   `put: K x V x Map[K,V] --> Map[K,V]`: Adiciona uma nova associação $(k,v)$ ao mapa ou atualiza o valor associado à chave $k$ se ela já existir. Retorna o novo mapa.
3.  **Acesso/Recuperação:**
    *   `get: K x Map[K,V] --> V`: Retorna o valor $v$ associado à chave $k$ no mapa. Esta operação é parcial; não é definida se a chave $k$ não estiver presente no mapa. (Alternativamente, pode retornar um tipo `Optional[V]` ou levantar uma exceção).
4.  **Remoção:**
    *   `remove: K x Map[K,V] --> Map[K,V]`: Remove a associação da chave $k$ (e seu valor correspondente) do mapa. Se a chave $k$ não estiver no mapa, o mapa permanece inalterado. Retorna o novo mapa.
5.  **Consulta/Observação:**
    *   `containsKey: K x Map[K,V] --> Boolean`: Verifica se uma chave $k$ está presente no mapa.
    *   `isEmpty: Map[K,V] --> Boolean`: Verifica se o mapa está vazio.
    *   `size: Map[K,V] --> Natural`: Retorna o número de associações (pares chave-valor) no mapa.

Para que operações como `put` (para atualização), `get`, `remove`, e `containsKey` funcionem corretamente, o tipo da chave `K` deve suportar uma operação de **igualdade** confiável.

**Relevância do TAD `Map[K,V]`:**
O TAD `Map[K,V]` é uma das estruturas de dados mais versáteis e amplamente utilizadas na computação, devido à sua capacidade de modelar associações diretas entre dados:

1.  **Recuperação Eficiente de Dados:** A principal vantagem é a capacidade de recuperar valores rapidamente usando suas chaves, idealmente em tempo $O(1)$ (amortizado para tabelas hash) ou $O(\log N)$ (para árvores de busca balanceadas).
2.  **Representação de Dicionários e Léxicos:** Como o nome sugere, é a estrutura natural para representar dicionários (palavra $\rightarrow$ definição).
3.  **Tabelas de Símbolos em Compiladores:** Compiladores usam mapas para armazenar informações sobre identificadores (variáveis, funções) e seus atributos (tipo, escopo, endereço).
4.  **Contadores de Frequência:** Mapas podem ser usados para contar ocorrências de itens, onde o item é a chave e a contagem é o valor.
5.  **Caches:** Usados para armazenar resultados de computações caras ou dados frequentemente acessados, com a chave identificando o dado e o valor sendo o dado em si.
6.  **Representação de Objetos e Registros:** Em muitas linguagens dinâmicas, os atributos de um objeto podem ser representados internamente como um mapa de nomes de atributos para seus valores.
7.  **Configurações e Mapeamentos:** Armazenar configurações de aplicativos (chave de configuração $\rightarrow$ valor da configuração) ou qualquer tipo de mapeamento entre dois conjuntos de dados.
8.  **Implementação de Conjuntos (`Set[K]`):** Um conjunto pode ser implementado como um mapa onde os valores são triviais (e.g., um booleano `true`) e apenas as chaves são de interesse.

As implementações mais comuns de mapas são tabelas hash (hash tables) e árvores de busca balanceadas (como árvores AVL ou Rubro-Negras). A escolha entre elas envolve um trade-off entre desempenho médio (geralmente melhor em tabelas hash) e desempenho no pior caso garantido (melhor em árvores balanceadas), além da capacidade de iterar sobre as chaves em ordem (inerente a árvores, possível mas menos natural em tabelas hash).

Neste capítulo, especificaremos `Map[K,V]` algebricamente, implementaremos `PyMap[K,V]` em Python usando dicionários Python como base, e verificaremos sua corretude.

**Exercício:**

Considere um TAD `Map[String, Natural]` usado para contar a frequência de palavras em um texto. As chaves são palavras (`String`) e os valores são suas contagens (`Natural`). Descreva a sequência de estados do mapa após as seguintes operações, começando com um mapa vazio `m0 = emptyMap`:
1.  `m1 = put("apple", one, m0)` (onde `one` é o `Natural` 1)
2.  `m2 = put("banana", one, m1)`
3.  `m3 = put("apple", succ(get("apple", m2)), m2)` (atualiza a contagem de "apple")
4.  `count_apple = get("apple", m3)`
5.  `has_orange = containsKey("orange", m3)`

Qual o estado de `m3` e os valores de `count_apple` e `has_orange`?

**Resolução:**

Vamos traçar a sequência de operações:

1.  **`m0 = emptyMap`**
    *   Estado de `m0`: $\{ \}$ (mapa vazio)

2.  **`m1 = put("apple", one, m0)`**
    *   `one` representa o `Natural` 1.
    *   Adiciona o par ("apple", 1) a `m0`.
    *   Estado de `m1`: $\{ \text{"apple"} \mapsto 1 \}$

3.  **`m2 = put("banana", one, m1)`**
    *   Adiciona o par ("banana", 1) a `m1`.
    *   Estado de `m2`: $\{ \text{"apple"} \mapsto 1, \text{"banana"} \mapsto 1 \}$

4.  **`m3 = put("apple", succ(get("apple", m2)), m2)`**
    *   Primeiro, avaliamos `get("apple", m2)`:
        *   `get("apple", m2)` retorna o valor associado a "apple" em `m2`, que é 1.
    *   Em seguida, `succ(get("apple", m2))` é `succ(1)`, que é o `Natural` 2.
    *   Agora, `put("apple", 2, m2)`:
        *   A chave "apple" já existe em `m2`. A operação `put` atualiza seu valor para 2.
    *   Estado de `m3`: $\{ \text{"apple"} \mapsto 2, \text{"banana"} \mapsto 1 \}$

5.  **`count_apple = get("apple", m3)`**
    *   Retorna o valor associado a "apple" em `m3`.
    *   `count_apple = 2` (o `Natural` 2).

6.  **`has_orange = containsKey("orange", m3)`**
    *   Verifica se a chave "orange" existe em `m3`. Ela não existe.
    *   `has_orange = false` (o `Boolean` `false`).

Concluindo:
*   Estado de `m3`: $\{ \text{"apple"} \mapsto 2, \text{"banana"} \mapsto 1 \}$
*   `count_apple`: o `Natural` 2
*   `has_orange`: o `Boolean` `false`

# 18.2 Especificação Algébrica Formal do TAD `Map[K,V]`

Para especificar formalmente o TAD `Map[K,V]`, precisamos parametrizá-lo pelos tipos da chave `K` e do valor `V`. A especificação do tipo da chave `K` deve, no mínimo, fornecer uma operação de igualdade, pois a unicidade das chaves e operações como `get`, `put` (para atualização) e `remove` dependem da capacidade de comparar chaves.

**SPEC ADT** KeyTypeRequirement
**sorts:**
*   K
**operations:**
*   eqK: K x K --> Boolean
**axioms:**
*   (KR1): eqK(k,k) = true
*   (KR2): eqK(k1,k2) = eqK(k2,k1)
for all k,k1,k2: K
**END SPEC**

**SPEC ADT** ValueType
**sorts:**
*   V
**END SPEC**

**SPEC ADT** Map[ParamK: KeyTypeRequirement, ParamV: ValueType]
**imports:**
*   ParamK
*   ParamV
*   Natural
*   Boolean

**sorts:**
*   Map

**operations:**
*   emptyMap: --> Map
*   put: K x V x Map --> Map
*   get: K x Map --> V
*   remove: K x Map --> Map
*   containsKey: K x Map --> Boolean
*   isEmpty: Map --> Boolean
*   size: Map --> Natural

**axioms:**
for all m: Map, k, k1, k2: K, v, v1: V

*   (M1): isEmpty(emptyMap) = true
*   (M2): isEmpty(put(k, v, m)) = false

*   (M3): containsKey(k, emptyMap) = false
*   (M4): containsKey(k1, put(k2, v, m)) = or(eqK(k1, k2), containsKey(k1, m))

*   (M5): get(k1, put(k2, v, m)) = ifThenElse(eqK(k1, k2), v, get(k1, m))
*   (M6): remove(k1, emptyMap) = emptyMap
*   (M7): remove(k1, put(k2, v, m)) =
        ifThenElse(eqK(k1, k2),
                   remove(k1, m),
                   put(k2, v, remove(k1, m)))

*   (M8): size(emptyMap) = zero
*   (M9): size(put(k, v, m)) =
        ifThenElse(containsKey(k, m),
                   size(m),
                   succ(size(m)))

**END SPEC**

A especificação `KeyTypeRequirement` define que o tipo da chave `K` deve fornecer uma operação de igualdade `eqK` que seja pelo menos reflexiva e simétrica (a transitividade seria ideal para uma relação de equivalência completa). `ValueType` simplesmente introduz o sort `V`.
A especificação `Map[ParamK, ParamV]` é parametrizada por `ParamK` (que satisfaz `KeyTypeRequirement`) e `ParamV`. Ela importa estes parâmetros, `Natural` e `Boolean`. O sort principal é `Map`.
As **operações** são `emptyMap` (construtor), `put` (construtor/modificador), `get` (observador parcial), `remove` (modificador), `containsKey` (observador), `isEmpty` (observador) e `size` (observador).
Os **axiomas** definem o comportamento:
(M1)-(M2) definem `isEmpty`. (M3)-(M4) definem `containsKey` recursivamente: uma chave está em `put(k2,v,m)` se é `k2` ou se já estava em `m`.
(M5) define `get`: se as chaves são iguais, retorna o novo valor `v`; caso contrário, busca no mapa `m` original. Este axioma torna `get(k, emptyMap)` indefinido, pois não há um `put` para decompor (é parcial).
(M6)-(M7) definem `remove`: remover de um mapa vazio não faz nada; remover de `put(k2,v,m)`: se `k1=k2`, a associação é removida (e recursivamente de `m` para limpar sobreposições anteriores de `k1`); se `k1!=k2`, a associação `(k2,v)` é mantida e a remoção continua em `m`.
(M8)-(M9) definem `size`: o tamanho de `emptyMap` é `zero`; `put` incrementa o tamanho somente se a chave não estava presente anteriormente.

## 18.2.1 Análise de Corretude e Completeza da Especificação `Map[K,V]`

Analisamos a especificação `Map[K,V]` quanto à consistência e completeza suficiente.

**Consistência:**
1.  **Proteção dos Tipos Importados:** Os axiomas parecem preservar a semântica de `Boolean`, `Natural`, `K` (com `eqK`) e `V`.
2.  **Consistência Interna:**
    *   `emptyMap` é distinto de `put(k,v,m)` via (M1), (M2) e `true != false`.
    *   A unicidade das chaves é implicitamente tratada pelos axiomas de `get` e `put`. Por exemplo, `get(k, put(k,v2,put(k,v1,m)))` resultaria em `v2`.
    *   O axioma (M7) para `remove` é complexo; sua interação com `put` para chaves sobrepostas precisa ser cuidadosamente considerada (e.g., `remove(k, put(k,v,m)) = remove(k,m)`). A versão dada, `remove(k1,m)` quando `k1=k2`, lida com isso.
3.  **Modelo Inicial:** O modelo de funções parciais finitas de `K` para `V` é um modelo consistente para esta especificação.

A especificação parece **consistente**. A complexidade do axioma (M7) sugere que simplificações ou uma ordem de aplicação de regras seriam importantes num sistema de reescrita. A operação `get` é parcial, indefinida para chaves ausentes (e.g. `get(k, emptyMap)`).

**Completeza Suficiente:**
Construtores: `emptyMap`, `put`.
Operações não-construtoras: `isEmpty`, `containsKey`, `get`, `remove`, `size`.

1.  **`isEmpty(map_arg: Map)`:**
    *   `isEmpty(emptyMap)` $\rightarrow_{M1}$ `true`.
    *   `isEmpty(put(k,v,m))` $\rightarrow_{M2}$ `false`.
    `isEmpty` é suficientemente completa.

2.  **`containsKey(key_arg: K, map_arg: Map)`:**
    *   `containsKey(k1, emptyMap)` $\rightarrow_{M3}$ `false`.
    *   `containsKey(k1, put(k2,v,m))` $\rightarrow_{M4}$ `or(eqK(k1,k2), containsKey(k1,m))`. Reduz a uma expressão `Boolean` usando `eqK` (do parâmetro `K`, assumido completo) e uma chamada recursiva a `containsKey` sobre `m` (que é estruturalmente menor ou eventualmente `emptyMap`).
    `containsKey` é suficientemente completa.

3.  **`get(key_arg: K, map_arg: Map)`:**
    *   `get(k1, emptyMap)`: Indefinido pelos axiomas. Não suficientemente completa para este caso (parcial).
    *   `get(k1, put(k2,v,m))` $\rightarrow_{M5}$ `ifThenElse(eqK(k1,k2), v, get(k1,m))`.
        *   Se `eqK(k1,k2)` é `true`, reduz a `v` (do tipo `V`).
        *   Se `eqK(k1,k2)` é `false`, reduz a `get(k1,m)`. A recursão sobre `m` eventualmente atingirá `emptyMap` (se `k1` não estava em `m`) ou um `put` onde as chaves casam. Se atinge `get(k1, emptyMap)`, fica indefinido.
    `get` é suficientemente completa para chaves existentes, mas parcial para chaves ausentes.

4.  **`remove(key_arg: K, map_arg: Map)`:**
    *   `remove(k1, emptyMap)` $\rightarrow_{M6}$ `emptyMap`. (Termo construtor `Map`).
    *   `remove(k1, put(k2,v,m))` $\rightarrow_{M7}$ `ifThenElse(...)`. O resultado do `ifThenElse` será `remove(k1,m)` ou `put(k2,v,remove(k1,m))`. Ambos os ramos envolvem `remove` em um mapa menor `m` ou `emptyMap` (coberto por M6), e `put`. Por indução, `remove` se reduz a um termo `Map` construtor.
    `remove` é suficientemente completa.

5.  **`size(map_arg: Map)`:**
    *   `size(emptyMap)` $\rightarrow_{M8}$ `zero`. (Termo `Natural` construtor).
    *   `size(put(k,v,m))` $\rightarrow_{M9}$ `ifThenElse(containsKey(k,m), size(m), succ(size(m)))`.
        Como `containsKey` e `size` (em `size(m)`) são suficientemente completas (assumindo HI para `size(m)`), e `succ` é um construtor `Natural`, esta expressão se reduz a um termo `Natural` construtor.
    `size` é suficientemente completa.

**Conclusão da Análise:**
*   **Consistência:** A especificação `Map[K,V]` parece consistente.
*   **Completeza Suficiente:** `isEmpty`, `containsKey`, `remove`, `size` são suficientemente completas. `get` é parcial (não completa para chaves ausentes).

Para tornar `get` total, precisaríamos de um mecanismo para retornar um valor de erro ou um valor padrão (e.g., um tipo `Optional[V]` ou `Result[V, Error]`).

**Exercício:**

Suponha que a especificação do parâmetro `ParamK: KeyTypeRequirement` não incluísse o axioma de simetria (KR2): `eqK(k1,k2) = eqK(k2,k1)`. Como a ausência deste axioma para `eqK` poderia afetar a semântica esperada ou a capacidade de provar propriedades do TAD `Map[K,V]`, especialmente em relação às operações `containsKey` e `get`?

**Resolução:**

Se o axioma de simetria (KR2) `eqK(k1,k2) = eqK(k2,k1)` fosse omitido da `KeyTypeRequirement`, a operação `eqK` fornecida pelo tipo da chave `K` não seria necessariamente simétrica. Isso teria implicações significativas para o TAD `Map[K,V]`:

1.  **Comportamento de `containsKey` (Axioma M4):**
    `containsKey(k1, put(k2, v, m)) = or(eqK(k1, k2), containsKey(k1, m))`
    Se `eqK` não é simétrica, então `eqK(k1, k2)` poderia ser `true` enquanto `eqK(k2, k1)` poderia ser `false` (ou vice-versa). A definição de `containsKey` depende da ordem dos argumentos em `eqK`.
    *   Se `put(k_chave, valor_v, mapa_m)` foi feito, e depois se consulta `containsKey(k_consulta, mapa_resultante)`, a verificação será `eqK(k_consulta, k_chave)`. Se esta for `true`, a chave é encontrada.
    *   O problema não é tanto na *definição* de `containsKey` em si, mas na *interpretação* da igualdade de chaves. Se `eqK(k_consulta, k_chave)` é `true` mas `eqK(k_chave, k_consulta)` é `false`, o conceito de "a chave `k_consulta` é a mesma que `k_chave`" se torna ambíguo e dependente da ordem.
    *   A consequência mais direta é que o mapa pode se comportar de forma inesperada se o usuário assumir que a igualdade de chaves é simétrica. Por exemplo, `put(k,v,m)` insere com base na chave `k`. Se depois `containsKey(k', m')` for chamado onde `k'` é "simetricamente igual" a `k` mas `eqK(k',k)` é `false` (enquanto `eqK(k,k')` é `true`), a chave `k'` não seria encontrada por esta regra.

2.  **Comportamento de `get` (Axioma M5):**
    `get(k1, put(k2, v, m)) = ifThenElse(eqK(k1, k2), v, get(k1, m))`
    Similarmente, a recuperação de um valor dependerá da avaliação de `eqK(k1, k2)`. Se a igualdade não for simétrica, buscar por uma chave `k1` que é "semanticamente" a mesma que `k2` mas para a qual `eqK(k1, k2)` é `false` resultaria em não encontrar o valor, mesmo que `eqK(k2, k1)` fosse `true`. Isso viola a expectativa de que se uma chave foi inserida, ela (ou uma chave "igual" a ela em um sentido intuitivo) possa ser usada para recuperar o valor.

3.  **Comportamento de `remove` (Axioma M7):**
    `remove(k1, put(k2, v, m)) = ifThenElse(eqK(k1, k2), remove(k1, m), put(k2, v, remove(k1, m)))`
    A decisão de remover a associação `(k2,v)` ou mantê-la depende de `eqK(k1,k2)`. Novamente, a falta de simetria em `eqK` poderia levar a um comportamento onde uma chave `k1` falha em remover uma entrada `(k2,v)` mesmo que `k2` seja "simetricamente igual" a `k1`.

4.  **Comportamento de `size` (Axioma M9):**
    `size(put(k, v, m)) = ifThenElse(containsKey(k, m), size(m), succ(size(m)))`
    A corretude do `size` depende de `containsKey` identificar corretamente se a chave `k` já está no mapa `m` (para decidir se é uma atualização ou uma nova inserção que aumenta o tamanho). Se `containsKey` se comporta de forma assimétrica devido a `eqK`, então `size` também pode ser calculado incorretamente. Por exemplo, `put(k, v1, emptyMap)` seguido de `put(k', v2, ...)` onde `eqK(k,k')` é `false` mas `eqK(k',k)` é `true` (ou vice-versa) poderia levar a `k` e `k'` serem tratadas como chaves distintas para fins de `size`, mesmo que intuitivamente devessem ser a mesma.

**Impacto na Prova de Propriedades:**
Muitas propriedades desejáveis de um mapa (e.g., idempotência de `put` com a mesma chave e valor, ou que `get(k, put(k,v,m))=v`) dependem de uma noção bem comportada de igualdade de chaves. Se `eqK` não é uma relação de equivalência (reflexiva, simétrica, transitiva), então:
*   Provar que `put(k, v, put(k, v, m)) = put(k, v, m)` (idempotência de `put` para atualização) se torna difícil, pois a segunda chamada a `put` usaria `eqK(k,k_interna)` para decidir se é uma atualização. Se `k_interna` foi a `k` da primeira `put`, `eqK(k,k)` deve ser `true` (garantido por KR1, reflexividade, que ainda está presente).
*   Se `eqK` não for transitiva, poderíamos ter `eqK(k1,k2)=true` e `eqK(k2,k3)=true`, mas `eqK(k1,k3)=false`. Isso levaria a um comportamento muito confuso, onde `k1` é igual a `k2`, `k2` é igual a `k3`, mas `k1` não é igual a `k3`. Um mapa se comportaria de forma muito estranha com tal noção de igualdade.

Em resumo, a ausência de simetria (e transitividade, se também omitida) para `eqK` quebraria a semântica esperada de um mapa onde chaves "iguais" devem se referir à mesma entrada. A especificação do TAD `Map[K,V]` seria válida apenas para tipos `K` cuja operação `eqK` fornecida fosse, de fato, uma relação de equivalência (ou pelo menos simétrica, para o comportamento mais básico). Sem isso, o TAD `Map` não se comportaria como um mapa associativo padrão.

# 18.3 Projeto e Implementação em Python com Tipagem Estática (MyPy)

Para implementar o TAD `Map[K,V]` em Python, criaremos uma classe genérica `PyMap[K,V]`. A estrutura de dados subjacente mais natural e eficiente em Python para realizar um mapa é o dicionário embutido (`dict`). Os dicionários Python já garantem a unicidade das chaves (baseado na igualdade e hash das chaves) e fornecem operações eficientes. Nossa classe `PyMap` atuará como um wrapper em torno de um dicionário Python, possivelmente adotando um estilo funcional/imutável para suas operações, conforme a semântica dos axiomas. Utilizaremos `typing.TypeVar` e `typing.Generic` para os tipos genéricos `K` (chave) e `V` (valor), e anotações MyPy.

**Design da Classe `PyMap[K,V]`:**
As chaves `K` em Python devem ser hasheáveis (implementar `__hash__` e `__eq__`).

```python
# py_map.py
from __future__ import annotations # For type hints of PyMap[K,V] within the class itself
from typing import TypeVar, Generic, Optional, Dict as PyDictInternal, Any, Iterable

K = TypeVar('K') # Generic type for Keys (must be hashable for dict keys)
V = TypeVar('V') # Generic type for Values

class PyMap(Generic[K, V]):
    _internal_dict: PyDictInternal[K, V] # Internal Python dictionary

    def __init__(self, initial_items: Optional[Iterable[tuple[K, V]]] = None) -> None:
        # Constructor for PyMap.
        # Accepts an optional iterable of (key, value) tuples for initialization.
        if initial_items is None:
            self._internal_dict = {}
        else:
            # Type checking for K and V would be handled by MyPy at call sites
            # where PyMap[ConcreteK, ConcreteV] is instantiated and used.
            self._internal_dict = dict(initial_items) # Creates a new dict

    @staticmethod
    def empty_map() -> PyMap[Any, Any]: 
        # Corresponds to 'emptyMap: --> Map[K,V]'
        # Returns an empty map. Types K, V are Any here.
        return PyMap()

    def put(self, key: K, value: V) -> PyMap[K, V]:
        # Corresponds to 'put: K x V x Map --> Map'.
        # Returns a NEW PyMap with the key-value association added/updated. Immutable style.
        new_dict_items = list(self._internal_dict.items()) # Get current items
        
        # Check if key exists to replace, otherwise append.
        # More efficient would be to copy and then set key.
        updated = False
        for i, (k_existing, v_existing) in enumerate(new_dict_items):
            if k_existing == key:
                new_dict_items[i] = (key, value)
                updated = True
                break
        if not updated:
            new_dict_items.append((key, value))
            
        # Alternative (more Pythonic and efficient for dicts):
        new_py_dict = self._internal_dict.copy()
        new_py_dict[key] = value
        return PyMap(new_py_dict.items()) # Pass items to constructor which creates new dict

    def get(self, key: K) -> V: 
        # Corresponds to 'get: K x Map --> V'.
        # Raises KeyError if the key is not found (Python dict behavior).
        # Specification is partial; this matches Python's partiality for missing keys.
        if key not in self._internal_dict: # More explicit check than just letting it raise
            raise KeyError(f"Key not found in PyMap: {key}")
        return self._internal_dict[key]

    def remove(self, key: K) -> PyMap[K, V]:
        # Corresponds to 'remove: K x Map --> Map'.
        # Returns a NEW PyMap with the key (and its value) removed. Immutable style.
        # If key is not found, returns a new map equivalent to the original.
        if key not in self._internal_dict:
            return PyMap(self._internal_dict.items()) # Return a copy of the current one

        new_py_dict = self._internal_dict.copy()
        del new_py_dict[key]
        return PyMap(new_py_dict.items())

    def contains_key(self, key: K) -> bool:
        # Corresponds to 'containsKey: K x Map --> Boolean'
        return key in self._internal_dict

    def is_empty(self) -> bool:
        # Corresponds to 'isEmpty: Map --> Boolean'
        return not self._internal_dict # Empty dict is Falsy

    def size(self) -> int: # Size is a Natural (int >= 0)
        # Corresponds to 'size: Map --> Natural'
        return len(self._internal_dict)

    # Methods for equality and representation
    def __eq__(self, other: object) -> bool:
        if not isinstance(other, PyMap):
            return NotImplemented
        # This assumes K and V types are compatible for comparison of dicts.
        return self._internal_dict == other._internal_dict

    def __repr__(self) -> str:
        return f"PyMap({self._internal_dict})"

    # Helper for testing
    def to_dict(self) -> PyDictInternal[K,V]:
        return self._internal_dict.copy()
```
O código Python acima, no arquivo `py_map.py`, define a classe genérica `PyMap[K,V]`.
Ela é parametrizada pelos tipos `K` (chave) e `V` (valor). Internamente, utiliza um dicionário Python `_internal_dict`. O construtor `__init__` permite inicialização opcional a partir de um iterável de tuplas chave-valor, criando uma cópia para o dicionário interno. `empty_map()` é um método estático que retorna um `PyMap` vazio.
As operações `put`, `get`, e `remove` são implementadas de forma imutável, sempre retornando uma nova instância de `PyMap`.
*   `put` cria uma cópia do dicionário interno, e então insere ou atualiza a associação chave-valor na cópia.
*   `get` usa o acesso `[]` do dicionário Python, que levanta `KeyError` se a chave não for encontrada, alinhando-se com a natureza parcial da operação `get` na especificação. Uma verificação explícita com `in` é adicionada para clareza do `KeyError`.
*   `remove` também opera sobre uma cópia. Se a chave não existe, retorna uma cópia do mapa original; caso contrário, remove a chave da cópia.
As operações `containsKey`, `isEmpty`, e `size` são observadores que consultam o dicionário interno.
`__eq__` e `__repr__` são fornecidos para comparabilidade e representação textual. `to_dict()` é um auxiliar para testes.

**Exercício:**

Adicione um método `keys(self) -> PyVector[K]` à classe `PyMap[K,V]`. Este método deve retornar um `PyVector` (da implementação do Capítulo 8, `py_vector.PyVector`) contendo todas as chaves presentes no mapa. A ordem das chaves no vetor não é especificada (pode depender da implementação do dicionário Python subjacente). Assuma que a classe `PyVector` está disponível para importação.

**Resolução:**

Para adicionar o método `keys` à classe `PyMap[K,V]`:

```python
# py_map.py (continuação da classe PyMap[K,V])
# Assume PyVector is importable:
# from py_vector import PyVector # (From Chapter 8 implementation)
# For standalone, mock PyVector if needed for type hinting, or use PyListInternal[K]
from typing import List as PyListInternal # Placeholder if PyVector not available for snippet
# If PyVector is available:
# from py_vector import PyVector

# Placeholder for PyVector if its actual import is not part of this snippet
class PyVector(Generic[T]): # Minimal mock for type hinting
    def __init__(self, initial_elements: Optional[PyListInternal[T]] = None):
        if initial_elements is None: self._elements = []
        else: self._elements = list(initial_elements)
    def __eq__(self, other: object) -> bool:
        if not isinstance(other, PyVector): return NotImplemented
        return self._elements == other._elements
    # Add other methods if PyVector is actually used beyond type hinting


# ... (dentro da classe PyMap[K,V]) ...

    def keys(self) -> PyVector[K]: # Returning PyVector of keys
        # Returns a PyVector containing all keys in the map.
        # The order of keys is not guaranteed.
        
        # Get keys from the internal Python dictionary
        dict_keys = list(self._internal_dict.keys())
        
        # Create a PyVector from these keys
        return PyVector(dict_keys) # Assumes PyVector can be initialized with a list

    # ... (restante da classe) ...
```
O método `keys` foi adicionado à classe `PyMap[K,V]`.
1.  **Assinatura:** `keys(self) -> PyVector[K]`. Retorna um `PyVector` cujos elementos são do tipo `K` (o tipo da chave do mapa).
2.  **Implementação:**
    *   `dict_keys = list(self._internal_dict.keys())`: Primeiro, obtemos uma visão das chaves do dicionário interno usando `self._internal_dict.keys()`. Isso é convertido para uma lista Python explícita.
    *   `return PyVector(dict_keys)`: Em seguida, uma nova instância de `PyVector[K]` é criada, inicializada com esta lista de chaves. Isso assume que a classe `PyVector` (implementada no Capítulo 8) pode ser construída a partir de uma lista de elementos do tipo `K`.

Este método fornece uma maneira de obter todos as chaves do mapa, encapsuladas em uma estrutura de `PyVector` (que também é imutável em sua versão do Capítulo 8). A ordem não é garantida pelos dicionários Python antes da versão 3.7 (e mesmo depois, a especificação do TAD `Map` geralmente não garante ordem de chaves a menos que seja um `SortedMap`).

# 18.4 Verificação de Propriedades Axiomáticas com Hypothesis

Com a especificação algébrica de `Map[K,V]` e a implementação `PyMap[K,V]`, usamos Hypothesis para verificar se a implementação satisfaz os axiomas.

**Estratégias Hypothesis para `PyMap[K,V]`, `K`, e `V`:**
Para testar `PyMap[K,V]`, precisamos de estratégias para `K` (chaves) e `V` (valores), e depois para `PyMap[K,V]`. Para os exemplos, usaremos `st.text()` para `K` e `st.integers()` para `V`.

```python
# test_py_map.py
import pytest
from hypothesis import given, strategies as st, assume, settings, HealthCheck
from typing import TypeVar, Any, Dict as PyDictInternal, Tuple

# Import the implementation
from py_map import PyMap # Assuming py_map.py is accessible

# Strategies for Key (K) and Value (V) types
# For keys, ensure they are hashable. Text (strings) are fine.
st_key_type = st.text(max_size=5) 
st_value_type = st.integers(min_value=-100, max_value=100)

# Strategy for generating instances of PyMap[K,V]
# K=str, V=int for these tests
st_py_map = st.builds(
    PyMap[str, int],
    initial_items=st.lists(st.tuples(st_key_type, st_value_type), max_size=5)
    # Build from a list of (key,value) tuples.
    # dict() constructor handles duplicate keys by taking the last one.
)

# Strategy to generate a map and a key that might be in it
@st.composite
def st_map_and_key(draw):
    m = draw(st_py_map)
    # Draw a key that could be new, or could be one of the existing keys
    if draw(st.booleans()) and not m.is_empty(): # 50% chance to pick an existing key if map not empty
        k = draw(st.sampled_from(list(m.to_dict().keys())))
    else:
        k = draw(st_key_type)
    return m, k
```
O código de teste `test_py_map.py` configura as estratégias. `st_key_type` gera strings curtas (hasheáveis). `st_value_type` gera inteiros. `st_py_map` constrói `PyMap[str, int]` usando listas de tuplas `(chave, valor)`. A estratégia `st_map_and_key` gera um mapa e uma chave, com uma chance de a chave já existir no mapa.

**Testes de Propriedade para os Axiomas de `Map[K,V]`:**
Axiomas (M1)-(M9) da Seção 18.2.

```python
# test_py_map.py (continuação)

# --- Axioms for isEmpty ---
def test_M1_is_empty_emptyMap():
    # (M1): isEmpty(emptyMap) = true
    assert PyMap[Any, Any].empty_map().is_empty() == True

@given(m=st_py_map, k=st_key_type, v=st_value_type)
def test_M2_is_empty_put(m: PyMap[str, int], k: str, v: int):
    # (M2): isEmpty(put(k, v, m)) = false
    assert m.put(k, v).is_empty() == False

# --- Axioms for containsKey ---
@given(k=st_key_type)
def test_M3_containsKey_emptyMap(k: str):
    # (M3): containsKey(k, emptyMap) = false
    assert PyMap[Any, Any].empty_map().contains_key(k) == False

@given(m=st_py_map, k1=st_key_type, k2=st_key_type, v=st_value_type)
def test_M4_containsKey_put(m: PyMap[str, int], k1: str, k2: str, v: int):
    # (M4): containsKey(k1, put(k2, v, m)) = or(eqK(k1, k2), containsKey(k1, m))
    # Python's '==' for strings implements eqK for this instance.
    # Python's 'or' implements the Boolean 'or'.
    map_after_put = m.put(k2, v)
    lhs = map_after_put.contains_key(k1)
    rhs = (k1 == k2) or m.contains_key(k1)
    assert lhs == rhs

# --- Axiom for get ---
@given(m=st_py_map, k1=st_key_type, k2=st_key_type, v=st_value_type)
def test_M5_get_put(m: PyMap[str, int], k1: str, k2: str, v: int):
    # (M5): get(k1, put(k2, v, m)) = ifThenElse(eqK(k1, k2), v, get(k1, m))
    map_after_put = m.put(k2, v)
    if k1 == k2:
        assert map_after_put.get(k1) == v
    else:
        # If k1 is not in m, get(k1,m) will raise KeyError.
        # The test needs to handle this to match the partial nature of get.
        if m.contains_key(k1):
            assert map_after_put.get(k1) == m.get(k1)
        else:
            # k1 != k2 AND k1 not in m. So get(k1, map_after_put) should also fail.
            with pytest.raises(KeyError):
                map_after_put.get(k1)

# Test behavior for get on emptyMap (expect KeyError from Python impl)
@given(k=st_key_type)
def test_get_on_empty_map_raises_exception(k: str):
    empty_m = PyMap[Any, Any].empty_map()
    with pytest.raises(KeyError):
        empty_m.get(k)

# --- Axioms for remove ---
@given(k=st_key_type)
def test_M6_remove_emptyMap(k: str):
    # (M6): remove(k1, emptyMap) = emptyMap
    empty_m = PyMap[Any,Any].empty_map()
    assert empty_m.remove(k) == empty_m

@given(m=st_py_map, k1=st_key_type, k2=st_key_type, v=st_value_type)
def test_M7_remove_put(m: PyMap[str, int], k1: str, k2: str, v: int):
    # (M7): remove(k1, put(k2, v, m)) =
    #       ifThenElse(eqK(k1, k2), remove(k1, m), put(k2, v, remove(k1, m)))
    map_after_put = m.put(k2, v)
    lhs = map_after_put.remove(k1)
    
    if k1 == k2:
        # If keys are same, original value v at k2 is removed by k1.
        # Then we remove k1 from m (which might be k2 again).
        # The axiom states: remove(k1, m)
        # If k1=k2, then remove(k2, m) is the state.
        rhs = m.remove(k1) # Or m.remove(k2) - same thing
    else:
        # If keys are different, (k2,v) is kept, and k1 is removed from m.
        # Then (k2,v) is put back on top of remove(k1,m).
        rhs = (m.remove(k1)).put(k2, v)
    assert lhs == rhs

# --- Axioms for size ---
def test_M8_size_emptyMap():
    # (M8): size(emptyMap) = zero
    assert PyMap[Any, Any].empty_map().size() == 0

@given(m=st_py_map, k=st_key_type, v=st_value_type)
def test_M9_size_put(m: PyMap[str, int], k: str, v: int):
    # (M9): size(put(k, v, m)) =
    #       ifThenElse(containsKey(k, m), size(m), succ(size(m)))
    
    lhs_size = m.put(k, v).size()
    
    if m.contains_key(k):
        rhs_size = m.size() # Key existed, size doesn't change (value update)
    else:
        rhs_size = m.size() + 1 # New key, size increments
    
    assert lhs_size == rhs_size
```
Os testes acima verificam os axiomas (M1)-(M9).
Os testes para `isEmpty` e `size` são diretos.
`test_M4_containsKey_put` traduz a lógica `or` e `eqK` para Python.
`test_M5_get_put` lida com a natureza parcial de `get` verificando `containsKey` antes de tentar `m.get(k1)` no ramo `else`, ou esperando `KeyError` se a chave não deveria existir.
`test_M7_remove_put` implementa a lógica condicional de `remove` em relação a `put`.
Estes testes, quando executados com `pytest`, validariam o comportamento da classe `PyMap` contra sua especificação algébrica.

**Exercício:**

O TAD `Map[K,V]` pode ter uma operação `toList: Map[K,V] -> List[Pair[K,V]]` (onde `Pair[K,V]` é um TAD de par e `List` é um TAD de lista, e a ordem na lista não é especificada). Suponha que `PyMap` tem um método `to_list_of_pairs(self) -> list[tuple[K,V]]` que retorna uma lista Python de tuplas. Um axioma para `size` em termos de `toList` poderia ser: `size(m) = length(toList(m))` (onde `length` é do TAD Lista). Escreva um teste de propriedade para este axioma para `PyMap`. Assuma que `PyVector` (do Cap 8) pode ser usado como a implementação de `List` e tem um método `length()`.

**Resolução:**

```python
# test_py_map.py (continuação)
# from py_vector import PyVector # Assume PyVector is available

# Mock PyVector for this test if not fully integrated for brevity
class PyVector(Generic[T]):
    def __init__(self, initial_elements: Optional[list[T]] = None):
        self._elements = list(initial_elements) if initial_elements else []
    def length(self) -> int:
        return len(self._elements)
    def __eq__(self, other): # Simplified for test
        if not isinstance(other, PyVector): return NotImplemented
        return self._elements == other._elements


# Add to_list_of_pairs method to PyMap if it doesn't exist
# class PyMap(Generic[K,V]):
#   ...
#   def to_list_of_pairs(self) -> list[tuple[K,V]]:
#       return list(self._internal_dict.items())
#   ...

# Assume PyMap is extended with:
# def to_vector_of_pairs(self) -> PyVector[Tuple[K,V]]:
#     return PyVector(list(self._internal_dict.items()))


# For the test, let's assume PyMap has a method to_vector_of_pairs returning a PyVector
# We'll add a dummy one if PyMap isn't actually extended, just for the test to run.
if not hasattr(PyMap, "to_vector_of_pairs"):
    def to_vector_of_pairs_impl(self_map: PyMap[K,V]) -> PyVector[Tuple[K,V]]:
        return PyVector(list(self_map.to_dict().items()))
    PyMap.to_vector_of_pairs = to_vector_of_pairs_impl


@given(m=st_py_map)
def test_prop_size_equals_length_of_toList(m: PyMap[str, int]):
    """
    Tests the property: size(m) = length(toList(m))
    Python: m.size() == m.to_vector_of_pairs().length()
    """
    
    # Get the size using the map's size operation
    size_from_map_op = m.size()
    
    # Get the list of pairs and then its length
    # Assuming to_vector_of_pairs returns a PyVector-like object with a length() method
    vector_of_pairs = m.to_vector_of_pairs() 
    length_from_list_op = vector_of_pairs.length()
    
    assert size_from_map_op == length_from_list_op
```
O teste `test_prop_size_equals_length_of_toList` verifica a propriedade de que `size(m)` é igual ao `length(toList(m))`.
Primeiro, uma implementação simulada (mock) de `PyVector` com um método `length()` é fornecida para que o teste seja executável sem a dependência completa do Capítulo 8. Em seguida, um método `to_vector_of_pairs` é adicionado dinamicamente à classe `PyMap` para o propósito deste teste (em um projeto real, ele seria parte da classe `PyMap`). Este método converte os itens do dicionário interno do `PyMap` em uma lista de tuplas e, em seguida, cria um `PyVector` a partir dessa lista.
O teste, decorado com `@given(m=st_py_map)`, recebe um `PyMap` gerado. Ele calcula `m.size()` e `m.to_vector_of_pairs().length()` e assere que os dois são iguais. Este teste valida a consistência entre a operação `size` do mapa e o comprimento da coleção de seus pares chave-valor.

# 18.5 Análise de Complexidade Algorítmica

A implementação `PyMap[K,V]` utiliza um dicionário Python (`dict`) como sua estrutura de dados interna. A complexidade das operações de `PyMap` será largamente determinada pela complexidade das operações correspondentes em dicionários Python, com a ressalva de que nossas operações `put`, `remove` (e `set` se fosse um `Vector`) são imutáveis, o que implica a cópia do dicionário.

**Complexidade das Operações de Dicionário Python (`dict`):**
(Onde $N$ é o número de itens no dicionário)
*   Acesso (`d[k]`): $O(1)$ em média, $O(N)$ no pior caso (raro, devido a colisões de hash).
*   Inserção/Atualização (`d[k] = v`): $O(1)$ em média, $O(N)$ no pior caso.
*   Remoção (`del d[k]`): $O(1)$ em média, $O(N)$ no pior caso.
*   Verificar pertinência (`k in d`): $O(1)$ em média, $O(N)$ no pior caso.
*   Obter tamanho (`len(d)`): $O(1)$.
*   Copiar (`d.copy()` ou `dict(d)`): $O(N)$.
*   Obter itens (`d.items()`): Retorna uma visão em $O(1)$, iterar sobre ela é $O(N)$.

**Análise de Complexidade das Operações de `PyMap[K,V]` (Imutável):**

1.  **`__init__(self, initial_items: Optional[Iterable[tuple[K, V]]] = None)`:**
    *   Se `initial_items` é `None`: $O(1)$.
    *   Se `initial_items` tem $M$ tuplas: `dict(initial_items)` constrói um dicionário. Se houver $N \le M$ chaves únicas, o custo é aproximadamente $O(M)$ em média (para processar todas as tuplas) para resultar em um dicionário de tamanho $N$.
    *   **Complexidade Total:** $O(M)$ em média.

2.  **`empty_map()`:** $O(1)$.

3.  **`put(self, key: K, value: V) -> PyMap[K, V]`:**
    *   `self._internal_dict.copy()`: $O(N)$, onde $N$ é o tamanho do mapa original.
    *   `new_py_dict[key] = value` (na cópia): $O(1)$ em média.
    *   `PyMap(new_py_dict.items())`: O construtor usa `dict(new_py_dict.items())`. `items()` é $O(1)$ para criar a visão, `list(new_py_dict.items())` (implícito se `initial_items` é uma visão) é $O(N)$, e `dict(...)` sobre isso é $O(N)$.
    *   **Complexidade Total:** $O(N)$ em média.

4.  **`get(self, key: K) -> V`:**
    *   `key not in self._internal_dict`: $O(1)$ em média.
    *   `self._internal_dict[key]`: $O(1)$ em média.
    *   **Complexidade:** $O(1)$ em média.

5.  **`remove(self, key: K) -> PyMap[K, V]`:**
    *   `key not in self._internal_dict`: $O(1)$ em média.
    *   Se não encontrado, `self._internal_dict.copy()` ($O(N)$) e construtor ($O(N)$). Total $O(N)$.
    *   Se encontrado, `self._internal_dict.copy()` ($O(N)$), `del new_py_dict[key]` ($O(1)$ média), construtor ($O(N-1)$). Total $O(N)$.
    *   **Complexidade Total:** $O(N)$ em média.

6.  **`containsKey(self, key: K) -> bool`:**
    *   `key in self._internal_dict`: $O(1)$ em média.
    *   **Complexidade:** $O(1)$ em média.

7.  **`is_empty(self) -> bool`:**
    *   `not self._internal_dict`: $O(1)$ (verifica se o dicionário está vazio).
    *   **Complexidade:** $O(1)$.

8.  **`size(self) -> int`:**
    *   `len(self._internal_dict)`: $O(1)$.
    *   **Complexidade:** $O(1)$.

9.  **`keys(self) -> PyVector[K]` (do exercício 18.3):**
    *   `list(self._internal_dict.keys())`: $O(N)$ para criar a lista de chaves.
    *   `PyVector(dict_keys)`: $O(N)$ para inicializar o `PyVector`.
    *   **Complexidade Total:** $O(N)$.

**Resumo da Complexidade (Implementação Imutável `PyMap[K,V]`):**
*   `__init__(M_items)`: $O(M)$ média.
*   `empty_map()`: $O(1)$.
*   `put(m, k, v)`: $O(N)$ média.
*   `get(m, k)`: $O(1)$ média.
*   `remove(m, k)`: $O(N)$ média.
*   `containsKey(m, k)`: $O(1)$ média.
*   `is_empty(m)`: $O(1)$.
*   `size(m)`: $O(1)$.
*   `keys(m)`: $O(N)$.

A imutabilidade (criação de novas cópias do dicionário para `put` e `remove`) faz com que essas operações sejam $O(N)$, enquanto as operações de consulta permanecem eficientes ($O(1)$ em média) devido à eficiência do dicionário Python subjacente. Se uma implementação mutável fosse usada (modificando o dicionário interno no local), `put` e `remove` seriam $O(1)$ em média.

**Exercício:**

Se a operação `put` na classe `PyMap[K,V]` fosse implementada para modificar o mapa no local (mutável), usando `self._internal_dict[key] = value` e a função retornasse `None` (ou `self`), qual seria a complexidade de tempo desta versão mutável de `put`?

**Resolução:**

Se a operação `put` na classe `PyMap[K,V]` fosse implementada de forma mutável:
```python
# Hypothetical mutable put method
# def put_mutable(self, key: K, value: V) -> None: # Or -> PyMap[K,V] returning self
#     self._internal_dict[key] = value
```

1.  **Operação `self._internal_dict[key] = value`:**
    Esta é a operação de inserção/atualização padrão para dicionários Python.
    Sua complexidade de tempo é **$O(1)$ em média**. No pior caso (extremamente raro com boas funções hash e tratamento de colisão), pode degradar para $O(N)$, onde $N$ é o número de itens no dicionário.

**Complexidade Total da Versão Mutável de `put`:**
A complexidade seria dominada pela operação de atribuição do dicionário.
Portanto, a complexidade de tempo desta versão mutável de `put` seria **$O(1)$ em média**.

Isso contrasta com a complexidade $O(N)$ em média da versão imutável, que precisa copiar o dicionário inteiro antes de fazer a modificação.

# 18.6 Exercícios Teóricos e Práticos

Esta seção apresenta um exercício para consolidar os conceitos do TAD `Map[K,V]`.

**Exercício:**

**Contexto:** Este exercício foca na extensão do TAD `Map[K,V]` e sua implementação `PyMap[K,V]` com uma operação para obter todos os valores.

**Parte a) Especificação Algébrica para `values` em `Map[K,V]`**
Adicione uma operação `values: Map[K,V] --> Bag[V]` à especificação algébrica `Map[K,V]` (da Seção 18.2). Esta operação deve retornar um multiconjunto (`Bag`) de todos os valores presentes no mapa. A ordem não importa, e se múltiplos pares chave-valor diferentes tiverem o mesmo valor, esse valor aparecerá múltiplas vezes no multiconjunto (uma vez para cada chave à qual está associado).
Assuma que você tem um TAD `Bag[V]` (multiconjunto de V) com operações:
*   `emptyBag: --> Bag[V]`
*   `addValueToBag: V x Bag[V] --> Bag[V]`
Defina os axiomas para `values(m: Map[K,V])` em termos dos construtores de `Map` (`emptyMap`, `put`) e das operações de `Bag`.

**Parte b) Implementação Python para `get_values_as_bag` em `PyMap[K,V]`**
Implemente um método `get_values_as_bag(self) -> PyBag[V]` na classe `PyMap[K,V]` (da Seção 18.3). Este método deve retornar uma instância de uma classe `PyBag[V]` (você pode assumir uma implementação simples de `PyBag[V]` que usa uma lista interna para armazenar elementos e tem métodos `empty_bag()` e `add_value(v)`).

**Parte c) Testes de Propriedade para `get_values_as_bag` em `PyMap[K,V]`**
Usando Hypothesis, escreva pelo menos um teste de propriedade para sua implementação `get_values_as_bag`. Uma propriedade poderia ser que `size(get_values_as_bag(m))` (onde `size` é do TAD `Bag`, significando o número total de itens, incluindo duplicatas) é igual a `size(m)` (onde `size` é do TAD `Map`, significando o número de pares chave-valor).

**Resolução:**

**Parte a) Especificação Algébrica para `values` em `Map[K,V]`**

Assumimos a especificação `Bag[V]` com `emptyBag` e `addValueToBag`.

**SPEC ADT** MapWithValues[ParamK: KeyTypeRequirement, ParamV: ValueType]
**imports:**
*   ParamK
*   ParamV
*   Natural
*   Boolean
*   Map[ParamK, ParamV]
*   Bag[ParamV] (* Assume esta especificação existe *)

**sorts:**
*   Map (* herdado *)
*   Bag (* do Bag[ParamV] *)

**operations:**
*   values: Map --> Bag

**axioms:**
for all m: Map, k: K, v: V

*   (MV1): values(emptyMap) = emptyBag
*   (MV2): values(put(k, v, m)) =
        ifThenElse(containsKey(k, m),
                   addValueToBag(v, removeValueFromBag(get(k,m), values(m))),
                   addValueToBag(v, values(m)))
    (* Nota: removeValueFromBag(val, bag) é uma operação hipotética para Bag,
       que remove uma ocorrência de val. Esta definição é complexa devido à
       possível atualização de valor para uma chave existente. *)

    (* Uma definição mais simples se focarmos apenas nos valores, sem nos preocupar
       em remover o valor antigo na atualização de um put. No entanto, para a propriedade
       size(values(m)) = size(m) ser verdadeira, a semântica de 'values' deve
       corresponder a um valor por chave única. *)

    (* Definição alternativa e mais comum para 'values' que simplesmente coleta todos os valores presentes: *)
*   (MV2_alt): values(put(k, v, m)) = addValueToBag(v, values(remove(k,m)))
    (* Esta forma é mais simples: o saco de valores de put(k,v,m) é v adicionado ao
       saco de valores do mapa m sem a chave k (para evitar contar o valor antigo de k, se houver).
       Isso garante que cada chave contribua com seu valor (mais recente) exatamente uma vez. *)

**END SPEC**

A especificação `MapWithValues` enriquece `Map[K,V]` (e implicitamente importa `Bag[V]`).
A nova operação `values` mapeia um `Map` para um `Bag` de seus valores.
*   (MV1): O saco de valores de um `emptyMap` é um `emptyBag`.
*   (MV2_alt): O saco de valores de `put(k,v,m)` é o valor `v` adicionado (`addValueToBag`) ao saco de valores do mapa que resulta da remoção da chave `k` de `m` (`values(remove(k,m))`). Isso garante que, se `k` já estava em `m` com um valor antigo, esse valor antigo não é incluído (pois `remove(k,m)` o tira), e apenas o novo valor `v` é adicionado. Se `k` não estava em `m`, `remove(k,m)` é `m`, então é `addValueToBag(v, values(m))`. Esta definição garante que o número de itens no `Bag` é igual ao número de chaves no `Map`.

**Parte b) Implementação Python para `get_values_as_bag` em `PyMap[K,V]`**

Primeiro, uma implementação mock simples de `PyBag[V]` para o exercício:
```python
# py_bag.py (mock implementation)
from __future__ import annotations
from typing import TypeVar, Generic, Optional, List as PyListInternal, Iterable, Any

V_Bag = TypeVar('V_Bag')

class PyBag(Generic[V_Bag]):
    _elements: PyListInternal[V_Bag]

    def __init__(self, initial_elements: Optional[Iterable[V_Bag]] = None) -> None:
        self._elements = list(initial_elements) if initial_elements else []

    @staticmethod
    def empty_bag() -> PyBag[Any]:
        return PyBag()

    def add_value(self, value: V_Bag) -> PyBag[V_Bag]:
        # Immutable add
        new_elements = self._elements + [value]
        return PyBag(new_elements)
    
    def size(self) -> int:
        return len(self._elements)

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, PyBag):
            return NotImplemented
        # Equality for bags is tricky (multiset equality), for simplicity here use list equality
        # if we assume a canonical representation (e.g. sorted) or just for testing specific axioms.
        # For this exercise, direct list comparison is fine if the construction order is consistent.
        # A true bag equality would count occurrences.
        # Let's assume for testing the property size(values(m))=size(m), this __eq__ is not directly used.
        # The test will compare sizes.
        # If we need to compare bags, a Counter-based comparison would be better.
        # For now, using this for simplicity, assuming tests will focus on properties like size.
        # A robust __eq__ for bag: from collections import Counter; return Counter(self._elements) == Counter(other._elements)
        return sorted(self._elements) == sorted(other._elements) # Order-insensitive for simple values

    def __repr__(self) -> str:
        return f"PyBag({self._elements})"

```

Agora, o método em `PyMap[K,V]`:
```python
# py_map.py (adicionando à classe PyMap[K,V])
# ... (imports e definições de classe anteriores) ...
# from py_bag import PyBag # Assuming py_bag.py is in the same directory or accessible

class PyMap(Generic[K, V]):
    # ... (previous methods) ...

    def get_values_as_bag(self) -> PyBag[V]:
        # Returns a PyBag containing all values in the map.
        # Each value appears as many times as it is a value for a unique key.
        if self.is_empty():
            return PyBag.empty_bag()
        
        # Python's dict.values() returns a view of the dictionary's values
        # For PyBag, we can just add all of them.
        # PyBag's constructor takes an iterable.
        return PyBag(list(self._internal_dict.values()))
    # ...
```
O método `get_values_as_bag` é adicionado à classe `PyMap[K,V]`. Se o mapa está vazio, retorna um `PyBag` vazio. Caso contrário, ele obtém todos os valores do dicionário interno usando `self._internal_dict.values()`, converte a visão de valores para uma lista, e usa essa lista para inicializar e retornar um novo `PyBag[V]`.

**Parte c) Testes de Propriedade para `get_values_as_bag` em `PyMap[K,V]`**

```python
# test_py_map.py (continuação)
# ... (importações e estratégias st_py_map como antes) ...
# from py_bag import PyBag # Assuming PyBag is importable

@given(m=st_py_map)
def test_prop_size_of_values_bag_equals_map_size(m: PyMap[str, int]):
    """
    Tests the property: size(values(m)) = size(m)
    Python: m.get_values_as_bag().size() == m.size()
    """
    
    # Get the bag of values
    values_bag = m.get_values_as_bag()
    
    # Get the size of the bag (number of items, including duplicates from different keys)
    size_of_bag = values_bag.size()
    
    # Get the size of the map (number of unique key-value pairs)
    size_of_map = m.size()
    
    assert size_of_bag == size_of_map

# Adicionalmente, podemos testar o axioma MV2_alt:
# values(put(k, v, m)) = addValueToBag(v, values(remove(k,m)))
@given(m=st_py_map, k=st_key_type, v=st_value_type)
@settings(suppress_health_check=[HealthCheck.filter_too_much], max_examples=50) # Due to remove creating copies
def test_prop_values_of_put(m: PyMap[str, int], k: str, v: int):
    """
    Tests axiom (MV2_alt): values(put(k,v,m)) = addValueToBag(v, values(remove(k,m)))
    """
    # LHS: values(put(k, v, m))
    map_after_put = m.put(k, v)
    lhs_bag = map_after_put.get_values_as_bag()
    
    # RHS: addValueToBag(v, values(remove(k,m)))
    map_after_remove_k = m.remove(k)
    bag_of_removed_map = map_after_remove_k.get_values_as_bag()
    rhs_bag = bag_of_removed_map.add_value(v)
    
    # Equality for bags needs to be multiset equality.
    # Our PyBag.__eq__ uses sorted lists, which works for simple comparable elements.
    assert lhs_bag == rhs_bag
```
Os testes de propriedade para `get_values_as_bag` são adicionados:
*   `test_prop_size_of_values_bag_equals_map_size`: Verifica a propriedade de que o número de valores no `PyBag` retornado (que é o número de itens, já que `dict.values()` retorna um valor por chave) é igual ao número de pares chave-valor no `PyMap`.
*   `test_prop_values_of_put`: Tenta verificar o axioma (MV2_alt). Ele constrói o lado esquerdo e o lado direito do axioma usando as operações de `PyMap` e `PyBag` e depois os compara. A comparação de `PyBag`s foi definida usando listas ordenadas para simplicidade, o que funciona se os valores `V` forem ordenáveis e hasheáveis para o `Counter` se uma comparação de multiconjunto mais robusta fosse usada. Para este teste, com valores inteiros, `sorted(list)` deve funcionar para a igualdade de `PyBag`.

Este capítulo introduziu o TAD `Map[K,V]`, sua especificação algébrica formal parametrizada, e uma implementação Python genérica `PyMap[K,V]` com tipagem estática. Demonstramos como os axiomas definem o comportamento de um mapa associativo e como eles podem ser traduzidos em testes de propriedade com Hypothesis para validar a implementação. A análise de complexidade e um exercício prático estenderam a compreensão do TAD. O próximo capítulo, Capítulo 19, investigará o TAD `HashTable[K,V]`, uma das implementações concretas mais comuns e eficientes para o TAD `Map[K,V]`, explorando os conceitos de funções hash, tratamento de colisões e suas implicações de desempenho.

---
## REFERÊNCIAS BIBLIOGRÁFICAS
AHO, A. V.; HOPCROFT, J. E.; ULLMAN, J. D. **Data Structures and Algorithms**. Reading, MA: Addison-Wesley, 1983.
*   *Resumo: Um texto clássico que discute tabelas de símbolos e mapas (dicionários) como estruturas fundamentais, cobrindo os conceitos básicos de chaves, valores e operações associativas. Relevante para a definição abstrata do TAD Map[K,V].*

CORMEN, T. H.; LEISERSON, C. E.; RIVEST, R. L.; STEIN, C. **Introduction to Algorithms**. 3. ed. Cambridge, MA: MIT Press, 2009.
*   *Resumo: Este livro de referência contém capítulos detalhados sobre tabelas hash e árvores de busca balanceadas, que são implementações comuns do TAD Map. A discussão sobre tabelas de endereçamento direto e dicionários é particularmente relevante.*

GOODRICH, M. T.; TAMASSIA, R.; GOLDWASSER, M. H. **Data Structures and Algorithms in Python**. Hoboken, NJ: Wiley, 2013.
*   *Resumo: Apresenta o TAD Mapa (Map) e suas implementações em Python, incluindo aquelas baseadas em listas não ordenadas, listas ordenadas (com busca binária) e tabelas hash. Relevante para a implementação `PyMap[K,V]` e sua análise.*

KNUTH, D. E. **The Art of Computer Programming, Volume 3: Sorting and Searching**. 2. ed. Reading, MA: Addison-Wesley, 1998.
*   *Resumo: Embora focado em ordenação e busca, este volume da obra monumental de Knuth discute extensivamente tabelas de símbolos e técnicas de busca por chave, que são a essência do TAD Map. Particularmente, a seção sobre hashing é um clássico.*

PYTHON SOFTWARE FOUNDATION. **Python `dict` documentation and `typing` module documentation**. Disponível em: https://docs.python.org/3/library/stdtypes.html#mapping-types-dict e https://docs.python.org/3/library/typing.html. Acesso em: 10 ago. 2023.
*   *Resumo: A documentação do Python para o tipo `dict` (a base para `PyMap`) e para o módulo `typing` (para `TypeVar`, `Generic`) é crucial para a implementação prática de `PyMap[K,V]` em Python e para a compreensão das ferramentas de genericidade e dos requisitos de hasheabilidade de chaves.*

SEDGEWICK, R.; WAYNE, K. **Algorithms**. 4. ed. Upper Saddle River, NJ: Addison-Wesley Professional, 2011.
*   *Resumo: Inclui capítulos sobre tabelas de símbolos (STs), que são uma aplicação direta do TAD Map. Discute várias implementações, incluindo árvores de busca binária e tabelas hash, com análises de desempenho, fornecendo um contexto prático para o TAD.*
---
