# Docker Compose

Docker Compose is a tool for defining and running **multi-container** Docker applications using a YAML configuration file.

## Why Compose?

- Define your entire application stack in a single file
- Start all services with one command
- Manage networking between containers automatically
- Perfect for local development and testing

## docker-compose.yml Example

```yaml
version: '3.8'

services:
  web:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - api
    environment:
      - API_URL=http://api:8080

  api:
    build: ./backend
    ports:
      - "8080:8080"
    depends_on:
      - db
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/mydb

  db:
    image: postgres:15-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=mydb

volumes:
  pgdata:
```

## Essential Commands

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop all services
docker-compose down

# Rebuild and restart
docker-compose up -d --build

# Scale a service
docker-compose up -d --scale web=3
```
