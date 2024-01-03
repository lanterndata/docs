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
    f"SELECT * FROM books ORDER BY book_embedding <-> %s LIMIT 5",
    (embedding,)
)
cur.execute(
    f"SELECT * FROM books ORDER BY book_embedding <-> text_embedding('BAAI/bge-small-en', %s) LIMIT 5",
    (query,)
)
```

## psycopg3

Enable the extension

```python
conn.execute('CREATE EXTENSION IF NOT EXISTS lantern')
```

Create a table

```python
conn.execute('CREATE TABLE items (id bigserial PRIMARY KEY, embedding REAL[3])')
```

Insert a vector

```python
embedding = [1, 2, 3]
conn.execute('INSERT INTO items (embedding) VALUES (%s)', (embedding,))
```

Get the nearest neighbors to a vector

```python
conn.execute('SELECT * FROM items ORDER BY embedding <-> %s LIMIT 5', (embedding,)).fetchall()
```

## asyncpg

Insert a vector

```python
await conn.execute("INSERT INTO books (book_embedding) VALUES ($1)", embedding)
```

Find nearest rows

```python
await conn.fetch(
    f"SELECT * FROM books ORDER BY book_embedding <-> $1 LIMIT 5",
    embedding
)
await conn.fetch(
    f"SELECT * FROM books ORDER BY book_embedding <-> text_embedding('BAAI/bge-small-en', $1) LIMIT 5",
    query
)
```
