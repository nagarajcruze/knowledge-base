# Docker Best Practices

## Image Optimization

1. **Use multi-stage builds** to reduce image size
2. **Use `.dockerignore`** to exclude unnecessary files
3. **Order layers by change frequency** — put rarely-changing layers first
4. **Use specific base image tags** — never `latest` in production

```dockerfile
# Multi-stage build example
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
```

## Security

- **Don't run as root** — use `USER` directive
- **Scan images** — `docker scout`, `trivy`
- **Use read-only filesystems** where possible
- **Never store secrets in images** — use env vars or secrets management

## Resource Management

```bash
# Limit container resources
docker run -d \
  --memory="256m" \
  --cpus="0.5" \
  --restart=unless-stopped \
  my-app:1.0
```
