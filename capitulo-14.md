---
# Capítulo 14
# Tipo Abstrato de Dados BinarySearchTree
---

Dando prosseguimento ao estudo de estruturas de dados hierárquicas não lineares, este capítulo foca no Tipo Abstrato de Dados (TAD) `BinarySearchTree[T]`, ou Árvore Binária de Busca. Esta estrutura de dados é uma especialização da Árvore Binária (discutida no Capítulo 13) que impõe uma restrição de ordem crucial sobre os elementos armazenados em seus nós: para qualquer nó $x$, todos os valores na subárvore esquerda de $x$ são menores que o valor em $x$, e todos os valores na subárvore direita de $x$ são maiores que o valor em $x$. (Convenções para valores iguais variam; aqui, para simplificar, assumiremos que não há duplicatas ou que elas são tratadas de forma consistente, por exemplo, inserindo à direita). Esta propriedade de ordem da árvore binária de busca (BST) permite operações de busca, inserção e remoção eficientes, com desempenho médio logarítmico em relação ao número de nós, tornando-a uma estrutura fundamental para armazenamento e recuperação de dados ordenados. O capítulo seguirá a metodologia estabelecida: iniciaremos com uma definição abstrata do TAD `BinarySearchTree[T]`, enfatizando a propriedade de busca binária e suas operações chave como inserção (que mantém a propriedade), busca, remoção (que também deve manter a propriedade), e travessias que exploram a ordem (como a travessia em ordem). Em seguida, desenvolveremos uma especificação algébrica formal para `BinarySearchTree[T]`, parametrizada pelo tipo de elemento `T` (que agora exigirá uma relação de ordem total). Os axiomas se concentrarão em definir o comportamento das operações de forma a preservar o invariante da árvore de busca. Uma análise de consistência e completeza será discutida. Posteriormente, projetaremos e implementaremos uma classe genérica `PyBinarySearchTree[T]` em Python, com tipagem MyPy. A validação da implementação em relação aos axiomas será realizada usando Teste Baseado em Propriedades com Hypothesis. Finalmente, analisaremos a complexidade algorítmica das operações e proporemos exercícios.

# 14.1 Definição Abstrata e Relevância do Tipo de Dados `BinarySearchTree[T]`

O Tipo Abstrato de Dados `BinarySearchTree[T]` (Árvore Binária de Busca, ou BST) é uma estrutura de dados baseada em nós, onde cada nó armazena um elemento do tipo `T` (que deve ser ordenável) e possui, no máximo, dois filhos: um filho esquerdo e um filho direito. A característica fundamental que distingue uma BST de uma árvore binária genérica é a **propriedade da árvore binária de busca**:

Para qualquer nó $x$ na árvore:
*   Se $y$ é um nó na subárvore esquerda de $x$, então o valor de $y$ é menor que o valor de $x$ (i.e., $value(y) < value(x)$).
*   Se $z$ é um nó na subárvore direita de $x$, então o valor de $z$ é maior que o valor de $x$ (i.e., $value(z) > value(x)$).

Esta propriedade deve ser mantida recursivamente para todos os nós da árvore. Para esta definição, assumiremos que os elementos na árvore são **únicos**. Se duplicatas fossem permitidas, a propriedade teria que ser ajustada (e.g., menor ou igual para a esquerda, ou maior ou igual para a direita, ou uma política específica para duplicatas).

**Definição Abstrata:**
Um `BinarySearchTree[T]` é um TAD que representa uma coleção ordenada de elementos únicos do tipo `T` (onde `T` possui uma relação de ordem total). As operações essenciais incluem:

1.  **Criação:**
    *   `emptyBST: --> BinarySearchTree[T]`: Cria uma BST vazia.
2.  **Modificação (preservando a propriedade BST):**
    *   `insert: T x BinarySearchTree[T] --> BinarySearchTree[T]`: Insere um elemento `T` na BST, mantendo a propriedade da árvore de busca. Se o elemento já existe, a árvore permanece inalterada (devido à unicidade).
    *   `delete: T x BinarySearchTree[T] --> BinarySearchTree[T]`: Remove um elemento `T` da BST, mantendo a propriedade da árvore de busca. Se o elemento não existe, a árvore permanece inalterada. A remoção, especialmente de nós com dois filhos, é a operação mais complexa de se definir corretamente.
3.  **Busca/Consulta:**
    *   `contains: T x BinarySearchTree[T] --> Boolean`: Verifica se um elemento `T` está presente na BST.
    *   `findMin: BinarySearchTree[T] --> T`: Retorna o menor elemento na BST. (Parcial: não definida para BST vazia).
    *   `findMax: BinarySearchTree[T] --> T`: Retorna o maior elemento na BST. (Parcial: não definida para BST vazia).
4.  **Observação:**
    *   `isEmpty: BinarySearchTree[T] --> Boolean`: Verifica se a BST está vazia.
    *   `inOrderTraversal: BinarySearchTree[T] --> List[T]`: Retorna uma lista dos elementos da BST em ordem crescente (uma propriedade chave da travessia em ordem de uma BST). (Aqui, `List[T]` é o TAD Lista genérica).

**Relevância do TAD `BinarySearchTree[T]`:**
A propriedade da árvore binária de busca é o que confere a esta estrutura sua principal vantagem: eficiência em operações de busca, inserção e remoção.

1.  **Busca Eficiente:** A busca por um elemento em uma BST pode ser realizada em tempo proporcional à altura da árvore, $O(h)$. Em uma BST razoavelmente balanceada, a altura $h$ é $O(\log N)$, onde $N$ é o número de nós. Isso leva a um desempenho de busca logarítmico em média. No pior caso (árvore degenerada), a altura pode ser $O(N)$.

2.  **Inserção e Remoção Eficientes:** Similarmente, a inserção e a remoção de elementos, enquanto mantêm a propriedade BST, também podem ser feitas em tempo $O(h)$, levando a $O(\log N)$ em média para árvores balanceadas.

3.  **Manutenção da Ordem:** Uma BST armazena os elementos de forma ordenada. Uma travessia em ordem (`inOrderTraversal`) da árvore visita os nós em ordem crescente de seus valores.

4.  **Base para Estruturas Mais Avançadas:** BSTs são a base para estruturas de dados mais sofisticadas que garantem o balanceamento e, portanto, desempenho logarítmico no pior caso, como Árvores AVL (Capítulo 15), Árvores Rubro-Negras, Árvores B (Capítulo 17).

5.  **Implementação de Conjuntos e Mapas Ordenados:** BSTs são uma forma natural de implementar TADs `Set[T]` (Conjunto Ordenado) e `Map[K,V]` (Mapa Ordenado, onde as chaves K são ordenadas).

**Desafios:**
O principal desafio das BSTs simples é que seu desempenho depende crucialmente de sua forma, que é determinada pela ordem de inserção e remoção dos elementos. Sem mecanismos de balanceamento, uma BST pode degenerar, levando ao desempenho de pior caso $O(N)$.

Neste capítulo, focaremos na especificação e implementação de uma BST básica, assumindo que o tipo `T` dos elementos é comparável.

**Exercício:**

Dada uma Árvore Binária de Busca `bst` contendo os números naturais $\{10, 5, 15, 3, 7, 12, 17\}$. Se você realizar a operação `insert(8, bst)`, descreva a estrutura da árvore resultante (onde o 8 seria inserido) para manter a propriedade da BST. Em seguida, qual seria o resultado de uma `inOrderTraversal` da nova árvore?

**Resolução:**

A árvore inicial `bst` contendo $\{10, 5, 15, 3, 7, 12, 17\}$ teria uma estrutura como (uma das possibilidades, dependendo da ordem de inserção original, mas respeitando a propriedade BST):
```
      10
     /  \
    5    15
   / \   / \
  3   7 12 17
```

**Inserção de `8`:**
Para inserir `8`, seguimos a propriedade da BST a partir da raiz:
1.  Comparar `8` com `10` (raiz): `8 < 10`, então vamos para a subárvore esquerda (nó `5`).
2.  Comparar `8` com `5`: `8 > 5`, então vamos para a subárvore direita de `5` (nó `7`).
3.  Comparar `8` com `7`: `8 > 7`, então tentamos ir para a subárvore direita de `7`.
4.  A subárvore direita de `7` está vazia. Portanto, `8` é inserido como o filho direito de `7`.

**Estrutura da Árvore Resultante:**
```
      10
     /  \
    5    15
   / \   / \
  3   7 12 17
       \
        8 
```

**Resultado de `inOrderTraversal` da Nova Árvore:**
A travessia em ordem (esquerda, raiz, direita) de uma BST sempre produz os elementos em ordem crescente.
Para a árvore resultante, a travessia em ordem seria:
3, 5, 7, 8, 10, 12, 15, 17.

Portanto, o resultado de `inOrderTraversal(insert(8, bst))` seria a lista $\langle 3, 5, 7, 8, 10, 12, 15, 17 \rangle$.

# 14.2 Especificação Algébrica Formal do TAD `BinarySearchTree[T]`

A especificação algébrica de `BinarySearchTree[T]` (BST) deve capturar a propriedade de ordem. Para isso, o tipo do elemento `T` (o parâmetro) deve fornecer operações de comparação.

**SPEC ADT** OrderedElementType
**sorts:**
*   T
**operations:**
*   eqT: T x T --> Boolean
*   ltT: T x T --> Boolean
**axioms:**
for all x,y,z: T
*   (OE1): eqT(x,x) = true
*   (OE2): eqT(x,y) = eqT(y,x)
*   (OE3): eqT(x,y) = true AND eqT(y,z) = true => eqT(x,z) = true
*   (OE4): ltT(x,y) = true AND ltT(y,z) = true => ltT(x,z) = true
*   (OE5): eqT(x,y) = true => ltT(x,y) = false
*   (OE6): or(ltT(x,y), or(eqT(x,y), ltT(y,x))) = true
**END SPEC**

A especificação `OrderedElementType` define os requisitos para o tipo `T` dos elementos: um sort `T` e operações de igualdade `eqT` e "menor que" `ltT`, com axiomas para relação de equivalência (OE1-OE3), transitividade de `<` (OE4), irreflexividade de `<` em relação a `=` (OE5), e totalidade da ordem (OE6 - tricotomia).

**SPEC ADT** BinarySearchTree[ParamT: OrderedElementType]
**imports:**
*   ParamT
*   Boolean
*   Natural
*   List[ParamT]

**sorts:**
*   BinarySearchTree

**operations:**
*   emptyBST: --> BinarySearchTree
*   makeBST: T x BinarySearchTree x BinarySearchTree --> BinarySearchTree
*   insert: T x BinarySearchTree --> BinarySearchTree
*   contains: T x BinarySearchTree --> Boolean
*   delete: T x BinarySearchTree --> BinarySearchTree
*   findMin: BinarySearchTree --> T
*   isEmpty: BinarySearchTree --> Boolean
*   inOrder: BinarySearchTree --> List

**axioms:**
for all bst, l, r: BinarySearchTree, elem, x, y: T

*   (BST1): isEmpty(emptyBST) = true
*   (BST2): isEmpty(makeBST(elem, l, r)) = false

*   (BST3): contains(elem, emptyBST) = false
*   (BST4): eqT(elem, x) = true => contains(elem, makeBST(x, l, r)) = true
*   (BST5): ltT(elem, x) = true => contains(elem, makeBST(x, l, r)) = contains(elem, l)
*   (BST6): ltT(x, elem) = true => contains(elem, makeBST(x, l, r)) = contains(elem, r)

*   (BST7): insert(elem, emptyBST) = makeBST(elem, emptyBST, emptyBST)
*   (BST8): eqT(elem, x) = true => insert(elem, makeBST(x, l, r)) = makeBST(x, l, r)
*   (BST9): ltT(elem, x) = true => insert(elem, makeBST(x, l, r)) = makeBST(x, insert(elem, l), r)
*   (BST10): ltT(x, elem) = true => insert(elem, makeBST(x, l, r)) = makeBST(x, l, insert(elem, r))

*   (BST11): inOrder(emptyBST) = nil
*   (BST12): inOrder(makeBST(elem, l, r)) = append(inOrder(l), cons(elem, inOrder(r)))

*   (BST13): isEmpty(l) = true => findMin(makeBST(elem, l, r)) = elem
*   (BST14): isEmpty(l) = false => findMin(makeBST(elem, l, r)) = findMin(l)

    (* Axiomas para delete são complexos e geralmente omitidos em especificações introdutórias, *)
    (* ou definidos por casos (nó folha, nó com um filho, nó com dois filhos). *)
    (* Por exemplo, um caso simples: *)
*   (BST_DEL_EMPTY): delete(elem, emptyBST) = emptyBST
*   (BST_DEL_LEAF_MATCH): ltT(x,elem)=false AND ltT(elem,x)=false AND isEmpty(l)=true AND isEmpty(r)=true =>
                        delete(elem, makeBST(x,l,r)) = emptyBST

**END SPEC**

A especificação `BinarySearchTree[ParamT: OrderedElementType]` define o TAD BST. Ela importa o parâmetro `ParamT` (que fornece `T`, `eqT`, `ltT`), `Boolean`, `Natural` (para uso futuro potencial, e.g., altura) e `List[ParamT]` (para o resultado de `inOrder`). O sort principal é `BinarySearchTree`.
Os **construtores** são `emptyBST` e `makeBST(elem, left_subtree, right_subtree)`. Note que `makeBST` é um construtor "potencial" que só forma uma BST válida se as subárvores `l` e `r` e o elemento `elem` respeitarem a propriedade BST. As operações `insert` e `delete` são os construtores semânticos que garantem essa propriedade.
**Operações:** `insert` adiciona um elemento, `contains` verifica a presença, `delete` remove, `findMin` encontra o menor, `isEmpty` verifica vacuidade, `inOrder` retorna a lista ordenada.
**Axiomas:**
*   (BST1)-(BST2) definem `isEmpty`.
*   (BST3)-(BST6) definem `contains` recursivamente, usando `eqT` e `ltT` para guiar a busca.
*   (BST7)-(BST10) definem `insert` recursivamente, garantindo a propriedade BST. (BST8) lida com o caso de elemento já existente (não insere duplicatas).
*   (BST11)-(BST12) definem `inOrder` para produzir uma `List` ordenada (assumindo `List` com `nil`, `cons`, `append`).
*   (BST13)-(BST14) definem `findMin` recursivamente, sempre buscando na subárvore esquerda.
*   Axiomas para `delete` são complexos. (BST_DEL_EMPTY) é o caso base. (BST_DEL_LEAF_MATCH) mostra um caso para remover uma folha que casa com o elemento. A remoção completa exige casos para nós com um filho e dois filhos (geralmente envolvendo `findMin` da subárvore direita ou `findMax` da subárvore esquerda).

## 14.2.1 Análise de Corretude e Completeza da Especificação `BinarySearchTree[T]`

Analisaremos a especificação `BinarySearchTree[T]` quanto à consistência e completeza suficiente, considerando `emptyBST` e `insert` como os principais construtores semânticos que mantêm a propriedade BST, ou `emptyBST` e `makeBST` como construtores sintáticos (neste caso, os axiomas para `insert`, `delete` devem garantir que eles produzem termos `makeBST` que são BSTs válidas). Assumiremos `emptyBST` e `makeBST` como construtores sintáticos para a análise de completeza das outras operações.

**Consistência:**
1.  **Proteção dos Tipos Importados:** `Boolean`, `Natural`, `List[T]` e o parâmetro `T` com `eqT`/`ltT`. Os axiomas de BST operam sobre estes mas não parecem redefinir suas igualdades (e.g., não se deduz `true=false`). Os axiomas para `contains`, `insert`, `findMin` retornam os tipos corretos.
2.  **Consistência Interna de `BinarySearchTree[T]`:**
    *   `emptyBST` é distinto de `makeBST(...)` devido a (BST1), (BST2).
    *   A distinção entre BSTs não-vazias diferentes é garantida pela propriedade de ordem e pelas operações `contains` e `inOrder`. Se duas árvores são diferentes, ou `contains` dará resultados diferentes para algum elemento, ou `inOrder` produzirá listas diferentes.
    *   Os axiomas para `insert` (BST7-BST10) são projetados para manter a propriedade BST. Os axiomas para `contains` (BST3-BST6) e `findMin` (BST13-BST14) exploram essa propriedade.
3.  **Modelo Inicial:** O modelo inicial seria árvores binárias que satisfazem a propriedade de ordem, onde os elementos são de um tipo `T` ordenado. Este modelo é consistente.

A especificação `BinarySearchTree[T]` parece **consistente**, assumindo que as operações `eqT` e `ltT` do parâmetro `T` formam uma relação de ordem total consistente.

**Completeza Suficiente:**
Analisamos `isEmpty`, `contains`, `insert`, `inOrder`, `findMin` (e `delete` parcialmente) aplicadas a `emptyBST` e `makeBST(elem, l, r)`.

1.  **`isEmpty(bst: BinarySearchTree)`:** Suficientemente completa (casos BST1, BST2).
2.  **`contains(elem_to_find: T, bst: BinarySearchTree)`:**
    *   Caso $bst = \text{emptyBST}$: `contains(elem_to_find, emptyBST)` $\rightarrow_{BST3}$ `false`.
    *   Caso $bst = \text{makeBST(x, l, r)}$: Os axiomas (BST4), (BST5), (BST6) cobrem os três casos da relação de ordem entre `elem_to_find` e `x` (`eqT(elem_to_find, x)`, `ltT(elem_to_find, x)`, `ltT(x, elem_to_find)`). A totalidade da ordem (OE6) garante que um desses casos se aplica. A recursão é sobre `l` ou `r`, que são estruturalmente menores.
    `contains` é suficientemente completa.

3.  **`insert(elem_to_insert: T, bst: BinarySearchTree)`:**
    *   Caso $bst = \text{emptyBST}$: `insert(elem_to_insert, emptyBST)` $\rightarrow_{BST7}$ `makeBST(elem_to_insert, emptyBST, emptyBST)`. (Resultado é um termo construtor `BinarySearchTree`).
    *   Caso $bst = \text{makeBST(x, l, r)}$: Similar a `contains`, os axiomas (BST8), (BST9), (BST10) cobrem os três casos da relação de ordem entre `elem_to_insert` e `x`. A recursão é sobre `l` ou `r`.
    `insert` é suficientemente completa (retorna um termo `BinarySearchTree` construído).

4.  **`inOrder(bst: BinarySearchTree)`:**
    *   Caso $bst = \text{emptyBST}$: `inOrder(emptyBST)` $\rightarrow_{BST11}$ `nil`. (Resultado `List[T]` construtor).
    *   Caso $bst = \text{makeBST(x, l, r)}$: `inOrder(makeBST(x, l, r))` $\rightarrow_{BST12}$ `append(inOrder(l), cons(x, inOrder(r)))`. Por indução, `inOrder(l)` e `inOrder(r)` produzem `List[T]` construtores, e `append` e `cons` são construtores de `List[T]`.
    `inOrder` é suficientemente completa.

5.  **`findMin(bst: BinarySearchTree)`:**
    *   Caso $bst = \text{emptyBST}$: `findMin(emptyBST)`. Nenhum axioma cobre este caso. Indefinido.
    *   Caso $bst = \text{makeBST(x, l, r)}$:
        *   Se $l = \text{emptyBST}$ (`isEmpty(l)=true`): `findMin(makeBST(x, emptyBST, r))` $\rightarrow_{BST13}$ `x`. (Resultado `T`).
        *   Se $l \neq \text{emptyBST}$ (`isEmpty(l)=false`): `findMin(makeBST(x, l, r))` $\rightarrow_{BST14}$ `findMin(l)`. Recursão sobre `l`.
    `findMin` é suficientemente completa para árvores não-vazias. Parcial para `emptyBST`.

6.  **`delete(elem_to_delete: T, bst: BinarySearchTree)`:**
    *   Caso $bst = \text{emptyBST}$: `delete(elem_to_delete, emptyBST)` $\rightarrow_{BST\_DEL\_EMPTY}$ `emptyBST`.
    *   Para $bst = \text{makeBST(x,l,r)}$, a especificação fornecida só tem um axioma para remover uma folha que casa. Para ser suficientemente completa, `delete` precisaria de axiomas cobrindo:
        *   Elemento a ser deletado é menor que `x` (recursão na subárvore esquerda).
        *   Elemento a ser deletado é maior que `x` (recursão na subárvore direita).
        *   Elemento a ser deletado é igual a `x`, e `x` é:
            *   Uma folha (coberto por BST_DEL_LEAF_MATCH, se as condições se aplicam).
            *   Tem apenas filho direito.
            *   Tem apenas filho esquerdo.
            *   Tem dois filhos (o caso mais complexo, usualmente envolve `findMin` da subárvore direita ou `findMax` da esquerda).
    Com os axiomas atuais, `delete` **não é suficientemente completa**.

**Conclusão da Análise:**
*   **Consistência:** A especificação `BinarySearchTree[T]` parece consistente.
*   **Completeza Suficiente:** `isEmpty`, `contains`, `insert`, `inOrder` são. `findMin` é parcial (indefinida para `emptyBST`). `delete` é significativamente incompleta com os axiomas fornecidos.

**Exercício:**

Para a especificação `BinarySearchTree[T]`, adicione uma operação `findMax: BinarySearchTree --> T` e seus axiomas correspondentes, similarmente a `findMin`. `findMax` deve retornar o maior elemento na BST e é parcial (não definida para `emptyBST`).

**Resolução:**

Adicionamos `findMax` à especificação `BinarySearchTree[T]`.

**SPEC ADT** BinarySearchTreeWithFindMax[ParamT: OrderedElementType]
**imports:**
*   ParamT
*   Boolean
*   Natural
*   List[ParamT]
*   BinarySearchTree[ParamT]

**sorts:**
*   BinarySearchTree

**operations:**
*   findMax: BinarySearchTree --> T

**axioms:**
for all bst, l, r: BinarySearchTree, elem: T

*   (BST_FM1): isEmpty(r) = true => findMax(makeBST(elem, l, r)) = elem
*   (BST_FM2): isEmpty(r) = false => findMax(makeBST(elem, l, r)) = findMax(r)

**END SPEC**

A especificação `BinarySearchTreeWithFindMax` enriquece `BinarySearchTree[T]`. A nova operação `findMax` recebe uma `BinarySearchTree` e retorna um elemento `T`.
*   (BST_FM1): Se a subárvore direita `r` de `makeBST(elem, l, r)` está vazia, então `elem` (o valor na raiz dessa subárvore) é o maior elemento.
*   (BST_FM2): Se a subárvore direita `r` não está vazia, então o maior elemento deve estar na subárvore direita `r`. A operação é chamada recursivamente em `r`.
A operação `findMax` não é definida para `emptyBST`, tornando-a parcial.

# 14.3 Projeto e Implementação em Python com Tipagem Estática (MyPy)

Com a especificação algébrica formal do TAD `BinarySearchTree[T]`, passamos à sua implementação em Python. Criaremos uma classe genérica `PyBinarySearchTree[T]` que realiza o comportamento definido, incluindo a manutenção da propriedade da árvore binária de busca. Usaremos uma estrutura de nós para representar a árvore, e anotações de tipo MyPy para rigor. O tipo `T` será genérico, mas para as operações de comparação, a implementação Python de `T` deverá suportar os operadores `<`, `>`, `==`.

**Design da Classe `PyBinarySearchTree[T]` e `_Node[T]`:**
Uma árvore binária de busca é tipicamente implementada com uma classe para a própria árvore e uma classe interna ou auxiliar para os nós.

```python
# py_bst.py
from __future__ import annotations # For type hints within class definitions
from typing import TypeVar, Generic, Optional, List as PyListTypeHint, Any, Callable

T = TypeVar('T') # Generic type parameter for elements, should be comparable

class _Node(Generic[T]):
    # Internal class representing a node in the BST
    def __init__(self, value: T):
        self.value: T = value
        self.left: Optional[_Node[T]] = None
        self.right: Optional[_Node[T]] = None

class PyBinarySearchTree(Generic[T]):
    _root: Optional[_Node[T]]
    # Comparison functions might be passed if T doesn't have natural ordering
    # or if a custom order is needed. For simplicity, we'll use Python's
    # built-in comparison operators (<, >, ==) on T.
    # A more robust generic BST might take key extraction and comparison functions.

    def __init__(self) -> None:
        # Constructor for PyBinarySearchTree.
        self._root = None

    @staticmethod
    def empty_bst() -> PyBinarySearchTree[Any]: 
        # Corresponds to 'emptyBST: --> BinarySearchTree[T]'
        return PyBinarySearchTree()

    def is_empty(self) -> bool:
        # Corresponds to 'isEmpty: BinarySearchTree[T] --> Boolean'
        return self._root is None

    def insert(self, value: T) -> PyBinarySearchTree[T]:
        # Corresponds to 'insert: T x BinarySearchTree[T] --> BinarySearchTree[T]'
        # Returns a NEW BST with the value inserted (immutable style for the tree structure).
        # Node objects themselves might be reused if subtrees are unchanged.
        # For a truly immutable ADT, nodes should also be copied if modified.
        # This implementation modifies the node structure if seen as mutable,
        # or returns a new tree root if seen as immutable.
        # Let's aim for an interface that returns a new tree,
        # by creating a new root if the current one is None,
        # or by calling a recursive helper that reconstructs paths.

        new_tree = self._copy_tree_structure() # Create a new tree for immutability
        new_tree._root = self._insert_recursive(new_tree._root, value)
        return new_tree

    def _insert_recursive(self, current_node: Optional[_Node[T]], value: T) -> _Node[T]:
        # Recursive helper for insert. Returns the root of the modified subtree.
        if current_node is None:
            return _Node(value) # Base case: insert new node

        # Assuming T supports comparison operators <, >
        if value < current_node.value:
            # Must create a new node for the left child if immutable path is desired
            new_left_child = self._insert_recursive(current_node.left, value)
            if new_left_child is not current_node.left: # If left subtree changed
                new_current = _Node(current_node.value) # Create new current node
                new_current.left = new_left_child
                new_current.right = current_node.right # Reuse right subtree
                return new_current
            else: # Left subtree unchanged, can reuse current_node if no other changes
                return current_node 
        elif value > current_node.value:
            new_right_child = self._insert_recursive(current_node.right, value)
            if new_right_child is not current_node.right:
                new_current = _Node(current_node.value)
                new_current.left = current_node.left
                new_current.right = new_right_child
                return new_current
            else:
                return current_node
        else: # value == current_node.value (duplicate, no change based on unique elements)
            return current_node 

    def _copy_tree_structure(self) -> PyBinarySearchTree[T]:
        # Helper to create a new tree with a (shallow or deep) copy of nodes
        # For true immutability, this should be a deep copy of the path being modified.
        # For this example, let's start with a new root and rebuild paths on modification.
        # A simpler immutable approach: make insert/delete always return new PyBinarySearchTree objects
        # where the _root is the result of a recursive op that may create new nodes.
        new_tree = PyBinarySearchTree[T]() # Create a new empty BST shell
        if self._root:
            # This copy needs to be path-copying for immutability
            new_tree._root = self._deep_copy_nodes_recursive(self._root)
        return new_tree
        
    def _deep_copy_nodes_recursive(self, node: Optional[_Node[T]]) -> Optional[_Node[T]]:
        if node is None:
            return None
        new_node = _Node(node.value)
        new_node.left = self._deep_copy_nodes_recursive(node.left)
        new_node.right = self._deep_copy_nodes_recursive(node.right)
        return new_node

    def contains(self, value: T) -> bool:
        # Corresponds to 'contains: T x BinarySearchTree[T] --> Boolean'
        current = self._root
        while current is not None:
            if value == current.value:
                return True
            elif value < current.value:
                current = current.left
            else: # value > current.value
                current = current.right
        return False

    def find_min(self) -> T:
        # Corresponds to 'findMin: BinarySearchTree[T] --> T'
        if self.is_empty():
            raise ValueError("findMin from empty BST")
        current = self._root
        assert current is not None # for MyPy, due to is_empty check
        while current.left is not None:
            current = current.left
        return current.value
        
    def in_order_traversal(self) -> PyListInternal[T]:
        # Corresponds to 'inOrderTraversal: BinarySearchTree[T] --> List[T]'
        result: PyListInternal[T] = []
        self._in_order_recursive(self._root, result)
        return result

    def _in_order_recursive(self, current_node: Optional[_Node[T]], result_list: PyListInternal[T]) -> None:
        if current_node is not None:
            self._in_order_recursive(current_node.left, result_list)
            result_list.append(current_node.value)
            self._in_order_recursive(current_node.right, result_list)

    # delete operation is complex, simplified or omitted for brevity here
    # For a full implementation, it would involve finding the node,
    # and handling cases: no children, one child, two children (requiring findMin of right subtree or similar).
    # And it should also return a new tree for immutability.

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, PyBinarySearchTree):
            return NotImplemented
        # For BSTs, equality usually means same elements and same structure.
        # Or, if seen as sets, just same elements.
        # Here, compare in-order traversals for value equality in order.
        return self.in_order_traversal() == other.in_order_traversal()

    def __repr__(self) -> str:
        return f"PyBinarySearchTree({self.in_order_traversal()})"

```
O código Python acima, no arquivo `py_bst.py`, define a classe genérica `PyBinarySearchTree[T]` e uma classe interna `_Node[T]`.
`PyBinarySearchTree[T]` armazena a raiz `_root`. O construtor cria uma árvore vazia. `empty_bst()` é um método estático. `is_empty()` verifica se a raiz é `None`.
A operação `insert` é implementada para manter um estilo imutável para a estrutura da árvore, retornando uma nova `PyBinarySearchTree`. Ela usa um método auxiliar recursivo `_insert_recursive` que reconstrói o caminho modificado com novos nós. O método `_copy_tree_structure` e `_deep_copy_nodes_recursive` foram adicionados para auxiliar na criação de uma nova árvore base para as operações imutáveis, embora a cópia completa antes de cada `insert` seja ineficiente; uma cópia de caminho (path copying) durante a recursão de `_insert_recursive` é mais comum para imutabilidade eficiente. A versão de `_insert_recursive` tenta reconstruir o caminho ao retornar.
`contains` realiza a busca padrão em BST. `find_min` encontra o menor elemento. `in_order_traversal` (com `_in_order_recursive`) produz uma lista ordenada dos elementos. A operação `delete` é omitida por sua complexidade, mas seria necessária para um TAD completo. `__eq__` compara árvores com base em suas travessias em ordem. `__repr__` usa a travessia em ordem para representação.

**Exercício:**

Implemente o método `find_max(self) -> T` na classe `PyBinarySearchTree[T]`. Ele deve corresponder à operação `findMax: BinarySearchTree[T] --> T`, retornando o maior elemento na BST. Se a árvore estiver vazia, deve levantar `ValueError`.

**Resolução:**

```python
# py_bst.py (continuação da classe PyBinarySearchTree[T])

    # ... (outros métodos como definidos anteriormente) ...

    def find_max(self) -> T:
        # Corresponds to 'findMax: BinarySearchTree[T] --> T'
        # Returns the largest element in the BST.
        # Raises ValueError if the BST is empty.
        if self.is_empty():
            raise ValueError("findMax from empty BST")
        
        current = self._root
        assert current is not None # Assured by not is_empty() check
        while current.right is not None:
            current = current.right
        return current.value

    # ... (restante da classe) ...
```
O método `find_max` é adicionado à classe `PyBinarySearchTree[T]`.
Primeiro, ele verifica se a árvore está vazia usando `self.is_empty()`. Se estiver, levanta um `ValueError`.
Caso contrário, começa da raiz (`self._root`) e itera continuamente para o filho direito (`current.right`) até que não haja mais filho direito. O nó atual (`current`) nesse ponto conterá o maior valor na BST, que é então retornado. A asserção `assert current is not None` é para auxiliar o MyPy, dado que `is_empty()` já foi verificado.

# 14.4 Verificação de Propriedades Axiomáticas com Hypothesis

Com a especificação algébrica de `BinarySearchTree[T]` e a implementação `PyBinarySearchTree[T]`, podemos usar Hypothesis para verificar se a implementação satisfaz os axiomas. A chave aqui é definir uma estratégia que possa construir instâncias de `PyBinarySearchTree[T]` de forma recursiva.

**Estratégias Hypothesis para `PyBinarySearchTree[T]` e `T`:**
Usaremos `st.integers()` como estratégia para `T` e elementos únicos para simplificar (BSTs clássicas geralmente assumem elementos únicos ou uma política para duplicatas).

```python
# test_py_bst.py
import pytest
from hypothesis import given, strategies as st, assume, settings, HealthCheck
from typing import TypeVar, Any, List as PyListInternal

# Import the implementation
from py_bst import PyBinarySearchTree # Assuming py_bst.py is accessible

# Strategy for generating element type 'T'. For this example, use integers.
st_element_type = st.integers(min_value=-100, max_value=100)

# Strategy for building PyBinarySearchTree[int]
# This is complex because insert itself builds the tree and maintains invariants.
# A common way to test BSTs is to generate a list of unique items and insert them sequentially.
@st.composite
def st_py_bst_from_unique_list(draw):
    # Generate a list of unique integers to insert
    elements_to_insert = draw(st.lists(st_element_type, min_size=0, max_size=15, unique=True))
    tree = PyBinarySearchTree[int]()
    for elem in elements_to_insert:
        tree = tree.insert(elem) # Using the immutable insert
    return tree

st_py_bst = st_py_bst_from_unique_list()

# Strategy for a non-empty BST
@st.composite
def st_non_empty_py_bst(draw):
    tree = draw(st_py_bst)
    assume(not tree.is_empty())
    return tree
```
O código de teste `test_py_bst.py` configura as estratégias. `st_element_type` gera inteiros. A estratégia `st_py_bst_from_unique_list` (decorada com `@st.composite`) é uma forma comum de gerar BSTs válidas: ela primeiro gera uma lista de elementos únicos e depois insere esses elementos sequencialmente em uma BST vazia usando a própria operação `insert` da BST. Isso garante que as árvores geradas satisfaçam a propriedade BST se a operação `insert` estiver correta. `st_py_bst` é uma instância desta estratégia. `st_non_empty_py_bst` filtra para BSTs não vazias.

**Testes de Propriedade para os Axiomas de `BinarySearchTree[T]`:**
Testaremos os axiomas (BST1)-(BST12) (exceto `delete`).

```python
# test_py_bst.py (continuação)

# --- Axioms for isEmpty ---
def test_BST1_is_empty_emptyBST():
    # (BST1): isEmpty(emptyBST) = true
    assert PyBinarySearchTree[Any].empty_bst().is_empty() == True

@given(elem=st_element_type, 
       left_bst=st_py_bst, # Using the generator for valid BSTs
       right_bst=st_py_bst) 
@settings(suppress_health_check=[HealthCheck.too_slow, HealthCheck.data_too_large], deadline=1000, max_examples=50)
def test_BST2_is_empty_makeBST(elem: int, left_bst: PyBinarySearchTree[int], right_bst: PyBinarySearchTree[int]):
    # (BST2): isEmpty(makeBST(elem, l, r)) = false
    # This test assumes a direct makeBST-like constructor or internal node creation for testing
    # Our PyBinarySearchTree builds via insert. A direct test for makeBST implies internal node creation.
    # For our current PyBinarySearchTree, any non-empty tree is effectively a result of makeBST.
    # So, we test if a tree built by inserting at least one element is not empty.
    tree_with_one_elem = PyBinarySearchTree[int]().insert(elem)
    assert tree_with_one_elem.is_empty() == False
    
    # If we had direct node manipulation or a makeBST test constructor:
    # test_node = _Node(elem) # This would be internal
    # test_node.left = left_bst._root
    # test_node.right = right_bst._root
    # temp_tree = PyBinarySearchTree[int]()
    # temp_tree._root = test_node
    # assert temp_tree.is_empty() == False
    # For now, the simpler test above for a tree with one element suffices.

# --- Axioms for contains ---
@given(elem_to_find=st_element_type)
def test_BST3_contains_in_emptyBST(elem_to_find: int):
    # (BST3): contains(elem, emptyBST) = false
    empty_tree = PyBinarySearchTree[Any].empty_bst()
    assert empty_tree.contains(elem_to_find) == False

# Axioms BST4, BST5, BST6 are implicitly tested by testing insert and then contains.
# A direct test would require constructing a tree with a specific root, left, and right.
# We can test the property: if 'e' was inserted, 'e' is contained.
@given(bst=st_py_bst, elem_to_insert=st_element_type)
def test_contains_after_insert(bst: PyBinarySearchTree[int], elem_to_insert: int):
    # After inserting elem_to_insert, it must be in the tree.
    new_bst = bst.insert(elem_to_insert)
    assert new_bst.contains(elem_to_insert) == True

# --- Axioms for insert ---
# (BST7): insert(elem, emptyBST) = makeBST(elem, emptyBST, emptyBST)
# Test: inserting into empty makes a non-empty tree containing that elem.
@given(elem=st_element_type)
def test_BST7_insert_into_empty(elem: int):
    empty_tree = PyBinarySearchTree[int]()
    new_tree = empty_tree.insert(elem)
    assert not new_tree.is_empty()
    assert new_tree.contains(elem)
    assert new_tree.in_order_traversal() == [elem] # Stronger check

# (BST8): eqT(elem, x) = true => insert(elem, makeBST(x, l, r)) = makeBST(x, l, r)
# Test: inserting an existing element doesn't change the in-order traversal (our equality proxy)
@given(bst_non_empty=st_non_empty_py_bst)
@settings(suppress_health_check=[HealthCheck.filter_too_much])
def test_BST8_insert_existing_element(bst_non_empty: PyBinarySearchTree[int]):
    # Pick an element already in the tree (if non-empty)
    element_to_reinsert = bst_non_empty.find_min() # Any element from tree will do
    
    traversal_before = bst_non_empty.in_order_traversal()
    bst_after_reinsert = bst_non_empty.insert(element_to_reinsert)
    traversal_after = bst_after_reinsert.in_order_traversal()
    
    assert traversal_before == traversal_after

# (BST9) & (BST10) define recursive insertion.
# These are implicitly tested by the overall correctness of insert,
# especially that inOrderTraversal remains sorted and contains all unique inserted elements.
# A more direct test for these specific axioms would be complex without exposing tree structure.
# We can test that insert maintains the BST property.
@given(bst=st_py_bst, elem_to_insert=st_element_type)
def test_insert_maintains_bst_property(bst: PyBinarySearchTree[int], elem_to_insert: int):
    new_bst = bst.insert(elem_to_insert)
    traversal = new_bst.in_order_traversal()
    # Check if traversal is sorted
    for i in range(len(traversal) - 1):
        assert traversal[i] < traversal[i+1] # Assumes unique elements, strict order
    # Check if all inserted elements are present (if unique were inserted by st_py_bst)
    # and the new one too. This is harder without knowing original elements.
    # Simpler: check if the new element is present.
    if not bst.contains(elem_to_insert): # If it wasn't there before
         assert new_bst.contains(elem_to_insert)


# --- Axioms for inOrder ---
def test_BST11_inOrder_emptyBST():
    # (BST11): inOrder(emptyBST) = nil (empty list)
    assert PyBinarySearchTree[Any].empty_bst().in_order_traversal() == []

@given(bst=st_py_bst)
@settings(suppress_health_check=[HealthCheck.too_slow, HealthCheck.data_too_large], deadline=None)
def test_BST12_property_inOrder_is_sorted(bst: PyBinarySearchTree[int]):
    # (BST12 related property): inOrder(bst) is always sorted.
    # The axiom itself is structural. This tests a key consequence.
    traversal = bst.in_order_traversal()
    for i in range(len(traversal) - 1):
        assert traversal[i] < traversal[i+1]

# --- Axioms for findMin ---
# (BST13) & (BST14) define findMin recursively.
# Property: findMin(bst) should be the first element of inOrder(bst) for non-empty bst.
@given(bst_non_empty=st_non_empty_py_bst)
@settings(suppress_health_check=[HealthCheck.filter_too_much])
def test_findMin_is_first_in_inOrder(bst_non_empty: PyBinarySearchTree[int]):
    # bst_non_empty strategy ensures not tree.is_empty()
    min_val = bst_non_empty.find_min()
    traversal = bst_non_empty.in_order_traversal()
    assert min_val == traversal[0]

def test_findMin_on_empty_raises_exception():
    empty_tree = PyBinarySearchTree[Any].empty_bst()
    with pytest.raises(ValueError):
        empty_tree.find_min()
```
Os testes acima verificam os axiomas (BST1)-(BST14) (com `delete` omitido).
*   Testes para `isEmpty` (BST1, BST2) são feitos. O teste para BST2 é adaptado, pois nossa `PyBinarySearchTree` não expõe `makeBST` diretamente; em vez disso, testamos que inserir um elemento em uma árvore vazia resulta em uma árvore não vazia.
*   Testes para `contains` (BST3 e uma propriedade relacionada a BST4-BST6).
*   Testes para `insert` (BST7, BST8 e uma propriedade geral de manutenção da BST).
*   Testes para `inOrder` (BST11 e uma propriedade de ordenação relacionada a BST12).
*   Testes para `findMin` (uma propriedade relacionada a BST13-BST14 e o comportamento de erro).
Alguns axiomas estruturais (como BST4-BST6, BST9-BST10, BST12-BST14) são mais diretamente testados verificando as *propriedades emergentes* que eles garantem (e.g., `contains` funciona corretamente após `insert`, `inOrder` é sempre ordenada, `findMin` retorna o menor). Testar a forma exata de um axioma recursivo pode requerer introspecção na estrutura da árvore, o que é contrário à ideia de testar a interface do TAD.

**Exercício:**

Considerando a operação `findMax: BinarySearchTree --> T` e seus axiomas (BST_FM1, BST_FM2) da resolução do exercício da Seção 14.2. Escreva testes de propriedade Hypothesis para sua implementação `py_bst.PyBinarySearchTree.find_max`. Uma propriedade chave é que `findMax(bst)` deve ser o último elemento de `inOrderTraversal(bst)` para uma BST não vazia.

**Resolução:**

```python
# test_py_bst.py (continuação)

# --- Testes para findMax (do exercício 14.2) ---

@given(bst_non_empty=st_non_empty_py_bst)
@settings(suppress_health_check=[HealthCheck.filter_too_much])
def test_findMax_is_last_in_inOrder(bst_non_empty: PyBinarySearchTree[int]):
    # Property: findMax(bst) should be the last element of inOrderTraversal(bst)
    # for a non-empty bst.
    # The st_non_empty_py_bst strategy ensures the tree is not empty.
    
    max_val = bst_non_empty.find_max()
    traversal = bst_non_empty.in_order_traversal()
    
    assert max_val == traversal[-1] # Last element of the in-order list

def test_findMax_on_empty_raises_exception():
    # Test that findMax() on an empty BST raises ValueError.
    empty_tree = PyBinarySearchTree[Any].empty_bst()
    with pytest.raises(ValueError):
        empty_tree.find_max()
```
Os testes para `findMax` são adicionados:
*   `test_findMax_is_last_in_inOrder`: Verifica a propriedade fundamental de que o resultado de `find_max()` em uma BST não vazia deve ser igual ao último elemento obtido pela travessia em ordem (`in_order_traversal()`). A estratégia `st_non_empty_py_bst` é usada para garantir que a árvore não esteja vazia.
*   `test_findMax_on_empty_raises_exception`: Verifica o comportamento de erro esperado, que `find_max()` levante um `ValueError` quando chamada em uma BST vazia, conforme a implementação.

# 14.5 Análise de Complexidade Algorítmica

A eficiência das operações em uma Árvore Binária de Busca (BST) depende crucialmente da **altura** $h$ da árvore. Seja $N$ o número de nós na árvore.

*   **Árvore Balanceada:** Em uma BST razoavelmente balanceada (e.g., onde as alturas das subárvores esquerda e direita de qualquer nó diferem por no máximo uma constante, ou onde a árvore é completa ou quase completa), a altura $h$ é $O(\log N)$.
*   **Árvore Degenerada (Pior Caso):** Se a árvore degenerar em uma lista encadeada (e.g., inserindo elementos em ordem estritamente crescente ou decrescente), a altura $h$ pode ser $O(N)$.

**Análise de Complexidade das Operações de `PyBinarySearchTree[T]` (Imutável com Path Copying Implícito):**
Nossa implementação de `insert` visa um estilo imutável, reconstruindo o caminho afetado. As operações de busca não modificam a árvore.

1.  **`__init__(self)` ou `empty_bst()`:** $O(1)$.

2.  **`is_empty(self)`:** $O(1)$ (verifica se `self._root` é `None`).

3.  **`insert(self, value: T)`:**
    *   A busca pela posição de inserção (ou para verificar se o elemento existe) leva $O(h)$ tempo.
    *   Se a implementação é imutável e usa path copying (onde apenas os nós no caminho da raiz até o ponto de inserção/modificação e seus ancestrais são copiados), a criação de novos nós e a ligação das subárvores inalteradas também leva $O(h)$ tempo.
    *   **Complexidade Total:** $O(h)$. Em média $O(\log N)$, pior caso $O(N)$.
    (Nota: Nossa implementação de `insert` com `_copy_tree_structure` seguida por `_insert_recursive` pode ser menos eficiente do que path copying puro se `_copy_tree_structure` fizer uma cópia completa. Uma implementação imutável otimizada de `_insert_recursive` faria path copying implicitamente).

4.  **`contains(self, value: T)`:**
    *   Percorre um caminho da raiz até uma folha (ou até encontrar o elemento).
    *   **Complexidade:** $O(h)$. Em média $O(\log N)$, pior caso $O(N)$.

5.  **`find_min(self)` / `find_max(self)`:**
    *   Percorre o caminho mais à esquerda (para `find_min`) ou mais à direita (para `find_max`).
    *   **Complexidade:** $O(h)$. Em média $O(\log N)$, pior caso $O(N)$.

6.  **`in_order_traversal(self)`:**
    *   Visita cada nó da árvore exatamente uma vez.
    *   **Complexidade:** $O(N)$, pois todos os $N$ nós devem ser visitados e seus valores adicionados à lista.

7.  **`delete(self, value: T)` (se implementado):**
    *   A busca pelo nó a ser deletado é $O(h)$.
    *   O processo de remoção, especialmente para um nó com dois filhos (que pode envolver encontrar o sucessor em ordem, que é $O(h)$), e a subsequente reestruturação (com path copying para imutabilidade) também é $O(h)$.
    *   **Complexidade Total:** $O(h)$. Em média $O(\log N)$, pior caso $O(N)$.

**Conclusão da Análise de Complexidade:**
Para uma BST não balanceada, as operações `insert`, `delete`, `contains`, `find_min`, `find_max` têm desempenho $O(h)$, que varia de $O(\log N)$ (caso médio, árvore balanceada) a $O(N)$ (pior caso, árvore degenerada). A travessia em ordem é sempre $O(N)$. A imutabilidade com path copying eficiente não altera a ordem assintótica dessas complexidades em termos de $h$ ou $N$, mas adiciona uma sobrecarga na constante de tempo e no uso de memória devido à criação de novos nós.

**Exercício:**

Se uma Árvore Binária de Busca `PyBinarySearchTree[T]` for construída inserindo $N$ elementos que já estão perfeitamente ordenados (e.g., $1, 2, 3, \ldots, N$), qual será a altura $h$ da árvore resultante? Qual será a complexidade de tempo da operação `contains` nesta árvore específica?

**Resolução:**

Se $N$ elementos já ordenados (e.g., $1, 2, 3, \ldots, N$) são inseridos sequencialmente em uma Árvore Binária de Busca que usa a regra padrão (menor vai para a esquerda, maior vai para a direita):

1.  **Inserir 1:** Árvore é `(1)`
2.  **Inserir 2:** `2 > 1`, vai para a direita. Árvore: `1 -> R: (2)`
3.  **Inserir 3:** `3 > 1` (direita), `3 > 2` (direita). Árvore: `1 -> R: (2 -> R: (3))`
4.  ... e assim por diante.

A árvore resultante será uma **árvore degenerada**, onde cada nó (exceto o último inserido) tem apenas um filho direito (ou apenas um filho esquerdo se os elementos fossem inseridos em ordem decrescente). Ela se assemelhará a uma lista encadeada.

**Altura $h$ da Árvore Resultante:**
A altura de uma árvore é o comprimento do caminho mais longo da raiz até uma folha. Nesta árvore degenerada com $N$ nós, o caminho mais longo (e único caminho significativo) vai da raiz (elemento 1) até o último elemento inserido (elemento $N$). Este caminho tem $N-1$ arestas.
Portanto, a altura $h$ da árvore será $N-1$, o que é **$O(N)$**.

**Complexidade de Tempo da Operação `contains` Nesta Árvore:**
A operação `contains(value, bst)` em uma BST tem complexidade $O(h)$.
Como $h = O(N)$ para esta árvore degenerada:
*   **Pior Caso para `contains`:** Se buscarmos um elemento que não está na árvore mas seria posicionado no final da "lista" (e.g., buscar $N+1$), ou se buscarmos o elemento $N$ (o mais profundo), a busca percorrerá todos os $N$ níveis. A complexidade será $O(N)$.
*   **Caso Médio para `contains` (nesta árvore específica):** Se buscarmos elementos aleatórios que estão na árvore, a profundidade média de um nó ainda será proporcional a $N$. A complexidade média será $O(N)$.

Portanto, para uma BST construída com elementos já ordenados, a complexidade da operação `contains` é **$O(N)$**, tanto no pior caso quanto no caso médio para essa estrutura específica. Este é o mesmo desempenho de uma busca linear em uma lista não ordenada, e ilustra a degradação de desempenho de uma BST não balanceada.

# 14.6 Exercícios Teóricos e Práticos

Esta seção apresenta um exercício para consolidar os conceitos do TAD `BinarySearchTree[T]`.

**Exercício:**

**Contexto:** Este exercício foca na operação `delete` para o TAD `BinarySearchTree[T]`, especificamente o caso de deletar um nó folha.

**Parte a) Especificação Algébrica para `delete` (caso folha)**
Revisite a especificação `BinarySearchTree[ParamT: OrderedElementType]` da Seção 14.2. Adicione axiomas para a operação `delete: T x BinarySearchTree --> BinarySearchTree` que cubram especificamente:
1.  Deletar um elemento de uma `emptyBST` (deve retornar `emptyBST`).
2.  Deletar um elemento `elem` de uma BST `makeBST(x, l, r)` quando `elem < x` (recursão na subárvore esquerda `l`).
3.  Deletar um elemento `elem` de uma BST `makeBST(x, l, r)` quando `elem > x` (recursão na subárvore direita `r`).
4.  Deletar um elemento `elem` de uma BST `makeBST(x, l, r)` quando `elem = x` E o nó `x` é uma folha (ambas as subárvores `l` e `r` são `emptyBST`). O resultado deve ser `emptyBST`.

**Parte b) Implementação Python para `delete` (caso folha) em `PyBinarySearchTree[T]`**
Implemente parcialmente o método `delete(self, value: T) -> PyBinarySearchTree[T]` na classe `PyBinarySearchTree[T]`. Sua implementação deve, por enquanto, cobrir corretamente os quatro casos especificados na Parte (a). Para os casos de deleção de um nó com um ou dois filhos, você pode, por ora, simplesmente retornar a árvore original (ou levantar `NotImplementedError`) – o foco é implementar corretamente a lógica para os casos folha e recursão. Mantenha o estilo imutável.

**Parte c) Testes de Propriedade para `delete` (caso folha) em `PyBinarySearchTree[T]`**
Usando Hypothesis, escreva testes de propriedade para sua implementação `delete` que verifiquem:
1.  `delete(elem, emptyBST)` resulta em `emptyBST`.
2.  Se `elem` é uma folha em `bst` e `elem == x`, então `delete(x, bst)` não contém mais `x` e a propriedade BST é mantida (a travessia em ordem ainda é ordenada e os outros elementos estão presentes).
3.  Se `elem` não está em `bst`, `delete(elem, bst)` é igual a `bst`.

**Resolução:**

**Parte a) Especificação Algébrica para `delete` (caso folha)**

Adicionando à especificação `BinarySearchTree[ParamT: OrderedElementType]`:

**SPEC ADT** BinarySearchTreeWithDeleteLeaf[ParamT: OrderedElementType]
**imports:**
*   BinarySearchTree[ParamT]

**operations:**
    (* delete já declarada, aqui focamos nos axiomas *)

**axioms:**
for all bst, l, r: BinarySearchTree, elem, x: T

*   (BST_DEL1): delete(elem, emptyBST) = emptyBST
*   (BST_DEL2): ltT(elem, x) = true =>
                delete(elem, makeBST(x, l, r)) = makeBST(x, delete(elem, l), r)
*   (BST_DEL3): ltT(x, elem) = true =>
                delete(elem, makeBST(x, l, r)) = makeBST(x, l, delete(elem, r))
*   (BST_DEL4): eqT(elem, x) = true AND isEmpty(l) = true AND isEmpty(r) = true =>
                delete(elem, makeBST(x, l, r)) = emptyBST
    (* Faltam axiomas para eqT(elem,x) = true e nó não é folha *)

**END SPEC**

A especificação `BinarySearchTreeWithDeleteLeaf` enriquece `BinarySearchTree[T]`.
*   (BST_DEL1): Deletar de uma árvore vazia resulta em uma árvore vazia.
*   (BST_DEL2): Se o elemento a ser deletado é menor que o valor da raiz, a deleção ocorre recursivamente na subárvore esquerda, e a raiz e a subárvore direita são mantidas.
*   (BST_DEL3): Se o elemento a ser deletado é maior que o valor da raiz, a deleção ocorre recursivamente na subárvore direita.
*   (BST_DEL4): Se o elemento a ser deletado é igual ao valor da raiz, E a raiz é uma folha (ambas as subárvores são vazias), então o resultado da deleção é uma árvore vazia.

**Parte b) Implementação Python para `delete` (caso folha) em `PyBinarySearchTree[T]`**

```python
# py_bst.py (adicionando/modificando o método delete na classe PyBinarySearchTree[T])

    # ... (outros métodos) ...

    def delete(self, value: T) -> PyBinarySearchTree[T]:
        # Returns a NEW BST with the value removed (if present).
        # This version primarily handles deleting leaves and recursive calls.
        # Cases for nodes with one or two children are placeholders.
        
        new_tree = PyBinarySearchTree[T]() # Start with a new tree shell
        new_tree._root = self._delete_recursive(self._root, value)
        return new_tree

    def _delete_recursive(self, current_node: Optional[_Node[T]], value: T) -> Optional[_Node[T]]:
        # Recursive helper for delete. Returns the root of the modified subtree.
        if current_node is None:
            return None # Value not found, tree unchanged (or subtree is empty)

        # Assuming T supports comparison operators <, >, ==
        if value < current_node.value:
            new_left_child = self._delete_recursive(current_node.left, value)
            if new_left_child is current_node.left: # No change in left subtree
                return current_node # Can reuse the current node if no structural change needed
            # Create new node only if subtree actually changed
            new_node = _Node(current_node.value)
            new_node.left = new_left_child
            new_node.right = current_node.right # Reuse right subtree
            return new_node
        elif value > current_node.value:
            new_right_child = self._delete_recursive(current_node.right, value)
            if new_right_child is current_node.right: # No change in right subtree
                return current_node
            new_node = _Node(current_node.value)
            new_node.left = current_node.left # Reuse left subtree
            new_node.right = new_right_child
            return new_node
        else: # value == current_node.value: This is the node to delete
            # Case 1: Node is a leaf (no children)
            if current_node.left is None and current_node.right is None:
                return None # Remove the node by returning None to its parent

            # Case 2: Node has only one child
            elif current_node.left is None: # Only has right child
                return current_node.right # Promote right child
            elif current_node.right is None: # Only has left child
                return current_node.left # Promote left child
            
            # Case 3: Node has two children (Placeholder - requires findMin/findMax logic)
            # For this exercise, we are asked to focus on the leaf case.
            # A full delete would find inorder successor (min in right subtree),
            # copy its value to current_node, and then delete that successor.
            # As a placeholder for this exercise if only leaf deletion is required by spec:
            # This part would need the full logic from a standard BST delete.
            # If we strictly follow the exercise to only implement leaf case when value matches:
            # then this 'else' only correctly handles the leaf. If it's not a leaf,
            # and the exercise implies just returning original for non-leaf match,
            # the logic would be:
            # if current_node.left is None and current_node.right is None: return None
            # else: return current_node # Not deleting non-leaf matching nodes yet

            # For a more complete (but still simplified for this context) two-child case:
            # Find the smallest value in the right subtree (inorder successor)
            temp_val = self._find_min_recursive(current_node.right) # Need a helper for _Node
            new_node_val = temp_val # temp_val is guaranteed not None if right subtree exists
            
            # Create a new node with the successor's value
            new_deleted_node_replacement = _Node(new_node_val)
            new_deleted_node_replacement.left = current_node.left
            # Delete the inorder successor from the right subtree
            new_deleted_node_replacement.right = self._delete_recursive(current_node.right, new_node_val)
            return new_deleted_node_replacement

    def _find_min_recursive(self, node: Optional[_Node[T]]) -> T:
        # Helper to find min value in a subtree rooted at node. Node must not be None.
        if node is None: # Should not be called on None by delete logic for two children
            raise ValueError("Cannot find min in an empty subtree for delete.")
        current = node
        while current.left is not None:
            current = current.left
        return current.value
```
A implementação de `delete` e `_delete_recursive` é fornecida. `_delete_recursive` trata os casos:
1.  Nó não encontrado (caso base da recursão): retorna `None` (ou o nó atual se a subárvore não mudou).
2.  Valor menor que o nó atual: recursão à esquerda. Cria novo nó no caminho de volta se a subárvore esquerda mudou.
3.  Valor maior que o nó atual: recursão à direita. Cria novo nó no caminho de volta se a subárvore direita mudou.
4.  Valor igual ao nó atual (nó a ser deletado):
    *   Se é folha: retorna `None` para o pai, efetivamente removendo-o.
    *   Se tem um filho: retorna esse filho para o pai.
    *   Se tem dois filhos: encontra o sucessor em ordem (mínimo da subárvore direita), copia seu valor para o nó atual, e recursivamente deleta o sucessor de sua posição original. (Esta parte é a mais complexa e foi esboçada). A imutabilidade é mantida criando novos nós no caminho de modificação.

**Parte c) Testes de Propriedade para `delete` (caso folha) em `PyBinarySearchTree[T]`**

```python
# test_py_bst.py (continuação)

# Strategy to get a BST and an element that is a leaf (if any)
@st.composite
def st_bst_and_leaf_element(draw):
    tree = draw(st_py_bst)
    assume(not tree.is_empty())
    
    # Find a leaf element
    leaf_elements = []
    # This is a simplified way to find leaves; a full traversal would be better
    # For testing, we can just pick an element that might be a leaf
    # More robust: traverse and identify actual leaves
    
    # Let's pick min or max, which are often leaves in random trees or small trees
    # This is not guaranteed to be a leaf.
    # A better strategy would be to construct trees where we know leaf nodes.
    # Or, get all elements and try deleting them one by one.
    # For simplicity, let's use an element from the tree.
    # The property should hold even if it's not a leaf,
    # for the "delete not present" or "delete and structure is ok" aspects.
    
    all_elements = tree.in_order_traversal()
    if not all_elements: # Should be caught by assume not tree.is_empty()
        assume(False) # Force retry if somehow traversal is empty for non-empty tree
    
    element_to_delete = draw(st.sampled_from(all_elements))
    return tree, element_to_delete

# --- Testes para delete (foco nos casos da Parte a) ---

@given(elem=st_element_type)
def test_BST_DEL1_delete_from_empty(elem: int):
    # (BST_DEL1): delete(elem, emptyBST) = emptyBST
    empty_tree = PyBinarySearchTree[Any].empty_bst()
    assert empty_tree.delete(elem) == empty_tree

@given(bst=st_py_bst, elem_to_delete=st_element_type)
@settings(suppress_health_check=[HealthCheck.too_slow, HealthCheck.data_too_large], deadline=None, max_examples=50)
def test_delete_maintains_bst_properties(bst: PyBinarySearchTree[int], elem_to_delete: int):
    # Tests general properties after delete:
    # 1. BST property is maintained (inOrder is sorted)
    # 2. If element was present, it's no longer there.
    # 3. If element was not present, tree is unchanged (by value via inOrder).
    
    was_present = bst.contains(elem_to_delete)
    traversal_before_delete = bst.in_order_traversal()
    
    tree_after_delete = bst.delete(elem_to_delete)
    traversal_after_delete = tree_after_delete.in_order_traversal()
    
    # 1. Check BST property (sortedness)
    for i in range(len(traversal_after_delete) - 1):
        assert traversal_after_delete[i] < traversal_after_delete[i+1]
        
    # 2. Check element removal/presence
    assert tree_after_delete.contains(elem_to_delete) == False
    
    # 3. Check other elements
    if was_present:
        # If element was deleted, the new list should be the old one minus the element
        expected_after_delete = [e for e in traversal_before_delete if e != elem_to_delete]
        assert traversal_after_delete == expected_after_delete
        assert tree_after_delete.length() == bst.length() - 1
    else:
        # If element was not present, tree (and traversal) should be unchanged
        assert traversal_after_delete == traversal_before_delete
        assert tree_after_delete.length() == bst.length()

# Test for BST_DEL4 (deleting a leaf that matches)
# This requires a more specific strategy to create a tree and identify a leaf
@st.composite
def st_bst_with_known_leaf(draw):
    # Build a tree and identify a leaf to delete.
    # For example, insert 3 elements such that one is a clear leaf.
    # e.g., insert(10), insert(5), insert(15), insert(3) -> 3 is a leaf
    val_root = draw(st_element_type)
    val_smaller = draw(st_element_type.filter(lambda x: x < val_root))
    # val_larger = draw(st_element_type.filter(lambda x: x > val_root)) # Not needed for this leaf
    
    tree = PyBinarySearchTree[int]().insert(val_root).insert(val_smaller)
    # Now, val_smaller is a leaf, assuming val_smaller != val_root
    assume(val_smaller != val_root) # Ensure they are distinct for simplicity
    return tree, val_smaller

@given(data=st_bst_with_known_leaf)
@settings(max_examples=50)
def test_BST_DEL4_delete_matching_leaf(data: tuple[PyBinarySearchTree[int], int]):
    # (BST_DEL4): eqT(elem, x) = true AND isEmpty(l) = true AND isEmpty(r) = true =>
    #             delete(elem, makeBST(x, l, r)) = emptyBST
    # This test checks if deleting a known leaf 'x' results in 'x' no longer being contained
    # and the tree size decreasing by 1.
    bst, leaf_to_delete = data
    
    assume(bst.contains(leaf_to_delete)) # Ensure it's actually there
    # We also need to ensure it's a leaf for this specific axiom test,
    # which st_bst_with_known_leaf tries to construct.
    # A full verification of it being a leaf in the test is complex without node access.
    
    length_before = bst.length()
    bst_after_delete = bst.delete(leaf_to_delete)
    
    assert not bst_after_delete.contains(leaf_to_delete)
    assert bst_after_delete.length() == length_before - 1
    
    # Check sortedness
    traversal = bst_after_delete.in_order_traversal()
    for i in range(len(traversal) - 1):
        assert traversal[i] < traversal[i+1]

```
Os testes de propriedade para `delete` são adicionados:
*   `test_BST_DEL1_delete_from_empty`: Verifica o axioma (BST_DEL1).
*   `test_delete_maintains_bst_properties`: Um teste mais geral que verifica se, após uma deleção:
    1.  A propriedade BST é mantida (a travessia em ordem permanece ordenada).
    2.  O elemento a ser deletado não está mais na árvore.
    3.  Se o elemento estava presente, os outros elementos permanecem e o tamanho diminui. Se não estava, a árvore não muda.
    Esta propriedade cobre implicitamente os casos recursivos (BST_DEL2, BST_DEL3) e o resultado da deleção de um nó folha ou com um filho, e o caso de não encontrar o elemento.
*   `st_bst_with_known_leaf` é uma estratégia tentativa para criar uma árvore com uma folha conhecida, para testar mais diretamente o axioma (BST_DEL4). `test_BST_DEL4_delete_matching_leaf` usa esta estratégia para verificar o comportamento da deleção de uma folha. A dificuldade em gerar árvores com estruturas específicas (como uma folha garantida) apenas através da interface do TAD é uma limitação comum do PBT "black-box".

Este capítulo introduziu o TAD `BinarySearchTree[T]`, destacando sua propriedade de ordem e relevância. Apresentamos sua especificação algébrica formal, analisamos sua consistência e completeza, e detalhamos uma implementação Python genérica `PyBinarySearchTree[T]` com tipagem estática. A verificação da implementação via testes de propriedade com Hypothesis foi demonstrada para as principais operações, incluindo a complexa operação `delete` (parcialmente). Finalmente, a análise de complexidade algorítmica reforçou a dependência do desempenho da BST em sua altura. O próximo capítulo, Capítulo 15, abordará o TAD `AVLTree[T]`, uma forma de árvore binária de busca auto-balanceável que garante desempenho logarítmico no pior caso para suas operações.

---
## REFERÊNCIAS BIBLIOGRÁFICAS
AHO, A. V.; HOPCROFT, J. E.; ULLMAN, J. D. **Data Structures and Algorithms**. Reading, MA: Addison-Wesley, 1983.
*   *Resumo: Um texto clássico que fornece uma discussão fundamental sobre árvores binárias de busca, incluindo algoritmos para inserção, deleção e busca, e a importância da propriedade de ordem. Relevante para a definição abstrata e análise do TAD BST.*

CORMEN, T. H.; LEISERSON, C. E.; RIVEST, R. L.; STEIN, C. **Introduction to Algorithms**. 3. ed. Cambridge, MA: MIT Press, 2009.
*   *Resumo: Contém um capítulo detalhado sobre árvores binárias de busca, cobrindo operações, propriedades (como a travessia em ordem produzindo uma sequência ordenada), e a análise de desempenho em termos da altura da árvore. Essencial para as Seções 14.1, 14.3 e 14.5.*

GOODRICH, M. T.; TAMASSIA, R.; GOLDWASSER, M. H. **Data Structures and Algorithms in Python**. Hoboken, NJ: Wiley, 2013.
*   *Resumo: Este livro oferece uma perspectiva moderna sobre BSTs com implementações em Python, incluindo o uso de nós e recursão. É particularmente relevante para o projeto da classe `PyBinarySearchTree[T]` e a discussão de sua implementação.*

KNUTH, D. E. **The Art of Computer Programming, Volume 3: Sorting and Searching**. 2. ed. Reading, MA: Addison-Wesley, 1998.
*   *Resumo: Uma obra monumental que discute árvores de busca em grande detalhe, incluindo árvores binárias de busca e suas muitas variantes e análises. Embora denso, é a referência definitiva para um entendimento profundo.*

OKASAKI, C. **Purely Functional Data Structures**. Cambridge: Cambridge University Press, 1998.
*   *Resumo: Discute a implementação de estruturas de dados, incluindo árvores de busca, em um paradigma puramente funcional, com ênfase em imutabilidade e persistência. Relevante para a abordagem de implementação imutável de `PyBinarySearchTree[T]` e as considerações de path copying.*

SEDGEWICK, R.; WAYNE, K. **Algorithms**. 4. ed. Upper Saddle River, NJ: Addison-Wesley Professional, 2011.
*   *Resumo: Fornece uma cobertura clara e concisa de árvores binárias de busca, suas implementações e aplicações. A discussão sobre o impacto do balanceamento da árvore no desempenho é um bom prelúdio para os capítulos seguintes sobre árvores balanceadas.*
---
