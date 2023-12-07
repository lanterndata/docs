# Store Embeddings

Note: All operations here are designed for embeddings that are of type `INTEGER[]`, `REAL[]`, or `pgvector`'s `VECTOR` type. In the subsequent examples, we use `REAL[]`.

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
INSERT INTO books (id, title, author, published_at, text_embedding, reviews) VALUES
    (1, 'The Lightning Thief', 'Rick Riordan', 1999, '{0,0,1}', NULL),
    (2, 'White Fang', 'Jack London', 2000, '{1,0,1}', 'Good');
```

### Upsert embedding

Insert a new row or update the embedding of an existing row.

```sql
INSERT INTO books (id, title, text_embedding) VALUES
    (4, 'The Lord of the Rings', '{1,1,0}')
ON CONFLICT (id)
DO UPDATE SET text_embedding = EXCLUDED.text_embedding;
```

### Update embeddings

```sql
UPDATE books SET text_embedding = '{0,0,0}' WHERE id = 1;
UPDATE books SET text_embedding = ARRAY[0,0,0] WHERE id = 2;
```

### Delete embeddings

```sql
DELETE FROM books WHERE id = 1;
```
