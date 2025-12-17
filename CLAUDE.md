# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a Claude Code plugin ("pras-claude-code") that provides 17 slash commands and 11 specialized AI agents for modern web development workflows. The plugin is optimized for Next.js, TypeScript, React, and Supabase/Convex projects.

## Project Structure

```
.claude/
├── commands/           # Slash command definitions (markdown files)
│   ├── api/           # API-related commands (api-new, api-test, api-protect)
│   ├── convex/        # Convex commands (schema-gen, function-new)
│   ├── misc/          # General dev commands (lint, code-optimize, docs-generate, etc.)
│   ├── supabase/      # Supabase commands (types-gen, edge-function-new)
│   ├── ui/            # UI commands (component-new, page-new)
│   └── new-task.md    # Task planning command
├── agents/            # AI agent definitions (11 specialized agents)
└── settings.template.json

.claude-plugin/
├── plugin.json        # Plugin manifest (commands, agents, MCP servers)
├── marketplace.json   # Marketplace registration metadata
└── MCP-SERVERS.md     # MCP server documentation
```

## Key Files

- **`.claude-plugin/plugin.json`**: Central manifest defining all commands, agents, and MCP server configurations. This is what Claude Code reads to understand the plugin structure.
- **Command files**: Markdown files with YAML frontmatter (`description`, `model`) containing prompt templates. Use `$ARGUMENTS` placeholder for user input.
- **Agent files**: Markdown files with YAML frontmatter (`name`, `description`, `model`, `color`) containing agent system prompts.

## Development Workflow

### Adding a New Command

1. Create a markdown file in `.claude/commands/<category>/command-name.md`
2. Add YAML frontmatter with `description` (required) and optional `model`
3. Register the command in `.claude-plugin/plugin.json` under `commands` array
4. Use `$ARGUMENTS` in the prompt to capture user input

### Adding a New Agent

1. Create a markdown file in `.claude/agents/agent-name.md`
2. Add YAML frontmatter with `name`, `description` (required), and optional `model`, `color`
3. Register the agent in `.claude-plugin/plugin.json` under `agents` array

### Plugin Distribution

The plugin is distributed via GitHub. Users install with:
```bash
/plugin marketplace add andhikapraa/pras-claude-code
/plugin install pras-claude-code
```

## Testing Changes

After modifying commands or agents, test by:
1. Uninstall: `/plugin uninstall pras-claude-code`
2. Reinstall from local: `/plugin marketplace add /path/to/repo` then `/plugin install pras-claude-code`
3. Verify command appears with `/` prefix

## Conventions

- Command files use kebab-case naming
- Agents should have descriptive `description` fields that specify when they should be activated
- Plugin version is tracked in `.claude-plugin/plugin.json` (follow semver)
- MCP servers are defined in `plugin.json` under `mcpServers` object

## Planning Mode Behavior

**IMPORTANT:** When entering plan mode (via `EnterPlanMode`) or planning any implementation task, ALWAYS use these MCP tools:

### Required Steps for Planning

1. **Sequential Thinking FIRST**
   - Use `mcp__sequential-thinking__*` tools to break down the task
   - Analyze complexity and dependencies
   - Structure your thinking before exploring code

2. **Context7 for Best Practices**
   - Use `mcp__context7__*` tools to fetch current documentation
   - Check best practices for libraries/frameworks involved
   - Ensure recommendations follow latest patterns

3. **Optional: Web Research**
   - If Exa is available: Use for searching solutions and recent updates
   - If Firecrawl is available: Use for fetching detailed implementation references

### Planning Workflow

```
Task Received
    ↓
[Sequential Thinking] → Break down into logical steps
    ↓
[Context7] → Check best practices for tech stack
    ↓
[Exa/Firecrawl] → Research if needed (optional)
    ↓
[Explore Codebase] → Understand existing patterns
    ↓
[Write Plan] → Create structured implementation plan
```

This ensures all plans are well-structured and follow current best practices.
