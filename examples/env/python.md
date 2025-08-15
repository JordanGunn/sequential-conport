---
description: Example .env.md configuration for Python projects using Poetry
---

```yaml
# Python + Poetry Environment Configuration Example
# Copy this to your project's .windsurf/workflows/conport/.env.md and customize paths

# Project Structure
project:
  root_path: "/path/to/your/python/project"
  src_path: "/path/to/your/python/project/src"
  tests_path: "/path/to/your/python/project/tests"
  docs_path: "/path/to/your/python/project/docs"
  
# Language/Framework Detection
language: python
framework: poetry

# Generic Tool Categories (language-agnostic)
tools:
  package_manager: poetry
  test_runner: pytest
  formatter: black
  linter: ruff
  type_checker: mypy
  import_organizer: isort
  dependency_analyzer: pipdeptree
  build_tool: poetry

# Tool-specific configurations
test:
  runner_args: ["-v", "--tb=short"]
  coverage_threshold: 80
  
lint:
  safe_autofix_args: ["check", "--fix", "--select", "I"]  # Only import fixes
  report_args: ["check", "--format", "json"]
  
format:
  args: ["--line-length", "88"]
  
type_check:
  args: ["--strict"]

# Legacy compatibility (for existing workflows)
executables: 
  poetry: /home/user/.local/bin/poetry
  git: /usr/bin/git
  python: /usr/bin/python3
```
