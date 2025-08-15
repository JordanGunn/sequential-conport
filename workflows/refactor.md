---
description: 
---

```yaml
name: conport_refactor
description: "Plan and execute a safe refactor: log a Decision, apply edits, run strict quality, and link results."

inputs:
  task:      { type: string, required: true }   # e.g., "Split tabular CLI into subcommands"
  scope:     { type: string, required: true }   # file or directory (directory implies recursive)
  rationale: { type: string, default: "" }      # optional human rationale to seed planning
  dry_run:   { type: boolean, default: false }
  push:      { type: boolean, default: false }

steps:
  # 0) Env + load (centralized)
  - id: env
    action: include
    file: .windsurf/workflows/conport/.env.md

  - id: load
    action: include
    file: .windsurf/workflows/conport/load.md

  # 1) Pre-patch Active Context (OBJECT patch, never string)
  - id: pre_patch
    action: include
    file: .windsurf/workflows/conport/update.md
    with:
      active_patch:
        current_focus: "{{ inputs.task }}"
        requested_scope: "{{ inputs.scope }}"
        workflow: "refactor"
        last_run: "{{ now_iso() }}"

  # 2) Analyze & plan a refactor (deterministic summary to aid idempotency)
  - id: plan
    action: coding_op
    tool: propose_refactor_plan
    params:
      scope: "{{ inputs.scope }}"
      repo_root: "{{ context.project.root_path }}"
      rationale_seed: "{{ inputs.rationale }}"
      # expected: { summary, detailed_plan_md, risks, touched_paths[], test_impact[] }

  # 3) Log a Decision in ConPort (source of truth for WHY)
  - id: log_decision
    action: conport_op
    tool: log_decision
    params:
      workspace_id: "{{ context.workspace_id }}"
      summary: "Refactor: {{ inputs.task }} (scope={{ inputs.scope }})"
      rationale: "{{ inputs.rationale or plan.risks or plan.summary }}"
      implementation_details: "{{ steps.plan.detailed_plan_md }}"
      tags: ["refactor","architecture","conport_chain"]
    # expect: id → steps.log_decision.id

  # 4) Apply edits via your standard dev loop (strict but practical gates)
  - id: apply_and_validate
    when: "{{ not inputs.dry_run }}"
    action: include
    file: .windsurf/workflows/conport/update.md
    with:
      scope: "{{ inputs.scope }}"
      quality_gate:
        format: true
        lint: true
        typecheck: true
        tests: true
        coverage_threshold: 0
      commit_type: "refactor"
      commit_scope: "auto"
      commit_subject: "{{ inputs.task }}"
      do_stage: true
      do_commit: true
      do_push: "{{ inputs.push }}"
    # expose: commit_hash, branch (as your update.md already does)

  # 5) Link the Decision → Progress/Commit (if we committed)
  - id: log_progress
    when: "{{ not inputs.dry_run and steps.apply_and_validate and steps.apply_and_validate.commit_hash }}"
    action: conport_op
    tool: log_progress
    params:
      workspace_id: "{{ context.workspace_id }}"
      status: "completed"
      description: "Refactor '{{ inputs.task }}' @ {{ steps.apply_and_validate.commit_hash }}"
    # expect: id → steps.log_progress.id

  - id: link_decision_to_progress
    when: "{{ steps.log_progress and steps.log_decision and steps.apply_and_validate.commit_hash }}"
    action: conport_op
    tool: link_conport_items
    params:
      workspace_id: "{{ context.workspace_id }}"
      source_item_type: "decision"
      source_item_id: "{{ steps.log_decision.id }}"
      target_item_type: "progress_entry"
      target_item_id: "{{ steps.log_progress.id }}"
      relationship_type: "implements"
      description: "Progress entry implements the refactor decision."

  # 6) Post-patch Active Context with branch/commit (if present)
  - id: post_patch
    when: "{{ steps.apply_and_validate and steps.apply_and_validate.commit_hash }}"
    action: include
    file: .windsurf/workflows/conport/update.md
    with:
      active_patch:
```