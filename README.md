# eBPF Observability Stack

This Helm chart deploys a complete observability stack based on Grafana's LGTM stack (Loki, Grafana, Tempo, Mimir) with Pyroscope for continuous profiling and Alloy for eBPF-based observability.

## Prerequisites

- Kubernetes cluster (v1.19+)
- Helm 3.2.0+
- Access to an S3-compatible object storage for persistent storage of metrics, logs, traces, and profiles (e.g. Minio)

## Components

This Helm chart includes the following components:

- **Grafana**: Visualization and dashboarding
- **Mimir**: Metrics storage and querying (Prometheus-compatible)
- **Loki**: Log aggregation and querying
- **Tempo**: Distributed tracing backend
- **Pyroscope**: Continuous profiling
- **Alloy**: eBPF-based observability agent

## Installation

### 1. Add the Helm repository

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

### 2. Configure S3 storage

Edit the `values.yaml` file to configure your S3-compatible storage:

```yaml
# For Loki
loki:
  loki:
    storage:
      s3:
        endpoint: "your-s3-endpoint"
        accessKeyId: "your-access-key"
        secretAccessKey: "your-secret-key"

# For Mimir
mimir-distributed:
  mimir:
    structuredConfig:
      blocks_storage:
        s3:
          endpoint: "your-s3-endpoint"
          access_key_id: "your-access-key"
          secret_access_key: "your-secret-key"
      alertmanager_storage:
        s3:
          endpoint: "your-s3-endpoint"
          access_key_id: "your-access-key"
          secret_access_key: "your-secret-key"
      ruler_storage:
        s3:
          endpoint: "your-s3-endpoint"
          access_key_id: "your-access-key"
          secret_access_key: "your-secret-key"

# For Tempo
tempo-distributed:
  storage:
    trace:
      s3:
        endpoint: "your-s3-endpoint"
        access_key: "your-access-key"
        secret_key: "your-secret-key"
```

### 3. Install the Helm chart

```bash
# Clone this repository
git clone https://github.com/yourusername/ebpf-observability-setup.git
cd ebpf-observability-setup

# Install the chart
helm install observability-stack . -n observability --create-namespace
```

### 4. Verify the installation

```bash
kubectl get pods -n observability
```

### 5. Access Grafana

```bash
# Port-forward Grafana service
kubectl port-forward svc/grafana 8080:80 -n observability
```

```bash
# Get the admin user default password
kubectl get secrets grafana -o jsonpath='{.data.admin-password}' | base64 -d
```

Then open your browser and navigate to `http://localhost:8080`. Login with:
- Username: `admin`
- Password: `<output from above kubectl command>`

## Configuration

### Customizing values

You can customize the installation by creating your own `values.yaml` file:

```bash
helm install observability-stack . -n observability --create-namespace -f my-values.yaml
```

### Important configuration options

#### Resource limits

Adjust resource limits based on your cluster capacity:

```yaml
mimir-distributed:
  ingester:
    resources:
      limits:
        cpu: 1
        memory: 2Gi
      requests:
        cpu: 500m
        memory: 1Gi

loki:
  ingester:
    resources:
      limits:
        cpu: 1
        memory: 2Gi
      requests:
        cpu: 500m
        memory: 1Gi
```

#### Persistence

By default, this chart uses S3 for persistence. You can enable local persistence for development:

```yaml
mimir-distributed:
  ingester:
    persistentVolume:
      enabled: true
      size: 10Gi
```

#### Alloy eBPF Configuration

Alloy requires privileged access to use eBPF features. The default configuration enables this, but you can customize the eBPF programs by modifying the `alloy-config.yaml` template.

## Included Dashboards

This chart includes several pre-configured dashboards:
- APM Dashboard
- Span Metrics Demo
- MLT (Metrics, Logs, Traces) Dashboard

## Upgrading

To upgrade the chart:

```bash
helm upgrade observability-stack . -n observability
```

## Uninstalling

To uninstall the chart:

```bash
helm uninstall observability-stack -n observability
```
