# Create Index

With the Lantern CLI's `create-index` routine, you can create an HNSW index externally to Postgres, without consuming database resources, and import it.

You can read more about how we were able to improve index creation times by 90x over `pgvector` in [this](/blog/hnsw-index-creation) blog post.

## Prerequisites

- [Lantern CLI](/docs/lantern-cli/install)
- Postgres database with the [Lantern extension](/docs/lantern-db/install) installed

### Set Up Data

Note: You can skip this step if you already have vector data in your database

```sql
CREATE TABLE embeddings (id SERIAL PRIMARY KEY, v REAL[]);
INSERT INTO embeddings (v) VALUES ('{0,0,0}'), ('{0,1,0}'), ('{1,0,0}');
```

## Run Index Creation

```bash
lantern-cli create-index \
    --uri 'postgresql://[username]:[password]@localhost:5432/[db]' \
    --table "embeddings" \
    --column "v" \
    -m 10 \
    --efc 128 \
    --ef 64 \
    --metric-kind l2sq \
    --out /tmp/index.usearch \
    --import
```

After this the index will be created and imported to your database, even if the database is on remote server!

**Make sure to provide database uri with superuser**

```sql
-- verify that index is created properly
SET enable_seqscan=false
SET lantern.pgvector_compat=false
-- you should see index scan on query planner
EXPLAIN SELECT * FROM embeddings WHERE v <?> ARRAY[1,1,1];
```

Note: External indexes should be reindexed using `SELECT lantern_reindex_external_index('<index_name>')` if you have `lantern_extras` extension installed or by running the same cli command with `--index-name` param specified.

## CLI parameters

Run `bash lantern-cli create-index --help` to get available CLI parameters

```bash
Create external index

Usage: lantern-cli create-index --uri <URI> --table <TABLE> --column <COLUMN> -d <DIMS> [MORE OPTIONS]

Options:
  -u, --uri <URI>                  Fully associated database connection string including db name
  -s, --schema <SCHEMA>            Schema name [default: public]
  -t, --table <TABLE>              Table name
  -c, --column <COLUMN>            Column name
  -m <M>                           Number of neighbours for each vector [default: 16]
      --efc <EFC>                  The size of the dynamic list for the nearest neighbors in construction [default: 128]
      --ef <EF>                    The size of the dynamic list for the nearest neighbors in search [default: 64]
  -d <DIMS>                        Dimensions of vector
      --metric-kind <METRIC_KIND>  Distance algorithm [default: l2sq] [possible values: l2sq, cos, hamming]
  -o, --out <OUT>                  Index output file [default: index.usearch]
  -i, --import                     Import index to database (should be run as db superuser to have access)
      --index-name <INDEX_NAME>    Index name to use when imporrting index to database
  -h, --help                       Print help
```
