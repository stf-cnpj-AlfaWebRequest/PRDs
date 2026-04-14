# Plano de Execução — Adequação ao CNPJ Alfanumérico em Ambiente Mainframe

**Demanda:** SPBCA21C-287
**Projeto:** Demanda CNPJ Alfanumérico
**Data de Geração:** 02/04/2026 03:06
**Versão:** 1.0
**Status:** PLANEJAMENTO

---

## Introdução

Este documento detalha a estratégia operacional para execução da adequação ao CNPJ alfanumérico no Sistema VIP, especificamente nos programas VIPP4365 e VIPP4553, em ambiente mainframe. O plano fundamenta-se no diagnóstico técnico que identificou 10 campos CNPJ/CGC classificados como Chave (7 campos) ou Valor (3 campos), com impacto confirmado em 11 artefatos e 2 tabelas DB2. A execução adotará o cenário de complexidade técnica crescente, iniciando por artefatos de menor acoplamento e progredindo conforme validação das etapas anteriores.

---

## Validação do Diagnóstico

A tabela abaixo consolida os campos CNPJ/CGC identificados no diagnóstico técnico, suas classificações, níveis de confiança e ações de execução correspondentes:

| Artefato | Campo | Tipo Detectado | Classificação | Confiança | Ação |
|:---|:---|:---|:---|:---|:---|
| VIPP4365.cbl | 317E-COD-CPF-CGC-PJ | PIC S9(014) COMP-3 | CHAVE com consumo VALOR | Determinística (100%) | Converter para alfanumérico via DFE; propagar em VIPF940S |
| VIPP4365.cbl | 317E-COD-CPF-CGC-PF | PIC S9(014) COMP-3 | CHAVE com consumo VALOR | Determinística (100%) | Converter para alfanumérico via DFE; propagar em VIPF940S |
| VIPP4365.cbl | 782-NR-CPF-PORT-PLST | PIC S9(011) COMP-3 | VALOR | Determinística (100%) | Preservar como CPF (11 dígitos); não alfanumérico |
| VIPP4553.cbl | F104E-CPF-CNPJ | Numérico (entrada) | CHAVE com consumo VALOR | Determinística (100%) | Converter para alfanumérico via DFE; propagar em VIPF104S |
| VIPP4553.cbl | F104E-CPF-CNPJ-RPRT | Numérico (entrada) | CHAVE com consumo VALOR | Determinística (100%) | Converter para alfanumérico via DFE; propagar em VIPF104S |
| VIPK317D.cpy | 317E-COD-CPF-CGC-PJ | PIC S9(014) COMP-3 | CHAVE | Determinística (100%) | Não alterar; suportado por redefinição dinâmica |
| VIPK317D.cpy | 317E-COD-CPF-CGC-PF | PIC S9(014) COMP-3 | CHAVE | Determinística (100%) | Não alterar; suportado por redefinição dinâmica |
| DAFK6452.cpy | DAF-CNPJ-IN / DAF-CNPJ-OUT | Numérico (parâmetro) | CHAVE com consumo VALOR | Determinística (100%) | Módulo secundário — não alterável; informativo apenas |
| DB2VIP.TABCNPJ | CNPJ | Numérico | CHAVE | Determinística (100%) | Potencial nova coluna alfanumérica; decisão pendente |
| DB2MCI.CLIENTE | CNPJ | Numérico | CHAVE | Determinística (100%) | Módulo externo (origem externa); não alterável; preservar |

**Observação:** A classificação como CHAVE não impede o consumo como VALOR em interfaces externas. Nesses casos, o campo deve ser transformado para Valor via Serviço Central de Conversão (DFE) antes de propagação.

---

## Regras de Bloqueio Operacional

As seguintes condições são bloqueios técnicos obrigatórios durante a execução:

### Bloqueio 1: Campo Indefinido
**Regra:** Campos cuja classificação entre Chave e Valor não possa ser determinada com confiança >= 100% não devem ser alterados sob nenhuma circunstância.

**Aplicação:** Nesta demanda, todas as classificações alcançaram nível determinístico. Não há campos Indefinidos.

### Bloqueio 2: Origem Externa
**Regra:** Campos originários de sistemas externos (DB2MCI.CLIENTE, DAFS6452) não devem sofrer transformação, conversão, normalização ou enriquecimento, devendo ser preservados exatamente como recebidos.

**Aplicação:** Campos originários de DB2MCI.CLIENTE (externo ao VIP) serão lidos mas não alterados. Sua transformação para Valor será delegada ao ponto de consumo (programa VIP produtor).

### Bloqueio 3: Classificação Não-Determinística
**Regra:** A execução depende exclusivamente de classificação determinística oriunda do diagnóstico. Inferências fracas (confiança < 100%) bloqueiam qualquer modificação.

**Aplicação:** Todas as classificações nesta demanda têm confiança determinística (100%). Execução prossegue.

### Bloqueio 4: Conversão Centralizada Obrigatória
**Regra:** Sempre que Valor de CNPJ precisar ser convertido em Chave de CNPJ, ou vice-versa, a conversão será realizada **exclusivamente** pelo Serviço Central de Conversão (DFE).

**Aplicação:** Quando campo classificado como CHAVE (origem numérica) for propagado como VALOR em saída alfanumérica, invocará DFE para obter correlação 1:1.

---

## Análise Consolidada de Impacto para Execução

### Impacto Estrutural

**Artefatos Estruturais Afetados:** 2 programas COBOL principais (VIPP4365, VIPP4553), 2 copybooks (VIPK317D, potencialmente DAFK6452), 2 tabelas DB2 (DB2VIP.TABCNPJ, DB2VIP.PORT_PLST_PRE_PG).

**Impacto:** Médio
- **Razão:** Alterações concentradas nos programas principais; copybooks compartilhados serão impactados indiretamente via redefinição de campos em MOVE.
- **Ação:** Revisar Picture Clauses em campos de saída (VIPF940S, VIPF104S) para suportar 14 caracteres alfanuméricos. Manter estrutura existente; não criar versões paralelas (V2, ALT, NEW).

### Impacto Técnico

**Mudança de Tipo de Dado:** Conversão de numérico (PIC 9 / COMP-3) para alfanumérico (PIC X) em 5 campos de saída.

**Impacto:** Alto
- **Riscos Identificados:**
  1. Operações numéricas (`COMPUTE`, `IF EQUAL ZEROS`, somas, divisões) sobre campos alfanuméricos causarão erros de abend em runtime.
  2. Uso de `MOVE FUNCTION NUMVAL()` pode ser necessário para conversão.
  3. Campos COMP-3 em entrada (originários de arquivo externo numérico) devem ser lidos como numéricos; conversão para alfanumérico ocorre apenas na saída após lookup de DFE.
- **Mitigação:** 
  - Identificar todas operações numéricas sobre campos CNPJ; substituir por operações de string se aplicável.
  - Aplicar validação de dígito verificador apenas em Valor de CNPJ, nunca em Chave.
  - Implementar invocação a DFE imediatamente antes de propagação de campo como VALOR.

### Impacto Funcional

**Fluxo de Processamento:** Adição de passo de conversão via DFE antes de gravação em saída alfanumérica.

**Impacto:** Médio
- **Mudança Funcional:** Programa acionará Serviço Central de Conversão (DFE) para cada ocorrência de CNPJ a ser propagado em formato alfanumérico. Tempo de processamento aumentará conforme latência de DFE (estimado: +2-5% por acesso).
- **Ação:** Implementar cache local (VSAM ou estrutura em memória) para reduzir acessos repetidos a DFE.

### Impacto de Dependência

**Dependências Internas:**
- VIPP4365 depende de VIPK317D (copybook compartilhado)
- VIPP4365 depende de VIPF317E (arquivo entrada) e VIPF940S (arquivo saída)
- VIPP4553 depende de VIPF104E (entrada) e VIPF104S (saída)
- VIPP4553 depende de DAFS6452 (serviço externo) para enriquecimento

**Impacto:** Alto
- **Razão:** Copybook compartilhado (VIPK317D) declara campos CNPJ; alterações podem impactar outros programas que a incluem (análise futura fora escopo VIP).
- **Ação:** Manter compatibilidade regressiva em VIPK317D; não alterar declaração original de COMP-3. Usar redefinição em nível de programa.

### Impacto de Dados

**Campos Afetados:** 10 campos CNPJ/CGC (7 Chave, 3 Valor).

**Impacto:** Crítico
- **Delta de Armazenamento:** 
  - VIPF940S: +14 bytes (1 coluna nova alfanumérica) × quantidade de registros/período
  - VIPF104S: +70 bytes (5 colunas novas) × quantidade de registros/período
  - DB2VIP.TABCNPJ: +0 a +14 bytes (depende de decisão de adição de coluna nova)
- **Ação:** Provisionar espaço adicional em disco/DB2 antes de execução. Validar crescimento de dataset de saída.

### Impacto de Integração

**Sistemas Integrados:**
- Sistema externo SIAFI (consumidor de VIPF940S)
- Serviço Federal DAFS6452 (enriquecimento de CNPJ)
- Tabelas DB2MCI (origem externa, apenas leitura)

**Impacto:** Crítico
- **Razão:** Sistemas downstream (SIAFI) devem estar preparados para consumir campo CNPJ alfanumérico. Contrato de interface pode quebrar se consumidor não suportar novo tamanho.
- **Ação:** Validar compatibilidade de SIAFI com campo CNPJ de até 14 caracteres alfanuméricos. Coordenar com time SIAFI antes de go-live.

### Impacto de Risco

**Riscos Identificados:**
1. **Risco Contratual (Crítico):** Consumidores de VIPF940S e VIPF104S devem tolerar novo tamanho de campo CNPJ. Falha causará rejeição de registros.
2. **Risco de Dados (Alto):** Conversão com DFE pode retornar Chave inválida se CNPJ não existir na tabela central. Estratégia de fallback necessária.
3. **Risco de Performance (Médio):** Acessos adicionais a DFE podem impactar throughput batch. Cache mitiga.
4. **Risco de Compatibilidade (Alto):** Campo COMP-3 em entrada convertido para PIC X causará problemas se código downstream assumir formato numérico.

**Conclusão da Análise:** Impacto geral **Alto-Crítico**. Execução viável, mas requer coordenação com sistemas externos e validação rigorosa de contrato de interface.

---

## Análise de Dependência

### Grafo de Dependências

```
DB2MCI.CLIENTE (origem externa)
    ↓ [leitura via VIPP4365]
VIPK317D.cpy (copybook compartilhado)
    ↓ [include]
VIPP4365.cbl (programa principal)
    ├─→ DFE (Serviço Central de Conversão) [lookup 1:1]
    └─→ VIPF940S (saída)
        └─→ SIAFI (consumidor externo) [potencial quebra contratual]

DB2VIP.PORT_PLST_PRE_PG (origem interna)
    ↓ [leitura via VIPP4365]
VIPP4365.cbl
    └─→ VIPF940S (saída)

VIPF104E (entrada externa)
    ↓ [leitura via VIPP4553]
VIPP4553.cbl (programa principal)
    ├─→ DAFS6452 (serviço externo) [enriquecimento]
    ├─→ DFE (Serviço Central de Conversão) [lookup]
    └─→ VIPF104S (saída)

DAFK6452.cpy (copybook módulo secundário)
    ├─ DAFS6452 (serviço integrado)
    └─ VIPP4553 (consumidor via CALL)
```

### Grau de Dependência por Nó

| Nó | In-Degree | Out-Degree | Criticidade |
|:---|:---:|:---:|:---|
| DB2MCI.CLIENTE | 0 | 1 | Crítica (origem) |
| VIPK317D.cpy | 1 | 1 | Alta (compartilhado) |
| VIPP4365.cbl | 2 | 3 | Crítica (hub) |
| VIPF940S | 1 | 1 | Crítica (saída) |
| VIPP4553.cbl | 1 | 2 | Crítica (hub) |
| VIPF104S | 1 | 1 | Crítica (saída) |
| DFE | 2 | 0 | Crítica (lookup centralizado) |
| DAFS6452 | 1 | 1 | Alta (integração) |
| SIAFI | 1 | 0 | Crítica (consumidor externo) |

**Observação:** Programas VIPP4365 e VIPP4553 são hubs críticos; falha em qualquer um impacta toda cadeia subsequente.

---

## Estratégias de Execução

### Possíveis Cenários

#### Cenário 1: Complexidade Crescente (Técnica)

**Princípio:** Progressão ordenada por esforço e risco crescente, similar a Canary Release. Início por artefatos isolados, baixo acoplamento; avanço apenas após validação da etapa anterior.

**Passos Sucessivos:**

**Passo 1 — Alterações em Copybooks Internos (Impacto Contido)**
- Artefatos: VIPK317D.cpy
- Ação: Revisar Picture Clauses; manter COMP-3 original (não converter tipo de dado). Usar redefinição em nível de programa para suportar consumo como VALOR.
- Risco: Baixo (mudança localizada, compatibilidade regressiva mantida)
- Validação: Recompilação de VIPP4365 e VIPP4553; testes unitários

**Passo 2 — Alteração de Programa VIPP4365 (Campo Único de Saída)**
- Artefatos: VIPP4365.cbl, VIPF940S
- Ação: Adicionar lógica de conversão via DFE para campos 317E-COD-CPF-CGC-PJ e 317E-COD-CPF-CGC-PF. Expandir Picture Clause de VIPF940S-CNPJ-HDR/DET de PIC 9(14) para PIC X(14). Implementar cache VSAM para reduzir acessos a DFE.
- Risco: Médio (novo campo de saída maior; consumidor SIAFI pode rejeitar)
- Validação: Teste de contrato com SIAFI; validação de aumento de dataset

**Passo 3 — Alteração de Programa VIPP4553 (Múltiplos Campos de Saída)**
- Artefatos: VIPP4553.cbl, VIPF104S
- Ação: Adicionar lógica de conversão via DFE para campos F104E-CPF-CNPJ e F104E-CPF-CNPJ-RPRT. Expandir Picture Clause de VIPF104S de 454B para ~524B. Reutilizar cache de Passo 2.
- Risco: Alto (múltiplos campos; crescimento significativo de dataset)
- Validação: Teste de capacidade de armazenamento; validação de contrato downstream

**Passo 4 — Tabelas DB2 (Decisão Arquitetural Pendente)**
- Artefatos: DB2VIP.TABCNPJ, DB2VIP.PORT_PLST_PRE_PG
- Ação: Decidir se adicionar coluna nova alfanumérica ou fazer in-place update (decisão não tomada neste plano). Se coluna nova: provisionar espaço, atualizar índices. Se in-place: risco de incompatibilidade com aplicações que leem campo original.
- Risco: Crítico (impacto global em DB2; potencial indisponibilidade)

**Vantagens:**
- Validação incremental reduz janela de risco
- Reversão isolada sem impactar etapas posteriores
- Aprendizado técnico acumulativo
- Sistema operacional durante execução

**Limitações:**
- Tempo total maior (sequencial, não paralelo)
- Dependências não mapeadas podem emergir
- Demanda sincronização com consumidores externos em cada passo

---

#### Cenário 2: Estratégico (Negócio)

**Princípio:** Execução orientada por fatores operacionais, regulatórios ou de negócio, não por esforço técnico. Decisão tomada por executivos com suporte técnico informativo.

**Possíveis Drivers Estratégicos:**
- **Regulatório:** Receita Federal define prazo para suporte a CNPJ alfanumérico (ex.: Dec. 2026). Execução acelerada, fora da sequência técnica ótima.
- **Sazonalidade:** Período de baixo volume (ex.: agosto) escolhido como janela de implantação, mesmo que tecnicamente seja possível executar anytime.
- **Coordenação com Projetos:** Execução coincide com upgrade de plataforma mainframe ou manutenção programada.
- **Priorização de Negócio:** Artefatos que sustentam processos de maior receita (ex.: pré-pago PJ) priorizados, mesmo que mais complexos.

**Características:**
- Decisão executiva, não técnica
- Pode divergir do sequenciamento técnico ótimo
- Risco aceito documentado
- Agente não escolhe; apenas apresenta

---

### Cenário Escolhido

Conforme premissas desta PoC, será adotado **Cenário 1 — Complexidade Crescente (Decisão Técnica)**.

**Justificativa:** Abordagem de menor risco inicial, aprendizado incremental, permite reversão isolada. Adotará progressão desde artefatos de menor acoplamento, cadeia de impacto curta e classificação determinística, avançando apenas após validação.

**Nenhuma antecipação de etapa é permitida** sem reclassificação formal do cenário.

---

## Ordem de Execução

### Fase 1: Preparação

**Duração Estimada:** 1-2 semanas (paralelizável)

**Atividades:**

1. **Ambiente de Homologação**
   - Provisionar ambiente mainframe (cópia de produção ou equivalente)
   - Preparar dados de teste representativos (arquivos VIPF317E, VIPF104E com CNPJs variados)
   - Configurar mecanismo de cache (VSAM ou equivalente)

2. **Validação de Contrato Externo**
   - Coordenar com time SIAFI para confirmar suporte a CNPJ alfanumérico de 14 caracteres
   - Coordenar com time DAF para confirmar evolução de DAFS6452 (se necessário)
   - Documentar respostas em Change Log

3. **Preparação de Código**
   - Identificar todos MOVE, operações numéricas, validações sobre campos CNPJ
   - Mapear chamadas a DFE (verificar disponibilidade, SLA, fallback)
   - Preparar biblioteca de funções para conversão (MOVE FUNCTION, validação DV, etc.)

4. **Preparação de Dados**
   - Validar integridade de DB2MCI.CLIENTE e DB2VIP.TABCNPJ
   - Preparar script de backup antes de qualquer alteração

---

### Fase 2: Execução Conforme Diagnóstico (Passo 1-2)

**Duração Estimada:** 2-3 semanas

**Passo 1: Copybook VIPK317D**
- Revisar fields CNPJ em VIPK317D; confirmar tipo COMP-3
- Documentar redefinição em nível de programa (VIPP4365)
- Recompilar VIPP4365, VIPP4553; testes unitários

**Passo 2: Programa VIPP4365**
- Modificar lógica de processamento:
  - Ao ler 317E-COD-CPF-CGC-PJ de VIPF317E: armazenar como Chave numérica
  - Antes de escrever em VIPF940S: invocar DFE para obter Valor alfanumérico
  - Escrever campo expansível VIPF940S-CNPJ-HDR/DET como PIC X(14)
  
- Implementar cache VSAM:
  - Estrutura: KEY=Chave (14 dígitos numéricos), VALUE=Valor (14 caracteres)
  - Política: LRU (Least Recently Used), TTL 24h
  
- Tratamento de Erro:
  - Se DFE retorna erro (CNPJ não encontrado): log de auditoria; utilizar Chave como fallback temporário
  
- Recompilar e testar em ambiente homologação

**Validação:**
- Teste com dados reais de DB2MCI.CLIENTE
- Validar integridade de saída VIPF940S
- Confirmar contrato com SIAFI

---

### Fase 3: Execução Conforme Diagnóstico (Passo 3)

**Duração Estimada:** 1-2 semanas

**Passo 3: Programa VIPP4553**
- Modificar lógica:
  - Ao ler F104E-CPF-CNPJ de VIPF104E: armazenar como Chave
  - Antes de escrever em VIPF104S: invocar DFE (reutilizar cache de VIPP4365)
  - Escrever campos expansíveis de VIPF104S como PIC X(14)
  
- Reutilizar cache VSAM de VIPP4365 (compartilhado via área de comunicação ou chamada a programa utilitário)

- Tratamento de integração com DAFS6452:
  - Confirmar se DAFS6452 já retorna CNPJ alfanumérico ou numérico
  - Se alfanumérico: usar direto; se numérico: invocar DFE

- Recompilar e testar

**Validação:**
- Teste com dados de VIPF104E
- Validar crescimento de VIPF104S de 454B para ~524B
- Confirmar contrato downstream

---

### Fase 4: Integrações (Passo 4 — Decisão Pendente)

**Duração Estimada:** 2-4 semanas (condicionada a decisão)

**Passo 4: Tabelas DB2**
- **Decisão Arquitetural Requerida:** Antes de executar este passo, cliente deve decidir:
  - Opção A: Adicionar coluna nova `CNPJ_ALFA` (PIC X(14)) em DB2VIP.TABCNPJ
  - Opção B: In-place update de coluna existente (risco de incompatibilidade)
  - Opção C: Não alterar tabela DB2; usar cache VSAM como fonte-verdade (recomendado)
  
- Se Opção A: provisionar espaço, criar coluna, popular com dados via batch
- Se Opção C: validar que cache VSAM tem TTL adequado e capacidade suficiente

**Validação:**
- Teste de integridade DB2
- Teste de performance (índices, scans)

---

### Fase 5: Consolidação

**Duração Estimada:** 1 semana

**Atividades:**
- Testes de integração end-to-end
- Teste de carga (volume esperado de processamento)
- Simulação de failover (DFE indisponível)
- Preparação de runbook operacional
- Treinamento de suporte

---

## Execução Detalhada

### Alterações em VIPP4365.cbl

**Contexto:** Programa lê dados de VIPF317E (provenientes de DB2MCI.CLIENTE), processa, escreve em VIPF940S.

**Tipos de Alteração:**

**Tipo 1: Adição de Invocação a DFE**
- **Localização:** Após MOVE de campo CNPJ de origem para Working Storage, antes de MOVE para arquivo de saída
- **Padrão de Ajuste:**
  ```
  MOVE 317E-COD-CPF-CGC-PJ TO WS-CNPJ-CHAVE.
  CALL 'DFE' USING WS-CNPJ-CHAVE WS-CNPJ-VALOR WS-DFE-STATUS.
  IF WS-DFE-STATUS NOT = ZERO
     MOVE WS-CNPJ-CHAVE TO WS-CNPJ-VALOR  (fallback: usar Chave)
     WRITE LOG-AUDITORIA
  END-IF.
  MOVE WS-CNPJ-VALOR TO 940S-CNPJ-HDR.
  ```
- **Ocorrências:** 2 campos em VIPP4365 (317E-COD-CPF-CGC-PJ, 317E-COD-CPF-CGC-PF)

**Tipo 2: Expansão de Picture Clause**
- **Localização:** Seção de Arquivo de Saída (FD VIPF940S, registros 940S-HEADER e 940S-DETALHE)
- **Padrão de Ajuste:**
  ```
  Antes:  03  940S-CNPJ-HDR           PIC  9(014).
  Depois: 03  940S-CNPJ-HDR           PIC  X(014).
  ```
- **Ocorrências:** 2 campos (940S-CNPJ-HDR, 940S-CNPJ-DET)
- **Impacto:** Aumento de tamanho de registro não significativo (PIC 9 = 2 bytes, PIC X = 14 bytes; delta +12 bytes por campo, máximo +24 por registro em worst case)

**Tipo 3: Implementação de Cache VSAM**
- **Localização:** Novo programa utilitário ou seção em VIPP4365
- **Padrão de Ajuste:**
  ```
  01  WS-CACHE-VSAM.
      03  WS-CACHE-KEY           PIC 9(14).
      03  WS-CACHE-VALUE         PIC X(14).
      03  WS-CACHE-HIT-FLAG      PIC 9 VALUE ZERO.
  
  PERFORM LOOKUP-CACHE.
    IF WS-CACHE-HIT-FLAG = ZERO
       CALL 'DFE' USING WS-CNPJ-CHAVE WS-CNPJ-VALOR WS-DFE-STATUS
       PERFORM CACHE-STORE
    END-IF.
  ```
- **Impacto:** Redução de acessos a DFE em ~60-80% (estimado, depende de padrão de dados)

---

### Alterações em VIPP4553.cbl

**Contexto:** Programa lê dados de VIPF104E (arquivo externo), enriquece via DAFS6452, escreve em VIPF104S.

**Tipos de Alteração:**

**Tipo 1: Adição de Invocação a DFE**
- **Localização:** Mesmo padrão de VIPP4365
- **Ocorrências:** 2 campos em VIPP4553 (F104E-CPF-CNPJ, F104E-CPF-CNPJ-RPRT)

**Tipo 2: Expansão de Picture Clause**
- **Localização:** FD VIPF104S, registros de detalhe
- **Padrão de Ajuste:** Similar a VIPP4365
- **Ocorrências:** 5 campos (conforme diagnóstico)
- **Impacto:** Aumento de tamanho de registro: +70 bytes (5 campos × 14 caracteres)

**Tipo 3: Reutilização de Cache VSAM**
- **Localização:** Chamar programa utilitário compartilhado (se separado) ou reutilizar seção em VIPP4365
- **Impacto:** Economia de código; sincronização de cache entre VIPP4365 e VIPP4553

**Tipo 4: Integração com DAFS6452**
- **Ação:** Confirmar se DAFS6452 retorna CNPJ alfanumérico ou não
- **Padrão de Ajuste:** Se numérico, aplicar DFE após retorno de DAFS6452

---

### Operações Técnicas Incompatíveis

**Risco Crítico: Operações Numéricas sobre Campos Alfanuméricos**

Qualquer operação que assuma campo numérico e tente executar aritmética sobre campo PIC X causará abend:

| Operação | Risco | Exemplo | Ação |
|:---|:---|:---|:---|
| COMPUTE | Crítico | `COMPUTE WS-TOTAL = WS-CNPJ-VALOR / 100` | Identificar; remover ou substituir por MOVE |
| IF EQUAL ZEROS | Crítico | `IF 940S-CNPJ-HDR = ZEROS` | Alterar para `IF 940S-CNPJ-HDR = SPACES` |
| MOVE numérico-para-COMP-3 | Crítico | `MOVE 940S-CNPJ-HDR TO 940S-CNPJ-COMP3` | Alterar para MOVE STRING ou parsing |
| ADD / SUBTRACT | Crítico | `ADD 940S-CNPJ-HDR TO WS-COUNTER` | Remover se aplicável ao campo |

**Mitigação:** Auditoria de código executada durante Fase 2 (revisão de VIPP4365, VIPP4553). Qualquer operação numérica identificada bloqueará execução até resolução.

---

## Riscos Técnicos de Conversão de Tipo

### Risco 1: Comparação com ZEROS em Campo Alfanumérico

**Cenário:** Código legado contém validação como `IF 940S-CNPJ-HDR = ZEROS`.

**Impacto:** Em COBOL, `ZEROS` é interpretado como sequência de bytes zero (x'00'). Um campo PIC X preenchido com espaços (x'20') ou caracteres não será igual a ZEROS.

**Mitigação:** Substituir por `IF 940S-CNPJ-HDR = SPACES OR 940S-CNPJ-HDR = '00000000000000'`.

---

### Risco 2: Somas ou Cálculos sobre CNPJ

**Cenário:** Raro, mas possível: código tenta calcular check digit ou hash sobre CNPJ numérico.

**Impacto:** Em campo PIC X, cálculos numéricos causam abend.

**Mitigação:** Validação de dígito verificador aplicada **apenas em Valor de CNPJ**, nunca em Chave. Se DV existir, invocar rotina especializada de validação (não aritmética nativa).

---

### Risco 3: Conversão Implícita em MOVE

**Cenário:** `MOVE 940S-CNPJ-HDR (PIC X) TO WS-CNPJ-NUMERIC (PIC 9(14))`.

**Impacto:** COBOL permitirá MOVE alfanumérico → numérico, mas resultará em zeros ou lixo se campo contiver caracteres não-numéricos.

**Mitigação:** Usar `MOVE FUNCTION NUMVAL()` com validação de erro. Se campo for realmente alfanumérico, **não** converter para numérico; manter como string.

---

## Estratégia de Dados

### Coexistência de Formatos

**Premissa:** Durante transição, sistema deve tolerar coexistência de CNPJ numérico (legado) e alfanumérico (novo) em produção.

**Estratégia:**
1. **Na Entrada:** Manter arquivo VIPF317E e VIPF104E em formato numérico (não alterar contrato com produtor externo).
2. **Na Saída:** 
   - VIPF940S e VIPF104S suportarão ambos: campo numérico legado (se consumidor antigo ainda lê) + campo alfanumérico novo (se consumidor novo já lê).
   - Recomendação: adicionar coluna nova (940S-CNPJ-HDR-ALFA, 940S-CNPJ-DET-ALFA) ao invés de sobrescrever campo existente. Consumidores legados ignoram coluna nova; novos consumidores leem coluna nova.
3. **Na Persistência DB2:** Cache VSAM ou coluna nova (decisão pendente) para Valor alfanumérico.

### Sincronização Entre Ambientes

**Ambiente Mainframe:** Múltiplos ambientes (dev, teste, homologação, produção) devem ser sincronizados.

**Estratégia:**
- **Cache VSAM:** Replicado via batch distribuidora (step posterior de JCL) que popula cache em cada ambiente
- **DB2:** Mirroring automático (replicação native do DB2) ou manual (dump/restore)
- **Arquivos:** Manter snapshot de VIPF317E, VIPF104E em cada ambiente com dados de teste consistentes

---

## Estratégia de Transição

### Fase de Introdução (Semanas 1-4)

**Objetivo:** Validar padrão de conversão em ambiente isolado.

- Executar VIPP4365 e VIPP4553 em homologação com dados de teste
- Validar saída VIPF940S e VIPF104S
- Confirmar contrato com SIAFI

### Fase de Adoção (Semanas 5-8)

**Objetivo:** Migração controlada para produção com rollback pronto.

- Deploy em produção (modo silent, sem impacto a usuários)
- Coletar métricas: tempo de processamento, taxa de erro DFE, crescimento de dataset
- Monitoramento 24/7 durante primeira semana

### Fase de Domínio (Semanas 9-16)

**Objetivo:** Consolidação; desativar fallbacks e otimizar.

- Remover código de fallback (usar Chave como Valor) após 100% de cobertura DFE validada
- Otimizar cache VSAM com base em padrão de dados observado
- Estender suporte a CNPJ alfanumérico em copybooks compartilhados (VIPK317D) se outros programas o exigirem

### Fase Futura (Indefinida)

**Objetivo:** Modernização de código legado (fora escopo desta demanda).

- Refatorar VIPP4365 e VIPP4553 para separar Chave e Valor em estruturas distintas
- Implementar DFE como middleware (não CALL, mas HTTP/REST)
- Suportar novos formatos de identificador (ex.: CNAE, inscrição estadual)

---

## Estratégia de Lookup

### Serviço Central de Conversão (DFE)

**Responsabilidade:** Manter correlação 1:1 entre Chave de CNPJ (numérica) e Valor de CNPJ (alfanumérica).

**Contrato:**
```
CALL 'DFE' USING:
  Input:  WS-CNPJ-CHAVE (PIC 9(14))
  Output: WS-CNPJ-VALOR (PIC X(14)), WS-STATUS (PIC 9(4))
```

**Status Esperado:**
- 0000 = Sucesso; Valor retornado em WS-CNPJ-VALOR
- 0001 = Chave não encontrada; usar fallback (log + usar Chave)
- 9999 = Erro de sistema; uso fallback + alert

### Estratégia de Cache

**Tipo de Cache:** VSAM local ou estrutura em memória (Working Storage expandido).

**Estrutura:**
```
01  WS-CACHE-ENTRY.
    03  WS-CACHE-KEY          PIC 9(14).
    03  WS-CACHE-VALUE        PIC X(14).
    03  WS-CACHE-TIMESTAMP    PIC 9(10).
    03  WS-CACHE-HIT-COUNT    PIC 9(5).

01  WS-CACHE-TABLE OCCURS 10000 TIMES.
    05  WS-CACHE-ENTRY
```

**Política de Invalidação:**
- TTL (Time-To-Live): 24 horas
- LRU (Least Recently Used): evicção quando cache cheio (10.000 entradas)
- Manual: step batch que limpa e repopula cache diariamente

**Impacto Esperado:**
- Hit Rate: 60-80% (reduz acessos a DFE significativamente)
- Miss Penalty: +50-100ms por acesso a DFE (vs. <1ms hit local)
- Armazenamento: ~500KB (10.000 entradas × 50 bytes/entrada)

### Denormalização (Alternativa Futura)

**Cenário:** Se DFE não for suficientemente rápido ou confiável, considerar denormalização:
- Copiar Valor alfanumérico para DB2VIP.TABCNPJ como coluna nova
- Eliminar necessidade de CALL a DFE em runtime (lookup local em DB2)
- Trade-off: aumenta footprint de dados; requer sincronização periódica com DFE

**Não será implementado neste plano**, mas documentado como contingência.

---

## Contingência Técnica

### Procedimento de Falha do DFE

**Cenário:** Serviço Central de Conversão indisponível durante execução batch.

**Estratégia:**
1. **Detecção:** CALL a DFE retorna status 9999 ou timeout
2. **Fallback Imediato:**
   ```
   IF WS-DFE-STATUS NOT = 0000
      MOVE WS-CNPJ-CHAVE TO WS-CNPJ-VALOR  (usar Chave como Valor temporariamente)
      WRITE AUDIT-LOG ('DFE_UNAVAILABLE', WS-CNPJ-CHAVE)
      ADD 1 TO WS-DFE-ERROR-COUNT
   END-IF.
   ```
3. **Continuidade:** Batch continua com Chave como substituto de Valor; arquivo de saída válido (menor risc que abend)
4. **Recuperação:** Após DFE recuperação, re-processar registros com erro (identificados em audit log)

**SLA Esperado:** DFE disponível 99.9% do tempo durante janela batch

---

## Plano Alternativo para Integrações

### Sistema SIAFI (Consumidor de VIPF940S)

**Contrato Atual:** SIAFI consome arquivo VIPF940S com CNPJ numérico (PIC 9(14)).

**Risco:** Se SIAFI não suportar PIC X(14), rejeitará registros com CNPJ alfanumérico.

**Alternativa 1 — Migração Coordenada:**
- Coordenar com time SIAFI
- Ambos sistemas (VIP + SIAFI) evoluem simultaneamente
- VIP produz VIPF940S com PIC X(14); SIAFI consome

**Alternativa 2 — Convivência Temporária:**
- Manter coluna numérica legado (940S-CNPJ-HDR-NUM) com Chave original
- Adicionar coluna nova (940S-CNPJ-HDR-ALFA) com Valor alfanumérico
- SIAFI legado consome coluna NUM; SIAFI novo consome coluna ALFA
- Delta de dataset: +14 bytes por registro

**Alternativa 3 — Transformação em Saída:**
- VIP escreve arquivo VIPF940S com Valor alfanumérico em PIC X(14)
- Programa transformador (job batch) executa post-processamento: cria segundo arquivo VIPF940S-COMPAT com campo numérico para SIAFI legado
- SIAFI legado consome VIPF940S-COMPAT; SIAFI novo consome VIPF940S
- Impacto: reprocessamento, latência adicional

**Recomendação:** Alternativa 1 (migração coordenada). Validar com SIAFI antes de go-live.

---

## Monitoramento

### Ferramenta de Monitoramento

**Ambiente:** IBM Mainframe (Tivoli, CA APM, ou equivalente interno)

**Métricas Críticas:**

| Métrica | Threshold Alerta | Ação |
|:---|:---|:---|
| Taxa de erro DFE | > 5% | Escalar; verificar disponibilidade DFE |
| Tempo de processamento batch | > 120% da baseline | Investigar; verificar cache hit rate |
| Taxa de crescimento dataset VIPF940S | > 115% esperado | Investigar; possível falha de compressão |
| Taxa de crescimento dataset VIPF104S | > 115% esperado | Investigar; possível falha de compressão |
| Disponibilidade de cache VSAM | < 99% | Reinicializar cache |

### Observabilidade

**Logs Requeridos:**
- **Auditoria DFE:** cada CALL, sucesso/falha, Chave + Valor retornado
- **Performance:** timestamp entrada → timestamp saída por registro
- **Erro:** qualquer abend ou exceção, stacktrace completo
- **Cache:** hit/miss rate, tempo de lookup, evicção

**Dashboards:**
- Taxa de erro DFE (gráfico temporal)
- Hit rate cache (gráfico de tendência)
- Tamanho de dataset (saída vs. baseline histórico)
- Throughput batch (registros/segundo)

---

## Deploy

### Ferramenta de Deploy

**Ambiente:** IBM Mainframe / CA Endevor (ou equivalente de versionamento)

**Procedimento de Deploy:**

1. **Pré-Deploy Checklist:**
   - [ ] Código recompilado e testado em homologação
   - [ ] Teste de contrato com SIAFI validado
   - [ ] Backup de código antigo realizado
   - [ ] JCL revisado e testado
   - [ ] Runbook operacional preparado

2. **Execução de Deploy:**
   - Janela de manutenção programada (ex.: sábado 02:00-06:00)
   - Link-editar binários novos (linkedição COBOL)
   - Atualizar JCL em produção (se mudança de steps necessária)
   - Validar que programa novo está linkado corretamente

3. **Pós-Deploy:**
   - Executar primeira batch com dados de teste reduzido
   - Validar saída (VIPF940S e VIPF104S)
   - Monitorar logs de erro
   - Se OK: liberar para produção plena

### Rollback Procedure

**Se detecção de erro crítico pós-deploy:**
1. Parar execução batch em andamento
2. Deslinkedar binário antigo; relinkedar programa anterior
3. Re-executar batch com código antigo
4. Notificar stakeholders
5. Abrir incident ticket para investigação

**Tempo estimado de rollback:** 15 minutos

---

## Riscos

| ID | Risco | Probabilidade | Impacto | Nível | Mitigação |
|:---|:---|:---:|:---:|:---|:---|
| R01 | Consumidor externo (SIAFI) não suporta PIC X(14) | Média (40%) | Crítico (rejeição de registros) | **Crítico** | Validar contrato SIAFI antes de go-live; manter alternativa de coluna dupla |
| R02 | Falha do DFE durante execução batch | Baixa (5%) | Alto (fallback afeta dados) | **Alto** | Cache local reduz acessos; procedure de fallback documentada |
| R03 | Crescimento de dataset além de capacidade de disco | Média (35%) | Alto (abend de falta de espaço) | **Alto** | Provisionar 120% de espaço estimado; monitorar crescimento |
| R04 | Operação numérica não identificada sobre campo CNPJ | Baixa (10%) | Crítico (abend) | **Crítico** | Auditoria de código rigorosa antes de deploy |
| R05 | Cache VSAM saturado / performance degradada | Baixa (8%) | Médio (lentidão batch) | **Médio** | Políticas de invalidação (TTL, LRU) configuradas; monitoramento |
| R06 | Incompatibilidade com nova versão de COBOL/compilador | Baixa (5%) | Alto (necessário recompilação) | **Alto** | Testar recompilação em ambiente homologação antes de produção |
| R07 | Dados inconsistentes entre ambientes (dev/tst/prod) | Média (30%) | Médio (debugging difícil) | **Médio** | Sincronização de cache e dados via script batch; validação pré-deploy |

**Conclusão de Risco:** Nível geral **Alto**. Execução viável com mitigações documentadas. Risco R01 é crítico e requer validação externa (SIAFI).

---

## Rollback

### Estratégia de Versionamento

**Artefatos Versionados:**
- VIPP4365.cbl (versão anterior salva como backup nomeado)
- VIPP4553.cbl (versão anterior salva como backup nomeado)
- VIPF940S layout (documentado em comentário de mudança)
- VIPF104S layout (documentado em comentário de mudança)
- JCL de execução (versão anterior arquivada)

**Não será criada versão V2, ALT ou NEW dos artefatos.** Alterações serão in-place.

### Procedimento de Reversão

**Cenário 1 — Rollback Imediato (primeira 24h pós-deploy):**
1. Recuperar backup de código antigo de VIPP4365 e VIPP4553
2. Relinkedar binários antigos
3. Re-executar batch com código antigo
4. Validar saída (deve ser compatível com formato numérico)
5. Investigação de causa raiz

**Cenário 2 — Rollback Parcial (após 24h, alguns registros já processados):**
1. Executar batch paralelo com código novo e antigo (não afeta produção)
2. Comparar saídas; identificar registros divergentes
3. Avaliar se divergência é tolerável (ex.: apenas no campo CNPJ, que é novo)
4. Decidir: continuar com novo código ou reverter

**Tempo de Rollback:** 15-30 minutos (pós-deploy), até 2 horas (investigação)

---

## Go-Live

### Critérios de Prontidão

Antes de executar primeiro batch em produção:

- [ ] Teste de contrato externo (SIAFI) validado ✓
- [ ] Ambiente de homologação executado com sucesso ✓
- [ ] Cache VSAM provisionado e testado ✓
- [ ] Backup de código antigo realizado ✓
- [ ] Runbook operacional aprovado ✓
- [ ] Time de suporte (24/7) on-call ✓
- [ ] Dashboard de monitoramento configurado ✓
- [ ] Plano de rollback validado ✓

### Plano de Testes

| Fase | Escopo | Duração | Critério de Sucesso |
|:---|:---|:---|:---|
| Unitário | VIPP4365, VIPP4553 isolados; dados de teste reduzido | 1 semana | 100% de campos CNPJ convertidos via DFE |
| Integração | VIPP4365 + SIAFI; VIPP4553 + DAFS6452 | 1 semana | Contrato de arquivo validado (tamanho, conteúdo) |
| Carga | Dataset completo de produção; simulação 1 dia | 1 semana | Throughput >= baseline; sem abends |
| Regressão | Funcionalidade legada não alterada (campos não-CNPJ) | 1 semana | Output byte-for-byte idêntico à versão anterior |

### Plano de Monitoramento Pós-Implantação

**Primeira Semana (Crítica):**
- Monitoramento 24/7
- Alert instantâneo para taxa erro > 1%
- Daily review de logs de auditoria
- Contato diário com time SIAFI (se integrando)

**Segunda Semana (Vigilância):**
- Monitoramento 8x5 (horário comercial)
- Daily review reduzido
- Weekly review com stakeholders

**Terceira Semana em Diante (Operacional):**
- Monitoramento padrão (alertas automáticos)
- Investigação reativa (se erro)
- Monthly review de métricas

---

## Critérios de Sucesso

1. **Estabilidade Operacional:** Execução batch concluída sem abends; taxa de erro DFE < 1%
2. **Integridade de Dados:** Arquivo VIPF940S e VIPF104S gerado corretamente; consumidores externos (SIAFI) aceitam saída
3. **Compatibilidade Regressiva:** Campos não-CNPJ em saída byte-for-byte idênticos à versão anterior
4. **Performance Aceitável:** Tempo total de batch <= 120% da baseline (crescimento tolerável due a acessos DFE + cache)
5. **Rastreabilidade:** Logs de auditoria completos; cada CNPJ processado rastreável (Chave → Valor)

---

## Definition of Done

A execução é considerada **concluída** quando:

1. ✓ Código de VIPP4365 e VIPP4553 alterado, recompilado, deployado em produção
2. ✓ Cache VSAM operacional; batch em execução com DFE integrado
3. ✓ Arquivo VIPF940S e VIPF104S gerados em produção com campos alfanuméricos
4. ✓ Consumidor externo (SIAFI) validado; processando arquivos sem erro
5. ✓ Monitoramento de 24h validado; nenhum alerta crítico aberto
6. ✓ Rollback procedure testado e documentado
7. ✓ Knowledge transfer realizado; time de suporte treinado
8. ✓ Documentação (runbook, changelog) atualizada

---

## Conclusão

Este Plano de Execução detalha a estratégia operacional para adequação do CNPJ alfanumérico no Sistema VIP. A execução será realizada em 5 fases (Preparação, Execução x3, Consolidação) com progressão de complexidade crescente, iniciando por artefatos de menor acoplamento. 

**Pontos-Chave:**
1. **Classificação Determinística:** Todos os 10 campos CNPJ foram classificados com confiança 100%; execução desbloqueada
2. **Integração Crítica:** Conversão via Serviço Central de Conversão (DFE) é obrigatória; nenhuma geração local permitida
3. **Consumidor Externo:** SIAFI é consumidor crítico de VIPF940S; validação de contrato é pré-requisito de go-live
4. **Contingência:** Cache VSAM e fallback para Chave mitiga falha de DFE
5. **Risco Aceitável:** Impacto geral Alto-Crítico, mas viável com mitigações documentadas

**Próximos Passos:**
- [ ] Cliente valida Critérios de Prontidão
- [ ] Fase 1 (Preparação) iniciada
- [ ] Validação de SIAFI coordenada
- [ ] Ambiente de homologação preparado

---

**Documento Produzido por:** Agente Especializado em Arquitetura de Software
**Data de Geração:** 02/04/2026 03:06
**Status:** PRONTO PARA APROVAÇÃO DO CLIENTE
