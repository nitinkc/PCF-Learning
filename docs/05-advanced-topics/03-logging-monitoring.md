# Logging and Monitoring

## Application Logs

### Accessing Logs

```bash
# Real-time streaming
$ cf logs my-app

# Recent logs
$ cf logs my-app --recent

# Follow with grep
$ cf logs my-app | grep -i error

# Save to file
$ cf logs my-app --recent > app-logs.txt
```

### Log Format

Logs aggregated from all instances:

```
[TIMESTAMP] [INSTANCE] [TYPE] MESSAGE

[2024-04-28T14:30:00.000Z] [APP/PROC/WEB/0] [OUT] App started
[2024-04-28T14:30:01.000Z] [APP/PROC/WEB/1] [OUT] Listening on port 8080
[2024-04-28T14:30:02.000Z] [CELL/0] [ERR] Instance crashed
```

## Structured Logging

Best practice: output JSON

```javascript
// Structured logging
console.log(JSON.stringify({
  timestamp: new Date().toISOString(),
  level: 'INFO',
  message: 'User logged in',
  user_id: userId,
  ip: req.ip,
  duration_ms: responseTime
}));
```

Parse and analyze:

```bash
# Extract duration values
$ cf logs my-app | grep duration | \
  sed 's/.*"duration_ms":\([0-9]*\).*/\1/' | \
  awk '{sum+=$1; count++} END {print sum/count " ms avg"}'

# Count error levels
$ cf logs my-app | grep '"level":"ERROR"' | wc -l
```

## Application Metrics

### Statistics

```bash
# CPU and memory
$ cf stats my-app
INSTANCE   STATE    CPU      MEMORY  DISK
#0         running  15.2%    234M    156M
#1         running  12.8%    198M    156M

# Watch continuously
$ watch -n 2 cf stats my-app
```

### Custom Metrics Endpoint

Expose metrics from application:

```javascript
app.get('/metrics', (req, res) => {
  res.json({
    uptime_seconds: (Date.now() - startTime) / 1000,
    requests_total: requestCount,
    errors_total: errorCount,
    memory_mb: process.memoryUsage().heapUsed / 1024 / 1024,
    response_time_ms: avgResponseTime
  });
});
```

Query metrics:

```bash
$ curl https://my-app.cf-k8s.io/metrics | jq .
{
  "uptime_seconds": 3600,
  "requests_total": 15234,
  "errors_total": 12,
  "memory_mb": 125.5,
  "response_time_ms": 45.2
}
```

## Integration with External Systems

### Log Forwarding

Forward logs to external system:

```bash
# SSH into app
$ cf ssh my-app

# Configure rsyslog
$ echo "*.* @@syslog-server:514" | sudo tee -a /etc/rsyslog.conf

# Or via app code
const winston = require('winston');
const transport = new winston.transports.Syslog({
  host: 'syslog-server',
  port: 514
});
```

### DataDog Integration

```bash
# Set DataDog API key
$ cf set-env my-app DD_API_KEY "your-api-key"
$ cf set-env my-app DD_TRACE_ENABLED "true"

# Instrument app
const tracer = require('dd-trace').init();

app.use(tracer.middleware.express());
```

### Prometheus Metrics

Expose Prometheus format:

```javascript
app.get('/prometheus', (req, res) => {
  res.set('Content-Type', 'text/plain');
  res.send(`
# HELP requests_total Total HTTP requests
# TYPE requests_total counter
requests_total{method="GET"} 1234

# HELP response_time_ms Response time in milliseconds
# TYPE response_time_ms histogram
response_time_ms_bucket{le="10"} 100
response_time_ms_bucket{le="100"} 500
  `);
});
```

Scrape with Prometheus:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'my-app'
    static_configs:
      - targets: ['my-app.cf-k8s.io:443']
    metrics_path: '/prometheus'
```

## Monitoring Best Practices

1. **Health Endpoints**: Implement /health
2. **Metrics Exposed**: Publish internal metrics
3. **Structured Logging**: JSON format for parsing
4. **Alerting**: Set up alerts on key metrics
5. **Tracing**: Trace requests across services
6. **Aggregation**: Centralize logs from all instances

---

See [Lab 6: Monitoring & Logging](../06-labs/06-monitoring-lab.md) for hands-on practice.
