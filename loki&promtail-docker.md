
---

### 🛠 Prerequisites
- Docker installed
- A working directory (e.g., `~/loki-stack`)
- Optional: Docker Compose for easier orchestration

---

### ⚙️ Step-by-Step: Loki & Promtail with Docker CLI

1. **Create a working directory**:
   ```bash
   mkdir /opt/monitoring/loki-stack && cd /opt/monitoring/loki-stack
   ```

2. **Download config files**:
   ```bash
   wget https://raw.githubusercontent.com/grafana/loki/v3.4.1/cmd/loki/loki-local-config.yaml -O loki-config.yaml
   wget https://raw.githubusercontent.com/grafana/loki/v3.4.1/clients/cmd/promtail/promtail-docker-config.yaml -O promtail-config.yaml
   ```

3. **Run Loki container**:
   ```bash
   docker run --name loki -d \
     -v $(pwd):/mnt/config \
     -p 3100:3100 \
     grafana/loki:3.4.1 \
     -config.file=/mnt/config/loki-config.yaml
   ```

4. **Run Promtail container**:
   ```bash
    docker run -d \
    --name promtail \
    -v $(pwd):/mnt/config \
    -v /var/log:/var/log \
    -v /var/lib/docker/containers:/var/lib/docker/containers \
    -v /var/run/docker.sock:/var/run/docker.sock \
    grafana/promtail:3.4.1 \
    -config.file=/mnt/config/promtail-config.yaml

   ```

5. **Verify containers**:
   ```bash
   docker container ls
   ```
6. **Create Promtail Config File**: 
- sudo vi /opt/monitoring/loki-stack/promtail-config.yaml

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /var/log/positions.yaml

clients:
  - url: http://<ip of loki server>:3100/loki/api/v1/push  # Replace with loki server IP

scrape_configs:
  # System logs
  - job_name: system-logs
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          host: monitoring-server  # Replace with Current server name
          __path__: /var/log/*log

  # Docker logs via service discovery
  - job_name: docker-logs
    docker_sd_configs:
      - host: unix:///var/run/docker.sock
        refresh_interval: 5s

    pipeline_stages:
      - docker: {}

    relabel_configs:
      - source_labels: [__meta_docker_container_name]
        regex: "(.*)"
        target_label: container_name
        replacement: "$1"

      - source_labels: [__meta_docker_container_id]
        target_label: container_id

      - source_labels: [__address__]
        target_label: host
        replacement: monitoring-server  # Replace with Current server name

  # Fallback for containers not discovered via docker_sd
  - job_name: docker-containers
    static_configs:
      - targets:
          - localhost
        labels:
          job: docker-containers
          host: monitoring-server                     # Replace with Current server name
          __path__: /var/lib/docker/containers/*/*.log

```

Save this as `/opt/monitoring/loki-stack/promtail-config.yaml`.

---
7. **Restart Docker**:
- If any errors are encountered, restart the Docker service:
```sh
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl status docker 

```

8. **Check Loki status**:
   - Ready endpoint: [http://localhost:3100/ready](http://localhost:3100/ready)
   - Metrics endpoint: [http://localhost:3100/metrics](http://localhost:3100/metrics)

---
