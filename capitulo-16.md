---
# Capítulo 16
# Tipo Abstrato de Dados Heap
---

Este capítulo se dedica ao estudo do Tipo Abstrato de Dados (TAD) `Heap[T]`, uma estrutura de dados especializada baseada em árvores que é fundamental para a implementação eficiente de filas de prioridade e para algoritmos de ordenação como o Heapsort. Um heap mantém uma coleção de elementos de um tipo genérico `T` (sobre o qual uma relação de ordem total é definida) e suporta eficientemente operações como a inserção de novos elementos, a remoção do elemento de maior (ou menor) prioridade, e a consulta a esse elemento de maior (ou menor) prioridade. A característica definidora de um heap é a **propriedade de heap**: para qualquer nó na árvore (exceto possivelmente a raiz, dependendo da variante), o valor do nó é maior ou igual (em um max-heap) ou menor ou igual (em um min-heap) aos valores de seus filhos. Este capítulo seguirá a metodologia estabelecida: iniciaremos com uma definição abstrata do TAD `Heap[T]`, destacando sua relevância e operações canônicas. Em seguida, desenvolveremos uma especificação algébrica formal, parametrizada por `T` e uma relação de ordem sobre `T`, detalhando sorts, operações e axiomas que capturam a propriedade de heap e o comportamento das operações. Analisaremos a consistência e completeza desta especificação. Posteriormente, projetaremos e implementaremos uma classe genérica `PyHeap[T]` em Python, tipicamente usando uma representação baseada em array (lista Python) para eficiência espacial e para facilitar a manutenção da estrutura da árvore. A validação da corretude da implementação será conduzida via Teste Baseado em Propriedades com Hypothesis, traduzindo os axiomas em propriedades verificáveis. Concluiremos com uma análise da complexidade algorítmica das operações e um exercício prático.

# 16.1 Definição Abstrata e Relevância do Tipo de Dados `Heap[T]`

O Tipo Abstrato de Dados `Heap[T]`, ou simplesmente Heap, é uma estrutura de dados especializada, baseada em árvore, que satisfaz a **propriedade de heap**. Esta propriedade dita que se $P$ é um nó pai de $C$, então o valor da chave em $P$ é ordenado em relação ao valor da chave em $C$ de uma maneira específica em toda a árvore. Existem dois tipos principais de heaps:

1.  **Max-Heap:** Para qualquer nó $i$ diferente da raiz, o valor de $A[pai(i)]$ é maior ou igual ao valor de $A[i]$. Consequentemente, o maior elemento da coleção está sempre na raiz.
2.  **Min-Heap:** Para qualquer nó $i$ diferente da raiz, o valor de $A[pai(i)]$ é menor ou igual ao valor de $A[i]$. Consequentemente, o menor elemento da coleção está sempre na raiz.

Neste capítulo, focaremos primariamente no **Max-Heap**, mas os conceitos são facilmente adaptáveis para Min-Heaps. A estrutura de árvore subjacente a um heap é geralmente uma **árvore binária quase completa** (ou completa), o que permite uma representação eficiente usando um array.

**Definição Abstrata (para Max-Heap[T]):**
Um `MaxHeap[T]` é um TAD que representa uma coleção de elementos do tipo `T` (sobre o qual uma relação de ordem total $\le_T$ está definida) e suporta as seguintes operações principais:

1.  **Criação:**
    *   `emptyHeap: --> MaxHeap[T]`: Cria um heap vazio.
    *   `buildHeap: Collection[T] --> MaxHeap[T]`: Cria um heap a partir de uma coleção de elementos `T`.
2.  **Inserção:**
    *   `insert: T x MaxHeap[T] --> MaxHeap[T]`: Insere um novo elemento `T` no heap, mantendo a propriedade de max-heap.
3.  **Extração do Máximo:**
    *   `extractMax: MaxHeap[T] --> MaxHeap[T]`: Remove o elemento máximo (raiz) do heap, mantendo a propriedade de max-heap. Parcial: não definida para heap vazio.
4.  **Consulta ao Máximo:**
    *   `findMax: MaxHeap[T] --> T`: Retorna o elemento máximo (raiz) sem removê-lo. Parcial: não definida para heap vazio.
5.  **Verificação de Vacuidade:**
    *   `isEmpty: MaxHeap[T] --> Boolean`: Verifica se o heap está vazio.
6.  **(Opcional) Aumento de Chave/Prioridade:**
    *   `increaseKey: MaxHeap[T] x NodeLocator x T_newKey -> MaxHeap[T]`: Aumenta a chave de um elemento específico para `T_newKey` (que deve ser maior ou igual à chave atual) e restaura a propriedade de heap. (Requer uma forma de localizar o nó).

**Relevância do TAD `Heap[T]`:**
Heaps são estruturas de dados extremamente importantes e eficientes para diversas aplicações:

1.  **Filas de Prioridade (Priority Queues):** Esta é a aplicação mais proeminente. Heaps fornecem uma maneira eficiente de implementar filas onde cada elemento tem uma prioridade, e a operação de "remover próximo" sempre remove o elemento de maior (ou menor, para min-heap) prioridade. Inserções e extrações são tipicamente $O(\log N)$.

2.  **Algoritmo de Ordenação Heapsort:** Heapsort é um algoritmo de ordenação eficiente ($O(N \log N)$ no pior caso) e in-place (se implementado cuidadosamente com um array). Ele primeiro constrói um max-heap a partir dos dados e, em seguida, extrai repetidamente o elemento máximo para construir o array ordenado.

3.  **Seleção do K-ésimo Maior/Menor Elemento:** Heaps podem ser usados para encontrar o k-ésimo maior (usando um min-heap de tamanho k) ou menor (usando um max-heap de tamanho k) elemento em uma coleção de forma eficiente, geralmente em tempo $O(N \log k)$.

4.  **Algoritmos de Grafos:** Algoritmos como o de Dijkstra (para caminhos mínimos) e o de Prim (para árvores geradoras mínimas) frequentemente usam min-heaps (filas de prioridade) para gerenciar eficientemente os vértices a serem explorados.

5.  **Sistemas de Eventos Discretos e Escalonadores:** Onde eventos ou tarefas têm prioridades e precisam ser processados na ordem de sua prioridade.

A representação comum de um heap binário usando um array aproveita a estrutura de árvore quase completa: para um nó no índice $i$, seus filhos (se existirem) estão nos índices $2i+1$ e $2i+2$ (para indexação baseada em 0), e seu pai está no índice $\lfloor (i-1)/2 \rfloor$. Isso evita a necessidade de ponteiros explícitos, tornando a estrutura compacta em memória e com boa localidade de referência.

**Exercício:**

Considere um Max-Heap[Natural] contendo os elementos $\{10, 5, 20, 8, 15\}$. Se estes elementos fossem inseridos em um heap vazio nesta ordem, qual seria o elemento na raiz após todas as inserções? Se, após isso, a operação `extractMax` fosse realizada, qual seria o novo elemento na raiz (assumindo que o heap se reorganiza corretamente)?

**Resolução:**

Vamos simular a inserção dos elementos $\{10, 5, 20, 8, 15\}$ em um Max-Heap vazio e a subsequente operação `extractMax`.

**Inserções:**
1.  **`emptyHeap`**: Heap é $\langle \rangle$.
2.  **`insert(10, emptyHeap)`**:
    Heap: $\langle 10 \rangle$. Raiz: 10.
3.  **`insert(5, \langle 10 \rangle)`**:
    Adiciona 5. Heap (árvore): $10 \leftarrow 5$. Propriedade de Max-Heap mantida (10 > 5).
    Heap (array se fôssemos representar): `[10, 5]`. Raiz: 10.
4.  **`insert(20, \langle 10, 5 \rangle)`**:
    Adiciona 20. Heap (árvore) inicial: $10 \leftarrow 5, 20$.
    20 é filho de 10. $20 > 10$. Viola a propriedade de Max-Heap. Precisa "borbulhar para cima" (heapify-up).
    Troca 20 com 10.
    Heap (árvore) final: $20 \leftarrow 5, 10$.
    Heap (array): `[20, 5, 10]`. Raiz: 20.
5.  **`insert(8, \langle 20, 5, 10 \rangle)`**:
    Adiciona 8. Heap (árvore) inicial: $20 \leftarrow (5 \leftarrow 8), 10$. (8 se torna filho de 5).
    $8 > 5$. Viola propriedade. Troca 8 com 5.
    Heap (árvore) após primeira troca: $20 \leftarrow (8 \leftarrow 5), 10$.
    Agora 8 é filho de 20. $8 < 20$. Propriedade mantida nesse ramo.
    Heap (array): `[20, 8, 10, 5]`. Raiz: 20.
6.  **`insert(15, \langle 20, 8, 10, 5 \rangle)`**:
    Adiciona 15. Heap (árvore) inicial: $20 \leftarrow (8 \leftarrow 5), (10 \leftarrow 15)$. (15 se torna filho de 10).
    $15 > 10$. Viola propriedade. Troca 15 com 10.
    Heap (árvore) após primeira troca: $20 \leftarrow (8 \leftarrow 5), (15 \leftarrow 10)$.
    Agora 15 é filho de 20. $15 < 20$. Propriedade mantida nesse ramo.
    Heap (array): `[20, 8, 15, 5, 10]`. Raiz: 20.

Após todas as inserções, o elemento na raiz é **20**.
O heap (em representação de array, por exemplo) seria algo como `[20, 8, 15, 5, 10]` (onde filhos de `i` são `2i+1`, `2i+2`).
Árvore:
```
      20
     /  \
    8    15
   / \
  5  10
```

**Operação `extractMax`:**
1.  O elemento máximo (raiz) **20** é removido.
2.  Para preencher a lacuna na raiz, o último elemento do heap (na representação de array, seria o último elemento da última camada, aqui **10**) é movido para a raiz.
    Heap (árvore) temporário: $10 \leftarrow (8 \leftarrow 5), 15$. (O 20 foi removido, o 10 do final movido para a raiz).
    Heap (array) após remoção do 20 e antes do heapify-down do 10 (e remoção do último elemento): `[10, 8, 15, 5]`
3.  Agora, a propriedade de Max-Heap pode estar violada na raiz. O elemento 10 precisa "afundar" (heapify-down).
    Compara 10 com seus filhos (8 e 15). O maior filho é 15.
    Como $15 > 10$, troca 10 com 15.
    Heap (árvore) após troca: $15 \leftarrow (8 \leftarrow 5), 10$.
    O nó que era 15 (agora contém 10) não tem filhos, então o heapify-down para.
    Heap (array) final: `[15, 8, 10, 5]`.

Após `extractMax` e a reorganização, o novo elemento na raiz é **15**.

# 10.2 Especificação Algébrica Formal do TAD `Heap[T]`

A especificação algébrica de um `Heap[T]` é desafiadora devido à necessidade de manter a propriedade de heap globalmente e à natureza das operações `insert` e `extractMax` que envolvem reorganizações complexas (heapify-up/down). Uma especificação puramente equacional que define construtivamente essas operações pode ser muito complexa.

Frequentemente, as especificações de `Heap` focam nas propriedades observáveis ou usam uma abordagem baseada em modelos. Para uma especificação algébrica mais direta, podemos definir os construtores e os observadores, e os axiomas para `insert`, `extractMax`, `findMax` em termos de como eles interagem e o que é observável, mesmo que os axiomas não descrevam o algoritmo de "heapify" explicitamente, mas sim o resultado final esperado.

Assumimos um parâmetro `OrderedElementType` que fornece um sort `T` e uma operação de comparação `leq: T x T --> Boolean` (menor ou igual). Para um Max-Heap, usaremos `leq(e1, e2)` para significar $e1 \le e2$.

**SPEC ADT** OrderedElementType
**sorts:**
*   T
**operations:**
*   leq: T x T --> Boolean
**axioms:**
*   (LEQ1): leq(x,x) = true
*   (LEQ2): leq(x,y) = true AND leq(y,x) = true => eqT(x,y) = true
*   (LEQ3): leq(x,y) = true AND leq(y,z) = true => leq(x,z) = true
*   (LEQ4): or(leq(x,y), leq(y,x)) = true
for all x,y,z: T
(eqT: T x T --> Boolean é uma igualdade presumida ou derivada de leq)
**END SPEC**

A especificação `OrderedElementType` define o requisito para o tipo `T` do elemento: um sort `T` e uma operação `leq` (menor ou igual) que deve ser reflexiva (LEQ1), antissimétrica (LEQ2, implicando uma igualdade `eqT` se `leq` em ambas direções), transitiva (LEQ3), e total (LEQ4 - quaisquer dois elementos são comparáveis).

**SPEC ADT** MaxHeap[ParamT: OrderedElementType]
**imports:**
*   ParamT
*   Boolean
*   Natural

**sorts:**
*   MaxHeap

**operations:**
*   emptyHeap: --> MaxHeap
*   insert: T x MaxHeap --> MaxHeap
*   findMax: MaxHeap --> T
*   extractMax: MaxHeap --> MaxHeap
*   isEmpty: MaxHeap --> Boolean
*   size: MaxHeap --> Natural

**axioms:**
for all h: MaxHeap, e, e1, e2: T

*   (MH1): isEmpty(emptyHeap) = true
*   (MH2): isEmpty(insert(e, h)) = false

*   (MH3): size(emptyHeap) = zero
*   (MH4): size(insert(e, h)) = succ(size(h))

*   (MH5): findMax(insert(e, emptyHeap)) = e
*   (MH6): isEmpty(h) = false AND leq(e, findMax(h)) = true  =>
           findMax(insert(e, h)) = findMax(h)
*   (MH7): isEmpty(h) = false AND leq(findMax(h), e) = true  =>
           findMax(insert(e, h)) = e

*   (MH8): extractMax(insert(e, emptyHeap)) = emptyHeap

*   (MH9): isEmpty(h) = false AND leq(e, findMax(h)) = true =>
           extractMax(insert(e, h)) = insert(e, extractMax(h))

*   (MH10): isEmpty(h) = false AND leq(findMax(h), e) = true AND eqT(e, findMax(h)) = false =>
            extractMax(insert(e, h)) = h
    (* Se e > findMax(h), inserir e depois extrair o máximo (que é e) retorna o heap original h *)
    (* Este axioma é complexo e pode precisar de refinamento ou de uma abordagem baseada em multiconjuntos *)
    (* A propriedade mais fundamental é que extractMax(h) seguido por findMax(extractMax(h)) (se não vazio) *)
    (* deve dar o "segundo maior" elemento original. A especificação de extractMax é difícil sem *)
    (* recorrer a multiconjuntos ou a uma estrutura de "heapify". *)

    (* Uma abordagem mais comum para especificar heap algebricamente é focar nas propriedades da fila de prioridade: *)
    (* 1. findMax(insert(e,h)) = e   IF e é maior ou igual a findMax(h) (ou h é vazio) *)
    (* 2. findMax(insert(e,h)) = findMax(h) IF e é menor que findMax(h) *)
    (* 3. Se e1 > e2, então após inserir ambos, findMax deve ser e1. *)
    (* Os axiomas MH6 e MH7 tentam capturar o item 1 e 2 para findMax. *)
    (* Os axiomas MH9 e MH10 para extractMax são simplificações e não capturam completamente o processo de heapify. *)
    (* Uma especificação completa e puramente algébrica de heap é notoriamente difícil. *)
    (* Frequentemente, se usa uma abordagem de "observadores ocultos" ou se especifica como uma fila de prioridade. *)

**END SPEC**

A especificação `MaxHeap[ParamT: OrderedElementType]` define um TAD Max-Heap genérico. `OrderedElementType` fornece `T` e `leq`. A especificação importa `ParamT`, `Boolean` e `Natural`. O sort principal é `MaxHeap`.
As **operações** são `emptyHeap`, `insert`, `findMax` (parcial), `extractMax` (parcial), `isEmpty` e `size`.
Os **axiomas** tentam definir o comportamento:
(MH1)-(MH2) para `isEmpty`. (MH3)-(MH4) para `size`.
(MH5)-(MH7) para `findMax`: (MH5) é o caso base. (MH6) diz que se `e` é menor ou igual ao máximo atual, o máximo não muda. (MH7) diz que se `e` é maior que o máximo atual, `e` se torna o novo máximo. (A igualdade `eqT` em (MH7) seria derivada de `leq(x,y)` e `leq(y,x)`).
(MH8)-(MH10) para `extractMax`: (MH8) é o caso base. (MH9) e (MH10) são tentativas de definir `extractMax` após uma inserção, dependendo se o elemento inserido era ou não o novo máximo. Estes axiomas são simplificações e não capturam a complexidade total da reorganização do heap (heapify). Uma especificação algébrica completa e robusta de `extractMax` é muito difícil sem recorrer a uma semântica baseada em multiconjuntos (o que `extractMax` remove e o que permanece) ou a uma definição operacional interna (que a especificação algébrica tenta evitar).
Devido a essa complexidade, os testes de propriedade para `extractMax` e `insert` (além de `findMax`) frequentemente se baseiam mais em verificar a propriedade de heap e a equivalência de multiconjuntos de elementos antes e depois das operações (exceto pelo elemento extraído/inserido).

## 10.2.1 Análise de Corretude e Completeza da Especificação `MaxHeap[T]`

**Consistência:**
Assumindo que `OrderedElementType`, `Boolean` e `Natural` são consistentes.
1.  **Proteção dos Tipos Importados:** Os axiomas não parecem redefinir `Boolean` ou `Natural`. A operação `findMax` retorna `T`, e `leq` é usada consistentemente.
2.  **Consistência Interna:** `emptyHeap` é distinto de `insert(e,h)` (via MH1, MH2). A distinção entre heaps não-vazios é mais sutil. Os axiomas para `findMax` (MH5-MH7) parecem consistentes entre si para os casos que cobrem. Os axiomas para `extractMax` (MH8-MH10) são os mais problemáticos; sua consistência e se eles realmente definem uma operação de "extração do máximo" que mantém a propriedade de heap para todos os casos subsequentes é complexa de garantir sem uma definição mais operacional ou baseada em modelos. No entanto, se interpretarmos os axiomas como propriedades que devem valer, um modelo de heap (e.g., array binário) pode satisfazê-los.
A especificação parece **condicionalmente consistente**, com a maior ressalva sobre a completude e o poder definidor dos axiomas de `extractMax`.

**Completeza Suficiente:**
Construtores: `emptyHeap`, `insert`.
Operações não-construtoras: `isEmpty`, `size`, `findMax`, `extractMax`.

1.  **`isEmpty(h: MaxHeap)`:** Suficientemente completa (casos MH1, MH2).
2.  **`size(h: MaxHeap)`:** Suficientemente completa (casos MH3, MH4, por indução).
3.  **`findMax(h: MaxHeap)`:**
    *   Caso $h = \text{emptyHeap}$: `findMax(emptyHeap)`. Não coberto por (MH5)-(MH7). Parcial.
    *   Caso $h = \text{insert(e, h')}$:
        *   Se $h' = \text{emptyHeap}$: `findMax(insert(e, emptyHeap))` $\rightarrow_{MH5}$ `e`. (Resultado `T`).
        *   Se $h' \neq \text{emptyHeap}$: Coberto por (MH6) ou (MH7) dependendo da comparação de `e` com `findMax(h')`. Se `findMax(h')` é recursivamente bem definido (reduz a `T`), então `findMax(insert(e,h'))` reduz a `T`.
    `findMax` é suficientemente completa para heaps não-vazios.

4.  **`extractMax(h: MaxHeap)`:**
    *   Caso $h = \text{emptyHeap}$: `extractMax(emptyHeap)`. Não coberto. Parcial.
    *   Caso $h = \text{insert(e, h')}$:
        *   Se $h' = \text{emptyHeap}$: `extractMax(insert(e, emptyHeap))` $\rightarrow_{MH8}$ `emptyHeap`. (Construtor).
        *   Se $h' \neq \text{emptyHeap}$: Coberto por (MH9) ou (MH10) dependendo da comparação de `e` com `findMax(h')`. Os lados direitos `insert(e, extractMax(h'))` ou `h'` envolvem `extractMax` recursivamente ou `insert`. Se `extractMax(h')` e `findMax(h')` são bem definidos e reduzem a termos construtores (ou `T`), e se a recursão termina, então `extractMax` poderia ser suficientemente completa. No entanto, a estrutura de (MH9) e (MH10) é complexa e sua terminação e confluência como regras de reescrita não são óbvias. (MH10) retorna `h'`, que pode não estar em uma forma construtora canônica se `h'` não for ele mesmo um termo construído apenas com `emptyHeap` e `insert`.
    A completeza suficiente de `extractMax` com estes axiomas é **questionável e difícil de estabelecer** sem uma análise muito mais profunda ou uma reformulação dos axiomas.

**Conclusão da Análise:**
*   **Consistência:** Provavelmente consistente, mas os axiomas para `extractMax` são complexos.
*   **Completeza Suficiente:** `isEmpty`, `size` são. `findMax` é para heaps não-vazios. `extractMax` tem uma definição axiomática complexa cuja completeza suficiente é difícil de garantir sem uma análise de reescrita mais profunda ou uma abordagem baseada em modelos/propriedades de multiconjunto.

**Exercício:**

Adicione uma operação `buildFromArray: List[T] --> MaxHeap` à especificação `MaxHeap[T]`. Esta operação constrói um heap a partir de uma lista de elementos. Não precisa fornecer os axiomas que definem *como* `buildFromArray` funciona internamente (o algoritmo de heapify), mas forneça axiomas que relacionem `isEmpty(buildFromArray(L))` e `size(buildFromArray(L))` com as propriedades da lista de entrada `L` (assumindo que `List[T]` tem operações `isEmptyList: List[T] -> Boolean` e `lengthList: List[T] -> Natural`).

**Resolução:**

Assumimos uma especificação `List[ParamT: OrderedElementType]` importada, com operações `isEmptyList` e `lengthList`.

**SPEC ADT** MaxHeapWithBuild[ParamT: OrderedElementType]
**imports:**
*   ParamT
*   Boolean
*   Natural
*   MaxHeap[ParamT]
*   List[ParamT] (* Assume esta especificação de lista também existe *)

**sorts:**
*   MaxHeap (* herdado *)
*   List (* da importação de List[ParamT] *)

**operations:**
*   buildFromArray: List --> MaxHeap

**axioms:**
for all L: List

*   (MH_BUILD1): isEmpty(buildFromArray(L)) = isEmptyList(L)
*   (MH_BUILD2): size(buildFromArray(L)) = lengthList(L)

**END SPEC**

A especificação `MaxHeapWithBuild` enriquece `MaxHeap[T]` com a operação `buildFromArray`.
1.  **Nova Operação:**
    *   `buildFromArray: List --> MaxHeap` é adicionada. Ela recebe uma `List` (do tipo `List[T]`) e retorna um `MaxHeap`.

2.  **Novos Axiomas para `buildFromArray`:**
    *   **(MH_BUILD1): `isEmpty(buildFromArray(L)) = isEmptyList(L)`**
        Este axioma relaciona a vacuidade do heap construído com a vacuidade da lista de entrada. Se a lista `L` está vazia, o heap resultante também estará vazio, e vice-versa.
    *   **(MH_BUILD2): `size(buildFromArray(L)) = lengthList(L)`**
        Este axioma estabelece que o tamanho do heap construído a partir da lista `L` é igual ao comprimento da lista `L`. Todos os elementos da lista são incorporados ao heap.

Estes axiomas não definem o *processo* de construção do heap (o algoritmo "heapify"), mas especificam propriedades observáveis do heap resultante em termos da lista de entrada. Axiomas adicionais seriam necessários para relacionar `findMax(buildFromArray(L))` ou `extractMax(buildFromArray(L))` com os elementos de `L`, o que seria consideravelmente mais complexo.

# 10.3 Projeto e Implementação em Python com Tipagem Estática (MyPy)

Para implementar o TAD `MaxHeap[T]` em Python, criaremos uma classe genérica `PyMaxHeap[T]`. A representação interna mais comum e eficiente para um heap binário é um array (lista Python), onde a estrutura de árvore é mantida implicitamente pelas relações entre os índices. As operações de inserção (`insert`) e extração do máximo (`extract_max`) envolverão os algoritmos de "heapify-up" (ou "sift-up", "bubble-up") e "heapify-down" (ou "sift-down", "bubble-down") respectivamente, para manter a propriedade de max-heap.

A implementação será genérica em `T`. Para que a propriedade de heap seja mantida, os elementos do tipo `T` devem ser comparáveis. Em Python, isso significa que eles devem suportar operadores como `>` e `<`. A implementação assumirá que os elementos `T` fornecidos são comparáveis. Idealmente, `T` seria um `TypeVar` com um `bound` a um protocolo que define `__lt__` e `__gt__`, mas para simplificar, confiaremos na tipagem dinâmica de Python para as comparações, e MyPy poderá inferir ou verificar se os tipos usados com `PyMaxHeap` são de fato comparáveis.

Esta implementação será **mutável** para eficiência, o que é comum para heaps usados em filas de prioridade e heapsort. Isso difere do estilo imutável usado para `PyVector` e `PyStack` nos capítulos anteriores.

**Design da Classe `PyMaxHeap[T]`:**

```python
# py_max_heap.py
from __future__ import annotations # For type hints of PyMaxHeap[T] within the class itself
from typing import List as PyListInternal, Optional, TypeVar, Generic, Any, Callable

T = TypeVar('T') # Generic type parameter for elements

class PyMaxHeap(Generic[T]):
    _elements: PyListInternal[T] # Internal Python list to store heap elements
                                # Index 0 is the root (max element)

    def __init__(self, initial_elements: Optional[PyListInternal[T]] = None) -> None:
        # Constructor for PyMaxHeap.
        # If initial_elements are provided, builds a heap from them.
        self._elements = []
        if initial_elements:
            self._elements = list(initial_elements) # Make a copy
            self._build_max_heap() # Convert the list into a max-heap

    def _parent_idx(self, i: int) -> int:
        # Returns parent index of node i
        return (i - 1) // 2

    def _left_child_idx(self, i: int) -> int:
        # Returns left child index of node i
        return 2 * i + 1

    def _right_child_idx(self, i: int) -> int:
        # Returns right child index of node i
        return 2 * i + 2

    def _swap(self, i: int, j: int) -> None:
        # Swaps elements at indices i and j in the internal list
        self._elements[i], self._elements[j] = self._elements[j], self._elements[i]

    def _sift_up(self, i: int) -> None:
        # Moves element at index i up to its correct position (maintaining max-heap property)
        parent = self._parent_idx(i)
        while i > 0 and self._elements[i] > self._elements[parent]: # Max-heap: parent >= child
            self._swap(i, parent)
            i = parent
            parent = self._parent_idx(i)

    def _sift_down(self, i: int) -> None:
        # Moves element at index i down to its correct position (maintaining max-heap property)
        size = len(self._elements)
        current_idx = i
        while True:
            left_idx = self._left_child_idx(current_idx)
            right_idx = self._right_child_idx(current_idx)
            largest_idx = current_idx

            if left_idx < size and self._elements[left_idx] > self._elements[largest_idx]:
                largest_idx = left_idx
            if right_idx < size and self._elements[right_idx] > self._elements[largest_idx]:
                largest_idx = right_idx
            
            if largest_idx == current_idx:
                break # Heap property is satisfied for this subtree
            
            self._swap(current_idx, largest_idx)
            current_idx = largest_idx


    def _build_max_heap(self) -> None:
        # Converts the current _elements list into a max-heap in-place.
        # Starts from the last non-leaf node and sifts down.
        size = len(self._elements)
        # Last non-leaf node index is (size // 2) - 1
        for i in range((size // 2) - 1, -1, -1):
            self._sift_down(i)

    @staticmethod
    def empty_heap() -> PyMaxHeap[Any]: 
        # Corresponds to 'emptyHeap: --> MaxHeap[T]'
        return PyMaxHeap()

    def insert(self, value: T) -> None: # Mutating operation
        # Corresponds to 'insert: T x MaxHeap[T] --> MaxHeap[T]' (but mutates self)
        self._elements.append(value)
        self._sift_up(len(self._elements) - 1)

    def find_max(self) -> T: 
        # Corresponds to 'findMax: MaxHeap[T] --> T'.
        # Raises IndexError if heap is empty.
        if self.is_empty():
            raise IndexError("find_max from empty heap")
        return self._elements[0] # Root is the max element

    def extract_max(self) -> T: # Mutating, and returns the max element
        # Corresponds to 'extractMax: MaxHeap[T] --> MaxHeap[T]' (mutates self)
        # and also returns the max element (common in priority queue interfaces)
        if self.is_empty():
            raise IndexError("extract_max from empty heap")
        
        max_element = self._elements[0]
        last_element = self._elements.pop() # Removes last, $O(1)$
        
        if not self.is_empty(): # If heap became empty after pop
            self._elements[0] = last_element
            self._sift_down(0)
            
        return max_element

    def is_empty(self) -> bool:
        # Corresponds to 'isEmpty: MaxHeap[T] --> Boolean'
        return not self._elements 

    def size(self) -> int: 
        # Corresponds to 'size: MaxHeap[T] --> Natural'
        return len(self._elements)

    # Methods for equality and representation
    def __eq__(self, other: object) -> bool:
        # Equality for heaps is tricky. Two heaps are semantically equal if they
        # contain the same multiset of elements, regardless of internal array order.
        # A simple list comparison is not sufficient if internal order can vary
        # for the same conceptual heap (which it can't for a canonical heap array).
        # For simplicity here, we'll compare the _elements list, assuming a canonical form.
        if not isinstance(other, PyMaxHeap):
            return NotImplemented
        # This implies they must have the same internal array representation.
        # More robustly, one might sort both lists before comparing for multiset equality.
        return self._elements == other._elements

    def __repr__(self) -> str:
        # String representation.
        return f"PyMaxHeap({self._elements})"

    def to_list_internal_representation(self) -> PyListInternal[T]:
        # Helper for testing, returns internal list.
        return list(self._elements)
```
O código Python acima, no arquivo `py_max_heap.py`, define a classe genérica `PyMaxHeap[T]`.
Ela é parametrizada pelo tipo `T`. Uma lista Python interna `_elements` armazena os elementos do heap. O índice 0 da lista é a raiz.
O construtor `__init__` pode receber uma lista de elementos iniciais e, se fornecida, chama `_build_max_heap` para transformar essa lista em um max-heap.
Métodos auxiliares privados `_parent_idx`, `_left_child_idx`, `_right_child_idx`, `_swap`, `_sift_up`, e `_sift_down` implementam a lógica de navegação na árvore e os algoritmos de heapify.
O método estático `empty_heap()` cria um heap vazio.
A operação `insert` adiciona um elemento ao final da lista e depois chama `_sift_up` para restaurar a propriedade de heap. Esta é uma operação **mutável**.
A operação `find_max` retorna o elemento na raiz (índice 0), levantando `IndexError` se o heap estiver vazio.
A operação `extract_max` remove e retorna o elemento máximo. Ela move o último elemento para a raiz, remove o último da lista, e então chama `_sift_down` na raiz para restaurar a propriedade de heap. Esta também é uma operação **mutável**.
`is_empty` e `size` são observadores padrão.
`__eq__` compara a representação interna (lista de elementos). Para uma igualdade semântica de heap mais robusta (mesmo multiconjunto de elementos), seria necessário, por exemplo, extrair todos os elementos de ambos os heaps em ordem e comparar as sequências resultantes.
`__repr__` e `to_list_internal_representation` são para depuração e teste.

**Exercício:**

Implemente um método estático `heapify_list(cls, elements: PyListInternal[T]) -> PyMaxHeap[T]` na classe `PyMaxHeap[T]`. Este método deve receber uma lista Python de elementos do tipo `T` e retornar uma nova instância de `PyMaxHeap[T]` que seja um max-heap válido contendo todos esses elementos. Este método é uma alternativa ao construtor que já chama `_build_max_heap`.

**Resolução:**

```python
# py_max_heap.py (adicionando à classe PyMaxHeap[T])

    # ... (métodos anteriores, incluindo __init__ e _build_max_heap) ...

    @classmethod
    def heapify_list(cls, elements: PyListInternal[T]) -> PyMaxHeap[T]:
        # Creates a new PyMaxHeap instance from a given Python list of elements
        # by heapifying them.
        # 'cls' refers to the class PyMaxHeap itself.
        
        # Create an instance using the list (which will be copied by __init__)
        # The __init__ method already calls _build_max_heap if initial_elements are provided.
        new_heap_instance = cls(initial_elements=elements)
        return new_heap_instance

    # ... (restante da classe) ...
```
O método de classe `heapify_list` foi adicionado à `PyMaxHeap[T]`.
Ele recebe `cls` (uma referência à própria classe `PyMaxHeap`) e uma lista Python `elements` do tipo `T`.
Ele simplesmente cria uma nova instância da classe `PyMaxHeap` passando a lista `elements` para o construtor `__init__`. O construtor `__init__`, como definido anteriormente, já faz uma cópia da lista de entrada e chama `self._build_max_heap()` para transformar essa lista interna em um max-heap.
Portanto, `heapify_list` serve como um método de fábrica nomeado alternativo para criar um heap a partir de uma coleção existente, aproveitando a lógica já presente no construtor.

# 10.4 Verificação de Propriedades Axiomáticas com Hypothesis

A especificação algébrica de um heap (Seção 10.2) é complexa para definir completamente `extractMax` de forma construtiva. Os testes de propriedade para heaps frequentemente se concentram em verificar:
1.  A **propriedade de heap** após cada operação.
2.  As **propriedades de fila de prioridade**: `findMax` sempre retorna o maior elemento; `extractMax` remove o maior, e o novo `findMax` é o segundo maior anterior, etc.
3.  A **conservação do multiconjunto de elementos** (exceto pelo elemento extraído/inserido).

Dada a dificuldade de axiomas algébricos puros para `extractMax`, vamos focar os testes nas propriedades mais observáveis e naquelas relacionadas aos axiomas mais simples (MH1-MH7) e na propriedade de heap.

**Estratégias Hypothesis para `PyMaxHeap[T]` e `T`:**
Usaremos `st.integers()` para `T` e uma estratégia para construir `PyMaxHeap`.

```python
# test_py_max_heap.py
import pytest
from hypothesis import given, strategies as st, assume, settings, HealthCheck
from typing import TypeVar, Any, List as PyListInternal

# Import the implementation
from py_max_heap import PyMaxHeap # Assuming py_max_heap.py is accessible

# Strategy for generating element type 'T'. For this example, use integers.
st_element_type = st.integers(min_value=-100, max_value=100)

# Strategy for generating lists of T that will be used to build heaps
st_list_of_elements = st.lists(st_element_type, min_size=0, max_size=20)

# Strategy for PyMaxHeap[int] by building from a list
st_py_max_heap = st.builds(PyMaxHeap[int], initial_elements=st_list_of_elements)

# Helper function to check heap property (for max-heap)
def is_max_heap_valid(heap_list: PyListInternal[Any]) -> bool:
    n = len(heap_list)
    for i in range(n):
        left_child_idx = 2 * i + 1
        right_child_idx = 2 * i + 2
        if left_child_idx < n and heap_list[i] < heap_list[left_child_idx]:
            return False
        if right_child_idx < n and heap_list[i] < heap_list[right_child_idx]:
            return False
    return True
```
O código de teste `test_py_max_heap.py` configura as estratégias. `st_element_type` gera inteiros. `st_list_of_elements` gera listas desses inteiros. `st_py_max_heap` usa `st.builds` para criar instâncias de `PyMaxHeap[int]` a partir dessas listas (o construtor chamará `_build_max_heap`). A função `is_max_heap_valid` é um helper para verificar a propriedade de max-heap em uma representação de lista.

**Testes de Propriedade para Axiomas/Propriedades de `MaxHeap[T]`:**

```python
# test_py_max_heap.py (continuação)

# --- Testes para isEmpty e size (análogos a MH1-MH4) ---
def test_MH1_MH3_empty_heap_properties():
    # (MH1): isEmpty(emptyHeap) = true
    # (MH3): size(emptyHeap) = zero
    h_empty = PyMaxHeap[Any].empty_heap()
    assert h_empty.is_empty() == True
    assert h_empty.size() == 0

@given(h_list=st_list_of_elements, elem=st_element_type)
@settings(suppress_health_check=[HealthCheck.too_slow], deadline=None)
def test_MH2_MH4_insert_properties(h_list: PyListInternal[int], elem: int):
    # (MH2): isEmpty(insert(e, h)) = false
    # (MH4): size(insert(e, h)) = succ(size(h))
    h = PyMaxHeap(h_list) # Build heap from list
    original_size = h.size()
    
    h.insert(elem) # Mutating insert
    
    assert h.is_empty() == False
    assert h.size() == original_size + 1
    assert is_max_heap_valid(h._elements) # Check heap property after insert

# --- Testes para findMax (análogos a MH5-MH7) ---
@given(elem=st_element_type)
def test_MH5_findMax_insert_on_empty(elem: int):
    # (MH5): findMax(insert(e, emptyHeap)) = e
    h_empty = PyMaxHeap[Any].empty_heap()
    h_empty.insert(elem)
    assert h_empty.find_max() == elem
    assert is_max_heap_valid(h_empty._elements)

@given(h_list=st_list_of_elements, elem=st_element_type)
@settings(suppress_health_check=[HealthCheck.too_slow], deadline=None)
def test_MH6_MH7_findMax_insert_on_non_empty(h_list: PyListInternal[int], elem: int):
    # (MH6): isEmpty(h) = false AND leq(e, findMax(h)) = true => findMax(insert(e, h)) = findMax(h)
    # (MH7): isEmpty(h) = false AND leq(findMax(h), e) = true => findMax(insert(e, h)) = e
    assume(h_list) # Ensure list is not empty, so heap will not be empty
    
    h = PyMaxHeap(h_list)
    max_before_insert = h.find_max()
    
    h.insert(elem) # Mutating insert
    max_after_insert = h.find_max()
    
    if elem <= max_before_insert: # leq(elem, max_before_insert)
        assert max_after_insert == max_before_insert
    else: # leq(max_before_insert, elem) and not eq (i.e. elem > max_before_insert)
        assert max_after_insert == elem
    assert is_max_heap_valid(h._elements)

# --- Testes para extractMax (propriedades gerais) ---
@given(h_list=st_list_of_elements)
@settings(suppress_health_check=[HealthCheck.too_slow], deadline=None)
def test_extractMax_maintains_heap_property(h_list: PyListInternal[int]):
    # Test that after extract_max, the heap property is maintained.
    assume(h_list) # Ensure heap is not empty before extract_max
    h = PyMaxHeap(h_list)
    
    _ = h.extract_max() # Perform extraction
    
    if not h.is_empty():
        assert is_max_heap_valid(h._elements)
    else:
        assert len(h._elements) == 0 # Should be empty if last element was extracted

@given(h_list=st.lists(st_element_type, min_size=1, max_size=20, unique=True))
@settings(suppress_health_check=[HealthCheck.too_slow], deadline=None)
def test_extractMax_returns_and_removes_max(h_list: PyListInternal[int]):
    # Test that extract_max returns the largest element and removes it.
    # Using unique=True to simplify finding the second max.
    assume(h_list) 
    h = PyMaxHeap(list(h_list)) # Use a copy for modification
    
    max_val = h.find_max()
    extracted_max = h.extract_max()
    
    assert max_val == extracted_max
    assert h.size() == len(h_list) - 1
    
    # Check that extracted_max is no longer in the heap's elements
    # This is a simple check, a full multiset check would be more robust
    if not h.is_empty():
        assert extracted_max not in h._elements # Simple check, may fail for duplicates if not unique
        # More robust: check that find_max now returns the second largest of original list
        if len(h_list) > 1:
            remaining_elements = sorted([e for e in h_list if e != extracted_max], reverse=True)
            if remaining_elements: # If there are remaining elements
                 assert h.find_max() == remaining_elements[0]
    assert is_max_heap_valid(h._elements)

```
Os testes acima para `PyMaxHeap[int]` incluem:
*   `test_MH1_MH3_empty_heap_properties`: Testa `isEmpty` e `size` para `emptyHeap`.
*   `test_MH2_MH4_insert_properties`: Testa `isEmpty` e `size` após `insert`, e verifica a propriedade de heap. Usa `@settings` para permitir testes potencialmente mais lentos devido à construção do heap e inserção.
*   `test_MH5_findMax_insert_on_empty`: Testa `findMax` após inserir em um heap vazio.
*   `test_MH6_MH7_findMax_insert_on_non_empty`: Testa as propriedades condicionais (MH6, MH7) para `findMax` após `insert` em heap não vazio, e verifica a propriedade de heap.
*   `test_extractMax_maintains_heap_property`: Verifica se a propriedade de heap se mantém após `extract_max`.
*   `test_extractMax_returns_and_removes_max`: Verifica se `extract_max` retorna o elemento correto e o remove, e se o novo máximo (se houver) é o segundo maior original. Usa `unique=True` na estratégia de lista para simplificar a lógica de verificação do segundo máximo.

Estes testes focam em verificar as propriedades fundamentais do Max-Heap e o comportamento das operações em relação a essas propriedades, dado que uma especificação algébrica construtiva completa para `extractMax` é complexa.

**Exercício:**

Considerando a operação `buildFromArray: List[T] --> MaxHeap` e seus axiomas da resolução do exercício da Seção 10.2 (MH_BUILD1, MH_BUILD2). Escreva testes de propriedade Hypothesis para verificar esses dois axiomas para a implementação `py_max_heap.PyMaxHeap.heapify_list` (ou para o construtor `PyMaxHeap(initial_elements=...)` que tem o mesmo efeito).

**Resolução:**

```python
# test_py_max_heap.py (continuação)

# --- Testes para buildFromArray (usando o construtor ou heapify_list) ---

@given(element_list=st_list_of_elements)
def test_MH_BUILD1_isEmpty_after_buildFromArray(element_list: PyListInternal[int]):
    # (MH_BUILD1): isEmpty(buildFromArray(L)) = isEmptyList(L)
    # Here, isEmptyList(L) corresponds to (not L) in Python for a Python list L.
    
    heap_from_list = PyMaxHeap(element_list) # أو PyMaxHeap.heapify_list(element_list)
    
    assert heap_from_list.is_empty() == (not element_list)

@given(element_list=st_list_of_elements)
def test_MH_BUILD2_size_after_buildFromArray(element_list: PyListInternal[int]):
    # (MH_BUILD2): size(buildFromArray(L)) = lengthList(L)
    # Here, lengthList(L) corresponds to len(L) for a Python list L.

    heap_from_list = PyMaxHeap(element_list) # أو PyMaxHeap.heapify_list(element_list)
    
    assert heap_from_list.size() == len(element_list)
    # Also, ensure the heap property holds after building
    assert is_max_heap_valid(heap_from_list._elements)

```
Os testes para `buildFromArray` (realizado pelo construtor `PyMaxHeap(list)` ou pelo método `heapify_list`) são:
*   `test_MH_BUILD1_isEmpty_after_buildFromArray`: Verifica (MH_BUILD1), que a vacuidade do heap construído corresponde à vacuidade da lista original.
*   `test_MH_BUILD2_size_after_buildFromArray`: Verifica (MH_BUILD2), que o tamanho do heap construído é igual ao comprimento da lista original. Adicionalmente, verifica se a propriedade de max-heap é satisfeita após a construção.

# 10.5 Análise de Complexidade Algorítmica

A implementação `PyMaxHeap[T]` usa uma lista Python como base. As operações de heap dependem dos algoritmos `_sift_up` e `_sift_down`, que percorrem um caminho da árvore (altura $O(\log N)$).

**Análise de Complexidade das Operações de `PyMaxHeap[T]` (Mutável):**
Seja $N$ o número de elementos no heap.
1.  **`__init__(self, initial_elements: Optional[PyListInternal[T]] = None)`:**
    *   Se `initial_elements` é `None`: $O(1)$.
    *   Se `initial_elements` tem $K$ elementos: `list(initial_elements)` é $O(K)$. `_build_max_heap()` é $O(K)$ (o algoritmo de construção de heap linear).
    *   **Complexidade Total:** $O(K)$.

2.  **`empty_heap()`:** $O(1)$.

3.  **`insert(self, value: T)`:**
    *   `self._elements.append(value)`: $O(1)$ amortizado.
    *   `self._sift_up(...)`: $O(\log N)$ (altura da árvore).
    *   **Complexidade Total:** $O(\log N)$ amortizado.

4.  **`find_max(self) -> T`:**
    *   Acesso a `self._elements[0]`: $O(1)$.
    *   **Complexidade:** $O(1)$.

5.  **`extract_max(self) -> T`:**
    *   `self._elements[0]`: $O(1)$.
    *   `self._elements.pop()`: $O(1)$ amortizado.
    *   `self._elements[0] = last_element`: $O(1)$.
    *   `self._sift_down(0)`: $O(\log N)$.
    *   **Complexidade Total:** $O(\log N)$ amortizado.

6.  **`is_empty(self) -> bool`:** $O(1)$.

7.  **`size(self) -> int`:** $O(1)$.

8.  **`heapify_list(cls, elements: PyListInternal[T])` (método de classe):**
    *   Chama `cls(initial_elements=elements)`, que é $O(K)$ onde $K = \text{len(elements)}$.
    *   **Complexidade:** $O(K)$.

As operações chave `insert` e `extract_max` são eficientes, com complexidade logarítmica, o que torna o heap adequado para filas de prioridade. `find_max` é muito rápido ($O(1)$).

**Exercício:**

A operação `_sift_down(self, i: int)` é chamada em `_build_max_heap` e `extract_max`. Qual é a altura máxima da subárvore em que `_sift_down` pode operar, e como isso se relaciona com a complexidade $O(\log N)$ da operação?

**Resolução:**

A operação `_sift_down(self, i: int)` começa no nó `i` e potencialmente troca o elemento em `i` com um de seus filhos, descendo na árvore até que a propriedade de max-heap seja restaurada para a subárvore enraizada em `i` (ou até que um nó folha seja alcançado).

*   **Altura Máxima da Subárvore:** A altura de um heap binário (quase completo) com $N$ elementos é $\lfloor \log_2 N \rfloor$. A operação `_sift_down`, no pior caso, percorre um caminho da raiz da subárvore onde é chamada (o nó `i`) até um nó folha. A altura da subárvore enraizada em `i` é no máximo a altura total do heap, que é $O(\log N)$.

*   **Relação com a Complexidade $O(\log N)$:**
    Em cada nível do caminho que `_sift_down` percorre, ela realiza um número constante de comparações (para encontrar o maior filho) e possivelmente uma troca. Como o número de níveis que ela pode percorrer é limitado pela altura da árvore (ou subárvore), que é $O(\log N)$, o número total de operações realizadas por `_sift_down` é proporcional a essa altura.
    Portanto, a complexidade de tempo de `_sift_down` é $O(h)$, onde $h$ é a altura da subárvore em que opera. Como $h \le \log N$ para a altura total do heap, a complexidade é $O(\log N)$.

    Quando `_sift_down(0)` é chamada em `extract_max`, ela opera a partir da raiz, então $h$ é a altura do heap, e a complexidade é $O(\log N)$.
    Em `_build_max_heap`, `_sift_down` é chamada para múltiplos nós. Embora cada chamada individual seja $O(\log N)$ no máximo, uma análise mais refinada mostra que o custo total de `_build_max_heap` é $O(N)$, não $O(N \log N)$, devido ao fato de que a maioria dos nós está perto da base da árvore e tem alturas pequenas.

# 10.6 Exercícios Teóricos e Práticos

Esta seção apresenta um exercício combinado para aplicar e estender os conceitos do TAD `Heap[T]`.

**Exercício:**

**Contexto:** Este exercício foca na compreensão das propriedades de um Max-Heap e na extensão da especificação algébrica e implementação `PyMaxHeap[T]`.

**Parte a) Verificação da Propriedade de Max-Heap**
Para a implementação `PyMaxHeap[T]`, a função `is_max_heap_valid(heap_list: PyListInternal[Any]) -> bool` foi fornecida como um helper nos testes. Explique por que esta função verifica corretamente a propriedade de max-heap para uma lista que representa um heap.

**Parte b) Especificação Algébrica para `getElementsSorted: MaxHeap[T] --> List[T]`**
Adicione uma operação `getElementsSorted: MaxHeap[T] --> List[T]` à especificação algébrica `MaxHeap[T]`. Esta operação deve retornar uma `List[T]` (assumindo um TAD `List` com `nilL`, `consL`, e `appendL` como operações conhecidas) contendo todos os elementos do heap em ordem decrescente (do maior para o menor). Forneça axiomas que definam esta operação. (Dica: pense em como `extractMax` e `findMax` se relacionam com a obtenção de elementos em ordem).

**Parte c) Implementação Python para `get_elements_sorted` em `PyMaxHeap[T]`**
Implemente o método correspondente `get_elements_sorted(self) -> PyListInternal[T]` na classe `PyMaxHeap[T]`. Este método não deve modificar o heap original, mas retornar uma nova lista Python com os elementos ordenados em ordem decrescente.

**Parte d) Teste de Propriedade para `get_elements_sorted`**
Escreva um teste de propriedade Hypothesis que verifique se a lista retornada por `get_elements_sorted()` está de fato ordenada em ordem decrescente e se contém os mesmos elementos (como um multiconjunto) que o heap original.

**Resolução:**

**Parte a) Verificação da Propriedade de Max-Heap por `is_max_heap_valid`**
A função `is_max_heap_valid(heap_list)` itera por todos os nós da lista que representa o heap (de `i=0` até `n-1`). Para cada nó `i`:
1.  Calcula os índices de seus filhos esquerdo (`2*i + 1`) e direito (`2*i + 2`).
2.  Verifica se o filho esquerdo existe (`left_child_idx < n`). Se existir, compara o pai (`heap_list[i]`) com o filho esquerdo (`heap_list[left_child_idx]`). Para um max-heap, o pai deve ser maior ou igual ao filho (`heap_list[i] >= heap_list[left_child_idx]`). A função fornecida verifica `heap_list[i] < heap_list[left_child_idx]` e retorna `False` se isso for verdade, o que é correto (se o pai é menor, a propriedade é violada).
3.  Faz o mesmo para o filho direito.
Se todas essas comparações para todos os nós pais com seus filhos existentes satisfazem a condição de max-heap, a função retorna `True`. Caso contrário, retorna `False` assim que uma violação é encontrada. Isso cobre a definição da propriedade de max-heap para todos os nós.

**Parte b) Especificação Algébrica para `getElementsSorted`**

Assumimos que temos `List[T]` com `nilL: --> List`, `consL: T x List --> List` (adiciona no início), e `appendL: List x T --> List` (adiciona no final). Para ordem decrescente, é mais fácil construir com `consL` se extraímos do maior para o menor, ou com `appendL` e depois reverter, ou construir na ordem inversa. Vamos usar `appendL` para adicionar na ordem em que são extraídos (maior primeiro).

**SPEC ADT** MaxHeapWithSortedElements[ParamT: OrderedElementType]
**imports:**
*   ParamT
*   Boolean
*   Natural
*   MaxHeap[ParamT]
*   List[ParamT] (* Assumindo esta especificação de lista *)

**sorts:**
*   MaxHeap
*   List

**operations:**
*   getElementsSorted: MaxHeap --> List

**axioms:**
for all h: MaxHeap, l: List, e: T

*   (MHS1): getElementsSorted(emptyHeap) = nilL
*   (MHS2): isEmpty(h) = false =>
            getElementsSorted(h) = appendL(getElementsSorted(extractMax(h)), findMax(h))
            (* Ou, para construir em ordem decrescente diretamente com consL: *)
            (* getElementsSorted(h) = consL(findMax(h), getElementsSorted(extractMax(h))) *)
            (* A segunda versão com consL é mais natural para ordem decrescente. *)
            (* Vamos usar a versão com consL para obter ordem decrescente diretamente. *)
*   (MHS2_alt): isEmpty(h) = false =>
                 getElementsSorted(h) = consL(findMax(h), getElementsSorted(extractMax(h)))

**END SPEC**

A especificação `MaxHeapWithSortedElements` enriquece `MaxHeap[T]`. A nova operação `getElementsSorted` retorna uma `List` de `T`.
*   (MHS1): Obter elementos ordenados de um `emptyHeap` resulta em uma lista vazia (`nilL`).
*   (MHS2_alt): Se o heap `h` não está vazio, os elementos ordenados são obtidos construindo uma lista onde o primeiro elemento é `findMax(h)` (o maior), e o resto da lista são os elementos ordenados de `extractMax(h)` (o heap sem o maior elemento). Isso define a lista em ordem decrescente.

**Parte c) Implementação Python para `get_elements_sorted` em `PyMaxHeap[T]`**

```python
# py_max_heap.py (adicionando à classe PyMaxHeap[T])

    # ... (métodos anteriores) ...

    def get_elements_sorted(self) -> PyListInternal[T]:
        # Returns a new Python list with all elements from the heap,
        # sorted in descending (max-heap) order.
        # This method should not modify the original heap.
        
        # To avoid modifying self, we can work on a copy or extract all.
        # A simple way is to repeatedly extract_max.
        
        if self.is_empty():
            return []
            
        # Create a temporary list of elements from the heap
        # This is not the most efficient way if we had direct access to a copy
        # of _elements and could sort that, but it reflects the ADT operations.
        
        # Alternative 1: Create a copy and extract from copy
        # temp_heap_list = list(self._elements) # Get a copy of internal list
        # temp_heap = PyMaxHeap(temp_heap_list) # Build a heap from this copy
        
        # More directly, if we want to create a new heap instance for popping:
        temp_heap = PyMaxHeap(list(self._elements)) # Create a copy of the heap
        
        sorted_elements: PyListInternal[T] = []
        while not temp_heap.is_empty():
            sorted_elements.append(temp_heap.extract_max()) # extract_max is mutating temp_heap
            
        return sorted_elements
```
O método `get_elements_sorted` é adicionado à `PyMaxHeap[T]`.
1.  Se o heap estiver vazio, retorna uma lista Python vazia.
2.  Cria uma cópia do heap atual (`temp_heap = PyMaxHeap(list(self._elements))`) para não modificar o original.
3.  Entra em um loop `while` enquanto `temp_heap` não estiver vazio.
4.  Em cada iteração, `temp_heap.extract_max()` remove e retorna o maior elemento restante de `temp_heap`. Este elemento é adicionado à lista `sorted_elements`.
5.  Como `extract_max` sempre retorna o maior elemento atual, a lista `sorted_elements` será preenchida com os elementos em ordem decrescente.
6.  Finalmente, `sorted_elements` é retornada.

**Parte d) Teste de Propriedade para `get_elements_sorted`**

```python
# test_py_max_heap.py (continuação)
from collections import Counter # For multiset comparison

# ... (estratégias st_list_of_elements, st_py_max_heap como antes) ...

@given(h_list=st_list_of_elements)
@settings(suppress_health_check=[HealthCheck.too_slow], deadline=None)
def test_prop_get_elements_sorted_is_ordered_and_preserves_elements(h_list: PyListInternal[int]):
    # Test that get_elements_sorted returns a list that is:
    # 1. Sorted in descending order.
    # 2. Contains the same multiset of elements as the original list used to build the heap.
    
    if not h_list: # Handle empty list case for clarity, though covered by logic
        heap = PyMaxHeap(h_list)
        sorted_list_from_heap = heap.get_elements_sorted()
        assert sorted_list_from_heap == []
        return

    heap = PyMaxHeap(h_list)
    sorted_list_from_heap = heap.get_elements_sorted()
    
    # 1. Check if sorted_list_from_heap is sorted in descending order
    for i in range(len(sorted_list_from_heap) - 1):
        assert sorted_list_from_heap[i] >= sorted_list_from_heap[i+1] # For max-heap -> descending
        
    # 2. Check if it contains the same elements (multiset equality)
    # This compares the frequency of each element.
    assert Counter(sorted_list_from_heap) == Counter(h_list)
    
    # 3. (Implicitly covered by 2, but good to state) Check length
    assert len(sorted_list_from_heap) == len(h_list)
```
O teste `test_prop_get_elements_sorted_is_ordered_and_preserves_elements` verifica duas propriedades principais:
1.  **Ordenação:** Itera pela lista `sorted_list_from_heap` retornada e verifica se cada elemento é maior ou igual ao seu sucessor, garantindo a ordem decrescente (para um max-heap).
2.  **Preservação de Elementos (Multiconjunto):** Usa `collections.Counter` para comparar a frequência dos elementos na lista original `h_list` (usada para construir o heap) com a frequência dos elementos na `sorted_list_from_heap`. Isso garante que todos os elementos originais estão presentes com as mesmas multiplicidades, e nenhum elemento foi perdido ou adicionado.
O teste também verifica o comprimento, que é uma consequência da igualdade de multiconjuntos.

Este capítulo introduziu o TAD `Heap[T]`, com foco em Max-Heaps, sua relevância, especificação algébrica (com desafios notados para `extractMax`), e uma implementação Python mutável `PyMaxHeap[T]` usando um array. Discutimos a verificação de propriedades axiomáticas e de heap com Hypothesis, e analisamos a complexidade logarítmica das operações chave. O exercício prático explorou a extração de elementos ordenados. O próximo capítulo, Capítulo 17, abordará o TAD `BTree`, uma estrutura de árvore balanceada fundamental para sistemas de gerenciamento de bancos de dados e sistemas de arquivos, otimizada para operações em disco.

---
## REFERÊNCIAS BIBLIOGRÁFICAS
CORMEN, T. H.; LEISERSON, C. E.; RIVEST, R. L.; STEIN, C. **Introduction to Algorithms**. 3. ed. Cambridge, MA: MIT Press, 2009.
*   *Resumo: Este texto canônico dedica um capítulo inteiro a heaps binários e heapsort, cobrindo em detalhes a propriedade de heap, os algoritmos de heapify, e a implementação baseada em array. É uma referência primária para os conceitos e algoritmos do Capítulo 16.*

GONNET, G. H.; BAEZA-YATES, R. **Handbook of Algorithms and Data Structures: In Pascal and C**. 2. ed. Wokingham, England: Addison-Wesley, 1991.
*   *Resumo: Um manual clássico que apresenta diversas estruturas de dados, incluindo heaps e suas variantes. Fornece uma perspectiva sobre diferentes formas de implementar e analisar heaps, relevante para um entendimento mais amplo do TAD.*

KING, V. A simpler minimum spanning tree algorithm. In: *Proceedings of the Workshop on Algorithms and Data Structures (WADS)*. Lecture Notes in Computer Science, vol 709. Springer, Berlin, Heidelberg, 1993. p. 495-506.
*   *Resumo: Embora focado em algoritmos de árvore geradora mínima, este artigo e outros na área de algoritmos de grafos frequentemente utilizam heaps (filas de prioridade) como um componente crucial, ilustrando a relevância prática do TAD Heap em algoritmos avançados.*

SEDGEWICK, R.; WAYNE, K. **Algorithms**. 4. ed. Upper Saddle River, NJ: Addison-Wesley Professional, 2011.
*   *Resumo: Oferece uma discussão moderna e acessível sobre filas de prioridade e heaps binários, incluindo implementações em Java e análise de desempenho. Sua abordagem é útil para entender a aplicação prática de heaps e os tradeoffs de design.*

TARJAN, R. E. **Data Structures and Network Algorithms**. Philadelphia, PA: Society for Industrial and Applied Mathematics, 1983. (CBMS-NSF Regional Conference Series in Applied Mathematics).
*   *Resumo: Um livro influente de um especialista em estruturas de dados e algoritmos. Inclui discussões sobre heaps (como d-heaps e Fibonacci heaps) e seu uso em algoritmos de otimização de rede, mostrando a profundidade e variedade das aplicações de heaps.*

VUILLEMIN, J. A data structure for manipulating priority queues. **Communications of the ACM**, v. 21, n. 4, p. 309-315, abr. 1978.
*   *Resumo: Este artigo introduziu o conceito de heap binomial, uma variante de heap que suporta eficientemente a operação de união de dois heaps. Relevante para mostrar que o TAD "Heap" pode ter diversas realizações com diferentes tradeoffs de desempenho para operações estendidas.*
----
