# Contributing to Django Web App

Thank you for considering contributing to this project! This document provides guidelines and instructions for contributing.

## Getting Started

1. Fork the repository
2. Clone your fork locally
3. Set up your development environment
4. Create a new branch for your changes
5. Make your changes
6. Test your changes
7. Submit a pull request

## Development Setup

### Prerequisites

- Python 3.12 or higher
- [uv](https://github.com/astral-sh/uv) package manager
- Docker (for container testing)
- Git

### Setting Up Your Environment

```bash
# Clone your fork
git clone https://github.com/YOUR-USERNAME/django-demo.git
cd django-demo

# Install dependencies
uv sync

# Run migrations
uv run manage.py migrate

# Create a superuser for admin access
uv run manage.py createsuperuser

# Start the development server
uv run manage.py runserver
```

## Commit Convention

This project uses [Conventional Commits](https://www.conventionalcommits.org/) for automatic semantic versioning.

### Commit Message Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

- **feat**: A new feature (triggers a minor version bump)
- **fix**: A bug fix (triggers a patch version bump)
- **docs**: Documentation only changes
- **style**: Changes that don't affect code meaning (formatting, etc.)
- **refactor**: Code changes that neither fix bugs nor add features
- **perf**: Performance improvements
- **test**: Adding or correcting tests
- **chore**: Changes to build process or auxiliary tools
- **ci**: Changes to CI/CD configuration

### Breaking Changes

To trigger a major version bump, include `BREAKING CHANGE:` in the commit footer:

```
feat: change API response format

BREAKING CHANGE: The API now returns data in a different structure.
Users will need to update their integrations.
```

### Examples

```bash
# New feature
git commit -m "feat: add user authentication"

# Bug fix
git commit -m "fix: resolve database connection timeout"

# Documentation
git commit -m "docs: update deployment instructions"

# Breaking change
git commit -m "feat: redesign polls model

BREAKING CHANGE: Poll model now requires category field"
```

## Code Style

### Python

- Follow [PEP 8](https://pep8.org/) style guide
- Use meaningful variable and function names
- Add docstrings to functions and classes
- Keep functions focused and concise

### Django Best Practices

- Use Django's built-in features when possible
- Follow Django's naming conventions
- Use class-based views for complex views
- Keep business logic in models and services
- Use Django forms for data validation

## Testing

### Running Tests

```bash
# Run all tests
uv run manage.py test

# Run specific test
uv run manage.py test polls.tests.TestQuestion

# Run with coverage
uv run coverage run --source='.' manage.py test
uv run coverage report
```

### Writing Tests

- Write tests for new features
- Ensure tests pass before submitting PR
- Aim for good test coverage
- Test edge cases and error conditions

## Pull Request Process

1. **Create a feature branch**
   ```bash
   git checkout -b feat/your-feature-name
   ```

2. **Make your changes**
   - Write clean, readable code
   - Follow the commit conventions
   - Add tests for new features
   - Update documentation as needed

3. **Test your changes**
   ```bash
   uv run manage.py test
   uv run manage.py runserver  # Manual testing
   ```

4. **Commit your changes**
   ```bash
   git add .
   git commit -m "feat: add your feature description"
   ```

5. **Push to your fork**
   ```bash
   git push origin feat/your-feature-name
   ```

6. **Create a Pull Request**
   - Go to the original repository
   - Click "New Pull Request"
   - Select your branch
   - Fill out the PR template
   - Submit the PR

### Pull Request Guidelines

- **Title**: Use conventional commit format
- **Description**: Clearly describe what changes you made and why
- **Tests**: Ensure all tests pass
- **Documentation**: Update docs if needed
- **Single Purpose**: Each PR should address one concern
- **Small PRs**: Keep changes focused and reviewable

## Code Review

All submissions require review. We use GitHub pull requests for this purpose.

### Review Process

1. Automated checks run (tests, linting)
2. Maintainers review the code
3. Feedback may be provided
4. Make requested changes
5. Once approved, maintainers will merge

### Responding to Feedback

- Be open to suggestions
- Discuss technical decisions respectfully
- Make requested changes in new commits
- Update your PR branch as needed

## Development Workflow

### Branch Strategy

- `main`: Production-ready code
- `feat/*`: New features
- `fix/*`: Bug fixes
- `docs/*`: Documentation updates
- `chore/*`: Maintenance tasks

### Testing Locally with Docker

```bash
# Build the image
docker build -t django-web-app:dev .

# Run the container
docker run -p 8000:8000 django-web-app:dev

# Test the application
curl http://localhost:8000
```

### Testing Kubernetes Manifests

```bash
# Validate Kustomize configuration
kubectl kustomize kustomize/overlays/dev/

# Apply to test cluster
kubectl apply -k kustomize/overlays/dev/ --dry-run=client
```

## Reporting Issues

### Bug Reports

Include:
- Clear description of the issue
- Steps to reproduce
- Expected behavior
- Actual behavior
- Environment details (OS, Python version, etc.)
- Relevant logs or error messages

### Feature Requests

Include:
- Clear description of the feature
- Use case and motivation
- Proposed implementation (if any)
- Potential impact on existing features

## Questions?

If you have questions about contributing, please:
1. Check existing documentation
2. Search existing issues
3. Open a new issue with your question

## License

By contributing, you agree that your contributions will be licensed under the same license as the project.
