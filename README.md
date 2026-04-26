# Engenharia de Dados — Delta Lake & Apache Iceberg

**Grupo:** Alexandre Tibes · Roger Balcevicz · Murilo Salvan  
**Documentação Completa:** [https://Xandetds.github.io/Apache-spark/](https://Xandetds.github.io/Apache-spark/)

## Sobre o Projeto

Este projeto acadêmico demonstra as capacidades de dois dos principais formatos modernos de tabelas para a arquitetura Data Lakehouse: **Delta Lake** e **Apache Iceberg**. 

Utilizando o motor do **Apache Spark**, o projeto ingere um dataset local com 500 vagas de emprego na área de Inteligência Artificial e executa operações transacionais (ACID: INSERT, UPDATE, DELETE), além de demonstrar o recurso de Time Travel e controle de Snapshots.

---

## Pré-requisitos

Para garantir a execução correta sem conflitos de ambiente, o projeto foi estruturado para rodar no **Ubuntu (WSL 2)**. 

| Ferramenta | Versão | Finalidade |
|---|---|---|
| Ubuntu (WSL 2) | 22.04+ | Ambiente isolado de execução |
| Java (OpenJDK) | 17 | Runtime obrigatório do Apache Spark |
| uv | 0.4+ | Gerenciamento de dependências e ambiente virtual |
| Python | 3.11+ | Linguagem base (gerenciada pelo uv) |

---

## Configurando o Ambiente

### 1. Instalação do Java 17

O Spark exige o Java para rodar. No terminal do seu Ubuntu (WSL), execute:

```bash
sudo apt update && sudo apt install -y openjdk-17-jdk
```

Para garantir que o Java fique salvo nas variáveis de ambiente:

```bash
echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64' >> ~/.bashrc
echo 'export PATH="$JAVA_HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### 2. Instalação do gerenciador uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc
```

### 3. Clone e Sincronização

Clone este repositório e baixe todas as bibliotecas necessárias de forma automática:

```bash
git clone https://github.com/Xandetds/Apache-spark.git
cd Apache-spark
uv sync
```

> O comando `uv sync` cria o ambiente virtual e instala PySpark, Delta, JupyterLab e MkDocs automaticamente.

---

## Executando os Notebooks

> **IMPORTANTE:** Para evitar o erro `ModuleNotFoundError`, não abra os notebooks clicando diretamente neles pelo VS Code no Windows. Inicie o servidor pelo terminal do WSL usando o `uv`.

No terminal do Ubuntu, dentro da pasta do projeto, inicie o JupyterLab:

```bash
uv run jupyter lab
```

O terminal exibirá uma URL com um token (ex: `http://localhost:8888/lab?token=...`). Copie essa URL e cole no seu navegador.

### Ordem de Execução

1. `delta_lake.ipynb` — Execute todas as células em ordem (`Shift + Enter`). Este notebook criará a tabela Delta e demonstrará as operações.

2. **Desligue o Kernel do Delta** — No menu superior do JupyterLab, vá em `Kernel > Shut Down Kernel`. Isso evita que duas instâncias do Spark briguem pela mesma porta local.

3. `apache_iceberg.ipynb` — Abra o arquivo e execute as células. Na primeira execução, o Spark baixará o conector do Iceberg via Maven Central (~80 MB).

---

## Documentação (MkDocs)

A documentação detalhada foi gerada usando o MkDocs Material e está publicada no GitHub Pages.

### Visualizando localmente

```bash
uv run mkdocs serve
```

Acesse `http://127.0.0.1:8000` no seu navegador.

### Publicando atualizações

```bash
uv run mkdocs gh-deploy
```

---

## Comparativo Técnico do Projeto

| Característica | Delta Lake | Apache Iceberg |
|---|---|---|
| Armazenamento | Parquet + `_delta_log/` (JSON) | Parquet + `metadata/` (Avro/JSON) |
| Controle de versão | Versões numéricas (0, 1, 2...) | Snapshot IDs únicos (long integer) |
| Time Travel | `VERSION AS OF 0` | `VERSION AS OF <snapshot_id>` |
| Auditoria | `DESCRIBE HISTORY <table>` | `SELECT * FROM <table>.snapshots` |
| Engines suportadas | Foco em Spark e Databricks | Spark, Flink, Trino, Presto, Snowflake |
| Setup do Catálogo | `DeltaCatalog` (substitui o padrão) | `SparkCatalog` (adicionado via Hadoop) |

---

