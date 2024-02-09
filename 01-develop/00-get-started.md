# Get Started

## Lantern Cloud

The easiest way to get started with all of our tools is with [Lantern Cloud](/).

We're currently in closed beta. Reach out at support@lantern.dev for access. We'd love to learn more about your use case and help out.

In Lantern cloud, you can create create a Lantern database with just a few clicks. Then, once you load your data into Lantern,
you can [generate embeddings](/docs/develop/generate) with a single click from dozens of provided open source or proprietary embedding models.
You can then create a vector index from your dashboard or run an index-tuning experiment to choose the best parameters for index creation.

## Self-Host

Alternatively, you can also use our tools locally or self-host them. There are three tools that are provided out-of-the-box with Lantern Cloud.

- [Lantern](/docs/lantern-db/install), our core Postgres extension, provides vector search in Postgres.
- [Lantern Extras](/docs/lantern-extras/install), which further extends Postgres to support embedding generation.
- [Lantern CLI](/docs/lantern-cli/install) provides routines for generating embeddings and indexes.

You can install the tools individually by following the instructions linked.

## Overview

Here is a non-comprehensive overview of what you can do with Lantern. The examples below use SQL, but we also provide client libraries for [Python](/docs/languages/python) and [JavaScript](/docs/languages/javascript).

Create a table with an embedding column

```sql
CREATE TABLE books (id SERIAL PRIMARY KEY, book_embedding REAL[3]);
```

Insert embeddings

```sql
INSERT INTO books (book_embedding) VALUES ('{0,1,0}'), ('{3,2,4}');
```

Calculate distance and select nearest rows without using an index

```sql
SELECT book_embedding <-> '{0,0,0}' FROM books
    ORDER BY book_embedding <-> '{0,0,0}' LIMIT 1;
```

Create an index

```sql
CREATE INDEX book_index ON books USING hnsw(book_embedding dist_l2sq_ops)
    WITH (M=2, ef_construction=10, ef=4, dim=3);
```

Select nearest rows using the index

```sql
SELECT book_embedding <-> '{0,0,0}' FROM books
    ORDER BY book_embedding <-> '{0,0,0}' LIMIT 1;
```

Generate embeddings

```sql
SELECT text_embedding('BAAI/bge-base-en', 'My text input');
```
