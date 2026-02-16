# Documentation Index

Navigation hub for all inGitDB planning and architecture documents.

## Core Documents

| Document | Description |
|----------|-------------|
| [Development Plan](./dev-plan/) | Agent roles, prompts, 6-phase execution plan, tech stack, risks |
| [OpenAPI Specification](./openapi-spec.yaml) | Complete REST API contract (OpenAPI 3.0, 25+ endpoints) |
| [TypeScript Client Architecture](./ingitdb-ts-architecture.md) | Client design, fluent API, query builder, error handling |
| [GitHub Actions Integration](./github-actions-integration.md) | CI/CD workflows, custom actions, validation automation |
| [Summary](./summary.md) | Deliverable status, statistics, next steps |

## AI Prompts

10 ready-to-use prompts for AI agents live in [`ai/prompts/`](../ai/prompts/).

| Prompts | Architecture Doc |
|---------|-----------------|
| 1-2 (API design, server) | [openapi-spec.yaml](./openapi-spec.yaml) |
| 3, 9 (TS client, DALgo) | [ingitdb-ts-architecture.md](./ingitdb-ts-architecture.md) |
| 4 (GitHub Actions) | [github-actions-integration.md](./github-actions-integration.md) |
| 5-8, 10 (testing, extensions) | [Development Plan](./dev-plan/) |

## Quick Links by Role

### Project Manager
1. [Execution Plan](./dev-plan/#execution-plan) — timeline and milestones
2. [Skills Matrix](./dev-plan/#skills-matrix) — team requirements
3. [Risk Management](./dev-plan/#risk-management) — planning
4. [Summary](./summary.md) — progress tracking

### Developer
1. [OpenAPI Spec](./openapi-spec.yaml) — API contract
2. [TS Client Architecture](./ingitdb-ts-architecture.md) — client implementation
3. [GitHub Actions](./github-actions-integration.md) — CI/CD setup
4. [AI Prompts](../ai/prompts/) — development guidance

### AI Agent
1. Find your role in the [Development Plan](./dev-plan/#ai-agents--responsibilities)
2. Pick your [prompt](../ai/prompts/)
3. Reference the relevant architecture doc above
