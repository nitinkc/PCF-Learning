# Lab 2: Scaling and Availability

## Objectives

- Configure autoscaling policies
- Test instance failure recovery
- Monitor scaling behavior
- Understand health checks

## Prerequisites

- Lab 1 completed
- Application deployed

## Duration: ~45 minutes

## Part 1: Manual Scaling

```bash
# Start with 3 instances
$ cf scale my-app -i 3

# Watch instances scale
$ watch cf app my-app

# Add one more
$ cf scale my-app -i 4

# Scale down
$ cf scale my-app -i 2
```

## Part 2: Configure Health Checks

```bash
# Redeploy with HTTP health check
$ cf push my-app \
  --health-check-type http \
  --health-check-http-endpoint /health \
  -i 2

# Verify
$ cf app my-app
health checks:  http:/health
```

## Part 3: Test Failure Recovery

```bash
# Get list of instances
$ cf app my-app
instance #0: running
instance #1: running

# SSH into instance (if enabled)
$ cf ssh my-app -i 0

# Kill process in instance
$ pkill -f "node app.js"

# Exit SSH
$ exit

# Watch recovery
$ watch cf app my-app
# Within 30s instance should restart
```

## Part 4: Autoscaling Policy

Create `policy.json`:

```json
{
  "instance_min_count": 2,
  "instance_max_count": 6,
  "scaling_rules": [
    {
      "metric_type": "cpu",
      "threshold": 70,
      "operator": ">",
      "cool_down_seconds": 60,
      "adjustment": "+1"
    },
    {
      "metric_type": "memory",
      "threshold": 80,
      "operator": ">",
      "cool_down_seconds": 60,
      "adjustment": "+2"
    }
  ]
}
```

Apply policy:

```bash
# Create autoscaler policy
$ cf attach-asg-autoscaler-policy my-app policy.json

# Verify
$ cf get-autoscaler-policy my-app
Min instances:    2
Max instances:    6
CPU threshold:    70
Memory threshold: 80
```

## Part 5: Load Testing

Create load to trigger scaling:

```bash
# Install Apache Bench (macOS)
$ brew install httpd

# Or use curl in loop
$ for i in {1..1000}; do \
  curl -s https://my-app.cf-k8s.io > /dev/null & \
done

# Monitor scaling
$ watch -n 5 "cf app my-app | grep instance"

# Watch metrics
$ cf stats my-app
```

## Part 6: Memory Scaling

Update app to use more memory:

```javascript
// Add to app.js - memory consumer endpoint
app.get('/memory-stress', (req, res) => {
  const mb = 100;
  const size = mb * 1024 * 1024;
  const arr = new Array(size / 8);
  res.json({ allocated: mb + 'MB' });
});
```

Redeploy and test:

```bash
# Redeploy
$ npm install
$ cf push my-app

# Trigger memory scaling
$ while true; do curl -s https://my-app.cf-k8s.io/memory-stress; done &

# Watch scaling
$ watch cf stats my-app

# Should scale up when memory > 80%
```

## Verification

- ✅ Can manually scale instances
- ✅ Health checks configured
- ✅ Failed instances auto-recover
- ✅ Autoscaling policies work
- ✅ App scales based on metrics
- ✅ Minimum instances maintained

## Cleanup

```bash
# Remove autoscaler policy
cf detach-asg-autoscaler-policy my-app

# Scale back to 1
cf scale my-app -i 1
```

## Key Learnings

1. **Self-Healing**: CF automatically restarts failed instances
2. **Horizontal Scaling**: Easy to scale instances up/down
3. **Health Checks**: CPU-level monitoring ensures responsiveness
4. **Autoscaling**: Automatic scaling based on metrics
5. **Monitoring**: Real-time stats for making decisions

---

Next: [Lab 3: Service Binding](03-service-binding.md)
