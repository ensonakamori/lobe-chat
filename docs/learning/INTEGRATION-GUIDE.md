# Integration Guide

**Documented:** November 2025
**Tech Stack Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)
**Target:** Mid-level developers proficient in JS/TS/React

This guide explains how all parts of LobeHub integrate together, from frontend to backend to database and external services.

**Prerequisites:**
- [Frontend Architecture](./FRONTEND-ARCHITECTURE-PART-01.md) - Understand frontend patterns
- [Backend Architecture](./BACKEND-ARCHITECTURE-PART-01.md) - Understand backend patterns
- [Database Architecture](./DATABASE-ARCHITECTURE-PART-01.md) - Understand database layer

**Time to read:** 40 minutes

---

## Table of Contents

1. [Integration Overview](#integration-overview)
2. [Frontend ↔ Backend Integration](#frontend--backend-integration)
3. [Backend ↔ Database Integration](#backend--database-integration)
4. [Client ↔ Server Database Sync](#client--server-database-sync)
5. [Third-Party Service Integration](#third-party-service-integration)
6. [Authentication Flow](#authentication-flow)
7. [Complete Feature Example](#complete-feature-example)
8. [Best Practices](#best-practices)

---

## Integration Overview

**LobeHub Integration Architecture:**

```
┌──────────────────────────────────────────────────────────────┐
│                     Browser (Client)                         │
│                                                              │
│  ┌─────────────┐    ┌─────────────┐    ┌───────────────┐   │
│  │   React     │ ←→ │   Zustand   │ ←→ │  PGLite DB    │   │
│  │ Components  │    │   Stores    │    │   (Client)    │   │
│  └─────────────┘    └──────┬──────┘    └───────┬───────┘   │
│         ↓                   ↓                   ↓           │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Client Services                        │   │
│  └─────────────────────┬───────────────────────────────┘   │
│                        ↓                                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              tRPC Client                            │   │
│  └─────────────────────────────────────────────────────┘   │
└──────────────────────────┼──────────────────────────────────┘
                           │
                      (HTTPS/WS)
                           │
┌──────────────────────────┼──────────────────────────────────┐
│                      Server                                  │
│                          ↓                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              tRPC Server                            │   │
│  └─────────────────────┬───────────────────────────────┘   │
│                        ↓                                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │           Server Services                           │   │
│  └─────────────────────┬───────────────────────────────┘   │
│                        ↓                                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │           Drizzle ORM                               │   │
│  └─────────────────────┬───────────────────────────────┘   │
│                        ↓                                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │           Neon PostgreSQL                           │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │        Third-Party Services                         │   │
│  │  • OpenAI API  • S3 Storage  • NextAuth           │   │
│  └─────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

---

## Frontend ↔ Backend Integration

### tRPC Setup

**1. Define Backend Router:**

```typescript
// src/server/routers/lambda/message.ts
import { router, protectedProcedure } from '@/server/trpc';
import { MessageService } from '@/server/services/message';
import { z } from 'zod';

export const messageRouter = router({
  list: protectedProcedure
    .input(z.object({ conversationId: z.string().uuid() }))
    .query(async ({ input, ctx }) => {
      return await MessageService.list(ctx.user.id, input.conversationId);
    }),

  create: protectedProcedure
    .input(z.object({
      conversationId: z.string().uuid(),
      content: z.string().min(1),
    }))
    .mutation(async ({ input, ctx }) => {
      return await MessageService.create({
        userId: ctx.user.id,
        conversationId: input.conversationId,
        content: input.content,
      });
    }),
});
```

---

**2. Create Frontend Hook:**

```typescript
// src/services/message/hooks/useMessages.ts
import { trpc } from '@/utils/trpc';

export function useMessages(conversationId: string) {
  const { data: messages, isLoading, error } = trpc.lambda.message.list.useQuery(
    { conversationId },
    {
      enabled: !!conversationId,  // Only fetch if conversationId exists
      refetchInterval: 5000,      // Refetch every 5s
    }
  );

  const createMutation = trpc.lambda.message.create.useMutation({
    onSuccess: () => {
      // Invalidate list to refetch
      trpc.useContext().lambda.message.list.invalidate({ conversationId });
    },
  });

  const createMessage = async (content: string) => {
    return await createMutation.mutateAsync({ conversationId, content });
  };

  return {
    messages,
    loading: isLoading,
    error,
    createMessage,
  };
}
```

---

**3. Use in Component:**

```typescript
// src/app/chat/components/MessageList.tsx
'use client';

import { useMessages } from '@/services/message/hooks/useMessages';

export function MessageList({ conversationId }) {
  const { messages, loading, createMessage } = useMessages(conversationId);

  if (loading) return <Skeleton />;

  return (
    <div>
      {messages?.map((msg) => (
        <div key={msg.id}>{msg.content}</div>
      ))}
      <button onClick={() => createMessage('Hello!')}>
        Send
      </button>
    </div>
  );
}
```

**🎯 Key Integration Points:**
1. tRPC provides type safety from server to client
2. React Query (via tRPC) handles caching and refetching
3. Custom hooks encapsulate tRPC calls
4. Components consume hooks for clean separation

---

## Backend ↔ Database Integration

### Service → Model → Repository Pattern

**1. Repository (Complex Queries):**

```typescript
// packages/database/src/repositories/message.ts
import { serverDB } from '../server';
import { messages } from '../schemas/message';
import { eq, and, desc } from 'drizzle-orm';

export class MessageRepository {
  static async findWithUser(conversationId: string) {
    return await serverDB.query.messages.findMany({
      where: eq(messages.conversationId, conversationId),
      with: {
        user: true,  // Include user data
      },
      orderBy: [desc(messages.createdAt)],
    });
  }

  static async findRecent(userId: string, limit: number = 10) {
    return await serverDB.query.messages.findMany({
      where: eq(messages.userId, userId),
      limit,
      orderBy: [desc(messages.createdAt)],
    });
  }
}
```

---

**2. Model (CRUD Operations):**

```typescript
// packages/database/src/models/message.ts
import { messages, type NewMessage } from '../schemas/message';
import { eq } from 'drizzle-orm';

export class MessageModel {
  static async create(db: Database, data: NewMessage) {
    const [message] = await db.insert(messages)
      .values(data)
      .returning();
    return message;
  }

  static async findById(db: Database, id: string) {
    return await db.query.messages.findFirst({
      where: eq(messages.id, id),
    });
  }

  static async update(db: Database, id: string, data: Partial<NewMessage>) {
    const [updated] = await db.update(messages)
      .set(data)
      .where(eq(messages.id, id))
      .returning();
    return updated;
  }

  static async delete(db: Database, id: string) {
    await db.delete(messages).where(eq(messages.id, id));
  }
}
```

---

**3. Service (Business Logic):**

```typescript
// src/server/services/message/index.ts
import { serverDB } from '@lobechat/database/server';
import { MessageModel } from '@lobechat/database/models/message';
import { MessageRepository } from '@lobechat/database/repositories/message';
import { TRPCError } from '@trpc/server';

export class MessageService {
  static async list(userId: string, conversationId: string) {
    // Verify user owns conversation
    const conversation = await serverDB.query.conversations.findFirst({
      where: and(
        eq(conversations.id, conversationId),
        eq(conversations.userId, userId)
      ),
    });

    if (!conversation) {
      throw new TRPCError({ code: 'FORBIDDEN' });
    }

    // Use repository for complex query
    return await MessageRepository.findWithUser(conversationId);
  }

  static async create(data: {
    userId: string;
    conversationId: string;
    content: string;
  }) {
    // Validate conversation exists
    const conversation = await serverDB.query.conversations.findFirst({
      where: eq(conversations.id, data.conversationId),
    });

    if (!conversation) {
      throw new TRPCError({ code: 'NOT_FOUND' });
    }

    // Use model for CRUD
    return await MessageModel.create(serverDB, data);
  }
}
```

**🎯 Integration Flow:**
1. tRPC Router → Service (business logic)
2. Service → Repository (complex queries) or Model (CRUD)
3. Model/Repository → Drizzle ORM → PostgreSQL

---

## Client ↔ Server Database Sync

### Sync Pattern

**1. Client Service (Optimistic Update + Sync):**

```typescript
// src/services/message/client.ts
import { clientDB } from '@lobechat/database/client';
import { MessageModel } from '@lobechat/database/models/message';
import { trpc } from '@/utils/trpc';

export class MessageClientService {
  // Create message (optimistic)
  static async create(data: {
    conversationId: string;
    content: string;
  }) {
    // 1. Save to PGLite immediately
    const message = await MessageModel.create(clientDB, {
      ...data,
      userId: 'current-user-id',  // Get from session
      synced: false,  // Mark as not synced
    });

    // 2. Sync to server in background
    try {
      await trpc.lambda.message.create.mutate(data);

      // 3. Mark as synced
      await MessageModel.update(clientDB, message.id, { synced: true });
    } catch (error) {
      // 4. Queue for retry if offline
      await this.queueForSync(message.id);
    }

    return message;
  }

  // Queue message for later sync
  private static async queueForSync(messageId: string) {
    await clientDB.insert(syncQueue).values({
      entity: 'message',
      entityId: messageId,
      action: 'create',
    });
  }

  // Sync pending messages
  static async syncPending() {
    const pending = await clientDB.query.syncQueue.findMany();

    for (const item of pending) {
      try {
        const message = await MessageModel.findById(clientDB, item.entityId);

        if (message) {
          await trpc.lambda.message.create.mutate({
            conversationId: message.conversationId,
            content: message.content,
          });

          // Remove from queue
          await clientDB.delete(syncQueue).where(eq(syncQueue.id, item.id));

          // Mark as synced
          await MessageModel.update(clientDB, message.id, { synced: true });
        }
      } catch (error) {
        console.error('Sync failed:', error);
      }
    }
  }
}
```

---

**2. Zustand Store Integration:**

```typescript
// src/store/chat/slices/message/action.ts
import { MessageClientService } from '@/services/message/client';

export const createMessageSlice = (set, get) => ({
  messages: [],

  sendMessage: async (content: string) => {
    const conversationId = get().currentConversationId;

    // Optimistic update
    const tempMessage = {
      id: `temp-${Date.now()}`,
      content,
      status: 'sending',
    };

    set((state) => ({
      messages: [...state.messages, tempMessage],
    }));

    try {
      // Save to PGLite + sync to server
      const message = await MessageClientService.create({
        conversationId,
        content,
      });

      // Replace temp with real
      set((state) => ({
        messages: state.messages.map((m) =>
          m.id === tempMessage.id ? { ...message, status: 'sent' } : m
        ),
      }));
    } catch (error) {
      // Remove temp message on error
      set((state) => ({
        messages: state.messages.filter((m) => m.id !== tempMessage.id),
      }));

      throw error;
    }
  },
});
```

**🎯 Sync Flow:**
1. User action → Zustand store
2. Store → Client service → PGLite (instant)
3. Client service → tRPC → Server (background)
4. Server → Neon database (persistent)

---

## Third-Party Service Integration

### OpenAI Integration Example

**1. Create Module:**

```typescript
// src/server/modules/OpenAI/index.ts
import OpenAI from 'openai';

export const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateCompletion(prompt: string) {
  const response = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: [{ role: 'user', content: prompt }],
  });

  return response.choices[0].message.content;
}

export async function* streamCompletion(prompt: string) {
  const stream = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: [{ role: 'user', content: prompt }],
    stream: true,
  });

  for await (const chunk of stream) {
    yield chunk.choices[0]?.delta?.content || '';
  }
}
```

---

**2. Use in Service:**

```typescript
// src/server/services/ai/index.ts
import { generateCompletion, streamCompletion } from '@/server/modules/OpenAI';
import { MessageService } from '../message';

export class AIService {
  static async generateResponse(conversationId: string, prompt: string) {
    // Save user message
    await MessageService.create({
      conversationId,
      role: 'user',
      content: prompt,
    });

    // Generate AI response
    const response = await generateCompletion(prompt);

    // Save AI message
    const aiMessage = await MessageService.create({
      conversationId,
      role: 'assistant',
      content: response,
    });

    return aiMessage;
  }

  static async* streamResponse(conversationId: string, prompt: string) {
    // Save user message
    await MessageService.create({
      conversationId,
      role: 'user',
      content: prompt,
    });

    let fullResponse = '';

    // Stream AI response
    for await (const chunk of streamCompletion(prompt)) {
      fullResponse += chunk;
      yield chunk;
    }

    // Save complete AI message
    await MessageService.create({
      conversationId,
      role: 'assistant',
      content: fullResponse,
    });
  }
}
```

---

**3. Expose via tRPC:**

```typescript
// src/server/routers/lambda/ai.ts
import { router, protectedProcedure } from '@/server/trpc';
import { AIService } from '@/server/services/ai';
import { z } from 'zod';

export const aiRouter = router({
  chat: protectedProcedure
    .input(z.object({
      conversationId: z.string().uuid(),
      prompt: z.string(),
    }))
    .mutation(async ({ input }) => {
      return await AIService.generateResponse(
        input.conversationId,
        input.prompt
      );
    }),

  streamChat: protectedProcedure
    .input(z.object({
      conversationId: z.string().uuid(),
      prompt: z.string(),
    }))
    .subscription(async function* ({ input }) {
      yield* AIService.streamResponse(input.conversationId, input.prompt);
    }),
});
```

---

## Authentication Flow

### Complete Auth Integration

**1. NextAuth Setup:**

```typescript
// src/app/api/auth/[...nextauth]/route.ts
import NextAuth from 'next-auth';
import type { NextAuthOptions } from 'next-auth';
import CredentialsProvider from 'next-auth/providers/credentials';
import { serverDB } from '@lobechat/database/server';
import bcrypt from 'bcrypt';

export const authOptions: NextAuthOptions = {
  providers: [
    CredentialsProvider({
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' },
      },
      async authorize(credentials) {
        const user = await serverDB.query.users.findFirst({
          where: eq(users.email, credentials.email),
        });

        if (!user) return null;

        const valid = await bcrypt.compare(
          credentials.password,
          user.password
        );

        if (!valid) return null;

        return {
          id: user.id,
          name: user.name,
          email: user.email,
        };
      },
    }),
  ],
  callbacks: {
    async jwt({ token, user }) {
      if (user) {
        token.id = user.id;
      }
      return token;
    },
    async session({ session, token }) {
      if (session.user) {
        session.user.id = token.id as string;
      }
      return session;
    },
  },
};

const handler = NextAuth(authOptions);
export { handler as GET, handler as POST };
```

---

**2. Auth Provider:**

```typescript
// src/layout/AuthProvider/index.tsx
'use client';

import { SessionProvider } from 'next-auth/react';

export function AuthProvider({ children }) {
  return <SessionProvider>{children}</SessionProvider>;
}
```

---

**3. Use in Components:**

```typescript
'use client';

import { useSession, signIn, signOut } from 'next-auth/react';

export function UserMenu() {
  const { data: session, status } = useSession();

  if (status === 'loading') return <div>Loading...</div>;

  if (!session) {
    return <button onClick={() => signIn()}>Sign In</button>;
  }

  return (
    <div>
      <p>Welcome, {session.user.name}</p>
      <button onClick={() => signOut()}>Sign Out</button>
    </div>
  );
}
```

---

**4. Protect tRPC Routes:**

```typescript
// src/server/trpc.ts
import { getServerSession } from 'next-auth';
import { authOptions } from '@/app/api/auth/[...nextauth]/route';

export async function createContext({ req, res }) {
  const session = await getServerSession(req, res, authOptions);

  return {
    session,
    db: serverDB,
  };
}

export const protectedProcedure = publicProcedure.use(async ({ ctx, next }) => {
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
```

---

## Complete Feature Example

### Building a Chat Feature (End-to-End)

**1. Database Schema:**

```typescript
// packages/database/src/schemas/message.ts
export const messages = pgTable('messages', {
  id: uuid('id').defaultRandom().primaryKey(),
  conversationId: uuid('conversation_id').notNull().references(() => conversations.id),
  userId: uuid('user_id').notNull().references(() => users.id),
  role: messageRoleEnum('role').notNull(),
  content: text('content').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});
```

---

**2. Backend Service:**

```typescript
// src/server/services/message/index.ts
export class MessageService {
  static async create(data) {
    return await MessageModel.create(serverDB, data);
  }

  static async list(conversationId: string) {
    return await MessageRepository.findWithUser(conversationId);
  }
}
```

---

**3. tRPC Router:**

```typescript
// src/server/routers/lambda/message.ts
export const messageRouter = router({
  list: protectedProcedure
    .input(z.object({ conversationId: z.string().uuid() }))
    .query(async ({ input }) => MessageService.list(input.conversationId)),

  create: protectedProcedure
    .input(z.object({ conversationId: z.string().uuid(), content: z.string() }))
    .mutation(async ({ input, ctx }) =>
      MessageService.create({ ...input, userId: ctx.user.id })
    ),
});
```

---

**4. Client Service:**

```typescript
// src/services/message/client.ts
export class MessageClientService {
  static async create(data) {
    const message = await MessageModel.create(clientDB, data);
    await trpc.lambda.message.create.mutate(data);
    return message;
  }
}
```

---

**5. Zustand Store:**

```typescript
// src/store/chat/slices/message/action.ts
export const createMessageSlice = (set) => ({
  messages: [],
  sendMessage: async (content) => {
    const message = await MessageClientService.create({ content });
    set((s) => ({ messages: [...s.messages, message] }));
  },
});
```

---

**6. React Component:**

```typescript
// src/features/ChatWindow/index.tsx
'use client';

export function ChatWindow() {
  const messages = useChatStore((s) => s.messages);
  const sendMessage = useChatStore((s) => s.sendMessage);
  const [input, setInput] = useState('');

  return (
    <div>
      {messages.map((msg) => (
        <div key={msg.id}>{msg.content}</div>
      ))}
      <input value={input} onChange={(e) => setInput(e.target.value)} />
      <button onClick={() => sendMessage(input)}>Send</button>
    </div>
  );
}
```

**🎯 Complete Integration:**
React → Zustand → Client Service → PGLite + tRPC → Server Service → Neon

---

## Best Practices

### ✅ DO

1. **Use type-safe APIs (tRPC)**
2. **Separate concerns (Service layer)**
3. **Handle offline gracefully**
4. **Cache frequently accessed data**
5. **Validate inputs at boundaries**
6. **Use optimistic updates for UX**
7. **Implement proper error handling**
8. **Monitor third-party API usage**

### ❌ DON'T

1. **Don't mix business logic in components**
2. **Don't skip input validation**
3. **Don't expose sensitive data**
4. **Don't block UI on network requests**
5. **Don't ignore sync failures**
6. **Don't hardcode API keys in code**
7. **Don't trust client-side data without validation**

---

## Summary

**Integration in One Paragraph:**

> LobeHub integrates React (UI) → Zustand (state) → Client Services (logic) → tRPC (API) → Server Services (business logic) → Drizzle ORM (queries) → PostgreSQL (storage). PGLite provides instant client operations while Neon ensures cloud persistence. All components are type-safe end-to-end with TypeScript and tRPC.

**Key Integration Points:**

1. 🔗 **Frontend ↔ Backend** - tRPC for type-safe APIs
2. 🗄️ **Backend ↔ Database** - Service → Model/Repository → Drizzle
3. 🔄 **Client ↔ Server DB** - Optimistic updates + background sync
4. 🔌 **Third-Party Services** - Wrapped in modules, used in services
5. 🔐 **Authentication** - NextAuth throughout the stack

---

**Document Status:** ✅ Complete | **Last Updated:** November 20, 2025

**Next:** [Patterns & Conventions](./PATTERNS_AND_CONVENTIONS.md) - Code style and best practices
