# Create Index

## Create an index

To create an index you can use the following syntax:

```sql
CREATE INDEX ON [TABLE] USING hnsw ([column] [operator class])
    WITH (M=[int], ef_construction=[int], ef=[int], dim=[int]);
```

For example, in the query below we create an index on our `books` table, over the `book_embedding` column which has dimension `3`, using the L2 squared distance operator class `dist_l2sq_ops`. We choose the HNSW parameters `M=2`, `ef_construction=4`, and `ef=4`.

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

## Operator classes

The following distance metrics are available for indexes, using the corresponding operator classes.

```table
| Distance metric | Distance operator class     | Supported data types |
|---------------- | --------------------------- | -------------------- |
|**Euclidean**    | `dist_l2sq_ops`             | `REAL[]`, `VECTOR`   |
|**Cosine**       | `dist_cos_ops`              | `REAL[]`, `VECTOR`   |
|**Hamming**      | `dist_hamming_ops`          | `INTEGER[]`          |
```

## Index Parameters

```documentation
m
number
default: 16
```

The number of bi-directional links created for every new element during construction. Reasonable range for `m` is 2-100. Higher `m` work better on datasets with high intrinsic dimensionality and/or high recall, while low `m` work better for datasets with low intrinsic dimensionality and/or low recalls.

Maximum allowed value for `m` is 128

---

```documentation
ef_construction
number
default: 128
```

The size of the dynamic list for the nearest neighbors (used during the index construction). Higher `ef_construction` leads to better index quality, but reduces indexing speed.

Maximum allowed value for `ef_construction` is 400

---

```documentation
ef
number
default: 64
```

The size of the dynamic list for the nearest neighbors (used during search). Higher `ef` leads to more accurate but slower search. (this parameter is also controlled via seesion based `hnsw.ef` variable)

Maximum allowed value for `ef` is 400

This parameter is also controlled via session based `hnsw.ef` variable, which has precedence over this parameter
`sql SET hnsw.ef=128`

---

```documentation
dim
number
```

Vector dimensions that will be stored in the table. If not specified it will try to be inferred from the existing data.

Maximum allowed dimensions for now is 2000

---

```documentation
_experimental_index_path
string
```

Path to index file created via [Lantern CLI](https://github.com/lanterndata/lantern_extras#lantern-index-builder)

In order for the index file to function properly, it should be present in the server where the database is running and accessible to the user running the database.

---

```documentation
hnsw.init_k
string
default: 10
```

Number of items you are expecting from index to return. This is session based variable.

It is important to set this value according the `LIMIT` in your query or the search performance will be decreased. For example if you do a query like this

```sql
SELECT * FROM lantern_demo WHERE v <-> ARRAY[1,1,1] LIMIT 100;
```

You should set the `hnsw.init_k` to 100. So the query will become like this

```sql
SET hnsw.init_k = 100;
SELECT * FROM lantern_demo WHERE v <-> ARRAY[1,1,1] LIMIT 100;
```

Maximum allowed value for `hnsw.init_k` is 1000

---

```documentation
hnsw.ef
string
default: 64
```

This is session based variable which will controll the `ef` parameter of your index.

This variable has priority over the `ef` option specified in the index creation.

If you create an index like this

```sql
CREATE INDEX ON lantern_demo USING lantern_hnsw(v) WITH (m=4, ef_construction=8, ef=16);
```

And do a query like this

```sql
SET hnsw.ef = 128;
SELECT * FROM lantern_demo WHERE v <-> ARRAY[1,1,1];
```

Your query will run with `ef` parameter set to 128 instead of 16.

Maximum allowed value for `hnsw.ef` is 400
