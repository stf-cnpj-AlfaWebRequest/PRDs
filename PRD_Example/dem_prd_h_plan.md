# PRD — Plano de Execução: Adequação ao CNPJ Alfanumérico em Ambiente Mainframe

---

# Definições

**Chave de CNPJ:** Identificador técnico interno do CNPJ. Representa a chave primária ou estrangeira utilizada internamente pelo sistema para relacionamentos, indexação e navegação de dados. Não deve ser tratada como documento oficial de negócio. Exemplos: `78465270000165`, `00000000000001`.

**Valor de CNPJ:** Conteúdo do CNPJ como dado de negócio, utilizado em regras de negócio, cálculos, validações, telas (GUI), interfaces externas e relatórios. É o identificador oficial utilizado pelo negócio. Exemplos: `78465270000165`, `6QNJ8VY2JIC341`.

**Módulos:** Qualquer artefato de código, incluindo programas COBOL e copybooks (`.CPY`) utilizados pelos programas COBOL. Exemplos de prefixos: RF, DAF, VIP, HLP.

**Módulo Principal:** Módulo que pertence ao mesmo sistema dos programas principais. Identificado dinamicamente pelo prefixo do nome do artefato. Pode ou não requerer alteração, mas pertence ao mesmo time e sistema. Exemplo: se os programas principais são `VIPP4365` e `VIPP4553`, qualquer artefato com prefixo `VIP` é considerado principal.

**Módulo Secundário:** Módulo que pertence a um sistema diferente dos programas principais, pertencente a outro time ou domínio. Identificado dinamicamente pelo prefixo do nome do artefato. Exemplo: artefatos com prefixo `DAF` ou `HLP` são secundários em relação a programas `VIP`.

**Artefato:** Qualquer unidade técnica versionável que compõe, suporta ou influencia a execução do sistema. Inclui programas (`.cbl`), copybooks (`.cpy`), JCLs, artefatos de dados, layouts, tabelas e demais elementos estruturais. É tratado como elemento rastreável, analisável e passível de impacto.

**Programas:** Artefatos de código COBOL (`.CBL`, `.CPY`, entre outros) que contêm a lógica de negócio e que serão modificados pelo patch. Seguem o padrão de um prefixo de três letras maiúsculas seguido de um número. Exemplos: `VIPP4365.cbl`, `DAF1234.cbl`, `HLP5678.cbl`.

**Correlação 1:1:** Relação obrigatória e inequívoca entre uma Chave de CNPJ e um Valor de CNPJ, preservando rastreabilidade, integridade relacional, auditabilidade e determinismo da conversão.

**CGC:** Denominação anterior do CNPJ (Cadastro Geral de Contribuintes), equivalente ao CNPJ para todos os efeitos desta demanda. Campos nomeados como CGC devem ser tratados com as mesmas regras aplicáveis ao CNPJ.

**PF (Pessoa Física):** Indivíduo, pessoa natural. Campos relacionados a PF não estão no escopo desta demanda, mas podem ser utilizados para comparação e inferência quando necessário.

**PJ (Pessoa Jurídica):** Empresa, organização. Campos relacionados a PJ estão no escopo desta demanda, pois o CNPJ é o identificador oficial de pessoas jurídicas.

**Fonte Externa:** Origem de dados pertencente a um sistema externo ao escopo desta modificação, como tabelas DB2 de outros domínios (ex.: `DB2MCI.CLIENTE`), artefatos legados de terceiros ou integrações de entrada. Dados provenientes de fontes externas não estão no escopo de modificação e não devem ser alterados.

**Campo Indefinido:** Campo de CNPJ/CGC cuja classificação semântica entre **Chave** e **Valor** não possa ser determinada de forma objetiva, rastreável e suficientemente confiável com base nos artefatos de **discovery**, **diagnóstico**, análise de código, contratos de interface, layouts, regras de negócio e contexto de processamento.

    Um campo deve ser classificado como **Indefinido** quando os elementos analisados não fornecerem evidência técnica suficiente para sustentar, com segurança, uma decisão inequívoca sobre sua natureza funcional e estrutural no fluxo. Nessa condição, o campo **não é elegível para alteração**, devendo ser **preservado em seu estado atual** até que evidências adicionais permitam sua reclassificação formal.

    A inferência será considerada **fraca** sempre que houver qualquer grau de incerteza residual na classificação, isto é, quando o nível de confiança da análise for inferior a 100% e a incerteza apurada for superior a 0%.

**Serviço Central de Conversão:** Serviço oficial responsável por converter Chave de CNPJ em Valor de CNPJ e vice-versa. É proibida a geração local ou distribuída dessa correlação fora do sistema central autorizado.

**Base Central de Correlação (DFE):** Estrutura central responsável por manter a relação 1:1 entre Chave de CNPJ e Valor de CNPJ, além de metadados de controle e auditoria.

**Cache:** Camadas auxiliares de desempenho utilizadas para reduzir acessos à base central de correlação. Pode incluir mecanismos equivalentes a Redis, VSAM, processamento batch ou estruturas legadas compatíveis com o ambiente mainframe.

### Módulo Secundário

A definição do módulo secundário é tão importante que há uma definição prática e consolidada para ele, que deve ser seguida rigorosamente. O módulo secundário é o nome dado a um módulo que pertence a um sistema diferente dos programas principais. Módulos secundários **não são modificados por esta demanda**, pois pertencem a outros times/sistemas. São, entretanto, **objeto de diagnóstico integral**: todas as ocorrências de CNPJ identificadas na etapa de discovery dentro de módulos secundários devem ser analisadas e registradas para garantir rastreabilidade completa upstream e downstream, sem que isso implique qualquer autorização de alteração. A identificação de módulos secundários é feita dinamicamente com base no prefixo do nome do artefato. (ex: se os programas principais são VIPP4365 e VIPP4553, então artefatos com prefixo DAF ou HLP são considerados secundários).

---

### Definição de Módulo Secundário (Externo)

**Módulo Secundário (Externo)** é todo artefato técnico que **pertence a sistema, domínio funcional, contexto operacional ou responsabilidade de time diferente** em relação aos programas principais da demanda, ainda que mantenha relação funcional, estrutural, operacional ou contratual com o fluxo analisado.

No contexto desta documentação, o termo **externo** não significa necessariamente fora da aplicação em sentido físico, fora do repositório ou fora da plataforma mainframe. Significa, tecnicamente, que o artefato está **fora do núcleo principal da implementação**, pertencendo a **outro módulo lógico**, **outro agrupamento funcional** ou **outro contexto sistêmico**, podendo inclusive estar sob responsabilidade de **outro time**.

Um módulo secundário é, portanto, um artefato que **não compõe o conjunto principal de programas da demanda**, mas que pode participar do processamento por meio de chamadas, integrações, compartilhamento de estruturas, consumo de arquivos, uso de copybooks, trocas de dados ou dependências de execução.

#### Identificação Dinâmica

A identificação de um módulo secundário deve ser feita de forma **dinâmica**, com base na análise técnica do artefato e, em especial, na **convenção de nomenclatura e no prefixo do nome do artefato**.

Essa identificação não deve ser tratada como cadastro fixo ou lista estática pré-definida. A LLM deve inferir a classificação comparando o prefixo do artefato analisado com o prefixo dos programas principais da demanda.

Exemplo:

* se os programas principais da demanda forem **VIPP4365** e **VIPP4553**, então o prefixo principal do escopo é **VIP**;
* nesse cenário, artefatos com prefixos como **DAF** ou **HLP** devem ser tratados, em regra, como **módulos secundários**, por pertencerem a outro módulo, outro sistema ou outro domínio em relação ao núcleo principal.

Assim, artefatos com prefixo **DAF** ou **HLP** são secundários em relação a programas **VIP**, salvo quando a documentação da demanda dispuser explicitamente em sentido diverso.

#### Relação com os Programas Principais

Os programas principais são os artefatos diretamente vinculados ao escopo central da implementação. Já o módulo secundário externo é o artefato que, embora tecnicamente relacionado ao fluxo, **não representa o ponto principal da alteração solicitada**.

Ele pode:

* ser chamado pelo programa principal;
* chamar o programa principal;
* compartilhar copybooks, layouts ou áreas de comunicação;
* trocar dados com o fluxo principal;
* manipular campos correlatos ao CNPJ;
* participar de validações, integrações, consultas, persistência, formatação, enriquecimento ou apoio operacional.

Ainda assim, sua participação no fluxo **não o transforma automaticamente em módulo principal**.

#### Pertencimento a Outro Sistema, Time ou Domínio

Um dos critérios centrais para caracterização do módulo secundário é o seu pertencimento a **outro sistema**, **outro domínio funcional** ou **outro contexto de manutenção**, frequentemente sob responsabilidade de **outro time**.

Por essa razão, módulos secundários exigem tratamento mais controlado sob a ótica de governança, pois uma alteração nesses artefatos extrapolaria o escopo original da demanda e produziria impacto em fluxos paralelos, contratos técnicos compartilhados ou responsabilidades sistêmicas de terceiros.

#### Possibilidade de Alteração

Módulos secundários **não são modificados por esta demanda**. Sua classificação como módulo secundário é, por si só, critério suficiente para exclusão do escopo de modificação, independentemente da presença de CNPJ, da relevância técnica do artefato ou de sua participação no fluxo de processamento.

A proibição de alteração é incondicional e não depende de análise adicional. Não há autorização possível para modificação de módulo secundário no âmbito desta demanda.

O módulo secundário deve ser tratado como **artefato analisável e registrável**, cujo diagnóstico é obrigatório para fins de rastreabilidade upstream e downstream, mas **não elegível para modificação em nenhuma circunstância**.

#### Características Técnicas Comuns

Um módulo secundário externo normalmente apresenta uma ou mais das seguintes características:

* pertence a prefixo diferente do prefixo dos programas principais;
* pertence a outro sistema, outro time ou outro domínio funcional;
* atua como suporte, integração, utilitário, consulta, persistência ou serviço compartilhado;
* não concentra a regra principal da demanda;
* participa do fluxo apenas como dependência técnica ou funcional;
* pode manipular CNPJ, mas sem ser o núcleo da implementação;
* está fora da lista principal de artefatos-alvo da alteração.

#### Relação com a Manipulação de CNPJ

A presença de CNPJ em um módulo secundário não altera, por si só, sua classificação.

Um módulo secundário pode:

* receber CNPJ como entrada;
* retornar CNPJ como saída;
* transportar CNPJ em registros;
* compartilhar estruturas com campos de CNPJ;
* usar CNPJ como **Chave**;
* usar CNPJ como **Valor**.

Mesmo nesses casos, a classificação como módulo secundário permanece válida e a proibição de alteração permanece absoluta. O diagnóstico de tais ocorrências é obrigatório para garantir a rastreabilidade da cadeia de impacto, mas não constitui autorização para modificação.

#### Exemplos Típicos

Exemplos comuns de módulos secundários externos incluem:

* **DAF**
* **HLP**
* rotinas utilitárias
* programas de apoio
* artefatos de integração
* módulos de consulta compartilhados
* componentes pertencentes a outros agrupamentos funcionais

Exemplo prático:

* Programas principais: **VIPP4365**, **VIPP4553**
* Prefixo principal inferido: **VIP**
* Artefatos com prefixos **DAF** e **HLP**: classificados como **módulos secundários**
* Motivo: pertencem a outro módulo/sistema/domínio em relação ao núcleo principal
* Conduta: diagnosticados integralmente para rastreabilidade; não elegíveis para modificação

#### Definição Consolidada

Em termos consolidados, **Módulo Secundário (Externo)** é o nome dado ao artefato que:

* pertence a sistema, time ou domínio diferente dos programas principais;
* é identificado dinamicamente, principalmente pelo prefixo do nome do artefato e pela análise contextual;
* pode manter relação técnica com o fluxo principal;
* **não pode ser modificado** no âmbito desta demanda;
* deve ser **diagnosticado integralmente** para garantir rastreabilidade upstream e downstream de todos os campos CNPJ descobertos na etapa de discovery;
* não deve ser tratado como parte do escopo de alteração apenas por participar do processamento ou por manipular campos de CNPJ.

### Definição Técnica de Origem de Dados

**Origem de Dados** é a caracterização técnica que determina **de qual sistema, módulo, processo, arquivo, tabela ou contrato o dado se origina**, bem como **qual domínio funcional detém a responsabilidade de produção, manutenção, controle e governança sobre seu conteúdo**.

Em ambiente **mainframe**, a origem de dados não deve ser inferida apenas pela presença física do campo em um programa COBOL, copybook, layout, FD, registro, área de comunicação, tabela DB2, arquivo VSAM, arquivo sequencial, mensagem ou estrutura intermediária. A presença do campo em tais artefatos pode representar apenas **declaração**, **transporte**, **repasse**, **espelhamento**, **persistência transitória** ou **consumo operacional**, sem caracterizar, por si só, a verdadeira origem do dado.

A determinação da origem deve considerar, de forma combinada:

- o **sistema produtor** do dado;
- o **módulo responsável** pela geração ou manutenção do conteúdo;
- a **estrutura de origem** que atua como fonte de verdade;
- o **domínio funcional** que possui governança sobre o valor;
- o **contexto operacional** em que o dado é recebido, transformado, propagado, persistido ou consumido.

No contexto mainframe, a origem de dados pode estar associada, por exemplo, a:

- programa COBOL produtor;
- transação on-line;
- fluxo batch;
- etapa de JCL;
- arquivo de entrada;
- arquivo de saída;
- arquivo VSAM;
- tabela DB2;
- copybook corporativo;
- COMMAREA ou outra área de comunicação;
- mensagem recebida de integração;
- contrato de interface entre sistemas;
- estrutura proveniente de módulo secundário externo.

A análise da origem deve observar, no mínimo, três dimensões técnicas:

#### Origem Lógica
Refere-se ao sistema, domínio ou processo que **efetivamente cria ou controla o significado funcional do dado**.

#### Origem Física
Refere-se ao artefato em que o dado está materializado em determinado ponto do fluxo, como tabela DB2, arquivo, layout, copybook, registro ou área de comunicação.

#### Origem Operacional
Refere-se ao ponto do processamento em que o dado é introduzido, recebido, propagado ou consumido no fluxo técnico, como uma transação on-line, um step batch, uma leitura de arquivo, uma consulta DB2 ou uma interface entre módulos.

A origem de dados deve ser classificada com base principalmente na **origem lógica**, utilizando a origem física e a origem operacional como elementos de apoio à inferência técnica.

Um dado deve ser tratado como de **origem interna** quando sua produção, manutenção, controle e governança pertencem ao próprio sistema em intervenção e ao escopo funcional da demanda.

Um dado deve ser tratado como de **origem externa** quando:

- é produzido, mantido ou controlado por sistema, módulo, domínio ou processo externo ao escopo da demanda; ou
- é apenas declarado, propagado ou manipulado por artefato pertencente a módulo secundário externo, sem que o sistema em intervenção detenha governança sobre o conteúdo.

Em ambiente mainframe, é obrigatório distinguir entre:

- **artefato que transporta o dado**; e
- **artefato que é a verdadeira origem do dado**.

Assim, copybooks, layouts, FDs, registros, áreas de comunicação, arquivos intermediários, estruturas de staging e programas utilitários **não devem ser tratados automaticamente como origem**, salvo quando forem também o ponto efetivo de produção e governança do conteúdo.

A definição de origem de dados é essencial para determinar a conduta técnica permitida sobre o campo, pois estabelece se o sistema em intervenção possui autoridade para:

- transformar;
- converter;
- validar;
- normalizar;
- reclassificar;
- enriquecer;
- persistir;
- ou apenas transportar e preservar o dado.

Portanto, em ambiente mainframe, **Origem de Dados** é o atributo técnico que identifica a procedência lógica, física e operacional de um dado no ecossistema de processamento, permitindo distinguir se o conteúdo pertence ao domínio do sistema em intervenção ou se deve ser tratado como informação externa, preservada conforme seu contrato de origem.

---

# Contexto

Este documento é o PRD que governa a produção do Plano de Execução para adequação ao CNPJ alfanumérico em ambiente mainframe. Ele deve ser lido e executado por um agente especializado, que produzirá o documento `dem_h_plan.md` como saída.

O agente atuará como especialista sênior em Arquitetura de Software, Sistemas Legados, Integração de Sistemas, Engenharia de Dados e Gestão de Projetos.

O objetivo do Plano de Execução é detalhar a estratégia operacional para aplicação de mudança de requisito visando o atendimento à regulação federal do CNPJ alfanumérico, consumindo os seguintes artefatos de entrada sem replicar seu conteúdo:

- `dem_prd_requisito.md`
- `dem_a_overview_1.md`
- `dem_b_user_story.md`
- `dem_c_functional_1.md`
- `dem_d_technical_1.md`
- `dem_e_discovery.md`
- `dem_f_diagnostic.md`
- `dem_g_impact.md`

O foco do plano produzido deve ser exclusivamente operacional e de execução.

---

# Artefatos de Entrada

| Artefato | Finalidade | Leitura Obrigatória |
| :--- | :--- | :---: |
| `.pss/b_new/w_output/dem_a_cnpj_alfa_prd/dem_prd_requisito.md` | Base funcional e técnica obrigatória | Sim |
| `.pss/b_new/b_doc_b_dem` | Demais documentos de referência do projeto | Sim — todos disponíveis |
| `.pss/a_old/a_code` | Código-fonte a ser lido para entendimento do plano de execução | Sim — todos os arquivos do escopo |

---

# Artefato de Saída

O documento final a ser gerado pelo agente é:

`.pss/b_new/b_doc_b_dem/dem_h_plan.md`

O documento deve refletir este PRD de plano de execução já limpo, preservando somente o que auxilia a análise e cobrindo explicitamente os requisitos do PRD base.

---

# Objetivo do Plano

O Plano de Execução gerado pelo agente deve conter:

- Estratégias de execução com opções para decisão do cliente
- Ordem lógica de execução
- Mapeamento de dependências
- Controle de risco
- Observabilidade e monitoramento
- Rastreabilidade de artefatos

O plano não deve escolher a estratégia de execução. Deve apresentar opções para que o cliente tome a decisão.

---

# Escopo

O escopo é restrito ao tratamento do CNPJ. São proibidos:

- Criação de código ou scripts
- Repetição de conteúdo do diagnóstico
- Classificação própria de campos como Chave ou Valor
- Alteração de módulos externos ao escopo autorizado

---
# Premissas

### Premissa de Estratégia de Execução Adotada

Esta PoC adota como premissa a execução pelo caminho de menor complexidade técnica. A progressão seguirá uma abordagem de esforço e risco crescente, iniciando pelos artefatos de menor acoplamento, cadeia de impacto mais curta e classificação determinística no Diagnóstico, avançando para etapas de maior complexidade somente após a validação da etapa anterior. Nenhuma antecipação de etapa é permitida por conveniência operacional ou pressão de prazo sem reclassificação formal do cenário. A decisão por esta estratégia é técnica, não executiva, e está condicionada exclusivamente às evidências produzidas pelo Diagnóstico — sendo o resultado de negócio e os fatores estratégicos externos considerados apenas como contexto informativo, sem poder de alteração do sequenciamento definido.

---

### Premissas de Produção

* Não alterar artefatos diretamente
* Linguagem técnica em todo o documento
* Documento produzido em português
* Não expor caminhos físicos de sistema

---

### Premissa Técnica — Não Criação de Nova Versão de Artefatos (V2)

Nesta demanda, **não será adotada estratégia de versionamento funcional de artefatos**, tais como criação de programas, copybooks, layouts, JCLs, PROCedures, mapas ou componentes correlatos com sufixação de versão lógica ou física, como `V2`, `NEW`, `_N`, `_ALT` ou equivalentes. A atuação deverá ocorrer **sobre os artefatos correntes já homologados na cadeia operacional da plataforma alta**, por meio de **ajustes pontuais e controlados**, preservando nomenclatura, identidade física, contratos de execução, encadeamentos batch, referências cruzadas, catálogo de carga e vínculos operacionais já existentes no ambiente mainframe.

Essa premissa visa evitar **duplicidade de objetos**, **desvio de roteamento**, **quebra de dependências estáticas**, **aumento desnecessário da superfície de manutenção** e **complexidade adicional de promoção entre ambientes**, assegurando que a solução permaneça integrada ao ciclo normal de compilação, linkedição, implantação e operação produtiva, sem introdução de uma trilha paralela de artefatos versionados.


---

### Premissas de Execução Pontual

**Premissa de Execução Pontual:** A demanda de adequação ao CNPJ alfanumérico opera em dois regimes distintos de abrangência, aplicados sequencialmente e com propósitos complementares, não intercambiáveis.

**No Diagnóstico**, a abrangência é **máxima e irrestrita dentro do escopo de discovery**. Todos os artefatos identificados na etapa de Descoberta devem ser analisados integralmente, independentemente da expectativa de alteração. Isso inclui artefatos de módulos secundários, cujo diagnóstico é obrigatório para garantir rastreabilidade upstream e downstream de toda a cadeia de impacto. A análise não deve ser antecipadamente filtrada por critério de relevância, pois a relevância só pode ser determinada após a análise. A priorização da intervenção é consequência da análise — não sua premissa. Artefatos com alta taxa de inferência de impacto determinam o grau e a urgência da intervenção. Artefatos com baixa taxa de inferência são igualmente analisados, mas resultam em classificação como Indefinido e são preservados. O Diagnóstico não decide o que alterar: ele produz a evidência técnica que fundamenta essa decisão.

**No Plano de Execução**, a abrangência é **mínima e estritamente delimitada pelo Diagnóstico**. Esta demanda não caracteriza modificação ampla, reestruturação integral, refatoração global ou modernização irrestrita de código. A execução deve ser tratada como intervenção pontual, limitada ao conjunto exato de artefatos e campos pertencentes ao módulo principal para os quais o Diagnóstico produziu evidência determinística de necessidade de alteração. **Módulos secundários não compõem o escopo de execução do plano.** Qualquer referência a módulos secundários no plano de execução tem finalidade exclusivamente informativa, destinada a subsidiar ações de contingência, controle de impacto e gestão de risco decorrentes das alterações aplicadas nos módulos principais. Essas referências sinalizam ao time responsável pelo módulo secundário a necessidade de tratamento equivalente em sua própria rodada de execução, sem gerar qualquer autorização de modificação no âmbito desta demanda. Nenhuma modificação pode ser realizada por conveniência técnica, padronização, estética, organização ou oportunidade de melhoria paralela. A intervenção autorizada é exclusivamente aquela rastreável a uma ocorrência concreta identificada no Diagnóstico, com classificação confiável e ação definida, em artefato pertencente ao módulo principal. Todo o restante deve ser preservado intacto.

> A assimetria entre os dois regimes é intencional e estrutural: o Diagnóstico é amplo para garantir que nenhum risco seja ignorado; o Plano é restrito para garantir que nenhuma alteração seja indevida.

## Premissa Estratégia de Execução Adotada

Esta PoC adota como premissa a execução pelo caminho de menor complexidade técnica. A progressão seguirá uma abordagem de esforço e risco crescente, iniciando pelos artefatos de menor acoplamento, cadeia de impacto mais curta e classificação determinística no Diagnóstico, avançando para etapas de maior complexidade somente após a validação da etapa anterior. Nenhuma antecipação de etapa é permitida por conveniência operacional ou pressão de prazo sem reclassificação formal do cenário. A decisão por esta estratégia é técnica, não executiva, e está condicionada exclusivamente às evidências produzidas pelo Diagnóstico — sendo o resultado de negócio e os fatores estratégicos externos considerados apenas como contexto informativo, sem poder de alteração do sequenciamento definido.

## Premissas de Produção

- Não alterar artefatos diretamente
- Linguagem técnica em todo o documento
- Documento produzido em português
- Não expor caminhos físicos de sistema

---

## Premissas de Execução Pontual

**Premissa de Execução Pontual:** A demanda de adequação ao CNPJ alfanumérico opera em dois regimes distintos de abrangência, aplicados sequencialmente e com propósitos complementares, não intercambiáveis.

**No Diagnóstico**, a abrangência é **máxima e irrestrita dentro do escopo de discovery**. Todos os artefatos identificados na etapa de Descoberta devem ser analisados integralmente, independentemente da expectativa de alteração. A análise não deve ser antecipadamente filtrada por critério de relevância, pois a relevância só pode ser determinada após a análise. A priorização da intervenção é consequência da análise — não sua premissa. Artefatos com alta taxa de inferência de impacto determinam o grau e a urgência da intervenção. Artefatos com baixa taxa de inferência são igualmente analisados, mas resultam em classificação como Indefinido e são preservados. O Diagnóstico não decide o que alterar: ele produz a evidência técnica que fundamenta essa decisão.

**No Plano de Execução**, a abrangência é **mínima e estritamente delimitada pelo Diagnóstico**. Esta demanda não caracteriza modificação ampla, reestruturação integral, refatoração global ou modernização irrestrita de código. A execução deve ser tratada como intervenção pontual, limitada ao conjunto exato de artefatos e campos para os quais o Diagnóstico produziu evidência determinística de necessidade de alteração. Nenhuma modificação pode ser realizada por conveniência técnica, padronização, estética, organização ou oportunidade de melhoria paralela. A intervenção autorizada é exclusivamente aquela rastreável a uma ocorrência concreta identificada no Diagnóstico, com classificação confiável e ação definida. Todo o restante deve ser preservado intacto.

>A assimetria entre os dois regimes é intencional e estrutural: o Diagnóstico é amplo para garantir que nenhum risco seja ignorado; o Plano é restrito para garantir que nenhuma alteração seja indevida.

---


# Cenários de Classificação de Uso do CNPJ

O termo **Cenário** refere-se ao papel funcional e técnico desempenhado pelo CNPJ dentro do programa ou fluxo de processamento, permitindo classificar o uso do dado em Chave ou Valor.

A classificação deve ser realizada com base na análise contextual do uso do campo no código-fonte e na sua função dentro da arquitetura de dados, considerando:

- O comportamento funcional do campo no processamento
- Sua participação em operações técnicas de indexação, relacionamento e filtros
- Sua semântica de negócio: exibição, comunicação, interface

## CNPJ como Chave

- Utilizado como identificador interno para relacionamentos ou acesso a dados
- Participa de JOINs, MATCHs, índices, buscas, validações de integridade ou relacionamentos entre registros
- Pode estar associado a constraints, ponteiros lógicos, chaves primárias ou estrangeiras, ou estruturas de lookup
- Não representa o dado de negócio em si, mas sim um identificador interno para navegação e correlação de dados

## CNPJ como Valor

- Representa o CNPJ como informação de negócio para exibição, relatórios, integração externa ou regras de negócio
- Tratado como conteúdo informacional e não como referência estrutural para acesso a dados
- Pode ser exibido, enviado a outros sistemas ou validado segundo regras de negócio

## Classificação Acumulativa

A classificação não é excludente. Um mesmo campo pode atuar simultaneamente como Chave e como Valor, desde que haja evidências claras no código de que o campo cumpre ambos os papéis no fluxo de processamento. Essa avaliação deve considerar o contexto específico de uso no programa, o papel funcional desempenhado em cada trecho do código e a intenção semântica e técnica do campo no processamento.

---

# Regra de Classificação no Plano

Na etapa de Plano de Execução, o foco é exclusivamente operacional. A classificação de Chave ou Valor deve ser consumida do diagnóstico e não pode ser inferida ou redefinida pelo agente. O plano deve sinalizar qualquer dependência de classificação que não esteja claramente definida no diagnóstico, mas não deve tentar classificar campos por conta própria.

---

# Regra Mandatória de Explicitação de Consumo Subsequente

Sempre que o agente identificar que um campo de CNPJ está inserido em contexto de saída — especialmente em estruturas de detalhe, áreas de comunicação, buffers de output, arquivos de remessa ou retorno, arquivos de saída, estruturas de saída, segmentos de transmissão, interfaces entre programas ou qualquer outro artefato de propagação de dado de negócio — não é suficiente classificá-lo apenas como Valor de Negócio.

Nesses casos, o agente deve obrigatoriamente explicitar, de forma técnica e rastreável, quais são os consumidores subsequentes do campo após a saída do programa, da rotina, do módulo ou do artefato analisado.

A análise deve deixar claro:

- Quem consome o dado posteriormente, de forma nominal e técnica
- Qual artefato subsequente recebe o campo
- Qual é a finalidade do consumo
- Se o campo é apenas transportado ou efetivamente utilizado em lógica de negócio
- Quais impactos decorrem da mudança do CNPJ numérico para alfanumérico nesse encadeamento subsequente

O agente deve identificar, sempre que possível, os consumidores subsequentes dentro da cadeia de processamento, tais como:

- Programas COBOL chamadores ou chamados
- Steps seguintes de JOB/JCL
- Transações CICS
- Rotinas batch
- Arquivos físicos ou lógicos
- Copybooks compartilhados
- Tabelas DB2
- Mensagens MQ
- APIs
- Sistemas satélites
- Módulos de integração
- Rotinas de impressão, extrato, relatório, auditoria, conciliação ou retorno

Além de identificar os consumidores, o agente deve explicitar a natureza do consumo, informando se o campo é: exibido, transmitido, persistido, validado, transformado, replicado, conciliado, utilizado em regra de negócio, utilizado em interface externa ou apenas propagado como carga útil.

A conclusão não deve ser genérica. O agente não deve afirmar apenas que se trata de output, transmissão de dados de negócio ou valor de negócio. Deve detalhar o encadeamento técnico subsequente, deixando explícito:

- A origem do campo
- O destino do campo
- O artefato intermediário, se houver
- O contrato técnico em que ele trafega
- O impacto potencial da alteração de formato ao longo dessa cadeia

O agente deve utilizar como evidências técnicas, sempre que disponíveis: nome do campo, nome do registro, nome do layout, sufixos semânticos como `-DET`, `-OUT`, `-SAI`, `-RET`, instruções `MOVE`, gravação em arquivo, passagem por parâmetro, montagem de mensagem, envio para outro programa e uso em tela, relatório ou interface.

Em ambiente mainframe, a presença do CNPJ em estrutura de saída ou detalhe deve ser tratada como forte indicativo de participação em uma cadeia de consumo downstream. A análise do agente deve ser explícita, nominativa, técnica e rastreável.

Se uma tabela contiver esse tipo de informação, é obrigatório adicionar a coluna de Consumidor a ela para identificar quem consome a saída do dado.

---

# Regra Mandatória de Explicitação de Consumo Upstream e Downstream

Sempre que o agente identificar que um campo de CNPJ está inserido em contexto de saída — especialmente em estruturas de detalhe, áreas de comunicação, buffers de output, arquivos de remessa ou retorno, arquivos de saída, estruturas de saída, segmentos de transmissão, interfaces entre programas ou qualquer outro artefato de propagação de dado de negócio — não é suficiente classificá-lo apenas como Valor de Negócio.

De forma equivalente e simétrica, sempre que o agente identificar que um campo de CNPJ está inserido em contexto de entrada — especialmente em parâmetros recebidos via LINKAGE SECTION, arquivos de remessa ou retorno recebidos, interfaces de entrada, buffers de input, estruturas de recepção de dados ou qualquer artefato que produza ou alimente o campo analisado — não é suficiente identificar apenas o uso local do campo. É obrigatório rastrear de onde o dado provém, qual artefato o originou, por qual contrato técnico ele chegou e qual formato é assumido na entrada, pois a mudança para CNPJ alfanumérico pode quebrar premissas implícitas do produtor upstream.

---

## O que a análise deve deixar claro

### Consumo Downstream

A análise deve deixar claro:

- Quais artefatos consomem o campo após o programa analisado
- Por qual mecanismo o dado é transmitido (parâmetro, arquivo, buffer, segmento, interface)
- Qual é o contrato técnico de saída — tamanho, tipo, formato esperado pelo consumidor
- Se o consumidor downstream aplica validação, máscara, truncamento ou conversão sobre o campo recebido
- Qual é o impacto potencial da mudança de formato do CNPJ sobre cada consumidor identificado

### Consumo Upstream

A análise deve deixar claro:

- Quem produziu o dado anteriormente, de forma nominal e técnica
- Qual artefato anterior gerou ou transmitiu o campo
- Qual é a origem funcional do dado — se produzido internamente, recebido de sistema externo, lido de banco de dados ou passado por parâmetro
- Se o campo chega já transformado ou em formato original
- Quais premissas de formato o programa receptor assume sobre o dado de entrada e como a mudança para CNPJ alfanumérico impacta essas premissas

---

## Produtores e Consumidores a Identificar

### Produtores Upstream

O agente deve identificar, sempre que possível, os produtores upstream dentro da cadeia de processamento, tais como:

- Programas COBOL chamadores que passam o campo por parâmetro
- Steps anteriores de JOB/JCL que escrevem o arquivo lido
- Transações CICS que acionam o programa com dados preenchidos
- Rotinas batch anteriores que produzem o arquivo de entrada
- Arquivos físicos ou lógicos de onde o campo é lido
- Copybooks compartilhados que declaram o campo
- Tabelas DB2 de onde o campo é selecionado
- Mensagens MQ recebidas
- APIs que entregam o dado
- Sistemas satélites que alimentam o fluxo
- Módulos de integração de entrada
- Rotinas de carga, extrato, conciliação ou recebimento que produzem o campo

### Consumidores Downstream

O agente deve identificar, sempre que possível, os consumidores downstream dentro da cadeia de processamento, tais como:

- Programas COBOL chamados que recebem o campo por parâmetro
- Steps posteriores de JOB/JCL que leem o arquivo produzido
- Transações CICS acionadas com o campo preenchido
- Rotinas batch posteriores que consomem o arquivo de saída
- Arquivos físicos ou lógicos nos quais o campo é gravado
- Copybooks compartilhados que declaram o campo e são incluídos por múltiplos programas receptores
- Tabelas DB2 nas quais o campo é inserido ou atualizado
- Mensagens MQ enviadas pelo programa
- APIs que recebem o dado
- Sistemas satélites alimentados pelo fluxo
- Módulos de integração de saída
- Rotinas de carga, extrato, conciliação ou remessa que consomem o campo

---

## Natureza da Produção e do Consumo

### Natureza da Produção Upstream

Além de identificar os produtores, o agente deve explicitar a natureza da produção upstream, informando se o campo é:

- Gerado localmente pelo produtor
- Recebido de fonte externa e repassado sem transformação
- Transformado pelo produtor antes de ser entregue
- Lido de banco de dados
- Construído por composição de campos
- Recebido via parâmetro de chamada
- Lido de arquivo físico sem tratamento

### Natureza do Consumo Downstream

Além de identificar os consumidores, o agente deve explicitar a natureza do consumo downstream, informando se o campo é:

- Consumido diretamente sem transformação
- Transformado, mascarado ou convertido pelo consumidor
- Armazenado em banco de dados
- Transmitido a outro sistema sem alteração
- Utilizado como chave de busca ou comparação
- Exibido em tela ou relatório
- Incluído em arquivo de remessa ou retorno

---

## Conclusão

A conclusão não deve ser genérica. O agente não deve afirmar apenas que se trata de input, output, transmissão de dados de negócio ou valor de negócio. Deve detalhar o encadeamento técnico completo, upstream e downstream, deixando explícito:

- A origem upstream do campo — quem o produziu e como
- O uso local do campo no artefato analisado
- O destino downstream do campo — quem o consome e como
- O artefato intermediário, em qualquer direção, se houver
- O contrato técnico em que ele trafega em cada sentido
- O impacto potencial da alteração de formato ao longo de toda essa cadeia, em ambas as direções

---

## Instrução de Coluna para Tabelas

Se uma tabela contiver esse tipo de informação, é **obrigatório** adicionar as colunas abaixo para garantir rastreabilidade completa da cadeia:

| Coluna | Descrição |
|---|---|
| **Produtor** | Artefato que gerou ou transmitiu o dado upstream (nome nominal e técnico) |
| **Consumidor** | Artefato que consome o dado downstream (nome nominal e técnico) |

---

# Diretrizes Herdadas do PRD

## Princípios Estruturais

- Separação explícita entre identificador técnico e valor de negócio
- Relação determinística e idempotente entre Chave de CNPJ e Valor de CNPJ
- Uso interno exclusivo da Chave de CNPJ
- Uso externo e de negócio exclusivo do Valor de CNPJ

## Regras Mandatórias

- Validação de dígito verificador (DV) aplicada apenas ao Valor de CNPJ
- Formatação e máscara aplicadas apenas ao Valor de CNPJ
- Proibida exposição da Chave de CNPJ em interfaces externas
- Conversão obrigatoriamente centralizada no Serviço Central de Conversão

## Regras Novas do PRD
As regras abaixo já foram definidas nas especificações funcionais e técnicas, mas devem ser explicitamente reiteradas no plano de execução para garantir que sejam consideradas como bloqueios operacionais obrigatórios:
### RF09 — Preservação de identificadores externos (CGC ou Valor de CNPJ)

O requisito RF09 determina que todo identificador recebido de fonte externa, como uma tabela DB2, deve ser preservado exatamente no formato em que foi recebido, sem qualquer alteração de conteúdo, estrutura ou representação. Isso vale tanto para CGC quanto para Valor de CNPJ, independentemente de o valor ser numérico ou de haver possibilidade técnica de conversão. Em termos práticos, o sistema não pode converter, reclassificar, normalizar nem adaptar esses identificadores para formato alfanumérico, devendo apenas trafegar e persistir o dado de forma íntegra, respeitando rigorosamente sua origem externa.

### RF10 — Classificação com base em nível de inferência

O requisito RF10 estabelece que a classificação de um identificador como Chave de CNPJ ou Valor de CNPJ só pode ocorrer quando houver inferência determinística, isto é, quando o contexto técnico e funcional permitir uma decisão segura e inequívoca. Se a inferência for fraca, insuficiente ou ambígua, o campo deve ser obrigatoriamente classificado como Indefinido, e nessa condição nenhuma transformação pode ser executada, incluindo conversão, validação como CNPJ ou qualquer outro processamento derivado. Assim, o requisito impõe que baixa confiança semântica bloqueia ação automática e obriga a preservação integral do valor original.

### RT13 — Bloqueio de transformação para dados externos

A RT13 define que, uma vez identificado que um campo possui origem externa, o pipeline de transformação deve ser integralmente desviado, impedindo qualquer processamento que altere seu conteúdo ou formato. Isso significa que dados externos não podem ser convertidos para padrão alfanumérico, normalizados, enriquecidos ou sofrer qualquer modificação estrutural, devendo ser mantidos e persistidos exatamente como foram recebidos. Em essência, a regra funciona como um bloqueio técnico absoluto sobre transformações aplicadas a dados cuja origem esteja fora do escopo de controle do sistema.

### RT14 - Controle de inferência e estado "Indefinido":
A RT14 estabelece que, após a execução do processo de classificação, só é permitido transformar um campo de CNPJ quando sua inferência for determinística; caso contrário, ele deve ser obrigatoriamente classificado como **Indefinido**. Esse estado representa baixa confiança semântica e funciona como bloqueio técnico absoluto, impedindo qualquer modificação no campo, incluindo conversão, validação de dígito verificador, geração de Chave de CNPJ e até alteração estrutural de tipo, como trocar `PIC 9` por `PIC X`. Nessa situação, o campo deve ser preservado exatamente como está, sem qualquer intervenção automática, até que uma análise manual produza uma classificação confiável entre **Chave** e **Valor**.

---

# Estrutura Obrigatória do Plano de Execução

O agente deve produzir o documento `dem_h_plan.md` contendo obrigatoriamente as seguintes seções, organizadas por cabeçalhos markdown e sem numeração de seções no corpo do texto.

## Introdução

Parágrafo curto explicando o objetivo do plano de execução.

## Validação do Diagnóstico

Tabela com as seguintes colunas:

| Artefato | Campo | Tipo Detectado | Classificação | Confiança | Ação |

Legenda obrigatória:
- **Artefato:** origem do campo
- **Campo:** nome técnico
- **Tipo Detectado:** formato atual do campo
- **Classificação:** oriunda do diagnóstico, não inferida pelo agente
- **Confiança:** nível de inferência disponível
- **Ação:** decisão de execução decorrente da classificação

## Regras de Bloqueio Operacional

Esta seção deve declarar explicitamente as condições que bloqueiam a execução:

- Campo Indefinido não deve ser alterado
- Campo de origem externa não deve ser transformado
- A execução depende de classificação confiável oriunda do diagnóstico

---

## Regra de Conversão centralizada obrigatória entre Chave e Valor de CNPJ

Sempre que houver necessidade de transformar **Valor de CNPJ** em **Chave de CNPJ**, ou **Chave de CNPJ** em **Valor de CNPJ**, a conversão deverá ser realizada **exclusivamente** por meio da invocação do **Serviço Central de Conversão (DFE)**, sendo **vedada** qualquer tentativa de geração, derivação, manutenção ou persistência local da correlação entre esses identificadores. Essa regra garante que a correspondência Chave-Valor permaneça **determinística, única, auditável e aderente à arquitetura corporativa**, proibindo o uso de tabelas auxiliares, algoritmos internos, caches de correlação com autoria local ou qualquer lógica distribuída que reproduza a função do serviço central.

---

## Análise Consolidada de Impacto para Execução

Seção obrigatória. Se já existir conteúdo semelhante nos artefatos de entrada, realizar merge sem duplicação. Esta seção deve:

- Consolidar impactos oriundos dos artefatos de diagnóstico e impacto
- Qualificar cada impacto como: baixo, médio, alto ou crítico
- Quantificar quando possível
- Traduzir cada impacto em ação de execução concreta

Deve conter as seguintes subseções:

- Impacto Estrutural
- Impacto Técnico
- Impacto Funcional
- Impacto de Dependência
- Impacto de Dados
- Impacto de Integração
- Impacto de Risco
- Conclusão obrigatória da análise

## Análise de Dependência

Com base no diagnóstico, construir o grafo de dependências entre artefatos e calcular o grau de dependência de cada nó.

---

## Estratégias de Execução

Conceito exclusivo do PRD_PLAN — inexistente no PRD_DIAGNOSTICO. O Diagnostic não contempla esse nível de tomada de decisão: ele apenas produz a evidência técnica. É responsabilidade do Plan apresentar os cenários possíveis de execução, com detalhamento funcional e técnico suficiente para que a decisão seja tomada pelo cliente — nunca pelo agente.

---

### Possíveis Cenários

#### Cenário 1 — Complexidade (Decisão Técnica)

A escolha técnica se baseia em **esforço e risco crescente**, seguindo uma abordagem análoga ao modelo *Canary Release*: parte-se sempre pelo caminho de menor perturbação ao sistema, e avança-se em complexidade apenas quando o passo anterior estiver estabilizado e validado.

##### Princípio da Progressão por Complexidade Crescente

A progressão não é arbitrária — ela é guiada pelas evidências do Diagnóstico. Artefatos com classificação determinística, baixo acoplamento e cadeia upstream/downstream curta são os candidatos naturais ao primeiro passo. Artefatos com alta taxa de inferência, múltiplos consumidores downstream, dependências cruzadas entre módulos ou interação com sistemas satélites são deixados para etapas posteriores, quando o padrão de correção já estiver consolidado e testado.

##### Progressão Típica de Complexidade

**Passo 1 — Artefatos isolados, campo único, sem consumidores downstream críticos**

O ponto de entrada ideal. Trata-se de artefatos que manipulam o campo CNPJ de forma local, sem propagar o dado para outros módulos ou sistemas. A correção é contida: ajusta-se a Picture Clause, a validação ou a máscara, e o impacto se encerra no próprio artefato. O risco de regressão é mínimo e a reversão, se necessária, é trivial.

**Passo 2 — Artefatos com consumidores downstream identificados e rastreáveis**

Introduz a necessidade de coordenação entre o artefato corrigido e seus consumidores subsequentes. A correção no produtor pode quebrar contratos implícitos com quem recebe o campo — especialmente se o consumidor ainda assumir formato numérico. Este passo exige que o agente aplique as correções de forma sequencial e validada: primeiro o produtor, depois o consumidor, verificando a integridade do contrato técnico entre eles.

**Passo 3 — Copybooks compartilhados e estruturas de dados comuns**

Alterações em copybooks afetam todos os programas que os incluem (`COPY`). Um ajuste em `PIC 9(14)` para `PIC X(14)` em um copybook compartilhado pode impactar dezenas de artefatos simultaneamente. Este passo exige mapeamento completo de dependências antes de qualquer modificação, e a execução deve ser acompanhada de validação em todos os artefatos impactados.

**Passo 4 — Fluxos batch com múltiplos steps de JCL**

Fluxos batch encadeados introduzem dependências temporais e de ordem: um step posterior consome o arquivo produzido pelo step anterior. A correção deve respeitar a sequência do JCL, garantindo que o formato do campo seja consistente em todos os steps da cadeia — da leitura inicial à gravação final. Falhas de formato neste nível podem causar abend silencioso ou rejeição de registros em lote.

**Passo 5 — Interfaces com sistemas externos, módulos secundários e satélites**

O nível de maior risco e menor controle. Sistemas externos têm contratos estabelecidos, SLAs e proprietários distintos. Alterações aqui exigem alinhamento com outros times, validação de contrato de interface e, frequentemente, execução coordenada entre múltiplos sistemas. Este passo só deve ser executado após todos os anteriores estarem estabilizados e com o padrão de correção completamente validado internamente.

##### Vantagens desta Abordagem

- Permite validação incremental a cada passo, reduzindo a janela de risco
- Facilita reversão isolada sem comprometer etapas já estabilizadas
- Gera aprendizado técnico acumulativo: os padrões de correção identificados nos primeiros passos informam e aceleram os seguintes
- Mantém o sistema operacional durante toda a execução, sem necessidade de congelamento total

##### Riscos e Limitações

- Requer disciplina de sequenciamento: executar passos fora de ordem pode introduzir inconsistências temporárias de formato entre produtor e consumidor
- O tempo total de execução é maior do que uma intervenção global simultânea
- Dependências não mapeadas no Diagnóstico podem emergir durante a execução e forçar revisão do sequenciamento

---

#### Cenário 2 — Estratégico (Decisão Executiva)

A escolha estratégica não é primariamente técnica — ela é orientada por **fatores operacionais, negociais, sazonais ou de mercado**. O driver da decisão aqui é o resultado de negócio, e não o esforço ou o risco técnico.

##### Exemplos de Drivers Estratégicos

**Sazonalidade operacional:** determinados períodos do ano concentram volumes críticos de transações (fechamento fiscal, processamento de folha, conciliação de remessas). A estratégia pode determinar que a execução ocorra fora desses períodos — mesmo que tecnicamente seja possível executar durante eles — para preservar a estabilidade do ambiente em janelas de alta criticidade.

**Prazo regulatório:** a Receita Federal pode estabelecer uma data-limite para suporte ao CNPJ alfanumérico. Nesse caso, o critério estratégico é o compliance regulatório: a execução deve ser concluída antes do prazo, independentemente do sequenciamento técnico ideal.

**Priorização por impacto de negócio:** artefatos que sustentam processos de maior receita, maior visibilidade ou maior risco regulatório podem ser priorizados estrategicamente, mesmo que tecnicamente sejam mais complexos. A decisão aqui é: qual falha, se ocorrer, causa maior dano ao negócio?

**Coordenação com outros projetos:** a execução pode ser acelerada, postergada ou reordenada para coincidir — ou evitar conflito — com outras iniciativas em andamento no mesmo ambiente (migrações, atualizações de plataforma, janelas de manutenção programadas).

**Custo de execução vs. janela de oportunidade:** em alguns contextos, a disponibilidade de equipe técnica, ambiente de homologação ou janela de deploy é limitada. A estratégia pode determinar a concentração de esforço em uma janela específica, mesmo que isso implique maior risco técnico pontual.

##### Características desta Abordagem

- A decisão é tomada por stakeholders executivos ou de negócio, com suporte técnico informativo
- O agente não escolhe esta estratégia — ele a descreve com fidelidade suficiente para que o cliente possa avaliá-la
- Pode resultar em sequenciamento que diverge do ótimo técnico, o que deve ser documentado como risco aceito

---

### Cenário Escolhido

Conforme definido nas premissas desta PoC, a execução seguirá o **Cenário 1 — Complexidade (Decisão Técnica)**, adotando a abordagem de menor esforço e menor risco como ponto de partida.

A progressão será iniciada pelos artefatos de menor acoplamento e cadeia de impacto mais curta, conforme evidenciado pelo Diagnóstico. O avanço para etapas de maior complexidade será condicionado à validação das etapas anteriores. Nenhuma etapa será antecipada por conveniência operacional ou pressão de prazo sem reclassificação formal do cenário escolhido.


---

## Ordem de Execução

Organizada nas seguintes fases:

- Preparação
- Execução conforme diagnóstico
- Integrações
- Consolidação

## Execução Detalhada

Camada operacional obrigatória. Deve:

- Referenciar os artefatos impactados nominalmente
- Descrever os tipos de alteração previstos
- Apresentar padrões de ajuste
- Não citar linhas de código

## Auditoria de Operações Incompatíveis

Identificar operações que se tornam incompatíveis com o formato alfanumérico:

- Operações numéricas sobre campos CNPJ
- Uso de `COMP-3` em campos que serão convertidos para `PIC X`
- Comparações numéricas que devem ser revistas
- Descrever a ação esperada para cada caso

## Riscos Técnicos de Conversão de Tipo

Descrever os riscos da conversão `PIC 9` para `PIC X`, incluindo:

- Uso de `IF EQUAL ZEROS` sobre campos alfanuméricos
- Operações de `MOVE` numérico para campo alfanumérico
- Demais riscos identificados no diagnóstico
- Mitigação prevista para cada risco

## Estratégia de Dados

- Estratégia de convivência entre formatos durante a transição
- Estratégia de sincronização entre ambientes

## Estratégia de Transição

Organizada nas seguintes fases:

- Introdução
- Adoção
- Domínio
- Futuro

## Estratégia de Lookup

- Estratégia de uso do Serviço Central de Conversão
- Estratégia de cache (Redis, VSAM, batch ou equivalente mainframe)
- Estratégia de denormalização quando aplicável

## Consulta por Radical

Diretriz futura. Não executar neste plano.

## Contingência Técnica

- Procedimento para falha do Serviço Central de Conversão
- Estratégia de fallback
- Garantia de continuidade do processamento batch

## Plano Alternativo para Integrações

Caso sistema externo não evolua para suportar o CNPJ alfanumérico, apresentar estratégia de adaptação para o sistema interno.

## Monitoramento

Identificar a ferramenta de monitoramento aplicável ao ambiente e as métricas de observabilidade necessárias.

## Deploy

Identificar a ferramenta de deploy aplicável ao ambiente e o procedimento de aplicação do patch.

## Riscos

Tabela com legenda obrigatória. Deve conter no mínimo as colunas: Risco, Probabilidade, Impacto, Nível e Mitigação.

## Rollback

- Estratégia de versionamento dos artefatos
- Procedimento de reversão em caso de falha

## Go-Live

- Critérios de prontidão para go-live
- Plano de testes
- Plano de monitoramento pós-implantação

## Critérios de Sucesso

- Estabilidade operacional confirmada
- Ausência de regressão em funcionalidades existentes

## Definition of Done

Condições que encerram formalmente a execução do plano.

## Conclusão

Seção obrigatória. Síntese do plano, decisões pendentes e próximos passos.

---

# Grafos de Estratégia

O agente deve gerar os seguintes grafos via NetworkX:

- `b_dem_a_grafo_estrategia_1.png` — grafo correspondente à Estratégia 1
- `b_dem_b_grafo_estrategia_2.png` — grafo correspondente à Estratégia 2

Os grafos devem ser salvos no mesmo diretório do artefato de saída.

---

# Regras Críticas de Produção

- Proibido o uso de emojis
- Proibido classificar campos como Chave ou Valor sem base no diagnóstico
- Proibido repetir ou transcrever o conteúdo dos artefatos de diagnóstico
- Revisão ortográfica obrigatória antes da entrega do documento

---

# Requisitos Funcionais da Solução

Os requisitos a seguir devem ser referenciados no plano como fundamento das decisões de execução. O agente não deve reescrevê-los, mas deve consumi-los como base de rastreabilidade.

**RF01:** Suporte à coexistência de formatos numérico e alfanumérico do Valor de CNPJ.

**RF02:** Separação obrigatória entre Chave de CNPJ e Valor de CNPJ em todos os artefatos.

**RF03:** Preservação dos fluxos de negócio existentes. Apenas o tratamento do identificador é alterado.

**RF04:** Adaptação obrigatória de todos os pontos de uso identificados no diagnóstico.

**RF05:** Tratamento distinto em entradas e saídas conforme o tipo do campo.

**RF06:** Uso exclusivo do Valor de CNPJ em interfaces externas.

**RF07:** Conversão obrigatória via Serviço Central de Conversão.

**RF08:** Determinismo e idempotência das conversões.

**RF09:** Preservação de identificadores recebidos de fontes externas.

**RF10:** Classificação com base em inferência determinística. Inferência fraca resulta em estado Indefinido.

---

# Requisitos Técnicos da Solução

Os requisitos a seguir devem ser referenciados no plano como fundamento das decisões de execução.

**RT01:** Redefinição de estruturas físicas para separar Chave de CNPJ e Valor de CNPJ com tipagem distinta.

**RT02:** Revisão de contratos de dados em layouts e copybooks.

**RT03:** Aplicação de DV apenas no Valor de CNPJ.

**RT04:** Implementação compatível com ambiente mainframe, incluindo tabela de conversão controlada.

**RT05:** Validação e formatação diferenciadas por tipo de campo.

**RT06:** Revisão de indexação e busca para respeitar a semântica do campo.

**RT07:** Compatibilidade entre processamento batch e online.

**RT08:** Rastreamento e auditabilidade da correlação 1:1.

**RT09:** Centralização da geração de Chave de CNPJ no sistema central autorizado.

**RT10:** Estratégia de cache obrigatória para redução de acessos ao sistema central.

**RT11:** Distribuição batch de dados para ambientes mainframe dependentes.

**RT12:** Proibição de exposição da Chave de CNPJ em saídas para usuário ou sistema externo.

**RT13:** Bloqueio de transformação para dados de origem externa.

**RT14:** Controle de inferência com estado Indefinido explícito, bloqueando qualquer transformação.

---

# Regras Mandatórias de Execução para o Agente

## Separação entre Diagnóstico e Plano de Execução

A execução do patch deve observar obrigatoriamente a separação entre:

- **Diagnóstico:** define como a correção deve ser implementada no código
- **Plano de Execução:** define em quais artefatos e códigos a correção deve ser aplicada ou preservada sem alteração

Essa separação não deve ser confundida, misturada, fundida ou reinterpretada durante a execução.

## Natureza da Intervenção no Código

Esta demanda não caracteriza modificação ampla, reestruturação integral, refatoração global ou modernização irrestrita de código.

A execução deve ser tratada como intervenção pontual, limitada ao atendimento objetivo da demanda. Portanto:

- Não é necessário alterar integralmente os artefatos
- As modificações devem ser pontuais
- O agente deve alterar apenas o mínimo necessário para viabilizar a adequação requerida
- A execução não deve promover mudanças estruturais, estéticas, organizacionais ou de padronização que não sejam estritamente necessárias para o patch solicitado

Qualquer alteração que extrapole a intervenção mínima necessária deve ser considerada indevida.

## Regra Mandatória do Diagnóstico

O Diagnóstico deve descrever de forma explícita, objetiva e tecnicamente verificável: a estratégia técnica de correção, a lógica do patch a ser aplicado, o critério de alteração do código, a forma exata de implementação da modificação e as restrições técnicas que devem ser respeitadas.

A execução deve seguir estritamente o Diagnóstico no que se refere a como fazer o patch. O agente não deve:

- Adotar estratégia alternativa à definida no Diagnóstico
- Introduzir refatoração não solicitada
- Promover otimização paralela
- Alterar abordagem técnica sem previsão explícita
- Ampliar o escopo técnico da correção
- Implementar solução tecnicamente válida, porém diferente da explicitamente definida

## Regra Mandatória do Plano de Execução

O Plano de Execução deve identificar de forma explícita, objetiva e inequívoca: quais artefatos devem ser alterados, quais devem ser preservados sem alteração, e o conjunto exato de atuação autorizado para a aplicação do patch.

A execução deve seguir estritamente o Plano de Execução no que se refere a em quais artefatos fazer o patch. O agente não deve:

- Modificar artefato não previsto no Plano de Execução
- Incluir arquivos, programas, copybooks, módulos, tabelas, layouts ou componentes fora do escopo autorizado
- Reinterpretar o plano para expandir a abrangência da correção
- Alterar artefatos explicitamente marcados para preservação

## Restrição ao Escopo Funcional e Técnico do CNPJ

A execução deve restringir todas as modificações ao escopo funcional e técnico relacionado ao CNPJ. O agente não deve:

- Alterar trechos não relacionados ao tratamento do CNPJ
- Expandir a intervenção para outros dados, rotinas, layouts ou regras de negócio não vinculados ao CNPJ
- Realizar correções paralelas por conveniência técnica
- Aproveitar a execução para promover ajustes não demandados

Toda modificação deve possuir vínculo direto, explícito e justificável com a necessidade de adequação do CNPJ no contexto da demanda.

## Preservação de Origem Externa

O agente não deve alterar campos de CNPJ ou CGC originados de fontes externas, mesmo quando presentes em estruturas locais, layouts, copybooks, cópias, interfaces ou áreas de comunicação.

A origem externa é, por si só, critério suficiente para exclusão do escopo de modificação, independentemente do nome do campo, do formato atual, do conteúdo armazenado, da similaridade com campos internos ou da possibilidade técnica de alteração.

Exemplos típicos de fonte externa incluem: tabelas DB2 de outros sistemas, contratos de interface externa, arquivos recebidos de sistemas terceiros e estruturas cujo domínio de manutenção não pertença ao sistema em intervenção.

A conduta obrigatória é preservar o campo intacto, salvo determinação explícita em contrário no escopo formal da demanda.

## Conservadorismo na Inferência

Na dúvida, o agente não deve alterar.

Quando a taxa de inferência ou o grau de confiança para classificar um campo como Chave, Valor ou Fonte Externa for insuficiente, fraco, ambíguo ou inconclusivo, a conduta obrigatória é classificar o item como Indefinido e preservá-lo integralmente, sem alteração.

A incerteza nunca autoriza modificação. A ausência de evidência suficiente para alterar deve ser tratada como evidência suficiente para preservar.

## Regra Geral de Conformidade

A execução deve respeitar simultaneamente:

- O escopo técnico definido no Diagnóstico
- O escopo material definido no Plano de Execução
- O escopo funcional restrito ao tratamento do CNPJ, sem extrapolação para elementos adjacentes

## Vedação de Extrapolação

É vedado ao agente:

- Extrapolar o modo de correção definido no Diagnóstico
- Extrapolar o conjunto de artefatos definido no Plano de Execução
- Alterar artefatos preservados
- Modificar elementos de origem externa
- Realizar alterações implícitas, inferidas sem evidência ou não autorizadas
- Promover mudanças adicionais sob justificativa de melhoria técnica, padronização, estética, organização, refatoração ou conveniência operacional
- Ampliar o escopo da demanda para além do necessário à adequação pontual do CNPJ

A execução deve se limitar estritamente ao patch solicitado, na forma solicitada, somente nos artefatos autorizados e apenas no escopo relacionado ao CNPJ.

## Fórmula de Interpretação Obrigatória

- **Diagnóstico** = como fazer o patch
- **Plano de Execução** = em quais artefatos fazer o patch
- **Escopo da Demanda** = o que pode ou não pode ser alterado

## Regra Final de Execução

O agente deve adotar postura estritamente restritiva e conservadora durante toda a execução.

Somente o que estiver explicitamente autorizado pode ser alterado. Somente a forma explicitamente definida pode ser utilizada. Todo o restante deve ser preservado.

Na ausência de autorização explícita, a decisão correta é não alterar.