---
description: Selectively port unique items from Memory MCP into ConPort (idempotent).   Compares against existing ConPort data, skips duplicates, patches where safe,   and links relationships. Supports dry-run reporting by default.
---

```yaml
name: conport_migrate_memory
description: >
  Selectively port unique items from Memory MCP into ConPort (idempotent).
  Compares against existing ConPort data, skips duplicates, patches where safe,
  and links relationships. Supports dry-run reporting by default.

inputs:
  types:         { type: string,  default: "Task,Decision,SystemPattern,Glossary,CustomData" }
  since_hours:   { type: integer, default: 0 }     # 0 = no time filter
  limit:         { type: integer, default: 500 }   # soft guardrail
  dry_run:       { type: boolean, default: true }
  include_links: { type: boolean, default: true }
  report_only:   { type: boolean, default: false } # true => generate report even if dry_run=false

steps:
  # 0) Environment & Context
  - id: env
    action: include
    file: .windsurf/workflows/conport/.env.md

  - id: load
    action: include
    file: .windsurf/workflows/conport/load.md

  - id: pre_patch_active
    action: include
    file: .windsurf/workflows/conport/update.md
    with:
      active_patch:
        current_focus: "Migrate Memory → ConPort"
        requested_scope: "graph"
        workflow: "migrate_memory"
        last_run: "{{ now_iso() }}"

  # 1) Read Memory graph snapshot (nodes + edges)
  - id: mem_graph
    action: memory_op
    tool: read_graph
    params:
      # Implementers: your read_graph can ignore or support since filter; we pass meta anyway
      since_hours: "{{ inputs.since_hours }}"
      limit: "{{ inputs.limit }}"
    # expected: { nodes: [{id,name,entityType,observations,updated_at?...}], edges: [{from,to,relationType}] }

  # 2) Fetch existing ConPort state (for dedupe/match)
  - id: cp_decisions
    action: conport_op
    tool: get_decisions
    params: { workspace_id: "{{ context.workspace_id }}", limit: 1000 }

  - id: cp_patterns
    action: conport_op
    tool: get_system_patterns
    params: { workspace_id: "{{ context.workspace_id }}", tags_filter_include_any: ["*"] }

  - id: cp_progress
    action: conport_op
    tool: get_progress
    params: { workspace_id: "{{ context.workspace_id }}", limit: 1000 }

  - id: cp_glossary_search_seed
    action: conport_op
    tool: get_custom_data
    params: { workspace_id: "{{ context.workspace_id }}", category: "ProjectGlossary" }

  - id: cp_custom_any
    action: conport_op
    tool: get_custom_data
    params: { workspace_id: "{{ context.workspace_id }}" }

  # 3) Plan the migration (pure function; builds diffs + upserts)
  - id: plan
    action: coding_op
    tool: plan_migration_from_memory
    params:
      memory_nodes: "{{ steps.mem_graph.nodes }}"
      memory_edges: "{{ steps.mem_graph.edges }}"
      conport_existing:
        decisions: "{{ steps.cp_decisions.items }}"
        patterns:  "{{ steps.cp_patterns.items }}"
        progress:  "{{ steps.cp_progress.items }}"
        glossary:  "{{ steps.cp_glossary_search_seed.items }}"
        custom:    "{{ steps.cp_custom_any.items }}"
      include_types: "{{ inputs.types }}"
      match_rules:
        task_to_progress:
          # Prefer exact description match, else fuzzy on normalized name
          description_fields: ["name","observations"]
          status_default: "IN_PROGRESS"
        decision:
          match_on_summary: true
          fuzzy_threshold: 0.9
        pattern:
          match_on_name: true
        glossary:
          match_on_key: true
        custom:
          category_key_from_entityType: true
      limit: "{{ inputs.limit }}"
    # expected:
    #   { to_log:
    #      decisions:[{summary,rationale,implementation_details,tags}],
    #      progress:[{status,description}],
    #      patterns:[{name,description,tags}],
    #      glossary:[{key,value}],
    #      custom:[{category,key,value}],
    #     to_link:[{source_item_type,source_item_id,target_item_type,target_item_id,relationship_type,description}],
    #     deduped:{...}, skipped:{reasons:[]}, stats:{...}, notes:"..."
    #   }

  # 4) Dry-run / report path
  - id: render_report
    action: coding_op
    tool: render_migration_report_md
    params:
      plan: "{{ steps.plan }}"
      dry_run: "{{ inputs.dry_run }}"
      since_hours: "{{ inputs.since_hours }}"
      types: "{{ inputs.types }}"
  - id: show_report
    action: system
    say: |
      {{ steps.render_report.markdown }}

  - id: stop_if_report_only
    when: "{{ inputs.report_only or inputs.dry_run }}"
    action: system
    output: "Report only; no writes."

  # 5) Execute writes — batched where possible (idempotent upserts)
  - id: batch_decisions
    when: "{{ not inputs.dry_run and steps.plan.to_log.decisions and steps.plan.to_log.decisions|length>0 }}"
    action: conport_op
    tool: batch_log_items
    params:
      workspace_id: "{{ context.workspace_id }}"
      item_type: "decision"
      items: "{{ steps.plan.to_log.decisions }}"

  - id: batch_progress
    when: "{{ not inputs.dry_run and steps.plan.to_log.progress and steps.plan.to_log.progress|length>0 }}"
    action: conport_op
    tool: batch_log_items
    params:
      workspace_id: "{{ context.workspace_id }}"
      item_type: "progress_entry"
      items: "{{ steps.plan.to_log.progress }}"

  - id: batch_patterns
    when: "{{ not inputs.dry_run and steps.plan.to_log.patterns and steps.plan.to_log.patterns|length>0 }}"
    action: conport_op
    tool: batch_log_items
    params:
      workspace_id: "{{ context.workspace_id }}"
      item_type: "system_pattern"
      items: "{{ steps.plan.to_log.patterns }}"

  - id: batch_glossary
    when: "{{ not inputs.dry_run and steps.plan.to_log.glossary and steps.plan.to_log.glossary|length>0 }}"
    action: conport_op
    tool: batch_log_items
    params:
      workspace_id: "{{ context.workspace_id }}"
      item_type: "custom_data"
      items: "{% for g in steps.plan.to_log.glossary %}{ 'category':'ProjectGlossary','key': g.key,'value': g.value }{% endfor %}"

  - id: batch_custom
    when: "{{ not inputs.dry_run and steps.plan.to_log.custom and steps.plan.to_log.custom|length>0 }}"
    action: conport_op
    tool: batch_log_items
    params:
      workspace_id: "{{ context.workspace_id }}"
      item_type: "custom_data"
      items: "{{ steps.plan.to_log.custom }}"

  # 6) Link graph edges (optional)
  - id: link_edges
    when: "{{ not inputs.dry_run and inputs.include_links and steps.plan.to_link and steps.plan.to_link|length>0 }}"
    action: coding_op
    tool: apply_conport_links
    params:
      workspace_id: "{{ context.workspace_id }}"
      links: "{{ steps.plan.to_link }}"
      # Implementer batches into multiple link_conport_items calls; handles failures individually

  # 7) Record the migration summary into ConPort for traceability
  - id: log_report
    when: "{{ not inputs.dry_run }}"
    action: conport_op
    tool: log_custom_data
    params:
      workspace_id: "{{ context.workspace_id }}"
      category: "MigrationReports"
      key: "Memory→ConPort {{ now_iso() }}"
      value:
        types: "{{ inputs.types }}"
        since_hours: "{{ inputs.since_hours }}"
        stats: "{{ steps.plan.stats }}"
        skipped: "{{ steps.plan.skipped }}"
        notes: "{{ steps.plan.notes }}"

  # 8) Post patch active with result
  - id: post_patch_active
    action: include
    file: .windsurf/workflows/conport/update.md
    with:
      active_patch:
        workflow: "migrate_memory"
        last_run: "{{ now_iso() }}"
        migrated_types: "{{ inputs.types }}"
        since_hours: "{{ inputs.since_hours }}"

outputs:
  success:
    status: ok
    message: >
      Memory→ConPort migration {{ inputs.dry_run and 'planned (dry-run)' or 'completed' }}.

```