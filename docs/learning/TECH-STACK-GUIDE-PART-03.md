# Tech Stack Guide - Part 3 of 4

**Documented:** November 2025
**Tech Stack Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)
**Target:** Mid-level developers proficient in JS/TS/React

**📄 This is Part 3 of 4** | **[← Part 2](./TECH-STACK-GUIDE-PART-02.md)** | **[Part 4 →](./TECH-STACK-GUIDE-PART-04.md)**

This guide continues the technology stack overview, covering backend APIs and database technologies.

**Prerequisites:**
- **[Part 1](./TECH-STACK-GUIDE-PART-01.md)** - Frontend core technologies
- **[Part 2](./TECH-STACK-GUIDE-PART-02.md)** - State management and data fetching

**Time to read:** 30 minutes (Part 3)

---

## Table of Contents - Part 3

1. [Backend APIs](#backend-apis)
   - [tRPC 11.7.1](#trpc-1171)
   - [Next.js API Routes](#nextjs-api-routes)
2. [Database](#database)
   - [Drizzle ORM 0.44.7](#drizzle-orm-0447)
   - [PGLite 0.2.12](#pglite-0212)
   - [Neon PostgreSQL](#neon-postgresql)
3. [Database Strategy](#database-strategy)

**[← Part 2](./TECH-STACK-GUIDE-PART-02.md)** covered:
- State Management (Zustand, nuqs)
- Data Fetching (SWR, TanStack Query)
- Internationalization (react-i18next)
- Utilities (aHooks, dayjs, lodash-es)

**[Part 4 →](./TECH-STACK-GUIDE-PART-04.md)** will cover:
- Testing (Vitest, Playwright)
- Build Tools (Turbopack, Bun)
- Infrastructure & Deployment

---

## Backend APIs

### tRPC 11.7.1

**✅ CURRENT** | **Released:** November 2024 | **Next.js 16 Compatible:** Yes

**What is it?**
tRPC enables end-to-end type-safe APIs without code generation or manual type definitions.

**Why tRPC?**

**Problem with REST APIs:**

```typescript
// ❌ REST: Manual types, can drift from reality
// server/api/user.ts
export async function GET(req: Request) {
  const user = await db.query.users.findFirst();
  return Response.json(user);
}

// client
interface User {  // Manual type definition!
  id: string;
  name: string;
  email: string;
}

async function getUser(): Promise<User> {
  const res = await fetch('/api/user');
  return res.json(); // Hope this matches User interface!
}
```

**Solution with tRPC:**

```typescript
// ✅ tRPC: Automatic type safety, no code generation
// server/routers/user.ts
export const userRouter = router({
  getUser: publicProcedure
    .input(z.object({ id: z.string() }))
    .query(async ({ input }) => {
      return await db.query.users.findFirst({
        where: eq(users.id, input.id),
      });
    }),
});

// client
const { data } = trpc.user.getUser.useQuery({ id: '123' });
//    ^? User | undefined (type automatically inferred!)

// TypeScript KNOWS:
// - data has id, name, email properties
// - data can be undefined (loading)
// - input must be { id: string }
```

**🌉 Bridge from REST:** Like REST, but with automatic TypeScript types end-to-end!

---

**Key Features:**

**1. Type-Safe Procedures**

```typescript
import { router, publicProcedure } from '@/server/trpc';
import { z } from 'zod';

export const messageRouter = router({
  // Query (fetch data)
  list: publicProcedure
    .input(z.object({
      limit: z.number().optional(),
    }))
    .query(async ({ input }) => {
      return await db.query.messages.findMany({
        limit: input.limit ?? 10,
      });
    }),

  // Mutation (modify data)
  create: publicProcedure
    .input(z.object({
      content: z.string(),
    }))
    .mutation(async ({ input }) => {
      return await db.insert(messages).values({
        content: input.content,
      });
    }),
});
```

**Client usage:**

```typescript
// Query
const { data, isLoading } = trpc.message.list.useQuery({ limit: 20 });
//    ^? Message[] | undefined (fully typed!)

// Mutation
const mutation = trpc.message.create.useMutation();
await mutation.mutateAsync({ content: 'Hello!' });
```

---

**2. Input Validation with Zod**

tRPC uses Zod for runtime validation:

```typescript
const createUser = publicProcedure
  .input(
    z.object({
      name: z.string().min(1).max(100),
      email: z.string().email(),
      age: z.number().int().min(0).max(150).optional(),
    })
  )
  .mutation(async ({ input }) => {
    // input is validated and typed!
    return await db.insert(users).values(input);
  });

// Client: TypeScript enforces correct types
mutation.mutate({
  name: 'Alice',
  email: 'alice@example.com',
  age: 25,
});

// ❌ TypeScript error:
mutation.mutate({
  name: '',          // Error: name must be non-empty
  email: 'invalid',  // Error: invalid email
  age: -5,           // Error: age must be >= 0
});
```

**🎯 Benefit:** Validation happens both compile-time (TypeScript) and runtime (Zod)!

---

**3. Context and Middleware**

**Context** provides data to procedures (auth, database, etc.):

```typescript
// server/trpc.ts
export const createContext = async ({ req, res }) => {
  const session = await getSession(req);

  return {
    session,
    db: serverDB,
  };
};

// server/routers/user.ts
export const userRouter = router({
  me: publicProcedure
    .query(async ({ ctx }) => {
      // ctx has session and db!
      if (!ctx.session) throw new Error('Not authenticated');

      return await ctx.db.query.users.findFirst({
        where: eq(users.id, ctx.session.userId),
      });
    }),
});
```

**Middleware** for authentication:

```typescript
const isAuthed = middleware(async ({ ctx, next }) => {
  if (!ctx.session?.user) {
    throw new TRPCError({ code: 'UNAUTHORIZED' });
  }

  return next({
    ctx: {
      session: ctx.session,
      user: ctx.session.user,
    },
  });
});

export const protectedProcedure = publicProcedure.use(isAuthed);

// Usage:
export const userRouter = router({
  updateProfile: protectedProcedure  // Requires auth!
    .input(z.object({ name: z.string() }))
    .mutation(async ({ ctx, input }) => {
      // ctx.user is guaranteed to exist
      return await ctx.db.update(users)
        .set({ name: input.name })
        .where(eq(users.id, ctx.user.id));
    }),
});
```

---

**4. Router Organization**

LobeHub organizes routers by runtime and domain:

```
server/routers/
├── lambda/           ← Edge runtime (fast, serverless)
│   ├── user.ts
│   ├── message.ts
│   ├── agent.ts
│   └── index.ts      ← Combined router
│
├── async/            ← Node runtime (background jobs)
│   ├── fileUpload.ts
│   └── index.ts
│
├── desktop/          ← Desktop-specific APIs
│   ├── system.ts
│   └── index.ts
│
└── index.ts          ← Root router
```

**Root router combines all:**

```typescript
// server/routers/index.ts
import { router } from '../trpc';
import { lambdaRouter } from './lambda';
import { asyncRouter } from './async';
import { desktopRouter } from './desktop';

export const appRouter = router({
  lambda: lambdaRouter,
  async: asyncRouter,
  desktop: desktopRouter,
});

export type AppRouter = typeof appRouter;
```

**Client setup:**

```typescript
// utils/trpc.ts
import { createTRPCReact } from '@trpc/react-query';
import type { AppRouter } from '@/server/routers';

export const trpc = createTRPCReact<AppRouter>();
```

---

**5. Streaming (SSE)**

tRPC 11 supports streaming for real-time data:

```typescript
// Server
export const chatRouter = router({
  streamMessage: publicProcedure
    .input(z.object({ prompt: z.string() }))
    .subscription(async function* ({ input }) {
      const stream = await openai.chat.completions.create({
        messages: [{ role: 'user', content: input.prompt }],
        stream: true,
      });

      for await (const chunk of stream) {
        yield chunk.choices[0]?.delta?.content || '';
      }
    }),
});

// Client
const subscription = trpc.chat.streamMessage.useSubscription(
  { prompt: 'Hello!' },
  {
    onData: (chunk) => {
      console.log('Received:', chunk);
    },
  }
);
```

**🆕 NEW IN 2025:** Native streaming support in tRPC 11

---

**Learning Resources:**

- [tRPC Documentation](https://trpc.io/docs)
- [tRPC with Next.js](https://trpc.io/docs/client/nextjs)
- [tRPC Best Practices](https://trpc.io/docs/server/procedures)

**🚨 Migration from tRPC 10:**
- `router()` replaces `createRouter()`
- TanStack Query v5 required (was v4)
- Subscriptions API redesigned

📚 **More details:** [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md)

---

### Next.js API Routes

**✅ CURRENT** | **Part of:** Next.js 16

**What is it?**
Next.js API Routes provide REST API endpoints within your Next.js app.

**When to use API Routes vs tRPC?**

| Use Case | Technology |
|----------|------------|
| **Type-safe internal APIs** | tRPC (preferred) |
| **Webhooks from external services** | API Routes |
| **Public REST API** | API Routes |
| **File uploads** | API Routes |
| **OAuth callbacks** | API Routes |

**🎯 LobeHub uses:** tRPC for internal APIs, API Routes for webhooks and public endpoints

---

**API Route Structure:**

```
app/(backend)/
├── api/
│   ├── auth/           ← NextAuth.js endpoints
│   │   └── [...nextauth]/
│   │       └── route.ts
│   │
│   └── webhooks/       ← External webhooks
│       ├── stripe/
│       │   └── route.ts
│       └── github/
│           └── route.ts
│
├── trpc/               ← tRPC handler
│   └── [trpc]/
│       └── route.ts
│
└── webapi/             ← Legacy REST APIs
    ├── chat/
    │   └── route.ts
    └── tts/
        └── route.ts
```

---

**Example API Route:**

```typescript
// app/api/webhooks/stripe/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  const body = await req.json();

  // Verify webhook signature
  const signature = req.headers.get('stripe-signature');
  // ... verification logic

  // Process webhook
  await handleStripeWebhook(body);

  return NextResponse.json({ received: true });
}
```

---

**Edge vs Node Runtime:**

```typescript
// app/api/edge/route.ts
export const runtime = 'edge'; // Edge runtime (fast, limited APIs)

export async function GET() {
  return new Response('Hello from Edge!');
}

// app/api/node/route.ts
export const runtime = 'nodejs'; // Node runtime (full Node.js APIs)

import fs from 'fs'; // ✅ Works in Node runtime
                     // ❌ Doesn't work in Edge runtime

export async function GET() {
  const data = fs.readFileSync('data.json');
  return new Response(data);
}
```

**Choose Edge when:**
- ✅ Fast response required
- ✅ Deploying to Vercel Edge
- ✅ No file system / Node.js APIs needed

**Choose Node when:**
- ✅ Need file system access
- ✅ Need full Node.js APIs
- ✅ Long-running tasks (background jobs)

---

## Database

### Drizzle ORM 0.44.7

**✅ CURRENT** | **Released:** November 2024 | **TypeScript-first:** Yes

**What is it?**
Drizzle ORM is a lightweight, TypeScript-first ORM with zero dependencies and excellent DX.

**Why Drizzle?**

**Alternatives considered:**
- Prisma: Popular, but heavy runtime overhead
- TypeORM: Good, but complex decorators
- Kysely: Excellent, but query builder only (no schema)

**Drizzle wins:**
- ✅ TypeScript-first (types from schema)
- ✅ Zero runtime overhead
- ✅ SQL-like API (familiar)
- ✅ Migrations built-in
- ✅ Both query builder AND schema

---

**Schema Definition:**

```typescript
// packages/database/src/schemas/user.ts
import { pgTable, text, timestamp, uuid } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: uuid('id').defaultRandom().primaryKey(),
  name: text('name').notNull(),
  email: text('email').notNull().unique(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

// Infer TypeScript type from schema
export type User = typeof users.$inferSelect;
export type NewUser = typeof users.$inferInsert;
```

**🎯 Benefit:** Schema = source of truth for both database AND TypeScript types!

---

**Queries:**

**1. Select**

```typescript
import { db } from '@lobechat/database/client';
import { users } from '@lobechat/database/schemas/user';
import { eq } from 'drizzle-orm';

// Find all
const allUsers = await db.query.users.findMany();

// Find one
const user = await db.query.users.findFirst({
  where: eq(users.id, '123'),
});

// Find with relations
const userWithPosts = await db.query.users.findFirst({
  where: eq(users.id, '123'),
  with: {
    posts: true,
  },
});
```

---

**2. Insert**

```typescript
import { db } from '@lobechat/database/client';
import { users } from '@lobechat/database/schemas/user';

// Insert one
const newUser = await db.insert(users).values({
  name: 'Alice',
  email: 'alice@example.com',
}).returning();

// Insert multiple
const newUsers = await db.insert(users).values([
  { name: 'Bob', email: 'bob@example.com' },
  { name: 'Charlie', email: 'charlie@example.com' },
]).returning();
```

---

**3. Update**

```typescript
import { db } from '@lobechat/database/client';
import { users } from '@lobechat/database/schemas/user';
import { eq } from 'drizzle-orm';

const updatedUser = await db.update(users)
  .set({ name: 'Alice Updated' })
  .where(eq(users.id, '123'))
  .returning();
```

---

**4. Delete**

```typescript
import { db } from '@lobechat/database/client';
import { users } from '@lobechat/database/schemas/user';
import { eq } from 'drizzle-orm';

await db.delete(users)
  .where(eq(users.id, '123'));
```

---

**5. Relations**

Define relations between tables:

```typescript
// schemas/user.ts
export const users = pgTable('users', {
  id: uuid('id').defaultRandom().primaryKey(),
  name: text('name').notNull(),
});

export const usersRelations = relations(users, ({ many }) => ({
  posts: many(posts),
}));

// schemas/post.ts
export const posts = pgTable('posts', {
  id: uuid('id').defaultRandom().primaryKey(),
  title: text('title').notNull(),
  userId: uuid('user_id').notNull().references(() => users.id),
});

export const postsRelations = relations(posts, ({ one }) => ({
  author: one(users, {
    fields: [posts.userId],
    references: [users.id],
  }),
}));

// Query with relations:
const userWithPosts = await db.query.users.findFirst({
  with: {
    posts: true,
  },
});
//    ^? { id, name, posts: Post[] }
```

---

**Migrations:**

```bash
# Generate migration
bunx drizzle-kit generate

# Run migration
bunx drizzle-kit migrate
```

**Migration file example:**

```sql
-- migrations/0001_create_users.sql
CREATE TABLE "users" (
  "id" uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  "name" text NOT NULL,
  "email" text NOT NULL UNIQUE,
  "created_at" timestamp DEFAULT now() NOT NULL,
  "updated_at" timestamp DEFAULT now() NOT NULL
);
```

---

**LobeHub Database Architecture:**

```
packages/database/
├── src/
│   ├── schemas/        ← Drizzle table schemas
│   │   ├── user.ts
│   │   ├── message.ts
│   │   └── ...
│   │
│   ├── models/         ← CRUD operations (uses Drizzle)
│   │   ├── user.ts
│   │   └── ...
│   │
│   ├── repositories/   ← Complex queries (uses Drizzle)
│   │   ├── user.ts
│   │   └── ...
│   │
│   ├── client/         ← PGLite instance
│   └── server/         ← Neon instance
```

**🎯 Pattern:**
- **Schemas** = Table definitions
- **Models** = Basic CRUD (create, read, update, delete)
- **Repositories** = Complex queries (joins, aggregations, etc.)

📚 **More details:**
- [DATABASE_ARCHITECTURE.md](./DATABASE_ARCHITECTURE.md)
- [.cursor/rules/drizzle-schema-style-guide.mdc](./.cursor/rules/drizzle-schema-style-guide.mdc)

---

**Learning Resources:**

- [Drizzle ORM Documentation](https://orm.drizzle.team/docs/overview)
- [Drizzle with PostgreSQL](https://orm.drizzle.team/docs/get-started-postgresql)

---

### PGLite 0.2.12

**✅ CURRENT** | **Released:** November 2024 | **WASM PostgreSQL:** Yes

**What is it?**
PGLite is a WASM (WebAssembly) build of PostgreSQL that runs in the browser and Node.js.

**Why PGLite?**

**Problem:**
Traditional web apps require a server for database queries:

```
Browser → Network → Server → Database → Network → Browser
         (slow)            (latency)    (slow)
```

**Solution with PGLite:**

```
Browser → PGLite (in browser, instant!)
```

**Benefits:**
- ✅ **Offline-first** - App works without internet
- ✅ **Instant queries** - No network latency
- ✅ **Full PostgreSQL** - Same SQL as server PostgreSQL
- ✅ **Privacy** - Data stays on device

**🚀 Innovation:** LobeHub is one of the first major apps using WASM PostgreSQL in production!

---

**Usage:**

```typescript
// Client-side (browser)
import { clientDB } from '@lobechat/database/client';

// Works exactly like PostgreSQL!
const messages = await clientDB.query.messages.findMany();

const newMessage = await clientDB.insert(messages).values({
  content: 'Hello from PGLite!',
}).returning();
```

**No difference from server PostgreSQL!** Same queries, same schema.

---

**PGLite Setup:**

```typescript
// packages/database/src/client/index.ts
import { PGlite } from '@electric-sql/pglite';
import { drizzle } from 'drizzle-orm/pglite';
import * as schema from '../schemas';

// Create PGLite instance (runs in browser)
const pglite = new PGlite('idb://lobechat', {
  // Store in IndexedDB for persistence
});

// Wrap with Drizzle ORM
export const clientDB = drizzle(pglite, { schema });
```

---

**Storage:**

PGLite can store data in:

1. **IndexedDB** (default) - Persistent across sessions
   ```typescript
   new PGlite('idb://lobechat');
   ```

2. **Memory** - Cleared on page reload
   ```typescript
   new PGlite();
   ```

3. **File System (Node.js)** - Desktop app
   ```typescript
   new PGlite('./data/lobechat.db');
   ```

---

**Limitations:**

- ❌ No extensions (pg_trgm, etc.)
- ❌ Limited concurrent connections
- ❌ No replication (use sync to server)

**✅ Perfect for:** Client-side apps with offline-first requirements

---

**Learning Resources:**

- [PGLite Documentation](https://pglite.dev/)
- [PGLite GitHub](https://github.com/electric-sql/pglite)

---

### Neon PostgreSQL

**✅ CURRENT** | **Serverless:** Yes | **Compatible with:** PostgreSQL 17

**What is it?**
Neon is a serverless PostgreSQL database with auto-scaling and branching.

**Why Neon?**

**Alternatives considered:**
- Supabase: Good, but heavier runtime
- PlanetScale: Good, but MySQL (not PostgreSQL)
- Vercel Postgres: Good, but Neon-powered anyway
- AWS RDS: Good, but not serverless

**Neon wins:**
- ✅ True serverless (scale to zero)
- ✅ Instant database branching
- ✅ PostgreSQL 17 compatible
- ✅ Generous free tier
- ✅ Low latency (global)

---

**Key Features:**

**1. Serverless**

Neon automatically:
- Scales to zero when idle (no cost!)
- Scales up on demand
- Handles connection pooling

```typescript
// No connection management needed!
import { serverDB } from '@lobechat/database/server';

const users = await serverDB.query.users.findMany();
// Neon handles:
// - Connection pooling
// - Auto-scaling
// - High availability
```

---

**2. Database Branching**

Create database branches like Git:

```bash
# Create branch for PR
neon branches create --name pr-123

# Run tests on branch
DATABASE_URL=<branch-url> npm test

# Merge to main when PR approved
```

**🎯 Benefit:** Test migrations safely without affecting production!

---

**3. Time Travel**

Restore database to any point in the past 7 days:

```bash
# Restore to 2 hours ago
neon branches create --name recovery --restore-to "2 hours ago"
```

---

**Neon Setup:**

```typescript
// packages/database/src/server/index.ts
import { neon } from '@neondatabase/serverless';
import { drizzle } from 'drizzle-orm/neon-http';
import * as schema from '../schemas';

const sql = neon(process.env.DATABASE_URL!);

export const serverDB = drizzle(sql, { schema });
```

**Environment:**

```bash
# .env.local
DATABASE_URL=postgresql://user:pass@ep-xxx.neon.tech/dbname
```

---

**Connection Pooling:**

Neon provides built-in connection pooling:

```
Your App → Neon Proxy → Connection Pool → PostgreSQL
           (handles)     (reuses)          (efficient)
```

**No manual pooling needed!**

---

**Learning Resources:**

- [Neon Documentation](https://neon.tech/docs/introduction)
- [Neon with Drizzle](https://neon.tech/docs/guides/drizzle)

---

## Database Strategy

### Dual Database Architecture

LobeHub uses **both** PGLite and Neon for an offline-first, cloud-optional experience:

```
┌─────────────────────────────────────┐
│           Browser                   │
│                                     │
│  ┌─────────────────────────────┐   │
│  │  PGLite (Client Database)   │   │
│  │  • Instant queries           │   │
│  │  • Offline support           │   │
│  │  • Privacy                   │   │
│  └─────────────────────────────┘   │
│              ↕                      │
│        (Sync when online)           │
│              ↕                      │
│  ┌─────────────────────────────┐   │
│  │  tRPC Client                │   │
│  └─────────────────────────────┘   │
└─────────────────────────────────────┘
              ↓
         (Network)
              ↓
┌─────────────────────────────────────┐
│          Server (Edge)              │
│                                     │
│  ┌─────────────────────────────┐   │
│  │  tRPC Server                │   │
│  └─────────────────────────────┘   │
│              ↓                      │
│  ┌─────────────────────────────┐   │
│  │  Neon (Server Database)     │   │
│  │  • Multi-device sync         │   │
│  │  • Backup                    │   │
│  │  • Collaboration             │   │
│  └─────────────────────────────┘   │
└─────────────────────────────────────┘
```

---

### When to Use Each Database

**Use PGLite (Client) when:**
- ✅ User is browsing (read operations)
- ✅ User is creating/editing (optimistic updates)
- ✅ User is offline
- ✅ Privacy-sensitive data

**Use Neon (Server) when:**
- ✅ User needs cross-device sync
- ✅ User needs collaboration
- ✅ User needs backup
- ✅ Long-term storage required

---

### Sync Strategy

**Pattern: Optimistic Updates + Background Sync**

```typescript
// 1. Update PGLite immediately (optimistic)
await clientDB.insert(messages).values({ content: 'Hello!' });

// 2. Sync to server in background
await trpc.message.create.mutate({ content: 'Hello!' });

// 3. Server saves to Neon
await serverDB.insert(messages).values({ content: 'Hello!' });

// 4. Server returns success
// 5. Client marks as synced
```

**🎯 Result:** Instant UI updates, reliable persistence!

---

**Learning Resources:**

- [Offline-First Architecture](https://www.inkandswitch.com/local-first/)
- [Database Sync Strategies](https://electric-sql.com/)

📚 **More details:** [DATABASE_ARCHITECTURE.md](./DATABASE_ARCHITECTURE.md)

---

## Summary - Part 3

**Technologies Covered:**

| Technology | Version | Purpose | Status |
|------------|---------|---------|--------|
| **tRPC** | 11.7.1 | Type-safe APIs | ✅ CURRENT |
| **Next.js API Routes** | (Next.js 16) | REST APIs, webhooks | ✅ CURRENT |
| **Drizzle ORM** | 0.44.7 | TypeScript-first ORM | ✅ CURRENT |
| **PGLite** | 0.2.12 | WASM PostgreSQL (client) | ✅ CURRENT |
| **Neon** | PostgreSQL 17 | Serverless PostgreSQL (server) | ✅ CURRENT |

**Key Takeaways:**

1. 🔒 **tRPC** - End-to-end type safety with zero code generation
2. ⚡ **Edge Runtime** - Fast API responses with serverless functions
3. 🗄️ **Drizzle ORM** - TypeScript types from database schema
4. 💻 **PGLite** - Full PostgreSQL in browser (offline-first!)
5. ☁️ **Neon** - Serverless PostgreSQL with branching and auto-scaling
6. 🔄 **Dual Database** - Instant local queries + reliable cloud sync

---

**Document Status:** ✅ Part 3 Complete | **Last Updated:** November 20, 2025

**[← Part 2](./TECH-STACK-GUIDE-PART-02.md)** | **[Continue to Part 4 →](./TECH-STACK-GUIDE-PART-04.md)** - Testing, Build Tools, and Infrastructure
