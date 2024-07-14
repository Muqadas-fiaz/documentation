# Documentation
        Prometheus: Designed for metric collection and storage, Prometheus scrapes metrics from monitored targets at regular intervals. It stores these metrics locally in a time-series database, making it efficient for high-dimensional data collection.
        Grafana: While Grafana itself doesn't collect data, it serves as a visualization tool that connects to data sources like Prometheus. It allows users to create rich, customizable dashboards to visualize and analyze the collected metrics.

    Scalability and Flexibility:
        Prometheus: Built with a focus on scalability and reliability, Prometheus can handle large-scale deployments and dynamic environments. It supports service discovery and flexible querying capabilities.
        Grafana: Grafana complements Prometheus by providing flexible visualization options. It supports various data sources beyond Prometheus, allowing users to create unified dashboards combining data from different monitoring systems if needed.

    Alerting and Notification:
        Prometheus: Includes a powerful alerting mechanism based on Prometheus Query Language (PromQL). It can trigger alerts based on predefined conditions, helping operators to react promptly to issues.
        Grafana: Grafana integrates with Prometheus alerting, allowing users to create alert rules visually and receive notifications through various channels like email, Slack, or other alerting systems.

    Community and Ecosystem:
        Both Prometheus and Grafana have vibrant open-source communities and extensive documentation. This makes it easier to find support, share best practices, and extend functionality through plugins and integrations.

    Monitoring Kubernetes and Cloud-Native Environments:
        Prometheus is particularly well-suited for monitoring cloud-native applications and microservices architectures, including Kubernetes clusters. It supports automatic service discovery and metrics scraping, making it ideal for dynamic environments.
        Grafana's support for Prometheus makes it a preferred choice for visualizing metrics from Kubernetes and other cloud-native platforms, offering pre-built dashboards and plugins tailored to these environments.

    Customizability and Extensibility:
        Both tools are highly customizable. Prometheus allows custom metrics to be defined and collected, while Grafana enables users to create dashboards with flexible layouts, templating, and plugins to extend functionality.

In summary, Prometheus and Grafana together provide a robust solution for monitoring, alerting, and visualizing metrics in modern IT environments. Their combination offers scalability, flexibility, and ease of use, making them popular choices for both small-scale deployments and large-scale enterprise monitoring systems.

Install Prometheus on Ubuntu 20.04
Create Prometheus System User

    Create a system user for Prometheus:

    bash

    sudo useradd \
      --system \
      --no-create-home \
      --shell /bin/false prometheus

Download and Setup Prometheus

    Download Prometheus:

    bash

wget https://github.com/prometheus/prometheus/releases/download/v2.32.1/prometheus-2.32.1.linux-amd64.tar.gz

Extract Prometheus files:

bash

tar -xvf prometheus-2.32.1.linux-amd64.tar.gz

Create necessary directories:

bash

sudo mkdir -p /data /etc/prometheus

Move binaries and configuration files:

bash

cd prometheus-2.32.1.linux-amd64
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml

Set permissions:

bash

sudo chown -R prometheus:prometheus /etc/prometheus/ /data/

Cleanup:

bash

    cd
    rm -rf prometheus*

Configure Prometheus as a Service

    Create systemd unit file for Prometheus:

    bash

sudo vim /etc/systemd/system/prometheus.service

makefile

[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target

Enable and start Prometheus service:

bash

sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus

Verify status:

bash

    sudo systemctl status prometheus

    Access Prometheus:
    Open a browser and go to http://<ip>:9090 where <ip> is your server's IP address.

Install Node Exporter on Ubuntu 20.04
Create Node Exporter System User

    Create a system user for Node Exporter:

    bash

    sudo useradd \
      --system \
      --no-create-home \
      --shell /bin/false node_exporter

Download and Setup Node Exporter

    Download Node Exporter:

    bash

wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz

Extract Node Exporter files:

bash

tar -xvf node_exporter-1.3.1.linux-amd64.tar.gz

Move Node Exporter binary:

bash

sudo mv node_exporter-1.3.1.linux-amd64/node_exporter /usr/local/bin/

Cleanup:

bash

    rm -rf node_exporter*

Configure Node Exporter as a Service

    Create systemd unit file for Node Exporter:

    bash

sudo vim /etc/systemd/system/node_exporter.service

makefile

[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter \
  --collector.logind

[Install]
WantedBy=multi-user.target

Enable and start Node Exporter service:

bash

sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter

Verify status:

bash

    sudo systemctl status node_exporter

Install Grafana on Ubuntu 20.04
Install Grafana

    Install dependencies and add Grafana repository:

    bash

sudo apt-get install -y apt-transport-https software-properties-common
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get -y install grafana

Enable and start Grafana service:

bash

sudo systemctl enable grafana-server
sudo systemctl start grafana-server

Verify status:

bash

    sudo systemctl status grafana-server

    Access Grafana:
    Open a browser and go to http://<ip>:3000. Log in with username admin and password admin. You will be prompted to change the password.

Adding Data Source to Grafana

    Configure Prometheus as a data source:
        Go to Grafana UI -> Configuration -> Data Sources -> Add data source.
        Choose Prometheus, set URL to http://localhost:9090, and click Save & Test.

Importing Grafana Dashboards

    Import Node Exporter Dashboard:
        Go to Grafana UI -> Dashboards -> Import.
        Use the dashboard ID 1860 to import Node Exporter metrics.

This README.md provides step-by-step instructions for installing Prometheus, Node Exporter, and Grafana on Ubuntu 20.04, configuring them as services, and setting up Grafana to visualize metrics from Prometheus. Adjust <ip> and other settings as per your environment.










