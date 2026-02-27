---
name: requirements-executor
description: Fully autonomous build pipeline. Given a requirements document and project path, automatically plans, implements, tests, and retries until all tasks pass. Use when the user wants to build a feature or project from a specification document with zero manual intervention.
version: 2.0.0
metadata:
  openclaw:
    requires:
      bins:
        - claude
        - git
        - python3
    emoji: "🚀"
---

## Instructions

You are running a fully autonomous development pipeline. Follow these steps precisely and completely. Do not ask the user for input during execution — only report at the end.

---

### PHASE 0 — Collect inputs

Extract from the user's message:
- `PROJECT_PATH`: absolute path to the project (e.g. `/home/user/HydroMAS`)
- `REQUIREMENTS_PATH`: path to the requirements document, OR inline requirements text

If `REQUIREMENTS_PATH` is a file path, read it:
`exec cat {REQUIREMENTS_PATH}`

Store the requirements text as `REQUIREMENTS_CONTENT`.

---

### PHASE 1 — Snapshot current state

Run:
`exec git -C {PROJECT_PATH} status --short`

If uncommitted changes exist, stash them:
`exec git -C {PROJECT_PATH} stash push -m "auto-stash before requirements-executor"`

Create a dedicated branch:
`exec git -C {PROJECT_PATH} checkout -b claude/auto-$(date +%Y%m%d-%H%M%S)`

Run baseline tests and record result:
`exec python3 -m pytest {PROJECT_PATH}/tests -q --tb=no 2>&1 | tail -3`

Store as `BASELINE_TEST_STATUS`.

---

### PHASE 2 — Plan: decompose requirements into tasks

Send requirements to the Planner agent:

`sessions_send("planner", "PROJECT_PATH: {PROJECT_PATH}\n\nREQUIREMENTS:\n{REQUIREMENTS_CONTENT}\n\nReturn ONLY a JSON task array.")`

Wait for the Planner's response. Parse the JSON task list.
Store as `TASK_LIST` (an ordered array of task objects).

If parsing fails, ask Planner to retry once with: "Your previous response was not valid JSON. Please return ONLY the JSON array, no other text."

Log: "📋 Plan created: {N} tasks"

---

### PHASE 3 — Execute tasks in dependency order

Process tasks in order of their `depends_on` dependencies (tasks with no dependencies first, then tasks that depend on completed tasks).

**For each task in TASK_LIST:**

#### 3a. Announce task start

Note: "⚙️ Starting Task {task.id}: {task.title}"

#### 3b. Execute with Claude Code

Run:
`exec claude --project {PROJECT_PATH} --task "{task.description}" --auto-accept --model sonnet`

#### 3c. Commit intermediate work

`exec git -C {PROJECT_PATH} add -A`
`exec git -C {PROJECT_PATH} commit -m "task({task.id}): {task.title}" --allow-empty`

#### 3d. Validate with tests

Run the task's test command:
`exec python3 -m pytest {PROJECT_PATH}/{task.test_command_args} --tb=short 2>&1 | tail -20`

Capture output as `TEST_OUTPUT`.

#### 3e. Retry loop (max 3 attempts)

If tests FAIL:

**Attempt 2**: Run Claude Code with fix instruction:
`exec claude --project {PROJECT_PATH} --task "Fix failing tests. Error output:\n{TEST_OUTPUT}\n\nOriginal task: {task.description}" --auto-accept --model sonnet`

`exec python3 -m pytest {PROJECT_PATH}/{task.test_command_args} --tb=short 2>&1 | tail -20`

**Attempt 3**: If still failing, run with senior-level fix instruction:
`exec claude --project {PROJECT_PATH} --task "Tests still failing after one fix attempt. Do a thorough analysis and fix. Test output:\n{TEST_OUTPUT}" --auto-accept --model sonnet`

`exec python3 -m pytest {PROJECT_PATH}/{task.test_command_args} --tb=short 2>&1 | tail -20`

**If still failing after 3 attempts**: Mark task as FAILED, continue to next task. Record the failure.

#### 3f. Commit after successful validation

If tests pass:
`exec git -C {PROJECT_PATH} add -A`
`exec git -C {PROJECT_PATH} commit -m "✅ task({task.id}): {task.title} — tests passing" --allow-empty`

Note: "✅ Task {task.id} complete"

---

### PHASE 4 — Full regression test

After all tasks are processed, run the complete test suite:
`exec python3 -m pytest {PROJECT_PATH}/tests -q --tb=short 2>&1 | tail -30`

If regression failures are found (tests that passed in BASELINE_TEST_STATUS now fail):

Run one final fix pass:
`exec claude --project {PROJECT_PATH} --task "Fix regression: these tests were passing before but now fail:\n{REGRESSION_FAILURES}" --auto-accept --model sonnet`

`exec python3 -m pytest {PROJECT_PATH}/tests -q --tb=short 2>&1 | tail -10`

---

### PHASE 5 — Generate final report

Collect:
`exec git -C {PROJECT_PATH} diff --stat main 2>/dev/null || git -C {PROJECT_PATH} diff --stat HEAD~{N_TASKS}`
`exec git -C {PROJECT_PATH} log --oneline -{N_TASKS+2}`
`exec python3 -m pytest {PROJECT_PATH}/tests -q --tb=no 2>&1 | tail -5`

Format and send to user:

```
🚀 Autonomous Build Complete
════════════════════════════
📋 Tasks Planned:    {N}
✅ Tasks Succeeded:  {success_count}
❌ Tasks Failed:     {fail_count}

📝 Git Summary:
  Branch: {branch_name}
  {git_diff_stat}

🧪 Final Test Suite:
  {final_test_results}

❌ Failed Tasks (if any):
  - Task {id}: {title} — {reason}

📌 Next Steps:
  Reply "push" to push branch and open a PR
  Reply "show diff" to see full code changes
  Reply "fix {task_id}" to retry a specific failed task
════════════════════════════
```

---

### Error handling

- If `claude` CLI not found: "Install with: npm install -g @anthropic-ai/claude-code"
- If `git` fails: report exact error, do not continue
- If Planner returns no tasks: ask user to provide more specific requirements
- If all tasks fail: run `sessions_send("architect", "All tasks failed. Review requirements and suggest a simpler implementation approach:\n{REQUIREMENTS_CONTENT}")` and relay response to user
