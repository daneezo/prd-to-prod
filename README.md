# From PRD to Prod: Spec-Driven Development Workshop

Learn to build production-ready applications using AI assistants with intentional, spec-driven methodology.

## What's Inside

### 📊 Workshop Slides (`slides/`)
46-slide deck teaching the methodology:
- **Vibe vs Spec Coding**: When to trust the vibe vs. when to spec it out
- **The Pipeline**: PRD → Spec → Guardrails → Acceptance Criteria → Implementation
- **Atomic Design**: Breaking features into Organisms (Epics) → Molecules (Stories) → Atoms (Tasks)
- **8 Prompt Frameworks**: S.P.A.R.K, F.O.C.U.S, C.R.A.F.T, I.D.E.A, S.T.A.G.E, M.A.P, P.R.O.M.P.T, D.E.C.I.D.E
- **Hands-On Coding**: Guided implementation session with visual target reference

**Run Slides**:
```bash
cd slides && npx @slidev/cli slides.md
```

### 📚 Prompt Templates (`docs/PromptTemplates/`)
Ready-to-use prompts for each phase:

**Phase 1: Foundation** - Generate core documents
- `spec.md` (technical specification)
- `guardrails.md` (constraints and rules)
- `acceptance_criteria.md` (verification checklist)

**Phase 2: Implementation** - Execute atomic tasks
- Use AI coding assistants (Claude/Gemini) with individual atom prompts
- Each task is small, testable, and self-contained

**Phase 3: Milestone Planning** - Document hierarchical structure
- Organisms (MS-01, MS-02) = Major feature areas
- Molecules (MS-01.01) = Specific features
- Atoms (MS-01.01-A) = Implementation tasks

### 🚀 FIFA Atlanta Navigator Demo (`fifa-traffic-demo/`) -- on `demo` branch
Working Next.js app built using this methodology:
- Google Maps with real-time traffic
- MARTA Bus & Train API integration
- Mock FIFA match schedule
- Multi-language support (EN, ES, DE, KO)
- Animated transit markers

**Try the Demo**:
```bash
git checkout demo
cd fifa-traffic-demo
pnpm install
pnpm run build
pnpm run dev
# Open http://localhost:3000
```

## How to Use Spec-Driven Development

### Quick Start: Build the FIFA Demo

**1. Start with the PRD**
- Review the Product Requirements Document
- Understand the business goals and user needs
- Identify major feature areas (Organisms/Epics)

**2. Generate Foundation Documents** (Phase 1)
Use AI assistant with prompt frameworks:
```bash
# In your AI chat (Claude/Gemini):
- Load: docs/PromptTemplates/01-PHASE-1-Foundation-Prompts.md
- Run C.R.A.F.T framework → Generate spec.md
- Run D.E.C.I.D.E framework → Generate guardrails.md
- Run M.A.P framework → Generate acceptance_criteria.md
```

**3. Create Atomic Breakdown** (Phase 3)
Break the project into hierarchical structure:
```
MS-01: Project Setup & Core Features (Organism/Epic)
  ├─ MS-01.01: Next.js Project Init (Molecule/Story)
  │   ├─ MS-01.01-A: Create Next.js app (Atom/Task)
  │   └─ MS-01.01-B: Configure TypeScript (Atom/Task)
  ├─ MS-01.02: Google Maps Integration (Molecule/Story)
  │   ├─ MS-01.02-A: Install Google Maps library (Atom/Task)
  │   └─ MS-01.02-B: Add traffic layer (Atom/Task)
  └─ MS-01.03: MARTA API Integration (Molecule/Story)
      ├─ MS-01.03-A: Connect to Bus API (Atom/Task)
      └─ MS-01.03-B: Connect to Train API (Atom/Task)
```

**4. Implement Atoms** (Phase 2)
For each atom:
```bash
# In AI coding assistant (Claude Code / Gemini Code Assist):
1. Load the atom prompt from Phase 2 docs
2. Reference spec.md, guardrails.md, acceptance_criteria.md
3. Let AI implement the feature
4. Verify against acceptance criteria
5. Move to next atom
```

**5. Deploy to Production**
```bash
cd fifa-traffic-demo
vercel --prod
```

## Why This Matters

**Without Spec-Driven Development:**
- AI hallucinations and feature drift
- Context loss across sessions
- Inconsistent implementation
- No verification process

**With Spec-Driven Development:**
- Clear technical specifications
- Guardrails prevent common mistakes
- Verifiable acceptance criteria
- Atomic tasks maintain focus
- Scalable to large projects

## Repository Structure

```
prd-to-prod/
├── slides/                    # Workshop presentation
│   ├── 00-title/             # Introduction
│   ├── 01-vibe-coding/       # Context setting
│   ├── 02-spec-driven/       # Core methodology
│   ├── 03-pipeline/          # PRD → Prod flow
│   ├── 04-atomic-design/     # Hierarchical breakdown
│   ├── 05-environment/       # Setup instructions
│   ├── 06-framework/         # 8 prompt frameworks
│   ├── 07-guardrails-ac/     # Constraints & verification
│   ├── 08-milestones/        # Progress tracking
│   └── 09-wrap-up/           # Next steps
│
├── docs/
│   ├── PromptTemplates/      # Ready-to-use prompts
│   │   ├── 00-MASTER-GUIDE.md
│   │   ├── 00-META-FRAMEWORK-GUIDE.md
│   │   ├── 01-PHASE-1-Foundation-Prompts.md
│   │   ├── 02-PHASE-2-Implementation-Prompts.md
│   │   └── 03-PHASE-3-Milestone-Prompts.md
│   └── PRD/                  # Example product requirements
│
├── fifa-traffic-demo/         # Working demo application
│   ├── src/
│   │   ├── app/              # Next.js App Router
│   │   ├── components/       # React components
│   │   ├── lib/              # Utilities & config
│   │   └── types/            # TypeScript definitions
│   ├── public/               # Static assets
│   └── package.json          # Dependencies
│
└── .devcontainer/            # Pre-configured dev environment
```

## Environment Setup

### Option 1: GitHub Codespaces (Recommended)
```bash
# Fork this repo
# Click "Code" → "Create codespace on demo"
# Everything pre-configured!
```

### Option 2: Local Dev Container
```bash
# Clone the repo
git clone https://github.com/doctor-ew/prd-to-prod.git
cd prd-to-prod

# Open in VS Code with Dev Containers extension
# VS Code will prompt to "Reopen in Container"
```

### Option 3: Manual Setup
```bash
# Requires Node.js 20+, pnpm
git clone https://github.com/doctor-ew/prd-to-prod.git
cd prd-to-prod/fifa-traffic-demo
pnpm install
pnpm dev
```

## Required API Keys

Create `.env` in `fifa-traffic-demo/`:
```bash
# Google Maps
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=your_key_here

# MARTA Transit APIs (optional - has fallback mock data)
MARTA_API_KEY=your_key_here
MARTA_TRAIN_API_KEY=your_key_here
```

## Key Files to Read

1. **Start Here**: `docs/PromptTemplates/00-MASTER-GUIDE.md`
2. **Framework Selection**: `docs/PromptTemplates/00-META-FRAMEWORK-GUIDE.md`
3. **Quick Reference**: `docs/PromptTemplates/01-QUICK-REFERENCE-CARD.md`
4. **Class Workflow**: `docs/PromptTemplates/02-CLASS-SESSION-WORKFLOW.md`
5. **Project Context**: `CLAUDE.md` (instructions for AI assistants)

## Three-Hour Workshop Timeline

```
0:00-0:10  Theory: Vibe vs Spec, The Pipeline
0:10-0:25  Methodology: Atomic Design, 8 Frameworks
0:25-0:45  Setup: Environment, API keys, DevContainer
0:45-1:00  Demo: Live spec generation
1:00-2:30  Hands-On: Build FIFA Navigator (guided atoms)
2:30-2:45  Review: Milestones, reflection questions
2:45-3:00  Deployment: Push to Vercel, Q&A
```

## Learning Outcomes

By the end of this workshop, you will:
- ✅ Understand when to use vibe coding vs. spec-driven development
- ✅ Generate technical specifications from PRDs using AI
- ✅ Create guardrails and acceptance criteria
- ✅ Break complex features into atomic tasks
- ✅ Use 8 different prompt frameworks effectively
- ✅ Implement a production app with AI coding assistants
- ✅ Deploy to Vercel with proper configuration
- ✅ Maintain context across long development sessions

## Tech Stack (FIFA Demo)

- **Framework**: Next.js 15 (App Router)
- **Language**: TypeScript 5
- **Styling**: Tailwind CSS
- **Maps**: @vis.gl/react-google-maps
- **Transit**: gtfs-realtime-bindings (MARTA API)
- **Data Fetching**: SWR
- **Deployment**: Vercel
- **Dev Environment**: DevContainer (Node.js 20)

## Resources

- **Repository**: [github.com/doctor-ew/prd-to-prod](https://github.com/doctor-ew/prd-to-prod)
- **Demo Branch**: `demo` (complete working code)
- **Main Branch**: `main` (teaching materials only)
- **Slides**: Run locally with Slidev
- **AI Assistants**: Claude Code, Gemini Code Assist

## Common Pitfalls & Solutions

| Problem | Solution |
|---------|----------|
| AI hallucinates features | Use guardrails.md to constrain behavior |
| Context lost mid-session | Work in atomic tasks, reference spec.md |
| Inconsistent code style | Define style guide in guardrails.md |
| Can't verify completion | Check against acceptance_criteria.md |
| MARTA API fails | Use mock data fallback (built-in) |
| Codespaces port blocked | Use proxy pattern (see lib/codespaces.ts) |

## Next Steps

1. **Today**: Fork the repo, explore the slides
2. **This Week**: Run the FIFA demo locally, read PromptTemplates
3. **Advanced**: Build your own project using the methodology

## Contributing

This is an educational repository. Issues and PRs welcome for:
- Typo fixes in documentation
- Additional example prompts
- Framework enhancements
- Demo app improvements

## License

MIT - Use freely for education and commercial projects

---

**Built with**: Claude Code, intentional Spec-Driven prompting, Information-Dense Keywords, and a belief that AI assistants work best with clear specifications.

**"Never trust anything that can think for itself if you can't see where it keeps its brain"** 🧙‍♂️
