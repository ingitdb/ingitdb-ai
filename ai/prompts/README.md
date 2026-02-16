# Development Prompts

Ready-to-use prompts for AI agents working on inGitDB components.
Each prompt targets a specific agent and development phase defined in the [Development Plan](../../docs/dev-plan/).

## Prompts

| # | Prompt | Agent | Phase |
|---|--------|-------|-------|
| 1 | [OpenAPI Specification Design](./01-openapi-specification-design.md) | API Architect | 1 — Foundation |
| 2 | [Server Implementation](./02-server-implementation.md) | Server Backend Developer | 2 — Server |
| 3 | [TypeScript Client Development](./03-typescript-client-development.md) | TypeScript Client Developer | 3 — Client |
| 4 | [GitHub Actions Integration](./04-github-actions-integration.md) | DevOps & Integration Engineer | 4 — CI/CD |
| 5 | [Testing Strategy](./05-testing-strategy.md) | Testing & QA Engineer | 6 — Release |
| 6 | [DB Migration Scripts Generator](./06-db-migration-scripts-generator.md) | Integration Engineer | 2 — Server (ext.) |
| 7 | [inGitDB Triggers System](./07-triggers-system.md) | Integration Engineer | 2 — Server (ext.) |
| 8 | [Access Management Policy](./08-access-management-policy.md) | Integration Engineer | 2 — Server (ext.) |
| 9 | [DALgo Client Integration](./09-dalgo-client-integration.md) | Integration Engineer | 3 — Client (parallel) |
| 10 | [Datatug Integration](./10-datatug-integration.md) | Integration Engineer | 5 — Docs (parallel) |

## Usage

Copy a prompt into your AI coding agent of choice (Claude, Copilot, Cursor, etc.) to kick off the corresponding development task.
Pair each prompt with the relevant architecture document for full context:

- Prompts 1-2: [`openapi-spec.yaml`](../../docs/openapi-spec.yaml)
- Prompt 3, 9: [`ingitdb-ts-architecture.md`](../../docs/ingitdb-ts-architecture.md)
- Prompt 4: [`github-actions-integration.md`](../../docs/github-actions-integration.md)
- Prompts 5-8, 10: [Development Plan](../../docs/dev-plan/)
