## Contexto:
Você está analisando um repositório contendo programas COBOL e Copybooks.

## Objetivo:
Gerar um relatório de Code Coverage baseado na análise dos programas COBOL presentes na pasta: .pss/b_new/a_code e os arquivos de Teste em RobotFramework dentro da pasta .pss/b_new/c_test/b_robot_script e da pasta .pss/b_new/c_test/c_rexx_script, após, documentar o resultado em um arquivo Markdown estruturado. 

Além disso, leve em consideração o documento de requisitos localizado no caminho .pss\b_new\w_output\dem_a_cnpj_alfa_prd\dem_prd_requisito.md para entender o contexto do projeto e dos programas COBOL.


---

## Instruções:

1. Entendimento do Sistema

- Leia todos os arquivos da pasta "docs"
- Priorize o entendimento do arquivo:
  docs/system-overview.md
- Utilize os scripts de Teste em ROBOT e TREXX feitos para os arquivos COBOL do repositório para fazer o Code Coverage
- Utilize essas informações para compreender:
  - Fluxos principais
  - Entradas (inputs)
  - Processamentos
  - Saídas

---

2. Análise dos Programas COBOL

Para cada programa COBOL:

- Identifique:
  - Parágrafos (PERFORM, SECTION, etc.)
  - Estruturas condicionais (IF, EVALUATE)
  - Chamadas externas (CALL)
  - Uso de variáveis de entrada

- Classifique trechos como:
  - Executáveis
  - Condicionais
  - Não executados (potenciais gaps)

---

3. Geração do Code Coverage (Estimado)

Com base na análise dos códigos COBOL e dos testes em Robot Framework/Treex scripts:

Faça separadamente, para ROBOT e para T-REXX, as etapas abaixo:

- Calcule:
  - Line Coverage (%)
  - Branch Coverage (%)
  - Parágrafos cobertos vs não cobertos

- Identifique:
  - Trechos críticos não cobertos
  - Regras de negócio sem validação
  - Fluxos alternativos não testados

OBS: Qualquer tabela gerada deve haver as MESMAS colunas para ambos os scripts
---

4. Geração do Documento Markdown

Crie a subpasta dem_n_coverage dentro da pasta .pss/b_new/w_output/

Crie o arquivo:
.pss/b_new/w_output/dem_n_coverage/dem_n_test_coverage.md

Com a seguinte estrutura obrigatória:

# Code Coverage Report

## 1. Introdução

- Descrição do sistema
- Escopo analisado

## 2. Coverage Summary

- Tabela com:
  - Programa
  - Line Coverage
  - Branch Coverage
  - Observações

## 3. Detailed Analysis

Para cada programa:

- Descrição
- Pontos cobertos
- Pontos não cobertos
- Riscos

## 4. Critical Gaps

- Regras de negócio não testadas
- Fluxos críticos ignorados

## 5. Recommendations

- Sugestões de melhoria (Caso haja)
- Prioridades de testes

---

## Premissas:
Essa documentação precisa ter as seguintes seções:
- **Sessão**: sempre colocar um paragrafo curto, explicando o que a sessão vai abordar.
- **Conclusão**: sempre colocar uma conclusão ao final de cada tarefas/módulos/doc/md.
- Toda tabela necessita ter uma legenda explicando/descrevendo cada coluna.
- Verificar se temos algo em inglês, pois tem de ser em português. Toda a documentação tem que ser em português, a menos que tenhamos algum termo técnico em inglês.
- Teremos 2 casos de uso, para realizar a mudança do CNPJ de numérico para alfanumerico:
  - **Chave**: A variável de CNPJ no código é uma chave estrangeira e não o CNPJ propriamente dito.
  - **Valor**: A variável de CNPJ no código é o próprio conteúdo da CNPJ.
- Chave/Valor:
  - **Mudar de REAL para VALOR**
  - **Mudar de LOOKUP para CHAVE**
- Para essa documentação, não faremos a seção de Resumo Executivo.
- Assumimos que não faremos a validação da rotina acima, nesta PoC.
- Considerando que podemos ter 2 cenários, onde temos campo CHAVE e campo VALOR, decidimos por considerar de 1 arquivos cobol será de CHAVE e o outro será de VALOR, mesmo sabendo que ambos os arquivos são originalmente de CHAVE. Esta decisão intenciona mostrar que identificamos e entendemos os dois casos e somos capazes de performar em ambos:
  - Não pode alterar módulos externos (DAF, HLP, etc).
  - Não pode pegar a Chave nem o Valor dos módulos que sejam principais.
  - VIPP4365.cbl = Considere esse arquivo sendo como chave (ID), ou seja, o campo do CNPJ seria apenas um identificador (chave estrangeira, fk), (Chave).
  - VIPP4553.cbl = Considere esse arquivo como se fosse um caso de valor do CNPJ (Valor). \* Funções para obter/validar o CNPJ:
  - **Plataforma Baixa**: Chamar de "Função de Obternção/Validação do CNPJ em Plataforma Baixa", e não utilizar o nome verdade (getXxxbyXxx).
** Plataforma Alta: Chamar de "Função de Obternção/Validação do CNPJ em Plataforma Alta", e não utilizar o nome verdade (SISXXXX).

## Regras adicionais:
- Dentro do documento gerado, na parte do Coverage propriamente dito, faça uma divisão na análise entre os scripts ROBOTs e os scripts TREXX.
- Não invente dados: baseie-se apenas no código e documentos.
- Seja técnico e objetivo.
- Não coloque "visão geral" no começo do arquivo
- Use tabelas sempre que possível.
- Destaque riscos reais.
- Priorize qualidade sobre quantidade.
- Escreva todo o documento em Português.
- Toda tabela necessita ter uma legenda explicando/descrevendo cada coluna.
- Ao final do documento, faça uma sessão de conclusão.
- não coloque no topo do documento metadados. Exemplo: "Data: 25 de Março de 2026 Versão: v3.0 Especificação: bb_cartao_pj_prepago_b_cnpj_alfa_c_technical_20260322.md"
- NÃO utilizar emojis em nenhuma parte da saída.
- como última sessão do .md gerado, coloque no padrão:
  "

## Rodapé

---

Documento Elaborado: <Data de hoje> ex: dd/mm/yyyy
Versão: 5.0

---

"

- Não escrever caminhos de arquivos, ex: ".pss/"
- Cada sessão deve começar com um breve parágrafo explicando o que tal sessão irá abordar.
- Tabelas geradas na parte do Robot devem ter uma tabela identica na parte do Trexx, ou seja, as mesmas colunas, para facilitar a comparação entre os dois scripts.