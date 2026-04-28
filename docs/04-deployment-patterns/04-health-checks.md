# Health Checks and Auto-Healing

## How PCF Ensures Availability

PCF automatically monitors applications and restarts failed instances through health checks.

## Health Check Types

### 1. HTTP Health Check (Recommended)

Application exposes endpoint returning 200-299 status:

```bash
$ cf push my-app \
  --health-check-type http \
  --health-check-http-endpoint /health
```

Application code:

```javascript
app.get('/health', (req, res) => {
  // Check critical dependencies
  const isHealthy = checkDatabase() && checkCache();
  
  if (isHealthy) {
    res.status(200).json({ status: 'ok' });
  } else {
    res.status(503).json({ status: 'degraded' });
  }
});
```

PCF checks every 30 seconds. If:
- Status not 200-299 → restart instance
- Timeout > 5 seconds → restart instance
- Connection refused → restart instance

### 2. Port Health Check

Simply checks if application is listening:

```bash
$ cf push my-app --health-check-type port
```

PCF checks if port ($PORT) is listening. Works but doesn't verify true health.

### 3. Process Health Check

Verifies app process is running:

```bash
$ cf push my-app --health-check-type process
```

If process exits → restart. Basic level of checking.

## Configuration

```bash
# Custom health check configuration
$ cf push my-app \
  --health-check-type http \
  --health-check-http-endpoint /api/health \
  --health-check-invocation-timeout 5 \
  --health-check-interval 30

# Check config
$ cf app my-app
health checks: http GET /api/health
invocation_timeout: 5
interval: 30
```

## Failure Recovery Flow

```
Instance Running
    ↓
Health Check Fails
    ↓
Diego marks UNHEALTHY (immediately)
    ↓
Gorouter removes from routing (within 5 seconds)
    ↓
Diego restarts instance (within 10 seconds)
    ↓
Health check succeeds
    ↓
Gorouter readds to routing
    ↓
Instance Running (Normal)
```

**Total time:** Typically < 30 seconds

## Application Lifecycle Hooks

For graceful shutdown:

```javascript
// Node.js example
process.on('SIGTERM', async () => {
  console.log('SIGTERM: Starting graceful shutdown');
  
  // Stop accepting new requests
  server.close();
  
  // Wait for existing requests to finish (up to 30s)
  setTimeout(() => {
    console.log('Forcing shutdown');
    process.exit(1);
  }, 30000);
});
```

Deployment sequence:

1. New instance starts (running old code)
2. Health checks pass
3. Gorouter adds to routing
4. New instance receives traffic
5. Diego sends SIGTERM to old instance
6. Old instance waits for requests to finish
7. Old instance exits cleanly
8. Diego removes old instance

## Multiple Health Checks

Configure multiple checks for redundancy:

```bash
# HTTP endpoint
$ cf push my-app --health-check-type http

# Also create a separate liveness check
$ cf ssh my-app -c "curl localhost:$PORT/health"

# Or use monitoring to supplement
# Application exports Prometheus metrics
```

## Monitoring Health

```bash
# View health check status
$ cf events my-app
2024-04-28 14:30:00 app.crash        instance 0: app instance exited, restarting
2024-04-28 14:30:05 app.update       diego: app instance added

# Detailed investigation
$ cf logs my-app --recent | grep -i health
[APP/PROC...] Health check status: passed
[APP/PROC...] Health check status: failed
```

## Best Practices

1. **Implement /health Endpoint**: Don't rely on port check
2. **Check Dependencies**: Verify DB, cache, services in health endpoint
3. **Avoid False Positives**: Health shouldn't return 200 if degraded
4. **Reasonable Timeouts**: 5 second timeout is usually adequate
5. **Logging**: Log health check results
6. **Circuit Breakers**: Fail fast rather than hang

## Common Health Check Issues

### App slow to startup

```bash
# Increase timeout
$ cf set-env my-app CF_HEALTH_CHECK_TIMEOUT 10
```

### Health check endpoint throws errors

Fix:

```javascript
app.get('/health', (req, res) => {
  try {
    res.status(200).json({ ok: true });
  } catch (e) {
    // Catch ALL errors, always return response
    res.status(200).json({ ok: false, msg: e.message });
  }
});
```

### Dependencies slow/timing out

Use circuit breaker in health check:

```javascript
const db = require('./db');

app.get('/health', async (req, res) => {
  try {
    // Try with short timeout
    const health = await Promise.race([
      db.query('SELECT 1'),
      new Promise((_, reject) => 
        setTimeout(() => reject('timeout'), 1000)
      )
    ]);
    res.json({ status: 'ok' });
  } catch (e) {
    // Return unhealthy but don't crash
    res.status(503).json({ status: 'degraded' });
  }
});
```

---

See [Lab 2: Scaling & Availability](../06-labs/02-scaling-availability.md) for hands-on testing of health checks.
