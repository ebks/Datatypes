**DIRETRIZES FUNDAMENTAIS PARA GERAÇÃO DE CAPÍTULOS DO LIVRO**

**Assunto do Livro:** Tipos Abstratos e Estruturas de Dados: Uma Abordagem Formal com Especificação Algébrica, Teoria de Categorias e Python.

**I. ESTRUTURA E CONTEÚDO GERAL DO CAPÍTULO:**

1.  **Contagem de Palavras:** O capítulo deve ter aproximadamente [7000 a 100000] palavras. O conteúdo deve ser denso e bem desenvolvido, evitando superficialidade para atingir a meta.
2.  **Linguagem:** Utilizar linguagem estritamente formal e acadêmica, em Português (Brasil).
3.  **Parágrafo Introdutório do Capítulo:**
    *   Iniciar o capítulo com um único parágrafo extenso (e.g., 200-300 palavras).
    *   Este parágrafo NÃO deve ser numerado como seção.
    *   Deve resumir o conteúdo e os objetivos do capítulo de forma abrangente, sem mencionar números ou nomes de seções específicas que virão a seguir.
4.  **Seções e Subseções:**
    *   Seções de **Nível 1** devem ser numeradas como `XX.1`, `XX.2`, etc. (onde `XX` é o número do capítulo). Título: `# XX.N TÍTULO DA SEÇÃO NÍVEL 1`.
    *   Seções de **Nível 2** devem ser numeradas como `XX.Y.1`, `XX.Y.2`, etc. Título: `## XX.Y.Z TÍTULO DA SEÇÃO NÍVEL 2`.
    *   Seções de **Nível 3 (e inferiores)**:
        *   **NÃO DEVEM SER NUMERADAS.**
        *   O título deve ser formatado apenas em **negrito**. Exemplo: `**Título da Sub-Subseção em Negrito**`.
5.  **Exercícios:**
    *   **NÃO DEVEM SER NUMERADOS.**
    *   Incluir um (1) exercício ao final de cada seção de Nível 1. 
    *   O enunciado do exercício deve ser claro.
    *   A resolução deve vir IMEDIATAMENTE após o enunciado do exercício.
    *   A resolução deve começar com a palavra "**Resolução:**" em negrito.
    
6.  **Parágrafo de Conclusão do Capítulo:**
    *   Finalizar o capítulo com um parágrafo conciso (e.g., 100-150 palavras).
    *   Este parágrafo deve resumir os principais aprendizados do capítulo.
    *   Deve indicar o que será abordado no capítulo seguinte.

7.  **Referências Bibliográficas:**
    *   Título da Seção: `## REFERÊNCIAS BIBLIOGRÁFICAS` (sem numeração, como um subtítulo de nível 2).
    *   Quantidade: Exatamente 6 (seis) referências por capítulo (livros ou artigos).
    *   Atualidade: Preferencialmente com datas de publicação posteriores a 2020, exceto para obras clássicas e fundamentais na área, que são permitidas.
    *   Formato: Estritamente ABNT (Norma Brasileira de Referências).
    *   Ordenação: Alfabética pelo sobrenome do primeiro autor.
    *   Estrutura de cada entrada de referência:
        1.  Referência completa no formato ABNT.
        2.  Em uma nova linha, indentado: `*   *Resumo: [Resumo conciso de aproximadamente três linhas sobre o conteúdo e a relevância da referência para o capítulo.]*`

**II. FORMATAÇÃO DE ELEMENTOS ESPECÍFICOS:**

1.  **Especificações Algébricas de TADs:**
    *   **NÃO utilizar delimitadores de bloco de código** (como ` ``` `) para formatar a especificação. Deve ser texto normal com a formatação abaixo.
    *   Todo o conteúdo *dentro* do bloco de especificação (nomes de sorts, operações, variáveis em axiomas, palavras-chave como `SPEC ADT`, `sorts`, `operations`, `axioms`, `for all`, `END SPEC`) deve estar **obrigatoriamente em Inglês**.
    *   Nenhum comentário ou texto explicativo em português/inglês deve aparecer *dentro* do bloco `SPEC ADT ... END SPEC`.
    *   Formato do Bloco:
        **SPEC ADT** NameOfTAD

        **sorts:**
        *   MainSortName
        *   [AnotherNeededSort1]

        **operations:**
        *   operationName1: [ArgumentSort1 x ArgumentSort2 x ...] --> ReturnSort
        *   operationName2: --> ReturnSort

        **axioms:**
        *   (Axiom PrefixNumber1): leftHandSideExpression1 = rightHandSideExpression1
        *   (Axiom PrefixNumber2): leftHandSideExpression2 = rightHandSideExpression2

        for all variable1: SortOfVariable1, variable2: SortOfVariable2, ...

        **END SPEC**
    *   **Parágrafo Explicativo:** Imediatamente após o bloco `**END SPEC**`, deve haver um único parágrafo em Português, explicando detalhadamente a especificação (sorts, operações, axiomas, cláusula `for all`), sem usar os símbolos ou a notação formal da especificação diretamente no texto explicativo (e.g., descrever "a operação `not` que recebe um booleano e retorna um booleano" em vez de "a operação `not: Boolean --> Boolean`").

2.  **Código Python:**
    *   Todo o código Python, incluindo nomes de variáveis, funções, classes, métodos E **comentários internos no código** (`# este é um comentário`), deve estar **obrigatoriamente em Inglês**.
    *   Os blocos de código devem ser delimitados por ` ```python ... ``` `.
    *   **Parágrafo Explicativo:** Imediatamente após cada bloco de código Python, deve haver um parágrafo em Português, explicando detalhadamente o código. Este parágrafo deve referenciar os nomes (variáveis, funções, etc.) usados no trecho de código. **NÃO** iniciar este parágrafo com frases como "A explicação do código Python acima:".

3.  **Equações Matemáticas (fora das especificações ADT):**
    *   Todas as equações matemáticas devem ser formatadas usando LaTeX dentro de `$ ... $`.
    *   **NÃO** deixar espaço em branco imediatamente após o primeiro `$` e imediatamente antes do segundo `$`. Exemplo correto: `$a^2 + b^2 = c^2$`.
    *   Quando uma equação precisar ser numerada para referência posterior, a numeração deve seguir o formato `(Equação XX.N)`, onde `XX` é o número do capítulo e `N` é o sequencial da equação no capítulo. Exemplo: `A relação é dada por $a^2 + b^2 = c^2$ (Equação 3.1).`

4.  **Demonstrações (Provas Matemáticas, não resoluções de exercícios):**
    *   Devem começar com a palavra "**Prova:**" em negrito.
    *   Devem terminar com o símbolo de fim de prova: `□` (quadrado vazado).

**III. FOCO DO CONTEÚDO (ESPECÍFICO PARA PARTE I, se aplicável ao capítulo atual):**

1.  **Exemplos de TADs (para Capítulos 1-7):** Ao ilustrar conceitos de especificação, reescrita, análise, refinamento, teste e modularização, os exemplos devem se concentrar **primariamente** nos TADs `Boolean`, `Natural` (e.g., Peano) e `NaturalList` (lista de `Natural`s). Evitar estruturas de dados mais complexas (Pilhas, Filas, Árvores, etc.) como exemplos principais na Parte I.

