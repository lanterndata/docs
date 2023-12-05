# Create Index

With the Lantern CLI, you can create an HNSW index externally to Postgres, without consuming database resources, and import it.

You can read more about the benchmarking and performance of Lantern Index in [this](/blog/hnsw-index-creation) blog post.

## Prerequisites

- [Lantern CLI](/docs/lantern-cli/install)
- Postgres database with the [Lantern extension](/docs/lantern-db/install) installed

## Set Up Data

Note: You can skip this step if you already have vector data in your database

```sql
CREATE TABLE embeddings (id SERIAL PRIMARY KEY, v REAL[]);
INSERT INTO embeddings (v) VALUES ('{0,0,0}'), ('{0,1,0}'), ('{1,0,0}');
```

## Run Index Creation

```bash
lantern-cli create-index --uri 'postgresql://[username]:[password]@localhost:5432/[db]' --table "embeddings" --column "v" -m 10 --efc 128 --ef 64 --metric-kind l2sq --out /tmp/index.usearch
```

Note: If you have generated index file in a different server, copy the index file to your database server in a location where the user running database will have the necessary permissions to access the file. You can put it in `/tmp/index.usearch` and run `chmod 766 /tmp/index.usearch` on it.

## Import Index to Database

You can now import the index file to your database and use the index!

```sql
CREATE INDEX ON embeddings USING hnsw (v) WITH (_experimental_index_file='/tmp/index.usearch');

-- verify that index is created properly
SET enable_seqscan=false
-- you should see index scan on query planner
EXPLAIN SELECT * FROM embeddings WHERE v <-> ARRAY[1,1,1];
```

## CLI parameters

Run `bash lantern-cli create-index --help` to get available CLI parameters

```bash
Create external index

Usage: lantern-cli create-index --uri <URI> --table <TABLE> --column <COLUMN> -d <DIMS> [MORE OPTIONS]

Options:
  -u, --uri <URI>                  Fully associated database connection string including db name
  -t, --table <TABLE>              Table name
  -c, --column <COLUMN>            Column name
  -m <M>                           Number of neighbours for each vector [default: 16]
      --efc <EFC>                  The size of the dynamic list for the nearest neighbors in construction [default: 128]
      --ef <EF>                    The size of the dynamic list for the nearest neighbors in search [default: 64]
  -d <DIMS>                        Dimensions of vector
      --metric-kind <METRIC_KIND>  Distance algorithm [default: l2sq] [possible values: l2sq, cos, hamming]
  -o, --out <OUT>                  Index output file [default: index.usearch]
  -h, --help                       Print help
```
