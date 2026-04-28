# Lab 3: Service Binding

## Objectives

- Create a service instance
- Bind service to application
- Use injected credentials
- Update app with service integration

## Prerequisites

- Lab 1 and 2 completed
- PostgreSQL service available

## Duration: ~45 minutes

## Part 1: Create Service Instance

```bash
# View available services
$ cf marketplace
Getting services from marketplace in org my-org / space development as admin...
OK

service     plans        description
postgresql  free,small   PostgreSQL Database Service
redis       free,small   Redis In-Memory Cache
rabbitmq    free,small   RabbitMQ Message Broker

# Create PostgreSQL instance
$ cf create-service postgresql free my-database
Creating service instance my-database in org my-org / space development as admin...
OK

# Verify
$ cf services
name           service     plan   status
my-database    postgresql  free   create succeeded
```

## Part 2: Bind Service to App

```bash
# Bind service
$ cf bind-service my-app my-database
Binding service instance my-database to app my-app...
OK

# Restage app to inject credentials
$ cf restage my-app
Restaging app my-app in org my-org / space development as admin...
[... staging output ...]
```

## Part 3: View Injected Credentials

```bash
# Check service keys
$ cf service-key my-database admin-key
{
  "hostname": "postgres-host",
  "port": 5432,
  "username": "user",
  "password": "password",
  "database": "database_name",
  "uri": "postgresql://user:password@host:5432/database_name"
}

# View in app env
$ cf env my-app
User-Provided:
VCAP_SERVICES:
{
  "postgresql": [
    {
      "binding_name": null,
      "credentials": {
        "uri": "postgresql://..."
      },
      "instance_name": "my-database",
      "label": "postgresql",
      "name": "my-database",
      "plan": "free",
      "provider": null,
      "syslog_drain_url": null,
      "tags": [...],
      "volume_mounts": []
    }
  ]
}
```

## Part 4: Update App to Use Service

Create `db.js`:

```javascript
const { Pool } = require('pg');

// Parse connection URL from environment
const connectionString = process.env.DATABASE_URL || 
  `postgresql://localhost/mydb`;

const pool = new Pool({
  connectionString: connectionString,
  ssl: { rejectUnauthorized: false } // For self-signed certs
});

module.exports = pool;
```

Update `app.js`:

```javascript
const db = require('./db');

// Endpoint that uses database
app.get('/users', async (req, res) => {
  try {
    const result = await db.query(
      'SELECT NOW() as now'
    );
    res.json(result.rows);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Create table on startup
async function initDb() {
  try {
    await db.query(`
      CREATE TABLE IF NOT EXISTS users (
        id SERIAL PRIMARY KEY,
        name VARCHAR(255),
        created_at TIMESTAMP DEFAULT NOW()
      )
    `);
    console.log('Database initialized');
  } catch (err) {
    console.error('DB init error:', err);
  }
}

initDb();
```

Update `package.json`:

```json
{
  "dependencies": {
    "express": "^4.18.2",
    "pg": "^8.8.0"
  }
}
```

## Part 5: Deploy Updated App

```bash
# Install dependencies
$ npm install

# Deploy
$ cf push my-app

# Test database connectivity
$ curl https://my-app.cf-k8s.io/users
[{"now":"2024-04-28T14:30:00Z"}]

# View logs
$ cf logs my-app --recent
[APP...] Database initialized
```

## Part 6: Service Management

```bash
# Add another service  
$ cf create-service redis free my-cache
$ cf bind-service my-app my-cache
$ cf restage my-app

# View all bound services
$ cf services -o my-org
name         service     plan   bound apps
my-database  postgresql  free   my-app
my-cache     redis       free   my-app

# Unbind if needed
$ cf unbind-service my-app my-database

# Delete service (must unbind first)
$ cf delete-service my-database
```

## Verification

- ✅ Service instance created
- ✅ Service bound to app
- ✅ Credentials injected into environment
- ✅ App connects to database
- ✅ Can query database from app
- ✅ Multiple services can be bound

## Key Learnings

1. **Service Broker**: Abstraction for provisioning services
2. **Automatic Binding**: Credentials injected as environment variables
3. **VCAP_SERVICES**: Standard format for service metadata
4. **Connection Pooling**: Apps manage their own connections
5. **Multi-Binding**: Apps can bind to multiple services
6. **Credentials Lifecycle**: Automatic credential generation and rotation

---

Next: [Lab 4: Blue-Green Deployment](04-blue-green-lab.md)
