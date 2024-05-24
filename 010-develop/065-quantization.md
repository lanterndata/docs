# Quantization

## Overview

High-dimensional vectors are typically stored as 32 bit floating point numbers. Quantization is the process of reducing the precision of vectors to save memory and improve computational efficiency.

Lantern supports three forms of quantization: binary, scalar, and product quantization. Each technique offers different trade-offs between recall, memory efficiency, and computational efficiency. The right technique depends on the specific use case and the distribution of the vectors.

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

At the moment, product quantization is only available via the [Lantern CLI](/docs/lantern-cli/pq).

## Reranking

Quantization techniques can sometimes reduce the recall of the search results. Reranking is a common approach to compensate for this reduction in recall.

In the context of k-nearest neighbors (KNN) search, the goal is to find the top k nearest neighbors to a query vector. With reranking, the quantized search is used to find a larger set of candidate results, and then the original vectors are used to re-rank the candidates and select the top k nearest neighbors.

The idea behind reranking is that since quantized search is faster, it can be used to quickly identify a larger set of potential candidates. Then, the original vectors are used to refine the results and find the top k nearest neighbors with higher recall. This technique can be useful even when the index does not use quantization, as the index search is an approximate as opposed to exact nearest neighbors search.

Reranking allows for a balance between search speed and recall, leveraging the efficiency of quantized search while maintaining the accuracy of the final results.
