# Lab 4: Blue-Green Deployment

## Objectives

- Deploy blue version (current)
- Deploy green version (new)
- Switch traffic with zero downtime
- Rollback if needed

## Prerequisites

- Labs 1-3 completed
- Application working

## Duration: ~1 hour

## Part 1: Deploy Blue Instance

```bash
# Current app becomes "blue"
$ cf push my-app-blue -m 512M -i 2
Pushing app my-app-blue...
OK

# Map main route
$ cf map-route my-app-blue cf-k8s.io --hostname my-app
Mapping route my-app-blue.cf-k8s.io to my-app-blue...
OK

# Verify
$ cf routes | grep my-app
my-app-blue.cf-k8s.io    my-app-blue
my-app.cf-k8s.io         my-app-blue
```

## Part 2: Deploy Green Instance (New Version)

Update `app.js` to show version:

```javascript
const version = process.env.APP_VERSION || 'blue';

app.get('/', (req, res) => {
  res.json({
    message: 'Hello from Cloud Foundry!',
    version: version,
    hostname: require('os').hostname(),
    timestamp: new Date()
  });
});
```

Update `package.json` build script:

```json
{
  "scripts": {
    "start": "APP_VERSION=green node app.js"
  }
}
```

Deploy green:

```bash
# Deploy new version (green)
$ cf push my-app-green -m 512M -i 2
Pushing app my-app-green...
OK

# Verify it's separate
$ cf apps | grep my-app
my-app-blue   started   2/2   512M
my-app-green  started   2/2   512M

# Test green directly
$ curl https://my-app-green.cf-k8s.io
{"version":"green", ...}

# Test blue still works
$ curl https://my-app.cf-k8s.io
{"version":"blue", ...}
```

## Part 3: Verify Green Version

```bash
# Check logs
$ cf logs my-app-green --recent

# Run tests
$ for i in {1..10}; do
  curl -s https://my-app-green.cf-k8s.io | jq .version
done
# All should return "green"

# Check metrics
$ cf stats my-app-green
```

## Part 4: Switch Traffic Gradually

```bash
# Add route to green
$ cf map-route my-app-green cf-k8s.io --hostname my-app
Mapping route my-app.cf-k8s.io to my-app-green...
OK

# Now main route splits traffic between blue and green
$ for i in {1..20}; do
  curl -s https://my-app.cf-k8s.io | jq .version
  sleep 1
done
# Should see mix of "blue" and "green"

# Remove main route from blue
$ cf unmap-route my-app-blue cf-k8s.io --hostname my-app
Unmapping route from my-app-blue...
OK

# Verify all traffic goes to green
$ for i in {1..10}; do
  curl -s https://my-app.cf-k8s.io | jq .version
done
# All should be "green"
```

## Part 5: Parallel Routes

```bash
# Test that parallel routes still work
$ curl -s https://my-app-blue.cf-k8s.io | jq .version
"blue"

$ curl -s https://my-app-green.cf-k8s.io | jq .version  
"green"

# Main route only goes to green now
$ curl -s https://my-app.cf-k8s.io | jq .version
"green"
```

## Part 6: Rollback (If Needed)

```bash
# If green has problems, route back to blue
$ cf unmap-route my-app-green cf-k8s.io --hostname my-app
$ cf map-route my-app-blue cf-k8s.io --hostname my-app

# Verify
$ for i in {1..5}; do
  curl -s https://my-app.cf-k8s.io | jq .version
done
# Back to "blue"

# Blue is still accessible on its own route
$ curl -s https://my-app-blue.cf-k8s.io | jq .version
"blue"
```

## Part 7: Cleanup Old Version

```bash
# When green is stable, delete blue
$ cf delete my-app-blue -f

# Rename green to standard name (optional)
$ cf rename my-app-green my-app

# Verify
$ cf routes | grep my-app
my-app.cf-k8s.io    my-app

$ cf app my-app
instances: 2/2
```

## Advanced: Canary Release

For gradual rollout, use traffic split:

```bash
# Deploy both versions
$ cf push my-app-blue -i 5     # 5 instances - 83%
$ cf push my-app-green -i 1    # 1 instance - 17%

# Both on main route
$ cf map-route my-app-blue cf-k8s.io --hostname my-app
$ cf map-route my-app-green cf-k8s.io --hostname my-app

# Monitor green version
$ cf stats my-app-green

# Increase green instances
$ cf scale my-app-green -i 2   # Now 33% traffic
$ cf scale my-app-green -i 3   # Now 37% traffic
$ cf scale my-app-green -i 5   # Now 50% traffic

# Once stable, remove blue
$ cf unmap-route my-app-blue cf-k8s.io --hostname my-app
```

## Verification

- ✅ Both blue and green deployed
- ✅ Can test each separately
- ✅ Switch traffic gradually
- ✅ Traffic properly routes
- ✅ Can rollback easily
- ✅ Old version remains available
- ✅ Zero downtime during switch

## Key Learnings

1. **Multi-Route Apps**: Single app can have multiple routes
2. **Gradual Switch**: Route traffic gradually for safety
3. **Parallel Testing**: Test new version while old is live
4. **Quick Rollback**: Easy to revert if issues found
5. **Canary Pattern**: Increase instances for gradual rollout
6. **Infrastructure Agnostic**: PCF handles routing automatically

---

Next: [Lab 5: Custom Buildpack](05-custom-buildpack.md)
