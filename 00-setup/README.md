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

**2. Install Django**
```bash
pip install django
```

**3. Create the project and the app**
```bash
django-admin startproject student_registration_system .
python manage.py startapp students
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

---
⬅ [Course home](../README.md) | ➡ [Next: 01 - Models & Database](../01-models-database/README.md)