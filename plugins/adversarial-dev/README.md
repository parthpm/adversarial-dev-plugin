# adversarial-dev plugin

Two workflow skills for adversarial development with cross-model Codex review.

## Skills

### `/adversarial-dev:plan`

Read-only planning workflow. Explores the codebase, drafts an implementation plan, and pressure-tests it with Codex adversarial review until clean. Writes the final plan to `plans/`.

### `/adversarial-dev:build`

Implementation workflow. Reads a plan file (or takes a task description), implements changes with tests, and pressure-tests with Codex adversarial review until clean. Leaves changes uncommitted.

## How they connect

```
/adversarial-dev:plan  ──►  plan file (plans/)  ──►  /adversarial-dev:build
```

The plan phase produces a file. The build phase consumes it. Both are independently reviewed by Codex.

## Setup

See the [root README](../../README.md) for installation instructions and prerequisites.
