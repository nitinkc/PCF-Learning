# Routing and Networking

## How Requests Reach Your Application

PCF uses **Gorouter** (a lightweight HTTP router) to direct traffic to applications.

```
Internet
  ↓
Load Balancer (external)
  ↓
Gorouter (multiple instances)
  ↓
Application Instances
```

## Routes and Domains

### Domains

PCF Foundation has one or more domains:

```
example.com         (default domain)
apps.example.com    (apps domain)
api.example.com     (API domain)
```

Each domain points to Gorouter IP (or load balancer frontend).

### Routes

Routes are URLs within those domains:

```
my-app.example.com    → Route for my-app
my-api.apps.example.com → Route for my-api
webhook.api.example.com → Route for webhook
```

### Creating Routes

Routes created automatically on `cf push`:

```bash
$ cf push my-app
# Automatically creates route: my-app.example.com

$ cf push my-app --hostname api
# Route: api.example.com

$ cf push my-app --domain apps.example.com --hostname my-service
# Route: my-service.apps.example.com
```

## Route Mapping Model

A single application can have **multiple routes**:

```bash
$ cf push my-app
Successfully pushed my-app to: my-app.example.com

# Map additional routes
$ cf map-route my-app example.com --hostname old-app
$ cf map-route my-app example.com --hostname v2-app

# Now all three URLs route to same app:
# my-app.example.com      → my-app instances
# old-app.example.com     → my-app instances
# v2-app.example.com      → my-app instances
```

This enables:
- **Gradual traffic migration** (route old → new)
- **A/B testing** (some requests to v1, some to v2)
- **Canary releases** (small % to new version)
- **Blue-green deployments** (switch between versions)

## Load Balancing

### Gorouter Round-Robin

Gorouter uses **round-robin** load balancing:

```
Request 1 → Instance 0
Request 2 → Instance 1
Request 3 → Instance 2
Request 4 → Instance 0 (cycle)
```

### Sticky Sessions

For session-dependent apps, use sticky sessions:

```bash
$ cf create-route example.com --hostname my-app
$ cf map-route my-app example.com --hostname my-app --random-port
```

Or configure application-level cookie-based sessions.

### Request Flow

```
1. $ curl my-app.example.com/api
2. DNS resolves: my-app.example.com → Load balancer IP
3. Load balancer forwards to Gorouter
4. Gorouter looks up route table:
   "my-app.example.com" → [instance0, instance1, instance2]
5. Gorouter picks instance (round-robin): instance1
6. Gorouter connects to instance1:8080
7. Request forwarded to app
8. App processes request
9. Response returned through Gorouter
10. Response sent to client
```

## HTTP Headers

Gorouter adds useful headers:

```
X-Forwarded-For: 203.0.113.1          # Original client IP
X-Forwarded-Proto: https               # Original protocol
X-Forwarded-Host: my-app.example.com   # Original host
X-B3-TraceId: abc123                   # Trace ID for logging
```

Your app can use these for logging, security, tracking:

```javascript
// Node.js
const clientIp = req.headers['x-forwarded-for'];
const originalProto = req.headers['x-forwarded-proto'];
```

## Networking

### Container Network

Each instance runs in isolated container:

```
┌─ Container 1 ──────┐
│ App listening:8080 │
│ IP: 10.0.0.5       │
└────────────────────┘

┌─ Container 2 ──────┐
│ App listening:8080 │
│ IP: 10.0.0.6       │
└────────────────────┘
```

Applications listen on `$PORT` (usually 8080):

```bash
export PORT=8080
node app.js
```

### Egress (Outbound Connections)

Applications can make outbound connections:

```javascript
// Node.js making outbound call
const response = await fetch('https://external-api.com/data');
```

**Network policies** restrict which destinations are allowed:

```bash
cf add-network-policy my-app my-cache-app --protocol tcp --port 6379
# my-app can now access my-cache-app on port 6379
```

### Service Binding Network

When bound to service, app connects via injected credentials:

```
Application → Environment Variables
           → DATABASE_URL=postgres://host:5432/db
           → Application code makes connection
           → Network policy permits
           → Connected to database
```

## Blue-Green Deployments

PCF makes blue-green deployments straightforward via routes:

### Setup Blue Instance

```bash
$ cf push my-app-blue
Created route: my-app-blue.example.com
Bound to: my-app-blue
```

### Deploy Green Instance

```bash
$ cf push my-app-green
Created route: my-app-green.example.com
Bound to: my-app-green
```

### Switch Traffic

```bash
# Both versions running separately:
# my-app-blue.example.com → Blue (v1)
# my-app-green.example.com → Green (v2)

# Map main route to green:
$ cf map-route my-app-green example.com --hostname my-app

# Test against green:
$ curl my-app-green.example.com

# Once verified, unmap from blue:
$ cf unmap-route my-app-blue example.com --hostname my-app

# Traffic now at:
# my-app.example.com → Green (v2)

# Blue remains available for rollback:
$ cf map-route my-app-blue example.com --hostname my-app
```

## Advanced: Custom Domains

### Add Domain to Foundation

```bash
# Operators add domain:
cf create-domain my-org example.com
```

### Map Route with Custom Domain

```bash
$ cf push my-app --domain example.com --hostname my-service
# Route: my-service.example.com
```

### Wildcard DNS

DNS configuration:

```
*.example.com  CNAME  elb.cloud-foundry.example.com
```

All routes automatically work without DNS entries.

## Security Considerations

### HTTPS / TLS

- Gorouter terminates TLS
- Connections to instances are HTTP
- Certificate management handled by platform

```
Internet (HTTPS)
    ↓ (TLS termination at Gorouter)
Gorouter (HTTP)
    ↓ (Internal network)
App Instances (HTTP :8080)
```

### Ports

- Applications must listen on `$PORT`
- Only port 8080 exposed by default
- Cannot listen on privileged ports (<1024)
- One application per port

---

Next: [Scaling & Availability](05-scaling.md)
