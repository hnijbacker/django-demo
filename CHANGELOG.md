# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

This file is automatically updated by [semantic-release](https://github.com/semantic-release/semantic-release).

## [Unreleased]

### Added
- Initial Django web application with polls functionality
- Docker containerization with uv package manager
- Kubernetes deployment with Kustomize
- GitHub Actions CI/CD pipeline
- Container image signing with Sigstore Cosign
- OCI artifacts for Kustomize bundles
- GitHub Container Registry integration
- Comprehensive documentation

### Changed
- Migrated from Docker Hub to GitHub Container Registry (ghcr.io)

### Security
- Implemented container image signing with keyless OIDC
- Added security policies and vulnerability reporting process

---

**Note**: This changelog is maintained automatically by semantic-release based on commit messages following the Conventional Commits specification. Manual entries should be added to the [Unreleased] section only.
