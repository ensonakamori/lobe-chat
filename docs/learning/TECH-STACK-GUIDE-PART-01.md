# Tech Stack Guide - Part 1 of 4

**Documented:** November 2025
**Tech Stack Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)
**Target:** Mid-level developers proficient in JS/TS/React

**📄 This is Part 1 of 4** | **[Continue to Part 2 →](./TECH-STACK-GUIDE-PART-02.md)**

This comprehensive guide explains every technology in the LobeHub stack, why we chose it, and how to use it effectively.

**Prerequisites:**
- [Getting Started](./GETTING_STARTED.md) - Have the project running
- [Architecture Overview](./ARCHITECTURE-OVERVIEW-PART-01.md) - Understand the system architecture

**Time to read:** 30 minutes (Part 1)

---

## Table of Contents - Part 1

1. [Introduction](#introduction)
2. [Frontend Core Technologies](#frontend-core-technologies)
   - [React 19.2.0](#react-1920)
   - [Next.js 16.0.3](#nextjs-1603)
   - [TypeScript 5.9.3](#typescript-593)
3. [UI & Styling](#ui--styling)
   - [@lobehub/ui](#lobeubui)
   - [Ant Design 5.23.3](#ant-design-5233)
   - [antd-style 4.0.0](#antd-style-400)
4. [Icons & Graphics](#icons--graphics)
   - [Lucide React 0.469.0](#lucide-react-04690)
   - [@ant-design/icons](#ant-designicons)
5. [Layout](#layout)
   - [react-layout-kit 2.0.0](#react-layout-kit-200)

**[Part 2 →](./TECH-STACK-GUIDE-PART-02.md)** will cover:
- State Management (Zustand, nuqs)
- Data Fetching (SWR, React Query)
- Internationalization (react-i18next)
- Utilities (aHooks, dayjs, lodash-es)

**[Part 3 →](./TECH-STACK-GUIDE-PART-03.md)** will cover:
- Backend (tRPC, Next.js API Routes)
- Database (Drizzle ORM, PGLite, Neon PostgreSQL)

**[Part 4 →](./TECH-STACK-GUIDE-PART-04.md)** will cover:
- Testing (Vitest, Playwright)
- Build Tools (Turbopack, Bun)
- Infrastructure & Deployment

---

## Introduction

**LobeHub Tech Stack Philosophy:**

> We choose technologies that are **modern, actively maintained, and production-ready** as of November 2025. Every technology in our stack is either at its latest stable version or has a clear upgrade path.

**Key Principles:**

1. **✅ CURRENT** - All dependencies at latest stable versions
2. **🔒 Type Safety** - TypeScript everywhere, end-to-end
3. **⚡ Performance** - Edge-first, optimized for speed
4. **🎨 Developer Experience** - Best-in-class DX
5. **📦 Ecosystem Fit** - Technologies work well together

**🎯 Status as of November 2025:** All technologies are production-ready and actively maintained.

---

## Frontend Core Technologies

### React 19.2.0

**✅ CURRENT** | **Released:** December 2024 | **LTS:** Yes

**What is it?**
React 19 is the latest major version of React, introducing the React Compiler, Server Components, Server Actions, and new hooks.

**Why React 19?**

Traditional React (v18 and earlier) required manual optimization:

```typescript
// ❌ React 18: Manual optimization required
const MyComponent = memo(({ user }) => {
  const fullName = useMemo(() => `${user.first} ${user.last}`, [user]);
  const onClick = useCallback(() => alert(fullName), [fullName]);

  return <button onClick={onClick}>{fullName}</button>;
});
```

React 19 automatically optimizes this:

```typescript
// ✅ React 19: Automatic optimization by React Compiler
function MyComponent({ user }) {
  const fullName = `${user.first} ${user.last}`;
  const onClick = () => alert(fullName);

  return <button onClick={onClick}>{fullName}</button>;
}
// React Compiler automatically memoizes when needed!
```

**Key Features:**

**1. React Compiler (Automatic Memoization)**

The React Compiler analyzes your code and automatically adds memoization where needed.

```typescript
// You write simple code:
function ProductList({ products }) {
  return products.map(product => (
    <ProductCard key={product.id} product={product} />
  ));
}

// React Compiler optimizes it automatically:
// - Memoizes ProductCard renders
// - Optimizes product.map
// - Skips re-renders when props don't change
```

**🌉 Bridge from React 18:** No more manual `memo`, `useMemo`, `useCallback` in most cases!

---

**2. Server Components**

Server Components render on the server, reducing JavaScript sent to the client.

```typescript
// Server Component (default in Next.js 16 App Router)
async function UserProfile({ userId }) {
  // Direct database access! No API needed
  const user = await db.query.users.findFirst({
    where: eq(users.id, userId),
  });

  return <div>{user.name}</div>;
}
```

**Benefits:**
- ✅ Direct database access (no API layer needed)
- ✅ Async by default (use `await` naturally)
- ✅ Zero JavaScript to client
- ✅ Better performance

**🎯 Remember:** Server Components can't use hooks or browser APIs

---

**3. Server Actions**

Server Actions are functions that run on the server but can be called from the client.

```typescript
// app/actions.ts
'use server';

export async function createUser(formData: FormData) {
  const name = formData.get('name');

  await db.insert(users).values({ name });

  revalidatePath('/users'); // Refresh the page
}

// app/page.tsx
import { createUser } from './actions';

function SignupForm() {
  return (
    <form action={createUser}>
      <input name="name" />
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

**🌉 Bridge from React 18:** Like API routes, but type-safe and no boilerplate!

---

**4. New Hooks**

**`useActionState` - Form state management**

```typescript
import { useActionState } from 'react';

function SignupForm() {
  const [state, action, isPending] = useActionState(createUser, null);

  return (
    <form action={action}>
      <input name="name" />
      <button disabled={isPending}>
        {isPending ? 'Submitting...' : 'Sign Up'}
      </button>
      {state?.error && <div>{state.error}</div>}
    </form>
  );
}
```

**`useFormStatus` - Access form submission state**

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
```

**`useOptimistic` - Optimistic updates**

```typescript
import { useOptimistic } from 'react';

function MessageList({ messages }) {
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (state, newMessage) => [...state, newMessage]
  );

  async function sendMessage(text) {
    addOptimisticMessage({ id: 'temp', text, sending: true });
    await createMessage(text);
  }

  return optimisticMessages.map(msg => (
    <div key={msg.id} opacity={msg.sending ? 0.5 : 1}>
      {msg.text}
    </div>
  ));
}
```

---

**Learning Resources:**

- [React 19 Official Docs](https://react.dev/blog/2024/12/05/react-19) ✅ CURRENT
- [React Compiler Playground](https://playground.react.dev/)
- [Server Components Deep Dive](https://react.dev/reference/react/use-server)

**🚨 Migration Notes:**

If upgrading from React 18:
- React Compiler is opt-in (enabled in `next.config.js`)
- Most React 18 code works without changes
- Remove manual `memo`/`useMemo`/`useCallback` where compiler handles it

📚 **More details:** [FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md)

---

### Next.js 16.0.3

**✅ CURRENT** | **Released:** November 2024 | **LTS:** Yes

**What is it?**
Next.js is a React framework providing server-side rendering, routing, API routes, and optimizations.

**Why Next.js 16?**

**Problem with SPAs:**
- Slow initial load (all JavaScript upfront)
- Poor SEO (content not in HTML)
- Manual routing setup
- No built-in API layer

**Solution: Next.js provides:**
- Server-side rendering (fast initial load)
- File-based routing
- API routes + tRPC support
- Automatic code splitting
- Image/font optimization

---

**Key Features:**

**1. App Router (Stable)**

The App Router uses React Server Components by default.

```
app/
├── layout.tsx              ← Root layout
├── page.tsx                ← Home page (/)
├── about/
│   └── page.tsx            ← About page (/about)
└── blog/
    ├── page.tsx            ← Blog list (/blog)
    └── [slug]/
        └── page.tsx        ← Blog post (/blog/post-title)
```

**File = URL:**
```
page.tsx         → /
about/page.tsx   → /about
blog/[slug]/page.tsx → /blog/anything
```

**🌉 Bridge from Pages Router:** No `getServerSideProps`, just use `async` components!

---

**2. Turbopack (Stable)**

Turbopack is Next.js's new bundler, replacing Webpack.

**Performance Comparison:**

| Bundler | Dev Server Start | Hot Reload |
|---------|------------------|------------|
| Webpack | ~10s | ~2s |
| Turbopack | ~1s | <200ms |

**✅ Stable in Next.js 16** - Production-ready as of November 2024

---

**3. Partial Pre-Rendering (PPR)**

PPR combines static and dynamic content in one page.

```typescript
// app/page.tsx
export default async function Page() {
  return (
    <div>
      {/* Static: Pre-rendered at build time */}
      <StaticHeader />

      {/* Dynamic: Rendered at request time */}
      <Suspense fallback={<Skeleton />}>
        <DynamicUserContent />
      </Suspense>
    </div>
  );
}
```

**Benefits:**
- Static parts served instantly from CDN
- Dynamic parts stream in progressively
- Best of both worlds

**🆕 NEW IN 2025** - Experimental in Next.js 16

---

**4. Proxy API**

The Proxy API provides a simple way to proxy requests to external APIs.

```typescript
// src/proxy.ts
export default {
  '/api/openai': {
    target: 'https://api.openai.com',
    changeOrigin: true,
    pathRewrite: { '^/api/openai': '' },
  },
};
```

**🆕 NEW IN 2025** - Replaces custom middleware for proxying

---

**5. Built-in Optimizations**

**Image Optimization:**
```typescript
import Image from 'next/image';

<Image
  src="/photo.jpg"
  width={500}
  height={300}
  alt="Photo"
  // Automatic:
  // - WebP/AVIF conversion
  // - Responsive sizes
  // - Lazy loading
/>
```

**Font Optimization:**
```typescript
import { Inter } from 'next/font/google';

const inter = Inter({ subsets: ['latin'] });

<div className={inter.className}>
  {/* Font automatically:
    - Self-hosted (no Google Fonts request)
    - Preloaded
    - Subsetted */}
</div>
```

---

**Learning Resources:**

- [Next.js 16 Docs](https://nextjs.org/docs) ✅ CURRENT
- [App Router Guide](https://nextjs.org/docs/app)
- [Turbopack Documentation](https://nextjs.org/docs/architecture/turbopack)

**🚨 Migration Notes:**

If upgrading from Next.js 15:
- Turbopack is now stable (remove experimental flags)
- PPR is still experimental (opt-in)
- Pages Router still supported

📚 **More details:** [FRONTEND_ARCHITECTURE.md](./FRONTEND_ARCHITECTURE.md)

---

### TypeScript 5.9.3

**✅ CURRENT** | **Released:** September 2024 | **LTS:** Yes

**What is it?**
TypeScript adds static typing to JavaScript, catching errors at compile time.

**Why TypeScript 5.9?**

**Problem with JavaScript:**
```javascript
// ❌ JavaScript: No type safety
function getUser(id) {
  return fetch(`/api/users/${id}`).then(r => r.json());
}

const user = await getUser(123);
console.log(user.name); // What if user is null? What properties exist?
```

**Solution with TypeScript:**
```typescript
// ✅ TypeScript: Type-safe
interface User {
  id: number;
  name: string;
  email: string;
}

async function getUser(id: number): Promise<User | null> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

const user = await getUser(123);
if (user) {
  console.log(user.name); // ✅ TypeScript knows name exists
  console.log(user.age);  // ❌ Error: Property 'age' does not exist
}
```

---

**Key Features in TypeScript 5.9:**

**1. Deferred Imports**

Improves performance by deferring type-only imports.

```typescript
// ✅ TypeScript 5.9: Only imports when needed
import type { User } from './types';

// User type is not bundled in production!
```

**2. Expanded Type Previews**

Better IntelliSense for complex types:

```typescript
// TypeScript 5.9 shows full type on hover:
type UserWithPosts = User & { posts: Post[] };
//   ^? User { id: number; name: string; email: string; posts: Post[] }

// TypeScript 5.8 just showed:
//   ^? UserWithPosts
```

**3. Improved Inference**

Better type inference for generics:

```typescript
// TypeScript 5.9 correctly infers:
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
];
//    ^? { id: number; name: string }[]

// No need for explicit typing!
```

---

**LobeHub TypeScript Configuration:**

**`tsconfig.json` highlights:**
```json
{
  "compilerOptions": {
    "strict": true,                    // All strict checks enabled
    "target": "ES2022",                // Modern JavaScript
    "module": "ESNext",                // Latest module system
    "moduleResolution": "bundler",     // For Next.js 16
    "esModuleInterop": true,           // Better CommonJS interop
    "skipLibCheck": true,              // Faster builds
    "paths": {
      "@/*": ["./src/*"],              // Path aliases
      "@lobechat/*": ["./packages/*/src"]
    }
  }
}
```

**🎯 Remember:** We use `strict: true` - maximum type safety!

---

**Learning Resources:**

- [TypeScript 5.9 Release Notes](https://devblogs.microsoft.com/typescript/announcing-typescript-5-9/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [Type Challenges](https://github.com/type-challenges/type-challenges)

**Code Style:**
📚 **See:** [.cursor/rules/typescript.mdc](./.cursor/rules/typescript.mdc) for TypeScript style guide

---

## UI & Styling

### @lobehub/ui

**✅ CURRENT** | **Version:** 1.177.18 | **Maintained by:** LobeHub Team

**What is it?**
`@lobehub/ui` is LobeHub's custom component library, providing pre-built, styled components for the LobeHub design system.

**Why @lobehub/ui?**

Built on top of Ant Design, `@lobehub/ui` provides:
- Consistent design language across the app
- Dark mode support built-in
- Responsive components
- Pre-configured with antd-style

**Example Components:**

```typescript
import { ChatInput, MessageList, Avatar } from '@lobehub/ui';

<ChatInput
  onSend={(message) => sendMessage(message)}
  placeholder="Type a message..."
/>

<MessageList
  messages={messages}
  renderMessage={(msg) => <div>{msg.content}</div>}
/>
```

**🎯 Benefit:** Domain-specific components (chat, messages, agents) ready to use

---

### Ant Design 5.23.3

**✅ CURRENT** | **Released:** November 2024 | **React 19 Compatible:** Yes

**What is it?**
Ant Design is a comprehensive React UI library with 50+ components.

**Why Ant Design?**

**Alternatives considered:**
- Material-UI: Good, but Ant Design has better theming
- Chakra UI: Good DX, but smaller component library
- shadcn/ui: Excellent, but requires more setup

**Ant Design wins:**
- ✅ 50+ production-ready components
- ✅ Excellent TypeScript support
- ✅ Built-in dark mode
- ✅ Highly customizable theming
- ✅ Active maintenance

**Example Usage:**

```typescript
import { Button, Input, Modal, Table } from 'antd';

<Button type="primary" onClick={handleClick}>
  Click Me
</Button>

<Input
  placeholder="Enter text"
  onChange={(e) => setText(e.target.value)}
/>

<Modal
  title="Confirm"
  open={isOpen}
  onOk={handleOk}
  onCancel={handleCancel}
>
  Are you sure?
</Modal>
```

**🌉 Bridge from React:** Like Bootstrap for React, but more comprehensive

---

### antd-style 4.0.0

**✅ CURRENT** | **Released:** November 2024 | **React 19 Compatible:** Yes

**What is it?**
`antd-style` is a CSS-in-JS library designed for Ant Design, providing theme tokens and responsive styling.

**Why antd-style?**

**Problem with traditional CSS:**
```css
/* ❌ Hard-coded colors don't support dark mode */
.button {
  background: #1890ff;
  color: #ffffff;
}
```

**Solution with antd-style:**
```typescript
import { createStyles } from 'antd-style';

const useStyles = createStyles(({ token, css }) => ({
  button: css`
    background: ${token.colorPrimary};  /* Automatically switches for dark mode */
    color: ${token.colorTextLightSolid};
    padding: ${token.paddingMD}px;

    &:hover {
      background: ${token.colorPrimaryHover};
    }
  `,
}));

function MyButton() {
  const { styles } = useStyles();
  return <button className={styles.button}>Click</button>;
}
```

**Key Features:**

**1. Theme Tokens**

Access Ant Design theme tokens in your styles:

```typescript
const useStyles = createStyles(({ token }) => ({
  container: {
    background: token.colorBgContainer,    // Background color
    padding: token.paddingLG,              // Large padding
    borderRadius: token.borderRadiusLG,    // Large border radius
    boxShadow: token.boxShadow,            // Shadow
  },
}));
```

**Available tokens:**
- Colors: `colorPrimary`, `colorSuccess`, `colorWarning`, `colorError`
- Spacing: `paddingXS`, `paddingSM`, `paddingMD`, `paddingLG`, `paddingXL`
- Typography: `fontSize`, `fontSizeLG`, `lineHeight`
- And 100+ more...

**🎯 Benefit:** Dark mode works automatically!

---

**2. Responsive Styles**

```typescript
const useStyles = createStyles(({ token, responsive }) => ({
  container: {
    padding: token.paddingMD,

    [responsive.mobile]: {
      padding: token.paddingSM,  // Smaller padding on mobile
    },

    [responsive.tablet]: {
      padding: token.paddingLG,  // Larger padding on tablet
    },
  },
}));
```

**🌉 Bridge from CSS:** Like CSS media queries, but with semantic breakpoints

---

**Learning Resources:**

- [antd-style Documentation](https://github.com/ant-design/antd-style)
- [Ant Design Theming](https://ant.design/docs/react/customize-theme)

---

## Icons & Graphics

### Lucide React 0.469.0

**✅ CURRENT** | **Released:** November 2024 | **Icons:** 1,500+

**What is it?**
Lucide is a modern icon library with 1,500+ beautiful, consistent icons.

**Why Lucide?**

**Alternatives considered:**
- Font Awesome: Good, but heavier bundle size
- Heroicons: Good, but smaller library
- React Icons: Good, but inconsistent styles

**Lucide wins:**
- ✅ 1,500+ icons, all consistent style
- ✅ Tree-shakeable (only bundle icons you use)
- ✅ TypeScript types included
- ✅ Customizable (size, color, stroke width)

**Example Usage:**

```typescript
import { MessageSquare, Send, User, Settings } from 'lucide-react';

<MessageSquare size={24} />              {/* Default size */}
<Send size={16} color="blue" />          {/* Custom size and color */}
<User strokeWidth={1.5} />               {/* Custom stroke width */}
<Settings className="icon-settings" />   {/* Custom className */}
```

**🎯 Benefit:** Bundle size = ~1KB per icon (much smaller than Font Awesome)

---

### @ant-design/icons

**✅ CURRENT** | **Version:** 5.5.2 | **Icons:** 700+

**What is it?**
Official icon library for Ant Design.

**When to use which?**

| Icon Library | Use When |
|--------------|----------|
| **Lucide** | General UI icons (send, user, settings, etc.) |
| **@ant-design/icons** | Ant Design component icons (built-in styling) |

**Example:**

```typescript
import { LoadingOutlined, CheckCircleOutlined } from '@ant-design/icons';
import { Spin } from 'antd';

<Spin indicator={<LoadingOutlined spin />} />
<CheckCircleOutlined style={{ color: 'green' }} />
```

---

## Layout

### react-layout-kit 2.0.0

**✅ CURRENT** | **Released:** November 2024 | **React 19 Compatible:** Yes

**What is it?**
`react-layout-kit` provides flex layout components for building responsive UIs quickly.

**Why react-layout-kit?**

**Problem with manual flexbox:**
```typescript
// ❌ Verbose flexbox styling
<div style={{
  display: 'flex',
  flexDirection: 'column',
  alignItems: 'center',
  justifyContent: 'space-between',
  gap: '16px',
}}>
  <div>Header</div>
  <div>Content</div>
  <div>Footer</div>
</div>
```

**Solution with react-layout-kit:**
```typescript
import { Flexbox } from 'react-layout-kit';

<Flexbox
  direction="column"
  align="center"
  justify="space-between"
  gap={16}
>
  <div>Header</div>
  <div>Content</div>
  <div>Footer</div>
</Flexbox>
```

**Key Components:**

**1. Flexbox**

```typescript
<Flexbox
  direction="row"           // flex-direction
  align="center"            // align-items
  justify="space-between"   // justify-content
  gap={16}                  // gap
  wrap="wrap"               // flex-wrap
>
  {children}
</Flexbox>
```

**2. Center**

```typescript
<Center width="100%" height="100vh">
  <div>Perfectly centered content</div>
</Center>
```

**3. Grid** (Coming in v2.1)

**🌉 Bridge from CSS:** Like writing `display: flex`, but in JSX with semantic props

📚 **More details:** [.cursor/rules/packages/react-layout-kit.mdc](./.cursor/rules/packages/react-layout-kit.mdc)

---

## Summary - Part 1

**Frontend Technologies Covered:**

| Technology | Version | Purpose | Status |
|------------|---------|---------|--------|
| **React** | 19.2.0 | UI library with Server Components | ✅ CURRENT |
| **Next.js** | 16.0.3 | React framework with SSR | ✅ CURRENT |
| **TypeScript** | 5.9.3 | Type safety | ✅ CURRENT |
| **@lobehub/ui** | 1.177.18 | Custom component library | ✅ CURRENT |
| **Ant Design** | 5.23.3 | UI component library | ✅ CURRENT |
| **antd-style** | 4.0.0 | CSS-in-JS with theme tokens | ✅ CURRENT |
| **Lucide React** | 0.469.0 | Icon library | ✅ CURRENT |
| **react-layout-kit** | 2.0.0 | Flex layout components | ✅ CURRENT |

**Key Takeaways:**

1. 🚀 **React 19** - Automatic optimization via React Compiler
2. ⚡ **Next.js 16** - Turbopack is stable, PPR is experimental
3. 🔒 **TypeScript 5.9** - All strict checks enabled
4. 🎨 **antd-style** - Theme tokens enable automatic dark mode
5. 📦 **Lucide** - Tree-shakeable icons for minimal bundle size

---

**Document Status:** ✅ Part 1 Complete | **Last Updated:** November 20, 2025

**[Continue to Part 2 →](./TECH-STACK-GUIDE-PART-02.md)** - State Management, Data Fetching, Internationalization, and Utilities
