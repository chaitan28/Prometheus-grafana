 Promtail is the agent that **collects logs** from your system and **pushes them to Loki**, which then stores and indexes them for Grafana to visualize.

---

### 🚀 Promtail Setup to Send Logs to Loki

#### 1. **Install Promtail**
If you haven’t already:

```bash
wget -qO promtail-linux-amd64.zip https://github.com/grafana/loki/releases/latest/download/promtail-linux-amd64.zip
unzip promtail-linux-amd64.zip
mv promtail-linux-amd64 promtail
chmod +x promtail
sudo mv promtail /usr/local/bin/
```

---

#### 2. **Create Promtail Config File**
Here’s a basic config that scrapes system logs and sends them to Loki:

- sudo mkdir -p /etc/promtail
- sudo vi /etc/promtail/promtail-config.yaml

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /var/log/positions.yaml

clients:
  - url: http://172.173.113.245:3100/loki/api/v1/push

scrape_configs:
  # System logs
  - job_name: system-logs
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          host: sample-server
          __path__: /var/log/*log

  # Docker logs with container names
  - job_name: docker-logs
    docker_sd_configs:
      - host: unix:///var/run/docker.sock
        refresh_interval: 5s

    pipeline_stages:
      - docker: {}

    relabel_configs:
      - source_labels: [__meta_docker_container_name]
        target_label: container
      - source_labels: [__meta_docker_container_id]  
        target_label: container_id

```

Save this as `/etc/promtail/promtail-config.yaml`.

---

#### 3. **Create Promtail Systemd Service**
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

---

### ✅ Verify It’s Working

- Check Promtail status:
  ```bash
  systemctl status promtail
  ```

- Tail logs to confirm it's pushing:
  ```bash
  journalctl -u promtail -f
  ```

- In Grafana → Explore → Loki:
  ```logql
  {job="varlogs"}
  ```
---