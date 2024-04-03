# Store Embeddings

Embeddings are stored in a table as a column of type `INTEGER[]`, `REAL[]`, or `pgvector`'s `VECTOR` type. This guide provides examples of how to create a table with an embedding column, insert embeddings into the table, and update or delete embeddings. Note that the examples use SQL, but we also provide examples for [Python](/docs/languages/python), [JavaScript](/docs/languages/javascript), and [Rust](/docs/languages/rust).

## Table Operations

### Create table with embedding column

Define a table structure that contains an embedding column.

```sql
CREATE TABLE books (
    id INTEGER PRIMARY KEY,
    title TEXT NOT NULL,
    author TEXT,
    book_summary TEXT,
    published_at INTEGER,
    text_embedding REAL[3],
    reviews TEXT
);
```

### Add embedding column to table

Add an additional embedding column to your table.

```sql
ALTER TABLE books ADD COLUMN reviews_embedding REAL[];
```

## Storing Rows

### Insert embeddings into the table

Populate your table with embedding data.

```sql
INSERT INTO books (id, title, author, published_at, book_embedding, reviews) VALUES
    (1, 'The Lightning Thief', 'Rick Riordan', 1999, '{0,0,1}', NULL),
    (2, 'White Fang', 'Jack London', 2000, '{1,0,1}', 'Good');
```

### Upsert embedding

Insert a new row or update the embedding of an existing row.

```sql
INSERT INTO books (id, title, book_embedding) VALUES
    (4, 'The Lord of the Rings', '{1,1,0}')
ON CONFLICT (id)
DO UPDATE SET book_embedding = EXCLUDED.book_embedding;
```

### Update embeddings

```sql
UPDATE books SET book_embedding = '{0,0,0}' WHERE id = 1;
UPDATE books SET book_embedding = ARRAY[0,0,0] WHERE id = 2;
```

### Delete embeddings

```sql
DELETE FROM books WHERE id = 1;
```
