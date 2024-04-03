# Python

In the subsequent examples, assume the following variables are defined

```python
DATABASE_URL = "postgresql://postgres:postgres@localhost:5432/postgres"
embedding = [1, 2, 3]
query = "My text input"
```

## psycopg2

Install `psycopg2`

```bash
pip install psycopg2-binary
```

Connect to database and store / query vectors

```python
import psycopg2

conn = psycopg2.connect(DATABASE_URL)
cur = conn.cursor()

# Insert a vector
cur.execute("INSERT INTO books (book_embedding) VALUES (%s)", (embedding,))

# Find nearest rows to a vector
cur.execute(f"SELECT * FROM books ORDER BY book_embedding <-> %s LIMIT 5", (embedding,))

# Find nearest rows to a vector generated from text
cur.execute(f"SELECT * FROM books ORDER BY book_embedding <-> text_embedding('BAAI/bge-small-en', %s) LIMIT 5", (query,))

cur.close()
conn.close()
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

Install `asyncpg`

```bash
pip install asyncpg
```

Connect to database and store / query vectors

```python
import asyncpg

conn = await asyncpg.connect(DATABASE_URL)

# Insert a vector
await conn.execute("INSERT INTO books (book_embedding) VALUES ($1)", embedding)

# Find nearest rows to a vector
await conn.fetch(f"SELECT * FROM books ORDER BY book_embedding <-> $1 LIMIT 5", embedding)

# Find nearest rows to a vector generated from text
await conn.fetch(f"SELECT * FROM books ORDER BY book_embedding <-> text_embedding('BAAI/bge-small-en', $1) LIMIT 5", query)

await conn.close()
```

## Lantern Python Client

See the [Github repo](https://github.com/lanterndata/lantern-python/tree/main/lantern) for documentation and examples.

## Lantern Pinecone Client

See the [Github repo](https://github.com/lanterndata/lantern-python/tree/main/lantern_pinecone) for documentation and examples.
