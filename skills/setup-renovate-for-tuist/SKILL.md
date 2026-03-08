---
name: setup-renovate-for-tuist
description: Sets up Renovate automated dependency updates for Tuist iOS projects. Detects integration style (Project.swift-based or Tuist/Package.swift-based), handles registry vs URL packages, and creates renovate.json plus an optional GitHub Actions workflow. Use when you want to automate dependency bump PRs for a Tuist project.
---

# Setup Renovate for Tuist

Renovate automates dependency update PRs for your Tuist iOS project. This skill creates the correct `renovate.json` based on your project's integration style and package format.

## Preflight Checklist

Before starting:
- [ ] Confirm the project uses Tuist (look for `Tuist.swift` or `Tuist/` directory)
- [ ] Identify the integration style (see below)
- [ ] Check whether Tuist registry is enabled
- [ ] Confirm the project is hosted on GitHub (Renovate GitHub App or self-hosted)

## Step 1 — Detect Integration Style

Check where packages are declared:

**Tuist/Package.swift-based** — look for non-empty `dependencies` array:
```swift
// Tuist/Package.swift
let package = Package(
    dependencies: [
        .package(url: "https://github.com/firebase/firebase-ios-sdk", from: "11.8.1")
    ]
)
```

**Project.swift-based** — look for `packages:` arrays in `Projects/*/Project.swift`:
```swift
// Projects/App/Project.swift
let project = Project(
    packages: [
        .remote(url: "https://github.com/firebase/firebase-ios-sdk", requirement: .upToNextMajor(from: "11.8.1"))
    ]
)
```

> A project uses one style only. If `Tuist/Package.swift` has non-empty dependencies → Tuist/Package.swift-based. If `Project.swift` files have non-empty `packages:` → Project.swift-based.

## Step 2 — Detect Package Format

Within the detected files, check whether packages use **URL format** or **registry format**:

| Format | Example |
|--------|---------|
| URL-based | `.package(url: "https://github.com/firebase/firebase-ios-sdk", from: "11.8.1")` |
| Registry-based | `.package(id: "firebase.firebase-ios-sdk", from: "11.8.1")` |
| URL-based (Project.swift) | `.remote(url: "https://github.com/...", requirement: .upToNextMajor(from: "1.0.0"))` |

## Step 3 — Check for mise.toml

Check if `mise.toml` exists in the project root. If yes, Renovate can also update tool versions (tuist, swiftlint, etc.) automatically.

## Step 4 — Create renovate.json

Create `renovate.json` in the project root based on the detected configuration.

---

### Case A: Tuist/Package.swift + URL-based packages (most common)

The Renovate Swift manager natively handles `Package.swift` files. The default pattern `/(^|/)Package\.swift/` already matches `Tuist/Package.swift`.

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:recommended"],
  "packageRules": [
    {
      "matchManagers": ["swift"],
      "groupName": "Swift dependencies",
      "schedule": ["before 9am on monday"]
    }
  ]
}
```

---

### Case B: Tuist/Package.swift + Registry-based packages (id: format)

Renovate's Swift manager only understands `url:`-based packages. For `id:`-based registry packages, add a `customManagers` entry that extracts the package id and maps it to a GitHub datasource.

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:recommended"],
  "customManagers": [
    {
      "customType": "regex",
      "managerFilePatterns": ["/Tuist/Package\\.swift/"],
      "matchStrings": [
        "\\.package\\(id:\\s*\"(?<depName>[^\"]+)\",\\s*from:\\s*\"(?<currentValue>[^\"]+)\""
      ],
      "datasourceTemplate": "github-tags",
      "packageNameTemplate": "{{replace '\\.' '/' depName}}"
    }
  ],
  "packageRules": [
    {
      "matchManagers": ["custom.regex"],
      "groupName": "Swift dependencies",
      "schedule": ["before 9am on monday"]
    }
  ]
}
```

> **Note on dotted repo names**: If a repo name had dots replaced with underscores (e.g., `groue.GRDB_swift`), the `packageNameTemplate` replacement `groue/GRDB_swift` won't match the real GitHub repo `groue/GRDB.swift`. For these packages, add an explicit `packageRules` override:
> ```json
> {
>   "matchPackageNames": ["groue.GRDB_swift"],
>   "packageName": "groue/GRDB.swift"
> }
> ```

---

### Case C: Project.swift-based + URL-based packages

Project.swift uses `.remote(url:)` which is a Tuist-specific syntax, not standard SPM. Renovate's Swift manager won't parse this. Use a `customManagers` regex entry:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:recommended"],
  "customManagers": [
    {
      "customType": "regex",
      "managerFilePatterns": ["/Projects/.*Project\\.swift/", "/Project\\.swift/"],
      "matchStrings": [
        "\\.remote\\(url:\\s*\"https://github\\.com/(?<depName>[^\"]+)\",\\s*requirement:\\s*\\.upToNextMajor\\(from:\\s*\"(?<currentValue>[^\"]+)\""
      ],
      "datasourceTemplate": "github-tags"
    }
  ],
  "packageRules": [
    {
      "matchManagers": ["custom.regex"],
      "groupName": "Swift dependencies",
      "schedule": ["before 9am on monday"]
    }
  ]
}
```

---

### Case D: Mixed (URL + registry packages)

Combine both `customManagers` entries from Case B and Case C as needed.

---

### Adding mise.toml support (optional)

If `mise.toml` exists, add the `mise` manager to the extends list:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:recommended"],
  "enabledManagers": ["swift", "mise", "custom.regex"],
  "packageRules": [
    {
      "matchManagers": ["mise"],
      "groupName": "Dev tools",
      "schedule": ["before 9am on monday"]
    }
  ]
}
```

> The `mise` manager reads `mise.toml` and updates tool versions like `tuist`, `swiftlint`, etc.

## Step 5 — Enable Renovate

Choose one of the two approaches:

### Option A: Renovate GitHub App (recommended)

1. Install the [Renovate GitHub App](https://github.com/apps/renovate) on the repository.
2. Merge the `renovate.json` — Renovate will auto-create an onboarding PR, then begin raising dependency update PRs.

No additional files needed.

### Option B: Self-hosted via GitHub Actions

Create `.github/workflows/renovate.yml`:

```yaml
name: Renovate

on:
  schedule:
    - cron: '0 8 * * 1'  # Every Monday at 8am UTC
  workflow_dispatch:

jobs:
  renovate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: renovatebot/github-action@v40
        with:
          configurationFile: renovate.json
          token: ${{ secrets.RENOVATE_TOKEN }}
```

Set `RENOVATE_TOKEN` in GitHub repository secrets (a PAT with `repo` scope).

## Step 6 — Verify

After merging:
- Renovate creates a **Configure Renovate** PR (if using GitHub App) — merge it
- Renovate opens PRs for outdated packages on the configured schedule
- Check `https://app.renovatebot.com/dashboard` for run logs

## Common Issues

| Issue | Fix |
|-------|-----|
| No PRs created | Confirm `renovate.json` is valid JSON; check app dashboard for errors |
| `customManagers` not matching | Test the regex against your `Package.swift` at regex101.com |
| Registry package PRs have wrong version | Override `packageName` in `packageRules` to point to the correct GitHub repo |
| Renovate opens too many PRs at once | Add `"prConcurrentLimit": 3` to `renovate.json` |
| Package updates break build | Add `"automerge": false` (it's the default — confirm it's not set to `true`) |

## Done Checklist

- [ ] `renovate.json` created in project root with correct config for detected style
- [ ] JSON is valid (no syntax errors)
- [ ] `customManagers` regex tested against actual `Package.swift` content
- [ ] Renovate GitHub App installed OR `.github/workflows/renovate.yml` created
- [ ] `RENOVATE_TOKEN` secret set (self-hosted only)
