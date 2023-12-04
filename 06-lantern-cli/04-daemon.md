# Daemon

With the Lantern CLI Daemon, you can continuously generate embeddings for your Postgres table data without affecting database performance.

## Prerequisites

- [Lantern CLI](/lantern-cli/install)
- [ONNX Runtime](/lantern-cli/install)
- Postgres database

## Architecture

![Lantern Daemon Architecture](https://storage.googleapis.com/lantern-web/daemon-architecture.jpg)

You should have `jobs` table in your database. The Daemon will read jobs from this table, and set up continuous listeners for the target database.

```sql
CREATE TABLE jobs (
  id SERIAL PRIMARY KEY,
  db_connection TEXT NOT NULL, -- target database connection url. Like postgresql://[username]:[password]@[host]:[port]/[dbname]
  init_started_at TIMESTAMP, -- first time when job was started
  init_failed_at TIMESTAMP, -- if the job is failed during the first run
  init_finished_at TIMESTAMP, -- first run finish time
  init_failure_reason TEXT, -- error message if job will be failed
  canceled_at BOOL, -- if set to true all listeners will be closed for that job. This can be change while daemon is running
  schema TEXT, -- target schema name in destination database (default is public)
  table TEXT, -- target table name in destination database
  src_column TEXT, -- name of the source column in target database table
  dst_column TEXT, -- name of the destination column in target database table under which the embeddings will be generated
  embedding_model TEXT, -- model name to use (you can get the models by running lantern-cli show-models)
)
```

After you have the `jobs` table set up you can run the daemon

```bash
lantern-cli start-daemon --uri 'postgresql://[username]:[password]@[host]:[port]/[dbname]' --table jobs
```

And insert a new job

```sql
INSERT INTO jobs (db_connection, schema, "table", src_column, dst_column, embedding_model)
VALUES
('postgres://postgres@localhost:5432/test', 'public', 'articles', 'title', 'title_embedding', 'microsoft/all-MiniLM-L12-v2');
```

The Daemon will pick up the job as soon as it appears on the table.

You can now add new rows to target database, and the Daemon will generate embeddings for those rows

```sql
-- Lantern Daemon will batch the insertions for the same table/column jobs and generate embeddings for them
INSERT INTO articles (title) VALUES ('My articles title');
```
