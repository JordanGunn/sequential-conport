# First Run Guide

This guide walks you through the initial setup and validation of the Context-Persistent MCP Framework after installation and configuration.

## Prerequisites

Before proceeding, ensure you have completed:

1. **[Installation](installation.md)** - uvx, npx, and MCP servers installed
2. **[Configuration](configuration.md)** - Environment, context, and policies configured

## Step 1: Validate Configuration

### Dry Run (Highly Recommended)

Before making any changes to your repository, validate your setup with a dry run:

```
Please execute a dry run of the /bootstrap workflow and report unresolved paths or missing info.
```

This command will:
- Check all tool paths in `config/env.md`
- Validate MCP server connectivity
- Verify policy file structure
- Test ConPort initialization without making changes

### Fix Common Issues

The dry run may flag issues like:

**Missing Tools**
- Update `config/env.md` with correct tool paths
- Install missing development tools (linters, formatters, etc.)

**Path Errors**
- Ensure all paths in configuration files are absolute
- Verify directory structure exists

**MCP Connection Issues**
- Check uvx installation and PATH
- Verify MCP server configurations
- Test ConPort server connectivity

## Step 2: Bootstrap the Framework

Once the dry run passes successfully, initialize the framework:

```
/bootstrap
```

### Bootstrap Process

The bootstrap workflow performs the following operations:

1. **ConPort Initialization**
   - Creates or connects to ConPort workspace
   - Imports project context from `config/brief.md`
   - Sets up project knowledge graph

2. **Environment Loading**
   - Loads tool configurations from `config/env.md`
   - Validates executable paths
   - Sets up quality gate configurations

3. **Policy Application**
   - Loads behavioral policies from `policies/`
   - Configures workflow behavior guidelines
   - Sets up safety constraints

4. **Context Snapshot**
   - Performs lightweight Git repository analysis
   - Captures current project state
   - Establishes baseline context

5. **Validation Checks**
   - Verifies all components are working
   - Tests MCP server connections
   - Validates workflow execution environment

### Expected Output

A successful bootstrap should show:
- ✅ ConPort workspace initialized
- ✅ Environment configuration loaded
- ✅ Policies applied successfully
- ✅ Git repository analyzed
- ✅ Framework ready for use

## Step 3: Verify Installation

### Test Core Workflows

After bootstrap, test basic workflow functionality:

```bash
# Test documentation workflow
/document

# Test planning workflow  
/plan

# Test status reporting
/report
```

### Verify Context Management

Check that ConPort is maintaining context:

1. Make a small change to your project
2. Run `/continue` with a simple task
3. Verify the change is tracked in ConPort
4. Check that context persists across sessions

### Test Quality Gates

Verify quality assurance features:

1. Create a file with formatting issues
2. Run a workflow that includes quality gates
3. Confirm formatting/linting is applied
4. Check that quality failures are caught

## Troubleshooting First Run Issues

### ConPort Connection Problems

**Symptoms**: Bootstrap fails with ConPort errors

**Solutions**:
- Verify uvx installation: `uvx --version`
- Test ConPort server: `uvx --from context-portal-mcp conport-mcp --help`
- Check MCP configuration in your agent
- Ensure workspace_id path is absolute and accessible

### Environment Configuration Issues

**Symptoms**: Tool-related errors during workflows

**Solutions**:
- Review `config/env.md` for correct tool paths
- Test tools individually: `poetry --version`, `pytest --version`, etc.
- Update paths to use absolute references where needed
- Remove tools you don't actually use from the configuration

### Policy Loading Failures

**Symptoms**: Warnings about policy files or behavior

**Solutions**:
- Check `policies/.index.md` references existing files
- Ensure policy files contain valid content
- Verify natural language policies are clear and actionable

### Git Integration Issues

**Symptoms**: Git-related workflow failures

**Solutions**:
- Ensure you're in a Git repository
- Verify Git is properly configured
- Check that uvx can run mcp-server-git
- Test Git MCP server connectivity

## Next Steps

Once first run is successful:

1. **Start Development**: Use `/continue` for your first development task
2. **Explore Workflows**: Try different workflows (`/refactor`, `/tests`, `/document`)
3. **Customize Configuration**: Adjust settings based on your experience
4. **Review Architecture**: Read the [Architecture Guide](architecture.md) to understand framework internals

## Best Practices for Ongoing Use

- **Always start with dry runs** when making configuration changes
- **Update context regularly** through normal workflow usage
- **Monitor quality gates** and adjust thresholds as needed
- **Review ConPort exports** periodically to understand captured context
- **Use natural language** in policies and configurations for maintainability

## Getting Help

If you encounter issues not covered here:

1. Check the [Workflow Usage Guide](workflows.md) for common patterns
2. Review the [Configuration Guide](configuration.md) for setup details
3. Consult the [Architecture Guide](architecture.md) for design understanding
4. Use the `/debug` workflow to gather diagnostic information