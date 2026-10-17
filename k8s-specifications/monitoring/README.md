# Monitoring & Logging Stack (Prometheus, Loki, Promtail, Grafana)

This directory contains the Kubernetes specifications for a complete observability stack to store and visualize metrics and logs from the Voting App.

## Architecture

| Component | Type | Function | Access Port |
| :--- | :--- | :--- | :--- |
| **Prometheus** | Deployment | Scrapes and stores metrics from nodes, pods, and cAdvisor | NodePort: `30090` |
| **Loki** | Deployment | Ingests and indexes logs | ClusterIP: `3100` |
| **Promtail** | DaemonSet | Runs on each Kubernetes node, tails container logs (`/var/log/pods`), and streams them to Loki | Internal (HostPath) |
| **Grafana** | Deployment | Visualizes metrics and logs with pre-provisioned Prometheus and Loki datasources | NodePort: `32000` |

---

## Deployment

Deploy the entire monitoring and logging stack with a single command:

```bash
kubectl apply -f k8s-specifications/monitoring/
```

Verify that all pods are running:

```bash
kubectl get pods -l 'app in (prometheus, loki, promtail, grafana)'
```

---

## Accessing the Dashboards

### 1. Grafana (Unified Metrics & Logs)
- **URL**: `http://<node-ip>:32000`
- **Default Credentials**:
  - Username: `admin`
  - Password: `admin`
- **Pre-configured Data Sources**:
  - `Prometheus`: Pre-configured as the default metrics source (`http://prometheus:9090`).
  - `Loki`: Pre-configured as the log exploration source (`http://loki:3100`).

To explore application logs in Grafana:
1. Navigate to **Explore** in the left sidebar.
2. Select the **Loki** data source.
3. Query logs by label, for example:
   ```logql
   {app="vote"}
   {app="result"}
   {app="worker"}
   ```

### 2. Prometheus Web Console
- **URL**: `http://<node-ip>:30090`
- Query cluster, pod, and container metrics (e.g., `container_cpu_usage_seconds_total`, `container_memory_working_set_bytes`).

---

## Teardown

To delete the monitoring and logging stack:

```bash
kubectl delete -f k8s-specifications/monitoring/
```
