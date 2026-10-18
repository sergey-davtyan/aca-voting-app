# Kubernetes Specifications for Voting App

This directory contains the Kubernetes manifests to deploy the Voting App stack.

## Recent Architectural Updates Reflected

1. **Redis / Valkey Authentication**:
   - `worker` and `vote` now use authenticated connections to Redis.
   - `redis-deployment.yaml` starts with `--requirepass` using credentials from `secret.yaml`.
   - `worker` requires `REDIS_PASSWORD` to be present (otherwise it aborts startup).
2. **Postgres / RDS Connection Parameters**:
   - `result` and `worker` read host, port, username, password, database name, and SSL settings from environment variables.
   - Credentials are systematically decoupled into `secret.yaml`.
3. **Environment & Configuration Decoupling**:
   - `configmap.yaml` provides non-sensitive parameters (`REDIS_HOST`, `REDIS_PORT`, `REDIS_SSL`, `POSTGRES_HOST`, `POSTGRES_PORT`, `POSTGRES_SSL`, `OPTION_A`, `OPTION_B`, `PORT`).
   - `secret.yaml` provides sensitive credentials (`POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`, `REDIS_USERNAME`, `REDIS_PASSWORD`).
4. **Health Checks & Resilience**:
   - Added HTTP readiness and liveness probes for `vote` and `result`.
   - Added authenticated `redis-cli ping` and `pg_isready` probes for data stores.
   - Added CPU and memory resource requests/limits across all services.

## Building Images Locally

To build and tag the container images from the repository root:

```bash
docker build -t aca-voting-app-vote:latest ./vote
docker build -t aca-voting-app-result:latest ./result
docker build -t aca-voting-app-worker:latest ./worker
```

> **Note**: If using Minikube or Kind, run `eval $(minikube docker-env)` or `kind load docker-image ...` before building so images are available inside the cluster.

## Deploying to Kubernetes

Deploy all resources:

```bash
kubectl apply -f k8s-specifications/
```

Access the applications:
- **Vote App**: `http://<node-ip>:31000` (NodePort)
- **Result App**: `http://<node-ip>:31001` (NodePort)

## ECR Private Registry Authentication

Workloads using the AWS ECR registry (`597936860210.dkr.ecr.eu-north-1.amazonaws.com`) reference an image pull secret named `ecr-secret`.

- [ecr-secret.yaml](file:///Users/sergey.davtyan/code/aca-voting-app/k8s-specifications/ecr-secret.yaml): Contains the `kubernetes.io/dockerconfigjson` secret manifest.
- [ecr-renew-cronjob.yaml](file:///Users/sergey.davtyan/code/aca-voting-app/k8s-specifications/ecr-renew-cronjob.yaml): (Optional) Runs every 6 hours inside the cluster to automatically renew `ecr-secret` since AWS ECR tokens expire after 12 hours.

To generate or refresh `ecr-secret.yaml` on a machine with AWS CLI & kubectl:
```bash
kubectl create secret docker-registry ecr-secret \
  --docker-server=597936860210.dkr.ecr.eu-north-1.amazonaws.com \
  --docker-username=AWS \
  --docker-password=$(aws ecr get-login-password --region eu-north-1) \
  --dry-run=client -o yaml > k8s-specifications/ecr-secret.yaml
```

## Monitoring & Logging Stack (Prometheus, Loki, Promtail, Grafana)

A complete monitoring and logging stack is available in [k8s-specifications/monitoring/](file:///Users/sergey.davtyan/code/aca-voting-app/k8s-specifications/monitoring):
- **Prometheus** (metrics): NodePort `30090`
- **Loki** (log aggregation): ClusterIP `3100`
- **Promtail** (log shipper): Runs as a **DaemonSet** collecting `/var/log/pods` from all nodes
- **Grafana** (dashboard): NodePort `32000` (pre-configured with Prometheus and Loki data sources)

To deploy monitoring:

```bash
kubectl apply -f k8s-specifications/monitoring/
```

To delete monitoring:

```bash
kubectl delete -f k8s-specifications/monitoring/
```

