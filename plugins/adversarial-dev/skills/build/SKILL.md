---
name: build
model: sonnet
disable-model-invocation: true
argument-hint: "<task description or plan reference (optional)>"
description: Implement changes with full tests and Codex adversarial review
---

Implement the requested changes, verify with tests, and pressure-test with Codex adversarial review.

**Task:** $ARGUMENTS

## Workflow

### 0. Preflight Check

**0a. Plugin presence** — Confirm `/codex:adversarial-review` is a recognized skill. If absent, stop immediately and tell the user:

"The adversarial-dev plugin requires the Codex plugin for Claude Code.

**Prerequisites:**
- Node.js 22+
- Codex CLI: `npm install -g @openai/codex`
- OpenAI account: `codex login`

**Install the plugin:**
1. `/plugin marketplace add parthpm/codex-plugin-cc`
2. `/plugin install codex@openai-codex`
3. `/reload-plugins`
4. `/codex:setup`

See: https://github.com/openai/codex-plugin-cc"

**0b. CLI health** — Run `/codex:setup` to verify the Codex CLI is installed and authenticated. If it reports issues, stop and relay the specific fix to the user.

### 1. Understand the Task
Use the task description above and determine what to implement from one of these sources (in priority order):
1. **A plan file** — if the task references a plan or no task is specified, check `plans/` and `.claude/plans/` for the most recent plan matching the context. Read the plan, then re-read the source files it references to confirm the code still matches its assumptions. If anything has diverged (lines shifted, code changed), flag it to the user before proceeding.
2. **Task description** — the user described what to build or fix above.
3. **Uncommitted changes** — if no task is specified and no plan exists, run `git diff` and `git status` to understand existing work that needs testing and review.

If the task is ambiguous, ask the user before proceeding.

### 2. Pipeline Gate
Invoke `/codex:adversarial-review --wait` to validate the adversarial review pipeline and review task scope:

```
/codex:adversarial-review --wait "Review the task scope and plan for risks — flag ambiguities, unstated assumptions, or missing prerequisite changes that should be addressed before implementation begins."
```

If this fails with a `disable-model-invocation` error, stop and tell the user:

"The official Codex plugin blocks programmatic invocation of `/codex:adversarial-review`. Switch to the fork that removes this restriction:

1. `/plugin marketplace add parthpm/codex-plugin-cc`
2. `/plugin update codex@openai-codex`
3. `/reload-plugins`

Tracking: https://github.com/openai/codex-plugin-cc/pull/156"

If it fails for any other reason (API error, timeout, CLI crash), stop and suggest the user run `/codex:setup` to diagnose.

If it succeeds, use the review output to inform implementation.

### 3. Implement
Make **all** changes: production code, tests, and documentation. Do **not** commit.

### 4. Verify
If the changes touch code with tests, run the full test suite — not just targeted tests. Fix any failures before proceeding. For non-code changes (skills, docs, config), verify by reading the changed files and checking for internal consistency.

### 5. Adversarial Review
Invoke `/codex:adversarial-review --wait` to pressure-test the implementation:

```
/codex:adversarial-review --wait "Find correctness bugs — wrong results, misclassified behavior, broken existing functionality. Check that edge cases identified during design review have test coverage. Look for code paths where old behavior leaks through. Skip nitpicks, style opinions, and manufactured concerns."
```

### 6. Iterate
If the adversarial review finds real issues, fix them, re-run full tests, and run `/codex:adversarial-review --wait` again. Repeat until clean.

### 7. Report
Summarize:
- What was implemented
- Test results (pass count)
- What the adversarial review validated
- Changes are ready to commit

## Rules
- **Always run the full test suite** after implementation — not just targeted tests.
- **Do NOT skip adversarial review** — it catches what tests miss.
- **Do NOT commit or push** — leave that to the user.

## Dependencies
Adversarial review is powered by the [Codex plugin for Claude Code](https://github.com/openai/codex-plugin-cc). If the plugin is unavailable, inform the user — the review step cannot be skipped.
