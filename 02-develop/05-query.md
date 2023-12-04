# Query Embeddings

## Select nearest row with filters using index

Assuming that you have created an index, query the database to retrieve records based on the proximity of embedding values using the index.

```sql
SELECT
    title,
    author
FROM
    books
WHERE
    published_at < 2010
ORDER BY
    text_embedding <-> '{0,0,0}'
LIMIT 1;
```

## Select nearest rows without using index

If you choose not to use an index, you can still fetch records based on embedding proximity.

```sql
SELECT
    title,
    author
FROM
    books
ORDER BY
    l2sq_dist(text_embedding, '{0,0,0}')
LIMIT 2;
```
