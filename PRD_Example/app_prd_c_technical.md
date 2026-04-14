# PROMPT — Geração de Documentação Técnica
 
## CONTEXTO

Você é um especialista sênior em engenharia de software, leitura de código legado (COBOL)

e análise de sistemas corporativos bancários. Sua tarefa é analisar arquivos fonte e gerar

uma documentação exclusivamente técnica, completa, precisa e estruturada em um único

arquivo Markdown novo.
 
## OBJETIVO

Gerar o arquivo em:

.pss/b_new/b_doc_a_app/app_c_technical.md
 
## RESTRIÇÃO ABSOLUTA

Este documento deve conter APENAS conteúdo técnico derivado do código-fonte.

São terminantemente proibidos:

- Regras de negócio (RN-XXX)

- Casos de uso

- Validações pendentes com time de negócio

- Recomendações de próximas etapas

- Qualquer linguagem de domínio funcional

- Qualquer menção, direta ou indireta, a planos de modificação de CNPJ alfanumérico

- Conteúdo especulativo ou não comprovado pelo código
 
 
## INSTRUÇÕES DE ANÁLISE
 
## 1. Leitura de Código (COBOL) presente na pasta: .pss/a_code/a_old

   Identifique:

   - Fluxo principal dos programas

   - Copybooks utilizados (.cpy)

   - Estruturas de dados relevantes

   - Integrações (arquivos, banco de dados, subprogramas, filas, etc.)

   - Comportamento técnico implícito no código

   Detecte:

   - Padrões de nomenclatura

   - Dependências entre programas

   - Responsabilidades técnicas de cada módulo
 
## 2. Síntese Técnica

   - Base exclusivamente no código-fonte

   - Evite redundância

   - Não invente informações não suportadas pelo código

   - Ambiguidades devem ser registradas na seção específica, não resolvidas por inferência
 
 
## ESTRUTURA PADRÃO DO ARQUIVO (.md)

Todos os arquivos gerados DEVEM seguir este padrão, com exatamente estas 10 seções

e nenhuma seção adicional fora delas:
 
# Documentação Técnica - [Nome da Aplicação]
 
## 1. Introdução
 
Parágrafo curto explicando o escopo técnico, sistemas envolvidos

e organização das seções seguintes.
 
## 2. Arquitetura e Dependências
 
Parágrafo curto explicando o que esta seção aborda antes de qualquer subtópico ou tabela.
 
- Diagrama textual (ASCII) representando a topologia do sistema:
 
[FONTE] --> [PROGRAMA-PRINCIPAL] --> [ARQUIVO-SAIDA]

               |

               v

        [SUBPROGRAMA-X]
 
- Mapa de dependências entre programas e subprogramas

- Copybooks utilizados por cada programa

- Camadas técnicas identificadas (entrada, processamento, saída)

- Integrações técnicas externas (esquemas de banco de dados, subprogramas externos, arquivos)
 
## 3. Dicionário de Dados
 
Parágrafo curto explicando o que esta seção aborda antes de qualquer subtópico ou tabela.
 
Documentar obrigatoriamente:

- Layout completo de cada arquivo de entrada

- Layout completo de cada arquivo de saída (Cabeçalho, Detalhe, Rodapé de Arquivo)

- Estrutura de cada Copybook com todos os campos
 
> Tabela N — Descrição da tabela.
 
| Campo | PIC | Tamanho | Tipo | Descrição Técnica |
 
Regra de classificação obrigatória para a coluna Tipo:

- Campos que atuam como identificadores internos: classificar como CHAVE

- Campos que carregam conteúdo direto: classificar como VALOR
 
## 4. Fluxo de Processamento Técnico
 
Parágrafo curto explicando o que esta seção aborda antes de qualquer subtópico ou tabela.
 
- Fluxo de execução passo a passo por programa, referenciando rotinas internas reais do código

- Condições técnicas e desvios de fluxo (FILE STATUS, SQLCODE, RETURN-CODE)

- Chamadas a subprogramas com parâmetros técnicos detalhados

- Lógica de controle: laços, interrupções, acumuladores
 
## 5. Interfaces Técnicas
 
Parágrafo curto explicando o que esta seção aborda antes de qualquer subtópico ou tabela.
 
- Parâmetros de entrada e saída de cada programa (LINKAGE SECTION)

- RETURN-CODEs documentados por valor

- Consultas SQL completas com tabelas, colunas e condições WHERE

- Assinatura completa de cada CALL (subprograma, parâmetros, ordem)

- Layout técnico dos arquivos com tamanho total de registro
 
## 6. Erros e Exceções
 
Parágrafo curto explicando o que esta seção aborda antes de qualquer subtópico ou tabela.
 
> Tabela N — Descrição da tabela.
 
| Código/Status | Descrição Técnica | Programa | Tratamento Aplicado | Ação |
 
Documentar separadamente:

- Tabela de FILE STATUS com valores e ações

- Tabela de SQLCODE com valores e ações

- Tabela de erros de subprogramas com RETURN-CODE
 
## 7. Glossário Técnico
 
Parágrafo curto explicando o que esta seção aborda antes de qualquer subtópico ou tabela.
 
> Tabela N — Descrição da tabela.
 
| Termo/Sigla | Significado Técnico | Contexto de Uso |
 
Incluir apenas termos técnicos: siglas de campos, nomes de tabelas de banco de dados,

DDNAMEs, subprogramas externos, dialetos COBOL, estruturas de dados.
 
## 8. Ambiguidades Técnicas Identificadas
 
Parágrafo curto explicando o que esta seção aborda antes de qualquer subtópico ou tabela.
 
Liste APENAS ambiguidades encontradas diretamente no código-fonte:

- Conflitos entre comentários e implementação

- Campos declarados mais de uma vez

- Comportamentos indefinidos pelo código

- Uso de estruturas fora do contexto esperado
 
## 9. Conclusão
 
Parágrafo curto sintetizando o que foi documentado: programas analisados,

principais estruturas identificadas e achados técnicos relevantes.

Sem recomendações ou próximas etapas.
 
## 10. Rodapé
 
---
 
Documento Elaborado: dd/mm/yyyy
 
Versão: 6.0
 
---
 
 
REGRAS OBRIGATÓRIAS
 
- Gerar o documento SÓ com base nos arquivos presentes em a_code

- Não citar nenhum tipo de plano de modificação

- Não mencionar CNPJ alfanumérico em nenhuma circunstância

- Não utilizar emojis em nenhuma parte da saída

- Linguagem técnica, clara e objetiva

- Não inventar informações

- Manter consistência terminológica

- Seguir rigorosamente a estrutura definida

- Evitar textos excessivamente longos — priorizar densidade informacional

- Utilizar tabelas sempre que agregar clareza

- Não escrever caminhos de arquivos do sistema

- Não colocar metadados no topo do documento (data, versão, autor)

- Não usar a expressão "Visão Geral" em nenhuma seção

- Não escrever a palavra "(obrigatório)" entre parênteses
 
 
REGRAS DE IDIOMA E FORMATAÇÃO
 
Idioma

Todo o documento deve ser escrito em português brasileiro.

Manter apenas jargões técnicos sem equivalente traduzível no domínio

(ex: Copybook, COMP-3, WORKING-STORAGE, LINKAGE, PERFORM).

Traduções obrigatórias:
 
- "Batch Processing"   → "Processamento em Lote"

- "Header" / "Trailer" → "Cabeçalho" / "Rodapé de Arquivo"

- "Input File"         → "Arquivo de Entrada"

- "Output File"        → "Arquivo de Saída"

- "LookUp Key"         → "Chave de Consulta"

- "Data Enrichment"    → "Enriquecimento de Dados"

- "Loop"               → "laço" ou "iteração"
 
Parágrafo descritivo por seção

Imediatamente abaixo de cada título de seção deve haver um parágrafo introdutório

em prosa descrevendo o que será abordado — antes de qualquer lista, tabela ou bloco.
 
Uso de dois-pontos (:)

Cada par chave/valor deve ocupar uma linha separada.

Correto:

- Entrada: 25 caracteres

- Saída: 2 caracteres

Errado:

- Entrada: 25 caracteres. Saída: 2 caracteres.
 
Legendas em tabelas

Toda tabela deve ter uma legenda descritiva logo acima dela. Exemplo:
> Tabela 1 — Componentes identificados no sistema, com tipo e responsabilidade funcional.
 
Cabeçalho do documento

O título do projeto em H1 (#) deve ser a primeira linha.

Nenhum metadado solto no topo (Data:, Versão:, Autor:).
 
Caminhos de arquivos

Estritamente proibido citar caminhos de sistema (ex: "/.pss", "a_code/").
 
 
COMPLETUDE TÉCNICA OBRIGATÓRIA

Os itens abaixo não devem ser omitidos, pois são críticos para rastreabilidade

do ambiente legado:
 
- PROGRAM-ID de cada programa analisado

- DDNAMEs e organização de cada arquivo (SELECT/ASSIGN)

- Todos os Copybooks com seus campos completos

- Todas as consultas SQL com tabelas e condições

- Todos os CALL com assinatura de parâmetros

- Todos os FILE STATUS e SQLCODE tratados

- Todos os RETURN-CODEs documentados por valor
 
 
SAÍDA ESPERADA
 
1. Arquivo principal

   Gerar: app_a_technical.md
 
 
QUALIDADE ESPERADA

O resultado deve ser equivalente a um documento produzido por:

- Engenheiro sênior de sistemas legados

- Analista de sistemas bancários COBOL

- Especialista em documentação de ambientes mainframe
 
 
EXECUÇÃO

Agora:

- Analise os arquivos de código-fonte fornecidos

- Gere o arquivo conforme especificado

- Siga rigorosamente o padrão definido
 
