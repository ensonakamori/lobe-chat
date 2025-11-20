# Frontend Architecture - Part 2 of 2

**Documented:** November 2025
**Tech Stack Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)
**Target:** Mid-level developers proficient in JS/TS/React

**📄 This is Part 2 of 2** | **[← Back to Part 1](./FRONTEND-ARCHITECTURE-PART-01.md)**

This document continues the frontend architecture guide, covering state management, styling, data fetching, and common patterns.

**Prerequisites:**
- **[Part 1](./FRONTEND-ARCHITECTURE-PART-01.md)** - Read this first!

**Time to read:** 30 minutes (Part 2)

---

## Table of Contents - Part 2

1. [State Management Patterns](#state-management-patterns)
2. [Styling Architecture](#styling-architecture)
3. [Data Fetching Patterns](#data-fetching-patterns)
4. [Common Patterns](#common-patterns)
5. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)
6. [Performance Optimization](#performance-optimization)
7. [Summary](#summary)

**[← Part 1](./FRONTEND-ARCHITECTURE-PART-01.md)** covered:
- Frontend Architecture Principles
- React 19 Patterns (Server Components, Server Actions, New Hooks)
- Component Architecture (Types, Organization, Patterns)
- File Organization

---

## State Management Patterns

LobeHub uses different state management solutions for different use cases:

| State Type | Solution | Example |
|------------|----------|---------|
| **Global app state** | Zustand | User data, chat messages, UI state |
| **Server data** | SWR / TanStack Query | API responses, cached data |
| **URL state** | nuqs | Filters, pagination, search queries |
| **Local component state** | `useState` | Form inputs, UI toggles |

---

### Zustand: Global State Management

**Pattern: Slice Organization**

Organize stores by domain, then slice by feature:

```
src/store/
├── user/
│   ├── index.ts          ← Public exports
│   ├── store.ts          ← Store creation
│   └── slices/
│       ├── auth/
│       │   ├── action.ts       ← Auth actions
│       │   ├── selectors.ts    ← Auth selectors
│       │   └── initialState.ts ← Initial state
│       └── settings/
│           ├── action.ts
│           └── initialState.ts
```

---

**Creating a Store:**

```typescript
// store/user/slices/auth/action.ts
export const createAuthSlice = (set, get) => ({
  user: null,
  loading: false,
  error: null,

  login: async (email, password) => {
    set({ loading: true, error: null });

    try {
      const user = await trpc.auth.login.mutate({ email, password });
      set({ user, loading: false });
    } catch (error) {
      set({ error: error.message, loading: false });
    }
  },

  logout: async () => {
    await trpc.auth.logout.mutate();
    set({ user: null });
  },
});

// store/user/store.ts
import { create } from 'zustand';
import { createAuthSlice } from './slices/auth/action';
import { createSettingsSlice } from './slices/settings/action';

export const useUserStore = create((set, get) => ({
  ...createAuthSlice(set, get),
  ...createSettingsSlice(set, get),
}));

// store/user/index.ts
export { useUserStore } from './store';
```

---

**Using the Store:**

```typescript
'use client';

import { useUserStore } from '@/store/user';

function LoginButton() {
  // ✅ GOOD: Selective subscription
  const login = useUserStore((s) => s.login);
  const loading = useUserStore((s) => s.loading);

  return (
    <button onClick={() => login('user@example.com', 'password')}>
      {loading ? 'Logging in...' : 'Log In'}
    </button>
  );
}

// ❌ BAD: Subscribes to entire store
function BadLoginButton() {
  const store = useUserStore();  // Re-renders on ANY store change!
  return <button onClick={() => store.login(/* ... */)}>Log In</button>;
}
```

**🎯 Rule:** Always use selectors to prevent unnecessary re-renders!

---

**Selectors for Derived State:**

```typescript
// store/user/slices/auth/selectors.ts
export const selectIsAuthenticated = (state) => state.user !== null;
export const selectUserName = (state) => state.user?.name ?? 'Guest';

// Usage:
import { useUserStore } from '@/store/user';
import { selectIsAuthenticated, selectUserName } from '@/store/user/slices/auth/selectors';

function Header() {
  const isAuthenticated = useUserStore(selectIsAuthenticated);
  const userName = useUserStore(selectUserName);

  return (
    <header>
      {isAuthenticated ? `Welcome, ${userName}` : 'Please log in'}
    </header>
  );
}
```

---

**Persisting State:**

```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

export const useSettingsStore = create(
  persist(
    (set) => ({
      theme: 'light',
      language: 'en',
      setTheme: (theme) => set({ theme }),
      setLanguage: (language) => set({ language }),
    }),
    {
      name: 'settings-storage',  // localStorage key
    }
  )
);
```

**🎯 Benefit:** State persists across page reloads!

---

### SWR: Server Data Management

**Pattern: Custom Hooks for API Calls**

```typescript
// services/user/hooks/useUser.ts
import useSWR from 'swr';
import { trpc } from '@/utils/trpc';

export function useUser(userId: string) {
  const { data, error, isLoading, mutate } = useSWR(
    userId ? ['user', userId] : null,  // Key (null = don't fetch)
    () => trpc.user.getUser.query({ id: userId })
  );

  return {
    user: data,
    loading: isLoading,
    error,
    refresh: mutate,  // Manual refresh
  };
}

// Usage in component:
function UserProfile({ userId }) {
  const { user, loading, error } = useUser(userId);

  if (loading) return <Skeleton />;
  if (error) return <Error message={error.message} />;

  return <div>{user.name}</div>;
}
```

---

**Mutations with SWR:**

```typescript
import { mutate } from 'swr';
import { trpc } from '@/utils/trpc';

async function updateUser(userId: string, data: Partial<User>) {
  // Optimistic update
  mutate(['user', userId], { ...currentUser, ...data }, false);

  // Send request
  await trpc.user.updateUser.mutate({ id: userId, ...data });

  // Revalidate
  mutate(['user', userId]);
}
```

---

### URL State with nuqs

**Pattern: Type-Safe Query Params**

```typescript
'use client';

import { useQueryState, parseAsInteger, parseAsString } from 'nuqs';

function UserList() {
  const [page, setPage] = useQueryState(
    'page',
    parseAsInteger.withDefault(1)
  );
  const [search, setSearch] = useQueryState(
    'search',
    parseAsString.withDefault('')
  );

  // URL: /users?page=2&search=alice

  return (
    <div>
      <input
        value={search}
        onChange={(e) => setSearch(e.target.value)}
        placeholder="Search..."
      />

      <UserGrid page={page} search={search} />

      <Pagination
        current={page}
        onChange={setPage}
      />
    </div>
  );
}
```

**🎯 Benefit:** URL is source of truth, works with browser back/forward!

---

## Styling Architecture

LobeHub uses **antd-style** for CSS-in-JS with theme tokens.

### Basic Styling Pattern

```typescript
import { createStyles } from 'antd-style';

const useStyles = createStyles(({ token, css }) => ({
  container: css`
    padding: ${token.paddingLG}px;
    background: ${token.colorBgContainer};
    border-radius: ${token.borderRadiusLG}px;
  `,

  title: css`
    font-size: ${token.fontSizeLG}px;
    color: ${token.colorTextHeading};
    margin-bottom: ${token.marginMD}px;
  `,

  button: css`
    background: ${token.colorPrimary};
    color: ${token.colorTextLightSolid};

    &:hover {
      background: ${token.colorPrimaryHover};
    }
  `,
}));

function MyComponent() {
  const { styles } = useStyles();

  return (
    <div className={styles.container}>
      <h1 className={styles.title}>Hello</h1>
      <button className={styles.button}>Click</button>
    </div>
  );
}
```

---

### Theme Tokens

**Common tokens:**

```typescript
// Colors
token.colorPrimary        // Primary brand color
token.colorSuccess        // Success color (green)
token.colorWarning        // Warning color (yellow)
token.colorError          // Error color (red)
token.colorText           // Default text color
token.colorTextSecondary  // Secondary text color
token.colorBgContainer    // Container background

// Spacing
token.paddingXS           // Extra small padding (4px)
token.paddingSM           // Small padding (8px)
token.paddingMD           // Medium padding (12px)
token.paddingLG           // Large padding (16px)
token.paddingXL           // Extra large padding (24px)

// Typography
token.fontSize            // Base font size (14px)
token.fontSizeLG          // Large font size (16px)
token.lineHeight          // Line height (1.5)

// Border radius
token.borderRadius        // Default border radius (6px)
token.borderRadiusLG      // Large border radius (8px)
```

**🎯 Benefit:** Automatic dark mode support! Tokens change based on theme.

---

### Responsive Styling

```typescript
const useStyles = createStyles(({ token, responsive }) => ({
  container: {
    padding: token.paddingLG,

    // Mobile
    [responsive.mobile]: {
      padding: token.paddingSM,
    },

    // Tablet
    [responsive.tablet]: {
      padding: token.paddingMD,
    },

    // Desktop
    [responsive.desktop]: {
      padding: token.paddingXL,
    },
  },
}));
```

---

### Component Variants

```typescript
const useStyles = createStyles(({ token }) => ({
  button: (variant: 'primary' | 'secondary' | 'danger') => ({
    padding: `${token.paddingSM}px ${token.paddingMD}px`,
    borderRadius: token.borderRadius,
    border: 'none',
    cursor: 'pointer',

    ...(variant === 'primary' && {
      background: token.colorPrimary,
      color: token.colorTextLightSolid,
    }),

    ...(variant === 'secondary' && {
      background: token.colorBgContainer,
      color: token.colorText,
      border: `1px solid ${token.colorBorder}`,
    }),

    ...(variant === 'danger' && {
      background: token.colorError,
      color: token.colorTextLightSolid,
    }),
  }),
}));

function Button({ variant = 'primary', children }) {
  const { styles } = useStyles();

  return (
    <button className={styles.button(variant)}>
      {children}
    </button>
  );
}
```

---

## Data Fetching Patterns

### Pattern 1: Server Component Data Fetching

Fetch data directly in Server Components:

```typescript
// app/users/page.tsx
async function UsersPage() {
  // Direct database access!
  const users = await serverDB.query.users.findMany();

  return <UserList users={users} />;
}
```

**✅ Use when:** Initial page load, SEO needed

---

### Pattern 2: Client Component with SWR

Fetch data on the client with caching:

```typescript
'use client';

import useSWR from 'swr';

function UserList() {
  const { data: users, error } = useSWR('/api/users', fetcher);

  if (!users) return <Skeleton />;
  if (error) return <Error />;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

**✅ Use when:** Client-side filtering/sorting, real-time updates

---

### Pattern 3: Server Component + Client Hydration

Fetch on server, enhance on client:

```typescript
// app/users/page.tsx (Server Component)
async function UsersPage() {
  const initialUsers = await serverDB.query.users.findMany();

  return <UserList initialUsers={initialUsers} />;
}

// components/UserList.tsx (Client Component)
'use client';

import useSWR from 'swr';

function UserList({ initialUsers }) {
  const { data: users = initialUsers } = useSWR('/api/users', fetcher, {
    fallbackData: initialUsers,  // Use server data initially
    revalidateOnMount: false,    // Don't refetch immediately
  });

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

**✅ Use when:** Best of both (fast initial load + client updates)

---

### Pattern 4: tRPC Queries

Type-safe API calls with TanStack Query:

```typescript
'use client';

import { trpc } from '@/utils/trpc';

function UserProfile({ userId }) {
  const { data: user, isLoading } = trpc.user.getUser.useQuery({ id: userId });
  //    ^? User | undefined (fully typed!)

  if (isLoading) return <Skeleton />;

  return <div>{user.name}</div>;
}
```

**✅ Use when:** Type safety required, complex queries

---

### Pattern 5: Infinite Loading

Load more data as user scrolls:

```typescript
'use client';

import useSWRInfinite from 'swr/infinite';

function MessageList() {
  const { data, size, setSize } = useSWRInfinite(
    (index) => `/api/messages?page=${index + 1}`,
    fetcher
  );

  const messages = data ? data.flat() : [];
  const isLoadingMore = data && typeof data[size - 1] === 'undefined';

  return (
    <div>
      {messages.map((msg) => (
        <MessageItem key={msg.id} message={msg} />
      ))}

      <button
        onClick={() => setSize(size + 1)}
        disabled={isLoadingMore}
      >
        {isLoadingMore ? 'Loading...' : 'Load More'}
      </button>
    </div>
  );
}
```

---

## Common Patterns

### Pattern: Conditional Rendering

```typescript
// ✅ GOOD: Early returns
function UserProfile({ user }) {
  if (!user) return <div>No user found</div>;
  if (user.deleted) return <div>User deleted</div>;

  return <div>{user.name}</div>;
}

// ❌ BAD: Nested ternaries
function UserProfile({ user }) {
  return user ? (
    user.deleted ? (
      <div>User deleted</div>
    ) : (
      <div>{user.name}</div>
    )
  ) : (
    <div>No user found</div>
  );
}
```

---

### Pattern: List Rendering

```typescript
// ✅ GOOD: Stable keys
function UserList({ users }) {
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

// ❌ BAD: Index as key (causes bugs with reordering)
function UserList({ users }) {
  return (
    <ul>
      {users.map((user, index) => (
        <li key={index}>{user.name}</li>
      ))}
    </ul>
  );
}
```

---

### Pattern: Error Boundaries

```typescript
// components/ErrorBoundary.tsx
'use client';

import { Component } from 'react';

class ErrorBoundary extends Component {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, info) {
    console.error('Error caught:', error, info);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h1>Something went wrong</h1>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage:
<ErrorBoundary>
  <MyComponent />
</ErrorBoundary>
```

---

### Pattern: Loading States

```typescript
function UserProfile({ userId }) {
  const { user, loading, error } = useUser(userId);

  // Loading skeleton
  if (loading) {
    return (
      <div>
        <Skeleton width={200} height={20} />
        <Skeleton width={300} height={16} />
      </div>
    );
  }

  // Error state
  if (error) {
    return (
      <div className="error">
        <p>Failed to load user</p>
        <button onClick={() => refetch()}>Retry</button>
      </div>
    );
  }

  // Success state
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

---

### Pattern: Form Handling

```typescript
'use client';

import { useState } from 'react';
import { useActionState } from 'react';
import { createUser } from './actions';

function UserForm() {
  const [state, action, isPending] = useActionState(createUser, null);

  return (
    <form action={action}>
      <div>
        <label htmlFor="name">Name</label>
        <input
          id="name"
          name="name"
          required
          aria-invalid={state?.errors?.name ? 'true' : 'false'}
        />
        {state?.errors?.name && (
          <span className="error">{state.errors.name}</span>
        )}
      </div>

      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          name="email"
          type="email"
          required
          aria-invalid={state?.errors?.email ? 'true' : 'false'}
        />
        {state?.errors?.email && (
          <span className="error">{state.errors.email}</span>
        )}
      </div>

      <button disabled={isPending}>
        {isPending ? 'Creating...' : 'Create User'}
      </button>

      {state?.success && (
        <div className="success">User created successfully!</div>
      )}
    </form>
  );
}
```

---

## Anti-Patterns to Avoid

### ❌ Anti-Pattern 1: Prop Drilling

**Problem:** Passing props through many layers

```typescript
// ❌ BAD
function App() {
  const user = useUser();
  return <Layout user={user} />;
}

function Layout({ user }) {
  return <Header user={user} />;
}

function Header({ user }) {
  return <UserMenu user={user} />;
}

function UserMenu({ user }) {
  return <div>{user.name}</div>;
}
```

**Solution:** Use Zustand or Context

```typescript
// ✅ GOOD
function App() {
  return <Layout />;
}

function UserMenu() {
  const user = useUserStore((s) => s.user);
  return <div>{user.name}</div>;
}
```

---

### ❌ Anti-Pattern 2: Massive Components

**Problem:** Components with too many responsibilities

```typescript
// ❌ BAD: 500+ line component
function ChatPage() {
  // 50 lines of state
  // 100 lines of event handlers
  // 200 lines of JSX
  // Everything in one component!
}
```

**Solution:** Break into smaller components

```typescript
// ✅ GOOD
function ChatPage() {
  return (
    <div>
      <ChatHeader />
      <ChatMessages />
      <ChatInput />
    </div>
  );
}
```

---

### ❌ Anti-Pattern 3: Direct State Mutation

**Problem:** Mutating state objects/arrays directly

```typescript
// ❌ BAD
function addMessage(newMessage) {
  messages.push(newMessage);  // Mutates array!
  set({ messages });          // Won't trigger re-render!
}
```

**Solution:** Create new objects/arrays

```typescript
// ✅ GOOD
function addMessage(newMessage) {
  set({ messages: [...messages, newMessage] });
}
```

---

### ❌ Anti-Pattern 4: useEffect for Everything

**Problem:** Overusing `useEffect`

```typescript
// ❌ BAD
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then((r) => r.json())
      .then(setUser);
  }, [userId]);

  return <div>{user?.name}</div>;
}
```

**Solution:** Use data fetching libraries

```typescript
// ✅ GOOD
function UserProfile({ userId }) {
  const { user } = useUser(userId);
  return <div>{user?.name}</div>;
}
```

---

### ❌ Anti-Pattern 5: Inline Functions in JSX

**Problem:** Creating new functions on every render

```typescript
// ❌ BAD
function UserList({ users }) {
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id} onClick={() => handleClick(user.id)}>
          {user.name}
        </li>
      ))}
    </ul>
  );
}
```

**Solution:** React Compiler handles this! Or use `useCallback` if needed

```typescript
// ✅ GOOD (React 19 Compiler handles it)
function UserList({ users }) {
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id} onClick={() => handleClick(user.id)}>
          {user.name}
        </li>
      ))}
    </ul>
  );
}

// Or manually with useCallback for expensive operations:
import { useCallback } from 'react';

function UserList({ users }) {
  const handleClick = useCallback((id) => {
    // Expensive operation
  }, []);

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id} onClick={() => handleClick(user.id)}>
          {user.name}
        </li>
      ))}
    </ul>
  );
}
```

---

## Performance Optimization

### Optimization 1: Code Splitting

Split large components into separate bundles:

```typescript
import dynamic from 'next/dynamic';

// Lazy load heavy component
const HeavyChart = dynamic(() => import('./HeavyChart'), {
  loading: () => <Skeleton />,
  ssr: false,  // Don't render on server
});

function Dashboard() {
  return (
    <div>
      <Header />
      <HeavyChart />  {/* Only loads when needed */}
    </div>
  );
}
```

---

### Optimization 2: Image Optimization

Use Next.js Image component:

```typescript
import Image from 'next/image';

// ✅ GOOD: Automatic optimization
<Image
  src="/photo.jpg"
  width={500}
  height={300}
  alt="Photo"
  priority  // For above-the-fold images
/>

// ❌ BAD: No optimization
<img src="/photo.jpg" alt="Photo" />
```

---

### Optimization 3: Suspense Boundaries

Stream content as it's ready:

```typescript
import { Suspense } from 'react';

async function Page() {
  return (
    <div>
      {/* Header renders immediately */}
      <Header />

      {/* Messages stream in when ready */}
      <Suspense fallback={<Skeleton />}>
        <Messages />
      </Suspense>

      {/* Sidebar streams in when ready */}
      <Suspense fallback={<Skeleton />}>
        <Sidebar />
      </Suspense>
    </div>
  );
}

async function Messages() {
  const messages = await db.query.messages.findMany();
  return <MessageList messages={messages} />;
}
```

---

### Optimization 4: Virtualized Lists

For long lists, only render visible items:

```typescript
import { FixedSizeList } from 'react-window';

function VirtualizedMessageList({ messages }) {
  return (
    <FixedSizeList
      height={600}
      itemCount={messages.length}
      itemSize={50}
      width="100%"
    >
      {({ index, style }) => (
        <div style={style}>
          {messages[index].content}
        </div>
      )}
    </FixedSizeList>
  );
}
```

**🎯 Benefit:** Render 10,000 messages without lag!

---

## Summary

**Frontend Architecture in One Paragraph:**

> LobeHub's frontend uses React 19 Server Components for data fetching, Client Components for interactivity, Zustand for global state, SWR/TanStack Query for server data, and antd-style for theming. Components are organized by reusability, colocated with styles and tests. The React Compiler handles most optimizations automatically.

**Key Patterns:**

1. 🎯 **State Management** - Zustand (global), SWR (server data), nuqs (URL state)
2. 🎨 **Styling** - antd-style with theme tokens for automatic dark mode
3. 📡 **Data Fetching** - Server Components (initial), SWR (client), tRPC (type-safe)
4. ✅ **Best Practices** - Early returns, stable keys, error boundaries
5. ❌ **Avoid** - Prop drilling, massive components, direct mutation, useEffect overuse
6. ⚡ **Performance** - Code splitting, image optimization, Suspense, virtualized lists

**Technologies Summary:**

| Technology | Purpose | When to Use |
|------------|---------|-------------|
| **Server Components** | Data fetching | Initial page load, SEO |
| **Client Components** | Interactivity | Hooks, browser APIs |
| **Server Actions** | Form submissions | Creating/updating data |
| **Zustand** | Global state | App-wide state |
| **SWR** | Server data | Client-side data fetching |
| **nuqs** | URL state | Filters, pagination |
| **antd-style** | Styling | Theme-aware CSS-in-JS |
| **tRPC** | Type-safe APIs | Complex queries |

---

**Document Status:** ✅ Complete (Part 2 of 2) | **Last Updated:** November 20, 2025

**[← Part 1](./FRONTEND-ARCHITECTURE-PART-01.md)** | **Next:** [Backend Architecture](./BACKEND_ARCHITECTURE.md)
