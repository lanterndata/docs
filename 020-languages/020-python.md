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

with psycopg2.connect(DATABASE_URL) as conn:
    with conn.cursor() as cur:

        # Enable the extension
        cur.execute('CREATE EXTENSION IF NOT EXISTS lantern')

        # Create a table
        cur.execute('CREATE TABLE IF NOT EXISTS items (id bigserial PRIMARY KEY, embedding REAL[3])')
        conn.commit()

        # Insert a vector
        cur.execute("INSERT INTO books (book_embedding) VALUES (%s)", (embedding,))
        conn.commit()

        # Find nearest rows to a vector
        cur.execute("SELECT * FROM books ORDER BY book_embedding <-> %s LIMIT 5", (embedding,))
        nearest_rows = cur.fetchall()

        # Find nearest rows to a vector generated from text
        cur.execute("SELECT * FROM books ORDER BY book_embedding <-> text_embedding('BAAI/bge-small-en', %s) LIMIT 5", (query,))
        nearest_rows = cur.fetchall()
```

## psycopg3

Install `psycopg3`

```bash
pip install psycopg[binary]
```

Connect to the database and store / query vectors

```python
import psycopg

with psycopg.connect(DATABASE_URL, autocommit=True) as conn:
    with conn.cursor() as cur:

        # Enable the extension
        cur.execute('CREATE EXTENSION IF NOT EXISTS lantern')

        # Create a table
        conn.execute('CREATE TABLE items (id bigserial PRIMARY KEY, embedding REAL[3])')

        # Insert a vector
        cur.execute("INSERT INTO books (book_embedding) VALUES (%s)", (embedding,))

        # Find nearest rows to a vector
        cur.execute("SELECT * FROM books ORDER BY book_embedding <-> %s LIMIT 5", (embedding,))
        nearest_rows = cur.fetchall()

        # Find nearest rows to a vector generated from text
        cur.execute("SELECT * FROM books ORDER BY book_embedding <-> text_embedding('BAAI/bge-small-en', %s) LIMIT 5", (query,))
        nearest_rows = cur.fetchall()
```

## asyncpg

Install `asyncpg`

```bash
pip install asyncpg
```

Connect to database and store / query vectors

```python
import asyncio
import asyncpg

async def main():
    conn = await asyncpg.connect(DATABASE_URL)
    try:
        # Enable the extension
        await conn.execute('CREATE EXTENSION IF NOT EXISTS lantern')

        # Create a table
        await conn.execute('CREATE TABLE IF NOT EXISTS items (id bigserial PRIMARY KEY, embedding REAL[3])')

        # Insert a vector
        await conn.execute("INSERT INTO books (book_embedding) VALUES ($1)", embedding)

        # Find nearest rows to a vector
        nearest_rows = await conn.fetch("SELECT * FROM books ORDER BY book_embedding <-> $1 LIMIT 5", embedding)

        # Find nearest rows to a vector generated from text
        nearest_text_rows = await conn.fetch(
            "SELECT * FROM books ORDER BY book_embedding <-> text_embedding('BAAI/bge-small-en', $1) LIMIT 5",
            query
        )
    finally:
        # Close the connection
        await conn.close()

# Run the async function
asyncio.run(main())
```

## Lantern Python Client

See the [Github repo](https://github.com/lanterndata/lantern-python/tree/main/lantern) for documentation and examples.

## Lantern Pinecone Client

See the [Github repo](https://github.com/lanterndata/lantern-python/tree/main/lantern_pinecone) for documentation and examples.
