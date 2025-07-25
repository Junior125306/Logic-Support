# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a VS Code extension that provides comprehensive support for the Logic scripting language, part of the AF SystemV4 architecture. The extension includes syntax highlighting, code snippets, language configuration, and Context7 integration for AI editors.

Logic is a JavaScript/Python-like scripting language designed for enterprise business logic development with built-in database operations, plugin system, and Java interoperability.

## Common Development Commands

### Building and Packaging

```bash
# Package the VS Code extension
npx vsce package

# Install the packaged extension locally
code --install-extension logic-support-*.vsix
```

### Installation

```bash
# Install dependencies (if any)
npm install

# Install the extension in VS Code directly
code --install-extension logic-support-*.vsix
```

## Architecture and Key Components

### Core VS Code Extension Files

- `package.json` - Extension manifest defining languages, grammars, snippets, and contributions
- `syntaxes/logic.tmLanguage.json` - TextMate grammar for Logic syntax highlighting
- `language/language-configuration.json` - Language configuration (brackets, comments, auto-closing pairs)
- `snippets/logic.json` - Code snippets for common Logic patterns

### Context7 Integration

- `context7.json` - Context7 configuration for AI editor integration
- `docs/` - Comprehensive documentation accessible through Context7 MCP servers

### Documentation Structure

- `docs/getting-started.md` - Quick start guide for Logic development
- `docs/syntax-reference.md` - Complete Logic syntax reference
- `docs/database-operations.md` - Entity and SQL object usage
- `docs/plugin-system.md` - Built-in and custom plugins
- `docs/advanced-features.md` - Lambda, async, multi-datasource features
- `docs/best-practices.md` - Enterprise development guidelines

## Logic Language Key Concepts

### Syntax Rules (from context7.json)

- Every expression must end with a comma (,) except return statements
- Conditional expressions must have both true and false branches, use null for empty branches
- Use `data.fieldName` to access input parameters passed to Logic scripts

### Built-in Objects

- `entity` - Database ORM operations (getById, partialSave, deleteById)
- `sql` - SQL query operations (querySQL, execute with parameterized queries)
- `log` - Logging operations (info, error, debug, warn)
- `logic` - Call other Logic scripts (apply, run, remoteRun, runAsync)

### Common Plugins

- `commonTools` - General utilities
- `dateTools` - Date/time operations
- `jsonTools` - JSON manipulation
- `restTools` - REST API operations
- `securityUtils` - Security functions
- `convertTools` - Data conversion

### Language Features

- Parameter validation with `validate` blocks
- Exception handling with try-catch blocks and throw statements
- Loop operations using `.each()` with row, rowIndex, rowKey variables
- Lambda expressions with `(params) => { return result }` syntax and `.apply()` method
- String interpolation in double quotes using `{variable}` syntax

## File Extensions and Recognition

- Logic files use `.logic` extension
- Syntax highlighting automatically applies to `.logic` files
- Language server features are configured for Logic language ID

## Development Workflow

1. Edit syntax highlighting in `syntaxes/logic.tmLanguage.json`
2. Update language configuration in `language/language-configuration.json`
3. Add/modify code snippets in `snippets/logic.json`
4. Test changes by packaging and installing the extension
5. Update documentation in `docs/` directory for Context7 integration

## Context7 AI Editor Support

The extension provides Context7 integration allowing AI editors (Cursor, Claude Desktop, VS Code with AI extensions) to access comprehensive Logic documentation. The `context7.json` file defines rules, capabilities, and use cases for AI assistance.