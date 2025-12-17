# Pras's Claude Code Setup

My personal Claude Code configuration for productive web development. This plugin provides **17 slash commands** and **11 specialized AI agents** to supercharge your development workflow.

## Quick Install

```bash
# Step 1: Add the marketplace
/plugin marketplace add andhikapraa/pras-claude-code

# Step 2: Install the plugin
/plugin install pras-claude-code
```

## What's Inside

### Development Commands (7)

- `/new-task` - Analyze code for performance issues
- `/code-explain` - Generate detailed explanations
- `/code-optimize` - Performance optimization
- `/code-cleanup` - Refactoring and cleanup
- `/feature-plan` - Feature implementation planning
- `/lint` - Linting and fixes
- `/docs-generate` - Documentation generation

### API Commands (4)

- `/api-new` - Create new API endpoints
- `/api-test` - Test API endpoints
- `/api-protect` - Add protection & validation (Supabase/NextAuth)
- `/api-protect-convex` - Add Convex authentication

### UI Commands (2)

- `/component-new` - Create React components (with shadcn/ui)
- `/page-new` - Create Next.js pages

### Supabase Commands (2)

- `/types-gen` - Generate TypeScript types
- `/edge-function-new` - Create Edge Functions

### Convex Commands (2)

- `/schema-gen` - Generate Convex schema and types
- `/function-new` - Create Convex functions

### Specialized AI Agents (11)

**Architecture & Planning**
- **tech-stack-researcher** - Technology choice recommendations with trade-offs
- **system-architect** - Scalable system architecture design
- **backend-architect** - Backend systems with data integrity & security
- **frontend-architect** - Performant, accessible UI architecture
- **requirements-analyst** - Transform ideas into concrete specifications

**Code Quality & Performance**
- **refactoring-expert** - Systematic refactoring and clean code
- **performance-engineer** - Measurement-driven optimization
- **security-engineer** - Vulnerability identification and security standards

**Documentation & Research**
- **technical-writer** - Clear, comprehensive documentation
- **learning-guide** - Teaching programming concepts progressively
- **deep-research-agent** - Comprehensive research with adaptive strategies

## Installation

### From GitHub (Recommended)

```bash
# Add marketplace
/plugin marketplace add andhikapraa/pras-claude-code

# Install plugin
/plugin install pras-claude-code
```

### From Local Clone (for development)

```bash
git clone https://github.com/andhikapraa/pras-claude-code.git
cd pras-claude-code

# Add as local marketplace
/plugin marketplace add /path/to/pras-claude-code

# Install plugin
/plugin install pras-claude-code
```

## Best For

- Next.js developers
- TypeScript projects
- Supabase users
- Convex users
- React developers
- Full-stack engineers

## Usage Examples

### Planning a Feature

```bash
/feature-plan
# Then describe your feature idea
```

### Creating an API

```bash
/api-new
# Claude will scaffold a complete API route with types, validation, and error handling
```

### Research Tech Choices

Just ask Claude questions like:
- "Should I use WebSockets or SSE?"
- "How should I structure this database?"
- "What's the best library for X?"

The tech-stack-researcher agent automatically activates and provides detailed, researched answers.

## Philosophy

This setup emphasizes:
- **Type Safety**: Never uses `any` types
- **Best Practices**: Follows modern Next.js/React patterns
- **Productivity**: Reduces repetitive scaffolding
- **Research**: AI-powered tech decisions with evidence

## Requirements

- Claude Code 2.0.13+
- Works with any project (optimized for Next.js + Supabase/Convex)

## Customization

After installation, you can customize any command by editing files in `.claude/commands/` and `.claude/agents/`.

## Contributing

Feel free to:
- Fork and customize for your needs
- Submit issues or suggestions
- Share your improvements

## Credits

Based on [Edmund's Claude Code Setup](https://github.com/edmund-io/edmunds-claude-code) by Edmund.

## License

MIT - Use freely in your projects

## Author

Created by Pras

---

**Note**: This is my personal setup that I've refined over time. Commands are optimized for Next.js + Supabase/Convex workflows but work great with any modern web stack.
