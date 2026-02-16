# Conventions

> Full details in [`AGENTS.md`](../AGENTS.md).

## Code Conventions

### TypeScript
- Strict mode (`"strict": true`)
- ESLint + Prettier enforced
- Named exports preferred over default exports
- Async/await over raw Promises
- Explicit return types on public functions
- JSDoc on all public APIs

### Go
- Follow Effective Go and `gofmt`
- Error wrapping with `fmt.Errorf("context: %w", err)`
- Table-driven tests
- `golint` / `staticcheck` clean

### YAML (OpenAPI)
- 2-space indentation
- `$ref` for reusable schemas — avoid inline duplication
- Comments for non-obvious design decisions

### Markdown
- ATX-style headers (`#`, not underline)
- One sentence per line (clean diffs)
- Mermaid diagrams for architecture and flows
- Tables for structured data

## Git Workflow

### Branches
- `main` — stable, always deployable
- `develop` — integration branch
- `feature/<name>`, `fix/<name>`, `docs/<name>`, `chore/<name>`

### Conventional Commits
Format: `<type>(<scope>): <description>`

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`

Scopes: `api`, `server`, `client`, `actions`, `docs`

## Quality Targets

| Metric | Target |
|--------|--------|
| Test coverage | 80%+ |
| API response (p95) | < 100 ms |
| Critical vulnerabilities | 0 |
| API uptime | 99.9% |
| Documentation coverage | 100% |
