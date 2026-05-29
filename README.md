# NexusCore

Monorepo for a full-stack banking-style app: **Go REST API**, **Next.js frontend** (planned), and **Docker Compose** for local orchestration.

Repository: [ArbinBhasimaOfficial/NexusCore](https://github.com/ArbinBhasimaOfficial/NexusCore)

## Architecture

**Target (Docker Compose):**

```
Browser → frontend (:3000, Next.js)
              ↓
         backend (:8080, Go)
              ↓
           redis (:6379)
```

**Current backend (local dev):**

```
HTTP client → backend (:3000, Go + gorilla/mux)
                    ↓
              PostgreSQL (:5432)
```

| Component  | Port | Technology              | Status        |
|------------|------|-------------------------|---------------|
| backend    | 3000 | Go 1.26, gorilla/mux    | Implemented   |
| PostgreSQL | 5432 | Postgres (local)        | Used by API   |
| frontend   | 3000 | Next.js (Node 26)       | Not scaffolded |
| redis      | 6379 | Redis 7 Alpine          | Compose only  |

Compose still defines Redis and maps the backend to **8080**; the running server listens on **3000** and uses Postgres, not Redis. Align ports, storage, and env vars before relying on `docker compose up`.

## Project status

**Phase:** Backend API with PostgreSQL — frontend and Docker path not wired end-to-end.

### Done

- **Backend** — `go.mod`, account REST API, Postgres store (`storage.go`), `Makefile` (`build`, `run`, `test`)
- **Docker scaffolding** — `docker-compose.yml`, `backend/Dockerfile`, `frontend/Dockerfile`
- **Agent context** — `.cursor/rules/nexuscore-status.mdc`
- **Build guide** — `steps.txt` (step-by-step checklist for the original Redis + health stack)

### Not started / incomplete

- **Frontend:** no `package.json`, Next.js app, or `next.config.*`
- **Docker integration:** backend port, Postgres service, and env-based DB config not aligned with Compose
- **Health endpoint:** `GET /api/health` (planned in `steps.txt`, not implemented)
- **`.gitignore`:** empty — should ignore `node_modules`, `.next`, `backend/bin/`, and local env files
- **Secrets:** database URL is hardcoded in `storage.go`; move to environment variables

### Known issues

1. **Port mismatch** — `main.go` serves on `:3000`; Compose exposes `8080:8080`.
2. **Storage mismatch** — API uses PostgreSQL; Compose only starts Redis.
3. **`frontend/.env`** — may contain JavaScript instead of `KEY=VALUE` pairs; use `.env.example` and an API client module.
4. **`NEXT_PUBLIC_API_URL`** — `http://backend:8080` works inside Docker; browser calls from the host need `http://localhost:8080` (or your actual backend port).
5. **Transfer endpoint** — `POST /transfer` decodes JSON but does not persist transfers yet.

## API (backend)

Base URL when running locally: `http://localhost:3000`

| Method | Path            | Description                    |
|--------|-----------------|--------------------------------|
| GET    | `/account`      | List all accounts              |
| POST   | `/account`      | Create account (JSON body)     |
| GET    | `/account/{id}` | Get account by ID              |
| DELETE | `/account/{id}` | Delete account by ID           |
| POST   | `/transfer`     | Transfer request (stub)        |

**Create account body:**

```json
{
  "FirstName": "Jane",
  "LastName": "Doe"
}
```

**Transfer body:**

```json
{
  "toAccount": 2,
  "amount": 100
}
```

Responses are JSON. Errors return `{ "error": "..." }` with HTTP 400.

## Repository layout

```
NexusCore/
├── backend/
│   ├── main.go           # Entry: Postgres store + HTTP server
│   ├── api.go            # Routes and handlers
│   ├── storage.go        # Postgres implementation
│   ├── types.go          # Account, requests
│   ├── Makefile          # build, run, test
│   ├── Dockerfile
│   ├── go.mod / go.sum
│   └── bin/              # Built binary (nexuscore)
├── frontend/
│   ├── Dockerfile
│   └── .env              # Fix or replace with .env.example
├── docker-compose.yml
├── steps.txt             # Full build checklist
├── .cursor/rules/
│   └── nexuscore-status.mdc
├── .gitignore
└── README.md
```

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and Docker Compose v2+ (for Compose workflow)
- **Go 1.26+** — local backend development
- **PostgreSQL** — running locally with a database the app can connect to (see `storage.go` for connection settings; configure before first run)
- **Node.js 26+** — when the frontend is scaffolded

## Running the backend locally

1. Start PostgreSQL and ensure the database exists and matches your connection settings in `storage.go` (or update the connection string).
2. From `backend/`:

```bash
make run
```

Or manually:

```bash
go build -o bin/nexuscore .
./bin/nexuscore
```

3. Example requests:

```bash
curl -s http://localhost:3000/account
curl -s -X POST http://localhost:3000/account \
  -H 'Content-Type: application/json' \
  -d '{"FirstName":"Jane","LastName":"Doe"}'
curl -s http://localhost:3000/account/1
```

## Running with Docker

When backend and frontend match the Compose layout (port **8080**, dependencies, env vars):

```bash
docker compose up --build
```

| URL | Service |
|-----|---------|
| http://localhost:3000 | Frontend (when added) |
| http://localhost:8080 | Backend API (Compose default) |
| localhost:6379 | Redis |

Until the backend is updated for Compose, prefer local `make run` with Postgres.

## Conventions

- Prefer **environment variables** for database and Redis addresses (do not commit secrets).
- Health check (planned): **`GET /api/health`**
- Pin base images in Dockerfiles (`golang:1.26.3-alpine`, `node:26.2.0-alpine`)

## Next steps

1. Externalize Postgres config (`DATABASE_URL` or similar) and add Postgres to `docker-compose.yml`.
2. Align backend listen port with Compose (`:8080`) or update Compose port mapping.
3. Scaffold Next.js frontend and `frontend/.env.example`.
4. Implement `GET /api/health` and optional Redis ping if Redis stays in the stack.
5. Populate `.gitignore` and verify `docker compose up --build` from a clean clone.

## License

Not specified yet.
