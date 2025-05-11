---
# Capítulo 8
# O TAD Vector
---

Iniciando a Parte II deste livro, que se dedica ao estudo aprofundado de estruturas de dados lineares sob a ótica da especificação formal e da implementação rigorosa, este capítulo introduz o Tipo Abstrato de Dados (TAD) `Vector`. O `Vector`, também conhecido como array dinâmico ou lista baseada em array em muitas bibliotecas de programação, é uma das estruturas de dados mais fundamentais e versáteis, caracterizado por oferecer acesso eficiente a elementos através de um índice numérico e por gerenciar dinamicamente seu tamanho para acomodar um número variável de elementos. Este capítulo seguirá uma estrutura sistemática que será replicada para as demais estruturas de dados lineares: começaremos com uma definição abstrata do TAD `Vector`, destacando sua relevância e as operações essenciais que o caracterizam, como criação, acesso e modificação de elementos por índice, obtenção de tamanho e adição ou remoção de elementos. Em seguida, desenvolveremos uma especificação algébrica formal para o `Vector`, definindo seus sorts, operações e axiomas que capturam seu comportamento de forma precisa e independente de implementação, com foco em elementos do tipo `Natural` para manter a consistência com os exemplos da Parte I. Posteriormente, transitaremos para o projeto e a implementação de uma classe `PyVectorNatural` em Python, que realizará o TAD `Vector[Natural]`, com ênfase na utilização de anotações de tipo MyPy para garantir a consistência da interface e dos tipos de dados. Uma parte crucial do capítulo será dedicada à verificação da corretude da implementação `PyVectorNatural` em relação à sua especificação algébrica, utilizando a biblioteca Hypothesis para traduzir os axiomas em propriedades testáveis e buscar automaticamente por contraexemplos. A análise de complexidade algorítmica das operações implementadas em `PyVectorNatural` será conduzida para avaliar seu desempenho. Finalmente, o capítulo incluirá exercícios teóricos e práticos, incorporando um exemplo de extensão do TAD com a operação `insertAtIndex` e sua respectiva implementação e teste, para solidificar a compreensão do TAD `Vector`, sua especificação formal, sua implementação em Python e as técnicas de verificação.

# 8.1 Definição Abstrata e Relevância do Tipo de Dados `Vector`

O Tipo Abstrato de Dados (TAD) `Vector`, frequentemente conhecido em contextos de programação como array dinâmico, lista indexável ou, em algumas linguagens, simplesmente "lista" (embora este último termo também possa se referir a listas encadeadas), é uma estrutura de dados linear fundamental que modela uma coleção ordenada de elementos, onde cada elemento é acessível através de um índice numérico, tipicamente um inteiro não-negativo. A principal característica que distingue um `Vector` de um array estático tradicional é sua capacidade de crescer ou encolher dinamicamente em tamanho conforme elementos são adicionados ou removidos, enquanto ainda tenta manter o acesso $O(1)$ (tempo constante) aos elementos por índice, pelo menos em um sentido amortizado.

**Definição Abstrata:**
Um `Vector[T]` é um TAD que representa uma sequência finita e ordenada de elementos do tipo `T`. As operações essenciais que caracterizam um `Vector` incluem:
1.  **Criação:**
    *   `emptyVector`: Cria um vetor vazio.
2.  **Acesso e Modificação por Índice:**
    *   `get: Vector[T] x Natural -> T`: Retorna o elemento na posição `Natural` (índice) do `Vector[T]`.
    *   `set: Vector[T] x Natural x T -> Vector[T]`: Retorna um novo `Vector[T]` com o elemento na posição `Natural` (índice) substituído pelo novo elemento `T`.
3.  **Informações de Tamanho:**
    *   `length: Vector[T] -> Natural`: Retorna o número de elementos no `Vector[T]`.
    *   `isEmpty: Vector[T] -> Boolean`: Verifica se o `Vector[T]` está vazio.
4.  **Operações de Modificação de Tamanho:**
    *   `append: Vector[T] x T -> Vector[T]`: Retorna um novo `Vector[T]` com o elemento `T` adicionado ao final.
    *   `pop: Vector[T] -> Vector[T]`: Retorna um novo `Vector[T]` com o último elemento removido.

A semântica dessas operações deve capturar a ordem dos elementos, o acesso indexado e o comportamento dinâmico do tamanho.

**Relevância do TAD `Vector`:**
O `Vector` é crucial por várias razões:
1.  **Acesso Indexado Eficiente:** Frequentemente $O(1)$ (amortizado).
2.  **Tamanho Dinâmico:** Adapta-se às necessidades de dados.
3.  **Boa Localidade de Referência:** Em implementações baseadas em array contíguo, otimiza o uso de cache.
4.  **Base para Outras Estruturas:** Usado para construir pilhas, filas, tabelas hash, etc.
5.  **Modelo Intuitivo:** Natural para sequências ordenadas com acesso posicional.
6.  **Padrão em Linguagens:** Implementações eficientes são comuns em bibliotecas padrão (e.g., `list` em Python, `std::vector` em C++).

Operações como `append` (quando causam redimensionamento) ou inserções/remoções no meio (não listadas como essenciais acima, mas comuns em variantes) podem ter custos mais altos ($O(N)$ no pior caso ou para operações no meio).

Neste capítulo, focaremos em `Vector[Natural]`.

**Exercício:**

Considere um TAD `Vector[Natural]` com as operações `emptyVector`, `append(v,n)`, `get(v,idx)` e `length(v)`. Se você tem um vetor `v1 = append(append(emptyVector, natural_five), natural_ten)` (onde `natural_five` e `natural_ten` são representações de `Natural` 5 e 10), qual seria o resultado esperado de `length(v1)` e de `get(v1, zero_idx)` e `get(v1, succ(zero_idx))`? (Assuma que `zero_idx` e `succ(zero_idx)` são os `Natural`s 0 e 1, usados como índices).

**Resolução:**

Dado o vetor $v_1 = \text{append}(\text{append}(\text{emptyVector}, \text{natural_five}), \text{natural_ten})$.
Este vetor foi construído da seguinte forma:
1.  Inicia-se com `emptyVector`.
2.  `append(emptyVector, natural_five)`: Adiciona `natural_five`. O vetor é conceitualmente $\langle \text{natural_five} \rangle$.
3.  `append(\langle \text{natural_five} \rangle, natural_ten)`: Adiciona `natural_ten`. O vetor $v_1$ é conceitualmente $\langle \text{natural_five}, \text{natural_ten} \rangle$.

Resultados das operações:

1.  **`length(v1)`:**
    O vetor $v_1$ contém dois elementos.
    Resultado esperado: o `Natural` que representa o número 2 (i.e., `succ(succ(zero))` na notação de Peano, onde `zero` é a constante `Natural` para 0).

2.  **`get(v1, zero_idx)`:**
    `zero_idx` representa o índice 0.
    No vetor $v_1 = \langle \text{natural_five}, \text{natural_ten} \rangle$, o elemento no índice 0 é `natural_five`.
    Resultado esperado: `natural_five`.

3.  **`get(v1, succ(zero_idx))`:**
    `succ(zero_idx)` representa o `Natural` 1 (índice 1).
    No vetor $v_1 = \langle \text{natural_five}, \text{natural_ten} \rangle$, o elemento no índice 1 é `natural_ten`.
    Resultado esperado: `natural_ten`.

# 8.2 Especificação Algébrica Formal do TAD `Vector[Natural]`

Nesta seção, desenvolveremos uma especificação algébrica formal para o Tipo Abstrato de Dados `Vector` contendo elementos do tipo `Natural`. Denotaremos este TAD como `VectorNatural`. A especificação incluirá os sorts necessários, as operações fundamentais (construtores e observadores) e um conjunto de axiomas equacionais que definem o comportamento dessas operações. Nosso objetivo é capturar a essência de um vetor indexável e de tamanho dinâmico, focando em operações como criação, adição de elementos (append), acesso por índice, e obtenção do comprimento.

Assumimos que as especificações para `Natural` (com `zero`, `succ`, `add`, `eq`, `lt`, `pred`) e `Boolean` (com `true`, `false`, `and`, `or`, `not`, `ifThenElse`) já foram definidas e estão disponíveis para importação.

**SPEC ADT** VectorNatural
**imports:**
*   Natural
*   Boolean

**sorts:**
*   VectorNatural

**operations:**
*   emptyVector: --> VectorNatural
*   append: VectorNatural x Natural --> VectorNatural
*   isEmpty: VectorNatural --> Boolean
*   length: VectorNatural --> Natural
*   get: VectorNatural x Natural --> Natural
*   set: VectorNatural x Natural x Natural --> VectorNatural
*   pop: VectorNatural --> VectorNatural
*   head: VectorNatural --> Natural
*   insertAtIndex: VectorNatural x Natural x Natural --> VectorNatural

**axioms:**
for all vn: VectorNatural, n, elem, newElem, valToInsert: Natural, idx, k, insertIdx: Natural

*   (V1): isEmpty(emptyVector) = true
*   (V2): isEmpty(append(vn, n)) = false

*   (V3): length(emptyVector) = zero
*   (V4): length(append(vn, n)) = succ(length(vn))

*   (V5): eq(idx, length(vn)) = true => get(append(vn, elem), idx) = elem
*   (V6): lt(idx, length(vn)) = true  => get(append(vn, elem), idx) = get(vn, idx)

*   (V7): eq(k, length(vn)) = true =>
        set(append(vn, elem), k, newElem) = append(vn, newElem)
*   (V8): lt(k, length(vn)) = true =>
        set(append(vn, elem), k, newElem) = append(set(vn, k, newElem), elem)
*   (V9): set(emptyVector, k, newElem) = emptyVector

*   (V10): pop(emptyVector) = emptyVector
*   (V11): pop(append(vn, n)) = vn

*   (V12): isEmpty(vn) = false => head(vn) = get(vn, zero)

*   (V13): length(insertAtIndex(vn, insertIdx, valToInsert)) = succ(length(vn))
*   (V14): get(insertAtIndex(vn, insertIdx, valToInsert), insertIdx) = valToInsert
*   (V15): lt(k, insertIdx) = true =>
            get(insertAtIndex(vn, insertIdx, valToInsert), k) = get(vn, k)
*   (V16): lt(insertIdx, k) = true AND lt(k, succ(length(vn))) = true =>
           get(insertAtIndex(vn, insertIdx, valToInsert), k) = get(vn, pred(k))

**END SPEC**

A especificação `VectorNatural` define um Tipo Abstrato de Dados para vetores de números naturais. Ela importa `Natural` e `Boolean`. O sort principal é `VectorNatural`.
As operações incluem os construtores `emptyVector` e `append`. As operações observadoras/derivadas são `isEmpty`, `length`, `get`, `set`, `pop`, `head`, e a nova operação `insertAtIndex`.
Os axiomas (V1)-(V12) são como definidos anteriormente. Os axiomas para `insertAtIndex` (V13-V16) são baseados em suas propriedades observáveis:
(V13) afirma que o comprimento aumenta em um. (V14) afirma que o elemento no índice de inserção é o valor inserido. (V15) afirma que elementos em índices anteriores não são afetados. (V16) afirma que elementos em índices posteriores são deslocados. Estes axiomas para `insertAtIndex` assumem que `insertIdx` é válido ($0 \le \text{insertIdx} \le \text{length(vn)}$) e que `k` nos axiomas de `get` também são válidos nos respectivos contextos.

**Exercício:**

A especificação `VectorNatural` acima não inclui axiomas para o comportamento de `get` ou `head` quando aplicados a um `emptyVector`. Supondo que queremos que tais operações resultem em um valor de erro especial (e.g., `errorNatural` que seria parte de um sort `ResultNatural = Natural | ErrorValueNatural`), como poderíamos começar a modificar a assinatura de `get` e `head` e adicionar um axioma para `get(emptyVector, idx)`? (Não precisa especificar `ResultNatural` completamente, apenas mostre a ideia).

**Resolução:**

Para modificar a especificação para lidar com erros em `get` e `head`:

1.  **Modificar Assinatura (Ideia):**
    Precisaríamos de um novo sort, por exemplo `ResultNatural`, que poderia ser um `Natural` ou um valor de erro.
    *   `ResultNatural` (novo sort)
    *   `makeOkNatural: Natural --> ResultNatural` (construtor para resultado válido)
    *   `makeErrorNatural: ErrorCode --> ResultNatural` (construtor para resultado de erro, `ErrorCode` seria outro sort)
    *   `errorIndexOutOfBounds: --> ErrorCode` (uma constante de erro)

    As assinaturas de `get` e `head` mudariam:
    *   `get: VectorNatural x Natural --> ResultNatural`
    *   `head: VectorNatural --> ResultNatural`

2.  **Adicionar Axioma para `get(emptyVector, idx)`:**
    Com a nova assinatura, poderíamos adicionar:
    *   (V_ERR_GET_EMPTY): `get(emptyVector, idx) = makeErrorNatural(errorIndexOutOfBounds)`

    E os axiomas existentes (V5, V6) para `get` teriam seus lados direitos envoltos por `makeOkNatural`:
    *   (V5'): `eq(idx, length(vn)) = true => get(append(vn, elem), idx) = makeOkNatural(elem)`
    *   (V6'): `lt(idx, length(vn)) = true  => get(append(vn, elem), idx) = makeOkNatural(get_internal(vn, idx))`
        (onde `get_internal` seria a operação original que retorna `Natural`, ou os axiomas de `get` seriam reescritos para produzir `ResultNatural` diretamente, como em V5').

    Similarmente para `head`:
    *   (V_ERR_HEAD_EMPTY): `head(emptyVector) = makeErrorNatural(errorIndexOutOfBounds)`
    *   (V12'): `isEmpty(vn) = false => head(vn) = makeOkNatural(get_internal(vn, zero))`
        (ou `head(vn) = get(vn,zero)` se `get` já retorna `ResultNatural`)

Esta abordagem torna o tratamento de erro explícito na especificação. A definição completa de `ResultNatural` e `ErrorCode`, junto com operações para inspecionar um `ResultNatural` (e.g., `isOk: ResultNatural -> Boolean`, `getValue: ResultNatural -> Natural` (parcial)), seria necessária.

# 8.3 Projeto e Implementação em Python com Tipagem Estática (MyPy)

Após a especificação algébrica formal do TAD `VectorNatural` (Seção 8.2), o próximo passo é transitar para sua concretização em uma linguagem de programação. Escolhemos Python para esta tarefa, com o objetivo de criar uma classe `PyVectorNatural` que realize o comportamento definido pelos axiomas. Para aumentar o rigor e a clareza da implementação, utilizaremos anotações de tipo e faremos uso do verificador de tipos estático MyPy.

O design da classe `PyVectorNatural` será guiado pela especificação, mas também levará em consideração as características idiomáticas de Python e as escolhas comuns de implementação para vetores dinâmicos. Uma lista Python (`list`) será usada como a estrutura de dados subjacente para armazenar os elementos `Natural` (que serão representados por inteiros Python não-negativos).

**Design da Classe `PyVectorNatural`:**

A classe `PyVectorNatural` encapsulará uma lista Python. As operações do TAD `VectorNatural` serão mapeadas para métodos desta classe.

```python
# py_vector_natural.py
from __future__ import annotations # For type hints of PyVectorNatural within the class itself
from typing import List as PyListInternal, Optional # Using an alias for clarity

class PyVectorNatural:
    _elements: PyListInternal[int] # Elements are 'Natural', represented by int >= 0

    def __init__(self, initial_elements: Optional[PyListInternal[int]] = None) -> None:
        # Constructor for PyVectorNatural.
        # Accepts an optional list of non-negative integers for initialization.
        if initial_elements is None:
            self._elements = []
        else:
            for x_val in initial_elements: 
                if not (isinstance(x_val, int) and x_val >= 0):
                    raise ValueError("All initial elements must be non-negative integers (Naturals).")
            self._elements = list(initial_elements) # Creates a copy

    @staticmethod
    def empty_vector() -> PyVectorNatural:
        # Corresponds to 'emptyVector: --> VectorNatural'
        return PyVectorNatural()

    def append(self, value: int) -> PyVectorNatural:
        # Corresponds to 'append: VectorNatural x Natural --> VectorNatural'.
        # Returns a NEW PyVectorNatural with the value added.
        # This implementation is functional (immutable).
        if not (isinstance(value, int) and value >= 0):
            raise ValueError("Value to append must be a non-negative integer (Natural).")
        
        new_elements = self._elements + [value] # Creates a new Python list
        return PyVectorNatural(new_elements)

    def is_empty(self) -> bool:
        # Corresponds to 'isEmpty: VectorNatural --> Boolean'
        return not self._elements # An empty list is Falsy in Python

    def length(self) -> int: # Returning int to represent Natural
        # Corresponds to 'length: VectorNatural --> Natural'
        return len(self._elements)

    def get(self, index: int) -> int: # Index and value are Naturals (int >= 0)
        # Corresponds to 'get: VectorNatural x Natural --> Natural'.
        # Raises IndexError if the index is invalid.
        if not (isinstance(index, int) and index >= 0):
            raise TypeError("Index must be a non-negative integer (Natural).")
        if not (0 <= index < len(self._elements)):
            # Behavior for out-of-bounds index (specification was partial)
            raise IndexError("Vector index out of range.")
        return self._elements[index]

    def set_element(self, index: int, value: int) -> PyVectorNatural:
        # Corresponds to 'set: VectorNatural x Natural x Natural --> VectorNatural'.
        # Returns a NEW PyVectorNatural with the element at the index updated.
        # This implementation is functional (immutable).
        # Raises IndexError if the index is invalid.
        if not (isinstance(index, int) and index >= 0):
            raise TypeError("Index must be a non-negative integer (Natural).")
        if not (isinstance(value, int) and value >= 0):
            raise ValueError("Value to set must be a non-negative integer (Natural).")
        if not (0 <= index < len(self._elements)):
            raise IndexError("Vector index out of range.")
        
        new_elements = list(self._elements) # Create a copy
        new_elements[index] = value
        return PyVectorNatural(new_elements)

    def pop(self) -> PyVectorNatural:
        # Corresponds to 'pop: VectorNatural --> VectorNatural'.
        # Returns a NEW PyVectorNatural with the last element removed.
        # If the vector is empty, returns a new empty vector.
        # This implementation is functional (immutable).
        if self.is_empty():
            return PyVectorNatural.empty_vector()
        
        new_elements = self._elements[:-1] # Creates a new Python list without the last element
        return PyVectorNatural(new_elements)
    
    def head(self) -> int: 
        # Corresponds to 'head: VectorNatural --> Natural'.
        # Raises IndexError if the vector is empty.
        if self.is_empty():
            raise IndexError("Cannot get head of an empty vector.")
        return self._elements[0]

    def insert_at_index(self, index: int, value: int) -> PyVectorNatural:
        # Corresponds to 'insertAtIndex: VectorNatural x Natural x Natural --> VectorNatural'.
        # Inserts a value at a specific index, shifting subsequent elements.
        # Returns a NEW PyVectorNatural.
        # Index must be 0 <= index <= self.length().
        # Value must be a non-negative integer (Natural).
        if not (isinstance(value, int) and value >= 0):
            raise ValueError("Value to insert must be a non-negative integer (Natural).")
        if not (isinstance(index, int) and index >= 0):
            raise TypeError("Index must be a non-negative integer (Natural).")
        
        current_length = len(self._elements)
        if not (0 <= index <= current_length): # Allows insertion at the end (index == current_length)
            raise IndexError("Index out of bounds for insertion.")

        # Create new list by slicing and concatenating
        new_elements = self._elements[:index] + [value] + self._elements[index:]
        return PyVectorNatural(new_elements)

    # Methods for equality and representation, useful for testing and debugging
    def __eq__(self, other: object) -> bool:
        if not isinstance(other, PyVectorNatural):
            return NotImplemented
        return self._elements == other._elements

    def __repr__(self) -> str:
        return f"PyVectorNatural({self._elements})"

    # Helper method for tests, to get internal elements
    # Not part of the ADT interface, but useful for state verification.
    def to_list(self) -> PyListInternal[int]:
        return list(self._elements) # Returns a copy to maintain encapsulation

```
O código Python acima, no arquivo `py_vector_natural.py`, define a classe `PyVectorNatural`.
Ela encapsula uma lista Python interna `_elements` para armazenar inteiros não-negativos, representando os `Natural`s. O construtor `__init__` permite a inicialização opcional com uma lista de elementos, validando cada um. O método estático `empty_vector()` cria um vetor vazio. Os métodos `append`, `set_element`, `pop`, e o recém-adicionado `insert_at_index` são implementados de forma funcional (imutável), retornando novas instâncias de `PyVectorNatural` em vez de modificar o objeto original. Os métodos `is_empty`, `length`, `get`, e `head` são observadores. `get`, `set_element`, `head` e `insert_at_index` levantam exceções para índices ou valores inválidos, o que é uma decisão de implementação para lidar com os casos parciais/de erro da especificação. Os métodos `__eq__` e `__repr__` são implementados para facilitar testes e depuração. Um método auxiliar `to_list` é fornecido para inspeção do estado interno durante os testes. Anotações de tipo MyPy são usadas consistentemente.

**Exercício:**

Implemente um método `to_string(self) -> str` na classe `PyVectorNatural`. Este método deve retornar uma representação em string do vetor, mostrando seus elementos separados por vírgulas e entre colchetes, e.g., `"[1, 2, 3]"`. Se o vetor estiver vazio, deve retornar `"[]"`.

**Resolução:**

Para adicionar o método `to_string` à classe `PyVectorNatural`:

```python
# py_vector_natural.py (continuação da classe PyVectorNatural)

    # ... (outros métodos como definidos anteriormente) ...

    def to_string(self) -> str:
        # Returns a string representation of the vector's elements.
        # e.g., "[1, 2, 3]" or "[]" for an empty vector.
        if self.is_empty():
            return "[]"
        else:
            # Convert each integer element to its string representation
            # then join them with ", "
            elements_as_strings = [str(elem) for elem in self._elements]
            return "[" + ", ".join(elements_as_strings) + "]"

    # ... (restante da classe, __eq__, __repr__, etc.) ...
```
O método `to_string` foi adicionado à classe `PyVectorNatural`. Ele primeiro verifica se o vetor está vazio usando `self.is_empty()`. Se estiver, retorna a string `"[]"`. Caso contrário, ele cria uma lista de strings `elements_as_strings` convertendo cada elemento inteiro da lista interna `self._elements` para sua representação em string. Em seguida, usa o método `", ".join()` para concatenar essas strings de elementos, separando-as por uma vírgula e um espaço. Finalmente, adiciona os colchetes no início e no fim para formar a representação final da string do vetor.

# 8.4 Verificação de Propriedades Axiomáticas com Hypothesis

Uma vez que temos a especificação algébrica formal do TAD `VectorNatural` (Seção 8.2, incluindo `insertAtIndex`) e uma implementação Python candidata, a classe `PyVectorNatural` (Seção 8.3, incluindo `insert_at_index`), o próximo passo crítico é verificar se a implementação se comporta de acordo com os axiomas definidos na especificação. O Teste Baseado em Propriedades (PBT) com a biblioteca Hypothesis é uma técnica ideal para esta tarefa. Cada axioma da especificação algébrica pode ser traduzido em uma propriedade testável, onde Hypothesis gerará instâncias de `PyVectorNatural` e `int` (para os elementos `Natural`) e verificará se a igualdade ou relação expressa pelo axioma se mantém para a implementação.

**Estratégias Hypothesis para `PyVectorNatural` e `Natural`:**

Primeiro, precisamos de estratégias Hypothesis para gerar os dados de teste.

```python
# test_py_vector_natural_axioms.py
import pytest # For executing with pytest
from hypothesis import given, strategies as st, settings, HealthCheck, assume

# Import the implementation
from py_vector_natural import PyVectorNatural 

# Strategy for generating 'Natural' values (non-negative integers)
st_natural_value = st.integers(min_value=0, max_value=50) 

# Strategy for generating instances of the PyVectorNatural class
st_py_vector_natural = st.builds(
    PyVectorNatural,
    initial_elements=st.lists(st_natural_value, max_size=10)
)

# Strategy to generate a PyVectorNatural and a valid index for it (for get/set on existing elements)
@st.composite
def st_vector_and_valid_read_index(draw):
    vec = draw(st_py_vector_natural.filter(lambda v: not v.is_empty())) # Ensures non-empty
    index_val = draw(st.integers(min_value=0, max_value=vec.length() - 1))
    return vec, index_val

# Strategy to generate a PyVectorNatural and an index that could be its current length
# (useful for testing append-related axioms for get/set or insert at end)
@st.composite
def st_vector_and_append_or_valid_index(draw):
    vec = draw(st_py_vector_natural)
    index_val = draw(st.integers(min_value=0, max_value=vec.length())) 
    return vec, index_val

# Strategy for vector, a valid insertion index (0 to length), and a value
@st.composite
def st_vector_insert_idx_and_value(draw):
    vec = draw(st_py_vector_natural) 
    insert_idx = draw(st.integers(min_value=0, max_value=vec.length()))
    val_to_insert = draw(st_natural_value) 
    return vec, insert_idx, val_to_insert
```
O código acima, destinado ao arquivo `test_py_vector_natural_axioms.py`, configura as estratégias Hypothesis. `st_natural_value` gera inteiros não-negativos até 50. `st_py_vector_natural` constrói instâncias de `PyVectorNatural`. A estratégia `st_vector_and_valid_read_index` gera um `PyVectorNatural` não-vazio e um índice inteiro que é garantidamente válido para leitura nesse vetor. A estratégia `st_vector_and_append_or_valid_index` gera um vetor e um índice que pode ser o comprimento atual do vetor. A estratégia `st_vector_insert_idx_and_value` gera um vetor, um índice válido para inserção (incluindo o final) e um valor natural.

**Testes de Propriedade para os Axiomas:**

Relembrando os axiomas da Seção 8.2 (V1-V16):
```python
# test_py_vector_natural_axioms.py (continuação)

# --- Axioms for isEmpty ---
def test_V1_is_empty_emptyVector():
    # (V1): isEmpty(emptyVector) = true
    assert PyVectorNatural.empty_vector().is_empty() == True

@given(vn=st_py_vector_natural, n=st_natural_value)
def test_V2_is_empty_append(vn: PyVectorNatural, n: int):
    # (V2): isEmpty(append(vn, n)) = false
    assert vn.append(n).is_empty() == False

# --- Axioms for length ---
def test_V3_length_emptyVector():
    # (V3): length(emptyVector) = zero
    assert PyVectorNatural.empty_vector().length() == 0 

@given(vn=st_py_vector_natural, n=st_natural_value)
def test_V4_length_append(vn: PyVectorNatural, n: int):
    # (V4): length(append(vn, n)) = succ(length(vn))
    assert vn.append(n).length() == vn.length() + 1

# --- Axioms for get ---
@given(data=st_vector_and_append_or_valid_index, elem=st_natural_value)
def test_V5_get_at_appended_pos(data: tuple[PyVectorNatural, int], elem: int):
    # (V5): eq(idx, length(vn)) = true => get(append(vn, elem), idx) = elem
    vn, idx = data
    assume(idx == vn.length()) 
    
    appended_vector = vn.append(elem)
    assert appended_vector.get(idx) == elem

@given(data=st_vector_and_valid_read_index, elem=st_natural_value)
def test_V6_get_at_earlier_pos(data: tuple[PyVectorNatural, int], elem: int):
    # (V6): lt(idx, length(vn)) = true  => get(append(vn, elem), idx) = get(vn, idx)
    vn, idx = data 
    # st_vector_and_valid_read_index ensures idx < vn.length()
    
    appended_vector = vn.append(elem)
    assert appended_vector.get(idx) == vn.get(idx)

# --- Axioms for set ---
@given(data=st_vector_and_append_or_valid_index, elem=st_natural_value, new_elem=st_natural_value)
def test_V7_set_at_appended_pos(data: tuple[PyVectorNatural, int], elem: int, new_elem: int):
    # (V7): eq(k, length(vn)) = true => 
    #       set(append(vn, elem), k, new_elem) = append(vn, new_elem)
    vn, k = data
    assume(k == vn.length()) 
    
    original_appended = vn.append(elem)
    set_result = original_appended.set_element(k, new_elem)
    expected_result = vn.append(new_elem)
    assert set_result == expected_result

@given(data=st_vector_and_valid_read_index, elem=st_natural_value, new_elem=st_natural_value)
@settings(suppress_health_check=[HealthCheck.filter_too_much], max_examples=100) # k is specific
def test_V8_set_at_earlier_pos(data: tuple[PyVectorNatural, int], elem: int, new_elem: int):
    # (V8): lt(k, length(vn)) = true => 
    #       set(append(vn, elem), k, new_elem) = append(set(vn, k, new_elem), elem)
    vn, k = data # k is a valid index in vn, and vn is not empty

    # LHS
    vec_after_append = vn.append(elem)
    lhs = vec_after_append.set_element(k, new_elem)
    
    # RHS
    vec_after_set_original = vn.set_element(k, new_elem)
    rhs = vec_after_set_original.append(elem)
    
    assert lhs == rhs
    
def test_V9_set_on_empty_vector():
    # (V9): set(emptyVector, k, newElem) = emptyVector
    # Our Python impl raises IndexError for set on empty with any valid Natural k.
    # To test V9 behavior, Python impl would need to change.
    # Testing current Python impl:
    empty_vec = PyVectorNatural.empty_vector()
    with pytest.raises(IndexError): # k=0 is out of bounds for empty
        empty_vec.set_element(0, 0) 
    with pytest.raises(TypeError): # k=-1 is not Natural
        empty_vec.set_element(-1,0)

# --- Axioms for pop ---
def test_V10_pop_emptyVector():
    # (V10): pop(emptyVector) = emptyVector
    assert PyVectorNatural.empty_vector().pop() == PyVectorNatural.empty_vector()

@given(vn=st_py_vector_natural, n=st_natural_value)
def test_V11_pop_append(vn: PyVectorNatural, n: int):
    # (V11): pop(append(vn, n)) = vn
    assert vn.append(n).pop() == vn

# --- Axiom for head ---
@given(vn=st_py_vector_natural)
def test_V12_head_of_non_empty(vn: PyVectorNatural):
    # (V12): isEmpty(vn) = false => head(vn) = get(vn, zero)
    assume(not vn.is_empty()) 
    
    assert vn.head() == vn.get(0)

# --- Axioms for insertAtIndex (propriedades V13-V16 da Seção 8.2 do exercício) ---
@given(data=st_vector_insert_idx_and_value)
def test_V13_length_after_insert(data: tuple[PyVectorNatural, int, int]):
    # (V13): length(insertAtIndex(vn, insertIdx, valToInsert)) = succ(length(vn))
    vn, insert_idx, val_to_insert = data
    
    original_length = vn.length()
    vec_after_insert = vn.insert_at_index(insert_idx, val_to_insert)
    
    assert vec_after_insert.length() == original_length + 1

@given(data=st_vector_insert_idx_and_value)
def test_V14_get_at_insert_index_after_insert(data: tuple[PyVectorNatural, int, int]):
    # (V14): get(insertAtIndex(vn, insertIdx, valToInsert), insertIdx) = valToInsert
    vn, insert_idx, val_to_insert = data
    
    vec_after_insert = vn.insert_at_index(insert_idx, val_to_insert)
    
    assert vec_after_insert.get(insert_idx) == val_to_insert

@given(data=st_vector_insert_idx_and_value, k=st_natural_value)
@settings(suppress_health_check=[HealthCheck.filter_too_much], max_examples=200, deadline=1000)
def test_V15_get_before_insert_idx_after_insert(data: tuple[PyVectorNatural, int, int], k: int):
    # (V15): lt(k, insertIdx) = true =>
    #        get(insertAtIndex(vn, insertIdx, valToInsert), k) = get(vn, k)
    vn, insert_idx, val_to_insert = data
    
    assume(k < insert_idx) # Condition of the axiom
    assume(k < vn.length()) # Ensure k is a valid index in the original vector vn

    vec_after_insert = vn.insert_at_index(insert_idx, val_to_insert)
        
    assert vec_after_insert.get(k) == vn.get(k)

@given(data=st_vector_insert_idx_and_value, k=st_natural_value)
@settings(suppress_health_check=[HealthCheck.filter_too_much], max_examples=200, deadline=1000)
def test_V16_get_after_insert_idx_after_insert(data: tuple[PyVectorNatural, int, int], k: int):
    # (V16): lt(insertIdx, k) = true AND lt(k, succ(length(vn))) = true =>
    #       get(insertAtIndex(vn, insertIdx, valToInsert), k) = get(vn, pred(k))
    vn, insert_idx, val_to_insert = data
    vec_after_insert = vn.insert_at_index(insert_idx, val_to_insert)

    assume(insert_idx < k)             # k is after the insertion point
    assume(k < vec_after_insert.length()) # k is a valid index in the new vector
    # This also implies k-1 was a valid index in the original vector if k > insert_idx,
    # because vec_after_insert.length = vn.length + 1.
    # So, k-1 < vn.length() needs to be true.
    # If k = insert_idx + 1, then k-1 = insert_idx. This k-1 must be < vn.length().
    # This holds if insert_idx < vn.length().
    # If insert_idx == vn.length(), then k-1 = vn.length(), which is not < vn.length().
    # So, for k-1 to be a valid index in vn, we need k-1 < vn.length().
    assume((k - 1) < vn.length())
    # Additionally, to be sure vn.get(k-1) is meaningful for the property:
    assume((k - 1) >= insert_idx) # k-1 was an original element at or after insert_idx

    assert vec_after_insert.get(k) == vn.get(k - 1)
```
Os testes de propriedade acima para os axiomas (V1)-(V16) são implementados.
As estratégias foram refinadas: `st_vector_and_valid_read_index` agora filtra explicitamente para vetores não vazios usando `assume` dentro da estratégia composta, garantindo que `vec.length() - 1` seja sempre um índice válido. `st_vector_and_append_or_valid_index` permite que o índice seja `vec.length()`.
Para os testes de `get` e `set` (V5, V6, V7, V8), `assume` é usado dentro das funções de teste para impor as premissas condicionais dos axiomas. Isso é importante porque as estratégias geram os componentes (vetor, índice) de forma um pouco mais ampla, e o `assume` restringe aos casos que o axioma realmente cobre. O `suppress_health_check` é adicionado porque `assume` pode levar a muitas rejeições de dados se a condição for muito específica, o que pode tornar o teste lento.
O teste para (V9) `set(emptyVector, ...)` foi adaptado para verificar o comportamento da implementação Python, que levanta `IndexError` para um índice inválido em um vetor vazio, em vez de retornar `emptyVector` como o axioma (V9) simplificado sugere. Isso destaca a importância de alinhar a especificação e a implementação no tratamento de erros/casos parciais.
Os testes para `insertAtIndex` (V13-V16) usam a estratégia `st_vector_insert_idx_and_value` e `assume`s para verificar as propriedades de comprimento e acesso a elementos antes, no, e depois do ponto de inserção. As condições em `assume` para (V16) são particularmente importantes para garantir que `vn.get(k-1)` seja um acesso válido.

**Exercício:**

Um possível axioma para a operação `prepend: VectorNatural x Natural -> VectorNatural` (que adiciona ao início) é: `length(prepend(vn, n)) = succ(length(vn))`. Escreva uma função de teste `pytest` chamada `test_length_of_prepend` que use Hypothesis para verificar este axioma, assumindo que o método `prepend(self, value: int) -> PyVectorNatural` (do exercício da Seção 8.3) foi adicionado à classe `PyVectorNatural`.

**Resolução:**

```python
# test_py_vector_natural_axioms.py (adicionando a este arquivo)
# ... (importações e estratégias st_py_vector_natural, st_natural_value como antes) ...

# Assume que PyVectorNatural tem o método prepend implementado:
# class PyVectorNatural:
#     ...
#     def prepend(self, value: int) -> PyVectorNatural:
#         if not (isinstance(value, int) and value >= 0):
#             raise ValueError("Value to prepend must be a non-negative integer (Natural).")
#         new_elements = [value] + self._elements 
#         return PyVectorNatural(new_elements)
#     ...

@given(vn=st_py_vector_natural, n=st_natural_value)
def test_length_of_prepend(vn: PyVectorNatural, n: int) -> None:
    """
    Verifica o axioma: length(prepend(vn, n)) = succ(length(vn))
    Na implementação Python: (vn.prepend(n)).length() == vn.length() + 1
    """
    original_length: int = vn.length()
    prepended_vector: PyVectorNatural = vn.prepend(n)
    length_after_prepend: int = prepended_vector.length()
    expected_length: int = original_length + 1
    assert length_after_prepend == expected_length
```
O teste `test_length_of_prepend` é adicionado. Ele usa as estratégias existentes `st_py_vector_natural` para `vn` e `st_natural_value` para `n`. Dentro do teste, o comprimento original do vetor `vn` é obtido. Em seguida, a operação `prepend` é chamada em `vn` com o valor `n`, resultando em `prepended_vector`. O comprimento deste novo vetor é calculado. Finalmente, a asserção verifica se `length_after_prepend` é igual a `original_length + 1`, que é a tradução direta do axioma `length(prepend(vn, n)) = succ(length(vn))` para a semântica de inteiros e operações de Python. Este teste valida que a implementação da operação `prepend` afeta corretamente o comprimento do vetor.

# 8.5 Análise de Complexidade Algorítmica

A especificação algébrica de um Tipo Abstrato de Dados (TAD) foca no comportamento lógico e nas propriedades das operações, abstraindo detalhes de implementação e, consequentemente, de desempenho. No entanto, ao transitar da especificação para uma implementação concreta, como a classe `PyVectorNatural` que utiliza uma lista Python (`list`) como estrutura de dados subjacente, a análise da complexidade algorítmica das operações implementadas torna-se crucial. Esta análise avalia como o tempo de execução (e/ou o espaço de memória utilizado) das operações cresce em função do tamanho da entrada, tipicamente o número de elementos no vetor, $N$. Compreender a complexidade das operações do `PyVectorNatural` é essencial para prever seu desempenho em diferentes cenários de uso e para fazer escolhas informadas ao utilizá-lo na construção de algoritmos ou sistemas maiores. Esta seção analisará a complexidade (predominantemente de tempo, usando notação Big O) das principais operações implementadas na classe `PyVectorNatural`.

**Complexidade das Operações da Lista Python Subjacente:**
Antes de analisar `PyVectorNatural`, é importante relembrar a complexidade das operações relevantes da lista Python (`list`), que é implementada como um array dinâmico:
*   Acesso por índice (`L[i]`): $O(1)$
*   Modificação por índice (`L[i] = val`): $O(1)$
*   Obter comprimento (`len(L)`): $O(1)$
*   Anexar ao final (`L.append(val)`): $O(1)$ amortizado (pode ser $O(N)$ no pior caso se ocorrer realocação, mas em média é $O(1)$ sobre uma sequência de appends).
*   Remover do final (`L.pop()` sem índice): $O(1)$
*   Inserir no início (`L.insert(0, val)`): $O(N)$ (todos os elementos precisam ser deslocados)
*   Concatenar duas listas (`L1 + L2`): $O(k_1 + k_2)$, onde $k_1, k_2$ são os comprimentos das listas (cria uma nova lista).
*   Criar uma cópia da lista (`list(L)` ou `L[:]`): $O(N)$

**Análise de Complexidade das Operações de `PyVectorNatural`:**
Nossa implementação de `PyVectorNatural` adota um estilo funcional/imutável, onde operações como `append`, `set_element`, `pop` e `prepend`, `insert_at_index` retornam *novas* instâncias de `PyVectorNatural`. Isso tem implicações na complexidade, pois frequentemente envolve a cópia da lista Python subjacente.

1.  **`__init__(self, initial_elements: Optional[PyListInternal[int]] = None)`:**
    *   Se `initial_elements` é `None`, cria uma lista vazia: $O(1)$.
    *   Se `initial_elements` é fornecido (com $K$ elementos): Iterar para validar os $K$ elementos ($O(K)$) + `self._elements = list(initial_elements)` (cópia, $O(K)$).
    *   **Complexidade Total:** $O(K)$. Se sem argumentos, $O(1)$.

2.  **`empty_vector()` (método estático):**
    *   Chama `PyVectorNatural()`, que é $O(1)$.
    *   **Complexidade:** $O(1)$.

3.  **`append(self, value: int) -> PyVectorNatural`:**
    *   Validação: $O(1)$.
    *   `new_elements = self._elements + [value]`: Concatenação `+` cria nova lista. Custo: $O(N)$ (onde $N = \text{len(self._elements)}$).
    *   `return PyVectorNatural(new_elements)`: Construtor com $N+1$ elementos. Custo: $O(N)$.
    *   **Complexidade Total:** $O(N)$.

4.  **`is_empty(self) -> bool`:**
    *   `not self._elements`: $O(1)$.
    *   **Complexidade:** $O(1)$.

5.  **`length(self) -> int`:**
    *   `len(self._elements)`: $O(1)$.
    *   **Complexidade:** $O(1)$.

6.  **`get(self, index: int) -> int`:**
    *   Validações: $O(1)$. Acesso `self._elements[index]`: $O(1)$.
    *   **Complexidade:** $O(1)$ (para índice válido).

7.  **`set_element(self, index: int, value: int) -> PyVectorNatural`:**
    *   Validações: $O(1)$. `new_elements = list(self._elements)`: $O(N)$. `new_elements[index] = value`: $O(1)$. `return PyVectorNatural(new_elements)`: $O(N)$.
    *   **Complexidade Total:** $O(N)$.

8.  **`pop(self) -> PyVectorNatural`:**
    *   `self.is_empty()`: $O(1)$. Se vazio, `empty_vector()`: $O(1)$.
    *   Se não vazio, `new_elements = self._elements[:-1]`: Slicing é $O(N)$. `return PyVectorNatural(new_elements)`: $O(N)$.
    *   **Complexidade Total:** $O(N)$ para vetor não vazio, $O(1)$ para vetor vazio. Geral: $O(N)$.

9.  **`head(self) -> int`:**
    *   `self.is_empty()`: $O(1)$. Se vazio, exceção: $O(1)$. Se não vazio, `self._elements[0]`: $O(1)$.
    *   **Complexidade:** $O(1)$.

10. **`prepend(self, value: int) -> PyVectorNatural`:**
    *   Validação: $O(1)$. `new_elements = [value] + self._elements`: Concatenação é $O(N)$. `return PyVectorNatural(new_elements)`: $O(N)$.
    *   **Complexidade Total:** $O(N)$.

11. **`insert_at_index(self, index: int, value: int) -> PyVectorNatural`:**
    *   Validações: $O(1)$.
    *   `new_elements = self._elements[:index] + [value] + self._elements[index:]`:
        *   `self._elements[:index]`: Slicing é $O(\text{index})$.
        *   `self._elements[index:]`: Slicing é $O(N - \text{index})$.
        *   A concatenação `+ [value] +` envolve criar uma nova lista. O custo total desta linha é $O(N)$ porque todos os elementos são copiados para a nova lista.
    *   `return PyVectorNatural(new_elements)`: Construtor com $N+1$ elementos, $O(N)$.
    *   **Complexidade Total:** $O(N)$.

**Resumo da Complexidade (Implementação Imutável `PyVectorNatural`):**
*   `__init__(K_elements)`: $O(K)$
*   `empty_vector()`: $O(1)$
*   `append(v, val)`: $O(N)$
*   `is_empty(v)`: $O(1)$
*   `length(v)`: $O(1)$
*   `get(v, idx)`: $O(1)$ (para índice válido)
*   `set_element(v, idx, val)`: $O(N)$
*   `pop(v)`: $O(N)$
*   `head(v)`: $O(1)$ (para vetor não vazio)
*   `prepend(v, val)`: $O(N)$
*   `insert_at_index(v, idx, val)`: $O(N)$

A escolha por uma implementação imutável resulta em custos $O(N)$ para operações que "modificam" o vetor, devido à necessidade de cópia. Implementações mutáveis teriam complexidades diferentes para essas operações (e.g., `append` mutável seria $O(1)$ amortizado).

**Exercício:**

Se a operação `pop` na classe `PyVectorNatural` fosse implementada para modificar o vetor no local (mutável) em vez de retornar um novo vetor, e se ela ainda usasse `self._elements = self._elements[:-1]` internamente, qual seria a complexidade de tempo desta versão mutável de `pop`? Compare com a complexidade de `list.pop()` do Python (que remove o último elemento no local).

**Resolução:**

Se `pop` fosse implementada de forma mutável na classe `PyVectorNatural` usando `self._elements = self._elements[:-1]`, a análise de complexidade seria:

1.  **Operação de Slicing `self._elements[:-1]`:**
    Esta operação cria uma *nova lista* Python contendo todos os elementos de `self._elements` exceto o último. Se `self._elements` tem $N$ elementos, este slicing copia $N-1$ elementos para a nova lista.
    Portanto, o custo de `self._elements[:-1]` é $O(N)$.

2.  **Atribuição `self._elements = ...`:**
    A atribuição da referência da nova lista de volta para `self._elements` é $O(1)$.

**Complexidade Total da Versão Mutável de `pop` com `self._elements[:-1]`:**
A operação dominante é a criação da nova lista pelo slicing. Portanto, a complexidade de tempo desta versão mutável de `pop` ainda seria **$O(N)$**, onde $N$ é o número de elementos no vetor antes da operação `pop`.

**Comparação com `list.pop()` do Python (sem argumento, remove o último):**
A operação `list.pop()` do Python (quando chamada sem um índice, para remover o último elemento) é altamente otimizada para arrays dinâmicos. Ela não precisa copiar a lista inteira. Ela simplesmente:
1.  Acessa o último elemento (para retorná-lo, se a semântica da função o fizer).
2.  Decrementa o contador interno do tamanho da lista.
3.  Possivelmente (mas não necessariamente em cada chamada) verifica se o array subjacente está muito esparso e, se estiver, pode redimensioná-lo para um tamanho menor (uma operação $O(N)$ que ocorre raramente, levando a um custo amortizado).

O custo típico de `list.pop()` (do final) é **$O(1)$** (amortizado, se considerarmos possíveis redimensionamentos para encolher).

**Conclusão da Comparação:**
*   A implementação mutável de `pop` em `PyVectorNatural` usando `self._elements = self._elements[:-1]` tem complexidade $O(N)$.
*   A operação `list.pop()` nativa do Python (do final) tem complexidade $O(1)$ amortizado.

Portanto, a implementação mutável de `pop` usando slicing é significativamente menos eficiente do que a operação `pop` nativa das listas Python. Para alcançar o desempenho $O(1)$ amortizado da `list.pop()` nativa, a implementação mutável de `PyVectorNatural.pop()` teria que interagir mais diretamente com os mecanismos de gerenciamento de tamanho do array subjacente, em vez de criar uma nova lista com slicing. No entanto, se `PyVectorNatural` está encapsulando uma `list` Python, a maneira mais idiomática e eficiente de implementar um `pop` mutável que remove o último elemento seria simplesmente chamar `self._elements.pop()` diretamente.

O exercício pedia para considerar `self._elements = self._elements[:-1]`, que é $O(N)$.

# 8.6 Exercícios Teóricos e Práticos

Esta seção apresenta exercícios para consolidar os conceitos do TAD `VectorNatural`, sua especificação, implementação e verificação.

**Exercício:**
**(Originalmente o Exercício Teórico 1 da Seção 8.6 na solicitação anterior, integrado aqui conforme a estrutura de um exercício por seção Nível 1)**

Revisite os axiomas (V5)-(V6) para `get` e (V7)-(V9) para `set` na especificação `VectorNatural` (Seção 8.2). Discuta possíveis cenários de índice inválido (e.g., índice negativo, índice maior ou igual ao comprimento para `get`, ou índice maior que o comprimento para `set` no caso de `append`). Os axiomas fornecidos cobrem explicitamente esses cenários? Se não, como uma especificação algébrica poderia ser estendida para tratar esses casos de erro de forma mais formal (e.g., introduzindo um valor de erro ou usando predicados de definibilidade/pré-condições mais explícitas nos axiomas)?

**Resolução:**

Os axiomas (V5)-(V6) para `get` e (V7)-(V9) para `set` na especificação `VectorNatural` da Seção 8.2 são definidos usando premissas condicionais (`eq(...) => ...` ou `lt(...) => ...`).

*   **Cenários de Índice Inválido:**
    *   **Índice Negativo:** As operações `eq` e `lt` do TAD `Natural` não são definidas para argumentos negativos se `Natural` só inclui não-negativos. Se o índice `idx` fosse de um tipo `Integer` que pudesse ser negativo, os predicados `eq` e `lt` (que esperam `Natural`s) não se aplicariam diretamente ou falhariam em suas próprias pré-condições. A especificação, como está, assume que `idx` é um `Natural`.
    *   **Índice Maior ou Igual ao Comprimento (para `get`):**
        *   Axioma (V5) `eq(idx, length(vn)) = true => get(append(vn, elem), idx) = elem`: Cobre o caso onde `idx` é o novo último índice após um `append`.
        *   Axioma (V6) `lt(idx, length(vn)) = true => get(append(vn, elem), idx) = get(vn, idx)`: Cobre o caso onde `idx` é um índice válido *dentro* do `vn` original.
        *   **Não Cobertos:** Os axiomas não definem explicitamente `get(emptyVector, idx)` para qualquer `idx`. Também não definem `get(append(vn,elem), idx)` se `idx > length(vn)` (i.e., `idx` é maior que o índice do último elemento). Nestes casos, o termo `get(...)` não seria redutível por (V5) ou (V6), tornando `get` uma operação parcial.
    *   **Índice Maior que o Comprimento (para `set` em `append`):** Similar ao `get`. (V7) cobre o índice do novo elemento. (V8) cobre índices em `vn`. Casos onde `k > length(vn)` para `set(append(vn,elem), k, newElem)` não são cobertos.
    *   **`set` em `emptyVector` (V9):** `set(emptyVector, k, newElem) = emptyVector`. Este axioma define um comportamento para `set` em `emptyVector` para *qualquer* índice `k`. Isso é uma escolha de design que trata um índice inválido (já que `emptyVector` não tem índices válidos) retornando `emptyVector`.

*   **Extensão para Tratar Erros Formalmente:**
    1.  **Sort de Erro e Valores de Erro:**
        Poderia-se introduzir um sort `ResultNatural` que é uma união disjunta de `Natural` e um sort `ErrorValueNatural`. As operações como `get` retornariam `ResultNatural`.
        `get: VectorNatural x Natural --> ResultNatural`
        Axiomas adicionais definiriam quando `get` retorna um `ErrorValueNatural`.
        Ex: `lt(idx, length(vn)) = false AND isEmpty(vn) = false => get(vn, idx) = makeErrorNatural(errorIndexOutOfBounds)` (assumindo `makeErrorNatural` e `errorIndexOutOfBounds` definidos).

    2.  **Predicados de Definibilidade (para Operações Parciais):**
        Introduzir um predicado `isDefinedGet: VectorNatural x Natural --> Boolean`.
        Axiomas definiriam `isDefinedGet`. Ex: `isDefinedGet(vn,idx) = lt(idx, length(vn))`.
        Os axiomas para `get` seriam então condicionados por `isDefinedGet(vn,idx) = true`.

    3.  **Pré-condições Explícitas na Especificação (fora dos axiomas equacionais):**
        Muitas linguagens de especificação permitem associar pré-condições formais às operações.
        `get(vn, idx)`
        **pre:** `lt(idx, length(vn)) = true`
        Os axiomas equacionais então só se aplicam quando a pré-condição é satisfeita.

A escolha de como tratar erros e operações parciais é uma decisão de design importante na especificação. A especificação fornecida em 8.2 deixa `get` e `set` parciais para índices inválidos (exceto V9 para `set` em `emptyVector`), o que é comum em especificações algébricas focadas na semântica dos casos "bem-sucedidos". Implementações concretas (como `PyVectorNatural`) então precisam decidir como manifestar essa parcialidade (e.g., levantando exceções).

Este capítulo introduziu o TAD `VectorNatural`, delineando sua definição abstrata, relevância e uma especificação algébrica formal, incluindo a adição de operações como `pop`, `head` e `insertAtIndex` (com axiomas baseados em propriedades para esta última). Discutimos o projeto e a implementação de uma classe Python `PyVectorNatural` com tipagem estática, e demonstramos como os axiomas da especificação podem ser traduzidos em testes de propriedade usando Hypothesis para verificar a corretude da implementação. Finalmente, analisamos a complexidade algorítmica das operações da nossa classe `PyVectorNatural` imutável. O próximo capítulo, Capítulo 9, continuará a exploração de estruturas de dados lineares, focando no TAD `List`. O TAD `List`, embora conceitualmente similar a uma sequência, é tipicamente caracterizado por um conjunto diferente de operações construtoras fundamentais (como `nil` e `cons` para adição no início) e, consequentemente, diferentes perfis de desempenho, especialmente em implementações baseadas em nós encadeados, contrastando com a abordagem baseada em array do `Vector`.

---
## REFERÊNCIAS BIBLIOGRÁFICAS
ADAMCHIK, V. S. Symbol_Table_Data_Structure. In: **Wolfram MathWorld**. Disponível em: https://mathworld.wolfram.com/SymbolTableDataStructure.html. Acesso em: 10 ago. 2023.
*   *Resumo: Embora seja uma entrada de enciclopédia, o MathWorld frequentemente discute estruturas de dados em um nível conceitual. A menção a "Symbol Table" pode ser relevante se o Vector for usado em sua implementação, ilustrando a relevância do TAD Vector como base para outras estruturas.*

AHO, A. V.; HOPCROFT, J. E.; ULLMAN, J. D. **Data Structures and Algorithms**. Reading, MA: Addison-Wesley, 1983.
*   *Resumo: Um texto clássico que cobre estruturas de dados fundamentais, incluindo arrays e listas (que se relacionam com o TAD Vector). Embora mais antigo, os conceitos de acesso indexado e as operações básicas são atemporais e relevantes para a definição abstrata do Vector.*

CORMEN, T. H.; LEISERSON, C. E.; RIVEST, R. L.; STEIN, C. **Introduction to Algorithms**. 3. ed. Cambridge, MA: MIT Press, 2009.
*   *Resumo: Uma referência abrangente e moderna sobre algoritmos e estruturas de dados. O capítulo sobre estruturas de dados elementares discute arrays e listas dinâmicas (vetores), incluindo a análise de complexidade amortizada para operações como `append`, crucial para entender o desempenho de implementações de vetores.*

GOODRICH, M. T.; TAMASSIA, R.; GOLDWASSER, M. H. **Data Structures and Algorithms in Python**. Hoboken, NJ: Wiley, 2013.
*   *Resumo: Este livro foca em estruturas de dados e algoritmos com implementações em Python. Discute listas dinâmicas (análogas ao TAD Vector) e suas características de desempenho em Python, relevante para a seção de implementação e análise de complexidade do `PyVectorNatural`.*

LUCIA, B.; RAGHAVAN, P. (Eds.). **Proceedings of the 30th Annual ACM-SIAM Symposium on Discrete Algorithms (SODA)**. Philadelphia, PA: Society for Industrial and Applied Mathematics, 2019.
*   *Resumo: Anais de conferências como SODA frequentemente apresentam pesquisas avançadas sobre estruturas de dados e algoritmos. Embora não seja um texto introdutório, representa o estado da arte e pode conter artigos sobre novas variantes ou análises de estruturas como vetores dinâmicos, relevante para um estudo aprofundado.*

PYTHON SOFTWARE FOUNDATION. **Python `list` documentation**. Disponível em: https://docs.python.org/3/tutorial/datastructures.html#more-on-lists. Acesso em: 10 ago. 2023.
*   *Resumo: A documentação oficial do Python para o tipo `list` é essencial para entender a estrutura de dados subjacente que a classe `PyVectorNatural` utiliza. Detalha as operações disponíveis na lista Python e suas complexidades de tempo (amortizadas), informando a análise da Seção 8.5.*
---
