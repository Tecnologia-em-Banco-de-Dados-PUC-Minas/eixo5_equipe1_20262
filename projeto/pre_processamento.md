# Arquitetura e tratamento dos dados

A arquitetura do projeto representa, de forma visual, o fluxo completo de tratamento e análise dos dados, estruturado para suportar o processamento de dados estáticos e uma futura automação do pipeline.

No escopo atual, os dados são estáticos, o que dispensa a orquestração (parte da automação do pipeline). Caso a empresa opte por integrar dados dinâmicos, a infraestrutura Azure já está preparada para receber a camada de orquestração via Azure Data Factory, sem necessidade de reestruturação.

<img width="560" height="348" alt="Diagrama da arquitetura do projeto" src="https://github.com/user-attachments/assets/aae46c64-d635-40d0-9dbd-26719bd9a1ca" />

---

## 1. Armazenamento e processamento

Foi utilizado o **Microsoft Fabric**, plataforma que reúne em um só ambiente o armazenamento (de dados brutos e processados), a transformação e o consumo dos dados. Isso evita a movimentação entre ferramentas diferentes e facilita o controle do fluxo.

- **Data Lakehouse:** armazena os dados brutos, na origem, sem exigir esquema prévio, preservando a fonte original para consulta ou reprocessamento sempre que necessário.
- **Data Warehouse:** armazena os dados tratados, organizados em tabelas relacionais com colunas e tipos bem definidos, prontos para consulta em SQL e para as etapas de análise e treinamento do modelo.

---

## 2. Transformação

A transformação foi feita no **Dataflow**, baseado em **Power Query**. Sua interface visual registra cada passo em ordem, o que torna a limpeza rastreável e fácil de revisar. O Dataflow busca os arquivos-base no Data Lakehouse, trata os dados e os direciona ao Data Warehouse, para uso em dashboards e em projetos de machine learning.

O projeto reúne **7 consultas**, cada uma conectada a um arquivo CSV:

1. Frota de veículos por município e tipo (dezembro de 2025);
2. População de Minas Gerais em 2025;
3. Tabela `base`, usada como base de integração;
4. Composição das regiões geográficas por município (IBGE, 2017);
5. Vendas anuais de etanol hidratado por município;
6. Vendas anuais de gasolina C por município;
7. Vendas anuais de óleo diesel por município.

 <img width="1793" height="802" alt="image" src="https://github.com/user-attachments/assets/3527f37e-494b-4944-89a1-2009f0495fa0" />
 
<img width="1791" height="807" alt="image" src="https://github.com/user-attachments/assets/004c6859-c83f-4377-960c-da0780616fb1" />

### 2.1 Extração e padronização inicial

Em todas as consultas foram executadas as mesmas etapas básicas:

- **Origem e Navegação:** conexão com a fonte e acesso ao arquivo CSV;
- **CSV Importado:** leitura do arquivo e conversão em tabela estruturada;
- **Cabeçalhos promovidos:** a primeira linha passou a ser o nome das colunas;
- **Tipo de coluna alterado:** definição do tipo de dado de cada coluna, garantindo consistência para cálculos e relacionamentos.

### 2.2 Padronização do código do município

O código de município do IBGE possui **7 dígitos**: os dois primeiros identificam a **UF** e os cinco seguintes, o município dentro do estado. Em todas as tabelas, o campo foi tratado da mesma forma:

- **Extração dos 2 primeiros caracteres:** isolou o código da UF;
- **Extração dos 5 últimos caracteres:** gerou uma nova coluna apenas com o código do município;
- **Renomeação e reordenação das colunas:** os campos receberam nomes padronizados e posição consistente entre as consultas.

Com isso, o código do município ficou **uniforme em todas as fontes**, formando a chave de relacionamento entre as tabelas.

### 2.3 Ajustes específicos

- **Tabela de população:** aplicou-se a **substituição de valores**, corrigindo inconsistências para que a coluna fosse reconhecida como numérica.
- **Tabela `base`:** a coluna de **valor** exigiu correção do separador decimal. No padrão brasileiro, a vírgula separa as casas decimais (`1234,56`), mas em arquivos CSV ela também funciona como delimitador de campos, o que poderia dividir os valores em colunas distintas ou lê-los como texto. A **vírgula foi substituída por ponto** (`1234.56`), padrão reconhecido pelo Power Query, e a coluna passou a ser lida como **número decimal**.

### 2.4 Finalidade do tratamento

- **Limpeza:** correção de inconsistências, como a substituição na tabela de população e o ajuste do separador decimal na `base`;
- **Padronização:** nomes, tipos e ordem das colunas uniformes, com a chave do município no mesmo formato em todas as tabelas;
- **Adequação de formatos:** compatibilização de idioma e localidade entre o CSV e o Power Query, evitando erros de leitura numérica;
- **Preparação para integração:** com a chave comum e os tipos corretos, é possível relacionar frota, população, regiões geográficas e vendas de combustíveis em um único modelo analítico.

---

## 3. Machine learning

Para o desenvolvimento do modelo de machine learning, será utilizado o ambiente de machine learning do próprio Fabric e o Google Colab, com a linguagem **Python**.
