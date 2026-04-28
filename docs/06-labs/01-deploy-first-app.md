# Lab 1: Deploy Your First Application

## Objectives

- Deploy a simple application to cf4k8s
- View application logs and metrics
- Understand the deployment lifecycle
- Scale instances
- View application status

## Prerequisites

- ✅ cf4k8s installed and running
- ✅ CF CLI configured and authenticated
- ✅ Org and space targeted
- ✅ Git installed

## Duration: ~30 minutes

## Step 1: Clone Sample Application

```bash
# Clone sample Node.js app
git clone https://github.com/cloudfoundry-samples/nodejs-cf-sample-app.git
cd nodejs-cf-sample-app

# Or create simple app locally:
mkdir my-first-app
cd my-first-app
```

## Step 2: Create Simple Node.js App

Create `app.js`:

```javascript
const express = require('express');
const app = express();
const port = process.env.PORT || 8080;

app.get('/', (req, res) => {
  res.json({
    message: 'Hello from Cloud Foundry!',
    hostname: require('os').hostname(),
    timestamp: new Date()
  });
});

app.get('/health', (req, res) => {
  res.status(200).json({ status: 'healthy' });
});

app.listen(port, '0.0.0.0', () => {
  console.log(`App listening on port ${port}`);
});
```

Create `package.json`:

```json
{
  "name": "my-first-app",
  "version": "1.0.0",
  "description": "First CF app",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

## Step 3: Deploy Application

```bash
# From app directory
cf push my-app

# Output:
# Pushing app my-app to organization my-org / space development as admin...
# Packaging files to upload
# Uploading files
#  11.46 MB
# Processing uploaded files
# Staging app 'my-app'
# Buildpack: nodejs
# Downloaded nodejs buildpack v1.8.0
# Staging...
# [... build output ...]
# Uploading complete
# Waiting for app to start...
# #0   running   2024-04-28 14:30:31 UTC
```

## Step 4: Verify Deployment

```bash
# Check app status
$ cf app my-app
name:         my-app
requested state:   started
instances:    1/1
usage:        512M x 1 instances
urls:         my-app.cf-k8s.io
last uploaded:    2024-04-28 14:30:29 UTC

instance   state      since                  cpu    memory       disk
#0         running    2024-04-28 14:30:31    0.0%   85.3M of 512M   156M of 1G
```

## Step 5: View Logs

```bash
# Tail logs in real-time
$ cf logs my-app
[APP/PROC/WEB/0] OUT > my-first-app@1.0.0 start /home/vcap/app
[APP/PROC/WEB/0] OUT > node app.js
[APP/PROC/WEB/0] OUT App listening on port 8080

# View recent logs
$ cf logs my-app --recent

# Exit with Ctrl+C
```

## Step 6: Test Application

```bash
# Get app URL
$ cf app my-app | grep urls
urls:         my-app.cf-k8s.io

# Test endpoint
$ curl https://my-app.cf-k8s.io
{"message":"Hello from Cloud Foundry!","hostname":"...","timestamp":"2024-04-28T..."}

# Test health endpoint
$ curl https://my-app.cf-k8s.io/health
{"status":"healthy"}

# (May need --insecure flag for self-signed cert)
# curl -k https://my-app.cf-k8s.io
```

## Step 7: Scale Application

```bash
# Scale to 3 instances
$ cf scale my-app -i 3
Scaling app my-app
OK

# Verify
$ cf app my-app
instances: 3/3

#0   running   2024-04-28 14:30:31
#1   running   2024-04-28 14:30:45
#2   running   2024-04-28 14:30:47

# View load balancing
$ for i in {1..6}; do curl -s https://my-app.cf-k8s.io | jq .hostname; done
# Should see different hostnames as load balancer distributes
```

## Step 8: Change Memory

```bash
# Allocate more memory
$ cf scale my-app -m 1G
Scaling app my-app
OK

# Verify
$ cf app my-app
usage: 1G x 3 instances
```

## Verification Checklist

- ✅ App deployed successfully
- ✅ App is in RUNNING state
- ✅ Can access app via URL
- ✅ Logs are visible
- ✅ Health endpoint returns 200
- ✅ Can scale instances
- ✅ Load balancing works across instances

## Cleanup (Optional)

```bash
# Delete app (keeps data)
cf stop my-app

# Delete app completely  
cf delete my-app

# Reclaim space
cf delete-orphaned-routes
```

## Key Learnings

1. **Simple Deployment**: `cf push` handles everything (build, containerization, deployment)
2. **Automatic Buildpack**: Detected Node.js from package.json
3. **Instance Health**: Automatically monitors and restarts
4. **Load Balancing**: Multiple instances automatically load balanced
5. **Scaling**: Easy horizontal scaling with one command
6. **Logging**: Real-time access to app logs

## Next Steps

- [Lab 2: Scaling & Availability](02-scaling-availability.md)
- [Lab 3: Service Binding](03-service-binding.md)
