# Installation

Lantern is a Postgres extension that provides vector search inside Postgres. It is available as a Docker image, a Homebrew package, a binary, or from source.

## Run with Docker

For convenience, we provide the [Lantern Suite](https://hub.docker.com/r/lanterndata/lantern-suite/tags) Docker image, which includes both Lantern and Lantern Extras. This enables both vector search and vector generation inside Postgres.

```bash
docker run -p 5432:5432 --name lantern-demo -e 'POSTGRES_PASSWORD=postgres' -d lanterndata/lantern-suite:pg15-latest
```

Alternatively, to install only Lantern for vector search, we provide the [Lantern](https://hub.docker.com/r/lanterndata/lantern/tags) Docker image.

```bash
docker run -p 5432:5432 --name lantern-demo -e 'POSTGRES_PASSWORD=postgres' -d lanterndata/lantern:pg15-latest
```

Note: The image is based on the [official Postgres docker image](https://hub.docker.com/%5F/postgres). Please refer to the Postgres image documentation for a full list of supported features and flags.

## Install From Binaries

### Prerequisites

- PostgreSQL 11, 12, 13, 14, 15 or 16

### Supported platforms

- Linux (x86_64)
- Mac (intel)

### Installation

Use our releases from [GitHub](https://github.com/lanterndata/lantern/releases).

Note: You can replace `LDB_VERSION` with the version you want to install

```bash
cd /tmp
LDB_VERSION=0.3.4
wget "https://github.com/lanterndata/lantern/releases/download/v${LDB_VERSION}/lantern-${LDB_VERSION}.tar"
tar xf "lantern-${LDB_VERSION}.tar"
cd "lantern-${LDB_VERSION}"
make install
```

## Install From Homebrew

### Prerequisites

- [Homebrew](https://brew.sh/)

### Supported platforms

- Mac

### Installation

```bash
brew tap lanterndata/lantern
brew install lantern && lantern_install
```

## Install From Source

### Prerequisites

- cmake version: >=3.3
- gcc && g++ version: >=11 when building portable binaries, >= 12 when building on new hardware or with CPU-specific vectorization
- PostgreSQL 11, 12, 13, 14, 15 or 16
- Corresponding development package for PostgreSQL (postgresql-server-dev-$version)

### Supported platforms

- Linux
- Mac

### Installation

```bash
git clone --recursive https://github.com/lanterndata/lantern.git
cd lantern
cmake -DMARCH_NATIVE=ON -S lantern_hnsw -B build
make -C build install -j
```

## Post Installation

If you are not on Lantern Cloud, make sure you enable the extension after following the installation steps for Lantern.

```sql
CREATE EXTENSION IF NOT EXISTS lantern;
```
