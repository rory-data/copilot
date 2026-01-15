---
description: "Shared tool configurations and usage patterns"
applyTo: "**"
---

# Tool Configurations

## Quick Reference

- **Core Tools**: `codebase`, `search`, `editFiles`, `runCommands` - available in most modes
- **Analysis Tools**: `usages`, `problems`, `changes`, `findTestFiles` - for code investigation
- **External Tools**: `fetch`, `githubRepo`, `openSimpleBrowser` - for research
- **Execution Tools**: `runTests`, `runTasks`, `runNotebooks` - for validation

## Common Tool Groups

**Code Analysis Group**: codebase, usages, search, problems, changes, findTestFiles  
→ Use for: Understanding existing code, finding usage patterns, identifying issues

**Implementation Group**: editFiles, runCommands, runTasks, runTests, new  
→ Use for: Making code changes, executing builds, running tests

**Research Group**: fetch, githubRepo, openSimpleBrowser, searchResults  
→ Use for: External research, documentation lookup, best practice investigation

**Validation Group**: runTests, testFailure, problems, terminalSelection, terminalLastCommand  
→ Use for: Testing implementations, debugging failures, checking results

**Full Development Group**: All tools above combined  
→ Use for: Complete development workflows requiring all capabilities

## Tool Usage Patterns

### Progressive Tool Use

1. **Start with Research**: `search`, `codebase`, `usages`
2. **Plan Changes**: `findTestFiles`, `problems`
3. **Implement**: `editFiles`, `new`
4. **Validate**: `runTests`, `runCommands`
5. **Debug**: `testFailure`, `terminalSelection`, `problems`

### Intelligent Tool Selection

- **Use `codebase` first** to understand existing structure
- **Use `search` for specific patterns** or functionality
- **Use `usages` for impact analysis** before changes
- **Use `problems` to identify existing issues**
- **Use `fetch` for external documentation** or standards

### Tool Combination Strategies

- **Code Investigation**: `codebase` + `search` + `usages`
- **Safe Refactoring**: `usages` + `findTestFiles` + `runTests`
- **Feature Development**: `codebase` + `editFiles` + `runTests` + `runCommands`
- **Bug Fixing**: `problems` + `testFailure` + `editFiles` + `runTests`

## VSCode-Specific Tools

### API Integration

```yaml
tool: "vscodeAPI"
```

**Use for**: VS Code extension development, accessing editor capabilities

### Extension Management

```yaml
tool: "extensions"
```

**Use for**: Installing and configuring VS Code extensions

## Terminal Integration

### Command Execution

```yaml
tools: ["runCommands", "terminalSelection", "terminalLastCommand"]
```

**Use for**: Running build scripts, package managers, deployment commands

### Output Analysis

- **`terminalSelection`**: Get selected terminal text
- **`terminalLastCommand`**: Retrieve last executed command
- **`runCommands`**: Execute and capture command output

## Notebook-Specific Tools

### Jupyter Integration

```yaml
tool: "runNotebooks"
```

**Use for**: Executing notebook cells, data analysis workflows

## Best Practices

### Tool Efficiency

- **Minimise tool calls** by using appropriate scope
- **Combine related operations** in single tool calls where possible
- **Use progressive refinement** rather than broad searches

### Error Handling

- **Check `problems`** before and after changes
- **Use `testFailure`** to diagnose test issues
- **Leverage `terminalSelection`** for command debugging

### Context Management

- **Start broad** (`codebase`) then narrow (`search`)
- **Validate assumptions** with `usages` and `findTestFiles`
- **Confirm changes** with `runTests` and `problems`
