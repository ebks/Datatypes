---
# Capítulo 15
# Tipo Abstrato de Dados AVLTree
---

Após a introdução às árvores binárias de busca (BSTs) no capítulo anterior, este capítulo se aprofunda em uma de suas variantes mais importantes e eficientes: a **Árvore AVL (Adelson-Velsky e Landis Tree)**. As árvores AVL são árvores binárias de busca auto-balanceadas que garantem que a altura das duas subárvores de qualquer nó difira em no máximo um. Esta propriedade de balanceamento, mantida através de operações de rotação durante inserções e remoções, assegura que as operações fundamentais da árvore (busca, inserção, remoção) mantenham uma complexidade de tempo de $O(\log N)$ no pior caso, onde $N$ é o número de nós na árvore. Isso contrasta com as BSTs simples, que podem degenerar para uma estrutura linear (com complexidade $O(N)$) no pior caso. O capítulo seguirá nossa metodologia padrão: iniciaremos com uma definição abstrata do TAD `AVLTree[T]`, destacando sua relevância como uma estrutura de dados que combina a eficiência da busca logarítmica com a garantia de desempenho no pior caso. Em seguida, apresentaremos uma especificação algébrica formal para `AVLTree[T]`, parametrizada pelo tipo de elemento `T` (que deverá ser ordenável), definindo seus sorts, operações (incluindo construtores, operações de busca, inserção, remoção e as operações internas de balanceamento e rotação, embora estas últimas possam ser mais abstratamente representadas nos axiomas de inserção/remoção) e os axiomas que governam seu comportamento e a manutenção do invariante de balanceamento AVL. Uma análise da consistência e completeza da especificação será considerada. Posteriormente, abordaremos o projeto e a implementação de uma classe genérica `PyAVLTree[T]` em Python, com tipagem MyPy. A validação da corretude da implementação em relação aos axiomas (e à propriedade de balanceamento) será discutida, com ênfase no Teste Baseado em Propriedades usando Hypothesis. Concluiremos com uma análise da complexidade algorítmica das operações da `PyAVLTree[T]` e um exercício prático.

# 15.1 Definição Abstrata e Relevância do Tipo de Dados `AVLTree[T]`

O Tipo Abstrato de Dados `AVLTree[T]` representa uma árvore binária de busca (BST) com uma restrição adicional de balanceamento. Nomeada em homenagem a seus inventores, Georgy Adelson-Velsky e Evgenii Landis (1962), uma árvore AVL garante que para cada nó na árvore, as alturas de suas duas subárvores (esquerda e direita) diferem em no máximo um. Esta diferença de altura é chamada de **fator de balanceamento** de um nó, que pode ser -1, 0 ou 1 para uma árvore AVL válida. Se, após uma operação de inserção ou remoção, o fator de balanceamento de algum nó se torna -2 ou 2, a árvore é rebalanceada através de uma ou mais operações de **rotação** (simples ou dupla).

**Definição Abstrata:**
Um `AVLTree[T]` é um TAD que armazena elementos de um tipo ordenável `T` de forma hierárquica, satisfazendo as propriedades de uma árvore binária de busca e a condição de balanceamento AVL. As operações essenciais incluem:

1.  **Criação:**
    *   `emptyAVL: --> AVLTree[T]`: Cria uma árvore AVL vazia.
2.  **Modificação:**
    *   `insertAVL: T x AVLTree[T] --> AVLTree[T]`: Insere um elemento `T` na árvore, mantendo as propriedades de BST e o balanceamento AVL.
    *   `deleteAVL: T x AVLTree[T] --> AVLTree[T]`: Remove um elemento `T` da árvore, mantendo as propriedades de BST e o balanceamento AVL.
3.  **Busca/Observação:**
    *   `memberAVL: T x AVLTree[T] --> Boolean`: Verifica se um elemento `T` está presente na árvore.
    *   `findMinAVL: AVLTree[T] --> T`: Retorna o menor elemento na árvore (parcial).
    *   `findMaxAVL: AVLTree[T] --> T`: Retorna o maior elemento na árvore (parcial).
    *   `isEmptyAVL: AVLTree[T] --> Boolean`: Verifica se a árvore está vazia.

A chave para `insertAVL` e `deleteAVL` é que elas mantêm o invariante AVL através de rotações.

**Relevância do TAD `AVLTree[T]`:**

1.  **Desempenho Garantido no Pior Caso:** Operações de busca, inserção e remoção são $O(\log N)$ no pior caso.
2.  **Eficiência em Aplicações Críticas:** Ideal para bancos de dados, sistemas de tempo real, indexação.
3.  **Estrutura de Dicionário/Conjunto Eficiente:** Boa implementação para `Map[K,V]` e `Set[T]`.
4.  **Base para Algoritmos Mais Complexos:** Útil quando dados ordenados com manipulação eficiente são necessários.
5.  **Conceito Histórico e Pedagógico:** Primeira BST auto-balanceada, importante para ensinar balanceamento e rotações.

**Desvantagens Comparativas:**
*   **Complexidade de Implementação:** Maior que BSTs simples.
*   **Overhead das Rotações:** Adiciona constante de tempo às operações.
*   **Alternativas:** Árvores Vermelho-Preto podem ter menor overhead de rebalanceamento em alguns casos.

Apesar disso, árvores AVL são importantes e eficientes, especialmente quando $O(\log N)$ no pior caso é prioritário.

**Exercício:**

Considere uma Árvore AVL. Se uma inserção causa um desbalanceamento em um nó onde o fator de balanceamento se torna +2 e a subárvore direita deste nó tem um fator de balanceamento de +1, qual tipo de rotação (ou rotações) seria necessário para restaurar o balanceamento AVL? Descreva brevemente o que essa rotação faz.

**Resolução:**

Se um nó $X$ tem fator de balanceamento +2 (subárvore direita mais pesada) e seu filho direito $Y$ tem fator de balanceamento +1 (subárvore direita de $Y$ mais pesada), este é o caso de desbalanceamento "direita-direita" (RR).

A rotação necessária é uma **Rotação Simples à Esquerda** em torno do nó $X$.

**Descrição da Rotação Simples à Esquerda em torno de $X$:**
1.  $Y$ (filho direito de $X$) torna-se a nova raiz da subárvore.
2.  $X$ torna-se o filho esquerdo de $Y$.
3.  A subárvore esquerda de $Y$ (se existir, $T_2$) torna-se a nova subárvore direita de $X$.
As outras subárvores ($T_1$ de $X$ e $T_3$ de $Y$) mantêm suas posições relativas aos seus pais originais (que agora podem ter novos pais/filhos). As alturas são atualizadas.

# 15.2 Especificação Algébrica Formal do TAD `AVLTree[T]`

A especificação algébrica de uma `AVLTree[T]` foca nas propriedades observáveis e no comportamento das operações principais, assumindo que a manutenção do invariante AVL é uma propriedade semântica fundamental. O parâmetro `T` deve ser um tipo ordenável.

**SPEC ADT** OrderedElementType
**sorts:**
*   T
**operations:**
*   ltT: T x T --> Boolean
*   eqT: T x T --> Boolean
**axioms:**
*   (OET1): eqT(x,x) = true
*   (OET2): eqT(x,y) = eqT(y,x)
*   (OET3): not(ltT(x,x)) = true
*   (OET4): ltT(x,y) AND ltT(y,z) => ltT(x,z) = true
*   (OET5): eqT(x,y) => (ltT(x,z) = ltT(y,z) AND ltT(z,x) = ltT(z,y))
*   (OET6): or(or(ltT(x,y), ltT(y,x)), eqT(x,y)) = true
for all x,y,z: T
**END SPEC**

A especificação `OrderedElementType` define os requisitos para o tipo de elemento `T` de uma `AVLTree`. Ela requer um sort `T` e duas operações de comparação: `ltT` (menor que) e `eqT` (igual a), ambas retornando um `Boolean`. Os axiomas (OET1) e (OET2) estabelecem que `eqT` é reflexiva e simétrica (a transitividade seria necessária para uma relação de equivalência completa). O axioma (OET3) estabelece que `ltT` é irreflexiva. (OET4) estabelece a transitividade de `ltT`. (OET5) relaciona `eqT` e `ltT` (substitutividade). (OET6) estabelece a tricotomia (para quaisquer x, y, ou x < y, ou y < x, ou x = y).

**SPEC ADT** AVLTree[ParamT: OrderedElementType]
**imports:**
*   ParamT
*   Boolean
*   Natural

**sorts:**
*   AVLTree

**operations:**
*   emptyAVL: --> AVLTree
*   insertAVL: T x AVLTree --> AVLTree
*   deleteAVL: T x AVLTree --> AVLTree
*   memberAVL: T x AVLTree --> Boolean
*   isEmptyAVL: AVLTree --> Boolean
*   heightAVL: AVLTree --> Natural
*   isValidAVL: AVLTree --> Boolean
*   isBST: AVLTree --> Boolean

**axioms:**
for all t: AVLTree, elem, x: T

*   (AVL1): isEmptyAVL(emptyAVL) = true
*   (AVL2): isEmptyAVL(insertAVL(elem, t)) = false

*   (AVL3): memberAVL(elem, emptyAVL) = false
*   (AVL4): memberAVL(elem, insertAVL(x, t)) =
        ifThenElse(eqT(elem, x),
            true,
            memberAVL(elem, t))

*   (AVL5): isBST(insertAVL(elem, t)) = true
*   (AVL6): isBST(deleteAVL(elem, t)) = true
*   (AVL7): isBST(emptyAVL) = true

*   (AVL8): isValidAVL(insertAVL(elem, t)) = true
*   (AVL9): isValidAVL(deleteAVL(elem, t)) = true
*   (AVL10): isValidAVL(emptyAVL) = true

*   (AVL11): heightAVL(emptyAVL) = zero

*   (AVL12): memberAVL(elem, deleteAVL(elem, t)) = false
*   (AVL13): eqT(elem, x) = false =>
        memberAVL(elem, deleteAVL(x, t)) = memberAVL(elem, t)

*   (AVL14): memberAVL(elem, insertAVL(elem,t)) = true

**END SPEC**

A especificação `AVLTree[ParamT: OrderedElementType]` define um TAD para Árvores AVL genéricas. Ela importa `ParamT`, `Boolean` e `Natural`. O sort principal é `AVLTree`. As operações incluem `emptyAVL`, `insertAVL`, `deleteAVL`, `memberAVL`, `isEmptyAVL`, `heightAVL`, e dois predicados hipotéticos: `isValidAVL` (verifica o invariante de balanceamento AVL) e `isBST` (verifica a propriedade de ordenação da árvore binária de busca).
Os axiomas (AVL1)-(AVL2) definem `isEmptyAVL`.
(AVL3) e (AVL14) definem o comportamento de `memberAVL` em relação a `emptyAVL` e `insertAVL` (o axioma AVL4 é uma simplificação; uma definição completa de `memberAVL` em uma BST seria recursiva sobre a estrutura interna da árvore, que não é explicitada aqui por construtores de nó, mas é implicitamente mantida por `isBST`).
Os axiomas (AVL5)-(AVL7) postulam que `emptyAVL` é uma BST e que `insertAVL` e `deleteAVL` preservam a propriedade de BST.
Os axiomas (AVL8)-(AVL10) postulam que `emptyAVL` é uma AVL válida e que `insertAVL` e `deleteAVL` preservam a validade AVL.
(AVL11) define a altura de uma árvore vazia.
(AVL12)-(AVL13) definem o efeito de `deleteAVL` na pertença de elementos.
Esta especificação foca nas propriedades que devem ser mantidas, em vez de detalhar os algoritmos de rotação, que são complexos para uma especificação puramente equacional sem operações de baixo nível. Os predicados `isBST` e `isValidAVL` seriam definidos por outro conjunto de axiomas ou verificados externamente na implementação durante os testes.

## 10.2.1 Análise de Corretude e Completeza da Especificação `AVLTree[T]`

Analisar a consistência e completeza de uma especificação como `AVLTree[T]` com axiomas de propriedade (como `isBST` e `isValidAVL`) é diferente de analisar TADs com axiomas puramente construtivos/destrutivos.

**Consistência:**
A consistência depende da consistência dos predicados hipotéticos `isBST` e `isValidAVL` e das operações importadas. Assumindo que `Boolean`, `Natural` e `ParamT` são consistentes, e que é possível definir `isBST` e `isValidAVL` de forma consistente (o que é, pois correspondem a propriedades matemáticas bem definidas):
1.  **Proteção dos Tipos Importados:** Os axiomas não parecem redefinir ou contradizer as propriedades de `Boolean`, `Natural` ou `T`.
2.  **Consistência Interna:** Não há contradições óbvias entre os axiomas. Os axiomas de propriedade (AVL5-AVL10) postulam que as operações de modificação mantêm certos invariantes. Se esses invariantes são de fato alcançáveis (o que são para árvores AVL), a especificação pode ser consistente.
3.  **Modelo Inicial:** O modelo pretendido são as árvores binárias de busca que satisfazem a condição de balanceamento AVL. Como tais árvores existem e as operações podem ser definidas para manter essas propriedades, um modelo consistente existe.

A especificação `AVLTree[T]`, focada em propriedades, é provavelmente **consistente**.

**Completeza Suficiente:**
Aqui, `emptyAVL` e `insertAVL` são os principais construtores (com `deleteAVL` também modificando a estrutura). As operações a serem analisadas seriam `isEmptyAVL`, `memberAVL`, `heightAVL`, `isValidAVL`, `isBST`.

1.  **`isEmptyAVL(avl: AVLTree)`:** Suficientemente completa (casos AVL1, AVL2).
2.  **`memberAVL(elem: T, avl: AVLTree)`:**
    *   Com (AVL3) e (AVL14), e os axiomas para `deleteAVL` (AVL12, AVL13), temos uma cobertura para `memberAVL` em relação às operações modificadoras. A completeza para uma estrutura de árvore arbitrária dependeria de axiomas que decompõem a árvore (não presentes aqui, mas implícitos em `isBST`). Se consideramos `insertAVL` como o único construtor de árvores não vazias, então (AVL4) ou uma forma recursiva de `memberAVL` baseada na estrutura interna (se exposta) definiria o comportamento. Com a especificação atual, a completeza de `memberAVL` é mais baseada em suas interações com `insertAVL` e `deleteAVL`.
3.  **`heightAVL(avl: AVLTree)`:**
    *   (AVL11) define `heightAVL(emptyAVL)`.
    *   Não há axioma construtivo para `heightAVL(insertAVL(e, t'))`. A especificação é incompleta para `heightAVL` neste sentido.
4.  **`isValidAVL(avl: AVLTree)` e `isBST(avl: AVLTree)`:**
    *   Os axiomas (AVL5-AVL10) afirmam que o resultado dessas operações é `true` após as operações de construção/modificação. No entanto, não há axiomas que definam construtivamente o que `isValidAVL` ou `isBST` retornam para uma árvore arbitrária (e.g., como eles inspecionam a árvore). São incompletas como operações definidas axiomaticamente, mas servem como invariantes a serem mantidos.

**Conclusão da Análise:**
*   **Consistência:** Provavelmente consistente.
*   **Completeza Suficiente:**
    *   `isEmptyAVL` é suficientemente completa.
    *   `memberAVL` é parcialmente coberta em relação a `insertAVL` e `deleteAVL`.
    *   `heightAVL`, `isValidAVL`, `isBST` não são definidas construtivamente para todas as formas de árvore e, portanto, não são suficientemente completas como operações observáveis definidas axiomaticamente; elas atuam mais como restrições de propriedade.

Uma especificação algébrica construtiva completa para árvores AVL é muito complexa e geralmente não é perseguida. Em vez disso, as propriedades de BST e balanceamento AVL são tratadas como invariantes que devem ser mantidos pela implementação das operações de inserção e remoção.

**Exercício:**

Se quiséssemos adicionar uma operação `sizeAVL: AVLTree[T] --> Natural` à especificação `AVLTree[T]` (que retorna o número de elementos/nós na árvore), como seriam os axiomas para `sizeAVL` em termos dos construtores `emptyAVL` e `insertAVL`? (Considere que `insertAVL` não adiciona duplicatas, ou que o tamanho conta nós distintos).

**Resolução:**

Adicionamos `sizeAVL` à especificação `AVLTree[T]`.

**SPEC ADT** AVLTreeWithSize[ParamT: OrderedElementType]
**imports:**
*   ParamT
*   Boolean
*   Natural
*   AVLTree[ParamT]

**sorts:**
*   AVLTree

**operations:**
*   sizeAVL: AVLTree --> Natural

**axioms:**
for all t: AVLTree, elem: T

*   (AVL_SIZE1): sizeAVL(emptyAVL) = zero
*   (AVL_SIZE2): memberAVL(elem, t) = true => sizeAVL(insertAVL(elem, t)) = sizeAVL(t)
*   (AVL_SIZE3): memberAVL(elem, t) = false => sizeAVL(insertAVL(elem, t)) = succ(sizeAVL(t))

**END SPEC**

A especificação `AVLTreeWithSize` enriquece `AVLTree[T]`. A nova operação `sizeAVL` recebe uma `AVLTree` e retorna um `Natural`.
O axioma (AVL_SIZE1) define que o tamanho de uma `emptyAVL` é `zero`.
O axioma (AVL_SIZE2) estabelece que se um elemento `elem` já é membro da árvore `t`, então inserir `elem` em `t` (assumindo que `insertAVL` não adiciona duplicatas ou substitui o existente sem alterar a contagem de nós distintos) não altera o tamanho da árvore.
O axioma (AVL_SIZE3) estabelece que se um elemento `elem` não é membro da árvore `t`, então inserir `elem` em `t` aumenta o tamanho da árvore em uma unidade (`succ(sizeAVL(t))`).
Estes axiomas definem `sizeAVL` recursivamente com base no comportamento de `insertAVL` e `memberAVL`, refletindo a política de não duplicatas para a contagem de tamanho.

# 15.3 Projeto e Implementação em Python com Tipagem Estática (MyPy)

A implementação de uma Árvore AVL em Python, `PyAVLTree[T]`, requer a definição de uma estrutura de nó que armazene o valor, as referências às subárvores esquerda e direita, e a altura do nó (ou fator de balanceamento). As operações de inserção e remoção devem incluir a lógica de rebalanceamento através de rotações.

**Design da Classe `PyAVLNode[T]` e `PyAVLTree[T]`:**

```python
# py_avl_tree.py
from __future__ import annotations # For type hints
from typing import TypeVar, Generic, Optional, Callable, List as PyListInternal, Any

T = TypeVar('T') # Generic type parameter for elements, must be orderable

class PyAVLNode(Generic[T]):
    # Node for the AVL Tree
    value: T
    left: Optional[PyAVLNode[T]]
    right: Optional[PyAVLNode[T]]
    height: int

    def __init__(self, value: T):
        # Constructor for an AVL Node
        self.value = value
        self.left = None
        self.right = None
        self.height = 1 # Height of a new leaf node is 1

class PyAVLTree(Generic[T]):
    _root: Optional[PyAVLNode[T]]
    # Comparison logic for elements of type T
    # will rely on Python's built-in comparison operators (<, >, ==)
    # TypeVar T can be bound to a protocol requiring these if needed for more safety.
    # e.g., T = TypeVar('T', bound=SupportsComparison)

    def __init__(self) -> None:
        # Constructor for PyAVLTree, creates an empty tree.
        self._root = None

    @staticmethod
    def empty_avl() -> PyAVLTree[Any]:
        # Corresponds to 'emptyAVL: --> AVLTree[T]'
        return PyAVLTree()

    def is_empty(self) -> bool:
        # Corresponds to 'isEmptyAVL: AVLTree[T] --> Boolean'
        return self._root is None

    def _get_height(self, node: Optional[PyAVLNode[T]]) -> int:
        # Helper to get height of a node (0 if None)
        if node is None:
            return 0
        return node.height

    def _get_balance_factor(self, node: Optional[PyAVLNode[T]]) -> int:
        # Helper to get balance factor of a node
        if node is None:
            return 0
        return self._get_height(node.left) - self._get_height(node.right)

    def _update_height(self, node: PyAVLNode[T]) -> None:
        # Helper to update height of a node based on children's heights
        node.height = 1 + max(self._get_height(node.left), self._get_height(node.right))

    def _rotate_right(self, y: PyAVLNode[T]) -> PyAVLNode[T]:
        # Performs a right rotation around node y
        # y is the root of the subtree to be rotated, which is unbalanced.
        # x becomes the new root.
        x = y.left
        assert x is not None # y must have a left child for right rotation
        t2 = x.right # t2 is the right child of x (middle subtree)

        # Perform rotation
        x.right = y
        y.left = t2

        # Update heights (order is important: children first)
        self._update_height(y) 
        self._update_height(x) 
        return x # New root of this subtree

    def _rotate_left(self, x: PyAVLNode[T]) -> PyAVLNode[T]:
        # Performs a left rotation around node x
        # x is the root of the subtree to be rotated.
        # y becomes the new root.
        y = x.right
        assert y is not None # x must have a right child for left rotation
        t2 = y.left # t2 is the left child of y (middle subtree)

        # Perform rotation
        y.left = x
        x.right = t2

        # Update heights
        self._update_height(x) 
        self._update_height(y) 
        return y # New root of this subtree

    def _insert_node(self, node: Optional[PyAVLNode[T]], value: T) -> PyAVLNode[T]:
        # Recursive helper to insert a value into the subtree rooted at 'node'
        # Returns the new root of this (potentially rebalanced) subtree.
        if node is None:
            return PyAVLNode(value) # Base case: insert new node

        if value < node.value: 
            node.left = self._insert_node(node.left, value)
        elif value > node.value: 
            node.right = self._insert_node(node.right, value)
        else:
            return node # Value already exists, tree structure unchanged (no duplicates)

        # Update height of the current node
        self._update_height(node)

        # Get the balance factor to check if this node became unbalanced
        balance = self._get_balance_factor(node)

        # Rebalance if necessary
        # Case 1: Left Left (balance > 1, new value inserted in left child's left subtree)
        if balance > 1 and node.left is not None and value < node.left.value:
            return self._rotate_right(node)
        
        # Case 2: Right Right (balance < -1, new value inserted in right child's right subtree)
        if balance < -1 and node.right is not None and value > node.right.value:
            return self._rotate_left(node)
        
        # Case 3: Left Right (balance > 1, new value inserted in left child's right subtree)
        if balance > 1 and node.left is not None and value > node.left.value:
            node.left = self._rotate_left(node.left) # Perform left rotation on left child
            return self._rotate_right(node)          # Then right rotation on current node
        
        # Case 4: Right Left (balance < -1, new value inserted in right child's left subtree)
        if balance < -1 and node.right is not None and value < node.right.value:
            node.right = self._rotate_right(node.right) # Perform right rotation on right child
            return self._rotate_left(node)             # Then left rotation on current node
        
        return node # Node is balanced, return it

    def insert(self, value: T) -> None: 
        # Public method to insert a value. Modifies the tree in-place.
        self._root = self._insert_node(self._root, value)

    def _member_node(self, node: Optional[PyAVLNode[T]], value: T) -> bool:
        # Recursive helper for member check
        if node is None:
            return False
        if value == node.value: 
            return True
        elif value < node.value:
            return self._member_node(node.left, value)
        else:
            return self._member_node(node.right, value)

    def member(self, value: T) -> bool:
        # Checks if a value is in the AVL tree.
        return self._member_node(self._root, value)
        
    # Deletion (deleteAVL) is omitted for brevity due to its complexity with rebalancing.

    def height(self) -> int:
        # Returns the height of the AVL tree.
        return self._get_height(self._root)

    def _is_avl_node_recursive(self, node: Optional[PyAVLNode[T]]) -> bool:
        # Helper to check AVL property recursively for a subtree
        if node is None:
            return True # An empty subtree is a valid AVL subtree
        
        balance = self._get_balance_factor(node)
        if abs(balance) > 1:
            return False # Balance factor out of {-1, 0, 1}
            
        # Check BST property (simplified for this check, assumes data integrity)
        # A full BST check would pass min/max bounds.
        if node.left is not None and node.left.value >= node.value: return False
        if node.right is not None and node.right.value <= node.value: return False

        return self._is_avl_node_recursive(node.left) and self._is_avl_node_recursive(node.right)

    def is_valid_avl(self) -> bool:
        # Public method to check if the entire tree is a valid AVL tree.
        # Also implicitly checks BST property if _is_avl_node_recursive includes it.
        return self._is_avl_node_recursive(self._root)

    def _in_order_traverse_node(self, node: Optional[PyAVLNode[T]], result_list: PyListInternal[T]) -> None:
        # Helper for in-order traversal
        if node is not None:
            self._in_order_traverse_node(node.left, result_list)
            result_list.append(node.value)
            self._in_order_traverse_node(node.right, result_list)

    def get_in_order_elements(self) -> PyListInternal[T]:
        # Returns a Python list of elements in-order.
        elements: PyListInternal[T] = []
        self._in_order_traverse_node(self._root, elements)
        return elements

    def _find_min_node(self, node: Optional[PyAVLNode[T]]) -> Optional[PyAVLNode[T]]:
        # Helper to find the node with the minimum value in a subtree
        if node is None or node.left is None:
            return node 
        return self._find_min_node(node.left) 

    def find_min(self) -> T:
        # Returns the minimum element in the AVL tree.
        # Raises ValueError if the tree is empty.
        if self.is_empty():
            raise ValueError("Cannot find minimum in an empty AVL tree.")
        min_node = self._find_min_node(self._root)
        if min_node is None: 
            raise RuntimeError("Internal error: Non-empty tree has no minimum node.") 
        return min_node.value
    
    def _find_max_node(self, node: Optional[PyAVLNode[T]]) -> Optional[PyAVLNode[T]]:
        # Helper to find the node with the maximum value in a subtree
        if node is None or node.right is None:
            return node
        return self._find_max_node(node.right)

    def find_max(self) -> T:
        # Returns the maximum element in the AVL tree.
        # Raises ValueError if the tree is empty.
        if self.is_empty():
            raise ValueError("Cannot find maximum in an empty AVL tree.")
        max_node = self._find_max_node(self._root)
        if max_node is None:
            raise RuntimeError("Internal error: Non-empty tree has no maximum node.")
        return max_node.value

    def __eq__(self, other: object) -> bool:
        # Simplified equality based on in-order traversal and height.
        if not isinstance(other, PyAVLTree):
            return NotImplemented
        return self.get_in_order_elements() == other.get_in_order_elements() and \
               self.height() == other.height()

    def __repr__(self) -> str:
        # Basic representation.
        return f"PyAVLTree(root_val={self._root.value if self._root else None}, height={self.height()})"

```
O código Python acima, no arquivo `py_avl_tree.py`, define classes `PyAVLNode[T]` e `PyAVLTree[T]`.
`PyAVLNode` armazena o valor, filhos e altura. `PyAVLTree` gerencia o nó raiz `_root` e as operações. Inclui métodos privados para obter altura, fator de balanceamento, atualizar altura e para as rotações (`_rotate_left`, `_rotate_right`). O método `insert` (e seu auxiliar `_insert_node`) implementa a inserção de BST seguida por rebalanceamento AVL. Esta implementação de `insert` é mutável. Os métodos `member`, `height`, `is_empty`, `is_valid_avl`, `find_min`, `find_max`, e `get_in_order_elements` são fornecidos. `deleteAVL` é omitida.

**Exercício:**

Implemente o método `find_min(self) -> T` na classe `PyAVLTree[T]`. Este método deve retornar o menor elemento na árvore AVL. Se a árvore estiver vazia, deve levantar `ValueError`.
*(Este exercício já foi resolvido e incorporado na classe `PyAVLTree[T]` acima.)*

**Resolução:**

O método `find_min` já foi adicionado à classe `PyAVLTree[T]` na listagem de código acima. Sua implementação, junto com o auxiliar `_find_min_node`, é:
```python
# py_avl_tree.py (dentro da classe PyAVLTree[T])
    # ...
    def _find_min_node(self, node: Optional[PyAVLNode[T]]) -> Optional[PyAVLNode[T]]:
        # Helper to find the node with the minimum value in a subtree
        if node is None or node.left is None:
            return node 
        return self._find_min_node(node.left) 

    def find_min(self) -> T:
        # Returns the minimum element in the AVL tree.
        # Raises ValueError if the tree is empty.
        if self.is_empty():
            raise ValueError("Cannot find minimum in an empty AVL tree.")
        min_node = self._find_min_node(self._root)
        if min_node is None: 
            # This case should not be reached if is_empty() is false.
            raise RuntimeError("Internal error: Non-empty tree has no minimum node.") 
        return min_node.value
    # ...
```
Este método `find_min` levanta `ValueError` para árvore vazia e usa `_find_min_node` para encontrar o nó com o menor valor navegando recursivamente para a esquerda.

# 15.4 Verificação de Propriedades Axiomáticas com Hypothesis

Verificar uma implementação de Árvore AVL usando os axiomas da Seção 15.2 (que são de alto nível e baseados em propriedades como `isBST` e `isValidAVL`) requer a implementação desses predicados em Python e, em seguida, o uso de Hypothesis para testar se as operações `insertAVL` e `deleteAVL` (se implementada) mantêm essas propriedades.

**Estratégias Hypothesis para `PyAVLTree[T]` e `T`:**
Usaremos `st.integers()` como estratégia para `T`.

```python
# test_py_avl_tree.py
import pytest
from hypothesis import given, strategies as st, settings, HealthCheck, assume
from typing import TypeVar, Any, List as PyListInternal, Optional

# Import the implementation
from py_avl_tree import PyAVLTree, PyAVLNode # Assuming py_avl_tree.py is accessible

# Strategy for generating element type 'T'. For this example, use integers.
st_element_type = st.integers(min_value=-100, max_value=100)

# Strategy to build a PyAVLTree[int] by a sequence of insertions
@st.composite
def st_py_avl_tree(draw):
    tree = PyAVLTree[int]()
    # Draw a list of elements to insert
    elements_to_insert = draw(st.lists(st_element_type, max_size=15)) # Control tree size
    for elem_val in elements_to_insert: # Renamed to avoid conflict
        tree.insert(elem_val) # Using the mutable insert
    return tree

# Strategy to generate a non-empty PyAVLTree[int]
@st.composite
def st_non_empty_py_avl_tree(draw):
    tree = PyAVLTree[int]()
    # Ensure at least one element is inserted
    elements_to_insert = draw(st.lists(st_element_type, min_size=1, max_size=15))
    for elem_val in elements_to_insert: # Renamed to avoid conflict
        tree.insert(elem_val)
    return tree
```
O código de teste `test_py_avl_tree.py` configura as estratégias. `st_element_type` gera inteiros. A estratégia `st_py_avl_tree` constrói uma `PyAVLTree[int]` realizando uma sequência de inserções. `st_non_empty_py_avl_tree` garante que a árvore gerada tenha pelo menos um elemento.

**Testes de Propriedade para os Axiomas/Propriedades de `AVLTree[T]`:**

```python
# test_py_avl_tree.py (continuação)

# --- Helper function to check BST property (for testing AVL5, AVL6) ---
def is_bst_ordered(node: Optional[PyAVLNode[int]], 
                   min_val: Optional[int] = None, 
                   max_val: Optional[int] = None) -> bool:
    # Checks if the subtree rooted at 'node' satisfies the BST ordering property.
    if node is None:
        return True
    if min_val is not None and node.value <= min_val: # Value must be > min_val
        return False
    if max_val is not None and node.value >= max_val: # Value must be < max_val
        return False
    return is_bst_ordered(node.left, min_val, node.value) and \
           is_bst_ordered(node.right, node.value, max_val)

# --- Testes para isEmptyAVL (AVL1, AVL2) ---
def test_AVL1_is_empty_emptyAVL():
    # (AVL1): isEmptyAVL(emptyAVL) = true
    assert PyAVLTree[Any].empty_avl().is_empty() == True

@given(t=st_py_avl_tree, elem=st_element_type)
def test_AVL2_is_empty_insertAVL(t: PyAVLTree[int], elem: int):
    # (AVL2): isEmptyAVL(insertAVL(elem, t)) = false
    # For mutable insert:
    new_tree = PyAVLTree[int]() # Test insertion into an empty tree
    new_tree.insert(elem) 
    assert new_tree.is_empty() == False
    
    t.insert(elem) # Test insertion into an existing tree (possibly empty from generator)
    assert t.is_empty() == False


# --- Testes para memberAVL (AVL3, AVL14) ---
@given(elem=st_element_type)
def test_AVL3_memberAVL_emptyAVL(elem: int):
    # (AVL3 / AVL4_BST_MEMBER_EMPTY): memberAVL(elem, emptyAVL) = false
    assert PyAVLTree[Any].empty_avl().member(elem) == False

@given(t=st_py_avl_tree, elem_to_insert=st_element_type)
def test_AVL14_memberAVL_insertAVL(t: PyAVLTree[int], elem_to_insert: int):
    # (AVL14): memberAVL(elem, insertAVL(elem,t)) = true
    t.insert(elem_to_insert) # Mutable insert
    assert t.member(elem_to_insert) == True


# --- Testes para propriedades de BST e Balanceamento AVL (AVL5, AVL7, AVL8, AVL10) ---
@given(t=st_py_avl_tree, elem=st_element_type)
@settings(suppress_health_check=[HealthCheck.too_slow], deadline=1000)
def test_AVL5_AVL8_properties_after_insert(t: PyAVLTree[int], elem: int):
    # (AVL5_BST_PROP): isBST(insertAVL(elem, t)) = true
    # (AVL8_BALANCE_PROP): isValidAVL(insertAVL(elem, t)) = true
    t.insert(elem) # Mutable insert
    assert is_bst_ordered(t._root) == True 
    assert t.is_valid_avl() == True       

def test_AVL7_AVL10_properties_of_empty():
    # (AVL7_BST_PROP): isBST(emptyAVL) = true
    # (AVL10_BALANCE_PROP): isValidAVL(emptyAVL) = true
    empty_t = PyAVLTree[Any].empty_avl()
    assert is_bst_ordered(empty_t._root) == True
    assert empty_t.is_valid_avl() == True

# --- Teste para heightAVL (AVL11) ---
def test_AVL11_height_emptyAVL():
    # (AVL11_HEIGHT_EMPTY): heightAVL(emptyAVL) = zero
    assert PyAVLTree[Any].empty_avl().height() == 0

@given(initial_elements=st.lists(st_element_type, max_size=15), elem_to_insert=st_element_type)
@settings(suppress_health_check=[HealthCheck.too_slow], deadline=1000)
def test_height_increase_constraint_after_insert(
    initial_elements: PyListInternal[int], 
    elem_to_insert: int
    ):
    # This test checks the property that height increases by at most 1.
    # (AVL12_HEIGHT_INSERT): heightAVL(insertAVL(elem, t)) <= add(heightAVL(t), one)
    
    # Create the initial tree t1
    t1 = PyAVLTree[int]()
    for item in initial_elements:
        t1.insert(item)
    height_before_insert = t1.height()

    # Create a copy t2 and insert the new element
    t2 = PyAVLTree[int]()
    for item in initial_elements: # Rebuild t2 to be identical to t1 initially
        t2.insert(item)
    t2.insert(elem_to_insert) 
    height_after_insert = t2.height()
    
    assert height_after_insert <= height_before_insert + 1
```
Os testes acima verificam alguns dos axiomas e propriedades de `PyAVLTree[int]`.
`is_bst_ordered` é uma função auxiliar para verificar a propriedade de ordenação BST.
`test_AVL1_is_empty_emptyAVL` e `test_AVL2_is_empty_insertAVL` testam `isEmptyAVL`. (Nota: `test_AVL2_is_empty_insertAVL` foi adaptado para a natureza mutável da `insert` implementada).
`test_AVL3_memberAVL_emptyAVL` e `test_AVL14_memberAVL_insertAVL` testam `memberAVL`.
`test_AVL5_AVL8_properties_after_insert` e `test_AVL7_AVL10_properties_of_empty` verificam que a árvore permanece uma BST e uma AVL válida.
`test_AVL11_height_emptyAVL` testa a altura da árvore vazia. O teste `test_height_increase_constraint_after_insert` verifica que a altura aumenta no máximo em 1 após uma inserção, conforme esperado para árvores AVL.

**Exercício:**

Considerando a operação `find_min: AVLTree[T] --> T` e sua implementação `PyAVLTree.find_min() -> T`. Uma propriedade esperada é que, se inserirmos um elemento `e` em uma árvore `avl_tree`, e `e` for menor que todos os elementos já presentes em `avl_tree` (ou se `avl_tree` estiver vazia), então `find_min` na nova árvore deve retornar `e`. Escreva um teste de propriedade Hypothesis para verificar este comportamento.

**Resolução:**

```python
# test_py_avl_tree.py (continuação)

@st.composite
def st_tree_elements_and_new_element_for_min_test(draw):
    # Strategy to generate a list of elements for the tree,
    # and a new element that is guaranteed to be smaller than any in the list (if list not empty).
    # Or, the list can be empty.
    
    # First, decide if the initial list will be empty or not
    is_initial_list_empty = draw(st.booleans())
    
    new_element_val = draw(st_element_type) # Draw the element to be inserted
    
    if is_initial_list_empty:
        tree_elements_list = []
    else:
        # Ensure all elements in the list are greater than new_element_val
        tree_elements_list = draw(st.lists(
            st.integers(min_value=new_element_val + 1, max_value=150), # Elements > new_element_val
            max_size=15 
        ))
        # To also cover cases where new_element_val is smaller than some but not all,
        # or equal, this strategy would need to be more complex or use 'assume'.
        # For this specific property (e is smallest), this targeted generation is fine.

    return tree_elements_list, new_element_val

@given(data=st_tree_elements_and_new_element_for_min_test())
@settings(suppress_health_check=[HealthCheck.too_slow], deadline=1000)
def test_find_min_after_inserting_smallest_element(data: tuple[PyListInternal[int], int]):
    """
    Tests that if 'e' is inserted and it's smaller than all existing elements
    (or the tree was empty), then 'e' becomes the new minimum.
    """
    tree_elements, e = data
    
    avl_tree = PyAVLTree[int]()
    for item in tree_elements:
        avl_tree.insert(item)
        
    # The strategy st_tree_elements_and_new_element_for_min_test ensures that
    # 'e' is smaller than any element in 'tree_elements', or 'tree_elements' is empty.
    
    avl_tree.insert(e) # Insert e
    
    assert not avl_tree.is_empty() # After inserting e, tree cannot be empty
    assert avl_tree.find_min() == e
```
O teste `test_find_min_after_inserting_smallest_element` usa uma estratégia composta `st_tree_elements_and_new_element_for_min_test`. Esta estratégia gera uma lista de elementos `tree_elements_list` (que pode ser vazia) e um `new_element_val`. Se `tree_elements_list` não for vazia, ela é construída de forma que todos os seus elementos sejam estritamente maiores que `new_element_val`.
O teste então constrói uma `PyAVLTree` com `tree_elements_list` e insere `new_element_val`. A asserção final verifica se `find_min()` na árvore resultante retorna `new_element_val`, o que deve ser verdade dado como os dados foram gerados.

# 15.5 Análise de Complexidade Algorítmica

A implementação `PyAVLTree[T]` envolve operações de BST mais a sobrecarga de verificar e manter o balanceamento através de rotações.
A altura de uma árvore AVL com $N$ nós é $O(\log N)$.

**Análise de Complexidade das Operações de `PyAVLTree[T]` (para a versão mutável de `insert`):**

1.  **`__init__`**: $O(1)$.
2.  **`empty_avl()`**: $O(1)$.
3.  **`is_empty()`**: $O(1)$.
4.  **`_get_height()`**: $O(1)$.
5.  **`_get_balance_factor()`**: $O(1)$.
6.  **`_update_height()`**: $O(1)$.
7.  **`_rotate_left()`, `_rotate_right()`**: $O(1)$.
8.  **`insert(self, value: T)`**: $O(\log N)$. (Descida $O(\log N)$, atualizações e rotações no caminho de volta $O(\log N)$).
9.  **`member(self, value: T)`**: $O(\log N)$.
10. **`height(self)`**: $O(1)$.
11. **`is_valid_avl(self)`**: $O(N)$ (visita todos os nós).
12. **`find_min(self)`, `find_max(self)`**: $O(\log N)$.
13. **`deleteAVL` (não implementada):** Esperado $O(\log N)$.
14. **`get_in_order_elements(self)`**: $O(N)$ (visita todos os nós).

As árvores AVL garantem desempenho logarítmico para as operações principais.

**Exercício:**

Se a operação `get_in_order_elements()` fosse uma operação fundamental do TAD `AVLTree[T]`, qual seria sua complexidade de tempo na implementação `PyAVLTree[T]` fornecida, e por quê?

**Resolução:**

A operação `get_in_order_elements()` na classe `PyAVLTree[T]` é implementada usando um percurso em ordem (`_in_order_traverse_node`).
O método `_in_order_traverse_node(node, result_list)` visita cada nó da árvore (ou subárvore) exatamente uma vez. Para cada nó visitado, realiza duas chamadas recursivas e uma operação `result_list.append(node.value)`. A operação `list.append()` em Python é $O(1)$ amortizado.
O método `get_in_order_elements()` inicializa uma lista vazia e chama `_in_order_traverse_node`. Se a árvore tem $N$ nós, o percurso visita cada um dos $N$ nós, e em cada visita, uma operação `append` ($O(1)$ amortizado) é realizada.
Portanto, o custo total do percurso e da construção da lista `elements` é $N \times O(1) = O(N)$.
A complexidade de tempo de `get_in_order_elements()` é **$O(N)$**, onde $N$ é o número de nós na árvore AVL.

# 15.6 Exercícios Teóricos e Práticos

Esta seção apresenta um exercício combinado para aplicar e estender os conceitos do TAD `AVLTree[T]`.

**Exercício:**

**Contexto:** Este exercício foca na verificação de uma propriedade fundamental das árvores AVL após múltiplas inserções.

**Parte a) Propriedade de Balanceamento Pós-Inserções**
Formule uma propriedade que afirme que, começando com uma árvore AVL vazia, após qualquer sequência de $k$ operações `insertAVL` válidas, a árvore resultante ainda satisfaz a propriedade de ser uma árvore AVL válida (ou seja, `isValidAVL` retorna `true`).

**Parte b) Esboço de Teste de Propriedade com Hypothesis**
Esboce uma função de teste `pytest` usando Hypothesis que verifique a propriedade formulada na Parte (a) para a sua implementação `PyAVLTree[T]`. Descreva a(s) estratégia(s) Hypothesis que você usaria para gerar a sequência de elementos a serem inseridos.

**Resolução:**

**Parte a) Propriedade de Balanceamento Pós-Inserções**

Seja $AVL_0 = \text{emptyAVL}$.
Para qualquer $k \ge 0$, e qualquer sequência de $k$ elementos $e_1, e_2, \ldots, e_k$ do tipo `T`:
Seja $AVL_i = \text{insertAVL}(e_i, AVL_{i-1})$ para $i = 1, \ldots, k$.
Então, a propriedade é: **`isValidAVL(AVL_k) = true`**.

Em linguagem natural: Para qualquer árvore AVL $t_{final}$ construída começando de uma `emptyAVL` e aplicando uma sequência finita de operações `insertAVL` com elementos de tipo `T`, a árvore $t_{final}$ deve ser uma árvore AVL válida. Adicionalmente, ela também deve ser uma Árvore Binária de Busca válida.

**Parte b) Esboço de Teste de Propriedade com Hypothesis**

```python
# test_py_avl_tree.py (continuação)
# ... (importações e estratégias st_element_type como antes) ...

# Strategy to generate a list of elements of type T for a sequence of insertions
st_list_of_elements_for_insertion = st.lists(st_element_type, min_size=0, max_size=20) 
# min_size=0 covers the case of the initially empty tree.
# max_size=20 to keep tests reasonably fast; can be adjusted.

@given(elements_to_insert=st_list_of_elements_for_insertion)
@settings(suppress_health_check=[HealthCheck.too_slow], deadline=2000) 
def test_avl_and_bst_properties_maintained_after_multiple_insertions(elements_to_insert: PyListInternal[int]):
    """
    Tests that the AVL and BST properties hold true after
    any sequence of valid insertions into an initially empty tree.
    """
    # 1. Start with an empty AVL tree
    avl_tree = PyAVLTree[int]() # Assuming T is int for this test instance
    
    # Optional: Check initial state
    assert avl_tree.is_empty()
    assert avl_tree.is_valid_avl() # An empty tree is a valid AVL tree
    assert is_bst_ordered(avl_tree._root) # An empty tree is a valid BST

    # 2. Perform a sequence of insertions
    current_elements_in_tree = set() # To track unique elements for BST check
    for i, elem in enumerate(elements_to_insert):
        avl_tree.insert(elem) 
        current_elements_in_tree.add(elem) # Add to set to get unique elements for BST verification
        
        # After each insertion, the tree should remain a valid AVL tree and BST
        bst_check_ok = is_bst_ordered(avl_tree._root)
        avl_check_ok = avl_tree.is_valid_avl()
        
        assert bst_check_ok, \
            f"BST property violated after inserting {elem} (step {i+1}) in sequence {elements_to_insert}"
        assert avl_check_ok, \
            f"AVL property violated after inserting {elem} (step {i+1}) in sequence {elements_to_insert}"

    # 3. Final check (also somewhat redundant if checks in loop are present)
    assert is_bst_ordered(avl_tree._root)
    assert avl_tree.is_valid_avl()
    
    # Additional check: all unique inserted elements should be members
    for unique_elem in current_elements_in_tree:
        assert avl_tree.member(unique_elem), \
            f"Element {unique_elem} missing after all insertions {elements_to_insert}"
    
    # Additional check: in-order traversal should yield sorted unique elements
    if not avl_tree.is_empty():
        in_order_list = avl_tree.get_in_order_elements()
        assert sorted(list(current_elements_in_tree)) == in_order_list, \
             f"In-order traversal not sorted or incorrect elements for {elements_to_insert}"

```

**Explicação do Teste de Propriedade:**

1.  **Estratégia `st_list_of_elements_for_insertion`:** Gera listas de elementos `T` (inteiros) para simular sequências de inserção.
2.  **Função de Teste:**
    *   Cria uma árvore AVL vazia.
    *   Verifica opcionalmente o estado inicial.
    *   Itera sobre os elementos gerados, inserindo cada um.
    *   **Verificação Intermediária (Mais Forte):** Após *cada* inserção, verifica se as propriedades `is_bst_ordered` e `avl_tree.is_valid_avl()` são mantidas. Isso garante que o invariante AVL é preservado em cada etapa, o que é crucial para a corretude da operação `insert`.
    *   **Verificações Adicionais (Opcionais, para robustez):** Após todas as inserções, verifica se todos os elementos únicos inseridos são de fato membros da árvore e se um percurso em ordem produz os elementos de forma ordenada e correta.

Este teste submete a implementação `PyAVLTree.insert()` a diversas sequências de inserção, verificando rigorosamente sua capacidade de manter os invariantes de BST e de balanceamento AVL.

Este capítulo introduziu o TAD `AVLTree[T]`, destacando sua importância para garantir desempenho logarítmico no pior caso através do auto-balanceamento. Apresentamos uma especificação algébrica focada nas propriedades observáveis e nos invariantes de BST e AVL. Discutimos o projeto e a implementação de uma classe `PyAVLTree[T]` em Python, detalhando as operações de inserção e as rotações necessárias para manter o balanceamento. A verificação da implementação através de testes de propriedade com Hypothesis foi esboçada, enfatizando a checagem dos invariantes AVL e BST após as operações. Finalmente, analisamos a complexidade $O(\log N)$ das operações principais. O próximo capítulo, Capítulo 16, explorará outra estrutura de dados hierárquica importante, o TAD `Heap`, que é fundamental para implementações eficientes de filas de prioridade.

---
## REFERÊNCIAS BIBLIOGRÁFICAS
ADELSON-VELSKY, G. M.; LANDIS, E. M. An algorithm for the organization of information. **Soviet Mathematics Doklady**, v. 3, p. 1259-1263, 1962.
*   *Resumo: Este é o artigo original onde Adelson-Velsky e Landis introduziram as árvores AVL. É a referência fundamental para a origem da estrutura de dados, seus algoritmos de balanceamento e a prova de sua propriedade de altura logarítmica.*

AHO, A. V.; HOPCROFT, J. E.; ULLMAN, J. D. **Data Structures and Algorithms**. Reading, MA: Addison-Wesley, 1983.
*   *Resumo: Um texto clássico que inclui uma discussão detalhada sobre árvores AVL, incluindo os algoritmos de inserção, remoção e as diferentes rotações necessárias para manter o balanceamento. Fornece uma base teórica sólida para a compreensão do TAD.*

CORMEN, T. H.; LEISERSON, C. E.; RIVEST, R. L.; STEIN, C. **Introduction to Algorithms**. 3. ed. Cambridge, MA: MIT Press, 2009.
*   *Resumo: Contém um capítulo dedicado a árvores balanceadas, incluindo árvores AVL. Apresenta análises de complexidade, pseudocódigo para as operações e discute as rotações de forma clara, sendo uma referência moderna essencial.*

KNUTH, D. E. **The Art of Computer Programming, Volume 3: Sorting and Searching**. 2. ed. Reading, MA: Addison-Wesley, 1998.
*   *Resumo: A obra monumental de Knuth cobre árvores balanceadas, incluindo árvores AVL, com uma análise matemática profunda de suas propriedades e desempenho. É uma referência definitiva para os aspectos teóricos e algorítmicos.*

SEDGEWICK, R.; WAYNE, K. **Algorithms**. 4. ed. Upper Saddle River, NJ: Addison-Wesley Professional, 2011.
*   *Resumo: Este livro texto moderno sobre algoritmos oferece uma excelente cobertura de árvores binárias de busca e árvores balanceadas, incluindo árvores AVL e Vermelho-Preto. As explicações são claras e acompanhadas de implementações (em Java), úteis para entender os aspectos práticos.*

WEISS, M. A. **Data Structures and Algorithm Analysis in C++**. 4. ed. Harlow, England: Pearson Education Limited, 2014.
*   *Resumo: Fornece uma discussão detalhada e análise de várias estruturas de dados, incluindo árvores AVL, com exemplos de implementação em C++. É útil para entender as complexidades de implementação e as considerações de desempenho em uma linguagem de sistemas.*
---
