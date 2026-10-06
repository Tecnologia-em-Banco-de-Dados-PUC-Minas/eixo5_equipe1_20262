# Descrição das Bases de Dados Utilizadas

Foram utilizadas quatro bases de dados principais para a análise, todas em formato CSV, o que facilitou a integração e o tratamento dos dados. São elas:

Vendas (janeiro de 2025 a fevereiro de 2026): dados internos do Grupo Comolatti, obtidos por meio de reunião realizada em parceria com a empresa, na qual foi concedido acesso, via Google Drive, aos arquivos de vendas necessários para a análise. Os dados, disponibilizados em formato CSV, contêm informações sobre o desempenho comercial de peças automotivas.

Dados demográficos e regionais (2025): dados obtidos junto ao Instituto Brasileiro de Geografia e Estatística (IBGE), com informações sobre municípios, estados e estimativas populacionais. O documento utilizado é intitulado "Estimativas de população enviadas ao TCU". Os arquivos estão originalmente em formato XLS e, para viabilizar a ingestão na plataforma AWS, foram convertidos para CSV.
Fonte: https://www.ibge.gov.br/estatisticas/sociais/populacao/9103-estimativas-de-populacao.html

Dados de combustíveis (2024): provenientes da Agência Nacional do Petróleo, Gás Natural e Biocombustíveis (ANP). Foram utilizadas as bases de vendas anuais, por município, de etanol hidratado, gasolina C e óleo diesel, todas referentes ao período de 1990 a 2024.
Fonte: https://www.gov.br/anp/pt-br/centrais-de-conteudo/dados-abertos/vendas-de-derivados-de-petroleo-e-b…

Dados de frota de veículos por município (2025): provenientes do Ministério dos Transportes, por meio do Sistema Nacional de Trânsito (SENATRAN). A base analisada é a Frota Nacional de Veículos por Município, disponibilizada no arquivo "Frota por Município e Tipo – dezembro/2025".
Fonte: https://www.gov.br/transportes/pt-br/assuntos/transito/conteudo-Senatran/frota-de-veiculos-2025

# Governança de Dados

Os dados de vendas do Grupo Comolatti são protegidos por um robusto esquema de governança, principalmente em razão do alto volume de informações sensíveis (CNPJ, valores de venda, telefone e e-mail) armazenadas nos bancos de dados.

Somente o time de dados tem acesso ao banco de dados e às suas fontes, e o armazenamento, o tratamento e a gestão das informações são realizados na AWS (S3, Glue, Athena, entre outros serviços). A disponibilização de dashboards e análises é feita por meio do Qlik Sense, e cada filial possui um acesso exclusivo, que exibe apenas os dados da própria unidade. Não há permissão para exportação ou manipulação direta dos dados disponibilizados.

Algumas áreas específicas da Matriz têm acesso a uma versão mais completa, que permite criar relatórios personalizados e extrair dados.

As demais bases são públicas e estão disponíveis nos sites oficiais do governo. Sua governança e gestão são de responsabilidade do Estado e/ou do respectivo ministério.
