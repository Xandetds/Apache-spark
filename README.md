# Arquitetura de Dados — Delta Lake & Apache Iceberg

**Grupo:** Alexandre Tibes · Roger Balcevicz · Murilo Salvan

Projeto acadêmico que demonstra, lado a lado, as capacidades de dois formatos modernos de tabela
para Data Lakehouse: **Delta Lake** e **Apache Iceberg**. Ambos operam sobre o mesmo dataset de
vagas de emprego em IA, provando suporte a operações ACID e Time Travel via Apache Spark.

---

## Arquitetura do Projeto

```
Apache-spark/
├── data/
│   └── ai_job_market_insights.csv   # Dataset: 500 vagas de emprego em IA
│
├── delta_lake.ipynb                 # Notebook 1 — Delta Lake
├── apache_iceberg.ipynb             # Notebook 2 — Apache Iceberg
│
├── docs/                            # Documentação MkDocs (publicada no GitHub Pages)
│   ├── index.md
│   ├── apache-spark.md
│   ├── delta-lake.md
│   └── apache-iceberg.md
├── mkdocs.yml                       # Configuração do MkDocs
│
├── spark-warehouse/                 # Gerado automaticamente — tabelas Delta
├── iceberg-warehouse/               # Gerado automaticamente — tabelas Iceberg
│
├── pyproject.toml                   # Dependências (uv)
└── uv.lock                          # Lockfile com versões exatas
```

---

## Pré-requisitos

| Ferramenta | Versão mínima | Instalação |
|---|---|---|
| Ubuntu (WSL 2) | 22.04+ | — |
| Java (OpenJDK) | **17** | `sudo apt install openjdk-17-jdk` |
| uv | latest | `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| Python | 3.11+ | gerenciado pelo uv |

---

## Parte 1 — Configuração do Ambiente

### Passo 1 — Java 17

Abra o terminal do **Ubuntu (WSL)** e execute:

```bash
sudo apt update && sudo apt install -y openjdk-17-jdk
```

Adicione ao `~/.bashrc` para persistir entre sessões:

```bash
echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64' >> ~/.bashrc
echo 'export PATH="$JAVA_HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Verifique:

```bash
java -version
# Esperado: openjdk version "17.x.x"
```

---

### Passo 2 — uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc
uv --version
```

---

### Passo 3 — Clone e instalação das dependências

```bash
git clone <URL-DO-REPOSITORIO>
cd ~/Apache-spark
uv venv
source .venv/bin/activate
uv sync
```

`uv sync` instala as versões exatas do `uv.lock`:
- `pyspark==3.5.3`
- `delta-spark==3.2.0`
- `jupyterlab`, `ipykernel`, `mkdocs`, `mkdocs-material`

---

## Parte 2 — Rodando os Notebooks

> **IMPORTANTE:** sempre use o terminal do Ubuntu/WSL para rodar o Jupyter.
> Abrir os notebooks pelo VS Code no Windows usa o Python do Windows (não o virtualenv WSL),
> causando `ModuleNotFoundError: No module named 'pyspark'`.

### Via terminal Ubuntu (recomendado)

```bash
cd ~/Apache-spark
source .venv/bin/activate
jupyter lab
```

Copie a URL com token que aparece no terminal (ex.: `http://localhost:8888/lab?token=abc123`)
e cole em qualquer browser no Windows. O JupyterLab abrirá com o ambiente correto.

### Via VS Code com WSL (alternativa)

1. No Windows, abra o VS Code.
2. Clique no botão verde **`><`** no canto inferior esquerdo → **"Connect to WSL"**.
3. Abra a pasta do projeto: `File > Open Folder > /home/alexandre/Apache-spark`.
4. Abra um terminal integrado no VS Code: **Terminal > New Terminal**.
5. No terminal (que agora é WSL), execute:

```bash
source .venv/bin/activate
jupyter lab
```

6. Copie a URL com token e cole no browser.

### Ordem de execução dos notebooks

> Execute as células **de cima para baixo** (`Shift + Enter` ou **Run All** `▶▶`).

**`delta_lake.ipynb`** — execute primeiro.

| Célula | O que faz |
|---|---|
| Setup | Cria SparkSession com Delta, limpa execuções anteriores |
| Carga Inicial | Lê o CSV e cria a tabela `ai_jobs_delta` |
| INSERT | Adiciona vaga de Data Engineer em Urussanga, SC |
| UPDATE | Reajuste salarial +10% para vagas de alto impacto |
| DELETE | Remove vagas com perspectiva `Decline` |
| DESCRIBE HISTORY | Exibe o log de versões da tabela |
| Time Travel | Consulta os dados na Versão 0 (estado original) |

**`apache_iceberg.ipynb`** — execute após encerrar o kernel do Delta.

> Encerre o kernel do Delta antes: **Kernel > Shut Down Kernel** no JupyterLab.

| Célula | O que faz |
|---|---|
| Setup | Cria SparkSession com Iceberg, limpa execuções anteriores |
| Carga Inicial | Lê o CSV e cria `local.db_jobs.ai_jobs_iceberg` |
| INSERT | Adiciona vaga de Data Engineer em Urussanga, SC |
| UPDATE | Reajuste salarial +10% para vagas de alto impacto |
| DELETE | Remove vagas com perspectiva `Decline` |
| Snapshots | Lista o histórico de snapshots |
| Time Travel | Consulta o snapshot inicial via `VERSION AS OF <id>` |

> **Na primeira execução de cada notebook**, o Spark baixa os JARs via Maven (~100 MB Delta, ~80 MB Iceberg). Aguarde.

---

## Parte 3 — Rodando o MkDocs

### Preview local

Com o ambiente virtual ativo, rode:

```bash
cd ~/Apache-spark
source .venv/bin/activate
mkdocs serve
```

Acesse `http://127.0.0.1:8000` no browser. O servidor recarrega automaticamente ao editar os arquivos em `docs/`.

Para parar: `Ctrl + C`.

---

### Deploy no GitHub Pages

O comando abaixo constrói o site e faz push para o branch `gh-pages` do repositório:

```bash
cd ~/Apache-spark
source .venv/bin/activate
mkdocs gh-deploy
```

Você verá algo como:

```
INFO - Building documentation...
INFO - Deploying to GitHub Pages
INFO - Your documentation should shortly be available at:
       https://<seu-usuario>.github.io/<nome-do-repo>/
```

---

### Ativar o GitHub Pages no repositório (apenas na primeira vez)

1. Acesse o repositório no GitHub.
2. Vá em **Settings > Pages**.
3. Em **Source**, selecione **"Deploy from a branch"**.
4. Em **Branch**, selecione **`gh-pages`** e pasta **`/ (root)`**.
5. Clique em **Save**.

Após alguns segundos, o site estará disponível em:

```
https://<seu-usuario>.github.io/<nome-do-repo>/
```

---

## Comparativo Técnico

| Característica | Delta Lake | Apache Iceberg |
|---|---|---|
| Armazenamento | Parquet + `_delta_log/` (JSON) | Parquet + `metadata/` (Avro/JSON) |
| Controle de versão | Versões numéricas (0, 1, 2...) | Snapshot IDs únicos (long) |
| Time Travel | `VERSION AS OF 0` | `VERSION AS OF <snapshot_id>` |
| Auditoria | `DESCRIBE HISTORY <table>` | `SELECT * FROM <table>.snapshots` |
| Engines suportadas | Spark, Databricks | Spark, Flink, Trino, Presto, Hive |
| Catálogo | `DeltaCatalog` (substitui default) | `SparkCatalog` (catálogo adicional) |

---

## Erros Comuns e Soluções

**`JAVA_HOME not set` ou `UnsupportedClassVersionError`**
→ Java 17 não está configurado. Reveja o Passo 1 acima.

---

**`ModuleNotFoundError: No module named 'pyspark'`**
→ O virtualenv não está ativo. Execute:

```bash
source .venv/bin/activate
jupyter lab
```

---

**`DELTA_CREATE_TABLE_WITH_NON_EMPTY_LOCATION`**
→ A célula de setup já limpa os arquivos anteriores com `shutil.rmtree`. Reexecute o notebook desde o início (Kernel > Restart & Run All).

---

**Iceberg — JAR não encontrado no Maven**
→ Limpe o cache do Ivy:

```bash
rm -rf ~/.ivy2/cache/org.apache.iceberg
```

---

**Dois notebooks abertos travam**
→ Encerre o kernel de um antes de abrir o outro: **Kernel > Shut Down Kernel**.
