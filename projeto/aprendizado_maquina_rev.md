# Modelagem Espaço-Temporal da Demanda Automotiva e de Combustíveis (Minas Gerais)

Este repositório contém a infraestrutura de dados, modelagem preditiva e econometria espacial desenvolvida no arquivo `PUC - ML`[cite: 5, 6]. O objetivo central é diagnosticar e projetar o mercado automotivo e o consumo de combustíveis nos 853 municípios de Minas Gerais[cite: 5].

## 🛠️ Engenharia de Dados (Base Master)
O pipeline consolida diferentes fontes governamentais em uma estrutura analítica unificada (Base Master 2016-2025)[cite: 5]:
*   **Demografia (IBGE):** Conexão com a API SIDRA para extração dos Censos de 2010 e 2022, calculando a Taxa de Crescimento Anual Composta (CAGR) para interpolar e projetar a evolução populacional de cada município[cite: 5].
*   **Frota (SENATRAN):** Consolidação de diversos arquivos anuais da frota de veículos (2013-2025)[cite: 5], aplicando normalização textual para padronizar os nomes dos municípios e viabilizar o cruzamento exato com os códigos do IBGE[cite: 5].
*   **Combustíveis (ANP):** Tratamento estrutural das vendas anuais de gasolina, etanol e óleo diesel, agregando os volumes diretamente no ciclo leve (`consumo_leve_m3`) e no ciclo pesado (`consumo_diesel_m3`)[cite: 5].

## 🧠 Arquitetura de Machine Learning e Econometria
A modelagem analítica segue uma esteira sequencial implementada no `PUC - ML`[cite: 5, 6]:

### 1. Segmentação de Mercado (K-Means)
*   **Objetivo:** Agrupar municípios com portes e perfis econômicos semelhantes antes de qualquer previsão, garantindo que o modelo não trate metrópoles e áreas rurais com a mesma métrica[cite: 5].
*   **Implementação:** Aplicação de padronização (`StandardScaler`) e algoritmo não-supervisionado `KMeans(n_clusters=6)` sobre as médias históricas de população, frota e volumes de consumo[cite: 5].
*   **Resultado:** Nomeação hierárquica automática dos grupos com base na "força bruta" de mercado (desde "Polo Logístico e Metropolitano" até "Micromunicípios / Rurais")[cite: 5].

### 2. Regressão Linear (Demanda Segmentada)
*   **Objetivo:** Modelar o volume de vendas a partir das características populacionais e de frota[cite: 5].
*   **Implementação:** Construção de um `Pipeline` contendo `ColumnTransformer` (para escalonamento e codificação de variáveis categóricas via OneHotEncoder) e o estimador `LinearRegression`[cite: 5].
*   **Validação:** Separação de treino (80%) e teste (20%), resultando na avaliação direta do desempenho preditivo através de indicadores como R², MAE e MAPE[cite: 5].

### 3. Efeito Spillover (Transbordo Espacial)
*   **Objetivo:** Quantificar o impacto geográfico que os grandes centros exercem no consumo das cidades adjacentes[cite: 5].
*   **Implementação:** Utilização da biblioteca `libpysal` para extrair matrizes de pesos espaciais baseadas nos 5 vizinhos mais próximos (`KNN` com padronização em linha) usando latitude e longitude[cite: 5].
*   **Métricas Dinâmicas:** Cálculo do Lag Espacial para determinar taxas de absorção locais e estimar volumes incrementais gerados puramente pelo efeito vizinhança[cite: 5].

### 4. Projeção Temporal (2027-2031)
*   **Objetivo:** Estimar o comportamento futuro das variáveis macroeconômicas no estado de Minas Gerais.
*   **Implementação:** Regressão Polinomial de Grau 2 (`PolynomialFeatures(degree=2)`) sobre o horizonte consolidado, garantindo uma curva preditiva realista para população, frota e consumo sem as explosões típicas do overfitting de graus superiores.
*   **Entregáveis:** Exportação do relatório consolidado contendo os dados absolutos projetados (2027-2031), variações percentuais face ao histórico (YoY) (`Projecao_Crescimento_MG_2027_2031.xlsx`) e visualização gráfica local da projeção e janela tendencial (`projecao_tendencias_2027_2031.png`).

## 📊 Relatório de Diagnóstico de Eficiência
A avaliação dos modelos é centralizada em um script de diagnóstico automatizado[cite: 5]:

| Frente Analítica | Diagnóstico e Resultados |
| :--- | :--- |
| **Demanda (Regressão Linear)** | Avalia as pontuações de R² e MAPE, verificando a capacidade de generalização e prevenindo underfitting/overfitting[cite: 5]. |
| **Avaliação Espacial** | Mensura a dependência geográfica das vendas calculando o Índice de Moran e o R² Espacial[cite: 5]. |
| **Sustentabilidade da Projeção (2027-2031)** | **Crescimento Acumulado (2031 vs Histórico):** Frota 24.36% \| Diesel 58.55%<br>**Crescimento YoY (Ano-a-Ano no fim de 2031):** Frota 2.96% \| Diesel 7.13%<br>**Crescimento apenas na Janela de Projeção (2027-2031):** Frota 12.84% \| Diesel 31.11% |

## 📁 Estrutura de Artefatos Gerados

```text
├── Base_Master_Analitica_MG_2016_2025.xlsx    # Base histórica unificada (demografia, frota e consumo)
├── Base_Consolidada_Clusters_MG.xlsx          # Rótulos hierárquicos do K-Means por município
├── Base_Academica_Spillover_Segmentada.xlsx   # Matriz espacial, lags e volumes com spillover
├── Projecao_Crescimento_MG_2027_2031.xlsx     # Série projetada de mercado para o próximo quinquênio
└── projecao_tendencias_2027_2031.png          # Visualização gráfica das curvas tendenciais
