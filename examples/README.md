# Language-Specific Environment Configuration Examples

This directory contains example `env.md` configurations for different programming languages and frameworks. These examples demonstrate how to configure the language-agnostic workflows for specific development environments.

## Available Examples

- **[python.md](env/python.md)** - Python projects using Poetry
- **[javascript.md](env/javascript.md)** - JavaScript/Node.js projects using npm
- **[go.md](env/go.md)** - Go projects using go modules
- **[rust.md](env/rust.md)** - Rust projects using Cargo

## How to Use

1. **Choose the appropriate example** for your project's language and framework
2. **Copy the example** to your project's `.windsurf/workflows/conport/.env.md`
3. **Customize the paths** and tool configurations for your specific project
4. **Update executable paths** to match your system's tool installations

## Configuration Structure

Each example includes:

### Project Structure
- `root_path` - Project root directory
- `src_path` - Source code directory
- `tests_path` - Test files directory
- `docs_path` - Documentation directory

### Language Detection
- `language` - Programming language identifier
- `framework` - Package manager/framework identifier

### Generic Tool Categories
- `package_manager` - Dependency management tool
- `test_runner` - Test execution tool
- `formatter` - Code formatting tool
- `linter` - Code analysis tool
- `type_checker` - Static type checking tool
- `import_organizer` - Import statement organizer
- `dependency_analyzer` - Dependency analysis tool
- `build_tool` - Build system tool

### Tool-Specific Configurations
- `test` - Test runner arguments and coverage settings
- `lint` - Linter arguments for safe fixes and reporting
- `format` - Formatter arguments and options
- `type_check` - Type checker arguments

### Legacy Compatibility
- `executables` - Absolute paths to tool binaries

## Creating Custom Configurations

To create a configuration for a language not included in these examples:

1. **Start with the closest example** (e.g., use Python example for other interpreted languages)
2. **Update the language and framework** identifiers
3. **Replace tool names** with your language's equivalents
4. **Adjust tool arguments** to match your tools' command-line interfaces
5. **Update executable paths** to match your system's installations

## Tool Mapping Examples

| Language | Package Manager | Test Runner | Formatter | Linter | Type Checker |
|----------|----------------|-------------|-----------|--------|--------------|
| Python | poetry/pip | pytest | black | ruff/flake8 | mypy |
| JavaScript | npm/yarn | jest/mocha | prettier | eslint | tsc |
| Go | go | go test | gofmt | golangci-lint | go build |
| Rust | cargo | cargo test | rustfmt | clippy | cargo check |
| Java | maven/gradle | junit | google-java-format | checkstyle | javac |
| C# | dotnet | dotnet test | dotnet format | dotnet analyze | dotnet build |

## Contributing

To contribute additional language examples:

1. Create a new `env/{language}.md` file following the established pattern
2. Include comprehensive tool configurations and realistic examples
3. Test the configuration with actual workflows
4. Update this README with the new example

## Troubleshooting

**Common Issues:**

- **Tool not found**: Verify executable paths in the `executables` section
- **Wrong arguments**: Check your tool's documentation for correct command-line syntax
- **Permission errors**: Ensure executable files have proper permissions
- **Path issues**: Use absolute paths for all directory and executable references

**Testing Your Configuration:**

1. Run `/debug` workflow to test runner configuration
2. Run `/lint` workflow to test formatter and linter configuration
3. Check workflow logs for any configuration errors
