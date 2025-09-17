# Context-Persistent MCP Framework
Turns your repo into a self-documenting, idempotent workbench for AI agents (Windsurf/Cascade, Copilot, etc.).

---

## 0) Important Layout Note
**Do not rename or move files/directories** unless you also update their references in configs. This framework relies on the existing paths under `.windsurf/workflows/`.

---

## 1) Prerequisites

**Install `uvx`**
```bash
pip install uv                # installs `uv` & `uvx`
uvx --version                 # verify
````

**Install `npx`**
```bash
node -v && npm -v             # ensure Node.js & npm installed
npx --version                 # verify `npx`
```

### MCP config

Copy the provided [MCP config](./mcp_config.json) into your agent’s MCP settings:

---

## 2) Configure Your Repo (one-time)

### 2.1 `config/env.md` — language profile & environment facts

Open `.windsurf/workflows/config/env.md` and set:

* **Language profile** (toolchain, format/lint/type/test commands, etc.)
* **Executable paths** you actually know (e.g., poetry, pytest, node)
* **Project paths** (root, tests, src, etc.)

**See templates & guidance:** `examples/env/README.md`

Keep `env.md` factual and minimal; you can override commands per run later.

### 2.2 `config/brief.md` — project context

Open `.windsurf/workflows/config/brief.md` and describe:

* What the project is and its end goals
* Constraints & assumptions the agent should honor
* Any “must-know” system facts (domains, data sources, deployment notes)

### 2.3 `policies/` — behavior in plain English

Edit natural-language policies in `.windsurf/workflows/policies/`:

* `always.md` (do this), `never.md` (don’t do this), `personality.md` (tone/role)
* If you add a new policy file, register it in `policies/.index.md`

> **No code changes required** — these files are plain English guidance.

---

## 3) First Run

### 3.1 Dry run (recommended)

Ask your agent to validate paths and config **without** making changes:

Example:
```
Please execute a dry run of the /bootstrap workflow and report unresolved paths or missing info.
```

Fix anything it flags (missing tools, paths, etc.) in `config/env.md`.

### 3.2 Bootstrap

Trigger the bootstrap workflow:

```
/bootstrap
```

This will:
* Initialize ConPort (imports `config/brief.md` if needed)
* Loads policies & env
* Performs a lightweight Git sweep (configurable) to align context
* Runs with **pre → body → post** hooks for consistency

---

## 4) Daily Use
All workflow names are designed to align with your natural language to encourage their use.

* **Continue development**

  ```
  /continue
  ```

  Use with a prompt, e.g.:

  ```
  Please /continue by refactoring the API auth module for clearer boundaries.
  ```

* **Other common workflows**

  * `/refactor`, `/tests`, `/document`, `/plan`, `/report`

* Note: You can use multiple workflows in a single prompt:
  ```
  Create a /plan for unit /tests and /report the results.
  ```

All top-level workflows follow the same pattern:
**[HOOK:PRE] → [DEV:LOOP(QUALITY):<prompt>] → [HOOK:POST]**.

---

## 5) Using with Copilot / Other Agents

Most agents support command-style triggers or file-driven instructions.

* **Windsurf / Cascade**: Use slash commands matching workflow filenames:

  ```
  /bootstrap
  /continue
  ```

  (Workflows live in `.windsurf/workflows/` — filenames map to the slash names.)

* **Other Agents**:
  * Place the contents of the workflow directory in whatever directory your agent designates for custom instructions and trigger the workflow using the agent's syntax.

* **Any agent**: Keep the folder structure intact and ensure it can read from `.windsurf/workflows/`.

---

## 6) Directory Overview (short)

```
.windsurf/
  workflows/
    config/
      env.md          # language profile, paths, tools (see examples/env/README.md)
      brief.md        # high-level project context
    policies/
      .index.md       # register active policy files here
      always.md
      never.md
      personality.md
    hooks/
      pre.md          # loads env+policies; light ConPort/Git snapshot; emits preflight
      post.md         # optional Git capture; ConPort sync/export/log; emits postflight
    atoms/
      conport/        # initialize, load, search, relate, log, sync, export…
      git/            # status, diff, stage, commit, sweep…
    dev/
      loop.md         # task+scope → apply quality gates → summary
      quality.md      # format/lint/types/tests (booleans; commands from env)
    # top-level workflows live here (continue, bootstrap, refactor, tests, document, plan, report, …)
```

---

## 7) Safety & Idempotence
* Atoms are single-purpose and idempotent.
* All workflows are built from atoms to ensure consistent behaviour.
* Hooks centralize environment/policy/context handling, and ensure context is fetched and updated before and after every workflow.
* Quality gates are toggleable by setting the boolean flags in dev/quality.md; skipped if tools aren’t present.
* Prefer **dry runs** first to surface path/config issues safely.
