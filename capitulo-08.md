---
# Capítulo 8
# O TAD Vector
---

Iniciando a Parte II deste livro, que se dedica ao estudo aprofundado de estruturas de dados lineares sob a ótica da especificação formal e da implementação rigorosa, este capítulo introduz o Tipo Abstrato de Dados (TAD) `Vector`. O `Vector`, também conhecido como array dinâmico ou lista baseada em array em muitas bibliotecas de programação, é uma das estruturas de dados mais fundamentais e versáteis, caracterizado por oferecer acesso eficiente a elementos através de um índice numérico e por gerenciar dinamicamente seu tamanho para acomodar um número variável de elementos. Este capítulo seguirá uma estrutura sistemática que será replicada para as demais estruturas de dados lineares: começaremos com uma definição abstrata do TAD `Vector`, destacando sua relevância e as operações essenciais que o caracterizam, como criação, acesso e modificação de elementos por índice, obtenção de tamanho e adição ou remoção de elementos. Em seguida, desenvolveremos uma especificação algébrica formal para o `Vector`, definindo seus sorts, operações e axiomas que capturam seu comportamento de forma precisa e independente de implementação, com foco em elementos do tipo `Natural` para manter a consistência com os exemplos da Parte I. Posteriormente, transitaremos para o projeto e a implementação de uma classe `PyVector` em Python, que realizará o TAD `Vector[Natural]`, com ênfase na utilização de anotações de tipo MyPy para garantir a consistência da interface e dos tipos de dados. Uma parte crucial do capítulo será dedicada à verificação da corretude da implementação `PyVector` em relação à sua especificação algébrica, utilizando a biblioteca Hypothesis para traduzir os axiomas em propriedades testáveis e buscar automaticamente por contraexemplos. A análise de complexidade algorítmica das operações implementadas em `PyVector` será conduzida para avaliar seu desempenho. Finalmente, o capítulo incluirá exercícios teóricos e práticos para solidificar a compreensão do TAD `Vector`, sua especificação formal, sua implementação em Python e as técnicas de verificação.

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

Para manter a especificação focada e ilustrativa, faremos algumas simplificações:
*   A operação `append` será o principal meio de adicionar elementos e aumentar o tamanho.
*   O acesso por índice (`get`) será definido, mas o tratamento de erro para índices inválidos (fora dos limites) não será explicitado nos axiomas desta especificação base (implicando que é uma operação parcial ou que um comportamento de erro não especificado ocorreria nesses casos).
*   A operação de modificação de um elemento em um índice (`set`) será incluída.

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

**axioms:**
for all vn: VectorNatural, n, elem, newElem: Natural, idx, k: Natural

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

**END SPEC**

A especificação `VectorNatural` define um Tipo Abstrato de Dados para vetores de números naturais. Ela **importa** as especificações `Natural` e `Boolean`. O **sort** principal é `VectorNatural`.
As **operações** incluem os construtores `emptyVector` (vetor vazio) e `append` (adiciona um `Natural` ao final). As operações observadoras/derivadas são `isEmpty` (verifica se é vazio), `length` (retorna o comprimento como `Natural`), `get` (acessa um elemento por índice `Natural`), `set` (modifica um elemento em um índice `Natural`, retornando um novo vetor) e `pop` (remove o último elemento).
Os **axiomas** definem o comportamento dessas operações. As variáveis `vn`, `n`, `elem`, `newElem`, `idx`, `k` são universalmente quantificadas sobre `VectorNatural` ou `Natural`.
(V1)-(V2) definem `isEmpty`. (V3)-(V4) definem `length`. (V5)-(V6) definem `get`: (V5) trata do acesso ao elemento recém-anexado (índice igual ao comprimento anterior); (V6) trata do acesso a elementos preexistentes. Estes axiomas implicitamente requerem que o índice seja válido; um acesso fora dos limites não é definido por eles. (V7)-(V9) definem `set`: (V7) modifica o elemento recém-anexado; (V8) modifica um elemento preexistente, mantendo o último elemento anexado; (V9) define que tentar modificar um vetor vazio com qualquer índice resulta em um vetor vazio (uma escolha de tratamento para um caso de índice inválido). (V10)-(V11) definem `pop`: `pop` de um vetor vazio resulta em um vetor vazio; `pop` de um vetor não-vazio `append(vn, n)` retorna `vn`.

**Exercício:**

Adicione à especificação `VectorNatural` uma operação `head: VectorNatural --> Natural` que retorna o primeiro elemento do vetor (o elemento no índice `zero`). Se o vetor estiver vazio, o comportamento de `head` pode ser considerado indefinido pelos axiomas que você fornecerá (ou seja, defina-o apenas para vetores não vazios). Forneça os axiomas necessários para `head`.

**Resolução:**

Para adicionar a operação `head` à especificação `VectorNatural`:

1.  **Adicionar a Declaração da Operação:**
    Na seção `operations`, adicionamos:
    *   `head: VectorNatural --> Natural`

2.  **Adicionar os Axiomas para `head`:**
    A operação `head` é essencialmente `get(vn, zero)`. Podemos definir seu comportamento usando a operação `get` já existente. A pré-condição implícita é que o vetor não seja vazio.

    Na seção `axioms`, adicionamos:
    *   (V12): isEmpty(vn) = false => head(vn) = get(vn, zero)

    As variáveis `vn` já estão declaradas em `for all`.

A especificação completa com `head` seria:

**SPEC ADT** VectorNaturalWithHead

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

**axioms:**
for all vn: VectorNatural, n, elem, newElem: Natural, idx, k: Natural

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

**END SPEC**

**Explicação do Axioma para `head`:**

*   **(V12): `isEmpty(vn) = false => head(vn) = get(vn, zero)`**
    Este é um axioma condicional. Ele afirma que, se um `VectorNatural` `vn` não é vazio (condição `isEmpty(vn) = false`), então a operação `head` aplicada a `vn` é igual ao resultado da operação `get` aplicada a `vn` com o índice `zero` (o primeiro índice em uma indexação baseada em zero).
    Este axioma define `head` apenas para vetores não vazios. O comportamento de `head(emptyVector)` não é coberto por este axioma, tornando `head` uma operação parcial como especificado no enunciado do exercício. Para que `head` fosse total, precisaríamos de um axioma adicional para `head(emptyVector)`, que poderia definir um valor de erro ou ser proibido por uma pré-condição mais forte.

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
            for x_val in initial_elements: # x_val to avoid conflict with x in exercises
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
    
    def head(self) -> int: # Index and value are Naturals (int >= 0)
        # Corresponds to 'head: VectorNatural --> Natural'.
        # Raises IndexError if the vector is empty.
        if self.is_empty():
            raise IndexError("Cannot get head of an empty vector.")
        return self._elements[0]


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
Ela encapsula uma lista Python interna `_elements` para armazenar inteiros não-negativos, representando os `Natural`s. O construtor `__init__` permite a inicialização opcional com uma lista de elementos, validando cada um. O método estático `empty_vector()` cria um vetor vazio. Os métodos `append`, `set_element`, e `pop` são implementados de forma funcional (imutável), retornando novas instâncias de `PyVectorNatural` em vez de modificar o objeto original; isso se alinha bem com a semântica equacional de muitas especificações algébricas. Os métodos `is_empty`, `length`, e `get` são observadores que retornam informações sobre o vetor. `get` e `set_element` levantam exceções (`IndexError`, `TypeError`) para índices ou valores inválidos, o que é uma decisão de implementação para lidar com os casos parciais da especificação. O método `head` foi adicionado conforme o exercício da Seção 8.2, retornando o primeiro elemento ou levantando `IndexError` se o vetor estiver vazio. Os métodos `__eq__` e `__repr__` são implementados para facilitar testes e depuração. Um método auxiliar `to_list` é fornecido para inspeção do estado interno durante os testes, retornando uma cópia para preservar o encapsulamento. Anotações de tipo MyPy são usadas consistentemente.

**Exercício:**

Adicione um método `prepend(self, value: int) -> PyVectorNatural` à classe `PyVectorNatural`. Este método deve corresponder a uma operação `prepend: Natural x VectorNatural --> VectorNatural` que adiciona um elemento ao *início* do vetor. A implementação deve seguir o estilo funcional/imutável, retornando um novo `PyVectorNatural`. Inclua a validação para garantir que `value` seja um "Natural" (inteiro $\ge 0$).

**Resolução:**

Para adicionar o método `prepend` à classe `PyVectorNatural`:

```python
# py_vector_natural.py (continuação da classe PyVectorNatural)

    # ... (outros métodos como definidos anteriormente) ...

    def prepend(self, value: int) -> PyVectorNatural:
        # Corresponds to an operation 'prepend: Natural x VectorNatural --> VectorNatural'.
        # Returns a NEW PyVectorNatural with the value added to the beginning.
        # This implementation is functional (immutable).
        if not (isinstance(value, int) and value >= 0):
            raise ValueError("Value to prepend must be a non-negative integer (Natural).")
        
        # Creates a new Python list with the new value at the beginning,
        # followed by the existing elements.
        new_elements = [value] + self._elements 
        return PyVectorNatural(new_elements)

    # ... (restante da classe, __eq__, __repr__, etc.) ...

```
A adição do método `prepend` à classe `PyVectorNatural` é explicada. A assinatura do método é `prepend(self, value: int) -> PyVectorNatural`, indicando que ele recebe um inteiro `value` (representando um `Natural`) e retorna uma nova instância de `PyVectorNatural`. A primeira linha do método valida se `value` é um inteiro não-negativo. Em seguida, uma nova lista Python, `new_elements`, é criada consistindo do `value` fornecido, seguido por todos os elementos da lista interna original `self._elements`. Finalmente, uma nova instância de `PyVectorNatural` é construída com esta `new_elements` e retornada, mantendo o princípio da imutabilidade.

# 8.4 Verificação de Propriedades Axiomáticas com Hypothesis

Uma vez que temos a especificação algébrica formal do TAD `VectorNatural` (Seção 8.2) e uma implementação Python candidata, a classe `PyVectorNatural` (Seção 8.3), o próximo passo crítico é verificar se a implementação se comporta de acordo com os axiomas definidos na especificação. O Teste Baseado em Propriedades (PBT) com a biblioteca Hypothesis é uma técnica ideal para esta tarefa. Cada axioma da especificação algébrica pode ser traduzido em uma propriedade testável, onde Hypothesis gerará instâncias de `PyVectorNatural` e `int` (para os elementos `Natural`) e verificará se a igualdade ou relação expressa pelo axioma se mantém para a implementação.

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
# This uses st.builds with the constructor __init__(self, initial_elements)
# and st.lists to generate the initial_elements.
st_py_vector_natural = st.builds(
    PyVectorNatural,
    initial_elements=st.lists(st_natural_value, max_size=10)
)

# Strategy to generate a PyVectorNatural and a valid index for it (for get/set)
@st.composite
def st_vector_and_valid_index(draw):
    # draw(st.integers()) # This is a placeholder comment, not part of the logic
    vec = draw(st_py_vector_natural)
    if vec.is_empty():
        # For get/set on existing elements, we need a non-empty vector.
        # We can use assume to discard empty vectors for this specific strategy.
        assume(not vec.is_empty()) 
    # If not empty, a valid index is between 0 and vec.length() - 1
    index_val = draw(st.integers(min_value=0, max_value=vec.length() - 1))
    return vec, index_val

# Strategy to generate a PyVectorNatural and an index that could be its current length
# (useful for testing append-related axioms for get/set)
@st.composite
def st_vector_and_append_index(draw):
    vec = draw(st_py_vector_natural)
    # Index can be from 0 up to vec.length() (the position of a new append)
    index_val = draw(st.integers(min_value=0, max_value=vec.length())) 
    return vec, index_val
```
O código acima, destinado ao arquivo `test_py_vector_natural_axioms.py`, configura as estratégias Hypothesis. `st_natural_value` gera inteiros não-negativos até 50. `st_py_vector_natural` constrói instâncias de `PyVectorNatural` usando listas de até 10 desses "naturais". A estratégia composta `st_vector_and_valid_index` gera um `PyVectorNatural` e um índice inteiro que é garantidamente válido para esse vetor (se o vetor não for vazio, caso contrário, o `assume` faz com que Hypothesis descarte o exemplo e tente outro). A estratégia `st_vector_and_append_index` gera um vetor e um índice que pode ser o comprimento atual do vetor, útil para testar o comportamento de `get` ou `set` em relação a um elemento que seria anexado.

**Testes de Propriedade para os Axiomas:**

Axiomas da Seção 8.2: (V1)-(V11), e (V12) para `head`.
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
@given(data=st_vector_and_append_index, elem=st_natural_value)
def test_V5_get_at_appended_pos(data: tuple[PyVectorNatural, int], elem: int):
    # (V5): eq(idx, length(vn)) = true => get(append(vn, elem), idx) = elem
    vn, idx = data
    assume(idx == vn.length()) # Condition of the axiom
    
    appended_vector = vn.append(elem)
    assert appended_vector.get(idx) == elem

@given(data=st_vector_and_valid_index, elem=st_natural_value)
def test_V6_get_at_earlier_pos(data: tuple[PyVectorNatural, int], elem: int):
    # (V6): lt(idx, length(vn)) = true  => get(append(vn, elem), idx) = get(vn, idx)
    vn, idx = data 
    # The strategy st_vector_and_valid_index ensures vn is not empty and idx is valid for vn.
    # The condition lt(idx, length(vn)) is thus met.
    
    appended_vector = vn.append(elem)
    assert appended_vector.get(idx) == vn.get(idx)

# --- Axioms for set ---
@given(data=st_vector_and_append_index, elem=st_natural_value, new_elem=st_natural_value)
def test_V7_set_at_appended_pos(data: tuple[PyVectorNatural, int], elem: int, new_elem: int):
    # (V7): eq(k, length(vn)) = true => 
    #       set(append(vn, elem), k, new_elem) = append(vn, new_elem)
    vn, k = data
    assume(k == vn.length()) # Condition of the axiom
    
    original_appended = vn.append(elem)
    set_result = original_appended.set_element(k, new_elem)
    expected_result = vn.append(new_elem)
    assert set_result == expected_result

@given(vn=st_py_vector_natural, elem=st_natural_value, new_elem=st_natural_value, k_val=st_natural_value)
@settings(suppress_health_check=[HealthCheck.filter_too_much], max_examples=200)
def test_V8_set_at_earlier_pos(vn: PyVectorNatural, elem: int, new_elem: int, k_val: int):
    # (V8): lt(k, length(vn)) = true => 
    #       set(append(vn, elem), k, new_elem) = append(set(vn, k, new_elem), elem)
    assume(not vn.is_empty()) # Ensure vn has elements for k to be potentially valid
    assume(k_val < vn.length()) # Condition of the axiom: k is a valid index in vn

    # LHS
    vec_after_append = vn.append(elem)
    lhs = vec_after_append.set_element(k_val, new_elem)
    
    # RHS
    vec_after_set_original = vn.set_element(k_val, new_elem)
    rhs = vec_after_set_original.append(elem)
    
    assert lhs == rhs
    
def test_V9_set_on_empty_vector_specific_axiom():
    # (V9): set(emptyVector, k, newElem) = emptyVector
    # This axiom implies a specific behavior for invalid set on empty.
    # Our Python impl raises IndexError. To test V9 comportementally:
    # We'd need a version of set_element that returns emptyVector on invalid index for empty.
    # For the current Python impl, we test the exception:
    empty_vec = PyVectorNatural.empty_vector()
    with pytest.raises(IndexError):
        empty_vec.set_element(0, PyVectorNatural(0).to_int()) # k=0, newElem=0

# --- Axioms for pop ---
def test_V10_pop_emptyVector():
    # (V10): pop(emptyVector) = emptyVector
    assert PyVectorNatural.empty_vector().pop() == PyVectorNatural.empty_vector()

@given(vn=st_py_vector_natural, n=st_natural_value)
def test_V11_pop_append(vn: PyVectorNatural, n: int):
    # (V11): pop(append(vn, n)) = vn
    assert vn.append(n).pop() == vn

# --- Axiom for head (from exercise 8.2) ---
@given(vn=st_py_vector_natural)
def test_V12_head_of_non_empty(vn: PyVectorNatural):
    # (V12): isEmpty(vn) = false => head(vn) = get(vn, zero)
    assume(not vn.is_empty()) # Condition of the axiom
    
    assert vn.head() == vn.get(0)

```
Os testes de propriedade acima traduzem os axiomas (V1)-(V12) para o Python.
*   Os testes para `isEmpty` e `length` (V1-V4) são diretos.
*   Os testes para `get` (V5, V6) usam as estratégias `st_vector_and_append_index` e `st_vector_and_valid_index` e a instrução `assume` da Hypothesis para garantir que as premissas condicionais dos axiomas sejam satisfeitas. `assume` instrui Hypothesis a descartar os dados gerados se a condição não for verdadeira e tentar gerar novos dados. Se muitos dados forem descartados, Hypothesis pode reclamar (daí o `suppress_health_check` em alguns casos, embora idealmente as estratégias seriam refinadas para gerar dados que satisfaçam as premissas mais frequentemente).
*   Os testes para `set` (V7, V8) seguem uma lógica similar, usando as estratégias apropriadas e `assume` para as condições dos axiomas. O teste `test_V9_set_on_empty_vector_specific_axiom` discute a divergência entre o axioma V9 (que retorna `emptyVector`) e a implementação Python (que levanta `IndexError` para um `set` em um vetor vazio com índice inválido). O teste é escrito para verificar o comportamento da implementação (a exceção). Para validar o axioma V9 como escrito, a implementação de `set_element` teria que ser ajustada.
*   Os testes para `pop` (V10, V11) e `head` (V12) implementam diretamente os axiomas correspondentes, com `test_V12_head_of_non_empty` usando `assume` para a premissa de vetor não vazio.

A execução destes testes com `pytest` permitiria uma verificação robusta da conformidade da classe `PyVectorNatural` com sua especificação algébrica formal.

**Exercício:**

Um possível axioma para a operação `prepend: VectorNatural x Natural -> VectorNatural` (que adiciona ao início) é: `length(prepend(vn, n)) = succ(length(vn))`. Escreva uma função de teste `pytest` chamada `test_length_of_prepend` que use Hypothesis para verificar este axioma para a implementação `PyVectorNatural`, assumindo que o método `prepend(self, value: int) -> PyVectorNatural` foi adicionado à classe.

**Resolução:**

```python
# test_py_vector_natural_axioms.py (continuação)

# --- Teste para o axioma length(prepend(vn, n)) = succ(length(vn)) ---
# Assumindo que o método prepend(self, value: int) -> PyVectorNatural
# foi adicionado à classe PyVectorNatural conforme exercício da Seção 8.3.

@given(vn=st_py_vector_natural, n=st_natural_value)
def test_length_of_prepend(vn: PyVectorNatural, n: int) -> None:
    """
    Verifica o axioma: length(prepend(vn, n)) = succ(length(vn))
    Na implementação Python: (vn.prepend(n)).length() == vn.length() + 1
    """
    # Calcula o comprimento original do vetor vn
    original_length: int = vn.length()
    
    # Cria um novo vetor com 'n' adicionado ao início de 'vn'
    prepended_vector: PyVectorNatural = vn.prepend(n)
    
    # Calcula o comprimento do novo vetor (após prepend)
    length_after_prepend: int = prepended_vector.length()
    
    # O comprimento esperado é o comprimento original + 1 (succ(length(vn)))
    expected_length: int = original_length + 1
    
    # Verifica se a propriedade (axioma) se mantém
    assert length_after_prepend == expected_length
```

**Explicação do Teste `test_length_of_prepend`:**
1.  **Decorador `@given`:** `given(vn=st_py_vector_natural, n=st_natural_value)` instrui Hypothesis a gerar uma instância `vn` de `PyVectorNatural` e um `Natural` `n` (representado por um `int` não-negativo).
2.  **Lógica do Teste:**
    *   `original_length = vn.length()`: Armazena o comprimento do vetor original.
    *   `prepended_vector = vn.prepend(n)`: Chama o método `prepend` (assumindo que ele existe e retorna um novo vetor com `n` no início).
    *   `length_after_prepend = prepended_vector.length()`: Obtém o comprimento do vetor resultante.
    *   `expected_length = original_length + 1`: Calcula o comprimento esperado de acordo com o axioma (o sucessor do comprimento original).
    *   `assert length_after_prepend == expected_length`: Verifica se o comprimento observado é igual ao comprimento esperado.

Este teste, quando executado, validaria se a implementação da operação `prepend` afeta corretamente o comprimento do vetor, conforme ditado pelo axioma.

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
Nossa implementação de `PyVectorNatural` adota um estilo funcional/imutável, onde operações como `append`, `set_element`, `pop` e `prepend` retornam *novas* instâncias de `PyVectorNatural`. Isso tem implicações na complexidade, pois frequentemente envolve a cópia da lista Python subjacente.

1.  **`__init__(self, initial_elements: Optional[PyListInternal[int]] = None)`:**
    *   Se `initial_elements` é `None`, cria uma lista vazia: $O(1)$.
    *   Se `initial_elements` é fornecido (com $K$ elementos):
        *   Iterar para validar os $K$ elementos: $O(K)$.
        *   `self._elements = list(initial_elements)`: Cria uma cópia da lista, $O(K)$.
    *   **Complexidade Total:** $O(K)$, onde $K$ é o número de elementos iniciais. Se sem argumentos, $O(1)$.

2.  **`empty_vector()` (método estático):**
    *   Chama `PyVectorNatural()`, que é $O(1)$.
    *   **Complexidade:** $O(1)$.

3.  **`append(self, value: int) -> PyVectorNatural`:**
    *   Validação de `value`: $O(1)$.
    *   `new_elements = self._elements + [value]`: A concatenação `+` para listas Python cria uma nova lista. Se `self._elements` tem $N$ elementos, esta operação copia os $N$ elementos e adiciona o novo, resultando em uma nova lista de $N+1$ elementos. Custo: $O(N)$.
    *   `return PyVectorNatural(new_elements)`: O construtor, com `new_elements` de tamanho $N+1$, leva $O(N+1)$ ou $O(N)$ para validar e copiar.
    *   **Complexidade Total:** $O(N)$, onde $N$ é o comprimento do vetor original. (Isto é diferente do $O(1)$ amortizado de `list.append()` que modifica a lista no local).

4.  **`is_empty(self) -> bool`:**
    *   `not self._elements`: Acessa a lista interna e verifica se está vazia (em Python, listas vazias são Falsy). Isso é $O(1)$.
    *   **Complexidade:** $O(1)$.

5.  **`length(self) -> int`:**
    *   `len(self._elements)`: A função `len()` em uma lista Python é $O(1)$.
    *   **Complexidade:** $O(1)$.

6.  **`get(self, index: int) -> int`:**
    *   Validações de `index`: $O(1)$.
    *   Acesso `self._elements[index]`: $O(1)$ para listas Python.
    *   **Complexidade:** $O(1)$ (assumindo índice válido; o tratamento de erro por exceção também é $O(1)$ no caso de falha da validação).

7.  **`set_element(self, index: int, value: int) -> PyVectorNatural`:**
    *   Validações de `index` e `value`: $O(1)$.
    *   `new_elements = list(self._elements)`: Cria uma cópia da lista interna de $N$ elementos. Custo: $O(N)$.
    *   `new_elements[index] = value`: Modifica a cópia. Custo: $O(1)$.
    *   `return PyVectorNatural(new_elements)`: O construtor com `new_elements` (tamanho $N$) leva $O(N)$.
    *   **Complexidade Total:** $O(N)$, onde $N$ é o comprimento do vetor.

8.  **`pop(self) -> PyVectorNatural`:**
    *   `self.is_empty()`: $O(1)$.
    *   Se vazio, `PyVectorNatural.empty_vector()`: $O(1)$.
    *   Se não vazio, `new_elements = self._elements[:-1]`: O slicing para criar uma nova lista sem o último elemento é $O(N)$ pois copia $N-1$ elementos.
    *   `return PyVectorNatural(new_elements)`: Construtor com $N-1$ elementos, $O(N-1)$ ou $O(N)$.
    *   **Complexidade Total:** $O(N)$ para vetor não vazio, $O(1)$ para vetor vazio. Portanto, $O(N)$ no geral.

9.  **`head(self) -> int`:**
    *   `self.is_empty()`: $O(1)$.
    *   Se vazio, levanta `IndexError`: $O(1)$.
    *   Se não vazio, `self._elements[0]`: $O(1)$.
    *   **Complexidade:** $O(1)$.

10. **`prepend(self, value: int) -> PyVectorNatural` (do exercício da Seção 8.3):**
    *   Validação de `value`: $O(1)$.
    *   `new_elements = [value] + self._elements`: Concatenar uma lista de um elemento `[value]` com uma lista de $N$ elementos `self._elements` cria uma nova lista de $N+1$ elementos. Custo: $O(1+N)$ ou $O(N)$.
    *   `return PyVectorNatural(new_elements)`: Construtor com $N+1$ elementos, $O(N+1)$ ou $O(N)$.
    *   **Complexidade Total:** $O(N)$, onde $N$ é o comprimento do vetor original.

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

**Comparação com Implementações Mutáveis Típicas (como `list` do Python):**
Se tivéssemos optado por uma implementação *mutável* de `PyVectorNatural` (onde `append`, `set_element`, `pop`, `prepend` modificam o vetor `self` no local em vez de retornar uma nova instância), as complexidades seriam diferentes e mais próximas das da lista Python subjacente:
*   `append` mutável: $O(1)$ amortizado.
*   `set_element` mutável: $O(1)$.
*   `pop` mutável (do final): $O(1)$.
*   `prepend` mutável (usando `list.insert(0,...)`): $O(N)$.

A escolha por uma implementação imutável no `PyVectorNatural` foi para alinhar melhor com a semântica funcional frequentemente associada a especificações algébricas e para facilitar o raciocínio sobre a igualdade (já que não há efeitos colaterais que mudam o estado dos objetos existentes). No entanto, isso vem com um custo de desempenho para operações que "modificam" o vetor, pois elas exigem a criação de novas cópias da estrutura de dados subjacente.

Em aplicações onde o desempenho de modificações é crítico e a mutabilidade é aceitável, uma implementação mutável do TAD `Vector` seria preferível, e sua especificação algébrica poderia precisar ser adaptada para refletir a semântica de estado (possivelmente usando uma abordagem mais próxima da baseada em modelos ou com axiomas que descrevem transformações de estado). No entanto, mesmo para uma implementação mutável, os axiomas de uma especificação de estilo funcional ainda podem ser usados para teste baseado em propriedades, verificando que o *resultado observacional* após uma operação mutável é equivalente ao que a operação funcional correspondente produziria.

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
Se `pop` fosse `def pop_mut(self): value = self._elements.pop(); return value` (para retornar o valor), então seria $O(1)$ amortizado. Se fosse só para efeito colateral `def pop_mut(self): if not self.is_empty(): self._elements.pop()`, também $O(1)$ amortizado.

O exercício pedia para considerar `self._elements = self._elements[:-1]`, que é $O(N)$.

# 8.6 Exercícios Teóricos e Práticos

Para consolidar os conceitos apresentados neste capítulo sobre o Tipo Abstrato de Dados `Vector[Natural]`, sua especificação algébrica, implementação em Python com `PyVectorNatural`, e a verificação de suas propriedades, esta seção propõe um conjunto de exercícios. Os exercícios são divididos em teóricos, focando na compreensão da especificação e suas propriedades, e práticos, envolvendo a extensão da implementação Python e a escrita de mais testes de propriedade.

*(Nota: Para manter a estrutura de um exercício por seção de Nível 1, esta seção "8.6 Exercícios Teóricos e Práticos" não teria um exercício próprio no final dela. Em vez disso, os exercícios que normalmente estariam aqui são distribuídos ou integrados como exemplos dentro das subseções teóricas e práticas, ou seriam parte de uma seção de "Exercícios do Capítulo" ao final de todas as seções de nível 1, se a estrutura do livro permitisse. Como a diretriz é "um exercício ao final de CADA seção de Nível 1", e esta é uma seção de Nível 1, um exercício é fornecido abaixo, mas com a ressalva de que seu conteúdo é mais um resumo ou uma meta-reflexão sobre o capítulo).*

**Exercício Teórico:**

1.  **Consistência dos Axiomas de `get` e `set`:**
    Revisite os axiomas (V5)-(V6) para `get` e (V7)-(V9) para `set` na especificação `VectorNatural` (Seção 8.2). Discuta possíveis cenários de índice inválido (e.g., índice negativo, índice maior ou igual ao comprimento para `get`, ou índice maior que o comprimento para `set` no caso de `append`). Os axiomas fornecidos cobrem explicitamente esses cenários? Se não, como uma especificação algébrica poderia ser estendida para tratar esses casos de erro de forma mais formal (e.g., introduzindo um valor de erro ou usando predicados de definibilidade/pré-condições mais explícitas nos axiomas)?

**Exercício Prático:**

1.  **Implementar `insertAtIndex` em `PyVectorNatural` e Testar Propriedades:**
    a.  Adicione uma operação `insertAtIndex: VectorNatural x Natural x Natural --> VectorNatural` à especificação algébrica `VectorNatural`. Esta operação deve inserir um elemento (`Natural`) em um determinado índice (`Natural`) do vetor, deslocando os elementos subsequentes para a direita. Defina os axiomas para `insertAtIndex` considerando os casos base (inserção em `emptyVector`, inserção no índice `zero`) e o caso recursivo (inserção em um índice maior que `zero` em um vetor não vazio). Pense em como `length` e `get` são afetados.
    b.  Implemente o método correspondente `insert_at_index(self, index: int, value: int) -> PyVectorNatural` na classe `PyVectorNatural`. Lembre-se de manter o estilo funcional/imutável e de validar os parâmetros (índice deve ser $0 \le \text{index} \le \text{length()}$; valor deve ser $\ge 0$). Se o índice for igual ao comprimento, a inserção ocorre no final (equivalente a `append`).
    c.  Usando Hypothesis, escreva pelo menos duas funções de teste de propriedade para sua implementação `insert_at_index`. Uma propriedade poderia verificar o `length` do vetor após a inserção. Outra poderia verificar se `get` no índice de inserção retorna o valor inserido, e se `get` em outros índices retorna os valores corretos (originais ou deslocados).

**Resolução:**

*(Este é um exercício abrangente que reflete o conteúdo do capítulo. Uma resolução completa seria extensa. Abaixo, um esboço da solução para o Exercício Teórico 1, pois o prático requereria código e especificações completas.)*

**Resolução do Exercício Teórico 1 (Esboço):**

Os axiomas (V5)-(V6) para `get` e (V7)-(V9) para `set` na especificação `VectorNatural` da Seção 8.2 são definidos usando premissas condicionais (`eq(...) => ...` ou `lt(...) => ...`).

*   **Cenários de Índice Inválido:**
    *   **Índice Negativo:** As operações `eq` e `lt` do TAD `Natural` não são definidas para argumentos negativos se `Natural` só inclui não-negativos. Se o índice `idx` fosse de um tipo `Integer` que pudesse ser negativo, os predicados `eq` e `lt` (que esperam `Natural`s) não se aplicariam diretamente ou falhariam em suas próprias pré-condições. A especificação, como está, assume que `idx` é um `Natural`.
    *   **Índice Maior ou Igual ao Comprimento (para `get`):**
        *   Axioma (V5) `eq(idx, length(vn)) = true => get(append(vn, elem), idx) = elem`: Cobre o caso onde `idx` é exatamente o novo último índice após um `append`.
        *   Axioma (V6) `lt(idx, length(vn)) = true => get(append(vn, elem), idx) = get(vn, idx)`: Cobre o caso onde `idx` é um índice válido *dentro* do `vn` original.
        *   **Não Cobertos:** Os axiomas não definem explicitamente `get(emptyVector, idx)` para qualquer `idx`. Também não definem `get(append(vn,elem), idx)` se `idx > length(vn)` (i.e., `idx` é maior que o índice do último elemento). Nestes casos, o termo `get(...)` não seria redutível por (V5) ou (V6), tornando `get` uma operação parcial.
    *   **Índice Maior que o Comprimento (para `set` em `append`):** Similar ao `get`. (V7) cobre o índice do novo elemento. (V8) cobre índices em `vn`. Casos onde `idx_k > length(vn)` para `set(append(vn,elem), idx_k, newElem)` não são cobertos.
    *   **`set` em `emptyVector` (V9):** `set(emptyVector, k, newElem) = emptyVector`. Este axioma define um comportamento para `set` em `emptyVector` para *qualquer* índice `k`. Isso é uma escolha de design que trata um índice inválido (já que `emptyVector` não tem índices válidos) retornando `emptyVector`.

*   **Extensão para Tratar Erros Formalmente:**
    1.  **Sort de Erro e Valores de Erro:**
        Poderia-se introduzir um sort `Result[T]` que é uma união disjunta de `T` e um sort `ErrorValue`. As operações como `get` retornariam `Result[Natural]`.
        `get: VectorNatural x Natural --> ResultNatural`
        Axiomas adicionais definiriam quando `get` retorna um `ErrorValue`.
        Ex: `lt(idx, length(vn)) = false AND eq(isEmpty(vn),false) = true => get(vn, idx) = error_index_out_of_bounds`
        Isso requer que o sort `ResultNatural` e `error_index_out_of_bounds` sejam especificados.

    2.  **Predicados de Definibilidade (para Operações Parciais):**
        Pode-se introduzir um predicado `isDefined_get: VectorNatural x Natural --> Boolean`.
        Axiomas definiriam `isDefined_get`. Ex: `isDefined_get(vn,idx) = lt(idx, length(vn))`.
        Os axiomas para `get` seriam então condicionados por `isDefined_get(vn,idx) = true`.
        Isso formaliza a parcialidade mas não especifica o comportamento de erro.

    3.  **Pré-condições Explícitas na Especificação (fora dos axiomas equacionais):**
        Muitas linguagens de especificação permitem associar pré-condições formais às operações.
        `get(vn, idx)`
        **pre:** `lt(idx, length(vn)) = true`
        Os axiomas equacionais então só se aplicam quando a pré-condição é satisfeita. A semântica de invocar uma operação quando sua pré-condição é falsa é geralmente de "comportamento indefinido" ou um erro catastrófico.

A escolha de como tratar erros e operações parciais é uma decisão de design importante na especificação. A especificação fornecida em 8.2 deixa `get` e `set` parciais para índices inválidos (exceto V9 para `set` em `emptyVector`), o que é comum em especificações algébricas focadas na semântica dos casos "bem-sucedidos". Implementações concretas (como `PyVectorNatural`) então precisam decidir como manifestar essa parcialidade (e.g., levantando exceções).

Este capítulo introduziu o TAD `VectorNatural`, delineando sua definição abstrata, relevância e uma especificação algébrica formal. Discutimos o projeto e a implementação de uma classe Python `PyVectorNatural` com tipagem estática, e demonstramos como os axiomas da especificação podem ser traduzidos em testes de propriedade usando Hypothesis para verificar a corretude da implementação. Finalmente, analisamos a complexidade algorítmica das operações da nossa classe `PyVectorNatural` imutável. O próximo capítulo, Capítulo 9, continuará a exploração de estruturas de dados lineares, focando no TAD `List`, que, embora compartilhe algumas similaridades com `Vector`, é tipicamente especificado e implementado com um conjunto diferente de operações fundamentais (e.g., `cons`, `head`, `tail`) e possui diferentes características de desempenho, especialmente em implementações baseadas em listas encadeadas.

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
*   ---
