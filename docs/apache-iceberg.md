# Apache Iceberg

## O que é o Apache Iceberg

O Apache Iceberg é um formato de tabela analítica open-source criado pela Netflix, projetado
para data lakes em escala. Diferentemente do Delta Lake, o Iceberg foi projetado desde o início
para ser **agnóstico de engine**, funcionando com Spark, Flink, Trino, Presto e Hive.

Principais características:

- transações ACID completas;
- versionamento por **snapshots** (IDs únicos do tipo `long`, gerados automaticamente);
- evolução de schema sem reescrita de dados;
- partition evolution (mudança de estratégia de particionamento sem reescrita);
- Time Travel com `VERSION AS OF <snapshot_id>`.

## Diferença fundamental em relação ao Delta Lake

| Aspecto | Delta Lake | Apache Iceberg |
|---|---|---|
| Versões | Números sequenciais (0, 1, 2...) | Snapshot IDs únicos (ex.: 8473920183746...) |
| Time Travel | `VERSION AS OF 0` | `VERSION AS OF <snapshot_id>` |
| Auditoria | `DESCRIBE HISTORY` | `SELECT * FROM <table>.snapshots` |
| Engines suportadas | Spark, Databricks | Spark, Flink, Trino, Presto, Hive |
| Catálogo padrão | Substitui o Spark default | Catálogo adicional (`local`, `glue`, etc.) |

## Como funciona internamente

Cada operação cria novos arquivos na pasta `metadata/`:

- **Snapshot**: ponteiro para o estado atual da tabela;
- **Manifest list**: lista todos os manifest files do snapshot;
- **Manifest file**: lista os arquivos Parquet que fazem parte daquele snapshot.

Os dados físicos (Parquet) em `data/` nunca são sobrescritos.

## Objetivo do notebook Iceberg

O notebook `apache_iceberg.ipynb` demonstra:

- criação da SparkSession com extensões Iceberg via `spark.jars.packages`;
- configuração de catálogo Hadoop local (`local`);
- leitura do CSV de vagas em IA (`ai_job_market_insights.csv`);
- criação do namespace `local.db_jobs` e da tabela `ai_jobs_iceberg`;
- carga inicial dos 500 registros;
- execução de `INSERT`, `UPDATE` e `DELETE`;
- consulta do histórico via `<table>.snapshots`;
- Time Travel para o snapshot inicial.

## Estrutura criada

**Catálogo:** `local` (HadoopCatalog)

**Namespace:** `local.db_jobs`

**Tabela:** `local.db_jobs.ai_jobs_iceberg`

**Armazenamento gerado automaticamente:**

```
iceberg-warehouse/
└── db_jobs/
    └── ai_jobs_iceberg/
        ├── data/
        │   └── *.parquet              # dados
        └── metadata/
            ├── v1.metadata.json       # schema + histórico de snapshots
            ├── snap-*.avro            # manifest lists
            └── *.avro                 # manifest files
```

## Operações demonstradas

### INSERT

Inserção de uma nova vaga de **Data Engineer** em Urussanga, SC:

```sql
INSERT INTO local.db_jobs.ai_jobs_iceberg VALUES
  ('Data Engineer', 'Technology', 'Medium', 'Urussanga',
   'High', 'Low', 'Python,Spark,SQL', 95000.00, 'Yes', 'Growth')
```

### UPDATE

Reajuste salarial de **+10%** para vagas de alto impacto:

```sql
UPDATE local.db_jobs.ai_jobs_iceberg
SET Salary_USD = ROUND(Salary_USD * 1.10, 2)
WHERE AI_Adoption_Level = 'High' AND Automation_Risk = 'Low'
```

### DELETE

Remoção de vagas com perspectiva de declínio:

```sql
DELETE FROM local.db_jobs.ai_jobs_iceberg
WHERE Job_Growth_Projection = 'Decline'
```

## Snapshots e Time Travel

```sql
-- Lista todos os snapshots com ID, timestamp e operação
SELECT * FROM local.db_jobs.ai_jobs_iceberg.snapshots

-- Volta ao estado do snapshot inicial (dados originais)
SELECT * FROM local.db_jobs.ai_jobs_iceberg VERSION AS OF <snapshot_id>
```

O `<snapshot_id>` é obtido da query de snapshots acima — o notebook captura isso automaticamente.

## Resultado esperado

O notebook mostra como o Iceberg organiza tabelas analíticas com versionamento por snapshots,
suporte completo a operações ACID e compatibilidade com múltiplos engines de processamento.
