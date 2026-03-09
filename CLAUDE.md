# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **Claude Code skill** repository that provides the `/setup-renovate-for-tuist` slash command. It helps automate Renovate dependency update configuration for Tuist iOS projects.

## Structure

```
skills/
  setup-renovate-for-tuist/
    SKILL.md          # The skill definition — frontmatter + procedural instructions
README.md             # Installation and usage docs
```

The skill is distributed via `npx skills add 2sem/tuist-renovate-skill` and installed into Claude Code's skills directory.

## Skill Architecture

`SKILL.md` is the single source of truth. It contains:
- **Frontmatter** (`name`, `description`) — consumed by the skills CLI
- **Step-by-step instructions** for the Claude agent to follow when `/setup-renovate-for-tuist` is invoked

The skill's logic is entirely in prose and JSON examples within `SKILL.md`. There is no build system, no tests, and no runtime code.

## Skill Logic Summary

When invoked, the skill:
1. Detects integration style — `Tuist/Package.swift` vs `Project.swift`-based packages
2. Detects package format — URL-based (`.package(url:)`, `.remote(url:)`) vs registry-based (`.package(id:)`)
3. Detects `mise.toml` for tool version management
4. Generates the correct `renovate.json` (4 cases: A/B/C/D) based on detected style
5. Optionally creates `.github/workflows/renovate.yml` for self-hosted Renovate

## Key Decisions

- **Case A** (Tuist/Package.swift + URL): Renovate's native Swift manager handles this — no `customManagers` needed.
- **Case B** (Tuist/Package.swift + registry `id:`): Requires `customManagers` regex; `packageNameTemplate` converts dotted id to `owner/repo` format (dots → slashes). Repos with underscored names (e.g. `groue.GRDB_swift`) need explicit `packageName` overrides.
- **Case C** (Project.swift + URL): Requires `customManagers` regex for `.remote(url:)` Tuist-specific syntax.
- **Case D**: Combine Case B + C managers.

## Editing Guidelines

When updating `SKILL.md`:
- Keep the frontmatter `description` concise — it appears in skill listings.
- Regex patterns in `matchStrings` must be tested against real `Package.swift` content before updating (use regex101.com).
- JSON examples must be valid — validate before committing.
- The `schedule`, `groupName`, and `prConcurrentLimit` values are recommendations; the skill prompts users to adjust as needed.
