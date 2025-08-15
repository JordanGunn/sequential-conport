---
description: Summarize Product/Active contexts, recent decisions/progress, and git changes.
---

```yaml
name: conport_report
description: "Summarize Product/Active contexts, recent decisions/progress, and git changes."
inputs:
  hours_ago: { type: integer, default: 48 }
steps:
  - id: env
    action: include
    file: .windsurf/workflows/conport/.env.md
  - id: load
    action: include
    file: .windsurf/workflows/conport/load.md
  - id: recent
    action: conport_op
    tool: get_recent_activity_summary
    params:
      workspace_id: "{{ context.workspace_id }}"
      hours_ago: "{{ inputs.hours_ago }}"
      limit_per_type: 5
  - id: git_status
    action: shell_op
    tool: run_shell
    params:
      cmd: "git status --porcelain"
      cwd: "{{ context.project.root_path }}"
  - id: say
    action: system
    say: |
      ## ConPort Report ({{ inputs.hours_ago }}h)
      **Product Context (keys):** {{ steps.load.product_keys }}
      **Active Context:** {{ steps.load.active_summary }}
      **Recent:** {{ steps.recent.summary }}
      **Uncommitted files:** 
      {{ steps.git_status.stdout or 'None' }}
outputs:
  success:
    status: ok
    message: "Report displayed."

```