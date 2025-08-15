---
description: Lint and auto-correct source files using configured language tools.
---

```yaml
name: lint
description: "Lint & refactor safely: format/imports auto-fix; plan & confirm for risky changes."

inputs:
  targets:       { type: list,   default: ["." ] }       # files/dirs
  mode:          { type: string, default: "safe" }       # safe|plan|apply
  confirm:       { type: boolean, default: true }        # require confirmation before edits
  create_branch: { type: boolean, default: true }
  run_tests:     { type: boolean, default: false }
  max_deletions: { type: integer, default: 10 }          # hard stop if exceeded
  max_changed:   { type: integer, default: 200 }         # lines (add+del) guard

steps:
  - id: env
    action: include
    file: .windsurf/workflows/conport/.env.md
  - id: ack
    action: system
    output: |
      Lint strategy:
      1) Auto-fix only non-destructive items (formatter, import organizer).
      2) Collect lint diagnostics → build refactor plan.
      3) Apply plan only after guardrails & confirmation. Never mass-delete code.

  - id: ensure_install
    action: shell_op
    tool: run_command
    params:
      executable: "{{ context.tools.package_manager }}"
      args: ["install"]
      cwd: "{{ context.project.root_path }}"

  # --- Safety branch ---
  - id: branch_name
    when: "{{ inputs.create_branch }}"
    action: shell_op
    tool: run_shell
    params:
      cmd: "echo lint/$(date +%Y%m%d-%H%M%S)"
      cwd: "{{ context.project.root_path }}"

  - id: create_branch
    when: "{{ inputs.create_branch }}"
    action: shell_op
    tool: run_shell
    params:
      cmd: "git checkout -b {{ steps.branch_name.stdout|trim }}"
      cwd: "{{ context.project.root_path }}"

  # --- Non-destructive auto-fixes ---
  - id: format
    action: shell_op
    tool: run_command
    params:
      executable: "{{ context.tools.package_manager }}"
      args: ["run","{{ context.tools.formatter }}"] + inputs.targets
      cwd: "{{ context.project.root_path }}"

  - id: organize_imports
    when: "{{ context.tools.import_organizer }}"
    action: shell_op
    tool: run_command
    params:
      executable: "{{ context.tools.package_manager }}"
      args: ["run","{{ context.tools.import_organizer }}"] + inputs.targets
      cwd: "{{ context.project.root_path }}"

  # Safe linter pass: only safe auto-fixes
  - id: lint_safe
    when: "{{ context.lint.safe_autofix_args }}"
    action: shell_op
    tool: run_command
    params:
      executable: "{{ context.tools.package_manager }}"
      args: ["run","{{ context.tools.linter }}"] + context.lint.safe_autofix_args + inputs.targets
      cwd: "{{ context.project.root_path }}"

  # --- Full diagnostics (no fixing) ---
  - id: lint_report
    action: shell_op
    tool: run_command
    params:
      executable: "{{ context.tools.package_manager }}"
      args: ["run","{{ context.tools.linter }}"] + context.lint.report_args + inputs.targets
      cwd: "{{ context.project.root_path }}"

    on_error:
      # linter exits nonzero when issues exist; capture but continue
      emit:
        status: "lint_issues_found"
        stderr: "{{ step.stderr }}"
        stdout: "{{ step.stdout }}"

  - id: type_report
    when: "{{ context.tools.type_checker }}"
    action: shell_op
    tool: run_command
    params:
      executable: "{{ context.tools.package_manager }}"
      args: ["run","{{ context.tools.type_checker }}"] + inputs.targets
      cwd: "{{ context.project.root_path }}"
    on_error:
      emit:
        status: "type_issues_found"
        stderr: "{{ step.stderr }}"
        stdout: "{{ step.stdout }}"

  # --- Build a human-safe refactor plan (no mass deletions) ---
  - id: plan_refactors
    action: coding_op
    tool: build_lint_refactor_plan
    params:
      lint_output: "{{ steps.lint_report.stdout }}"
      type_output: "{{ steps.type_report.stdout or '' }}"
      policy:
        # Never remove code blocks to satisfy a rule; prefer renames or adding handling.
        allow_deletions: false
        # Permit removing truly-unused imports only with confirmation.
        allow_remove_unused_imports: false
        # Preferred remediations:
        strategy_order:
          - "rename_unused_vars_to_underscore"     # e.g., x -> _x for F841
          - "add_explicit_returns_or_raises"       # to fix unreachable/flow issues
          - "introduce_narrower_excepts"           # B broad except
          - "add_type_annotations_or_narrow_types" # mypy
          - "split_large_functions"                # C90 complexity → refactor not delete
          - "replace_bare_except_with_specific"
          - "introduce_const_or_helper_function"
        # Rules we will NEVER auto-fix, only suggest:
        never_autofix:
          - "F401"   # unused import (ask first)
          - "F841"   # unused local (prefer rename to _x)
          - "PLR0915" # too many statements
          - "C901"   # complexity
          - "ERA"    # commented-out code (don't delete automatically)
      targets: "{{ inputs.targets }}"

  # Show plan and require confirmation unless mode forces behavior
  - id: present_plan
    action: system
    say: |
      Lint plan ready (non-destructive). Mode={{ inputs.mode }}.
      - Proposed edits: {{ steps.plan_refactors.summary.change_count }}
      - Estimated deletions: {{ steps.plan_refactors.summary.deletions }}
      - Estimated total changed lines: {{ steps.plan_refactors.summary.total_changed }}
      Proceed?

  # Guardrails
  - id: guardrails
    action: system
    when: "{{ steps.plan_refactors.summary.deletions > inputs.max_deletions or steps.plan_refactors.summary.total_changed > inputs.max_changed }}"
    output: "Refactor plan exceeds guardrails; edits will NOT be applied."

  # Apply only when allowed and confirmed
  - id: maybe_apply
    when: >
      {{ inputs.mode in ['apply','safe'] and
         not steps.guardrails and
         (inputs.confirm == false) }}
    action: coding_op
    tool: perform_task_edits
    params:
      plan: "{{ steps.plan_refactors.plan }}"
      repo_root: "{{ context.project.root_path }}"

  # Optional test run after changes
  - id: run_tests
    when: "{{ inputs.run_tests }}"
    action: shell_op
    tool: run_command
    params:
      executable: "{{ context.tools.package_manager }}"
      args: ["run","{{ context.tools.test_runner }}","-q"]
      cwd: "{{ context.project.root_path }}"

  # Final reports
  - id: lint_post
    action: shell_op
    tool: run_command
    params:
      executable: "{{ context.tools.package_manager }}"
      args: ["run","{{ context.tools.linter }}","check"] + inputs.targets
      cwd: "{{ context.project.root_path }}"
```