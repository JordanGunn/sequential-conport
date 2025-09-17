# Installation Guide

This guide covers the prerequisites and installation steps for the Context-Persistent MCP Framework.

## Prerequisites

### Core Tools

**Install `uvx` (Recommended)**
```bash
pip install uv                # installs `uv` & `uvx`
uvx --version                 # verify
```

`uvx` is the recommended way to run Python-based MCP servers. It provides:
- **Isolated environments**: Each MCP server runs in its own Python environment
- **Automatic dependency management**: No need to manage virtual environments manually
- **Simple deployment**: Direct execution without Docker complexity
- **Community standard**: Widely adopted in the MCP development community

**Install `npx`**
```bash
node -v && npm -v             # ensure Node.js & npm installed
npx --version                 # verify `npx`
```

## MCP Configuration

### Using uvx (Recommended)

The provided MCP configuration emphasizes `uvx` for Python-based servers and `npx` for Node.js servers. This approach offers several advantages over containerized environments:

- **Simplicity**: No Docker setup required
- **Performance**: Direct execution without container overhead
- **Development-friendly**: Easier debugging and development
- **Accessible**: Lower barrier to entry for users unfamiliar with containers

### FastMCP Integration

When using fastMCP for development, you can leverage `uvx` to run MCP servers directly:

```bash
# Example of running a fastMCP-based server with uvx
uvx --from your-fastmcp-package your-server --mode stdio
```

This pattern is reflected in our MCP configuration where we use `uvx` for Python-based servers like ConPort.

### Configuration File

Copy the provided [MCP config](../../mcp_config.json) into your agent's MCP settings. For detailed configuration examples and troubleshooting, see the [MCP Configuration Guide](mcp-config-example.md).

The configuration is designed to work with:

1. **Sequential Thinking**: Via `npx` for Node.js-based functionality
2. **Git Integration**: Via `uvx` for Python-based git operations
3. **ConPort**: Via `uvx` for context management

### Alternative: Docker Approach

While uvx is recommended, you can still use containerized environments if preferred. However, uvx provides a simpler alternative that aligns with MCP community standards and offers easier configuration for most users.

## Verification

After installation, verify your setup:

```bash
# Verify core tools
uvx --version
npx --version

# Test MCP server connectivity (adjust paths as needed)
uvx mcp-server-git --help
uvx --from context-portal-mcp conport-mcp --help
```

## Next Steps

Once installation is complete, proceed to the [Configuration Guide](configuration.md) to set up your project-specific settings.