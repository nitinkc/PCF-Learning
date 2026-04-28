# Canary Releases

## Strategy

Canary release gradually routes small percentage of traffic to new version. If metrics are good, gradually increase percentage until all traffic routes to new version.

```
Day 1:  98% Blue, 2% Green   (2 instances)
Day 2:  95% Blue, 5% Green   (5 instances)
Day 3:  90% Blue, 10% Green  (10 instances)
Day 4:  50% Blue, 50% Green  (rollout halfway)
Day 5:  0% Blue, 100% Green  (fully deployed)
```

## Implementation

### Manual Canary (Using Instance Counts)

```bash
# Deploy new version
$ cf push my-app-green -i 1   # 1 instance = ~9%

# Keep old version
$ cf scale my-app-blue -i 10  # 10 instances = ~91%

# Monitor metrics
$ cf stats my-app-green
$ cf stats my-app-blue

# Gradually increase green
$ cf scale my-app-green -i 3   # ~23% traffic
$ cf scale my-app-green -i 5   # ~33% traffic
$ cf scale my-app-green -i 10  # ~50% traffic

# Once stable, finish migration
$ cf scale my-app-blue -i 0    # No traffic to blue
$ cf delete my-app-blue        # Remove old
```

### Traffic Splitting (Advanced)

PCF supports weighted routing:

```bash
# Deploy both versions
$ cf push my-app-blue -i 1

# Configure traffic weighting (if supported)
$ cf route-service example.com \
  --hostname my-app \
  my-app-blue:0.9 \
  my-app-green:0.1   # 10% to green
```

## Monitoring During Canary

**Key Metrics:**

```bash
# Error rate
$ cf logs my-app-green | grep -i error | wc -l

# Response time (from app logs)
$ cf logs my-app-green --recent | grep "duration_ms" | \
  sed 's/.*duration_ms":\([0-9]*\).*/\1/' | \
  awk '{sum+=$1; count++} END {print sum/count " ms avg"}'

# Capacity
$ cf stats my-app-green | grep memory

# CPU usage
$ cf stats my-app-blue | grep cpu
```

## Advantages

| Aspect | Benefit |
|--------|---------|
| **Safety** | Minimal risk with small percentage |
| **Validation** | Real production data before full rollout |
| **Rollback** | Easy to revert if issues found |
| **Gradual** | Smooth transition for users |
| **Observation** | Time to monitor behavior |

## Disadvantages

| Aspect | Issue |
|--------|-------|
| **Duration** | Takes longer than blue-green |
| **Complexity** | More monitoring required |
| **Resources** | Run both versions during rollout |
| **User Impact** | Some users on old, some on new |

## Deciding Canary vs Blue-Green

**Use Canary if:**
- Changes may have unknown impact
- Want to validate with real traffic
- Risk of issues justifies slower rollout

**Use Blue-Green if:**
- Confident in changes (tested)
- Need quick deployment
- Can tolerate resource overhead
- Want immediate rollback

---

See [Lab 4: Blue-Green Deployment](../06-labs/04-blue-green-lab.md) for hands-on practice with traffic management.
