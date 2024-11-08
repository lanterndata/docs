# Rust

In the subsequent examples, assume the following variables are defined

```rust
// 384 is the dimensions of bge-small-en model
let embedding: vec<f32> = vec![1.0; 384];
let query = "my text input";
```

## rust-postgres

```shell
cargo add postgres
```

Connect

```rust
use postgres::{types::ToSql, Client, NoTls};

let mut client = Client::connect("postgres://postgres@localhost:5432/postgres", NoTls).unwrap();
let mut tx = client.transaction().unwrap();

tx.execute("CREATE EXTENSION IF NOT EXISTS lantern)", &[])
    .unwrap();
tx.execute("CREATE EXTENSION IF NOT EXISTS lantern_extras)", &[])
    .unwrap();
```

Insert a vector

```rust
tx.execute("CREATE TABLE books (book_embedding REAL[])", &[])
    .unwrap();

tx.execute(
    "INSERT INTO books (book_embedding) VALUES ($1)",
    &[&embedding as &(dyn ToSql + Sync)],
)
.unwrap();
```

Find nearest rows

```rust
tx.execute(
    "SELECT * FROM books ORDER BY book_embedding <-> $1 LIMIT 5",
    &[&embedding as &(dyn ToSql + Sync)],
)
.unwrap();

let rows = tx.query(
    "SELECT * FROM books ORDER BY book_embedding <-> text_embedding('BAAI/bge-small-en', $1) LIMIT 5",
    &[&query],
).unwrap();
```

## tokio-postgres

```shell
cargo add tokio-postgres
```

Insert a vector

```rust
tx.execute("CREATE TABLE books (book_embedding REAL[])", &[])
    .await.unwrap();

tx.execute(
    "INSERT INTO books (book_embedding) VALUES ($1)",
    &[&embedding as &(dyn ToSql + Sync)],
)
.await.unwrap();
```

Find nearest rows

```rust
tx.execute(
    "SELECT * FROM books ORDER BY book_embedding <-> $1 LIMIT 5",
    &[&embedding as &(dyn ToSql + Sync)],
)
.await.unwrap();

let rows = tx.query(
    "SELECT * FROM books ORDER BY book_embedding <-> text_embedding('BAAI/bge-small-en', $1) LIMIT 5",
    &[&query],
).await.unwrap();
```
