# Installation

Lantern Extras is a Postgres extension that enables embedding generation inside Postgres. It is available as a Docker image, a Homebrew package, a binary, or from source.

## Run with Docker

We provide the [Lantern Suite](https://hub.docker.com/r/lanterndata/lantern-suite/tags) Docker image, which includes both Lantern and Lantern Extras. This enables both vector search and vector generation inside Postgres.

```bash
docker run -p 5432:5432 --name lantern-demo -e 'POSTGRES_PASSWORD=postgres' -d lanterndata/lantern-suite:pg15-latest
```

Note: The image is based on the [official Postgres docker image](https://hub.docker.com/%5F/postgres). Please refer to the Postgres image documentation for a full list of supported features and flags.

## Install From Binaries

### Prerequisites

- PostgreSQL 11, 12, 13, 14, 15 or 16

### Supported platforms

- Linux (x86_64)

### Install

Use our releases from GitHub. You can find available versions on the [releases](https://github.com/lanterndata/lantern/releases) page.

Note: You can replace `VERSION` with the version you want to install

```bash
cd /tmp
VERSION=0.1.0
wget "https://github.com/lanterndata/lantern/releases/download/v${VERSION}/lantern-extras-${VERSION}.tar"
tar xf "lantern-${VERSION}.tar"
cd "lantern-${VERSION}"
make install
```

## Install From Source

### Prerequisites

- cmake version: >=3.3
- gcc && g++ version: >=11
- Rust version: >= 1.70.0
- PostgreSQL 11, 12, 13, 14, 15 or 16
- Corresponding development package for PostgreSQL (postgresql-server-dev-$version)

### Supported platforms

- Linux
- Mac

### Install

1. Clone repository from GitHub

   ```bash
   git clone https://github.com/lanterndata/lantern.git
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

## Post Installation

If you are not on Lantern Cloud, make sure you enable the extension after following the installation steps for Lantern Extras.

```sql
CREATE EXTENSION IF NOT EXISTS lantern_extras;
```

To test the installation, you can run the following SQL commands:

```sql
SELECT get_available_models(); -- get available models
SELECT clip_text('Hello world!'); -- generate embeddings using openai clip model (textual)
SELECT clip_image('https://storage.googleapis.com/lanterndata/images/icon100x100.png'); -- generate embeddings using openai clip model (visual)
-- using any model from the list
SELECT text_embedding('BAAI/bge-small-en', 'Hello world!');
```
