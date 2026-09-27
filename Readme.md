MVP - Pipeline de Engenharia de Dados: Análise de Viagens a Serviço (Gov)

Este repositório contém a solução completa de Engenharia de Dados desenvolvida para o MVP (Minimum Viable Product). O objetivo do projeto é ingerir, tratar, modelar e analisar os dados públicos de viagens a serviço do Governo Federal (disponibilizados no Portal da Transparência), utilizando o ecossistema **Databricks**, **PySpark**, **Spark SQL** e **Delta Lake**.

Arquitetura da Solução

O pipeline foi construído seguindo a **Arquitetura Medalhão (Lakehouse Architecture)** no **Databricks**, garantindo qualidade, governança e rastreabilidade dos dados do início ao fim:

[ Portal da Transparência (Arquivos CSV) ]
                   │
                   ▼
 [ Unity Catalog / Volumes (Landing Zone) ]
                   │
                   ▼
 [ Camada BRONZE ] (Delta Tables - Ingestão Bruta e Schema Enforcement)
                   │
                   ▼
 [ Camada SILVER ] (Delta Tables - Limpeza, Deduplicação, Casts e LGPD)
                   │
                   ▼
 [ Camada GOLD ]   (Delta Tables - Modelagem Dimensional Star Schema)
                   │
                   ▼
 [ Databricks SQL Queries & Gráficos ]

 Componentes da Camada Gold (Star Schema)

 * fato_viagens: Tabela Fato contendo as métricas de valores de diárias, passagens, devoluções, custos totais e duração em dias.

* dim_orgao: Dimensão com a hierarquia dos órgãos governamentais solicitantes.

* dim_tempo: Dimensão temporal estruturada (Ano, Mês, Trimestre, Semestre, Dia da Semana).

* dim_meio_transporte: Dimensão com os modais de transporte utilizados.

* dim_viajante: Dimensão de viajantes com mascaramento de dados sensíveis (CPF) para conformidade com a LGPD.

Tecnologias Itilizadas

* Linguagens: Python (PySpark) e SQL (Spark SQL).

* Plataforma: Databricks (Databricks Runtime 13.3+ LTS).

* Armazenamento: Delta Lake.

* Governança: Unity Catalog (Volumes e Catálogo Centralizado).

* Visualização: Databricks Visualizations / Editor SQL Nativo.

Estrutura do Repositório

├── README.md                              <- Guia de execução e visão geral do projeto
├── docs/
│   ├── Relatorio_MVP_Engenharia_Dados.pdf <- Relatório acadêmico completo em PDF
│   ├── data_lineage.png                   <- Diagrama de Linhagem de Dados
│   └── dashboard_graficos.png             <- Evidências das visualizações geradas
└── notebooks/
    ├── 01_Bronze_Ingestao.py              <- Carga dos dados brutos para formato Delta
    ├── 02_Silver_Limpeza_LGPD.py          <- Tratamento, sanitização e mascaramento
    ├── 03_Gold_Modelagem_StarSchema.py    <- Construção das Dimensões e Tabela Fato
    └── 04_Gold_Queries_Analiticas.sql     <- Resolução das perguntas de negócio e gráficos

Guia de Reprodução no Databricks

1. Pré-requisitos:

* Um workspace Databricks ativo;
* Um cluster configurado com Spark 3.4+ e Databricks Runtime 13.3 LTS ou superior;
* Acesso ao Unity Catalog para criação de schema e volumes.

2. Configuração do Ambiente e Ingestão:

* Crie o schema/banco de dados executando:
CREATE DATABASE IF NOT EXISTS workspace.db_viagens_gov; (SQL)

* Faça o upload dos arquivos CSV extraídos do Portal da Transparência no Volume do Unity Catalog no caminho configurado (ex: /Volumes/workspace/db_viagens_gov/landing_zone/).

* Execute os notebooks armazenados na pasta notebooks/ na seguinte sequência:

01_Bronze_Ingestao: Lê os arquivos brutos da landing zone e grava as tabelas bronze_viagens e bronze_passagens no formato Delta Lake.

02_Silver_Limpeza_LGPD: Aplica casting de tipos (Datas, Reais), realiza a remoção de duplicidades e aplica mascaramento de PII (CPF) criando as tabelas silver_viagens e silver_passagens.

03_Gold_Modelagem_StarSchema: Processa as chaves surrogadas (sk_*), constrói as dimensões (dim_orgao, dim_tempo, dim_meio_transporte, dim_viajante) e popula a fato_viagens.

04_Gold_Queries_Analiticas: Roda as consultas SQL de respostas de negócio (Top órgãos em gastos, análise de sazonalidade, etc.) e gera os gráficos de suporte.

Principais Insights de Negócio Encontrados

* Órgãos com Maiores Gastos: Identificação clara da concentração de despesas de viagem em Ministérios de grande porte operacional.

* Sazonalidade nos Custos: Mapeamento de picos de despesas ao longo do ano, evidenciando o mês de Novembro como o período de maior volume orçamentário executado em passagens e diárias.

* Acurácia do Pipeline: Validação e integridade dos totais consolidados entre a origem pública e a camada final de consumo.