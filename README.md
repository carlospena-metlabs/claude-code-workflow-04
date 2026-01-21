# Workflow: SRS to MVP

Workflow de Claude Code para transformar un documento SRS (Software Requirements Specification) en un MVP funcional con Next.js + Supabase.

---

## Flujo General

```
┌─────────────────────────────────────────────────────────────────────────┐
│  1. SRS (PDF)                                                           │
│     Colocar documento de requisitos en .claude/docs/pdf/                │
└─────────────────────────────────┬───────────────────────────────────────┘
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  2. /projectSpec/firstStep-spec                                         │
│     Orquesta: media-interpreter → srs-to-spec                           │
│     Genera:                                                             │
│       - .claude/docs/projectSpec/projectSpec.md                         │
│       - .claude/docs/pages-tree.md                                      │
│       - CLAUDE.md                                                       │
│       - .env.example                                                    │
└─────────────────────────────────┬───────────────────────────────────────┘
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  3. Añadir URLs de Figma                                                │
│     Editar pages-tree.md y reemplazar [FIGMA_URL] con URLs reales       │
└─────────────────────────────────┬───────────────────────────────────────┘
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  4. /sessions-maker                                                     │
│     Lee: projectSpec.md + pages-tree.md                                 │
│     Genera: sesiones de desarrollo en .claude/sessions/                 │
└─────────────────────────────────┬───────────────────────────────────────┘
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  5. Ejecutar Sesiones                                                   │
│     Session 01: Project Setup (Next.js, Supabase, estructura)           │
│     Session 02: Design System (prompt1-tokens + prompt2-components)     │
│     Session 03+: Features/Páginas (invoca ui-builder para cada página)  │
└─────────────────────────────────┬───────────────────────────────────────┘
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  6. MVP Funcional                                                       │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Estructura de Archivos

```
.claude/
├── agents/
│   ├── srs-to-spec.md        # Transforma requisitos en especificación técnica
│   ├── media-interpreter.md  # Extrae información de PDFs/imágenes
│   └── ui-builder.md         # Implementa páginas combinando Figma + lógica
│
├── commands/
│   ├── sessions-maker.md     # Genera sesiones de desarrollo
│   ├── figma/
│   │   ├── prompt1-tokens.md      # Setup Design System (shadcn + tokens CSS)
│   │   └── prompt2-components.md  # Crear componentes individuales
│   ├── projectSpec/
│   │   └── firstStep-spec.md      # PUNTO DE ENTRADA: orquesta todo el proceso
│   └── todo/
│       ├── ralph.md          # Ejecuta sesiones secuencialmente
│       └── qa.md             # Testing
│
├── docs/
│   ├── pdf/                  # Colocar aquí el SRS en PDF
│   ├── projectSpec/
│   │   └── projectSpec.md    # Especificación técnica generada
│   └── pages-tree.md         # Árbol de páginas con URLs de Figma
│
└── sessions/                 # Sesiones de desarrollo generadas
    ├── session-01-setup.md
    ├── session-02-design-system.md
    └── session-XX-feature.md
```

---

## Paso a Paso Detallado

### Paso 1: Preparar el SRS

Coloca el documento SRS en formato PDF en:
```
.claude/docs/pdf/mi-proyecto.pdf
```

Puede haber múltiples PDFs; se procesarán todos como parte del mismo proyecto.

### Paso 2: Generar Especificación

Ejecuta el comando:
```
/projectSpec/firstStep-spec
```

Este comando:
1. Usa `media-interpreter` para extraer requisitos del PDF
2. Usa `srs-to-spec` para generar la especificación técnica
3. Genera los siguientes archivos:

| Archivo | Descripción |
|---------|-------------|
| `.claude/docs/projectSpec/projectSpec.md` | Especificación técnica completa |
| `.claude/docs/pages-tree.md` | Árbol de páginas con `[FIGMA_URL]` placeholders |
| `CLAUDE.md` | Resumen del proyecto para Claude |
| `.env.example` | Variables de entorno necesarias |

### Paso 3: Añadir URLs de Figma

Edita `.claude/docs/pages-tree.md` y reemplaza cada `[FIGMA_URL]` con la URL real de Figma:

**Antes:**
```markdown
## Dashboard (client, agent, super_agent)
├── /dashboard                     [FIGMA_URL]
├── /dashboard/returns             [FIGMA_URL]
```

**Después:**
```markdown
## Dashboard (client, agent, super_agent)
├── /dashboard                     https://figma.com/design/xxx?node-id=10-100
├── /dashboard/returns             https://figma.com/design/xxx?node-id=10-200
```

### Paso 4: Generar Sesiones

Ejecuta:
```
/sessions-maker
```

Esto genera sesiones en `.claude/sessions/` ordenadas por dependencias lógicas.

### Paso 5: Ejecutar Sesiones

Ejecuta cada sesión en orden:

| Sesión | Contenido |
|--------|-----------|
| 01 | **Project Setup:** Next.js, Tailwind, Supabase, estructura de carpetas |
| 02 | **Design System:** Ejecutar `/figma/prompt1-tokens` + `/figma/prompt2-components` × N |
| 03+ | **Features:** Páginas usando el agente `ui-builder` |

#### Sesiones con UI

Cuando una sesión incluye un bloque `UI Builder Invocation`, invócalo así:

```yaml
Task: UI Builder Agent
Prompt: |
  Figma: https://figma.com/design/xxx?node-id=10-100
  Target: Dashboard
  Route: /dashboard
  Functionality: |
    - Fetch user data from users table
    - Show balance summary
    - Display returns chart
  Context: |
    See projectSpec/projectSpec.md section 3.5 for database schema
```

---

## Comandos Disponibles

| Comando | Descripción |
|---------|-------------|
| `/projectSpec/firstStep-spec` | **Punto de entrada.** Procesa SRS y genera toda la especificación |
| `/sessions-maker` | Genera sesiones de desarrollo desde projectSpec + pages-tree |
| `/figma/prompt1-tokens` | Configura Design System (shadcn init + tokens CSS) |
| `/figma/prompt2-components` | Crea un componente individual con showcase en styleguide |
| `/todo/ralph` | Ejecuta todas las sesiones secuencialmente |

---

## Agentes

| Agente | Función | Invocado por |
|--------|---------|--------------|
| `media-interpreter` | Extrae información de PDFs e imágenes | `firstStep-spec` |
| `srs-to-spec` | Genera projectSpec.md y pages-tree.md | `firstStep-spec` |
| `ui-builder` | Implementa páginas con Figma + lógica de negocio | Sesiones (manual) |

---

## Design System (Session 02)

El Design System se crea con dos prompts:

### /figma/prompt1-tokens

Ejecutar **una vez** al inicio:
- Inicializa shadcn/ui (`npx shadcn@latest init`)
- Extrae tokens de color, tipografía, spacing del Figma
- Genera `/app/globals.css` con variables CSS
- Crea styleguide base en `/app/styleguide/`

### /figma/prompt2-components

Ejecutar **una vez por cada componente** que necesites:
- Instala componente de shadcn si existe
- Customiza si es necesario
- Crea página de showcase en `/app/styleguide/components/[name]/`
- Actualiza navegación del styleguide

---

## UI Builder Agent

El agente `ui-builder` se invoca durante las sesiones para implementar páginas completas. Combina:

- **Diseño visual** → del Figma (via Figma MCP)
- **Lógica de negocio** → del projectSpec
- **Componentes** → del Design System existente

### Qué hace:

1. Lee el diseño de Figma
2. Inventaría componentes existentes en el proyecto
3. Instala componentes shadcn faltantes
4. Crea types TypeScript
5. Crea Server Actions con validación Zod
6. Implementa la página completa (auth, data fetching, estados de UI)

### Formato de invocación:

```yaml
Task: UI Builder Agent
Prompt: |
  Figma: [URL completa de Figma con node-id]
  Target: [Nombre de la página]
  Route: [Ruta destino, ej: /dashboard]
  Functionality: |
    - [Lista de funcionalidades requeridas]
    - [Server actions necesarios]
    - [Data fetching requerido]
  Context: |
    [Secciones relevantes del projectSpec]
    [Tablas de base de datos involucradas]
```

### Regla importante:

**Una invocación = Una página completa.** El agente decide internamente qué extraer a `_components/`.

**Excepción:** Para componentes reutilizables en múltiples páginas (ej: un chart en Dashboard Y Backoffice), crear una invocación separada ANTES apuntando a `/components/[ComponentName].tsx`.

---

## Notas Importantes

### URLs de Figma

Las URLs de Figma incluyen un `node-id` que identifica el frame:
```
https://figma.com/design/ABC123/FileName?node-id=100-500
                                                 ^^^^^^^^
```

- El `node-id` es estable mientras no elimines y recrees el frame
- Si reorganizas tu Figma, verifica que los IDs sigan siendo válidos

### Orden de Sesiones

Las sesiones tienen dependencias:
- **Session 01** crea el proyecto base (Next.js, Supabase)
- **Session 02** crea el Design System (requiere Session 01)
- **Session 03+** crean features (requieren Session 01 y 02)

No saltes sesiones.

### Componentes Reutilizables

Si un componente se usa en múltiples páginas:
1. Créalo primero en `/components/` (no en `_components/`)
2. Luego las páginas lo importan con `import { Component } from '@/components/...'`

---

## Stack Técnico

| Capa | Tecnología |
|------|------------|
| Frontend | Next.js 14+ (App Router) |
| Styling | Tailwind CSS + shadcn/ui |
| Backend | Supabase (PostgreSQL, Auth, Storage) |
| Email | Resend |
| Testing | Playwright / Cypress |
| Deploy | Vercel |

---

## Troubleshooting

### "El ui-builder no encuentra componentes"
Ejecuta Session 02 (Design System) antes de las sesiones de features.

### "Las URLs de Figma no funcionan"
1. Verifica acceso al archivo Figma
2. Copia la URL directamente desde Figma (con node-id correcto)
3. Verifica que Figma MCP esté configurado

### "sessions-maker genera placeholders"
Olvidaste editar `pages-tree.md`. Reemplaza todos los `[FIGMA_URL]` antes de ejecutar.

### "firstStep-spec no encuentra el PDF"
El PDF debe estar en `.claude/docs/pdf/`. Verifica la ruta.

---

## Vulnerabilidades Conocidas

| Problema | Workaround |
|----------|------------|
| URLs de Figma sin llenar | Revisar pages-tree.md antes de ejecutar sessions-maker |
| Ralph no invoca ui-builder automáticamente | Invocar manualmente el Task cuando veas el bloque YAML en la sesión |
| No hay rollback automático | Hacer commits de git entre sesiones |
