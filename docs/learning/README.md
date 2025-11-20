# LobeHub Learning Path - START HERE 🤯

**Welcome to LobeHub!** This comprehensive learning guide will help you become productive with this cutting-edge AI chat framework.

**Documented:** November 2025
**Tech Stack Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)
**Target Audience:** Mid-level developers proficient in JavaScript/TypeScript and React

---

## 🎯 About This Project

**LobeHub** (formerly lobe-chat) is an open-source, modern-design AI chat framework supporting:
- 🌐 **Web** (desktop/mobile)
- 💻 **Desktop** (Electron)
- 📱 **Mobile App** (React Native - coming soon)

**Logo:** 🤯

### What Makes This Project Special (Nov 2025)

This project uses **cutting-edge, production-ready technologies**:

✅ **React 19.2.0** - Latest stable with Server Components, Server Actions, new hooks
✅ **Next.js 16.0.3** - Latest with stable Turbopack, Proxy API, enhanced PPR
✅ **TypeScript 5.9.3** - Latest with deferred imports, expandable types
✅ **tRPC 11.7** - Type-safe APIs without code generation
✅ **Zustand 5.0** - Modern state management (React 18+ required)
✅ **Drizzle ORM 0.44** - Lightweight, TypeScript-first database toolkit
✅ **PGLite 0.2** - PostgreSQL in WebAssembly (runs in browser!)
✅ **Vitest 3.2** - Next-gen testing framework

**All documentation reflects November 2025 best practices.** Outdated patterns are clearly marked with ⚠️.

---

## 🚀 Quick Start

**First time here?** Follow this path:

1. **[Getting Started](./GETTING_STARTED.md)** - Install, setup, run the project (30 min)
2. **[Architecture Overview](./ARCHITECTURE_OVERVIEW.md)** - Understand the big picture (45 min)
3. **[Project Structure](./PROJECT_STRUCTURE.md)** - Know where everything lives (30 min)
4. **[Tech Stack Guide](./TECH_STACK_GUIDE.md)** - Deep dive into technologies (2 hours)
5. **[Development Workflow](./DEVELOPMENT_WORKFLOW.md)** - Daily development flow (30 min)

**Total onboarding time:** ~4-5 hours to productivity

---

## 📚 Documentation Index

### 🏗️ Foundation (Start Here)

Essential documents for understanding the project:

| Document | What You'll Learn | Time | Prerequisites |
|----------|-------------------|------|---------------|
| **[Getting Started](./GETTING_STARTED.md)** | Installation, setup, first run | 30 min | Node.js knowledge |
| **[Architecture Overview](./ARCHITECTURE_OVERVIEW.md)** | System design, data flow, patterns | 45 min | React basics |
| **[Project Structure](./PROJECT_STRUCTURE.md)** | Directory organization, monorepo layout | 30 min | None |
| **[Tech Stack Guide](./TECH_STACK_GUIDE.md)** | All technologies explained with React analogies | 2 hrs | JS/TS/React |
| **[Data Flow Guide](./DATA_FLOW_GUIDE.md)** | Request lifecycle, state management, DB flow | 1 hr | Architecture Overview |

### 🔬 Deep Dives (Technology-Specific)

Detailed exploration of major subsystems:

| Document | Focus Area | Time | Best For |
|----------|------------|------|----------|
| **[Frontend Architecture](./FRONTEND_ARCHITECTURE.md)** | React 19, Next.js 16, UI components | 1.5 hrs | Frontend developers |
| **[Backend Architecture](./BACKEND_ARCHITECTURE.md)** | tRPC, API routes, authentication | 1.5 hrs | Backend developers |
| **[Database Architecture](./DATABASE_ARCHITECTURE.md)** | Drizzle ORM, PGLite, Neon, migrations | 1.5 hrs | Database work |
| **[Integration Guide](./INTEGRATION_GUIDE.md)** | LLM providers, plugins, external services | 1 hr | Integration work |

### 🛠️ Practical Guides (Hands-On)

Real-world development guides:

| Document | What You'll Do | Time | When to Read |
|----------|----------------|------|--------------|
| **[Patterns & Conventions](./PATTERNS_AND_CONVENTIONS.md)** | Code style, current vs outdated patterns | 1 hr | Before coding |
| **[How-To Guide](./HOW_TO_GUIDE.md)** | Step-by-step common tasks | 1.5 hrs | When building features |
| **[Code Tours](./CODE_TOURS.md)** | Follow code paths through the system | 1.5 hrs | Understanding flows |
| **[Development Workflow](./DEVELOPMENT_WORKFLOW.md)** | Daily dev routine, tools, scripts | 30 min | Day 1 |

### 📋 Quality & Reference

Testing, debugging, and security:

| Document | Coverage | Time | When to Read |
|----------|----------|------|--------------|
| **[Testing Guide](./TESTING_GUIDE.md)** | Vitest, Playwright, best practices | 1 hr | Writing tests |
| **[Debugging Guide](./DEBUGGING_GUIDE.md)** | Tools, techniques, common issues | 45 min | When stuck |
| **[Security Guide](./SECURITY_GUIDE.md)** | Auth, validation, SSRF protection | 1 hr | Handling user data |
| **[API Documentation](./API_DOCUMENTATION.md)** | tRPC routes, REST endpoints | 1 hr | API development |
| **[Database Schema](./DATABASE_SCHEMA.md)** | Tables, relationships, indexes | 45 min | Database work |

### 🎓 Learning & Contributing

Exercises and contribution guides:

| Document | Purpose | Time | Best For |
|----------|---------|------|----------|
| **[Exercises](./EXERCISES.md)** | Hands-on practice tasks | 3-4 hrs | Learning by doing |
| **[First Contributions](./FIRST_CONTRIBUTIONS.md)** | How to contribute, good first issues | 30 min | New contributors |
| **[FAQ](./FAQ.md)** | Common questions answered | 20 min | Quick answers |

### 📊 Meta Documentation

Planning and research documents:

| Document | Purpose |
|----------|---------|
| **[Tech Stack Research](./TECH_STACK_RESEARCH.md)** | Nov 2025 version research, what's current |
| **[Execution Plan](./EXECUTION_PLAN.md)** | Documentation creation roadmap |

---

## 🎓 Learning Paths by Role

Choose your path based on your goals:

### 🎨 Frontend Developer Path

**Goal:** Build UI features, work with React 19 and Next.js 16

```
1. Getting Started (30 min)
2. Architecture Overview (45 min)
3. Tech Stack Guide - Focus on React, Next.js, Zustand (1 hr)
4. Frontend Architecture (1.5 hrs)
5. Patterns & Conventions (1 hr)
6. How-To Guide - UI tasks (30 min)
7. Code Tours - Component rendering (30 min)
```

**Total:** ~6 hours → Start building!

**Key Technologies:**
- React 19 (Server Components, Server Actions, new hooks)
- Next.js 16 (App Router, Turbopack, Proxy API)
- Zustand 5.0 (state management)
- Ant Design + antd-style (UI + CSS-in-JS)
- SWR / TanStack Query (data fetching)

---

### ⚙️ Backend Developer Path

**Goal:** Build APIs, manage data, integrate services

```
1. Getting Started (30 min)
2. Architecture Overview (45 min)
3. Tech Stack Guide - Focus on tRPC, Drizzle (1 hr)
4. Backend Architecture (1.5 hrs)
5. Database Architecture (1.5 hrs)
6. API Documentation (1 hr)
7. How-To Guide - API tasks (30 min)
8. Security Guide (1 hr)
```

**Total:** ~8 hours → Start building!

**Key Technologies:**
- tRPC 11.7 (type-safe APIs)
- Drizzle ORM (database)
- Next.js Server Actions
- NextAuth 5.0 (authentication)
- PGLite + Neon (dual DB strategy)

---

### 🗄️ Full-Stack Developer Path

**Goal:** Understand the complete system

```
1. Getting Started (30 min)
2. Architecture Overview (45 min)
3. Project Structure (30 min)
4. Tech Stack Guide (2 hrs)
5. Data Flow Guide (1 hr)
6. Frontend Architecture (1.5 hrs)
7. Backend Architecture (1.5 hrs)
8. Database Architecture (1.5 hrs)
9. Patterns & Conventions (1 hr)
10. Testing Guide (1 hr)
```

**Total:** ~12.5 hours → Start building!

---

### 🔌 Integration Developer Path

**Goal:** Add LLM providers, plugins, external services

```
1. Getting Started (30 min)
2. Architecture Overview (45 min)
3. Integration Guide (1 hr)
4. Backend Architecture - Focus on modules (1 hr)
5. How-To Guide - Integration tasks (30 min)
6. Code Tours - Integration flow (30 min)
7. API Documentation (1 hr)
```

**Total:** ~5.5 hours → Start integrating!

**Key Technologies:**
- LLM provider SDKs (OpenAI, Anthropic, Google, etc.)
- Model Runtime (packages/model-runtime)
- Plugin system
- File loaders, web crawler

---

### 🧪 QA / Testing Path

**Goal:** Write tests, ensure quality

```
1. Getting Started (30 min)
2. Project Structure (30 min)
3. Testing Guide (1 hr)
4. Patterns & Conventions - Testing patterns (30 min)
5. Frontend Architecture - Component testing (30 min)
6. Backend Architecture - API testing (30 min)
7. Database Architecture - DB testing (30 min)
```

**Total:** ~4 hours → Start testing!

**Key Technologies:**
- Vitest 3.2 (unit/integration tests)
- Playwright 1.56 (E2E tests)
- Testing Library (React component tests)

---

## 🌉 For React Developers New to This Stack

**You know React well, but this project uses some new technologies.** Here's what to focus on:

### Already Familiar ✅

If you know React, you already understand:
- Component composition
- Props and state
- Hooks (useState, useEffect, useContext, etc.)
- JSX syntax
- Virtual DOM concepts

### New in React 19 🆕

**React 19 introduced major features** that this project uses:

1. **Server Components** - Components that render on the server (🌉 like PHP, but React)
2. **Server Actions** - Call server functions from client (🌉 like tRPC, but built-in)
3. **New Hooks:**
   - `useActionState` (🌉 like useState + async action)
   - `useFormStatus` (🌉 like form state manager)
   - `useOptimistic` (🌉 like optimistic UI updates)
4. **ref as prop** - No more `forwardRef` needed!

📚 **Read:** [Tech Stack Guide - React 19](./TECH_STACK_GUIDE.md#react-19) for detailed explanations

### New Patterns to Learn 📖

| Technology | What It Is | React Analogy | Time to Learn |
|------------|------------|---------------|---------------|
| **Next.js 16** | React framework | Create React App, but server-rendered | 2-3 hours |
| **tRPC** | Type-safe APIs | REST API, but with TypeScript superpowers | 1-2 hours |
| **Zustand** | State management | Redux, but simpler (no reducers!) | 1 hour |
| **Drizzle ORM** | Database toolkit | Prisma, but lighter | 1-2 hours |
| **SWR** | Data fetching | useEffect + fetch, but better | 30 min |
| **antd-style** | CSS-in-JS | Styled-components, for Ant Design | 30 min |

**Total learning time:** ~8-12 hours of focused reading

💡 **Pro Tip:** All these technologies make React development **easier** and **more type-safe**, not harder!

---

## 📖 Understanding the Tech Stack

### Core Technologies (November 2025)

This project is built with **modern, production-ready** technologies:

#### Frontend Stack

```
React 19.2.0
  ↓ (renders with)
Next.js 16.0.3
  ↓ (styled with)
Ant Design 5.28 + antd-style 3.7
  ↓ (state managed by)
Zustand 5.0.4
  ↓ (data fetched with)
SWR 2.3.6 + TanStack Query 5.90
```

#### Backend Stack

```
Next.js Server Components
  ↓ (APIs built with)
tRPC 11.7.1
  ↓ (data stored with)
Drizzle ORM 0.44.7
  ↓ (databases)
PGLite 0.2 (client) + Neon (server)
```

#### Development Stack

```
TypeScript 5.9.3
  ↓ (tested with)
Vitest 3.2.4 + Playwright 1.56
  ↓ (built with)
Turbopack (Next.js 16 default)
  ↓ (packages managed by)
pnpm 10.20.0
  ↓ (scripts run with)
Bun 1.3.1
```

**Want details?** See [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md) for version research and current status.

---

## 🧠 Key Concepts to Understand

Before diving deep, grasp these foundational concepts:

### 1. Monorepo Structure

**🧠 Mental Model:** One repository containing multiple related packages

```
lobe-chat/
├── src/              ← Main app
└── packages/         ← Shared libraries (19 packages)
    ├── database/     ← DB schemas, models
    ├── types/        ← TypeScript types
    ├── utils/        ← Shared utilities
    └── ...
```

**🌉 Bridge from React:** Like having multiple npm packages in one repo, all versioned together

📚 **Read:** [Project Structure](./PROJECT_STRUCTURE.md)

---

### 2. Dual Database Strategy

**🧠 Mental Model:** Two databases for different purposes

- **PGLite** (client-side): PostgreSQL compiled to WebAssembly, runs in browser
- **Neon** (server-side): Serverless PostgreSQL in the cloud

**🌉 Bridge from React:** Like having localStorage (client) and a REST API database (server)

**💡 Aha Moment:** The same PostgreSQL SQL works in both! Same schema, same queries.

📚 **Read:** [Database Architecture](./DATABASE_ARCHITECTURE.md)

---

### 3. Type-Safe APIs with tRPC

**🧠 Mental Model:** API routes that share TypeScript types with the client

```typescript
// Server defines route
export const userRouter = router({
  getUser: publicProcedure.query(() => { ... })
});

// Client calls it (fully typed!)
const user = await trpc.user.getUser.query();
//    ^? User type automatically inferred!
```

**🌉 Bridge from React:** Like REST API + TypeScript, but the types are automatically shared

**🎯 Remember This:** "No API documentation needed - the types ARE the documentation"

📚 **Read:** [Backend Architecture - tRPC](./BACKEND_ARCHITECTURE.md#trpc)

---

### 4. React Server Components (RSC)

**🧠 Mental Model:** Components that render on the server, send HTML to client

```typescript
// Server Component (default in Next.js App Router)
async function UserProfile() {
  const user = await db.query.users.findFirst(); // Direct DB access!
  return <div>{user.name}</div>;
}
```

**🌉 Bridge from React:** Like getServerSideProps, but for components

**💡 Aha Moment:** No useEffect + fetch needed! Just async/await in the component.

**⚠️ Common Pitfall:** Can't use useState/useEffect in Server Components (they don't run in browser)

📚 **Read:** [Frontend Architecture - Server Components](./FRONTEND_ARCHITECTURE.md#react-server-components)

---

### 5. App Router vs Pages Router

**🧠 Mental Model:** Two routing systems in Next.js

- **Pages Router** (old): `pages/about.tsx` → `/about`
- **App Router** (new, this project uses): `app/about/page.tsx` → `/about`

**🆕 NEW IN 2025:** App Router is now the recommended approach

**🌉 Bridge from React:** Like React Router, but file-based

✅ **This project uses App Router exclusively** (Next.js 16 best practice)

📚 **Read:** [Frontend Architecture - App Router](./FRONTEND_ARCHITECTURE.md#app-router)

---

## ⚡ Quick Reference

### Common Commands

```bash
# Install dependencies
pnpm install

# Run dev server (web)
bun run dev              # Port 3010

# Run dev server (desktop)
bun run dev:desktop      # Port 3015

# Run tests
bunx vitest run --silent='passed-only' 'path/to/test'

# Type checking
bun run type-check

# Build for production
bun run build

# Database commands
bun run db:generate      # Generate migrations
bun run db:migrate       # Run migrations
bun run db:studio        # Open Drizzle Studio
```

📚 **Full commands:** [Development Workflow](./DEVELOPMENT_WORKFLOW.md)

---

### Project File Organization

```
Where do I put...?

✅ React component      → src/components/ (shared) or src/features/ (feature-specific)
✅ New page             → src/app/[variants]/(main)/[page-name]/page.tsx
✅ tRPC route           → src/server/routers/lambda/ or /async/ or /desktop/
✅ Database schema      → packages/database/src/schemas/
✅ TypeScript type      → packages/types/src/
✅ Utility function     → packages/utils/src/ (shared) or src/utils/ (app-specific)
✅ Test file            → Same directory as the file, with .test.ts suffix
✅ Translation key      → src/locales/default/[namespace].ts
```

📚 **Full structure:** [Project Structure](./PROJECT_STRUCTURE.md)

---

### Code Patterns (Current vs Outdated)

| Pattern | Status | Use This Instead |
|---------|--------|------------------|
| `getServerSideProps` | 🚨 **DEPRECATED** | React Server Components |
| `getStaticProps` | 🚨 **DEPRECATED** | React Server Components |
| Pages Router | ⚠️ **OUTDATED** | App Router |
| `forwardRef` | ⚠️ **OUTDATED** | `ref` as prop (React 19) |
| `useEffect` for data fetching | ⚠️ **OUTDATED** | SWR, TanStack Query, or RSC |
| Zustand v4 patterns | ⚠️ **OUTDATED** | Zustand v5 with `useShallow` |
| Class components | 🚨 **DEPRECATED** | Function components |

📚 **Full patterns:** [Patterns & Conventions](./PATTERNS_AND_CONVENTIONS.md)

---

## 🎯 Your First Week

A suggested timeline for getting productive:

### Day 1: Setup & Understanding (4-5 hours)

- [ ] Clone repository
- [ ] Read [Getting Started](./GETTING_STARTED.md)
- [ ] Install dependencies and run dev server
- [ ] Read [Architecture Overview](./ARCHITECTURE_OVERVIEW.md)
- [ ] Browse [Project Structure](./PROJECT_STRUCTURE.md)
- [ ] Explore the codebase in your editor

**Goal:** Have the project running and understand the big picture

---

### Day 2: Deep Dive Technologies (6-8 hours)

- [ ] Read [Tech Stack Guide](./TECH_STACK_GUIDE.md) cover to cover
- [ ] Try examples in the codebase
- [ ] Read [Data Flow Guide](./DATA_FLOW_GUIDE.md)
- [ ] Follow a request from UI to database and back

**Goal:** Understand how technologies work together

---

### Day 3: Frontend or Backend Focus (4-6 hours)

**Choose your path:**

**Frontend Track:**
- [ ] Read [Frontend Architecture](./FRONTEND_ARCHITECTURE.md)
- [ ] Study components in `src/components/`
- [ ] Look at pages in `src/app/[variants]/(main)/`
- [ ] Try modifying a component

**Backend Track:**
- [ ] Read [Backend Architecture](./BACKEND_ARCHITECTURE.md)
- [ ] Read [Database Architecture](./DATABASE_ARCHITECTURE.md)
- [ ] Study tRPC routes in `src/server/routers/`
- [ ] Try adding a simple tRPC procedure

**Goal:** Understand your primary focus area

---

### Day 4: Patterns & Workflows (4-5 hours)

- [ ] Read [Patterns & Conventions](./PATTERNS_AND_CONVENTIONS.md)
- [ ] Read [Development Workflow](./DEVELOPMENT_WORKFLOW.md)
- [ ] Read [How-To Guide](./HOW_TO_GUIDE.md)
- [ ] Try completing one "How-To" task

**Goal:** Learn the project's way of doing things

---

### Day 5: Practice & Explore (4-6 hours)

- [ ] Read [Code Tours](./CODE_TOURS.md)
- [ ] Pick one code tour and follow it step-by-step
- [ ] Try an exercise from [Exercises](./EXERCISES.md)
- [ ] Read [First Contributions](./FIRST_CONTRIBUTIONS.md)
- [ ] Find a "good first issue" to attempt

**Goal:** Practice what you've learned

---

### Week 1 Checkpoint

**Can you:**
- ✅ Run the development server?
- ✅ Explain the dual database strategy?
- ✅ Add a simple React component?
- ✅ Create a basic tRPC route?
- ✅ Write a simple test?
- ✅ Find your way around the codebase?

**If yes → You're ready to contribute!**

**If no → Revisit the relevant guides and ask questions**

---

## 🤔 Common Questions

### "This seems overwhelming. Where do I really start?"

**Answer:** Just three documents to start:
1. [Getting Started](./GETTING_STARTED.md) - Get it running (30 min)
2. [Architecture Overview](./ARCHITECTURE_OVERVIEW.md) - Understand the system (45 min)
3. [Tech Stack Guide](./TECH_STACK_GUIDE.md) - Learn the tech (2 hours)

**Everything else is reference material** - read as needed!

---

### "I know React but not Next.js. Can I contribute?"

**Absolutely!** Next.js is just React with:
- File-based routing (easy to understand)
- Server-side rendering (optional to use)
- Some extra features (learn as you go)

**Start here:** [Tech Stack Guide - Next.js](./TECH_STACK_GUIDE.md#nextjs) - we explain it from a React perspective!

---

### "What's the difference between `src/services/` and `src/server/services/`?"

**Great question!**

- **`src/services/`** - Client-side services (can access PGLite client DB)
- **`src/server/services/`** - Server-side services (can access Neon server DB)

📚 **Read:** [Project Structure - Services](./PROJECT_STRUCTURE.md#services)

---

### "Why two databases (PGLite and Neon)?"

**Innovation!** This project pioneered dual database architecture:

- **PGLite** - For offline use, instant queries, no server needed
- **Neon** - For server-only data, collaboration, sync

**💡 Best of both worlds:** Fast local experience + cloud sync when available

📚 **Read:** [Database Architecture - Dual Strategy](./DATABASE_ARCHITECTURE.md#dual-database-strategy)

---

### "Is this documentation accurate for November 2025?"

**Yes!** All documentation:
- ✅ Researched current technology versions (Nov 2025)
- ✅ Marks outdated patterns with ⚠️
- ✅ Links to current official docs
- ✅ Notes what's new since January 2025

📚 **See research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)

---

## 📞 Getting Help

### Within This Documentation

- **Quick answers:** [FAQ](./FAQ.md)
- **Debugging issues:** [Debugging Guide](./DEBUGGING_GUIDE.md)
- **Common tasks:** [How-To Guide](./HOW_TO_GUIDE.md)

### External Resources

- **Official Docs:** Links provided in each technology section
- **Project Issues:** [GitHub Issues](https://github.com/lobehub/lobe-chat/issues)
- **Current Patterns:** All marked with ✅ in this documentation

---

## 🌟 Next Steps

**Ready to start?** Choose your path:

**→ New to the project?** Start with [Getting Started](./GETTING_STARTED.md)

**→ Want the big picture?** Read [Architecture Overview](./ARCHITECTURE_OVERVIEW.md)

**→ Need to find something?** Check [Project Structure](./PROJECT_STRUCTURE.md)

**→ Want to learn technologies?** Dive into [Tech Stack Guide](./TECH_STACK_GUIDE.md)

**→ Ready to code?** Follow [Development Workflow](./DEVELOPMENT_WORKFLOW.md)

**→ Want to contribute?** See [First Contributions](./FIRST_CONTRIBUTIONS.md)

---

## 📊 Documentation Status

| Document | Status | Last Updated |
|----------|--------|--------------|
| README.md (this file) | ✅ Complete | Nov 20, 2025 |
| TECH_STACK_RESEARCH.md | ✅ Complete | Nov 20, 2025 |
| EXECUTION_PLAN.md | ✅ Complete | Nov 20, 2025 |
| GETTING_STARTED.md | 🚧 In Progress | - |
| ARCHITECTURE_OVERVIEW.md | 📋 Planned | - |
| PROJECT_STRUCTURE.md | 📋 Planned | - |
| (Additional docs) | 📋 Planned | - |

**Documentation is being created systematically.** Check back for updates!

---

**Happy learning! Welcome to the LobeHub team! 🤯**
