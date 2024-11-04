# Indexing Server

With Lantern CLI's `start-indexing-server` routine, you can create an HNSW index externally to Postgres, without consuming database resources.

## Prerequisites

- [Lantern CLI](/docs/lantern-cli/install)
- Postgres database with the [Lantern HNSW extension](/docs/lantern-hnsw/install) installed

### Set Up Data

Note: You can skip this step if you already have vector data in your database

```sql
CREATE TABLE embeddings (id SERIAL PRIMARY KEY, v REAL[]);
INSERT INTO embeddings (v) VALUES ('{0,0,0}'), ('{0,1,0}'), ('{1,0,0}');
```

## Run Indexing Server

```bash
lantern-cli start-indexing-server --host 127.0.0.1 --port 8998 --status-port 8990
```

After this, the indexing server will start listening on port 8998 and the status server will be available on port 8990.

The status server can be used to query the current state of the indexing server:  

```bash
$ curl http://127.0.0.1:8990  

{"status":0,"status_updated_at":1727964328269}
```

The status can be one of the following values:  

- `0` - Idle
- `1` - In Progress
- `2` - Failed
- `3` - Succeded

The indexing server can accept only one connection at a time, because the indexing process will consume all available CPU resources.  

The server also accepts `--cert` and `--key` parameters which are SSL certificate and certificate key files.  

Self signed certificate can be generated using the following command:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /tmp/key.pem -out /tmp/cert.pem -subj '/C=US/ST=California/L=San Francisco/O=MyCompany/CN=example.com'

# Then start the server using the generated certificate
lantern-cli start-indexing-server --host 127.0.0.1 --port 8998 --status-port 8990 --cert /tmp/cert.pem --key /tmp/key.pem
```

## Create External Index

```sql
SET lantern.external_index_host='127.0.0.1';
SET lantern.external_index_port=8998;
SET lantern.external_index_secure=false; -- set this to true for SSL servers

CREATE INDEX ON embeddings USING lantern_hnsw(v dist_cos_ops) WITH (m=12, ef_construction=64, external=true);
```

With the `lantern.external_index_*` GUC variables we are configuring the destination of our external indexing server and whether to use TLS connection.

When the `external=true` parameter is be passed in the `CREATE INDEX` statement, the database will connect to the provided external indexing endpoint, stream tuples and receive back the index file.

You can read more about external indexing in the blog post [here](/blog/pgvector-external-indexing).

```sql
-- verify that index is created properly
SET enable_seqscan=false
SET lantern.pgvector_compat=false
-- you should see index scan on query planner
EXPLAIN SELECT * FROM embeddings ORDER BY v <=> ARRAY[1,1,1];
```

Note: When reindexing external indexes make sure that the server is available as it will go with the same routine as regular external index creation

## CLI parameters

Run `bash lantern-cli start-indexing-server --help` to get available CLI parameters

```bash
Start external index server

Usage: lantern-cli start-indexing-server [OPTIONS]

Options:
      --host <HOST>                Host to bind [default: 0.0.0.0]
      --tmp-dir <TMP_DIR>          Temp directory to save intermediate files [default: /tmp]
      --port <PORT>                Port to bind [default: 8998]
      --status-port <STATUS_PORT>  Status Server Port to bind [default: 8999]
      --cert <CERT>                SSL Certificate path
      --key <KEY>                  SSL Certificate key path
  -h, --help                       Print help
```
