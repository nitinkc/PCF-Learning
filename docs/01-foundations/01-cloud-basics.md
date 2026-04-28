# Cloud Computing Basics

## Quick Refresher for Advanced Developers

Since you already know cloud, this is a **quick context** section, not a tutorial.

## Service Models

```
┌──────────────────────────────────────────┐
│ SaaS: Applications (Salesforce, Office)  │
├──────────────────────────────────────────┤
│ PaaS: Platforms (PCF, Heroku, App Engine)│
├──────────────────────────────────────────┤
│ IaaS: Infrastructure (AWS, Azure, GCP)   │
├──────────────────────────────────────────┤
│ On-Prem: You manage everything           │
└──────────────────────────────────────────┘

Your responsibility ▼
```

### Where PCF Fits

| Layer | Provider | You Manage |
|-------|----------|-----------|
| IaaS | AWS/Azure/GCP | Applications, Container images, Config |
| **PaaS** | **PCF, Heroku** | **Applications, Source code** |
| SaaS | Salesforce, GitHub | Data |

## Deployment Models

- **Public Cloud**: Shared infrastructure (AWS, Azure, GCP public regions)
- **Private Cloud**: Your own infrastructure (often PCF on-prem, customer-managed cf4k8s)
- **Hybrid**: Combination of public and private

## Key Concepts You Already Know

### Containerization
- PCF uses containers internally (via buildpacks)
- You don't write Dockerfiles for PCF
- Platform manages container lifecycle

### Orchestration
- PCF has built-in orchestration (what Kubernetes does)
- cf4k8s **uses** Kubernetes as orchestrator
- You generally don't interact with Kubernetes directly

### Multi-Tenancy
- PCF supports multiple orgs and spaces
- Similar to AWS accounts or cloud projects
- Quota and RBAC per organization

### Auto-Scaling
- PCF can automatically scale instances
- Policy-based or manual
- Health checks for failed instances

## What's Different in PCF?

### Application-Centric Model
- In AWS: Think "I need 3 t2.medium instances with auto scaling"
- In PCF: Think "I have an app that needs instances"
- **Platform handles** instance type, networking, OS patching

### Service Brokers
- Consistent way to provision any service
- Database credentials automatically injected
- No manual connection string management

### Buildpacks
- Automatic language/framework detection
- Build process is standardized
- No need to create/maintain images

## Deployment Comparison

### AWS (IaaS)
```
git push → Triggers pipeline → Create AMI → 
Launch instances → Configure load balancers → 
Monitor + Scale → Done
```

### Kubernetes (Container Platform)
```
git push → Build image → Registry → Create deployment → 
Configure ingress → Configure HPA → Monitor → Done
```

### PCF (PaaS)
```
git push → Done
```

---

Next: [Container Orchestration Context](02-container-orchestration.md)
