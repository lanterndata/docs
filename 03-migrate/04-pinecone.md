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

   Next, we will initialize the Lantern client.

   ```python
   import lantern_pinecone

   LANTERN_DATABASE_URL = "<your_lantern_database_url>"
   lantern_pinecone.init(LANTERN_DATABASE_URL)
   ```

   For optimal performance, provide a list of `pinecone_ids` to the client. Otherwise, the client will use a workaround to query the Pinecone API to retrieve the IDs. For more details, see [[1]](https://support.pinecone.io/hc/en-us/articles/12438275491741-How-do-I-export-my-Pinecone-index-) [[2]](https://community.pinecone.io/t/how-to-retrieve-list-of-ids-in-an-index/380).

   In the example below, we assume that the Pinecone has vectors with sequential ids from 0 to 1000.

   ```python
   pinecone_ids = list(map(lambda x: str(x), range(1000)))

   index = lantern_pinecone.create_from_pinecone(
        api_key=<your_pinecone_api_key>,
        environment=<your_pinecone_environment>,
        index_name=<pinecone_index_name>,
        pinecone_ids=pinecone_ids,
   )
   ```

   See the [documentation](https://github.com/lanterndata/lantern-python/blob/main/lantern_pinecone) for more details on the `lantern-pinecone` client.

4. Final steps

   After this step the data will be copied to your database under a table with the same name as your `<pinecone_index_name>` with `HNSW` index on `embedding` column and `GIN` index on the `metadata` column

   You can view index stats using

   ```python
   index.describe_index_stats()
   ```

## Support

To read more about the `lantern-pinecone` client, check out the [Github repo](https://github.com/lanterndata/lantern-python/tree/main/lantern_pinecone).

Reach out to support@lantern.dev for any questions or assistance with migrations. We're happy to help.
