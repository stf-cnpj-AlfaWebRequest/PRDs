# ROBOT TEST SCRIPT

## Description

Dado o repositório e os códigos COBOL presentes em .pss/b_new/a_code, crie uma pasta chamada "b_robot_script" dentro de ".pss/b_new/c_test" e siga as orientações abaixo.

Além disso, leve em consideração o documento de requisitos localizado no caminho .pss\b_new\w_output\dem_a_cnpj_alfa_prd\dem_prd_requisito.md para entender o contexto do projeto e dos programas COBOL.

## Objective

Gerar scripts Robot de teste para os códigos cobol da pasta .pss/b_new/a_code

## Contexto

Leia os CSVs dentro da pasta .pss/b_new/c_test/a_csv.

A partir dos códigos COBOL localizados na pasta ".pss/b_new/a_code" e a partir dos csv especificando os cenários de testes localizados na pasta ".pss/b_new/c_test/a_csv", crie um script em Robot Framework para cada arquivo COBOL com o objetivo de executar e validar os programas em ambiente de mainframe, simulando a interação do usuário e verificando os resultados retornados.

Os testes devem abranger todas as funcionalidades relevantes de cada arquivo COBOL, contemplando cenários positivos, nos quais os dados de entrada são válidos e o fluxo ocorre corretamente, e cenários negativos, nos quais entradas inválidas ou condições de erro são tratadas pelo programa, validando mensagens, retornos ou comportamentos esperados.

Caso um código COBOL realize chamadas para outros programas, por meio de instruções como CALL, LINK ou XCTL, isso deve ser considerado na construção dos scripts de teste em Robot Framework, garantindo que o fluxo completo seja exercitado e que todos os programas dependentes necessários para a execução sejam incluídos e considerados nos testes.

Os scripts devem ser compatíveis com execução em ambiente de mainframe (z/OS), sendo capazes de interagir com o sistema conforme necessário para execução dos testes automatizados, incluindo o uso de bibliotecas apropriadas quando necessário.

Dentro da pasta "b_robot_script", crie subpastas com o nome dos arquivos COBOL selecionados e inclua em cada uma delas o respectivo script de teste em Robot Framework, uma cópia do arquivo COBOL testado e uma cópia de todos os arquivos dependentes necessários para a execução do teste, como outros programas COBOL ou copybooks. 

## Documentação

Após a criação dos scripts, gere um arquivo de documentação chamado "dem_l_test_script_robot.md" em formato markdown dentro da pasta "b_robot_script", consolidando as informações dos testes desenvolvidos.

A documentação gerada deve seguir as seguintes diretrizes:

- Iniciar com a seção "Introdução" e finalizar com a seção "Conclusão", ambas contendo um parágrafo curto explicando o que será abordado.
- Todo o conteúdo deve ser escrito em português, utilizando linguagem técnica, clara e objetiva.
- Não incluir metadados no topo do documento.
- Não escrever caminhos de arquivos, ex: ".pss/".
- Não escrever estrutura dos arquivos em nenhum momento. Sem excessões.
- Não escrever o nome dos arquivos em nenhum momento. Sem excessões.
- Não referenciar o PRD diretamente.
- Analisar cuidadosamente o documento de requisitos para entender o contexto do projeto e dos programas COBOL, a fim de evitar contradições internas.
- Analisar cuidadosamente as PREMISSAS para evitar contradições internas.
- Manter consistência terminológica ao longo do texto.
- NÃO utilizar emojis.
- Não inventar informações; quaisquer inferências devem ser plausíveis e justificáveis.
- Evitar textos excessivamente longos, priorizando densidade informacional.
- Utilizar tabelas sempre que agregar clareza, sendo obrigatório incluir legenda explicando cada coluna
- A documentação deve conter, no mínimo:
- Descrição dos programas COBOL testados
- Descrição dos scripts em Robot Framework criados
- Cenários de teste executados, incluindo casos positivos e negativos
- Comportamentos esperados e validações realizadas
- Estratégia de execução dos testes no ambiente de mainframe
- Bibliotecas e dependências utilizadas no Robot Framework
- Observações sobre limitações conhecidas e pré-requisitos para execução
- Versão do Robot Framework utilizada
- Versões de COBOL compatíveis com os testes desenvolvidos
- Ao final do documento, incluir a seguinte seção:

## Rodapé

---

Documento Elaborado: <Data de hoje>
Versão: 6.0

---

## Conteúdo extra (OBRIGATÓRIO)

O documento gerado "dem_l_test_script_robot.md" deve ter em suas seções finais (Antes do rodapé) informações como:

- **Sumário de Entrega**: Programas testados, Estrutura de diretórios criada, arquivos criados juntamente com seu tipo, tamanho e descrição, número de testes implementados, cobertura de cenarios (Cenários positivos, negativos e casos extremos), requisitos atendidos ferramentas utilizadas e etc.
- **Guia de Execução no Mainframe**
- **Referência de Testes**: Por funcionalidade, específica, por tipo de teste, estrutura de resultado e etc.

OBS: **TODOS** os itens acima devem estar bem detalhados, específicados e em linguagem clara.


## Premissas:

Essa documentação precisa ter as seguintes seções:
- **Sessão**: sempre colocar um paragrafo curto, explicando o que a sessão vai abordar.
- **Conclusão**: sempre colocar uma conclusão ao final de cada tarefas/módulos/doc/md.
- **Toda tabela** necessita ter uma legenda explicando/descrevendo cada coluna.
- Verificar se temos algo em inglês, pois tem de ser em português. Toda a documentação tem que ser em português, a menos que tenhamos algum termo técnico em inglês.
- Teremos 2 casos de uso, para realizar a mudança do CNPJ de numérico para alfanumerico:
   - **Chave**: A variável de CNPJ no código é uma chave estrangeira e não o CNPJ propriamente dito.
   - **Valor**: A variável de CNPJ no código é o próprio conteúdo da CNPJ.
- Chave/Valor:
   - **Mudar de REAL para VALOR**
   - **Mudar de LOOKUP para CHAVE**
- Para essa documentação, não faremos a seção de Resumo Executivo.
Assumimos que não faremos a validação da rotina acima, nesta PoC.
Considerando que podemos ter 2 cenários, onde temos campo CHAVE e campo VALOR, decidimos por considerar de 1 arquivos cobol será de CHAVE e o outro será de VALOR, mesmo sabendo que ambos os arquivos são originalmente de CHAVE. Esta decisão intenciona mostrar que identificamos e entendemos os dois casos e somos capazes de performar em ambos:
- Não pode alterar módulos externos (DAF, HLP, etc).
- Não pode pegar a Chave nem o Valor dos módulos que sejam principais.
- VIPP4365.cbl = Considere esse arquivo sendo como chave (ID), ou seja, o campo do CNPJ seria apenas um identificador (chave estrangeira, fk), (Chave).
- VIPP4553.cbl = Considere esse arquivo como se fosse um caso de valor do CNPJ (Valor). \* Funções para obter/validar o CNPJ:
   - **Plataforma Baixa**: Chamar de "Função de Obternção/Validação do CNPJ em Plataforma Baixa", e não utilizar o nome verdade (getXxxbyXxx).
   - **Plataforma Alta**: Chamar de "Função de Obternção/Validação do CNPJ em Plataforma Alta", e não utilizar o nome verdade (SISXXXX).

Não altere nenhum código COBOL do repositório.