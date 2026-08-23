# 05 — Templates

## What you'll build
A real HTML page — a shared base layout, plus a page that lists all students
by looping over the database with a template tag.

## Concepts
- A **template** is an HTML file with special tags Django fills in with data.
- `{% block %}` in a base template marks a slot that child templates can
  override — change the shared layout once, every page updates.
- `{% for %}` loops over a queryset the view passed in; `{{ variable }}` prints
  a value.

## Steps

**1. Create the templates folder**
```bash
mkdir students/templates
```
Django looks in each app's `templates/` folder automatically.

**2. Build the shared layout**

Create `students/templates/base.html`:
```html
<!DOCTYPE html>
<html>
<head>
  <title>{% block title %}Student Registration{% endblock %}</title>
</head>
<body>
  <header>
    <h1>Student Registration System</h1>
    <nav>
      <a href="{% url 'student_list' %}">All Students</a> |
      <a href="{% url 'register_student' %}">Register a Student</a>
    </nav>
  </header>
  <main>
    {% block content %}{% endblock %}
  </main>
</body>
</html>
```

**3. Build the student list page**

Create `students/templates/student_list.html`:
```html
{% extends 'base.html' %}
{% block title %}All Students{% endblock %}

{% block content %}
<h2>All Students</h2>
<table border="1" cellpadding="6">
  <tr><th>Name</th><th>Grade</th><th>Email</th></tr>
  {% for student in students %}
  <tr>
    <td>{{ student.full_name }}</td>
    <td>{{ student.grade }}</td>
    <td>{{ student.email }}</td>
  </tr>
  {% empty %}
  <tr><td colspan="3">No students registered.</td></tr>
  {% endfor %}
</table>
{% endblock %}
```

**4. Write the matching view**

Add this to `students/views.py` (keep `hello`, don't delete it):
```python
from django.shortcuts import render
from .models import Student

def student_list(request):
    students = Student.objects.all().order_by('-registration_date')
    return render(request, 'student_list.html', {'students': students})
```

**5. Wire up the URL**

Add to `students/urls.py`:
```python
path('', views.student_list, name='student_list'),
```

## Checkpoint
Visit `http://127.0.0.1:8000/` — you should see a real HTML table listing
every student currently in the database, styled by nothing but the browser
default (that's fine, this lesson is about data flow, not design).

## Common mistakes
- `{% url 'register_student' %}` in `base.html` will error with
  `NoReverseMatch` until that URL name exists — this page will briefly break
  when you add the nav link, and get fixed again in the next lesson once
  `register_student` is wired up. Expected, not a bug.
- Forgetting `{% extends 'base.html' %}` as the very first line — without it,
  none of the blocks connect and you just get a blank page.
- A queryset is **lazy** — `Student.objects.all()` doesn't hit the database
  until the template actually loops over it with `{% for %}`.

## Exercise
Add a column to the table for `registration_date`, formatted with Django's
`date` filter: `{{ student.registration_date|date:"M d, Y" }}`.

---
⬅ [Previous: 04 - Views & URLs](../04-views-urls/README.md) | ➡ [Next: 06 - CRUD in the Browser](../06-crud-in-browser/README.md)