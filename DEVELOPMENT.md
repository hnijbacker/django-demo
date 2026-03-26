# Development Guide

This guide provides detailed information for developers working on the Django Web App.

## Table of Contents

- [Development Environment Setup](#development-environment-setup)
- [Project Architecture](#project-architecture)
- [Development Workflow](#development-workflow)
- [Testing](#testing)
- [Docker Development](#docker-development)
- [Database Management](#database-management)
- [Debugging](#debugging)
- [Common Tasks](#common-tasks)

## Development Environment Setup

### Prerequisites

- **Python 3.12+**: Required for running the application
- **uv**: Fast Python package manager
- **Git**: Version control
- **Docker**: For containerized development (optional)
- **Code Editor**: VS Code, PyCharm, or your preferred editor

### Initial Setup

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd django-demo
   ```

2. **Install uv** (if not already installed)
   ```bash
   # macOS/Linux
   curl -LsSf https://astral.sh/uv/install.sh | sh

   # Windows
   powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

   # Or via pip
   pip install uv
   ```

3. **Create and activate virtual environment**
   ```bash
   # uv handles this automatically, but you can create one explicitly
   uv venv
   source .venv/bin/activate  # Linux/macOS
   # or
   .venv\Scripts\activate  # Windows
   ```

4. **Install dependencies**
   ```bash
   uv sync
   ```

5. **Set up environment variables**
   ```bash
   cp .env.example .env  # If .env.example exists
   # Edit .env with your local settings
   ```

6. **Run database migrations**
   ```bash
   uv run manage.py migrate
   ```

7. **Create a superuser**
   ```bash
   uv run manage.py createsuperuser
   ```

8. **Start development server**
   ```bash
   uv run manage.py runserver
   ```

9. **Access the application**
   - Main app: http://localhost:8000
   - Admin interface: http://localhost:8000/admin
   - Polls app: http://localhost:8000/polls

## Project Architecture

### Directory Structure

```
django-demo/
├── django_project/          # Main Django project
│   ├── __init__.py
│   ├── settings.py          # Project settings
│   ├── urls.py              # Root URL configuration
│   ├── wsgi.py              # WSGI application
│   └── asgi.py              # ASGI application
│
├── polls/                   # Polls Django app
│   ├── migrations/          # Database migrations
│   ├── templates/           # HTML templates
│   │   └── polls/
│   │       └── index.html
│   ├── __init__.py
│   ├── admin.py             # Admin configuration
│   ├── apps.py              # App configuration
│   ├── models.py            # Data models
│   ├── tests.py             # Unit tests
│   ├── urls.py              # App URL patterns
│   └── views.py             # View functions
│
├── staticfiles/             # Collected static files (generated)
├── .github/                 # GitHub Actions workflows
│   └── workflows/
│       └── ci.yml           # CI/CD pipeline
│
├── kustomize/               # Kubernetes configurations
│   ├── base/                # Base manifests
│   └── overlays/            # Environment overlays
│
├── Dockerfile               # Container image definition
├── docker-entrypoint.sh     # Container startup script
├── manage.py                # Django CLI tool
├── pyproject.toml           # Project metadata and dependencies
├── uv.lock                  # Locked dependencies
└── README.md                # Project documentation
```

### Django Apps

#### django_project
Main project configuration including settings, URLs, and WSGI/ASGI configuration.

#### polls
Sample Django app demonstrating:
- Model creation and relationships
- Views and templates
- URL routing
- Admin interface

### Settings

Key settings in `django_project/settings.py`:

- **DEBUG**: Development/production mode
- **ALLOWED_HOSTS**: Permitted hostnames
- **DATABASES**: Database configuration
- **INSTALLED_APPS**: Enabled Django apps
- **MIDDLEWARE**: Request/response processing
- **STATIC_URL**: Static files URL prefix

## Development Workflow

### Daily Development

1. **Start development server**
   ```bash
   uv run manage.py runserver
   ```

2. **Make code changes**
   - Edit files in your code editor
   - Server auto-reloads on file changes

3. **Run tests**
   ```bash
   uv run manage.py test
   ```

4. **Commit changes**
   ```bash
   git add .
   git commit -m "feat: description of changes"
   ```

### Adding Dependencies

```bash
# Add a new dependency
uv add package-name

# Add a dev dependency
uv add --dev package-name

# Update dependencies
uv sync

# Update a specific package
uv add package-name@latest
```

### Creating a New Django App

```bash
# Create new app
uv run manage.py startapp myapp

# Add to INSTALLED_APPS in settings.py
INSTALLED_APPS = [
    # ...
    'myapp',
]

# Create migrations
uv run manage.py makemigrations myapp

# Apply migrations
uv run manage.py migrate
```

### Working with Models

1. **Define model in `models.py`**
   ```python
   from django.db import models

   class MyModel(models.Model):
       name = models.CharField(max_length=200)
       created_at = models.DateTimeField(auto_now_add=True)

       def __str__(self):
           return self.name
   ```

2. **Create migrations**
   ```bash
   uv run manage.py makemigrations
   ```

3. **Apply migrations**
   ```bash
   uv run manage.py migrate
   ```

4. **Register in admin**
   ```python
   # admin.py
   from django.contrib import admin
   from .models import MyModel

   admin.site.register(MyModel)
   ```

### Working with Views

```python
# views.py
from django.shortcuts import render, get_object_or_404
from .models import MyModel

def index(request):
    items = MyModel.objects.all()
    return render(request, 'myapp/index.html', {'items': items})

def detail(request, pk):
    item = get_object_or_404(MyModel, pk=pk)
    return render(request, 'myapp/detail.html', {'item': item})
```

### URL Configuration

```python
# urls.py
from django.urls import path
from . import views

app_name = 'myapp'
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:pk>/', views.detail, name='detail'),
]
```

## Testing

### Running Tests

```bash
# Run all tests
uv run manage.py test

# Run specific app tests
uv run manage.py test polls

# Run specific test class
uv run manage.py test polls.tests.QuestionModelTests

# Run with verbose output
uv run manage.py test --verbosity=2

# Keep test database
uv run manage.py test --keepdb
```

### Writing Tests

```python
# tests.py
from django.test import TestCase
from django.urls import reverse
from .models import Question

class QuestionModelTests(TestCase):
    def test_string_representation(self):
        question = Question(question_text="Test question")
        self.assertEqual(str(question), "Test question")

class QuestionViewTests(TestCase):
    def test_index_view(self):
        response = self.client.get(reverse('polls:index'))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "No polls are available")
```

### Test Coverage

```bash
# Install coverage
uv add --dev coverage

# Run tests with coverage
uv run coverage run --source='.' manage.py test

# Generate coverage report
uv run coverage report

# Generate HTML report
uv run coverage html
# Open htmlcov/index.html in browser
```

## Docker Development

### Building the Image

```bash
# Build image
docker build -t django-web-app:dev .

# Build with cache disabled
docker build --no-cache -t django-web-app:dev .
```

### Running Locally

```bash
# Run container
docker run -p 8000:8000 django-web-app:dev

# Run with environment variables
docker run -p 8000:8000 \
  -e DEBUG=True \
  -e SECRET_KEY=dev-secret-key \
  django-web-app:dev

# Run with volume mount for development
docker run -p 8000:8000 \
  -v $(pwd):/app \
  django-web-app:dev
```

### Docker Compose (Optional)

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DEBUG=True
      - SECRET_KEY=dev-secret-key
    volumes:
      - .:/app
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=django
      - POSTGRES_USER=django
      - POSTGRES_PASSWORD=django
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Run with Docker Compose:
```bash
docker-compose up
docker-compose down
```

## Database Management

### Django Shell

```bash
# Open Django shell
uv run manage.py shell

# Or with IPython
uv add --dev ipython
uv run manage.py shell
```

```python
# Inside shell
from polls.models import Question
from django.utils import timezone

# Create objects
q = Question(question_text="What's new?", pub_date=timezone.now())
q.save()

# Query objects
Question.objects.all()
Question.objects.filter(id=1)
Question.objects.filter(question_text__startswith='What')

# Update objects
q = Question.objects.get(id=1)
q.question_text = "Updated question"
q.save()

# Delete objects
q.delete()
```

### Database Commands

```bash
# Show migrations
uv run manage.py showmigrations

# Check for migration issues
uv run manage.py makemigrations --check --dry-run

# SQL for migration
uv run manage.py sqlmigrate polls 0001

# Flush database (delete all data)
uv run manage.py flush

# Database shell
uv run manage.py dbshell
```

### Fixtures

```bash
# Export data
uv run manage.py dumpdata polls > polls.json

# Import data
uv run manage.py loaddata polls.json

# Export specific model
uv run manage.py dumpdata polls.Question > questions.json
```

## Debugging

### Django Debug Toolbar

```bash
# Install
uv add --dev django-debug-toolbar

# Add to INSTALLED_APPS
INSTALLED_APPS = [
    # ...
    'debug_toolbar',
]

# Add to MIDDLEWARE
MIDDLEWARE = [
    # ...
    'debug_toolbar.middleware.DebugToolbarMiddleware',
]

# Configure internal IPs
INTERNAL_IPS = ['127.0.0.1']
```

### Using Python Debugger

```python
# Add breakpoint in code
def my_view(request):
    import pdb; pdb.set_trace()
    # Code execution pauses here
    return HttpResponse("Hello")
```

### Print Debugging

```python
import logging
logger = logging.getLogger(__name__)

def my_view(request):
    logger.debug("Request received")
    logger.info("Processing request")
    logger.warning("Something unexpected")
    logger.error("Error occurred")
```

## Common Tasks

### Collecting Static Files

```bash
# Collect static files for production
uv run manage.py collectstatic

# Clear existing static files first
uv run manage.py collectstatic --clear --noinput
```

### Creating Admin User

```bash
# Interactive
uv run manage.py createsuperuser

# Non-interactive
uv run manage.py createsuperuser \
  --username admin \
  --email admin@example.com \
  --noinput
```

### Changing Admin Password

```bash
uv run manage.py changepassword username
```

### Running Management Commands

```bash
# List all available commands
uv run manage.py help

# Get help for specific command
uv run manage.py help migrate
```

### Custom Management Commands

Create `polls/management/commands/mycommand.py`:

```python
from django.core.management.base import BaseCommand

class Command(BaseCommand):
    help = 'Description of command'

    def add_arguments(self, parser):
        parser.add_argument('poll_ids', nargs='+', type=int)

    def handle(self, *args, **options):
        for poll_id in options['poll_ids']:
            self.stdout.write(f'Processing poll {poll_id}')
```

Run it:
```bash
uv run manage.py mycommand 1 2 3
```

### Code Formatting and Linting

```bash
# Install development tools
uv add --dev black ruff

# Format code with Black
uv run black .

# Lint with Ruff
uv run ruff check .

# Auto-fix issues
uv run ruff check --fix .
```

### Type Checking

```bash
# Install mypy
uv add --dev django-stubs mypy

# Run type checking
uv run mypy .
```

## IDE Configuration

### VS Code

Create `.vscode/settings.json`:

```json
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  "python.linting.enabled": true,
  "python.formatting.provider": "black",
  "editor.formatOnSave": true,
  "[python]": {
    "editor.defaultFormatter": "ms-python.black-formatter"
  }
}
```

### PyCharm

1. Open Settings → Project → Python Interpreter
2. Select the virtual environment at `.venv`
3. Enable Django support in Settings → Languages & Frameworks → Django
4. Set Django project root and settings file

## Performance Optimization

### Database Query Optimization

```python
# Use select_related for ForeignKey
questions = Question.objects.select_related('category').all()

# Use prefetch_related for ManyToMany
questions = Question.objects.prefetch_related('choices').all()

# Use only() to select specific fields
questions = Question.objects.only('id', 'question_text')

# Use defer() to exclude fields
questions = Question.objects.defer('description')
```

### Caching

```python
from django.core.cache import cache

# Set cache
cache.set('my_key', 'my_value', timeout=300)

# Get cache
value = cache.get('my_key')

# Delete cache
cache.delete('my_key')
```

## Troubleshooting

### Common Issues

**Port already in use**
```bash
# Find process using port 8000
lsof -i :8000
# Kill the process
kill -9 <PID>
```

**Database locked**
```bash
# Delete SQLite database
rm db.sqlite3
# Recreate database
uv run manage.py migrate
```

**Module not found**
```bash
# Reinstall dependencies
uv sync
```

**Static files not loading**
```bash
# Collect static files
uv run manage.py collectstatic --clear
```

## Additional Resources

- [Django Documentation](https://docs.djangoproject.com/)
- [Django Best Practices](https://django-best-practices.readthedocs.io/)
- [Two Scoops of Django](https://www.feldroy.com/books/two-scoops-of-django-3-x)
- [uv Documentation](https://github.com/astral-sh/uv)
