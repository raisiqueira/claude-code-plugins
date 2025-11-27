# Claude Code Plugins Collection

A curated collection of specialized Claude Code plugins that provide expert development guidance for popular frameworks and libraries. Each plugin includes custom agents, skills, and tools tailored for specific development scenarios.

This collection is under active development. Contributions and suggestions are welcome!

## What are Claude Code Plugins?

Claude Code plugins extend Claude's capabilities with specialized knowledge and workflows for specific frameworks, libraries, or development tasks. Each plugin can include:

- **Agents**: Specialized AI assistants with deep expertise in specific technologies
- **Skills**: Reusable workflows and code generation capabilities
- **Commands**: Custom slash commands for common tasks
- **Hooks**: Event-driven automation for your development workflow

## Available Plugins

This collection includes specialized plugins for common development scenarios:

### Frontend Design
**Location**: `plugins/frontend-design/`
- **Capabilities**: Create distinctive, production-grade frontend interfaces with high design quality
- **Includes**: Skills for building web components, pages, and applications
- **Best For**: Generating creative, polished UI code that avoids generic AI aesthetics

### Next.js Expert
**Location**: `plugins/next-js-expert/`
- **Capabilities**: Expert guidance on Next.js development with App Router
- **Includes**: Specialized agent with deep Next.js knowledge
- **Best For**: Server/Client Components, data fetching, routing, performance optimization

### React Router Expert
**Location**: `plugins/react-router-expert/`
- **Capabilities**: Expert guidance on React development with React Router V7
- **Includes**: Specialized agent for modern React Router patterns
- **Best For**: Complex routing, data loading, error boundaries, performance optimization

### Vue.js Expert
**Location**: `plugins/vue-js-expert/`
- **Capabilities**: Expert Vue.js 3 development assistance
- **Includes**: Specialized agent for Vue 3 Composition API
- **Best For**: Component architecture, state management, reactivity, TypeScript integration

## Installation

### Prerequisites

Make sure you have Claude Code installed. Visit the [official documentation](https://docs.anthropic.com/en/docs/claude-code) for installation instructions.

### Option 1: Install from GitHub (Recommended)

Add this plugin marketplace to your Claude Code installation:

```bash
# Add the marketplace
/plugin marketplace add raisiqueira/claude-code-plugins

# Install individual plugins
/plugin install nextjs-code-expert@raisiqueira-plugins
/plugin install react-router-expert@raisiqueira-plugins
/plugin install vue-js-expert@raisiqueira-plugins
/plugin install frontend-design@raisiqueira-plugins

# Or browse and install interactively
/plugin
```

### Option 2: Local Development/Testing

For local testing or development:

```bash
# Clone the repository
git clone https://github.com/raisiqueira/claude-code-plugins.git
cd claude-code-plugins

# Add the local marketplace
/plugin marketplace add .

# Install plugins from local marketplace
/plugin install nextjs-code-expert@raisiqueira-plugins
/plugin install react-router-expert@raisiqueira-plugins
/plugin install vue-js-expert@raisiqueira-plugins
/plugin install frontend-design@raisiqueira-plugins
```

### Verify Installation

List your installed marketplaces and plugins:

```bash
/plugin marketplace list
/plugin list
```

## Contributing

Contributions are welcome! If you'd like to add a new plugin or improve an existing one:

1. Fork this repository
2. Create a new branch for your plugin or improvement
3. Follow the plugin structure used in existing plugins
4. Submit a pull request with a clear description

### Plugin Structure

Each plugin follows the official Claude Code plugin structure:

```
plugins/your-plugin-name/
├── .claude-plugin/
│   └── plugin.json                    # Plugin manifest (required)
├── .claude/
│   ├── agents/                        # Custom agents (optional)
│   │   └── your-agent.md
│   ├── skills/                        # Custom skills (optional)
│   │   └── your-skill/
│   │       └── SKILL.md
│   ├── commands/                      # Custom commands (optional)
│   │   └── your-command.md
│   └── hooks/                         # Custom hooks (optional)
│       └── hooks.json
├── .mcp.json                          # MCP servers (optional)
└── README.md                          # Plugin documentation
```

The `plugin.json` manifest must include:
- `name`: Plugin identifier (kebab-case)
- `version`: Semantic version (e.g., "1.0.0")
- `description`: Clear description of plugin capabilities
- `author`: Object with name and optional URL
- `repository`: String URL to the repository
- `license`: License identifier (e.g., "MIT")

## References

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Awesome Claude Agents](https://github.com/vijaythecoder/awesome-claude-agents)
- [React Router Cursor Rules](https://github.com/brookslybrand/react-router-cursor-rules)

## License

This collection is licensed under the [MIT License](https://opensource.org/license/mit/).