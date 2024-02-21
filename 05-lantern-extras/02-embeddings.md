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
SET lantern_extras.openai_azure_api_token='xxxxxxxxxxxxx'; -- For Azure deployment with API Key authentication
SET lantern_extras.openai_azure_entra_token='xxxxxxxxxxxxx'; -- For Azure deployment with Microsoft Entra ID authentication
SET lantern_extras.openai_deployment_url='https://YOUR_RESOURCE_NAME.openai.azure.com/openai/deployments/YOUR_DEPLOYMENT_NAME/embeddings?api-version=2023-05-15' -- You can set this GUC or pass via arguments

SELECT openai_embedding('openai/text-embedding-ada-002', 'My text input');
SELECT openai_embedding('openai/text-embedding-ada-002', 'My text input', 'https://YOUR_RESOURCE_NAME.openai.azure.com/openai/deployments/YOUR_DEPLOYMENT_NAME/embeddings?api-version=2023-05-15'); -- Use azure deployment
SELECT openai_embedding('openai/text-embedding-v3-small', 'My text input', '', 768); -- Provide dimensions for new models
SELECT openai_embedding('openai/text-embedding-v3-large', 'My text input', '', 3072); -- Provide dimensions for new models
```

> For more info about azure_api_token and azure_entra_token variables refer to [Azure Docs](https://learn.microsoft.com/en-us/azure/ai-services/openai/reference#authentication)

Cohere embeddings

```sql
SET lantern_extras.cohere_token='xxxxxxxxxxxxx';
SELECT cohere_embedding('cohere/embed-multilingual-v3.0 ', 'My text input');
SELECT cohere_embedding('cohere/embed-multilingual-v3.0 ', 'My text input', 'search_query'); -- This is the default type for embedding. Use this when doing queries
SELECT cohere_embedding('cohere/embed-multilingual-v3.0 ', 'My text input', 'search_document'); -- This is type for embedding when you want to store in database
```

> For more info about embedding type refer to [Cohere Docs](https://docs.cohere.com/reference/embed)

## Supported Models

The following embedding models are currently supported:

CONTENT_VAR_MODELS
