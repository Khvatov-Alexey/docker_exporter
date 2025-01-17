# docker_exporter

This app exports metrics about a Docker installation and any running containers to the [Prometheus](https://prometheus.io) metrics and monitoring system.

Note: *Relatively https://github.com/prometheus-net/docker_exporter , docker image name changed here*

![](Screenshot.png)

# Quick start

Given a Docker installation with default configuration (listening on Unix pipe):

1. Start the exporter by executing `docker run --name docker_exporter --detach --restart unless-stopped --volume "/var/run/docker.sock:/var/run/docker.sock" --publish 9417:9417 fratiq/docker_exporter`
2. Navigate to http://localhost:9417/metrics to explore the available metrics.
3. Register `localhost:9417` in your Prometheus configuration as a scrape target.
4. If using Grafana, [install the template dashboard](https://grafana.com/grafana/dashboards/11467)

Example Prometheus scrape configuration:

```yaml
  - job_name: 'docker_exporter'
    static_configs:
      - targets:
        - localhost:9417
```

# What metrics are exported?

Basic state (running or not), lifecycle (unexpected restart count) and resource usage metrics for each container, labeled by container name. Metrics are collected from the same instance of Docker that is running the exporter app.

To see the detailed list and documentation on each metric type, open the `/metrics` URL of the running app and read the output.

If you want more metrics exposed, file an issue and describe your exact needs.

# Compatibility

Detailed testing has not been done to establish the limits of compatibility. No exotic APIs are used, so incompatibilities are not expected.

# Authentication

The exporter does not currently support authentication between the exporter and Docker.

# Remote collection

The exporter must be executed by the same instance of Docker that it is monitoring.

# Doesn't Docker itself already export Prometheus metrics?

If you enable the experimental features mode in Docker, [it does expose some very basic metrics on the Docker engine](https://docs.docker.com/config/thirdparty/prometheus/). However, those metrics form a minimal set that is not very informative in real world use cases.

# Version upgrades

To upgrade to a new version:

1. Execute `docker rm --force docker_exporter` to stop the existing instance.
2. Execute `docker pull fratiq/docker_exporter` to download the new version.
3. Execute the `docker run` command from the quick start to start the new version.

# Troubleshooting

Error messages can typically be found in `docker logs docker_exporter`.

You can append the `--verbose` parameter to the end of the `docker run` command to show more detailed information in the logs.

Docker can occasionally be very slow or even hang due to ongoing Docker operations blocking the information-gathering by this app. The app will try its best to work around such slowness:

1. If probing Docker takes too long (more than 20 seconds), the probe is continued in the background and stale data is returned to Prometheus. You can observe the `docker_probe_successfully_completed_time` metric to identify whether you are seeing stale data.
2. If probing Docker in the background still takes too long (more than a few minutes), the probe is aborted and the next scrape will try again from the beginning.
