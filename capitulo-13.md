---
# Capítulo 13
# Tipo Abstrato de Dados BinaryTree
---

Iniciando a Parte III deste livro, que se volta para as estruturas de dados não-lineares hierárquicas, este capítulo introduz o Tipo Abstrato de Dados (TAD) `BinaryTree[T]`, ou Árvore Binária Genérica. As árvores binárias são estruturas de dados fundamentais caracterizadas por nós que possuem, no máximo, dois filhos, tipicamente referidos como filho esquerdo e filho direito. Sua natureza recursiva e capacidade de representar hierarquias as tornam ubíquas em ciência da computação, servindo de base para estruturas mais especializadas como árvores de busca binária, heaps, árvores de expressão, e muitas outras. Este capítulo seguirá a metodologia estabelecida: começaremos com uma definição abstrata do TAD `BinaryTree[T]`, identificando suas operações construtoras essenciais, como a criação de uma árvore vazia (`emptyTree`) e a construção de um nó com um valor e duas subárvores (`makeTree`), e operações observadoras como `isEmpty`, `rootValue`, `leftSubtree`, e `rightSubtree`. Em seguida, desenvolveremos uma especificação algébrica formal para `BinaryTree[T]`, parametrizada pelo tipo de elemento `T`, detalhando seus sorts, operações e o conjunto de axiomas equacionais que capturam sua estrutura recursiva. Analisaremos a consistência e a completeza suficiente desta especificação. Posteriormente, projetaremos e implementaremos uma classe genérica `PyBinaryTree[T]` em Python, com anotações de tipo MyPy. A validação da corretude da implementação será conduzida através do Teste Baseado em Propriedades com Hypothesis. Concluiremos com uma análise da complexidade algorítmica das operações da classe `PyBinaryTree[T]` e um exercício prático para solidificar os conceitos.

# 13.1 Definição Abstrata e Relevância do Tipo de Dados `BinaryTree[T]`

O Tipo Abstrato de Dados `BinaryTree[T]`, ou Árvore Binária, é uma estrutura de dados hierárquica finita que consiste em um conjunto de nós. Cada nó em uma árvore binária armazena um valor de um tipo genérico `T` e possui referências (ou ponteiros) para, no máximo, dois outros nós, chamados de **filho esquerdo** e **filho direito**. Uma árvore binária pode ser vazia (não contendo nenhum nó) ou pode ser composta por um nó raiz e duas subárvores binárias disjuntas: a subárvore esquerda e a subárvore direita. Esta definição é inerentemente recursiva.

**Definição Abstrata:**
Formalmente, um `BinaryTree[T]` é:
*   Ou uma árvore vazia (representando a ausência de nós).
*   Ou um nó contendo um valor do tipo `T` (o valor da raiz), uma subárvore esquerda (que é ela mesma uma `BinaryTree[T]`) e uma subárvore direita (que também é uma `BinaryTree[T]`).

As operações essenciais que caracterizam um `BinaryTree[T]` incluem:
1.  **Criação/Construção:**
    *   `emptyTree: --> BinaryTree[T]`: Cria uma árvore binária vazia.
    *   `makeTree: T x BinaryTree[T] x BinaryTree[T] --> BinaryTree[T]`: Cria uma nova árvore binária (um nó) com um valor `T` dado, uma subárvore esquerda e uma subárvore direita.
2.  **Observação/Acesso:**
    *   `isEmpty: BinaryTree[T] --> Boolean`: Verifica se a árvore binária está vazia.
    *   `rootValue: BinaryTree[T] --> T`: Retorna o valor armazenado no nó raiz da árvore. (Parcial: não definida para uma árvore vazia).
    *   `leftSubtree: BinaryTree[T] --> BinaryTree[T]`: Retorna a subárvore esquerda da árvore. (Parcial: não definida para uma árvore vazia).
    *   `rightSubtree: BinaryTree[T] --> BinaryTree[T]`: Retorna a subárvore direita da árvore. (Parcial: não definida para uma árvore vazia).

Outras operações comuns, frequentemente derivadas, incluem travessias (pré-ordem, em-ordem, pós-ordem), cálculo de altura, contagem de nós, busca por um elemento, etc.

**Relevância do TAD `BinaryTree[T]`:**
As árvores binárias são uma das estruturas de dados não-lineares mais importantes e versáteis devido a uma série de razões:

1.  **Representação de Hierarquias:** Elas fornecem uma maneira natural de representar dados que possuem uma estrutura hierárquica ou relações de ancestralidade, como árvores genealógicas, estruturas de diretórios de arquivos, ou a sintaxe de expressões aritméticas (árvores de expressão).

2.  **Base para Estruturas de Busca Eficientes:** Árvores Binárias de Busca (BSTs), um tipo especializado de árvore binária onde os nós são ordenados, permitem operações eficientes de busca, inserção e remoção (tipicamente $O(\log N)$ em média para árvores balanceadas, onde $N$ é o número de nós).

3.  **Implementação de Heaps (Filas de Prioridade):** Heaps binários, que são árvores binárias com propriedades de ordem específicas (heap de máximo ou de mínimo), são usados para implementar filas de prioridade eficientes, com inserção e remoção do elemento de maior/menor prioridade em tempo $O(\log N)$.

4.  **Algoritmos de Compressão:** Árvores de Huffman, usadas em algoritmos de compressão de dados (como a codificação de Huffman), são árvores binárias.

5.  **Tomada de Decisão e Classificação:** Árvores de decisão em aprendizado de máquina e inteligência artificial são frequentemente estruturas de árvore binária (ou n-ária) onde cada nó interno representa um teste sobre um atributo e cada ramo uma saída do teste, com as folhas representando as classificações ou decisões.

6.  **Estrutura Recursiva Natural:** A definição recursiva das árvores binárias se presta bem a algoritmos recursivos elegantes e eficientes para sua manipulação e travessia.

7.  **Bloco de Construção para Estruturas Mais Complexas:** Muitas estruturas de dados avançadas são baseadas ou relacionadas a árvores binárias, como árvores AVL, árvores Rubro-Negras, B-trees (que generalizam árvores de busca para múltiplos filhos), etc.

A eficiência das operações em árvores binárias depende muito de sua forma, particularmente de sua altura. Árvores desbalanceadas (e.g., degeneradas em uma lista) podem levar a um desempenho de pior caso $O(N)$ para operações que dependem da altura. Por isso, grande parte da teoria de árvores de busca se concentra em algoritmos para manter as árvores balanceadas.

Neste capítulo, focaremos na especificação e implementação da estrutura `BinaryTree[T]` fundamental, sobre a qual estruturas mais especializadas serão construídas em capítulos subsequentes.

**Exercício:**

Considere um TAD `BinaryTree[T]` com as operações `emptyTree`, `makeTree(value: T, left: BinaryTree[T], right: BinaryTree[T])`, `rootValue(bt: BinaryTree[T])`, `leftSubtree(bt: BinaryTree[T])` e `rightSubtree(bt: BinaryTree[T])`. Seja `T` o tipo `Natural`.
Desenhe a árvore binária `bt1` resultante da seguinte sequência de construções:
1.  `t_empty = emptyTree`
2.  `t_leaf_5 = makeTree(natural_5, t_empty, t_empty)`
3.  `t_leaf_15 = makeTree(natural_15, t_empty, t_empty)`
4.  `t_node_10 = makeTree(natural_10, t_leaf_5, t_leaf_15)`
Qual seria o resultado de `rootValue(t_node_10)`, `rootValue(leftSubtree(t_node_10))`, e `rootValue(rightSubtree(t_node_10))`?

**Resolução:**

Vamos construir a árvore `bt1` (que é `t_node_10`) passo a passo:

1.  **`t_empty = emptyTree`**
    *   Representa uma árvore vazia.

2.  **`t_leaf_5 = makeTree(natural_5, t_empty, t_empty)`**
    *   Cria um nó folha com valor `natural_5`. Ambas as subárvores (esquerda e direita) são vazias.
    *   Estrutura de `t_leaf_5`:
        ```
          5
         / \
       emp emp
        ```
        (onde `emp` denota `emptyTree`)

3.  **`t_leaf_15 = makeTree(natural_15, t_empty, t_empty)`**
    *   Cria um nó folha com valor `natural_15`. Ambas as subárvores são vazias.
    *   Estrutura de `t_leaf_15`:
        ```
         15
         / \
       emp emp
        ```

4.  **`t_node_10 = makeTree(natural_10, t_leaf_5, t_leaf_15)`**
    *   Cria um novo nó com valor `natural_10`.
    *   Sua subárvore esquerda é `t_leaf_5`.
    *   Sua subárvore direita é `t_leaf_15`.
    *   Estrutura de `t_node_10` (que é `bt1`):
        ```
            10
           /  \
          5    15
         / \  / \
       emp emp emp emp
        ```

Agora, os resultados das operações:

*   **`rootValue(t_node_10)`:**
    A raiz de `t_node_10` contém o valor `natural_10`.
    Resultado: `natural_10`.

*   **`leftSubtree(t_node_10)`:**
    A subárvore esquerda de `t_node_10` é `t_leaf_5`.
    **`rootValue(leftSubtree(t_node_10))`** é `rootValue(t_leaf_5)`.
    A raiz de `t_leaf_5` contém o valor `natural_5`.
    Resultado: `natural_5`.

*   **`rightSubtree(t_node_10)`:**
    A subárvore direita de `t_node_10` é `t_leaf_15`.
    **`rootValue(rightSubtree(t_node_10))`** é `rootValue(t_leaf_15)`.
    A raiz de `t_leaf_15` contém o valor `natural_15`.
    Resultado: `natural_15`.

# 13.2 Especificação Algébrica Formal do TAD `BinaryTree[T]`

A especificação algébrica para o TAD `BinaryTree[T]` (Árvore Binária Genérica) definirá formalmente sua estrutura e operações básicas. Ela será parametrizada pelo tipo `T` dos valores armazenados nos nós.

**SPEC ADT** ElementTypeForTree
**sorts:**
*   T
**END SPEC**

**SPEC ADT** BinaryTree[ParamT: ElementTypeForTree]
**imports:**
*   ParamT
*   Boolean

**sorts:**
*   BinaryTree

**operations:**
*   emptyTree: --> BinaryTree
*   makeTree: T x BinaryTree x BinaryTree --> BinaryTree
*   isEmpty: BinaryTree --> Boolean
*   rootValue: BinaryTree --> T
*   leftSubtree: BinaryTree --> BinaryTree
*   rightSubtree: BinaryTree --> BinaryTree

**axioms:**
for all bt, left, right: BinaryTree, val: T

*   (BT1): isEmpty(emptyTree) = true
*   (BT2): isEmpty(makeTree(val, left, right)) = false
*   (BT3): rootValue(makeTree(val, left, right)) = val
*   (BT4): leftSubtree(makeTree(val, left, right)) = left
*   (BT5): rightSubtree(makeTree(val, left, right)) = right

**END SPEC**

A especificação `BinaryTree[ParamT: ElementTypeForTree]` define um TAD de Árvore Binária genérica. `ElementTypeForTree` é uma especificação de parâmetro formal que apenas introduz um sort `T` para os valores nos nós. A especificação `BinaryTree` importa `ParamT` (fornecendo o sort `T`) e `Boolean` (para o resultado de `isEmpty`). O sort principal definido é `BinaryTree`.
As **operações** são:
*   `emptyTree`: Uma constante que representa a árvore vazia. É um construtor de base.
*   `makeTree`: Recebe um valor `T` (para a raiz), uma `BinaryTree` (para a subárvore esquerda) e outra `BinaryTree` (para a subárvore direita), e retorna uma nova `BinaryTree` (o nó raiz com suas subárvores). É o construtor recursivo.
*   `isEmpty`: Recebe uma `BinaryTree` e retorna `true` se a árvore estiver vazia, `false` caso contrário. É um observador.
*   `rootValue`: Recebe uma `BinaryTree` e retorna o valor `T` armazenado em sua raiz. É um observador parcial, não definido para `emptyTree`.
*   `leftSubtree`: Recebe uma `BinaryTree` e retorna sua subárvore esquerda. É um observador parcial.
*   `rightSubtree`: Recebe uma `BinaryTree` e retorna sua subárvore direita. É um observador parcial.

Os **axiomas** definem o comportamento dessas operações em termos dos construtores:
*   (BT1) e (BT2) definem `isEmpty`: uma `emptyTree` é vazia; uma árvore construída com `makeTree` não é vazia.
*   (BT3) define `rootValue`: o valor da raiz de uma árvore construída com `makeTree(val, left, right)` é `val`.
*   (BT4) define `leftSubtree`: a subárvore esquerda de `makeTree(val, left, right)` é `left`.
*   (BT5) define `rightSubtree`: a subárvore direita de `makeTree(val, left, right)` é `right`.
As variáveis `bt`, `left`, `right` (do sort `BinaryTree`) e `val` (do sort `T`) são universalmente quantificadas.
Os axiomas (BT3), (BT4), (BT5) definem `rootValue`, `leftSubtree`, e `rightSubtree` apenas para árvores não vazias (aquelas formadas por `makeTree`). O comportamento dessas operações em `emptyTree` não é definido por estes axiomas, tornando-as operações parciais. Uma especificação mais completa para um ambiente de produção precisaria de tratamento de erro explícito ou pré-condições formais.

## 13.2.1 Análise de Corretude e Completeza da Especificação `BinaryTree[T]`

Analisaremos a especificação `BinaryTree[T]` quanto à consistência e completeza suficiente.

**Consistência:**
A especificação é construída sobre `Boolean` e um parâmetro `T` sem axiomas próprios.
1.  **Proteção dos Tipos Importados:** As operações de `BinaryTree[T]` retornam `Boolean`, `BinaryTree` ou `T`. Os axiomas não parecem introduzir novas igualdades ou contradições nos sorts `Boolean` ou `T`.
2.  **Consistência Interna de `BinaryTree[T]`:**
    *   Construtores: `emptyTree` e `makeTree(val, left, right)`.
    *   Axiomas (BT1) e (BT2), junto com `true != false` de `Boolean`, garantem que `emptyTree` é diferente de qualquer árvore construída com `makeTree`.
    *   A distinção entre árvores não vazias diferentes (e.g., `makeTree(v1, l1, r1)` vs `makeTree(v2, l2, r2)`) é garantida se seus componentes (`v1` vs `v2`, ou `l1` vs `l2`, ou `r1` vs `r2`) forem distinguíveis e as operações `rootValue`, `leftSubtree`, `rightSubtree` puderem revelar essas diferenças. Os axiomas (BT3-BT5) atuam como seletores para os componentes de um `makeTree`.
3.  **Modelo Inicial:** O modelo inicial para `BinaryTree[T]` é o conjunto de todas as árvores binárias finitas com nós rotulados por elementos de `T`, onde `emptyTree` é a árvore vazia e `makeTree` constrói uma árvore a partir de um rótulo e duas subárvores. Este modelo é consistente.

A especificação `BinaryTree[T]` parece **consistente**.

**Completeza Suficiente:**
Analisamos as operações não-construtoras (`isEmpty`, `rootValue`, `leftSubtree`, `rightSubtree`) aplicadas a termos formados pelos construtores `emptyTree` e `makeTree(val, left, right)`.

1.  **`isEmpty(bt: BinaryTree)`:**
    *   Caso $bt = \text{emptyTree}$: `isEmpty(emptyTree)` $\rightarrow_{BT1}$ `true`. (Resultado `Boolean` construtor).
    *   Caso $bt = \text{makeTree(v, l, r)}$: `isEmpty(makeTree(v, l, r))` $\rightarrow_{BT2}$ `false`. (Resultado `Boolean` construtor).
    `isEmpty` é suficientemente completa.

2.  **`rootValue(bt: BinaryTree)`:**
    *   Caso $bt = \text{emptyTree}$: `rootValue(emptyTree)`. Nenhum axioma (BT1-BT5) tem `rootValue(emptyTree)` no lado esquerdo. A operação é indefinida para este caso.
    *   Caso $bt = \text{makeTree(v, l, r)}$: `rootValue(makeTree(v, l, r))` $\rightarrow_{BT3}$ `v`. O resultado `v` é do tipo `T` (o sort de resultado esperado).
    `rootValue` é suficientemente completa para árvores não-vazias. Parcial para `emptyTree`.

3.  **`leftSubtree(bt: BinaryTree)`:**
    *   Caso $bt = \text{emptyTree}$: `leftSubtree(emptyTree)`. Indefinido.
    *   Caso $bt = \text{makeTree(v, l, r)}$: `leftSubtree(makeTree(v, l, r))` $\rightarrow_{BT4}$ `l`. Se `l` é um termo `BinaryTree` construtor, o resultado está na forma correta. Por indução (ou porque `l` é diretamente um dos argumentos construtores), sim.
    `leftSubtree` é suficientemente completa para árvores não-vazias. Parcial para `emptyTree`.

4.  **`rightSubtree(bt: BinaryTree)`:**
    *   Caso $bt = \text{emptyTree}$: `rightSubtree(emptyTree)`. Indefinido.
    *   Caso $bt = \text{makeTree(v, l, r)}$: `rightSubtree(makeTree(v, l, r))` $\rightarrow_{BT5}$ `r`. Similar a `leftSubtree`.
    `rightSubtree` é suficientemente completa para árvores não-vazias. Parcial para `emptyTree`.

**Conclusão da Análise:**
*   **Consistência:** A especificação `BinaryTree[T]` é consistente.
*   **Completeza Suficiente:** `isEmpty` é suficientemente completa. As operações `rootValue`, `leftSubtree`, e `rightSubtree` são suficientemente completas apenas para árvores não-vazias; seus comportamentos para `emptyTree` não são definidos pelos axiomas (BT1)-(BT5), tornando-as operações parciais. Para uma especificação totalmente completa para todos os termos, seria necessário adicionar tratamento para esses casos.

**Exercício:**

Adicione uma operação `countNodes: BinaryTree[T] --> Natural` à especificação `BinaryTree[T]`. Esta operação deve retornar o número total de nós na árvore (uma árvore vazia tem 0 nós). Forneça os axiomas para `countNodes`. Assuma que `Natural` com `zero` e `succ`, e a operação `add: Natural x Natural --> Natural` estão disponíveis.

**Resolução:**

Adicionamos `countNodes` à especificação `BinaryTree[T]`.

**SPEC ADT** BinaryTreeWithCount[ParamT: ElementTypeForTree]
**imports:**
*   ParamT
*   Boolean
*   Natural
*   BinaryTree[ParamT]

**sorts:**
*   BinaryTree

**operations:**
*   countNodes: BinaryTree --> Natural

**axioms:**
for all bt, left, right: BinaryTree, val: T

*   (BT_COUNT1): countNodes(emptyTree) = zero
*   (BT_COUNT2): countNodes(makeTree(val, left, right)) =
                   succ(add(countNodes(left), countNodes(right)))

**END SPEC**

A especificação `BinaryTreeWithCount` enriquece `BinaryTree[T]`. A nova operação `countNodes` recebe uma `BinaryTree` e retorna um `Natural`.
*   **(BT_COUNT1): `countNodes(emptyTree) = zero`**
    Este é o caso base: uma árvore vazia não contém nós, então a contagem é `zero`.
*   **(BT_COUNT2): `countNodes(makeTree(val, left, right)) = succ(add(countNodes(left), countNodes(right)))`**
    Este é o caso recursivo: o número de nós em uma árvore não-vazia construída por `makeTree` é 1 (para o nó raiz, representado pelo `succ`) mais a soma (`add`) do número de nós na subárvore esquerda (`countNodes(left)`) e o número de nós na subárvore direita (`countNodes(right)`).

Estes dois axiomas definem `countNodes` recursivamente sobre a estrutura da `BinaryTree`.

# 13.3 Projeto e Implementação em Python com Tipagem Estática (MyPy)

Com a especificação algébrica formal do TAD `BinaryTree[T]` definida, passamos à sua implementação em Python. Criaremos uma classe genérica `PyBinaryTree[T]` que realizará o comportamento recursivo e as operações especificadas. Dada a natureza recursiva das árvores binárias, uma representação onde cada nó é um objeto contendo um valor e referências às subárvores esquerda e direita (que são elas mesmas instâncias de `PyBinaryTree[T]` ou um valor especial para árvores vazias) é natural. Usaremos `None` para representar árvores vazias, ou uma instância singleton de uma classe `EmptyBinaryTree`. Para este exemplo, uma abordagem comum é uma classe que pode representar tanto um nó quanto o conceito de árvore, e que pode ser "vazia" em um certo sentido.

**Design da Classe `PyBinaryTree[T]`:**
Vamos optar por uma representação onde um objeto `PyBinaryTree` representa um nó, e pode ter subárvores que são também `PyBinaryTree` ou `None` (para subárvores vazias). Uma árvore vazia global pode ser um `None` especial ou uma instância de `PyBinaryTree` com atributos nulos. Para alinhar com `emptyTree` como um construtor, vamos fazer `PyBinaryTree` representar um nó, e `None` representará a árvore vazia.

```python
# py_binary_tree.py
from __future__ import annotations # For type hints like Optional[PyBinaryTree[T]]
from typing import TypeVar, Generic, Optional, Any

T = TypeVar('T') # Generic type parameter for elements

class PyBinaryTree(Generic[T]):
    _root_value: T
    _left_subtree: Optional[PyBinaryTree[T]]
    _right_subtree: Optional[PyBinaryTree[T]]
    _is_empty_flag: bool # To distinguish a constructed empty tree from None

    # Private constructor for internal use by static factory methods
    def __init__(self, value: T, 
                 left: Optional[PyBinaryTree[T]], 
                 right: Optional[PyBinaryTree[T]],
                 is_empty: bool = False) -> None:
        # This constructor is now more for make_tree logic; empty_tree handled by a specific state
        if not is_empty:
            self._root_value = value # Value is only relevant if not empty
        self._left_subtree = left
        self._right_subtree = right
        self._is_empty_flag = is_empty

    @staticmethod
    def empty_tree() -> PyBinaryTree[Any]: 
        # Corresponds to 'emptyTree: --> BinaryTree[T]'
        # Returns a special instance representing an empty tree.
        # We need a way to hold a value of type T for the _root_value if not empty,
        # but not if empty. This design is a bit tricky for is_empty_flag.
        # A common Pythonic way is to use None for empty trees.
        # Let's adjust the design: PyBinaryTree always represents a node.
        # None will represent the empty tree.
        # So, this static method is not needed if None is used.
        # For the ADT structure: use a sentinel object or a specific class.
        # Let's use a flag for now and refine if needed.
        # To make it a proper object, we pass placeholder values for value/left/right if empty.
        # This is not ideal. A better way is a separate EmptyTree class or using Optional[PyBinaryTree].
        # For this example, let's use the _is_empty_flag and make a sentinel.
        
        # Revision: It's cleaner if PyBinaryTree objects always represent non-empty nodes,
        # and the TYPE of a tree is Optional[PyBinaryTree[T]].
        # The ADT operations would then be static methods or free functions.
        # Let's proceed with PyBinaryTree representing a node, and None as empty.
        # The ADT operations will be static methods on a "companion" or utility class,
        # or we adapt the PyBinaryTree class to handle None gracefully in its instance methods
        # by making them static or by having a separate wrapper.
        # For now, let's make PyBinaryTree represent a node and adapt ADT operations.
        # The ADT's emptyTree will be represented by Python's None.
        # The ADT's makeTree will be the constructor of PyBinaryTree.
        pass # empty_tree() will be represented by None.

    # The ADT operations should ideally be defined outside if None represents empty.
    # Or, the class methods must handle self being None, which is not Pythonic for instance methods.
    # Alternative: create a wrapper or use static methods for ADT ops.
    # Let's make the class represent a NODE, and operations handle None.
    
    # Constructor for a non-empty tree (node)
    def __init__(self, value: T, 
                 left: Optional[PyBinaryTree[T]] = None, 
                 right: Optional[PyBinaryTree[T]] = None) -> None:
        self._root_value = value
        self._left_subtree = left
        self._right_subtree = right
        # _is_empty_flag is no longer needed if None represents emptyTree

    # ADT operations as static methods or part of a different structure.
    # For this exercise, let's define them as static methods of a conceptual
    # "BinaryTreeOps" or make them top-level functions that operate on Optional[PyBinaryTree[T]].
    # To keep it within the class structure for now, we'll use static methods where appropriate
    # and instance methods that assume self is not None (a non-empty tree).

    # Static methods for ADT operations that can take an "empty tree" (None)
    @staticmethod
    def is_node_empty(tree: Optional[PyBinaryTree[T]]) -> bool:
        # Corresponds to 'isEmpty: BinaryTree[T] --> Boolean'
        return tree is None

    @staticmethod
    def make_node(value: T, 
                  left: Optional[PyBinaryTree[T]], 
                  right: Optional[PyBinaryTree[T]]) -> PyBinaryTree[T]:
        # Corresponds to 'makeTree: T x BinaryTree[T] x BinaryTree[T] --> BinaryTree[T]'
        # This is effectively the constructor for a non-empty tree.
        return PyBinaryTree(value, left, right)

    # Instance methods, assume self is a non-empty tree (not None)
    def root_value(self) -> T:
        # Corresponds to 'rootValue: BinaryTree[T] --> T'
        # This method is called on an instance, so it's never emptyTree (None) here.
        return self._root_value

    def left_subtree(self) -> Optional[PyBinaryTree[T]]:
        # Corresponds to 'leftSubtree: BinaryTree[T] --> BinaryTree[T]'
        return self._left_subtree

    def right_subtree(self) -> Optional[PyBinaryTree[T]]:
        # Corresponds to 'rightSubtree: BinaryTree[T] --> BinaryTree[T]'
        return self._right_subtree
    
    def count_nodes(self) -> int: # Returns Natural (int >=0)
        # Corresponds to 'countNodes: BinaryTree[T] --> Natural' (from exercise 10.2.1)
        # This method is on an instance, so this node itself counts as 1.
        left_count = 0
        if self._left_subtree is not None:
            left_count = self._left_subtree.count_nodes()
        
        right_count = 0
        if self._right_subtree is not None:
            right_count = self._right_subtree.count_nodes()
            
        return 1 + left_count + right_count

    # --- For equality and representation ---
    def __eq__(self, other: object) -> bool:
        # Equality check.
        if not isinstance(other, PyBinaryTree):
            # This also means it won't be equal to None directly via this method
            return NotImplemented
        
        # Both are PyBinaryTree nodes, compare value and subtrees
        return (self._root_value == other._root_value and
                self._left_subtree == other._left_subtree and # Relies on __eq__ for Optional[PyBinaryTree]
                self._right_subtree == other._right_subtree)

    def __repr__(self) -> str:
        # String representation for a node.
        return f"PyBinaryTree({self._root_value!r}, {self._left_subtree!r}, {self._right_subtree!r})"

# Helper functions to more closely match ADT operation names and handle None for emptyTree
# The type `TreeT[T]` is an alias for `Optional[PyBinaryTree[T]]`
TreeT = Optional[PyBinaryTree[T]]

def adt_empty_tree() -> TreeT[Any]: # Using Any for T in truly empty tree
    return None

def adt_make_tree(value: T, left: TreeT[T], right: TreeT[T]) -> PyBinaryTree[T]: # Returns a node
    return PyBinaryTree(value, left, right)

def adt_is_empty(tree: TreeT[T]) -> bool:
    return tree is None

def adt_root_value(tree: PyBinaryTree[T]) -> T: # Assumes tree is not None (non-empty)
    # Precondition: tree is not None
    if tree is None: # Should be guarded by caller or spec pre-condition
        raise ValueError("Cannot get root value of an empty tree.")
    return tree.root_value()

def adt_left_subtree(tree: PyBinaryTree[T]) -> TreeT[T]: # Assumes tree is not None
    if tree is None:
        raise ValueError("Cannot get left subtree of an empty tree.")
    return tree.left_subtree()

def adt_right_subtree(tree: PyBinaryTree[T]) -> TreeT[T]: # Assumes tree is not None
    if tree is None:
        raise ValueError("Cannot get right subtree of an empty tree.")
    return tree.right_subtree()

def adt_count_nodes(tree: TreeT[T]) -> int: # Takes Optional tree
    if tree is None: # emptyTree case
        return 0
    else: # makeTree case
        return tree.count_nodes() # Calls instance method

```
O código Python acima, no arquivo `py_binary_tree.py`, define a classe genérica `PyBinaryTree[T]`.
A classe `PyBinaryTree` em si representa um *nó* não-vazio da árvore. A árvore binária vazia (`emptyTree` da especificação) é representada pelo valor `None` de Python. O tipo `TreeT[T]` é um alias para `Optional[PyBinaryTree[T]]` para representar uma árvore que pode ser vazia.
O construtor `__init__` da classe `PyBinaryTree` inicializa um nó com um valor (`_root_value`) e referências opcionais para subárvores esquerda (`_left_subtree`) e direita (`_right_subtree`), que também são do tipo `Optional[PyBinaryTree[T]]`.
As operações do TAD são implementadas como uma combinação de:
*   Funções auxiliares de alto nível (`adt_empty_tree`, `adt_make_tree`, `adt_is_empty`, `adt_root_value`, `adt_left_subtree`, `adt_right_subtree`, `adt_count_nodes`) que operam sobre `TreeT[T]` (i.e., `Optional[PyBinaryTree[T]]`) e lidam com o caso `None` (árvore vazia).
*   Métodos de instância na classe `PyBinaryTree` (e.g., `root_value`, `left_subtree`, `right_subtree`, `count_nodes`) que são chamados quando se tem certeza que a árvore não é vazia (i.e., é uma instância de `PyBinaryTree`).
Por exemplo, `adt_is_empty(tree)` retorna `True` se `tree` for `None`. `adt_make_tree` é o construtor `PyBinaryTree`. `adt_root_value` espera um `PyBinaryTree` (não `None`) e chama seu método de instância `root_value()`. `adt_count_nodes` lida com o caso `None` retornando 0, e caso contrário, delega para o método `count_nodes()` da instância.
Os métodos `__eq__` e `__repr__` são definidos para `PyBinaryTree` para facilitar testes e depuração.

**Exercício:**

Adicione um método de instância `is_leaf(self) -> bool` à classe `PyBinaryTree[T]`. Este método deve retornar `True` se o nó da árvore (representado por `self`) for uma folha (ou seja, suas subárvores esquerda e direita são ambas vazias/`None`), e `False` caso contrário. Lembre-se que este método será chamado em uma instância de `PyBinaryTree`, então `self` nunca será `None`.

**Resolução:**

Para adicionar o método `is_leaf` à classe `PyBinaryTree[T]`:

```python
# py_binary_tree.py (continuação da classe PyBinaryTree[T])

    # ... (outros métodos como definidos anteriormente) ...

    def is_leaf(self) -> bool:
        # A node is a leaf if both its left and right subtrees are empty (None).
        # This method is called on a PyBinaryTree instance, so 'self' is a non-empty node.
        return (self._left_subtree is None) and \
               (self._right_subtree is None)

    # ... (restante da classe) ...

# Função auxiliar no nível do ADT, se desejado (para operar sobre TreeT[T])
def adt_is_leaf(tree: TreeT[T]) -> bool:
    # An empty tree is not considered a leaf in most contexts.
    # A leaf is a non-empty node with no children.
    if tree is None: # emptyTree
        return False 
    # tree is a PyBinaryTree node, call its instance method
    return tree.is_leaf()

```
O método de instância `is_leaf` foi adicionado à classe `PyBinaryTree[T]`. Ele verifica se tanto `self._left_subtree` quanto `self._right_subtree` são `None`. Como este é um método de instância de `PyBinaryTree`, `self` representa um nó não-vazio; portanto, um nó é uma folha se não tiver filhos.
Uma função auxiliar `adt_is_leaf` também foi fornecida para demonstrar como ela operaria no tipo `TreeT[T]` (que pode ser `None`). Ela define que uma árvore vazia não é uma folha e, caso contrário, delega para o método de instância.

# 13.4 Verificação de Propriedades Axiomáticas com Hypothesis

Com a especificação algébrica de `BinaryTree[T]` e a implementação `PyBinaryTree[T]` (onde `None` representa `emptyTree` e instâncias de `PyBinaryTree` representam nós `makeTree`), podemos usar Hypothesis para validar os axiomas.

**Estratégias Hypothesis para `PyBinaryTree[T]` e `T`:**
Precisamos de uma estratégia para `T` e uma estratégia recursiva para `Optional[PyBinaryTree[T]]`.

```python
# test_py_binary_tree.py
import pytest
from hypothesis import given, strategies as st, assume, settings, HealthCheck
from typing import TypeVar, Optional, Any, List as PyListInternal

# Import the implementation (assuming None is emptyTree)
from py_binary_tree import PyBinaryTree, adt_empty_tree, adt_make_tree, \
                           adt_is_empty, adt_root_value, adt_left_subtree, \
                           adt_right_subtree, adt_count_nodes, adt_is_leaf 
                           # Using TreeT = Optional[PyBinaryTree[T]]

# Strategy for generating element type 'T'. For this example, use integers.
st_element_type = st.integers(min_value=0, max_value=100) # Using Naturals for T

# Strategy for Optional[PyBinaryTree[int]]
# This needs to be recursive.
# Base case: None (emptyTree)
# Recursive step: PyBinaryTree(value, left_child, right_child)
# where left_child and right_child are drawn from this same strategy.

# Define the strategy for PyBinaryTree[int] using st.recursive
# `children` in extend is a strategy that will produce TreeT[int]
st_py_binary_tree_optional = st.recursive(
    base=st.none(),  # Represents adt_empty_tree()
    extend=lambda children_strategy: st.builds(
        PyBinaryTree[int], # The constructor for a node
        value=st_element_type,
        left=children_strategy,
        right=children_strategy
    ),
    max_leaves=10 # Controls the size of the generated trees
)
# The functions adt_make_tree etc. would use PyBinaryTree constructor.
# st.builds(adt_make_tree, ...) would also work if adt_make_tree is the sole constructor.
```
O código de teste `test_py_binary_tree.py` configura as estratégias. `st_element_type` gera inteiros não-negativos (para `T` como `Natural`). A estratégia `st_py_binary_tree_optional` é crucial:
*   Ela usa `st.recursive` para definir uma estratégia para `Optional[PyBinaryTree[int]]` (que chamamos de `TreeT[int]` conceitualmente).
*   O `base` da recursão é `st.none()`, que gera `None` (representando `adt_empty_tree()`).
*   A função `extend` recebe `children_strategy` (que, quando chamada, gera um `Optional[PyBinaryTree[int]]`). Ela constrói um nó `PyBinaryTree[int]` (representando `adt_make_tree(val, left, right)`) usando `st.builds`. Os argumentos `value`, `left`, e `right` para o construtor `PyBinaryTree` são gerados por `st_element_type` e duas chamadas recursivas a `children_strategy`, respectivamente.
*   `max_leaves=10` limita o tamanho das árvores geradas para evitar recursão infinita e manter os testes práticos.

**Testes de Propriedade para os Axiomas de `BinaryTree[T]`:**
Axiomas (BT1)-(BT5) da Seção 13.2 e (BT_COUNT1)-(BT_COUNT2) do exercício da Seção 13.2.1.

```python
# test_py_binary_tree.py (continuação)

# --- Axioms for isEmpty ---
def test_BT1_isEmpty_emptyTree():
    # (BT1): isEmpty(emptyTree) = true
    tree = adt_empty_tree()
    assert adt_is_empty(tree) == True

@given(val=st_element_type, 
       left=st_py_binary_tree_optional, 
       right=st_py_binary_tree_optional)
def test_BT2_isEmpty_makeTree(val: int, 
                              left: Optional[PyBinaryTree[int]], 
                              right: Optional[PyBinaryTree[int]]):
    # (BT2): isEmpty(makeTree(val, left, right)) = false
    tree_node = adt_make_tree(val, left, right) # This creates a PyBinaryTree instance (non-empty)
    assert adt_is_empty(tree_node) == False 
    # Note: adt_is_empty checks if the tree (Optional[PyBinaryTree]) is None.
    # A PyBinaryTree instance is never None, so adt_is_empty(PyBinaryTree_instance) is False.

# --- Axioms for rootValue, leftSubtree, rightSubtree ---
@given(val=st_element_type, 
       left=st_py_binary_tree_optional, 
       right=st_py_binary_tree_optional)
def test_BT3_rootValue_makeTree(val: int, 
                                left: Optional[PyBinaryTree[int]], 
                                right: Optional[PyBinaryTree[int]]):
    # (BT3): rootValue(makeTree(val, left, right)) = val
    tree_node = adt_make_tree(val, left, right)
    assert adt_root_value(tree_node) == val

@given(val=st_element_type, 
       left=st_py_binary_tree_optional, 
       right=st_py_binary_tree_optional)
def test_BT4_leftSubtree_makeTree(val: int, 
                                   left: Optional[PyBinaryTree[int]], 
                                   right: Optional[PyBinaryTree[int]]):
    # (BT4): leftSubtree(makeTree(val, left, right)) = left
    tree_node = adt_make_tree(val, left, right)
    # Need to handle __eq__ for Optional[PyBinaryTree] correctly.
    # PyBinaryTree.__eq__ needs to handle other being None if self is not None.
    # Or compare if both are None, or both are PyBinaryTree and then compare their contents.
    # The default == on custom objects without __eq__ checks identity.
    # Our PyBinaryTree.__eq__ should handle this.
    assert adt_left_subtree(tree_node) == left

@given(val=st_element_type, 
       left=st_py_binary_tree_optional, 
       right=st_py_binary_tree_optional)
def test_BT5_rightSubtree_makeTree(val: int, 
                                    left: Optional[PyBinaryTree[int]], 
                                    right: Optional[PyBinaryTree[int]]):
    # (BT5): rightSubtree(makeTree(val, left, right)) = right
    tree_node = adt_make_tree(val, left, right)
    assert adt_right_subtree(tree_node) == right

# --- Axioms for countNodes (from exercise 13.2.1) ---
def test_BT_COUNT1_countNodes_emptyTree():
    # (BT_COUNT1): countNodes(emptyTree) = zero
    tree = adt_empty_tree()
    assert adt_count_nodes(tree) == 0

@given(val=st_element_type, 
       left=st_py_binary_tree_optional, 
       right=st_py_binary_tree_optional)
@settings(suppress_health_check=[HealthCheck.too_slow, HealthCheck.data_too_large], deadline=1000, max_examples=50)
def test_BT_COUNT2_countNodes_makeTree(val: int, 
                                       left: Optional[PyBinaryTree[int]], 
                                       right: Optional[PyBinaryTree[int]]):
    # (BT_COUNT2): countNodes(makeTree(val, left, right)) = 
    #                succ(add(countNodes(left), countNodes(right)))
    tree_node = adt_make_tree(val, left, right)
    
    # Python equivalent: 1 + adt_count_nodes(left) + adt_count_nodes(right)
    expected_count = 1 + adt_count_nodes(left) + adt_count_nodes(right)
    assert adt_count_nodes(tree_node) == expected_count
```
Os testes acima verificam os axiomas (BT1)-(BT5) e os de `countNodes`.
*   `test_BT1_isEmpty_emptyTree` e `test_BT_COUNT1_countNodes_emptyTree` testam os casos base com `adt_empty_tree()` (que é `None`).
*   Os testes decorados com `@given` usam a estratégia recursiva `st_py_binary_tree_optional` para gerar subárvores `left` e `right` (que podem ser `None` ou instâncias de `PyBinaryTree`), e `st_element_type` para `val`.
*   `test_BT2_isEmpty_makeTree` verifica que uma árvore construída com `adt_make_tree` não é vazia.
*   `test_BT3_rootValue_makeTree`, `test_BT4_leftSubtree_makeTree`, `test_BT5_rightSubtree_makeTree` verificam as propriedades dos seletores em relação ao construtor `adt_make_tree`. A correção destes testes depende de uma implementação robusta de `__eq__` na classe `PyBinaryTree` que possa comparar corretamente duas instâncias de `PyBinaryTree` ou uma instância com `None`.
*   `test_BT_COUNT2_countNodes_makeTree` verifica o axioma recursivo para `countNodes`. Devido à natureza recursiva e ao potencial de árvores grandes, `@settings` é usado para ajustar `max_examples`, `deadline` e suprimir avisos de saúde se a geração for lenta.

**Exercício:**

Considerando a operação `is_leaf(self) -> bool` adicionada à classe `PyBinaryTree[T]` (e a função auxiliar `adt_is_leaf(tree: Optional[PyBinaryTree[T]]) -> bool`) na resolução do exercício da Seção 13.3. Formule axiomas algébricos para `isLeaf: BinaryTree[T] -> Boolean` e escreva os testes de propriedade Hypothesis correspondentes.

**Resolução:**

**Axiomas Algébricos para `isLeaf`:**
Operação: `isLeaf: BinaryTree --> Boolean`

*   **(BT_LEAF1): `isLeaf(emptyTree) = false`**
    (Uma árvore vazia não é considerada uma folha).
*   **(BT_LEAF2): `isLeaf(makeTree(val, emptyTree, emptyTree)) = true`**
    (Uma árvore construída com um valor e duas subárvores vazias é uma folha).
*   **(BT_LEAF3): `isEmpty(left) = false => isLeaf(makeTree(val, left, right)) = false`**
    (Se a subárvore esquerda não é vazia, a árvore não é uma folha).
*   **(BT_LEAF4): `isEmpty(right) = false => isLeaf(makeTree(val, left, right)) = false`**
    (Se a subárvore direita não é vazia, a árvore não é uma folha).

Nota: (BT_LEAF3) e (BT_LEAF4) podem ser combinadas. Uma forma mais concisa, usando a definição de folha (um nó não vazio cujos filhos são ambos vazios):
*   (BT_LEAF2_alt): `isLeaf(makeTree(val, left, right)) = and(isEmpty(left), isEmpty(right))`
    Este único axioma para `makeTree`, junto com (BT_LEAF1), define `isLeaf`.

**Testes de Propriedade Hypothesis:**
Usaremos a definição (BT_LEAF1) e (BT_LEAF2_alt).

```python
# test_py_binary_tree.py (continuação)

# --- Axioms for isLeaf ---
def test_BT_LEAF1_isLeaf_emptyTree():
    # (BT_LEAF1): isLeaf(emptyTree) = false
    tree = adt_empty_tree()
    assert adt_is_leaf(tree) == False

@given(val=st_element_type, 
       left=st_py_binary_tree_optional, 
       right=st_py_binary_tree_optional)
@settings(suppress_health_check=[HealthCheck.too_slow], deadline=500, max_examples=50)
def test_BT_LEAF2_alt_isLeaf_makeTree(val: int, 
                                      left: Optional[PyBinaryTree[int]], 
                                      right: Optional[PyBinaryTree[int]]):
    # (BT_LEAF2_alt): isLeaf(makeTree(val, left, right)) = and(isEmpty(left), isEmpty(right))
    tree_node = adt_make_tree(val, left, right) # This is never None
    
    expected_is_leaf = adt_is_empty(left) and adt_is_empty(right)
    assert adt_is_leaf(tree_node) == expected_is_leaf
```
*   `test_BT_LEAF1_isLeaf_emptyTree`: Verifica que uma árvore vazia não é uma folha.
*   `test_BT_LEAF2_alt_isLeaf_makeTree`: Verifica que uma árvore não vazia (`tree_node`) é uma folha se e somente se ambas as suas subárvores (`left` e `right`) são vazias. A função `adt_is_leaf` (que chama o método `is_leaf` da classe `PyBinaryTree`) é usada para obter o resultado da implementação, e este é comparado com o resultado esperado calculado usando `adt_is_empty` nas subárvores.

# 13.5 Análise de Complexidade Algorítmica

A implementação `PyBinaryTree[T]` (onde `None` representa árvores vazias e instâncias de `PyBinaryTree` representam nós) tem operações cuja complexidade depende frequentemente da estrutura da árvore, como sua altura ou número de nós. Assumimos que operações sobre os elementos `T` são $O(1)$.

**Análise de Complexidade das Operações de `PyBinaryTree[T]` e Funções `adt_*`:**

1.  **`PyBinaryTree.__init__(value, left, right)` (Construtor de Nó):**
    *   Atribui referências: $O(1)$.
    *   **Complexidade:** $O(1)$.
    *   `adt_make_tree` chama este construtor, então também é $O(1)$.

2.  **`adt_empty_tree()`:**
    *   Retorna `None`: $O(1)$.
    *   **Complexidade:** $O(1)$.

3.  **`adt_is_empty(tree)`:**
    *   Verifica se `tree is None`: $O(1)$.
    *   **Complexidade:** $O(1)$.

4.  **`PyBinaryTree.root_value(self)` (e `adt_root_value` em nó não-vazio):**
    *   Acesso a atributo `self._root_value`: $O(1)$.
    *   **Complexidade:** $O(1)$. (Levantar exceção para `adt_root_value(None)` também é $O(1)$).

5.  **`PyBinaryTree.left_subtree(self)` (e `adt_left_subtree` em nó não-vazio):**
    *   Acesso a atributo `self._left_subtree`: $O(1)$.
    *   **Complexidade:** $O(1)$.

6.  **`PyBinaryTree.right_subtree(self)` (e `adt_right_subtree` em nó não-vazio):**
    *   Acesso a atributo `self._right_subtree`: $O(1)$.
    *   **Complexidade:** $O(1)$.

7.  **`PyBinaryTree.count_nodes(self)` (método de instância para nó não-vazio):**
    *   A operação visita cada nó na subárvore rooted em `self` exatamente uma vez. Se há $N_{self}$ nós nesta subárvore.
    *   **Complexidade:** $O(N_{self})$.
    *   **`adt_count_nodes(tree)`:** Se `tree` é `None`, $O(1)$. Se `tree` não é `None` e tem $N$ nós no total, chama `tree.count_nodes()`, então $O(N)$.

8.  **`PyBinaryTree.is_leaf(self)` (método de instância para nó não-vazio):**
    *   Verifica `self._left_subtree is None` e `self._right_subtree is None`: $O(1)$.
    *   **Complexidade:** $O(1)$.
    *   **`adt_is_leaf(tree)`:** Se `tree` é `None`, $O(1)$. Se não, chama `tree.is_leaf()`, então $O(1)$.

**Resumo da Complexidade:**
*   Criação de árvore vazia (`adt_empty_tree`): $O(1)$.
*   Criação de nó (`adt_make_tree` ou `PyBinaryTree.__init__`): $O(1)$.
*   Verificação de vacuidade (`adt_is_empty`): $O(1)$.
*   Acesso à raiz, subárvore esquerda, subárvore direita (em árvore não vazia): $O(1)$.
*   Contagem de nós (`adt_count_nodes`): $O(N)$, onde $N$ é o número de nós na árvore.
*   Verificação se é folha (`adt_is_leaf`): $O(1)$.

Estas são complexidades típicas para as operações básicas de uma árvore binária quando representada por uma estrutura de nós ligados por referência. Operações mais complexas como busca, inserção (em árvores de busca), ou travessias completas terão complexidades que dependem da altura da árvore (melhor caso $O(\log N)$, pior caso $O(N)$) ou do número total de nós ($O(N)$ para travessias).

**Exercício:**

Considere uma operação `height: BinaryTree[T] --> Natural` que calcula a altura de uma árvore binária (altura da árvore vazia é -1 ou 0 dependendo da convenção; altura de uma árvore com um nó é 0 ou 1. Vamos usar: altura de `emptyTree` = -1, altura de um nó folha = 0).
Escreva a assinatura e os axiomas para `height`. Depois, implemente `adt_height(tree: TreeT[T]) -> int` e o método de instância `height(self) -> int` na classe `PyBinaryTree` (retornando -1 para árvore vazia). Analise sua complexidade.

**Resolução:**

**Especificação Algébrica para `height`:**
Operação: `height: BinaryTree --> Natural`
(Para alinhar com a convenção de -1 para árvore vazia, o sort de resultado precisaria ser `Integer` ou um `ExtendedNatural` que inclua -1, ou redefinir `Natural` para incluir -1, o que é incomum. Vamos usar `Integer` para o resultado na especificação, e adaptar a implementação Python para retornar `int`.)

**SPEC ADT** BinaryTreeWithHeight[ParamT: ElementTypeForTree]
**imports:**
*   ParamT
*   Boolean
*   Integer (* Assumindo um TAD Integer com zeroInt, succInt, predInt, maxInt *)
*   BinaryTree[ParamT]

**sorts:**
*   BinaryTree (* herdado *)
*   Integer (* para o resultado da altura *)

**operations:**
*   height: BinaryTree --> Integer

**axioms:**
for all bt, left, right: BinaryTree, val: T

*   (BT_H1): height(emptyTree) = predInt(zeroInt) (* Representando -1 *)
*   (BT_H2): height(makeTree(val, left, right)) =
              succInt(maxInt(height(left), height(right)))
    (* Onde maxInt: Integer x Integer --> Integer retorna o maior de dois inteiros *)
    (* E succInt aqui significa +1 para o tipo Integer *)

**END SPEC**

A especificação `BinaryTreeWithHeight` enriquece `BinaryTree[T]`. A nova operação `height` retorna um `Integer`.
(BT_H1): A altura de uma `emptyTree` é -1 (representado por `predInt(zeroInt)`).
(BT_H2): A altura de uma árvore não vazia `makeTree(val, left, right)` é 1 (representado por `succInt`) mais a altura máxima entre suas subárvores esquerda e direita. A operação `maxInt` seria parte do TAD `Integer` ou definida separadamente.

**Implementação Python para `height`:**

```python
# py_binary_tree.py (continuação da classe PyBinaryTree[T] e funções adt_*)

# ... (dentro da classe PyBinaryTree[T]) ...
    def height(self) -> int:
        # Calculates the height of this node.
        # Height of a leaf node is 0.
        # Assumes self is not None.
        
        left_h = -1
        if self._left_subtree is not None:
            left_h = self._left_subtree.height() # Recursive call
            
        right_h = -1
        if self._right_subtree is not None:
            right_h = self._right_subtree.height() # Recursive call
            
        return 1 + max(left_h, right_h)

# ... (funções adt_* fora da classe) ...
def adt_height(tree: TreeT[T]) -> int: # TreeT = Optional[PyBinaryTree[T]]
    # Height of an empty tree is -1.
    # Height of a tree with a single node (leaf) is 0.
    if tree is None: # emptyTree case
        return -1
    else: # makeTree case, tree is a PyBinaryTree node
        return tree.height() # Calls instance method
```
O método de instância `height(self)` na classe `PyBinaryTree` calcula recursivamente a altura. Se uma subárvore é `None`, sua altura é considerada -1 para o cálculo `max`. O resultado é `1 + max(altura_esquerda, altura_direita)`.
A função `adt_height(tree)` lida com o caso de `tree` ser `None` (retornando -1) e, caso contrário, chama o método de instância.

**Análise de Complexidade para `height`:**
A operação `adt_height` (e o método `height` da instância) visita cada nó da árvore exatamente uma vez para calcular as alturas das subárvores.
Se $N$ é o número de nós na árvore.
**Complexidade:** $O(N)$. Cada nó é visitado, e as operações em cada nó (chamadas recursivas, `max`, adição) são $O(1)$.

Este capítulo introduziu o TAD `BinaryTree[T]`, sua definição abstrata, relevância, e uma especificação algébrica formal parametrizada. Analisamos a corretude e completeza desta especificação. Projetamos e implementamos uma classe Python genérica `PyBinaryTree[T]` (usando `None` para árvores vazias) com tipagem estática. Demonstramos como os axiomas podem ser traduzidos em testes de propriedade com Hypothesis, incluindo uma estratégia recursiva para gerar árvores. Finalmente, analisamos a complexidade algorítmica das operações. O próximo capítulo, Capítulo 14, focará no TAD `BinarySearchTree[T]` (Árvore de Busca Binária), uma especialização da árvore binária que impõe uma ordem nos seus elementos para permitir buscas eficientes.

---
## REFERÊNCIAS BIBLIOGRÁFICAS
AHO, A. V.; HOPCROFT, J. E.; ULLMAN, J. D. **Data Structures and Algorithms**. Reading, MA: Addison-Wesley, 1983.
*   *Resumo: Este texto clássico oferece uma discussão fundamental sobre árvores binárias, incluindo suas representações, travessias e a base para árvores de busca. Relevante para a definição abstrata e conceitual do TAD `BinaryTree[T]`.*

CORMEN, T. H.; LEISERSON, C. E.; RIVEST, R. L.; STEIN, C. **Introduction to Algorithms**. 3. ed. Cambridge, MA: MIT Press, 2009.
*   *Resumo: Contém capítulos detalhados sobre árvores binárias e árvores de busca binária, abordando suas propriedades, operações e análise de complexidade. Essencial para compreender a relevância e as características de desempenho do `BinaryTree[T]`.*

GOODRICH, M. T.; TAMASSIA, R.; GOLDWASSER, M. H. **Data Structures and Algorithms in Python**. Hoboken, NJ: Wiley, 2013.
*   *Resumo: Este livro apresenta implementações de árvores binárias em Python, incluindo a representação de nós e algoritmos de travessia. Fornece um contexto prático para a implementação `PyBinaryTree[T]` e o uso de genéricos em Python.*

KNUTH, D. E. **The Art of Computer Programming, Volume 1: Fundamental Algorithms**. 3. ed. Reading, MA: Addison-Wesley, 1997.
*   *Resumo: Uma obra monumental que, em sua seção sobre estruturas de informação, detalha árvores extensivamente, incluindo árvores binárias, suas propriedades matemáticas e várias representações. É uma referência profunda para os aspectos teóricos das árvores.*

OKASAKI, C. **Purely Functional Data Structures**. Cambridge: Cambridge University Press, 1998.
*   *Resumo: Explora a implementação de estruturas de dados, incluindo árvores, em um paradigma puramente funcional. Relevante para entender abordagens imutáveis para árvores e a análise de suas operações, o que se alinha com o estilo da implementação `PyBinaryTree[T]`.*

WIRSING, M. Algebraic Specification. In: VAN LEEUWEN, J. (Ed.). **Handbook of Theoretical Computer Science, Volume B: Formal Models and Semantics**. Amsterdam: Elsevier; Cambridge, MA: MIT Press, 1990. p. 675-788.
*   *Resumo: Este capítulo de handbook detalha a especificação algébrica de TADs, e árvores são frequentemente usadas como exemplos para ilustrar definições recursivas e axiomatização. Fornece o embasamento teórico para a Seção 13.2.*
---
