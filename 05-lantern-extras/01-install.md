# Installation

## Install From Binaries

### Prerequisites

- PostgreSQL 11, 12, 13, 14, 15 or 16

### Supported platforms

- Linux (x86_64)

### Installation

Use our releases from GitHub. You can find available versions on the [releases](https://github.com/lanterndata/lantern/releases) page.

```bash
cd /tmp
VERSION=0.0.11
wget "https://github.com/lanterndata/lantern_extras/releases/download/${VERSION}/lantern-extras-${VERSION}.tar"
tar xf "lantern-extras-${VERSION}.tar"
cd "lantern-extras-${VERSION}"
make install
```

## Test

Note: The first run of each model will take longer as it will download the model file and tokenizer.

```sql
CREATE EXTENSION lantern_extras;
SELECT get_available_models(); -- get available models
SELECT clip_text('Hello world!'); -- generate embeddings using openai clip model (textual)
SELECT clip_image('https://storage.googleapis.com/lanterndata/images/icon100x100.png'); -- generate embeddings using openai clip model (visual)
-- using any model from the list
SELECT text_embedding('BAAI/bge-small-en', 'Hello world!');
```

## Install From Source

### Prerequisites

- cmake (>=3.3)
- gcc
- g++ (>=11)
- PostgreSQL 11, 12, 13, 14, 15 or 16
- Rust >= 1.70.0
- Corresponding development package for PostgreSQL (postgresql-server-dev-$version)

### Supported platforms

- Linux
- Mac

### Install

1. Clone repository from GitHub

   ```bash
   git clone https://github.com/lanterndata/lantern_extras.git
   ```

2. Install PGRX

   ```bash
   # install pgrx prerequisites
   sudo apt install pkg-config libssl-dev zlib1g-dev libreadline-dev
   sudo apt-get install clang
   cargo install --locked cargo-pgrx --version 0.9.7
   cargo pgrx init --pg$PG_VERSION pg_config
   ```

3. Install the extension

   ```bash
   cargo pgrx install --package lantern_extras --pg-config $(which pg_config)
   ```

4. Install ONNX Runtime

   Refer to [Installing ONNX Runtime guide](/docs/lantern-cli/install#install-onnx-runtime)

   Note: You should add the onnx library path to `ld.conf`, as environment variables may not be accessible from Postgres Server

5. Test Installation
   ```sql
   CREATE EXTENSION lantern_extras;
   SELECT get_available_models(); -- get available models
   SELECT clip_text('Hello world!'); -- generate embeddings using openai clip model (textual)
   SELECT clip_image('https://storage.googleapis.com/lanterndata/images/icon100x100.png'); -- generate embeddings using openai clip model (visual)
   -- using any model from the list
   SELECT text_embedding('BAAI/bge-small-en', 'Hello world!');
   ```
