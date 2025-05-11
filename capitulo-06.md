---
# Capítulo 6
# Validação de Implementações via Teste Baseado em Propriedades com Hypothesis
---

Após a exploração do processo de especificação formal de Tipos Abstratos de Dados (TADs) através da abordagem algébrica, da análise de suas propriedades como consistência e completeza, e do processo de refinamento que nos aproxima de uma implementação, este capítulo se dedica a uma etapa crucial no ciclo de desenvolvimento de software rigoroso: a **validação de implementações concretas** em relação às suas especificações formais. Especificamente, focaremos no **Teste Baseado em Propriedades (PBT - Property-Based Testing)** como uma técnica poderosa para verificar se o código Python que implementa um TAD adere ao comportamento ditado pelos axiomas de sua especificação algébrica. O capítulo iniciará estabelecendo a conexão fundamental entre os axiomas algébricos (equações que definem o comportamento) e os testes executáveis, argumentando que os axiomas podem servir como "oráculos" para a verificação do código. Contrastaremos as limitações dos testes tradicionais baseados em exemplos manuais com a capacidade do PBT de explorar um espaço de entrada muito mais vasto e descobrir casos de borda inesperados. Em seguida, introduziremos os princípios fundamentais do PBT e apresentaremos a biblioteca **Hypothesis** para Python como uma ferramenta prática e expressiva para sua implementação. Detalharemos como definir estratégias de geração de dados em Hypothesis, incluindo estratégias para tipos primitivos, coleções, e, crucialmente, para instâncias de classes customizadas que representam nossos TADs, incluindo o uso de `st.builds` e `st.recursive` para estruturas de dados complexas e recursivas. A formulação de propriedades usando o decorador `@given` e a escrita de asserções que correspondem aos axiomas serão centrais. Exploraremos também estratégias avançadas de geração de dados, como a composição e mapeamento de estratégias e a geração de dados dependentes. O importante processo de "shrinking" (encolhimento) de contraexemplos, uma característica distintiva do Hypothesis que facilita a depuração, será explicado. A integração de Hypothesis com frameworks de teste populares, como `pytest`, também será abordada. Para consolidar todos esses conceitos, um exemplo prático detalhado será desenvolvido: a implementação em Python do TAD `Natural` (com tipagem estática MyPy) e a subsequente escrita e execução de testes de propriedade com Hypothesis para validar os axiomas de suas operações, como `add` e `mult`, demonstrando assim a sinergia entre especificação formal, implementação rigorosa e teste avançado.

# 6.1 A Ponte entre Axiomas e Testes Executáveis

As especificações algébricas, como vimos nos capítulos anteriores, definem o comportamento de um Tipo Abstrato de Dados (TAD) através de um conjunto de axiomas equacionais. Estes axiomas, da forma $t_1 = t_2$, são declarações formais sobre as relações entre as operações do TAD, válidas para todas as instanciações de suas variáveis. Tradicionalmente, tais axiomas são usados para raciocínio lógico, provas de propriedades e para guiar a derivação de implementações corretas. No entanto, eles também podem servir a um propósito pragmático e poderoso no contexto da verificação de software: eles podem atuar como a **base para a geração de testes executáveis**. Esta seção explora como os axiomas algébricos, que são essencialmente propriedades universais do TAD, podem ser transformados em testes concretos que validam se uma implementação em código (e.g., Python) se comporta de acordo com sua especificação formal. Discutiremos também as limitações inerentes aos testes baseados em exemplos manuais, que frequentemente falham em cobrir a vasta gama de comportamentos e casos de borda que os axiomas, por sua natureza universal, abrangem.

**Axiomas como Oráculos para Testes de Software**

No jargão de teste de software, um **oráculo de teste** é um mecanismo que determina se o resultado de um teste (o comportamento observado do sistema sob teste) é correto ou não. Para um dado conjunto de entradas de teste, o oráculo decide se a saída produzida ou o estado resultante do sistema está em conformidade com o esperado.

Os axiomas de uma especificação algébrica $SP = (\Sigma, E)$ podem ser vistos como uma fonte rica e formal de oráculos de teste para qualquer implementação $I_{SP}$ do TAD descrito por $SP$. Cada axioma $l=r$ (onde $l$ e $r$ são termos, possivelmente com variáveis universalmente quantificadas) postula uma propriedade que deve ser verdadeira para todas as instâncias. Se traduzirmos as operações de $\Sigma$ para funções/métodos em $I_{SP}$, então, para qualquer atribuição de valores concretos às variáveis do axioma:
*   A avaliação do termo $l$ usando as funções/métodos de $I_{SP}$ deve produzir um resultado que seja "igual" (de acordo com uma noção apropriada de igualdade na implementação) ao resultado da avaliação do termo $r$ usando as mesmas funções/métodos.

**Exemplo:**
Considere o TAD `Natural` e o axioma para adição:
(N3): `add(n, zero) = n` (para todo `n: Natural`)

Se tivermos uma implementação em Python com uma classe `PyNatural` e métodos `py_add(self, other_natural)` e uma representação para `py_zero`, este axioma se traduz na seguinte propriedade testável:
"Para qualquer instância `n_val` de `PyNatural`, o resultado de `n_val.py_add(py_zero)` deve ser igual a `n_val`."

Aqui, o axioma (N3) atua como o oráculo: ele nos diz qual deve ser o resultado esperado (`n`) quando `add` é aplicado a `n` e `zero`. Um caso de teste consistiria em:
1.  Criar/escolher um valor `n_val` para `n`.
2.  Criar/obter o valor `py_zero`.
3.  Computar `resultado_obtido = n_val.py_add(py_zero)`.
4.  Computar `resultado_esperado = n_val`.
5.  Verificar se `resultado_obtido` é igual a `resultado_esperado`.

Esta verificação de igualdade pode ser feita usando o método `__eq__` da classe `PyNatural`, que, idealmente, também seria projetado para refletir a semântica de igualdade do TAD `Natural`.

**Transformando Axiomas em Propriedades Testáveis:**
O processo geral é:
1.  **Identificar Variáveis e seus Tipos (Sorts):** Para um axioma $l=r$ com variáveis $x_1:S_1, \ldots, x_k:S_k$.
2.  **Gerar Instâncias Concretas para Variáveis:** Criar ou gerar valores $v_1, \ldots, v_k$ que são instâncias das implementações dos sorts $S_1, \ldots, S_k$.
3.  **Avaliar Lados da Equação:**
    *   Computar $LHS_{val} = \text{evaluate}(l, [x_1 \mapsto v_1, \ldots, x_k \mapsto v_k])$ usando a implementação do TAD.
    *   Computar $RHS_{val} = \text{evaluate}(r, [x_1 \mapsto v_1, \ldots, x_k \mapsto v_k])$ usando a implementação do TAD.
4.  **Comparar Resultados:** Verificar se $LHS_{val}$ é igual a $RHS_{val}$ na implementação.

Se esta igualdade se mantiver para uma grande e diversificada gama de instâncias $v_i$, aumenta-se a confiança de que a implementação satisfaz o axioma. Se uma instância for encontrada para a qual a igualdade não se sustenta (um **contraexemplo**), então um bug (uma discrepância entre a implementação e a especificação) foi encontrado.

O Teste Baseado em Propriedades (PBT), que será detalhado com Hypothesis, automatiza os passos 2, 3 e 4, especialmente a geração de diversas instâncias $v_i$ e a busca por contraexemplos. O programador apenas precisa "traduzir" o axioma para uma asserção de código.

**Limitações de Testes Baseados em Exemplos Manuais**

A abordagem tradicional para testar software, e por extensão, implementações de TADs, é o **teste baseado em exemplos** (example-based testing). Nesta abordagem, o desenvolvedor ou testador seleciona manualmente um pequeno conjunto de entradas específicas (casos de teste) para cada função ou método e verifica se a saída produzida é a esperada para essas entradas.

**Exemplo (para o axioma `add(n, zero) = n`):**
Um teste manual poderia ser:
*   `test_add_zero_to_five()`:
    `n_val = PyNatural(5)`
    `py_zero = PyNatural(0)`
    `assert n_val.py_add(py_zero) == n_val`
*   `test_add_zero_to_zero()`:
    `n_val = PyNatural(0)`
    `py_zero = PyNatural(0)`
    `assert n_val.py_add(py_zero) == n_val`

Embora úteis, os testes baseados em exemplos manuais sofrem de várias limitações, especialmente quando se tenta validar propriedades universais como os axiomas de TADs:

1.  **Cobertura Limitada do Espaço de Entrada:** Os axiomas são universalmente quantificados (devem valer para *todos* os valores possíveis das variáveis). Testes manuais só podem cobrir um número muito pequeno e finito de instâncias. É impraticável testar manualmente todos os naturais ou todas as listas possíveis.

2.  **Viés na Seleção de Casos de Teste:** Os desenvolvedores tendem a escolher exemplos que são "típicos" ou que eles sabem que deveriam funcionar, ou alguns casos de borda óbvios (e.g., zero, lista vazia, um). Eles podem, inconscientemente, evitar ou não pensar em combinações de entradas mais complexas, obscuras ou inesperadas que poderiam revelar bugs.

3.  **Dificuldade em Encontrar Casos de Borda Sutis:** Muitos bugs residem em casos de borda ou interações inesperadas entre diferentes valores ou estados. Estes são difíceis de prever e codificar manualmente em testes. Por exemplo, para uma lista, o que acontece com strings muito longas, ou listas contendo listas, ou listas com elementos que têm implementações de `__eq__` ou `__hash__` incomuns?

4.  **Manutenção e Escalabilidade dos Testes:** À medida que o TAD evolui ou se torna mais complexo, o número de casos de teste manuais necessários para manter uma cobertura razoável pode crescer exponencialmente, tornando a suíte de testes difícil de manter e demorada para executar.

5.  **Foco em Instâncias Específicas, Não em Propriedades Gerais:** Testes baseados em exemplos verificam se o código funciona para *aqueles exemplos específicos*. Eles não verificam diretamente se a *propriedade geral* (o axioma) é válida para todas as entradas. Um teste que passa para `add(5,0)=5` não garante que `add(n,0)=n` funcione para $n=123456789$ ou para $n$ sendo um `PyNatural` construído de uma forma particular.

Em contraste, o Teste Baseado em Propriedades (PBT) tenta superar essas limitações. Em vez de o programador fornecer os dados de entrada, ele fornece a **propriedade** (derivada do axioma) e uma **descrição dos tipos de dados de entrada**. A ferramenta de PBT então gera automaticamente um grande número de entradas diversas, tentando encontrar um contraexemplo que falsifique a propriedade. Isso permite uma exploração muito mais ampla e menos enviesada do espaço de entrada, com uma maior chance de descobrir bugs que os testes manuais não pegariam.

A ponte entre axiomas e testes executáveis é, portanto, fortalecida e automatizada pelo PBT, transformando as leis abstratas dos TADs em verificações concretas e poderosas sobre suas implementações.

**Exercício:**

Considere o TAD `Boolean` com a operação `and: Boolean x Boolean -> Boolean` e o axioma (B3): `and(true, b) = b` (para toda variável `b: Boolean`).
a) Descreva um ou dois casos de teste manuais (baseados em exemplos) que você escreveria para uma implementação Python `py_and(b1: bool, b2: bool)` deste axioma.
b) Qual é a principal limitação desses testes manuais em termos de verificar a validade universal do axioma (B3)?

**Resolução:**

a) **Casos de Teste Manuais para `py_and(b1: bool, b2: bool)` baseados no axioma `and(true, b) = b`:**

O axioma é `and(true, b) = b`. Isso significa que o primeiro argumento para `py_and` deve ser a representação de `true` (que em Python é `True`), e o segundo argumento `b` pode ser `True` ou `False`.

*   **Caso de Teste 1: `b` é `true` (Python `True`)**
    *   Entrada para `py_and`: `py_and(True, True)`
    *   Lado esquerdo do axioma instanciado: `and(true, true)`
    *   Lado direito do axioma instanciado: `true`
    *   Verificação: `assert py_and(True, True) == True`

*   **Caso de Teste 2: `b` é `false` (Python `False`)**
    *   Entrada para `py_and`: `py_and(True, False)`
    *   Lado esquerdo do axioma instanciado: `and(true, false)`
    *   Lado direito do axioma instanciado: `false`
    *   Verificação: `assert py_and(True, False) == False`

Esses dois casos de teste cobrem todas as instanciações possíveis para a variável `b` no axioma (B3), dado que o sort `Boolean` tem apenas dois valores.

b) **Principal Limitação dos Testes Manuais em Termos de Verificar a Validade Universal do Axioma (B3):**

Para o TAD `Boolean`, que tem um domínio muito pequeno (apenas dois valores: `true` e `false`), os testes manuais descritos acima são, na verdade, **exaustivos** para o axioma `and(true, b) = b`. Eles cobrem todas as instanciações possíveis da variável `b`. Portanto, *neste caso específico do TAD `Boolean` e deste axioma particular*, a limitação usual de "cobertura limitada do espaço de entrada" dos testes manuais não é tão pronunciada, pois o espaço de entrada para `b` é totalmente coberto.

No entanto, se considerarmos o espírito da pergunta em um contexto mais geral (ou se o TAD tivesse um domínio maior, como `Natural`), a principal limitação seria:

*   **Não Verificação da Universalidade Intrínseca:** Mesmo que os testes passem, eles apenas confirmam que a propriedade `and(true, b) = b` é válida para os valores específicos de `b` que foram testados (`True` e `False`). Eles não provam que a *implementação* de `py_and` satisfaz a propriedade *devido à sua lógica interna geral* de uma forma que se alinharia com o quantificador universal "para todo `b`". Por exemplo, se `py_and` fosse implementada erroneamente como:
    ```python
    def py_and(b1: bool, b2: bool) -> bool:
        if b1 == True and b2 == True: return True
        if b1 == True and b2 == False: return False
        # Outros casos podem estar errados ou ausentes
        if b1 == False and b2 == True: return False # Correto
        if b1 == False and b2 == False: return False # Correto
        # Se houver algum outro tipo de 'bool' (impossível em Python padrão, mas conceitualmente)
        # ou se a lógica fosse mais complexa, a propriedade poderia falhar.
    ```
    Para `Boolean`, os dois testes manuais são suficientes. Mas se `b` fosse de um tipo com muitos valores (e.g., `Natural`), testar `and(constant, n)` para $n=0$ e $n=1$ não garantiria que funciona para $n=1000$. A limitação é que testes baseados em exemplos não exploram a estrutura da implementação para garantir que a lei se mantém para *todas* as entradas devido à generalidade do código, mas apenas para as instâncias testadas.

    Com PBT, mesmo para `Boolean`, a ferramenta geraria `True` e `False` para `b`, mas a filosofia é que a propriedade é definida e a ferramenta explora o domínio. Para domínios maiores, essa exploração automática se torna indispensável para ganhar confiança na universalidade. A limitação dos testes manuais é a confiança que eles dão sobre a universalidade do axioma, que é inerentemente menor do que a confiança dada por PBT (para domínios grandes) ou uma prova formal.

Para ser mais preciso sobre a limitação mesmo para `Boolean`: os testes manuais verificam instâncias. Se a implementação fosse `def py_and(b1, b2): if b1==True and b2==True: return True; if b1==True and b2==False: return False; else: return "bug"`, os testes passariam, mas a função não retorna sempre um `bool`, violando o tipo de retorno da especificação, o que PBT com verificação de tipo (ou MyPy) poderia pegar de forma mais robusta se a geração de dados fosse mais ampla ou se o tipo de retorno fosse inspecionado. No entanto, para o axioma específico e o tipo `Boolean`, a cobertura manual é invulgarmente boa. A verdadeira limitação se manifesta com tipos de dados com domínios infinitos ou muito grandes.

# 6.2 Introdução ao Teste Baseado em Propriedades (PBT) e a Biblioteca Hypothesis

O Teste Baseado em Propriedades (PBT) é uma técnica de teste de software que se diferencia fundamentalmente do teste baseado em exemplos tradicional. Em vez de o testador selecionar manualmente entradas específicas e verificar saídas correspondentes, no PBT, o testador define **propriedades** que o código sob teste deve satisfazer para uma ampla gama de entradas. Uma propriedade é uma asserção de alto nível sobre o comportamento do código que deve ser verdadeira para todas (ou um subconjunto bem definido de) entradas válidas. Uma ferramenta de PBT então gera automaticamente um grande número de exemplos de dados de entrada diversificados e, frequentemente, complexos, e executa a propriedade com esses dados, procurando por **contraexemplos** – entradas que violem a propriedade. Se um contraexemplo é encontrado, a ferramenta o reporta, idealmente após tentar "encolhê-lo" para a forma mais simples possível que ainda demonstre a falha, facilitando a depuração. A biblioteca **Hypothesis** é uma implementação avançada e popular de PBT para Python, conhecida por sua capacidade de gerar dados complexos e por suas poderosas estratégias de encolhimento. Esta seção introduzirá os princípios fundamentais do PBT e como eles são concretizados pela Hypothesis.

**Princípios Fundamentais do PBT**

O Teste Baseado em Propriedades se baseia em alguns princípios chave:

1.  **Especificação de Propriedades em Vez de Exemplos:** O foco está em declarar o que deve ser verdade sobre o comportamento do código de forma geral, em vez de verificar instâncias específicas. Uma propriedade é uma função que toma dados de entrada gerados e faz asserções sobre o resultado da execução do código com esses dados.
    *   Exemplo de propriedade para uma função `sort(list)`: "Para qualquer lista de entrada `L`, a lista resultante `sorted_L = sort(L)` deve estar ordenada." Outra propriedade: "Para qualquer lista `L`, `sort(L)` deve ser uma permutação de `L`."

2.  **Geração Automática de Dados de Teste:** A ferramenta de PBT é responsável por gerar os dados de entrada que serão usados para testar as propriedades. O desenvolvedor fornece "estratégias" que descrevem o tipo e as características dos dados a serem gerados (e.g., inteiros, strings, listas de floats, datas, objetos customizados).
    *   A geração é frequentemente aleatória, mas guiada por heurísticas para cobrir casos de borda e valores "interessantes" que têm maior probabilidade de revelar bugs.

3.  **Busca por Contraexemplos:** O objetivo da ferramenta de PBT não é provar que a propriedade é verdadeira (o que geralmente requer prova formal), mas sim tentar vigorosamente **falsificá-la** encontrando um contraexemplo. Se, após gerar um grande número de casos de teste diversificados, nenhum contraexemplo for encontrado, a confiança na corretude da propriedade (e do código) aumenta significativamente.

4.  **Encolhimento (Shrinking) de Contraexemplos:** Quando um contraexemplo é encontrado, ele pode ser grande e complexo, tornando difícil entender a causa raiz do bug. Ferramentas de PBT como Hypothesis implementam algoritmos de "encolhimento" que tentam simplificar o contraexemplo, removendo partes ou reduzindo valores, enquanto ainda mantêm a falha. Isso resulta em um caso de teste minimalista que demonstra o bug, o que é imensamente valioso para a depuração.
    *   Exemplo: Se uma propriedade sobre uma lista falha para `[10, 0, 20, 5, -50]`, Hypothesis pode tentar encolhê-la para `[0, -50]` ou `[10, 0]` se essas listas menores ainda causarem a falha.

5.  **Reprodutibilidade:** Os testes de propriedade devem ser reprodutíveis. Ferramentas de PBT geralmente permitem que a semente (seed) do gerador de números aleatórios seja fixada ou reportada quando um erro ocorre, para que o mesmo contraexemplo possa ser regenerado.

PBT complementa o teste baseado em exemplos. Testes de exemplo são bons para verificar comportamentos específicos conhecidos e como documentação de casos de uso. PBT é excelente para explorar o comportamento do código de forma mais ampla, testar invariantes e descobrir bugs inesperados.

**Geração de Dados Aleatórios e Estratégias em Hypothesis (`strategies`)**

Em Hypothesis, a geração de dados de teste é controlada por **estratégias**. Uma estratégia é um objeto que descreve como gerar dados de um determinado tipo ou com certas características. Hypothesis fornece um módulo `hypothesis.strategies` (comumente importado como `st`) que contém uma vasta coleção de estratégias pré-definidas e ferramentas para construir estratégias customizadas.

**Estratégias Comuns:**
*   **Tipos Primitivos:**
    *   `st.integers(min_value=None, max_value=None)`: Gera inteiros. Pode-se especificar um intervalo.
    *   `st.floats(min_value=None, max_value=None, allow_nan=False, allow_infinity=False)`: Gera números de ponto flutuante.
    *   `st.booleans()`: Gera `True` ou `False`.
    *   `st.text(alphabet=st.characters(), min_size=0, max_size=None)`: Gera strings de texto. O `alphabet` pode ser customizado.
    *   `st.binary(min_size=0, max_size=None)`: Gera bytes.
*   **Datas e Horas:**
    *   `st.dates()`, `st.times()`, `st.datetimes()`, `st.timedeltas()`.
*   **Coleções:**
    *   `st.lists(elements, min_size=0, max_size=None, unique_by=None, unique=False)`: Gera listas. `elements` é outra estratégia que descreve os elementos da lista.
    *   `st.tuples(*args)`: Gera tuplas onde cada elemento vem de uma estratégia correspondente em `args`.
    *   `st.sets(elements, min_size=0, max_size=None)`: Gera conjuntos.
    *   `st.dictionaries(keys, values, min_size=0, max_size=None)`: Gera dicionários. `keys` e `values` são estratégias.
*   **Estratégias de Composição e Utilitárias:**
    *   `st.just(value)`: Gera sempre o mesmo `value`. Útil para constantes.
    *   `st.sampled_from(elements)`: Gera valores amostrados de uma coleção `elements` dada.
    *   `st.one_of(*strategies)`: Gera um valor a partir de uma das estratégias fornecidas.
    *   `st.builds(target, *args_strategies, **kwargs_strategies)`: Gera instâncias de uma classe `target` chamando seu construtor com argumentos gerados pelas `args_strategies` e `kwargs_strategies`. Crucial para testar TADs implementados como classes.
    *   `st.recursive(base, extend, max_leaves=100)`: Permite definir estratégias para tipos de dados recursivos (como árvores ou listas encadeadas). `base` é uma estratégia para o caso base, e `extend` é uma função que recebe uma estratégia para o tipo recursivo e retorna uma estratégia para o passo recursivo.
    *   `st.deferred(lambda: strategy)`: Para quebrar ciclos de dependência na definição de estratégias recursivas.

**Exemplo de Uso de Estratégias:**
```python
from hypothesis import strategies as st

# Estratégia para gerar uma lista de até 10 inteiros entre 0 e 100
strategy_for_natural_list = st.lists(st.integers(min_value=0, max_value=100), max_size=10)

# Estratégia para gerar um par (booleano, string)
strategy_for_pair = st.tuples(st.booleans(), st.text(max_size=5))

# Suponha uma classe Point(x: int, y: int)
# class Point:
#     def __init__(self, x: int, y: int):
#         self.x = x
#         self.y = y
# strategy_for_point = st.builds(Point, st.integers(), st.integers())
```
O parágrafo acima descreve como definir estratégias em Hypothesis. A primeira, `strategy_for_natural_list`, gera listas de inteiros não-negativos (simulando `Natural`) até 100, com no máximo 10 elementos. A segunda, `strategy_for_pair`, gera tuplas contendo um booleano e uma string de até 5 caracteres. A terceira (comentada, mas conceitual) mostra como `st.builds` poderia ser usado para gerar instâncias de uma classe `Point`, passando estratégias para inteiros para seus argumentos de construtor `x` e `y`.

Hypothesis usa essas estratégias para explorar o espaço de entrada de forma inteligente. Ela não apenas gera valores aleatórios simples, mas também tenta valores de borda (zero, máximo/mínimo, strings vazias, NaN para floats, etc.) e combinações que são mais propensas a encontrar problemas.

**O Decorador `@given` e a Formulação de Propriedades**

Em Hypothesis, uma propriedade é tipicamente uma função de teste que é decorada com `@given(...)`. O decorador `@given` recebe uma ou mais estratégias como argumentos. Quando o teste é executado (geralmente por um executor de testes como `pytest`), Hypothesis chama a função de teste múltiplas vezes (por padrão, 100 vezes, mas configurável), cada vez com dados gerados pelas estratégias fornecidas.

**Formato de uma Função de Teste com `@given`:**
```python
from hypothesis import given
from hypothesis import strategies as st
# Assume a function to test, e.g., my_function(arg1, arg2)

@given(strategy_for_arg1, strategy_for_arg2)
def test_my_property(generated_arg1, generated_arg2):
    # Code that uses generated_arg1 and generated_arg2
    # to call my_function or other code under test.
    result = my_function(generated_arg1, generated_arg2)
    # Assertions that check if the property holds for this result.
    assert some_condition(result, generated_arg1, generated_arg2)
```
Neste esqueleto, `strategy_for_arg1` e `strategy_for_arg2` são objetos de estratégia Hypothesis. Para cada execução do teste, `generated_arg1` receberá um valor de `strategy_for_arg1`, e `generated_arg2` um valor de `strategy_for_arg2`. A função `test_my_property` então usa esses valores gerados para interagir com o código que está sendo testado e faz asserções (`assert`) sobre o resultado para verificar se a propriedade desejada é mantida.

**Exemplo de Propriedade (para `abs(x) >= 0`):**
```python
from hypothesis import given
from hypothesis import strategies as st

@given(st.integers())
def test_abs_is_always_non_negative(x: int):
    assert abs(x) >= 0
```
Neste exemplo, `test_abs_is_always_non_negative` é uma propriedade que testa se o valor absoluto de qualquer inteiro `x` (gerado por `st.integers()`) é sempre não-negativo. Hypothesis irá chamar esta função com muitos inteiros diferentes (positivos, negativos, zero, grandes, pequenos). Se `abs(x)` alguma vez resultar em um número negativo (o que é impossível para a função `abs` padrão, mas poderia acontecer para uma implementação customizada com bug), a asserção falhará, e Hypothesis reportará `x` como um contraexemplo.

A combinação de estratégias de geração de dados flexíveis e o decorador `@given` torna a escrita de testes baseados em propriedades concisa e poderosa em Hypothesis, permitindo que os desenvolvedores foquem na especificação das propriedades em si, enquanto a biblioteca cuida da geração de dados e da busca por falhas. Isso é particularmente valioso para validar a conformidade de implementações de TADs com seus axiomas algébricos, onde os axiomas se traduzem naturalmente em propriedades.

**Exercício:**

Suponha que você tem uma função Python `implement_add_natural(n1: int, n2: int) -> int` que visa implementar a operação `add: Natural x Natural -> Natural` do TAD `Natural`. Um dos axiomas para `add` é `add(n, zero) = n`. Usando Hypothesis, esboce como você escreveria um teste de propriedade para verificar este axioma. Indique quais estratégias você usaria e como seria a função de teste. (Assuma que `zero` do TAD `Natural` é representado pelo inteiro `0` em Python).

**Resolução:**

Para testar o axioma `add(n, zero) = n` para a implementação `implement_add_natural(n1: int, n2: int) -> int` usando Hypothesis, onde `zero` é representado por `0`:

1.  **Identificar Variáveis e Estratégias:**
    *   O axioma tem uma variável `n` do sort `Natural`. Em Python, representaremos `Natural` por `int` não-negativo.
    *   A estratégia para `n` será `st.integers(min_value=0)` para gerar inteiros não-negativos.
    *   A constante `zero` do TAD é representada pelo inteiro `0`. Podemos usar `st.just(0)` ou simplesmente o literal `0`.

2.  **Escrever a Função de Teste com `@given`:**
    A função de teste receberá um valor para `n` gerado pela estratégia. Ela então chamará `implement_add_natural` com `n` e `0` e verificará se o resultado é igual a `n`.

**Esboço do Teste de Propriedade:**
```python
from hypothesis import given
from hypothesis import strategies as st

# Assume this function is defined elsewhere:
# def implement_add_natural(n1: int, n2: int) -> int:
#     # ... implementation ...
#     return n1 + n2 # Example correct implementation

# For the test, we might redefine it or import it.
# Let's assume it's available.
def implement_add_natural(n1: int, n2: int) -> int:
    # Placeholder for the actual implementation to be tested
    return n1 + n2 

@given(st.integers(min_value=0)) # Strategy for the variable 'n' (Natural number)
def test_add_natural_with_zero_identity(n: int) -> None:
    """
    Tests the property corresponding to the axiom: add(n, zero) = n
    In Python terms: implement_add_natural(n, 0) == n
    """
    # The 'zero' value from the axiom
    natural_zero: int = 0
    
    # Call the implementation with 'n' and 'natural_zero'
    result = implement_add_natural(n, natural_zero)
    
    # Assert that the result is equal to 'n'
    assert result == n

# To run: pytest test_file_name.py
```
Neste esboço:
*   `@given(st.integers(min_value=0))` instrui Hypothesis a gerar valores inteiros não-negativos `n`.
*   A função `test_add_natural_with_zero_identity` recebe este `n` gerado.
*   Dentro da função, `natural_zero` é definido como `0`.
*   A função `implement_add_natural(n, natural_zero)` é chamada.
*   A asserção `assert result == n` verifica se a propriedade (axioma) é válida para o `n` gerado.

Hypothesis executará este teste múltiplas vezes com diferentes valores de `n` (e.g., 0, 1, números grandes, etc.). Se para algum `n`, `implement_add_natural(n, 0)` não for igual a `n`, Hypothesis reportará uma falha com o valor de `n` que causou o problema (após tentar encolhê-lo).

# 6.3 Estratégias Avançadas de Geração de Dados

Embora as estratégias básicas fornecidas por Hypothesis (como `st.integers`, `st.lists`, `st.text`) sejam poderosas para tipos de dados comuns, a testagem de Tipos Abstratos de Dados (TADs) complexos ou estruturas de dados customizadas frequentemente requer a construção de estratégias de geração de dados mais sofisticadas. Hypothesis oferece mecanismos avançados para criar tais estratégias, permitindo que os desenvolvedores gerem instâncias de suas próprias classes, construam estruturas de dados recursivas, componham e transformem estratégias existentes, e até mesmo gerem dados que dependem de outros dados já gerados. Dominar essas técnicas avançadas é crucial para aplicar o Teste Baseado em Propriedades (PBT) de forma eficaz a sistemas de software do mundo real, especialmente aqueles que implementam TADs com estruturas internas não triviais. Esta seção explorará algumas dessas capacidades avançadas de Hypothesis, incluindo `st.builds` para instâncias de classes, `st.recursive` para tipos recursivos, métodos de composição como `map`, `flatmap`, e `filter`, e a estratégia `st.data()` para geração de dados dependentes.

**`st.builds` para Instâncias de Classes Customizadas**

Frequentemente, um TAD é implementado em Python como uma classe. Para testar propriedades sobre instâncias dessa classe, precisamos de uma estratégia Hypothesis que saiba como construir essas instâncias. A estratégia `st.builds` é a ferramenta primária para isso.

`st.builds(target, *args_strategies, **kwargs_strategies)` funciona da seguinte maneira:
*   `target`: É a função ou classe chamável (callable) que você deseja usar para construir o objeto (tipicamente, o construtor da classe, e.g., `MinhaClasse`).
*   `*args_strategies`: Uma sequência de estratégias, uma para cada argumento posicional que `target` espera.
*   `**kwargs_strategies`: Um dicionário onde as chaves são nomes de argumentos nomeados (keyword arguments) que `target` espera, e os valores são as estratégias para gerar esses argumentos.

Hypothesis usará as `args_strategies` e `kwargs_strategies` para gerar os valores dos argumentos e, em seguida, chamará `target(*generated_args, **generated_kwargs)` para criar a instância.

**Exemplo:**
Suponha que temos uma classe `Natural` que implementa nosso TAD `Natural` (baseado em Peano, mas usando um `int` internamente para simplificar a representação em Python, com validação).

```python
# natural_adt.py
class Natural:
    def __init__(self, value: int):
        if not isinstance(value, int) or value < 0:
            raise ValueError("Natural value must be a non-negative integer")
        self._value: int = value

    def get_value(self) -> int:
        return self._value

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, Natural):
            return NotImplemented
        return self._value == other._value
    
    def __repr__(self) -> str:
        return f"Natural({self._value})"

# Teste usando st.builds
# test_natural_properties.py
from hypothesis import strategies as st
# from natural_adt import Natural # Assume Natural is importable

# Estratégia para gerar instâncias de Natural
# Geramos um inteiro não-negativo e passamos para o construtor de Natural
natural_strategy = st.builds(Natural, st.integers(min_value=0))

# @given(natural_strategy)
# def test_some_property_of_natural(n: Natural):
#     assert n.get_value() >= 0
```
No código acima, `natural_strategy = st.builds(Natural, st.integers(min_value=0))` define uma estratégia que primeiro gera um inteiro não-negativo usando `st.integers(min_value=0)`, e então passa esse inteiro para o construtor da classe `Natural` (i.e., `Natural(generated_integer)`). Isso produzirá instâncias válidas de `Natural` que podem ser usadas em testes de propriedade decorados com `@given(natural_strategy)`.

`st.builds` é fundamental para testar TADs implementados como classes, pois permite que Hypothesis crie automaticamente os objetos sobre os quais as propriedades serão verificadas.

**`st.recursive` para Estruturas de Dados Recursivas**

Muitos TADs, como listas, árvores e outras estruturas de dados indutivas, têm uma natureza inerentemente recursiva. Por exemplo, uma `NaturalList` é ou vazia (`nil`) ou é um `Natural` `cons`truído sobre outra `NaturalList`. Gerar instâncias de tais estruturas requer uma estratégia que possa lidar com essa recursão. A estratégia `st.recursive` é projetada para este propósito.

`st.recursive(base, extend, max_leaves=N)` funciona da seguinte forma:
*   `base`: É uma estratégia que gera os casos base não recursivos da estrutura (e.g., para uma lista, a lista vazia; para uma árvore, um nó folha ou uma árvore vazia).
*   `extend`: É uma função que recebe uma estratégia `children_strategy` (que representa a própria estratégia recursiva sendo definida, permitindo gerar subestruturas do mesmo tipo) e deve retornar uma nova estratégia que constrói instâncias maiores/recursivas da estrutura usando os valores gerados por `children_strategy` e possivelmente outras estratégias para os dados.
*   `max_leaves` (opcional, padrão 100): Controla o tamanho máximo ou a profundidade da recursão para evitar a geração de estruturas excessivamente grandes ou infinitas, ajudando na terminação da geração de dados.

**Exemplo (para `NaturalList` baseada em `nil` e `cons`):**
Suponha as classes `EmptyList` e `ConsList` como definidas no exercício da Seção 5.5.
E o tipo `NaturalListType = Union[EmptyList, ConsList]`.
Estratégia para `Natural` (inteiros não-negativos): `st_natural = st.integers(min_value=0)`

```python
from typing import Union # For Python < 3.10, else use |
from hypothesis import strategies as st
# from natural_adt import Natural # Assume Natural class and its strategy exist
# from list_adt_classes import EmptyList, ConsList, NaturalListType # Assume these are defined

# Dummy classes for demonstration if not importing
class NaturalListBase: pass # Common base for type hinting
class EmptyList(NaturalListBase):
    def __init__(self): pass
    def __repr__(self): return "nil"
class ConsList(NaturalListBase):
    def __init__(self, head, tail): self.head, self.tail = head, tail
    def __repr__(self): return f"cons({self.head}, {self.tail})"
NaturalListType = Union[EmptyList, ConsList]
st_natural = st.integers(min_value=0, max_value=10) # Bounded for simpler example


# Estratégia para NaturalList
# Precisamos de st.deferred para quebrar a dependência cíclica na definição
list_strategy_deferred = st.deferred(lambda: natural_list_strategy)

natural_list_strategy = st.recursive(
    base=st.builds(EmptyList),  # Caso base: a lista vazia (nil)
    extend=lambda children: st.builds(ConsList, st_natural, children), # Passo recursivo: cons(Natural, List)
    max_leaves=5  # Limita o tamanho/profundidade das listas geradas
)

# @given(natural_list_strategy)
# def test_list_property(lst: NaturalListType):
#     # ... test some property of lst ...
#     pass
```
Neste exemplo, `natural_list_strategy` é definida usando `st.recursive`.
*   O `base` é `st.builds(EmptyList)`, que gera a lista vazia (`nil`).
*   A função `extend` recebe `children` (que é uma estratégia para gerar `NaturalList`s, ou seja, `natural_list_strategy` em si, gerenciada internamente por `st.recursive`). Ela retorna `st.builds(ConsList, st_natural, children)`. Isso significa: para construir uma lista não vazia, gere um `Natural` (usando `st_natural`) para ser a cabeça, e gere recursivamente outra `NaturalList` (usando `children`, que é a própria `natural_list_strategy`) para ser a cauda, e passe-os para o construtor `ConsList`.
*   `max_leaves=5` limita o número de "folhas" na construção recursiva (aqui, aproximadamente o comprimento da lista) para evitar listas muito longas.
*   `st.deferred` não foi estritamente necessário na estrutura final acima, mas é comum em definições recursivas mais complexas onde a estratégia é referenciada antes de sua definição completa.

`st.recursive` é essencial para testar TADs com definições indutivas, permitindo que Hypothesis gere uma variedade de estruturas, desde as mais simples (casos base) até as mais complexas (múltiplos níveis de recursão).

**Composição e Mapeamento de Estratégias (`map`, `flatmap`, `filter`)**

Hypothesis permite que você refine e transforme estratégias existentes usando métodos como `.map()`, `.flatmap()`, e `.filter()`.

*   **`strategy.map(function)`:**
    Gera um valor a partir de `strategy` e então aplica `function` a esse valor para produzir o resultado final. A `function` deve ser pura (sem efeitos colaterais) e o mapeamento deve ser, idealmente, reversível ou bem comportado para que o encolhimento funcione eficazmente.
    *   Exemplo: Gerar um `Natural` par.
        `st_even_natural = st.integers(min_value=0).map(lambda x: x * 2)`

*   **`strategy.flatmap(function)`:**
    Similar a `map`, mas a `function` que é aplicada ao valor gerado por `strategy` deve ela mesma retornar uma *nova estratégia*. Hypothesis então gera um valor a partir dessa nova estratégia retornada. `flatmap` é útil quando a natureza dos dados que você quer gerar depende de um valor gerado anteriormente.
    *   Exemplo: Gerar uma lista e depois um elemento dessa lista (se a lista não for vazia).
        ```python
        # strategy_list_and_element_from_it = st.lists(st.integers(), min_size=1).flatmap(
        #     lambda lst: st.tuples(st.just(lst), st.sampled_from(lst))
        # )
        # Este exemplo específico é mais simples com st.data() como veremos.
        # Um exemplo melhor para flatmap: gerar um tamanho N, depois uma lista de tamanho N.
        # strategy_sized_list = st.integers(min_value=0, max_value=10).flatmap(
        #    lambda n: st.lists(st.booleans(), min_size=n, max_size=n)
        # )
        ```

*   **`strategy.filter(predicate_function)`:**
    Gera valores a partir de `strategy` e descarta aqueles para os quais `predicate_function` retorna `False`. Só os valores que satisfazem o predicado são produzidos.
    *   **Cuidado:** `filter` deve ser usado com cautela. Se o predicado for muito restritivo (rejeitar a maioria dos dados gerados pela estratégia base), Hypothesis pode ter dificuldade em encontrar dados válidos e o teste pode se tornar muito lento ou falhar ao encontrar dados suficientes. Hypothesis emitirá um aviso de saúde (health check) se isso acontecer.
    *   Exemplo: Gerar um `Natural` ímpar (não a melhor maneira, mas ilustrativa).
        `st_odd_natural = st.integers(min_value=0).filter(lambda x: x % 2 != 0)`
        (Melhor seria `st.integers(min_value=0).map(lambda x: 2 * x + 1)`)

Esses métodos de composição permitem criar estratégias muito específicas e adaptadas às necessidades do código sob teste, a partir de blocos de construção mais simples.

**Geração de Dados Dependentes e Estado (`data()` strategy)**

Às vezes, os dados que você precisa gerar para uma parte de um teste dependem de valores que foram gerados (ou seriam gerados) para outras partes do mesmo teste. A estratégia especial `st.data()` (anteriormente `st.streaming()` ou acessada via o decorador `@composite` com `draw`) fornece uma maneira de lidar com essa **geração de dados dependentes** dentro de uma única execução da função de teste.

Quando uma função de teste é anotada com `@given(st.data())`, ela recebe um objeto especial `data` como argumento. Este objeto `data` tem um método `data.draw(strategy, label=None)` que pode ser chamado dentro da função de teste para "puxar" um valor de uma `strategy` fornecida. O crucial é que `data.draw()` pode ser chamado múltiplas vezes, e as estratégias passadas para chamadas subsequentes de `draw` podem depender dos valores retornados por chamadas anteriores. Hypothesis gerencia a geração e o encolhimento desses dados interdependentes.

**Exemplo: Gerar uma lista e, em seguida, um índice válido para essa lista.**
```python
from hypothesis import given, strategies as st

@given(st.data()) # Passa o objeto 'data' para o teste
def test_list_and_valid_index(data):
    # Primeiro, gere uma lista (não vazia para garantir que um índice exista)
    my_list = data.draw(st.lists(st.integers(), min_size=1, max_size=10), label="generated_list")
    
    # Agora, gere um índice válido para ESTA lista específica
    # Os índices válidos vão de 0 a len(my_list) - 1
    valid_index = data.draw(st.integers(min_value=0, max_value=len(my_list) - 1), label="valid_index")
    
    # Agora podemos usar my_list e valid_index no teste
    assert 0 <= valid_index < len(my_list)
    element = my_list[valid_index]
    # ... outras asserções sobre o elemento ou a lista ...
    assert isinstance(element, int)
```
Neste teste `test_list_and_valid_index`, o objeto `data` é usado.
1.  `data.draw(st.lists(...), label="generated_list")` gera `my_list`.
2.  `data.draw(st.integers(min_value=0, max_value=len(my_list) - 1), label="valid_index")` gera `valid_index`. Note que `max_value` para esta segunda chamada `draw` depende do `len(my_list)` que foi gerado na primeira chamada `draw`.
O argumento `label` é opcional, mas ajuda Hypothesis a fornecer melhores mensagens de erro e a otimizar o encolhimento.

A estratégia `st.data()` (e o mecanismo subjacente `@composite` que permite criar estratégias complexas usando uma função `draw` similar) é extremamente poderosa para cenários onde a estrutura ou os valores dos dados de teste são interdependentes. Isso é comum ao testar operações de TADs que dependem do estado de uma instância ou de relações entre múltiplas instâncias.

Dominar estas estratégias avançadas de geração de dados em Hypothesis permite que o desenvolvedor crie testes de propriedade muito mais expressivos e eficazes, capazes de validar comportamentos complexos e encontrar bugs sutis em implementações de TADs e outras estruturas de código.

**Exercício:**

Suponha que você tem um TAD `NaturalPair` implementado como uma classe Python `class NaturalPair: def __init__(self, n1: int, n2: int): ...` onde `n1` e `n2` devem ser `Natural`s (inteiros $\ge 0$). Além disso, para um par $(n_1, n_2)$ ser "equilibrado", a especificação requer que $n_1 \le n_2$.
Usando Hypothesis e suas estratégias avançadas, esboce uma estratégia `st_balanced_natural_pair` que gere instâncias de `NaturalPair(val1, val2)` onde `val1` e `val2` são `Natural`s (inteiros $\ge 0$) e `val1 <= val2`. (Dica: você pode precisar de `st.data()` ou de compor estratégias com `.flatmap()` ou `.map()` sobre tuplas).

**Resolução:**

Podemos criar a estratégia `st_balanced_natural_pair` de algumas maneiras. Uma abordagem usando `st.data()` ou `@st.composite` é bastante flexível. Alternativamente, podemos usar `st.tuples` e depois `flatmap` ou `map` com `filter` (com cuidado).

**Abordagem 1: Usando `st.data()`**
Esta abordagem é frequentemente a mais clara para dependências.

```python
from hypothesis import given, strategies as st

# Definição da classe NaturalPair para o exemplo
class NaturalPair:
    def __init__(self, n1: int, n2: int):
        if not (isinstance(n1, int) and n1 >= 0 and isinstance(n2, int) and n2 >= 0):
            raise ValueError("Both n1 and n2 must be non-negative integers.")
        if not (n1 <= n2):
            # This assertion might be better handled by the generation strategy
            # or tested as a property of a less constrained generator.
            # For this exercise, the strategy should ensure n1 <= n2.
            pass # Assume constructor doesn't enforce n1 <= n2, strategy does.
        self.n1 = n1
        self.n2 = n2
    def __repr__(self) -> str:
        return f"NaturalPair({self.n1}, {self.n2})"

# Estratégia para gerar um NaturalPair equilibrado
@st.composite
def st_balanced_natural_pair_composite(draw) -> NaturalPair:
    # Primeiro, gere o primeiro Natural, n1
    val1 = draw(st.integers(min_value=0, max_value=100), label="val1") # max_value for practical example
    
    # Em seguida, gere o segundo Natural, n2, tal que n2 >= val1
    # min_value para n2 é o val1 que acabamos de gerar
    val2 = draw(st.integers(min_value=val1, max_value=100), label="val2") # max_value for practical example
    
    # Construa e retorne a instância de NaturalPair
    return NaturalPair(val1, val2)

# Se preferir st.data() diretamente no teste:
# @given(st.data())
# def test_with_balanced_pair(data):
#     val1 = data.draw(st.integers(min_value=0, max_value=100))
#     val2 = data.draw(st.integers(min_value=val1, max_value=100))
#     pair = NaturalPair(val1, val2)
#     # ... seu teste ...
#     assert pair.n1 <= pair.n2
```

No código acima, a função `st_balanced_natural_pair_composite` é decorada com `@st.composite`. Esta função recebe um argumento `draw` que funciona como `data.draw` mencionado anteriormente.
1.  `val1 = draw(st.integers(min_value=0, max_value=100))` gera o primeiro número natural `val1` (limitei a 100 para exemplos práticos, mas poderia ser ilimitado ou ter outro `max_value`).
2.  `val2 = draw(st.integers(min_value=val1, max_value=100))` gera o segundo número natural `val2`, garantindo que `val2` seja maior ou igual a `val1` (ao definir `min_value=val1`).
3.  Finalmente, `NaturalPair(val1, val2)` é construído e retornado. Esta estratégia agora pode ser usada em `@given(st_balanced_natural_pair_composite())`.

**Abordagem 2: Usando `st.tuples` e `flatmap` (mais complexo de encadear para `builds`) ou `map` com `filter`**

Usar `flatmap` para `st.builds` diretamente pode ser um pouco mais verboso, mas o princípio é gerar `val1` primeiro, e então, baseado em `val1`, gerar uma estratégia para `val2`, e então construir.
Uma forma mais simples, mas que requer cuidado com `filter`, seria:

```python
from hypothesis import strategies as st
# from your_module import NaturalPair # Assuming NaturalPair is defined

# Estratégia para gerar tuplas de dois naturais
# (Poderia usar max_values menores para testes mais rápidos se apropriado)
two_naturals_strategy = st.tuples(st.integers(min_value=0), st.integers(min_value=0))

# Filtrar para manter apenas tuplas (v1, v2) onde v1 <= v2
# E então mapear para construir NaturalPair
st_balanced_natural_pair_map_filter = two_naturals_strategy.filter(
    lambda pair_tuple: pair_tuple[0] <= pair_tuple[1]
).map(
    lambda pair_tuple: NaturalPair(pair_tuple[0], pair_tuple[1])
)

# Teste de exemplo
# @given(st_balanced_natural_pair_map_filter)
# def test_property_with_balanced_pair(pair: NaturalPair):
#     assert pair.n1 <= pair.n2
```
Nesta segunda abordagem:
1.  `two_naturals_strategy` gera tuplas de dois inteiros não-negativos $(v_1, v_2)$ sem restrição de ordem entre eles.
2.  `.filter(lambda pair_tuple: pair_tuple[0] <= pair_tuple[1])` descarta todas as tuplas geradas onde $v_1 > v_2$. **Cuidado:** Este filtro pode ser ineficiente se a estratégia base gerar muitos pares $v_1 > v_2$ (o que acontece aproximadamente 50% do tempo se $v_1, v_2$ são de distribuições similares). Hypothesis pode reclamar sobre a taxa de rejeição.
3.  `.map(lambda pair_tuple: NaturalPair(pair_tuple[0], pair_tuple[1]))` então toma as tuplas filtradas (onde $v_1 \le v_2$) e as usa para construir instâncias de `NaturalPair`.

**Qual abordagem é melhor?**
A abordagem com `@st.composite` (ou `st.data()` diretamente no teste) é geralmente preferida para dados dependentes porque é mais explícita sobre como os dados são construídos passo a passo e tende a ser mais eficiente na geração, pois não depende de rejeitar muitos exemplos (como `filter` pode fazer). O encolhimento (shrinking) também tende a funcionar melhor com `@st.composite` e `st.data()` porque Hypothesis tem mais informações sobre a estrutura da geração.

Para este problema, `@st.composite` é a solução mais robusta e idiomática em Hypothesis.

# 6.4 O Processo de "Shrinking" e a Minimização de Contraexemplos

Um dos recursos mais poderosos e distintivos das bibliotecas de Teste Baseado em Propriedades (PBT) avançadas, como Hypothesis, é o processo de **encolhimento** (shrinking) de contraexemplos. Quando Hypothesis executa um teste de propriedade e encontra um conjunto de entradas que faz com que uma asserção falhe (um contraexemplo), ela não para por aí. Em vez de simplesmente reportar o contraexemplo original (que pode ser grande, complexo e, portanto, difícil de depurar), Hypothesis inicia um processo iterativo para tentar encontrar um contraexemplo **minimalista** que ainda exiba a mesma falha. Este contraexemplo "encolhido" é geralmente muito menor e mais simples, o que torna significativamente mais fácil para o desenvolvedor entender a causa raiz do bug e corrigi-lo. O processo de shrinking é uma parte integral do que torna o PBT com Hypothesis tão eficaz na prática.

**Como Funciona o Shrinking (Conceitualmente):**

1.  **Detecção da Falha:** Hypothesis gera um conjunto de dados de entrada $D_{original}$ usando as estratégias fornecidas. O teste de propriedade é executado com $D_{original}$. Se uma asserção falha (ou uma exceção inesperada é levantada), $D_{original}$ é marcado como um contraexemplo.

2.  **Início do Processo de Encolhimento:** Uma vez que um $D_{original}$ falho é encontrado, Hypothesis tenta "encolhê-lo". "Encolher" significa tentar variações de $D_{original}$ que são consideradas "menores" ou "mais simples" de acordo com a natureza dos dados e das estratégias usadas para gerá-los.
    *   Para **inteiros**, "menor" pode significar mais próximo de zero, ou um valor absoluto menor.
    *   Para **listas ou strings**, "menor" pode significar mais curta, ou com elementos/caracteres que são individualmente "menores" (e.g., uma lista `[0,0]` é menor que `[10,20]`).
    *   Para **objetos complexos** construídos com `st.builds` ou `st.recursive`, Hypothesis tenta encolher cada um dos componentes usados para construir o objeto.

3.  **Tentativa e Verificação:** Para cada variação "encolhida" $D_{shrunk}$ de um contraexemplo conhecido $D_{current\_failure}$ (inicialmente $D_{original}$):
    *   Hypothesis reexecuta o teste de propriedade com $D_{shrunk}$.
    *   **Se o teste ainda falha com $D_{shrunk}$**: Isso é ótimo! $D_{shrunk}$ é um contraexemplo menor que ainda demonstra o bug. $D_{shrunk}$ agora se torna o $D_{current\_failure}$, e Hypothesis tenta encolhê-lo ainda mais.
    *   **Se o teste passa com $D_{shrunk}$**: Esta tentativa de encolhimento foi "longe demais" – $D_{shrunk}$ não exibe mais o bug. Hypothesis descarta $D_{shrunk}$ e tenta outra forma de encolher $D_{current\_failure}$.

4.  **Iteração até um Mínimo Local:** O processo de tentar encolher e verificar continua iterativamente. Hypothesis explora diferentes caminhos de encolhimento. O objetivo é encontrar um contraexemplo $D_{minimal}$ tal que nenhuma variação "menor" de $D_{minimal}$ (que Hypothesis consegue encontrar) ainda cause a falha. Este $D_{minimal}$ é um "mínimo local" no espaço de busca do encolhimento.

5.  **Reporte do Contraexemplo Encolhido:** Finalmente, Hypothesis reporta o $D_{minimal}$ encontrado como o contraexemplo para o desenvolvedor.

**Exemplo Ilustrativo:**
Suponha que temos uma propriedade para uma função `process_list(data: list[int])` que falha para a lista `data = [10, 20, 0, 30, 40]`.
Hypothesis pode tentar encolher `data` da seguinte forma (esta é uma simplificação, o processo real é mais sofisticado):
*   Tentar listas mais curtas:
    *   `[10, 20, 0, 30]` (remove o último) - Se falhar, continua com esta.
    *   `[20, 0, 30, 40]` (remove o primeiro) - Se falhar...
    *   ...
*   Tentar elementos menores (se a lista `[10, 0, 30]` ainda falhar):
    *   `[5, 0, 30]` (encolhe 10 para 5) - Se falhar...
    *   `[0, 0, 30]` (encolhe 10 para 0) - Se falhar...
    *   `[10, 0, 15]` (encolhe 30 para 15) - Se falhar...
*   Tentar remover elementos do meio.
*   Tentar substituir elementos por "valores canônicos" (e.g., 0, 1).

Se, por exemplo, descobre-se que a lista `[0, 30]` ainda causa a falha, e nenhuma simplificação adicional de `[0, 30]` (como `[0]`, `[30]`, `[]`, `[0,0]`, `[0,15]`) causa a falha, então `[0, 30]` seria reportado.

**Como Hypothesis Sabe Como "Encolher"?**
*   **Estratégias Embutidas:** As estratégias padrão de Hypothesis (para inteiros, listas, etc.) vêm com lógica de encolhimento embutida e bem testada. Elas "sabem" quais são os valores "menores" ou "mais simples" para seus respectivos tipos.
*   **`st.builds` e `@st.composite`:** Quando você usa `st.builds` para construir objetos de suas classes, Hypothesis tenta encolher os argumentos que foram passados para o construtor. Se o construtor foi chamado com `MyClass(arg1_val, arg2_val)`, e isso causou uma falha, Hypothesis tentará chamar `MyClass` com versões encolhidas de `arg1_val` e `arg2_val`.
*   **`st.recursive`:** Para estruturas recursivas, Hypothesis tenta encolher tanto os dados nos nós quanto a própria estrutura da recursão (e.g., transformando um nó interno em um caso base, ou reduzindo o número de filhos).
*   **`map` e `filter`:** O encolhimento pode se tornar menos eficaz ou mais complexo com `map` e `filter`. Hypothesis tenta reverter o `map` durante o encolhimento, se possível. Para `filter`, ela tenta encontrar valores menores que ainda passem pelo filtro e causem a falha. Se o `filter` for muito agressivo, o encolhimento pode ser difícil.
*   **`st.data()` e `draw()`:** Hypothesis rastreia as escolhas feitas pelo método `draw()` e tenta encolher cada uma delas na ordem inversa ou de forma interdependente. Os `label`s fornecidos a `draw()` podem ajudar nesse processo.

**Benefícios do Shrinking:**
1.  **Depuração Facilitada:** Contraexemplos menores são muito mais fáceis de entender. Em vez de analisar um objeto complexo ou uma lista longa, o desenvolvedor pode se concentrar em um caso mínimo que isola o bug.
2.  **Melhor Compreensão da Falha:** O contraexemplo encolhido frequentemente revela a condição exata ou o padrão que causa o problema.
3.  **Casos de Teste de Regressão Mais Concisos:** O contraexemplo encolhido pode ser diretamente adicionado como um teste de regressão baseado em exemplo, sendo mais focado e mais fácil de manter.

O processo de encolhimento não é garantido de encontrar o contraexemplo *globalmente* mínimo (isso seria computacionalmente intratável), mas ele é muito eficaz em encontrar um contraexemplo significativamente mais simples do que o original que disparou a falha. É uma das razões pelas quais PBT com ferramentas como Hypothesis é tão produtivo para encontrar e corrigir bugs.

**Exercício:**

Suponha que você está testando uma função `process_naturals(n1: Natural, n2: Natural)` com a propriedade de que o resultado deve ser sempre um `Natural` (inteiro $\ge 0$). A estratégia para `n1` e `n2` gera inteiros entre 0 e 100. Hypothesis encontra um contraexemplo onde `n1=90` e `n2=60` causam uma falha (e.g., a função retorna -1). Descreva, em termos gerais, alguns dos passos ou tipos de valores que Hypothesis poderia tentar durante o processo de "shrinking" para `n1` e `n2` para encontrar um contraexemplo menor.

**Resolução:**

Se Hypothesis encontrou a falha para `n1=90` e `n2=60` na função `process_naturals(n1, n2)` (que deveria retornar um `Natural` $\ge 0$ mas retornou -1), o processo de shrinking tentaria encontrar valores menores para `n1` e `n2` que ainda causassem a falha.

Hypothesis tentaria reduzir os valores de `n1` e `n2` (que são gerados por uma estratégia de inteiros, provavelmente `st.integers(min_value=0, max_value=100)`), visando valores "mais simples", que para inteiros geralmente significa valores mais próximos de zero ou com menor magnitude. O encolhimento ocorre em ambos os argumentos, e Hypothesis tenta várias combinações.

**Alguns passos e tipos de valores que Hypothesis poderia tentar para `(n1, n2)` a partir de `(90, 60)`:**

1.  **Reduzir um argumento de cada vez, mantendo o outro:**
    *   **Encolhendo `n1` (mantendo `n2=60`):**
        *   Tentar valores menores para `n1`: `(89, 60)`, `(45, 60)` (metade), `(0, 60)`, `(1, 60)`.
        *   Se `(0, 60)` ainda falhar, este é um bom candidato para ser o novo `n1`.
    *   **Encolhendo `n2` (mantendo `n1` no valor atual que causa falha, e.g., `n1=90` ou um `n1` já encolhido):**
        *   Se `n1` foi encolhido para `0` e `(0, 60)` falha, tentar encolher `n2`: `(0, 59)`, `(0, 30)`, `(0, 0)`, `(0, 1)`.

2.  **Reduzir ambos os argumentos simultaneamente (ou em alternância):**
    Hypothesis não necessariamente encolhe um argumento até o fim antes de tocar no outro. Ela explora o espaço de forma mais global.
    *   Poderia tentar pares como `(45, 30)` (ambos pela metade).
    *   Poderia tentar valores canônicos ou de borda para um ou ambos:
        *   `(0, 0)`, `(1, 0)`, `(0, 1)`, `(1, 1)`.
        *   `(90, 0)` (se `n1` está fixo por um momento).
        *   `(0, 60)` (se `n2` está fixo por um momento).

3.  **Estratégia de Encolhimento para Inteiros:**
    Para um inteiro `x`, Hypothesis tenta encolhê-lo para:
    *   Valores mais próximos de 0 (e.g., se $x > 0$, tenta $0, 1, x/2, x-1$).
    *   Se $x < 0$ (não aplicável aqui pois são `Natural`s $\ge 0$), tentaria valores mais próximos de 0.
    *   Ela explora a representação binária e tenta zerar bits.

**Processo Iterativo:**
Suponha que `(90, 60)` falha.
*   Hypothesis testa `(0, 60)`. Se falhar, `(0, 60)` se torna o novo "melhor" contraexemplo.
    *   Agora, a partir de `(0, 60)`, tenta encolher mais.
    *   Testa `(0, 0)`. Se `(0,0)` falhar, este é ainda melhor.
        *   A partir de `(0,0)`, tenta encolher (mas 0 já é o mínimo para `Natural`).
        *   Então, `(0,0)` seria provavelmente o contraexemplo final reportado.
*   Se `(0, 60)` passou (não falhou), Hypothesis volta para `(90, 60)` e tenta outro caminho de encolhimento, e.g., `(90, 0)`.
    *   Se `(90, 0)` falhar, torna-se o novo "melhor".
        *   A partir de `(90,0)`, tenta encolher `n1`. Testa `(0,0)`. Se falhar, `(0,0)` é o novo melhor.
*   Se `process_naturals(n1,n2)` falha sempre que $n1 > n2$ (hipotético), o encolhimento poderia levar a algo como `(1,0)`.

O objetivo é encontrar o par `(n1_min, n2_min)` "lexicograficamente menor" ou "estruturalmente mais simples" que ainda causa a falha. Se a falha ocorre, por exemplo, sempre que $n_1 = 0$ e $n_2 > 50$, Hypothesis tentaria encontrar o menor $n_2 > 50$ que causa a falha, possivelmente reportando `(0, 51)`. Se a falha ocorre para qualquer $n_1=0, n_2=0$, então `(0,0)` seria o resultado ideal do encolhimento.

O processo exato de encolhimento é complexo e heurístico, mas a ideia geral é explorar sistematicamente variações "mais simples" do contraexemplo encontrado, verificando se a falha persiste, até que não se consiga mais simplificar o contraexemplo que ainda falha.

# 6.5 Integrando Hypothesis com Frameworks de Teste (e.g., `pytest`)

Embora Hypothesis possa ser usado de forma independente para executar testes de propriedade, sua integração com frameworks de teste estabelecidos, como `pytest` ou `unittest` (o framework embutido de Python), é a maneira mais comum e conveniente de incorporar o Teste Baseado em Propriedades (PBT) em um fluxo de trabalho de desenvolvimento e em suítes de teste existentes. `pytest`, em particular, possui uma excelente integração com Hypothesis, permitindo que testes de propriedade sejam descobertos, executados e reportados da mesma forma que testes unitários convencionais, ao mesmo tempo em que se beneficia dos recursos avançados de ambos os sistemas. Esta seção focará na integração de Hypothesis com `pytest`, que é uma combinação popular e poderosa no ecossistema Python.

**Benefícios da Integração:**

1.  **Descoberta Automática de Testes:** `pytest` pode descobrir automaticamente funções de teste decoradas com `@given` da Hypothesis, assim como descobre funções de teste nomeadas `test_*` ou métodos `test_*` em classes `Test*`. Não é necessário nenhum boilerplate extra para registrar os testes de propriedade.

2.  **Execução Unificada:** Todos os testes (baseados em exemplos e baseados em propriedades) podem ser executados com um único comando (e.g., `pytest`).

3.  **Relatórios Consistentes:** Os resultados dos testes de Hypothesis (passagens, falhas, contraexemplos) são integrados aos relatórios de `pytest`, facilitando a visualização do status geral da suíte de testes. Quando um teste de Hypothesis falha, `pytest` reportará o contraexemplo encolhido fornecido por Hypothesis.

4.  **Recursos de `pytest` Disponíveis:** Testes de Hypothesis podem usar outros recursos de `pytest`, como fixtures (para setup/teardown), marks (para categorizar ou pular testes), e plugins.

5.  **Configuração de Hypothesis via `pytest`:** Muitas configurações de Hypothesis (como o número de exemplos a serem executados, perfis de execução, estado do banco de dados de exemplos) podem ser gerenciadas através de opções de linha de comando de `pytest` ou de arquivos de configuração de `pytest` (como `pytest.ini` ou `pyproject.toml`).

**Como Funciona a Integração com `pytest`:**

Para usar Hypothesis com `pytest`, geralmente você só precisa:
1.  Instalar ambos os pacotes: `pip install pytest hypothesis`.
2.  Escrever suas funções de teste em arquivos Python (e.g., `test_meu_modulo.py`).
3.  Decorar as funções de teste de propriedade com `@given(...)` da Hypothesis, fornecendo as estratégias apropriadas.
4.  Usar asserções Python padrão (`assert`) dentro das funções de teste para verificar as propriedades.
5.  Executar `pytest` a partir da linha de comando no diretório do seu projeto.

**Exemplo de um Arquivo de Teste para `pytest` com Hypothesis:**
```python
# test_arithmetic_properties.py
from hypothesis import given, strategies as st, settings, HealthCheck

# Funções (ou métodos de uma classe) a serem testadas
def add(x: int, y: int) -> int:
    return x + y

def buggy_sort_list(numbers: list[int]) -> list[int]:
    # Implementação intencionalmente com bug para listas com duplicatas
    if len(numbers) > 1 and numbers[0] == numbers[1]:
        return numbers[1:] # Errado, remove um elemento
    return sorted(numbers)


# Propriedade 1: Comutatividade da adição
@given(st.integers(), st.integers())
def test_addition_is_commutative(x: int, y: int) -> None:
    assert add(x, y) == add(y, x)

# Propriedade 2: Adicionar zero é identidade
@given(st.integers())
def test_add_zero_is_identity(x: int) -> None:
    assert add(x, 0) == x
    assert add(0, x) == x

# Propriedade 3: Ordenar uma lista não deve diminuir seu tamanho
# Desabilitar a verificação de saúde para exemplos lentos se necessário,
# mas é melhor otimizar a geração de dados se possível.
# Aqui, vamos ilustrar uma falha com a função buggy_sort_list.
@settings(suppress_health_check=[HealthCheck.too_slow]) # Exemplo de configuração
@given(st.lists(st.integers(min_value=0, max_value=10)))
def test_sorting_preserves_length_or_bug_found(numbers: list[int]) -> None:
    original_length = len(numbers)
    sorted_list = buggy_sort_list(numbers)
    # A propriedade correta seria: assert len(sorted_list) == original_length
    # Mas para demonstrar a falha com buggy_sort_list:
    if len(numbers) > 1 and numbers[0] == numbers[1]:
        # Esperamos que o bug reduza o tamanho
        assert len(sorted_list) == original_length - 1
    else:
        assert len(sorted_list) == original_length

# Para executar:
# No terminal, no diretório contendo este arquivo (e após instalar pytest e hypothesis):
# $ pytest
```
Neste arquivo `test_arithmetic_properties.py`:
*   Definimos funções simples `add` e uma `buggy_sort_list` para serem testadas.
*   `test_addition_is_commutative` e `test_add_zero_is_identity` são testes de propriedade para `add`. `pytest` as descobrirá e Hypothesis gerará os inteiros `x` e `y`.
*   `test_sorting_preserves_length_or_bug_found` testa uma propriedade (modificada para pegar o bug) da função `buggy_sort_list`. Ele usa o decorador `@settings` da Hypothesis para suprimir um aviso de saúde (health check) que poderia ocorrer se a geração de dados para uma propriedade fosse muito lenta (neste caso, é apenas ilustrativo).
*   Quando `pytest` é executado, ele encontrará essas três funções `test_*`. Para as funções decoradas com `@given`, `pytest` delegará a execução para Hypothesis, que rodará cada propriedade múltiplas vezes com dados gerados.
*   Se `buggy_sort_list` for chamada com uma lista como `[5, 5, 1]`, ela retornará `[5, 1]`. O teste `test_sorting_preserves_length_or_bug_found` tem uma lógica que espera essa falha específica. Se a propriedade fosse a correta `assert len(sorted_list) == original_length`, Hypothesis encontraria e reportaria o contraexemplo (e.g., `[5,5,1]`) que causa a falha.

**Gerenciando o Estado de Hypothesis (Banco de Dados de Exemplos):**
Hypothesis mantém um pequeno banco de dados local (no diretório `.hypothesis/`) de exemplos que causaram falhas anteriormente ou que foram particularmente interessantes. Ao reexecutar os testes, Hypothesis pode priorizar esses exemplos para tentar encontrar regressões rapidamente.
*   Este banco de dados pode ser gerenciado (e.g., limpo) se necessário.
*   A localização pode ser configurada.

**Configurações Comuns de Hypothesis (via `settings` ou `pytest`):**
*   `max_examples`: O número de exemplos a serem gerados para cada propriedade (padrão 100). Pode ser aumentado para testes mais exaustivos (e.g., `@settings(max_examples=500)`).
*   `deadline`: Tempo máximo que uma única execução de uma propriedade pode levar. Se excedido, Hypothesis assume que pode haver um loop infinito ou desempenho muito ruim com certos dados e reporta um erro.
*   `phases`: Controla as diferentes fases da geração de dados (e.g., explícita, reuso, geração, encolhimento).
*   `suppress_health_check`: Permite desabilitar certos avisos de saúde que Hypothesis emite se suspeitar que as estratégias ou os testes podem não ser ideais (e.g., filtragem excessiva, geração lenta).

Essas configurações podem ser aplicadas globalmente (via `pytest.ini` ou linha de comando para `pytest`) ou por teste individual usando o decorador `@settings(...)` da Hypothesis.

**Exemplo de configuração em `pytest.ini`:**
```ini
[pytest]
hypothesis_max_examples = 200
hypothesis_deadline = 1000 ; em milissegundos
```

A integração de Hypothesis com `pytest` é transparente e poderosa, tornando o Teste Baseado em Propriedades uma adição natural e de baixo atrito a qualquer projeto Python que já utilize `pytest` para seus testes. Isso facilita a adoção de PBT e ajuda a melhorar significativamente a qualidade e a robustez do software.

**Exercício:**

Suponha que você tenha uma especificação para o TAD `Natural` com uma operação `is_odd: Natural -> Boolean` (que verifica se um número natural é ímpar). Você implementou isso em Python como `py_is_odd(n: int) -> bool`. Um axioma (ou propriedade derivada) poderia ser que `is_odd(n)` é o oposto de `is_odd(succ(n))` (ou seja, `is_odd(n) != is_odd(n+1)`). Escreva uma função de teste `pytest` chamada `test_is_odd_flips_with_successor` que use Hypothesis para verificar esta propriedade para `py_is_odd`.

**Resolução:**

Aqui está um esboço de como seria a função de teste `pytest` usando Hypothesis para verificar a propriedade `is_odd(n) != is_odd(succ(n))`.

```python
# test_natural_odd_properties.py
from hypothesis import given, strategies as st

# Suponha que esta é a implementação a ser testada
# (pode estar em outro módulo e ser importada)
def py_is_odd(n: int) -> bool:
    """
    Implementation of is_odd for Natural (non-negative integers).
    Returns True if n is odd, False otherwise.
    Raises ValueError if n is negative.
    """
    if n < 0:
        raise ValueError("Input must be a non-negative integer.")
    return n % 2 != 0

def py_succ(n: int) -> int:
    """
    Implementation of succ for Natural.
    Raises ValueError if n is negative (as succ is for Naturals).
    """
    if n < 0:
        raise ValueError("Input to succ must be a non-negative integer.")
    return n + 1

# Teste de propriedade para pytest e Hypothesis
@given(st.integers(min_value=0, max_value=1000)) # Estratégia para gerar 'n' (Natural)
                                                # max_value é opcional, mas bom para limitar tempo de teste
def test_is_odd_flips_with_successor(n: int) -> None:
    """
    Verifica a propriedade: is_odd(n) != is_odd(succ(n))
    Para a implementação Python: py_is_odd(n) != py_is_odd(py_succ(n))
    """
    # Calcula is_odd para n
    is_odd_n = py_is_odd(n)
    
    # Calcula o sucessor de n
    succ_n = py_succ(n)
    
    # Calcula is_odd para o sucessor de n
    is_odd_succ_n = py_is_odd(succ_n)
    
    # Verifica a propriedade: o valor de is_odd deve ser diferente
    assert is_odd_n != is_odd_succ_n

# Para executar este teste:
# 1. Salve o código como, por exemplo, test_natural_odd_properties.py
# 2. Certifique-se de ter pytest e hypothesis instalados: pip install pytest hypothesis
# 3. No terminal, navegue até o diretório do arquivo e execute: pytest
```

**Explicação do Código de Teste:**

1.  **Importações:** Importamos `given` e `strategies as st` de `hypothesis`.
2.  **Funções a Serem Testadas:** Incluímos as implementações de `py_is_odd` e `py_succ` (que poderiam estar em um módulo separado e serem importadas). `py_is_odd` verifica se um inteiro não-negativo é ímpar. `py_succ` calcula o sucessor. Ambas incluem verificações para garantir que operam sobre `Natural`s (inteiros não-negativos).
3.  **Decorador `@given`:**
    *   `@given(st.integers(min_value=0, max_value=1000))` instrui Hypothesis a chamar a função `test_is_odd_flips_with_successor` múltiplas vezes.
    *   Em cada chamada, o argumento `n` será um inteiro gerado pela estratégia `st.integers(min_value=0, max_value=1000)`, que produz inteiros não-negativos (representando `Natural`s) até 1000. O `max_value` ajuda a manter os testes rápidos, mas pode ser ajustado ou removido para uma gama maior.
4.  **Lógica do Teste (`test_is_odd_flips_with_successor`):**
    *   `is_odd_n = py_is_odd(n)`: Calcula se `n` é ímpar.
    *   `succ_n = py_succ(n)`: Calcula o sucessor de `n`.
    *   `is_odd_succ_n = py_is_odd(succ_n)`: Calcula se o sucessor de `n` é ímpar.
    *   `assert is_odd_n != is_odd_succ_n`: Esta é a asserção principal que verifica a propriedade. Se um número `n` é ímpar, seu sucessor `n+1` deve ser par, e vice-versa. Portanto, seus valores de `is_odd` devem ser diferentes (um `True`, o outro `False`).

Quando `pytest` executar este arquivo, ele descobrirá `test_is_odd_flips_with_successor` e a executará sob o controle de Hypothesis. Hypothesis gerará muitos valores para `n` e verificará se a asserção se mantém. Se encontrar um `n` para o qual `py_is_odd(n)` seja igual a `py_is_odd(py_succ(n))` (indicando um bug na implementação de `py_is_odd` ou `py_succ`), Hypothesis reportará uma falha com esse `n` como contraexemplo (após encolhimento).

# 6.6 Exemplo Prático: Testando os Axiomas da Especificação `Natural` Implementada em Python

Para consolidar os conceitos de Teste Baseado em Propriedades (PBT) com Hypothesis e sua aplicação na validação de implementações de Tipos Abstratos de Dados (TADs) especificados algebricamente, esta seção apresentará um exemplo prático completo. Vamos considerar o TAD `Natural` (números naturais de Peano), cuja especificação algébrica foi discutida no Capítulo 2. Primeiramente, desenvolveremos uma implementação Python simples para este TAD, utilizando anotações de tipo MyPy para manter o rigor na interface. Em seguida, definiremos estratégias Hypothesis para gerar instâncias da nossa classe `Natural` em Python. Finalmente, escreveremos e executaremos testes de propriedade com Hypothesis (integrados com `pytest`) que correspondem diretamente aos axiomas da especificação algébrica de `Natural`, focando particularmente nos axiomas das operações de adição (`add`) e multiplicação (`mult`). Este exemplo ilustrará o fluxo de trabalho desde a especificação formal, passando pela implementação tipada, até a verificação robusta de comportamento usando PBT.

**Implementação Python da Classe `Natural` (com tipagem MyPy)**

Vamos implementar uma classe `PyNatural` em Python que representa os números naturais. Para simplificar, usaremos um inteiro Python internamente para armazenar o valor, mas garantiremos que ele seja sempre não-negativo através do construtor e das operações. As operações fundamentais serão `zero` (uma forma de obter 0), `succ` (sucessor), e as operações derivadas `add` e `mult`. Também incluiremos `is_zero` e uma representação de igualdade (`__eq__`).

```python
# py_natural.py
from __future__ import annotations # Para type hints de PyNatural dentro da própria classe
from typing import Any

class PyNatural:
    _value: int

    def __init__(self, value: int) -> None:
        if not isinstance(value, int) or value < 0:
            raise ValueError("PyNatural value must be a non-negative integer.")
        self._value = value

    @staticmethod
    def zero() -> PyNatural:
        return PyNatural(0)

    def succ(self) -> PyNatural:
        return PyNatural(self._value + 1)

    def is_zero(self) -> bool:
        return self._value == 0

    def add(self, other: PyNatural) -> PyNatural:
        if not isinstance(other, PyNatural):
            return NotImplemented # Ou raise TypeError
        # Implementação recursiva baseada nos axiomas de Peano
        # add(n, zero) = n
        # add(n, succ(m)) = succ(add(n, m))
        # Para fins de teste, podemos usar a + interna do Python,
        # mas uma implementação fiel aos axiomas seria recursiva ou iterativa.
        # Aqui, vamos usar a + do Python para eficiência na demo, mas
        # o teste de propriedade verificará a conformidade com os axiomas.
        return PyNatural(self._value + other._value)

    def mult(self, other: PyNatural) -> PyNatural:
        if not isinstance(other, PyNatural):
            return NotImplemented
        # mult(n, zero) = zero
        # mult(n, succ(m)) = add(mult(n, m), n)
        return PyNatural(self._value * other._value)

    def __eq__(self, other: Any) -> bool:
        if not isinstance(other, PyNatural):
            return NotImplemented
        return self._value == other._value

    def __hash__(self) -> int:
        # Necessário se PyNatural for usado como chave de dicionário ou em conjuntos
        return hash(self._value)
        
    def __repr__(self) -> str:
        return f"PyNatural({self._value})"

    # Para ajudar nos testes, podemos adicionar um método para obter o valor int
    def to_int(self) -> int:
        return self._value
```

O código acima define a classe `PyNatural` em `py_natural.py`. O construtor `__init__` garante que o valor interno `_value` seja um inteiro não-negativo. O método estático `zero()` retorna uma instância de `PyNatural(0)`. O método `succ()` retorna um novo `PyNatural` com o valor incrementado. `is_zero()` verifica se o valor é zero. Os métodos `add()` e `mult()` implementam a adição e multiplicação; para este exemplo, eles usam diretamente as operações `+` e `*` de inteiros de Python para simplicidade e eficiência, mas nosso objetivo com PBT será verificar se essas implementações ainda satisfazem os axiomas de Peano (que são recursivos). O método `__eq__` define a igualdade entre instâncias de `PyNatural` com base em seus valores internos, e `__hash__` é incluído para boa medida. `__repr__` fornece uma representação textual útil, e `to_int()` é um auxiliar. As anotações de tipo MyPy são usadas em toda a classe.

**Definição de Estratégias Hypothesis para `Natural`**

Agora, precisamos de uma estratégia Hypothesis que possa gerar instâncias da nossa classe `PyNatural`. Usaremos `st.builds` combinado com uma estratégia para gerar inteiros não-negativos.

```python
# test_py_natural.py
from hypothesis import strategies as st
from py_natural import PyNatural # Assumindo que py_natural.py está no mesmo diretório ou PYTHONPATH

# Estratégia para gerar inteiros não-negativos (representando valores Naturais)
# Limitamos o max_value para evitar números excessivamente grandes nos testes,
# o que pode tornar os testes lentos ou causar estouro se a implementação
# de add/mult fosse puramente recursiva sem otimização de cauda.
# Para testes mais robustos de lógica, max_value pode ser maior.
# Para este exemplo, 100 é um limite razoável para demonstração.
st_natural_value = st.integers(min_value=0, max_value=100)

# Estratégia para gerar instâncias da classe PyNatural
# Ela usa st_natural_value para gerar o argumento 'value' para o construtor PyNatural.
st_py_natural = st.builds(PyNatural, value=st_natural_value)

# Alternativamente, se o construtor de PyNatural só tivesse um argumento posicional:
# st_py_natural_alt = st.builds(PyNatural, st.integers(min_value=0, max_value=100))
# Mas nomear 'value=...' é mais explícito se o construtor aceitar kwargs ou tiver múltiplos args.
```
No arquivo de teste `test_py_natural.py`, a estratégia `st_natural_value` gera inteiros entre 0 e 100. A estratégia `st_py_natural` usa `st.builds` para chamar o construtor `PyNatural(value=...)` com valores gerados por `st_natural_value`. Com isso, Hypothesis pode agora criar objetos `PyNatural` para nossos testes.

**Escrevendo e Executando Testes de Propriedade para os Axiomas de `add` e `mult`**

Vamos escrever testes de propriedade para os axiomas de `add` e `mult` do TAD `Natural`.
Axiomas para `add`:
(N3): `add(n, zero) = n`
(N4): `add(n, succ(m)) = succ(add(n, m))`

Axiomas para `mult`:
(N5): `mult(n, zero) = zero`
(N6): `mult(n, succ(m)) = add(mult(n, m), n)`

```python
# test_py_natural.py (continuação)
from hypothesis import given, settings
# import pytest # Não estritamente necessário para rodar com 'pytest' se não usar fixtures etc.

# --- Testes para a operação add ---

@given(n=st_py_natural)
def test_add_identity_with_zero(n: PyNatural) -> None:
    # Axioma (N3): add(n, zero) = n
    # Em Python: n.add(PyNatural.zero()) == n
    assert n.add(PyNatural.zero()) == n

@given(n=st_py_natural, m=st_py_natural)
@settings(max_examples=50) # Reduzir exemplos para mult pode ser útil se for lento
def test_add_with_successor(n: PyNatural, m: PyNatural) -> None:
    # Axioma (N4): add(n, succ(m)) = succ(add(n, m))
    # Em Python: n.add(m.succ()) == (n.add(m)).succ()
    lhs = n.add(m.succ())
    rhs = (n.add(m)).succ()
    assert lhs == rhs

# --- Testes para a operação mult ---

@given(n=st_py_natural)
def test_mult_with_zero_is_zero(n: PyNatural) -> None:
    # Axioma (N5): mult(n, zero) = zero
    # Em Python: n.mult(PyNatural.zero()) == PyNatural.zero()
    assert n.mult(PyNatural.zero()) == PyNatural.zero()

@given(n=st_py_natural, m=st_py_natural)
@settings(max_examples=50) # Reduzir exemplos pode ser útil devido à recursão implícita
def test_mult_with_successor(n: PyNatural, m: PyNatural) -> None:
    # Axioma (N6): mult(n, succ(m)) = add(mult(n, m), n)
    # Em Python: n.mult(m.succ()) == (n.mult(m)).add(n)
    lhs = n.mult(m.succ())
    rhs = (n.mult(m)).add(n)
    assert lhs == rhs

# Outras propriedades interessantes (teoremas deriváveis) que poderiam ser testadas:
# Comutatividade da Adição: add(n,m) = add(m,n)
@given(n=st_py_natural, m=st_py_natural)
@settings(max_examples=50)
def test_add_commutative(n: PyNatural, m: PyNatural) -> None:
    assert n.add(m) == m.add(n)

# Associatividade da Adição: add(add(n,m),p) = add(n,add(m,p))
# Para esta, precisaríamos de uma estratégia para três PyNaturals
st_three_py_naturals = st.tuples(st_py_natural, st_py_natural, st_py_natural)
@given(data=st_three_py_naturals)
@settings(max_examples=30)
def test_add_associative(data: tuple[PyNatural, PyNatural, PyNatural]) -> None:
    n, m, p = data
    assert (n.add(m)).add(p) == n.add(m.add(p))

```
Neste arquivo de teste `test_py_natural.py`:
*   As funções `test_add_identity_with_zero` e `test_add_with_successor` verificam os axiomas da adição. Elas são decoradas com `@given` e usam a estratégia `st_py_natural` para gerar instâncias de `n` e `m`. As asserções comparam os resultados das chamadas aos métodos da classe `PyNatural`, traduzindo diretamente os axiomas.
*   Similarmente, `test_mult_with_zero_is_zero` e `test_mult_with_successor` verificam os axiomas da multiplicação.
*   Foram adicionados também testes para propriedades como comutatividade e associatividade da adição, que seriam teoremas deriváveis da especificação de Peano. Para `test_add_associative`, uma estratégia `st_three_py_naturals` foi criada para gerar uma tupla de três instâncias `PyNatural`.
*   O decorador `@settings(max_examples=...)` é usado em alguns testes para reduzir o número de exemplos gerados, o que pode ser útil se a combinação de operações (como em `mult` que usa `add`, e `add` que poderia ser recursiva) tornar o teste mais lento, especialmente com `max_value` maiores nas estratégias de inteiros.

**Execução dos Testes (com `pytest`):**
1.  Certifique-se de que `pytest` e `hypothesis` estão instalados (`pip install pytest hypothesis`).
2.  Salve os arquivos `py_natural.py` e `test_py_natural.py` no mesmo diretório.
3.  Abra um terminal nesse diretório e execute o comando: `pytest`

`pytest` descobrirá e executará as funções `test_*` no arquivo `test_py_natural.py`. Hypothesis gerará os dados, chamará os testes, e reportará quaisquer falhas (contraexemplos) ou passagens.

Se a implementação de `add` ou `mult` em `PyNatural` fosse feita de forma puramente recursiva seguindo os axiomas de Peano (em vez de usar `+` e `*` do Python diretamente, o que foi feito por simplicidade na classe `PyNatural`), esses testes de propriedade seriam ainda mais cruciais para verificar a corretude dessas implementações recursivas. Mesmo usando `+` e `*` do Python, os testes verificam se essas operações nativas, quando encapsuladas na classe `PyNatural`, ainda se conformam axiomaticamente ao comportamento esperado para `Natural` segundo Peano. Qualquer desvio (e.g., se houvesse um erro na lógica de `succ` ou `zero`, ou se `add` tivesse um bug sutil) seria potencialmente capturado por Hypothesis.

Este exemplo demonstra como o PBT com Hypothesis pode fechar o ciclo entre a especificação algébrica formal e a validação da implementação Python, aumentando a confiança na corretude do código em relação ao seu design formal.

**Exercício:**

Considerando a classe `PyNatural` e a estratégia `st_py_natural` definidas acima. A especificação de `Natural` também inclui o axioma para `isZero` em relação a `succ`: (N2) `isZero(succ(n)) = false`. Escreva uma função de teste `pytest` chamada `test_is_zero_of_successor_is_false` que use Hypothesis para verificar este axioma para a implementação `PyNatural`.

**Resolução:**

Aqui está a função de teste `pytest` para verificar o axioma (N2) `isZero(succ(n)) = false`.

```python
# test_py_natural.py (continuação, ou em um novo arquivo importando o necessário)
from hypothesis import given, strategies as st
from py_natural import PyNatural # Assume que py_natural.py está acessível

# Reutilizando as estratégias definidas anteriormente:
# st_natural_value = st.integers(min_value=0, max_value=100)
# st_py_natural = st.builds(PyNatural, value=st_natural_value)

# --- Teste para o axioma isZero(succ(n)) = false ---

@given(n=st_py_natural)
def test_is_zero_of_successor_is_false(n: PyNatural) -> None:
    """
    Verifica o axioma (N2): isZero(succ(n)) = false
    Em Python: (n.succ()).is_zero() == False
    """
    # Calcula o sucessor de n
    succ_of_n: PyNatural = n.succ()
    
    # Verifica se is_zero aplicado ao sucessor é False
    # A constante 'false' do TAD Boolean é representada por Python 'False'
    assert succ_of_n.is_zero() == False

# Para executar este teste junto com os outros:
# No terminal, no diretório do projeto: pytest
```

**Explicação do Teste:**

1.  **Importações e Estratégias:** Reutilizamos `given` de `hypothesis`, `strategies as st`, e a classe `PyNatural`. A estratégia `st_py_natural` é usada para gerar a instância `n` do tipo `PyNatural`.

2.  **Decorador `@given(n=st_py_natural)`:** Informa a Hypothesis para gerar instâncias `PyNatural` e passá-las como o argumento `n` para a função de teste.

3.  **Lógica do Teste (`test_is_zero_of_successor_is_false`):**
    *   `succ_of_n: PyNatural = n.succ()`: Primeiro, calculamos o sucessor do `PyNatural` `n` gerado, usando o método `succ()` da classe `PyNatural`. O resultado é armazenado em `succ_of_n`.
    *   `assert succ_of_n.is_zero() == False`: Esta é a asserção principal. Ela chama o método `is_zero()` sobre `succ_of_n` e verifica se o resultado é igual a `False` (a representação Python da constante `false` do TAD `Boolean`). Isso corresponde diretamente ao axioma `isZero(succ(n)) = false`.

Quando este teste for executado por `pytest`, Hypothesis irá gerar diversos objetos `PyNatural` para `n`, calcular `n.succ()`, e então verificar se `(n.succ()).is_zero()` é sempre `False`. Como, por definição, o sucessor de qualquer número natural nunca é zero, esta propriedade deve sempre se manter para uma implementação correta.

Este capítulo demonstrou a aplicação do Teste Baseado em Propriedades (PBT) com a biblioteca Hypothesis para validar implementações de TADs em Python, utilizando os axiomas de suas especificações algébricas como fonte de propriedades. Foi estabelecida a ponte entre axiomas e testes executáveis, contrastando PBT com testes manuais. Introduzimos os princípios do PBT, a geração de dados via estratégias em Hypothesis (incluindo `st.builds` para classes, `st.recursive` para tipos recursivos, e métodos de composição), o processo de "shrinking" de contraexemplos, e a integração com `pytest`. Um exemplo prático com o TAD `Natural` ilustrou como implementar uma classe Python tipada e testar seus axiomas de adição e multiplicação. O próximo capítulo, "Modularização e Reuso em Especificações Algébricas: Abordagens In-the-Large", mudará o foco de especificações individuais para como elas podem ser estruturadas e combinadas para construir sistemas maiores e mais complexos, promovendo reuso e gerenciamento de complexidade em larga escala.

---
## REFERÊNCIAS BIBLIOGRÁFICAS
ARCURIE, A.; FRASER, G. **Practical Unit Testing with JUnit and Mockito**. Birmingham: Packt Publishing, 2013. (Capítulo sobre Property-Based Testing, embora com foco em Java/JUnit, os princípios são gerais).
*   *Resumo: Embora focado em Java, este livro contém discussões sobre os princípios do teste de unidade e introduz o teste baseado em propriedades como uma técnica avançada. A relevância reside na contextualização do PBT dentro das práticas de teste mais amplas e seus benefícios gerais.*

BACH, J.; BOLTON, M. **Rapid Software Testing: A Practitioner's Handbook for Testers, Test Managers and Developers**. Boston: Addison-Wesley Professional, 2022.
*   *Resumo: Um livro moderno sobre abordagens exploratórias e contextuais para teste de software. Embora não foque exclusivamente em PBT, a filosofia de explorar o software em busca de falhas de maneiras não óbvias se alinha com os benefícios do PBT, e pode inspirar a formulação de propriedades interessantes.*

CLAESSEN, K.; HUGHES, J. QuickCheck: a lightweight tool for random testing of Haskell programs. **ACM SIGPLAN Notices**, v. 35, n. 9, p. 268-279, set. 2000. (ICFP '00).
*   *Resumo: Este é o artigo seminal que introduziu QuickCheck, a biblioteca original de teste baseado em propriedades para Haskell, que inspirou Hypothesis e muitas outras. É fundamental para entender as origens, a motivação e os mecanismos centrais do PBT, como geração de dados e shrinking.*

HYPOTHESIS. **Hypothesis documentation**. Disponível em: https://hypothesis.readthedocs.io/. Acesso em: 10 ago. 2023.
*   *Resumo: A documentação oficial da biblioteca Hypothesis é a referência primária e mais completa para aprender sobre suas funcionalidades, estratégias de geração de dados, configurações e exemplos de uso. É um recurso indispensável para qualquer desenvolvedor que utilize Hypothesis para PBT em Python.*

MACIVER, D. R. **Hypothesis: A new approach to property-based testing**. Apresentado em: PyCon US 2015, Montréal, QC, Canada. Disponível em: https://www.youtube.com/watch?v=E8LdYJ2jYlE. Acesso em: 10 ago. 2023.
*   *Resumo: Uma apresentação do criador do Hypothesis, David R. MacIver, que explica a filosofia por trás da biblioteca, suas principais características e como ela se diferencia de outras abordagens de teste. Fornece insights valiosos sobre o design e a aplicação prática do Hypothesis.*

PERCIVAL, H. J. W. **Test-Driven Development with Python**: Obey the Testing Goat: Using Django, Selenium, and JavaScript. 2. ed. Sebastopol, CA: O'Reilly Media, 2017. 
*   *Resumo: Embora o foco principal seja TDD e testes de integração/funcionais com Django, este livro prático sobre teste em Python fornece um excelente contexto sobre a cultura de teste em Python. Discussões sobre como escrever testes eficazes e robustos são relevantes, e o PBT complementa as técnicas de TDD.*
---
