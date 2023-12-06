# Generate Embeddings

With Lantern CLI, you can generate embeddings externally to Postgres, without affecting database performance, and import them. This is beneficial for large-scale embedding generation.

For one-off embedding generation, you can use [embedding generation inside Postgres](/docs/develop/generate) provided by Lantern Extras.

## Prerequisites

- [Lantern CLI](/docs/lantern-cli/install)
- [ONNX Runtime](/docs/lantern-cli/install)
- Running Postgres database

## Get Available Models

```bash
lantern-cli show-models
```

You will see an output like this

- `downloaded`: if the model onnx file and tokenizer are already downloaded or not (it will be automatically downloaded on the first run)
- `type`: if visual you should provide either image url or image path as input to generate embeddings for the image data

```bash
[*] [Lantern Embeddings] Available Models

BAAI/bge-small-en - type: textual, downloaded: true
transformers/multi-qa-mpnet-base-dot-v1 - type: textual, downloaded: false
microsoft/all-mpnet-base-v2 - type: textual, downloaded: false
thenlper/gte-base - type: textual, downloaded: true
clip/ViT-B-32-textual - type: textual, downloaded: true
llmrails/ember-v1 - type: textual, downloaded: true
microsoft/all-MiniLM-L12-v2 - type: textual, downloaded: true
BAAI/bge-base-en - type: textual, downloaded: true
intfloat/e5-large-v2 - type: textual, downloaded: false
intfloat/e5-base-v2 - type: textual, downloaded: true
thenlper/gte-large - type: textual, downloaded: true
BAAI/bge-large-en - type: textual, downloaded: true
clip/ViT-B-32-visual - type: visual, downloaded: true
```

Below is the table with specifications for each model

```table
| Model Name                              | Dimensions | Max Tokens | Avg Speed in Cloud |
| --------------------------------------- | ---------- | ---------- | ------------------ |
| microsoft/all-MiniLM-L12-v2             | 384        | 128        | 550 emb/s          |
| clip/ViT-B-32-textual                   | 512        | 77         | 500 emb/s          |
| BAAI/bge-small-en                       | 384        | 512        | 380 emb/s          |
| thenlper/gte-base                       | 768        | 128        | 250 emb/s          |
| intfloat/e5-base-v2                     | 768        | 512        | 230 emb/s          |
| microsoft/all-mpnet-base-v2             | 768        | 128        | 200 emb/s          |
| transformers/multi-qa-mpnet-base-dot-v1 | 768        | 250        | 120 emb/s          |
| BAAI/bge-base-en                        | 768        | 512        | 100 emb/s          |
| thenlper/gte-large                      | 1024       | 128        | 100 emb/s          |
| clip/ViT-B-32-visual                    | 512        | 224        | 50 emb/s           |
| llmrails/ember-v1                       | 1024       | 512        | 45 emb/s           |
| intfloat/e5-large-v2                    | 1024       | 512        | 40 emb/s           |
| BAAI/bge-large-en                       | 1024       | 512        | 25 emb/s           |
```

## Set Up Data

Note: You can skip this step if you already have data in your database

```sql
CREATE TABLE articles (id SERIAL PRIMARY KEY, title TEXT);
INSERT INTO articles (title) VALUES ('What is vector search'), ('Getting your AI application up and running in minutes'), ('HNSW vs IVFFLAT');
```

## Run Embedding Generation

```bash
lantern-cli create-embeddings  --model 'microsoft/all-MiniLM-L12-v2'  --uri 'postgresql://[username]:[password]@localhost:5432/[db]' --table "articles" --column "title" --out-column "title_embedding" --pk id --batch-size 100
```

## Verify Results

You can now query the database and see that embeddings have been generated for your data.

```sql
SELECT title_embedding FROM articles;
```

## CLI parameters

Run `bash lantern-cli create-embeddings --help` to get available CLI parameters

```bash
Usage: lantern-cli create-embeddings [OPTIONS] --model <MODEL> --uri <URI> --table <TABLE> --column <COLUMN> --out-column <OUT_COLUMN>

Options:
  -m, --model <MODEL>            Model name
  -u, --uri <URI>                Fully associated database connection string including db name
  -t, --table <TABLE>            Table name
  -s, --schema <SCHEMA>          Schema name [default: public]
  -p, --pk <PK>                  Table primary key column name [default: id]
  -c, --column <COLUMN>          Column name to generate embeddings for
      --out-uri <OUT_URI>        Output db uri, fully associated database connection string including db name. Defaults to
      --out-table <OUT_TABLE>    Output table name. Defaults to table
      --out-column <OUT_COLUMN>  Output column name
  -b, --batch-size <BATCH_SIZE>  Batch size
  -d, --data-path <DATA_PATH>    Data path
      --visual                   If model is visual
  -o, --out-csv <OUT_CSV>        Output csv path. If specified result will be written in csv instead of database
  -f, --filter <FILTER>          Filter which will be used when getting data from source table
  -l, --limit <LIMIT>            Limit will be applied to source table if specified
      --stream                   Stream data to output table while still generating
  -h, --help                     Print help
  -V, --version                  Print version
```
