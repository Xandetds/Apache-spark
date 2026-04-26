# Contextualização

Este projeto foi desenvolvido para demonstrar, na prática, o uso do **Apache Spark com PySpark** em conjunto com dois formatos modernos de tabela para data lake:

- **Delta Lake**
- **Apache Iceberg**

A ideia central do trabalho é mostrar como essas tecnologias funcionam sobre uma fonte de dados real, permitindo:

- leitura de dados com Spark;
- criação de tabelas analíticas;
- carga inicial;
- execução de operações ACID: `INSERT`, `UPDATE` e `DELETE`;
- consulta do estado e do histórico/snapshots das tabelas;
- Time Travel (consultar o estado dos dados em versões anteriores).

## Cenário adotado

Foi utilizado o dataset público **AI Job Market Insights**, com 500 registros de vagas de emprego
na área de Inteligência Artificial ao redor do mundo. O dataset permite explorar temas como
adoção de IA nas empresas, risco de automação de cargos, crescimento de vagas e distribuição salarial.

## Fonte de dados

O dataset está em um único arquivo CSV:

```
data/ai_job_market_insights.csv
```

## Modelo de Dados

| Coluna | Tipo | Descrição |
|---|---|---|
| Job_Title | STRING | Título da vaga (ex.: Data Engineer) |
| Industry | STRING | Setor da empresa (ex.: Technology) |
| Company_Size | STRING | Porte: Small / Medium / Large |
| Location | STRING | Cidade / País |
| AI_Adoption_Level | STRING | Nível de adoção de IA: Low / Medium / High |
| Automation_Risk | STRING | Risco de automação: Low / Medium / High |
| Required_Skills | STRING | Habilidades exigidas |
| Salary_USD | DOUBLE | Salário anual em dólares |
| Remote_Friendly | STRING | Aceita trabalho remoto: Yes / No |
| Job_Growth_Projection | STRING | Perspectiva de crescimento: Growth / Stable / Decline |

## Organização do repositório

```
Apache-spark/
├── data/ai_job_market_insights.csv   # Dataset
├── delta_lake.ipynb                  # Notebook Delta Lake
├── apache_iceberg.ipynb              # Notebook Apache Iceberg
├── docs/                             # Esta documentação (MkDocs)
├── pyproject.toml                    # Dependências (uv)
└── uv.lock
```

## Diferença entre README e MkDocs

**README.md** — guia técnico de configuração do ambiente e execução do projeto.

**MkDocs (este site)** — documentação acadêmica explicando os conceitos e as decisões de projeto,
publicada como site no GitHub Pages.
