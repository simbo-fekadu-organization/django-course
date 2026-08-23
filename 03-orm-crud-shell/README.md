# 03 — CRUD from the Django Shell (ORM)

## What you'll build
Nothing visual this time — you'll do everything the admin does, but in Python,
using the Django ORM directly. This is the foundation for the views you'll
write in later lessons.

## Concepts
- The **ORM** (Object-Relational Mapper) lets you query and change the database
  using Python objects and method calls instead of raw SQL.
- CRUD = **C**reate, **R**ead, **U**pdate, **D**elete — the four things every
  data-backed feature eventually needs.

## Steps

**1. Open the shell**
```bash
python manage.py shell
```

**2. Create**
```python
from students.models import Student

# Method 1: build then save
s = Student(first_name="Almaz", last_name="Tadesse",
            email="almaz@school.com", grade=9,
            date_of_birth="2010-05-15")
s.save()

# Method 2: create in one call
Student.objects.create(first_name="Bereket", last_name="Alemayehu",
                        email="bereket@school.com", grade=7,
                        date_of_birth="2012-03-02")
```

**3. Read**
```python
Student.objects.all()                          # every student
Student.objects.get(id=1)                       # exactly one
Student.objects.filter(grade=9)                  # matching subset
Student.objects.all().order_by('last_name')      # sorted
```

**4. Update**
```python
student = Student.objects.get(id=1)
student.grade = 10
student.save()

Student.objects.filter(grade=9).update(grade=10)  # bulk update
```

**5. Delete**
```python
student = Student.objects.get(id=2)
student.delete()

Student.objects.filter(grade=9).delete()          # ⚠ bulk delete — filter carefully
```

Exit with `exit()` when done.

**6. Wrap it in reusable functions**

These aren't run in the shell — put them somewhere like `students/utils.py` as
a preview of what a view function will look like in Lesson 4:

```python
def add_student(first_name, last_name, email, grade, dob):
    try:
        student = Student(
            first_name=first_name, last_name=last_name,
            email=email, grade=grade, date_of_birth=dob
        )
        student.save()
        print(f"Added: {student.full_name()}")
    except Exception as e:
        print(f"Error: {e}")

def list_students():
    students = Student.objects.all()
    if students:
        for student in students:
            print(f"{student.full_name()} - Grade {student.grade}")
    else:
        print("No students found")

def find_student(email):
    try:
        return Student.objects.get(email=email)
    except Student.DoesNotExist:
        return None
```

## Checkpoint
From the shell, `Student.objects.all()` should print the students you created,
and `Student.objects.filter(grade=10)` should return only the ones you updated.

## Common mistakes
- Using `.get()` when more than one row could match — it raises an error if
  it finds 0 or more than 1 result. Use `.filter()` for anything that might
  return several rows, `.get()` only when you're certain there's exactly one.
- Running a `.filter().delete()` that's broader than intended — always run the
  matching `.filter()` alone first and check what it returns before calling
  `.delete()` on it.

## Exercise
Write a shell snippet that finds every student below grade 5, prints their full
names, then bulk-updates them all to grade 5. Do it in two steps — print first,
update second — so you can see what you're about to change before you change it.

---
⬅ [Previous: 02 - Admin Panel](../02-admin-panel/README.md) | ➡ [Next: 04 - Views & URLs](../04-views-urls/README.md)