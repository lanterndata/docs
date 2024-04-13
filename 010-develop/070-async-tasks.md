# Asynchronous Tasks

## Motivation

Index creation on a large lable is a time-consuming operation. Vanilla Postgres does not have a built-in mechanism to run such time-consuming operations in the background, so a client must maintain a connection to the database for the whole duration of such an operation. In some cases, it is not desirable to keep a connection open for a long time for these operations. For example, when the backend interacting with the database is in a serverless environment.

For such cases, Lantern provides an API to run arbitrary queries in the background.

## Start an asynchronous query

```sql
SELECT lantern.async_task('QUERY_STRING');
```

For example, to launch an index creation in the background on column `v` of table `my_table`:

```sql
SELECT lantern.async_task('CREATE INDEX ON my_table USING hnsw(v) WITH (m=16, ef_construction=8, ef=16);');
```

The `lantern.async_task` function returns a `jobid` which can be used to track the progress of the task.

The `lantern.async_task` also accepts an optional `job_name` parameter which can be used to give a name to the task.

```sql
SELECT lantern.async_task('CREATE INDEX ON my_table USING hnsw(v) WITH (m=16, ef_construction=8, ef=16);', 'Create index on my_table');
```

You can view progress of asynchronous tasks by querying the `lantern.tasks` table which has the structure below:

```bash
                                               Table "lantern.tasks"
      Column      |           Type           | Collation | Nullable |                   Default
------------------+--------------------------+-----------+----------+----------------------------------------------
 jobid            | bigint                   |           | not null | nextval('lantern.tasks_jobid_seq'::regclass)
 query            | text                     |           | not null |
 pg_cron_job_name | text                     |           |          |
 job_name         | text                     |           |          |
 username         | text                     |           | not null | CURRENT_USER
 started_at       | timestamp with time zone |           | not null | now()
 duration         | interval                 |           |          |
 status           | text                     |           |          |
 error_message    | text                     |           |          |
```
