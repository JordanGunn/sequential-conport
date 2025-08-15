---
description: Harden then ship: quality gate at max strictness, commit & push, log in ConPort.
---

```yaml
name: finalize
description: "Harden then ship: quality gate at max strictness, commit & push, log in ConPort."
inputs:
  task: { type: string, required: true }
  scope: { type: string }
  type:  { type: string, default: "feat" }
  scope_commit: { type: string }
  message: { type: string, default: "" }

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
        workflow: "ship"
        last_run: "{{ now_iso() }}"

  - id: dev_strict
    action: include
    file: .windsurf/workflows/conport/update.md
    with:
      scope: "{{ inputs.scope or '' }}"
      quality_gate:
        format: true
        lint: true
        typecheck: true
        tests: true
        coverage_threshold: 80
      commit_type: "{{ inputs.type }}"
      commit_scope: "{{ inputs.scope_commit or '' }}"
      commit_subject: "{{ inputs.message or inputs.task }}"
      do_stage: true
      do_commit: true
      do_push: true

  - id: progress
    when: "{{ steps.dev_strict and steps.dev_strict.commit_hash }}"
    action: include
    file: .windsurf/workflows/conport/log.md
    with:
      progress:
        status: "completed"
        description: "Shipped '{{ inputs.task }}' @ {{ steps.dev_strict.commit_hash }}"

outputs:
  success:
    status: ok
    message: "Shipped '{{ inputs.task }}'{{ steps.dev_strict and steps.dev_strict.commit_hash and (' @ '+steps.dev_strict.commit_hash[:7]) or '' }}"

```