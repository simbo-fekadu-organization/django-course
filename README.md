# Django Course — Student Registration System

A hands-on Django course taught by building one real project: a **Student
Registration System**. Each lesson is a self-contained folder with its own
`README.md` — read it top to bottom, run every command as you go, and check
the "Checkpoint" before moving to the next lesson.

By the end, you'll have built: a database-backed model, an admin dashboard,
full CRUD from the Python shell, and full CRUD through real HTML pages in
the browser.

## Prerequisites
- Python 3.10+ installed
- Basic Python (variables, functions, classes)
- A terminal you're comfortable typing commands into

## Lessons

| # | Lesson | What you'll build |
|---|---|---|
| 00 | [Setup](00-setup/README.md) | A running Django project and app |
| 01 | [Models & Database](01-models-database/README.md) | The `Student` model and its database table |
| 02 | [Admin Panel](02-admin-panel/README.md) | A working `/admin/` dashboard |
| 03 | [ORM & CRUD in the Shell](03-orm-crud-shell/README.md) | Create/Read/Update/Delete via Python |
| 04 | [Views & URLs](04-views-urls/README.md) | Your first browser-served page |
| 05 | [Templates](05-templates/README.md) | A real HTML page listing all students |
| 06 | [CRUD in the Browser](06-crud-in-browser/README.md) | A register form and delete link, fully working |

## How to use this course
1. Work through the lessons in order — each one assumes the previous one is done.
2. Every lesson has a **Checkpoint** — don't move on until you see that result.
3. Every lesson has **Common mistakes** — read these even if your code worked,
   so you recognize the errors when you hit them later.
4. Every lesson ends with an **Exercise** — do it before moving on. No
   solutions are provided on purpose.

If you fall badly behind or your project breaks in a way you can't fix, ask
your instructor for the checkpoint tag for the last completed lesson
(e.g. `lesson-03-done`) and reset with:
```bash
git fetch --tags
git checkout lesson-03-done
```