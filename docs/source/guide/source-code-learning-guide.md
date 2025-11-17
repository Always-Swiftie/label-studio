# Label Studio Source Code Learning Guide (5-Day Plan)

> Senior Developer's Guide: How to Systematically Learn the Label Studio Data Annotation Platform

## Table of Contents

1. [Project Overview](#project-overview)
2. [5-Day Learning Plan](#5-day-learning-plan)
3. [Core Architecture](#core-architecture)
4. [Key Modules](#key-modules)
5. [Development Environment Setup](#development-environment-setup)
6. [Code Navigation Tips](#code-navigation-tips)
7. [Hands-on Exercises](#hands-on-exercises)
8. [Learning Resources](#learning-resources)

---

## Project Overview

### What is Label Studio?

Label Studio is an open-source data labeling tool that supports annotation of various data types (images, text, audio, video, time series, etc.). It can be used to prepare training data or improve existing machine learning model training data.

### Technology Stack

**Backend:**
- **Python 3.10+**
- **Django 5.1+** - Web Framework
- **Django REST Framework** - API Framework
- **PostgreSQL/SQLite** - Database
- **Redis** - Cache and Task Queue
- **RQ (Redis Queue)** - Background Task Processing

**Frontend:**
- **React** - UI Framework
- **MobX State Tree** - State Management
- **TypeScript/JavaScript**
- **Nx** - Monorepo Tool

### Project Scale

- Approximately **42,000+ lines** of Python code (excluding tests and migrations)
- Main Modules: 20+ independent Django apps
- API Endpoints: 100+ RESTful interfaces

---

## 5-Day Learning Plan

### Day 1: Project Structure & Environment Setup

**Goal:** Understand overall project structure, set up development environment, and run the application

#### Morning (3-4 hours)

1. **Clone and Browse Project Structure**
   ```bash
   git clone https://github.com/HumanSignal/label-studio.git
   cd label-studio
   ```

2. **Read Core Documentation**
   - `README.md` - Project introduction
   - `CONTRIBUTING.md` - Contribution guide
   - `docs/source/guide/` - User documentation

3. **Understand Directory Structure**
   ```
   label-studio/
   ├── label_studio/          # Main application code
   │   ├── core/             # Core functionality (config, permissions, middleware)
   │   ├── projects/         # Project management
   │   ├── tasks/            # Task management
   │   ├── users/            # User management
   │   ├── organizations/    # Organization management
   │   ├── data_import/      # Data import
   │   ├── data_export/      # Data export
   │   ├── data_manager/     # Data manager
   │   ├── ml/               # Machine learning integration
   │   ├── io_storages/      # Storage backends (S3, GCS, etc.)
   │   └── ...
   ├── web/                  # Frontend code
   │   ├── apps/labelstudio/ # Main application
   │   ├── libs/editor/      # Annotation editor
   │   └── libs/datamanager/ # Data manager UI
   ├── docs/                 # Documentation
   └── deploy/               # Deployment configurations
   ```

#### Afternoon (3-4 hours)

4. **Set Up Development Environment**
   ```bash
   # Install dependencies
   pip install poetry
   poetry install
   
   # Run database migrations
   python label_studio/manage.py migrate
   
   # Collect static files
   python label_studio/manage.py collectstatic --noinput
   
   # Create superuser
   python label_studio/manage.py createsuperuser
   
   # Start development server
   python label_studio/manage.py runserver
   ```

5. **Experience Features**
   - Visit http://localhost:8080
   - Create account and login
   - Create a test project
   - Import some test data
   - Perform simple annotation operations

6. **Read Configuration Files**
   - `pyproject.toml` - Project dependencies
   - `label_studio/core/settings/label_studio.py` - Django configuration
   - `.env.development` - Environment variable examples

**Day 1 Summary:**
- Document the directory structure
- List main Django apps and their purposes
- Understand project execution flow
- Record issues encountered during environment setup

---

### Day 2: Core Modules & Data Models

**Goal:** Deep dive into core data models and module relationships

#### Morning (3-4 hours)

1. **Study Core Data Models**

   Read `models.py` files in this order:

   a. **Users and Organizations** (`organizations/models.py`, `users/models.py`)
   - `Organization` - Organization entity
   - `User` - User (extends Django User)
   - Permissions and role management

   b. **Projects and Tasks** (`projects/models.py`, `tasks/models.py`)
   - `Project` - Project (annotation config, members)
   - `Task` - Task (data to be annotated)
   - `Annotation` - Annotation results
   - `Prediction` - Prediction results (from ML models)

   **Key Relationships:**
   ```
   Organization 
     └── Project
           ├── Task
           │    ├── Annotation
           │    └── Prediction
           └── Member (User)
   ```

2. **Explore Data Using Django Shell**
   ```bash
   python label_studio/manage.py shell
   ```

   ```python
   # Execute in shell
   from projects.models import Project
   from tasks.models import Task, Annotation
   from users.models import User
   
   # View all projects
   Project.objects.all()
   
   # View tasks of a project
   project = Project.objects.first()
   project.tasks.all()
   
   # View annotations of a task
   task = Task.objects.first()
   task.annotations.all()
   ```

#### Afternoon (3-4 hours)

3. **Study Label Configuration**

   Label configuration is the core of Label Studio - it defines the annotation interface.

   - Read `label_studio/core/label_config.py`
   - Understand XML config parsing
   - View template examples in `label_studio/annotation_templates/`

4. **Data Flow Analysis**

   Trace a complete annotation workflow:
   ```
   1. User creates project → projects/api.py (ProjectViewSet)
   2. Configure labels → Save to Project.label_config
   3. Import data → data_import/api.py
   4. Create tasks → tasks/models.py (Task)
   5. User annotates → tasks/api.py (AnnotationViewSet)
   6. Save annotation → tasks/models.py (Annotation)
   7. Export results → data_export/api.py
   ```

5. **Understand Django Signals**
   - `projects/signals.py` - Hooks for project create/update
   - `tasks/models.py` - Signals for task state changes
   - Signals trigger async tasks, update statistics, etc.

**Day 2 Summary:**
- Draw ER diagram of core data models
- List key fields and relationships of each model
- Understand the role of label configuration
- Document a complete data flow

---

### Day 3: API Layer & Business Logic

**Goal:** Master REST API implementation and business logic processing

#### Morning (3-4 hours)

1. **Django REST Framework Basics**

   Label Studio extensively uses DRF. Need to understand:
   - `ViewSet` - View sets
   - `Serializer` - Serializers
   - `Permission` - Permission classes
   - `Filter` - Filters

2. **Study Core API Modules**

   Read ViewSet implementations in these files in order:

   a. **User API** (`users/api.py`)
   - User registration, login, profile management
   - Understand JWT authentication (`jwt_auth/`)

   b. **Project API** (`projects/api.py`)
   - `ProjectViewSet` - CRUD operations
   - Understand pagination, filtering, sorting

   c. **Task API** (`tasks/api.py`)
   - `TaskViewSet` - Task management
   - `AnnotationViewSet` - Annotation management
   - Batch operation implementation

3. **Permission System**
   - `core/permissions.py` - Base permission classes
   - `projects/permissions.py` - Project-level permissions
   - Uses `django-rules` for rule-based permissions

#### Afternoon (3-4 hours)

4. **Test API with Tools**

   Use Postman or curl to test APIs:

   ```bash
   # Get token
   curl -X POST http://localhost:8080/api/auth/login/ \
     -H "Content-Type: application/json" \
     -d '{"username":"admin","password":"password"}'
   
   # Create project
   curl -X POST http://localhost:8080/api/projects/ \
     -H "Authorization: Token YOUR_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{
       "title": "Test Project",
       "label_config": "<View>...</View>"
     }'
   
   # Get project list
   curl http://localhost:8080/api/projects/ \
     -H "Authorization: Token YOUR_TOKEN"
   ```

5. **Deep Dive into Serializers**
   - `projects/serializers.py`
   - `tasks/serializers.py`
   - Understand how serializers transform data
   - Learn custom fields and validation

6. **Filtering and Search**
   - `data_manager/` - Data manager filtering logic
   - Use `django-filter` for complex filtering
   - Full-text search implementation

**Day 3 Summary:**
- List main API endpoints
- Understand request-response flow
- Document permission check logic
- Try creating a custom API endpoint

---

### Day 4: Storage, Import/Export & Async Tasks

**Goal:** Understand data persistence, file handling, and background tasks

#### Morning (3-4 hours)

1. **Storage Backends**

   Label Studio supports multiple storage types:
   - Local filesystem (`io_storages/localfiles/`)
   - Amazon S3 (`io_storages/s3/`)
   - Google Cloud Storage (`io_storages/gcs/`)
   - Azure Blob (`io_storages/azure_blob/`)
   - Redis (`io_storages/redis/`)

   **Key Focus:**
   - `io_storages/models.py` - Storage configuration models
   - `io_storages/api.py` - Storage management API
   - How to implement new storage backends (review existing implementations)

2. **Data Import Flow**

   - `data_import/models.py` - FileUpload model
   - `data_import/api.py` - Import API
   - `data_import/uploader.py` - File upload handling

   **Supported Formats:**
   - JSON, CSV, TSV
   - Image files (auto-create tasks)
   - Archives (auto-extract)

3. **Data Export Flow**

   - `data_export/models.py` - Export model
   - `data_export/api.py` - Export API
   - `data_export/serializers.py` - Export formats

   **Export Formats:**
   - JSON, JSON-MIN
   - CSV, TSV
   - COCO, YOLO (via label-studio-converter)

#### Afternoon (3-4 hours)

4. **Async Task System (RQ - Redis Queue)**

   Label Studio uses RQ for time-consuming tasks:

   - `core/redis.py` - Redis connection configuration
   - Find functions using `@job` decorator
   - Understand task queues and workers

   **Common Async Tasks:**
   - Batch data import
   - Data export
   - ML model predictions
   - Project statistics calculation

   ```bash
   # Start RQ worker (in new terminal)
   python label_studio/manage.py rqworker default
   ```

5. **Caching Strategy**

   - Use Redis for caching
   - Django cache framework
   - Review `@cached_property` usage

6. **File Handling & Media Serving**

   - `core/middleware.py` - Middleware processing
   - Serving static and media files
   - Using Nginx reverse proxy (production)

**Day 4 Summary:**
- Draw data import/export flow diagram
- List all async tasks
- Understand storage backend abstract design
- Try adding a simple background task

---

### Day 5: ML Integration & Advanced Features

**Goal:** Understand ML integration, advanced features, and code quality

#### Morning (3-4 hours)

1. **Machine Learning Backend Integration**

   Label Studio can connect to ML models:

   - `ml/` - ML backend management
   - `ml_models/models.py` - MLBackend model
   - `ml/api.py` - ML backend API

   **Features:**
   - Pre-labeling
   - Active Learning
   - Model prediction display

   **Learn about ML Backend SDK:**
   ```bash
   # View examples
   # https://github.com/HumanSignal/label-studio-ml-backend
   ```

2. **Feature Flags**

   - `core/feature_flags/` - Feature flag implementation
   - Using LaunchDarkly or local config
   - How to use in code:
     ```python
     from core.feature_flags import flag_set
     
     if flag_set('ff_enable_new_feature', user=request.user):
         # New feature code
     ```

3. **Webhooks**

   - `webhooks/models.py` - Webhook configuration
   - `webhooks/api.py` - Webhook management
   - Event triggering mechanism

#### Afternoon (3-4 hours)

4. **Testing Framework**

   Understand how to test:

   ```bash
   # Run tests
   cd label_studio
   DJANGO_SETTINGS_MODULE=core.settings.label_studio pytest -vv
   ```

   - `*/tests/` - Tests for each module
   - Using pytest + pytest-django
   - Using tavern for API testing
   - Mocking and Fixtures

5. **Code Quality Tools**

   - `ruff` - Code linting and formatting
   - `blue` - Code style
   - `mypy` - Type checking
   - `pre-commit` - Pre-commit hooks

   ```bash
   # Run code checks
   ruff check label_studio/
   
   # Auto-fix
   ruff check --fix label_studio/
   ```

6. **Performance Optimization Points**

   - Database query optimization (`select_related`, `prefetch_related`)
   - Batch operations (`bulk_create`, `bulk_update`)
   - Cache usage
   - Async task separation

7. **Security Considerations**

   - CSRF protection
   - XSS prevention (using `bleach`)
   - SQL injection protection (Django ORM)
   - File upload security
   - API authentication and authorization

**Day 5 Summary:**
- Understand complete ML workflow
- List main feature flags
- Write a simple test case
- Summarize learning insights and improvement suggestions

---

## Core Architecture

### 1. Overall Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   Frontend (React + MST)                 │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │  Annotation │  │ Data Manager │  │   Project    │   │
│  │   Editor    │  │              │  │  Management  │   │
│  └─────────────┘  └──────────────┘  └──────────────┘   │
└─────────────────────────────────────────────────────────┘
                           ↓ REST API
┌─────────────────────────────────────────────────────────┐
│              Django + Django REST Framework              │
│  ┌─────────────────────────────────────────────────┐   │
│  │    API Layer (ViewSets + Serializers + Perms)   │   │
│  └─────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Business Logic (Models + Managers + Services)  │   │
│  └─────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Core Services (Permissions, Config, Signals)   │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                           ↓
┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐
│ PostgreSQL/ │  │    Redis    │  │  File Storage       │
│   SQLite    │  │ (Cache/RQ)  │  │  (S3/GCS/Local)     │
└─────────────┘  └─────────────┘  └─────────────────────┘
```

### 2. Request Processing Flow

```
1. User Request → Nginx (production)
              ↓
2. Django Middleware
   - CSRF check
   - Authentication (JWT/Session)
   - CORS handling
              ↓
3. URL Routing (urls.py)
              ↓
4. ViewSet (DRF)
   - Permission check
   - Input deserialization
              ↓
5. Business Logic
   - Model operations
   - Signal triggers
   - Async task triggers
              ↓
6. Output Serialization
              ↓
7. JSON Response
```

### 3. Data Model Hierarchy

```
Organization
  └── User - many-to-many
  └── Project
        ├── label_config (XML)
        ├── ProjectMember
        └── Task
              ├── data (JSON)
              ├── Annotation - multiple users can annotate
              │     ├── result (JSON)
              │     └── completed_by
              └── Prediction - from ML models
                    └── result (JSON)
```

---

## Key Modules

### 1. Core Module

**Location:** `label_studio/core/`

**Responsibilities:**
- Django configuration (`settings/`)
- Label config parsing (`label_config.py`)
- Permission system (`permissions.py`, `api_permissions.py`)
- Middleware (`middleware.py`)
- Common utilities (`utils/`)
- Feature Flags (`feature_flags/`)

**Key Files:**
- `label_config.py` - Parse and validate label config XML
- `permissions.py` - Rule-based permission checks using django-rules
- `utils/common.py` - Common helper functions

### 2. Projects Module

**Location:** `label_studio/projects/`

**Responsibilities:**
- Project CRUD operations
- Label config management
- Project member management
- Project statistics

**Key Models:**
- `Project` - Main project model
- `ProjectSummary` - Project statistics

**API:**
- `ProjectViewSet` - Project operations API
- Supports import/export, copy, archive

### 3. Tasks Module

**Location:** `label_studio/tasks/`

**Responsibilities:**
- Task CRUD
- Annotation management
- Prediction management
- Task assignment and queues

**Key Models:**
- `Task` - Task (data to annotate)
- `Annotation` - Annotation results
- `AnnotationDraft` - Annotation drafts
- `Prediction` - ML model predictions

**API:**
- `TaskViewSet` - Task operations
- `AnnotationViewSet` - Annotation operations
- Batch operation support

### 4. Data Manager Module

**Location:** `label_studio/data_manager/`

**Responsibilities:**
- Advanced data filtering and search
- Data visualization
- Batch operations

**Features:**
- Complex query builder
- Multi-condition filtering
- Sorting and pagination
- Column customization

### 5. IO Storages Module

**Location:** `label_studio/io_storages/`

**Supported Storage:**
- `s3/` - Amazon S3
- `gcs/` - Google Cloud Storage
- `azure_blob/` - Azure Blob Storage
- `localfiles/` - Local filesystem
- `redis/` - Redis storage

**Pattern:**
- Use abstract base class to define interface
- Each storage implements the base class
- Support sync and async operations

---

## Development Environment Setup

### Complete Development Environment

```bash
# 1. Clone repository
git clone https://github.com/HumanSignal/label-studio.git
cd label-studio

# 2. Install Python dependencies
pip install poetry
poetry install --with test,build

# 3. Configure environment variables
cp .env.development .env
# Edit .env to set database and other configs

# 4. Run migrations
poetry run python label_studio/manage.py migrate

# 5. Collect static files
poetry run python label_studio/manage.py collectstatic --noinput

# 6. Create superuser
poetry run python label_studio/manage.py createsuperuser

# 7. Start development server
poetry run python label_studio/manage.py runserver

# 8. (Optional) Start RQ worker
poetry run python label_studio/manage.py rqworker default

# 9. (Optional) Frontend development
cd web
yarn install
yarn start
```

### Using Docker for Development

```bash
# Use Docker Compose
docker-compose up

# Or use development Dockerfile
docker build -f Dockerfile.development -t label-studio:dev .
docker run -it -p 8080:8080 -v $(pwd):/label-studio label-studio:dev
```

### Common Management Commands

```bash
# Create migrations
python label_studio/manage.py makemigrations

# Apply migrations
python label_studio/manage.py migrate

# Django Shell
python label_studio/manage.py shell

# Create test data
python label_studio/manage.py init_test_data

# Run tests
cd label_studio
pytest -vv

# Code checks
ruff check label_studio/

# Format code
ruff format label_studio/
```

---

## Code Navigation Tips

### 1. Using IDE Tools

**Recommended IDEs:**
- **PyCharm Professional** - Best Django support
- **VS Code** - With Python and Django extensions
- **Vim/Neovim** - With LSP and ctags

**Essential Plugins/Extensions:**
- Python/Django syntax highlighting
- Code navigation (Go to Definition)
- Auto-completion
- Debugger
- Git integration

### 2. Code Finding Strategies

**Finding Feature Implementations:**

```bash
# Search keywords
grep -r "function_name" label_studio/

# Search API endpoints
grep -r "api/projects" label_studio/

# Search model definitions
find label_studio -name "models.py" | xargs grep "class Project"

# Search URL configs
grep -r "ProjectViewSet" label_studio/*/urls.py
```

**Understanding Data Flow:**

1. Start from URL → `urls.py`
2. Find ViewSet → `api.py`
3. View serializer → `serializers.py`
4. Understand model → `models.py`
5. Check permissions → `permissions.py`

**Debugging Tips:**

```python
# Add breakpoint in code
import pdb; pdb.set_trace()

# Or use IPython
import IPython; IPython.embed()

# Print debug info
import logging
logger = logging.getLogger(__name__)
logger.debug(f"Variable value: {var}")
```

---

## Hands-on Exercises

### Exercise 1: Create Custom API Endpoint

**Goal:** Create an API endpoint that returns project statistics

```python
# Add to projects/api.py
from rest_framework.decorators import action
from rest_framework.response import Response

class ProjectViewSet(viewsets.ModelViewSet):
    # ... existing code ...
    
    @action(detail=True, methods=['get'])
    def statistics(self, request, pk=None):
        """Get detailed project statistics"""
        project = self.get_object()
        
        stats = {
            'total_tasks': project.tasks.count(),
            'completed_tasks': project.tasks.filter(
                annotations__isnull=False
            ).distinct().count(),
            'total_annotations': project.annotations.count(),
            'unique_annotators': project.annotations.values(
                'completed_by'
            ).distinct().count(),
        }
        
        return Response(stats)
```

**Test:**
```bash
curl http://localhost:8080/api/projects/1/statistics/ \
  -H "Authorization: Token YOUR_TOKEN"
```

### Exercise 2: Add Custom Export Format

**Goal:** Create a custom JSON export format

```python
# Add to data_export/serializers.py
from data_export.serializers import ExportDataSerializer

class CustomJSONExportSerializer(ExportDataSerializer):
    def serialize_task(self, task):
        """Custom task serialization"""
        return {
            'id': task.id,
            'data': task.data,
            'annotations': [
                {
                    'result': ann.result,
                    'user': ann.completed_by.email,
                    'created_at': ann.created_at.isoformat(),
                }
                for ann in task.annotations.all()
            ]
        }
```

### Exercise 3: Create Custom Management Command

**Goal:** Create a command to batch import tasks

```python
# projects/management/commands/import_tasks.py
from django.core.management.base import BaseCommand
from projects.models import Project
from tasks.models import Task
import json

class Command(BaseCommand):
    help = 'Batch import tasks from JSON file'
    
    def add_arguments(self, parser):
        parser.add_argument('project_id', type=int)
        parser.add_argument('json_file', type=str)
    
    def handle(self, *args, **options):
        project = Project.objects.get(id=options['project_id'])
        
        with open(options['json_file'], 'r') as f:
            tasks_data = json.load(f)
        
        tasks = [
            Task(project=project, data=task_data)
            for task_data in tasks_data
        ]
        
        Task.objects.bulk_create(tasks)
        
        self.stdout.write(
            self.style.SUCCESS(
                f'Successfully imported {len(tasks)} tasks'
            )
        )
```

**Usage:**
```bash
python label_studio/manage.py import_tasks 1 tasks.json
```

---

## Learning Resources

### Official Resources

1. **Label Studio Documentation**
   - https://labelstud.io/guide/
   - Complete user and developer documentation

2. **API Documentation**
   - https://labelstud.io/api/
   - REST API reference

3. **GitHub Repository**
   - https://github.com/HumanSignal/label-studio
   - Source code, Issues, Discussions

4. **Label Studio SDK**
   - https://github.com/HumanSignal/label-studio-sdk
   - Python SDK documentation and examples

5. **ML Backend**
   - https://github.com/HumanSignal/label-studio-ml-backend
   - Machine learning integration examples

### Community Resources

1. **Slack Community**
   - https://slack.labelstud.io/
   - Active developer community

2. **YouTube Tutorials**
   - Label Studio official channel
   - Video tutorials and demos

3. **Blog Posts**
   - Label Studio official blog
   - Use cases and best practices

### Related Technical Documentation

1. **Django Documentation**
   - https://docs.djangoproject.com/
   - Django core concepts

2. **Django REST Framework**
   - https://www.django-rest-framework.org/
   - API development guide

3. **React Documentation**
   - https://react.dev/
   - Frontend development (if modifying UI)

4. **PostgreSQL Documentation**
   - https://www.postgresql.org/docs/
   - Database optimization

---

## Summary & Next Steps

### 5-Day Learning Summary

Through these 5 days of learning, you should have:

✅ Understood Label Studio's overall architecture
✅ Familiarized with core data models and relationships
✅ Mastered API implementation approaches
✅ Understood data import/export flows
✅ Comprehended ML integration mechanisms
✅ Able to read and understand most of the code
✅ Know how to debug and test
✅ Understand code contribution process

### Continuous Learning Recommendations

1. **Deep Dive into Specific Modules**
   - Focus on modules relevant to company needs
   - Examples: storage backends, ML integration, permission systems

2. **Participate in Open Source Contributions**
   - Fix small bugs
   - Improve documentation
   - Add new features

3. **Follow Project Updates**
   - Subscribe to GitHub repository
   - Watch new releases
   - Read Release Notes

4. **Build Your Own Features**
   - Customize features based on Label Studio
   - Develop plugins or extensions
   - Share experiences and code

---

**Good luck with your Label Studio journey! 🚀**

**For questions, visit the Slack community or GitHub discussions.**
