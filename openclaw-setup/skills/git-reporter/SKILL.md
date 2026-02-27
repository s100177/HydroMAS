---
name: git-reporter
description: Generate a development progress report from a git repository. Shows recent commits, changes, test status, and branch activity. Use when asked for a project status update or development summary.
version: 1.0.0
metadata:
  openclaw:
    requires:
      bins:
        - git
        - python3
    emoji: "📊"
---

## Instructions

### Step 1 — Get project path

Use project path from context, or ask user: "Which project directory should I report on?"

Default: use current working directory.

### Step 2 — Collect git statistics

Run each of these:

`exec git -C {PROJECT_PATH} log --oneline --since="7 days ago" | wc -l`

`exec git -C {PROJECT_PATH} log --oneline -10 --format="%h %s (%cr)"`

`exec git -C {PROJECT_PATH} branch -a | grep -E "claude/|codex/" | head -10`

`exec git -C {PROJECT_PATH} diff --stat HEAD~3 HEAD 2>/dev/null || echo "Not enough commits"`

### Step 3 — Run test suite

`exec python3 -m pytest {PROJECT_PATH}/tests -q --tb=no 2>&1 | tail -5`

### Step 4 — Count code metrics

`exec find {PROJECT_PATH} -name "*.py" -not -path "*/.*" | xargs wc -l 2>/dev/null | tail -1`

### Step 5 — Format and send report

Format the report as:

```
📊 Project Report: {PROJECT_NAME}
━━━━━━━━━━━━━━━━━━━━━━━━
🔀 Recent Activity (7 days): {N} commits
📝 Last 10 Commits:
  {commit list}

🌿 Agent Branches:
  {branch list}

✅ Test Suite: {PASS/FAIL — X passed, Y failed}
📏 Total Lines of Code: {N}
━━━━━━━━━━━━━━━━━━━━━━━━
```

Send this report to the user.
