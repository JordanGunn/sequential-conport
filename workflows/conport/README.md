# ConPort Building Block Workflows

This directory contains **atomic, reusable workflow building blocks** that perform single, well-defined ConPort operations. These workflows are designed to be composed together to create larger, more complex workflows.

## Core Principles

- **Atomic**: Each workflow performs one specific ConPort operation
- **Idempotent**: Safe to run multiple times without side effects
- **Composable**: Can be combined using `action: include` references
- **Reusable**: Used by multiple higher-level workflows

## Building Block Reference

### Environment & Context Management

#### **[`.env.md`](.env.md)**
**Purpose**: Establishes workspace environment and resolves ConPort workspace ID  
**Usage**: Required first step in all workflows to set up proper workspace context  
**Key Function**: Validates workspace paths and sets up environment variables

#### **[`load.md`](load.md)**
**Purpose**: Load existing ConPort context from the workspace and summarize it  
**Usage**: Early workflow step to retrieve current project state  
**Key Function**: Sets status to `[CONPORT_ACTIVE]` or `[CONPORT_INACTIVE]` based on existing context

#### **[`initialize.md`](initialize.md)**
**Purpose**: Bootstrap ConPort for new workspaces or detect existing installations  
**Usage**: First-time setup or workspace initialization  
**Key Function**: Creates ConPort database, imports project brief, establishes initial context

### Context Operations

#### **[`update.md`](update.md)**
**Purpose**: Update Product Context and/or Active Context with new information  
**Usage**: Core workflow step for maintaining current project state  
**Key Function**: Supports both full content replacement and patch-based updates

#### **[`sync.md`](sync.md)**
**Purpose**: Synchronize ConPort with current conversation state  
**Usage**: Ensures new context, decisions, and progress are captured without duplication  
**Key Function**: Idempotent synchronization of conversation state to ConPort

### Data Logging

#### **[`log.md`](log.md)**
**Purpose**: Log new items into ConPort (decisions, progress, patterns, custom data)  
**Usage**: Record important project information and track progress  
**Key Function**: Safe, idempotent logging with duplicate detection

### Information Retrieval

#### **[`search.md`](search.md)**
**Purpose**: Search and retrieve information from ConPort using various methods  
**Usage**: Find existing decisions, patterns, progress entries, or custom data  
**Key Function**: Supports full-text search, semantic search, and filtered retrieval

### Relationship Management

#### **[`relate.md`](relate.md)**
**Purpose**: Create relationships between ConPort items to build the knowledge graph  
**Usage**: Link decisions to progress entries, patterns to implementations, etc.  
**Key Function**: Explicitly builds out the project knowledge graph through item linking

### Data Management

#### **[`export.md`](export.md)**
**Purpose**: Export ConPort data to markdown files for backup or sharing  
**Usage**: Create portable backups or share project knowledge  
**Key Function**: Converts ConPort database to structured markdown files

#### **[`.brief.md`](.brief.md)**
**Purpose**: Contains comprehensive project overview and context  
**Usage**: Referenced during initialization to populate Product Context  
**Key Function**: Serves as the canonical project description and architectural overview

## Reference Files

### **[`.index.md`](.index.md)**
Complete index of all building block files with:
- Relative file paths for easy reference
- Usage hints and best practices
- Idempotency guidelines

### **[`.tools.md`](.tools.md)**
Comprehensive reference for all ConPort MCP tools including:
- Tool names and descriptions
- Required and optional parameters
- Usage examples and patterns

## Composition Patterns

### Standard Workflow Pattern
Most workflows follow this pattern:
```yaml
steps:
  - id: env
    action: include
    file: .windsurf/workflows/conport/.env.md
  
  - id: load
    action: include
    file: .windsurf/workflows/conport/load.md
  
  # ... specific workflow steps ...
  
  - id: update_context
    action: include
    file: .windsurf/workflows/conport/update.md
    with:
      active_patch:
        current_focus: "{{ inputs.task }}"
```

### Development Loop Pattern
Development workflows typically use:
```yaml
steps:
  - id: env
    action: include
    file: .windsurf/workflows/conport/.env.md
  
  - id: load
    action: include
    file: .windsurf/workflows/conport/load.md
  
  - id: pre_patch
    action: include
    file: .windsurf/workflows/conport/update.md
    with:
      active_patch:
        current_focus: "{{ inputs.task }}"
  
  # ... development work ...
  
  - id: log_progress
    action: include
    file: .windsurf/workflows/conport/log.md
    with:
      progress:
        status: "completed"
        description: "{{ inputs.task }}"
```

## Usage Guidelines

1. **Always start with `.env.md`** to establish proper workspace context
2. **Use `load.md` early** to understand current project state
3. **Prefer `patch_content` over `content`** for context updates unless replacing entirely
4. **Use small retrieval sets first** (3-5 items) and expand as needed
5. **Ensure idempotency** - workflows should be safe to run multiple times
6. **Stop if no diffs** - avoid unnecessary operations when state hasn't changed

## Integration with Main Workflows

These building blocks are referenced by main workflows in the parent `workflows/` directory:

- **`/bootstrap`**: Uses `env`, `initialize`, `load`, `sync`
- **`/develop`**: Uses `env`, `load`, `update`, `log`
- **`/debug`**: Uses `env`, `load`, `update`, `log`
- **`/test`**: Uses `env`, `load`, `update`
- **`/document`**: Uses `env`, `load`, `update`

This modular approach ensures consistency across all workflows while maintaining clear separation of concerns and enabling easy maintenance and extension.
