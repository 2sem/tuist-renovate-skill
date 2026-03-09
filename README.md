# tuist-renovate-skill

Sets up [Renovate](https://renovatebot.com/) automated dependency updates for [Tuist](https://tuist.io/) iOS projects — available as a Claude Code skill or as a standalone guide for any AI assistant.

## Why

GitHub's Dependabot doesn't understand Tuist's dependency syntax. When I first set up Renovate on my Tuist project, it detected **1 out of 12** Swift package dependencies. The other 11 were completely invisible.

Tuist projects declare dependencies in ways standard tools can't parse:

- **`.package(id:)`** — Swift Package Registry format with dot notation (`firebase.firebase-ios-sdk`). Renovate needs to transform dots to slashes to look up the GitHub repo (`firebase/firebase-ios-sdk`), and must handle multiline declarations.
- **`.remote(url:)`** — Tuist-specific syntax that neither Dependabot nor Renovate's built-in Swift manager recognises.

This skill encodes the `customManagers` regex patterns and configuration needed to make Renovate detect all your dependencies — not just the lucky one.

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
