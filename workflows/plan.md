---
description: Create a development plan and reconcile with ConPort progress, appending next steps, and committing if changed.
---

```yaml
---
description: Maintain plan.md: reconcile with ConPort progress, append next steps, and commit if changed. Hooked; no tests.
---

name: plan
description: |
  Maintain {{ context.project.root_path }}/plan.md (or a custom path).
  - Preflight to load env/policies + ConPort/Git grounding
  - Close/mark done items in plan from current ConPort progress
  - Append a concise “Next Steps” section
  - Format/lint only (no tests); commit if there are diffs
  - Postflight to sync/log/export as configured

inputs:
  plan_path: { type: string, required: false, default: "plan.md" }
  pre_opts:  { type: object, required: false, default: {} }
  post_opts:
    type: object
    required: false
    default: { capture_git_snapshot: true, log_run_record: true, export_on_change: true }

steps:
  # ---------- PRE HOOK ----------
  - id: preflight
    action: Read the contents of the file.
    file: .windsurf/workflows/hooks/pre.md
    with: "{{ inputs.pre_opts }}"

  # ---------- Load/prepare existing plan ----------
  - id: read_plan
    action: fs_op
    tool: read_text_if_exists
    params:
      path: "{{ inputs.plan_path }}"

  # ---------- Pull recent progress from ConPort (source of truth) ----------
  - id: get_progress
    action: conport_op
    tool: get_progress
    params:
      workspace_id: "{{ context.workspace_id }}"
      limit: 200

  # ---------- Reconcile plan with progress ----------
  - id: clean_plan
    action: coding_op
    tool: reconcile_plan_with_progress
    params:
      plan_md: "{{ steps.read_plan.text or '' }}"
      progress: "{{ steps.get_progress.results or [] }}"

  - id: write_plan
    action: fs_op
    tool: write_text
    params:
      path: "{{ inputs.plan_path }}"
      text: "{{ steps.clean_plan.updated_plan_md }}"

  # ---------- Draft minimal, testable next steps ----------
  - id: think_next
    action: sequential_thinking
    tool: sequential_thinking
    params:
      thought: |
        Draft a short, concrete “Next Steps” list based on the cleaned plan and current progress.
        Prefer 3–6 bullets, each action-oriented and verifiable.
        Avoid speculative architecture; keep it executable within 1–2 working sessions.
      nextThoughtNeeded: false
      thoughtNumber: 1
      totalThoughts: 1

  - id: append_next
    action: fs_op
    tool: append_text
    params:
      path: "{{ inputs.plan_path }}"
      text: |
        {% raw %}

        ## Next Steps
        {{ steps.think_next.thought }}
        {% endraw %}

  # ---------- Format/lint and commit if changed (docs-only discipline) ----------
  - id: qa_and_commit
    action: Read the contents of the file.
    file: .windsurf/workflows/atoms/conport/update.md
    with:
      scope: "{{ inputs.plan_path }}"
      quality_gate:
        format: true
        lint: true
        typecheck: false
        tests: false
        coverage_threshold: 0
      commit_type: "docs"
      commit_scope: "plan"
      commit_subject: "Update plan and next steps"
      do_stage: true
      do_commit: true
      do_push: false

  # ---------- Prepare progress payload for postflight ----------
  - id: progress_payload
    action: system
    output_transform:
      progress:
        - description: >-
            Plan reconciled and extended — {{ inputs.plan_path }}
            {{ steps.qa_and_commit.commit_hash and 'committed '+steps.qa_and_commit.commit_hash[:7] or 'no changes' }}
          status: "{{ steps.qa_and_commit.commit_hash and 'DONE' or 'IN_PROGRESS' }}"

  # ---------- POST HOOK ----------
  - id: postflight
    action: Read the contents of the file.
    file: .windsurf/workflows/hooks/post.md
    with:
      decisions: []
      progress: "{{ steps.progress_payload.progress }}"
      active_patch:
        current_focus: "Plan maintenance"
        requested_scope: "{{ inputs.plan_path }}"
        workflow: "plan"
        last_run: "{{ now_iso() }}"
      {{ inputs.post_opts | tojson }}

# -------------------------------- Outputs ------------------------------------
outputs:
  success:
    status: ok
    message: >-
      Plan reconciled and extended.
      {{ steps.qa_and_commit.commit_hash and ('commit='+steps.qa_and_commit.commit_hash[:7]) or '(no changes)' }}
```