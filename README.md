# NexusCore

Early-stage monorepo for a full-stack app: **Go API**, **Next.js frontend**, and **Redis**, containerized with Docker Compose.

Repository: [ArbinBhasimaOfficial/NexusCore](https://github.com/ArbinBhasimaOfficial/NexusCore)

## Architecture

```
Browser → frontend (:3000, Next.js)
              ↓
         backend (:8080, Go)
              ↓
           redis (:6379)
```

| Service   | Port | Technology        |
|-----------|------|-------------------|
| frontend  | 3000 | Next.js (Node 26) |
| backend   | 8080 | Go 1.26           |
| redis     | 6379 | Redis 7 Alpine    |

## Project status

**Phase:** Docker scaffolding only — not yet runnable end-to-end.

### Done

- `docker-compose.yml` — three services with dependency order (`redis` → `backend` → `frontend`)
- `backend/Dockerfile` — multi-stage Go build producing a `server` binary
- `frontend/Dockerfile` — multi-stage Next.js build (`npm ci` → `build` → `start`)
- `.cursor/rules/nexuscore-status.mdc` — agent context for stack and conventions

### Not started

- **Backend:** no `go.mod`, `go.sum`, or application source (`main.go`, handlers, etc.)
- **Frontend:** no `package.json`, Next.js app tree, or `next.config.*`
- **API:** planned health endpoint `GET /api/health` (referenced in frontend snippet, not implemented)
- **`.gitignore`:** file exists but is empty — should ignore `node_modules`, `.next`, binaries, and local env files

### Known issues

1. **`frontend/.env`** — currently contains JavaScript, not environment variables. Move that logic into a proper module (e.g. API client) and use `.env.example` for `NEXT_PUBLIC_API_URL`.
2. **`NEXT_PUBLIC_API_URL` in Compose** — `http://backend:8080` works for server-side calls inside Docker. Browser-side requests from the host need `http://localhost:8080` (or a reverse proxy).
3. **Docker builds will fail** until backend and frontend application code exists.

## Repository layout

```
NexusCore/
├── backend/
│   └── Dockerfile          # Expects go.mod + Go source at build time
├── frontend/
│   ├── Dockerfile          # Expects package.json + Next.js app
│   └── .env                # Misplaced; should be refactored (see above)
├── docker-compose.yml
├── .cursor/rules/
│   └── nexuscore-status.mdc
├── .gitignore
└── README.md
```

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and Docker Compose v2+
- For local development (once scaffolded): Go 1.26+, Node.js 26+

## Running with Docker

When backend and frontend source are in place:

```bash
docker compose up --build
```

| URL | Service |
|-----|---------|
| http://localhost:3000 | Frontend |
| http://localhost:8080 | Backend API |
| localhost:6379 | Redis |

Backend environment (set in Compose):

| Variable     | Value (in Compose) |
|--------------|--------------------|
| `REDIS_ADDR` | `redis:6379`       |

Frontend environment:

| Variable               | Compose default           | Notes |
|------------------------|---------------------------|-------|
| `NEXT_PUBLIC_API_URL`  | `http://backend:8080`     | Use `http://localhost:8080` for browser access from the host |

## Conventions

- Health check endpoint: **`GET /api/health`**
- Do not commit secrets; use `frontend/.env.example` with documented variables
- Pin base images in Dockerfiles (`golang:1.26.3-alpine`, `node:26.2.0-alpine`)

## Next steps

1. Scaffold Go backend: `go mod init`, HTTP server on `:8080`, Redis client, `GET /api/health`
2. Scaffold Next.js frontend and wire API URL for dev vs Docker
3. Populate `.gitignore` and add `frontend/.env.example`
4. Verify `docker compose up --build` runs all three services

## License

Not specified yet.
