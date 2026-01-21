# .claude/docs/projectSpec/projectSpec.md

## 1. Product Requirements

### 1.1 Product Overview

APPCHULA is a private institutional capital management and visualization platform designed for high-value clients in regulated investment environments. The platform serves as a technology layer providing visibility and control over investment processes that occur primarily outside the system.

**Target Users:**
- Family Offices
- Venture Capital firms
- Patrimonial companies
- Institutional investors
- High net worth individuals (exceptional cases)

**Core Value Proposition:**
- Clear and updated view of capital and balance
- Transparent reflection of returns evolution over time
- Internal control, traceability, and administrative management tool

The platform does not replace financial, contractual, or compliance processes. All clients accessing the platform have been pre-qualified commercially, validated by compliance, and accepted internally before gaining access.

### 1.2 Goals and Success Criteria

**Primary Goals:**
- Provide institutional clients with a secure, frictionless dashboard to view their investment status
- Enable administrators to manage users, balances, returns, and withdrawals with full traceability
- Maintain complete audit trails for all financial operations
- Support agent/super-agent commission structures

**Success Metrics:**
- 100% audit trail coverage for financial operations
- Zero unauthorized data modifications
- Sub-second dashboard load times
- 99.9% platform availability
- Complete compliance with withdrawal window rules

### 1.3 In Scope / Out of Scope

**In Scope:**
- User management (Clients, Agents, Super Agents) via Backoffice
- Client dashboard with financial visualization
- Monthly returns calculation and application
- Quarterly withdrawal request system
- Agent and Super Agent commission management
- Backoffice for administrative operations
- Email notifications for key events
- Audit logging for all critical operations
- Data export (CSV/Excel)
- Responsive web interface (desktop, tablet, mobile)

**Out of Scope:**
- Public registration or self-service onboarding
- Direct deposit/investment flows within platform
- Automatic bank transfers or crypto transactions
- Native mobile applications
- Multi-currency support (USD only)
- Real-time return calculations
- KYC/KYB verification (prepared for future integration)
- Social login or third-party authentication

---

## 2. Product Roadmap

### 2.1 Milestones Overview

| Phase | Focus | Description |
|-------|-------|-------------|
| MVP | Core Platform | Authentication, Client Dashboard, Basic Backoffice, Monthly Closing |
| v1.1 | Withdrawals & Commissions | Full withdrawal flow, Agent/Super Agent commission system |
| v1.2 | Enhanced Backoffice | Advanced reporting, bulk operations, improved audit views |
| v2.0 | KYC Integration | Sumsub integration, verification workflows |

### 2.2 MVP Definition

The MVP delivers a fully functional platform enabling administrators to manage institutional clients and their investments, while clients can view their financial status through a secure dashboard.

**Core Features:**

1. **Authentication System**
   - Invitation-based account activation with token (24h expiry)
   - Email + password login
   - 2FA via email OTP for sensitive actions
   - Session management (20-minute inactivity timeout)
   - Password recovery via email

2. **Client Dashboard**
   - Financial summary (initial capital, current balance, accumulated return)
   - Balance evolution chart (monthly)
   - Returns history table
   - Withdrawals history and status tracking
   - Assigned agent contact information
   - Data export (CSV/Excel)

3. **Backoffice - User Management**
   - Create users with type (Client, Agent, Super Agent)
   - Assign operational roles (Administrator, Operator)
   - Manage user status (Active, Blocked)
   - Send/resend invitation emails
   - Internal notes per user

4. **Backoffice - Financial Management**
   - Input and manage initial capital
   - Configure guaranteed minimum return percentage per client
   - Execute monthly closing process with validation
   - View and export financial data

5. **Monthly Closing Process**
   - Input monthly return percentage
   - Validation list with per-client approval
   - Irreversible balance updates
   - Historical record generation

6. **Withdrawal System**
   - Client withdrawal requests within valid windows
   - Administrator approval/execution workflow
   - Status tracking (Requested → Approved → Executed)

7. **Agent/Super Agent Commissions**
   - Commission calculation based on client balances
   - Monthly accrual for Agents (quarterly payout)
   - Monthly credit for Super Agents

8. **Audit & Logging**
   - Immutable logs for all critical operations
   - User action tracking with timestamps

**MVP Limitations:**
- KYC/KYB fields exist but verification is manual/external
- No automated bank transfers (execution marked manually)
- Single monthly return percentage for all clients (with individual minimums)
- No real-time data updates

### 2.3 Future Versions

**Version 1.1 - Enhanced Operations**
- Batch withdrawal processing
- Advanced commission reports
- Agent hierarchy visualization
- Improved notification templates

**Version 1.2 - Reporting & Analytics**
- Custom report builder
- Historical trend analysis
- Administrative dashboard with KPIs
- Bulk user operations

**Version 2.0 - KYC Integration**
- Sumsub integration for KYC/KYB
- Verification status workflows
- Document upload and management
- Compliance reporting

---

## 3. Technical Specification

### 3.1 Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | Next.js 14+ (App Router) |
| Styling | Tailwind CSS + shadcn/ui |
| Backend | Supabase (PostgreSQL, Auth, Storage, Edge Functions) |
| Email | Resend (SMTP) |
| Testing | Cypress (E2E), Vitest (Unit) |
| Deployment | Vercel |
| Documentation | MCP Context7 for library documentation |

### 3.2 System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Client Browser                          │
│                    (Desktop / Tablet / Mobile)                  │
└─────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Next.js Application                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │   App Router │  │    Server    │  │      Middleware      │  │
│  │   (Pages)    │  │   Actions    │  │  (Auth, RLS Check)   │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                         Supabase                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  PostgreSQL  │  │     Auth     │  │       Storage        │  │
│  │   (+ RLS)    │  │   (JWT)      │  │   (Documents)        │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    Edge Functions                         │  │
│  │         (Scheduled jobs, complex operations)              │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                          Resend                                 │
│                    (Transactional Email)                        │
└─────────────────────────────────────────────────────────────────┘
```

### 3.3 Folder Structure

```
/
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   ├── activate/
│   │   ├── forgot-password/
│   │   └── reset-password/
│   ├── (dashboard)/
│   │   ├── layout.tsx
│   │   ├── page.tsx                    # Client dashboard home
│   │   ├── returns/
│   │   ├── withdrawals/
│   │   └── profile/
│   ├── (backoffice)/
│   │   ├── layout.tsx
│   │   ├── page.tsx                    # Backoffice home
│   │   ├── users/
│   │   │   ├── page.tsx                # User list
│   │   │   ├── [id]/
│   │   │   └── new/
│   │   ├── monthly-closing/
│   │   ├── withdrawals/
│   │   ├── commissions/
│   │   └── logs/
│   ├── api/
│   │   └── [...]/
│   ├── layout.tsx
│   └── globals.css
├── components/
│   ├── ui/                             # shadcn/ui components
│   ├── dashboard/
│   ├── backoffice/
│   ├── forms/
│   └── shared/
├── lib/
│   ├── supabase/
│   │   ├── client.ts
│   │   ├── server.ts
│   │   └── admin.ts
│   ├── actions/
│   │   ├── auth.ts
│   │   ├── users.ts
│   │   ├── financial.ts
│   │   ├── withdrawals.ts
│   │   └── commissions.ts
│   ├── utils/
│   │   ├── formatting.ts
│   │   ├── calculations.ts
│   │   └── validation.ts
│   └── constants/
├── types/
│   ├── database.ts                     # Supabase generated types
│   ├── user.ts
│   ├── financial.ts
│   └── index.ts
├── hooks/
├── middleware.ts
├── supabase/
│   ├── migrations/
│   └── seed.sql
├── cypress/
│   ├── e2e/
│   └── support/
├── public/
├── .env.example
├── .env.local
├── next.config.js
├── tailwind.config.ts
├── tsconfig.json
└── package.json
```

### 3.4 Data Flow

1. **Client Request** → Browser sends request to Next.js application
2. **Middleware** → Validates JWT token, checks user role and permissions
3. **Server Component/Action** → Processes business logic server-side
4. **Supabase Client** → Executes database operations with RLS enforcement
5. **RLS Policies** → Database-level authorization ensures data isolation
6. **Response** → Data returned to client, rendered by React

**Key Principles:**
- All financial mutations go through Server Actions
- RLS policies enforce data access at database level
- Middleware validates session and role before routing
- No direct database access from client components

### 3.5 Database Schema

#### Users Table
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  auth_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  email TEXT UNIQUE NOT NULL,
  full_name TEXT NOT NULL,
  phone TEXT,

  -- User type (mutually exclusive)
  user_type TEXT NOT NULL CHECK (user_type IN ('client', 'agent', 'super_agent')),

  -- Operational roles (can have multiple)
  is_administrator BOOLEAN DEFAULT FALSE,
  is_operator BOOLEAN DEFAULT FALSE,

  -- Status
  status TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'unverified', 'blocked')),

  -- Financial data (for clients/agents/super_agents)
  initial_capital DECIMAL(15,2) DEFAULT 0,
  current_balance DECIMAL(15,2) DEFAULT 0,
  guaranteed_min_return DECIMAL(5,2) CHECK (guaranteed_min_return IN (2.5, 3.25, 4.25, 5.0)),
  returns_start_date DATE,

  -- Relationships
  assigned_agent_id UUID REFERENCES users(id),
  parent_super_agent_id UUID REFERENCES users(id),

  -- KYC preparation (future)
  kyc_status TEXT DEFAULT 'pending',
  kyc_provider TEXT,
  kyc_verified_at TIMESTAMPTZ,

  -- Timestamps
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),

  CONSTRAINT valid_agent_hierarchy CHECK (
    (user_type = 'client' AND parent_super_agent_id IS NULL) OR
    (user_type = 'agent' AND parent_super_agent_id IS NOT NULL) OR
    (user_type = 'super_agent')
  )
);
```

#### Invitations Table
```sql
CREATE TABLE invitations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  token TEXT UNIQUE NOT NULL,
  expires_at TIMESTAMPTZ NOT NULL,
  used_at TIMESTAMPTZ,
  created_by UUID REFERENCES users(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### Capital Movements Table
```sql
CREATE TABLE capital_movements (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  movement_type TEXT NOT NULL CHECK (movement_type IN ('initial_deposit', 'additional_deposit', 'withdrawal', 'return', 'commission_credit', 'commission_debit', 'adjustment')),
  amount DECIMAL(15,2) NOT NULL,
  balance_before DECIMAL(15,2) NOT NULL,
  balance_after DECIMAL(15,2) NOT NULL,
  reference_id UUID,  -- Links to withdrawal, monthly_closing, commission record
  description TEXT,
  executed_by UUID REFERENCES users(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### Monthly Closings Table
```sql
CREATE TABLE monthly_closings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  year INTEGER NOT NULL,
  month INTEGER NOT NULL CHECK (month BETWEEN 1 AND 12),
  return_percentage DECIMAL(5,2) NOT NULL,
  status TEXT NOT NULL DEFAULT 'draft' CHECK (status IN ('draft', 'executed')),
  executed_by UUID REFERENCES users(id),
  executed_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW(),

  UNIQUE(year, month)
);
```

#### Monthly Closing Items Table
```sql
CREATE TABLE monthly_closing_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  closing_id UUID REFERENCES monthly_closings(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  balance_before DECIMAL(15,2) NOT NULL,
  applied_percentage DECIMAL(5,2) NOT NULL,
  return_amount DECIMAL(15,2) NOT NULL,
  balance_after DECIMAL(15,2) NOT NULL,
  is_approved BOOLEAN DEFAULT TRUE,
  is_prorated BOOLEAN DEFAULT FALSE,  -- 50% for mid-month entries
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### Withdrawals Table
```sql
CREATE TABLE withdrawals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  amount DECIMAL(15,2) NOT NULL,
  withdrawal_type TEXT NOT NULL CHECK (withdrawal_type IN ('partial', 'total')),
  status TEXT NOT NULL DEFAULT 'requested' CHECK (status IN ('requested', 'approved', 'executed', 'rejected')),
  target_window_date DATE NOT NULL,

  -- Approval
  approved_by UUID REFERENCES users(id),
  approved_at TIMESTAMPTZ,

  -- Execution
  executed_by UUID REFERENCES users(id),
  executed_at TIMESTAMPTZ,

  -- Rejection
  rejected_by UUID REFERENCES users(id),
  rejected_at TIMESTAMPTZ,
  rejection_reason TEXT,

  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### Commissions Table
```sql
CREATE TABLE commissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  beneficiary_id UUID REFERENCES users(id) ON DELETE CASCADE,  -- Agent or Super Agent
  source_client_id UUID REFERENCES users(id) ON DELETE CASCADE,
  year INTEGER NOT NULL,
  month INTEGER NOT NULL CHECK (month BETWEEN 1 AND 12),
  base_balance DECIMAL(15,2) NOT NULL,
  commission_percentage DECIMAL(5,2) NOT NULL,
  commission_amount DECIMAL(15,2) NOT NULL,
  status TEXT NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'accrued', 'paid')),
  paid_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### Accumulated Commissions Table (for Agents)
```sql
CREATE TABLE accumulated_commissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_id UUID REFERENCES users(id) ON DELETE CASCADE,
  accumulated_amount DECIMAL(15,2) NOT NULL DEFAULT 0,
  last_payout_date DATE,
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### User Notes Table
```sql
CREATE TABLE user_notes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  content TEXT NOT NULL,
  created_by UUID REFERENCES users(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### Audit Logs Table
```sql
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  action TEXT NOT NULL,
  entity_type TEXT NOT NULL,
  entity_id UUID,
  old_values JSONB,
  new_values JSONB,
  ip_address INET,
  user_agent TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### Email Logs Table
```sql
CREATE TABLE email_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  recipient_id UUID REFERENCES users(id),
  recipient_email TEXT NOT NULL,
  email_type TEXT NOT NULL,
  subject TEXT NOT NULL,
  status TEXT NOT NULL DEFAULT 'sent',
  error_message TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 3.6 Authentication & Authorization

**Authentication Flow:**
1. Administrator creates user in Backoffice
2. System generates invitation token (24h expiry)
3. Email sent with activation link
4. User clicks link, sets password
5. Account activated, can login

**Authorization Layers:**

| Layer | Implementation |
|-------|----------------|
| Route | Next.js Middleware checks role before serving pages |
| Component | Server Components verify permissions before rendering |
| Action | Server Actions validate user role and ownership |
| Database | RLS policies enforce row-level access control |

**Role Permissions Matrix:**

| Action | Client | Agent | Super Agent | Operator | Administrator |
|--------|--------|-------|-------------|----------|---------------|
| View own dashboard | ✓ | ✓ | ✓ | - | - |
| View associated clients | - | ✓ | ✓ | - | - |
| Request withdrawal | ✓ | ✓ | ✓ | - | - |
| Access Backoffice | - | - | - | ✓ | ✓ |
| Create users | - | - | - | - | ✓ |
| Modify financial data | - | - | - | - | ✓ |
| Execute monthly closing | - | - | - | - | ✓ |
| Approve withdrawals | - | - | - | - | ✓ |
| Add internal notes | - | - | - | ✓ | ✓ |
| View audit logs | - | - | - | ✓ | ✓ |

**2FA Triggers:**
- Login from unrecognized device/location
- Withdrawal request submission
- Financial data modification
- Credential changes

### 3.7 API Design

All data mutations use Next.js Server Actions with the following pattern:

```typescript
// lib/actions/financial.ts
'use server'

import { createServerClient } from '@/lib/supabase/server'
import { revalidatePath } from 'next/cache'
import { z } from 'zod'

const executeClosingSchema = z.object({
  closingId: z.string().uuid(),
  approvedItems: z.array(z.string().uuid())
})

export async function executeMonthlyClosing(data: z.infer<typeof executeClosingSchema>) {
  const supabase = createServerClient()

  // Verify administrator role
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) throw new Error('Unauthorized')

  const { data: profile } = await supabase
    .from('users')
    .select('is_administrator')
    .eq('auth_id', user.id)
    .single()

  if (!profile?.is_administrator) throw new Error('Forbidden')

  // Execute closing logic...

  // Log action
  await logAuditAction('monthly_closing_executed', 'monthly_closings', data.closingId, ...)

  revalidatePath('/backoffice/monthly-closing')
  return { success: true }
}
```

**Key Server Actions:**

| Module | Actions |
|--------|---------|
| Auth | `sendInvitation`, `activateAccount`, `requestPasswordReset`, `verify2FA` |
| Users | `createUser`, `updateUser`, `blockUser`, `assignAgent`, `addNote` |
| Financial | `updateCapital`, `executeMonthlyClosing`, `adjustBalance` |
| Withdrawals | `requestWithdrawal`, `approveWithdrawal`, `executeWithdrawal`, `rejectWithdrawal` |
| Commissions | `calculateCommissions`, `payAgentCommissions`, `creditSuperAgentCommissions` |

### 3.8 Non-Functional Requirements

**Performance:**
- Dashboard load time < 2 seconds
- Server Action response < 500ms
- Database queries optimized with proper indexes

**Security:**
- HTTPS only
- JWT tokens with short expiry
- RLS policies on all tables
- Input validation with Zod
- CSRF protection via Server Actions
- Rate limiting on auth endpoints
- Encrypted backups

**Availability:**
- 99.9% uptime target
- Daily automated backups
- 30-90 day backup retention
- RPO: 24 hours
- RTO: 4 hours

**Maintainability:**
- TypeScript for type safety
- Comprehensive audit logging
- Modular component architecture
- Context7 for up-to-date library docs

**Accessibility:**
- Responsive design (desktop, tablet, mobile)
- WCAG 2.1 AA compliance target
- Semantic HTML structure

### 3.9 Assumptions & Open Questions

**Assumptions:**
- Supabase Auth handles JWT token management and refresh
- Resend handles email deliverability and bounce handling
- All financial values are in USD with 2 decimal precision
- Monthly closing is executed once per month (no partial closings)
- Commission percentages are fixed per agent/super-agent relationship
- Bank account details for withdrawals are managed externally

**Open Questions:**
1. What are the specific commission percentages for Agents and Super Agents?
2. Should there be email notifications for pending withdrawal requests to administrators?
3. Is there a maximum withdrawal amount limit?
4. How should the system handle a client who becomes an Agent (type change)?
5. What is the exact 2FA implementation preference (TOTP app vs email OTP)?
6. Should blocked users see a specific message explaining their status?
7. What is the retention period for audit logs?
8. Are there specific requirements for the balance evolution chart (time range, granularity)?

---

## Style Guidelines

- Clear, concise, and technical
- No marketing language
- Written for developers
- Markdown only
