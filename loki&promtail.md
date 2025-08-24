
---

### ⚙️ Step-by-Step Installation on Ubuntu

#### 1. **Install Dependencies**
```bash
sudo apt update && sudo apt install unzip curl -y
```

---

#### 2. **Download Loki Binary**
```bash
wget -qO loki-linux-amd64.zip https://github.com/grafana/loki/releases/latest/download/loki-linux-amd64.zip
unzip loki-linux-amd64.zip
mv loki-linux-amd64 loki
chmod +x loki
sudo mv loki /usr/local/bin/
```

---

#### 3. **Download Loki Config**
```bash
wget -qO loki-config.yaml https://raw.githubusercontent.com/grafana/loki/main/cmd/loki/loki-local-config.yaml
sudo mkdir -p /etc/loki
sudo mv loki-config.yaml /etc/loki/
```

---

#### 4. **Create Loki Systemd Service**
- create loki.service file in /etc/systemd/system/loki.service <br>
- sudo vi /etc/systemd/system/loki.service <br>
```bash
[Unit]
Description=Loki Log Aggregation
After=network.target

[Service]
ExecStart=/usr/local/bin/loki -config.file=/etc/loki/loki-config.yaml
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now loki
```

---

#### 5. **Download Promtail Binary**
```bash
wget -qO promtail-linux-amd64.zip https://github.com/grafana/loki/releases/latest/download/promtail-linux-amd64.zip
unzip promtail-linux-amd64.zip
mv promtail-linux-amd64 promtail
chmod +x promtail
sudo mv promtail /usr/local/bin/
```

---

#### 6. **Download Promtail Config**
```bash
wget -qO promtail-config.yaml https://raw.githubusercontent.com/grafana/loki/main/clients/cmd/promtail/promtail-local-config.yaml
sudo mkdir -p /etc/promtail
sudo mv promtail-config.yaml /etc/promtail/
```

---

#### 7. **Create Promtail Systemd Service**
- create loki.service file in /etc/systemd/system/promtail.service  <br>
- sudo vi /etc/systemd/system/promtail.service  <br>
```bash
[Unit]
Description=Promtail Log Collector
After=network.target

[Service]
ExecStart=/usr/local/bin/promtail -config.file=/etc/promtail/promtail-config.yaml
Restart=always

[Install]
WantedBy=multi-user.target
```
- Reload the daemon-service
```sh
sudo systemctl daemon-reload
sudo systemctl enable --now promtail
```

#### 8. **Inbound Ports for Running Services**
- Loki is accessible via inbound port 3100
- Promtail listens on inbound port 9080

---

### 9. **Verify Installation**
Check if services are running:
```bash
systemctl status loki
systemctl status promtail
```
Restart services if not running:
```sh
sudo systemctl restart loki
sudo systemctl status loki 
sudo systemctl restart promtail
sudo systemctl status promtail
```
---



####  10. **Confirm Loki Is Running**
If you followed the previous steps, Loki should be active. You can verify:

```bash
sudo systemctl status loki
```

Or test the endpoint:

```bash
curl http://localhost:3100/ready
```

You should see `ready` if it's working.

---

#### 11. **Confirm Promtail Is Running**
Promtail should be collecting logs and pushing them to Loki:

```bash
sudo systemctl status promtail
```

Check logs if needed:

```bash
journalctl -u promtail -f
```

---

###  12. **Add Loki as a Data Source in Grafana**

1. Open Grafana in your browser:  
   `http://localhost:3000`

2. Log in (default is `admin` / `admin` unless changed).

3. Go to **Settings → Data Sources → Add data source**.

4. Choose **Loki**.

5. Set the URL to:  
   `http://localhost:3100`

6. Click **Save & Test**.

---

### 13. **Explore Logs in Grafana**

- Go to **Explore** tab.
- Select **Loki** as the data source.
- Use a query like:

```logql
{job="promtail"}
```



