# UI Builder Agent

Invocable agent during development sessions to implement functional UI combining Figma designs + existing components + business logic.

## Invocation

From a session, invoke with Task tool:

```
Task: UI Builder Agent
Prompt: |
  Figma: [URL or node-id]
  Target: [what to implement - section, page, component]
  Route: [destination route, e.g., /dashboard, /backoffice/users]
  Functionality: |
    - [required functionality description]
    - [server actions needed]
    - [data fetching required]
    - [validations]
  Context: [any relevant projectSpec context]
```

---

## Agent Workflow

### Step 1: Get Figma Design

Use Figma MCP to extract the design:

```
mcp__figma__get_design_context or mcp__figma_desktop__get_design_context
```

Extract:
- Layout structure
- UI elements and their states
- Colors, spacing, typography used
- Required images/assets

### Step 2: Inventory Existing Components

Scan the project to know what's available:

```bash
# Installed shadcn components
ls components/ui/

# Custom project components
ls components/

# Styleguide (if exists)
ls app/styleguide/components/
```

Create list of:
- Already installed shadcn components
- Already created custom components
- Missing components needed

### Step 3: Install Missing Components

If shadcn components are missing:

1. Verify availability with Context7 MCP:
```
mcp__context7__resolve-library-id: shadcn
mcp__context7__query-docs: [specific component]
```

2. Install:
```bash
npx shadcn@latest add [component1] [component2] ...
```

### Step 4: Map Design to Components

Create visual → code mapping:

| Figma Element | Component | Notes |
|---------------|-----------|-------|
| Header card | Card + CardHeader | With metrics |
| Data table | Table | With sorting |
| Action button | Button (variant="default") | With loading state |
| Form field | Input + Label | With error state |
| Status indicator | Badge | variant by status |

### Step 5: Implement with Functionality

#### 5.1 Create Types (if not existing)

```typescript
// types/[feature].ts
export interface [EntityName] {
  id: string
  // ... DB fields
}

export interface [ActionInput] {
  // ... form/action input
}
```

#### 5.2 Create Server Actions (if required)

```typescript
// lib/actions/[feature].ts
'use server'

import { z } from 'zod'
import { createServerClient } from '@/lib/supabase/server'
import { revalidatePath } from 'next/cache'

const schema = z.object({
  // validation
})

export async function actionName(input: z.infer<typeof schema>) {
  const supabase = createServerClient()

  // 1. Auth check
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) throw new Error('Unauthorized')

  // 2. Validate
  const validated = schema.parse(input)

  // 3. Execute
  const { data, error } = await supabase
    .from('table')
    .insert(validated)
    .select()
    .single()

  if (error) throw error

  // 4. Revalidate
  revalidatePath('/route')

  return { success: true, data }
}
```

#### 5.3 Create Component/Page

**For Server Component (default):**
```typescript
// app/[route]/page.tsx
import { createServerClient } from '@/lib/supabase/server'
import { redirect } from 'next/navigation'

// UI Components
import { Card, CardHeader, CardContent } from '@/components/ui/card'
import { Button } from '@/components/ui/button'

// Page-specific components
import { DataTable } from './_components/DataTable'
import { ActionForm } from './_components/ActionForm'

export default async function PageName() {
  const supabase = createServerClient()

  // Auth
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) redirect('/login')

  // Data fetching
  const { data } = await supabase
    .from('table')
    .select('*')

  return (
    <div className="flex flex-col gap-6">
      {/* Implementation following Figma design */}
    </div>
  )
}
```

**For Client Component (when interactivity needed):**
```typescript
// app/[route]/_components/InteractiveSection.tsx
'use client'

import { useState } from 'react'
import { useActionState } from 'react'
import { actionName } from '@/lib/actions/feature'
import { Button } from '@/components/ui/button'
import { toast } from 'sonner'

export function InteractiveSection({ initialData }) {
  const [state, formAction, pending] = useActionState(actionName, null)

  return (
    <form action={formAction}>
      {/* Form fields */}
      <Button type="submit" disabled={pending}>
        {pending ? 'Processing...' : 'Submit'}
      </Button>
    </form>
  )
}
```

### Step 6: Apply Design System Styles

Use CSS variables defined in globals.css:

```typescript
// Backgrounds
className="bg-background"      // main background
className="bg-card"            // card background
className="bg-muted"           // secondary background
className="bg-primary"         // primary color

// Text
className="text-foreground"         // main text
className="text-muted-foreground"   // secondary text
className="text-primary"            // accent text

// Borders
className="border-border"      // standard borders
className="rounded-lg"         // system border radius

// Spacing (use Tailwind scale)
className="p-4 gap-6 space-y-4"
```

### Step 7: UI States

Implement all necessary states:

```typescript
// Loading
{isLoading && <Skeleton className="h-10 w-full" />}

// Empty
{data.length === 0 && (
  <div className="text-center py-12 text-muted-foreground">
    No data available
  </div>
)}

// Error
{error && (
  <Alert variant="destructive">
    <AlertDescription>{error.message}</AlertDescription>
  </Alert>
)}
```

---

## Agent Output

Upon completion, the agent should have created/modified:

1. **Type files** (if new ones needed)
2. **Server Actions** (if functionality required)
3. **Page components** in `_components/`
4. **Main page.tsx**
5. **Loading.tsx** and **Error.tsx** if applicable

And report:
- Installed shadcn components
- Created/modified files
- Implemented functionality
- Any pending TODOs

---

## Invocation Example

```
Task: UI Builder Agent
Prompt: |
  Figma: https://figma.com/design/abc123?node-id=10-200
  Target: User creation form
  Route: /backoffice/users/new
  Functionality: |
    - Server action createUser that inserts into users table
    - Zod validation: email required, full_name required, user_type enum
    - user_type selector (client, agent, super_agent)
    - Assigned agent selector (only if user_type is client)
    - Send invitation email on create
    - Redirect to /backoffice/users after creation
  Context: |
    See projectSpec/projectSpec.md section 3.5 for users schema
    See section 3.6 for invitation flow
```

---

## Important Notes

- **Always use existing components** - don't recreate what already exists
- **Server Components by default** - 'use client' only when necessary
- **Follow design system** - use CSS variables, not hardcoded values
- **Validate with Zod** - all user input
- **RLS applies automatically** - don't duplicate authorization logic in queries
- **Accessibility** - labels, aria attributes, keyboard navigation
