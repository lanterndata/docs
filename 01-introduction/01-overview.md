# Overview

Lantern is the only database you need for your AI applications.

- **Built on top of Postgres** - Add AI to your existing applications by just writing another SQL query. We support Postgres 11 - 16.
- **Fast index creation that scales to billions of vectors** - Create an HNSW index for your data in minutes (see our [benchmarks](https://lantern.dev/blog/hnsw-index-creation))
- **Create embeddings without worrying about infrastructure** - With [Lantern Cloud](https://lantern.dev), simply select the model you'd like to use to embed your data, and we'll handle the rest. No need to worry about finding the right library, switching languages, or handling API call failures
- **_Coming Soon:_ Index hyperparameter optimization** - With Lantern Cloud, we'll automatically tune your index for you, so you don't have to worry about finding the right set of parameters, and changing them over time

## Getting Started

The easiest way to get started with Lantern is with [Lantern Cloud](https://lantern.dev). Lantern Cloud is a fully managed database offering with support for embedding generation and management. We also provide [guides](/develop/get-started) on self-hosting Lantern.

## Develop

Check our [development guide](/develop/store) to learn more about how to use Lantern. We support generating vectors with a a variety of embedding models out of the box. With a single operator, you can query your vectors using a range of distance functions, and filter these results using your existing data.
