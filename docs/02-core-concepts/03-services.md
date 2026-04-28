# Services and Service Brokers

## The Service Model

PCF provides a standardized way to integrate any backing service.

### What Are Services?

Services are **managed stateful systems** that applications depend on:

- Databases (PostgreSQL, MySQL, MongoDB)
- Caches (Redis, Memcached)
- Message queues (RabbitMQ, Kafka)
- Search engines (Elasticsearch)
- External SaaS (SendGrid, DataDog, etc.)

### Service Model in PCF

```
┌──────────────────────────────┐
│ PCF Foundation               │
├──────────────────────────────┤
│ Org: My Company              │
│ ├─ Space: Production         │
│ │  ├─ App: my-api            │
│ │  ├─ App: my-worker         │
│ │  ├─ Service: my-db         │
│ │  └─ Service: my-cache      │
```

## Service Brokers: The Bridge

**Service Broker**: API that implements the Open Service Broker specification.

Brokers translate PCF requests to service-specific operations:

```
cf create-service ↓
    ↓
Cloud Controller ↓
    ↓
Service Broker API ↓
    ↓
Terraform/API calls ↓
    ↓
Provisions PostgreSQL instance
```

### Broker Responsibility

1. **Provision**: Create instance (database, cache, etc.)
2. **Bind**: Create credentials and connect to app
3. **Unbind**: Remove app access
4. **Deprovision**: Delete instance

## Complete Service Workflow

### Step 1: Create a Service Instance

```bash
$ cf marketplace
Getting services from marketplace in org MyOrg / space Production as admin...
OK

service          plans                                      description
postgresql       free, 10gb, 50gb                        PostgreSQL Database Service
redis            free, 10gb                               Redis Cache Service

$ cf create-service postgresql 10gb my-db
Creating service instance my-db in org MyOrg / space Production as admin...
OK
```

**What happens:**
1. Cloud Controller validates request
2. Contacts PostgreSQL service broker
3. Broker creates PostgreSQL instance
4. Stores credentials in vault
5. Returns connection details to broker
6. Service appears in space

### Step 2: Bind Service to Application

```bash
$ cf bind-service my-app my-db
Binding service instance my-db to app my-app...
OK

$ cf restage my-app
Restaging app my-app...
OK
```

**What happens:**
1. Cloud Controller contacts broker
2. Broker generates unique credentials
3. Creates database username/password
4. Cloud Controller stores binding
5. App environment updated
6. App restaged with new environment

### Step 3: Application Uses Service

**In your application code:**

```javascript
// Node.js example
const url = process.env.DATABASE_URL;
// postgresql://user:pass@host:5432/dbname

const pool = new Pool({ connectionString: url });
const result = await pool.query('SELECT * FROM users');
```

**In Python:**
```python
import os
db_url = os.environ.get('DATABASE_URL')
# postgresql://user:pass@host:5432/dbname
```

## Service Binding Details

### Environment Variables Injected

When service is bound, PCF injects `VCAP_SERVICES` JSON:

```json
{
  "postgresql": [
    {
      "name": "my-db",
      "instance_name": "my-db",
      "binding_name": null,
      "label": "postgresql",
      "tags": ["database", "sql"],
      "plan": "10gb",
      "credentials": {
        "uri": "postgresql://user:pass@host:5432/db",
        "hostname": "host",
        "port": 5432,
        "username": "user",
        "password": "pass",
        "database": "db"
      }
    }
  ]
}
```

Libraries parse this automatically or use `DATABASE_URL` format.

## Service Marketplace

Check available services:

```bash
$ cf marketplace
Getting services from marketplace...

service          plans            description
postgresql       free, 10gb       PostgreSQL Database
redis            free, 10gb       Redis In-Memory Cache
rabbitmq         free, 10gb       RabbitMQ Message Broker
s3               access-key       Amazon S3 Storage

$ cf marketplace -s postgresql
Getting service plan information for service postgresql...

name        description            free or paid
free        Shared PostgreSQL      free
10gb        Dedicated 10GB instance paid
50gb        Dedicated 50GB instance paid
```

## Service Lifecycle

```bash
# Create
$ cf create-service postgresql 10gb my-db

# Check status
$ cf services
name          service     plan   status
my-db         postgresql  10gb   create succeeded

# Bind to app
$ cf bind-service my-app my-db

# Update plan
$ cf update-service my-db -p 50gb

# Unbind from app
$ cf unbind-service my-app my-db

# Delete
$ cf delete-service my-db
```

## User-Provided Services

For services without brokers, create manually:

```bash
$ cf create-user-provided-service external-api \
  -p '{"url":"https://api.example.com","key":"secret-key"}'

$ cf bind-service my-app external-api

# In app:
var creds = JSON.parse(process.env.VCAP_SERVICES);
var url = creds['user-provided'][0].credentials.url;
```

## Service Security

### Credential Management

- Credentials generated unique per binding
- Stored in Cloud Controller database
- Encrypted at rest
- Injected into container via environment
- No service sees another app's credentials

### Network Policies

- Services typically run in private network
- Applications access via internal network
- Can be restricted via network policies
- Egress traffic can be controlled

## Advanced Service Topics

### Service Sharing

Share service across orgs/spaces:

```bash
$ cf update-service my-db -k org1
$ cf access-service-instance org2 space2 my-db
```

### Service Keys

Create credentials without binding to app:

```bash
$ cf create-service-key my-db admin-key
$ cf service-key my-db admin-key
{
  "username": "admin",
  "password": "secret",
  "uri": "postgresql://admin:secret@host/db"
}
```

Useful for CI/CD, one-off tasks, admin tools.

### Route Services

Transform HTTP requests before reaching app:

```bash
# SSL certificate termination
# Authentication gateway
# API management

$ cf create-user-provided-service my-auth-gateway \
  -r https://auth-gateway.example.com
$ cf bind-route-service example.com my-auth-gateway
```

---

Next: [Routing & Networking](04-routing.md)
