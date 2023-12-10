# Calculate Distance Between Embeddings

Currently we support the following distance functions and distance operators:

```table
| Distance metric | Distance function| Distance operator | Supported data types         |
| --------------- | -----------------| ------------------|----------------------------- |
| **Euclidean**   | `l2sq_dist`      | `<->`             | `REAL[]`, `VECTOR`           |
| **Cosine**      | `cos_dist`       | `<=>`             | `REAL[]`, `VECTOR`           |
| **Hamming**     | `hamming_dist`   | `<+>`             | `INTEGER[]`                  |
```

## Distance functions

You can use the provided distance functions to calculate the distance between vectors

```sql
SELECT l2sq_dist(ARRAY[0,0.1,0], ARRAY[0.5,0.0,0.2]); -- Euclidean
SELECT cos_dist(ARRAY[0,1,0], ARRAY[1,1,1]);          -- Cosine
SELECT hamming_dist(ARRAY[0,1,0], ARRAY[1,1,1]);      -- Hamming
```

You can also use the provided distance functions to fetch records based on embedding distance without an index. This will run an exact search over all rows.

```sql
SELECT title FROM books ORDER BY l2sq_dist(book_embedding, '{0,0,0}') LIMIT 2;
```

## Distance operators

You can use the provided distance operators to calculate the distance between vectors

```sql
SELECT ARRAY[0,0.1,0] <-> ARRAY[0.5,0.0,0.2]; -- Euclidean
SELECT ARRAY[0,1,0] <=> ARRAY[1,1,1];         -- Cosine
SELECT ARRAY[0,1,0] <+> ARRAY[1,1,1];         -- Hamming
```

You can also use the provided distance operators to fetch records based on embedding distance. If you are using an index, this query will use the index. Otherwise, it will run an exact search over all rows.

```sql
SELECT title FROM books ORDER BY book_embedding <-> '{0,0,0}' LIMIT 2;
```
