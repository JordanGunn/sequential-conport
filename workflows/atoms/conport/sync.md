---
description:  Synchronize ConPort with the current conversation state.   Ensures new context, decisions, progress, and relations are captured   without duplication. Designed to be idempotent.
---

```yaml
name: sync_conport_atom
description: |
  Synchronize ConPort with the current conversation state.
  Ensures new context, decisions, progress, and relations are captured
  without duplication. Designed to be idempotent.

inputs:
  active_patch:
    type: object
    required: false
    description: Optional updates to Active Context
  product_patch:
    type: object
    required: false
    description: Optional updates to Product Context
  decisions:
    type: array
    required: false
    items:
      type: object
      properties:
        summary: { type: string }
        rationale: { type: string }
  progress:
    type: array
    required: false
    items:
      type: object
      properties:
        description: { type: string }
        status: { type: string }

steps:
  - id: ack
    action: system
    description: "Signal that sync has started."
    output: "[CONPORT_SYNCING]"

  - id: update
    action: Read the contents of the file.
    file: update.md
    with:
      active_patch: "{{ inputs.active_patch }}"
      product_patch: "{{ inputs.product_patch }}"
      decisions: "{{ inputs.decisions | default([]) }}"
      progress: "{{ inputs.progress | default([]) }}"

  - id: confirm
    action: conport_op
    tool: get_recent_activity_summary
    params_json:
      workspace_id: "{{ context.workspace_id }}"

  - id: notify
    action: system
    description: "Signal completion of sync."
    output: "[CONPORT_SYNC_COMPLETE]"

outputs:
  success:
    status: ok
    message: "ConPort synchronized with conversation."
    summary: "{{ steps.confirm }}"
```
