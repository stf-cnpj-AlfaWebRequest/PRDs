PROMPT — Geração de Overview Técnico

CONTEXTO
Você é um especialista sênior em engenharia de software, leitura de código legado (COBOL) e análise de sistemas corporativos bancários. Sua tarefa é analisar arquivos fonte e documentação para gerar um overview técnico estruturado e padronizado.

OBJETIVO
Gerar o arquivo:
Diretório de documentação de saída
.pss\b_new\b_doc_b_dem\dem_a_overview_1.md

Com base em:
Arquivos da pasta:
Pasta de código (contendo arquivos .cbl e .cpy)
Documentação da pasta:
/docs (no diretório raiz)

Considere o nosso Overview do Sistema legado, que está em: 
   `.pss/b_new/w_input/a_sai/sai_system_overview_dem_a.md`

Considere também o PRD de requisitos (apenas traga o que for necessário e conciso com o overview):
   `.pss/b_new/w_output/dem_a_cnpj_alfa_prd/dem_prd_requisito.md`



INSTRUÇÕES DE ANÁLISE

1. Leitura de Código (COBOL)
   Identifique:
   Fluxo principal dos programas
   Copybooks utilizados (.cpy)
   Estruturas de dados relevantes
   Integrações (arquivos, DB, APIs, filas, etc.)
   Regras de negócio implícitas
   Detecte:
   Padrões de nomenclatura
   Dependências entre programas
   Possíveis responsabilidades de cada módulo

2. Leitura da Documentação (/docs)
   Extraia:
   Contexto funcional
   Objetivo do sistema
   Regras de negócio explícitas
   Glossário (se existir)
   Diagramas
   Cruze com o código para validar consistência

3. Síntese Inteligente
   Una código + documentação
   Evite redundância
   Preencha lacunas com inferência técnica coerente
   Não invente informações não suportadas
   Na análise de subrotinas de cálculo de Dígito Verificador (ex: SBDIGITO), documentar explicitamente que a entrada recebida é o Valor de CNPJ — nunca a Chave de CNPJ

ESTRUTURA PADRÃO DO ARQUIVO (.md)
Todos os arquivos gerados DEVEM seguir este padrão, com exatamente estas 6 e nenhuma seção adicional fora delas:

# Overview - Projeto Demanda CPNJ Alfanumérico

## 1. Visão Geral

Descrição clara e objetiva do sistema/módulo.

## 2. Resumo Executivo

## 3. Objetivo do Sistema

Finalidade principal e problema que resolve.

## 4. Escopo

O que está incluído e o que não está.

## 5. Glossário

Termos relevantes do domínio. Formato obrigatório: tabela com legenda descritiva acima. Não utilizar lista de bullets.

REGRAS OBRIGATÓRIAS
NÃO utilizar emojis em nenhuma parte da saída
Linguagem técnica, clara e objetiva
Não inventar informações
Inferências devem ser plausíveis e justificáveis
O documento deve refletir exclusivamente o comportamento atual e o design estabelecido do sistema. É proibido registrar riscos futuros, pendências, estados não implementados ou comportamentos esperados que ainda não fazem parte do design documentado
Manter consistência terminológica
Seguir rigorosamente a estrutura definida
Evitar textos excessivamente longos — priorizar densidade informacional
Utilizar tabelas sempre que agregar clareza
Para cada campo de CNPJ identificado em layouts de arquivo, Copybooks de interface e tabelas DB2, classificar explicitamente se o campo representa Chave de CNPJ ou Valor de CNPJ — sem exceção e sem ambiguidade

REGRAS ADICIONAIS DE FORMATAÇÃO E IDIOMA

Idioma
Todo o documento deve ser escrito em português brasileiro. Manter apenas jargões técnicos sem equivalente traduzível no domínio (ex: Copybook, COMP-3, Batch, Linkage Section, File Status, Timestamp). Qualquer expressão em inglês que possua equivalente técnico consagrado em português deve ser traduzida, sem exceção. Os exemplos abaixo são ilustrativos, não exaustivos:

- "Batch Processing" → "Processamento em Lote"
- "Header" / "Trailer" → "Cabeçalho" / "Rodapé de Arquivo"
- "Input File" / "Output File" → "Arquivo de Entrada" / "Arquivo de Saída"
- "LookUp Key" → "Chave de Consulta"
- "Data Enrichment" → "Enriquecimento de Dados"
- "Silent Data Loss" → "Perda Silenciosa de Dados"

Parágrafo descritivo por seção
Imediatamente abaixo de cada título de seção deve haver um parágrafo introdutório em prosa descrevendo o que será abordado — antes de qualquer lista, tabela ou bloco de código.

Uso de dois-pontos (:)
Cada par chave/valor deve ocupar uma linha separada. É proibido agrupar múltiplos pares chave: valor na mesma linha.
Correto:

- Entrada: 25 caracteres
- Saída: 2 caracteres
  Errado:
- Entrada: 25 caracteres. Saída: 2 caracteres.

Legendas em tabelas
Toda tabela, individualmente, deve ter uma legenda descritiva logo acima dela — sem exceção, incluindo tabelas de campos, contratos de interface, subrotinas e glossário. Exemplo:

> Tabela 1 — Componentes identificados no sistema, com seu tipo e responsabilidade funcional.

Cabeçalho do documento
O documento não deve conter metadados soltos no topo (como Data:, Versão:, Autor:). O título do projeto em H1 (#) deve ser a primeira linha.

Completude técnica obrigatória
Os itens abaixo não devem ser omitidos, pois são críticos para rastreabilidade do ambiente legado:

- Seção 11 (Dependências): mencionar explicitamente o ambiente IBM MVS (z/OS) e o compilador COBOL SOS 13
- Seção 10 (Integrações): listar pontualmente todos os schemas do DB2 lidos (ex: PLST_PORT_PRE_PG, PORT_CRT, TIP_RST_CRT_CRD e demais identificados no código)
- Seção 7 (Fluxo de Processamento): detalhar o parâmetro de data no formato exato de 6 bytes (YYMMDD)
- Seção 12 (Pontos de Atenção): preservar os riscos técnicos identificados (ex: silent data loss, necessidade de ordenação intermediária, risco de timeout do CICS em batch, limites de CPU relacionados ao Módulo 11)

## 6. Rodapé

---

Documento Elaborado: <Data de hoje> ex: dd/mm/yyyy

Versão: 5.0

---

"

SAÍDAS ESPERADAS

1. Arquivo principal
   Gerar:
   dem_a_overview_1.md

QUALIDADE ESPERADA
O resultado deve ser equivalente a um documento produzido por:
Engenheiro sênior
Analista de sistemas bancários
Especialista em sistemas legados COBOL

EXECUÇÃO
Agora:
Analise os arquivos fornecidos
Cruce com a documentação
Gere o arquivo conforme especificado
Siga rigorosamente o padrão definido