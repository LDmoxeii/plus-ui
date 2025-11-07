# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RuoYi-Vue-Plus is a multi-tenant management system built with Vue 3 + TypeScript + Element Plus + Vite. This is the frontend project that pairs with RuoYi-Vue-Plus (distributed cluster) or RuoYi-Cloud-Plus (microservices) backend frameworks.

**Technology Stack:**
- Vue 3.5 (Composition API)
- TypeScript 5.8
- Element Plus 2.9 (UI components)
- Vite 6.3 (build tool)
- Pinia (state management)
- Vue Router 4.5
- UnoCSS (atomic CSS)
- VXE-Table (advanced table component)
- Axios (HTTP client)

## Development Commands

### Setup and Development
```bash
# Install dependencies (using Chinese mirror)
npm install --registry=https://registry.npmmirror.com

# Start development server (runs on port 80)
npm run dev

# Preview production build
npm run preview
```

### Building
```bash
# Build for production
npm run build:prod

# Build for development environment
npm run build:dev
```

### Code Quality
```bash
# Run ESLint
npm run lint:eslint

# Fix ESLint issues automatically
npm run lint:eslint:fix

# Format code with Prettier
npm run prettier
```

## Architecture

### Project Structure

```
src/
├── api/              # API request definitions organized by module
├── assets/           # Static assets (styles, images, icons)
├── components/       # Reusable Vue components
├── directive/        # Custom Vue directives (v-hasPermi, v-hasRoles, v-copyText)
├── enums/            # TypeScript enums
├── hooks/            # Vue composables
├── lang/             # i18n internationalization files
├── layout/           # Layout components (main app shell)
├── plugins/          # Vue plugins (modal, tab, cache, download, auth)
├── router/           # Vue Router configuration
├── store/            # Pinia store modules
├── types/            # TypeScript type definitions
├── utils/            # Utility functions
├── views/            # Page components organized by module
├── App.vue           # Root component
├── main.ts           # Application entry point
├── permission.ts     # Route permission guard
└── settings.ts       # Application settings
```

### Key Architectural Concepts

#### 1. Dynamic Routing & Permissions
- Static routes defined in `src/router/index.ts` (login, register, 404, etc.)
- Dynamic routes loaded from backend based on user roles/permissions
- Route guard in `src/permission.ts` handles authentication and dynamic route generation
- Uses `usePermissionStore().generateRoutes()` to build user-specific menu structure

#### 2. State Management (Pinia)
Store modules in `src/store/modules/`:
- `user.ts` - User authentication, info, and logout
- `permission.ts` - Dynamic route generation based on user permissions
- `settings.ts` - App settings (theme, layout, language)
- `tagsView.ts` - Visited page tags
- `dict.ts` - Data dictionary cache
- `app.ts` - App-wide state (sidebar, device type)

#### 3. API Request Architecture
- All API calls centralized in `src/api/` organized by module (system, monitor, tool, etc.)
- Axios instance configured in `src/utils/request.ts` with:
  - Request encryption (RSA + AES) when `VITE_APP_ENCRYPT=true`
  - Response decryption
  - Token injection via `Authorization: Bearer <token>`
  - Duplicate submission prevention (500ms interval)
  - 401 auto-logout and redirect
  - Language header for i18n (`Content-Language`)
  - Client ID header for tenant identification

#### 4. Auto Import Configuration
- Vue APIs (ref, computed, watch, etc.) auto-imported via `unplugin-auto-import`
- Element Plus components auto-imported via `unplugin-vue-components`
- Custom components in `src/components/` are auto-registered
- Icons via `unplugin-icons` with `@iconify` collections
- No need to manually import Vue/Element Plus APIs or components

#### 5. Multi-Tenancy Support
- Client ID sent in every request (`VITE_APP_CLIENT_ID`)
- Backend determines tenant context from client ID
- Tenant-specific data isolation handled on backend

#### 6. Real-time Communication
- SSE (Server-Sent Events) enabled by default (`VITE_APP_SSE=true`)
- WebSocket available as alternative (`VITE_APP_WEBSOCKET`)
- Used for notifications and real-time updates

### Configuration Files

#### Environment Variables
- `.env.development` - Development environment (proxy to `localhost:8080`)
- `.env.production` - Production environment
- Key variables:
  - `VITE_APP_BASE_API` - API base path
  - `VITE_APP_TITLE` - Application title
  - `VITE_APP_CLIENT_ID` - Tenant client ID
  - `VITE_APP_ENCRYPT` - Enable request/response encryption
  - `VITE_APP_RSA_PUBLIC_KEY` / `VITE_APP_RSA_PRIVATE_KEY` - Encryption keys

#### Vite Configuration (`vite.config.ts`)
- Base path from `VITE_APP_CONTEXT_PATH`
- Path alias: `@` → `./src`
- Dev server proxy: `/dev-api` → `http://localhost:8080`
- SCSS with modern compiler API
- CSS autoprefixer for browser compatibility
- Pre-optimization for common dependencies

#### TypeScript Configuration (`tsconfig.json`)
- Target: ES2020
- Module: ESNext with Bundler resolution
- Strict mode with some relaxed rules:
  - `noImplicitAny: false`
  - `strictFunctionTypes: false`
  - `strictNullChecks: false`

### Custom Global Properties
Registered via `src/plugins/index.ts`:
- `$tab` - Tab operations (open, close, refresh)
- `$modal` - Modal dialogs (confirm, alert, message)
- `$cache` - Local/session storage wrapper
- `$download` - File download helper
- `$auth` - Permission checking utilities
- `useDict()` - Dictionary data loading
- `parseTime()`, `handleTree()`, `addDateRange()` - Data formatting utilities

### Custom Directives
- `v-hasPermi="['system:user:add']"` - Show element if user has permission
- `v-hasRoles="['admin']"` - Show element if user has role
- `v-copyText` - Copy text to clipboard

## Working with This Codebase

### Adding a New Page
1. Create component in `src/views/{module}/{page}.vue`
2. Add API methods in `src/api/{module}.ts`
3. Route will be dynamically added by backend menu configuration
4. Use `<script setup>` with TypeScript
5. Leverage auto-imports for Vue APIs and Element Plus components

### Adding API Requests
1. Define request function in appropriate `src/api/{module}.ts` file
2. Import and use the configured axios instance from `@/utils/request`
3. For encrypted APIs, add header: `headers: { isEncrypt: 'true' }`
4. For public APIs (no auth), add header: `headers: { isToken: false }`
5. For APIs allowing duplicate submission, add: `headers: { repeatSubmit: false }`

### Working with Permissions
- Check permissions in components: `v-hasPermi="['system:user:edit']"`
- Check programmatically: use `$auth.hasPermi()` or `$auth.hasRole()`
- Permissions loaded from backend in `useUserStore().getInfo()`

### Styling
- Primary styling via UnoCSS atomic classes
- Element Plus theme customization in `src/settings.ts`
- Global styles in `src/assets/styles/`
- SCSS variables available in components
- Dark mode support via `dark: true` in settings

### Data Dictionaries
- Use `useDict('dict_type')` to load dictionary data
- Cached in Pinia store to avoid repeated requests
- Display with `<DictTag>` component or `selectDictLabel()` utility

### Testing
No test suite is currently configured in this project.

## Branch Strategy
- `ts` - Stable release branch (production ready)
- `dev` - Development branch (active development)

## Backend Integration
This frontend requires the RuoYi-Vue-Plus or RuoYi-Cloud-Plus backend running on `http://localhost:8080` (or configured proxy target).

Backend provides:
- JWT authentication tokens
- Dynamic menu/route configuration
- Permission data
- All business APIs
- Data dictionary definitions
