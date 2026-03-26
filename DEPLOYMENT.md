# Deployment Guide

This guide covers deploying the Django Web App to various environments.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Container Registry](#container-registry)
- [Kubernetes Deployment](#kubernetes-deployment)
- [Environment Configuration](#environment-configuration)
- [Security](#security)
- [Monitoring and Maintenance](#monitoring-and-maintenance)
- [Troubleshooting](#troubleshooting)

## Prerequisites

### Required Tools

- kubectl (Kubernetes CLI)
- Docker or compatible container runtime
- Access to a Kubernetes cluster
- GitHub account with repository access

### Kubernetes Cluster

Supported platforms:
- Local: Docker Desktop, Minikube, Kind
- Cloud: GKE, EKS, AKS, DigitalOcean Kubernetes
- Self-hosted: kubeadm, k3s, RKE

## Container Registry

### GitHub Container Registry (ghcr.io)

This project uses GitHub Container Registry for storing container images.

#### Pulling Images

1. **Authenticate to ghcr.io**
   ```bash
   echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin
   ```

2. **Pull the image**
   ```bash
   docker pull ghcr.io/hnijbacker/django-web-app:latest
   ```

3. **Verify image signature**
   ```bash
   cosign verify \
     --certificate-identity-regexp="https://github.com/.*/.github/workflows/.*" \
     --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
     ghcr.io/hnijbacker/django-web-app:latest
   ```

#### Available Tags

- `latest`: Latest stable release
- `v*.*.*`: Semantic version tags (e.g., `v1.0.0`)
- `v*.*.*-*`: Version with git SHA (e.g., `v1.0.0-abc1234`)

## Kubernetes Deployment

### Quick Start

Deploy to development environment:
```bash
kubectl apply -k kustomize/overlays/dev/
```

Deploy to production environment:
```bash
kubectl apply -k kustomize/overlays/prod/
```

### Using OCI Bundles

The CI/CD pipeline publishes Kustomize configurations as OCI artifacts.

1. **Install ORAS CLI**
   ```bash
   # macOS
   brew install oras

   # Linux
   curl -LO https://github.com/oras-project/oras/releases/download/v1.0.0/oras_1.0.0_linux_amd64.tar.gz
   tar -xzf oras_1.0.0_linux_amd64.tar.gz
   sudo mv oras /usr/local/bin/
   ```

2. **Pull Kustomize bundle**
   ```bash
   oras pull ghcr.io/hnijbacker/django-web-app-config:v1.0.0
   ```

3. **Extract and apply**
   ```bash
   tar -xzf kustomize.tar.gz
   kubectl apply -k kustomize/overlays/prod/
   ```

4. **Verify bundle signature**
   ```bash
   cosign verify \
     --certificate-identity-regexp="https://github.com/.*/.github/workflows/.*" \
     --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
     ghcr.io/hnijbacker/django-web-app-config:v1.0.0
   ```

### Customizing Deployments

#### Development Environment

Located at `kustomize/overlays/dev/`:

```bash
# Preview changes
kubectl kustomize kustomize/overlays/dev/

# Apply
kubectl apply -k kustomize/overlays/dev/

# Get resources
kubectl get all -n django-dev
```

Features:
- Single replica
- Development namespace
- Lower resource limits
- Local persistent volume

#### Production Environment

Located at `kustomize/overlays/prod/`:

```bash
# Preview changes
kubectl kustomize kustomize/overlays/prod/

# Apply
kubectl apply -k kustomize/overlays/prod/

# Get resources
kubectl get all -n django-prod
```

Features:
- Multiple replicas for HA
- Production namespace
- Higher resource limits
- Production-grade storage

### Step-by-Step Deployment

1. **Create namespace**
   ```bash
   kubectl create namespace django-prod
   ```

2. **Create secrets**
   ```bash
   kubectl create secret generic django-secrets \
     --from-literal=SECRET_KEY='your-secret-key' \
     --from-literal=DATABASE_URL='postgresql://user:pass@host/db' \
     -n django-prod
   ```

3. **Create configmaps**
   ```bash
   kubectl create configmap django-config \
     --from-literal=DEBUG='False' \
     --from-literal=ALLOWED_HOSTS='yourdomain.com,www.yourdomain.com' \
     -n django-prod
   ```

4. **Apply Kustomize configuration**
   ```bash
   kubectl apply -k kustomize/overlays/prod/
   ```

5. **Verify deployment**
   ```bash
   kubectl get pods -n django-prod
   kubectl get services -n django-prod
   kubectl get ingress -n django-prod
   ```

6. **Check logs**
   ```bash
   kubectl logs -f deployment/django-app -n django-prod
   ```

## Environment Configuration

### Required Environment Variables

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `SECRET_KEY` | Django secret key | Yes | - |
| `DEBUG` | Enable debug mode | No | `False` |
| `ALLOWED_HOSTS` | Allowed hostnames | Yes | - |
| `DATABASE_URL` | Database connection | No | SQLite |

### Optional Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `STATIC_URL` | Static files URL | `/static/` |
| `MEDIA_URL` | Media files URL | `/media/` |
| `LOG_LEVEL` | Logging level | `INFO` |

### Database Configuration

#### SQLite (Development)

Default configuration, no setup required.

#### PostgreSQL (Production)

```bash
DATABASE_URL=postgresql://username:password@hostname:5432/database
```

#### MySQL

```bash
DATABASE_URL=mysql://username:password@hostname:3306/database
```

### Secrets Management

#### Using Kubernetes Secrets

```bash
kubectl create secret generic django-secrets \
  --from-literal=SECRET_KEY='$(python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())")' \
  --from-literal=DATABASE_URL='postgresql://user:pass@host/db' \
  -n django-prod
```

#### Using External Secrets Operator

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: django-secrets
spec:
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: django-secrets
  data:
    - secretKey: SECRET_KEY
      remoteRef:
        key: django/secret-key
    - secretKey: DATABASE_URL
      remoteRef:
        key: django/database-url
```

## Security

### Image Verification

Always verify container signatures before deployment:

```bash
# Install Cosign
brew install cosign  # macOS
# or download from https://github.com/sigstore/cosign/releases

# Verify image
cosign verify \
  --certificate-identity-regexp="https://github.com/.*/.github/workflows/.*" \
  --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
  ghcr.io/hnijbacker/django-web-app:v1.0.0
```

### Network Security

#### Ingress Configuration

The application supports multiple ingress options:

1. **Gateway API** (Recommended)
   - Modern Kubernetes ingress
   - Advanced traffic management
   - Configuration: `kustomize/base/django-gatewayapi.yaml`

2. **IngressRoute** (Traefik)
   - Traefik-specific ingress
   - Custom routing features
   - Configuration: `kustomize/base/django-ingressroute.yaml`

#### TLS Configuration

```yaml
spec:
  tls:
    - hosts:
        - yourdomain.com
      secretName: django-tls-cert
```

Generate certificate:
```bash
kubectl create secret tls django-tls-cert \
  --cert=path/to/cert.crt \
  --key=path/to/key.key \
  -n django-prod
```

### Pod Security

The deployment uses:
- Non-root user
- Read-only root filesystem where possible
- Dropped Linux capabilities
- Security contexts

## Monitoring and Maintenance

### Health Checks

The application exposes health endpoints:
- Liveness: `/` (checks if app is running)
- Readiness: `/` (checks if app is ready to serve traffic)

### Logging

View application logs:
```bash
# All pods
kubectl logs -l app=django-app -n django-prod

# Specific pod
kubectl logs django-app-xxxxx -n django-prod

# Follow logs
kubectl logs -f deployment/django-app -n django-prod

# Last 100 lines
kubectl logs --tail=100 deployment/django-app -n django-prod
```

### Scaling

#### Manual Scaling

```bash
# Scale up
kubectl scale deployment django-app --replicas=5 -n django-prod

# Scale down
kubectl scale deployment django-app --replicas=2 -n django-prod
```

#### Horizontal Pod Autoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: django-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: django-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

### Updates and Rollbacks

#### Rolling Updates

```bash
# Update image
kubectl set image deployment/django-app \
  django-app=ghcr.io/hnijbacker/django-web-app:v2.0.0 \
  -n django-prod

# Check rollout status
kubectl rollout status deployment/django-app -n django-prod
```

#### Rollback

```bash
# Rollback to previous version
kubectl rollout undo deployment/django-app -n django-prod

# Rollback to specific revision
kubectl rollout undo deployment/django-app --to-revision=2 -n django-prod

# View rollout history
kubectl rollout history deployment/django-app -n django-prod
```

### Database Migrations

```bash
# Run migrations manually
kubectl exec -it deployment/django-app -n django-prod -- \
  uv run manage.py migrate

# Create migration
kubectl exec -it deployment/django-app -n django-prod -- \
  uv run manage.py makemigrations
```

## Troubleshooting

### Pod Not Starting

```bash
# Check pod status
kubectl get pods -n django-prod

# Describe pod
kubectl describe pod django-app-xxxxx -n django-prod

# Check logs
kubectl logs django-app-xxxxx -n django-prod

# Check events
kubectl get events -n django-prod --sort-by='.lastTimestamp'
```

### Database Connection Issues

```bash
# Test database connectivity
kubectl exec -it deployment/django-app -n django-prod -- \
  uv run manage.py dbshell

# Check environment variables
kubectl exec deployment/django-app -n django-prod -- env | grep DATABASE
```

### Image Pull Errors

```bash
# Create image pull secret
kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username=$GITHUB_USERNAME \
  --docker-password=$GITHUB_TOKEN \
  -n django-prod

# Update deployment to use secret
kubectl patch serviceaccount default \
  -p '{"imagePullSecrets": [{"name": "ghcr-secret"}]}' \
  -n django-prod
```

### Performance Issues

```bash
# Check resource usage
kubectl top pods -n django-prod
kubectl top nodes

# Check resource requests/limits
kubectl describe deployment django-app -n django-prod | grep -A 5 "Limits:"
```

### Connectivity Issues

```bash
# Test service connectivity
kubectl run -it --rm debug --image=busybox --restart=Never -n django-prod -- sh
# Inside container:
wget -O- http://django-service:8000

# Check service endpoints
kubectl get endpoints django-service -n django-prod
```

## CI/CD Integration

### GitHub Actions

The deployment is automated via GitHub Actions. See [.github/workflows/ci.yml](.github/workflows/ci.yml).

Trigger deployment:
```bash
# Create and push a tag
git tag v1.0.0
git push origin v1.0.0
```

### Manual Deployment from CI/CD Artifacts

```bash
# Download artifacts from GitHub Actions
gh run download <run-id>

# Or use ORAS to pull bundle
oras pull ghcr.io/hnijbacker/django-web-app-config:v1.0.0

# Deploy
tar -xzf kustomize.tar.gz
kubectl apply -k kustomize/overlays/prod/
```

## Best Practices

1. **Always verify image signatures** before deploying
2. **Use specific version tags** in production (not `latest`)
3. **Test in dev environment** before deploying to production
4. **Monitor resource usage** and adjust limits accordingly
5. **Keep secrets secure** using proper secrets management
6. **Perform rolling updates** to minimize downtime
7. **Keep backups** of database and persistent volumes
8. **Document changes** and maintain deployment history
9. **Use health checks** for automatic recovery
10. **Monitor logs** for errors and security issues
