# Create External index

The Lantern Extras Postgres extension enables creating an external index directly from SQL using the `lantern_create_external_index` function and the `lantern_reindex_external_index` function.

## Run Index Creation

It provides the following function to create an external index

```sql
lantern_create_external_index(
	"column" TEXT,
	"table" TEXT,
	"schema" TEXT DEFAULT 'public',
	"metric_kind" TEXT DEFAULT 'l2sq',
	"dim" INT DEFAULT 0,
	"m" INT DEFAULT 16,
	"ef_construction" INT DEFAULT 16,
	"ef" INT DEFAULT 16,
	"pq" BOOL DEFAULT FALSE,
	"index_name" TEXT DEFAULT
);
```

For example, the code below creates an external index `embeddings_v_idx` on the `v` column of the `embeddings` table.

```sql
-- Create table and add some data
CREATE TABLE embeddings (id SERIAL PRIMARY KEY, v REAL[]);
INSERT INTO embeddings (v) VALUES ('{0,0,0}'), ('{0,1,0}'), ('{1,0,0}');

-- Create external index
SELECT lantern_create_external_index('v', 'embeddings');
```

## Reindexing

In time, we may want to re-index an existing external index due to significant changes in the data distribution. This can be done in SQL using the function `lantern_reindex_external_index`, which accepts as input the index name.

For example, to re-index the `embeddings_v_idx` index from above, run:

```sql
SELECT lantern_reindex_external_index('embeddings_v_idx');
```
