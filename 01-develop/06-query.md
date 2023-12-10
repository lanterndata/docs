# Query Embeddings

## Select nearest row with filters using index

Assuming that you have created an index for the `book_embedding` column using the L2 distance metric, you can query the database to retrieve records based on the proximity of embedding values using the index.

```sql
SELECT
    title,
    author
FROM
    books
WHERE
    published_at < 2010
ORDER BY
    book_embedding <-> '{0,0,0}'
LIMIT 1;
```

Note that if you created an index using a different distance metric, this query will not use the index. It will instead run exact search over all rows.

## Select nearest rows using exact search

If you want to use exact search to select the nearest rows, you can use the distance functions directly.

```sql
SELECT
    title,
    author
FROM
    books
ORDER BY
    l2sq_dist(book_embedding, '{0,0,0}')
LIMIT 2;
```

You can also use the distance operator, as seen below. Note that if you have an index created, then the query will use the index and return approximate nearest rows.

```sql
SELECT
    title,
    author
FROM
    books
ORDER BY
    book_embedding <-> '{0,0,0}'
LIMIT 2;
```
