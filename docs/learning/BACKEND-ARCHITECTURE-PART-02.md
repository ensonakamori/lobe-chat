# Backend Architecture - Part 2 of 2

**Documented:** November 2025
**Tech Stack Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)
**Target:** Mid-level developers proficient in JS/TS/React

**📄 This is Part 2 of 2** | **[← Back to Part 1](./BACKEND-ARCHITECTURE-PART-01.md)**

This document continues the backend architecture guide, covering database integration, authentication, file handling, and advanced patterns.

**Prerequisites:**
- **[Part 1](./BACKEND-ARCHITECTURE-PART-01.md)** - Read this first!

**Time to read:** 30 minutes (Part 2)

---

## Table of Contents - Part 2

1. [Database Integration](#database-integration)
2. [Authentication & Authorization](#authentication--authorization)
3. [File Handling](#file-handling)
4. [Background Jobs](#background-jobs)
5. [Streaming and Real-Time](#streaming-and-real-time)
6. [Performance Optimization](#performance-optimization)
7. [Testing](#testing)
8. [Best Practices](#best-practices)
9. [Summary](#summary)

**[← Part 1](./BACKEND-ARCHITECTURE-PART-01.md)** covered:
- Backend Architecture Principles
- tRPC Architecture (Routers, Procedures, Context, Middleware)
- API Design Patterns (Naming, Validation, Error Handling)
- Server Services

---

## Database Integration

### Drizzle ORM with tRPC

**Pattern: Service Layer for Database Access**

```typescript
// src/server/services/message/index.ts
import { serverDB } from '@lobechat/database/server';
import { messages } from '@lobechat/database/schemas/message';
import { eq, and, desc } from 'drizzle-orm';

export class MessageService {
  static async getMessagesForConversation(conversationId: string) {
    return await serverDB.query.messages.findMany({
      where: eq(messages.conversationId, conversationId),
      orderBy: [desc(messages.createdAt)],
    });
  }

  static async createMessage(data: {
    conversationId: string;
    userId: string;
    content: string;
  }) {
    const [message] = await serverDB.insert(messages)
      .values(data)
      .returning();

    return message;
  }

  static async getMessage(id: string) {
    return await serverDB.query.messages.findFirst({
      where: eq(messages.id, id),
      with: {
        user: true,         // Include user relation
        attachments: true,  // Include attachments
      },
    });
  }

  static async updateMessage(id: string, updates: { content?: string }) {
    const [updated] = await serverDB.update(messages)
      .set(updates)
      .where(eq(messages.id, id))
      .returning();

    return updated;
  }

  static async deleteMessage(id: string) {
    await serverDB.delete(messages)
      .where(eq(messages.id, id));
  }
}
```

---

**Using in tRPC Router:**

```typescript
// src/server/routers/lambda/message.ts
import { router, protectedProcedure } from '@/server/trpc';
import { MessageService } from '@/server/services/message';
import { z } from 'zod';

export const messageRouter = router({
  list: protectedProcedure
    .input(z.object({
      conversationId: z.string().uuid(),
    }))
    .query(async ({ input, ctx }) => {
      // Verify user owns conversation
      await verifyConversationOwnership(input.conversationId, ctx.user.id);

      return await MessageService.getMessagesForConversation(
        input.conversationId
      );
    }),

  create: protectedProcedure
    .input(z.object({
      conversationId: z.string().uuid(),
      content: z.string().min(1),
    }))
    .mutation(async ({ input, ctx }) => {
      // Verify user owns conversation
      await verifyConversationOwnership(input.conversationId, ctx.user.id);

      return await MessageService.createMessage({
        conversationId: input.conversationId,
        userId: ctx.user.id,
        content: input.content,
      });
    }),

  update: protectedProcedure
    .input(z.object({
      id: z.string().uuid(),
      content: z.string().min(1),
    }))
    .mutation(async ({ input, ctx }) => {
      // Verify user owns message
      const message = await MessageService.getMessage(input.id);
      if (message.userId !== ctx.user.id) {
        throw new TRPCError({ code: 'FORBIDDEN' });
      }

      return await MessageService.updateMessage(input.id, {
        content: input.content,
      });
    }),

  delete: protectedProcedure
    .input(z.object({
      id: z.string().uuid(),
    }))
    .mutation(async ({ input, ctx }) => {
      // Verify user owns message
      const message = await MessageService.getMessage(input.id);
      if (message.userId !== ctx.user.id) {
        throw new TRPCError({ code: 'FORBIDDEN' });
      }

      await MessageService.deleteMessage(input.id);
      return { success: true };
    }),
});
```

---

### Transactions

For operations that must succeed or fail together:

```typescript
export class MessageService {
  static async createMessageWithAttachments(data: {
    conversationId: string;
    userId: string;
    content: string;
    attachments: { url: string; type: string }[];
  }) {
    return await serverDB.transaction(async (tx) => {
      // 1. Create message
      const [message] = await tx.insert(messages)
        .values({
          conversationId: data.conversationId,
          userId: data.userId,
          content: data.content,
        })
        .returning();

      // 2. Create attachments
      if (data.attachments.length > 0) {
        await tx.insert(attachments)
          .values(
            data.attachments.map((att) => ({
              messageId: message.id,
              url: att.url,
              type: att.type,
            }))
          );
      }

      // 3. Update conversation last message time
      await tx.update(conversations)
        .set({ lastMessageAt: new Date() })
        .where(eq(conversations.id, data.conversationId));

      return message;
    });
    // If any step fails, ALL changes are rolled back!
  }
}
```

---

### Query Optimization

**1. Select Only Needed Columns:**

```typescript
// ❌ BAD: Fetches all columns
const messages = await serverDB.query.messages.findMany();

// ✅ GOOD: Select specific columns
const messages = await serverDB.query.messages.findMany({
  columns: {
    id: true,
    content: true,
    createdAt: true,
  },
});
```

---

**2. Use Eager Loading for Relations:**

```typescript
// ❌ BAD: N+1 queries
const messages = await serverDB.query.messages.findMany();
for (const message of messages) {
  const user = await serverDB.query.users.findFirst({
    where: eq(users.id, message.userId),
  });
}

// ✅ GOOD: Single query with join
const messages = await serverDB.query.messages.findMany({
  with: {
    user: true,  // Joined in single query
  },
});
```

---

**3. Use Indexes:**

```typescript
// packages/database/src/schemas/message.ts
import { pgTable, text, timestamp, uuid, index } from 'drizzle-orm/pg-core';

export const messages = pgTable('messages', {
  id: uuid('id').defaultRandom().primaryKey(),
  conversationId: uuid('conversation_id').notNull(),
  userId: uuid('user_id').notNull(),
  content: text('content').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
}, (table) => ({
  // Indexes for common queries
  conversationIdIdx: index('messages_conversation_id_idx').on(table.conversationId),
  userIdIdx: index('messages_user_id_idx').on(table.userId),
  createdAtIdx: index('messages_created_at_idx').on(table.createdAt),
}));
```

---

## Authentication & Authorization

### NextAuth.js Integration

**Pattern: Protected Procedures**

```typescript
// src/server/trpc.ts
import { TRPCError } from '@trpc/server';

export const isAuthed = t.middleware(async ({ ctx, next }) => {
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
```

---

**Role-Based Access Control (RBAC):**

```typescript
// src/server/trpc.ts
const hasRole = (role: 'user' | 'admin' | 'moderator') => {
  return t.middleware(async ({ ctx, next }) => {
    if (!ctx.session?.user) {
      throw new TRPCError({ code: 'UNAUTHORIZED' });
    }

    if (ctx.session.user.role !== role && ctx.session.user.role !== 'admin') {
      throw new TRPCError({ code: 'FORBIDDEN' });
    }

    return next({
      ctx: {
        session: ctx.session,
        user: ctx.session.user,
      },
    });
  });
};

export const adminProcedure = publicProcedure.use(hasRole('admin'));
export const moderatorProcedure = publicProcedure.use(hasRole('moderator'));
```

**Usage:**

```typescript
export const userRouter = router({
  // Anyone can view
  listUsers: publicProcedure
    .query(async () => {
      return await UserService.listUsers();
    }),

  // Only authenticated users
  getMyProfile: protectedProcedure
    .query(async ({ ctx }) => {
      return await UserService.getUser(ctx.user.id);
    }),

  // Only moderators
  banUser: moderatorProcedure
    .input(z.object({ userId: z.string().uuid() }))
    .mutation(async ({ input }) => {
      return await UserService.banUser(input.userId);
    }),

  // Only admins
  deleteUser: adminProcedure
    .input(z.object({ userId: z.string().uuid() }))
    .mutation(async ({ input }) => {
      return await UserService.deleteUser(input.userId);
    }),
});
```

---

**Resource Ownership:**

```typescript
export class MessageService {
  static async verifyOwnership(messageId: string, userId: string) {
    const message = await serverDB.query.messages.findFirst({
      where: eq(messages.id, messageId),
    });

    if (!message) {
      throw new TRPCError({ code: 'NOT_FOUND' });
    }

    if (message.userId !== userId) {
      throw new TRPCError({ code: 'FORBIDDEN' });
    }

    return message;
  }
}

// Usage in router:
export const messageRouter = router({
  update: protectedProcedure
    .input(z.object({
      id: z.string().uuid(),
      content: z.string(),
    }))
    .mutation(async ({ input, ctx }) => {
      // Verify user owns the message
      await MessageService.verifyOwnership(input.id, ctx.user.id);

      return await MessageService.updateMessage(input.id, {
        content: input.content,
      });
    }),
});
```

---

## File Handling

### File Upload Pattern

**Using Async Router (Node Runtime for large files):**

```typescript
// src/server/routers/async/fileUpload.ts
import { router, protectedProcedure } from '@/server/trpc';
import { z } from 'zod';
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';

const s3 = new S3Client({ region: 'us-east-1' });

export const fileUploadRouter = router({
  uploadFile: protectedProcedure
    .input(z.object({
      fileName: z.string(),
      fileType: z.string(),
      fileSize: z.number().max(10 * 1024 * 1024), // Max 10MB
      base64Data: z.string(),
    }))
    .mutation(async ({ input, ctx }) => {
      // 1. Validate file type
      const allowedTypes = ['image/jpeg', 'image/png', 'application/pdf'];
      if (!allowedTypes.includes(input.fileType)) {
        throw new TRPCError({
          code: 'BAD_REQUEST',
          message: 'Invalid file type',
        });
      }

      // 2. Decode base64
      const buffer = Buffer.from(input.base64Data, 'base64');

      // 3. Generate unique file name
      const uniqueFileName = `${ctx.user.id}/${Date.now()}-${input.fileName}`;

      // 4. Upload to S3
      await s3.send(
        new PutObjectCommand({
          Bucket: process.env.S3_BUCKET_NAME,
          Key: uniqueFileName,
          Body: buffer,
          ContentType: input.fileType,
        })
      );

      // 5. Save metadata to database
      const fileUrl = `https://${process.env.S3_BUCKET_NAME}.s3.amazonaws.com/${uniqueFileName}`;

      const [file] = await serverDB.insert(files)
        .values({
          userId: ctx.user.id,
          fileName: input.fileName,
          fileType: input.fileType,
          fileSize: input.fileSize,
          url: fileUrl,
        })
        .returning();

      return file;
    }),

  getUploadUrl: protectedProcedure
    .input(z.object({
      fileName: z.string(),
      fileType: z.string(),
    }))
    .mutation(async ({ input, ctx }) => {
      // Generate presigned URL for direct upload
      const { getSignedUrl } = await import('@aws-sdk/s3-request-presigner');

      const uniqueFileName = `${ctx.user.id}/${Date.now()}-${input.fileName}`;

      const command = new PutObjectCommand({
        Bucket: process.env.S3_BUCKET_NAME,
        Key: uniqueFileName,
        ContentType: input.fileType,
      });

      const url = await getSignedUrl(s3, command, { expiresIn: 3600 });

      return {
        uploadUrl: url,
        fileUrl: `https://${process.env.S3_BUCKET_NAME}.s3.amazonaws.com/${uniqueFileName}`,
      };
    }),
});
```

---

**Client-side file upload:**

```typescript
'use client';

import { trpc } from '@/utils/trpc';

function FileUpload() {
  const uploadMutation = trpc.async.fileUpload.uploadFile.useMutation();

  const handleFileUpload = async (file: File) => {
    // Convert file to base64
    const base64 = await new Promise<string>((resolve) => {
      const reader = new FileReader();
      reader.onload = () => resolve(reader.result as string);
      reader.readAsDataURL(file);
    });

    const base64Data = base64.split(',')[1]; // Remove data:image/jpeg;base64, prefix

    // Upload via tRPC
    const result = await uploadMutation.mutateAsync({
      fileName: file.name,
      fileType: file.type,
      fileSize: file.size,
      base64Data,
    });

    console.log('Uploaded:', result.url);
  };

  return (
    <input
      type="file"
      onChange={(e) => {
        const file = e.target.files?.[0];
        if (file) handleFileUpload(file);
      }}
    />
  );
}
```

---

## Background Jobs

### Pattern: Async Router for Long-Running Tasks

```typescript
// src/server/routers/async/generation.ts
import { router, protectedProcedure } from '@/server/trpc';
import { z } from 'zod';
import { openai } from '@/server/modules/OpenAI';

export const generationRouter = router({
  generateImage: protectedProcedure
    .input(z.object({
      prompt: z.string(),
      size: z.enum(['256x256', '512x512', '1024x1024']),
    }))
    .mutation(async ({ input, ctx }) => {
      // This can take 10-30 seconds
      const response = await openai.images.generate({
        prompt: input.prompt,
        size: input.size,
        n: 1,
      });

      const imageUrl = response.data[0].url;

      // Save to database
      const [generation] = await serverDB.insert(generations)
        .values({
          userId: ctx.user.id,
          prompt: input.prompt,
          imageUrl,
        })
        .returning();

      return generation;
    }),

  generateContent: protectedProcedure
    .input(z.object({
      prompt: z.string(),
      model: z.enum(['gpt-4', 'gpt-3.5-turbo']),
    }))
    .mutation(async ({ input, ctx }) => {
      // This can take 5-15 seconds
      const response = await openai.chat.completions.create({
        model: input.model,
        messages: [{ role: 'user', content: input.prompt }],
      });

      const content = response.choices[0].message.content;

      // Save to database
      const [generation] = await serverDB.insert(generations)
        .values({
          userId: ctx.user.id,
          prompt: input.prompt,
          content,
        })
        .returning();

      return generation;
    }),
});
```

---

**Queue Pattern (for very long tasks):**

```typescript
import { Queue } from 'bullmq';
import Redis from 'ioredis';

const redis = new Redis({
  host: process.env.REDIS_HOST,
  port: parseInt(process.env.REDIS_PORT || '6379'),
});

const generationQueue = new Queue('generation', { connection: redis });

export const generationRouter = router({
  queueGeneration: protectedProcedure
    .input(z.object({
      prompt: z.string(),
    }))
    .mutation(async ({ input, ctx }) => {
      // Add job to queue (returns immediately)
      const job = await generationQueue.add('generate', {
        userId: ctx.user.id,
        prompt: input.prompt,
      });

      return {
        jobId: job.id,
        status: 'queued',
      };
    }),

  checkStatus: protectedProcedure
    .input(z.object({
      jobId: z.string(),
    }))
    .query(async ({ input }) => {
      const job = await generationQueue.getJob(input.jobId);

      if (!job) {
        throw new TRPCError({ code: 'NOT_FOUND' });
      }

      return {
        jobId: job.id,
        status: await job.getState(),
        progress: job.progress,
        result: job.returnvalue,
      };
    }),
});
```

---

## Streaming and Real-Time

### Server-Sent Events (SSE) with tRPC

**Streaming Chat Completions:**

```typescript
// src/server/routers/lambda/chat.ts
import { router, protectedProcedure } from '@/server/trpc';
import { observable } from '@trpc/server/observable';
import { z } from 'zod';
import { openai } from '@/server/modules/OpenAI';

export const chatRouter = router({
  streamChat: protectedProcedure
    .input(z.object({
      messages: z.array(z.object({
        role: z.enum(['user', 'assistant', 'system']),
        content: z.string(),
      })),
    }))
    .subscription(async function* ({ input }) {
      // Create streaming completion
      const stream = await openai.chat.completions.create({
        model: 'gpt-4',
        messages: input.messages,
        stream: true,
      });

      // Yield each chunk
      for await (const chunk of stream) {
        const content = chunk.choices[0]?.delta?.content || '';
        if (content) {
          yield { content, done: false };
        }
      }

      // Final chunk
      yield { content: '', done: true };
    }),
});
```

---

**Client-side streaming:**

```typescript
'use client';

import { trpc } from '@/utils/trpc';
import { useState } from 'react';

function ChatWindow() {
  const [response, setResponse] = useState('');

  trpc.lambda.chat.streamChat.useSubscription(
    {
      messages: [
        { role: 'user', content: 'Tell me a story' },
      ],
    },
    {
      onData: (chunk) => {
        if (!chunk.done) {
          setResponse((prev) => prev + chunk.content);
        }
      },
      onError: (error) => {
        console.error('Streaming error:', error);
      },
    }
  );

  return <div>{response}</div>;
}
```

---

### WebSocket Alternative

For bidirectional real-time communication:

```typescript
// src/server/routers/lambda/chat.ts
import { observable } from '@trpc/server/observable';
import { EventEmitter } from 'events';

const chatEvents = new EventEmitter();

export const chatRouter = router({
  // Subscribe to chat updates
  onChatUpdate: protectedProcedure
    .input(z.object({
      conversationId: z.string().uuid(),
    }))
    .subscription(({ input }) => {
      return observable<{ type: string; data: any }>((emit) => {
        const handler = (data: any) => {
          if (data.conversationId === input.conversationId) {
            emit.next(data);
          }
        };

        chatEvents.on('message', handler);

        return () => {
          chatEvents.off('message', handler);
        };
      });
    }),

  // Send message (triggers event)
  sendMessage: protectedProcedure
    .input(z.object({
      conversationId: z.string().uuid(),
      content: z.string(),
    }))
    .mutation(async ({ input, ctx }) => {
      const message = await MessageService.createMessage({
        conversationId: input.conversationId,
        userId: ctx.user.id,
        content: input.content,
      });

      // Emit event
      chatEvents.emit('message', {
        type: 'new_message',
        conversationId: input.conversationId,
        data: message,
      });

      return message;
    }),
});
```

---

## Performance Optimization

### 1. Response Caching

```typescript
import { redis } from '@/server/modules/Redis';

export const userRouter = router({
  getUser: publicProcedure
    .input(z.object({ id: z.string().uuid() }))
    .query(async ({ input }) => {
      const cacheKey = `user:${input.id}`;

      // Check cache
      const cached = await redis.get(cacheKey);
      if (cached) {
        return JSON.parse(cached);
      }

      // Query database
      const user = await UserService.getUser(input.id);

      // Cache for 5 minutes
      await redis.set(cacheKey, JSON.stringify(user), 'EX', 300);

      return user;
    }),
});
```

---

### 2. Request Deduplication

TanStack Query (used by tRPC) automatically deduplicates requests:

```typescript
// Multiple components call this simultaneously:
const { data: user } = trpc.lambda.user.getUser.useQuery({ id: '123' });

// Only ONE network request is made!
// All components share the same data.
```

---

### 3. Batch Requests

```typescript
// src/server/routers/lambda/user.ts
export const userRouter = router({
  getManyUsers: publicProcedure
    .input(z.object({
      ids: z.array(z.string().uuid()).max(100),
    }))
    .query(async ({ input }) => {
      return await serverDB.query.users.findMany({
        where: inArray(users.id, input.ids),
      });
    }),
});

// Client usage:
const { data: users } = trpc.lambda.user.getManyUsers.useQuery({
  ids: ['id1', 'id2', 'id3'],
});
// Single request for multiple users!
```

---

### 4. Pagination

```typescript
export const messageRouter = router({
  listMessages: publicProcedure
    .input(z.object({
      conversationId: z.string().uuid(),
      limit: z.number().min(1).max(100).default(20),
      cursor: z.string().uuid().optional(), // Cursor-based pagination
    }))
    .query(async ({ input }) => {
      const messages = await serverDB.query.messages.findMany({
        where: input.cursor
          ? and(
              eq(messages.conversationId, input.conversationId),
              lt(messages.createdAt, input.cursor)
            )
          : eq(messages.conversationId, input.conversationId),
        limit: input.limit + 1, // Fetch one extra to check if there's more
        orderBy: [desc(messages.createdAt)],
      });

      let nextCursor: string | undefined;
      if (messages.length > input.limit) {
        const nextItem = messages.pop();
        nextCursor = nextItem!.id;
      }

      return {
        messages,
        nextCursor,
      };
    }),
});
```

---

## Testing

### Unit Testing tRPC Routers

```typescript
// src/server/routers/lambda/__tests__/user.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { appRouter } from '@/server/routers';
import { createContext } from '@/server/trpc';

describe('userRouter', () => {
  let caller: ReturnType<typeof appRouter.createCaller>;

  beforeEach(async () => {
    // Create test context
    const ctx = await createContext({
      req: {} as any,
      res: {} as any,
    });

    caller = appRouter.createCaller(ctx);
  });

  it('should get user by id', async () => {
    const user = await caller.lambda.user.getUser({ id: 'test-id' });

    expect(user).toMatchObject({
      id: 'test-id',
      name: expect.any(String),
      email: expect.any(String),
    });
  });

  it('should throw error for invalid user', async () => {
    await expect(
      caller.lambda.user.getUser({ id: 'invalid-id' })
    ).rejects.toThrow('User not found');
  });
});
```

---

### Integration Testing with Database

```typescript
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import { serverDB } from '@lobechat/database/server';
import { users } from '@lobechat/database/schemas/user';

describe('User Integration Tests', () => {
  let testUserId: string;

  beforeAll(async () => {
    // Create test user
    const [user] = await serverDB.insert(users)
      .values({
        name: 'Test User',
        email: 'test@example.com',
      })
      .returning();

    testUserId = user.id;
  });

  afterAll(async () => {
    // Clean up
    await serverDB.delete(users).where(eq(users.id, testUserId));
  });

  it('should fetch user from database', async () => {
    const user = await caller.lambda.user.getUser({ id: testUserId });

    expect(user.name).toBe('Test User');
    expect(user.email).toBe('test@example.com');
  });
});
```

---

## Best Practices

### ✅ DO

1. **Use services for business logic**
   ```typescript
   // ✅ GOOD
   export const userRouter = router({
     createUser: publicProcedure
       .mutation(async ({ input }) => {
         return await UserService.createUser(input);
       }),
   });
   ```

2. **Validate all inputs with Zod**
   ```typescript
   .input(z.object({
     email: z.string().email(),
     age: z.number().min(0).max(150),
   }))
   ```

3. **Use proper error codes**
   ```typescript
   throw new TRPCError({
     code: 'NOT_FOUND',  // Correct HTTP status
     message: 'User not found',
   });
   ```

4. **Separate by runtime (Edge/Node)**
   ```typescript
   // Fast operations → lambda/ (Edge)
   // Long operations → async/ (Node)
   ```

5. **Use middleware for cross-cutting concerns**
   ```typescript
   const loggedProcedure = publicProcedure.use(logger);
   ```

---

### ❌ DON'T

1. **Don't put business logic in routers**
   ```typescript
   // ❌ BAD
   .mutation(async ({ input, ctx }) => {
     const user = await ctx.db.insert(users).values(input);
     // 50 lines of business logic...
   });
   ```

2. **Don't skip input validation**
   ```typescript
   // ❌ BAD: No validation
   .mutation(async ({ input }) => {
     await UserService.createUser(input);  // What if invalid?
   });
   ```

3. **Don't return sensitive data**
   ```typescript
   // ❌ BAD
   return user;  // Includes password hash!

   // ✅ GOOD
   const { password, ...safeUser } = user;
   return safeUser;
   ```

4. **Don't ignore authorization**
   ```typescript
   // ❌ BAD: No ownership check
   .mutation(async ({ input }) => {
     await MessageService.deleteMessage(input.id);
   });

   // ✅ GOOD: Verify ownership
   .mutation(async ({ input, ctx }) => {
     await MessageService.verifyOwnership(input.id, ctx.user.id);
     await MessageService.deleteMessage(input.id);
   });
   ```

5. **Don't use `any` types**
   ```typescript
   // ❌ BAD
   .input(z.any())

   // ✅ GOOD
   .input(z.object({ ... }))
   ```

---

## Summary

**Backend Architecture in One Paragraph:**

> LobeHub's backend separates routers (routing), services (business logic), and models (database) for clear concerns. tRPC provides end-to-end type safety with Zod validation. Edge runtime handles fast operations, Node runtime handles long tasks. Middleware adds authentication, logging, and rate limiting. Services use Drizzle ORM for type-safe database access.

**Key Patterns:**

1. 🗂️ **Service Layer** - Business logic separated from routing
2. 🔒 **Authentication** - Protected procedures with middleware
3. 📁 **File Handling** - Async router for uploads, presigned URLs
4. ⏰ **Background Jobs** - Async router or queue for long tasks
5. 📡 **Streaming** - Subscriptions for real-time data
6. ⚡ **Performance** - Caching, deduplication, batching, pagination
7. ✅ **Testing** - Unit tests for routers, integration tests for services

**Technologies Summary:**

| Technology | Purpose | Benefits |
|------------|---------|----------|
| **tRPC** | API layer | Type safety, no code generation |
| **Zod** | Validation | Runtime + compile-time checks |
| **Drizzle ORM** | Database | Type-safe queries |
| **Neon** | Database | Serverless PostgreSQL |
| **NextAuth** | Authentication | Session management |
| **Edge Runtime** | Fast operations | <100ms global latency |
| **Node Runtime** | Long operations | No timeout limits |

---

**Document Status:** ✅ Complete (Part 2 of 2) | **Last Updated:** November 20, 2025

**[← Part 1](./BACKEND-ARCHITECTURE-PART-01.md)** | **Next:** [Database Architecture](./DATABASE-ARCHITECTURE-PART-01.md)
