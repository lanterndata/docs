# Daemon

With the Lantern CLI Daemon, you can continuously generate embeddings, indexes and autotune jobs for your Postgres table data without affecting database performance.

## Prerequisites

- [Lantern CLI](/docs/lantern-cli/install)
- [ONNX Runtime](/docs/lantern-cli/install)
- Postgres database

## Architecture

![Lantern Daemon Architecture](https://storage.googleapis.com/lantern-web/daemon-architecture.jpg)

You should have `jobs` table in your database for embeddings, external indexes and autotune. The Daemon will read jobs from these tables, and set up continuous listeners to the target database (only for embedding jobs).

```sql
CREATE TABLE "public"."embedding_generation_jobs" (
    "id" SERIAL PRIMARY KEY,
    "database_id" text NOT NULL,
    "db_connection" text NOT NULL,
    "schema" text NOT NULL,
    "table" text NOT NULL,
    "src_column" text NOT NULL,
    "dst_column" text NOT NULL,
    "embedding_model" text NOT NULL,
    "created_at" timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "updated_at" timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "canceled_at" timestamp,
    "init_started_at" timestamp,
    "init_finished_at" timestamp,
    "init_failed_at" timestamp,
    "init_failure_reason" text,
    "init_progress" int2 DEFAULT 0
);

CREATE TABLE "public"."external_index_jobs" (
    "id" SERIAL PRIMARY KEY,
    "database_id" text NOT NULL,
    "db_connection" text NOT NULL,
    "schema" text NOT NULL,
    "table" text NOT NULL,
    "column" text NOT NULL,
    "index" text,
    "operator" text NOT NULL,
    "efc" INT NOT NULL,
    "ef" INT NOT NULL,
    "m" INT NOT NULL,
    "created_at" timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "updated_at" timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "canceled_at" timestamp,
    "started_at" timestamp,
    "finished_at" timestamp,
    "failed_at" timestamp,
    "failure_reason" text,
    "progress" INT2 DEFAULT 0
);

CREATE TABLE "public"."index_autotune_jobs" (
    "id" SERIAL PRIMARY KEY,
    "database_id" text NOT NULL,
    "db_connection" text NOT NULL,
    "schema" text NOT NULL,
    "table" text NOT NULL,
    "column" text NOT NULL,
    "metric_kind" text NOT NULL,
    "target_recall" int NOT NULL,
    "k" int NOT NULL,
    "create_index" bool NOT NULL,
    "embedding_model" text NULL,
    "created_at" timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "updated_at" timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "canceled_at" timestamp,
    "started_at" timestamp,
    "finished_at" timestamp,
    "failed_at" timestamp,
    "failure_reason" text,
    "progress" INT2 DEFAULT 0
);
```

After you have the `jobs` table set up you can run the daemon

```bash
lantern-cli start-daemon --uri 'postgresql://[username]:[password]@[host]:[port]/[dbname]' --embedding-table embedding_generation_jobs --external-index-table external_index_jobs --autotune-table index_autotune_jobs
```

Note: It is not required to have all the tables for different kind of jobs, for example you can run just embedding jobs and omit `--external-index-table` and `--autotune-table` arguments

And insert a new job

```sql
INSERT INTO embedding_generation_jobs (db_connection, schema, "table", src_column, dst_column, embedding_model)
VALUES
('postgres://postgres@localhost:5432/test', 'public', 'articles', 'title', 'title_embedding', 'microsoft/all-MiniLM-L12-v2');
```

The Daemon will pick up the job as soon as it appears on the table.

You can now add new rows to target database, and the Daemon will generate embeddings for those rows

```sql
-- Lantern Daemon will batch the insertions for the same table/column jobs and generate embeddings for them
INSERT INTO articles (title) VALUES ('My articles title');
```
