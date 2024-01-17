# Generate Embeddings

The Lantern Extras Postgres extension enables generating embeddings using SQL with the functions `text_embedding` and `image_embedding`.

Note that generating embeddings is a CPU-intensive task and large scale embedding generation processes. For large scale embedding generation, the Lantern CLI provides a [separate process](/docs/lantern-cli/embeddings).

## Run Embedding Generation

To generate one-off text embeddings, use the `text_embedding` function. For example, to generate an embedding for the text `My text input` using the embedding model `BAAI/bge-small-en`, run

```sql
SELECT text_embedding('BAAI/bge-small-en', 'My text input');
```

To generate image embeddings, use the `image_embedding` function. For example, to generate an embedding for the image `https://lantern.dev/images/home/footer.png` using the embedding model `clip/ViT-B-32-visual`, run

```sql
SELECT image_embedding('clip/ViT-B-32-visual', 'https://lantern.dev/images/home/footer.png');
```

The above mentioned functions will use local models using `ort` runtime.

If you want to generate embeddings using OpenAI or Cohere APIs you can use the following functions:

```sql
SET lantern_extras.openai_token='xxxxxxxxxxxxx';
SELECT openai_embedding('openai/text-embedding-ada-002', 'My text input');

SET lantern_extras.cohere_token='xxxxxxxxxxxxx';
SELECT cohere_embedding('cohere/embed-multilingual-v3.0 ', 'My text input');
```

## Supported Models

The following embedding models are currently supported:

CONTENT_VAR_MODELS
