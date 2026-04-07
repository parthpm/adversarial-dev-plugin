---
description: Plan a change with codebase exploration, Codex adversarial review, and simplification pass
argument-hint: "<task description>"
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git:*), Agent, Skill(codex:setup), Skill(codex:adversarial-review), AskUserQuestion
---

Plan the following change with codebase exploration and adversarial pressure-testing. No code edits — planning only.

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

### 1. Explore

Launch up to 3 Explore agents in parallel to understand the relevant code paths, existing patterns, and test infrastructure. The quality of the plan depends on understanding the codebase first — don't shortcut this.

### 2. Draft Plan

Write the plan to `plans/YYYY-MM-DD-<task-slug>.md`. The plan must include:

- **Context**: why this change is needed and what problem it solves
- **Implementation steps**: specific file paths and line references — no "implement validation" or "handle edge cases" hand-waving
- **Edge cases**: enumerate each one and how it's handled
- **Files to modify**: every file, including tests
- **Verification**: concrete commands or steps to test end-to-end

The plan must not contain placeholders like "TBD", "TODO", "similar to above", or vague phrases that defer decisions.

### 3. Adversarial Review

Invoke `/codex:adversarial-review --wait` to pressure-test the plan:

```
/codex:adversarial-review --wait "Review the plan file in plans/ against the actual source code — flag incorrect assumptions about current behavior, missed code paths, and over- or under-engineering."
```

If this fails with a `disable-model-invocation` error, stop and tell the user:

"The Codex plugin blocks programmatic invocation. Switch to the fork:

1. `/plugin marketplace add parthpm/codex-plugin-cc`
2. `/plugin update codex@openai-codex`
3. `/reload-plugins`

Tracking: https://github.com/openai/codex-plugin-cc/pull/156"

If it fails for any other reason, stop and suggest `/codex:setup` to diagnose. The plan file is kept — the user can re-run the review after fixing the issue.

### 4. Iterate

If the adversarial review finds real issues, redesign the affected section — don't just patch the text. Run `/codex:adversarial-review --wait` again on the revised plan. Repeat until clean.

### 5. Simplify

Before presenting, review the approved plan for unnecessary complexity:
- Can any new flags or state variables be eliminated by deriving the answer from data already available?
- Can any special-case branches be folded into the general case?
- Is any step doing work that another step already does?

If you simplify, update the plan and run `/codex:adversarial-review --wait` again to verify.

### 6. Present

Present the final plan to the user with:
- What the plan does and why
- What the adversarial review caught and how it was fixed
- Any simplifications made

Then ask: **Ready to implement? Run `/adversarial-dev:build` to start.**

## Rules

- **No code edits.** The only file you may write is the plan file.
- **Do NOT skip adversarial review** — pressure-testing is the point of this workflow.
- **Ask the user** if the task is ambiguous. Don't make large assumptions.
