# Design Documentation

This directory contains comprehensive design and usage documentation for the Context-Persistent MCP Framework, split into focused, digestible documents for easier navigation and maintenance.

## Documentation Structure

### [Installation Guide](installation.md)
Prerequisites, installation steps, and MCP server configuration. Emphasizes the use of `uvx` as the recommended approach for Python MCP servers and explains the benefits over containerized environments.

**Key Topics:**
- uvx installation and setup
- fastMCP integration patterns
- MCP server configuration
- Alternative approaches (Docker vs uvx)

**Additional Resources:**
- [MCP Configuration Examples](mcp-config-example.md) - Detailed configuration guide

### [Configuration Guide](configuration.md)
One-time setup and configuration of the framework in your repository. Covers environment setup, project context, and behavioral policies.

**Key Topics:**
- Environment configuration (`config/env.md`)
- Project context setup (`config/brief.md`)
- Behavioral policies (`policies/`)
- MCP server configuration for uvx

### [First Run Guide](first-run.md)
Initial setup validation and bootstrap process for new installations.

**Key Topics:**
- Configuration validation and dry runs
- Bootstrap workflow execution
- Common setup issues and solutions
- Installation verification steps

### [Workflow Usage Guide](workflows.md)
Daily usage patterns, workflow commands, and best practices for development with the framework.

**Key Topics:**
- Core development workflows
- Agent-specific usage (Windsurf, Copilot, etc.)
- Quality gates and safety features
- Troubleshooting common issues

### [Architecture and Design](architecture.md)
Comprehensive overview of the framework's architecture, design principles, and internal structure.

**Key Topics:**
- Design philosophy and principles
- Directory structure and component organization
- MCP integration patterns with uvx
- Context management and ConPort integration
- Extensibility and customization options

## Quick Start Path

For new users, follow this recommended reading order:

1. **[Installation Guide](installation.md)** - Set up tools and MCP servers
2. **[Configuration Guide](configuration.md)** - Configure your project
3. **[First Run Guide](first-run.md)** - Validate setup and bootstrap
4. **[Workflow Usage Guide](workflows.md)** - Start using workflows
5. **[Architecture Guide](architecture.md)** - Understand the design (optional)

## Key Framework Benefits

### uvx-First Approach
- **Simplicity**: Direct Python execution without Docker complexity
- **Community Standard**: Aligns with MCP development community practices  
- **Development-Friendly**: Easier debugging and development workflow
- **Performance**: Reduced overhead compared to containerized approaches

### Context Persistence
- Maintains project knowledge across development sessions
- Semantic search capabilities through ConPort integration
- Automatic context synchronization and export

### Agent Compatibility
- Works with Windsurf, GitHub Copilot, and other AI agents
- Consistent behavior across different development environments
- Natural language configuration for easy maintenance

### Quality Assurance
- Configurable quality gates (format, lint, type check, tests)
- Idempotent operations safe to run multiple times
- Comprehensive audit trail and rollback capabilities

## Contributing to Documentation

When updating documentation:

1. Keep documents focused and digestible
2. Update cross-references when moving content
3. Maintain consistency in examples and code snippets
4. Test all configuration examples and code samples
5. Update this index when adding new documents

## Migration from Monolithic README

This documentation structure replaces the previous monolithic README.md approach, providing:

- **Better Navigation**: Topic-focused documents
- **Easier Maintenance**: Smaller, focused files
- **Improved Discoverability**: Clear document purposes
- **Enhanced Readability**: Reduced cognitive load per document

The main README.md now serves as a concise entry point with links to these detailed guides.