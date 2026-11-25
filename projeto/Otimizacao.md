# Etapa 6 — Otimização e Governança de Dados

Esta etapa consolida a avaliação dos controles de governança e segurança aplicados ao pipeline, o histórico de otimizações técnicas implementadas nos algoritmos de Machine Learning e as recomendações estratégicas de mercado para o **Grupo Comolatti**.

---

## 1. Confronto de Governança e Segurança de Dados (Planejado vs. Realizado)

A arquitetura do projeto foi migrada de um ambiente legado (AWS) para o ecossistema **Microsoft Fabric**, unificando armazenamento, transformação e ciência de dados em um único ambiente governado. Abaixo está o confronto entre os princípios de governança planejados na Etapa 2 e as ações efetivamente executadas:

| Pilar de Governança | Planejamento (Etapa 2) | Execução Realizada e Ajustes (Etapa 6) |
| :--- | :--- | :--- |
| **Conformidade com a LGPD** | Garantir que o projeto não utilize dados pessoais sensíveis de clientes ou condutores. | **100% em conformidade:** O pipeline utiliza exclusivamente dados públicos agregados em nível municipal (IBGE, SENATRAN e ANP) e uma base de faturamento comercial simulada (`Simulacao_Vendas_Auto_MG_400_Reais.xlsx`), eliminando riscos de identificação de pessoas físicas (CPF, placas ou endereços individuais). |
| **Segurança e Operação de Banco de Dados** | Segregar ambientes de dados brutos e tratados, garantindo controle de acesso e integridade. | **Arquitetura em Camadas (Medallion/Fabric):** Separação física e lógica entre o **Data Lakehouse** (armazenamento imutável dos arquivos brutos originais) e o **Data Warehouse** (tabelas relacionais tipadas e prontas para consumo analítico via SQL e Python), impedindo sobrescrita acidental da fonte primária. |
| **Administração de Dados e Linhagem** | Manter rastreabilidade das transformações aplicadas sobre as bases públicas heterogêneas. | **Transformação Rastreável:** Uso de **Dataflows (Power Query)** no Fabric para registrar sequencialmente cada etapa de limpeza e tipagem, complementado por scripts versionados em Python para consolidação histórica (2016–2025) e chaves primárias padronizadas (`CÓDIGO IBGE` + `ANO`). |
| **Padrões de Qualidade de Dados** | Padronizar formatos numéricos, datas e chaves municipais entre fontes distintas. | **Saneamento Automatizado:** Implementação de normalização textual (`unicodedata`), dicionário de correção toponímica para os 853 municípios de Minas Gerais, remoção de registros nulos/inválidos (`MUNICIPIO NAO INFORMADO`), conversão de separadores decimais da ANP e validação de integridade referencial sem perda de municípios. |
| **Transparência e Prestação de Contas** | Documentar premissas, limitações estatísticas e códigos de forma auditável. | **Repositório Auditável:** Publicação integral dos notebooks, parâmetros de hiperparâmetros (`random_state=42`), métricas de validação ($R^2$, MAE, RMSE, MAPE e Índice de Moran) e explicitação clara sobre a natureza simulada da base de vendas utilizada para fins acadêmicos. |

---

## 2. Histórico de Otimizações Técnicas Implementadas

Ao longo do desenvolvimento do pipeline (`PUC - ML`), identificamos gargalos estatísticos e estruturais que foram corrigidos por meio das seguintes otimizações:

1. **Segmentação Prévia com K-Means ($K=6$) antes da Regressão Linear:**
   * *Problema:* Treinar um único modelo linear para todos os 853 municípios de Minas Gerais misturava metrópoles (como Belo Horizonte) com micromunicípios rurais, gerando alta variância percentual (heterocedasticidade).
   * *Otimização:* Aplicação de padronização (`StandardScaler`) e clusterização hierárquica em 6 perfis econômicos antes da modelagem de demanda. Isso estabilizou o poder preditivo ($R^2 = 0.9932$ no teste) e garantiu zero previsões negativas.
2. **Correção da Curva Demográfica via CAGR Censitário (2010 vs. 2022):**
   * *Problema:* As séries intercensitárias do IBGE mantinham a população "congelada" após o Censo de 2022, distorcendo a projeção de demanda dos anos seguintes.
   * *Otimização:* Integração direta com a API SIDRA do IBGE para extrair os Censos oficiais de 2010 (Tabela 202) e 2022 (Tabela 4714), calculando a **Taxa de Crescimento Anual Composta (CAGR)** específica de cada município para interpolar e projetar a evolução populacional real entre 2016 e 2025.
3. **Taxas Dinâmicas de Absorção no Modelo de *Spillover* Espacial:**
   * *Problema:* Coeficientes fixos de transbordo espacial superestimavam a atração de pequenos municípios isolados e subestimavam o efeito de vizinhança em torno de grandes polos.
   * *Otimização:* Criação de uma matriz de vizinhança (`KNN = 5` via `libpysal`) com cálculo dinâmico do coeficiente $\rho$ baseado na razão entre o *Spatial Lag* e a venda local, segmentado por ciclo de veículo (5% a 30% para Linha Leve; 8% a 40% para Linha Pesada) e validado pelo Índice de Moran ($0.1559$ leve e $0.1660$ pesada).
4. **Controle de *Overfitting* na Projeção Temporal (2027–2031):**
   * *Problema:* Polinômios de grau elevado ($\ge 3$) geravam explosões exponenciais irreais ao projetar o consumo pós-pandemia.
   * *Otimização:* Adoção da **Regressão Polinomial de Grau 2**, suavizando anomalias históricas (como a retração de 2018–2020) e projetando um crescimento sustentável para o quinquênio 2027–2031 (+12,84% na frota total e +31,11% no consumo de diesel na janela projetada).

---

## 3. Oportunidades de Mercado (Grupo Comolatti)

Com base nos indicadores gerados pelo **Atlas de Potencial Comercial**, destacam-se quatro frentes imediatas de atuação comercial e logística para o Grupo Comolatti em Minas Gerais:

* **Priorização da Linha Pesada (Ciclo Diesel):** A projeção temporal aponta um crescimento acumulado de **58,55% no consumo de diesel até 2031** frente ao histórico (ritmo de +7,13% ao ano ao final do período), superando o ritmo de expansão da frota leve. Recomenda-se reforçar o mix de peças de reposição pesada e componentes de motor/transmissão nos centros de distribuição que atendem corredores rodoviários.
* **Estratégia *Hub-and-Spoke* nos Polos Logísticos:** Os clusters **Polo Logístico e Metropolitano** e **Centro Regional Principal** (como Belo Horizonte, Contagem, Betim e Uberlândia) concentram a maior parte do volume absoluto e apresentam forte aderência preditiva. O estoque de giro rápido e peças de maior valor agregado deve ser centralizado nesses *hubs*.
* **Captura de Demanda por *Spillover* em Cidades-Satélite:** A análise espacial comprovou que cerca de **10% da variação comercial** ($R^2$ espacial de ~0,10) decorre do transbordo entre municípios vizinhos. Cidades classificadas nos clusters intermediários próximas a grandes polos absorvem demanda de manutenção de frotas que circulam nas regiões metropolitanas — representando oportunidades para equipes de vendas externas e televendas ativo sem necessidade de abertura imediata de novas filiais físicas.
* **Atendimento Digital nos Micromunicípios:** Como a maioria dos 853 municípios mineiros pertence aos clusters de **Pequenos Municípios de Apoio** e **Micromunicípios / Rurais** (onde a demanda individual é baixa e pulverizada), o atendimento deve ser priorizado via canais digitais (B2B e-commerce) e rotas logísticas fracionadas, preservando margem operacional.

---

## 4. Propostas Futuras de Evolução do Projeto

Para evoluir o projeto de um modelo analítico estático para uma solução corporativa em produção:

1. **Orquestração Automatizada em Nuvem:** Ativar pipelines agendados no **Azure Data Factory / Microsoft Fabric** para ingestão automática das atualizações anuais da ANP e mensais da SENATRAN, eliminando cargas manuais.
2. **Substituição da Base Simulada por Dados Reais de ERP:** Conectar o Data Warehouse diretamente ao histórico real de faturamento por SKU e CNPJ do Grupo Comolatti, mantendo a anonimização/agregação por município para preservar a conformidade com a LGPD.
3. **Transformação Logarítmica nos Modelos de Demanda:** Aplicar transformação logarítmica (`np.log1p`) na variável alvo da Regressão Linear ou treinar regressores específicos por cluster para reduzir ainda mais o erro percentual (MAPE) nos micromunicípios.
4. **Expansão Nacional e Mapas Interativos:** Replicar a arquitetura para as demais Unidades da Federação onde o grupo possui cerca de 120 filiais e disponibilizar mapas interativos (Folium / Power BI conectado ao Fabric) para consulta direta dos gerentes regionais.
