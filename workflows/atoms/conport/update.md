```yaml
# Include with:
#   action: Read the contents of the file.
#   file: .windsurf/workflows/atoms/conport/update.md
#   with:
#     active_patch:  { ... }        # optional map
#     product_patch: { ... }        # optional map
#     decision:      { summary: "...", rationale: "...", tags: [...] }  # optional
#     progress:      { description: "...", status: "IN_PROGRESS" }      # optional

name: update_conport_atom
description: "Safe, idempotent ConPort updates with explicit branching."

inputs:
  active_patch:  { type: object }
  product_patch: { type: object }
  decision:
    type: object
    properties:
      summary: { type: string }
      rationale: { type: string }
      implementation_details: { type: string }
      tags: { type: array }
  progress:
    type: object
    properties:
      description: { type: string }
      status: { type: string }

steps:
  # ---------- Active Context (merge → content) ----------
  - id: load_active
    when: "{{ inputs.active_patch }}"
    action: conport_op
    tool: get_active_context
    params_json: { workspace_id: "{{ context.workspace_id }}" }

  - id: merge_active
    when: "{{ inputs.active_patch }}"
    action: coding_op
    tool: merge_objects
    params_json:
      base:  "{{ steps.load_active.content or {} }}"
      patch: "{{ inputs.active_patch }}"

  - id: update_active
    when: "{{ inputs.active_patch }}"
    action: conport_op
    tool: update_active_context
    params_json:
      workspace_id: "{{ context.workspace_id }}"
      content: "{{ steps.merge_active.result }}"   # send final OBJECT

  # ---------- Product Context (merge → content) ----------
  - id: load_product
    when: "{{ inputs.product_patch }}"
    action: conport_op
    tool: get_product_context
    params_json: { workspace_id: "{{ context.workspace_id }}" }

  - id: merge_product
    when: "{{ inputs.product_patch }}"
    action: coding_op
    tool: merge_objects
    params_json:
      base:  "{{ steps.load_product.content or {} }}"
      patch: "{{ inputs.product_patch }}"

  - id: update_product
    when: "{{ inputs.product_patch }}"
    action: conport_op
    tool: update_product_context
    params_json:
      workspace_id: "{{ context.workspace_id }}"
      content: "{{ steps.merge_product.result }}"  # send final OBJECT

  # ---------- Decision (confirm → log) ----------
  - id: confirm_decision
    when: "{{ inputs.decision and inputs.decision.summary }}"
    action: system
    say: |
      Log decision?
      summary: {{ inputs.decision.summary }}
      rationale: {{ inputs.decision.rationale or '—' }}
      tags: {{ (inputs.decision.tags or []) | tojson }}

  - id: log_decision
    when: "{{ inputs.decision and inputs.decision.summary }}"
    action: conport_op
    tool: log_decision
    params_json:
      workspace_id: "{{ context.workspace_id }}"
      summary: "{{ inputs.decision.summary }}"
      rationale: "{{ inputs.decision.rationale or '' }}"
      implementation_details: "{{ inputs.decision.implementation_details or '' }}"
      tags: "{{ inputs.decision.tags or [] }}"

  # ---------- Progress (find → branch: update OR log) ----------
  - id: find_progress
    when: "{{ inputs.progress and inputs.progress.description }}"
    action: conport_op
    tool: get_progress
    params_json:
      workspace_id: "{{ context.workspace_id }}"
      limit: 100
    output_transform:
      existing: "{{ find_progress_for_task(outputs, inputs.progress.description) }}"

  # Branch A: update existing
  - id: update_progress
    when: "{{ inputs.progress and steps.find_progress.existing }}"
    action: conport_op
    tool: update_progress
    params_json:
      workspace_id: "{{ context.workspace_id }}"
      progress_id: "{{ steps.find_progress.existing.id }}"
      description: "{{ inputs.progress.description }}"
      # status optional on update; include if provided:
      {% raw %}{{ inputs.progress.status and 'status: \"' ~ inputs.progress.status ~ '\"' or '' }}{% endraw %}

  # Branch B: log new
  - id: log_progress
    when: "{{ inputs.progress and not steps.find_progress.existing }}"
    action: conport_op
    tool: log_progress
    params_json:
      workspace_id: "{{ context.workspace_id }}"
      status: "{{ inputs.progress.status or 'IN_PROGRESS' }}"
      description: "{{ inputs.progress.description }}"

  # ---------- Optional sanity checks (catch stringified JSON early) ----------
  - id: assert_types
    when: "{{ inputs.active_patch or inputs.product_patch }}"
    action: coding_op
    tool: assert_multiple_objects
    params_json:
      objects:
        - { name: "active_patch",  value: "{{ inputs.active_patch  or {} }}" }
        - { name: "product_patch", value: "{{ inputs.product_patch or {} }}" }
```