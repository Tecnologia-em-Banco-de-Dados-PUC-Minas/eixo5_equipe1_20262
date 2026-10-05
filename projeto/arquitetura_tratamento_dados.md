# Arquitetura e tratamento dos dados

A arquitetura do projeto representa, de forma visual, o fluxo completo de tratamento e análise dos dados, estruturado para suportar tanto o processamento de dados estáticos quanto uma futura automação do pipeline.
No escopo atual, os dados utilizados são estáticos, o que torna a orquestração (parte da automação do pipeline) dispensável. Caso a empresa opte por integrar dados dinâmicos, a infraestrutura Azure já está preparada para receber a camada de orquestração via Azure Data Factory, sem necessidade de reestruturação.
Vale destacar que os dados utilizados já haviam sido tratados no semestre anterior, em ambiente AWS, e foram migrados para o Microsoft Fabric para dar continuidade ao trabalho. Nesta etapa, a equipe analisou as transformações aplicadas no Dataflow e verificou o conjunto de dados migrado.




