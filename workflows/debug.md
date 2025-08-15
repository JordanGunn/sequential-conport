---
description: Reproduce test failures, iterate fixes, and re-run tests until resolution.
---

```yaml
name: debug
description: "Reproduce test failures, iterate fixes, and re-run tests until resolution."
inputs:
  task:  { type: string, required: true }
  scope: { type: string, default: "" }   # file/dir; dir is recursive by default
  attempts: { type: integer, default: 2 } # re-run loops max
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
        workflow: "debug"
        last_run: "{{ now_iso() }}"
  - id: test_run
    action: shell_op
    tool: run_command
    params:
      executable: "{{ context.tools.package_manager }}"
      args:
        - "run"
        - "{{ context.tools.test_runner }}"
        - "{{ context.test.runner_args | join(' ') | default('-q') }}"
        - "{% if inputs.scope %}{{ inputs.scope }}{% else %}{{ context.project.tests_path }}{% endif %}"
      cwd: "{{ context.project.root_path }}"
    on_error:
      emit:
        status: failed
        message: "Initial tests failed; attempting fixes."
        stderr: "{{ step.stderr }}"
  - id: fix_loop
    when: "{{ steps.test_run.on_error is defined }}"
    action: include
    file: .windsurf/workflows/conport/update.md
    with:
      # Delegate edits to your standard dev loop (fmt/lint/types/tests + commit on diffs)
      scope: "{{ inputs.scope or '' }}"
      quality_gate:
        format: true
        lint: true
        typecheck: true
        tests: false
        coverage_threshold: 0
      commit_type: "fix"
      commit_scope: "debug"
      commit_subject: "{{ inputs.task }} (auto-fix pass)"
      do_stage: true
      do_commit: true
      do_push: false
  - id: test_verify
    when: "{{ steps.test_run.on_error is defined }}"
    action: shell_op
    tool: run_command
    params:
      executable: "{{ context.tools.package_manager }}"
      args: ["run","{{ context.tools.test_runner }}","{{ context.test.runner_args | join(' ') | default('-q') }}","{% if inputs.scope %}{{ inputs.scope }}{% else %}{{ context.project.tests_path }}{% endif %}"]
      cwd: "{{ context.project.root_path }}"
  - id: progress
    action: include
    file: .windsurf/workflows/conport/log.md
    with:
      progress:
        status: "{{ (steps.test_run.on_error is defined and steps.test_verify.exit_code == 0) or (steps.test_run.exit_code == 0) and 'completed' or 'IN_PROGRESS' }}"
        description: "Debug '{{ inputs.task }}' — tests {{ (steps.test_run.on_error is defined and steps.test_verify.exit_code == 0) or (steps.test_run.exit_code == 0) and 'passed' or 'still failing' }}"
outputs:
  success:
    status: ok
```