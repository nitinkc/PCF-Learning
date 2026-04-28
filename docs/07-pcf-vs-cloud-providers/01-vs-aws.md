# PCF vs AWS

## Service Model

| Aspect | PCF | AWS |
|--------|-----|-----|
| Level | PaaS | IaaS |
| Abstraction | Applications | Infrastructure |
| Deployment | `cf push` | CloudFormation, Terraform |
| Scaling | Automatic, app-aware | EC2 ASG, ECS, Fargate |
| Networking | Routes + Gorouter | VPC, Security Groups, ALB |

## Application Deployment

### AWS Approach

```
Source → Build (CodeBuild) → Image (ECR) → Provision (EC2/ECS) → Deploy → Run
1. Manage Docker image
2. Manage container registry
3. Manage orchestration
4. Manage networking
```

### PCF Approach

```
Source → Push → Buildpack → Build → Run (Routing automatic)
1. Buildpack handles images
2. Platform manages registry
3. Platform orchestrates
4. Routing automatic
```

## Services

### RDS (AWS)

```bash
# Manual steps
1. Create RDS instance
2. Configure security group
3. Create database
4. Get connection string
5. Set in app environment
6. Manage backup, patching

# In code
const dbUrl = process.env.DB_URL;
```

### Service Brokers (PCF)

```bash
# Automated
$ cf create-service postgresql free my-db
$ cf bind-service my-app my-db

# Credentials injected automatically
const dbUrl = process.env.DATABASE_URL;
```

## Scaling

### AWS

```bash
# Setup ASG
1. Create launch template
2. Create ASG
3. Set min/max instances
4. Configure metrics
5. Set scaling policies
6. Monitor CloudWatch

# Manual scaling
aws autoscaling set-desired-capacity --auto-scaling-group-name my-asg --desired-capacity 5
```

### PCF

```bash
# Simple commands
cf scale my-app -i 5              # Immediate
cf create-autoscaler-policy ...   # Metric-based
cf attach-asg-autoscaler-policy my-app policy.json
```

## Cost Model

### AWS

```
Per-usage billing:
- EC2: $0.10/hour per instance
- RDS: $0.15/hour per database
- Data transfer: $0.10/GB
- Storage: $0.05/GB
```

Advantages:
- Pay only for what you use
- Scale down to zero (serverless)
- Cost optimization options (spot instances)

### PCF

```
Subscription-based:
- Foundation license: $X/month
- Cost per GB-memory: included
- Unlimited scaling included
```

Advantages:
- Predictable costs
- No per-instance charges
- Scaling included

## Learning Curve

### AWS
- Extensive (VPC, IAM, RDS, EC2, networking)
- 6-12 months to proficiency
- Many services to learn

### PCF
- Moderate (apps, services, scaling)
- 1-2 months to proficiency
- Fewer concepts to master

## When to Choose

### Choose PCF if:
- Developer velocity is priority
- Team doesn't have DevOps expertise
- Want PaaS simplicity
- Multi-cloud strategy
- Internal platform requirement

### Choose AWS if:
- Need specific AWS services
- Infrastructure customization needed
- Existing AWS investment
- Cost optimization critical
- Specialized compute needs

## Hybrid Approach

Use both:

```
AWS:
- Hosting infrastructure
- Managed services (RDS, S3, etc.)
- Networking setup

PCF:
- Application deployment
- Service integration
- Developer platform
```

---

See [Comparison Matrix](04-comparison-matrix.md) for detailed feature comparison.
