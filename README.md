# Sequential ConPort Workflow Architecture

This repository demonstrates a workflow architecture designed for AI agents (specifically Cascade via 
Windsurf) that combines **Sequential Thinking**, **ConPort knowledge management**, and **Git MCP integration**. 

The setup can be adapted for other AI agent tools through whatever custom instructions medium is provided by the particular vendor.

---

## 1.0 ARCHITECTURE OVERVIEW

---

> **Note that if you just want the setup instructions, you can skip to `2.0 Setup Instructions` below.**

The workflow system is built on a **compositional architecture** with two distinct layers:

### 1. **Atomic Building Blocks** (`workflows/conport/`)
The `workflows/conport/` directory contains **atomic, reusable workflow verbs** that perform single, well-defined 
operations. These building blocks are designed to be:
- **Atomic**: Each workflow performs one specific action
- **Composable**: Can be combined to create larger workflows  
- **Idempotent**: Safe to run multiple times
- **Reusable**: Referenced by multiple higher-level workflows

### 2. **Idempotent Workflows** (`workflows/`)
The main `workflows/` directory contains **complete, production-ready workflows** that compose the atomic building 
blocks using `action: include` references. These workflows provide:
- **End-to-end functionality** for complex development tasks
- **Language-agnostic design** supporting Python, JavaScript, Go, Rust, and more
- **Consistent patterns** across all workflow implementations
- **Maintainable composition** through building block reuse

### 3. **Language-Agnostic Configuration (`workflows/conport/.env.md`)**
All workflows are designed to work across multiple programming languages through centralized tool configuration:
- **Single Source of Truth**: `.env.md` defines all language-specific tools and paths
- **Generic Tool Categories**: Workflows reference abstract tools (formatter, linter, test_runner)
- **Example Configurations**: Pre-built configurations for popular languages in `examples/`
- **Easy Customization**: Copy and modify example configurations for your specific setup

## Key Reference Files

Two critical files support this architecture:

- **[`.index.md`](workflows/conport/.index.md)**: Complete index of all building block files with relative paths
- **[`.tools.md`](workflows/conport/.tools.md)**: Reference documentation for all ConPort MCP tools and their parameters

## Building Block Documentation

For detailed descriptions of each atomic building block workflow, see:

**[ConPort Building Blocks Documentation →](workflows/conport/README.md)**

## Workflow Composition Pattern

Main workflows reference building blocks using this pattern:

```yaml
steps:
  - id: env
    action: include
    file: .windsurf/workflows/conport/.env.md
  
  - id: load
    action: include
    file: .windsurf/workflows/conport/load.md
    
  - id: update_context
    action: include
    file: .windsurf/workflows/conport/update.md
    with:
      active_patch:
        current_focus: "{{ inputs.task }}"
```

This architecture ensures:
- **Consistency** across all workflows
- **Maintainability** through centralized building blocks
- **Extensibility** for new workflow creation
- **Reliability** through proven atomic operations

---

## 2.0 SETUP INSTRUCTIONS

---

### Part 1: Prerequisites

Before setting up the workflow architecture, ensure you have the required tools installed:

#### 1. Install NPX (Node Package Executor)
```bash
# Check if npx is installed
npx --version

# If not installed, install Node.js (includes npx)
# On Ubuntu/Debian:
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs

# On macOS with Homebrew:
brew install node

# On Windows:
# Download and install from https://nodejs.org/
```

#### 2. Install UVX (Python Package Runner)
```bash
# Check if uvx is installed
uvx --version

# If not installed, install via pip:
pip install uvx

# Or via pipx (recommended):
pipx install uvx
```

### MCP Configuration Setup

The setup described in the section below leverages three MCPs: 
- `sequential-thinking`
- `context-portal` (aka `conport`)
- `git`

#### MCP Repositories/Documentation

For convenience, links to their repositories/documentation are provided here:
- [sequential-thinking](https://github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking)
- [git](https://github.com/modelcontextprotocol/servers/tree/main/src/git)
- [context-portal](https://github.com/GreatScottyMac/context-portal)

#### 1. Configure MCP Servers
Copy the configuration from [mcp_config.json](mcp_config.json) to your AI agent's settings:

- **For Windsurf**: Add to your Windsurf MCP settings
- **For other agents**: Adapt the configuration format as needed

#### 2. Update Binary Paths
**Important**: To avoid binary detection issues, use absolute paths in the MCP configuration:

```bash
# Find absolute paths for your binaries:
which npx     # Usually /usr/bin/npx or /usr/local/bin/npx
which uvx     # Usually ~/.local/bin/uvx or similar
```

Replace the placeholders in [mcp_config.json](mcp_config.json):
- `<absolute_path>/npx` → Your actual npx path (e.g., `/usr/bin/npx`)
- `<absolute_path>/uvx` → Your actual uvx path (e.g., `/home/username/.local/bin/uvx`)

#### 3. Update Project Paths
Replace the project-specific placeholders:
- `<Absolute path to project root>` → Your project's root directory
- `<Destination of conport log file>` → Where you want ConPort logs (e.g., `{project_root}/.windsurf/conport.log`)

### Part 2: Initial Workflow Setup
Some intital configurations are required to get started with the setup. The files explicitly requiring changes are:
- `workflows/conport/.env.md`
- `workflows/conport/.brief.md`

#### 1. Customize Project Brief
Edit [`workflows/conport/.brief.md`](workflows/conport/.brief.md):
- Replace the example content with a high-level summary of **your** project
- Include your project's goals, architecture, and key components
- This file serves as the canonical project description for ConPort initialization

> Note that I have left an example of the one I was using in the file for reference (hopefully this doesn't get me in trouble).
> Your `.brief.md` does not necessarily need to be this detailed for this setup to work, but it's a good reference point.


#### 2. Configure Environment for Your Language

The workflows are **language-agnostic** and work with any programming language. Choose the appropriate setup:

##### Option A: Use Pre-built Language Examples
Copy one of the example configurations to get started quickly:

```bash
# For Python projects
cp examples/env/python.md workflows/conport/.env.md

# For JavaScript/Node.js projects  
cp examples/env/javascript.md workflows/conport/.env.md

# For Go projects
cp examples/env/go.md workflows/conport/.env.md

# For Rust projects
cp examples/env/rust.md workflows/conport/.env.md
```

Then customize the paths and tool configurations for your specific project.

##### Option B: Manual Configuration
Edit [`workflows/conport/.env.md`](workflows/conport/.env.md) to define your language-specific tools:

```yaml
# Project Structure
project:
  root_path: "/path/to/your/project"
  src_path: "/path/to/your/project/src"
  tests_path: "/path/to/your/project/tests"

# Language/Framework Detection  
language: python  # or javascript, go, rust, etc.
framework: poetry  # or npm, cargo, go, etc.

# Generic Tool Categories
tools:
  package_manager: poetry  # or npm, cargo, go, etc.
  test_runner: pytest     # or jest, go, cargo, etc.
  formatter: black        # or prettier, gofmt, rustfmt, etc.
  linter: ruff           # or eslint, golangci-lint, clippy, etc.
  type_checker: mypy     # or tsc, go, cargo, etc.

# Executable paths
executables: 
  poetry: /home/username/.local/bin/poetry
  git: /usr/bin/git
  # Add paths for your language's tools
```

##### Language Examples Available:
- **[Python + Poetry](examples/env/env-python.md)** - pytest, black, ruff, mypy
- **[JavaScript + npm](examples/env/env-javascript.md)** - jest, prettier, eslint, tsc  
- **[Go + modules](examples/env/env-go.md)** - go test, gofmt, golangci-lint
- **[Rust + Cargo](examples/env/env-rust.md)** - cargo test, rustfmt, clippy

See **[Language Examples Documentation](examples/README.md)** for detailed setup instructions.

> **Why this matters**: The language-agnostic design allows the same workflows to work across different programming languages by simply changing the tool configuration.

#### 3. Verify Configuration
Double-check all workflow files in `workflows/conport/` for any remaining hardcoded paths or project-specific references that need updating.

#### 4. Run `/bootstrap`
Once you have completed these initial steps, run the `/bootstrap` workflow to initialize the setup within your project.

That's it! You're ready to go. See the following section (`3.0 WORKFLOWS`) for documentation on the various workflows
and their use cases.

> If you encounter issues here (e.g. workflows not being detected in chat instance), scroll to the 
> bottom of the file for `TROUBLESHOOTING` information.

---

## 3.0 WORKFLOWS

---

The workflows are designed as **single-word verbs** that align with natural language to encourage their use within 
Windsurf's chat interface. Simply type `/<workflow-name>` to invoke any workflow.

The behaviours of all workflows have been designed to leverage the following philosophies:
- **Idempotency**: Repeated execution is safe and won't cause unintended side effects.
- **Safety**: Destructive workflows execute in **dry-run mode by default** and request explicit permission before making changes that could affect your codebase.

### 🚀 **Core Workflows**

#### `/bootstrap`
**Purpose**: Idempotently initializes ConPort for your project, whether at the beginning or midway through development.

**Key Features**:
- Detects existing ConPort installations or creates new ones
- Imports project brief into Product Context
- Seeds Active Context with current project state  
- Sweeps recent git history to capture existing progress
- **Safe to run multiple times** - won't duplicate or overwrite existing data

**Usage in Windsurf**: `/bootstrap` when starting with this workflow system or when joining an existing project that needs ConPort initialization.

#### `/continue` 
**Purpose**: The **catch-all workflow** that provides a safe, context-persistent way to proceed with any development task.

**Key Features**:
- Patches Active Context with current focus
- Respects scope and recursive directory processing
- Runs quality gates (formatting, linting, type checking)
- Auto-commits changes with conventional commit messages
- Maintains full ConPort context throughout the process

**Usage in Windsurf**: 
- `/continue task="implement user authentication" scope="src/auth/"` (your go-to workflow for most development tasks.)
- `/continue with developing the database schema` (Using natural language.)

### 🔧 **Development Workflows**

#### `/debug`
**Purpose**: Reproduce test failures, iterate on fixes, and re-run tests until resolution.

**Key Features**:
- Runs testing tools configured in `conport/.env.md` on specified scope
- Iterates through fix attempts with configurable retry limit
- Logs debugging progress and results to ConPort
- Maintains context of debugging decisions

**Usage in Windsurf**: 
- `/debug task="fix authentication tests" scope="tests/auth/"` when you need systematic debugging.
- `/debug by fixing authentication tests` (Using natural language.)

#### `/refactor`
**Purpose**: Plan and execute safe refactoring with decision logging and quality enforcement.

**Key Features**:
- Logs refactoring decisions with rationale
- Applies edits with strict quality gates
- Links refactoring results to original decisions
- **Dry-run by default** for safety

**Usage in Windsurf**: 
- `/refactor task="extract user service from controller"` (for planned architectural changes.)
- `/refactor by extracting the user service from controller"` (Using natural language.)

### 📝 **Documentation & Quality**

#### `/document`
**Purpose**: Generate or update documentation for files in scope and commit changes.

**Key Features**:
- Analyzes code structure and generates appropriate documentation
- Updates existing docs to reflect code changes
- Commits documentation updates automatically
- Integrates with ConPort for documentation decisions

**Usage in Windsurf**: 
- `/document task="update API docs" scope="src/api/"` to maintain current documentation.
- `/document by updating API docs` (Using natural language.)

#### `/test`
**Purpose**: Generate tests for specified scope, run pytest, and commit if tests pass.

**Key Features**:
- Generates comprehensive test coverage
- Runs full test suite to ensure no regressions
- Commits new tests only if they pass
- Logs test coverage decisions to ConPort

**Usage in Windsurf**: 
- `/tests task="add tests for user service" scope="src/services/user.py"` for test-driven development.
- `Develop /tests for the user service` (Using natural language.)

#### **`/lint`
**Purpose**: Lint and auto-correct using configured `workflows/conport/.env.md` tools with safety controls.

**Key Features**:
- Auto-fixes formatting and import organization
- Plans and confirms risky changes before applying
- Uses project-specific linting configuration
- **Safe mode** - asks permission for potentially breaking changes

**Usage in Windsurf**: 
- `/lint targets=["src/", "tests/"]` to maintain code quality standards.
- `/lint the auth module` (Using natural language.)

### 🔍 **Analysis & Maintenance**

#### `/trace`
**Purpose**: Generate import and dependency traces for a scope, save artifacts, and log to ConPort for RAG.

**Key Features**:
- Maps import dependencies and relationships
- Saves trace artifacts for future reference
- Logs dependency insights to ConPort knowledge graph
- Supports both file and directory scoping

**Usage in Windsurf**: 
- `/trace scope="src/auth/"` to understand component dependencies before refactoring.
- `Please /trace the authentication module` (Using natural language.)

#### `/imprint`
**Purpose**: Identify and capture coding styles, patterns, and conventions from your codebase for consistent development.

**Key Features**:
- Analyzes repository-wide coding patterns and style conventions
- Extracts architectural patterns and design decisions
- Logs style imprint as custom data in ConPort for future reference
- Helps maintain consistency across team development

**Usage in Windsurf**: 
- `/imprint` to capture current codebase style patterns for new team members or AI context.
- `Please /imprint the current coding style` (Using natural language.)

#### `/sweep`
**Purpose**: Sweep recent git activity and integrate changes into ConPort context.

**Key Features**:
- Analyzes git status, log, and diffs
- Creates ConPort progress entries from git history
- Updates Active Context with recent changes
- Configurable time window for activity analysis

**Usage in Windsurf**: 
- `/sweep since="3.days"` to catch up ConPort with recent development activity.
- `Please /sweep the last three days` (Using natural language.)

#### `/plan`
**Purpose**: Maintain project planning documents, close completed items, and generate next steps.

**Key Features**:
- Updates `plan.md` with current project status
- Marks completed items as done
- Generates next steps based on current context
- Logs planning decisions to ConPort

**Usage in Windsurf**:
- `Create a /plan for implementation of user authentication` (Using natural language.)

### 🚢 **Deployment & Finalization**

#### `/finalize`
**Purpose**: Apply maximum quality strictness, commit, push, and log completion in ConPort.

**Key Features**:
- **Maximum strictness** quality gates
- Comprehensive testing and validation
- Auto-commit and push to remote
- Logs completion milestones in ConPort

**Usage in Windsurf**: 
- `/finalize task="prepare v1.2.0 release"`
- `Please /finalize the release` (Using natural language.)

### 🔄 **Workflow Characteristics**

#### **Idempotent Design**
All workflows can be run multiple times safely:
- **State checking**: Workflows detect existing state and avoid duplication
- **Incremental updates**: Only apply changes that haven't been made
- **Safe defaults**: Conservative settings prevent accidental damage

#### **Dry-Run Safety**
Potentially destructive workflows include dry-run protection:
- **Preview mode**: Shows planned actions without executing
- **User confirmation**: Requests explicit permission for risky operations
- **Rollback capability**: Maintains state to undo changes if needed

#### **Natural Language Integration**
Workflow names are designed to feel natural in conversation:
- **Single-word verbs**: `/continue`, `/debug`, `/refactor`, `/finalize`
- **Intuitive usage**: Names match their intended actions
- **Chat-friendly**: Easy to remember and type in Windsurf

### 🛠 **Extensibility & Contributions**

This workflow system is **actively evolving**. Extensions, additions, and improvements are welcomed:

- **Custom workflows**: Add new workflows following the established patterns
- **Enhanced building blocks**: Extend the atomic operations in `workflows/conport/`
- **Integration improvements**: Better tool integration and error handling
- **Documentation updates**: Help improve workflow descriptions and usage examples

The architecture is designed to be **modular and extensible** - new workflows can leverage existing building blocks, 
and new building blocks can be used across multiple workflows.

---

## TROUBLESHOOTING

---

### Binary Path Issues
If your agent can't find binaries:
1. Use `which <command>` to find absolute paths
2. Update MCP configuration with full paths
3. Ensure paths are accessible to your agent's execution environment

### ConPort Database Issues
If ConPort initialization fails:
1. Ensure the workspace directory exists and is writable
2. Check that uvx can install and run `context-portal-mcp`
3. Verify log file destination is writable

### Workflow Reference Errors
If workflows can't find building blocks:
1. Ensure all files in `workflows/conport/` are present
2. Check that relative paths in workflow references are correct
3. Verify `.windsurf/workflows/conport/` path structure matches your setup

### Chat Instance Not Detecting Workflows
If your chat instance can't detect workflows, check the "description" of your workflows 
Windsurf seems to dislike non-alpha-numeric characters in the descriptions. 

For example the following "description": 

> "Continue: Continue with development + get git updates/sweep"

Will likely result in the workflow being undetected by Windsurf as a result of the "+", ":" and "/" characters.
If you experience this, double-check your descriptions, and remove characters like this. An example of a corrected
version of the above would be:

> "Continue development idempotently by patching Active Context, sweeping recent changes, and running quality checks."


