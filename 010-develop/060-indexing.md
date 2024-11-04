# Create Index

## Create an index

To create an index you can use the following syntax:

```sql
CREATE INDEX ON [TABLE] USING lantern_hnsw ([column] [operator class])
    WITH (M=[int], ef_construction=[int], ef=[int], dim=[int]);
```

For example, in the query below we create an index on our `books` table, over the `book_embedding` column which has dimension `3`, using the L2 squared distance operator class `dist_l2sq_ops`. We choose the HNSW parameters `M=2`, `ef_construction=4`, and `ef=4`.

```sql
CREATE INDEX
    book_index
ON
    books
USING
    lantern_hnsw (book_embedding dist_l2sq_ops)
WITH (
    M = 2,
    ef_construction = 10,
    ef = 4,
    dim = 3
);
```

In general, lower values of `M` and `ef_construction` improve search speed and lowers index creation times, at the cost of recall. Tuning these parameters will require experimentation for your specific use case.

## Operator classes

The following distance metrics are available for indexes, using the corresponding operator classes.

```table
| Distance metric | Distance operator class     | Supported data types | Distance operator    |
|---------------- | --------------------------- | -------------------- | -------------------- |
|**Euclidean**    | `dist_l2sq_ops`             | `REAL[]`             | `<->`                |
|**Euclidean**    | `dist_vec_l2sq_ops`         | `VECTOR`             | `<->`                |
|**Cosine**       | `dist_cos_ops`              | `REAL[]`             | `<=>`                |
|**Cosine**       | `dist_vec_cos_ops`          | `VECTOR`             | `<=>`                |
|**Hamming**      | `dist_hamming_ops`          | `INTEGER[]`          | `<+>`                |
```

The index is created using a specified distance metric. As such, vector queries should use the corresponding distance operator. For example, an index created with the `dist_l2sq_ops` operator class should be queried using the `<->` operator.

If an operator is used that does not have a corresponding index with the same operator class, the query will not use the index and will run an exact search over all rows.

## Index Parameters

```documentation
m
number
default: 16
```

The number of bi-directional links created for every new element during construction. Reasonable range for `m` is 2-100. Higher `m` works better on datasets with high dimensionality and/or high recall, while low `m` works better for datasets with low dimensionality and/or low recalls.

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

The size of the dynamic list for the nearest neighbors (used during search). Higher `ef` leads to more accurate but slower search. (this parameter is also controlled via seesion based `lantern_hnsw.ef` variable)

Maximum allowed value for `ef` is 400

This parameter is also controlled via session based `lantern_hnsw.ef` variable, which has precedence over this parameter. It can be set by running `SET hnsw.ef=128` before the query.

---

```documentation
dim
number
```

Vector dimensions that will be stored in the table. If not specified it will try to be inferred from the existing data.

Maximum allowed dimensions for now is 2000

---

```documentation
quant_bits
number
default: 32
```

Number of bits to use for scalar quantization.  
Default value is 32 (`f32`) which means to not apply any quantization over vector elements

Allowed values (1, 8, 16, 32)

---

```documentation
external
bool
```

Connect to external indexing server using `lantern.external_index_*` GUC variables and do the indexing process in an external server.

---

```documentation
lantern_hnsw.init_k
string
default: 10
```

Number of items you are expecting from index to return. This is session based variable.

It is important to set this value according the `LIMIT` in your query or the search performance will be decreased. For example if you do a query like this

```sql
SELECT * FROM lantern_demo ORDER BY v <-> ARRAY[1,1,1] LIMIT 100;
```

You should set the `lantern_hnsw.init_k` to 100. So the query will become like this

```sql
SET lantern_hnsw.init_k = 100;
SELECT * FROM lantern_demo ORDER BY v <-> ARRAY[1,1,1] LIMIT 100;
```

Maximum allowed value for `lantern_hnsw.init_k` is 1000

---

```documentation
lantern_hnsw.ef
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
SET lantern_hnsw.ef = 128;
SELECT * FROM lantern_demo ORDER BY v <-> ARRAY[1,1,1];
```

Your query will run with `ef` parameter set to 128 instead of 16.

Maximum allowed value for `lantern_hnsw.ef` is 400

```documentation
lantern.external_index_host
string
default: 127.0.0.1
```

Host of the server where [Lantern External Index Server](https://github.com/lanterndata/lantern_extras?tab=readme-ov-file#lantern-index-server) is running

---

```documentation
lantern.external_index_port
number
default: 8998
```

Port of the external indexing server

---

```documentation
lantern.external_index_secure
bool
default: true
```

If set to true it will try to initialize TLS connection when connecting to external indexing server

---
