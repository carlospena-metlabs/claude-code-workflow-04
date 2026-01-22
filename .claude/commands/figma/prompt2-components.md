# Prompt 2: Component Development

Build or customize a component using shadcn/ui, optionally from a Figma design.

## Input

Component name is required. Figma URL is optional.

**Example invocations:**
```bash
# From shadcn registry (no Figma needed)
/figma/prompt2-components Button

# From Figma design
/figma/prompt2-components Sidebar https://figma.com/design/abc123?node-id=1-200

# Custom component from Figma
/figma/prompt2-components MetricCard https://figma.com/design/abc123?node-id=5-100
```

---

## Prerequisites

- **prompt1-tokens must be completed first**
- Design system tokens configured in `src/index.css`
- `tailwind.config.js` with semantic colors
- shadcn initialized in project

---

## Workflow

### Step 1: Determine Component Source

**Option A: Figma URL provided**
→ Extract design from Figma, then check if shadcn has a base component

**Option B: No Figma URL**
→ Use shadcn component directly or build custom

### Step 2: Extract from Figma (if URL provided)

Use Figma MCP to get the component design:

```
mcp__plugin_figma_figma__get_design_context
  fileKey: [extracted from URL]
  nodeId: [extracted from URL]
```

For visual reference:
```
mcp__plugin_figma_figma__get_screenshot
  fileKey: [extracted from URL]
  nodeId: [extracted from URL]
```

Analyze:
- Component structure and hierarchy
- Variants (sizes, states, colors)
- Interactive states (hover, focus, disabled)
- Spacing and dimensions
- Icons or images used

### Step 3: Check shadcn Registry

Search for existing shadcn component:

```bash
# Check if component exists in your project
ls src/components/ui/

# Search shadcn docs for component
# Use Context7 MCP for documentation:
mcp__plugin_context7_context7__resolve-library-id
  libraryName: "shadcn"
  query: "[component name]"
```

**Common shadcn components:**
| Category | Components |
|----------|------------|
| Layout | Card, Separator, Tabs, Accordion, Collapsible, Sheet |
| Forms | Button, Input, Select, Checkbox, Radio, Switch, Textarea, Label, Form |
| Feedback | Alert, Toast, Progress, Skeleton, Badge |
| Overlay | Dialog, Drawer, Popover, Tooltip, Dropdown Menu, Alert Dialog |
| Navigation | Navigation Menu, Breadcrumb, Pagination, Command, Sidebar |
| Data | Table, Data Table, Calendar, Chart |

**Decision tree:**
```
Component exists in shadcn?
├── YES → Install and customize if needed (Step 4)
└── NO → Build custom component (Step 5)
```

### Step 4: Install & Customize shadcn Component

**Install:**
```bash
npx shadcn@latest add [component-name]
```

This adds the component to `src/components/ui/`.

**Review the installed component:**
- Available variants
- Props interface
- How it uses CSS variables

**Customize if needed** - Create a wrapped version in `src/components/[ComponentName].tsx`:

```tsx
import { Button } from '@/components/ui/button';
import { cn } from '@/lib/utils';

interface CustomButtonProps extends React.ComponentProps<typeof Button> {
  intent?: 'default' | 'success' | 'warning' | 'info';
}

export function CustomButton({
  intent = 'default',
  className,
  ...props
}: CustomButtonProps) {
  return (
    <Button
      className={cn(
        intent === 'success' && 'bg-success text-success-foreground hover:bg-success/90',
        intent === 'warning' && 'bg-warning text-warning-foreground hover:bg-warning/90',
        intent === 'info' && 'bg-info text-info-foreground hover:bg-info/90',
        className
      )}
      {...props}
    />
  );
}
```

**Customization patterns:**
- Add semantic color variants using CSS variables (`bg-success`, `text-warning`)
- Add new size variants
- Compose multiple shadcn components
- Add loading states, icons, or animations

### Step 5: Build Custom Component (if not in shadcn)

If shadcn doesn't have this component, build it using:
- shadcn primitives as building blocks
- CSS variables via Tailwind classes
- shadcn's patterns for consistency

```tsx
// src/components/MetricCard.tsx
import { cn } from '@/lib/utils';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { TrendingUp, TrendingDown } from 'lucide-react';

interface MetricCardProps {
  title: string;
  value: string | number;
  change?: number;
  changeLabel?: string;
  className?: string;
}

export function MetricCard({
  title,
  value,
  change,
  changeLabel,
  className,
}: MetricCardProps) {
  const isPositive = change && change > 0;
  const isNegative = change && change < 0;

  return (
    <Card className={cn('', className)}>
      <CardHeader className="pb-2">
        <CardTitle className="text-sm font-medium text-muted-foreground">
          {title}
        </CardTitle>
      </CardHeader>
      <CardContent>
        <div className="text-2xl font-bold">{value}</div>
        {change !== undefined && (
          <div className={cn(
            'flex items-center gap-1 text-sm mt-1',
            isPositive && 'text-success',
            isNegative && 'text-destructive',
          )}>
            {isPositive && <TrendingUp className="h-4 w-4" />}
            {isNegative && <TrendingDown className="h-4 w-4" />}
            <span>{Math.abs(change)}%</span>
            {changeLabel && <span className="text-muted-foreground">{changeLabel}</span>}
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```

### Step 6: Create Component Showcase

Add showcase page in `src/pages/styleguide/components/[component-name].tsx`:

```tsx
// src/pages/styleguide/components/metric-card.tsx
import { MetricCard } from '@/components/MetricCard';

export default function MetricCardShowcase() {
  return (
    <div className="p-8 space-y-8">
      <div>
        <h1 className="text-3xl font-bold">MetricCard</h1>
        <p className="text-muted-foreground mt-2">
          Display key metrics with optional trend indicators.
        </p>
      </div>

      {/* Variants */}
      <section>
        <h2 className="text-xl font-semibold mb-4">Variants</h2>
        <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
          <MetricCard title="Total Users" value="12,345" />
          <MetricCard title="Revenue" value="$45,678" change={12.5} changeLabel="vs last month" />
          <MetricCard title="Bounce Rate" value="23%" change={-5.2} changeLabel="vs last week" />
        </div>
      </section>

      {/* Usage */}
      <section>
        <h2 className="text-xl font-semibold mb-4">Usage</h2>
        <pre className="bg-muted p-4 rounded-lg overflow-x-auto">
          <code>{`import { MetricCard } from '@/components/MetricCard';

<MetricCard
  title="Total Users"
  value="12,345"
  change={12.5}
  changeLabel="vs last month"
/>`}</code>
        </pre>
      </section>

      {/* Props */}
      <section>
        <h2 className="text-xl font-semibold mb-4">Props</h2>
        <table className="w-full text-sm">
          <thead>
            <tr className="border-b">
              <th className="text-left py-2">Prop</th>
              <th className="text-left py-2">Type</th>
              <th className="text-left py-2">Default</th>
              <th className="text-left py-2">Description</th>
            </tr>
          </thead>
          <tbody>
            <tr className="border-b">
              <td className="py-2 font-mono">title</td>
              <td className="py-2">string</td>
              <td className="py-2">-</td>
              <td className="py-2">Metric label</td>
            </tr>
            <tr className="border-b">
              <td className="py-2 font-mono">value</td>
              <td className="py-2">string | number</td>
              <td className="py-2">-</td>
              <td className="py-2">Main metric value</td>
            </tr>
            <tr className="border-b">
              <td className="py-2 font-mono">change</td>
              <td className="py-2">number</td>
              <td className="py-2">undefined</td>
              <td className="py-2">Percentage change (positive/negative)</td>
            </tr>
            <tr className="border-b">
              <td className="py-2 font-mono">changeLabel</td>
              <td className="py-2">string</td>
              <td className="py-2">undefined</td>
              <td className="py-2">Label for the change period</td>
            </tr>
          </tbody>
        </table>
      </section>
    </div>
  );
}
```

### Step 7: Add Route for Showcase

In your router configuration:

```tsx
// src/routes.tsx
import MetricCardShowcase from '@/pages/styleguide/components/metric-card';

// Add to routes:
{
  path: '/styleguide/components/metric-card',
  element: <MetricCardShowcase />,
}
```

### Step 8: Update Styleguide Navigation (optional)

If you have a styleguide navigation config, add the new component:

```tsx
// src/pages/styleguide/navigation.ts
export const componentRoutes = [
  { name: 'Button', path: '/styleguide/components/button' },
  { name: 'Card', path: '/styleguide/components/card' },
  { name: 'MetricCard', path: '/styleguide/components/metric-card' }, // NEW
];
```

---

## Directory Structure

```
src/
├── components/
│   ├── ui/                      # Base shadcn components (auto-generated)
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   └── ...
│   ├── MetricCard.tsx           # Custom components
│   └── CustomButton.tsx         # Wrapped/extended shadcn components
│
└── pages/
    └── styleguide/
        ├── index.tsx            # Main styleguide (from prompt1)
        └── components/
            ├── button.tsx       # Button showcase
            ├── card.tsx         # Card showcase
            └── metric-card.tsx  # Custom component showcase
```

---

## Output

After running this prompt:

- Component installed/created in `src/components/` or `src/components/ui/`
- Showcase page in `src/pages/styleguide/components/[name].tsx`
- Route added for showcase
- Component documented with:
  - All variants displayed
  - Usage code examples
  - Props table
  - Interactive states (if applicable)

---

## Notes

- **CSS variables are the source of truth** - defined in `src/index.css`
- **Tailwind classes reference CSS variables** - `bg-primary`, `text-muted-foreground`
- **Extend, don't rebuild** - customize shadcn components rather than building from scratch
- **Figma is optional** - for custom designs. Use shadcn directly when possible.
- **Always check existing components** - run `ls src/components/ui/` before creating new ones
