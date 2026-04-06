---
name: plan
description: Plan a change with codebase exploration, Codex adversarial review, and simplification pass
argument-hint: "<task description>"
disable-model-invocation: true
---

Plan the following change with codebase exploration and adversarial pressure-testing. No code edits — planning only.

**Task:** $ARGUMENTS

## Workflow

### 0. Preflight Check
Before starting, verify that the Codex plugin is available by confirming that `/codex:adversarial-review` is a recognized command. If it is not available, stop immediately and tell the user:

"The adversarial-dev plugin requires the Codex plugin for Claude Code. Install it:
1. `/plugin marketplace add parthpm/codex-plugin-cc`
2. `/plugin install codex@openai-codex`
3. `/reload-plugins`
4. `/codex:setup`

See: https://github.com/openai/codex-plugin-cc"

### 1. Explore
Launch up to 3 Explore agents in parallel to understand the relevant code paths, existing patterns, and test infrastructure. The quality of the plan depends on understanding the codebase first — don't shortcut this.

### 2. Draft Plan
Launch a Plan agent with comprehensive context from exploration. The plan must include:
- **Context**: why this change is needed
- **Implementation steps** with specific file paths and line references
- **Edge cases** and how they're handled
- **Files to modify** including test changes
- **Verification**: how to test end-to-end

Write the plan to a file under `plans/` with a descriptive name.

### 3. Adversarial Review
Invoke `/codex:adversarial-review --wait` with focus text to pressure-test the plan:

```
/codex:adversarial-review --wait "Review the implementation plan for correctness red flags. Verify each claim about current behavior against actual source code — if the plan assumes X but the code does Y, flag it. Focus on: (1) logic errors or incorrect assumptions, (2) over/under-engineering, (3) missed code paths or edge cases. Skip nitpicks, style opinions, and hypothetical concerns."
```

### 4. Iterate
If the adversarial review finds real issues, redesign the affected section of the plan — don't just patch the text. Run `/codex:adversarial-review --wait` again on the revised plan. Repeat until clean.

### 5. Simplification Pass
Before presenting, review the approved plan for unnecessary complexity:
- Can any new flags, enum values, or state variables be eliminated by deriving the answer from data already available at that point in the code?
- Can any special-case branches be folded into the general case?
- Is any step doing work that another step already does?

If you find a simplification, update the plan and run `/codex:adversarial-review --wait` again to verify the simpler version is still correct.

### 6. Present
Present the final plan to the user with:
- What the plan does and why
- What the adversarial review validated (and any issues caught and fixed)
- Any simplifications made in step 5

End with: **Run `/adversarial-dev:build` to implement this plan.**

## Rules
- **No code edits.** Read-only exploration only. The only file you may write is the plan file.
- **Do NOT skip adversarial review** — pressure-testing is the point of this workflow.
- **Ask the user** clarifying questions if the task is ambiguous. Don't make large assumptions.

## Dependencies
Adversarial review is powered by the [Codex plugin for Claude Code](https://github.com/openai/codex-plugin-cc). If the plugin is unavailable, inform the user — the review step cannot be skipped.
