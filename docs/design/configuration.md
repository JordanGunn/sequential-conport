# Configuration Guide

This guide covers the one-time configuration steps needed to set up the Context-Persistent MCP Framework in your repository.

## Important Layout Note

**Do not rename or move files/directories** unless you also update their references in configs. This framework relies on the existing paths under `.windsurf/workflows/`.

## Configuration Files Overview

The framework uses three main configuration areas:

1. **Environment Configuration** (`config/env.md`) - Language profiles, tools, paths
2. **Project Context** (`config/brief.md`) - High-level project information
3. **Behavioral Policies** (`policies/`) - Natural language behavior guidelines

## 1. Environment Configuration

### Location: `.windsurf/workflows/config/env.md`

Open `.windsurf/workflows/config/env.md` and configure:

* **Language profile** (toolchain, format/lint/type/test commands, etc.)
* **Executable paths** you actually know (e.g., poetry, pytest, node)
* **Project paths** (root, tests, src, etc.)

**See templates & guidance:** `examples/env/README.md`

Keep `env.md` factual and minimal; you can override commands per run later.

### Key Principles

- Use absolute paths where possible
- Test tool availability before listing them
- Focus on tools you actually use in your project
- Document any non-standard toolchain requirements

## 2. Project Context

### Location: `.windsurf/workflows/config/brief.md`

Open `.windsurf/workflows/config/brief.md` and describe:

* What the project is and its end goals
* Constraints & assumptions the agent should honor
* Any "must-know" system facts (domains, data sources, deployment notes)

This file provides essential context to AI agents about your project's purpose and constraints.

## 3. Behavioral Policies

### Location: `.windsurf/workflows/policies/`

Edit natural-language policies to guide agent behavior:

* **`always.md`** - Things the agent should always do
* **`never.md`** - Things the agent must never do  
* **`personality.md`** - Tone and role guidelines

If you add a new policy file, register it in `policies/.index.md`.

> **No code changes required** — these files are plain English guidance that gets loaded by workflow hooks.

### Policy Integration

The policies are loaded by the pre-hook system and influence agent behavior throughout all workflows. They provide:

- Consistent behavior across different workflows
- Project-specific guardrails and preferences  
- Natural language configuration that's easy to maintain

## MCP Server Configuration

### Using uvx for Python Servers

The framework leverages `uvx` for running Python-based MCP servers. Update your MCP client configuration to use paths like:

```json
{
  "mcpServers": {
    "conport": {
      "command": "/path/to/uvx",
      "args": [
        "--from",
        "context-portal-mcp",
        "conport-mcp",
        "--mode",
        "stdio",
        "--workspace_id",
        "/absolute/path/to/project/root",
        "--log-file",
        "/path/to/conport.log",
        "--log-level",
        "INFO"
      ]
    }
  }
}
```

### FastMCP Considerations

When developing with fastMCP, ensure your server configuration aligns with the uvx execution model. This provides better integration with the MCP ecosystem while maintaining development simplicity.

## Directory Structure After Configuration

```
.windsurf/
  workflows/
    config/
      env.md          # ✓ Configured language profile, paths, tools
      brief.md        # ✓ Configured project context
    policies/
      .index.md       # ✓ Register active policy files here
      always.md       # ✓ Configured behavior guidelines
      never.md        # ✓ Configured restrictions
      personality.md  # ✓ Configured tone/role
    # ... rest of framework structure
```

## Validation

After configuration, validate your setup by running a dry run:

```
Please execute a dry run of the /bootstrap workflow and report unresolved paths or missing info.
```

Fix any issues flagged by the dry run before proceeding to actual workflow usage.

## Next Steps

Once configuration is complete, proceed to:
- [First Run Guide](first-run.md) - Initial bootstrap and validation
- [Workflow Usage](workflows.md) - Daily usage patterns
- [Architecture Overview](architecture.md) - Understanding the framework structure