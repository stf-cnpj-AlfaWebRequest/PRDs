# **Especificação Técnica — Diagnostic**

## Definições

Chave: É uma chave (primária ou estrangeira) do CNPJ, e não do conteúdo do CNPJ, onde o campo de chave estrangeira em um artefato de transação, ou campo de chave primária em um artefato de consulta, etc. (ex: 78465270000165, 00000000000001). A \*\*Chave de CNPJ\*\* é um identificador técnico interno e não deve ser tratada como documento oficial de negócio.

Valor: É o conteúdo do CNPJ, ou seja, o campo é manipulado como valor de negócio, onde o campo é utilizado em regras de negócio, cálculos, validações, tela (GUI) etc. (ex: 78465270000165, 6QNJ8VY2JIC341). O \*\*Valor de CNPJ\*\* é o identificador oficial utilizado pelo negócio e por interfaces externas.

Módulos: Refere-se a qualquer artefato de código, incluindo programas COBOL e copybooks (artefatos .CPY) que são utilizados pelos programas COBOL. (ex: RF, DAF, VIP, HLP, etc.)

Módulo Principal: É o nome dado a um módulo que pertence ao mesmo sistema dos programas principais. Módulos principais podem ou não ser alterados a depender do plano de execução e execução propriamente dita, mas pertencem ao mesmo time/sistema. A identificação de módulos principais é feita dinamicamente com base no prefixo do nome do artefato. (ex: se os programas principais são VIPP4365 e VIPP4553, então qualquer artefato com prefixo VIP é considerado principal, mesmo que não requeira alteração).

Módulo Secundário: A definição do módulo secundário é tão importante que há uma definição prática e consolidada para ele, que deve ser seguida rigorosamente. O módulo secundário é o nome dado a um módulo que pertence a um sistema diferente dos programas principais. Módulos secundários não podem ser alterados, pois pertencem a outros times/sistemas. A identificação de módulos secundários é feita dinamicamente com base no prefixo do nome do artefato. (ex: se os programas principais são VIPP4365 e VIPP4553, então artefatos com prefixo DAF ou HLP são considerados secundários).

Artefato: Artefato é qualquer unidade técnica versionável que compõe, suporta ou influencia a execução do sistema. Isso inclui programas (ex: .cbl), copybooks (.cpy), JCLs, artefatos de dados, layouts, tabelas e demais elementos estruturais. Um artefato pode representar lógica de negócio, estrutura de dados ou configuração operacional, sendo tratado como elemento rastreável, analisável e passível de impacto dentro do ecossistema.

Programas: Refere-se ao código COBOL em si, ou seja, aos artefatos .CBL, .CPY, entre outros. Esses artefatos são os artefatos que contêm a lógica de negócio e que serão modificados (patch). No cliente, seus nomes seguem o padrão de um prefixo de 3 letras maiúsculas seguido de um número, por exemplo: VIPP4365.cbl, DAF1234.cbl, HLP5678.cbl.

Correlação 1:1: Relação obrigatória e inequívoca entre uma \*\*Chave de CNPJ\*\* e um \*\*Valor de CNPJ\*\*, preservando rastreabilidade, integridade relacional, auditabilidade e determinismo da conversão.

CGC: Denominação anterior do CNPJ (Cadastro Geral de Contribuintes), equivalente ao CNPJ para todos os efeitos desta demanda. Campos nomeados como CGC devem ser tratados com as mesmas regras aplicáveis ao CNPJ.

PF: Pessoa Física, ou seja, indivíduo, pessoa natural. Campos relacionados a PF não estão no escopo desta demanda, mas podem ser utilizados para comparação e inferência quando necessário.

PJ: Pessoa Jurídica, ou seja, empresa, organização. Campos relacionados a PJ estão no escopo desta demanda, pois o CNPJ é o identificador oficial de pessoas jurídicas.

Fonte Externa: Origem de dados pertencente a um sistema externo ao escopo desta modificação, como tabelas DB2 de outros domínios (ex: DB2MCI.CLIENTE), artefatos legados de terceiros ou integrações de entrada. Dados provenientes de fontes externas não estão no escopo de modificação desta demanda e não devem ser alterados.

**Campo Indefinido:** Campo de CNPJ/CGC cuja classificação semântica entre **Chave** e **Valor** não possa ser determinada de forma objetiva, rastreável e suficientemente confiável com base nos artefatos de **discovery**, **diagnóstico**, análise de código, contratos de interface, layouts, regras de negócio e contexto de processamento.

    Um campo deve ser classificado como **Indefinido** quando os elementos analisados não fornecerem evidência técnica suficiente para sustentar, com segurança, uma decisão inequívoca sobre sua natureza funcional e estrutural no fluxo. Nessa condição, o campo **não é elegível para alteração**, devendo ser **preservado em seu estado atual** até que evidências adicionais permitam sua reclassificação formal.

    A inferência será considerada **fraca** sempre que houver qualquer grau de incerteza residual na classificação, isto é, quando o nível de confiança da análise for inferior a 100% e a incerteza apurada for superior a 0%.

Serviço Central de Conversão: Serviço oficial responsável por converter \*\*Chave de CNPJ\*\* em \*\*Valor de CNPJ\*\* e \*\*Valor de CNPJ\*\* em \*\*Chave de CNPJ\*\*. É proibida a geração local ou distribuída dessa correlação fora do sistema central autorizado.

Base Central de Correlação (DFE): Estrutura central responsável por manter a relação 1:1 entre \*\*Chave de CNPJ\*\* e \*\*Valor de CNPJ\*\*, bem como metadados de controle e auditoria.

Cache: Camadas auxiliares de desempenho utilizadas para reduzir acessos à base central de correlação, podendo incluir mecanismos equivalentes a Redis, VSAM, processamento batch ou estruturas legadas compatíveis com o ambiente.

Claro — abaixo está a **seção de definição** já com o **merge** das suas definições, preservando o conceito original que eu havia estruturado, mas incorporando explicitamente os pontos de **sistema diferente**, **outro time/domínio**, **identificação dinâmica por prefixo** e o fato de que a possibilidade de alteração depende do **plano de execução** e da **execução da demanda**.

---
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
---

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

### Definição de Módulo Secundário (Externo)**

**Módulo Secundário (Externo)** é todo artefato técnico que **pertence a sistema, domínio funcional, contexto operacional ou responsabilidade de time diferente** em relação aos programas principais da demanda, ainda que mantenha relação funcional, estrutural, operacional ou contratual com o fluxo analisado.

No contexto desta documentação, o termo **externo** não significa necessariamente fora da aplicação em sentido físico, fora do repositório ou fora da plataforma mainframe. Significa, tecnicamente, que o artefato está **fora do núcleo principal da implementação**, pertencendo a **outro módulo lógico**, **outro agrupamento funcional** ou **outro contexto sistêmico**, podendo inclusive estar sob responsabilidade de **outro time**.

Um módulo secundário é, portanto, um artefato que **não compõe o conjunto principal de programas da demanda**, mas que pode participar do processamento por meio de chamadas, integrações, compartilhamento de estruturas, consumo de arquivos, uso de copybooks, trocas de dados ou dependências de execução.

#### **Identificação Dinâmica**

A identificação de um módulo secundário deve ser feita de forma **dinâmica**, com base na análise técnica do artefato e, em especial, na **convenção de nomenclatura e no prefixo do nome do artefato**.

Essa identificação não deve ser tratada como cadastro fixo ou lista estática pré-definida. A LLM deve inferir a classificação comparando o prefixo do artefato analisado com o prefixo dos programas principais da demanda.

Exemplo:

* se os programas principais da demanda forem **VIPP4365** e **VIPP4553**, então o prefixo principal do escopo é **VIP**;
* nesse cenário, artefatos com prefixos como **DAF** ou **HLP** devem ser tratados, em regra, como **módulos secundários**, por pertencerem a outro módulo, outro sistema ou outro domínio em relação ao núcleo principal.

Assim, artefatos com prefixo **DAF** ou **HLP** são secundários em relação a programas **VIP**, salvo quando a documentação da demanda dispuser explicitamente em sentido diverso.

#### **Relação com os Programas Principais**

Os programas principais são os artefatos diretamente vinculados ao escopo central da implementação. Já o módulo secundário externo é o artefato que, embora tecnicamente relacionado ao fluxo, **não representa o ponto principal da alteração solicitada**.

Ele pode:

* ser chamado pelo programa principal;
* chamar o programa principal;
* compartilhar copybooks, layouts ou áreas de comunicação;
* trocar dados com o fluxo principal;
* manipular campos correlatos ao CNPJ;
* participar de validações, integrações, consultas, persistência, formatação, enriquecimento ou apoio operacional.

Ainda assim, sua participação no fluxo **não o transforma automaticamente em módulo principal**.

#### **Pertencimento a Outro Sistema, Time ou Domínio**

Um dos critérios centrais para caracterização do módulo secundário é o seu pertencimento a **outro sistema**, **outro domínio funcional** ou **outro contexto de manutenção**, frequentemente sob responsabilidade de **outro time**.

Por essa razão, módulos secundários exigem tratamento mais controlado sob a ótica de governança, pois uma alteração nesses artefatos pode extrapolar o escopo original da demanda e produzir impacto em fluxos paralelos, contratos técnicos compartilhados ou responsabilidades sistêmicas de terceiros.

#### **Possibilidade de Alteração**

Módulos secundários **não podem ser alterados**, esta impossibilidade **não decorre automaticamente de sua participação no fluxo**, e também não da existência de manipulação de CNPJ, nem da relevância técnica do artefato.

A autorização para alteração depende de forma cumulativa de:

* previsão no **plano de execução**;
* autorização no **escopo formal da demanda**;
* direcionamento explícito na **execução propriamente dita**;
* compatibilidade com a governança de alteração entre módulos, sistemas e times.

Na ausência desses elementos, o módulo secundário deve ser tratado como **artefato analisável e registrável**, mas **não elegível para modificação**.

#### **Características Técnicas Comuns**

Um módulo secundário externo normalmente apresenta uma ou mais das seguintes características:

* pertence a prefixo diferente do prefixo dos programas principais;
* pertence a outro sistema, outro time ou outro domínio funcional;
* atua como suporte, integração, utilitário, consulta, persistência ou serviço compartilhado;
* não concentra a regra principal da demanda;
* participa do fluxo apenas como dependência técnica ou funcional;
* pode manipular CNPJ, mas sem ser o núcleo da implementação;
* está fora da lista principal de artefatos-alvo da alteração.

#### **Relação com a Manipulação de CNPJ**

A presença de CNPJ em um módulo secundário não altera, por si só, sua classificação.

Um módulo secundário pode:

* receber CNPJ como entrada;
* retornar CNPJ como saída;
* transportar CNPJ em registros;
* compartilhar estruturas com campos de CNPJ;
* usar CNPJ como **Chave**;
* usar CNPJ como **Valor**.

Mesmo nesses casos, a classificação como módulo secundário permanece válida se o artefato continuar pertencendo a **outro sistema/domínio/time** e não estiver enquadrado como programa principal da demanda.

#### **Exemplos Típicos**

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

#### **Definição Consolidada**

Em termos consolidados, **Módulo Secundário (Externo)** é o nome dado ao artefato que:

* pertence a sistema, time ou domínio diferente dos programas principais;
* é identificado dinamicamente, principalmente pelo prefixo do nome do artefato e pela análise contextual;
* pode manter relação técnica com o fluxo principal;
* pode ou não ser alterado, dependendo do plano de execução e da autorização formal da demanda;
* não deve ser tratado automaticamente como parte do escopo principal apenas por participar do processamento.



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
### Regra Mandatória de Explicitação de Consumo Upstream e Downstream

Sempre que a LLM identificar que um campo de CNPJ está inserido em contexto de saída, especialmente em estruturas de detalhe, áreas de comunicação, buffers de output, arquivos de remessa ou retorno, arquivos de saída, estrutura de saída, segmentos de transmissão, interfaces entre programas ou qualquer outro artefato de propagação de dado de negócio, não é suficiente classificá-lo apenas como Valor de Negócio.

De forma equivalente e simétrica, sempre que a LLM identificar que um campo de CNPJ está inserido em contexto de entrada, especialmente em parâmetros recebidos via LINKAGE SECTION, arquivos de remessa ou retorno recebidos, interfaces de entrada, buffers de input, estruturas de recepção de dados ou qualquer artefato que produza ou alimente o campo analisado, não é suficiente identificar apenas o uso local do campo. É obrigatório rastrear de onde o dado provém, qual artefato o originou, por qual contrato técnico ele chegou e qual é o formato esperado na entrada.

---

#### O que a análise deve deixar claro

##### Consumo Downstream

A análise deve deixar claro:

- Quais artefatos consomem o campo após o programa analisado
- Por qual mecanismo o dado é transmitido (parâmetro, arquivo, buffer, segmento, interface)
- Qual é o contrato técnico de saída — tamanho, tipo, formato esperado pelo consumidor
- Se o consumidor downstream aplica validação, máscara, truncamento ou conversão sobre o campo recebido
- Qual é o impacto potencial da mudança de formato do CNPJ sobre cada consumidor identificado

#### Consumo Upstream

A análise deve deixar claro:

- Quem produziu o dado anteriormente, de forma nominal e técnica
- Qual artefato anterior gerou ou transmitiu o campo
- Qual é a origem funcional do dado — se é produzido internamente, recebido de sistema externo, lido de banco de dados ou passado por parâmetro de chamada
- Se o campo chega já transformado ou em formato original
- Quais premissas de formato o programa receptor assume sobre o dado de entrada e como a mudança para CNPJ alfanumérico impacta essas premissas

---

#### Produtores e Consumidores a Identificar

##### Produtores Upstream

A LLM deve identificar, sempre que possível, os produtores upstream dentro da cadeia de processamento, tais como:

- Programas COBOL chamadores que passam o campo por parâmetro
- Steps anteriores de JOB/JCL que escrevem o arquivo lido pelo programa
- Transações CICS que acionam o programa com dados preenchidos
- Rotinas batch anteriores que produzem o arquivo de entrada
- Arquivos físicos ou lógicos de onde o campo é lido
- Copybooks compartilhados que declaram o campo e são incluídos por múltiplos programas
- Tabelas DB2 de onde o campo é selecionado via EXEC SQL
- Mensagens MQ recebidas pelo programa
- APIs que entregam o dado ao programa
- Sistemas satélites que alimentam o fluxo
- Módulos de integração de entrada
- Rotinas de carga, extrato, conciliação ou recebimento que produzem o campo

##### Consumidores Downstream

A LLM deve identificar, sempre que possível, os consumidores downstream dentro da cadeia de processamento, tais como:

- Programas COBOL chamados que recebem o campo por parâmetro
- Steps posteriores de JOB/JCL que leem o arquivo produzido pelo programa
- Transações CICS acionadas com o campo preenchido
- Rotinas batch posteriores que consomem o arquivo de saída
- Arquivos físicos ou lógicos nos quais o campo é gravado
- Copybooks compartilhados que declaram o campo e são incluídos por múltiplos programas receptores
- Tabelas DB2 nas quais o campo é inserido ou atualizado via EXEC SQL
- Mensagens MQ enviadas pelo programa
- APIs que recebem o dado do programa
- Sistemas satélites alimentados pelo fluxo
- Módulos de integração de saída
- Rotinas de carga, extrato, conciliação ou remessa que consomem o campo

---

#### Natureza da Produção e do Consumo

##### Natureza da Produção Upstream

Além de identificar os produtores upstream, a LLM deve explicitar a natureza da produção, informando se o campo upstream é:

- Gerado localmente pelo produtor
- Recebido de fonte externa pelo produtor e repassado sem transformação
- Transformado pelo produtor antes de ser entregue
- Lido de banco de dados
- Construído por composição de campos
- Recebido via parâmetro de chamada
- Lido de arquivo físico sem tratamento

##### Natureza do Consumo Downstream

Além de identificar os consumidores downstream, a LLM deve explicitar a natureza do consumo, informando se o campo downstream é:

- Consumido diretamente sem transformação
- Transformado, mascarado ou convertido pelo consumidor
- Armazenado em banco de dados
- Transmitido a outro sistema sem alteração
- Utilizado como chave de busca ou comparação
- Exibido em tela ou relatório
- Incluído em arquivo de remessa ou retorno

---

#### Conclusão

A conclusão não deve ser genérica. A LLM não deve afirmar apenas que se trata de "input", "output", "transmissão de dados de negócio" ou "valor de negócio". Ela deve detalhar o encadeamento técnico completo, upstream e downstream, deixando explícito:

- A origem upstream do campo — quem o produziu e como
- O uso local do campo no artefato analisado
- O destino downstream do campo — quem o consome e como
- O artefato intermediário, em qualquer direção, se houver
- O contrato técnico em que ele trafega em cada sentido
- O impacto potencial da alteração de formato ao longo de toda essa cadeia, em ambas as direções

---

##### Instrução de Coluna para Tabelas

Se uma tabela tiver esse tipo de informação, **OBRIGATORIAMENTE** adicione as colunas abaixo para garantir rastreabilidade completa da cadeia:

| Coluna | Descrição |
|---|---|
| **Produtor** | Artefato que gerou ou transmitiu o dado upstream (nome nominal e técnico) |
| **Consumidor** | Artefato que consome o dado downstream (nome nominal e técnico) |
---

## Modelo de Dados

Esta seção consolida a estrutura mínima esperada para suportar o novo modelo de identificação.

### Estrutura obrigatória

- **Chave de CNPJ** (numérica, 14 posições)
- **Valor de CNPJ** (alfanumérico, 14 posições)
- Indicadores de controle
- Metadados de auditoria

### Regra fundamental

- Deve existir relação **1:1 obrigatória** entre **Chave de CNPJ** e **Valor de CNPJ**
- A **Chave de CNPJ** não substitui semanticamente o **Valor de CNPJ**
- O **Valor de CNPJ** não deve ser usado como chave técnica por inferência automática sem análise do caso

---

## **1\. Objetivo**

Construir um processo automatizado que consome um artefato em formato Markdown oriundo do discovery, analisa os artefatos listados, identifica usos de CNPJ no código e determina o impacto da mudança para o formato alfanumérico.

O resultado deve ser estruturado em dois níveis:

* **Macro**: visão consolidada por artefato  
* **Micro**: detalhamento por ocorrência dentro do artefato

Além disso, o processo deve identificar dependências não mapeadas e classificá-las.

---

## **2\. Premissas**

### 2.1 Premissas principais

* Todos os artefatos do discovery existem e são acessíveis  
* O código fonte está legível (não compilado/binário)  
* O sistema atual considera CNPJ numérico  
* O objetivo é orientar refatoração  
* Dependências externas podem estar incompletas    
* Estaremos aplicando as modificações dos artefatos do módulo principal, sem modificar os artefatos de módulos vizinhos, por entender que estes serão modificados durante a rodada de execução das mudanças do módulo vizinho em questão.    
* Não pode pegar a KEY nem o VALUE dos módulos que não sejam principais    
* Não pode alterar módulos secundários.
* Não deve ser considerada a identificação de CNPJ como chave ou valor em artefatos pertencentes a módulos classificados como não principais  
* A demanda de adequação ao CNPJ alfanumérico opera em dois regimes distintos de abrangência, aplicados sequencialmente e com propósitos complementares, não intercambiáveis.

### 2.2 Premissas e Regras Críticas de Transformação

Esta seção define restrições obrigatórias que devem ser consideradas durante o diagnóstico e que impactam diretamente a decisão de alteração dos artefatos.

Essas premissas têm **prioridade máxima sobre qualquer regra de refatoração**.

### 2.3 Premissas de Execução Pontual

**Premissa de Execução Pontual:** A demanda de adequação ao CNPJ alfanumérico opera em dois regimes distintos de abrangência, aplicados sequencialmente e com propósitos complementares, não intercambiáveis.

**No Diagnóstico**, a abrangência é **máxima e irrestrita dentro do escopo de discovery**. Todos os artefatos identificados na etapa de Descoberta devem ser analisados integralmente, independentemente da expectativa de alteração. A análise não deve ser antecipadamente filtrada por critério de relevância, pois a relevância só pode ser determinada após a análise. A priorização da intervenção é consequência da análise — não sua premissa. Artefatos com alta taxa de inferência de impacto determinam o grau e a urgência da intervenção. Artefatos com baixa taxa de inferência são igualmente analisados, mas resultam em classificação como Indefinido e são preservados. O Diagnóstico não decide o que alterar: ele produz a evidência técnica que fundamenta essa decisão.

**No Plano de Execução**, a abrangência é **mínima e estritamente delimitada pelo Diagnóstico**. Esta demanda não caracteriza modificação ampla, reestruturação integral, refatoração global ou modernização irrestrita de código. A execução deve ser tratada como intervenção pontual, limitada ao conjunto exato de artefatos e campos para os quais o Diagnóstico produziu evidência determinística de necessidade de alteração. Nenhuma modificação pode ser realizada por conveniência técnica, padronização, estética, organização ou oportunidade de melhoria paralela. A intervenção autorizada é exclusivamente aquela rastreável a uma ocorrência concreta identificada no Diagnóstico, com classificação confiável e ação definida. Todo o restante deve ser preservado intacto.

>A assimetria entre os dois regimes é intencional e estrutural: o Diagnóstico é amplo para garantir que nenhum risco seja ignorado; o Plano é restrito para garantir que nenhuma alteração seja indevida.

---

### Premissas Principais

* Todos os artefatos do discovery existem e são acessíveis
* O código fonte está legível (não compilado/binário)
* O sistema atual considera CNPJ numérico
* O objetivo é orientar refatoração
* Dependências externas podem estar incompletas
* As modificações de código serão aplicadas exclusivamente nos artefatos do módulo principal. Artefatos de módulos secundários **não serão modificados** por esta demanda, cabendo a cada módulo secundário tratar suas próprias adequações em rodada de execução própria e sob responsabilidade de seu time.
* Módulos secundários **devem ser diagnosticados integralmente**, incluindo identificação e classificação de todos os campos CNPJ descobertos na etapa de discovery. O diagnóstico é obrigatório para expor a rastreabilidade completa upstream e downstream da cadeia de impacto, independentemente de qualquer decisão de alteração. A ausência de modificação não dispensa a análise.
* A identificação de CNPJ como Chave ou Valor em artefatos de módulos secundários deve ser realizada e registrada no diagnóstico para fins de rastreabilidade, mas **não gera autorização de alteração** sobre esses artefatos. A classificação semântica em módulos secundários serve exclusivamente para mapear o impacto da cadeia e direcionar o time responsável pelo módulo secundário.
* A demanda de adequação ao CNPJ alfanumérico opera em dois regimes distintos de abrangência, aplicados sequencialmente e com propósitos complementares, não intercambiáveis.

---

### Premissas e Regras Críticas de Transformação

Esta seção define restrições obrigatórias que devem ser consideradas durante o diagnóstico e que impactam diretamente a decisão de alteração dos artefatos.

Essas premissas têm **prioridade máxima sobre qualquer regra de refatoração**.

---

### Premissa Técnica — Não Criação de Nova Versão de Artefatos (V2)

Nesta demanda, **não será adotada estratégia de versionamento funcional de artefatos**, tais como criação de programas, copybooks, layouts, JCLs, PROCedures, mapas ou componentes correlatos com sufixação de versão lógica ou física, como `V2`, `NEW`, `_N`, `_ALT` ou equivalentes. A atuação deverá ocorrer **sobre os artefatos correntes já homologados na cadeia operacional da plataforma alta**, por meio de **ajustes pontuais e controlados**, preservando nomenclatura, identidade física, contratos de execução, encadeamentos batch, referências cruzadas, catálogo de carga e vínculos operacionais já existentes no ambiente mainframe.

Essa premissa visa evitar **duplicidade de objetos**, **desvio de roteamento**, **quebra de dependências estáticas**, **aumento desnecessário da superfície de manutenção** e **complexidade adicional de promoção entre ambientes**, assegurando que a solução permaneça integrada ao ciclo normal de compilação, linkedição, implantação e operação produtiva, sem introdução de uma trilha paralela de artefatos versionados.


---

### Premissas de Execução Pontual

**Premissa de Execução Pontual:** A demanda de adequação ao CNPJ alfanumérico opera em dois regimes distintos de abrangência, aplicados sequencialmente e com propósitos complementares, não intercambiáveis.

**No Diagnóstico**, a abrangência é **máxima e irrestrita dentro do escopo de discovery**. Todos os artefatos identificados na etapa de Descoberta devem ser analisados integralmente, independentemente da expectativa de alteração. Isso inclui obrigatoriamente os artefatos de módulos secundários, cujo diagnóstico é necessário para garantir rastreabilidade upstream e downstream de toda a cadeia de impacto. A análise não deve ser antecipadamente filtrada por critério de relevância, pois a relevância só pode ser determinada após a análise. A priorização da intervenção é consequência da análise — não sua premissa. Artefatos com alta taxa de inferência de impacto determinam o grau e a urgência da intervenção. Artefatos com baixa taxa de inferência são igualmente analisados, mas resultam em classificação como Indefinido e são preservados. O Diagnóstico não decide o que alterar: ele produz a evidência técnica que fundamenta essa decisão. Módulos secundários diagnosticados com ocorrências de CNPJ devem ter seus resultados registrados como evidência de impacto downstream e upstream, sinalizando ao time responsável pelo módulo secundário a necessidade de tratamento equivalente em sua própria rodada de execução.

**No Plano de Execução**, a abrangência é **mínima e estritamente delimitada pelo Diagnóstico**. Esta demanda não caracteriza modificação ampla, reestruturação integral, refatoração global ou modernização irrestrita de código. A execução deve ser tratada como intervenção pontual, limitada ao conjunto exato de artefatos e campos para os quais o Diagnóstico produziu evidência determinística de necessidade de alteração. Módulos secundários estão excluídos do escopo de execução, mesmo quando diagnosticados com ocorrências de CNPJ. Nenhuma modificação pode ser realizada por conveniência técnica, padronização, estética, organização ou oportunidade de melhoria paralela. A intervenção autorizada é exclusivamente aquela rastreável a uma ocorrência concreta identificada no Diagnóstico, com classificação confiável e ação definida, em artefato pertencente ao módulo principal. Todo o restante deve ser preservado intacto.

> A assimetria entre os dois regimes é intencional e estrutural: o Diagnóstico é amplo para garantir que nenhum risco seja ignorado; o Plano é restrito para garantir que nenhuma alteração seja indevida.

---

#### 2.2.1 Preservação de Dados Externos

**Premissa**

Dados de CNPJ ou CGC provenientes de sistemas externos **não devem sofrer qualquer tipo de transformação**.

---

**Regra**

* Se a origem do dado for externa (ex: DB2, artefatos, integrações):  
  * **Não deve converter para formato alfanumérico**  
  * **Não deve normalizar**  
  * **Não deve reclassificar**  
  * **Não deve validar ou recalcular**

---

**Impacto no Diagnóstico**

* Qualquer ocorrência identificada como origem externa deve resultar em:

| campo | valor esperado |
| ----- | ----- |
| ação | Nenhuma |
| observação | Dado externo — transformação proibida |
| prioridade | baixa (do ponto de vista de alteração) |

---

**Implicação prática**

Mesmo que o campo contenha CNPJ numérico:

* **não deve ser alterado**  
* deve ser apenas **preservado**

---

#### 2.2.2 Bloqueio de Transformação (Bypass de Pipeline)

**Premissa**

Campos identificados como origem externa devem **ignorar completamente o pipeline de transformação**.

---

**Regra**

* Se `origem = externa`:  
  * bypass total das regras de refatoração

---

**Impacto no Diagnóstico**

Adicionar observação:

"Transformação bloqueada por origem externa"

---

#### 2.2.3 Controle de Inferência e Classificação

**Premissa**

A classificação do CNPJ (CHAVE ou VALOR) só pode ocorrer quando houver **inferência determinística**.


---

**Regra**

* Inferência forte → pode classificar  
* Inferência fraca → classificar como **INCERTO (Indefinido)**

---

**Impacto no Diagnóstico**

Conectar diretamente com suas tabelas:

* **qtd\_incerto aumenta**  
* classificação macro (`é incerto = true`)  
* entrada na tabela de **Taxa de Inferência**

---

#### 2.2.4 Bloqueio por Estado INCERTO

**Premissa**

Ocorrências classificadas como **INCERTO (Indefinido)** não podem sofrer transformação.

---

**Regra**

Se classificação \= INCERTO:

* Não converter  
* Não validar  
* Não gerar chave  
* Não alterar tipo

---

**Impacto no Diagnóstico**

| campo | valor esperado |
| ----- | ----- |
| ação | Revisar manualmente |
| prioridade | média/alta |
| observação | Classificação indefinida — transformação bloqueada |

---

#### 2.2.5 Regra de Ouro do Diagnóstico

**Nenhuma transformação deve ser aplicada sem certeza semântica e sem garantia de origem interna controlada.**

---

## **3\. Escopo (nome padrão)**

### **Inclui:**

* Análise estática de código  
* Diagnóstico de incompatibilidade  
* Mapeamento de dependências  
* Consumo do artefato gerado pelo Discovery

### **Não inclui:**

* Refatoração automática  
* Execução do sistema

---

## **4\. Tabela Macro e Micro**

É necessário que seja gerado uma tabela macro e várias micro por artefato de código, seguindo os modelos abaixo:

### **4.1** Estrutura MACRO

#### 4.1.1 Tabela Principal

| id | artefato | prioridade | observacao |
| :---: | :---: | :---: | :---: |

Legenda:

- id: identificação  
- artefato: nome do artefato  
- prioridade: ALTA, MÉDIA ou BAIXA  
- observacao: observações do código

**Regra de preenchimento da coluna observacao**

A coluna deve ser composta por **frases curtas, separadas por “;”**, contendo apenas problemas detectados.

**Template de preenchimento:**  
Uso de tipo numerico para CNPJ; Conversao numerica identificada; Validacao restritiva detectada; Exposicao em saida; Inconsistencia de entrada/saida

**Regras objetivas:**

* Inserir **somente problemas encontrados no artefato**  
* Não descrever solução  
* Não repetir informação já presente na prioridade

---

**Regra de preenchimento da coluna prioridade**

| Condição encontrada | Prioridade |
| ----- | ----- |
| Tipo numérico / parsing / persistência inválida | ALTA |
| Conversão indevida | ALTA |
| Exposição em saída | ALTA |
| Validação incorreta | ALTA |
| Formatação/máscara | MÉDIA |
| Inconsistência de fluxo | MÉDIA |

---

#### 4.1.2 Ocorrência

Classifica como o CNPJ é manipulado dentro do artefato.

Campos:

* id  
* artefato  
* tem função  
* faz chamada  
* tem variável  
* tem comentário


**Legenda:**

* **id**: identificador único  
* **artefato**: nome do artefato  
* **tem função**: existe função relacionada a CNPJ  
* **faz chamada**: chamada de função relacionada a CNPJ  
* **tem variável**: variável associada a CNPJ  
* **tem comentário**: menção em comentário de CNPJ

Todos os campos são booleanos (exceto id e artefato).

#### ---

#### 4.1.3 Chave, Valor e Incerto

Classifica o papel do CNPJ nas estruturas de dados.

---

### **Campos:**

id  
artefato  
é chave  
é valor  
é incerto
definição

---

### **Regra de classificação:**

Deve ser avaliado se o artefato manipula CNPJ como chave, valor ou se há ocorrências onde não é possível determinar com precisão.

A classificação é **acumulativa por artefato**, baseada nas ocorrências identificadas no nível detalhado (micro).

---

### **Legenda:**

* **id**: identificador único  
* **artefato**: nome do artefato  
* **é chave**:  
  Indica que o artefato possui ao menos uma ocorrência onde o CNPJ é utilizado como identificador único  
* **é valor**:  
  Indica que o artefato possui ao menos uma ocorrência onde o CNPJ é utilizado como dado informacional  
* **é incerto**:  
  Indica que o artefato possui ao menos uma ocorrência onde não foi possível determinar o papel do CNPJ

---

### **Regras adicionais:**

* Os campos **é chave**, **é valor** e **é incerto** são booleanos  
* **Mais de um campo pode ser verdadeiro simultaneamente**  
* Cada campo representa a existência de pelo menos uma ocorrência daquele tipo no artefato  
* **é incerto \= true** indica que há necessidade de análise manual parcial ou complementar  
* A classificação macro deve ser derivada da análise micro (taxa de inferência)

---

### **Diretrizes de Classificação**

A classificação deve considerar o **contexto de uso do CNPJ no artefato**, e não apenas a estrutura técnica.

Deve-se observar:

* Uso em identificadores → marcar **é chave \= true**  
* Uso como dado informacional → marcar **é valor \= true**  
* Uso ambíguo ou insuficiente → marcar **é incerto \= true**

---

### **Exemplo**

| id | artefato | é chave | é valor | é incerto |
| ----- | ----- | ----- | ----- | ----- |
| 1 | CUSTINQ1.cbl | true | false | false |
| 2 | CUSTDISP.cbl | false | true | false |
| 3 | CUSTTEMP.cbl | false | false | true |
| 4 | CUSTFULL.cbl | true | true | false |
| 5 | CUSTMIX.cbl | true | true | true |

---

### 4.2. Estrutura MICRO (por artefato)

#### 4.2.1 Antes/Depois

Esta seção descreve, para cada ocorrência identificada, o estado atual do código (**antes**), a proposta de alteração (**depois**) e a justificativa técnica (**motivo**), com base na análise do contexto.

---

**Estrutura**

| id\_macro | linha | antes | depois | acao | motivo |
| :---: | :---: | :---: | :---: | :---: | :---: |

---

**Definição dos Campos**

* **id\_macro**: identificação do artefato (relacionado à tabela macro)  
* **linha**: número da linha no código  
* **antes**: representação do trecho atual  
* **depois**: representação proposta após refatoração  
* **acao**: ação a ser executada (ex: ajustar tipo, revisar manualmente, nenhuma)

---

**motivo (campo obrigatório e detalhado)**

O campo **motivo** deve conter uma explicação completa, técnica e contextualizada, incluindo:

---

**Regras para preenchimento do "motivo"**

O motivo deve obrigatoriamente descrever:

**1\. Origem da variável/dado**

* De onde o CNPJ está vindo:  
  * parâmetro de entrada (LINKAGE SECTION)  
  * variável local (WORKING-STORAGE)  
  * banco de dados (DB2)  
  * chamada de programa  
  * artefato externo

---

2**. Destino da variável/dado**

* Para onde o valor está sendo utilizado:  
  * validação  
  * comparação  
  * persistência  
  * output (DISPLAY, retorno, API)  
  * passagem para outro programa

---

**3\. Papel da ocorrência**

* Se atua como:  
  * chave  
  * valor  
  * incerto

---

**4\. Problema identificado**

* Qual a limitação atual:  
  * tipo numérico  
  * regex restritiva  
  * conversão inadequada  
  * formatação fixa  
  * incompatibilidade com alfanumérico

---

**5\. Regra aplicada no diagnóstico**

* Qual regra motivou a decisão:  
  * adaptação para CNPJ alfanumérico  
  * bloqueio por inferência  
  * preservação de dado externo  
  * classificação semântica

---

**6\. Justificativa da ação**

* Por que a alteração deve (ou não) ser feita:  
  * garantir compatibilidade  
  * evitar quebra de lógica  
  * preservar integridade do dado  
  * evitar transformação indevida

---

**Padrão esperado do texto**

O campo **motivo** deve ser escrito de forma estruturada, contendo:

origem → uso → problema → decisão → justificativa

---

**Exemplos**

**Exemplo 1 — Transformação necessária**

| linha | antes | depois | acao | motivo |
| ----- | ----- | ----- | ----- | ----- |
| 45 | IF CNPJ \= WS-CNPJ | IF CNPJ \= WS-CNPJ | Ajustar tipo | Variável CNPJ originada da LINKAGE SECTION (entrada do programa) e utilizada em comparação direta como identificador (chave). Atualmente definida como numérica, o que impede suporte a caracteres alfanuméricos. A regra de adaptação para CNPJ alfanumérico exige alteração do tipo para string, garantindo compatibilidade sem alterar a lógica de comparação. |

---

**Exemplo 2 — Bloqueio por dado externo**

| linha | antes | depois | acao | motivo |
| ----- | ----- | ----- | ----- | ----- |
| 78 | MOVE DB-CNPJ TO WS-CNPJ | (sem alteração) | Nenhuma | Valor de CNPJ originado de base externa (DB2) e transferido para variável interna sem transformação. De acordo com a regra de preservação de dados externos, não é permitida conversão, normalização ou alteração estrutural. O valor deve ser mantido exatamente como recebido. |

---

**Exemplo 3 — Incerto**

| linha | antes | depois | acao | motivo |
| ----- | ----- | ----- | ----- | ----- |
| 120 | MOVE CNPJ TO WS-TEMP | (indefinido) | Revisar manualmente | Variável CNPJ com origem não claramente identificada e sendo movimentada para variável temporária sem contexto explícito de uso. Não há evidência suficiente para determinar se atua como chave ou valor. De acordo com a regra de inferência, a classificação é incerta, bloqueando qualquer transformação automática até análise manual. |

---

**Regras Críticas**

* O campo **motivo é obrigatório para todas as linhas**  
* Motivos genéricos são proibidos (ex: "ajuste necessário")  
* Deve sempre haver referência ao **contexto de uso**  
* Deve refletir as decisões das seções:  
  * classificação (chave/valor/incerto)  
  * taxa de inferência  
  * regras de transformação

---

#### 4.2.2 Taxa de Inferência

Esta seção complementa a classificação com uma análise probabilística, permitindo entender o grau de confiança da decisão tomada pela LLM e **determinando diretamente a ação a ser aplicada no diagnóstico**.

---

**Estrutura**

| id\_macro | artefato | linha | codigo | % chave | % valor | % incerto | porque\_chave | porque\_valor | porque\_incerto |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |

---

**Definição das Colunas**

* **id\_macro**: identificação do id macro relacionado  
* **artefato**: nome do artefato analisado  
* **linha**: número da linha  
* **codigo**: conteúdo da linha de código  
* **% chave**: probabilidade de ser classificado como CHAVE  
* **% valor**: probabilidade de ser classificado como VALOR  
* **% incerto**: probabilidade de ser classificado como INCERTO  
* **porque\_chave**: justificativa da inferência como CHAVE  
* **porque\_valor**: justificativa da inferência como VALOR  
* **porque\_incerto**: justificativa da inferência como INCERTO

---

**Regras de Inferência**

* A soma de **% chave \+ % valor \+ % incerto deve ser igual a 100%**  
* Deve existir ao menos uma justificativa preenchida  
* A inferência deve considerar o contexto semântico da linha  
* Evidências funcionais têm prioridade sobre estrutura técnica

---

**Regra de Threshold (Obrigatória)**

* LLM deve explicar qual threshold foi definido e por que foi definido esse threshold.  
* Esta regra deve ser aplicada em todas as colunas onde é solicitado uma inferência de threshold.  
* Sempre que for pedido inferência coloque as colunas abaixo:  
  * % da inferência: Percentual de probabilidade sobre o tema a ser inferido  
  * Label: É o rótulo que a LLM irá categorizar e fará a inferência (Ex: chave, valor, complexo, simples, baixo, alto).  
  * Porque: O motivo que a LLM atribui o determinado rótulo. A LLM deve explicar com riquezas de detalhes e exemplos, critérios para a categorização ou a rotulação.  
* Esta regra deve ser explícita logo abaixo da tabela de forma textual e tabular.

---

**Integração com a Coluna "ação" (Tabela MICRO)**

A taxa de inferência deve **determinar diretamente a ação a ser registrada na tabela micro do diagnóstico**.

---

**Regras de Decisão**

**1\. Presença de INCERTO (qualquer nível)**

**Condição:**

* % incerto \> 0%

**Ação (OBRIGATÓRIA):**

* **Revisar manualmente**  
* Não aplicar transformação automática

---

**Regras Críticas**

* **% incerto \> 0% sempre bloqueia automação**  
* Nenhuma transformação deve ser aplicada sem confiança determinística  
* A coluna **ação** deve refletir exatamente o resultado da inferência

---

**Exemplo de Aplicação**

| id\_macro | linha | % chave | % valor | % incerto | ação esperada |
| ----- | ----- | ----- | ----- | ----- | ----- |
| 1 | 45 | 85% | 10% | 5% | Revisar manualmente |
| 1 | 78 | 5% | 90% | 5% | Revisar manualmente |
| 1 | 120 | 30% | 40% | 30% | Revisar manualmente |

---

## **5\. Regras de Diagnóstico Aplicadas**

### **Regra 1 — Tipo Numérico**

* **Explicação:** Detecta uso de tipos numéricos para armazenamento de CNPJ  
* **Problema:** Não suporta caracteres alfabéticos  
* **Aplicado em:** variáveis e estruturas de dados  
* **Ação:** Alterar tipo para string  
* **Severidade:** Alta

---

### **Regra 2 — Conversão Numérica**

* **Explicação:** Identifica conversões explícitas para número  
* **Problema:** Quebra com CNPJ alfanumérico  
* **Aplicado em:** funções com parse e casting  
* **Ação:** Remover conversões numéricas  
* **Severidade:** Alta

---

### **Regra 3 — Validação Restritiva**

* **Explicação:** Detecta validações que aceitam apenas números  
* **Problema:** Bloqueia novo formato  
* **Aplicado em:** regras de validação  
* **Ação:** Permitir caracteres alfanuméricos  
* **Severidade:** Alta

---

### **Regra 4 — Persistência em Banco**

* **Explicação:** Identifica campos incompatíveis  
* **Problema:** Tipos numéricos não suportam letras  
* **Aplicado em:** DB2, artefatos de dados  
* **Ação:** Migrar para VARCHAR(14)  
* **Severidade:** Alta

---

### **Regra 5 — Máscara Fixa**

* **Explicação:** Detecta formatação baseada em números  
* **Problema:** Não suporta letras  
* **Aplicado em:** formatadores e UI  
* **Ação:** Adaptar máscara  
* **Severidade:** Média

---

### **Regra 6 — Código Compatível**

* **Explicação:** Código já suporta alfanumérico  
* **Problema:** Nenhum  
* **Aplicado em:** validações modernas  
* **Ação:** Nenhuma  
* **Severidade:** Baixa

---

### **Regra 7 — Uso Indevido em Contexto**

* Explicação: identifica uso de CNPJ em fluxo incorreto (entrada, saída ou processamento)  
* Problema: quebra de regra funcional de uso  
* Aplicado em: entradas, saídas e processamento  
* Ação: corrigir uso conforme fluxo  
* Severidade: Alta

---

### **Regra 8 — Exposição Indevida**

* Explicação: identifica exposição de identificador em saída  
* Problema: violação de regra de exposição  
* Aplicado em: logs, telas, APIs, artefatos  
* Ação: ajustar saída  
* Severidade: Alta

---

### **Regra 9 — Conversão Não Autorizada**

* Explicação: identifica transformação local de CNPJ  
* Problema: quebra de centralização  
* Aplicado em: parsing, casting, manipulação manual  
* Ação: remover lógica de conversão  
* Severidade: Alta

---

### **Regra 10 — Geração Indevida**

* Explicação: identifica criação local de identificador  
* Problema: quebra de governança  
* Aplicado em: inicializações, sequências, cálculos  
* Ação: remover geração local  
* Severidade: Alta

---

### **Regra 11 — Inconsistência de Fluxo**

* Explicação: identifica divergência entre entrada e saída  
* Problema: comportamento inconsistente  
* Aplicado em: interfaces e processamento  
* Ação: ajustar fluxo  
* Severidade: Média 


---

### **Regra 12 — Conversão centralizada obrigatória entre Chave e Valor de CNPJ**

Sempre que houver necessidade de transformar **Valor de CNPJ** em **Chave de CNPJ**, ou **Chave de CNPJ** em **Valor de CNPJ**, a conversão deverá ser realizada **exclusivamente** por meio da invocação do **Serviço Central de Conversão (DFE)**, sendo **vedada** qualquer tentativa de geração, derivação, manutenção ou persistência local da correlação entre esses identificadores. Essa regra garante que a correspondência Chave-Valor permaneça **determinística, única, auditável e aderente à arquitetura corporativa**, proibindo o uso de tabelas auxiliares, algoritmos internos, caches de correlação com autoria local ou qualquer lógica distribuída que reproduza a função do serviço central.

---

## **6\. Definição de Prioridade**

| prioridade | critério |
| :---- | :---- |
| alta | quebra funcional |
| média | impacto indireto |
| baixa | ajuste simples |

---

## **7\. Recomendações Prioritárias**

| ordem | recomendacao | descricao | impacto |
| :---- | :---- | :---- | :---- |
| 1 | Ajustar validações | Atualizar regex e regras | Alto |
| 2 | Corrigir tipos | Substituir número por string | Alto |
| 3 | Remover parsing | Eliminar conversões | Alto |
| 4 | Ajustar persistência | Migrar banco | Alto |
| 5 | Atualizar APIs | Aceitar string | Médio |
| 6 | Ajustar formatação | Atualizar máscaras | Médio |
| 7 | Revisar dependências | Garantir compatibilidade | Médio |

---

## **8\. Dependências**

Tabela que mapeia as relacoes de dependencia entre artefatos, identificando quais programas chamam e sao chamados por outros artefatos no contexto do CNPJ. A tabela deve ser ordenada pelo artefato mais referenciado primeiro (maior valor de qtd é chamado), de forma a priorizar os artefatos de maior acoplamento.

| id | artefato | identificado | tipo | qtd chama | é chamado por | qtd é chamado |
| :---: | :---- | :---- | :---- | :---: | :---- | :---: |

**Legenda:**

- **id**: identificador unico sequencial (DEP-01, DEP-02, ...)  
- **artefato**: nome do programa ou copybook  
- **identificado**: indicador de que o artefato está presente ou não nos artefatos  
- **tipo**: classificacao do artefato (Programa, Copybook, artefatos de dados, DB2, Biblioteca)  
- **qtd chama**: quantidade total de artefatos que este artefato invoca  
- **é chamado por**: lista de artefatos que invocam este artefato  
- **qtd é chamado**: quantidade total de artefatos que invocam este artefato

**Regra de ordenacao:** a tabela deve ser ordenada de forma decrescente pela coluna **qtd é chamado**. Artefatos mais referenciados (mais chamados por outros) devem aparecer primeiro, pois representam maior risco de impacto em cadeia

Separe a tabela nas seguintes seções:

- Programas / Copybooks  
- Persistência (artefatos de dados e DB2)  
- Bibliotecas

**Regra de preenchimento da coluna tipo**

Adicionar classificação adicional quando relacionado a CNPJ:

* Programa  
* Copybook  
* DB2  
* Biblioteca

(sem criar novos tipos)

**Regra de identificação:**

Considerar dependência quando houver:

* CALL  
* COPY  
* INCLUDE  
* EXEC SQL

**Regra adicional:**

* Se houver manipulação de CNPJ → garantir que o artefato esteja listado

---

## 9\. Persistencias Identificadas que Requerem Modificacoes

Esta secao consolida as persistencias identificadas durante o diagnostico que necessitam de modificacao para suportar o CNPJ alfanumerico. O levantamento abrange tanto persistencias em banco de dados (DB2) quanto mapear as relacoes de chamada entre artefatos e fornecer rastreabilidade campo-a-campo.

### 9.1 Artefato vs Banco de Dados

Tabela que relaciona artefatos de codigo com campos de banco de dados (DB2) que requerem modificacao para suportar CNPJ alfanumerico.

| id | tabela | campo | artefato | ocorrencia |
| :---: | :---- | :---- | :---- | :---- |

**Legenda:**

- **id**: identificador unico sequencial (PDB-01, PDB-02, ...)  
- **tabela**: nome da tabela DB2 que requer modificacao  
- **campo**: nome do campo/coluna da tabela que requer modificacao  
- **artefato**: nome do artefato de codigo  
- **ocorrencia**: trecho do codigo-fonte onde o campo e referenciado (ex.: instrucao SQL, EXEC SQL, MOVE, etc.)

**Regra de preenchimento da coluna ocorrencia**

A ocorrência deve conter **trecho real de acesso ao banco**.

**Exemplo de preenchimento:**  
EXEC SQL SELECT CNPJ FROM CLIENTE END-EXEC

**Regra de identificação de problema (implícita na tabela)**

Sempre considerar problema quando houver:

* Campo numérico  
* Uso de CAST para número  
* Comparação numérica

**Regra de consistência**

* Não descrever problema fora da tabela  
* A evidência deve estar no trecho

---

### **9.2 Tabela de Correlação**

Seção adicional para garantir rastreabilidade consolidada entre artefatos, evidenciando relações de dependência e impacto entre artefatos através de uma matriz cruzada.

| artefato \\ artefato | ARQ01 | ARQ02 | ARQ03 | ARQ04 |
| :---: | :---: | :---: | :---: | :---: |
| **ARQ01** | \- | X |  | X |
| **ARQ02** | X | \- | X |  |
| **ARQ03** |  | X | \- | X |
| **ARQ04** | X |  | X | \- |

Legenda:

- artefato (linhas): artefato de origem da análise  
- artefato (colunas): artefato relacionado  
- X: indica que há relação, dependência ou impacto entre os artefatos  
- \-: indica o próprio artefato (não aplicável)

Regras:

- A matriz deve ser simétrica quando a relação for bidirecional  
- Relações unidirecionais podem ser representadas apenas em um dos sentidos  
- Todos os artefatos identificados no diagnóstico devem estar presentes tanto nas linhas quanto nas colunas  
- A marcação deve considerar chamadas, compartilhamento de dados, uso de copybooks e dependências indiretas

---

### 9.3 Matriz de Rastreabilidade Campo-a-Campo

Tabela de rastreabilidade campo-a-campo que permite localizar, para cada campo CNPJ identificado, a cadeia completa: campo, tipo atual, padrao de tratamento, artefato de origem, copybook de interface, programa que manipula e sistema destino. Esta tabela deve ser consistente com a rastreabilidade produzida no Patch Report (secao 7.2), garantindo continuidade entre diagnostico e implementacao.

| id | campo | tipo atual (PIC) | padrao de tratamento | artefato de origem | copybook de interface | programa que manipula | sistema destino |
| :---: | :---- | :---- | :---- | :---- | :---- | :---- | :---- |

**Legenda:**

- **id**: identificador unico sequencial (TRC-C01, TRC-C02, ...)  
- **campo**: nome do campo CNPJ identificado  
- **tipo atual (PIC)**: tipo COBOL exato do campo (ex.: PIC S9(014) COMP-3, PIC 9(014), PIC X(014))  
- **padrao de tratamento**: classificacao conforme padroes definidos — LookUp Content / Real Content / Ambos  
- **artefato de origem**: artefato onde o campo e declarado  
- **copybook de interface**: copybook que define ou propaga o campo  
- **programa que manipula**: programa COBOL que efetivamente manipula o campo  
- **sistema Destino**: sistema para o qual o dado e destinado (ex.: VIP, DAF, SIAFI)

**Regras:**

- Cada campo CNPJ identificado nas secoes anteriores (11.1 e 11.2) deve ter ao menos uma linha correspondente nesta tabela  
- Campos que passam por multiplos programas devem ter multiplas linhas  
- A coluna "Padrao de Tratamento" deve indicar LookUp / Real / Ambos  
- Quando um campo pertence a um modulo vizinho (ex.: DAF), registrar na tabela mas indicar que a modificacao sera tratada pelo modulo responsavel, conforme premissa estabelecida na secao 2  
- Esta tabela serve como base de entrada para a rastreabilidade do Patch Report (secao 7.2), que adicionara colunas de artefato NEW, Linha OLD/NEW e Tipo NEW apos a implementacao

---

## **10\. Rodapé do Documento (Obrigatório)**

Ao final do documento deve ser incluído e levar em consideração o dia atual para dd/mm/yyyy:

Documentação gerada em:

Documento Elaborado: dd/mm/yyyy

Versão: 9.0

---

## **11\. Restrições de Conteúdo Gerado**

### 11.1 Formatação

* Não deve ser utilizado nenhum tipo de emoji  
* A documentação deve manter padrão técnico e formal  
* Colunas com nomes compostos devem ser escritas com espaço entre as palavras, sem utilização de underscore (\_)

### 11.2 Conteúdo

No início de cada seção, deve haver um parágrafo curto, explicando o que a seção vai abordar.

Cada seção deve ser numerada, a partir do número um (1).

O output deve conter apenas e na seguinte ordem:

* Introdução (Curta explicação do documento)  
* Escopo  
* Premissas  
* Estruturas de dados (tabelas)  
* Regras de diagnóstico  
* Conclusão

Qualquer conteúdo fora desse padrão deve ser considerado inválido.

### 11.3 Tabela

Cada tabela deve ter uma parte de legenda seguindo o seguinte padrão:

Legenda:

- campo: descrição

### 11.4 Título

O título do artefato gerado deve ser: Diagnostic

---

## 12\. Conclusão

Ao final do diagnostic deve ser gerada uma conclusão objetiva baseada exclusivamente nos dados coletados durante a análise.

A conclusão deve:

* Resumir o que foi feito com base no outputs,  devendo ser um único parágrafo técnico

A conclusão deve ser:

* Técnica  
* Objetiva

Não deve conter:

* Explicações conceituais  
* Repetição de conteúdo já detalhado nas tabelas  
* Recomendações de implementação

**Regras:**

* Não listar itens  
* Não explicar regras  
* Não sugerir solução  
* Apenas consolidar achados

---

## **13\. Diagnostic Complementar (Ambiguidades e Ausência de Informação)**

Seção destinada a explicitar lacunas de análise, pontos ambíguos e riscos decorrentes de ausência de informação durante o discovery.

* Ausência de definição clara sobre origem de determinados campos CNPJ em fluxos indiretos  
* Ambiguidade na distinção entre campos classificados como KEY e VALUE em alguns artefatos  
* Falta de padronização na nomenclatura de variáveis relacionadas ao CNPJ  
* Dependências externas não completamente mapeadas  
* Possível existência de regras de negócio implícitas não documentadas  
* Inconsistência entre copybooks e uso real em programas COBOL  
* Lacunas na rastreabilidade entre entrada, processamento e saída

---

## **14\. Módulos Secundários (Externo)**


Seção que reforça as restrições relacionadas a módulos aa e garante governança sobre o escopo de alteração.

| modulo | tipo | pode alterar | observacao |
| :---: | :---: | :---: | :---: |

Legenda:

* modulo: nome do módulo externo (ex.: DAF, HLP)  
* tipo: classificação do módulo  
* pode alterar: SIM ou NÃO  
* observacao: restrições aplicáveis

Regras:

* Não pode alterar módulos aa (DAF, HLP, etc)  
* Caso identificado impacto, apenas registrar

A inferência de quem é módulo principal e módulo secundário (externo) deve ser feita com base na análise de código e na identificação de artefatos, e módulos secundários são aqueles podem ou não ter manipulação direta de "CNPJ", ou que são claramente identificados como módulos de apoio (ex.: DAF, HLP, etc).   

---

## **15\. Validação de DV (Módulo 11\)**

Seção que define controle sobre chamada de função de validação de dígito verificador, indicando se o artefato precisa ou não dessa função.

| id | artefato | campo | aplica DV | chama funcao | observacao |
| :---: | :---: | :---: | :---: | :---: | :---: |

Legenda:

* id: identificador  
* artefato: artefato analisado  
* campo: campo CNPJ  
* aplica DV: SIM ou NÃO  
* chama funcao: SIM ou NÃO  
* observacao: detalhe técnico

**Regra de preenchimento:**

| Situação encontrada | aplica DV | chama funcao | observacao |
| ----- | ----- | ----- | ----- |
| Validação presente | SIM | SIM | Validacao identificada |
| Validação ausente | NÃO | NÃO | Validacao nao encontrada |
| Validação indevida | SIM | SIM | Aplicacao inconsistente |

**Regra:**

* Não inferir regra de negócio  
* Apenas registrar evidência no código

---

## 16\. Tamanho e Consumo de Memória (Antes/Depois)

Esta seção tem como objetivo mensurar o impacto da alteração do CNPJ numérico para alfanumérico no consumo de memória dos artefatos, considerando:

* Parâmetros de entrada (input)  
* Dados de saída (output)  
* Expansão total de bytes no fluxo do programa

A análise deve ser aplicada **apenas em módulos principais que manipulem CNPJ diretamente em cláusulas VALUE ou estruturas equivalentes** e que devem estar em acordo com a tabela.

Após análise e geração da tabela, deve ser feita uma verificação entre ambos, onde caso haja diferença, será necessário ajustar ambos até que fiquem em conformidade. 

---

### 16.1 Estrutura

| id | artefato | antes\_in | antes\_out | depois\_in | depois\_out | delta\_in | delta\_out |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |

---

### 16.2 Definições

* **artefato**: nome do artefato analisado  
* **antes\_in**: total de bytes consumidos nas entradas do artefato antes da alteração  
* **antes\_out**: total de bytes gerados nas saídas do artefato antes da alteração  
* **depois\_in**: total de bytes consumidos nas entradas após adaptação  
* **depois\_out**: total de bytes gerados nas saídas após adaptação  
* **delta\_in**: diferença de consumo de entrada (depois\_in \- antes\_in)  
* **delta\_out**: diferença de saída (depois\_out \- antes\_out)

---

### 16.3 Regras de Cálculo

1. **Mapear entradas do programa**  
   * Parâmetros recebidos (LINKAGE SECTION, USING, etc.)  
   * Campos com CNPJ em estruturas de entrada  
2. **Mapear saídas do programa**  
   * Campos retornados  
   * Registros escritos (artefatos, banco, comunicação)  
3. **Cálculo de delta**  
   * delta\_in \= depois\_in \- antes\_in  
   * delta\_out \= depois\_out \- antes\_out

---

## **17\. Geração de Grafo de Dependências (JSON)**

Esta seção define a geração de um artefato adicional em formato JSON, representando o grafo de dependências entre os artefatos analisados no diagnóstico, tomando como base a tabela de correlação (9.2).

O objetivo é permitir:

* Visualização estrutural de dependências  
* Análise de impacto entre programas  
* Identificação de elementos não resolvidos (unresolved)  
* Integração com ferramentas de grafo

---

### 17.1 Estrutura do JSON

{  
  "status": "success",  
  "total\_files": 0,  
  "total\_nodes": 0,  
  "total\_edges": 0,  
  "nodes": \[\],  
  "edges": \[\]  
}

---

### 17.2 Definição dos Atributos

**status**

Tipo: string  
Obrigatório: sim  
Valores esperados: success ou error  
Regra:  
use success quando houver grafo válido; use error quando não houver dados ou ocorrer falha

---

**total\_files**

Tipo: inteiro  
Obrigatório: sim  
Regra:  
quantidade de artefatos reais analisados (não incluir nós unresolved)

---

**total\_nodes**

Tipo: inteiro  
Obrigatório: sim  
Regra:  
quantidade total de nós (artefatos reais \+ unresolved)

---

**total\_edges**

Tipo: inteiro  
Obrigatório: sim  
Regra:  
quantidade total de relações entre nós

---

**nodes**

Tipo: array de objetos  
Obrigatório: sim  
Regra:  
cada item representa um nó do grafo

---

**edges**

Tipo: array de objetos  
Obrigatório: sim  
Regra:  
cada item representa uma relação entre dois nós

---

### 17.3 Estrutura de Cada Node

{  
  "id": "/app/w\_data/.../CUSTINQ1.cbl",  
  "label": "CUSTINQ1.cbl",  
  "type": "file",  
  "path": "/app/w\_data/.../CUSTINQ1.cbl",  
  "program\_id": "CUSTINQ1",  
  "copies": \["INQSET1.", "KIKAID."\],  
  "calls": \["INVMENU"\],  
  "includes": \[\],  
  "pkgs": \["."\],  
  "bms": \[\],  
  "jcl": \[\],  
  "lines": 196,  
  "last\_modified": "2026-03-02T17:42:01.631461",  
  "source": "BASE64\_DO\_CODIGO\_FONTE"  
}

---

**id**

Tipo: string  
Obrigatório: sim  
Regra:  
para nó real usar caminho absoluto do artefato  
para nó unresolved usar formato unresolved::NOME\_REFERENCIA  
deve ser único no array

---

**label**

Tipo: string  
Obrigatório: sim  
Regra:  
nome amigável do nó (artefato ou referência)

---

**type**

Tipo: string  
Obrigatório: sim  
Valores esperados: file, cpy, bms, jcl, unresolved  
Regra:  
definir conforme tipo/extensão do artefato

---

**path**

Tipo: string  
Obrigatório: sim  
Regra:  
para nó real usar caminho completo  
para unresolved usar string vazia

---

**program\_id**

Tipo: string ou null  
Obrigatório: sim  
Regra:  
valor extraído do PROGRAM-ID; null quando não aplicável

---

**copies**

Tipo: array de string  
Obrigatório: sim  
Regra:  
lista de COPY encontrados no código

---

**calls**

Tipo: array de string  
Obrigatório: sim  
Regra:  
lista de CALL, LINK ou XCTL

---

**includes**

Tipo: array de string  
Obrigatório: sim  
Regra:  
lista de INCLUDE encontrados

---

**pkgs**

Tipo: array de string  
Obrigatório: sim  
Regra:  
representa pacote lógico; usar \["." \] quando não houver segmentação

---

**bms**

Tipo: array  
Obrigatório: sim  
Regra:  
reservado; pode permanecer vazio

---

**jcl**

Tipo: array  
Obrigatório: sim  
Regra:  
reservado; pode permanecer vazio

---

**lines**

Tipo: inteiro  
Obrigatório: sim  
Regra:  
quantidade de linhas do artefato; usar 0 para unresolved

---

**last\_modified**

Tipo: string  
Obrigatório: sim  
Regra:  
data no formato ISO-8601; vazio para unresolved

---

**source**

Tipo: string  
Obrigatório: sim  
Regra:  
conteúdo do artefato em Base64 UTF-8; vazio para unresolved

---

### 17.4 Estrutura de Cada Edge

{  
  "id": "edge\_0",  
  "from": "/app/w\_data/.../CUSTINQ1.cbl",  
  "to": "unresolved::INQSET1.",  
  "type": "uses\_copy"  
}

---

**id**

Tipo: string  
Obrigatório: sim  
Regra:  
identificador único (ex: edge\_0, edge\_1, ...)

---

**from**

Tipo: string  
Obrigatório: sim  
Regra:  
id do nó de origem (quem depende ou chama)

---

**to**

Tipo: string  
Obrigatório: sim  
Regra:  
id do nó de destino (dependência ou chamado)

---

**type**

Tipo: string  
Obrigatório: sim  
Valores esperados: depends, uses\_copy, calls  
Regra:  
classificar conforme o tipo de relação

---

### 17.5 Regras de Consistência

1. Todos os valores de from e to devem existir em nodes.id  
2. Não duplicar node id  
3. Não duplicar edge id  
4. total\_nodes deve corresponder ao total de nodes  
5. total\_edges deve corresponder ao total de edges  
6. total\_files deve considerar apenas nós reais  
7. Toda referência não resolvida deve gerar um node unresolved  
8. Toda referência unresolved deve possuir um edge associado  
9. source deve ser Base64 válido para artefatos reais

---

### 17.6 Mapeamento a partir da Tabela de Correlação (9.2)

* Cada artefato → um node  
* COPY → edge do tipo uses\_copy  
* INCLUDE → edge do tipo uses\_copy  
* CALL/LINK/XCTL → edge do tipo calls  
* Referências não resolvidas → node unresolved \+ edge associado

---

### 17.7 Exemplo de Saída

{

  "status": "success",

  "total\_files": 5,

  "total\_nodes": 7,

  "total\_edges": 3,

  "nodes": \[

    {

      "id": "/workspace/repo/.pss/a\_old/a\_code/VIPP4553.cbl",

      "label": "VIPP4553.cbl",

      "type": "file",

      "path": "/workspace/repo/.pss/a\_old/a\_code/VIPP4553.cbl",

      "program\_id": "VIPP4553",

      "copies": \["DAFK6452"\],

      "calls": \["D6452"\],

      "includes": \[\],

      "pkgs": \["."\],

      "bms": \[\],

      "jcl": \[\],

      "lines": 196,

      "last\_modified": "2026-03-26T00:00:00",

      "source": "SURFTlRJRklDQVRJT04gRElWSVNJT04uClBST0dSQU0tSUQuIFZJUFA0NTUzLgo="

    },

    {

      "id": "/workspace/repo/.pss/a\_old/a\_code/VIPP4365.cbl",

      "label": "VIPP4365.cbl",

      "type": "file",

      "path": "/workspace/repo/.pss/a\_old/a\_code/VIPP4365.cbl",

      "program\_id": "VIPP4365",

      "copies": \["MCIKCLIE"\],

      "calls": \[\],

      "includes": \[\],

      "pkgs": \["."\],

      "bms": \[\],

      "jcl": \[\],

      "lines": 178,

      "last\_modified": "2026-03-26T00:00:00",

      "source": "SURFTlRJRklDQVRJT04gRElWSVNJT04uClBST0dSQU0tSUQuIFZJUFA0MzY1Lgo="

    },

    {

      "id": "/workspace/repo/.pss/a\_old/a\_code/DAFK6452.cpy",

      "label": "DAFK6452.cpy",

      "type": "cpy",

      "path": "/workspace/repo/.pss/a\_old/a\_code/DAFK6452.cpy",

      "program\_id": null,

      "copies": \[\],

      "calls": \[\],

      "includes": \[\],

      "pkgs": \["."\],

      "bms": \[\],

      "jcl": \[\],

      "lines": 64,

      "last\_modified": "2026-03-26T00:00:00",

      "source": "MDEgRDY0NTJJLUNOUEogUElDIDkoMDE0KS4KMDEgRDY0NTJTLUNOUEogUElDIDkoMDE0KS4K"

    },

    {

      "id": "/workspace/repo/.pss/a\_old/a\_code/VIPK160D.cpy",

      "label": "VIPK160D.cpy",

      "type": "cpy",

      "path": "/workspace/repo/.pss/a\_old/a\_code/VIPK160D.cpy",

      "program\_id": null,

      "copies": \[\],

      "calls": \[\],

      "includes": \[\],

      "pkgs": \["."\],

      "bms": \[\],

      "jcl": \[\],

      "lines": 52,

      "last\_modified": "2026-03-26T00:00:00",

      "source": "MDEgVklQSzE2MEQtUkVHTVRSQU4u"

    },

    {

      "id": "/workspace/repo/.pss/a\_old/a\_code/HLPKDFHE.cpy",

      "label": "HLPKDFHE.cpy",

      "type": "cpy",

      "path": "/workspace/repo/.pss/a\_old/a\_code/HLPKDFHE.cpy",

      "program\_id": null,

      "copies": \[\],

      "calls": \[\],

      "includes": \[\],

      "pkgs": \["."\],

      "bms": \[\],

      "jcl": \[\],

      "lines": 48,

      "last\_modified": "2026-03-26T00:00:00",

      "source": "MDEgRUlCLUNPTU1BUkVBLg=="

    },

    {

      "id": "unresolved::D6452",

      "label": "D6452",

      "type": "unresolved",

      "path": "",

      "program\_id": null,

      "copies": \[\],

      "calls": \[\],

      "includes": \[\],

      "pkgs": \["."\],

      "bms": \[\],

      "jcl": \[\],

      "lines": 0,

      "last\_modified": "",

      "source": ""

    },

    {

      "id": "unresolved::MCIKCLIE",

      "label": "MCIKCLIE",

      "type": "unresolved",

      "path": "",

      "program\_id": null,

      "copies": \[\],

      "calls": \[\],

      "includes": \[\],

      "pkgs": \["."\],

      "bms": \[\],

      "jcl": \[\],

      "lines": 0,

      "last\_modified": "",

      "source": ""

    }

  \],

  "edges": \[

    {

      "id": "edge\_0",

      "from": "/workspace/repo/.pss/a\_old/a\_code/VIPP4553.cbl",

      "to": "/workspace/repo/.pss/a\_old/a\_code/DAFK6452.cpy",

      "type": "uses\_copy"

    },

    {

      "id": "edge\_1",

      "from": "/workspace/repo/.pss/a\_old/a\_code/VIPP4553.cbl",

      "to": "unresolved::D6452",

      "type": "calls"

    },

    {

      "id": "edge\_2",

      "from": "/workspace/repo/.pss/a\_old/a\_code/VIPP4365.cbl",

      "to": "unresolved::MCIKCLIE",

      "type": "uses\_copy"

    }

  \]

}

## 18\. Problemas a serem resolvidos

### 18.1 Violações de Uso de CNPJ por Contexto

Identifica uso incorreto do CNPJ considerando regras do PRD (interno vs externo, entrada vs saída).

**Estrutura:**

| id | artefato | linha | campo | contexto | problema | acao |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |

Legenda:

* id: identificador único  
* artefato: nome do artefato  
* linha: localização no código  
* campo: campo CNPJ envolvido  
* contexto: interno / externo / interface / batch / online  
* problema: descrição objetiva da violação  
* acao: correção necessária

---

### 18.2 Exposição Indevida de CNPJ Técnico (Chave)

Focada exclusivamente em **vazamento de identificador interno**.

**Estrutura:**

| id | artefato | linha | campo | tipo exposicao | destino | problema | acao |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |

Legenda:

* id: identificador único  
* artefato: artefato analisado  
* linha: localização no código  
* campo: campo envolvido  
* tipo exposicao: log / tela / API / artefato  
* destino: consumidor da informação  
* problema: descrição da exposição indevida  
* acao: ajuste necessário

---

### 18.3 Conversões Indevidas de CNPJ

Identifica qualquer tentativa de conversão fora do serviço central.

**Estrutura:**

| id | artefato | linha | origem | destino | tipo conversao | problema | acao |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |

Legenda:

* id: identificador único  
* artefato: artefato  
* linha: localização  
* origem: formato de entrada  
* destino: formato de saída  
* tipo conversao: parsing / casting / transformação manual  
* problema: descrição objetiva  
* acao: correção necessária

---

### 18.4 Geração Indevida de Identificador

Detecta criação local de identificadores (violação grave).

**Estrutura:**

| id | artefato | linha | campo | indicio | problema | acao |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |

Legenda:

* id: identificador único  
* artefato: artefato  
* linha: localização  
* campo: campo envolvido  
* indicio: sequência, cálculo, incremento etc  
* problema: descrição da violação  
* acao: ajuste necessário

---

### 18.5 Inconsistências entre Entrada e Saída

Foco em erros de tratamento entre input/output.

Estrutura:

| id | artefato | linha | fluxo | problema | acao |
| :---: | :---: | :---: | :---: | :---: | :---: |

Legenda:

* id: identificador único  
* artefato: artefato  
* linha: localização  
* fluxo: entrada / saída / ambos  
* problema: inconsistência identificada  
* acao: correção necessária

---

## **19\. Diagnóstico Consolidado**

As tabelas de **Diagnóstico Consolidado** (classificação semântica e estrutural) devem ser obrigatoriamente as **primeiras a serem exibidas na saída do diagnóstico**, precedendo todas as demais tabelas.

Essa priorização é necessária pois essas tabelas fornecem uma **visão macro e executiva do impacto**, permitindo rápida compreensão de:

* volume de ocorrências  
* criticidade funcional (chave, valor, incerto)  
* esforço de refatoração (alterações necessárias)

A partir dessa visão consolidada, as demais tabelas devem ser analisadas de forma complementar e detalhada.

### 19.1 Diagnóstico Consolidado — Classificação Semântica

Esta tabela apresenta a classificação semântica das ocorrências de CNPJ por artefato, permitindo identificar o papel funcional do dado no sistema.

---

**Estrutura**

| id | artefato | quantidade chave | quantidade valor | quantidade incerto |
| :---: | :---: | :---: | :---: | :---: |

---

**Definição dos Campos**

* **id**: identificador único  
* **artefato**: nome do artefato analisado  
* **quantidade chave**:  
  quantidade de ocorrências onde o CNPJ é utilizado como identificador  
* **quantidade valor**:  
  quantidade de ocorrências onde o CNPJ é utilizado como dado  
* **quantidade incerto**:  
  quantidade de ocorrências com classificação indefinida

---

**Regras de Consistência**

* **quantidade chave \+ quantidade valor \+ quantidade incerto \= quantidade variável(tabela estrutural)**  
* Pode haver coexistência entre os três tipos no mesmo artefato  
* Valores elevados de **quantidade incerto** necessidade de revisão manual

---

**Exemplo**

| id | artefato | quantidade chave | quantidade valor | quantidade incerto |
| ----- | ----- | ----- | ----- | ----- |
| 1 | CUSTINQ1.cbl | 4 | 2 | 1 |
| 2 | CUSTDISP.cbl | 0 | 5 | 1 |
| 3 | CUSTTEMP.cbl | 1 | 1 | 3 |

---

### 19.2 Diagnóstico Consolidado — Estrutural e Ações

Esta tabela apresenta a distribuição estrutural das ocorrências e o esforço de refatoração por artefato.

---

**Estrutura**

| id | artefato | quantidade variável | quantidade função | quantidade chamada | quantidade comentário | quantidade ocorrência | quantidade alteração | quantidade sem ação |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |

---

**Definição dos Campos**

* **id**: identificador único  
* **artefato**: nome do artefato analisado

---

**Tipo Estrutural**

* **quantidade variável**:  
  ocorrências em variáveis  
* **quantidade função**:  
  ocorrências em funções ou regras  
* **quantidade chamada**:  
  ocorrências em chamadas de programas/funções  
* **quantidade comentário**:  
  ocorrências em comentários

---

**Métricas Consolidadas**

* **quantidade ocorrência**:  
  total de ocorrências estruturais  
  (qtd\_variavel \+ qtd\_funcao \+ qtd\_chamada \+ qtd\_comentario)  
* **quantidade alteração**:  
  ocorrências que exigem modificação  
* **quantidade sem ação**:  
  ocorrências que não necessitam alteração

---

**Regras de Consistência**

* **quantidade chave \+ quantidade valor \+ quantidade incerto \= quantidade variável(tabela estrutural)**  
*  **quantidade ocorrência \=  quantidade variável \+  quantidade função \+  quantidade chamada \+  quantidade comentário**  
*  **quantidade alteração \+   quantidade sem ação \=  quantidade ocorrência**

---

**Exemplo**

| id | artefato | quantidade variável | quantidade função | quantidade chamada | quantidade comentário | quantidade ocorrência | quantidade alteração | quantidade sem ação |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| 1 | CUSTINQ1.cbl | 7 | 2 | 1 | 1 | 11 | 9 | 2 |
| 2 | CUSTDISP.cbl | 6 | 1 | 0 | 2 | 9 | 6 | 3 |
| 3 | CUSTTEMP.cbl | 5 | 0 | 1 | 0 | 6 | 3 | 3 |

---
