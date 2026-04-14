# PROMPT FINAL V8 — GERAÇÃO DE DOCUMENTAÇÃO FUNCIONAL (ENTERPRISE HARDENED)

## CONTEXTO
Você é um especialista sênior em análise de negócios e documentação funcional, com experiência em sistemas corporativos de grande escala, incluindo ambientes legados e distribuídos.

Sua missão é gerar uma documentação funcional completa, consolidada, consistente, rastreável e aderente ao PRD, pronta para execução em ambiente corporativo.

A documentação deve considerar:
- A evolução do CNPJ para o formato alfanumérico
- A separação entre identificador interno e identificador externo
- A tabela de correlação entre identificador interno e CNPJ
- A classificação dos módulos entre principal e satélite

---

## Definições

Chave: É uma chave (primária ou estrangeira) do CNPJ, e não do conteúdo do CNPJ, onde o campo de chave estrangeira em um arquivo de transação, ou campo de chave primária em um arquivo de consulta, etc. (ex: 78465270000165, 00000000000001). A **Chave de CNPJ** é um identificador técnico interno e não deve ser tratada como documento oficial de negócio.

Valor: É o conteúdo do CNPJ, ou seja, o campo é manipulado como valor de negócio, onde o campo é utilizado em regras de negócio, cálculos, validações, tela (GUI) etc. (ex: 78465270000165, 6QNJ8VY2JIC341). O **Valor de CNPJ** é o identificador oficial utilizado pelo negócio e por interfaces externas.

Módulos: Refere-se a qualquer artefato de código, incluindo programas COBOL e copybooks (arquivos .CPY) que são utilizados pelos programas COBOL. (ex: RF, DAF, VIP, HLP, etc.)

Módulo Principal: É o nome dado a um módulo que pertence ao mesmo sistema dos programas principais. Módulos principais podem ou não ser alterados a depender do plano de execução e execução propriamente dita, mas pertencem ao mesmo time/sistema. A identificação de módulos principais é feita dinamicamente com base no prefixo do nome do artefato. (ex: se os programas principais são VIPP4365 e VIPP4553, então qualquer artefato com prefixo VIP é considerado principal, mesmo que não requeira alteração).

Módulo Secundário: É o nome dado a um módulo que pertence a um sistema diferente dos programas principais. Módulos secundários podem ou não ser alterados a depender do plano de execução e execução propriamente dita, pois pertencem a outros times/sistemas. A identificação de módulos secundários é feita dinamicamente com base no prefixo do nome do artefato. (ex: se os programas principais são VIPP4365 e VIPP4553, então artefatos com prefixo DAF ou HLP são considerados secundários).

Programas: Refere-se ao código COBOL em si, ou seja, aos arquivos .CBL, .CPY, entre outros. Esses arquivos são os artefatos que contêm a lógica de negócio e que serão modificados (patch). No cliente, seus nomes seguem o padrão de um prefixo de 3 letras maiúsculas seguido de um número, por exemplo: VIPP4365.cbl, DAF1234.cbl, HLP5678.cbl.

Correlação 1:1: Relação obrigatória e inequívoca entre uma **Chave de CNPJ** e um **Valor de CNPJ**, preservando rastreabilidade, integridade relacional, auditabilidade e determinismo da conversão.

CGC: Denominação anterior do CNPJ (Cadastro Geral de Contribuintes), equivalente ao CNPJ para todos os efeitos desta demanda. Campos nomeados como CGC devem ser tratados com as mesmas regras aplicáveis ao CNPJ.

PF: Pessoa Física, ou seja, indivíduo, pessoa natural. Campos relacionados a PF não estão no escopo desta demanda, mas podem ser utilizados para comparação e inferência quando necessário.

PJ: Pessoa Jurídica, ou seja, empresa, organização. Campos relacionados a PJ estão no escopo desta demanda, pois o CNPJ é o identificador oficial de pessoas jurídicas.

Fonte Externa: Origem de dados pertencente a um sistema externo ao escopo desta modificação, como tabelas DB2 de outros domínios (ex: DB2MCI.CLIENTE), arquivos legados de terceiros ou integrações de entrada. Dados provenientes de fontes externas não estão no escopo de modificação desta demanda e não devem ser alterados.

Campo Indefinido: Campo CNPJ ou CGC cuja natureza semântica (Chave ou Valor) não pode ser determinada com segurança a partir dos artefatos de discovery e diagnostic — ou seja, onde a taxa de inferência for fraca. Nesses casos, o campo deve ser classificado como Indefinido e não deve ser alterado.

Serviço Central de Conversão: Serviço oficial responsável por converter **Chave de CNPJ** em **Valor de CNPJ** e **Valor de CNPJ** em **Chave de CNPJ**. É proibida a geração local ou distribuída dessa correlação fora do sistema central autorizado.

Base Central de Correlação (DFE): Estrutura central responsável por manter a relação 1:1 entre **Chave de CNPJ** e **Valor de CNPJ**, bem como metadados de controle e auditoria.

Cache: Camadas auxiliares de desempenho utilizadas para reduzir acessos à base central de correlação, podendo incluir mecanismos equivalentes a Redis, VSAM, processamento batch ou estruturas legadas compatíveis com o ambiente.

---

## PREMISSAS GERAIS
- Iniciar com a seção "Introdução"
- Encerrar com "Conclusão"
- Cada seção deve possuir um parágrafo curto explicativo
- Não utilizar inglês
- Não utilizar caminhos de arquivos
- Não incluir metadados no topo
- Não utilizar expressões entre parênteses indicando obrigatoriedade
- Não criar seção "Resumo Executivo"
- Todas as tabelas devem conter legenda
- Diagramas Mermaid devem estar corretos e funcionais

---

## DIRETRIZ DE CONSTRUÇÃO
A documentação deve ser construída com merge inteligente:

- Utilizar “dem_p_overview_2.md” como base estrutural
- Utilizar “dem_prd_requisito.md” como base de requisitos funcionais
- Complementar com demais artefatos sem duplicidade
- Não remover conteúdo relevante
- Priorizar versão mais completa em caso de conflito
- Não inventar informações

---

## MODELO DE IDENTIFICAÇÃO
| Tipo | Descrição |
|------|----------|
| CHAVE | Identificador interno |
| VALOR | CNPJ real |

Regras:
- CHAVE vem do id
- VALOR vem do CNPJ
- READ → VALOR
- LOOKUP → CHAVE

---

## NOMENCLATURA DE FUNÇÕES
- Função de Obtenção/Validação do CNPJ em Plataforma Baixa
- Função de Obtenção/Validação do CNPJ em Plataforma Alta

---

## SEÇÃO — MACRO REQUISITOS DO CLIENTE

Inserir após "Objetivo e Contexto de Negócio".

Extrair EXATAMENTE do PRD:

RF01 até RF10

Regras:
- Não alterar numeração
- Não resumir
- Não reinterpretar
- Incluir:
  - descrição
  - pré-condição
  - pós-condição
  - invariante
  - exemplos

---

## ESTRUTURA OBRIGATÓRIA DO PRD

### Tabela 1 — Identificação Funcional
| Nome do Sistema | Finalidade | Público-Alvo | Problema de Negócio |

---

### Tabela 2 — Pontos de Decisão
| Ponto de Decisão | Condição Avaliada | Resultado A | Resultado B |

---

### Tabelas 3 e 4 — Entradas e Saídas
| Campo de Negócio | Descrição Funcional | Exemplo |

---

### Tabela 7 — Integrações
| Sistema Integrado | Finalidade Funcional | Dados Trocados | Impacto em Caso de Falha |

Obrigatório:
- Criar subseção para módulos utilitários

---

## PRESERVAÇÃO DE REGRAS EXISTENTES

Manter e formalizar:

- Classificação NR/RC
- Filtro FLTR-AUTZ
- Contagem de rodapé
- Integração seletiva DAFS6452

---

## REGRAS DE NEGÓCIO
Devem ser rastreáveis e completas.

---

## REGRAS DE TRANSFORMAÇÃO
- VALOR → CHAVE
- CHAVE → VALOR
- Persistência com CHAVE
- Exposição com VALOR

---

## REGRAS DE ABSTRAÇÃO FUNCIONAL

PROIBIDO:
- nomes de variáveis técnicas
- tipos COBOL
- copybook, linkage
- dependências técnicas

USAR:
- nomes funcionais

---

## RISCOS OBRIGATÓRIOS

Incluir:
- portador sem cadastro correspondente
- encerramento de arquivo interpretado como erro

---

## FLUXOS FUNCIONAIS
Devem conter:
- principal
- alternativos
- exceção

---

## DIAGRAMAS

Obrigatório:
- diagrama VIPP4365
- diagrama VIPP4553
- consistência com texto

---

## GLOSSÁRIO

Regras:
- linguagem de negócio
- incluir DV
- remover:
  - COMP-3
  - Abend
- usar:
  - Processamento em Lote (Batch)

---

## VALIDAÇÃO FINAL

Validar:

- todas as tabelas obrigatórias existem
- RF01–RF10 completos
- riscos obrigatórios presentes
- sem termos técnicos proibidos
- sem inglês
- sem perda de conteúdo
- diagramas completos
- regras preservadas

---

## REGRAS GERAIS

- linguagem técnica clara
- sem emojis
- alta densidade informacional
- consistência terminológica

---

## ARQUIVO FINAL

Gerar:

dem_q_functional_2.md

Salvar:

.pss/b_new/b_doc_b_dem
