# Populate an embedding column

## Lantern Cloud

[Lantern Cloud](/) seamlessly supports automatically generating embeddings and inserting them into a column. To get started, navigate to the Embeddings page in the dashboard. You can generate an embedding column for a table by specifying

- Source column (e.g., `book_summary`),
- Model (e.g., `BAAI/bge-small-en`)
- Embedding column to insert the data into (e.g., `book_summary_embedding`)

As before, the following models are available for generating embeddings.

CONTENT_VAR_MODELS

## Self-Hosting

Generating embedding columns requires the Lantern CLI. Installation steps are found [here](/docs/lantern-cli/install).

Using the CLI, you can generate embeddings in a [one-off job](/docs/lantern-cli/embeddings).

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

You can also use the Lantern CLI [daemon](/docs/lantern-cli/daemon) to continuously generate embeddings for new columns.

The Lantern Cloud dashboard combines both of these processes in a seamless UI.
