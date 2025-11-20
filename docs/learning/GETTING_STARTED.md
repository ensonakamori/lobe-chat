# Getting Started with LobeHub

**Documented:** November 2025
**Tech Stack Research:** [TECH_STACK_RESEARCH.md](./TECH_STACK_RESEARCH.md)
**Target:** Mid-level developers proficient in JS/TS/React

This guide will help you install, setup, and run LobeHub for local development in ~30 minutes.

**Prerequisites:**
- Basic understanding of JavaScript/TypeScript and React
- Familiarity with command line/terminal
- Git installed on your system

---

## Table of Contents

1. [System Requirements](#system-requirements)
2. [Quick Start (5 minutes)](#quick-start-5-minutes)
3. [Full Setup (15-20 minutes)](#full-setup-15-20-minutes)
4. [Verifying Your Setup](#verifying-your-setup)
5. [Running Tests](#running-tests)
6. [Common Issues & Solutions](#common-issues--solutions)
7. [Development Tools](#development-tools)
8. [Next Steps](#next-steps)

---

## System Requirements

### Required Software

| Software | Version | Purpose |
|----------|---------|---------|
| **Node.js** | 18.x LTS (Krypton) or higher | JavaScript runtime |
| **pnpm** | 10.x or higher | Package manager (dependency management) |
| **Bun** | 1.3.x or higher | Script runner (fast execution) |
| **Git** | Any recent version | Version control |

### Recommended (Optional)

| Software | Purpose |
|----------|---------|
| **PostgreSQL** | For server-side database (optional for basic dev) |
| **Docker** | For running full stack with services |
| **nvm** or **fnm** | Node.js version management |

### Operating System

✅ **macOS** - Fully supported
✅ **Linux** - Fully supported
✅ **Windows** - Supported (via WSL2 recommended)

### Hardware

- **RAM:** 8GB minimum, 16GB recommended
- **Storage:** 5GB free space
- **CPU:** Modern multi-core processor

---

## Quick Start (5 minutes)

**Goal:** Get the app running locally with minimal setup

### Step 1: Install Node.js

**Using nvm (recommended):**

```bash
# Install nvm (if not installed)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Install Node.js 18 LTS
nvm install lts/krypton
nvm use lts/krypton

# Verify
node -v  # Should show v18.x.x
```

**Or download directly:** https://nodejs.org/ (choose 18.x LTS)

---

### Step 2: Install pnpm

```bash
# Install pnpm globally
npm install -g pnpm@10

# Verify
pnpm -v  # Should show 10.x.x
```

**🌉 Bridge from npm:** pnpm works like npm, but faster and more efficient

---

### Step 3: Install Bun

```bash
# Install Bun
curl -fsSL https://bun.sh/install | bash

# Verify
bun -v  # Should show 1.x.x
```

**🧠 Mental Model:** Bun is like Node.js but faster - used here to run scripts

---

### Step 4: Clone the Repository

```bash
# Clone the repo
git clone https://github.com/lobehub/lobe-chat.git

# Navigate to the project
cd lobe-chat
```

**🎯 Remember:** We're on the `next` branch (v2.x development)

---

### Step 5: Install Dependencies

```bash
# Install all dependencies (this may take 2-5 minutes)
pnpm install
```

**What this does:**
- Installs dependencies for main app (`src/`)
- Installs dependencies for all 19 workspace packages (`packages/`)
- Sets up monorepo linking

**💡 Expected output:** You should see progress bars and eventually a success message

---

### Step 6: Run the Development Server

```bash
# Start the dev server
bun run dev
```

**What happens:**
- Next.js 16 development server starts
- Turbopack (fast bundler) compiles your code
- Server runs on http://localhost:3010

**✅ Success looks like:**
```
▲ Next.js 16.0.3
- Local:        http://localhost:3010
- Turbopack:    Enabled

✓ Ready in 2.5s
```

---

### Step 7: Open in Browser

Open your browser to:

```
http://localhost:3010
```

**🎉 You should see the LobeHub interface!**

**⚠️ Note:** Without environment variables configured, some features (like AI chat) won't work yet. That's expected!

---

## Full Setup (15-20 minutes)

**Goal:** Configure environment variables for full functionality

### Environment Configuration

The project supports two modes:
1. **Client-only mode** - PGLite database runs in browser (no server setup needed)
2. **Server mode** - Neon PostgreSQL + auth + S3 storage

**For learning/development, client-only mode is sufficient.**

---

### Option A: Minimal Setup (Client-Only)

**No configuration needed!** The app runs with defaults:
- PGLite database (runs in browser)
- No authentication
- Local file storage
- Basic features enabled

**This is perfect for:**
- Learning the codebase
- Frontend development
- Component development
- Testing UI changes

---

### Option B: Full Setup (Server + Database)

**For backend development, auth, or production-like setup:**

#### 1. Copy Environment Template

```bash
# Copy the development example
cp .env.example.development .env.local
```

**🎯 Remember:** `.env.local` is git-ignored (safe for secrets)

---

#### 2: Configure Basic Settings

Edit `.env.local`:

```bash
# Minimum required configuration
KEY_VAULTS_SECRET="your-secret-key-here"
NEXT_AUTH_SECRET="your-auth-secret-here"
```

**Generate secrets:**
```bash
# Generate a secure random secret
openssl rand -base64 32
```

**💡 Tip:** Run the command twice to get two different secrets

---

#### 3. Database Setup (Optional)

**Option 1: Use PGLite (Client-Side)**

No setup needed! PGLite runs in the browser automatically.

**Option 2: Use PostgreSQL (Server-Side)**

**With Docker:**
```bash
# Start PostgreSQL with Docker Compose
docker compose -f docker-compose.development.yml up -d postgres

# Run migrations
bun run db:migrate
```

**Without Docker:**
```bash
# Install PostgreSQL locally (macOS)
brew install postgresql@15

# Start PostgreSQL
brew services start postgresql@15

# Create database
createdb lobechat

# Add to .env.local
DATABASE_URL=postgresql://localhost:5432/lobechat

# Run migrations
bun run db:migrate
```

---

#### 4. Authentication Setup (Optional)

**Default:** No authentication (dev mode)

**To enable auth**, add to `.env.local`:

```bash
NEXT_PUBLIC_ENABLE_NEXT_AUTH=1
NEXT_AUTH_SECRET="your-secret-from-step-2"
AUTH_URL=http://localhost:3010/api/auth
```

📚 **More on auth:** [Backend Architecture - Authentication](./BACKEND_ARCHITECTURE.md#authentication)

---

#### 5. API Keys (Optional)

**To use AI models**, add your API keys to `.env.local`:

```bash
# OpenAI
OPENAI_API_KEY=sk-your-key-here

# Anthropic
ANTHROPIC_API_KEY=sk-ant-your-key-here

# Google
GOOGLE_API_KEY=your-key-here
```

**⚠️ Security:** Never commit `.env.local` to git! It's already in `.gitignore`.

📚 **More on integrations:** [Integration Guide](./INTEGRATION_GUIDE.md)

---

### Restart the Server

After configuring environment variables:

```bash
# Stop the server (Ctrl+C)

# Start again
bun run dev
```

**The server will now use your configuration!**

---

## Verifying Your Setup

### Check 1: Dev Server is Running

```bash
# In your terminal, you should see:
✓ Ready in X.Xs
- Local: http://localhost:3010
```

**✅ Pass:** Server is accessible

---

### Check 2: Hot Reload Works

1. Open `src/app/[variants]/(main)/chat/page.tsx`
2. Make a small change (add a comment or log)
3. Save the file

**✅ Pass:** Browser updates automatically (Fast Refresh)

---

### Check 3: TypeScript is Working

```bash
# Run type checking
bun run type-check
```

**✅ Pass:** No TypeScript errors (or only known issues)

---

### Check 4: Database Connection (If Using Server DB)

```bash
# Open Drizzle Studio (database GUI)
bun run db:studio
```

**✅ Pass:** Opens at http://localhost:4983 with your database

---

## Running Tests

### Unit & Integration Tests (Vitest)

```bash
# Run all tests (not recommended - takes ~10 minutes)
bun run test

# Run specific test file
bunx vitest run --silent='passed-only' 'src/services/chat/client.test.ts'

# Run tests for a directory
bunx vitest run --silent='passed-only' 'src/services/**/*.test.ts'

# Run tests in watch mode
bunx vitest 'src/services/chat/client.test.ts'
```

**🎯 Remember:** Always wrap file patterns in single quotes!

**💡 Tip:** Use `--silent='passed-only'` to hide passing tests (cleaner output)

---

### End-to-End Tests (Playwright)

```bash
# Install Playwright browsers (first time only)
bunx playwright install

# Run E2E tests
bun run test:e2e

# Run in UI mode (interactive)
bun run e2e:ui
```

📚 **More on testing:** [Testing Guide](./TESTING_GUIDE.md)

---

## Common Issues & Solutions

### Issue: "Module not found" errors

**Problem:** Missing dependencies or stale lockfile

**Solution:**
```bash
# Clean install
rm -rf node_modules pnpm-lock.yaml
pnpm install
```

---

### Issue: Port 3010 is already in use

**Problem:** Another process is using port 3010

**Solution:**
```bash
# Find and kill the process
lsof -ti:3010 | xargs kill -9

# Or use a different port
bun run dev -- -p 3011
```

---

### Issue: "EACCES: permission denied"

**Problem:** Permission issues with npm global packages

**Solution:**
```bash
# Fix npm permissions (macOS/Linux)
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'

# Add to ~/.zshrc or ~/.bashrc
export PATH=~/.npm-global/bin:$PATH

# Re-install pnpm
npm install -g pnpm
```

---

### Issue: Build fails with "JavaScript heap out of memory"

**Problem:** Node.js needs more memory

**Solution:**
```bash
# Increase Node.js memory limit
export NODE_OPTIONS="--max-old-space-size=6144"

# Then build again
bun run build
```

**💡 This is already configured in package.json for builds**

---

### Issue: TypeScript errors about missing types

**Problem:** Types not generated yet

**Solution:**
```bash
# Generate database types
bun run db:generate

# Generate client DB types
bun run db:generate-client
```

---

### Issue: Hot reload not working

**Problem:** File watcher limits (Linux)

**Solution:**
```bash
# Increase file watcher limit
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

### Issue: pnpm install hangs or is slow

**Problem:** Network issues or registry problems

**Solution:**
```bash
# Use faster registry mirror (optional)
pnpm config set registry https://registry.npmjs.org/

# Or clear pnpm cache
pnpm store prune
```

---

## Development Tools

### Recommended VS Code Extensions

```json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "bradlc.vscode-tailwindcss",
    "Prisma.prisma",
    "oven.bun-vscode",
    "ms-playwright.playwright"
  ]
}
```

**These are already in `.vscode/extensions.json`** - VS Code will prompt you to install them!

---

### Useful Commands Reference

```bash
# Development
bun run dev                 # Start web dev server (port 3010)
bun run dev:desktop         # Start desktop dev server (port 3015)
bun run dev:mobile          # Start mobile dev server (port 3018)

# Building
bun run build               # Build for production
bun run build:analyze       # Build with bundle analyzer
bun run start               # Start production server

# Database
bun run db:generate         # Generate migrations from schema
bun run db:migrate          # Run migrations
bun run db:studio           # Open Drizzle Studio GUI

# Testing
bunx vitest run --silent='passed-only' 'path/to/test'
bun run test:e2e            # Run Playwright E2E tests
bun run e2e:ui              # Interactive E2E testing

# Code Quality
bun run type-check          # TypeScript type checking
bun run lint                # Run all linters
bun run lint:ts             # Lint TypeScript
bun run lint:style          # Lint styles
bun run prettier            # Format all files

# Cleaning
bun run clean:node_modules  # Remove all node_modules
bun run reinstall           # Clean reinstall dependencies
```

📚 **Full commands:** [Development Workflow](./DEVELOPMENT_WORKFLOW.md)

---

### Development Ports Reference

| Port | Service | Command |
|------|---------|---------|
| 3010 | Web dev server | `bun run dev` |
| 3015 | Desktop dev server | `bun run dev:desktop` |
| 3018 | Mobile dev server | `bun run dev:mobile` |
| 3210 | Production server | `bun run start` |
| 4983 | Drizzle Studio | `bun run db:studio` |
| 5432 | PostgreSQL | Docker compose |

---

## Development Workflow Tips

### 🧠 Mental Models

**1. Monorepo Structure**
```
Think of it as multiple npm packages in one repo.
Changes in packages/database affect the main app instantly.
```

**2. Next.js App Router**
```
File system = URL structure
app/[variants]/(main)/chat/page.tsx → /chat
```

**3. Dual Database**
```
PGLite (browser) = localStorage++ (but it's full PostgreSQL!)
Neon (server) = Traditional backend database
```

**4. tRPC Routes**
```
Type-safe API calls without API documentation
Client knows server types automatically
```

---

### 💡 Pro Tips

**Tip 1: Use Turbopack**
Next.js 16 uses Turbopack by default - it's 5-10x faster than Webpack!

**Tip 2: Use `bunx` for one-off commands**
```bash
bunx vitest      # Run vitest without global install
bunx playwright  # Run playwright without global install
```

**Tip 3: Check package.json scripts**
```bash
# See all available commands
cat package.json | jq .scripts
```

**Tip 4: Use Drizzle Studio for DB inspection**
```bash
bun run db:studio
# Visual interface for your database!
```

**Tip 5: Enable React DevTools**
React 19 works with React DevTools - install the browser extension!

---

## Next Steps

Now that you have LobeHub running, here's what to do next:

### Understanding the System (2-3 hours)

1. **[Architecture Overview](./ARCHITECTURE_OVERVIEW.md)** (45 min)
   - Understand the big picture
   - How components fit together
   - Data flow patterns

2. **[Project Structure](./PROJECT_STRUCTURE.md)** (30 min)
   - Learn where everything lives
   - Navigate the codebase confidently
   - Understand monorepo organization

3. **[Tech Stack Guide](./TECH_STACK_GUIDE.md)** (2 hours)
   - Deep dive into React 19 features
   - Learn Next.js 16 App Router
   - Understand tRPC, Zustand, Drizzle
   - "Bridge from React" explanations

---

### Practical Development (2-3 hours)

4. **[Development Workflow](./DEVELOPMENT_WORKFLOW.md)** (30 min)
   - Daily development routine
   - Best practices
   - Tool usage

5. **[Patterns & Conventions](./PATTERNS_AND_CONVENTIONS.md)** (1 hour)
   - Code style guide
   - Current vs outdated patterns
   - Project conventions

6. **[How-To Guide](./HOW_TO_GUIDE.md)** (1.5 hours)
   - Add a new page
   - Create a tRPC route
   - Add a database table
   - Step-by-step recipes

---

### Diving Deep (As Needed)

Choose based on your focus area:

- **Frontend:** [Frontend Architecture](./FRONTEND_ARCHITECTURE.md)
- **Backend:** [Backend Architecture](./BACKEND_ARCHITECTURE.md)
- **Database:** [Database Architecture](./DATABASE_ARCHITECTURE.md)
- **Testing:** [Testing Guide](./TESTING_GUIDE.md)

---

## Quick Reference Card

**Clone and Run:**
```bash
git clone https://github.com/lobehub/lobe-chat.git
cd lobe-chat
pnpm install
bun run dev
# → http://localhost:3010
```

**Test a Component:**
```bash
bunx vitest run --silent='passed-only' 'src/components/Button.test.tsx'
```

**Check Types:**
```bash
bun run type-check
```

**View Database:**
```bash
bun run db:studio
# → http://localhost:4983
```

**Common Ports:**
- 3010 = Dev server (web)
- 3015 = Dev server (desktop)
- 4983 = Drizzle Studio

---

## Getting Help

### Within Documentation

- **Issues:** [Debugging Guide](./DEBUGGING_GUIDE.md)
- **Questions:** [FAQ](./FAQ.md)
- **Tasks:** [How-To Guide](./HOW_TO_GUIDE.md)

### External Resources

- **GitHub Issues:** [Report bugs or ask questions](https://github.com/lobehub/lobe-chat/issues)
- **Discord:** [Join the community](https://discord.gg/lobehub) *(check main README for link)*
- **Docs:** [Official documentation](https://lobehub.com/docs)

### Before Asking for Help

**Include this information:**
1. What you're trying to do
2. What command you ran
3. The error message (full text)
4. Your environment (OS, Node version, pnpm version)

```bash
# Get environment info quickly
node -v && pnpm -v && bun -v && git --version
```

---

## Troubleshooting Checklist

**Before filing an issue, try these:**

- [ ] `rm -rf node_modules && pnpm install` (clean install)
- [ ] `bun run type-check` (check for type errors)
- [ ] Check `.env.local` configuration
- [ ] Restart the dev server
- [ ] Clear browser cache / try incognito mode
- [ ] Check [Debugging Guide](./DEBUGGING_GUIDE.md)
- [ ] Search [GitHub Issues](https://github.com/lobehub/lobe-chat/issues)

---

**🎉 Congratulations! You're ready to start developing with LobeHub!**

**Next:** Read [Architecture Overview](./ARCHITECTURE_OVERVIEW.md) to understand the system

---

**Document Status:** ✅ Complete | **Last Updated:** November 20, 2025
