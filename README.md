# dev-infrastructure

Centralized development and DevOps infrastructure repository. This repository serves as the single source of truth for reusable CI/CD pipelines, project templates, and engineering conventions shared across all of my software projects.

## Purpose

Instead of duplicating build, test, and deployment configuration in every project repository, this repository provides:

- **Reusable GitHub Actions workflows** that individual projects reference or call, keeping pipelines consistent and maintained in one place.
- **Project templates** that bootstrap new repositories with a proven, pre-configured starting point.
- **Documentation** of the conventions, standards, and infrastructure decisions that apply across projects.

The goal is to reduce setup time for new projects, keep quality gates uniform, and make infrastructure improvements propagate to all projects automatically.

## Supported Technologies

The infrastructure in this repository is intended to support the following technology stack:

| Area | Technologies |
| --- | --- |
| Mobile | Flutter |
| Backend / APIs | .NET, Node.js, Python |
| Frontend | React, Next.js |
| AI / LLM | AI-assisted and LLM-based applications (Python / Node.js) |
| Containers | Docker |
| Data | Relational and NoSQL databases |

## Repository Structure

The current layout of this repository:

```
dev-infrastructure/
|-- .github/
|   `-- workflows/
|       |-- dotnet-ci.yml     # Reusable .NET CI workflow (restore, build, test)
|       `-- gitleaks.yml      # Reusable secret-scanning workflow (gitleaks)
|-- docs/
|   `-- security-scanning.md  # Automated security scanning documentation
|-- templates/                # Project templates (planned)
|   |-- docker/               # Docker / containerization templates
|   |-- dotnet/               # .NET project templates
|   |-- flutter/              # Flutter project templates
|   |-- node/                 # Node.js / React / Next.js project templates
|   `-- python/               # Python project templates
`-- README.md
```

> **Note:** The `templates/` directories currently exist only as empty placeholders; their contents are planned. The reusable workflows and the security-scanning documentation listed above are implemented and available.

## Reusable GitHub Actions Workflows

Rather than copying workflow definitions into every project, this repository hosts **reusable workflows** (using GitHub Actions' `workflow_call` mechanism). Individual projects keep a minimal workflow file that delegates to the definitions stored here.

The benefits:

- **Consistency** - every project runs the same build, test, and release steps.
- **Central maintenance** - a fix or improvement to a pipeline applies to all consuming projects.
- **Simplified project repos** - project repositories contain only a thin entry-point workflow.

Workflows are organized by technology. Two are implemented today - [.NET CI](.github/workflows/dotnet-ci.yml) and [Gitleaks secret scanning](.github/workflows/gitleaks.yml). Reusable CI workflows for Flutter, Node.js, and Python, along with shared steps for common concerns such as dependency caching, artifact publishing, and release tagging, are planned (see [Planned Infrastructure](#planned-infrastructure)).

### .NET CI

The [.NET CI workflow](.github/workflows/dotnet-ci.yml) restores, builds, and tests a .NET solution or project. It installs the requested .NET SDK version, runs `dotnet restore`, `dotnet build --no-restore`, and (optionally) `dotnet test --no-build`.

**Inputs:**

| Input | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `dotnet-version` | string | Yes | none | .NET SDK version to install (e.g. `10.0.x`) |
| `project-path` | string | Yes | none | Path to the solution or project file to build |
| `run-tests` | boolean | No | `true` | Whether to run tests after the build |

**Calling the workflow from another repository:**

Create a minimal workflow file (for example `.github/workflows/ci.yml`) in the consuming repository:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  dotnet-ci:
    # Reference this repository by owner, path, and branch or tag.
    uses: polatturk/dev-infrastructure/.github/workflows/dotnet-ci.yml@main
    with:
      dotnet-version: "10.0.x"
      project-path: ./src/MySolution.sln
      run-tests: true
```

The workflow checks out and builds the **calling** repository, not this one.

> **Note:** Reusable workflows can only be called from other repositories if this repository's visibility and access allow it (public, or private within the same organization with appropriate settings).

### Secret Scanning (Gitleaks)

The [Gitleaks workflow](.github/workflows/gitleaks.yml) runs [gitleaks-action](https://github.com/gitleaks/gitleaks-action) to detect leaked secrets in the full history of the **calling** repository. The workflow is not triggered by activity in this repository; it runs when a consuming project calls it from its own `push` and `pull_request` workflows (see the example below). It requires no inputs and no custom secrets - it uses the calling repository's own `GITHUB_TOKEN` and requests only `contents: read`.

**Calling the workflow from another repository:**

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

For details on how the scan works, configuration via `.gitleaks.toml`, and how to handle a detected secret, see [Security Scanning](docs/security-scanning.md).

## Project Templates

The `templates/` directory is intended to contain ready-to-use starting points for new projects, organized by technology. The per-technology directories (`flutter/`, `dotnet/`, `node/`, `python/`, `docker/`) currently exist as empty placeholders; their contents are planned. Each template is intended to include:

- A sensible default project structure following the conventions of its ecosystem.
- Configuration wired to the reusable CI/CD workflows in this repository.
- Baseline quality tooling (linting, formatting, testing setup).
- A `.gitignore` appropriate for the technology, including exclusion of local environment files.

Templates will be maintained as reference implementations: kept minimal, up to date, and aligned with the workflows and documentation in this repository.

## Security Principles

This repository follows a strict set of security rules. They apply to this repository and to every project that consumes its infrastructure.

Automated secret detection is already available: consuming projects call the reusable [Gitleaks workflow](.github/workflows/gitleaks.yml) from their own `push` and `pull_request` workflows, so every push and pull request is scanned for leaked credentials. See [Security Scanning](docs/security-scanning.md) for usage and configuration details.

**Never commit any of the following to version control:**

- API keys
- Passwords and database credentials
- Access tokens, refresh tokens, and session secrets
- JWT signing secrets
- OAuth client secrets
- SSH private keys
- Cloud provider credentials
- Private certificates
- Any other confidential configuration

**Required practices:**

- Secrets are supplied at runtime through GitHub Actions Secrets, environment variables, or a dedicated secret manager - never stored in source files.
- `.env` files are kept out of Git via `.gitignore`; only `.env.example` files with placeholder values are committed.
- Client-side secrets are never placed in frontend code or distributed binaries.
- CI/CD pipelines use least-privilege credentials.
- Secret scanning via the reusable Gitleaks workflow is part of the quality gates for every project.
- If a secret is ever exposed, it must be revoked and rotated immediately - removing it from a file does not remove it from Git history.

## Planned Infrastructure

The following capabilities are planned for this repository but not yet implemented; they will be built out incrementally. Implemented capabilities - the [.NET CI workflow](.github/workflows/dotnet-ci.yml), the [Gitleaks secret-scanning workflow](.github/workflows/gitleaks.yml), and the [security-scanning documentation](docs/security-scanning.md) - are described in the sections above.

### CI/CD

- Reusable build and test workflows for Flutter, Node.js, and Python (the .NET CI workflow is already available).
- Automated versioning and changelog generation.
- Artifact publishing and release tagging.

### Automated Testing

- Standardized test execution for the Flutter, Node.js, and Python pipelines (the .NET CI workflow already runs tests).
- Unit and integration test stages per technology.
- Test result reporting in pull requests.

### Security Scanning

- Dependency vulnerability scanning for all ecosystems.
- Static analysis (SAST) beyond secret detection.
- Container image scanning for Docker-based deployments.

Secret detection via Gitleaks is already available; see [Security Scanning](docs/security-scanning.md).

### Deployment

- Reusable deployment workflows targeting containerized environments.
- Explicit environment protection rules and manual approval gates for production deployments.
- Clearly separated development, staging, and production configurations.

## Status

| Component | Status |
| --- | --- |
| Repository structure | [x] Exists |
| This README | [x] Exists |
| Reusable workflows | [x] Available - [.NET CI](.github/workflows/dotnet-ci.yml) and [Gitleaks secret scanning](.github/workflows/gitleaks.yml); Flutter, Node.js, and Python CI planned |
| Project templates | [~] Planned - placeholder directories exist |
| Documentation | [~] In progress - [security scanning](docs/security-scanning.md) documented |
| Security scanning | [~] Partial - secret detection available; dependency, SAST, and container scanning planned |
| Deployment workflows | [~] Planned |
