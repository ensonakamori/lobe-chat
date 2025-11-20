# Backend Architecture - Part 1 of 2

**Documented:** November 2025
**Tech Stack Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)
**Target:** Mid-level developers proficient in JS/TS/React

**📄 This is Part 1 of 2** | **[Continue to Part 2 →](./BACKEND-ARCHITECTURE-PART-02.md)**

This comprehensive guide explains LobeHub's backend architecture, focusing on tRPC, API design, and server-side patterns.

**Prerequisites:**
- [Architecture Overview](./ARCHITECTURE-OVERVIEW-PART-01.md) - System architecture
- [Tech Stack Guide - Part 3](./TECH-STACK-GUIDE-PART-03.md) - tRPC, Database

**Time to read:** 30 minutes (Part 1)

---

## Table of Contents - Part 1

1. [Backend Architecture Principles](#backend-architecture-principles)
2. [tRPC Architecture](#trpc-architecture)
   - [Router Organization](#router-organization)
   - [Procedures](#procedures)
   - [Context and Middleware](#context-and-middleware)
3. [API Design Patterns](#api-design-patterns)
   - [Naming Conventions](#naming-conventions)
   - [Input Validation](#input-validation)
   - [Error Handling](#error-handling)
4. [Server Services](#server-services)

**[Part 2 →](./BACKEND-ARCHITECTURE-PART-02.md)** will cover:
- Database Integration
- Authentication & Authorization
- File Handling
- Background Jobs
- Streaming and Real-Time
- Performance Optimization

---

## Backend Architecture Principles

**LobeHub Backend Philosophy:**

> Build a **type-safe, performant, scalable** backend using tRPC, Next.js Edge Runtime, and PostgreSQL with end-to-end type safety.

**Core Principles:**

1. **🔒 Type Safety End-to-End**
   - No manual type definitions
   - Shared types between client and server
   - Compile-time error detection

2. **⚡ Edge-First Architecture**
   - Lambda router runs on Edge Runtime (fast, global)
   - Async router runs on Node Runtime (long-running tasks)
   - Choose runtime based on requirements

3. **🎯 Single Responsibility**
   - Routers handle routing only
   - Services contain business logic
   - Models handle database operations
   - Clear separation of concerns

4. **📦 Modular Design**
   - Organized by domain (user, message, agent)
   - Independent, composable modules
   - Easy to test and maintain

5. **🚀 Performance-First**
   - Edge deployment for low latency
   - Connection pooling
   - Caching strategies
   - Efficient database queries

---

## tRPC Architecture

### Router Organization

LobeHub organizes routers by **runtime** and **domain**:

```
src/server/routers/
├── lambda/              ← Edge Runtime (serverless, fast)
│   ├── user.ts          ← User operations
│   ├── message.ts       ← Message operations
│   ├── agent.ts         ← Agent operations
│   ├── conversation.ts  ← Conversation operations
│   └── index.ts         ← Combined lambda router
│
├── async/               ← Node Runtime (background jobs)
│   ├── fileUpload.ts    ← File processing
│   ├── generation.ts    ← AI generation tasks
│   └── index.ts         ← Combined async router
│
├── desktop/             ← Desktop-specific APIs
│   ├── system.ts        ← System operations
│   ├── window.ts        ← Window management
│   └── index.ts         ← Combined desktop router
│
├── tools/               ← Utility endpoints
│   ├── health.ts        ← Health checks
│   └── index.ts
│
└── index.ts             ← Root router (combines all)
```

**Why separate by runtime?**

| Runtime | Use Case | Limits | Performance |
|---------|----------|--------|-------------|
| **Edge** | Quick operations | 10s timeout, 50MB | <100ms latency globally |
| **Node** | Long tasks | No timeout limit | Standard latency |

---

**Root Router Pattern:**

```typescript
// src/server/routers/index.ts
import { router } from '../trpc';
import { lambdaRouter } from './lambda';
import { asyncRouter } from './async';
import { desktopRouter } from './desktop';
import { toolsRouter } from './tools';

export const appRouter = router({
  lambda: lambdaRouter,
  async: asyncRouter,
  desktop: desktopRouter,
  tools: toolsRouter,
});

export type AppRouter = typeof appRouter;
```

**Client usage:**

```typescript
// Client calls:
trpc.lambda.user.getUser.useQuery({ id: '123' });
trpc.async.fileUpload.processFile.useMutation();
trpc.desktop.system.getInfo.useQuery();
```

---

### Procedures

tRPC has three procedure types:

**1. Query** - Read operations (GET)

```typescript
// src/server/routers/lambda/user.ts
import { router, publicProcedure } from '@/server/trpc';
import { z } from 'zod';

export const userRouter = router({
  getUser: publicProcedure
    .input(z.object({
      id: z.string().uuid(),
    }))
    .query(async ({ input, ctx }) => {
      const user = await ctx.db.query.users.findFirst({
        where: eq(users.id, input.id),
      });

      if (!user) {
        throw new TRPCError({
          code: 'NOT_FOUND',
          message: 'User not found',
        });
      }

      return user;
    }),

  listUsers: publicProcedure
    .input(z.object({
      limit: z.number().min(1).max(100).optional(),
      offset: z.number().min(0).optional(),
    }))
    .query(async ({ input, ctx }) => {
      return await ctx.db.query.users.findMany({
        limit: input.limit ?? 10,
        offset: input.offset ?? 0,
      });
    }),
});
```

**Client usage:**

```typescript
const { data: user } = trpc.lambda.user.getUser.useQuery({ id: '123' });
const { data: users } = trpc.lambda.user.listUsers.useQuery({ limit: 20 });
```

---

**2. Mutation** - Write operations (POST/PUT/DELETE)

```typescript
export const userRouter = router({
  createUser: publicProcedure
    .input(z.object({
      name: z.string().min(1).max(100),
      email: z.string().email(),
    }))
    .mutation(async ({ input, ctx }) => {
      // Check if user exists
      const existing = await ctx.db.query.users.findFirst({
        where: eq(users.email, input.email),
      });

      if (existing) {
        throw new TRPCError({
          code: 'CONFLICT',
          message: 'User already exists',
        });
      }

      // Create user
      const [newUser] = await ctx.db.insert(users)
        .values({
          name: input.name,
          email: input.email,
        })
        .returning();

      return newUser;
    }),

  updateUser: publicProcedure
    .input(z.object({
      id: z.string().uuid(),
      name: z.string().min(1).max(100).optional(),
      email: z.string().email().optional(),
    }))
    .mutation(async ({ input, ctx }) => {
      const { id, ...updates } = input;

      const [updatedUser] = await ctx.db.update(users)
        .set(updates)
        .where(eq(users.id, id))
        .returning();

      if (!updatedUser) {
        throw new TRPCError({
          code: 'NOT_FOUND',
          message: 'User not found',
        });
      }

      return updatedUser;
    }),

  deleteUser: publicProcedure
    .input(z.object({
      id: z.string().uuid(),
    }))
    .mutation(async ({ input, ctx }) => {
      await ctx.db.delete(users)
        .where(eq(users.id, input.id));

      return { success: true };
    }),
});
```

**Client usage:**

```typescript
const createMutation = trpc.lambda.user.createUser.useMutation();
await createMutation.mutateAsync({ name: 'Alice', email: 'alice@example.com' });

const updateMutation = trpc.lambda.user.updateUser.useMutation();
await updateMutation.mutateAsync({ id: '123', name: 'Alice Updated' });

const deleteMutation = trpc.lambda.user.deleteUser.useMutation();
await deleteMutation.mutateAsync({ id: '123' });
```

---

**3. Subscription** - Real-time data (WebSocket/SSE)

```typescript
export const chatRouter = router({
  streamMessage: publicProcedure
    .input(z.object({
      prompt: z.string(),
      conversationId: z.string().uuid(),
    }))
    .subscription(async function* ({ input, ctx }) {
      // Stream from OpenAI
      const stream = await openai.chat.completions.create({
        model: 'gpt-4',
        messages: [{ role: 'user', content: input.prompt }],
        stream: true,
      });

      // Yield chunks as they arrive
      for await (const chunk of stream) {
        const content = chunk.choices[0]?.delta?.content || '';
        yield content;
      }
    }),
});
```

**Client usage:**

```typescript
const subscription = trpc.lambda.chat.streamMessage.useSubscription(
  { prompt: 'Hello!', conversationId: '123' },
  {
    onData: (chunk) => {
      console.log('Received:', chunk);
    },
    onError: (error) => {
      console.error('Error:', error);
    },
  }
);
```

**🆕 NEW IN 2025:** Native streaming support in tRPC 11!

---

### Context and Middleware

**Context** provides data to all procedures:

```typescript
// src/server/trpc.ts
import { inferAsyncReturnType } from '@trpc/server';
import { CreateNextContextOptions } from '@trpc/server/adapters/next';
import { getServerSession } from 'next-auth';
import { serverDB } from '@lobechat/database/server';

export async function createContext({ req, res }: CreateNextContextOptions) {
  // Get session from NextAuth
  const session = await getServerSession(req, res, authOptions);

  return {
    session,
    db: serverDB,
    req,
    res,
  };
}

export type Context = inferAsyncReturnType<typeof createContext>;
```

**All procedures receive context:**

```typescript
.query(async ({ input, ctx }) => {
  // ctx.session  ← User session
  // ctx.db       ← Database instance
  // ctx.req      ← Request object
  // ctx.res      ← Response object
});
```

---

**Middleware** adds logic before/after procedures:

**1. Authentication Middleware:**

```typescript
// src/server/trpc.ts
const isAuthed = t.middleware(async ({ ctx, next }) => {
  if (!ctx.session?.user) {
    throw new TRPCError({ code: 'UNAUTHORIZED' });
  }

  return next({
    ctx: {
      // Extend context with authenticated user
      session: ctx.session,
      user: ctx.session.user,
    },
  });
});

export const protectedProcedure = publicProcedure.use(isAuthed);
```

**Usage:**

```typescript
export const userRouter = router({
  // Anyone can call
  getPublicUsers: publicProcedure
    .query(async ({ ctx }) => {
      return await ctx.db.query.users.findMany();
    }),

  // Only authenticated users can call
  getMyProfile: protectedProcedure
    .query(async ({ ctx }) => {
      // ctx.user is guaranteed to exist!
      return await ctx.db.query.users.findFirst({
        where: eq(users.id, ctx.user.id),
      });
    }),
});
```

---

**2. Logging Middleware:**

```typescript
const logger = t.middleware(async ({ path, type, next }) => {
  const start = Date.now();
  const result = await next();
  const duration = Date.now() - start;

  console.log(`${type} ${path} - ${duration}ms`);

  return result;
});

export const loggedProcedure = publicProcedure.use(logger);
```

---

**3. Rate Limiting Middleware:**

```typescript
import { RateLimiterMemory } from 'rate-limiter-flexible';

const rateLimiter = new RateLimiterMemory({
  points: 10,  // 10 requests
  duration: 60,  // per 60 seconds
});

const rateLimit = t.middleware(async ({ ctx, next }) => {
  const ip = ctx.req.headers['x-forwarded-for'] || ctx.req.socket.remoteAddress;

  try {
    await rateLimiter.consume(ip);
  } catch {
    throw new TRPCError({
      code: 'TOO_MANY_REQUESTS',
      message: 'Rate limit exceeded',
    });
  }

  return next();
});

export const rateLimitedProcedure = publicProcedure.use(rateLimit);
```

---

**Composing Middleware:**

```typescript
// Combine multiple middleware
export const secureProtectedProcedure = publicProcedure
  .use(logger)        // Log all requests
  .use(rateLimit)     // Apply rate limiting
  .use(isAuthed);     // Require authentication

// Usage:
export const userRouter = router({
  sensitiveOperation: secureProtectedProcedure
    .mutation(async ({ ctx }) => {
      // Logged, rate-limited, and authenticated!
    }),
});
```

---

## API Design Patterns

### Naming Conventions

**Resource-oriented naming:**

```typescript
// ✅ GOOD: Clear, RESTful naming
export const userRouter = router({
  // Queries (read)
  getUser: publicProcedure.query(...),
  listUsers: publicProcedure.query(...),
  searchUsers: publicProcedure.query(...),

  // Mutations (write)
  createUser: publicProcedure.mutation(...),
  updateUser: publicProcedure.mutation(...),
  deleteUser: publicProcedure.mutation(...),
});

// ❌ BAD: Unclear naming
export const userRouter = router({
  user: publicProcedure.query(...),        // Get or list?
  users: publicProcedure.query(...),       // Same as above?
  add: publicProcedure.mutation(...),      // Add what?
  remove: publicProcedure.mutation(...),   // Remove what?
});
```

**Naming patterns:**

| Operation | Pattern | Example |
|-----------|---------|---------|
| **Get one** | `get{Resource}` | `getUser` |
| **Get many** | `list{Resources}` | `listUsers` |
| **Search** | `search{Resources}` | `searchUsers` |
| **Create** | `create{Resource}` | `createUser` |
| **Update** | `update{Resource}` | `updateUser` |
| **Delete** | `delete{Resource}` | `deleteUser` |
| **Batch** | `{operation}Many{Resources}` | `deleteManyUsers` |

---

### Input Validation

Use **Zod** for runtime validation:

**Basic validation:**

```typescript
import { z } from 'zod';

export const userRouter = router({
  createUser: publicProcedure
    .input(
      z.object({
        name: z.string()
          .min(1, 'Name is required')
          .max(100, 'Name is too long'),
        email: z.string()
          .email('Invalid email'),
        age: z.number()
          .int('Age must be an integer')
          .min(0, 'Age must be positive')
          .max(150, 'Age must be realistic')
          .optional(),
      })
    )
    .mutation(async ({ input }) => {
      // input is validated and typed!
    }),
});
```

---

**Advanced validation:**

```typescript
const userSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  password: z.string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain uppercase letter')
    .regex(/[a-z]/, 'Password must contain lowercase letter')
    .regex(/[0-9]/, 'Password must contain number'),
  role: z.enum(['user', 'admin', 'moderator']),
  metadata: z.record(z.string(), z.unknown()).optional(),
});

export const userRouter = router({
  createUser: publicProcedure
    .input(userSchema)
    .mutation(async ({ input }) => {
      // Fully validated!
    }),
});
```

---

**Reusable schemas:**

```typescript
// src/server/schemas/user.ts
export const userIdSchema = z.object({
  id: z.string().uuid('Invalid user ID'),
});

export const createUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
});

export const updateUserSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1).max(100).optional(),
  email: z.string().email().optional(),
});

// Usage:
export const userRouter = router({
  getUser: publicProcedure
    .input(userIdSchema)
    .query(...),

  createUser: publicProcedure
    .input(createUserSchema)
    .mutation(...),

  updateUser: publicProcedure
    .input(updateUserSchema)
    .mutation(...),
});
```

---

### Error Handling

**tRPC Error Codes:**

```typescript
import { TRPCError } from '@trpc/server';

// Standard error codes:
throw new TRPCError({ code: 'UNAUTHORIZED' });        // 401
throw new TRPCError({ code: 'FORBIDDEN' });           // 403
throw new TRPCError({ code: 'NOT_FOUND' });           // 404
throw new TRPCError({ code: 'CONFLICT' });            // 409
throw new TRPCError({ code: 'BAD_REQUEST' });         // 400
throw new TRPCError({ code: 'INTERNAL_SERVER_ERROR' }); // 500
throw new TRPCError({ code: 'TOO_MANY_REQUESTS' });   // 429
```

---

**Error Handling Pattern:**

```typescript
export const userRouter = router({
  getUser: publicProcedure
    .input(z.object({ id: z.string().uuid() }))
    .query(async ({ input, ctx }) => {
      try {
        const user = await ctx.db.query.users.findFirst({
          where: eq(users.id, input.id),
        });

        if (!user) {
          throw new TRPCError({
            code: 'NOT_FOUND',
            message: 'User not found',
          });
        }

        return user;
      } catch (error) {
        // Database errors
        if (error instanceof DatabaseError) {
          throw new TRPCError({
            code: 'INTERNAL_SERVER_ERROR',
            message: 'Database error',
            cause: error,
          });
        }

        // Re-throw tRPC errors
        if (error instanceof TRPCError) {
          throw error;
        }

        // Unknown errors
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'An unexpected error occurred',
        });
      }
    }),
});
```

---

**Client-side error handling:**

```typescript
const { data, error, isLoading } = trpc.lambda.user.getUser.useQuery(
  { id: '123' },
  {
    retry: 3,  // Retry 3 times
    retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
  }
);

if (error) {
  if (error.data?.code === 'NOT_FOUND') {
    return <div>User not found</div>;
  }
  if (error.data?.code === 'UNAUTHORIZED') {
    return <div>Please log in</div>;
  }
  return <div>Error: {error.message}</div>;
}
```

---

## Server Services

Server services contain **business logic** and are called by routers:

```
src/server/services/
├── message/
│   ├── index.ts              ← MessageService class
│   └── __tests__/
│       └── index.test.ts     ← Tests
│
├── user/
│   ├── index.ts              ← UserService class
│   └── __tests__/
│
├── agent/
│   ├── index.ts
│   └── __tests__/
│
└── file/
    ├── index.ts
    └── __tests__/
```

---

**Service Pattern:**

```typescript
// src/server/services/user/index.ts
import { serverDB } from '@lobechat/database/server';
import { users } from '@lobechat/database/schemas/user';
import { eq } from 'drizzle-orm';
import { TRPCError } from '@trpc/server';

export class UserService {
  static async getUser(id: string) {
    const user = await serverDB.query.users.findFirst({
      where: eq(users.id, id),
    });

    if (!user) {
      throw new TRPCError({
        code: 'NOT_FOUND',
        message: 'User not found',
      });
    }

    return user;
  }

  static async createUser(data: { name: string; email: string }) {
    // Check if user exists
    const existing = await serverDB.query.users.findFirst({
      where: eq(users.email, data.email),
    });

    if (existing) {
      throw new TRPCError({
        code: 'CONFLICT',
        message: 'User already exists',
      });
    }

    // Create user
    const [newUser] = await serverDB.insert(users)
      .values(data)
      .returning();

    return newUser;
  }

  static async updateUser(id: string, updates: Partial<{ name: string; email: string }>) {
    const [updatedUser] = await serverDB.update(users)
      .set(updates)
      .where(eq(users.id, id))
      .returning();

    if (!updatedUser) {
      throw new TRPCError({
        code: 'NOT_FOUND',
        message: 'User not found',
      });
    }

    return updatedUser;
  }

  static async deleteUser(id: string) {
    await serverDB.delete(users)
      .where(eq(users.id, id));
  }

  static async listUsers(options: { limit?: number; offset?: number } = {}) {
    return await serverDB.query.users.findMany({
      limit: options.limit ?? 10,
      offset: options.offset ?? 0,
    });
  }
}
```

---

**Using Service in Router:**

```typescript
// src/server/routers/lambda/user.ts
import { router, publicProcedure } from '@/server/trpc';
import { UserService } from '@/server/services/user';
import { z } from 'zod';

export const userRouter = router({
  getUser: publicProcedure
    .input(z.object({ id: z.string().uuid() }))
    .query(async ({ input }) => {
      return await UserService.getUser(input.id);
    }),

  createUser: publicProcedure
    .input(z.object({
      name: z.string().min(1).max(100),
      email: z.string().email(),
    }))
    .mutation(async ({ input }) => {
      return await UserService.createUser(input);
    }),

  updateUser: publicProcedure
    .input(z.object({
      id: z.string().uuid(),
      name: z.string().min(1).max(100).optional(),
      email: z.string().email().optional(),
    }))
    .mutation(async ({ input }) => {
      const { id, ...updates } = input;
      return await UserService.updateUser(id, updates);
    }),

  deleteUser: publicProcedure
    .input(z.object({ id: z.string().uuid() }))
    .mutation(async ({ input }) => {
      await UserService.deleteUser(input.id);
      return { success: true };
    }),

  listUsers: publicProcedure
    .input(z.object({
      limit: z.number().min(1).max(100).optional(),
      offset: z.number().min(0).optional(),
    }))
    .query(async ({ input }) => {
      return await UserService.listUsers(input);
    }),
});
```

**🎯 Benefits:**
- ✅ Business logic separated from routing
- ✅ Easy to test (test service directly)
- ✅ Reusable across multiple routers
- ✅ Clear separation of concerns

---

## Summary - Part 1

**Backend Architecture in One Paragraph:**

> LobeHub's backend uses tRPC for type-safe APIs organized by runtime (Edge/Node) and domain (user/message/agent). Routers handle routing and validation, services contain business logic, and middleware adds cross-cutting concerns. All code is fully type-safe from client to database with zero code generation.

**Key Patterns:**

1. 🏗️ **Router Organization** - Separate by runtime (lambda/async) and domain
2. 🔒 **Type Safety** - Zod validation + TypeScript inference
3. 🎯 **Procedures** - Query (read), Mutation (write), Subscription (real-time)
4. 🛡️ **Middleware** - Authentication, logging, rate limiting
5. 📦 **Services** - Business logic separated from routing
6. ⚡ **Edge-First** - Fast global responses for most operations

---

**Document Status:** ✅ Part 1 Complete | **Last Updated:** November 20, 2025

**[Continue to Part 2 →](./BACKEND-ARCHITECTURE-PART-02.md)** - Database Integration, Auth, File Handling, Background Jobs, Streaming
