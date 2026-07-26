# conduit-application

A containerized setup of the RealWorld "Conduit" application, consisting of
a Django REST backend, an Angular frontend, and a PostgreSQL database — all
orchestrated via Docker Compose.

## Table of Contents

- [Description](#description)
- [Quickstart](#quickstart)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
- [Usage](#usage)
  - [1. Environment Variables](#1-environment-variables)
  - [2. Networking](#2-networking)
  - [3. Persistence](#3-persistence)
  - [4. Restart Behavior](#4-restart-behavior)
  - [5. Viewing and Saving Logs](#5-viewing-and-saving-logs)
  - [6. Stopping the Stack](#6-stopping-the-stack)
- [Testing](#testing)

## Description

This repository contains everything needed to run the Conduit application
("RealWorld" example app — a Medium.com-style blogging platform) using
Docker Compose. It consists of three services:

- **`frontend`** – the Angular application, built via a multi-stage Docker
  build and served as static files through `nginx`.
- **`backend`** – the Django REST API, served via `gunicorn` (a production
  WSGI server, not the Django development server).
- **`db`** – a PostgreSQL database that stores all application data.

All three services run in a shared Docker network (`conduit_network`), so
the backend can reach the database by its service name (`db`), and the
frontend can reach the backend through the network as well. Database
content is persisted through a named Docker volume, so data survives
container restarts and recreations.

### Repository contents

| Path                                | Purpose                                                      |
|--------------------------------------|-----------------------------------------------------------------|
| `docker-compose.yaml`                | Defines and configures the `frontend`, `backend`, and `db` services |
| `example.env`                        | Template with default values for all environment variables       |
| `.gitignore`                          | Excludes secrets and irrelevant files from git                     |
| `.dockerignore`                       | Excludes irrelevant files (e.g. `node_modules`) from Docker builds  |
| `backend/conduit-backend/Dockerfile` | Builds the Django backend image                                    |
| `frontend/conduit-frontend/Dockerfile` | Multi-stage build for the Angular frontend, served via nginx     |
| `README.md`                           | This documentation                                                   |

## Quickstart

### Prerequisites

In order to run this project you need the following:

- **Git** – to clone the repository
- An **OCI-compliant container engine with Compose support** (e.g. Docker
  Engine + Compose plugin)
- A **host or VM with a public IP** and **port `8282` open** (frontend)
- **Terminal/SSH access** to that host

### Installation

To get started, follow these steps:

1. Clone the repository:
   ```bash
   git clone git@github.com:FabianRitzmann/conduit-application.git
   cd conduit-application
   ```

2. Create your own environment file from the template:
   ```bash
   cp example.env .env
   ```

3. Open `.env` and set `DJANGO_SECRET_KEY` and `POSTGRES_PASSWORD` to strong,
   unique values.

> [!TIP]
> You can generate a random, secure value with:

 ```bash
 openssl rand -base64 32
```

4. Start the stack:
   ```bash
   docker compose up -d
   ```


5. Open `http://<your-host-ip>:8282` in your browser to use the
   application

## Usage

In this section you can read about the project configuration in more detail.

---

### 1. Environment Variables

All configuration is controlled through environment variables, defined in
`example.env` and loaded via your local `.env` file.

| Variable               | Required | Default (in `example.env`) | Description                                       |
|--------------------------|----------|--------------------------------|-------------------------------------------------------|
| `DJANGO_SECRET_KEY`    | Yes      | `change_me` — **change this**  | Django's cryptographic secret key                       |
| `DJANGO_DEBUG`         | Yes      | `True`                          | Enables/disables Django debug mode                      |
| `DJANGO_ALLOWED_HOSTS` | Yes      | `localhost,127.0.0.1`           | Comma-separated list of allowed hostnames                |
| `BACKEND_PORT`         | Yes      | `8000`                          | Host port on which the Django admin/API is exposed        |
| `DATABASE_HOST`        | Yes      | `db`                            | Hostname of the database service                          |
| `DATABASE_PORT`        | Yes      | `5432`                          | Port of the database service                               |
| `POSTGRES_DB`          | Yes      | `conduit`                       | Name of the PostgreSQL database                             |
| `POSTGRES_USER`        | Yes      | `conduit`                       | Database user used internally by the backend                |
| `POSTGRES_PASSWORD`    | Yes      | `change_me` — **change this**  | Password for the database user                              |
| `FRONTEND_PORT`        | Yes      | `8282`                          | Host port on which the frontend is exposed                   |

> [!IMPORTANT]
> `example.env` is committed to the repository and must only ever contain
> placeholder values. Your real, secret values belong exclusively in
> `.env`, which is excluded from git via `.gitignore` and must never be
> committed.

---

### 2. Networking

All three services are attached to a single bridge network,
`conduit_network`. This allows the `backend` container to reach the
database simply by using the service name `db` as the hostname, no
manual IP configuration required.

---

### 3. Persistence

The `db` service uses a named Docker volume mounted at
`/var/lib/postgresql/data`, so the database content survives
`docker compose down` (without `-v`) and subsequent `docker compose up -d`
runs. Only `docker compose down -v` removes the volume and resets the
database completely.

---

### 4. Restart Behavior

All services are configured with `restart: unless-stopped`, so Docker
automatically restarts a container if it crashes or the host reboots,
unless it was explicitly stopped by the user.

---

### 5. Viewing and Saving Logs

To view the logs of a running container:

```bash
docker compose logs -f backend
```

To save a container's logs to a file for later use:

```bash
docker logs conduit_backend > backend-logs.txt
```

(Replace `conduit_backend` with the container name of the service you want
to inspect.)

---

### 6. Stopping the Stack

```bash
docker compose down
```

## Testing

Before considering the setup complete, the following was verified:

- The frontend is reachable at `http://<host-ip>:8282`
- The backend runs via `gunicorn` (a WSGI server), not the Django
  development server
- If a container terminates unexpectedly, it restarts automatically due
  to the `restart: unless-stopped` policy
- Navigating through the application loads data correctly everywhere
- Container logs can be viewed via the CLI and saved to a file for later
  use