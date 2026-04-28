# Blue-Green Deployments

## Strategy

Blue-green deployment is a release technique that reduces downtime and risk by running two identical production environments.

```
┌─ Initial State ───────────┐
│ Users → Router → Blue     │ (current version)
│                           │
│ Green (staging)           │ (new version, not receiving traffic)
└───────────────────────────┘

┌─ After Testing ──────────┐
│ Users → Router ──┬──Blue  │ (old)
│                  └─Green  │ (new - ready)
└──────────────────────────┘

┌─ After Switch ────────────┐
│ Blue → archived           │ (rollback available)
│ Users → Router → Green    │ (current)
└──────────────────────────┘
```

## Implementation in PCF

### Step 1: Deploy New Version

```bash
# Current version as blue
$ cf push my-app-blue -i 3

# New version as green
$ cf push my-app-green -i 3
```

### Step 2: Route Main Domain

```bash
# Map main route to blue (currently receiving traffic)
$ cf map-route my-app-blue example.com --hostname my-app

# Now switch to green (no downtime - both temporarily receiving traffic)
$ cf map-route my-app-green example.com --hostname my-app

# Verify traffic split
$ for i in {1..10}; do curl my-app.example.com/version; done

# Remove from blue
$ cf unmap-route my-app-blue example.com --hostname my-app
```

## Advantages

| Aspect | Benefit |
|--------|---------|
| **Downtime** | Zero downtime during deployment |
| **Rollback** | Instant rollback if issues found |
| **Testing** | Full system test on production-like environment |
| **Safety** | Easy to go back if problems detected |
| **Stages** | Clear separation between old and new |

## Disadvantages

| Aspect | Issue |
|--------|-------|
| **Resources** | Need 2x resources temporarily |
| **Data** | Database schema changes need migration |
| **Complexity** | More steps than simple push |
| **Sessions** | Sticky sessions need user re-login |

## Use Cases

**Good for:**
- Stateless applications
- Quick deployments
- Safety-critical systems
- HA environments

**Consider alternatives for:**
- Very resource-constrained
- Complex data migrations
- Long-running processes

---

See [Lab 4: Blue-Green Deployment](../06-labs/04-blue-green-lab.md) for hands-on practice.
