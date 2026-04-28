# Local cf4k8s Setup

## Prerequisites

Before installing cf4k8s, ensure you have:

### System Requirements
- **OS**: macOS, Linux, or Windows (WSL2)
- **Docker Desktop v4.0+**: [Download](https://www.docker.com/products/docker-desktop)
- **kubectl v1.24+**: Install via `brew install kubectl` or [Docker Desktop](https://www.docker.com/products/docker-desktop)
-  **4GB+ RAM minimum** (8GB recommended)
- **20GB disk space** for cf4k8s and workloads

### Command Line Tools
- **Homebrew** (macOS): [https://brew.sh](https://brew.sh)
- **bosh CLI**: `brew install cloudfoundry/tap/bosh-cli`
- **CF CLI**: `brew tap cloudfoundry/tap && brew install cf-cli`

## Verification

```bash
# Check Docker
$ docker --version
Docker version 24.0.0

# Check kubectl
$ kubectl version --client
Client Version: v1.27.0

# Check available resources
$ docker stats --no-stream
# Should show sufficient memory available
```

---

Next: [Installing cf4k8s](02-install-cf4k8s.md)
