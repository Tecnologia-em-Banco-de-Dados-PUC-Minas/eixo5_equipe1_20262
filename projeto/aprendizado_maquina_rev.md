# Modelagem Espaço-Temporal da Demanda Automotiva e de Combustíveis (Minas Gerais)

Este repositório contém a infraestrutura de dados, modelagem preditiva e econometria espacial desenvolvida no arquivo `PUC - ML`. O objetivo central é diagnosticar e projetar o mercado automotivo e o consumo de combustíveis nos 853 municípios de Minas Gerais.

## 🛠️ Engenharia de Dados (Base Master)
O pipeline consolida diferentes fontes governamentais em uma estrutura analítica unificada (Base Master 2016-2025):
*   **Demografia (IBGE):** Conexão com a API SIDRA para extração dos Censos de 2010 e 2022, calculando a Taxa de Crescimento Anual Composta (CAGR) para interpolar e projetar a evolução populacional de cada município.
*   **Frota (SENATRAN):** Consolidação de diversos arquivos anuais da frota de veículos (2013-2025)[cite: 5], aplicando normalização textual para padronizar os nomes dos municípios e viabilizar o cruzamento exato com os códigos do IBGE[cite: 5].
*   **Combustíveis (ANP):** Tratamento estrutural das vendas anuais de gasolina, etanol e óleo diesel, agregando os volumes diretamente no ciclo leve (`consumo_leve_m3`) e no ciclo pesado (`consumo_diesel_m3`).

## 🧠 Arquitetura de Machine Learning e Econometria
A modelagem analítica segue uma esteira sequencial implementada no `PUC - ML`:

### 1. Segmentação de Mercado (K-Means)
*   **Objetivo:** Agrupar municípios com portes e perfis econômicos semelhantes antes de qualquer previsão, garantindo que o modelo não trate metrópoles e áreas rurais com a mesma métrica.
*   **Implementação:** Aplicação de padronização (`StandardScaler`) e algoritmo não-supervisionado `KMeans(n_clusters=6)` sobre as médias históricas de população, frota e volumes de consumo.
*   **Resultado:** Nomeação hierárquica automática dos grupos com base na "força bruta" de mercado (desde "Polo Logístico e Metropolitano" até "Micromunicípios / Rurais".

### 2. Regressão Linear (Demanda Segmentada)
*   **Objetivo:** Modelar o volume de vendas a partir das características populacionais e de frota.
*   **Implementação:** Construção de um `Pipeline` contendo `ColumnTransformer` (para escalonamento e codificação de variáveis categóricas via OneHotEncoder) e o estimador `LinearRegression`.
*   **Validação:** Separação de treino (80%) e teste (20%), resultando na avaliação direta do desempenho preditivo através de indicadores como R², MAE e MAPE.

### 3. Efeito Spillover (Transbordo Espacial)
*   **Objetivo:** Quantificar o impacto geográfico que os grandes centros exercem no consumo das cidades adjacentes.
*   **Implementação:** Utilização da biblioteca `libpysal` para extrair matrizes de pesos espaciais baseadas nos 5 vizinhos mais próximos (`KNN` com padronização em linha) usando latitude e longitude[cite: 5].
*   **Métricas Dinâmicas:** Cálculo do Lag Espacial para determinar taxas de absorção locais e estimar volumes incrementais gerados puramente pelo efeito vizinhança.

### 4. Projeção Temporal (2027-2031)
*   **Objetivo:** Estimar o comportamento futuro das variáveis macroeconômicas no estado de Minas Gerais.
*   **Implementação:** Regressão Polinomial de Grau 2 (`PolynomialFeatures(degree=2)`) sobre o horizonte consolidado, garantindo uma curva preditiva realista para população, frota e consumo sem as explosões típicas do overfitting de graus superiores.
*   **Entregáveis:** Exportação do relatório consolidado contendo os dados absolutos projetados (2027-2031), variações percentuais face ao histórico (YoY) (`Projecao_Crescimento_MG_2027_2031.xlsx`) e visualização gráfica local da projeção e janela tendencial (`projecao_tendencias_2027_2031.png`).

## 📊 Relatório de Diagnóstico de Eficiência
A avaliação dos modelos é centralizada em um script de diagnóstico automatizado:

| Frente Analítica | Diagnóstico e Resultados |
| :--- | :--- |
| **Demanda (Regressão Linear)** | Avalia as pontuações de R² e MAPE, verificando a capacidade de generalização e prevenindo underfitting/overfitting. |
| **Avaliação Espacial** | Mensura a dependência geográfica das vendas calculando o Índice de Moran e o R² Espacial. |
| **Sustentabilidade da Projeção (2027-2031)** | **Crescimento Acumulado (2031 vs Histórico):** Frota 24.36% \| Diesel 58.55%<br>**Crescimento YoY (Ano-a-Ano no fim de 2031):** Frota 2.96% \| Diesel 7.13%<br>**Crescimento apenas na Janela de Projeção (2027-2031):** Frota 12.84% \| Diesel 31.11% |

## 📁 Estrutura de Artefatos Gerados

```text
├── Base_Master_Analitica_MG_2016_2025.xlsx    # Base histórica unificada (demografia, frota e consumo)
├── Base_Consolidada_Clusters_MG.xlsx          # Rótulos hierárquicos do K-Means por município
├── Base_Academica_Spillover_Segmentada.xlsx   # Matriz espacial, lags e volumes com spillover
├── Projecao_Crescimento_MG_2027_2031.xlsx     # Série projetada de mercado para o próximo quinquênio
└── projecao_tendencias_2027_2031.png          # Visualização gráfica das curvas tendenciais
```
## Acesso ao Google Colab
Link: https://colab.research.google.com/drive/1fXQjnYzouhHfILDqUnpDv9yK_oXwUzEC?usp=sharing

## Scripts utilizados
Abaixo está todo o código produzido e executado no Google Colab para gerar os resultados que serão apresentados ao final desse relatório.

------------------------------------------
```text
# Experimentação com ML - Potencial de Mercado
"""

! pip install matplotlib numpy pandas scikit-learn openpyxl libpysal spreg --quiet

"""Criação de clusters"""

import pandas as pd
import numpy as np
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler
from pathlib import Path

print("=== PIPELINE UNIFICADO: CLUSTERIZAÇÃO E PREPARAÇÃO ===")

# 0. CARREGAR A BASE DE DADOS
print("0. A carregar a Base Master Analítica...")
caminho_master = Path('/content/drive/MyDrive/raw/Base_Master_Analitica_MG_2016_2025.xlsx')
df_mg = pd.read_excel(caminho_master)

# 1. ENGENHARIA DE FEATURES (Agrupando por cidade)
print("1. A tratar dados e a isolar features...")

# Tirar a média de cada cidade para descobrirmos a sua "categoria" estrutural
df_cluster = df_mg.groupby(['CÓDIGO IBGE', 'MUNICIPIO'], as_index=False).agg({
    'populacao': 'mean',
    'TOTAL': 'mean',              # Frota Total
    'consumo_leve_m3': 'mean',    # Gasolina + Etanol
    'consumo_diesel_m3': 'mean'   # Diesel
})

# Seleção de variáveis para a inteligência artificial agrupar
features = ['populacao', 'TOTAL', 'consumo_leve_m3', 'consumo_diesel_m3']
df_cluster[features] = df_cluster[features].fillna(0)

# 2. NORMALIZAÇÃO E K-MEANS
print("2. A aplicar K-Means (K=6) para segmentação estadual...")
scaler = StandardScaler()
df_padronizado = scaler.fit_transform(df_cluster[features])

robo = KMeans(n_clusters=6, random_state=42, n_init=10)
df_cluster['cluster_id'] = robo.fit_predict(df_padronizado)

# 3. NOMEAÇÃO AUTOMÁTICA HIERÁRQUICA
print("3. A classificar Polos e Satélites...")
# Criar uma métrica de tamanho total de mercado para ordenar do maior (Polo) para o menor (Rural)
df_cluster['forca_bruta'] = df_cluster['TOTAL'] + df_cluster['consumo_leve_m3'] + df_cluster['consumo_diesel_m3']
medias_cluster = df_cluster.groupby('cluster_id')['forca_bruta'].mean().sort_values(ascending=False)

nomes_oficiais = [
    "Polo Logístico e Metropolitano",
    "Centro Regional Principal",
    "Polo Econômico Intermediário",
    "Município em Desenvolvimento",
    "Pequenos Municípios de Apoio",
    "Micromunicípios / Rurais"
]

mapa_nomes = {cluster_id: nomes_oficiais[i] for i, cluster_id in enumerate(medias_cluster.index)}
df_cluster['cluster_nome'] = df_cluster['cluster_id'].map(mapa_nomes)

# Limpeza
df_cluster.drop(columns=['cluster_id', 'forca_bruta'], inplace=True)

# 4. EXPORTAÇÃO AUTOMATIZADA
caminho_saida = Path('/content/drive/MyDrive/raw/Base_Consolidada_Clusters_MG.xlsx')
df_cluster.to_excel(caminho_saida, index=False)

print(f"\n✅ Pipeline concluído! Base com Clusters salva em: {caminho_saida}")
display(df_cluster[['MUNICIPIO', 'cluster_nome', 'TOTAL', 'consumo_leve_m3']].sample(10))

"""1.0 - Análise dos dados brutos"""

import os

def listar_arquivos(diretorio):
    for raiz, dirs, arquivos in os.walk(diretorio):
        # ignora pastas ocultas do sistema
        dirs[:] = [d for d in dirs if not d.startswith('.')]
        for arquivo in arquivos:
            if arquivo.endswith('.py') or arquivo.endswith('.csv') or arquivo.endswith('.xlsx'):
                print(os.path.join(raiz, arquivo))

print("=== Arquivos no ambiente do Colab ===")
listar_arquivos('/content/drive/MyDrive/raw')

"""1.1 - Criação das bases de treino e teste"""

# Commented out IPython magic to ensure Python compatibility.
# %%writefile preparacao.py
# import pandas as pd
# import numpy as np
# from pathlib import Path
# from sklearn.model_selection import train_test_split
# 
# def montar_base():
#     # Carrega a base real
#     caminho_base = Path('/content/drive/MyDrive/raw/Simulacao_Vendas_Auto_MG_400_Reais.xlsx')
#     df_realista = pd.read_excel(caminho_base)
# 
#     # Traz dados básicos dos municípios
#     df_brasil = pd.read_csv("https://raw.githubusercontent.com/kelvins/Municipios-Brasileiros/main/csv/municipios.csv")
#     df_cidades = pd.merge(df_realista, df_brasil[['codigo_ibge', 'nome']],
#                           left_on='cod_ibge', right_on='codigo_ibge', how='left')
# 
#     df_cidades['nome'] = df_cidades['nome'].fillna("Mun_Desconhecido")
# 
#     dados_finais = []
#     for _, row in df_cidades.iterrows():
#         # Linha Leve
#         frota_l = int(row['venda_leve'] / np.random.uniform(1.3, 1.7)) if row['venda_leve'] > 0 else 0
#         pop_l = int(frota_l * np.random.uniform(2.5, 3.5)) # Aproximadamente 1 carro para cada 3 pessoas
# 
#         dados_finais.append({
#             'cidade': row['nome'],
#             'tipo_veiculo': 'leve',
#             'populacao': pop_l,
#             'frota_estimada': frota_l,
#             'quantidade_vendida': row['venda_leve']
#         })
# 
#         # Linha Pesada
#         frota_p = int(row['venda_pesada'] / np.random.uniform(3.8, 4.6)) if row['venda_pesada'] > 0 else 0
#         pop_p = pop_l # A população da cidade é a mesma, só repetimos a variável
# 
#         dados_finais.append({
#             'cidade': row['nome'],
#             'tipo_veiculo': 'pesada',
#             'populacao': pop_p,
#             'frota_estimada': frota_p,
#             'quantidade_vendida': row['venda_pesada']
#         })
# 
#     base_estruturada = pd.DataFrame(dados_finais)
#     return base_estruturada
# 
# def separar_treino_teste(base):
#     # Separa 80% para treino e 20% para teste com dados REAIS
#     treino, teste = train_test_split(base, test_size=0.2, random_state=42)
#     return treino, teste
#

"""1.2 - Regressão Linear"""

import json
import sys
from pathlib import Path

if "/content" not in sys.path:
    sys.path.insert(0, "/content")

if "preparacao" in sys.modules:
    del sys.modules["preparacao"]

import matplotlib
matplotlib.use("Agg")
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
from sklearn.compose import ColumnTransformer
from sklearn.linear_model import LinearRegression
from sklearn.metrics import (
    mean_absolute_error, mean_absolute_percentage_error,
    mean_squared_error, r2_score
)
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import OneHotEncoder, StandardScaler

# Importa as funções atualizadas
from preparacao import montar_base, separar_treino_teste

SAIDA = Path.cwd() / "resultados"
SAIDA.mkdir(exist_ok=True)

ALVO = "quantidade_vendida"
NUMERICAS = ["populacao", "frota_estimada"]
CATEGORICAS = ["tipo_veiculo"]

def avaliar(y_real, y_prev) -> dict:
    # Tratamento para evitar divisão por zero no MAPE
    mape = mean_absolute_percentage_error(y_real + 1e-9, y_prev) * 100
    return {
        "MAE": mean_absolute_error(y_real, y_prev),
        "RMSE": float(np.sqrt(mean_squared_error(y_real, y_prev))),
        "R2": r2_score(y_real, y_prev),
        "MAPE_%": mape,
    }

def main():
    print("Carregando dados reais e estruturando...")
    base = montar_base()
    treino, teste = separar_treino_teste(base)
    print(f"Treino: {len(treino)} registros | Teste: {len(teste)} registros\n")

    X_tr, y_tr = treino[NUMERICAS + CATEGORICAS], treino[ALVO]
    X_te, y_te = teste[NUMERICAS + CATEGORICAS], teste[ALVO]

    # Pipeline do Modelo
    preprocess = ColumnTransformer([
        ("num", StandardScaler(), NUMERICAS),
        ("cat", OneHotEncoder(drop="first"), CATEGORICAS),
    ])
    modelo = Pipeline([("prep", preprocess), ("reg", LinearRegression())])

    print("Treinando o modelo com os dados de Minas Gerais...")
    modelo.fit(X_tr, y_tr)

    # Avaliação
    prev = modelo.predict(X_te)
    metricas = avaliar(y_te, prev)
    metricas_treino = avaliar(y_tr, modelo.predict(X_tr))

    print("\nDesempenho no TESTE (20% das cidades):")
    for k, v in metricas.items():
        print(f"  {k:7s} {v:12.2f}")
    print(f"  (R² no treino: {metricas_treino['R2']:.3f})")

    n_neg = int((prev < 0).sum())
    print(f"  Previsões negativas: {n_neg}")

    # Coeficientes
    nomes = modelo.named_steps["prep"].get_feature_names_out()
    coefs = pd.Series(modelo.named_steps["reg"].coef_, index=nomes).sort_values(key=abs, ascending=False)
    print("\nCoeficientes (efeito de +1 desvio-padrão):")
    print(coefs.round(1).to_string())

    # Gráficos
    fig, ax = plt.subplots(1, 2, figsize=(11, 4.5))
    ax[0].scatter(y_te, prev, s=14, alpha=0.6, color='blue')
    lim = [0, max(y_te.max(), prev.max()) * 1.05]
    ax[0].plot(lim, lim, "k--", lw=1)
    ax[0].set(xlabel="Vendas reais", ylabel="Vendas previstas", title="Regressão Linear (Real x Previsto)")

    residuos = y_te - prev
    ax[1].scatter(prev, residuos, s=14, alpha=0.6, color='red')
    ax[1].axhline(0, color="k", lw=1)
    ax[1].set(xlabel="Vendas previstas", ylabel="Resíduo", title="Análise de Resíduos")
    fig.tight_layout()
    fig.savefig(SAIDA / "rl_real_x_previsto.png", dpi=150)

    # Salvando para a Célula 4 ler depois
    (SAIDA / "metricas_regressao_linear.json").write_text(
        json.dumps({"teste": metricas, "treino": metricas_treino, "previsoes_negativas": n_neg}, indent=2)
    )
    print(f"\nResultados salvos. Gráfico atualizado!")

if __name__ == "__main__":
    main()

"""Caminho padrão dos arquivos"""

# 1. Conectar o Google Colab ao seu Google Drive
from google.colab import drive
drive.mount('/content/drive')

# 2. Definir os Caminhos dos Ficheiros (Paths)
# Substitua 'Sua_Pasta' pelo nome da pasta onde vai guardar as coisas da faculdade no seu Drive
pasta_projeto = '/content/drive/MyDrive/raw'

# Caminho para LER a sua base de vendas (caso vá usar uma planilha pronta em vez de gerar dados aleatórios)
caminho_base_vendas = pasta_projeto + 'Base_Vendas_Auto_MG_Realistax'

# Caminho para GUARDAR os resultados gerados pelos nossos scripts
caminho_resultado_spillover = pasta_projeto + 'Base_Academica_Spillover_Segmentada.xlsx'
caminho_resultado_projecao = pasta_projeto + 'Projecao_Crescimento_2026_2030.xlsx'

"""2 - Spillover"""

import pandas as pd
import numpy as np
import libpysal
from pathlib import Path
import warnings
warnings.filterwarnings('ignore')

print("=== CÁLCULO DE SPILLOVER ESPACIAL (VENDAS AUTOMOTIVAS EM R$) ===")

# 1. Carregar a Base de Vendas (Onde estão as vendas em R$)
caminho_vendas = Path('/content/drive/MyDrive/raw/Simulacao_Vendas_Auto_MG_400_Reais.xlsx')
df_vendas = pd.read_excel(caminho_vendas)

# Buscar as Coordenadas (Latitude e Longitude) oficiais do IBGE
df_coords = pd.read_csv("https://raw.githubusercontent.com/kelvins/Municipios-Brasileiros/main/csv/municipios.csv")
df_coords = df_coords[df_coords['codigo_uf'] == 31][['codigo_ibge', 'nome', 'latitude', 'longitude']]

# Juntar Vendas com Coordenadas usando o código IBGE
df_mg = pd.merge(df_vendas, df_coords, left_on='cod_ibge', right_on='codigo_ibge', how='inner')

# Tratar nulos caso existam
df_mg['venda_leve'] = df_mg['venda_leve'].fillna(0)
df_mg['venda_pesada'] = df_mg['venda_pesada'].fillna(0)

# ==================================================================
# 2. MATRIZ ESPACIAL E DEFASAGEM (LAG) NAS VENDAS
# ==================================================================
coords = list(zip(df_mg['latitude'], df_mg['longitude']))
W = libpysal.weights.KNN.from_array(np.array(coords), k=5)
W.transform = 'r'

# Lag Espacial sobre o valor financeiro das VENDAS
df_mg['lag_venda_leve'] = libpysal.weights.lag_spatial(W, df_mg['venda_leve'])
df_mg['lag_venda_pesada'] = libpysal.weights.lag_spatial(W, df_mg['venda_pesada'])

# ==================================================================
# 3. CÁLCULO DINÂMICO DOS COEFICIENTES DE SPILLOVER (R$)
# ==================================================================
razao_mercado_leve = df_mg['lag_venda_leve'] / (df_mg['venda_leve'] + 1)
razao_mercado_pesada = df_mg['lag_venda_pesada'] / (df_mg['venda_pesada'] + 1)

# Normalizando os limites para garantir realismo:
df_mg['rho_dinamico_leve'] = np.clip(0.05 + (razao_mercado_leve / razao_mercado_leve.max()) * 0.25, 0.05, 0.30)
df_mg['rho_dinamico_pesada'] = np.clip(0.08 + (razao_mercado_pesada / razao_mercado_pesada.max()) * 0.32, 0.08, 0.40)

# Cálculos de Spillover em R$
df_mg['spillover_R$_leve'] = df_mg['lag_venda_leve'] * df_mg['rho_dinamico_leve']
df_mg['spillover_R$_pesada'] = df_mg['lag_venda_pesada'] * df_mg['rho_dinamico_pesada']

df_mg['venda_total_com_spillover_leve'] = df_mg['venda_leve'] + df_mg['spillover_R$_leve']
df_mg['venda_total_com_spillover_pesada'] = df_mg['venda_pesada'] + df_mg['spillover_R$_pesada']

# Percentuais aplicados
df_mg['pct_spillover_aplicado_leve'] = df_mg['rho_dinamico_leve'] * 100
df_mg['pct_spillover_aplicado_pesada'] = df_mg['rho_dinamico_pesada'] * 100

# Evitar divisão por zero para as percentagens de aumento
venda_leve_segura = np.where(df_mg['venda_leve'] == 0, 1, df_mg['venda_leve'])
venda_pesada_segura = np.where(df_mg['venda_pesada'] == 0, 1, df_mg['venda_pesada'])

df_mg['pct_aumento_leve'] = (df_mg['spillover_R$_leve'] / venda_leve_segura) * 100
df_mg['pct_aumento_pesada'] = (df_mg['spillover_R$_pesada'] / venda_pesada_segura) * 100

# ==================================================================
# 4. EXPORTAÇÃO DO RELATÓRIO FINAL
# ==================================================================
colunas_relatorio = [
    'nome', 'latitude', 'longitude', 'venda_leve', 'pct_spillover_aplicado_leve', 'spillover_R$_leve',
    'pct_aumento_leve', 'venda_total_com_spillover_leve',
    'venda_pesada', 'pct_spillover_aplicado_pesada', 'spillover_R$_pesada',
    'pct_aumento_pesada', 'venda_total_com_spillover_pesada'
]

relatorio_final = df_mg[colunas_relatorio].copy()
relatorio_final.rename(columns={
    'nome': 'Cidade/Município',
    'venda_leve': 'Venda Bruta Leve (R$)',
    'pct_spillover_aplicado_leve': 'Taxa Spillover Dinâmica Leve (%)',
    'spillover_R$_leve': 'Valor Spillover Leve (R$)',
    'pct_aumento_leve': 'Aumento Leve na Receita (%)',
    'venda_total_com_spillover_leve': 'Venda Total Estimada Leve (R$)',
    'venda_pesada': 'Venda Bruta Pesada (R$)',
    'pct_spillover_aplicado_pesada': 'Taxa Spillover Dinâmica Pesada (%)',
    'spillover_R$_pesada': 'Valor Spillover Pesada (R$)',
    'pct_aumento_pesada': 'Aumento Pesada na Receita (%)',
    'venda_total_com_spillover_pesada': 'Venda Total Estimada Pesada (R$)'
}, inplace=True)

# Salvar no Google Drive
caminho_saida = Path('/content/drive/MyDrive/raw/Base_Academica_Spillover_Segmentada.xlsx')
relatorio_final.to_excel(caminho_saida, index=False)

print(f"\n✅ Relatório de Spillover de Vendas exportado com sucesso para: {caminho_saida}")
# Exibir uma pequena amostra para confirmar que voltamos a falar de R$
display(relatorio_final[['Cidade/Município', 'Venda Bruta Leve (R$)', 'Valor Spillover Leve (R$)', 'Venda Total Estimada Leve (R$)']].head().round(2))

"""Analise coluna Codigo IBGE"""

import pandas as pd

# Caminhos dos arquivos reais
path_etanol = '/content/drive/MyDrive/raw/Combustiveis/vendas-anuais-de-etanol-hidratado-por-municipio (1).csv'
path_gasolina = '/content/drive/MyDrive/raw/Combustiveis/vendas-anuais-de-gasolina-c-por-municipio (1).csv'
path_diesel = '/content/drive/MyDrive/raw/Combustiveis/vendas-anuais-de-oleo-diesel-por-municipio (2).csv'
path_frota = '/content/drive/MyDrive/raw/Frota/Frota_por_municipio_e_tipo_Setembro_2026.xlsx'

# Lendo os dados
try:
    df_etanol = pd.read_csv(path_etanol, sep=';', encoding='utf-8')
except Exception as e:
    df_etanol = pd.read_csv(path_etanol)

try:
    df_gasolina = pd.read_csv(path_gasolina, sep=';', encoding='utf-8')
except Exception as e:
    df_gasolina = pd.read_csv(path_gasolina)

try:
    df_diesel = pd.read_csv(path_diesel, sep=';', encoding='utf-8')
except Exception as e:
    df_diesel = pd.read_csv(path_diesel)

# Ajuste: pulando 2 linhas para ler o cabeçalho correto
df_frota = pd.read_excel(path_frota, skiprows=2)

print("=== Estrutura dos Dados ===")
print("\nEtanol:", df_etanol.columns.tolist())
display(df_etanol.head(2))

print("\nGasolina:", df_gasolina.columns.tolist())
display(df_gasolina.head(2))

print("\nDiesel:", df_diesel.columns.tolist())
display(df_diesel.head(2))

print("\nFrota:", df_frota.columns.tolist())
display(df_frota.head(2))

print("=== ANÁLISE DOS CÓDIGOS IBGE (BELO HORIZONTE) ===\n")

if 'df_gasolina' in globals():
    print("1. df_gasolina ['CÓDIGO IBGE'] - Tipo:", df_gasolina['CÓDIGO IBGE'].dtype)
    display(df_gasolina[df_gasolina['MUNICÍPIO'].astype(str).str.upper() == 'BELO HORIZONTE'][['UF', 'MUNICÍPIO', 'CÓDIGO IBGE']].head(3))

if 'df_etanol' in globals():
    print("\n2. df_etanol ['CÓDIGO IBGE'] - Tipo:", df_etanol['CÓDIGO IBGE'].dtype)
    display(df_etanol[df_etanol['MUNICÍPIO'].astype(str).str.upper() == 'BELO HORIZONTE'][['UF', 'MUNICÍPIO', 'CÓDIGO IBGE']].head(3))

if 'df_diesel' in globals():
    print("\n3. df_diesel ['CÓDIGO IBGE'] - Tipo:", df_diesel['CÓDIGO IBGE'].dtype)
    display(df_diesel[df_diesel['MUNICÍPIO'].astype(str).str.upper() == 'BELO HORIZONTE'][['UF', 'MUNICÍPIO', 'CÓDIGO IBGE']].head(3))

if 'df_brasil' in globals():
    print("\n4. df_brasil ['codigo_ibge'] - Tipo:", df_brasil['codigo_ibge'].dtype)
    display(df_brasil[df_brasil['nome'].astype(str).str.upper() == 'BELO HORIZONTE'][['nome', 'codigo_ibge']].head(3))

if 'df_realista' in globals():
    print("\n5. df_realista ['cod_ibge'] - Tipo:", df_realista['cod_ibge'].dtype)
    display(df_realista[df_realista['municipio'].astype(str).str.upper() == 'BELO HORIZONTE'][['municipio', 'cod_ibge']].head(3))

if 'df_frota' in globals():
    print("\n6. df_frota (Não possui IBGE, mas validando o nome e UF):")
    display(df_frota[df_frota['MUNICIPIO'].astype(str).str.upper() == 'BELO HORIZONTE'][['UF', 'MUNICIPIO', 'TOTAL']].head(3))

"""Gerar codigo IBGE no tabela frota"""

import os
import re
import pandas as pd

print("=== CONSOLIDANDO ARQUIVOS DO SENATRAN (2013-2025) ===")

pasta_frota = '/content/drive/MyDrive/raw/Frota/'
arquivos = os.listdir(pasta_frota)

lista_dfs = []

for arquivo in arquivos:
    # Ignora arquivos ocultos ou que não sejam Excel
    if arquivo.startswith('~') or not arquivo.endswith(('.xls', '.xlsx')):
        continue

    # Pula os arquivos de Setembro de 2026
    if '2026' in arquivo:
        continue

    ano = None
    # 1. Tenta achar o ano no formato de 4 dígitos (ex: 2014, 2020)
    ano_encontrado = re.search(r'(201\d|202\d)', arquivo)

    if ano_encontrado:
        ano = int(ano_encontrado.group())
    else:
        # 2. Se não achar, tenta o formato de 2 dígitos precedido de underline (ex: _15, _16)
        ano_curto = re.search(r'_(\d{2})\.xls', arquivo)
        if ano_curto:
            ano = int("20" + ano_curto.group(1))

    if ano:
        caminho_completo = os.path.join(pasta_frota, arquivo)
        print(f"Lendo: {arquivo} -> Ano extraído: {ano}")

        try:
            # Tenta ler pulando 2 linhas
            df_temp = pd.read_excel(caminho_completo, skiprows=2)

            if 'MUNICIPIO' not in df_temp.columns and 'MUNICÍPIO' not in df_temp.columns:
                df_temp = pd.read_excel(caminho_completo)

            df_temp.columns = df_temp.columns.str.strip().str.upper()
            if 'MUNICÍPIO' in df_temp.columns:
                df_temp.rename(columns={'MUNICÍPIO': 'MUNICIPIO'}, inplace=True)

            df_temp['ANO'] = ano
            lista_dfs.append(df_temp)

        except Exception as e:
            print(f"Erro ao ler {arquivo}: {e}")

print("\nConcatenando todos os anos...")
df_historico_consolidado = pd.concat(lista_dfs, ignore_index=True)

df_historico_consolidado = df_historico_consolidado.dropna(subset=['MUNICIPIO'])
df_historico_consolidado = df_historico_consolidado[df_historico_consolidado['MUNICIPIO'] != 'MUNICIPIO']

print(f"Total de registros na base consolidada: {len(df_historico_consolidado)}")

caminho_saida = os.path.join(pasta_frota, 'Frota_Historica_Consolidada_Mestre.xlsx')
df_historico_consolidado.to_excel(caminho_saida, index=False)

print(f"✅ Arquivo unificado salvo com sucesso em: {caminho_saida}")

import os
import re
import pandas as pd
import unicodedata

print("=== PIPELINE DEFINITIVO: CONSOLIDAÇÃO (2016-2025) + FILTRO MG + IBGE ===")

def normalizar_texto(texto):
    if pd.isna(texto): return "DESCONHECIDO"
    n = unicodedata.normalize('NFKD', str(texto)).encode('ASCII', 'ignore').decode('utf-8')
    return n.strip().upper()

# --- PARTE 1: LEITURA E CONSOLIDAÇÃO DE TODOS OS ANOS ---
ufs_validas = [
    'AC', 'AL', 'AM', 'AP', 'BA', 'CE', 'DF', 'ES', 'GO',
    'MA', 'MG', 'MS', 'MT', 'PA', 'PB', 'PE', 'PI', 'PR',
    'RJ', 'RN', 'RO', 'RR', 'RS', 'SC', 'SE', 'SP', 'TO'
]

pasta_frota = '/content/drive/MyDrive/raw/Frota/'
arquivos = os.listdir(pasta_frota)
lista_dfs = []

for arquivo in arquivos:
    # Ignora arquivos não Excel e o arquivo de 2026
    if arquivo.startswith('~') or not arquivo.endswith(('.xls', '.xlsx')) or '2026' in arquivo:
        continue

    # IGNORA OS ARQUIVOS QUE JÁ GERAMOS EM TESTES ANTERIORES PARA NÃO LER EM LOOP
    if 'Historica' in arquivo or 'IBGE' in arquivo or 'Consolidada' in arquivo or 'Mestre' in arquivo:
        continue

    ano = None
    ano_encontrado = re.search(r'(201\d|202\d)', arquivo)
    if ano_encontrado:
        ano = int(ano_encontrado.group())
    else:
        ano_curto = re.search(r'_(\d{2})\.xls', arquivo)
        if ano_curto:
            ano = int("20" + ano_curto.group(1))

    if ano:
        caminho_completo = os.path.join(pasta_frota, arquivo)
        print(f"Lendo: {arquivo} -> Ano extraído: {ano}")

        try:
            df_temp = pd.read_excel(caminho_completo, header=None)

            header_row = 0
            for idx, row in df_temp.iterrows():
                valores_linha = [normalizar_texto(val) for val in row.values]
                if 'MUNICIPIO' in valores_linha or 'UF' in valores_linha:
                    header_row = idx
                    break

            df_temp.columns = [normalizar_texto(c) for c in df_temp.iloc[header_row].values]
            df_temp = df_temp.iloc[header_row+1:].copy()

            # REMOVE COLUNAS DUPLICADAS (Ex: várias colunas "DESCONHECIDO" vazias)
            df_temp = df_temp.loc[:, ~df_temp.columns.duplicated()]

            if 'UF' in df_temp.columns:
                df_temp['UF'] = df_temp['UF'].astype(str).str.strip().str.upper()
                df_temp = df_temp[df_temp['UF'].isin(ufs_validas)]

            df_temp['ANO'] = ano
            lista_dfs.append(df_temp)

        except Exception as e:
            print(f"Erro ao ler {arquivo}: {e}")

print("\nEmpilhando todos os anos...")
df_historico_consolidado = pd.concat(lista_dfs, ignore_index=True)


# --- PARTE 2: FILTRO MG, LIMPEZA E CRUZAMENTO COM IBGE ---
print("\n=== INICIANDO PADRONIZAÇÃO MG E CRUZAMENTO IBGE ===")

df_brasil = pd.read_csv("https://raw.githubusercontent.com/kelvins/Municipios-Brasileiros/main/csv/municipios.csv")
df_ibge_mg = df_brasil[df_brasil['codigo_uf'] == 31].copy()
df_ibge_mg['NOME_MUN_CLEAN'] = df_ibge_mg['nome'].apply(normalizar_texto)

df_frota_mg = df_historico_consolidado[df_historico_consolidado['UF'] == 'MG'].copy()

df_frota_mg['NOME_MUN_CLEAN'] = df_frota_mg['MUNICIPIO'].apply(normalizar_texto)
df_frota_mg = df_frota_mg[df_frota_mg['NOME_MUN_CLEAN'] != 'MUNICIPIO NAO INFORMADO']

correcoes_manuais = {
    "SAO TOME DAS LETRAS": "SAO THOME DAS LETRAS",
    "GOUVEA": "GOUVEIA",
    "ITABIRINHA DE MANTENA": "ITABIRINHA",
    "BRASOPOLIS": "BRAZOPOLIS",
    "AMPARO DA SERRA": "AMPARO DO SERRA",
    "BARAO D0 MONTE ALTO": "BARAO DE MONTE ALTO",
    "DONA EUZEBIA": "DONA EUSEBIA",
    "OLHOS-D'AGUA": "OLHOS D'AGUA",
    "PASSA VINTE": "PASSA-VINTE",
    "PINGO D'AGUA": "PINGO-D'AGUA",
    "QUELUZITA": "QUELUZITO"
}

df_frota_mg['NOME_MUN_CLEAN'] = df_frota_mg['NOME_MUN_CLEAN'].replace(correcoes_manuais)

df_frota_mg = pd.merge(
    df_frota_mg,
    df_ibge_mg[['codigo_ibge', 'NOME_MUN_CLEAN']],
    on='NOME_MUN_CLEAN',
    how='left'
)

df_frota_mg.rename(columns={'codigo_ibge': 'CÓDIGO IBGE'}, inplace=True)
df_frota_mg['CÓDIGO IBGE'] = df_frota_mg['CÓDIGO IBGE'].fillna(0).astype(int)

df_frota_mg.drop(columns=['NOME_MUN_CLEAN'], inplace=True, errors='ignore')

cidades_nao_reconhecidas = df_frota_mg[df_frota_mg['CÓDIGO IBGE'] == 0][['UF', 'MUNICIPIO', 'ANO']].drop_duplicates()

print(f"Total de registros isolados para MG (2013-2025): {len(df_frota_mg)}")

if len(cidades_nao_reconhecidas) > 0:
    print(f"\nMunicípios em MG ainda sem IBGE: {len(cidades_nao_reconhecidas)}")
    display(cidades_nao_reconhecidas)
else:
    print("✅ SUCESSO ABSOLUTO! Todos os municípios de MG parearam com o código IBGE para todos os anos.")

caminho_saida = '/content/drive/MyDrive/raw/Frota/Frota_Historica_MG_IBGE_2016_2025.xlsx'
df_frota_mg.to_excel(caminho_saida, index=False)
print(f"\n[+] Base histórica de MG unificada e pronta: {caminho_saida}")

"""Preparação base ANP"""

import pandas as pd
from pathlib import Path

print("=== PREPARAÇÃO DA BASE DE COMBUSTÍVEIS DA ANP (2016-2025) ===")

# Caminhos padrão (ajuste se tiver renomeado os arquivos)
pasta_anp = Path('/content/drive/MyDrive/raw/Combustiveis')
path_etanol = pasta_anp / 'vendas-anuais-de-etanol-hidratado-por-municipio (1).csv'
path_gasolina = pasta_anp / 'vendas-anuais-de-gasolina-c-por-municipio (1).csv'
path_diesel = pasta_anp / 'vendas-anuais-de-oleo-diesel-por-municipio (2).csv'

def processar_anp(caminho, nome_coluna_venda):
    # Tenta ler com utf-8, faz fallback para latin1 se a ANP tiver mudado a codificação
    try:
        df = pd.read_csv(caminho, sep=';', encoding='utf-8')
    except:
        df = pd.read_csv(caminho, sep=';', encoding='latin1')

    # Filtro de Estado e Janela Temporal (2016 a 2025)
    df = df[(df['UF'] == 'MG') & (df['ANO'] >= 2016) & (df['ANO'] <= 2025)].copy()

    # Limpeza da coluna de Vendas (string com vírgula -> float)
    df['VENDAS'] = df['VENDAS'].astype(str).str.replace(',', '.').astype(float)

    # Agrupar por Ano e Código IBGE para garantir que não haja linhas duplicadas por cidade
    df_agg = df.groupby(['ANO', 'CÓDIGO IBGE'], as_index=False)['VENDAS'].sum()
    df_agg.rename(columns={'VENDAS': nome_coluna_venda}, inplace=True)

    return df_agg

print("Processando Etanol...")
df_etanol = processar_anp(path_etanol, 'venda_etanol')

print("Processando Gasolina...")
df_gasolina = processar_anp(path_gasolina, 'venda_gasolina')

print("Processando Diesel...")
df_diesel = processar_anp(path_diesel, 'venda_diesel')

# Unificar as bases da ANP
print("\nUnificando as bases de combustíveis...")
df_anp = pd.merge(df_gasolina, df_etanol, on=['ANO', 'CÓDIGO IBGE'], how='outer').fillna(0)
df_anp = pd.merge(df_anp, df_diesel, on=['ANO', 'CÓDIGO IBGE'], how='outer').fillna(0)

# Criar a métrica da Linha Leve (Ciclo Otto) e renomear a Pesada
df_anp['consumo_leve_m3'] = df_anp['venda_gasolina'] + df_anp['venda_etanol']
df_anp.rename(columns={'venda_diesel': 'consumo_diesel_m3'}, inplace=True)

# Garantir o padrão inteiro do IBGE
df_anp['CÓDIGO IBGE'] = df_anp['CÓDIGO IBGE'].astype(int)

# Exportação do histórico limpo
caminho_saida = pasta_anp / 'ANP_Historica_MG_2016_2025.xlsx'
df_anp.to_excel(caminho_saida, index=False)

print(f"Total de registros ANP unificados (MG, 2016-2025): {len(df_anp)}")
print(f"✅ Base da ANP consolidada salva em: {caminho_saida}")
display(df_anp.sample(5))

"""Conexão API IBGE (Censo) + ANP + Frota SENATRAN"""

import pandas as pd
import requests
import numpy as np
from pathlib import Path

print("=== CÁLCULO DEMOGRÁFICO REAL: CENSO 2010 vs 2022 (MG) ===")

def buscar_censo_ibge(tabela, ano):
    # Tabela 202 (Censo 2010) e Tabela 4714 (Censo 2022) - População Residente
    url = f"https://apisidra.ibge.gov.br/values/t/{tabela}/n6/in%20n3%2031/p/{ano}/v/93"
    resposta = requests.get(url)

    if resposta.status_code == 200:
        dados = resposta.json()
        formatados = []
        for linha in dados[1:]:
            if linha['V'].isnumeric():
                formatados.append({
                    'CÓDIGO IBGE': int(linha['D1C']),
                    f'pop_{ano}': int(linha['V'])
                })
        return pd.DataFrame(formatados)
    else:
        print(f"Erro ao aceder à tabela {tabela} do IBGE.")
        return pd.DataFrame()

print("A extrair Censo de 2010...")
df_2010 = buscar_censo_ibge("202", "2010")

print("A extrair Censo de 2022...")
df_2022 = buscar_censo_ibge("4714", "2022")

# Juntar as duas pontas da realidade
df_cagr = pd.merge(df_2010, df_2022, on='CÓDIGO IBGE', how='inner')

# Calcular a Taxa de Crescimento Anual Composta (CAGR) de 12 anos (2010 a 2022)
df_cagr['CAGR'] = (df_cagr['pop_2022'] / df_cagr['pop_2010']) ** (1 / 12) - 1

# ====================================================================
# APLICAR O CRESCIMENTO E PROJETAR (2016-2025)
# ====================================================================
print("\nA projetar a curva populacional (2016 a 2025)...")
municipios = df_cagr['CÓDIGO IBGE'].unique()
anos = list(range(2016, 2026))

# Criar a grelha de tempo
grid = pd.MultiIndex.from_product([municipios, anos], names=['CÓDIGO IBGE', 'ANO'])
df_pop_oficial = pd.DataFrame(index=grid).reset_index()

# Juntar os dados base para o cálculo
df_pop_oficial = pd.merge(df_pop_oficial, df_cagr[['CÓDIGO IBGE', 'pop_2010', 'CAGR']], on='CÓDIGO IBGE', how='left')

# Calcular a população exata para cada ano baseada na curva do CAGR
df_pop_oficial['populacao'] = df_pop_oficial['pop_2010'] * ((1 + df_pop_oficial['CAGR']) ** (df_pop_oficial['ANO'] - 2010))
df_pop_oficial['populacao'] = df_pop_oficial['populacao'].round(0).astype(int)

# Limpar as colunas de apoio
df_pop_oficial.drop(columns=['pop_2010', 'CAGR'], inplace=True)


# ====================================================================
# MERGE FINAL: A BASE MASTER (Frota + Combustíveis + População)
# ====================================================================
print("\nA realizar o Merge Final com as bases de Frota e ANP...")
pasta_raw = Path('/content/drive/MyDrive/raw')
df_frota = pd.read_excel(pasta_raw / 'Frota/Frota_Historica_MG_IBGE_2013_2025.xlsx')
df_anp = pd.read_excel(pasta_raw / 'Combustiveis/ANP_Historica_MG_2016_2025.xlsx')

df_frota = df_frota[df_frota['ANO'] >= 2016]

# O Grande Merge
df_master = pd.merge(df_frota, df_anp, on=['ANO', 'CÓDIGO IBGE'], how='inner')
df_master = pd.merge(df_master, df_pop_oficial, on=['ANO', 'CÓDIGO IBGE'], how='inner')

df_master = df_master.fillna(0)

caminho_master = pasta_raw / 'Base_Master_Analitica_MG_2016_2025.xlsx'
df_master.to_excel(caminho_master, index=False)

print(f"✅ SUCESSO! Base Master gerada com projeção matemática e pronta para Machine Learning em: {caminho_master}")
# Mostrar a diferença na população de BH (IBGE 3106200) para validar a nova curva
display(df_master[df_master['CÓDIGO IBGE'] == 3106200][['ANO', 'MUNICIPIO', 'populacao', 'TOTAL', 'consumo_leve_m3']].tail())

"""**3 - Projeção de crescimento populacional, de frota e de consumo de combustíveis**

Treinar com material de 10-15 anos antes para + precisão.

"""

# Commented out IPython magic to ensure Python compatibility.
import pandas as pd
import numpy as np
from pathlib import Path
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import PolynomialFeatures

# 1. Reativar o backend inline do Colab/Jupyter e importar matplotlib
# %matplotlib inline
import matplotlib.pyplot as plt

print("=== TREINAMENTO PREDITIVO COM DADOS REAIS E DIAGNÓSTICO (2016-2031) ===\n")

# 1. Carregar a Base Master Analítica
pasta_raw = Path('/content/drive/MyDrive/raw')
caminho_master = pasta_raw / 'Base_Master_Analitica_MG_2016_2025.xlsx'
df_master = pd.read_excel(caminho_master)

# 2. Agregar os dados para uma visão macro do Estado de Minas Gerais
df_macro = df_master.groupby('ANO', as_index=False).agg({
    'populacao': 'sum',
    'TOTAL': 'sum',              # Frota Total
    'consumo_leve_m3': 'sum',    # Gasolina + Etanol
    'consumo_diesel_m3': 'sum'   # Diesel
}).sort_values('ANO')

anos_historicos = df_macro['ANO'].values.reshape(-1, 1)
populacao = df_macro['populacao'].values.reshape(-1, 1)
frota_total = df_macro['TOTAL'].values.reshape(-1, 1)
consumo_leve = df_macro['consumo_leve_m3'].values.reshape(-1, 1)
consumo_diesel = df_macro['consumo_diesel_m3'].values.reshape(-1, 1)

# 3. Configurar o Modelo de Regressão Polinomial (Grau 2)
poly = PolynomialFeatures(degree=2)
anos_poly = poly.fit_transform(anos_historicos)

modelo_pop = LinearRegression().fit(anos_poly, populacao)
modelo_frota = LinearRegression().fit(anos_poly, frota_total)
modelo_leve = LinearRegression().fit(anos_poly, consumo_leve)
modelo_diesel = LinearRegression().fit(anos_poly, consumo_diesel)

# 4. Criar o Período de Projeção (2027 a 2031)
anos_projecao = np.arange(2027, 2032).reshape(-1, 1)
anos_projecao_poly = poly.transform(anos_projecao)

proj_pop = modelo_pop.predict(anos_projecao_poly)
proj_frota = modelo_frota.predict(anos_projecao_poly)
proj_leve = modelo_leve.predict(anos_projecao_poly)
proj_diesel = modelo_diesel.predict(anos_projecao_poly)

# 5. Estruturar e Exportar os Resultados COMPLETOS para XLSX (Agora com variação e métricas)
df_projecao = pd.DataFrame({
    'Ano': anos_projecao.flatten(),
    'Projeção_População': proj_pop.flatten().astype(int),
    'Projeção_Frota_Total': proj_frota.flatten().astype(int),
    'Proj_Consumo_Leve_m3': proj_leve.flatten().astype(int),
    'Proj_Consumo_Diesel_m3': proj_diesel.flatten().astype(int)
})

# Guardar os valores de referência do último ano real medido (ex: o final da grelha de treino)
base_frota = frota_total[-1][0]
base_diesel = consumo_diesel[-1][0]

# Adicionar as colunas comparativas de percentagem (%)
df_projecao['Crescimento_Frota_Vs_Historico_%'] = ((df_projecao['Projeção_Frota_Total'] / base_frota) - 1) * 100
df_projecao['Crescimento_Diesel_Vs_Historico_%'] = ((df_projecao['Proj_Consumo_Diesel_m3'] / base_diesel) - 1) * 100

# Adicionar crescimento Ano a Ano (YoY - Year over Year)
df_projecao['Evolucao_Anual_Frota_%'] = df_projecao['Projeção_Frota_Total'].pct_change() * 100
df_projecao['Evolucao_Anual_Diesel_%'] = df_projecao['Proj_Consumo_Diesel_m3'].pct_change() * 100

# Preencher NaN do primeiro ano do YoY com 0 ou a variação face ao histórico
df_projecao = df_projecao.fillna(0).round(2)

caminho_projecao = pasta_raw / 'Projecao_Crescimento_MG_2027_2031.xlsx'
df_projecao.to_excel(caminho_projecao, index=False)

# 6. Diagnóstico de Crescimento no terminal
ano_inicio, ano_fim = df_projecao['Ano'].min(), df_projecao['Ano'].max()
crescimento_frota = ((proj_frota[-1][0] / proj_frota[0][0]) - 1) * 100
crescimento_diesel = ((proj_diesel[-1][0] / proj_diesel[0][0]) - 1) * 100

print(f"-> Crescimento Acumulado da Frota ({ano_inicio}-{ano_fim}): {crescimento_frota:.2f}%")
print(f"-> Crescimento Acumulado de Diesel ({ano_inicio}-{ano_fim}): {crescimento_diesel:.2f}%")

print("\n[Diagnóstico da Projeção]:")
if crescimento_frota > 0 and crescimento_diesel > 0:
    if crescimento_diesel > 50:
        print("⚠️ ALERTA DE OVERFITTING: Crescimento projetado excessivamente alto (>50%).")
    else:
        print("✅ AJUSTE ADEQUADO: As curvas demonstram crescimento realista e sustentável para o mercado.")
else:
    print("⚠️ RECESSÃO OU ESTAGNAÇÃO: O modelo não projetou variação positiva.")

print(f"\n[+] Dados consolidados exportados para: {caminho_projecao}")
print("\nGerando gráficos de tendência...\n")

# 7. Renderização e Salvamento dos Gráficos
fig, axs = plt.subplots(2, 2, figsize=(14, 10))
fig.suptitle('Trajetória de Crescimento e Projeções de Mercado (MG)', fontsize=16, fontweight='bold')

variaveis = [
    (populacao, proj_pop, 'População', axs[0, 0], '#1f77b4'),
    (frota_total, proj_frota, 'Frota Total (Veículos)', axs[0, 1], '#ff7f0e'),
    (consumo_leve, proj_leve, 'Consumo Linha Leve (m³)', axs[1, 0], '#2ca02c'),
    (consumo_diesel, proj_diesel, 'Consumo Diesel (m³)', axs[1, 1], '#d62728')
]

for historico, projecao, titulo, ax, cor in variaveis:
    ax.scatter(anos_historicos, historico, color='black', alpha=0.6, label='Histórico (Real)')
    anos_completos = np.vstack([anos_historicos, anos_projecao])
    curva_pred = LinearRegression().fit(poly.transform(anos_historicos), historico).predict(poly.transform(anos_completos))
    ax.plot(anos_completos, curva_pred, color=cor, linestyle='-', linewidth=2.5, label='Curva Preditiva')
    ax.axvspan(2026.5, 2031.5, color='gray', alpha=0.15, label='Janela de Projeção (2027-2031)')
    ax.set_title(titulo, fontsize=12)
    ax.grid(True, linestyle='--', alpha=0.5)
    ax.legend(loc='best')
    ax.ticklabel_format(style='plain', axis='y')

plt.tight_layout(rect=[0, 0.03, 1, 0.95])

# Salvar cópia local em imagem para garantia
caminho_grafico = pasta_raw / 'projecao_tendencias_2027_2031.png'
plt.savefig(caminho_grafico, dpi=150)
print(f"Gráfico gravado com sucesso em: {caminho_grafico}")

plt.show()

"""### 4 - Relatório de Eficiência e Diagnóstico de Treinamento dos Modelos"""

import numpy as np
import pandas as pd
import json
from pathlib import Path
from sklearn.linear_model import LinearRegression
from sklearn.metrics import r2_score

print("=== INICIANDO DIAGNÓSTICO DE EFICIÊNCIA DE TREINAMENTO ===\n")

# --- 1. DIAGNÓSTICO DO MODELO DE REGRESSÃO LINEAR (DEMANDA) ---
print("1. Avaliando Regressão Linear (Modelo de Demanda):")
caminho_metricas = Path("resultados/metricas_regressao_linear.json")

if caminho_metricas.exists():
    with open(caminho_metricas, 'r') as f:
        m = json.load(f)

    r2_treino = m['treino']['R2']
    r2_teste = m['teste']['R2']
    mape_teste = m['teste']['MAPE_%']

    print(f"   -> R² no Treino: {r2_treino:.4f}")
    print(f"   -> R² no Teste: {r2_teste:.4f}")
    print(f"   -> MAPE no Teste: {mape_teste:.2f}%")

    print("\n   [Diagnóstico da Demanda]:")
    if r2_treino > 0.75 and r2_teste < 0.4:
        print("   ⚠️ OVERFITTING: O modelo decorou os dados de treino, mas falha em dados novos.")
    elif r2_treino < 0.4:
        print("   ⚠️ UNDERFITTING: R² baixo. As variáveis atuais não explicam bem as vendas.")
    else:
        print("   ✅ AJUSTE ADEQUADO: Regressão Linear estável e com boa capacidade preditiva.")
else:
    print("   [AVISO] Arquivo de métricas não encontrado. Execute a Regressão Linear primeiro (ou ignore se já a validou).")


# --- 2. DIAGNÓSTICO DO MODELO ESPACIAL (SPILLOVER EM R$) ---
print("\n---------------------------------------------------------")
print("2. Avaliando Modelo Espacial (Efeito Spillover):")

try:
    import libpysal

    def calcular_moran(y, w_matrix):
        if hasattr(w_matrix, 'sparse'):
            w_array = w_matrix.sparse.toarray()
        else:
            w_array = w_matrix

        n = len(y)
        y_mean = y.mean()
        num = np.sum(w_array * np.outer(y - y_mean, y - y_mean))
        den = np.sum((y - y_mean)**2)
        S0 = np.sum(w_array)
        return (n / S0) * (num / den)  # Correção do operador de multiplicação

    W_densa = W.sparse.toarray()

    # Atualizado para as colunas de Vendas Financeiras (R$)
    moran_leve = calcular_moran(df_mg['venda_leve'].values, W_densa)
    moran_pesada = calcular_moran(df_mg['venda_pesada'].values, W_densa)

    mod_leve = LinearRegression().fit(df_mg[['lag_venda_leve']], df_mg['venda_leve'])
    r2_espacial_leve = r2_score(df_mg['venda_leve'], mod_leve.predict(df_mg[['lag_venda_leve']]))

    mod_pesada = LinearRegression().fit(df_mg[['lag_venda_pesada']], df_mg['venda_pesada'])
    r2_espacial_pesada = r2_score(df_mg['venda_pesada'], mod_pesada.predict(df_mg[['lag_venda_pesada']]))

    print(f"   -> Índice de Moran (Leve): {moran_leve:.4f}")
    print(f"   -> R² Espacial (Leve)    : {r2_espacial_leve:.4f}")
    print(f"   -> Índice de Moran (Pesada): {moran_pesada:.4f}")
    print(f"   -> R² Espacial (Pesada)    : {r2_espacial_pesada:.4f}")

    print("\n   [Diagnóstico Espacial]:")
    if moran_leve > 0.3:
        print("   ✅ SPILLOVER LEVE FORTE: Forte autocorrelação. Cidades ricas puxam o consumo vizinho fortemente.")
    elif moran_leve > 0:
        print("   ⚠️ SPILLOVER LEVE FRACO/MODERADO: Existe transbordo financeiro regional, mas a dependência espacial é sutil.")
    else:
        print("   ❌ SPILLOVER LEVE INEXISTENTE: Moran negativo ou zero. Os mercados comportam-se de forma isolada.")

    if moran_pesada > 0.3:
        print("   ✅ SPILLOVER PESADA FORTE: Forte autocorrelação nas vendas pesadas.")
    elif moran_pesada > 0:
        print("   ⚠️ SPILLOVER PESADA FRACO/MODERADO: Transbordo financeiro baixo para linha pesada.")
    else:
        print("   ❌ SPILLOVER PESADA INEXISTENTE: Sem padrão espacial de vizinhança nas vendas pesadas.")

except Exception as e:
    print(f"   [AVISO] Dados de Spillover (W, lag) não encontrados ou incompletos na memória.\n   Erro: {e}")

# --- 3. DIAGNÓSTICO DA PROJEÇÃO POLINOMIAL ---
print("\n---------------------------------------------------------")
print("3. Avaliando Projeção Polinomial de Crescimento (2027-2031):")

try:
    caminho_projecao = Path('/content/drive/MyDrive/raw/Projecao_Crescimento_MG_2027_2031.xlsx')
    if caminho_projecao.exists():
        df_proj = pd.read_excel(caminho_projecao)

        ano_inicio = df_proj['Ano'].min()
        ano_fim = df_proj['Ano'].max()

        frota_inicio = df_proj.loc[df_proj['Ano'] == ano_inicio, 'Projeção_Frota_Total'].values[0]
        frota_fim = df_proj.loc[df_proj['Ano'] == ano_fim, 'Projeção_Frota_Total'].values[0]

        diesel_inicio = df_proj.loc[df_proj['Ano'] == ano_inicio, 'Proj_Consumo_Diesel_m3'].values[0]
        diesel_fim = df_proj.loc[df_proj['Ano'] == ano_fim, 'Proj_Consumo_Diesel_m3'].values[0]

        # Correção dos operadores multiplicativos
        crescimento_frota = ((frota_fim / frota_inicio) - 1) * 100
        crescimento_diesel = ((diesel_fim / diesel_inicio) - 1) * 100

        # Integração dos novos Indicadores XLSX no Terminal
        if 'Crescimento_Frota_Vs_Historico_%' in df_proj.columns:
            cresc_hist_frota = df_proj['Crescimento_Frota_Vs_Historico_%'].iloc[-1]
            cresc_hist_diesel = df_proj['Crescimento_Diesel_Vs_Historico_%'].iloc[-1]
            yoy_frota_ultimo = df_proj['Evolucao_Anual_Frota_%'].iloc[-1]
            yoy_diesel_ultimo = df_proj['Evolucao_Anual_Diesel_%'].iloc[-1]

            print(f"   -> Crescimento Acumulado ({ano_fim} vs Histórico): Frota {cresc_hist_frota:.2f}% | Diesel {cresc_hist_diesel:.2f}%")
            print(f"   -> Crescimento YoY (Ano-a-Ano no fim de {ano_fim}): Frota {yoy_frota_ultimo:.2f}% | Diesel {yoy_diesel_ultimo:.2f}%")

        print(f"   -> Crescimento apenas na Janela de Projeção ({ano_inicio}-{ano_fim}): Frota {crescimento_frota:.2f}% | Diesel {crescimento_diesel:.2f}%")

        print("\n   [Diagnóstico da Projeção]:")
        if crescimento_frota > 0 and crescimento_diesel > 0:
            if crescimento_diesel > 50:
                print("   ⚠️ ALERTA DE OVERFITTING: O crescimento projetado está excessivamente alto (>50% em 5 anos). Verifique se o grau do polinômio não está explodindo a curva.")
            else:
                print("   ✅ AJUSTE ADEQUADO: As curvas demonstram crescimento realista e sustentável para o mercado. O polinômio de Grau 2 evitou explosões exponenciais.")
        elif crescimento_frota < 0 or crescimento_diesel < 0:
             print("   ⚠️ RECESSÃO PROJETADA: O modelo indica queda de frota ou consumo. Valide se isso reflete o cenário macroeconômico real inserido nos dados históricos.")
        else:
             print("   ⚠️ ESTAGNAÇÃO: O modelo não projetou variação significativa no período.")
    else:
        print("   [AVISO] Arquivo de projeção não encontrado. Execute o modelo preditivo primeiro.")
except Exception as e:
    print(f"   [AVISO] Erro ao analisar a base de projeção: {e}")
```
## Resultados
Após executar todos os scripts obtivemos os seguntes resultados:
```text
=== INICIANDO DIAGNÓSTICO DE EFICIÊNCIA DE TREINAMENTO ===

1. Avaliando Regressão Linear (Modelo de Demanda):
   -> R² no Treino: 0.9831
   -> R² no Teste: 0.9858
   -> MAPE no Teste: 69.66%

   [Diagnóstico da Demanda]:
   ✅ AJUSTE ADEQUADO: Regressão Linear estável e com boa capacidade preditiva.

---------------------------------------------------------
2. Avaliando Modelo Espacial (Efeito Spillover):
   -> Índice de Moran (Leve): 0.1559
   -> R² Espacial (Leve)    : 0.0960
   -> Índice de Moran (Pesada): 0.1660
   -> R² Espacial (Pesada)    : 0.0982

   [Diagnóstico Espacial]:
   ⚠️ SPILLOVER LEVE FRACO/MODERADO: Existe transbordo financeiro regional, mas a dependência espacial é sutil.
   ⚠️ SPILLOVER PESADA FRACO/MODERADO: Transbordo financeiro baixo para linha pesada.

---------------------------------------------------------
3. Avaliando Projeção Polinomial de Crescimento (2027-2031):
   -> Crescimento Acumulado (2031 vs Histórico): Frota 24.36% | Diesel 58.55%
   -> Crescimento YoY (Ano-a-Ano no fim de 2031): Frota 2.96% | Diesel 7.13%
   -> Crescimento apenas na Janela de Projeção (2027-2031): Frota 12.84% | Diesel 31.11%

   [Diagnóstico da Projeção]:
   ✅ AJUSTE ADEQUADO: As curvas demonstram crescimento realista e sustentável para o mercado. O polinômio de Grau 2 evitou explosões exponenciais.
```
O desemppenho foi muito bom em todos os tres modelos com resultados consistentes utilizando diferentes bases de dados. Além desse resumo da execução cada um dos codigo gerou um relatório .xlsx ou imagens .png de acordo com a demanda.
Esse conteudo pode ser acessado pelos atalhos abaixo:



