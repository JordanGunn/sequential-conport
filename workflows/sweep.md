---
description: Sweep recent activity activity with git and note changes.
---

```yaml
name: conport_sweep
description: "Sweep recent activity: git status/log/diff → ConPort progress & Active Context."
inputs:
  since: { type: string, default: "7.days" }
  task:  { type: string, default: "Planning sweep" }
  dry_run: { type: boolean, default: false }

steps:
  - id: env
    action: include
    file: .windsurf/workflows/conport/.env.md

  - id: load
    action: include
    file: .windsurf/workflows/conport/load.md

  - id: pre
    action: include
    file: .windsurf/workflows/conport/update.md
    with:
      active_patch:
        current_focus: "{{ inputs.task }}"
        workflow: "plan"
        last_run: "{{ now_iso() }}"

  - id: sync
    action: include
    file: .windsurf/workflows/conport/sync.md
    with:
      since: "{{ inputs.since }}"
      dry_run: "{{ inputs.dry_run }}"

outputs:
  success:
    status: ok
    message: "Sweep complete (since={{ inputs.since }})"
```