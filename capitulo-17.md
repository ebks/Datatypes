---
# Capítulo 17
# Tipo Abstrato de Dados BTree
---

Este capítulo marca a conclusão da Parte III, dedicada às estruturas de dados não-lineares hierárquicas, com um estudo aprofundado sobre o Tipo Abstrato de Dados (TAD) `BTree[T]`, ou Árvore-B. As Árvores-B, introduzidas por Bayer e McCreight, são estruturas de árvore autobalanceadas projetadas especificamente para otimizar operações em dispositivos de armazenamento secundário, como discos rígidos, onde o custo de acesso a um bloco de dados é significativamente mais alto do que o custo de processamento em memória principal. Sua principal característica é a capacidade de manter um grande fator de ramificação (ou ordem), o que resulta em árvores relativamente rasas, mesmo para um grande número de chaves, minimizando assim o número de acessos a disco necessários para localizar um dado. As Árvores-B e suas variantes (como B+-Trees e B*-Trees) são a espinha dorsal de muitos sistemas de gerenciamento de banco de dados (SGBDs) e sistemas de arquivos. Este capítulo seguirá a metodologia padrão: iniciaremos com uma definição abstrata do TAD `BTree[T]`, enfocando suas propriedades fundamentais (ordem, balanceamento, chaves e ponteiros para filhos) e as operações canônicas de busca, inserção e remoção, que envolvem processos mais complexos de divisão (split) e fusão (merge) de nós para manter os invariantes da árvore. Desenvolveremos uma especificação algébrica formal para uma versão simplificada de `BTree[T]`, parametrizada pelo tipo de chave `T` (que deve ser ordenável) e pelo grau mínimo `t` da árvore, e analisaremos sua consistência e completeza. Posteriormente, discutiremos os desafios e as abordagens para o projeto e a implementação de uma classe `PyBTree[T]` em Python, com tipagem MyPy. A validação da corretude desta implementação, dada a complexidade das operações de inserção e remoção, será abordada conceitualmente em termos de teste baseado em propriedades com Hypothesis. Finalmente, analisaremos a complexidade algorítmica das operações em Árvores-B, destacando sua eficiência logarítmica em relação ao número de acessos a disco, e concluiremos com um exercício prático.

# 17.1 Definição Abstrata e Relevância do Tipo de Dados `BTree[T]`

O Tipo Abstrato de Dados `BTree[T]`, ou Árvore-B de ordem $m$ (ou grau mínimo $t$), é uma estrutura de árvore de busca autobalanceada projetada para armazenar um grande volume de dados de forma ordenada, permitindo operações eficientes de busca, inserção, remoção e travessia sequencial, especialmente quando os dados residem em memória secundária (disco). O tipo `T` dos elementos (chaves) armazenados na Árvore-B deve ser totalmente ordenado.

**Definição Abstrata e Propriedades Fundamentais:**
Uma Árvore-B de grau mínimo $t \ge 2$ (algumas definições usam ordem $m=2t$) satisfaz as seguintes propriedades:

1.  **Estrutura dos Nós:**
    *   Cada nó interno (não folha e não raiz) contém $k$ chaves $key_1, key_2, \ldots, key_k$ e $k+1$ ponteiros para filhos $child_0, child_1, \ldots, child_k$.
    *   As chaves em cada nó são armazenadas em ordem crescente: $key_1 < key_2 < \ldots < key_k$.
    *   Dentro de um nó, para um nó interno com chaves $key_1, \ldots, key_k$ e filhos $child_0, \ldots, child_k$:
        *   Todas as chaves na subárvore $child_0$ são menores que $key_1$.
        *   Para $1 \le i < k$, todas as chaves na subárvore $child_i$ são maiores que $key_i$ e menores que $key_{i+1}$.
        *   Todas as chaves na subárvore $child_k$ são maiores que $key_k$.
    *   Nós folha não têm filhos e armazenam apenas chaves.

2.  **Número de Chaves por Nó:**
    *   Todo nó, exceto a raiz, deve ter pelo menos $t-1$ chaves. Portanto, cada nó interno, exceto a raiz, tem pelo menos $t$ filhos.
    *   Todo nó pode conter no máximo $2t-1$ chaves. Portanto, um nó interno pode ter no máximo $2t$ filhos.
    *   A raiz é uma exceção: pode ter de 1 a $2t-1$ chaves (se não for folha, deve ter pelo menos 2 filhos). Se a árvore estiver vazia, a raiz é nula. Se a raiz for uma folha, ela pode ter de 0 (para árvore vazia conceitual) a $2t-1$ chaves.

3.  **Balanceamento:**
    *   Todas as folhas aparecem no mesmo nível (profundidade).

O parâmetro $t$ (grau mínimo) é escolhido de acordo com o tamanho do bloco de disco.

**Operações Essenciais do TAD `BTree[T]`:**
(Assumindo `T` é um tipo ordenável com operações `eqT`, `ltT`)
1.  **Criação:**
    *   `emptyBTree(degree: Natural) --> BTree[T]`: Cria uma Árvore-B vazia com grau mínimo $t$.
2.  **Busca:**
    *   `search(key: T, tree: BTree[T]) --> Boolean`: Verifica a presença de uma `key`.
3.  **Inserção:**
    *   `insert(key: T, tree: BTree[T]) --> BTree[T]`: Insere a `key`, mantendo propriedades (envolve **split**).
4.  **Remoção:**
    *   `delete(key: T, tree: BTree[T]) --> BTree[T]`: Remove a `key`, mantendo propriedades (envolve **merge** ou **rotação**).
5.  **Observadores:**
    *   `isEmpty(tree: BTree[T]) --> Boolean`: Verifica se a árvore está vazia.

**Relevância do TAD `BTree[T]`:**
1.  **Sistemas de Gerenciamento de Banco de Dados (SGBDs):** Estrutura de indexação padrão.
2.  **Sistemas de Arquivos:** Gerenciamento de metadados e indexação de diretórios.
3.  **Eficiência Logarítmica para Disco:** $O(\log_t N)$ acessos a disco.
4.  **Auto-Balanceamento Robusto:** Mantém desempenho logarítmico.
5.  **Suporte a Grandes Volumes de Dados:** Ideal para dados que não cabem em memória principal.

**Exercício:**

Considere uma Árvore-B de grau mínimo $t=2$. Isso significa que cada nó (exceto a raiz) deve ter entre $t-1=1$ e $2t-1=3$ chaves, e entre $t=2$ e $2t=4$ filhos. Se uma Árvore-B com $t=2$ tem altura $h=1$ (raiz e uma camada de folhas), qual o número mínimo e máximo de chaves que ela pode conter no total? (Lembre-se: a raiz pode ter de 1 a $2t-1$ chaves).

**Resolução:**

Uma Árvore-B de grau mínimo $t=2$:
*   Nós (exceto raiz): $1$ a $3$ chaves. Filhos (se interno): $2$ a $4$.
*   Raiz: $1$ a $3$ chaves. Filhos (se não folha): $2$ a $4$.

Altura $h=1$: raiz e seus filhos são folhas.

**Número Mínimo de Chaves:**
*   Raiz: Mínimo $1$ chave.
*   Filhos da Raiz (se a raiz tem 1 chave, tem 2 filhos): Cada filho é uma folha e deve ter no mínimo $t-1=1$ chave.
*   Total Mínimo: $1 (\text{raiz}) + 2 \times 1 (\text{folhas}) = 3$ chaves.

**Número Máximo de Chaves:**
*   Raiz: Máximo $2t-1=3$ chaves.
*   Filhos da Raiz (se a raiz tem 3 chaves, tem 4 filhos): Cada filho é uma folha e pode ter no máximo $2t-1=3$ chaves.
*   Total Máximo: $3 (\text{raiz}) + 4 \times 3 (\text{folhas}) = 3 + 12 = 15$ chaves.

Portanto, para uma Árvore-B de grau mínimo $t=2$ e altura $h=1$:
*   Mínimo de chaves: 3
*   Máximo de chaves: 15

# 17.2 Especificação Algébrica Formal do TAD `BTree[T]`

A especificação algébrica completa de uma Árvore-B é complexa. Esboçaremos uma especificação focada na estrutura de um nó e operações de alto nível.

**SPEC ADT** KeyTypeRequirement
**sorts:**
*   T
**operations:**
*   eqT: T x T --> Boolean
*   ltT: T x T --> Boolean
**axioms:**
*   (KT1): eqT(k,k) = true
*   (KT2): eqT(k1,k2) = eqT(k2,k1)
*   (KT3): ltT(k1,k2) AND ltT(k2,k3) => ltT(k1,k3) = true
*   (KT4): not(eqT(k1,k2)) = true => or(ltT(k1,k2), ltT(k2,k1)) = true
for all k,k1,k2,k3: T
**END SPEC**

A especificação `KeyTypeRequirement` define que o tipo `T` (renomeado para `T` no corpo de `BTree`) deve ser um sort `T` com operações de igualdade `eqT` e "menor que" `ltT`, satisfazendo propriedades básicas de uma relação de ordem total.

**SPEC ADT** BTreeNode[ParamT: KeyTypeRequirement, ParamDegree: NaturalConstant]
**imports:**
*   ParamT
*   Natural
*   Boolean
*   Vector[T]
*   Vector[BTreeNode]

**sorts:**
*   BTreeNode

**operations:**
*   createLeafNode: Vector[T] --> BTreeNode
*   createInternalNode: Vector[T] x Vector[BTreeNode] --> BTreeNode
*   isLeaf: BTreeNode --> Boolean
*   getKeys: BTreeNode --> Vector[T]
*   getChildren: BTreeNode --> Vector[BTreeNode]
*   numKeysNode: BTreeNode --> Natural
*   getKeyNode: BTreeNode x Natural --> T
*   getChildNode: BTreeNode x Natural --> BTreeNode

**axioms:**
for all vt: Vector[T], vbtn: Vector[BTreeNode], btn: BTreeNode, idx: Natural
*   (BTN1): isLeaf(createLeafNode(vt)) = true
*   (BTN2): isLeaf(createInternalNode(vt, vbtn)) = false
*   (BTN3): getKeys(createLeafNode(vt)) = vt
*   (BTN4): getKeys(createInternalNode(vt, vbtn)) = vt
*   (BTN5): getChildren(createInternalNode(vt, vbtn)) = vbtn
*   (BTN6): numKeysNode(btn) = length(getKeys(btn))
*   (BTN7): getKeyNode(btn, idx) = get(getKeys(btn), idx)
*   (BTN8): isLeaf(btn) = false => getChildNode(btn, idx) = get(getChildren(btn), idx)
**END SPEC**

A especificação `BTreeNode` é parametrizada por `ParamT` (tipo da chave) e `ParamDegree` (grau mínimo $t$). Importa `Vector[T]` para chaves e `Vector[BTreeNode]` para filhos (introduzindo recursão de tipos na especificação). Define `BTreeNode` com construtores `createLeafNode` e `createInternalNode`. Observadores incluem `isLeaf`, `getKeys`, `getChildren` (parcial para nós internos), `numKeysNode`, `getKeyNode` e `getChildNode`. Axiomas (BTN1)-(BTN8) definem estas operações em termos dos construtores e operações de `Vector`.

**SPEC ADT** BTree[ParamT: KeyTypeRequirement, ParamDegree: NaturalConstant]
**imports:**
*   ParamT
*   Natural
*   Boolean
*   BTreeNode[ParamT, ParamDegree]

**sorts:**
*   BTree

**operations:**
*   emptyBTree: --> BTree
*   makeBTree: BTreeNode --> BTree
*   getRoot: BTree --> BTreeNode
*   searchKey: T x BTree --> Boolean
*   insertKey: T x BTree --> BTree
*   deleteKey: T x BTree --> BTree

**axioms:**
for all tKey: T, bt: BTree, btn: BTreeNode

*   (BT1): searchKey(tKey, emptyBTree) = false
*   (BT2): isEmpty(bt) = false => searchKey(tKey, bt) = searchKeyInNode(tKey, getRoot(bt))

*   (BT_IS_EMPTY1): isEmpty(emptyBTree) = true
*   (BT_IS_EMPTY2): isEmpty(makeBTree(btn)) = false 
*   (BT_GET_ROOT): isEmpty(bt) = false AND bt = makeBTree(btn) => getRoot(bt) = btn

**END SPEC**

A especificação `BTree` é parametrizada e importa `BTreeNode`. O sort principal é `BTree`. Operações incluem `emptyBTree`, `makeBTree` (cria árvore com nó raiz), `getRoot` (observador), e `searchKey`, `insertKey`, `deleteKey`. Axiomas (BT1)-(BT2) definem `searchKey`, delegando para `searchKeyInNode` (cuja especificação completa é omitida, mas envolveria recursão no `BTreeNode`). Axiomas (BT_IS_EMPTY1), (BT_IS_EMPTY2) e (BT_GET_ROOT) definem `isEmpty` e `getRoot`. Os axiomas para `insertKey` e `deleteKey` (que são o cerne da complexidade da B-Tree) são omitidos por brevidade, mas envolveriam a manutenção dos invariantes da B-Tree através de splits, merges e rotações.

## 10.2.1 Análise de Corretude e Completeza da Especificação `BTree[T]`

**Consistência:**
A parte especificada de `BTreeNode` e a busca em `BTree` parecem consistentes, assumindo a consistência dos TADs importados (`Vector`, `Natural`, `Boolean`). O modelo pretendido das Árvores-B é bem definido. A principal fonte de potencial inconsistência residiria nos axiomas complexos e omitidos de `insertKey` e `deleteKey`.

**Completeza Suficiente:**
*   **Para `BTreeNode`:**
    *   `isLeaf`, `getKeys`, `numKeysNode` são suficientemente completas.
    *   `getChildren`, `getKeyNode`, `getChildNode` são parciais como esperado (e.g., `getChildren` não definida para folhas).
*   **Para `BTree`:**
    *   `isEmpty`: Suficientemente completa (BT_IS_EMPTY1, BT_IS_EMPTY2).
    *   `getRoot`: Parcial para `emptyBTree`, definida para árvores não-vazias (BT_GET_ROOT).
    *   `searchKey`: Sua completeza depende da especificação completa de `searchKeyInNode`.
    *   `insertKey`, `deleteKey`: Incompletas na especificação fornecida.

Uma especificação algébrica totalmente construtiva e completa para `BTree` é um desafio significativo.

**Exercício:**

Para a especificação `BTreeNode`, o axioma (BTN6) é `numKeysNode(btn) = length(getKeys(btn))`. Se os construtores de `BTreeNode` são `createLeafNode(vt: Vector[T])` e `createInternalNode(vt_keys: Vector[T], vbtn_children: Vector[BTreeNode])`, realize uma análise de casos sobre os construtores de `btn` para mostrar que `numKeysNode` é suficientemente completa (ou seja, sempre se reduz a um `Natural`).

**Resolução:**

Operação: `numKeysNode(btn: BTreeNode)`.
Construtores de `BTreeNode`: `createLeafNode(vt)`, `createInternalNode(vt_keys, vbtn_children)`.
Axioma (BTN6): `numKeysNode(btn) = length(getKeys(btn))`.
Axiomas para `getKeys`:
(BTN3): `getKeys(createLeafNode(vt)) = vt`
(BTN4): `getKeys(createInternalNode(vt_keys, vbtn_children)) = vt_keys`
Assumimos `length: Vector[T] -> Natural` é suficientemente completa.

**Caso 1: `btn = createLeafNode(vt_inst)`**
`numKeysNode(createLeafNode(vt_inst))`
$= length(getKeys(createLeafNode(vt_inst)))` (por BTN6)
$= length(vt_inst)$ (por BTN3)
Como `length(vt_inst)` reduz a um `Natural` construtor, `numKeysNode` é completa para este caso.

**Caso 2: `btn = createInternalNode(vt_keys_inst, vbtn_children_inst)`**
`numKeysNode(createInternalNode(vt_keys_inst, vbtn_children_inst))`
$= length(getKeys(createInternalNode(vt_keys_inst, vbtn_children_inst)))` (por BTN6)
$= length(vt_keys_inst)$ (por BTN4)
Como `length(vt_keys_inst)` reduz a um `Natural` construtor, `numKeysNode` é completa para este caso.

**Conclusão:** `numKeysNode` é suficientemente completa.

# 17.3 Projeto e Implementação em Python com Tipagem Estática (MyPy)

A implementação de uma Árvore-B completa em Python é um projeto substancial. Focaremos no design da estrutura de um nó de Árvore-B e da própria Árvore-B, e esboçaremos a lógica da operação de busca.

**Design das Classes `BTreeNode` e `PyBTree`:**

```python
# py_btree.py
from __future__ import annotations 
from typing import TypeVar, Generic, List as PyList, Optional, Any, Tuple

K = TypeVar('K') # Type for keys, should be comparable

class BTreeNode(Generic[K]):
    # Represents a node in a B-Tree.
    def __init__(self, t_degree: int, is_leaf: bool = False):
        # t_degree is the minimum degree of the B-Tree.
        if t_degree < 2:
            raise ValueError("B-Tree minimum degree t must be at least 2.")
        
        self.t_degree: int = t_degree
        self.is_leaf: bool = is_leaf
        self.keys: PyList[K] = []
        self.children: PyList[BTreeNode[K]] = [] # Only used if not a leaf

    def __repr__(self) -> str:
        return f"BTreeNode(keys={self.keys}, leaf={self.is_leaf}, children_count={len(self.children)})"

    def is_full(self) -> bool:
        # A node is full if it has 2*t - 1 keys.
        return len(self.keys) == (2 * self.t_degree - 1)

    def _find_key_index_in_node(self, key_to_find: K) -> Tuple[int, bool]:
        # Performs a search for key_to_find within the node's keys list.
        # Returns: (index, found_boolean)
        # 'index' is where key is or should be inserted.
        # 'found_boolean' is True if key is found at that index.
        # Assumes self.keys is sorted.
        i = 0
        while i < len(self.keys) and key_to_find > self.keys[i]:
            i += 1
            
        if i < len(self.keys) and key_to_find == self.keys[i]:
            return (i, True) # Found
        else:
            return (i, False) # Not found, i is insertion point or child index

    def search_key_in_node(self, key_to_find: K) -> Tuple[Optional[BTreeNode[K]], Optional[int]]:
        # Searches for key_to_find in the subtree rooted at this node.
        # Returns a tuple: (node_for_next_step, index_in_node_if_key_found_here)
        idx, found = self._find_key_index_in_node(key_to_find)
        
        if found:
            return (self, idx) # Key found in this current node
        
        if self.is_leaf:
            return (self, None) # Key not in tree (if this is where it would be)
        else:
            # Descend to the appropriate child
            return (self.children[idx], None)


class PyBTree(Generic[K]):
    # Represents the B-Tree itself.
    def __init__(self, t_degree: int):
        if t_degree < 2:
            raise ValueError("B-Tree minimum degree t must be at least 2.")
        self.root: Optional[BTreeNode[K]] = None
        self.t_degree: int = t_degree

    def is_empty(self) -> bool:
        # Corresponds to 'isEmpty(tree: BTree[T]) --> Boolean'
        return self.root is None

    def search(self, key_to_find: K) -> Optional[Tuple[BTreeNode[K], int]]:
        # Corresponds to 'searchKey: T x BTree --> Boolean' (or finding the value)
        # Returns (node_containing_key, index_of_key_in_node) if found, else None.
        current_node = self.root
        while current_node is not None:
            # In search_key_in_node, 'found_node_for_next_step' is either the current node (if key found here),
            # or the child node to descend to, or self if leaf and not found.
            # 'key_idx_if_found' is the index if found in current_node, else None.
            found_node_for_next_step, key_idx_if_found = current_node.search_key_in_node(key_to_find)
            
            if key_idx_if_found is not None: # Key was found in current_node
                return (current_node, key_idx_if_found) 
            
            # If key not found in current_node
            if current_node.is_leaf: 
                return None # Not found and it's a leaf
            
            # Descend: found_node_for_next_step is the child node
            current_node = found_node_for_next_step 
        return None
        
    # Insert and Delete methods are complex and omitted for brevity.
    # def insert(self, key_to_insert: K) -> None: ...
    # def delete(self, key_to_delete: K) -> None: ...
```
O código Python acima, no arquivo `py_btree.py`, define as classes `BTreeNode[K]` e `PyBTree[K]`.
`BTreeNode` armazena chaves, filhos, grau e se é folha. Inclui `is_full` e `_find_key_index_in_node` (para busca interna no nó) e `search_key_in_node` (para busca na subárvore do nó).
`PyBTree` gerencia a raiz e o grau. `is_empty` verifica se a raiz é `None`. `search` implementa a busca descendo na árvore. As complexas operações de `insert` e `delete` são omitidas.

**Exercício:**

Implemente um método auxiliar `_find_key_index_in_node(self, key_to_find: K) -> tuple[int, bool]` na classe `BTreeNode[K]`. Este método deve realizar uma busca (e.g., linear ou binária se as chaves estiverem ordenadas) dentro do array `self.keys` do nó. Ele deve retornar uma tupla `(index, found)`, onde `index` é o índice da chave se encontrada, ou o índice onde a chave *deveria ser inserida* para manter a ordem se não encontrada. `found` é um booleano indicando se a chave foi encontrada. (Para este exercício, pode assumir uma busca linear).
*(Este exercício já foi resolvido e incorporado na classe `BTreeNode[K]` acima.)*

**Resolução:**

O método `_find_key_index_in_node` já foi adicionado à classe `BTreeNode[K]` na listagem de código acima. Sua implementação é:
```python
# py_btree.py (dentro da classe BTreeNode[K])
# ...
    def _find_key_index_in_node(self, key_to_find: K) -> Tuple[int, bool]:
        # Performs a search for key_to_find within the node's keys list.
        # Returns: (index, found_boolean)
        # 'index' is where key is or should be inserted.
        # 'found_boolean' is True if key is found at that index.
        # Assumes self.keys is sorted.
        i = 0
        while i < len(self.keys) and key_to_find > self.keys[i]: # Assumes K supports >
            i += 1
            
        if i < len(self.keys) and key_to_find == self.keys[i]: # Assumes K supports ==
            return (i, True) # Found
        else:
            return (i, False) # Not found, i is insertion point or child index
# ...
```
Este método realiza uma busca linear. Se a chave é encontrada, retorna seu índice e `True`. Caso contrário, retorna o índice onde a chave seria inserida e `False`.

# 17.4 Verificação de Propriedades Axiomáticas com Hypothesis

Verificar uma implementação de Árvore-B com Hypothesis focaria em:
1.  **Invariantes da Estrutura:** Após inserções/remoções, a árvore deve manter as propriedades de Árvore-B.
2.  **Propriedades das Operações:** `search` deve encontrar chaves inseridas e não encontrar chaves não inseridas/removidas. `insert`/`delete` devem alterar o conjunto de chaves corretamente.
3.  **Modelo de Referência:** Comparar o conjunto de chaves na `PyBTree` com um `set` Python após operações.

Gerar `PyBTree` válidas diretamente é complexo. Uma abordagem comum é gerar sequências de operações (inserções, deleções) a partir de uma árvore vazia e verificar propriedades em cada etapa ou no final.

**Exemplo de Propriedade para Testar (Conceitual):**
```python
# test_py_btree.py (esboço conceitual)
from hypothesis import given, strategies as st, assume, settings, HealthCheck
# from py_btree import PyBTree # Assuming K is int

st_keys_to_operate = st.lists(st.integers(min_value=0, max_value=1000), unique=True, max_size=30)
st_degree_t = st.integers(min_value=2, max_value=5)

# @given(t_degree=st_degree_t, keys_to_insert=st_keys_to_operate)
# @settings(deadline=None, suppress_health_check=[HealthCheck.too_slow])
# def test_insert_results_in_member(t_degree: int, keys_to_insert: PyList[int]):
#     btree = PyBTree[int](t_degree)
#     for key_val in keys_to_insert:
#         btree.insert(key_val) # Assumes insert is implemented and mutative
#     
#     for key_val in keys_to_insert:
#         search_result = btree.search(key_val)
#         assert search_result is not None
#         if search_result:
#             node_found, index_in_node = search_result
#             assert node_found.keys[index_in_node] == key_val
```
O código de teste conceitual `test_py_btree.py` define estratégias para o grau $t$ e uma lista de chaves para inserir. A função `test_insert_results_in_member` criaria uma `PyBTree`, inseriria todas as chaves e depois verificaria se cada chave inserida pode ser encontrada usando `search`. (Este teste requer uma implementação funcional de `insert` e `search`).

**Exercício:**

Descreva uma propriedade que relacione as operações `insert` e `delete` em uma `PyBTree[K]`. Por exemplo, o que deveria acontecer se você inserir uma chave `k` e depois deletá-la imediatamente? Como você testaria isso com Hypothesis, focando no estado da árvore (e.g., seu conjunto de chaves) em relação a um modelo de referência como um `set` Python?

**Resolução:**

**Propriedade:** Se uma chave `k` é inserida em uma Árvore-B `tree` e, em seguida, a mesma chave `k` é deletada da árvore resultante, o conjunto de chaves na árvore final deve ser o mesmo que o conjunto de chaves em `tree` antes da inserção de `k`, a menos que `k` já estivesse presente em `tree` (neste caso, o conjunto final seria `tree` sem `k`). Uma forma mais simples: o conjunto de chaves $(S \cup \{k\}) \setminus \{k\}$ deve ser igual a $S \setminus \{k\}$, onde $S$ é o conjunto inicial de chaves.

**Estratégia de Teste com Hypothesis e Modelo de Referência:**
1.  **Estratégias:** `st_degree_t`, `st_initial_keys` (lista de chaves únicas), `st_key_to_op` (chave para inserir/deletar).
2.  **Função de Teste:**
    ```python
    # test_py_btree.py (continuação)
    # Assume PyBTree tem insert(key), delete(key), e to_set_of_keys() -> set[K]

    # Mock implementation of to_set_of_keys for PyBTree for the test structure
    # In a real scenario, PyBTree would have this method.
    # def _mock_to_set_of_keys(btree_root_node):
    #     if not btree_root_node: return set()
    #     keys = set(btree_root_node.keys)
    #     if not btree_root_node.is_leaf:
    #         for child in btree_root_node.children:
    #             keys.update(_mock_to_set_of_keys(child))
    #     return keys

    @given(t_degree=st_degree_t,
           initial_keys=st.lists(st.integers(min_value=0, max_value=100), unique=True, max_size=20),
           key_to_op=st.integers(min_value=0, max_value=100))
    @settings(deadline=None, suppress_health_check=[HealthCheck.too_slow]) 
    def test_model_based_insert_then_delete(t_degree: int, initial_keys: PyList[int], key_to_op: int):
        # 1. Setup B-Tree and reference model (Python set)
        btree = PyBTree[int](t_degree)
        reference_set = set()
        for k_init in initial_keys:
            # Assume btree.insert and btree.delete are mutative for this test
            btree.insert(k_init) 
            reference_set.add(k_init)
        
        # 2. Perform operations on both B-Tree and reference set
        # Operation 1: Insert key_to_op
        btree.insert(key_to_op)
        reference_set.add(key_to_op) # Set add is idempotent
        
        # Operation 2: Delete key_to_op
        btree.delete(key_to_op)
        reference_set.discard(key_to_op) # Set discard removes if present, no error if not
        
        # 3. Verification: Compare the set of keys in B-Tree with the reference set
        # This requires PyBTree to have a method to get all its keys as a set.
        # assert btree.to_set_of_keys() == reference_set 
        # For now, we'll just assert search behavior based on reference_set
        
        # All keys in reference_set should be in btree
        for k_ref in reference_set:
            assert btree.search(k_ref) is not None, f"{k_ref} should be in btree"
            
        # No key from btree should be absent from reference_set (needs to_set_of_keys)
        # For simplicity here, we can check one extra key that shouldn't be there
        # if it's not in reference_set
        # For a full check, a to_set_of_keys() method on PyBTree is essential.
        # Example check:
        if not reference_set: # if reference_set is empty
             temp_btree_keys = btree.to_list() # conceptual: gets all keys in btree
             assert not temp_btree_keys, "BTree should be empty if reference_set is empty"

    # A more direct test for the property (S union {k}) - {k} = S - {k}
    # requires a reliable way to get all keys from the BTree.
    # Let's assume btree.to_set_of_keys() exists and works.
    @given(t_degree=st_degree_t,
           initial_keys_list=st.lists(st.integers(min_value=0, max_value=100), unique=True, max_size=20),
           key_k=st.integers(min_value=0, max_value=100))
    @settings(deadline=None, suppress_health_check=[HealthCheck.too_slow])
    def test_insert_delete_sequence_model(t_degree: int, initial_keys_list: PyList[int], key_k: int):
        # Setup
        btree = PyBTree[int](t_degree)
        model_set = set()
        for k_val in initial_keys_list:
            btree.insert(k_val)
            model_set.add(k_val)

        # State before key_k operations
        # keys_in_btree_before_op_k = btree.to_set_of_keys() # Needs implementation of to_set_of_keys
        model_set_before_op_k = model_set.copy()

        # Insert key_k
        btree.insert(key_k)
        model_set.add(key_k)
        # assert btree.to_set_of_keys() == model_set

        # Delete key_k
        btree.delete(key_k)
        model_set.discard(key_k) # Use discard to avoid error if key_k not in set
        
        # Verify
        # assert btree.to_set_of_keys() == model_set
        # For now, check individual elements based on model_set
        all_possible_keys_in_domain = set(initial_keys_list).union({key_k})
        for test_key in all_possible_keys_in_domain:
            btree_has_key = (btree.search(test_key) is not None)
            model_has_key = (test_key in model_set)
            assert btree_has_key == model_has_key, \
                f"Mismatch for key {test_key}: BTree has={btree_has_key}, Model has={model_has_key}"

    ```

**Explicação do Teste de Propriedade `test_model_based_insert_then_delete` e `test_insert_delete_sequence_model`:**
O teste `test_model_based_insert_then_delete` (primeira versão) e `test_insert_delete_sequence_model` (versão mais completa) usam um `set` Python como modelo de referência.
1.  Uma `PyBTree` e um `set` são inicializados com as mesmas `initial_keys`.
2.  A `key_to_op` (ou `key_k`) é inserida em ambos.
3.  A `key_to_op` (ou `key_k`) é deletada de ambos.
4.  A verificação final compara o conteúdo da `PyBTree` com o `reference_set`. A maneira mais robusta é implementar um método `to_set_of_keys()` na `PyBTree` que retorne todas as chaves como um conjunto Python, e então `assert btree.to_set_of_keys() == reference_set`. Na ausência disso, o teste `test_insert_delete_sequence_model` itera sobre um domínio de chaves relevantes e verifica a pertença em ambos, o que é uma aproximação. Este teste verifica se a sequência de `insert` e `delete` na Árvore-B espelha o comportamento de `add` e `discard` (ou `remove`) em um conjunto Python.

# 17.5 Análise de Complexidade Algorítmica

A principal razão de ser das Árvores-B é sua eficiência em operações sobre grandes volumes de dados armazenados em disco. Seja $N$ o número total de chaves e $t$ o grau mínimo ($t \ge 2$).

1.  **Altura da Árvore-B:** $h \le \log_t \frac{N+1}{2}$. Como $t$ é grande, $h$ é muito pequena.

2.  **Busca (`search`):**
    *   Acessos a disco: $O(h) = O(\log_t N)$.
    *   CPU por nó: $O(t)$ (busca linear) ou $O(\log t)$ (busca binária).
    *   CPU Total: $O(t \log_t N)$ ou $O(\log t \cdot \log_t N)$.

3.  **Inserção (`insert`):**
    *   Busca: $O(\log_t N)$ acessos a disco.
    *   Splits (pior caso, propagam até a raiz): $O(h)$ splits. Cada split é $O(t)$ CPU e $O(1)$ acessos a disco.
    *   Disco Total: $O(\log_t N)$. CPU Total: $O(t \log_t N)$.

4.  **Remoção (`delete`):**
    *   Busca: $O(\log_t N)$ acessos a disco.
    *   Rebalanceamento (rotações/fusões, pior caso propagam até a raiz): Cada operação é $O(t)$ CPU e $O(1)$ acessos a disco.
    *   Disco Total: $O(\log_t N)$. CPU Total: $O(t \log_t N)$.

As Árvores-B são eficientes devido ao grande fator de ramificação $t$.

**Exercício:**

Se uma Árvore-B tem $N=1,000,000$ (um milhão) de chaves e seu grau mínimo $t=50$. Qual é a altura máxima $h$ aproximada desta árvore? Como isso se compara com a altura de uma árvore de busca binária balanceada (como AVL ou Rubro-Negra) para o mesmo número de chaves?

**Resolução:**

Altura $h \le \log_t \frac{N+1}{2}$.
$N = 10^6$, $t=50$.
$h \le \log_{50} \frac{10^6+1}{2} \approx \log_{50} (5 \times 10^5)$.
$h \le \frac{\log_{10} (5 \times 10^5)}{\log_{10} 50} = \frac{5.69897}{1.69897} \approx 3.354$.
Altura máxima $h = 3$.

**Comparação com Árvore Binária Balanceada:**
Altura $h_{bin} \approx O(\log_2 N)$.
$h_{bin} \approx \log_2 10^6 \approx 19.93$. Altura $\approx 20$.

**Comparativo:** Árvore-B ($t=50$): Altura $\approx 3$. Árvore Binária Balanceada: Altura $\approx 20$.
A Árvore-B é significativamente mais rasa.

# 17.6 Exercícios Teóricos e Práticos

Esta seção apresenta um exercício para consolidar os conceitos do TAD `BTree[T]`.

**Exercício:**

**Contexto:** Considerando a estrutura de um nó de Árvore-B e a operação de busca.

**Parte a) Teórica:**
Em um nó de uma Árvore-B de grau mínimo $t$, não sendo a raiz e não estando cheio, quantas chaves ele contém no mínimo e no máximo (sem contar a raiz)? Se este nó é interno, quantos filhos ele terá no mínimo e no máximo?

**Parte b) Prática (Conceitual):**
Esboce, em pseudocódigo ou em português estruturado, a lógica de uma função `find_path_to_key(key_to_find: K, current_node: BTreeNode[K]) -> PathInfo` que, em vez de apenas retornar se uma chave foi encontrada, retorna o "caminho" de busca. `PathInfo` poderia ser uma estrutura que contém uma lista de tuplas `(node, index_in_node_keys_or_child_index)`, representando os nós visitados e o índice da chave (se encontrada nesse nó) ou o índice do filho tomado para descer.

**Resolução:**

**Parte a) Teórica: Número de Chaves e Filhos em um Nó de Árvore-B**

Para uma Árvore-B de grau mínimo $t$ ($t \ge 2$):
1.  **Número de Chaves em um Nó (não-raiz):** Mínimo: $t-1$. Máximo: $2t-1$.
2.  **Número de Filhos em um Nó Interno (não-raiz):** Mínimo: $t$. Máximo: $2t$.

**Parte b) Prática (Conceitual): Esboço de `find_path_to_key`**

```
Function find_path_to_key(key_to_find: K, start_node: BTreeNode[K]) -> List[Tuple[BTreeNode[K], int]]:
  // Path is a list of (node_visited, index_chosen_or_key_found_at)
  // If key found at node_visited.keys[index_chosen_or_key_found_at], then it's key index.
  // Else, it's the child_index to take from node_visited.children.

  path_taken = EmptyList
  current_node = start_node

  While current_node IS NOT NULL:
    index_in_node, found_in_node = current_node._find_key_index_in_node(key_to_find)
    // _find_key_index_in_node returns (idx_where_key_is_or_should_be, boolean_found)
    
    Add (current_node, index_in_node) to path_taken

    If found_in_node:
      Return path_taken // Key found, path leads to it
    
    // Key not found in current_node
    If current_node.is_leaf:
      Return path_taken // Reached leaf, key not found, path shows where it would be
    Else:
      // Descend to appropriate child; index_in_node is the child index to follow
      current_node = current_node.children[index_in_node]
    End If
  End While

  Return path_taken // Should only be reached if start_node was NULL (empty tree)
End Function
```

**Explicação do Pseudocódigo:**
`find_path_to_key` percorre a árvore a partir de `start_node`. Em cada `current_node`, usa `_find_key_index_in_node` para localizar a chave ou o ponto de descida. O par `(current_node, index_in_node)` é adicionado ao `path_taken`. Se a chave é encontrada, o caminho é retornado. Se é uma folha e não encontrou, o caminho termina. Se é um nó interno e não encontrou, desce para o filho indicado por `index_in_node`.

Este capítulo concluiu a Parte III, focada em estruturas de dados hierárquicas, com uma análise detalhada do TAD `BTree[T]`. Discutimos sua definição abstrata, propriedades fundamentais (ordem, balanceamento) e a relevância de sua otimização para armazenamento em disco. Esboçamos uma especificação algébrica para `BTreeNode` e `BTree`, e o design de classes Python `BTreeNode` e `PyBTree` com foco na busca. A verificação com Hypothesis foi discutida conceitualmente. Analisamos a eficiência logarítmica das operações da Árvore-B. A Parte IV do livro avançará para Estruturas de Dados para Coleções e Relações, começando com o TAD `Map` no Capítulo 18.

---
## REFERÊNCIAS BIBLIOGRÁFICAS
BAYER, R.; MCCREIGHT, E. M. Organization and maintenance of large ordered indexes. **Acta Informatica**, v. 1, n. 3, p. 173-189, 1972.
*   *Resumo: O artigo seminal que introduziu as Árvores-B. É a referência original fundamental para entender o design, as propriedades e os algoritmos básicos de manutenção (inserção, deleção, splits, merges) das Árvores-B.*

COMER, D. The ubiquitous B-tree. **ACM Computing Surveys**, v. 11, n. 2, p. 121-137, jun. 1979.
*   *Resumo: Um excelente artigo de survey que explica de forma clara e abrangente as Árvores-B, suas variantes (B+-Trees, B*-Trees), e suas aplicações. Essencial para uma compreensão conceitual e prática da relevância e funcionamento das B-Trees.*

CORMEN, T. H.; LEISERSON, C. E.; RIVEST, R. L.; STEIN, C. **Introduction to Algorithms**. 3. ed. Cambridge, MA: MIT Press, 2009. (Chapter 18: B-Trees).
*   *Resumo: Este renomado livro texto de algoritmos possui um capítulo dedicado exclusivamente às Árvores-B, cobrindo suas propriedades, operações (busca, inserção, deleção com exemplos detalhados) e análise de complexidade. É uma referência padrão para o estudo algorítmico das B-Trees.*

ELMASRI, R.; NAVATHE, S. B. **Fundamentals of Database Systems**. 7. ed. Boston: Pearson, 2017. (Chapters on Indexing Structures).
*   *Resumo: Livros texto sobre sistemas de banco de dados geralmente contêm capítulos detalhados sobre estruturas de indexação, com foco proeminente em B-Trees e B+-Trees, explicando seu uso prático em SGBDs. Relevante para a seção sobre a relevância do TAD.*

GRAEFE, G. **Implementing B-Trees in C++**. Redmond, WA: Microsoft Research, Technical Report MSR-TR-2020-25, May 2020.
*   *Resumo: Um relatório técnico moderno que discute os desafios e as melhores práticas para a implementação eficiente de Árvores-B em C++. Embora focado em C++, as considerações sobre design e otimização são relevantes para qualquer implementação, incluindo em Python.*

KNUTH, D. E. **The Art of Computer Programming, Volume 3: Sorting and Searching**. 2. ed. Reading, MA: Addison-Wesley, 1998. (Section 6.2.4: Multiway Trees).
*   *Resumo: A obra monumental de Knuth discute árvores multi-caminhos, incluindo as Árvores-B, com sua profundidade e rigor característicos. Fornece uma análise matemática detalhada e o contexto histórico e teórico dessas estruturas.*
---
