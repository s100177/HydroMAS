---
name: claude-coder
description: Invoke Claude Code to perform autonomous multi-agent development tasks on a local project. Use when asked to develop, review, fix, refactor, or test code in a repository.
version: 1.0.0
metadata:
  openclaw:
    requires:
      bins:
        - claude
        - git
        - python3
    emoji: "🦞"
    homepage: https://github.com/openclaw/openclaw
---

## Instructions

You are invoking Claude Code CLI to perform autonomous code development.

### Step 1 — Resolve project path

Ask the user (or use context from the conversation) to confirm:
- The absolute path of the project: e.g. `/home/user/HydroMAS`
- The task description: e.g. "add unit tests for the scheduling module"

If both are already clear from context, proceed without asking.

### Step 2 — Confirm git status

Run: `exec git -C {PROJECT_PATH} status --short`

If there are uncommitted changes, warn the user and ask whether to proceed.

### Step 3 — Create a feature branch

Run: `exec git -C {PROJECT_PATH} checkout -b claude/task-$(date +%Y%m%d-%H%M%S)`

Confirm the branch was created successfully.

### Step 4 — Invoke Claude Code with multi-agent pipeline

Run the following command (replace {PROJECT_PATH} and {TASK} with actual values):

`exec claude --project {PROJECT_PATH} --task "{TASK}" --auto-accept --model sonnet`

This will trigger Claude Code's internal multi-agent pipeline:
- Explore agent: maps the codebase
- Architect agent: reviews structure
- Reviewer agent: checks code quality
- Coder agent: implements changes
- Tester agent: validates results

Wait for Claude Code to complete. This may take several minutes.

### Step 5 — Capture results

After Claude Code finishes, run:

`exec git -C {PROJECT_PATH} diff --stat HEAD`

Then run:

`exec git -C {PROJECT_PATH} log --oneline -5`

### Step 6 — Run tests to verify

Run: `exec python3 -m pytest {PROJECT_PATH}/tests -q --tb=short 2>&1 | tail -20`

### Step 7 — Report to user

Summarize:
- What branch was created
- How many files were changed (+lines / -lines)
- Test results (passed/failed)
- Key changes made (from git log)

Ask the user: "Should I push this branch and open a PR, or would you like to review first?"

### Error handling

- If `claude` command not found: tell user to install Claude Code CLI with `npm install -g @anthropic-ai/claude-code`
- If git branch creation fails: try `git -C {PROJECT_PATH} checkout master && git pull` first
- If tests fail after Claude Code runs: report the failures and ask whether to trigger another fix cycle
