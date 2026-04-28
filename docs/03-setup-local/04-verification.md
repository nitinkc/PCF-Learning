# Verification Steps

## Complete Verification Checklist

### 1. Kubernetes Running

```bash
$ kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:6443

$ kubectl get nodes
NAME             STATUS   ROLES           AGE    VERSION
docker-desktop   Ready    control-plane   10m    v1.27.0
```

### 2. CF System Pods Running

```bash
$ kubectl get pods -n cf-system

# Should see pods for:
# api-*, scheduler-*, diego-*, gorouter-*, etc
# All should be RUNNING status

# Check pod health
kubectl describe pod -n cf-system api-0
```

### 3. CF CLI Connected

```bash
$ cf login -a https://api.cf-k8s.io --skip-ssl-validation
API endpoint: https://api.cf-k8s.io

$ cf target
api:   https://api.cf-k8s.io
user:  admin
org:   my-org
space: development
```

### 4. Can Create Organization

```bash
$ cf create-org test-org
Creating organization test-org...
OK

$ cf orgs
Getting organizations...
my-org
test-org
```

### 5. Can Create Space

```bash
$ cf create-space test-space -o test-org
Creating space test-space in organization test-org...
OK

$ cf target -o test-org -s test-space
api:   https://api.cf-k8s.io
user:  admin  
org:   test-org
space: test-space
```

### 6. Buildpacks Available

```bash
$ cf buildpacks
Getting buildpacks...
Name                Position   Enabled   Locked
nodejs_buildpack    1          true      false
ruby_buildpack      2          true      false
python_buildpack    3          true      false
java_buildpack      4          true      false
go_buildpack        5          true      false
```

## Troubleshooting

### Can't connect to API

```bash
# Check cf4k8s pods
kubectl get pods -n cf-system | grep api

# If not running, check logs
kubectl logs -n cf-system api-0

# Restart if needed
kubectl delete pod -n cf-system api-0
# (Kubernetes will recreate it)
```

### SSL certificate error

```bash
# For local setup, skip SSL verification
cf login -a https://api.cf-k8s.io --skip-ssl-validation

# Or trust self-signed cert
# macOS: sudo security add-trusted-cert /path/to/cert
```

### No buildpacks listed

```bash
# Buildpacks might not be uploaded yet
# Check pod logs
kubectl logs -n cf-system -l app=cc-api

# May need to upload buildpacks
cf create-buildpack nodejs_buildpack nodejs-buildpack.zip 1
```

## Success Criteria

You're ready to proceed when:

- ✅ Kubernetes cluster is running
- ✅ All CF system pods are RUNNING
- ✅ CF CLI authentication works
- ✅ Can create orgs and spaces
- ✅ Can list buildpacks
- ✅ Can target an org/space

---

Next: [Deploy Your First App](../06-labs/01-deploy-first-app.md)
