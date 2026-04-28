# Service Brokers

## What is a Service Broker?

Service broker is API server that implements Open Service Broker specification. It translates CF requests into service-specific operations.

```
cf create-service
    ↓
Cloud Controller
    ↓
Service Broker API (HTTP)
    ↓
Service Infrastructure
    ↓
Instance created, credentials returned
```

## Service Broker Operations

### Provision (Create Instance)

```
POST /v2/service_instances/{instance_id}
Body: {
  "service_id": "...",
  "plan_id": "...",
  "organization_guid": "...",
  "space_guid": "..."
}
Response: 201
{
  "dashboard_url": "...",
  "operation": "provision_in_progress"
}
```

### Bind (Connect App to Instance)

```
PUT /v2/service_instances/{instance_id}/service_bindings/{binding_id}
Body: {
  "service_id": "...",
  "plan_id": "...",
  "app_guid": "..."
}
Response: 201
{
  "credentials": {
    "uri": "postgresql://user:pass@host/dbname",
    "username": "user",
    "password": "pass",
    "hostname": "host",
    "port": 5432,
    "database": "dbname"
  }
}
```

### Unbind (Disconnect App)

```
DELETE /v2/service_instances/{instance_id}/service_bindings/{binding_id}
Response: 200
```

### Deprovision (Delete Instance)

```
DELETE /v2/service_instances/{instance_id}
Response: 200
```

## Building a Simple Broker

Node.js example:

```javascript
const express = require('express');
const app = express();

const instances = {};  // In-memory storage

app.use(express.json());

// Brokers typically require basic auth
const auth = (req, res, next) => {
  const [user, pass] = Buffer.from(
    req.headers.authorization.slice(6)
  ).toString().split(':');
  
  if (user === 'admin' && pass === 'secret') {
    next();
  } else {
    res.status(401).json({ error: 'Unauthorized' });
  }
};

// Provision
app.put('/v2/service_instances/:id', auth, (req, res) => {
  const { id } = req.params;
  
  // Create actual service
  const credentials = {
    uri: `postgresql://user:pass@localhost:5432/${id}`,
    username: 'user',
    password: 'pass',
    database: id
  };
  
  instances[id] = credentials;
  
  res.status(201).json({
    dashboard_url: `https://broker.example.com/instances/${id}`,
    operation: 'provision'
  });
});

// Bind
app.put('/v2/service_instances/:instanceId/service_bindings/:bindingId',
  auth, (req, res) => {
  const { instanceId, bindingId } = req.params;
  
  if (!instances[instanceId]) {
    return res.status(404).json({ error: 'Instance not found' });
  }
  
  res.status(201).json({
    credentials: instances[instanceId]
  });
});

// Unbind
app.delete('/v2/service_instances/:instanceId/service_bindings/:bindingId',
  auth, (req, res) => {
  res.status(200).json({});
});

// Deprovision
app.delete('/v2/service_instances/:id', auth, (req, res) => {
  delete instances[req.params.id];
  res.status(200).json({});
});

app.listen(3000);
```

## Registering Service Broker

```bash
# Register broker with CF
cf create-service-broker my-broker \
  admin secret \
  https://broker.example.com

# Enable public access
cf enable-service-access my-service -b my-broker

# Check marketplace
cf marketplace
service           plans   description
my-service        free    My Custom Service
```

## Marketplace Metadata

Service broker advertises capabilities:

```
GET /v2/catalog
Response:
{
  "services": [{
    "id": "service-1",
    "name": "my-database",
    "description": "My Custom Database",
    "plans": [{
      "id": "plan-1",
      "name": "free",
      "description": "Free plan"
    }, {
      "id": "plan-2",
      "name": "paid",
      "description": "Paid plan"
    }]
  }]
}
```

## Built-In Brokers

PCF typically includes brokers for:

-  PostgreSQL
- MySQL
- Redis
- RabbitMQ
- MongoDB (with extensions)

## Best Practices

1. **Idempotence**: Same request twice should be safe
2. **Cleanup**: Delete all resources on deprovision
3. **Credentials**: Generate unique credentials per binding
4. **Documentation**: Document broker behavior
5. **Logging**: Log all requests
6. **Security**: Require authentication, use HTTPS
7. **Error Handling**: Return standard HTTP codes

## Advanced: Async Operations

For long-running operations:

```javascript
// Return 202 Accepted with operation ID
res.status(202).json({
  operation: "provisioning-database",
  state: "in progress"
});

// Cloud Controller polls for completion
GET /v2/service_instances/{id}/last_operation?operation=...
Response: {
  "state": "in progress",  // or "succeeded" or "failed"
  "description": "Creating database..."
}
```

---

For hands-on service usage: [Lab 3: Service Binding](../06-labs/03-service-binding.md).
