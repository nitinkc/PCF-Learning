# Installing cf4k8s

## Overview

cf4k8s is Cloud Foundry running on Kubernetes. This guide sets it up locally using Kind (Kubernetes in Docker).

## Quick Installation (Docker Desktop)

The easiest way is using Docker Desktop's built-in Kubernetes:

### Step 1: Enable Kubernetes in Docker Desktop

1. Open Docker Desktop → Preferences
2. Go to **Kubernetes** tab
3. Check "Enable Kubernetes"
4. Click **Apply & Restart**
5. Wait for Kubernetes to start (5-10 minutes)

Verify:
```bash
$ kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:6443

$ kubectl get nodes
NAME             STATUS   ROLES           AGE    VERSION
docker-desktop   Ready    control-plane   2m     v1.27.0
```

### Step 2: Install cf4k8s Components

```bash
# Create namespace for Cloud Foundry
kubectl create namespace cf-system

# Download cf4k8s (or clone from GitHub)
git clone https://github.com/cloudfoundry/cf-for-k8s.git
cd cf-for-k8s

# Install dependencies (requires bosh)
./bin/install-tools.sh

# Install cf4k8s
./bin/install-cf-for-k8s.sh
```

### Step 3: Verify Installation

```bash
# Check all pods are running
kubectl get pods -n cf-system

# Should see pods like:
# api-0, scheduler-*,diego-*,gorouter-*,etc
```

## Alternative: Using Kind (Kubernetes in Docker)

For more control, use Kind:

```bash
# Install Kind
brew install kind

# Create Kind cluster
kind create cluster --name cf4k8s

# Switch kubectl context
kubectl cluster-info --context kind-cf4k8s
```

## Troubleshooting Installation

### Issue: Kubernetes won't start in Docker Desktop

**Solution:**
```bash
# Reset Kubernetes
# In Docker Desktop → Preferences → Kubernetes → Reset

# Or from terminal:
docker run --rm -it --privileged \
  docker:stable \
  sh -c 'rm -rf /var/lib/docker/*'
```

### Issue: Insufficient memory

```bash
# Increase Docker memory allocation
# Docker Desktop → Preferences → Resources
# Set to at least 8GB
```

### Issue: Pod not starting

```bash
# Check pod status
kubectl describe pod -n cf-system POD_NAME

# Check logs
kubectl logs -n cf-system POD_NAME

# Check events
kubectl get events -n cf-system
```

---

Next: [CLI Configuration](03-cli-setup.md)
