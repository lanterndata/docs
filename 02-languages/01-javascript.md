# Javascript, Typescript

In the subsequent examples, assume the following variables are defined

```javascript
const embedding = [1, 2, 3];
const query = "My text input";
```

## node-postgres

Insert embedding

```javascript
await client.query("INSERT INTO books (book_embedding) VALUES ($1)", embedding);
```

Find nearest rows

```javascript
const result = await client.query(
  "SELECT * FROM book_embedding ORDER BY book_embedding <-> $1 LIMIT 5",
  embedding
);
const result = await client.query(
  "SELECT * FROM book_embedding ORDER BY book_embedding <-> text_embedding('BAAI/bge-small-en', $1) LIMIT 5",
  query
);
```

## pg-promise

Insert embedding

```javascript
await db.none("INSERT INTO books (book_embedding) VALUES ($1)", embedding);
```

Find nearest rows

```javascript
const result = await db.any(
  "SELECT * FROM book_embedding ORDER BY book_embedding <-> $1 LIMIT 5",
  embedding
);
const result = await db.any(
  "SELECT * FROM book_embedding ORDER BY book_embedding <-> text_embedding('BAAI/bge-small-en', $1) LIMIT 5",
  query
);
```

## Prisma

Insert embedding

```javascript
await db.books.create({ book_embedding: embedding });
```

Find nearest rows

```javascript
const result =
  await Prisma.$queryRaw`SELECT * FROM books ORDER BY book_embedding <-> ${embedding} LIMIT 5`;
const result =
  await Prisma.$queryRaw`SELECT * FROM books ORDER BY text_embedding('BAAI/bge-small-en', ${query}) <-> book_embedding LIMIT 5`;
```

## Kysely

Insert embedding

```javascript
await db.insert({ book_embedding: embedding }).into("books");
```

Find nearest rows

```javascript
const result = await db
  .select()
  .from("books")
  .orderBy(sql`book_embedding <-> ${embedding}`)
  .limit(5);
const result = await db
  .selectAll()
  .from("books")
  .orderBy(
    sql`text_embedding('BAAI/bge-small-en', ${query}) <-> book_embedding`
  )
  .limit(5);
```

## Postgres.js

## Drizzle ORM

## Sequelize

## Knex
