# Tech Stack Guide - Part 4 of 4

**Documented:** November 2025
**Tech Stack Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)
**Target:** Mid-level developers proficient in JS/TS/React

**📄 This is Part 4 of 4** | **[← Part 3](./TECH-STACK-GUIDE-PART-03.md)**

This final part covers testing frameworks, build tools, and infrastructure.

**Prerequisites:**
- **[Part 1](./TECH-STACK-GUIDE-PART-01.md)** - Frontend core technologies
- **[Part 2](./TECH-STACK-GUIDE-PART-02.md)** - State management and data fetching
- **[Part 3](./TECH-STACK-GUIDE-PART-03.md)** - Backend APIs and database

**Time to read:** 30 minutes (Part 4)

---

## Table of Contents - Part 4

1. [Testing](#testing)
   - [Vitest 3.2.4](#vitest-324)
   - [Playwright 1.56.0](#playwright-1560)
2. [Build Tools](#build-tools)
   - [Turbopack (Stable)](#turbopack-stable)
   - [Bun 1.3.1](#bun-131)
   - [pnpm 9.15.4](#pnpm-9154)
3. [Infrastructure & Deployment](#infrastructure--deployment)
   - [Vercel](#vercel)
   - [Docker](#docker)
4. [Development Tools](#development-tools)
   - [ESLint 9.x](#eslint-9x)
   - [Prettier 3.x](#prettier-3x)
5. [Summary](#summary)

**[← Part 3](./TECH-STACK-GUIDE-PART-03.md)** covered:
- Backend APIs (tRPC, Next.js API Routes)
- Database (Drizzle ORM, PGLite, Neon PostgreSQL)

---

## Testing

### Vitest 3.2.4

**✅ CURRENT** | **Released:** November 2024 | **Vite-powered:** Yes

**What is it?**
Vitest is a blazing-fast unit test framework powered by Vite.

**Why Vitest?**

**Alternatives considered:**
- Jest: Popular, but slow configuration and setup
- Mocha/Chai: Good, but manual setup required
- Jasmine: Good, but smaller ecosystem

**Vitest wins:**
- ✅ **Blazing fast** - Powered by Vite/esbuild
- ✅ **Jest-compatible API** - Easy migration from Jest
- ✅ **TypeScript support** - Zero config for TS
- ✅ **ESM support** - Native ES modules
- ✅ **Watch mode** - Instant feedback

---

**Basic Usage:**

```typescript
// src/utils/math.ts
export function add(a: number, b: number) {
  return a + b;
}

// src/utils/math.test.ts
import { describe, it, expect } from 'vitest';
import { add } from './math';

describe('math utils', () => {
  it('adds numbers correctly', () => {
    expect(add(1, 2)).toBe(3);
    expect(add(-1, 1)).toBe(0);
  });
});
```

**🌉 Bridge from Jest:** Same API as Jest! `describe`, `it`, `expect` work the same.

---

**React Component Testing:**

```typescript
import { render, screen } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import { Button } from './Button';

describe('Button component', () => {
  it('renders button text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('calls onClick when clicked', async () => {
    const onClick = vi.fn();
    render(<Button onClick={onClick}>Click me</Button>);

    await screen.getByText('Click me').click();
    expect(onClick).toHaveBeenCalledOnce();
  });
});
```

---

**Mocking:**

```typescript
import { vi, describe, it, expect } from 'vitest';

// Mock function
const mockFn = vi.fn();
mockFn('hello');
expect(mockFn).toHaveBeenCalledWith('hello');

// Mock module
vi.mock('./database', () => ({
  db: {
    query: vi.fn(() => Promise.resolve({ id: 1, name: 'Alice' })),
  },
}));

// Mock implementation
const getUserMock = vi.fn().mockImplementation((id) => {
  if (id === 1) return { id: 1, name: 'Alice' };
  return null;
});
```

---

**Snapshot Testing:**

```typescript
import { render } from '@testing-library/react';
import { it, expect } from 'vitest';
import { UserCard } from './UserCard';

it('matches snapshot', () => {
  const { container } = render(
    <UserCard name="Alice" email="alice@example.com" />
  );
  expect(container).toMatchSnapshot();
});
```

---

**Coverage:**

```bash
# Run tests with coverage
bunx vitest run --coverage

# Coverage report:
# statements: 85%
# branches: 80%
# functions: 90%
# lines: 85%
```

---

**LobeHub Testing Conventions:**

```bash
# Run specific test file (REQUIRED FORMAT)
bunx vitest run --silent='passed-only' 'src/components/Button/index.test.tsx'

# Web tests
bunx vitest run --silent='passed-only' '[file-path-pattern]'

# Package tests (e.g., database)
cd packages/database && bunx vitest run --silent='passed-only' '[file-path-pattern]'
```

**🚨 IMPORTANT:**
- Wrap file path in single quotes!
- NEVER run `bun run test` (runs ALL tests, takes 10+ minutes)
- Use `--silent='passed-only'` to reduce noise

📚 **More details:** [.cursor/rules/testing-guide/testing-guide.mdc](./.cursor/rules/testing-guide/testing-guide.mdc)

---

**Learning Resources:**

- [Vitest Documentation](https://vitest.dev/)
- [Vitest API Reference](https://vitest.dev/api/)
- [Testing Library](https://testing-library.com/)

---

### Playwright 1.56.0

**✅ CURRENT** | **Released:** November 2024 | **Cross-browser:** Yes

**What is it?**
Playwright is an end-to-end (E2E) testing framework for web applications.

**Why Playwright?**

**Alternatives considered:**
- Cypress: Popular, but slower and more limited
- Puppeteer: Good, but Chrome-only
- Selenium: Old, complex setup

**Playwright wins:**
- ✅ **Cross-browser** - Chrome, Firefox, Safari (WebKit)
- ✅ **Fast execution** - Parallel by default
- ✅ **Auto-wait** - Waits for elements automatically
- ✅ **Network interception** - Mock API responses
- ✅ **TypeScript support** - First-class TS support

---

**Basic Usage:**

```typescript
// e2e/src/login.spec.ts
import { test, expect } from '@playwright/test';

test('user can log in', async ({ page }) => {
  // Navigate to login page
  await page.goto('http://localhost:3000/login');

  // Fill form
  await page.fill('input[name="email"]', 'alice@example.com');
  await page.fill('input[name="password"]', 'password123');

  // Click submit
  await page.click('button[type="submit"]');

  // Assert redirected to dashboard
  await expect(page).toHaveURL('http://localhost:3000/dashboard');

  // Assert welcome message appears
  await expect(page.locator('text=Welcome, Alice')).toBeVisible();
});
```

---

**Key Features:**

**1. Auto-waiting**

Playwright automatically waits for elements:

```typescript
// ❌ Other frameworks: Manual waits
await driver.wait(until.elementLocated(By.id('button')), 5000);
await driver.findElement(By.id('button')).click();

// ✅ Playwright: Automatic waiting
await page.click('#button');
// Playwright automatically:
// - Waits for element to exist
// - Waits for element to be visible
// - Waits for element to be enabled
// - Clicks when ready
```

---

**2. Network Interception**

Mock API responses:

```typescript
test('shows error when API fails', async ({ page }) => {
  // Intercept API call
  await page.route('**/api/users', (route) => {
    route.fulfill({
      status: 500,
      body: JSON.stringify({ error: 'Server error' }),
    });
  });

  await page.goto('/users');

  // Assert error message appears
  await expect(page.locator('text=Failed to load users')).toBeVisible();
});
```

---

**3. Multiple Browsers**

Test across Chrome, Firefox, Safari:

```typescript
// playwright.config.ts
export default {
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
  ],
};

// Run tests:
bunx playwright test  // Runs on all browsers in parallel
```

---

**4. Mobile Testing**

Test mobile viewports:

```typescript
test('mobile menu works', async ({ page }) => {
  // Emulate iPhone 13
  await page.setViewportSize({ width: 390, height: 844 });

  await page.goto('/');

  // Mobile menu should be visible
  await expect(page.locator('[data-testid="mobile-menu"]')).toBeVisible();

  // Desktop menu should be hidden
  await expect(page.locator('[data-testid="desktop-menu"]')).toBeHidden();
});
```

---

**5. Screenshots & Videos**

Automatic failure screenshots:

```typescript
// playwright.config.ts
export default {
  use: {
    screenshot: 'only-on-failure',  // Screenshot on failure
    video: 'retain-on-failure',     // Video on failure
  },
};

// Manual screenshots:
await page.screenshot({ path: 'screenshot.png' });
await page.screenshot({ path: 'full-page.png', fullPage: true });
```

---

**Learning Resources:**

- [Playwright Documentation](https://playwright.dev/)
- [Playwright Best Practices](https://playwright.dev/docs/best-practices)

📚 **More details:** [TESTING_GUIDE.md](./TESTING_GUIDE.md)

---

## Build Tools

### Turbopack (Stable)

**✅ CURRENT** | **Part of:** Next.js 16 | **Production-ready:** Yes

**What is it?**
Turbopack is Next.js's new Rust-based bundler, replacing Webpack.

**Why Turbopack?**

**Problem with Webpack:**
- Slow cold starts (10+ seconds)
- Slow hot reloads (1-2 seconds)
- Complex configuration

**Solution: Turbopack**
- ✅ **10x faster** - Rust-based, highly optimized
- ✅ **<1s cold start** - Incremental compilation
- ✅ **<200ms HMR** - Blazing fast hot module reload
- ✅ **Zero config** - Works out of the box

---

**Performance Comparison:**

| Metric | Webpack | Turbopack |
|--------|---------|-----------|
| **Cold start** | ~10s | ~1s |
| **Hot reload** | ~2s | <200ms |
| **Production build** | ~60s | ~30s |

**🆕 NEW IN 2025:** Turbopack is now **stable** and production-ready in Next.js 16!

---

**Usage:**

Turbopack is enabled by default in Next.js 16:

```bash
# Development (uses Turbopack automatically)
bun run dev

# Production build (uses Turbopack)
bun run build
```

**No configuration needed!**

---

**Learning Resources:**

- [Turbopack Documentation](https://nextjs.org/docs/architecture/turbopack)
- [Turbopack Performance](https://turbo.build/pack/docs/benchmarks)

---

### Bun 1.3.1

**✅ CURRENT** | **Released:** November 2024 | **All-in-one:** Yes

**What is it?**
Bun is an all-in-one JavaScript runtime, bundler, test runner, and package manager.

**Why Bun?**

**LobeHub uses Bun for:**
- ✅ Running scripts (`bun run dev`, `bun run build`)
- ✅ Running executables (`bunx vitest`, `bunx playwright`)
- ❌ NOT for package management (uses pnpm instead)

**Why Bun for scripts?**

| Tool | Script Startup | Speed |
|------|----------------|-------|
| **Node.js** | ~100ms | Baseline |
| **Bun** | ~10ms | **10x faster** |

---

**Usage:**

```bash
# Run scripts (RECOMMENDED)
bun run dev          # Start dev server
bun run build        # Build for production
bun run test         # Run tests

# Run executables (RECOMMENDED)
bunx vitest run      # Run Vitest
bunx playwright test # Run Playwright

# NOT used for package management:
# ❌ bun install      (use pnpm install)
# ❌ bun add          (use pnpm add)
```

**🎯 LobeHub convention:** `bun` for running, `pnpm` for package management

---

**Learning Resources:**

- [Bun Documentation](https://bun.sh/docs)
- [Bun Runtime](https://bun.sh/docs/runtime)

---

### pnpm 9.15.4

**✅ CURRENT** | **Released:** November 2024 | **Monorepo:** Yes

**What is it?**
pnpm is a fast, disk-space-efficient package manager with monorepo support.

**Why pnpm?**

**Alternatives considered:**
- npm: Slow, disk space inefficient
- Yarn: Good, but pnpm is faster
- Bun: Good, but less mature for monorepos

**pnpm wins:**
- ✅ **Disk efficient** - Symlinks to global store (saves GB)
- ✅ **Fast** - Faster than npm/yarn
- ✅ **Strict** - No phantom dependencies
- ✅ **Monorepo support** - Built-in workspaces

---

**Disk Space Efficiency:**

```
npm:  node_modules/  ~500 MB (per project)
pnpm: node_modules/  ~50 MB (symlinks to shared store!)
```

**🎯 Benefit:** 10+ projects = ~5GB saved!

---

**Monorepo Workspaces:**

```yaml
# pnpm-workspace.yaml
packages:
  - 'packages/*'
  - 'apps/*'
```

**Commands:**

```bash
# Install all workspace dependencies
pnpm install

# Add dependency to root
pnpm add -w <package>

# Add dependency to specific workspace
pnpm add <package> --filter @lobechat/database

# Run script in specific workspace
pnpm --filter @lobechat/database test

# Run script in all workspaces
pnpm -r test
```

---

**LobeHub Package Management:**

```bash
# Install dependencies
pnpm install

# Add package to workspace
pnpm add <package> --filter <workspace-name>

# Update all dependencies
pnpm up -r

# Check for outdated packages
pnpm outdated
```

---

**Learning Resources:**

- [pnpm Documentation](https://pnpm.io/)
- [pnpm Workspaces](https://pnpm.io/workspaces)

---

## Infrastructure & Deployment

### Vercel

**✅ RECOMMENDED** | **Next.js Optimized:** Yes | **Edge Network:** Yes

**What is it?**
Vercel is a cloud platform for deploying Next.js applications with zero configuration.

**Why Vercel?**

**Alternatives considered:**
- AWS: Powerful, but complex setup
- Netlify: Good, but not Next.js optimized
- Cloudflare Pages: Good, but limited features

**Vercel wins:**
- ✅ **Zero config** - Deploy with `git push`
- ✅ **Next.js optimized** - Made by Next.js creators
- ✅ **Global edge network** - Low latency worldwide
- ✅ **Preview deployments** - Automatic PR previews
- ✅ **Serverless functions** - Auto-scaling

---

**Deployment:**

```bash
# 1. Connect GitHub repo
# 2. Import project to Vercel
# 3. Push to main branch
git push origin main

# Vercel automatically:
# - Builds the app
# - Deploys to production
# - Updates DNS
```

**That's it!** No configuration needed.

---

**Environment Variables:**

```bash
# Add via Vercel Dashboard or CLI
vercel env add DATABASE_URL production
vercel env add NEXT_AUTH_SECRET production
```

**Automatic injection** - Available in `process.env`

---

**Preview Deployments:**

Every pull request gets an automatic preview URL:

```
Pull Request #123
→ https://lobe-chat-git-pr-123-team.vercel.app

- Live preview
- Isolated from production
- Shareable link
```

**🎯 Benefit:** Test changes before merging!

---

**Learning Resources:**

- [Vercel Documentation](https://vercel.com/docs)
- [Vercel with Next.js](https://vercel.com/docs/frameworks/nextjs)

---

### Docker

**✅ SUPPORTED** | **Self-hosting:** Yes

**What is it?**
Docker allows packaging LobeHub as a container for self-hosting.

**Why Docker for self-hosting?**

- ✅ **Consistent environment** - Same on all machines
- ✅ **Easy deployment** - Single command to run
- ✅ **Isolated** - No dependency conflicts
- ✅ **Portable** - Works on any Docker host

---

**Usage:**

```bash
# Build Docker image
docker build -t lobechat .

# Run container
docker run -p 3000:3000 \
  -e DATABASE_URL=postgresql://... \
  -e NEXT_AUTH_SECRET=... \
  lobechat

# Or use Docker Compose:
docker compose up
```

**`docker-compose.yml` example:**

```yaml
version: '3.8'

services:
  lobechat:
    build: .
    ports:
      - '3000:3000'
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/lobechat
      - NEXT_AUTH_SECRET=${NEXT_AUTH_SECRET}
    depends_on:
      - db

  db:
    image: postgres:17
    environment:
      - POSTGRES_DB=lobechat
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

---

**Multi-stage Build:**

LobeHub uses multi-stage builds for smaller images:

```dockerfile
# Stage 1: Dependencies
FROM node:18-alpine AS deps
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile

# Stage 2: Build
FROM node:18-alpine AS builder
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN pnpm run build

# Stage 3: Production
FROM node:18-alpine AS runner
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/public ./public
CMD ["node", "server.js"]
```

**🎯 Result:** ~200MB final image (vs ~2GB without multi-stage)

---

**Learning Resources:**

- [Docker Documentation](https://docs.docker.com/)
- [LobeHub Self-Hosting](../self-hosting/)

📚 **More details:** [docs/self-hosting/](../self-hosting/)

---

## Development Tools

### ESLint 9.x

**✅ CURRENT** | **Linting:** Yes | **Flat Config:** Yes

**What is it?**
ESLint finds and fixes problems in JavaScript/TypeScript code.

**LobeHub ESLint Configuration:**

```javascript
// .eslintrc.js
module.exports = {
  extends: [
    'next/core-web-vitals',     // Next.js rules
    'plugin:@typescript-eslint/recommended',  // TypeScript rules
    'prettier',                 // Prettier compatibility
  ],
  rules: {
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/no-unused-vars': 'warn',
    'no-console': 'warn',
  },
};
```

**Run ESLint:**

```bash
# Check for errors
bun run lint

# Fix auto-fixable errors
bun run lint:fix
```

---

### Prettier 3.x

**✅ CURRENT** | **Formatting:** Yes

**What is it?**
Prettier automatically formats code for consistency.

**LobeHub Prettier Configuration:**

```json
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "printWidth": 100,
  "trailingComma": "es5"
}
```

**Run Prettier:**

```bash
# Check formatting
bun run format:check

# Fix formatting
bun run format
```

---

## Summary

### Complete Tech Stack Overview

**Frontend (Part 1):**
- React 19.2 - Server Components, React Compiler
- Next.js 16.0 - Turbopack, App Router, PPR
- TypeScript 5.9 - Type safety
- Ant Design 5.23 - UI components
- antd-style 4.0 - CSS-in-JS with theme tokens

**State & Data (Part 2):**
- Zustand 5.0 - State management
- nuqs 2.2 - URL query state
- SWR 2.3 - Simple data fetching
- TanStack Query 5.76 - Advanced data management
- react-i18next 15.1 - Internationalization

**Backend & Database (Part 3):**
- tRPC 11.7 - Type-safe APIs
- Drizzle ORM 0.44 - TypeScript-first ORM
- PGLite 0.2 - WASM PostgreSQL (client)
- Neon - Serverless PostgreSQL (server)

**Testing & Tools (Part 4):**
- Vitest 3.2 - Unit testing
- Playwright 1.56 - E2E testing
- Turbopack - Fast bundler (stable!)
- Bun 1.3 - Script runner
- pnpm 9.15 - Package manager
- Vercel - Deployment platform

---

### Technology Status (November 2025)

**✅ ALL TECHNOLOGIES ARE:**
- Current and actively maintained
- Production-ready
- At latest stable versions
- Following 2025 best practices

**🚀 INNOVATIONS:**
1. WASM PostgreSQL in browser (PGLite) - **Industry first!**
2. Dual database strategy (offline-first)
3. End-to-end type safety (tRPC + Drizzle)
4. Automatic React optimization (React Compiler)
5. Stable Turbopack in production

---

### Next Steps

Now that you understand the tech stack:

1. **[Patterns & Conventions](./PATTERNS_AND_CONVENTIONS.md)** (1 hr)
   - Code style guide
   - Best practices
   - Current vs outdated patterns

2. **[How-To Guide](./HOW_TO_GUIDE.md)** (1.5 hrs)
   - Add a new page
   - Create a component
   - Add a database table
   - Common tasks

3. **[Testing Guide](./TESTING_GUIDE.md)** (1.5 hrs)
   - Writing unit tests
   - Writing E2E tests
   - Testing best practices

---

**Document Status:** ✅ Complete (Part 4 of 4) | **Last Updated:** November 20, 2025

**[← Part 1](./TECH-STACK-GUIDE-PART-01.md)** | **[← Part 2](./TECH-STACK-GUIDE-PART-02.md)** | **[← Part 3](./TECH-STACK-GUIDE-PART-03.md)**

**Next:** [Patterns & Conventions](./PATTERNS_AND_CONVENTIONS.md)
