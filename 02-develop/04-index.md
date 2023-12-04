# Create Embedding Index

## Create an index

Boost the efficiency of your queries by indexing the embedding column.

```sql
CREATE INDEX
    book_index
ON
    books
USING
    hnsw (text_embedding dist_l2sq_ops)
WITH (
    M = 2,
    ef_construction = 10,
    ef = 4,
    dims = 3
);
```

Note: `dist_l2sq_ops` is a distance function. It can be substituted with other appropriate distance functions depending on your requirements.

## Operator classes

```table
| Name          | Operator class    | Supported data types         |
| ------------- | ------------------| ---------------------------- |
| **Euclidean** | `dist_l2sq_ops`   | `REAL[]`, `VECTOR`           |
| **Cosine**    | `dist_cos_ops`    | `REAL[]`, `VECTOR`           |
| **Hamming**   | `dist_hamming_ops`| `INTEGER[]`                  |
```

To create an index you can use the following syntax:

```sql
CREATE INDEX ON [TABLE] USING hnsw ([column] [operator class]) WITH (m=[int], ef_construction=[int], ef=[int]);
```

## Index Parameters
