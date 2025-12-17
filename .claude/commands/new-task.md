---
description: Analyze task complexity and create actionable implementation plan
model: claude-opus-4-5
---

Analyze the following task and create a clear, actionable implementation plan.

## Task

$ARGUMENTS

## MCP Tools Integration

**IMPORTANT:** Use these tools during analysis for better planning:

### Required Tools (Auto-enabled)
1. **Sequential Thinking** - Use this FIRST to break down the task into logical steps
2. **Context7** - Fetch best practices and current documentation for any libraries involved

### Optional Tools (If Available)
3. **Exa** - Web search for research, finding solutions, or checking recent updates
4. **Firecrawl** - Fetch and parse web pages for detailed implementation references

### Workflow
1. **Think** → Use Sequential Thinking to analyze task complexity and structure
2. **Research** → Use Context7 to check best practices for the tech stack
3. **Explore** → If needed, use Exa/Firecrawl to research external solutions
4. **Plan** → Generate the structured implementation plan below

---

## Analysis Framework

### 1. **Task Breakdown**
- Understand requirements
- Identify dependencies
- List affected files/components
- Estimate complexity (Small/Medium/Large)

### 2. **Time Estimation**
- **Small**: 1-2 hours (simple bug fix, minor feature)
- **Medium**: Half day to 1 day (new component, API endpoint)
- **Large**: 2-5 days (complex feature, multiple integrations)
- **Very Large**: 1+ week (major refactor, new subsystem)

### 3. **Risk Assessment**
Identify potential blockers:
- Unknown dependencies
- API limitations
- Data migration needs
- Breaking changes
- Third-party service issues

### 4. **Implementation Steps**

Create sequential, logical steps:
1. Setup/preparation
2. Backend changes
3. Frontend changes
4. Testing
5. Documentation
6. Deployment

### 5. **Success Criteria**

Define "done":
- Feature works as specified
- Tests pass
- No regressions
- Code reviewed
- Documented

## Output Format

### Task Analysis
- **Type**: [Bug Fix / Feature / Refactor / Infrastructure]
- **Complexity**: [Small / Medium / Large / Very Large]
- **Estimated Time**: X hours/days
- **Priority**: [High / Medium / Low]

### Implementation Plan

**Phase 1: [Name]** (Time estimate)
- [ ] Step 1
- [ ] Step 2

**Phase 2: [Name]** (Time estimate)
- [ ] Step 3
- [ ] Step 4

### Files to Modify/Create
```
app/page.tsx (modify)
components/NewComponent.tsx (create)
lib/utils.ts (modify)
```

### Dependencies
```bash
npm install package-name
```

### Testing Strategy
- Unit tests for X
- Integration tests for Y
- Manual testing steps

### Potential Issues
- Issue 1 and mitigation
- Issue 2 and mitigation

### Next Steps
1. Start with Phase 1, Step 1
2. Test incrementally
3. Commit often

Provide a clear, solo-developer-friendly plan that breaks down complex tasks into manageable steps.

---
*Originally created by [Edmund](https://github.com/edmund-io/edmunds-claude-code)*
