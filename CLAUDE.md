# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an educational repository for teaching spec-driven development: "From PRD to Prod". It contains:

1. **Prompt framework documentation** (`docs/PromptTemplates/`) - Teaching materials for using AI assistants to go from Product Requirements Documents to working code
2. **FIFA Traffic Demo** (`fifa-traffic-demo/`) - A sample Next.js application demonstrating the methodology in action

The demo app is a FIFA Atlanta Navigator showing:
- Google Maps integration with traffic
- MARTA Bus & Train API integration
- Mock FIFA team schedule
- Multi-language support (EN, ES, DE, KO)
- Animated transit vehicle markers

## Development Environment

### DevContainer Setup (Pre-Configured - DO NOT MODIFY)

The repository uses a fully configured DevContainer with:
- Node.js 20 (Bookworm)
- pnpm package manager
- Bun runtime
- VS Code extensions: Claude Code, Gemini Code Assist, ESLint, Prettier
- Serena MCP server integration
- Auto-forwarded port 3000

**CRITICAL**: The DevContainer is production-ready. Never modify `.devcontainer/devcontainer.json` or related files unless explicitly requested.

### Post-Create Automation

The `postCreate.sh` script automatically:
- Enables corepack and configures pnpm
- Installs Gemini CLI globally
- Installs Vercel CLI for deployment
- Conditionally installs Claude CLI if `ANTHROPIC_API_KEY` is present
- Conditionally installs OpenAI Codex if `OPENAI_API_KEY` is present
- Skips automatic dependency installation for faster boot (run `pnpm install` manually)

## Working with the FIFA Traffic Demo

### Location

All demo application code is in `fifa-traffic-demo/` subdirectory.

### Common Commands

```bash
# Navigate to demo app
cd fifa-traffic-demo

# Install dependencies
pnpm install

# Run development server (port 3000)
pnpm dev

# Build for production
pnpm build

# Start production server
pnpm start

# Lint code
pnpm lint
```

### Tech Stack

- **Framework**: Next.js 15 (App Router)
- **Language**: TypeScript 5
- **Styling**: Tailwind CSS
- **Maps**: `@vis.gl/react-google-maps` for Google Maps React integration
- **Transit Data**: `gtfs-realtime-bindings` for MARTA API parsing
- **Data Fetching**: SWR for client-side data fetching
- **Deployment**: Vercel

### Architecture

#### App Structure (Next.js App Router)

```
src/
├── app/
│   ├── page.tsx                     # Root redirect to /en
│   ├── [lang]/                      # Internationalized routes
│   │   ├── page.tsx                 # Main app page
│   │   ├── layout.tsx               # Language-aware layout
│   │   └── globals.css              # Global styles
│   └── api/                         # API routes
│       ├── events/route.ts          # Mock FIFA events
│       ├── transit/route.ts         # MARTA transit data
│       └── venues/route.ts          # Stadium/venue data
├── components/
│   ├── AnimatedTransitMarker.tsx    # Smooth-moving transit markers
│   ├── EventCard.tsx                # FIFA event display
│   ├── EventList.tsx                # List of events
│   ├── LanguageSwitcher.tsx         # i18n language selector
│   └── MapView.tsx                  # Google Maps component
├── lib/
│   ├── codespaces.ts                # Codespaces environment detection
│   ├── data.ts                      # Mock data (FIFA matches)
│   ├── dateFormat.ts                # Date formatting utilities
│   ├── i18n.ts                      # Internationalization config
│   └── storage.ts                   # Local/cloud storage abstraction
└── types/
    └── index.ts                     # TypeScript type definitions
```

#### Key Architectural Patterns

**Codespaces Proxy Solution**: The `lib/codespaces.ts` utility detects GitHub Codespaces environment. MARTA Train API runs on port 18096 which Codespaces blocks, so API routes conditionally use proxy URLs in Codespaces but direct connections in production.

**Internationalization**: Uses Next.js dynamic routing with `[lang]` parameter. Supported languages: en, es, de, ko. The `lib/i18n.ts` provides translation dictionaries.

**Animated Markers**: `AnimatedTransitMarker.tsx` smoothly animates transit vehicles on the map (similar to Uber car animations) rather than jumping positions.

**API Route Pattern**: Backend API routes (`app/api/*/route.ts`) handle:
- Environment detection (Codespaces vs production)
- External API calls to MARTA
- Mock data serving for FIFA schedules
- Error handling and fallbacks

### Environment Variables

Copy `.env` from root or create in `fifa-traffic-demo/` with:

```bash
# Google Maps
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=your_key_here

# MARTA APIs
MARTA_API_KEY=your_key_here
MARTA_TRAIN_API_KEY=your_key_here

# Stadium location (public)
NEXT_PUBLIC_STADIUM_NAME="Mercedes-Benz Stadium"
NEXT_PUBLIC_STADIUM_LAT="33.754542"
NEXT_PUBLIC_STADIUM_LNG="-84.402492"

# Optional features
USE_MOCK_MARTA_DATA=false
DATABASE_URL="file:./dev.db"
```

**Note**: `NEXT_PUBLIC_*` variables are exposed to the browser.

## Prompt Framework Methodology

This repository teaches an atomic breakdown methodology:

### Hierarchy: Organisms → Molecules → Atoms

- **Organisms** (MS-01, MS-02): Major feature areas / Epics
- **Molecules** (MS-01.01, MS-01.02): Specific features / Stories
- **Atoms** (MS-01.01-A, MS-01.01-B): Atomic implementation tasks

### Framework Selection

The `docs/PromptTemplates/00-META-FRAMEWORK-GUIDE.md` provides decision logic for choosing the right prompt framework:

- **S.P.A.R.K**: Educational setup tasks
- **F.O.C.U.S**: Problem-solving with constraints (e.g., Codespaces proxy)
- **C.R.A.F.T**: Technical context-driven tasks (e.g., API integration)
- **I.D.E.A**: Creative/design tasks (e.g., UI components)
- **S.T.A.G.E**: Step-by-step implementation
- **M.A.P**: Metrics and planning / acceptance criteria
- **P.R.O.M.P.T**: Testing strategy
- **D.E.C.I.D.E**: Decision-making (e.g., tech stack choices)

### Three-Phase Workflow

1. **Phase 1: Foundation Documents** - Generate spec.md, guardrails.md, acceptance_criteria.md from PRD
2. **Phase 2: Implementation** - Execute atomic tasks using AI coding assistants
3. **Phase 3: Milestone Planning** - Document what was built in hierarchical structure

See `docs/PromptTemplates/00-MASTER-GUIDE.md` for complete workflow.

## Deployment

### Vercel Deployment

```bash
cd fifa-traffic-demo

# Login to Vercel (first time)
vercel login

# Deploy to preview
vercel

# Deploy to production
vercel --prod
```

The `vercel.json` configuration is pre-configured for Next.js deployment.

## Important Constraints

1. **DO NOT modify DevContainer files** unless explicitly requested
2. **Codespaces port 18096 is blocked** - use the proxy pattern in `lib/codespaces.ts`
3. **Multi-language support required** - all user-facing strings must use i18n
4. **App Router only** - do not use Pages Router patterns
5. **Environment detection matters** - code must work in both Codespaces and production

## Common Pitfalls

- Forgetting to add `NEXT_PUBLIC_` prefix for client-side env vars
- Not handling Codespaces environment detection for MARTA Train API
- Using Pages Router patterns instead of App Router
- Hardcoding strings instead of using i18n translations
- Not running `pnpm install` after pulling changes (it's not automatic)

## Testing Approach

No formal test suite currently exists. Verification is manual:
- Start dev server: `pnpm dev`
- Check map loads with traffic layer
- Verify MARTA transit data appears
- Test language switcher (EN, ES, DE, KO)
- Confirm animated markers move smoothly
- Check mobile responsiveness
