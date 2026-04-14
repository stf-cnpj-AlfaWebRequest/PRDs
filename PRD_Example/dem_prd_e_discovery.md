# **Especificação — Discovery de artefatos com Manipulação de CNPJ**

## Definições

Chave: É uma chave (primária ou estrangeira) do CNPJ, e não do conteúdo do CNPJ, onde o campo de chave estrangeira em um artefato de transação, ou campo de chave primária em um artefato de consulta, etc. (ex: 78465270000165, 00000000000001). A \*\*Chave de CNPJ\*\* é um identificador técnico interno e não deve ser tratada como documento oficial de negócio.

Valor: É o conteúdo do CNPJ, ou seja, o campo é manipulado como valor de negócio, onde o campo é utilizado em regras de negócio, cálculos, validações, tela (GUI) etc. (ex: 78465270000165, 6QNJ8VY2JIC341). O \*\*Valor de CNPJ\*\* é o identificador oficial utilizado pelo negócio e por interfaces externas.

Módulos: Refere-se a qualquer artefato de código, incluindo programas COBOL e copybooks (artefatos .CPY) que são utilizados pelos programas COBOL. (ex: RF, DAF, VIP, HLP, etc.)

Módulo Principal: É o nome dado a um módulo que pertence ao mesmo sistema dos programas principais. Módulos principais podem ou não ser alterados a depender do plano de execução e execução propriamente dita, mas pertencem ao mesmo time/sistema. A identificação de módulos principais é feita dinamicamente com base no prefixo do nome do artefato. (ex: se os programas principais são VIPP4365 e VIPP4553, então qualquer artefato com prefixo VIP é considerado principal, mesmo que não requeira alteração).

Módulo Secundário: É o nome dado a um módulo que pertence a um sistema diferente dos programas principais. Módulos secundários podem ou não ser alterados a depender do plano de execução e execução propriamente dita, pois pertencem a outros times/sistemas. A identificação de módulos secundários é feita dinamicamente com base no prefixo do nome do artefato. (ex: se os programas principais são VIPP4365 e VIPP4553, então artefatos com prefixo DAF ou HLP são considerados secundários).

Artefato: Artefato é qualquer unidade técnica versionável que compõe, suporta ou influencia a execução do sistema. Isso inclui programas (ex: .cbl), copybooks (.cpy), JCLs, artefatos de dados, layouts, tabelas e demais elementos estruturais. Um artefato pode representar lógica de negócio, estrutura de dados ou configuração operacional, sendo tratado como elemento rastreável, analisável e passível de impacto dentro do ecossistema.

Programas: Refere-se ao código COBOL em si, ou seja, aos artefatos .CBL, .CPY, entre outros. Esses artefatos são os artefatos que contêm a lógica de negócio e que serão modificados (patch). No cliente, seus nomes seguem o padrão de um prefixo de 3 letras maiúsculas seguido de um número, por exemplo: VIPP4365.cbl, DAF1234.cbl, HLP5678.cbl.

Correlação 1:1: Relação obrigatória e inequívoca entre uma \*\*Chave de CNPJ\*\* e um \*\*Valor de CNPJ\*\*, preservando rastreabilidade, integridade relacional, auditabilidade e determinismo da conversão.

CGC: Denominação anterior do CNPJ (Cadastro Geral de Contribuintes), equivalente ao CNPJ para todos os efeitos desta demanda. Campos nomeados como CGC devem ser tratados com as mesmas regras aplicáveis ao CNPJ.

PF: Pessoa Física, ou seja, indivíduo, pessoa natural. Campos relacionados a PF não estão no escopo desta demanda, mas podem ser utilizados para comparação e inferência quando necessário.

PJ: Pessoa Jurídica, ou seja, empresa, organização. Campos relacionados a PJ estão no escopo desta demanda, pois o CNPJ é o identificador oficial de pessoas jurídicas.

Fonte Externa: Origem de dados pertencente a um sistema externo ao escopo desta modificação, como tabelas DB2 de outros domínios (ex: DB2MCI.CLIENTE), artefatos legados de terceiros ou integrações de entrada. Dados provenientes de fontes externas não estão no escopo de modificação desta demanda e não devem ser alterados.

Campo Indefinido: Campo CNPJ ou CGC cuja natureza semântica (Chave ou Valor) não pode ser determinada com segurança a partir dos artefatos de discovery e diagnostic — ou seja, onde a taxa de inferência for fraca. Nesses casos, o campo deve ser classificado como Indefinido e não deve ser alterado.

Serviço Central de Conversão: Serviço oficial responsável por converter \*\*Chave de CNPJ\*\* em \*\*Valor de CNPJ\*\* e \*\*Valor de CNPJ\*\* em \*\*Chave de CNPJ\*\*. É proibida a geração local ou distribuída dessa correlação fora do sistema central autorizado.

Base Central de Correlação (DFE): Estrutura central responsável por manter a relação 1:1 entre \*\*Chave de CNPJ\*\* e \*\*Valor de CNPJ\*\*, bem como metadados de controle e auditoria.

Cache: Camadas auxiliares de desempenho utilizadas para reduzir acessos à base central de correlação, podendo incluir mecanismos equivalentes a Redis, VSAM, processamento batch ou estruturas legadas compatíveis com o ambiente.

## Cenários de Classificação de Uso do CNPJ (Chave x Valor)

O termo **“Cenário”** refere-se ao papel funcional e técnico desempenhado pelo CNPJ dentro do programa ou fluxo de processamento, permitindo classificar o uso do dado em **Chave** ou **Valor**.

A classificação deve ser realizada com base na **análise contextual do uso do campo no código-fonte** e na sua função dentro da arquitetura de dados, considerando:

- O comportamento funcional do campo no processamento;
- Sua participação em operações técnicas (indexação, relacionamento, filtros);
- Sua semântica de negócio (exibição, comunicação, interface).

---

## Critérios de Classificação

### CNPJ como Chave (Identificador Técnico)

- Utilizado como identificador interno para relacionamentos ou acesso a dados;
- Participa de JOINs, MATCHs, índices, buscas, validações de integridade ou relacionamentos entre registros;
- Pode estar associado a constraints, ponteiros lógicos, chaves primárias/estrangeiras ou estruturas de lookup;
- Não representa o dado de negócio em si, mas sim um **identificador interno** usado para navegação e correlação de dados.

---

### CNPJ como Valor (Dado de Negócio)

- Representa o CNPJ como informação de negócio, utilizado para exibição, relatórios, integração externa ou regras de negócio;
- É tratado como conteúdo informacional e não como referência estrutural para acesso a dados;
- Pode ser exibido, enviado a outros sistemas ou validado segundo regras de negócio.

---

## Classificação Acumulativa

A classificação **não é excludente**.

Um mesmo campo pode atuar simultaneamente como **Chave** e como **Valor**, desde que haja evidências claras no código de que o campo cumpre ambos os papéis no fluxo de processamento.

Essa avaliação deve considerar:

- O contexto específico de uso no programa;
- O papel funcional desempenhado em cada trecho do código;
- A intenção semântica e técnica do campo no processamento.

---

## Cenários de Classificação de Uso do CNPJ (Chave x Valor)

O termo **“Cenário”** refere-se ao papel funcional e técnico desempenhado pelo CNPJ dentro do programa ou fluxo de processamento, permitindo classificar o uso do dado em **Chave** ou **Valor**.

A classificação deve ser realizada com base na **análise contextual do uso do campo no código-fonte** e na sua função dentro da arquitetura de dados, considerando:

- O comportamento funcional do campo no processamento;
- Sua participação em operações técnicas (indexação, relacionamento, filtros);
- Sua semântica de negócio (exibição, comunicação, interface).

---

## Critérios de Classificação

### CNPJ como Chave (Identificador Técnico)

- Utilizado como identificador interno para relacionamentos ou acesso a dados;
- Participa de JOINs, MATCHs, índices, buscas, validações de integridade ou relacionamentos entre registros;
- Pode estar associado a constraints, ponteiros lógicos, chaves primárias/estrangeiras ou estruturas de lookup;
- Não representa o dado de negócio em si, mas sim um **identificador interno** usado para navegação e correlação de dados.

---

### CNPJ como Valor (Dado de Negócio)

- Representa o CNPJ como informação de negócio, utilizado para exibição, relatórios, integração externa ou regras de negócio;
- É tratado como conteúdo informacional e não como referência estrutural para acesso a dados;
- Pode ser exibido, enviado a outros sistemas ou validado segundo regras de negócio.

---

## Classificação Acumulativa

A classificação **não é excludente**.

Um mesmo campo pode atuar simultaneamente como **Chave** e como **Valor**, desde que haja evidências claras no código de que o campo cumpre ambos os papéis no fluxo de processamento.

Essa avaliação deve considerar:

- O contexto específico de uso no programa;
- O papel funcional desempenhado em cada trecho do código;
- A intenção semântica e técnica do campo no processamento.

---


## **1\. Visão Geral do Discovery**

O discovery consiste em um processo automatizado de análise de código-fonte com o objetivo de **identificar, classificar e estruturar informações sobre artefatos que manipulam CNPJ**.

Essa análise considera tanto identificações explícitas quanto implícitas, incluindo padrões de uso, estruturas de dados e comportamentos associados ao dado.

O resultado do discovery permite:

* Rastreabilidade dos artefatos analisados  
* Identificação de evidências técnicas  
* Classificação do uso do CNPJ  
* Apoio à análise de impacto e refatoração

Esta seção existe apenas como definição da especificação e não deve ser reproduzida no output gerado pelo processo de discovery.

---

## **2\. Objetivo**

Esta seção define o propósito da especificação.

O objetivo é estabelecer um padrão para execução do discovery que permita:

* Identificar artefatos que manipulam CNPJ  
* Classificar como essa manipulação ocorre  
* Estruturar os resultados em tabelas padronizadas  
* Garantir consistência e auditabilidade

---

## **3\. Escopo**

Esta seção delimita o que será coberto pelo discovery.

O processo deve:

* Percorrer artefatos de forma recursiva a partir de um diretório raiz  
* Analisar o conteúdo dos artefatos  
* Identificar padrões relacionados a CNPJ  
* Registrar evidências e classificações

Não faz parte do escopo:

* Alteração de código  
* Execução de regras de negócio  
* Correção automática

Leitura da Documentação (/docs)

Extraia:

Contexto funcional

Objetivo do sistema

Regras de negócio explícitas

Glossário (se existir)

Diagramas

Cruze com o código para validar consistência

---

## **4\. Critério de Inclusão**

Esta seção define quando um artefato deve ser considerado relevante.

Um artefato deve ser incluído quando houver evidência de manipulação de CNPJ, incluindo:

* Declaração de variáveis relacionadas a CNPJ  
* Regex de validação  
* Funções de validação ou formatação  
* Literais compatíveis com estrutura de CNPJ  
* Cálculo de dígito verificador  
* Estruturas de dados contendo CNPJ  
* Uso de tamanho fixo de 14 caracteres em validações  
* Regex que aceite exclusivamente números para CNPJ  
* Conversões de CNPJ para tipo numérico (parse, cast, etc.)  
* Algoritmos de cálculo de dígito verificador  
* Operações de sanitização (remoção de máscara)  
* Máscaras de formatação (ex: 99.999.999/9999-99)  
* Campos utilizados em formulários de entrada  
* Filtros e buscas que utilizam CNPJ  
* Presença em logs e auditoria  
* Presença em relatórios

### **4.1 Detecção implícita (não explícita)**

O discovery deve identificar também casos onde o CNPJ não está nomeado diretamente.

Exemplos:

* Variáveis com nome genérico (ex: `documento`, `idFiscal`)  
* Campos identificados como CPF, mas com tamanho compatível com CNPJ  
* Strings numéricas utilizadas em validações ou persistência com comportamento compatível  
* dentificação de campos em APIs (request/response) com estrutura compatível com CNPJ  
* Identificação em artefatos de contrato (OpenAPI, Swagger, JSON Schema)  
* Identificação em artefatos de dados estruturados (CSV, XML, TXT)  
* Identificação em eventos de mensageria

Esses casos devem ser tratados como **potenciais CNPJ**, desde que haja contexto compatível.

### 4.2 Regra — Regex Numérica

#### Definição

Identifica validações, cláusulas, expressões ou rotinas que restringem o conteúdo de um campo CNPJ
a caracteres exclusivamente numéricos (`0–9`), tornando-o incompatível com o novo padrão
alfanumérico (`A–Z`, `0–9`).

---

#### Objetivo da Regra no Diagnóstico

Detectar **portões de bloqueio** — pontos do código que vão **rejeitar ativamente** um CNPJ
alfanumérico válido antes mesmo de qualquer processamento de negócio, quebrando o fluxo
silenciosamente ou gerando erro explícito de validação. A severidade é Alta quando o campo é
Valor de CNPJ porque o impacto é determinístico: não é um risco futuro, é uma quebra garantida
na entrada em vigor do novo padrão. O Diagnóstico deve nomear o artefato, transcrever o trecho,
classificar o campo semanticamente, registrar a ação necessária e rastrear toda a cadeia upstream
e downstream afetada pela validação — sem corrigir nada.

Esta regra se alinha diretamente com:

- **RF04** — Todos os pontos de uso devem ser classificados; nenhum campo pode permanecer
  semanticamente ambíguo
- **RF05** — Entradas e saídas devem aplicar regras distintas conforme o tipo (Chave ou Valor)
- **RT01** — Redefinição de estruturas físicas: `PIC 9(14)` deve evoluir para `PIC X(14)` onde
  o campo representa Valor de CNPJ
- **RT02** — Contratos de dados não podem assumir CNPJ como exclusivamente numérico

---

#### Padrões Detectados

A LLM deve identificar qualquer construção que imponha restrição numérica ao campo de CNPJ,
incluindo mas não se limitando a:

##### Validações explícitas em COBOL

```cobol
IF CNPJ-VALOR NOT NUMERIC
IF CNPJ-CHAVE IS NUMERIC
EVALUATE TRUE
  WHEN CNPJ-CAMPO NUMERIC ...
```

###### Picture Clause como validação implícita

```cobol
05 CNPJ-VALOR     PIC 9(14).
05 CGC-CAMPO      PIC 9(14) VALUE ZEROS.
```

> **Atenção:** `PIC 9(14)` só caracteriza a Regra 1 quando utilizada como **critério de
> aceitação ou rejeição** do conteúdo. Declarations puramente estruturais sem uso em
> validação devem ser analisadas sob a Regra de Tipo Numérico, não aqui.

###### Expressões regulares equivalentes (em rotinas ou copybooks)

```
^\d{14}$
[0-9]{14}
\d+
```

##### Rotinas customizadas byte a byte

```cobol
PERFORM VARYING WS-IDX FROM 1 BY 1 UNTIL WS-IDX > 14
  IF CNPJ-CAMPO(WS-IDX:1) NOT >= '0'
    OR CNPJ-CAMPO(WS-IDX:1) NOT <= '9'
      MOVE 'N' TO WS-VALIDO
  END-IF
END-PERFORM
```

---

#### Por que é um Problema

O novo CNPJ alfanumérico — com 14 posições e caracteres `A–Z0–9` — é **incompatível** com
qualquer construção que restrinja o campo a `[0-9]`.

O impacto não é potencial: é **determinístico e imediato** na entrada em vigor do novo padrão.
Qualquer CNPJ alfanumérico que passe por essa validação será **barrado ou classificado como
inválido**, podendo:

- Gerar rejeição silenciosa do registro
- Provocar erro explícito de validação
- Interromper o fluxo de negócio antes do processamento
- Contaminar o resultado de etapas downstream com dado ausente ou inválido

---

## O que o Diagnóstico deve Produzir

Ao detectar uma ocorrência desta regra, a LLM **obrigatoriamente** deve:

### 1. Identificação Nominal
Registrar o nome exato do artefato (programa `.CBL`, copybook `.CPY`, JCL, layout) onde a
validação foi encontrada, com localização técnica precisa (parágrafo, seção, linha ou contexto).

### 2. Transcrição Literal
Reproduzir o trecho exato do código — a cláusula, a condição, a rotina ou a expressão — sem
paráfrase ou abstração.

### 3. Classificação Semântica do Campo
Determinar se o campo validado atua como:

- **Chave de CNPJ** → a validação numérica pode ser aceitável, pois a Chave permanece `PIC 9(14)`
- **Valor de CNPJ** → a validação numérica é incompatível e deve ser corrigida
- **Indefinido** → inferência insuficiente; campo preservado e sinalizado

> A Regra 1 somente gera ação de correção quando o campo classificado for **Valor de CNPJ**.
> Para campos classificados como Chave, a validação numérica pode ser **legítima e preservada**.

### 4. Severidade
**Alta** — quando o campo for Valor de CNPJ. A quebra é garantida e imediata.

**Média** — quando o campo for Indefinido. O risco existe, mas a ação está bloqueada até
classificação determinística.

**Não aplicável** — quando o campo for Chave de CNPJ e a validação numérica for intencional.

### 5. Ação Registrada
Documentar a ação necessária sem executá-la:

- Substituir a expressão ou cláusula restritiva por equivalente alfanumérico: `[A-Z0-9]`,
  `PIC X(14)`, `IF campo ALPHABETIC OR NUMERIC`
- Ajustar a Picture Clause de `PIC 9(14)` para `PIC X(14)` onde o campo for Valor de CNPJ
- Revisar rotinas customizadas byte a byte para aceitar o intervalo `A–Z` além de `0–9`

### 6. Rastreabilidade Upstream e Downstream
Aplicando a **Regra Mandatória de Explicitação de Consumo Upstream e Downstream**:

**Upstream — de onde veio o dado:**
- Qual artefato produziu ou transmitiu o campo CNPJ que chega à validação
- Se o dado vem de arquivo externo, DB2, parâmetro de chamada, LINKAGE SECTION ou produção interna
- Qual formato é assumido pelo produtor upstream e se essa premissa será quebrada com o novo padrão

**Downstream — quem consome o resultado:**
- Se a validação **rejeitar** o campo, qual etapa downstream será privada do dado ou receberá
  um campo vazio, zerado ou em estado de erro
- Se a validação **aceitar** o campo (antes da correção), qual consumidor downstream receberá
  um dado potencialmente incorreto
- Identificar nominalmente: programas chamados, steps seguintes de JCL, transações CICS,
  rotinas batch, arquivos de saída, tabelas DB2, mensagens MQ, APIs ou sistemas satélites

---

#### O que o Diagnóstico NÃO deve fazer

| Proibição | Justificativa |
|---|---|
| Corrigir o código | O Diagnóstico apenas documenta; a correção pertence ao Plano de Execução |
| Assumir que toda `PIC 9(14)` é validação | A Picture Clause pode ser declaração estrutural sem uso em critério de aceitação |
| Generalizar para todos os campos numéricos | A regra se aplica **exclusivamente** ao campo de CNPJ no contexto analisado |
| Classificar sem evidência | Campos sem inferência determinística devem ser marcados como Indefinido |
| Sugerir refatoração estrutural | A demanda é intervenção pontual; não autoriza reorganização de código |

---

##### Critério de Bloqueio

Se o campo onde a Regra 1 foi detectada estiver classificado como **Indefinido** (RF10 / RT14),
a ação de atualização da expressão **não pode ser registrada como mandatória**. O campo deve ser
sinalizado como risco potencial com ação bloqueada, aguardando evidência adicional que permita
classificação determinística.
---

## **5\. Premissas**

Esta seção define condições que devem ser assumidas como verdadeiras durante o processo de construção do discovery..

* O processo deve ser determinístico  
* A análise deve ser baseada em conteúdo  
* A identificação de linguagem deve ser feita pela extensão do artefato  
* artefatos podem conter múltiplas evidências  
* Um artefato pode gerar registros em múltiplas tabelas  
* O processo deve ser não intrusivo  
* Caminhos devem ser relativos à raiz  
* Suporte a múltiplas linguagens sem configuração prévia  
* Assume-se ausência de código já adaptado para CNPJ alfanumérico  
* Não será realizada validação das rotinas de obtenção/validação de CNPJ identificadas durante o discovery, no contexto desta PoC  
* Para fins de análise, será considerada a existência de dois cenários de uso do CNPJ:  
* CNPJ como chave  
* CNPJ como valor  
* Não deve ser considerada a identificação de CNPJ como chave ou valor em artefatos pertencentes a módulos classificados como não principais  
* Estaremos aplicando as modificações dos artefatos do módulo principal, sem modificar os artefatos de módulos vizinhos, por entender que estes serão modificados durante a rodada de execução das mudanças do módulo vizinho em questão.  
* Não devem ser realizadas análises que impliquem alteração ou interpretação funcional de módulos externos: (ex: DAF, HLP ou equivalentes)  
* Módulos externos deverão ficar na tabela de exclusão  
* Módulos principais são artefatos COBOL.

---

## **6\. Estrutura das Tabelas**

### 6.1 Discovery Consolidado

Esta tabela deve ser obrigatoriamente a primeira tabela a ser exibida, precedendo todas as demais tabelas.

**Descrição**

Esta tabela representa uma visão **quantitativa e consolidada** do discovery, permitindo avaliar:

* Volume de artefatos analisados por tipo  
* Quantidade de artefatos impactados  
* Intensidade de evidências relacionadas a CNPJ

---

**Campos da Tabela**

Campos:

* id  
* tipo do artefato  
* linguagem de programação  
* quantidade de artefatos analisados  
* quantidade de artefatos com ocorrência  
* quantidade de evidências  
* percentual de artefatos com ocorrência

---

**Legenda dos Campos**

* **id**: identificador único do registro  
* **tipo do artefato**: extensão do artefato (ex: JAVA, TS, SQL)  
* **linguagem de programação**: linguagem associada ao tipo  
* **quantidade de artefatos analisados**: total de artefatos daquele tipo percorridos  
* **quantidade de artefatos com ocorrência**: artefatos que possuem ao menos uma evidência  
* **quantidade de evidências**: total de evidências registradas  
* **percentual de artefatos com ocorrência**:  
  (artefatos com ocorrência / artefatos analisados) \* 100

---

**Exemplo**

| id | tipo do artefato | linguagem de programação | quantidade de artefatos analisados | quantidade de artefatos com ocorrência | quantidade de evidências | percentual de artefatos com ocorrência |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| 1 | JAVA | JAVA | 120 | 45 | 98 | 37,5% |
| 2 | TS | TYPESCRIPT | 80 | 30 | 60 | 37,5% |
| 3 | SQL | SQL | 40 | 20 | 45 | 50% |
| 4 | CBL | COBOL | 25 | 10 | 18 | 40% |
| 5 | TXT | UNKNOWN | 15 | 2 | 2 | 13,3% |

---

**Considerações Técnicas**

* Todos os dados devem ser derivados das tabelas após essa, portanto mesmo sendo a primeira a ser exibida, deve ser a última a ser gerada.  
* Não deve conter dados redundantes ou recalculáveis sem necessidade

---

### **6.2 Características dos artefatos**

Armazena os metadados dos artefatos analisados.

Campos:

* id  
* artefato  
* linguagem de programação  
* tipo do artefato


**Legenda:**

* **id**: identificador único e sequencial  
* **artefato**: nome do artefato  
* **linguagem de programação**: identificada pela extensão  
* **tipo do artefato**: extensão normalizada

---

### **6.2 Evidências**

Armazena os indícios que justificam a inclusão do artefato.

Campos:

* id  
* artefato  
* evidencias


**Legenda:**

* **id**: identificador único  
* **artefato**: nome do artefato  
* **evidencias**: descrição do padrão encontrado

As evidências devem permitir inferir o contexto de uso do CNPJ, possibilitando sua classificação como chave ou valor.

Sempre que possível, devem indicar:

* Se o CNPJ está sendo utilizado como identificador  
* Se o CNPJ está sendo utilizado como dado de negócio  
* Se há indícios de transformação entre usos

---

### **6.3 Motivo Exclusão**

Registra artefatos analisados que não foram considerados relevantes.

Campos:

* id  
* artefato  
* evidencias


**Legenda:**

* **id**: identificador único  
* **artefato**: nome do artefato  
* **evidencias**: justificativa da não inclusão


  
---

### 6.4 Artefato vs Módulo:

Campos:

* id:  
* artefato  
* modulo  
* é principal

Legenda:

* **id**: identificador único  
* **artefato**: nome do artefato  
* **modulo**: Prefixo do módulo  
* **é principa**l: Flag informando se é um módulo principal ou vizinho (satélite)

---

### **6.5 Camada do Sistema**

Classifica em qual camada arquitetural o artefato está inserido.

Campos:

* id  
* artefato  
* camada

Legenda:

* **id**: identificador único  
* **artefato**: nome do artefato  
* **camada**: Classificação da camada do sistema. Valores possíveis:  
  * BACKEND  
  * FRONTEND  
  * API  
  * DATABASE  
  * INTEGRACAO  
  * CONFIGURACAO  
  * DESCONHECIDO

---

### **6.6 Persistência**

Identifica características de armazenamento do CNPJ.

Campos:

* id  
* artefato  
* tipo\_dado  
* tamanho  
* indexado  
* unique

Legenda:

* **id**: identificador único  
* **artefato**: nome do artefato  
* tipo\_dado: tipo da coluna (INT, BIGINT, VARCHAR, etc.)  
* tamanho: tamanho da coluna  
* indexado: indica presença de índice  
* unique: indica constraint de unicidade

---

## **7\. Detalhamento do Escopo do Discovery**

Esta seção deve documentar **onde o discovery foi executado e quais artefatos foram considerados no processo**.

Deve conter:

* Diretório raiz analisado  
* Lista ou descrição dos principais diretórios percorridos  
* Tipos de artefatos analisados (ex: .java, .ts, .sql, etc.)  
* Critérios técnicos utilizados para leitura dos artefatos

Além disso, deve::

* Registrar artefatos ou diretórios desconsiderados durante a análise  
* Registrar motivos de exclusão (ex: irrelevância, ausência de conteúdo analisável, padrão não compatível)  
* Indicar se foram encontrados artefatos onde o CNPJ apresenta múltiplos contextos de uso dentro do mesmo artefato.

Essa seção garante transparência e rastreabilidade do escopo real do discovery executado.

---

## **8\. Detecção de Linguagem e Tipo**

A identificação deve ser feita com base na extensão do artefato:

* Remover o ponto (`.`)  
* Converter para maiúsculo

artefatos sem extensão:

* linguagem de programação: UNKNOWN  
* tipo do artefato: UNKNOWN

---

## **9\. Pipeline**

O pipeline de execução não precisa ser documentado nesta especificação.

---

## **10\. Requisitos**

* Determinismo  
* Performance  
* Auditabilidade

---

## **11\. Rodapé**

Ao final do documento deve ser incluído e levar em consideração o dia atual para dd/mm/yyyy:

Documentação gerada em:

Documento Elaborado: dd/mm/yyyy

Versão: 8.0

---

## **12\. Resumo**

Esta especificação define um discovery:

* Baseado em análise semântica (explícita e implícita)  
* Com múltiplas tabelas de rastreabilidade  
* Com classificação detalhada de uso de CNPJ  
* Independente de linguagem  
* Com transparência do escopo analisado  
* Com controle formal de versão

Esta seção não deve ser incluída no resultado do discovery, sendo utilizada apenas como referência desta especificação.

---

## **13\. Restrições de Conteúdo Gerado**

### 13.1 Formatação

* Não deve ser utilizado nenhum tipo de emoji  
* A documentação deve manter padrão técnico e formal  
* Colunas com nomes compostos devem ser escritas com espaço entre as palavras, sem utilização de underscore (\_)

### 13\. 2 Tabelas

* As evidências devem ser descritas de forma estruturada, utilizando quebra de linha para separar cada ocorrência identificada dentro do artefato. Não deve ser gerado bloco único de texto contínuo.  
* Cada tabela deve ter uma parte de legenda seguindo o seguinte padrão:

Legenda:

- campo: descrição

### 13.2 Conteúdo

O resultado do discovery deve ser estritamente técnico, estruturado e baseado em dados extraídos da análise dos artefatos.

Não deve ser gerado conteúdo descritivo, narrativo ou interpretativo além do necessário para suporte às tabelas e rastreabilidade.

No início de cada seção, deve haver um parágrafo curto, explicando o que a seção vai abordar.

Em nenhuma hipótese deve exibir o caminho dos artefatos.

Fica explicitamente proibida a geração das seguintes seções no output do discovery:

* Visão Geral  
* Resumo Executivo  
* Introduções descritivas  
* Explicações conceituais não derivadas diretamente dos dados analisados

O output deve conter apenas:

* Estruturas de dados (tabelas)  
* Critérios de Inclusão  
* Premissas  
* Conclusão

Qualquer conteúdo fora desse padrão deve ser considerado inválido.

---

## 14\. Conclusão

Ao final do discovery deve ser gerada uma conclusão objetiva baseada exclusivamente nos dados coletados durante a análise.

A conclusão deve:

* Resumir o que foi feito com base no outputs

A conclusão deve ser:

* Técnica  
* Objetiva

A conclusão deve incluir:

* Quantidade de ocorrências classificadas como chave  
* Quantidade de ocorrências classificadas como valor  
* Quantidade de artefatos que apresentam múltiplos contextos de uso

Não deve conter:

* Explicações conceituais  
* Repetição de conteúdo já detalhado nas tabelas  
* Recomendações de implementação

---

## 15\. Padronização de Identificação de Funções

Para fins de padronização do output do discovery, funções relacionadas à obtenção ou validação de CNPJ não devem utilizar seus nomes reais definidos no código.

Deve ser aplicada a seguinte regra de abstração:

**Plataforma Baixa (ex: COBOL)**

* Nome padrão a ser utilizado:  
    
  **"Função de Obtenção/Validação do CNPJ em Plataforma Baixa"**  
    
* Não deve ser utilizado o nome real da função (ex: getXxxByXxx ou similares)

---

**Plataforma Alta (ex: sistemas modernos)**

* Nome padrão a ser utilizado:  
    
  **"Função de Obtenção/Validação do CNPJ em Plataforma Alta"**  
    
* Não deve ser utilizado o nome real da função (ex: SISXXXX ou similares)

---

Essa padronização é obrigatória e tem como objetivo:

* Evitar exposição de nomenclatura técnica específica  
* Garantir uniformidade no resultado  
* Facilitar análise independente de tecnologia

---

## 16\. Classificação Semântica do CNPJ

Todo CNPJ identificado no discovery deve ser classificado de acordo com seu papel no sistema.

A classificação obrigatória é:

* CHAVE  
* VALOR

---

**Definições operacionais:**

* **CHAVE de CNPJ**  
  * Utilizado como identificador em estruturas de dados  
  * Presente em chaves primárias, estrangeiras ou índices  
  * Utilizado para associação entre entidades  
* **VALOR de CNPJ**  
  * Utilizado como dado de negócio  
  * Presente em campos de entrada, saída, relatórios e integrações  
  * Representa o CNPJ como informação consumível

---

**Regra obrigatória**

Todo uso identificado de CNPJ deve ser classificado como CHAVE ou VALOR.

Não é permitido:

* Não classificar  
* Classificação genérica

---

## 17\. Identificação de Conversões de CNPJ

O discovery deve identificar pontos onde ocorre transformação entre diferentes representações de CNPJ.

---

**Considerar como conversão:**

* Transformação de dado utilizado como VALOR para uso como CHAVE  
* Transformação de dado utilizado como CHAVE para uso como VALOR  
* Normalização ou adaptação de formato com mudança de finalidade

---

**O discovery deve registrar:**

* Existência de conversão  
* artefato onde ocorre  
* Contexto de uso (entrada, processamento, persistência)

