# Migrate from pgvector to Lantern Cloud

This guide assumes that you are using Postgres with `pgvector`, and that you want to use Lantern Cloud instead.

## Steps

1. Create a Lantern Cloud database

   Sign up for [Lantern Cloud](/) and create a database. Obtain a database URL. We'll call this `LANTERN_DATABASE_URL`.

2. Save pgvector indexes

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

5. Disable `pgvector` extension

   Disable the `pgvector` extension.

   ```sql
   DROP EXTENSION IF EXISTS vector;
   ```

6. Backup the Source Database

   Use the `pgdump` utility to create a database backup.

   ```bash
   pg_dump $OLD_DATABASE_URL > backup.sql
   ```

7. (Optional) Stop the old database

   You may want to disable the old database at this time, to prevent data from being dropped.

8. Transfer the Data

   ```sql
   psql $LANTERN_DATABASE_URL < backup.sql
   ```

9. Re-create your indexes

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

10. Use the new database

    Update any applications, scripts, or services to point to the new database.

## Support

Reach out to support@lantern.dev for any questions or assistance with migrations. We're happy to help.
