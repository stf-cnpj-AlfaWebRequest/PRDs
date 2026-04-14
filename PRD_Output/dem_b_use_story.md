# User Stories do Projeto

## Introdução

Este documento apresenta as user stories para a adequação do sistema VIP de processamento de cartões pré-pagos ao suporte do CNPJ alfanumérico em ambiente mainframe. As histórias foram extraídas a partir da análise dos requisitos funcionais, do sistema existente e dos processos críticos de cadastro, cálculo de dígitos verificadores e enriquecimento de dados. O escopo concentra-se na separação obrigatória entre Chave de CNPJ (identificador técnico) e Valor de CNPJ (dado de negócio), preservando a integridade operacional e a rastreabilidade de dados ao longo de toda a cadeia de processamento.

---

## User Story 01 — Suportar Coexistência de Formatos Numérico e Alfanumérico de CNPJ

### Narrativa
Como um operador de processamento em lote do sistema VIP  
Eu quero que o sistema suporte simultaneamente CNPJ no formato numérico (14 dígitos) e alfanumérico (14 caracteres)  
Para que migrações graduais entre formatos não interrompam processos operacionais em produção

### Descrição
O sistema deve ser capaz de receber, processar e armazenar identificadores de pessoa jurídica em ambos os formatos simultaneamente. Esta capacidade é fundamental para viabilizar a transição de ambientes corporativos legados sem causar indisponibilidade. A coexistência garante que sistemas chamadores em diferentes estágios de migração possam interoperar com segurança.

### Critérios de Aceite
- Dado que um registro de entrada contém CNPJ no formato numérico (ex: 78465270000165)
  Quando o programa VIPP4365 ou VIPP4553 processa o registro
  Então o campo é aceito e processado sem erro de validação

- Dado que um registro de entrada contém CNPJ no formato alfanumérico (ex: 6QNJ8VY2JIC341)
  Quando o programa VIPP4365 ou VIPP4553 processa o registro
  Então o campo é aceito e processado sem erro de validação

- Dado que um lote contém registros com CNPJ em formato numérico e outros em formato alfanumérico
  Quando VIPP4365 ou VIPP4553 executa o processamento em lote
  Então todos os registros são processados com sucesso e arquivo de saída é gerado

- Dado que o campo de CNPJ no arquivo de entrada é numérico
  Quando o arquivo de saída é gerado
  Então o campo de CNPJ no arquivo de saída permanece no formato original recebido

### Contexto Adicional

Regras de negócio relevantes:
- A aceitação de ambos os formatos não implica conversão automática entre eles na saída; cada programa preserva o formato recebido
- Ambos os formatos ocupam 14 posições em disco, conforme definido nos layouts VIPK317D (VIPF317E) e F104E (VIPF104E)
- O sistema legado não realiza conversão local; conversões só são autorizadas pelo Serviço Central de Conversão (DFE)

Dependências funcionais:
- Requisito RF01 do PRD de requisitos alfanuméricos
- Compatibilidade com layouts de entrada VIPF317E e VIPF104E conforme definido nos artefatos de data layout

Premissas:
- O Serviço Central de Conversão (DFE) está disponível e operacional para futuras conversões
- Campos de CNPJ internos mantêm Chave de CNPJ durante processamento interno

### Anexos e Referências
- Especificação de layout VIPF317E (cartões pré-pagos) no overview do Sistema VIP
- Especificação de layout VIPF104E (custo de centro) no overview do Sistema VIP
- PRD de Requisitos: RF01 — Suporte à coexistência de formatos

### Subtarefas Técnicas
- Validar que estruturas de dados em VIPK317D aceitam ambos os formatos numéricos e alfanuméricos
- Revisar lógica de leitura de arquivo de entrada (VIPF317E) para aceitar caracteres não-numéricos
- Validar que estruturas de dados em F104E aceitam ambos os formatos numéricos e alfanuméricos
- Revisar lógica de leitura de arquivo de entrada (VIPF104E) para aceitar caracteres não-numéricos
- Verificar se tabelas DB2 (DB2VIP, DB2VIPA, DB2MCI) permitem armazenamento de CNPJ alfanumérico
- Adaptar exceções e tratamento de erros para reconhecer caracteres alfanuméricos como válidos
- Testar casos extremos: registros mistos, transições de formato dentro do mesmo lote

### Checklist INVEST

- Independent
  - Pode ser desenvolvida de forma independente, validando apenas a aceitação de ambos os formatos sem exigir integração com sistemas externos neste estágio

- Negotiable
  - Permite discussão sobre escopo de validação de dígito verificador (DV) para formato alfanumérico, se aplicável ao contexto de negócio

- Valuable
  - Entrega valor direto ao negócio ao viabilizar coexistência de ambientes em diferentes estágios de migração, reduzindo risco operacional

- Estimable
  - Possui escopo claro: aceitar e processar dois formatos distintos com critérios de validação bem definidos

- Small
  - Pode ser concluída dentro de uma sprint através de validação de layouts e testes de aceitação

- Testable
  - Pode ser validada através de testes de aceitação com arquivos de entrada contendo registros em ambos os formatos, verificando saída esperada

---

## User Story 02 — Classificar e Separar Chave de CNPJ de Valor de CNPJ em Processamento Interno

### Narrativa
Como um analista de dados do sistema de cartões pré-pagos  
Eu quero que cada campo de CNPJ seja explicitamente classificado como Chave de CNPJ (identificador técnico) ou Valor de CNPJ (dado de negócio)  
Para que o processamento interno aplique regras corretas de validação, transformação e propagação de dados

### Descrição
Toda ocorrência de campo de CNPJ ou CGC no sistema deve ter sua semântica clara e documentada: se representa um identificador técnico para acesso a dados (Chave) ou um identificador de negócio utilizado em relatórios, interfaces e cálculos (Valor). Esta clareza é essencial para garantir que a evolução para CNPJ alfanumérico não introduza ambiguidades que comprometam integridade de dados ou conformidade regulatória.

### Critérios de Aceite
- Dado que um campo de CNPJ é utilizado para relacionamento entre tabelas DB2 (ex: chave estrangeira em FK)
  Quando o campo é analisado em contexto de carga processada
  Então o campo é classificado como Chave de CNPJ e documentado como tal

- Dado que um campo de CNPJ é exibido em arquivo de saída ou interface de negócio (ex: VIPF940S-CNPJ-DET)
  Quando o campo é analisado em contexto de comunicação
  Então o campo é classificado como Valor de CNPJ e documentado como tal

- Dado que um campo de CNPJ é utilizado simultaneamente como chave de acesso e como informação de saída
  Quando o campo é analisado
  Então o campo é classificado como ambos Chave e Valor de CNPJ, com justificativa técnica explícita

- Dado que um campo de CNPJ é identificado em código legado cuja função não pode ser determinada com segurança
  Quando análise de contexto é insuficiente
  Então o campo é classificado como Indefinido, marcado para revisão manual e não é alterado automaticamente

### Contexto Adicional

Regras de negócio relevantes:
- Chave de CNPJ é identificador técnico interno (numérico, 14 posições, sem validação de dígito verificador)
- Valor de CNPJ é identificador de negócio (numérico ou alfanumérico, 14 posições, sujeito a validação de dígito verificador)
- A mesma tabela ou programa pode conter ambas as classificações em campos distintos
- Classificação deve ser realizada para cada campo individualmente, sem assumir uniformidade por tabela ou programa

Dependências funcionais:
- Requisito RF02 e RF04 do PRD de requisitos alfanuméricos
- Análise completa de todas as tabelas DB2, layouts de entrada/saída, copybooks e instruções MOVE

Premissas:
- O Serviço Central de Conversão (DFE) é responsável por manter correlação 1:1 entre Chave e Valor
- Campos classificados como Indefinidos serão tratados em iteração posterior com envolvimento de especialistas de negócio

Observações de uso:
- Presença de sufixos como `-DET`, `-OUT`, `-SAI`, `-RET`, `-CNPJ` em nomes de campo são indicadores de uso como Valor de CNPJ
- Participação em cláusulas JOIN, MATCH ou índices é indicador forte de uso como Chave de CNPJ

### Anexos e Referências
- PRD de Requisitos: RF02 — Separação obrigatória entre Chave de CNPJ e Valor de CNPJ
- PRD de Requisitos: RF04 — Adaptação obrigatória de todos os pontos de uso
- Documentação técnica: Regras Mandatória de Explicitação de Consumo Subsequente (PRD requisito)

### Subtarefas Técnicas
- Identificar todos os campos de CNPJ e CGC em tabelas DB2 utilizadas por VIPP4365 e VIPP4553
- Classificar cada campo como Chave, Valor ou Acumulativo (ambos) com evidência técnica justificada
- Documentar classificação em matriz de rastreabilidade incluindo: nome do campo, artefato, contexto de uso, classificação
- Revisar layoius de entrada (VIPK317D, F104E) e classificar cada campo de CNPJ
- Revisar layouts de saída (VIPF940S, VIPF104S) e classificar cada campo de CNPJ com identificação de consumidor subsequente
- Revisar copybooks utilizados por VIPP4365 e VIPP4553 (VIPK160D, etc.) e classificar campos de CNPJ
- Validar que nenhum campo permanece sem classificação ou com classificação ambígua

### Checklist INVEST

- Independent
  - Pode ser desenvolvida como atividade de análise e documentação sem bloqueadores externos

- Negotiable
  - Permite discussão sobre escopo de análise (quantos programas satélites incluir) e critério de confiança para classificação

- Valuable
  - Entrega valor crítico ao negócio ao eliminar ambiguidades que poderiam comprometer migração para CNPJ alfanumérico

- Estimable
  - Possui escopo claro: análise sistemática de todos os campos de CNPJ com critérios de classificação bem definidos

- Small
  - Pode ser concluída dentro de uma sprint com suporte de especialista de banco de dados

- Testable
  - Pode ser validada através de revisão de rastreabilidade e validação de completude da matriz de classificação

---

## User Story 03 — Validar Valor de CNPJ sem Alterar Comportamento de Chave de CNPJ

### Narrativa
Como um especialista de integridade de dados do banco  
Eu quero que o sistema aplique validação de dígito verificador apenas a Valor de CNPJ em entrada  
Para que dados tecnicamente válidos (Chaves de CNPJ) não sejam rejeitados por regras de negócio inaplicáveis

### Descrição
Chaves de CNPJ, por serem identificadores técnicos internos, não devem ser submetidas a validação de dígito verificador ou formatação de documento. Valores de CNPJ, por serem dados de negócio, devem ser validados conforme regras regulatórias. Esta distinção evita rejeição incorreta de dados legítimos e preserva a integridade de fluxos existentes.

### Critérios de Aceite
- Dado que um campo é classificado como Valor de CNPJ e entra via interface externa
  Quando o campo é processado
  Então é aplicada validação de dígito verificador e campo é rejeitado se inválido

- Dado que um campo é classificado como Chave de CNPJ e entra como chave estrangeira
  Quando o campo é processado
  Então nenhuma validação de dígito verificador é aplicada e campo é aceito se existir na correlação 1:1

- Dado que um campo é classificado como Valor de CNPJ e entra em arquivo de entrada
  Quando o campo é lido do arquivo
  Então é aplicada validação conforme regras de negócio do PRD

- Dado que um campo é classificado como Chave de CNPJ e entra em arquivo de entrada
  Quando o campo é lido do arquivo
  Então é aceito sem validação de dígito verificador

### Contexto Adicional

Regras de negócio relevantes:
- Chave de CNPJ não requer validação (é apenas referência técnica)
- Valor de CNPJ deve ser validado conforme algoritmo de dígito verificador CNPJ
- Para CNPJ alfanumérico, a validação de dígito verificador pode usar alfabeto estendido conforme padrão do Serviço Central de Conversão

Dependências funcionais:
- Requisito RF05 do PRD de requisitos alfanuméricos
- Subroutina SBDIGITO já disponível no ambiente legado
- Classificação de campos do CNPJ (História anterior)

Premissas:
- Algoritmo de validação para CNPJ alfanumérico será fornecido pelo Serviço Central de Conversão (DFE)
- Campos numéricos continuam utilizando algoritmo SBDIGITO existente

Observações de uso:
- VIPP4553 já utiliza SBDIGITO para cálculo de dígito verificador de agência e conta
- Lógica de validação deve ser aplicada em ponto de entrada (arquivo ou interface), não em saída

### Anexos e Referências
- PRD de Requisitos: RF05 — Tratamento distinto em entradas e saídas
- Overview do Sistema VIP: subroutina SBDIGITO

### Subtarefas Técnicas
- Identificar todos os pontos de entrada de Valor de CNPJ em VIPP4365 e VIPP4553
- Implementar lógica de validação de dígito verificador para Valor de CNPJ em entrada
- Garantir que Chaves de CNPJ não passam por validação de dígito
- Revisar e atualizar exceções para diferenciar erro de validação de erro de ausência de chave
- Testar validação com Valor de CNPJ válido e inválido
- Testar que Chaves de CNPJ não são rejeitadas por falha de validação

### Checklist INVEST

- Independent
  - Pode ser desenvolvida após classificação de campos de CNPJ, com validação independente de outros componentes

- Negotiable
  - Permite discussão sobre ponto exato de aplicação de validação (entrada ou processamento) e tratamento de erros

- Valuable
  - Entrega valor ao negócio ao aplicar regras corretas sem rejeitar dados legítimos

- Estimable
  - Possui escopo claro: identificar pontos de entrada e aplicar validação diferenciada

- Small
  - Pode ser concluída dentro de uma sprint

- Testable
  - Pode ser validada através de testes de aceitação com dados válidos e inválidos

---

## User Story 04 — Identificar Consumidores Subsequentes de Valor de CNPJ em Saída

### Narrativa
Como um arquiteto de integração de sistemas  
Eu quero que cada Valor de CNPJ propagado em estrutura de saída seja rastreado até seus consumidores subsequentes  
Para que mudanças de formato sejam implementadas com compreensão completa do impacto downstream

### Descrição
Campos de CNPJ em estruturas de saída (arquivos, detalhes, interfaces) não existem isoladamente; eles alimentam sistemas subsequentes, relatórios, auditorias e interfaces externas. A identificação explícita de consumidores (que programa ou sistema consome o campo, como o utiliza, em qual artefato) é crítica para garantir que nenhuma cadeia de impacto seja deixada implícita ou esquecida durante migração.

### Critérios de Aceite
- Dado que um Valor de CNPJ é gravado em arquivo de saída (ex: VIPF940S)
  Quando o arquivo é analisado
  Então é identificado cada programa ou sistema que lê o arquivo subsequentemente

- Dado que um Valor de CNPJ é propagado em estrutura de detalhe
  Quando o campo é analisado
  Então é documentado se o campo é: exibido, transmitido, persistido, validado, transformado, replicado, conciliado ou propagado

- Dado que um Valor de CNPJ é utilizado em interface entre programas
  Quando a interface é analisada
  Então é documentado o contrato técnico em que o campo trafega (copybook, linkage section, etc.)

- Dado que múltiplos consumidores leem Valor de CNPJ da mesma saída
  Quando consumidores são identificados
  Então cada consumidor é listado nominativamente com sua função e impacto

### Contexto Adicional

Regras de negócio relevantes:
- Toda saída de Valor de CNPJ deve ter rastreabilidade até ponto de consumo final
- Cadeia de consumo pode incluir: programas COBOL subsequentes, steps de JCL, transações CICS, rotinas batch, tabelas DB2, mensagens MQ, APIs, sistemas satélites, módulos de integração, rotinas de relatório/auditoria/conciliação

Dependências funcionais:
- Requisito de Explicitação de Consumo Subsequente do PRD de requisitos alfanuméricos
- Classificação anterior de Valor de CNPJ
- Conhecimento de fluxos downstream de VIPP4365 e VIPP4553

Premissas:
- Documentação de integrações downstream está disponível ou pode ser obtida através de análise de código
- Sistemas externos estão identificados na documentação de integrações

Observações de uso:
- VIPF940S é saída de VIPP4365; deve ser analisado quem lê VIPF940S subsequentemente
- VIPF104S é saída de VIPP4553; deve ser analisado quem lê VIPF104S subsequentemente
- Camadas de cache (Redis, VSAM, batch) são consumidores potenciais

### Anexos e Referências
- PRD de Requisitos: Regra Mandatória de Explicitação de Consumo Subsequente
- Overview do Sistema VIP: VIPF940S e VIPF104S

### Subtarefas Técnicas
- Mapear consumidores de arquivo VIPF940S (saída de VIPP4365)
- Mapear consumidores de arquivo VIPF104S (saída de VIPP4553)
- Documentar para cada consumidor: nome do artefato, tipo de consumo, impacto da mudança para CNPJ alfanumérico
- Revisar integrações com sistemas DAF, SIAFI, DB2 para identificar consumo de CNPJ
- Revisar integrações com camadas de cache para identificar consumo de CNPJ
- Revisar relatórios e interfaces externas para identificação de Valores de CNPJ propagados
- Validar que nenhum consumidor downstream é deixado sem documentação

### Checklist INVEST

- Independent
  - Pode ser desenvolvida como atividade de análise e documentação

- Negotiable
  - Permite discussão sobre profundidade de rastreamento (apenas consumidores diretos vs. toda cadeia transitiva)

- Valuable
  - Entrega valor crítico ao negócio ao prevenir surpresas downstream durante implementação

- Estimable
  - Possui escopo claro: rastrear e documentar consumidores

- Small
  - Pode ser concluída dentro de uma sprint com envolvimento de arquitetos de integração

- Testable
  - Pode ser validada através de revisão de rastreabilidade e validação de completude da cadeia de consumo

---

## User Story 05 — Preservar Fluxos de Negócio Existentes Sem Alterações Funcionais

### Narrativa
Como um gerente de operações do sistema VIP  
Eu quero que a adaptação para CNPJ alfanumérico não altere o comportamento funcional de processos existentes  
Para que risco operacional seja minimizado e competência de time seja preservada

### Descrição
A introdução de suporte ao CNPJ alfanumérico não deve exigir renegociação de regras de negócio, ajuste de fluxos críticos ou modificação de contatos entre programas. Apenas o tratamento do identificador muda; lógica de negócio permanece invariável. Esta história garante que o escopo de mudança seja circunscrito e rastreável.

### Critérios de Aceite
- Dado que um fluxo de negócio existente utiliza CNPJ em processamento
  Quando adaptação para CNPJ alfanumérico é implementada
  Então fluxo permanece funcionalmente idêntico (mesmos cálculos, mesmas validações, mesma sequência de passos)

- Dado que VIPP4365 processa cartões pré-pagos agrupados por EPRL
  Quando suporte a CNPJ alfanumérico é adicionado
  Então agrupamento por EPRL permanece inalterado, assim como formato de Header/Detail/Trailer

- Dado que VIPP4553 calcula dígitos verificadores e enriquece restrições de cartão
  Quando suporte a CNPJ alfanumérico é adicionado
  Então cálculo de dígitos e enriquecimento de restrições funcionam idêntico, modificadas apenas regras de entrada de Valor de CNPJ

- Dado que programa satélite (ex: DAF, HLP) consome saída de VIPP4365 ou VIPP4553
  Quando suporte a CNPJ alfanumérico é adicionado
  Então programa satélite recebe campos em formato idêntico, sem exigência de mudança funcional

### Contexto Adicional

Regras de negócio relevantes:
- Nenhuma regra de negócio é alterada; apenas representação técnica de CNPJ muda
- Fluxos de erro, tratamento de exceções e contadores permanecem idênticos
- Integrações com DB2, DAF, SIAFI permanecem inalteradas em lógica

Dependências funcionais:
- Requisito RF03 do PRD de requisitos alfanuméricos
- Conhecimento detalhado de regras de negócio de VIPP4365 e VIPP4553

Premissas:
- Serviço Central de Conversão (DFE) oferece interfaces transparentes para programas legados
- Layouts de entrada/saída suportam ambos os formatos de CNPJ

Observações de uso:
- Mudanças devem ser limitadas a: recepção de entrada, validação, acesso a tabelas de correlação, saída
- Mudanças não devem afetar: cálculo de dígitos de agência/conta, classificação NR/RC, enriquecimento de restrições, contadores

### Anexos e Referências
- PRD de Requisitos: RF03 — Preservação dos fluxos de negócio
- Overview do Sistema VIP: regras de negócio de VIPP4365 e VIPP4553

### Subtarefas Técnicas
- Documentar regras de negócio atuais de VIPP4365 (agrupamento EPRL, cálculo NR/RC, validações)
- Documentar regras de negócio atuais de VIPP4553 (cálculo de dígitos, enriquecimento de restrições, integração DAF)
- Validar que nenhuma regra de negócio é modificada; apenas entrada/saída de CNPJ
- Testar que contadores e relatórios produzem números idênticos para dados equivalentes
- Testar que arquivo de saída (Header/Detail/Trailer) mantém estrutura e semântica idêntica

### Checklist INVEST

- Independent
  - Pode ser validada de forma independente através de testes de regressão

- Negotiable
  - Permite discussão sobre definição de "funcionalmente idêntico" em casos limítrofes

- Valuable
  - Entrega valor ao negócio ao reduzir risco e custo de mudança

- Estimable
  - Possui escopo claro: validação de invariância de regras de negócio

- Small
  - Pode ser concluída dentro de uma sprint como atividade de teste e validação

- Testable
  - Pode ser validada através de testes de regressão e comparação de saídas antes/depois

---

## User Story 06 — Usar Exclusivamente Valor de CNPJ em Interfaces Externas

### Narrativa
Como um responsável por conformidade regulatória  
Eu quero que apenas Valor de CNPJ seja exposto em interfaces externas e relatórios  
Para que conformidade regulatória seja garantida e identificadores técnicos internos permaneçam privados

### Descrição
Chaves de CNPJ são artefatos técnicos internos do mainframe e não devem ser expostos a entidades externas. Todas as interfaces externas (APIs, Web Services, EDI, relatórios com dados de terceiros, transmissões regulatórias) devem exibir apenas Valor de CNPJ, garantindo que dados confidenciais e tecnicamente específicos não vazem para fora da organização.

### Critérios de Aceite
- Dado que uma API externa solicita dados de cartão pré-pagado
  Quando resposta é montada
  Então apenas Valor de CNPJ é retornado, nunca Chave de CNPJ

- Dado que um relatório regulatório é gerado contendo dados de pessoa jurídica
  Quando relatório é exportado
  Então apenas Valor de CNPJ é incluído na saída

- Dado que um arquivo de remessa para sistema externo é criado
  Quando arquivo é transmitido
  Então apenas Valor de CNPJ é incluído, não Chave de CNPJ

- Dado que programa satélite recebe dados do VIPP4365 ou VIPP4553
  Quando programa satélite é externo ao escopo desta modificação
  Então contrato entre programas inclui apenas Valor de CNPJ (ou Chave claramente separada)

### Contexto Adicional

Regras de negócio relevantes:
- Proibição absoluta de exposição de Chave de CNPJ em saída
- Valor de CNPJ é identificador oficial de negócio e pode ser exposto
- Conformidade regulatória exige que apenas Valor de CNPJ seja visível para reguladores e terceiros

Dependências funcionais:
- Requisito RF06 do PRD de requisitos alfanuméricos
- Classificação anterior de Chave vs. Valor de CNPJ
- Identificação de interfaces externas

Premissas:
- Interfaces externas já mapeadas
- Serviço Central de Conversão fornece mecanismo de acesso a Valor de CNPJ a partir de Chave

Observações de uso:
- Programa satélite dentro da organização pode receber ambos (Chave e Valor), desde que claramente separados
- Entidade completamente externa não pode ter acesso a Chave

### Anexos e Referências
- PRD de Requisitos: RF06 — Uso exclusivo do Valor de CNPJ em interfaces externas

### Subtarefas Técnicas
- Identificar todas as interfaces externas que comunicam dados contendo CNPJ
- Revisar e atualizar cada interface externa para expor apenas Valor de CNPJ
- Revisar APIs (se existentes) para garantir que apenas Valor é retornado
- Revisar relatórios regulatórios para garantir que apenas Valor é incluído
- Revisar arquivos de remessa para garantir que apenas Valor é transmitido
- Validar contrato de interface com programas satélites para clareza de Chave vs. Valor
- Testar interfaces externas para confirmar que Chave não é exposta

### Checklist INVEST

- Independent
  - Pode ser desenvolvida após identificação de interfaces externas

- Negotiable
  - Permite discussão sobre definição de "interface externa" em casos limítrofes (programa satélite do mesmo banco vs. terceiro)

- Valuable
  - Entrega valor crítico ao negócio ao garantir conformidade regulatória e privacidade de dados técnicos

- Estimable
  - Possui escopo claro: revisar e atualizar interfaces para exibir apenas Valor de CNPJ

- Small
  - Pode ser concluída dentro de uma sprint

- Testable
  - Pode ser validada através de testes de interfaces externas confirmando que Chave não é exposta

---

## Conclusão

As user stories apresentadas consolidam requisitos funcionais para adequação do sistema VIP ao suporte de CNPJ alfanumérico em ambiente mainframe. O conjunto de histórias estabelece a separação obrigatória entre Chave de CNPJ (identificador técnico interno) e Valor de CNPJ (dado de negócio), preservando a integridade operacional, a rastreabilidade de dados e a conformidade regulatória. Cada história é orientada ao valor de negócio, contém critérios de aceite verificáveis e evita prescrição de implementação técnica, permitindo que o time de desenvolvimento mantenha autonomia arquitetural. O escopo está limitado aos comportamentos descritos nas fontes de verdade (PRD de requisitos e overview do sistema), sem extrapolação ou invenção de requisitos.
