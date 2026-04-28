# CF CLI Configuration

## Installing CF CLI

```bash
# macOS
brew tap cloudfoundry/tap
brew install cf-cli@8

# Linux
wget -q -O - https://packages.cloudfoundry.org/debian/cli.cloudfoundry.org.key | sudo apt-key add -
sudo bash -c 'echo "deb https://packages.cloudfoundry.org/debian stable main" > /etc/apt/sources.list.d/cloudfoundry-cli.list'
sudo apt update
sudo apt install cf-cli

# Windows (PowerShell)
choco install cloudfoundry-cli

# Verify
cf version
```

## Logging In

### Local cf4k8s

For local Docker Desktop setup:

```bash
# Get API endpoint
kubectl get ingress -n cf-system

# Typical local endpoint:
# https://api.cf-k8s.io

# Login
cf login -a https://api.cf-k8s.io --skip-ssl-validation

# Username: admin
# Password: (provided by cf4k8s installation)
```

### Creating Organization and Space

```bash
# Create organization
cf create-org my-org

# Create space
cf create-space development -o my-org

# Target org/space
cf target -o my-org -s development
```

## Verifying CLI

```bash
# Check connection
$ cf orgs
Getting organizations as admin on https://api.cf-k8s.io...
my-org

# Check spaces
$ cf spaces
Getting spaces in organization my-org as admin on https://api.cf-k8s.io...
development

# Check buildpacks
$ cf buildpacks
Getting buildpacks...
Name           Position   Enabled   Locked   Filename
nodejs_buildpack   1         false     false    nodejs-buildpack-v1.8.0.zip
ruby_buildpack     2         false     false    ruby-buildpack-v1.9.0.zip
python_buildpack   3         false     false    python-buildpack-v1.8.0.zip
```

## Common CLI Configuration

### Setting Persistent Properties

```bash
# Always target same org/space
cf target -o my-org -s development

# Save config
# Config stored at: ~/.cf/config.json (don't edit manually)

# Verify current target
cf target
api:   https://api.cf-k8s.io
user:  admin
org:   my-org
space: development
```

### Creating Service Accounts

```bash
# Create user for deployments
cf create-user deploy-user password123

# Grant developer role
cf set-space-role deploy-user my-org development SpaceDeveloper
```

---

Next: [Verification Steps](04-verification.md)
