# PR-2: Switch workflow to English and unify the local environment

**Status:** Done
**Scope:** umbrella, backend, chat, frontend

## Description

Two changes to how the repo is worked on and run:

1. **English only.** Every comment, commit message, PR title, task file and doc must be written in English from now on.
   - Translate `CLAUDE.md`, `.claude/skills/task-workflow/SKILL.md`, `.claude/skills/task-workflow/template.md` and `tasks/PR-1.md` to English.
   - Add an explicit "English only" rule to `CLAUDE.md` and the skill.
2. **One Docker Compose file at the umbrella repo.** Today each sub-repo ships its own infrastructure setup (`backend/docker-compose.yml` for SQL Server, Redis, MinIO and MeiliSearch; `chat/docker-compose.yml` for MongoDB). Move all of it into a single `docker-compose.yml` at the umbrella root.
   - Add `docker-compose.yml` + `.env.example` at the root, covering SQL Server, Redis, MinIO, MeiliSearch and MongoDB. Ignore the root `.env`.
   - Add per-service config templates under `config/` whose values match the `.env.example` defaults, so a fresh clone runs with copy-paste only.
   - Rewrite the root `README.md` with the full environment setup and how to run each service.
   - In each sub-repo: remove its `docker-compose.yml` (backend, chat), drop the Docker targets from `chat/Makefile`, and trim its README/CLAUDE.md so setup points to the umbrella README instead of duplicating it.

## Test cases

- [x] TC1: Grep `CLAUDE.md`, `.claude/`, `tasks/` for Vietnamese diacritics → no matches.
  - Verified: `grep -rnP "[ăâđêôơư...]"` over those paths. The only hit is the quoted title of the already-merged PR #1 in `tasks/PR-1.md` (TC5), kept as a historical record until that PR is renamed on GitHub.
- [x] TC2: `CLAUDE.md` and the skill both state that comments, commits, PRs and docs must be in English.
  - Verified: "Language" section in `CLAUDE.md`, step "0. Language" in `SKILL.md`.
- [x] TC3: Root `docker-compose.yml` parses as valid YAML and defines `database`, `cache`, `storage`, `search`, `mongodb` services with the same ports as before (1433, 6379, 9000/9001, 7700, 27017).
  - Verified: parsed with the `yaml` npm package and listed services/ports (plus the one-off `storage-init`).
- [x] TC4: Every `${VAR}` used in `docker-compose.yml` is defined in `.env.example`.
  - Verified: script compared the variables; none missing, none unused.
- [x] TC5: Credentials in `config/` templates (SQL Server, Redis, MinIO, MeiliSearch, MongoDB) match `.env.example`; the chat `JWT_SECRET` matches the backend `Token:AccessTokenKey`.
  - Verified: same script; all checks true, JWT keys ≥ 32 chars, Meili key ≥ 16 bytes.
- [x] TC6: Root `.env` is git-ignored; `.env.example` and `config/` are tracked.
  - Verified: `git check-ignore -v .env config/chat/.env config/frontend/.env.local` only matches `.env` (rule `/.env`).
- [x] TC7: `backend/docker-compose.yml` and `chat/docker-compose.yml` are removed; `chat/Makefile` has no Docker targets; no sub-repo README/CLAUDE.md tells the reader to run Docker from inside the sub-repo.
  - Verified: `ls` on both files fails; `grep -i docker` over sub-repo READMEs/CLAUDE.md/Makefile only finds links to the umbrella setup.
- [ ] TC8: `docker compose up -d` at the root starts all five containers and they become healthy.
  - Not verified: Docker isn't installed on the machine this was done on. Run `cp .env.example .env && docker compose up -d && docker compose ps` to check.
- [x] TC9: Every commit on the PR-2 branches (umbrella + submodules) matches `[<type>] PR-2: <summary>`; every PR is titled `PR-2: <task name>` with an empty body.
  - Verified: `git log --format=%s main..HEAD` in each repo; PRs created with `--body ""`.

## Result

- **English only**: translated [CLAUDE.md](../CLAUDE.md), the [task-workflow skill](../.claude/skills/task-workflow/SKILL.md), its [template](../.claude/skills/task-workflow/template.md) and [PR-1.md](PR-1.md); both `CLAUDE.md` and the skill now state the rule. The two Vietnamese code comments left in the sub-repos (`frontend/src/components/pet/FormPetDetail.tsx`) were translated. Vietnamese UI strings (labels, notifications) are product copy, not comments, and were left as they are.
- **Unified environment** (umbrella):
  - [docker-compose.yml](../docker-compose.yml): SQL Server, Redis, MinIO (+ `storage-init` job that creates the public `image` bucket), MeiliSearch, MongoDB, with named volumes.
  - [.env.example](../.env.example): local credentials; `/.env` is git-ignored.
  - [config/](../config): `appsettings.Development.json` for the backend, `.env` for chat, `.env.local` for the frontend, all matching `.env.example`.
  - [README.md](../README.md): prerequisites, infra, config mapping, migrations + seeding, how to run each service, URLs, infra management.
- **Sub-repos**: removed `docker-compose.yml` from backend and chat, dropped `docker-up`/`docker-down` from `chat/Makefile`, cleaned `chat/.env.example`, and trimmed backend/chat/frontend READMEs and CLAUDE.md files to point to the umbrella README. The backend README's tech stack now says .NET 7 (the projects target `net7.0`).
- PRs:
  - Backend: https://github.com/Aaron-24DucAnh24/Petopia-Backend/pull/24
  - Chat: https://github.com/Aaron-24DucAnh24/Petopia-Chat/pull/1
  - Frontend: https://github.com/Aaron-24DucAnh24/Petopia-Frontend/pull/1
  - Umbrella: https://github.com/Aaron-24DucAnh24/Petopia/pull/2
- Follow-up: after the submodule PRs are merged, bump the pointers with `[chore] PR-2: bump submodules`.
