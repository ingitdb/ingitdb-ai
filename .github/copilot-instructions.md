# GitHub Copilot Instructions for inGitDB

> **Important**: Read [`AGENTS.md`](../AGENTS.md) for the complete AI agent guidelines.

## Project Context

inGitDB is an open-source versioned database for collaboration that stores data in text files (JSON, YAML, CSV) with Git-based version control.
This repository (`ingitdb-ai`) contains planning and architecture documents only — no application source code.

## Key Files

| Path | Content |
|------|---------|
| `AGENTS.md` | Central AI agent guidelines — **read this first** |
| `docs/dev-plan/` | 6 AI agents, prompts, execution plan, tech stack |
| `docs/openapi-spec.yaml` | REST API specification (OpenAPI 3.0, 25+ endpoints) |
| `docs/ingitdb-ts-architecture.md` | TypeScript client architecture and design |
| `docs/github-actions-integration.md` | CI/CD workflows and custom GitHub Actions |
| `ai/prompts/` | 10 ready-to-use development task prompts |

## Technology Context

- **Server**: Node.js/TypeScript or Go, Express/Fastify or Gin/Echo, JWT auth
- **Client**: TypeScript 5+, Rollup/tsup, Jest
- **CLI**: Go 1.21+, go-git
- **CI/CD**: GitHub Actions, Docker

## Constraints

- This is a documentation repo — do not generate application source code
- Do not modify `docs/openapi-spec.yaml` unless explicitly asked
- Use Markdown with Mermaid diagrams for visualizations
- Follow Conventional Commits: `<type>(<scope>): <description>`
- Reference existing docs instead of duplicating content

## Conventions

- Markdown: ATX headers, one sentence per line, relative links
- TypeScript: strict mode, ESLint + Prettier, named exports
- Go: Effective Go, gofmt, table-driven tests
- YAML: 2-space indentation, `$ref` for reusable schemas
- Test coverage target: 80%+
