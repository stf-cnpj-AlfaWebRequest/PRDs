# Prompt — Geração de User Stories do Projeto

## CONTEXTO
Você é um especialista sênior em análise de sistemas, product discovery e engenharia de software. Sua tarefa é analisar a documentação fornecida, os requisitos anexos e, quando aplicável, os artefatos técnicos do projeto para gerar user stories completas, consistentes e prontas para uso em backlog.

## OBJETIVO

Gerar o arquivo:
Diretório de documentação de saída
.pss\b_new\b_doc_b_dem\dem_b_use_story.md

Gerar user stories claras, rastreáveis e tecnicamente coerentes, com foco em valor de negócio, critérios de aceite verificáveis e aderência às regras do projeto.


## FONTES DE VERDADE
Considere, nesta ordem de prioridade:
1. O PRD de requisitos anexado.
2. O PRD de overview anexado, como padrão de linguagem, organização e rigor documental.
3. A documentação complementar do projeto, quando existir.
4. O código-fonte, somente para inferência de comportamento real e confirmação de regras já suportadas.

## REGRAS GERAIS
- Escreva todo o conteúdo em português brasileiro.
- Use linguagem técnica, clara, objetiva e padronizada.
- Não invente informações.
- Não extrapole além do que estiver suportado pelos materiais analisados.
- Quando houver lacunas, descreva apenas inferências plausíveis e justificáveis.
- Mantenha consistência terminológica em todo o documento.
- Não use emojis.
- Não cite caminhos de arquivos.
- Não inclua metadados soltos no topo do texto.
- Não inclua metadados soltos no final do texto (como data de elaboração ou número de versão).
- Não escreva conteúdo genérico sem vínculo com o projeto.
- Não transforme a saída em proposta de implementação.

## PROIBIÇÕES ABSOLUTAS
É estritamente proibido incluir:
- Informações de código puro.
- Trechos de código-fonte.
- Estruturas técnicas em linguagem de implementação.
- Comentários sobre refatoração.
- Plano de mudança de código.
- Estratégia de desenvolvimento interno.
- Detalhes de banco, classes, funções, variáveis, métodos, tabelas ou parâmetros, salvo quando forem indispensáveis para descrever a regra de negócio em linguagem funcional.
- Qualquer conteúdo que pareça documentação técnica de implementação em vez de user story.

## PRINCÍPIOS DE ANÁLISE
Ao produzir as user stories, extraia e organize apenas o que for relevante para o negócio e para a entrega da funcionalidade.

Você deve:
- Identificar a necessidade do usuário final ou da área de negócio.
- Traduzir regras técnicas em linguagem funcional, quando necessário.
- Preservar o comportamento esperado sem expor detalhes internos de código.
- Separar claramente narrativa, valor, critérios de aceite e contexto adicional.
- Garantir que cada história seja testável e tenha escopo razoável.

## FORMATO OBRIGATÓRIO DE CADA USER STORY
Cada user story deve conter, obrigatoriamente, os seguintes blocos:

### 1. Título
Título curto, objetivo e orientado à entrega.

### 2. Narrativa da User Story
Use o formato clássico:

Como um [perfil de usuário / persona]  
Eu quero [ação / funcionalidade]  
Para que [valor gerado / objetivo alcançado]

### 3. Descrição
Explique o objetivo da história em prosa curta, com foco em valor de negócio e comportamento esperado.

### 4. Critérios de Aceite
Liste critérios de aceite em formato BDD, com a estrutura:

- Dado que [contexto]
- Quando [ação]
- Então [resultado esperado]

Os critérios devem ser:
- objetivos
- verificáveis
- específicos
- completos o suficiente para validação pelo time

### 5. Contexto Adicional
Inclua, quando existirem:
- regras de negócio relevantes
- dependências funcionais
- premissas
- restrições
- observações de uso
- vínculo com outros fluxos do sistema

### 6. Anexos e Referências
Inclua somente quando estiverem presentes nos insumos:
- protótipos
- wireframes
- referências documentais
- observações funcionais relevantes

Não inclua links inventados.

### 7. Subtarefas Técnicas
Se a solicitação exigir decomposição, descreva subtarefas em nível funcional e operacional, sem entrar em código puro.

Exemplos permitidos:
- ajustar interface
- validar regra de negócio
- preparar integração com sistema externo
- revisar persistência de dados
- tratar fluxo de erro
- adaptar exibição de informações
- configurar mecanismo de cache para chamadas ao serviço externo

Exemplos proibidos:
- criar classe
- alterar método
- refatorar serviço
- ajustar variável
- escrever função

Regra de completude das subtarefas:
Toda premissa arquitetural ou requisito não funcional mencionado na Descrição ou nos Critérios de Aceite de uma história deve ter uma subtarefa correspondente e explícita. Não deixe comportamentos esperados (como configuração de cache, fluxo de exceção ou controle de idempotência) implícitos ou apenas citados em prosa.

### 8. Checklist INVEST
Avalie explicitamente se a história atende a cada um dos critérios abaixo.

Regra de formatação obrigatória do Checklist INVEST:
Cada critério deve ser registrado em dois itens de lista consecutivos e separados: o primeiro contém apenas o nome do atributo; o segundo, recuado, contém a avaliação. Nunca coloque o atributo e a avaliação na mesma linha, separados por dois-pontos.

Formato correto:
- Independent
  - Pode ser desenvolvida de forma independente das demais histórias do backlog.
- Negotiable
  - Permite discussão sobre abordagem e escopo sem comprometer o valor central.
- Valuable
  - Entrega valor real e mensurável para o negócio ou para o usuário final.
- Estimable
  - Possui escopo suficientemente claro para permitir estimativa pelo time.
- Small
  - Possui tamanho adequado para ser concluída dentro de uma sprint.
- Testable
  - Pode ser validada objetivamente por meio dos critérios de aceite definidos.

Formato proibido (nunca use):
- Independent: pode ser desenvolvida...
- Negotiable: permite discussão...

## REGRAS DE QUALIDADE PARA AS USER STORIES
Cada história deve:
- representar uma necessidade real do negócio
- ter escopo claro
- evitar ambiguidade
- conter critérios de aceite suficientes para homologação
- respeitar o limite de uma necessidade funcional por história, sempre que possível
- não misturar objetivos distintos na mesma narrativa
- preservar o alinhamento com as premissas dos documentos anexos

## REGRAS DE LEITURA DOS DOCUMENTOS
Ao analisar os materiais fornecidos:
- Use o PRD de requisitos como base primária para comportamento e restrições.
- Use o PRD de overview como padrão de redação, organização e rigor formal.
- Extraia apenas o que for necessário para construir user stories.
- Não copie textos literalmente, exceto quando necessário para preservar termos oficiais do domínio.
- Não crie dependências que não estejam suportadas pelos insumos.

## FORMATAÇÃO DE SAÍDA
A saída deve ser organizada em Markdown e seguir esta estrutura:

# User Stories do Projeto

## Introdução
Resumo breve do objetivo do documento e do recorte analisado.

## User Story 01 — [Título]
### Narrativa
Como um ...
Eu quero ...
Para que ...

### Descrição
...

### Critérios de Aceite
- Dado que ...
  Quando ...
  Então ...
- Dado que ...
  Quando ...
  Então ...

### Contexto Adicional
...

### Anexos e Referências
...

### Subtarefas Técnicas
- ...
- ...

### Checklist INVEST
- Independent
  - [avaliação]
- Negotiable
  - [avaliação]
- Valuable
  - [avaliação]
- Estimable
  - [avaliação]
- Small
  - [avaliação]
- Testable
  - [avaliação]

## User Story 02 — [Título]
[Repita a mesma estrutura]

## Conclusão
Inclua apenas conclusões funcionais relevantes, sem propor código, refatoração ou plano técnico de implementação.

## INSTRUÇÃO FINAL
Gere as user stories com base estrita nos materiais fornecidos, preservando o valor de negócio, a rastreabilidade funcional e a clareza para o time de produto e desenvolvimento.
