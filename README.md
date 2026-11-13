# setup-diverge

GitHub Action to install and cache the [Diverge](https://github.com/divergedev/diverge) CLI for preview environment management.

## Usage

```yaml
- uses: divergedev/setup-diverge@v1
```

That's it. The action:
1. Resolves the latest version (or uses a pinned version)
2. Caches the binary across workflow runs
3. Verifies the SHA-256 checksum against `checksums.txt`
4. Adds `diverge` to `PATH`

### Pin a version

```yaml
- uses: divergedev/setup-diverge@v1
  with:
    version: '0.24.0'
```

### Full workflow example

```yaml
name: Preview Environment
on:
  pull_request:
    types: [opened, synchronize, reopened, closed]

jobs:
  preview:
    if: github.event.action != 'closed'
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: divergedev/setup-diverge@v1

      - name: Detect changed services
        run: diverge diff --output json --base origin/main

      - name: Deploy preview
        env:
          PR_NUMBER: ${{ github.event.pull_request.number }}
        run: diverge create --mr "${PR_NUMBER}"

  cleanup:
    if: github.event.action == 'closed'
    runs-on: ubuntu-latest
    steps:
      - uses: divergedev/setup-diverge@v1
      - env:
          PR_NUMBER: ${{ github.event.pull_request.number }}
        run: diverge delete "preview-mr-${PR_NUMBER}" || true
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `version` | No | `latest` | CLI version to install (with or without `v` prefix) |
| `token` | No | `${{ github.token }}` | GitHub token for resolving latest version |

## Outputs

| Output | Description |
|--------|-------------|
| `version` | The installed CLI version |
| `cache-hit` | Whether the binary was restored from cache |

## Supported Platforms

| Runner OS | Architectures |
|-----------|---------------|
| `ubuntu-latest` | `X64`, `ARM64` |
| `macos-latest` | `X64`, `ARM64` |

## License

Apache 2.0
