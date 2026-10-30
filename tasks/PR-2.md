# PR-2: Switch workflow to English and unify the local environment

**Status:** In progress
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

- [ ] TC1: Grep `CLAUDE.md`, `.claude/`, `tasks/` for Vietnamese diacritics → no matches.
- [ ] TC2: `CLAUDE.md` and the skill both state that comments, commits, PRs and docs must be in English.
- [ ] TC3: Root `docker-compose.yml` parses as valid YAML and defines `database`, `cache`, `storage`, `search`, `mongodb` services with the same ports as before (1433, 6379, 9000/9001, 7700, 27017).
- [ ] TC4: Every `${VAR}` used in `docker-compose.yml` is defined in `.env.example`.
- [ ] TC5: Credentials in `config/` templates (SQL Server, Redis, MinIO, MeiliSearch, MongoDB) match `.env.example`; the chat `JWT_SECRET` matches the backend `Token:AccessTokenKey`.
- [ ] TC6: Root `.env` is git-ignored; `.env.example` and `config/` are tracked.
- [ ] TC7: `backend/docker-compose.yml` and `chat/docker-compose.yml` are removed; `chat/Makefile` has no Docker targets; no sub-repo README/CLAUDE.md tells the reader to run Docker from inside the sub-repo.
- [ ] TC8: `docker compose up -d` at the root starts all five containers and they become healthy.
- [ ] TC9: Every commit on the PR-2 branches (umbrella + submodules) matches `[<type>] PR-2: <summary>`; every PR is titled `PR-2: <task name>` with an empty body.

## Result

<Fill in after coding: summary of changes, main files/modules, how it was verified, PR links.>
