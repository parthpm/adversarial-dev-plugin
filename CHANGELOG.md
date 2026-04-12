# Changelog

## [1.0.3] - 2026-04-12

### Fixed
- Removed `name:` field from commands so the plugin namespace applies correctly — commands now surface as `/adversarial-dev:plan` and `/adversarial-dev:build` (#5)
- Added `allowed-tools` to workflows to prevent mid-workflow permission prompts (#6)
- Reduced wasted review rounds caused by stateless Codex context and blind compliance with reviewer suggestions (#8)

### Changed
- Streamlined plan and build workflows; tightened allowed-tools list (#7)

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
