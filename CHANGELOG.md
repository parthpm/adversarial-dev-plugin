# Changelog

## [1.0.2] - 2026-04-07

### Fixed
- Reverted repo flattening: plugin files are back in `plugins/adversarial-dev/` with `source: "./plugins/adversarial-dev"` in marketplace.json. The plugin installer does not handle `source: "./"` (root) — it passes schema validation but fails at path resolution during install.

## [1.0.1] - 2026-04-06

### Fixed
- Moved commands from `skills/` to `commands/` so they appear as `/adversarial-dev:plan` and `/adversarial-dev:build` instead of unnamespaced `/plan` and `/build`
- Flattened repo structure — plugin files now live at root instead of `plugins/adversarial-dev/`

## [1.0.0] - 2026-04-06

### Added
- `/adversarial-dev:plan` — Plan changes with codebase exploration, Codex adversarial review, and simplification pass
- `/adversarial-dev:build` — Implement changes with full tests and Codex adversarial review iterate-until-clean loop
