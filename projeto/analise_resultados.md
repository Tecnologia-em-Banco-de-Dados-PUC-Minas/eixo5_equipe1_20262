## Análise de Resultados

Após a execução dos scripts apresentados anteriormente, obtivemos 4 arquivos distintos como entregáveis. Abaixo está a análise detalhada de cada etapa do pipeline:

### 1. Clusterização de Municípios (K-Means)

A primeira etapa consiste no agrupamento (*clustering*) dos **853** municípios do estado de Minas Gerais. Com essa técnica, segmentamos as cidades em 6 grupos estruturais, tornando a modelagem subsequente mais precisa e adaptada à realidade de cada porte econômico.

Como resultado, é gerado um arquivo no formato `.xlsx` (`Base_Consolidada_Clusters_MG.xlsx`) contendo as seguintes colunas: `Código **IBGE**`, `Município`, `População`, `Consumo total etanol + gasolina`, `Consumo diesel` e `cluster_nome`.

Os 6 clusters formados — classificados hierarquicamente com base nos principais indicadores do estudo (população, frota total e consumo de combustíveis leves e diesel) — são:
1. **Polo Logístico e Metropolitano**
2. **Centro Regional Principal**
3. **Polo Econômico Intermediário**
4. **Município em Desenvolvimento**
5. **Pequenos Municípios de Apoio**
6. **Micromunicípios / Rurais**

**Exemplo dos dados produzidos:**

<img width="1207" height="375" alt="Amostra da tabela de clusters gerada" src="https://github.com/user-attachments/assets/2fbfabb4-3ca5-4a9d-a646-f59de32333e4" />

---

### 2. Regressão Linear (Modelo de Demanda)

Na etapa de regressão linear, são gerados dois artefatos complementares: um arquivo `.json` contendo as métricas estatísticas de treino e teste, e uma visualização gráfica comparando os valores reais, os valores previstos e a distribuição dos resíduos.

```json
{
    *teste*: {
    *MAE*: **22931**.78,
    *RMSE*: **59094**.30,
    *R2*: 0.**9932**,
    *MAPE_%*: 48.50
    },
    *treino*: {
    *MAE*: **26272**.97,
    *RMSE*: **126577**.90,
    *R2*: 0.**9875**,
    *MAPE_%*: 46.16
    },
    *previsoes_negativas*: 0
}
```
<img width="1421" height="586" alt="image" src="https://github.com/user-attachments/assets/f06c0bcf-8f93-4b16-bf38-a004c4b6c6d2" />

## Interpretação dos Resultados:
### Capacidade Preditiva ($R^2 = 0.**9932**$ no teste): 
O modelo explicou mais de 99% da variância dos dados de teste, sem gerar previsões negativas e mantendo estabilidade entre treino e teste (ausência de overfitting).
### Comportamento por Porte Municipal: 
No gráfico à esquerda (Real x Previsto), a grande concentração de pontos próximos à origem (0, 0) representa os pequenos municípios e micromunicípios, que compõem a maioria do estado. Já os pontos isolados na parte superior direita correspondem aos grandes polos urbanos, onde o modelo acompanhou fielmente a tendência linear, apresentando apenas uma leve subestimação nos extremos.
### Análise de Resíduos e **MAPE** ($48.50\%$): 
O gráfico à direita evidencia a heterocedasticidade natural da base (efeito *funil*). Nos municípios menores, pequenas variações absolutas na previsão geram erros percentuais elevados (puxando o **MAPE** para cima), embora tenham impacto financeiro irrelevante no volume total do estado.

## 3. Spillover Espacial
Como resultado da análise de spillover, é gerado um arquivo .xlsx contendo os dados base de venda por município, seguidos do percentual de venda complementar calculada, o aumento em reais (R$) dessa venda e os valores percentuais e monetários do impacto total no faturamento.Com essa análise, compreendemos o impacto que os municípios adjacentes têm sobre a venda de cada região. Esses valores são mais perceptíveis em grandes cidades como Belo Horizonte, Contagem e Uberlândia, devido ao alto faturamento e à concentração populacional e de consumo de combustíveis.Um detalhe fundamental para o bom desempenho dessa modelagem é a separação das vendas entre linha leve e linha pesada, já que representam mercados distintos e com sazonalidades variadas.

<img width="1487" height="428" alt="image" src="https://github.com/user-attachments/assets/fe466146-1b9f-4425-8745-9d0d492ad9e7" />

## 4. Projeção de Crescimento dos KPIs

Como quarta e última análise, realizamos a projeção para os próximos 5 anos dos principais indicadores utilizados ao longo de todo o trabalho (consumo de combustível, população e frota de veículos). Isso permite validar qual deverá ser o foco para estudos futuros e qual desses indicadores exerce maior impacto sobre as vendas.Como resultado da execução, são gerados um gráfico contendo o histórico e a curva de crescimento dos indicadores, além de um arquivo .xlsx com os mesmos dados detalhados ano a ano.

<img width="1296" height="932" alt="image" src="https://github.com/user-attachments/assets/6d1b8215-2664-4d9e-bfc4-eb3caa35910c" />


<img width="1836" height="181" alt="image" src="https://github.com/user-attachments/assets/a184666d-ee63-4490-9019-5895261dedf5" />
