# aperim/.github

Shared GitHub Actions workflows and composite actions for all Aperim product repositories.

## Contents

| Path | Type | Purpose |
|---|---|---|
| `.github/actions/setup-node-pnpm/` | Composite action | Checkout + pnpm setup + Node.js + `pnpm install` |
| `.github/workflows/ci-quality.yml` | Reusable workflow | Lint + type check |
| `.github/workflows/security-scan.yml` | Reusable workflow | CodeQL SAST + pnpm dependency audit |

All actions are **SHA-pinned** following the pattern established in [aperim/careswap](https://github.com/aperim/careswap).

---

## Usage

### Composite action: `setup-node-pnpm`

Embeds checkout + pnpm + Node.js + `pnpm install --frozen-lockfile` as steps inside any job.

```yaml
steps:
  - name: Setup Node.js + pnpm
    uses: aperim/.github/.github/actions/setup-node-pnpm@main
    with:
      ref: ${{ github.ref }}           # optional, defaults to triggering ref
      node-version-file: '.nvmrc'      # optional, default .nvmrc
      fetch-depth: '1'                 # optional, default 1
```

---

### Reusable workflow: `ci-quality.yml`

Runs **lint** and **typecheck** as parallel jobs.

```yaml
jobs:
  quality:
    uses: aperim/.github/.github/workflows/ci-quality.yml@main
    with:
      # Self-hosted Aperim runner:
      runner: '["self-hosted","ubuntu-latest","aperim"]'
      # Or GitHub-hosted (default):
      # runner: '"ubuntu-latest"'
      ref: ${{ github.ref }}            # optional
      node-version-file: '.nvmrc'       # optional
      lint-command: 'pnpm lint'         # optional
      format-check-command: 'pnpm format:check'  # optional, set '' to skip
      typecheck-command: 'pnpm -r --parallel typecheck'  # optional
```

**Permissions required** (caller job): none beyond the default `contents: read`.

---

### Reusable workflow: `security-scan.yml`

Runs **CodeQL SAST** and **pnpm dependency audit**.

Recommended trigger pattern:

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 20 * * 0'   # Weekly — Sunday 20:00 UTC (Mon 07:00 AEST)

jobs:
  security:
    uses: aperim/.github/.github/workflows/security-scan.yml@main
    with:
      runner: '["self-hosted","ubuntu-latest","aperim"]'
      run-codeql: true      # requires GHAS / Code Security enabled
      run-audit: true
      audit-level: 'high'   # low | moderate | high | critical
    permissions:
      actions: read
      contents: read
      security-events: write
```

#### Audit allowlist

Create `audit-allowlist.json` at the repo root to suppress known advisory exceptions:

```json
{
  "allowlist": ["GHSA-xxxx-yyyy-zzzz"]
}
```

When this file is present, audit findings produce a warning instead of a build failure.

#### CodeQL

Requires [GitHub Code Security](https://docs.github.com/en/code-security/code-scanning/automatically-scanning-your-code-for-vulnerabilities-and-errors/about-code-scanning) to be enabled on the repository. Enable at:
`Settings → Code security → Code scanning → Set up → CodeQL analysis`

If not enabled, the analyze step uses `continue-on-error: true` to avoid blocking CI.

---

## SHA pins

Action SHAs are taken from `aperim/careswap` and must be updated together when bumping action versions. Search for the SHA comment pattern (`# v*`) across this repo to find all pins.

| Action | SHA | Tag |
|---|---|---|
| `actions/checkout` | `de0fac2e4500dabe0009e67214ff5f5447ce83dd` | v6 |
| `pnpm/action-setup` | `41ff72655975bd51cab0327fa583b6e92b6d3061` | v4 |
| `actions/setup-node` | `53b83947a5a98c8d113130e565377fae1a50d02f` | v6 |
| `github/codeql-action/init` | `820e3160e279568db735cee8ed8f8e77a6da7818` | v3 |
| `github/codeql-action/autobuild` | `820e3160e279568db735cee8ed8f8e77a6da7818` | v3 |
| `github/codeql-action/analyze` | `820e3160e279568db735cee8ed8f8e77a6da7818` | v3 |

---

## Related

- Parent issue: [APE-276](https://plane.aperim.com/APE/issues/APE-276) — Create aperim/.github org repo
- CI standardisation epic: [APE-273](https://plane.aperim.com/APE/issues/APE-273) — Platform CI/CD pipeline standardisation