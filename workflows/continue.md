---
description: Continue development idempotently.
---

```yaml
name: continue
description: "Continue dev idempotently: patch Active Context, respect scope/recursive, run quality + commit/push."
inputs:
  task: { type: string, required: true }
  scope: { type: string }
  recursive:
    type: boolean
    default: true
    description: "If scope is a directory, recurse by default."
  stage:   { type: boolean, default: true }
  commit:  { type: boolean, default: true }
  push:    { type: boolean, default: false }
  type:    { type: string, default: "feat" }
  scope_commit: { type: string }
  message: { type: string, default: "" }
  quality_gate:
    type: object
    default: { format: true, lint: true, typecheck: true, tests: true, coverage_threshold: 0 }

steps:
  - id: env
    action: include
    file: .windsurf/workflows/conport/.env.md

  # Ensure contexts are loaded first
  - id: load
    action: include
    file: .windsurf/workflows/conport/load.md

  # Patch Active Context (object patch, not stringified)
  - id: pre_patch
    action: include
    file: .windsurf/workflows/conport/update.md
    with:
      active_patch:
        current_focus: "{{ inputs.task }}"
        requested_scope: "{{ inputs.scope or '' }}"
        recursive: "{{ inputs.recursive }}"
        workflow: "continue"
        last_run: "{{ now_iso() }}"

  # Run the development loop (fmt/lint/types/tests) and commit if diffs
  - id: dev
    action: include
    file: .windsurf/workflows/conport/update.md
    with:
      run_trace: false
      run_deps: false
      scope: "{{ inputs.scope or '' }}"
      quality_gate: "{{ inputs.quality_gate }}"
      commit_type: "{{ inputs.type }}"
      commit_scope: "{{ inputs.scope_commit or '' }}"
      commit_subject: "{{ inputs.message or inputs.task }}"
      do_stage: "{{ inputs.stage }}"
      do_commit: "{{ inputs.commit }}"
      do_push: "{{ inputs.push }}"

  # Post-patch with branch + last_commit from update flow if available
  - id: post_patch
    when: "{{ steps.dev and steps.dev.commit_hash }}"
    action: include
    file: .windsurf/workflows/conport/update.md
    with:
      active_patch:
        current_focus: "{{ inputs.task }}"
        requested_scope: "{{ inputs.scope or '' }}"
        recursive: "{{ inputs.recursive }}"
        workflow: "continue"
        branch: "{{ steps.dev.branch }}"
        last_commit: "{{ steps.dev.commit_hash }}"
        last_run: "{{ now_iso() }}"

outputs:
  success:
    status: ok
    message: "Continued: '{{ inputs.task }}'{{ steps.dev and steps.dev.commit_hash and (' @ '+steps.dev.commit_hash[:7]) or '' }}"
```