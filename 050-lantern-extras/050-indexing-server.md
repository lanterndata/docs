# Lantern Indexing Server

Lantern Indexing Server is also available as background worker inside the database

## Run Lantern Indexing Server

To run Lantern Indexing Server inside the database as background worker you need to add `lantern_extras` in `shared_preload_libraries` in `postgresql.conf` file like this:

```
shared_preload_libraries='lantern_extras'
```

Then after restarting the database the server will be enabled by default and bind to 127.0.0.1:8998


You can create an index using the server by configuring GUC variables like this:

```sql
SET lantern.external_index_host='127.0.0.1';
SET lantern.external_index_port=8998;
SET lantern.external_index_secure=false;

CREATE INDEX ON embeddings USING lantern_hnsw(v dist_cos_ops) WITH (m=12, ef_construction=64, external=true);
```

## Stop Lantern Indexing Server
```sql
ALTER SYSTEM SET lantern_extras.enable_indexing_server=false;
SELECT pg_reload_conf();
```
