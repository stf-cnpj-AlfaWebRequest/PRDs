# Overview - Projeto Demanda CPNJ Alfanumérico

## 1. Visão Geral

O Sistema VIP é um conjunto integrado de programas COBOL em ambiente mainframe IBM z/OS dedicado ao processamento em lote de cartões pré-pagos para pessoas jurídicas, consolidando dados cadastrais de plásticos, contas de cartão e informações de agências a partir de múltiplas fontes (tabelas DB2, arquivos sequenciais, consultas ao subsistema DAF). A arquitetura segue o padrão sequencial de lote com estrutura padronizada de cabeçalho, detalhe e rodapé, garantindo rastreabilidade de cada etapa de processamento. O programa principal VIPP4365 lê dados de cartões pré-pagos do arquivo VIPF317E, enriquece-os com informações técnicas de agências, contas e restrições, e gera o arquivo consolidado VIPF940S. O programa secundário VIPP4553 processa dados de contas correntes do arquivo VIPF104E, validando e formatando informações de CNPJ, beneficiários e agências, produzindo a saída enriquecida VIPF104S. O escopo deste projeto está focado na adequação do tratamento de CNPJ para suportar simultaneamente o formato numérico legado (Chave de CNPJ) e o formato alfanumérico (Valor de CNPJ), mantendo a separação semântica entre identificadores técnicos internos e dados de negócio.

## 2. Resumo Executivo

O projeto Demanda CNPJ Alfanumérico representa uma transformação no tratamento de identificadores de pessoa jurídica dentro do Sistema VIP. Historicamente, o CNPJ era manipulado como um identificador único, indiferenciado e numérico. A demanda introduz uma dicotomia arquitetural essencial: a separação obrigatória entre Chave de CNPJ (identificador técnico interno, numérico, 14 dígitos) e Valor de CNPJ (identificador de negócio, alfanumérico, 14 caracteres). Esta separação exige revisão dos contratos de dados (copybooks, layouts de arquivo), implementação de conversão centralizada via serviço de correlação (DFE), aplicação de validação de dígito verificador apenas ao Valor de CNPJ, e garantia de determinismo nas transformações. Os programas VIPP4365 e VIPP4553 serão modificados de forma pontual para reconhecer e manipular ambas as formas sem alteração estrutural do fluxo de processamento existente. As alterações mantêm compatibilidade com dados externos provenientes de tabelas DB2 do subsistema MCI, garantindo que valores recebidos sejam preservados conforme originais. A integração com a subroutine SBDIGITO (função 011) fornecerá validação centralizada de dígito verificador para Valor de CNPJ, enquanto a conversão bidirecional será realizada exclusivamente via serviço central (DFE).

## 3. Objetivo do Sistema

O Sistema VIP atende ao seguinte objetivo de negócio: gerar e manter arquivos cadastrais padronizados de cartões pré-pagos e contas correntes para pessoas jurídicas, permitindo que sistemas downstream (operacionais, de reporte, de integração) processem informações completas, enriquecidas e mutuamente consistentes sobre titulares, contas, agências, beneficiários e restrições. O projeto Demanda CNPJ Alfanumérico complementa este objetivo ao garantir que o tratamento de identificadores legais seja compatível com a evolução regulatória e tecnológica da instituição financeira, suportando a coexistência de identificadores numéricos legados com novos formatos alfanuméricos, sem ambiguidade semântica, com rastreabilidade completa de conversões, e com preservação integral de dados originários de fontes externas que não integram o escopo de transformação.

## 4. Escopo

**Incluído:**

- Análise e redefinição de campos de CNPJ nos programas COBOL principais (VIPP4365, VIPP4553)
- Classificação explícita de cada ocorrência de CNPJ como Chave de CNPJ ou Valor de CNPJ
- Revisão de copybooks compartilhados que declaram ou utilizam estruturas de CNPJ (VIPK317D, VIPK160D, DAFK6452)
- Modificação de layouts de arquivo de entrada (VIPF317E, VIPF104E) e saída (VIPF940S, VIPF104S)
- Implementação de lógica de tratamento diferenciado: Chave de CNPJ versus Valor de CNPJ
- Integração com serviço central de conversão (DFE) para transformações entre Chave e Valor
- Aplicação de validação de dígito verificador exclusivamente ao Valor de CNPJ via subroutine SBDIGITO (função 011)
- Consultas e enriquecimento de dados a partir de tabelas DB2 (DB2VIP, DB2VIPA) com identificação explícita de CNPJ em cada contexto
- Preservação integral de dados externos provenientes de tabelas DB2MCI.CLIENTE e consultas ao subsistema DAF, conforme recebidos
- Documentação de consumidores subsequentes de campos de CNPJ em estruturas de saída (VIPF940S, VIPF104S)
- Análise de impacto em estruturas intermediárias como D6452I-CNPJ e D6452S-CNPJ (copybook DAFK6452)

**Não incluído:**

- Refatoração estrutural ou estética de código não relacionado a CNPJ
- Modernização de arquitetura ou padrões de programação COBOL
- Alteração de JCL ou parâmetros de execução fora do escopo de data/hora e movimentação de CNPJ
- Modificação de subroutines compartilhadas (SBVERSAO, SBABEND) que não tratem diretamente de CNPJ
- Mudanças em formatos de erro ou tratamento de exceção além do necessário para CNPJ
- Eliminação de campos ou estruturas legadas (apenas reclassificação semântica)
- Transformação de dados provenientes de fontes externas (DB2MCI.CLIENTE, DAF)
- Alteração de operações matemáticas, de data ou outras lógicas de negócio não relacionadas a CNPJ

## 5. Glossário

> Tabela 1 — Termos técnicos e conceituais utilizados no projeto Demanda CNPJ Alfanumérico, com definições que garantem consistência de vocabulário, rastreabilidade nas discussões de classificação, conversão, integração de dados e identificação de consumidores subsequentes.

| Termo | Definição |
|-------|-----------|
| Chave de CNPJ | Identificador técnico interno, numérico, 14 posições (PIC 9(14) em COBOL). Utilizado exclusivamente para relacionamentos, índices, buscas, validações de integridade e navegação de dados dentro do sistema. Nunca exposto externamente, em relatórios, telas ou interfaces de negócio. Exemplo: 00000000000001. |
| Valor de CNPJ | Identificador de negócio, alfanumérico, 14 caracteres (PIC X(14) em COBOL). Representa o documento oficial de pessoa jurídica, podendo ser numérico legado (ex: 12345678000195) ou alfanumérico moderno (ex: 6QNJ8VY2JIC341). Utilizado em interfaces externas, relatórios, telas e validações de negócio. Único formato visível a usuários e sistemas externos. |
| Classificação | Processo de análise técnica de um campo de CNPJ para determinar se atua como Chave de CNPJ, Valor de CNPJ, ou ambos (classificação acumulativa). Baseada em contexto de uso, participação em operações técnicas (indexação, navegação), semântica de negócio (exibição, transmissão) e inferência determinística. |
| Indefinido | Estado de classificação para campos cuja natureza semântica não pode ser determinada com confiança. Campos Indefinidos não sofrem transformação, validação de DV, geração de Chave ou alteração de tipo. Preservados exatamente conforme originais até análise manual. |
| Correlação 1:1 | Relação obrigatória e determinística entre uma Chave de CNPJ e um Valor de CNPJ, rastreável, auditável e imutável. Garantida exclusivamente pelo serviço central de conversão (DFE). Uma Chave sempre mapeia para exatamente um Valor, e um Valor sempre mapeia para exatamente uma Chave. |
| Serviço Central de Conversão | Sistema autorizado (DFE) responsável por converter Chave de CNPJ em Valor de CNPJ e vice-versa, garantindo determinismo e rastreabilidade. Proibida geração local ou distribuída desta correlação fora de DFE. |
| Base Central de Correlação (DFE) | Estrutura centralizada que mantém a relação 1:1 entre Chave de CNPJ e Valor de CNPJ com metadados de controle, auditoria, rastreamento de mudanças e validação de dígito verificador. Ponto único de verdade para conversões. |
| Cache | Camadas auxiliares de desempenho (Redis, VSAM, processamento batch) que reduzem acessos à base central. Deve ser consistente com DFE e atualizado periodicamente via distribuição batch. |
| Dígito Verificador (DV) | Dígito de validação matemática, 1 posição (PIC X em COBOL), aplicado ao Valor de CNPJ via algoritmo módulo 11 com tabela ASCII. Nunca aplicado à Chave de CNPJ. Implementado via subroutine SBDIGITO (função 011). |
| Consumidor Subsequente | Sistema, programa, rotina, arquivo ou artefato que recebe e processa um campo de CNPJ após sua saída de um programa ou módulo. Deve ser explicitamente identificado por nome, função, contexto e tipo de consumo (exibição, transmissão, persistência, validação, transformação). |
| Fonte Externa | Sistema ou tabela fora do escopo desta modificação (ex.: DB2MCI.CLIENTE, DAF). Dados provenientes de fontes externas não estão no escopo de transformação desta demanda e devem ser preservados conforme recebidos, sem formatação, normalização ou conversão. |
| Módulo Principal | Artefato de código (programa COBOL, copybook, arquivo) pertencente ao sistema VIP, identificado dinamicamente pelo prefixo (ex.: VIP, VIPK, VIPF). Contém lógica de negócio ou manipula CNPJ diretamente. Impactado estruturalmente pela separação de Chave e Valor. |
| Módulo Satélite | Artefato compartilhado com outros sistemas (ex.: DAF, HLP, SBDIGITO, SBABEND). Pode estar sujeito a mudanças em outros projetos. Alteração deve ser coordenada com equipes responsáveis. Dados recebidos devem ser preservados conforme origem. |
| Artefato | Qualquer unidade técnica versionável que compõe, suporta ou influencia a execução do sistema: programas (.cbl), copybooks (.cpy), JCLs, layouts de arquivo, tabelas DB2, estruturas de dados, archivos sequenciais, estruturas de trabalho. Passível de impacto dentro do ecossistema. |
| Enriquecimento de Dados | Processo de agregar informações complementares a um registro (ex.: descrição de restrição de cartão a partir de DB2VIP.TIP_RST_CRT_CRD, dados de agência a partir de DB2VIP.PORT_CRT). Deve respeitar a semântica de CNPJ de cada campo origem e destino. |
| Inferência Determinística | Análise que permite classificar um campo de CNPJ com alta confiança, baseada em evidências técnicas claras (contexto de uso, operações de índice/busca, semântica de negócio). Autoriza transformação. |
| Inferência Fraca | Análise com incerteza quanto à natureza semântica do campo. Resulta em classificação Indefinido, bloqueando qualquer transformação. Requer análise manual para reclassificação. |
| CGC | Denominação anterior do CNPJ (Cadastro Geral de Contribuintes). Campos nomeados CGC devem ser tratados com as mesmas regras aplicáveis ao CNPJ. Exemplo: F104E-CD-CGC-MUN em VIPP4553. |
| PJ | Pessoa Jurídica. Entidade legal de negócio identificada por CNPJ. Foco deste projeto. |
| PF | Pessoa Física. Indivíduo identificado por CPF. Fora do escopo desta demanda. Exemplo: CPF em DB2MCI ou campo 317E-COD-CPF-CGC-PF em VIPP4365. |
| Processamento em Lote | Execução batch de programa COBOL sob controle JCL, lendo arquivo sequencial de entrada e produzindo arquivo(s) de saída. Padrão do Sistema VIP (VIPP4365 lê VIPF317E e gera VIPF940S; VIPP4553 lê VIPF104E e gera VIPF104S). |
| Cabeçalho (Header) | Primeiro registro de um arquivo ou grupo de registros, contendo metadados (identificador, data, identificadores de empresa/cliente, contadores). Tipo de registro = 0 (VIPF940S) ou específico em VIPF104S. Consumidores subsequentes: sistemas de análise de qualidade, auditoria. |
| Detalhe | Registros de conteúdo principal, portadores de dados de negócio (cartões, contas, beneficiários). Tipo de registro = 1. Consumidores subsequentes: sistemas operacionais, de reporte, de integração externa. |
| Rodapé (Trailer) | Último registro de um arquivo ou grupo, contendo resumo (contagem total de registros, validações). Tipo de registro = 9. Consumidores subsequentes: sistemas de controle, auditoria, balanceamento. |
| DCLGEN | Ferramenta DB2 que gera copybooks COBOL a partir de definições de tabela. Utilizados para host variables em EXEC SQL. Exemplo: VIPK317D (DB2VIP.PLST_PORT_PRE_PG), VIPK160D (DB2VIP.TIP_RST_CRT_CRD). |
| Copybook | Arquivo COBOL (.cpy) contendo definições de estruturas de dados reutilizáveis. Incluído via diretiva =INC em programas. Exemplos: VIPK317D (layout de entrada), VIPK160D (tabelas de restrição), DAFK6452 (interface com DAF). |
| COMP, COMP-3, COMP-5 | Tipos de compactação COBOL. COMP-3 = empacotado decimal (2 dígitos por byte). Usado para campos numéricos compactados em DB2 e arquivos. Exemplo: PIC S9(017) COMP-3 em VIPP4365. |
| PIC 9(14), PIC X(14) | Notação PIC (Picture) COBOL. 9 = dígito numérico. X = caractere alfanumérico. Chave de CNPJ = PIC 9(14). Valor de CNPJ = PIC X(14). |
| Ambiente IBM z/OS | Mainframe IBM OS. Compilador COBOL SOS 13. Suporta DB2, VSAM, JCL, CICS. Ambiente padrão do Sistema VIP. Encoding caracteres: ASCII para cálculo de DV, EBCDIC para armazenamento. |
| Layout | Estrutura física de registro em arquivo sequencial ou tabela DB2, definindo campos, tipos, comprimentos e ordens. Exemplo: VIPF317E (358 bytes), VIPF940S (500 bytes), VIPF104E (454 bytes), VIPF104S (579 bytes). |
| Arquivo Sequencial | Arquivo de acesso sequencial em mainframe, com registros de comprimento fixo ou variável. Padrão do Sistema VIP. Leitura: VIPF317E (entrada VIPP4365), VIPF104E (entrada VIPP4553). Escrita: VIPF940S (saída VIPP4365), VIPF104S (saída VIPP4553). |
| Sistema DAF | Subsistema externo de consultoria de beneficiários, consultado por VIPP4553 via subroutine DAFS6452 (função 2 = pesquisa por CNPJ). Retorna: beneficiário, dígitos verificadores, dados de agência. Dados retornados devem ser preservados. |
| DB2VIP | Tabela DB2 no subsistema VIP, armazenando dados cadastrais de cartões, plásticos, portas, restrições. Contém múltiplos layouts (PLST_PORT_PRE_PG, SLCT_PLST_PRE_PG, PLST_PORT, PORT_CRT, CT_CRT_PRE_PG, UND_FAT_EPRL, CEN_CST_EPRL, CLI_EPRL, TIP_RMAT, TIP_RST_CRT_CRD). |
| DB2VIPA | Tabela DB2 no subsistema VIP, armazenando dados de agências, contas e relacionamentos. Consultada para enriquecimento. |
| DB2MCI | Tabela DB2 do subsistema MCI, armazenando dados cadastrais de clientes (CLIENTE). Fonte externa. Dados devem ser preservados conforme origem. Exemplo: 317E-COD-CPF-CGC-PJ em VIPP4365 (MCIKCLIE book). |
| VIPF317E | Arquivo sequencial de entrada (358 bytes), lido por VIPP4365. Contém dados de cartões pré-pagos com CNPJ de pessoa jurídica (317E-COD-CPF-CGC-PJ) e de pessoa física (317E-COD-CPF-CGC-PF). Consumidor: VIPP4365. |
| VIPF940S | Arquivo sequencial de saída (500 bytes), gerado por VIPP4365. Layout de cabeçalho, detalhe e rodapé. Contém CNPJ em contexto de saída (940S-CNPJ-HDR em header, 940S-CNPJ-DET em detalhe). Consumidores: sistemas operacionais, reporte, integração. |
| VIPF104E | Arquivo sequencial de entrada (454 bytes), lido por VIPP4553. Contém dados de contas correntes com múltiplos campos de CNPJ (F104E-CPF-CNPJ, F104E-CPF-CNPJ-RPRT, F104E-CPF-CNPJ-CEN, F104E-CPF-CNPJ-PORT, F104E-CD-CGC-MUN). Consumidor: VIPP4553. |
| VIPF104S | Arquivo sequencial de saída (579 bytes), gerado por VIPP4553. Contém CNPJs em contexto de saída (F104S-CPF-CNPJ, F104S-CPF-CNPJ-RPRT, F104S-CPF-CNPJ-CEN, F104S-CPF-CNPJ-PORT, F104S-CD-CGC-MUN, campos de erro). Consumidores: sistemas operacionais, reporte, integração, análise de erro. |
| SBDIGITO | Subroutine compartilhada que implementa cálculo de dígito verificador via função 011, utilizada para validação de Valor de CNPJ. Entrada recebida é Valor de CNPJ (nunca Chave). Algoritmo: módulo 11 com tabela ASCII. |
| SBVERSAO | Subroutine de controle de versão, data de execução e logging. Não trata CNPJ. Fora do escopo. |
| SBABEND | Subroutine de tratamento de abend (abnormal end) e geração de dump para diagnóstico. Não trata CNPJ. Fora do escopo. |
| DAFS6452 | Subroutine do subsistema DAF (externa), consultada com função 2 para pesquisa de beneficiário por CNPJ. Retorna dados cadastrais (código, dígito verificador, agência, nome, UF). Dados retornados devem ser preservados conforme origem. |
| VIPK317D | Copybook gerado por DCLGEN a partir de DB2VIP.PLST_PORT_PRE_PG, contendo múltiplos sub-layouts (313, 103, 102, 321, 338, 337, 336, 782, MCIKCLIE, 077, TIP-RMAT). Utilizado em VIPP4365 para estrutura de entrada e trabalho. Contém 317E-COD-CPF-CGC-PJ (origem DB2MCI, fonte externa). |
| VIPK160D | Copybook gerado por DCLGEN a partir de DB2VIP.TIP_RST_CRT_CRD, contendo definição de tabela de tipo de restrição. Não contém CNPJ diretamente. Utilizado para leitura de restrições. |
| DAFK6452 | Copybook compartilhado que define interface (Linkage) para subroutine DAFS6452. Contém D6452I-CNPJ (input, PIC 9(014), Valor de CNPJ recebido de VIPP4553) e D6452S-CNPJ (output, PIC 9(014), Chave de CNPJ retornado por DAF). Utilizado por VIPP4553. Entrada é Valor; saída é Chave interna ao DAF. |

---

## 6. Rodapé

---

Documento Elaborado: 02/04/2026

Versão: 1.0

---
