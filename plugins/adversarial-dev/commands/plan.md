---
description: Plan a change with codebase exploration, Codex adversarial review, and simplification pass
argument-hint: "<task description>"
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git:*), Agent, Skill(codex:setup), Skill(codex:adversarial-review), AskUserQuestion
---

Plan the following change with codebase exploration and adversarial pressure-testing. No code edits — planning only.

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

### 1. Explore
Launch up to 3 Explore agents in parallel to understand the relevant code paths, existing patterns, and test infrastructure. The quality of the plan depends on understanding the codebase first — don't shortcut this.

### 2. Pipeline Gate
Invoke `/codex:adversarial-review --wait` to validate the adversarial review pipeline and review exploration completeness:

```
/codex:adversarial-review --wait "Review the exploration findings for completeness — flag missed code paths, unexamined edge cases, or integration points that should be investigated before drafting a plan."
```

If this fails with a `disable-model-invocation` error, stop and tell the user:

"The official Codex plugin blocks programmatic invocation of `/codex:adversarial-review`. Switch to the fork that removes this restriction:

1. `/plugin marketplace add parthpm/codex-plugin-cc`
2. `/plugin update codex@openai-codex`
3. `/reload-plugins`

Tracking: https://github.com/openai/codex-plugin-cc/pull/156"

If it fails for any other reason (API error, timeout, CLI crash), stop and suggest the user run `/codex:setup` to diagnose.

If it succeeds, use the review output to fill exploration gaps before drafting the plan.

### 3. Draft Plan
Launch a Plan agent with comprehensive context from exploration and the Pipeline Gate review. The plan must include:
- **Context**: why this change is needed
- **Implementation steps** with specific file paths and line references
- **Edge cases** and how they're handled
- **Files to modify** including test changes
- **Verification**: how to test end-to-end

Write the plan to a file under `plans/` with a descriptive name.

### 4. Adversarial Review
Invoke `/codex:adversarial-review --wait` with focus text to pressure-test the plan:

```
/codex:adversarial-review --wait "Review the implementation plan for correctness red flags. Verify each claim about current behavior against actual source code — if the plan assumes X but the code does Y, flag it. Focus on: (1) logic errors or incorrect assumptions, (2) over/under-engineering, (3) missed code paths or edge cases. Skip nitpicks, style opinions, and hypothetical concerns."
```

### 5. Iterate
If the adversarial review finds real issues, redesign the affected section of the plan — don't just patch the text. Run `/codex:adversarial-review --wait` again on the revised plan. Repeat until clean.

### 6. Simplification Pass
Before presenting, review the approved plan for unnecessary complexity:
- Can any new flags, enum values, or state variables be eliminated by deriving the answer from data already available at that point in the code?
- Can any special-case branches be folded into the general case?
- Is any step doing work that another step already does?

If you find a simplification, update the plan and run `/codex:adversarial-review --wait` again to verify the simpler version is still correct.

### 7. Present
Present the final plan to the user with:
- What the plan does and why
- What the adversarial review validated (and any issues caught and fixed)
- Any simplifications made in step 6

End with: **Run `/adversarial-dev:build` to implement this plan.**

## Rules
- **No code edits.** Read-only exploration only. The only file you may write is the plan file.
- **Do NOT skip adversarial review** — pressure-testing is the point of this workflow.
- **Ask the user** clarifying questions if the task is ambiguous. Don't make large assumptions.

## Dependencies
Adversarial review is powered by the [Codex plugin for Claude Code](https://github.com/openai/codex-plugin-cc). If the plugin is unavailable, inform the user — the review step cannot be skipped.
