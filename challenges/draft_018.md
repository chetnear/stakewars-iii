# Stake Wars: Episode III. Challenge  017

- Published on: 2022-08-26
- Submitted by: Everstake
- Rewards: 15 DNP

## Challenge submission

For submission, please fill out the [form](https://forms.gle/1MS9Jvhvq9YWbbwk7)  

Make sure to include your Grafana IP & Port `http://<IP>:3000/`.` 

Also, the name & password of the user with the “**Viewer”** role.  

**Note:** points can be deducted for partial completion

## Acceptance Criteria

1. Create a **Dashboard** for hardware
2. Create a **Dashboard** for software 

**Note:** sections 1 & 2 can be combined if desired

## Steps

### 1. Download and install Grafana OSS

- Get latest release Grafana OSS from [https://grafana.com/grafana/download?platform=linux](https://grafana.com/grafana/download?platform=linux)

```bash
wget https://dl.grafana.com/oss/release/grafana_9.0.5_amd64.deb
sudo dpkg -i grafana_9.0.5_amd64.deb
```

- Start the grafana-server

To start the service and verify that the service has started:

```bash
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl status grafana-server
```

### 2. ****Download and install Prometheus and Node Exporter****

- Create a user for Prometheus and Node Exporter

**Note: Prometheus** - is simply a database with values for metrics. Quite often, Prometheus is configured in conjunction with Grafana, which allows you to visualize metrics. **Node Exporter** - is necessary for collecting Linux system indicators, such as CPU load, monitoring file systems, disks, network statistics (and others).

```bash
sudo useradd --no-create-home --shell /usr/sbin/nologin prometheus
sudo useradd --no-create-home --shell /bin/false node_exporter
```

- Create folders

```bash
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```

- Set the ownership of these directories

```bash
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

- Download the latest version of **Node Exporter** [https://prometheus.io/download/](https://prometheus.io/download/):

```bash
wget https://github.com/prometheus/node_exporter/releases/download/<LATEST VERSION>/node_exporter-<LATEST VERSION>linux-amd64.tar.gz

tar xvf node_exporter-<LATEST VERSION>linux-amd64.tar.gz

sudo cp node_exporter-<LATEST VERSION>linux-amd64/node_exporter /usr/local/bin
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

- Create the Systemd service file node_exporter

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

```bash
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

```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl status node_exporter
```

- Download the latest version of ****Prometheus**** [https://prometheus.io/download/](https://prometheus.io/download/):

```bash
wget https://github.com/prometheus/prometheus/releases/download/<LATEST VERSION>/prometheus-<LATEST VERSION>linux-amd64.tar.gz

tar xfz prometheus-<LATEST VERSION>linux-amd64.tar.gz
cd prometheus-<LATEST VERSION>linux-amd64/

sudo cp ./prometheus /usr/local/bin/
sudo cp ./promtool /usr/local/bin/

sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool

sudo cp -r ./consoles /etc/prometheus
sudo cp -r ./console_libraries /etc/prometheus

sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```

- Configure ****Prometheus****

```bash
sudo nano /etc/prometheus/prometheus.yml
```

```bash
global:
  scrape_interval:     15s
  evaluation_interval: 15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: 'near_node'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090'] # example - targets: ['99.999.99.99:3030']
```

```bash
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

- Create the Systemd service file prometheus

```bash
sudo nano /etc/systemd/system/prometheus.service
```

```bash
[Unit]
Description=Prometheus Monitoring
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
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl status prometheus
```

### 3. Sign in to Grafana

To sign in to Grafana for the first time:

1. Open your web browser and go to `http://<IP>:3000/`. The default HTTP port that Grafana listens to is 3000 unless you have configured a different port.
2. On the sign-in page, enter admin for the username and password.
3. Click Sign in.

If successful, you will see a prompt to change the password.

1. Click OK on the prompt and change your password.

**Note:** We strongly recommend that you change the default administrator password. We recommend no less than 14 characters including numbers, symbols, uppercase, and lowercase letters. 

- Create a **new view-only user**

Log in under admin and go to:

1. Server Admin – Users
2. Click “New user”
3. Fill out he fields and Click “Create user”
4. Configuration – Users. Check “Role” in the newly created user. There should be “Viewer” permissions. 

**Note:** Create a strong password. We recommend no less than 14 characters including numbers, symbols, uppercase, and lowercase letters. 

- Create **Data Sources / Prometheus**
1. Click the configuration gear icon, then Data Source
2. Select prometheus
3. Set URL to http://localhost:9090
4. Click **Save & Test**

If everything is fine, then you will see a message.

- Add **Dashboard**

## Tasks

#### 1. Create NEAR general dashboard. 
You can use **Screenshot1**, **Screenshot2**, **Screenshot3** as a reference. 

![GrafanaScreenshot1](https://user-images.githubusercontent.com/68015865/187438878-e0490c9d-545f-491a-996b-f4dc64aa6fbc.png)
![GrafanaScreenshot2](https://user-images.githubusercontent.com/68015865/187438934-51f0af65-1dce-436a-9c32-536cfcbe29c9.png)
![GrafanaScreenshot3](https://user-images.githubusercontent.com/68015865/187438956-62e28a4c-d0bb-421e-840b-f3387904241e.png)


> **Block Height** - near_block_height_head  
> **Total Transactions** - near_transaction_processed_successfully_total   
> **Block Processed** - rate(near_block_processed_total[$__rate_interval])   
> **Blocks Per Minute** - near_blocks_per_minute  
> **Validators** - near_is_validator 
>                - near_validator_active_total  
> **Chunk** - histogram_quantile(0.95, sum(rate(near_chunk_tgas_used_hist_bucket[$__rate_interval])) by (le))   
> **Block Processing** - rate(near_block_processing_time_count[$__rate_interval])  
> **Transactions Pool Entries** - near_transaction_pool_entries  
> **Blocks Per Minute** - near_blocks_per_minute   
> **Processed Total Transactions** - rate(near_transaction_processed_total[$__rate_interval])   
> **Processed Successfully Transactions** - rate(near_transaction_processed_successfully_total[$__rate_interval])  
> **Reachable Peers** - near_peer_reachable  
> **Chunk Tgas Used** - near_chunk_tgas_used    
> **Block Processing Time Count** - rate(near_block_processing_time_count[$__rate_interval])    
 
#### 2. Create NEAR validator dashboard 
 
> **Node is validator**  
> **Missed blocks**   
> **Percent of produced blocks**   
> **Missed chunks**   
> **Percent of produced chunks**  
> **Connected peers**   
> **Total validator stake**   

#### 3. Create or reconfigure the [existing](https://grafana.com/grafana/dashboards) dashboard for monitoring **CPU, RAM, file systems, disks, and network statistics**. 

**Note:** to check if **Prometheus** is receiving node metrics, you can use`http://<IP>:9090/graph` in the Web Browser. Alternatively, you can use the Dashboard itself.  When the name is entered, the metric responses will be displayed. **screenshot4**
![near_metrics_web](https://user-images.githubusercontent.com/68015865/187439030-ee995f46-976e-4cd3-9b59-7e0c69e66a8f.png)

## **Update log**

Updated 2022-08-26: Creation
