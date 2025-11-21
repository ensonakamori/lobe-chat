# Database Architecture - Part 2 of 2

**Documented:** November 2025
**Tech Stack Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)
**Target:** Mid-level developers proficient in JS/TS/React

**📄 This is Part 2 of 2** | **[← Back to Part 1](./DATABASE-ARCHITECTURE-PART-01.md)**

This document continues the database architecture guide, covering migrations, relationships, query patterns, and optimization.

**Prerequisites:**
- **[Part 1](./DATABASE-ARCHITECTURE-PART-01.md)** - Read this first!

**Time to read:** 30 minutes (Part 2)

---

## Table of Contents - Part 2

1. [Migrations](#migrations)
2. [Relationships and Joins](#relationships-and-joins)
3. [Query Patterns](#query-patterns)
4. [Performance Optimization](#performance-optimization)
5. [Best Practices](#best-practices)
6. [Summary](#summary)

**[← Part 1](./DATABASE-ARCHITECTURE-PART-01.md)** covered:
- Database Architecture Principles
- Dual Database Strategy (PGLite + Neon)
- Schema Design with Drizzle
- Database Layer Organization (Schemas, Models, Repositories)
- Common Patterns

---

## Migrations

### Creating Migrations

**Drizzle Kit** generates migrations from schema changes:

```bash
# Generate migration
bunx drizzle-kit generate

# This creates:
# packages/database/src/migrations/0001_create_users.sql
# packages/database/src/migrations/meta/_journal.json
```

---

**Migration File Example:**

```sql
-- packages/database/src/migrations/0001_create_users.sql
CREATE TABLE "users" (
  "id" uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  "name" text NOT NULL,
  "email" text NOT NULL UNIQUE,
  "email_verified" boolean DEFAULT false NOT NULL,
  "image" text,
  "created_at" timestamp DEFAULT now() NOT NULL,
  "updated_at" timestamp DEFAULT now() NOT NULL
);

CREATE INDEX "users_email_idx" ON "users" ("email");
```

---

### Running Migrations

**Server (Neon):**

```bash
# Run migrations on Neon
bunx drizzle-kit migrate
```

**Client (PGLite):**

```typescript
// packages/database/src/client/index.ts
import { PGlite } from '@electric-sql/pglite';
import { drizzle } from 'drizzle-orm/pglite';
import { migrate } from 'drizzle-orm/pglite/migrator';
import * as schema from '../schemas';

export const pglite = new PGlite('idb://lobechat');
export const clientDB = drizzle(pglite, { schema });

// Run migrations on app load
export async function initClientDB() {
  await migrate(clientDB, {
    migrationsFolder: './src/migrations',
  });
}
```

---

### Migration Best Practices

**1. Small, Incremental Changes**

```bash
# ✅ GOOD: One change per migration
0001_create_users.sql
0002_add_users_role.sql
0003_create_messages.sql

# ❌ BAD: Too many changes
0001_initial_schema.sql  # Creates 20 tables!
```

---

**2. Always Add Indexes in Migrations**

```sql
-- ✅ GOOD: Index added in same migration
CREATE TABLE "messages" (
  "id" uuid PRIMARY KEY,
  "conversation_id" uuid NOT NULL,
  "user_id" uuid NOT NULL,
  "content" text NOT NULL,
  "created_at" timestamp DEFAULT now() NOT NULL
);

CREATE INDEX "messages_conversation_id_idx" ON "messages" ("conversation_id");
CREATE INDEX "messages_user_id_idx" ON "messages" ("user_id");
```

---

**3. Handle Data Migrations**

```sql
-- Migration: Add role column with default
ALTER TABLE "users"
ADD COLUMN "role" text NOT NULL DEFAULT 'user';

-- Update existing users
UPDATE "users"
SET "role" = 'admin'
WHERE "email" LIKE '%@company.com';
```

---

**4. Make Migrations Reversible**

```sql
-- Up migration
ALTER TABLE "users" ADD COLUMN "phone" text;

-- Down migration (manual, for reference)
-- ALTER TABLE "users" DROP COLUMN "phone";
```

---

### Drizzle Configuration

```typescript
// drizzle.config.ts
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  schema: './packages/database/src/schemas/index.ts',
  out: './packages/database/src/migrations',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

---

## Relationships and Joins

### Defining Relationships

**One-to-Many: User has many Messages**

```typescript
// packages/database/src/schemas/user.ts
import { relations } from 'drizzle-orm';
import { messages } from './message';

export const users = pgTable('users', {
  id: uuid('id').defaultRandom().primaryKey(),
  name: text('name').notNull(),
});

export const usersRelations = relations(users, ({ many }) => ({
  messages: many(messages),
}));

// packages/database/src/schemas/message.ts
import { relations } from 'drizzle-orm';
import { users } from './user';

export const messages = pgTable('messages', {
  id: uuid('id').defaultRandom().primaryKey(),
  userId: uuid('user_id')
    .notNull()
    .references(() => users.id, { onDelete: 'cascade' }),
  content: text('content').notNull(),
});

export const messagesRelations = relations(messages, ({ one }) => ({
  user: one(users, {
    fields: [messages.userId],
    references: [users.id],
  }),
}));
```

---

**Query with Relations:**

```typescript
// Get user with all messages
const user = await db.query.users.findFirst({
  where: eq(users.id, 'user-id'),
  with: {
    messages: true,  // Include all messages
  },
});
// Result:
// {
//   id: 'user-id',
//   name: 'Alice',
//   messages: [
//     { id: 'msg-1', content: '...' },
//     { id: 'msg-2', content: '...' },
//   ]
// }

// Get message with user
const message = await db.query.messages.findFirst({
  where: eq(messages.id, 'message-id'),
  with: {
    user: true,  // Include user
  },
});
// Result:
// {
//   id: 'message-id',
//   content: 'Hello!',
//   user: { id: 'user-id', name: 'Alice' }
// }
```

---

**Many-to-Many: Users and Agents**

```typescript
// packages/database/src/schemas/userAgent.ts
export const userAgents = pgTable('user_agents', {
  userId: uuid('user_id')
    .notNull()
    .references(() => users.id, { onDelete: 'cascade' }),
  agentId: uuid('agent_id')
    .notNull()
    .references(() => agents.id, { onDelete: 'cascade' }),
  createdAt: timestamp('created_at').defaultNow().notNull(),
}, (table) => ({
  pk: primaryKey({ columns: [table.userId, table.agentId] }),
}));

// Relations
export const usersRelations = relations(users, ({ many }) => ({
  userAgents: many(userAgents),
}));

export const agentsRelations = relations(agents, ({ many }) => ({
  userAgents: many(userAgents),
}));

export const userAgentsRelations = relations(userAgents, ({ one }) => ({
  user: one(users, {
    fields: [userAgents.userId],
    references: [users.id],
  }),
  agent: one(agents, {
    fields: [userAgents.agentId],
    references: [agents.id],
  }),
}));
```

**Query:**

```typescript
// Get user with agents
const user = await db.query.users.findFirst({
  where: eq(users.id, 'user-id'),
  with: {
    userAgents: {
      with: {
        agent: true,  // Include agent data
      },
    },
  },
});
// Result:
// {
//   id: 'user-id',
//   name: 'Alice',
//   userAgents: [
//     {
//       userId: 'user-id',
//       agentId: 'agent-1',
//       agent: { id: 'agent-1', name: 'Assistant' }
//     }
//   ]
// }
```

---

### Manual Joins

For more control, use SQL-style joins:

```typescript
import { eq } from 'drizzle-orm';

// Inner join
const result = await db
  .select({
    message: messages,
    user: users,
  })
  .from(messages)
  .innerJoin(users, eq(messages.userId, users.id))
  .where(eq(messages.conversationId, 'conv-id'));

// Left join (include messages without users)
const result = await db
  .select({
    message: messages,
    user: users,
  })
  .from(messages)
  .leftJoin(users, eq(messages.userId, users.id));
```

---

### Nested Queries

```typescript
// Get user with conversations and their messages
const user = await db.query.users.findFirst({
  where: eq(users.id, 'user-id'),
  with: {
    conversations: {
      with: {
        messages: {
          limit: 10,  // Latest 10 messages per conversation
          orderBy: [desc(messages.createdAt)],
        },
      },
    },
  },
});
```

---

## Query Patterns

### Pattern 1: Filtering

```typescript
import { eq, ne, gt, gte, lt, lte, like, ilike, and, or, not, isNull, isNotNull, inArray } from 'drizzle-orm';

// Equal
await db.query.users.findMany({
  where: eq(users.id, 'user-id'),
});

// Not equal
await db.query.users.findMany({
  where: ne(users.role, 'banned'),
});

// Greater than / Less than
await db.query.messages.findMany({
  where: gte(messages.createdAt, new Date('2025-01-01')),
});

// Like (case-sensitive)
await db.query.users.findMany({
  where: like(users.email, '%@example.com'),
});

// iLike (case-insensitive, PostgreSQL only)
await db.query.users.findMany({
  where: ilike(users.name, '%alice%'),
});

// NULL checks
await db.query.users.findMany({
  where: isNull(users.deletedAt),  // Not deleted
});

// IN array
await db.query.users.findMany({
  where: inArray(users.id, ['id-1', 'id-2', 'id-3']),
});

// AND conditions
await db.query.users.findMany({
  where: and(
    eq(users.role, 'admin'),
    isNull(users.deletedAt)
  ),
});

// OR conditions
await db.query.users.findMany({
  where: or(
    eq(users.role, 'admin'),
    eq(users.role, 'moderator')
  ),
});

// NOT
await db.query.users.findMany({
  where: not(eq(users.role, 'banned')),
});
```

---

### Pattern 2: Sorting

```typescript
import { asc, desc } from 'drizzle-orm';

// Ascending
await db.query.users.findMany({
  orderBy: [asc(users.name)],
});

// Descending
await db.query.messages.findMany({
  orderBy: [desc(messages.createdAt)],
});

// Multiple columns
await db.query.users.findMany({
  orderBy: [asc(users.role), desc(users.createdAt)],
});
```

---

### Pattern 3: Pagination

**Offset-based:**

```typescript
const page = 2;
const pageSize = 20;

const messages = await db.query.messages.findMany({
  limit: pageSize,
  offset: (page - 1) * pageSize,
  orderBy: [desc(messages.createdAt)],
});
```

**Cursor-based (Recommended):**

```typescript
// First page
const messages = await db.query.messages.findMany({
  limit: 21,  // Fetch one extra to check if more exist
  orderBy: [desc(messages.createdAt)],
});

let nextCursor: string | undefined;
if (messages.length > 20) {
  const lastMessage = messages.pop()!;
  nextCursor = lastMessage.id;
}

// Next page
const nextMessages = await db.query.messages.findMany({
  where: lt(messages.createdAt, lastMessageCreatedAt),
  limit: 21,
  orderBy: [desc(messages.createdAt)],
});
```

**🎯 Cursor-based is faster for large datasets!**

---

### Pattern 4: Aggregations

```typescript
import { count, sum, avg, max, min } from 'drizzle-orm';

// Count
const result = await db
  .select({ count: count() })
  .from(users);

// Count with condition
const activeUsers = await db
  .select({ count: count() })
  .from(users)
  .where(isNull(users.deletedAt));

// Sum
const totalTokens = await db
  .select({ total: sum(messages.tokens) })
  .from(messages);

// Average
const avgTokens = await db
  .select({ average: avg(messages.tokens) })
  .from(messages);

// Max / Min
const latest = await db
  .select({ latest: max(messages.createdAt) })
  .from(messages);

// Group by
const messageCountByUser = await db
  .select({
    userId: messages.userId,
    count: count(),
  })
  .from(messages)
  .groupBy(messages.userId);
```

---

### Pattern 5: Transactions

Ensure all operations succeed or fail together:

```typescript
await db.transaction(async (tx) => {
  // 1. Create user
  const [user] = await tx.insert(users)
    .values({ name: 'Alice', email: 'alice@example.com' })
    .returning();

  // 2. Create conversation
  const [conversation] = await tx.insert(conversations)
    .values({ userId: user.id, title: 'New Chat' })
    .returning();

  // 3. Create initial message
  await tx.insert(messages)
    .values({
      conversationId: conversation.id,
      userId: user.id,
      content: 'Hello!',
    });

  // If any step fails, all changes are rolled back!
});
```

---

### Pattern 6: Raw SQL

For complex queries not supported by Drizzle's query builder:

```typescript
import { sql } from 'drizzle-orm';

// Raw SQL query
const result = await db.execute(sql`
  SELECT
    u.id,
    u.name,
    COUNT(m.id) as message_count
  FROM users u
  LEFT JOIN messages m ON m.user_id = u.id
  WHERE u.created_at > NOW() - INTERVAL '30 days'
  GROUP BY u.id, u.name
  HAVING COUNT(m.id) > 10
  ORDER BY message_count DESC
  LIMIT 10
`);

// With parameters (prevents SQL injection)
const userId = 'user-id';
const result = await db.execute(sql`
  SELECT * FROM users WHERE id = ${userId}
`);
```

---

## Performance Optimization

### 1. Use Indexes

```typescript
// Add indexes to columns used in WHERE, ORDER BY, JOIN
export const messages = pgTable('messages', {
  id: uuid('id').defaultRandom().primaryKey(),
  conversationId: uuid('conversation_id').notNull(),
  userId: uuid('user_id').notNull(),
  content: text('content').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
}, (table) => ({
  // Single-column indexes
  conversationIdIdx: index().on(table.conversationId),
  userIdIdx: index().on(table.userId),
  createdAtIdx: index().on(table.createdAt),

  // Composite index (queries using both columns)
  conversationCreatedIdx: index().on(table.conversationId, table.createdAt),
}));
```

**When to use composite indexes:**
```typescript
// This query benefits from composite index:
await db.query.messages.findMany({
  where: eq(messages.conversationId, 'conv-id'),
  orderBy: [desc(messages.createdAt)],
});
// Uses: conversationCreatedIdx (conversationId + createdAt)
```

---

### 2. Select Only Needed Columns

```typescript
// ❌ BAD: Fetches all columns (including large content)
const messages = await db.query.messages.findMany();

// ✅ GOOD: Select only needed columns
const messages = await db.query.messages.findMany({
  columns: {
    id: true,
    createdAt: true,
  },
});

// ✅ BETTER: Use projection
const messages = await db
  .select({
    id: messages.id,
    preview: sql<string>`LEFT(${messages.content}, 100)`,  // First 100 chars
    createdAt: messages.createdAt,
  })
  .from(messages);
```

---

### 3. Batch Operations

```typescript
// ❌ BAD: Individual inserts (N queries)
for (const message of messages) {
  await db.insert(messages).values(message);
}

// ✅ GOOD: Batch insert (1 query)
await db.insert(messages).values(messages);

// ❌ BAD: Individual updates (N queries)
for (const id of ids) {
  await db.update(messages)
    .set({ read: true })
    .where(eq(messages.id, id));
}

// ✅ GOOD: Batch update (1 query)
await db.update(messages)
  .set({ read: true })
  .where(inArray(messages.id, ids));
```

---

### 4. Use Connection Pooling

**Neon (Server) - Built-in pooling:**

```typescript
// packages/database/src/server/index.ts
import { neon } from '@neondatabase/serverless';

// Neon automatically pools connections
const sql = neon(process.env.DATABASE_URL!);
```

**Custom pool configuration:**

```typescript
import { Pool } from '@neondatabase/serverless';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,  // Max connections
  idleTimeoutMillis: 30000,  // Close idle connections after 30s
  connectionTimeoutMillis: 2000,  // Timeout after 2s
});
```

---

### 5. Query Caching

**Redis caching pattern:**

```typescript
import { redis } from '@/server/modules/Redis';

export class UserRepository {
  static async findWithCache(db: Database, userId: string) {
    const cacheKey = `user:${userId}`;

    // Check cache
    const cached = await redis.get(cacheKey);
    if (cached) {
      return JSON.parse(cached);
    }

    // Query database
    const user = await db.query.users.findFirst({
      where: eq(users.id, userId),
    });

    // Cache for 5 minutes
    if (user) {
      await redis.set(cacheKey, JSON.stringify(user), 'EX', 300);
    }

    return user;
  }

  static async invalidateCache(userId: string) {
    await redis.del(`user:${userId}`);
  }
}
```

---

### 6. Explain Query Plans

Analyze slow queries:

```typescript
import { sql } from 'drizzle-orm';

const result = await db.execute(sql`
  EXPLAIN ANALYZE
  SELECT * FROM messages
  WHERE conversation_id = 'conv-id'
  ORDER BY created_at DESC
  LIMIT 20
`);

console.log(result);
// Shows: Index scan, execution time, etc.
```

---

## Best Practices

### ✅ DO

1. **Use UUIDs for primary keys**
   ```typescript
   id: uuid('id').defaultRandom().primaryKey()
   ```

2. **Add timestamps to all tables**
   ```typescript
   createdAt: timestamp('created_at').defaultNow().notNull()
   updatedAt: timestamp('updated_at').defaultNow().notNull()
   ```

3. **Use foreign key constraints**
   ```typescript
   userId: uuid('user_id')
     .notNull()
     .references(() => users.id, { onDelete: 'cascade' })
   ```

4. **Add indexes to foreign keys**
   ```typescript
   }, (table) => ({
     userIdIdx: index().on(table.userId),
   }))
   ```

5. **Use transactions for multi-step operations**
   ```typescript
   await db.transaction(async (tx) => {
     // Multiple operations
   });
   ```

6. **Validate data before insertion**
   ```typescript
   const schema = z.object({
     name: z.string().min(1).max(100),
     email: z.string().email(),
   });
   const validated = schema.parse(data);
   await db.insert(users).values(validated);
   ```

7. **Use relations for type-safe joins**
   ```typescript
   const user = await db.query.users.findFirst({
     with: { messages: true }
   });
   ```

---

### ❌ DON'T

1. **Don't use auto-increment IDs**
   ```typescript
   // ❌ BAD: Doesn't work offline
   id: serial('id').primaryKey()

   // ✅ GOOD: Works offline
   id: uuid('id').defaultRandom().primaryKey()
   ```

2. **Don't fetch all data**
   ```typescript
   // ❌ BAD: Loads everything
   const users = await db.query.users.findMany();

   // ✅ GOOD: Paginate
   const users = await db.query.users.findMany({
     limit: 20,
     offset: page * 20,
   });
   ```

3. **Don't forget indexes**
   ```typescript
   // ❌ BAD: No index on frequently queried column
   conversationId: uuid('conversation_id').notNull()

   // ✅ GOOD: Add index
   }, (table) => ({
     conversationIdIdx: index().on(table.conversationId),
   }))
   ```

4. **Don't ignore N+1 queries**
   ```typescript
   // ❌ BAD: N+1 queries
   const messages = await db.query.messages.findMany();
   for (const message of messages) {
     const user = await db.query.users.findFirst({
       where: eq(users.id, message.userId),
     });
   }

   // ✅ GOOD: Single query with join
   const messages = await db.query.messages.findMany({
     with: { user: true },
   });
   ```

5. **Don't use raw SQL without parameters**
   ```typescript
   // ❌ BAD: SQL injection risk
   await db.execute(sql`SELECT * FROM users WHERE id = '${userId}'`);

   // ✅ GOOD: Parameterized query
   await db.execute(sql`SELECT * FROM users WHERE id = ${userId}`);
   ```

6. **Don't store sensitive data unencrypted**
   ```typescript
   // ❌ BAD: Plain text password
   password: text('password').notNull()

   // ✅ GOOD: Hash before storing
   import bcrypt from 'bcrypt';
   const hashedPassword = await bcrypt.hash(password, 10);
   await db.insert(users).values({ password: hashedPassword });
   ```

---

## Summary

**Database Architecture in One Paragraph:**

> LobeHub uses PGLite (client) for instant offline operations and Neon (server) for cloud persistence with Drizzle ORM providing type-safe queries. The three-layer architecture (schemas, models, repositories) separates concerns: schemas define tables, models handle CRUD, repositories handle complex queries. UUIDs enable offline-first operation, and migrations keep schemas in sync across environments.

**Key Patterns:**

1. 🗄️ **Migrations** - Drizzle Kit for schema changes
2. 🔗 **Relationships** - One-to-many, many-to-many with type-safe joins
3. 🔍 **Query Patterns** - Filtering, sorting, pagination, aggregations
4. ⚡ **Performance** - Indexes, batching, caching, connection pooling
5. ✅ **Best Practices** - UUIDs, timestamps, foreign keys, transactions

**Technologies Summary:**

| Technology | Purpose | Benefits |
|------------|---------|----------|
| **Drizzle ORM** | Type-safe queries | Inferred types, no code generation |
| **PGLite** | Client database | Offline-first, instant queries |
| **Neon** | Server database | Serverless, auto-scaling |
| **Drizzle Kit** | Migrations | Schema changes tracked |
| **UUID** | Primary keys | Works offline, no collisions |
| **Indexes** | Performance | Fast lookups and sorts |

**Query Optimization Checklist:**

- ✅ Add indexes to foreign keys
- ✅ Select only needed columns
- ✅ Use cursor-based pagination
- ✅ Batch operations when possible
- ✅ Use transactions for consistency
- ✅ Cache frequently accessed data
- ✅ Profile slow queries with EXPLAIN

---

**Document Status:** ✅ Complete (Part 2 of 2) | **Last Updated:** November 20, 2025

**[← Part 1](./DATABASE-ARCHITECTURE-PART-01.md)** | **Next:** [Integration Guide](./INTEGRATION_GUIDE.md)
