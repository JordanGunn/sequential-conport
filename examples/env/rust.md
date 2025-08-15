---
description: Example .env.md configuration for Rust projects using Cargo
---

```yaml
# Rust + Cargo Environment Configuration Example
# Copy this to your project's .windsurf/workflows/conport/.env.md and customize paths

# Project Structure
project:
  root_path: "/path/to/your/rust/project"
  src_path: "/path/to/your/rust/project/src"
  tests_path: "/path/to/your/rust/project/tests"
  docs_path: "/path/to/your/rust/project/docs"
  
# Language/Framework Detection
language: rust
framework: cargo

# Generic Tool Categories (language-agnostic)
tools:
  package_manager: cargo
  test_runner: cargo
  formatter: rustfmt
  linter: clippy
  type_checker: cargo  # Built into rustc
  import_organizer: null  # Rust doesn't need separate import organizer
  dependency_analyzer: cargo
  build_tool: cargo

# Tool-specific configurations
test:
  runner_args: ["test", "--verbose"]
  coverage_threshold: 80
  
lint:
  safe_autofix_args: ["clippy", "--fix", "--allow-dirty"]
  report_args: ["clippy", "--message-format", "json"]
  
format:
  args: ["fmt"]
  
type_check:
  args: ["check"]

# Legacy compatibility (for existing workflows)
executables: 
  cargo: /home/user/.cargo/bin/cargo
  git: /usr/bin/git
  rustc: /home/user/.cargo/bin/rustc
  rustfmt: /home/user/.cargo/bin/rustfmt
```
