# Prometheus and Grafana Configuration Guide for EPICS IOC Host Monitoring

**Author:** Seth Nemesure
**Version:** 1.0
**Status:** Draft

---

# 1. Purpose

This document describes the complete process for configuring Prometheus and Grafana to monitor Linux hosts running EPICS IOC services.

The objectives are:

- Monitor Linux host health
- Monitor IOC service status
- Build reusable Grafana dashboards
- Establish a framework for future EPICS-specific monitoring
- Provide sufficient documentation for recreating the monitoring environment

This document focuses on a monitoring host running:

- Prometheus
- Grafana

and an IOC host running:

- node_exporter
- EPICS IOC services managed by systemd

---

# 2. System Architecture

```
                   +----------------------+
                   |      Grafana         |
                   | Dashboards           |
                   +----------+-----------+
                              |
                        PromQL Queries
                              |
                   +----------v-----------+
                   |     Prometheus       |
                   | Time Series DB       |
                   +----------+-----------+
                              |
                        HTTP Scrape
                              |
              +---------------+----------------+
              |                                |
      +-------v--------+               +-------v--------+
      | demo01         |               | hcoatingdev01 |
      | node_exporter  |               | node_exporter |
      +----------------+               +----------------+
```

Future architecture will include EPICS-specific metrics.

```
                   +----------------------+
                   | EPICS Exporter        |
                   | IOC Metrics           |
                   | Alarm Metrics         |
                   +----------------------+
```

---

# 3. Software Versions

| Component | Version |
|------------|---------|
| CentOS Stream | 8 |
| Prometheus | |
| Grafana | |
| node_exporter | |

---

# 4. Prerequisites

- Prometheus installed
- Grafana installed
- Administrative access
- Network connectivity between Prometheus and IOC hosts
- Port 9100 accessible from Prometheus
- node_exporter access

---

## What is `node_exporter`?

`node_exporter` is a Prometheus exporter that collects operating system and hardware metrics from a Linux host and makes them available to Prometheus over HTTP.

Unlike Prometheus, which stores and queries time-series data, `node_exporter` does not retain any historical information. Instead, it periodically reads information from the local operating system (such as `/proc` and `/sys`) and exposes the current values through a `/metrics` endpoint, typically on TCP port **9100**.

Prometheus periodically scrapes this endpoint, stores the collected metrics, and makes them available for querying and visualization through Grafana.

By default, `node_exporter` provides hundreds of metrics describing the health and performance of the host, including:

* CPU utilization
* Memory utilization
* Disk usage and I/O statistics
* Filesystem utilization
* Network throughput
* System load
* Uptime
* Hardware information

Additional collectors can be enabled to expose metrics from other parts of the operating system. For this project, the **systemd collector** was enabled to publish the status of IOC services managed by `systemd`, allowing Grafana to display the operational status of EPICS IOCs without requiring any modifications to the IOC applications themselves.

`node_exporter` is intentionally lightweight and read-only. It does not control or modify the system it monitors; its sole purpose is to expose system metrics in a format that Prometheus can collect.

---

# 5. Installing node_exporter

On CentOS Stream 8 the package is named:

```bash
sudo dnf install golang-github-prometheus-node-exporter
```

On newer EL9 systems the package name may instead be:

```bash
prometheus-node-exporter
```

---

# 6. Starting node_exporter

Enable the service:

```bash
sudo systemctl enable prometheus-node-exporter
```

Start the service:

```bash
sudo systemctl start prometheus-node-exporter
```

Verify:

```bash
systemctl status prometheus-node-exporter
```

Expected:

```
Active: active (running)
```

Verify metrics are available:

```bash
curl http://localhost:9100/metrics
```

---

# 7. Enabling the systemd Collector

The default node_exporter installation exports operating system metrics.

To monitor IOC services, the systemd collector must be enabled.

Create a systemd override:

```bash
sudo systemctl edit prometheus-node-exporter
```

Example:

```ini
[Service]
ExecStart=
ExecStart=/usr/bin/prometheus-node-exporter \
    --collector.systemd \
    --collector.systemd.unit-include='softioc-.*\.service'
```

Reload:

```bash
sudo systemctl daemon-reload
sudo systemctl restart prometheus-node-exporter
```

Verify:

```bash
curl http://localhost:9100/metrics | grep node_systemd
```
---
The example above enables the **systemd collector** and limits metric collection to services whose names match the regular expression `softioc-.*\.service`. This keeps the exported metrics focused on EPICS IOC services and avoids collecting state information for every systemd unit on the host.

Additional collectors and options can be enabled as monitoring requirements evolve. For example:

- `--collector.systemd.enable-restarts-metrics` — Export service restart counts.
- `--collector.systemd.enable-start-time-metrics` — Export service start times and uptime information.
- `--collector.systemd.enable-task-metrics` — Export task counts for each service.
- `--collector.textfile.directory=<directory>` — Enable the textfile collector for publishing custom metrics generated by scripts or other applications.

These options are not required for the initial host and IOC monitoring described in this guide but provide a straightforward path for expanding the monitoring infrastructure in the future.

---

# 8. Configuring Prometheus
## Overview
Prometheus collects metrics by periodically **scraping** one or more targets over HTTP. Each target is defined in the `prometheus.yml` configuration file as a *scrape job*. A scrape job specifies the host, port, and other parameters Prometheus uses to retrieve metrics from an exporter.

In this example, `node_exporter` is running on the IOC host and exposing metrics on TCP port **9100**. A new scrape job is added to instruct Prometheus to collect these metrics at regular intervals. Once the configuration is reloaded, the metrics become available for querying with PromQL and can be visualized using Grafana dashboards.

Edit:

```
/etc/prometheus/prometheus.yml
```

Example:

```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
       # The label name is added as a label `label_name=<label_value>` to any timeseries scraped from this config.
        labels:
          app: "prometheus"

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100', 'demo01.eic.bnl.gov:9100']

  - job_name: hcoatingdev01
    static_configs:
      - targets: ['hcoatingdev01:9100']
```

Restart Prometheus.

Verify:

```
Status
    Targets
```

Expected:

```
UP
```
## Prometheus Interface
<img width="1405" height="458" alt="Screenshot 2026-07-14 at 10 35 10 AM" src="https://github.com/user-attachments/assets/4e83c1da-c235-4433-b23f-532c0f16a724" />

---

# 9. Learning PromQL

PromQL is the Prometheus Query Language.

Useful starter queries:

Host reachable:

```promql
up
```

Available memory:

```promql
node_memory_MemAvailable_bytes
```

CPU count:

```promql
count(node_cpu_seconds_total{mode="idle"}) by (instance)
```

Load average:

```promql
node_load1
```

IOC service status:

```promql
node_systemd_unit_state
```

---

# 10. Creating the Host Dashboard

A custom host dashboard was created containing:

- CPU Utilization
- Memory Utilization
- Disk Utilization
- Network Traffic
- Load Average
- Uptime

Document for each panel:

- Purpose
- PromQL Query
- Visualization Type
- Thresholds
- Notes

<img width="1322" height="590" alt="Screenshot 2026-07-14 at 10 39 45 AM" src="https://github.com/user-attachments/assets/b5d513b5-f4cd-465c-ac5a-7870930ac5bb" />

<img width="1125" height="1053" alt="Screenshot 2026-07-14 at 10 40 48 AM" src="https://github.com/user-attachments/assets/7febc4f0-ae03-4968-a020-9464bd9727cf" />

---

# 11. Creating the IOC Dashboard

## Goal

Display IOC service status.

IOC services are managed by systemd.

Rather than modifying IOC software, node_exporter collects service state using the systemd collector.

---

## IOC Query

Example:

```promql
node_systemd_unit_state{
    instance="hcoatingdev01:9100",
    name=~"softioc-.*\\.service"
}
```

---

## Query Options

Type:

```
Instant
```

Format:

```
Time Series
```

---

## Grafana Transformation

Transformation:

```
Series to Rows
```

This converts one time series per IOC into a table.

---

## Value Mapping

Map values:

| Value | Display |
|---------|---------|
| 1 | OK |
| 0 | Disabled |

Additional mappings may be added later.

<img width="1128" height="493" alt="Screenshot 2026-07-14 at 10 42 32 AM" src="https://github.com/user-attachments/assets/3b836554-7f9e-4f07-a2cc-6d75cbc6c592" />

---

## Future Improvements

Shorten service names.

Group IOCs by subsystem.

Color-code subsystem health.

---

# 12. Troubleshooting

## node_exporter not running

```
systemctl status prometheus-node-exporter
```

---

## Metrics unavailable

```
curl http://localhost:9100/metrics
```

---

## Prometheus cannot scrape target

Verify:

```
Status
    Targets
```

---

## No data in Grafana

Verify:

- Prometheus datasource
- Target is UP
- Query executes in Prometheus
- Time range
- Instant query vs Range query

---

## Only one service displayed

Cause:

Grafana displays one series.

Resolution:

Transformation:

```
Series to Rows
```

---

## node_systemd metrics unavailable

Verify collector enabled.

```
curl http://localhost:9100/metrics | grep node_systemd
```

---

# 13. Future Monitoring Roadmap

## Stage 1 (Completed)

Host monitoring

- CPU
- Memory
- Disk
- Network
- Load
- Uptime

---

## Stage 2 (Completed)

IOC Service Monitoring

- systemd collector
- IOC status dashboard

---

## Stage 3

IOC Health

- Heartbeat PV
- IOC uptime
- Last update timestamp

---

## Stage 4

EPICS Monitoring

- Connected PV count
- Gateway status
- IOC connection health
- PV freshness

---

## Stage 5

Application Monitoring

Examples:

- notif2epics
- FastAPI services
- Alarm services

Metrics:

- Requests/sec
- Error rate
- Latency
- Queue depth
- Publish rate

---

# 14. Appendix A - Useful PromQL Queries

```
up
```

```
node_load1
```

```
node_memory_MemAvailable_bytes
```

```
node_memory_MemTotal_bytes
```

```
node_filesystem_avail_bytes
```

```
node_systemd_unit_state
```

---

# 15. Appendix B - Useful Linux Commands

Install:

```bash
dnf install golang-github-prometheus-node-exporter
```

Enable:

```bash
systemctl enable prometheus-node-exporter
```

Start:

```bash
systemctl start prometheus-node-exporter
```

Status:

```bash
systemctl status prometheus-node-exporter
```

Metrics:

```bash
curl http://localhost:9100/metrics
```

Verify systemd collector:

```bash
curl http://localhost:9100/metrics | grep node_systemd
```

---

# 16. Appendix C - Grafana Tips

Useful transformations:

- Series to Rows
- Labels to Fields
- Reduce
- Organize Fields

Useful panel types:

- Time Series
- Table
- Stat
- Gauge
- Status History

Use **Instant** queries for current system state.

Use **Range** queries for historical trends.
