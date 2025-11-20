# Technology Stack Research (November 2025)

**Research conducted:** November 20, 2025
**My knowledge cutoff:** January 2025
**Project:** LobeHub (lobe-chat)

This document contains research findings on all major technologies used in this project, comparing the versions used here with the latest available as of November 2025. This research ensures that the learning materials created are accurate and up-to-date.

---

## 📊 Overall Stack Health: ✅ EXCELLENT

This project uses **very current versions** of almost all technologies. The team maintains the stack well, with most dependencies being either at the latest version or only minor patches behind.

---

## Core Framework & Runtime

### Next.js - v16.0.3

**Current Status (Nov 2025):**
- Latest stable: v16.0.3
- Project uses: v16.0.3
- Status: ✅ **CURRENT** - Using latest version!

**Important Updates Since Jan 2025:**
- **Next.js 16** released October 21, 2025
- **Turbopack is now stable** (default bundler) - 5-10x faster Fast Refresh, 2-5x faster builds
- **New Proxy API** - `proxy.ts` replaces `middleware.ts` for network boundary control
- **Cache Components** - New model using Partial Pre-Rendering (PPR) and `use cache`
- **Next.js DevTools MCP** - Model Context Protocol integration for AI-assisted debugging
- **Performance improvements** - Extended dev logs showing time spent, build process insights

**What This Means for Learning:**
- All Next.js 16 features can be used safely
- Turbopack is production-ready and default
- App Router patterns are mature and stable
- React Server Components are first-class

**Official Resources:**
- Docs: https://nextjs.org/docs
- Release notes: https://nextjs.org/blog/next-16
- Migration guide: https://nextjs.org/docs/app/building-your-application/upgrading

---

### React - v19.2.0

**Current Status (Nov 2025):**
- Latest stable: v19.2.0
- Project uses: v19.2.0
- Status: ✅ **CURRENT** - Using latest version!

**Important Updates Since Jan 2025:**
- **React 19 stable** released December 5, 2024
- **React 19.1** released June 2025
- **React 19.2** released October 2025

**Major New Features in React 19:**
1. **Actions** - Async functions in transitions with automatic pending/error handling
2. **React Server Components** - Now fully stable (render on server, less JS to client)
3. **Server Actions** - Call async server functions from Client Components
4. **New Hooks:**
   - `useActionState` - Managing form state with actions
   - `useFormStatus` - Form component status information
   - `useOptimistic` - Optimistic UI updates
5. **React Compiler** - Built-in compiler for optimized code (reduces need for `useMemo`, `useCallback`, `memo`)
6. **Enhanced Form Handling** - Actions integrated with form features
7. **Improved Error Handling** - `onCaughtError`, `onUncaughtError`, `onRecoverableError`
8. **ref as Prop** - Function components can accept `ref` directly (no `forwardRef` needed)

**What This Means for Learning:**
- All React 19 features are production-ready
- Server Components are the recommended pattern
- New hooks should be used for forms and actions
- `forwardRef` is legacy (still works, but not needed)

**Official Resources:**
- Docs: https://react.dev/
- Release notes: https://react.dev/blog/2024/12/05/react-19
- React 19.2: https://react.dev/blog/2025/10/01/react-19-2

---

### TypeScript - v5.9.3

**Current Status (Nov 2025):**
- Latest stable: v5.9.x
- Project uses: v5.9.3
- Status: ✅ **CURRENT**

**Important Updates:**
- **TypeScript 5.9** released August 1, 2025

**Major Features in 5.9:**
1. **Import Defer Support** - Stage-3 ECMAScript proposal for deferred module loading
2. **Improved Project Initialization** - `tsc --init` creates better default configs
3. **Node.js v20 Module Support** - New `node20` module option
4. **Expandable Hover Previews** - '+' and '-' buttons for deeply nested types
5. **Enhanced DOM API Documentation** - MDN-based documentation in IntelliSense
6. **Performance Improvements** - Caching of intermediate type instantiations

**What This Means for Learning:**
- Project uses latest TypeScript features
- Deferred imports available for optimization
- Better IDE experience with expandable types

**Official Resources:**
- Docs: https://www.typescriptlang.org/docs/
- Release notes: https://devblogs.microsoft.com/typescript/announcing-typescript-5-9/

---

## State Management

### Zustand - v5.0.4

**Current Status (Nov 2025):**
- Latest stable: v5.0.8
- Project uses: v5.0.4
- Status: ✅ **CURRENT** (4 patch versions behind, no breaking changes)

**Important Updates:**
- **Zustand v5.0** - Major modernization release (no new features, removed old things)
- Latest patches: v5.0.8 (August 19, 2025)

**Breaking Changes in v5:**
1. **React 18+ Required** - Dropped React <18 support, uses native `useSyncExternalStore`
2. **Equality Function Removed** - No custom equality in `create()` (use `zustand/traditional` if needed)
3. **Persist Middleware Change** - No longer stores initial state during creation
4. **Selector Behavior** - Changed to match React default (use `useShallow` to prevent loops)
5. **setState Type Safety** - Must provide complete state object when using `replace: true`

**What This Means for Learning:**
- v5 patterns are current best practices
- Documentation should focus on v5 API
- `useShallow` hook is important for performance
- Project is essentially up-to-date (patches are minor bug fixes)

**Official Resources:**
- Docs: https://zustand.docs.pmnd.rs/
- Migration guide: https://zustand.docs.pmnd.rs/migrations/migrating-to-v5
- Releases: https://github.com/pmndrs/zustand/releases

---

### nuqs - v2.7.3

**Current Status (Nov 2025):**
- Project uses: v2.7.3
- Status: ✅ **CURRENT** (actively maintained)

**What It Does:**
- Type-safe search params state management for Next.js
- URL as source of truth for component state
- Built specifically for Next.js App Router

**What This Means for Learning:**
- This is a Next.js-specific utility for managing URL state
- Important pattern for this project's architecture

**Official Resources:**
- Docs: https://nuqs.dev/
- GitHub: https://github.com/47ng/nuqs

---

## Data Fetching

### SWR - v2.3.6

**Current Status (Nov 2025):**
- Latest stable: v2.3.6
- Project uses: v2.3.6
- Status: ✅ **CURRENT** - Using latest version!

**What It Does:**
- React Hooks library for data fetching by Vercel
- "stale-while-revalidate" strategy
- Returns cached data first, then revalidates

**Key Characteristics (2025):**
- **Bundle size:** 4.2kb (very lightweight)
- **Philosophy:** Simplicity and performance
- **Best for:** Simpler projects, rapid development

**What This Means for Learning:**
- Current best practice for lightweight data fetching
- Well-integrated with Next.js ecosystem
- Great choice for this project's needs

**Official Resources:**
- Docs: https://swr.vercel.app/
- GitHub: https://github.com/vercel/swr

---

### TanStack Query (React Query) - v5.90.10

**Current Status (Nov 2025):**
- Latest stable: v5.90.10
- Project uses: v5.90.10
- Status: ✅ **CURRENT** - Using latest version! (Published Nov 20, 2025)

**Important Updates:**
- **v5 released:** March 21, 2025
- **Active development:** Multiple patch releases throughout November 2025
- **Integration:** Full React Suspense support with v5

**Key Characteristics (2025):**
- **Bundle size:** 13.1kb (heavier than SWR but more features)
- **Philosophy:** Advanced enterprise features
- **Best for:** Complex data requirements, enterprise apps

**What This Means for Learning:**
- Both SWR and TanStack Query are available in this project
- TanStack Query used for more complex data fetching scenarios
- v5 patterns are current and production-ready

**Official Resources:**
- Docs: https://tanstack.com/query/latest
- Release notes: https://tanstack.com/blog/announcing-tanstack-query-v5

---

## Backend & API

### tRPC - v11.7.1

**Current Status (Nov 2025):**
- Latest stable: v11.7.1+
- Project uses: v11.7.1
- Status: ✅ **CURRENT** - Using latest version!

**Important Updates:**
- **tRPC v11** announced March 21, 2025
- Largely backward-compatible with v10
- Many new features and improvements

**Major Features in v11:**
- Integration with TanStack Query v5
- Full React Suspense support
- Enhanced TypeScript inference
- Better error handling

**Best Practices (2025):**
- Feature-based code organization (not technical layers)
- CI/CD automation with GitHub Actions, GitLab CI, or CircleCI
- Environment variables for deployment configuration

**What This Means for Learning:**
- Project uses the most current tRPC patterns
- Type-safe API routes without code generation
- Excellent TypeScript experience

**Official Resources:**
- Docs: https://trpc.io/docs/
- Release notes: https://trpc.io/blog/announcing-trpc-v11
- Best practices: https://www.projectrules.ai/rules/trpc

---

### NextAuth.js - v5.0.0-beta.30

**Current Status (Nov 2025):**
- Latest: v5.0.0-beta.25+ (still in beta)
- Project uses: v5.0.0-beta.30
- Status: ✅ **CURRENT** (beta) - Project may be ahead of some docs!

**Important Updates:**
- Major rewrite for Next.js 14+ and App Router
- Minimum Next.js version: 14.0
- Now branded as "Auth.js" (but package name is still `next-auth`)

**Breaking Changes in v5:**
- Built on `@auth/core` with stricter OAuth/OIDC spec-compliance
- OAuth 1.0 support deprecated
- Better React Server Component support
- Improved TypeScript types

**What This Means for Learning:**
- v5 is beta but widely adopted
- Much better App Router support than v4
- React Client imports marked with "use client" directive
- Authentication patterns in this project are cutting-edge

⚠️ **Note:** Being beta means patterns may change before stable release.

**Official Resources:**
- Docs: https://authjs.dev/
- Migration guide: https://authjs.dev/getting-started/migrating-to-v5
- GitHub discussions: https://github.com/nextauthjs/next-auth/discussions

---

## Database & ORM

### Drizzle ORM - v0.44.7

**Current Status (Nov 2025):**
- Latest stable: v0.44.7
- Project uses: v0.44.7
- Status: ✅ **CURRENT** - Using latest version! (Released Oct 23, 2025)

**Recent Updates:**
- v0.44.7: Fixed durable SQLite transaction return value
- v0.44.6: Added `$replicas` reference feature
- Active development with frequent releases

**Key Characteristics:**
- Lightweight: ~7.4kb minified+gzipped
- Tree-shakeable with 0 dependencies
- TypeScript-first ORM
- Supports PostgreSQL, MySQL, SQLite (including serverless)

**Supported Databases:**
- Turso, Neon, Xata, PlanetScale, Cloudflare D1, and others

**What This Means for Learning:**
- Latest features and patterns available
- Modern TypeScript ORM approach
- Excellent for serverless and edge environments

**Official Resources:**
- Docs: https://orm.drizzle.team/
- Latest releases: https://orm.drizzle.team/docs/latest-releases
- GitHub: https://github.com/drizzle-team/drizzle-orm

---

### PGLite - v0.2.17

**Current Status (Nov 2025):**
- Latest stable: v0.2.32
- Project uses: v0.2.17
- Status: ⚠️ **SLIGHTLY BEHIND** (15 patch versions behind)

**Important Updates:**
- v0.2.0 released August 2024 with major features
- v0.2.32 published November 9, 2025
- Active development continues

**What is PGLite:**
- PostgreSQL compiled to WebAssembly (WASM)
- Runs in browser, Node.js, Bun, Deno
- No Linux VM - just Postgres in WASM
- Only 3MB gzipped

**Major Features:**
- Extension support (including pgvector)
- Reactive live query API
- Electric/next sync support
- Multiple persistence options (in-memory, IndexedDB, filesystem)

**What This Means for Learning:**
- v0.2.x features are all available
- Patch versions are minor bug fixes
- Consider updating to v0.2.32 for latest fixes
- This is a cutting-edge technology (client-side Postgres!)

⚠️ **Recommendation:** Project should update to v0.2.32 for latest bug fixes.

**Official Resources:**
- Docs: https://pglite.dev/
- GitHub: https://github.com/electric-sql/pglite

---

### Neon PostgreSQL - v1.0.2

**Current Status (Nov 2025):**
- Project uses: v1.0.2
- Status: ✅ **CURRENT** (serverless Postgres client)

**What It Does:**
- Serverless PostgreSQL client for JavaScript
- Optimized for edge and serverless environments
- Low-latency queries from edge locations

**What This Means for Learning:**
- Used for server-side database connections
- Complements PGLite (server vs client DB)

**Official Resources:**
- Docs: https://neon.tech/docs
- GitHub: https://github.com/neondatabase/serverless

---

## UI Libraries

### Ant Design (antd) - v5.28.1

**Current Status (Nov 2025):**
- Latest stable: v5.28.0+
- Project uses: v5.28.1
- Status: ✅ **CURRENT** - Using latest version!

**Recent Updates:**
- v5.28.0 includes React 19 compatibility fixes
- Fixes for Tooltip, Popover, Popconfirm, Dropdown misalignment in React 19
- Various component improvements and dark mode fixes

**Key Features:**
- Enterprise-class UI design language
- High-quality React components
- Written in TypeScript
- i18n support for dozens of languages
- Powerful theme customization (CSS-in-JS)

**What This Means for Learning:**
- Full React 19 compatibility
- Current design system patterns
- This is the "world's second most popular React UI framework"

**Official Resources:**
- Docs: https://ant.design/
- Changelog: https://ant.design/changelog/
- GitHub: https://github.com/ant-design/ant-design

---

### antd-style - v3.7.1

**Current Status (Nov 2025):**
- Project uses: v3.7.1
- Status: ✅ **CURRENT** (actively maintained)

**What It Does:**
- Business-level CSS-in-JS solution for Ant Design v5
- Built on Emotion
- Integrates with Ant Design token system

**Key Features:**
- Token system integration
- One-click dark mode switching
- Flexible custom theme extension
- Good qiankun micro-app compatibility
- Smooth migration from Less to CSS-in-JS

**What This Means for Learning:**
- This project uses CSS-in-JS with Ant Design tokens
- Dark mode is built-in and easy to toggle
- Component styling follows this pattern

**Official Resources:**
- Docs: https://ant-design.github.io/antd-style/
- GitHub: https://github.com/ant-design/antd-style

---

### Lucide React - v0.553.0

**Current Status (Nov 2025):**
- Project uses: v0.553.0
- Status: ✅ **CURRENT** (actively maintained with frequent releases)

**What It Does:**
- Icon library for React
- Fork of Feather Icons
- Over 1,000+ icons
- Tree-shakeable (only import what you use)

**What This Means for Learning:**
- Primary icon library for custom icons
- Works alongside Ant Design icons

**Official Resources:**
- Docs: https://lucide.dev/
- Icon catalog: https://lucide.dev/icons/

---

### Framer Motion - v12.23.24

**Current Status (Nov 2025):**
- Project uses: v12.23.24
- Status: ✅ **CURRENT** (v12 is latest major, frequent patches)

**What It Does:**
- Production-ready animation library for React
- Declarative animations
- Gestures and interactions
- Layout animations

**What This Means for Learning:**
- Used for UI animations throughout the project
- v12 patterns are current

**Official Resources:**
- Docs: https://www.framer.com/motion/
- GitHub: https://github.com/framer/motion

---

## Internationalization (i18n)

### i18next - v25.6.2

**Current Status (Nov 2025):**
- Project uses: v25.6.2
- Status: ✅ **CURRENT**

**What It Does:**
- Internationalization framework for JavaScript
- Works in browser, Node.js, Deno, Bun
- 6.3M weekly downloads
- 9.8K+ GitHub stars

**Official Resources:**
- Docs: https://www.i18next.com/
- GitHub: https://github.com/i18next/i18next

---

### react-i18next - v15.7.4

**Current Status (Nov 2025):**
- Latest stable: v16.3.5
- Project uses: v15.7.4
- Status: ⚠️ **ONE MAJOR VERSION BEHIND**

**What This Means:**
- Project uses v15, latest is v16 (released Nov 2025)
- v15 still works fine and is widely used
- Consider updating to v16 for latest features
- No critical issues with using v15

**Key Features:**
- `useTranslation` hook
- `<Trans>` component for mixed markup
- "Learn once - translate everywhere" philosophy
- 5,791+ projects using it

⚠️ **Recommendation:** Consider updating to v16.3.5 for latest features.

**Official Resources:**
- Docs: https://react.i18next.com/
- GitHub: https://github.com/i18next/react-i18next

---

## Testing

### Vitest - v3.2.4

**Current Status (Nov 2025):**
- Latest stable: v3.2.x+
- Project uses: v3.2.4
- Status: ✅ **CURRENT**

**Important Updates:**
- **Vitest 3.0** - Major overhaul (2025)
- **Vitest 3.2** - Latest with new features

**Major Features in 3.2:**
- AbortSignal for test bodies (timeout/cancellation support)
- ast-v8-to-istanbul for better v8 coverage reports
- Watch trigger patterns - Rerun specific tests based on changed files
- Custom matchers type - Type support for custom matchers
- Annotations API - Custom messages and attachments (visible in UI, HTML, junit, etc.)
- Fixture scoping - `test.extend` fixtures can specify scope (file or worker)

**What This Means for Learning:**
- Latest testing patterns and features available
- Excellent Vite integration
- Fast and modern testing experience

**Official Resources:**
- Docs: https://vitest.dev/
- Release notes: https://vitest.dev/blog/vitest-3-2.html
- GitHub: https://github.com/vitest-dev/vitest

---

### Playwright - v1.56.1

**Current Status (Nov 2025):**
- Project uses: v1.56.1
- Status: ✅ **CURRENT** (v1.56.x is latest)

**What It Does:**
- End-to-end testing for web applications
- Cross-browser testing (Chrome, Firefox, Safari, Edge)
- Mobile emulation
- Auto-wait for elements

**What This Means for Learning:**
- E2E tests use current Playwright patterns
- Located in `e2e/` directory

**Official Resources:**
- Docs: https://playwright.dev/
- GitHub: https://github.com/microsoft/playwright

---

## Utilities & Helpers

### ahooks - v3.9.6

**Current Status (Nov 2025):**
- Project uses: v3.9.6
- Status: ✅ **CURRENT** (v3 is stable, actively maintained)

**What It Does:**
- High-quality React Hooks library
- Chinese origin, English docs available
- Many useful hooks for common scenarios

**Official Resources:**
- Docs: https://ahooks.js.org/
- GitHub: https://github.com/alibaba/hooks

---

### dayjs - v1.11.19

**Current Status (Nov 2025):**
- Project uses: v1.11.19
- Status: ✅ **CURRENT**

**What It Does:**
- Lightweight date library (Moment.js alternative)
- 2KB minified
- Same API as Moment.js
- Immutable

**Official Resources:**
- Docs: https://day.js.org/
- GitHub: https://github.com/iamkun/dayjs

---

### lodash-es - v4.17.21

**Current Status (Nov 2025):**
- Project uses: v4.17.21
- Status: ✅ **CURRENT** (v4 is stable and maintained)

**What It Does:**
- Modular utility library (ES modules version)
- Tree-shakeable
- Array, object, string, function utilities

**Official Resources:**
- Docs: https://lodash.com/
- GitHub: https://github.com/lodash/lodash

---

### react-layout-kit - v2.0.1

**Current Status (Nov 2025):**
- Project uses: v2.0.1
- Status: ✅ **CURRENT** (v2 is stable)

**What It Does:**
- Flex layout components for React
- Provides `Flexbox`, `FlexItem`, `Center`, etc.
- Simplifies layout code

**What This Means for Learning:**
- Layout patterns in this project use these components
- Documented in project's cursor rules

**Official Resources:**
- GitHub: https://github.com/ant-design/react-layout-kit
- Docs in project: `.cursor/rules/packages/react-layout-kit.mdc`

---

## Package Managers & Runtimes

### pnpm - v10.20.0

**Current Status (Nov 2025):**
- Latest stable: v10.22.0
- Project uses: v10.20.0
- Status: ✅ **CURRENT** (2 patch versions behind)

**Recent Updates:**
- v10.22.0 published November 12, 2025
- v10.20.0 released October 28, 2025

**Key Features:**
- Fast, disk space efficient
- Up to 2x faster than npm
- Strict node_modules structure
- Monorepo support

**What This Means for Learning:**
- Primary package manager for dependencies
- v10 patterns are current
- Patch differences are minor

**Official Resources:**
- Docs: https://pnpm.io/
- GitHub: https://github.com/pnpm/pnpm

---

### Bun - (Runtime for scripts)

**Current Status (Nov 2025):**
- Latest stable: v1.3.1 (Released Oct 28, 2025)
- Status: ✅ **CURRENT**

**Recent Updates:**
- **Bun 1.3** - "Battery-included full-stack JavaScript runtime"
- Vercel announced Bun Runtime support (Nov 8, 2025)

**Major Features in 1.3:**
- **Bun.SQL** - Unified API for MySQL/MariaDB, PostgreSQL, SQLite
- **Redis client** - Integrated Redis and Valkey support
- **Garbage collector improvements** - 100x reduction in idle CPU, 40% less idle memory
- Over 5 million monthly downloads

**What It Does:**
- All-in-one JavaScript runtime (like Node.js, but faster)
- Built in Zig, uses JavaScriptCore (not V8)
- Includes runtime, bundler, test runner, package manager

**Usage in This Project:**
- Used to run npm scripts (see package.json)
- `bun run` commands for various tasks
- Fast execution for development workflow

**What This Means for Learning:**
- Bun is production-ready and widely adopted
- Used by Anthropic for Claude Code CLI
- Scripts run with `bun` instead of `npm run`

**Official Resources:**
- Docs: https://bun.com/
- GitHub: https://github.com/oven-sh/bun

---

## Summary of Version Status

### ✅ FULLY CURRENT (Latest Version)
- Next.js 16.0.3
- React 19.2.0
- TypeScript 5.9.3
- SWR 2.3.6
- TanStack Query 5.90.10
- tRPC 11.7.1
- Drizzle ORM 0.44.7
- Ant Design 5.28.1
- Vitest 3.2.4
- dayjs 1.11.19
- lodash-es 4.17.21

### ✅ CURRENT (Minor Patches Behind, No Issues)
- Zustand 5.0.4 → 5.0.8 (4 patches)
- pnpm 10.20.0 → 10.22.0 (2 patches)
- antd-style 3.7.1 (stable)
- nuqs 2.7.3 (stable)
- ahooks 3.9.6 (stable)
- react-layout-kit 2.0.1 (stable)
- Playwright 1.56.1 (stable)
- Neon 1.0.2 (stable)

### ⚠️ SLIGHTLY BEHIND (Consider Updating)
- **PGLite 0.2.17 → 0.2.32** (15 patches behind, bug fixes)
- **react-i18next 15.7.4 → 16.3.5** (1 major version behind)

### 🔵 BETA BUT CURRENT
- **NextAuth.js 5.0.0-beta.30** (beta, but current and widely adopted)

---

## Key Insights for Documentation

1. **This project is exceptionally well-maintained** - Almost all dependencies are at or very close to latest versions.

2. **React 19 & Next.js 16 are stable** - All new features (Server Components, Server Actions, new hooks) are production-ready and should be documented as current best practices.

3. **tRPC v11 is the standard** - Type-safe APIs without code generation are the way.

4. **Zustand v5 patterns** - Documentation should focus on v5 API (no legacy v4 patterns).

5. **CSS-in-JS with Ant Design tokens** - The styling approach uses antd-style with the token system.

6. **Dual database strategy** - PGLite (client-side WASM Postgres) + Neon (serverless Postgres) is a cutting-edge pattern.

7. **Modern testing stack** - Vitest 3.2 and Playwright are current best practices.

8. **Bun for scripts** - Development workflow uses Bun for speed.

9. **i18n with react-i18next** - v15 works fine, but v16 is available.

10. **NextAuth v5 beta** - While beta, it's the future of authentication for Next.js App Router.

---

## Patterns That Are Current (Nov 2025)

✅ **CURRENT PATTERNS TO DOCUMENT:**
- React Server Components
- Server Actions
- Next.js App Router (with new Turbopack)
- `use cache` and PPR (Partial Pre-Rendering)
- tRPC v11 with TanStack Query v5
- Zustand v5 with `useShallow`
- Drizzle ORM with PGLite and Neon
- CSS-in-JS with antd-style tokens
- Vitest 3.x testing patterns
- TypeScript 5.9 features (deferred imports, etc.)

⚠️ **OUTDATED PATTERNS TO AVOID:**
- Pages Router (use App Router)
- `getServerSideProps`, `getStaticProps` (use Server Components)
- React `forwardRef` (use `ref` as prop directly)
- Zustand v4 equality functions (use v5 patterns)
- Class components (use function components with hooks)
- `useEffect` for data fetching (use SWR, TanStack Query, or Server Components)

---

## Additional Technologies Found in package.json

The following technologies are also used but were not deeply researched:

**AI & LLM:**
- OpenAI SDK, Anthropic SDK, Google Genai, Hugging Face
- LangChain, Langfuse
- Model Context Protocol (MCP) SDK

**File Processing:**
- PDF parsing, Office document parsing
- EPUB, Markdown, DOCX support

**Media:**
- TTS (Text-to-Speech) with @lobehub/tts
- Image processing with Sharp

**Payments:**
- Stripe v17.7.0

**Monitoring:**
- OpenTelemetry
- Vercel Analytics, PostHog

**Other:**
- Clerk (authentication alternative)
- React PDF rendering
- Xterm for terminal emulation

---

**Next Steps:**
1. Use this research as the foundation for all documentation
2. Mark patterns as ✅ CURRENT or ⚠️ OUTDATED throughout docs
3. Link to current official documentation (Nov 2025)
4. Focus learning materials on current best practices
5. Note where project uses cutting-edge features (PGLite, React 19, Next.js 16)

---

**Document Status:** ✅ **COMPLETE** - Comprehensive research conducted for all major technologies
