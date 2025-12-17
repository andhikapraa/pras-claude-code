---
description: Universal task planner with AI-powered analysis for any language or framework
model: claude-opus-4-5
---

Analyze and plan the implementation for the following task using structured thinking and best practices research.

## Task

$ARGUMENTS

---

## MCP Tools Integration

**MANDATORY:** Use these tools in order for thorough analysis:

### Step 1: Sequential Thinking (REQUIRED)
Use the Sequential Thinking MCP tool to:
- Break down the task into logical components
- Identify dependencies and order of operations
- Analyze complexity and potential challenges
- Structure your reasoning before exploring code

### Step 2: Context7 (REQUIRED)
Use the Context7 MCP tool to:
- Fetch current documentation for any libraries/frameworks mentioned
- Check best practices and recommended patterns
- Verify API usage and latest conventions

### Step 3: Web Research (IF AVAILABLE)
If Exa or Firecrawl MCP tools are available:
- Search for solutions to similar problems
- Find recent updates or breaking changes
- Research community best practices

---

## Analysis Process

### 1. **Understand the Task**
- What is the core objective?
- What are the inputs and expected outputs?
- What constraints exist?

### 2. **Explore the Codebase**
- Identify relevant existing code
- Understand current patterns and conventions
- Find similar implementations to follow

### 3. **Identify Dependencies**
- External libraries needed
- Internal modules to modify
- Potential conflicts or breaking changes

### 4. **Assess Complexity**
| Level | Description | Characteristics |
|-------|-------------|-----------------|
| **Small** | Quick fix or minor addition | Single file, < 50 lines changed |
| **Medium** | Feature or significant change | 2-5 files, clear scope |
| **Large** | Complex feature or refactor | Multiple files, new patterns |
| **Very Large** | Major system change | Architectural impact, many files |

### 5. **Risk Assessment**
- What could go wrong?
- What assumptions are being made?
- What needs testing?

---

## Output Format

### Task Summary
- **Objective**: [Clear statement of what needs to be done]
- **Type**: [Bug Fix / Feature / Refactor / Infrastructure / Documentation]
- **Complexity**: [Small / Medium / Large / Very Large]
- **Languages/Frameworks**: [Detected from codebase]

### Implementation Plan

**Phase 1: [Preparation]**
- [ ] Step 1
- [ ] Step 2

**Phase 2: [Core Implementation]**
- [ ] Step 3
- [ ] Step 4

**Phase 3: [Testing & Validation]**
- [ ] Step 5
- [ ] Step 6

### Files to Modify
```
path/to/file1 (modify) - reason
path/to/file2 (create) - reason
path/to/file3 (delete) - reason
```

### Dependencies
```bash
# Package manager commands if needed
```

### Testing Strategy
- How to verify the implementation works
- Edge cases to consider
- Manual testing steps

### Potential Issues & Mitigations
| Risk | Likelihood | Mitigation |
|------|------------|------------|
| Issue 1 | Low/Med/High | How to handle |

### Success Criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] All tests pass
- [ ] No regressions

---

**Generate a clear, actionable plan that can be executed step-by-step.**
