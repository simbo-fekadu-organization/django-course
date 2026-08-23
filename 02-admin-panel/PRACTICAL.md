# 02 — Django Admin Panel

## What you'll build
A working admin dashboard where you can add, search, filter, and delete
students — with zero HTML written.

## Concepts
- A model is invisible in `/admin/` until it's **registered** in `admin.py`.
- The admin site is generated entirely from your model + a small config class —
  it's a management tool, not something you'd show end users.

## Steps

**1. Register the model**

Open `students/admin.py`:

```python
# students/admin.py
from django.contrib import admin
from .models import Student

@admin.register(Student)
class StudentAdmin(admin.ModelAdmin):
    list_display = ['first_name', 'last_name', 'email', 'grade']
    search_fields = ['first_name', 'last_name', 'email']
    list_filter = ['grade']
```

- `list_display` = which columns show in the list view.
- `search_fields` = which fields the search box checks.
- `list_filter` = adds a sidebar filter (try it with grade).

**2. Create a superuser**
```bash
python manage.py createsuperuser
```
Follow the prompts for username, email, and password. Write this down — it's
your login for the rest of the course.

**3. Explore the dashboard**
```bash
python manage.py runserver
```
- Go to `http://127.0.0.1:8000/admin/` and log in
- Click **Students → Add student**, fill the form, Save
- Add 2–3 more students with different grades
- Try the search box and the grade filter

## Checkpoint
You should be able to add, search, filter, and delete a student entirely
through the browser, with no code beyond `admin.py`.

## Common mistakes
- Forgetting to register the model — it exists in the database but never
  appears under `/admin/`.
- Losing the superuser password — there's no "forgot password" flow by default;
  just run `python manage.py createsuperuser` again with a different username,
  or `python manage.py changepassword <username>`.

## Exercise
Add a `search_fields` entry for a field that doesn't exist yet (like `sex` from
the previous lesson's exercise) and see the error Django gives you. Then fix it
by adding the field for real.

---
⬅ [Previous: 01 - Models & Database](../01-models-database/PRACTICAL.md) | ➡ [Next: 03 - ORM & CRUD in the Shell](../03-orm-crud-shell/PRACTICAL.md)