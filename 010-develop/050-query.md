# Query Embeddings

This section provides examples of how to query a table for the rows with the nearest embeddings to a given vector or row. The examples use the `books` table with the `book_embedding` column, which contains the embeddings of books.

## Get the nearest rows to a vector

Retrieve the two closest rows by vector distance:

```sql
SELECT * FROM books ORDER BY book_embedding <-> '{0,0,0}' LIMIT 2;
```

**Note:** Using an index significant speeds up the query. To use an index, create an index for the vector column, and the query should use `ORDER BY` with the corresponding distance operator and `LIMIT`.

## Get the nearest rows to a row

Find the two nearest rows to a specific row, excluding the row itself:

```sql
SELECT * FROM books WHERE id != 1 ORDER BY book_embedding <-> (SELECT book_embedding FROM books WHERE id = 1) LIMIT 2;
```

## Get rows within a certain distance

Search for rows within a specific distance to a given vector:

```sql
SELECT * FROM books WHERE embedding <-> '[3,1,2]' < 5;
```

## Get nearest neighbors to a vector with filters

Retrieve the nearest book with a publication date before 2010:

```sql
SELECT * FROM books WHERE published_at < 2010 ORDER BY book_embedding <-> '{0,0,0}' LIMIT 1;
```

## Select nearest row using text embeddings

Find books similar to a given text description:

```sql
SELECT * FROM books
ORDER BY book_openai_embedding <-> openai_embedding('openai/text-embedding-ada-002', 'A book about space for children that is fun and educational')
LIMIT 2
```

## Select nearest rows using exact search

To use exact search to select the nearest rows, use the distance functions directly. For example, the following query will run an exact search over all rows using the L2 Distance function.

```sql
SELECT * FROM books ORDER BY l2sq_dist(book_embedding, '{0,0,0}') LIMIT 2;
```

You can also use the distance operator, as seen below, and get an exact search if there is no index created. Note that if you have an index created, then the query will use the index and return approximate nearest rows.

```sql
SELECT * FROM books ORDER BY book_embedding <-> '{0,0,0}' LIMIT 2;
```
