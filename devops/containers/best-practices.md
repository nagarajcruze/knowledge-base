# Docker Production Best Practices

Deploying Docker containers to production requires focusing on image size optimization, build speed caching, container security, and runtime configuration.

---

## 1. Image Size Optimization

Smaller images deploy faster, consume less network bandwidth, and reduce the attack surface.

### A. Use Multi-Stage Builds
Multi-stage builds allow you to use different base images for the "build" stage and the "runtime" stage. This separates compiling dependencies from running the application, leaving compilers, build tools, and tests out of the final image.

#### Before/After Example (React Application)

```dockerfile
# STAGE 1: Build the application
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# STAGE 2: Package runtime assets only
FROM nginx:alpine
# Copy compiled output from the builder stage
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### B. Use a `.dockerignore` File
Prevent large local files (like `node_modules`, build logs, `.git`, and secrets) from being copied into the Docker build context. Create a `.dockerignore` file in your root folder:
```text
.git
node_modules
npm-debug.log
Dockerfile
docker-compose.yml
.env
```

---

## 2. Optimizing Build Speed (Layer Caching)

Every command instruction in a `Dockerfile` (like `RUN`, `COPY`, `ADD`) creates a read-only layer. Docker caches these layers. If a layer's contents and all previous layers do not change, Docker reuses the cache.

### Order Instructions from Least to Most Frequently Changed
- Place infrequently changed instructions (like OS package installations and dependencies setup) at the top.
- Place frequently changed instructions (like copying source code) at the bottom.

#### Bad Caching Pattern:
```dockerfile
COPY . .
RUN npm install   # Every tiny code change busts cache, triggering a full npm install
```

#### Good Caching Pattern:
```dockerfile
COPY package*.json ./
RUN npm install   # Cached and reused unless package.json modifications occur
COPY . .          # Only copies source code changes afterwards
```

---

## 3. Container Security

By default, Docker containers run with high privilege levels. Secure them before production deployment.

### A. Do Not Run as Root
By default, processes inside containers run as the superuser (root). If an attacker compromises your application, they gain root access to the container and potentially the host machine.
- Configure a non-root user using the `USER` directive:

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .

# Create a non-root group and user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 3000
CMD ["node", "index.js"]
```

### B. Mount the Filesystem as Read-Only
If your application does not need to write files to disk, run the container with a read-only filesystem. This prevents attackers from installing packages or modifying application source files.
```bash
docker run --read-only -d my-app:1.0
```

### C. Vulnerability Scanning
Regularly scan your images for known CVE security vulnerabilities:
- **Trivy**:
  ```bash
  trivy image my-app:1.0
  ```
- **Docker Scout**:
  ```bash
  docker scout cves my-app:1.0
  ```

---

