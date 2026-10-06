# Arquitetura e tratamento dos dados

A arquitetura do projeto representa, de forma visual, o fluxo completo de tratamento e análise dos dados, estruturado para suportar tanto o processamento de dados estáticos quanto uma futura automação do pipeline.

No escopo atual, os dados utilizados são estáticos, o que torna a orquestração (parte da automação do pipeline) dispensável. Caso a empresa opte por integrar dados dinâmicos, a infraestrutura Azure já está preparada para receber a camada de orquestração via Azure Data Factory, sem necessidade de reestruturação.

<img width="560" height="348" alt="Diagrama sem nome drawio" src="https://github.com/user-attachments/assets/aae46c64-d635-40d0-9dbd-26719bd9a1ca" />

Para armazenar e processar os dados, foi utilizado o Microsoft Fabric, uma plataforma que reúne em um só ambiente o armazenamento (dos dados brutos e processados), a transformação e o consumo dos dados. Isso evita a movimentação entre ferramentas diferentes e facilita o controle do fluxo.

Para o armazenamento, o Data Lakehouse armazena os dados brutos, na sua origem, sem exigir um esquema definido de antemão, e preserva a fonte original para consulta ou reprocessamento sempre que necessário. Já o Data Warehouse armazena os dados tratados, organizados em tabelas relacionais com colunas e tipos de dados bem definidos, esses dados ficam prontos para consulta com SQL e para as etapas seguintes de análise e treinamento do modelo

Para a transformação, foi utilizado o Dataflow, baseado em Power Query. Ele oferece uma interface visual em que cada passo da transformação fica registrado em ordem, o que torna a limpeza rastreável e fácil de revisar.

Para o desenvolvimento do modelo de machine learning, será utilizado o ambiente de machine learning do próprio Fabric utilizando a linguagem de programação Python.

<img width="593" height="231" alt="image" src="https://github.com/user-attachments/assets/5426a01e-7f5f-4689-999f-cb235a54bdc2" />

<img width="569" height="144" alt="image" src="https://github.com/user-attachments/assets/97b2be8e-cae3-4880-9af5-365b390eee31" />

<img width="619" height="288" alt="image" src="https://github.com/user-attachments/assets/13e7632a-ce93-4528-9f06-b65facca1429" />

<img width="600" height="349" alt="image" src="https://github.com/user-attachments/assets/a5d27f09-bbc6-45f1-b132-823e3ccecd11" />

<img width="613" height="291" alt="image" src="https://github.com/user-attachments/assets/3fa9facd-9808-4f6d-8ed7-6762c6254d2a" />

<img width="569" height="322" alt="image" src="https://github.com/user-attachments/assets/1299792b-6ea7-4bf0-98ea-2d4be35d2e4f" />

<img width="579" height="256" alt="image" src="https://github.com/user-attachments/assets/7fbe682f-4c9c-4a44-9936-e38685503bba" />

<img width="620" height="490" alt="image" src="https://github.com/user-attachments/assets/28d719e9-b627-4664-9708-f44aeb04f6a8" />

<img width="606" height="521" alt="image" src="https://github.com/user-attachments/assets/2192bf99-9ce8-4da8-b05f-c1451f4bad8a" />

<img width="598" height="224" alt="image" src="https://github.com/user-attachments/assets/1c7018ba-d81a-4fa4-a9d3-17a9951c48e3" />

<img width="609" height="392" alt="image" src="https://github.com/user-attachments/assets/99c84cf3-0319-4d77-9237-2b7480b2f9bd" />


Vale destacar que os dados utilizados já haviam sido tratados no semestre anterior, em ambiente AWS, e foram migrados para o Microsoft Fabric para dar continuidade ao trabalho. Nesta etapa, a equipe analisou as transformações aplicadas no Dataflow e verificou o conjunto de dados migrado.




