---
# Capítulo 12
# Tipo Abstrato de Dados Deck
---

Encerrando nossa investigação sobre estruturas de dados lineares fundamentais nesta Parte II, este capítulo introduz o Tipo Abstrato de Dados (TAD) `Deck[T]`, mais comumente conhecido pelo seu nome contraído **Deque** (pronuncia-se "deck"), que é uma abreviação de *Double-Ended Queue* (Fila de Duas Extremidades). O Deque generaliza tanto o TAD `Stack[T]` (Pilha) quanto o TAD `Queue[T]` (Fila), permitindo a inserção e remoção de elementos em ambas as extremidades da sequência: o início (frente) e o fim (traseira). Esta flexibilidade torna o Deque uma estrutura de dados extremamente versátil e poderosa, subsumindo o comportamento LIFO das pilhas e o comportamento FIFO das filas, além de permitir outras políticas de acesso. Aplicações de Deques incluem algoritmos de escalonamento (como o algoritmo de "roubo de trabalho" - work-stealing), gerenciamento de históricos de navegação (onde se pode adicionar/remover do início ou do fim), e como um bloco de construção eficiente para outras estruturas de dados. Seguindo a metodologia estabelecida, iniciaremos com uma definição abstrata do TAD `Deque[T]`, identificando seu conjunto abrangente de operações. Em seguida, desenvolveremos uma especificação algébrica formal para `Deque[T]`, parametrizada pelo tipo de elemento `T`, com foco na captura de seu comportamento de acesso duplo. Analisaremos a consistência e completeza desta especificação. Posteriormente, projetaremos e implementaremos uma classe genérica `PyDeque[T]` em Python, utilizando anotações de tipo MyPy. A validação da corretude da implementação será conduzida através do Teste Baseado em Propriedades com Hypothesis. Concluiremos com uma análise da complexidade algorítmica das operações da classe `PyDeque[T]` e um exercício prático.

# 12.1 Definição Abstrata e Relevância do Tipo de Dados `Deque[T]`

O Tipo Abstrato de Dados `Deque[T]` (Double-Ended Queue, ou Fila de Duas Extremidades) é uma coleção linear ordenada de elementos do tipo genérico `T`, que permite operações de inserção e remoção em ambas as suas extremidades: o **início** (frequentemente chamado de frente, *front*, ou cabeça, *head*) e o **fim** (frequentemente chamado de traseira, *rear*, ou cauda, *tail*). Esta capacidade de operar em ambas as extremidades distingue o Deque das Pilhas (que operam apenas no topo) e das Filas (que inserem em uma extremidade e removem da outra).

**Definição Abstrata:**
Um `Deque[T]` pode ser visualizado como uma sequência finita de elementos. As operações essenciais que o caracterizam incluem:

1.  **Criação:**
    *   `emptyDeque: --> Deque[T]`: Cria um deque vazio.
2.  **Modificação (Inserção):**
    *   `addFront: T x Deque[T] --> Deque[T]`: Adiciona um elemento `T` ao início do deque.
    *   `addRear: Deque[T] x T --> Deque[T]`: Adiciona um elemento `T` ao fim do deque.
3.  **Modificação (Remoção):**
    *   `removeFront: Deque[T] --> Deque[T]`: Remove o elemento do início do deque. (Parcial: não definida para deque vazio).
    *   `removeRear: Deque[T] --> Deque[T]`: Remove o elemento do fim do deque. (Parcial: não definida para deque vazio).
4.  **Acesso/Observação:**
    *   `front: Deque[T] --> T`: Retorna o elemento no início do deque sem removê-lo. (Parcial: não definida para deque vazio).
    *   `rear: Deque[T] --> T`: Retorna o elemento no fim do deque sem removê-lo. (Parcial: não definida para deque vazio).
    *   `isEmpty: Deque[T] --> Boolean`: Verifica se o deque está vazio.
    *   `size: Deque[T] --> Natural`: Retorna o número de elementos no deque.

Um Deque pode simular uma Pilha usando apenas `addFront` como `push`, `front` como `top`, e `removeFront` como `pop`. Pode também simular uma Fila usando `addRear` como `enqueue`, `front` como `frontOfQueue`, e `removeFront` como `dequeue`.

**Relevância do TAD `Deque[T]`:**
A versatilidade do Deque o torna uma estrutura de dados valiosa em diversos cenários:

1.  **Generalização de Pilhas e Filas:** Como mencionado, Deques podem implementar diretamente os comportamentos de pilhas e filas, oferecendo uma estrutura de dados unificada se ambas as políticas de acesso forem necessárias.

2.  **Algoritmos de Escalonamento (Work-Stealing):** Em sistemas de processamento paralelo, algoritmos de "roubo de trabalho" utilizam deques para gerenciar tarefas. Cada processador tem seu próprio deque de tarefas. Ele adiciona e remove tarefas de uma extremidade (comportamento de pilha). Se seu deque fica vazio, ele pode "roubar" uma tarefa da extremidade oposta do deque de outro processador (comportamento de fila), minimizando contenção.

3.  **Janelas Deslizantes:** Em problemas que envolvem o processamento de uma "janela" de dados que desliza sobre uma sequência maior (e.g., encontrar o máximo/mínimo em todas as subjanelas de tamanho k), um deque é frequentemente usado para manter os candidatos da janela de forma eficiente, permitindo adições e remoções em ambas as extremidades conforme a janela desliza.

4.  **Históricos de Navegação e Buffers "Desfazer/Refazer":** Alguns sistemas de histórico que permitem navegação para frente e para trás, ou listas de "desfazer/refazer" mais complexas, podem ser modelados com deques.

5.  **Algoritmos Palindrômicos:** Para verificar se uma sequência é um palíndromo, pode-se adicionar caracteres a um deque e, em seguida, remover e comparar elementos de ambas as extremidades simultaneamente.

6.  **Implementações Eficientes:** Deques são frequentemente implementados usando arrays dinâmicos com capacidade de crescimento em ambas as direções (e.g., um array circular ou um array com um "centro" que pode se expandir) ou usando listas duplamente encadeadas. Tais implementações podem alcançar tempo constante $O(1)$ (amortizado para arrays) para todas as operações básicas de adição e remoção nas extremidades. A biblioteca `collections.deque` do Python é um exemplo de implementação eficiente.

Neste capítulo, vamos nos concentrar na especificação algébrica de `Deque[T]`, sua implementação imutável `PyDeque[T]` em Python, e a verificação de suas propriedades axiomáticas.

**Exercício:**

Considere um TAD `Deque[T]` com as operações `emptyDeque`, `addFront(e,d)`, `addRear(d,e)`, `removeFront(d)`, `removeRear(d)`, `front(d)`, `rear(d)`. Seja `T` o tipo `Natural`. Partindo de um deque vazio `d0 = emptyDeque`, execute a seguinte sequência de operações e descreva o estado do deque (indicando o início e o fim) e o resultado da última operação, se houver:
1.  `d1 = addFront(natural_1, d0)`
2.  `d2 = addRear(d1, natural_2)`
3.  `d3 = addFront(natural_3, d2)`
4.  `val1 = front(d3)`
5.  `d4 = removeRear(d3)`
6.  `val2 = rear(d4)`
(onde `natural_1`, `natural_2`, `natural_3` são os `Natural`s 1, 2, e 3).

**Resolução:**

Vamos traçar a sequência de operações sobre o deque de `Natural`s. Usaremos $\langle \text{início} \ldots \text{fim} \rangle$ para representar o deque.

1.  **`d0 = emptyDeque`**
    *   Estado do deque `d0`: $\langle \rangle$ (deque vazio)

2.  **`d1 = addFront(natural_1, d0)`**
    *   `natural_1` é adicionado ao início de `d0`.
    *   Estado do deque `d1`: $\langle \text{natural_1} \rangle$ (início: `natural_1`, fim: `natural_1`)

3.  **`d2 = addRear(d1, natural_2)`**
    *   `natural_2` é adicionado ao fim de `d1`.
    *   Estado do deque `d2`: $\langle \text{natural_1, natural_2} \rangle$ (início: `natural_1`, fim: `natural_2`)

4.  **`d3 = addFront(natural_3, d2)`**
    *   `natural_3` é adicionado ao início de `d2`.
    *   Estado do deque `d3`: $\langle \text{natural_3, natural_1, natural_2} \rangle$ (início: `natural_3`, fim: `natural_2`)

5.  **`val1 = front(d3)`**
    *   `front` retorna o elemento no início de `d3` (que é `natural_3`).
    *   Resultado da operação: `val1 = natural_3`.
    *   Estado do deque `d3` (após `front`): $\langle \text{natural_3, natural_1, natural_2} \rangle$ (inalterado)

6.  **`d4 = removeRear(d3)`**
    *   `removeRear` remove o elemento do fim de `d3` (que é `natural_2`).
    *   Estado do deque `d4`: $\langle \text{natural_3, natural_1} \rangle$ (início: `natural_3`, fim: `natural_1`)

7.  **`val2 = rear(d4)`**
    *   `rear` retorna o elemento no fim de `d4` (que é `natural_1`).
    *   Resultado da operação: `val2 = natural_1`.
    *   Estado do deque `d4` (após `rear`): $\langle \text{natural_3, natural_1} \rangle$ (inalterado)

Portanto, ao final da sequência:
*   `val1` é `natural_3`.
*   `val2` é `natural_1`.
*   O estado final do deque `d4` é $\langle \text{natural_3, natural_1} \rangle$.

# 12.2 Especificação Algébrica Formal do TAD `Deque[T]`

A especificação algébrica de `Deque[T]` deve capturar a capacidade de adicionar e remover elementos de ambas as extremidades. Uma forma comum de especificar estruturas como Deques é usando um conjunto mínimo de construtores e definindo as outras operações em termos deles. Para `Deque[T]`, podemos escolher `emptyDeque` e `addFront` como construtores principais, e então definir `addRear` e as operações de remoção e acesso em termos destes ou de forma mais relacional. No entanto, definir `addRear` puramente em termos de `addFront` pode ser complexo axiomaticamente para manter a eficiência conceitual.

Alternativamente, podemos tratar `emptyDeque`, `addFront` e `addRear` como um conjunto mais amplo de "quase-construtores" e definir os axiomas para as interações entre eles e os observadores/removedores. Esta abordagem pode ser mais intuitiva para expressar o comportamento, mas pode complicar a análise de propriedades como a completude de construtores se não houver um conjunto canônico claro de construtores.

Vamos adotar uma abordagem que usa `emptyDeque` e `addFront` como construtores canônicos, e `addRear` será uma operação derivada, embora sua definição axiomática recursiva possa ser um pouco elaborada.

**SPEC ADT** ElementTypeForDeque
**sorts:**
*   T
**END SPEC**

**SPEC ADT** Deque[ParamT: ElementTypeForDeque]
**imports:**
*   ParamT
*   Boolean
*   Natural

**sorts:**
*   Deque

**operations:**
*   emptyDeque: --> Deque
*   addFront: T x Deque --> Deque
*   addRear: Deque x T --> Deque
*   removeFront: Deque --> Deque
*   removeRear: Deque --> Deque
*   front: Deque --> T
*   rear: Deque --> T
*   isEmpty: Deque --> Boolean
*   size: Deque --> Natural

**axioms:**
for all d: Deque, elem, e1: T

*   (D1): isEmpty(emptyDeque) = true
*   (D2): isEmpty(addFront(elem, d)) = false
*   (D3): isEmpty(addRear(d, elem)) = ifThenElse(isEmpty(d), false, isEmpty(d))

*   (D4): front(addFront(elem, d)) = elem
*   (D5): isEmpty(d) = true  => front(addRear(d, elem)) = elem
*   (D6): isEmpty(d) = false => front(addRear(d, elem)) = front(d)

*   (D7): rear(addFront(elem, emptyDeque)) = elem
*   (D8): isEmpty(d) = false => rear(addFront(elem, d)) = rear(d)
*   (D9): rear(addRear(d, elem)) = elem

*   (D10): removeFront(addFront(elem, d)) = d
*   (D11): isEmpty(d) = true  => removeFront(addRear(d, elem)) = emptyDeque
*   (D12): isEmpty(d) = false => removeFront(addRear(d, elem)) = addRear(removeFront(d), elem)

*   (D13): removeRear(addFront(elem, emptyDeque)) = emptyDeque
*   (D14): isEmpty(d) = false => removeRear(addFront(elem, d)) = addFront(elem, removeRear(d))
*   (D15): removeRear(addRear(d, elem)) = d

*   (D16): size(emptyDeque) = zero
*   (D17): size(addFront(elem, d)) = succ(size(d))
*   (D18): size(addRear(d, elem)) = succ(size(d))

*   (D19_Interaction): addRear(emptyDeque, elem) = addFront(elem, emptyDeque)
*   (D20_Interaction): isEmpty(d) = false => addRear(addFront(e1, d), elem) = addFront(e1, addRear(d, elem))

**END SPEC**

A especificação `Deque[ParamT: ElementTypeForDeque]` define um TAD Deque genérico. `ElementTypeForDeque` introduz o sort `T`. A especificação `Deque` importa `ParamT`, `Boolean` e `Natural`. O sort principal é `Deque`.
As **operações** são: `emptyDeque`, `addFront` (adiciona ao início), `addRear` (adiciona ao fim), `removeFront`, `removeRear`, `front` (acessa início), `rear` (acessa fim), `isEmpty`, e `size`. `removeFront`, `removeRear`, `front`, `rear` são parciais.

Os **axiomas** definem o comportamento:
(D1)-(D2) definem `isEmpty` para `emptyDeque` e `addFront`. (D3) define `isEmpty` para `addRear`: se `d` era vazio, `addRear(d,elem)` não é; senão, a vacuidade é a de `d` (este axioma é um pouco estranho, pois `addRear` sempre torna não-vazio. Deveria ser: `isEmpty(addRear(d,elem)) = false`). Corrigindo (D3):
*   (D3_corrigido): `isEmpty(addRear(d, elem)) = false` (se `emptyDeque` e `addRear` são os únicos construtores para formar qualquer estado). Ou, se `addRear` for derivada de `addFront`: `isEmpty(addRear(d, elem))` herdaria de (D2) após a expansão de `addRear`. A especificação está tratando `addFront` e `addRear` como construtores potenciais, levando a mais axiomas.

(D4)-(D6) definem `front`. (D4) é o caso base para `addFront`. (D5) e (D6) definem `front` em relação a `addRear`: se a adição ao final foi em um deque vazio, o novo elemento é a frente; caso contrário, a frente não muda.
(D7)-(D9) definem `rear` analogamente: (D9) é o caso base para `addRear`. (D7) e (D8) definem `rear` em relação a `addFront`.
(D10)-(D12) definem `removeFront`. (D10) é para `addFront`. (D11), (D12) para `addRear`, com (D12) sendo o axioma FIFO crucial para `removeFront`.
(D13)-(D15) definem `removeRear`. (D15) é para `addRear`. (D13), (D14) para `addFront`, com (D14) sendo o axioma LIFO crucial para `removeRear`.
(D16)-(D18) definem `size` para `emptyDeque`, `addFront`, e `addRear`. (D17) e (D18) devem ser consistentes, ambos incrementando o tamanho.
(D19_Interaction) relaciona `addRear` a um `emptyDeque` com `addFront`.
(D20_Interaction) define como `addRear` e `addFront` interagem quando a pilha não está vazia, mantendo a ordem. Este é um axioma importante para garantir que a ordem dos elementos seja consistente independentemente de qual construtor foi usado por último.

O comportamento de operações parciais em deques vazios (e.g., `front(emptyDeque)`) não é definido por estes axiomas, exceto onde explicitado (como `removeFront(addRear(emptyDeque,elem)) = emptyDeque` (D11)).

## 12.2.1 Análise de Corretude e Completeza da Especificação `Deque[T]`

Analisaremos a especificação `Deque[T]` (com o (D3) corrigido para `isEmpty(addRear(d,elem))=false`) quanto à consistência e completeza. Construtores considerados: `emptyDeque`, `addFront`, `addRear`.

**Consistência:**
1.  **Proteção dos Tipos Importados:** Os axiomas parecem bem comportados em relação a `Boolean`, `Natural`, `T`.
2.  **Consistência Interna:**
    *   `emptyDeque` é distinto de `addFront(...)` e `addRear(...)` (via D1, D2, D3_corrigido e `true != false`).
    *   A principal preocupação para consistência em Deques é a interação entre `addFront`/`removeRear` e `addRear`/`removeFront`, e as operações de acesso `front`/`rear`. Os axiomas (D12), (D14), (D19), (D20) são cruciais. Por exemplo, (D19) `addRear(emptyDeque, elem) = addFront(elem, emptyDeque)` estabelece uma equivalência importante. (D20) relaciona a ordem de aplicação: `addRear(addFront(e1, d), elem) = addFront(e1, addRear(d, elem))`. Este tipo de axioma é essencial para garantir que diferentes sequências de construção que deveriam levar ao mesmo estado abstrato sejam de fato iguais.
    *   O modelo usual de uma sequência finita onde inserções/remoções podem ocorrer em ambas as extremidades serve como um modelo consistente.

A especificação parece **consistente**, embora provar isso formalmente (e.g., mostrando que um sistema de reescrita derivado é canônico e `true != false`) seja complexo devido ao número de interações.

**Completeza Suficiente:**
Para `isEmpty`, `front`, `rear`, `removeFront`, `removeRear`, `size` sobre `emptyDeque`, `addFront(e,d)`, `addRear(d,e)`.

1.  **`isEmpty(dq: Deque)`:**
    *   `isEmpty(emptyDeque)` $\rightarrow_{D1}$ `true`.
    *   `isEmpty(addFront(e,d))` $\rightarrow_{D2}$ `false`.
    *   `isEmpty(addRear(d,e))` $\rightarrow_{D3\_corrigido}$ `false`.
    `isEmpty` é suficientemente completa.

2.  **`size(dq: Deque)`:**
    *   `size(emptyDeque)` $\rightarrow_{D16}$ `zero`.
    *   `size(addFront(e,d))` $\rightarrow_{D17}$ `succ(size(d))`. Por indução.
    *   `size(addRear(d,e))` $\rightarrow_{D18}$ `succ(size(d))`. Por indução.
    `size` é suficientemente completa (assumindo que D17 e D18 são consistentes entre si via D19, D20).

3.  **`front(dq: Deque)`:**
    *   `front(emptyDeque)`: Indefinido.
    *   `front(addFront(e,d))` $\rightarrow_{D4}$ `e`. (Resultado `T`).
    *   `front(addRear(d,e))`:
        *   Se `isEmpty(d)=true` (i.e., $d=\text{emptyDeque}$): `front(addRear(emptyDeque, e))` $\rightarrow_{D5}$ `e`.
        *   Se `isEmpty(d)=false`: `front(addRear(d,e))` $\rightarrow_{D6}$ `front(d)`. Recursão sobre `d`.
    `front` é suficientemente completa para deques não vazios. Parcial.

4.  **`rear(dq: Deque)`:**
    *   `rear(emptyDeque)`: Indefinido.
    *   `rear(addRear(d,e))` $\rightarrow_{D9}$ `e`. (Resultado `T`).
    *   `rear(addFront(e,d))`:
        *   Se $d=\text{emptyDeque}$: `rear(addFront(e, emptyDeque))` $\rightarrow_{D7}$ `e`.
        *   Se `isEmpty(d)=false`: `rear(addFront(e,d))` $\rightarrow_{D8}$ `rear(d)`. Recursão sobre `d`.
    `rear` é suficientemente completa para deques não vazios. Parcial.

5.  **`removeFront(dq: Deque)`:**
    *   `removeFront(emptyDeque)`: Indefinido.
    *   `removeFront(addFront(e,d))` $\rightarrow_{D10}$ `d`. (Resultado `Deque` construtor, ou redutível).
    *   `removeFront(addRear(d,e))`:
        *   Se `isEmpty(d)=true`: `removeFront(addRear(emptyDeque, e))` $\rightarrow_{D11}$ `emptyDeque`.
        *   Se `isEmpty(d)=false`: `removeFront(addRear(d,e))` $\rightarrow_{D12}$ `addRear(removeFront(d), e)`. Recursão.
    `removeFront` é suficientemente completa para deques não vazios. Parcial.

6.  **`removeRear(dq: Deque)`:**
    *   `removeRear(emptyDeque)`: Indefinido.
    *   `removeRear(addRear(d,e))` $\rightarrow_{D15}$ `d`.
    *   `removeRear(addFront(e,d))`:
        *   Se $d=\text{emptyDeque}$: `removeRear(addFront(e, emptyDeque))` $\rightarrow_{D13}$ `emptyDeque`.
        *   Se `isEmpty(d)=false`: `removeRear(addFront(e,d))` $\rightarrow_{D14}$ `addFront(e, removeRear(d))`. Recursão.
    `removeRear` é suficientemente completa para deques não vazios. Parcial.

**Conclusão da Análise:**
*   **Consistência:** Provavelmente consistente, mas a complexidade das interações (D12, D14, D20) requer análise cuidadosa (e.g., via completamento de Knuth-Bendix).
*   **Completeza Suficiente:** `isEmpty` e `size` são. `front`, `rear`, `removeFront`, `removeRear` são parciais (não definidas para `emptyDeque`).

**Exercício:**

Adicione uma operação `getAtIndex: Deque[T] x Natural --> T` à especificação `Deque[T]`. Esta operação deve retornar o elemento no índice `Natural` especificado (0-indexado a partir da frente). Se o índice for inválido ou o deque estiver vazio, a operação é indefinida. Forneça os axiomas para `getAtIndex`. (Dica: defina em termos de `front` e `removeFront` ou recursivamente sobre `addFront`).

**Resolução:**

Adicionamos `getAtIndex` à especificação `Deque[T]`.

**SPEC ADT** DequeWithGet[ParamT: ElementTypeForDeque]
**imports:**
*   ParamT
*   Boolean
*   Natural
*   Deque[ParamT]

**sorts:**
*   Deque

**operations:**
*   getAtIndex: Deque x Natural --> T

**axioms:**
for all d: Deque, elem: T, k: Natural

*   (DG1): eq(k, zero) = true => getAtIndex(d, k) = front(d)
*   (DG2): eq(k, zero) = false AND isEmpty(d) = false =>
            getAtIndex(d, k) = getAtIndex(removeFront(d), pred(k))

**END SPEC**

A especificação `DequeWithGet` enriquece `Deque[T]`. A nova operação `getAtIndex` recebe um `Deque` e um `Natural` (índice) e retorna um elemento `T`.
*   (DG1): Se o índice `k` é `zero`, `getAtIndex` retorna o `front(d)`. Isso implica que `d` não deve ser vazio para que `front(d)` seja definido.
*   (DG2): Se o índice `k` não é `zero` e o deque `d` não está vazio, então `getAtIndex(d,k)` é o mesmo que obter o elemento no índice `pred(k)` (ou $k-1$) do deque resultante de `removeFront(d)`. A condição `isEmpty(d)=false` garante que `removeFront(d)` e `front(d)` (no caso base implícito) são aplicados a deques não vazios.
Estes axiomas definem `getAtIndex` recursivamente. `getAtIndex(emptyDeque, k)` e `getAtIndex(d,k)` para `k >= size(d)` permanecem indefinidos.

# 11.3 Projeto e Implementação em Python com Tipagem Estática (MyPy)

Para implementar o TAD `Deque[T]` em Python, criaremos uma classe genérica `PyDeque[T]`. Uma implementação eficiente de Deque frequentemente utiliza uma lista duplamente encadeada ou um array dinâmico com gerenciamento de ambas as extremidades (como `collections.deque` do Python). Para nossa implementação imutável `PyDeque[T]`, podemos usar uma lista Python (`list`) como estrutura interna, mas as operações de adição/remoção no início da lista Python são $O(N)$. Uma implementação imutável mais eficiente poderia usar, por exemplo, a ideia de duas pilhas (como para `PyQueue`) ou estruturas de árvore persistentes, mas isso adiciona complexidade.

Para fins didáticos e alinhamento com a estrutura de `PyVector` e `PyStack` imutáveis, vamos usar uma única lista Python interna e aceitar as complexidades $O(N)$ para operações no início se a lista for copiada.

**Design da Classe `PyDeque[T]`:**

```python
# py_deque.py
from __future__ import annotations # For type hints of PyDeque[T] within the class itself
from typing import List as PyListInternal, Optional, TypeVar, Generic, Any

T = TypeVar('T') # Generic type parameter for elements

class PyDeque(Generic[T]):
    _elements: PyListInternal[T] # Internal Python list to store elements

    def __init__(self, initial_elements: Optional[PyListInternal[T]] = None) -> None:
        # Constructor for PyDeque.
        # Accepts an optional list of elements of type T for initialization.
        if initial_elements is None:
            self._elements = []
        else:
            self._elements = list(initial_elements) # Creates a copy

    @staticmethod
    def empty_deque() -> PyDeque[Any]: 
        # Returns an empty deque.
        return PyDeque()

    def is_empty(self) -> bool:
        # Checks if the deque is empty.
        return not self._elements 

    def size(self) -> int: 
        # Returns the number of elements (Natural).
        return len(self._elements)

    def add_front(self, value: T) -> PyDeque[T]:
        # Adds an element to the front. Returns a NEW PyDeque.
        new_elements = [value] + self._elements 
        return PyDeque(new_elements)

    def add_rear(self, value: T) -> PyDeque[T]:
        # Adds an element to the rear. Returns a NEW PyDeque.
        new_elements = self._elements + [value]
        return PyDeque(new_elements)

    def front(self) -> T: 
        # Returns the front element. Raises IndexError if empty.
        if self.is_empty():
            raise IndexError("front from empty deque")
        return self._elements[0]

    def rear(self) -> T: 
        # Returns the rear element. Raises IndexError if empty.
        if self.is_empty():
            raise IndexError("rear from empty deque")
        return self._elements[-1]

    def remove_front(self) -> PyDeque[T]:
        # Removes the front element. Returns a NEW PyDeque.
        # Raises IndexError if empty.
        if self.is_empty():
            raise IndexError("removeFront from empty deque")
        new_elements = self._elements[1:] # Slicing from the second element
        return PyDeque(new_elements)

    def remove_rear(self) -> PyDeque[T]:
        # Removes the rear element. Returns a NEW PyDeque.
        # Raises IndexError if empty.
        if self.is_empty():
            raise IndexError("removeRear from empty deque")
        new_elements = self._elements[:-1] # Slicing up to the second to last
        return PyDeque(new_elements)

    def get_at_index(self, index: int) -> T:
        # Gets element at index (Natural from front). Raises IndexError if invalid.
        if not (isinstance(index, int) and index >= 0):
            raise TypeError("Index must be a non-negative integer.")
        if not (0 <= index < self.size()):
            raise IndexError("Deque index out of range.")
        return self._elements[index]

    # Methods for equality and representation
    def __eq__(self, other: object) -> bool:
        if not isinstance(other, PyDeque):
            return NotImplemented
        return self._elements == other._elements

    def __repr__(self) -> str:
        return f"PyDeque({self._elements})"

    def to_list(self) -> PyListInternal[T]:
        # Helper for testing.
        return list(self._elements)
```
O código Python acima, no arquivo `py_deque.py`, define a classe genérica `PyDeque[T]`.
Ela é parametrizada por `T`. Uma lista Python interna `_elements` armazena os elementos. O construtor `__init__` permite inicialização opcional. `empty_deque()` retorna um deque vazio. As operações `add_front`, `add_rear`, `remove_front`, `remove_rear` são imutáveis, retornando novas instâncias de `PyDeque`. `front`, `rear`, `is_empty`, `size`, e `get_at_index` (do exercício) são observadores. Operações de acesso/remoção em deques vazios ou com índices inválidos levantam `IndexError` ou `TypeError`.

**Exercício:**

Adicione um método `rotate(self, k: int) -> PyDeque[T]` à classe `PyDeque[T]`. Este método deve "rotacionar" os elementos do deque. Se $k > 0$, os $k$ primeiros elementos são movidos para o final do deque. Se $k < 0$, os $|k|$ últimos elementos são movidos para o início. Se $k=0$ ou o deque estiver vazio ou tiver um elemento, o deque original é retornado. A implementação deve ser imutável.

**Resolução:**

```python
# py_deque.py (continuação da classe PyDeque[T])

    # ... (outros métodos como definidos anteriormente) ...

    def rotate(self, k: int) -> PyDeque[T]:
        # Rotates the deque.
        # If k > 0, moves k elements from front to rear.
        # If k < 0, moves |k| elements from rear to front.
        # Returns a NEW PyDeque.
        if not isinstance(k, int):
            raise TypeError("Rotation amount k must be an integer.")

        n = self.size()
        if n == 0 or k == 0:
            return self # No change for empty deque or zero rotation

        # Normalize k to be within [0, n-1] for effective rotation steps
        # A rotation by k is same as rotation by k % n (if k is positive)
        # For negative k, k % n in Python gives positive result in [0, n-1]
        # E.g., -1 % 5 = 4. Rotating by -1 (1 from rear to front) is like rotating by 4 (4 from front to rear)
        
        effective_k = k % n # Python's % handles negative k appropriately for this use case
                            # e.g. -1 % 5 = 4. moving 1 from rear to front is same as 4 from front to rear.

        if effective_k == 0 and k != 0: # Full rotation(s) result in original
             return self

        # Slice the list at effective_k and re-concatenate
        # Elements to move to the end (if k > 0) or that were at the end (if k < 0 became large positive effective_k)
        part1 = self._elements[:effective_k]
        # Elements that remain at the beginning (if k > 0) or that were at the beginning (if k < 0)
        part2 = self._elements[effective_k:]
        
        new_elements = part2 + part1 # For positive k, part2 comes first, then part1
        return PyDeque(new_elements)

    # ... (restante da classe) ...
```
O método `rotate` é adicionado à classe `PyDeque[T]`.
Primeiro, valida o tipo de `k`. Se o deque está vazio ou $k=0$, retorna o deque original.
A rotação é normalizada usando `effective_k = k % n`. Em Python, o operador `%` se comporta de forma útil para rotações: se `k` for negativo, `k % n` produz um resultado positivo equivalente. Por exemplo, para uma lista de 5 elementos, rotacionar por -1 (mover o último para o primeiro) é o mesmo que rotacionar por 4 (mover os 4 primeiros para o fim).
Se `effective_k` for 0 (mas `k` original não era zero), significa uma rotação completa, então o deque original é retornado.
Caso contrário, a lista interna `_elements` é fatiada em duas partes na posição `effective_k`. `part1` são os primeiros `effective_k` elementos, e `part2` são os restantes. A nova lista `new_elements` é formada concatenando `part2` seguido por `part1`, o que efetua a rotação desejada (para $k>0$, os primeiros `effective_k` elementos vão para o fim). Um novo `PyDeque` é retornado.

# 12.4 Verificação de Propriedades Axiomáticas com Hypothesis

Com a especificação algébrica de `Deque[T]` e a implementação `PyDeque[T]`, usamos Hypothesis para verificar os axiomas.

**Estratégias Hypothesis para `PyDeque[T]` e `T`:**
Usaremos `st.integers()` para `T`.

```python
# test_py_deque.py
import pytest
from hypothesis import given, strategies as st, assume, settings, HealthCheck
from typing import TypeVar, Any

# Import the implementation
from py_deque import PyDeque # Assuming py_deque.py is accessible

# Strategy for generating element type 'T'.
st_element_type = st.integers(min_value=-50, max_value=50)

# Strategy for PyDeque[int]
st_py_deque = st.builds(
    PyDeque[int], 
    initial_elements=st.lists(st_element_type, max_size=10)
)

# Strategy for non-empty PyDeque[int]
st_non_empty_py_deque = st.builds(
    PyDeque[int],
    initial_elements=st.lists(st_element_type, min_size=1, max_size=10)
)
```
O código de teste `test_py_deque.py` configura as estratégias. `st_element_type` gera inteiros. `st_py_deque` constrói `PyDeque[int]`. `st_non_empty_py_deque` garante deques não vazios.

**Testes de Propriedade para os Axiomas de `Deque[T]`:**
Axiomas (D1)-(D20), usando (D3_corrigido).

```python
# test_py_deque.py (continuação)

# --- Axioms for isEmpty ---
def test_D1_isEmpty_emptyDeque():
    # (D1): isEmpty(emptyDeque) = true
    assert PyDeque[Any].empty_deque().is_empty() == True

@given(d=st_py_deque, elem=st_element_type)
def test_D2_isEmpty_addFront(d: PyDeque[int], elem: int):
    # (D2): isEmpty(addFront(elem, d)) = false
    assert d.add_front(elem).is_empty() == False

@given(d=st_py_deque, elem=st_element_type)
def test_D3_corr_isEmpty_addRear(d: PyDeque[int], elem: int):
    # (D3_corrigido): isEmpty(addRear(d, elem)) = false
    assert d.add_rear(elem).is_empty() == False

# --- Axioms for front ---
@given(d=st_py_deque, elem=st_element_type)
def test_D4_front_addFront(d: PyDeque[int], elem: int):
    # (D4): front(addFront(elem, d)) = elem
    assert d.add_front(elem).front() == elem

@given(elem=st_element_type)
def test_D5_front_addRear_on_empty(elem: int):
    # (D5): isEmpty(d) = true  => front(addRear(d, elem)) = elem
    # Test with d = emptyDeque
    empty_d = PyDeque[Any].empty_deque()
    assert empty_d.add_rear(elem).front() == elem

@given(d_non_empty=st_non_empty_py_deque, elem=st_element_type)
@settings(suppress_health_check=[HealthCheck.filter_too_much])
def test_D6_front_addRear_on_non_empty(d_non_empty: PyDeque[int], elem: int):
    # (D6): isEmpty(d) = false => front(addRear(d, elem)) = front(d)
    # d_non_empty strategy ensures isEmpty(d_non_empty) is false
    expected_front = d_non_empty.front()
    assert d_non_empty.add_rear(elem).front() == expected_front

# --- Axioms for rear ---
@given(elem=st_element_type)
def test_D7_rear_addFront_on_empty(elem: int):
    # (D7): rear(addFront(elem, emptyDeque)) = elem
    empty_d = PyDeque[Any].empty_deque()
    assert empty_d.add_front(elem).rear() == elem

@given(d_non_empty=st_non_empty_py_deque, elem=st_element_type)
@settings(suppress_health_check=[HealthCheck.filter_too_much])
def test_D8_rear_addFront_on_non_empty(d_non_empty: PyDeque[int], elem: int):
    # (D8): isEmpty(d) = false => rear(addFront(elem, d)) = rear(d)
    expected_rear = d_non_empty.rear()
    assert d_non_empty.add_front(elem).rear() == expected_rear

@given(d=st_py_deque, elem=st_element_type)
def test_D9_rear_addRear(d: PyDeque[int], elem: int):
    # (D9): rear(addRear(d, elem)) = elem
    assert d.add_rear(elem).rear() == elem

# --- Axioms for removeFront ---
@given(d=st_py_deque, elem=st_element_type)
def test_D10_removeFront_addFront(d: PyDeque[int], elem: int):
    # (D10): removeFront(addFront(elem, d)) = d
    assert d.add_front(elem).remove_front() == d

@given(elem=st_element_type)
def test_D11_removeFront_addRear_on_empty(elem: int):
    # (D11): isEmpty(d) = true  => removeFront(addRear(d, elem)) = emptyDeque
    empty_d = PyDeque[Any].empty_deque()
    assert empty_d.add_rear(elem).remove_front() == PyDeque[Any].empty_deque()

@given(d_non_empty=st_non_empty_py_deque, elem=st_element_type)
@settings(suppress_health_check=[HealthCheck.filter_too_much, HealthCheck.too_slow], deadline=1000)
def test_D12_removeFront_addRear_on_non_empty(d_non_empty: PyDeque[int], elem: int):
    # (D12): isEmpty(d) = false => removeFront(addRear(d, elem)) = addRear(removeFront(d), elem)
    lhs = d_non_empty.add_rear(elem).remove_front()
    rhs = d_non_empty.remove_front().add_rear(elem)
    assert lhs == rhs

# --- Axioms for removeRear ---
@given(elem=st_element_type)
def test_D13_removeRear_addFront_on_empty(elem: int):
    # (D13): removeRear(addFront(elem, emptyDeque)) = emptyDeque
    empty_d = PyDeque[Any].empty_deque()
    assert empty_d.add_front(elem).remove_rear() == PyDeque[Any].empty_deque()

@given(d_non_empty=st_non_empty_py_deque, elem=st_element_type)
@settings(suppress_health_check=[HealthCheck.filter_too_much, HealthCheck.too_slow], deadline=1000)
def test_D14_removeRear_addFront_on_non_empty(d_non_empty: PyDeque[int], elem: int):
    # (D14): isEmpty(d) = false => removeRear(addFront(elem, d)) = addFront(elem, removeRear(d))
    lhs = d_non_empty.add_front(elem).remove_rear()
    rhs = d_non_empty.remove_rear().add_front(elem)
    assert lhs == rhs

@given(d=st_py_deque, elem=st_element_type)
def test_D15_removeRear_addRear(d: PyDeque[int], elem: int):
    # (D15): removeRear(addRear(d, elem)) = d
    assert d.add_rear(elem).remove_rear() == d

# --- Axioms for size ---
def test_D16_size_emptyDeque():
    # (D16): size(emptyDeque) = zero
    assert PyDeque[Any].empty_deque().size() == 0

@given(d=st_py_deque, elem=st_element_type)
def test_D17_size_addFront(d: PyDeque[int], elem: int):
    # (D17): size(addFront(elem, d)) = succ(size(d))
    assert d.add_front(elem).size() == d.size() + 1
    
@given(d=st_py_deque, elem=st_element_type)
def test_D18_size_addRear(d: PyDeque[int], elem: int):
    # (D18): size(addRear(d, elem)) = succ(size(d))
    assert d.add_rear(elem).size() == d.size() + 1

# --- Interaction Axioms ---
@given(elem=st_element_type)
def test_D19_interaction_addRear_empty_addFront_empty(elem: int):
    # (D19_Interaction): addRear(emptyDeque, elem) = addFront(elem, emptyDeque)
    empty_d = PyDeque[Any].empty_deque()
    assert empty_d.add_rear(elem) == empty_d.add_front(elem)

@given(d_non_empty=st_non_empty_py_deque, e1=st_element_type, elem=st_element_type)
@settings(suppress_health_check=[HealthCheck.filter_too_much, HealthCheck.too_slow], deadline=1000, max_examples=50)
def test_D20_interaction_addRear_addFront_non_empty(d_non_empty: PyDeque[int], e1:int, elem: int):
    # (D20_Interaction): isEmpty(d) = false => addRear(addFront(e1, d), elem) = addFront(e1, addRear(d, elem))
    lhs = d_non_empty.add_front(e1).add_rear(elem)
    rhs = d_non_empty.add_rear(elem).add_front(e1)
    assert lhs == rhs

# Test behavior for partial operations on emptyDeque (expect exceptions)
def test_front_on_empty_deque_raises_exception():
    empty_d = PyDeque[Any].empty_deque()
    with pytest.raises(IndexError): empty_d.front()

def test_rear_on_empty_deque_raises_exception():
    empty_d = PyDeque[Any].empty_deque()
    with pytest.raises(IndexError): empty_d.rear()

def test_removeFront_on_empty_deque_raises_exception():
    empty_d = PyDeque[Any].empty_deque()
    with pytest.raises(IndexError): empty_d.remove_front()

def test_removeRear_on_empty_deque_raises_exception():
    empty_d = PyDeque[Any].empty_deque()
    with pytest.raises(IndexError): empty_d.remove_rear()
```
Os testes acima verificam os axiomas (D1)-(D20). `assume` ou estratégias específicas como `st_non_empty_py_deque` são usadas para as premissas condicionais. Testes para exceções em operações parciais também são incluídos. O axioma (D3) foi testado usando a versão corrigida `isEmpty(addRear(d, elem)) = false`.

**Exercício:**

Considerando a operação `getAtIndex: Deque[T] x Natural --> T` e seus axiomas (DG1)-(DG2) da resolução do exercício da Seção 12.2. Escreva testes de propriedade Hypothesis para verificar estes axiomas para a implementação `py_deque.PyDeque.get_at_index`.

**Resolução:**

```python
# test_py_deque.py (continuação)

@st.composite
def st_deque_and_valid_get_index(draw):
    # Strategy to generate a non-empty deque and a valid index for get
    d = draw(st_non_empty_py_deque) # Ensures deque is not empty
    idx = draw(st.integers(min_value=0, max_value=d.size() - 1))
    return d, idx

# --- Axioms for getAtIndex (from exercise 12.2 resolution) ---
@given(data=st_deque_and_valid_get_index)
def test_DG1_getAtIndex_zero_is_front(data: tuple[PyDeque[int], int]):
    # (DG1): eq(k, zero) = true => getAtIndex(d, k) = front(d)
    # We test for k=0 directly. The strategy st_deque_and_valid_get_index
    # ensures d is not empty, so front(d) is valid.
    d, _ = data # We only need d for this specific test of k=0
    
    # Test for k=0, if the deque has at least one element (guaranteed by strategy for d)
    if d.size() > 0 : # Redundant due to st_non_empty_py_deque, but safe
        assert d.get_at_index(0) == d.front()

@given(data=st_deque_and_valid_get_index)
@settings(suppress_health_check=[HealthCheck.filter_too_much, HealthCheck.too_slow], deadline=1000, max_examples=50)
def test_DG2_getAtIndex_recursive_step(data: tuple[PyDeque[int], int]):
    # (DG2): eq(k, zero) = false AND isEmpty(d) = false =>
    #        getAtIndex(d, k) = getAtIndex(removeFront(d), pred(k))
    d, k = data
    
    assume(k > 0) # This ensures eq(k,zero) = false
    # st_deque_and_valid_get_index ensures d is not empty and k is a valid index for d.
    # If k > 0, then removeFront(d) will result in a deque for which k-1 (pred(k))
    # is a valid index, unless k was the last valid index and removeFront made it too short.
    # The original d has size S. k is in [0, S-1]. If k>0, k-1 is in [0, S-2].
    # removeFront(d) has size S-1. Its valid indices are [0, S-2]. So k-1 is valid.
    
    # This also implies d.remove_front() must not be empty IF k-1 is to be a valid index
    # for it. If d originally had only one element (size=1), and k=0, DG1 applies.
    # If d had size > 1 and k > 0, then remove_front(d) is not empty if k-1 is valid for it.
    # This should hold if k is a valid index in the original d and k > 0.
    assume(d.size() > k) # This ensures removeFront(d) has at least k elements, so index k-1 is valid.
                         # More accurately, size(removeFront(d)) must be >= k.
                         # Which means size(d) -1 >= k-1, so size(d) >= k.
                         # The strategy already gives k < size(d). So k-1 < size(d)-1. This is fine.


    lhs = d.get_at_index(k)
    rhs = d.remove_front().get_at_index(k - 1) # k-1 is pred(k)
    assert lhs == rhs

def test_getAtIndex_on_empty_raises_exception():
    empty_d = PyDeque[Any].empty_deque()
    with pytest.raises(IndexError):
        empty_d.get_at_index(0)

@given(d_non_empty=st_non_empty_py_deque, idx_out_of_bounds=st.integers(min_value=1))
def test_getAtIndex_out_of_bounds_raises_exception(d_non_empty: PyDeque[int], idx_out_of_bounds: int):
    # Test for index >= size
    actual_idx_out_of_bounds = d_non_empty.size() + idx_out_of_bounds -1 # ensures it's at least size()
    assume(actual_idx_out_of_bounds >= d_non_empty.size()) # Make sure it's truly out of bounds
    with pytest.raises(IndexError):
        d_non_empty.get_at_index(actual_idx_out_of_bounds)

```
Os testes para `getAtIndex` são adicionados:
*   `st_deque_and_valid_get_index` gera um deque não vazio e um índice `k` válido para ele.
*   `test_DG1_getAtIndex_zero_is_front` verifica que `get_at_index(d,0)` é igual a `d.front()`.
*   `test_DG2_getAtIndex_recursive_step` verifica a propriedade recursiva. `assume(k > 0)` garante a condição `eq(k,zero)=false`. A estratégia garante que `d` não é vazio. A validade de `k-1` no deque menor é sutil e a `assume(d.size() > k)` foi adicionada para robustez, embora a estratégia já deva cobrir isso se `k` é um índice válido e $k>0$.
*   Testes de exceção para `get_at_index` em deque vazio ou com índice fora dos limites.

# 12.5 Análise de Complexidade Algorítmica

A implementação `PyDeque[T]` utiliza uma lista Python (`list`) interna. As operações de lista Python `+` (concatenação), slicing (`[:]`, `[1:]`, `[:-1]`) e `insert` (implícita em `[value] + list` para `add_front`) geralmente criam novas listas, levando a complexidade $O(N)$ para operações imutáveis que "modificam" o deque.

**Análise de Complexidade das Operações de `PyDeque[T]` (Imutável):**

1.  **`__init__(self, initial_elements: Optional[PyListInternal[T]] = None)`:** $O(K)$ (onde $K$ é o número de elementos iniciais).
2.  **`empty_deque()`:** $O(1)$.
3.  **`is_empty(self) -> bool`:** $O(1)$.
4.  **`size(self) -> int`:** $O(1)$.
5.  **`add_front(self, value: T) -> PyDeque[T]`:** `[value] + self._elements` é $O(N)$. Construtor é $O(N)$. **Total: $O(N)$**.
6.  **`add_rear(self, value: T) -> PyDeque[T]`:** `self._elements + [value]` é $O(N)$. Construtor é $O(N)$. **Total: $O(N)$**.
7.  **`front(self) -> T`:** Acesso `self._elements[0]` é $O(1)$. **Total: $O(1)$**.
8.  **`rear(self) -> T`:** Acesso `self._elements[-1]` é $O(1)$. **Total: $O(1)$**.
9.  **`remove_front(self) -> PyDeque[T]`:** `self._elements[1:]` (slicing) é $O(N)$. Construtor é $O(N)$. **Total: $O(N)$**.
10. **`remove_rear(self) -> PyDeque[T]`:** `self._elements[:-1]` (slicing) é $O(N)$. Construtor é $O(N)$. **Total: $O(N)$**.
11. **`get_at_index(self, index: int) -> T`:** Acesso `self._elements[index]` é $O(1)$. **Total: $O(1)$**.
12. **`rotate(self, k: int) -> PyDeque[T]`:** Slicing e concatenação são $O(N)$. Construtor é $O(N)$. **Total: $O(N)$**.

A implementação imutável baseada em lista Python resulta em $O(N)$ para todas as operações que modificam a estrutura (exceto `empty_deque`), devido à necessidade de copiar a lista interna. Operações de acesso são $O(1)$. Implementações mutáveis de Deque (como `collections.deque`) podem alcançar $O(1)$ para adições/remoções nas extremidades.

**Exercício:**

Se a operação `add_front` na classe `PyDeque[T]` fosse implementada para modificar o deque no local (mutável), usando `self._elements.insert(0, value)`, qual seria a complexidade de tempo desta versão mutável de `add_front`?

**Resolução:**

Se `add_front` fosse implementada de forma mutável usando `self._elements.insert(0, value)`:
A operação `list.insert(0, value)` do Python insere `value` no início da lista. Para fazer isso, todos os elementos existentes na lista precisam ser deslocados uma posição para a direita para abrir espaço para o novo elemento. Se a lista tem $N$ elementos, isso requer $N$ operações de deslocamento.
Portanto, a complexidade de tempo de `list.insert(0, value)` é **$O(N)$**.

A complexidade da versão mutável de `add_front` usando `list.insert(0, ...)` seria **$O(N)$**.

# 12.6 Exercícios Teóricos e Práticos

Esta seção apresenta um exercício combinado para aplicar e estender os conceitos do TAD `Deque[T]`.

**Exercício:**

**Contexto:** Este exercício foca na extensão do TAD `Deque[T]` e sua implementação `PyDeque[T]` com uma operação `multi_remove_front` que remove múltiplos elementos do início do deque.

**Parte a) Especificação Algébrica para `multi_remove_front` em `Deque[T]`**
Adicione uma operação `multiRemoveFront: Natural x Deque[T] --> Deque[T]` à especificação algébrica `Deque[T]` (da Seção 12.2). Esta operação deve remover `k` (um `Natural`) elementos do início do deque `d`.
1.  Se `k` for `zero`, o deque original `d` é retornado.
2.  Se o deque `d` estiver vazio, `multiRemoveFront(k, d)` retorna `emptyDeque`.
3.  Se $k > 0$ e o deque $d$ não estiver vazio (i.e., $d = \text{addFront(e, d')}$ ou $d = \text{addRear(d',e)}$), então `multiRemoveFront(k, d)` é equivalente a `multiRemoveFront(pred(k), removeFront(d))`.
Forneça os axiomas para `multiRemoveFront`.

**Parte b) Implementação Python para `multi_remove_front` em `PyDeque[T]`**
Implemente o método correspondente `multi_remove_front(self, k: int) -> PyDeque[T]` na classe `PyDeque[T]` (da Seção 12.3). A implementação deve:
1.  Seguir o estilo funcional/imutável.
2.  Validar que `k` é um `int >= 0`. Se `k` for negativo, levantar `ValueError`.
3.  Remover os `k` elementos do início. Se `k` for maior ou igual ao tamanho do deque, o resultado deve ser um deque vazio.

**Parte c) Testes de Propriedade para `multi_remove_front` em `PyDeque[T]`**
Usando Hypothesis, escreva testes de propriedade para sua implementação `multi_remove_front` que correspondam aos axiomas definidos na Parte (a).

**Resolução:**

**Parte a) Especificação Algébrica para `multi_remove_front` em `Deque[T]`**

**SPEC ADT** DequeWithMultiRemoveFront[ParamT: ElementTypeForDeque]
**imports:**
*   ParamT
*   Boolean
*   Natural
*   Deque[ParamT]

**sorts:**
*   Deque

**operations:**
*   multiRemoveFront: Natural x Deque --> Deque

**axioms:**
for all d: Deque, elem: T, k: Natural

*   (D_MRF1): multiRemoveFront(zero, d) = d
*   (D_MRF2): multiRemoveFront(k, emptyDeque) = emptyDeque
*   (D_MRF3): eq(k,zero) = false AND isEmpty(d) = false =>
              multiRemoveFront(k, d) = multiRemoveFront(pred(k), removeFront(d))

**END SPEC**

A especificação `DequeWithMultiRemoveFront` enriquece `Deque[T]`. A nova operação `multiRemoveFront` recebe um `Natural` `k` e um `Deque` `d`.
*   (D_MRF1): Remover `zero` elementos de `d` resulta em `d`.
*   (D_MRF2): Tentar remover `k` elementos de um `emptyDeque` resulta em `emptyDeque`.
*   (D_MRF3): Se `k > 0` e o deque `d` não está vazio, remover `k` elementos de `d` é o mesmo que remover `pred(k)` (ou $k-1$) elementos do deque resultante de aplicar `removeFront(d)` uma vez. A condição `isEmpty(d)=false` garante que `removeFront(d)` é aplicado a um deque não vazio.

**Parte b) Implementação Python para `multi_remove_front` em `PyDeque[T]`**

```python
# py_deque.py (adicionando à classe PyDeque[T])

    # ... (outros métodos como definidos anteriormente) ...

    def multi_remove_front(self, k: int) -> PyDeque[T]:
        # Removes k elements from the front of the deque.
        # Returns a NEW PyDeque.
        # k must be a non-negative integer.
        # If k is larger than or equal to deque size, an empty deque is returned.
        if not (isinstance(k, int) and k >= 0):
            raise ValueError("k must be a non-negative integer.")
        
        if k == 0:
            return self # Immutable, no change
        
        current_size = self.size()
        if self.is_empty() or k >= current_size:
            return PyDeque.empty_deque()
            
        # Elements to keep start from index k
        new_elements = self._elements[k:] # Slice from index k to the end
        return PyDeque(new_elements)
    # ... (restante da classe) ...
```
O método `multi_remove_front` é adicionado à classe `PyDeque[T]`.
1.  Valida `k`.
2.  Se `k` é 0, retorna `self`.
3.  Se o deque está vazio ou `k` é maior/igual ao tamanho, retorna um `empty_deque`.
4.  Caso contrário, cria `new_elements` fatiando a lista interna `_elements` a partir do índice `k` até o final.
5.  Retorna um novo `PyDeque` com `new_elements`.

**Parte c) Testes de Propriedade para `multi_remove_front` em `PyDeque[T]`**

```python
# test_py_deque.py (continuação)
# ... (importações e estratégias st_py_deque, st_element_type, st_natural_value como antes) ...

st_k_for_multi_remove = st.integers(min_value=0, max_value=12) # k can be up to size + 2

# --- Testes para multi_remove_front Axioms ---

@given(d=st_py_deque)
def test_D_MRF1_multiRemoveFront_zero_is_identity(d: PyDeque[int]):
    # (D_MRF1): multiRemoveFront(zero, d) = d
    assert d.multi_remove_front(0) == d

@given(k=st_k_for_multi_remove)
def test_D_MRF2_multiRemoveFront_on_empty_is_empty(k: int):
    # (D_MRF2): multiRemoveFront(k, emptyDeque) = emptyDeque
    empty_d = PyDeque[Any].empty_deque()
    assert empty_d.multi_remove_front(k) == empty_d

@given(d_non_empty=st_non_empty_py_deque, k=st.integers(min_value=1, max_value=12)) 
@settings(suppress_health_check=[HealthCheck.filter_too_much], max_examples=100)
def test_D_MRF3_multiRemoveFront_recursive_step(d_non_empty: PyDeque[int], k: int):
    # (D_MRF3): eq(k,zero) = false AND isEmpty(d) = false =>
    #           multiRemoveFront(k, d) = multiRemoveFront(pred(k), removeFront(d))
    # k > 0 is ensured by st.integers(min_value=1)
    # d is not empty by st_non_empty_py_deque
    
    # Ensure k is not larger than current size for this recursive axiom to make sense directly,
    # as removeFront might be called on an empty deque otherwise inside the RHS recursion.
    # The Python impl handles k >= size by returning empty, which is fine.
    # The axiom implies we only "peel off" one layer at a time if elements are available.
    assume(k <= d_non_empty.size()) # Make sure k is not excessively large for this specific test form

    lhs = d_non_empty.multi_remove_front(k)
    
    # Calculate RHS: multiRemoveFront(pred(k), removeFront(d))
    # removeFront(d_non_empty) is safe as d_non_empty is not empty
    d_after_remove_front = d_non_empty.remove_front()
    rhs = d_after_remove_front.multi_remove_front(k - 1) # k-1 is pred(k)
    
    assert lhs == rhs
```
Os testes de propriedade para `multi_remove_front`:
*   `test_D_MRF1_multiRemoveFront_zero_is_identity` verifica (D_MRF1).
*   `test_D_MRF2_multiRemoveFront_on_empty_is_empty` verifica (D_MRF2).
*   `test_D_MRF3_multiRemoveFront_recursive_step` verifica (D_MRF3). Usa `st_non_empty_py_deque` e `st.integers(min_value=1)` para `k`. Adiciona-se `assume(k <= d_non_empty.size())` para que o lado direito do axioma (que envolve `removeFront(d)`) não tente aplicar `removeFront` em um deque que se tornou vazio muito cedo na "simulação" recursiva do axioma, se `k` fosse maior que o tamanho original.

Este capítulo explorou o TAD `Deque[T]`, sua especificação algébrica, uma implementação Python genérica `PyDeque[T]` imutável, e a verificação de seus axiomas com Hypothesis. A análise de complexidade e o exercício de extensão com `multi_remove_front` completaram o estudo. As Partes II, III e IV continuarão a aplicar esta metodologia sistemática (definição abstrata, especificação algébrica, análise, implementação Python, verificação com Hypothesis, análise de complexidade) a outras estruturas de dados clássicas, começando com árvores na Parte III.

---
## REFERÊNCIAS BIBLIOGRÁFICAS
AHO, A. V.; HOPCROFT, J. E.; ULLMAN, J. D. **Data Structures and Algorithms**. Reading, MA: Addison-Wesley, 1983.
*   *Resumo: Embora mais antigo, este texto clássico introduz o conceito de deque (double-ended queue) e discute suas operações fundamentais. É uma referência útil para a definição abstrata e o papel do TAD Deque.*

CORMEN, T. H.; LEISERSON, C. E.; RIVEST, R. L.; STEIN, C. **Introduction to Algorithms**. 3. ed. Cambridge, MA: MIT Press, 2009.
*   *Resumo: Este livro de referência sobre algoritmos inclui uma seção sobre estruturas de dados elementares onde deques são discutidos, geralmente como uma generalização de pilhas e filas, com implementações baseadas em arrays ou listas duplamente encadeadas.*

GOODRICH, M. T.; TAMASSIA, R.; GOLDWASSER, M. H. **Data Structures and Algorithms in Python**. Hoboken, NJ: Wiley, 2013.
*   *Resumo: Aborda a implementação de deques em Python, discutindo o uso da classe `collections.deque` da biblioteca padrão, que é uma implementação eficiente de deque. Fornece um contexto prático para a implementação `PyDeque[T]`.*

KNUTH, D. E. **The Art of Computer Programming, Volume 1: Fundamental Algorithms**. 3. ed. Reading, MA: Addison-Wesley, 1997.
*   *Resumo: A obra monumental de Knuth discute deques em sua seção sobre listas lineares. Apresenta diversas representações e algoritmos para operações de deque, sendo uma fonte profunda para os aspectos algorítmicos.*

OKASAKI, C. **Purely Functional Data Structures**. Cambridge: Cambridge University Press, 1998.
*   *Resumo: Este livro é uma referência chave para a implementação de estruturas de dados, incluindo deques, em um paradigma puramente funcional, com ênfase na persistência e eficiência amortizada. Relevante para o estilo imutável adotado em `PyDeque[T]`.*

PYTHON SOFTWARE FOUNDATION. **Python `collections.deque` documentation**. Disponível em: https://docs.python.org/3/library/collections.html#collections.deque. Acesso em: 10 ago. 2023.
*   *Resumo: A documentação oficial da `collections.deque` do Python é essencial para entender uma implementação de deque otimizada e eficiente na linguagem. Serve como um ponto de comparação e referência para implementações customizadas como `PyDeque[T]`.*

---
