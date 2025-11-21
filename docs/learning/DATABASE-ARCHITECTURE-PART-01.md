# Database Architecture - Part 1 of 2

**Documented:** November 2025
**Tech Stack Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)
**Target:** Mid-level developers proficient in JS/TS/React

**📄 This is Part 1 of 2** | **[Continue to Part 2 →](./DATABASE-ARCHITECTURE-PART-02.md)**

This comprehensive guide explains LobeHub's database architecture, focusing on the dual-database strategy, Drizzle ORM patterns, and schema design.

**Prerequisites:**
- [Architecture Overview](./ARCHITECTURE-OVERVIEW-PART-01.md) - System architecture
- [Tech Stack Guide - Part 3](./TECH-STACK-GUIDE-PART-03.md) - Database technologies

**Time to read:** 30 minutes (Part 1)

---

## Table of Contents - Part 1

1. [Database Architecture Principles](#database-architecture-principles)
2. [Dual Database Strategy](#dual-database-strategy)
3. [Schema Design with Drizzle](#schema-design-with-drizzle)
4. [Database Layer Organization](#database-layer-organization)
5. [Common Patterns](#common-patterns)

**[Part 2 →](./DATABASE-ARCHITECTURE-PART-02.md)** will cover:
- Migrations
- Relationships and Joins
- Query Patterns
- Performance Optimization
- Best Practices

---

## Database Architecture Principles

**LobeHub Database Philosophy:**

> Build an **offline-first, type-safe, performant** database layer using PGLite (client) and Neon (server) with Drizzle ORM for type safety.

**Core Principles:**

1. **💾 Offline-First**
   - PGLite runs in browser (WASM PostgreSQL)
   - App works without network
   - Sync to cloud when online

2. **🔒 Type Safety End-to-End**
   - Schema = source of truth
   - TypeScript types inferred from schema
   - No manual type definitions

3. **🎯 Three-Layer Architecture**
   - **Schemas** - Table definitions (Drizzle)
   - **Models** - CRUD operations (create, read, update, delete)
   - **Repositories** - Complex queries (joins, aggregations)

4. **⚡ Performance-First**
   - Indexes on common queries
   - Efficient query patterns
   - Connection pooling (Neon)

5. **🔄 Dual Database Sync**
   - Client (PGLite) for instant operations
   - Server (Neon) for persistence and sync
   - Optimistic updates with background sync

---

## Dual Database Strategy

### Architecture Overview

```
┌──────────────────────────────────────────────────────────┐
│                    Browser (Client)                      │
│                                                          │
│  ┌────────────────────────────────────────────────┐    │
│  │         PGLite Database (WASM)                 │    │
│  │  • Full PostgreSQL in browser                  │    │
│  │  • Stored in IndexedDB                         │    │
│  │  • Instant queries (no network)                │    │
│  │  • Works offline                               │    │
│  └────────────────────────────────────────────────┘    │
│                      ↕                                  │
│              ┌───────────────┐                          │
│              │  Sync Manager │                          │
│              └───────────────┘                          │
│                      ↕                                  │
└──────────────────────┼──────────────────────────────────┘
                       │
                  (Network)
                       │
┌──────────────────────┼──────────────────────────────────┐
│                  Server (Cloud)                          │
│                      ↓                                  │
│  ┌────────────────────────────────────────────────┐    │
│  │      Neon PostgreSQL (Serverless)              │    │
│  │  • Cloud PostgreSQL database                   │    │
│  │  • Auto-scaling                                │    │
│  │  • Multi-device sync                           │    │
│  │  • Backup and durability                       │    │
│  └────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────┘
```

---

### When to Use Each Database

**Use PGLite (Client) for:**

| Operation | Reason |
|-----------|--------|
| **Read operations** | Instant, no network latency |
| **Optimistic updates** | Update UI immediately |
| **Offline usage** | Works without internet |
| **Privacy-sensitive data** | Stays on device |
| **Drafts** | Save work-in-progress locally |

**Use Neon (Server) for:**

| Operation | Reason |
|-----------|--------|
| **Cross-device sync** | Share data across devices |
| **Collaboration** | Multiple users, same data |
| **Backup** | Durable storage |
| **Server validation** | Enforce business rules |
| **Analytics** | Aggregate data across users |

---

### Setup: PGLite (Client)

```typescript
// packages/database/src/client/index.ts
import { PGlite } from '@electric-sql/pglite';
import { drizzle } from 'drizzle-orm/pglite';
import * as schema from '../schemas';

// Create PGLite instance (runs in browser)
export const pglite = new PGlite('idb://lobechat', {
  // Store in IndexedDB for persistence
});

// Wrap with Drizzle ORM
export const clientDB = drizzle(pglite, { schema });

// Run migrations on first load
export async function initClientDB() {
  // Apply migrations
  await pglite.exec(migrations);
}
```

**Usage in client code:**

```typescript
// Client-side component or service
import { clientDB } from '@lobechat/database/client';
import { messages } from '@lobechat/database/schemas/message';

// Query runs in browser, instant!
const messages = await clientDB.query.messages.findMany();
```

---

### Setup: Neon (Server)

```typescript
// packages/database/src/server/index.ts
import { neon } from '@neondatabase/serverless';
import { drizzle } from 'drizzle-orm/neon-http';
import * as schema from '../schemas';

// Create Neon connection
const sql = neon(process.env.DATABASE_URL!);

// Wrap with Drizzle ORM
export const serverDB = drizzle(sql, { schema });
```

**Usage in server code:**

```typescript
// Server-side service or tRPC route
import { serverDB } from '@lobechat/database/server';
import { messages } from '@lobechat/database/schemas/message';

// Query runs on Neon (cloud)
const messages = await serverDB.query.messages.findMany();
```

**🎯 Key Insight:** Same schema, same queries, different database!

---

## Schema Design with Drizzle

### Three-Layer Pattern

```
packages/database/src/
├── schemas/              ← Layer 1: Table definitions
│   ├── user.ts
│   ├── message.ts
│   └── conversation.ts
│
├── models/               ← Layer 2: CRUD operations
│   ├── user.ts
│   ├── message.ts
│   └── conversation.ts
│
└── repositories/         ← Layer 3: Complex queries
    ├── user.ts
    ├── message.ts
    └── conversation.ts
```

---

### Layer 1: Schemas (Table Definitions)

**Basic Schema:**

```typescript
// packages/database/src/schemas/user.ts
import { pgTable, text, timestamp, uuid, boolean } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: uuid('id').defaultRandom().primaryKey(),
  name: text('name').notNull(),
  email: text('email').notNull().unique(),
  emailVerified: boolean('email_verified').default(false).notNull(),
  image: text('image'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

// Infer TypeScript types from schema
export type User = typeof users.$inferSelect;
export type NewUser = typeof users.$inferInsert;
```

**🎯 Benefit:** TypeScript types automatically generated from schema!

---

**Schema with Enum:**

```typescript
// packages/database/src/schemas/message.ts
import { pgTable, text, timestamp, uuid, pgEnum } from 'drizzle-orm/pg-core';

// Define enum
export const messageRoleEnum = pgEnum('message_role', [
  'user',
  'assistant',
  'system',
]);

export const messages = pgTable('messages', {
  id: uuid('id').defaultRandom().primaryKey(),
  conversationId: uuid('conversation_id').notNull(),
  userId: uuid('user_id').notNull(),
  role: messageRoleEnum('role').notNull(),
  content: text('content').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

export type Message = typeof messages.$inferSelect;
export type NewMessage = typeof messages.$inferInsert;
```

---

**Schema with JSON Column:**

```typescript
import { pgTable, uuid, jsonb } from 'drizzle-orm/pg-core';

export const agents = pgTable('agents', {
  id: uuid('id').defaultRandom().primaryKey(),
  name: text('name').notNull(),

  // JSON column with type
  config: jsonb('config').$type<{
    temperature: number;
    maxTokens: number;
    systemPrompt: string;
  }>().notNull(),

  // JSON column without specific type
  metadata: jsonb('metadata').$type<Record<string, unknown>>(),
});

export type Agent = typeof agents.$inferSelect;
```

---

**Schema with Foreign Keys:**

```typescript
// packages/database/src/schemas/message.ts
import { pgTable, uuid, foreignKey } from 'drizzle-orm/pg-core';
import { users } from './user';
import { conversations } from './conversation';

export const messages = pgTable('messages', {
  id: uuid('id').defaultRandom().primaryKey(),

  // Foreign key to conversations
  conversationId: uuid('conversation_id')
    .notNull()
    .references(() => conversations.id, { onDelete: 'cascade' }),

  // Foreign key to users
  userId: uuid('user_id')
    .notNull()
    .references(() => users.id, { onDelete: 'cascade' }),

  content: text('content').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});
```

**`onDelete` options:**
- `cascade` - Delete related records
- `set null` - Set foreign key to NULL
- `restrict` - Prevent deletion if related records exist
- `no action` - Default behavior

---

**Schema with Indexes:**

```typescript
import { pgTable, uuid, text, timestamp, index } from 'drizzle-orm/pg-core';

export const messages = pgTable('messages', {
  id: uuid('id').defaultRandom().primaryKey(),
  conversationId: uuid('conversation_id').notNull(),
  userId: uuid('user_id').notNull(),
  content: text('content').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
}, (table) => ({
  // Single-column indexes
  conversationIdIdx: index('messages_conversation_id_idx')
    .on(table.conversationId),

  userIdIdx: index('messages_user_id_idx')
    .on(table.userId),

  createdAtIdx: index('messages_created_at_idx')
    .on(table.createdAt),

  // Composite index (multiple columns)
  conversationCreatedIdx: index('messages_conversation_created_idx')
    .on(table.conversationId, table.createdAt),
}));
```

**When to add indexes:**
- Columns used in `WHERE` clauses
- Columns used in `ORDER BY`
- Foreign key columns
- Columns used in `JOIN` conditions

---

### Layer 2: Models (CRUD Operations)

Models provide **basic CRUD** operations:

```typescript
// packages/database/src/models/user.ts
import { eq } from 'drizzle-orm';
import { users, type User, type NewUser } from '../schemas/user';

export class UserModel {
  // Create
  static async create(db: Database, data: NewUser): Promise<User> {
    const [user] = await db.insert(users).values(data).returning();
    return user;
  }

  // Read
  static async findById(db: Database, id: string): Promise<User | undefined> {
    return await db.query.users.findFirst({
      where: eq(users.id, id),
    });
  }

  static async findByEmail(db: Database, email: string): Promise<User | undefined> {
    return await db.query.users.findFirst({
      where: eq(users.email, email),
    });
  }

  static async findMany(db: Database, options?: {
    limit?: number;
    offset?: number;
  }): Promise<User[]> {
    return await db.query.users.findMany({
      limit: options?.limit ?? 10,
      offset: options?.offset ?? 0,
    });
  }

  // Update
  static async update(
    db: Database,
    id: string,
    data: Partial<NewUser>
  ): Promise<User | undefined> {
    const [updated] = await db.update(users)
      .set({ ...data, updatedAt: new Date() })
      .where(eq(users.id, id))
      .returning();

    return updated;
  }

  // Delete
  static async delete(db: Database, id: string): Promise<void> {
    await db.delete(users).where(eq(users.id, id));
  }
}

// Type for database (works with both PGLite and Neon)
type Database = typeof import('../client').clientDB | typeof import('../server').serverDB;
```

---

**Usage in Services:**

```typescript
// Client-side service
import { clientDB } from '@lobechat/database/client';
import { UserModel } from '@lobechat/database/models/user';

export class UserService {
  async createUser(data: NewUser) {
    return await UserModel.create(clientDB, data);
  }

  async getUser(id: string) {
    return await UserModel.findById(clientDB, id);
  }
}
```

---

### Layer 3: Repositories (Complex Queries)

Repositories handle **complex queries** with joins, aggregations, etc.:

```typescript
// packages/database/src/repositories/user.ts
import { eq, and, gte, count } from 'drizzle-orm';
import { users } from '../schemas/user';
import { messages } from '../schemas/message';
import { conversations } from '../schemas/conversation';

export class UserRepository {
  // Get user with message count
  static async findWithMessageCount(db: Database, userId: string) {
    const result = await db
      .select({
        user: users,
        messageCount: count(messages.id),
      })
      .from(users)
      .leftJoin(messages, eq(messages.userId, users.id))
      .where(eq(users.id, userId))
      .groupBy(users.id);

    return result[0];
  }

  // Get users with recent activity
  static async findActiveUsers(db: Database, since: Date) {
    return await db
      .select({
        id: users.id,
        name: users.name,
        email: users.email,
        lastMessageAt: max(messages.createdAt),
      })
      .from(users)
      .innerJoin(messages, eq(messages.userId, users.id))
      .where(gte(messages.createdAt, since))
      .groupBy(users.id, users.name, users.email)
      .orderBy(desc(max(messages.createdAt)));
  }

  // Get user with conversations and messages
  static async findWithDetails(db: Database, userId: string) {
    return await db.query.users.findFirst({
      where: eq(users.id, userId),
      with: {
        conversations: {
          with: {
            messages: {
              limit: 10,
              orderBy: [desc(messages.createdAt)],
            },
          },
        },
      },
    });
  }
}
```

---

## Database Layer Organization

### Complete Structure

```
packages/database/
├── src/
│   ├── schemas/                    ← Table definitions
│   │   ├── user.ts
│   │   ├── message.ts
│   │   ├── conversation.ts
│   │   ├── agent.ts
│   │   ├── file.ts
│   │   └── index.ts                ← Export all schemas
│   │
│   ├── models/                     ← CRUD operations
│   │   ├── user.ts
│   │   ├── message.ts
│   │   ├── conversation.ts
│   │   └── index.ts
│   │
│   ├── repositories/               ← Complex queries
│   │   ├── user.ts
│   │   ├── message.ts
│   │   ├── conversation.ts
│   │   └── index.ts
│   │
│   ├── migrations/                 ← Database migrations
│   │   ├── 0001_create_users.sql
│   │   ├── 0002_create_messages.sql
│   │   └── meta/
│   │       └── _journal.json
│   │
│   ├── client/                     ← PGLite setup
│   │   └── index.ts
│   │
│   └── server/                     ← Neon setup
│       └── index.ts
│
├── drizzle.config.ts               ← Drizzle configuration
├── package.json
└── tsconfig.json
```

---

### Export Pattern

```typescript
// packages/database/src/schemas/index.ts
export * from './user';
export * from './message';
export * from './conversation';
export * from './agent';
export * from './file';

// packages/database/src/models/index.ts
export * from './user';
export * from './message';
export * from './conversation';

// packages/database/src/repositories/index.ts
export * from './user';
export * from './message';
export * from './conversation';
```

**Usage:**

```typescript
// Import from single entry point
import { users, messages } from '@lobechat/database/schemas';
import { UserModel, MessageModel } from '@lobechat/database/models';
import { UserRepository } from '@lobechat/database/repositories';
```

---

## Common Patterns

### Pattern 1: Timestamps

Add automatic timestamps to all tables:

```typescript
import { pgTable, timestamp } from 'drizzle-orm/pg-core';

export const baseColumns = {
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
};

// Use in tables:
export const users = pgTable('users', {
  id: uuid('id').defaultRandom().primaryKey(),
  name: text('name').notNull(),
  ...baseColumns,  // Add timestamps
});
```

---

### Pattern 2: Soft Deletes

Mark records as deleted instead of removing them:

```typescript
export const users = pgTable('users', {
  id: uuid('id').defaultRandom().primaryKey(),
  name: text('name').notNull(),
  deletedAt: timestamp('deleted_at'),  // NULL = not deleted
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

// Model methods:
export class UserModel {
  static async softDelete(db: Database, id: string) {
    await db.update(users)
      .set({ deletedAt: new Date() })
      .where(eq(users.id, id));
  }

  static async findActive(db: Database) {
    return await db.query.users.findMany({
      where: isNull(users.deletedAt),  // Only non-deleted
    });
  }
}
```

---

### Pattern 3: Optimistic Locking

Prevent concurrent update conflicts:

```typescript
export const users = pgTable('users', {
  id: uuid('id').defaultRandom().primaryKey(),
  name: text('name').notNull(),
  version: integer('version').default(1).notNull(),  // Version number
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

export class UserModel {
  static async updateWithLock(
    db: Database,
    id: string,
    currentVersion: number,
    updates: Partial<NewUser>
  ) {
    const [updated] = await db.update(users)
      .set({
        ...updates,
        version: currentVersion + 1,
        updatedAt: new Date(),
      })
      .where(and(
        eq(users.id, id),
        eq(users.version, currentVersion)  // Check version matches
      ))
      .returning();

    if (!updated) {
      throw new Error('Update conflict: record was modified by another user');
    }

    return updated;
  }
}
```

---

### Pattern 4: UUID vs Auto-increment ID

**UUID (Recommended for LobeHub):**

```typescript
export const users = pgTable('users', {
  id: uuid('id').defaultRandom().primaryKey(),  // ✅ UUID
  name: text('name').notNull(),
});
```

**Benefits:**
- ✅ Works offline (generate on client)
- ✅ No collision across devices
- ✅ Secure (not sequential)

**Auto-increment (Traditional):**

```typescript
import { serial } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),  // ❌ Auto-increment
  name: text('name').notNull(),
});
```

**Drawbacks:**
- ❌ Requires server to generate
- ❌ Conflicts in distributed systems
- ❌ Predictable (security issue)

**🎯 LobeHub uses UUIDs for offline-first architecture!**

---

### Pattern 5: JSON Columns for Flexible Data

```typescript
export const agents = pgTable('agents', {
  id: uuid('id').defaultRandom().primaryKey(),
  name: text('name').notNull(),

  // Structured JSON with type
  config: jsonb('config').$type<{
    model: string;
    temperature: number;
    maxTokens: number;
    systemPrompt?: string;
  }>().notNull(),

  // Flexible metadata
  metadata: jsonb('metadata').$type<Record<string, unknown>>(),
});

// Usage:
const agent = await db.insert(agents).values({
  name: 'Assistant',
  config: {
    model: 'gpt-4',
    temperature: 0.7,
    maxTokens: 2000,
  },
  metadata: {
    createdBy: 'user-123',
    version: '1.0',
  },
}).returning();

// Query JSON fields:
const agents = await db.query.agents.findMany({
  where: sql`config->>'model' = 'gpt-4'`,
});
```

---

## Summary - Part 1

**Database Architecture in One Paragraph:**

> LobeHub uses a dual-database strategy: PGLite (WASM PostgreSQL in browser) for instant offline operations, and Neon (serverless PostgreSQL) for cloud persistence. Drizzle ORM provides type safety with a three-layer architecture: schemas (table definitions), models (CRUD), and repositories (complex queries). All TypeScript types are inferred from schemas, eliminating manual type definitions.

**Key Patterns:**

1. 💾 **Dual Database** - PGLite (client) + Neon (server)
2. 🔒 **Type Safety** - Types inferred from Drizzle schemas
3. 🎯 **Three Layers** - Schemas, Models, Repositories
4. 📋 **Common Patterns** - Timestamps, Soft deletes, Optimistic locking
5. 🆔 **UUIDs** - For offline-first compatibility
6. 📦 **JSON Columns** - For flexible, typed data

---

**Document Status:** ✅ Part 1 Complete | **Last Updated:** November 20, 2025

**[Continue to Part 2 →](./DATABASE-ARCHITECTURE-PART-02.md)** - Migrations, Relationships, Query Patterns, Performance Optimization
