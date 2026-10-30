# Petopia

Umbrella repo; the code lives in the `backend/`, `frontend/` and `chat/` submodules (see [README.md](README.md)). Local infrastructure for all services is defined once in the root [docker-compose.yml](docker-compose.yml); sub-repos don't ship their own.

## Language

Everything is written in **English**: code comments, commit messages, PR titles, task files, READMEs and other docs — even when the request is in another language.

## Mandatory workflow

Every request to build a feature / fix a bug / change something follows the [task-workflow](.claude/skills/task-workflow/SKILL.md) skill:

0. Pull the latest umbrella `main` (`git pull && git submodule update --init --recursive`).
1. Create `tasks/PR-X.md` (X counts up from 1): task name, detailed description, test cases.
2. Code, then update the md file (completed test cases, result).
3. Commit: `[<type>] PR-X: <summary>` (type: feat, fix, chore, refactor, docs, test, ci...).
4. Open a PR titled `PR-X: <task name>`, with no description.
