# Frontend Architecture - Part 1 of 2

**Documented:** November 2025
**Tech Stack Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)
**Target:** Mid-level developers proficient in JS/TS/React

**📄 This is Part 1 of 2** | **[Continue to Part 2 →](./FRONTEND-ARCHITECTURE-PART-02.md)**

This comprehensive guide explains LobeHub's frontend architecture, focusing on React 19 patterns, component organization, and modern best practices.

**Prerequisites:**
- [Architecture Overview](./ARCHITECTURE-OVERVIEW-PART-01.md) - System architecture
- [Tech Stack Guide - Part 1](./TECH-STACK-GUIDE-PART-01.md) - React 19, Next.js 16

**Time to read:** 30 minutes (Part 1)

---

## Table of Contents - Part 1

1. [Frontend Architecture Principles](#frontend-architecture-principles)
2. [React 19 Patterns](#react-19-patterns)
   - [Server Components](#server-components)
   - [Server Actions](#server-actions)
   - [New Hooks](#new-hooks)
   - [React Compiler](#react-compiler)
3. [Component Architecture](#component-architecture)
   - [Component Types](#component-types)
   - [Component Organization](#component-organization)
   - [Component Patterns](#component-patterns)
4. [File Organization](#file-organization)

**[Part 2 →](./FRONTEND-ARCHITECTURE-PART-02.md)** will cover:
- State Management Patterns
- Styling Architecture
- Data Fetching Patterns
- Common Patterns and Anti-Patterns
- Performance Optimization

---

## Frontend Architecture Principles

**LobeHub Frontend Philosophy:**

> Build a **modern, performant, maintainable** frontend using React 19, Next.js 16, and TypeScript with end-to-end type safety.

**Core Principles:**

1. **🚀 Server-First Rendering**
   - Use Server Components by default
   - Client Components only when needed (interactivity, hooks)
   - Minimize JavaScript sent to browser

2. **🎯 Component Composition**
   - Small, focused components (single responsibility)
   - Compose complex UIs from simple parts
   - Reusable across the application

3. **🔒 Type Safety**
   - TypeScript everywhere
   - Strict null checks
   - No `any` types

4. **⚡ Performance-First**
   - React Compiler handles optimization
   - Code splitting by route
   - Lazy load heavy components

5. **📦 Colocation**
   - Keep related code together
   - Components with styles and tests in same folder
   - Feature-based organization

---

## React 19 Patterns

### Server Components

**What are Server Components?**

Server Components render on the server and send HTML to the client (not JavaScript).

**Default in Next.js 16:**

```typescript
// app/page.tsx - This is a Server Component by default!
async function Page() {
  // ✅ Can use async/await
  const users = await db.query.users.findMany();

  // ✅ Can access server-only APIs
  const fs = await import('fs');

  // ✅ Can import large libraries (won't bloat client bundle)
  const heavyLib = await import('heavy-library');

  return <UserList users={users} />;
}
```

**Benefits:**

- ✅ Direct database access (no API needed)
- ✅ Async by default
- ✅ Zero JavaScript to client
- ✅ Faster initial page load
- ✅ Better SEO

**Limitations:**

- ❌ Can't use hooks (`useState`, `useEffect`, etc.)
- ❌ Can't use browser APIs (`window`, `localStorage`, etc.)
- ❌ Can't attach event handlers (`onClick`, etc.)

**🎯 Remember:** Server Components = Data fetching, Client Components = Interactivity

---

**When to Use Server Components:**

```typescript
// ✅ GOOD: Server Component for data fetching
async function UserProfile({ userId }) {
  const user = await db.query.users.findFirst({
    where: eq(users.id, userId),
  });

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

---

**When to Use Client Components:**

```typescript
// ✅ GOOD: Client Component for interactivity
'use client';

import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

**🎯 Rule:** Add `'use client'` only when you need hooks or browser APIs

---

**Composing Server and Client Components:**

```typescript
// app/page.tsx - Server Component
async function Page() {
  const messages = await db.query.messages.findMany();

  return (
    <div>
      {/* Server Component renders first */}
      <Header />

      {/* Pass data to Client Component */}
      <MessageList messages={messages} />

      {/* Server Component */}
      <Footer />
    </div>
  );
}

// components/MessageList.tsx - Client Component
'use client';

import { useState } from 'react';

function MessageList({ messages }) {
  const [filter, setFilter] = useState('all');

  const filteredMessages = messages.filter(/* ... */);

  return (
    <div>
      <select value={filter} onChange={(e) => setFilter(e.target.value)}>
        <option value="all">All</option>
        <option value="unread">Unread</option>
      </select>

      {filteredMessages.map((msg) => (
        <MessageItem key={msg.id} message={msg} />
      ))}
    </div>
  );
}
```

**🌉 Bridge from React 18:** Like `getServerSideProps`, but components can be async directly!

---

### Server Actions

**What are Server Actions?**

Server Actions are functions that run on the server but can be called from the client.

**Basic Usage:**

```typescript
// app/actions.ts
'use server';

export async function createUser(formData: FormData) {
  const name = formData.get('name') as string;
  const email = formData.get('email') as string;

  // Validate
  if (!name || !email) {
    return { error: 'Name and email required' };
  }

  // Save to database
  await db.insert(users).values({ name, email });

  // Revalidate cache
  revalidatePath('/users');

  return { success: true };
}

// app/page.tsx
import { createUser } from './actions';

function SignupForm() {
  return (
    <form action={createUser}>
      <input name="name" required />
      <input name="email" type="email" required />
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

**🎯 Benefit:** No API route needed! Type-safe end-to-end.

---

**With State Management:**

```typescript
'use client';

import { useActionState } from 'react';
import { createUser } from './actions';

function SignupForm() {
  const [state, action, isPending] = useActionState(createUser, null);

  return (
    <form action={action}>
      <input name="name" required />
      <input name="email" type="email" required />

      <button disabled={isPending}>
        {isPending ? 'Creating...' : 'Sign Up'}
      </button>

      {state?.error && (
        <div className="error">{state.error}</div>
      )}

      {state?.success && (
        <div className="success">User created!</div>
      )}
    </form>
  );
}
```

---

**When to Use Server Actions:**

**✅ Use Server Actions for:**
- Form submissions
- Data mutations (create, update, delete)
- Operations requiring server validation
- Invalidating caches

**❌ Don't use Server Actions for:**
- Read operations (use Server Components instead)
- Real-time updates (use tRPC subscriptions)
- Client-only operations

---

**Server Actions vs tRPC:**

| Feature | Server Actions | tRPC |
|---------|----------------|------|
| **Form submissions** | ✅ Best choice | ❌ Overkill |
| **Type safety** | ✅ Built-in | ✅ Built-in |
| **Real-time** | ❌ No | ✅ Subscriptions |
| **Complex queries** | ❌ Limited | ✅ Powerful |
| **File uploads** | ✅ Easy | ⚠️ Requires config |

**🎯 LobeHub uses:** Server Actions for forms, tRPC for complex data operations

---

### New Hooks

React 19 introduces new hooks for common patterns:

**1. `useActionState` - Form State Management**

```typescript
import { useActionState } from 'react';

function LoginForm() {
  const [state, action, isPending] = useActionState(loginAction, null);

  return (
    <form action={action}>
      <input name="email" type="email" required />
      <input name="password" type="password" required />

      <button disabled={isPending}>
        {isPending ? 'Logging in...' : 'Log In'}
      </button>

      {state?.error && <div>{state.error}</div>}
    </form>
  );
}
```

**🌉 Bridge from React 18:** Like `useState` + `useTransition` for forms

---

**2. `useFormStatus` - Access Form Submission State**

Use in child components to access form status:

```typescript
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();

  return (
    <button disabled={pending}>
      {pending ? 'Submitting...' : 'Submit'}
    </button>
  );
}

function MyForm() {
  return (
    <form action={myAction}>
      <input name="name" />
      <SubmitButton />  {/* Knows form status! */}
    </form>
  );
}
```

---

**3. `useOptimistic` - Optimistic Updates**

Update UI immediately, revert on error:

```typescript
import { useOptimistic } from 'react';

function MessageList({ messages }) {
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (state, newMessage) => [...state, { ...newMessage, sending: true }]
  );

  async function sendMessage(content) {
    // Add optimistic message
    addOptimisticMessage({ id: 'temp', content });

    // Send to server
    await createMessage(content);
    // When done, React auto-replaces optimistic with real
  }

  return (
    <div>
      {optimisticMessages.map((msg) => (
        <div key={msg.id} style={{ opacity: msg.sending ? 0.5 : 1 }}>
          {msg.content}
        </div>
      ))}
      <button onClick={() => sendMessage('Hello!')}>Send</button>
    </div>
  );
}
```

**🌉 Bridge from React 18:** Built-in optimistic updates (no manual state management!)

---

**4. `use` - Read Promises in Render**

```typescript
import { use } from 'react';

function UserProfile({ userPromise }) {
  const user = use(userPromise);  // Suspends until resolved

  return <div>{user.name}</div>;
}

// Usage:
function Page() {
  const userPromise = fetchUser('123');

  return (
    <Suspense fallback={<div>Loading...</div>}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}
```

**🎯 Benefit:** Read promises in render (not just in effects!)

---

### React Compiler

**What is the React Compiler?**

The React Compiler automatically optimizes React code by adding memoization where needed.

**Before (React 18):** Manual optimization required

```typescript
// ❌ React 18: Lots of manual work
const MyComponent = memo(({ user }) => {
  const fullName = useMemo(
    () => `${user.first} ${user.last}`,
    [user]
  );

  const onClick = useCallback(
    () => alert(fullName),
    [fullName]
  );

  return <button onClick={onClick}>{fullName}</button>;
});
```

**After (React 19):** Automatic optimization

```typescript
// ✅ React 19: Write simple code, compiler optimizes!
function MyComponent({ user }) {
  const fullName = `${user.first} ${user.last}`;
  const onClick = () => alert(fullName);

  return <button onClick={onClick}>{fullName}</button>;
}
// React Compiler automatically:
// - Memoizes fullName when needed
// - Memoizes onClick when needed
// - Skips re-renders when props haven't changed
```

---

**When to Still Use Memo Manually:**

```typescript
// ✅ Still useful: useMemo for expensive computations
function SearchResults({ items, query }) {
  const filteredItems = useMemo(() => {
    // Expensive operation (searching 10,000 items)
    return items.filter((item) =>
      item.name.toLowerCase().includes(query.toLowerCase())
    );
  }, [items, query]);

  return <List items={filteredItems} />;
}

// ✅ Still useful: memo for expensive renders
const ExpensiveChart = memo(({ data }) => {
  // Complex visualization that takes 100ms to render
  return <ComplexD3Chart data={data} />;
});
```

**🎯 Rule:** React Compiler handles most cases, use manual memo for expensive operations only

---

## Component Architecture

### Component Types

LobeHub categorizes components by responsibility:

**1. Page Components** (`app/[page]/page.tsx`)
- Entry point for routes
- Fetch data (Server Components)
- Compose layout and features
- Usually Server Components

```typescript
// app/chat/page.tsx
async function ChatPage() {
  const conversations = await db.query.conversations.findMany();

  return (
    <div>
      <ChatLayout>
        <ConversationList conversations={conversations} />
        <ChatWindow />
      </ChatLayout>
    </div>
  );
}
```

---

**2. Layout Components** (`app/[page]/layout.tsx`)
- Wrap pages with common UI
- Persist across navigation
- Server Components by default

```typescript
// app/chat/layout.tsx
function ChatLayout({ children }) {
  return (
    <div className="chat-layout">
      <Sidebar />
      <main>{children}</main>
    </div>
  );
}
```

---

**3. Feature Components** (`src/features/[FeatureName]/`)
- Complex, multi-component features
- Used in 2-3 places
- Can be Server or Client

```typescript
// src/features/ChatInput/index.tsx
'use client';

function ChatInput({ onSend }) {
  return (
    <div>
      <ActionBar />
      <TextArea onSend={onSend} />
      <FileUpload />
    </div>
  );
}
```

---

**4. Shared Components** (`src/components/[ComponentName]/`)
- Generic, reusable UI
- Used across many pages
- Usually Client Components (interactive)

```typescript
// src/components/Button/index.tsx
'use client';

function Button({ children, onClick, variant = 'primary' }) {
  return (
    <button
      onClick={onClick}
      className={styles[variant]}
    >
      {children}
    </button>
  );
}
```

---

**5. Page-Specific Components** (`app/[page]/components/`)
- Used in only ONE page
- Keep close to where used
- Can be Server or Client

```typescript
// app/chat/components/ChatHeader.tsx
function ChatHeader({ conversationName }) {
  return (
    <header>
      <h1>{conversationName}</h1>
      <ChatActions />
    </header>
  );
}
```

---

### Component Organization

**Decision Tree: Where to Put a Component?**

```
Is the component used in only ONE page?
├─ YES → app/[page]/components/
│
└─ NO → Is it simple and generic (Button, Input)?
    ├─ YES → src/components/
    │
    └─ NO → Is it complex with multiple sub-components?
        ├─ YES → src/features/
        │
        └─ NO → Start in src/components/, move later if needed
```

---

**File Structure Pattern:**

```
ComponentName/
├── index.tsx              ← Main component
├── style.ts               ← Styles (antd-style)
├── index.test.tsx         ← Tests (Vitest)
├── types.ts               ← TypeScript types (if complex)
└── components/            ← Sub-components (if needed)
    ├── SubComponent1.tsx
    └── SubComponent2.tsx
```

**Example:**

```
ChatInput/
├── index.tsx              ← Main ChatInput component
├── style.ts               ← Styles
├── index.test.tsx         ← Tests
└── components/
    ├── ActionBar.tsx      ← Sub-component
    ├── TextArea.tsx       ← Sub-component
    └── FileUpload.tsx     ← Sub-component
```

---

### Component Patterns

**Pattern 1: Compound Components**

Build complex components from smaller, composable parts:

```typescript
// src/components/Card/index.tsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}

function CardHeader({ children }) {
  return <div className="card-header">{children}</div>;
}

function CardBody({ children }) {
  return <div className="card-body">{children}</div>;
}

function CardFooter({ children }) {
  return <div className="card-footer">{children}</div>;
}

// Compose together:
Card.Header = CardHeader;
Card.Body = CardBody;
Card.Footer = CardFooter;

export { Card };

// Usage:
<Card>
  <Card.Header>
    <h2>Title</h2>
  </Card.Header>
  <Card.Body>
    <p>Content here</p>
  </Card.Body>
  <Card.Footer>
    <button>Action</button>
  </Card.Footer>
</Card>
```

**🌉 Bridge from React:** Like Ant Design's `Layout.Header`, `Layout.Content`, `Layout.Footer`

---

**Pattern 2: Render Props**

Pass rendering logic as a prop:

```typescript
function DataFetcher({ url, render }) {
  const { data, loading, error } = useSWR(url, fetcher);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return render(data);
}

// Usage:
<DataFetcher
  url="/api/users"
  render={(users) => (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )}
/>
```

---

**Pattern 3: Custom Hooks**

Extract logic into reusable hooks:

```typescript
// hooks/useUser.ts
function useUser(userId) {
  const { data, error, isLoading } = useSWR(
    `/api/users/${userId}`,
    fetcher
  );

  return {
    user: data,
    loading: isLoading,
    error,
  };
}

// Usage in component:
function UserProfile({ userId }) {
  const { user, loading, error } = useUser(userId);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error</div>;

  return <div>{user.name}</div>;
}
```

---

**Pattern 4: Higher-Order Components (HOCs)**

Wrap components to add functionality:

```typescript
// hoc/withAuth.tsx
function withAuth<P extends object>(Component: React.ComponentType<P>) {
  return function AuthenticatedComponent(props: P) {
    const { user, loading } = useAuth();

    if (loading) return <div>Loading...</div>;
    if (!user) return <Navigate to="/login" />;

    return <Component {...props} />;
  };
}

// Usage:
const ProtectedProfile = withAuth(UserProfile);
```

**⚠️ Note:** HOCs are less common in modern React. Prefer hooks when possible.

---

**Pattern 5: Polymorphic Components**

Components that can render as different HTML elements:

```typescript
type PolymorphicProps<E extends React.ElementType> = {
  as?: E;
  children: React.ReactNode;
} & React.ComponentPropsWithoutRef<E>;

function Text<E extends React.ElementType = 'span'>({
  as,
  children,
  ...props
}: PolymorphicProps<E>) {
  const Component = as || 'span';
  return <Component {...props}>{children}</Component>;
}

// Usage:
<Text>Default span</Text>
<Text as="p">Paragraph</Text>
<Text as="h1">Heading</Text>
<Text as="a" href="/link">Link</Text>
```

---

## File Organization

### Import Aliases

LobeHub uses path aliases for clean imports:

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"],
      "@lobechat/*": ["./packages/*/src"]
    }
  }
}

// Usage:
import { Button } from '@/components/Button';
import { useUserStore } from '@/store/user';
import { clientDB } from '@lobechat/database/client';
```

**🎯 Benefit:** No more `../../../components/Button`!

---

### Import Order

Organize imports for readability:

```typescript
// 1. React and Next.js
import { useState } from 'react';
import Image from 'next/image';

// 2. Third-party libraries
import { Button } from 'antd';
import { useTranslation } from 'react-i18next';

// 3. Internal packages
import { clientDB } from '@lobechat/database/client';
import type { User } from '@lobechat/types/user';

// 4. Internal modules
import { useUserStore } from '@/store/user';
import { Avatar } from '@/components/Avatar';

// 5. Relative imports
import { ChatHeader } from './components/ChatHeader';
import { useStyles } from './style';
```

---

### Barrel Exports

Use `index.ts` for clean exports:

```typescript
// components/index.ts
export { Button } from './Button';
export { Input } from './Input';
export { Modal } from './Modal';

// Usage:
import { Button, Input, Modal } from '@/components';
```

**⚠️ Caution:** Can cause circular dependencies. Use sparingly.

---

## Summary - Part 1

**Frontend Architecture in One Paragraph:**

> LobeHub uses React 19 Server Components by default for data fetching, with Client Components only for interactivity. Components are organized by reusability (page-specific → features → shared), colocated with styles and tests. The React Compiler handles most optimizations automatically, reducing boilerplate.

**Key Patterns:**

1. 🖥️ **Server Components** - Default for data fetching
2. 💻 **Client Components** - Only for interactivity (`'use client'`)
3. 🎯 **Server Actions** - Type-safe form submissions
4. 🪝 **New Hooks** - `useActionState`, `useFormStatus`, `useOptimistic`
5. ⚡ **React Compiler** - Automatic memoization
6. 📦 **Component Organization** - Colocation and composition

---

**Document Status:** ✅ Part 1 Complete | **Last Updated:** November 20, 2025

**[Continue to Part 2 →](./FRONTEND-ARCHITECTURE-PART-02.md)** - State Management, Styling, Data Fetching, Common Patterns
