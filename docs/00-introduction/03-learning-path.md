# Learning Path

## Your Journey to PCF Mastery

This guide is structured for your specific background: **advanced cloud developer new to PCF**.

### Timeline
- **Complete Foundation**: 1-2 hours
- **Local Setup**: 30 minutes
- **Labs 1-3**: 2-3 hours
- **Advanced Topics**: 2-3 hours (optional)
- **Labs 4-6**: 3-4 hours (optional)

**Total: 8-13 hours to mastery**

## Progressive Learning Structure

### Phase 1: Context & Setup (1.5 hours)

Since you know cloud, we'll quickly review:

1. **[Cloud Basics Review](../01-foundations/01-cloud-basics.md)** (15 min)
   - How PCF fits in cloud ecosystem
   - Quick refresher on PaaS model

2. **[Container Orchestration Context](../01-foundations/02-container-orchestration.md)** (15 min)
   - How PCF compares to Kubernetes
   - Why cf4k8s matters

3. **[Local Setup](../03-setup-local/01-prerequisites.md)** (45 min)
   - Install cf4k8s
   - Configure CF CLI
   - Verify installation

### Phase 2: Core Concepts (Mandatory, 2 hours)

These are unique to PCF—no skipping:

1. **[PCF Architecture](../02-core-concepts/01-architecture.md)** (30 min)
   - Components and how they interact
   - Deployment structure

2. **[Apps & Buildpacks](../02-core-concepts/02-apps-buildpacks.md)** (30 min)
   - How apps are deployed
   - Buildpack magic

3. **[Services & Service Brokers](../02-core-concepts/03-services.md)** (30 min)
   - Binding and credential management
   - How services work

4. **[Routing & Networking](../02-core-concepts/04-routing.md)** (20 min)
   - How requests reach your app
   - Blue-green capability

### Phase 3: Hands-On Labs 1-3 (2.5 hours)

Apply learning by doing:

1. **[Lab 1: Deploy First App](../06-labs/01-deploy-first-app.md)** (30 min)
   - Push a simple app
   - View logs and metrics
   - Scale instances

2. **[Lab 2: Scaling & Availability](../06-labs/02-scaling-availability.md)** (45 min)
   - Configure autoscaling
   - Test health checks
   - Handle failures

3. **[Lab 3: Service Binding](../06-labs/03-service-binding.md)** (45 min)
   - Create a service
   - Bind to app
   - Use injected credentials

### Phase 4: Strategic Comparison (Optional, 1 hour)

Understand what makes PCF unique:

1. **[PCF vs AWS](../07-pcf-vs-cloud-providers/01-vs-aws.md)** (15 min)
2. **[PCF vs Azure](../07-pcf-vs-cloud-providers/02-vs-azure.md)** (15 min)
3. **[PCF vs GCP](../07-pcf-vs-cloud-providers/03-vs-gcp.md)** (15 min)
4. **[Comparison Matrix](../07-pcf-vs-cloud-providers/04-comparison-matrix.md)** (15 min)

### Phase 5: Advanced Topics & Labs (Optional, 7 hours)

Go deeper:

1. **[Deployment Patterns](../04-deployment-patterns/01-blue-green.md)** (1 hour)
   - Blue-green strategy
   - Canary releases
   - Service discovery

2. **[Advanced Topics](../05-advanced-topics/01-resource-management.md)** (2 hours)
   - Resource management
   - Security & RBAC
   - Monitoring & logging
   - Custom buildpacks

3. **[Lab 4: Blue-Green](../06-labs/04-blue-green-lab.md)** (1 hour)
   - Implement zero-downtime deployment

4. **[Lab 5: Custom Buildpack](../06-labs/05-custom-buildpack.md)** (1.5 hours)
   - Build custom build process

5. **[Lab 6: Monitoring](../06-labs/06-monitoring-lab.md)** (1.5 hours)
   - Set up observability

## Recommended Learning Paths by Goal

### Path A: "I want mastery in minimum time" ⚡
1. Phase 1: Setup (1.5h)
2. Phase 2: Core concepts (2h)
3. Phase 3: Labs 1-3 (2.5h)
4. **Total: 6 hours → Productive**

### Path B: "I need complete understanding" 📚
1. Phase 1: Setup (1.5h)
2. Phase 2: Core concepts (2h)
3. Phase 3: Labs 1-3 (2.5h)
4. Phase 4: Cloud provider comparison (1h)
5. Phase 5: Advanced + Labs (7h)
6. **Total: 13.5 hours → Expert**

### Path C: "I'm comparing PCF with alternatives" 🤔
1. Phase 4: Comparisons (1h)
2. Phase 1: Setup (1.5h)
3. Phase 2: Core concepts (2h)
4. Phase 3: Labs 1-3 (2.5h)
5. **Total: 7 hours → Informed decision**

## How to Use This Guide

### 📖 Reading Format
- **Core sections** explain concepts with examples
- **Code blocks** show practical commands
- **Diagrams** visualize architecture
- **Labs** are hands-on practice

### 🧪 Lab Format
Each lab includes:
- **Objectives**: What you'll learn
- **Prerequisites**: What you need first
- **Step-by-step instructions**: Commands to run
- **Verification**: How to confirm success
- **Troubleshooting**: Common issues and fixes

### 📋 Reference
- [CLI Commands](../08-reference/01-cli-commands.md) for quick lookups
- [Manifest Reference](../08-reference/02-manifest-reference.md) for deployment files
- [Troubleshooting Guide](../08-reference/03-troubleshooting.md) for common issues

## Success Criteria

You've mastered PCF when you can:

1. ✅ Deploy an app with `cf push`
2. ✅ Explain PCF vs Kubernetes vs IaaS
3. ✅ Create and bind a service
4. ✅ Implement blue-green deployment
5. ✅ Monitor logs and metrics
6. ✅ Understand when to use PCF vs alternatives
7. ✅ Deploy multi-instance applications with scaling

## Next Steps

Ready to start? → **[Phase 1: Cloud Basics Review](../01-foundations/01-cloud-basics.md)**

Or jump to specific areas:
- 🔧 [Local Setup](../03-setup-local/01-prerequisites.md)
- 💻 [First Lab](../06-labs/01-deploy-first-app.md)
- 🆚 [Comparisons](../07-pcf-vs-cloud-providers/04-comparison-matrix.md)

---

*This learning path is designed for you. Feel free to adjust based on your needs. If something feels too basic, skip ahead. If you need more detail, use the reference section.*
