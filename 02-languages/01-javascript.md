# Javascript

In the subsequent examples, assume the following variables are defined

```javascript
const embedding = [1, 2, 3];
const query = "My text input";
```

## node-postgres

Insert embedding

```javascript
await client.query("INSERT INTO books (embedding) VALUES ($1)", embedding);
```

Find nearest rows

```javascript
const result = await client.query(
  "SELECT * FROM books ORDER BY embedding <-> $1 LIMIT 5",
  embedding
);
const result = await client.query(
  "SELECT * FROM books ORDER BY embedding <-> text_embedding('BAAI/bge-small-en', $1) LIMIT 5",
  query
);
```

## pg-promise

Insert embedding

```javascript
await db.none("INSERT INTO books (embedding) VALUES ($1)", embedding);
```

Find nearest rows

```javascript
const result = await db.any(
  "SELECT * FROM books ORDER BY embedding <-> $1 LIMIT 5",
  embedding
);
const result = await db.any(
  "SELECT * FROM books ORDER BY embedding <-> text_embedding('BAAI/bge-small-en', $1) LIMIT 5",
  query
);
```

## Prisma

Insert embedding

```javascript
await db.books.create({ embedding });
```

Find nearest rows

```javascript
const result =
  await Prisma.$queryRaw`SELECT * FROM books ORDER BY embedding <-> ${embedding} LIMIT 5`;
const result =
  await Prisma.$queryRaw`SELECT * FROM books ORDER BY text_embedding('BAAI/bge-small-en', ${query}) <-> book_embedding LIMIT 5`;
```

## Kysely

Insert embedding

```javascript
await db.insert({ embedding }).into("books");
```

Find nearest rows

```javascript
const result = await db
  .select()
  .from("books")
  .orderBy(sql`embedding <-> ${embedding}`)
  .limit(5);
const result = await db
  .selectAll()
  .from("books")
  .orderBy(
    sql`text_embedding('BAAI/bge-small-en', ${query}) <-> embedding`
  )
  .limit(5);
```

## Sequelize

See the [client library documentation](https://github.com/lanterndata/lantern-js/blob/main/src/sequelize) for more examples.

Setup

```typescript
import lantern from 'lanterndata/sequelize';
import { Sequelize } from 'sequelize';

const sequelize = Sequelize();

lantern.extend(sequelize);

await sequelize.createLanternExtension();
await sequelize.createLanternExtrasExtension();
```

Create table and add index


```typescript
const Book = sequelize.define('Book', {
  id: { type: DataTypes.INTEGER, autoIncrement: true, primaryKey: true },
  embedding: { type: DataTypes.ARRAY(DataTypes.REAL) },
  name: { type: DataTypes.TEXT },
  url: { type: DataTypes.TEXT },
}, {
  modelName: 'Book',
  tableName: 'books',
});

await sequelize.query(`
  CREATE INDEX book_index ON books USING hnsw(book_embedding dist_l2sq_ops)
  WITH (M=2, ef_construction=10, ef=4, dims=3);
`);
```

Insert embedding

```javascript
await Book.create({ embedding });
```

Vector search

```typescript
await Book.findAll({
  order: sequelize.l2Distance('embedding', [1, 1, 1]),
  limit: 5,
});

await Book.findAll({
  order: sequelize.cosineDistance('embedding', [1, 1, 1]),
  limit: 5,
});

await Book.findAll({
  order: sequelize.hammingDistance('embedding', [1, 1, 1]),
  limit: 5,
});
```

Vector search with embedding generation

```typescript
import { ImageEmbeddingModels } from 'lanterndata/embeddings';

const {CLIP_VIT_B_32_VISUAL} = ImageEmbeddingModels;

const bookEmbeddingsOrderd = await Book.findAll({
  order: [[sequelize.l2Distance('embedding', sequelize.imageEmbedding(CLIP_VIT_B_32_VISUAL, 'url')), 'desc']],
  where: { url: { [Op.not]: null } },
  limit: 2,
});
```

Generate text and image embeddings (static)

```typescript
import { TextEmbeddingModels, ImageEmbeddingModels } from 'lanterndata/embeddings';

const text = 'hello world';
const [result] = await sequelize.generateTextEmbedding(TextEmbeddingModels.BAAI_BGE_BASE_EN, text);

const imageUrl = 'https://lantern.dev/images/home/footer.png';
const [result] = await sequelize.generateImageEmbedding(ImageEmbeddingModels.CLIP_VIT_B_32_VISUAL, imageUrl);
```

Generate text and image embeddings (dynamic)

```typescript
import { TextEmbeddingModels, ImageEmbeddingModels } from 'lanterndata/embeddings';

const bookTextEmbeddings = await Book.findAll({
  attributes: ['name', sequelize.textEmbedding(TextEmbeddingModels.BAAI_BGE_BASE_EN, 'name')],
  where: { name: { [Op.not]: null } },
  limit: 5,
  raw: true,
});

const bookImageEmbeddings = await Book.findAll({
  attributes: ['url', sequelize.imageEmbedding(ImageEmbeddingModels.CLIP_VIT_B_32_VISUAL, 'url')],
  where: { url: { [Op.not]: null } },
  limit: 5,
  raw: true,
});
```

## Knex

See the [client library documentation](https://github.com/lanterndata/lantern-js/blob/main/src/knex) for more examples.

Setup

```typescript
import Knex from 'knex';
import 'lanterndata/knex';

const knex = Knex();

await knex.schema.createLanternExtension();
await knex.schema.createLanternExtrasExtension()
```

Create table and add index

```typescript
await knex.schema.createTable('books', (table) => {
  table.increments('id');
  table.specificType('url', 'TEXT');
  table.specificType('name', 'TEXT');
  table.specificType('embedding', 'REAL[]');

  knex.raw(`
    CREATE INDEX book_index ON books USING hnsw(book_embedding dist_l2sq_ops)
    WITH (M=2, ef_construction=10, ef=4, dims=3);
  `);
});
```

Insert embedding

```javascript
await knex("books").insert({ embedding });
```

Vector search

```typescript
await knex('books')
  .orderBy(knex.l2Distance('embedding', [1, 1, 1]))
  .limit(5);

await knex('books')
  .orderBy(knex.cosineDistance('embedding', [1, 1, 1]))
  .limit(5);

await knex('books')
  .orderBy(knex.hammingDistance('embedding', [1, 1, 1]))
  .limit(5);

// Vector search with embedding generation
await knex('books')
  .orderBy(
    knex.l2Distance(
      'embedding',
      knex.imageEmbedding(TextEmbeddingModels.BAAI_BGE_BASE_EN, 'name')
    )
  )
  .limit(2);
```

Generate text and image embeddings (static)

```typescript
import Knex from 'knex';
import 'lanterndata/knex';
import { TextEmbeddingModels, ImageEmbeddingModels } from 'lanterndata/embeddings';

const text = 'hello world';
const embedding = await knex.generateTextEmbedding(TextEmbeddingModels.BAAI_BGE_BASE_EN, text);

const imageUrl = 'https://lantern.dev/images/home/footer.png';
const embedding = await knex.generateImageEmbedding(ImageEmbeddingModels.CLIP_VIT_B_32_VISUAL, imageUrl);
```

Generate text and image embeddings (dynamic)

```typescript
import Knex from 'knex';
import 'lanterndata/knex';
import { TextEmbeddingModels, ImageEmbeddingModels } from 'lanterndata/embeddings';

const selectLiteral = knex.textEmbedding(TextEmbeddingModels.BAAI_BGE_BASE_EN, 'name');
const bookTextEmbeddings = await knex('books')
    .select('name')
    .select(selectLiteral)

const selectLiteral = knex.imageEmbedding(ImageEmbeddingModels.BAAI_BGE_BASE_EN, 'url');
const bookImageEmbeddings = await knex('books')
    .select('url')
    .select(selectLiteral)
```

## Drizzle ORM

See the [client library documentation](https://github.com/lanterndata/lantern-js/blob/main/src/drizzle-orm) for more examples.

Setup

```typescript
import postgres from 'postgres';
import { drizzle } from 'drizzle-orm/postgres-js';
import { createLanternExtension, createLanternExtrasExtension } from 'lanterndata/drizzle-orm';

const client = await postgres();
const db = drizzle(client);

await db.execute(createLanternExtension());
await db.execute(createLanternExtrasExtension());
```

Create table and add index

```typescript
const Book = pgTable('books', {
  id: serial('id').primaryKey(),
  name: text('name'),
  url: text('url'),
  embedding: real('embedding').array(),
});

await client`CREATE INDEX book_index ON books USING hnsw(embedding dist_l2sq_ops)`;
```

Vector search

```typescript
import { l2Distance, cosineDistance, hammingDistance } from 'lanterndata/drizzle-orm';

await db
  .select()
  .from(Book)
  .orderBy(l2Distance(Book.embedding, [1, 1, 1]))
  .limit(5);

await db
  .select()
  .from(Book)
  .orderBy(cosineDistance(Book.embedding, [1, 1, 1]))
  .limit(5);

await db
  .select()
  .from(Book)
  .orderBy(hammingDistance(Book.embedding, [1, 1, 1]))
  .limit(5);
```

Vector search with embedding generation

```typescript
import { l2Distance, imageEmbedding } from 'lanterndata/drizzle-orm';
import { TextEmbeddingModels, ImageEmbeddingModels } from 'lanterndata/embeddings';

const { CLIP_VIT_B_32_VISUAL } = ImageEmbeddingModels;

await db
  .select()
  .from(Book)
  .orderBy(desc(l2Distance(Book.embedding, imageEmbedding(CLIP_VIT_B_32_VISUAL, Book.url))))
  .limit(2);
```

Generate text and image embeddings (static)

```typescript
import { TextEmbeddingModels, ImageEmbeddingModels } from 'lanterndata/embeddings';
import { generateTextEmbedding, generateImageEmbedding } from 'lanterndata/drizzle-orm';

const text = 'hello world';
const result = await db.execute(generateTextEmbedding(TextEmbeddingModels.BAAI_BGE_BASE_EN, text));

const imageUrl = 'https://lantern.dev/images/home/footer.png';
const result = await db.execute(generateImageEmbedding(ImageEmbeddingModels.CLIP_VIT_B_32_VISUAL, imageUrl));
```

Generate text and image embeddings (dynamic)

```typescript
import { textEmbedding, imageEmbedding } from 'lanterndata/drizzle-orm';
import { TextEmbeddingModels, ImageEmbeddingModels } from 'lanterndata/embeddings';

const bookTextEmbeddings = await db
  .select({
    name: Book.name,
    text_embedding: textEmbedding(TextEmbeddingModels.BAAI_BGE_BASE_EN, Book.name),
  })
  .from(Book)
  .where(isNotNull(Book.name))
  .limit(5);

const bookImageEmbeddings = await db
  .select({
    url: Book.url,
    image_embedding: imageEmbedding(ImageEmbeddingModels.CLIP_VIT_B_32_VISUAL, Book.url),
  })
  .from(Book)
  .where(isNotNull(Book.name))
  .limit(5);
```

## Postgres.js
