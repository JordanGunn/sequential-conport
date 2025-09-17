# Context-Persistent MCP Framework

Turns your repository into a self-documenting, idempotent workbench for AI agents (Windsurf/Cascade, Copilot, etc.).

## Quick Start

1. **[Install prerequisites](docs/design/installation.md)** - Set up `uvx`, `npx`, and MCP servers
2. **[Configure your repository](docs/design/configuration.md)** - One-time setup of environment, context, and policies
3. **[Complete first run](docs/design/first-run.md)** - Validate setup and bootstrap the framework
4. **[Start using workflows](docs/design/workflows.md)** - Begin development with the framework

## Key Features

- **uvx-First Approach**: Simple Python MCP server deployment without Docker complexity
- **fastMCP Integration**: Seamless support for fastMCP-based development
- **Context Persistence**: Maintains project knowledge across development sessions
- **Agent Agnostic**: Works with Windsurf, GitHub Copilot, and other AI development tools
- **Quality Assurance**: Configurable gates for formatting, linting, type checking, and testing
- **Natural Language Configuration**: Policies and behavior defined in plain English

## Documentation

### 📚 [Complete Design Documentation](docs/design/)

- **[Installation Guide](docs/design/installation.md)** - Prerequisites, uvx setup, and MCP configuration
- **[Configuration Guide](docs/design/configuration.md)** - Environment setup and project configuration
- **[First Run Guide](docs/design/first-run.md)** - Setup validation and framework bootstrap
- **[Workflow Usage](docs/design/workflows.md)** - Daily development patterns and commands
- **[Architecture Overview](docs/design/architecture.md)** - Framework design and extensibility

## Core Workflow Commands

```bash
/bootstrap    # Initialize the framework
/continue     # Main development workflow  
/refactor     # Safe refactoring operations
/tests        # Test development and execution
/document     # Generate/update documentation
/plan         # Project planning and breakdown
/report       # Status reporting and summaries
```

## uvx Integration Highlights

This framework emphasizes `uvx` for Python MCP servers, offering:

- **Community Standard**: Aligns with MCP development community practices
- **Development Simplicity**: Direct execution without containerization overhead  
- **Isolated Environments**: Each MCP server runs in its own Python environment
- **fastMCP Compatibility**: Seamless integration with fastMCP-based servers

## Framework Architecture

```
.windsurf/workflows/
├── config/          # Environment and project configuration
├── policies/        # Natural language behavioral guidelines  
├── hooks/           # Pre/post workflow lifecycle management
├── atoms/           # Atomic, reusable operations (conport/, git/)
├── dev/             # Development loop and quality gates
└── *.md             # Top-level workflows (bootstrap, continue, etc.)
```

## Safety & Best Practices

- **Idempotent Design**: All operations safe to run multiple times
- **Dry Run First**: Validate configurations before making changes
- **Quality Gates**: Automated formatting, linting, and testing
- **Context Awareness**: Full audit trail and project knowledge persistence
- **Policy-Driven**: Natural language behavior guidelines

---

**Important**: Do not rename or move framework files unless you also update their references in configurations. The framework relies on consistent paths under `.windsurf/workflows/`.