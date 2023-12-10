# Troubleshooting

## Check if you are using the index

You can use `EXPLAIN` to validate if you are using the index.

For example, this query is using a sequential scan. This means that an index is not being used to make search faster, and the query performs an exact search over all rows.

```sql
EXPLAIN (COSTS FALSE) SELECT 1 FROM books WHERE v <-> '{0,0,0}';
            QUERY PLAN
-----------------------------------
 Limit
   ->  Sort
         Sort Key: (v <-> '{1,0,0}'::real[])
         ->  Seq Scan on books
```

In contrast, this query uses an index.

```sql
EXPLAIN (COSTS FALSE) SELECT 1 FROM books ORDER BY v <-> '{1,0,0}' LIMIT 1;
                       QUERY PLAN
---------------------------------------------------------
 Limit
   ->  Index Scan using books_book_embedding_idx on books
         Order By: (v <-> '{1,0,0}'::real[])
```

## Why is my query not using an index?

### Postgres estimated that a table scan would be faster

When determining how to perform a query, Postgres makes an estimate of various execution plans. Especially for small tables, it is possible that Postgres estimated that a sequential scan would be more efficient than an index scan. This estimate is not 100% accurate, and in general we expect index scans to perform faster than sequential scans, even for smaller tables.

To avoid this, you can instruct Postgres to avoid sequential scans whenever possible by setting the `enable_seqscan` parameter to `FALSE`. See below for an example, and see notes on [Postgres configuration](/docs/develop/postgres) for more details on how to set this parameter.

```sql
SET enable_seqscan=FALSE;
```

You can validate that this uses an index

```sql
EXPLAIN (COSTS FALSE) SELECT 1 FROM books order by v <-> '{1,0,0}' LIMIT 1;
                       QUERY PLAN
---------------------------------------------------------
 Limit
   ->  Index Scan using books_book_embedding_idx on books
         Order By: (v <-> '{1,0,0}'::real[])
```

### Your query was written incorrectly

If after disabling sequential scans, you still find that your query does not use the index, it may be the case that the operator was used incorrectly.

Here are examples of SQL queries that will use the index

```sql
-- Directly orders by the result of the distance calculation
SELECT v <-> ARRAY[0,0,0] FROM books ORDER BY v <-> ARRAY[0,0,0];

-- Orders by the first column (result of the distance calculation)
SELECT v <-> ARRAY[0,0,0] FROM books ORDER BY 1;

-- Orders by the first column (result of the distance calculation)
SELECT v <-> ARRAY[0,0,0] AS distance FROM books ORDER BY 1;
```

Here are examples of SQL queries that will not use the index.

```sql
-- Orders by the alias instead of the actual distance calculation
SELECT v <-> ARRAY[0,0,0] AS distance FROM books ORDER BY distance;

-- Using an alias in ORDER BY with a mathematical expression
SELECT v <-> ARRAY[0,0,0] AS distance FROM books ORDER BY distance * 2;

-- Using an alias in ORDER BY with a function
SELECT v <-> ARRAY[0,0,0] AS distance FROM books ORDER BY sqrt(distance);

-- Using a complex expression in ORDER BY
SELECT v <-> ARRAY[0,0,0] FROM books ORDER BY v <-> ARRAY[1,1,1] + 2;

-- Using a non-trivial expression involving multiple columns
SELECT (v <-> ARRAY[0,0,0]) * (x <-> ARRAY[0,0,0]) FROM books ORDER BY 1;

-- Using a CASE statement in ORDER BY
SELECT v <-> ARRAY[0,0,0] AS distance FROM books ORDER BY CASE WHEN distance > 5 THEN 1 ELSE 0 END;

-- Using a subquery in ORDER BY
SELECT v <-> ARRAY[0,0,0] AS distance FROM books ORDER BY (SELECT MAX(distance) FROM books);
```

In the second group of examples, complexity is introduced by mathematical expressions, functions, and aliases that hinder the query planner's ability to recognize the correlation between the index and the distance calculation.

If you are having this issue, try simplifying your query. Feel free to contact us at support@lantern.dev for additional help.
