# Theory: Django Project Structure & Setup

## Understanding Django's Architecture

Django follows the **Model-View-Template (MVT)** pattern, which is a variation of the classic MVC (Model-View-Controller) pattern. In Django's case:

- **Model**: Defines your data structure (the `Student` class in our Student Registration System)
- **View**: Handles the business logic and returns a response (Python functions in `views.py`)
- **Template**: Handles the presentation layer (HTML files)

Our Student Registration System will have all three components working together.

## Project vs App: The Core Distinction

### Django Project
A **project** is the overall container for your application. It includes:
- Configuration/settings (`settings.py`)
- URL routing (`urls.py`)
- WSGI application entry point (`wsgi.py`)

In our course, `student_registration_system` is the project. Think of it as the entire school management system.

**Key file: `settings.py`**
This is where you configure:
- `INSTALLED_APPS`: List of all apps in your project
- `DATABASES`: Database configuration
- `SECRET_KEY`: Security key for your application
- `ROOT_URLCONF`: Where to find the main URL patterns

### Django App
An **app** is a self-contained module that provides a specific set of features. In Django philosophy: "An app should do one thing and do it well."

In our course, `students` is the app. It handles everything related to student management.

**Why separate projects and apps?**
- **Reusability**: Apps can be reused across multiple projects (e.g., a `users` app could be used in many different systems)
- **Modularity**: Different teams can work on different apps independently
- **Organization**: Keeps related functionality together

**Example from our Student Registration System:**
- The **project** (`student_registration_system`) could eventually have multiple apps:
  - `students` - for student management (what we're building)
  - `courses` - for course management
  - `enrollments` - for tracking which students are in which courses
  - `attendance` - for tracking student attendance

## Virtual Environments

### Why Use a Virtual Environment?

Python projects can have conflicting package requirements. A virtual environment creates an isolated space where you can install specific versions of packages without affecting other projects on your machine.

**Without a virtual environment:**
```
Your System Python
├── Project A (needs Django 4.0)
├── Project B (needs Django 5.0)  ← Conflict!
└── Global packages
```

**With a virtual environment:**
```
Your System Python
├── Project A
│   └── venv (Django 4.0 installed here)
├── Project B
│   └── venv (Django 5.0 installed here)
└── Global packages (unchanged)
```

### How It Works in Our Project

When you run `python -m venv venv`, Python creates a folder called `venv` that contains:
- A copy of the Python interpreter
- A `bin/` (or `Scripts/`) directory with the `activate` script
- A `lib/` directory where all packages will be installed

Activating it (`source venv/bin/activate` or `venv\Scripts\activate`) modifies your shell's `PATH` so that `python` and `pip` commands use the versions inside the virtual environment.

**Why this matters for our Student Registration System:**
- Ensures all developers use the exact same Django version
- Prevents conflicts with other Python projects on your machine
- Makes it easy to track and reproduce the exact environment

## Django's Command-Line Interface

Django provides several essential management commands via `python manage.py`:

| Command | Purpose | Example in Our Project |
|---------|---------|------------------------|
| `startproject` | Creates a new Django project | `django-admin startproject student_registration_system .` |
| `startapp` | Creates a new app within a project | `python manage.py startapp students` |
| `runserver` | Starts a development web server | `python manage.py runserver` |
| `migrate` | Applies database migrations | `python manage.py migrate` |
| `makemigrations` | Creates migration files from model changes | `python manage.py makemigrations` |

## File Structure Explained

After setup, your Student Registration System will have this structure:

```
student_registration_system/      # Project root (folder)
├── student_registration_system/   # Project package (also folder)
│   ├── __init__.py
│   ├── settings.py          # Project settings
│   ├── urls.py             # Main URL routing
│   └── wsgi.py             # WSGI config for deployment
├── students/                      # Our app
│   ├── __init__.py
│   ├── admin.py            # Admin configuration
│   ├── apps.py             # App configuration
│   ├── models.py           # Data models (Student goes here)
│   ├── tests.py            # Tests
│   └── views.py            # View functions
├── db.sqlite3              # SQLite database file (created later)
└── manage.py               # Django management script
```

## The Request/Response Cycle (Preview)

When a user visits your Student Registration System in a browser:

```
Browser Request → Django Project (urls.py) → App URL routing → View function → Response
                                                    ↑
                                                 Models (Database)
```

This is the flow you'll be building throughout the course. Lesson 00 sets up the foundation; subsequent lessons fill in each piece.
