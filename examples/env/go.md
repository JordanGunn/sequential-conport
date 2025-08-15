---
description: Example .env.md configuration for Go projects using go modules
---

```yaml
# Go + go modules Environment Configuration Example
# Copy this to your project's .windsurf/workflows/conport/.env.md and customize paths

# Project Structure
project:
  root_path: "/path/to/your/go/project"
  src_path: "/path/to/your/go/project"
  tests_path: "/path/to/your/go/project"
  docs_path: "/path/to/your/go/project/docs"
  
# Language/Framework Detection
language: go
framework: go

# Generic Tool Categories (language-agnostic)
tools:
  package_manager: go
  test_runner: go
  formatter: gofmt
  linter: golangci-lint
  type_checker: go  # Built into go compiler
  import_organizer: goimports
  dependency_analyzer: go
  build_tool: go

# Tool-specific configurations
test:
  runner_args: ["test", "-v", "./..."]
  coverage_threshold: 80
  
lint:
  safe_autofix_args: ["run", "--fix"]
  report_args: ["run", "--out-format", "json"]
  
format:
  args: ["-w", "."]
  
type_check:
  args: ["build", "./..."]

# Legacy compatibility (for existing workflows)
executables: 
  go: /usr/local/go/bin/go
  git: /usr/bin/git
  gofmt: /usr/local/go/bin/gofmt
  goimports: /home/user/go/bin/goimports
  golangci-lint: /home/user/go/bin/golangci-lint
```
