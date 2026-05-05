# Django Notes App

A full-stack Notes application built with **React** (frontend) and **Django REST Framework** (backend), containerized with Docker and served via Nginx reverse proxy. Supports CI/CD through Jenkins.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Tech Stack](#tech-stack)
3. [Project Structure](#project-structure)
4. [Architecture](#architecture)
5. [API Endpoints](#api-endpoints)
6. [Prerequisites](#prerequisites)
7. [Environment Variables](#environment-variables)
8. [Running Locally (Without Docker)](#running-locally-without-docker)
9. [Running with Docker](#running-with-docker)
10. [Running with Docker Compose](#running-with-docker-compose)
11. [Nginx Configuration](#nginx-configuration)
12. [CI/CD with Jenkins](#cicd-with-jenkins)

---

## Project Overview

This app allows users to create, view, update, and delete notes. The React frontend communicates with the Django REST API backend. The entire stack is containerized and orchestrated using Docker Compose, with Nginx acting as a reverse proxy in front of the Django/Gunicorn server.

---

## Tech Stack

| Layer      | Technology                          |
|------------|-------------------------------------|
| Frontend   | React.js                            |
| Backend    | Django 4.1.5, Django REST Framework |
| Database   | MySQL                               |
| Web Server | Nginx (reverse proxy)               |
| App Server | Gunicorn                            |
| Container  | Docker, Docker Compose              |
| CI/CD      | Jenkins                             |

---

## Project Structure

```
django-notes-app/
├── api/                    # Django REST API app
│   ├── models.py           # Note model
│   ├── serializers.py      # DRF serializer
│   ├── views.py            # API views (CRUD)
│   └── urls.py             # API URL routes
├── notesapp/               # Django project config
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── mynotes/                # React frontend
│   ├── src/
│   │   ├── components/     # AddButton, Header, ListItem
│   │   ├── pages/          # NotesListPage, NotePage
│   │   └── App.js
│   └── Dockerfile          # Frontend Docker build
├── nginx/
│   ├── default.conf        # Nginx reverse proxy config
│   └── Dockerfile          # Nginx Docker build
├── Dockerfile              # Backend Docker build
├── docker-compose.yml      # Multi-container orchestration
├── Jenkinsfile             # CI/CD pipeline
├── requirements.txt        # Python dependencies
└── .env                    # Environment variables
```

---

## Architecture

```
Browser
   │
   ▼
Nginx (port 80)
   │  reverse proxy
   ▼
Django + Gunicorn (port 8000)
   │
   ▼
MySQL Database
```

- Nginx listens on port 80 and forwards all requests to the Django container on port 8000.
- Django serves both the React static build and the REST API.
- MySQL is used as the production database.
- All three services communicate over a shared Docker network (`notes-app-nw`).

---

## API Endpoints

| Method | Endpoint              | Description            |
|--------|-----------------------|------------------------|
| GET    | `/api/`               | List all available routes |
| GET    | `/api/notes/`         | Get all notes          |
| GET    | `/api/notes/<id>/`    | Get a single note      |
| POST   | `/api/notes/create/`  | Create a new note      |
| PUT    | `/api/notes/<id>/update/` | Update a note      |
| DELETE | `/api/notes/<id>/delete/` | Delete a note      |

---

## Prerequisites

- Python 3.9+
- Node.js & npm
- Docker & Docker Compose
- MySQL (if running without Docker)

---

## Environment Variables

Create a `.env` file in the project root:

```env
DB_NAME=test_db
DB_USER=root
DB_PASSWORD=root
DB_HOST=db
DB_PORT=3306
```

---

## Running Locally (Without Docker)

**Backend:**

```bash
pip install -r requirements.txt
python manage.py migrate
python manage.py runserver
```

**Frontend:**

```bash
cd mynotes
npm install
npm start
```

---

## Running with Docker

Build and run only the backend:

```bash
docker build -t notes-app .
docker run -d -p 8000:8000 notes-app:latest
```

---

## Running with Docker Compose

This starts all three services — Nginx, Django, and MySQL — together:

```bash
docker-compose up --build
```

- App is accessible at: `http://localhost`
- Django admin at: `http://localhost/admin`

To stop:

```bash
docker-compose down
```

---

## Nginx Configuration

Nginx is configured as a reverse proxy (`nginx/default.conf`):

- Listens on port **80**
- Forwards all traffic to `django_cont:8000`
- Passes real client IP and forwarded headers

---

## CI/CD with Jenkins

The `Jenkinsfile` defines a 4-stage pipeline using a shared Jenkins library:

| Stage           | Description                                      |
|-----------------|--------------------------------------------------|
| Code Clone      | Clones the repo from GitHub (`main` branch)      |
| Code Build      | Builds the Docker image (`notes-app:latest`)     |
| Push to DockerHub | Pushes the image to DockerHub                  |
| Deploy          | Deploys the app on the target server             |

The pipeline runs on a Jenkins agent labeled `dev-server`.
