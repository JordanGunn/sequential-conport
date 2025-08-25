# Context-persistent MCP framework.

A modular workflow system for AI agents that turns your repo into a self‑documenting, idempotent workbench. 

## Core Architecture

The framework is built on five interconnected components:

**Atoms** - Miniature workflows based on verbs that are directly related to MCP tools. Used as building blocks for all top-level workflows to ensure consistency.

**Hooks** - Agent instructions that execute before and after every workflow (`pre → [main steps] → post`).

**Policies** - Natural language descriptions (always, never, personality) that guide agent behavior. New policies can be added by creating files under `policies/` and adding them to `policies/.index.md`.

**Dev Module** - Contains the core development loop and configurable quality gates. Quality gates have boolean flags for turning on/off and are consumed from the env language profile. If no quality tools exist, they are ignored. The loop consumes a prompted task and safely performs it in a consistent manner.

**Top-level Workflows** - Follow the pattern: `func(prompt) → pre-hook → instructions(loop(prompt), quality) → post-hook`

Everything is designed to be safe, repeatable, and composable.


---

## Quick Setup

After ensuring dependencies and prerequisites are installed, you only need to edit:

1. **`workflows/config/env.md`** - Configure your development environment (paths, tools, language profile)
2. **`workflows/policies/`** - Customize agent behavior with natural language descriptions:
   - Edit existing policies: `always.md`, `never.md`, `personality.md`
   - Add new policies by creating files and adding them to `policies/.index.md`

All policies are natural language descriptions - no code changes needed.

---

## Architecture Details

### Components Overview

#### **Atoms** - Building Blocks
Miniature workflows based on verbs that are directly related to MCP tools. Used as building blocks for all top-level workflows to ensure consistency:
- `atoms/conport/` - ConPort operations: initialize, load, search, relate, log, sync, export
- `atoms/git/` - Git operations: status, diff, stage, commit, sweep
- All workflows compose these atoms rather than implementing MCP calls directly

#### **Hooks** - Execution Framework  
Agent instructions that execute before and after every workflow:
- `hooks/pre.md` - Preflight: loads env, policies; takes ConPort + Git snapshots
- `hooks/post.md` - Postflight: syncs ConPort, exports snapshots, logs results
- Pattern: `pre → [workflow instructions] → post`

#### **Policies** - Behavioral Guidelines
Natural language descriptions that guide agent behavior:
- `policies/always.md` - Standing positive instructions ("do this")
- `policies/never.md` - Guardrails and prohibitions ("don't do this") 
- `policies/personality.md` - Role, tone, and behavioral guidelines
- `policies/.index.md` - Registry of active policies (add new policies here)

#### **Dev Module** - Development Loop
Contains the core development loop and configurable quality gates:
- `dev/loop.md` - VCS-free dev loop that consumes a prompted task and safely performs it
- `dev/quality.md` - Quality gates with boolean flags for format/lint/types/tests
- Quality gates consume settings from env language profile; missing tools are ignored

#### **Top-level Workflows** - Complete Operations
Follow the pattern: `func(prompt) → pre-hook → instructions(loop(prompt), quality) → post-hook`
- Examples: `continue`, `bootstrap`, `refactor`, `tests`, `document`, `plan`, `report`
- Each handles specific development tasks while maintaining consistency

#### **Configuration** - Environment Setup
- `config/env.md` - Allows swapping language files, provides factual dev environment info
- `config/brief.md` - High-level project overview and contextual information for agents

### Directory Structure

```
.windsurf/
  workflows/
    config/
      env.md          # environment facts (paths, executables, language profile)
      paths.md        # optional path manifest for portable includes
      brief.md        # high-level project overview for the agent
    policies/
      .index.md
      always.md       # natural language “do this”
      never.md        # natural language “don’t do this”
      personality.md  # natural language “how to behave”
    hooks/
      pre.md          # preflight: load env, policies; light ConPort + Git snapshots
      post.md         # postflight: optional git snapshot; ConPort sync/export/log
    atoms/
      conport/        # initialize, load, search, relate, log, sync, export…
      git/            # status, diff_* , stage, commit, sweep…
    dev/
      .index.md       # module index & usage
      quality.md      # quality gates (format/lint/types/tests), toggleable & overridable
      loop.md         # VCS-free dev loop (task+scope → quality), emits a summary
    # high-level workflows live here (continue, bootstrap, refactor, tests, document, plan, report, …)
```


---

## 2) Configuration

### `config/env.md` (environment facts)
Allows swapping of language files and provides a location for factual information regarding your development environment. Keep this file focused on facts only:

* Project paths (`root_path`, `tests_path`, etc.)
* Executable paths you actually know (e.g., Poetry path)
* Language/framework profile (e.g., `language: python`, `framework: poetry`)
* Minimal test defaults (`profiles.python.test_args`)
* Language profiles (see `examples/env/`)

### `config/brief.md`
Provides high-level overview and contextual information about your project in natural language for your agent:
* A high-level description of your project
* Provides context, end-goal, constraints, etc.

### `policies/` (Natural Language Descriptions)
Current policies are `always`, `never`, and `personality`. All are **plain English**:
* `always.md` - Standing positive instructions (what to do)
* `never.md` - Guardrails and prohibitions (what not to do) 
* `personality.md` - Role, tone, and behavioral guidelines

**Adding New Policies:**
1. Create a new `.md` file under `policies/` with natural language descriptions
2. Add the new policy to `policies/.index.md` to make it active
3. No code changes needed - policies are just natural language descriptions

### Quality gates (dev module)
* **Toggle gates** via booleans:
  * `run_format`, `run_lint`, `run_types`, `run_tests`
  
* **Override commands** per run:
  * Arrays: `fmt_cmds`, `lint_cmds`, `types_cmds`
  * String: `tests_cmd`
  
* If a gate is **off** or a command list is **empty**, that gate is skipped.
* Commands are **derived from env** at runtime.

Example (from a higher‑level workflow calling the dev loop):

```yaml
- id: dev
  action: Read the contents of the file.
  file: .windsurf/workflows/dev/loop.md
  with:
    task:  "{{ inputs.task }}"
    scope: "{{ inputs.scope or '.' }}"
    run_format:  "{{ inputs.quality_gate.format }}"
    run_lint:    "{{ inputs.quality_gate.lint }}"
    run_types:   "{{ inputs.quality_gate.typecheck }}"
    run_tests:   "{{ inputs.quality_gate.tests }}"
    # Optional per-run overrides:
    # fmt_cmds:   ["poetry run black {{ inputs.scope }}", "poetry run isort {{ inputs.scope }}"]
    # lint_cmds:  ["poetry run ruff check --fix --select I {{ inputs.scope }}"]
    # types_cmds: ["poetry run mypy {{ inputs.scope }}"]
    # tests_cmd:  "poetry run pytest -q"
```

---

## 3) Hooks
* **Pre (`hooks/pre.md`)**
  1. Reads `env.md`; 
  2. Loads policies; 
  3. Takes a light ConPort snapshot (status, counts, focus)
  4. Performs Git sweep (commits, previews). 
  5. Emits a deterministic `preflight` object for downstream steps.

* **Postflight (`hooks/post.md`)**
  1. Optionally captures Git, 
  2. syncs ConPort with any decisions/progress, 
  3. optionally exports a snapshot and logs a run record. 
  4. Emits `postflight` summary.

> Hooks are used by all top‑level workflows. They keep environment + context handling consistent and out of the dev loop and workflows.

---

## 4) Dev Module

Contains the core development loop and configurable quality gates:

* **`dev/loop.md`**: VCS-free development orchestration that consumes a prompted **task** and safely performs the task in a consistent manner. Takes a prompted task + scope, executes it, and applies configured quality gates.

* **`dev/quality.md`**: Configurable quality gates with boolean flags for turning on/off (format, lint, types, tests). Quality gates are consumed from the env language profile. If no quality tools exist, they are ignored.

Pattern: Quality gates have boolean flags that enable/disable each check, and commands are derived from your language profile in `config/env.md`.

---

## 5) Quick start

1. **Dry-run a workflow**
   Ask your agent to run any workflow in dry‑run/analyze mode. It will report any path‑resolution 
   problems immediately (e.g., missing files, wrong directories). Fix those first.
   
   Example:
   > `Please execute a dry run of the /continue workflow and report any unresolved paths or missing information`

2. **Bootstrap the project**
   Run `/bootstrap`. It’s idempotent:

   * Initializes ConPort if missing (imports `config/brief.md`)
   * Loads contexts to verify state
   * Sweeps recent Git (configurable window) into ConPort as progress
   * Wraps with pre/post hooks for consistency

3. **Continue development**
   Use `/continue` as your catch‑all:
   * Example: `Please /continue with development of the auth module.`

---

## 6) Notable workflows

* **`bootstrap`**: initialize/refresh ConPort; optional deps install; sweep recent Git; pre/post wrapped.
* **`continue`**: patch Active Context, run **dev loop** (quality only), then postflight.
* **`imprint`**: Imprints on your programming style, patterns, and idiosyncrasies. (hint: Use this after a session of refactoring the agents code.)
* **`debug`**: create an environment‑native debug script/executable and run it (no tests here unless you ask).
* **`document`**: generate/update docs for scope; commit if changed.
* **`tests`**: generate tests for scope; run `pytest`; commit if green.
* **`refactor`**: styleguide‑grounded planning; safe application via quality gate; decision/progress linking.
* **`plan`**: reconcile `plan.md` with recent progress, draft next steps.
* **`report`**: ConPort + Git snapshot report; optional file export.

> All top‑level workflows follow the same structure: **preflight → body → postflight**.

---

## 7) Safety & idempotence

* **Atoms** are single‑purpose and idempotent.
* **Hooks** centralize environment/policy/context handling.
* **Dev loop** encapsulates the core development loop (i.e. handles your prompted request)
* Many operations support **dry‑run** to preview behavior before changes.

---

## 8) Extending the system

* Add new atoms under `atoms/<mcp_name>/…` (ConPort/Git or others).
* Add language/tool support by enhancing `config/env.md` tool resolution or by passing command overrides.
* Add policies as natural language bullets—no code changes needed.
* Use `config/paths.md` to keep includes portable across repos.

---

## 9) FAQ

**Q: Where do I configure black/ruff/mypy/pytest?**
A: In `config/env.md` (or pass overrides from the caller). Add a language profile as shown in the examples or configure a new one with the same schema.

**Q: How do I change behavior/tone?**
A: Update `policies/personality.md` (natural language). Also use `always.md` and `never.md` for do/don’t rules.

**Q: Can I run this mid‑project?**
A: Yes. `/bootstrap` is designed to be safe on existing repos.
