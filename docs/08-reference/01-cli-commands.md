# CF CLI Commands Reference

## Authentication

```bash
cf login -a <API>                    # Login with password
cf login -a <API> --sso               # Login with SSO
cf logout                             # Logout
cf auth <USERNAME> <PASSWORD>         # Authenticate
cf target                             # Show current target
cf target -o ORG -s SPACE             # Switch org/space
```

## Organizations and Spaces

```bash
# Organizations
cf orgs                                # List organizations
cf create-org <ORG>                   # Create org
cf delete-org <ORG>                   # Delete org
cf org <ORG>                          # Show org details

# Spaces
cf spaces                              # List spaces in current org
cf create-space <SPACE>               # Create space
cf delete-space <SPACE>               # Delete space
cf space <SPACE>                      # Show space details
```

## Applications

```bash
# Push and manage
cf push <APP>                         # Deploy app
cf push <APP> -i 3                    # Deploy with 3 instances
cf push <APP> -m 1G                   # Deploy with 1GB memory
cf push <APP> --health-check-type http  # With health checks

cf apps                               # List apps
cf app <APP>                          # Show app details
cf start <APP>                        # Start app
cf stop <APP>                         # Stop app
cf restart <APP>                      # Restart app
cf restage <APP>                      # Restage app
cf delete <APP>                       # Delete app

# Scaling
cf scale <APP> -i 5                   # Set to 5 instances
cf scale <APP> -m 2G                  # Set to 2GB memory
cf scale <APP> -i 3 -m 512M           # Both instance and memory

# Environment
cf env <APP>                          # Show environment
cf set-env <APP> KEY VALUE            # Set variable
cf unset-env <APP> KEY                # Unset variable
cf logs <APP>                         # Stream logs
cf logs <APP> --recent                # Show recent logs
cf events <APP>                       # Show events
cf stats <APP>                        # Show resource usage
```

## Routes and Domains

```bash
# Routes
cf routes                             # List all routes
cf app-routes <APP>                   # Routes for app
cf create-route <DOMAIN> --hostname <HOST>  # Create route
cf map-route <APP> <DOMAIN> --hostname <HOST>    # Map to app
cf unmap-route <APP> <DOMAIN> --hostname <HOST>  # Unmap from app
cf delete-route <DOMAIN> --hostname <HOST>       # Delete route

# Domains
cf domains                            # List domains
cf create-domain <ORG> <DOMAIN>       # Create domain
cf delete-domain <DOMAIN>             # Delete domain
```

## Services and Bindings

```bash
# Services
cf marketplace                        # Show available services
cf services                           # List service instances
cf create-service <SERVICE> <PLAN> <NAME>     # Create service
cf update-service <SERVICE> -p <NEWPLAN>      # Change plan
cf delete-service <SERVICE>           # Delete service
cf service <SERVICE>                  # Show service details

# Bindings
cf bind-service <APP> <SERVICE>       # Bind service to app
cf unbind-service <APP> <SERVICE>     # Unbind service
cf bind-route-service <DOMAIN> <SERVICE> --hostname <HOST>  # Route service

# Service keys
cf create-service-key <SERVICE> <KEY>         # Create key
cf service-key <SERVICE> <KEY>                # Show key
cf delete-service-key <SERVICE> <KEY>         # Delete key
```

## Users and Roles

```bash
# Users
cf create-user <USERNAME> <PASSWORD>  # Create user
cf delete-user <USERNAME>             # Delete user
cf passwd                             # Change password

# Organization roles
cf org-users <ORG>                    # Show org users
cf set-org-role <USER> <ORG> <ROLE>   # Assign org role
cf unset-org-role <USER> <ORG> <ROLE> # Remove org role

# Space roles
cf space-users <ORG> <SPACE>          # Show space users
cf set-space-role <USER> <ORG> <SPACE> <ROLE>    # Assign role
cf unset-space-role <USER> <ORG> <SPACE> <ROLE>  # Remove role
```

## Quotas and Limits

```bash
# Organization quotas
cf quotas                             # List quotas
cf create-quota <QUOTA> ...           # Create quota
cf set-quota <ORG> <QUOTA>            # Apply quota
cf quota <QUOTA>                      # Show quota details

# Space quotas
cf space-quotas                       # List space quotas
cf create-space-quota <QUOTA>         # Create space quota
cf set-space-quota <SPACE> <QUOTA>    # Apply space quota
```

## Buildpacks

```bash
cf buildpacks                         # List buildpacks
cf create-buildpack <NAME> <FILE> <POSITION>  # Create
cf delete-buildpack <NAME>            # Delete
cf update-buildpack <NAME> <FILE>     # Update
```

## Security Groups

```bash
cf security-groups                    # List groups
cf create-security-group <GROUP> <RULES>      # Create
cf delete-security-group <GROUP>              # Delete
cf bind-security-group <GROUP> <ORG> <SPACE> # Bind
cf unbind-security-group <GROUP> <ORG> <SPACE>  # Unbind
```

## Advanced

```bash
# Manifest
cf push -f manifest.yml               # Deploy with manifest

# SSH into app
cf ssh <APP>                          # SSH into instance 0
cf ssh <APP> -i 2                     # SSH into instance 2

# SSH key management
cf enable-ssh <APP>                   # Enable SSH
cf disable-ssh <APP>                  # Disable SSH

# One-off tasks
cf run-task <APP> "<CMD>"             # Run command in app
cf tasks <APP>                        # List running tasks
```

---

For manifest syntax, see [Manifest Reference](02-manifest-reference.md).
