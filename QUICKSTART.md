# Quick Start Guide

Get up and running with the Django Web App in under 5 minutes.

## Prerequisites

- Python 3.12+
- [uv](https://github.com/astral-sh/uv) package manager

## Installation

### 1. Install uv (if not already installed)

**macOS/Linux:**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Windows:**
```powershell
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### 2. Clone and Setup

```bash
# Clone the repository
git clone <repository-url>
cd django-demo

# Install dependencies
uv sync

# Run migrations
uv run manage.py migrate

# Create superuser (optional)
uv run manage.py createsuperuser

# Start the server
uv run manage.py runserver
```

### 3. Access the Application

- **Main App**: http://localhost:8000
- **Polls**: http://localhost:8000/polls/
- **Admin**: http://localhost:8000/admin/

## Docker Quick Start

```bash
# Build the image
docker build -t django-web-app .

# Run the container
docker run -p 8000:8000 django-web-app

# Access at http://localhost:8000
```

## Next Steps

- **Development**: Read [DEVELOPMENT.md](DEVELOPMENT.md) for detailed development guide
- **Deployment**: Check [DEPLOYMENT.md](DEPLOYMENT.md) for Kubernetes deployment
- **Contributing**: See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines

## Common Commands

```bash
# Run tests
uv run manage.py test

# Create migrations
uv run manage.py makemigrations

# Apply migrations
uv run manage.py migrate

# Collect static files
uv run manage.py collectstatic

# Create superuser
uv run manage.py createsuperuser

# Start development server
uv run manage.py runserver

# Open Django shell
uv run manage.py shell
```

## Troubleshooting

**Port 8000 already in use?**
```bash
# Use a different port
uv run manage.py runserver 8080
```

**Dependencies not installing?**
```bash
# Clear cache and reinstall
rm -rf .venv
uv sync
```

**Database errors?**
```bash
# Reset database
rm db.sqlite3
uv run manage.py migrate
```

## Need Help?

- 📖 Full documentation: [README.md](README.md)
- 🛠️ Development guide: [DEVELOPMENT.md](DEVELOPMENT.md)
- 🐛 Report issues: [GitHub Issues](https://github.com/your-repo/issues)
