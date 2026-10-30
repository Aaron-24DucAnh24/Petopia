# PR-1: Set up task workflow

**Status:** Done
**Scope:** umbrella

## Description

Standardize how tasks are done in the Petopia repo so that every change comes with documentation and is easy to follow:

- Add a `tasks/` folder. Each request (feature, bug fix, change...) gets a `tasks/PR-X.md` file, with X counting up from 1, containing the task name, a detailed description, test cases and which cases are done.
- Create the task file before coding, update it after coding.
- Commit message: `[<type>] PR-X: <summary of the change>`.
- PR titled `PR-X: <task name>`, with no description.

Approach:

- The `.claude/skills/task-workflow/` skill (SKILL.md + template.md) describes each step in detail, including how to handle code that lives in a submodule.
- The root `CLAUDE.md` summarizes the rules and points to the skill, so the workflow is loaded in every session.

## Test cases

- [x] TC1: Open a new session in the repo → the `task-workflow` skill shows up in the list of available skills.
  - Verified: Claude Code picked up the skill right after the file was created and listed it.
- [x] TC2: The root `CLAUDE.md` has a "Mandatory workflow" section pointing to the skill → the rules are loaded every session.
  - Verified: re-read `CLAUDE.md`.
- [x] TC3: No `tasks/PR-*.md` file and no `pr-*` branch exist yet → the first task is numbered PR-1.
  - Verified: `git branch -a --list '*pr-*'` returned no branches, `tasks/` had no task file.
- [x] TC4: Every commit on branch `pr-1/task-workflow` has the form `[<type>] PR-1: <summary>`.
  - Verified: `git log --oneline main..pr-1/task-workflow`.
- [x] TC5: The PR is created with the title `PR-1: Thiết lập task workflow` and an empty description.
  - Verified: `gh pr view 1 --json title,body` returned `body: ""`.

## Result

- Added the [task-workflow](../.claude/skills/task-workflow/SKILL.md) skill and a [template](../.claude/skills/task-workflow/template.md) for task files.
- Added [CLAUDE.md](../CLAUDE.md) summarizing the mandatory workflow.
- Added the `tasks/` folder, starting with this file.
- PR: https://github.com/Aaron-24DucAnh24/Petopia/pull/1
- Note: this file was translated to English in PR-2. The merged PR title above is kept as it was created.
