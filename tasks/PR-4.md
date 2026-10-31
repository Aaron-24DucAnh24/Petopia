# PR-4: Dockerize services and add image build CI

**Status:** In progress
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

- [ ] TC1: `docker build` succeeds for backend, frontend and chat from a clean context.
- [ ] TC2: Backend container starts, listens on 9999 as a non-root user and serves `/swagger/index.html` when given config via env vars pointing at the umbrella infrastructure.
- [ ] TC3: Frontend container starts as a non-root user and `GET /` on port 3000 returns 200; the build args are inlined into the client bundle.
- [ ] TC4: Chat container starts as a non-root user with env vars pointing at the umbrella infrastructure, and `GET /docs` on port 8000 returns 200.
- [ ] TC5: Missing required config fails loudly (chat container exits with a validation error when `JWT_SECRET` is unset).
- [ ] TC6: `.dockerignore` keeps local secrets and build output out of the context (`.env`, `.env.local`, `appsettings.Development.json`, `node_modules`, `.next`, `bin`/`obj`, `.venv`).
- [ ] TC7: The `docker-build` workflow is valid YAML, triggers on `pull_request` and `push` to `main`, does not push, and passes on the PR in each sub-repo.
- [ ] TC8: Every commit matches `[<type>] PR-4: <summary>`; every PR is titled `PR-4: Dockerize services and add image build CI` with an empty body.

## Result

<Fill in after coding.>
