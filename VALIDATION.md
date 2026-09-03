# Validation and PromQL

## Checks against the running lab

Inspected on 3 September 2026. This is a point-in-time record, not an ongoing uptime guarantee. VM addresses and machine-specific labels are omitted.

| Check | Result |
| --- | --- |
| Grafana `GET /api/health` | HTTP 200; database `ok`; version 13.1.2 |
| Prometheus `GET /-/ready` | HTTP 200; ready |
| Prometheus `GET /api/v1/status/buildinfo` | Version 3.13.2 |
| Prometheus target | `up`; no scrape error |
| Node Exporter target | `up`; no scrape error |
| cAdvisor target | `up`; no scrape error |

The active configuration was read through `/api/v1/status/config`; target health came from `/api/v1/targets`. Queries below were executed through `/api/v1/query`.

| Executed query | Observed result |
| --- | --- |
| `count by (job) (up)` | One target series each for `prometheus`, `node-exporter`, and `cadvisor` |
| `count(node_cpu_seconds_total)` | 16 series |
| `count(node_memory_MemAvailable_bytes)` | 1 series |
| `count(container_cpu_usage_seconds_total)` | 51 series |
| `count(container_memory_working_set_bytes)` | 52 series |

`count(up)` counts target series; it does not prove their health. The target-health results above separately confirmed successful scrapes. Container series counts do not represent a unique number of application containers; cAdvisor can expose several cgroups and resource scopes.

## Example queries for further exploration

These are suggested review queries, not exported Grafana panel definitions. Their underlying metric families were observed; the expressions below were not tested during this inspection.

Target availability (`1` means the latest scrape succeeded):

```promql
up
```

Host CPU utilization, averaged across CPUs:

```promql
100 * (1 - avg by (instance) (rate(node_cpu_seconds_total{job="node-exporter",mode="idle"}[5m])))
```

Available host memory in bytes:

```promql
node_memory_MemAvailable_bytes{job="node-exporter"}
```

Container CPU usage rate in CPU cores, retaining individual series:

```promql
rate(container_cpu_usage_seconds_total{job="cadvisor"}[5m])
```

Container working-set memory in bytes:

```promql
container_memory_working_set_bytes{job="cadvisor"}
```

Inspect labels before filtering or aggregating cAdvisor series to avoid counting parent and child cgroups together.

## Not yet verified

Original startup commands, exporter versions, and clean-environment deployment. The Monitoring folder showed zero Grafana alert rules; other alerting configurations have not been audited.

## Grafana dashboard validation

After authenticated access was available, the default `prometheus` data source was observed pointing to `http://prometheus:9090`. The Monitoring folder contained **Node Exporter Dashboard EN 20201010-StarsL.cn** and **VM and Containers**. Both were exported through Grafana's export UI in external-sharing mode.

All eight original expressions from **VM and Containers** were queried against the live Prometheus API. For this check, Grafana's `$__rate_interval` macro was replaced with `5m`.

| Panel | Result series |
| --- | --- |
| Scrape targets up | 1; dashboard displayed 3 healthy targets |
| VM uptime | 1 |
| VM memory used | 1 |
| Root disk used | 1 |
| VM CPU usage | 1 |
| Container CPU usage | 4 |
| Container memory working set | 4 |
| VM network receive | 6 |

These results validate the original expressions against the existing lab. Public exports replace the original `origin_prometheus` value with `lab`, matching the published Prometheus configuration. They have not been imported into a fresh Grafana instance during this review.
