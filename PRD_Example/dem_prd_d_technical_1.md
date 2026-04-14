ASSUMA o papel de um ARQUITETO DE SOFTWARE SÊNIOR e ESPECIALISTA EM DOCUMENTAÇÃO TÉCNICA ENTERPRISE, com ampla experiência em sistemas distribuídos, mainframe, APIs, bancos de dados, integrações complexas e modernização de legado.

CONTEXTO:
Você receberá documentos anexos contendo especificações técnicas, funcionais e materiais de apoio relacionados ao CNPJ alfanumérico, incluindo um artefato de overview, especificações técnicas, funcionais e documentos auxiliares.

OBJETIVO:
Gerar uma DOCUMENTAÇÃO TÉCNICA COMPLETA, CONSOLIDADA E EM NÍVEL ENTERPRISE, eliminando duplicidades, incorporando apenas conteúdos novos e relevantes, garantindo coerência arquitetural, rastreabilidade e aplicabilidade prática.

---

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

### Regra Mandatória de Explicitação de Consumo Subsequente

Sempre que a LLM identificar que um campo de CNPJ está inserido em **contexto de saída**, especialmente em estruturas de **detalhe**, **áreas de comunicação**, **buffers de output**, **arquivos de remessa/retorno**, **arquivos de saída**, **estrutura de saída**, **segmentos de transmissão**, **interfaces entre programas** ou qualquer outro artefato de **propagação de dado de negócio**, **não é suficiente** classificá-lo apenas como **Valor de Negócio**.

Nesses casos, a LLM **deve obrigatoriamente** explicitar, de forma técnica e rastreável, **quais são os consumidores subsequentes do campo** após a saída do programa, da rotina, do módulo ou do artefato analisado.

A análise **deve deixar claro**:

- **quem consome o dado posteriormente**, de forma nominal e técnica;
- **qual artefato subsequente recebe o campo**;
- **qual é a finalidade do consumo**;
- **se o campo é apenas transportado ou efetivamente utilizado em lógica de negócio**;
- **quais impactos decorrem da mudança do CNPJ numérico para alfanumérico nesse encadeamento subsequente**.

A LLM deve identificar, sempre que possível, os consumidores subsequentes dentro da cadeia de processamento, tais como:

- programas COBOL chamadores ou chamados;
- steps seguintes de JOB/JCL;
- transações CICS;
- rotinas batch;
- arquivos físicos ou lógicos;
- copybooks compartilhados;
- tabelas DB2;
- mensagens MQ;
- APIs;
- sistemas satélites;
- módulos de integração;
- rotinas de impressão, extrato, relatório, auditoria, conciliação ou retorno.

Além de identificar os consumidores subsequentes, a LLM **deve explicitar a natureza do consumo**, informando se o campo é:

- exibido;
- transmitido;
- persistido;
- validado;
- transformado;
- replicado;
- conciliado;
- utilizado em regra de negócio;
- utilizado em interface externa;
- ou apenas propagado como carga útil.

A conclusão **não deve ser genérica**. A LLM **não deve afirmar apenas** que se trata de “output”, “transmissão de dados de negócio” ou “valor de negócio”. Ela **deve detalhar o encadeamento técnico subsequente**, deixando explícito:

- a **origem** do campo;
- o **destino** do campo;
- o **artefato intermediário**, se houver;
- o **contrato técnico** em que ele trafega;
- e o **impacto potencial da alteração de formato** ao longo dessa cadeia.

A LLM deve utilizar como evidências técnicas, sempre que disponíveis:

- nome do campo;
- nome do registro;
- nome do layout;
- sufixos semânticos como `-DET`, `-OUT`, `-SAI`, `-RET`;
- instruções `MOVE`;
- gravação em arquivo;
- passagem por parâmetro;
- montagem de mensagem;
- envio para outro programa;
- uso em tela, relatório ou interface.

Em ambiente **mainframe**, a presença do CNPJ em estrutura de **saída/detalhe** deve ser tratada como forte indicativo de que o dado participa de uma **cadeia de consumo downstream**. Portanto, a análise da LLM **deve ser explícita, nominativa, técnica e rastreável**, demonstrando não apenas que o campo representa **Valor de Negócio**, mas também **quem o consome depois, como o consome, em qual artefato o consome e quais impactos subsequentes a mudança para CNPJ alfanumérico pode provocar**.

Se uma tabela tiver esse tipo de informação, OBRIGATORIAMENTE adicione a coluna à ela para identificar quem é o Consumidor da saída desse dado.

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


PREMISSA DE CONSOLIDAÇÃO

1. Utilize o artefato que está em: ".pss/b_new/b_doc_b_dem/dem_a_overview_1.md" como base arquitetural da análise.
- Utilizar o artefato que está em: “.pss/b_new/w_output/dem_a_cnpj_alfa_prd/dem_prd_requisito.md” como fonte obrigatória para extração dos macro requisitos técnicos do cliente
2. Realize um merge controlado com os demais documentos:
   - eliminar duplicidades
   - incorporar apenas conteúdos novos, relevantes e não conflitantes
   - enriquecer tecnicamente o documento final
3. Em caso de conflito entre documentos:
   - priorizar a visão arquitetural e de escopo definida no overview
   - registrar decisões quando necessário
4. Não replicar conteúdo redundante
5. Não ignorar detalhes técnicos relevantes presentes fora do overview

PREMISSA ADICIONAL
Para essa documentação, não faremos a seção de Resumo Executivo.

---------------------------------------------------------------------

ETAPA 0 — IDENTIFICAÇÃO DE MÓDULOS

Antes de qualquer análise técnica:

1. Identificar todos os módulos envolvidos
2. Classificar em:

- PRINCIPAIS:
  módulos que sofrem alteração direta, contêm regra de negócio ou manipulam o CNPJ como VALOR

- SATÉLITES:
  módulos vizinhos que apenas consomem dados e não devem sofrer alteração estrutural

3. Gerar tabela:

| Módulo | Descrição | Tipo | Justificativa | Deve Alterar? |

Legenda da tabela de módulos:
- Módulo: Nome do componente
- Descrição: Responsabilidade do módulo
- Tipo: Principal ou Satélite
- Justificativa: Motivo da classificação
- Deve Alterar: Indica se sofrerá alteração

4. Explicar critérios de classificação
5. Mapear dependências entre módulos

---------------------------------------------------------------------

PRINCÍPIO FUNDAMENTAL DA SOLUÇÃO

A solução é baseada em:

- CHAVE: identificador técnico interno
- VALOR: conteúdo real do CNPJ

Tabela de correlação:
- Id: chave interna
- CNPJ: valor oficial

---------------------------------------------------------------------

ETAPA 1 — CLASSIFICAÇÃO CHAVE x VALOR

Para todas as ocorrências:
- banco de dados
- APIs
- variáveis
- layouts
- copybooks
- arquivos
- integrações
- relatórios

Classificar:

- CHAVE:
  quando utiliza o Id como referência

- VALOR:
  quando utiliza diretamente o CNPJ

---------------------------------------------------------------------

ETAPA 2 — MATRIZ DE CLASSIFICAÇÃO

Gerar tabela:

| Item Técnico | Tipo de Artefato | Local | Classificação | Justificativa | Ação Técnica |

Legenda da tabela:
- Item Técnico: Nome do campo ou variável
- Tipo de Artefato: Tipo do componente
- Local: Onde aparece
- Classificação: CHAVE ou VALOR
- Justificativa: Motivo técnico
- Ação Técnica: Alterar, manter ou avaliar

---------------------------------------------------------------------

ETAPA 3 — REGRAS DE DECISÃO

Para CHAVE:
- não alterar estrutura se já for identificador interno
- não expor externamente
- consultar via Id
- conversão apenas em serviços

Para VALOR:
- suportar CNPJ alfanumérico
- validar DV por módulo 11 com ASCII
- aceitar letras e números
- manter compatibilidade com numérico

---------------------------------------------------------------------

ETAPA 4 — DETECÇÃO DE FALSOS POSITIVOS

Identificar:
- campos nomeados como CNPJ que são Id
- campos numéricos que são apenas referência
- inconsistências de uso

Marcar como INDETERMINADO quando necessário e justificar

REGRA DE FALLBACK OBRIGATÓRIA PARA ESTADO INDETERMINADO

Quando um campo for classificado como INDETERMINADO, a documentação
deve definir explicitamente o comportamento seguro de fallback:

- o campo não deve sofrer conversão até que a classificação seja confirmada
- o processamento em lote que dependa desse campo deve adotar comportamento
  conservador: tratar o valor como numérico legado e registrar ocorrência
  em log de auditoria
- a regra de fallback deve ser documentada na seção de Regras de Negócio
  e referenciada na Matriz de Classificação para cada item INDETERMINADO
- não é permitido deixar a decisão delegada ao desenvolvedor sem ao menos
  definir o comportamento padrão de segurança

---------------------------------------------------------------------

ETAPA 5 — PRESERVAÇÃO DE CONTEÚDO RELEVANTE

Durante o merge:

- Não remover:
  - inconsistências técnicas
  - alertas de risco
  - ambiguidades de regra
  - problemas de schema ou ambiente

- Quando identificado:
  - registrar na seção de Riscos e Mitigações
  - explicar impacto técnico

---------------------------------------------------------------------

ETAPA 6 — INCORPORAÇÃO DE MACRO REQUISITOS DO CLIENTE

Criar a seção:

## MACRO REQUISITOS DO CLIENTE

Regras obrigatórias:

1. Fonte:
   - Extrair exclusivamente do PRD atualizado “dem_prd_requisito.md”

2. Conteúdo a ser incorporado:
   - RF01 até RF10
   - RT01 até RT14

3. Forma de incorporação:
   - Manter numeração original exatamente como definida
   - Não renumerar
   - Não resumir
   - Não reinterpretar semanticamente
   - Não consolidar requisitos
   - Não eliminar exemplos, contratos ou invariantes
   - Preservar integralmente o conteúdo técnico

4. Estrutura de cada requisito:
   - Título
   - Contrato:
     - Pré-condição
     - Pós-condição
     - Invariante
   - Exemplo quando existir

5. Proibições:
   - Não alterar nomenclatura
   - Não adaptar linguagem técnica do requisito
   - Não misturar requisitos

6. Objetivo:
   - Garantir rastreabilidade entre PRD e documentação técnica

7. Exclusões:
   - Não incluir critérios de aceitação
   - Não incluir direcionamento estratégico
   - Não incluir evolução futura
   - Não incluir regras de execução da LLM

---------------------------------------------------------------------

ESTRUTURA DO DOCUMENTO

## Introdução
Parágrafo curto explicando objetivo e contexto da documentação.

## Modelo Conceitual
Parágrafo curto explicando separação entre CHAVE e VALOR.

## Arquitetura da Solução
Parágrafo curto explicando componentes e fluxos.
Incluir diagrama Mermaid válido.

## Diagramas Técnicos
Parágrafo curto contextualizando os diagramas.
Incluir os quatro diagramas Mermaid obrigatórios, todos orientados LR:
- Diagrama de Fluxo de Dados
- Diagrama de Estado
- Diagrama de Sequência
- Diagrama de Atividades


## Diagramas Técnicos

Esta seção contém os diagramas técnicos obrigatórios da solução.
Todos os diagramas devem ser gerados em Mermaid válido, orientados na
horizontal (left to right), com nomenclatura exclusivamente em português
e cobrindo o contexto da transição do CNPJ numérico para alfanumérico.

### Diagrama de Fluxo de Dados

Representar o fluxo de dados do CNPJ pela solução, contemplando:
- Entrada do CNPJ (numérico ou alfanumérico) pelo canal de origem
- Passagem pelo ponto de conversão/validação
- Separação entre CHAVE (Id interno) e VALOR (CNPJ real)
- Persistência no banco de dados
- Consumo pelos módulos satélites
- Retorno ao canal de destino

Orientação obrigatória: esquerda para direita (LR).
Incluir legenda identificando origem, transformação e destino dos dados.

### Diagrama de Estado

Representar os estados possíveis de um registro de CNPJ durante o ciclo
de vida da transição, contemplando:
- Estado: CNPJ numérico legado
- Estado: CNPJ em processo de migração
- Estado: CNPJ alfanumérico validado
- Estado: CNPJ inválido / rejeitado
- Transições entre estados com os eventos que as disparam
- Estado final: CNPJ persistido e disponível para consumo

Orientação obrigatória: esquerda para direita (LR).
Incluir legenda identificando eventos de transição.

### Diagrama de Sequência

Representar a sequência de interações entre os componentes da solução
para o fluxo principal de recebimento, validação e persistência de um
CNPJ alfanumérico, contemplando:
- Ator: sistema ou canal de origem
- Função de Obtenção/Validação do CNPJ em Plataforma Baixa
- Função de Obtenção/Validação do CNPJ em Plataforma Alta
- Serviço de conversão CHAVE x VALOR
- Banco de dados
- Módulo consumidor (satélite)
- Retornos e mensagens de erro em cada etapa

Orientação obrigatória: esquerda para direita (LR).
Incluir numeração das mensagens e identificação dos participantes.

### Diagrama de Atividades

Representar o fluxo de atividades do processo de validação e
processamento do CNPJ alfanumérico, contemplando:
- Recebimento do CNPJ
- Decisão: formato numérico ou alfanumérico
- Aplicação do algoritmo de validação de dígito verificador
  por módulo 11 com tabela ASCII
- Decisão: CNPJ válido ou inválido
- Caminho de rejeição com registro de erro
- Caminho de sucesso com conversão para CHAVE interna
- Persistência e notificação ao módulo consumidor

Orientação obrigatória: esquerda para direita (LR).
Incluir pontos de decisão, bifurcações e convergências claramente
identificados.

## Identificação de Módulos
Parágrafo curto explicando classificação de módulos.
Incluir tabela.

Além da tabela de módulos, incluir obrigatoriamente:

### Mapa de Dependências por Módulo

Para cada módulo PRINCIPAL, listar explicitamente:
- programas COBOL que o compõem ou invocam
- arquivos sequenciais lidos ou gravados
- copybooks utilizados
- tabelas de banco de dados acessadas
- outros módulos dos quais depende

Formato: lista estruturada por módulo, com identificação do tipo de
dependência (leitura, gravação, invocação, referência).

Objetivo: garantir que o desenvolvedor tenha rastreabilidade completa
antes de iniciar qualquer alteração.

## Matriz de Classificação
Parágrafo curto explicando CHAVE x VALOR.
Incluir tabela.

## Modelo de Dados
Parágrafo curto explicando estrutura e correlação.

## Regras de Negócio
Parágrafo curto explicando validações e coexistência.

## Serviços e Contratos
Utilizar nomes:
- Função de Obtenção/Validação do CNPJ em Plataforma Baixa
- Função de Obtenção/Validação do CNPJ em Plataforma Alta

## Impacto por Camada

Separar obrigatoriamente:

- Banco de dados
- Mainframe
   -Layouts de Arquivos Sequenciais
      Para cada arquivo sequencial impactado, documentar obrigatoriamente:
      - nome do arquivo
      - tamanho total do registro em bytes
      - tabela de campos com: posição inicial, posição final, tamanho em bytes,
      tipo (numérico, alfanumérico, binário) e descrição
      - identificação dos campos que contêm CHAVE ou VALOR de CNPJ
      - indicação de alteração necessária (sim ou não) com justificativa
      A omissão dessa tabela é proibida para arquivos que contenham campos
      classificados como CHAVE ou VALOR de CNPJ, pois o tamanho exato do
      registro dita como o arquivo será lido e gravado pelo COBOL.
- APIs
- Integrações
- Processamento em lote
   Descrever obrigatoriamente o fluxo iterativo do processamento em lote,
   contemplando as seguintes etapas na sequência exata:
      1. leitura sequencial do arquivo de entrada
      2. identificação do campo de CNPJ no registro lido
      3. ponto de acionamento do serviço de conversão (DFE): quando acionar,
         com qual entrada e qual retorno esperado
      4. transformação e agrupamento do registro
      5. gravação no arquivo de saída ou tabela de destino
   O objetivo é que o desenvolvedor identifique com precisão o momento
   exato dentro do pipeline em lote em que o serviço de conversão deve
   ser invocado, sem ambiguidade.
- Interface
- Análise de dados

Mesmo sem impacto:
- declarar ausência explicitamente

## Itens que Devem Ser Alterados

## Itens que Não Devem Ser Alterados

## Pontos de Conversão

## Performance e Escalabilidade

## Segurança e Governança

## Riscos e Mitigações

## Critérios de Aceite

## Estratégia de Testes

## Roadmap Evolutivo

## MACRO REQUISITOS DO CLIENTE

## Conclusão
Parágrafo curto consolidando decisões técnicas.

## Rodapé
---
Documento Elaborado: <dd/mm/yyyy>
Versão: 8.0
---

---------------------------------------------------------------------

NORMALIZAÇÃO DE LINGUAGEM

Antes da saída final:

- Traduzir todos os termos técnicos para português
- Não permitir nenhuma palavra em inglês

---------------------------------------------------------------------

VALIDAÇÃO FINAL OBRIGATÓRIA

Antes de finalizar:

1. Verificar ausência total de termos em inglês
2. Confirmar presença de todas as camadas
3. Confirmar legenda em todas as tabelas
4. Confirmar legenda da tabela de módulos conforme definido
5. Confirmar inclusão da seção "MACRO REQUISITOS DO CLIENTE"
6. Confirmar inclusão completa de RF01–RF10
7. Confirmar inclusão completa de RT01–RT14
8. Confirmar que nenhum conteúdo técnico relevante foi perdido
9.  Confirmar presença do Diagrama de Fluxo de Dados em Mermaid válido e orientado LR
10. Confirmar presença do Diagrama de Estado em Mermaid válido e orientado LR
11. Confirmar presença do Diagrama de Sequência em Mermaid válido e orientado LR
12. Confirmar presença do Diagrama de Atividades em Mermaid válido e orientado LR
13. Confirmar que todos os diagramas utilizam nomenclatura exclusivamente em português
14. Confirmar que todos os diagramas cobrem o contexto da transição CNPJ numérico
    para alfanumérico e não são genéricos
15. Verificar ausência de caracteres corrompidos ou quebras de codificação
    em qualquer palavra do documento
16. Verificar que nenhuma célula da tabela de casos de teste contém
    saídas de log de sistema, comandos de terminal ou texto não relacionado
    ao resultado esperado do teste
17. Confirmar presença de layout detalhado (posição e tamanho em bytes)
    para cada arquivo sequencial de mainframe que contenha campo
    classificado como CHAVE ou VALOR de CNPJ
18. Confirmar presença do Mapa de Dependências para cada módulo PRINCIPAL,
    com listagem de programas COBOL, arquivos sequenciais, copybooks,
    tabelas de banco de dados e módulos dependentes

Se qualquer item não for atendido:
- corrigir automaticamente antes da saída final

---------------------------------------------------------------------

REGRAS DE ESCRITA

- não utilizar inglês
- não utilizar caminhos de arquivos
- não utilizar emojis
- não usar "visão geral" no início
- não incluir metadados no topo
- não utilizar parênteses para obrigatoriedade
- linguagem técnica clara e objetiva
- manter consistência terminológica
- utilizar tabelas quando necessário
- evitar textos longos
- garantir qualidade enterprise
- garantir que nenhuma palavra contenha caracteres corrompidos ou quebras
  de codificação (ex: "validaç\no", "exposi\no", "evoluç\no")
- revisar todos os termos com acentuação antes da saída final
- nas tabelas de casos de teste, a coluna "saída esperada" deve conter
  exclusivamente resultados de negócio (ex: código de retorno, mensagem
  de validação); nunca inserir logs de sistema operacional, saídas de
  terminal ou comandos de inicialização de serviço

---------------------------------------------------------------------

ARQUIVO FINAL

O arquivo final deve ter o nome: `dem_d_technical_1`
Gerar o arquivo final em: .pss/b_new/b_doc_b_dem/dem_d_technical_1.md
