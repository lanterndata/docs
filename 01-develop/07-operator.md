# Single operator

Lantern adds experimental support for a single operator `<?>`. It has the following benefits

- One operator for all index queries
- Automatically detect when the operator is being used without an index

You can create an index using the steps documented [here](/docs/develop/index). Then, at query time, you can just use the operator `<?>` to perform nearest neighbor search queries. Lantern will automatically infer the distance metric to use.

For example, you can create the following index.

```sql
CREATE INDEX
    book_index
ON
    books
USING
    hnsw (book_embedding dist_l2sq_ops)
WITH (
    M = 2,
    ef_construction = 10,
    ef = 4,
    dim = 3
);
```

Then, you can query the index like so:

```sql
SELECT title FROM books ORDER BY ARRAY[0,0,1] <?> book_embedding;
```

Note that the operator can only be used in an `ORDER BY`. For example, the following queries will **not** work

```sql
SELECT book_embedding <?> ARRAY[0,0,1] FROM books;
SELECT book_embedding <?> ARRAY[0,0,1] FROM books <?> book_embedding;
```
