# Troubleshooting Guide

## Common Deployment Issues

### "Failed to upload droplet"

**Symptoms:**
```
Error uploading application.
```

**Causes and Solutions:**

1. App too large
```bash
# Check size
du -sh .

# Exclude files
echo "node_modules/" >> .cfignore
echo "*.log" >> .cfignore
```

2. Network timeout
```bash
# Retry push
cf push my-app

# Use larger timeout
cf push my-app --push-timeout 300
```

3. Auth issue
```bash
# Re-authenticate
cf logout
cf login -a <API>
```

### "App keeps crashing"

**Check logs:**
```bash
$ cf logs my-app --recent
[APP...] SyntaxError: Unexpected token }
```

**Solutions:**
1. Fix application error (shown in logs)
2. Check environment variables: `cf env my-app`
3. Check buildpack: `cf app my-app | grep buildpack`

### "App won't start"

**Check status:**
```bash
$ cf app my-app
instances: 0/1
state: down

# Detailed reason
$ cf events my-app
instance 0: failed to start, code: 225
```

**Troubleshooting:**

```bash
# View logs
cf logs my-app --recent

# Check memory allocation
cf scale my-app -m 1G

# Restart
cf restart my-app

# Restage (rebuild)
cf restage my-app
```

## Scaling Issues

### "Max instances reached"

**Check quota:**
```bash
$ cf quota my-quota
memory: 10G
instances: 50 (in use: 48)
routes: 100

# Need higher quota
cf update-quota my-quota --instances 100
```

### "Scaling doesn't happen"

**Check autoscaler policy:**
```bash
$ cf get-autoscaler-policy my-app
# Review thresholds

# Check current metrics
cf stats my-app

# Manually scale to verify
cf scale my-app -i 5
```

## Service Issues

### "Service bind fails"

```bash
# Check service exists
$ cf services
name           service     plan   status
my-database    postgresql  free   create failed

# Wait for creation
$ cf service my-database
Status: The service instance is being created...

# If stuck
cf delete-service my-database
cf create-service postgresql free my-database
```

### "Can't connect to service"

**Check credentials injected:**
```bash
$ cf env my-app | grep -A 20 VCAP_SERVICES
DATABASE_URL=postgresql://...

# If not there
cf bind-service my-app my-database
cf restage my-app
```

**Test connection:**
```bash
cf ssh my-app
$ psql postgresql://user:pass@host/db
# Test query
```

## Routing Issues

### "404 Not Found"

**Verify route exists:**
```bash
$ cf routes
host    domain        space
my-app  cf-k8s.io     development

# Add if missing
cf map-route my-app cf-k8s.io --hostname my-app
```

**Check app is running:**
```bash
$ cf app my-app
instances: 1/1
state: running

# If not
cf restart my-app
```

### "502 Bad Gateway"

App is running but not responding to health checks.

**Check health endpoint:**
```bash
cf ssh my-app
$ curl localhost:$PORT/health
# Should return 200
```

**Fix health check:**
```bash
# Verify endpoint exists
# Add timeout if slow

cf push my-app \
  --health-check-http-endpoint /api/health \
  --health-check-invocation-timeout 10
```

## Performance Issues

### "App responding slowly"

**Check resource usage:**
```bash
$ cf stats my-app
cpu: 95%      # High CPU
memory: 450M of 512M  # High memory

# Solutions
cf scale my-app -m 1G   # More memory
cf scale my-app -i 3    # More instances
```

**Check logs for bottlenecks:**
```bash
cf logs my-app | grep "duration"
# Look for slow requests > 1000ms
```

### "Memory leak"

Monitor memory over time:

```bash
watch -n 10 "cf stats my-app | grep memory"

# Stop and restart to reset
cf restart my-app
```

**Fix:**
1. Update application code
2. Deploy new version: `cf push`

## Logging Issues

### "No logs appearing"

```bash
# Check if app is running
cf app my-app

# Check if logging to stdout
# Apps must log to stdout/stderr

# In Node.js
console.log('message')  // OK
fs.appendFile('/tmp/log.txt', ...)  // Lost

# Restart and check
cf restart my-app
cf logs my-app
```

### "Too many logs"

```bash
# Filter logs
cf logs my-app | grep ERROR

# Save to file
cf logs my-app --recent > app-logs.txt

# Limit verbosity in app
export DEBUG=false
cf set-env my-app DEBUG false
cf restage my-app
```

## CLI Issues

### "Cannot authenticate"

```bash
# Check API endpoint
cf api

# Login again
cf logout
cf login -a https://api.cf-k8s.io

# Use --skip-ssl-validation for self-signed certs
cf login -a https://api.cf-k8s.io --skip-ssl-validation
```

### "Timeout connecting to API"

```bash
# Test connectivity
ping api.cf-k8s.io

# Check API health
curl -k https://api.cf-k8s.io/healthz

# Try different network
# Use VPN if available
```

## Getting Help

### Check application details

```bash
cf app my-app
cf env my-app
cf services
cf events my-app
```

### View all logs

```bash
cf logs my-app --recent | tee debug.log
```

### Check platform status

```bash
# Check Cloud Controller
cf curl /v3/admin/info

# Check quota usage
cf org-quota my-org
```

### Common Error Codes

| Code | Meaning | Solution |
|------|---------|----------|
| 225 | Insufficient disk | `cf push -d 2G` |
| 233 | Insufficient memory | `cf scale -m 1G` |
| 241 | App crashed | Check logs, fix code |
| 243 | Health check failed | Fix health endpoint |
| 244 | Build timeout | Increase timeout |

---

For more help: Check [Cloud Foundry documentation](https://docs.cloudfoundry.org/) or community support.
