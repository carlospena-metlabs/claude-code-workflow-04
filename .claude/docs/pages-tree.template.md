# Pages Tree Template

Copy this file to `pages-tree.md` and customize it for your project.

---

# Pages Tree

## Web Backoffice

### Authentication (public)
├── /login                              [FIGMA_URL]
├── /register                           [FIGMA_URL]
├── /forgot-password                    [FIGMA_URL]
└── /reset-password                     [FIGMA_URL]

### Dashboard (admin, user)
├── /dashboard                          [FIGMA_URL]
└── /dashboard/analytics                [FIGMA_URL]

### Users Management (admin)
├── /users                              [FIGMA_URL]
├── /users/:id                          [FIGMA_URL]
└── /users/new                          [FIGMA_URL]

### Settings (admin, user)
├── /settings                           [FIGMA_URL]
├── /settings/profile                   [FIGMA_URL]
└── /settings/security                  [FIGMA_URL]

## Mobile App

### Onboarding (public)
├── WelcomeScreen                       [FIGMA_URL]
├── LoginScreen                         [FIGMA_URL]
└── RegisterScreen                      [FIGMA_URL]

### Main (authenticated)
├── HomeScreen                          [FIGMA_URL]
├── ProfileScreen                       [FIGMA_URL]
└── SettingsScreen                      [FIGMA_URL]

---

## Format Rules

1. **Structure:**
   - Separate `## Web Backoffice` and `## Mobile App` sections
   - Group pages by area with `### Group Name (roles)`
   - Use tree characters: `├──`, `│`, `└──`

2. **Routes:**
   - Web: Use URL paths (`/dashboard`, `/users/:id`)
   - Mobile: Use screen names (`HomeScreen`, `ProfileScreen`)

3. **Figma URLs:**
   - Replace `[FIGMA_URL]` with actual Figma links
   - Format: `https://figma.com/design/xxx/file?node-id=xx-xx`

4. **Roles:**
   - Add roles in parentheses after group name
   - `(public)` = no authentication required
   - `(admin)` = admin only
   - `(admin, user)` = multiple roles allowed
