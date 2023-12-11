# Python

In the subsequent examples, assume the following variables are defined

```python
embedding = [1, 2, 3]
query = "My text input"
```

## psycopg2

Insert a vector

```python
cur.execute("INSERT INTO books (book_embedding) VALUES (%s)", (embedding,))
```

Find nearest rows

```python
cur.execute(
    f"SELECT * FROM books ORDER BY vector <-> %s LIMIT 5",
    (embedding,)
)
cur.execute(
    f"SELECT * FROM books ORDER BY vector <-> text_embedding('BAAI/bge-small-en', %s) LIMIT 5",
    (query,)
)
```
