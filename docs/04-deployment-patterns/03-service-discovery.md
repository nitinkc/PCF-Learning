# Service Discovery

## How Services Find Each Other

In PCF, applications discover and communicate with each other through:

1. **Environment Variables** (injected from service bindings)
2. **Internal Routes** (DNS for app-to-app communication)
3. **Service Brokers** (provisioning and credential injection)

## Service Binding Discovery

When service is bound to application:

```bash
$ cf bind-service my-api my-database
$ cf env my-api
```

Application receives credentials via `VCAP_SERVICES`:

```json
{
  "postgresql": [{
    "credentials": {
      "uri": "postgresql://user:pass@postgres-host:5432/db"
    }
  }]
}
```

Application code:

```javascript
const db_url = process.env.DATABASE_URL;
const pool = new Pool({ connectionString: db_url });
```

## App-to-App Communication

Applications communicate via HTTP using internal routes:

```
my-web-app → my-api-app → my-database (service)
```

Configuration:

```bash
# my-web-app calls my-api-app
# my-api-app.cf-k8s.io (internal route)
```

Environment variable in my-web-app:

```javascript
const apiUrl = process.env.API_URL || 
  'http://my-api-app.cf-k8s.io';
```

## Service-to-Service Communication

For microservices:

```bash
# Deploy services
$ cf push service-1
$ cf push service-2 
$ cf push service-3

# Service discovery automatic via naming
# service-1.cf-k8s.io
# service-2.cf-k8s.io
# service-3.cf-k8s.io
```

Call between services:

```javascript
// service-1 calling service-2
fetch('http://service-2.cf-k8s.io/api')
  .then(r => r.json())
  .then(data => res.json(data))
```

## Network Policies

Restrict service-to-service communication:

```bash
# Allow service-1 to call service-2 on port 8080
$ cf add-network-policy service-1 service-2 \
  --protocol tcp --port 8080

# View policies
$ cf network-policies

# Remove if needed
$ cf remove-network-policy service-1 service-2 \
  --protocol tcp --port 8080
```

## Dynamic Service Discovery Pattern

For applications that need to find services dynamically:

```javascript
// Query VCAP_SERVICES at runtime
const services = JSON.parse(process.env.VCAP_SERVICES || '{}');

function getServiceUrl(serviceType) {
  if (services[serviceType]) {
    return services[serviceType][0].credentials.uri;
  }
  throw new Error(`Service ${serviceType} not found`);
}

const dbUrl = getServiceUrl('postgresql');
const cacheUrl = getServiceUrl('redis');
```

## Health Checks in Service Mesh

Each service exposes health endpoint:

```javascript
app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});
```

Consumer services check health before calling:

```javascript
async function checkService(url) {
  try {
    const response = await fetch(`${url}/health`);
    return response.ok;
  } catch {
    return false;
  }
}

// Use circuit breaker pattern
if (await checkService(serviceUrl)) {
  // Call service
} else {
  // Return cached response or error
}
```

## Best Practices

1. **Use Environment Variables**: Injected by CF
2. **Health Checks**: Each service exposes /health
3. **Retry Logic**: Handle transient failures
4. **Circuit Breakers**: Prevent cascading failures
5. **Logging**: Request tracing across services
6. **Timeouts**: Set reasonable timeout values

---

See Labs 1-3 for hands-on examples of service binding and communication.
