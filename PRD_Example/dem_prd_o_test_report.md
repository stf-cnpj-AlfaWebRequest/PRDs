## Contexto:
Você está analisando um repositório contendo programas COBOL e Copybooks.
## Objetivo:
Gerar uma documentação estruturada de testes para os programas COBOL que estão na pasta ".pss/b_new/a_code" levando em consideração o documento na pasta .pss/b_new/w_output/dem_n_coverage/dem_n_test_coverage.md e os arquivos Robots de teste dentro da pasta .pss/b_new/c_test/b_robot_scripts e os arquivos de teste TREXX dentro da pasta .pss/b_new/c_test/c_rexx_scripts.
Leve em consideração quaisquer arquivo.md presente em tais pastas também. 

Além disso, leve em consideração o documento de requisitos localizado no caminho .pss\b_new\w_output\dem_a_cnpj_alfa_prd\dem_prd_requisito.md para entender o contexto do projeto e dos programas COBOL.

Instruções:

1. Regras Gerais

- Utilize o COBOL como fonte principal (source of truth) para estruturas de dados e variáveis
- Garanta consistência entre todos os artefatos gerados
- Todas as saídas devem ser determinísticas e estruturadas
- Considerar arquivos .CPY (copybooks) como extensão dos programas COBOL
- Expandir instruções COPY durante a análise
- Variáveis definidas em copybooks devem ser tratadas como parte do programa
- Nunca criar variáveis, regras ou fluxos que não estejam explicitamente no COBOL ou docs
- Em caso de dúvida, marcar como UNKNOWN
- Criar uma legenda, quando necessário, especificado que um cenário SUNNY é um cenário positivo e RAINY é um cenário negativo.

- Antes de gerar qualquer coisa, leia todos os documentos da pasta "docs", que está na raiz do projeto e da pasta .pss/b_new/c_test/a_csv.

2. Documentação Técnica
   Gerar um arquivo dentro do repositório especificado abaixo:
   .pss/b_new/b_doc_b_dem/dem_o_test_report.md

Leia quaisquer MDs presentes nas pastas .pss/b_new/c_test/b_robot_scripts e .pss/b_new/c_test/c_rexx_scripts e crie sessões específicas dentro da documentação. Adicione o conteúdo presente nos MDs encontrados entre os tópicos "9. Painel de Cobertura" e "10. Rastreablidade"

   Conteúdo obrigatório:

1. Introdução
2. Estrutura de Dados (Data Dictionary)
3. Fluxo Funcional
4. Cenários (Scenarios)
5. Casos de Teste (Cases)
6. Massa de Teste (Data Mass)
7. Scripts (Robot)
8. Scripts (TREXX)
9. Painel de Cobertura
10. Rastreabilidade
11. Premissas e Limitações
12. Conclusão
13. Rodapé

## 1. Introdução

- Descrição do programa COBOL
- Objetivo principal
- Contexto de uso

## 2. Estrutura de Dados

- Explicação das principais variáveis
- Relação com os dicionarios de dados da pasta .pss/b_new/c_test/a_csv

## 3. Fluxo Funcional

- Descrição dos principais fluxos do programa
- Baseado no COBOL
- Explicação de decisões, validações e branches

## 4. Estratégia de Testes

- Explicação de como os cenários foram definidos
- Diferença entre Sunny e Rainy aplicada ao caso
- Cobertura de testes

## 5. Relação entre Artefatos

- Como cenários, casos e massa se conectam
- Explicação da rastreabilidade

## 6. Premissas e Limitações

- O que foi inferido
- O que não pôde ser determinado (UNKNOWN)

## 7. Matriz de rastreabilidade:

    Gerar uma matriz de rastreabilidade de chamadas entre programas COBOL (Call Traceability Matrix), identificando quais programas chamam outros programas.

    Instruções:

    1. Identificação dos Programas

    * Antes de construir a matriz, gere explicitamente a lista de todos os arquivos COBOL identificados no input. Essa lista será a base oficial da matriz.
    * O nome do arquivo na tabela deve ser exatamente igual ao nome real do arquivo (Inclui extensão)
    2. Identificação de Chamadas

Analise o código COBOL e identifique chamadas entre programas usando:

    * CALL 'NOME-PROGRAMA'
    * CALL "NOME-PROGRAMA"
    * EXEC CICS LINK PROGRAM('NOME-PROGRAMA')
    * EXEC CICS XCTL PROGRAM('NOME-PROGRAMA')

    3. Regras de Extração

    * Ignore chamadas dinâmicas (ex: CALL WS-PGM-NAME)
    * Considere apenas chamadas explícitas (hardcoded)
    * Ignore PERFORM (pois são chamadas internas)
    * Ignore includes/copybooks (COPY)

    4. Construção da Matriz

    * Crie uma matriz no formato tabela
    * Linhas representam o programa que FAZ a chamada
    * Colunas representam o programa que É CHAMADO
    * Cada célula deve conter:

      * "X" se houver chamada
      * vazio caso contrário
      * "-" na diagonal principal (programa chamando a si mesmo)

    5. Padronização

    * Use nomes de programas em UPPERCASE
    * Ordene linhas e colunas alfabeticamente

    6. Saída Esperada

    A saída deve conter:

    (1) Lista de Programas Identificados
    (2) Matriz de Rastreabilidade no formato de tabela (Markdown ou CSV)
    (3) Lista de chamadas detectadas no formato:
    PROGRAMA_ORIGEM → PROGRAMA_DESTINO

    7. Exemplo de saída:

    Programas:
    A, B, C

    Matriz:

    | Programa | A | B | C |
    | -------- | - | - | - |
    | A        | - | X |   |
    | B        |   | - | X |
    | C        | X |   | - |

    Chamadas:
    A → B
    B → C
    C → A

    8. Regras adicionais (importante)

    * Programas chamados que não existam entre os arquivos analisados devem:

- Ser incluídos como colunas na matriz
- NÃO aparecer como linhas
- Ser listados separadamente como "External Programs"
  - Não invente chamadas
  - Baseie-se exclusivamente no código fornecido
  9. Análise da Matriz (obrigatório)

  Após gerar a matriz, incluir:
  - Programas mais chamados (alta dependência)
  - Programas que não são chamados por nenhum outro (possíveis órfãos)
  - Possíveis ciclos de chamada (loops)

Regras:

- Linguagem clara, técnica e objetiva
- Escreva todo o documento em Português.
- Toda tabela necessita ter uma legenda explicando/descrevendo cada coluna.
- Ao final do documento, faça uma sessão de conclusão.
- não coloque no topo do documento metadados. Exemplo: "Data: 25 de Março de 2026 Versão: v3.0 Especificação: bb_cartao_pj_prepago_b_cnpj_alfa_c_technical_20260322.md"
- Ao início do arquivo, coloque uma sessão "Introdução"
- como última sessão do .md gerado, coloque no padrão:
  "

## Rodapé

---

Documento Elaborado: <Data de hoje> ex: dd/mm/yyyy
Versão: 6.0

---

"

- Não escrever caminhos de arquivos, ex: ".pss/".
- Não escrever estrutura dos arquivos em nenhum momento. Sem excessões.
- Não escrever o nome dos arquivos em nenhum momento. Sem excessões.
- Não referenciar o PRD diretamente.
- Analisar cuidadosamente o documento de requisitos para entender o contexto do projeto e dos programas COBOL, a fim de evitar contradições internas.
- Analisar cuidadosamente as PREMISSAS para evitar contradições internas.
- Cada sessão deve começar com um breve parágrafo explicando o que tal sessão irá abordar.
- Evitar explicações genéricas.
- Não coloque "visão geral" no começo do arquivo
- Sempre relacionar com os CSVs gerados.
- Gere um painel de cobertura no documento.
- O painel de cobertura deve ser bem estruturado, completo e deve conter:
  Total de cenários
  Total de casos de teste
  Quantidade de variáveis cobertas
  Estimativa de cobertura de branches (%)
  A estimativa de cobertura deve ser baseada em: - Quantidade de decisões (IF/EVALUATE) no COBOL - Quantidade de decisões cobertas por cenários nos CSVs
  Relação com CodeCoverage.md
- Use o arquivo CodeCoverage que está no caminho .pss/b_new/w_output/CodeCoverage.md como contexto para gerar o painel de cobertura também.
- NÃO utilizar emojis em nenhuma parte da saída

## Premissas:

Essa documentação precisa ter as seguintes seções:
- **Sessão**: sempre colocar um paragrafo curto, explicando o que a sessão vai abordar.
- **Conclusão**: sempre colocar uma conclusão ao final de cada tarefas/módulos/doc/md.
- **Toda tabela** necessita ter uma legenda explicando/descrevendo cada coluna.
- Verificar se temos algo em inglês, pois tem de ser em português. Toda a documentação tem que ser em português, a menos que tenhamos algum termo técnico em inglês.
- Teremos 2 casos de uso, para realizar a mudança do CNPJ de numérico para alfanumerico:
   - **Chave**: A variável de CNPJ no código é uma chave estrangeira e não o CNPJ propriamente dito.
   - **Valor**: A variável de CNPJ no código é o próprio conteúdo da CNPJ.
- Chave/Valor:
   - **Mudar de REAL para VALOR**
   - **Mudar de LOOKUP para CHAVE**
- Para essa documentação, não faremos a seção de Resumo Executivo.
Assumimos que não faremos a validação da rotina acima, nesta PoC.
Considerando que podemos ter 2 cenários, onde temos campo CHAVE e campo VALOR, decidimos por considerar de 1 arquivos cobol será de CHAVE e o outro será de VALOR, mesmo sabendo que ambos os arquivos são originalmente de CHAVE. Esta decisão intenciona mostrar que identificamos e entendemos os dois casos e somos capazes de performar em ambos:
- Não pode alterar módulos externos (DAF, HLP, etc).
- Não pode pegar a Chave nem o Valor dos módulos que sejam principais.
- VIPP4365.cbl = Considere esse arquivo sendo como chave (ID), ou seja, o campo do CNPJ seria apenas um identificador (chave estrangeira, fk), (Chave).
- VIPP4553.cbl = Considere esse arquivo como se fosse um caso de valor do CNPJ (Valor). \* Funções para obter/validar o CNPJ:
   - **Plataforma Baixa**: Chamar de "Função de Obternção/Validação do CNPJ em Plataforma Baixa", e não utilizar o nome verdade (getXxxbyXxx).
   - **Plataforma Alta**: Chamar de "Função de Obternção/Validação do CNPJ em Plataforma Alta", e não utilizar o nome verdade (SISXXXX).
