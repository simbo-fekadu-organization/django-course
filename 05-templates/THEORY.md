# Theory: Django Templates

## What is a Template?

A **template** is an HTML file with special syntax that allows you to:
- Insert dynamic content
- Control flow (loops, conditionals)
- Reuse common layout elements
- Organize your presentation logic

In Django, templates are the **V** in MVT (Model-View-Template). They separate the presentation layer from the business logic, allowing designers to work on HTML/CSS while developers work on Python code.

**In our Student Registration System:**
- `base.html` - Shared layout for all pages
- `student_list.html` - Displays all students in a table
- `register.html` - Form for registering new students

## The Template Rendering Process

```
1. View Function (students/views.py)
   - Fetches data from database
   - Creates context dictionary
        ↓
2. render() Function
   - Loads the template file
   - Passes context to template
        ↓
3. Template Engine
   - Processes template tags and variables
   - Replaces them with actual values
        ↓
4. Rendered HTML
   - Sent to browser as HttpResponse
```

**Example from our course:**
```python
# views.py
def student_list(request):
    students = Student.objects.all().order_by('-registration_date')
    return render(request, 'student_list.html', {'students': students})
```

```html
<!-- student_list.html -->
{% for student in students %}
<tr>
  <td>{{ student.full_name }}</td>
  <td>{{ student.grade }}</td>
</tr>
{% endfor %}
```

## Template Language Syntax

Django's template language has four constructs:

1. **Variables**: `{{ variable }}` - Outputs a value
2. **Tags**: `{% tag %}` - Control logic
3. **Filters**: `{{ variable|filter }}` - Modifies a value
4. **Comments**: `{# comment #}` - Not displayed in output

### Variables

Variables display the value of an object from the context:

```html
<p>Hello, {{ user.name }}!</p>
<p>There are {{ students.count }} students.</p>
```

**Dot notation:** You can access attributes, method calls, and dictionary keys using dots:

```html
{{ student.first_name }}        <!-- Attribute access -->
{{ student.full_name }}         <!-- Method call (no parentheses) -->
{{ student.full_name|upper }}  <!-- Method + filter -->
{{ student.0.first_name }}      <!-- List index -->
{{ context_dict.key }}          <!-- Dictionary access -->
```

**Important:** Method calls in templates are called without parentheses, and without arguments. The method must take no arguments (or have defaults for all arguments).

### Tags

Tags provide control flow and other functionality:

#### `for` Tag - Looping

```html
{% for student in students %}
  <p>{{ student.full_name }} - Grade {{ student.grade }}</p>
{% endfor %}
```

**Special loop variables:**
- `forloop.counter` - 1-based index
- `forloop.counter0` - 0-based index
- `forloop.revcounter` - Reverse counter
- `forloop.revcounter0` - Reverse counter (0-based)
- `forloop.first` - True if first iteration
- `forloop.last` - True if last iteration
- `forloop.parentloop` - Parent loop context (for nested loops)

```html
{% for student in students %}
  <p>{{ forloop.counter }}. {{ student.full_name }}</p>
{% endfor %}
```

#### `if` / `elif` / `else` Tags - Conditionals

```html
{% if student.grade >= 8 %}
  <p>{{ student.full_name }} is in high grade.</p>
{% elif student.grade >= 5 %}
  <p>{{ student.full_name }} is in middle grade.</p>
{% else %}
  <p>{{ student.full_name }} is in low grade.</p>
{% endif %}
```

**Boolean operators:**
```html
{% if student and student.is_active %}
  ...
{% endif %}

{% if student.grade > 8 or student.email %}
  ...
{% endif %}

{% if not student %}
  ...
{% endif %}
```

#### `block` Tag - Template Inheritance

The `block` tag defines a section that can be overridden by child templates:

```html
<!-- base.html -->
{% block content %}
  <p>Default content</p>
{% endblock %}

<!-- child.html -->
{% extends 'base.html' %}

{% block content %}
  <p>Custom content for this page</p>
{% endblock %}
```

#### `extends` Tag - Inheritance

The `extends` tag establishes inheritance from a parent template. It must be the **first tag** in the template:

```html
{% extends 'base.html' %}
{% block title %}Student List{% endblock %}
{% block content %}
  <!-- Page-specific content -->
{% endblock %}
```

#### `include` Tag - Including Other Templates

```html
{% include 'navbar.html' %}
```

This inserts the contents of `navbar.html` at this point. The included template has access to the current context.

#### `empty` Clause in `for` Tags

```html
{% for student in students %}
  <p>{{ student.full_name }}</p>
{% empty %}
  <p>No students found.</p>
{% endfor %}
```

The `empty` clause is displayed if the list is empty.

### Filters

Filters transform the value of a variable:

| Filter | Purpose | Example |
|--------|---------|---------|
| `lower` | Lowercase | `{{ "Hello"|lower }}` → `hello` |
| `upper` | Uppercase | `{{ "Hello"|upper }}` → `HELLO` |
| `title` | Title case | `{{ "hello world"|title }}` → `Hello World` |
| `capfirst` | Capitalize first letter | `{{ "hello"|capfirst }}` → `Hello` |
| `length` | Length | `{{ "hello"|length }}` → `5` |
| `truncatechars:n` | Truncate to n chars | `{{ "hello world"|truncatechars:5 }}` → `hello...` |
| `truncatewords:n` | Truncate to n words | `{{ "hello world"|truncatewords:1 }}` → `hello...` |
| `date` | Format date | `{{ student.registration_date|date:"M d, Y" }}` → `Jan 15, 2024` |
| `time` | Format time | `{{ student.registration_date|time:"H:i" }}` → `14:30` |
| `default` | Default if empty | `{{ student.middle_name|default:"N/A" }}` |
| `addslashes` | Add slashes | `{{ "Hello"|addslashes }}` → `\\Hello\\` |
| `striptags` | Remove HTML tags | `{{ "<b>Hello</b>"|striptags }}` → `Hello` |
| `escape` | HTML escape | `{{ "<b>Hello</b>"|escape }}` → `&lt;b&gt;Hello&lt;/b&gt;` |
| `safe` | Mark as safe | `{{ "<b>Hello</b>"|safe }}` → `<b>Hello</b>` |

**Chaining filters:**
```html
{{ student.full_name|upper|truncatechars:10 }}
```

### Date Formatting Filters

Our Student Registration System uses dates, so these filters are particularly useful:

```html
<!-- Default format -->
{{ student.registration_date }}  <!-- 2024-01-15 14:30:00 -->

<!-- Date formats -->
{{ student.registration_date|date:"Y-m-d" }}       <!-- 2024-01-15 -->
{{ student.registration_date|date:"M d, Y" }}      <!-- Jan 15, 2024 -->
{{ student.registration_date|date:"D, F j, Y" }}   <!-- Mon, January 15, 2024 -->
{{ student.registration_date|date:"d/m/Y" }}      <!-- 15/01/2024 -->

<!-- Time formats -->
{{ student.registration_date|time:"H:i" }}          <!-- 14:30 -->
{{ student.registration_date|time:"H:i:s" }}       <!-- 14:30:00 -->
{{ student.registration_date|time:"g:i A" }}       <!-- 2:30 PM -->

<!-- Date + Time -->
{{ student.registration_date|date:"Y-m-d H:i" }}  <!-- 2024-01-15 14:30 -->
```

## Template Inheritance

Template inheritance is one of Django's most powerful features. It allows you to:
- Define a base layout once
- Reuse it across all pages
- Override specific sections as needed

### Base Template (base.html)

```html
<!DOCTYPE html>
<html>
<head>
  <title>{% block title %}Student Registration System{% endblock %}</title>
  <style>
    {% block style %}{% endblock %}
  </style>
</head>
<body>
  <header>
    {% block header %}
      <h1>Student Registration System</h1>
      <nav>
        {% block nav %}
          <a href="{% url 'student_list' %}">All Students</a> |
          <a href="{% url 'register_student' %}">Register</a>
        {% endblock %}
      </nav>
    {% endblock %}
  </header>
  
  <main>
    {% block content %}{% endblock %}
  </main>
  
  <footer>
    {% block footer %}
      <p>&copy; 2024 Student Registration System</p>
    {% endblock %}
  </footer>
</body>
</html>
```

### Child Template (student_list.html)

```html
{% extends 'base.html' %}

{% block title %}All Students{% endblock %}

{% block content %}
  <h2>All Students</h2>
  <table>
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

**How it works:**
1. `{% extends 'base.html' %}` tells Django this template inherits from `base.html`
2. Any `{% block %}` tags in the child override the corresponding blocks in the parent
3. Blocks not defined in the child use the parent's default content

### Multiple Levels of Inheritance

You can have multiple levels:

```html
<!-- base.html -->
{% block content %}{% endblock %}

<!-- students/base.html -->
{% extends 'base.html' %}
{% block content %}
  <h1>Students</h1>
  {% block student_content %}{% endblock %}
{% endblock %}

<!-- student_list.html -->
{% extends 'students/base.html' %}
{% block student_content %}
  <!-- Student list table -->
{% endblock %}
```

## Template Context

The **context** is a dictionary containing all the variables available to the template. It's passed from the view to the template:

```python
# views.py
def student_list(request):
    students = Student.objects.all().order_by('-registration_date')
    context = {
        'students': students,
        'title': 'All Students',
        'count': students.count(),
    }
    return render(request, 'student_list.html', context)
```

In the template, you can access any variable in the context:

```html
<h1>{{ title }}</h1>
<p>Total: {{ count }} students</p>
```

### Context Processors

Django automatically adds certain variables to every template's context via **context processors**. These are defined in `TEMPLATES` in `settings.py`:

| Variable | Description |
|----------|-------------|
| `request` | The current HttpRequest object |
| `user` | The currently authenticated user (if any) |
| `perms` | Permissions for the current user |
| `messages` | Messages framework for notifications |

**Example:**
```html
{% if user.is_authenticated %}
  <p>Welcome, {{ user.username }}!</p>
{% endif %}
```

## Template Loading

### How Django Finds Templates

Django's template loader looks for templates in this order:

1. **App directories**: Each app's `templates/` directory
2. **Project templates directory**: `TEMPLATES['DIRS']` in settings.py

By convention, each app has its own `templates/` directory:

```
students/
├── __init__.py
├── models.py
├── views.py
└── templates/
    ├── base.html
    ├── student_list.html
    └── register.html
```

Django automatically looks in `app/templates/` for each installed app.

### Template Names and Namespaces

**Problem:** If two apps have a template with the same name (e.g., both have `index.html`), Django might load the wrong one.

**Solution:** Use **template namespaces** by creating an app-specific subdirectory:

```
students/
└── templates/
    └── students/
        ├── student_list.html
        └── register.html
```

Then reference it with the namespace:

```python
# views.py
return render(request, 'students/student_list.html', context)
```

In our course, we use the simpler approach (flat templates directory) since we only have one app.

## Static Files in Templates

Static files (CSS, JavaScript, images) are handled separately from templates:

```html
<!-- Load static files -->
{% load static %}

<!-- Reference static files -->
<link rel="stylesheet" href="{% static 'css/style.css' %}">
<script src="{% static 'js/app.js' %}"></script>
<img src="{% static 'images/logo.png' %}" alt="Logo">
```

**How it works:**
1. `{% load static %}` loads the static template tag library
2. `{% static 'path/to/file' %}` generates the correct URL for the file

**Static files directory structure:**
```
students/
└── static/
    └── students/
        ├── css/
        │   └── style.css
        ├── js/
        │   └── app.js
        └── images/
            └── logo.png
```

## Template Best Practices

1. **Keep logic out of templates**: Templates should handle presentation, not business logic
2. **Use template tags for display logic**: `if`, `for` are fine; complex Python belongs in views
3. **Use meaningful block names**: `content`, `sidebar`, `header` are better than `block1`, `block2`
4. **Keep templates small**: Break large templates into smaller, reusable ones
5. **Use template inheritance**: Avoid repeating the same layout code
6. **Escape user content by default**: Always use `{{ variable|escape }}` for user-provided content (Django does this automatically for most cases)
7. **Use `safe` sparingly**: Only mark content as safe if you're certain it's safe

## Template Security

### Cross-Site Scripting (XSS) Protection

Django templates **automatically escape** variables by default to prevent XSS attacks:

```html
<!-- If student.name = "<script>alert('XSS')</script>" -->
{{ student.name }}  <!-- Outputs: &lt;script&gt;alert('XSS')&lt;/script&gt; -->

<!-- Only use safe if you're certain the content is safe -->
{{ student.name|safe }}  <!-- Outputs: <script>alert('XSS')</script> -->
```

**Rule:** Never mark user-provided content as `safe` unless you've explicitly sanitized it.

### Cross-Site Request Forgery (CSRF) Protection

Django provides built-in CSRF protection for forms. In templates:

```html
<form method="POST">
  {% csrf_token %}
  <!-- form fields -->
</form>
```

The `{% csrf_token %}` tag inserts a hidden input field with a CSRF token. Django validates this token on POST requests to prevent CSRF attacks.

## Custom Template Tags and Filters

While Django's built-in template language is powerful, sometimes you need more. You can create **custom template tags and filters**:

```python
# students/templatetags/student_tags.py
from django import template

register = template.Library()

@register.simple_tag
def greeting(user):
    return f"Hello, {user.first_name}!"

@register.filter
def grade_status(grade):
    if grade >= 8:
        return "High"
    elif grade >= 5:
        return "Medium"
    return "Low"
```

In templates:

```html
{% load student_tags %}

<p>{% greeting request.user %}</p>
<p>Status: {{ student.grade|grade_status }}</p>
```

**Template tag directory structure:**
```
students/
└── templatetags/
    ├── __init__.py
    └── student_tags.py
```

## Templates in Our Student Registration System

In Lesson 5, you created:

1. **base.html** - Shared layout with:
   - HTML skeleton
   - Title block
   - Header with navigation
   - Main content block
   - Footer

2. **student_list.html** - Displays all students:
   - Extends base.html
   - Shows student table
   - Uses `{% for %}` to loop through students
   - Uses `{% empty %}` for empty list
   - Uses `{{ student.full_name }}` to display computed property

3. **register.html** (Lesson 6) - Registration form:
   - Extends base.html
   - Contains HTML form
   - Uses `{% csrf_token %}` for security

This structure provides a consistent look and feel across all pages while allowing each page to have its own content.
