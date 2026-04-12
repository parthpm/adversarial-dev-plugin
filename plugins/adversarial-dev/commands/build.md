---
description: Implement changes with full tests and Codex adversarial review
argument-hint: "<task description or plan reference (optional)>"
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Agent, Skill(codex:setup), Skill(codex:adversarial-review), AskUserQuestion
---

Implement the requested changes, verify with tests, and pressure-test with Codex adversarial review.

**Task:** $ARGUMENTS

## Workflow

### 0. Preflight

Run `/codex:setup` to verify the Codex CLI is installed and authenticated. If the skill is not found (Codex plugin not installed), stop and tell the user:

"The adversarial-dev plugin requires the Codex plugin for Claude Code.

1. `/plugin marketplace add parthpm/codex-plugin-cc`
2. `/plugin install codex@openai-codex`
3. `/reload-plugins`
4. `/codex:setup`"

If `/codex:setup` reports CLI or auth issues, stop and relay the fix.

### 1. Understand the Task

Determine what to implement from one of these sources (first match wins):

1. **Plan file** — if the task references a plan or no task is specified, check `plans/` and `.claude/plans/` for plan files. If multiple candidates exist, ask the user which one to use — do not auto-pick. If exactly one exists, read it, then re-read the source files it references to confirm the code still matches its assumptions. If anything has diverged, flag it to the user before proceeding.
2. **Task description** — the user described what to build or fix above.
3. **Uncommitted changes** — if no task and no plan, run `git diff` and `git status` to understand existing work that needs testing and review.

If the task is ambiguous, ask the user before proceeding.

### 2. Implement

Make all changes: production code, tests, and documentation. Do not commit.

### 3. Verify

Run the full test suite — not just targeted tests. Fix any failures before proceeding. If no tests exist or changes are non-code (docs, config, skills), verify by reading the changed files for correctness and consistency.

### 4. Adversarial Review

Invoke `/codex:adversarial-review --wait` to pressure-test the implementation:

```
/codex:adversarial-review --wait "Review the implementation changes for correctness regressions, broken existing behavior, missing edge-case coverage, and whether tests cover the identified risks."
```

If this fails with a `disable-model-invocation` error, stop and tell the user:

"The Codex plugin blocks programmatic invocation. Switch to the fork:

1. `/plugin marketplace add parthpm/codex-plugin-cc`
2. `/plugin update codex@openai-codex`
3. `/reload-plugins`

Tracking: https://github.com/openai/codex-plugin-cc/pull/156

**Warning:** Implementation changes are uncommitted and have NOT been adversarially reviewed. Run `git diff` to inspect, `git stash -u` to set aside (includes untracked files)."

If it fails for any other reason, stop and warn the user that changes are unreviewed, then suggest `/codex:setup` to diagnose.

### 5. Iterate

Each Codex review starts fresh — the evolving diff is its primary context. If a plan file exists, maintain a `## Review Log` there with what was flagged, fixed, and kept with reasons.

Evaluate each finding before acting on it — not every finding warrants a change.

Re-run step 4 after meaningful changes. Done when a round surfaces nothing new, or nothing you can't reasonably justify keeping.

### 6. Report

Summarize:
- What was implemented
- Test results
- What the adversarial review validated
- Changes are uncommitted and ready for user review

## Rules

- **Always run the full test suite** — not just targeted tests.
- **Do NOT skip adversarial review** — it catches what tests miss.
- **Do NOT commit or push** — leave that to the user.
