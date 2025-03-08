# Prometheus Monitoring System Setup Guide

This README provides detailed instructions for setting up Prometheus monitoring system on a Linux server, along with frequently asked questions about Prometheus for interviews.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation Steps](#installation-steps)
  - [Setup Prometheus Binaries](#setup-prometheus-binaries)
  - [Setup Prometheus Configuration](#setup-prometheus-configuration)
  - [Setup Prometheus Service](#setup-prometheus-service)
  - [Access Prometheus Web UI](#access-prometheus-web-ui)
- [Node Exporter Setup](#node-exporter-setup)
- [FAQ](#faq)
  - [What is Prometheus?](#what-is-prometheus)
  - [Prometheus vs Node Exporter](#prometheus-vs-node-exporter)
  - [Common Interview Questions](#common-interview-questions)

## Prerequisites

Before you begin, ensure that:

- You have sudo access to the Linux server
- The server has internet access for downloading the Prometheus binary
- Firewall rules are configured to allow access to Prometheus port 9090

## Installation Steps

### Setup Prometheus Binaries

1. Update the package repositories:

```bash
sudo yum update -y
```

2. Download the Prometheus binary:

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.22.0/prometheus-2.53.3.linux-amd64.tar.gz
tar -xvf prometheus-2.22.0.linux-amd64.tar.gz
mv prometheus-2.22.0.linux-amd64 prometheus-files
```

3. Create a Prometheus user and required directories:

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

4. Copy Prometheus binaries and set permissions:

```bash
sudo cp prometheus-files/prometheus /usr/local/bin/
sudo cp prometheus-files/promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

5. Copy console files and set permissions:

```bash
sudo cp -r prometheus-files/consoles /etc/prometheus
sudo cp -r prometheus-files/console_libraries /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```

### Setup Prometheus Configuration

1. Create the Prometheus configuration file:

```bash
sudo vi /etc/prometheus/prometheus.yml
```

2. Add the following content to the configuration file:

```yaml
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']
```

3. Set correct ownership for the configuration file:

```bash
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

### Setup Prometheus Service

1. Create a systemd service file for Prometheus:

```bash
sudo vi /etc/systemd/system/prometheus.service
```

2. Add the following content to the service file:

```ini
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

3. Reload systemd, start and check Prometheus service:

```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl status prometheus
```

4. Enable Prometheus to start at boot:

```bash
sudo systemctl enable prometheus
```

### Access Prometheus Web UI

Once Prometheus is running, you can access the web interface at:

```
http://<prometheus-ip>:9090/graph
```

## Node Exporter Setup

To monitor additional servers with Prometheus, you need to install Node Exporter on those servers. Follow these steps:

1. Download and extract Node Exporter:

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
tar -xvf node_exporter-1.3.1.linux-amd64.tar.gz
```

2. Move the binary to the correct location:

```bash
sudo cp node_exporter-1.3.1.linux-amd64/node_exporter /usr/local/bin/
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

3. Create a systemd service for Node Exporter:

```bash
sudo vi /etc/systemd/system/node_exporter.service
```

4. Add the following content:

```ini
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

5. Start Node Exporter:

```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```

6. Update Prometheus configuration to scrape the Node Exporter metrics:

```bash
sudo vi /etc/prometheus/prometheus.yml
```

7. Add a new job to the scrape_configs section:

```yaml
  - job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['<node-exporter-ip>:9100']
```

8. Restart Prometheus to apply changes:

```bash
sudo systemctl restart prometheus
```

## FAQ

### What is Prometheus?

Prometheus is an open-source systems monitoring and alerting toolkit originally built at SoundCloud. It features:

- A multi-dimensional data model with time series data identified by metric name and key/value pairs
- PromQL, a flexible query language to leverage this dimensionality
- No reliance on distributed storage; single server nodes are autonomous
- Time series collection happens via a pull model over HTTP
- Pushing time series is supported via an intermediary gateway
- Targets are discovered via service discovery or static configuration
- Multiple modes of graphing and dashboarding support

### Prometheus vs Node Exporter

| Prometheus | Node Exporter |
|------------|---------------|
| A complete monitoring and alerting system | A single exporter for hardware and OS metrics |
| Collects metrics from various exporters | Exposes system metrics for Prometheus to scrape |
| Stores, processes, and analyzes metrics | Does not store data, only exports it |
| Provides a query language and alerting | Has no query capabilities |
| Serves as the central monitoring system | Acts as an agent on monitored hosts |
| Runs on a dedicated server or VM | Runs on every server you want to monitor |
| Handles data visualization and alerting | Only exposes metrics, no visualization |

### Common Interview Questions

#### 1. What is the Prometheus architecture?

Prometheus consists of several components:
- The main Prometheus server which scrapes and stores time series data
- Client libraries for instrumenting application code
- A push gateway for supporting short-lived jobs
- Exporters for services like HAProxy, StatsD, Graphite, etc.
- An alertmanager to handle alerts
- Various support tools

#### 2. What are the key features of Prometheus?

- Multi-dimensional data model
- Flexible query language
- No distributed storage dependency
- Pull model for data collection
- Service discovery for targets
- Visualizations and dashboarding

#### 3. What is the difference between push and pull model in monitoring?

Prometheus uses a pull model, where it periodically scrapes metrics from configured targets. In a push model (like Graphite), the monitored targets push their metrics to a central server. 

The pull model is advantageous because:
- Prometheus can determine if a target is down
- Centralized control over scrape frequency
- No need for firewall configuration to allow outbound connections from targets

#### 4. What is PromQL?

PromQL (Prometheus Query Language) is Prometheus's functional query language that lets users select and aggregate time series data in real time. It allows you to:
- Filter time series data based on labels
- Apply mathematical operations to metrics
- Create alerts and visualizations
- Perform rate calculations, predictions, and other complex analytics

#### 5. What are exporters in Prometheus?

Exporters are tools that collect metrics from a third-party system and expose them in a format Prometheus can scrape. Examples include:
- Node Exporter (for hardware and OS metrics)
- MySQL Exporter (for MySQL database metrics)
- Blackbox Exporter (for probing endpoints)
- JMX Exporter (for Java application metrics)

#### 6. How does Prometheus handle high availability?

Prometheus servers are designed to operate independently, not as a distributed system. For high availability:
- Run multiple replicated Prometheus servers
- Use external storage solutions with remote write/read APIs
- Use Thanos or Cortex for long-term storage and high availability

#### 7. What are the limitations of Prometheus?

- Not suitable for storing event logs or individual events
- Not ideal for 100% accuracy (uses a pull model with potential gaps)
- Can struggle with very high cardinality data
- Built for reliability, not for long-term storage of historical data

#### 8. What is service discovery in Prometheus?

Service discovery allows Prometheus to automatically find and monitor targets without manual configuration. Prometheus supports multiple service discovery mechanisms:
- File-based
- Kubernetes
- Consul
- AWS EC2
- Azure
- And many others

#### 9. How do you configure alerting in Prometheus?

Alerting in Prometheus is configured through:
1. Alert rules defined in Prometheus configuration
2. AlertManager for handling, grouping, and routing alerts
3. Receivers for notification channels (email, Slack, PagerDuty, etc.)

#### 10. What is the role of Grafana in a Prometheus setup?

While Prometheus has basic visualization capabilities, Grafana is often used alongside it to:
- Create more advanced and visually appealing dashboards
- Combine data from multiple data sources
- Provide better sharing and user management features
- Enable annotations and more interactive visualizations

#### 11. How do you scale Prometheus for large environments?

Scaling approaches include:
- Functional sharding (different Prometheus servers for different services)
- Hierarchical federation (higher-level Prometheus instances scrape lower-level ones)
- Using remote storage for long-term data retention
- Implementing solutions like Thanos or Cortex for global query view
