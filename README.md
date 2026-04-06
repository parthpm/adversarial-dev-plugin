# adversarial-dev

Plan and build with cross-model adversarial review.

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that adds two workflow skills — `/adversarial-dev:plan` and `/adversarial-dev:build` — backed by [Codex](https://github.com/openai/codex) adversarial review. A different model (GPT via Codex) reviews Claude's output at every step, catching what same-model review cannot.

## How It Works

Most AI code review is same-model: Claude reviews Claude's output. This has a blind spot — the reviewer shares the same biases, training artifacts, and failure modes as the author.

Cross-model adversarial review breaks this by routing review through an independent model (GPT via Codex CLI). The reviewer has different strengths, different blind spots, and no shared context with the author — so it catches assumptions, logic errors, and missed edge cases that same-model review systematically misses.

```
┌─────────────────────────────────────────────────────────────┐
│                    /adversarial-dev:plan                     │
│                                                             │
│  Explore ──► Draft Plan ──► Codex Review ──► Simplify       │
│                              ▲       │                      │
│                              └───────┘                      │
│                            iterate until clean              │
│                                    │                        │
│                              plan file (plans/)             │
└────────────────────────────────────┬────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────┐
│                   /adversarial-dev:build                     │
│                                                             │
│  Read Plan ──► Implement ──► Test ──► Codex Review          │
│                                        ▲       │            │
│                                        └───────┘            │
│                                      iterate until clean    │
└─────────────────────────────────────────────────────────────┘
```

The plan phase produces a file. The build phase consumes it. Both phases are independently pressure-tested by Codex adversarial review before proceeding.

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- [Codex CLI](https://github.com/openai/codex) — `npm install -g @openai/codex`
- [Codex plugin for Claude Code](https://github.com/openai/codex-plugin-cc)
- Node.js 22+
- OpenAI account with Codex access

## Installation

> **Note:** The official Codex plugin blocks programmatic invocation of `/codex:adversarial-review` ([openai/codex-plugin-cc#156](https://github.com/openai/codex-plugin-cc/pull/156) fixes this). Until that PR merges, install from the fork below.

### Step 1: Install Codex plugin (fork with fix)

```
/plugin marketplace add parthpm/codex-plugin-cc
/plugin install codex@openai-codex
/codex:setup
```

### Step 2: Install adversarial-dev plugin

```
/plugin marketplace add parthpm/adversarial-dev-plugin
/plugin install adversarial-dev@parthpm-adversarial-dev
```

### Step 3: Reload

```
/reload-plugins
```

### After openai/codex-plugin-cc#156 merges

Switch to the official Codex source:

```
/plugin marketplace add openai/codex-plugin-cc
/plugin update codex@openai-codex
```

The plugin identity (`openai-codex`) is the same in both the fork and official repo, so this is a seamless switch.

## Usage

### Plan a change

```
/adversarial-dev:plan add rate limiting to the /api/upload endpoint
```

This explores the codebase, drafts an implementation plan, pressure-tests it with Codex adversarial review, simplifies, and presents the result. No code is modified — only a plan file is written to `plans/`.

### Build from a plan

```
/adversarial-dev:build
```

With no arguments, it picks up the most recent plan from `plans/` or `.claude/plans/`. You can also give it a task directly:

```
/adversarial-dev:build fix the race condition in the connection pool
```

This implements the changes, runs the full test suite, pressure-tests with Codex adversarial review, and iterates until clean. Changes are left uncommitted for your review.

## Known Limitation

The Codex plugin's `adversarial-review` skill has `disable-model-invocation: true`, which prevents programmatic invocation. PR [openai/codex-plugin-cc#156](https://github.com/openai/codex-plugin-cc/pull/156) removes this restriction. Until it merges, use the fork marketplace (`parthpm/codex-plugin-cc`) as shown in the installation instructions above.

## License

[MIT](LICENSE)
