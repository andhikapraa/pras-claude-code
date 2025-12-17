# MCP Servers Included

This plugin includes 4 pre-configured MCP servers that enhance Claude Code's capabilities.

## Included Servers

### 1. **Context7** (`@upstash/context7-mcp`)
**Purpose**: Access up-to-date, version-specific documentation for any library

**Usage**: Just mention "use context7" in your prompt when you need current library documentation

**Benefits**:
- Always up-to-date docs
- Version-specific information
- Works with thousands of libraries
- No manual searching required

### 2. **Sequential Thinking** (`@modelcontextprotocol/server-sequential-thinking`)
**Purpose**: Step-by-step reasoning for complex problem solving

**Capabilities**:
- Break down complex tasks into logical steps
- Structured planning and analysis
- Better reasoning for multi-step problems

**Use Cases**:
- Task planning with `/new-task`
- Complex debugging
- Architecture decisions
- Multi-step implementations

### 3. **Playwright** (`@playwright/mcp`)
**Purpose**: Browser automation and web testing

**Capabilities**:
- Navigate websites
- Take screenshots
- Interact with web elements
- Generate test code
- Access accessibility trees

**Use Cases**:
- E2E testing
- Web scraping
- Browser automation
- Visual testing

### 4. **Supabase** (`@supabase/mcp-server-supabase`)
**Purpose**: Supabase database operations

**Capabilities**:
- Query databases
- Manage tables
- Execute SQL
- Handle authentication
- Work with storage

**Use Cases**:
- Database management
- Schema exploration
- Data queries
- Admin operations

## Optional Servers (Require API Keys)

These servers enhance the `/new-task` command with web research capabilities. Add them to your local `.claude/.mcp.json` if you have API keys.

### Exa Search (`@anthropic/mcp-server-exa`)
**Purpose**: Web search for research and finding solutions

**Setup**:
```json
"exa": {
  "command": "npx",
  "args": ["-y", "@anthropic/mcp-server-exa"],
  "env": { "EXA_API_KEY": "your-key" }
}
```

**Get API Key**: https://exa.ai

### Firecrawl (`@anthropic/mcp-server-firecrawl`)
**Purpose**: Web scraping and page content extraction

**Setup**:
```json
"firecrawl": {
  "command": "npx",
  "args": ["-y", "@anthropic/mcp-server-firecrawl"],
  "env": { "FIRECRAWL_API_KEY": "your-key" }
}
```

**Get API Key**: https://firecrawl.dev

---

## Servers Not Yet Available

The following servers were requested but don't have official MCP implementations yet:

- **chrome-devtools** - No official MCP server found
- **stripe** - No official MCP server found
- **vercel** - No official MCP server found

## Using MCP Servers

After installing this plugin:

1. **Automatic Activation**: MCP servers start automatically when you use the plugin
2. **Restart Required**: Restart Claude Code after plugin installation
3. **Tool Access**: MCP tools appear in Claude's available tools list

## Adding More MCP Servers

You can add custom MCP servers to your local `.claude/.mcp.json`:

```json
{
  "server-name": {
    "command": "npx",
    "args": ["-y", "package-name"],
    "env": {
      "API_KEY": "your-key"
    }
  }
}
```

## Troubleshooting

**MCP servers not loading?**
1. Restart Claude Code
2. Check that npm/npx is installed
3. Verify network connection (MCP servers download on first use)

**Performance issues?**
- MCP servers run on-demand
- First use may be slower (package download)
- Subsequent uses are fast

## Learn More

- Official MCP Documentation: https://modelcontextprotocol.io
- Claude Code MCP Guide: https://docs.claude.com/en/docs/claude-code/mcp
- MCP Server Directory: https://mcpcat.io
