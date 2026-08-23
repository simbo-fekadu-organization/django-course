# Theory: Full CRUD in the Browser

## What is CRUD?

**CRUD** stands for the four basic operations that can be performed on data:

| Operation | HTTP Method | Description | Example in Student Registration System |
|-----------|-------------|-------------|--------------------------------------|
| **C**reate | POST | Add new data | Register a new student |
| **R**ead | GET | Retrieve data | View student list, view single student |
| **U**pdate | PUT/PATCH | Modify data | Edit student information |
| **D**elete | DELETE | Remove data | Delete a student |

In a web application, these operations are typically triggered by user actions in the browser and handled by server-side code.

## The Web Request/Response Cycle for CRUD

### Create (POST)
```
Browser: User fills form and clicks Submit
        ↓
Browser: Sends POST request with form data
        ↓
Django: Routes to register_student view
        ↓
View: Validates data, creates Student object, saves to database
        ↓
View: Returns redirect to student list
        ↓
Browser: Follows redirect, loads student list (now includes new student)
```

### Read (GET)
```
Browser: User visits / or /admin/
        ↓
Django: Routes to student_list view
        ↓
View: Queries database for all students
        ↓
View: Renders template with student data
        ↓
Browser: Displays student table
```

### Update (POST to edit view)
```
Browser: User fills edit form and clicks Save
        ↓
Browser: Sends POST request with updated data
        ↓
Django: Routes to edit_student view
        ↓
View: Fetches student, updates fields, saves to database
        ↓
View: Returns redirect to student list or detail
        ↓
Browser: Follows redirect, shows updated data
```

### Delete (GET to delete URL - not ideal)
```
Browser: User clicks Delete link
        ↓
Django: Routes to delete_student view
        ↓
View: Fetches student, deletes from database
        ↓
View: Returns redirect to student list
        ↓
Browser: Follows redirect, student no longer in list
```

## HTML Forms

### Form Basics

An HTML form collects user input and sends it to the server:

```html
<form method="POST" action="/register/">
  <input type="text" name="first_name" placeholder="First Name">
  <input type="text" name="last_name" placeholder="Last Name">
  <button type="submit">Register</button>
</form>
```

| Attribute | Purpose | Values |
|-----------|---------|--------|
| `method` | HTTP method | GET, POST |
| `action` | Where to send | URL path |
| `enctype` | Encoding type | `application/x-www-form-urlencoded`, `multipart/form-data` |

### Form Methods: GET vs POST

| Aspect | GET | POST |
|--------|-----|------|
| Visibility | Data in URL | Data in request body |
| Bookmarkable | Yes | No |
| Cacheable | Yes | No |
| Idempotent | Yes | No (usually) |
| CSRF protection | Not needed | Required |
| Data limit | ~2048 chars | Unlimited |

**Rule of thumb:**
- Use **GET** for read-only operations (search, filter, view)
- Use **POST** for operations that change data (create, update, delete)

### Form Input Types

In our Student Registration System:

| Input Type | Purpose | Example |
|------------|---------|---------|
| `text` | Single-line text | First name, last name |
| `email` | Email address (with validation) | Email field |
| `number` | Numeric value | Grade field |
| `date` | Date picker | Date of birth |
| `password` | Hidden text | (Not used in our example) |
| `checkbox` | Boolean | (Not used in our example) |
| `select` | Dropdown | (Could be used for grade level) |
| `textarea` | Multi-line text | (Not used in our example) |
| `hidden` | Invisible field | (Not used in our example) |

**Our registration form:**
```html
<form method="POST">
  {% csrf_token %}
  <input type="text" name="first_name" placeholder="First Name" required><br>
  <input type="text" name="last_name" placeholder="Last Name" required><br>
  <input type="email" name="email" placeholder="Email" required><br>
  <input type="number" name="grade" placeholder="Grade" required><br>
  <input type="date" name="dob" required><br>
  <button type="submit">Register</button>
</form>
```

### Form Attributes

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `name` | Identifies the field (used in `request.POST`) | `name="first_name"` |
| `value` | Default/initial value | `value="John"` |
| `placeholder` | Hint text | `placeholder="First Name"` |
| `required` | Field must be filled | `required` |
| `disabled` | Field cannot be edited | `disabled` |
| `readonly` | Field cannot be edited but is submitted | `readonly` |
| `min`/`max` | Numeric range | `min="1" max="12"` for grade |
| `minlength`/`maxlength` | String length limits | `minlength="2" maxlength="100"` |
| `pattern` | Regex validation | `pattern="[A-Za-z]+"` |

## CSRF Protection

### What is CSRF?

**CSRF (Cross-Site Request Forgery)** is an attack where a malicious website tricks a user's browser into making requests to another website where the user is already authenticated.

Example attack scenario:
1. User logs into your Student Registration System at `school.com`
2. User visits malicious site `evil.com`
3. `evil.com` has an invisible form that submits to `school.com/delete_all_students/`
4. If the user's browser is still logged into `school.com`, the request succeeds!

### Django's Solution

Django provides built-in CSRF protection:

1. **CSRF Token in Forms**: Include `{% csrf_token %}` in every POST form
2. **CSRF Middleware**: Validates the token on POST requests
3. **CSRF Cookie**: Stores the token in a cookie for comparison

**In our templates:**
```html
<form method="POST">
  {% csrf_token %}  <!-- This MUST be inside the <form> -->
  <!-- form fields -->
</form>
```

**What happens without `{% csrf_token %}`:**
- The form submits
- Django's middleware detects the missing/invalid CSRF token
- Returns a **403 Forbidden** error

### How CSRF Tokens Work

```
1. Django generates a unique, random token
   ↓
2. Token stored in:
   - The session (server-side)
   - A cookie (client-side)
   - The form as a hidden field
   ↓
3. User submits form
   ↓
4. Django compares:
   - Token from form
   - Token from cookie
   - Token from session
   ↓
5. If all match → request is legitimate → process it
   ↓
6. If any don't match → request might be CSRF → reject with 403
```

### CSRF Exemptions

For APIs or special cases, you can exempt views from CSRF protection:

```python
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt
def my_view(request):
    # No CSRF protection
    ...
```

Or disable for all POST requests to a view class:

```python
from django.utils.decorators import method_decorator
from django.views.decorators.csrf import csrf_exempt

@method_decorator(csrf_exempt, name='dispatch')
class MyView(View):
    ...
```

**Note:** Only exempt views that genuinely need it (e.g., webhook endpoints). Regular forms should always use CSRF protection.

## Processing Form Data in Views

### Reading Form Data

In Django views, POST data is available via `request.POST` (a QueryDict):

```python
def register_student(request):
    if request.method == 'POST':
        first_name = request.POST['first_name']
        last_name = request.POST['last_name']
        email = request.POST['email']
        grade = request.POST['grade']
        dob = request.POST['dob']
```

### Handling Missing Fields

Always use `.get()` or check if the key exists:

```python
# Safe approach
first_name = request.POST.get('first_name', '')
if not first_name:
    return HttpResponse("First name is required", status=400)

# Or with default
first_name = request.POST.get('first_name', 'Unknown')
```

### Creating Objects from Form Data

```python
try:
    student = Student(
        first_name=request.POST['first_name'],
        last_name=request.POST['last_name'],
        email=request.POST['email'],
        grade=int(request.POST['grade']),  # Convert to int
        date_of_birth=request.POST['dob'],
    )
    student.save()
    messages.success(request, f"Student {student.full_name()} added!")
    return redirect('student_list')
except Exception as e:
    messages.error(request, f"Error: {e}")
    return render(request, 'register.html')
```

**Key points:**
- Wrap in `try/except` to handle validation errors
- Use `int()` to convert numeric fields
- Use `messages` framework to give feedback to users
- Always `redirect()` after a successful POST

## Redirects: The Post/Redirect/Get Pattern

### The Problem: Duplicate Form Submissions

Without redirects, this can happen:
1. User submits form (POST)
2. Server creates student, returns HTML response
3. User refreshes page (re-POSTs)
4. **Duplicate student created!**

### The Solution: PRG Pattern

**Post/Redirect/Get** is a web development best practice:

```
User submits form (POST)
        ↓
Server processes data (creates student)
        ↓
Server returns redirect response (302)
        ↓
Browser follows redirect (GET)
        ↓
User sees result page
        ↓
User refreshes → GET request → Safe!
```

**In Django:**
```python
from django.shortcuts import redirect

def register_student(request):
    if request.method == 'POST':
        # Process form, create student
        student.save()
        return redirect('student_list')  # Redirect to list view
    return render(request, 'register.html')
```

### Benefits of Redirects

1. **Prevents duplicate submissions** on page refresh
2. **Bookmarkable URLs** - Users can bookmark the result page
3. **Back button works** - Users can use the back button without re-submitting
4. **Clean URLs** - The URL reflects the resource being viewed, not the action taken

### The `redirect()` Function

Django provides several ways to redirect:

```python
from django.shortcuts import redirect

# By URL name (recommended)
return redirect('student_list')

# By URL path
return redirect('/students/')

# By URL with arguments
return redirect('delete_student', student_id=1)

# Permanent redirect (301)
from django.http import HttpResponsePermanentRedirect
return HttpResponsePermanentRedirect('/new-url/')
```

**Best practice:** Use URL names (`redirect('student_list')`) instead of hardcoded paths. This way, if you change the URL pattern, all redirects still work.

## The Messages Framework

Django's **messages framework** provides a way to give users feedback about actions they've taken. This is especially useful after form submissions and redirects.

### Setting Messages

```python
from django.contrib import messages

# Different message levels
messages.debug(request, "Debug message")
messages.info(request, "Informational message")
messages.success(request, "Success! Student created.")
messages.warning(request, "Warning: something might be wrong.")
messages.error(request, "Error: could not create student.")
```

**In our registration view:**
```python
try:
    student.save()
    messages.success(request, f"Student {student.full_name()} added!")
    return redirect('student_list')
except Exception as e:
    messages.error(request, f"Error: {e}")
    return render(request, 'register.html')
```

### Displaying Messages in Templates

Messages are available in templates via the `messages` context variable:

```html
{% if messages %}
  <ul>
    {% for message in messages %}
      <li>
        {% if message.tags %}
          <strong class="{{ message.tags }}">{{ message }}</strong>
        {% else %}
          <strong>{{ message }}</strong>
        {% endif %}
      </li>
    {% endfor %}
  </ul>
{% endif %}
```

**With Bootstrap classes:**
```html
{% for message in messages %}
  <div class="alert alert-{{ message.tags }}">
    {{ message }}
  </div>
{% endfor %}
```

### Message Tags

Messages have tags that indicate their level:
- `debug`
- `info`
- `success`
- `warning`
- `error`

These map to CSS classes for styling.

## Delete Operations

### Simple Delete (Our Approach in Lesson 6)

```python
def delete_student(request, student_id):
    student = Student.objects.get(id=student_id)
    student.delete()
    messages.success(request, "Student deleted.")
    return redirect('student_list')
```

With URL pattern:
```python
path('delete/<int:student_id>/', views.delete_student, name='delete_student')
```

And link in template:
```html
<a href="{% url 'delete_student' student.id %}">Delete</a>
```

### The Problem with GET for Delete

Using a simple link (GET request) for delete operations has issues:

1. **Accidental deletion**: Users can accidentally click the link
2. **No confirmation**: No "Are you sure?" prompt
3. **Search engine indexing**: Search engines might follow delete links
4. **Bookmarkable**: Users could bookmark delete links
5. **CSRF**: GET requests can be forged more easily

### Better Approach: Confirmation Page

```python
def delete_student(request, student_id):
    student = Student.objects.get(id=student_id)
    
    if request.method == 'POST':
        # User confirmed deletion
        student.delete()
        messages.success(request, "Student deleted.")
        return redirect('student_list')
    
    # Show confirmation page
    return render(request, 'delete_confirm.html', {'student': student})
```

With template:
```html
<!-- delete_confirm.html -->
{% extends 'base.html' %}

{% block content %}
  <h2>Delete Student</h2>
  <p>Are you sure you want to delete {{ student.full_name }}?</p>
  <form method="POST">
    {% csrf_token %}
    <button type="submit" class="btn btn-danger">Confirm Delete</button>
    <a href="{% url 'student_list' %}" class="btn btn-secondary">Cancel</a>
  </form>
{% endblock %}
```

### Even Better: POST with Hidden Form

For a better user experience, use JavaScript to confirm:

```html
<a href="{% url 'delete_student' student.id %}" 
   onclick="return confirm('Are you sure you want to delete {{ student.full_name }}?');">
  Delete
</a>
```

Or use a form with a button styled as a link:

```html
<form method="POST" action="{% url 'delete_student' student.id %}" style="display: inline;">
  {% csrf_token %}
  <button type="submit" class="delete-btn"
          onclick="return confirm('Are you sure?');">
    Delete
  </button>
</form>
```

## Update Operations (Stretch Goal)

While not covered in Lesson 6, update operations follow the same pattern as create:

### Edit View

```python
def edit_student(request, student_id):
    student = Student.objects.get(id=student_id)
    
    if request.method == 'POST':
        # Update the student
        student.first_name = request.POST['first_name']
        student.last_name = request.POST['last_name']
        student.email = request.POST['email']
        student.grade = int(request.POST['grade'])
        student.date_of_birth = request.POST['dob']
        student.save()
        messages.success(request, f"Student {student.full_name()} updated!")
        return redirect('student_list')
    
    # Show edit form with current data
    return render(request, 'edit.html', {'student': student})
```

### Edit Template

```html
<!-- edit.html -->
{% extends 'base.html' %}

{% block content %}
  <h2>Edit Student</h2>
  <form method="POST">
    {% csrf_token %}
    <input type="text" name="first_name" 
           value="{{ student.first_name }}" required><br>
    <input type="text" name="last_name" 
           value="{{ student.last_name }}" required><br>
    <input type="email" name="email" 
           value="{{ student.email }}" required><br>
    <input type="number" name="grade" 
           value="{{ student.grade }}" required><br>
    <input type="date" name="dob" 
           value="{{ student.date_of_birth }}" required><br>
    <button type="submit">Save Changes</button>
  </form>
{% endblock %}
```

## Form Validation

### Model-Level Validation

Add validation to your model:

```python
from django.core.validators import MinValueValidator, MaxValueValidator, EmailValidator

class Student(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)
    email = models.EmailField(unique=True, 
                              validators=[EmailValidator(message="Invalid email address")])
    grade = models.IntegerField(
        validators=[
            MinValueValidator(1, message="Grade must be at least 1"),
            MaxValueValidator(12, message="Grade must be at most 12")
        ]
    )
    date_of_birth = models.DateField()
```

**Validation is automatically run when:**
- Calling `model.full_clean()`
- Saving a model (if `validate` is called)
- Using ModelForms (covered in advanced Django)

### Manual Validation in Views

```python
def register_student(request):
    if request.method == 'POST':
        # Validate required fields
        required_fields = ['first_name', 'last_name', 'email', 'grade', 'dob']
        for field in required_fields:
            if not request.POST.get(field):
                messages.error(request, f"{field.replace('_', ' ').title()} is required")
                return render(request, 'register.html')
        
        # Validate email format
        email = request.POST['email']
        if '@' not in email:
            messages.error(request, "Invalid email address")
            return render(request, 'register.html')
        
        # Validate grade
        try:
            grade = int(request.POST['grade'])
            if grade < 1 or grade > 12:
                messages.error(request, "Grade must be between 1 and 12")
                return render(request, 'register.html')
        except ValueError:
            messages.error(request, "Grade must be a number")
            return render(request, 'register.html')
        
        # If all validations pass, create the student
        try:
            student = Student.objects.create(
                first_name=request.POST['first_name'],
                last_name=request.POST['last_name'],
                email=request.POST['email'],
                grade=grade,
                date_of_birth=request.POST['dob'],
            )
            messages.success(request, f"Student {student.full_name()} added!")
            return redirect('student_list')
        except IntegrityError:
            messages.error(request, "A student with this email already exists")
            return render(request, 'register.html')
```

### Using Django's Validator Classes

Django provides built-in validators:

```python
from django.core.validators import (
    MinValueValidator, MaxValueValidator,
    MinLengthValidator, MaxLengthValidator,
    EmailValidator, URLValidator,
    RegexValidator
)

# In your model
class Student(models.Model):
    grade = models.IntegerField(
        validators=[MinValueValidator(1), MaxValueValidator(12)]
    )
    email = models.EmailField(
        validators=[EmailValidator(message="Please enter a valid email")]
    )
```

## HTTP Status Codes for CRUD

| Operation | Success Status | Error Status |
|-----------|----------------|--------------|
| Create | 201 Created | 400 Bad Request |
| Read | 200 OK | 404 Not Found |
| Update | 200 OK | 400 Bad Request, 404 Not Found |
| Delete | 204 No Content | 404 Not Found |

**In Django:**
```python
from django.http import HttpResponse, JsonResponse

# Create - 201 Created
return HttpResponse(status=201)
return JsonResponse({'message': 'Created'}, status=201)

# Read - 200 OK
return HttpResponse("OK", status=200)

# Not Found - 404
from django.http import Http404
raise Http404("Student not found")
# or
from django.shortcuts import get_object_or_404
student = get_object_or_404(Student, id=student_id)

# Bad Request - 400
return HttpResponse("Invalid data", status=400)

# No Content - 204
return HttpResponse(status=204)
```

## Putting It All Together in Our Student Registration System

In Lesson 6, you implemented:

1. **Registration Form (Create)**:
   - HTML form with CSRF token
   - View that processes POST data
   - Redirect after successful creation
   - Success/error messages

2. **Delete Links (Delete)**:
   - Simple GET-based delete (for course simplicity)
   - Redirect after deletion
   - Success messages

3. **Student List (Read)**:
   - Already implemented in Lesson 5
   - Displays all students in a table
   - Links to delete action

**What's missing for complete CRUD:**
- **Update/Edit**: A form to edit existing students (exercise in Lesson 6)
- **Detail View**: A page showing a single student's details
- **Confirmation**: Proper confirmation for delete operations

## Security Considerations

1. **Always use CSRF protection** on POST forms
2. **Validate all user input** - never trust form data
3. **Use HTTPS** in production to encrypt form submissions
4. **Sanitize output** to prevent XSS (Django does this by default)
5. **Handle exceptions** gracefully - don't expose internal errors to users
6. **Use proper HTTP methods** - GET for read, POST for write operations
7. **Consider rate limiting** for forms to prevent brute force attacks

## Performance Considerations

1. **Use `select_related` for foreign keys** to avoid N+1 queries
2. **Use `prefetch_related` for many-to-many** relationships
3. **Only fetch what you need** - use `.only()` and `.defer()`
4. **Use bulk operations** when possible (`.update()`, `.delete()`)
5. **Consider pagination** for large lists

## Real-World Enhancements

For a production Student Registration System, consider:

1. **User Authentication**: Only allow registered users to add/edit students
2. **Permissions**: Different roles (admin, teacher, student) have different access
3. **Form Classes**: Use Django's `ModelForm` to auto-generate forms from models
4. **Class-Based Views**: Use Django's generic views for common patterns
5. **Pagination**: Split large student lists across multiple pages
6. **Search & Filter**: Allow filtering students by grade, name, etc.
7. **Export**: Add CSV/Excel export functionality
8. **Import**: Add bulk import from CSV
9. **API**: Add a REST API for programmatic access
10. **AJAX**: Use JavaScript for smoother user experience
