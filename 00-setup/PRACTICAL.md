# 00 — Project Setup

## What you'll build
An empty, running Django project with one app inside it — the foundation
every later lesson builds on. By the end of this lesson `python manage.py runserver`
shows Django's default welcome page in your browser.

## Concepts
- A Django **project** is the overall settings/config container.
- A Django **app** is one feature module inside a project (in this course, `students`).
- A **virtual environment** keeps this project's Python packages separate from
  everything else on your machine.

## Steps

**1. Create a folder and a virtual environment**
```bash
mkdir student_registration && cd student_registration
python -m venv venv
venv\Scripts\activate     # Linux: source venv/bin/activate 
```
Expected output:
```
# After venv creation, you should see a `venv` folder
# After activation, your terminal prompt should show (venv) at the beginning:
(venv) $
```

**2. Install Django**
```bash
pip install django
```
Expected output:
```
Collecting django
  Downloading django-5.x.x-py3-none-any.whl (8.0 MB)
  ...
Successfully installed django-5.x.x
```

**3. Create the project and the app**
```bash
django-admin startproject student_registration_system .
python manage.py startapp students
```
Expected output:
```
# Both commands should complete silently, creating these files:
student_registration/
├── student_registration_system/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── students/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
└── manage.py
```

**4. Register the app**

Open `student_registration_system/settings.py` and add `'students'` to
`INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'students',          # <-- add this line
]
```

**5. Run the server**
```bash
python manage.py runserver
```
Expected output:
```
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have unapplied migrations. Your project may not work properly until you apply
the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.

August 23, 2024 - 15:30:00
Django version 5.x, using settings 'student_registration_system.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```

## Checkpoint
Visit `http://127.0.0.1:8000/` — you should see Django's "The install worked
successfully!" page. Stop the server with `Ctrl+C` before moving on.

## Common mistakes
- Forgetting to activate the virtual environment before `pip install django` —
  Django installs globally instead, and later "why does this work on your
  machine but not mine" problems trace back to this.
- Forgetting to add `'students'` to `INSTALLED_APPS` — the app exists as a folder
  but Django doesn't know about it yet, so models and admin registration in later
  lessons silently fail.

## Exercise
Run `python manage.py startapp` a second time for a second, unused app (call it
`sandbox`), add it to `INSTALLED_APPS`, confirm the server still runs, then
delete the folder and remove it from settings again. This is good practice for
recognizing what "app not registered" errors look like later.

Expected output:
```bash
# Create sandbox app
$ python manage.py startapp sandbox

# Add 'sandbox' to INSTALLED_APPS
INSTALLED_APPS = [
    ...
    'students',
    'sandbox',
]

# Run server - should start normally
$ python manage.py runserver
Watching for file changes with StatReloader...
Starting development server at http://127.0.0.1:8000/

# Remove from INSTALLED_APPS and delete folder
INSTALLED_APPS = [
    ...
    'students',
    # 'sandbox',  # removed
]

$ rm -rf sandbox/

# Server should still run normally
```

---
⬅ [Course home](../README.md) | ➡ [Next: 01 - Models & Database](../01-models-database/PRACTICAL.md)