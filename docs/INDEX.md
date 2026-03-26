# Documentation Index

Welcome to the Django Web App documentation. This index provides quick access to all available documentation.

## Getting Started

Perfect for first-time users and quick setup.

| Document | Description | When to Use |
|----------|-------------|-------------|
| [QUICKSTART.md](../QUICKSTART.md) | 5-minute quick start guide | Getting started immediately |
| [README.md](../README.md) | Project overview and introduction | Understanding the project |

## Development

Essential guides for developers working on the project.

| Document | Description | When to Use |
|----------|-------------|-------------|
| [DEVELOPMENT.md](../DEVELOPMENT.md) | Comprehensive development guide | Setting up dev environment, daily workflows |
| [ARCHITECTURE.md](../ARCHITECTURE.md) | Technical architecture and design | Understanding system design, making architectural decisions |
| [CONTRIBUTING.md](../CONTRIBUTING.md) | Contribution guidelines | Before contributing code |

## Operations

Guides for deploying and operating the application.

| Document | Description | When to Use |
|----------|-------------|-------------|
| [DEPLOYMENT.md](../DEPLOYMENT.md) | Deployment guide | Deploying to Kubernetes, production setup |
| [SECURITY.md](../SECURITY.md) | Security policies and practices | Security configuration, vulnerability reporting |

## Reference

| Document | Description | When to Use |
|----------|-------------|-------------|
| [CHANGELOG.md](../CHANGELOG.md) | Version history | Checking what changed between versions |

## Documentation by Role

### For New Users
1. [QUICKSTART.md](../QUICKSTART.md) - Get started in 5 minutes
2. [README.md](../README.md) - Understand the project
3. [DEVELOPMENT.md](../DEVELOPMENT.md) - Set up your environment

### For Contributors
1. [CONTRIBUTING.md](../CONTRIBUTING.md) - Understand contribution process
2. [DEVELOPMENT.md](../DEVELOPMENT.md) - Set up development environment
3. [ARCHITECTURE.md](../ARCHITECTURE.md) - Understand the architecture

### For Operators/DevOps
1. [DEPLOYMENT.md](../DEPLOYMENT.md) - Deploy to Kubernetes
2. [SECURITY.md](../SECURITY.md) - Configure security
3. [ARCHITECTURE.md](../ARCHITECTURE.md) - Understand infrastructure

### For Security Researchers
1. [SECURITY.md](../SECURITY.md) - Report vulnerabilities
2. [ARCHITECTURE.md](../ARCHITECTURE.md) - Understand security architecture

## Documentation by Task

### Setup and Installation
- Local development: [QUICKSTART.md](../QUICKSTART.md) → [DEVELOPMENT.md](../DEVELOPMENT.md)
- Docker: [QUICKSTART.md](../QUICKSTART.md) → [README.md](../README.md)
- Kubernetes: [DEPLOYMENT.md](../DEPLOYMENT.md)

### Development
- First-time setup: [DEVELOPMENT.md](../DEVELOPMENT.md) → Environment Setup
- Daily workflow: [DEVELOPMENT.md](../DEVELOPMENT.md) → Development Workflow
- Testing: [DEVELOPMENT.md](../DEVELOPMENT.md) → Testing
- Debugging: [DEVELOPMENT.md](../DEVELOPMENT.md) → Debugging

### Deployment
- Initial deployment: [DEPLOYMENT.md](../DEPLOYMENT.md) → Quick Start
- Configuration: [DEPLOYMENT.md](../DEPLOYMENT.md) → Environment Configuration
- Updates: [DEPLOYMENT.md](../DEPLOYMENT.md) → Updates and Rollbacks
- Troubleshooting: [DEPLOYMENT.md](../DEPLOYMENT.md) → Troubleshooting

### Contributing
- First contribution: [CONTRIBUTING.md](../CONTRIBUTING.md) → Getting Started
- Commit format: [CONTRIBUTING.md](../CONTRIBUTING.md) → Commit Convention
- Pull requests: [CONTRIBUTING.md](../CONTRIBUTING.md) → Pull Request Process

### Security
- Reporting vulnerabilities: [SECURITY.md](../SECURITY.md) → Reporting a Vulnerability
- Image verification: [SECURITY.md](../SECURITY.md) → Container Image Security
- Production checklist: [SECURITY.md](../SECURITY.md) → Security Checklist

## Quick Links

### Code
- [django_project/](../django_project/) - Main Django project
- [polls/](../polls/) - Example Django app
- [kustomize/](../kustomize/) - Kubernetes manifests

### Configuration
- [pyproject.toml](../pyproject.toml) - Python dependencies
- [Dockerfile](../Dockerfile) - Container image
- [.github/workflows/ci.yml](../.github/workflows/ci.yml) - CI/CD pipeline
- [.env.example](../.env.example) - Environment variables template

### Resources
- [Django Documentation](https://docs.djangoproject.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Sigstore Documentation](https://docs.sigstore.dev/)
- [uv Documentation](https://github.com/astral-sh/uv)

## Documentation Standards

### Updating Documentation

When updating documentation:
1. Keep it concise and actionable
2. Use clear headings and structure
3. Include code examples where helpful
4. Add diagrams for complex concepts
5. Link between related documents
6. Update this index if adding new files

### Documentation Format

All documentation uses:
- **Markdown** for formatting
- **Conventional structure** (consistent sections)
- **Clear examples** with code blocks
- **Cross-references** for related content

## Getting Help

Can't find what you're looking for?

1. **Search documentation**: Use your editor's search (Cmd/Ctrl+Shift+F)
2. **Check GitHub Issues**: Existing solutions may exist
3. **Ask questions**: Open a GitHub Discussion
4. **Report gaps**: Create an issue to improve docs

## Contributing to Documentation

Documentation improvements are always welcome!

1. Found a typo? Fix it and submit a PR
2. Found missing information? Add it and submit a PR
3. Found confusing sections? Improve clarity and submit a PR

See [CONTRIBUTING.md](../CONTRIBUTING.md) for details.

---

**Last Updated**: 2026-03-26
