# Installation

## Run with Docker

You can use the [Docker](https://hub.docker.com/r/lanterndata/lantern-cli/tags) image to quickly run Lantern CLI with docker.

### Run CPU image

```bash
docker run --rm lanterndata/lantern-cli -h
```

### Run GPU image

```bash
nvidia-docker run --rm lanterndata/lantern-cli:gpu -h
```

### Using volumes

When running `create-embeddings` command pass `-v lantern-models:/models` to Docker and `--data-path /models` to CLI, so the downloaded models will persist in a Docker volume.
Example:

```bash
docker run -v lantern-models:/models --rm --network host lanterndata/lantern-cli create-embeddings --model 'BAAI/bge-large-en' --uri 'postgresql://postgres@host.docker.internal:5432/postgres' --table "wiki" --column "content" --out-column "content_embedding" --pk "id" --batch-size 40 --data-path /models
```

In case of `start-daemon` command, just pass `-v lantern-daemon-data:/var/lib/lantern-daemon` to Docker

## Install ONNX Runtime

[ONNX Runtime](https://onnxruntime.ai/) is required for Lantern CLI Embeddings. This is pre-packaged in the Docker image. It needs to be installed separately when installing from binaries. For Cargo, it is required except for Linux.

1. Get ONNX Runtime library

   Get ONNX Runtime library from [Github release page](https://github.com/microsoft/onnxruntime/releases/tag/v1.16.1)

   ```bash
   PLATFORM=linux
   ARCH=x64
   ONNX_VERSION=1.16.1

   wget "https://github.com/microsoft/onnxruntime/releases/download/v${ONNX_VERSION}/onnxruntime-${PLATFORM}-${ARCH}-${ONNX_VERSION}.tgz"
   ```

   Note: If you have GPU, download the GPU version of onnxruntime library.

2. Extract archive

   Extract the downloaded archive in `/usr/local/lib`

   ```bash
     mkdir -p /usr/local/lib
     tar xzf "onnxruntime-${PLATFORM}-${ARCH}-${ONNX_VERSION}.tar" -C /usr/local/lib
   ```

3. Setup environment

   Export path to ONNX library

   ```bash
   export ORT_STRATEGY=system
   export ORT_DYLIB_PATH=/usr/local/lib/onnxruntime-${PLATFORM}-${ARCH}-${ONNX_VERSION}/lib/libonnxruntime.so
   ```

   Note: On MacOS the library will have `.dylib` extension instead of `.os`
   You can export this variables from your shell profile or add them to `ld` search path like this:

   ```bash
   echo "/usr/local/lib/onnxruntime-${PLATFORM}-${ARCH}-${ONNX_VERSION}/lib/libonnxruntime.so" > /etc/ld.so.conf.d/onnx.conf
   ldconfig
   ```

## Install From Binaries

### Prerequisities

- [ONNX Runtime](https://onnxruntime.ai/) - see previous section for instructions

### Supported platforms

- Linux (x86_64)

### Use our releases from GitHub

You can download prebuilt binaries from our [releases](https://github.com/lanterndata/lantern/releases) page

## Install With Cargo

### Prerequisites

- Rust >=1.70.0
- Cargo
- [ONNX Runtime](https://onnxruntime.ai/) - see earlier section for instructions

### Installation

```bash
cargo install --git https://github.com/lanterndata/lantern.git
```

### Verify installation

```bash
lantern-cli -h
```
