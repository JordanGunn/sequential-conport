---
description: 
---

```yaml
name: conport_plan
description: "Maintain ${.root_path}/plan.md: close done items, generate next steps, and log progress."
inputs:
  plan_path: { type: string, default: "plan.md" }
steps:
  - id: env
    action: include
    file: .windsurf/workflows/conport/.env.md
  - id: load
    action: include
    file: .windsurf/workflows/conport/load.md
  - id: read_plan
    action: fs_op
    tool: read_text_if_exists
    params: { path: "{{ inputs.plan_path }}" }
  - id: summarize_progress
    action: conport_op
    tool: get_progress
    params:
      workspace_id: "{{ context.workspace_id }}"
      limit: 100
  - id: clean_plan
    action: coding_op
    tool: reconcile_plan_with_progress
    params:
      plan_md: "{{ steps.read_plan.text or '' }}"
      progress: "{{ steps.summarize_progress.items }}"
  - id: write_plan
    action: fs_op
    tool: write_text
    params:
      path: "{{ inputs.plan_path }}"
      text: "{{ steps.clean_plan.updated_plan_md }}"
  - id: think_next
    action: sequential_thinking
    tool: sequential_thinking
    params:
      thought: "Draft a minimal, testable next-steps plan based on current contexts and cleaned plan.md."
      nextThoughtNeeded: false
      thoughtNumber: 1
      totalThoughts: 1
  - id: append_next
    action: fs_op
    tool: append_text
    params:
      path: "{{ inputs.plan_path }}"
      text: "\n\n## Next Steps\n{{ steps.think_next.thought }}"
  - id: log
    action: include
    file: .windsurf/workflows/conport/log.md
    with:
      progress:
        status: "IN_PROGRESS"
        description: "Plan updated; next steps drafted."
outputs:
  success:
    status: ok
    message: "Plan reconciled and extended."
 ```