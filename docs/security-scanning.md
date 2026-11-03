# Security Scanning

This document describes the automated security scanning provided by this repository, starting with secret detection via [gitleaks](https://github.com/gitleaks/gitleaks). Additional scanning capabilities (dependency vulnerability scanning, SAST, container image scanning) are planned and will be documented here as they are added.

## Secret Detection (gitleaks)

The reusable workflow at [`.github/workflows/gitleaks.yml`](../.github/workflows/gitleaks.yml) runs [gitleaks-action](https://github.com/gitleaks/gitleaks-action), the official GitHub Action wrapper around the gitleaks secret scanner.

### How the reusable workflow is designed

This workflow is a **reusable workflow** — it is triggered only via `workflow_call` and has no `push` or `pull_request` triggers of its own. It lives centrally in this repository so that every project runs the same secret-scanning quality gate, and improvements made here propagate to all consuming projects automatically.

**Important:** when a project calls this workflow, gitleaks checks out and scans **the calling repository**, not this dev-infrastructure repository. The workflow does not override the checkout repository, so `actions/checkout` operates on whatever repository invoked the call. Likewise, the `GITHUB_TOKEN` used is the calling repository's own token, available automatically when a reusable workflow is invoked — no secrets need to be passed by the caller.

### Calling this workflow from another repository

Add a minimal workflow file (e.g. `.github/workflows/security-scan.yml`) to the consuming repository:

```yaml
name: Security Scan

on:
  push:
  pull_request:

permissions:
  contents: read

jobs:
  gitleaks:
    uses: polatturk/dev-infrastructure/.github/workflows/gitleaks.yml@main
```

Notes on the caller configuration:

- The `uses:` line points at `owner/repo/.github/workflows/<file>@ref`. The ref (`main` here) pins the version of the reusable workflow that runs. Choose a branch, tag, or commit SHA — a tag or SHA is recommended for stability.
- Triggers (`on:`, shown above as every push and pull request) are defined by the **caller**, not by the reusable workflow. Each project decides when scanning runs.
- The caller grants `permissions: contents: read`, which applies to the jobs it calls. The reusable workflow itself requests only `contents: read`, so no additional permissions are ever needed.
- With the trigger above, leaked credentials are caught **before** merge into a protected branch, not after — provided branch protection requires this status check to pass.

### How it works

- The job checks out the **calling** repository with `fetch-depth: 0` (full history). Gitleaks inspects commit contents, so a shallow checkout would cause findings in older commits to be missed.
- The action authenticates with the calling repository's built-in `GITHUB_TOKEN`. No custom secrets or additional permissions are required.
- The workflow is granted only `contents: read`, following the least-privilege principle in this repository's [security rules](../README.md#security-principles).
- If gitleaks detects a secret, the job fails and the caller's pull request is blocked (when branch protection requires status checks).
- Pull-request comments from gitleaks are disabled (`GITLEAKS_ENABLE_COMMENTS: "false"`): findings are reported only in the workflow run log and as the failed check, keeping pull-request conversations clean.

### Configuration

By default, gitleaks runs with its built-in rule set (API keys, tokens, passwords, private keys, etc. for common providers). To customize behavior — add allowlists for false positives or custom rules — commit a `.gitleaks.toml` file to the **calling repository's** root; gitleaks-action picks it up automatically from the checked-out repository.

## Handling a Detected Secret

If gitleaks reports a finding:

1. **Treat it as a live exposure.** Never assume a committed secret is unused.
2. **Revoke and rotate the secret immediately** at its provider. This is the only reliable remediation.
3. **Do not just delete the secret from the file.** It remains in Git history and in every existing clone.
4. Remove the secret from history only with the repository owner's approval, using a history-rewriting tool (e.g. `git filter-repo`), and coordinate with everyone who has a clone. Rewriting shared history requires explicit approval per the repository's Git safety rules.
5. If the finding is a **false positive**, add an allowlist entry in `.gitleaks.toml` with a comment explaining why the value is not a real secret.

## Relationship to Manual Practices

Automated scanning is a safety net, not a substitute for the manual practices in the README:

- Keep secrets in environment variables or a secret manager; never in source files.
- Keep `.env` files out of Git; commit only `.env.example` with placeholder values.
- Use GitHub Actions Secrets for CI/CD credentials.

Scanning catches mistakes; the practices above prevent them.

## Planned Capabilities

The following are planned for this repository but **not yet implemented**:

- Dependency vulnerability scanning (all supported ecosystems).
- Static analysis (SAST) beyond secret detection.
- Container image scanning for Docker-based deployments.
