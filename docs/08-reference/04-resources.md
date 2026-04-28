# Resources & References

## Official Documentation

### Pivotal Cloud Foundry

- [Cloud Foundry Official Docs](https://docs.cloudfoundry.org/) - Authoritative source
- [Cloud Foundry Architecture](https://docs.cloudfoundry.org/concepts/architecture/) - System components and flow
- [Cloud Foundry API Reference](https://v3-apidocs.cloudfoundry.org/) - REST API documentation
- [CF CLI Reference](https://cli.cloudfoundry.org/en-US/) - All CLI commands

### cf4k8s (Cloud Foundry on Kubernetes)

- [cf4k8s GitHub Repository](https://github.com/cloudfoundry/cf-for-k8s) - Source code and releases
- [cf4k8s Installation Guide](https://github.com/cloudfoundry/cf-for-k8s/blob/main/README.md) - Detailed setup instructions
- [KinD Setup Guide](https://kind.sigs.k8s.io/docs/user/quick-start/) - For lightweight local K8s
- [Docker Desktop Kubernetes](https://docs.docker.com/desktop/kubernetes/) - Desktop K8s setup

### Buildpacks

- [Buildpacks Documentation](https://docs.cloudfoundry.org/buildpacks/) - General buildpack concepts
- [Buildpack API Specification](https://github.com/buildpacks/spec) - Detailed API/spec
- [Paketo Buildpacks](https://paketo.io/) - Modern buildpacks for Cloud Native Buildpack spec
- [Node.js Buildpack](https://github.com/cloudfoundry/nodejs-buildpack) - CF nodejs buildpack source

### Service Brokers

- [Open Service Broker API](https://github.com/openservicebrokerapi/servicebroker) - OSB specification
- [Service Broker Implementation Guide](https://docs.cloudfoundry.org/services/overview.html) - CF service integration
- [Service Broker Best Practices](https://docs.cloudfoundry.org/services/best-practices.html) - Design patterns

### Security & Authentication

- [UAA (User Account and Authentication)](https://github.com/cloudfoundry/uaa) - UAA source
- [Cloud Foundry RBAC](https://docs.cloudfoundry.org/concepts/roles.html) - Role-based access control
- [CredHub Documentation](https://github.com/cloudfoundry-incubator/credhub) - Credential management
- [Network Policies](https://docs.cloudfoundry.org/concepts/understand-cf-networking.html) - Container-to-container networking

## Learning Resources

### Books & Guides

- **"Cloud Foundry: The Definitive Guide"** by Mark Lonsdale - Comprehensive PaaS guide
- **"Beyond the Cloud"** by Mark Templeton - Enterprise cloud strategies
- **"Kubernetes in Action"** by Marko Luksa - K8s fundamentals (relevant for cf4k8s)

### Tutorials & Courses

- [Cloud Foundry Foundation Courses](https://training.cloudfoundry.org/) - Official training
- [Pluralsight Cloud Foundry Courses](https://www.pluralsight.com/courses/cloud-foundry-developer) - Video tutorials
- [Cloud Native Computing Foundation (CNCF)](https://www.cncf.io/training/) - Container orchestration training
- [Linux Academy PCF Path](https://linuxacademy.com/) - Hands-on labs

### Blogs & Articles

- [Cloud Foundry Blog](https://www.cloudfoundry.org/blog/) - Official updates
- [Pivotal Engineering Journal](https://pivotalcf.com/blog) - Technical deep-dives
- [Cloud Native Buildpacks Blog](https://cloud.google.com/blog/products/containers-kubernetes) - Buildpack innovations
- [Kubernetes Blog](https://kubernetes.io/blog/) - K8s updates (relevant for cf4k8s)

## Community & Support

### Chat & Forums

- [Cloud Foundry Slack](https://slack.cloudfoundry.org/) - Active community chat
- [GitHub Discussions](https://github.com/cloudfoundry/cf-for-k8s/discussions) - cf4k8s discussions
- [Stack Overflow](https://stackoverflow.com/questions/tagged/cloud-foundry) - Q&A tagged with `cloud-foundry`

### Events & Conferences

- [Cloud Foundry Summit](https://www.cloudfoundry.org/events/summit/) - Annual conference
- [KubeCon](https://www.cncf.io/kubecon-cloudnativecon-events/) - Container conferences
- [CNCF Webinars](https://www.cncf.io/webinars/) - Free technical sessions

### Developer Communities

- [Cloud Foundry Foundation](https://www.cloudfoundry.org/) - Official foundation
- [Buildpack Community](https://github.com/buildpacks/community) - Buildpack discussions
- [CNCF Community](https://www.cncf.io/community/) - Cloud native ecosystem

## Development Tools

### CLI & Management

- [CF CLI Releases](https://github.com/cloudfoundry/cli/releases) - Download latest CLI
- [BOSH CLI](https://bosh.io/docs/cli-v2/) - Deployment tool for CF
- [Cloud Foundry UAA CLI](https://github.com/cloudfoundry/cf-uaac) - UAA management
- [CF Curl](https://docs.cloudfoundry.org/cf-cli/getting-started.html#curl) - HTTP client for CF API

### Monitoring & Logging

- [Prometheus Cloud Foundry Exporter](https://github.com/cloudfoundry-community/prometheus-cf-exporter) - Metrics collection
- [Loggregator](https://docs.cloudfoundry.org/loggregator/architecture.html) - CF logging system
- [DataDog Cloud Foundry Integration](https://docs.datadoghq.com/integrations/cloud_foundry/) - APM integration
- [New Relic Cloud Foundry](https://docs.newrelic.com/docs/agents/cloud-foundry-agent/) - Application monitoring

### Testing & Simulation

- [Load Testing (Apache Bench)](https://httpd.apache.org/docs/current/programs/ab.html) - HTTP benchmarking
- [Load Testing (wrk)](https://github.com/wg/wrk) - Modern load testing
- [Chaos Engineering (Gremlin)](https://www.gremlin.com/) - Failure testing
- [Kind (Kubernetes in Docker)](https://kind.sigs.k8s.io/) - Local K8s clusters

## Language-Specific Resources

### Node.js

- [Node Buildpack Documentation](https://docs.cloudfoundry.org/buildpacks/node/index.html)
- [npm Shrinkwrap Guide](https://docs.npmjs.com/cli/shrinkwrap) - Dependency locking
- [Express.js Deployment](https://expressjs.com/en/advanced/best-practice-security.html) - Security best practices

### Python

- [Python Buildpack Documentation](https://docs.cloudfoundry.org/buildpacks/python/index.html)
- [Flask Deployment](https://flask.palletsprojects.com/en/latest/deploying/) - Production deployment
- [Django on Cloud Foundry](https://docs.run.pivotal.io/buildpacks/python/index.html) - Django setup

### Go

- [Go Buildpack Documentation](https://docs.cloudfoundry.org/buildpacks/go/index.html)
- [Go Best Practices](https://golang.org/doc/effective_go) - Language fundamentals
- [Cloud-Native Go](https://github.com/cloudfoundry-samples) - CF Go examples

### Java

- [Java Buildpack Documentation](https://docs.cloudfoundry.org/buildpacks/java/index.html)
- [Spring Cloud Services](https://spring.io/projects/spring-cloud) - Spring Cloud Framework
- [Spring Boot on Cloud Foundry](https://docs.run.pivotal.io/buildpacks/java/index.html) - Boot deployment

## Related Technologies

### Container Orchestration

- [Kubernetes Documentation](https://kubernetes.io/docs/) - K8s reference
- [Docker Documentation](https://docs.docker.com/) - Container fundamentals
- [Helm Package Manager](https://helm.sh/) - K8s package management

### DevOps & CI/CD

- [Cloud Foundry Concourse CI](https://concourse-ci.org/) - CF CI/CD tool
- [Cloud Native Buildpacks](https://buildpacks.io/) - OCI-compliant buildpacks
- [GitOps Principles](https://www.gitops.tech/) - Git-based deployment

### Observability

- [Prometheus Monitoring](https://prometheus.io/) - Metrics collection
- [Grafana Dashboards](https://grafana.com/) - Visualization
- [ELK Stack](https://www.elastic.co/what-is/elk-stack) - Elasticsearch/Logstash/Kibana
- [Jaeger Distributed Tracing](https://www.jaegertracing.io/) - Request tracing

## Quick Reference

### Must-Know URLs

```
CF API: https://api.cf-k8s.io
Dashboard: https://console.cf-k8s.io
Buildpacks: https://github.com/cloudfoundry/buildpacks
Source: https://github.com/cloudfoundry/cf-for-k8s
```

### Essential Commands

```bash
# Authentication
cf login -a https://api.cf-k8s.io

# Deployment
cf push my-app

# Monitoring
cf logs my-app
cf app my-app

# Management
cf services
cf routes
cf events my-app
```

### Key Concepts Quick Links

| Concept | Resource | Time |
|---------|----------|------|
| PaaS Model | [Cloud Foundry Concepts](https://docs.cloudfoundry.org/concepts/) | 30min |
| Buildpacks | [Buildpack Guide](https://docs.cloudfoundry.org/buildpacks/) | 1hr |
| Service Brokers | [OSB Specification](https://github.com/openservicebrokerapi/servicebroker) | 1.5hr |
| RBAC | [CF Roles](https://docs.cloudfoundry.org/concepts/roles.html) | 20min |
| Networking | [CF Networking](https://docs.cloudfoundry.org/concepts/understand-cf-networking.html) | 45min |

---

**Last Updated:** 2024

For the most current information, always check the [official Cloud Foundry documentation](https://docs.cloudfoundry.org/).
