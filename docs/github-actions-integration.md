# GitHub Actions Integration for inGitDB

## Overview

This document outlines the GitHub Actions workflows and custom actions for validating inGitDB data files using the ingitdb-go CLI. These workflows ensure data integrity, schema compliance, and referential consistency before merging changes.

## Components

### 1. Custom Action: validate-ingitdb
### 2. Reusable Workflow: data-validation
### 3. PR Validation Workflow
### 4. Continuous Validation Workflow
### 5. Release Workflow

---

## 1. Custom Action: validate-ingitdb

A reusable GitHub Action that validates inGitDB data files using the ingitdb-go CLI.

### Location
`ingitdb/validate-ingitdb-action`

### action.yml

```yaml
name: 'Validate inGitDB Data'
description: 'Validate inGitDB data files using ingitdb-go CLI'
author: 'inGitDB Team'

branding:
  icon: 'check-circle'
  color: 'green'

inputs:
  data-path:
    description: 'Path to data directory'
    required: false
    default: './data'
  
  schema-path:
    description: 'Path to schema directory'
    required: false
    default: './schemas'
  
  config-file:
    description: 'Path to ingitdb config file'
    required: false
    default: '.ingitdb.yaml'
  
  ingitdb-version:
    description: 'Version of ingitdb-go CLI to use'
    required: false
    default: 'latest'
  
  fail-on-warnings:
    description: 'Fail validation if warnings are found'
    required: false
    default: 'false'
  
  output-format:
    description: 'Output format (text, json, markdown)'
    required: false
    default: 'text'
  
  github-token:
    description: 'GitHub token for posting comments'
    required: false
    default: ${{ github.token }}

outputs:
  validation-status:
    description: 'Validation status (success, warnings, failed)'
    value: ${{ steps.validate.outputs.status }}
  
  errors-count:
    description: 'Number of validation errors'
    value: ${{ steps.validate.outputs.errors }}
  
  warnings-count:
    description: 'Number of validation warnings'
    value: ${{ steps.validate.outputs.warnings }}
  
  report-path:
    description: 'Path to validation report file'
    value: ${{ steps.validate.outputs.report }}

runs:
  using: 'composite'
  steps:
    - name: Setup ingitdb-go CLI
      shell: bash
      run: |
        echo "Installing ingitdb-go CLI version ${{ inputs.ingitdb-version }}"
        
        if [ "${{ inputs.ingitdb-version }}" = "latest" ]; then
          VERSION=$(curl -s https://api.github.com/repos/ingitdb/ingitdb-go/releases/latest | jq -r .tag_name)
        else
          VERSION=${{ inputs.ingitdb-version }}
        fi
        
        OS=$(uname -s | tr '[:upper:]' '[:lower:]')
        ARCH=$(uname -m)
        
        if [ "$ARCH" = "x86_64" ]; then
          ARCH="amd64"
        elif [ "$ARCH" = "aarch64" ]; then
          ARCH="arm64"
        fi
        
        DOWNLOAD_URL="https://github.com/ingitdb/ingitdb-go/releases/download/${VERSION}/ingitdb-${OS}-${ARCH}"
        
        echo "Downloading from: $DOWNLOAD_URL"
        curl -L -o /usr/local/bin/ingitdb "$DOWNLOAD_URL"
        chmod +x /usr/local/bin/ingitdb
        
        echo "Installed ingitdb version:"
        ingitdb version

    - name: Validate Data
      id: validate
      shell: bash
      run: |
        echo "Starting validation..."
        
        # Run validation
        ingitdb validate \
          --data-path "${{ inputs.data-path }}" \
          --schema-path "${{ inputs.schema-path }}" \
          --config "${{ inputs.config-file }}" \
          --format "${{ inputs.output-format }}" \
          --output validation-report.txt
        
        VALIDATION_EXIT_CODE=$?
        
        # Parse results
        ERRORS=$(grep -c "ERROR" validation-report.txt || echo "0")
        WARNINGS=$(grep -c "WARNING" validation-report.txt || echo "0")
        
        # Set outputs
        echo "errors=$ERRORS" >> $GITHUB_OUTPUT
        echo "warnings=$WARNINGS" >> $GITHUB_OUTPUT
        echo "report=validation-report.txt" >> $GITHUB_OUTPUT
        
        # Determine status
        if [ $VALIDATION_EXIT_CODE -ne 0 ] || [ $ERRORS -gt 0 ]; then
          echo "status=failed" >> $GITHUB_OUTPUT
          echo "::error::Validation failed with $ERRORS error(s)"
        elif [ $WARNINGS -gt 0 ]; then
          echo "status=warnings" >> $GITHUB_OUTPUT
          echo "::warning::Validation passed with $WARNINGS warning(s)"
        else
          echo "status=success" >> $GITHUB_OUTPUT
          echo "::notice::Validation passed successfully"
        fi
        
        # Display report
        cat validation-report.txt
        
        # Fail if needed
        if [ "${{ inputs.fail-on-warnings }}" = "true" ] && [ $WARNINGS -gt 0 ]; then
          echo "::error::Failing due to warnings (fail-on-warnings=true)"
          exit 1
        fi
        
        if [ $VALIDATION_EXIT_CODE -ne 0 ] || [ $ERRORS -gt 0 ]; then
          exit 1
        fi

    - name: Upload Validation Report
      if: always()
      uses: actions/upload-artifact@v4
      with:
        name: validation-report
        path: validation-report.txt
        retention-days: 30
```

---

## 2. Setup Action: setup-ingitdb-go

A setup action to install the ingitdb-go CLI (can be used separately).

### action.yml

```yaml
name: 'Setup ingitdb-go'
description: 'Install ingitdb-go CLI'
author: 'inGitDB Team'

branding:
  icon: 'download'
  color: 'blue'

inputs:
  version:
    description: 'Version of ingitdb-go to install'
    required: false
    default: 'latest'
  
  github-token:
    description: 'GitHub token for API requests'
    required: false
    default: ${{ github.token }}

outputs:
  version:
    description: 'Installed version'
    value: ${{ steps.install.outputs.version }}

runs:
  using: 'composite'
  steps:
    - name: Install ingitdb-go
      id: install
      shell: bash
      env:
        GITHUB_TOKEN: ${{ inputs.github-token }}
      run: |
        if [ "${{ inputs.version }}" = "latest" ]; then
          VERSION=$(curl -s -H "Authorization: token $GITHUB_TOKEN" \
            https://api.github.com/repos/ingitdb/ingitdb-go/releases/latest | \
            jq -r .tag_name)
        else
          VERSION=${{ inputs.version }}
        fi
        
        echo "Installing ingitdb-go version: $VERSION"
        
        OS=$(uname -s | tr '[:upper:]' '[:lower:]')
        ARCH=$(uname -m)
        
        case "$ARCH" in
          x86_64) ARCH="amd64" ;;
          aarch64|arm64) ARCH="arm64" ;;
          armv7l) ARCH="arm" ;;
        esac
        
        BINARY_NAME="ingitdb-${OS}-${ARCH}"
        DOWNLOAD_URL="https://github.com/ingitdb/ingitdb-go/releases/download/${VERSION}/${BINARY_NAME}"
        
        echo "Downloading from: $DOWNLOAD_URL"
        curl -L -o /usr/local/bin/ingitdb "$DOWNLOAD_URL"
        chmod +x /usr/local/bin/ingitdb
        
        # Verify installation
        INSTALLED_VERSION=$(ingitdb version --short)
        echo "Successfully installed ingitdb version: $INSTALLED_VERSION"
        echo "version=$INSTALLED_VERSION" >> $GITHUB_OUTPUT
```

---

## 3. PR Validation Workflow

Validates data changes in pull requests.

### .github/workflows/validate-pr.yml

```yaml
name: Validate PR Data Changes

on:
  pull_request:
    types: [opened, synchronize, reopened]
    paths:
      - 'data/**'
      - 'schemas/**'
      - '.ingitdb.yaml'
      - '.github/workflows/validate-pr.yml'

permissions:
  contents: read
  pull-requests: write
  checks: write

jobs:
  validate:
    name: Validate Data
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for better diff analysis
      
      - name: Get changed files
        id: changed-files
        uses: tj-actions/changed-files@v41
        with:
          files: |
            data/**
            schemas/**
      
      - name: Validate inGitDB data
        id: validate
        uses: ingitdb/validate-ingitdb-action@v1
        with:
          data-path: ./data
          schema-path: ./schemas
          config-file: .ingitdb.yaml
          fail-on-warnings: false
          output-format: markdown
          github-token: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Create validation summary
        if: always()
        run: |
          echo "## Validation Results" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          
          if [ "${{ steps.validate.outputs.validation-status }}" = "success" ]; then
            echo "✅ **Status:** Passed" >> $GITHUB_STEP_SUMMARY
          elif [ "${{ steps.validate.outputs.validation-status }}" = "warnings" ]; then
            echo "⚠️ **Status:** Passed with warnings" >> $GITHUB_STEP_SUMMARY
          else
            echo "❌ **Status:** Failed" >> $GITHUB_STEP_SUMMARY
          fi
          
          echo "- Errors: ${{ steps.validate.outputs.errors-count }}" >> $GITHUB_STEP_SUMMARY
          echo "- Warnings: ${{ steps.validate.outputs.warnings-count }}" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "### Changed Files" >> $GITHUB_STEP_SUMMARY
          echo "${{ steps.changed-files.outputs.all_changed_files }}" >> $GITHUB_STEP_SUMMARY
      
      - name: Comment PR
        if: always() && github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            const fs = require('fs');
            const status = '${{ steps.validate.outputs.validation-status }}';
            const errors = '${{ steps.validate.outputs.errors-count }}';
            const warnings = '${{ steps.validate.outputs.warnings-count }}';
            
            let emoji = '✅';
            let statusText = 'Passed';
            
            if (status === 'failed') {
              emoji = '❌';
              statusText = 'Failed';
            } else if (status === 'warnings') {
              emoji = '⚠️';
              statusText = 'Passed with warnings';
            }
            
            const report = fs.readFileSync('${{ steps.validate.outputs.report-path }}', 'utf8');
            
            const body = `## ${emoji} inGitDB Validation ${statusText}
            
            - **Errors:** ${errors}
            - **Warnings:** ${warnings}
            
            <details>
            <summary>View detailed report</summary>
            
            \`\`\`
            ${report}
            \`\`\`
            
            </details>
            
            ${status === 'failed' ? '❌ **This PR cannot be merged until validation passes.**' : ''}
            `;
            
            // Find existing comment
            const { data: comments } = await github.rest.issues.listComments({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
            });
            
            const botComment = comments.find(comment => 
              comment.user.type === 'Bot' && 
              comment.body.includes('inGitDB Validation')
            );
            
            if (botComment) {
              // Update existing comment
              await github.rest.issues.updateComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                comment_id: botComment.id,
                body: body,
              });
            } else {
              // Create new comment
              await github.rest.issues.createComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: context.issue.number,
                body: body,
              });
            }
      
      - name: Set commit status
        if: always()
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            const status = '${{ steps.validate.outputs.validation-status }}';
            
            let state = 'success';
            let description = 'Validation passed';
            
            if (status === 'failed') {
              state = 'failure';
              description = 'Validation failed';
            } else if (status === 'warnings') {
              state = 'success';
              description = 'Validation passed with warnings';
            }
            
            await github.rest.repos.createCommitStatus({
              owner: context.repo.owner,
              repo: context.repo.repo,
              sha: context.sha,
              state: state,
              description: description,
              context: 'inGitDB Validation',
            });
```

---

## 4. Continuous Validation Workflow

Runs full validation on main branch commits.

### .github/workflows/validate-main.yml

```yaml
name: Continuous Data Validation

on:
  push:
    branches:
      - main
      - develop
    paths:
      - 'data/**'
      - 'schemas/**'
      - '.ingitdb.yaml'
  
  schedule:
    # Run daily at 2 AM UTC
    - cron: '0 2 * * *'
  
  workflow_dispatch:
    inputs:
      full-scan:
        description: 'Run full validation scan'
        required: false
        type: boolean
        default: false

jobs:
  validate:
    name: Full Validation
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup ingitdb-go
        uses: ingitdb/setup-ingitdb-go@v1
        with:
          version: latest
      
      - name: Run full validation
        id: validate
        run: |
          ingitdb validate \
            --data-path ./data \
            --schema-path ./schemas \
            --config .ingitdb.yaml \
            --format json \
            --output validation-results.json \
            --verbose
          
          # Parse results
          ERRORS=$(jq '.errors | length' validation-results.json)
          WARNINGS=$(jq '.warnings | length' validation-results.json)
          
          echo "errors=$ERRORS" >> $GITHUB_OUTPUT
          echo "warnings=$WARNINGS" >> $GITHUB_OUTPUT
          
          # Generate summary
          ingitdb validate \
            --data-path ./data \
            --schema-path ./schemas \
            --config .ingitdb.yaml \
            --format markdown \
            --output validation-summary.md
      
      - name: Upload results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: validation-results-${{ github.sha }}
          path: |
            validation-results.json
            validation-summary.md
          retention-days: 90
      
      - name: Generate report
        if: always()
        run: |
          echo "# Validation Report" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          cat validation-summary.md >> $GITHUB_STEP_SUMMARY
      
      - name: Create issue on failure
        if: failure() && steps.validate.outputs.errors > 0
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            const fs = require('fs');
            const summary = fs.readFileSync('validation-summary.md', 'utf8');
            
            const title = '🚨 Data Validation Failure on Main Branch';
            const body = `## Validation Failed
            
            Commit: ${{ github.sha }}
            Branch: ${{ github.ref_name }}
            Workflow: ${{ github.workflow }}
            
            **Errors:** ${{ steps.validate.outputs.errors }}
            **Warnings:** ${{ steps.validate.outputs.warnings }}
            
            ### Details
            
            ${summary}
            
            [View workflow run](${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }})
            `;
            
            await github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: title,
              body: body,
              labels: ['bug', 'data-validation'],
            });
      
      - name: Notify Slack on failure
        if: failure()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "🚨 inGitDB validation failed on ${{ github.ref_name }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*inGitDB Validation Failed*\n\nBranch: `${{ github.ref_name }}`\nErrors: ${{ steps.validate.outputs.errors }}\nWarnings: ${{ steps.validate.outputs.warnings }}"
                  }
                },
                {
                  "type": "actions",
                  "elements": [
                    {
                      "type": "button",
                      "text": {
                        "type": "plain_text",
                        "text": "View Workflow"
                      },
                      "url": "${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
                    }
                  ]
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_WEBHOOK_TYPE: INCOMING_WEBHOOK
```

---

## 5. Release Workflow

Validates and releases inGitDB packages.

### .github/workflows/release.yml

```yaml
name: Release

on:
  push:
    tags:
      - 'v*.*.*'

permissions:
  contents: write
  packages: write

jobs:
  validate:
    name: Pre-release Validation
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Validate all data
        uses: ingitdb/validate-ingitdb-action@v1
        with:
          data-path: ./data
          schema-path: ./schemas
          fail-on-warnings: true
      
      - name: Run security scan
        run: |
          # Scan for sensitive data in files
          echo "Running security scan..."
          ingitdb scan-security \
            --data-path ./data \
            --check-secrets \
            --check-pii

  build-server:
    name: Build Server
    needs: validate
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
      
      - name: Install dependencies
        working-directory: ./server
        run: npm ci
      
      - name: Run tests
        working-directory: ./server
        run: npm test
      
      - name: Build
        working-directory: ./server
        run: npm run build
      
      - name: Build Docker image
        run: |
          docker build -t ingitdb/server:${{ github.ref_name }} ./server
          docker tag ingitdb/server:${{ github.ref_name }} ingitdb/server:latest
      
      - name: Push to Docker Hub
        run: |
          echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
          docker push ingitdb/server:${{ github.ref_name }}
          docker push ingitdb/server:latest

  build-client:
    name: Build TypeScript Client
    needs: validate
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          registry-url: 'https://registry.npmjs.org'
      
      - name: Install dependencies
        working-directory: ./ingitdb-ts
        run: npm ci
      
      - name: Run tests
        working-directory: ./ingitdb-ts
        run: npm test
      
      - name: Build
        working-directory: ./ingitdb-ts
        run: npm run build
      
      - name: Publish to NPM
        working-directory: ./ingitdb-ts
        run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}

  create-release:
    name: Create GitHub Release
    needs: [build-server, build-client]
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Generate changelog
        id: changelog
        run: |
          # Extract version from tag
          VERSION=${GITHUB_REF#refs/tags/}
          
          # Generate changelog from commits
          git log $(git describe --tags --abbrev=0 HEAD^)..HEAD --pretty=format:"- %s (%h)" > CHANGELOG.txt
          
          echo "version=$VERSION" >> $GITHUB_OUTPUT
      
      - name: Create Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ steps.changelog.outputs.version }}
          body_path: CHANGELOG.txt
          draft: false
          prerelease: false
```

---

## 6. Configuration File

Example `.ingitdb.yaml` configuration file.

```yaml
# inGitDB Configuration

version: 1

# Database settings
database:
  name: my-database
  storage_format: json  # json, yaml, columnar
  default_branch: main

# Validation rules
validation:
  # Schema validation
  schemas:
    enabled: true
    strict: true  # Fail on unknown properties
    
  # Reference integrity
  references:
    enabled: true
    check_foreign_keys: true
    
  # Custom validators
  custom:
    - name: email-format
      type: regex
      pattern: '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
      
    - name: phone-format
      type: regex
      pattern: '^\+?[1-9]\d{1,14}$'

# File organization
structure:
  tables_dir: data
  schemas_dir: schemas
  
  # File naming conventions
  naming:
    tables: kebab-case  # kebab-case, snake_case, camelCase
    files: snake_case

# Git integration
git:
  auto_commit: true
  commit_message_template: "chore: update {table} data"
  
  # Branching strategy
  branches:
    feature_prefix: feature/
    release_prefix: release/
    
  # Merge strategy
  merge:
    strategy: merge  # merge, squash, rebase
    conflict_resolution: manual  # manual, auto

# Performance
performance:
  cache:
    enabled: true
    ttl: 300  # seconds
    
  batch_size: 100

# Security
security:
  # Scan for sensitive data
  scan_secrets: true
  scan_pii: true
  
  # Allowed file extensions
  allowed_extensions:
    - .json
    - .yaml
    - .yml
    
  # Maximum file size (MB)
  max_file_size: 10

# Logging
logging:
  level: info  # debug, info, warn, error
  format: json  # json, text
```

---

## 7. Badge Configuration

Add validation badge to README.md:

```markdown
[![Data Validation](https://github.com/ingitdb/my-repo/actions/workflows/validate-main.yml/badge.svg)](https://github.com/ingitdb/my-repo/actions/workflows/validate-main.yml)
```

---

## Usage Examples

### Basic PR Validation

```yaml
# .github/workflows/pr-validation.yml
name: PR Validation

on:
  pull_request:

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: ingitdb/validate-ingitdb-action@v1
        with:
          data-path: ./data
          schema-path: ./schemas
```

### Custom Validation Rules

```yaml
# .github/workflows/custom-validation.yml
name: Custom Validation

on:
  pull_request:

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: ingitdb/setup-ingitdb-go@v1
      
      - name: Custom validation
        run: |
          # Check for required metadata
          ingitdb validate --data-path ./data \
            --custom-rule "all-tables-have-description" \
            --custom-rule "all-records-have-id"
          
          # Check data quality
          ingitdb quality-check --data-path ./data \
            --check-duplicates \
            --check-nulls \
            --check-outliers
```

### Multi-Environment Validation

```yaml
# .github/workflows/multi-env-validation.yml
name: Multi-Environment Validation

on:
  push:
    branches: [main, staging, production]

jobs:
  validate:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [development, staging, production]
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: ingitdb/validate-ingitdb-action@v1
        with:
          data-path: ./data/${{ matrix.environment }}
          schema-path: ./schemas
          config-file: .ingitdb.${{ matrix.environment }}.yaml
```

---

## Best Practices

### 1. Version Pinning
Always pin action versions for reproducibility:
```yaml
uses: ingitdb/validate-ingitdb-action@v1.2.3
```

### 2. Caching
Cache CLI downloads to speed up workflows:
```yaml
- name: Cache ingitdb-go
  uses: actions/cache@v3
  with:
    path: ~/.ingitdb
    key: ingitdb-${{ runner.os }}-${{ hashFiles('.ingitdb.yaml') }}
```

### 3. Fail Fast
Use matrix jobs with fail-fast for quick feedback:
```yaml
strategy:
  fail-fast: true
  matrix:
    validation: [schema, references, security]
```

### 4. Conditional Validation
Only validate changed files in PRs:
```yaml
- name: Get changed files
  id: changed
  run: |
    git diff --name-only origin/main...HEAD | \
      grep '^data/' > changed-files.txt || true

- name: Validate changed files
  if: steps.changed.outputs.has-changes == 'true'
  run: |
    cat changed-files.txt | xargs ingitdb validate
```

---

## Troubleshooting

### Common Issues

1. **CLI not found**: Ensure `setup-ingitdb-go` runs before validation
2. **Permission denied**: Check file permissions and workflow permissions
3. **Rate limiting**: Use `GITHUB_TOKEN` for API requests
4. **Large files**: Increase timeout or split validation into smaller chunks

### Debug Mode

Enable debug logging:
```yaml
- name: Debug validation
  env:
    INGITDB_DEBUG: true
    RUNNER_DEBUG: 1
  run: ingitdb validate --verbose --debug
```

---

## Future Enhancements

1. **Progressive Validation**: Validate only changed tables
2. **Parallel Validation**: Split validation across multiple jobs
3. **Custom Rules Engine**: User-defined validation rules
4. **Auto-fix**: Automatically fix common issues
5. **Performance Metrics**: Track validation performance over time
6. **IDE Integration**: VS Code extension for live validation

---

## Conclusion

The GitHub Actions integration provides comprehensive validation for inGitDB data files, ensuring data quality and integrity throughout the development lifecycle. By automating validation in CI/CD pipelines, teams can catch issues early and maintain high data quality standards.
