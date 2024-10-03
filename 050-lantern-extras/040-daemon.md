# Lantern Daemon

Lantern Daemon can also be enabled inside the database.

## Run Lantern Daemon

To run Lantern Daemon inside the database as background worker you need to add `lantern_extras.so` in `shared_preload_libraries` in `postgresql.conf` file like this:

```
shared_preload_libraries='lantern_extras.so'
```

Then after restarting the database you need to set `lantern_extras.enable_daemon=true`

```sql
ALTER SYSTEM SET lantern_extras.enable_daemon=true;
SELECT pg_reload_conf();
```

After this the daemon will start and watch `postgres` database. 
If the table on which you want to add embedding job is on another database you will need to configure it using `lantern_extras.daemon_databases` GUC.

```sql
ALTER SYSTEM SET lantern_extras.daemon_databases='mydb';
-- Toggle daemon to restart and watch the specified database
ALTER SYSTEM SET lantern_extras.enable_daemon=false;
SELECT pg_reload_conf();
ALTER SYSTEM SET lantern_extras.enable_daemon=true;
SELECT pg_reload_conf();
```

To add embedding job use the following function:

```sql
SELECT add_embedding_job(
    'table_name',        -- Name of the table
    'src_column',        -- Source column for embeddings
    'dst_column',        -- Destination column for embeddings
    'embedding_model',   -- Embedding model to use
    'runtime',           -- Runtime environment (default: 'ort')
    'runtime_params',    -- Runtime parameters (default: '{}')
    'pk',                -- Primary key column (default: 'id')
    'schema'             -- Schema name (default: 'public')
);
```

For `openai` and `cohere` runtimes if the `api_token` will not be provided via `runtime_params` it will be set from `lantern_extras.openai_token` or `lantern_extras.cohere_token` GUC variables  

**Getting Embedding Job Status**  
To get the status of an embedding job, use the `get_embedding_job_status` function:

```sql
SELECT * FROM get_embedding_job_status(job_id);
```
This will return a table with the following columns:

- `status`: The current status of the job.
- `progress`: The progress of the job as a percentage.
- `error`: Any error message if the job failed.

**Getting All Embedding Jobs**  
To get the status of all embedding jobs, use the `get_embedding_jobs` function:

```sql
SELECT * FROM get_embedding_jobs();

```
This will return a table with the following columns:

- `id`: Id of the job
- `status`: The current status of the job.
- `progress`: The progress of the job as a percentage.
- `error`: Any error message if the job failed.

**Canceling an Embedding Job**  
To cancel an embedding job, use the `cancel_embedding_job` function:

```sql
SELECT cancel_embedding_job(job_id);
```

**Resuming an Embedding Job**  
To resume a paused embedding job, use the `resume_embedding_job` function:

```sql
SELECT resume_embedding_job(job_id);
```
