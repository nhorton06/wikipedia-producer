# Wikipedia Producer

A containerized Kafka producer. This repository helps demonstrate automated tests and builds with GitHub Actions.

## Run locally against a Kafka/Redpanda stack in Docker

The container must join the same Docker network as your brokers so it can reach them by container name.

### 1. Find the network and broker names

List running containers with the networks they are attached to:

```bash
docker ps --format 'table {{.Names}}\t{{.Networks}}'
```

Note the network your broker containers use (e.g. `stream_network`). To list every container on that network:

```bash
docker network inspect <network> --format '{{range .Containers}}{{.Name}} {{end}}'
```

To confirm the broker addresses and internal port, ask a broker directly (Redpanda):

```bash
docker exec <broker-container> rpk cluster info
```

### 2. Build and run on that network

```bash
docker build -t wikipedia-producer .

docker run --rm --name wikipedia-producer \
  --network <network> \
  -e KAFKA_BROKER=<broker-1>:<port>,<broker-2>:<port>,<broker-3>:<port> \
  -e MAX_EVENTS=100 \
  wikipedia-producer
```

Example for a three-node Redpanda stack on `stream_network`:

```bash
docker run --rm --name wikipedia-producer \
  --network stream_network \
  -e KAFKA_BROKER=redpanda-0:9092,redpanda-1:9092,redpanda-2:9092 \
  -e MAX_EVENTS=100 \
  wikipedia-producer
```

Use the brokers' internal listener port (typically `9092`), not the host-mapped ports such as `19092`. `MAX_EVENTS` stops the producer after that many events; omit it (or set `0`) to run indefinitely.

## Unit Tests

Install the dependencies, including pytest:

```bash
pipenv sync --dev
```

Run the unit tests, which don't need a broker:

```bash
pipenv run pytest -m "not kafka" -v
```

Run the Kafka integration tests against a temporary Redpanda container:

```bash
docker compose -f compose.test.yaml up -d --wait
KAFKA_BROKER=127.0.0.1:49092 pipenv run pytest -m kafka -v
docker compose -f compose.test.yaml down -v
```

Test addition for automated test
