# Quantization

## Overview

High-dimensional vectors are typically stored as 32 bit floating point numbers. Quantization is the process of reducing the precision of vectors to save memory and improve computational efficiency.

Lantern supports three forms of quantization: binary, scalar, and product quantization. Each technique offers different trade-offs between recall, memory efficiency, and computational efficiency. The right technique depends on the specific use case and the distribution of the vectors.

## Reranking

Quantization techniques can sometimes reduce the recall of the search results. Reranking is a common approach to compensate for this reduction in recall.

In the context of k-nearest neighbors (KNN) search, the goal is to find the top k nearest neighbors to a query vector. With reranking, the quantized search is used to find a larger set of candidate results, and then the original vectors are used to re-rank the candidates and select the top k nearest neighbors.

The idea behind reranking is that since quantized search is faster, it can be used to quickly identify a larger set of potential candidates. Then, the original vectors are used to refine the results and find the top k nearest neighbors with higher recall. This technique can be useful even when the index does not use quantization, as the index search is an approximate as opposed to exact nearest neighbors search.

Reranking allows for a balance between search speed and recall, leveraging the efficiency of quantized search while maintaining the accuracy of the final results.

## Scalar Quantization

Scalar quantization is the simplest form of quantization. It involves reducing the precision of the vector, converting from an array of higher precision numbers to an array of lower precision numbers. Scalar quantization generally strikes a balance between memory efficiency and precision. Lantern supports scalar quantization to `float32`, `float16`, and `float8`.

To create an index with scalar quantization on the table `books` with `v` as the vector column:

```sql
CREATE INDEX ON books USING lantern_hnsw (v) WITH (quant_bits=16);
```

`quant_bits` specifies the number of bits to quantize the vectors to. The values 8, 16, and 32 are supported. The default value is 32, which means no quantization.

To query the index, you can use the usual query operator. There is no need to convert the query vector to the scalar-quantized format, as this is handled internally by Lantern.

```sql
SELECT * FROM books ORDER BY v <-> '[1,2,3]' LIMIT 5;
```

## Binary Quantization

In binary quantization, vectors are represented using binary values (0s and 1s). Generally, values less than or equal to 0 are mapped to 0, and values greater than 0 are mapped to 1.

Binary quantization reduces the memory footprint of 32-bit vectors by a factor of 32. In addition, computing distances between binary vectors is much faster since it only involves bitwise operations.

It is important to measure recall when using binary quantization, since the effectiveness of binary quantization depends on the distribution of the original vectors. If the vectors are not well-separated, binary quantization can lead to a significant loss of recall.

To create an index with binary quantization on the table `books` with `v` as the vector column:

```sql
CREATE INDEX ON books USING lantern_hnsw (v) WITH (quant_bits=1);
```

To query the index, you can use the usual query operator. There is no need to convert the query vector to the binary-quantized format, as this is handled internally by Lantern.

```sql
SELECT * FROM books ORDER BY v <-> '[1,2,3]' LIMIT 5;
```

## Product Quantization

Product quantization is a more advanced technique that involves breaking down high-dimensional vectors into smaller sub-vectors and quantizing each sub-vector independently. Before quantization can be performed, a training stage is required.

During the training stage, a codebook is constructed for each sub-vector. The codebook contains a set of representative vectors generated using clustering algorithms like k-means over a large set of input vectors. The vectors in the index are then quantized by finding the closest representative vector in each sub-vector codebook, and the index stores the index of the representative vector.

During the search process, the query vector is quantized in the same way, and the search is performed over the quantized vectors. Product quantization allows for significant memory reduction while maintaining a good level of precision.

Product quantization can achieve greater memory savings than binary and scalar quantization, but less computation savings.

### Basic Usage

The first step of product quantization is the `quantize_table` function. This function creates a codebook for each subvector and quantizes the vectors in the table into a column with the same name as the original column with `_pq` appended.

For example, the following command quantizes the table `books` with `v` as the vector column, and generates a column `v_pq` with 256 clusters, 16 subvectors, and the Euclidean distance metric:

---

```sql
SELECT quantize_table('books', 'v', 256, 16, 'l2sq');
```

To create the index, set `pq=True` in the `WITH` clause of the index creation statement:

```sql
CREATE INDEX hnsw_l2_index ON small_world_pq USING lantern_hnsw(v) WITH (pq=True);
```

Finally, to query the index, you can use the usual query operator. There is no need to convert the query vector to the product-quantized format, as this is handled internally by Lantern.

```sql
SELECT * FROM books ORDER BY v <-> '[1,2,3]' LIMIT 5;
```

The full list of available functions for product quantization follows.

## Product Quantization API

The following functions are available in SQL for product quantization:

### quantize_table

This function creates a codebook for each subvector and quantizes the vectors in the table into a column with the same name as the original column with `_pq` appended.

The API details are as follows:

```sql
quantize_table(p_tbl regclass, p_col NAME, cluster_cnt INT, subvector_count INT, distance_metric TEXT, dataset_size_limit INT DEFAULT 0);
```

---

```documentation
p_tbl
regclass
```

The table to quantize.

---

```documentation
p_col
NAME
```

The column to quantize.

---

```documentation
cluster_cnt
INT
```

The number of clusters to use for each subvector.

---

```documentation
subvector_count
INT
```

The number of subvectors to split the vector into.

---

```documentation
distance_metric
TEXT
```

The distance metric to use for quantization. The supported distance metrics are `l2sq`, `cos`, and `hamming`.

---

```documentation
dataset_size_limit
INT
```

k-means clustering is a memory-intensive operation. If the dataset is too large, the clustering operation may fail due to memory constraints. To avoid this, you can set a limit on the number of rows to use for clustering. If the dataset size exceeds this limit, the function will sample the dataset to the specified limit.

### create_pq_codebook

Unlike `quantize_table`, `create_pq_codebook` only creates the codebook without creating the column It has the same arguments as `quantize_table`.

### quantize_vector

Unlike `quantize_table`, `quantize_vector` creates a new column with the quantized vectors.

If a PQ codebook already exists, use the function `quantize_vector` to create a new column with the quantized vectors.

```sql
quantize_vector(v REAL[], codebook regclass, distance_metric TEXT)
```

---

```documentation
v
REAL[]
```

The vector to quantize.

---

```documentation
codebook
regclass
```

The codebook to use for quantization.

---

```documentation
distance_metric
TEXT
```

The distance metric to use for quantization. The supported distance metrics are `euclidean`, `cosine`, and `hamming`.

### dequantize_vector

If a codebook exists, you can dequantize a vector using the `dequantize_vector` function, i.e., convert the quantized vector back to the lower precision vector.

```sql
dequantize_vector(v pqvec, codebook regclass)
```

--- d

```documentation
v
pqvec
```

The quantized vector.

---

```documentation
codebook
regclass
```

The codebook to use for dequantization.

### drop_quantization

Given a table and a column that was quantized, this function drops the codebook table and the quantized column if either exists.

```sql
drop_quantization(p_tbl regclass, p_col NAME)
```

---

```documentation
p_tbl
regclass
```

The table to drop quantization from.

---

```documentation
p_col
NAME
```

The column to drop quantization from.

### Lantern CLI

Product quantization is also available via the [Lantern CLI](/docs/lantern-cli/pq). In particular, the Lantern CLI supports quantization of large datasets using GCP batch jobs to parallelize the workload over hundreds of VMs.
