# Lab 5: Custom Buildpack

## Objectives

- Understand buildpack structure
- Create custom buildpack
- Deploy app with custom buildpack
- Debug buildpack behavior

## Prerequisites

- Labs 1-3 completed
- Understanding of buildpacks

## Duration: ~1.5 hours

## Part 1: Understanding Buildpacks

Standard buildpack structure:

```
my-buildpack/
├── bin/
│   ├── compile   # Build phase
│   ├── detect    # Detect language
│   └── release   # Runtime configuration
├── lib/          # Helper scripts
└── README.md
```

## Part 2: Create Custom Buildpack

Create directory:

```bash
mkdir -p my-custom-buildpack/bin
cd my-custom-buildpack
```

Create `bin/detect`:

```bash
#!/bin/bash
# Detect if app should use this buildpack
# Exit 0 (success) if yes, 1 (failure) if no

if [[ -f "$1/Procfile" ]]; then
  echo "Custom Buildpack"
  exit 0
fi

exit 1
```

Create `bin/compile`:

```bash
#!/bin/bash
# $1 = BUILD_DIR (app source)
# $2 = CACHE_DIR (persistent)
# $3 = DEPS_DIR (dependencies)

BUILD_DIR=$1
CACHE_DIR=$2
DEPS_DIR=$3

echo "-----> Building with Custom Buildpack"

# Install runtime (example: Python)
echo "-----> Installing Python 3.9"
mkdir -p $DEPS_DIR/python
cd $DEPS_DIR/python

# Download and install (simplified)
wget https://python.org/python-3.9.tar.gz
tar xzf python-3.9.tar.gz

# Set environment
export PATH="$DEPS_DIR/python/bin:$PATH"
export LD_LIBRARY_PATH="$DEPS_DIR/python/lib:$LD_LIBRARY_PATH"

echo "Python installed"

# Install dependencies
if [[ -f "$BUILD_DIR/requirements.txt" ]]; then
  echo "-----> Installing dependencies"
  pip install -r "$BUILD_DIR/requirements.txt"
fi

echo "-----> Build successful"
```

Create `bin/release`:

```bash
#!/bin/bash
# Output runtime configuration
# Sets environment for running app

BUILD_DIR=$1

cat <<EOF
---
default_process_types:
  web: python app.py
EOF
```

Make executable:

```bash
chmod +x bin/*
```

## Part 3: Create Test Application

Create sample app (`app.py`):

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return {'message': 'Hello from custom buildpack!'}

@app.route('/health')
def health():
    return {'status': 'healthy'}

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=int(os.getenv('PORT', 8080)))
```

Create `requirements.txt`:

```
Flask==2.0.1
```

Create `Procfile`:

```
web: python app.py
```

## Part 4: Register Buildpack

```bash
# Create buildpack package
cd my-custom-buildpack
zip -r ../my-custom-buildpack.zip *

# Register with cf4k8s
$ cf create-buildpack my-custom-buildpack \
  ../my-custom-buildpack.zip 5

# Verify
$ cf buildpacks
Name                      Position   Enabled   Locked
nodejs_buildpack          1          true      false
ruby_buildpack            2          true      false
python_buildpack          3          true      false
my-custom-buildpack       5          true      false
```

## Part 5: Deploy with Custom Buildpack

```bash
# Deploy using custom buildpack
$ cf push my-custom-app \
  --buildpack my-custom-buildpack \
  -m 512M

# View build output
$ cf logs my-custom-app --recent
[STG] -----> Building with Custom Buildpack
[STG] -----> Installing Python 3.9
[STG] -----> Installing dependencies
[STG] -----> Build successful
[APP] App started on port 8080
```

## Part 6: Debug Buildpack

```bash
# SSH into app container
$ cf ssh my-custom-app

# Check installed binaries
$ which python
/app/.heroku/python/bin/python

# Check environment
$ env | grep PATH
PATH=/app/.heroku/python/bin:/app/bin:...

# Check buildpack files
$ ls -la /buildpacks/
$ cat /buildpacks/release

# Exit
$ exit
```

## Part 7: Advanced Buildpack Features

Add caching to `bin/compile`:

```bash
# Cache dependencies to speed up rebuilds
if [[ -d "$CACHE_DIR/python" ]]; then
  echo "-----> Using cached Python"
  cp -r "$CACHE_DIR/python" "$DEPS_DIR/"
else
  # ... install steps ...
  cp -r "$DEPS_DIR/python" "$CACHE_DIR/"
fi
```

Add custom environment variables to `bin/release`:

```bash
cat <<EOF
---
default_process_types:
  web: python app.py
build_variables:
  BUILD_TIME: "$(date)"
config_vars:
  PYTHONUNBUFFERED: "true"
  PYTHONHASHSEED: "random"
EOF
```

## Verification

- ✅ Buildpack structure correct
- ✅ Detect script works
- ✅ Compile script executes
- ✅ Release provides runtime config
- ✅ App deploys with custom buildpack
- ✅ Build output visible
- ✅ Dependencies installed
- ✅ App runs successfully

## Key Learning

1. **Buildpack Phases**: Detect → Compile → Release
2. **Customization**: Control build process completely
3. **Language Support**: Add support for any language
4. **Caching**: Speed up builds with persistent cache
5. **Environment**: Inject variables for runtime
6. **Debugging**: Can SSH to inspect buildpack state

---

Next: [Lab 6: Monitoring & Logging](06-monitoring-lab.md)
