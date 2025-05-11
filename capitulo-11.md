---
# Capítulo 11
# Tipo Abstrato de Dados Queue
---

Avançando em nossa jornada pelas estruturas de dados lineares fundamentais, este capítulo se debruça sobre o Tipo Abstrato de Dados (TAD) `Queue[T]`, comumente conhecido como Fila. Em contraste com o TAD `Stack[T]` e sua política de acesso LIFO (Last-In, First-Out) explorada no capítulo anterior, o TAD `Queue[T]` é caracterizado pela política **FIFO (First-In, First-Out)**. Esta disciplina dita que o primeiro elemento inserido na fila é o primeiro a ser removido, um comportamento análogo ao de filas de espera em cenários cotidianos. As filas desempenham um papel vital em inúmeras aplicações computacionais, incluindo o escalonamento de processos em sistemas operacionais, o gerenciamento de buffers em sistemas de comunicação de dados, a simulação de eventos, algoritmos de busca em largura (BFS) em grafos, e em qualquer situação onde o processamento de itens deva respeitar a ordem de chegada. Este capítulo seguirá a estrutura metodológica estabelecida: iniciaremos com uma definição abstrata do TAD `Queue[T]`, identificando suas operações canônicas como `emptyQueue` (criação de uma fila vazia), `enqueue` (inserção de um elemento no final da fila), `dequeue` (remoção do elemento do início da fila), `front` (acesso ao elemento do início) e `isEmpty` (verificação de vacuidade). Em seguida, desenvolveremos uma especificação algébrica formal para `Queue[T]`, parametrizada pelo tipo de elemento `T`, detalhando seus sorts, operações e o conjunto de axiomas equacionais que capturam rigorosamente seu comportamento FIFO. Uma análise da consistência e completeza suficiente desta especificação será realizada. Posteriormente, projetaremos e implementaremos uma classe genérica `PyQueue[T]` em Python, utilizando anotações de tipo MyPy, que realizará o TAD `Queue[T]`. A validação da corretude desta implementação será conduzida através do Teste Baseado em Propriedades com a biblioteca Hypothesis. Concluiremos com uma análise da complexidade algorítmica das operações da classe `PyQueue[T]` e um exercício prático para reforçar os conceitos abordados.

# 11.1 Definição Abstrata e Relevância do Tipo de Dados `Queue[T]`

O Tipo Abstrato de Dados `Queue[T]`, ou Fila, é uma coleção linear de elementos do tipo genérico `T`, onde as inserções ocorrem em uma extremidade, denominada **final** (ou cauda, *rear*, *tail*) da fila, e as remoções e acessos ao próximo elemento a ser processado ocorrem na outra extremidade, denominada **início** (ou cabeça, *front*, *head*) da fila. A característica fundamental de uma fila é sua política de acesso **FIFO (First-In, First-Out)**: o elemento que está na fila há mais tempo (o primeiro a ser inserido) é o primeiro a ser removido ou acessado. Esta disciplina de ordenação temporal de processamento é a marca registrada das filas.

**Definição Abstrata:**
Um `Queue[T]` pode ser conceituado como uma sequência finita de elementos do tipo `T`. As operações essenciais que caracterizam um `Queue[T]` incluem:

1.  **Criação:**
    *   `emptyQueue: --> Queue[T]`: Cria uma fila vazia.
2.  **Modificação:**
    *   `enqueue: Queue[T] x T --> Queue[T]`: Adiciona um elemento `T` ao final da fila, resultando em uma nova fila. (Nota: algumas convenções usam `T x Queue[T]`).
    *   `dequeue: Queue[T] --> Queue[T]`: Remove o elemento do início da fila, resultando na fila restante. Esta operação é parcial; não é definida para uma fila vazia.
3.  **Acesso/Observação:**
    *   `front: Queue[T] --> T`: Retorna o elemento no início da fila sem removê-lo. Esta operação também é parcial; não é definida para uma fila vazia.
    *   `isEmpty: Queue[T] --> Boolean`: Verifica se a fila está vazia, retornando `true` se estiver, e `false` caso contrário.

Em especificações funcionais/imutáveis, `enqueue` e `dequeue` retornam novas instâncias da fila.

**Relevância do TAD `Queue[T]`:**
A política FIFO torna o TAD `Queue[T]` indispensável em uma variedade de contextos computacionais:

1.  **Gerenciamento de Recursos Compartilhados:** Filas são usadas para gerenciar o acesso a recursos limitados (impressoras, processadores, conexões de rede).
2.  **Buffers de Comunicação:** Armazenam temporariamente mensagens ou pacotes, preservando a ordem.
3.  **Simulação de Eventos:** Uma fila de eventos gerencia os próximos eventos a serem processados em ordem cronológica.
4.  **Algoritmos de Busca em Largura (BFS):** Na travessia de árvores ou grafos, a BFS utiliza uma fila para manter o rastro dos nós a serem visitados.
5.  **Escalonamento de Tarefas:** Schedulers em diversas aplicações usam filas para gerenciar tarefas pendentes.
6.  **Sistemas de Mensageria:** Arquiteturas baseadas em mensagens usam filas para desacoplar produtores e consumidores.

A eficiência das operações de fila depende da sua implementação. Implementações comuns incluem listas encadeadas (com ponteiros para o início e o fim, permitindo `enqueue` e `dequeue` em $O(1)$) ou arrays circulares (que também podem alcançar $O(1)$ para as operações básicas).

Neste capítulo, exploraremos a especificação algébrica de `Queue[T]`, sua implementação `PyQueue[T]` e a verificação de suas propriedades.

**Exercício:**

Considere um TAD `Queue[T]` com as operações `emptyQueue`, `enqueue(q,e)`, `dequeue(q)`, `front(q)`, e `isEmpty(q)`. Seja `T` o tipo `Natural`. Partindo de uma fila vazia `q0 = emptyQueue`, execute a seguinte sequência de operações e descreva o estado da fila (indicando o início e o fim) e o resultado da última operação, se houver:
1.  `q1 = enqueue(q0, natural_5)`
2.  `q2 = enqueue(q1, natural_10)`
3.  `val1 = front(q2)`
4.  `q3 = dequeue(q2)`
5.  `q4 = enqueue(q3, natural_3)`
6.  `val2 = front(q4)`

(onde `natural_5`, `natural_10`, `natural_3` são os `Natural`s 5, 10, e 3).

**Resolução:**

Vamos traçar a sequência de operações sobre a fila de `Natural`s. Usaremos $\langle \text{início} \ldots \text{fim} \rangle$ para representar a fila. A operação `enqueue(q,e)` adiciona `e` ao fim de `q`.

1.  **`q0 = emptyQueue`**
    *   Estado da fila `q0`: $\langle \rangle$ (fila vazia)

2.  **`q1 = enqueue(q0, natural_5)`**
    *   A operação `enqueue` adiciona `natural_5` ao final da fila `q0`.
    *   Estado da fila `q1`: $\langle \text{natural_5} \rangle$ (início: `natural_5`, fim: `natural_5`)

3.  **`q2 = enqueue(q1, natural_10)`**
    *   A operação `enqueue` adiciona `natural_10` ao final da fila `q1`.
    *   Estado da fila `q2`: $\langle \text{natural_5, natural_10} \rangle$ (início: `natural_5`, fim: `natural_10`)

4.  **`val1 = front(q2)`**
    *   A operação `front` retorna o elemento no início da fila `q2` (que é `natural_5`) sem removê-lo.
    *   Resultado da operação: `val1 = natural_5`.
    *   Estado da fila `q2` (após `front`): $\langle \text{natural_5, natural_10} \rangle$ (permanece inalterado)

5.  **`q3 = dequeue(q2)`**
    *   A operação `dequeue` remove o elemento do início da fila `q2` (que é `natural_5`).
    *   Estado da fila `q3`: $\langle \text{natural_10} \rangle$ (início: `natural_10`, fim: `natural_10`)

6.  **`q4 = enqueue(q3, natural_3)`**
    *   A operação `enqueue` adiciona `natural_3` ao final da fila `q3`.
    *   Estado da fila `q4`: $\langle \text{natural_10, natural_3} \rangle$ (início: `natural_10`, fim: `natural_3`)

7.  **`val2 = front(q4)`**
    *   A operação `front` retorna o elemento no início da fila `q4` (que é `natural_10`).
    *   Resultado da operação: `val2 = natural_10`.
    *   Estado da fila `q4` (após `front`): $\langle \text{natural_10, natural_3} \rangle$ (permanece inalterado)

Portanto, ao final da sequência:
*   `val1` é `natural_5`.
*   `val2` é `natural_10`.
*   O estado final da fila `q4` é $\langle \text{natural_10, natural_3} \rangle$.

# 11.2 Especificação Algébrica Formal do TAD `Queue[T]`

A especificação algébrica do TAD `Queue[T]` (Fila genérica) deve capturar formalmente seu comportamento FIFO.

**SPEC ADT** ElementTypeForQueue
**sorts:**
*   T
**END SPEC**

**SPEC ADT** Queue[ParamT: ElementTypeForQueue]
**imports:**
*   ParamT
*   Boolean
*   Natural

**sorts:**
*   Queue

**operations:**
*   emptyQueue: --> Queue
*   enqueue: Queue x T --> Queue
*   dequeue: Queue --> Queue
*   front: Queue --> T
*   isEmpty: Queue --> Boolean
*   size: Queue --> Natural

**axioms:**
for all q: Queue, elem, elem1: T

*   (Q1): isEmpty(emptyQueue) = true
*   (Q2): isEmpty(enqueue(q, elem)) = false

*   (Q3): front(enqueue(emptyQueue, elem)) = elem
*   (Q4): isEmpty(q) = false => front(enqueue(q, elem)) = front(q)

*   (Q5): dequeue(enqueue(emptyQueue, elem)) = emptyQueue
*   (Q6): isEmpty(q) = false => dequeue(enqueue(q, elem)) = enqueue(dequeue(q), elem)

*   (Q7): size(emptyQueue) = zero
*   (Q8): size(enqueue(q, elem)) = succ(size(q))

**END SPEC**

A especificação `Queue[ParamT: ElementTypeForQueue]` define um TAD de Fila genérica. `ElementTypeForQueue` introduz o sort `T`. A especificação `Queue` importa `ParamT`, `Boolean` e `Natural`. O sort principal é `Queue`.
As **operações** são: `emptyQueue` (cria fila vazia), `enqueue` (adiciona `T` ao final da fila `q`), `dequeue` (remove `T` do início), `front` (acessa `T` do início), `isEmpty` (verifica vacuidade) e `size` (retorna tamanho `Natural`).
`dequeue` e `front` são parciais, não definidas para `emptyQueue` por estes axiomas.

Os **axiomas** definem o comportamento FIFO:
*   (Q1) e (Q2) definem `isEmpty`.
*   (Q3) e (Q4) definem `front`: (Q3) é o caso base para uma fila com um elemento. (Q4) afirma que enfileirar um novo elemento `elem` no final de uma fila não vazia `q` não altera o elemento `front` da fila (que continua sendo `front(q)`).
*   (Q5) e (Q6) definem `dequeue`: (Q5) é o caso base. (Q6) é o axioma crucial para FIFO: desenfileirar de uma fila `q` (não vazia) à qual `elem` foi adicionado ao final é o mesmo que adicionar `elem` ao final da fila resultante de `dequeue(q)`. Isso efetivamente "move" `elem` para o final da fila menor.
*   (Q7) e (Q8) definem `size` recursivamente.
As variáveis `q`, `elem`, `elem1` são universalmente quantificadas. `front(emptyQueue)` e `dequeue(emptyQueue)` permanecem indefinidas.

## 11.2.1 Análise de Corretude e Completeza da Especificação `Queue[T]`

Analisaremos a especificação `Queue[T]` quanto à consistência e completeza suficiente, considerando `emptyQueue` e `enqueue` como construtores.

**Consistência:**
1.  **Proteção dos Tipos Importados:** Os axiomas não parecem introduzir inconsistências em `Boolean`, `Natural` ou `T`.
2.  **Consistência Interna de `Queue[T]`:** `emptyQueue` é distinto de `enqueue(q, elem)` (via Q1, Q2). A distinção entre filas não vazias é mantida pela semântica FIFO capturada pelos axiomas (Q3)-(Q6).
3.  **Modelo Inicial:** O modelo de sequências finitas com `enqueue` no final e `dequeue`/`front` no início é consistente com os axiomas.

A especificação `Queue[T]` parece **consistente**.

**Completeza Suficiente:**
Analisamos `isEmpty`, `front`, `dequeue`, `size` aplicadas a `emptyQueue` e `enqueue(q, elem)`.

1.  **`isEmpty(que: Queue)`:** Suficientemente completa (casos Q1, Q2).
2.  **`front(que: Queue)`:**
    *   Caso $que = \text{emptyQueue}$: Indefinido.
    *   Caso $que = \text{enqueue(q', e)}$:
        *   Se $q' = \text{emptyQueue}$: `front(enqueue(emptyQueue, e))` $\rightarrow_{Q3}$ `e`. (Resultado `T`).
        *   Se $q' \neq \text{emptyQueue}$: `front(enqueue(q', e))` $\rightarrow_{Q4}$ `front(q')`. Redução sobre `q'`. Por indução, atinge o caso (Q3).
    `front` é suficientemente completa para filas não-vazias. Parcial para `emptyQueue`.

3.  **`dequeue(que: Queue)`:**
    *   Caso $que = \text{emptyQueue}$: Indefinido.
    *   Caso $que = \text{enqueue(q', e)}$:
        *   Se $q' = \text{emptyQueue}$: `dequeue(enqueue(emptyQueue, e))` $\rightarrow_{Q5}$ `emptyQueue`. (Resultado `Queue` construtor).
        *   Se $q' \neq \text{emptyQueue}$: `dequeue(enqueue(q', e))` $\rightarrow_{Q6}$ `enqueue(dequeue(q'), e)`. Redução sobre `dequeue(q')`. Por indução, se reduz a termos construtores.
    `dequeue` é suficientemente completa para filas não-vazias. Parcial para `emptyQueue`.

4.  **`size(que: Queue)`:** Suficientemente completa (casos Q7, Q8, por indução).

**Conclusão da Análise:**
*   **Consistência:** A especificação `Queue[T]` é consistente.
*   **Completeza Suficiente:** `isEmpty` e `size` são suficientemente completas. `front` e `dequeue` são parciais, não cobrindo `emptyQueue`.

**Exercício:**

Adicione uma operação `rear: Queue[T] --> T` à especificação `Queue[T]`. Esta operação deve retornar o elemento no final da fila (o último elemento inserido) sem removê-lo. Se a fila estiver vazia, a operação é indefinida. Forneça os axiomas para `rear`. (Lembre-se que o perfil de `enqueue` é `Queue[T] x T --> Queue[T]`).

**Resolução:**

Adicionamos `rear` à especificação `Queue[T]`.

**SPEC ADT** QueueWithRear[ParamT: ElementTypeForQueue]
**imports:**
*   ParamT
*   Boolean
*   Natural
*   Queue[ParamT]

**sorts:**
*   Queue

**operations:**
*   rear: Queue --> T

**axioms:**
for all q: Queue, elem: T

*   (QR1): rear(enqueue(emptyQueue, elem)) = elem
*   (QR2): isEmpty(q) = false => rear(enqueue(q, elem)) = elem

**END SPEC**

A especificação `QueueWithRear` enriquece `Queue[T]`. A nova operação `rear` recebe uma `Queue` e retorna um elemento `T`.
O axioma (QR1) é o caso base: se uma fila é formada apenas por `enqueue(emptyQueue, elem)` (contém um único elemento), então `rear` retorna esse `elem`.
O axioma (QR2) é o caso geral: se um novo `elem` é adicionado ao final de uma fila `q` não vazia (formando `enqueue(q, elem)`), então `rear` dessa nova fila é o `elem` recém-adicionado.
`rear(emptyQueue)` permanece indefinida, tornando `rear` uma operação parcial.

# 11.3 Projeto e Implementação em Python com Tipagem Estática (MyPy)

Para implementar o TAD `Queue[T]` em Python, criaremos uma classe genérica `PyQueue[T]`. Uma escolha comum para implementar filas eficientemente é usar duas pilhas (`PyStack[T]`) internas: uma `in_stack` para operações `enqueue` e uma `out_stack` para operações `dequeue` e `front`. Quando a `out_stack` fica vazia e uma operação de `dequeue` ou `front` é necessária, todos os elementos da `in_stack` são movidos para a `out_stack`, invertendo sua ordem no processo. Isso garante que as operações `enqueue` e `dequeue` tenham custo $O(1)$ amortizado. A implementação seguirá um estilo funcional/imutável.

**Design da Classe `PyQueue[T]`:**

```python
# py_queue.py
from __future__ import annotations # For type hints of PyQueue[T] within the class itself
from typing import TypeVar, Generic, Any, Optional, List as PyListInternal
# Assuming PyStack implementation from Chapter 10 is in py_stack.py
from py_stack import PyStack 

T = TypeVar('T') # Generic type parameter for elements

class PyQueue(Generic[T]):
    # Represents the queue using two stacks:
    # _in_stack: for enqueuing new elements (newest at top of _in_stack)
    # _out_stack: for dequeuing and front access (oldest at top of _out_stack)
    _in_stack: PyStack[T]
    _out_stack: PyStack[T]

    # Private constructor, primarily for internal use by class methods.
    # Users should use empty_queue() and instance methods like enqueue().
    def __init__(self, in_stack: PyStack[T], out_stack: PyStack[T]) -> None:
        self._in_stack = in_stack
        self._out_stack = out_stack

    @staticmethod
    def empty_queue() -> PyQueue[Any]: 
        # Corresponds to 'emptyQueue: --> Queue[T]'
        return PyQueue(PyStack.empty_stack(), PyStack.empty_stack())

    def is_empty(self) -> bool:
        # Corresponds to 'isEmpty: Queue[T] --> Boolean'
        return self._in_stack.is_empty() and self._out_stack.is_empty()

    def _transfer_in_to_out(self) -> PyQueue[T]:
        # Internal helper to transfer elements from _in_stack to _out_stack
        # if _out_stack is empty. This is called to "normalize" the queue
        # before a front or dequeue operation.
        # Returns a new PyQueue instance representing the normalized state.
        if self._out_stack.is_empty():
            if self._in_stack.is_empty():
                return self # Both empty, already normalized and empty
            
            # Transfer elements: pop from _in_stack and push to new_out_stack
            new_out_stack_transferred = PyStack.empty_stack()
            temp_in_stack_for_transfer = self._in_stack
            while not temp_in_stack_for_transfer.is_empty():
                new_out_stack_transferred = new_out_stack_transferred.push(temp_in_stack_for_transfer.top())
                temp_in_stack_for_transfer = temp_in_stack_for_transfer.pop()
            # After transfer, _in_stack becomes empty, _out_stack has the transferred elements
            return PyQueue(PyStack.empty_stack(), new_out_stack_transferred)
        return self # No transfer was needed, _out_stack was not empty

    def enqueue(self, value: T) -> PyQueue[T]:
        # Corresponds to 'enqueue: Queue[T] x T --> Queue[T]'.
        # Returns a NEW PyQueue with the value added to the rear.
        # In the two-stack model, new elements are pushed onto the _in_stack.
        new_in_stack = self._in_stack.push(value)
        return PyQueue(new_in_stack, self._out_stack)

    def front(self) -> T:
        # Corresponds to 'front: Queue[T] --> T'.
        # Returns the front element. Raises IndexError if empty.
        if self.is_empty():
            raise IndexError("front from empty queue")
        
        # Normalize by potentially transferring from in_stack to out_stack
        normalized_queue = self._transfer_in_to_out()
        # After normalization, if the queue was not truly empty,
        # the front element is at the top of the (new) _out_stack.
        if normalized_queue._out_stack.is_empty():
            # This case should ideally not be reached if self.is_empty() was false.
            # It implies _in_stack was also empty after an attempted transfer,
            # which means the original queue was indeed empty.
             raise IndexError("front from empty queue (internal state error after transfer)")
        return normalized_queue._out_stack.top()


    def dequeue(self) -> PyQueue[T]:
        # Corresponds to 'dequeue: Queue[T] --> Queue[T]'.
        # Returns a NEW PyQueue with the front element removed.
        # Raises IndexError if the queue is empty.
        if self.is_empty():
            raise IndexError("dequeue from empty queue")
            
        # Normalize by potentially transferring from in_stack to out_stack
        normalized_queue = self._transfer_in_to_out()
        
        if normalized_queue._out_stack.is_empty():
             # Similar to front(), this implies original queue was empty.
             raise IndexError("dequeue from effectively empty queue after transfer attempt")

        # Pop from the (new) _out_stack
        new_out_stack_after_pop = normalized_queue._out_stack.pop()
        # The _in_stack of normalized_queue is empty if a transfer occurred.
        return PyQueue(normalized_queue._in_stack, new_out_stack_after_pop)

    def size(self) -> int: 
        # Corresponds to 'size: Queue[T] --> Natural'
        return self._in_stack.size() + self._out_stack.size()
    
    def rear(self) -> T:
        # Corresponds to 'rear: Queue[T] --> T'
        # Returns the rear element (last enqueued). Raises IndexError if empty.
        if self.is_empty():
            raise IndexError("rear from empty queue")
        
        # If _in_stack is not empty, its top is the most recently enqueued (rear).
        if not self._in_stack.is_empty():
            return self._in_stack.top()
        else:
            # If _in_stack is empty, all elements are in _out_stack (already transferred).
            # The rear element was the one at the "bottom" of _out_stack.
            # PyStack._elements[0] is the bottom if top is at end of list.
            if self._out_stack.is_empty(): # Should be caught by self.is_empty()
                 raise IndexError("rear from effectively empty queue (internal error)")
            return self._out_stack._elements[0] # Accessing internal structure for this logic

    # Methods for equality and representation
    def __eq__(self, other: object) -> bool:
        # Equality check. For robustness, compare canonical list representations.
        if not isinstance(other, PyQueue):
            return NotImplemented
        return self.to_list_front_to_back() == other.to_list_front_to_back()

    def __repr__(self) -> str:
        # Representation showing elements in FIFO order for clarity.
        return f"PyQueue({self.to_list_front_to_back()})"

    def to_list_front_to_back(self) -> PyListInternal[T]:
        # Helper for testing and robust __eq__. Returns elements in FIFO order.
        
        # Elements in _out_stack are in LIFO order (top is front of queue).
        # We need to reverse them to get them in FIFO for the out_stack part.
        out_elements_fifo_ordered = []
        temp_out_stack_for_list = self._out_stack
        while not temp_out_stack_for_list.is_empty():
            out_elements_fifo_ordered.append(temp_out_stack_for_list.top())
            temp_out_stack_for_list = temp_out_stack_for_list.pop()
        # Now out_elements_fifo_ordered has [front_of_out, next_of_out, ... last_of_out (deepest)]
        # We need to reverse it for the final list: [deepest_of_out, ..., next_of_out, front_of_out] NO
        # Actually, PyStack.pop removes from end, PyStack.top is end.
        # So, if _out_stack's internal list is [o1, o2, o3(top)], then front is o3.
        # The list from out_stack should be [o3, o2, o1] for FIFO.
        
        # Corrected logic for to_list_front_to_back:
        # _out_stack elements (top is front of queue): iterate popping from top
        out_part: PyListInternal[T] = []
        current_out_stack = self._out_stack
        while not current_out_stack.is_empty():
            out_part.append(current_out_stack.top())
            current_out_stack = current_out_stack.pop()
        # out_part is now [front_elem, second_elem, ...], needs to be reversed for stack interpretation.
        # No, if top of out_stack is the front of queue, then out_part is [front, 2nd, ...]. This is fine.
        # Example: out_stack._elements = [c,b,a] (a is top/front). Popping yields a, then b, then c.
        # So, out_part = [a,b,c]. This is correct order for front part.

        # _in_stack elements (top is most_recent_enqueue, i.e. rear of in_stack part)
        # We need them in order of enqueue: bottom of in_stack first.
        in_part_bottom_to_top = self._in_stack.to_list_bottom_to_top()
        
        return out_part + in_part_bottom_to_top
```
O código Python acima, no arquivo `py_queue.py`, define a classe genérica `PyQueue[T]`.
Ela é implementada usando duas pilhas internas, `_in_stack` (para enfileirar) e `_out_stack` (para desenfileirar/frente), ambas instâncias de `PyStack[T]`. O construtor `__init__` é privado. `empty_queue()` cria uma fila vazia. `is_empty()` verifica ambas as pilhas. O método auxiliar `_transfer_in_to_out()` move elementos de `_in_stack` para `_out_stack` se `_out_stack` estiver vazia, invertendo a ordem para manter a lógica FIFO, e retorna uma *nova* instância de `PyQueue`. `enqueue` adiciona à `_in_stack`. `front` e `dequeue` primeiro chamam `_transfer_in_to_out()` para normalizar a fila (garantindo que o elemento da frente esteja no topo de `_out_stack`, se a fila não estiver vazia) e então operam em `_out_stack`. `size` soma os tamanhos das pilhas. `rear` obtém o topo de `_in_stack` ou, se vazia, o elemento da base de `_out_stack`. As operações que modificam a fila são imutáveis. Métodos `__eq__`, `__repr__`, e `to_list_front_to_back` (para obter elementos em ordem FIFO) são fornecidos.

**Exercício:**

Implemente um método `rear(self) -> T` na classe `PyQueue[T]`. Este método deve corresponder à operação `rear: Queue[T] --> T` da resolução do exercício da Seção 10.2 (Capítulo da Fila, mas o número do exercício seria 11.2), retornando o elemento no final da fila (o último elemento inserido) sem removê-lo. Se a fila estiver vazia, deve levantar `IndexError`.
*(Este exercício já foi resolvido e incorporado na classe `PyQueue[T]` acima.)*

**Resolução:**

O método `rear` já foi adicionado à classe `PyQueue[T]` na listagem de código acima. Sua implementação é:
```python
# py_queue.py (dentro da classe PyQueue[T])
    # ...
    def rear(self) -> T:
        # Returns the rear element (last enqueued). Raises IndexError if empty.
        if self.is_empty():
            raise IndexError("rear from empty queue")
        
        # If _in_stack is not empty, its top is the most recently enqueued (rear).
        if not self._in_stack.is_empty():
            return self._in_stack.top()
        else:
            # If _in_stack is empty, all elements are in _out_stack (already transferred).
            # The rear element was the one at the "bottom" of _out_stack.
            # PyStack._elements[0] is the bottom if top is at end of list.
            if self._out_stack.is_empty(): # Should be caught by self.is_empty()
                 raise IndexError("rear from effectively empty queue (internal error)") # Should not happen if not self.is_empty()
            return self._out_stack._elements[0] # Accessing internal structure's first element.
    # ...
```
Este método retorna o topo da `_in_stack` se ela não estiver vazia (pois é o último elemento enfileirado). Se `_in_stack` estiver vazia, mas a fila não (então `_out_stack` não está), o elemento final é o que está na "base" da `_out_stack` (o primeiro elemento da sua lista interna `_elements`, assumindo que o topo da `PyStack` é o final da lista `_elements`).

# 11.4 Verificação de Propriedades Axiomáticas com Hypothesis

Com a especificação algébrica de `Queue[T]` e a implementação `PyQueue[T]`, usamos Hypothesis para verificar se a implementação satisfaz os axiomas.

**Estratégias Hypothesis para `PyQueue[T]` e `T`:**
Usaremos `st.integers()` para `T`.

```python
# test_py_queue.py
import pytest
from hypothesis import given, strategies as st, assume, settings, HealthCheck
from typing import TypeVar, Any

# Import the implementations
from py_stack import PyStack # Needed by PyQueue
from py_queue import PyQueue 

# Strategy for generating element type 'T'. For this example, use integers.
st_element_type = st.integers(min_value=-50, max_value=50)

# Strategy for PyStack[int] (dependency for PyQueue)
st_py_stack_for_queue = st.builds(
    PyStack[int],
    initial_elements=st.lists(st_element_type, max_size=5) 
)

# Strategy for PyQueue[int] using its private constructor
# This generates the two internal stacks directly.
st_py_queue = st.builds(
    PyQueue[int],
    in_stack=st_py_stack_for_queue,
    out_stack=st_py_stack_for_queue
)

# Strategy to ensure a non-empty queue (for testing front/dequeue)
@st.composite
def st_non_empty_py_queue(draw):
    q = draw(st_py_queue)
    assume(not q.is_empty())
    return q
```
O código de teste `test_py_queue.py` configura as estratégias. `st_element_type` gera inteiros. `st_py_stack_for_queue` é uma estratégia para as pilhas internas. `st_py_queue` constrói `PyQueue[int]` gerando suas duas pilhas internas. `st_non_empty_py_queue` filtra para filas não vazias.

**Testes de Propriedade para os Axiomas de `Queue[T]`:**
Axiomas (Q1)-(Q8) da Seção 11.2 e (QR1) do exercício para `rear`.

```python
# test_py_queue.py (continuação)

# --- Axioms for isEmpty ---
def test_Q1_is_empty_emptyQueue():
    # (Q1): isEmpty(emptyQueue) = true
    assert PyQueue[Any].empty_queue().is_empty() == True

@given(q=st_py_queue, elem=st_element_type)
def test_Q2_is_empty_enqueue(q: PyQueue[int], elem: int):
    # (Q2): isEmpty(enqueue(q, elem)) = false
    assert q.enqueue(elem).is_empty() == False

# --- Axioms for front ---
@given(elem=st_element_type)
def test_Q3_front_enqueue_on_empty(elem: int):
    # (Q3): front(enqueue(emptyQueue, elem)) = elem
    empty_q = PyQueue[Any].empty_queue()
    assert empty_q.enqueue(elem).front() == elem

@given(q_non_empty=st_non_empty_py_queue, elem=st_element_type)
@settings(suppress_health_check=[HealthCheck.filter_too_much, HealthCheck.too_slow], deadline=1000)
def test_Q4_front_enqueue_on_non_empty(q_non_empty: PyQueue[int], elem: int):
    # (Q4): isEmpty(q) = false => front(enqueue(q, elem)) = front(q)
    # q_non_empty strategy ensures isEmpty(q_non_empty) is false
    
    front_of_original_q = q_non_empty.front()
    front_after_enqueue = q_non_empty.enqueue(elem).front()
    assert front_after_enqueue == front_of_original_q

# --- Axioms for dequeue ---
@given(elem=st_element_type)
def test_Q5_dequeue_enqueue_on_empty(elem: int):
    # (Q5): dequeue(enqueue(emptyQueue, elem)) = emptyQueue
    empty_q = PyQueue[Any].empty_queue()
    assert empty_q.enqueue(elem).dequeue() == PyQueue[Any].empty_queue()

@given(q_non_empty=st_non_empty_py_queue, elem=st_element_type)
@settings(suppress_health_check=[HealthCheck.filter_too_much, HealthCheck.too_slow], deadline=2000, max_examples=50) # Increased deadline
def test_Q6_dequeue_enqueue_on_non_empty(q_non_empty: PyQueue[int], elem: int):
    # (Q6): isEmpty(q) = false => dequeue(enqueue(q, elem)) = enqueue(dequeue(q), elem)
    # q_non_empty strategy ensures isEmpty(q_non_empty) is false
    
    lhs = q_non_empty.enqueue(elem).dequeue()
    rhs = q_non_empty.dequeue().enqueue(elem)
    assert lhs == rhs

# Test behavior for front/dequeue on emptyQueue (expect exceptions)
def test_front_on_empty_queue_raises_exception():
    empty_q = PyQueue[Any].empty_queue()
    with pytest.raises(IndexError):
        empty_q.front()

def test_dequeue_on_empty_queue_raises_exception():
    empty_q = PyQueue[Any].empty_queue()
    with pytest.raises(IndexError):
        empty_q.dequeue()

# --- Axioms for size ---
def test_Q7_size_emptyQueue():
    # (Q7): size(emptyQueue) = zero
    assert PyQueue[Any].empty_queue().size() == 0

@given(q=st_py_queue, elem=st_element_type)
def test_Q8_size_enqueue(q: PyQueue[int], elem: int):
    # (Q8): size(enqueue(q, elem)) = succ(size(q))
    assert q.enqueue(elem).size() == q.size() + 1

# --- Axiom for rear ---
@given(q=st_py_queue, elem=st_element_type)
@settings(suppress_health_check=[HealthCheck.filter_too_much, HealthCheck.too_slow], deadline=1000)
def test_QR1_rear_of_enqueue(q: PyQueue[int], elem: int):
    # Axiom: rear(enqueue(q, elem)) = elem
    # (This assumes enqueue(q,elem) adds elem at the end)
    new_q = q.enqueue(elem) 
    assert new_q.rear() == elem

def test_rear_on_empty_queue_raises_exception():
    empty_q = PyQueue[Any].empty_queue()
    with pytest.raises(IndexError):
        empty_q.rear()
```
Os testes acima verificam os axiomas (Q1)-(Q8) e (QR1).
Testes para `isEmpty`, `front`, `dequeue`, `size` seguem a lógica dos axiomas, usando `assume` ou estratégias específicas para lidar com premissas condicionais ou casos parciais. O teste para `rear` (QR1) verifica se o elemento enfileirado por último é de fato o `rear`. Testes para exceções em `front`, `dequeue` e `rear` em filas vazias também são incluídos. Devido à complexidade potencial da normalização interna e criação de objetos no `PyQueue` imutável com pilhas imutáveis, alguns `settings` (como `deadline` e `max_examples`) podem precisar de ajuste para evitar timeouts de Hypothesis, especialmente para (Q6).

**Exercício:**

Considerando a operação `multi_pop(k: Natural, s: Stack[T]) -> Stack[T]` do capítulo anterior. Defina uma operação análoga `multi_dequeue(k: Natural, q: Queue[T]) -> Queue[T]` para a Fila, que remove `k` elementos do início da fila. Escreva um teste de propriedade para o axioma `multi_dequeue(zero, q) = q`. Assuma que `PyQueue` tem um método `multi_dequeue(self, k:int) -> PyQueue[T]` implementado.

**Resolução:**

Primeiro, a definição axiomática de `multi_dequeue` (semelhante a `multi_pop`):
**SPEC ADT** QueueWithMultiDequeue[ParamT: ElementTypeForQueue]
**imports:** Queue[ParamT], Natural
**operations:** multi_dequeue: Natural x Queue --> Queue
**axioms:**
for all q: Queue, k: Natural, elem: T
*   (Q_MD1): multi_dequeue(zero, q) = q
*   (Q_MD2): multi_dequeue(k, emptyQueue) = emptyQueue
*   (Q_MD3): eq(k,zero) = false AND isEmpty(q) = false =>
             multi_dequeue(k, q) = multi_dequeue(pred(k), dequeue(q))
**END SPEC**

Agora, o teste de propriedade para (Q_MD1):

```python
# test_py_queue.py (continuação)

# Assume PyQueue has a method multi_dequeue implemented like this for the test:
# class PyQueue(Generic[T]):
#   ...
#   def multi_dequeue(self, k: int) -> PyQueue[T]:
#       if not (isinstance(k, int) and k >= 0):
#           raise ValueError("k must be a non-negative integer.")
#       if k == 0:
#           return self
#       
#       current_q_state = self
#       elements_dequeued = 0
#       # This is a conceptual loop for an immutable version.
#       # An efficient immutable version would reconstruct from remaining elements.
#       # For this test, we will assume a correct immutable multi_dequeue based on this logic.
#       # Simplistic immutable implementation based on to_list and re-creating:
#       if k >= self.size():
#           return PyQueue.empty_queue()
#       
#       all_elements = self.to_list_front_to_back()
#       remaining_elements = all_elements[k:]
#       # Rebuild _in_stack and _out_stack appropriately. A simple way is all to _in_stack
#       # but this isn't efficient or how the 2-stack queue is normally maintained.
#       # For testing the axiom q.multi_dequeue(0)==q, if k=0 returns self, it's fine.
#       # For k > 0, this test requires a full multi_dequeue implementation.
#       # Let's assume a more direct immutable construction for the test:
#       
#       temp_q = self
#       count = 0
#       while count < k and not temp_q.is_empty():
#           temp_q = temp_q.dequeue() # This uses the immutable dequeue
#           count += 1
#       return temp_q
#   ...

# Test for multi_dequeue(zero, q) = q
@given(q=st_py_queue)
def test_prop_multi_dequeue_zero_is_identity(q: PyQueue[int]):
    """
    Tests the property: multi_dequeue(zero, q) = q
    For the Python implementation: q.multi_dequeue(0) == q
    """
    # Assuming PyQueue has a method multi_dequeue(self, k: int)
    # For k=0, it should return the original queue (or an equivalent one).
    assert q.multi_dequeue(0) == q
```
O teste `test_prop_multi_dequeue_zero_is_identity` verifica que chamar `multi_dequeue` com `k=0` em qualquer fila `q` resulta na própria fila `q`, correspondendo ao axioma (Q_MD1). A implementação de `multi_dequeue` foi esboçada nos comentários para contextualizar o teste.

# 10.5 Análise de Complexidade Algorítmica

A implementação de `PyQueue[T]` usando duas instâncias de `PyStack[T]` (que por sua vez usam listas Python imutáveis) tem as seguintes características de complexidade, onde $N$ é o número total de elementos na fila ($N = N_{in} + N_{out}$):

1.  **`__init__`**: $O(1)$ (assume que pilhas são passadas como referência).
2.  **`empty_queue()`**: $O(1)$.
3.  **`is_empty()`**: $O(1)$.
4.  **`enqueue(self, value: T)`**: $O(N_{in})$ (devido ao `push` na `PyStack` imutável `_in_stack`). No pior caso, $O(N)$.
5.  **`_transfer_in_to_out()`**: Se implementado com `PyStack` imutáveis e loop de `pop`/`push`, pode ser $O(N_{in}^2)$. Se otimizado para $O(N_{in})$ (manipulando listas internas).
6.  **`front(self) -> T`**: Pior caso $O(N_{in}^2)$ (transferência ingênua) ou $O(N_{in})$ (transferência otimizada). Se `_out_stack` não vazia, $O(1)$. **Amortizado $O(1)$** se pilhas internas fossem mutáveis e eficientes e transferência otimizada.
7.  **`dequeue(self) -> PyQueue[T]`**: Pior caso $O(N_{in}^2 + N_{out})$ (transf. ingênua + pop imutável) ou $O(N_{in} + N_{out}) = O(N)$ (transf. otimizada + pop imutável). **Amortizado $O(1)$** nas mesmas condições ideais de `front`.
8.  **`size(self) -> int`**: $O(1)$.
9.  **`rear(self) -> T`**: $O(1)$.

**Conclusão da Análise de Complexidade:**
Com as atuais `PyStack` imutáveis (onde `push`/`pop` são $O(N_{stack})$), a implementação `PyQueue` não alcança o $O(1)$ amortizado ideal para `enqueue`/`dequeue`/`front`. Para isso, as `PyStack` internas precisariam ser mutáveis com operações $O(1)$ amortizado, e a transferência otimizada.

**Exercício:**

Se a classe `PyStack[T]` fosse implementada de forma *mutável* (modificando `self._elements` no local) de modo que `push`, `pop` (do topo) e `top` fossem todas operações $O(1)$ amortizado (como uma lista Python usada como pilha), qual seria a complexidade amortizada de `enqueue`, `dequeue`, e `front` na implementação `PyQueue[T]` de duas pilhas (assumindo que `_transfer_in_to_out` também é otimizada para ser $O(N_{in})$ usando as operações de pilha mutáveis)?

**Resolução:**

Assumindo `PyStack[T]` mutável com operações $O(1)$ amortizado e `_transfer_in_to_out()` otimizada para $O(N_{in})$ (fazendo $N_{in}$ pops e $N_{in}$ pushes $O(1)$ amortizado):

1.  **`enqueue(self, value: T)`:**
    *   Envolve um `push` na `_in_stack` (mutável): $O(1)$ amortizado.
    *   Se `PyQueue` retorna nova instância, copia as duas pilhas (referências $O(1)$ ou cópia de dados $O(N_{in})+O(N_{out})$). Se `PyQueue` é mutável, é $O(1)$ amortizado.
    *   Considerando a lógica FIFO e o retorno de nova instância apenas com a `_in_stack` modificada: a criação da nova `PyQueue` envolveria criar uma nova `_in_stack` (baseada na antiga + novo elemento, $O(N_{in})$ se a pilha é copiada, ou $O(1)$ se a pilha subjacente é copiada por referência e o push é nela) e reutilizar `_out_stack` (referência $O(1)$). Se a pilha `_in_stack` fosse copiada e depois `push` $O(N_{in})$. Se o push fosse feito em uma cópia da lista interna da `_in_stack`, ainda $O(N_{in})$.
    *   Para `enqueue` ser $O(1)$ amortizado, a `PyQueue` teria que ser mutável, ou as pilhas internas serem persistentes (compartilhando estrutura). Com a nossa estrutura atual de retornar `PyQueue(new_in_stack, self._out_stack)` e `PyStack`s imutáveis sendo copiadas no `push`, continua $O(N_{in})$.
    *   **Se `PyQueue` e `PyStack` fossem ambas mutáveis e eficientes:** `enqueue` seria **$O(1)$ amortizado**.

2.  **`dequeue(self)` (e `front(self)`):**
    *   `_transfer_in_to_out()`: Custa $O(N_{in})$ quando a transferência ocorre (pois faz $N_{in}$ pops da `_in_stack` e $N_{in}$ pushes para `_out_stack`, todos $O(1)$ amortizado).
    *   Após a transferência, `_out_stack.pop()` (ou `top()`) é $O(1)$ amortizado.
    *   **Análise Amortizada:** Uma transferência de $N$ elementos custa $O(N)$. Essas $N$ operações de `dequeue/front` subsequentes custam $O(1)$ cada. O custo total para as $N$ `dequeue/front`s que esvaziam a `_out_stack` é $O(N)$. Portanto, o custo por operação é $O(1)$.
    *   **Complexidade Amortizada para `dequeue` e `front`:** **$O(1)$ amortizado**.

Assim, com pilhas internas mutáveis e eficientes, a implementação de duas pilhas para `PyQueue` atinge o $O(1)$ amortizado para suas operações principais, mesmo que a `PyQueue` em si retorne novas instâncias (desde que a cópia das pilhas seja apenas de referências ou que as pilhas usadas na nova `PyQueue` sejam as modificadas).

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

Este capítulo introduziu o TAD `Queue[T]`, sua especificação algébrica formal parametrizada, e uma implementação Python genérica `PyQueue[T]` com tipagem estática, usando uma abordagem de duas pilhas. Demonstramos como os axiomas são cruciais para definir o comportamento FIFO e como eles podem ser traduzidos em testes de propriedade com Hypothesis para validar a implementação. A análise de complexidade destacou os custos e benefícios da implementação de duas pilhas, especialmente em relação à imutabilidade das pilhas subjacentes. O exercício prático estendeu o TAD `Stack[T]` (referenciado pela implementação de `PyQueue`) com `multipop` e seus testes, embora o exercício do capítulo `Queue` tenha focado em `rear`. O próximo capítulo, Capítulo 12, abordará o TAD `Deck[T]` (Deque, ou Double-Ended Queue), que generaliza tanto a pilha quanto a fila, permitindo inserções e remoções em ambas as extremidades, e apresentando seus próprios desafios de especificação e implementação.

---
## REFERÊNCIAS BIBLIOGRÁFICAS
AHO, A. V.; HOPCROFT, J. E.; ULLMAN, J. D. **Data Structures and Algorithms**. Reading, MA: Addison-Wesley, 1983.
*   *Resumo: Este texto clássico descreve filas (queues) como estruturas de dados fundamentais, suas operações FIFO e implementações comuns, como arrays circulares ou listas encadeadas, fornecendo uma base conceitual para o TAD Queue.*

CORMEN, T. H.; LEISERSON, C. E.; RIVEST, R. L.; STEIN, C. **Introduction to Algorithms**. 3. ed. Cambridge, MA: MIT Press, 2009.
*   *Resumo: Inclui uma discussão detalhada sobre filas, suas operações, e implementações usando arrays e listas. A análise de complexidade e as aplicações em algoritmos como BFS são relevantes para este capítulo.*

GOODRICH, M. T.; TAMASSIA, R.; GOLDWASSER, M. H. **Data Structures and Algorithms in Python**. Hoboken, NJ: Wiley, 2013.
*   *Resumo: Apresenta o TAD Fila e várias implementações em Python, incluindo o uso de adaptadores sobre listas Python ou a implementação de duas pilhas para eficiência amortizada. Diretamente relevante para as Seções 11.3 e 11.5.*

HOOD, R.; MELVILLE, R. Real-time queue operations in pure LISP. **Information Processing Letters**, v. 13, n. 2, p. 50-54, nov. 1981.
*   *Resumo: Um artigo clássico que descreve uma implementação funcional (imutável) eficiente de uma fila usando duas listas (análogas a duas pilhas), alcançando tempo real (pior caso) para algumas operações, relevante para discussões avançadas sobre implementações de filas funcionais.*

OKASAKI, C. **Purely Functional Data Structures**. Cambridge: Cambridge University Press, 1998.
*   *Resumo: Este livro seminal explora a implementação de estruturas de dados, incluindo filas, em um paradigma puramente funcional, com foco em persistência e eficiência amortizada. A implementação de filas com duas listas (pilhas) é um exemplo canônico discutido.*

PYTHON SOFTWARE FOUNDATION. **Python `collections.deque` documentation**. Disponível em: https://docs.python.org/3/library/collections.html#collections.deque. Acesso em: 10 ago. 2023.
*   *Resumo: A `collections.deque` do Python é uma fila de duas extremidades (deque) altamente otimizada, que pode ser usada para implementar filas FIFO eficientes. Conhecer esta estrutura é importante para o contexto prático de implementação em Python.*
---
