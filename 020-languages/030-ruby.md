# Ruby

In the following examples, assume the following variables are defined

```ruby
DATABASE_URL = "postgres://postgres:postgres@localhost:5432/postgres"
embedding = [1, 2, 3]
query = "My text input"
```

## pg

Install the `pg` gem

```bash
gem install pg
```

Connect to database and store / query vectors

```ruby
require 'pg'

conn = PG.connect(DATABASE_URL)

# Enable the extension
conn.exec('CREATE EXTENSION IF NOT EXISTS lantern')

# Create a table
conn.exec('CREATE TABLE IF NOT EXISTS items (id bigserial PRIMARY KEY, embedding REAL[3])')

# Insert a vector
conn.exec_params("INSERT INTO books (book_embedding) VALUES ($1)", [embedding])

# Find nearest rows to a vector
conn.exec_params("SELECT * FROM books ORDER BY book_embedding <-> $1 LIMIT 5", [embedding])

# Find nearest rows to a vector generated from text
conn.exec_params("SELECT * FROM books ORDER BY book_embedding <-> text_embedding('BAAI/bge-small-en', $1) LIMIT 5", [query])

conn.close
```

## Sequel

Install the `sequel` gem

```bash
gem install sequel
```

Connect to database and store / query vectors

```ruby
require 'sequel'

# Connect to the database
DB = Sequel.connect(DATABASE_URL)

# Create a table
DB.create_table?(:items) do
  primary_key :id
  column :embedding, 'REAL[3]'
end

# Insert a vector
DB[:books].insert(book_embedding: embedding)

# Find nearest rows to a vector
DB[:books].order(Sequel.lit('book_embedding <-> ?', embedding)).limit(5)

# Find nearest rows to a vector generated from text
DB[:books].order(Sequel.lit("book_embedding <-> text_embedding(?, ?)", 'BAAI/bge-small-en', query)).limit(5)
```

