# inGitDB AI — Development Plan & Resources

Planning documents, architecture specs, and AI agent prompts for the [inGitDB](https://github.com/ingitdb) ecosystem — an open-source versioned database that stores data in text files (JSON, YAML, columnar storage) with Git-based version control.

## Contents

| Document | Description |
|----------|-------------|
| [Development Plan](./docs/dev-plan/) | Skills matrix, 6 AI agents, prompts, 6-phase execution plan, tech stack, risks |
| [OpenAPI Specification](./docs/openapi-spec.yaml) | Complete REST API contract (OpenAPI 3.0, 25+ endpoints) |
| [TypeScript Client Architecture](./docs/ingitdb-ts-architecture.md) | Client design, fluent API, query builder, error handling |
| [GitHub Actions Integration](./docs/github-actions-integration.md) | CI/CD workflows, custom actions, validation automation |
| [AI Prompts](./ai/prompts/) | 10 ready-to-use prompts for AI agents |
| [Docs Index](./docs/) | Full documentation navigation |

## What is inGitDB?

A Git-backed versioned database designed for:
- **Collaboration** — multiple users work on data with Git workflows
- **Version Control** — full history, branching, and merging of data changes
- **Text-Based Storage** — JSON, YAML, and columnar formats
- **Reference Data** — configuration, reference tables, slowly-changing data
- **Validation** — schema validation and referential integrity checks

### Components

```
┌─────────────────────────────────────────────────────┐
│                   GitHub Actions                     │
│              (Validation & CI/CD)                    │
└─────────────────────────────────────────────────────┘
                         ▲
                         │ Uses
                         │
┌────────────────┐   ┌───▼──────────────┐   ┌─────────────────┐
│   ingitdb-ts   │◄──┤  ingitdb-server  │───┤   ingitdb-go    │
│  (TS Client)   │   │   (REST API)     │   │   (CLI/Lib)     │
└────────────────┘   └──────────────────┘   └─────────────────┘
                              │
                              ▼
                     ┌────────────────┐
                     │   Git Storage  │
                     │ (JSON/YAML/CSV)│
                     └────────────────┘
```

## Quick Start

**Project Managers** — review the [Execution Plan](./docs/dev-plan/#execution-plan), [Skills Matrix](./docs/dev-plan/#skills-matrix), and [Risk Management](./docs/dev-plan/#risk-management).

**Developers** — browse the [AI Prompts](./ai/prompts/), follow [Technology Stack](./docs/dev-plan/#technology-stack-recommendations) recommendations, and reference the architecture docs.

**AI Agents** — find your [role and deliverables](./docs/dev-plan/#ai-agents--responsibilities), pick your prompt, and follow acceptance criteria.

### Generate Server from OpenAPI

```bash
npx @openapitools/openapi-generator-cli generate \
  -i docs/openapi-spec.yaml \
  -g nodejs-express-server \
  -o ./ingitdb-server
```

### Generate TypeScript Client

```bash
npx @openapitools/openapi-generator-cli generate \
  -i docs/openapi-spec.yaml \
  -g typescript-axios \
  -o ./ingitdb-ts/src/generated

# See docs/ingitdb-ts-architecture.md for wrapper implementation
```

## Development Phases

| Phase | Duration | Focus | Status |
|-------|----------|-------|--------|
| 1 | Weeks 1-2 | API Design & OpenAPI Spec | Done |
| 2 | Weeks 3-6 | Server Implementation | Planned |
| 3 | Weeks 5-7 | TypeScript Client | Planned |
| 4 | Weeks 7-8 | GitHub Actions Integration | Planned |
| 5 | Weeks 9-10 | Documentation & Polish | Planned |
| 6 | Weeks 11-12 | Testing & Release | Planned |

## Related Resources

- [inGitDB Organization](https://github.com/ingitdb)
- [ingitdb-go CLI](https://github.com/ingitdb/ingitdb-go)
- [inGitDB Documentation](https://ingitdb.github.io/)

## Contributing

This repository contains planning documents.
For contributing to implementations:

1. **CLI & Server**: [`ingitdb/ingitdb-go`](https://github.com/ingitdb/ingitdb-go)
2. **TypeScript Client**: [`ingitdb/ingitdb-ts`](https://github.com/ingitdb/ingitdb-ts)
3. **GitHub Actions**: [`ingitdb/ingitdb-action`](https://github.com/ingitdb/ingitdb-action) (to be created)

## License

MIT License — see [LICENSE](./LICENSE) for details.
