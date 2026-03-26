# Security Policy

## Supported Versions

We actively maintain and provide security updates for the following versions:

| Version | Supported          |
| ------- | ------------------ |
| 1.x.x   | :white_check_mark: |
| < 1.0   | :x:                |

## Reporting a Vulnerability

We take the security of this project seriously. If you discover a security vulnerability, please follow these steps:

### How to Report

1. **Do NOT** create a public GitHub issue for security vulnerabilities
2. Email security concerns to the maintainers (replace with actual contact)
3. Include detailed information:
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested fix (if any)

### What to Expect

- **Acknowledgment**: We will acknowledge receipt of your report within 48 hours
- **Initial Assessment**: We will provide an initial assessment within 5 business days
- **Updates**: You will receive regular updates on the progress
- **Resolution**: We aim to resolve critical vulnerabilities within 30 days
- **Credit**: With your permission, we will credit you in the security advisory

## Security Measures

### Container Image Security

#### Image Signing

All container images are signed using [Sigstore Cosign](https://www.sigstore.dev/) with keyless OIDC signing:

```bash
# Verify image signature
cosign verify \
  --certificate-identity-regexp="https://github.com/.*/.github/workflows/.*" \
  --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
  ghcr.io/hnijbacker/django-web-app:v1.0.0
```

**Benefits:**
- Ensures images haven't been tampered with
- Verifies images were built by the official CI/CD pipeline
- No long-lived signing keys to manage
- Transparent verification process

#### Base Image

We use official Python base images:
- `python:3.12-slim-bookworm` from Docker Hub
- Regular updates applied via CI/CD
- Minimal attack surface with slim variant

### Dependency Management

#### uv Package Manager

We use [uv](https://github.com/astral-sh/uv) for dependency management:

- **Lockfile**: `uv.lock` ensures reproducible builds
- **Fast resolution**: Quick security updates
- **Automatic updates**: Dependabot monitors dependencies

#### Dependency Updates

Dependencies are regularly updated through:
1. Automated Dependabot PRs
2. Manual security reviews
3. CI/CD pipeline validation

### Application Security

#### Django Security Features

Enabled security features:
- CSRF protection
- XSS protection
- SQL injection prevention
- Clickjacking protection
- SSL redirect (production)
- Secure cookies (production)

#### Environment Variables

Sensitive data is managed via environment variables:
- `SECRET_KEY`: Django secret key (required)
- `DATABASE_URL`: Database credentials
- Never committed to repository
- Stored in Kubernetes secrets

#### Security Headers

Recommended headers in production:

```python
# settings.py (production)
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
X_FRAME_OPTIONS = 'DENY'
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
```

### Kubernetes Security

#### Pod Security

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000
  capabilities:
    drop:
      - ALL
  readOnlyRootFilesystem: true
```

#### Network Policies

Implement network policies to restrict traffic:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: django-app-policy
spec:
  podSelector:
    matchLabels:
      app: django-app
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: ingress-controller
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: postgres
```

#### Secrets Management

- Use Kubernetes secrets for sensitive data
- Consider external secret management (Vault, AWS Secrets Manager)
- Rotate secrets regularly
- Use RBAC to limit secret access

### CI/CD Security

#### GitHub Actions Security

- Uses OIDC for authentication (no long-lived tokens)
- Minimal required permissions
- Secrets stored in GitHub Secrets
- Workflow runs are isolated

#### Permissions

```yaml
permissions:
  contents: write      # For git operations
  packages: write      # For pushing to ghcr.io
  id-token: write      # For OIDC signing
```

### Access Control

#### Repository Access

- Protected main branch
- Required PR reviews
- Status checks must pass
- Limited write access

#### Container Registry

- Images hosted on GitHub Container Registry
- Access controlled by GitHub permissions
- Public read access
- Write access limited to CI/CD

## Security Best Practices

### Development

1. **Never commit secrets**
   - Use `.gitignore` for sensitive files
   - Use environment variables
   - Scan commits with git-secrets

2. **Dependency scanning**
   ```bash
   # Check for known vulnerabilities
   uv pip list --format=json | python -m json.tool
   ```

3. **Code review**
   - Review all code changes
   - Look for security issues
   - Use automated security scanning

### Deployment

1. **Always verify image signatures**
   ```bash
   cosign verify <image>
   ```

2. **Use specific version tags**
   - Never use `latest` in production
   - Use semantic versions (e.g., `v1.0.0`)

3. **Implement least privilege**
   - Minimal container permissions
   - Restricted network access
   - Limited resource access

4. **Regular updates**
   - Apply security patches promptly
   - Update base images regularly
   - Monitor security advisories

### Production

1. **Enable all security features**
   ```python
   DEBUG = False
   SECURE_SSL_REDIRECT = True
   SESSION_COOKIE_SECURE = True
   CSRF_COOKIE_SECURE = True
   ```

2. **Use strong secrets**
   ```python
   from django.core.management.utils import get_random_secret_key
   SECRET_KEY = get_random_secret_key()
   ```

3. **Configure allowed hosts**
   ```python
   ALLOWED_HOSTS = ['yourdomain.com', 'www.yourdomain.com']
   ```

4. **Use HTTPS**
   - Configure TLS certificates
   - Enable HSTS
   - Use secure cookies

5. **Database security**
   - Use strong passwords
   - Limit network access
   - Enable encryption at rest
   - Regular backups

## Security Checklist

Before deploying to production:

- [ ] `DEBUG = False` in production
- [ ] Strong `SECRET_KEY` configured
- [ ] `ALLOWED_HOSTS` properly set
- [ ] HTTPS/TLS configured
- [ ] Security headers enabled
- [ ] Database credentials secured
- [ ] Container images signed
- [ ] Image signatures verified
- [ ] Dependencies up to date
- [ ] Security scanning enabled
- [ ] Logs monitoring configured
- [ ] Backups configured
- [ ] Secrets properly managed
- [ ] Network policies configured
- [ ] Pod security contexts set

## Vulnerability Disclosure

### Timeline

- **Day 0**: Vulnerability reported
- **Day 2**: Acknowledgment sent
- **Day 5**: Initial assessment provided
- **Day 30**: Target resolution for critical issues
- **Day 45**: Public disclosure (after fix is released)

### Severity Levels

- **Critical**: Immediate action required (24-48 hours)
- **High**: Fix within 7 days
- **Medium**: Fix within 30 days
- **Low**: Fix in next release cycle

## Security Resources

### Tools

- [Cosign](https://github.com/sigstore/cosign) - Container signing
- [Trivy](https://github.com/aquasecurity/trivy) - Vulnerability scanning
- [git-secrets](https://github.com/awslabs/git-secrets) - Prevent secrets in git
- [Safety](https://github.com/pyupio/safety) - Python dependency checker

### References

- [Django Security](https://docs.djangoproject.com/en/stable/topics/security/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker)
- [Kubernetes Security](https://kubernetes.io/docs/concepts/security/)
- [Sigstore Documentation](https://docs.sigstore.dev/)

## Updates to This Policy

This security policy is reviewed and updated regularly. Last update: 2026-03-26

## Acknowledgments

We thank all security researchers who responsibly disclose vulnerabilities to help keep this project secure.
