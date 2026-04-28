# PCF vs Azure

## Service Model

| Aspect | PCF | Azure |
|--------|-----|-------|
| Level | PaaS | IaaS + PaaS |
| Deployment | Buildpacks | App Service, Kubernetes |
| Scaling | Built-in | App Service scale-out |
| Integration | Service Brokers | 300+ services |

## Deployment

### Azure Approach

```bash
# App Service (Azure PaaS)
az appservice plan create --name myplan --resource-group mygroup
az webapp create --resource-group mygroup --plan myplan --name myapp
az webapp deployment source config --resource-group mygroup --name myapp --repo-url https://...

# Alternative: Container Instances
az container create --resource-group mygroup --image myimage:latest --name myapp

# Alternative: Kubernetes (AKS)
az aks create --resource-group mygroup --name myaks
kubectl apply -f deployment.yaml
```

### PCF Approach

```bash
cf push my-app
# Done
```

## Services

### Azure Services

```bash
# Create database
az mysql server create --resource-group mygroup --name mydb

# Create cache
az redis create --resource-group mygroup --name mycache

# Connect manually
# Manage connection strings in code
```

### PCF Services

```bash
# Create database
cf create-service postgresql free my-db

# Create cache
cf create-service redis free my-cache

# Automatic connection injection
cf bind-service my-app my-db
```

## Terminology Mapping

| Azure | PCF | Purpose |
|-------|-----|---------|
| Resource Group | Organization | Logical grouping |
| Subscription | Foundation | Billing/access boundary |
| App Service Plan | Quota | Resource allocation |
| Virtual Network | Space | Network isolation |

## Cost Comparison

### Azure Pricing

```
App Service: $0.07-$1/hour per instance
Database: $0.12-$2/hour
Data transfer: $0.10-$0.20/GB
Storage: $0.024/GB
```

### PCF Pricing

```
Foundation: $10,000-$100,000/year
Per-memory allocation: included
Scaling: unlimited
```

## Integration

### Azure Ecosystem

- Tight integration with Microsoft stack
- SQL Server, Active Directory, Office 365
- Strong for enterprise Microsoft shops

### PCF Ecosystem

- Language/framework agnostic
- Integrates with any cloud provider
- Strong for polyglot environments

## When to Choose

### Choose PCF if:
- Prefer platform simplicity
- Multi-cloud strategy
- Non-Microsoft stack
- App deployment focus

### Choose Azure if:
- Existing Microsoft investment
- Need Active Directory integration
- Office 365/Dynamics integration needed
- SQL Server required

---

See [Comparison Matrix](04-comparison-matrix.md) for detailed feature comparison.
