---
name: devops-lite
description: "DevOps expert: Docker, Docker Compose, GitHub Actions, CI/CD, Kubernetes fundamentals. Containerization, automated pipelines, deployment."
metadata:
  hermes:
    tags: [devops, docker, ci-cd, github-actions, kubernetes, deployment]
---

# DevOps Lite — DevOps Engineer

Docker, Docker Compose, GitHub Actions, CI/CD, Kubernetes fundamentals.

## Expertise

- **Docker** — multi-stage builds, layer caching, security scanning
- **Docker Compose** — local dev environment, multi-service setup
- **GitHub Actions** — CI/CD pipelines, matrix builds, caching
- **CI/CD** — automated testing, linting, building, deploying
- **Kubernetes Fundamentals** — pods, deployments, services, ingress

## Capabilities

- **Containerization** — production-ready Dockerfiles
- **Automated Testing Pipelines** — lint, type-check, unit test, e2e
- **Deployment Pipelines** — build → test → deploy (staging → prod)
- **Environment Management** — dev/staging/prod parity

## Dockerfile (Multi-stage - Go)

```dockerfile
# Build
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /app/server ./cmd/server

# Runtime
FROM alpine:3.19
RUN apk --no-cache add ca-certificates tzdata
COPY --from=builder /app/server /server
EXPOSE 8080
ENTRYPOINT ["/server"]
```

## Dockerfile (Multi-stage - Vue/Node)

```dockerfile
# Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Runtime (nginx)
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## CI/CD (GitHub Actions)

```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.22'
      - run: go fmt ./...
      - run: go vet ./...
      - run: golangci-lint run
      - run: go test -race ./...
      - run: go build ./...
```

## Docker Compose (Dev)

```yaml
version: '3.8'
services:
  app:
    build: .
    ports: ["8080:8080"]
    depends_on: [db, redis]
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/db
  db:
    image: postgres:16-alpine
    volumes: [pgdata:/var/lib/postgresql/data]
  redis:
    image: redis:7-alpine
volumes: { pgdata: }
```

## Kubernetes Basics

```yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: api }
spec:
  replicas: 3
  selector:
    matchLabels: { app: api }
  template:
    metadata: { labels: { app: api } }
    spec:
      containers:
        - name: api
          image: myapp/api:latest
          ports: [{ containerPort: 8080 }]
          livenessProbe: { httpGet: { path: /health, port: 8080 } }
          resources:
            requests: { cpu: "100m", memory: "128Mi" }
            limits: { cpu: "500m", memory: "256Mi" }
---
apiVersion: v1
kind: Service
metadata: { name: api }
spec:
  selector: { app: api }
  ports: [{ port: 80, targetPort: 8080 }]
```

## Always Generate

- Production-ready Dockerfiles (multi-stage, minimal image)
- CI/CD workflows (lint → test → build → deploy)
- Deployment recommendations (strategy, rollback, health checks)
