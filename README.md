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


# 🐳 Docker Compose File Explained

---

## 🤔 What is Docker Compose?

Docker Compose is a tool that lets you define and run **multiple containers at once**.
You write a single file — `docker-compose.yml` — and start everything with one command.

```bash
docker-compose up --build
```

---

## 🏗️ Our Project has 3 Services

```
🌐 [Nginx]       → Gateway / Entry Point  (port 80)
⚙️  [Django App]  → Application Server     (port 8000)
🗄️  [MySQL DB]    → Database Server

All 3 communicate on the same network: notes-app-nw
```

---

## 📄 Full File

```yaml
version: "3.8"

services:
  nginx:
    build: ./nginx
    image: nginx
    container_name: "nginx_cont"
    ports:
      - "80:80"
    restart: always
    depends_on:
      - django_app
    networks:
      - notes-app-nw

  django_app:
    build:
      context: .
    image: django_app
    container_name: "django_cont"
    ports:
      - "8000:8000"
    command: sh -c "python manage.py migrate --noinput && gunicorn notesapp.wsgi --bind 0.0.0.0:8000"
    env_file:
      - ".env"
    depends_on:
      - db
    restart: always
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:8000/admin || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    networks:
      - notes-app-nw

  db:
    image: mysql
    container_name: "db_cont"
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=test_db
    volumes:
      - ./data/mysql/db:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-uroot", "-proot"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 60s
    networks:
      - notes-app-nw

networks:
  notes-app-nw:
```

---

## 🌐 Service 1 — Nginx

```yaml
nginx:
  build: ./nginx
```
> 🔨 Build the Docker image using the `Dockerfile` inside the `./nginx` folder

```yaml
  image: nginx
```
> 🏷️ Name the built image `nginx`

```yaml
  container_name: "nginx_cont"
```
> 📦 The running container will be named `nginx_cont`

```yaml
  ports:
    - "80:80"
```
> 🔌 Map your machine's port 80 to the container's port 80
> Accessing `http://localhost` in the browser hits this container

```yaml
  restart: always
```
> 🔄 If the container crashes, it will automatically restart

```yaml
  depends_on:
    - django_app
```
> ⏳ Start `django_app` first, then start nginx

```yaml
  networks:
    - notes-app-nw
```
> 🔗 Join this network so nginx can communicate with django internally

---

## ⚙️ Service 2 — Django App

```yaml
django_app:
  build:
    context: .
```
> 🔨 Build the Docker image using the `Dockerfile` in the root folder (`.`)

```yaml
  container_name: "django_cont"
```
> 📦 The running container will be named `django_cont`

```yaml
  ports:
    - "8000:8000"
```
> 🔌 Map your machine's port 8000 to the container's port 8000
> Direct access: `http://localhost:8000`

```yaml
  command: sh -c "python manage.py migrate --noinput && gunicorn notesapp.wsgi --bind 0.0.0.0:8000"
```
> ▶️ When the container starts, run these 2 steps:
> 1. `python manage.py migrate` → Create/update database tables
> 2. `gunicorn ... --bind 0.0.0.0:8000` → Start the app server on port 8000

```yaml
  env_file:
    - ".env"
```
> 🔐 Load environment variables from the `.env` file (DB_NAME, DB_USER, DB_PASSWORD, etc.)

```yaml
  depends_on:
    - db
```
> ⏳ Start `db` (MySQL) first, then start django

```yaml
  healthcheck:
    test: ["CMD-SHELL", "curl -f http://localhost:8000/admin || exit 1"]
    interval: 10s
    timeout: 5s
    retries: 5
    start_period: 30s
```
> 🏥 Periodically check if Django is alive and responding
> - `interval: 10s` → Check every 10 seconds
> - `timeout: 5s` → If no response in 5 seconds, mark as failed
> - `retries: 5` → Try 5 times before marking unhealthy
> - `start_period: 30s` → Ignore failures for the first 30 seconds (startup time)

---

## 🗄️ Service 3 — MySQL Database

```yaml
db:
  image: mysql
```
> 📥 Use the official MySQL image from DockerHub (no need to build)

```yaml
  container_name: "db_cont"
```
> 📦 The running container will be named `db_cont`

```yaml
  environment:
    - MYSQL_ROOT_PASSWORD=root
    - MYSQL_DATABASE=test_db
```
> 🔐 When MySQL starts:
> - Set root password to `root`
> - Automatically create a database named `test_db`

```yaml
  volumes:
    - ./data/mysql/db:/var/lib/mysql
```
> 💾 Save MySQL data to `./data/mysql/db` on your machine
> Even if the container is stopped or deleted, data will not be lost

```yaml
  healthcheck:
    test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-uroot", "-proot"]
    interval: 10s
    timeout: 5s
    retries: 5
    start_period: 60s
```
> 🏥 Check every 10 seconds if MySQL is ready to accept connections
> `start_period: 60s` → MySQL takes time to initialize, so wait 60 seconds before checking

---

## 🔗 Networks

```yaml
networks:
  notes-app-nw:
```
> 🌐 Create a custom network named `notes-app-nw`
> All 3 services are on this network so they can talk to each other internally
> Nginx → Django → MySQL all communicate through this network

---

## 🚀 Service Start Order

```
Step 1 🗄️  → db (MySQL)     Database must be ready first
Step 2 ⚙️  → django_app     Run migrations, then start Gunicorn server
Step 3 🌐  → nginx          Open the gateway last
```

---

## 🔌 Ports Mapping

```
Browser → localhost:80  →  nginx_cont:80  →  django_cont:8000
                                (reverse proxy)
```

| Service    | Machine Port | Container Port | Access URL              |
|------------|--------------|----------------|-------------------------|
| 🌐 Nginx   | 80           | 80             | http://localhost        |
| ⚙️ Django  | 8000         | 8000           | http://localhost:8000   |
| 🗄️ MySQL   | -            | 3306           | Internal only           |

---

## 📚 Key Concepts — Quick Reference

| Keyword             | Meaning                                          |
|---------------------|--------------------------------------------------|
| 🔨 `build`          | Where to find the Dockerfile to build the image  |
| 🏷️ `image`          | Name to give the built image                     |
| 📦 `container_name` | Name of the running container                    |
| 🔌 `ports`          | `machine:container` port mapping                 |
| 🔄 `restart: always`| Auto-restart the container if it crashes         |
| ⏳ `depends_on`     | Start this service only after another one        |
| ▶️ `command`        | Command to run when the container starts         |
| 🔐 `env_file`       | Load environment variables from a file           |
| 🏥 `healthcheck`    | Periodically check if the service is healthy     |
| 💾 `volumes`        | Persist data on your machine permanently         |
| 🔗 `networks`       | Allow services to communicate with each other    |
| 🌍 `environment`    | Set environment variables directly in the file   |

