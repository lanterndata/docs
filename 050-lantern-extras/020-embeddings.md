# Generate Embeddings

The Lantern Extras Postgres extension enables generating embeddings using SQL with the function `llm_embedding`

Note that generating embeddings is a CPU-intensive task and large scale embedding generation processes. For large scale embedding generation, the Lantern CLI provides a [separate process](/docs/lantern-cli/embeddings).

## Run Embedding Generation

To generate one-off text embeddings, use the `llm_embedding` function. For example, to generate an embedding for the text `My text input` using the embedding model `BAAI/bge-small-en`, run

```sql
SELECT llm_embedding(
    input => 'User input', -- User prompt to LLM model
    model => 'gpt-4o', -- Model for runtime to use (default: 'gpt-4o')
    base_url => 'https://api.openai.com', -- If you have custom LLM deployment provide the server url. (default: OpenAi API URL)
    api_token => '<llm_api_token>', -- API token for LLM server. (default: inferred from lantern_extras.llm_token GUC)
    azure_entra_token => '', -- If this is Azure deployment it supports Auth with entra token too
    dimensions => 1536, -- For new generation OpenAi models you can provide dimensions for returned embeddings. (default: 1536)
    input_type => 'search_query', -- Needed only for cohere runtime to indicate if this input is for search or storing. (default: 'search_query'). Can also be 'search_document'
    runtime => 'openai' -- Runtime to use. (default: 'openai'). Use `SELECT get_available_runtimes()` for list
);

-- generate text embedding
SELECT llm_embedding(model => 'BAAI/bge-base-en', input => 'My text input', runtime => 'ort');
-- generate image embedding with image url
SELECT llm_embedding(model => 'clip/ViT-B-32-visual', input => 'https://lantern.dev/images/home/footer.png', runtime => 'ort');
-- generate image embedding with image path (this path should be accessible from postgres server)
SELECT llm_embedding(model => 'clip/ViT-B-32-visual', input => '/path/to/image/in-postgres-server', runtime => 'ort');
-- get available list of models
SELECT get_available_models();

If you want to generate embeddings using OpenAI or Cohere APIs you can use the following functions:

```sql
-- generate openai embeddings
SELECT llm_embedding(model => 'text-embedding-3-small', api_token => '<openai_api_token>', input => 'My text input', runtime => 'openai');
-- generate cohere embeddings
SELECT llm_embedding(model => 'embed-multilingual-light-v3.0', api_token => '<cohere_api_token>', input => 'My text input', runtime => 'cohere');
```
> For more info about embedding type refer to [Cohere Docs](https://docs.cohere.com/reference/embed)

```sql
-- api_token can be set via GUC
SET lantern_extras.llm_token = '<api_token>';
SELECT llm_embedding(model => 'text-embedding-3-small', input => 'My text input', runtime => 'openai');
```

```sql
SET lantern_extras.openai_azure_entra_token='xxxxxxxxxxxxx'; -- For Azure deployment with Microsoft Entra ID authentication
SET lantern_extras.llm_deployment_url='https://YOUR_RESOURCE_NAME.openai.azure.com/openai/deployments/YOUR_DEPLOYMENT_NAME/embeddings?api-version=2023-05-15' -- You can set this GUC or pass via arguments
```

> For more info about azure_api_token and azure_entra_token variables refer to [Azure Docs](https://learn.microsoft.com/en-us/azure/ai-services/openai/reference#authentication)

## Supported Models

The following embedding models are currently supported:

CONTENT_VAR_MODELS
