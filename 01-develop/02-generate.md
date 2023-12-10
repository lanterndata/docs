# Generate embeddings

Lantern supports generating embeddings inside the database for one-off transactions. Note that generating embeddings is a CPU-intensive task and large scale embedding generation processes. Lantern provides a [separate process](/docs/develop/generate) for large scale embedding generation.

The following embedding models are supported

CONTENT_VAR_MODELS

## Lantern Cloud

[Lantern Cloud](/) supports one-off embedding generation out of the box.

To generate text embeddings, use the `text_embedding` function. For example, to generate an embedding for the text `My text input`, run using the embedding model `BAAI/bge-small-en`, run

```sql
SELECT text_embedding('BAAI/bge-small-en', 'My text input');
```

To generate image embeddings, use the `image_embedding` function. For example, to generate an embedding for the image `https://lantern.dev/images/home/footer.png`, run using the embedding model `clip/ViT-B-32-visual`, run

```sql
SELECT image_embedding('clip/ViT-B-32-visual', 'https://lantern.dev/images/home/footer.png');
```

## Self-Hosting

Generating one-off embeddings requires the Lantern Extras extension. Installation steps are found [here](/docs/lantern-extras/install).

Once the extension is installed, the above functions are available.
