---
name: task-workflow
description: Mandatory workflow for EVERY request that builds a feature, fixes a bug, edits, refactors or reconfigures anything in the Petopia repo. Create tasks/PR-X.md (task name, description, test cases), code, update the md file, commit as "[type] PR-X: summary", then open a PR titled "PR-X: <task name>" with no description. Use before touching any file.
---

# Task workflow (PR-X)

Applies to every request that changes code, config or docs: new features, bug fixes, edits, refactors, CI... It does not apply to pure questions (explaining code, looking things up) that change no files.

## 0. Language

Write **everything in English**: code comments, commit messages, PR titles, task files, READMEs and other docs. This holds even when the user writes in another language.

## 1. Pick the number X

- X = the highest number among existing `tasks/PR-*.md` files + 1. If there are none, X = 1.
- Also check remote `pr-*` branches so an unfinished task doesn't get its number reused:
  ```bash
  ls tasks/PR-*.md 2>/dev/null
  git branch -a --list '*pr-*'
  ```
- One request = one X. If the user asks for more changes to a task that is still open (not merged), update the existing PR-X.md instead of creating a new number.

## 2. Create the task file BEFORE coding

1. From the latest `main` of the umbrella repo, create a branch `pr-X/<short-slug>` (e.g. `pr-3/adoption-filter`).
2. Create `tasks/PR-X.md` from [template.md](template.md) and fill in:
   - **Task name**: short; it becomes the PR title.
   - **Description**: context, requirements, scope (which service: backend/frontend/chat), planned approach.
   - **Test cases**: a `- [ ]` checklist; each case states the input/action and the expected result. Cover the main path, edge cases and error cases.
   - **Status**: `In progress`.
3. Commit: `[docs] PR-X: add task description`.

## 3. Code

- When the code lives in a submodule (`backend/`, `frontend/`, `chat/`), work in that submodule's repo:
  - `git -C <submodule> checkout main && git -C <submodule> pull`
  - Create a branch with the same name `pr-X/<slug>` in the submodule.
- Make small commits, one per logical step. Every commit (in both the umbrella and submodules) uses the message:
  ```
  [<type>] PR-X: <summary of the change>
  ```
  `<type>` ∈ `feat`, `fix`, `chore`, `refactor`, `docs`, `test`, `ci`, `perf`, `style`, `build`. Example: `[feat] PR-3: add species filter to adoption list`.
- Run the relevant tests/build/lint to verify each test case.

## 4. Update the task file AFTER coding

Update `tasks/PR-X.md`:
- Tick `- [x]` for each passing test case and note how it was verified (test command, manual steps...). Leave cases that failed or couldn't be checked as `- [ ]` with the reason.
- Fill in **Result**: summary of changes, main files/modules affected, PR link for each repo.
- Set **Status** to `Done` (or `Blocked` with the reason).
- Commit: `[docs] PR-X: update task result`.

## 5. Open the PRs

- Push the branch and open a PR in **every repo that changed** (submodules first, umbrella last), all with the same title and **no description**:
  ```bash
  git push -u origin pr-X/<slug>
  gh pr create --base main --title "PR-X: <Task name from the md file>" --body ""
  ```
  For a submodule: run inside the submodule folder (or add `--repo Aaron-24DucAnh24/Petopia-<Service>`).
- The umbrella PR contains `tasks/PR-X.md`. Don't bump a submodule pointer to a commit on an unmerged branch; once the submodule PR is merged into `main`, bump it with a `[chore] PR-X: bump <submodule>` commit.
- Report back to the user with the PR links and any test cases that are not done.
