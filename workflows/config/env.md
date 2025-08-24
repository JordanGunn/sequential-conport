---
Project environment reference.
---

```yaml
name: env
context:
  workspace_id: spexi

  project:
    id: spexi
    root_path: /home/jgodau/work/personal/spexi
    source_path: {.root_path}/loki
    tests_path: {.root_path}/tests
    docs_path: {.root_path}/docs

  # Language / framework
  language: python
  framework: poetry

  # Generic tool categories (high-level only)
  tools:
    package_manager: poetry

  # Explicit executables available in this workspace
  executables:
    poetry: /home/jgodau/.local/bin/poetry

  # Language-specific metadata (kept minimal)
  profiles:
    python:
      pyproject_path: /home/jgodau/work/personal/spexi/pyproject.toml
      venv_path: /home/jgodau/work/personal/spexi/.venv
      package_manager: poetry
      build_tool: poetry
      test_runner: pytest
      test_args: ["-v", "--tb=short"]
```
