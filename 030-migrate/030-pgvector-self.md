# Migrate from pgvector to self-hosted Lantern

This guide assumes that you are self-hosting Postgres with `pgvector`, and that you want to self-host Postgres with Lantern.

## Use alongside `pgvector`

Lantern works alongside `pgvector` and is compatible with the `vector` type. If you already have a Postgres database running with `pgvector`, you can easily install and try out Lantern without encountering any issues.

Install Lantern using one of the methods mentioned in the [Getting Started](/docs/getting-started/overview) section. Lantern will automatically detect the presence of the existing `hnsw` access method in your database and create `lantern_hnsw` instead.

Suppose you already have a Postgres table with a vector index created using pgvector, like this:

```sql
CREATE EXTENSION vector;
CREATE TABLE lantern_pgvector (id SERIAL, v vector(3)); -- 'vector' is the type provided by pgvector
INSERT INTO lantern_pgvector (v) VALUES ('[0,0,0]'), ('[0,1,0]'), ('[1,0,0]');
CREATE INDEX vector_idx ON lantern_pgvector USING lantern_hnsw(v vector_l2_ops) WITH (m=4, ef_construction=8); -- This 'hnsw' access method is provided by pgvector
```

You can install Lantern over it without any issues

```sql
SET enable_seqscan=off; -- Always use the index on scans for demonstration purposes

-- Lantern will detect the presence of another 'hnsw' access method in your database and will add 'lantern_hnsw' instead. (A warning will be displayed in your psql console)
CREATE EXTENSION lantern;
CREATE INDEX lantern_idx ON lantern_pgvector USING lantern_hnsw(v) WITH (m=4, ef_construction=8);
DROP INDEX vector_idx;
SELECT * FROM lantern_pgvector ORDER BY v <-> '[1,1,1]'; -- 'lantern_idx' will be used for this scan
EXPLAIN SELECT * FROM lantern_pgvector ORDER BY v <-> '[1,1,1]'; -- You can verify that 'lantern_idx' is used here
```

## Migrate from `pgvector`

To completely remove `pgvector` from your database, there are a few approaches. The approach below is the simplest and will not result in any data loss, but there will be a brief period of time during which your data will be unindexed and vector queries will not work.

1. Drop Lantern Extension

   If you tested out Lantern earlier, you can drop the Lantern extension, and any corresponding indices.

   ```sql
   DROP EXTENSION IF EXISTS lantern CASCADE;
   ```

2. Save `pgvector` indexes

   To get the commands that were used to generate the `pgvector` indexes, run the following query

   ```sql
   SELECT
       x.indexdef
   FROM
       pg_indexes x
   WHERE
       x.indexdef LIKE '%hnsw%';
   ```

   It should generate output such as

   ```sql
   CREATE INDEX your_table_embedding_idx ON public.your_table USING hnsw (embedding vector_l2_ops) WITH (m='16', ef_construction='64')
   ```

   Save this output for later to re-create your vector indexes.

3. Drop `pgvector` indexes

   Next, drop all indexes created by `pgvector`. Note that by doing this, your queries will now be unindexed.

   ```sql
   DO $$
   DECLARE
       r RECORD;
   BEGIN
       FOR r IN SELECT indexname FROM pg_indexes WHERE indexdef LIKE '%hnsw%'
       LOOP
           EXECUTE 'DROP INDEX IF EXISTS ' || quote_ident(r.indexname);
       END LOOP;
   END $$;
   ```

   To do this manually, your queries will look something like this

   ```sql
   DROP INDEX IF EXISTS index_name;
   ```

4. Migrate vector columns to `REAL[]` columns

   Then, you will first need to migrate your data type from `vector` to `REAL[]`.

   ```sql
   DO $$
   DECLARE
       r record;
   BEGIN
       FOR r IN
           SELECT
               t.table_name,
               c.column_name,
               format('ALTER TABLE %I ALTER COLUMN %I TYPE REAL[] USING %I::REAL[]', t.table_name, c.column_name, c.column_name) as alter_stmt
           FROM
               information_schema.columns c
               JOIN information_schema.tables t ON t.table_name = c.table_name
           WHERE
               c.udt_name = 'vector'
       LOOP
           EXECUTE r.alter_stmt;
       END LOOP;
   END $$;
   ```

   To do this manually, the queries will look something like this

   ```sql
   ALTER TABLE table_name ALTER COLUMN vector_column_name TYPE REAL[];
   ```

5. Disable `pgvector` and enable Lantern

   Disable the `pgvector` extension, and enable the Lantern extension.

   ```sql
   DROP EXTENSION IF EXISTS vector;
   CREATE EXTENSION IF NOT EXISTS lantern;
   ```

6. Re-create your indexes

   Lantern has a slightly different syntax for creating HNSW indexes than `pgvector`.

   `pgvector` uses the syntax

   ```sql
   CREATE INDEX ON items USING hnsw (embedding vector_l2_ops) WITH (m = 16, ef_construction = 64);
   ```

   Lantern uses the syntax

   ```sql
   CREATE INDEX ON small_world USING hnsw (vector dist_l2sq_ops)
   WITH (M=2, ef_construction=10, ef=4, dim=3);
   ```

   Edit the output from step 2 to re-create your original `pgvector` indexes with Lantern. Run the new commands. And you're done!

## Support

If you're looking for a zero-downtime migration off of `pgvector`, reach out to support@lantern.dev. We're happy to help with that or with any other questions about migrating from `pgvector`.
