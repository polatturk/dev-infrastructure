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
├── .github/
│   └── workflows/     # Reusable GitHub Actions workflows (planned)
├── docs/              # Infrastructure documentation and conventions (planned)
├── templates/         # Project templates (planned)
│   ├── docker/        # Docker / containerization templates
│   ├── dotnet/        # .NET project templates
│   ├── flutter/       # Flutter project templates
│   ├── node/          # Node.js / React / Next.js project templates
│   └── python/        # Python project templates
└── README.md
```

> **Note:** At present, this repository contains only the directory structure and this README. The workflows, templates, and documentation described in this file are planned and will be added incrementally.

## Reusable GitHub Actions Workflows

Rather than copying workflow definitions into every project, this repository will host **reusable workflows** (using GitHub Actions' `workflow_call` mechanism) and **composite actions**. Individual projects will keep a minimal workflow file that delegates to the definitions stored here.

The intended benefits:

- **Consistency** — every project runs the same build, test, and release steps.
- **Central maintenance** — a fix or improvement to a pipeline applies to all consuming projects.
- **Simplified project repos** — project repositories contain only a thin entry-point workflow.

Workflows will be organized by technology (for example, dedicated pipelines for Flutter, .NET, Node.js, and Python) with shared steps for common concerns such as dependency caching, artifact publishing, and release tagging.

## Project Templates

The `templates/` directory will contain ready-to-use starting points for new projects, organized by technology. Each template is intended to include:

- A sensible default project structure following the conventions of its ecosystem.
- Configuration wired to the reusable CI/CD workflows in this repository.
- Baseline quality tooling (linting, formatting, testing setup).
- A `.gitignore` appropriate for the technology, including exclusion of local environment files.

Templates are maintained as reference implementations: they are kept minimal, up to date, and aligned with the workflows and documentation in this repository.

## Security Principles

This repository follows a strict set of security rules. They apply to this repository and to every project that consumes its infrastructure.

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

- Secrets are supplied at runtime through GitHub Actions Secrets, environment variables, or a dedicated secret manager — never stored in source files.
- `.env` files are kept out of Git via `.gitignore`; only `.env.example` files with placeholder values are committed.
- Client-side secrets are never placed in frontend code or distributed binaries.
- CI/CD pipelines use least-privilege credentials.
- Secret scanning is part of the planned quality gates for every project.
- If a secret is ever exposed, it must be revoked and rotated immediately — removing it from a file does not remove it from Git history.

## Planned Infrastructure

The following capabilities are planned for this repository. None of them are implemented yet; they will be built out incrementally.

### CI/CD

- Reusable build and release workflows per technology (Flutter, .NET, Node.js, Python, Docker).
- Automated versioning and changelog generation.
- Artifact publishing and release tagging.

### Automated Testing

- Standardized test execution steps integrated into every pipeline.
- Unit and integration test stages per technology.
- Test result reporting in pull requests.

### Security Scanning

- Dependency vulnerability scanning for all ecosystems.
- Static analysis (SAST) and secret detection on every push and pull request.
- Container image scanning for Docker-based deployments.

### Deployment

- Reusable deployment workflows targeting containerized environments.
- Explicit environment protection rules and manual approval gates for production deployments.
- Clearly separated development, staging, and production configurations.

## Status

| Component | Status |
| --- | --- |
| Repository structure | ✅ Exists |
| This README | ✅ Exists |
| Reusable workflows | 🚧 Planned |
| Project templates | 🚧 Planned |
| Documentation | 🚧 Planned |
| Security scanning pipelines | 🚧 Planned |
| Deployment workflows | 🚧 Planned |
