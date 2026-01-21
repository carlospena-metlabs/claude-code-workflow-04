Analyze the projectSpec.md and pages-tree.md files to generate a development session plan to build the MVP.

## Input Files

1. `.claude/docs/projectSpec/projectSpec.md` - Technical specification, features, database schema
2. `.claude/docs/pages-tree.md` - Pages structure with Figma URLs

**Important:** Read pages-tree.md to get the Figma URLs for each page. Use these URLs in the UI Builder invocations.

## Session Structure

For each session, create a markdown file with this structure:

```markdown
# Session [N]: [Title]

## Objective
[Clear description of what this session achieves]

## Scope
[What's included and what's NOT included]

## Prerequisites
- [Previous sessions that must be completed]
- [Dependencies that must be installed]

## UI Implementation (if applicable)

When this session requires UI, include ONE invocation block per PAGE:

### UI Builder Invocation

\`\`\`yaml
Task: UI Builder Agent
Prompt: |
  Figma: [URL from pages-tree.md]
  Target: [full page name]
  Route: [destination route]
  Functionality: |
    - [all required functionality for this page]
    - [server actions needed]
    - [data fetching required]
    - [validations]
    - [interactive behaviors]
  Context: |
    [relevant projectSpec sections]
    [database tables involved]
    [authorization rules]
\`\`\`

**Rule: One invocation = One complete page.** The agent decides internally what to extract to `_components/`.

**Exception:** For reusable components shared across MULTIPLE pages (e.g., a chart used in Dashboard AND Backoffice), create a separate invocation BEFORE the pages that use it, targeting `/components/[ComponentName].tsx`.

## Development Tasks

1. [ ] Task 1: [description]
2. [ ] Task 2: [description]
3. [ ] Task N: [description]

## Technical Details

### Files to Create/Modify
- `path/to/file.ts` - [purpose]

### Database Tables Involved
- `table_name` - [how it's used]

### Server Actions Required
- `actionName` - [what it does]

## Completion Criteria

- [ ] All tasks completed
- [ ] Functional tests pass (Playwright MCP)
- [ ] No TypeScript errors
- [ ] No linter errors
- [ ] Code simplified (run code-simplifier)

## Notes
[Any additional context or warnings]
```

## Generation Rules

1. **Define objective and scope** - Be specific about what's in/out
2. **Identify UI requirements** - If UI needed, include full ui-builder invocation block
3. **Reference projectSpec.md** - Link to relevant sections for business logic
4. **List concrete tasks** - Each task should be independently verifiable
5. **Include technical details** - Files, tables, actions involved
6. **Define completion criteria** - Always include Playwright tests
7. **End with code-simplifier** - Clean up at session end

## Organization

- Order sessions by logical dependencies
- Each session must be self-contained and independently executable
- Save all sessions in `.claude/sessions/` directory
- Name format: `session-[NN]-[slug].md` (e.g., `session-01-project-setup.md`)

## Output

1. Session files in `.claude/sessions/`
2. `CLAUDE.md` file with execution instructions