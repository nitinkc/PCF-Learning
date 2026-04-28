# Resource Management

## Memory Management

### Per-Application Memory

```bash
# Set 512MB per instance
$ cf push my-app -m 512M

# Set 2GB per instance
$ cf push my-app -m 2G

# Change running app
$ cf scale my-app -m 1G
```

Application gets memory as limit—can't exceed.

### Disk Allocation

Ephemeral disk (~1GB default):

```bash
# Set disk size
$ cf push my-app --disk 2G
```

Lost on every redeploy. For persistent storage, use services.

## Organization and Space Quotas

### Organization Quotas

```bash
# Create quota
$ cf create-quota large \
  --memory 100G \
  --routes 1000 \
  --service-instances 50

# Apply to org
$ cf set-quota my-org large

# View
$ cf quota large
...
memory: 100G
routes: 1000
service instances: 50
```

### Space Quotas

Further limit within org:

```bash
# Create space quota
$ cf create-space-quota dev-small \
  --memory 10G \
  --routes 100

# Apply to space
$ cf set-space-quota dev dev-small
```

## Monitoring Resource Usage

```bash
# Real-time usage
$ cf stats my-app
INSTANCE   STATE    CPU      MEMORY
#0         running  24.5%    234M of 512M
#1         running  15.2%    187M of 512M
#2         running  19.8%    245M of 512M

# Watch continuously
$ watch cf stats my-app

# Memory trends
$ cf stats my-app | awk '{print $4}' | grep M
234M
187M
245M
```

## Right-Sizing Applications

### For CPU-Bound Apps

More instances, less memory each:

```bash
# Instead of:
cf scale my-app -i 2 -m 2G   # 4GB total

# Better:
cf scale my-app -i 4 -m 512M # 2GB total, more parallelism
```

### For Memory-Hungry Apps

Fewer instances, more memory each:

```bash
$ cf scale my-app -i 2 -m 2G  # 2 big instances
```

### Finding Right Size

1. Deploy with 512MB
2. Monitor memory usage: `cf stats`
3. If consistently >80%: increase
4. If consistently <30%: decrease
5. Balance cost vs performance

## Autoscaling Based on Metrics

### CPU-Based Scaling

```json
{
  "instance_min_count": 2,
  "instance_max_count": 10,
  "scaling_rules": [{
    "metric_type": "cpu",
    "threshold": 70,
    "operator": ">",
    "cool_down_seconds": 60,
    "adjustment": "+1"
  }]
}
```

Scales up when CPU > 70% for 60 seconds.

### Memory-Based Scaling

```json
{
  "scaling_rules": [{
    "metric_type": "memory",
    "threshold": 85,
    "operator": ">",
    "cool_down_seconds": 60,
    "adjustment": "+2"
  }]
}
```

Scales up when memory > 85%.

### Custom Metrics

Extract from application logs:

```bash
# Parse response time from logs
$ cf logs my-app | grep duration | \
  awk -F'duration_ms":' '{print $2}' | \
  awk -F',' '{print $1}' | \
  awk '{sum+=$1; count++} END {print sum/count " ms"}'

# If > 500ms avg, scale up
```

## Cost Optimization

### Strategies

1. **Right-size**: Use monitoring to find minimum
2. **Autoscale**: Let platform scale with demand
3. **Consolidate**: Multiple small apps → single larger
4. **Schedule**: Off-peak scale down (if stateless)

### Cost Analysis

```bash
# Instance cost calculation
instances * memory * hourly_rate = cost

# Example: 3 instances x 512MB x $0.10/GB-hour = $0.15/hour
3 * 0.5 * 0.10 = $0.15

# Scale to 2:
2 * 0.5 * 0.10 = $0.10 (33% less)
```

---

See [Lab 2: Scaling & Availability](../06-labs/02-scaling-availability.md) for hands-on resource management.
