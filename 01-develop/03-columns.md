# Automatic Embedding Generation

## Lantern Cloud

[Lantern Cloud](/) supports automatic embedding generation. This means that Lantern will automatically generate embeddings for your data as you insert it into the database. This is useful for applications where you want to generate embeddings for new data as it comes in, such as for recommendation systems or similarity search.

To get started, navigate to the Embeddings page in the dashboard. You can generate an embedding column for a table by specifying

- Source column (e.g., `book_summary`),
- Embedding Model (e.g., `openai/text-embedding-ada-002` or `BAAI/bge-small-en`)
- Embedding column to insert the data into (e.g., `book_summary_embedding`)

This will generate embeddings for your existing data, automatically update embeddings when the source column is updated, and generate embeddings for new rows as they are inserted.

![Automatic Embedding generation](https://lantern.dev/videos/vector.gif)

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

You can also use the Lantern CLI [daemon](/docs/lantern-cli/daemon) to listen to updates and inserts to update embeddings.

The Lantern Cloud dashboard combines both of these processes in a seamless experience.
