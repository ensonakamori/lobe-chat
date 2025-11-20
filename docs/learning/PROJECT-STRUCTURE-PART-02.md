# Project Structure Guide - Part 2 of 2

**Documented:** November 2025
**Tech Stack Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)
**Target:** Mid-level developers proficient in JS/TS/React

**📄 This is Part 2 of 2** | **[← Back to Part 1](./PROJECT-STRUCTURE-PART-01.md)**

This document continues the project structure guide, covering packages, quick references, naming conventions, and practical guides.

**Prerequisites:**
- **[Part 1](./PROJECT-STRUCTURE-PART-01.md)** - Read this first!

**Time to read:** 15 minutes (Part 2)

---

## Table of Contents - Part 2

1. [Packages (`packages/`)](#packages-packages)
2. [Quick Reference: Where Do I Put...?](#quick-reference-where-do-i-put)
3. [Naming Conventions](#naming-conventions)
4. [Finding Things Quickly](#finding-things-quickly)
5. [Testing Structure](#testing-structure)
6. [Configuration Files](#configuration-files)
7. [Key Files to Know](#key-files-to-know)
8. [Next Steps](#next-steps)
9. [Quick Reference Card](#quick-reference-card)
10. [Summary](#summary)

**[← Part 1](./PROJECT-STRUCTURE-PART-01.md)** covered:
- Root Directory Overview
- Source Directory (`src/`)
- Detailed Directory Breakdown (app, components, features, store, services, server, layout)

---

## Packages (`packages/`)

**🧠 Mental Model:** Shared npm packages in the monorepo

```
packages/
├── database/                      ← Database layer ⭐
│   ├── src/
│   │   ├── schemas/               ← Table schemas (Drizzle)
│   │   │   ├── user.ts
│   │   │   ├── message.ts
│   │   │   └── ...
│   │   ├── models/                ← CRUD operations
│   │   │   ├── user.ts
│   │   │   └── ...
│   │   ├── repositories/          ← Complex queries
│   │   │   ├── user.ts
│   │   │   └── ...
│   │   ├── client/                ← PGLite setup
│   │   └── server/                ← Neon setup
│   └── package.json
│
├── types/                         ← TypeScript types ⭐
│   ├── src/
│   │   ├── message/
│   │   ├── user/
│   │   └── ...
│   └── package.json
│
├── utils/                         ← Utility functions ⭐
│   ├── src/
│   │   ├── string.ts
│   │   ├── date.ts
│   │   └── ...
│   └── package.json
│
├── const/                         ← Constants
├── model-runtime/                 ← LLM provider integrations
├── agent-runtime/                 ← AI agent execution
├── electron-client-ipc/           ← Desktop IPC (client)
├── electron-server-ipc/           ← Desktop IPC (server)
├── file-loaders/                  ← File parsing (PDF, DOC, etc.)
├── web-crawler/                   ← Web crawling utilities
├── context-engine/                ← Context management
├── conversation-flow/             ← Conversation logic
├── prompts/                       ← AI prompts
├── fetch-sse/                     ← SSE client
├── python-interpreter/            ← Python interpreter (WASM)
├── obervability-otel/             ← OpenTelemetry
├── ssrf-safe-fetch/               ← SSRF-protected fetch
├── memory-extract/                ← Memory extraction
└── model-bank/                    ← Model configurations
```

**⭐ = Most used packages**

---

### Package: `@lobechat/database`

**Purpose:** Complete database layer (schemas, models, repositories)

```
packages/database/
├── src/
│   ├── schemas/                   ← Drizzle table schemas
│   │   ├── user.ts                ← User table
│   │   ├── message.ts             ← Message table
│   │   └── ...                    ← More tables
│   │
│   ├── models/                    ← CRUD operations
│   │   ├── user.ts                ← User model
│   │   │   ├── create()
│   │   │   ├── update()
│   │   │   ├── delete()
│   │   │   └── findById()
│   │   └── ...
│   │
│   ├── repositories/              ← Complex queries (BFF)
│   │   ├── user.ts                ← User repository
│   │   │   ├── findWithMessages()
│   │   │   ├── findPopular()
│   │   │   └── ...
│   │   └── ...
│   │
│   ├── client/                    ← PGLite (browser)
│   │   └── index.ts               ← Client DB instance
│   │
│   └── server/                    ← Neon (server)
│       └── index.ts               ← Server DB instance
│
├── package.json
└── tsconfig.json
```

**Usage Examples:**

```typescript
// Client-side (uses PGLite)
import { clientDB } from '@lobechat/database/client';
const messages = await clientDB.query.messages.findMany();

// Server-side (uses Neon)
import { serverDB } from '@lobechat/database/server';
const users = await serverDB.query.users.findMany();

// Using models
import { UserModel } from '@lobechat/database/models/user';
const user = await UserModel.create({ name: 'Alice' });

// Using repositories
import { UserRepository } from '@lobechat/database/repositories/user';
const userWithMessages = await UserRepository.findWithMessages('user-id');
```

📚 **Deep dive:** [DATABASE_ARCHITECTURE.md](./DATABASE_ARCHITECTURE.md)

---

### Package: `@lobechat/types`

**Purpose:** Shared TypeScript types across the entire app

```
packages/types/
├── src/
│   ├── message/
│   │   ├── index.ts               ← Message types
│   │   └── enums.ts
│   ├── user/
│   │   └── index.ts               ← User types
│   ├── agent/
│   ├── chat/
│   └── ...
└── package.json
```

**Usage:**
```typescript
import type { Message, MessageRole } from '@lobechat/types/message';
import type { User } from '@lobechat/types/user';
```

**🎯 Benefit:** One source of truth for all types

---

### Package: `@lobechat/utils`

**Purpose:** Utility functions used everywhere

```
packages/utils/
├── src/
│   ├── string.ts                  ← String utilities
│   ├── date.ts                    ← Date utilities
│   ├── url.ts                     ← URL utilities
│   ├── crypto.ts                  ← Crypto utilities
│   └── ...
└── package.json
```

**Usage:**
```typescript
import { formatDate } from '@lobechat/utils/date';
import { slugify } from '@lobechat/utils/string';
```

---

## Quick Reference: Where Do I Put...?

### ✅ Adding a New Feature

| What | Where | Example |
|------|-------|---------|
| New page | `src/app/[variants]/(main)/[page-name]/page.tsx` | `/dashboard` page |
| Page component | Same folder as page: `/[page]/components/` | Dashboard-specific component |
| Shared component | `src/components/[ComponentName]/` | Button, Input, Modal |
| Complex feature | `src/features/[FeatureName]/` | ChatInput, FileUpload |
| Global state | `src/store/[domain]/` | Chat store, User store |
| Client service | `src/services/[domain]/client.ts` | Message service |
| tRPC route | `src/server/routers/lambda/[resource].ts` | User API |
| Server service | `src/server/services/[domain]/` | Email service |
| Database schema | `packages/database/src/schemas/[table].ts` | New table |
| TypeScript type | `packages/types/src/[domain]/` | New type definition |
| Utility function | `packages/utils/src/[category].ts` | Date helper |
| Hook | `src/hooks/use[HookName].ts` | useDebounce |
| Test file | Same folder: `[file].test.ts` | Component tests |
| Translation key | `src/locales/default/[namespace].ts` | i18n keys |
| Environment variable | `.env.example` | New config |

---

### 🎯 Component Placement Decision Tree

```
Where should my component go?

Is it used in only ONE page?
├─ YES → Put it in that page's components/ folder
│         Example: src/app/[variants]/(main)/chat/components/
│
└─ NO → Is it simple and generic (Button, Input)?
    ├─ YES → src/components/
    │
    └─ NO → Is it complex with multiple sub-components?
        ├─ YES → src/features/
        │
        └─ NO → Still not sure? Start in components/, move later if needed
```

---

### 📊 Code Organization Principles

**1. Colocate Related Code**
```
✅ GOOD:
MyFeature/
├── index.tsx
├── style.ts
├── hooks/
└── components/

❌ BAD:
components/MyFeature.tsx
styles/MyFeature.css
hooks/useMyFeature.ts
```

**2. Generic → Specific**
```
Reusability hierarchy:
packages/ (most reusable)
  ↓
src/components/ (reusable across pages)
  ↓
src/features/ (reusable in few pages)
  ↓
src/app/[page]/components/ (page-specific, least reusable)
```

**3. Clear Naming**
```
✅ GOOD:
UserProfileCard.tsx
useUserProfile.ts
userService.ts

❌ BAD:
card.tsx
hook.ts
service.ts
```

---

## Naming Conventions

### Files and Folders

| Type | Convention | Example |
|------|------------|---------|
| React component | PascalCase | `Button.tsx`, `UserProfile.tsx` |
| Utility function | camelCase | `formatDate.ts`, `apiClient.ts` |
| Hook | camelCase with `use` | `useDebounce.ts`, `useUser.ts` |
| Store | camelCase | `userStore.ts`, `chatStore.ts` |
| Service | camelCase | `messageService.ts` |
| Type/Interface | PascalCase | `User.ts`, `Message.ts` |
| Constant | UPPER_SNAKE_CASE | `API_URL.ts` |
| Test file | Same as source | `Button.test.tsx` |
| Folder | kebab-case or PascalCase | `chat-input/` or `ChatInput/` |

**🎯 Project uses:** Mostly **PascalCase for components**, **camelCase for logic**

---

### Exports

**Named exports (preferred):**
```typescript
// ✅ GOOD
export function Button() { }
export function Input() { }

// Usage
import { Button, Input } from '@/components';
```

**Default exports (for pages, layouts):**
```typescript
// ✅ GOOD for Next.js pages
export default function ChatPage() { }
```

**Barrel exports (`index.ts`):**
```typescript
// components/index.ts
export { Button } from './Button';
export { Input } from './Input';

// Usage
import { Button, Input } from '@/components';
```

**🎯 Rule:** Use named exports everywhere except Next.js pages

---

## Finding Things Quickly

### VS Code Tips

**Quick file navigation:**
```
Cmd/Ctrl + P → Type filename
```

**Find all references:**
```
Right-click symbol → Find All References
```

**Go to definition:**
```
Cmd/Ctrl + Click on symbol
```

---

### Command Line Tools

**Find a file:**
```bash
find src -name "*User*.tsx"
```

**Search for code:**
```bash
grep -r "useUserStore" src/
```

**Find component usage:**
```bash
grep -r "import.*Button" src/
```

---

### Project-Specific Shortcuts

**Import Aliases:**
```typescript
@/                  → src/
@/components        → src/components/
@/services          → src/services/
@lobechat/database  → packages/database/src/
@lobechat/types     → packages/types/src/
```

**Example:**
```typescript
// Instead of:
import { Button } from '../../../components/Button';

// Use:
import { Button } from '@/components/Button';
```

---

## Testing Structure

Tests are colocated with source files:

```
src/components/Button/
├── index.tsx                      ← Component
├── style.ts                       ← Styles
└── index.test.tsx                 ← Tests ✅

src/services/message/
├── client.ts                      ← Service
└── __tests__/                     ← Test folder
    └── client.test.ts             ← Tests ✅
```

**Conventions:**
- Component tests: `[Component].test.tsx` (same folder)
- Service tests: `__tests__/[service].test.ts` (subfolder)
- Integration tests: `__tests__/integration/`
- E2E tests: `e2e/src/`

📚 **More details:** [TESTING_GUIDE.md](./TESTING_GUIDE.md)

---

## Configuration Files

### Root Configuration

| File | Purpose |
|------|---------|
| `next.config.js` | Next.js configuration |
| `tsconfig.json` | TypeScript compiler options |
| `vitest.config.ts` | Vitest test configuration |
| `playwright.config.ts` | Playwright E2E configuration |
| `drizzle.config.ts` | Drizzle ORM configuration |
| `.eslintrc.js` | ESLint rules |
| `.prettierrc` | Prettier formatting |
| `pnpm-workspace.yaml` | pnpm monorepo configuration |
| `package.json` | Dependencies and scripts |

---

## Key Files to Know

### Entry Points

| File | Purpose |
|------|---------|
| `src/app/[variants]/layout.tsx` | Root layout (wraps all pages) |
| `src/app/[variants]/page.tsx` | Home page |
| `src/layout/GlobalProvider/` | Global context setup |
| `packages/database/client/index.ts` | Client DB instance |
| `packages/database/server/index.ts` | Server DB instance |

---

### Configuration

| File | Purpose |
|------|---------|
| `src/config/` | App configuration |
| `src/envs/` | Environment variable validation |
| `src/const/` | App constants |

---

## Next Steps

Now that you know where everything is, learn more:

### Understanding the Code

1. **[Tech Stack Guide](./TECH_STACK_GUIDE.md)** (2 hrs)
   - Deep dive into each technology
   - Why we use them
   - How they work together

2. **[Code Tours](./CODE_TOURS.md)** (1.5 hrs)
   - Follow code paths through the system
   - See how features work end-to-end

---

### Practical Development

3. **[Patterns & Conventions](./PATTERNS_AND_CONVENTIONS.md)** (1 hr)
   - Code style guide
   - Best practices
   - Current vs outdated patterns

4. **[How-To Guide](./HOW_TO_GUIDE.md)** (1.5 hrs)
   - Add a new page (step-by-step)
   - Create a component
   - Add a database table
   - Common tasks

---

### Architecture Deep Dives

5. **[Frontend Architecture](./FRONTEND_ARCHITECTURE.md)** (1.5 hrs)
   - React 19 patterns
   - Component architecture
   - State management

6. **[Backend Architecture](./BACKEND_ARCHITECTURE.md)** (1.5 hrs)
   - tRPC patterns
   - Server services
   - API organization

7. **[Database Architecture](./DATABASE_ARCHITECTURE.md)** (1.5 hrs)
   - Schema design
   - Drizzle ORM
   - PGLite vs Neon

---

## Quick Reference Card

**Find a Component:**
```
Generic → src/components/
Feature → src/features/
Page-specific → src/app/[page]/components/
```

**Find Business Logic:**
```
Client-side → src/services/
Server-side → src/server/services/
```

**Find API Routes:**
```
tRPC → src/server/routers/
REST → src/app/(backend)/webapi/
```

**Find State:**
```
Global → src/store/
Server → SWR/React Query in components
```

**Find Types:**
```
Shared → packages/types/
Local → Same folder as code
```

**Find Database:**
```
Schemas → packages/database/src/schemas/
Models → packages/database/src/models/
Queries → packages/database/src/repositories/
```

---

## Summary

**LobeHub Project Structure in One Paragraph:**

> A Next.js 16 monorepo with a clear separation between client (`src/services/`) and server (`src/server/`) code. Components are organized by reusability (generic → `components/`, complex → `features/`, page-specific → inline). State lives in Zustand stores (`src/store/`), APIs in tRPC routers (`src/server/routers/`), and database logic in the `@lobechat/database` package. The App Router (`src/app/`) uses route groups to separate backend and frontend routes.

**🎯 Remember:** Everything is organized by **function and domain**, not by technical type

---

**Document Status:** ✅ Complete (Part 2 of 2) | **Last Updated:** November 20, 2025

**[← Back to Part 1](./PROJECT-STRUCTURE-PART-01.md)** | **Next:** [Tech Stack Guide](./TECH_STACK_GUIDE.md)
