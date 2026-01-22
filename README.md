# Workflow: SRS to MVP (Enterprise Stack)

Workflow de Claude Code para transformar un documento SRS (Software Requirements Specification) en un MVP funcional con arquitectura multi-proyecto.

## Stack Tecnológico

| Capa | Tecnología | Proyecto |
|------|------------|----------|
| Backend API | NestJS (TypeScript) | proyecto-api |
| Database | PostgreSQL (AWS RDS) | - |
| Web Frontend | React.js + Vite + TailwindCSS | proyecto-web |
| Mobile Frontend | React Native (TypeScript) | proyecto-mobile |
| Blockchain | Solidity + Hardhat | proyecto-contracts |
| Infraestructura | AWS (ECS, SQS, KMS, S3) | - |

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
│       - .claude/docs/pages-tree.md (Web + Mobile)                       │
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
│  5. Ejecutar Sesiones por Fases                                         │
│                                                                         │
│     FASE 1: Backend + Web                                               │
│     ├── Session 01: API Setup (NestJS)                                  │
│     ├── Session 02: Web Setup (React.js + Vite)                         │
│     ├── Session 03: Database Schema                                     │
│     ├── Session 04: Auth Module (API)                                   │
│     ├── Session 05: Auth Integration (Web)                              │
│     ├── Session 06: Web Design System                                   │
│     └── Session 07+: Feature Modules (API → Web)                        │
│                                                                         │
│     FASE 2: Mobile (cuando Fase 1 esté completa)                        │
│     ├── Session M01: Mobile Setup (React Native)                        │
│     ├── Session M02: Mobile Design System                               │
│     └── Session M03+: Mobile Screens                                    │
│                                                                         │
│     FASE 3: Blockchain (cuando sea necesario)                           │
│     ├── Session B01: Contracts Setup (Hardhat)                          │
│     └── Session B02+: Smart Contracts                                   │
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
│   ├── api-builder.md        # Implementa módulos NestJS (NEW)
│   ├── ui-builder.md         # Implementa páginas React.js (UPDATED)
│   ├── mobile-builder.md     # Implementa screens React Native (NEW)
│   └── blockchain-builder.md # Implementa smart contracts (NEW)
│
├── commands/
│   ├── sessions-maker.md     # Genera sesiones de desarrollo
│   ├── figma/
│   │   ├── prompt1-tokens.md      # Setup Design System
│   │   └── prompt2-components.md  # Crear componentes individuales
│   ├── projectSpec/
│   │   └── firstStep-spec.md      # PUNTO DE ENTRADA
│   └── todo/
│       ├── ralph.md          # Ejecuta sesiones secuencialmente
│       └── qa.md             # Testing
│
├── docs/
│   ├── pdf/                  # Colocar aquí el SRS en PDF
│   ├── projectSpec/
│   │   └── projectSpec.md    # Especificación técnica generada
│   ├── design-tokens.json    # Tokens de diseño (fuente de verdad)
│   └── pages-tree.md         # Árbol de páginas/screens con URLs de Figma
│
└── sessions/                 # Sesiones de desarrollo generadas
    ├── session-01-api-setup.md
    ├── session-02-web-setup.md
    ├── session-M01-mobile-setup.md
    └── session-B01-contracts-setup.md
```

---

## Paso a Paso Detallado

### Paso 1: Preparar el SRS

Coloca el documento SRS en formato PDF en:
```
.claude/docs/pdf/mi-proyecto.pdf
```

### Paso 2: Generar Especificación

Ejecuta el comando:
```
/projectSpec/firstStep-spec
```

Este comando genera:

| Archivo | Descripción |
|---------|-------------|
| `.claude/docs/projectSpec/projectSpec.md` | Especificación técnica completa |
| `.claude/docs/pages-tree.md` | Árbol de páginas Web + screens Mobile |
| `CLAUDE.md` | Resumen del proyecto para Claude |
| `.env.example` | Variables de entorno por proyecto |

### Paso 3: Añadir URLs de Figma

Edita `.claude/docs/pages-tree.md` y reemplaza cada `[FIGMA_URL]`:

**Antes:**
```markdown
## Web Backoffice

### Dashboard (admin)
├── /dashboard                     [FIGMA_URL]
├── /dashboard/users               [FIGMA_URL]

## Mobile App

### Home (user)
├── HomeScreen                     [FIGMA_URL]
├── ProfileScreen                  [FIGMA_URL]
```

**Después:**
```markdown
## Web Backoffice

### Dashboard (admin)
├── /dashboard                     https://figma.com/design/xxx?node-id=10-100
├── /dashboard/users               https://figma.com/design/xxx?node-id=10-200

## Mobile App

### Home (user)
├── HomeScreen                     https://figma.com/design/xxx?node-id=20-100
├── ProfileScreen                  https://figma.com/design/xxx?node-id=20-200
```

### Paso 4: Generar Sesiones

Ejecuta:
```
/sessions-maker
```

### Paso 5: Ejecutar Sesiones

Ejecuta las sesiones en orden por fase.

---

## Agentes Disponibles

| Agente | Función | Invocado por | Fase |
|--------|---------|--------------|------|
| `media-interpreter` | Extrae info de PDFs | `firstStep-spec` | - |
| `srs-to-spec` | Genera projectSpec.md | `firstStep-spec` | - |
| `api-builder` | Módulos NestJS (Controller, Service, Entity) | Sesiones | 1 |
| `ui-builder` | Páginas React.js con hooks y API calls | Sesiones | 1 |
| `mobile-builder` | Screens React Native | Sesiones | 2 |
| `blockchain-builder` | Smart Contracts Solidity | Sesiones | 3 |

---

## Invocación de Agentes

### API Builder (NestJS)

```yaml
Task: API Builder Agent
Prompt: |
  Module: users
  Endpoints:
    - GET /api/v1/users - List users with pagination
    - POST /api/v1/users - Create user
    - GET /api/v1/users/:id - Get user by ID
    - PATCH /api/v1/users/:id - Update user
    - DELETE /api/v1/users/:id - Delete user
  Entity: |
    id: uuid (PK)
    email: varchar(255) unique
    name: varchar(255)
    role: enum (admin, user)
    createdAt, updatedAt
  Business Logic: |
    - Email must be unique
    - Only admin can create other admins
    - Send welcome email on creation
  Context: |
    See projectSpec section 3.5 for schema
```

### UI Builder (React.js Web)

```yaml
Task: UI Builder Agent (Web)
Prompt: |
  Figma: https://figma.com/design/xxx?node-id=10-200
  Target: Users list page
  Route: /users
  API Endpoints: |
    - GET /api/v1/users - List users
    - DELETE /api/v1/users/:id - Delete user
  Functionality: |
    - Table with pagination
    - Search by name/email
    - Delete with confirmation modal
    - Link to create/edit pages
  Context: |
    Admin only page
```

### Mobile Builder (React Native)

```yaml
Task: Mobile Builder Agent
Prompt: |
  Figma: https://figma.com/design/xxx?node-id=20-300
  Target: ProfileScreen
  Navigation: BottomTabs.Profile
  API Endpoints: |
    - GET /api/v1/users/me - Get current user
    - PATCH /api/v1/users/me - Update profile
  Functionality: |
    - Display user info
    - Edit profile form
    - Change avatar with camera/gallery
    - Logout button
  Context: |
    Authenticated users only
```

### Blockchain Builder (Solidity)

```yaml
Task: Blockchain Builder Agent
Prompt: |
  Contract: TokenVault
  Type: Custom
  Functions: |
    - deposit(uint256 amount): Deposit tokens
    - withdraw(uint256 amount): Withdraw tokens
    - balanceOf(address): Get balance
  Storage: |
    - balances: mapping(address => uint256)
    - totalDeposits: uint256
  Events: |
    - Deposited(address user, uint256 amount)
    - Withdrawn(address user, uint256 amount)
  Access Control: |
    - Owner can pause/unpause
    - Anyone can deposit/withdraw
  Context: |
    See projectSpec for token requirements
```

---

## Comandos Disponibles

| Comando | Descripción |
|---------|-------------|
| `/projectSpec/firstStep-spec` | **Punto de entrada.** Procesa SRS y genera especificación |
| `/sessions-maker` | Genera sesiones de desarrollo |
| `/figma/prompt1-tokens [URL]` | Configura Design System (tokens sincronizados web + mobile) |
| `/figma/prompt2-components [nombre] [URL?]` | Crea/customiza un componente |
| `/todo/ralph` | Ejecuta sesiones secuencialmente |

---

## Design Tokens (Sincronización Web + Mobile)

El comando `/figma/prompt1-tokens` genera una **fuente de verdad única** que sincroniza los tokens de diseño entre todas las plataformas:

```
Figma Design
     ↓
/figma/prompt1-tokens [URL]
     ↓
┌────────────────────────────────────────────────────────┐
│  .claude/docs/design-tokens.json  ← FUENTE DE VERDAD  │
└────────────────────────────────────────────────────────┘
     │
     ├──→ proyecto-web/src/index.css        (CSS variables)
     ├──→ proyecto-web/tailwind.config.js   (Tailwind colors)
     │
     └──→ proyecto-mobile/src/theme/tokens.ts (React Native)
              ├── colors, colorsDark
              ├── spacing, borderRadius
              ├── typography, shadows
              └── getColors(scheme) para dark mode
```

**Beneficio:** Cuando cambias un color en Figma y re-ejecutas el comando, se actualiza automáticamente en web Y mobile.

### Uso en Web (Vite + React)

```tsx
// Usa clases de Tailwind con CSS variables
<div className="bg-primary text-primary-foreground">
  <span className="text-muted-foreground">Texto secundario</span>
</div>
```

### Uso en Mobile (React Native)

```tsx
import { colors, spacing } from '@/theme';
import { getColors } from '@/theme';

// Light/Dark mode automático
const colorScheme = useColorScheme();
const themeColors = getColors(colorScheme ?? 'light');

<View style={{ backgroundColor: themeColors.background }}>
  <Text style={{ color: themeColors.foreground }}>Hola</Text>
</View>
```

---

## Estructura de Proyectos

```
proyecto-api/                 # NestJS Backend
├── src/
│   ├── modules/             # Feature modules
│   ├── config/              # Configuration
│   ├── database/            # Entities, migrations
│   └── common/              # Shared utilities
├── test/
└── package.json

proyecto-web/                 # React.js Backoffice
├── src/
│   ├── components/          # UI components
│   ├── pages/               # Route pages
│   ├── hooks/               # Custom hooks
│   ├── services/            # API services
│   ├── stores/              # Zustand stores
│   └── types/               # TypeScript types
└── package.json

proyecto-mobile/              # React Native App
├── src/
│   ├── components/          # Shared components
│   ├── screens/             # App screens
│   ├── navigation/          # React Navigation
│   ├── services/            # API client
│   └── theme/               # Design tokens
├── ios/
├── android/
└── package.json

proyecto-contracts/           # Solidity Contracts
├── contracts/               # Smart contracts
├── scripts/                 # Deploy scripts
├── test/                    # Contract tests
└── hardhat.config.ts
```

---

## Notas Importantes

### Orden de Sesiones

Las sesiones tienen dependencias:

**Fase 1:**
1. Session 01 (API Setup) → Antes que todo
2. Session 02 (Web Setup) → Puede ser paralela a 01
3. Session 03 (Database) → Requiere Session 01
4. Session 04 (Auth API) → Requiere Session 03
5. Session 05 (Auth Web) → Requiere Sessions 02 y 04
6. Session 06 (Design System) → Requiere Session 02
7. Sessions 07+ → Requieren Sessions 04 y 06

**Fase 2:** Requiere que Fase 1 esté completa (al menos el API)

**Fase 3:** Independiente, pero la integración requiere Fase 1

### URLs de Figma

Las URLs de Figma incluyen un `node-id` que identifica el frame:
```
https://figma.com/design/ABC123/FileName?node-id=100-500
                                                 ^^^^^^^^
```

### API First

Para cada feature:
1. Primero implementar el módulo API (api-builder)
2. Luego implementar la UI Web (ui-builder)
3. Finalmente la pantalla Mobile (mobile-builder)

---

## Troubleshooting

### "El ui-builder no encuentra el endpoint"
Ejecuta primero la sesión del API module correspondiente.

### "Las URLs de Figma no funcionan"
1. Verifica acceso al archivo Figma
2. Copia la URL directamente desde Figma (con node-id correcto)
3. Verifica que Figma MCP esté configurado

### "sessions-maker genera placeholders"
Olvidaste editar `pages-tree.md`. Reemplaza todos los `[FIGMA_URL]` antes de ejecutar.

### "Error de tipos entre proyectos"
Los tipos de API deben mantenerse sincronizados manualmente entre api y web/mobile.
Considera crear un paquete npm compartido para tipos.

### "Los colores de mobile no coinciden con web"
Re-ejecuta `/figma/prompt1-tokens [URL]` para regenerar los tokens en ambas plataformas desde la misma fuente.

---

## Vulnerabilidades Conocidas

| Problema | Workaround |
|----------|------------|
| URLs de Figma sin llenar | Revisar pages-tree.md antes de sessions-maker |
| Tipos API desincronizados | Mantener types/ actualizado en todos los proyectos |
| No hay rollback automático | Hacer commits de git entre sesiones |
| API no disponible para web | Ejecutar sesiones API antes que Web |
| Design tokens desactualizados | Re-ejecutar `/figma/prompt1-tokens` después de cambios en Figma |
