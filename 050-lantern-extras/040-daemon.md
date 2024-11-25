# Lantern Daemon

Lantern Daemon can also be enabled inside the database.

## Run Lantern Daemon

To run Lantern Daemon inside the database as background worker you need to add `lantern_extras` in `shared_preload_libraries` in `postgresql.conf` file like this:

```
shared_preload_libraries='lantern_extras'
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
    table_name => 'articles', -- Name of the table
    src_column => 'content', -- Source column for embeddings
    dst_column => 'content_embedding', -- Destination column for embeddings (will be created automatically)
    model => 'text-embedding-3-small', -- Model for runtime to use (default: 'text-embedding-3-small')
    pk => 'id', -- Primary key of the table. It is required for table to have primary key (default: id)
    schema => 'public', -- Schema on which the table is located (default: 'public')
    base_url => 'https://api.openai.com', -- If you have custom LLM deployment provide the server url. (default: OpenAi API URL)
    batch_size => 500, -- Batch size for the inputs to use when requesting LLM server. This is based on your API tier. (default: determined based on model and runtime)
    dimensions => 1536, -- For new generation OpenAi models you can provide dimensions for returned embeddings. (default: 1536)
    api_token => '<llm_api_token>', -- API token for LLM server. (default: inferred from lantern_extras.llm_token GUC)
    azure_entra_token => '', -- If this is Azure deployment it supports Auth with entra token too
    runtime => 'openai' -- Runtime to use. (default: 'openai'). Use `SELECT get_available_runtimes()` for list
);
```

For `openai` and `cohere` runtimes if the `api_token` will not be provided via `runtime_params` it will be set from `lantern_extras.llm_token`

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

**Adding a Completion Job**  
To add a new completion job, use the `add_completion_job` function:

```sql
SELECT add_completion_job(
    table_name => 'articles', -- Name of the table
    src_column => 'content', -- Source column for embeddings
    dst_column => 'content_summary', -- Destination column for llm response (will be created automatically)
    system_prompt => 'Provide short summary for the given text', -- System prompt for LLM (default: '')
    column_type => 'TEXT', -- Destination column type
    model => 'gpt-4o', -- Model for runtime to use (default: 'gpt-4o')
    pk => 'id', -- Primary key of the table. It is required for table to have primary key (default: id)
    schema => 'public', -- Schema on which the table is located (default: 'public')
    base_url => 'https://api.openai.com', -- If you have custom LLM deployment provide the server url. (default: OpenAi API URL)
    batch_size => 10, -- Batch size for the inputs to use when requesting LLM server. This is based on your API tier. (default: determined based on model and runtime)
    api_token => '<llm_api_token>', -- API token for LLM server. (default: inferred from lantern_extras.llm_token GUC)
    azure_entra_token => '', -- If this is Azure deployment it supports Auth with entra token too
    runtime => 'openai' -- Runtime to use. (default: 'openai'). Use `SELECT get_available_runtimes()` for list
);
```

**Getting All Completion Jobs**  
To get the status of all completion jobs, use the `get_completion_jobs` function:

```sql
SELECT * FROM get_completion_jobs();

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

**Getting All Failed Rows for Completion Job**  
To get failed rows for completion job, use the `get_completion_job_failures(job_id)` function:

```sql
SELECT * FROM get_completion_job_failures(1);

```
This will return a table with the following columns:

- `row_id`: Primary key of the failed row in source table
- `value`: The value returned from LLM response

### LLM Query

***Calling LLM Completion API***
```sql
SET lantern_extras.llm_token='xxxx'; -- this will be used as api_token if it is not passed via arguments
SELECT llm_completion(
    user_prompt => 'User input', -- User prompt to LLM model
    model => 'gpt-4o', -- Model for runtime to use (default: 'gpt-4o')
    system_prompt => 'Provide short summary for the given text', -- System prompt for LLM (default: '')
    base_url => 'https://api.openai.com', -- If you have custom LLM deployment provide the server url. (default: OpenAi API URL)
    api_token => '<llm_api_token>', -- API token for LLM server. (default: inferred from lantern_extras.llm_token GUC)
    azure_entra_token => '', -- If this is Azure deployment it supports Auth with entra token too
    runtime => 'openai' -- Runtime to use. (default: 'openai'). Use `SELECT get_available_runtimes()` for list
);
```
