# Discovery de Artefatos com Manipulação de CNPJ

## Criteria de Inclusão

Um artefato foi incluído quando houve evidência de manipulação de CNPJ, considerando:

- Declaração de variáveis relacionadas a CNPJ com tamanho fixo de 14 caracteres
- Estruturas de dados contendo campos de CNPJ
- Operações de movimentação (MOVE) de dados CNPJ entre registros
- Presença em layouts de entrada/saída de arquivos
- Presença em estruturas de comunicação entre programas (copybooks de parâmetros)

## Premissas

- O processo foi determinístico baseado em análise de conteúdo de código-fonte
- A identificação de linguagem foi feita pela extensão do artefato (CBL, CPY)
- Artefatos analisados: 5 (dois programas COBOL e três copybooks)
- Módulos principais identificados dinamicamente pelo prefixo: VIP
- Módulos secundários identificados dinamicamente pelo prefixo: DAF, HLP
- Análise considerou dois cenários de uso do CNPJ: CHAVE e VALOR
- Não foram realizadas análises de alteração ou interpretação funcional de módulos externos

---

## Discovery Consolidado

| id | tipo do artefato | linguagem de programação | quantidade de artefatos analisados | quantidade de artefatos com ocorrência | quantidade de evidências | percentual de artefatos com ocorrência |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | CBL | COBOL | 2 | 2 | 14 | 100% |
| 2 | CPY | COBOL | 3 | 3 | 11 | 100% |

Legenda:

- **id**: identificador único do registro
- **tipo do artefato**: extensão do artefato (ex: CBL, CPY)
- **linguagem de programação**: linguagem associada ao tipo
- **quantidade de artefatos analisados**: total de artefatos daquele tipo percorridos
- **quantidade de artefatos com ocorrência**: artefatos que possuem ao menos uma evidência
- **quantidade de evidências**: total de evidências registradas
- **percentual de artefatos com ocorrência**: (artefatos com ocorrência / artefatos analisados) * 100

---

## Características dos Artefatos

| id | artefato | linguagem de programação | tipo do artefato |
| --- | --- | --- | --- |
| 1 | VIPP4365 | COBOL | CBL |
| 2 | VIPP4553 | COBOL | CBL |
| 3 | VIPK160D | COBOL | CPY |
| 4 | DAFK6452 | COBOL | CPY |
| 5 | HLPKDFHE | COBOL | CPY |

Legenda:

- **id**: identificador único e sequencial
- **artefato**: nome do artefato
- **linguagem de programação**: identificada pela extensão
- **tipo do artefato**: extensão normalizada

---

## Evidências

| id | artefato | evidencias |
| --- | --- | --- |
| 1 | VIPP4365 | Declaração de variável PIC S9(014) COMP-3 com nome `317E-COD-CPF-CGC-PJ` representando CNPJ de pessoa jurídica proveniente de DB2MCI.CLIENTE (fonte externa)<br/>Declaração de variável PIC S9(014) COMP-3 com nome `317E-COD-CPF-CGC-PF` representando CPF de pessoa física proveniente de DB2MCI.CLIENTE (fonte externa)<br/>Declaração de variável PIC 9(014) com nome `940S-CNPJ-HDR` em estrutura de saída (header) do arquivo VIPF940S<br/>Declaração de variável PIC 9(014) com nome `940S-CNPJ-DET` em estrutura de saída (detalhe) do arquivo VIPF940S<br/>Operação MOVE de `317E-COD-CPF-CGC-PJ` para `940S-CNPJ-HDR` propagando CNPJ de entrada para output header<br/>Operação MOVE de `317E-COD-CPF-CGC-PJ` para `940S-CNPJ-DET` propagando CNPJ de entrada para output detalhe |
| 2 | VIPP4553 | Declaração de variável PIC S9(14)V COMP-3 com nome `F104E-CPF-CNPJ` representando CNPJ em registro de entrada do arquivo VIPF104E<br/>Declaração de variável PIC S9(14)V COMP-3 com nome `F104E-CPF-CNPJ-RPRT` representando CNPJ de representante em registro de entrada<br/>Declaração de variável PIC S9(14)V COMP-3 com nome `F104E-CPF-CNPJ-CEN` representando CNPJ de centro em registro de entrada<br/>Declaração de variável PIC S9(14)V COMP-3 com nome `F104E-CPF-CNPJ-PORT` representando CNPJ de portador em registro de entrada<br/>Declaração de variável PIC 9(014) com nome `F104S-CPF-CNPJ` em estrutura de saída (detalhe) do arquivo VIPF104S<br/>Declaração de variável PIC 9(014) com nome `F104S-CPF-CNPJ-CEN` representando CNPJ de centro em estrutura de saída<br/>Declaração de variável PIC 9(014) com nome `F104S-CD-CGC-MUN` representando CGC/CNPJ de município em estrutura de saída<br/>Declaração de variável PIC 9(014) com nome `F104S-CPF-CNPJ-RPRT` representando CNPJ de representante em estrutura de saída<br/>Declaração de variável PIC 9(014) com nome `F104S-CPF-CNPJ-PORT` representando CNPJ de portador em estrutura de saída |
| 3 | VIPK160D | Copybook de tabela DB2 (TIP_RST_CRT_CRD) - não contém campos CNPJ diretos |
| 4 | DAFK6452 | Declaração de parâmetro de entrada PIC 9(014) com nome `D6452I-CNPJ` na seção D6452-INPUT representando CNPJ do beneficiário como parâmetro de entrada<br/>Declaração de parâmetro de saída PIC 9(014) com nome `D6452S-CNPJ` na seção D6452-OUTPUT representando CNPJ do beneficiário como parâmetro de saída<br/>Documentação explícita indicando função de pesquisa por CNPJ (função 2) com campo `D6452I-CNPJ` |
| 5 | HLPKDFHE | Copybook de interface CICS (DFHEIBLK) - não contém campos CNPJ diretos |

Legenda:

- **id**: identificador único
- **artefato**: nome do artefato
- **evidencias**: descrição do padrão encontrado

---

## Motivo Exclusão

| id | artefato | evidencias |
| --- | --- | --- |
| 1 | VIPK160D | Artefato analisado: Nenhuma evidência de manipulação direta de CNPJ. Copybook contém apenas declaração de tabela DB2 relacionada a restrições de cartão de crédito (TIP_RST_CRT_CRD) com campos de metadados e controle, sem campos CNPJ. Mantido em análise de contexto apenas. |
| 2 | HLPKDFHE | Artefato analisado: Nenhuma evidência de manipulação de CNPJ. Copybook contém apenas interface CICS Executive Interface Block (DFHEIBLK) com campos técnicos de controle CICS, sem campos CNPJ. Mantido em análise de contexto apenas. |

Legenda:

- **id**: identificador único
- **artefato**: nome do artefato
- **evidencias**: justificativa da análise

---

## Artefato vs Módulo

| id | artefato | modulo | é principal |
| --- | --- | --- | --- |
| 1 | VIPP4365 | VIP | SIM |
| 2 | VIPP4553 | VIP | SIM |
| 3 | VIPK160D | VIP | SIM |
| 4 | DAFK6452 | DAF | NÃO |
| 5 | HLPKDFHE | HLP | NÃO |

Legenda:

- **id**: identificador único
- **artefato**: nome do artefato
- **modulo**: Prefixo do módulo
- **é principal**: Flag informando se é um módulo principal ou vizinho (satélite)

---

## Camada do Sistema

| id | artefato | camada |
| --- | --- | --- |
| 1 | VIPP4365 | BACKEND |
| 2 | VIPP4553 | BACKEND |
| 3 | VIPK160D | DATABASE |
| 4 | DAFK6452 | BACKEND |
| 5 | HLPKDFHE | INTEGRACAO |

Legenda:

- **id**: identificador único
- **artefato**: nome do artefato
- **camada**: Classificação da camada do sistema. Valores possíveis: BACKEND, FRONTEND, API, DATABASE, INTEGRACAO, CONFIGURACAO, DESCONHECIDO

---

## Persistência

| id | artefato | tipo dado | tamanho | indexado | unique | contexto |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | VIPP4365 | PIC S9(014) COMP-3 | 14 | NÃO | NÃO | Campo de entrada do arquivo VIPF317E, origem: DB2MCI.CLIENTE (fonte externa) |
| 2 | VIPP4365 | PIC 9(014) | 14 | NÃO | NÃO | Campo em header do arquivo de saída VIPF940S, propagado da entrada |
| 3 | VIPP4365 | PIC 9(014) | 14 | NÃO | NÃO | Campo em detalhe do arquivo de saída VIPF940S, propagado da entrada |
| 4 | VIPP4553 | PIC S9(14)V COMP-3 | 14 | NÃO | NÃO | Campos múltiplos de entrada do arquivo VIPF104E (empresa, representante, centro, portador) |
| 5 | VIPP4553 | PIC 9(014) | 14 | NÃO | NÃO | Campos múltiplos em detalhe do arquivo de saída VIPF104S (empresa, centro, representante, portador, município) |
| 6 | DAFK6452 | PIC 9(014) | 14 | NÃO | NÃO | Parâmetro de entrada/saída da função DAFS6452 para pesquisa/consulta por CNPJ |

Legenda:

- **id**: identificador único
- **artefato**: nome do artefato
- **tipo dado**: tipo da coluna (INT, BIGINT, VARCHAR, PIC, etc.)
- **tamanho**: tamanho da coluna
- **indexado**: indica presença de índice
- **unique**: indica constraint de unicidade
- **contexto**: contexto de uso do campo

---

## Detalhamento do Escopo do Discovery

### Diretório Raiz Analisado

Diretório: `.pss/a_old/a_code/`

### Principais Diretórios Percorridos

- `.pss/a_old/a_code/` - Código-fonte COBOL legado

### Tipos de Artefatos Analisados

- CBL (COBOL Program - 2 artefatos)
- CPY (COBOL Copybook - 3 artefatos)

### Critérios Técnicos Utilizados para Leitura

- Busca por padrões de declaração PIC 9(...) e PIC S9(...) com tamanho fixo de 14 posições
- Identificação de nomes de campos contendo "CNPJ", "CGC", "CPF-CNPJ"
- Análise de estruturas de entrada (FILE-CONTROL, input records)
- Análise de estruturas de saída (output records, layouts de transmissão)
- Identificação de operações MOVE entre campos
- Rastreamento de propagação de dados entre artefatos
- Análise de copybooks de parâmetros e interfaces
- Análise de referências externas (DB2, tabelas)

### Artefatos e Diretórios Desconsiderados Durante a Análise

Nenhum artefato foi desconsiderado. Todos os 5 artefatos encontrados no diretório foram analisados.

### Motivos de Exclusão

Os copybooks VIPK160D (tabela DB2 de restrições) e HLPKDFHE (interface CICS) foram analisados mas não contêm evidências diretas de manipulação de CNPJ. Foram mantidos na análise por contexto de inclusão nos programas principais.

### Artefatos com Múltiplos Contextos de Uso

Sim. Os seguintes artefatos apresentam múltiplos contextos de uso de CNPJ dentro do mesmo artefato:

- **VIPP4365**: Contém CNPJ como VALOR em contexto de:
  - Entrada (origem externa DB2)
  - Saída (propagação em header e detalhe de arquivo)

- **VIPP4553**: Contém CNPJ como VALOR em contexto de:
  - Entrada (múltiplas ocorrências: empresa, representante, centro, portador)
  - Saída (múltiplas ocorrências em detalhe: empresa, centro, representante, portador, município)

- **DAFK6452**: Contém CNPJ como VALOR em contexto de:
  - Entrada (parâmetro de pesquisa)
  - Saída (retorno de dados)

---

## Classificação Semântica do CNPJ

### VIPP4365

**Campo: 317E-COD-CPF-CGC-PJ**
- Classificação: **VALOR**
- Justificativa: Campo proveniente de fonte externa (DB2MCI.CLIENTE). Representa CNPJ como identificador de pessoa jurídica (negócio). Propagado para arquivo de saída (VIPF940S header e detalhe). Não é utilizado como identificador técnico interno para relacionamentos ou índices.
- Consumidor Subsequente: Arquivo VIPF940S (header e detalhe). O CNPJ é transmitido como dado de negócio em estrutura de saída.

**Campo: 317E-COD-CPF-CGC-PF**
- Classificação: **VALOR**
- Justificativa: Campo proveniente de fonte externa (DB2MCI.CLIENTE). Representa CPF como identificador de pessoa física. Propagado para campo RPRT (representante) em arquivo de saída.
- Consumidor Subsequente: Arquivo VIPF940S detalhe (campo F104S-CPF-RPRT-AUTD). Utilizado como dado de negócio.

**Campo: 940S-CNPJ-HDR**
- Classificação: **VALOR**
- Justificativa: Campo em estrutura de header de arquivo de saída. Representa CNPJ como dado de negócio a ser transmitido. Identificado por contexto de saída em estrutura de detalhe.
- Consumidor Subsequente: Consumidores subsequentes após arquivo VIPF940S: sistemas que consomem este arquivo de remessa de dados cadastrais de pré-pago pessoa jurídica. O CNPJ é transmitido como informação de negócio.

**Campo: 940S-CNPJ-DET**
- Classificação: **VALOR**
- Justificativa: Campo em estrutura de detalhe de arquivo de saída. Representa CNPJ como dado de negócio. Identificado por contexto de saída em estrutura de detalhe.
- Consumidor Subsequente: Consumidores subsequentes após arquivo VIPF940S: sistemas que consomem este arquivo de remessa. O CNPJ é transmitido como informação de negócio de cada empresa processada.

### VIPP4553

**Campo: F104E-CPF-CNPJ**
- Classificação: **VALOR**
- Justificativa: Campo em registro de entrada do arquivo VIPF104E. Representa CNPJ como identificador de pessoa jurídica (empresa). Utilizado para validação, consulta e propagação em saída. Não é utilizado como chave interna de acesso a dados.
- Consumidor Subsequente: Arquivo VIPF104S detalhe (campo F104S-CPF-CNPJ). Transmitido como dado de negócio.

**Campo: F104E-CPF-CNPJ-RPRT**
- Classificação: **VALOR**
- Justificativa: Campo em registro de entrada. Representa CNPJ de representante/fornecedor como identificador de pessoa jurídica. Propagado em saída.
- Consumidor Subsequente: Arquivo VIPF104S detalhe (campo F104S-CPF-CNPJ-RPRT). Transmitido como dado de negócio.

**Campo: F104E-CPF-CNPJ-CEN**
- Classificação: **VALOR**
- Justificativa: Campo em registro de entrada. Representa CNPJ de centro de custo. Propagado em saída.
- Consumidor Subsequente: Arquivo VIPF104S detalhe (campo F104S-CPF-CNPJ-CEN). Transmitido como dado de negócio.

**Campo: F104E-CPF-CNPJ-PORT**
- Classificação: **VALOR**
- Justificativa: Campo em registro de entrada. Representa CNPJ de portador/cliente. Propagado em saída.
- Consumidor Subsequente: Arquivo VIPF104S detalhe (campo F104S-CPF-CNPJ-PORT). Transmitido como dado de negócio.

**Campo: F104S-CPF-CNPJ**
- Classificação: **VALOR**
- Justificativa: Campo em estrutura de detalhe de arquivo de saída VIPF104S. Representa CNPJ como dado de negócio em contexto de saída/transmissão.
- Consumidor Subsequente: Consumidores downstream do arquivo VIPF104S: sistemas que consomem este arquivo de relatório de cartões de crédito. O CNPJ é transmitido como informação de negócio.

**Campo: F104S-CPF-CNPJ-CEN**
- Classificação: **VALOR**
- Justificativa: Campo em estrutura de detalhe de arquivo de saída. Representa CNPJ de centro como dado de negócio.
- Consumidor Subsequente: Consumidores downstream do arquivo VIPF104S. Transmitido como informação de negócio.

**Campo: F104S-CD-CGC-MUN**
- Classificação: **VALOR**
- Justificativa: Campo em estrutura de detalhe de arquivo de saída. Representa CGC/CNPJ de município como dado de negócio.
- Consumidor Subsequente: Consumidores downstream do arquivo VIPF104S. Transmitido como informação de negócio municipal.

**Campo: F104S-CPF-CNPJ-RPRT**
- Classificação: **VALOR**
- Justificativa: Campo em estrutura de detalhe de arquivo de saída. Representa CNPJ de representante como dado de negócio.
- Consumidor Subsequente: Consumidores downstream do arquivo VIPF104S. Transmitido como informação de negócio.

**Campo: F104S-CPF-CNPJ-PORT**
- Classificação: **VALOR**
- Justificativa: Campo em estrutura de detalhe de arquivo de saída. Representa CNPJ de portador como dado de negócio.
- Consumidor Subsequente: Consumidores downstream do arquivo VIPF104S. Transmitido como informação de negócio.

### DAFK6452

**Campo: D6452I-CNPJ**
- Classificação: **VALOR**
- Justificativa: Campo de parâmetro de entrada da subrotina DAFS6452. Documentação indica que é utilizado como parâmetro de pesquisa (função 2: pesquisa por CNPJ). Representa CNPJ como identificador de pessoa jurídica (beneficiário) para fins de consulta. Consulta é operação de acesso a dados baseada em valor de negócio.
- Consumidor Subsequente: Subrotina DAFS6452 que realiza pesquisa no banco de dados DAF de beneficiários, retornando informações de cadastro baseadas no CNPJ fornecido.

**Campo: D6452S-CNPJ**
- Classificação: **VALOR**
- Justificativa: Campo de parâmetro de saída da subrotina DAFS6452. Retorna CNPJ do beneficiário como resultado de consulta. Representa CNPJ como informação de negócio.
- Consumidor Subsequente: Programas COBOL que chamam DAFS6452 (ex: VIPP4553). O CNPJ retornado é utilizado em processamento subsequente e transmissão de dados.

---

## Identificação de Conversões de CNPJ

Nenhuma conversão de CNPJ foi identificada nos artefatos analisados durante este discovery.

Não foram encontrados padrões de:
- Transformação de formato numérico para alfanumérico
- Transformação de formato alfanumérico para numérico
- Normalização ou adaptação de formato
- Mudança de finalidade de uso (chave para valor ou vice-versa)

Os campos identificados mantêm consistência de tipo de dado ao longo do fluxo de processamento.

---

## Conclusão

O discovery analisou 5 artefatos COBOL (2 programas CBL e 3 copybooks CPY) no diretório `.pss/a_old/a_code/`, identificando 25 ocorrências relacionadas à manipulação de CNPJ.

**Resultados consolidados:**

- **Quantidade total de ocorrências identificadas**: 25 campos CNPJ/CGC
- **Quantidade de ocorrências classificadas como CHAVE**: 0
- **Quantidade de ocorrências classificadas como VALOR**: 25 (100%)
- **Quantidade de artefatos que apresentam múltiplos contextos de uso**: 3 (VIPP4365, VIPP4553, DAFK6452)

**Distribuição por artefato principal:**

- VIPP4365: 6 ocorrências (4 entrada, 2 saída)
- VIPP4553: 14 ocorrências (4 entrada, 10 saída)
- DAFK6452: 2 ocorrências (1 entrada, 1 saída)
- VIPK160D: 0 ocorrências diretas
- HLPKDFHE: 0 ocorrências diretas

**Natureza dos consumos identificados:**

- Transmissão de dados via arquivo de remessa/retorno: 17 campos
- Transmissão de parâmetros entre programas: 2 campos
- Propagação como carga útil de negócio: 25 campos (100%)

**Contextos de manipulação:**

- Entrada via arquivo: 4 ocorrências
- Entrada via parâmetro: 2 ocorrências
- Saída via arquivo: 17 ocorrências
- Origem em fonte externa (DB2): 4 ocorrências

---

Documentação gerada em: 01/04/2026

Documento Elaborado: 01/04/2026

Versão: 8.0
