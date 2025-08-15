---
description: Example .env.md configuration for JavaScript/Node.js projects using npm
---

```yaml
# JavaScript/Node.js + npm Environment Configuration Example
# Copy this to your project's .windsurf/workflows/conport/.env.md and customize paths

# Project Structure
project:
  root_path: "/path/to/your/javascript/project"
  src_path: "/path/to/your/javascript/project/src"
  tests_path: "/path/to/your/javascript/project/tests"
  docs_path: "/path/to/your/javascript/project/docs"
  
# Language/Framework Detection
language: javascript
framework: npm

# Generic Tool Categories (language-agnostic)
tools:
  package_manager: npm
  test_runner: jest
  formatter: prettier
  linter: eslint
  type_checker: tsc
  import_organizer: null  # ESLint handles imports
  dependency_analyzer: npm-check
  build_tool: npm

# Tool-specific configurations
test:
  runner_args: ["--verbose", "--coverage"]
  coverage_threshold: 80
  
lint:
  safe_autofix_args: ["--fix", "--ext", ".js,.ts,.jsx,.tsx"]
  report_args: ["--format", "json"]
  
format:
  args: ["--write", "**/*.{js,ts,jsx,tsx,json,md}"]
  
type_check:
  args: ["--noEmit", "--strict"]

# Legacy compatibility (for existing workflows)
executables: 
  npm: /usr/bin/npm
  git: /usr/bin/git
  node: /usr/bin/node
```
