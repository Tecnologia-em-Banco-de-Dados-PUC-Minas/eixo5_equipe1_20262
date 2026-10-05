# Arquitetura e tratamento dos dados

A arquitetura do projeto representa, de forma visual, o fluxo completo de tratamento e análise dos dados, estruturado para suportar tanto o processamento de dados estáticos quanto uma futura automação do pipeline.

No escopo atual, os dados utilizados são estáticos, o que torna a orquestração (parte da automação do pipeline) dispensável. Caso a empresa opte por integrar dados dinâmicos, a infraestrutura Azure já está preparada para receber a camada de orquestração via Azure Data Factory, sem necessidade de reestruturação.

<img width="560" height="348" alt="Diagrama sem nome drawio" src="https://github.com/user-attachments/assets/aae46c64-d635-40d0-9dbd-26719bd9a1ca" />

Para armazenar e processar os dados, foi utilizado o Microsoft Fabric, uma plataforma que reúne em um só ambiente o armazenamento (dos dados brutos e processados), a transformação e o consumo dos dados. Isso evita a movimentação entre ferramentas diferentes e facilita o controle do fluxo.

Para o armazenamento, o Data Lakehouse armazena os dados brutos, na sua origem, sem exigir um esquema definido de antemão, e preserva a fonte original para consulta ou reprocessamento sempre que necessário. Já o Data Warehouse armazena os dados tratados, organizados em tabelas relacionais com colunas e tipos de dados bem definidos, esses dados ficam prontos para consulta com SQL e para as etapas seguintes de análise e treinamento do modelo

Para a transformação, foi utilizado o Dataflow, baseado em Power Query. Ele oferece uma interface visual em que cada passo da transformação fica registrado em ordem, o que torna a limpeza rastreável e fácil de revisar.

Para o desenvolvimento do modelo de machine learning, será utilizado o ambiente de machine learning do próprio Fabric utilizando a linguagem de programação Python.



















!!!!!****COLOCAR AQUI IMAGENS DOS PRINTS DAS TRANSFORMAÇÕES*****!!!!!

Vale destacar que os dados utilizados já haviam sido tratados no semestre anterior, em ambiente AWS, e foram migrados para o Microsoft Fabric para dar continuidade ao trabalho. Nesta etapa, a equipe analisou as transformações aplicadas no Dataflow e verificou o conjunto de dados migrado.




