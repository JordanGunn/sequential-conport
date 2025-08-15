---
description: Generate or update documentation for files in scope and commit if changed.
---

```yaml
name: document
description: "Generate or update documentation for files in scope and commit if changed."
inputs:
  task:  { type: string, required: true }
  scope: { type: string, required: true }
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
        requested_scope: "{{ inputs.scope }}"
        workflow: "document"
        last_run: "{{ now_iso() }}"
  - id: gen_docs
    action: coding_op
    tool: generate_docs_for_scope
    params:
      scope: "{{ inputs.scope }}"
      repo_root: "{{ context.project.root_path }}"
  - id: qa_and_commit
    action: include
    file: .windsurf/workflows/conport/update.md
    with:
      scope: "{{ inputs.scope }}"
      quality_gate:
        format: true
        lint: true
        typecheck: false
        tests: false
        coverage_threshold: 0
      commit_type: "docs"
      commit_scope: "auto"
      commit_subject: "{{ inputs.task }}"
      do_stage: true
      do_commit: true
      do_push: false
outputs:
  success:
    status: ok
    message: "Documentation updated."

```