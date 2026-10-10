# Spitfire examples

Example load tests for [Spitfire](https://spitfire.tr/en), a self-hosted, distributed load testing platform with a web UI. Each file is a complete test: open it in the web editor or run it with the CLI.

| Example | Protocol | What it shows |
|---|---|---|
| [`http/shop-checkout.json`](http/shop-checkout.json) | HTTP | A user journey: log in once, take the token, list products, add to cart, check out; checks and per-step thresholds |
| [`http/constant-rate-with-spike.json`](http/constant-rate-with-spike.json) | HTTP | A constant arrival rate with a spike scenario starting at 4 minutes; abort when the error rate crosses 2% |
| [`grpc/orders-and-chat.json`](grpc/orders-and-chat.json) | gRPC | 300 unary calls/s with metadata, plus 20 bidirectional streams; time to first message and per-message latency thresholds |
| [`kafka/produce-consume.json`](kafka/produce-consume.json) | Kafka | 500 events/s, 10 consumers, end-to-end latency and the lag of a real service's consumer group |
| [`mqtt/device-telemetry.json`](mqtt/device-telemetry.json) | MQTT | 500 devices connecting over 2 minutes, QoS 1 telemetry every 4–6 s, a wildcard listener |
| [`rabbitmq/order-queue.json`](rabbitmq/order-queue.json) | AMQP 0-9-1 | 300 persistent messages/s with publisher confirms, 10 consumers |
| [`redis/session-cache.json`](redis/session-cache.json) | Redis | 2,000 iterations/s of GET + SET with a TTL; a missing key counts as success |
| [`sql/postgres-read.json`](sql/postgres-read.json) | PostgreSQL | 30 concurrent readers: a lookup by id and an hourly summary query |
| [`ci/github-actions.yml`](ci/github-actions.yml) | — | A performance gate on every pull request with the [Spitfire GitHub Action](https://github.com/taner-akdemir/spitfire-action) |
| [`ci/gitlab-ci.yml`](ci/gitlab-ci.yml) | — | The same in GitLab CI with the [Spitfire template](ci/gitlab/spitfire.gitlab-ci.yml) (`include: remote` + `extends`) |
| [`grafana/`](grafana) | — | A Grafana dashboard for Spitfire's Prometheus endpoint (live runs, runners, the controller's health) and a scrape example |

Every test here is checked with `spitfire validate`.

## Install Spitfire

```bash
# one machine: Postgres + controller + web UI + 2 runners (Docker)
curl -fsSL https://spitfire.tr/install.sh | bash -s -- docker

# the current kubectl context
curl -fsSL https://spitfire.tr/install.sh | bash -s -- kubernetes
```

The web UI opens on port 8470. More: [installation guide](https://spitfire.tr/en/install).

## Use an example

**In the web UI:** Tests → **Open file** → pick a `.json` file. Non-HTTP examples refer to a connection by name (`orders-grpc`, `chat-grpc`, `kafka`, `broker`, `rabbit`, `cache`, `orders-db`): add a connection with that name under **Connections**, or rename it in the test. Change `base` and the other variables to your own system.

**With the CLI**, no controller needed. The CLI ships in the Docker image:

```bash
docker run --rm -v "$PWD:/work" -w /work algebransoft/spitfire spitfire validate http/shop-checkout.json
docker run --rm -v "$PWD:/work" -w /work algebransoft/spitfire spitfire run http/constant-rate-with-spike.json
```

For the non-HTTP examples, copy [`connections.example.json`](connections.example.json), fill in your brokers and databases, and pass it with `-c`:

```bash
spitfire run kafka/produce-consume.json -c connections.json
```

The gRPC example needs the service's schema: Spitfire reads it from server reflection, or from `.proto` files uploaded on the connection.

`http/shop-checkout.json` posts to a cart and a checkout endpoint, so Spitfire treats it as a test that changes data: it asks for confirmation before every run (`--confirm-writes` on the CLI). Point it at a test environment, never at production.

## Guides

The examples come from these guides, which explain what to measure and the usual mistakes:

- [Load testing without scripting](https://spitfire.tr/en/guides/load-testing-without-scripting)
- [gRPC load testing](https://spitfire.tr/en/guides/grpc-load-testing)
- [Kafka load testing](https://spitfire.tr/en/guides/kafka-load-testing)
- [MQTT load testing](https://spitfire.tr/en/guides/mqtt-load-testing)
- [RabbitMQ load testing](https://spitfire.tr/en/guides/rabbitmq-load-testing)
- [Redis load testing](https://spitfire.tr/en/guides/redis-load-testing)
- [Database load testing](https://spitfire.tr/en/guides/database-load-testing)
- [Performance testing in CI/CD](https://spitfire.tr/en/guides/performance-testing-in-ci-cd)

All guides: https://spitfire.tr/en/guides

## About

Spitfire is commercial software with a free edition (100 virtual users, 15-minute runs, every protocol and CI pass/fail). This repository holds examples only, not Spitfire's source code. The examples are MIT licensed: copy and change them freely.

Questions or an example you'd like to see: open an issue.

## Grafana dashboard

Spitfire's controller serves Prometheus metrics on `/metrics` (scraped with a personal API token):
live runs (virtual users, requests/s, p50/p95/p99 latency, error ratio, per-step p95), runners by
status and location, and the controller's own health.

1. Create a token in Spitfire (user menu, top right → **API tokens** → **Create token**) and add the job in
   [`grafana/prometheus.yml`](grafana/prometheus.yml) to your Prometheus.
2. In Grafana: **Dashboards → New → Import**, upload
   [`grafana/spitfire-dashboard.json`](grafana/spitfire-dashboard.json) and pick that Prometheus.

The **Test** variable filters the run panels by test name. A run stays on the dashboard for two
minutes after it ends, with its final status.

