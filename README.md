# Petopia — Online Pet Adoption Platform

Petopia is a social platform for pet adoption. It connects pet owners with adopters and gives pet lovers a place to meet. It started as a capstone project.

This repository brings the whole system together in one place. Each service is still developed in its own repository, and is included here as a [git submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules). The local environment for **all** services (databases, cache, storage, search) is defined once here, in [docker-compose.yml](docker-compose.yml), and this README is the single source of truth for setting it up.

| Folder | Service | Stack | Repository |
| --- | --- | --- | --- |
| [`frontend/`](frontend) | Web app | Next.js 13 · TypeScript · Tailwind CSS | [Petopia-Frontend](https://github.com/Aaron-24DucAnh24/Petopia-Frontend) |
| [`backend/`](backend) | REST API + real-time hub | ASP.NET Core (net7.0) · EF Core · SQL Server | [Petopia-Backend](https://github.com/Aaron-24DucAnh24/Petopia-Backend) |
| [`chat/`](chat) | Chat microservice | FastAPI · MongoDB · WebSocket | [Petopia-Chat](https://github.com/Aaron-24DucAnh24/Petopia-Chat) |

## Architecture

```mermaid
flowchart LR
    U([User browser]) --> FE[Frontend<br/>Next.js :3000]

    FE -- REST + SignalR --> API[Backend API<br/>ASP.NET Core :9999]
    FE -- REST + WebSocket --> CHAT[Chat Service<br/>FastAPI :8000]

    API --> SQL[(SQL Server)]
    API --> REDIS[(Redis)]
    API --> MINIO[(MinIO<br/>file storage)]
    API --> MEILI[(MeiliSearch)]

    CHAT --> MONGO[(MongoDB)]
    CHAT -- Pub/Sub --> REDIS
```

- The **Backend** handles authentication, pets, adoption, blogs and payments. It issues JWT access tokens and sends real-time notifications through a SignalR hub (`/hubs/realtime`).
- The **Chat Service** runs on its own, but accepts the same JWT tokens (shared HS256 secret), so users don't log in twice. It uses the Backend's Redis instance for Pub/Sub.
- The **Frontend** talks to both services: `NEXT_PUBLIC_API_ENDPOINT` for the Backend and `NEXT_PUBLIC_CHAT_API_ENDPOINT` for Chat.

In local development the infrastructure runs in Docker, and the three services run directly on your machine.

## Prerequisites

| Tool | Version | Used by |
| --- | --- | --- |
| [Git](https://git-scm.com/) | any recent | everything |
| [Docker](https://www.docker.com/products/docker-desktop) with Compose v2 | any recent | infrastructure |
| [.NET SDK](https://dotnet.microsoft.com/download) | 8.0 or later | backend build |
| [ASP.NET Core Runtime 7.0](https://dotnet.microsoft.com/download/dotnet/7.0) | 7.0 | backend run (projects target `net7.0`) |
| [dotnet-ef](https://learn.microsoft.com/en-us/ef/core/cli/dotnet) | 7.x | backend migrations |
| [Python](https://www.python.org/downloads/) | 3.9+ | chat |
| [Node.js](https://nodejs.org/) | 18+ (with npm) | frontend |

```bash
dotnet tool install --global dotnet-ef --version "7.*"
```

> Only have the .NET 8 runtime? Set `DOTNET_ROLL_FORWARD=Major` before `dotnet run` / `dotnet ef` to run the `net7.0` projects on it.

## 1. Get the code

```bash
git clone --recurse-submodules https://github.com/Aaron-24DucAnh24/Petopia.git
cd Petopia
```

If you already cloned without submodules:

```bash
git submodule update --init --recursive
```

## 2. Start the infrastructure

Every container the services need is in the root [docker-compose.yml](docker-compose.yml):

| Container | Image | Port(s) | Used by |
| --- | --- | --- | --- |
| `database` | Azure SQL Edge (SQL Server) | 1433 | backend (data + Hangfire) |
| `cache` | Redis 7 | 6379 | backend (cache), chat (Pub/Sub) |
| `storage` | MinIO | 9000 (API), 9001 (console) | backend (images) |
| `storage-init` | MinIO client | — | one-off job: creates the public `image` bucket |
| `search` | MeiliSearch | 7700 | backend (search on `pet` and `blog`) |
| `mongodb` | MongoDB 7 | 27017 | chat |

Credentials come from a root `.env` file. Copy the template; the defaults work out of the box for local development:

```bash
cp .env.example .env
docker compose up -d
docker compose ps        # storage-init should show "Exited (0)", everything else "Up"
```

Data is kept in named Docker volumes, so it survives `docker compose down`.

## 3. Configure the services

Each service reads its own config file, which is git-ignored inside its repo. Ready-to-use templates live in [config/](config); their credentials match the defaults in [.env.example](.env.example).

```bash
cp config/backend/appsettings.Development.json backend/Petopia.API/appsettings.Development.json
cp config/chat/.env                            chat/.env
cp config/frontend/.env.local                  frontend/.env.local
```

If you change a password in `.env`, update the matching value in these files too:

| `.env` variable | Backend `appsettings.Development.json` | Chat `.env` |
| --- | --- | --- |
| `MSSQL_SA_PASSWORD` | `ConnectionStrings:Database` (`PWD=`) | — |
| `REDIS_PASSWORD` | `RedisCache:ConnectionString` (`password=`) | `REDIS_URL` |
| `MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD` | `Minio:User` / `Minio:Password` | — |
| `MEILI_MASTER_KEY` | `MeiliSearch:ApiKey` | — |
| `MONGODB_USERNAME` / `MONGODB_PASSWORD` | — | `MONGODB_URL` |

Values shared between services:

- `Token:AccessTokenKey` and `Token:Issuer` in the backend must equal `JWT_SECRET` and `JWT_ISSUER` in chat, otherwise chat rejects the backend's tokens. The key must be at least 32 characters (HS256).
- `Minio:Endpoint` stays `localhost:9000`: uploaded image URLs are built from it, and the frontend only renders images from `localhost:9000`.
- `NEXT_PUBLIC_API_ENDPOINT` ends with `/api`; the frontend strips it to reach the SignalR hub.

Optional backend settings, left empty in the template (the related features fail until you fill them in):

| Section | Feature |
| --- | --- |
| `Email` | Sending emails (account activation, password reset...) over SMTP |
| `GoogleAuth:ClientId` | Login with Google |
| `GoogleRecaptcha` | reCAPTCHA on login/register |
| `Braintree` | Payments |

## 4. Run the backend

```bash
cd backend

# Create the database schema (first run, and after pulling new migrations)
cd Petopia.Data
ASPNETCORE_ENVIRONMENT=Development dotnet ef database update
cd ..

# Seed reference data: admin accounts, email templates, provinces/districts/wards,
# dog/cat breeds, advertisements, vaccines (first run only; must run from this folder)
cd Petopia.SeedingData
ASPNETCORE_ENVIRONMENT=Development dotnet run
cd ..

# Start the API on http://127.0.0.1:9999
cd Petopia.API
dotnet run -e ASPNETCORE_ENVIRONMENT=Development
```

`ASPNETCORE_ENVIRONMENT=Development` matters for `dotnet ef` and the seeder: without it they look for `appsettings.json` instead of `appsettings.Development.json`. On Windows PowerShell set it first with `$env:ASPNETCORE_ENVIRONMENT = "Development"`.

The seeder creates this admin account:

```
Email:    anh.bui24ducanh24@hcmut.edu.vn
Password: 123456789
```

## 5. Run the chat service

```bash
cd chat
make install     # creates .venv and installs dependencies
make dev         # http://localhost:8000, auto-reload
make test        # runs pytest (needs the infrastructure running)
```

On Windows (no `make`):

```powershell
cd chat
python -m venv .venv
.venv\Scripts\pip install -r requirements.txt -r requirements-dev.txt
.venv\Scripts\uvicorn app.main:app --reload
```

## 6. Run the frontend

```bash
cd frontend
npm install
npm run dev      # http://localhost:3000
```

Other scripts: `npm run build`, `npm run start`, `npm run lint`. End-to-end tests use Cypress (`npx cypress open`) and need the backend and chat running.

## Service URLs

| Service | URL |
| --- | --- |
| Web app | http://localhost:3000 |
| Backend Swagger | http://127.0.0.1:9999/swagger/index.html |
| Chat Swagger | http://localhost:8000/docs |
| MinIO console | http://127.0.0.1:9001/browser (login: `MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD`) |
| MeiliSearch | http://127.0.0.1:7700 (API key: `MEILI_MASTER_KEY`) |

## Managing the infrastructure

```bash
docker compose ps                 # status
docker compose logs -f <service>  # follow logs, e.g. database, mongodb
docker compose stop               # stop, keep containers and data
docker compose down               # remove containers, keep data
docker compose down -v            # remove containers AND data (full reset)
```

After `docker compose down -v`, run the migrations and the seeder again (step 4).

## Docker images

Each sub-repo has a production `Dockerfile` (multi-stage, runs as a non-root user). Its `docker-build` workflow builds the image on every pull request and every push to `main`; images are not pushed to a registry yet.

| Service | Build | Port | Runtime config |
| --- | --- | --- | --- |
| backend | `docker build -t petopia-backend backend` | 9999 | Env vars using `__` for nesting (`ConnectionStrings__Database`, `Token__AccessTokenKey`, ...) or a mounted `/app/appsettings.Production.json`. `ASPNETCORE_ENVIRONMENT` defaults to `Production`. |
| chat | `docker build -t petopia-chat chat` | 8000 | Env vars from [chat/.env.example](chat/.env.example) (`JWT_SECRET`, `MONGODB_URL`, `REDIS_URL`, ...). The container exits if a required one is missing. |
| frontend | `docker build -t petopia-frontend --build-arg NEXT_PUBLIC_API_ENDPOINT=... --build-arg NEXT_PUBLIC_CHAT_API_ENDPOINT=... frontend` | 3000 | None at runtime. `NEXT_PUBLIC_*` values are inlined by `next build`, so they are build args and each environment needs its own image. `NEXT_PUBLIC_CKEDITOR_LICENSE` defaults to `GPL`. |

Notes:

- The backend listens on `http://+:9999` through the `Kestrel__Endpoints__Http__Url` env var, which overrides any `Kestrel` section in a mounted appsettings file (the local template binds `127.0.0.1`, which isn't reachable from outside a container).
- The backend image doesn't run migrations or the seeder; run them as in step 4 against the target database.
- Inside a container, `localhost` is the container itself. To reach the local infrastructure from a container, use `host.docker.internal` in the connection strings.

## Working with submodules

Each submodule is pinned to a specific commit, and CI keeps the pins current: every push to a sub-repo's `main` runs its `notify-umbrella` workflow, which triggers [bump-submodules](.github/workflows/bump-submodules.yml) here. That workflow points every submodule at its latest `main` and commits `[chore] PR-X: bump <submodules>` to this repo's `main`. To sync a local clone:

```bash
git pull
git submodule update --init --recursive
```

Make code changes inside the submodule folder and push them to that service's own repository. Don't commit submodule pointer changes here by hand. See [CLAUDE.md](CLAUDE.md) for the task and commit conventions.

### CI setup

The `notify-umbrella` workflow in each sub-repo needs a repository secret named `UMBRELLA_DISPATCH_TOKEN`. It holds a fine-grained personal access token scoped to `Aaron-24DucAnh24/Petopia` with **Contents: Read and write**. The bump workflow pushes with the built-in `GITHUB_TOKEN`. If `main` here gets branch protection later, allow GitHub Actions to bypass it, or the bump commit will be rejected. To run a bump by hand, use **Actions → Bump submodules → Run workflow**.
