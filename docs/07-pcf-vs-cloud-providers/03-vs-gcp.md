# PCF vs GCP

## Service Model

| Aspect | PCF | GCP |
|--------|-----|-----|
| Level | PaaS | IaaS + PaaS |
| Deployment | Buildpacks | App Engine, Cloud Run, GKE |
| Scaling | Built-in | Managed autoscaling |
| Integration | Service Brokers | Cloud services |

## Deployment Comparison

### GCP Approach

```bash
# App Engine (GCP PaaS)
gcloud app create
gcloud app deploy

# Cloud Run (container-first)
gcloud run deploy --image gcr.io/project/image:tag

# Kubernetes (GKE)
gcloud container clusters create my-cluster
kubectl apply -f deployment.yaml
```

### PCF Approach

```bash
cf push my-app
# Buildpack auto-detects language, builds, deploys
```

## App Engine vs PCF

### App Engine

Pros:
- True serverless
- Automatic scaling to zero
- Fine-grained pricing

Cons:
- Limited customization
- Startup latency
- Framework constraints

### PCF

Pros:
- Full platform control
- Instant scaling
- Any language/framework

Cons:
- Minimum scaling cost
- Requires infrastructure

## Cloud Run vs PCF

### Cloud Run

Uses container images:

```dockerfile
FROM node:18
COPY . .
RUN npm install
EXPOSE 8080
CMD ["node", "app.js"]
```

```bash
gcloud run deploy my-app --image my-image:latest
```

### PCF

Uses buildpacks:

```bash
cf push my-app
```

## Scaling

### Both have autoscaling

GCP:
```yaml
autoscaling:
  minInstances: 1
  maxInstances: 100
  targetCpuUtilization: 0.6
```

PCF:
```json
{
  "instance_min_count": 1,
  "instance_max_count": 100,
  "scaling_rules": [{
    "metric_type": "cpu",
    "threshold": 60
  }]
}
```

## Services Integration

### GCP Services

```bash
# Create Cloud SQL
gcloud sql instances create my-db

# Create Datastore
gcloud datastore create-indexes

# Connection in code
from google.cloud import sql
client = sql.Client()
```

### PCF Services

```bash
# Create service
cf create-service postgresql free my-db

# Auto-injected
DATABASE_URL=postgresql://...
```

## Cost Model

### GCP Pricing

```
App Engine: $0.05/instance-hour
Cloud Run: $0.00002400 per GB-second
Data transfer: $0.12/GB
Storage: $0.020/GB
```

Advantages:
- Pay per invocation for serverless
- Can scale to zero

### PCF Pricing

```
Foundation: subscription
Scaling: unlimited included
Per-memory: no additional cost
```

## Ecosystem

### GCP Strengths

- Data analytics (BigQuery, Dataflow)
- Machine learning (Vertex AI)
- Cloud-native tools (Kubernetes, Anthos)
- Google Cloud integration

### PCF Strengths

- Application deployment simplicity
- Multi-cloud flexibility
- Any language/framework
- Mature production-proven platform

## When to Choose

### Choose PCF if:
- Focus on application deployment
- Need platform simplicity
- Multi-cloud strategy
- Any language/framework

### Choose GCP if:
- Google Cloud services integration
- Data analytics workloads
- Serverless preference
- Machine learning needs

---

See [Comparison Matrix](04-comparison-matrix.md) for detailed feature comparison.
