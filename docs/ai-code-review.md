# AI Code Review Workflow

Centralized, parameterized reusable workflow for AI-powered code review across all repositories in the `johnsosoka` GitHub namespace.

## Architecture

```
johnsosoka/.github
├── .github/workflows/ai-code-review.yml  ← reusable workflow (this file)
└── docs/ai-code-review.md              ← this documentation

johnsosoka/<repo>
└── .github/workflows/ai-code-review.yml  ← thin caller (~10 lines)
```

## Usage (Caller Workflow)

Add this file to any repo under `.github/workflows/ai-code-review.yml`:

```yaml
name: AI Code Review

on:
  pull_request:
    types: [opened, synchronize]

permissions:
  contents: read
  pull-requests: write

jobs:
  review:
    uses: johnsosoka/.github/.github/workflows/ai-code-review.yml@main
    secrets:
      OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
```

### Override defaults per repo

```yaml
jobs:
  review:
    uses: johnsosoka/.github/.github/workflows/ai-code-review.yml@main
    with:
      openai_model: 'gpt-4o-mini'   # override model
      timeout-minutes: 15            # override timeout
    secrets:
      OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
```

## Parameters

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `ai_provider` | string | `openai` | AI provider (`openai`) |
| `openai_model` | string | `gpt-4o` | OpenAI model for review |
| `timeout-minutes` | number | `10` | Job timeout |
| `include_extensions` | string | `.py,.js,.ts,...` | Comma-separated file extensions to review |
| `exclude_paths` | string | `node_modules/,dist/,...` | Comma-separated paths to exclude |

## Security

- **No secrets in workflow definitions.** The `OPENAI_API_KEY` is passed via `secrets:` mapping and redacted in GitHub Actions logs.
- **Minimal permissions.** Callers use only `contents: read` + `pull-requests: write`. No `contents: write`, no `issues: write`.
- **Public callers require public `.github` repo.** This repository is public so that public repositories in the `johnsosoka` namespace can call the reusable workflow.

## Known Issues

- `AleksandrFurmenkovOfficial/ai-code-review@v1.0` runs on Node.js 20, which GitHub has deprecated. The action still works but will need an update (or a fork) before September 2026.
- To opt into Node.js 24 early, set `FORCE_JAVASCRIPT_ACTIONS_TO_NODE24=true` in the workflow job environment.

## Repository Visibility

This repository is **public** because GitHub requires public caller repositories to reference public reusable workflow repositories. The repository contains only workflow definitions and documentation — no secrets.
