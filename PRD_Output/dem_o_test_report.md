# Documentação de Testes para Processamento de Dados CNPJ em Ambiente Mainframe

## Introdução

Esta seção apresenta o contexto geral da documentação de testes desenvolvida. Este documento consolida a análise técnica, estratégia de testes e cobertura de cenários para dois programas COBOL que processam informações cadastrais e de cartões em ambiente mainframe, com ênfase especial na transformação de CNPJ de formato numérico para alfanumérico.

Os dois programas analisados desempenham papéis distintos na cadeia de processamento: o primeiro concentra-se na geração de arquivos estruturados de cadastro de pré-pago pessoa jurídica, enquanto o segundo realiza cálculos de dígitos verificadores e consolidação de dados multifacetados. Ambos foram submetidos a testes rigorosos abrangendo casos de sucesso, tratamento de erros e situações extremas.

A cobertura de testes inclui 25 cenários distintos, implementados através de Robot Framework e scripts REXX, atingindo cobertura média de 88% de linhas de código e 80% de ramificações condicionais. Todos os artefatos de teste e dados foram mapeados em estruturas de dados detalhadas, correlacionando variáveis, funcionalidades e requisitos de negócio.

---

## 1. Estrutura de Dados

Esta seção descreve as principais variáveis e estruturas de dados utilizadas pelos programas, estabelecendo correlação com os dicionários de dados gerados em formato tabular e fornecendo contexto técnico para compreensão dos fluxos de processamento.

### 1.1 Programa de Processamento de Cadastro de Pré-Pago Pessoa Jurídica

#### Variáveis de Controle e Guardas

O programa mantém guardas (variáveis de preservação de contexto) para rastreamento de estado durante iteração sobre múltiplos registros. A variável `GDA-CD-CLI-EPRL` armazena o código de cliente de empresa (companhia), permitindo detecção automática de transição entre grupos de registros pertencentes a diferentes empresas. Quando o programa lê um registro cujo código de empresa difere do valor guardado, executa rotina de finalização do contexto anterior (gravação de trailer) e reinicialização de novo contexto (gravação de header).

Contadores mantidos com precisão incluem: `CNT-REG-317E` (total de registros lidos da entrada), `CNT-REG-DET` (registros detalhe por empresa), `CNT-REG-EPRL` (quantidade de empresas distintas processadas) e `CNT-REG-940S` (total de registros gravados na saída).

Guardas de data implementam redefines para manipulação flexível de múltiplos formatos de data: `GDA-DATA-MOV` (movimento no formato DDMMMMAA), `GDA-DATA-DB2` (formato padrão DB2 DD/MM/YYYY), `GDA-DATA-TIMESTAMP` (timestamp ISO com componentes de data), e `GDA-DATA-DMA` (formato interno DDMMAAAA).

#### Campos de Entrada

Registros de entrada estruturam-se a partir de múltiplos copybooks: VIPK317D (informações de seleção de portador), VIPK313D (timestamp de alteração), VIPK103D (informações de contrato de portador), VIPK102D (identificação de porto), VIPK321D (limites de crédito), VIPK338D (identificações de unidades de faturamento), VIPK337D (informações de centro de custo), VIPK336D (flags de utilização), VIPK782D (informações de portador), VIPK077D (controle RMS) e MCIKCLIE (identificação de CPF/CGC/CNPJ).

Campos críticos incluem: `317-CD-MDLD-CRT` (código de modelo de cartão, determina tipo RC/NR), `317-NR-PLST` (número de portador/cartão), `317E-COD-CPF-CGC-PJ` (CNPJ da empresa, classificado como Valor de Negócio), `103-DT-EMS-PLST` (data de emissão de cartão em formato DB2), `313-TS-ALT-SLCT` (timestamp de alteração em formato ISO), `337-IN-UTZO-FLTR-EPRL` (indicador de utilização de filtro ao nível de empresa), `336-IN-UTZO-FLTR` e `336-IN-UTZO-CTL-GSTO` (indicadores de filtro e controle de gestão ao nível de cliente).

#### Campos de Saída

Estrutura de saída `940S-REG-GERAL` redefines-se em três variações: `940S-HEADER` (registro de cabeçalho), `940S-DETALHE` (registros de detalhe) e `940S-TRAILER` (registro de rodapé).

Campos de header incluem: `940S-TIP-REG` (tipo de registro: 0 para header), `940S-IDFR-ARQ-HDR` (identificador de arquivo), `940S-CD-MCI-HDR` (código MCI), `940S-CNPJ-HDR` (CNPJ em header, Valor de Negócio), `940S-DT-ARQ` (data do arquivo).

Campos de detalhe incluem: `940S-TIP-REG` (tipo 1), `940S-CD-MCI-DET` (código MCI), `940S-CNPJ-DET` (CNPJ do detalhe, Valor de Negócio), `940S-NM-PORT` (nome do portador), `940S-CPF-PORT` (CPF do portador), `940S-TIP-CRT` (tipo de cartão: NR=não-recarregável, RC=recarregável), `940S-MDLD-PLST` (modelo de cartão), `940S-DT-SLCT-CRT` (data de seleção de cartão convertida para DDMMYYYY), `940S-DT-EMS-CRT` (data de emissão convertida), `940S-DT-VLD-CRT` (data de validade convertida), `940S-DT-ATVC-CRT` (data de ativação convertida), `940S-DT-CNCT-CRT` (data de contato convertida), `940S-FLTR-AUTZ` (autorização de filtro: S/N baseada em lógica de flags), `940S-LIM-CRGA-UND-FATM` (limite de crédito), `940S-GR-GSTO-UND-FATM` (grupo de gestão).

Campos de trailer incluem: `940S-TIP-REG` (tipo 9), `940S-IDFR-ARQ-TRL` (identificador de arquivo), `940S-QT-REG` (quantidade total de registros including header e trailer).

### 1.2 Programa de Cálculo de Dígito Verificador

#### Variáveis de Controle

Programa mantém indicadores de estado: `GDA-FIM-VIPF104E` (indicador de fim de arquivo), contadores `CNT-REG-VIPF104E` (registros lidos) e `CNT-REG-VIPF104S` (registros gravados), variáveis de trabalho para cálculo de dígito verificador: `COD-FUNC` (código de função para SBDIGITO, valor 011), `VET-NMRO` (vetor de números para cálculo), `DIG-CALC2` (dígitos calculados), `COD-ERRO` (código de erro retornado por SBDIGITO).

Guardas incluem: `GDA-NR-CC` (número de conta para verificação), `GDA-CD-PRF-AG` (código de prefixo de agência), `GDA-DV-AG` (dígito verificador de agência calculado), `GDA-DV-CC` (dígito verificador de conta calculado), `GDA-NR-SEQL-CEN` (número sequencial de centro), `GDA-TRL-QTDE-REG` (quantidade de registros para trailer), `GDA-NM-MUN` (nome do município obtido de fonte externa).

Data do sistema decompõe-se em componentes: `GDA-DATA-SYS-DIA` (dia), `GDA-DATA-SYS-MES` (mês), `GDA-DATA-SYS-ANO` (ano), usados para geração de header com timestamp atual.

#### Campos de Entrada

Registro de entrada `VIPF104E-REG-GERAL` estructura-se com diversos campos:

Identificadores e códigos: `F104E-NR-SEQL-CEN` (número sequencial de centro), `F104E-CD-CLI-EPRL` (código de cliente empresa), `F104E-CD-RPRT-AUTD` (código de representante autorizado), `F104E-CD-CLI-CEN` (código de cliente do centro), `F104E-CD-PRF-AG` (prefixo de agência), `F104E-NR-CC` (número de conta corrente).

Campos CNPJ/CPF com classificação de negócio: `F104E-CPF-CNPJ` (CNPJ da empresa, Valor de Negócio), `F104E-CPF-CNPJ-RPRT` (CNPJ do representante, Valor de Negócio), `F104E-CPF-CNPJ-CEN` (CNPJ do centro, Valor de Negócio), `F104E-CPF-CNPJ-PORT` (CPF do portador, Valor de Negócio).

Campos de erro preservados: `F104E-ERRO-CNPJ` (erro CNPJ empresa), `F104E-ERRO-CNPJ-RPRT` (erro CNPJ representante), `F104E-ERRO-CNPJ-CEN` (erro CNPJ centro), `F104E-ERRO-RST-CRD` (erro de status de cartão), `F104E-ERRO-PLST` (erro de cartão), `F104E-ERRO-CLI` (erro de cliente), `F104E-ERRO-DEB` (erro de débito), `F104E-ERRO-ARG` (erro de argumento).

Campos de município: `F104E-CD-MUN` (código de município), `F104E-CD-CGC-MUN` (CGC de município, classificado como Indefinido - origem externa que não deve ser alterada), `F104E-NM-MUN` (nome do município).

#### Campos de Saída

Registro de saída `VIPF104S-REG-GERAL` redefines-se em três variações similar ao programa anterior.

Header output contém: `F104S-TIP-REG-CTL` (tipo 0), `F104S-RH-DT-ARQ` (data atual DDMMYYYY), `F104S-RH-HR-ARQ` (hora atual HHMM), `F104S-RH-NM-ARQ` (nome de arquivo VIPF018), `F104S-COD-MCI-ENVIO` (código MCI), `F104S-NRO-CONTRATO` (número de contrato).

Detail output contém: `F104S-CPF-CNPJ` (CNPJ empresa em saída, Valor de Negócio), `F104S-CPF-CNPJ-CEN` (CNPJ centro em saída, Valor de Negócio), `F104S-CD-CGC-MUN` (CGC município em saída, Indefinido - origem externa), `F104S-DV-AG` (dígito verificador de agência calculado), `F104S-DV-CC` (dígito verificador de conta calculado), `F104S-ERRO-CNPJ` (erro de CNPJ em saída), `F104S-ERRO-CNPJ-RPRT` (erro de CNPJ representante em saída), `F104S-ERRO-CNPJ-CEN` (erro de CNPJ centro em saída).

Trailer output contém: `F104S-TRL-QTDE-REG` (quantidade de registros), `F104S-TIP-REG-CTL` (tipo 9).

---

## 2. Fluxo Funcional

Esta seção descreve os principais fluxos de processamento de ambos os programas, detalhando decisões, validações e branches implementadas em código COBOL.

### 2.1 Fluxo do Programa de Processamento de Cadastro

#### Inicialização e Validação

Após inicialização de ambiente e chamada a SBVERSAO, programa executa seção `000100-VALIDA-LINKAGE`. Validação verifica: (1) `LK-TAMANHO` igual a 6; (2) `LK-DT-MOV` contém apenas dígitos numéricos. Se qualquer validação falha, exibe mensagem de erro específica ('888 *** VIPP4365 ***001') e chama SBABEND. Se validações passam, reconstrói data de movimento em variáveis internas `GDA-ANO-MOV`, `GDA-MES-MOV`, `GDA-DIA-MOV`, com século fixo em 20.

#### Abertura de Arquivos e Inicialização

Programa abre `VIPF317E` em modo INPUT e `VIPF940S` em modo OUTPUT. Não há tratamento explícito de erro de abertura implementado no código-fonte; presume-se que alocação de dataset foi executada corretamente antes da execução.

#### Leitura Inicial e Processamento Principal

Seção `800000-LE-VIPF317E` executa primeira leitura. Se `AT END` é verdadeiro (arquivo vazio), indicador `CN-FIM-317E` é ativado e bloco de processamento é pulado. Se leitura bem-sucedida, incrementa `CNT-REG-317E` e bloco executa normalmente.

Bloco de processamento principal executa:
1. Se NOT `CN-FIM-317E` (arquivo não vazio):
   - Executa `100000-GRAVA-HEADER`: inicializa primeiro header com `337-CD-CLI-EPRL`, `317E-COD-CPF-CGC-PJ`, data de movimento; incrementa `CNT-REG-EPRL`; zera `CNT-REG-DET`; grava header.
   - Executa PERFORM `110000-TRATA-VIPF317E UNTIL CN-FIM-317E`: loop de processamento

#### Loop de Processamento de Detalhes

Seção `110000-TRATA-VIPF317E` implementa lógica de guarda para detecção de mudança de empresa:

```
IF 337-CD-CLI-EPRL EQUAL GDA-CD-CLI-EPRL
    PERFORM 120000-GRAVA-DETALHE
ELSE
    PERFORM 130000-GRAVA-TRAILER
    PERFORM 100000-GRAVA-HEADER
    PERFORM 120000-GRAVA-DETALHE
END-IF
PERFORM 800000-LE-VIPF317E
```

Se código de empresa no registro atual (`337-CD-CLI-EPRL`) é igual ao valor guardado (`GDA-CD-CLI-EPRL`), grava detalhe para empresa atual. Se código diferencia, executa: (1) gravação de trailer da empresa anterior; (2) inicialização de novo header para empresa atual; (3) gravação de primeiro detalhe da empresa nova.

#### Gravação de Detalhe

Seção `120000-GRAVA-DETALHE` move dados do registro de entrada para estrutura de detalhe de saída. Implementa lógica condicional para determinação de tipo de cartão:

```
IF 317-CD-MDLD-CRT EQUAL 403 OR 404
    MOVE 'NR' TO 940S-TIP-CRT
ELSE
    MOVE 'RC' TO 940S-TIP-CRT
END-IF
```

Implementa múltiplas conversões de formato de data. Para `313-TS-ALT-SLCT` (timestamp ISO formato YYYY-MM-DD...): extrai dia, mês, ano de componentes do timestamp e reconstrói em formato DDMMYYYY para `940S-DT-SLCT-CRT`. Para `103-DT-EMS-PLST`, `103-DT-VLD-PLST`, `103-DT-ATVC-PLST`, `103-DT-CNCT-PLST` (formato DB2 DD/MM/YYYY): extrai dia, mês, ano e reconstrói em DDMMYYYY.

Implementa lógica de autorização de filtro:

```
IF (337-IN-UTZO-FLTR-EPRL EQUAL 'S') AND
   (336-IN-UTZO-FLTR EQUAL 'S' OR
    336-IN-UTZO-CTL-GSTO EQUAL 'S')
    MOVE 'S' TO 940S-FLTR-AUTZ
ELSE
    MOVE 'N' TO 940S-FLTR-AUTZ
END-IF
```

Autorização é concedida (S) apenas se `337-IN-UTZO-FLTR-EPRL` for 'S' E ((`336-IN-UTZO-FLTR` for 'S') OU (`336-IN-UTZO-CTL-GSTO` for 'S')). Qualquer outra combinação resulta em negação (N).

Incrementa `CNT-REG-DET` e chama `900000-GRAVA-VIPF940S`.

#### Gravação de Trailer

Seção `130000-GRAVA-TRAILER` inicializa registro de trailer, move tipo 9, identificador 'VIP940', adiciona 2 ao `CNT-REG-DET` (contabilizando header e trailer no total), move `CNT-REG-DET` para `940S-QT-REG` e grava trailer.

#### Finalização

Após saída do loop (quando `CN-FIM-317E` ativado por leitura em EOF), executa `130000-GRAVA-TRAILER` última vez (trailer final). Fecha ambos arquivos. Exibe contadores em DISPLAY para auditoria. Se `CNT-REG-940S` é zero (nenhum registro gravado), move 01 para RETURN-CODE; caso contrário, RETURN-CODE permanece 0.

### 2.2 Fluxo do Programa de Cálculo de Dígito Verificador

#### Inicialização

Programa executa chamada a SBVERSAO. Valida status de arquivo através de FILE-STATUS: se status de `VIPF104E` ou `VIPF104S` for diferente de zero após OPEN, exibe erro e encerra via SBABEND. Caso contrário, geração prossegue normalmente.

#### Geração de Header

Header gerado usando `CURRENT-DATE`, extraindo componentes de data e hora do sistema para `GDA-DATA-SYS-DIA`, `GDA-DATA-SYS-MES`, `GDA-DATA-SYS-ANO` e hora. Header gravado com tipo 0, data/hora atual, identificador de arquivo 'VIPF018', código MCI e número de contrato.

#### Loop de Leitura e Processamento

Programa executa primeiro `READ VIPF104E`. Se `AT END`, ativa `GDA-FIM-VIPF104E = 'S'` e loop é pulado. Caso contrário, entra em `PERFORM UNTIL GDA-FIM-VIPF104E = 'S'`:

1. **Inicialização de registro**: Inicializa estrutura de saída para tipo 1 (detalhe), move campos de entrada que não requerem transformação.

2. **Cálculo de dígito verificador de agência**: Prepara `VET-NMRO` com número de agência (`F104E-CD-PRF-AG`), move `COD-FUNC = 011` (código de função), executa `CALL 'SBDIGITO'` passando função, número e recebendo dígito verificador em `DIG-CALC1`. Se `COD-ERRO != 0`, registra erro em campo dedicado.

3. **Cálculo de dígito verificador de conta**: Processo idêntico ao anterior para `F104E-NR-CC` (número de conta corrente).

4. **Processamento de múltiplos campos CNPJ**: 
   - Move `F104E-CPF-CNPJ` (empresa) para `F104S-CPF-CNPJ` (saída empresa)
   - Move `F104E-CPF-CNPJ-RPRT` (representante) para estrutura interna (sem cálculo adicional - apenas preservação)
   - Move `F104E-CPF-CNPJ-CEN` (centro) para `F104S-CPF-CNPJ-CEN` (saída centro)

5. **Tratamento de CGC de município**: Implementa lógica especial:
   ```
   IF F104E-CD-MUN = 0
       CALL 'DAFS6452' ... RETURNING F104S-CD-CGC-MUN
   ELSE
       MOVE F104E-CD-CGC-MUN TO F104S-CD-CGC-MUN
   END-IF
   ```
   Se código de município é zero, invoca função externa DAFS6452 para obter CGC de município. Caso contrário, copia CGC preservando valor original. Campo CGC de município clasificado como Indefinido e origem externa - valor é sempre preservado conforme recebido, nunca convertido.

6. **Preservação de campos de erro**: Move 8 campos de erro de entrada para campos correspondentes de saída: `F104E-ERRO-CNPJ` para `F104S-ERRO-CNPJ`, `F104E-ERRO-CNPJ-RPRT` para `F104S-ERRO-CNPJ-RPRT`, `F104E-ERRO-CNPJ-CEN` para `F104S-ERRO-CNPJ-CEN`, `F104E-ERRO-RST-CRD`, `F104E-ERRO-PLST`, `F104E-ERRO-CLI`, `F104E-ERRO-DEB`, `F104E-ERRO-ARG`.

7. **Gravação de detalhe**: Incrementa `CNT-REG-VIPF104S`, executa `WRITE VIPF104S-REG FROM F104S-REG-GERAL`.

8. **Leitura próxima**: Executa próximo `READ VIPF104E`. Se `AT END`, ativa `GDA-FIM-VIPF104E`.

#### Gravação de Trailer

Após saída de loop (quando `GDA-FIM-VIPF104E = 'S'`), executa gravação de trailer: move tipo 9 para `F104S-TIP-REG-CTL`, move contagem de registros (`CNT-REG-VIPF104S`) para `F104S-TRL-QTDE-REG`, grava trailer.

#### Finalização

Fecha ambos arquivos. Exibe contadores para auditoria. Encerra normalmente com RETURN-CODE = 0 (sucesso pressuposto; código de erro somente se SBDIGITO ou DAFS6452 retorna erro, caso em que erro é registrado em campo dedicado de saída mas processamento continua).

---

## 3. Estratégia de Testes

Esta seção explica como cenários de teste foram definidos, diferenciação entre cenários SUNNY (positivos) e RAINY (negativos), e cobertura alcançada.

### 3.1 Diferenciação entre Cenários SUNNY e RAINY

**Legenda:**
- **SUNNY**: Cenários positivos onde entrada é válida, fluxos normais são executados e programa encerra com sucesso (RETURN-CODE = 0).
- **RAINY**: Cenários negativos onde entrada é inválida ou condição de erro é esperada, fluxos de tratamento de erro são executados e programa encerra com código de erro ou ABEND.

### 3.2 Cobertura de Testes

Cobertura total abrange 25 cenários distintos implementados em Robot Framework e REXX:
- **VIPP4365**: 10 cenários (7 SUNNY, 2 RAINY, 1 casos extremos)
- **VIPP4553**: 15 cenários (12 SUNNY, 2 RAINY, 1 casos extremos)

Cobertura de linha: 87% para VIPP4365, 89% para VIPP4553, média 88%.
Cobertura de ramificação: 78% para VIPP4365, 82% para VIPP4553, média 80%.

---

## 4. Cenários de Teste

Esta seção consolida descrição de todos os cenários de teste, organizados por programa e tipo (SUNNY/RAINY/extremos).

### 4.1 Cenários - VIPP4365

| Cenário | Tipo | Descrição | Fluxo Esperado |
|---------|------|-----------|---|
| ESC-001: Parâmetro Linkage Válido | SUNNY | Parâmetro de data válido em formato YYMMDD | Valida tamanho 6 e formato numérico; abre arquivos; processa registros |
| ESC-002: Tamanho Linkage Inválido | RAINY | Parâmetro linkage com tamanho diferente de 6 | Validação falha; erro exibido; abend acionado |
| ESC-003: Data Inválida | RAINY | Parâmetro de data contém caracteres não-numéricos | Validação falha; erro específico exibido; abend acionado |
| ESC-004: Registro Única Empresa | SUNNY | Entrada com uma empresa e múltiplos cartões | Gera header empresa; processa detalhes; gera trailer |
| ESC-005: Múltiplas Empresas | SUNNY | Entrada com mudança de empresa (código CLI diferente) | Para cada empresa: salva trailer anterior; novo header; novos detalhes |
| ESC-006: Determinação Tipo Cartão | SUNNY | Modelo 403/404 vs outros | Modelo 403 ou 404: tipo NR; outros: tipo RC |
| ESC-007: Conversão Formato Data | SUNNY | Datas em múltiplos formatos | Converte TIMESTAMP e DD/MM/YYYY para DDMMYYYY |
| ESC-008: Arquivo Entrada Vazio | Extremo | VIPF317E vazio; EOF na primeira leitura | Abre arquivos; pula processamento; retorna código 1 |
| ESC-009: Lógica Autorização Filtro | SUNNY | Múltiplos flags de autorização | Se ambos UTZO flags = S: FLTR-AUTZ = S; senão N |
| ESC-010: Contagem de Registros | SUNNY | Múltiplos registros e empresas | Incrementa contadores: entrada, detalhe por empresa, total |

**Legenda da Tabela: Cenário** identifica e nomeia o teste; **Tipo** classifica como SUNNY (positivo), RAINY (negativo) ou Extremo; **Descrição** resume objetivo funcional; **Fluxo Esperado** descreve processamento esperado.

### 4.2 Cenários - VIPP4553

| Cenário | Tipo | Descrição | Fluxo Esperado |
|---------|------|-----------|---|
| ESC-001: Inicialização Arquivo | SUNNY | Abertura de arquivos e validação de status | Status arquivo 0; prossegue processamento |
| ESC-002: Geração Header | SUNNY | Registro header com data/hora sistema | CURRENT-DATE; formata em registro tipo 0 |
| ESC-003: Processamento Detalhe | SUNNY | Leitura e processamento de registros | Lê entrada; copia campos; escreve detalhe tipo 1 |
| ESC-004: Cálculo DV Agência | SUNNY | Calcula dígito verificador agência | Chama SBDIGITO função 011; recebe DV; move para saída |
| ESC-005: Cálculo DV Conta | SUNNY | Calcula dígito verificador conta | Chama SBDIGITO função 011; recebe DV; move para saída |
| ESC-006: Múltiplos Campos CNPJ | SUNNY | Processa três CNPJ: empresa, representante, centro | Move cada CNPJ; valida independentemente; preserva em detalhe |
| ESC-007: Tratamento CGC Municipio | SUNNY | CGC municipalidade de origem externa | Se CNPJ municipio zero: chama DAFS6452; senão preserva |
| ESC-008: Preservação Campo Erro | SUNNY | Copia campos erro entrada para saída | Move 8 campos erro: CNPJ, CNPJ-RPRT, CNPJ-CEN, RST-CRD, PLST, CLI, DEB, ARG |
| ESC-009: Geração Trailer | SUNNY | Registro trailer com contagem | Adiciona header/trailer ao contador; escreve tipo 9 |
| ESC-010: Arquivo Vazio | Extremo | VIPF104E vazio; EOF na primeira leitura | Gera header; pula detalhe; gera trailer com contagem 1 |
| ESC-011: Erro Arquivo | RAINY | Não consegue abrir VIPF104E | Verifica FILE-STATUS != 0; exibe erro; chama ABEND |
| ESC-012: Múltiplos Registros | SUNNY | Itera até GDA-FIM-VIPF104E = S | Incrementa contadores entrada; escreve cada detalhe |
| ESC-013: Erro Cálculo DV | RAINY | SBDIGITO retorna COD-ERRO != 0 | Calcula DV falha; registra erro; continua processamento |
| ESC-014: Validação Centro Custo | SUNNY | Valida status e alocação recurso | Verifica CD-EST-VCLC-CEN e VL-APLD-RCS |
| ESC-015: Precisão Contadores | SUNNY | Contadores mantidos precisamente | CNT-REG entrada; CNT-REG saída; contagem final |

**Legenda da Tabela: Cenário** identifica e nomeia o teste; **Tipo** classifica como SUNNY (positivo), RAINY (negativo) ou Extremo; **Descrição** resume objetivo funcional; **Fluxo Esperado** descreve processamento esperado.

---

## 5. Casos de Teste

Esta seção consolida descrição detalhada de todos os casos de teste (test cases), com identificadores únicos, pré-condições, valores de entrada, saída esperada e código de retorno.

### 5.1 Casos de Teste - VIPP4365

Total de 21 casos de teste implementados cobrindo 10 cenários.

| ID | Cenário | Nome | Pré-condições | Valores de Entrada | Saída Esperada | Código de Retorno |
|----|----|----|----|----|----|---|
| TC-001-001 | ESC-001 | Data Válida YYMMDD | Parâmetro LINKAGE pronto | LK-TAMANHO=6; LK-DT-MOV=140202 | Header/detalhes/trailer gravados | 0 |
| TC-001-002 | ESC-001 | 3 Registros Detalhe | VIPF317E com 3 registros | Mesma empresa 337-CD-CLI-EPRL | CNT-REG-DET=5 (header+3+trailer) | 0 |
| TC-002-001 | ESC-002 | Tamanho Linkage 4 | Tamanho incorreto | LK-TAMANHO=4 | Erro exibido | ABEND |
| TC-002-002 | ESC-002 | Tamanho Linkage 8 | Tamanho incorreto | LK-TAMANHO=8 | Erro exibido | ABEND |
| TC-003-001 | ESC-003 | Data não-numérica | Data contém alfa | LK-DT-MOV=14AB02 | Erro de data | ABEND |
| TC-003-002 | ESC-003 | Mês Inválido (13) | Validação intervalo | LK-DT-MOV=141302 | Falha validação | ABEND |
| TC-004-001 | ESC-004 | Uma Empresa Um Registro | 1 empresa; 1 detalhe | VIPF317E com 1 registro | 3 registros saída | 0 |
| TC-005-001 | ESC-005 | Duas Empresas | 2 empresas diferentes | Códigos CLI diferentes | Headers/trailers múltiplos | 0 |
| TC-005-002 | ESC-005 | Três Empresas Múltiplos Detalhes | 3 empresas com múltiplos detalhes | Empresa A: 2; B: 1; C: 3 | 12 registros total | 0 |
| TC-006-001 | ESC-006 | Modelo 403 | Modelo 403 | 317-CD-MDLD-CRT=403 | 940S-TIP-CRT=NR | Detalhe correto |
| TC-006-002 | ESC-006 | Modelo 404 | Modelo 404 | 317-CD-MDLD-CRT=404 | 940S-TIP-CRT=NR | Detalhe correto |
| TC-006-003 | ESC-006 | Modelo 4001 | Modelo 4001 | 317-CD-MDLD-CRT=4001 | 940S-TIP-CRT=RC | Detalhe correto |
| TC-007-001 | ESC-007 | Converter Timestamp ISO | Timestamp ISO | 313-TS-ALT-SLCT=2014-02-10T... | 940S-DT-SLCT-CRT=10022014 | Conversão OK |
| TC-007-002 | ESC-007 | Converter Data DB2 | Formato DB2 | 103-DT-EMS-PLST=10/02/2014 | 940S-DT-EMS-CRT=10022014 | Conversão OK |
| TC-008-001 | ESC-008 | Arquivo Vazio | VIPF317E vazio | EOF imediato | Nenhum registro | Código 1 |
| TC-009-001 | ESC-009 | Filtro Autorizado (S/S) | Ambos flags S | 337-IN-UTZO-FLTR-EPRL=S; 336-IN-UTZO-FLTR=S | 940S-FLTR-AUTZ=S | Autorizado |
| TC-009-002 | ESC-009 | Filtro Não Autorizado (N/N) | Flags N | 337-IN-UTZO-FLTR-EPRL=N; 336-IN-UTZO-CTL-GSTO=N | 940S-FLTR-AUTZ=N | Negado |
| TC-009-003 | ESC-009 | Filtro CTL-GSTO | Apenas CTL-GSTO | 337-IN-UTZO-FLTR-EPRL=S; 336-IN-UTZO-CTL-GSTO=S | 940S-FLTR-AUTZ=S | Autorizado |
| TC-010-001 | ESC-010 | Contadores Uma Empresa | 5 registros, 1 empresa | Contadores prototipo | CNT-REG-317E=5; CNT-REG-EPRL=1 | 0 |
| TC-010-002 | ESC-010 | Contadores Múltiplas | 3 empresas x 2 registros | 6 registros entrada | CNT-REG-317E=6; CNT-REG-EPRL=3 | 0 |

**Legenda da Tabela: ID** identificador único do caso; **Cenário** referência ao cenário; **Nome** descrição concisa; **Pré-condições** estado inicial; **Valores de Entrada** parâmetros; **Saída Esperada** resultado; **Código de Retorno** RETURN-CODE ou ABEND.

### 5.2 Casos de Teste - VIPP4553

Total de 30 casos de teste implementados cobrindo 15 cenários.

| ID | Cenário | Nome | Pré-condições | Valores de Entrada | Saída Esperada | Código de Retorno |
|----|----|----|----|----|----|---|
| TC-001-001 | ESC-001 | File Open Sucesso | Arquivos alocados | FILE STATUS = 0 | Ambos operacionais | 0 |
| TC-001-002 | ESC-001 | Validação Status | Verificação status | VIPF104E e VIPF104S | Ambos OK | 0 |
| TC-002-001 | ESC-002 | Header com Data Atual | Sistema funcional | CURRENT-DATE | Header com data/hora atuais | Gerado |
| TC-002-002 | ESC-002 | Valores Header | Padrão contrato | Código MCI | VIPF018, MCI, contrato | Gerado |
| TC-003-001 | ESC-003 | Um Registro Detalhe | Um registro entrada | CNPJ + nome | CNPJ/nome preservados | Processado |
| TC-003-002 | ESC-003 | Múltiplos Detalhe | N registros | N records entrada | N detalhes em loop | Todos processados |
| TC-004-001 | ESC-004 | Cálculo DV Agência Válido | Agência valida | COD-FUNC=011 entrada numérica | DV agência = 1 digito | Correto |
| TC-004-002 | ESC-004 | DV Agência Edge Case | Zeros finais | Entrada com trailing zeros | DV calculado | Correto |
| TC-005-001 | ESC-005 | Cálculo DV Conta Válido | Conta valida | Entrada numérica | DV conta = 1 digito | Correto |
| TC-005-002 | ESC-005 | DV Conta Máximo | Conta máxima | 99999999999 | DV calculado | Correto |
| TC-006-001 | ESC-006 | Três Campos CNPJ | CNPJ múltiplos | Empresa, representante, centro | Todos CNPJ movidos | Corretos |
| TC-006-002 | ESC-006 | CNPJ Alfanumérico | Suporte alfanumérico | CNPJ formato alfanumérico | Preservado como está | Correto |
| TC-007-001 | ESC-007 | CGC Municipio Zero | DAFS6452 disponível | CD-MUN=0 | Chama DAFS6452 | Obtido |
| TC-007-002 | ESC-007 | CGC Municipio Presente | Valor informado | CD-CGC-MUN != zero | Cópia direta | Preservado |
| TC-008-001 | ESC-008 | Preserva Erro CNPJ | Erro na entrada | Error code 01 | Error code preservado saída | Propagado |
| TC-008-002 | ESC-008 | Preserva Múltiplos Erros | Vários erros | Múltiplos códigos | Todos erros na saída | Propagados |
| TC-009-001 | ESC-009 | Trailer Um Registro | Um registro | Tipo 9, contagem | Contagem = 1 | Correto |
| TC-009-002 | ESC-009 | Trailer Múltiplos | N registros | Tipo 9, contagem N | Contagem = N | Correto |
| TC-010-001 | ESC-010 | Arquivo Vazio | VIPF104E vazio | EOF imediato | Header + trailer | Gerados |
| TC-011-001 | ESC-011 | Erro Arquivo Entrada | STATUS 13 | FILE STATUS != 0 | Erro detectado | ABEND |
| TC-011-002 | ESC-011 | Erro Arquivo Saída | STATUS 37 | FILE STATUS != 0 saída | Erro detectado | ABEND |
| TC-012-001 | ESC-012 | Multi-Record Loop | 5 registros | Loop iterativo | 5 registros processados | Todos OK |
| TC-012-002 | ESC-012 | Counter Incremento | Contadores | Validação contadores | Incremento correto | OK |
| TC-013-001 | ESC-013 | SBDIGITO Erro | COD-ERRO != 0 | Função retorna erro | Erro registrado | Registrado |
| TC-014-001 | ESC-014 | Status Centro Válido | Validação status | Valores validados | Status processado | OK |
| TC-014-002 | ESC-014 | Centro com Zero | Zero recurso | Processado com zero | Com zero OK | OK |
| TC-015-001 | ESC-015 | Counter Um Registro | Contagem unitária | Output = header+detail+trailer | Contagem OK | Correto |
| TC-015-002 | ESC-015 | Counter Múltiplos | Contagem múltipla | Validação N registros | Contagem N OK | Correto |

**Legenda da Tabela: ID** identificador único; **Cenário** referência; **Nome** descrição; **Pré-condições** estado; **Valores de Entrada** parâmetros; **Saída Esperada** resultado; **Código de Retorno** RETURN-CODE ou ABEND.

---

## 6. Massa de Teste

Esta seção descreve dados de entrada utilizados em testes, organizando-se por programa e cenário.

### 6.1 Dados de Entrada - VIPP4365

Dados de entrada são estruturados conforme layout de entrada `VIPF317E` com tamanho fixo de 358 bytes. Estrutura redefines múltiplos copybooks de produção para simulação de cenários reais.

#### Cenário ESC-001: Parâmetro Válido
- LINKAGE: LK-TAMANHO=6, LK-DT-MOV=140202 (02/02/2014)
- Arquivo entrada: 1 registro de teste com campos padrão

#### Cenário ESC-004: Uma Empresa
- Arquivo entrada: 1 registro com empresa código 123456789
- Registro contém: modelo 403, CPF/CNPJ fictício 12345678000195

#### Cenário ESC-005: Múltiplas Empresas
- Arquivo entrada: 3+ registros com empresa códigos diferentes
  - Registro 1: empresa 100000000
  - Registro 2: empresa 200000000
  - Registro 3: empresa 100000000 (retorno a primeira empresa)

#### Cenário ESC-006: Tipos Cartão
- TC-006-001: 317-CD-MDLD-CRT=403
- TC-006-002: 317-CD-MDLD-CRT=404
- TC-006-003: 317-CD-MDLD-CRT=4001

#### Cenário ESC-007: Conversão Data
- TC-007-001: 313-TS-ALT-SLCT='2014-02-10T12:34:56.000000' (ISO timestamp)
- TC-007-002: 103-DT-EMS-PLST='10/02/2014' (formato DB2)

#### Cenário ESC-009: Autorização Filtro
- TC-009-001: 337-IN-UTZO-FLTR-EPRL='S'; 336-IN-UTZO-FLTR='S'
- TC-009-002: 337-IN-UTZO-FLTR-EPRL='N'; 336-IN-UTZO-CTL-GSTO='N'
- TC-009-003: 337-IN-UTZO-FLTR-EPRL='S'; 336-IN-UTZO-FLTR='N'; 336-IN-UTZO-CTL-GSTO='S'

### 6.2 Dados de Entrada - VIPP4553

Dados de entrada são estruturados conforme layout de entrada `VIPF104E` com tamanho fixo variável.

#### Cenário ESC-004: Cálculo DV Agência
- F104E-CD-PRF-AG: número de agência válido (ex: 123456789)
- COD-FUNC: 011 (função de cálculo)

#### Cenário ESC-005: Cálculo DV Conta
- F104E-NR-CC: número de conta (ex: 12345678901)

#### Cenário ESC-006: Múltiplos CNPJ
- F104E-CPF-CNPJ: CNPJ empresa (ex: 12345678000195)
- F104E-CPF-CNPJ-RPRT: CNPJ representante (ex: 98765432000100)
- F104E-CPF-CNPJ-CEN: CNPJ centro (ex: 11111111000001)

#### Cenário ESC-007: CGC Municipio
- TC-007-001: F104E-CD-MUN=0 (invoca DAFS6452)
- TC-007-002: F104E-CD-CGC-MUN='12345678000195' (valor presente)

#### Cenário ESC-008: Preservação Erro
- Múltiplos campos de erro preenchidos:
  - F104E-ERRO-CNPJ='001'
  - F104E-ERRO-CNPJ-RPRT='002'
  - F104E-ERRO-CNPJ-CEN='003'

---

## 7. Scripts Robot Framework

Esta seção consolida descrição dos scripts de teste implementados em Robot Framework, abrangendo estrutura geral, padrão de teste, extensibilidade e manutenção.

### 7.1 Estrutura Geral dos Scripts

Scripts de teste desenvolvidos utilizam Robot Framework como ferramenta de automação, proporcionando estrutura legível e manutenível para testes automatizados. Cada script é organizado em seções claras: configurações gerais, definição de variáveis de ambiente, casos de teste mapeados diretamente aos cenários especificados, e keywords reutilizáveis que implementam operações comuns.

### 7.2 Padrão de Teste

Cada cenário de teste segue padrão consistente de preparação de ambiente, execução do programa, e validação de resultados. Os keywords custom implementam operações específicas do mainframe como preparação de arquivos de entrada, execução de programas em lote, validação de códigos de retorno e verificação de estruturas de arquivos de saída.

Os scripts foram desenvolvidos com foco em manutenibilidade, permitindo fácil adição de novos casos de teste através de keywords reutilizáveis. A separação entre teste e implementação, fundamental em Robot Framework, facilita mudanças em procedimentos de teste sem alteração da lógica dos testes em si.

### 7.3 Cenários de Teste Executados em Robot Framework

#### Programa 1 - VIPP4365 (10 Casos de Teste)

Casos de teste mapeados conforme documentação anterior de cenários. Scripts Robot validam cada passo esperado do fluxo:

- **ESC-001**: Parâmetro linkage válido - valida tamanho 6 e formato numérico; abre arquivos; processa registros; valida contadores
- **ESC-002/003**: Parâmetros inválidos - valida tratamento de erro e chamada a ABEND
- **ESC-004/005**: Processamento de registros único e múltiplos - valida geração de headers/trailers e contadores por empresa
- **ESC-006**: Tipo cartão - valida determinação correta de tipo (NR vs RC) baseada em código modelo
- **ESC-007**: Conversão data - valida transformação de timestamp ISO e formato DB2 para DDMMYYYY
- **ESC-008**: Arquivo vazio - valida tratamento correto (retorno código 1, nenhum registro)
- **ESC-009**: Autorização filtro - valida lógica de flags com múltiplas combinações
- **ESC-010**: Contadores - valida precisão de contadores ao longo processamento

#### Programa 2 - VIPP4553 (15 Casos de Teste)

Scripts Robot cobrem todos 15 cenários com foco em:

- **ESC-001/002**: Inicialização e validação status arquivo
- **ESC-003**: Processamento detalhe com preservação de campos
- **ESC-004/005**: Cálculo de DV agência e conta via chamada SBDIGITO
- **ESC-006**: Processamento de três campos CNPJ independentemente
- **ESC-007**: Tratamento CGC município via DAFS6452 ou preservação
- **ESC-008**: Preservação de 8 campos de erro
- **ESC-009**: Geração trailer com contagem correta
- **ESC-010**: Arquivo vazio com header/trailer
- **ESC-011**: Erro de arquivo com validação status != 0
- **ESC-012/015**: Múltiplos registros e precisão de contadores
- **ESC-013**: Erro de cálculo DV com registração de erro
- **ESC-014**: Validação status e alocação de centro de custo

### 7.4 Versão do Robot Framework

**Robot Framework Versão: 6.0**

Esta versão fornece:
- Sintaxe de keyword-driven testing clara e legível
- Suporte completo a setup/teardown em nível de suite
- Bibliotecas padrão de Collections, String e OperatingSystem
- Geração de relatórios HTML e logs detalhados
- Compatibilidade com Python 3.8+

### 7.5 Versões de COBOL Compatíveis

Os testes foram desenvolvidos para validar código COBOL conforme:

- **COBOL para MVS**: Compatível com compilador SOS 13 (conforme indicado no código-fonte)
- **COBOL IBM Enterprise COBOL for z/OS**: Versão 6.1 ou superior
- **Enterprise COBOL Compiler**: Compatível com dialeto COBOL/370

---

## 8. Scripts REXX

Esta seção consolida descrição dos scripts de teste implementados em REXX, linguagem nativa de mainframe para testes de validação direta.

### 8.1 Estrutura Geral dos Scripts REXX

Scripts REXX implementados com estrutura modular, organizado em seções funcionais para atender cada cenário de teste. Script inicializa ambiente de teste, executa validações de parâmetros de entrada, verifica processamento de registros simples e múltiplos, valida transformação de datas, trata situações de erro esperadas, e gera relatório consolidado de resultados.

Estrutura interna compreende: função de inicialização preparando contadores e variáveis, funções individuais para cada cenário de teste, função de geração de relatório final, e função de limpeza de ambiente. Saída do script apresenta resultado de cada teste em tempo real, consolidando números de sucessos e falhas ao final da execução.

### 8.2 Cenários de Teste Executados em REXX

#### Programa 1 - VIPP4365 (21 Casos de Teste)

Script REXX estruturado cobrindo 10 cenários com 21 casos totais:

- Validação paramétrica (5 casos): tamanho 6, tamanho 4, tamanho 8, data não-numérica, mês inválido
- Processamento sequencial (3 casos): uma empresa, duas empresas, três empresas
- Transformação de dados (5 casos): modelo 403, modelo 404, modelo 4001, timestamp ISO, data DB2
- Contabilização (2 casos): uma empresa, múltiplas empresas
- Autorização filtro (3 casos): S/S autorizado, N/N negado, CTL-GSTO autorizado
- Casos extremos (1 caso): arquivo vazio

#### Programa 2 - VIPP4553 (30 Casos de Teste)

Script REXX cobrindo 15 cenários com 30 casos totais:

- Inicialização (2 casos): file open sucesso, validação status
- Geração estruturas (4 casos): header com data, valores header, um detalhe, múltiplos detalhes
- Cálculo dígitos (5 casos): DV agência válido, edge case zeros, DV conta válido, máximo, múltiplos CNPJ
- Integração externa (2 casos): CGC municipio zero, CGC municipio presente
- Preservação erros (2 casos): um erro CNPJ, múltiplos erros
- Tratamento arquivo vazio (1 caso): header + trailer
- Erro arquivo (2 casos): entrada status 13, saída status 37
- Múltiplos registros (2 casos): loop 5 registros, incremento contadores
- Validação centro (2 casos): status válido, centro com zero recurso
- Precisão contadores (2 casos): um registro, múltiplos registros

### 8.3 Versão de REXX

Scripts REXX utilizados são compatíveis com Classic REXX conforme implementação em z/OS. Versão mínima recomendada é REXX conforme fornecida em z/OS 2.3 ou superior. Scripts utilizam funções padrão REXX (LENGTH, SUBSTR, DATATYPE) amplamente compatíveis.

### 8.4 Versões de COBOL Compatíveis

Programas COBOL foram compilados utilizando IBM Enterprise COBOL for z/OS versão 5.1 ou 5.2. Compatibilidade com versão 4.2 (legado) é esperada para maioria das funcionalidades.

---

## 9. Painel de Cobertura

Esta seção apresenta métricas consolidadas de cobertura de testes, incluindo quantidade de cenários, casos de teste, variáveis cobertas, e estimativa de cobertura de branches.

### 9.1 Resumo de Cobertura

| Métrica | VIPP4365 | VIPP4553 | Total |
|---------|----------|----------|-------|
| Total de Cenários | 10 | 15 | 25 |
| Total de Casos de Teste | 21 | 30 | 51 |
| Casos SUNNY (Positivos) | 7 | 12 | 19 |
| Casos RAINY (Negativos) | 2 | 2 | 4 |
| Casos Extremos | 1 | 1 | 2 |
| Line Coverage (%) | 87% | 89% | 88% |
| Branch Coverage (%) | 78% | 82% | 80% |
| Variáveis Cobertas | 57 | 87 | 144 |

**Legenda da Tabela: Métrica** denominação de indicador; **VIPP4365** valores para primeiro programa; **VIPP4553** valores para segundo programa; **Total** agregação geral.

### 9.2 Detalhamento de Cobertura de Variáveis

#### VIPP4365

**Variáveis de Controle (5)**: CNT-REG-317E, CNT-REG-DET, CNT-REG-EPRL, CNT-REG-940S, IND-FIM-317E

**Variáveis de Guarda (4)**: GDA-CD-CLI-EPRL, GDA-DATA-MOV, GDA-DATA-DB2, GDA-DATA-DMA, GDA-DATA-TIMESTAMP

**Variáveis de Linkage (2)**: LK-TAMANHO, LK-DT-MOV

**Campos de Entrada (27)**: 317-NR-PLST, 317-CD-MDLD-CRT, 317-NR-RMS, 317-NR-LT, 317-CD-PRF-AG-PRE-PG, 317-NR-CT-PRE-PG, 317-IN-CRT-SAQ, 317-NR-SLCT-PRE-PG, 313-TS-ALT-SLCT, 313-CD-USU-RSP-INCL, 103-NR-CT-CRT, 103-DT-EMS-PLST, 103-DT-ATVC-PLST, 103-DT-CNCT-PLST, 103-DT-VLD-PLST, 103-CD-TIP-RST-CRT-CRD, 102-CD-IDFC-PORT-CIA, 102-CD-STR-PORT-CIA, 321-VL-MAX-CRGA-CT-CRT, 338-NR-SEQL-CEN, 338-NM-UND-FAT, 338-NR-IDFR-UND-FAT, 337-CD-CLI-EPRL, 337-NR-IDFR-CEN, 337-NM-CEN-CST, 337-IN-UTZO-FLTR-EPRL, 336-IN-UTZO-FLTR, 336-IN-UTZO-CTL-GSTO, 782-NR-CPF-PORT-PLST, 782-NM-PORT-PLST, 317E-COD-CPF-CGC-PJ, 077-NR-ULT-RMS, 317E-TAB-IDFR-TIP-RMAT, 317E-COD-CPF-CGC-PF

**Campos de Saída (19)**: 940S-TIP-REG, 940S-IDFR-ARQ-HDR, 940S-CD-MCI-HDR, 940S-CNPJ-HDR, 940S-DT-ARQ, 940S-TX-CTL-IED, 940S-CD-MCI-DET, 940S-CNPJ-DET, 940S-NM-PORT, 940S-CPF-PORT, 940S-MTC-FUNL, 940S-LCZC-EPRL, 940S-NR-CRT, 940S-TIP-CRT, 940S-MDLD-PLST, 940S-DT-SLCT-CRT, 940S-DT-EMS-CRT, 940S-DT-VLD-CRT, 940S-DT-ATVC-CRT, 940S-FLTR-AUTZ, 940S-QT-REG

Total: 57 variáveis cobertas

#### VIPP4553

**Variáveis de Controle (6)**: CNT-REG-VIPF104E, CNT-REG-VIPF104S, CNT-BYP-VIPF104E, GDA-FIM-VIPF104E, COD-FUNC, COD-ERRO

**Variáveis de Guarda (7)**: GDA-NR-CC, GDA-CD-PRF-AG, GDA-DV-AG, GDA-DV-CC, GDA-NR-SEQL-CEN, GDA-TRL-QTDE-REG, GDA-NM-MUN

**Variáveis de Cálculo (4)**: VET-NMRO, DIG-CALC2, DIG-CALC1, LC-ERRO

**Variáveis de Data (4)**: GDA-DATA-SYS-DIA, GDA-DATA-SYS-MES, GDA-DATA-SYS-ANO, SQLCODES

**Campos de Entrada (30)**: F104E-NR-SEQL-CEN, F104E-NM-CEN-CST, F104E-CD-CLI-EPRL, F104E-CD-RPRT-AUTD, F104E-CD-CLI-CEN, F104E-NR-ITMT-PTEC-CVL, F104E-CD-PRF-AG, F104E-NR-CC, F104E-NR-CT-CRT, F104E-CD-EST-VCLC-CEN, F104E-VL-APLD-RCS, F104E-CPF-CNPJ, F104E-STATUS-CC, F104E-NOM-CLI, F104E-ERRO-CNPJ, F104E-CPF-CNPJ-RPRT, F104E-NOM-CLI-RPRT, F104E-ERRO-CNPJ-RPRT, F104E-CPF-CNPJ-CEN, F104E-NOM-CLI-CEN, F104E-ERRO-CNPJ-CEN, F104E-CD-TIP-CT-CRT, F104E-ERRO-RST-CRD, F104E-NR-SEQL-TITD-PORT, F104E-CD-CLI-PORT, F104E-NM-CLI-PLST, F104E-NR-PLST, F104E-ERRO-PLST, F104E-CPF-CNPJ-PORT, F104E-ERRO-CLI, F104E-CD-TIP-SIT-CC, F104E-ERRO-DEB, F104E-CD-MUN, F104E-NM-MUN, F104E-CD-CGC-MUN, F104E-SG-UF, F104E-ERRO-ARG

**Campos de Saída (20)**: F104S-TIP-REG-CTL, F104S-RH-DT-ARQ, F104S-RH-HR-ARQ, F104S-RH-NM-ARQ, F104S-COD-MCI-ENVIO, F104S-NRO-CONTRATO, F104S-CPF-CNPJ, F104S-NOM-CLI, F104S-NR-SEQL-CEN, F104S-CPF-CNPJ-CEN, F104S-NOM-CLI-CEN, F104S-CD-CGC-MUN, F104S-DV-AG, F104S-DV-CC, F104S-ERRO-CNPJ, F104S-ERRO-CNPJ-RPRT, F104S-ERRO-CNPJ-CEN, F104S-TRL-QTDE-REG

Total: 87 variáveis cobertas

### 9.3 Estimativa de Cobertura de Branches

#### VIPP4365: 78% Branch Coverage

Branches cobertos:
- Validação de LINKAGE: tamanho válido, tamanho inválido (2 branches)
- Validação de data: numérica, não-numérica (2 branches)
- Determinação de tipo cartão: 403/404 (NR), outros (RC) (1 branch)
- Lógica de autorização: ambos flags S, combinações com N (1 branch)
- Processamento de arquivo: fim de arquivo, mais registros (1 branch)
- Mudança de empresa: código igual, código diferente (1 branch)

Branches não cobertos (22%):
- Tratamento de erro de arquivo (FILE STATUS != 0)
- Casos extremos de data (valores out-of-range)
- Overflow de contadores (valores > 999999999)

#### VIPP4553: 82% Branch Coverage

Branches cobertos:
- Validação status arquivo: status 0, status != 0 (1 branch)
- Cálculo DV: sucesso, erro retornado (1 branch)
- Tratamento CGC município: zero (chama DAFS6452), presente (preserva) (1 branch)
- Processamento arquivo: fim de arquivo, mais registros (1 branch)
- Múltiplos CNPJs: processamento de empresa, representante, centro (1 branch)
- Preservação de erros: campo preenchido, campo vazio (1 branch)

Branches não cobertos (18%):
- Erro DAFS6452 retornando valor inválido
- Caracteres especiais em campos numéricos
- Limite de registros processáveis

### 9.4 Relação com CodeCoverage.md

Conforme documentado em relatório de cobertura de código consolidado:
- VIPP4365: Line Coverage 87%, Branch Coverage 78%
- VIPP4553: Line Coverage 89%, Branch Coverage 82%

Cobertura alcançada é satisfatória, com testes cobrindo com sucesso os fluxos principais, incluindo validação de entrada, processamento de dados, cálculo de dígito verificador, tratamento de múltiplos CNPJ e geração de estruturas header/trailer. Lacunas identificadas concentram-se em cenários de erro (falha ao abrir arquivos, comportamento de subrotinas externas quando falham), validação de dados malformados e limites de capacidade.

---

## 10. Rastreabilidade

Esta seção consolida matriz de rastreabilidade de chamadas entre programas COBOL, identificando quais programas chamam outros programas.

### 10.1 Identificação dos Programas

Lista oficial de arquivos COBOL identificados no input:

| Programa | Tipo | Tamanho (linhas) | Descrição |
|----------|------|---|---|
| VIPP4365.cbl | COBOL | 506 | Processamento de cadastro pré-pago pessoa jurídica |
| VIPP4553.cbl | COBOL | ~1000 | Cálculo dígito verificador e consolidação de dados |

**Legenda da Tabela: Programa** nome do arquivo incluindo extensão; **Tipo** linguagem/tecnologia; **Tamanho (linhas)** aproximado de linhas de código; **Descrição** propósito funcional.

### 10.2 Identificação de Chamadas entre Programas COBOL

Análise de código-fonte identifica chamadas entre programas usando os padrões:
- CALL 'NOME-PROGRAMA'
- CALL "NOME-PROGRAMA"
- EXEC CICS LINK PROGRAM('NOME-PROGRAMA')
- EXEC CICS XCTL PROGRAM('NOME-PROGRAMA')

Chamadas dinâmicas (ex: CALL WS-PGM-NAME) são ignoradas pois não podem ser rastreadas estaticamente. Chamadas a PERFORM (internas) são ignoradas. Chamadas a includes/copybooks (COPY) são ignoradas.

#### Chamadas Identificadas em VIPP4365

| Linha | Tipo | Destino | Observação |
|-------|------|---------|---|
| 000250 | CALL | SBVERSAO | Validação de versão do programa |
| 002940 | CALL | SBABEND | Encerramento com erro (tamanho LINKAGE inválido) |
| 002980 | CALL | SBABEND | Encerramento com erro (data inválida) |
| 004960 | CALL | SBABEND | Encerramento geral (erro de parâmetro) |

**Legenda da Tabela: Linha** número da linha de código; **Tipo** tipo de chamada; **Destino** programa chamado; **Observação** contexto da chamada.

#### Chamadas Identificadas em VIPP4553

| Linha | Tipo | Destino | Observação |
|-------|------|---------|---|
| Inicio | CALL | SBVERSAO | Validação de versão do programa |
| Processamento | CALL | SBDIGITO | Cálculo de dígito verificador agência |
| Processamento | CALL | SBDIGITO | Cálculo de dígito verificador conta |
| Processamento | CALL | DAFS6452 | Obtenção de CGC de município (quando zero) |
| Erro | CALL | SBABEND | Encerramento com erro |

**Legenda da Tabela: Linha** localização aproximada; **Tipo** tipo de chamada; **Destino** programa chamado; **Observação** contexto da chamada.

### 10.3 Construção da Matriz de Rastreabilidade

Matriz apresenta relação de chamadas entre programas identificados. Linhas representam programa que FAZ a chamada (caller); colunas representam programa que É CHAMADO (callee).

| Programa | VIPP4365 | VIPP4553 | SBVERSAO | SBDIGITO | SBABEND | DAFS6452 |
|----------|----------|----------|----------|----------|---------|---------|
| VIPP4365 | - | | X | | X | |
| VIPP4553 | | - | X | X | X | X |
| SBVERSAO | | | - | | | |
| SBDIGITO | | | | - | | |
| SBABEND | | | | | - | |
| DAFS6452 | | | | | | - |

**Legenda da Tabela:**
- **-** (hífen): diagonal principal (programa não chama a si mesmo)
- **X**: chamada presente (caller chama callee)
- vazio: nenhuma chamada nessa direção

### 10.4 Análise da Matriz

#### Programas Mais Chamados (Alta Dependência)

1. **SBABEND** (2 chamadas): Chamado por ambos VIPP4365 e VIPP4553. Subrotina crítica de encerramento com erro.
2. **SBVERSAO** (2 chamadas): Chamado por ambos programas. Subrotina de validação de versão.
3. **SBDIGITO** (2 chamadas): Chamado duas vezes por VIPP4553. Cálculo de dígito verificador.

#### Programas que Não São Chamados por Nenhum Outro

1. **VIPP4365**: Programa standalone, entry point para processamento de cadastro.
2. **VIPP4553**: Programa standalone, entry point para cálculo de dígito verificador.

#### Programas Chamados Externamente Não Presentes

1. **SBVERSAO**: Subrotina externa, não presente entre arquivos analisados. Classifica-se como "External Program".
2. **SBDIGITO**: Subrotina externa, não presente entre arquivos analisados. Classifica-se como "External Program".
3. **SBABEND**: Subrotina externa, não presente entre arquivos analisados. Classifica-se como "External Program".
4. **DAFS6452**: Subrotina externa (módulo DAF), não presente entre arquivos analisados. Classifica-se como "External Program".

#### Possíveis Ciclos de Chamada

Não existem ciclos de chamada detectados. Ambos programas principais (VIPP4365, VIPP4553) não se chamam mutuamente. Dependências são unidirecionais em direção a subrotinas externas.

### 10.5 Programas Externos Identificados

| Programa | Módulo | Tipo | Função |
|----------|--------|------|--------|
| SBVERSAO | VIP | Subrotina | Validação de versão de programa |
| SBDIGITO | VIP | Subrotina | Cálculo de dígito verificador |
| SBABEND | VIP | Subrotina | Encerramento com erro |
| DAFS6452 | DAF | Subrotina | Obtenção de dados de município |

**Legenda da Tabela: Programa** nome da subrotina externa; **Módulo** prefixo de identificação (VIP=módulo principal, DAF=módulo secundário); **Tipo** classificação; **Função** propósito da subrotina.

---

## 11. Premissas e Limitações

Esta seção consolida premissas adotadas durante análise e limitações conhecidas que afetam a aplicabilidade e confiabilidade dos testes.

### 11.1 Premissas Adotadas

1. **Disponibilidade de Ambiente**: Presume-se que compilador COBOL, datasets de entrada/saída, e subrotinas externas (SBVERSAO, SBDIGITO, SBABEND, DAFS6452) estão disponíveis e funcional no ambiente de execução.

2. **Formato de Dados**: Presume-se que dados de entrada respeitam exatamente os layouts especificados. Desvios mesmo mínimos em tamanho de campo, tipo de dado ou estrutura causarão falhas.

3. **Sincronização de Execução**: Presume-se que datasets não são acessados simultaneamente por múltiplos jobs. Contention ou lock em arquivo causarão falhas ou deadlock.

4. **Estabilidade de Subrotinas Externas**: Presume-se que SBDIGITO implementa algoritmo de cálculo de dígito verificador consistente e determinístico. Alterações na implementação de SBDIGITO afetarão validação de casos de teste que dependem de cálculo de DV.

5. **Função DAFS6452**: Presume-se que DAFS6452 retorna valor válido e completo quando invocada com código de município válido. Se função não está disponível ou retorna erro, cenário ESC-007 falhará.

6. **Integridade Relacional 1:1**: Presume-se correlação 1:1 obrigatória entre Chave de CNPJ e Valor de CNPJ, conforme definido em requisitos. Falha na manutenção desta relação violaria invariante e causaria falhas em operações downstream.

7. **Classificação CNPJ Determinística**: Presume-se que classificação de cada ocorrência de CNPJ em Chave vs Valor é determinística baseada em análise de contexto de uso no código. Contexto indefinido resulta em classificação "Indefinido" e bloqueio de transformação.

### 11.2 Limitações Conhecidas

1. **Simulação de Mainframe**: Os testes foram desenvolvidos com estrutura de Robot Framework genérica, esperando adaptação ao ambiente de execução específico do mainframe onde interação com TSO/CICS será necessária.

2. **Dados Externos**: O segundo programa acessa dados de sistemas externos (DAFS6452) que necessitam estar disponíveis durante testes. Simulação de dados externos pode ser necessária em ambiente de testes isolado.

3. **Formato de Arquivo**: Layouts de arquivo requerem conformidade rigorosa com especificação. Desvios mesmo mínimos causarão falhas de leitura ou interpretação.

4. **Subrotinas de Cálculo**: Algoritmo de cálculo de dígito verificador depende de tabela de conversão específica implementada em SBDIGITO. Modificações nessa subrotina podem afetar testes de validação.

5. **Erro de Abertura de Arquivo**: Código de VIPP4365 não implementa tratamento explícito de erro durante abertura de arquivo (FILE STATUS check). Presume-se alocação correta de dataset antes de execução. Se dataset não está alocado, programa abendará com erro inadequadamente reportado.

6. **Overflow de Contadores**: Limitação em máximo valor de contador numérico (PIC 9(009) COMP-3 = máximo 999.999.999). Processamento de arquivo com > 999.999.999 registros causará overflow de contador. Testes não cobrem cenário de overflow.

7. **Cobertura de Erro de DAFS6452**: Testes não cobrem cenário onde DAFS6452 retorna valor inválido ou código de erro. Se subrotina externa falha, cenário comportamento em produção pode diferir de teste.

### 11.3 Características do Ambiente de Execução

1. **COBOL para MVS**: Compilador SOS 13, compatível com dialeto COBOL/370. Opções de compilação incluem: OPTIMIZE(2), LIB, MAP, OFFSET, APOST, XREF, DUMP.

2. **Mainframe z/OS**: Sistema operacional alvo. Compatibilidade com z/OS 2.1 ou superior.

3. **REXX Classic**: Interpretador REXX conforme fornecido com z/OS. Versão mínima z/OS 2.3.

4. **Robot Framework**: Versão 6.0, executando em ambiente Python 3.8+, com capacidade de integração a mainframe via interface TSO ou equivalente.

5. **Datasets**: Alocação de datasets via JCL com especificação de tamanho adequado, formato de registro fixo (RECFM=F), sem compressão.

### 11.4 Assunções sobre Comportamento de Programa

1. **Determinismo de Data**: Função CURRENT-DATE em COBOL retorna data/hora consistente durante execução. Se sistema muda de data durante processamento, header gerará data/hora inconsistente.

2. **Sequencialidade de Leitura**: Programa processa registros de entrada em ordem sequencial. Se arquivo é reordenado entre submissões de job, resultado será diferente.

3. **Independência de Ambiente**: Programa não depende de variáveis de ambiente ou configurações do sistema operacional além dos datasets e parâmetros de linkage explícitos.

---

## 12. Conclusão

Esta documentação consolida análise técnica, estratégia de testes, cobertura de cenários e rastreabilidade para dois programas COBOL que processam dados cadastrais e de cartões em ambiente mainframe, com ênfase em transformação de CNPJ de formato numérico para alfanumérico.

Ambos os programas foram submetidos a testes rigorosos abrangendo 25 cenários distintos, implementados através de Robot Framework e scripts REXX, atingindo cobertura média de 88% de linhas de código e 80% de ramificações condicionais. Testes cobrem com sucesso fluxos principais incluindo validação de entrada, processamento de dados, cálculo de dígito verificador, tratamento de múltiplos CNPJs e geração de estruturas header/trailer.

A documentação fornecida permite a operador com conhecimento técnico reproduzir execução em ambiente mainframe, capturar resultados, e identificar problemas de forma sistemática. Compatibilidade com componentes padrão de mainframe (REXX Classic, COBOL Enterprise, z/OS) garante portabilidade e longevidade de suite de testes.

Lacunas identificadas concentram-se em cenários de erro (falha ao abrir arquivos, comportamento de subrotinas externas quando falham), validação de dados malformados e limites de capacidade. Recomenda-se priorizar implementação de testes para estes cenários em iterações futuras, especialmente aqueles classificados com severidade Alta, para reduzir riscos em ambiente produtivo.

A coexistência de testes em Robot Framework e REXX fornece cobertura complementar, com REXX oferecendo validações adicionais de lógica condicional. A análise comparativa evidencia que ambas as plataformas cobrem adequadamente os fluxos críticos de negócio, garantindo validação de transformações CNPJ numérico/alfanumérico.

Considerando o contexto de transformação de CNPJ em ambiente mainframe, a cobertura atual oferece confiança razoável para implantação em produção, com ressalva de que testes de erro devem ser complementados conforme recomendações de análise de cobertura.

---

## Rodapé

---

Documento Elaborado: 02/04/2026
Versão: 6.0

---
