# Architecture and Design

This document provides an overview of the Context-Persistent MCP Framework's architecture, design principles, and internal structure.

## Design Philosophy

The framework is built around several core principles:

1. **Context Persistence**: Maintain project context across development sessions
2. **Idempotent Operations**: All operations can be safely run multiple times
3. **Agent Agnostic**: Work with different AI agents (Windsurf, Copilot, etc.)
4. **Natural Language First**: Configuration and policies in plain English
5. **Composable Workflows**: Build complex operations from simple, reusable atoms

## Architecture Overview

```
Context-Persistent MCP Framework
├── MCP Integration Layer (uvx/npx based servers)
├── Workflow Orchestration (.windsurf/workflows/)
├── Context Management (ConPort)
├── Quality Assurance (Configurable gates)
└── Agent Interface (Slash commands, file-based)
```

## Directory Structure

```
.windsurf/
  workflows/
    config/                 # Environment and project configuration
      env.md               # Language profile, paths, tools
      brief.md             # High-level project context
      paths.md             # Path definitions and mappings
    policies/               # Behavioral guidelines (natural language)
      .index.md            # Registry of active policy files
      always.md            # Things to always do
      never.md             # Things to never do
      personality.md       # Tone and role guidance
    hooks/                  # Workflow lifecycle management
      pre.md               # Pre-execution setup and validation
      post.md              # Post-execution cleanup and logging
    atoms/                  # Atomic, reusable operations
      conport/             # Context management operations
        initialize.md      # ConPort initialization
        load.md           # Context loading
        search.md         # Context search
        relate.md         # Relationship management
        log.md            # Activity logging
        sync.md           # Context synchronization
        export.md         # Context export
        tools.md          # Available ConPort tools
      git/                 # Git operations
        status.md         # Git status checking
        diff.md           # Git diff operations
        stage.md          # Git staging
        commit.md         # Git commit operations
        sweep.md          # Git repository sweeping
    dev/                   # Development workflow components
      loop.md             # Main development loop
      quality.md          # Quality gate definitions
    # Top-level workflows
    bootstrap.md           # Initial setup workflow
    continue.md            # Main development workflow  
    refactor.md            # Safe refactoring workflow
    tests.md               # Test development and execution
    document.md            # Documentation generation
    plan.md                # Project planning
    report.md              # Status reporting
    debug.md               # Debugging and diagnostics
```

## Component Interactions

### MCP Integration

The framework integrates with Model Context Protocol (MCP) servers through:

**uvx-based Python Servers** (Recommended):
- ConPort for context management
- Git server for repository operations
- Custom fastMCP-based servers

**npx-based Node.js Servers**:
- Sequential thinking server
- Other Node.js MCP implementations

### Workflow Execution Flow

1. **Trigger**: Agent receives workflow command (e.g., `/continue`)
2. **Pre-Hook**: 
   - Load environment configuration
   - Apply behavioral policies
   - Capture current context snapshot
   - Validate prerequisites
3. **Main Workflow**:
   - Execute workflow-specific logic
   - Use atomic operations for consistency
   - Apply quality gates as configured
4. **Post-Hook**:
   - Update project context
   - Log workflow execution
   - Export changes if needed
   - Generate summary

### Context Management

**ConPort Integration**:
- Maintains project knowledge graph
- Provides semantic search capabilities
- Stores product and active context
- Tracks relationships between project elements
- Enables RAG (Retrieval Augmented Generation)

**Context Types**:
- **Product Context**: Overall project goals, features, architecture
- **Active Context**: Current working focus, recent changes, open issues  
- **Historical Context**: Audit trail of workflow executions
- **Custom Data**: Project-specific metadata and glossaries

### Quality Assurance

**Configurable Quality Gates**:
- Format checking (code formatting)
- Linting (style and correctness)
- Type checking (static analysis)
- Test execution (unit/integration)
- Coverage validation (configurable thresholds)

**Safety Mechanisms**:
- Dry run capabilities for all workflows
- Idempotent operation design  
- Atomic commits with descriptive messages
- Rollback capabilities through Git integration

## Design Patterns

### Atomic Operations

All functionality is built from atomic, single-purpose operations:
- **Composable**: Atoms can be combined into complex workflows
- **Reusable**: Same atoms used across different workflows  
- **Testable**: Each atom can be validated independently
- **Idempotent**: Safe to run multiple times

### Hook Pattern

All workflows follow the pre→body→post hook pattern:
- **Consistency**: Same setup/teardown for all workflows
- **Context Management**: Centralized context handling
- **Policy Enforcement**: Behavioral policies applied uniformly
- **Auditability**: All executions logged and tracked

### Configuration-Driven Behavior

- **Environment**: Tool paths, language profiles, project structure
- **Policies**: Natural language behavioral guidelines
- **Quality Gates**: Configurable validation steps
- **Context**: Project-specific knowledge and constraints

## Integration with Development Tools

### uvx Benefits

Using `uvx` for Python MCP servers provides:
- **Isolation**: Each server runs in its own environment
- **Simplicity**: No Docker complexity or overhead
- **Development-Friendly**: Easy debugging and development
- **Community Standard**: Aligns with MCP ecosystem practices

### fastMCP Support

The architecture supports fastMCP development through:
- Direct uvx execution of fastMCP-based servers
- Consistent configuration patterns
- Development-friendly server management
- Integration with existing MCP tooling

### Agent Compatibility

**Multi-Agent Support**:
- Windsurf/Cascade: Native slash command support
- GitHub Copilot: File-based instruction integration
- Other agents: Flexible configuration directory support

**Universal Patterns**:
- Consistent file structure across agents
- Agent-agnostic workflow definitions
- Portable configuration files
- Standard MCP integration points

## Extensibility

### Adding Custom Workflows

1. Create workflow file in `.windsurf/workflows/`
2. Follow the hook pattern (pre→body→post)
3. Use existing atoms for consistency
4. Update workflow indexes

### Custom Atoms

1. Create atom files in appropriate `atoms/` subdirectory
2. Implement single-purpose, idempotent operations
3. Document inputs, outputs, and side effects
4. Test independently before integration

### Policy Extensions

1. Add policy files to `policies/` directory
2. Register in `policies/.index.md`
3. Use natural language for easy maintenance
4. Define clear behavioral expectations

## Performance Considerations

- **Lazy Loading**: Context loaded only when needed
- **Incremental Updates**: Only changed context synchronized
- **Caching**: ConPort provides efficient context retrieval
- **Parallel Execution**: Quality gates can run concurrently
- **Resource Management**: uvx handles Python environment isolation

## Security and Safety

- **Sandboxed Execution**: uvx provides process isolation
- **Policy Enforcement**: Behavioral policies prevent dangerous operations
- **Audit Trail**: All operations logged through ConPort
- **Rollback Capability**: Git integration enables easy recovery
- **Validation**: Dry runs prevent unintended changes

## Future Extensibility

The architecture supports future enhancements:
- Additional MCP server integrations
- New workflow types and patterns  
- Enhanced context management capabilities
- Integration with more development tools
- Support for additional agent types

This design ensures the framework remains flexible, maintainable, and extensible while providing consistent, reliable functionality across different development environments and AI agents.