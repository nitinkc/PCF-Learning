# Lab 6: Monitoring and Logging

## Objectives

- Stream and analyze application logs
- Monitor application metrics
- Set up structured logging
- Create log aggregation

## Prerequisites

- Labs 1-5 completed
- Application running

## Duration: ~1.5 hours

## Part 1: Real-Time Logs

```bash
# Stream logs in real-time
$ cf logs my-app
[APP/PROC/WEB/0] ERR Error: Cannot find module 'express'
[APP/PROC/WEB/1] OUT > my-app@1.0.0 start
[APP/PROC/WEB/1] OUT > node app.js
[APP/PROC/WEB/1] OUT App listening on port 8080
[CELL/0] Starting app instance...
[APP/PROC/WEB/0] OUT App listening on port 8080

# Logs from all instances aggregated with [instance] prefix

# Exit with Ctrl+C
```

## Part 2: Structured Logging

Update `app.js` with structured logging:

```javascript
const express = require('express');
const app = express();

// Structured logging middleware
app.use((req, res, next) => {
  const startTime = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - startTime;
    console.log(JSON.stringify({
      timestamp: new Date().toISOString(),
      level: 'INFO',
      method: req.method,
      path: req.path,
      status: res.statusCode,
      duration_ms: duration,
      instance: process.env.CF_INSTANCE_INDEX || '0'
    }));
  });
  
  next();
});

app.get('/', (req, res) => {
  console.log(JSON.stringify({
    timestamp: new Date().toISOString(),
    level: 'INFO',
    message: 'Root endpoint called',
    instance: process.env.CF_INSTANCE_INDEX
  }));
  res.json({ message: 'OK' });
});

app.listen(process.env.PORT || 8080);
```

Deploy:

```bash
$ cf push my-app

# View logs
$ cf logs my-app --recent
```

## Part 3: Real-Time Metrics

```bash
# View CPU and memory usage
$ cf stats my-app
state     since                  cpu    memory
running   2024-04-28 14:30:31    12.5%  125M of 512M
running   2024-04-28 14:30:35    8.3%   98M of 512M
running   2024-04-28 14:30:37    15.2%  145M of 512M

# Watch continuously
$ watch cf stats my-app

# Parse with jq (if available)
$ cf stats my-app | grep 'running'
```

## Part 4: Events and Lifecycle

```bash
# View app events
$ cf events my-app
time                        type        actor      description
2024-04-28T14:30:31.000Z    audit.app.create    user@example.com   Initiated creation
2024-04-28T14:30:32.000Z    audit.app.update    user@example.com   state changed to started
2024-04-28T14:30:40.000Z    app.crash       index 0    app instance exited
2024-04-28T14:30:42.000Z    app.update      diego     app instance added

# Real-time events
$ cf events my-app --follow
```

## Part 5: Log Aggregation Script

Create `log-analyzer.sh`:

```bash
#!/bin/bash
# Analyze logs for errors and patterns

echo "Fetching recent logs..."
cf logs my-app --recent | tee current-logs.txt

echo ""
echo "=== ERROR SUMMARY ==="
grep -i "error\|err" current-logs.txt | wc -l

echo ""
echo "=== ERROR DETAILS ==="
grep -i "error\|err" current-logs.txt | head -5

echo ""
echo "=== INSTANCE DISTRIBUTION ==="
grep "\[APP/PROC" current-logs.txt | \
  sed 's/.*\[APP\/PROC\/WEB\/\([0-9]*\).*/\1/' | \
  sort | uniq -c

echo ""
echo "=== SLOW REQUESTS (>1000ms) ==="
grep -o '"duration_ms":[0-9]*' current-logs.txt | \
  grep -o '[0-9]*' | \
  awk '$1 > 1000 { print $1 "ms" }'
```

Run:

```bash
$ chmod +x log-analyzer.sh
$ ./log-analyzer.sh
```

## Part 6: Application Metrics Endpoint

Add metrics endpoint to `app.js`:

```javascript
const requestCounts = {};
const startTime = Date.now();

app.use((req, res, next) => {
  requestCounts[req.path] = (requestCounts[req.path] || 0) + 1;
  next();
});

app.get('/metrics', (req, res) => {
  const uptime = Date.now() - startTime;
  res.json({
    uptime_ms: uptime,
    instance_index: process.env.CF_INSTANCE_INDEX,
    memory_usage: {
      rss: process.memoryUsage().rss,
      heapTotal: process.memoryUsage().heapTotal,
      heapUsed: process.memoryUsage().heapUsed
    },
    request_counts: requestCounts,
    timestamp: new Date().toISOString()
  });
});
```

Query metrics:

```bash
$ curl https://my-app.cf-k8s.io/metrics | jq .
{
  "uptime_ms": 45320,
  "instance_index": "0",
  "memory_usage": {
    "rss": 52429824,
    "heapTotal": 36864000,
    "heapUsed": 18432000
  },
  "request_counts": {
    "/": 145,
    "/health": 89,
    "/metrics": 32
  }
}
```

## Part 7: Log Format Standards

Common logging formats to use:

**JSON (Recommended)**:
```
{"timestamp":"2024-04-28T14:30:00Z","level":"INFO","message":"..."}
```

**Text (Simple)**:
```
2024-04-28 14:30:00 INFO: Request to /api
```

**Structured (Bunyan)**:
```
{"name":"my-app","hostname":"...","pid":123,"level":30,"msg":"..."}
```

## Verification

- ✅ Real-time logs accessible
- ✅ Structured logging configured
- ✅ Can view metrics
- ✅ Events recorded
- ✅ App endpoint exposed metrics
- ✅ Can parse and analyze logs

## Key Learnings

1. **Log Aggregation**: All instance logs in one stream
2. **Structured Logging**: JSON format for easy parsing
3. **Metrics**: Application and system metrics available
4. **Events**: Full lifecycle tracked
5. **Analysis**: Can summarize and diagnose issues
6. **Monitoring**: Foundation for alerts and dashboards

## Next Steps

- Export logs to external systems (Splunk, ELK, etc.)
- Set up alerts on error rates
- Create dashboards from metrics
- Integrate with APM tools

---

**Congratulations! You've completed all 6 labs.**

You now understand:
- ✅ Cloud Foundry deployment
- ✅ Scaling and availability
- ✅ Service integration
- ✅ Blue-green deployments
- ✅ Custom buildpacks
- ✅ Monitoring and logging
