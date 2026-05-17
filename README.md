# kokorolx/shared-workflows

Centralised GitHub Actions reusable workflows for all `kokorolx` repos.

## Usage

Pin to a major version tag. Never use `@main`.

```yaml
uses: kokorolx/shared-workflows/.github/workflows/<workflow>.yml@v1
```

---

## Workflows

### `publish-beta.yml`

Publishes a beta release to npm and creates a GitHub pre-release.

**Trigger:** Call from a workflow triggered on push to your beta branch.

| Input | Type | Default | Description |
|---|---|---|---|
| `package-name` | string | **required** | npm package name (e.g. `nano-brain`, `@kokorolx/ai-sandbox-wrapper`) |
| `beta-branch` | string | `beta` | Branch that triggers the beta publish |
| `node-version` | string | `22` | Node.js version |
| `build-command` | string | `""` | Optional pre-publish build step (e.g. `npm run build:web`) |

| Secret | Required | Description |
|---|---|---|
| `NPM_TOKEN` | ✅ | npm publish token |

**Example:**
```yaml
on:
  push:
    branches: [develop]

jobs:
  publish:
    uses: kokorolx/shared-workflows/.github/workflows/publish-beta.yml@v1
    with:
      package-name: "nano-brain"
      beta-branch: develop
      build-command: "npm run build:web"
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

---

### `publish-stable.yml`

Auto-detects semver bump from conventional commits, bumps `package.json`, updates `CHANGELOG.md`, commits, tags, publishes to npm `@latest`, and creates a GitHub release.

Includes bot guard to prevent infinite loops from its own commits.

| Input | Type | Default | Description |
|---|---|---|---|
| `package-name` | string | **required** | npm package name |
| `node-version` | string | `22` | Node.js version |
| `build-command` | string | `""` | Optional pre-publish build step |
| `changelog-insert-marker` | string | `All notable changes to this project will be documented in this file.` | String after which new CHANGELOG entries are inserted |

| Secret | Required | Description |
|---|---|---|
| `NPM_TOKEN` | ✅ | npm publish token |

**Semver detection:**
- `major` — any commit with `!` (e.g. `feat!:`) or `BREAKING CHANGE` in body
- `minor` — any `feat:` commit
- `patch` — everything else

**Example:**
```yaml
on:
  push:
    branches: [master]

jobs:
  publish:
    uses: kokorolx/shared-workflows/.github/workflows/publish-stable.yml@v1
    with:
      package-name: "@kokorolx/ai-sandbox-wrapper"
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

---

### `lint.yml`

Runs shellcheck and an optional hardcoded-secret pattern scan.

| Input | Type | Default | Description |
|---|---|---|---|
| `shellcheck-dirs` | string | `lib` | Space-separated directories to scan |
| `shellcheck-severity` | string | `warning` | Minimum severity: `error`, `warning`, `info`, `style` |
| `run-security-scan` | boolean | `true` | Run hardcoded secret pattern scan |

No secrets required.

**Example:**
```yaml
on: [push, pull_request]

jobs:
  lint:
    uses: kokorolx/shared-workflows/.github/workflows/lint.yml@v1
    with:
      shellcheck-dirs: "lib bin"
```

---

### `gemini-review.yml`

Posts an AI code review comment on pull requests using the Gemini API.

| Input | Type | Default | Description |
|---|---|---|---|
| `model` | string | `gemini-2.0-flash` | Gemini model to use |
| `max-files` | number | `20` | Max changed files to include in review |

| Secret | Required | Description |
|---|---|---|
| `GEMINI_API_KEY` | ✅ | Google AI Studio API key |

**Example:**
```yaml
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    uses: kokorolx/shared-workflows/.github/workflows/gemini-review.yml@v1
    secrets:
      GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
```

---

## Versioning

| Tag | Meaning |
|---|---|
| `@v1` | Current stable major version (auto-receives patches) |
| `@v1.2.3` | Pinned exact version |

Breaking changes → new major tag (`@v2`). Old major maintained for 4 weeks with migration guide.

## Access Policy

This repo is public. Any repo in the `kokorolx` org (or any public repo) can call these workflows.

For private org repos, no extra configuration needed.
