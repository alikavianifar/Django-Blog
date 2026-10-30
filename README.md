<div align="center">

# Django Blog

**A Django blog and lesson platform with custom authentication, SEO tooling and Docker support.**

![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)
![Django](https://img.shields.io/badge/Django-4.2_LTS-092E20?logo=django&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker&logoColor=white)
![Tests](https://img.shields.io/badge/tests-72-brightgreen)
![License](https://img.shields.io/badge/License-MIT-22c55e)

</div>

---

## Overview

Django Blog is a content platform built with Django 4.2 LTS. It combines a full blog workflow (scheduled publishing, categories, tags, moderated comments, search) with a login-protected lesson module that tracks each learner's progress. Authentication accepts either a username or an email address and is protected with CAPTCHA. All configuration comes from environment variables, with separate settings for development (SQLite) and production (PostgreSQL).

The project was built as a portfolio piece to demonstrate:

- A multi-settings Django architecture (base, development, production)
- Custom authentication, form handling and permission rules in the admin
- A Docker-based workflow with a multi-stage image
- Automated tests across all four apps

The bundled front end is themed as an English-learning website.

## Table of Contents

- [Screenshots](#screenshots)
- [Features](#features)
- [Routes](#routes)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Managing Content](#managing-content)
- [Environment Variables](#environment-variables)
- [Running Tests](#running-tests)
- [Deployment](#deployment)
- [Security](#security)
- [Tech Stack](#tech-stack)
- [License](#license)
- [Author](#author)

---

## Screenshots

<p align="center">
  <img src="static/img/Screenshot1.png" alt="Homepage" width="32%">
  <img src="static/img/Screenshot2.png" alt="Blog post list" width="32%">
  <img src="static/img/Screenshot3.png" alt="Post detail page" width="32%">
</p>

<p align="center"><em>Homepage · Blog list · Post detail</em></p>

---

## Features

### Blog

- **Scheduled publishing:** a post is visible only when its status is *published* and its publication date has passed
- **Rich-text content:** posts are written in the admin with the Summernote editor and can include a featured image
- **Categories and tags:** many-to-many categories and tags through `django-taggit`
- **Listing, filtering and search:** 4 posts per page, filtering by author, category or tag, and case-insensitive search across post content
- **Moderated comments:** comments stay hidden until approved in the admin, and the admins are notified by email. The post a comment belongs to is set on the server, never taken from form input
- **Restricted posts:** a post can require login; anonymous visitors are redirected to the login page and returned to the post afterwards
- **View counter:** counted whenever a reader is allowed to open a post
- **Sidebar widgets:** popular posts (top 3 by views), categories with post counts, recent posts, and site statistics, implemented as custom template tags
- **Post navigation:** links to the previous and next published post

### Lessons

- **Courses and lessons:** courses have a level (beginner, intermediate, advanced), a cover image and an active flag; lessons have an order, an estimated duration and HTML content
- **Progress tracking:** each user can mark lessons complete, and the course list shows the completion percentage of every course
- **Login required:** all lesson pages require authentication
- **Admin editing:** lessons are edited inline on the course page

### Accounts

- **Login with username or email:** custom authentication backend with CAPTCHA
- **Registration:** unique email validation, CAPTCHA and a welcome email
- **Password reset:** the full Django email flow with custom templates
- **Admin login:** CAPTCHA on the Django admin login as well (`django-multi-captcha-admin`)

### Site

- **Contact form:** messages are stored in the database and emailed to the configured recipients
- **Newsletter:** email subscription stored in the database
- **Maintenance mode:** switchable through an environment variable; staff and superusers bypass it
- **Custom error pages:** branded 400, 403, 404 and 500 pages
- **SEO:** `sitemap.xml` covering the static pages and all published posts, and `robots.txt` managed with `django-robots`

### Admin

- **Author-scoped permissions:** non-superusers can edit and delete only their own posts, are assigned as the author automatically, and cannot change view counts
- **Review workflow:** comment approval, plus searchable and filterable lists for posts, contact messages, subscribers and lesson progress

---

## Routes

| Path | Description |
|------|-------------|
| `/` | Homepage |
| `/contact`, `/newsletter` | Contact form and newsletter signup |
| `/blog/` | Post list |
| `/blog/post/<id>` | Post detail and comments |
| `/blog/author/<id>`, `/blog/category/<name>`, `/blog/tag/<name>` | Filtered lists |
| `/blog/search?s=<query>` | Search |
| `/Accounts/login/`, `/Accounts/logout/`, `/Accounts/signup/` | Authentication |
| `/password_reset/` | Password reset |
| `/Lessons/` | Course list (login required) |
| `/Lessons/course/<pk>/` | Course detail |
| `/Lessons/lesson/<pk>/` | Lesson detail |
| `/Lessons/lesson/<pk>/complete/` | Mark a lesson complete (POST) |
| `/sitemap.xml`, `/robots.txt` | SEO |
| `/admin/`, `/summernote/`, `/captcha/` | Admin, editor and CAPTCHA endpoints |

---

## Project Structure

```
Django-Blog/
├── CoreFolder/                  # Project configuration
│   ├── settings/
│   │   ├── base.py              # Shared settings
│   │   ├── development.py       # SQLite, console email, debug toolbar
│   │   └── production.py        # PostgreSQL, SMTP, optional SSL/HSTS
│   ├── urls.py                  # Root routing, sitemap, robots, error handlers
│   └── errors.py                # Custom 400/403/404/500 handlers
├── AppFolder/                   # Homepage, contact form, newsletter, static sitemap
├── BlogFolder/                  # Posts, categories, comments, post sitemap
│   └── templatetags/            # Sidebar and statistics template tags
├── AccountsFolder/              # Email-or-username backend, signup, login, logout
├── LessonsFolder/               # Courses, lessons and user progress
├── templates/                   # Base and app templates, error and 503 pages
├── static/                      # CSS, JavaScript, vendor libraries, images
├── Dockerfile                   # Multi-stage image served by Gunicorn
├── docker-compose.yml           # Django, PostgreSQL and optional smtp4dev
├── liara.json                   # Liara platform configuration
├── liara_nginx.conf             # Nginx headers and static/media rules for Liara
├── requirements.txt
└── .env.example                 # Environment variable template
```

---

## Getting Started

### Prerequisites

- Python 3.12 or later
- Docker and Docker Compose (optional, for the containerized setup)

### Option 1: Local development

Without any configuration, the project runs with the development settings: SQLite as the database and emails printed to the terminal.

```bash
# 1. Clone the repository
git clone https://github.com/alikavianifar/Django-Blog.git
cd Django-Blog

# 2. Create and activate a virtual environment
python -m venv venv
source venv/bin/activate        # macOS / Linux
# venv\Scripts\activate         # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. (Optional) Create a .env file to override the defaults
cp .env.example .env

# 5. Apply migrations and create an admin user
python manage.py migrate
python manage.py createsuperuser

# 6. Start the development server
python manage.py runserver
```

Open <http://127.0.0.1:8000>. The admin is at <http://127.0.0.1:8000/admin/>.

### Option 2: Docker

The compose file starts the application with the production settings and a PostgreSQL 16 database.

```bash
# 1. Clone the repository
git clone https://github.com/alikavianifar/Django-Blog.git
cd Django-Blog

# 2. Create the environment file and edit it
cp .env.example .env
# Set SECRET_KEY, DEBUG=False and PGDB_PASS, and make sure the database
# variables match the db service in docker-compose.yml

# 3. Build and start the stack
docker compose up --build

# 4. In a new terminal, create an admin user
docker compose exec web python manage.py createsuperuser
```

On startup the `web` container applies migrations, collects static files and starts Gunicorn with three workers. Open <http://localhost:8000>.

To add the development mail server (smtp4dev), start the stack with the `dev` profile and point the email variables at it (`EMAIL_HOST=mailserver`, `EMAIL_PORT=25`, `EMAIL_USE_TLS=False`):

```bash
docker compose --profile dev up --build
# smtp4dev web interface: http://localhost:5000
```

---

## Managing Content

1. Sign in to `/admin/` with the superuser account.
2. Create one or more **categories**, then add a **post**. A post appears on the site only when its **status** is checked and its **published date** is in the past.
3. Review incoming **comments** and tick **approved** to publish them.
4. For the lesson module, create a **course** and add its **lessons** inline. Lessons are ordered by their `order` value.
5. Additional staff accounts can write posts and manage only their own.

---

## Environment Variables

Configuration is read with `python-decouple` from a `.env` file or from the process environment. The `.env` file is ignored by Git and must never be committed. Values marked *production* are read only by `CoreFolder.settings.production`.

| Variable | Default | Description |
|----------|---------|-------------|
| `DJANGO_SETTINGS_MODULE` | development settings | Set to `CoreFolder.settings.production` for production (the compose file does this) |
| `SECRET_KEY` | insecure development key | Django secret key; **always set it in production** |
| `DEBUG` | `True` | Debug mode; **set to `False` in production** |
| `ALLOWED_HOSTS` | `localhost,127.0.0.1` | Comma-separated host names |
| `SITE_ID` | `2` | Django Sites framework ID |
| `SITE_URL` | `http://127.0.0.1:8000` | Base URL used in notification emails |
| `TIME_ZONE` | `UTC` | Time zone |
| `MAINTENANCE_MODE` | `False` | Show the maintenance page to everyone except staff and superusers |
| `PGDB_ENGINE` | `django.db.backends.postgresql` | Database engine (*production*) |
| `PGDB_NAME` | `postgres` | Database name (*production*) |
| `PGDB_USER` | `postgres` | Database user (*production*) |
| `PGDB_PASS` | empty | Database password (*production*) |
| `PGDB_HOST` | `localhost` | Database host; use `db` with Docker Compose (*production*) |
| `PGDB_PORT` | `5432` | Database port (*production*) |
| `EMAIL_BACKEND` | console (dev) / SMTP (production) | Email backend |
| `EMAIL_HOST` | `localhost` (dev) / `smtp.gmail.com` (production) | SMTP host |
| `EMAIL_PORT` | `25` (dev) / `587` (production) | SMTP port |
| `EMAIL_USE_TLS` | `True` (*production*) | Use TLS for SMTP |
| `EMAIL_HOST_USER` | empty | SMTP username |
| `EMAIL_HOST_PASSWORD` | empty | SMTP password or app password |
| `DEFAULT_FROM_EMAIL` | empty | Default sender address (*production*) |
| `CONTACT_FROM_EMAIL` | `noreply@example.com` | Sender of contact-form notifications |
| `CONTACT_RECIPIENTS` | `admin@example.com` | Comma-separated recipients of contact-form notifications |
| `BLOG_FROM_EMAIL` | `noreply@example.com` | Sender of new-comment notifications |
| `BLOG_RECIPIENTS` | `admin@example.com` | Comma-separated recipients of new-comment notifications |
| `SIGNUP_FROM_EMAIL` | `noreply@example.com` | Sender of the welcome email |
| `USE_SSL_CONFIG` | `False` | Enable HTTPS redirect, secure cookies and HSTS (*production*) |

---

## Running Tests

The project has **72 tests** across its four apps:

| App | Tests |
|-----|-------|
| `AppFolder` | 18 |
| `BlogFolder` | 22 |
| `AccountsFolder` | 14 |
| `LessonsFolder` | 18 |

```bash
# Run all tests
python manage.py test

# Run the tests of a single app
python manage.py test BlogFolder

# Verbose output
python manage.py test -v 2

# Inside the Docker container
docker compose exec web python manage.py test
```

---

## Deployment

- **Application server:** the image runs Gunicorn (`CoreFolder.wsgi:application`, three workers).
- **Static and media files:** Gunicorn does not serve them. Put a reverse proxy such as Nginx in front, serving `/static` from `staticfiles/` and `/media` from `media/`. The compose file stores both in named volumes.
- **HTTPS:** terminate TLS at the proxy and set `USE_SSL_CONFIG=True` (see [Security](#security)).
- **Liara:** `liara.json` and `liara_nginx.conf` contain the configuration used to deploy the project on the Liara platform (Django platform, Python 3.12, persistent disks for static and media files, security headers and static/media rules in Nginx).

### Production checklist

- [ ] `SECRET_KEY` set to a strong, unique value (for example `python -c "import secrets; print(secrets.token_urlsafe(50))"`)
- [ ] `DEBUG=False`
- [ ] `ALLOWED_HOSTS` and `SITE_URL` set to the real domain
- [ ] `PGDB_PASS` and the SMTP credentials set
- [ ] `USE_SSL_CONFIG=True` behind an HTTPS proxy

---

## Security

- **No secrets in source code:** sensitive values come from the environment through `python-decouple`
- **CSRF and clickjacking protection:** Django's CSRF middleware on all forms and the `X-Frame-Options` middleware
- **CAPTCHA:** on login, registration and the admin login
- **Password validation:** Django's four built-in validators
- **Scoped admin permissions:** authors can change only their own posts
- **Server-side comment assignment:** a comment's post is never read from user input
- **HTTPS hardening:** with `USE_SSL_CONFIG=True` the production settings enable the HTTPS redirect, secure session and CSRF cookies, HSTS (one year, subdomains, preload), `nosniff`, `SAMEORIGIN` framing, a strict referrer policy and proxy header handling
- **Debug tooling:** the debug toolbar is installed only with the development settings
- **Defaults to override:** without environment variables the project uses an insecure development `SECRET_KEY` and `DEBUG=True`, so always set both before deploying

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Language | Python 3.12 |
| Framework | Django 4.2 LTS |
| Database | PostgreSQL 16 (production), SQLite (development) |
| Rich text | django-summernote |
| Tags | django-taggit |
| Authentication | Custom email/username backend, django-simple-captcha, django-multi-captcha-admin |
| SEO | Django sitemaps, django-robots |
| Operations | django-maintenance-mode, python-decouple, Gunicorn |
| Front end | Django templates, Bootstrap 5.3, AOS, Swiper, GLightbox, Isotope |
| Containers | Docker (multi-stage), Docker Compose |

---

## License

Released under the [MIT License](LICENSE).

---

## Author

**Ali Kaviani Far**
Web & Backend Developer

[GitHub](https://github.com/alikavianifar) · [LinkedIn](https://www.linkedin.com/in/alikavianifar/) · [alikavianifar@proton.me](mailto:alikavianifar@proton.me)
