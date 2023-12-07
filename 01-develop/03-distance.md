# Calculate Distance Between Embeddings

Currently we support the following distance functions:

```table
| Name          | Distance function| Supported data types         |
| ------------- | -----------------| ---------------------------- |
| **Euclidean** | `l2sq_dist`      | `REAL[]`, `VECTOR`           |
| **Cosine**    | `cos_dist`       | `REAL[]`, `VECTOR`           |
| **Hamming**   | `hamming_dist`   | `INTEGER[]`                  |
```

You can use the provided distance functions to calculate the distance between vectors

```sql
SELECT l2sq_dist(ARRAY[0,0.1,0], ARRAY[0.5,0.0,0.2]); -- Euclidean
SELECT cos_dist(ARRAY[0,1,0], ARRAY[1,1,1]);          -- Cosine
SELECT hamming_dist(ARRAY[0,1,0], ARRAY[1,1,1]);      -- Hamming
```

You can also use the provided distance functions to fetch records based on embedding distance **without** an index

```sql
SELECT title FROM books ORDER BY l2sq_dist(text_embedding, '{0,0,0}') LIMIT 2;
```
