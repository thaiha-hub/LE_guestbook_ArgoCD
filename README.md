# Guestbook Project with ArgoCD

A Swedish Guestbook Web Application deployed on OpenShift/Kubernetes using ArgoCD for GitOps-based continuous deployment. Users can submit their name and message, which are stored in a database and displayed to visitors.

---

## Architecture

The project follows a **3-tier architecture**:

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Frontend  │────▶│   Backend   │────▶│  PostgreSQL │     │    Redis    │
│   (NGINX)   │     │    (Go)     │────▶│  (Database) │     │   (Cache)   │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
```

---

## Components

| Component | Technology | Port | Location |
|-----------|------------|------|----------|
| Frontend | NGINX 1.26 + HTML5/JS | 8080 | `frontend/` |
| Backend | Go 1.24 REST API | 8080 | `backend/` |
| Database | PostgreSQL 16 | 5432 | k8s deployment |
| Cache | Redis | 6379 | k8s deployment |

---

## Directory Structure

```
├── argocd-app.yaml              # ArgoCD Application manifest
├── .github/workflows/
│   └── pipeline.yml             # GitHub Actions CI/CD
├── backend/
│   ├── main.go                  # Go REST API
│   ├── go.mod                   # Dependencies
│   └── Dockerfile               # Multi-stage build
├── frontend/
│   ├── index.html               # SPA frontend
│   ├── nginx.conf               # Reverse proxy config
│   └── Dockerfile               # NGINX container
└── k8s/                         # Kubernetes manifests
    ├── backend.yaml             # Backend deployment (2 replicas)
    ├── frontend.yaml            # Frontend deployment (2 replicas)
    ├── postgres.yaml            # PostgreSQL + PVC
    ├── redis.yaml               # Redis deployment
    ├── services.yaml            # All services
    ├── configmap.yaml           # Non-secret config
    ├── secret-db.yaml           # DB credentials
    ├── secret-redis.yaml        # Redis password
    └── route.yaml               # OpenShift Route + TLS
```

---

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Health check with DB/cache status |
| GET | `/api/entries` | Fetch all guestbook entries (cached) |
| POST | `/api/entries` | Create new entry |
| GET | `/api/stats` | Statistics (total entries, cache status) |

---

## CI/CD Pipeline

The deployment uses a GitOps workflow:

```
Code Push → GitHub Actions → Build Images → Push to ghcr.io
                                    ↓
                          Update k8s/*.yaml with new tags
                                    ↓
                          Commit to main [skip ci]
                                    ↓
                          ArgoCD detects changes
                                    ↓
                          Auto-sync to OpenShift
```

---

## ArgoCD Configuration

From `argocd-app.yaml`:
- **Application**: `thaiha-guestbook-argocd`
- **Namespace**: `grupp2`
- **Source Path**: `k8s/`
- **Sync Policy**: Automated with auto-prune and self-heal

---

## Key Features

- **Caching**: Redis with 30-second TTL for performance
- **High Availability**: 2 replicas for frontend and backend
- **Persistent Storage**: PostgreSQL uses PVC (1Gi)
- **TLS**: Encrypted route via cert-manager
- **GitOps**: Automatic deployments via ArgoCD

---

## Resource Limits

| Service | CPU | Memory |
|---------|-----|--------|
| Backend | 100m-200m | 128Mi-256Mi |
| Frontend | 50m-100m | 64Mi-128Mi |
| PostgreSQL | 100m-300m | 256Mi-512Mi |
| Redis | 50m-100m | 64Mi-128Mi |







