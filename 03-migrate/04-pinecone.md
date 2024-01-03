# Migrate from Pinecone to Lantern Cloud

This guide assumes that you are using Pinecone, and that you want to use Lantern Cloud instead.

## Steps

1. Create a Lantern Cloud database

   Sign up for [Lantern Cloud](/) and create a database. Obtain a database URL. We'll call this `LANTERN_DATABASE_URL`.

2. Install the `lantern-pinecone` client

   The most straightforward way to migrate from Pinecone to Lantern Cloud is by using the [`lantern-pinecone`](https://github.com/lanterndata/lantern-python/blob/main/lantern_pinecone/README.md) Python client. Even if you don't want to use `lantern-pinecone` as your primary data client, you can use it to migrate the data and then interact with your data using another data client.

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

   [Currently]((https://support.pinecone.io/hc/en-us/articles/12438275491741-How-do-I-export-my-Pinecone-index-)) Pinecone does not allow you to export all the data or IDs of your embeddings. We assume that you have your IDs stored and can provide a list of IDs to the client via the `pinecone_ids` field below.

   In the example below, we assume that the Pinecone has vectors with sequential ids from 0 to 100000.

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

   If you do not have your IDs saved, you can leave `pinecone_ids` blank, and we will automatically retrieve your Pinecone IDs using the known [workaround](https://community.pinecone.io/t/how-to-retrieve-list-of-ids-in-an-index/380).

4. Final steps

   After this step the data will be copied to your database under a table with the same name as your `<pinecone_index_name>` with `HNSW` index on `embedding` column and `GIN` index on the `metadata` column

   You can view index stats using

   ```python
   index.describe_index_stats()
   ```

## Support

To read more about the `lantern-pinecone` client, check out the [Github repo](https://github.com/lanterndata/lantern-python/tree/main/lantern_pinecone).

Reach out to support@lantern.dev for any questions or assistance with migrations. We're happy to help.
