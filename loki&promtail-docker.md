Here’s a clean and practical way to install **Loki** and **Promtail** using Docker — perfect for testing or local development. Since you're already hands-on with container orchestration and observability tools, this should feel right at home.

---

### 🛠 Prerequisites
- Docker installed
- A working directory (e.g., `~/loki-stack`)
- Optional: Docker Compose for easier orchestration

---

### ⚙️ Step-by-Step: Loki & Promtail with Docker CLI

1. **Create a working directory**:
   ```bash
   mkdir /opt/loki-stack && cd loki-stack
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
    -v /etc/promtail:/etc/promtail \
    grafana/promtail:3.4.1 \
    -config.file=/mnt/config/promtail-config.yaml

   ```

5. **Verify containers**:
   ```bash
   docker container ls
   ```
6. **Create Promtail Config File**: 
- sudo mkdir -p /etc/promtail
- sudo vi /etc/promtail/promtail-config.yaml

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /var/log/positions.yaml

clients:
  - url: http://143.110.189.74:3100/loki/api/v1/push

scrape_configs:
  # System logs
  - job_name: system-logs
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          host: livekit-server
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
        replacement: livekit-server

  # Fallback for containers not discovered via docker_sd
  - job_name: docker-containers
    static_configs:
      - targets:
          - localhost
        labels:
          job: docker-containers
          host: livekit-server
          __path__: /var/lib/docker/containers/*/*.log
```

Save this as `/etc/promtail/promtail-config.yaml`.

---






7. **Check Loki status**:
   - Ready endpoint: [http://localhost:3100/ready](http://localhost:3100/ready)
   - Metrics endpoint: [http://localhost:3100/metrics](http://localhost:3100/metrics)

---

### 🧩 Optional: Use Docker Compose

If you'd prefer a `docker-compose.yml` setup, [this guide](https://docs.techdox.nz/loki/) walks you through it with config mounting, port mapping, and service dependencies.

---

Let me know if you want to integrate Grafana next or enrich Promtail with container metadata — I can help you fine-tune the pipeline for your job-ready observability stack.
