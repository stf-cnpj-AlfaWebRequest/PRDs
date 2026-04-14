## Contexto

Você está analisando um repositório contendo arquivos COBOL,

Os programas podem conter regras de negócio, validações, manipulação de arquivos e/ou interação com dados.

---

## Definições

Chave: É uma chave (primária ou estrangeira) do CNPJ, e não do conteúdo do CNPJ, onde o campo de chave estrangeira em um arquivo de transação, ou campo de chave primária em um arquivo de consulta, etc. (ex: 78465270000165, 00000000000001). A **Chave de CNPJ** é um identificador técnico interno e não deve ser tratada como documento oficial de negócio.

Valor: É o conteúdo do CNPJ, ou seja, o campo é manipulado como valor de negócio, onde o campo é utilizado em regras de negócio, cálculos, validações, tela (GUI) etc. (ex: 78465270000165, 6QNJ8VY2JIC341). O **Valor de CNPJ** é o identificador oficial utilizado pelo negócio e por interfaces externas.

Módulos: Refere-se a qualquer artefato de código, incluindo programas COBOL e copybooks (arquivos .CPY) que são utilizados pelos programas COBOL. (ex: RF, DAF, VIP, HLP, etc.)

Módulo Principal: É o nome dado a um módulo que pertence ao mesmo sistema dos programas principais. Módulos principais podem ou não ser alterados a depender do plano de execução e execução propriamente dita, mas pertencem ao mesmo time/sistema. A identificação de módulos principais é feita dinamicamente com base no prefixo do nome do artefato. (ex: se os programas principais são VIPP4365 e VIPP4553, então qualquer artefato com prefixo VIP é considerado principal, mesmo que não requeira alteração).

Módulo Secundário: É o nome dado a um módulo que pertence a um sistema diferente dos programas principais. Módulos secundários podem ou não ser alterados a depender do plano de execução e execução propriamente dita, pois pertencem a outros times/sistemas. A identificação de módulos secundários é feita dinamicamente com base no prefixo do nome do artefato. (ex: se os programas principais são VIPP4365 e VIPP4553, então artefatos com prefixo DAF ou HLP são considerados secundários).

Programas: Refere-se ao código COBOL em si, ou seja, aos arquivos .CBL, .CPY, entre outros. Esses arquivos são os artefatos que contêm a lógica de negócio e que serão modificados (patch). No cliente, seus nomes seguem o padrão de um prefixo de 3 letras maiúsculas seguido de um número, por exemplo: VIPP4365.cbl, DAF1234.cbl, HLP5678.cbl.

Correlação 1:1: Relação obrigatória e inequívoca entre uma **Chave de CNPJ** e um **Valor de CNPJ**, preservando rastreabilidade, integridade relacional, auditabilidade e determinismo da conversão.

CGC: Denominação anterior do CNPJ (Cadastro Geral de Contribuintes), equivalente ao CNPJ para todos os efeitos desta demanda. Campos nomeados como CGC devem ser tratados com as mesmas regras aplicáveis ao CNPJ.

PF: Pessoa Física, ou seja, indivíduo, pessoa natural. Campos relacionados a PF não estão no escopo desta demanda, mas podem ser utilizados para comparação e inferência quando necessário.

PJ: Pessoa Jurídica, ou seja, empresa, organização. Campos relacionados a PJ estão no escopo desta demanda, pois o CNPJ é o identificador oficial de pessoas jurídicas.

Fonte Externa: Origem de dados pertencente a um sistema externo ao escopo desta modificação, como tabelas DB2 de outros domínios (ex: DB2MCI.CLIENTE), arquivos legados de terceiros ou integrações de entrada. Dados provenientes de fontes externas não estão no escopo de modificação desta demanda e não devem ser alterados.

Campo Indefinido: Campo CNPJ ou CGC cuja natureza semântica (Chave ou Valor) não pode ser determinada com segurança a partir dos artefatos de discovery e diagnostic — ou seja, onde a taxa de inferência for fraca. Nesses casos, o campo deve ser classificado como Indefinido e não deve ser alterado.

Serviço Central de Conversão: Serviço oficial responsável por converter **Chave de CNPJ** em **Valor de CNPJ** e **Valor de CNPJ** em **Chave de CNPJ**. É proibida a geração local ou distribuída dessa correlação fora do sistema central autorizado.

Base Central de Correlação (DFE): Estrutura central responsável por manter a relação 1:1 entre **Chave de CNPJ** e **Valor de CNPJ**, bem como metadados de controle e auditoria.

Cache: Camadas auxiliares de desempenho utilizadas para reduzir acessos à base central de correlação, podendo incluir mecanismos equivalentes a Redis, VSAM, processamento batch ou estruturas legadas compatíveis com o ambiente.

---

## Objetivo

Gerar **4 artefatos essenciais de teste** para cada programa COBOL (.cbl) localizado dentro da pasta .pss/b_new/a_code:

1. Dicionário de Dados
2. Cenários de Teste
3. Casos de Teste
4. Massa de Teste

Esses artefatos serão utilizados posteriormente para:

- Implementação de testes automatizados com Robot Framework e TREXX scripts
- Geração de Code Coverage baseado em execução real

Além disso, leve em consideração o documento de requisitos localizado no caminho .pss\b_new\w_output\dem_a_cnpj_alfa_prd\dem_prd_requisito.md para entender o contexto do projeto e dos programas COBOL.


---

## Escopo

- Processar TODOS os programas COBOL do repositório
- Utilizar os copybooks como complemento estrutural
- Cada programa deve ter seus próprios artefatos
- Se um código Cobol chama outro, o ROBOT deve ser capaz de continuar o fluxo de teste e completá-lo.

---

## Instruções Gerais

### Fonte da Verdade

- O código COBOL é a fonte principal (source of truth)
- Copybooks devem ser expandidos logicamente no contexto do programa
- Não inventar regras que não existam no código

---

## 1. Dicionário de Dados

Gerar um arquivo estruturado contendo:

Para cada variável relevante (especialmente inputs):

- Nome da variável
- Nível (01, 05, etc.)
- PIC
- Tipo (Numérico / Alfanumérico)
- Tamanho
- Descrição (inferida com base no uso)
- Origem:
  - WORKING-STORAGE
  - LINKAGE SECTION
  - FILE SECTION
- Indicação se é:
  - Input
  - Output
  - Interna

Prioridade:

- Variáveis de entrada (INPUT) são obrigatórias
- Variáveis usadas em condições (IF, EVALUATE)
- Variáveis de saída

---

## 2. Cenários de Teste

Gerar cenários de alto nível com base na lógica do programa.

Cada cenário deve conter:

- ID do cenário
- Nome descritivo
- Descrição
- Fluxo principal coberto
- Tipo:
  - Positivo
  - Negativo
  - Borda (edge case)

Cobrir obrigatoriamente:

- Fluxos principais
- Validações (IF, EVALUATE)
- Tratamento de erro
- Caminhos alternativos

---

## 3. Casos de Teste

Para cada cenário, detalhar casos de teste contendo:

- ID do caso
- Cenário relacionado
- Descrição
- Pré-condições
- Dados de entrada
- Passos (alto nível, compatíveis com automação)
- Resultado esperado

Regras:

- Deve ser possível converter diretamente para Robot Framework
- Inputs devem referenciar o Dicionário de Dados

---

## 4. Massa de Teste

Gerar massa de dados estruturada para execução dos testes:

Formato sugerido:

- CSV ou tabela estruturada

Conteúdo:

- Valores válidos
- Valores inválidos
- Valores de borda
- Combinações relevantes

Regras:

- Cada massa deve estar vinculada a um caso de teste
- Dados devem respeitar o PIC das variáveis
- Incluir casos que forcem diferentes caminhos do código

---

## Regras de Qualidade

- Garantir rastreabilidade:

  Cenário → Caso de Teste → Massa

- Garantir cobertura lógica:
  - IFs
  - EVALUATE
  - PERFORMs relevantes

- Evitar redundância
- Ser claro, objetivo e estruturado

---

## Formato de Saída

Gerar 4 arquivos por programa COBOL:

### Estrutura:

onde salvar os arquivos:

dentro de .pss/b_new/c_test criar uma pasta chamada "a_csv"

dentro de "a_csv", criar uma pasta para cada arquivo cobol analisado.

o nome dos 4 arquivos devem ter como prefixo o nome em MAIÚSCULO do código COBOL, seguindo o modelo abaixo:

(NOME_DO_ARQUIVO_COBOL)\_data_dictionary
(NOME_DO_ARQUIVO_COBOL)\_test_cases
(NOME_DO_ARQUIVO_COBOL)\_test_data
(NOME_DO_ARQUIVO_COBOL)\_test_scenarios

---

## Observações Importantes

- Não gerar código Robot Framework ainda
- Preparar tudo para fácil conversão futura
- Focar em qualidade de cobertura (não apenas quantidade)
- Priorizar caminhos críticos do sistema

---

## Critério de Sucesso

O material gerado deve permitir:

- Criação direta de testes automatizados
- Execução real do COBOL
- Geração de Code Coverage confiável baseado em execução
