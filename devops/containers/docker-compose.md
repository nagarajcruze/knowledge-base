# Docker Compose: Multi-Container Orchestration

## 1. What is Docker Compose?

Running applications with multiple services (e.g., a frontend, a backend API, and a database) using `docker run` commands becomes tedious. You have to manually create networks, volumes, configure environment variables, and run commands in a specific order.

**Docker Compose** solves this. It is a tool that allows you to define and manage multi-container applications in a single YAML configuration file (`docker-compose.yml`). With Compose, you can start, stop, and configure your entire application stack with a single command.

---

## 2. The `docker-compose.yml` Structure

A standard compose file is broken down into four key top-level keys:
- **`version`**: Specifies the schema version of the compose file format.
- **`services`**: Defines the list of containers to deploy.
- **`networks`**: Configures custom networks for service isolation.
- **`volumes`**: Allocates persistent volumes.

### Example Stack: Frontend, Backend API, and PostgreSQL

Here is a production-ready template for a 3-tier application stack:

```yaml
version: '3.8'

services:
  # Service 1: React/Vue/Angular Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3000:80"
    depends_on:
      - api
    networks:
      - app-network

  # Service 2: Python/Node/Go Backend API
  api:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgres://db_user:db_pass@db:5432/app_db
    depends_on:
      - db
    networks:
      - app-network

  # Service 3: Database Store
  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=db_user
      - POSTGRES_PASSWORD=db_pass
      - POSTGRES_DB=app_db
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - app-network

volumes:
  pgdata:

networks:
  app-network:
    driver: bridge
```

---

## 3. Key Concepts Explained

### 1. Service Communication (DNS Resolution)
In the example above, all services are attached to `app-network`. Under the hood, Docker Compose sets up a bridge network and configures an internal DNS server. 
- The `api` service can reach the database by setting its connection host to `db` (the service name).
- The `frontend` service can connect to the api using the hostname `api`.

### 2. Startup Ordering (`depends_on`)
The `depends_on` setting controls container start ordering. 
- In our configuration, running Compose will spin up `db` first, then `api`, and finally `frontend`.
- *Note: `depends_on` only waits for the container to start, not for the service inside it to be fully "healthy" (e.g., PostgreSQL database initialization). To handle this, implement healthchecks or container startup delay scripts.*

### 3. Application Healthchecks
Use the `HEALTHCHECK` instruction to tell the Docker engine how to test if your container is still functioning properly.

```dockerfile
FROM nginx:alpine
# Check every 5 minutes if the homepage returns HTTP 200
HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```
You can view the health status of a container by running `docker ps` (it will display `(healthy)` or `(unhealthy)` next to the status).

---

## 4. Essential CLI Command Reference

Execute these commands in the directory containing your `docker-compose.yml` file.

- **Start all services (Detached Mode)**:
  ```bash
  docker-compose up -d
  ```
  *This automatically builds any custom images, creates networks, volumes, and starts containers.*
- **View aggregated real-time logs**:
  ```bash
  docker-compose logs -f
  ```
  *Color-codes the logs for each container for easy troubleshooting.*
- **Check service statuses**:
  ```bash
  docker-compose ps
  ```
- **Stop and remove all containers, networks, and configurations**:
  ```bash
  docker-compose down
  ```
  *Add `-v` to also delete the persistent volumes: `docker-compose down -v`.*
- **Rebuild and restart services after making code changes**:
  ```bash
  docker-compose up -d --build
  ```
- **Run a command inside a running service**:
  ```bash
  docker-compose exec api python manage.py migrate
  ```
