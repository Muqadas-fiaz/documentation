# Documentation
Overview

Prometheus and Grafana are popular tools used together for effective monitoring and visualization in IT environments. Here are several reasons why you might want to use Prometheus with Grafana:
Prometheus: Powerful Time-Series Data Handling

    Efficient Storage: Prometheus is designed for storing time-series data, which makes it highly efficient for handling large volumes of monitoring data.
    Flexible Query Language (PromQL): Prometheus Query Language (PromQL) allows for complex and flexible queries to extract meaningful insights from the collected data.
    Multi-dimensional Data Model: Prometheus uses a multi-dimensional data model with key/value pairs, enabling detailed and customizable metrics.

Grafana: Visualization and Dashboarding

    Rich Visualization Options: Grafana provides a wide range of visualization options, including graphs, heatmaps, and pie charts, which help in creating insightful dashboards.
    Customizable Dashboards: Grafana allows for the creation of highly customizable and interactive dashboards, making it easier to monitor and analyze metrics in real-time.
    Alerting: Grafana includes built-in alerting features, allowing you to set up alerts based on your monitoring data and receive notifications when certain conditions are met.

Scalability and Flexibility

    Scalable Architecture: Both Prometheus and Grafana are designed to be scalable, allowing you to monitor and visualize data from small setups to large-scale, distributed environments.
    Integration with Other Tools: Grafana can integrate with various data sources beyond Prometheus, such as Elasticsearch, InfluxDB, and MySQL, providing a unified view of different types of data.

Open Source and Community Support

    Active Community: Both Prometheus and Grafana are open-source projects with active communities, which means regular updates, improvements, and extensive community support.
    Cost-Effective: Being open-source, these tools are cost-effective, making them a preferred choice for many organizations.

Ease of Deployment and Use

    Deployment: Both tools are relatively easy to deploy and configure, with extensive documentation and community resources available.
    User-Friendly Interface: Grafana’s user-friendly interface makes it easy for users to create and manage dashboards without needing deep technical expertise.

Pre-Built Integrations and Exporters

    Prometheus Exporters: There are numerous exporters available for Prometheus that allow it to scrape metrics from various sources, such as operating systems, databases, and hardware.
    Grafana Plugins: Grafana supports a wide range of plugins that extend its functionality, providing additional visualization options and data source integrations.

Using Prometheus with Grafana provides a comprehensive monitoring solution that is powerful, flexible, and user-friendly, helping you to effectively monitor and visualize your IT infrastructure and applications.
Install Prometheus on Ubuntu 20.04
Create a Dedicated Linux User for Prometheus

Having individual users for each service serves two main purposes:

    It is a security measure to reduce the impact in case of an incident with the service.
    It simplifies administration as it becomes easier to track down what resources belong to which service.

To create a system user or system account, run the following command:

bash

sudo useradd --system --no-create-home --shell /bin/false prometheus

    --system: Will create a system account.
    --no-create-home: We don't need a home directory for Prometheus or any other system accounts in our case.
    --shell /bin/false: It prevents logging in as a Prometheus user.
    prometheus: Will create Prometheus user and a group with the exact same name.

Download and Extract Prometheus

Let's check the latest version of Prometheus from the download page. You can use the curl or wget command to download Prometheus.

bash

wget https://github.com/prometheus/prometheus/releases/download/v2.32.1/prometheus-2.32.1.linux-amd64.tar.gz

Then, extract all Prometheus files from the archive.

bash

tar -xvf prometheus-2.32.1.linux-amd64.tar.gz

Create necessary directories.

bash

sudo mkdir -p /data /etc/prometheus

Move the binaries and configuration files.

bash

cd prometheus-2.32.1.linux-amd64
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml

Set correct ownership for the directories.

bash

sudo chown -R prometheus:prometheus /etc/prometheus/ /data/

Verify that you can execute the Prometheus binary.

bash

prometheus --version
prometheus --help

Create Systemd Service for Prometheus

Create a systemd unit configuration file.

bash

sudo vim /etc/systemd/system/prometheus.service

Add the following content:

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

Enable and start Prometheus.

bash

sudo systemctl enable prometheus
sudo systemctl start prometheus

Check the status of Prometheus.

bash

sudo systemctl status prometheus

If you encounter any issues, use the journalctl command to search for errors.

bash

journalctl -u prometheus -f --no-pager

Access Prometheus via browser using http://<your_server_ip>:9090.
Install Node Exporter on Ubuntu 20.04
Create a System User for Node Exporter

bash

sudo useradd --system --no-create-home --shell /bin/false node_exporter

Download and Extract Node Exporter

bash

wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
tar -xvf node_exporter-1.3.1.linux-amd64.tar.gz
sudo mv node_exporter-1.3.1.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter*

Verify that you can run the binary.

bash

node_exporter --version
node_exporter --help

Create Systemd Service for Node Exporter

Create a systemd unit configuration file.

bash

sudo vim /etc/systemd/system/node_exporter.service

Add the following content:

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
ExecStart=/usr/local/bin/node_exporter --collector.logind

[Install]
WantedBy=multi-user.target

Enable and start Node Exporter.

bash

sudo systemctl enable node_exporter
sudo systemctl start node_exporter

Check the status of Node Exporter.

bash

sudo systemctl status node_exporter

If you have any issues, check logs with journalctl.

bash

journalctl -u node_exporter -f --no-pager

Configure Prometheus to Scrape Node Exporter

Add a static target to Prometheus configuration.

bash

sudo vim /etc/prometheus/prometheus.yml

Add the following content:

yaml

- job_name: node_export
  static_configs:
    - targets: ["localhost:9100"]

Check the config is valid.

bash

promtool check config /etc/prometheus/prometheus.yml

Reload the Prometheus config.

bash

curl -X POST http://localhost:9090/-/reload

Check the targets section http://<ip>:9090/targets.
Install Grafana on Ubuntu 20.04
Add Grafana Repository and Install

bash

sudo apt-get install -y apt-transport-https software-properties-common
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get -y install grafana

Enable and start Grafana.

bash

sudo systemctl enable grafana-server
sudo systemctl start grafana-server

Check the status of Grafana.

bash

sudo systemctl status grafana-server

Access Grafana via browser using http://<your_server_ip>:3000 with default credentials (username: admin, password: admin). Change the password upon first login.
Add Prometheus as a Data Source in Grafana

Add a data source and select Prometheus. For the URL, enter http://localhost:9090 and click Save and test. Alternatively, create a data source as code.

bash

sudo vim /etc/grafana/provisioning/datasources/datasources.yaml

Add the following content:

yaml

apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    url: http://localhost:9090
    isDefault: true

Restart Grafana to reload the config.

bash

sudo systemctl restart grafana-server

Refresh the Grafana page to see the Prometheus data source.
Create and Import Dashboards in Grafana

To create a simple graph, use the scrape_duration_seconds metric. To import an open-source dashboard, search for node exporter on the Grafana website and use the dashboard ID (e.g., 1860).

This guide provides a comprehensive setup for Prometheus and Grafana on Ubuntu 20.04. For more advanced configurations and customizations, refer to the official documentation and community resources.

