# PR-4: Dockerize services and add image build CI

**Status:** Done
**Scope:** backend, frontend, chat, umbrella

## Description

None of the sub-repos has CI that checks the code still builds, and none of them can be shipped as a container. The services will likely be deployed on Kubernetes later, so each one needs a production image.

Requirements:

1. **Dockerfile per sub-repo** (multi-stage, small runtime image, non-root user, plus a `.dockerignore` so local secrets/build output never enter the build context):
   - **backend**: build with `mcr.microsoft.com/dotnet/sdk:7.0`, `dotnet publish` `Petopia.API`, run on `mcr.microsoft.com/dotnet/aspnet:7.0`. Listen on port `9999` (`ASPNETCORE_URLS`), like local dev. Configuration is not baked in: it comes from env vars (`ConnectionStrings__Database`, ...) or a mounted `appsettings.<env>.json` at deploy time.
   - **frontend**: `node:20-alpine`, `npm ci` + `next build` with Next.js `output: 'standalone'`, run `node server.js` on port `3000`. `NEXT_PUBLIC_*` values are inlined at build time, so they are Docker build args (`NEXT_PUBLIC_API_ENDPOINT`, `NEXT_PUBLIC_CHAT_API_ENDPOINT`, `NEXT_PUBLIC_CKEDITOR_LICENSE`).
   - **chat**: `python:3.12-slim`, install `requirements.txt` only, run `uvicorn app.main:app` on port `8000`. Settings come from env vars.
2. **Verify locally** that each image builds and the container starts.
3. **CI per sub-repo**: `.github/workflows/docker-build.yml` builds the image on every pull request and every push to `main` with `docker/build-push-action` (`push: false`, GitHub Actions layer cache). No registry push yet.
4. Document the Docker images in the umbrella README.

Out of scope: pushing images to a registry, Kubernetes manifests, running migrations/seeding in containers.

## Test cases

- [x] TC1: `docker build` succeeds for backend, frontend and chat from a clean context.
  - Verified: built all three locally (Docker 24.0.6). Image sizes: backend 246 MB, chat 218 MB, frontend 173 MB.
- [x] TC2: Backend container starts, listens on 9999 as a non-root user and serves `/swagger/index.html` when given config pointing at the umbrella infrastructure.
  - Verified: mounted the `config/backend` template as `/app/appsettings.Production.json` with `127.0.0.1` replaced by `host.docker.internal`, after running `dotnet ef database update`. `/swagger/index.html` and `/swagger/v1/swagger.json` returned 200, `id -un` = `petopia`, and Serilog wrote to `/app/Logs`. The mounted file's `Kestrel` URL was overridden by the image's `Kestrel__Endpoints__Http__Url=http://+:9999`. Before migrations, the container exits with SQL error 4060 (database missing), as expected.
- [x] TC3: Frontend container starts as a non-root user and `GET /` on port 3000 returns 200; the build args are inlined into the client bundle.
  - Verified: built with `--build-arg NEXT_PUBLIC_API_ENDPOINT=http://127.0.0.1:9999/api --build-arg NEXT_PUBLIC_CHAT_API_ENDPOINT=http://127.0.0.1:8000`. `GET /` returned 200, user `node`, and both URLs appear in `.next/static/chunks/*.js`.
- [x] TC4: Chat container starts as a non-root user with env vars pointing at the umbrella infrastructure, and `GET /docs` on port 8000 returns 200.
  - Verified: `--env-file` built from `config/chat/.env` with `localhost` replaced by `host.docker.internal`. `/docs` returned 200, user `petopia`.
- [x] TC5: Missing required config fails loudly (chat container exits with a validation error when `JWT_SECRET` is unset).
  - Verified: `docker run --rm petopia-chat:test` with no env exits 1 with pydantic `Field required` errors for `jwt_secret`, `mongodb_url`, `redis_url`, ...
- [x] TC6: `.dockerignore` keeps local secrets and build output out of the context (`.env`, `.env.local`, `appsettings.Development.json`, `node_modules`, `.next`, `bin`/`obj`, `.venv`).
  - Verified: with those files present locally, built a throwaway `COPY . /ctx` image per repo and searched it. Only `frontend/.env.production` remains; it is committed, holds no secrets, and build args override it.
- [x] TC7: The `docker-build` workflow is valid YAML, triggers on `pull_request` and `push` to `main`, does not push, and passes on the PR in each sub-repo.
  - Verified: `yaml-lint` passes (actionlint not installed). The `build` check passed on all three PRs: backend 2m14s, frontend 8m0s (cold cache), chat 29s.
- [x] TC8: Every commit matches `[<type>] PR-4: <summary>`; every PR is titled `PR-4: Dockerize services and add image build CI` with an empty body.

## Result

- **backend**: `Dockerfile` (sdk:7.0 build → aspnet:7.0 runtime, user `petopia`, port 9999), `.dockerignore`, `.github/workflows/docker-build.yml`. PR: https://github.com/Aaron-24DucAnh24/Petopia-Backend/pull/26
- **frontend**: `Dockerfile` (node:20-alpine, `npm ci` with Cypress binary skipped, standalone output, user `node`, port 3000), `.dockerignore`, `output: 'standalone'` in `next.config.js`, `.github/workflows/docker-build.yml`. PR: https://github.com/Aaron-24DucAnh24/Petopia-Frontend/pull/3
- **chat**: `Dockerfile` (python:3.12-slim, user `petopia`, port 8000), `.dockerignore`, `.github/workflows/docker-build.yml`. PR: https://github.com/Aaron-24DucAnh24/Petopia-Chat/pull/3
- **umbrella**: "Docker images" section in `README.md` (build commands, ports, runtime config).

CI uses `docker/build-push-action@v6` with `push: false` and the GitHub Actions cache (`type=gha`, one scope per service).

Known issues, not fixed here:
- `docker compose up` currently fails to pull `minio/mc` (the `storage-init` job) from Docker Hub.
- The backend builds image URLs from `Minio:Endpoint` (`localhost:9000`), and the frontend only allows images from `localhost:9000`. Both need a real host before a Kubernetes deployment.
