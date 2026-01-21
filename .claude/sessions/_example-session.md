# Session 05: Client Dashboard

> **NOTE:** This is an example session file. Delete before running sessions-maker.

## Objective

Implement the Client Dashboard where users view their financial summary, balance evolution chart, returns history, and withdrawal status.

## Scope

**Included:**
- Complete dashboard page with all sections
- Financial summary cards
- Balance evolution chart
- Returns history table
- Withdrawals list
- Agent contact info

**Not included:**
- Withdrawal request flow (Session 07)
- Data export (Session 09)

## Prerequisites

- Session 01: Project Setup
- Session 02: Auth System
- Session 03: Database Setup
- Session 04: Design System

## UI Implementation

### UI Builder Invocation

```yaml
Task: UI Builder Agent
Prompt: |
  Figma: https://figma.com/design/abc123?node-id=100-500
  Target: Client Dashboard (full page)
  Route: /dashboard
  Functionality: |
    - Auth check: redirect to /login if not authenticated
    - Fetch user profile from users table (initial_capital, current_balance, assigned_agent_id)
    - Fetch balance history from capital_movements for chart
    - Fetch returns from monthly_closing_items for table
    - Fetch recent withdrawals from withdrawals table
    - Financial summary: show initial_capital, current_balance, total_returns (calculated)
    - Balance chart: line chart with monthly evolution (last 12 months)
    - Returns table: columns [Month, Applied %, Amount, Balance After], paginated
    - Withdrawals list: recent 5, with status badges
    - Agent card: show assigned agent contact info if exists
    - Loading states for each section
    - Empty states when no data
  Context: |
    projectSpec/projectSpec.md section 2.2 - Dashboard features
    projectSpec/projectSpec.md section 3.5 - Tables: users, capital_movements, monthly_closing_items, withdrawals
    Auth: user_type in (client, agent, super_agent) can access own dashboard
    RLS handles data isolation automatically
```

## Development Tasks

1. [ ] Invoke UI Builder Agent with above spec
2. [ ] Review generated code structure
3. [ ] Install recharts if not present: `npm install recharts`
4. [ ] Verify all data fetching works with test data
5. [ ] Test loading states
6. [ ] Test empty states
7. [ ] Test error boundary
8. [ ] Write Playwright E2E tests
9. [ ] Run code-simplifier

## Technical Details

### Files to Create

- `app/(dashboard)/page.tsx`
- `app/(dashboard)/loading.tsx`
- `app/(dashboard)/error.tsx`
- `app/(dashboard)/_components/*.tsx`
- `types/dashboard.ts`
- `cypress/e2e/dashboard.cy.ts`

### Database Tables

- `users` - profile, assigned_agent_id
- `capital_movements` - balance history
- `monthly_closing_items` - returns details
- `withdrawals` - withdrawal status

### Server Actions

None (read-only page). Export in Session 09.

## Completion Criteria

- [ ] Dashboard renders for authenticated users
- [ ] Unauthenticated redirected to /login
- [ ] All data sections display correctly
- [ ] Chart shows balance evolution
- [ ] Table has working pagination
- [ ] Loading/empty/error states work
- [ ] Playwright tests pass
- [ ] No TS errors
- [ ] Code simplified

## Notes

- Use recharts for chart (shadcn compatible)
- Handle empty state for new users with no movements
