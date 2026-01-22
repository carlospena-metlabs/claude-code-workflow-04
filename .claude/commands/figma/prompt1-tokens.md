# Prompt 1: Design System Setup

Extract design tokens from Figma and set up a complete design system for **Web (Vite + React)** and **Mobile (React Native)**.

## Input

Figma URL or node-id is required. Pass it when invoking this command.

Example invocation:
```
/figma/prompt1-tokens https://figma.com/design/abc123?node-id=0-1
```

---

## Output Files

This prompt generates a **single source of truth** that syncs Web and Mobile:

```
.claude/docs/design-tokens.json      ← Source of truth (JSON)
        │
        ├── proyecto-web/src/index.css           ← Web CSS variables
        ├── proyecto-web/tailwind.config.js      ← Tailwind config
        └── proyecto-mobile/src/theme/tokens.ts  ← Mobile theme (generated)
```

---

## Prerequisites

- Web project initialized with Vite + React + TypeScript
- TailwindCSS installed
- Project structure: `src/` (not `/app/`)

---

## Workflow

### Step 1: Extract Design from Figma

Use Figma MCP to get the design context:

```
mcp__plugin_figma_figma__get_design_context
  fileKey: [extracted from URL]
  nodeId: [extracted from URL]
```

If you need a visual reference:
```
mcp__plugin_figma_figma__get_screenshot
  fileKey: [extracted from URL]
  nodeId: [extracted from URL]
```

For design variables (colors, spacing):
```
mcp__plugin_figma_figma__get_variable_defs
  fileKey: [extracted from URL]
  nodeId: [extracted from URL]
```

### Step 2: Analyze the Design

From the Figma data, identify/infer:

**Colors:**
- Primary/brand color → generate full scale (50-950)
- Neutral/grey colors → generate full scale (50-950)
- Semantic colors (success, error, warning, info)
- Background and surface colors
- Border colors

**Typography:**
- Font family (sans-serif, serif, monospace)
- Heading sizes and weights
- Body text sizes

**Spacing & Radius:**
- Spacing rhythm (tight, normal, relaxed)
- Border radius style (sharp, rounded, pill)

**Shadows:**
- Shadow style (none, subtle, prominent)

### Step 3: Generate design-tokens.json (Source of Truth)

Create `.claude/docs/design-tokens.json` with all extracted values:

```json
{
  "colors": {
    "primary": {
      "hex": "#3B82F6",
      "hsl": "217 91% 60%",
      "rgb": "59, 130, 246"
    },
    "primaryForeground": {
      "hex": "#FFFFFF",
      "hsl": "0 0% 100%",
      "rgb": "255, 255, 255"
    },
    "secondary": {
      "hex": "#F1F5F9",
      "hsl": "210 40% 96%",
      "rgb": "241, 245, 249"
    },
    "secondaryForeground": {
      "hex": "#0F172A",
      "hsl": "222 47% 11%",
      "rgb": "15, 23, 42"
    },
    "background": {
      "hex": "#FFFFFF",
      "hsl": "0 0% 100%",
      "rgb": "255, 255, 255"
    },
    "foreground": {
      "hex": "#0F172A",
      "hsl": "222 47% 11%",
      "rgb": "15, 23, 42"
    },
    "card": {
      "hex": "#FFFFFF",
      "hsl": "0 0% 100%",
      "rgb": "255, 255, 255"
    },
    "cardForeground": {
      "hex": "#0F172A",
      "hsl": "222 47% 11%",
      "rgb": "15, 23, 42"
    },
    "muted": {
      "hex": "#F1F5F9",
      "hsl": "210 40% 96%",
      "rgb": "241, 245, 249"
    },
    "mutedForeground": {
      "hex": "#64748B",
      "hsl": "215 16% 47%",
      "rgb": "100, 116, 139"
    },
    "destructive": {
      "hex": "#EF4444",
      "hsl": "0 84% 60%",
      "rgb": "239, 68, 68"
    },
    "destructiveForeground": {
      "hex": "#FFFFFF",
      "hsl": "0 0% 100%",
      "rgb": "255, 255, 255"
    },
    "border": {
      "hex": "#E2E8F0",
      "hsl": "214 32% 91%",
      "rgb": "226, 232, 240"
    },
    "ring": {
      "hex": "#3B82F6",
      "hsl": "217 91% 60%",
      "rgb": "59, 130, 246"
    },
    "success": {
      "hex": "#22C55E",
      "hsl": "142 71% 45%",
      "rgb": "34, 197, 94"
    },
    "successForeground": {
      "hex": "#FFFFFF",
      "hsl": "0 0% 100%",
      "rgb": "255, 255, 255"
    },
    "warning": {
      "hex": "#F59E0B",
      "hsl": "38 92% 50%",
      "rgb": "245, 158, 11"
    },
    "warningForeground": {
      "hex": "#0F172A",
      "hsl": "222 47% 11%",
      "rgb": "15, 23, 42"
    },
    "info": {
      "hex": "#0EA5E9",
      "hsl": "199 89% 48%",
      "rgb": "14, 165, 233"
    },
    "infoForeground": {
      "hex": "#FFFFFF",
      "hsl": "0 0% 100%",
      "rgb": "255, 255, 255"
    }
  },
  "colorsDark": {
    "background": {
      "hex": "#0F172A",
      "hsl": "222 47% 11%",
      "rgb": "15, 23, 42"
    },
    "foreground": {
      "hex": "#F8FAFC",
      "hsl": "210 40% 98%",
      "rgb": "248, 250, 252"
    },
    "card": {
      "hex": "#1E293B",
      "hsl": "217 33% 17%",
      "rgb": "30, 41, 59"
    },
    "cardForeground": {
      "hex": "#F8FAFC",
      "hsl": "210 40% 98%",
      "rgb": "248, 250, 252"
    },
    "muted": {
      "hex": "#1E293B",
      "hsl": "217 33% 17%",
      "rgb": "30, 41, 59"
    },
    "mutedForeground": {
      "hex": "#94A3B8",
      "hsl": "215 20% 65%",
      "rgb": "148, 163, 184"
    },
    "border": {
      "hex": "#334155",
      "hsl": "217 19% 27%",
      "rgb": "51, 65, 85"
    }
  },
  "typography": {
    "fontFamily": "Inter",
    "fontFamilyFallback": "system-ui, sans-serif"
  },
  "spacing": {
    "xs": 4,
    "sm": 8,
    "md": 16,
    "lg": 24,
    "xl": 32,
    "2xl": 48
  },
  "borderRadius": {
    "sm": 4,
    "md": 6,
    "lg": 8,
    "xl": 12,
    "full": 9999
  },
  "shadows": {
    "sm": "0 1px 2px 0 rgb(0 0 0 / 0.05)",
    "md": "0 4px 6px -1px rgb(0 0 0 / 0.1)",
    "lg": "0 10px 15px -3px rgb(0 0 0 / 0.1)"
  }
}
```

**This JSON is the single source of truth.** All platform-specific files are generated from it.

### Step 4: Initialize shadcn (Web)

```bash
npx shadcn@latest init
```

When prompted:
- Style: Default
- Base color: Neutral (we'll override with our tokens)
- CSS variables: Yes

### Step 5: Generate globals.css (Web)

Replace `src/index.css` (or `src/globals.css`) with extracted design tokens:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    /* === BASE === */
    --background: [extracted - e.g., 0 0% 100%];
    --foreground: [extracted - e.g., 222 47% 11%];

    /* === CARD === */
    --card: [extracted card background];
    --card-foreground: [extracted card text];

    /* === POPOVER / DROPDOWN / TOOLTIP === */
    --popover: [same as card or white];
    --popover-foreground: [same as card-foreground];

    /* === PRIMARY (main brand color) === */
    --primary: [extracted primary - e.g., 221 83% 53%];
    --primary-foreground: [white or dark based on contrast];

    /* === SECONDARY === */
    --secondary: [light grey or muted version];
    --secondary-foreground: [dark text];

    /* === MUTED === */
    --muted: [light grey background];
    --muted-foreground: [medium grey text];

    /* === ACCENT === */
    --accent: [same as secondary or slight tint];
    --accent-foreground: [dark text];

    /* === DESTRUCTIVE === */
    --destructive: [red/error color - e.g., 0 84% 60%];
    --destructive-foreground: [white];

    /* === BORDERS & INPUTS === */
    --border: [extracted border color];
    --input: [slightly darker border for inputs];
    --ring: [primary color for focus rings];

    /* === BORDER RADIUS === */
    --radius: [extracted radius, e.g., 0.5rem];

    /* === CHART COLORS === */
    --chart-1: [primary];
    --chart-2: [complementary color];
    --chart-3: [variation];
    --chart-4: [variation];
    --chart-5: [variation];

    /* === SIDEBAR === */
    --sidebar-background: [sidebar background];
    --sidebar-foreground: [sidebar text];
    --sidebar-primary: [primary];
    --sidebar-primary-foreground: [white];
    --sidebar-accent: [accent];
    --sidebar-accent-foreground: [dark text];
    --sidebar-border: [border color];
    --sidebar-ring: [primary];

    /* === CUSTOM SEMANTIC COLORS === */
    --success: [green - e.g., 142 76% 36%];
    --success-foreground: [white];
    --warning: [yellow/orange - e.g., 38 92% 50%];
    --warning-foreground: [dark for contrast];
    --info: [blue - e.g., 199 89% 48%];
    --info-foreground: [white];
  }

  .dark {
    /* === BASE === */
    --background: [dark background - e.g., 222 47% 11%];
    --foreground: [light text - e.g., 210 40% 98%];

    /* === CARD === */
    --card: [dark card - e.g., 222 47% 15%];
    --card-foreground: [light text];

    /* === POPOVER === */
    --popover: [same as dark card];
    --popover-foreground: [light text];

    /* === PRIMARY === */
    --primary: [same or slightly adjusted];
    --primary-foreground: [adjusted for dark];

    /* === SECONDARY === */
    --secondary: [dark secondary];
    --secondary-foreground: [light text];

    /* === MUTED === */
    --muted: [dark muted];
    --muted-foreground: [medium grey];

    /* === ACCENT === */
    --accent: [dark accent];
    --accent-foreground: [light text];

    /* === DESTRUCTIVE === */
    --destructive: [same red, maybe brighter];
    --destructive-foreground: [white];

    /* === BORDERS & INPUTS === */
    --border: [dark border];
    --input: [dark input border];
    --ring: [primary];

    /* === SIDEBAR === */
    --sidebar-background: [dark sidebar];
    --sidebar-foreground: [light text];
    --sidebar-primary: [primary];
    --sidebar-primary-foreground: [white];
    --sidebar-accent: [dark accent];
    --sidebar-accent-foreground: [light text];
    --sidebar-border: [dark border];
    --sidebar-ring: [primary];

    /* === SEMANTIC (same or adjusted) === */
    --success: [green];
    --success-foreground: [white];
    --warning: [yellow/orange];
    --warning-foreground: [dark];
    --info: [blue];
    --info-foreground: [white];
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}
```

**Note:** shadcn uses HSL values without the `hsl()` wrapper. Convert hex to HSL and use format: `H S% L%`

### Step 6: Install Google Font (Web - if applicable)

If a Google Font matches the design, add it to `index.html`:

```html
<!-- public/index.html or index.html -->
<head>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
</head>
```

And update `tailwind.config.js`:

```js
/** @type {import('tailwindcss').Config} */
export default {
  // ...
  theme: {
    extend: {
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
      },
    },
  },
}
```

### Step 7: Extend Tailwind for Semantic Colors (Web)

Update `tailwind.config.js` to include semantic colors:

```js
/** @type {import('tailwindcss').Config} */
export default {
  darkMode: ["class"],
  content: ["./index.html", "./src/**/*.{js,ts,jsx,tsx}"],
  theme: {
    extend: {
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        secondary: {
          DEFAULT: "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        destructive: {
          DEFAULT: "hsl(var(--destructive))",
          foreground: "hsl(var(--destructive-foreground))",
        },
        muted: {
          DEFAULT: "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        accent: {
          DEFAULT: "hsl(var(--accent))",
          foreground: "hsl(var(--accent-foreground))",
        },
        popover: {
          DEFAULT: "hsl(var(--popover))",
          foreground: "hsl(var(--popover-foreground))",
        },
        card: {
          DEFAULT: "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
        // Semantic colors
        success: {
          DEFAULT: "hsl(var(--success))",
          foreground: "hsl(var(--success-foreground))",
        },
        warning: {
          DEFAULT: "hsl(var(--warning))",
          foreground: "hsl(var(--warning-foreground))",
        },
        info: {
          DEFAULT: "hsl(var(--info))",
          foreground: "hsl(var(--info-foreground))",
        },
        sidebar: {
          DEFAULT: "hsl(var(--sidebar-background))",
          foreground: "hsl(var(--sidebar-foreground))",
          primary: "hsl(var(--sidebar-primary))",
          "primary-foreground": "hsl(var(--sidebar-primary-foreground))",
          accent: "hsl(var(--sidebar-accent))",
          "accent-foreground": "hsl(var(--sidebar-accent-foreground))",
          border: "hsl(var(--sidebar-border))",
          ring: "hsl(var(--sidebar-ring))",
        },
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
    },
  },
  plugins: [require("tailwindcss-animate")],
}
```

### Step 8: Install Demo Components (Web)

```bash
npx shadcn@latest add button card badge alert input label
```

### Step 9: Create Styleguide Route (Web)

Create the styleguide page structure for Vite + React Router:

**Create `src/pages/styleguide/index.tsx`:**

```tsx
import { useState } from 'react';
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';
import { Alert, AlertDescription, AlertTitle } from '@/components/ui/alert';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Moon, Sun } from 'lucide-react';

const colorTokens = [
  { name: 'background', var: '--background' },
  { name: 'foreground', var: '--foreground' },
  { name: 'card', var: '--card' },
  { name: 'card-foreground', var: '--card-foreground' },
  { name: 'primary', var: '--primary' },
  { name: 'primary-foreground', var: '--primary-foreground' },
  { name: 'secondary', var: '--secondary' },
  { name: 'secondary-foreground', var: '--secondary-foreground' },
  { name: 'muted', var: '--muted' },
  { name: 'muted-foreground', var: '--muted-foreground' },
  { name: 'accent', var: '--accent' },
  { name: 'accent-foreground', var: '--accent-foreground' },
  { name: 'destructive', var: '--destructive' },
  { name: 'border', var: '--border' },
  { name: 'input', var: '--input' },
  { name: 'ring', var: '--ring' },
];

const semanticColors = [
  { name: 'success', var: '--success' },
  { name: 'warning', var: '--warning' },
  { name: 'info', var: '--info' },
];

export default function StyleguidePage() {
  const [isDark, setIsDark] = useState(false);

  const toggleTheme = () => {
    setIsDark(!isDark);
    document.documentElement.classList.toggle('dark');
  };

  return (
    <div className="min-h-screen bg-background text-foreground p-8">
      <div className="max-w-6xl mx-auto space-y-12">
        {/* Header */}
        <div className="flex justify-between items-center">
          <div>
            <h1 className="text-4xl font-bold">Design System</h1>
            <p className="text-muted-foreground mt-2">
              Design tokens and component library
            </p>
          </div>
          <Button variant="outline" size="icon" onClick={toggleTheme}>
            {isDark ? <Sun className="h-4 w-4" /> : <Moon className="h-4 w-4" />}
          </Button>
        </div>

        {/* Color Tokens */}
        <section>
          <h2 className="text-2xl font-semibold mb-6">Color Tokens</h2>
          <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
            {colorTokens.map((token) => (
              <div key={token.name} className="space-y-2">
                <div
                  className="h-16 rounded-lg border"
                  style={{ backgroundColor: `hsl(var(${token.var}))` }}
                />
                <p className="text-sm font-medium">{token.name}</p>
                <code className="text-xs text-muted-foreground">{token.var}</code>
              </div>
            ))}
          </div>
        </section>

        {/* Semantic Colors */}
        <section>
          <h2 className="text-2xl font-semibold mb-6">Semantic Colors</h2>
          <div className="grid grid-cols-3 gap-4">
            {semanticColors.map((token) => (
              <div key={token.name} className="space-y-2">
                <div
                  className="h-16 rounded-lg"
                  style={{ backgroundColor: `hsl(var(${token.var}))` }}
                />
                <p className="text-sm font-medium">{token.name}</p>
                <code className="text-xs text-muted-foreground">{token.var}</code>
              </div>
            ))}
          </div>
        </section>

        {/* Typography */}
        <section>
          <h2 className="text-2xl font-semibold mb-6">Typography</h2>
          <Card>
            <CardContent className="pt-6 space-y-4">
              <h1 className="text-4xl font-bold">Heading 1</h1>
              <h2 className="text-3xl font-semibold">Heading 2</h2>
              <h3 className="text-2xl font-semibold">Heading 3</h3>
              <h4 className="text-xl font-medium">Heading 4</h4>
              <p className="text-base">Body text - The quick brown fox jumps over the lazy dog.</p>
              <p className="text-sm text-muted-foreground">Small text - Secondary information</p>
              <code className="text-sm bg-muted px-2 py-1 rounded">Code text</code>
            </CardContent>
          </Card>
        </section>

        {/* Buttons */}
        <section>
          <h2 className="text-2xl font-semibold mb-6">Buttons</h2>
          <Card>
            <CardContent className="pt-6">
              <div className="flex flex-wrap gap-4">
                <Button>Default</Button>
                <Button variant="secondary">Secondary</Button>
                <Button variant="destructive">Destructive</Button>
                <Button variant="outline">Outline</Button>
                <Button variant="ghost">Ghost</Button>
                <Button variant="link">Link</Button>
              </div>
              <div className="flex flex-wrap gap-4 mt-4">
                <Button size="sm">Small</Button>
                <Button size="default">Default</Button>
                <Button size="lg">Large</Button>
              </div>
              <div className="flex flex-wrap gap-4 mt-4">
                <Button disabled>Disabled</Button>
                <Button className="bg-success hover:bg-success/90">Success</Button>
                <Button className="bg-warning hover:bg-warning/90 text-warning-foreground">Warning</Button>
                <Button className="bg-info hover:bg-info/90">Info</Button>
              </div>
            </CardContent>
          </Card>
        </section>

        {/* Cards */}
        <section>
          <h2 className="text-2xl font-semibold mb-6">Cards</h2>
          <div className="grid md:grid-cols-2 gap-4">
            <Card>
              <CardHeader>
                <CardTitle>Card Title</CardTitle>
              </CardHeader>
              <CardContent>
                <p>Card content goes here. This is an example of a basic card component.</p>
              </CardContent>
            </Card>
            <Card className="border-primary">
              <CardHeader>
                <CardTitle>Highlighted Card</CardTitle>
              </CardHeader>
              <CardContent>
                <p>A card with primary border for emphasis.</p>
              </CardContent>
            </Card>
          </div>
        </section>

        {/* Badges */}
        <section>
          <h2 className="text-2xl font-semibold mb-6">Badges</h2>
          <Card>
            <CardContent className="pt-6">
              <div className="flex flex-wrap gap-4">
                <Badge>Default</Badge>
                <Badge variant="secondary">Secondary</Badge>
                <Badge variant="destructive">Destructive</Badge>
                <Badge variant="outline">Outline</Badge>
                <Badge className="bg-success">Success</Badge>
                <Badge className="bg-warning text-warning-foreground">Warning</Badge>
                <Badge className="bg-info">Info</Badge>
              </div>
            </CardContent>
          </Card>
        </section>

        {/* Alerts */}
        <section>
          <h2 className="text-2xl font-semibold mb-6">Alerts</h2>
          <div className="space-y-4">
            <Alert>
              <AlertTitle>Default Alert</AlertTitle>
              <AlertDescription>This is a default alert message.</AlertDescription>
            </Alert>
            <Alert variant="destructive">
              <AlertTitle>Error Alert</AlertTitle>
              <AlertDescription>Something went wrong. Please try again.</AlertDescription>
            </Alert>
          </div>
        </section>

        {/* Form Elements */}
        <section>
          <h2 className="text-2xl font-semibold mb-6">Form Elements</h2>
          <Card>
            <CardContent className="pt-6 space-y-4 max-w-md">
              <div className="space-y-2">
                <Label htmlFor="email">Email</Label>
                <Input id="email" type="email" placeholder="Enter your email" />
              </div>
              <div className="space-y-2">
                <Label htmlFor="disabled">Disabled Input</Label>
                <Input id="disabled" disabled placeholder="Disabled" />
              </div>
            </CardContent>
          </Card>
        </section>

        {/* Border Radius */}
        <section>
          <h2 className="text-2xl font-semibold mb-6">Border Radius</h2>
          <Card>
            <CardContent className="pt-6">
              <div className="flex gap-8">
                <div className="text-center">
                  <div className="w-16 h-16 bg-primary rounded-sm" />
                  <p className="text-sm mt-2">sm</p>
                </div>
                <div className="text-center">
                  <div className="w-16 h-16 bg-primary rounded-md" />
                  <p className="text-sm mt-2">md</p>
                </div>
                <div className="text-center">
                  <div className="w-16 h-16 bg-primary rounded-lg" />
                  <p className="text-sm mt-2">lg</p>
                </div>
                <div className="text-center">
                  <div className="w-16 h-16 bg-primary rounded-full" />
                  <p className="text-sm mt-2">full</p>
                </div>
              </div>
            </CardContent>
          </Card>
        </section>
      </div>
    </div>
  );
}
```

### Step 10: Add Route to React Router (Web)

In your router configuration (e.g., `src/routes.tsx` or `src/App.tsx`):

```tsx
import StyleguidePage from '@/pages/styleguide';

// Add to your routes:
{
  path: '/styleguide',
  element: <StyleguidePage />,
}
```

### Step 11: Generate Mobile Theme (React Native)

**Only if mobile project exists.** Create `proyecto-mobile/src/theme/tokens.ts` from the JSON:

```typescript
// src/theme/tokens.ts
// AUTO-GENERATED FROM design-tokens.json - DO NOT EDIT MANUALLY

export const colors = {
  // Light theme (default)
  primary: '#3B82F6',
  primaryForeground: '#FFFFFF',
  secondary: '#F1F5F9',
  secondaryForeground: '#0F172A',
  background: '#FFFFFF',
  foreground: '#0F172A',
  card: '#FFFFFF',
  cardForeground: '#0F172A',
  muted: '#F1F5F9',
  mutedForeground: '#64748B',
  destructive: '#EF4444',
  destructiveForeground: '#FFFFFF',
  border: '#E2E8F0',
  ring: '#3B82F6',
  success: '#22C55E',
  successForeground: '#FFFFFF',
  warning: '#F59E0B',
  warningForeground: '#0F172A',
  info: '#0EA5E9',
  infoForeground: '#FFFFFF',
} as const;

export const colorsDark = {
  background: '#0F172A',
  foreground: '#F8FAFC',
  card: '#1E293B',
  cardForeground: '#F8FAFC',
  muted: '#1E293B',
  mutedForeground: '#94A3B8',
  border: '#334155',
  // Other colors same as light
  primary: colors.primary,
  primaryForeground: colors.primaryForeground,
  secondary: '#334155',
  secondaryForeground: '#F8FAFC',
  destructive: colors.destructive,
  destructiveForeground: colors.destructiveForeground,
  ring: colors.ring,
  success: colors.success,
  successForeground: colors.successForeground,
  warning: colors.warning,
  warningForeground: colors.warningForeground,
  info: colors.info,
  infoForeground: colors.infoForeground,
} as const;

export const spacing = {
  xs: 4,
  sm: 8,
  md: 16,
  lg: 24,
  xl: 32,
  '2xl': 48,
} as const;

export const borderRadius = {
  sm: 4,
  md: 6,
  lg: 8,
  xl: 12,
  full: 9999,
} as const;

export const typography = {
  fontFamily: 'Inter',
  h1: {
    fontSize: 28,
    fontWeight: '700' as const,
    lineHeight: 34,
  },
  h2: {
    fontSize: 22,
    fontWeight: '600' as const,
    lineHeight: 28,
  },
  h3: {
    fontSize: 18,
    fontWeight: '600' as const,
    lineHeight: 24,
  },
  body: {
    fontSize: 16,
    fontWeight: '400' as const,
    lineHeight: 22,
  },
  bodySmall: {
    fontSize: 14,
    fontWeight: '400' as const,
    lineHeight: 20,
  },
  caption: {
    fontSize: 12,
    fontWeight: '400' as const,
    lineHeight: 16,
  },
} as const;

export const shadows = {
  sm: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.05,
    shadowRadius: 2,
    elevation: 1,
  },
  md: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.1,
    shadowRadius: 6,
    elevation: 3,
  },
  lg: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 10 },
    shadowOpacity: 0.1,
    shadowRadius: 15,
    elevation: 5,
  },
} as const;

// Theme hook helper
export type ColorScheme = 'light' | 'dark';

export function getColors(scheme: ColorScheme) {
  return scheme === 'dark' ? { ...colors, ...colorsDark } : colors;
}
```

**Create `proyecto-mobile/src/theme/index.ts` to export everything:**

```typescript
// src/theme/index.ts
export * from './tokens';

import { colors, colorsDark, spacing, borderRadius, typography, shadows, getColors } from './tokens';

// Re-export as default theme object for convenience
export const theme = {
  colors,
  colorsDark,
  spacing,
  borderRadius,
  typography,
  shadows,
  getColors,
} as const;
```

---

## Output

After running this prompt:

**Source of Truth:**
- `.claude/docs/design-tokens.json` - All tokens in one place

**Web (proyecto-web/):**
- shadcn initialized
- `src/index.css` with CSS variables (light + dark)
- `tailwind.config.js` with semantic colors
- Google Font added (if applicable)
- Demo components installed
- Styleguide page at `/styleguide`

**Mobile (proyecto-mobile/) - if exists:**
- `src/theme/tokens.ts` - Auto-generated from JSON
- `src/theme/index.ts` - Theme exports
- Dark mode support included

---

## Design Summary (provide after setup)

After setup, summarize:
- **Primary color:** [hex and HSL]
- **Font:** [font name]
- **Style:** [e.g., "Modern minimal", "Bold colorful", "Soft friendly"]
- **Border radius:** [e.g., "0.5rem rounded", "Sharp 0", "Pill"]
- **Overall feel:** [brief description]

---

## Accessibility Check

Verify contrast ratios meet WCAG AA (4.5:1 for text):
- Primary on white/dark background
- Foreground on background
- Muted-foreground on muted

Use browser DevTools or https://webaim.org/resources/contrastchecker/ to validate.

---

## Notes

- **HSL Format:** shadcn expects `H S% L%` without `hsl()` wrapper
- **Figma Variables:** If Figma has design tokens defined, they'll be in `get_variable_defs` response
- **Fallbacks:** If colors aren't clearly visible in Figma, use harmonious defaults
- **Dark mode:** Always generate complete dark theme, not placeholders
