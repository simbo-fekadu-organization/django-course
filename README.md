# Django Course — Student Registration System

A hands-on Django course taught by building one real project: a **Student
Registration System**. Each lesson is a self-contained folder with its own
`README.md` — read it top to bottom, run every command as you go, and check
the "Checkpoint" before moving to the next lesson.

**What is Django?** Django is a high-level Python web framework that encourages rapid development and clean, pragmatic design. It handles much of the hassle of web development, so you can focus on writing your app without reinventing the wheel.

## Course Overview
By the end, you'll have built:
- A database-backed `Student` model
- An admin dashboard for managing records
- Full CRUD operations from the Python shell
- Full CRUD functionality through real HTML pages in the browser

## Prerequisites
- Python 3.10+ installed
- Basic Python (variables, functions, classes)
- A terminal you're comfortable typing commands into
- pip/package management basics
- A code editor (VS Code, PyCharm, etc.) — optional but helpful

**Note:** This course assumes Django 4.x or 5.x. All commands are for Unix-like systems. Windows users should use WSL or adjust commands accordingly.

## Lessons

| # | Lesson | What you'll build | Est. Time |
|---|---|---|---|
| 00 | [Setup](00-setup/README.md) | A running Django project and app | 20-30 min |
| 01 | [Models & Database](01-models-database/README.md) | The `Student` model and its database table | 30-40 min |
| 02 | [Admin Panel](02-admin-panel/README.md) | A working `/admin/` dashboard | 20-30 min |
| 03 | [ORM & CRUD in the Shell](03-orm-crud-shell/README.md) | Create/Read/Update/Delete via Python (no browser) | 30-45 min |
| 04 | [Views & URLs](04-views-urls/README.md) | Your first browser-served page | 30-40 min |
| 05 | [Templates](05-templates/README.md) | A real HTML page listing all students | 30-45 min |
| 06 | [CRUD in the Browser](06-crud-in-browser/README.md) | A register form and delete link, fully working | 45-60 min |

## Capstone
After completing all lessons, extend the system with features like:
- A `Course` model with many-to-many enrollment tracking
- Student search and filtering
- User authentication for the registration form

## How to use this course
1. Work through the lessons in order — each one assumes the previous one is done.
2. Every lesson has a **Checkpoint** — don't move on until you see that result.
3. Every lesson has **Common mistakes** — read these even if your code worked, so you recognize the errors when you hit them later.
4. Every lesson ends with an **Exercise** — do it before moving on. No solutions are provided on purpose.
5. Commit your work at each checkpoint so you can revert if needed.

**Tip:** Use `git status` frequently to track your changes.

If you fall badly behind or your project breaks in a way you can't fix, ask your instructor for the checkpoint tag for the last completed lesson (e.g. `lesson-03-done`) and reset with:
```bash
git fetch --tags
git checkout lesson-03-done
```

## Troubleshooting
- **Port already in use:** Run `lsof -i :8000` (or the port in error) and kill the process, or use a different port with `python manage.py runserver 8080`
- **Migration errors:** Delete the `db.sqlite3` file and `migrations/` folders (except `__init__.py`), then run `python manage.py makemigrations` and `python manage.py migrate`
- **Module not found:** Ensure you're in the correct virtual environment and have installed requirements with `pip install -r requirements.txt`

## Glossary
- **ORM (Object-Relational Mapper):** Django's system for interacting with your database using Python code instead of SQL
- **CRUD:** Create, Read, Update, Delete — the four basic database operations
- **Views:** Python functions that handle web requests and return web responses
- **Templates:** HTML files with Django template syntax for dynamic content
- **URLs:** The mapping between web addresses and your Django views