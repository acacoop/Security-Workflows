# Security-Workflows

Central repository for reusable security workflows, CI/CD automations, and DevSecOps integrations across the organization.

## Available reusable workflows

### `/.github/workflows/security-baseline.yml`
Runs a baseline security pipeline that includes:
- Dependency review (pull request diff)
- CodeQL analysis (configurable language)

## Quick start

In any repository, call the reusable workflow:

```yaml
name: Security Baseline

on:
  pull_request:

jobs:
  security:
    uses: acacoop/Security-Workflows/.github/workflows/security-baseline.yml@main
    permissions:
      contents: read
      pull-requests: write
      security-events: write
    with:
      base-ref: ${{ github.event.pull_request.base.sha }}
      head-ref: ${{ github.event.pull_request.head.sha }}
      codeql-language: javascript
```

A complete consumer example is available at `/examples/.github/workflows/use-security-baseline.yml`.
