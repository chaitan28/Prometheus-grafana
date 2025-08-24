Got it 👍 — thanks for clarifying.
So in your Promtail config you wrote:

```yaml
labels:
  job: varlogs
  host: ${HOSTNAME}
  __path__: /var/log/*.log
```

But the problem is → Promtail **does not expand environment variables** like `${HOSTNAME}` when they’re inside the `labels:` block.
That’s why Grafana literally shows `host="$HOSTNAME"` instead of `host="server1"` / `host="server2"`.

---

### ✅ Fix Options

#### **Option 1: Hardcode host label in each Promtail config**

On **server1**:

```yaml
labels:
  job: varlogs
  host: server1
  __path__: /var/log/*.log
```

On **server2**:

```yaml
labels:
  job: varlogs
  host: server2
  __path__: /var/log/*.log
```

👉 This way, Grafana queries will show:

```
{job="varlogs", host="server1"}
{job="varlogs", host="server2"}
```

---

#### **Option 2: Use Promtail `hostname` stage**

Instead of manually setting `host: ${HOSTNAME}`, let Promtail detect it:

```yaml
scrape_configs:
- job_name: system
  static_configs:
  - targets:
      - localhost
    labels:
      job: varlogs
      __path__: /var/log/*.log

  pipeline_stages:
  - hostname:
      target: host
```

👉 Here, Promtail will automatically set `host=<actual-hostname>` from the system.

---

### 🔍 Best Practice

If you’re running Promtail on **multiple servers**, the cleanest way is **Option 2** — that way you don’t need to manually edit configs for each server, and Grafana will let you filter `host=server1` / `host=server2`.

---

Do you want me to give you the **final full Promtail config** (with hostname + multiple log files like syslog/auth.log separated), so you can directly copy-paste it for both servers?
