# PR-3: Auto-bump submodules from CI

**Status:** In progress
**Scope:** umbrella, backend, frontend, chat

## Description

Today the umbrella repo's submodule pointers are bumped by hand with a `[chore] PR-X: bump <submodule>` commit after each submodule PR is merged. Move that to CI so the umbrella always points at the latest `main` of every sub-repo.

Approach (event-driven, `repository_dispatch`):

1. **Sub-repos (backend, frontend, chat):** add `.github/workflows/notify-umbrella.yml`. On every push to `main` it sends a `submodule-updated` `repository_dispatch` event to `Aaron-24DucAnh24/Petopia`, with the submodule name and the task id (`PR-X`) parsed from the head commit message. Sending the event needs a token with write access to the umbrella repo, stored as the `UMBRELLA_DISPATCH_TOKEN` secret in each sub-repo.
2. **Umbrella:** add `.github/workflows/bump-submodules.yml`. On `submodule-updated` (or a manual `workflow_dispatch`) it checks out the latest umbrella `main`, runs `git submodule update --remote`, and, if any pointer moved, commits `[chore] PR-X: bump <submodules>` and pushes to `main` with `GITHUB_TOKEN`. Runs are serialized with a `concurrency` group so two merges close together don't race. Payload values are passed through env vars and the task id is validated against `^PR-[0-9]+$` before it goes into the commit message.
3. **Task workflow skill:** drop the "bump the submodule pointer by hand" rule (CI does it now) and add a step that pulls the latest umbrella `main` (and syncs submodules to the pinned commits) before any coding starts. Update the "Working with submodules" section of the README to match.

## Test cases

- [ ] TC1: All four workflow files parse as valid YAML and have the expected triggers (`push` to `main` in sub-repos; `repository_dispatch: submodule-updated` + `workflow_dispatch` in the umbrella).
- [ ] TC2: Task id parsing: a head commit message containing `PR-3` (e.g. `PR-3: Auto-bump submodules from CI (#25)`) yields `PR-3`; a message with no task id yields an empty value; the umbrella falls back to `auto` and rejects anything not matching `^PR-[0-9]+$`.
- [ ] TC3: Bump logic: with a submodule behind its remote `main`, the bump script commits `[chore] PR-X: bump <submodule>` containing only the pointer change; with everything up to date it exits 0 without committing.
- [ ] TC4: No untrusted payload value is interpolated with `${{ }}` directly into a `run:` script.
- [ ] TC5: The skill no longer tells anyone to bump submodule pointers by hand, and has a step to pull the latest umbrella `main` before coding.
- [ ] TC6: End to end: after merge and with `UMBRELLA_DISPATCH_TOKEN` set, a push to a sub-repo `main` triggers the umbrella workflow, which pushes a bump commit to umbrella `main`.
- [ ] TC7: Every commit on the PR-3 branches matches `[<type>] PR-3: <summary>`; every PR is titled `PR-3: Auto-bump submodules from CI` with an empty body.

## Result

<Fill in after coding.>
