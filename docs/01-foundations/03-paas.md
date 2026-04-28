# Platform as a Service (PaaS) Model

## The PaaS Contract

PaaS says: "Give us your application code, we'll handle the rest."

The platform manages:
- ✅ Infrastructure provisioning
- ✅ Operating system updates
- ✅ Runtime environments (Ruby, Python, Node, Java, etc.)
- ✅ Scaling and load balancing
- ✅ Health checking and recovery
- ✅ Logging and monitoring

You focus on:
- ✅ Application code
- ✅ Application configuration
- ✅ Data models and schemas

## PaaS Abstraction Levels

```
IaaS (AWS, Azure, GCP)
↓
"Here's a VM"
↓
Kubernetes
↓
"Here's a container orchestrator"
↓
PCF / Cloud Foundry
↓
"Here's your deployed app"
↑
Highest abstraction
```

## Why This Matters

### Cost of Lower Abstraction

More control = More decisions = More work:

- **IaaS**: 50+ decisions per deployment
- **Kubernetes**: 30+ decisions per deployment
- **PCF**: 3-5 decisions per deployment

### Cost of Higher Abstraction

Less flexibility = Less control:

- **IaaS**: Can do anything
- **Kubernetes**: Can do almost anything
- **PCF**: Can do common things

## PaaS Characteristics

### 1. Application-Centric

Focus on applications, not infrastructure:

```bash
# IaaS thinking
"I need 3 c5.xlarge instances in us-west-2a"

# Kubernetes thinking
"I need a deployment with 3 replicas of my image"

# PCF thinking
"I need to deploy my application"
```

### 2. Opinionated Defaults

PCF makes good default choices:

| Choice | Decision | Why |
|--------|----------|-----|
| Scaling | CPU-based autoscaling | Works for most apps |
| Health | Liveness checks | Self-healing |
| Config | Environment variables | Simple, standard |
| Storage | Ephemeral (stateless) | Scale horizontally |

You **can** override defaults if needed.

### 3. Service Integration

Services (databases, caches) integrate seamlessly:

```bash
# In IaaS: Manage connection strings, secrets, networking
# In Kubernetes: ConfigMaps, Secrets, service discovery
# In PCF: Automatic injection via service binding

$ cf bind-service my-app my-database
# Done. Your app receives: DATABASE_URL, DB_USER, DB_PASS
```

### 4. Developer Velocity

Push code, get production:

```bash
$ git push heroku main
# 30 seconds later
$ curl https://my-app.herokuapp.com
# ✅ Running
```

## Trade-Offs: The PaaS Contract

### What You Gain

- 🚀 Developer velocity (days instead of weeks to production)
- 🎯 Reduced operational complexity
- 🔄 Faster iteration cycles
- 🛡️ Built-in security and compliance
- 📊 Included monitoring and logging

### What You Lose

- 🎛️ Infrastructure customization
- 📦 Fine-grained control
- 🔧 Custom container orchestration
- 💰 Potential cost efficiency (vs IaaS)

## When to Choose PaaS (PCF)

### Good Fit ✅

1. **Microservices Architecture**
   - Many services need rapid deployment
   - Service binding matches microservices design

2. **Developer-First Organizations**
   - Team productivity is priority
   - Less infrastructure expertise available

3. **Internal Developer Platforms**
   - Building platforms for other developers
   - Abstraction layers provide consistency

4. **SaaS Multi-Tenancy**
   - Need isolation and quota management
   - Orgs/spaces provide natural boundaries

5. **Rapid Experimentation**
   - New features/products need fast deployment
   - Overhead must be minimal

### Consider Alternatives ⚠️

1. **Highly Custom Infrastructure**
   - IaaS with custom orchestration

2. **Very Cost-Sensitive**
   - Bare metal or IaaS (spot instances)

3. **Specialized Compute**
   - GPU, custom hardware
   - IaaS gives more options

4. **Full Control Priority**
   - Bare metal or Kubernetes with custom controllers

## PCF as PaaS

PCF implements the PaaS model through:

- **Buildpacks**: Automatic build and deployment
- **Application model**: Simple deployment unit
- **Service brokers**: Service integration
- **Routes**: Built-in load balancing
- **Orgs/Spaces**: Multi-tenancy
- **Quotas/RBAC**: Resource and access control

---

Next: [PCF Architecture](../02-core-concepts/01-architecture.md)
