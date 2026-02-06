# Thai Ha Dang – Guestbook med ArgoCD

Detta projekt är ett individuellt CI/CD-slutprojekt för kursen DevOps 24. Projektet implementerar en 3-lager Guestbook-applikation på OpenShift/Kubernetes, där deployaringar och uppdateringar hanteras med ArgoCD (GitOps).

---

## Översikt

- Gästbok med frontend (HTML/JS + NGINX), backend (Go REST API), PostgreSQL (persistent storage) och Redis (cache)
- Användare kan:
  - Skapa inlägg
  - Visa alla tidigare inlägg
  - Se statistik över antal inlägg och cache-träffar
- ArgoCD hanterar deployment automatiskt vid ändringar i YAML eller nya Docker-images

---

## Arkitektur
┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│ Frontend │────▶│ Backend │────▶│ PostgreSQL │ │ Redis │
│ NGINX │ │ Go REST API │────▶│ Database │ │ Cache │
└─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘

- Frontend: `thaiha-guestbook-frontend`
- Backend: `thaiha-guestbook-backend`
- PostgreSQL: `thaiha-guestbook-postgres`
- Redis: `thaiha-guestbook-redis`

---

## URL

Guestbook är åtkomlig på:  
[https://thaiha-frontend-grupp2.apps.devops24.cloud2.se/]

---

## Resursnamnstandard

Alla resurser är prefixed med `thaiha-` för att undvika konflikter med andra studenter:

- `thaiha-guestbook-frontend` / `-backend`  
- `thaiha-guestbook-postgres` / `-redis`  
- `thaiha-guestbook-config` / `-secret`  
- `thaiha-guestbook-route`  

---

## CI/CD-flöde

### GitHub Actions (CI)

- Trigger: push till `main`  
- Steg:
  1. Bygger Docker-images för frontend och backend
  2. Pushar bilder till GitHub Container Registry (GHCR)
  3. Uppdaterar YAML-filer med nya image-tags
  4. Commit/push till `main` (`[skip ci]`)

### ArgoCD (CD / GitOps)

- Övervakar `k8s/` i GitHub
- Auto-sync, auto-prune och self-heal
- Applicerar YAML-ändringar automatiskt
- Ny version deployas direkt när Docker-images uppdateras

---

## API Endpoints

| Metod | Endpoint | Funktion |
|-------|---------|----------|
| GET   | `/health` | Kontroll av DB och cache |
| GET   | `/api/entries` | Hämta alla inlägg (Redis-cache) |
| POST  | `/api/entries` | Skapa nytt inlägg |
| GET   | `/api/stats` | Statistik: antal inlägg och cache-status |

---

## Reflektioner & Förbättringar

- GitOps gör deployment reproducerbar och versionstyrd  
- PVC för PostgreSQL säkerställer persistens av data  
- Redis-cache förbättrar prestanda  
- Förbättringsförslag:
  - Införa liveness/readiness probes
  - Kryptering av secrets (t.ex. SealedSecrets)
  - Fullständig TLS med cert-manager
  - Parametrisering via Helm Charts
