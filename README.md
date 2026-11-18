# node

node is a TypeScript application for timestamping documents using Bitcoin and IPFS. It accepts signed verifiable work claims and exposes an HTTP API for publishing and retrieving them.

## Features

- Publish signed verifiable work claims through `POST /works`.
- Retrieve claims by ID or list them with issuer filtering and pagination.
- Upload one file per request through `POST /files` and receive its hash and archive URL.
- Anchor claims to Bitcoin when anchoring is enabled and a funded Bitcoin Core wallet is available.

## Installation

Install [Docker](https://docs.docker.com/install/) and [docker-compose](https://docs.docker.com/compose/install/), then open a local checkout of this repository. From its parent directory, run:

```bash
cd node
docker-compose build
```

The Docker Compose environment sets up the application and its dependencies: RabbitMQ, IPFS, Bitcoin Core, and MongoDB.

## Usage

Start the local environment:

```bash
docker-compose up --build
```

In the default development setup, Bitcoin blocks must be generated manually. Blockchain anchoring is disabled by default (`enableAnchoring: false`). To enable it, set `ENABLE_ANCHORING=true` in the application's environment and provide Bitcoin Core with a funded wallet for the selected network. Mainnet anchoring requires real bitcoin to pay transaction fees.

Configuration defaults are defined in `src/Configuration.ts`. Override settings through environment variables using their SCREAMING_SNAKE_CASE names; for example, `rabbitmqUrl` becomes `RABBITMQ_URL`. Restart the application after changing configuration.

The HTTP API exposes these endpoints:

| Endpoint | Purpose |
| --- | --- |
| `GET /works?issuer=xxx&limit=x&offset=x` | List claims, optionally filtered by issuer. `limit` defaults to 10. |
| `GET /works/:id` | Retrieve a claim and its available `.anchor` metadata. |
| `POST /works` | Submit a fully constructed, signed verifiable work claim. |
| `POST /files` | Upload one file using a multipart request with one file field. |

`POST /works` acknowledges submissions asynchronously. An acknowledgement does not confirm processing, IPFS publication, or Bitcoin anchoring. Retrieve the claim with `GET /works/:id` and inspect `.anchor` to check anchoring information. A 404 means the claim is absent from this node's database, not necessarily from other nodes.

Stop the environment while keeping its data:

```bash
docker-compose down
```

To stop it and delete its stored volume data instead:

```bash
docker-compose down --volumes
```

## License

MIT. This project is based on [poetapp/node](https://github.com/poetapp/node), originally described as the core of Po.et. Retain the upstream copyright and MIT license notice when redistributing upstream code or substantial portions of it.
