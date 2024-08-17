# Dockerfile

[`Dockerfile.dev`](https://github.com/lanterndata/lantern/blob/main/Dockerfile.dev) streamlines the setup process for contributors, offering a consistent Dockerized development environment.

## Steps

1. Build the Docker image

   Navigate to the root directory and execute

   ```bash
   docker build -t lantern-dev -f docker/Dockerfile.dev .
   ```

2. Run the Docker container

   ```bash
   docker run --name lantern-container -d lantern-dev
   ```

3. Access the development environment

   ```bash
   docker exec -it lantern-container /bin/bash
   ```

Inside the container, the `lantern` source is ready in `/lantern` and benchmarking tools are set up in the `benchmark` directory.
