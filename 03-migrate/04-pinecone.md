# Migrate from Pinecone to Lantern Cloud

This guide assumes that you are using Pinecone, and that you want to use Lantern Cloud instead.

## Steps

1. Create a Lantern Cloud database

   Sign up for [Lantern Cloud](/) and create a database. Obtain a database URL. We'll call this `LANTERN_DATABASE_URL`.

2. Install lantern-pinecone client

   The most straightforward way to migrate from Pinecone to Lantern Cloud is using [`lantern-pinecone`](https://github.com/lanterndata/lantern-python/blob/main/lantern_pinecone/README.md) client.
   Even if you don't want to use the Python client you can migrate the data and then use your Postgres instance with other clients.

   ```bash
   pip install lantern-pinecone
   ```

3. Initialize client and migrate

   Next, we will initialize the Lantern client

   ```python
   import lantern_pinecone
   from getpass import getpass

   lantern_pinecone.init(LANTERN_DATABASE_URL)
   ```

   As currently there is a limitation on Pinecone and you can not export all the data or ids of your embeddings, we assume that you have your ids stored externally somewhere and you can provide list of `ids` to client.

   For this example we will assume that the Pinecone has vectors with sequential ids from 0 to 100000

   ```python
   pinecone_ids = list(map(lambda x: str(x), range(100000)))
   index = lantern_pinecone.create_from_pinecone(
        api_key=<your_pinecone_api_key>,
        environment=<your_pinecone_environment>,
        index_name=<pinecone_index_name>,
        namespace="",
        pinecone_ids=pinecone_ids,
        m=16,
        ef=128,
        ef_construction=96,
        recreate=True
   )
   ```

4. Finishing steps

After this step the data will be copied to your database under table with the same name as your `<pinecone_index_name>` with `HNSW` index on `embedding` column and `GIN` index on `metadata` column

You can view index stats using

```python
index.describe_index_stats()
```

## Support

Reach out to support@lantern.dev for any questions or assistance with migrations. We're happy to help.
