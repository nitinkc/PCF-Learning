# Security and RBAC

## Role-Based Access Control

### Roles

**Organization Roles:**

```
OrgManager        - Full org control
OrgBillingManager - Billing only
OrgAuditor        - Read-only
```

**Space Roles:**

```
SpaceManager      - Manage space
SpaceDeveloper    - Deploy, manage apps
SpaceAuditor      - Read-only
```

### Managing Users

```bash
# Create user
$ cf create-user username password@123

# Assign org role
$ cf set-org-role username my-org OrgManager

# Assign space role
$ cf set-space-role username my-org production SpaceDeveloper

# View roles
$ cf org-users my-org

# Remove role
$ cf unset-org-role username my-org OrgManager
```

## Securing Applications

### Environment Variable Secrets

Avoid credentials in code:

```bash
# Set sensitive variables
$ cf set-env my-app DB_PASSWORD "super-secret"
$ cf set-env my-app API_KEY "key-123"

# Access in app
const dbPassword = process.env.DB_PASSWORD;
const apiKey = process.env.API_KEY;
```

### Service Credentials

Use service binding instead:

```bash
# Automatic credential injection
$ cf bind-service my-app my-database

# Credentials in VCAP_SERVICES
const services = JSON.parse(process.env.VCAP_SERVICES);
const credentials = services.postgresql[0].credentials;
```

## Network Security

### Network Policies

Control inter-app communication:

```bash
# Allow my-api to call my-database
$ cf add-network-policy my-api my-database \
  --protocol tcp --port 5432

# Deny all others (implicit deny)

# View
$ cf network-policies
source   destination   protocol   port
my-api   my-database   tcp        5432
```

### HTTPS/TLS

PCF terminates TLS at gorouter:

```
Internet (HTTPS)
    ↓
Gorouter (TLS Termination)
    ↓
App (HTTP on internal network)
```

All external traffic is HTTPS, internal is HTTP.

## Secrets Management

### Using CredHub

Secure credential storage:

```bash
# Store secret
$ cf credhub set -n /my-org/my-space/db-password \
  -t password -w "super-secret-123"

# Reference in manifest
applications:
  - name: my-app
    env:
      DB_PASSWORD: ((db-password))

# Substitute during deploy
$ cf push
```

### Alternative: External Secret Store

Point to external secrets system:

```bash
# Deploy with reference to HashiCorp Vault
$ cf set-env my-app VAULT_TOKEN "token"
$ cf set-env my-app VAULT_ADDR "https://vault.example.com"

# App fetches secrets at startup
const vault = require('node-vault');
const client = vault({ endpoint: process.env.VAULT_ADDR });
const secret = await client.read('secret/database');
```

## Compliance

### Audit Logging

```bash
# View all org events
$ cf events -o my-org

# Shows all deployments, role changes, etc.
time                    type              actor            description
2024-04-28T14:30:00Z    audit.user.create admin            user created
2024-04-28T14:30:05Z    audit.app.create  developer@...    app created
```

### SSL Certificates

```bash
# Upload custom certificate
$ cf create-shared-domain example.com \
  --cert-file cert.pem \
  --key-file key.pem

# PCF uses cert for HTTPS
```

## Cloud Foundry UAA

Unified interface for authentication:

```bash
# Login with different providers
$ cf login (default local)
$ cf login --sso (SSO provider)
$ cf login --origin ldap (LDAP)

# OpenID Connect, SAML, OAuth2 supported
```

## Security Best Practices

1. **Never commit secrets** - Use environment variables
2. **Rotate credentials** - Regularly update passwords
3. **Network policies** - Restrict app-to-app communication
4. **Service binding** - Use instead of hardcoded credentials
5. **Audit logs** - Monitor access and changes
6. **RBAC** - Principle of least privilege
7. **TLS** - Always use HTTPS externally

---

For hands-on security: [Lab 3: Service Binding](../06-labs/03-service-binding.md).
