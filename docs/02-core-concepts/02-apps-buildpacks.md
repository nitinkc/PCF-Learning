# Applications and Buildpacks

## Applications: The Core Unit

In PCF, your application is the fundamental deployable unit.

### What Is an Application?

```bash
# Your app consists of:
$ ls -la my-app/
  app.js              # Application code
  package.json        # Dependencies
  Procfile            # Process types (optional)
  .cfignore           # Files to exclude (like .gitignore)
```

Push this to PCF:

```bash
$ cf push my-app
```

PCF automatically:
1. Detects language (JavaScript via package.json)
2. Selects Node.js buildpack
3. Installs runtime and dependencies
4. Starts application
5. Registers route
6. Starts health checks

### Application Lifecycle

```mermaid
graph TD
    A["Developer pushes code<br/>cf push my-app"] -->|Uploads app bits| B["Cloud Controller<br/>Stores source code"]
    B -->|Build phase| C["Buildpack<br/>Detect + Compile"]
    C -->|Container image| D["Store droplet<br/>Ready to run"]
    D -->|Staging| E["Create container<br/>with runtime"]
    E -->|Running| F["Diego starts instance<br/>App listening"]
    F -->|Register| G["Gorouter adds route<br/>Traffic directed"]
    G -->|User requests| H["App responds<br/>Running!"]
    H -->|cf stop| I["Running state<br/>Changed"]
```

## Buildpacks: The Magic

### What Buildpacks Do

Buildpacks transform source code into running applications.

**Without PCF (IaaS or raw Kubernetes):**
```
1. Write Dockerfile
2. Build image (20min for dependencies)
3. Push to registry
4. Deploy manifest
5. Debug if something broke
6. Repeat for every app
```

**With PCF:**
```
1. $ cf push
2. Done (buildpack handles Dockerfile equivalent)
```

### Buildpack Process

```mermaid
graph LR
    A["Source Code<br/>app.js, package.json"] 
    A -->|Detect| B["Identify language<br/>Node.js 18"]
    B -->|Build| C["Install runtime<br/>npm install<br/>Compile assets"]
    C -->|Release| D["Create droplet<br/>Ready to run<br/>8MB compressed"]
    D -->|Run| E["Start container<br/>with droplet<br/>app listening:8080"]
```

### Buildpack Detection

PCF includes standard buildpacks:

| Language | Detection | Buildpack |
|----------|-----------|-----------|
| Node.js | `package.json` | Node.js buildpack |
| Ruby | `Gemfile` | Ruby buildpack |
| Python | `requirements.txt` | Python buildpack |
| Java | `pom.xml`, `.jar` | Java buildpack |
| Go | `go.mod`, `.go` files | Go buildpack |

**Custom buildpacks** for other languages/frameworks.

### Buildpack Example: Ruby

**Your source code:**
```
my-app/
  app.rb
  Gemfile
  config.ru
```

**Buildpack does:**
1. **Detect**: Found `Gemfile` → Ruby app
2. **Compile**:
   - Download Ruby 3.1.0
   - Run `bundle install` → Install gems
   - Precompile assets (if Rails)
   - Create optimized layer
3. **Release**:
   - Create droplet (compressed app + runtime)
   - Document start command
   - Set environment

**Result:** Ready-to-run droplet (~5-50MB)

## Application Configuration

### Environment Variables

Variables injected into every application instance:

```bash
# PCF provides system variables:
PORT=8080                    # Listen port
CF_INSTANCE_ADDR=10.0.0.1   # Container IP
CF_INSTANCE_INDEX=0         # Instance number (0, 1, 2, etc.)
CF_APP_NAME=my-app          # Application name

# Service bindings inject credentials:
DATABASE_URL=postgres://user:pass@host:5432/db

# Custom user variables:
cf set-env my-app NODE_ENV production
```

### Procfile (Process Types)

Define how to start your application:

```
$ cat Procfile
web: node app.js
worker: node worker.js
scheduler: node scheduler.js
```

Deploy specific process:
```bash
cf push my-app --process-type web
```

## Instances: Scaling and Availability

### Instances vs Containers

- **Application**: Your code (Ruby, Python, Node, etc.)
- **Instance**: Running copy of app in a container
- Scale horizontally by increasing instances

```bash
# Start with 1 instance
$ cf push my-app -i 1
# App has 1 instance running

# Scale to 3 instances
$ cf scale my-app -i 3
# Diego starts 2 more instances
# Gorouter load balances across 3
```

### Instance Health

Each instance:
- Sends heartbeat every 10 seconds
- Diego checks /health endpoint (if configured)
- Failed instance automatically restarted
- Health check failures trigger replacement

```bash
# Specify health check:
$ cf push my-app --health-check-type http --endpoint /health
```

### Instance Indexing

Each instance gets unique:
- **Index** (0, 1, 2, ...): Order in space
- **GUID** (unique ID)
- **Port**: Usually `$PORT` (8080 default)
- **IP address**: Within container network

Apps can use index for ordering:
```javascript
// Node.js example
const instanceIndex = process.env.CF_INSTANCE_INDEX;
console.log(`Hello from instance ${instanceIndex}`);
```

## Application States

```mermaid
graph TD
    A["Created<br/>cf create-app"] -->|cf start| B["Starting<br/>Buildpack running"]
    B -->|Compilation complete| C["Running<br/>Instance healthy"]
    C -->|cf stop| D["Stopped<br/>Instance stopped"]
    D -->|cf start| B
    C -->|cf restart| E["Restarting<br/>Stop + Start"]
    E -->|Staging complete| C
    C -->|cf delete| F["Deleted<br/>Removed from system"]
```

## Deploying: cf push Explained

```bash
$ cf push my-app -c "node app.js" --no-start

# What happens:

1. Compress app files (respects .cfignore)
2. Upload to Cloud Controller
3. Cloud Controller stores bits
4. Diego orchestrates build
5. Buildpack detects language
6. Buildpack downloads runtime
7. Source code compiled
8. Droplet created (compressed app + runtime)
9. Droplet stored in blob store
10. Droplet staged (extract to container FS)
11. Application started
12. Instance health checked
13. Route registered if needed
14. Traffic directed via Gorouter
```

**Typical timing:**
- Simple app: 30-60 seconds
- App with many dependencies: 2-5 minutes

## Memory and Disk

### Memory
- Specified per application
- Each instance gets allocated memory
- Default: 1GB per instance

```bash
cf push my-app -m 512M    # 512MB per instance
cf scale my-app -i 4      # 4 instances, each 512MB
# Total: 2GB allocated
```

### Disk
- Ephemeral disk within container
- Lost on app restart/redeploy
- Default: 1GB per instance

For permanent storage → Use service binding (database, object storage)

---

Next: [Services & Service Brokers](03-services.md)
