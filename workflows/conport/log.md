---
description: Log new items into ConPort.
---

```yaml
name: log_conport
description: |
  Log new items into ConPort (decisions, progress, patterns, custom data).
  Safe & idempotent: confirms intent and avoids obvious duplicates.

inputs:
  decision:
    type: object
    required: false
    # { summary: str (req), rationale?: str, implementation_details?: str, tags?: [str] }
  progress:
    type: object
    required: false
    # { description: str (req), status?: 'TODO'|'IN_PROGRESS'|'DONE', parent_id?: int,
    #   linked_item_type?: str, linked_item_id?: str, link_relationship_type?: str }
  pattern:
    type: object
    required: false
    # { name: str (req), description?: str, tags?: [str] }
  custom:
    type: object
    required: false
    # { category: str (req), key: str (req), value: any (req) }
  auto_confirm:
    type: boolean
    required: false
    default: false

steps:
  # --- DECISION --------------------------------------------------------------
  - id: preview_decision
    when: "{{ inputs.decision and inputs.decision.summary }}"
    action: system
    output: |
      [DECISION PREVIEW]
      summary: {{ inputs.decision.summary }}
      rationale: {{ inputs.decision.rationale or '—' }}
      tags: {{ (inputs.decision.tags or []) | join(', ') }}

  - id: confirm_decision
    when: "{{ inputs.decision and inputs.decision.summary and not inputs.auto_confirm }}"
    action: user
    description: "Confirm logging this decision."

  - id: check_existing_decision
    when: "{{ inputs.decision and (inputs.auto_confirm or confirm_decision.approved) }}"
    action: conport_op
    tool: search_decisions_fts
    params:
      workspace_id: "{{ context.workspace_id }}"
      query_term: "{{ inputs.decision.summary }}"
      limit: 5

  - id: dedupe_decision
    when: "{{ inputs.decision and (inputs.auto_confirm or confirm_decision.approved) }}"
    action: coding_op
    tool: select_if
    params:
      condition: "{{ steps.check_existing_decision.results | any(lambda d: (d.summary or '') | lower == (inputs.decision.summary | lower)) }}"
      then: skip
      else: proceed

  - id: log_decision
    when: "{{ inputs.decision and (inputs.auto_confirm or confirm_decision.approved) and steps.dedupe_decision.result == 'proceed' }}"
    action: conport_op
    tool: log_decision
    params:
      workspace_id: "{{ context.workspace_id }}"
      summary: "{{ inputs.decision.summary }}"
      rationale: "{{ inputs.decision.rationale or '' }}"
      implementation_details: "{{ inputs.decision.implementation_details or '' }}"
      tags: "{{ inputs.decision.tags or [] }}"

  # --- PROGRESS --------------------------------------------------------------
  - id: preview_progress
    when: "{{ inputs.progress and inputs.progress.description }}"
    action: system
    output: |
      [PROGRESS PREVIEW]
      description: {{ inputs.progress.description }}
      status: {{ inputs.progress.status or 'IN_PROGRESS' }}

  - id: confirm_progress
    when: "{{ inputs.progress and inputs.progress.description and not inputs.auto_confirm }}"
    action: user
    description: "Confirm logging this progress entry."

  - id: find_progress
    when: "{{ inputs.progress and (inputs.auto_confirm or confirm_progress.approved) }}"
    action: conport_op
    tool: get_progress
    params:
      workspace_id: "{{ co_
```