# 06 — Full CRUD in the Browser

## What you'll build
A register form and a delete link, so Create, Read, and Delete all work end
to end through the browser — the same operations from Lesson 3's shell
session, now triggered by real user actions instead of typed commands.

## Concepts
- HTML `<form method="POST">` sends data to a view; `request.POST['field']`
  reads it out on the Django side.
- `{% csrf_token %}` is required on every POST form — it protects against
  cross-site request forgery and Django will reject the form without it.
- `redirect()` sends the browser to a different URL after a successful action,
  which avoids duplicate submissions if the user refreshes the page.

## Steps

**1. Build the registration form**

Create `students/templates/register.html`:
```html
{% extends 'base.html' %}
{% block title %}Register a Student{% endblock %}

{% block content %}
<h2>Register a New Student</h2>
<form method="POST">
  {% csrf_token %}
  <input type="text" name="first_name" placeholder="First Name" required><br>
  <input type="text" name="last_name" placeholder="Last Name" required><br>
  <input type="email" name="email" placeholder="Email" required><br>
  <input type="number" name="grade" placeholder="Grade" required><br>
  <input type="date" name="dob" required><br>
  <button type="submit">Register</button>
</form>
{% endblock %}
```

**2. Write the register view**

Add to `students/views.py`:
```python
from django.shortcuts import render, redirect
from django.contrib import messages

def register_student(request):
    if request.method == 'POST':
        try:
            student = Student(
                first_name=request.POST['first_name'],
                last_name=request.POST['last_name'],
                email=request.POST['email'],
                grade=request.POST['grade'],
                date_of_birth=request.POST['dob'],
            )
            student.save()
            messages.success(request, f"Student {student.full_name()} added!")
            return redirect('student_list')
        except Exception as e:
            messages.error(request, f"Error: {e}")
    return render(request, 'register.html')
```

**3. Add a delete view**

No new template needed — it just deletes and redirects back:
```python
def delete_student(request, student_id):
    student = Student.objects.get(id=student_id)
    student.delete()
    messages.success(request, "Student deleted.")
    return redirect('student_list')
```

**4. Add a delete link to the list page**

In `student_list.html`, add a column:
```html
<tr><th>Name</th><th>Grade</th><th>Email</th><th>Actions</th></tr>
{% for student in students %}
<tr>
  <td>{{ student.full_name }}</td>
  <td>{{ student.grade }}</td>
  <td>{{ student.email }}</td>
  <td><a href="{% url 'delete_student' student.id %}">Delete</a></td>
</tr>
```

**5. Wire up every URL**

Replace `students/urls.py` with the complete set:
```python
# students/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('hello/', views.hello, name='hello'),
    path('', views.student_list, name='student_list'),
    path('register/', views.register_student, name='register_student'),
    path('delete/<int:student_id>/', views.delete_student, name='delete_student'),
]
```

## Checkpoint — full demo flow
```bash
python manage.py runserver
```
1. `http://127.0.0.1:8000/` → shows the student list
2. Click **Register a Student** → fill the form → Submit
3. Redirects back to the list → the new student is there (Create + Read)
4. Click **Delete** next to a row → it disappears (Delete)
5. Open `/admin/` in a second tab → same data, same table — the admin, the
   shell, and this web page are three different doors into the exact same
   database.

## Common mistakes
- Missing `{% csrf_token %}` — the form submission fails with a 403 Forbidden
  error until it's added.
- `request.POST['field']` raises an error if a field is missing or misnamed —
  double-check the `name=` attribute in the HTML matches the key used in the
  view exactly.
- Deleting via a GET link like this is fine for a course demo, but in a real
  app deletes should require a POST/confirmation step to avoid accidental
  (or malicious) deletes from a bare link.

## Exercise — Update (stretch goal)
Build `edit.html` and an `edit_student(request, student_id)` view using the
same pattern as `register_student`, except the form should load with the
existing student's data already filled in. This completes the CRUD set.

---
⬅ [Previous: 05 - Templates](../05-templates/PRACTICAL.md) | ⬆ [Course home](../README.md)