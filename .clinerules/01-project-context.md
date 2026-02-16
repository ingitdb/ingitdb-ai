# Project Context

> Full guidelines: [`AGENTS.md`](../AGENTS.md) in the repository root.

## What is inGitDB?

inGitDB is an open-source versioned database for collaboration that stores data in text files (JSON, YAML, CSV) with Git-based version control. Best for change-managing reference data with validation through GitHub Actions using the ingitdb-go CLI.

## This Repository

`ingitdb-ai` is a **documentation-only** repository containing planning and architecture artifacts. There is no application source code here. Implementation happens in separate repos.

## Key Documents

| File | Content |
|------|---------|
| `AGENTS.md` | Central AI agent guidelines — **read this for full context** |
| `development-plan.md` | 6 AI agents, prompts, 6-phase execution plan, tech stack, risks |
| `openapi-spec.yaml` | Complete REST API specification (OpenAPI 3.0, 25+ endpoints) |
| `ingitdb-ts-architecture.md` | TypeScript client architecture and design |
| `github-actions-integration.md` | CI/CD workflows and custom GitHub Actions |
| `QUICK-REFERENCE.md` | Role-based navigation guide |
| `SUMMARY.md` | Deliverable status and next steps |

## Architecture

- **ingitdb-server** — REST API server (Node.js/TS or Go)
- **ingitdb-ts** — TypeScript client library (NPM package)
- **ingitdb-go** — Go CLI (existing)
- **GitHub Actions** — Validation workflows using ingitdb-go

## Tech Stack

- Server: Node.js 18+/TypeScript or Go 1.21+, JWT auth
- Client: TypeScript 5+, Rollup/tsup, Jest
- CLI: Go 1.21+, go-git
- CI/CD: GitHub Actions, Docker
