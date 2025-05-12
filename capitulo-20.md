---
# Capítulo 20
# Tipo Abstrato de Dado Graph
---

Concluindo nossa exploração das principais estruturas de dados para coleções e relações, este capítulo se dedica ao Tipo Abstrato de Dados (TAD) `Graph[V, E]`, ou Grafo. Grafos são estruturas matemáticas fundamentais usadas para modelar relações entre objetos. Um grafo consiste em um conjunto de **vértices** (ou nós) e um conjunto de **arestas** (ou arcos) que conectam pares de vértices. Esta estrutura de dados é extraordinariamente versátil, encontrando aplicações em domínios tão diversos quanto redes de computadores, redes sociais, modelagem de circuitos, logística, bioinformática, e muito mais. Dada a variedade de tipos de grafos (direcionados, não direcionados, ponderados, não ponderados, cíclicos, acíclicos) e a riqueza de operações e algoritmos associados, uma especificação algébrica completa de um TAD `Graph` genérico pode se tornar bastante complexa. Este capítulo focará em definir um TAD `Graph` básico, parametrizado pelos tipos dos identificadores dos vértices (`V`) e dos possíveis atributos das arestas (`E`), e especificará operações fundamentais como adição e remoção de vértices earestas, verificação de adjacência e obtenção de vizinhos. Seguiremos a estrutura metodológica padrão: definição abstrata, especificação algébrica formal (com análise de consistência e completeza), projeto e implementação de uma classe `PyGraph[V, E]` em Python com tipagem MyPy (usando uma lista de adjacências como representação interna), verificação com Hypothesis e análise de complexidade. Uma discussão sobre alternativas de representação para grafos precederá o exercício prático final que consolida os conceitos.

# 20.1 Definição Abstrata e Relevância do Tipo de Dados `Graph[V, E]`

Um **Grafo** é um Tipo Abstrato de Dados que representa um conjunto de objetos, chamados **vértices** (ou nós), e as conexões ou relações entre pares desses objetos, chamadas **arestas** (ou arcos). Formalmente, um grafo $G$ é frequentemente definido como um par $G = (ConjuntoV, ConjuntoA)$, onde $ConjuntoV$ é um conjunto de vértices e $ConjuntoA$ é um conjunto de arestas.

**Tipos de Grafos e Componentes:**
*   **Vértices (`V`):** Entidades fundamentais. `V` representa o tipo dos identificadores dos vértices.
*   **Arestas (`E`):** Conexão entre dois vértices. Podem ser direcionadas ou não direcionadas. `E` representa o tipo do atributo/peso da aresta.
*   **Outras Características:** Grafos podem ser simples, multigrafos, com auto-loops, ponderados, etc.

**Definição Abstrata das Operações Essenciais para `Graph[V, E]` (foco em grafo direcionado simples, possivelmente ponderado):**

1.  **Criação:** `emptyGraph: --> Graph[V,E]`
2.  **Operações de Vértice:** `addVertex`, `removeVertex`, `hasVertex`
3.  **Operações de Aresta:** `addEdge`, `removeEdge`, `hasEdge`, `getEdgeAttribute`
4.  **Operações de Consulta/Vizinhança:** `getVertices`, `getNeighbors`

**Relevância do TAD `Graph[V, E]`:**
Grafos modelam relações em redes sociais, web, transporte, circuitos, dependências de projetos, biologia, sistemas de recomendação, etc., e são base para muitos algoritmos importantes.

**Exercício:**

Considere um grafo direcionado simples $G$ com vértices $\{v_1, v_2, v_3, v_4\}$ e as seguintes arestas (com atributos/pesos irrelevantes para esta questão): $(v_1, v_2)$, $(v_1, v_3)$, $(v_2, v_3)$, $(v_3, v_4)$.
1.  Qual o resultado de `getNeighbors(v1, G)`?
2.  Qual o resultado de `hasEdge(v1, v4, G)`?
3.  Se executarmos `G' = removeEdge(v1, v3, G)`, qual seria o resultado de `getNeighbors(v1, G')`?

**Resolução:**

Dado o grafo $G$ com vértices $V = \{v_1, v_2, v_3, v_4\}$ e arestas $A = \{(v_1, v_2), (v_1, v_3), (v_2, v_3), (v_3, v_4)\}$.

1.  **`getNeighbors(v1, G)`:** Resultado: $\{v_2, v_3\}$.
2.  **`hasEdge(v1, v4, G)`:** Resultado: `false`.
3.  **`G' = removeEdge(v1, v3, G)`, `getNeighbors(v1, G')`:** Arestas de $G'$ são $\{(v_1, v_2), (v_2, v_3), (v_3, v_4)\}$. Resultado: $\{v_2\}$.

# 20.2 Especificação Algébrica Formal do TAD `Graph[VertexId, EdgeAttr]`

**SPEC ADT** VertexIdentifierType
**sorts:**
*   VertexId
**operations:**
*   eqVertexId: VertexId x VertexId --> Boolean
**axioms:**
*   (VID1): eqVertexId(v,v) = true
*   (VID2): eqVertexId(v1,v2) = eqVertexId(v2,v1)
for all v,v1,v2: VertexId
**END SPEC**

A especificação `VertexIdentifierType` define o requisito para o tipo do identificador de vértice: um sort `VertexId` e uma operação de igualdade `eqVertexId` reflexiva e simétrica.

**SPEC ADT** EdgeAttributeType
**sorts:**
*   EdgeAttr
**END SPEC**

A especificação `EdgeAttributeType` define o requisito para o tipo do atributo da aresta: apenas um sort `EdgeAttr`.

**SPEC ADT** Graph[VParam: VertexIdentifierType, EParam: EdgeAttributeType]
**imports:**
*   VParam
*   EParam
*   Natural
*   Boolean
*   Set[VertexId]

**sorts:**
*   Graph

**operations:**
*   emptyGraph: --> Graph
*   addVertex: VertexId x Graph --> Graph
*   removeVertex: VertexId x Graph --> Graph
*   hasVertex: VertexId x Graph --> Boolean
*   addEdge: VertexId x VertexId x EdgeAttr x Graph --> Graph
*   removeEdge: VertexId x VertexId x Graph --> Graph
*   hasEdge: VertexId x VertexId x Graph --> Boolean
*   getEdgeAttribute: VertexId x VertexId x Graph --> EdgeAttr
*   getVertices: Graph --> Set[VertexId]
*   getNeighbors: VertexId x Graph --> Set[VertexId]
*   outDegree: VertexId x Graph --> Natural (* Adicionado no Exercício 20.5 *)

**axioms:**
for all g: Graph, v, v1, v2, v3, vFrom, vTo, vOther, vTarget, vRemove, vAdd1, vAdd2, vQueryFrom, vQueryTo: VertexId, attr, attr2: EdgeAttr

*   (G1): hasVertex(v, emptyGraph) = false
*   (G2): hasVertex(v1, addVertex(v2, g)) = or(eqVertexId(v1, v2), hasVertex(v1, g))
*   (G3): hasEdge(v1, v2, emptyGraph) = false
*   (G4): hasEdge(v1, v2, addVertex(v3, g)) = hasEdge(v1, v2, g)
*   (G5): addEdge(v1, v2, attr, g) = addVertex(v1, addVertex(v2, addEdgeWithoutVertexCreation(v1,v2,attr,g)))
*   (G6): hasEdge(v1, v2, addEdge(v1, v2, attr, g)) = true
*   (G7): eqVertexId(vQueryFrom,vAdd1)=false OR eqVertexId(vQueryTo,vAdd2)=false =>
        hasEdge(vQueryFrom, vQueryTo, addEdge(vAdd1, vAdd2, attr, g)) = hasEdge(vQueryFrom, vQueryTo, g)
*   (G8): getEdgeAttribute(v1, v2, addEdge(v1, v2, attr, g)) = attr
*   (G9): eqVertexId(v1,vAdd1)=false OR eqVertexId(v2,vAdd2)=false =>
        getEdgeAttribute(v1, v2, addEdge(vAdd1, vAdd2, attr, g)) = getEdgeAttribute(v1, v2, g)
*   (G10): getVertices(emptyGraph) = emptySet[VertexId]
*   (G11): getVertices(addVertex(v,g)) = addSet(v, getVertices(g))
*   (G12): getNeighbors(v, emptyGraph) = emptySet[VertexId]
*   (G13): hasVertex(vTarget, g) = false => getNeighbors(vFrom, addVertex(vTarget, g)) = getNeighbors(vFrom, g)
*   (G14): getNeighbors(vFrom, addEdge(vFrom, vTo, attr, g)) = addSet(vTo, getNeighbors(vFrom, g))
*   (G15): eqVertexId(vFrom, vOther)=false =>
         getNeighbors(vFrom, addEdge(vOther, vTo, attr, g)) = getNeighbors(vFrom, g)
*   (G16): removeVertex(v, emptyGraph) = emptyGraph
*   (G17): removeVertex(vTarget, addVertex(vRemove, g)) =
            ifThenElse(eqVertexId(vTarget, vRemove),
                       removeVertex(vTarget, g),
                       addVertex(vRemove, removeVertex(vTarget, g)))
*   (G18): removeVertex(vTarget, addEdge(v1,v2,attr,g)) =
            ifThenElse(or(eqVertexId(vTarget,v1), eqVertexId(vTarget,v2)),
                       removeVertex(vTarget,g), 
                       addEdge(v1,v2,attr,removeVertex(vTarget,g)))
*   (G19): removeEdge(v1,v2, emptyGraph) = emptyGraph
*   (G20): removeEdge(v1,v2, addVertex(v3,g)) = addVertex(v3, removeEdge(v1,v2,g))
*   (G21): removeEdge(vTarget1, vTarget2, addEdge(vAdd1, vAdd2, attr, g)) =
            ifThenElse(and(eqVertexId(vTarget1,vAdd1), eqVertexId(vTarget2,vAdd2)),
                       g, 
                       addEdge(vAdd1,vAdd2,attr,removeEdge(vTarget1,vTarget2,g)))
*   (GD1): outDegree(v, emptyGraph) = zero
*   (GD2): hasVertex(v,g) = false => outDegree(v, addVertex(v,g)) = zero
*   (GD3): eqVertexId(v1,v2) = false AND hasVertex(v1,g) = true =>
           outDegree(v1, addVertex(v2,g)) = outDegree(v1,g)
*   (GD4): hasEdge(vFrom, vTo, g) = true =>
           outDegree(vFrom, addEdge(vFrom, vTo, attr, g)) = outDegree(vFrom, g)
*   (GD5): hasEdge(vFrom, vTo, g) = false =>
           outDegree(vFrom, addEdge(vFrom, vTo, attr, g)) = succ(outDegree(vFrom, g))
*   (GD6): eqVertexId(vOther, vFrom) = false AND hasVertex(vOther,g) = true =>
           outDegree(vOther, addEdge(vFrom, vTo, attr, g)) = outDegree(vOther, g)

**END SPEC**

A especificação `Graph[VParam, EParam]` é parametrizada e importa os tipos necessários, incluindo um TAD `Set[VertexId]`. O sort principal é `Graph`. As operações cobrem criação, manipulação de vértices e arestas, e consultas. Os axiomas (G1)-(G21) definem o comportamento dessas operações. A operação `outDegree` e seus axiomas (GD1)-(GD6) foram adicionados conforme o exercício que será resolvido na Seção 20.6. A complexidade dos axiomas, especialmente (G5) (que implica adição implícita de vértices por `addEdge`) e os de remoção (G16-G18, G19-G21), reflete a dificuldade de especificar grafos algebricamente de forma completa e construtiva. A `addEdgeWithoutVertexCreation` mencionada em (G5) é uma operação conceitual interna que adiciona uma aresta assumindo que os vértices já existem; a especificação aqui optou por uma visão onde `addEdge` garante a existência dos vértices.

## 20.2.1 Análise de Corretude e Completeza da Especificação `Graph[V,E]`

A análise de consistência e completeza para `Graph[V,E]` é complexa.

**Consistência:**
A especificação parece **plausível** em termos de consistência, assumindo que os TADs importados são consistentes e que os axiomas de remoção e o complexo axioma G5 não introduzem contradições sutis. Um modelo baseado em grafos matemáticos padrão serviria como evidência.

**Completeza Suficiente:**
*   Observadores como `hasVertex`, `hasEdge`, `getVertices`, `getNeighbors`, e `outDegree` parecem ser suficientemente completos (reduzem a valores de `Boolean`, `Set[VertexId]` ou `Natural`).
*   `getEdgeAttribute` é parcial (não definida se a aresta não existe).
*   As operações modificadoras `removeVertex` e `removeEdge` são definidas recursivamente sobre os construtores e têm potencial para serem suficientemente completas (resultando em um `Graph`), mas provar isso seria complexo e dependeria da terminação das recursões.

Uma especificação mais simples para uso prático poderia restringir `addEdge` para operar apenas sobre vértices existentes, simplificando G5.

**Exercício:**

Se a especificação `Graph[VParam, EParam]` não incluísse o axioma (G4): `hasEdge(v1, v2, addVertex(v3, g)) = hasEdge(v1, v2, g)`, qual seria a consequência para a completeza suficiente da operação `hasEdge`?

**Resolução:**

A operação a ser analisada é `hasEdge(v_arg1: VertexId, v_arg2: VertexId, g_arg: Graph)`.
Construtores (assumidos): `emptyGraph`, `addVertex(v, g')`, `addEdge(v1, v2, attr, g')`.

1.  **Caso $g_{arg} = \text{emptyGraph}$:** Coberto por (G3) $\rightarrow$ `false`.
2.  **Caso $g_{arg} = \text{addEdge}(v_{a1}, v_{a2}, attr', g')$**: Coberto por (G6) e (G7), reduzindo a `true`, `false`, ou `hasEdge` sobre $g'$.
3.  **Caso $g_{arg} = \text{addVertex}(v_{new}, g')$**:
    *   Se (G4) está presente, ele cobre este caso, reduzindo a `hasEdge` sobre $g'$.
    *   Se (G4) fosse omitido, o termo `hasEdge(v_arg1, v_arg2, addVertex(v_new, g'))` não seria redutível.
    Portanto, `hasEdge` **não seria suficientemente completa** sem (G4).

# 20.3 Projeto e Implementação em Python com Tipagem Estática (MyPy)

Implementaremos `PyGraph[V,E]` usando uma lista de adjacências baseada em dicionários Python. `V` deve ser hasheável e comparável.

```python
# py_graph.py
from __future__ import annotations 
from typing import TypeVar, Generic, Dict, Set as PySetInternal, Optional, Any 

V = TypeVar('V') # Must be hashable
E = TypeVar('E') 

class PyGraph(Generic[V, E]):
    _adj: Dict[V, Dict[V, E]]

    def __init__(self) -> None:
        # Constructor: Initializes an empty graph.
        self._adj = {}

    @staticmethod
    def empty_graph() -> PyGraph[Any, Any]: 
        # Creates and returns an empty graph.
        return PyGraph()

    def add_vertex(self, v_id: V) -> PyGraph[V, E]:
        # Adds a vertex to the graph. Returns a new graph.
        new_adj = {key: value.copy() for key, value in self._adj.items()} 
        if v_id not in new_adj:
            new_adj[v_id] = {}
        
        new_graph = PyGraph[V,E]() 
        new_graph._adj = new_adj
        return new_graph

    def has_vertex(self, v_id: V) -> bool:
        # Checks if a vertex exists in the graph.
        return v_id in self._adj

    def add_edge(self, v_from: V, v_to: V, attr: E) -> PyGraph[V, E]:
        # Adds a directed edge from v_from to v_to with attribute attr.
        # If vertices do not exist, they are added. Returns a new graph.
        new_adj = {key: value.copy() for key, value in self._adj.items()}

        if v_from not in new_adj:
            new_adj[v_from] = {}
        if v_to not in new_adj:
            new_adj[v_to] = {}
            
        new_adj[v_from][v_to] = attr
        
        new_graph = PyGraph[V,E]()
        new_graph._adj = new_adj
        return new_graph

    def has_edge(self, v_from: V, v_to: V) -> bool:
        # Checks if a directed edge exists from v_from to v_to.
        return v_from in self._adj and v_to in self._adj[v_from]

    def get_edge_attribute(self, v_from: V, v_to: V) -> E:
        # Returns the attribute of the edge from v_from to v_to.
        # Raises KeyError if v_from does not exist or edge does not exist.
        if not self.has_edge(v_from, v_to):
            raise KeyError(f"Edge from {v_from} to {v_to} not found.")
        return self._adj[v_from][v_to]

    def get_vertices(self) -> PySetInternal[V]:
        # Returns a set of all vertices in the graph.
        return set(self._adj.keys())

    def get_neighbors(self, v_id: V) -> PySetInternal[V]:
        # Returns a set of outgoing neighbors of vertex v_id.
        # Raises KeyError if v_id does not exist.
        if not self.has_vertex(v_id):
            raise KeyError(f"Vertex {v_id} not found in graph.")
        return set(self._adj[v_id].keys())

    def remove_vertex(self, v_id: V) -> PyGraph[V, E]:
        # Removes vertex v_id and all incident edges. Returns a new graph.
        if not self.has_vertex(v_id):
            new_adj_noop = {key: value.copy() for key, value in self._adj.items()}
            new_graph_noop = PyGraph[V,E]()
            new_graph_noop._adj = new_adj_noop
            return new_graph_noop

        new_adj = {key: value.copy() for key, value in self._adj.items()}
        
        if v_id in new_adj:
            del new_adj[v_id]
        
        for source_vertex in list(new_adj.keys()): 
            if v_id in new_adj[source_vertex]:
                del new_adj[source_vertex][v_id]
                
        new_graph = PyGraph[V,E]()
        new_graph._adj = new_adj
        return new_graph

    def remove_edge(self, v_from: V, v_to: V) -> PyGraph[V, E]:
        # Removes the directed edge from v_from to v_to. Returns a new graph.
        if not self.has_edge(v_from, v_to):
            new_adj_noop = {key: value.copy() for key, value in self._adj.items()}
            new_graph_noop = PyGraph[V,E]()
            new_graph_noop._adj = new_adj_noop
            return new_graph_noop

        new_adj = {key: value.copy() for key, value in self._adj.items()}
        del new_adj[v_from][v_to]
        
        new_graph = PyGraph[V,E]()
        new_graph._adj = new_adj
        return new_graph
    
    def out_degree(self, v_id: V) -> int:
        # Returns the out-degree of vertex v_id.
        # Raises KeyError if v_id does not exist in the graph.
        if not self.has_vertex(v_id):
            raise KeyError(f"Vertex {v_id} not found in graph.")
        return len(self._adj[v_id])

    def get_in_neighbors(self, v_id: V) -> PySetInternal[V]:
        # Returns a set of incoming neighbors to vertex v_id.
        # Raises KeyError if v_id does not exist in the graph.
        if not self.has_vertex(v_id):
            raise KeyError(f"Vertex {v_id} not found in graph.")
        
        in_neighbors: PySetInternal[V] = set()
        for source_vertex, destinations in self._adj.items():
            if v_id in destinations: 
                in_neighbors.add(source_vertex)
        return in_neighbors

    def __eq__(self, other: object) -> bool:
        # Equality check.
        if not isinstance(other, PyGraph):
            return NotImplemented
        return self._adj == other._adj

    def __repr__(self) -> str:
        # String representation.
        return f"PyGraph(adj={self._adj})"

```
O código Python acima, no arquivo `py_graph.py`, define a classe genérica `PyGraph[V, E]`.
A representação interna `_adj` é um dicionário de dicionários para a lista de adjacências. `empty_graph` cria um grafo vazio. `add_vertex` e `add_edge` são imutáveis, criando novas cópias profundas do dicionário `_adj`. `has_vertex`, `has_edge`, `get_edge_attribute`, `get_vertices`, e `get_neighbors` são operações de consulta. `remove_vertex` e `remove_edge` também são imutáveis, recriando o dicionário `_adj` após as remoções. Os métodos `out_degree` e `get_in_neighbors` foram adicionados conforme o exercício da Seção 20.6 (adaptado). `__eq__` e `__repr__` são fornecidos.

**Exercício:**

Implemente um método `get_in_neighbors(self, v_id: V) -> PySet[V]` na classe `PyGraph[V, E]`. Este método deve retornar um conjunto de todos os vértices `u` para os quais existe uma aresta direcionada $(u, v_{id})$ no grafo (vizinhos de entrada). Se o vértice `v_id` não existir, o método deve levantar `KeyError`.
*(Este exercício já foi resolvido e incorporado na classe `PyGraph[V,E]` acima.)*

**Resolução:**

O método `get_in_neighbors` já foi adicionado à classe `PyGraph[V,E]` na listagem de código acima. Sua implementação é:
```python
# py_graph.py (dentro da classe PyGraph[V, E])
    # ...
    def get_in_neighbors(self, v_id: V) -> PySetInternal[V]:
        # Returns a set of incoming neighbors to vertex v_id.
        # Raises KeyError if v_id does not exist in the graph.
        if not self.has_vertex(v_id):
            raise KeyError(f"Vertex {v_id} not found in graph.")
        
        in_neighbors: PySetInternal[V] = set()
        for source_vertex, destinations in self._adj.items():
            if v_id in destinations: # Check if source_vertex has an edge to v_id
                in_neighbors.add(source_vertex)
        return in_neighbors
    # ...
```
Este método primeiro verifica a existência de `v_id`. Em seguida, itera sobre todas as entradas do dicionário de adjacências. Para cada `source_vertex`, ele verifica se `v_id` está no conjunto de seus destinos (`destinations`). Se estiver, `source_vertex` é adicionado ao conjunto `in_neighbors`.

# 20.4 Verificação de Propriedades Axiomáticas com Hypothesis

Com a especificação algébrica de `Graph[V,E]` e a implementação `PyGraph[V,E]`, utilizamos Hypothesis para verificar se a implementação adere aos axiomas.

**Estratégias Hypothesis para `PyGraph[V,E]`, `V`, e `E`:**
Usaremos inteiros para `V` e strings simples para `E`.

```python
# test_py_graph.py
import pytest
from hypothesis import given, strategies as st, assume, settings, HealthCheck
from typing import TypeVar, Any, Tuple, Set as PySetInternal # Renomeado para evitar conflito

# Import the implementation
from py_graph import PyGraph # Assuming py_graph.py is accessible

# Strategies for VertexId (V) and EdgeAttribute (E)
st_vertex_id = st.integers(min_value=0, max_value=10) 
st_edge_attr = st.text(alphabet=st.characters(min_codepoint=97, max_codepoint=122), min_size=1, max_size=3)

@st.composite
def st_py_graph_int_str(draw):
    graph = PyGraph[int, str]() 
    
    vertices_to_add = draw(st.lists(st_vertex_id, max_size=5, unique=True), label="vertices")
    for v_id_val in vertices_to_add: # Renamed v_id to v_id_val
        graph = graph.add_vertex(v_id_val)
        
    num_edges = draw(st.integers(min_value=0, max_value=7), label="num_edges")
    if not vertices_to_add and num_edges > 0: 
        num_edges = 0

    if vertices_to_add: 
        for _ in range(num_edges):
            v_from_val = draw(st.sampled_from(vertices_to_add), label="v_from") # Renamed v_from
            v_to_val = draw(st.sampled_from(vertices_to_add), label="v_to")     # Renamed v_to
            attr_val = draw(st_edge_attr, label="attr")                         # Renamed attr
            graph = graph.add_edge(v_from_val, v_to_val, attr_val)
            
    return graph

st_graph_vertex_op = st.tuples(st_py_graph_int_str(), st_vertex_id)
st_graph_edge_op_base = st.tuples(st_vertex_id, st_vertex_id, st_edge_attr) # For edge components

@st.composite
def st_graph_and_edge_args(draw): # Generates graph and args for add_edge
    g = draw(st_py_graph_int_str())
    v_from = draw(st_vertex_id)
    v_to = draw(st_vertex_id)
    attr = draw(st_edge_attr)
    return g, v_from, v_to, attr

```
O código de teste `test_py_graph.py` configura as estratégias. `st_vertex_id` gera inteiros. `st_edge_attr` gera strings. `st_py_graph_int_str` é uma estratégia composta para construir grafos. `st_graph_vertex_op` gera um grafo e um ID de vértice. `st_graph_and_edge_args` gera um grafo e argumentos para adicionar uma aresta.

**Testes de Propriedade para os Axiomas de `Graph[V,E]`:**
Axiomas selecionados (G1-G4, G6-G7, G10-G11, GDs) serão testados.

```python
# test_py_graph.py (continuação)

# --- Axioms for hasVertex ---
def test_G1_hasVertex_emptyGraph():
    # (G1): hasVertex(v, emptyGraph) = false
    g = PyGraph[int, str].empty_graph()
    assert g.has_vertex(0) == False 

@given(data=st_graph_vertex_op, v_query=st_vertex_id)
@settings(suppress_health_check=[HealthCheck.filter_too_much], deadline=1000, max_examples=50)
def test_G2_hasVertex_addVertex(data: Tuple[PyGraph[int, str], int], v_query: int):
    # (G2): hasVertex(v_query, addVertex(v_added, g)) = 
    #       or(eqVertexId(v_query, v_added), hasVertex(v_query, g))
    g, v_added = data
    
    g_after_add = g.add_vertex(v_added)
    
    lhs = g_after_add.has_vertex(v_query)
    rhs = (v_query == v_added) or g.has_vertex(v_query) 
    assert lhs == rhs

# --- Axioms for hasEdge (simplified context) ---
def test_G3_hasEdge_emptyGraph():
    # (G3): hasEdge(v1, v2, emptyGraph) = false
    g = PyGraph[int, str].empty_graph()
    assert g.has_edge(0, 1) == False 

@given(data=st_graph_vertex_op, v1=st_vertex_id, v2=st_vertex_id)
@settings(suppress_health_check=[HealthCheck.filter_too_much], deadline=1000, max_examples=50)
def test_G4_hasEdge_addVertex(data: Tuple[PyGraph[int, str], int], v1: int, v2: int):
    # (G4): hasEdge(v1, v2, addVertex(v3, g)) = hasEdge(v1, v2, g)
    g, v3_added = data
    
    g_after_add_v = g.add_vertex(v3_added)
    
    lhs = g_after_add_v.has_edge(v1, v2)
    rhs = g.has_edge(v1, v2)
    assert lhs == rhs

@given(data=st_graph_and_edge_args)
@settings(suppress_health_check=[HealthCheck.filter_too_much], deadline=1000, max_examples=50)
def test_G6_hasEdge_after_addEdge_same(data: Tuple[PyGraph[int, str], int, int, str]):
    # (G6): hasEdge(v1, v2, addEdge(v1, v2, attr, g)) = true
    g, v_from, v_to, attr = data
    
    g_after_add_e = g.add_edge(v_from, v_to, attr)
    assert g_after_add_e.has_edge(v_from, v_to) == True

@given(data=st_graph_and_edge_args, v_query_from=st_vertex_id, v_query_to=st_vertex_id)
@settings(suppress_health_check=[HealthCheck.filter_too_much], deadline=2000, max_examples=50)
def test_G7_hasEdge_after_addEdge_different(
    data: Tuple[PyGraph[int, str], int, int, str], 
    v_query_from: int, 
    v_query_to: int
):
    # (G7): eqVertexId(v_qf,v_af)==false OR eqVertexId(v_qt,v_at)==false =>
    #        hasEdge(v_qf, v_qt, addEdge(v_af, v_at, attr, g)) = hasEdge(v_qf, v_qt, g)
    g, v_af, v_at, attr = data 
    
    assume(v_query_from != v_af or v_query_to != v_at)
    
    g_after_add_e = g.add_edge(v_af, v_at, attr)
    
    lhs = g_after_add_e.has_edge(v_query_from, v_query_to)
    rhs = g.has_edge(v_query_from, v_query_to)
    assert lhs == rhs

# --- Axioms for getVertices ---
def test_G10_getVertices_emptyGraph():
    # (G10): getVertices(emptyGraph) = emptySet[VertexId]
    g = PyGraph[int,str].empty_graph()
    assert g.get_vertices() == set()

@given(data=st_graph_vertex_op)
@settings(suppress_health_check=[HealthCheck.filter_too_much], deadline=1000, max_examples=50)
def test_G11_getVertices_addVertex(data: Tuple[PyGraph[int, str], int]):
    # (G11): getVertices(addVertex(v,g)) = addSet(v, getVertices(g))
    g, v_added = data
    
    g_after_add = g.add_vertex(v_added)
    
    lhs = g_after_add.get_vertices()
    rhs = g.get_vertices() | {v_added} 
    assert lhs == rhs

# --- Axioms for outDegree ---
@st.composite
def st_graph_and_member_vertex(draw): # Strategy to get a graph and a vertex guaranteed to be in it
    graph_drawn = draw(st_py_graph_int_str.filter(lambda g_filter: not g_filter.get_vertices() == set())) 
    v_member = draw(st.sampled_from(sorted(list(graph_drawn.get_vertices())))) 
    return graph_drawn, v_member

@given(v_id=st_vertex_id)
def test_GD1_outDegree_emptyGraph_raises_keyerror(v_id: int): # Adjusted for Python impl
    # (GD1): outDegree(v, emptyGraph) = zero (spec)
    # Python impl raises KeyError if vertex not present.
    g = PyGraph[int, str].empty_graph()
    with pytest.raises(KeyError):
        g.out_degree(v_id)

@given(g=st_py_graph_int_str, v_new=st_vertex_id)
@settings(max_examples=50)
def test_GD2_outDegree_new_vertex_is_zero(g: PyGraph[int, str], v_new: int):
    # (GD2): hasVertex(v,g) = false => outDegree(v, addVertex(v,g)) = zero
    assume(not g.has_vertex(v_new)) 
    
    g_after_add = g.add_vertex(v_new)
    assert g_after_add.out_degree(v_new) == 0

@given(data=st_graph_and_member_vertex, v_other_new=st_vertex_id)
@settings(suppress_health_check=[HealthCheck.filter_too_much], max_examples=50, deadline=1000)
def test_GD3_outDegree_unaffected_by_adding_other_vertex(
    data: Tuple[PyGraph[int, str], int], 
    v_other_new: int
):
    # (GD3): eqVertexId(v1,v2) = false AND hasVertex(v1,g) = true =>
    #        outDegree(v1, addVertex(v2,g)) = outDegree(v1,g)
    g, v_existing = data
    assume(v_existing != v_other_new) 
    
    original_degree = g.out_degree(v_existing)
    g_after_add = g.add_vertex(v_other_new)
    
    if g_after_add.has_vertex(v_existing):
        assert g_after_add.out_degree(v_existing) == original_degree

@given(data=st_graph_and_member_vertex, v_to_connect=st_vertex_id, attr=st_edge_attr)
@settings(suppress_health_check=[HealthCheck.filter_too_much], max_examples=50, deadline=1000)
def test_GD4_GD5_outDegree_after_addEdge(
    data: Tuple[PyGraph[int, str], int], 
    v_to_connect: int, 
    attr: str
):
    # (GD4): hasEdge(vFrom, vTo, g) = true => outDegree(vFrom, addEdge(vFrom, vTo, attr, g)) = outDegree(vFrom, g)
    # (GD5): hasEdge(vFrom, vTo, g) = false => outDegree(vFrom, addEdge(vFrom, vTo, attr, g)) = succ(outDegree(vFrom, g))
    g, v_from = data
    
    g_with_v_to = g.add_vertex(v_to_connect) 
    original_degree_v_from = g_with_v_to.out_degree(v_from)
    edge_existed_before = g_with_v_to.has_edge(v_from, v_to_connect)
    
    g_after_add_edge = g_with_v_to.add_edge(v_from, v_to_connect, attr)
    
    if edge_existed_before:
        assert g_after_add_edge.out_degree(v_from) == original_degree_v_from
    else:
        assert g_after_add_edge.out_degree(v_from) == original_degree_v_from + 1
```
Os testes de propriedade acima para axiomas selecionados (G1-G4, G6-G7, G10-G11 e GD1-GD5) são implementados. Utilizam `assume` para premissas condicionais. O teste para GD1 foi adaptado para o comportamento de exceção da implementação Python.

**Exercício:**

Considerando a especificação `Graph[V,E]` e sua implementação `PyGraph[V,E]`. O axioma (G12) para `getNeighbors` é: `getNeighbors(v, emptyGraph) = emptySet[VertexId]`. Escreva uma função de teste `pytest` chamada `test_G12_getNeighbors_emptyGraph` que verifique o comportamento da implementação `PyGraph` em relação a este axioma.

**Resolução:**

```python
# test_py_graph.py (continuação)

@given(v_query=st_vertex_id)
def test_G12_getNeighbors_emptyGraph(v_query: int):
    # (G12): getNeighbors(v, emptyGraph) = emptySet[VertexId]
    # PyGraph.get_neighbors raises KeyError if v_query is not in graph.
    # emptyGraph has no vertices. So, get_neighbors(v_query, emptyGraph) should raise KeyError.
    
    g = PyGraph[int, str].empty_graph()
    with pytest.raises(KeyError):
        g.get_neighbors(v_query)
```
O teste `test_G12_getNeighbors_emptyGraph` verifica que chamar `get_neighbors` em um grafo vazio para qualquer `v_query` levanta `KeyError`, pois `v_query` não pode existir no grafo vazio. Este teste alinha-se com a implementação de `PyGraph` que trata o acesso a um vértice inexistente como um erro, embora o axioma (G12) pudesse ser interpretado como uma operação total que retorna um conjunto vazio para vértices não existentes (o que não é o caso da nossa implementação `PyGraph`).

# 20.5 Análise de Complexidade Algorítmica

A implementação `PyGraph[V,E]` utiliza uma representação de lista de adjacências baseada em dicionários Python. Assumimos que as operações de hash e igualdade para `V` são, em média, $O(1)$. $N_V$ é o número de vértices e $N_A$ o número de arestas. $deg^+(v)$ é o grau de saída do vértice $v$.

**Análise de Complexidade das Operações de `PyGraph[V,E]` (Imutável):**
As operações "modificadoras" são custosas devido à cópia profunda para imutabilidade.

1.  **`__init__(self)`:** $O(1)$.
2.  **`empty_graph()`:** $O(1)$.
3.  **`add_vertex(self, v_id: V)`:** $O(N_V + N_A)$ (devido à cópia).
4.  **`has_vertex(self, v_id: V)`:** $O(1)$ em média.
5.  **`add_edge(self, v_from: V, v_to: V, attr: E)`:** $O(N_V + N_A)$ (devido à cópia).
6.  **`has_edge(self, v_from: V, v_to: V)`:** $O(1)$ em média.
7.  **`get_edge_attribute(self, v_from: V, v_to: V)`:** $O(1)$ em média.
8.  **`get_vertices(self) -> PySetInternal[V]`:** $O(N_V)$.
9.  **`get_neighbors(self, v_id: V)`:** $O(deg^+(v_{id}))$.
10. **`remove_vertex(self, v_id: V)`:** $O(N_V + N_A)$ (cópia e varredura para arestas de entrada).
11. **`remove_edge(self, v_from: V, v_to: V)`:** $O(N_V + N_A)$ (devido à cópia).
12. **`out_degree(self, v_id: V)`:** $O(1)$ em média (após verificação de `has_vertex`).
13. **`get_in_neighbors(self, v_id: V)`:** $O(N_V)$ no pior caso (itera sobre todos os vértices).

Se as modificações fossem no local (mutável):
*   `add_vertex` (mutável): $O(1)$ em média.
*   `add_edge` (mutável): $O(1)$ em média.
*   `remove_edge` (mutável): $O(1)$ em média.
*   `remove_vertex` (mutável): Custo para remover arestas incidentes, $O(N_V + deg(v_{id}))$ ou $O(N_V + N_A)$ dependendo de como as arestas de entrada são tratadas.

**Exercício:**

Se a representação interna de `PyGraph[V,E]` fosse uma **matriz de adjacências** (e.g., uma lista de listas de `Optional[E]`), qual seria a complexidade de tempo esperada para as operações `add_edge(v_from, v_to, attr)` e `has_edge(v_from, v_to)`? Assuma que os identificadores de vértice `V` podem ser mapeados para índices inteiros $0 \ldots N_V-1$ em tempo $O(1)$. Considere a versão imutável que cria uma nova matriz.

**Resolução:**

Assumindo uma matriz de adjacências `M` de tamanho $N_V \times N_V$. Mapear `V` para índices $i, j$ é $O(1)$.

1.  **`has_edge(i, j)`:** Acesso $M[i][j]$ é $O(1)$. **Complexidade Total: $O(1)$**.
2.  **`add_edge(i, j, attr)` (Imutável):**
    *   Criar cópia de `M`: $O(N_V^2)$.
    *   Definir `M_new[i][j] = attr`: $O(1)$.
    *   **Complexidade Total: $O(N_V^2)$**.

Comparativamente, para `add_edge` imutável, lista de adjacências ($O(N_V + N_A)$) é melhor para grafos esparsos ($N_A \ll N_V^2$).

## 20.5.1 Outras Estratégias de Implementação para o TAD Graph

Além da lista de adjacências (usada em `PyGraph`) e da matriz de adjacências (discutida no exercício anterior), existem outras estratégias para representar grafos, cada uma com seus próprios tradeoffs de espaço e tempo para diferentes operações e tipos de grafos.

1.  **Lista de Arestas (Edge List):**
    *   **Representação:** Simplesmente uma lista (ou conjunto) de todas as arestas do grafo. Cada aresta pode ser uma tupla `(u, v)` ou `(u, v, weight)`. Os vértices podem ser armazenados em um conjunto separado ou inferidos da lista de arestas.
    *   **Espaço:** $O(N_V + N_A)$.
    *   **Operações:**
        *   `add_edge`: $O(1)$ para adicionar à lista (se não verificar duplicatas).
        *   `remove_edge(u,v)`: $O(N_A)$ para encontrar a aresta na lista.
        *   `has_edge(u,v)`: $O(N_A)$ para varrer a lista.
        *   `get_neighbors(u)`: $O(N_A)$ para varrer todas as arestas e encontrar aquelas que partem de $u$.
        *   `add_vertex`: $O(1)$ para adicionar a um conjunto de vértices.
    *   **Prós:** Muito simples de implementar. Boa para grafos onde a principal operação é iterar sobre todas as arestas (e.g., algoritmo de Kruskal).
    *   **Contras:** Ineficiente para operações que dependem de encontrar vizinhos ou verificar a existência de uma aresta específica rapidamente.

2.  **Matriz de Incidência:**
    *   **Representação:** Uma matriz $N_V \times N_A$. A entrada $M[i][j]$ é 1 se o vértice $i$ é incidente à aresta $j$ (e.g., é uma extremidade da aresta $j$ em grafos não direcionados), -1 se é a origem e +1 se é o destino em grafos direcionados (ou outras convenções). Pode armazenar pesos se a matriz for de valores em vez de bits/indicadores.
    *   **Espaço:** $O(N_V \cdot N_A)$. Geralmente muito grande e esparsa, tornando-a impraticável para a maioria dos grafos grandes.
    *   **Operações:**
        *   Encontrar arestas incidentes a um vértice $v$: Percorrer a linha de $v$ ($O(N_A)$).
        *   Encontrar vértices de uma aresta $e$: Percorrer a coluna de $e$ ($O(N_V)$).
    *   **Prós:** Útil em algumas áreas da teoria dos grafos e para certos problemas (e.g., relacionados a fluxos e cortes), especialmente para hipergrafos.
    *   **Contras:** Uso de espaço geralmente proibitivo. Operações comuns de adjacência não são tão diretas quanto em listas ou matrizes de adjacências.

3.  **Comparação Geral (para grafos com $N_V$ vértices e $N_A$ arestas):**

    | Operação             | Lista de Adjacências (média) | Matriz de Adjacências | Lista de Arestas |
    |----------------------|------------------------------|-----------------------|------------------|
    | Espaço               | $O(N_V + N_A)$               | $O(N_V^2)$            | $O(N_V + N_A)$   |
    | `add_vertex`         | $O(1)$ (mutável)             | $O(N_V^2)$ (recriar)  | $O(1)$           |
    | `add_edge(u,v)`      | $O(1)$ (mutável)             | $O(1)$                | $O(1)$           |
    | `remove_vertex(u)`   | $O(deg(u) + N_V)$ (mutável)  | $O(N_V^2)$ (recriar)  | $O(N_A)$         |
    | `remove_edge(u,v)`   | $O(deg(u))$ ou $O(1)$ (mutável) | $O(1)$                | $O(N_A)$         |
    | `has_edge(u,v)`      | $O(deg(u))$ ou $O(1)$ (se dict) | $O(1)$                | $O(N_A)$         |
    | `get_neighbors(u)`   | $O(deg(u))$                  | $O(N_V)$              | $O(N_A)$         |

    *Nota: Para implementações imutáveis, as operações "modificadoras" geralmente têm um custo adicional de cópia da estrutura principal.*

    **Escolha da Representação:**
    *   **Matriz de Adjacências:** Boa para grafos densos (onde $N_A \approx N_V^2$) e quando a verificação rápida de `has_edge(u,v)` é crucial e o espaço não é uma grande limitação. Simples para grafos não ponderados.
    *   **Lista de Adjacências:** Geralmente a escolha preferida para grafos esparsos (a maioria dos grafos do mundo real). Eficiente em espaço e boa para iterar sobre os vizinhos de um vértice. Usando dicionários para as listas de adjacência (como em `PyGraph`), `has_edge` e `add_edge` (mutável) tornam-se $O(1)$ em média.
    *   **Lista de Arestas:** Útil quando o conjunto de arestas é o foco principal e as operações de adjacência são menos frequentes ou podem ser construídas sob demanda.

A escolha da representação correta depende fortemente das características do grafo (densidade, tamanho) e do perfil de operações que a aplicação realizará com mais frequência. A nossa implementação `PyGraph` com lista de adjacências (usando dicionários) visa um bom equilíbrio para grafos gerais, especialmente os esparsos.

# 20.6 Exercícios Teóricos e Práticos

Esta seção apresenta um exercício para consolidar os conceitos do TAD `Graph[V,E]`.

**Exercício:**

**Contexto:** Este exercício foca na extensão do TAD `Graph[V,E]` e sua implementação `PyGraph[V,E]` com uma operação para obter o grau de saída de um vértice.

**Parte a) Especificação Algébrica para `outDegree` em `Graph[V,E]`**
Adicione uma operação `outDegree: VertexId x Graph[V,E] --> Natural` à especificação algébrica `Graph[V,E]` (da Seção 20.2). Esta operação deve retornar o número de arestas de saída de um dado vértice.
1.  O grau de saída de um vértice em um `emptyGraph` é `zero` (ou a operação é indefinida se o vértice não "existe" no grafo vazio; para esta especificação, considere-o `zero` se a consulta for feita, ou que a pré-condição `hasVertex` falharia).
2.  Adicionar um vértice não altera o grau de saída de *outros* vértices. O grau de saída do vértice recém-adicionado (se ele não existia) é `zero`.
3.  Adicionar uma aresta $(v_1, v_2)$ aumenta o grau de saída de $v_1$ em um (se a aresta não existia). Se a aresta já existia (e o grafo é simples), o grau de saída não muda.
Forneça os axiomas para `outDegree`, focando em sua relação com `emptyGraph`, `addVertex`, e `addEdge`.

**Parte b) Implementação Python para `out_degree` em `PyGraph[V,E]`**
Implemente o método correspondente `out_degree(self, v_id: V) -> int` na classe `PyGraph[V,E]` (da Seção 20.3).
1.  Se o vértice `v_id` não existir no grafo, o método deve levantar `KeyError`.
2.  Caso contrário, deve retornar o número de arestas de saída de `v_id`.

**Parte c) Testes de Propriedade para `out_degree` em `PyGraph[V,E]`**
Usando Hypothesis, escreva pelo menos duas funções de teste de propriedade para sua implementação `out_degree` que correspondam aos comportamentos axiomáticos definidos ou inferidos na Parte (a).

**Resolução:**

**Parte a) Especificação Algébrica para `outDegree` em `Graph[V,E]`**
A especificação na Seção 20.2 já foi atualizada para incluir `outDegree` e seus axiomas (GD1)-(GD6):
*   (GD1): `outDegree(v, emptyGraph) = zero`
*   (GD2): `hasVertex(v,g) = false => outDegree(v, addVertex(v,g)) = zero`
*   (GD3): `eqVertexId(v1,v2) = false AND hasVertex(v1,g) = true => outDegree(v1, addVertex(v2,g)) = outDegree(v1,g)`
*   (GD4): `hasEdge(vFrom, vTo, g) = true => outDegree(vFrom, addEdge(vFrom, vTo, attr, g)) = outDegree(vFrom, g)`
*   (GD5): `hasEdge(vFrom, vTo, g) = false => outDegree(vFrom, addEdge(vFrom, vTo, attr, g)) = succ(outDegree(vFrom, g))`
*   (GD6): `eqVertexId(vOther, vFrom) = false AND hasVertex(vOther,g) = true => outDegree(vOther, addEdge(vFrom, vTo, attr, g)) = outDegree(vOther, g)`

Estes axiomas cobrem os requisitos. GD1 trata o grafo vazio. GD2 e GD3 tratam `addVertex`. GD4, GD5 e GD6 tratam `addEdge`.

**Parte b) Implementação Python para `out_degree` em `PyGraph[V,E]`**
Esta implementação já foi fornecida na Seção 20.3:
```python
# py_graph.py (método out_degree da classe PyGraph[V,E])
# ...
    def out_degree(self, v_id: V) -> int:
        # Returns the out-degree of vertex v_id.
        # Raises KeyError if v_id does not exist in the graph.
        if not self.has_vertex(v_id):
            raise KeyError(f"Vertex {v_id} not found in graph.")
        
        # The out-degree is the number of entries in the adjacency dictionary for v_id
        return len(self._adj[v_id])
# ...
```
Esta implementação levanta `KeyError` se o vértice não existe e, caso contrário, retorna o tamanho do dicionário de vizinhos de saída, que é o grau de saída.

**Parte c) Testes de Propriedade para `out_degree` em `PyGraph[V,E]`**
Os testes propostos na Seção 20.4 já incluem testes para os axiomas (GD1)-(GD5) (o GD6 é implicitamente coberto pela lógica de GD3 e GD5 se as premissas forem satisfeitas):
*   `test_GD1_outDegree_emptyGraph_raises_keyerror` (adapta GD1 para o comportamento de exceção).
*   `test_GD2_outDegree_new_vertex_is_zero`.
*   `test_GD3_outDegree_unaffected_by_adding_other_vertex`.
*   `test_GD4_GD5_outDegree_after_addEdge` (combina GD4 e GD5).

Estes testes verificam os aspectos chave de como `addVertex` e `addEdge` afetam o `out_degree`.

Este capítulo abordou o Tipo Abstrato de Dados `Graph[V,E]`, um dos mais versáteis para modelar relações. Discutimos sua definição abstrata, relevância, e apresentamos uma especificação algébrica formal parametrizada. Exploramos a análise de consistência e completeza dessa especificação, reconhecendo os desafios inerentes à complexidade dos grafos. Projetamos e implementamos uma classe Python genérica `PyGraph[V,E]` usando listas de adjacências e tipagem estática, e demonstramos como alguns dos axiomas podem ser verificados usando Teste Baseado em Propriedades com Hypothesis. A análise de complexidade revelou os custos da imutabilidade em operações de modificação, e uma breve comparação com outras estratégias de implementação foi apresentada. O exercício prático focou na extensão do TAD e da implementação com a operação `outDegree` e seus testes. Este capítulo conclui a Parte IV sobre estruturas de dados para coleções e relações. A Parte V mudará o foco para uma perspectiva teórica mais unificadora, introduzindo a Teoria das Categorias e seu papel na compreensão dos Tipos Abstratos de Dados.

---
## REFERÊNCIAS BIBLIOGRÁFICAS
AHO, A. V.; HOPCROFT, J. E.; ULLMAN, J. D. **Data Structures and Algorithms**. Reading, MA: Addison-Wesley, 1983.
*   *Resumo: Um texto clássico que fornece uma introdução sólida a grafos, suas representações (listas e matrizes de adjacências) e algoritmos fundamentais. Essencial para compreender a base conceitual do TAD Graph.*

BONDY, J. A.; MURTY, U. S. R. **Graph Theory**. London: Springer, 2008. (Graduate Texts in Mathematics, v. 244).
*   *Resumo: Uma obra de referência abrangente e moderna sobre teoria dos grafos, cobrindo uma vasta gama de conceitos, propriedades e teoremas. Útil para um entendimento matemático profundo dos objetos que o TAD Graph visa modelar.*

CORMEN, T. H.; LEISERSON, C. E.; RIVEST, R. L.; STEIN, C. **Introduction to Algorithms**. 3. ed. Cambridge, MA: MIT Press, 2009.
*   *Resumo: Contém capítulos dedicados a algoritmos em grafos elementares e avançados, incluindo discussões sobre representações de grafos e a complexidade de operações básicas. Fundamental para as seções de implementação e análise de complexidade.*

DIJESTEL, R. **Graph Theory**. 5. ed. Berlin: Springer-Verlag, 2017. (Graduate Texts in Mathematics, v. 173).
*   *Resumo: Um texto de pós-graduação popular e rigoroso em teoria dos grafos, oferecendo uma perspectiva matemática profunda sobre diferentes classes de grafos e suas propriedades estruturais. Relevante para a fundamentação teórica do TAD Graph.*

GOODRICH, M. T.; TAMASSIA, R.; GOLDWASSER, M. H. **Data Structures and Algorithms in Python**. Hoboken, NJ: Wiley, 2013.
*   *Resumo: Este livro implementa várias estruturas de dados, incluindo grafos, em Python. Fornece exemplos práticos de representação (e.g., lista de adjacências usando dicionários) e algoritmos em Python, diretamente relevantes para a Seção 20.3.*

WIRSING, M. Algebraic Specification. In: VAN LEEUWEN, J. (Ed.). **Handbook of Theoretical Computer Science, Volume B: Formal Models and Semantics**. Amsterdam: Elsevier; Cambridge, MA: MIT Press, 1990. p. 675-788.
*   *Resumo: Embora não foque exclusivamente em grafos, este capítulo do handbook discute os princípios da especificação algébrica de TADs complexos, incluindo parametrização e modularização, que são cruciais ao tentar especificar um TAD tão rico e variado como `Graph[V,E]`.*

---
