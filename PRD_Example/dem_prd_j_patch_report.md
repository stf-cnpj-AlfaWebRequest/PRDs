# PRD — Relatório de Execução do Patch: CNPJ Numérico → Alfanumérico

## Descrição

Este PRD orienta a geração do relatório de execução do patch que compara os artefatos legados com os artefatos alterados e registra, com evidência, o que foi modificado, o que foi preservado e onde a execução aderiu ou divergiu do requisito e do levantamento de impacto. O foco é execução realizada, não rediscussão do requisito nem nova análise de impacto.

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

## Como usar

Forneça como contexto ao agente:

- Código legado (OLD): `.pss/a_old/a_code`
- Código alterado (NEW): `.pss/b_new/a_code`
- PRD de requisito base: `.pss/b_new/w_output/dem_a_cnpj_alfa_prd/dem_prd_requisito.md`
- Documentação de apoio (discovery, diagnostic e análise de impacto relacionados): `.pss/b_new/b_doc_b_dem`

Salvar resultado em: `.pss/b_new/b_doc_b_dem/dem_j_patch_report.md`

Formato: Markdown  
Idioma: Português (Brasil)

**Conclusão da seção:** o relatório deve ser gerado a partir de comparação real entre OLD e NEW, usando o PRD de requisito como base e os demais artefatos apenas como apoio de escopo, classificação e rastreabilidade.

---

## Regras de Qualidade Documental

- não gerar resumo executivo;
- cada seção principal deve começar com um parágrafo curto explicando o que ela cobre;
- cada seção principal deve terminar com uma conclusão breve;
- toda tabela deve ter, logo após ela, uma legenda explicando as colunas;
- usar Mermaid apenas quando o diagrama realmente ajudar a leitura;
- não repetir texto do PRD de requisito ou do levantamento de impacto; referenciar por ID de requisito ou por ID de impacto;
- não tratar exemplos deste PRD como achados reais da execução.

**Conclusão da seção:** o documento final deve ser objetivo, rastreável e enxuto, evitando teoria repetida e exemplos transformados em falso achado.

---

## Premissas Operacionais

- o relatório deve usar diff real entre OLD e NEW; não pode inferir mudança sem evidência textual;
- a base funcional e técnica é o PRD de requisito; discovery, diagnostic e impacto servem para confirmar escopo, classificação esperada e evidência prévia;
- a classificação deve ser consumida dos artefatos anteriores; se não houver base determinística suficiente, usar `Indefinido`;
- módulo interno e módulo externo devem ser classificados pelo prefixo do artefato em relação ao sistema principal;
- módulos externos não podem ser alterados e seus campos não entram na contagem do impacto interno; qualquer mudança neles é `Inconformidade Crítica`;
- identificadores recebidos de origem externa devem ser preservados exatamente como chegam quando o requisito determinar bloqueio de transformação;
- o relatório não deve transformar em sucesso uma execução que contrarie a classificação esperada do impacto ou do requisito.

**Conclusão da seção:** o relatório deve espelhar a execução real do patch, respeitar a classificação herdada dos artefatos anteriores e explicitar qualquer divergência em vez de normalizá-la.

---

## Cobertura Mínima do PRD de Requisito

O relatório deve incluir uma matriz única de cobertura dos requisitos aplicáveis do PRD base, sem reescrever o conteúdo integral de cada requisito. A matriz deve registrar evidência, status e observação para cada item aplicável.

Cobertura mínima obrigatória:

- `RF01` a `RF05`: coexistência de formatos, separação entre chave e valor, preservação do fluxo, adaptação dos pontos de uso e tratamento distinto em entradas e saídas;
- `RF06` a `RF10`: não exposição de chave, conversão via serviço central, determinismo sem correlação local, preservação de identificadores externos e bloqueio de transformação em casos `Indefinido`;
- `RT01` a `RT05`: tipagem, estrutura, contratos de dados, DV apenas no valor, compatibilidade mainframe e formatação diferenciada;
- `RT06` a `RT14`: busca/indexação, batch/online, auditabilidade, centralização da geração da chave, cache, distribuição batch, bloqueio de exposição da chave, bloqueio de transformação externa e controle de inferência.

Regras de preenchimento da matriz:

- usar os status `SIM`, `PARCIAL`, `NÃO`, `A VALIDAR` ou `N/A`;
- sempre citar a evidência objetiva: artefato, linha, diff, contrato ou ausência comprovada de alteração;
- quando OLD e NEW não permitirem concluir, usar `A VALIDAR` em vez de inferir conformidade;
- quando o requisito não se aplicar ao escopo real dos artefatos comparados, usar `N/A` com justificativa curta.

**Conclusão da seção:** o relatório deve ser rastreável ao PRD de requisito por ID, com cobertura explícita e sem duplicar a redação original do requisito.

---

## Regras de Comparação e Métricas

Esta seção define como comparar os artefatos e como calcular as métricas, evitando ambiguidade entre tamanho do artefato e quantidade real de linhas modificadas.

| Métrica | Regra obrigatória |
| :--- | :--- |
| Diferença de tamanho do artefato | `linhas NEW - linhas OLD`; pode ser `0` mesmo quando houver alteração de conteúdo. |
| Linhas alteradas | Quantidade de substituições entre OLD e NEW no mesmo hunk lógico. Para um hunk com `n` linhas removidas e `m` linhas adicionadas, contar `min(n, m)` como linhas alteradas. |
| Linhas adicionadas | Apenas adição líquida. Para um hunk com `n` linhas removidas e `m` linhas adicionadas, contar `max(m - n, 0)`. |
| Linhas removidas | Apenas remoção líquida. Para um hunk com `n` linhas removidas e `m` linhas adicionadas, contar `max(n - m, 0)`. |
| Delta estrutural de campo | `bytes NEW - bytes OLD`, calculado no campo efetivamente alterado. |
| Delta estrutural de registro | diferença real entre o tamanho do registro em OLD e em NEW; não assumir delta fixo por quantidade de campos. |
| Complexidade | `linhas adicionadas + linhas removidas + linhas alteradas`. |

> **Legenda:** **Métrica** = indicador a ser apresentado no relatório; **Regra obrigatória** = forma de cálculo que o agente deve usar.

Regras adicionais:

- se uma linha foi substituída por outra na mesma posição lógica, a métrica de `linhas alteradas` não pode ficar zerada;
- não usar `Delta: 0` como sinônimo de `sem mudança`; `0` em diferença de tamanho do artefato significa apenas que o número total de linhas permaneceu igual;
- preservar a numeração de linhas COBOL original quando ela existir nas colunas 1 a 6;
- usar bloco `diff` para comparação e bloco `cobol` apenas quando houver necessidade de contexto adicional;
- agrupar alterações contíguas do mesmo artefato em um único bloco diff, evitando um bloco por linha sem necessidade;
- se o patch contrariar a classificação esperada de um campo, registrar como divergência ou inconformidade, não como implementação bem-sucedida;
- se houver compensação estrutural por `FILLER`, `REDEFINES` ou ajuste equivalente, registrar o delta do campo e o delta real do registro separadamente.

**Conclusão da seção:** a contagem de linhas deve refletir substituições reais, e a métrica de tamanho do artefato deve permanecer separada da métrica de alteração de conteúdo.

---

## Estrutura Obrigatória do Documento Gerado

### 1. Identificação da execução

Abrir o documento informando demanda, requisito base, artefatos consultados, escopo da comparação e data da geração.

Um diagrama Mermaid falandos Sobre artefatos alterados, por módulo, visão arquitetural do sitema, perceba que: se for chave, e o artefato não for alterado, o diagrama TEM QUE refletir isso!!


### 2. Inventário comparativo de artefatos

Registrar, em uma única visão, os artefatos analisados, seu status e a aderência ao impacto esperado.

| ID | Artefato | Tipo | Status | Classificação evidenciada | Impacto esperado | Impacto executado | Conformidade |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- | :--- |

> **Legenda:** **ID** = identificador rastreável do item; **Artefato** = artefato ou contrato comparado; **Tipo** = programa, copybook, interface, JCL, DB2 ou outro; **Status** = modificado, preservado, novo artefato ou ausente; **Classificação evidenciada** = Chave, Valor, Ambos, Indefinido ou N/A; **Impacto esperado** = o que o levantamento previa; **Impacto executado** = o que a comparação OLD × NEW mostrou; **Conformidade** = aderente, divergente, crítico ou N/A.

### 3. Análise comparativa por artefato

Para cada artefato analisado, registrar somente o que é útil para demonstrar a execução real.

Cabeçalho obrigatório por artefato:

```text
artefato: [NOME]
Tipo: [Programa COBOL / Copybook / Interface / Outro]
Status: [Modificado / Preservado / Novo artefato / Não localizado]
Classificações presentes: [Chave / Valor / Ambos / Indefinido / N/A]
Linhas OLD: [n]
Linhas NEW: [n]
Diferença de tamanho do artefato: [n]
Linhas adicionadas: [n]
Linhas removidas: [n]
Linhas alteradas: [n]
```

Tabela obrigatória de mudanças por artefato:

| Campo/Elemento | Classificação esperada | Classificação evidenciada | Tipo OLD | Tipo NEW | Bytes OLD | Bytes NEW | Linha OLD | Linha NEW | Tipo de mudança | ID Impacto | Status |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :---: | :---: | :--- | :--- | :--- |

> **Legenda:** **Campo/Elemento** = variável, coluna, layout, chamada ou trecho alterado; **Classificação esperada** = classificação herdada dos artefatos anteriores; **Classificação evidenciada** = o que o diff e o contexto mostram; **Tipo OLD/NEW** = tipagem ou forma estrutural antes e depois; **Bytes OLD/NEW** = tamanho do elemento quando aplicável; **Linha OLD/NEW** = linha evidenciada em cada versão; **Tipo de mudança** = Tipagem, Layout, Lógica, Validação, Integração, LookUp, Preservação, Divergência ou outro rótulo objetivo; **ID Impacto** = referência ao levantamento de impacto; **Status** = implementado, preservado, divergente, inconclusivo ou N/A.

Tabela opcional de campos preservados por regra, quando aplicável:

| Campo/Elemento | Motivo da preservação | Evidência | Regra relacionada |
| :--- | :--- | :--- | :--- |

> **Legenda:** **Campo/Elemento** = item mantido sem alteração; **Motivo da preservação** = por exemplo origem externa, módulo externo, campo indefinido ou sem impacto; **Evidência** = linha, contrato ou ausência comprovada de diff; **Regra relacionada** = RF, RT ou regra do impacto que justifica a preservação.

Diff obrigatório:

- apresentar todos os hunks relevantes de cada artefato modificado;
- usar `diff` e agrupar mudanças contíguas;
- não gerar diff para artefato sem alteração;
- após cada bloco, classificar o hunk e citar a referência de impacto ou requisito relacionada.

### 4. Impacto estrutural em records, layouts e contratos

Esta seção deve existir somente quando houver mudança estrutural relevante em FD, registros, copybooks, layouts, contratos ou tamanho físico.

| Artefato | Estrutura | Bytes OLD | Bytes NEW | Delta real | Motivo do delta | Observação |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- |

> **Legenda:** **Artefato** = artefato ou contrato afetado; **Estrutura** = record, FD, copybook, layout ou coluna; **Bytes OLD/NEW** = tamanho antes e depois; **Delta real** = diferença efetiva calculada; **Motivo do delta** = alteração de campo, compensação por filler, redefine ou outro ajuste; **Observação** = detalhe necessário para interpretar o resultado.

### 5. Conformidade com impacto e requisito

O relatório deve separar aderência ao levantamento de impacto da aderência ao PRD de requisito.

Tabela obrigatória de conformidade com impacto:

| ID Impacto | Artefato | Item esperado | Evidência de execução | Status | Observação |
| :---: | :--- | :--- | :--- | :--- | :--- |

> **Legenda:** **ID Impacto** = identificador do levantamento de impacto; **Artefato** = item relacionado; **Item esperado** = alteração ou preservação prevista; **Evidência de execução** = linha, diff, estrutura ou ausência de mudança; **Status** = SIM, PARCIAL, NÃO, A VALIDAR ou N/A; **Observação** = divergência, limitação ou justificativa.

Tabela obrigatória de cobertura do PRD de requisito:

| Requisito | Evidência | Status | Observação |
| :---: | :--- | :---: | :--- |

> **Legenda:** **Requisito** = RF ou RT do PRD base; **Evidência** = trecho do comparativo que sustenta a conclusão; **Status** = SIM, PARCIAL, NÃO, A VALIDAR ou N/A; **Observação** = justificativa curta para exceção, lacuna ou não aplicabilidade.

### 6. Inconformidades e riscos

Consolidar somente achados reais ou riscos diretamente suportados pela comparação e pelos artefatos de apoio.

| ID | Artefato/Camada | Inconformidade ou risco | Violação associada | Severidade | Evidência |
| :---: | :--- | :--- | :--- | :---: | :--- |

> **Legenda:** **ID** = identificador rastreável do achado; **Artefato/Camada** = onde o problema ocorre; **Inconformidade ou risco** = descrição objetiva do achado; **Violação associada** = RF, RT, regra de impacto ou regra operacional quebrada; **Severidade** = baixa, média, alta ou crítica; **Evidência** = diff, linha, contrato ou ausência comprovada de implementação.

Verificações mínimas obrigatórias:

- alteração em módulo externo;
- exposição de `Chave` em interface externa, layout de borda ou contrato de negócio;
- ausência de evidência de serviço oficial de conversão quando o ponto de borda exigir lookup;
- transformação indevida de identificador de origem externa;
- modificação de item `Indefinido` sem base determinística;
- coerção numérica, truncamento ou tipagem incompatível em campo `Valor`;
- divergência entre classificação esperada e execução realizada;
- mudança estrutural sem recálculo real do registro;
- mudança em busca, índice, ordenação, JOIN, batch, cache ou distribuição sem evidência suficiente para concluir conformidade.

### 7. Métricas consolidadas

Apresentar visão por artefato e visão agregada, usando a regra de cálculo definida neste PRD.

| artefato | Linhas adicionadas | Linhas removidas | Linhas alteradas | Campos alterados | Complexidade |
| :--- | :---: | :---: | :---: | :---: | :--- |

> **Legenda:** **artefato** = artefato comparado; **Linhas adicionadas/removidas/alteradas** = métricas calculadas segundo a regra desta especificação; **Campos alterados** = quantidade de elementos impactados no artefato; **Complexidade** = baixa, média ou alta conforme o volume consolidado de mudança.

Tabela agregada obrigatória:

| Métrica | Valor |
| :--- | :---: |
| Total linhas adicionadas | |
| Total linhas removidas | |
| Total linhas alteradas | |
| Total campos alterados | |
| Complexidade geral | |

> **Legenda:** **Métrica** = indicador consolidado do relatório; **Valor** = total apurado pela comparação real entre OLD e NEW.

Faixas sugeridas para complexidade geral:

- `Baixa`: até 30 linhas consolidadas de mudança;
- `Média`: de 31 a 100 linhas consolidadas de mudança;
- `Alta`: acima de 100 linhas consolidadas de mudança.

### 8. Diagrama de dependências, quando útil

Gerar diagrama Mermaid apenas quando houver mais de um artefato alterado, dependência cruzada relevante ou necessidade de explicar propagação da mudança. Não gerar diagrama decorativo.

### 9. Pendências e limitações

Encerrar o relatório com pendências objetivas e limitações reais da comparação, especialmente quando algum requisito ficar em `A VALIDAR` ou `N/A`.

**Conclusão da seção:** o documento final deve concentrar evidência de execução, conformidade e divergência em poucas estruturas úteis, sem repetir tabelas que descrevem o mesmo fato com nomes diferentes.

---

## Instruções Finais ao Agente

- não gerar conteúdo sobre teste, rollout, rollback, monitoramento ou recomendação futura;
- não assumir que um artefato modificado está conforme apenas porque o nome do campo sugere `Chave` ou `Valor`;
- não usar valores, campos ou conclusões exemplificativas deste PRD como se fossem resultados reais da comparação;
- quando houver conflito entre o patch executado e o requisito, o relatório deve registrar a divergência com evidência;
- quando houver conflito entre discovery, diagnostic, impacto e patch, o relatório deve explicitar a divergência por artefato e por campo;
- o documento gerado deve permanecer enxuto e orientado a evidência, usando o PRD de requisito como base e evitando duplicação desnecessária.

Documento gerado: `.pss/b_new/b_doc_b_dem/dem_j_patch_report.md`

**Conclusão da seção:** o agente deve produzir um relatório de execução compacto, rastreável e fiel ao que foi realmente alterado entre OLD e NEW, incluindo conformidade, divergência e métricas calculadas corretamente.

