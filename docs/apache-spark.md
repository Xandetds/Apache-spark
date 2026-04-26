# Apache Spark (PySpark)

## O que é o Apache Spark

O Apache Spark é um motor de processamento distribuído voltado para análise de dados em larga escala,
ETL, consultas SQL e machine learning. Suporta execução em cluster (YARN, Kubernetes) ou em modo
local para desenvolvimento e testes.

Neste trabalho foi utilizado o **PySpark** (API Python do Spark) em modo `local[*]`,
aproveitando todos os núcleos da máquina localmente.

## Por que usar PySpark neste trabalho

- Configuração simples em ambiente acadêmico (sem cluster real);
- integração nativa com notebooks Jupyter;
- API unificada de DataFrames e SQL;
- suporte nativo a Delta Lake e Apache Iceberg como extensões;
- permite demonstração prática de ACID e Time Travel sem infraestrutura complexa.

## Papel do Spark no projeto

O Spark foi usado como engine central para:

- ler o arquivo CSV (`ai_job_market_insights.csv`) com inferência de schema;
- criar tabelas nos formatos Delta e Iceberg;
- executar comandos SQL (`INSERT`, `UPDATE`, `DELETE`);
- consultar histórico de versões e snapshots;
- realizar Time Travel sobre os dados.

## Estrutura geral do fluxo

1. Criar a `SparkSession` com as extensões do formato de tabela (Delta ou Iceberg);
2. ler o CSV com `inferSchema`;
3. criar a tabela no formato correspondente via SQL (`CREATE TABLE ... USING DELTA/iceberg`);
4. inserir os dados via `INSERT INTO ... SELECT`;
5. executar operações ACID (`INSERT`, `UPDATE`, `DELETE`);
6. inspecionar o histórico (`DESCRIBE HISTORY` ou `.snapshots`);
7. demonstrar Time Travel para o estado original dos dados.

## Fonte de dados usada

```
data/ai_job_market_insights.csv  —  500 registros de vagas em Inteligência Artificial
```

Um único CSV com 10 colunas, incluindo título da vaga, setor, localização, salário,
nível de adoção de IA e perspectiva de crescimento.

## Benefícios do Spark nesse contexto

| Característica | Benefício |
|---|---|
| Modo `local[*]` | Roda sem cluster, usa todos os núcleos da máquina |
| SQL unificado | Mesma sintaxe SQL para Delta e Iceberg |
| `inferSchema` | Detecta tipos automaticamente no CSV |
| Extensões plugáveis | Delta e Iceberg são adicionados via `spark.jars.packages` |
| Time Travel nativo | `VERSION AS OF` funciona nos dois formatos |
