# 🐳 docker-project

> **Deploy a Python web application with a PostgreSQL database** using Docker and Docker Compose — a hands-on containerization project with Flask, psycopg, and pgAdmin.

---

## 📋 Description

**docker-project** is a fully containerized web application that demonstrates how to orchestrate multiple Docker services with **Docker Compose**. It connects a **Flask** Python web server to a **PostgreSQL** database pre-loaded with SQL data, and includes a **pgAdmin** interface for database management.

The project was built step by step — from creating individual Dockerfiles to wiring everything together via Docker Compose — making it an ideal reference for learning container-based deployment.

> **Created:** 21/02/2024 · **Author:** Yannis DUVIGNAU

---

## 🗂️ Project Structure

```
docker-project/
│
├── sql/
│   ├── Dockerfile          # Builds the custom PostgreSQL image
│   └── world.sql           # Initial SQL data loaded into the database on startup
│
├── web/
│   ├── app.py              # Flask application — connects to DB and serves data
│   ├── Dockerfile          # Builds the Python/Flask image (python:3.11-slim)
│   └── templates/
│       └── index.html      # Jinja2 HTML template — displays data from the database
│
├── docker-compose.yml      # Orchestrates all services: web, database, pgAdmin
└── README.md               # This file
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Web Framework | Flask (Python) |
| Database | PostgreSQL |
| DB Admin UI | pgAdmin 4 (`dpage/pgadmin4`) |
| Containerization | Docker |
| Orchestration | Docker Compose |
| DB Connector | psycopg (Python → PostgreSQL) |
| Templating | Jinja2 / HTML |
| Base Image (web) | `python:3.11-slim` |
| Base Image (db) | `postgres` |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Docker Network                      │
│                                                      │
│  ┌────────────────┐      ┌──────────────────────┐   │
│  │   web service  │      │   database service   │   │
│  │                │      │                      │   │
│  │  Flask app.py  │─────▶│  PostgreSQL + world  │   │
│  │  port: 5000    │      │  .sql initialized    │   │
│  │                │      │  port: 5432          │   │
│  └────────────────┘      └──────────────────────┘   │
│                                    ▲                 │
│  ┌─────────────────────────────────┘                 │
│  │   pgAdmin service                                 │
│  │   dpage/pgadmin4                                  │
│  │   port: 5050                                      │
│  └───────────────────────────────────────────────── │
└─────────────────────────────────────────────────────┘

depends_on order: database → web
                  database → pgAdmin
```

---

## 🚀 Getting Started

### Prerequisites

- [Docker](https://www.docker.com/get-started) installed
- [Docker Compose](https://docs.docker.com/compose/install/) (v2+)

---

### Quick Start — Run everything with Docker Compose

```bash
# Clone the repository
git clone https://github.com/yannisduvignau/docker-project.git
cd docker-project

# Build and start all services
docker-compose up -d
```

Once running, access the services at:

| Service | URL |
|---------|-----|
| 🌐 Flask Web App | http://localhost:5000 |
| 🗄️ pgAdmin UI | http://localhost:5050 |

---

### Step-by-Step Setup

#### Step 1 — Build and test the web container

```bash
cd web/
docker build -t projet_docker .
docker run -p 5000:5000 -d --name serverWeb projet_docker
```

#### Step 2 — Build and test the PostgreSQL container

```bash
cd sql/
docker build -t sql_image .
docker run -p 5432:5432 -d --name postgresdb_container sql_image

# Verify the database was initialized
docker exec -it postgresdb_container /bin/bash
  psql -U docker -d world
  \dt                   # list tables
  SELECT * FROM CD;     # check sample data
```

#### Step 3 — Orchestrate with Docker Compose

```bash
# From the project root
docker-compose up -d
```

The `docker-compose.yml` defines three services with the following configuration:

| Service | Image | Port | Notes |
|---------|-------|------|-------|
| `web` | Built from `web/Dockerfile` | 5000 | `depends_on: database` |
| `database` | Built from `sql/Dockerfile` | 5432 | Loads `world.sql` on init |
| `pgadmin` | `dpage/pgadmin4` | 5050 | `depends_on: database` |

#### Step 4 — Flask ↔ PostgreSQL connection

The `app.py` file uses:
- **Flask** — Python web framework and routing
- **render_template** — passes DB data to `index.html`
- **psycopg** — connects Python to the PostgreSQL database

```python
# Simplified connection flow in app.py
import flask
import psycopg

# Create Flask app
app = Flask(__name__)

# Connect to PostgreSQL
conn = psycopg.connect(...)

# Query data and render template
@app.route('/')
def index():
    data = ...  # fetch from DB
    return render_template('index.html', data=data)
```

---

## ⚙️ Docker Compose Services

```yaml
services:

  database:
    build: ./sql           # Custom Postgres image with world.sql
    ports: ["5432:5432"]
    environment:
      POSTGRES_DB: world
      POSTGRES_USER: docker
      POSTGRES_PASSWORD: docker

  web:
    build: ./web           # Python:3.11-slim + Flask
    ports: ["5000:5000"]
    depends_on: [database]

  pgadmin:
    image: dpage/pgadmin4
    ports: ["5050:5050"]
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: admin
    depends_on: [database]
```

---

## 🧠 Concepts Covered

| Topic | Description |
|-------|-------------|
| Dockerfile | Writing custom images for Python and PostgreSQL |
| Multi-stage services | Separate containers for app, DB, and admin UI |
| Docker Compose | Orchestrating multi-container apps with a single file |
| `depends_on` | Controlling service startup order |
| Volumes & networks | Persisting data and isolating container communication |
| psycopg | Connecting Python to PostgreSQL |
| Jinja2 templating | Rendering server-side HTML with dynamic DB data |
| pgAdmin | Administering PostgreSQL through a web UI |

---

## 🛑 Stop and Clean Up

```bash
# Stop all containers
docker-compose down

# Stop and remove volumes
docker-compose down -v

# Remove built images
docker rmi projet_docker sql_image
```

---

## 🤝 Contributing

1. Fork the project
2. Create your branch (`git checkout -b feature/add-redis-cache`)
3. Commit your changes (`git commit -m 'Add Redis caching layer'`)
4. Push to the branch (`git push origin feature/add-redis-cache`)
5. Open a Pull Request

---

## 👤 Author

**Yannis Duvignau** — [GitHub](https://github.com/yannisduvignau)

---

## 📚 Resources

- 🐳 [Docker Documentation](https://docs.docker.com/)
- 🐙 [Docker Compose Reference](https://docs.docker.com/compose/)
- 🐍 [Flask Documentation](https://flask.palletsprojects.com/)
- 🐘 [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- 🔌 [psycopg Documentation](https://www.psycopg.org/psycopg3/docs/)
- 🖥️ [pgAdmin 4 Documentation](https://www.pgadmin.org/docs/)

---

## 📄 License

This project is distributed under an open license. See the `LICENSE` file for more details.
