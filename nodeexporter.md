Perfect 👍. To monitor your Ubuntu server itself, you’ll need **Node Exporter** (it exposes Linux system metrics to Prometheus).

---

# **Install Node Exporter on Ubuntu**

### 1. Create user

```bash
sudo useradd --no-create-home --shell /bin/false node_exporter
```

### 2. Download Node Exporter

```bash
cd /tmp
curl -s https://api.github.com/repos/prometheus/node_exporter/releases/latest \
| grep browser_download_url \
| grep linux-amd64.tar.gz \
| cut -d '"' -f 4 \
| wget -i -
```

```bash
tar -xvzf node_exporter-*.linux-amd64.tar.gz
cd node_exporter-*.linux-amd64
```

### 3. Move binary

```bash
sudo cp node_exporter /usr/local/bin/
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

### 4. Create systemd service

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Paste:

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

### 5. Start Node Exporter

```bash
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
sudo systemctl status node_exporter
```

Node Exporter runs on 👉 `http://<server-ip>:9100/metrics`

---

# **6. Configure Prometheus to scrape Node Exporter**

Edit Prometheus config:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Add a new job under `scrape_configs`:

```yaml
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```

Save → Restart Prometheus:

```bash
sudo systemctl restart prometheus
```

---

# **7. Import Grafana Dashboard**

1. Open Grafana → **Dashboards → Import**
2. Use prebuilt **Node Exporter Full Dashboard (ID: 1860)**
   👉 [https://grafana.com/grafana/dashboards/1860](https://grafana.com/grafana/dashboards/1860)
3. Select Prometheus as data source → Import.

✅ Now you’ll see live CPU, memory, disk, and network metrics for your Ubuntu server in Grafana.

---

Do you want me to also show you **how to install Alertmanager** so you can get email/Slack alerts when CPU/memory is high?
