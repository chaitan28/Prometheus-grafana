## **1. Update your system**

```bash
sudo apt update && sudo apt upgrade -y
```

---

## **2. Install Prometheus**

### Create Prometheus user and directories

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

### Download Prometheus

```bash
cd /tmp
curl -s https://api.github.com/repos/prometheus/prometheus/releases/latest \
| grep browser_download_url \
| grep linux-amd64.tar.gz \
| cut -d '"' -f 4 \
| wget -i -
```

```bash
tar -xvzf prometheus-*.linux-amd64.tar.gz
cd prometheus-*.linux-amd64
```

### Move binaries

```bash
sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

### Move config files

```bash
sudo cp -r consoles /etc/prometheus
sudo cp -r console_libraries /etc/prometheus
sudo cp prometheus.yml /etc/prometheus/
sudo chown -R prometheus:prometheus /etc/prometheus
```

---

## **3. Create Prometheus Service**

```bash
sudo nano /etc/systemd/system/prometheus.service
```

Paste:

```ini
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
  --storage.tsdb.path /var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

Save & exit.

### Start Prometheus

```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```

Prometheus web UI 👉 `http://<server-ip>:9090`

---

## **4. Install Grafana**

### Add Grafana repo & install

```bash
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install grafana -y
```

### Enable & start Grafana

```bash
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
sudo systemctl status grafana-server
```

Grafana web UI 👉 `http://<server-ip>:3000`
(Default user/pass = **admin / admin**)

---

## **5. Configure Grafana to Use Prometheus**

1. Login to Grafana → **Configuration → Data Sources**
2. Add **Prometheus** with URL:

   ```
   http://localhost:9090
   ```
3. Save & Test 

## **6. Import Grafana Dashboard**

1. Open Grafana → **Dashboards → Import**
2. Use prebuilt **Node Exporter Full Dashboard (ID: 1860)**
   👉 [https://grafana.com/grafana/dashboards/1860](https://grafana.com/grafana/dashboards/1860)
3. Select Prometheus as data source → Import.

---

