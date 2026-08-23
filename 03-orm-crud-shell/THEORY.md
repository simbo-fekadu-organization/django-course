# Theory: ORM & CRUD Operations

## What is an ORM?

**ORM** stands for **Object-Relational Mapper**. It's a technique that lets you query and manipulate databases using an object-oriented paradigm in your programming language, instead of writing raw SQL.

### The Problem ORMs Solve

Without an ORM, interacting with a database looks like this:

```sql
-- SQL (traditional approach)
INSERT INTO students_student (first_name, last_name, email, grade, date_of_birth)
VALUES ('Almaz', 'Tadesse', 'almaz@school.com', 9, '2010-05-15');

SELECT * FROM students_student WHERE grade = 9;

UPDATE students_student SET grade = 10 WHERE id = 1;

DELETE FROM students_student WHERE id = 2;
```

With Django's ORM, the same operations look like this:

```python
# Django ORM (object-oriented approach)
student = Student(
    first_name='Almaz',
    last_name='Tadesse',
    email='almaz@school.com',
    grade=9,
    date_of_birth='2010-05-15'
)
student.save()

Student.objects.filter(grade=9)

student = Student.objects.get(id=1)
student.grade = 10
student.save()

student = Student.objects.get(id=2)
student.delete()
```

### Benefits of Django's ORM

1. **Pythonic**: Use Python syntax instead of SQL
2. **Database-agnostic**: Write once, run on SQLite, PostgreSQL, MySQL, etc.
3. **Safe**: Automatic escaping protects against SQL injection
4. **Maintainable**: Easier to read and understand
5. **Extensible**: Can still use raw SQL when needed

## The Manager: `objects`

Every Django model has a **manager** named `objects` by default. This is your gateway to querying the database.

```python
Student.objects.all()      # All students
Student.objects.get(id=1)   # One specific student
Student.objects.filter(grade=9)  # Students matching criteria
```

The manager returns a **QuerySet** — a collection of database rows represented as Python objects.

## QuerySets: Lazy Evaluation

One of the most powerful and sometimes confusing aspects of Django's ORM is that **QuerySets are lazy**. This means:

```python
# This doesn't hit the database yet!
all_students = Student.objects.all()

# This DOES hit the database
students_list = list(all_students)

# This also hits the database
for student in all_students:
    print(student.first_name)

# This also hits the database
count = all_students.count()
```

The QuerySet is only evaluated when you:
- Iterate over it (in a for loop)
- Convert it to a list
- Call methods that require evaluation (`.count()`, `.exists()`, etc.)
- Slice it (`.first()`, `[0]`, etc.)

**Why does this matter?**
- You can chain filters without performance penalty
- You can reuse the same QuerySet multiple times
- Django optimizes the final query

Example:
```python
# These two queries are equivalent
# Query 1: Two database hits
students = Student.objects.all()
high_grade = [s for s in students if s.grade >= 8]

# Query 2: One database hit (more efficient)
high_grade = Student.objects.filter(grade__gte=8)
```

## CRUD Operations in Detail

### CREATE: Adding New Records

There are two primary ways to create objects:

**Method 1: Create and Save (Two Steps)**
```python
student = Student(
    first_name="Almaz",
    last_name="Tadesse",
    email="almaz@school.com",
    grade=9,
    date_of_birth="2010-05-15"
)
student.save()  # This hits the database
```

**Method 2: Create in One Call**
```python
student = Student.objects.create(
    first_name="Bereket",
    last_name="Alemayehu",
    email="bereket@school.com",
    grade=7,
    date_of_birth="2012-03-02"
)
# save() is called automatically
```

**Bulk Create (Efficient for many records)**
```python
students = [
    Student(first_name="Abebe", last_name="Kebede", email="abebe@school.com", grade=10, date_of_birth="2009-08-20"),
    Student(first_name="Chala", last_name="Girma", email="chala@school.com", grade=8, date_of_birth="2011-02-14"),
]
Student.objects.bulk_create(students)
```

### READ: Querying Records

#### Getting All Records
```python
all_students = Student.objects.all()
```

#### Getting a Single Record
```python
# get() - expects exactly one result, raises DoesNotExist if none, MultipleObjectsReturned if multiple
student = Student.objects.get(id=1)

# first() - gets the first result, returns None if no results
student = Student.objects.filter(grade=9).first()

# last() - gets the last result, returns None if no results
student = Student.objects.filter(grade=9).last()
```

**Important:** Use `.get()` only when you're certain there's exactly one match. Otherwise, use `.filter()` and handle the QuerySet.

#### Filtering Records
```python
# Filter by exact value
Student.objects.filter(grade=9)

# Filter by field lookup (see below for many options)
Student.objects.filter(grade__gte=8)  # grade >= 8

# Multiple filters (AND logic)
Student.objects.filter(grade=9, email__contains='@school.com')

# Chained filters (also AND logic)
Student.objects.filter(grade=9).filter(email__contains='@school.com')

# OR logic using Q objects
from django.db.models import Q
Student.objects.filter(Q(grade=9) | Q(grade=10))
```

#### Field Lookups

Field lookups are how you specify the type of comparison. Use double underscores `__`:

| Lookup | SQL Equivalent | Example |
|--------|----------------|---------|
| `exact` | `=` | `grade__exact=9` (same as `grade=9`) |
| `iexact` | `LIKE` (case-insensitive) | `first_name__iexact='almaz'` |
| `contains` | `LIKE '%value%'` | `email__contains='school'` |
| `icontains` | `ILIKE '%value%'` | `first_name__icontains='al'` |
| `startswith` | `LIKE 'value%'` | `first_name__startswith='A'` |
| `istartswith` | `ILIKE 'value%'` | `first_name__istartswith='a'` |
| `endswith` | `LIKE '%value'` | `email__endswith='.com'` |
| `iendswith` | `ILIKE '%value'` | `email__iendswith='.com'` |
| `gt` | `>` | `grade__gt=8` (grade > 8) |
| `gte` | `>=` | `grade__gte=8` (grade >= 8) |
| `lt` | `<` | `grade__lt=9` (grade < 9) |
| `lte` | `<=` | `grade__lte=9` (grade <= 9) |
| `in` | `IN` | `grade__in=[8, 9, 10]` |
| `range` | `BETWEEN` | `grade__range=(8, 10)` |
| `date` | Date comparison | `registration_date__date=date(2024, 1, 1)` |
| `year` | Year extraction | `date_of_birth__year=2010` |
| `month` | Month extraction | `date_of_birth__month=5` |
| `day` | Day extraction | `date_of_birth__day=15` |

#### Ordering Results
```python
# Ascending (default)
Student.objects.all().order_by('last_name')

# Descending (use - prefix)
Student.objects.all().order_by('-registration_date')  # Newest first

# Multiple sort fields
Student.objects.all().order_by('grade', 'last_name')  # Sort by grade, then name
```

#### Limiting Results
```python
# Get first 5 students
Student.objects.all()[:5]

# Get students 5-10
Student.objects.all()[5:10]

# Get a random student
from random import choice
random_student = choice(Student.objects.all())
```

#### Aggregation and Annotation
```python
from django.db.models import Avg, Count, Max, Min, Sum

# Average grade
avg_grade = Student.objects.aggregate(Avg('grade'))

# Count students by grade
from django.db.models import Count
Student.objects.values('grade').annotate(total=Count('id'))
# Returns: [{'grade': 8, 'total': 5}, {'grade': 9, 'total': 7}, ...]

# Highest grade
highest = Student.objects.aggregate(Max('grade'))
```

### UPDATE: Modifying Records

**Update a single record:**
```python
student = Student.objects.get(id=1)
student.grade = 10
student.save()
```

**Bulk update (more efficient):**
```python
# Update all grade 9 students to grade 10
Student.objects.filter(grade=9).update(grade=10)
```

**Update with F() expressions (avoids race conditions):**
```python
from django.db.models import F

# Increment every student's grade by 1
Student.objects.all().update(grade=F('grade') + 1)
```

### DELETE: Removing Records

**Delete a single record:**
```python
student = Student.objects.get(id=1)
student.delete()
```

**Bulk delete:**
```python
# Delete all students in grade 6
Student.objects.filter(grade=6).delete()
```

**Important:** Bulk delete is permanent and irreversible. Always check your filter first:
```python
# WRONG: Don't do this without checking first
Student.objects.filter(grade__lt=8).delete()

# RIGHT: Check first
students_to_delete = Student.objects.filter(grade__lt=8)
print(students_to_delete)  # Verify this is the right set
students_to_delete.delete()  # Then delete
```

## Related Objects (Preview)

While our Student Registration System doesn't use relationships yet, it's worth knowing how the ORM handles them:

### ForeignKey (One-to-Many)

If we had a `Course` model with a foreign key to `Student`:

```python
# In models.py
class Course(models.Model):
    title = models.CharField(max_length=100)
    student = models.ForeignKey(Student, on_delete=models.CASCADE)

# Accessing related objects
student = Student.objects.get(id=1)
courses = student.course_set.all()  # All courses for this student (reverse relation)

course = Course.objects.get(id=1)
student = course.student  # The student for this course (forward relation)
```

### ManyToManyField (Many-to-Many)

For tracking which students are enrolled in which courses:

```python
class Course(models.Model):
    title = models.CharField(max_length=100)
    students = models.ManyToManyField(Student)

course = Course.objects.get(id=1)
students = course.students.all()  # All students in this course

student = Student.objects.get(id=1)
courses = student.course_set.all()  # All courses this student is in
```

## The Django Shell

The Django shell is a Python REPL (Read-Eval-Print Loop) with your Django environment already loaded. This means:
- All your models are imported
- Database connections are set up
- You can interact with your application directly

```bash
python manage.py shell
```

**Difference from regular Python shell:**
```bash
# Regular Python shell - Django not loaded
$ python
>>> from students.models import Student
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ModuleNotFoundError: No module named 'students'

# Django shell - Django loaded
$ python manage.py shell
>>> from students.models import Student
>>> Student.objects.all()
<QuerySet []>
```

**Pro tip:** Use `python manage.py shell_plus` if you have `django-extensions` installed — it automatically imports all your models.

## Common ORM Pitfalls

1. **N+1 Query Problem**: Making one query per object in a loop
   ```python
   # BAD: N+1 queries (one for each student)
   for student in Student.objects.all():
       print(student.course_set.count())  # Separate query for each student!
   
   # GOOD: Two queries total
   from django.db.models import Prefetch
   students = Student.objects.prefetch_related('course_set').all()
   for student in students:
       print(student.course_set.count())  # Uses preloaded data
   ```

2. **Not using `select_related` for foreign keys**: Similar to prefetch but for ForeignKey
   ```python
   # Without select_related: 1 query for students + 1 for each student's related data
   # With select_related: 1 query with JOIN
   students = Student.objects.select_related('some_foreign_key').all()
   ```

3. **Modifying QuerySets**: QuerySets are immutable
   ```python
   qs = Student.objects.filter(grade=9)
   qs = qs.filter(first_name__startswith='A')  # Creates new QuerySet
   ```

## ORM vs Raw SQL

| Task | ORM | Raw SQL |
|------|-----|---------|
| Readability | High | Lower |
| Database portability | High | Low (database-specific) |
| Performance | Usually good, sometimes needs optimization | Full control |
| Complex queries | Limited for very complex cases | Full power |
| SQL injection | Protected | Vulnerable if not careful |

**When to use raw SQL:**
- Extremely complex queries that are hard to express with the ORM
- Database-specific features
- Performance-critical sections where ORM overhead matters

```python
# Using raw SQL when needed
from django.db import connection

with connection.cursor() as cursor:
    cursor.execute("SELECT * FROM students_student WHERE grade > %s", [8])
    results = cursor.fetchall()
```

## Summary: ORM in Our Student Registration System

In Lesson 3, you practiced all CRUD operations on the `Student` model:
- **Create**: `Student.objects.create(...)` and `student.save()`
- **Read**: `Student.objects.all()`, `.filter()`, `.get()`
- **Update**: `student.grade = 10; student.save()` and `.update()`
- **Delete**: `student.delete()` and `.delete()`

These same operations power:
- The Django admin (Lesson 2)
- Custom views (Lessons 4-6)
- Any other part of your application that needs data access

The ORM is the foundation that connects your Python code to the database. Mastering it is essential for Django development.
