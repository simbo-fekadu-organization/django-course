# 04 — Views & URLs

## What you'll build
Your first page served by Django in a browser — plain text at first, wired up
through the URL system that every later page will reuse.

## Concepts
- A **view** is a Python function that takes a `request` and must return a
  `response`.
- A **URL pattern** maps a web address to a specific view function.
- Django checks the project's `urls.py` first, which can `include()` an app's
  own `urls.py` — this keeps each app's routes self-contained.

## Steps

**1. Write a view**

Open `students/views.py`:
```python
# students/views.py
from django.http import HttpResponse

def hello(request):
    return HttpResponse("Hello, World!")
```

**2. Create the app's URL file**

Create `students/urls.py`:
```python
# students/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('hello/', views.hello, name='hello'),
]
```

**3. Include it in the project**

Edit `student_registration_system/urls.py`:
```python
# student_registration_system/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('students.urls')),
]
```

**4. Run it**
```bash
python manage.py runserver
```

## Checkpoint
Visit `http://127.0.0.1:8000/hello/` — the browser should show the plain text
"Hello, World!" (not a template, not HTML — just text, on purpose, so the
view/URL connection is easy to see clearly).

## Common mistakes
- Forgetting `name='hello'` in the `path()` call — later lessons use
  `{% url 'hello' %}` in templates, which needs that name to exist.
- Forgetting to `include('students.urls')` in the project's `urls.py` — the
  app's URL file exists but Django never looks at it, so every app route
  404s.

## Exercise
Add a second view, `about`, that returns your name and today's date as plain
text, and wire it up at `/about/` the same way. This is just reps on the
view → URL → browser loop before templates get involved.

---
⬅ [Previous: 03 - ORM & CRUD in the Shell](../03-orm-crud-shell/README.md) | ➡ [Next: 05 - Templates](../05-templates/README.md)