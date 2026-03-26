# Django Web App

A containerized Django application with automated CI/CD, Kubernetes deployment, and container image signing using Sigstore Cosign.

## Features

- **Django 5.2+** web application with polls functionality
- **Containerized** using Docker with optimized multi-stage builds
- **Package management** with [uv](https://github.com/astral-sh/uv) for fast dependency resolution
- **Kubernetes deployment** with Kustomize overlays for dev/prod environments
- **Automated CI/CD** with GitHub Actions
- **Semantic versioning** with semantic-release
- **Container signing** with Sigstore Cosign (keyless OIDC)
- **Container registry** using GitHub Container Registry (ghcr.io)
- **OCI artifacts** for Kustomize bundles using ORAS

## Quick Start

⚡ **Want to get started immediately?** Check out the [QUICKSTART.md](QUICKSTART.md) guide for a 5-minute setup!

## Documentation

- **[QUICKSTART.md](QUICKSTART.md)** - Get started in 5 minutes
- **[README.md](README.md)** - Project overview and quick start (this file)
- **[ARCHITECTURE.md](ARCHITECTURE.md)** - System architecture and design decisions
- **[DEVELOPMENT.md](DEVELOPMENT.md)** - Detailed development guide, local setup, and workflows
- **[DEPLOYMENT.md](DEPLOYMENT.md)** - Kubernetes deployment, configuration, and operations
- **[CONTRIBUTING.md](CONTRIBUTING.md)** - Contribution guidelines and commit conventions
- **[SECURITY.md](SECURITY.md)** - Security policies, vulnerability reporting, and best practices
- **[CHANGELOG.md](CHANGELOG.md)** - Version history and release notes

## Tech Stack

- **Backend**: Django 5.2.4, Python 3.12
- **Package Manager**: uv 0.6.17
- **Containerization**: Docker
- **Orchestration**: Kubernetes with Kustomize
- **CI/CD**: GitHub Actions
- **Registry**: GitHub Container Registry (ghcr.io)
- **Security**: Sigstore Cosign for container image signing

## Prerequisites

### Local Development
- Python 3.12+
- [uv](https://github.com/astral-sh/uv) package manager

### Docker Development
- Docker Desktop or Docker Engine

### Kubernetes Deployment
- Kubernetes cluster (local or cloud)
- kubectl
- Kustomize (built into kubectl)

## Getting Started

### Local Development

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd django-demo
   ```

2. **Install dependencies**
   ```bash
   uv sync
   ```

3. **Run migrations**
   ```bash
   uv run manage.py migrate
   ```

4. **Create a superuser (optional)**
   ```bash
   uv run manage.py createsuperuser
   ```

5. **Start the development server**
   ```bash
   uv run manage.py runserver
   ```

6. **Access the application**
   - Application: http://localhost:8000
   - Polls: http://localhost:8000/polls/
   - Admin: http://localhost:8000/admin/

### Docker Development

1. **Build the Docker image**
   ```bash
   docker build -t django-web-app:latest .
   ```

2. **Run the container**
   ```bash
   docker run -p 8000:8000 django-web-app:latest
   ```

3. **Access the application**
   - Application: http://localhost:8000

## Project Structure

```
.
├── .github/
│   └── workflows/
│       └── ci.yml              # CI/CD pipeline
├── django_project/             # Django project settings
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── polls/                      # Polls Django app
│   ├── models.py
│   ├── views.py
│   ├── urls.py
│   ├── templates/
│   │   └── polls/
│   │       └── index.html
│   └── admin.py
├── kustomize/                  # Kubernetes manifests
│   ├── base/                   # Base configuration
│   │   ├── kustomization.yaml
│   │   ├── django-service.yaml
│   │   ├── django-gatewayapi.yaml
│   │   └── django-ingressroute.yaml
│   └── overlays/               # Environment-specific configs
│       ├── dev/
│       └── prod/
├── Dockerfile                  # Container image definition
├── docker-entrypoint.sh        # Container startup script
├── pyproject.toml              # Python project dependencies
├── uv.lock                     # Locked dependencies
└── manage.py                   # Django management script
```

## Deployment

### Kubernetes with Kustomize

The application includes Kustomize configurations for different environments.

**Deploy to development:**
```bash
kubectl apply -k kustomize/overlays/dev/
```

**Deploy to production:**
```bash
kubectl apply -k kustomize/overlays/prod/
```

**Using OCI bundles (pushed by CI/CD):**
```bash
# Pull and extract the Kustomize bundle
oras pull ghcr.io/hnijbacker/django-web-app-config:v1.0.0

# Apply the configuration
tar -xzf kustomize.tar.gz
kubectl apply -k kustomize/overlays/prod/
```

### Environment Variables

Configure the application using environment variables:

- `DEBUG`: Enable/disable Django debug mode (default: False)
- `SECRET_KEY`: Django secret key (required in production)
- `ALLOWED_HOSTS`: Comma-separated list of allowed hosts
- `DATABASE_URL`: Database connection string

## CI/CD Pipeline

The GitHub Actions workflow ([.github/workflows/ci.yml](.github/workflows/ci.yml)) automates:

1. **Semantic Release**: Analyzes commits and generates version tags
2. **Docker Build**: Builds and pushes container images to ghcr.io
3. **Image Signing**: Signs images with Sigstore Cosign (keyless OIDC)
4. **Kustomize Bundle**: Packages and pushes Kubernetes manifests as OCI artifacts
5. **Bundle Signing**: Signs the Kustomize bundle with Cosign

### Trigger

The pipeline runs on:
- Push to tags matching `v*` pattern

### Container Images

Images are pushed to GitHub Container Registry:
- `ghcr.io/hnijbacker/django-web-app:latest`
- `ghcr.io/hnijbacker/django-web-app:v1.0.0`
- `ghcr.io/hnijbacker/django-web-app:v1.0.0-abc1234`

### Verifying Signatures

Verify the container image signature:
```bash
cosign verify \
  --certificate-identity-regexp="https://github.com/.*/.github/workflows/.*" \
  --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
  ghcr.io/hnijbacker/django-web-app:v1.0.0
```

## Development

### Running Tests

```bash
uv run manage.py test
```

### Creating Migrations

```bash
uv run manage.py makemigrations
```

### Collecting Static Files

```bash
uv run manage.py collectstatic
```

### Django Admin

Access the Django admin interface at `/admin/` after creating a superuser.

## Contributing

We welcome contributions! This project uses [Conventional Commits](https://www.conventionalcommits.org/) for semantic versioning.

**Quick reference:**
- `feat:` - New features (minor version bump)
- `fix:` - Bug fixes (patch version bump)
- `BREAKING CHANGE:` - Breaking changes (major version bump)
- `docs:` - Documentation updates
- `chore:` - Maintenance tasks

📖 **See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines** on:
- Development setup
- Code style and best practices
- Testing requirements
- Pull request process
- Commit conventions

## Security

This project implements multiple security measures:
- **Container signing** with Sigstore Cosign (keyless OIDC)
- **Image verification** before deployment
- **No long-lived credentials** in CI/CD
- **Regular dependency updates** via uv
- **Signed OCI artifacts** for Kubernetes manifests

🔒 **See [SECURITY.md](SECURITY.md) for**:
- Vulnerability reporting process
- Security best practices
- Verification instructions
- Security checklist

**Found a security issue?** Please report it privately according to our [security policy](SECURITY.md).

## License

[Add your license here]

## Support and Resources

- 📖 **Documentation**: See the [Documentation](#documentation) section above
- 🐛 **Issues**: [GitHub Issues](https://github.com/your-repo/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/your-repo/discussions)
- 🚀 **Getting Started**: Follow the [Quick Start](#getting-started) guide
- 🛠️ **Development**: Check out [DEVELOPMENT.md](DEVELOPMENT.md)
- 🚢 **Deployment**: Read [DEPLOYMENT.md](DEPLOYMENT.md)