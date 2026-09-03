# Configuration notes

## Source of this export

`prometheus.yml` is a concise reconstruction from the running Prometheus configuration API. It preserves the three scrape jobs, collection interval, timeout, evaluation interval, targets, and metric paths. Prometheus-generated defaults are omitted, and a machine-specific label value is replaced with `lab`.

The live effective configuration also reported 15-day retention. The original deployment settings that establish retention have not been exported.

## Network assumptions

- `localhost:9090` refers to Prometheus's own network namespace.
- `host.docker.internal:9100` must resolve from the Prometheus runtime and reach Node Exporter. The existing environment supplies this resolution; the source configuration for that mapping remains to be exported.
- `cadvisor:8080` must resolve from the Prometheus runtime.
- The observed cAdvisor metrics path is `/cadvisor/metrics`. Preserve it when reproducing this lab unless the exporter is configured differently.

Internal service names suggest a container-network setup, but the original Compose file, container options, volumes, and host mappings have not been inspected. No replacement deployment file is presented as the user's original work.

## Completing the reproducible project

1. Export the actual service definitions or Compose files from the VM.
2. Replace secrets with environment-variable references and provide a value-free example file.
3. Import the included Grafana dashboards and adapt the documented data-source connection to the new runtime network.
4. Record exact component versions, mount points, network settings, and startup commands.
5. Validate configuration and startup in a separate environment, then add tested setup instructions.

The public repository does not include the VM disk, credentials, Grafana database, Prometheus time-series database, or raw host logs.

## Grafana connection

The running Grafana instance has a default Prometheus data source named `prometheus`, pointing to `http://prometheus:9090`. Dashboard definitions were exported from the authenticated UI. The provisioning example in `prometheus-datasource.example.yml` was written from this observed connection; it is not an export of the VM's original provisioning file.
