# 01 — Models & Database

## What you'll build
A `Student` model, and a real database table generated from it.

## Concepts
- A **model** is a Python class that describes one database table — each
  attribute becomes a column.
- **Migrations** are how Django turns model changes into actual SQL. Nothing
  touches the database until you run `migrate`.
- Rule to memorize: *change a model → `makemigrations` → `migrate`. Every time.*

## Steps

**1. Write the model**

Open `students/models.py` and replace its contents:

```python
# students/models.py
from django.db import models

class Student(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    grade = models.IntegerField()
    date_of_birth = models.DateField()
    registration_date = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"{self.first_name} {self.last_name}"

    def full_name(self):
        return f"{self.first_name} {self.last_name}"
```

Notes on the fields:
- `CharField` needs `max_length`. `EmailField` validates email format;
  `unique=True` stops duplicate emails.
- `registration_date` uses `auto_now_add=True` — Django stamps it automatically
  the moment a row is created. You'll use this later to sort "newest first."
- `__str__` controls how a Student prints — in the admin, the shell, everywhere.

**2. Create the migration**
```bash
python manage.py makemigrations
```
Expected output:
```
Migrations for 'students':
  students/migrations/0001_initial.py
    - Create model Student
```

**3. Apply it**
```bash
python manage.py migrate
```
Expected output:
```
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  ...
  Applying students.0001_initial... OK
```

## Checkpoint
Confirm the real table exists:
```bash
python manage.py dbshell
.tables
.schema students_student
.quit
```
Expected output:
```
# In dbshell, running .tables should show:
android_metadata  auth_group                  auth_user_groups
auth_group_permissions  auth_user_user_permissions  django_content_type
django_migrations       django_session          students_student

# Running .schema students_student should show:
CREATE TABLE IF NOT EXISTS "students_student" (
    "id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
    "first_name" varchar(100) NOT NULL,
    "last_name" varchar(100) NOT NULL,
    "email" varchar(254) NOT NULL UNIQUE,
    "grade" integer NOT NULL,
    "date_of_birth" date NOT NULL,
    "registration_date" datetime NOT NULL
);
```
You should see a `students_student` table with a column for every field you wrote.

## Common mistakes
- Running `makemigrations` but forgetting `migrate` — the migration *file* exists
  but the database hasn't actually changed yet. Both commands are required.
- Adding a field to `models.py` later, using the app, and getting
  `table students_student has no column named X` — that means a migration was
  written but never applied (or was applied to a different database file than
  the one you're pointed at). Fix: `makemigrations` then `migrate` again.
- Never hand-delete a migration file once `migrate` has already run against a
  database you care about. If you need to undo one, roll back first with
  `python manage.py migrate students <previous_migration_name>`, *then* delete
  the file.

## Exercise
Add a `sex` field to the model (`CharField` with `choices` and a `default`),
then run `makemigrations` and `migrate` again. Watch what Django asks you when
you add a non-nullable field to a model that already has rows in its table.

Expected output when running `makemigrations`:
```
Migrations for 'students':
  students/migrations/0002_student_sex.py
    - Add field sex to student
```

Expected output when running `migrate`:
```
Operations to perform:
  Target specific migration: 0002_student_sex, from students
Running migrations:
  Applying students.0002_student_sex... OK
```

---
⬅ [Previous: 00 - Setup](../00-setup/PRACTICAL.md) | ➡ [Next: 02 - Admin Panel](../02-admin-panel/PRACTICAL.md)