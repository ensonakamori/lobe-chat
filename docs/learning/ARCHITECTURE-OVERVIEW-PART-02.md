# Architecture Overview - Part 2 of 2

**Documented:** November 2025
**Tech Stack Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)
**Target:** Mid-level developers proficient in JS/TS/React

**📄 This is Part 2 of 2** | **[← Back to Part 1](./ARCHITECTURE-OVERVIEW-PART-01.md)**

This document continues the architecture overview, covering architectural decisions, security, performance, and deployment.

**Prerequisites:**
- **[Part 1](./ARCHITECTURE-OVERVIEW-PART-01.md)** - Read this first!

**Time to read:** 20 minutes (Part 2)

---

## Table of Contents - Part 2

1. [Key Architectural Decisions](#key-architectural-decisions)
2. [Security Architecture](#security-architecture)
3. [Performance Considerations](#performance-considerations)
4. [Deployment Architecture](#deployment-architecture)
5. [Development vs Production](#development-vs-production)
6. [Architecture Evolution](#architecture-evolution)
7. [Common Patterns](#common-patterns)
8. [Next Steps](#next-steps)

**[← Part 1](./ARCHITECTURE-OVERVIEW-PART-01.md)** covered:
- The Big Picture
- Architecture Principles
- System Architecture
- Technology Stack Overview
- Monorepo Structure
- Frontend Architecture
- Backend Architecture
- Database Strategy
- Data Flow Patterns
- Platform Variants

---

## Key Architectural Decisions

### Decision 1: Why React Server Components?

**Problem:** Traditional SPAs load all JavaScript upfront, slow initial load

**Solution:** React Server Components
- Render on server
- Send HTML instead of JavaScript
- Interactive parts load separately

**Trade-off:** More complex mental model, but better performance

✅ **Current (Nov 2025):** RSC is production-ready and recommended

---

### Decision 2: Why tRPC over REST?

**Problem:** REST APIs require:
- Manual type definitions
- API documentation
- Type drift between client/server

**Solution:** tRPC
- Types automatically shared
- No code generation needed
- TypeScript end-to-end

**Trade-off:** Locked into TypeScript ecosystem

**💡 Why it's worth it:** Type safety catches bugs at compile time

---

### Decision 3: Why Dual Databases?

**Problem:** Single database means:
- Always need internet
- Slow queries (network latency)
- Single point of failure

**Solution:** PGLite + Neon
- Work offline (PGLite)
- Sync when online (Neon)
- Fast local queries

**Trade-off:** Sync complexity, but better UX

**🚀 Innovation:** First major framework to use WASM PostgreSQL in production

---

### Decision 4: Why Monorepo?

**Problem:** Multiple repos mean:
- Version coordination
- Duplicate code
- Difficult atomic changes

**Solution:** pnpm workspaces monorepo
- Single source of truth
- Shared packages
- Atomic commits

**Trade-off:** Larger repo size, but better DX

---

### Decision 5: Why Next.js over CRA/Vite?

**Problem:** SPAs need:
- Server-side rendering
- API routes
- File-based routing
- Image optimization

**Solution:** Next.js provides all built-in
- SSR + SSG + ISR
- API routes (REST + tRPC)
- App Router
- Automatic optimizations

**Trade-off:** Framework lock-in, but huge productivity gain

✅ **Next.js 16 (Nov 2025):** Stable Turbopack, production-ready

---

## Security Architecture

### Authentication Flow

```
1. User logs in
   ↓
2. NextAuth / Clerk validates credentials
   ↓
3. JWT token issued (httpOnly cookie)
   ↓
4. Token included in requests
   ↓
5. Server validates token
   ↓
6. Access granted/denied
```

**Supported Methods:**
- NextAuth 5.0 (email, OAuth, OIDC)
- Clerk (managed auth service)
- Custom OIDC provider

📚 **Deep dive:** [SECURITY_GUIDE.md](./SECURITY_GUIDE.md)

---

### Data Security

**Client-Side (PGLite):**
- Encrypted at rest (browser storage encryption)
- Sandboxed (can't access other sites)
- No network exposure

**Server-Side (Neon):**
- TLS in transit
- Encrypted at rest
- Row-level security (RLS)
- API key management (KEY_VAULTS_SECRET)

---

## Performance Considerations

### Frontend Performance

**Optimizations:**
- React 19 compiler (automatic memoization)
- Code splitting (per route)
- Image optimization (next/image)
- Font optimization (next/font)
- Streaming (Suspense)

**Metrics:**
- First Contentful Paint (FCP): <1.5s
- Largest Contentful Paint (LCP): <2.5s
- Time to Interactive (TTI): <3.5s

---

### Backend Performance

**Optimizations:**
- Edge runtime (low latency)
- Database connection pooling
- Query optimization (Drizzle ORM)
- Caching (SWR, React Query)
- CDN for static assets

---

## Deployment Architecture

### Production Deployment

```
┌─────────────────────────────────────┐
│           CDN (Vercel Edge)         │
│   • Static assets                   │
│   • Edge functions                  │
└────────────┬────────────────────────┘
             ↓
┌─────────────────────────────────────┐
│      Next.js Application            │
│   • Server Components               │
│   • API Routes                      │
│   • tRPC                            │
└────────────┬────────────────────────┘
             ↓
┌──────────────────┬──────────────────┐
│  Neon PostgreSQL │   S3 Storage     │
│  (Serverless)    │   (Files)        │
└──────────────────┴──────────────────┘
```

**Hosting Options:**
- Vercel (recommended, automatic)
- Docker (self-hosted)
- Zeabur, Sealos, Alibaba Cloud

---

## Development vs Production

### Key Differences

| Aspect | Development | Production |
|--------|-------------|------------|
| **Bundler** | Turbopack (dev server) | Turbopack (optimized) |
| **Database** | Local PostgreSQL or PGLite | Neon PostgreSQL |
| **Auth** | Optional (can skip) | Required |
| **Storage** | Local files | S3 compatible |
| **Cache** | Disabled | Enabled |
| **Source maps** | Enabled | Disabled |
| **Hot reload** | Enabled | N/A |

---

## Architecture Evolution

### Current Architecture (v2.x)

✅ React 19 + Next.js 16
✅ tRPC for APIs
✅ PGLite + Neon dual DB
✅ Monorepo structure
✅ TypeScript throughout

### Historical Context

**v1.x (2023-2024):**
- Next.js 13/14 Pages Router
- Client-side only database
- Single deployment target

**v2.x (2025):**
- Next.js 15/16 App Router
- Dual database strategy
- Multi-platform (web, desktop, mobile)

---

## Common Patterns

### Pattern: Server Component with Client Component

```typescript
// app/page.tsx (Server Component)
async function Page() {
  const data = await db.query.users.findMany(); // Direct DB access!

  return <ClientComponent data={data} />;
}

// components/ClientComponent.tsx
'use client';

function ClientComponent({ data }) {
  const [selected, setSelected] = useState(null); // Client state

  return <div onClick={() => setSelected(data[0])}>{/*...*/}</div>;
}
```

**🎯 Remember:** Server = Async + DB, Client = Interactive + Hooks

---

### Pattern: Type-Safe API Call

```typescript
// Server: Define procedure
export const userRouter = router({
  getUser: publicProcedure
    .input(z.object({ id: z.string() }))
    .query(async ({ input }) => {
      return await db.query.users.findFirst({
        where: eq(users.id, input.id),
      });
    }),
});

// Client: Call procedure (fully typed!)
const { data } = trpc.user.getUser.useQuery({ id: '123' });
//    ^? User | undefined (type automatically inferred!)
```

**💡 No manual types, no API docs needed!**

---

### Pattern: Optimistic Updates

```typescript
const { mutate } = trpc.message.send.useMutation();

// Optimistic: Update UI immediately
const optimisticMessage = { id: 'temp', text: 'Hello', sending: true };
setMessages([...messages, optimisticMessage]);

// Actual: Send to server
await mutate({ text: 'Hello' });

// Server confirms, update with real ID
```

**🌉 Bridge from React:** Like setState, but with server sync

---

## Next Steps

Now that you understand the architecture, explore specific areas:

### For Frontend Developers

1. **[Frontend Architecture](./FRONTEND_ARCHITECTURE.md)** (1.5 hrs)
   - React 19 patterns in detail
   - Component organization
   - State management deep dive

2. **[Patterns & Conventions](./PATTERNS_AND_CONVENTIONS.md)** (1 hr)
   - Code style guide
   - Current vs outdated patterns

---

### For Backend Developers

1. **[Backend Architecture](./BACKEND_ARCHITECTURE.md)** (1.5 hrs)
   - tRPC router patterns
   - Server services
   - Authentication

2. **[Database Architecture](./DATABASE_ARCHITECTURE.md)** (1.5 hrs)
   - Drizzle ORM usage
   - Schema design
   - Migrations

---

### For Full-Stack Developers

1. **[Data Flow Guide](./DATA_FLOW_GUIDE.md)** (1 hr)
   - Request lifecycle
   - Sync strategies
   - Performance optimization

2. **[Tech Stack Guide](./TECH_STACK_GUIDE.md)** (2 hrs)
   - Every technology explained
   - Why we chose each one

---

### Practical Next Steps

1. **[Project Structure](./PROJECT_STRUCTURE.md)** (30 min)
   - Navigate the codebase
   - Find things quickly

2. **[How-To Guide](./HOW_TO_GUIDE.md)** (1.5 hrs)
   - Add a new page
   - Create a tRPC route
   - Common tasks

3. **[Code Tours](./CODE_TOURS.md)** (1.5 hrs)
   - Follow a chat message through the system
   - Trace authentication
   - Understand data flow

---

## Quick Reference

### Architecture Cheat Sheet

**File = URL:**
```
app/[variants]/(main)/chat/page.tsx → /chat
```

**Client vs Server:**
```
'use client' = Can use hooks, interactive
No directive = Server Component, can be async
```

**Database Choice:**
```
Client needs → PGLite
Server needs → Neon
Both need → Both (with sync)
```

**State Choice:**
```
Global app state → Zustand
Server data → SWR / React Query
URL state → nuqs
Local state → useState
```

**API Choice:**
```
Type-safe needed → tRPC
REST needed → Next.js API routes
Server function → Server Actions
```

---

## Summary

**LobeHub Architecture in One Sentence:**

> A Next.js 16 monorepo using React 19 Server Components, tRPC for type-safe APIs, dual database strategy (PGLite + Neon) for offline-first functionality, deployed on edge with serverless PostgreSQL.

**Key Innovations:**
1. 🚀 WASM PostgreSQL in browser (PGLite)
2. 🔒 End-to-end type safety (tRPC)
3. 📦 Monorepo with 19 shared packages
4. ⚡ Edge-first architecture
5. 🌐 Offline-first, cloud-optional

**Tech Stack (Nov 2025):**
- ✅ All technologies are current and production-ready
- ✅ No deprecated patterns in use
- ✅ Following 2025 best practices

---

**Document Status:** ✅ Complete (Part 2 of 2) | **Last Updated:** November 20, 2025

**[← Back to Part 1](./ARCHITECTURE-OVERVIEW-PART-01.md)** | **Next:** [Project Structure](./PROJECT_STRUCTURE.md)
