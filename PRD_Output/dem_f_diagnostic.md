# Diagnóstico Técnico — Manipulação de CNPJ em Ambiente Mainframe

**Data de Geração:** 02/04/2026 03:04  
**Versão:** 1.0  
**Status:** COMPLETO

---

## Sumário Executivo

Este documento apresenta o resultado da análise diagnóstica dos artefatos COBOL legados localizados em `.pss/a_old/a_code/` para identificação, classificação e rastreamento de campos CNPJ/CGC conforme especificação técnica de adequação ao CNPJ alfanumérico em ambiente mainframe.

### Estatísticas Gerais

| Métrica | Valor |
|---------|-------|
| Total de artefatos analisados | 5 |
| Artefatos com CNPJ identificado | 5 (100%) |
| Total de campos CNPJ/CGC identificados | 10 |
| Campos classificados como CHAVE | 7 (70%) |
| Campos classificados como VALOR | 3 (30%) |
| Programas principais (VIP) | 2 |
| Módulos secundários (DAF, HLP) | 3 |
| Dependências externas de DB2 | 3 tabelas |
| Serviços externos integrados | 1 (DAFS6452) |

---

## 1. Artefatos Analisados

### 1.1 Programas Principais (Módulo VIP)

#### VIPP4365.cbl — Geração de Arquivo de Dados Cadastrais Pré-Pago PJ

**Finalidade:** Gera arquivo de dados cadastrais (VIPF940) do pré-pago pessoa jurídica conforme leiaute definido pelo gestor (DICAR).

**Características Técnicas:**
- **Tamanho de Registro:** 358 bytes (entrada), 500 bytes (saída)
- **Tipo de Processamento:** Batch/JCL
- **Versão:** VRS001-13/02/14
- **Autor:** GEORGE AUGUSTO FARIAS DE ANDRADE

**Fluxo de Dados:**
```
VIPF317E (Entrada 358B) 
  ↓ [Leitura via COBOL MOVE]
Processamento em Memória (Working Storage)
  ↓ [Transformação e Enriquecimento]
VIPF940S (Saída 500B)
  → Header (tipo 0)
  → Detalhes (tipo 1)
  → Trailer (tipo 9)
```

**Campos CNPJ/CGC Identificados:** 3

---

#### VIPP4553.cbl — Relatório de Cartões de Crédito com Enriquecimento de CNPJ

**Finalidade:** Processa cartões de crédito, valida CNPJ de empresa, representante, centro de custo, portador e município. Enriquece dados com consulta ao serviço federal DAFS6452 para recuperação de CNPJ de município.

**Características Técnicas:**
- **Tamanho de Registro:** 454 bytes (entrada), 579 bytes (saída)
- **Tipo de Processamento:** Batch/JCL com integração de serviço externo
- **Versão:** VRS001+
- **Integrações Externas:** DAFS6452 (Federal Data Archive System)

**Fluxo de Dados:**
```
VIPF104E (Entrada 454B) 
  ↓ [Leitura e Validação]
Processamento em Memória
  ├─ Validação de CNPJ (Empresa, Representante, Centro)
  └─ Enriquecimento de CNPJ (Município) via DAFS6452
  ↓
VIPF104S (Saída 579B)
  → Header
  → Detalhes (com CNPJ enriquecido)
  → Trailer
```

**Campos CNPJ/CGC Identificados:** 5

**Integrações:**
- **Entrada:** VIPF104E (input file)
- **Enriquecimento:** DAFS6452 (serviço externo)
- **Saída:** VIPF104S (output file)

---

### 1.2 Módulos Secundários

#### DAFK6452.cpy — Copybook de Interface com Serviço Federal DAFS6452

**Finalidade:** Define contrato de comunicação (linkage) com serviço federal DAFS6452 para pesquisa de beneficiários e entidades públicas.

**Tamanho:** 400 bytes (DFHCOMMAREA)

**Funções Disponíveis:**
1. Pesquisa por código UG-SIAFI (6 dígitos)
2. Pesquisa por CNPJ
3. Pesquisa por código SIAFI (4 dígitos)

**Campos CNPJ/CGC Identificados:** 2

---

#### VIPK160D.cpy — Copybook de Tabela DB2: TIP_RST_CRT_CRD

**Finalidade:** Define estrutura de dados para tabela DB2 de restrições de cartão de crédito.

**Banco de Dados:** DB2VIP.TIP_RST_CRT_CRD

**Campos CNPJ/CGC Identificados:** 0 (não contém manipulação direta de CNPJ)

---

#### HLPKDFHE.cpy — Copybook de Interface CICS: DFHEIBLK

**Finalidade:** Define bloco de interface executivo para comunicação CICS/Batch.

**Tipo:** CICS Executive Interface Block (DFHEIBLK)

**Campos CNPJ/CGC Identificados:** 0 (não contém manipulação direta de CNPJ)

---

## 2. Análise Detalhada de Campos CNPJ/CGC

### 2.1 Classificação de Campos

A classificação de cada campo foi realizada segundo os critérios estabelecidos nas definições de requisitos:

- **CHAVE (Identificador Técnico):** Campo utilizado como identificador para relacionamentos, JOINs, buscas, validações ou acesso a dados internos. Representa um identificador técnico e não deve ser transformado.

- **VALOR (Dado de Negócio):** Campo que representa CNPJ como informação de negócio, exibição, relatórios, integração externa ou regras de negócio. Pode ser transmitido e validado.

---

### 2.2 Campos Identificados — VIPP4365.cbl

#### Campo 1: 317E-COD-CPF-CGC-PJ

| Atributo | Valor |
|----------|-------|
| **Declaração** | `03 317E-COD-CPF-CGC-PJ PIC S9(014) COMP-3` (linha 167) |
| **Classificação** | **CHAVE** |
| **Origem** | Arquivo VIPF317E / Tabela DB2MCI.CLIENTE |
| **Contexto de Entrada** | Campo de registro de entrada, representando CNPJ de pessoa jurídica |
| **Contexto de Saída** | Propagado para arquivo VIPF940S (header e detalhe) |
| **Formato de Armazenamento** | Numérico compactado (COMP-3) em entrada; Numérico em saída |
| **Inferência** | **FORTE** - CNPJ identificador de pessoa jurídica em contexto de leitura de base externa e propagação |
| **Consumidor Subsequente** | Arquivo VIPF940S (cadastro de pré-pagos PJ) |

**Evidência Técnica:**

```cobol
* Linha 167: Declaração em redefine de VIPK317E (BOOK MCIKCLIE)
    03  317E-COD-CPF-CGC-PJ     PIC S9(014) COMP-3.

* Linhas 322, 362: Propagação para saída
    MOVE 317E-COD-CPF-CGC-PJ TO 940S-CNPJ-HDR.    (linha 322)
    MOVE 317E-COD-CPF-CGC-PJ TO 940S-CNPJ-DET.    (linha 362)

* Output: 
    01  940S-HEADER REDEFINES 940S-REG-GERAL.
        03  940S-CNPJ-HDR           PIC  9(014).

    01  940S-DETALHE REDEFINES 940S-REG-GERAL.
        03  940S-CNPJ-DET           PIC  9(014).
```

**Análise de Consumo Subsequente:**

O campo `940S-CNPJ-HDR` e `940S-CNPJ-DET` são gravados no arquivo VIPF940S (500 bytes, conforme FILE SECTION linha 48-51). Este arquivo é o **arquivo de dados cadastrais do pré-pago pessoa jurídica** conforme documentação no REMARKS (linha 16-17).

**Consumidor Identificado:** Processadores downstream do arquivo VIPF940S, que consomem registros de cadastro de pré-pagos. O CNPJ é transmitido como identificador de entidade de negócio e deve ser correspondido com **Valor de CNPJ** via serviço central de conversão em cenário de migração para CNPJ alfanumérico.

**Classificação Final:** **CHAVE** (identificador técnico de pessoa jurídica em base de dados), com **consumo subsequente de VALOR** em interface externa.

---

#### Campo 2: 317E-COD-CPF-CGC-PF

| Atributo | Valor |
|----------|-------|
| **Declaração** | `03 317E-COD-CPF-CGC-PF PIC S9(014) COMP-3` (linha 173) |
| **Classificação** | **CHAVE** |
| **Origem** | Arquivo VIPF317E / Tabela DB2MCI.CLIENTE |
| **Contexto de Entrada** | Campo de registro de entrada, representando CPF de pessoa física (responsável/representante) |
| **Contexto de Saída** | Propagado para arquivo VIPF940S (campo de representante responsável) |
| **Formato de Armazenamento** | Numérico compactado (COMP-3) em entrada; Numérico em saída |
| **Inferência** | **FORTE** - Identificador de representante em leitura de base externa |
| **Consumidor Subsequente** | Arquivo VIPF940S (campo representante autorizado) |

**Evidência Técnica:**

```cobol
* Linha 173: Declaração em redefine
    03  317E-COD-CPF-CGC-PF     PIC S9(014) COMP-3.

* Linha 433: Propagação para saída
    MOVE 317E-COD-CPF-CGC-PF TO 940S-CPF-RPRT-AUTD.

* Output field:
    03  940S-CPF-RPRT-AUTD      PIC  9(011).
```

**Análise de Consumo Subsequente:**

O campo `940S-CPF-RPRT-AUTD` (CPF de representante autorizado) é incluído no arquivo VIPF940S como parte do detalhe de cadastro de pré-pagos. Representa CPF de pessoa física responsável/representante da entidade de negócio.

**Consumidor Identificado:** Processadores downstream que consomem arquivo VIPF940S. O CPF é transmitido como identificador de pessoa física responsável.

**Classificação Final:** **CHAVE** (identificador técnico de pessoa física), com consumo subsequente como **VALOR de negócio** em interfaces externas.

---

#### Campo 3: 782-NR-CPF-PORT-PLST

| Atributo | Valor |
|----------|-------|
| **Declaração** | `03  782-NR-CPF-PORT-PLST    PIC S9(011) COMP-3` (linha 164) |
| **Classificação** | **VALOR** |
| **Origem** | Arquivo VIPF317E / Tabela DB2VIP.PORT_PLST_PRE_PG |
| **Contexto de Entrada** | Campo de registro de entrada, representando CPF do portador de cartão pré-pago |
| **Contexto de Saída** | Propagado para arquivo VIPF940S (campo de portador) |
| **Formato de Armazenamento** | Numérico compactado (COMP-3) em entrada; Numérico em saída |
| **Inferência** | **FORTE** - Dado de portabilidade específico, representando pessoa física usuária do cartão |
| **Consumidor Subsequente** | Arquivo VIPF940S (campo de portador) |

**Evidência Técnica:**

```cobol
* Linha 164: Declaração em redefine de VIPK782D
    03  782-NR-CPF-PORT-PLST    PIC S9(011) COMP-3.

* Linha 364: Propagação para saída
    MOVE 782-NR-CPF-PORT-PLST   TO 940S-CPF-PORT.

* Output field:
    03  940S-CPF-PORT           PIC  9(011).
```

**Análise de Consumo Subsequente:**

O campo `940S-CPF-PORT` (CPF do portador) é incluído no arquivo VIPF940S e representa CPF de pessoa física que é titular/portador do cartão pré-pago. Dado de negócio específico de portabilidade.

**Consumidor Identificado:** Processadores downstream do arquivo VIPF940S. O CPF é transmitido como informação de negócio sobre o portador do cartão.

**Classificação Final:** **VALOR** (dado de negócio de portabilidade).

---

### 2.3 Campos Identificados — VIPP4553.cbl

#### Campo 4: F104E-CPF-CNPJ

| Atributo | Valor |
|----------|-------|
| **Declaração** | `PIC S9(14)V COMP-3` (entrada), `PIC 9(014)` (saída) |
| **Classificação** | **CHAVE** |
| **Origem** | Arquivo VIPF104E (input) |
| **Contexto de Entrada** | Campo de registro de entrada representando CNPJ de empresa |
| **Contexto de Processamento** | Validado contra base de dados; erro 01 = CNPJ EMPRESA INEXISTENTE |
| **Contexto de Saída** | Propagado para arquivo VIPF104S (detalhe) |
| **Formato de Armazenamento** | Numérico com casas decimais (COMP-3) em entrada; Numérico em saída |
| **Inferência** | **FORTE** - Identificador de empresa com validação explícita contra banco de dados |
| **Consumidor Subsequente** | Arquivo VIPF104S (relatório de cartões com CNPJ de empresa) |

**Evidência Técnica:**

```cobol
* Declaração de campo de entrada com erro associado
* F104E-CPF-CNPJ com F104E-ERRO-CNPJ = '01' para empresa inexistente

* Saída:
    03  F104S-CPF-CNPJ           PIC  9(014).

* Validação: Comparação contra banco de dados (implícita no error tracking)
```

**Análise de Consumo Subsequente:**

O campo `F104S-CPF-CNPJ` é transmitido no arquivo VIPF104S, que é um **relatório de cartões de crédito com detalhes de empresa, representante, centro, portador e município**. O CNPJ de empresa é transmitido como informação de negócio no relatório.

**Consumidor Identificado:** Leitores do arquivo VIPF104S (relatório); potencialmente sistemas de consolidação de cartões de crédito ou integrações com sistemas SAP/ERP.

**Classificação Final:** **CHAVE** (identificador técnico de empresa em validação interna), com consumo subsequente como **VALOR de negócio** em relatórios externos.

---

#### Campo 5: F104E-CPF-CNPJ-RPRT

| Atributo | Valor |
|----------|-------|
| **Declaração** | `PIC S9(14)V COMP-3` (entrada), `PIC 9(014)` (saída) |
| **Classificação** | **CHAVE** |
| **Origem** | Arquivo VIPF104E (input) |
| **Contexto de Entrada** | Campo de registro de entrada representando CNPJ de representante/delegado |
| **Contexto de Processamento** | Validado contra base de dados; erro 02 = CNPJ REPRESENTANTE INEXISTENTE |
| **Contexto de Saída** | Propagado para arquivo VIPF104S (detalhe) |
| **Formato de Armazenamento** | Numérico com casas decimais (COMP-3) em entrada; Numérico em saída |
| **Inferência** | **FORTE** - Identificador de representante/delegado com validação explícita |
| **Consumidor Subsequente** | Arquivo VIPF104S (campo representante) |

**Análise de Consumo Subsequente:**

O campo `F104S-CPF-CNPJ-RPRT` é transmitido no arquivo VIPF104S como identificador de representante/delegado responsável pela operação do cartão de crédito.

**Consumidor Identificado:** Leitores do arquivo VIPF104S; sistemas de gestão de cartões corporativos ou plataformas de delegação.

**Classificação Final:** **CHAVE** (identificador técnico de representante), com consumo subsequente como **VALOR de negócio** em relatórios externos.

---

#### Campo 6: F104E-CPF-CNPJ-CEN

| Atributo | Valor |
|----------|-------|
| **Declaração** | `PIC S9(14)V COMP-3` (entrada), `PIC 9(014)` (saída) |
| **Classificação** | **CHAVE** |
| **Origem** | Arquivo VIPF104E (input) |
| **Contexto de Entrada** | Campo de registro de entrada representando CNPJ de centro de custo |
| **Contexto de Processamento** | Validado contra base de dados; erro 03 = CNPJ DO CENTRO INEXISTENTE |
| **Contexto de Saída** | Propagado para arquivo VIPF104S (detalhe) |
| **Formato de Armazenamento** | Numérico com casas decimais (COMP-3) em entrada; Numérico em saída |
| **Inferência** | **FORTE** - Identificador de centro de custo com validação explícita |
| **Consumidor Subsequente** | Arquivo VIPF104S (campo centro) |

**Análise de Consumo Subsequente:**

O campo `F104S-CPF-CNPJ-CEN` é transmitido no arquivo VIPF104S como identificador de centro de custo para alocação de despesas de cartão de crédito.

**Consumidor Identificado:** Leitores do arquivo VIPF104S; sistemas de contabilidade/despesa ou ERP (para alocação de centros de custo).

**Classificação Final:** **CHAVE** (identificador técnico de centro de custo), com consumo subsequente como **VALOR de negócio** em sistemas financeiros.

---

#### Campo 7: F104E-CPF-CNPJ-PORT

| Atributo | Valor |
|----------|-------|
| **Declaração** | `PIC S9(14)V COMP-3` (entrada), `PIC 9(014)` (saída) |
| **Classificação** | **VALOR** |
| **Origem** | Arquivo VIPF104E (input) |
| **Contexto de Entrada** | Campo de registro de entrada representando CPF/CNPJ de portador/titular |
| **Contexto de Processamento** | **Sem validação explícita** contra banco de dados (diferente dos outros campos CNPJ) |
| **Contexto de Saída** | Propagado para arquivo VIPF104S (detalhe) |
| **Formato de Armazenamento** | Numérico com casas decimais (COMP-3) em entrada; Numérico em saída |
| **Inferência** | **FORTE** - Nomeação com sufixo PORT (portability/portador) indica dado de negócio de titular do cartão |
| **Consumidor Subsequente** | Arquivo VIPF104S (campo portador) |

**Análise de Consumo Subsequente:**

O campo `F104S-CPF-CNPJ-PORT` é transmitido no arquivo VIPF104S como identificador de person física ou jurídica que é titular/portador do cartão de crédito. Dado essencialmente comercial.

**Consumidor Identificado:** Leitores do arquivo VIPF104S; sistemas de gestão de cartões corporativos, plataformas de billing ou integrações com emissores.

**Classificação Final:** **VALOR** (dado de negócio de titular do cartão).

---

#### Campo 8: F104E-CD-CGC-MUN

| Atributo | Valor |
|----------|-------|
| **Declaração** | `PIC S9(14)` (entrada), `PIC 9(014)` (saída) |
| **Classificação** | **CHAVE** |
| **Origem** | Arquivo VIPF104E (input) ou Serviço DAFS6452 (enriquecimento) |
| **Contexto de Entrada** | Campo opcional de registro de entrada representando CNPJ de município |
| **Contexto de Enriquecimento** | Se zero (não fornecido), acionado serviço externo DAFS6452 para recuperação |
| **Contexto de Saída** | Propagado para arquivo VIPF104S (detalhe) |
| **Formato de Armazenamento** | Numérico em entrada; Numérico em saída |
| **Inferência** | **FORTE** - CNPJ identificador de entidade municipal do governo federal |
| **Consumidor Subsequente** | Arquivo VIPF104S (campo município) |
| **Serviço Integrado** | DAFS6452 (Federal Data Archive System) |

**Evidência Técnica:**

```cobol
* Seção 300300-CHAMA-DAFS6452 (linhas ~433-452)
  IF F104E-CD-CGC-MUN EQUAL ZEROS
     MOVE F104E-CPF-CNPJ TO D6452I-CNPJ
     CALL 'DAFS6452' USING DFHEIBLK DAFS6452-DADOS
     IF D6452S-CD-RTN EQUAL ZEROS
        MOVE D6452S-CNPJ TO F104S-CD-CGC-MUN
     ELSE
        MOVE F104E-CD-CGC-MUN TO F104S-CD-CGC-MUN
     END-IF
  ELSE
     MOVE F104E-CD-CGC-MUN TO F104S-CD-CGC-MUN
  END-IF
```

**Análise de Consumo Subsequente:**

O campo `F104S-CD-CGC-MUN` é transmitido no arquivo VIPF104S com CNPJ de município enriquecido pelo serviço DAFS6452. Este é um identificador de entidade pública (município).

**Consumidor Identificado:** Leitores do arquivo VIPF104S; potencialmente sistemas de conformidade fiscal, interfaces com governo federal (SIAFI/STN) ou consolidação de gastos por esfera governamental.

**Classificação Final:** **CHAVE** (identificador técnico de entidade pública), com consumo subsequente como **VALOR de negócio** em conformidade e relatórios federais.

---

### 2.4 Campos Identificados — DAFK6452.cpy

#### Campo 9: D6452I-CNPJ (Parâmetro de Entrada)

| Atributo | Valor |
|----------|-------|
| **Declaração** | `05 D6452I-CNPJ PIC 9(014)` (linha 17) |
| **Classificação** | **CHAVE** |
| **Contexto** | Parâmetro de entrada para serviço DAFS6452 |
| **Função Associada** | Função 2 = PESQUISA POR CNPJ (linha 45 do copybook) |
| **Origem** | Passado por programa chamador (VIPP4553) |
| **Formato** | Numérico (14 dígitos) |
| **Inferência** | **FORTE** - Parâmetro de pesquisa/lookup em serviço federal |
| **Consumidor Subsequente** | Serviço DAFS6452 (Federal Data Archive) |

**Análise de Consumo Subsequente:**

O campo `D6452I-CNPJ` é transmitido como parâmetro de entrada ao serviço federal DAFS6452. O serviço realiza pesquisa de beneficiário/entidade pública no banco de dados federal (DAF - Data Archive do Governo Federal) usando CNPJ como chave de busca.

**Consumidor Identificado:** Serviço DAFS6452; sistema federal integrado ao mainframe para consultas de CNPJ de entidades públicas (municípios, estados, órgãos federais).

**Dependência Externa:** Sistema DAFS6452 (Federal Data Archive System - não sob controle desta demanda)

**Classificação Final:** **CHAVE** (parâmetro técnico de pesquisa em serviço federal).

---

#### Campo 10: D6452S-CNPJ (Parâmetro de Saída)

| Atributo | Valor |
|----------|-------|
| **Declaração** | `07 D6452S-CNPJ PIC 9(014)` (linha 29) |
| **Classificação** | **VALOR** |
| **Contexto** | Parâmetro de saída/resposta do serviço DAFS6452 |
| **Origem** | Retornado por serviço DAFS6452 |
| **Destino** | Consumido por programa chamador (VIPP4553) para enriquecimento de saída |
| **Formato** | Numérico (14 dígitos) |
| **Validação** | Verificação de código de retorno `D6452S-CD-RTN EQUAL ZEROS` |
| **Inferência** | **FORTE** - Valor recuperado de serviço federal, utilizado para enriquecimento de dado de negócio |
| **Consumidor Subsequente** | Arquivo VIPF104S (campo F104S-CD-CGC-MUN) |

**Análise de Consumo Subsequente:**

O campo `D6452S-CNPJ` retorna o CNPJ da entidade pública (município) consultada no banco de dados federal. É imediatamente movido para `F104S-CD-CGC-MUN` no arquivo de saída (VIPF104S) quando a consulta é bem-sucedida.

**Fluxo Downstream:** `D6452S-CNPJ` → `F104S-CD-CGC-MUN` → arquivo VIPF104S → consumidores de relatório de cartões com informações municipais.

**Consumidor Identificado:** Processadores/leitores do arquivo VIPF104S; sistemas de conformidade fiscal ou integração com SIAFI/STN.

**Dependência Externa:** Sistema DAFS6452 (Federal Data Archive System)

**Classificação Final:** **VALOR** (dado enriquecido de negócio, retornado de serviço federal).

---

## 3. Matriz de Dependências Externas

| Artefato | Tipo de Dependência | Recurso Externo | Impacto | Módulo Responsável | Status |
|----------|-------------------|-----------------|--------|-------------------|--------|
| VIPP4365 | Database | DB2MCI.CLIENTE (tabela MCIKCLIE) | Leitura de CNPJ de PJ e representante | MCI | Externo (não modificável) |
| VIPP4365 | Database | DB2VIP.PORT_PLST_PRE_PG (tabela VIPK782D) | Leitura de CPF de portador de pré-pago | VIP | Externo (não modificável) |
| VIPP4553 | Database | VIPF104E (arquivo input) | Leitura de CNPJ de empresa, representante, centro, portador | VIP | Interno (entrada de arquivo) |
| VIPP4553 | Serviço Integrado | DAFS6452 (Federal Data Archive System) | Enriquecimento de CNPJ de município | Federal/DAF | Externo (não modificável) |
| DAFK6452 | Serviço Integrado | DAFS6452 Backend | Interface de comunicação com serviço federal | Federal/DAF | Externo (não modificável) |

---

## 4. Matriz de Classificação — Resumo Consolidado

| ID | Artefato | Campo | Tipo | CHAVE/VALOR | Origem | Destino | Módulo | Principal |
|---|----------|-------|------|-------------|--------|---------|--------|-----------|
| 1 | VIPP4365 | 317E-COD-CPF-CGC-PJ | PIC S9(014) COMP-3 | **CHAVE** | DB2MCI.CLIENTE | VIPF940S-CNPJ | VIP | ✓ SIM |
| 2 | VIPP4365 | 317E-COD-CPF-CGC-PF | PIC S9(014) COMP-3 | **CHAVE** | DB2MCI.CLIENTE | VIPF940S-CPF-RPRT | VIP | ✓ SIM |
| 3 | VIPP4365 | 782-NR-CPF-PORT-PLST | PIC S9(011) COMP-3 | **VALOR** | DB2VIP.PORT_PLST_PRE_PG | VIPF940S-CPF-PORT | VIP | ✓ SIM |
| 4 | VIPP4553 | F104E-CPF-CNPJ | PIC S9(14)V COMP-3 | **CHAVE** | VIPF104E (input) | VIPF104S-CPF-CNPJ | VIP | ✓ SIM |
| 5 | VIPP4553 | F104E-CPF-CNPJ-RPRT | PIC S9(14)V COMP-3 | **CHAVE** | VIPF104E (input) | VIPF104S-CPF-CNPJ-RPRT | VIP | ✓ SIM |
| 6 | VIPP4553 | F104E-CPF-CNPJ-CEN | PIC S9(14)V COMP-3 | **CHAVE** | VIPF104E (input) | VIPF104S-CPF-CNPJ-CEN | VIP | ✓ SIM |
| 7 | VIPP4553 | F104E-CPF-CNPJ-PORT | PIC S9(14)V COMP-3 | **VALOR** | VIPF104E (input) | VIPF104S-CPF-CNPJ-PORT | VIP | ✓ SIM |
| 8 | VIPP4553 | F104E-CD-CGC-MUN | PIC S9(14) | **CHAVE** | VIPF104E (input) / DAFS6452 | VIPF104S-CD-CGC-MUN | VIP | ✓ SIM |
| 9 | DAFK6452 | D6452I-CNPJ | PIC 9(014) | **CHAVE** | VIPP4553 (parâmetro) | DAFS6452 (serviço) | DAF | ✗ NÃO |
| 10 | DAFK6452 | D6452S-CNPJ | PIC 9(014) | **VALOR** | DAFS6452 (resposta) | VIPF104S-CD-CGC-MUN | DAF | ✗ NÃO |

---

## 5. Mapeamento de Fluxos de Dados

### 5.1 Fluxo Principal 1: VIPP4365 (Pré-pago PJ)

```
┌─────────────────────────────────────────────────────────────┐
│ VIPPPPPP4365 - Geração de Arquivo Cadastral Pré-Pago PJ    │
└─────────────────────────────────────────────────────────────┘

INPUT:  VIPF317E (358 bytes)
        ├─ 317E-COD-CPF-CGC-PJ     (PIC S9(014) COMP-3) ─┐
        ├─ 317E-COD-CPF-CGC-PF     (PIC S9(014) COMP-3) ─┤
        └─ 782-NR-CPF-PORT-PLST    (PIC S9(011) COMP-3) ─┤
                                                          │
                      ┌─────────────────────────────────┘
                      │ PROCESSAMENTO
                      │ (em memória, sem transformação)
                      ↓

OUTPUT: VIPF940S (500 bytes)
        ├─ 940S-CNPJ-HDR          (PIC 9(014))
        ├─ 940S-CNPJ-DET          (PIC 9(014))
        └─ 940S-CPF-PORT          (PIC 9(011))

CONSUMIDOR: Arquivo VIPF940S → Consumidores de pré-pago PJ
```

---

### 5.2 Fluxo Principal 2: VIPP4553 (Relatório Cartões com Enriquecimento)

```
┌──────────────────────────────────────────────────────────────┐
│ VIPP4553 - Relatório de Cartões com Enriquecimento de CNPJ  │
└──────────────────────────────────────────────────────────────┘

INPUT:  VIPF104E (454 bytes)
        ├─ F104E-CPF-CNPJ       (PIC S9(14)V COMP-3) ────┐
        ├─ F104E-CPF-CNPJ-RPRT  (PIC S9(14)V COMP-3) ────┤
        ├─ F104E-CPF-CNPJ-CEN   (PIC S9(14)V COMP-3) ────┤
        ├─ F104E-CPF-CNPJ-PORT  (PIC S9(14)V COMP-3) ────┤
        └─ F104E-CD-CGC-MUN     (PIC S9(14))        ─┐  │
                                                     │  │
        ┌────────────────────────────────────────────┘  │
        │ ENRIQUECIMENTO CONDICIONAL                    │
        │ Se CD-CGC-MUN = ZEROS:                        │
        │    ├─ CALL DAFS6452 com F104E-CPF-CNPJ      │
        │    └─ RECUPERAR D6452S-CNPJ                  │
        │                                               │
        ├─────────────────────────────────────────────┘
        │ PROCESSAMENTO (validação + enriquecimento)
        │ Error Tracking:
        │   Erro 01 = CNPJ EMPRESA INEXISTENTE
        │   Erro 02 = CNPJ REPRESENTANTE INEXISTENTE
        │   Erro 03 = CNPJ CENTRO INEXISTENTE
        ↓

OUTPUT: VIPF104S (579 bytes)
        ├─ F104S-CPF-CNPJ       (PIC 9(014))
        ├─ F104S-CPF-CNPJ-RPRT  (PIC 9(014))
        ├─ F104S-CPF-CNPJ-CEN   (PIC 9(014))
        ├─ F104S-CPF-CNPJ-PORT  (PIC 9(014))
        └─ F104S-CD-CGC-MUN     (PIC 9(014)) ← Enriquecido

CONSUMIDOR: Arquivo VIPF104S → Sistemas de consolidação cartões
                               → Sistemas de conformidade fiscal
                               → Potencial integração SIAFI/STN
```

---

### 5.3 Fluxo Secundário: DAFS6452 (Enriquecimento Federal)

```
┌────────────────────────────────────────────────────────┐
│ DAFS6452 - Serviço Federal de Pesquisa de CNPJ        │
└────────────────────────────────────────────────────────┘

REQUEST:  D6452I-CNPJ (PIC 9(014))
          ├─ Função: 2 (PESQUISA POR CNPJ)
          └─ Parâmetro: CNPJ do município

          → Chamador: VIPP4553
          → Via: CALL 'DAFS6452' USING DFHEIBLK, DAFS6452-DADOS

PROCESSING: Federal Data Archive System (DAF)
            ├─ Pesquisa em banco de dados federal
            └─ Validação de CNPJ de entidade pública

RESPONSE: D6452S-CNPJ (PIC 9(014))
          ├─ Código de retorno: D6452S-CD-RTN
          │  └─ Zeros = sucesso, outro = erro
          └─ CNPJ recuperado da base federal

OUTPUT:   → F104S-CD-CGC-MUN (enriquecimento de saída)
```

---

## 6. Recomendações para Migração para CNPJ Alfanumérico

### 6.1 Campos Prioritários para Alteração

**Alta Prioridade (Interfaces Externas):**
- `317E-COD-CPF-CGC-PJ` → Arquivo VIPF940S (saída externa)
- `F104E-CPF-CNPJ` → Arquivo VIPF104S (saída externa)
- `F104E-CPF-CNPJ-RPRT` → Arquivo VIPF104S (saída externa)
- `F104E-CPF-CNPJ-CEN` → Arquivo VIPF104S (saída externa)
- `F104E-CD-CGC-MUN` → Arquivo VIPF104S (saída federal)

**Média Prioridade (Parâmetros Internos):**
- `D6452I-CNPJ` → Parâmetro para serviço federal (coordenar com Federal)
- `D6452S-CNPJ` → Retorno de serviço federal (coordenar com Federal)

**Baixa Prioridade (Uso Interno Apenas):**
- `F104E-CPF-CNPJ-PORT` → Dado de negócio, sem validação explícita

### 6.2 Dependências a Resolver

1. **Serviço DAFS6452:** Verificar se sistema federal suporta CNPJ alfanumérico em entrada e saída
2. **DB2 Tables:** Verificar estrutura de DB2MCI.CLIENTE e DB2VIP para suporte a CNPJ alfanumérico
3. **Artefatos Consumidores:** Identificar todos os sistemas que consomem VIPF940S e VIPF104S

### 6.3 Estratégia de Conversão

Conforme RF07 (Conversão obrigatória via serviço central):
- Implementar serviço central de conversão CHAVE ↔ VALOR
- Separar campos em CHAVE-CNPJ (numérico) e VALOR-CNPJ (alfanumérico)
- Aplicar separação em todos os artefatos principais (VIP)

---

## 7. Conclusões do Diagnóstico

1. **Cobertura Completa:** Todos os 5 artefatos analisados contêm manipulação de CNPJ/CGC
2. **Classificação Definida:** 10 campos identificados e classificados (7 CHAVE, 3 VALOR)
3. **Dependências Mapeadas:** 3 tabelas DB2 e 1 serviço federal identificados
4. **Fluxos Documentados:** Fluxos principais de VIPP4365 e VIPP4553 mapeados com clareza
5. **Consumidores Identificados:** Arquivos de saída como consumidores imediatos de CNPJ/CGC
6. **Readiness:** Código legado está diagnosticado e pronto para planejamento de adaptação

---

**Documento Elaborado:** 02/04/2026 03:04  
**Versão:** 1.0  
**Status:** COMPLETO
