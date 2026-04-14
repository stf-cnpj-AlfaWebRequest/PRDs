PROMPT — Geração de Overview Técnico

CONTEXTO
Você é um especialista sênior em engenharia de software, leitura de código legado (COBOL) e análise de sistemas corporativos bancários. Sua tarefa é analisar arquivos fonte e documentação para gerar um overview técnico estruturado e padronizado.

ENTRADA
Você deve ler o PRD de requisitos para entender o contexto do projeto.
Ele está em: `.pss/b_new/w_output/dem_a_cnpj_alfa_prd/dem_prd_requisito.md` 


OBJETIVO
Gerar o arquivo em:
.pss\b_new\b_doc_a_app\app_a_overview.md

INSTRUÇÕES DE ANÁLISE

1. Leitura de Código (COBOL) presente na pasta: .pss\a_old\a_code
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

2. Síntese Inteligente
   Una código + documentação
   Evite redundância
   Preencha lacunas com inferência técnica coerente
   Não invente informações não suportadas

ESTRUTURA PADRÃO DO ARQUIVO (.md)
Todos os arquivos gerados DEVEM seguir este padrão, com exatamente estas 6 seções e nenhuma seção adicional fora delas:

# Overview - Aplicação

## 1. Visão Geral

Descrição clara e objetiva do sistema/módulo.

## 2. Resumo Executivo

## 3. Objetivo do Sistema

Finalidade principal e problema que resolve.

## 4. Escopo

O que está incluído e o que não está.

## 5. Glossário

Termos relevantes do domínio.

## 6. Rodapé

---

Documento Elaborado: <Data de hoje> ex: dd/mm/yyyy

Versão: 6.0

---

# REGRAS OBRIGATÓRIAS

FAZER O DOCUMENTO SÓ COM BASE NA PASTA A_CODE,

- entenda os requisitos necessários e pedidos, mas não cite nenhum tipo de plano de modificação ou cite CPNJ alfanumérico, é necessário que ele se baseie fortemente nos arquivos presentes em .pss\a_old\a_code

NÃO utilizar emojis em nenhuma parte da saída
Linguagem técnica, clara e objetiva
Não inventar informações
Inferências devem ser plausíveis e justificáveis
Manter consistência terminológica
Seguir rigorosamente a estrutura definida
Evitar textos excessivamente longos — priorizar densidade informacional
Utilizar tabelas sempre que agregar clareza

REGRAS ADICIONAIS DE FORMATAÇÃO E IDIOMA

Idioma
Todo o documento deve ser escrito em português brasileiro. Manter apenas jargões técnicos sem equivalente traduzível no domínio (ex: Copybook, COMP-3). Todos os demais termos em inglês devem ser traduzidos. Exemplos obrigatórios:

- "Batch Processing" → "Processamento em Lote"
- "Header" / "Trailer" → "Cabeçalho" / "Rodapé de Arquivo"
- "Input File" / "Output File" → "Arquivo de Entrada" / "Arquivo de Saída"
- "LookUp Key" → "Chave de Consulta"
- "Data Enrichment" → "Enriquecimento de Dados"

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
Toda tabela deve ter uma legenda descritiva logo acima dela. Exemplo:

> Tabela 1 — Componentes identificados no sistema, com seu tipo e responsabilidade funcional.

Cabeçalho do documento
O documento não deve conter metadados soltos no topo (como Data:, Versão:, Autor:). O título do projeto em H1 (#) deve ser a primeira linha.

Não dê caminhos de arquivos
ex: "/.pss", extritamente PROIBIDO

Completude técnica obrigatória
Os itens abaixo não devem ser omitidos, pois são críticos para rastreabilidade do ambiente legado:

"

SAÍDAS ESPERADAS

1. Arquivo principal
   Gerar:
   app_a_overview.md

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
