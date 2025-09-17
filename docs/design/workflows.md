# Workflow Usage Guide

This guide covers how to use the Context-Persistent MCP Framework workflows for daily development tasks.

## Prerequisites

Before using workflows, ensure you have completed:
- [Installation](installation.md) - Tools and MCP servers set up
- [Configuration](configuration.md) - Repository configured  
- [First Run](first-run.md) - Framework bootstrapped and validated

## Daily Workflow Usage

All workflow names are designed to align with natural language to encourage their use.

### Core Development Workflows

**Continue Development**
```
/continue
```

Use with a prompt for specific tasks:
```
Please /continue by refactoring the API auth module for clearer boundaries.
```

**Other Common Workflows**
* `/refactor` - Safe refactoring operations
* `/tests` - Test development and execution  
* `/document` - Generate or update documentation
* `/plan` - Project planning and task breakdown
* `/report` - Status reporting and summaries

### Workflow Combinations

You can use multiple workflows in a single prompt:
```
Create a /plan for unit /tests and /report the results.
```

### Workflow Pattern

All top-level workflows follow the same pattern:
**[HOOK:PRE] → [DEV:LOOP(QUALITY)] → [HOOK:POST]**

1. **Pre-hook**: Loads environment, policies, and context
2. **Main workflow**: Executes the specific task with quality gates
3. **Post-hook**: Updates context, logs activity, exports changes

## Agent-Specific Usage

### Windsurf / Cascade

Use slash commands matching workflow filenames:

```
/bootstrap
/continue
/refactor
```

Workflows live in `.windsurf/workflows/` — filenames map to the slash command names.

### GitHub Copilot / Other Agents

For agents that don't support slash commands:

1. Place the contents of the workflow directory in your agent's custom instructions directory
2. Trigger workflows using your agent's syntax
3. Ensure the agent can read from `.windsurf/workflows/`

### Universal Compatibility

The framework maintains folder structure integrity across different agents by:
- Using consistent file paths and references
- Providing clear workflow entry points
- Maintaining agent-agnostic configuration files

## Quality Gates

All workflows include configurable quality gates:

- **Format**: Code formatting (toggleable)
- **Lint**: Code linting and style checks (toggleable)  
- **Type Check**: Static type analysis (toggleable)
- **Tests**: Unit/integration test execution (toggleable)
- **Coverage**: Code coverage thresholds (configurable)

Quality gates are controlled by settings in `dev/quality.md` and skip automatically if tools aren't present.

## Workflow Hooks

### Pre-Hook (`hooks/pre.md`)
- Loads environment configuration
- Applies behavioral policies
- Takes lightweight ConPort/Git snapshot
- Emits preflight validation

### Post-Hook (`hooks/post.md`)  
- Captures Git changes (optional)
- Syncs ConPort context
- Exports/logs run records
- Emits postflight summary

## Safety and Best Practices

### Idempotent Design
- Atoms are single-purpose and idempotent
- All workflows are built from atoms for consistent behavior
- Hooks centralize environment/policy/context handling

### Safe Execution
- **Always start with dry runs** to surface configuration issues
- Quality gates prevent broken code from being committed
- Context is fetched and updated before/after every workflow
- All operations are logged for audit trails

### Development Loop

1. **Plan**: Use `/plan` to break down complex tasks
2. **Execute**: Use `/continue` or specific workflows for implementation
3. **Validate**: Quality gates automatically run format/lint/test checks
4. **Document**: Use `/document` to update documentation as needed
5. **Report**: Use `/report` to summarize progress and status

## Troubleshooting

### Common Issues

**Missing Tools**: Update `config/env.md` with correct tool paths
**Path Errors**: Ensure all paths in configuration are absolute and accessible
**Quality Gate Failures**: Check tool availability and configuration in `dev/quality.md`
**Hook Failures**: Verify ConPort MCP server is running and accessible

### Debug Workflow

Use the debug workflow for investigating issues:
```
/debug
```

This provides detailed diagnostic information about configuration, tools, and system state.

## Advanced Usage

### Custom Workflows

Create custom workflows by:
1. Adding new `.md` files to `.windsurf/workflows/`
2. Following the pre→body→post hook pattern
3. Using existing atoms for consistent behavior
4. Updating workflow indexes as needed

### Context Management

The framework maintains project context through ConPort:
- **Product Context**: Overall project goals and architecture
- **Active Context**: Current focus, recent changes, open issues
- **Historical Context**: Audit trail of all workflow executions

This context is available to all workflows and helps maintain consistency across development sessions.

## Next Steps

- [Architecture Overview](architecture.md) - Understand the framework's design
- [Configuration Guide](configuration.md) - Modify settings and preferences
- [Installation Guide](installation.md) - Set up additional tools or environments