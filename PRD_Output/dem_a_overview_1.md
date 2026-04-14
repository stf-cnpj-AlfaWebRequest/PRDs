# Overview - Projeto Demanda CPNJ Alfanumérico

## 1. Visão Geral

O Sistema VIP é um conjunto integrado de programas COBOL em ambiente mainframe IBM z/OS dedicado ao processamento em lote de cartões pré-pagos para pessoas jurídicas. O sistema consolida dados cadastrais de plásticos e contas de cartão a partir de múltiplas fontes (tabelas DB2, arquivos sequenciais, consultas ao subsistema DAF), enriquecendo-os com informações técnicas como dígitos verificadores de agência/conta e descrições de restrições de cartão. A arquitetura segue o padrão sequencial de lote com estrutura de arquivo padronizada em cabeçalho, detalhe e rodapé. O escopo deste projeto está focado na adequação do tratamento de CNPJ para suportar simultaneamente o formato numérico legado (Chave de CNPJ) e o formato alfanumérico (Valor de CNPJ), mantendo a separação semântica entre identificadores técnicos e dados de negócio.

## 2. Resumo Executivo

O projeto Demanda CNPJ Alfanumérico representa uma transformação no tratamento de identificadores de pessoa jurídica dentro do Sistema VIP. Historicamente, o CNPJ era tratado como um identificador único, indiferenciado e numérico. A demanda introduz uma dicotomia arquitetural essencial: a separação obrigatória entre Chave de CNPJ (identificador técnico interno, numérico, 14 dígitos) e Valor de CNPJ (identificador de negócio, alfanumérico, 14 caracteres). Esta separação exige revisão dos contratos de dados (copybooks, layouts de arquivo, estruturas DB2), implementação de conversão centralizada via serviço de correlação (DFE), e garantia de determinismo nas transformações. Os programas VIPP4365 e VIPP4553 serão modificados de forma pontual para reconhecer e manipular ambas as formas, sem alteração estrutural do fluxo de processamento existente.

## 3. Objetivo do Sistema

O Sistema VIP atende ao seguinte objetivo de negócio: gerar e manter arquivos cadastrais padronizados de cartões pré-pagos para pessoas jurídicas, permitindo que sistemas downstream (operacionais, de reporte, de integração) processem informações completas e enriquecidas sobre titulares, contas, agências e restrições. O projeto Demanda CNPJ Alfanumérico complementa este objetivo ao garantir que o tratamento de identificadores legais seja compatível com a evolução regulatória e tecnológica da instituição financeira, suportando a coexistência de identificadores numéricos legados com novos formatos alfanuméricos, sem ambiguidade semântica e com rastreabilidade total de conversões.

## 4. Escopo

**Incluído:**

- Revisão e redefinição de campos de CNPJ em programas COBOL (VIPP4365, VIPP4553)
- Revisão de copybooks compartilhados que declaram estruturas de CNPJ (VIPK317D, VIPK160D, DAFK6452)
- Modificação de layouts de arquivo de entrada (VIPF317E, VIPF104E) e saída (VIPF940S, VIPF104S)
- Implementação de lógica de classificação explícita: Chave de CNPJ vs. Valor de CNPJ
- Integração com serviço central de conversão (DFE) quando necessário
- Validação de dígito verificador aplicada exclusivamente ao Valor de CNPJ
- Consultas e enriquecimento de dados a partir de tabelas DB2 (DB2VIP, DB2VIPA, DB2MCI)
- Garantia de que dados externos (origem em DB2MCI.CLIENTE, DAF) sejam preservados conforme recebidos
- Documentação de consumidores subsequentes de campos de CNPJ em saídas

**Não incluído:**

- Refatoração estrutural ou estética de código não relacionado a CNPJ
- Modernização de arquitetura ou padrões de programação
- Alteração de JCL ou parâmetros de execução fora do escopo de data/hora
- Modificação de subroutines compartilhadas (SBVERSAO, SBABEND) que não tratem diretamente de CNPJ
- Mudanças em formatos de erro ou tratamento de exceção além do necessário para CNPJ
- Eliminação de campos ou estruturas legadas (apenas reclassificação semântica)
- Dados provenientes de fontes externas não devem sofrer transformação ou normalização

## 5. Glossário

> Tabela 1 — Termos técnicos e conceituais utilizados no projeto Demanda CNPJ Alfanumérico, com definições que garantem consistência de vocabulário e rastreabilidade nas discussões de classificação, conversão e integração de dados.

| Termo | Definição |
|-------|-----------|
| Chave de CNPJ | Identificador técnico interno, numérico, 14 posições (PIC 9(14) em COBOL). Utilizado exclusivamente para relacionamentos, índices, buscas e navegação de dados dentro do sistema. Nunca exposto externamente ou em relatórios. Exemplo: 00000000000001. |
| Valor de CNPJ | Identificador de negócio, alfanumérico, 14 caracteres (PIC X(14) em COBOL). Representa o documento oficial de pessoa jurídica. Utilizado em interfaces externas, relatórios, telas e validações. Exemplo: 6QNJ8VY2JIC341. |
| Classificação | Processo de análise técnica de um campo de CNPJ para determinar se atua como Chave de CNPJ, Valor de CNPJ, ou ambos (classificação acumulativa). Baseada em contexto de uso, participação em operações técnicas e semântica de negócio. |
| Indefinido | Estado de classificação para campos cuja natureza semântica não pode ser determinada com confiança. Campos Indefinidos não sofrem transformação ou validação de DV. |
| Correlação 1:1 | Relação obrigatória e determinística entre uma Chave de CNPJ e um Valor de CNPJ, rastreável e auditável. Garantida pelo serviço central de conversão (DFE). |
| Serviço Central de Conversão | Sistema autorizado (DFE) responsável por converter Chave de CNPJ em Valor de CNPJ e vice-versa. Proibida geração local ou distribuída desta correlação. |
| Base Central de Correlação (DFE) | Estrutura centralizada que mantém a relação 1:1 entre Chave de CNPJ e Valor de CNPJ, com metadados de controle e auditoria. |
| Cache | Camadas auxiliares de desempenho (Redis, VSAM, processamento batch) que reduzem acessos à base central. Deve ser consistente com DFE. |
| Dígito Verificador (DV) | Dígito de validação matemática aplicado ao Valor de CNPJ. Calculado apenas quando o campo representa Valor de CNPJ. Implementado via subroutine SBDIGITO (função 011). |
| Fonte Externa | Sistema ou tabela fora do escopo desta modificação (ex.: DB2MCI.CLIENTE). Dados provenientes de fontes externas devem ser preservados conforme recebido, sem transformação. |
| Módulo Principal | Artefato de código (programa COBOL, copybook) pertencente ao sistema VIP. Identificado dinamicamente pelo prefixo (ex.: VIP, VIPK, VIPF). |
| Módulo Secundário | Artefato compartilhado com outros sistemas (ex.: DAF, HLP, SBDIGITO). Pode estar sujeito a mudanças em outros projetos. |
| Artefato | Qualquer unidade técnica versionável que compõe ou influencia a execução: programas (.cbl), copybooks (.cpy), JCLs, layouts, tabelas DB2, estruturas de dados. |
| Enriquecimento de Dados | Processo de agregar informações complementares a um registro (ex.: descrição de restrição de cartão a partir de tabela DB2). Deve respeitar a semântica de CNPJ de cada campo origem. |
| Consumidor Subsequente | Sistema, programa, rotina ou artefato que recebe e processa um campo de CNPJ após sua saída de um programa ou módulo. Deve ser explicitamente identificado e documentado. |
| Inferência Determinística | Análise que permite classificar um campo de CNPJ com alta confiança, baseada em evidências técnicas claras. |
| Inferência Fraca | Análise com incerteza quanto à natureza semântica do campo. Resulta em classificação Indefinido. |
| CGC | Denominação anterior do CNPJ (Cadastro Geral de Contribuintes). Campos nomeados CGC devem ser tratados com as mesmas regras aplicáveis ao CNPJ. |
| PJ | Pessoa Jurídica. Entidade legal de negócio. Identificada por CNPJ. Foco deste projeto. |
| PF | Pessoa Física. Indivíduo. Identificado por CPF. Fora do escopo desta demanda. |
| Processamento em Lote | Execução batch de programa COBOL sob controle JCL, lendo arquivo sequencial de entrada e produzindo arquivo de saída. Padrão do Sistema VIP. |
| Cabeçalho (Header) | Primeiro registro de um arquivo ou grupo de registros, contendo metadados (data, hora, contagem, identificadores). Tipo de registro = 0. |
| Detalhe | Registros de conteúdo principal, portadores de dados de negócio. Tipo de registro = 1. |
| Rodapé (Trailer) | Último registro de um arquivo ou grupo, contendo resumo (contagem total de registros). Tipo de registro = 9. |
| DCLGEN | Ferramenta DB2 que gera copybooks COBOL a partir de definições de tabela. Utilizados para host variables em EXEC SQL. |
| Copybook | Arquivo COBOL (.cpy) contendo definições de estruturas de dados reutilizáveis. Incluído via diretiva =INC em programas. |
| COMP, COMP-3, COMP-5 | Tipos de compactação COBOL. COMP-3 = empacotado decimal (2 dígitos por byte). Usado para campos numéricos em DB2 e arquivos compactados. |
| PIC 9(14), PIC X(14) | Notação PIC (Picture) COBOL. 9 = dígito numérico. X = caractere alfanumérico. |
| Ambiente IBM z/OS | Mainframe IBM OS. Compilador COBOL SOS 13. Suporta DB2, VSAM, JCL, CICS. Ambiente padrão do Sistema VIP. |

---

## 6. Rodapé

---

Documento Elaborado: 01/04/2026

Versão: 5.0

---
