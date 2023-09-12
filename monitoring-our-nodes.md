# Module 6. Monitoring the nodes

The Cardano node provides crucial metrics about itself and the network, making them available for Prometheus and EKG. This tutorial explains how to set up a well-designed dashboard to monitor your nodes.

## Prometheus and node exporter

A reasonable monitoring setup looks as follows, with a dedicated monitoring server responsible for scraping and collecting data from all the nodes, which makes it accessible for a Grafana dashboard:

<figure><img src=".gitbook/assets/monitoring-cardano-node.png" alt=""><figcaption></figcaption></figure>

## Configuring the Cardano node to export metrics

First, configure your nodes to export metrics to Prometheus:&#x20;

* Turn on the log metrics setting its value to 'true'
* Set the listening port for `hasPrometheus`. The default value is 127.0.0.1. Change the value to 0.0.0.0 so that it can accept connections from the monitoring server. You can use any port you want, the example goes with the default 12798:

```
{
...
  "TurnOnLogMetrics": true,
...
 "hasPrometheus": [
  "0.0.0.0",
  12798
  ],
...
}
```

## Installing the Prometheus node exporter (optional)

The Prometheus node exporter can be useful as well. To install the node exporter, see the documentation below:&#x20;

* [https://prometheus.io/docs/guides/node-exporter/#monitoring-linux-host-metrics-with-the-node-exporter](https://prometheus.io/docs/guides/node-exporter/#monitoring-linux-host-metrics-with-the-node-exporter)

```
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
tar xfvz node_exporter-1.5.0.linux-amd64.tar.gz
```

* Copy the `node_exporter` executable to `/usr/local/bin/`:

```bash
sudo cp node_exporter-1.5.0.linux-amd64/node_exporter /usr/local/bin/
```

Configure the node exporter as a `systemd` unit:

```
sudo nano /etc/systemd/system/node_exporter.service
```

```yaml
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=clr
Type=simple
ExecStart=/usr/local/bin/node_exporter
KillSignal=SIGTERM
RestartKillSignal=SIGTERM
TimeoutStopSec=30s
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

* Start the node exporter:

```
sudo systemctl daemon-reload
sudo systemctl start node_exporter
```

* Update firewall rules to allow connections to port 12798 (to access your cardano-node metrics) and 9100 (the default port for node-exporter).

## Installing Prometheus on the monitoring server

* [https://prometheus.io/download/](https://prometheus.io/download/)
* Configure Prometheus:

```
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
   - job_name: 'CARDANO-NODE' # To scrape data from the Cardano node
     scrape_interval: 5s
     static_configs:
       - targets: ['x.x.x.x:12798']
       - targets: ['y.y.y.y:12798']
       - targets: ['z.z.z.z:12798']
   - job_name: 'EXPORTER' # To scrape data from a node exporter to monitor your Linux host metrics.
     scrape_interval: 5s
     static_configs:
       - targets: ['x.x.x.x:9100']
       - targets: ['y.y.y.y:9100']
       - targets: ['z.z.z.z:9100']
```

* Run Prometheus:

```
./prometheus --config.file=prometheus.yml
```

## Setting up a Grafana dashboard

There are two options:

* [Installing and running Grafana locally (ie, in the monitoring server)](https://grafana.com/get/?plcmt=top-nav\&cta=downloads\&tab=self-managed)
* [Using the cloud version](https://grafana.com/get/?plcmt=top-nav\&cta=downloads)

In either case:&#x20;

* Configure Grafana: [https://prometheus.io/docs/visualization/grafana/](https://prometheus.io/docs/visualization/grafana/)

