# Prompt 4: GitHub Actions Integration

> **Agent**: DevOps & Integration Engineer
> **Phase**: 4 — GitHub Actions Integration

Create GitHub Actions workflows and custom actions for inGitDB validation:

Components:

1. Custom Action: validate-ingitdb
   - Uses ingitdb-go CLI to validate data files
   - Checks JSON/YAML syntax
   - Validates schemas
   - Checks referential integrity
   - Reports errors with file locations

2. Workflow: PR Validation
   - Trigger: Pull request to main branches
   - Steps:
     a. Checkout code
     b. Setup ingitdb-go CLI
     c. Run validation on changed files
     d. Post validation results as PR comment
     e. Block merge if validation fails

3. Workflow: Continuous Validation
   - Trigger: Push to any branch
   - Steps:
     a. Full validation of all data files
     b. Generate validation report
     c. Upload as artifact
     d. Notify on failures

4. Workflow: Release
   - Trigger: Tag push
   - Steps:
     a. Full validation
     b. Build server and client
     c. Run tests
     d. Publish packages
     e. Create GitHub release

Implementation:
- Use TypeScript for custom actions
- Use @actions/core and @actions/github
- Cache dependencies
- Use matrix strategy for multi-version testing
- Add security scanning
- Generate SBOM (Software Bill of Materials)

Example .github/workflows/validate.yml:
```yaml
name: Validate Data
on:
  pull_request:
    paths:
      - 'data/**'
      - '*.json'
      - '*.yaml'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup ingitdb-go
        uses: ingitdb/setup-ingitdb-go@v1

      - name: Validate Data
        uses: ingitdb/validate-ingitdb@v1
        with:
          path: ./data
          schema-path: ./schemas

      - name: Comment PR
        if: failure()
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: 'Data validation failed. Check the logs for details.'
            })
```
