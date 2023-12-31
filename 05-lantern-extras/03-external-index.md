# Create External index

Lantern Extras enables creating external index right from the SQL without using `lantern-cli`

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
	"index_name" TEXT DEFAULT
);
```

```sql
CREATE TABLE embeddings (id SERIAL PRIMARY KEY, v REAL[]);
INSERT INTO embeddings (v) VALUES ('{0,0,0}'), ('{0,1,0}'), ('{1,0,0}');
SELECT lantern_create_external_index('v', 'embeddings');
```

Then this index can be reindexed using the following function:

```sql
SELECT lantern_reindex_external_index('embeddings_v_idx');
```
