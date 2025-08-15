---
description: Identify styles and patterns and log as Custom StyleImprint
---

```yaml
name: imprint
description: "Identify style/patterns and log as Custom Data: StyleImprint."
steps:
  - id: env
    action: include
    file: .windsurf/workflows/conport/.env.md
  - id: load
    action: include
    file: .windsurf/workflows/conport/load.md
  - id: extract
    action: coding_op
    tool: extract_style_imprint
    params:
      repo_root: "{{ context.project.root_path }}"
  - id: log_imprint
    action: conport_op
    tool: log_custom_data
    params:
      workspace_id: "{{ context.workspace_id }}"
      category: "StyleImprint"
      key: "repo-wide"
      value: "{{ steps.extract.imprint }}"
outputs:
  success:
    status: ok
    message: "Style imprint logged."
```