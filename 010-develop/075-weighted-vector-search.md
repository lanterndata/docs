# Weighted Vector Search

## Motivation

Consider a table `products` with columns `name` and `description`. You may want to prioritize products where the search query matches the `name` column over the `description` column. In this case, you can perform a weighted vector search, with a greater weight for the `name` column.

## Overview

Weighted vector search enables searching over multiple vector columns in a single query. It calculates a joint distance metric based on the weighted distance between the vectors in the table and the provided vectors. It allows simultaneously searching over up to 3 columns and ordering rows according to weighted vector distance of the query to each of the three columns.

The advantage of the function is that it allows using the vector index on each of the columns, while regular weighting via an SQL query would not allow that.

## API

The API of the function is below:

```sql
weighted_vector_search(
  relation_type, -- Type of the specific relation containing all the vectors
  max_dist double precision, -- Maximum weighted distance to keep
  w1 double precision, -- Weight of the first vector
  col1 text, -- Column name of the first vector
  vec1 vector, -- Query vector constant for the first column vector

  w2 double precision = 0, ...
  col2 text = NULL,
  vec2 vector = NULL,

  w3 double precision = 0,...
  col3 text = NULL,
  vec3 vector = NULL)
  RETURNS TABLE of relation_type containing only relevant rows
```

The function filters all results that have total weighted distance smaller than `max_dist`.

## Example Usage

Let's start by creating an example table with text and vector columns to demonstrate the usage of the `weighted_vector_search` function.

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    description TEXT,
    category TEXT,
    embedding VECTOR(1536)
);

INSERT INTO products (name, description, category, embedding)
VALUES
    ('Wrench Set', 'A set of wrenches for tightening and loosening nuts and bolts.', 'Tools', /* vector */),
    ('Shower Curtain', 'A waterproof curtain for the shower or bathtub.', 'Bathroom', /* vector */),
    ('Lawn Mower', 'A gas-powered mower for cutting grass.', 'Outdoor', /* vector */),
    ('Screwdriver Set', 'A set of screwdrivers with different head types.', 'Tools', /* vector */);
```

In this example, we have a products table with columns for name, description, category, and an embedding column of type `VECTOR(1536)` to store vector representations of the product data.

To perform a weighted vector search on this table, we can use the `weighted_vector_search` function. Let's say we want to find products related to cars, with a focus on the product description and category.

```sql
SELECT name, description, category
FROM weighted_vector_search(
    CAST(NULL AS products),
    max_dist => 5.0,
    w1 => 2.0, col1 => 'description', vec1 => /* vector describing car environment s*/,
    w2 => 1.0, col2 => 'category', vec2 => /* vector describing car environments */
);
```

In this example, we search for products with a maximum weighted distance of 5.0 from the query vector. We're giving a higher weight (2.0) to the description column and a lower weight (1.0) to the category column. The vec1 and vec2 parameters should be replaced with the actual vector representations of the word "plumbing" for the respective columns.

This query will return the name, description, and category of products that are most relevant to the query vector, with a higher emphasis on the product description.

We can control which column is more important for our query by adjusting the weights.
