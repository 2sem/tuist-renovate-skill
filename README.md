# tuist-renovate-skill

Sets up [Renovate](https://renovatebot.com/) automated dependency updates for [Tuist](https://tuist.io/) iOS projects — available as a Claude Code skill or as a standalone guide for any AI assistant.

## What it does

- Detects your Tuist integration style (`Tuist/Package.swift` vs `Project.swift`-based)
- Handles URL-based and Tuist registry (`id:`-based) packages
- Generates the correct `renovate.json` for your project
- Optionally sets up a GitHub Actions self-hosted Renovate workflow
- Supports `mise.toml` tool version updates

## Install

```bash
npx skills add 2sem/tuist-renovate-skill
```

Or install globally (available in all projects):

```bash
npx skills add 2sem/tuist-renovate-skill -g
```

## Usage

In Claude Code, run:

```
/setup-renovate-for-tuist
```

## Skill

| Slash command | Description |
|---|---|
| `/setup-renovate-for-tuist` | Sets up Renovate for a Tuist iOS project |
