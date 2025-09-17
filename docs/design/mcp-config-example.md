# MCP Configuration Example

This document provides examples of how to configure MCP servers using the uvx-first approach recommended by the Context-Persistent MCP Framework.

## Recommended Configuration

```json
{
  "mcpServers": {
    "sequential-thinking": {
      "command": "/absolute/path/to/npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-sequential-thinking"
      ]
    },
    "git": {
      "command": "/absolute/path/to/uvx",
      "args": [
        "mcp-server-git",
        "--repository",
        "/absolute/path/to/project/root"
      ]
    },
    "conport": {
      "command": "/absolute/path/to/uvx",
      "args": [
        "--from",
        "context-portal-mcp",
        "conport-mcp",
        "--mode",
        "stdio",
        "--workspace_id",
        "/absolute/path/to/project/root",
        "--log-file",
        "/absolute/path/to/conport.log",
        "--log-level",
        "INFO"
      ]
    }
  }
}
```

## Key Configuration Principles

### uvx for Python Servers
- **ConPort**: Uses `uvx --from context-portal-mcp` for clean package installation
- **Git Server**: Uses `uvx mcp-server-git` for repository operations
- **Custom Servers**: Use `uvx --from your-package your-server` pattern

### npx for Node.js Servers
- **Sequential Thinking**: Uses `npx -y @modelcontextprotocol/server-sequential-thinking`
- **Custom Node Servers**: Use `npx your-package` pattern

### FastMCP Integration

When developing with fastMCP, configure servers similarly:

```json
{
  "your-fastmcp-server": {
    "command": "/absolute/path/to/uvx",
    "args": [
      "--from",
      "your-fastmcp-package",
      "your-server-command",
      "--mode",
      "stdio",
      "--your-config-options"
    ]
  }
}
```

## Path Configuration

### Finding Tool Paths

```bash
# Find uvx path
which uvx

# Find npx path  
which npx

# Verify installations
uvx --version
npx --version
```

### Common Paths

**macOS (Homebrew)**:
- uvx: `/opt/homebrew/bin/uvx`
- npx: `/opt/homebrew/bin/npx`

**Linux (pip/apt)**:
- uvx: `/usr/local/bin/uvx` or `~/.local/bin/uvx`
- npx: `/usr/bin/npx` or `/usr/local/bin/npx`

**Windows**:
- uvx: `C:\Users\<username>\AppData\Local\Programs\Python\Python3X\Scripts\uvx.exe`
- npx: `C:\Program Files\nodejs\npx.cmd`

## Benefits of This Configuration

### uvx Advantages
- **Isolated Environments**: Each Python server runs in its own environment
- **Automatic Management**: Handles dependencies and Python versions
- **No Docker Required**: Simpler than containerized approaches
- **Development Friendly**: Easy debugging and development

### Community Alignment
- **Standard Practice**: Follows MCP community conventions
- **Tool Consistency**: Uses recommended tooling patterns
- **Future Compatibility**: Aligns with ecosystem direction

## Alternative: Docker-based Configuration

If you prefer containerized deployment:

```json
{
  "conport": {
    "command": "docker",
    "args": [
      "run",
      "--rm",
      "-i",
      "--network",
      "host",
      "-v",
      "/absolute/path/to/project:/workspace",
      "context-portal-mcp:latest",
      "--workspace_id",
      "/workspace",
      "--mode",
      "stdio"
    ]
  }
}
```

However, uvx is recommended for its simplicity and alignment with community practices.

## Validation

Test your configuration:

```bash
# Test uvx servers
uvx --from context-portal-mcp conport-mcp --help
uvx mcp-server-git --help

# Test npx servers
npx -y @modelcontextprotocol/server-sequential-thinking --help
```

## Troubleshooting

### Common Issues

**Command not found**:
- Verify tool installation: `uvx --version`, `npx --version`
- Use absolute paths in configuration
- Check PATH environment variable

**Permission errors**:
- Ensure tools are executable
- Check file permissions on executables
- Consider using absolute paths

**Package not found**:
- Verify package names are correct
- Test installation manually: `uvx --from package-name command --help`
- Check network connectivity for package downloads

### Debug Steps

1. Test tools individually outside MCP
2. Verify all paths are absolute and accessible
3. Check MCP client logs for detailed error messages
4. Use `uvx --verbose` flag for detailed output during testing