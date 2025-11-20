# Tech Stack Guide - Part 2 of 4

**Documented:** November 2025
**Tech Stack Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)
**Target:** Mid-level developers proficient in JS/TS/React

**📄 This is Part 2 of 4** | **[← Part 1](./TECH-STACK-GUIDE-PART-01.md)** | **[Part 3 →](./TECH-STACK-GUIDE-PART-03.md)**

This guide continues the technology stack overview, covering state management, data fetching, internationalization, and utilities.

**Prerequisites:**
- **[Part 1](./TECH-STACK-GUIDE-PART-01.md)** - Frontend core technologies

**Time to read:** 30 minutes (Part 2)

---

## Table of Contents - Part 2

1. [State Management](#state-management)
   - [Zustand 5.0.4](#zustand-504)
   - [nuqs 2.2.5](#nuqs-225)
2. [Data Fetching](#data-fetching)
   - [SWR 2.3.0](#swr-230)
   - [TanStack Query 5.76.5](#tanstack-query-5765)
3. [Internationalization](#internationalization)
   - [react-i18next 15.1.3](#react-i18next-1513)
4. [React Utilities](#react-utilities)
   - [aHooks 3.8.2](#ahooks-382)
5. [JavaScript Utilities](#javascript-utilities)
   - [dayjs 1.11.13](#dayjs-11113)
   - [lodash-es 4.17.21](#lodash-es-41721)

**[← Part 1](./TECH-STACK-GUIDE-PART-01.md)** covered:
- Frontend Core (React 19, Next.js 16, TypeScript)
- UI & Styling (@lobehub/ui, Ant Design, antd-style)
- Icons & Layout

**[Part 3 →](./TECH-STACK-GUIDE-PART-03.md)** will cover:
- Backend (tRPC, Next.js API Routes)
- Database (Drizzle ORM, PGLite, Neon PostgreSQL)

**[Part 4 →](./TECH-STACK-GUIDE-PART-04.md)** will cover:
- Testing (Vitest, Playwright)
- Build Tools (Turbopack, Bun)
- Infrastructure & Deployment

---

## State Management

### Zustand 5.0.4

**✅ CURRENT** | **Released:** November 2024 | **React 19 Compatible:** Yes

**What is it?**
Zustand is a small, fast state management library for React using hooks.

**Why Zustand?**

**Alternatives considered:**
- Redux: Powerful, but too much boilerplate
- MobX: Good, but magic can be confusing
- Jotai/Recoil: Good, but atomic model has learning curve
- Context API: Simple, but performance issues

**Zustand wins:**
- ✅ Simple API (minimal boilerplate)
- ✅ Great performance (no unnecessary re-renders)
- ✅ TypeScript-first
- ✅ Middleware support (persist, devtools, immer)
- ✅ No providers needed!

---

**Basic Usage:**

```typescript
// stores/user/store.ts
import { create } from 'zustand';

interface UserState {
  user: User | null;
  login: (user: User) => void;
  logout: () => void;
}

export const useUserStore = create<UserState>((set) => ({
  user: null,
  login: (user) => set({ user }),
  logout: () => set({ user: null }),
}));

// In component:
function Profile() {
  const user = useUserStore((s) => s.user);
  const logout = useUserStore((s) => s.logout);

  return (
    <div>
      <p>Welcome, {user?.name}</p>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

**🌉 Bridge from Redux:** Like Redux, but without actions, reducers, or providers!

---

**Advanced Patterns:**

**1. Slice Pattern (Recommended)**

For large stores, organize by feature slices:

```typescript
// stores/chat/slices/message/action.ts
export const createMessageSlice = (set, get) => ({
  messages: [],
  addMessage: (message) =>
    set((state) => ({
      messages: [...state.messages, message],
    })),
  deleteMessage: (id) =>
    set((state) => ({
      messages: state.messages.filter((m) => m.id !== id),
    })),
});

// stores/chat/slices/conversation/action.ts
export const createConversationSlice = (set, get) => ({
  conversations: [],
  createConversation: (conv) =>
    set((state) => ({
      conversations: [...state.conversations, conv],
    })),
});

// stores/chat/store.ts
import { create } from 'zustand';

export const useChatStore = create((set, get) => ({
  ...createMessageSlice(set, get),
  ...createConversationSlice(set, get),
}));
```

**🎯 Benefit:** Organized, maintainable large stores

📚 **More details:** [.cursor/rules/zustand-slice-organization.mdc](./.cursor/rules/zustand-slice-organization.mdc)

---

**2. Persist Middleware**

Save state to localStorage:

```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

export const useSettingsStore = create(
  persist(
    (set) => ({
      theme: 'light',
      setTheme: (theme) => set({ theme }),
    }),
    {
      name: 'settings-storage', // localStorage key
    }
  )
);
```

---

**3. Immer Middleware**

Simplify complex state updates:

```typescript
import { create } from 'zustand';
import { immer } from 'zustand/middleware/immer';

export const useUserStore = create(
  immer((set) => ({
    user: { name: 'Alice', age: 25 },
    updateUser: (updates) =>
      set((state) => {
        // Mutate state directly with Immer!
        Object.assign(state.user, updates);
      }),
  }))
);
```

---

**4. Selectors (Performance)**

Use selectors to prevent unnecessary re-renders:

```typescript
// ❌ BAD: Re-renders on any store change
function Profile() {
  const store = useUserStore();
  return <div>{store.user?.name}</div>;
}

// ✅ GOOD: Only re-renders when user changes
function Profile() {
  const user = useUserStore((s) => s.user);
  return <div>{user?.name}</div>;
}

// ✅ BETTER: Use useShallow for objects/arrays
import { useShallow } from 'zustand/react/shallow';

function Profile() {
  const { user, settings } = useUserStore(
    useShallow((s) => ({ user: s.user, settings: s.settings }))
  );
  return <div>{user?.name}</div>;
}
```

**🆕 NEW IN 2025:** `useShallow` replaces the old `shallow` function

---

**Learning Resources:**

- [Zustand Documentation](https://docs.pmnd.rs/zustand/getting-started/introduction)
- [Zustand 5.0 Release Notes](https://github.com/pmndrs/zustand/releases/tag/v5.0.0)

**🚨 Migration from Zustand 4:**
- Replace `shallow` with `useShallow` from `zustand/react/shallow`
- `create` now requires explicit typing for TypeScript
- React 18+ required (Zustand 5 uses `useSyncExternalStore`)

📚 **More details:** [.cursor/rules/zustand-action-patterns.mdc](./.cursor/rules/zustand-action-patterns.mdc)

---

### nuqs 2.2.5

**✅ CURRENT** | **Released:** November 2024 | **Next.js 16 Compatible:** Yes

**What is it?**
`nuqs` (Next.js URL Query State) provides type-safe URL search params management for Next.js.

**Why nuqs?**

**Problem with manual URL params:**
```typescript
// ❌ Manual URL param management
const searchParams = useSearchParams();
const page = searchParams.get('page');          // string | null (unsafe!)
const perPage = searchParams.get('perPage');    // string | null

// Need to manually parse and validate
const pageNum = page ? parseInt(page) : 1;
const perPageNum = perPage ? parseInt(perPage) : 10;
```

**Solution with nuqs:**
```typescript
import { useQueryState, parseAsInteger } from 'nuqs';

const [page, setPage] = useQueryState('page', parseAsInteger.withDefault(1));
const [perPage, setPerPage] = useQueryState('perPage', parseAsInteger.withDefault(10));
//    ^? number (type-safe!)

// Update URL:
setPage(2); // Updates URL to ?page=2
```

---

**Key Features:**

**1. Type-Safe Parsers**

```typescript
import {
  parseAsInteger,
  parseAsFloat,
  parseAsString,
  parseAsBoolean,
  parseAsArrayOf,
  parseAsStringEnum,
} from 'nuqs';

// Integer
const [page] = useQueryState('page', parseAsInteger);
//    ^? number | null

// String
const [search] = useQueryState('search', parseAsString);
//    ^? string | null

// Boolean
const [darkMode] = useQueryState('dark', parseAsBoolean);
//    ^? boolean | null

// Array
const [tags] = useQueryState('tags', parseAsArrayOf(parseAsString));
//    ^? string[] | null

// Enum
const [sort] = useQueryState(
  'sort',
  parseAsStringEnum(['name', 'date', 'popularity'])
);
//    ^? 'name' | 'date' | 'popularity' | null
```

---

**2. Default Values**

```typescript
const [page, setPage] = useQueryState(
  'page',
  parseAsInteger.withDefault(1) // Default to 1
);
//    ^? number (never null!)
```

---

**3. Multiple Query States**

```typescript
import { useQueryStates, parseAsInteger, parseAsString } from 'nuqs';

const [params, setParams] = useQueryStates({
  page: parseAsInteger.withDefault(1),
  perPage: parseAsInteger.withDefault(10),
  search: parseAsString.withDefault(''),
});

// Update multiple at once:
setParams({ page: 2, search: 'hello' });
// URL: ?page=2&perPage=10&search=hello
```

---

**4. Server Components**

```typescript
import { createSearchParamsCache, parseAsInteger } from 'nuqs/server';

// Server Component
const searchParamsCache = createSearchParamsCache({
  page: parseAsInteger.withDefault(1),
});

async function Page({ searchParams }) {
  const { page } = searchParamsCache.parse(searchParams);
  //      ^? number

  const users = await db.query.users.findMany({
    limit: 10,
    offset: (page - 1) * 10,
  });

  return <UserList users={users} />;
}
```

**🌉 Bridge from React Router:** Like `useSearchParams`, but type-safe!

---

**Learning Resources:**

- [nuqs Documentation](https://nuqs.47ng.com/)
- [nuqs with Next.js 16](https://nuqs.47ng.com/next)

📚 **More details:** [PATTERNS_AND_CONVENTIONS.md](./PATTERNS_AND_CONVENTIONS.md)

---

## Data Fetching

### SWR 2.3.0

**✅ CURRENT** | **Released:** November 2024 | **React 19 Compatible:** Yes

**What is it?**
SWR (stale-while-revalidate) is a React hooks library for data fetching with caching and revalidation.

**Why SWR?**

**Problem with manual fetching:**
```typescript
// ❌ Manual data fetching (lots of boilerplate)
function UserProfile() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch('/api/user')
      .then(r => r.json())
      .then(setUser)
      .catch(setError)
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{user.name}</div>;
}
```

**Solution with SWR:**
```typescript
import useSWR from 'swr';

function UserProfile() {
  const { data, error, isLoading } = useSWR('/api/user', fetcher);

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{data.name}</div>;
}

// Fetcher (reusable)
const fetcher = (url) => fetch(url).then(r => r.json());
```

**🌉 Bridge from useEffect:** Like `useEffect` + `fetch`, but with caching, revalidation, and less code!

---

**Key Features:**

**1. Automatic Revalidation**

SWR automatically refetches data when:
- User refocuses the window
- User reconnects to the internet
- Component remounts

```typescript
const { data } = useSWR('/api/user', fetcher, {
  refreshInterval: 1000,      // Poll every 1 second
  revalidateOnFocus: true,    // Refetch on window focus
  revalidateOnReconnect: true, // Refetch on reconnect
});
```

---

**2. Mutation**

Optimistic updates and revalidation:

```typescript
import { mutate } from 'swr';

async function updateUser(newData) {
  // Optimistic update
  mutate('/api/user', newData, false);

  // Send request
  await fetch('/api/user', {
    method: 'PUT',
    body: JSON.stringify(newData),
  });

  // Revalidate
  mutate('/api/user');
}
```

---

**3. Pagination**

```typescript
function UserList() {
  const [page, setPage] = useState(1);
  const { data, error } = useSWR(`/api/users?page=${page}`, fetcher);

  return (
    <div>
      {data.users.map(user => <div key={user.id}>{user.name}</div>)}
      <button onClick={() => setPage(page + 1)}>Next</button>
    </div>
  );
}
```

---

**4. Infinite Loading**

```typescript
import useSWRInfinite from 'swr/infinite';

function UserList() {
  const { data, size, setSize } = useSWRInfinite(
    (index) => `/api/users?page=${index + 1}`,
    fetcher
  );

  const users = data ? data.flat() : [];

  return (
    <div>
      {users.map(user => <div key={user.id}>{user.name}</div>)}
      <button onClick={() => setSize(size + 1)}>Load More</button>
    </div>
  );
}
```

---

**When to use SWR vs TanStack Query?**

| Feature | SWR | TanStack Query |
|---------|-----|----------------|
| **Bundle size** | ~5KB | ~13KB |
| **API** | Simple | Feature-rich |
| **Mutation helpers** | Basic | Advanced |
| **DevTools** | No | Yes |
| **Use case** | Simple data fetching | Complex data management |

**🎯 LobeHub uses:** SWR for simple data fetching, TanStack Query with tRPC

---

**Learning Resources:**

- [SWR Documentation](https://swr.vercel.app/)
- [SWR Examples](https://swr.vercel.app/examples)

📚 **More details:** [DATA_FLOW_GUIDE.md](./DATA_FLOW_GUIDE.md)

---

### TanStack Query 5.76.5

**✅ CURRENT** | **Released:** November 2024 | **React 19 Compatible:** Yes

**What is it?**
TanStack Query (formerly React Query) is a powerful data fetching and state management library.

**Why TanStack Query?**

Compared to SWR, TanStack Query provides:
- ✅ More powerful mutation handling
- ✅ Built-in DevTools
- ✅ Better infinite query support
- ✅ Query cancellation
- ✅ Advanced caching strategies

---

**Basic Usage:**

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

function UserProfile() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['user'],
    queryFn: () => fetch('/api/user').then(r => r.json()),
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{data.name}</div>;
}
```

---

**Mutations:**

```typescript
function UpdateProfile() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (newUser) =>
      fetch('/api/user', {
        method: 'PUT',
        body: JSON.stringify(newUser),
      }),
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['user'] });
    },
  });

  return (
    <button onClick={() => mutation.mutate({ name: 'Alice' })}>
      Update Profile
    </button>
  );
}
```

---

**Optimistic Updates:**

```typescript
const mutation = useMutation({
  mutationFn: updateUser,
  onMutate: async (newUser) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries({ queryKey: ['user'] });

    // Snapshot previous value
    const previousUser = queryClient.getQueryData(['user']);

    // Optimistically update
    queryClient.setQueryData(['user'], newUser);

    // Return context with snapshot
    return { previousUser };
  },
  onError: (err, newUser, context) => {
    // Rollback on error
    queryClient.setQueryData(['user'], context.previousUser);
  },
  onSettled: () => {
    // Always refetch after error or success
    queryClient.invalidateQueries({ queryKey: ['user'] });
  },
});
```

---

**Integration with tRPC:**

LobeHub uses TanStack Query with tRPC for type-safe data fetching:

```typescript
import { trpc } from '@/utils/trpc';

function UserProfile() {
  const { data, isLoading } = trpc.user.getUser.useQuery({ id: '123' });
  //    ^? User | undefined (fully typed!)

  const mutation = trpc.user.updateUser.useMutation();

  return (
    <div>
      <p>{data?.name}</p>
      <button onClick={() => mutation.mutate({ name: 'Alice' })}>
        Update
      </button>
    </div>
  );
}
```

**🎯 Benefit:** Type safety + powerful data management!

---

**Learning Resources:**

- [TanStack Query Documentation](https://tanstack.com/query/latest/docs/framework/react/overview)
- [TanStack Query DevTools](https://tanstack.com/query/latest/docs/framework/react/devtools)

📚 **More details:** [BACKEND_ARCHITECTURE.md](./BACKEND_ARCHITECTURE.md)

---

## Internationalization

### react-i18next 15.1.3

**✅ CURRENT** | **Released:** November 2024 | **React 19 Compatible:** Yes

**What is it?**
react-i18next is a powerful internationalization framework for React based on i18next.

**Why react-i18next?**

**Alternatives considered:**
- next-intl: Good for Next.js, but less flexible
- formatjs: Good, but more complex
- react-intl: Good, but larger bundle

**react-i18next wins:**
- ✅ Powerful translation features
- ✅ Namespaces for organization
- ✅ Interpolation and pluralization
- ✅ Lazy loading translations
- ✅ TypeScript support

---

**Basic Usage:**

**1. Define translations:**

```typescript
// src/locales/default/common.ts
export default {
  welcome: 'Welcome',
  greeting: 'Hello, {{name}}!',
  itemCount_one: '{{count}} item',
  itemCount_other: '{{count}} items',
};
```

**2. Use in components:**

```typescript
import { useTranslation } from 'react-i18next';

function Greeting() {
  const { t } = useTranslation('common');

  return (
    <div>
      <p>{t('welcome')}</p>
      <p>{t('greeting', { name: 'Alice' })}</p>
      <p>{t('itemCount', { count: 5 })}</p>
    </div>
  );
}

// Output:
// Welcome
// Hello, Alice!
// 5 items
```

---

**Key Features:**

**1. Namespaces**

Organize translations by feature:

```typescript
// src/locales/default/chat.ts
export default {
  sendMessage: 'Send Message',
  typing: 'Typing...',
};

// src/locales/default/settings.ts
export default {
  theme: 'Theme',
  language: 'Language',
};

// In component:
const { t } = useTranslation('chat');
t('sendMessage'); // "Send Message"

const { t: ts } = useTranslation('settings');
ts('theme'); // "Theme"
```

---

**2. Pluralization**

```typescript
// Translation:
{
  "item_one": "{{count}} item",
  "item_other": "{{count}} items"
}

// Usage:
t('item', { count: 1 }); // "1 item"
t('item', { count: 5 }); // "5 items"
```

---

**3. Interpolation**

```typescript
// Translation:
{
  "welcome": "Welcome, {{name}}!",
  "balance": "Your balance is ${{amount, number}}"
}

// Usage:
t('welcome', { name: 'Alice' }); // "Welcome, Alice!"
t('balance', { amount: 1234.56 }); // "Your balance is $1,234.56"
```

---

**4. Language Switching**

```typescript
import { useTranslation } from 'react-i18next';

function LanguageSwitcher() {
  const { i18n } = useTranslation();

  return (
    <select
      value={i18n.language}
      onChange={(e) => i18n.changeLanguage(e.target.value)}
    >
      <option value="en">English</option>
      <option value="zh-CN">简体中文</option>
      <option value="ja">日本語</option>
    </select>
  );
}
```

---

**LobeHub i18n Workflow:**

1. **Add keys** to `src/locales/default/namespace.ts`
2. **Translate** `locales/zh-CN/namespace.json` and `locales/en-US/namespace.json` for dev preview
3. **DON'T run** `pnpm i18n` - CI auto-generates all language files

📚 **More details:** [.cursor/rules/i18n.mdc](./.cursor/rules/i18n.mdc)

---

**Learning Resources:**

- [react-i18next Documentation](https://react.i18next.com/)
- [i18next Documentation](https://www.i18next.com/)

---

## React Utilities

### aHooks 3.8.2

**✅ CURRENT** | **Released:** November 2024 | **React 19 Compatible:** Yes

**What is it?**
aHooks is a high-quality React hooks library providing 100+ commonly used hooks.

**Why aHooks?**

Instead of writing custom hooks over and over, aHooks provides battle-tested implementations.

---

**Commonly Used Hooks:**

**1. useDebounce / useThrottle**

```typescript
import { useDebounce } from 'ahooks';

function SearchBox() {
  const [search, setSearch] = useState('');
  const debouncedSearch = useDebounce(search, { wait: 500 });

  useEffect(() => {
    // Only runs 500ms after user stops typing
    fetchResults(debouncedSearch);
  }, [debouncedSearch]);

  return <input value={search} onChange={(e) => setSearch(e.target.value)} />;
}
```

---

**2. useRequest**

```typescript
import { useRequest } from 'ahooks';

function UserProfile() {
  const { data, loading, error, run } = useRequest(
    () => fetch('/api/user').then(r => r.json()),
    {
      manual: true, // Don't run on mount
    }
  );

  return (
    <div>
      <button onClick={run}>Load User</button>
      {loading && <div>Loading...</div>}
      {data && <div>{data.name}</div>}
    </div>
  );
}
```

---

**3. useLocalStorage**

```typescript
import { useLocalStorageState } from 'ahooks';

function ThemeSwitcher() {
  const [theme, setTheme] = useLocalStorageState('theme', {
    defaultValue: 'light',
  });

  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Toggle Theme (current: {theme})
    </button>
  );
}
```

---

**4. useClickAway**

```typescript
import { useClickAway } from 'ahooks';
import { useRef } from 'react';

function Dropdown() {
  const ref = useRef(null);
  const [open, setOpen] = useState(false);

  useClickAway(() => {
    setOpen(false);
  }, ref);

  return (
    <div ref={ref}>
      <button onClick={() => setOpen(!open)}>Toggle</button>
      {open && <div>Dropdown content</div>}
    </div>
  );
}
```

---

**Learning Resources:**

- [aHooks Documentation](https://ahooks.js.org/)
- [aHooks Hook List](https://ahooks.js.org/hooks/use-request)

---

## JavaScript Utilities

### dayjs 1.11.13

**✅ CURRENT** | **Released:** November 2024

**What is it?**
dayjs is a lightweight (2KB) date manipulation library with the same API as Moment.js.

**Why dayjs?**

**Alternatives considered:**
- Moment.js: Deprecated, 70KB
- date-fns: Good, but larger bundle (~13KB)
- Luxon: Good, but larger bundle (~15KB)

**dayjs wins:**
- ✅ Only 2KB!
- ✅ Same API as Moment.js (easy migration)
- ✅ Immutable
- ✅ Plugin system

---

**Basic Usage:**

```typescript
import dayjs from 'dayjs';

// Parse
const date = dayjs('2025-11-20');

// Format
date.format('YYYY-MM-DD'); // "2025-11-20"
date.format('MMM DD, YYYY'); // "Nov 20, 2025"

// Manipulate
date.add(1, 'day'); // 2025-11-21
date.subtract(1, 'month'); // 2025-10-20

// Query
date.isBefore('2025-12-01'); // true
date.isAfter('2025-11-01'); // true
date.isSame('2025-11-20'); // true

// Relative time (with plugin)
import relativeTime from 'dayjs/plugin/relativeTime';
dayjs.extend(relativeTime);

dayjs().fromNow(); // "a few seconds ago"
date.fromNow(); // "in 5 days"
```

---

**Common Plugins:**

```typescript
import dayjs from 'dayjs';
import relativeTime from 'dayjs/plugin/relativeTime';
import utc from 'dayjs/plugin/utc';
import timezone from 'dayjs/plugin/timezone';

dayjs.extend(relativeTime);
dayjs.extend(utc);
dayjs.extend(timezone);

// Relative time
dayjs().fromNow(); // "a few seconds ago"

// UTC
dayjs.utc(); // Current time in UTC

// Timezone
dayjs.tz('2025-11-20', 'America/New_York');
```

---

**Learning Resources:**

- [dayjs Documentation](https://day.js.org/)
- [dayjs Plugins](https://day.js.org/docs/en/plugin/plugin)

---

### lodash-es 4.17.21

**✅ CURRENT** | **Tree-shakeable:** Yes

**What is it?**
lodash-es is the ES modules version of Lodash, providing 300+ utility functions.

**Why lodash-es (not lodash)?**

- **lodash:** CommonJS, entire library bundled (~70KB)
- **lodash-es:** ES modules, tree-shakeable (~2KB per function)

**✅ Always use:** `lodash-es` for better bundle size!

---

**Commonly Used Functions:**

**1. Array Utilities**

```typescript
import { chunk, uniq, difference } from 'lodash-es';

// Chunk array
chunk([1, 2, 3, 4, 5], 2); // [[1, 2], [3, 4], [5]]

// Unique values
uniq([1, 2, 2, 3, 3]); // [1, 2, 3]

// Difference
difference([1, 2, 3], [2, 3, 4]); // [1]
```

---

**2. Object Utilities**

```typescript
import { pick, omit, merge, cloneDeep } from 'lodash-es';

const user = { id: 1, name: 'Alice', age: 25, email: 'alice@example.com' };

// Pick properties
pick(user, ['name', 'email']); // { name: 'Alice', email: 'alice@example.com' }

// Omit properties
omit(user, ['age']); // { id: 1, name: 'Alice', email: 'alice@example.com' }

// Deep merge
merge({ a: 1 }, { b: 2 }, { c: 3 }); // { a: 1, b: 2, c: 3 }

// Deep clone
const clone = cloneDeep(user); // New object, not reference
```

---

**3. String Utilities**

```typescript
import { camelCase, kebabCase, snakeCase } from 'lodash-es';

camelCase('hello world'); // "helloWorld"
kebabCase('hello world'); // "hello-world"
snakeCase('hello world'); // "hello_world"
```

---

**4. Collection Utilities**

```typescript
import { groupBy, keyBy, orderBy } from 'lodash-es';

const users = [
  { id: 1, name: 'Alice', age: 25 },
  { id: 2, name: 'Bob', age: 30 },
  { id: 3, name: 'Charlie', age: 25 },
];

// Group by age
groupBy(users, 'age');
// { 25: [Alice, Charlie], 30: [Bob] }

// Key by id
keyBy(users, 'id');
// { 1: Alice, 2: Bob, 3: Charlie }

// Order by age
orderBy(users, ['age'], ['desc']);
// [Bob, Alice, Charlie]
```

---

**Learning Resources:**

- [Lodash Documentation](https://lodash.com/docs/)

**🎯 Remember:** Always import from `lodash-es`, not `lodash`!

---

## Summary - Part 2

**Technologies Covered:**

| Technology | Version | Purpose | Status |
|------------|---------|---------|--------|
| **Zustand** | 5.0.4 | State management | ✅ CURRENT |
| **nuqs** | 2.2.5 | URL query state | ✅ CURRENT |
| **SWR** | 2.3.0 | Data fetching | ✅ CURRENT |
| **TanStack Query** | 5.76.5 | Advanced data management | ✅ CURRENT |
| **react-i18next** | 15.1.3 | Internationalization | ✅ CURRENT |
| **aHooks** | 3.8.2 | React hooks library | ✅ CURRENT |
| **dayjs** | 1.11.13 | Date manipulation | ✅ CURRENT |
| **lodash-es** | 4.17.21 | Utility functions | ✅ CURRENT |

**Key Takeaways:**

1. 🚀 **Zustand** - Simple, performant state management with slice pattern
2. 🔍 **nuqs** - Type-safe URL query params for Next.js
3. 📡 **SWR** - Simple data fetching with caching
4. ⚡ **TanStack Query** - Powerful data management with tRPC integration
5. 🌍 **react-i18next** - Comprehensive i18n with namespaces
6. 🛠️ **aHooks** - 100+ ready-to-use React hooks
7. 📅 **dayjs** - Lightweight date library (2KB!)
8. 🔧 **lodash-es** - Tree-shakeable utilities

---

**Document Status:** ✅ Part 2 Complete | **Last Updated:** November 20, 2025

**[← Part 1](./TECH-STACK-GUIDE-PART-01.md)** | **[Continue to Part 3 →](./TECH-STACK-GUIDE-PART-03.md)** - Backend (tRPC, APIs) and Database (Drizzle, PGLite, Neon)
