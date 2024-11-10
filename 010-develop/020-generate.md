# Generate Embeddings

Lantern supports generating text and image embeddings inside the database. Try it out on [Lantern Cloud](/).

Note that generating embeddings is a compute-intensive task. For large scale embedding generation, such as generating embeddings over all of your data, Lantern provides a [separate process](/docs/develop/generate).

## Open AI Text Embeddings

Before using Open AI text embeddings, you need to have an Open AI API key. You can get one by signing up at [Open AI](https://openai.com/). Once you have an API key, set it as a parameter in Postgres.

```sql
ALTER ROLE [YOUR_USERNAME] SET lantern_extras.openai_token='[YOUR_API_KEY]';
SELECT pg_reload_conf(); 
```

Use the `openai_embedding` function to generate text embeddings using the Open AI embedding models. This function accepts a model name and text input as arguments, and for the `text-embedding-3-small` and `text-embedding-3-large` models, an optional dimension argument.

```sql
SELECT openai_embedding('openai/text-embedding-ada-002', 'My text input');
SELECT openai_embedding('openai/text-embedding-3-large', 'My text input');
SELECT openai_embedding('openai/text-embedding-3-large', 'My text input', 256);
```

The following embedding models are supported

CONTENT_VAR_MODELS_OPENAI

## Cohere Text Embeddings

Before using Cohere text embeddings, you need to have a Cohere API key. You can get one by signing up at [Cohere](https://cohere.ai/). Once you have an API key, set it as a parameter in Postgres.

```sql
ALTER ROLE [YOUR_USERNAME] SET lantern_extras.cohere_token='[YOUR_API_KEY]';
SELECT pg_reload_conf(); 
```

Use the `cohere_embedding` function to generate text embeddings using the Cohere embedding models. This function accepts a model name and text input as arguments, and an optional input type argument with values `search_document` or `search_query` (default is `search_query`).

```sql
SELECT cohere_embedding('cohere/embed-english-v3.0', 'My text input');
SELECT cohere_embedding('cohere/embed-english-v3.0', 'My text input', 'search_document');
```

The following embedding models are supported

CONTENT_VAR_MODELS_COHERE

## Open-Source Text Embeddings

For example, to generate an embedding for the text `My text input` using the open-source embedding model `BAAI/bge-small-en` in SQL, run

```sql
SELECT text_embedding('BAAI/bge-small-en', 'My text input');
```

The following embedding models are supported

CONTENT_VAR_MODELS_OPEN

## Image Embeddings

To generate image embeddings, use the `image_embedding` function. This function accepts a model name and image URL as arguments.

For example, to generate an embedding for the image `https://lantern.dev/images/home/footer.png` using the embedding model `clip/ViT-B-32-visual`, run

```sql
SELECT image_embedding('clip/ViT-B-32-visual', 'https://lantern.dev/images/home/footer.png');
```

The following embedding models are supported

CONTENT_VAR_MODELS_IMAGE

## Self-Hosting

For people self-hosting, generating embeddings requires the Lantern Extras extension. Installation steps are found [here](/docs/lantern-extras/install).

Once the extension is installed, the above functions are available.
