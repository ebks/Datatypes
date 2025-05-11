
---
# Capítulo 10
# Tipo Abstrato de Dados Stack
---

Continuando nossa exploração sistemática de estruturas de dados lineares sob o prisma da formalidade, este capítulo se dedica ao Tipo Abstrato de Dados (TAD) `Stack[T]`, também conhecido como Pilha. A pilha é uma estrutura de dados linear fundamental caracterizada por sua política de acesso restrita, conhecida como LIFO (Last-In, First-Out), onde o último elemento inserido é o primeiro a ser removido. Esta propriedade confere às pilhas um papel crucial em uma vasta gama de algoritmos e aplicações computacionais, desde o gerenciamento de chamadas de função em tempo de execução e mecanismos de "desfazer" (undo) em editores, até a avaliação de expressões aritméticas e algoritmos de busca em grafos (como a busca em profundidade). Seguindo a metodologia estabelecida nos capítulos anteriores desta Parte II, iniciaremos com uma definição abstrata do TAD `Stack[T]`, identificando suas operações canônicas como `emptyStack` (criação de uma pilha vazia), `push` (inserção de um elemento no topo), `pop` (remoção do elemento do topo), `top` (acesso ao elemento do topo) e `isEmpty` (verificação de vacuidade). Em seguida, desenvolveremos uma especificação algébrica formal para `Stack[T]`, parametrizada pelo tipo de elemento `T`, detalhando seus sorts, operações e o conjunto de axiomas equacionais que capturam rigorosamente seu comportamento LIFO. Analisaremos a consistência e a completeza suficiente desta especificação. Posteriormente, projetaremos e implementaremos uma classe genérica `PyStack[T]` em Python, utilizando anotações de tipo MyPy, que realizará o TAD `Stack[T]`. A validação da corretude desta implementação será conduzida através do Teste Baseado em Propriedades com a biblioteca Hypothesis, traduzindo os axiomas da pilha em propriedades verificáveis. Concluiremos com uma análise da complexidade algorítmica das operações da classe `PyStack[T]` e um exercício prático para reforçar os conceitos abordados.

# 10.1 Definição Abstrata e Relevância do Tipo de Dados `Stack[T]`

O Tipo Abstrato de Dados `Stack[T]`, ou Pilha, é uma coleção linear de elementos do tipo genérico `T`, cujo acesso é restrito a uma extremidade, denominada **topo** da pilha. A característica definidora de uma pilha é sua política de acesso **LIFO (Last-In, First-Out)**: o elemento mais recentemente adicionado à pilha é sempre o primeiro a ser removido ou acessado. Esta disciplina de acesso simples, porém poderosa, torna a pilha uma estrutura de dados fundamental em ciência da computação.

**Definição Abstrata:**
Um `Stack[T]` pode ser conceituado como uma sequência finita de elementos do tipo `T`, onde todas as inserções (operações `push`) e remoções (operações `pop`) ocorrem em uma única extremidade, o topo. As operações essenciais que caracterizam um `Stack[T]` incluem:

1.  **Criação:**
    *   `emptyStack: --> Stack[T]`: Cria uma pilha vazia.
2.  **Modificação:**
    *   `push: T x Stack[T] --> Stack[T]`: Adiciona um elemento `T` ao topo da pilha, resultando em uma nova pilha.
    *   `pop: Stack[T] --> Stack[T]`: Remove o elemento do topo da pilha, resultando na pilha restante. Esta operação é parcial; não é definida para uma pilha vazia.
3.  **Acesso/Observação:**
    *   `top: Stack[T] --> T`: Retorna o elemento no topo da pilha sem removê-lo. Esta operação também é parcial; não é definida para uma pilha vazia.
    *   `isEmpty: Stack[T] --> Boolean`: Verifica se a pilha está vazia, retornando `true` se estiver, e `false` caso contrário.

Em especificações funcionais/imutáveis, `push` e `pop` retornam novas instâncias da pilha. Em contextos imperativos/mutáveis, elas modificariam a pilha existente.

**Relevância do TAD `Stack[T]`:**
A simplicidade da política LIFO confere ao TAD `Stack[T]` uma ampla gama de aplicações e uma importância teórica e prática significativa:

1.  **Gerenciamento de Chamadas de Função:** A pilha de execução (call stack) em muitas linguagens de programação gerencia o fluxo de controle de sub-rotinas.
2.  **Avaliação de Expressões:** Pilhas são usadas para converter expressões infixas para pós-fixas (RPN) e para avaliar expressões RPN.
3.  **Mecanismos de "Desfazer" (Undo):** Ações do usuário podem ser empilhadas para permitir o "desfazer".
4.  **Algoritmos de Busca e Travessia:** A Busca em Profundidade (DFS) em grafos e árvores utiliza uma pilha.
5.  **Backtracking:** Em algoritmos que exploram espaços de solução, uma pilha pode armazenar pontos de decisão.
6.  **Análise Sintática (Parsing):** Alguns tipos de parsers utilizam pilhas.
7.  **Implementação de Recursão:** A recursão é tipicamente implementada usando a pilha de execução.

A eficiência das operações de pilha é uma de suas grandes vantagens. Em implementações típicas, `push`, `pop`, `top`, e `isEmpty` são $O(1)$ (custo amortizado para `push` e `pop` em array dinâmico).

Neste capítulo, focaremos na especificação algébrica de `Stack[T]`, sua implementação imutável `PyStack[T]` em Python e a verificação de suas propriedades axiomáticas.

**Exercício:**

Considere um TAD `Stack[T]` com as operações `emptyStack`, `push(e,s)`, `pop(s)`, `top(s)`, e `isEmpty(s)`. Seja `T` o tipo `Natural`. Partindo de uma pilha vazia `s0 = emptyStack`, execute a seguinte sequência de operações e descreva o estado da pilha e o resultado da última operação, se houver:
1.  `s1 = push(natural_3, s0)`
2.  `s2 = push(natural_7, s1)`
3.  `s3 = pop(s2)`
4.  `result = top(s3)`
(onde `natural_3` e `natural_7` são os `Natural`s 3 e 7).

**Resolução:**

Vamos traçar a sequência de operações sobre a pilha de `Natural`s:

1.  **`s0 = emptyStack`**
    *   Estado da pilha `s0`: $\langle \rangle$ (pilha vazia)

2.  **`s1 = push(natural_3, s0)`**
    *   Estado da pilha `s1`: $\langle$ `natural_3` $\rangle$ (topo é `natural_3`)

3.  **`s2 = push(natural_7, s1)`**
    *   Estado da pilha `s2`: $\langle$ `natural_7, natural_3` $\rangle$ (topo é `natural_7`)

4.  **`s3 = pop(s2)`**
    *   Estado da pilha `s3`: $\langle$ `natural_3` $\rangle$ (topo é `natural_3`)

5.  **`result = top(s3)`**
    *   Resultado da operação: `result = natural_3`.
    *   Estado da pilha `s3` (após `top`): $\langle$ `natural_3` $\rangle$.

Portanto, ao final da sequência:
*   O estado da pilha `s3` é $\langle$ `natural_3` $\rangle$.
*   O valor de `result` é `natural_3`.

# 10.2 Especificação Algébrica Formal do TAD `Stack[T]`

A especificação algébrica formal para o Tipo Abstrato de Dados `Stack[T]` (Pilha genérica) define sua estrutura sintática (sorts e operações) e seu comportamento semântico (axiomas) de maneira precisa e independente de implementação. A especificação será parametrizada pelo tipo `T` dos elementos que a pilha armazena.

**SPEC ADT** ElementTypeForStack
**sorts:**
*   T
**END SPEC**

**SPEC ADT** Stack[ParamT: ElementTypeForStack]
**imports:**
*   ParamT
*   Boolean
*   Natural

**sorts:**
*   Stack

**operations:**
*   emptyStack: --> Stack
*   push: T x Stack --> Stack
*   pop: Stack --> Stack
*   top: Stack --> T
*   isEmpty: Stack --> Boolean
*   size: Stack --> Natural

**axioms:**
for all s: Stack, elem: T

*   (S1): isEmpty(emptyStack) = true
*   (S2): isEmpty(push(elem, s)) = false
*   (S3): pop(push(elem, s)) = s
*   (S4): top(push(elem, s)) = elem
*   (S5): size(emptyStack) = zero
*   (S6): size(push(elem, s)) = succ(size(s))

**END SPEC**

A especificação `Stack[ParamT: ElementTypeForStack]` define um TAD de Pilha genérica. `ElementTypeForStack` é uma especificação de parâmetro formal que introduz um sort `T`. A especificação `Stack` importa `ParamT`, `Boolean` e `Natural`. O sort principal é `Stack`.
As **operações** incluem os construtores `emptyStack` e `push`. As operações `pop`, `top`, `isEmpty` e `size` são observadores ou operações derivadas. `pop` e `top` são parciais, não definidas para `emptyStack` por estes axiomas.
Os **axiomas** (S1)-(S6) definem o comportamento LIFO: (S1)-(S2) para `isEmpty`; (S3) para `pop`; (S4) para `top`; (S5)-(S6) para `size`. As variáveis `s` e `elem` são universalmente quantificadas.

## 10.2.1 Análise de Corretude e Completeza da Especificação `Stack[T]`

Analisaremos a especificação `Stack[T]` quanto à consistência e completeza suficiente.

**Consistência:**
A especificação `Stack[T]` é construída sobre `Boolean`, `Natural` e um parâmetro `T`.
1.  **Proteção dos Tipos Importados:** Os axiomas definem operações que retornam `Boolean`, `Natural`, `Stack` ou `T`. Não parecem introduzir novas igualdades problemáticas nos tipos importados.
2.  **Consistência Interna de `Stack[T]`:** `emptyStack` é distinto de `push(elem, s)` devido a (S1), (S2) e `true != false`. A distinção entre pilhas não vazias é garantida pelas operações `top` e `pop`.
3.  **Modelo Inicial:** O modelo inicial para `Stack[T]` (sequências finitas com acesso LIFO) é consistente.

A especificação `Stack[T]` parece **consistente**.

**Completeza Suficiente:**
Analisamos as operações não-construtoras (`isEmpty`, `pop`, `top`, `size`) aplicadas a termos formados pelos construtores `emptyStack` e `push(elem, s)`.

1.  **`isEmpty(stk: Stack)`:** Suficientemente completa (casos S1, S2).
2.  **`pop(stk: Stack)`:** Suficientemente completa para pilhas não-vazias (via S3). Não definida para `emptyStack` (parcial).
3.  **`top(stk: Stack)`:** Suficientemente completa para pilhas não-vazias (via S4). Não definida para `emptyStack` (parcial).
4.  **`size(stk: Stack)`:** Suficientemente completa (casos S5, S6, por indução).

**Conclusão da Análise:**
*   **Consistência:** A especificação `Stack[T]` é consistente.
*   **Completeza Suficiente:** `isEmpty` e `size` são suficientemente completas. `pop` e `top` são parciais, não cobrindo `emptyStack`.

**Exercício:**

Adicione uma operação `replaceTop: T x Stack --> Stack` à especificação `Stack[T]`. Esta operação deve substituir o elemento do topo da pilha pelo novo elemento `T` fornecido. Se a pilha estiver vazia, ela deve permanecer vazia. Forneça os axiomas para `replaceTop`.

**Resolução:**

Adicionamos `replaceTop` à especificação `Stack[T]`.

**SPEC ADT** StackWithReplaceTop[ParamT: ElementTypeForStack]
**imports:**
*   ParamT
*   Boolean
*   Natural
*   Stack[ParamT]

**sorts:**
*   Stack

**operations:**
*   replaceTop: T x Stack --> Stack

**axioms:**
for all s: Stack, elemNew, elemOld: T

*   (S7): replaceTop(elemNew, emptyStack) = emptyStack
*   (S8): replaceTop(elemNew, push(elemOld, s)) = push(elemNew, s)

**END SPEC**

A especificação `StackWithReplaceTop` enriquece `Stack[T]`. A nova operação `replaceTop` recebe um novo elemento `elemNew` do tipo `T` e uma `Stack`, e retorna uma nova `Stack`.
O axioma (S7) define que aplicar `replaceTop` a uma `emptyStack` resulta em `emptyStack`.
O axioma (S8) define que aplicar `replaceTop` com `elemNew` a uma pilha não vazia `push(elemOld, s)` resulta em uma nova pilha `push(elemNew, s)`, efetivamente substituindo `elemOld` por `elemNew` no topo, enquanto mantém a cauda `s`.

# 10.3 Projeto e Implementação em Python com Tipagem Estática (MyPy)

Com a especificação algébrica formal do TAD `Stack[T]` definida, procedemos à sua implementação em Python. Criaremos uma classe genérica `PyStack[T]` que realizará o comportamento LIFO e as operações especificadas. Utilizaremos uma lista Python (`list`) como a estrutura de dados subjacente para armazenar os elementos, aproveitando suas operações eficientes de `append` e `pop` (no final) para simular o topo da pilha. A implementação seguirá um estilo funcional/imutável, onde operações como `push` e `pop` retornam novas instâncias de `PyStack`, e faremos uso de anotações de tipo MyPy.

**Design da Classe `PyStack[T]`:**

```python
# py_stack.py
from __future__ import annotations # For type hints of PyStack[T] within the class itself
from typing import List as PyListInternal, Optional, TypeVar, Generic, Any

T = TypeVar('T') # Generic type parameter for elements

class PyStack(Generic[T]):
    _elements: PyListInternal[T] # Internal Python list to store elements of type T
                                 # The "top" of the stack will be the end of this list.

    def __init__(self, initial_elements: Optional[PyListInternal[T]] = None) -> None:
        # Constructor for PyStack.
        # Accepts an optional list of elements of type T for initialization.
        # The order in initial_elements is assumed to be from bottom to top.
        if initial_elements is None:
            self._elements = []
        else:
            self._elements = list(initial_elements) # Creates a copy

    @staticmethod
    def empty_stack() -> PyStack[Any]: 
        # Corresponds to 'emptyStack: --> Stack[T]'
        # Returns an empty stack. Type of T is Any here because it's a static method.
        return PyStack()

    def push(self, value: T) -> PyStack[T]:
        # Corresponds to 'push: T x Stack[T] --> Stack[T]'.
        # Returns a NEW PyStack with the value added to the top. Immutable style.
        new_elements = self._elements + [value] # Appending to Python list simulates push to top
        return PyStack(new_elements)

    def pop(self) -> PyStack[T]:
        # Corresponds to 'pop: Stack[T] --> Stack[T]'.
        # Returns a NEW PyStack with the top element removed. Immutable style.
        # Raises IndexError if the stack is empty, aligning with typical Python behavior.
        if self.is_empty():
            raise IndexError("pop from empty stack")
        
        new_elements = self._elements[:-1] # All elements except the last one
        return PyStack(new_elements)
    
    def top(self) -> T: 
        # Corresponds to 'top: Stack[T] --> T'.
        # Returns the top element. Raises IndexError if empty.
        if self.is_empty():
            raise IndexError("top from empty stack")
        return self._elements[-1] # Last element of Python list is the top

    def is_empty(self) -> bool:
        # Corresponds to 'isEmpty: Stack[T] --> Boolean'
        return not self._elements # An empty list is Falsy in Python

    def size(self) -> int: # Size is a Natural, represented by int >= 0
        # Corresponds to 'size: Stack[T] --> Natural'
        return len(self._elements)
    
    def replace_top(self, new_value: T) -> PyStack[T]:
        # Corresponds to 'replaceTop: T x Stack[T] --> Stack[T]'
        # Returns a NEW PyStack with the top element replaced.
        # If stack is empty, returns an empty stack (as per axiom S7).
        if self.is_empty():
            return PyStack.empty_stack() 
        
        # Elements without the old top, then add the new_value
        elements_before_top = self._elements[:-1]
        new_elements = elements_before_top + [new_value]
        return PyStack(new_elements)

    def peek(self, n_from_top: int) -> T:
        # Returns the n-th element from the top of the stack (0-indexed).
        # n_from_top = 0 is the top element.
        # Raises TypeError for invalid n_from_top type, IndexError if out of range.
        if not (isinstance(n_from_top, int) and n_from_top >= 0):
            raise TypeError("n_from_top must be a non-negative integer.")
        
        current_size = self.size()
        if n_from_top >= current_size:
            raise IndexError("Peek index out of range (stack is not deep enough).")
            
        python_list_index = current_size - 1 - n_from_top
        return self._elements[python_list_index]

    # Methods for equality and representation
    def __eq__(self, other: object) -> bool:
        # Equality check based on type and internal elements.
        if not isinstance(other, PyStack):
            return NotImplemented
        return self._elements == other._elements

    def __repr__(self) -> str:
        # String representation. Represents stack with top element to the right.
        return f"PyStack({self._elements})"

    def to_list_bottom_to_top(self) -> PyListInternal[T]:
        # Helper for testing, returns elements from bottom to top as a Python list.
        return list(self._elements)
```
O código Python acima, no arquivo `py_stack.py`, define a classe genérica `PyStack[T]`.
Ela é parametrizada pelo tipo `T`. Uma lista Python interna `_elements` armazena os elementos, onde o final da lista representa o topo da pilha. O construtor `__init__` permite a inicialização opcional. O método estático `empty_stack()` retorna uma pilha vazia. As operações `push`, `pop`, `replace_top` e `peek` (do exercício) são implementadas de forma imutável, retornando novas instâncias. `top`, `is_empty`, e `size` são observadores. `pop` e `top` levantam `IndexError` para pilhas vazias. `peek` também valida seu argumento e levanta `TypeError` ou `IndexError`. Os métodos `__eq__` e `__repr__` são fornecidos, assim como `to_list_bottom_to_top` para testes.

**Exercício:**

Adicione um método `peek(self, n_from_top: int) -> T` à classe `PyStack[T]`. Este método deve retornar o n-ésimo elemento a partir do topo da pilha, onde `n_from_top = 0` corresponde ao elemento do topo (mesmo que `self.top()`), `n_from_top = 1` ao segundo elemento do topo, e assim por diante. A pilha não deve ser modificada. Se `n_from_top` for inválido (negativo ou maior ou igual ao tamanho da pilha), o método deve levantar `IndexError` (ou `TypeError` se o tipo de `n_from_top` for incorreto).
*(Este exercício já foi resolvido e incorporado na classe `PyStack[T]` acima.)*

**Resolução:**

O método `peek` já foi adicionado à classe `PyStack[T]` na listagem de código acima. Sua implementação é:
```python
# py_stack.py (dentro da classe PyStack[T])
    # ...
    def peek(self, n_from_top: int) -> T:
        # Returns the n-th element from the top of the stack (0-indexed).
        # n_from_top = 0 is the top element.
        # Raises TypeError for invalid n_from_top type, IndexError if out of range.
        if not (isinstance(n_from_top, int) and n_from_top >= 0):
            raise TypeError("n_from_top must be a non-negative integer.")
        
        current_size = self.size()
        if n_from_top >= current_size:
            raise IndexError("Peek index out of range (stack is not deep enough).")
            
        # The top element is at _elements[-1] (index current_size - 1)
        # The n-th from top (0-indexed) is at _elements[current_size - 1 - n_from_top]
        python_list_index = current_size - 1 - n_from_top
        return self._elements[python_list_index]
    # ...
```
Este método valida `n_from_top`, verifica se o índice está dentro dos limites da pilha e, em caso afirmativo, calcula o índice correspondente na lista Python interna (lembrando que o topo da pilha é o final da lista) e retorna o elemento.

# 10.4 Verificação de Propriedades Axiomáticas com Hypothesis

Com a especificação algébrica de `Stack[T]` e a implementação `PyStack[T]`, usamos Hypothesis para verificar se a implementação satisfaz os axiomas.

**Estratégias Hypothesis para `PyStack[T]` e `T`:**
Usaremos `st.integers()` como estratégia para `T` para fins de exemplo.

```python
# test_py_stack.py
import pytest
from hypothesis import given, strategies as st, assume
from typing import TypeVar, Any

# Import the implementation
from py_stack import PyStack # Assuming py_stack.py is accessible

# Strategy for generating element type 'T'. For this example, use integers.
st_element_type = st.integers(min_value=-50, max_value=50)

# Strategy for generating instances of the PyStack[T] class
# We instantiate PyStack with int for T for these tests
st_py_stack = st.builds(
    PyStack[int], 
    initial_elements=st.lists(st_element_type, max_size=10)
)

# Strategy to generate a non-empty PyStack[T]
st_non_empty_py_stack = st.builds(
    PyStack[int],
    initial_elements=st.lists(st_element_type, min_size=1, max_size=10)
)
```
O código de teste `test_py_stack.py` configura as estratégias. `st_element_type` gera inteiros. `st_py_stack` constrói `PyStack[int]` usando listas desses elementos. `st_non_empty_py_stack` garante que a pilha gerada não seja vazia.

**Testes de Propriedade para os Axiomas de `Stack[T]`:**
Axiomas (S1)-(S8) da Seção 10.2 e seu exercício.

```python
# test_py_stack.py (continuação)

# --- Axioms for isEmpty ---
def test_S1_is_empty_emptyStack():
    # (S1): isEmpty(emptyStack) = true
    assert PyStack[Any].empty_stack().is_empty() == True

@given(s=st_py_stack, elem=st_element_type)
def test_S2_is_empty_push(s: PyStack[int], elem: int):
    # (S2): isEmpty(push(elem, s)) = false
    assert s.push(elem).is_empty() == False

# --- Axioms for pop and top ---
@given(s=st_py_stack, elem=st_element_type)
def test_S3_pop_push(s: PyStack[int], elem: int):
    # (S3): pop(push(elem, s)) = s
    assert s.push(elem).pop() == s

@given(s=st_py_stack, elem=st_element_type)
def test_S4_top_push(s: PyStack[int], elem: int):
    # (S4): top(push(elem, s)) = elem
    assert s.push(elem).top() == elem

# Test behavior for pop/top on emptyStack (expect exceptions from Python impl)
def test_pop_on_empty_stack_raises_exception():
    # Test that pop() on an empty stack raises IndexError
    empty_s = PyStack[Any].empty_stack()
    with pytest.raises(IndexError):
        empty_s.pop()

def test_top_on_empty_stack_raises_exception():
    # Test that top() on an empty stack raises IndexError
    empty_s = PyStack[Any].empty_stack()
    with pytest.raises(IndexError):
        empty_s.top()

# --- Axioms for size ---
def test_S5_size_emptyStack():
    # (S5): size(emptyStack) = zero
    assert PyStack[Any].empty_stack().size() == 0

@given(s=st_py_stack, elem=st_element_type)
def test_S6_size_push(s: PyStack[int], elem: int):
    # (S6): size(push(elem, s)) = succ(size(s))
    assert s.push(elem).size() == s.size() + 1

# --- Axioms for replaceTop ---
@given(new_elem=st_element_type)
def test_S7_replaceTop_on_emptyStack(new_elem: int):
    # (S7): replaceTop(elemNew, emptyStack) = emptyStack
    empty_s = PyStack[Any].empty_stack()
    assert empty_s.replace_top(new_elem) == PyStack[Any].empty_stack()

@given(s=st_py_stack, old_elem=st_element_type, new_elem=st_element_type)
def test_S8_replaceTop_on_push(s: PyStack[int], old_elem: int, new_elem: int):
    # (S8): replaceTop(elemNew, push(elemOld, s)) = push(elemNew, s)
    stack_after_push = s.push(old_elem)
    result_stack = stack_after_push.replace_top(new_elem)
    expected_stack = s.push(new_elem)
    assert result_stack == expected_stack
```
Os testes acima verificam os axiomas (S1)-(S8).
`test_S1_is_empty_emptyStack` e `test_S5_size_emptyStack` testam os casos base com `empty_stack`.
Os testes decorados com `@given` usam as estratégias para gerar pilhas e elementos.
`test_S2_is_empty_push` verifica `isEmpty` após um `push`.
`test_S3_pop_push` e `test_S4_top_push` verificam as propriedades fundamentais de `pop` e `top` em relação a `push`.
Os testes `test_pop_on_empty_stack_raises_exception` e `test_top_on_empty_stack_raises_exception` verificam o comportamento de erro da implementação Python para as operações parciais.
`test_S6_size_push` verifica a propriedade recursiva de `size`.
`test_S7_replaceTop_on_emptyStack` e `test_S8_replaceTop_on_push` verificam os axiomas da operação `replaceTop`.

**Exercício:**

Considerando a operação `multi_pop(k: Natural, s: Stack[T]) -> Stack[T]` e seus axiomas da resolução do exercício da Seção 10.2 (S_MP1, S_MP2, S_MP3). Escreva testes de propriedade Hypothesis para verificar esses três axiomas para a implementação `py_stack.PyStack.multi_pop`.

**Resolução:**

```python
# test_py_stack.py (continuação)

# Strategy for k in multi_pop
st_natural_k_for_multipop = st.integers(min_value=0, max_value=12) # max_size of list is 10, k up to 12 covers more than size

# --- Axioms for multi_pop (from exercise 10.2 resolution) ---

@given(s=st_py_stack)
def test_S_MP1_multipop_zero_is_identity(s: PyStack[int]):
    # (S_MP1): multipop(zero, s) = s
    assert s.multi_pop(0) == s

@given(k=st_natural_k_for_multipop)
def test_S_MP2_multipop_on_empty_is_empty(k: int):
    # (S_MP2): multipop(k, emptyStack) = emptyStack
    empty_s = PyStack[Any].empty_stack()
    assert empty_s.multi_pop(k) == empty_s

@given(s=st_non_empty_py_stack, k=st.integers(min_value=1, max_value=12)) 
def test_S_MP3_multipop_recursive_step(s: PyStack[int], k: int):
    # (S_MP3): eq(k,zero) = false AND isEmpty(s) = false =>
    #          multipop(k, s) = multipop(pred(k), pop(s))
    # Test condition k > 0 is ensured by st.integers(min_value=1, ...)
    # Test condition not s.is_empty() is ensured by st_non_empty_py_stack
    
    lhs = s.multi_pop(k)
    
    # Calculate pop(s) first, then apply multi_pop with k-1
    # This assumes pop() raises error on empty, but st_non_empty_py_stack handles this.
    s_after_pop = s.pop()
    rhs = s_after_pop.multi_pop(k - 1)
    
    assert lhs == rhs
```
Os testes de propriedade para `multi_pop` são adicionados:
*   `st_natural_k_for_multipop` gera valores para `k`.
*   `test_S_MP1_multipop_zero_is_identity` verifica (S_MP1).
*   `test_S_MP2_multipop_on_empty_is_empty` verifica (S_MP2).
*   `test_S_MP3_multipop_recursive_step` verifica (S_MP3). Usa `st_non_empty_py_stack` e `st.integers(min_value=1)` para garantir que as premissas do axioma (`k > 0` e `s` não vazia) sejam satisfeitas pelos dados gerados, simplificando as asserções `assume`.

# 10.5 Análise de Complexidade Algorítmica

A implementação `PyStack[T]` utiliza uma lista Python (`list`) como sua estrutura de dados interna, com o topo da pilha correspondendo ao final da lista Python. As operações da lista Python `append` (para `push`), `pop` (para `pop`), e acesso ao último elemento `[-1]` (para `top`) são todas $O(1)$ em tempo amortizado (ou $O(1)$ direto para `[-1]` e `len`). No entanto, como nossa implementação `PyStack[T]` adota um estilo funcional/imutável, as operações que "modificam" a pilha (`push`, `pop`, `replace_top`, `multi_pop`) criam novas listas Python.

**Análise de Complexidade das Operações de `PyStack[T]` (Imutável):**

1.  **`__init__(self, initial_elements: Optional[PyListInternal[T]] = None)`:** $O(K)$ (onde $K$ é o número de elementos iniciais).
2.  **`empty_stack()`:** $O(1)$.
3.  **`push(self, value: T) -> PyStack[T]`:** $O(N)$ (onde $N$ é o tamanho da pilha original, devido à cópia `self._elements + [value]`).
4.  **`pop(self) -> PyStack[T]`:** $O(N)$ (devido ao slicing `self._elements[:-1]`).
5.  **`top(self) -> T`:** $O(1)$ (para pilha não vazia).
6.  **`is_empty(self) -> bool`:** $O(1)$.
7.  **`size(self) -> int`:** $O(1)$.
8.  **`replace_top(self, new_value: T) -> PyStack[T]`:** $O(N)$ (devido ao slicing e concatenação para criar a nova lista).
9.  **`peek(self, n_from_top: int) -> T`:** $O(1)$ (para `n_from_top` válido).
10. **`multi_pop(self, k: int) -> PyStack[T]`:**
    *   Se $k=0$: $O(1)$.
    *   Se $k \ge \text{size()}$: $O(1)$ (criação de `PyStack` vazia).
    *   Caso contrário, `self._elements[:num_to_keep]` é um slicing que copia `num_to_keep = size() - k` elementos. Custo $O(\text{size()} - k)$, que é $O(N)$. O construtor `PyStack` então copia esses $N-k$ elementos, também $O(N-k)$.
    *   **Complexidade Total:** $O(N)$ no pior caso (quando $k$ é pequeno e $N-k$ é grande).

A imutabilidade, que envolve a cópia da lista subjacente para operações de "modificação", leva a uma complexidade $O(N)$ para essas operações.

**Exercício:**

Se a operação `push` na classe `PyStack[T]` fosse implementada para modificar a pilha no local (mutável), usando `self._elements.append(value)` e a função retornasse `None` (ou `self` se quiséssemos encadear), qual seria a complexidade de tempo desta versão mutável de `push`?

**Resolução:**

Se a operação `push` na classe `PyStack[T]` fosse implementada de forma mutável usando `self._elements.append(value)`:
A operação `list.append(value)` do Python tem complexidade de tempo **$O(1)$ amortizado**.
Portanto, a complexidade de tempo desta versão mutável de `push` seria **$O(1)$ amortizado**.

# 10.6 Exercícios Teóricos e Práticos

Esta seção apresenta um exercício combinado para aplicar e estender os conceitos do TAD `Stack[T]`.

**Exercício:**

**Contexto:** Este exercício foca na extensão do TAD `Stack[T]` e sua implementação `PyStack[T]` com uma operação `multipop` que remove múltiplos elementos do topo da pilha.

**Parte a) Especificação Algébrica para `multipop` em `Stack[T]`**
Adicione uma operação `multipop: Natural x Stack[T] --> Stack[T]` à especificação algébrica `Stack[T]` (da Seção 10.2). Esta operação deve remover `k` (um `Natural`) elementos do topo da pilha `s`.
1.  Se `k` for `zero`, a pilha original `s` é retornada.
2.  Se a pilha `s` estiver vazia, `multipop(k, s)` retorna `emptyStack`.
3.  Se $k > 0$ e a pilha $s$ não estiver vazia (i.e., $s = \text{push(e, s')}$), então `multipop(k, push(e, s'))` é equivalente a `multipop(pred(k), s')`. (Use esta forma construtiva).
Forneça os axiomas para `multipop`.

**Parte b) Implementação Python para `multi_pop` em `PyStack[T]`**
Implemente o método correspondente `multi_pop(self, k: int) -> PyStack[T]` na classe `PyStack[T]` (da Seção 10.3). A implementação deve:
1.  Seguir o estilo funcional/imutável.
2.  Validar que `k` é um `int >= 0`. Se `k` for negativo, levantar `ValueError`.
3.  Remover os `k` elementos do topo. Se `k` for maior que o tamanho da pilha, o resultado deve ser uma pilha vazia.

**Parte c) Testes de Propriedade para `multi_pop` em `PyStack[T]`**
Usando Hypothesis, escreva testes de propriedade para sua implementação `multi_pop` que correspondam aos axiomas definidos na Parte (a).

**Resolução:**

**Parte a) Especificação Algébrica para `multipop` em `Stack[T]`**

**SPEC ADT** StackWithMultiPop[ParamT: ElementTypeForStack]
**imports:**
*   ParamT
*   Boolean
*   Natural
*   Stack[ParamT]

**sorts:**
*   Stack

**operations:**
*   multipop: Natural x Stack --> Stack

**axioms:**
for all s: Stack, elem: T, k: Natural

*   (S_MP1): multipop(zero, s) = s
*   (S_MP2): multipop(k, emptyStack) = emptyStack
*   (S_MP3): eq(k,zero) = false => multipop(k, push(elem, s)) = multipop(pred(k), s)

**END SPEC**

A especificação `StackWithMultiPop` enriquece `Stack[T]`. A nova operação `multipop` recebe um `Natural` `k` e uma `Stack` `s`.
*   (S_MP1): Remover `zero` elementos de `s` resulta em `s`.
*   (S_MP2): Tentar remover `k` elementos de uma `emptyStack` resulta em `emptyStack`.
*   (S_MP3): Se `k` não é `zero` (i.e., $k > 0$), então `multipop(k, push(elem, s))` (remover $k$ elementos de uma pilha cujo topo é `elem` e cauda é `s`) é o mesmo que `multipop(pred(k), s)` (remover $k-1$ elementos da cauda `s`). A condição `eq(k,zero) = false` garante que `pred(k)` é aplicado a um `Natural` não-zero.

**Parte b) Implementação Python para `multi_pop` em `PyStack[T]`**
Esta implementação já foi fornecida na Seção 10.3 e está correta em relação aos requisitos:
```python
# py_stack.py (método multi_pop da classe PyStack[T])
# ...
    def multi_pop(self, k: int) -> PyStack[T]:
        # Removes k elements from the top of the stack.
        # Returns a NEW PyStack.
        # k must be a non-negative integer.
        # If k is larger than stack size, an empty stack is returned.
        if not (isinstance(k, int) and k >= 0):
            raise ValueError("k must be a non-negative integer.")
        
        if k == 0:
            return self # Return self as it's immutable and no change occurred
        
        current_size = self.size()
        if self.is_empty() or k >= current_size:
            return PyStack.empty_stack() 
            
        # Number of elements to keep
        num_to_keep = current_size - k
        new_elements = self._elements[:num_to_keep] # Slice from beginning up to num_to_keep
        return PyStack(new_elements)
# ...
```

**Parte c) Testes de Propriedade para `multi_pop` em `PyStack[T]`**

```python
# test_py_stack.py (continuação)
# ... (importações e estratégias st_py_stack, st_element_type, st_natural_value como antes) ...

# Strategy for k in multi_pop, using st_natural_value for consistency in range
st_k_for_multipop = st_natural_value # Generates Natural from 0 to 50

# --- Testes para multi_pop Axioms ---

@given(s=st_py_stack)
def test_S_MP1_multipop_zero_is_identity(s: PyStack[int]):
    # (S_MP1): multipop(zero, s) = s
    assert s.multi_pop(0) == s

@given(k=st_k_for_multipop)
def test_S_MP2_multipop_on_empty_is_empty(k: int):
    # (S_MP2): multipop(k, emptyStack) = emptyStack
    empty_s = PyStack[Any].empty_stack()
    assert empty_s.multi_pop(k) == empty_s

@given(s_base=st_py_stack, elem_pushed=st_element_type, k=st.integers(min_value=1, max_value=55))
# k starts from 1 due to pred(k) and eq(k,zero)=false condition
# max_value for k slightly larger than max stack size + 1 to test edge cases of k
def test_S_MP3_multipop_recursive_on_push(s_base: PyStack[int], elem_pushed: int, k: int):
    # (S_MP3): eq(k,zero) = false => multipop(k, push(elem, s)) = multipop(pred(k), s)
    # 'assume(k > 0)' is implicitly handled by st.integers(min_value=1)
    
    stack_with_pushed_elem = s_base.push(elem_pushed)
    
    lhs = stack_with_pushed_elem.multi_pop(k)
    rhs = s_base.multi_pop(k - 1) # k-1 represents pred(k)
    
    assert lhs == rhs
```
Os testes de propriedade para `multi_pop` são adicionados:
*   `st_k_for_multipop` usa `st_natural_value` para gerar `k`.
*   `test_S_MP1_multipop_zero_is_identity` verifica (S_MP1).
*   `test_S_MP2_multipop_on_empty_is_empty` verifica (S_MP2).
*   `test_S_MP3_multipop_recursive_on_push` verifica (S_MP3). A estratégia para `k` começa em 1, satisfazendo a premissa `eq(k,zero)=false`. O lado esquerdo é `stack_with_pushed_elem.multi_pop(k)` e o lado direito é `s_base.multi_pop(k-1)`.

Estes testes cobrem os axiomas especificados para `multi_pop`.

Este capítulo introduziu o TAD `Stack[T]`, sua especificação algébrica formal parametrizada, e uma implementação Python genérica `PyStack[T]` com tipagem estática. Demonstramos como os axiomas são cruciais para definir o comportamento LIFO e como eles podem ser traduzidos em testes de propriedade com Hypothesis para validar a implementação. A análise de complexidade destacou os custos associados ao estilo imutável. O exercício prático estendeu o TAD e a implementação com `multipop` e seus testes. O próximo capítulo, Capítulo 11, abordará o TAD `Queue[T]` (Fila), que opera sob a política FIFO (First-In, First-Out), contrastando com a pilha e apresentando novos desafios de especificação e implementação.

---
## REFERÊNCIAS BIBLIOGRÁFICAS
AHO, A. V.; HOPCROFT, J. E.; ULLMAN, J. D. **Data Structures and Algorithms**. Reading, MA: Addison-Wesley, 1983.
*   *Resumo: Um texto fundamental que discute pilhas (stacks) como uma das estruturas de dados elementares, explicando suas operações básicas e aplicações clássicas, como avaliação de expressões e gerenciamento de chamadas de procedimento. Fornece o contexto clássico para o TAD Stack.*

CLAESSEN, K.; HUGHES, J. QuickCheck: a lightweight tool for random testing of Haskell programs. **ACM SIGPLAN Notices**, v. 35, n. 9, p. 268-279, set. 2000. (ICFP '00).
*   *Resumo: O artigo seminal que introduziu QuickCheck e o teste baseado em propriedades. Relevante para a Seção 10.4, pois Hypothesis é fortemente inspirado por QuickCheck, e os princípios de testar propriedades axiomáticas se aplicam diretamente.*

CORMEN, T. H.; LEISERSON, C. E.; RIVEST, R. L.; STEIN, C. **Introduction to Algorithms**. 3. ed. Cambridge, MA: MIT Press, 2009.
*   *Resumo: Contém uma seção sobre estruturas de dados elementares, incluindo pilhas e filas. Descreve as operações e fornece pseudocódigo para implementações baseadas em array, relevante para a análise de complexidade e o design da implementação `PyStack[T]`.*

GUTTAG, J. V. Abstract data types and the development of data structures. **Communications of the ACM**, v. 20, n. 6, p. 396-404, jun. 1977.
*   *Resumo: Artigo clássico que discute a especificação algébrica de TADs, frequentemente usando a Pilha como um exemplo canônico para ilustrar a definição de operações e axiomas. Fundamental para a abordagem da Seção 10.2.*

---

PYTHON SOFTWARE FOUNDATION. **Python `collections.deque` documentation**. Disponível em: https://docs.python.org/3/library/collections.html#collections.deque. Acesso em: 10 ago. 2023.
*   *Resumo: A `collections.deque` do Python é otimizada para appends e pops eficientes de ambas as extremidades, tornando-a uma base ideal para implementações eficientes de pilhas (e filas) em Python. Relevante ao considerar alternativas de implementação para `PyStack[T]` com foco em desempenho.*

SEDGEWICK, R.; WAYNE, K. **Algorithms**. 4. ed. Upper Saddle River, NJ: Addison-Wesley Professional, 2011.
*   *Resumo: Este livro texto moderno sobre algoritmos inclui uma discussão clara sobre pilhas, suas implementações (e.g., usando listas encadeadas ou arrays redimensionáveis) e suas aplicações. Útil para contextualizar a relevância e o design do TAD Stack[T].*
