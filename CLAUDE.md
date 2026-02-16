# CLAUDE.md — Claude Code Configuration for inGitDB

> Read [`AGENTS.md`](./AGENTS.md) for the full AI agent guidelines.

## Project Overview

inGitDB is an open-source versioned database storing data in text files (JSON, YAML, CSV) with Git-based version control.
This repository (`ingitdb-ai`) contains **planning documents only** — no application source code.

## Key Documents

- `AGENTS.md` — Central AI agent guidelines (start here for full context)
- `docs/dev-plan/` — Agent roles, prompts, 6-phase execution plan, tech stack
- `docs/openapi-spec.yaml` — Complete REST API specification (25+ endpoints, OpenAPI 3.0)
- `docs/ingitdb-ts-architecture.md` — TypeScript client design and fluent API
- `docs/github-actions-integration.md` — CI/CD workflows and custom GitHub Actions
- `ai/prompts/` — 10 ready-to-use development task prompts

## Repository Boundaries

- This is a documentation-only repo: no `.ts`, `.go`, or `.js` source files
- Do NOT create application source code here
- Do NOT modify `docs/openapi-spec.yaml` unless explicitly instructed
- Implementation belongs in separate repos: `ingitdb-server`, `ingitdb-ts`, `ingitdb-go`

## Editing Rules

- Preserve existing document structure; add sections rather than reorganizing
- Use relative links between documents
- Use Mermaid diagrams for architecture and flow visualization
- Reference existing docs instead of duplicating content
- Update `docs/summary.md` when adding new deliverables
- Follow Markdown conventions: ATX headers, one sentence per line, tables for structured data

## Code Conventions (for implementation repos)

- **TypeScript**: strict mode, ESLint + Prettier, named exports, async/await
- **Go**: Effective Go, gofmt, table-driven tests, error wrapping
- **YAML**: 2-space indent, `$ref` for reusable schemas
- **Commits**: Conventional Commits — `<type>(<scope>): <description>`

## Quality Standards

- Test coverage: 80%+
- API response time: < 100ms (p95)
- Zero critical security vulnerabilities
- Documentation coverage: 100%

## Git Workflow

- Branch naming: `<type>/<short-description>` (e.g., `docs/agent-guidelines`)
- PR titles follow Conventional Commits format
- Always include a test plan or evidence of testing in PRs
