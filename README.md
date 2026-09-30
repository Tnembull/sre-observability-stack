# Production Observability & Alerting Stack

Automated, production-grade telemetry and observability platform based on Prometheus, Alertmanager, Node Exporter, and Grafana with automated Telegram incident alerting and Dashboards-as-Code.

---

## 🎯 Architecture Overview

```text
               +-------------------------------------------------+
               |                   Linux Host                    |
               +-----------------------+-------------------------+
                                       | (Host Metrics)
                                       v
                              +-----------------+
                              |  Node Exporter  | :9100
                              +--------+--------+
                                       |
                                       | (Pull / Scrape 15s)
                                       v
+------------------+          +-----------------+          +--------------------+
|  Telegram Alert  | <------- |  Alertmanager   | <------- |  Prometheus TSDB   | :9090
|  (Incidents)     |  :9093   +-----------------+  Rules   +---------+----------+
+------------------+                                                 |
                                                                     | (Query PromQL)
                                                                     v
                                                            +--------------------+
                                                            | Grafana Dashboard  | :3000
                                                            | (Dashboards-as-Code)|
                                                            +--------------------+
```

---

## 🚀 Key Technical Highlights

1. **Declarative Telemetry & Infrastructure as Code**:
   - Entire monitoring stack defined using Docker Compose with network isolation (`monitoring` bridge).
   - Dashboards provisioned via JSON code (`Dashboards-as-Code`), ensuring zero manual configuration drift.
   - Datasource automated discovery (`Prometheus` auto-wired at startup).

2. **Proactive Incident Alerting Rules (`alert.rules.yml`)**:
   - **`InstanceDown`**: Triggers when any scraped target becomes unreachable for > 1 minute.
   - **`HighCpuLoad`**: Warns when CPU load exceeds 85% continuously over 5 minutes.
   - **`HighMemoryUsage`**: Critical alert when available memory falls below 10%.
   - **`DiskSpaceLow`**: Critical alert when root filesystem free space drops under 15%.

3. **Telegram Notification Engine (`alertmanager.yml`)**:
   - Real-time formatted HTML push alerts with firing/resolved state, severity labels, and exact UTC timestamps.
   - Smart deduplication and alert grouping (`group_wait: 30s`, `repeat_interval: 4h`).

---

## 📁 Repository Structure

```text
├── docker-compose.yml                  # Container stack orchestration & network definitions
├── .env.example                        # Template for non-committed environment variables
├── prometheus/
│   ├── prometheus.yml                  # Scrape intervals & target endpoints
│   └── alert.rules.yml                 # Threshold definitions and severity rules
├── alertmanager/
│   └── alertmanager.yml                # Notification routing & Telegram templates
└── grafana/provisioning/
    ├── datasources/prometheus.yml      # Auto-provisioned Prometheus data connector
    └── dashboards/
        ├── dashboards.yml              # Dashboard provider definitions
        └── node-metrics.json           # Pre-configured production metric dashboards
```

---

## 🛠️ Deployment Instructions

### 1. Prerequisites
- Docker Engine & Docker Compose plugin installed.
- Telegram Bot Token & Target Chat ID.

### 2. Environment Configuration
Copy environment variables template:
```bash
cp .env.example .env
```
Update credentials and Telegram routing parameters in `.env` and `alertmanager/alertmanager.yml`.

### 3. Start the Observability Stack
```bash
docker compose up -d
```

### 4. Verify Services
- **Prometheus UI**: `http://localhost:9090`
- **Alertmanager UI**: `http://localhost:9093`
- **Node Exporter**: `http://localhost:9100/metrics`
- **Grafana Web**: `http://localhost:3000` (Default credentials from `.env`)

---

## 🛡️ Security & Zero Leak Posture
- All sensitive access tokens, bot secrets, and passwords are separated into `.env` (guarded by `.gitignore`).
- Local ports bind to `127.0.0.1` by default to prevent unintended exposure to public internet.

---
**Author**: Muhammad Nur Ashiddiqi  
**Portfolio**: [muhammadnurashiddiqi.my.id](https://muhammadnurashiddiqi.my.id)
