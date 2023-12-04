# Installation

## Run with Docker

You can use [Docker](https://hub.docker.com/r/lanterndata/lantern/tags) image to quickly install Lantern with docker.

Note: The image is based on the [official Postgres docker image](https://hub.docker.com/%5F/postgres). Please refer to the Postgres image documentation for a full list of supported features and flags.

```bash
docker run -p 5432:5432 --name lantern-demo -e 'POSTGRES_PASSWORD=postgres' -d lanterndata/lantern:latest-pg15
```

## Install From Binaries

### Prerequisites

- PostgreSQL 11, 12, 13, 14, 15 or 16

### Supported platforms

- Linux (x86_64)
- Mac (intel)

Use our releases from [GitHub](https://github.com/lanterndata/lantern/releases).

### Installation

Note: You can replace 0.0.5 with the version you want to install

```bash
cd /tmp
LDB_VERSION=0.0.8
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

- cmake (>=3.3)
- gcc
- g++ (>=11)
- PostgreSQL 11, 12, 13, 14, 15 or 16
- Corresponding development package for PostgreSQL (postgresql-server-dev-$version)

### Supported platforms

- Linux
- Mac

### Installation

```bash
git clone --recursive https://github.com/lanterndata/lantern.git
cd lantern
mkdir build
cd build
cmake ..
make install
```
