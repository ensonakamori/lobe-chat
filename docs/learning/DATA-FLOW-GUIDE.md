# Data Flow Guide

**Documented:** November 2025
**Tech Stack Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)
**Target:** Mid-level developers proficient in JS/TS/React

This guide explains how data flows through the LobeHub system, from user interaction to database persistence and back.

**Prerequisites:**
- [Architecture Overview](./ARCHITECTURE-OVERVIEW-PART-01.md) - Understand the system architecture
- [Tech Stack Guide](./TECH-STACK-GUIDE-PART-01.md) - Familiar with the technologies

**Time to read:** 45 minutes

---

## Table of Contents

1. [The Big Picture](#the-big-picture)
2. [Data Flow Patterns](#data-flow-patterns)
3. [Client-Side Data Flow](#client-side-data-flow)
4. [Server-Side Data Flow](#server-side-data-flow)
5. [Database Sync Strategy](#database-sync-strategy)
6. [Real-World Example: Sending a Message](#real-world-example-sending-a-message)
7. [Performance Optimization](#performance-optimization)
8. [Error Handling](#error-handling)
9. [Best Practices](#best-practices)

---

## The Big Picture

**🧠 Mental Model:** Data flows in layers, with each layer having a specific responsibility.

```
┌─────────────────────────────────────────────────────────────┐
│                       User Interface                        │
│              (React Components + Hooks)                     │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ↓
┌─────────────────────────────────────────────────────────────┐
│                    Global State                             │
│                  (Zustand Stores)                           │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ↓
┌─────────────────────────────────────────────────────────────┐
│                  Client Services                            │
│            (Business Logic + Data Layer)                    │
└─────────────────┬───────────────────┬───────────────────────┘
                  │                   │
                  ↓                   ↓
         ┌────────────────┐   ┌─────────────────┐
         │  PGLite (Web)  │   │  tRPC Client    │
         │ Client Database│   │   (API Calls)   │
         └────────────────┘   └────────┬────────┘
                                       │
                                       ↓ Network
                              ┌─────────────────┐
                              │   tRPC Server   │
                              └────────┬────────┘
                                       │
                                       ↓
                              ┌─────────────────┐
                              │ Server Services │
                              └────────┬────────┘
                                       │
                                       ↓
                              ┌─────────────────┐
                              │ Neon PostgreSQL │
                              │ Server Database │
                              └─────────────────┘
```

**Key Principles:**

1. **Unidirectional flow** - Data flows down, actions flow up
2. **Separation of concerns** - Each layer has a specific purpose
3. **Optimistic updates** - Update UI immediately, sync in background
4. **Offline-first** - App works without network connection

---

## Data Flow Patterns

LobeHub uses different data flow patterns depending on the use case:

### Pattern 1: Client-Only Flow (Fastest)

For offline functionality and instant UX:

```
User Action
    ↓
Component
    ↓
Zustand Store (update state)
    ↓
Client Service
    ↓
PGLite (save)
    ↓
Component Re-renders
```

**Example:** User creates a new chat message while offline

**Characteristics:**
- ✅ Instant (no network latency)
- ✅ Works offline
- ❌ No cross-device sync
- ❌ No backup

---

### Pattern 2: Client-First with Background Sync (Balanced)

For best UX with cloud backup:

```
User Action
    ↓
Component
    ↓
Zustand Store (optimistic update)
    ↓
Client Service
    ↓
PGLite (save immediately)
    ↓
Component Re-renders (instant!)
    ↓
tRPC Mutation (background sync)
    ↓
Server Service
    ↓
Neon PostgreSQL (backup)
```

**Example:** User sends a chat message

**Characteristics:**
- ✅ Instant UI update
- ✅ Works offline (queued for sync)
- ✅ Cross-device sync when online
- ✅ Cloud backup

**🎯 This is LobeHub's primary pattern!**

---

### Pattern 3: Server-First Flow (Authoritative)

For operations requiring server validation:

```
User Action
    ↓
Component
    ↓
tRPC Mutation
    ↓
Server Service (validate)
    ↓
Neon PostgreSQL (save)
    ↓
tRPC Response
    ↓
Client Service
    ↓
PGLite (sync from server)
    ↓
Zustand Store (update)
    ↓
Component Re-renders
```

**Example:** User purchases a subscription

**Characteristics:**
- ❌ Requires network
- ✅ Server validation (security)
- ✅ Authoritative source of truth
- ✅ Consistent across devices

---

## Client-Side Data Flow

### Layer 1: React Components

Components are the entry point for user interactions:

```typescript
// src/app/[variants]/(main)/chat/components/ChatInput.tsx
'use client';

import { useChatStore } from '@/store/chat';
import { useState } from 'react';

function ChatInput() {
  const [input, setInput] = useState('');
  const sendMessage = useChatStore((s) => s.sendMessage);

  const handleSend = async () => {
    await sendMessage(input);  // Triggers store action
    setInput('');
  };

  return (
    <div>
      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
      />
      <button onClick={handleSend}>Send</button>
    </div>
  );
}
```

**Responsibility:** UI rendering and user interaction

---

### Layer 2: Zustand Stores

Stores manage global state and coordinate data flow:

```typescript
// src/store/chat/slices/message/action.ts
import { messageService } from '@/services/message';

export const createMessageSlice = (set, get) => ({
  messages: [],

  sendMessage: async (content) => {
    const tempMessage = {
      id: `temp-${Date.now()}`,
      content,
      status: 'sending',
    };

    // 1. Optimistic update (instant UI)
    set((state) => ({
      messages: [...state.messages, tempMessage],
    }));

    try {
      // 2. Save to client database
      const savedMessage = await messageService.create({ content });

      // 3. Update with real ID
      set((state) => ({
        messages: state.messages.map((m) =>
          m.id === tempMessage.id ? savedMessage : m
        ),
      }));

      // 4. Sync to server in background
      await messageService.syncToServer(savedMessage);

      // 5. Mark as synced
      set((state) => ({
        messages: state.messages.map((m) =>
          m.id === savedMessage.id ? { ...m, status: 'sent' } : m
        ),
      }));
    } catch (error) {
      // Rollback on error
      set((state) => ({
        messages: state.messages.filter((m) => m.id !== tempMessage.id),
      }));
      throw error;
    }
  },
});
```

**Responsibility:** State management and orchestration

---

### Layer 3: Client Services

Services encapsulate business logic and data access:

```typescript
// src/services/message/client.ts
import { clientDB } from '@lobechat/database/client';
import { messages } from '@lobechat/database/schemas/message';
import { trpc } from '@/utils/trpc';

class MessageService {
  // Read from client database (instant)
  async getAll() {
    return await clientDB.query.messages.findMany();
  }

  // Create in client database (instant)
  async create(data) {
    const [message] = await clientDB.insert(messages).values(data).returning();
    return message;
  }

  // Sync to server (background)
  async syncToServer(message) {
    try {
      await trpc.message.create.mutate({
        id: message.id,
        content: message.content,
      });
    } catch (error) {
      // Queue for retry if offline
      await this.queueForSync(message);
    }
  }

  // Queue for later sync when back online
  async queueForSync(message) {
    await clientDB.insert(syncQueue).values({
      entity: 'message',
      entityId: message.id,
      action: 'create',
    });
  }
}

export const messageService = new MessageService();
```

**Responsibility:** Business logic and data persistence

---

## Server-Side Data Flow

### Layer 1: tRPC Routers

Routers define type-safe API endpoints:

```typescript
// src/server/routers/lambda/message.ts
import { router, protectedProcedure } from '@/server/trpc';
import { MessageService } from '@/server/services/message';
import { z } from 'zod';

export const messageRouter = router({
  create: protectedProcedure
    .input(
      z.object({
        id: z.string(),
        content: z.string(),
      })
    )
    .mutation(async ({ ctx, input }) => {
      // Call server service
      return await MessageService.create(ctx.user.id, input);
    }),

  list: protectedProcedure
    .query(async ({ ctx }) => {
      return await MessageService.listForUser(ctx.user.id);
    }),
});
```

**Responsibility:** API endpoint definition and validation

---

### Layer 2: Server Services

Services contain server-side business logic:

```typescript
// src/server/services/message/index.ts
import { serverDB } from '@lobechat/database/server';
import { messages } from '@lobechat/database/schemas/message';
import { eq } from 'drizzle-orm';

export class MessageService {
  static async create(userId: string, data: { id: string; content: string }) {
    // Validate user owns conversation
    const conversation = await serverDB.query.conversations.findFirst({
      where: eq(conversations.userId, userId),
    });

    if (!conversation) {
      throw new Error('Conversation not found');
    }

    // Save to Neon database
    const [message] = await serverDB
      .insert(messages)
      .values({
        id: data.id,
        conversationId: conversation.id,
        content: data.content,
        userId,
      })
      .returning();

    return message;
  }

  static async listForUser(userId: string) {
    return await serverDB.query.messages.findMany({
      where: eq(messages.userId, userId),
      orderBy: (messages, { desc }) => [desc(messages.createdAt)],
    });
  }
}
```

**Responsibility:** Server-side business logic and database access

---

### Layer 3: Database (Neon PostgreSQL)

The database is the source of truth for persisted data:

```sql
-- Drizzle generates and runs migrations
CREATE TABLE messages (
  id UUID PRIMARY KEY,
  conversation_id UUID NOT NULL REFERENCES conversations(id),
  user_id UUID NOT NULL REFERENCES users(id),
  content TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_messages_user_id ON messages(user_id);
CREATE INDEX idx_messages_conversation_id ON messages(conversation_id);
```

**Responsibility:** Persistent storage and data integrity

---

## Database Sync Strategy

### Sync Architecture

LobeHub uses a **bidirectional sync** strategy:

```
┌──────────────────────────────────────────────────────────┐
│                  Browser (Client)                        │
│                                                          │
│  ┌────────────────────────────────────────────────┐    │
│  │            PGLite Database                     │    │
│  │  • messages table                              │    │
│  │  • conversations table                         │    │
│  │  • sync_queue table (pending syncs)            │    │
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
│  │          Neon PostgreSQL Database              │    │
│  │  • messages table                              │    │
│  │  • conversations table                         │    │
│  │  • sync_log table (sync history)               │    │
│  └────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────┘
```

---

### Sync Patterns

**1. Push Sync (Client → Server)**

Upload local changes to server:

```typescript
// src/services/sync/client.ts
class SyncManager {
  async pushChanges() {
    // 1. Get pending changes from sync queue
    const pending = await clientDB.query.syncQueue.findMany();

    for (const item of pending) {
      try {
        // 2. Send to server via tRPC
        await trpc.sync.push.mutate({
          entity: item.entity,
          entityId: item.entityId,
          action: item.action,
          data: item.data,
        });

        // 3. Remove from queue on success
        await clientDB.delete(syncQueue).where(eq(syncQueue.id, item.id));
      } catch (error) {
        // Keep in queue for retry
        console.error('Push failed:', error);
      }
    }
  }
}
```

---

**2. Pull Sync (Server → Client)**

Download server changes to client:

```typescript
class SyncManager {
  async pullChanges() {
    // 1. Get last sync timestamp
    const lastSync = await this.getLastSyncTime();

    // 2. Fetch changes since last sync
    const changes = await trpc.sync.pull.query({
      since: lastSync,
    });

    // 3. Apply changes to PGLite
    for (const change of changes) {
      await this.applyChange(change);
    }

    // 4. Update last sync time
    await this.setLastSyncTime(new Date());
  }

  async applyChange(change) {
    switch (change.action) {
      case 'create':
        await clientDB.insert(messages).values(change.data);
        break;
      case 'update':
        await clientDB
          .update(messages)
          .set(change.data)
          .where(eq(messages.id, change.entityId));
        break;
      case 'delete':
        await clientDB.delete(messages).where(eq(messages.id, change.entityId));
        break;
    }
  }
}
```

---

**3. Conflict Resolution**

Handle conflicts when client and server have different versions:

```typescript
class SyncManager {
  async resolveConflict(clientData, serverData) {
    // Strategy: Last-write-wins (server timestamp authoritative)
    if (serverData.updatedAt > clientData.updatedAt) {
      // Server version is newer, accept server
      return serverData;
    } else {
      // Client version is newer, push to server
      await trpc.sync.push.mutate({
        entity: 'message',
        entityId: clientData.id,
        action: 'update',
        data: clientData,
      });
      return clientData;
    }
  }
}
```

---

### Sync Triggers

Sync happens on these events:

1. **User action** - After creating/updating/deleting data
2. **App focus** - When user switches back to tab
3. **Network reconnect** - When internet connection restored
4. **Periodic** - Every 5 minutes (configurable)
5. **Manual** - User clicks "Sync Now" button

```typescript
// src/services/sync/client.ts
class SyncManager {
  start() {
    // Trigger 1: After user action (handled in stores)

    // Trigger 2: On app focus
    window.addEventListener('focus', () => this.sync());

    // Trigger 3: On network reconnect
    window.addEventListener('online', () => this.sync());

    // Trigger 4: Periodic (every 5 minutes)
    setInterval(() => this.sync(), 5 * 60 * 1000);
  }

  async sync() {
    await this.pushChanges(); // Upload first
    await this.pullChanges(); // Then download
  }
}
```

---

## Real-World Example: Sending a Message

Let's trace a message from user input to database:

### Step 1: User Types Message

```typescript
// Component: ChatInput.tsx
function ChatInput() {
  const [input, setInput] = useState('');

  return (
    <input
      value={input}
      onChange={(e) => setInput(e.target.value)}
      // User types: "Hello, world!"
    />
  );
}
```

---

### Step 2: User Clicks Send

```typescript
// Component: ChatInput.tsx
const sendMessage = useChatStore((s) => s.sendMessage);

<button onClick={() => {
  sendMessage(input);  // Call store action
  setInput('');
}}>
  Send
</button>
```

---

### Step 3: Store Creates Optimistic Message

```typescript
// Store: chat/slices/message/action.ts
sendMessage: async (content) => {
  const tempMessage = {
    id: `temp-${Date.now()}`,
    content: 'Hello, world!',
    status: 'sending',
    createdAt: new Date(),
  };

  // Optimistic update (instant UI)
  set((state) => ({
    messages: [...state.messages, tempMessage],
  }));

  // Component re-renders immediately with new message!
```

---

### Step 4: Service Saves to PGLite

```typescript
// Service: message/client.ts
const savedMessage = await messageService.create({
  content: 'Hello, world!',
  conversationId: currentConversationId,
});

// Inside create():
async create(data) {
  const [message] = await clientDB
    .insert(messages)
    .values({
      id: uuid(),  // Real UUID instead of temp
      content: data.content,
      conversationId: data.conversationId,
      createdAt: new Date(),
    })
    .returning();

  return message;  // { id: 'uuid-123', content: 'Hello, world!', ... }
}
```

---

### Step 5: Store Updates with Real ID

```typescript
// Store: chat/slices/message/action.ts
set((state) => ({
  messages: state.messages.map((m) =>
    m.id === tempMessage.id  // Replace temp with real
      ? savedMessage
      : m
  ),
}));
```

---

### Step 6: Service Syncs to Server (Background)

```typescript
// Service: message/client.ts
await messageService.syncToServer(savedMessage);

// Inside syncToServer():
async syncToServer(message) {
  try {
    await trpc.message.create.mutate({
      id: message.id,
      content: message.content,
      conversationId: message.conversationId,
    });

    // Success! Server has the message
  } catch (error) {
    // Offline? Queue for later
    await this.queueForSync(message);
  }
}
```

---

### Step 7: Server Validates and Saves

```typescript
// Server Router: routers/lambda/message.ts
create: protectedProcedure
  .input(z.object({
    id: z.string(),
    content: z.string(),
    conversationId: z.string(),
  }))
  .mutation(async ({ ctx, input }) => {
    // Validate user owns conversation
    const conversation = await serverDB.query.conversations.findFirst({
      where: and(
        eq(conversations.id, input.conversationId),
        eq(conversations.userId, ctx.user.id)
      ),
    });

    if (!conversation) {
      throw new Error('Unauthorized');
    }

    // Save to Neon database
    return await serverDB.insert(messages).values({
      id: input.id,
      content: input.content,
      conversationId: input.conversationId,
      userId: ctx.user.id,
    }).returning();
  })
```

---

### Step 8: Store Marks as Synced

```typescript
// Store: chat/slices/message/action.ts
set((state) => ({
  messages: state.messages.map((m) =>
    m.id === savedMessage.id
      ? { ...m, status: 'sent' }  // Update status
      : m
  ),
}));

// Component re-renders with "sent" indicator ✓
```

---

### Timeline Summary

```
0ms    - User clicks "Send"
1ms    - Optimistic message appears in UI ✅
5ms    - Message saved to PGLite (browser database)
10ms   - Store updated with real ID
100ms  - Network request to server starts
200ms  - Server validates and saves to Neon
250ms  - Server responds with success
255ms  - Store marks message as "sent" ✓
```

**🎯 Key insight:** User sees message after 1ms, but sync takes 250ms in background!

---

## Performance Optimization

### Optimization 1: Selective Re-renders

Use Zustand selectors to prevent unnecessary re-renders:

```typescript
// ❌ BAD: Re-renders on any store change
function ChatMessage() {
  const store = useChatStore();
  return <div>{store.messages[0].content}</div>;
}

// ✅ GOOD: Only re-renders when messages change
function ChatMessage() {
  const firstMessage = useChatStore((s) => s.messages[0]);
  return <div>{firstMessage.content}</div>;
}

// ✅ BETTER: Use useShallow for objects
import { useShallow } from 'zustand/react/shallow';

function ChatMessage() {
  const { message, user } = useChatStore(
    useShallow((s) => ({
      message: s.messages[0],
      user: s.user,
    }))
  );
  return <div>{message.content} by {user.name}</div>;
}
```

---

### Optimization 2: Batched Updates

Batch multiple state updates:

```typescript
// ❌ BAD: Multiple re-renders
set({ messages: newMessages });  // Re-render 1
set({ loading: false });        // Re-render 2
set({ error: null });           // Re-render 3

// ✅ GOOD: Single re-render
set({
  messages: newMessages,
  loading: false,
  error: null,
});
```

---

### Optimization 3: Lazy Loading

Load data on demand:

```typescript
// List: Load minimal data
const messages = await clientDB.query.messages.findMany({
  columns: {
    id: true,
    content: true,
    createdAt: true,
  },
  limit: 50,  // Only latest 50
});

// Detail: Load full data when needed
const fullMessage = await clientDB.query.messages.findFirst({
  where: eq(messages.id, messageId),
  with: {
    attachments: true,
    reactions: true,
  },
});
```

---

### Optimization 4: Debounced Sync

Don't sync on every keystroke:

```typescript
import { useDebounce } from 'ahooks';

function ChatInput() {
  const [input, setInput] = useState('');
  const debouncedInput = useDebounce(input, { wait: 500 });

  useEffect(() => {
    // Only saves draft after user stops typing for 500ms
    if (debouncedInput) {
      saveDraft(debouncedInput);
    }
  }, [debouncedInput]);
}
```

---

## Error Handling

### Pattern 1: Graceful Degradation

Handle offline state gracefully:

```typescript
async sendMessage(content) {
  // 1. Always save locally first
  const message = await messageService.create({ content });

  // 2. Try to sync, but don't block UI if it fails
  try {
    await messageService.syncToServer(message);
    // Mark as synced
    set((state) => ({
      messages: state.messages.map((m) =>
        m.id === message.id ? { ...m, synced: true } : m
      ),
    }));
  } catch (error) {
    // Queue for retry, but don't show error
    await messageService.queueForSync(message);
    // Message appears with "sync pending" indicator
  }
}
```

---

### Pattern 2: Retry with Exponential Backoff

Retry failed syncs with increasing delays:

```typescript
async syncWithRetry(message, attempt = 1) {
  try {
    await trpc.message.create.mutate(message);
  } catch (error) {
    if (attempt < 5) {
      const delay = Math.min(1000 * 2 ** attempt, 30000);  // Max 30s
      await new Promise((resolve) => setTimeout(resolve, delay));
      return this.syncWithRetry(message, attempt + 1);
    } else {
      // Max retries reached, queue for manual sync
      await this.queueForSync(message);
    }
  }
}
```

---

### Pattern 3: User-Facing Errors

Show clear error messages for unrecoverable errors:

```typescript
async purchaseSubscription(planId) {
  try {
    const result = await trpc.billing.purchase.mutate({ planId });
    return result;
  } catch (error) {
    if (error.code === 'PAYMENT_FAILED') {
      // User-facing error
      set({ error: 'Payment failed. Please check your card details.' });
    } else if (error.code === 'UNAUTHORIZED') {
      // User-facing error
      set({ error: 'Please log in to purchase a subscription.' });
    } else {
      // Generic error
      set({ error: 'Something went wrong. Please try again.' });
    }
    throw error;
  }
}
```

---

## Best Practices

### ✅ DO

1. **Use optimistic updates** for better UX
   ```typescript
   // Update UI immediately
   set({ messages: [...messages, newMessage] });
   // Sync in background
   await syncToServer(newMessage);
   ```

2. **Save to PGLite first, server second**
   ```typescript
   await clientDB.insert(messages).values(message);  // First
   await trpc.message.create.mutate(message);       // Second
   ```

3. **Handle offline gracefully**
   ```typescript
   try {
     await syncToServer();
   } catch (error) {
     await queueForSync();  // Retry later
   }
   ```

4. **Use selectors to prevent re-renders**
   ```typescript
   const messages = useStore((s) => s.messages);  // Good
   const store = useStore();  // Bad (re-renders on any change)
   ```

5. **Batch state updates**
   ```typescript
   set({ messages, loading: false, error: null });  // Single re-render
   ```

---

### ❌ DON'T

1. **Don't block UI on network requests**
   ```typescript
   // ❌ BAD
   await syncToServer(message);  // User waits
   set({ messages: [...messages, message] });

   // ✅ GOOD
   set({ messages: [...messages, message] });  // Instant
   syncToServer(message);  // Background
   ```

2. **Don't save to server without saving to PGLite first**
   ```typescript
   // ❌ BAD: Data lost if network fails
   await trpc.message.create.mutate(message);

   // ✅ GOOD: Data saved locally first
   await clientDB.insert(messages).values(message);
   await trpc.message.create.mutate(message);
   ```

3. **Don't ignore sync failures**
   ```typescript
   // ❌ BAD
   try {
     await syncToServer();
   } catch (error) {
     // Ignored! Data not synced!
   }

   // ✅ GOOD
   try {
     await syncToServer();
   } catch (error) {
     await queueForSync();  // Retry later
   }
   ```

4. **Don't read entire store in components**
   ```typescript
   // ❌ BAD
   const store = useStore();
   const firstMessage = store.messages[0];

   // ✅ GOOD
   const firstMessage = useStore((s) => s.messages[0]);
   ```

5. **Don't mutate state directly**
   ```typescript
   // ❌ BAD
   messages.push(newMessage);
   set({ messages });

   // ✅ GOOD
   set({ messages: [...messages, newMessage] });
   ```

---

## Summary

**LobeHub Data Flow in One Paragraph:**

> LobeHub uses an **offline-first, optimistic update** strategy. User actions immediately update the UI via Zustand stores, save to the local PGLite database, then sync to the cloud (Neon PostgreSQL) in the background via tRPC. This provides instant feedback while ensuring data is backed up and synced across devices. If offline, changes are queued and synced when the connection is restored.

**Key Patterns:**

1. 🚀 **Optimistic Updates** - Update UI first, sync later
2. 💾 **Local-First** - Save to PGLite before network requests
3. ☁️ **Background Sync** - Sync to server without blocking UI
4. 📶 **Offline Support** - Queue changes for later sync
5. 🔄 **Bidirectional Sync** - Push local changes, pull server changes

---

**Document Status:** ✅ Complete | **Last Updated:** November 20, 2025

**Next:** [Frontend Architecture](./FRONTEND_ARCHITECTURE.md) - Deep dive into React patterns and component architecture
