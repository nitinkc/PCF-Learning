# Scaling and Availability

## The Scaling Model

PCF enables both **vertical** and **horizontal** scaling, with horizontal being the recommended approach.

### Horizontal Scaling: Instances

Run multiple copies of your application:

```bash
# Scale to 5 instances
$ cf scale my-app -i 5

# Gorouter load balances across 5 running instances
# Each handles 1/5th of traffic
# If one fails, 4 still serving users
```

### Vertical Scaling: Memory

Allocate more resources per instance:

```bash
# Give each instance 2GB memory
$ cf scale my-app -m 2G

# Each instance has more memory
# Better for memory-intensive apps
# But can't scale beyond pod/node limit
```

**Best practice:** Horizontal over vertical. Scale instances, not memory.

## Auto-Scaling with Policies

PCF can automatically scale applications based on metrics:

```bash
# Enable autoscaling
$ cf create-app-autoscaler-policy my-app
{
  "instance_min_count": 2,
  "instance_max_count": 10,
  "scaling_rules": [
    {
      "metric_type": "cpu",
      "threshold": 80,
      "operator": ">",
      "cool_down_seconds": 60,
      "adjustment": "+1"
    },
    {
      "metric_type": "memory",
      "threshold": 90,
      "operator": ">",
      "cool_down_seconds": 60,
      "adjustment": "+2"
    }
  ]
}
```

**Scaling rules:**
- When CPU > 80% for 60 seconds → add 1 instance
- When memory > 90% for 60 seconds → add 2 instances
- Minimum 2 instances, maximum 10 instances

## Health Checks and Self-Healing

PCF automatically manages instance health.

### Three Health Check Types

#### 1. Health Endpoint (Recommended)

Application exposes endpoint that indicates health:

```bash
$ cf push my-app --health-check-type http --health-check-http-endpoint /health
```

**Your application:**
```javascript
app.get('/health', (req, res) => {
  // Check dependencies if needed
  res.status(200).json({ status: 'healthy' });
});
```

PCF checks endpoint every 30 seconds. Failing response → restart instance.

#### 2. Port Health Check

PCF simply checks if port is listening:

```bash
$ cf push my-app --health-check-type port
```

If app stops listening on `$PORT` → restart.

#### 3. Process Health Check

PCF verifies app process is running:

```bash
$ cf push my-app --health-check-type process
```

If process exits → restart.

### Health Check Configuration

```bash
# Specify endpoint and timeout
$ cf push my-app \
  --health-check-type http \
  --health-check-http-endpoint /status \
  --health-check-invocation-timeout 5 \
  --health-check-interval 30

# Health check fails if:
# - Response not 200-299 status
# - Timeout > 5 seconds
# - Process crashes
```

### Failure Cascade

```
1. App instance running
2. Health check fails
3. Diego marks instance unhealthy
4. Gorouter removes from routing
5. Diego restarts instance
6. Health check succeeds
7. Gorouter readds to routing
8. Traffic flows again
```

Entire process typically < 60 seconds.

## Availability Patterns

### 1. Multi-Instance Deployment

```bash
$ cf scale my-app -i 3
my-app now has 3 instances running
```

**Benefit:**
- One instance fails → 2/3 still serving users
- Gorouter removes failed instance immediately
- Diego automatically restarts failed instance

### 2. Geographical Redundancy

Deploy to multiple availability zones:

```
┌─ Zone A ─────────────────┐
│ my-app instance 0        │
│ my-app instance 1        │
└──────────────────────────┘

┌─ Zone B ─────────────────┐
│ my-app instance 2        │
│ my-app instance 3        │
└──────────────────────────┘

Load balancer / DNS points to both zones
```

### 3. Graceful Shutdown

Best practice: handle SIGTERM and drain connections:

```javascript
// Node.js example
process.on('SIGTERM', () => {
  console.log('SIGTERM received, shutting down gracefully');
  
  // Stop accepting new requests
  server.close(() => {
    console.log('Server closed');
    process.exit(0);
  });
  
  // Wait 30 seconds for existing requests
  setTimeout(() => {
    console.log('Forced shutdown');
    process.exit(1);
  }, 30000);
});
```

When Diego stops instance:
1. Sends SIGTERM
2. App has ~30 seconds to finish requests
3. If not shutdown, SIGKILL is sent
4. Gorouter already removed instance from routing

### 4. Circuit Breaker Pattern

Prevent cascading failures:

```javascript
// Pseudo-code: Circuit breaker for external API
const client = new CircuitBreaker(
  () => fetch('https://external-api.com/data'),
  { timeout: 3000, errorThresholdPercentage: 50 }
);

try {
  const data = await client.fire();
} catch (err) {
  // Return cached data or error message
  res.json({ data: cachedData });
}
```

## Resource Management

### Quotas: Org Level

Limit resources per organization:

```bash
# Quotas
$ cf create-quota large \
  --memory 100G \
  --routes 1000 \
  --service-instances 50

$ cf set-quota my-org large
```

**Org gets max:**
- 100GB total memory across all apps
- 1000 routes
- 50 service instances

### Quotas: Space Level

Further restrict within space:

```bash
$ cf create-space-quota dev-limited --memory 10G

$ cf set-space-quota dev dev-limited
```

Space can only use 10GB even if org has 100GB.

### Instance Sizing Strategy

| Instance Type | Best For | Memory |
|---------------|----------|--------|
| Micro | Static sites, APIs | 256MB |
| Small | Simple apps | 512MB |
| Medium | Standard apps | 1GB |
| Large | Memory-intensive | 2-4GB |
| XL | Big processes | 8GB+ |

Scale instances horizontally, not vertically.

## Monitoring Availability

### Viewing Instances

```bash
$ cf app my-app
name:    my-app
state:   started
instances: 3/3

instance   state       since                  cpu    memory
0          running     2024-04-28 14:22:33    25.1%  256M of 512M
1          running     2024-04-28 14:22:35    18.5%  245M of 512M
2          running     2024-04-28 14:22:34    22.3%  251M of 512M
```

### Monitoring Logs

```bash
# Tail logs in real-time
$ cf logs my-app

# View recent logs
$ cf logs my-app --recent

# Logs from all instances aggregated
# Marked with instance index: [APP/instance]
```

### Events

```bash
$ cf events my-app
time                        type        actor      description
2024-04-28T14:22:00.000Z    audit.app.update   user@example.com   state changed to started
2024-04-28T14:22:05.000Z    app.crash   index 0    app instance crashed
2024-04-28T14:22:07.000Z    app.update  diego     app instance added
```

---

Next: [Local Setup](../03-setup-local/01-prerequisites.md)
