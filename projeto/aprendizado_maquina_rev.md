# Modelagem Espaço-Temporal e Potencial de Mercado Automotivo (Minas Gerais)

## 📌 Visão Geral do Projeto
Este módulo consolida o pipeline de **Engenharia de Dados**, **Machine Learning Não Supervisionado**, **Modelagem Preditiva de Demanda**, **Econometria Espacial (*Spillover*)** e **Projeções Temporais de Longo Prazo** para os 853 municípios de Minas Gerais.

A arquitetura do projeto supera as limitações de abordagens agregadas e estaduais ao modelar a heterogeneidade socioeconômica regional, tratando matematicamente a influência que polos logísticos e centros metropolitanos exercem sobre cidades periféricas.

---

## 🛠️ Pipeline de Engenharia e Integração de Dados

Para viabilizar o treinamento e a calibração analítica, foi estruturada a **Base Master Analítica (2016–2025)** por meio da harmonização de três fontes primárias:

1. **Demografia Oficial (IBGE / SIDRA):**
   * Extração direta dos dados censitários de 2010 e 2022 via API SIDRA.
   * Cálculo da Taxa de Crescimento Anual Composta ($CAGR$) intercensitária de 12 anos:
     $$CAGR = \left(\frac{\text{Pop}_{2022}}{\text{Pop}_{2010}}\right)^{\frac{1}{12}} - 1$$
   * Projeção exponencial dos anos intermédios (2016–2025), eliminando distorções de estimativas lineares e descontinuidades censitárias.

2. **Histórico da Frota Veicular (SENATRAN 2016–2025):**
   * Consolidação automatizada de séries anuais da frota de veículos (linhas leve e pesada).
   * Detecção dinâmica de cabeçalhos e normalização de strings via `unicodedata` (remoção de diacríticos e caracteres especiais).
   * Dicionário de padronização toponímica para pareamento com a malha oficial do IBGE, mitigando casos de homônimos e divergências cadastrais.

3. **Consumo de Combustíveis (ANP):**
   * Consolidação das vendas anuais de Etanol Hidratado, Gasolina C (agrupados no Ciclo Leve/Otto) e Óleo Diesel (Ciclo Pesado).
   * Tratamento de tipagem numérica, conversão volumétrica e agregação por Código IBGE.

---

## 🧠 Arquitetura de Modelagem e Algoritmos

O fluxo analítico segue uma sequência encadeada, onde o resultado de cada etapa atua como pré-requisito para o modelo subsequente:

```
[Base Master Analítica (2016-2025)]
               │
               ▼
   [1. K-Means (K=6 Clusters)]
               │
               ├───► [2. Regressão Linear de Demanda (Treino/Teste)]
               │
               ├───► [3. Econometria Espacial: Spillover Dinâmico (KNN=5)]
               │
               └───► [4. Projeção Temporal Macro: Grau 2 (2026-2030)]
```

---

### 1. Segmentação Hierárquica de Mercado (K-Means)
* **Objetivo:** Agrupar municípios por escala estrutural similar antes da regressão preditiva, evitando que metrópoles (ex.: Belo Horizonte) e micromunicípios rurais distorçam a reta de ajuste global.
* **Metodologia:**
  * Variáveis base: médias históricas municipais de população, frota total, consumo de ciclo leve e consumo de diesel.
  * Padronização via `StandardScaler`.
  * Clusterização com `KMeans(n_clusters=6, random_state=42, n_init=10)`.
* **Rotulagem Hierárquica Automática:** Classificação baseada no indicador de *força bruta*:
  $$\text{Força Bruta} = \overline{\text{Frota Total}} + \overline{\text{Consumo Leve}} + \overline{\text{Consumo Diesel}}$$
  * Nível 1: *Polo Logístico e Metropolitano*
  * Nível 2: *Centro Regional Principal*
  * Nível 3: *Polo Econômico Intermediário*
  * Nível 4: *Município em Desenvolvimento*
  * Nível 5: *Pequenos Municípios de Apoio*
  * Nível 6: *Micromunicípios / Rurais*
* **Artefato de Saída:** `Base_Consolidada_Clusters_MG.xlsx`

---

### 2. Modelo Preditivo de Demanda (Regressão Linear)
* **Objetivo:** Estimar o volume de vendas a partir do perfil populacional, frota estimada e categoria veicular.
* **Metodologia:**
  * `Pipeline` estruturado com `ColumnTransformer`.
  * Normalização z-score (`StandardScaler`) nas variáveis quantitativas e codificação categórica (`OneHotEncoder(drop='first')`) para o tipo de veículo.
  * Separação balanceada entre treino (80%) e teste (20%).
* **Papel da Clusterização Prévia:** Redução significativa do erro percentual ($MAPE$) ao permitir que a regressão capture o comportamento do mercado condicionado à escala estrutural do município.

---

### 3. Econometria Espacial: Efeito *Spillover* Dinâmico
* **Objetivo:** Quantificar o transbordo geográfico que cidades-polo exercem na demanda dos municípios circunvizinhos.
* **Metodologia:**
  * Extração das coordenadas geográficas (latitude e longitude) de cada município.
  * Construção da Matriz de Pesos Espaciais de 5 vizinhos mais próximos (`libpysal.weights.KNN(k=5)`) com padronização por linhas (`transform='r'`).
  * Cálculo da Defasagem Espacial (*Spatial Lag*):
    $$\text{Lag}_i = \sum_{j} w_{ij} y_j$$
* **Taxas Dinâmicas de Absorção ($\rho$):**
  A intensidade de absorção é calculada pela razão entre o mercado vizinho e o mercado local:
  $$\text{Razão de Mercado} = \frac{\text{Lag Espacial}}{\text{Consumo Local} + 1}$$
  * *Ciclo Leve:* $\rho \in [0.05, 0.30]$ (5% a 30%)
  * *Ciclo Pesado:* $\rho \in [0.08, 0.40]$ (8% a 40%, refletindo a dispersão de eixos rodoviários e frotas de carga)
* **Artefato de Saída:** `Base_Academica_Spillover_Segmentada.xlsx`

---

### 4. Projeção Temporal de Longo Prazo (2026–2030)
* **Objetivo:** Projetar o comportamento macroeconômico estadual para o horizonte de 2026 a 2030.
* **Metodologia:**
  * Agregação macro da frota, população e consumo de combustíveis.
  * Ajuste de curvas via `PolynomialFeatures(degree=2)` acoplado a `LinearRegression`.
* **Controle de Variância:** O Grau 2 foi selecionado especificamente para capturar tendências de aceleração sem induzir oscilações ou crescimento exponencial espúrio (risco clássico de sobreajuste em graus $\ge 3$).
* **Artefato de Saída:** `Projecao_Crescimento_MG_2026_2030.xlsx`

---

## 📊 Diagnóstico e Validação dos Modelos

O pipeline conta com um módulo de diagnóstico unificado para validação da estabilidade estatística:

| Frente Analítica | Métrica Avaliada | Valor Obtido | Diagnóstico |
| :--- | :--- | :--- | :--- |
| **Demanda (Regressão Linear)** | $R^2$ Treino / Teste <br> $MAPE$ Teste | **0.990 / 0.989** <br> **35.49%** | **Ajuste Adequado:** Generalização estável, sem sinais de memorização ou predições negativas. |
| **Econometria Espacial** | Índice de Moran (Leve / Pesado) <br> $R^2$ Espacial Puro | **0.135 / 0.138** <br> **~0.059** | **Spillover Moderado:** Autocorrelação espacial positiva comprovando que a demanda dos vizinhos afeta diretamente a demanda local. |
| **Projeção Macro (2026–2030)** | Crescimento Frota Total <br> Crescimento Óleo Diesel | **+13.15%** <br> **+30.45%** | **Curva Sustentável:** Projeção alinhada com as dinâmicas de renovação de frota e transporte rodoviário. |

---

## 📁 Estrutura de Artefatos Gerados

```text
├── Base_Master_Analitica_MG_2016_2025.xlsx     # Base histórica unificada (demografia, frota e consumo)
├── Base_Consolidada_Clusters_MG.xlsx          # Rótulos hierárquicos do K-Means por município
├── Base_Academica_Spillover_Segmentada.xlsx   # Matriz espacial, lags e volumes com spillover
└── Projecao_Crescimento_MG_2026_2030.xlsx     # Série projetada de mercado para o próximo quinquênio
```

---

## 💻 Dependências do Ambiente

Para reproduzir o pipeline, instale as dependências:

```bash
pip install matplotlib numpy pandas scikit-learn openpyxl libpysal spreg requests
```
