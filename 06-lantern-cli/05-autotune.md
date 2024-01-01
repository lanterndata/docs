# Autotune Index

With the Lantern CLI's `autotune-index` routine, you can autotune an HNSW and find the index parameters which can give you the best recall and latency.

## Prerequisites

- [Lantern CLI](/docs/lantern-cli/install)
- Postgres database with the [Lantern extension](/docs/lantern-db/install) installed

## Run Index Autotune

```bash
lantern-cli autotune-index --uri 'postgresql://[username]:[password]@localhost:5432/[db]' --table "sift1m" --column "v" --metric-kind l2sq --pk id --recall 99 -k 30 --create-index
```

After this you should see output like this:

```bash
[*] [Lantern Index Autotune] ========== Results for job 05a53289-7a80-4843-9ef7-bee951dbc13c ==========
[*] [Lantern Index Autotune] result(recall=96%, latency=7ms, indexing_duration=2s) index_params(m=6, ef=64, ef_construction=32)
[*] [Lantern Index Autotune] result(recall=98.5%, latency=7ms, indexing_duration=3s) index_params(m=8, ef=64, ef_construction=40)
[*] [Lantern Index Autotune] result(recall=98.9%, latency=7ms, indexing_duration=3s) index_params(m=12, ef=64, ef_construction=48)
[*] [Lantern Index Autotune] result(recall=99.2%, latency=9ms, indexing_duration=3s) index_params(m=16, ef=76, ef_construction=60)
[*] [Lantern Index Autotune] result(recall=99.8%, latency=11ms, indexing_duration=3s) index_params(m=32, ef=96, ef_construction=96)
[*] [Lantern Index Autotune] result(recall=99.9%, latency=14ms, indexing_duration=4s) index_params(m=48, ef=128, ef_construction=128)
```

And index will be created using `create-index` CLI function with the best recall and latency if `--create-index` is passed.

## CLI parameters

Run `bash lantern-cli autotune-index --help` to get available CLI parameters

```bash
Autotune index

Usage: lantern-cli autotune-index [OPTIONS] --uri <URI> --table <TABLE> --column <COLUMN> --pk <PK>

Options:
  -u, --uri <URI>
          Fully associated database connection string including db name
  -s, --schema <SCHEMA>
          Schema name [default: public]
  -t, --table <TABLE>
          Table name
  -c, --column <COLUMN>
          Column name
      --pk <PK>
          Primary key name
      --recall <RECALL>
          Target recall [default: 98]
      --k <K>
          K limit of elements for query [default: 10]
      --test-data-size <TEST_DATA_SIZE>
          Test data size [default: 10000]
      --metric-kind <METRIC_KIND>
          Distance algorithm [default: l2sq] [possible values: l2sq, cos, hamming]
      --create-index
          Create index with the best result
      --export
          Export results to table
      --job-id <JOB_ID>
          Job ID to use when exporting results, if not provided UUID will be generated
      --export-db-uri <EXPORT_DB_URI>
          Database URL for exporting results, if not specified the --uri will be used
      --export-schema-name <EXPORT_SCHEMA_NAME>
          Schame name in which the export table will be created [default: public]
      --export-table-name <EXPORT_TABLE_NAME>
          Table name to export results, table will be created if not exists [default: lantern_autotune_results]
      --model-name <MODEL_NAME>
          Model name to save in results
  -h, --help
          Print help
```
