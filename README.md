# Logic Language Support

[![VS Code Extension](https://img.shields.io/badge/VS%20Code-Extension-blue.svg)](https://marketplace.visualstudio.com/items?itemName=logicer.logic-support)
[![Context7 Documentation](https://img.shields.io/badge/Context7-Documentation-green.svg)](https://context7.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A comprehensive toolkit for Logic scripting language that provides both **VS Code extension** and **Context7 documentation** support, enabling developers to work with Logic in any development environment.

## 🚀 Dual Platform Support

### VS Code Extension
Complete IDE support for Logic development with syntax highlighting, code snippets, and intelligent editing features.

### Context7 Integration
AI-powered documentation accessible in any editor through Context7, providing instant Logic syntax help in Cursor, Claude Desktop, and other AI-enabled editors.

## ✨ Features

### VS Code Extension Features
- **Syntax Highlighting** - Full syntax highlighting for `.logic` files
- **Code Snippets** - 15+ pre-built code templates for common Logic patterns
- **Bracket Matching** - Automatic bracket and quote pairing
- **Language Configuration** - Proper indentation and formatting rules
- **File Association** - Automatic `.logic` file recognition

### Context7 Documentation Features
- **AI Editor Support** - Works with Cursor, VS Code with AI extensions, Claude Desktop
- **Comprehensive Documentation** - Complete Logic language reference
- **Interactive Examples** - Runnable code examples and templates
- **Plugin Reference** - Detailed plugin system documentation
- **Best Practices** - Enterprise-grade development guidelines

![功能预览](https://logic-6365.obs.cn-north-4.myhuaweicloud.com/preview.png)

## 📚 Documentation

| Document | Description | Target Audience |
|----------|-------------|-----------------|
| [Quick Start Guide](docs/getting-started.md) | Get up and running with Logic in minutes | Beginners |
| [Syntax Reference](docs/syntax-reference.md) | Complete syntax rules and examples | All developers |
| [Database Operations](docs/database-operations.md) | entity and sql object usage | Backend developers |
| [Plugin System](docs/plugin-system.md) | Built-in and custom plugins | All developers |
| [Advanced Features](docs/advanced-features.md) | Lambda, async, multi-datasource | Advanced users |
| [Best Practices](docs/best-practices.md) | Enterprise development guidelines | Team leads |

## 🎯 What is Logic?

Logic is a JavaScript/Python-like scripting language designed for enterprise business logic development. It's part of the AF SystemV4 architecture (琉璃架构) and provides:

### Core Capabilities
- **Database Integration** - Built-in ORM and SQL operations
- **Plugin System** - Extensive Java interoperability
- **Enterprise Features** - Authentication, logging, validation
- **Async Operations** - Non-blocking task processing
- **Multi-datasource** - Support for multiple databases

### Syntax Highlights
```javascript
// Parameter validation
validate {
    name: {
        required: true,
        message: "Name is required"
    }
},

// Database operations
user = entity.getById("t_user", data.userId),
user != null : (
    log.info("User found: " + user.f_name)
), (
    throw "User not found"
),

// Plugin usage
currentTime = dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss"),
isValid = commonTools.isNotEmpty(data.email),

// Return result
return {
    success: true,
    data: user,
    timestamp: currentTime
}
```

## 🛠️ Installation & Setup

### For VS Code Users

1. **Install the Extension**
   ```bash
   # Search "Logic Support" in VS Code Extensions
   # Or install via command line
   code --install-extension logicer.logic-support
   ```

2. **Start Coding**
   - Create a `.logic` file
   - Enjoy syntax highlighting and code snippets
   - Use `Ctrl+Space` for code completion

### For AI Editor Users (Context7)

#### Cursor Setup
Add to your `~/.cursor/mcp.json`:
```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["@upstash/context7-mcp"],
      "env": {
        "CONTEXT7_API_KEY": "your-api-key"
      }
    }
  }
}
```

#### Claude Desktop Setup  
Add to your `claude_desktop_config.json`:
```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["@upstash/context7-mcp"],
      "env": {
        "CONTEXT7_API_KEY": "your-api-key"
      }
    }
  }
}
```

#### Usage
Simply ask your AI editor questions like:
- "How do I query a database in Logic?"
- "Show me Logic syntax for conditions"
- "What plugins are available in Logic?"

## 📖 Quick Examples

### Basic CRUD Operations
```javascript
// Create/Update User
validate {
    name: { required: true },
    email: { required: true }
},

userData = {
    f_name: data.name,
    f_email: data.email,
    f_create_time: dateTools.format(dateTools.now(), "yyyy-MM-dd HH:mm:ss")
},

result = entity.partialSave("t_user", userData),

return {
    success: true,
    userId: result.id
}
```

### Complex Query with Plugins
```javascript
// Search users with pagination
validate {
    keyword: { default: "" },
    page: { default: 1 },
    pageSize: { default: 20 }
},

offset = (data.page - 1) * data.pageSize,

users = sql.querySQL("searchUsers", "
    SELECT u.*, d.f_name as dept_name
    FROM t_user u
    LEFT JOIN t_department d ON u.f_dept_id = d.id  
    WHERE u.f_name LIKE '%{data.keyword}%'
    ORDER BY u.f_create_time DESC
    LIMIT {data.pageSize} OFFSET {offset}
"),

// Process results
processedUsers = [],
users.each(
    processedUsers.push({
        id: row.id,
        name: row.f_name,
        email: row.f_email,
        department: row.dept_name,
        avatar: commonTools.isNotEmpty(row.f_avatar) ? row.f_avatar : "/default-avatar.png"
    })
),

return {
    success: true,
    data: processedUsers,
    pagination: {
        page: data.page,
        pageSize: data.pageSize,
        total: users.length
    }
}
```

## 🔧 VS Code Snippets Reference

| Trigger | Description | Usage |
|---------|-------------|-------|
| `validate` | Parameter validation block | Data validation |
| `condition` | Simple conditional | Basic if-else logic |
| `multicondition` | Multiple conditions | Complex branching |
| `each` | Loop through items | Array/object iteration |
| `trycatch` | Exception handling | Error management |
| `lambda` | Lambda expression | Functional programming |
| `logicrun` | Call other Logic | Modular development |
| `logictemplate` | Complete Logic template | Quick scaffolding |

## 🌟 Why Choose This Toolkit?

### For VS Code Developers
- **Native IDE Experience** - Full language support in VS Code
- **Productivity Boost** - Code snippets and syntax highlighting
- **Error Prevention** - Proper syntax validation and formatting

### for AI Editor Users
- **Universal Access** - Works in any AI-enabled editor
- **Context-Aware Help** - Get relevant code suggestions
- **Always Up-to-Date** - Documentation stays current with language updates

### For Teams
- **Standardization** - Consistent development experience across tools
- **Knowledge Sharing** - Comprehensive documentation accessible to all
- **Best Practices** - Built-in guidelines for enterprise development

## 🤝 Contributing

We welcome contributions! Here's how you can help:

### Documentation Improvements
- Update examples and use cases
- Add new plugin documentation
- Improve existing guides

### VS Code Extension
- Report bugs and issues
- Suggest new snippets
- Enhance syntax highlighting

### Context7 Integration
- Add new documentation sections
- Improve AI editor compatibility
- Enhance search and discovery

## 📝 Development

### Project Structure
```
logicsupport-vscodeext/
├── docs/                    # Context7 documentation
│   ├── getting-started.md
│   ├── syntax-reference.md
│   ├── database-operations.md
│   ├── plugin-system.md
│   ├── advanced-features.md
│   └── best-practices.md
├── examples/                # Code examples
│   └── sample.logic
├── snippets/               # VS Code snippets
│   └── logic.json
├── syntaxes/              # Syntax highlighting
│   └── logic.tmLanguage.json
├── language/              # Language configuration
│   └── language-configuration.json
├── context7.json          # Context7 configuration
├── package.json          # VS Code extension manifest
└── README.md             # This file
```

### Building the Extension
```bash
# Install dependencies
npm install

# Package the extension
npx vsce package

# Install locally
code --install-extension logic-support-*.vsix
```

## 📞 Support & Feedback

- **🐛 Bug Reports**: [GitHub Issues](https://github.com/your-username/logicsupport-vscodeext/issues)
- **💡 Feature Requests**: [GitHub Discussions](https://github.com/your-username/logicsupport-vscodeext/discussions)
- **📧 Email**: support@logic-lang.com
- **📖 Documentation**: [Context7 Page](https://context7.com/logic)

## 🏷️ Version History

### 1.0.0 (Current)
- ✅ Complete VS Code extension with syntax highlighting
- ✅ 15+ code snippets for common patterns
- ✅ Context7 integration for AI editors
- ✅ Comprehensive documentation suite
- ✅ Database operations guide
- ✅ Plugin system reference
- ✅ Best practices documentation

### Roadmap
- 🔄 Live syntax validation
- 🔄 Integrated debugger support
- 🔄 Code formatting tools
- 🔄 Interactive documentation

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

**Enjoy coding with Logic! 🎉**

Whether you're using VS Code or any AI-enabled editor, this toolkit provides everything you need for productive Logic development. Get started today and experience the power of modern Logic development tools.