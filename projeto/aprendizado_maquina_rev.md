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
   
<img width="396" height="115" alt="image" src="https://github.com/user-attachments/assets/e02fa63a-4053-4e42-beed-ea099bb7243d" />

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

```text
[Base Master Analítica (2016-2025)]
               │
               ▼
   [1. K-Means (K=6 Clusters)]
               │
               ├───► [2. Regressão Linear de Demanda (Treino/Teste)]
               │
               ├───► [3. Econometria Espacial: Spillover Dinâmico (KNN=5)]
               │
               └───► [4. Projeção Temporal Macro: Grau 2 (2027-2031)]
