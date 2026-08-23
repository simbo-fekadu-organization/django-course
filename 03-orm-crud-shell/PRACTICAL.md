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
Expected output:
```
Python 3.10.x (default, ...)
Type 'copyright', 'credits' or 'license' for more information
IPython 8.x.x -- An enhanced Interactive Python. Type '?' for help.

In [1]:
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
Expected output:
```python
# After each create, you should see:
In [2]: s.save()

In [3]: 

# Or for create():
In [2]: Student.objects.create(...)
Out[2]: <Student: Almaz Tadesse>
```

**3. Read**
```python
Student.objects.all()                          # every student
Student.objects.get(id=1)                       # exactly one
Student.objects.filter(grade=9)                  # matching subset
Student.objects.all().order_by('last_name')      # sorted
```
Expected output:
```python
In [3]: Student.objects.all()
Out[3]: <QuerySet [<Student: Almaz Tadesse>, <Student: Bereket Alemayehu>]>

In [4]: Student.objects.get(id=1)
Out[4]: <Student: Almaz Tadesse>

In [5]: Student.objects.filter(grade=9)
Out[5]: <QuerySet [<Student: Almaz Tadesse>]>

In [6]: Student.objects.all().order_by('last_name')
Out[6]: <QuerySet [<Student: Almaz Tadesse>, <Student: Bereket Alemayehu>]>
```

**4. Update**
```python
student = Student.objects.get(id=1)
student.grade = 10
student.save()

Student.objects.filter(grade=9).update(grade=10)  # bulk update
```
Expected output:
```python
In [7]: student = Student.objects.get(id=1)

In [8]: student.grade = 10

In [9]: student.save()

In [10]: Student.objects.filter(grade=9).update(grade=10)
Out[10]: 1  # Number of rows updated

# Verify the update:
In [11]: Student.objects.get(id=1).grade
Out[11]: 10
```

**5. Delete**
```python
student = Student.objects.get(id=2)
student.delete()

Student.objects.filter(grade=9).delete()          # ⚠ bulk delete — filter carefully
```
Expected output:
```python
In [12]: student = Student.objects.get(id=2)

In [13]: student.delete()
Out[13]: (1, {'students.Student': 1})  # (number of objects deleted, dict with counts per model)

In [14]: Student.objects.filter(grade=9).delete()
Out[14]: (1, {'students.Student': 1})

# Verify deletion:
In [15]: Student.objects.count()
Out[15]: 0  # Or fewer than before
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

Example solution and expected output:
```python
# Step 1: Print students below grade 5
low_grade_students = Student.objects.filter(grade__lt=5)
for s in low_grade_students:
    print(s.full_name())

# Expected output:
# Bereket Alemayehu
# Chala Girma

# Step 2: Update them all to grade 5
result = Student.objects.filter(grade__lt=5).update(grade=5)
print(f"Updated {result} students")

# Expected output:
# Updated 2 students

# Verify:
Student.objects.filter(grade=5).count()
# Expected output: 2 (or more, if you had more students below grade 5)
```

---
⬅ [Previous: 02 - Admin Panel](../02-admin-panel/PRACTICAL.md) | ➡ [Next: 04 - Views & URLs](../04-views-urls/PRACTICAL.md)