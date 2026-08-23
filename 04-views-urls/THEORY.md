# Theory: Views & URL Routing

## The Request/Response Cycle in Django

When a user visits a page in your Student Registration System, here's what happens:

```
1. Browser sends HTTP Request
   (e.g., GET http://localhost:8000/hello/)
        ↓
2. Django receives request at the web server
        ↓
3. Project URL Configuration (student_registration_system/urls.py)
   - Checks URL patterns
   - Matches '/hello/' to the students app's URLs
        ↓
4. App URL Configuration (students/urls.py)
   - Matches '/hello/' to the hello view
        ↓
5. View Function (students/views.py)
   - Executes hello() function
   - Processes request
        ↓
6. View returns HttpResponse
   - "Hello, World!"
        ↓
7. Django sends HTTP Response to browser
   - Browser displays "Hello, World!"
```

This is the **View-URL pattern** that Django uses to route requests to the appropriate code.

## What is a View?

In Django, a **view** is a Python function (or class) that:
1. Takes a **request** object as its first argument
2. Performs some logic
3. Returns a **response** object

The view is where your business logic lives. It decides:
- What data to fetch from the database
- What processing to perform
- What response to send back

### View Function Structure

```python
def view_name(request):
    # 1. Process the request
    #    - Read GET/POST parameters
    #    - Fetch data from database
    #    - Perform business logic
    
    # 2. Return a response
    return HttpResponse("content")
    # or return render(request, 'template.html', context)
    # or return redirect('some_url')
```

**In our Student Registration System:**
```python
def hello(request):
    return HttpResponse("Hello, World!")
```

This simple view:
- Takes a `request` parameter (which it doesn't use in this case)
- Returns an `HttpResponse` with the text "Hello, World!"

## The Request Object

The `request` object contains all information about the incoming HTTP request:

### Key Attributes

| Attribute | Type | Description | Example |
|-----------|------|-------------|---------|
| `request.method` | str | HTTP method ('GET', 'POST', etc.) | `request.method == 'GET'` |
| `request.GET` | QueryDict | GET parameters (from URL query string) | `request.GET['search']` |
| `request.POST` | QueryDict | POST data (from form submissions) | `request.POST['email']` |
| `request.path` | str | The path of the request | `/hello/` |
| `request.path_info` | str | Path without domain | `/hello/` |
| `request.COOKIES` | dict | Browser cookies | `request.COOKIES.get('sessionid')` |
| `request.user` | User | The authenticated user (if any) | `request.user.is_authenticated` |
| `request.META` | dict | All HTTP headers | `request.META['HTTP_USER_AGENT']` |

### QueryDict Objects

`request.GET` and `request.POST` are `QueryDict` objects (like dictionaries, but with special handling for multiple values with the same key):

```python
# GET request: /search/?q=django&page=2
request.GET['q']  # Returns 'django'
request.GET.get('q')  # Returns 'django' (safe, returns None if missing)
request.GET.get('q', 'default')  # Returns 'django' or 'default'

# POST request with form data
request.POST['username']  # Returns the username value
request.POST.getlist('tags')  # Returns list of all 'tags' values
```

**Important:** Always use `.get()` or check if the key exists before accessing, to avoid KeyError exceptions.

## The Response Object

Views must return a response. Django provides several response types:

### HttpResponse
The most basic response — sends a string with a given content type:

```python
from django.http import HttpResponse

return HttpResponse("Hello, World!")
return HttpResponse("Hello", content_type="text/plain")
return HttpResponse("Hello", status=201)  # HTTP 201 Created
```

### HttpResponseRedirect
Redirects the browser to another URL:

```python
from django.http import HttpResponseRedirect

return HttpResponseRedirect('/students/')
```

### render() Shortcut
Renders a template with context data (we'll use this in Lesson 5):

```python
from django.shortcuts import render

return render(request, 'template.html', {'key': 'value'})
```

This is equivalent to:
1. Loading the template
2. Passing the context to it
3. Rendering the template to a string
4. Returning an HttpResponse with that string

### redirect() Shortcut
A convenience function for redirects:

```python
from django.shortcuts import redirect

return redirect('student_list')  # By URL name
return redirect('/students/')  # By URL path
```

## URL Routing: Mapping URLs to Views

Django uses a **URL dispatcher** that examines the URL of an incoming request and routes it to the appropriate view.

### URL Configuration Files

Django checks URL patterns in this order:
1. Project's `urls.py` (main URL configuration)
2. Included app `urls.py` files

This hierarchical approach allows each app to manage its own URLs.

#### Project-level URLs (student_registration_system/urls.py)

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),           # Django admin
    path('', include('students.urls')),        # Include students app URLs
]
```

Key points:
- `admin.site.urls` provides all admin URLs
- `include('students.urls')` delegates to the students app
- The empty string `''` means these URLs are at the root

#### App-level URLs (students/urls.py)

```python
from django.urls import path
from . import views

urlpatterns = [
    path('hello/', views.hello, name='hello'),
]
```

Key points:
- `path('hello/', ...)` matches `/hello/` (relative to what's included)
- `views.hello` is the view function to call
- `name='hello'` names this URL pattern for reverse lookups

### The `path()` Function

The `path()` function defines a URL pattern:

```python
path(route, view, kwargs=None, name=None)
```

| Parameter | Type | Purpose |
|-----------|------|---------|
| `route` | str | The URL pattern to match |
| `view` | callable | The view function or class to use |
| `kwargs` | dict | Additional arguments to pass to the view |
| `name` | str | Name for reverse URL lookups |

### URL Pattern Syntax

| Pattern | Matches | Example |
|---------|---------|---------|
| `hello/` | Literal match | `/hello/` |
| `<name>` | Named capture group | `students/<int:pk>/` |
| `<converter:name>` | Typed capture group | `<int:year>/` |
| `path/` | Exact match | `/path/` but not `/path/extra/` |

### Path Converters

Django provides built-in converters for URL parameters:

| Converter | Matches | Example | View Parameter Type |
|-----------|---------|---------|------------------------|
| `str` (default) | Any non-empty string (no `/`) | `<name>` | `str` |
| `int` | Integer | `<int:id>` | `int` |
| `slug` | Slug (ASCII letters, numbers, hyphens, underscores) | `<slug:title>` | `str` |
| `uuid` | UUID | `<uuid:pk>` | `UUID` |
| `path` | Any string including `/` | `<path:full_path>` | `str` |

**In our Student Registration System (Lesson 6 preview):**
```python
path('delete/<int:student_id>/', views.delete_student, name='delete_student')
```

This captures an integer from the URL and passes it to the view as `student_id`.

### Reverse URL Lookups

Instead of hardcoding URLs in your code, you can use **URL names** and **URL tags** to generate URLs dynamically:

```python
# In urls.py
path('hello/', views.hello, name='hello')

# In code
from django.urls import reverse

# Generate the URL
url = reverse('hello')  # Returns '/hello/'

# In templates
{% url 'hello' %}  # Outputs: /hello/
```

**Benefits:**
- Change the URL pattern in one place, all links update automatically
- Avoid hardcoding URLs that might change
- Makes your code more maintainable

**In our base.html (Lesson 5):**
```html
<a href="{% url 'student_list' %}">All Students</a>
```

### URL Namespacing

For larger projects, you can namespace your URLs to avoid naming conflicts:

```python
# In students/urls.py
app_name = 'students'

urlpatterns = [
    path('', views.student_list, name='list'),
]

# In templates
{% url 'students:list' %}
```

## View Types

### Function-Based Views (FBVs)

Views defined as functions:

```python
def hello(request):
    return HttpResponse("Hello, World!")
```

**Pros:** Simple, easy to understand, explicit
**Cons:** Can get unwieldy for complex logic

### Class-Based Views (CBVs)

Views defined as classes (not covered in this course, but worth knowing):

```python
from django.views import View

class HelloView(View):
    def get(self, request):
        return HttpResponse("Hello, World!")
```

**Pros:** Reusable, organized, built-in methods for HTTP verbs
**Cons:** More complex, steeper learning curve

Django also provides **generic class-based views** that handle common patterns (list views, detail views, create/update/delete views) with minimal code.

## HTTP Methods

Different HTTP methods serve different purposes:

| Method | Purpose | View Handling |
|--------|---------|----------------|
| GET | Retrieve data | `def hello(request):` |
| POST | Submit data | Check `request.method == 'POST'` |
| PUT | Update data | Less common in Django |
| PATCH | Partial update | Less common in Django |
| DELETE | Delete data | Less common in Django |

**In Django views:**
```python
def register_student(request):
    if request.method == 'POST':
        # Handle form submission
        return HttpResponse("Student created")
    else:
        # Show the form
        return HttpResponse("Show form here")
```

## Status Codes

Always return the appropriate HTTP status code:

| Code | Name | When to Use |
|------|------|-------------|
| 200 | OK | Successful GET/POST |
| 201 | Created | Resource created (POST) |
| 301 | Moved Permanently | URL changed permanently |
| 302 | Found | URL changed temporarily (redirect) |
| 400 | Bad Request | Invalid input |
| 403 | Forbidden | Authenticated but not authorized |
| 404 | Not Found | Resource doesn't exist |
| 405 | Method Not Allowed | Wrong HTTP method |
| 500 | Internal Server Error | Server error |

```python
from django.http import HttpResponse, HttpResponseNotFound, HttpResponseForbidden

return HttpResponse("OK", status=200)
return HttpResponseNotFound("Not found")
return HttpResponseForbidden("Not allowed")
```

## URL Design Best Practices

1. **Use nouns, not verbs**: `/students/` not `/get_students/`
2. **Use plural for collections**: `/students/` for listing, `/students/1/` for detail
3. **Use RESTful conventions**:
   - `GET /students/` - List all students
   - `GET /students/1/` - Get student 1
   - `POST /students/` - Create a student
   - `PUT /students/1/` - Update student 1
   - `DELETE /students/1/` - Delete student 1
4. **Use hyphens, not underscores**: `/student-list/` not `/student_list/`
5. **Keep it simple**: `/students/1/` not `/app/v1/students/id/1/`
6. **Trailing slashes**: Django convention is to include them

**In our Student Registration System:**
- `/` - Home page (student list)
- `/hello/` - Demo page
- `/admin/` - Admin interface
- `/register/` - Registration form
- `/delete/1/` - Delete student 1

## Putting It All Together

In Lesson 4, you built:

1. **A view function** (`hello`) that returns a simple text response
2. **A URL pattern** in `students/urls.py` that maps `/hello/` to the view
3. **Included** the app's URLs in the project's URLs

This established the foundation for all future pages:
- The same pattern applies to every view
- The URL structure can grow as needed
- Views can become more complex (fetching data, processing forms)

In Lesson 5, you'll see how views work with templates to return HTML instead of plain text.
