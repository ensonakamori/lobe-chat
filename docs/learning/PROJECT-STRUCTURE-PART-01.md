# Project Structure Guide - Part 1 of 2

**Documented:** November 2025
**Tech Stack Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)
**Target:** Mid-level developers proficient in JS/TS/React

**📄 This is Part 1 of 2** | **[Continue to Part 2 →](./PROJECT-STRUCTURE-PART-02.md)**

This guide explains the complete directory structure of LobeHub, helping you find code quickly and understand the organization.

**Prerequisites:**
- [Getting Started](./GETTING_STARTED.md) - Have the project cloned
- [Architecture Overview](./ARCHITECTURE-OVERVIEW-PART-01.md) - Understand the system

**Time to read:** 15 minutes (Part 1)

---

## Table of Contents - Part 1

1. [Root Directory Overview](#root-directory-overview)
2. [Source Directory (`src/`)](#source-directory-src)
3. [Detailed Directory Breakdown](#detailed-directory-breakdown)
   - [`src/app/` - Next.js 16 App Router](#srcapp---nextjs-16-app-router)
   - [`src/components/` - Shared Components](#srccomponents---shared-components)
   - [`src/features/` - Feature Components](#srcfeatures---feature-components)
   - [`src/store/` - Zustand Stores](#srcstore---zustand-stores)
   - [`src/services/` - Client Services](#srcservices---client-services)
   - [`src/server/` - Server-Only Code](#srcserver---server-only-code)
   - [`src/layout/` - Global Providers](#srclayout---global-providers)

**[Part 2 →](./PROJECT-STRUCTURE-PART-02.md)** will cover:
- Packages (`packages/`)
- Quick Reference: Where Do I Put...?
- Naming Conventions
- Finding Things Quickly
- Testing Structure
- Configuration Files
- Next Steps

---

## Root Directory Overview

```
lobe-chat/                          ← Repository root
├── .cursor/                        ← Cursor IDE rules
├── .github/                        ← GitHub workflows, PR templates
├── .husky/                         ← Git hooks
├── .vscode/                        ← VS Code settings
├── apps/                           ← Multi-app workspace
│   └── desktop/                    ← Electron desktop app
├── changelog/                      ← Changelog entries
├── docs/                           ← Documentation
│   ├── learning/                   ← Learning guides (you are here!)
│   ├── development/                ← Development docs
│   ├── self-hosting/               ← Deployment guides
│   └── usage/                      ← User guides
├── e2e/                            ← Playwright E2E tests
├── locales/                        ← i18n translations (17 languages)
├── packages/                       ← Monorepo packages (19 packages)
├── public/                         ← Static assets
├── scripts/                        ← Build and utility scripts
├── src/                            ← Main application code ⭐
├── .env.example                    ← Environment variables template
├── .nvmrc                          ← Node version specification
├── next.config.js                  ← Next.js configuration
├── package.json                    ← Dependencies and scripts
├── pnpm-workspace.yaml             ← pnpm workspace config
├── tsconfig.json                   ← TypeScript configuration
└── vitest.config.ts                ← Test configuration
```

**🎯 Key Directories:**
- **`src/`** - Where you'll spend 90% of your time
- **`packages/`** - Shared code across the app
- **`docs/learning/`** - All learning documentation

---

## Source Directory (`src/`)

**🧠 Mental Model:** Organized by function, not by file type

```
src/
├── app/                           ← Next.js 16 App Router ⭐
│   ├── (backend)/                 ← Backend routes (APIs, auth)
│   └── [variants]/                ← Frontend routes (pages)
│
├── components/                    ← Shared React components
│   ├── Button/
│   ├── Input/
│   └── ...
│
├── features/                      ← Feature-specific components
│   ├── ChatInput/                 ← Complex multi-component features
│   ├── MessageList/
│   └── ...
│
├── store/                         ← Zustand state stores ⭐
│   ├── user/                      ← User store
│   ├── chat/                      ← Chat store
│   └── ...
│
├── services/                      ← Client-side services ⭐
│   ├── message/                   ← Message service (client DB)
│   ├── user/                      ← User service
│   └── ...
│
├── server/                        ← Server-only code ⭐
│   ├── routers/                   ← tRPC routers
│   ├── services/                  ← Server services (server DB)
│   └── modules/                   ← Third-party integrations
│
├── layout/                        ← Global providers
│   ├── AuthProvider/              ← Authentication context
│   └── GlobalProvider/            ← App-wide providers
│
├── hooks/                         ← Custom React hooks
├── utils/                         ← Utility functions
├── config/                        ← App configuration
├── const/                         ← Constants
├── types/                         ← TypeScript types
├── locales/                       ← i18n configuration
├── styles/                        ← Global styles
├── libs/                          ← Third-party lib configurations
├── tools/                         ← Development tools
├── helpers/                       ← Helper functions
├── envs/                          ← Environment validation
├── proxy.ts                       ← Next.js 16 Proxy API ✅ NEW
└── instrumentation.ts             ← OpenTelemetry setup
```

**⭐ = Most frequently used directories**

---

## Detailed Directory Breakdown

### `src/app/` - Next.js 16 App Router

**🧠 Mental Model:** File system = URL routes

```
src/app/
├── (backend)/                     ← Backend-only routes (no UI)
│   ├── api/                       ← REST API routes
│   │   ├── auth/                  ← Authentication endpoints
│   │   └── webhooks/              ← Webhook handlers
│   ├── trpc/                      ← tRPC route handler
│   │   └── [trpc]/                ← Catch-all for tRPC
│   ├── webapi/                    ← Legacy REST APIs
│   │   ├── chat/                  ← Chat API
│   │   └── tts/                   ← Text-to-speech API
│   ├── oidc/                      ← OIDC authentication
│   └── middleware/                ← API middleware
│
├── [variants]/                    ← Frontend routes (rendered)
│   ├── (auth)/                    ← Auth pages (route group)
│   │   ├── login/                 ← /login
│   │   │   └── [[...login]]/      ← Catch-all for auth variants
│   │   │       └── page.tsx
│   │   └── signup/                ← /signup
│   │       └── [[...signup]]/
│   │           └── page.tsx
│   │
│   └── (main)/                    ← Main app (route group)
│       ├── chat/                  ← /chat
│       │   ├── page.tsx           ← Chat page
│       │   ├── _layout/           ← Chat layout components
│       │   ├── components/        ← Chat-specific components
│       │   ├── features/          ← Chat features
│       │   └── settings/          ← Chat settings
│       │
│       ├── discover/              ← /discover (marketplace)
│       │   ├── (list)/            ← List view
│       │   ├── (detail)/          ← Detail view
│       │   ├── _layout/
│       │   └── features/
│       │
│       ├── knowledge/             ← /knowledge (knowledge base)
│       │   ├── routes/            ← Subroutes
│       │   ├── components/
│       │   └── hooks/
│       │
│       ├── image/                 ← /image (image generation)
│       │   ├── @menu/             ← Parallel route (menu slot)
│       │   ├── @topic/            ← Parallel route (topic slot)
│       │   └── features/
│       │
│       ├── profile/               ← /profile (user profile)
│       │   ├── (home)/            ← Profile home
│       │   ├── @category/         ← Parallel route
│       │   └── _layout/
│       │
│       ├── settings/              ← /settings
│       │   └── ...                ← Settings pages
│       │
│       ├── changelog/             ← /changelog
│       ├── labs/                  ← /labs (experimental)
│       └── layouts/               ← Layout components
│           ├── desktop/
│           └── mobile/
│
├── desktop/                       ← Desktop-only routes
├── market-auth-callback/          ← OAuth callback
├── manifest.ts                    ← PWA manifest
├── robots.tsx                     ← robots.txt
├── sitemap.tsx                    ← sitemap.xml
└── sw.ts                          ← Service Worker
```

**Key Concepts:**

**Route Groups:** `(main)` and `(auth)`
- Organize routes without affecting URLs
- `(main)` = authenticated routes
- `(auth)` = login/signup pages

**Catch-all Routes:** `[[...login]]`
- Double brackets = optional catch-all
- Handles multiple auth providers

**Parallel Routes:** `@menu`, `@topic`
- Multiple simultaneous UI sections
- Can show different content in slots

**🎯 Remember:** Parentheses `()` don't appear in URLs!

---

### `src/components/` - Shared Components

**🧠 Mental Model:** Generic, reusable UI building blocks

```
src/components/
├── Avatar/                        ← User avatar
│   ├── index.tsx                  ← Component
│   ├── style.ts                   ← Styles
│   └── index.test.tsx             ← Tests
│
├── Button/                        ← Button variants
├── Input/                         ← Form inputs
├── Modal/                         ← Modal dialogs
├── Tooltip/                       ← Tooltips
└── ...                            ← 50+ components
```

**Organization Pattern:**
```
Component/
├── index.tsx                      ← Main component
├── style.ts                       ← antd-style styling
├── index.test.tsx                 ← Vitest tests
├── types.ts                       ← Component types (if complex)
└── components/                    ← Sub-components (if needed)
    └── SubComponent.tsx
```

**🌉 Bridge from React:** Like `src/components/` in Create React App, but colocated with styles and tests

---

### `src/features/` - Feature Components

**🧠 Mental Model:** Complex, multi-component features

```
src/features/
├── ChatInput/                     ← Chat input feature
│   ├── index.tsx                  ← Main component
│   ├── ActionBar/                 ← Sub-feature
│   ├── FileUpload/                ← Sub-feature
│   ├── VoiceInput/                ← Sub-feature
│   └── hooks/                     ← Feature-specific hooks
│       └── useChatInput.ts
│
├── MessageList/                   ← Message list feature
│   ├── index.tsx
│   ├── MessageItem/
│   ├── MessageGroup/
│   └── ...
│
└── ...                            ← More features
```

**Difference from `components/`:**
- **Components** = Single, reusable (Button, Input)
- **Features** = Complex, composed (ChatInput, MessageList)

**💡 When to use which?**
- Used in 1 place → Page's `components/` folder
- Used in 2-3 places → `features/`
- Used everywhere → `components/`

---

### `src/store/` - Zustand Stores

**🧠 Mental Model:** Global app state by domain

```
src/store/
├── user/                          ← User store
│   ├── index.ts                   ← Store definition
│   ├── slices/                    ← State slices
│   │   ├── auth/                  ← Auth slice
│   │   │   ├── action.ts          ← Actions
│   │   │   └── selectors.ts       ← Selectors
│   │   ├── settings/              ← Settings slice
│   │   └── ...
│   └── store.ts                   ← Combined store
│
├── chat/                          ← Chat store
│   ├── slices/
│   │   ├── message/               ← Message slice
│   │   ├── conversation/          ← Conversation slice
│   │   └── ...
│   └── store.ts
│
├── global/                        ← Global UI store
└── ...                            ← More stores
```

**Organization Pattern (Recommended):**
```
store/[domain]/
├── index.ts                       ← Public API (exports)
├── store.ts                       ← Store creation
└── slices/                        ← Organized by feature
    └── [feature]/
        ├── action.ts              ← State mutations
        ├── selectors.ts           ← State selectors (optional)
        └── initialState.ts        ← Initial state
```

**Usage Example:**
```typescript
// In component
import { useUserStore } from '@/store/user';

const user = useUserStore((s) => s.user);
const login = useUserStore((s) => s.login);
```

**🎯 Remember:** One store per domain (user, chat, settings)

📚 **More details:** [Patterns & Conventions - Zustand](./PATTERNS_AND_CONVENTIONS.md#zustand)

---

### `src/services/` - Client Services

**🧠 Mental Model:** Client-side business logic + database access

```
src/services/
├── message/                       ← Message service
│   ├── client.ts                  ← Client DB operations (PGLite)
│   ├── index.ts                   ← Public API
│   ├── hooks/                     ← React hooks for this service
│   │   └── useMessages.ts
│   └── __tests__/                 ← Tests
│       └── client.test.ts
│
├── user/                          ← User service
│   ├── client.ts
│   └── ...
│
├── chat/                          ← Chat service
├── file/                          ← File service
├── export/                        ← Export service
└── ...                            ← More services
```

**Key Pattern:**
```typescript
// services/message/client.ts
import { clientDB } from '@lobechat/database/client';

class MessageService {
  async getMessages() {
    return clientDB.query.messages.findMany();
  }

  async createMessage(data) {
    return clientDB.insert(messages).values(data);
  }
}

export const messageService = new MessageService();
```

**✅ Can access:** PGLite database, tRPC client, browser APIs
**❌ Cannot access:** Server database, Node.js APIs

---

### `src/server/` - Server-Only Code

**🧠 Mental Model:** Backend business logic (never sent to browser)

```
src/server/
├── routers/                       ← tRPC routers ⭐
│   ├── lambda/                    ← Serverless functions (edge)
│   │   ├── user.ts
│   │   ├── message.ts
│   │   ├── agent.ts
│   │   └── ...
│   │
│   ├── async/                     ← Background jobs (Node runtime)
│   │   ├── fileUpload.ts
│   │   ├── generation.ts
│   │   └── ...
│   │
│   ├── desktop/                   ← Desktop-specific APIs
│   │   ├── system.ts
│   │   └── ...
│   │
│   ├── mobile/                    ← Mobile-specific APIs (future)
│   └── tools/                     ← Utility endpoints
│
├── services/                      ← Server services ⭐
│   ├── message/                   ← Message service (server)
│   │   ├── index.ts               ← Service class
│   │   └── __tests__/
│   │
│   ├── user/                      ← User service
│   ├── agent/                     ← Agent service
│   ├── file/                      ← File service
│   └── ...                        ← More services
│
├── modules/                       ← Third-party integrations
│   ├── S3/                        ← S3 storage
│   ├── PluginStore/               ← Plugin marketplace
│   ├── ModelRuntime/              ← LLM providers
│   ├── EdgeConfig/                ← Vercel Edge Config
│   └── ...
│
├── featureFlags/                  ← Feature flag logic
├── globalConfig/                  ← Global configuration
└── utils/                         ← Server utilities
```

**Key Differences:**

| | `src/services/` (Client) | `src/server/services/` (Server) |
|---|---|---|
| **Runtime** | Browser | Node.js / Edge |
| **Database** | PGLite (client) | Neon (server) |
| **APIs** | Browser APIs | Node.js APIs |
| **Size matters** | Yes (sent to browser) | No (server-only) |
| **Can use** | React hooks, DOM | File system, crypto |

**🎯 Remember:** Server code NEVER goes to the browser (bundle size ↓)

---

### `src/layout/` - Global Providers

**🧠 Mental Model:** App-wide context providers

```
src/layout/
├── AuthProvider/                  ← Authentication context
├── GlobalProvider/                ← All providers combined
├── StoreProvider/                 ← Zustand store setup
└── ...                            ← More providers
```

**Usage Pattern:**
```typescript
// layout/GlobalProvider/index.tsx
export function GlobalProvider({ children }) {
  return (
    <AuthProvider>
      <StoreProvider>
        <ThemeProvider>
          {children}
        </ThemeProvider>
      </StoreProvider>
    </AuthProvider>
  );
}
```

**🌉 Bridge from React:** Like wrapping your app in `<Provider>` tags

---

**Document Status:** ✅ Part 1 Complete | **Last Updated:** November 20, 2025

**[Continue to Part 2 →](./PROJECT-STRUCTURE-PART-02.md)** - Packages, Quick References, and Practical Guides
