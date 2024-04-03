# Generate Embeddings

With the Lantern CLI `create-embeddings` routine, you can populate an entire column with embeddings without affecting database performance. This is achieved by generating the embeddings outside of Postgres and importing them. This is useful for large-scale embedding population.

The Lantern CLI also supports continuously populating an embedding column as new rows are inserted.

For one-off embedding generation, you can use [embedding generation inside Postgres](/docs/lantern-extras/embeddings), a feature provided by Lantern Extras.

## Prerequisites

- [Lantern CLI](/docs/lantern-cli/install)
- [ONNX Runtime](/docs/lantern-cli/install)
- Running Postgres database

## Available Models

The following models are available in the latest version of the CLI

CONTENT_VAR_MODELS

To see available runtimes run

```bash
lantern-cli show-runtimes


[*] [Lantern Embeddings] Available Runtimes

ort
openai
cohere
```

To see the available models for specific runtime in the Lantern CLI, run

```bash
lantern-cli show-models --runtime ort
```

You will see an output like this

```bash
[*] [Lantern Embeddings] Available Models

intfloat/e5-base-v2 - type: textual, downloaded: true
BAAI/bge-large-en - type: textual, downloaded: true
BAAI/bge-small-en - type: textual, downloaded: true
microsoft/all-MiniLM-L12-v2 - type: textual, downloaded: true
clip/ViT-B-32-textual - type: textual, downloaded: true
clip/ViT-B-32-visual - type: visual, downloaded: true
jinaai/jina-embeddings-v2-small-en - type: textual, downloaded: true
thenlper/gte-base - type: textual, downloaded: true
BAAI/bge-base-en - type: textual, downloaded: true
microsoft/all-mpnet-base-v2 - type: textual, downloaded: true
llmrails/ember-v1 - type: textual, downloaded: true
intfloat/e5-large-v2 - type: textual, downloaded: true
thenlper/gte-large - type: textual, downloaded: true
transformers/multi-qa-mpnet-base-dot-v1 - type: textual, downloaded: true
jinaai/jina-embeddings-v2-base-en - type: textual, downloaded: true
```

```bash
lantern-cli show-models --runtime openai --runtime-params '{ "api_token": "xxx" }'

[*] [Lantern Embeddings] Available Models

openai/text-embedding-ada-002 - sequence_len: 8192, dimensions: 1536
```

```bash
lantern-cli show-models --runtime cohere --runtime-params '{ "api_token": "xxx" }'

[*] [Lantern Embeddings] Available Models

cohere/embed-english-v3.0 - sequence_len: 512, dimensions: 1024
cohere/embed-multilingual-v3.0 - sequence_len: 512, dimensions: 1024
cohere/embed-english-v2.0 - sequence_len: 512, dimensions: 4096
cohere/embed-english-light-v2.0 - sequence_len: 512, dimensions: 1024
cohere/embed-multilingual-v2.0 - sequence_len: 512, dimensions: 768
cohere/embed-english-light-v3.0 - sequence_len: 512, dimensions: 384
cohere/embed-multilingual-light-v3.0 - sequence_len: 512, dimensions: 384
```

The model is `downloaded` if the model onnx file and tokenizer are already downloaded. If false, it will be automatically downloaded on the first run.

The `type` of the model can be either visual or textual. If text, the input should be a string. If visual, the input should be either an image url or local image path.
The `ort` runtime models will be run locally on GPU or CPU. `openai` or `cohere` runtime models will be run through HTTP API, so you need to provide API Token via `--runtime-params '{ "api_token": "xxx-xxxx-xxxxx" }'` argument
Default runtime is `ort`

## Set Up Data

Note: You can skip this step if you already have data in your database

```sql
CREATE TABLE articles (id SERIAL PRIMARY KEY, title TEXT);
INSERT INTO articles (title) VALUES ('What is vector search'), ('Getting your AI application up and running in minutes'), ('HNSW vs IVFFLAT');
```

## Run Embedding Generation

```bash
lantern-cli create-embeddings \
    --model 'microsoft/all-MiniLM-L12-v2'  \
    --uri 'postgresql://[username]:[password]@localhost:5432/[db]' \
    --table "articles" \
    --column "title" \
    --out-column "title_embedding" \
    --pk id \
    --batch-size 100
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
  -m, --model <MODEL>
          Model name
  -u, --uri <URI>
          Fully associated database connection string including db name
  -t, --table <TABLE>
          Table name
  -s, --schema <SCHEMA>
          Schema name [default: public]
  -p, --pk <PK>
          Table primary key column name [default: id]
  -c, --column <COLUMN>
          Column name to generate embeddings for
      --out-uri <OUT_URI>
          Output db uri, fully associated database connection string including db name. Defaults to
      --out-table <OUT_TABLE>
          Output table name. Defaults to table
      --out-column <OUT_COLUMN>
          Output column name
  -b, --batch-size <BATCH_SIZE>
          Batch size
      --runtime <RUNTIME>
          Runtime [default: ort]
      --runtime-params <RUNTIME_PARAMS>
          Runtime Params JSON string [default: {}]
      --visual
          If model is visual
  -o, --out-csv <OUT_CSV>
          Output csv path. If specified result will be written in csv instead of database
  -f, --filter <FILTER>
          Filter which will be used when getting data from source table
  -l, --limit <LIMIT>
          Limit will be applied to source table if specified
      --stream
          Stream data to output table while still generating
      --create-column
          Create destination column if not exists
  -h, --help
          Print help
  -V, --version
          Print version
```
