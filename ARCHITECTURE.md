# Architecture Documentation

This document describes the technical architecture, design decisions, and implementation details of the Django Web App.

## Table of Contents

- [Overview](#overview)
- [System Architecture](#system-architecture)
- [Application Architecture](#application-architecture)
- [Infrastructure Architecture](#infrastructure-architecture)
- [CI/CD Pipeline](#cicd-pipeline)
- [Security Architecture](#security-architecture)
- [Design Decisions](#design-decisions)
- [Future Considerations](#future-considerations)

## Overview

The Django Web App is a containerized, cloud-native web application designed for Kubernetes deployment with automated CI/CD, security signing, and infrastructure-as-code practices.

### Key Principles

- **Cloud-Native**: Designed for containerized deployment
- **Security-First**: Image signing, verification, and secure-by-default configuration
- **Automation**: Fully automated build, test, and deployment pipeline
- **Scalability**: Horizontal scaling via Kubernetes
- **Maintainability**: Clear separation of concerns and comprehensive documentation

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         End Users                           │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    Ingress/Gateway API                      │
│                   (TLS Termination)                         │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                   Kubernetes Service                        │
│                  (Load Balancing)                           │
└───────────────────────────┬─────────────────────────────────┘
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
         ┌────────┐    ┌────────┐    ┌────────┐
         │  Pod 1 │    │  Pod 2 │    │  Pod N │
         │ Django │    │ Django │    │ Django │
         └────┬───┘    └────┬───┘    └────┬───┘
              │             │             │
              └─────────────┼─────────────┘
                            ▼
                   ┌─────────────────┐
                   │    Database     │
                   │  (PostgreSQL)   │
                   └─────────────────┘
```

### Components

1. **Ingress/Gateway**: Routes external traffic to services
2. **Kubernetes Service**: Load balances across pod replicas
3. **Django Pods**: Application containers running the web app
4. **Database**: Persistent data storage (SQLite in dev, PostgreSQL in prod)

## Application Architecture

### Django Project Structure

```
django-demo/
├── django_project/          # Project configuration
│   ├── settings.py          # Application settings
│   ├── urls.py              # URL routing
│   ├── wsgi.py              # WSGI entry point
│   └── asgi.py              # ASGI entry point
│
└── polls/                   # Example Django app
    ├── models.py            # Data models
    ├── views.py             # Business logic
    ├── urls.py              # App-specific URLs
    ├── admin.py             # Admin interface
    ├── templates/           # HTML templates
    └── migrations/          # Database migrations
```

### Request Flow

```
User Request
    │
    ▼
Ingress/Gateway (HTTPS)
    │
    ▼
Kubernetes Service
    │
    ▼
Django Pod
    │
    ├──▶ URL Router (urls.py)
    │       │
    │       ▼
    │   View (views.py)
    │       │
    │       ├──▶ Model (models.py)
    │       │       │
    │       │       ▼
    │       │   Database
    │       │
    │       ▼
    │   Template (templates/)
    │
    ▼
Response (HTML/JSON)
```

### Technology Stack

#### Backend
- **Django 5.2.4**: Web framework
- **Python 3.12**: Programming language
- **uv**: Package manager and virtual environment

#### Frontend
- Django Templates: Server-side rendering
- Static files: CSS, JavaScript, images

#### Database
- **Development**: SQLite (file-based)
- **Production**: PostgreSQL (recommended)

## Infrastructure Architecture

### Containerization

#### Dockerfile Strategy

```dockerfile
# Multi-stage build for optimization
FROM python:3.12-slim-bookworm

# Install uv package manager
COPY --from=ghcr.io/astral-sh/uv:0.6.17 /uv /uvx /bin/

# Install dependencies (cached layer)
COPY pyproject.toml uv.lock /_lock/
RUN uv sync --frozen --no-install-project

# Copy application code
COPY . .

# Run via entrypoint script
CMD ["./docker-entrypoint.sh"]
```

**Benefits:**
- Minimal base image (slim-bookworm)
- Fast dependency installation with uv
- Cached layers for quick rebuilds
- Non-root user execution
- Reproducible builds with lock file

### Kubernetes Architecture

#### Resource Hierarchy

```
Namespace (django-prod/django-dev)
    │
    ├── Deployment
    │   └── ReplicaSet
    │       ├── Pod 1
    │       ├── Pod 2
    │       └── Pod N
    │
    ├── Service
    │   └── ClusterIP/LoadBalancer
    │
    ├── Ingress/IngressRoute
    │   └── Gateway/HTTPRoute
    │
    ├── ConfigMap
    │   └── Application configuration
    │
    ├── Secret
    │   └── Sensitive data
    │
    └── PersistentVolumeClaim
        └── Database storage
```

#### Kustomize Structure

```
kustomize/
├── base/                    # Base configuration
│   ├── kustomization.yaml   # Base resource list
│   ├── django-service.yaml  # Service definition
│   ├── django-gatewayapi.yaml    # Gateway API ingress
│   └── django-ingressroute.yaml  # Traefik ingress
│
└── overlays/                # Environment-specific
    ├── dev/                 # Development
    │   ├── kustomization.yaml
    │   ├── ns.yaml
    │   ├── pv.yaml
    │   └── pvc.yaml
    │
    └── prod/                # Production
        ├── kustomization.yaml
        ├── ns.yaml
        ├── pv.yaml
        └── pvc.yaml
```

**Overlay Pattern Benefits:**
- Environment-specific configurations
- No duplication of base resources
- Easy to manage and version
- Clear separation of concerns

## CI/CD Pipeline

### GitHub Actions Workflow

```
Trigger: Push tag (v*)
    │
    ▼
┌─────────────────────────┐
│   Semantic Release      │ ◀── Analyze commits
│   (Version Generation)  │     Generate changelog
└──────────┬──────────────┘     Create GitHub release
           │
           ▼
┌─────────────────────────┐
│   Docker Build          │ ◀── Build container image
│   (Multi-arch)          │     Push to ghcr.io
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│   Cosign Signing        │ ◀── Sign image (keyless)
│   (Container Image)     │     OIDC authentication
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│   Kustomize Bundle      │ ◀── Package manifests
│   (OCI Artifact)        │     Push with ORAS
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│   Cosign Signing        │ ◀── Sign bundle (keyless)
│   (Kustomize Bundle)    │     OIDC authentication
└─────────────────────────┘
```

### Pipeline Stages

1. **Semantic Release**
   - Analyzes commit messages (Conventional Commits)
   - Determines version bump (major/minor/patch)
   - Generates changelog
   - Creates GitHub release

2. **Docker Build**
   - Builds container image
   - Tags with version and SHA
   - Pushes to GitHub Container Registry
   - Multi-platform support (optional)

3. **Image Signing**
   - Signs with Sigstore Cosign
   - Keyless signing via OIDC
   - Certificate stored in transparency log
   - Verifiable by anyone

4. **Kustomize Bundle**
   - Packages Kubernetes manifests
   - Creates OCI artifact with ORAS
   - Pushes to container registry
   - Version-tagged bundle

5. **Bundle Signing**
   - Signs Kustomize bundle
   - Same process as image signing
   - Ensures manifest integrity

## Security Architecture

### Container Image Security

#### Signing Flow

```
GitHub Actions Workflow
    │
    ▼
Generate OIDC Token
    │
    ▼
Sigstore Fulcio CA
    │ (Issues short-lived certificate)
    ▼
Sign Image with Cosign
    │
    ▼
Store in Rekor Transparency Log
    │
    ▼
Push Signature to Registry
```

#### Verification Flow

```
Pull Image
    │
    ▼
cosign verify
    │
    ├──▶ Verify Certificate Identity
    │    (GitHub Actions workflow)
    │
    ├──▶ Verify OIDC Issuer
    │    (token.actions.githubusercontent.com)
    │
    └──▶ Check Transparency Log
         (Rekor)
    │
    ▼
Verified ✓ / Failed ✗
```

### Security Layers

1. **Image Scanning**: Vulnerability detection
2. **Image Signing**: Authenticity verification
3. **RBAC**: Access control in Kubernetes
4. **Network Policies**: Traffic restriction
5. **Pod Security**: Non-root, restricted capabilities
6. **Secrets Management**: Encrypted sensitive data

## Design Decisions

### Why uv over pip?

**Decision**: Use uv for package management

**Rationale:**
- 10-100x faster than pip
- Built-in virtual environment management
- Better dependency resolution
- Modern lockfile format
- Drop-in replacement for pip

**Trade-offs:**
- Newer tool (less mature)
- Smaller community
- Requires team familiarity

### Why GitHub Container Registry?

**Decision**: Use ghcr.io instead of Docker Hub

**Rationale:**
- Integrated with GitHub
- Built-in OIDC support
- No rate limiting for authenticated users
- Better CI/CD integration
- Unified permissions model

**Trade-offs:**
- Less well-known than Docker Hub
- Requires GitHub account

### Why Kustomize over Helm?

**Decision**: Use Kustomize for Kubernetes manifests

**Rationale:**
- Native to kubectl (no separate tool)
- Declarative overlay pattern
- No templating language to learn
- Better for GitOps workflows
- Simpler for small applications

**Trade-offs:**
- Less powerful than Helm for complex apps
- No chart repository ecosystem
- Limited logic capabilities

### Why Sigstore Cosign?

**Decision**: Use keyless signing with Cosign

**Rationale:**
- No long-lived key management
- OIDC-based authentication
- Transparent verification log
- Industry standard
- Easy CI/CD integration

**Trade-offs:**
- Requires internet connectivity
- Depends on external services (Fulcio, Rekor)
- Learning curve for verification

### Why Semantic Release?

**Decision**: Automate versioning with semantic-release

**Rationale:**
- Automated version bumping
- Changelog generation
- Conventional Commits enforcement
- GitHub release automation
- Reduces human error

**Trade-offs:**
- Requires strict commit conventions
- Less control over versions
- Team must adopt conventions

## Future Considerations

### Potential Improvements

1. **Multi-Stage Database Migrations**
   - Separate init containers for migrations
   - Blue-green deployment support
   - Zero-downtime updates

2. **Observability**
   - Prometheus metrics
   - OpenTelemetry tracing
   - Centralized logging (ELK/Loki)
   - APM integration

3. **Advanced Deployment**
   - Canary deployments
   - Progressive delivery with Flagger
   - GitOps with ArgoCD/Flux

4. **Performance**
   - Redis caching
   - CDN for static files
   - Connection pooling
   - Query optimization

5. **High Availability**
   - Multi-region deployment
   - Database replication
   - Disaster recovery plan
   - Backup automation

6. **Testing**
   - Integration tests
   - End-to-end tests
   - Performance tests
   - Security scanning (SAST/DAST)

7. **Compliance**
   - SBOM generation
   - License scanning
   - Vulnerability remediation
   - Audit logging

### Scalability Path

```
Phase 1: Single Node (Current)
    ├── SQLite database
    ├── Single pod
    └── Local storage

Phase 2: Horizontal Scaling
    ├── PostgreSQL database
    ├── Multiple pods
    ├── Shared storage
    └── Load balancing

Phase 3: High Availability
    ├── Database replication
    ├── Pod autoscaling
    ├── Multiple availability zones
    └── CDN integration

Phase 4: Multi-Region
    ├── Geographic distribution
    ├── Edge caching
    ├── Data replication
    └── Global load balancing
```

## References

- [Django Documentation](https://docs.djangoproject.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Sigstore Documentation](https://docs.sigstore.dev/)
- [uv Documentation](https://github.com/astral-sh/uv)
- [Kustomize Documentation](https://kustomize.io/)
- [Semantic Release](https://semantic-release.gitbook.io/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [ORAS Documentation](https://oras.land/)
