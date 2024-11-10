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

## ActiveRecord

Install the `lanterndb` gem

```bash
gem install lanterndb
```

Connect to the database and enable the extension:

```ruby
require 'active_record'
require 'lantern'
ActiveRecord::Base.establish_connection(DATABASE_URL)
ActiveRecord::Base.connection.enable_extension("lantern")
conn = ActiveRecord::Base.connection
```

Create a model

```ruby
ActiveRecord::Migration.create_table :movies do |t|
  t.column :movie_embedding, :real, array: true
end
conn.execute("INSERT INTO movies (movie_embedding) VALUES ('{0,1,0}'), ('{3,2,4}')")
```

Embedding generation:

```ruby
embedding1 = Lantern.text_embedding('BAAI/bge-base-en', 'Your text here')

Lantern.set_api_token(openai_token: 'your_openai_token')
embedding2 = Lantern.openai_embedding('text-embedding-3-small', 'Hello')

Lantern.set_api_token(cohere_token: 'your_cohere_token')
embedding3 = Lantern.cohere_embedding('embed-english-v3.0', 'Hello')
```

The gem provides several ways to perform vector search with the following distance metrics:

- `l2` (Euclidean distance)
- `cosine` (Cosine similarity)

Vector search using pre-computed vectors:

```ruby
class Document < ApplicationRecord
  has_neighbors :embedding
end

# Find 5 nearest neighbors using L2 distance
Document.nearest_neighbors(:embedding, [0.1, 0.2, 0.3], distance: 'l2').limit(5)

# Given a document, find 5 nearest neighbors using cosine distance
document = Document.first
document.nearest_neighbors(:embedding, distance: 'cosine').limit(5)
```

Vector search using text embeddings:

```ruby
class Book < ApplicationRecord
  has_neighbors :embedding
end

# Find 5 nearest neighbors using open-source model
Book.nearest_neighbors(:embedding, 'The quick brown fox', model: 'BAAI/bge-small-en', distance: 'l2').limit(5)

# Find 5 nearest neighbors using OpenAI
Lantern.set_api_token(openai_token: 'your_openai_token')
Book.nearest_neighbors(:embedding, 'The quick brown fox', model: 'openai/text-embedding-3-small', distance: 'cosine').limit(5)
```

To speed up vector search queries, you can add an HNSW vector index to your model:

```ruby
class CreateVectorIndex < ActiveRecord::Migration[7.0]
  def up
    add_index :books, :embedding, using: :lantern_hnsw, opclass: :dist_l2sq_ops, name: 'book_embedding_index'
  end

  def down
    remove_index :books, name: 'book_embedding_index'
  end
end
```

Note: This does not support `WITH` parameters (e.g., `ef_construction`, `ef`, `m`, `dim`). To specify `WITH` parameters, you can pass them as options with raw SQL:

```ruby
class CreateHnswIndex < ActiveRecord::Migration[7.0]
  def up
    execute <<-SQL
      CREATE INDEX movie_embedding_hnsw_idx
      ON movies
      USING lantern_hnsw (movie_embedding dist_l2sq_ops)
      WITH (
        ef = 15,
        m = 16,
        ef_construction = 64
      )
    SQL
  end

  def down
    remove_index :movies, name: 'movie_embedding_hnsw_idx'
  end
end
```

## Rails

Add the gem to your Gemfile:

```ruby
gem 'lanterndb'
```

Run the following command to install the gem:

```bash
bundle install
```

Enable the Lantern extension using the provided generator:

```bash
rails generate lantern:install
rails db:migrate
```

This will:

- Create a migration to enable the Lantern extension
- Run the migration to enable the extension in your database

After this, you can use all ActiveRecord features described above in your Rails application.
