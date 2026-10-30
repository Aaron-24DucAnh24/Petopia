# PR-3: Auto-bump submodules from CI

**Status:** Done
**Scope:** umbrella, backend, frontend, chat

## Description

Today the umbrella repo's submodule pointers are bumped by hand with a `[chore] PR-X: bump <submodule>` commit after each submodule PR is merged. Move that to CI so the umbrella always points at the latest `main` of every sub-repo.

Approach (event-driven, `repository_dispatch`):

1. **Sub-repos (backend, frontend, chat):** add `.github/workflows/notify-umbrella.yml`. On every push to `main` it sends a `submodule-updated` `repository_dispatch` event to `Aaron-24DucAnh24/Petopia`, with the submodule name and the task id (`PR-X`) parsed from the head commit message. Sending the event needs a token with write access to the umbrella repo, stored as the `UMBRELLA_DISPATCH_TOKEN` secret in each sub-repo.
2. **Umbrella:** add `.github/workflows/bump-submodules.yml`. On `submodule-updated` (or a manual `workflow_dispatch`) it checks out the latest umbrella `main`, runs `git submodule update --remote`, and, if any pointer moved, commits `[chore] PR-X: bump <submodules>` and pushes to `main` with `GITHUB_TOKEN`. Runs are serialized with a `concurrency` group so two merges close together don't race. Payload values are passed through env vars and the task id is validated against `^PR-[0-9]+$` before it goes into the commit message.
3. **Task workflow skill:** drop the "bump the submodule pointer by hand" rule (CI does it now) and add a step that pulls the latest umbrella `main` (and syncs submodules to the pinned commits) before any coding starts. Update the "Working with submodules" section of the README to match.

## Test cases

- [x] TC1: All four workflow files parse as valid YAML and have the expected triggers (`push` to `main` in sub-repos; `repository_dispatch: submodule-updated` + `workflow_dispatch` in the umbrella).
  - Verified: parsed with the `yaml` npm package and printed `on` / `permissions` for each file. actionlint was not run (not installed locally).
- [x] TC2: Task id parsing: a head commit message containing `PR-3` (e.g. `PR-3: Auto-bump submodules from CI (#25)`) yields `PR-3`; a message with no task id yields an empty value; the umbrella drops the `PR-X: ` prefix for anything not matching `^PR-[0-9]+$`.
  - Verified: ran both snippets under `bash -eo pipefail` (the Actions default shell). Squash-merge titles and multi-line `[feat] PR-12: ...` messages yield the id. No id and an empty message yield `""` without failing the step. `PR-3; rm -rf /`, `PR-3$(id)`, `auto` and `""` all fall back to `[chore] bump <submodules>`.
- [x] TC3: Bump logic: with a submodule behind its remote `main`, the bump script commits `[chore] PR-X: bump <submodule>` containing only the pointer change; with everything up to date it exits 0 without committing.
  - Verified: bare clone of the umbrella with `main` reset to `7c16207` (old pins), cloned `-b main`, submodules initialised with `--depth=1` like `actions/checkout`, then ran the job's steps. The first run pushed `[chore] PR-3: bump backend, chat, frontend` (3 gitlink changes only, author `github-actions[bot]`). The second run printed "Submodules are already up to date." and pushed nothing.
- [x] TC4: No untrusted payload value is interpolated with `${{ }}` directly into a `run:` script.
  - Verified: `grep '${{'` over all workflows. Expressions only appear under `env:` (`client_payload.task`, `inputs.task`, `head_commit.message`, the secret).
- [x] TC5: The skill no longer tells anyone to bump submodule pointers by hand, and has a step to pull the latest umbrella `main` before coding.
  - Verified: new step "1. Pull the latest umbrella `main`" in `SKILL.md` (and step 0 in `CLAUDE.md`). Step 6 now says not to bump by hand and points to the CI workflow. `grep -i bump` finds no manual bump instructions in the skill, `CLAUDE.md` or `README.md`.
- [ ] TC6: End to end: after merge and with `UMBRELLA_DISPATCH_TOKEN` set, a push to a sub-repo `main` triggers the umbrella workflow, which pushes a bump commit to umbrella `main`.
  - Not verified: needs the PRs merged and the secret created in each sub-repo. Check it by merging a sub-repo PR, then looking at **Actions → Bump submodules** in the umbrella repo.
- [x] TC7: Every commit on the PR-3 branches matches `[<type>] PR-3: <summary>`; every PR is titled `PR-3: Auto-bump submodules from CI` with an empty body.
  - Verified: `git log --format=%s main..HEAD` in each repo; PRs created with `--body ""`.

## Result

- **Umbrella**: [.github/workflows/bump-submodules.yml](../.github/workflows/bump-submodules.yml) runs on `repository_dispatch: submodule-updated` or by hand (`workflow_dispatch`, with an optional `task` input). It checks out the latest `main` and runs `git submodule update --remote`. If any pointer moved, it commits `[chore] PR-X: bump <submodules>` as `github-actions[bot]` and pushes with `GITHUB_TOKEN`. Runs are serialized through a `concurrency` group.
- **Sub-repos**: `.github/workflows/notify-umbrella.yml` in backend, frontend and chat sends the dispatch on every push to `main`, with the submodule name and the `PR-X` parsed from the head commit message. The backend `.gitignore` no longer ignores `/.github`, so the workflow can be tracked.
- **Workflow docs**: the [task-workflow skill](../.claude/skills/task-workflow/SKILL.md) has a new step 1 (pull the latest umbrella `main` + `git submodule update --init --recursive` before anything else) and no longer asks for manual bump commits. [CLAUDE.md](../CLAUDE.md) and the README's "Working with submodules" section match, and the README documents the `UMBRELLA_DISPATCH_TOKEN` setup.
- **Manual setup required**: create a fine-grained PAT scoped to `Aaron-24DucAnh24/Petopia` with Contents: Read and write, and add it as the `UMBRELLA_DISPATCH_TOKEN` secret in Petopia-Backend, Petopia-Frontend and Petopia-Chat.
- PRs:
  - Backend: https://github.com/Aaron-24DucAnh24/Petopia-Backend/pull/25
  - Frontend: https://github.com/Aaron-24DucAnh24/Petopia-Frontend/pull/2
  - Chat: https://github.com/Aaron-24DucAnh24/Petopia-Chat/pull/2
  - Umbrella: (this PR)
