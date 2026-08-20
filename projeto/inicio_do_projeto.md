1. Introdução 

O Grupo Comolatti atua em diversos segmentos econômicos, como os setores imobiliário e alimentício, com destaque para a distribuição de peças automotivas, área na qual se consolidou como uma das principais referências do mercado brasileiro. A empresa conta com aproximadamente 120 filiais distribuídas em todos os estados do Brasil e no Distrito Federal, além de mais de 5 mil colaboradores, o que garante ampla capilaridade e forte presença nacional. 

Apesar dessa estrutura robusta e da atuação em um país de dimensões continentais, a empresa ainda enfrenta limitações na identificação precisa do potencial de mercado por região. Entende-se como necessário os indicadores: 

- motorização (veículos por habitante), 

- intensidade de consumo de combustível por município, 

- quantidade de clientes por munícipio, 

- índice combinado de potencial de mercado. Este indicador será analisado pontuado no próximo semestres. 

 Esses indicadores serão calculados a partir de dados públicos de população, como também da base de vendas de veículos e comercialização de combustíveis por município. O objetivo é definir prioridades comerciais e ter um direcionamento estratégico dos esforços de vendas. Nesse contexto, torna-se fundamental adotar uma abordagem analítica, que permita compreender melhor as particularidades econômicas, o perfil de consumo, a demanda local e o desempenho comercial por município, possibilitando decisões mais assertivas. 

A regionalização das estratégias comerciais, apoiada em dados estruturados, permite otimizar estoques, ajustar políticas comerciais, aprimorar a logística e aumentar a competitividade, promovendo maior alinhamento entre oferta e demanda em cada localidade. 

Para sustentar sua operação nacional, o Grupo Comolatti possui uma estrutura robusta de Tecnologia da Informação. Seu principal banco de dados é o Progress 4GL, hospedado na infraestrutura da Amazon Web Services (AWS), responsável por suportar processos críticos como logística, controle de estoque, cadastros e sistemas corporativos. 

No âmbito comercial, a empresa utiliza o WebSales como CRM, centralizando atividades como gestão de clientes, aplicação de descontos, políticas comerciais e acompanhamento de pedidos. Para prospecção e expansão da base de clientes, adota o Salesforce, voltado à gestão de leads e funil de vendas. Na camada analítica, utiliza o Qlik Sense como ferramenta de Business Intelligence, possibilitando a criação de dashboards gerenciais, monitoramento de indicadores de desempenho e análises estratégicas por região, produto e filial. 

2. Descrição das Bases de Dados Utilizadas 

Foram utilizadas quatro bases de dados principais para a análise, todas em formato CSV, facilitando a integração e análise dos dados. As bases utilizadas foram: 

Vendas (janeiro de 2025 a fevereiro de 2026): Dados internos do Grupo Comolatti, obtidos por meio de uma reunião realizada em parceria com a empresa, na qual foi concedido acesso, via Google Drive, aos arquivos de vendas necessários para a análise. Esses dados, disponibilizados em formato CSV, contêm informações sobre o desempenho comercial das peças automotivas. 

Dados demográficos e regionais (2025): Dados obtidos junto ao Instituto Brasileiro de Geografia e Estatística, contendo informações sobre municípios, estados e estimativas populacionais. Os arquivos disponibilizados no link de referência encontram-se originalmente em formato XLS; no entanto, para viabilizar a ingestão na plataforma AWS, foi realizada a conversão para o formato CSV. O documento utilizado está identificado com o título “Estimativas de população enviadas ao TCU”. 

<https://www.ibge.gov.br/estatisticas/sociais/populacao/9103-estimativas-de-populacao.html> 

 Dados de combustíveis (2024): Provenientes da Agência Nacional do Petróleo e Gás Natural e Biocombustíveis (ANP) - ao acessar o link será direcionado para a página da agência, a informação das bases analisadas são: Vendas anuais de etanol hidratado e derivados de petróleo por município e os links Etanol hidratado (Vendas de etanol hidratado período 1990-2024), Gasolina C (Vendas de gasolina C período 1990-2024) e óleo diesel (Vendas de óleo diesel período 1990-2024) 

<https://www.gov.br/anp/pt-br/centrais-de-conteudo/dados-abertos/vendas-de-derivados-de-petroleo-e-b…> 

 Dados de Frotas (2025): Provenientes do Ministério dos Transportes, por meio do Sistema Nacional de Trânsito (SENATRAN) — ao acessar o link será direcionado para a página do órgão. A informação da base analisada é: Frota Nacional de Veículos por Município, disponibilizada no arquivo Frota por Município e Tipo — dezembro/2025 

<https://www.gov.br/transportes/pt-br/assuntos/transito/conteudo-Senatran/frota-de-veiculos-2025> 
