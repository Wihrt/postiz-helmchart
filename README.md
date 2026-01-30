# Postiz Helm Chart

This Helm chart deploys the Postiz application on a Kubernetes cluster using the Helm package manager.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- **Temporal workflow engine** deployed and accessible
- PV provisioner support in the underlying infrastructure (if persistence is required)

## Temporal Workflow Engine Setup

Postiz requires Temporal for workflow orchestration. You must deploy Temporal before installing this chart.

### Option 1: Self-Hosted Temporal in Same Cluster (Recommended for Production)

Deploy Temporal using the official Helm chart in a dedicated namespace:

```bash
# Add Temporal Helm repository
helm repo add temporalio https://temporalio.github.io/helm-charts

# Create Temporal namespace
kubectl create namespace temporal

# Install Temporal with PostgreSQL backend
helm install temporal temporalio/temporal \
  --namespace temporal \
  --values examples/temporal-self-hosted.yaml
```

After deployment, configure Postiz to use it:
```yaml
env:
  TEMPORAL_ADDRESS: "temporal-frontend.temporal.svc.cluster.local:7233"
```

See the [Self-Hosted Temporal Deployment Guide](#self-hosted-temporal-deployment-guide) for detailed setup instructions.

### Option 2: Temporal Cloud (Managed SaaS)

Sign up at [temporal.io/cloud](https://temporal.io/cloud) and get your connection details:

```yaml
env:
  TEMPORAL_ADDRESS: "your-namespace.tmprl.cloud:7233"

secrets:
  # If using mTLS authentication
  TEMPORAL_CLIENT_CERT: "base64-encoded-cert"
  TEMPORAL_CLIENT_KEY: "base64-encoded-key"
```

### Option 3: Existing Temporal Instance

If you already have Temporal deployed, simply configure the address:

```yaml
env:
  TEMPORAL_ADDRESS: "your-temporal-host:7233"
```

## Installing the Chart

**Important**: Ensure Temporal is deployed and accessible before proceeding.

The Postiz helm chart registry uses the OCI format, not HTTP, which means you do not need to do a `helm repo add` to install the chart. You can install the chart directly from the GitHub repository.

To install the chart with the release name `postiz-app`:

```bash
# Install with Temporal address
helm install postiz-app oci://ghcr.io/gitroomhq/postiz-helmchart/charts/postiz-app \
  --set env.TEMPORAL_ADDRESS="temporal-frontend.temporal.svc.cluster.local:7233"
```

Or using a values file:
```yaml
# my-values.yaml
env:
  TEMPORAL_ADDRESS: "temporal-frontend.temporal.svc.cluster.local:7233"

  # Your other environment variables
  FRONTEND_URL: "https://postiz.example.com"
  NEXT_PUBLIC_BACKEND_URL: "https://api.postiz.example.com"
```

```bash
helm install postiz-app oci://ghcr.io/gitroomhq/postiz-helmchart/charts/postiz-app \
  -f my-values.yaml
```

The [Parameters](#parameters) section lists the parameters that can be configured during installation.

> **Tip**: List all releases using `helm list`

## Uninstalling the Chart

To uninstall/delete the `postiz` deployment:

```bash
$ helm delete postiz-app
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Parameters

The following table lists the configurable parameters of the Postiz chart and their default values.

| Parameter                     | Description                          | Default        |
| ----------------------------- | ------------------------------------ | -------------- |
| `replicaCount`                | Number of replicas                   | `1`            |
| `fullnameOverride`            | Override release name (affects all service hostnames) | `""`       |
| `image.repository`            | Image repository                     | `ghcr.io/gitroomhq/postiz-app` |
| `image.pullPolicy`            | Image pull policy                    | `IfNotPresent` |
| `image.tag`                   | Image tag (empty defaults to appVersion v2.13.0) | `""`           |
| `env.TEMPORAL_ADDRESS`        | **REQUIRED**: Temporal frontend service address | `""`      |
| `service.type`                | Kubernetes service type              | `ClusterIP`    |
| `service.port`                | Kubernetes service port              | `80`           |
| `postgresql.enabled`          | Deploy PostgreSQL                    | `true`         |
| `postgresql.auth.username`    | PostgreSQL username                  | `postiz`       |
| `postgresql.auth.password`    | PostgreSQL password                  | `postiz-password` |
| `postgresql.auth.database`    | PostgreSQL database                  | `postiz`       |
| `redis.enabled`               | Deploy Redis (Valkey)                | `true`         |
| `redis.auth.password`         | Redis/Valkey password                | `postiz-redis-password` |
| `secrets.autoGenerate.enabled`| Auto-generate connection strings     | `true`         |
| `secrets.autoGenerate.database`| Auto-generate DATABASE_URL          | `true`         |
| `secrets.autoGenerate.redis`  | Auto-generate REDIS_URL              | `true`         |
| `extraSecrets`                | Additional external secrets to inject| `[]`           |
| `podAnnotations`              | Pod template annotations             | `{}`           |
| `deploymentAnnotations`       | Deployment resource annotations      | `{}`           |
| `configMapAnnotations`        | ConfigMap resource annotations       | `{}`           |
| `secretAnnotations`           | Secret resource annotations          | `{}`           |
| `serviceAnnotations`          | Service resource annotations         | `{}`           |
| `ingress.enabled`             | Enable ingress controller resource   | `false`        |
| `ingress.className`           | IngressClass that will be used       | `""`           |
| `ingress.annotations`         | Ingress annotations                  | `{}`           |
| `ingress.hosts`               | Ingress hostnames                    | `[]`           |
| `ingress.tls`                 | Ingress TLS configuration            | `[]`           |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example,

```bash
$ helm install postiz-app oci://ghcr.io/gitroomhq/postiz-helmchart/charts/postiz-app \
  --set env.TEMPORAL_ADDRESS="temporal:7233" \
  --set postgresql.auth.password=secretpassword
```

Alternatively, you can use a YAML file to specify the values while installing the chart. Create a file called `custom-values.yaml` (or any name you prefer) and specify your values:

```yaml
env:
  TEMPORAL_ADDRESS: "temporal-frontend.temporal.svc.cluster.local:7233"

postgresql:
  auth:
    password: secretpassword

ingress:
  enabled: true
  hosts:
    - host: postiz.example.com
```

Then, you can install the chart using the `-f` flag:

```bash
$ helm install postiz-app oci://ghcr.io/gitroomhq/postiz-helmchart/charts/postiz-app -f custom-values.yaml
```

> **Tip**: You can use the default [values.yaml](charts/postiz/values.yaml) as a starting point for your custom configuration.

## Persistence

The chart mounts a [Persistent Volume](http://kubernetes.io/docs/user-guide/persistent-volumes/) for the PostgreSQL and Redis data. The volume is created using dynamic volume provisioning. If you want to disable this functionality you can change the values.yaml to disable persistence and use an emptyDir instead.

## Configuration and installation details

### Service Naming Convention

All internal service hostnames follow a consistent pattern based on the release name (or `fullnameOverride`):

- **PostgreSQL**: `{fullname}-postgresql`
- **Redis/Valkey**: `{fullname}-redis-primary`

Where `{fullname}` is determined by:
1. If `fullnameOverride` is set: uses that value
2. Otherwise: `{release-name}-postiz-app` (or just `{release-name}` if it already contains "postiz-app")

**Example**: Installing with `helm install my-release` will create:
- `my-release-postiz-app-postgresql`
- `my-release-postiz-app-redis-primary`

**Custom naming**: To use custom names, set `fullnameOverride`:
```bash
helm install my-release . --set fullnameOverride="custom-postiz"
```
This creates: `custom-postiz-postgresql`, `custom-postiz-redis-primary`, etc.

### External database support

You may want to have Postiz connect to an external database rather than installing one inside your cluster. Typical reasons for this are to use a managed database service, or to share a common database server for all your applications. To achieve this, set the `postgresql.enabled` parameter to `false` and specify the credentials for the external database using the `postgresql.auth.username`, `postgresql.auth.password`, and `postgresql.auth.database` parameters.

### External Redis support

Similar to the database, you can use an external Redis instance by setting `redis.enabled` to `false` and specifying the external Redis URL using the `REDIS_URL` environment variable in the `secrets` section of your values.yaml.

### Auto-Generated Connection Strings

The chart automatically generates connection strings for internal dependencies:

When using internal PostgreSQL and Redis (Valkey), the chart automatically generates connection strings:

```yaml
secrets:
  autoGenerate:
    enabled: true  # Default
    database: true
    redis: true
```

**Generated values:**
- `DATABASE_URL`: `postgresql://postiz:postiz-password@{fullname}-postgresql:5432/postiz`
- `REDIS_URL`: `redis://:postiz-redis-password@{fullname}-redis-primary:6379`

To disable auto-generation and use manual connection strings:

```yaml
secrets:
  autoGenerate:
    enabled: false
  DATABASE_URL: "postgresql://user:pass@external-db.example.com:5432/postiz"
  REDIS_URL: "redis://:password@external-redis.example.com:6379"
```

### External Secrets Integration

You can inject additional Kubernetes secrets (e.g., from Vault, AWS Secrets Manager) alongside the chart-managed secrets.

Create your external secret:
```bash
kubectl create secret generic vault-oauth-secrets \
  --from-literal=LINKEDIN_CLIENT_ID="your-id" \
  --from-literal=LINKEDIN_CLIENT_SECRET="your-secret"
```

Reference it in your values:
```yaml
extraSecrets:
  - name: "vault-oauth-secrets"
  - name: "cloudflare-credentials"
```

All secrets are merged as environment variables in the Postiz deployment.

### Custom Annotations

Add custom annotations to Kubernetes resources for integrations like Reloader, policy enforcement, or monitoring.

#### Reloader Integration

To automatically restart pods when ConfigMap or Secret changes (using [Stakater Reloader](https://github.com/stakater/Reloader)):

**Auto-reload (watches all referenced configs/secrets):**
```yaml
podAnnotations:
  reloader.stakater.com/auto: "true"
```

**Specific resource matching:**
```yaml
configMapAnnotations:
  reloader.stakater.com/match: "true"
secretAnnotations:
  reloader.stakater.com/match: "true"
deploymentAnnotations:
  reloader.stakater.com/search: "true"
```

#### Available Annotation Fields

- `podAnnotations` - Pod template annotations
- `deploymentAnnotations` - Deployment resource annotations
- `configMapAnnotations` - ConfigMap resource annotations
- `secretAnnotations` - Secret resource annotations
- `serviceAnnotations` - Service resource annotations

## Self-Hosted Temporal Deployment Guide

This guide explains how to deploy Temporal in the same Kubernetes cluster for use with Postiz.

### Architecture

```
┌─────────────────────────────────────────────────┐
│         Temporal Namespace (temporal)            │
├─────────────────────────────────────────────────┤
│  ┌─────────────────┐        ┌────────────────┐  │
│  │ Temporal Server │        │  Temporal Web  │  │
│  │   (3 replicas)  │        │      UI        │  │
│  └────────┬────────┘        └────────────────┘  │
│           │                                     │
│           ▼                                     │
│  ┌─────────────────────────────────────────┐   │
│  │    PostgreSQL (temporal databases)      │   │
│  │  - temporal (default)                   │   │
│  │  - temporal_visibility (visibility)     │   │
│  └─────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│         Postiz Namespace (default)              │
├─────────────────────────────────────────────────┤
│  ┌──────────────────┐                           │
│  │     Postiz       │                           │
│  │   Application    │                           │
│  └────────┬─────────┘                           │
│           │ TEMPORAL_ADDRESS:7233               │
│           │ (in-cluster service)                │
│           ▼                                     │
│  ┌──────────────────────────────────────────┐  │
│  │  PostgreSQL + Redis                      │  │
│  └──────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

### Prerequisites

- Helm with Temporal chart repository
- Sufficient cluster resources (minimum 2 CPUs, 2Gi RAM for Temporal)
- Optional: Persistent volume provisioner for production

### Step 1: Add Temporal Helm Repository

```bash
helm repo add temporalio https://temporalio.github.io/helm-charts
helm repo update
```

### Step 2: Create Temporal Namespace

```bash
kubectl create namespace temporal
```

### Step 3: Deploy Temporal

Use the provided example values file:

```bash
helm install temporal temporalio/temporal \
  --namespace temporal \
  --values examples/temporal-self-hosted.yaml
```

### Step 4: Verify Temporal Deployment

```bash
# Check if all pods are running
kubectl get pods -n temporal

# Expected output:
# NAME                                    READY   STATUS    RESTARTS   AGE
# temporal-postgresql-0                   1/1     Running   0          2m
# temporal-server-0                       1/1     Running   0          1m
# temporal-web-6d5f8c5f-hxyz              1/1     Running   0          1m

# Check if service is accessible
kubectl get svc -n temporal
```

### Step 5: Access Temporal Web UI

Option 1: Port forwarding
```bash
kubectl port-forward -n temporal svc/temporal-web 8080:8080
# Visit: http://localhost:8080
```

Option 2: Create Ingress (optional)
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: temporal-web
  namespace: temporal
spec:
  ingressClassName: nginx  # or your ingress controller
  rules:
    - host: temporal.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: temporal-web
                port:
                  number: 8080
```

### Step 6: Deploy Postiz with Temporal Address

```bash
helm install postiz-app oci://ghcr.io/gitroomhq/postiz-helmchart/charts/postiz-app \
  --set env.TEMPORAL_ADDRESS="temporal-frontend.temporal.svc.cluster.local:7233"
```

### Step 7: Verify Connection

```bash
# Check Postiz logs for successful Temporal connection
kubectl logs -l app.kubernetes.io/name=postiz-app --tail=50

# Look for messages like:
# "Successfully connected to Temporal"
# or check the Temporal Web UI Namespaces page
```

### Resource Recommendations

For production deployments:

```yaml
server:
  replicaCount: 3
  resources:
    requests:
      memory: "512Mi"
      cpu: "500m"
    limits:
      memory: "1Gi"
      cpu: "1000m"

web:
  replicaCount: 2
  resources:
    requests:
      memory: "256Mi"
      cpu: "200m"
    limits:
      memory: "512Mi"
      cpu: "500m"

postgresql:
  primary:
    persistence:
      size: 20Gi  # Adjust based on workflow history volume
```

### Troubleshooting

**Postiz cannot connect to Temporal:**
```bash
# Verify service name and port
kubectl get svc -n temporal
kubectl get endpoints -n temporal temporal-frontend

# Test connectivity from Postiz pod
kubectl exec -it {postiz-pod} -- nc -zv temporal-frontend.temporal.svc.cluster.local 7233
```

**Temporal pods not starting:**
```bash
# Check Temporal pod logs
kubectl logs -n temporal {temporal-pod}

# Check Temporal database
kubectl exec -it -n temporal temporal-postgresql-0 -- psql -U temporal -d temporal -c "SELECT * FROM schema_version;"
```

**Increase verbosity:**
```yaml
server:
  config:
    log:
      level: debug
```

## Migration from v1.1.0 (Embedded Temporal)

If you're upgrading from v1.1.0 which had embedded Temporal, follow these steps:

### Step 1: Export Temporal Data (if needed)

If you have workflows in the embedded Temporal instance that you want to preserve:

```bash
# Port forward to Temporal Web UI
kubectl port-forward svc/postiz-app-temporal-web 8080:8080

# Use Temporal CLI to export workflow history
temporal workflow list --address localhost:7233
temporal workflow describe --address localhost:7233 --workflow-id {workflow-id}
```

### Step 2: Deploy Standalone Temporal

```bash
helm install temporal temporalio/temporal \
  --namespace temporal \
  --create-namespace \
  --values examples/temporal-migration.yaml
```

### Step 3: Migrate Database (if preserving data)

The migration example values file configures Temporal to use the existing databases:
- Database: `temporal`
- Visibility Database: `temporal_visibility`

```bash
# Verify databases are accessible from new Temporal instance
kubectl exec -it -n temporal temporal-postgresql-0 -- psql -U postiz -d temporal -c "SELECT * FROM schema_version;"
```

### Step 4: Update Postiz Release

```bash
# Upgrade to v1.1.0 with external Temporal address
helm upgrade postiz-app oci://ghcr.io/gitroomhq/postiz-helmchart/charts/postiz-app \
  --set env.TEMPORAL_ADDRESS="temporal-frontend.temporal.svc.cluster.local:7233" \
  --reuse-values
```

### Step 5: Verify Connection

```bash
# Check Postiz logs
kubectl logs -l app.kubernetes.io/name=postiz-app --tail=50

# Should see successful Temporal connection
```

### Step 6: Clean Up Old Temporal (Optional)

Once verified, you can remove the old embedded Temporal resources:

```bash
# List old Temporal resources
kubectl get all -l app.kubernetes.io/name=postiz-app | grep temporal

# Delete old Temporal deployment (if needed)
# kubectl delete deployment postiz-app-temporal-server --cascade=orphan
```

**⚠️ Warning**: Do not delete the PostgreSQL databases (`temporal` and `temporal_visibility`) until you've fully validated the new Temporal instance.

## Upgrading

### To 1.1.0

Version 1.1.0 introduces significant changes:

**Key Changes:**
- ✅ Valkey replaces Redis as caching backend
- ✅ Temporal now deployed separately (not embedded)
- ✅ Simplified configuration and chart structure
- ✅ Comprehensive external Temporal setup documentation
- ⚠️ **REQUIRED**: Set `env.TEMPORAL_ADDRESS` in your values

**What you need to do:**
1. Deploy Temporal separately (see [Temporal Workflow Engine Setup](#temporal-workflow-engine-setup))
2. Update your values to include `env.TEMPORAL_ADDRESS`
3. Upgrade Postiz

**Upgrade Command:**
```bash
# First, deploy Temporal (if not already done)
helm install temporal temporalio/temporal \
  --namespace temporal \
  --create-namespace \
  --values examples/temporal-self-hosted.yaml

# Then, upgrade Postiz
helm upgrade postiz-app oci://ghcr.io/gitroomhq/postiz-helmchart/charts/postiz-app \
  --set env.TEMPORAL_ADDRESS="temporal-frontend.temporal.svc.cluster.local:7233" \
  --reuse-values
```

See [Migration from v1.1.0](#migration-from-v110-embedded-temporal) for detailed upgrade instructions.

### To 1.0.0

This is the first major release of the Postiz Helm chart.

## Contributing

We welcome contributions to this chart. Please read our [Contributing Guide](CONTRIBUTING.md) before submitting a pull request.

## License

This chart is licensed under the Apache License 2.0. See the [LICENSE](LICENSE) file for details.
