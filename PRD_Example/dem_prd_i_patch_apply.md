# Execução da Modificação (patch) do CNPJ Alfanumérico

## Caminhos do Repositório

Esta seção define o root path do repositório e os caminhos base utilizados ao longo deste documento. Todos os caminhos de artefatos devem ser interpretados a partir do root definido aqui, evitando repetição de paths completos no restante do documento.

| Identificador          | Root Path                                              | Descrição                                                                                                          |
| :--------------------- | :----------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------- |
| `ROOT_DOC_APP_PATH`    | `.pss/b_new/b_doc_b_dem`                             | Documentação geral da demanda, deve ser lida para compreensão da alteração, o ideal é ler a documentação de diagnostico, plano de execução pois são principais e essenciais à essa tarefa.  |
| `ROOT_DOC_DEM_PATH`    | `.pss/b_new/w_output/dem_prd_requisito.md` (timestamp)                             | Documentação específica da demanda de alteração atual do CNPJ de numérico para alfanumérico, tendo o  compromisso de cobrir outros requisitos ou especificações. |
| `ROOT_CODE_OLD`      | `.pss/a_old/a_code`                                  | Código-fonte legado — Contém o código antigo, legado                     |
| `ROOT_CODE_NEW`      | `.pss/b_new/a_code`                                  | Irá contem o código-fonte da modificação (patch) — Contém o código novo (modernizado)                     |


> **Legenda:** **Identificador** = nome simbólico utilizado neste documento para referenciar o caminho; **Root Path** = caminho raiz a partir do repositório (`.` representa a raiz do repositório); **Descrição** = finalidade do caminho.

---

## Objetivo

Esta seção define o objetivo da execução da modificação (patch) dos programas COBOL.

Executar a modificação (patch) dos programas COBOL para adequação do CNPJ numérico (14 dígitos numéricos) para alfanumérico (12 caracteres alfanuméricos + 2 dígitos verificadores numéricos), em ambiente mainframe, preservando os fluxos de negócio existentes e adotando, como princípio estrutural, a separação explícita entre:

- **Chave de CNPJ**: identificador técnico interno
- **Valor de CNPJ**: identificador oficial utilizado pelo negócio

A execução deve garantir que a solução final respeite a coexistência entre formatos, a correlação obrigatória entre chave e valor, e a distinção semântica e técnica entre uso interno e uso de negócio.

---

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

## Princípio Estrutural

Esta seção consolida o princípio arquitetural obrigatório que deve orientar toda a execução da modificação (patch).

A arquitetura deve garantir:

- Desacoplamento entre **Chave de CNPJ** e **Valor de CNPJ**
- Existência de uma correlação **1:1 obrigatória**
- Uso exclusivo da **Chave de CNPJ** em processamento interno
- Uso exclusivo do **Valor de CNPJ** em contexto externo e de negócio
- Proibição de ambiguidade semântica entre identificador técnico e identificador oficial

Esse princípio deve ser respeitado tanto nos programas COBOL quanto em copybooks, layouts, interfaces, artefatos, contratos de dados, integrações, batchs, rotinas online e estruturas de apoio.

---

## Modelo Arquitetural Consolidado

Esta seção descreve os componentes arquiteturais de referência que contextualizam a modificação (patch) no ambiente mainframe.

### Componentes

- Base central de correlação (DFE)
- Serviços de conversão:
  - **Chave de CNPJ → Valor de CNPJ**
  - **Valor de CNPJ → Chave de CNPJ**
- Camadas de cache
- Distribuição de dados para ambientes legados
- Processamento compatível entre online e batch

### Diretriz de uso

- O processamento interno deve preferir a **Chave de CNPJ**
- O contexto externo e funcional deve expor somente o **Valor de CNPJ**
- Conversões entre chave e valor devem ocorrer por meio do serviço oficial
- A correlação não pode ser recriada localmente dentro dos programas da modificação (patch)

---


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

## Requisitos Funcionais

### RF01 — Suportar coexistência de CNPJ numérico e alfanumérico como chave e como valor

Os sistemas devem suportar simultaneamente CNPJ numérico e alfanumérico, distinguindo quando o dado está sendo utilizado como identificador técnico e quando está sendo utilizado como conteúdo de negócio. Isso decorre da exigência oficial de convivência entre os dois formatos em todos os processos que usam a identificação do CNPJ.

**Contrato**
- **Pré-condição:** O sistema recebe, trafega, persiste ou manipula identificador de pessoa jurídica.
- **Pós-condição:** O sistema suporta simultaneamente CNPJ numérico e alfanumérico.
- **Invariante:** Não pode haver ambiguidade entre **Chave de CNPJ** e **Valor de CNPJ**.

**Exemplo**
- Chave = 00000000000001
- Valor = 6QNJ8VY2JIC341

---

### RF02 — Manter a estrutura formal do CNPJ de negócio sem confundir com a chave técnica

O valor de negócio do CNPJ deve obedecer à estrutura oficial de 14 posições, admitindo letras maiúsculas e números, com dígito verificador calculado por módulo 11. A chave técnica interna não deve ser validada como se fosse o CNPJ de negócio, exceto quando ambos coincidirem no cenário numérico.

**Contrato**
- **Pré-condição:** Campo identificado como documento de negócio ou identificador técnico.
- **Pós-condição:** O campo é tratado de acordo com sua natureza semântica.
- **Invariante:** A **Chave de CNPJ** não pode sofrer validação de DV nem formatação de documento oficial.

**Exemplo**
- Chave técnica = 00000000000001
- Valor de negócio = 12ABC34501DE35

---

### RF03 — Preservar o fluxo de negócio atual

A mudança não deve alterar o processo de inscrição, cadastro ou operação de negócio; deve alterar apenas o tratamento do identificador. Em termos funcionais, o sistema continua executando os mesmos fluxos, mas com distinção clara entre a referência técnica e o CNPJ utilizado como conteúdo de negócio.

**Contrato**
- **Pré-condição:** Processo existente utilizando CNPJ.
- **Pós-condição:** O fluxo permanece funcionalmente equivalente.
- **Invariante:** Apenas o tratamento do identificador é alterado.

**Exemplo**
- Tabela relacional grava ID-CNPJ = 00000000000001
- Tela funcional exibe CNPJ = 6QNJ8VY2JIC341

---

### RF04 — Adaptar todos os processos que usam CNPJ

Todo processo que utilize CNPJ deve ser revisado para identificar se o campo está sendo tratado como chave ou como valor. Isso vale para cadastros, consultas, relatórios, emissão documental, obrigações fiscais e integrações. Os sistemas internos, fiscais e contábeis devem estar preparados para ler o formato alfanumérico quando o campo representar valor de negócio.

**Contrato**
- **Pré-condição:** Existência de uso de identificador em qualquer ponto do sistema.
- **Pós-condição:** Cada uso é classificado como **Chave de CNPJ** ou **Valor de CNPJ**.
- **Invariante:** Nenhum campo pode permanecer semanticamente ambíguo.

**Exemplo**
- FK entre tabelas usa 00000000000001
- Relatório fiscal exibe 6QNJ8VY2JIC341

---

### RF05 — Tratar entradas, saídas e consultas de acordo com o papel do campo

Toda interface deve saber se está recebendo uma chave técnica ou um valor de CNPJ. Entradas de valor devem ser validadas conforme a regra oficial do CNPJ; entradas de chave não devem sofrer máscara, cálculo de DV ou normalizações típicas de documento.

**Contrato**
- **Pré-condição:** Interface recebe ou retorna identificador.
- **Pós-condição:** A regra aplicada é compatível com o tipo do campo.
- **Invariante:** **Valor de CNPJ** deve ser validado; **Chave de CNPJ** não.

**Exemplo**
- CNPJ-VALOR = 6QNJ8VY2JIC341 → validar como CNPJ
- ID-CNPJ = 00000000000001 → tratar como identificador interno

---

### RF06 — Uso exclusivo do Valor de CNPJ em interfaces externas

Toda comunicação com usuário, sistema externo, relatório funcional, saída documental, API externa ou integração de negócio deve expor apenas o **Valor de CNPJ**, nunca a **Chave de CNPJ**.

**Contrato**
- **Pré-condição:** Comunicação com entidade externa ou contexto de negócio.
- **Pós-condição:** Apenas o **Valor de CNPJ** é exposto.
- **Invariante:** É proibida a exposição da **Chave de CNPJ**.

**Exemplo**
- API externa retorna: 6QNJ8VY2JIC341
- Nunca retorna: 00000000000001

---

### RF07 — Conversão obrigatória via serviço central

Sempre que houver necessidade de obter a correspondência entre chave e valor, a conversão deve ocorrer via serviço oficial de correlação, e não por geração local ou heurística dentro do programa.

**Contrato**
- **Pré-condição:** Necessidade de converter **Chave de CNPJ** em **Valor de CNPJ** ou vice-versa.
- **Pós-condição:** Conversão realizada por serviço oficial.
- **Invariante:** É proibida a geração local de correlação.

**Exemplo**
- Entrada: Valor de CNPJ = 6QNJ8VY2JIC341
- Saída: Chave de CNPJ = 00000000000001

---

### RF08 — Determinismo e idempotência das conversões

Conversões entre chave e valor devem ser determinísticas e idempotentes, de modo que múltiplas chamadas com o mesmo input retornem sempre o mesmo resultado.

**Contrato**
- **Pré-condição:** Múltiplas chamadas com mesmo identificador de entrada.
- **Pós-condição:** O resultado retornado permanece o mesmo.
- **Invariante:** Não pode haver duplicidade de **Chave de CNPJ** para o mesmo **Valor de CNPJ**, nem divergência na correlação 1:1.

**Exemplo**
- Input repetido: 6QNJ8VY2JIC341
- Output constante: 00000000000001

---

### RF09 — Preservação de identificadores externos (CGC ou Valor de CNPJ)
Contrato

- **Pré-condição:** Identificador recebido de fonte externa (ex.: DB2)
- **Pós-condição:** O valor deve ser preservado exatamente como recebido
- **Invariante:** É proibida qualquer alteração estrutural, incluindo conversão para formato alfanumérico, independentemente de ser CGC ou Valor de CNPJ

**Exemplo** 1 (CNPJ numérico externo)

- **Origem:** DB2MCI.CLIENTE
- **Entrada:** Valor de CNPJ = 12345678000195
- **Saída:** 12345678000195
- **Proibido:** converter para formato alfanumérico

**Exemplo** 2 (CGC externo)

- **Origem:** DB2MCI.CLIENTE
- **Entrada:** CGC = 12345678000195
- **Saída:** 12345678000195
- **Proibido:** Reclassificar, Converter, Normalizar


PS: Campos de origem externa não devem sofrer alteração física, lógica, semântica ou estrutural no escopo deste patch, ainda que representem CNPJ ou CGC.
---

### RF10 — Classificação com base em nível de inferência
Contrato

- **Pré-condição:** Sistema precisa determinar se o identificador é Chave de CNPJ ou Valor de CNPJ
- **Pós-condição:** A classificação só pode ocorrer se houver inferência determinística
- **Invariante:** Inferência com baixa confiança não pode gerar transformação

Regra explícita

Se a inferência for fraca → classificar como Indefinido
Se classificado como Indefinido → nenhuma transformação pode ocorrer

**Exemplo 1 (inferência forte)**

Entrada: 6QNJ8VY2JIC341 | Classificação: Valor de CNPJ | Ação: aplicar validação

**Exemplo 2 (inferência fraca)**

Entrada: 00000000000001 | Contexto insuficiente | Classificação: Indefinido
Ação: Não converter, não validar como CNPJ, preservar valor

---

## Requisitos Técnicos

### RT01 — Redefinir campos físicos e lógicos para separar chave e valor

Todo campo que hoje represente CNPJ como atributo único deve ser reavaliado para suportar, quando necessário, dois conceitos distintos: chave técnica e valor de negócio. Em ambiente mainframe, isso afeta artefatos, registros, layouts, copybooks, áreas de memória e contratos compartilhados.

**Contrato**
- **Pré-condição:** Campo representando CNPJ existente.
- **Pós-condição:** Campo separado em **Chave de CNPJ** e **Valor de CNPJ**, quando aplicável.
- **Invariante:** Tipagem distinta obrigatória.

**Exemplo**
Antes:
- CNPJ PIC 9(14)

Depois:
- ID-CNPJ PIC 9(14)
- CNPJ-VALOR PIC X(14)

---

### RT02 — Revisar copybooks, layouts e contratos de interface

Os contratos de dados devem explicitar quando um campo representa chave relacional e quando representa o valor real do CNPJ. Estruturas compartilhadas entre online, batch e integração devem ser ajustadas para não assumir CNPJ exclusivamente numérico.

**Contrato**
- **Pré-condição:** Existência de layout, copybook ou contrato de dados.
- **Pós-condição:** Cada campo passa a ter semântica explícita.
- **Invariante:** É proibido assumir CNPJ como sempre numérico.

**Exemplo**
```cobol
05 ID-CNPJ      PIC 9(14).
05 CNPJ-NEGOCIO PIC X(14).
```

---

### RT03 — Aplicar cálculo do dígito verificador somente ao valor de negócio

O cálculo oficial do DV deve ser executado apenas sobre o valor de negócio do CNPJ. A chave técnica não deve entrar nessa rotina quando for apenas identificador interno.

**Contrato**
- **Pré-condição:** Execução de validação de CNPJ.
- **Pós-condição:** DV aplicado exclusivamente ao **Valor de CNPJ**.
- **Invariante:** **Chave de CNPJ** nunca entra no cálculo.

**Exemplo**
- Correto: validar 12ABC34501DE35
- Incorreto: validar 00000000000001 como se fosse CNPJ oficial

---

### RT04 — Implementar a regra oficial de DV de forma compatível com mainframe

Como o documento oficial usa referência ASCII para cálculo do DV, a implementação em mainframe deve usar tabela de conversão controlada sobre o valor de negócio, evitando depender diretamente da representação interna de caractere do ambiente.

**Contrato**
- **Pré-condição:** Execução do algoritmo de DV em ambiente mainframe.
- **Pós-condição:** Uso de tabela de conversão controlada.
- **Invariante:** Independência da codificação interna do ambiente.

**Exemplo**
- A pode ser tratado como valor lógico 17 para o cálculo, conforme a regra oficial derivada de 65 - 48.

---

### RT05 — Ajustar validações, máscaras e saneamento por tipo de campo

Rotinas de validação e formatação devem distinguir campo chave de campo valor. O valor de negócio pode exigir validação, normalização e exibição formatada; a chave técnica não deve ser mascarada nem tratada como documento oficial.

**Contrato**
- **Pré-condição:** Aplicação de máscara, saneamento, normalização ou exibição.
- **Pós-condição:** Apenas o **Valor de CNPJ** pode ser formatado como documento.
- **Invariante:** **Chave de CNPJ** permanece bruta.

**Exemplo**
- ID-CNPJ = 00000000000001 → sem máscara
- CNPJ-VALOR = 6QNJ8VY2JIC341 → com validação de estrutura e, se aplicável, máscara de apresentação

---

### RT06 — Revisar pesquisa, comparação, ordenação e indexação

Toda lógica que hoje pesquisa ou ordena CNPJ como número deve ser revista para considerar se a operação ocorre sobre chave ou sobre valor. Busca por identificador interno e busca por CNPJ funcional não são equivalentes e precisam permanecer semanticamente separadas.

**Contrato**
- **Pré-condição:** Operação de busca, comparação, indexação ou ordenação.
- **Pós-condição:** A operação respeita a semântica do campo.
- **Invariante:** Não misturar **Chave de CNPJ** com **Valor de CNPJ**.

**Exemplo**
- Índice por ID-CNPJ = 00000000000001
- Consulta funcional por CNPJ-VALOR = 6QNJ8VY2JIC341

---

### RT07 — Garantir compatibilidade entre online e batch com chave e valor desacoplados

Programas online e batch devem processar corretamente registros em que a chave interna e o valor exibido sejam diferentes. Isso é necessário para que o sistema continue consistente ao suportar o novo modelo alfanumérico sem romper estruturas legadas.

**Contrato**
- **Pré-condição:** Processamento de registros em ambiente online ou batch.
- **Pós-condição:** Suporte ao desacoplamento entre chave e valor.
- **Invariante:** Consistência entre ambientes.

**Exemplo**
- Batch lê ID-CNPJ = 00000000000001
- Recupera CNPJ-VALOR = 6QNJ8VY2JIC341
- Tela exibe somente 6QNJ8VY2JIC341

---

### RT08 — Garantir rastreabilidade entre chave técnica e valor de negócio

A solução deve manter vínculo inequívoco entre a chave usada internamente e o valor real do CNPJ, permitindo consistência transacional, integridade relacional e auditabilidade do dado.

**Contrato**
- **Pré-condição:** Existência de correlação entre identificadores.
- **Pós-condição:** Relação rastreável e auditável.
- **Invariante:** Relação 1:1 preservada.

**Exemplo**
- ID-CNPJ = 00000000000001
- CNPJ-VALOR = 6QNJ8VY2JIC341

---

### RT09 — Centralização da geração de Chave de CNPJ

A criação de nova correlação entre identificador técnico e identificador oficial deve ocorrer apenas no sistema central autorizado. A lógica distribuída de geração de chave é proibida.

**Contrato**
- **Pré-condição:** Necessidade de criação de nova correlação.
- **Pós-condição:** Apenas o sistema central autorizado gera a **Chave de CNPJ**.
- **Invariante:** É proibida geração distribuída.

**Exemplo**
- Sistema A solicita
- DFE gera: 00000000000001

---

### RT10 — Estratégia de cache obrigatória

Quando houver consultas recorrentes ou volume operacional que justifique otimização, a solução deve utilizar camada de cache para reduzir acesso repetitivo ao sistema central de correlação.

**Contrato**
- **Pré-condição:** Consulta recorrente ou necessidade de desempenho.
- **Pós-condição:** Utilização de cache ou mecanismo equivalente.
- **Invariante:** Redução de acesso ao sistema central sem quebrar consistência.

**Exemplo**
- Primeira consulta: DFE
- Próximas: cache / VSAM / estruturas equivalentes

---

### RT11 — Distribuição batch de dados

Ambientes mainframe dependentes da correlação entre chave e valor podem receber carga periódica de dados, desde que mantida consistência com a base central.

**Contrato**
- **Pré-condição:** Ambiente legado dependente de sincronização periódica.
- **Pós-condição:** Recebe carga batch compatível com a base central.
- **Invariante:** Consistência com a correlação oficial.

**Exemplo**
- artefato batch contém:
  - Chave de CNPJ
  - Valor de CNPJ

---

### RT12 — Proibição de exposição da Chave de CNPJ

A **Chave de CNPJ** não deve ser exibida para usuário, interface externa, relatório funcional, API pública, integração de negócio ou qualquer saída não estritamente técnica e interna.

**Contrato**
- **Pré-condição:** Saída para usuário ou sistema externo.
- **Pós-condição:** A **Chave de CNPJ** não é exposta.
- **Invariante:** Violação é erro crítico.

**Exemplo**
- Permitido: 6QNJ8VY2JIC341
- Proibido: 00000000000001

---

### RT13 — Bloqueio de transformação para dados externos
Contrato

- **Pré-condição:** Campo identificado como origem externa
- **Pós-condição:** Pipeline de transformação deve ser bypassado
- **Invariante:** Proibida conversão para formato alfanumérico ou qualquer modificação

**Exemplo**
- Entrada: 12345678000195 | Origem: DB2
- Não converter. Persistir como está.

---

### RT14 — Controle de inferência e estado "Indefinido"

**Contrato**
- **Pré-condição:** Processo de classificação executado
- **Pós-condição:** Deve existir estado explícito para baixa confiança
- **Invariante:** Estado "Indefinido" bloqueia qualquer transformação

**Regra técnica**
- **Inferência determinística → permitido classificar**
- **Inferência não determinística → classificar como Indefinido**

**Exemplo**
- Entrada: 00000000000001 | Classificação: INDEFINIDO
- Pipeline: não validar DV, não converter, não gerar Chave de CNPJ, manter valor

---

## Critérios Macro de Aceitação

Esta seção consolida os critérios para considerar a solução aderente.

A solução será considerada aderente quando:

- o sistema aceitar CNPJ numérico e alfanumérico em processos que identifiquem pessoa jurídica;
- o valor de negócio respeitar a estrutura oficial de 14 posições com DV por módulo 11;
- a chave técnica puder ser mantida separada do valor de negócio quando necessário;
- interfaces, artefatos e programas distingam corretamente chave e valor;
- o cálculo do DV seja aplicado apenas ao CNPJ de negócio;
- online e batch operem sem assumir que chave e valor são sempre iguais;
- a solução suportar **Valor de CNPJ** numérico e alfanumérico;
- a solução garantir separação total entre **Chave de CNPJ** e **Valor de CNPJ**;
- a solução não expor **Chave de CNPJ** externamente;
- a solução garantir determinismo das conversões;
- a solução manter integridade entre batch e online;
- a solução operar com mecanismo de cache eficiente, quando aplicável;
- a solução não gerar regressão indevida em sistemas legados;
- a correlação 1:1 entre chave e valor permanecer íntegra, rastreável e auditável.
- campos CNPJ ou CGC provenientes de fontes externas (ex: tabelas DB2 de outros domínios) não são alterados pela modificação (patch);
- campos com taxa de inferência fraca são classificados como Indefinido e não sofrem nenhuma alteração.

---

## Premissas

Esta seção apresenta as premissas técnicas e de negócio que norteiam a execução da modificação (patch).

- **2 Casos de uso**: Para realizar a mudança do CNPJ de numérico para alfanumérico, teremos 2 casos de uso, conforme definido na seção "Definições" deste documento:

  **Caso de uso 1**: Cenário onde o CNPJ é numérico, onde a chave e o valor sempre serão iguais, mas o campo é utilizado apenas como identificador (chave estrangeira, FK) e não manipulado como valor de negócio.
  Chave = 78465270000165
  Valor = 78465270000165

  **Caso de uso 2**: Cenário onde o CNPJ é alfanumérico, onde a chave sempre vai ser diferente do valor, e o campo é utilizado apenas como identificador (chave estrangeira, FK) e não manipulado como valor de negócio.
  Chave = 00000000000001
  Valor = 6QNJ8VY2JIC341

- **Separação obrigatória**: Toda análise e execução devem assumir como princípio obrigatório o desacoplamento entre **Chave de CNPJ** e **Valor de CNPJ**, com correlação 1:1.

- **Uso semântico obrigatório**:
  - **Chave de CNPJ** deve ser usada em processamento interno e relacional.
  - **Valor de CNPJ** deve ser usado em contexto externo, funcional e de negócio.

- **Correlação oficial**: A correlação entre chave e valor deve ser tratada como oficial, única, determinística e auditável.

- **Conversão oficial**: Sempre que houver necessidade de conversão entre chave e valor, deve-se assumir o uso de um serviço central oficial. Não se deve inferir, calcular ou gerar a correlação localmente dentro dos programas modificados (patch).

- **Validação de Rotina**: Assumimos que não faremos a validação da rotina de obtenção/validação do CNPJ nesta PoC.

- **Módulo principal**: Todos os artefatos do módulo principal que precisam ser modificados, SERÃO MODIFICADOS.

- **Módulo secundário**: Todos os artefatos do módulo secundário, mesmo que precisem ser modificados, NÃO SERÃO MODIFICADOS.

- **Principal sem alteração**: Um módulo pode ser principal ao sistema e mesmo assim **não requerer alteração** (ex: book de consulta a tabela de tipos que não contém campos CNPJ impactados, ou book somente leitura). A ausência de alteração **não implica** que o módulo é secundário.

- **Critério prático**: Verificar o prefixo do artefato (primeiros 3 caracteres do nome). Se coincide com o prefixo dos programas principais → **principal**. Se pertence a outro sistema → **secundário**.

- **Módulos secundários**: É proibida qualquer alteração em copybooks/módulos de sistemas **secundários** (identificados dinamicamente pelo critério de prefixo acima). Esses módulos pertencem a outros sistemas e não estão no escopo desta demanda. Não se pode pegar a Chave nem o Valor dos módulos que não sejam os principais.

- **Campos de Módulos secundários**: Campos de CNPJ pertencentes a módulos secundários NÃO devem ser contabilizados nesta tarefa de modificação de CNPJ numérico para alfanumérico como campos Chave ou Valor na análise. Somente campos dos programas principais e seus copybooks **principais** (mesmo prefixo) são considerados. **Se qualquer campo de módulo secundário aparecer na lista de campos impactados/alterados, isso constitui um erro que deve ser corrigido.**

- **Funções de Obtenção/Validação do CNPJ**: Não utilizar nomes reais de funções ou sistemas. Utilizar nomenclatura genérica:
  - Plataforma Baixa: "Função de Obtenção/Validação do CNPJ em Plataforma Baixa"
  - Plataforma Alta: "Função de Obtenção/Validação do CNPJ em Plataforma Alta"

- **Dados de fontes externas**: Campos CNPJ ou CGC provenientes de fontes externas (ex: tabelas DB2 como DB2MCI.CLIENTE) NÃO devem ser alterados, independentemente de sua natureza aparente (Chave ou Valor). A modificação (patch) aplica-se exclusivamente a campos cujos dados sejam originados e gerenciados pelo próprio escopo dos programas principais.

- **Taxa de inferência fraca**: Quando não for possível determinar com segurança se um campo é Chave, Valor ou Fonte Externa, o campo deve ser classificado como Indefinido e não deve ser alterado. A dúvida não autoriza a modificação; na ausência de certeza, a postura conservadora é obrigatória.

**Conclusão**: As premissas estabelecem as regras fundamentais para a execução: dois cenários (Chave e Valor), separação estrutural obrigatória entre chave e valor, correlação 1:1, classificação dinâmica de módulos por prefixo (principal vs secundário), proibição de alteração em módulos secundários, uso semântico correto dos identificadores e nomenclatura genérica para funções de obtenção/validação.

---

### Premissa Técnica — Não Criação de Nova Versão de Artefatos (V2)

Nesta demanda, **não será adotada estratégia de versionamento funcional de artefatos**, tais como criação de programas, copybooks, layouts, JCLs, PROCedures, mapas ou componentes correlatos com sufixação de versão lógica ou física, como `V2`, `NEW`, `_N`, `_ALT` ou equivalentes. A atuação deverá ocorrer **sobre os artefatos correntes já homologados na cadeia operacional da plataforma alta**, por meio de **ajustes pontuais e controlados**, preservando nomenclatura, identidade física, contratos de execução, encadeamentos batch, referências cruzadas, catálogo de carga e vínculos operacionais já existentes no ambiente mainframe.

Essa premissa visa evitar **duplicidade de objetos**, **desvio de roteamento**, **quebra de dependências estáticas**, **aumento desnecessário da superfície de manutenção** e **complexidade adicional de promoção entre ambientes**, assegurando que a solução permaneça integrada ao ciclo normal de compilação, linkedição, implantação e operação produtiva, sem introdução de uma trilha paralela de artefatos versionados.


---

## Direcionamento Estratégico

Esta seção consolida os direcionadores estratégicos que contextualizam a execução técnica.

A solução estabelece:

- Desacoplamento estrutural
- Centralização da lógica de correlação
- Compatibilidade retroativa com estruturas legadas
- Separação entre processamento interno e exposição funcional
- Evolução progressiva do modelo sem regressão operacional indevida

### Evolução futura

- Persistência nativa baseada apenas em **Valor de CNPJ**
- Eliminação progressiva da **Chave de CNPJ**

Esses itens orientam o desenho da solução, mas não alteram o escopo pontual desta execução.

---

## Instruções Gerais de Qualidade Documental

Esta seção define os padrões de qualidade obrigatórios para toda a documentação gerada durante a execução.

- **Idioma**: Toda a documentação deve ser redigida em português (Brasil). Termos técnicos em inglês são permitidos apenas quando forem termos consagrados (ex: COBOL, DB2, COMP-3, PIC, FILLER, COMMAREA, etc.).
- **Seções**: Cada seção do documento gerado deve iniciar com um parágrafo curto explicando o que a seção aborda.
- **Conclusão**: Cada módulo, tarefa ou seção principal deve terminar com uma conclusão breve.
- **Tabelas**: Toda tabela deve conter, imediatamente após ela, uma legenda explicando/descrevendo cada coluna.
- **Resumo Executivo**: Para esta documentação, NÃO produzir seção de Resumo Executivo.

## Regras Complementares de Qualidade do Relatório Final

Além das regras gerais de qualidade documental, o relatório final da execução deve obedecer obrigatoriamente às regras abaixo:

- **Rastreabilidade obrigatória**: toda conclusão deve estar vinculada a evidências observáveis no código, nos artefatos de discovery/diagnostic ou na documentação lida.
- **Sem inferência genérica sem evidência**: o relatório não pode afirmar que um campo é Chave, Valor, Externo ou Indefinido sem apresentar justificativa técnica.
- **Sem sucesso automático**: o relatório não deve declarar “sucesso” por padrão. O status final deve refletir o resultado real da execução, inclusive quando houver pendências, campos indefinidos, artefatos não alterados ou limitações de escopo.
- **Sem valores fictícios**: valores, quantidades, durações, impactos, percentuais e estatísticas devem refletir a execução real. Exemplos ilustrativos não devem aparecer no relatório final como se fossem resultado real.
- **Legenda obrigatória**: toda tabela deve ser seguida de uma legenda explicando o significado de cada coluna.
- **Threshold / Critério de Avaliação obrigatório**: toda tabela deve ser seguida de um bloco chamado `Threshold / Critério de Avaliação`, explicando as métricas, limiares, critérios de status, critérios de inferência, critérios de impacto ou critérios de classificação utilizados naquele contexto.
- **Consumidor subsequente obrigatório quando aplicável**: sempre que um campo for classificado como **Valor** em contexto de saída, detalhe, header, trailer, interface, arquivo, buffer de output, COMMAREA, integração, mensagem ou qualquer cadeia de propagação de dado de negócio, o relatório deve explicitar o consumidor subsequente. Se não for aplicável, o campo deve conter `N/A` com justificativa.
- **Diferenciação entre alterado e não alterado**: o relatório deve documentar tanto o que foi alterado quanto o que foi analisado e deliberadamente não alterado, com justificativa técnica.
- **Aderência ao PRD**: o relatório deve demonstrar aderência aos requisitos funcionais, técnicos, critérios macro de aceitação e regras de execução da LLM.
---

## Regras de Tamanho de Registro

Esta seção define as regras obrigatórias para ajuste de tamanho de registro quando variáveis internas são alteradas durante a modificação (patch).

### Regra Geral

Quando variáveis dentro de um registro COBOL são alteradas (ex: `PIC S9(n) COMP-3` → `PIC X(n)`), o tamanho total do registro (`FD RECORD`, variável `REG`, e registro pai como `REG-GERAL`) **DEVE** ser recalculado e atualizado proporcionalmente ao delta de bytes.

### Regra para Campos Valor

Quando o campo é **Valor**, o tipo é alterado (ex: `PIC S9(014) COMP-3` → `PIC X(014)`) e isso impacta o tamanho do registro. O delta deve ser calculado e refletido no `FD RECORD` e nas variáveis de registro associadas.

### Regra para Campos Chave

Em regra, quando o campo é classificado exclusivamente como Chave técnica interna, não deve haver alteração do tamanho da variável nem do registro. Exceções somente são permitidas quando houver evidência técnica explícita de contrato, layout compartilhado ou requisito estrutural que imponha a alteração.

### Cálculo de Bytes por Tipo COBOL

| Tipo COBOL                          | Fórmula de Tamanho (bytes) | Exemplo                        |
| :---------------------------------- | :------------------------- | :----------------------------- |
| `PIC 9(n)` (display)                | n                          | `PIC 9(014)` = 14 bytes        |
| `PIC S9(n) COMP-3` (packed decimal) | TRUNC((n+1)/2)             | `PIC S9(014) COMP-3` = 8 bytes |
| `PIC X(n)` (alfanumérico)           | n                          | `PIC X(014)` = 14 bytes        |

> **Legenda:** **Tipo COBOL** = declaração PIC utilizada no código-fonte; **Fórmula de Tamanho** = cálculo do tamanho em bytes conforme o tipo; **Exemplo** = aplicação prática da fórmula.

### Tabela de Impacto de Tamanho de Registro

O resultado da análise de impacto de tamanho deve ser consolidado em uma tabela com o seguinte formato obrigatório, a ser preenchida durante a execução:

| Nome artefato | Entrada Legada (byte) | Entrada Nova (byte) | Delta Entrada (byte) | Saída Legada (byte) | Saída Nova (byte) | Delta Saída (byte) |
| :----------- | --------------------: | ------------------: | -------------------: | ------------------: | ----------------: | -----------------: |
| _(nome do artefato .cbl ou .cpy)_ | _(tamanho da entrada antes da modificação)_ | _(tamanho da entrada após a modificação)_ | _(entrada nova − entrada legada)_ | _(tamanho da saída antes da modificação)_ | _(tamanho da saída após a modificação)_ | _(saída nova − saída legada)_ |

> **Legenda:** **Nome artefato** = nome do artefato COBOL (.cbl ou .cpy) analisado; **Entrada Legada (byte)** = tamanho em bytes do registro de entrada antes da modificação (patch); **Entrada Nova (byte)** = tamanho em bytes do registro de entrada após a modificação (patch); **Delta Entrada (byte)** = diferença entre entrada nova e entrada legada (positivo = crescimento, zero = sem impacto, negativo = redução); **Saída Legada (byte)** = tamanho em bytes do registro de saída antes da modificação (patch); **Saída Nova (byte)** = tamanho em bytes do registro de saída após a modificação (patch); **Delta Saída (byte)** = diferença entre saída nova e saída legada.

**Exemplo ilustrativo** *(valores fictícios para demonstrar o preenchimento — não representam dados reais da execução)*:

| Nome artefato | Entrada Legada (byte) | Entrada Nova (byte) | Delta Entrada (byte) | Saída Legada (byte) | Saída Nova (byte) | Delta Saída (byte) |
| :----------- | --------------------: | ------------------: | -------------------: | ------------------: | ----------------: | -----------------: |
| VIPP4553.cbl |                   454 |                 478 |                  +24 |                 320 |               320 |                  0 |
| VIPP4365.cbl |                   358 |                 358 |                    0 |                 358 |               358 |                  0 |

> **Legenda:** Mesma legenda da tabela de formato acima. Os valores apresentados são **exemplos ilustrativos** e não devem ser utilizados como referência de resultado da execução real.

**Conclusão**: As regras de tamanho de registro garantem que, ao alterar campos **Valor**, o `FD RECORD` e variáveis associadas sejam recalculados e refletidos na tabela de impacto. Campos **Chave** não sofrem impacto de tamanho. A tabela de impacto deve ser preenchida com os valores reais apurados durante a execução.

---


## Estrutura Obrigatória do Relatório Final da Execução

O relatório `dem_i_patch_apply.md` deve conter obrigatoriamente as seções abaixo, nesta ordem lógica, podendo incluir subseções adicionais quando necessário para melhorar a rastreabilidade técnica.

Toda subseção do relatório que contenha consolidação analítica deve apresentar, quando aplicável, tabela, legenda e bloco Threshold / Critério de Avaliação.

Não pode conter CAMINHOS DE ARQUIVOS, VALORES FICTÍCIOS, STATUS AUTOMÁTICO ou CONCLUSÕES SEM EVIDÊNCIA!

Se for fazer alguma inferência, deverá ter justificativa técnica e evidência clara. Se não tiver certeza, classificar como Indefinido e não alterar.

Essa estrutura é obrigatória para garantir a qualidade, rastreabilidade e aderência do relatório final aos requisitos deste PRD. A ausência de qualquer seção ou subseção obrigatória, ou a inclusão de informações sem evidência, constitui um erro que deve ser corrigido antes da conclusão da execução.

### 0. Visão Consolidada da Execução

Deve apresentar uma visão sintética e quantitativa da execução, consolidando no mínimo:

- Arquivos analisados;
- Arquivos modificados;
- Arquivos não modificados;
- Campos classificados como Chave;
- Campos classificados como Valor;
- Campos classificados como Indefinido;
- Campos Chave alterados;
- Campos Valor alterados;
- Módulos secundários respeitados / não alterados;
- Quantidade de artefatos lidos;
- Quantidade de artefatos gerados;
- Duração da execução;
- Status final.

A seção deve usar tabela de métricas consolidadas.

### 1. Documentação de Leitura

Deve listar todos os artefatos efetivamente lidos na execução, incluindo documentação funcional, técnica, discovery, diagnostic e código analisado.

A tabela desta seção deve conter, no mínimo:
- `Artefato`
- `Tipo`
- `Status`
- `Observação`

### 2. Análise de Contexto

#### 2.1 Classificação de Módulos

Deve demonstrar a classificação dinâmica dos artefatos com base no prefixo e no escopo do sistema principal, distinguindo:
- módulo principal;
- módulo secundário;
- principal sem alteração;
- secundário fora de escopo.

A tabela deve conter, no mínimo:
- `Arquivo`
- `Módulo`
- `Prefixo`
- `É Principal`
- `Ação`
- `Justificativa`

#### 2.2 Inferência de Classificação (Chave vs Valor vs Indefinido)

Deve documentar cada campo de CNPJ/CGC analisado, registrando a classificação inferida e sua justificativa técnica.

A tabela deve conter, no mínimo:
- `Artefato`
- `Campo`
- `Tipo Original`
- `Classificação`
- `Taxa de Inferência`
- `Evidências Utilizadas`
- `Consumidor Subsequente`
- `Justificativa`
- `Decisão`

Regras obrigatórias:
- `Consumidor Subsequente` é obrigatório quando o campo estiver em contexto de saída, propagação, interface ou dado de negócio downstream;
- quando não aplicável, deve ser preenchido com `N/A` e justificativa;
- `Decisão` deve indicar se o campo foi `Alterado`, `Não Alterado`, `Fora de Escopo` ou `Indefinido`.
- `Threshold de Inferência` abaixo da tabela, deve ser preenchido com o percentual real apurado durante a execução, e a classificação deve seguir os critérios definidos neste PRD (Chave, Valor, Indefinido).


### 3. Modificações Executadas

Deve detalhar, por artefato, as modificações efetivamente aplicadas.

#### 3.1 Alterações em Nível de Campo

Para cada artefato modificado, deve existir uma tabela contendo, no mínimo:
- `Linha`
- `Campo`
- `Antes`
- `Depois`
- `Classificação`
- `Consumidor Subsequente`
- `Motivo`
- `Status`

- `Threshold de Inferência` abaixo da tabela, deve ser preenchido com o percentual real apurado durante a execução, e a classificação deve seguir os critérios definidos neste PRD (Chave, Valor, Indefinido).

#### 3.2 Campos Avaliados e Não Alterados

Deve existir subseção específica para campos analisados, porém não alterados, com justificativa técnica.

A tabela deve conter, no mínimo:
- `Linha`
- `Campo`
- `Classificação`
- `Motivo da Não Alteração`
- `Status`

- `Threshold de Inferência` abaixo da tabela, deve ser preenchido com o percentual real apurado durante a execução, e a classificação deve seguir os critérios definidos neste PRD (Chave, Valor, Indefinido).

#### 3.3 Operações, MOVEs, regras e trechos preservados

Deve existir subseção específica para operações de transferência, regras, validações, rotinas e trechos de código que permaneceram inalterados por decisão técnica.

A subseção deve explicitar:
- o trecho preservado;
- a justificativa da preservação;
- se a preservação decorre de escopo, semântica de Chave, dado externo, ausência de impacto ou baixa inferência.

### 4. Análise de Impacto de Tamanho de Registro

Deve consolidar o impacto de tamanho em entradas, saídas, layouts e registros COBOL, com base nas fórmulas definidas neste PRD.

#### 4.1 Regras Aplicadas
Deve reproduzir as fórmulas de cálculo utilizadas.

#### 4.2 Tabela de Impacto de Tamanho de Registro
A tabela deve conter, no mínimo:
- `Nome Arquivo`
- `Entrada Legada (byte)`
- `Entrada Nova (byte)`
- `Delta Entrada (byte)`
- `Saída Legada (byte)`
- `Saída Nova (byte)`
- `Delta Saída (byte)`

#### 4.3 Cálculo Detalhado
Deve detalhar, por artefato, como cada delta foi obtido.

#### 4.4 Conclusão do Impacto
Deve consolidar o impacto total e o status de compatibilidade com processamento mainframe.

### 5. Verificação de Conformidade com Regras do PRD

Deve conter checklists objetivos de aderência à especificação.

#### 5.1 Checklist de Requisitos Funcionais (RF)
Tabela mínima:
- `RF`
- `Requisito`
- `Status`
- `Observação`
- `Evidência`

#### 5.2 Checklist de Requisitos Técnicos (RT)
Tabela mínima:
- `RT`
- `Requisito`
- `Status`
- `Observação`
- `Evidência`

#### 5.3 Critérios Macro de Aceitação
Tabela mínima:
- `Critério`
- `Status`
- `Evidência`

### 6. Verificação de Conformidade com as Regras de Execução para a LLM

Deve demonstrar aderência às regras de intervenção mínima, escopo restrito, não alteração de módulos secundários, não alteração de fonte externa e preservação de campos indefinidos.

Tabela mínima:
- `Regra`
- `Status`
- `Observação`
- `Evidência`

### 7. Artefatos de Saída Gerados

Deve listar todos os artefatos gerados como resultado da execução.

Tabela mínima:
- `Artefato`
- `Caminho`
- `Tipo`
- `Status`

### 8. Resumo de Mudanças por Arquivo

Deve consolidar, por artefato modificado, um resumo executivo técnico contendo no mínimo:
- tipo de modificação;
- quantidade de campos alterados;
- quantidade de campos Chave alterados;
- quantidade de campos Valor alterados;
- linhas modificadas;
- impacto de entrada;
- impacto de saída;
- nível de risco;
- observações.

### 9. Conclusão

Deve consolidar:
- resultado da execução;
- escopo efetivamente atendido;
- principais decisões de classificação;
- conformidade com o PRD;
- limitações, se houver;
- prontidão para próximas etapas (compilação, teste, homologação, etc.), sem afirmar etapas não executadas como concluídas.

### 10. Metadados de Execução

Deve apresentar métricas adicionais de execução, incluindo no mínimo:
- documentação lida;
- código analisado;
- campos alterados;
- linhas modificadas;
- módulos secundários respeitados;
- taxa média de inferência;
- arquivos gerados;
- conformidade RF;
- conformidade RT;
- duração;
- data de execução;
- executor;
- status final.

## Thresholds e Critérios Padronizados do Relatório

O relatório final deve utilizar thresholds e métricas padronizadas para garantir consistência entre tabelas, análises e conclusões.

### Threshold de inferência semântica

A classificação de campos como **Chave**, **Valor** ou **Indefinido** deve utilizar os seguintes limiares:

- **Inferência Forte (>= 90%)**: classificação permitida com evidência suficiente para decisão técnica;
- **Inferência Moderada (>= 75% e < 90%)**: classificação somente permitida se houver pelo menos 2 evidências convergentes e ausência de contradição material;
- **Inferência Fraca (< 75%)**: o campo deve ser classificado como **Indefinido** e não pode ser alterado.

### Critérios mínimos de evidência para inferência

A taxa de inferência deve considerar, quando aplicável:
- nome do campo;
- nome do registro;
- nome do layout;
- contexto de entrada/saída;
- ocorrência em `MOVE`;
- gravação em arquivo;
- passagem por parâmetro;
- ocorrência em COMMAREA;
- uso em tela, relatório ou interface;
- origem em fonte externa;
- participação em busca, chave, índice ou relacionamento;
- sufixos semânticos (`-DET`, `-HDR`, `-OUT`, `-RET`, etc.);
- evidências de consumidor subsequente downstream.

### Threshold de risco técnico por artefato

- **BAIXO**: alteração pontual, sem mudança de lógica, sem contrato compartilhado alterado e sem impacto downstream material identificado;
- **MÉDIO**: alteração em layout, record, copybook compartilhado ou artefato com propagação downstream controlada;
- **ALTO**: alteração com impacto em contrato externo, múltiplos consumidores subsequentes, dependência crítica ou risco de regressão estrutural.

### Threshold de status das verificações

- `✅ Atendido`: evidência objetiva de aderência;
- `⏺️ Parcial / N/A`: requisito parcialmente aplicável, não aplicável ou não verificável nesta execução;
- `❌ Não Atendido`: divergência, violação ou ausência de evidência suficiente.

### Regra obrigatória de uso dos thresholds nas tabelas

Toda tabela do relatório deve ser seguida de:
1. **Legenda**, explicando cada coluna;
2. **Threshold / Critério de Avaliação**, explicando as métricas efetivamente utilizadas naquele contexto.

Não é permitido omitir o bloco de threshold, mesmo em tabelas predominantemente descritivas. Nesses casos, o bloco deve explicar o critério de preenchimento, decisão ou classificação adotado.

### Regra Mandatória de Documentação de Consumidor Subsequente no Relatório Final

Sempre que o relatório classificar um campo como **Valor** em contexto de saída, detalhe, header, trailer, interface, arquivo, buffer, COMMAREA, área de comunicação, integração, mensagem, layout de remessa/retorno ou qualquer outro contexto de propagação de dado de negócio, a tabela correspondente deve conter obrigatoriamente a coluna:

- `Consumidor Subsequente`

Essa coluna deve registrar de forma nominativa e técnica:
- o artefato consumidor;
- a natureza do consumo;
- o contrato técnico ou meio de propagação;
- o impacto potencial da mudança de formato.

Quando não houver aplicabilidade, a coluna deve conter `N/A`, acompanhada de justificativa explícita.

---

## Artefatos de Entrada

Esta seção identifica os artefatos de entrada necessários para a execução. Os caminhos utilizam os root paths definidos na seção "Caminhos do Repositório".
| Identificador       | Root Path                                            | Descrição                                                                 |
| :------------------ | :--------------------------------------------------- | :------------------------------------------------------------------------ |
| `ROOT_CODE_OLD`     | `.pss/a_old/a_code`                                  | Código-fonte legado (baseline de comparação).                             |
| `ROOT_CODE_PATCH`   | `.pss/b_new/a_code`                                  | Código-fonte novo contendo exclusivamente os artefatos modificados.       |
| `ROOT_DOC_APP_PATH` | `.pss/b_new/b_doc_b_dem`                             | Documentação geral da demanda.                                            |
| `ROOT_DOC_DEM_PATH` | `.pss/b_new/w_output/dem_prd_requisito.md` | Documentação específica da demanda atual para leitura e entendimento do case.                                 |
| `ROOT_REPORT_PATH`  | `.pss/b_new/b_doc_b_dem/dem_i_patch_apply.md`        | Relatório final consolidado da execução da modificação (patch).           |
---

## Artefatos de Saída

Esta seção define o destino e as regras para entrega dos artefatos gerados pela execução.

### Código modificado (patch)

Root: `ROOT_CODE_PATCH`

**Regras de entrega:**
- Gere somente os artefatos de código modificados (patch).
- Não gere nenhum `.md` com logs ou sumário — apenas os códigos na pasta referenciada por `ROOT_CODE_PATCH`.

### Relatório de aplicação

Root: `ROOT_REPORT_FILE` — artefato único de relatório consolidado da execução da modificação (patch). 

---

## Output

Ao final da execução, deve ser gerado obrigatoriamente um relatório consolidado em formato Markdown contendo o detalhamento técnico da execução deste PRD, incluindo os artefatos lidos, os artefatos modificados, as decisões de alteração e não alteração, as evidências utilizadas, os impactos apurados e o status final da execução.

**Nome obrigatório do relatório:** `dem_i_patch_apply.md`  
**Path obrigatório do artefato de output:** `.pss/b_new/b_doc_b_dem/dem_i_patch_apply.md`

A execução somente será considerada concluída quando:

1. os artefatos de código modificados tiverem sido gerados no diretório de saída de código definido neste PRD; e  
2. o relatório final `dem_i_patch_apply.md` tiver sido gerado no path obrigatório acima.

### Schema mínimo obrigatório do relatório

| Coluna           | Tipo       | Descrição |
| :--------------- | :--------- | :-------- |
| `CardNm`         | `string`   | Nome ou identificador funcional do card/Jira associado à execução |
| `TaskNm`         | `string`   | Nome do PRD ou tarefa de entrada executada |
| `ExecutionDt`    | `string`   | Data e hora reais da execução |
| `Duration (min)` | `number`   | Duração real da execução em minutos |
| `Input`          | `string[]` | Lista dos artefatos efetivamente lidos durante a execução |
| `CodeOutput`     | `string[]` | Lista dos artefatos de código gerados ou modificados |
| `ReportOutput`   | `string`   | Nome do artefato Markdown de relatório gerado |
| `Executor`       | `string`   | Executor responsável pela geração do output |
| `FinalStatus`    | `string`   | Status final consolidado da execução |

**Legenda**:  
**CardNm** = identificador funcional da demanda;  
**TaskNm** = documento de entrada efetivamente executado;  
**ExecutionDt** = timestamp real da execução;  
**Duration (min)** = duração real apurada, vedado uso de estimativa fictícia;  
**Input** = artefatos efetivamente lidos na execução;  
**CodeOutput** = artefatos de código gerados, alterados ou modernizados;  
**ReportOutput** = nome do relatório final consolidado;  
**Executor** = agente, LLM ou executor responsável pela geração do output;  
**FinalStatus** = resultado consolidado da execução.

**Threshold / Critério de Avaliação**:  
- `Sucesso` = execução concluída, relatório gerado, artefatos obrigatórios gerados e sem violação material deste PRD;  
- `Sucesso Parcial` = execução concluída com restrições, pendências, campos indefinidos, limitações de escopo ou itens não aplicáveis relevantes devidamente documentados;  
- `Não Concluído` = ausência de artefatos obrigatórios, falha material de aderência ao PRD, inconsistência impeditiva ou impossibilidade de concluir a execução.

### Regras obrigatórias do output

- O relatório final deve refletir exclusivamente os resultados reais da execução, sendo proibido o uso de métricas fictícias, valores exemplificativos apresentados como reais ou conclusões sem evidência.
- O campo `Input` deve conter apenas artefatos efetivamente lidos.
- O campo `CodeOutput` deve conter apenas artefatos efetivamente gerados ou modificados.
- O campo `FinalStatus` deve ser coerente com o conteúdo do relatório e com os artefatos efetivamente produzidos.
- O output final deve ser consistente com a seção **Estrutura Obrigatória do Relatório Final da Execução** e com as **Regras Complementares de Qualidade do Relatório Final** definidas neste PRD.

---

## Instruções de Execução

Esta seção descreve o passo a passo que deve ser seguido para executar a modificação (patch) dos programas COBOL.

1. Leia toda a documentação disponível na pasta de documentação.
2. Com base na documentação, aplique as modificações (patch) nos artefatos COBOL seguindo as premissas acima.
3. **NÃO altere** módulos **secundários** (identificados pelo critério de prefixo: artefatos cujo prefixo pertence a outro sistema, diferente dos programas principais).
4. **NÃO altere** módulos **principais sem alteração** (mesmo prefixo dos programas principais, mas somente leitura ou sem campos CNPJ no escopo). A ausência de alteração se deve à natureza funcional, não por serem de outro sistema.
5. **NÃO extraia** campos Chave ou Valor de módulos que não sejam os principais (VIPP4365, VIPP4553).
6. Sempre que houver dependência de correlação entre **Chave de CNPJ** e **Valor de CNPJ**, assuma o uso de serviço central oficial e **não implemente geração local de correlação**.
7. Em qualquer interface externa, relatório funcional, tela, saída de negócio ou integração não estritamente técnica, **não exponha a Chave de CNPJ**.
8. Aplique apenas o mínimo necessário para atender ao escopo da demanda relacionado ao CNPJ.
9. Verifique **todos os artefatos** da pasta de código legado e aplique a modificação (patch) em **todos** os que necessitem de alteração.
10. NÃO altere campos CNPJ ou CGC cujos dados sejam provenientes de fontes externas (ex: leituras de tabelas DB2 de outros domínios, como DB2MCI.CLIENTE). Esses campos são somente leitura para esta modificação (patch).
11. Quando a natureza de um campo não puder ser determinada com segurança (taxa de inferência fraca), classifique-o como Indefinido no relatório e não aplique nenhuma alteração. Na dúvida, não altere.
12. Ao final da execução, gere obrigatoriamente o relatório consolidado `dem_i_patch_apply.md` em `ROOT_REPORT_PATH`, seguindo integralmente a seção **Estrutura Obrigatória do Relatório Final da Execução**.
13. O relatório deve documentar não apenas os artefatos alterados, mas também os artefatos analisados e não alterados, com justificativa técnica.
14. Toda tabela do relatório deve conter legenda e bloco de threshold / critério de avaliação.
15. Sempre que houver classificação de campo Valor em contexto de saída ou propagação, registrar o consumidor subsequente correspondente.
16. O relatório não pode conter métricas fictícias, suposições sem evidência ou conclusão genérica de sucesso sem lastro técnico.

---

## Regras Mandatórias de Execução para a LLM

### 1. Regra de Separação entre Diagnóstico e Plano de Execução

A execução do patch **DEVE** observar, de forma obrigatória, a separação entre:

* **Diagnóstico**: define **COMO** a correção deve ser implementada no código;
* **Plano de Execução**: define **EM QUAIS artefatos/códigos** a correção deve ser aplicada ou preservada sem alteração.

Essa separação **NÃO DEVE** ser confundida, misturada, fundida ou reinterpretada durante a execução.

---

### 2. Natureza da Intervenção no Código

Esta demanda **NÃO caracteriza modificação ampla, reestruturação integral, refatoração global ou modernização irrestrita de código**.

A execução **DEVE** ser tratada como **intervenção pontual**, limitada ao atendimento objetivo da demanda.

Portanto:

* **NÃO é necessário alterar integralmente os artefatos**;
* as modificações **DEVEM ser pontuais**;
* a LLM **DEVE alterar apenas o mínimo necessário** para viabilizar a adequação requerida;
* a execução **NÃO DEVE** promover mudanças estruturais, estéticas, organizacionais ou de padronização que não sejam estritamente necessárias para o patch solicitado.

Em consequência, qualquer alteração que extrapole a intervenção mínima necessária **DEVE** ser considerada indevida.

---

### 3. Regra Mandatória do Diagnóstico

O **Diagnóstico** **DEVE** descrever de forma explícita, objetiva e tecnicamente verificável:

* a estratégia técnica de correção;
* a lógica do patch a ser aplicado;
* o critério de alteração do código;
* a forma exata de implementação da modificação;
* as restrições técnicas que devem ser respeitadas durante a correção.

Com base nisso, a execução **DEVE** seguir estritamente o Diagnóstico no que se refere a **COMO fazer o patch**.

A execução **NÃO DEVE**:

* adotar estratégia alternativa à definida no Diagnóstico;
* introduzir refatoração não solicitada;
* promover otimização paralela;
* alterar abordagem técnica sem previsão explícita;
* ampliar o escopo técnico da correção;
* implementar solução tecnicamente válida, porém diferente da explicitamente definida.

Em resumo: **o Diagnóstico governa o modo de execução do patch**.

---

### 4. Regra Mandatória do Plano de Execução

O **Plano de Execução** **DEVE** identificar de forma explícita, objetiva e inequívoca:

* **quais artefatos/códigos devem ser alterados**;
* **quais artefatos/códigos devem ser preservados sem alteração**;
* o conjunto exato de atuação autorizado para a aplicação do patch.

Com base nisso, a execução **DEVE** seguir estritamente o Plano de Execução no que se refere a **EM QUAIS artefatos fazer o patch**.

A execução **NÃO DEVE**:

* modificar artefato não previsto no Plano de Execução;
* incluir arquivos, programas, copybooks, módulos, tabelas, layouts ou componentes fora do escopo autorizado;
* reinterpretar o plano para expandir a abrangência da correção;
* alterar artefatos explicitamente marcados para preservação.

Em resumo: **o Plano de Execução governa o universo de artefatos atingidos pelo patch**.

---

### 5. Regra de Restrição ao Escopo Funcional e Técnico do CNPJ

A execução **DEVE** restringir todas as modificações ao **escopo funcional e técnico relacionado ao CNPJ**.

Mesmo quando a demanda exigir modificação pontual para adequação ao **CNPJ alfanumérico**, isso **NÃO autoriza** alterações colaterais fora desse escopo.

Portanto, a LLM **NÃO DEVE**:

* alterar trechos não relacionados ao tratamento do CNPJ;
* expandir a intervenção para outros dados, rotinas, layouts ou regras de negócio não vinculados ao CNPJ;
* realizar correções paralelas por conveniência técnica;
* aproveitar a execução para promover ajustes não demandados.

Toda modificação **DEVE** possuir vínculo direto, explícito e justificável com a necessidade de adequação do CNPJ no contexto da demanda.

---

### 6. Regra de Preservação de Origem Externa

A LLM **NÃO DEVE** alterar campos de **CNPJ** ou **CGC** originados de **fontes externas**, inclusive quando presentes em estruturas locais, layouts, books, cópias, interfaces ou áreas de comunicação.

A **origem externa** é, por si só, critério suficiente para **exclusão do escopo de modificação**, independentemente:

* do nome do campo;
* do formato atual;
* do conteúdo armazenado;
* da similaridade com campos internos;
* da possibilidade técnica de alteração.

Exemplos típicos de fonte externa incluem, entre outros:

* tabelas **DB2 de outros sistemas**;
* contratos de interface externa;
* arquivos recebidos de sistemas terceiros;
* estruturas cujo domínio de manutenção não pertença ao sistema em intervenção.

Nesses casos, a conduta obrigatória é **preservar o campo intacto**, salvo determinação explícita em contrário no escopo formal da demanda.

---

### 7. Regra de Conservadorismo na Inferência

Na dúvida, a LLM **NÃO DEVE alterar**.

Quando a taxa de inferência ou o grau de confiança para classificar um campo como:

* **Chave**;
* **Valor**; ou
* **Fonte Externa**

for insuficiente, fraco, ambíguo ou inconclusivo, a conduta obrigatória **DEVE** ser:

* classificar o item como **Indefinido**; e
* **preservá-lo integralmente, sem alteração**.

A incerteza **NUNCA autoriza modificação**.

A ausência de evidência suficiente para alterar deve ser tratada como evidência suficiente para preservar.

---

### 8. Regra Geral de Conformidade

A execução **DEVE** respeitar simultaneamente:

* o **escopo técnico** definido no **Diagnóstico**;
* o **escopo material** definido no **Plano de Execução**; e
* o **escopo funcional** restrito ao tratamento do **CNPJ**, sem extrapolação para elementos adjacentes.

Portanto:

* o **Diagnóstico** controla **COMO alterar**;
* o **Plano de Execução** controla **ONDE alterar**;
* o **escopo da demanda** controla **O QUE pode ser objeto de alteração**.

Qualquer modificação que viole uma dessas delimitações **DEVE** ser considerada **não conforme**.

---

### 9. Vedação de Extrapolação

É **VEDADO** à LLM:

* extrapolar o modo de correção definido no Diagnóstico;
* extrapolar o conjunto de artefatos definido no Plano de Execução;
* alterar artefatos preservados;
* modificar elementos de origem externa;
* realizar alterações implícitas, inferidas sem evidência ou não autorizadas;
* promover mudanças adicionais sob justificativa de melhoria técnica, padronização, estética, organização, refatoração ou conveniência operacional;
* ampliar o escopo da demanda para além do necessário à adequação pontual do CNPJ.

A execução **DEVE** se limitar estritamente:

* ao **patch solicitado**;
* **na forma solicitada**;
* **somente nos artefatos autorizados**;
* **apenas no escopo relacionado ao CNPJ**.

---

### 10. Fórmula de Interpretação Obrigatória

**Diagnóstico = COMO fazer o patch**
**Plano de Execução = EM QUAIS artefatos fazer o patch**
**Escopo da Demanda = O QUE pode ou não pode ser alterado**

---

### 11. Regra Final de Execução

A LLM **DEVE** adotar postura estritamente restritiva e conservadora durante toda a execução.

Isso significa que:

* **somente o que estiver explicitamente autorizado pode ser alterado**;
* **somente a forma explicitamente definida pode ser utilizada**;
* **todo o restante deve ser preservado**.

Na ausência de autorização explícita, a decisão correta **DEVE** ser **não alterar**.

---

## Conclusão Geral

A execução da modificação (patch) do CNPJ alfanumérico deve seguir como base o tratamento pontual dos programas COBOL, preservando os fluxos atuais, respeitando o escopo estrito da demanda e incorporando o princípio estrutural de separação entre **Chave de CNPJ** e **Valor de CNPJ**. Toda implementação deve assumir correlação 1:1, distinção semântica obrigatória, não exposição da chave em contexto externo, compatibilidade com ambiente mainframe e intervenção mínima necessária nos artefatos do escopo.

Agora, você deve: Modificar todos os artefatos COBOL necessários para adequar o tratamento de CNPJ ao formato alfanumérico, seguindo as premissas, regras de execução e estrutura de relatório definidas neste PRD. Ao final, gerar o relatório consolidado `dem_i_patch_apply.md` com o detalhamento técnico da execução contendo TODO O CONTEÚDO que foi mapeado em: '## Estrutura Obrigatória do Relatório Final da Execução', certifique-se que tudo foi feito corretamente antes da entrega final.