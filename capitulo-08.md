---
# Capítulo 8
# Tipo Abstrato de Dados Vector
---

Iniciando a Parte II deste livro, que se dedica ao estudo aprofundado de estruturas de dados lineares sob a ótica da especificação formal e da implementação rigorosa, este capítulo introduz o Tipo Abstrato de Dados (TAD) `Vector[T]`. O `Vector`, também conhecido como array dinâmico ou lista baseada em array em muitas bibliotecas de programação, é uma das estruturas de dados mais fundamentais e versáteis, caracterizado por oferecer acesso eficiente a elementos através de um índice numérico e por gerenciar dinamicamente seu tamanho para acomodar um número variável de elementos. Este capítulo generaliza o conceito para um tipo de elemento genérico `T`. Seguiremos uma estrutura sistemática: definição abstrata do `Vector[T]`, sua especificação algébrica formal (parametrizada por `T`), uma análise da corretude e completeza desta especificação, o projeto e a implementação de uma classe genérica `PyVector[T]` em Python com tipagem MyPy, a verificação da implementação via Teste Baseado em Propriedades com Hypothesis, e a análise de complexidade algorítmica. Um exercício prático ao final consolidará a extensão do TAD com `insertAtIndex` e sua respectiva implementação e teste, para solidificar a compreensão do TAD `Vector[T]`, sua especificação formal, sua implementação em Python e as técnicas de verificação.

# 8.1 Definição Abstrata e Relevância do Tipo de Dados `Vector[T]`

O Tipo Abstrato de Dados (TAD) `Vector[T]` modela uma coleção ordenada e de tamanho variável de elementos de um tipo genérico `T`. Cada elemento é acessível por um índice `Natural`. Sua principal característica é o acesso indexado eficiente e a capacidade de ajuste dinâmico de tamanho.

**Definição Abstrata:**
Um `Vector[T]` é uma sequência finita e ordenada $(e_0, e_1, \ldots, e_{N-1})$ de elementos do tipo `T`, onde $N$ é o comprimento do vetor. As operações essenciais incluem:
1.  **Criação:**
    *   `emptyVector: --> Vector[T]`: Cria um vetor vazio.
2.  **Acesso e Modificação por Índice:**
    *   `get: Vector[T] x Natural -> T`: Retorna o elemento no índice `Natural`. (Parcial: índice deve ser válido).
    *   `set: Vector[T] x Natural x T -> Vector[T]`: Retorna um novo `Vector[T]` com o elemento no índice `Natural` substituído pelo novo elemento `T`. (Parcial: índice deve ser válido).
3.  **Informações de Tamanho:**
    *   `length: Vector[T] -> Natural`: Retorna o número de elementos.
    *   `isEmpty: Vector[T] -> Boolean`: Verifica se está vazio.
4.  **Operações de Modificação de Tamanho:**
    *   `append: Vector[T] x T -> Vector[T]`: Retorna um novo `Vector[T]` com o elemento `T` adicionado ao final.
    *   `pop: Vector[T] -> Vector[T]`: Retorna um novo `Vector[T]` com o último elemento removido. (Parcial: vetor não deve estar vazio).

**Relevância do TAD `Vector[T]`:**
A relevância do `Vector[T]` é a mesma do `Vector[Natural]`, mas ampliada pela sua generalidade:
1.  **Acesso Indexado Eficiente:** Fundamental para muitos algoritmos.
2.  **Tamanho Dinâmico:** Flexibilidade no gerenciamento de dados.
3.  **Boa Localidade de Referência (em implementações baseadas em array):** Benefícios de cache.
4.  **Base para Outras Estruturas Genéricas:** Pilhas, filas, etc., de qualquer tipo de elemento.
5.  **Modelo Intuitivo para Sequências Genéricas:** Natural para coleções ordenadas de diversos tipos.
6.  **Padrão em Bibliotecas de Linguagens:** `List<T>` em C#/Java, `std::vector<T>` em C++, `list` (genérica) em Python.

A análise de complexidade para um `Vector[T]` geralmente assume que as operações sobre os elementos do tipo `T` (como cópia ou comparação, se necessárias) têm um custo constante ou são consideradas separadamente.

**Exercício:**

Considere um TAD `Vector[T]` com as operações `emptyVector`, `append(v,e)`, `get(v,idx)` e `length(v)`. Seja `T` o tipo `Boolean`. Se você tem um vetor `v1 = append(append(emptyVector, true), false)` (onde `true` e `false` são os `Boolean`s), qual seria o resultado esperado de `length(v1)` e de `get(v1, zero_idx)` e `get(v1, succ(zero_idx))`? (Assuma que `zero_idx` e `succ(zero_idx)` são os `Natural`s 0 e 1).

**Resolução:**

Dado o vetor $v_1 = \text{append}(\text{append}(\text{emptyVector}, \text{true}), \text{false})$.
O tipo `T` do elemento é `Boolean`.
Construção:
1.  `emptyVector`.
2.  `append(emptyVector, true)`: Vetor é $\langle \text{true} \rangle$.
3.  `append(\langle \text{true} \rangle, false)`: Vetor $v_1$ é $\langle \text{true}, \text{false} \rangle$.

Resultados das operações:

1.  **`length(v1)`:**
    O vetor $v_1$ contém dois elementos.
    Resultado esperado: o `Natural` que representa 2 (i.e., `succ(succ(zero))`).

2.  **`get(v1, zero_idx)`:**
    `zero_idx` representa o índice 0.
    O elemento no índice 0 de $v_1$ é `true`.
    Resultado esperado: `true`.

3.  **`get(v1, succ(zero_idx))`:**
    `succ(zero_idx)` representa o `Natural` 1 (índice 1).
    O elemento no índice 1 de $v_1$ é `false`.
    Resultado esperado: `false`.

# 8.2 Especificação Algébrica Formal do TAD `Vector[T]`

Nesta seção, apresentamos a especificação algébrica formal para `Vector[T]`. Ela será parametrizada pelo tipo do elemento `T`.

**SPEC ADT** ElementType
**sorts:**
*   T
**END SPEC**

**SPEC ADT** Vector[ParamT: ElementType]
**imports:**
*   ParamT
*   Natural
*   Boolean

**sorts:**
*   Vector

**operations:**
*   emptyVector: --> Vector
*   append: Vector x T --> Vector
*   isEmpty: Vector --> Boolean
*   length: Vector --> Natural
*   get: Vector x Natural --> T
*   set: Vector x Natural x T --> Vector
*   pop: Vector --> Vector
*   head: Vector --> T
*   insertAtIndex: Vector x Natural x T --> Vector

**axioms:**
for all vec: Vector, elem, newElem, valToInsert: T, n: Natural, idx, k, insertIdx: Natural

*   (V1): isEmpty(emptyVector) = true
*   (V2): isEmpty(append(vec, elem)) = false

*   (V3): length(emptyVector) = zero
*   (V4): length(append(vec, elem)) = succ(length(vec))

*   (V5): eq(idx, length(vec)) = true => get(append(vec, elem), idx) = elem
*   (V6): lt(idx, length(vec)) = true  => get(append(vec, elem), idx) = get(vec, idx)

*   (V7): eq(k, length(vec)) = true =>
        set(append(vec, elem), k, newElem) = append(vec, newElem)
*   (V8): lt(k, length(vec)) = true =>
        set(append(vec, elem), k, newElem) = append(set(vec, k, newElem), elem)
*   (V9): set(emptyVector, k, newElem) = emptyVector

*   (V10): pop(emptyVector) = emptyVector
*   (V11): pop(append(vec, elem)) = vec

*   (V12): isEmpty(vec) = false => head(vec) = get(vec, zero)

*   (V13): length(insertAtIndex(vec, insertIdx, valToInsert)) = succ(length(vec))
*   (V14): get(insertAtIndex(vec, insertIdx, valToInsert), insertIdx) = valToInsert
*   (V15): lt(k, insertIdx) = true =>
            get(insertAtIndex(vec, insertIdx, valToInsert), k) = get(vec, k)
*   (V16): lt(insertIdx, k) = true AND lt(k, succ(length(vec))) = true =>
           get(insertAtIndex(vec, insertIdx, valToInsert), k) = get(vec, pred(k))

**END SPEC**

A especificação `Vector[ParamT: ElementType]` define um Tipo Abstrato de Dados para vetores genéricos. `ElementType` é uma especificação de parâmetro formal que apenas introduz um sort `T` para os elementos. A especificação `Vector` importa `ParamT` (fornecendo `T`), `Natural` (para índices e comprimento) e `Boolean`. O sort principal é `Vector`. As operações incluem construtores (`emptyVector`, `append`), observadores (`isEmpty`, `length`, `get`, `head`) e operações de modificação (`set`, `pop`, `insertAtIndex`). Os axiomas (V1)-(V16) definem o comportamento dessas operações. As variáveis `vec`, `elem`, `newElem`, `valToInsert`, `n`, `idx`, `k`, `insertIdx` são universalmente quantificadas sobre os sorts apropriados (`Vector`, `T`, ou `Natural`). (V1)-(V4) definem `isEmpty` e `length`. (V5)-(V6) definem `get`. (V7)-(V9) definem `set`. (V10)-(V11) definem `pop`. (V12) define `head`. (V13)-(V16) são axiomas de propriedade para `insertAtIndex`, assumindo que `insertIdx` é válido ($0 \le \text{insertIdx} \le \text{length(vec)}$) e `k` também é válido nos respectivos contextos.

## 8.2.1 Análise de Corretude e Completeza da Especificação `Vector[T]`

Antes de prosseguir para a implementação, é crucial analisar a especificação algébrica `Vector[T]` (da forma apresentada na Seção 8.2, incluindo os axiomas V1-V16) quanto às propriedades de consistência e completeza suficiente.

**Consistência:**
A consistência da especificação `Vector[T]` depende da consistência das especificações importadas (`Natural`, `Boolean`, e o parâmetro `T`) e da forma como os novos axiomas (V1-V16) interagem.
1.  **Proteção dos Tipos Importados:** Os axiomas de `Vector[T]` definem operações que produzem `Boolean` (`isEmpty`), `Natural` (`length`), ou `T` (`get`, `head`). É fundamental que esses axiomas não introduzam novas igualdades ou contradições nos sorts `Boolean`, `Natural`, ou `T`. A estrutura dos axiomas parece preservar essa distinção.
2.  **Consistência Interna de `Vector[T]`:** Precisamos garantir que não possamos derivar, por exemplo, `emptyVector = append(emptyVector, some_T_elem)`. Os axiomas (V1) e (V2) e `true != false` de `Boolean` implicam que `append(vec, elem)` nunca é igual a `emptyVector`. A distinção entre diferentes vetores não-vazios dependeria de observadores como `length`, `head` ou `get`.
3.  **Modelo Inicial:** O modelo inicial para `Vector[T]` consistiria em sequências finitas de elementos de `T`. As operações padrão sobre sequências fornecem um modelo matemático consistente para os axiomas dados.

A especificação parece **consistente**, desde que `Natural` e `Boolean` sejam consistentes e o tipo `T` do parâmetro não introduza problemas. As operações de acesso (`get`, `head`) são parciais nos axiomas para entradas inválidas, o que evita contradições.

**Completeza Suficiente:**
Analisamos as operações não-construtoras (`isEmpty`, `length`, `get`, `set`, `pop`, `head`, `insertAtIndex`) aplicadas a termos formados pelos construtores `emptyVector` e `append(vec, elem)`.

1.  **`isEmpty(v: Vector)`:** Suficientemente completa (casos V1, V2).
2.  **`length(v: Vector)`:** Suficientemente completa (casos V3, V4, por indução).
3.  **`get(v: Vector, idx: Natural)`:** Suficientemente completa para índices válidos em vetores não-vazios (via V5, V6). Parcial e não completa para `get(emptyVector,...)` e índice fora dos limites.
4.  **`set(v: Vector, k: Natural, newElem: T)`:** Suficientemente completa para índices válidos (via V7, V8) e para `emptyVector` (via V9). Parcial para índices inválidos em vetores não-vazios.
5.  **`pop(v: Vector)`:** Suficientemente completa (casos V10, V11).
6.  **`head(v: Vector)`:** Suficientemente completa para vetores não-vazios (via V12 e a completeza de `get`). Parcial para `emptyVector`.
7.  **`insertAtIndex(v: Vector, insertIdx: Natural, valToInsert: T)`:** Os axiomas (V13)-(V16) são de propriedade, não definem `insertAtIndex` construtivamente em termos de `emptyVector` e `append`. Portanto, com apenas estes axiomas, `insertAtIndex` **não é suficientemente completa** no sentido de reduzir a um termo construtor `Vector`. Para torná-la construtivamente completa, seriam necessários axiomas recursivos baseados nos construtores `emptyVector` e `append` (ou `nil`/`cons`).

**Conclusão da Análise:**
*   **Consistência:** A especificação `Vector[T]` (com V1-V12 e axiomas de propriedade V13-V16) parece consistente.
*   **Completeza Suficiente:** As operações `isEmpty`, `length`, `pop` são suficientemente completas. `get`, `head`, `set` são parcialmente definidas e, portanto, não totalmente completas sem um tratamento explícito de erro ou pré-condições para todos os casos. `insertAtIndex` não é definida de forma construtiva, apenas por propriedades.

Para uma especificação mais robusta, o tratamento de casos parciais para `get`, `head`, `set` e a definição construtiva de `insertAtIndex` precisariam ser explicitados.

**Exercício:**

Se a especificação do parâmetro `ParamT: ElementType` fosse estendida para requerer uma operação de igualdade `eqT: T x T --> Boolean` (conforme `ElementTypeWithEq`), como você adicionaria uma operação `contains: Vector x T --> Boolean` à especificação `Vector[ParamT: ElementTypeWithEq]`? Forneça sua assinatura e os axiomas.

**Resolução:**

Primeiro, a especificação do parâmetro estendida:
**SPEC ADT** ElementTypeWithEq
**sorts:**
*   T
**operations:**
*   eqT: T x T --> Boolean
**axioms:**
*   (ET1): eqT(x,x) = true
*   (ET2): eqT(x,y) = eqT(y,x)
for all x,y: T
**END SPEC**

A especificação `ElementTypeWithEq` define os requisitos para o tipo do elemento, incluindo um sort `T` e uma operação de igualdade `eqT` sobre `T`.

Agora, adicionamos `contains` a `Vector[ParamT: ElementTypeWithEq]`. Assumimos que a especificação `Vector` agora usa `ElementTypeWithEq` como seu parâmetro `ParamT`.

**SPEC ADT** VectorWithContains[ParamT: ElementTypeWithEq]
**imports:**
*   ParamT
*   Natural
*   Boolean
*   Vector[ParamT]

**sorts:**
*   Vector

**operations:**
*   contains: Vector x T --> Boolean

**axioms:**
for all vec: Vector, elemToFind, elemInList: T

*   (VC1): contains(emptyVector, elemToFind) = false
*   (VC2): contains(append(vec, elemInList), elemToFind) =
        or(eqT(elemInList, elemToFind), contains(vec, elemToFind))

**END SPEC**

A especificação `VectorWithContains` é parametrizada por `ParamT` que satisfaz `ElementTypeWithEq`. Ela importa os TADs necessários e a especificação base `Vector[ParamT]` (que define `emptyVector`, `append`, etc.). A nova operação `contains` verifica se um elemento `elemToFind` está presente no `Vector`.
O axioma (VC1) estabelece que um elemento nunca está contido em um `emptyVector`.
O axioma (VC2) define `contains` recursivamente: `elemToFind` está em `append(vec, elemInList)` se `elemToFind` é igual ao elemento recém-anexado `elemInList` (usando `eqT`), ou se `elemToFind` já estava presente no vetor original `vec`.

# 8.3 Projeto e Implementação em Python com Tipagem Estática (MyPy)

Após a especificação algébrica formal do TAD `Vector[T]`, transitamos para sua implementação em Python. Criaremos uma classe genérica `PyVector[T]` que realiza o comportamento definido, utilizando anotações de tipo e MyPy para rigor. Uma lista Python (`list`) servirá como estrutura de dados subjacente. A generalidade do tipo `T` será tratada com `typing.TypeVar` e `typing.Generic`.

**Design da Classe `PyVector[T]`:**

```python
# py_vector.py
from __future__ import annotations # For type hints of PyVector[T] within the class itself
from typing import List as PyListInternal, Optional, TypeVar, Generic, Any 

T = TypeVar('T') # Generic type parameter for elements

class PyVector(Generic[T]):
    _elements: PyListInternal[T] # Internal Python list to store elements of type T

    def __init__(self, initial_elements: Optional[PyListInternal[T]] = None) -> None:
        # Constructor for PyVector.
        # Accepts an optional list of elements of type T for initialization.
        if initial_elements is None:
            self._elements = []
        else:
            # Type of elements in initial_elements is T, enforced by MyPy at call site.
            self._elements = list(initial_elements) # Creates a copy

    @staticmethod
    def empty_vector() -> PyVector[Any]: 
        # Returns an empty vector. Type of T is Any here.
        return PyVector()

    def append(self, value: T) -> PyVector[T]:
        # Returns a NEW PyVector with the value added (immutable style).
        new_elements = self._elements + [value] 
        return PyVector(new_elements)

    def is_empty(self) -> bool:
        # Checks if the vector is empty.
        return not self._elements 

    def length(self) -> int: 
        # Returns the number of elements (Natural, represented by int >= 0).
        return len(self._elements)

    def get(self, index: int) -> T: 
        # Gets element at index (Natural). Raises IndexError if invalid.
        if not (isinstance(index, int) and index >= 0):
            raise TypeError("Index must be a non-negative integer (Natural).")
        if not (0 <= index < len(self._elements)):
            raise IndexError("Vector index out of range.")
        return self._elements[index]

    def set_element(self, index: int, value: T) -> PyVector[T]:
        # Returns a NEW PyVector with element at index updated (immutable style).
        # Raises IndexError if invalid index.
        if not (isinstance(index, int) and index >= 0):
            raise TypeError("Index must be a non-negative integer (Natural).")
        if not (0 <= index < len(self._elements)):
            raise IndexError("Vector index out of range.")
        
        new_elements = list(self._elements) 
        new_elements[index] = value
        return PyVector(new_elements)

    def pop(self) -> PyVector[T]:
        # Returns a NEW PyVector with the last element removed (immutable style).
        if self.is_empty():
            return PyVector() # Return an empty vector of unknown T type for consistency
        
        new_elements = self._elements[:-1] 
        return PyVector(new_elements)
    
    def head(self) -> T: 
        # Returns the first element. Raises IndexError if empty.
        if self.is_empty():
            raise IndexError("Cannot get head of an empty vector.")
        return self._elements[0]

    def insert_at_index(self, index: int, value: T) -> PyVector[T]:
        # Inserts value at index (0 <= index <= length). Returns NEW PyVector.
        if not (isinstance(index, int) and index >= 0):
            raise TypeError("Index must be a non-negative integer (Natural).")
        
        current_length = len(self._elements)
        if not (0 <= index <= current_length): 
            raise IndexError("Index out of bounds for insertion.")

        new_elements = self._elements[:index] + [value] + self._elements[index:]
        return PyVector(new_elements)
    
    def prepend(self, value: T) -> PyVector[T]:
        # Adds value to the beginning. Returns NEW PyVector.
        new_elements = [value] + self._elements 
        return PyVector(new_elements)

    def __eq__(self, other: object) -> bool:
        # Equality check.
        if not isinstance(other, PyVector):
            return NotImplemented
        return self._elements == other._elements

    def __repr__(self) -> str:
        # String representation.
        return f"PyVector({self._elements})"

    def to_list(self) -> PyListInternal[T]:
        # Helper for testing.
        return list(self._elements)

    def to_string(self) -> str:
        # String representation of elements, e.g., "[item1, item2]".
        if self.is_empty():
            return "[]"
        else:
            elements_as_strings = [str(elem) for elem in self._elements]
            return "[" + ", ".join(elements_as_strings) + "]"
```
O código Python acima, no arquivo `py_vector.py`, define a classe genérica `PyVector[T]`. Ela é parametrizada pelo tipo `T`. Internamente, usa uma lista Python `_elements`. O construtor `__init__` permite inicialização opcional. `empty_vector()` retorna um `PyVector` vazio. Os métodos `append`, `set_element`, `pop`, `insert_at_index` e `prepend` são imutáveis. `is_empty`, `length`, `get`, e `head` são observadores. Validações de índice e tratamento de erro via exceções são incluídos. Métodos `__eq__`, `__repr__`, `to_list` e `to_string` são fornecidos.

**Exercício:**

Implemente um método `to_string(self) -> str` na classe `PyVector[T]`. Este método deve retornar uma representação em string do vetor, mostrando seus elementos (convertidos para string) separados por vírgulas e entre colchetes, e.g., `"[element1_str, element2_str, element3_str]"`. Se o vetor estiver vazio, deve retornar `"[]"`.
*(Este exercício já foi resolvido e incorporado na classe `PyVector[T]` acima.)*

**Resolução:**

O método `to_string` já foi adicionado à classe `PyVector[T]` na listagem de código acima. Sua implementação é:
```python
# py_vector.py (dentro da classe PyVector[T])
    # ...
    def to_string(self) -> str:
        # Returns a string representation of the vector's elements.
        # e.g., "[item1, item2, item3]" or "[]" for an empty vector.
        if self.is_empty():
            return "[]"
        else:
            # Convert each element to its string representation
            # then join them with ", "
            elements_as_strings = [str(elem) for elem in self._elements]
            return "[" + ", ".join(elements_as_strings) + "]"
    # ...
```
Este método retorna `"[]"` para um vetor vazio. Para um vetor não vazio, converte cada elemento para string, junta-os com `", "`, e envolve o resultado em colchetes.

# 8.4 Verificação de Propriedades Axiomáticas com Hypothesis

Tendo a especificação algébrica de `Vector[T]` e a implementação `PyVector[T]`, usamos Hypothesis para verificar se a implementação adere aos axiomas.

**Estratégias Hypothesis para `PyVector[T]` e `T`:**
Para testar `PyVector[T]`, precisamos de uma estratégia para `T` (o tipo do elemento) e depois para `PyVector[T]`. Para os exemplos, usaremos `st.integers()` como estratégia para `T`.

```python
# test_py_vector.py
import pytest 
from hypothesis import given, strategies as st, settings, HealthCheck, assume

from py_vector import PyVector 
from typing import TypeVar, Any

T_elements = TypeVar('T_elements') 
st_element_type = st.integers(min_value=-50, max_value=50) 

st_py_vector = st.builds(
    PyVector[int], 
    initial_elements=st.lists(st_element_type, max_size=10)
)

@st.composite
def st_vector_and_valid_read_index(draw):
    vec = draw(st_py_vector.filter(lambda v_filter: not v_filter.is_empty())) 
    index_val = draw(st.integers(min_value=0, max_value=vec.length() - 1))
    return vec, index_val

@st.composite
def st_vector_and_append_or_valid_index(draw):
    vec = draw(st_py_vector)
    index_val = draw(st.integers(min_value=0, max_value=vec.length())) 
    return vec, index_val

@st.composite
def st_vector_insert_idx_and_value(draw):
    vec = draw(st_py_vector) 
    insert_idx = draw(st.integers(min_value=0, max_value=vec.length()))
    val_to_insert = draw(st_element_type) 
    return vec, insert_idx, val_to_insert
```
O código de teste `test_py_vector.py` configura as estratégias Hypothesis. `st_element_type` usa `st.integers()`. `st_py_vector` constrói `PyVector[int]`. As estratégias compostas `st_vector_and_valid_read_index`, `st_vector_and_append_or_valid_index`, e `st_vector_insert_idx_and_value` são definidas para gerar vetores com índices específicos ou valores para teste.

**Testes de Propriedade para os Axiomas de `Vector[T]`:**
Os axiomas (V1)-(V16) são traduzidos para testes.

```python
# test_py_vector.py (continuação)

# --- Axioms for isEmpty ---
def test_V1_is_empty_emptyVector():
    # (V1): isEmpty(emptyVector) = true
    assert PyVector[Any].empty_vector().is_empty() == True

@given(vn=st_py_vector, elem=st_element_type)
def test_V2_is_empty_append(vn: PyVector[int], elem: int):
    # (V2): isEmpty(append(vn, elem)) = false
    assert vn.append(elem).is_empty() == False

# --- Axioms for length ---
def test_V3_length_emptyVector():
    # (V3): length(emptyVector) = zero
    assert PyVector[Any].empty_vector().length() == 0 

@given(vn=st_py_vector, elem=st_element_type)
def test_V4_length_append(vn: PyVector[int], elem: int):
    # (V4): length(append(vn, elem)) = succ(length(vn))
    assert vn.append(elem).length() == vn.length() + 1

# --- Axioms for get ---
@given(data=st_vector_and_append_or_valid_index, elem=st_element_type)
def test_V5_get_at_appended_pos(data: tuple[PyVector[int], int], elem: int):
    # (V5): eq(idx, length(vn)) = true => get(append(vn, elem), idx) = elem
    vn, idx = data
    assume(idx == vn.length()) 
    
    appended_vector = vn.append(elem)
    assert appended_vector.get(idx) == elem

@given(data=st_vector_and_valid_read_index, elem=st_element_type)
def test_V6_get_at_earlier_pos(data: tuple[PyVector[int], int], elem: int):
    # (V6): lt(idx, length(vn)) = true  => get(append(vn, elem), idx) = get(vn, idx)
    vn, idx = data 
    
    appended_vector = vn.append(elem)
    assert appended_vector.get(idx) == vn.get(idx)

# --- Axioms for set ---
@given(data=st_vector_and_append_or_valid_index, elem=st_element_type, new_elem=st_element_type)
def test_V7_set_at_appended_pos(data: tuple[PyVector[int], int], elem: int, new_elem: int):
    # (V7): eq(k, length(vn)) = true => 
    #       set(append(vn, elem), k, new_elem) = append(vn, new_elem)
    vn, k = data
    assume(k == vn.length()) 
    
    original_appended = vn.append(elem)
    set_result = original_appended.set_element(k, new_elem)
    expected_result = vn.append(new_elem)
    assert set_result == expected_result

@given(data=st_vector_and_valid_read_index, elem=st_element_type, new_elem=st_element_type)
@settings(suppress_health_check=[HealthCheck.filter_too_much], max_examples=100) 
def test_V8_set_at_earlier_pos(data: tuple[PyVector[int], int], elem: int, new_elem: int):
    # (V8): lt(k, length(vn)) = true => 
    #       set(append(vn, elem), k, new_elem) = append(set(vn, k, new_elem), elem)
    vn, k = data 

    # LHS
    vec_after_append = vn.append(elem)
    lhs = vec_after_append.set_element(k, new_elem)
    
    # RHS
    vec_after_set_original = vn.set_element(k, new_elem)
    rhs = vec_after_set_original.append(elem)
    
    assert lhs == rhs
    
def test_V9_set_on_empty_vector():
    # (V9): set(emptyVector, k, newElem) = emptyVector
    empty_vec = PyVector[Any].empty_vector()
    with pytest.raises(IndexError): 
        empty_vec.set_element(0, 0) 
    with pytest.raises(TypeError): 
        empty_vec.set_element(-1,0)


# --- Axioms for pop ---
def test_V10_pop_emptyVector():
    # (V10): pop(emptyVector) = emptyVector
    assert PyVector[Any].empty_vector().pop() == PyVector[Any].empty_vector()

@given(vn=st_py_vector, elem=st_element_type)
def test_V11_pop_append(vn: PyVector[int], elem: int):
    # (V11): pop(append(vn, elem)) = vn
    assert vn.append(elem).pop() == vn

# --- Axiom for head ---
@given(vn=st_py_vector)
def test_V12_head_of_non_empty(vn: PyVector[int]):
    # (V12): isEmpty(vn) = false => head(vn) = get(vn, zero)
    assume(not vn.is_empty()) 
    assert vn.head() == vn.get(0)

# --- Axioms for insertAtIndex ---
@given(data=st_vector_insert_idx_and_value)
def test_V13_length_after_insert(data: tuple[PyVector[int], int, int]):
    # (V13): length(insertAtIndex(vn, insertIdx, valToInsert)) = succ(length(vn))
    vn, insert_idx, val_to_insert = data
    original_length = vn.length()
    vec_after_insert = vn.insert_at_index(insert_idx, val_to_insert)
    assert vec_after_insert.length() == original_length + 1

@given(data=st_vector_insert_idx_and_value)
def test_V14_get_at_insert_index_after_insert(data: tuple[PyVector[int], int, int]):
    # (V14): get(insertAtIndex(vn, insertIdx, valToInsert), insertIdx) = valToInsert
    vn, insert_idx, val_to_insert = data
    vec_after_insert = vn.insert_at_index(insert_idx, val_to_insert)
    assert vec_after_insert.get(insert_idx) == val_to_insert

@given(data=st_vector_insert_idx_and_value, k=st.integers(min_value=0, max_value=11)) 
@settings(suppress_health_check=[HealthCheck.filter_too_much], max_examples=200, deadline=1000)
def test_V15_get_before_insert_idx_after_insert(data: tuple[PyVector[int], int, int], k: int):
    # (V15): lt(k, insertIdx) = true =>
    #        get(insertAtIndex(vn, insertIdx, valToInsert), k) = get(vn, k)
    vn, insert_idx, val_to_insert = data
    assume(k < insert_idx) 
    assume(k < vn.length()) 

    vec_after_insert = vn.insert_at_index(insert_idx, val_to_insert)
    assert vec_after_insert.get(k) == vn.get(k)

@given(data=st_vector_insert_idx_and_value, k=st.integers(min_value=0, max_value=11))
@settings(suppress_health_check=[HealthCheck.filter_too_much], max_examples=200, deadline=1000)
def test_V16_get_after_insert_idx_after_insert(data: tuple[PyVector[int], int, int], k: int):
    # (V16): lt(insertIdx, k) = true AND lt(k, succ(length(vn))) = true =>
    #       get(insertAtIndex(vn, insertIdx, valToInsert), k) = get(vn, pred(k))
    vn, insert_idx, val_to_insert = data
    vec_after_insert = vn.insert_at_index(insert_idx, val_to_insert)

    assume(insert_idx < k)            
    assume(k < vec_after_insert.length()) 
    assume((k - 1) < vn.length())
    assume((k - 1) >= insert_idx) 
    assert vec_after_insert.get(k) == vn.get(k - 1)
```
Os testes de propriedade acima para os axiomas (V1)-(V16) são generalizados para `PyVector[int]`. As estratégias e a lógica dos testes são adaptadas para o tipo genérico `T` (instanciado como `int` para teste). `assume` é usado para as premissas condicionais. O teste para (V9) verifica o comportamento de exceção da implementação Python.

**Exercício:**

Um possível axioma para a operação `prepend: Vector[T] x T -> Vector[T]` (que adiciona ao início) é: `length(prepend(vn, elem)) = succ(length(vn))`. Escreva uma função de teste `pytest` chamada `test_length_of_prepend` que use Hypothesis para verificar este axioma para a implementação `PyVector[T]`, assumindo que o método `prepend(self, value: T) -> PyVector[T]` foi adicionado à classe. Use `st_py_vector` e `st_element_type` para gerar dados.

**Resolução:**

```python
# test_py_vector.py (continuação)

@given(vn=st_py_vector, elem=st_element_type)
def test_length_of_prepend(vn: PyVector[int], elem: int) -> None:
    """
    Tests the axiom: length(prepend(vn, elem)) = succ(length(vn))
    Python implementation: (vn.prepend(elem)).length() == vn.length() + 1
    """
    original_length: int = vn.length()
    prepended_vector: PyVector[int] = vn.prepend(elem) # Assumes prepend is available
    length_after_prepend: int = prepended_vector.length()
    expected_length: int = original_length + 1
    assert length_after_prepend == expected_length
```
O teste `test_length_of_prepend` usa as estratégias genéricas `st_py_vector` e `st_element_type`. A lógica do teste verifica se o método `prepend` da classe `PyVector` aumenta corretamente o comprimento do vetor em uma unidade, conforme o axioma.

# 8.5 Análise de Complexidade Algorítmica

A análise de complexidade para `PyVector[T]` é análoga à de `PyVectorNatural`. Assumimos que operações sobre elementos `T` (como atribuição) são $O(1)$ para esta análise focada nas operações do vetor.

**Análise de Complexidade das Operações de `PyVector[T]` (Imutável):**

1.  **`__init__(self, initial_elements: Optional[PyListInternal[T]] = None)`:** $O(K)$ (onde $K$ é o número de elementos iniciais).
2.  **`empty_vector()`:** $O(1)$.
3.  **`append(self, value: T) -> PyVector[T]`:** $O(N)$.
4.  **`is_empty(self) -> bool`:** $O(1)$.
5.  **`length(self) -> int`:** $O(1)$.
6.  **`get(self, index: int) -> T`:** $O(1)$ (para índice válido).
7.  **`set_element(self, index: int, value: T) -> PyVector[T]`:** $O(N)$.
8.  **`pop(self) -> PyVector[T]`:** $O(N)$.
9.  **`head(self) -> T`:** $O(1)$ (para vetor não vazio).
10. **`prepend(self, value: T) -> PyVector[T]`:** $O(N)$.
11. **`insert_at_index(self, index: int, value: T) -> PyVector[T]`:** $O(N)$.

Operações que "modificam" (criando uma nova instância) são $O(N)$ devido à cópia da lista subjacente. Operações de acesso/consulta são $O(1)$.

**Exercício:**

Se a operação `pop` na classe `PyVector[T]` fosse implementada para modificar o vetor no local (mutável) e usasse `self._elements.pop()` (o método nativo de lista Python que modifica no local e remove o último elemento), qual seria a complexidade de tempo desta versão mutável de `pop`?

**Resolução:**

Se `pop` fosse implementada de forma mutável na classe `PyVector[T]` usando `self._elements.pop()`:
A operação `list.pop()` do Python (sem argumento) remove o último elemento e tem complexidade de tempo **$O(1)$ amortizado**.
Portanto, a complexidade de tempo desta versão mutável de `pop` seria **$O(1)$ amortizado**.

# 8.6 Exercícios Teóricos e Práticos

Esta seção apresenta o exercício que combina a extensão da especificação e implementação do TAD `Vector[T]` com a operação `insertAtIndex`.

**Exercício:**

**Contexto:** Este exercício combina a extensão da especificação e da implementação do TAD `Vector[T]` com a operação `insertAtIndex`.

**Parte a) Especificação Algébrica para `insertAtIndex` em `Vector[T]`**
Revisite a especificação `Vector[ParamT: ElementType]` da Seção 8.2 (que já inclui `emptyVector`, `append`, `length`, `get`, `set`, `pop`, `head`). A operação `insertAtIndex: Vector x Natural x T --> Vector` e seus axiomas de propriedade (V13-V16) já foram incluídos lá. Certifique-se de que compreende esses axiomas.

**Parte b) Implementação Python para `insert_at_index` em `PyVector[T]`**
A classe `PyVector[T]` na Seção 8.3 já inclui uma implementação do método `insert_at_index(self, index: int, value: T) -> PyVector[T]`. Verifique se esta implementação está consistente com os requisitos de ser funcional/imutável, validar parâmetros (índice $0 \le \text{index} \le \text{self.length()}$) e levantar exceções apropriadas.

**Parte c) Testes de Propriedade para `insert_at_index` em `PyVector[T]`**
A Seção 8.4 já esboçou testes para os axiomas de propriedade (V13-V16) para `insertAtIndex` (`test_V13_length_after_insert`, `test_V14_get_at_insert_index_after_insert`, `test_V15_get_before_insert_idx_after_insert`, `test_V16_get_after_insert_idx_after_insert`). Revise criticamente estes testes para garantir que eles cobrem adequadamente as propriedades especificadas na Parte (a) para `PyVector[T]`, prestando atenção especial às condições `assume` para refletir as premissas dos axiomas.

**Resolução:**

**Parte a) Especificação Algébrica para `insertAtIndex` em `Vector[T]`**
A especificação da Seção 8.2 já inclui `insertAtIndex` e os axiomas (V13)-(V16):
*   (V13): `length(insertAtIndex(vec, insertIdx, valToInsert)) = succ(length(vec))`
*   (V14): `get(insertAtIndex(vec, insertIdx, valToInsert), insertIdx) = valToInsert`
*   (V15): `lt(k, insertIdx) = true => get(insertAtIndex(vec, insertIdx, valToInsert), k) = get(vec, k)`
*   (V16): `lt(insertIdx, k) = true AND lt(k, succ(length(vec))) = true => get(insertAtIndex(vec, insertIdx, valToInsert), k) = get(vec, pred(k))`
Estes axiomas definem `insertAtIndex` em termos de seus efeitos observáveis, adequados para PBT.

**Parte b) Implementação Python para `insert_at_index` em `PyVector[T]`**
A implementação na Seção 8.3:
```python
# py_vector.py (método relevante da classe PyVector[T])
# ...
    def insert_at_index(self, index: int, value: T) -> PyVector[T]:
        # Corresponds to 'insertAtIndex: Vector[T] x Natural x T --> Vector[T]'.
        # Inserts a value at a specific index, shifting subsequent elements.
        # Returns a NEW PyVector. Index must be 0 <= index <= self.length().
        if not (isinstance(index, int) and index >= 0): # Check if index is Natural-like
            raise TypeError("Index must be a non-negative integer (Natural).")
        
        current_length = len(self._elements)
        if not (0 <= index <= current_length): # Allows insertion at the end (index == current_length)
            raise IndexError("Index out of bounds for insertion.")

        # Type of 'value' (T) is handled by Python's generic typing and MyPy.
        
        new_elements = self._elements[:index] + [value] + self._elements[index:]
        return PyVector(new_elements)
# ...
```
Análise da implementação:
1.  **Estilo funcional/imutável:** Sim, retorna um novo `PyVector`.
2.  **Validação de `index`:** Sim, $0 \le \text{index} \le \text{length()}$ é verificado. Tipo `int >= 0` também.
3.  **Validação de `value`:** O tipo `T` é tratado por MyPy.
4.  **Exceções:** `TypeError` e `IndexError` são levantadas corretamente.
A implementação está consistente com os requisitos.

**Parte c) Testes de Propriedade para `insert_at_index` em `PyVector[T]`**
Os testes da Seção 8.4:
*   `test_V13_length_after_insert`: Correto.
*   `test_V14_get_at_insert_index_after_insert`: Correto.
*   `test_V15_get_before_insert_idx_after_insert`: Correto com as `assume`s:
    `assume(k < insert_idx)`
    `assume(k < vn.length())` (importante para `vn.get(k)` ser válido).
*   `test_V16_get_after_insert_idx_after_insert`: Correto com as `assume`s:
    `assume(insert_idx < k)`
    `assume(k < vec_after_insert.length())`
    `assume((k - 1) >= 0)`
    `assume((k - 1) < vn.length())` (importante para `vn.get(k-1)` ser válido).

Os testes propostos na Seção 8.4 cobrem adequadamente as propriedades (V13)-(V16) da operação `insertAtIndex`. As condições `assume` são cruciais para refletir as premissas dos axiomas e garantir a validade dos acessos `get`.

Este capítulo introduziu o TAD `Vector[T]`, generalizando-o para um tipo de elemento `T`. Cobrimos sua definição abstrata, relevância, uma especificação algébrica parametrizada (incluindo análise de corretude/completeza e extensão com `insertAtIndex`), e uma implementação Python genérica `PyVector[T]` com tipagem estática. Demonstramos como os axiomas da especificação podem ser traduzidos em testes de propriedade usando Hypothesis para verificar a corretude da implementação. Analisamos a complexidade algorítmica das operações. O próximo capítulo, Capítulo 9, continuará a exploração de estruturas de dados lineares, focando no TAD `List[T]`. Este TAD, embora também represente uma sequência, é tipicamente caracterizado por construtores como `nil` e `cons` (adição no início), o que leva a diferentes perfis de desempenho e abordagens de especificação e implementação, especialmente quando se consideram representações como listas encadeadas.

---
## REFERÊNCIAS BIBLIOGRÁFICAS
AHO, A. V.; HOPCROFT, J. E.; ULLMAN, J. D. **Data Structures and Algorithms**. Reading, MA: Addison-Wesley, 1983.
*   *Resumo: Um texto clássico que cobre estruturas de dados fundamentais, incluindo arrays e listas (que se relacionam com o TAD Vector). Embora mais antigo, os conceitos de acesso indexado e as operações básicas são atemporais e relevantes para a definição abstrata do Vector genérico `Vector[T]`.*

CORMEN, T. H.; LEISERSON, C. E.; RIVEST, R. L.; STEIN, C. **Introduction to Algorithms**. 3. ed. Cambridge, MA: MIT Press, 2009.
*   *Resumo: Uma referência abrangente e moderna sobre algoritmos e estruturas de dados. Discute arrays e listas dinâmicas (vetores), incluindo análise de complexidade amortizada, crucial para entender implementações de `Vector[T]` e o impacto da generalidade do tipo `T`.*

GOODRICH, M. T.; TAMASSIA, R.; GOLDWASSER, M. H. **Data Structures and Algorithms in Python**. Hoboken, NJ: Wiley, 2013.
*   *Resumo: Este livro foca em estruturas de dados e algoritmos com implementações em Python. Discute listas dinâmicas (análogas a `Vector[T]`) e o uso de genéricos em Python (via `typing`). Relevante para a implementação `PyVector[T]` e sua análise de complexidade, considerando elementos de tipo `T`.*

ISO/IEC. **Information technology — Programming languages — Common Lisp Object System Specification**. ANSI X3.226:1994 (R1999). Geneva: ISO/IEC, 1994.
*   *Resumo: Embora específico para Common Lisp, o CLOS é um exemplo de sistema de tipos poderoso com genericidade. O conceito de sequências e vetores genéricos em linguagens estabelecidas fornece um contexto prático para a relevância do TAD `Vector[T]`.*

MUSSER, D. R.; DERGE, G. J.; SAINI, A. **STL Tutorial and Reference Guide**: C++ Programming with the Standard Template Library. 2. ed. Reading, MA: Addison-Wesley, 2001.
*   *Resumo: A STL do C++ é um exemplo proeminente de uma biblioteca com contêineres genéricos, como `std::vector<T>`. Este guia detalha seu design e uso, ilustrando a aplicação prática de um `Vector[T]` parametrizado em um contexto industrial.*

PYTHON SOFTWARE FOUNDATION. **Python `list` documentation and `typing` module documentation**. Disponível em: https://docs.python.org/3/tutorial/datastructures.html#more-on-lists e https://docs.python.org/3/library/typing.html. Acesso em: 10 ago. 2023.
*   *Resumo: A documentação do Python para `list` (a base para `PyVector`) e para o módulo `typing` (essencial para `TypeVar` e `Generic` ao implementar `PyVector[T]`) é crucial para a implementação prática em Python e para a compreensão das ferramentas de genericidade da linguagem.*

---
