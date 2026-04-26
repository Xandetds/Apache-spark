# Delta Lake

## O que é o Delta Lake

O Delta Lake é uma camada open-source que adiciona confiabilidade e governança sobre data lakes
baseados em Parquet. Desenvolvido pela Databricks e doado à Linux Foundation, oferece:

- transações ACID completas;
- versionamento sequencial (versão 0, 1, 2...);
- histórico auditável via `DESCRIBE HISTORY`;
- suporte a `UPDATE`, `DELETE` e `MERGE`;
- Time Travel com `VERSION AS OF <número>`.

## Como funciona internamente

Cada operação cria um novo arquivo JSON no `_delta_log/`. Esse log é a fonte da verdade:
controla quais arquivos Parquet estão ativos, quais foram deletados logicamente e qual
é a versão atual da tabela. Os dados físicos (Parquet) nunca são sobrescritos — apenas novos
arquivos são adicionados.

## Objetivo do notebook Delta

O notebook `delta_lake.ipynb` demonstra:

- criação da SparkSession com suporte a Delta via `spark.jars.packages`;
- leitura do CSV de vagas em IA (`ai_job_market_insights.csv`);
- criação da tabela `ai_jobs_delta` em formato Delta;
- carga inicial dos 500 registros;
- execução de `INSERT`, `UPDATE` e `DELETE`;
- inspeção do histórico com `DESCRIBE HISTORY`;
- Time Travel para a Versão 0 (estado original dos dados).

## Estrutura criada

**Tabela:**

```
ai_jobs_delta
```

**Armazenamento gerado automaticamente:**

```
spark-warehouse/
└── ai_jobs_delta/
    ├── *.parquet              # dados
    └── _delta_log/
        ├── 00000000000000000000.json   # versão 0 — carga inicial
        ├── 00000000000000000001.json   # versão 1 — INSERT
        ├── 00000000000000000002.json   # versão 2 — UPDATE
        └── 00000000000000000003.json   # versão 3 — DELETE
```

## Operações demonstradas

### INSERT

Inserção de uma nova vaga de **Data Engineer** em Urussanga, SC:

```sql
INSERT INTO ai_jobs_delta VALUES
  ('Data Engineer', 'Technology', 'Medium', 'Urussanga',
   'High', 'Low', 'Python,Spark,SQL', 95000.00, 'Yes', 'Growth')
```

### UPDATE

Reajuste salarial de **+10%** para vagas de alto impacto:

```sql
UPDATE ai_jobs_delta
SET Salary_USD = ROUND(Salary_USD * 1.10, 2)
WHERE AI_Adoption_Level = 'High' AND Automation_Risk = 'Low'
```

### DELETE

Remoção de vagas com perspectiva de declínio:

```sql
DELETE FROM ai_jobs_delta
WHERE Job_Growth_Projection = 'Decline'
```

## Histórico e Time Travel

```sql
-- Exibe todas as versões com operação e timestamp
DESCRIBE HISTORY ai_jobs_delta

-- Volta ao estado original (antes de qualquer modificação)
SELECT * FROM ai_jobs_delta VERSION AS OF 0
```

## Resultado esperado

O notebook comprova que o Delta Lake consegue atuar como camada transacional sobre arquivos
analíticos, oferecendo controle, rastreabilidade e capacidade de voltar no tempo para qualquer
versão anterior dos dados.
