Analyze the projectSpec.md and pages-tree.md files to generate a development session plan to build the MVP.

## Input Files

1. `.claude/docs/projectSpec/projectSpec.md` - Technical specification, features, database schema
2. `.claude/docs/pages-tree.md` - Pages/screens structure with Figma URLs

**Important:** Read pages-tree.md to get the Figma URLs for each page/screen. Use these URLs in the UI/API Builder invocations.

## Development Phases

Sessions are organized by development phases:

### Phase 1: Backend + Web Backoffice
- Session 01: API Project Setup (NestJS)
- Session 02: Web Project Setup (React.js + Vite)
- Session 03: Database Schema & Migrations
- Session 04: Authentication Module
- Session 05: Web Design System
- Session 06+: Feature modules (API + Web pages)

### Phase 2: Mobile Application (separate execution)
- Session M01: Mobile Project Setup (React Native)
- Session M02: Mobile Design System
- Session M03+: Mobile screens

### Phase 3: Blockchain (separate execution)
- Session B01: Contracts Project Setup (Hardhat)
- Session B02+: Smart contracts development

## Session Structure

For each session, create a markdown file with this structure:

```markdown
# Session [N]: [Title]

## Objective
[Clear description of what this session achieves]

## Project
[Which repository this session affects: api | web | mobile | contracts]

## Scope
[What's included and what's NOT included]

## Prerequisites
- [Previous sessions that must be completed]
- [Dependencies that must be installed]

## API Implementation (if applicable)

When this session requires API endpoints, include ONE invocation block per module:

### API Builder Invocation

\`\`\`yaml
Task: API Builder Agent
Prompt: |
  Module: [module name, e.g., users, orders]
  Endpoints:
    - GET /api/v1/[resource] - [description]
    - POST /api/v1/[resource] - [description]
    - GET /api/v1/[resource]/:id - [description]
    - PATCH /api/v1/[resource]/:id - [description]
    - DELETE /api/v1/[resource]/:id - [description]
  Entity: |
    [TypeORM entity fields and relationships]
  Business Logic: |
    - [validation rules]
    - [business rules]
    - [side effects: emails, notifications, etc.]
  Context: |
    [relevant projectSpec sections]
    [related modules]
\`\`\`

## UI Implementation (if applicable)

When this session requires Web UI, include ONE invocation block per PAGE:

### UI Builder Invocation (Web)

\`\`\`yaml
Task: UI Builder Agent (Web)
Prompt: |
  Figma: [URL from pages-tree.md]
  Target: [full page name]
  Route: [destination route]
  API Endpoints: |
    - GET /api/v1/[resource] - for listing
    - POST /api/v1/[resource] - for creation
  Functionality: |
    - [all required functionality for this page]
    - [form validations]
    - [interactive behaviors]
  Context: |
    [relevant projectSpec sections]
    [authorization rules]
\`\`\`

**Rule: One invocation = One complete page.** The agent decides internally what to extract to components.

## Mobile Implementation (if applicable - Phase 2)

### Mobile Builder Invocation

\`\`\`yaml
Task: Mobile Builder Agent
Prompt: |
  Figma: [URL from pages-tree.md]
  Target: [screen name]
  Navigation: [stack/tab position]
  API Endpoints: |
    - GET /api/v1/[resource]
  Functionality: |
    - [screen functionality]
    - [gestures, animations]
  Context: |
    [relevant projectSpec sections]
\`\`\`

## Development Tasks

1. [ ] Task 1: [description]
2. [ ] Task 2: [description]
3. [ ] Task N: [description]

## Technical Details

### Files to Create/Modify
- `path/to/file.ts` - [purpose]

### Database Tables Involved (if applicable)
- `table_name` - [how it's used]

### API Endpoints Involved
- `GET /api/v1/resource` - [what it does]

## Completion Criteria

- [ ] All tasks completed
- [ ] Tests pass (Jest for API, Playwright for Web)
- [ ] No TypeScript errors
- [ ] No linter errors
- [ ] Code simplified (run code-simplifier)

## Notes
[Any additional context or warnings]
```

## Generation Rules

1. **Define objective and scope** - Be specific about what's in/out
2. **Specify project** - Which repository (api, web, mobile, contracts)
3. **API first** - For features, always implement API module before Web/Mobile UI
4. **Identify UI requirements** - If UI needed, include full builder invocation block
5. **Reference projectSpec.md** - Link to relevant sections for business logic
6. **List concrete tasks** - Each task should be independently verifiable
7. **Include technical details** - Files, tables, endpoints involved
8. **Define completion criteria** - Always include tests
9. **End with code-simplifier** - Clean up at session end

## Session Order (Phase 1)

```
Session 01: API Project Setup
    └── Initialize NestJS, configure TypeORM, setup Docker for local DB

Session 02: Web Project Setup
    └── Initialize React.js + Vite, configure Tailwind, setup API client

Session 03: Database Schema
    └── Create all TypeORM entities and run migrations
    └── Depends on: Session 01

Session 04: Auth Module (API)
    └── JWT auth, login, register, refresh tokens
    └── Depends on: Session 03

Session 05: Auth Integration (Web)
    └── Login page, auth context, protected routes
    └── Depends on: Session 02, Session 04

Session 06: Web Design System
    └── Run /figma/prompt1-tokens + /figma/prompt2-components
    └── Depends on: Session 02

Session 07+: Feature Modules
    └── Each feature: API module first, then Web pages
    └── Depends on: Session 04, Session 06
```

## Organization

- Order sessions by logical dependencies
- Each session must be self-contained and independently executable
- Save all sessions in `.claude/sessions/` directory
- Name format: `session-[NN]-[slug].md` (e.g., `session-01-api-setup.md`)
- For Phase 2: `session-M[NN]-[slug].md`
- For Phase 3: `session-B[NN]-[slug].md`

## Output

1. Session files in `.claude/sessions/`
2. `CLAUDE.md` file with execution instructions
3. Summary of session dependencies
