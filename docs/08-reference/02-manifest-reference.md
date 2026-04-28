# Manifest Reference

## Basic Manifest

```yaml
---
applications:
  - name: my-app
    instances: 3
    memory: 512M
    disk_quota: 1G
    routes:
      - route: my-app.cf-k8s.io
    buildpack: nodejs_buildpack
    command: npm start
```

## Complete Manifest

```yaml
---
applications:
  # Application info
  - name: my-app
    instances: 3
    memory: 512M
    disk_quota: 1G
    
    # Routing
    routes:
      - route: my-app.cf-k8s.io
      - route: my-app-backup.cf-k8s.io  # Multiple routes
    
    # Process types
    processes:
      - type: web
        instances: 3
        memory: 512M
      - type: worker
        instances: 2
        memory: 256M
    
    # Build and start
    buildpack: nodejs_buildpack
    command: npm start
    
    # Health checks
    health-check-type: http
    health-check-http-endpoint: /health
    health-check-invocation-timeout: 5
    health-check-interval: 30
    
    # Environment variables
    env:
      NODE_ENV: production
      LOG_LEVEL: info
      API_KEY: "{{ secrets API_KEY }}"
    
    # Service bindings
    services:
      - my-database
      - my-redis-cache
    
    # Metadata
    metadata:
      labels:
        env: production
        cost-center: engineering
      annotations:
        owner: platform-team
    
    # Stack (optional)
    stack: cflinuxfs4
    
    # Timeout (push)
    timeout: 180
    
    # Random route
    random-route: false
    
    # No-start (deploy but don't start)
    no-route: false
```

## Environment Variables

```yaml
env:
  SIMPLE_VAR: value
  DATABASE_URL: postgresql://user:pass@host:5432/db
  PORT: 8080
  DEBUG: "true"
  NUMBERS: "12345"
```

Access in application:

```javascript
const port = process.env.PORT || 3000;
const dbUrl = process.env.DATABASE_URL;
const debug = process.env.DEBUG === 'true';
```

## Services

```yaml
services:
  - my-database
  - my-cache
  - my-external-api  # User-provided service
```

Available services from marketplace:

```bash
cf marketplace
```

## Routes and Domains

```yaml
routes:
  # Simple route
  - route: my-app.cf-k8s.io
  
  # With hostname
  - route: example.com
    host: my-app
  
  # Multiple routes
  - route: my-app.cf-k8s.io
  - route: old-app.cf-k8s.io  # For migration
  - route: example.com
    host: api
```

## Buildpacks

```yaml
# Single buildpack (auto-detect if not specified)
buildpack: nodejs_buildpack

# Multiple buildpacks
buildpacks:
  - nodejs_buildpack
  - custom_buildpack

# Specific buildpack by URL
buildpack: https://github.com/org/buildpack.git#branch
```

Available buildpacks:

```bash
cf buildpacks
```

## Health Checks

```yaml
# HTTP health check
health-check-type: http
health-check-http-endpoint: /health
health-check-invocation-timeout: 5    # seconds
health-check-interval: 30              # seconds

# Port health check  
health-check-type: port

# Process health check
health-check-type: process
```

## Processes

Define different process types (web, worker, etc.):

```yaml
processes:
  - type: web
    instances: 3
    memory: 512M
    command: npm start
    env:
      ROLE: web
  
  - type: worker
    instances: 2
    memory: 256M
    command: npm run worker
    env:
      ROLE: worker
  
  - type: scheduler
    instances: 1
    memory: 128M
    command: npm run scheduler
```

Deploy specific process:

```bash
cf push my-app --process-type worker
```

## Metadata

```yaml
metadata:
  labels:
    environment: production
    team: platform
    cost-center: engineering
  
  annotations:
    description: "Production API server"
    owner: "team-email@example.com"
    support-link: "https://wiki.example.com/my-app"
```

## Variables and Secrets

```yaml
env:
  PUBLIC_KEY: "{{ variables public_key }}"
  DATABASE_URL: "{{ secrets DATABASE_URL }}"
  API_TOKEN: "{{ credential TOKEN }}"
```

Must be substituted at push time:

```bash
cf push --var PASSWORD=secret --var-file vars.yml
```

## Inheritance and Defaults

```yaml
---
default: &defaults
  instances: 2
  memory: 512M
  buildpack: nodejs_buildpack

applications:
  - name: my-app-prod
    <<: *defaults
    instances: 5       # Override default
    routes:
      - route: api.example.com
  
  - name: my-app-staging
    <<: *defaults
    routes:
      - route: api-staging.example.com
```

## Common Patterns

### Deploy Multiple Instances

```yaml
applications:
  - name: api
    instances: 3
    memory: 512M
```

### Blue-Green

```yaml
applications:
  - name: my-app-blue
    instances: 3
    routes:
      - route: my-app.cf-k8s.io
  
  - name: my-app-green
    instances: 3
    routes:
      - route: my-app-green.cf-k8s.io
```

### Microservices

```yaml
applications:
  - name: api-service
    instances: 2
    memory: 256M
  
  - name: web-service
    instances: 3
    memory: 512M
  
  - name: worker-service
    instances: 1
    memory: 128M
```

---

See [CLI Commands](01-cli-commands.md) for deployment commands.
