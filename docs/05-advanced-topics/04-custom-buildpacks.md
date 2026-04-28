# Custom Buildpacks

## When to Build Custom Buildpacks

**Use custom buildpacks for:**
- Unsupported languages
- Custom build toolchain
- Organization-specific conventions
- Pre-built binaries

**Don't use if:**
- Standard buildpack exists
- Only need simple customization
- Build process is complex

## Buildpack Structure

```
my-buildpack/
├── bin/
│   ├── compile    (bash script)
│   ├── detect     (bash script)
│   └── release    (bash script)
├── lib/
│   └── helpers.sh (optional)
└── README.md
```

## Detect Script

Determines if buildpack applies:

```bash
#!/bin/bash
# $1 = BUILD_DIR (app source directory)

BUILD_DIR=$1

if [[ -f "$BUILD_DIR/Gemfile" ]]; then
  echo "Ruby"
  exit 0
fi

exit 1
```

**Exit codes:**
- 0: Buildpack applies (print language name)
- 1: Buildpack doesn't apply

## Compile Script

Build application:

```bash
#!/bin/bash
# $1 = BUILD_DIR (app source)
# $2 = CACHE_DIR (persistent, reused across builds)
# $3 = DEPS_DIR (shared dependencies)

BUILD_DIR=$1
CACHE_DIR=$2
DEPS_DIR=$3

echo "-----> Installing Ruby"

# Download or copy pre-built binary
mkdir -p $DEPS_DIR/ruby/bin
cp /opt/ruby-3.0.0/bin/* $DEPS_DIR/ruby/bin/

# Export environment for later phases
export PATH="$DEPS_DIR/ruby/bin:$PATH"

cd $BUILD_DIR

# Install dependencies
echo "-----> Running bundler"
gem install bundler
bundle install

echo "-----> Compile successful"
```

## Release Script

Runtime configuration:

```bash
#!/bin/bash
# $1 = BUILD_DIR

BUILD_DIR=$1

cat <<EOF
---
default_process_types:
  web: bundle exec rails s -p $PORT
EOF
```

## Pre-Compiled Binaries

Optimize build time by pre-compiling:

```bash
# In compile script
if [[ ! -d "$CACHE_DIR/ruby-3.0.0" ]]; then
  echo "-----> Downloading Ruby"
  wget -O ruby.tar.gz https://example.com/ruby-3.0.0.tar.gz
  tar -xzf ruby.tar.gz -C $CACHE_DIR
fi

cp -r $CACHE_DIR/ruby-3.0.0 $DEPS_DIR/
```

## Testing Buildpack Locally

Create test app:

```bash
mkdir test-app
cd test-app

# Create trigger file
touch Gemfile (for Ruby example)

# Manual test
export BUILD_DIR=$(pwd)
export CACHE_DIR=$(mktemp -d)
export DEPS_DIR=$(mktemp -d)

# Run detect
../my-buildpack/bin/detect $BUILD_DIR
# Should print "Ruby"

# Run compile
../my-buildpack/bin/compile $BUILD_DIR $CACHE_DIR $DEPS_DIR
# Should build successfully
```

## Best Practices

1. **Fail Fast**: Detect early if buildpack applies
2. **Cache**: Reuse downloaded files
3. **Output**: Print progress messages with `---->`
4. **Errors**: Exit with clear error messages
5. **Environment**: Export variables for next phase
6. **Security**: Don't include secrets in scripts
7. **Testing**: Test locally before uploading

## Publishing Buildpack

```bash
# Package
zip -r my-buildpack.zip .

# Upload to foundation
cf create-buildpack my-buildpack my-buildpack.zip 10

# Users can reference it
cf push my-app --buildpack my-buildpack
```

---

See [Lab 5: Custom Buildpack](../06-labs/05-custom-buildpack.md) for hands-on buildpack creation.
