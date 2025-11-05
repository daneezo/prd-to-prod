# Guardrails: Atlanta FIFA Navigator
> Development Constraints and Best Practices
> Version 1.0 | Generated using DECIDE Framework

---

## Framework: D.E.C.I.D.E

### Define the Problem

We need clear boundaries and constraints to ensure the Atlanta FIFA Navigator project:
- Stays on track with approved technologies
- Maintains security best practices
- Avoids common pitfalls in web development and API integration
- Delivers a consistent, high-quality user experience
- Works reliably across development (Codespaces) and production environments

---

## 1. APPROVED Technologies

### ✅ MUST USE - Core Stack

| Technology | Version | Reason |
|------------|---------|--------|
| **Next.js** | 15.x (App Router) | Modern RSC, server components, optimized performance |
| **TypeScript** | 5.x (strict mode) | Type safety, better DX, fewer runtime errors |
| **pnpm** | 8.x | Fast, efficient, required by devcontainer setup |
| **React** | 19.x | Latest features, improved concurrent rendering |
| **Prisma** | 5.x | Type-safe ORM, excellent migration tooling |

### ✅ MUST USE - Maps & APIs

| Technology | Reason |
|------------|--------|
| **@vis.gl/react-google-maps** | Official React wrapper, best performance, proper typing |
| **gtfs-realtime-bindings** | Official GTFS-RT parser, handles binary protobuf |
| **Google Maps JavaScript API** | Required for traffic layer and place data |

### ✅ MUST USE - Styling & UI

| Technology | Reason |
|------------|--------|
| **Tailwind CSS** | Utility-first, consistent design, small bundle |
| **Headless UI** (optional) | Accessible components, integrates with Tailwind |

### ✅ MUST USE - Internationalization

| Technology | Reason |
|------------|--------|
| **next-intl** | Best Next.js i18n support, App Router compatible |

### ✅ MUST USE - Data Fetching

| Technology | Reason |
|------------|--------|
| **SWR** | Automatic revalidation, caching, optimal UX |
| **Built-in fetch** | Native Next.js support, server-side caching |

### ✅ MUST USE - Validation

| Technology | Reason |
|------------|--------|
| **Zod** | Runtime type validation, excellent TypeScript integration |

---

## 2. FORBIDDEN Technologies

### ❌ DO NOT USE - Wrong Architectures

| Technology | Why Forbidden | Alternative |
|------------|---------------|-------------|
| **Next.js Pages Router** | Legacy, App Router is required | Use App Router |
| **Create React App** | Deprecated, not serverless | Use Next.js |
| **npm or yarn** | Incompatible with devcontainer setup | Use pnpm only |

### ❌ DO NOT USE - Wrong Libraries

| Technology | Why Forbidden | Alternative |
|------------|---------------|-------------|
| **react-google-maps** | Outdated, poor TypeScript support | @vis.gl/react-google-maps |
| **google-map-react** | No longer maintained | @vis.gl/react-google-maps |
| **Any other Maps library** | Not approved, integration issues | @vis.gl/react-google-maps |

### ❌ DO NOT USE - Wrong Databases

| Technology | Why Forbidden | Alternative |
|------------|---------------|-------------|
| **MongoDB** | Not specified in PRD | SQLite (dev), PostgreSQL (prod) |
| **MySQL** | Not approved | PostgreSQL |
| **Firebase** | Not approved, vendor lock-in | Prisma + PostgreSQL |

### ❌ DO NOT USE - Security Violations

| Practice | Why Forbidden | Alternative |
|----------|---------------|-------------|
| **Hardcoded API keys** | Security vulnerability, leak risk | Environment variables |
| **Inline secrets** | Git history exposure | .env with .gitignore |
| **Client-side server keys** | Exposes private credentials | Use API routes |
| **No input validation** | SQL injection, XSS risk | Zod validation |

---

## 3. Security Guardrails

### 🔒 API Key Management - CRITICAL

**MUST DO:**
- ✅ All API keys MUST be in `.env` file
- ✅ Client-side keys MUST use `NEXT_PUBLIC_` prefix
- ✅ Server-side keys MUST NEVER use `NEXT_PUBLIC_` prefix
- ✅ `.env` MUST be in `.gitignore`
- ✅ Provide `.env.example` with placeholder values
- ✅ Document all required environment variables

**MUST NOT DO:**
- ❌ NEVER commit `.env` to git
- ❌ NEVER hardcode API keys in source code
- ❌ NEVER expose server-side keys to client
- ❌ NEVER log API keys (even in development)
- ❌ NEVER share API keys in screenshots or documentation

**Example - Correct Usage:**
```typescript
// ✅ CORRECT - Server-side API route
export async function GET() {
  const apiKey = process.env.MARTA_API_KEY; // Private, server-only
  const response = await fetch(`https://api.marta.com?key=${apiKey}`);
  return Response.json(await response.json());
}

// ✅ CORRECT - Client-side component
export function MapView() {
  return (
    <APIProvider apiKey={process.env.NEXT_PUBLIC_GOOGLE_MAPS_API_KEY}>
      {/* Public key, safe for client */}
    </APIProvider>
  );
}
```

**Example - Wrong Usage:**
```typescript
// ❌ WRONG - Hardcoded key
const apiKey = "sk-1234567890abcdef";

// ❌ WRONG - Server key exposed to client
const MARTA_KEY = process.env.MARTA_API_KEY; // No NEXT_PUBLIC_ but used in client

// ❌ WRONG - Key in git
// .env file committed to repository
```

### 🔒 Input Validation & Sanitization

**MUST DO:**
- ✅ Validate all user inputs with Zod
- ✅ Validate all query parameters in API routes
- ✅ Sanitize all data before database insertion
- ✅ Use parameterized queries (Prisma handles this)
- ✅ Validate environment variables on startup

**MUST NOT DO:**
- ❌ NEVER trust user input
- ❌ NEVER use `any` type without validation
- ❌ NEVER skip validation for "internal" APIs
- ❌ NEVER concatenate user input into SQL (use Prisma)
- ❌ NEVER use `dangerouslySetInnerHTML` without sanitization

**Example - Correct Validation:**
```typescript
// ✅ CORRECT - API route with validation
import { z } from 'zod';

const querySchema = z.object({
  type: z.enum(['bus', 'train']).optional(),
  route: z.string().max(10).regex(/^[A-Z0-9]+$/).optional()
});

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);

  const result = querySchema.safeParse({
    type: searchParams.get('type'),
    route: searchParams.get('route')
  });

  if (!result.success) {
    return Response.json(
      { error: 'Invalid parameters', details: result.error },
      { status: 400 }
    );
  }

  const { type, route } = result.data;
  // Safe to use validated data
}
```

### 🔒 CORS & Proxy Security

**MUST DO:**
- ✅ Use proxy (allOrigins) ONLY in Codespaces
- ✅ Use direct connection in production
- ✅ Always encode URLs passed to proxies: `encodeURIComponent(url)`
- ✅ Validate proxy responses before parsing
- ✅ Set proper CORS headers in middleware

**MUST NOT DO:**
- ❌ NEVER use proxy in production (performance + security risk)
- ❌ NEVER pass unencoded URLs to proxy services
- ❌ NEVER trust proxy responses without validation
- ❌ NEVER disable CORS without understanding implications

**Example - Correct Proxy Usage:**
```typescript
// ✅ CORRECT - Environment-aware proxy
import { isRunningInCodespaces } from '@/lib/codespaces';

export function getMartaTrainApiUrl(): string {
  const directUrl = 'http://developer.itsmarta.com:18096/api/train';

  if (isRunningInCodespaces()) {
    const encoded = encodeURIComponent(directUrl);
    return `https://api.allorigins.win/raw?url=${encoded}`;
  }

  return directUrl;
}
```

---

## 4. Performance Guardrails

### ⚡ Response Time Requirements

**MUST MEET:**
- ✅ Time to First Byte (TTFB): < 500ms
- ✅ First Contentful Paint (FCP): < 2s
- ✅ Largest Contentful Paint (LCP): < 2.5s
- ✅ API responses: < 5s (with timeout)
- ✅ Map load time: < 3s

**MUST NOT EXCEED:**
- ❌ LCP > 4s = Critical failure
- ❌ API response > 10s = Timeout required
- ❌ Bundle size > 500KB (initial load)

### ⚡ Caching Strategy

**MUST DO:**
- ✅ Cache transit API responses for 30 seconds
- ✅ Cache events API responses for 5 minutes
- ✅ Use SWR for client-side caching
- ✅ Use Next.js `revalidate` for server caching
- ✅ Implement stale-while-revalidate pattern

**MUST NOT DO:**
- ❌ NEVER cache for > 5 minutes without revalidation
- ❌ NEVER cache user-specific data globally
- ❌ NEVER disable caching without performance testing
- ❌ NEVER cache error responses

**Example - Correct Caching:**
```typescript
// ✅ CORRECT - Server-side caching
export const revalidate = 30; // 30 seconds

export async function GET() {
  const data = await fetchTransitData();
  return Response.json(data);
}

// ✅ CORRECT - Client-side caching
const { data } = useSWR('/api/transit', fetcher, {
  refreshInterval: 30000,
  revalidateOnFocus: false,
  dedupingInterval: 5000
});
```

### ⚡ Bundle Optimization

**MUST DO:**
- ✅ Use dynamic imports for heavy components
- ✅ Enable SWC minification
- ✅ Optimize images (use Next.js Image component)
- ✅ Code split by route
- ✅ Tree-shake unused dependencies

**MUST NOT DO:**
- ❌ NEVER import entire libraries (use named imports)
- ❌ NEVER load Maps API in server components
- ❌ NEVER skip image optimization
- ❌ NEVER bundle server-only code in client bundles

---

## 5. Development Environment Guardrails

### 🖥️ GitHub Codespaces - PRIMARY Environment

**MUST DO:**
- ✅ Primary development MUST happen in GitHub Codespaces
- ✅ Test Codespaces proxy solution thoroughly
- ✅ Verify environment detection works correctly
- ✅ Use `CODESPACES` environment variable for detection
- ✅ Log environment information on startup

**MUST NOT DO:**
- ❌ NEVER assume production = Codespaces
- ❌ NEVER hardcode Codespaces-specific URLs
- ❌ NEVER skip environment detection
- ❌ NEVER use proxy in production

**Example - Environment Detection:**
```typescript
// ✅ CORRECT - Explicit environment detection
export function isRunningInCodespaces(): boolean {
  return process.env.CODESPACES === 'true';
}

// Server startup logging
console.log('🚀 Environment:', {
  isCodespaces: isRunningInCodespaces(),
  nodeEnv: process.env.NODE_ENV,
  trainApiUrl: getMartaTrainApiUrl()
});
```

### 🖥️ Port 18096 Handling

**MUST DO:**
- ✅ Use proxy for port 18096 in Codespaces
- ✅ Use direct connection in production
- ✅ Implement fallback logic for proxy failures
- ✅ Test both proxy and direct connection paths

**MUST NOT DO:**
- ❌ NEVER hardcode proxy URL everywhere
- ❌ NEVER skip direct connection testing
- ❌ NEVER ignore proxy errors silently

---

## 6. Code Quality Guardrails

### 📝 TypeScript Standards

**MUST DO:**
- ✅ Use TypeScript strict mode
- ✅ Define interfaces for all data structures
- ✅ Type all function parameters and returns
- ✅ Use Zod for runtime validation
- ✅ No implicit `any` types

**MUST NOT DO:**
- ❌ NEVER use `any` without explicit justification
- ❌ NEVER use `@ts-ignore` without comment explaining why
- ❌ NEVER skip type definitions for external data
- ❌ NEVER use `as any` to bypass type errors

**Example - Type Standards:**
```typescript
// ✅ CORRECT - Properly typed
interface TransitVehicle {
  id: string;
  type: 'bus' | 'train';
  latitude: number;
  longitude: number;
}

async function fetchTransitData(): Promise<TransitVehicle[]> {
  const response = await fetch('/api/transit');
  const data = await response.json();

  // Runtime validation
  return transitVehicleArraySchema.parse(data);
}

// ❌ WRONG - Untyped
async function fetchTransitData() {
  const response = await fetch('/api/transit');
  return await response.json(); // any type!
}
```

### 📝 Error Handling

**MUST DO:**
- ✅ Catch and handle all API errors
- ✅ Provide fallback data on failures
- ✅ Log errors with context (server-side)
- ✅ Show user-friendly error messages
- ✅ Never expose internal errors to users

**MUST NOT DO:**
- ❌ NEVER let promises reject unhandled
- ❌ NEVER return 500 errors for expected failures
- ❌ NEVER show stack traces to users
- ❌ NEVER ignore errors silently
- ❌ NEVER crash the entire app on API failure

**Example - Error Handling:**
```typescript
// ✅ CORRECT - Graceful degradation
export async function GET() {
  try {
    const data = await fetchMartaData();
    return Response.json({
      buses: data.buses,
      trains: data.trains,
      source: 'live'
    });
  } catch (error) {
    console.error('MARTA API error:', error);

    // Fallback to mock data
    return Response.json({
      buses: [],
      trains: [],
      source: 'error',
      timestamp: new Date().toISOString()
    });
  }
}

// ❌ WRONG - Unhandled error
export async function GET() {
  const data = await fetchMartaData(); // Can throw!
  return Response.json(data);
}
```

### 📝 Console Logging

**MUST DO:**
- ✅ Log environment information on startup
- ✅ Log API errors with context
- ✅ Log proxy vs direct connection usage
- ✅ Use structured logging format
- ✅ Remove debug logs before production

**MUST NOT DO:**
- ❌ NEVER log API keys or secrets
- ❌ NEVER log personal user data
- ❌ NEVER leave excessive debug logs in production
- ❌ NEVER log sensitive error details to client

**Example - Logging Standards:**
```typescript
// ✅ CORRECT - Structured, informative logging
console.log('[Transit API] Fetching data', {
  environment: isRunningInCodespaces() ? 'Codespaces' : 'Production',
  useProxy: isRunningInCodespaces(),
  timestamp: new Date().toISOString()
});

// ❌ WRONG - Logs sensitive data
console.log('API Key:', process.env.MARTA_API_KEY);
```

---

## 7. Database Guardrails

### 💾 Prisma Usage

**MUST DO:**
- ✅ Use Prisma Client for all database operations
- ✅ Run migrations before deployment
- ✅ Seed database with sample data
- ✅ Use SQLite for development
- ✅ Use PostgreSQL for production (optional)

**MUST NOT DO:**
- ❌ NEVER write raw SQL queries
- ❌ NEVER skip migrations
- ❌ NEVER commit database files to git
- ❌ NEVER use SQLite in production without understanding limits

### 💾 Data Modeling

**MUST DO:**
- ✅ Define all models in `schema.prisma`
- ✅ Use proper relationships (foreign keys)
- ✅ Add indexes for frequently queried fields
- ✅ Use appropriate data types
- ✅ Include timestamps (createdAt, updatedAt)

**MUST NOT DO:**
- ❌ NEVER store JSON when relational data is better
- ❌ NEVER skip index optimization
- ❌ NEVER use String for dates (use DateTime)

---

## 8. Internationalization (i18n) Guardrails

### 🌍 Language Support

**MUST DO:**
- ✅ Support exactly 4 languages: EN, ES, DE, KO
- ✅ Use next-intl for routing and translations
- ✅ Store translations in `/public/locales/[lang]/`
- ✅ Use locale-aware date/time formatting
- ✅ Default to English (en) for unsupported locales

**MUST NOT DO:**
- ❌ NEVER hardcode strings in components
- ❌ NEVER use English strings as fallback in other languages
- ❌ NEVER skip translation for any language
- ❌ NEVER mix translation libraries

**Example - i18n Usage:**
```typescript
// ✅ CORRECT - Using translations
import { useTranslations } from 'next-intl';

export function EventList() {
  const t = useTranslations('events');

  return <h2>{t('title')}</h2>;
}

// ❌ WRONG - Hardcoded strings
export function EventList() {
  return <h2>FIFA Matches</h2>;
}
```

### 🌍 Routing

**MUST DO:**
- ✅ Use `/[lang]/` dynamic route pattern
- ✅ Redirect `/` to `/en`
- ✅ Validate locale in middleware
- ✅ Preserve locale on navigation

**MUST NOT DO:**
- ❌ NEVER allow invalid locale codes
- ❌ NEVER break URLs when switching languages

---

## 9. Accessibility Guardrails

### ♿ WCAG 2.1 AA Compliance

**MUST DO:**
- ✅ All interactive elements must be keyboard accessible
- ✅ Color contrast ratio ≥ 4.5:1 for text
- ✅ All images must have alt text
- ✅ Forms must have proper labels
- ✅ Use semantic HTML elements

**MUST NOT DO:**
- ❌ NEVER use color alone to convey information
- ❌ NEVER create keyboard traps
- ❌ NEVER skip focus indicators
- ❌ NEVER use `<div>` for buttons

---

## 10. Git & Version Control Guardrails

### 🔀 Commit Standards

**MUST DO:**
- ✅ Write clear, descriptive commit messages
- ✅ Use conventional commits format
- ✅ Review changes before committing
- ✅ Keep commits atomic and focused

**MUST NOT DO:**
- ❌ NEVER commit `.env` files
- ❌ NEVER commit `node_modules/`
- ❌ NEVER commit database files (`*.db`)
- ❌ NEVER commit API keys or secrets

### 🔀 .gitignore Requirements

**MUST INCLUDE:**
```
# Dependencies
node_modules/
.pnp
.pnp.js

# Environment
.env
.env.local
.env.*.local

# Database
*.db
*.db-journal
prisma/dev.db

# Build output
.next/
out/
dist/

# IDE
.vscode/
.idea/

# OS
.DS_Store
Thumbs.db
```

---

## 11. DevContainer Guardrails

### 🐳 DO NOT MODIFY

**CRITICAL - READ THIS:**

The `.devcontainer/` configuration is **FULLY CONFIGURED** and **PRODUCTION-READY**.

**MUST NOT DO:**
- ❌ **NEVER** modify `.devcontainer/devcontainer.json`
- ❌ **NEVER** modify `.devcontainer/postCreate.sh`
- ❌ **NEVER** change Node.js version
- ❌ **NEVER** change package manager from pnpm
- ❌ **NEVER** add/remove VS Code extensions without instructor approval

**EXCEPTION:**
Only modify devcontainer if explicitly instructed by the course instructor.

**RATIONALE:**
The devcontainer is carefully tuned for:
- Correct Node.js and pnpm versions
- Required VS Code extensions
- Port forwarding configuration
- Post-create automation
- Consistent environment across all students

Modifying it will cause:
- Build failures
- Incompatibility with course materials
- Difficult-to-debug environment issues

---

## 12. Testing Guardrails

### 🧪 Testing Requirements

**MUST DO:**
- ✅ Test in GitHub Codespaces first
- ✅ Verify environment detection works
- ✅ Test error conditions (bad API keys, timeouts)
- ✅ Test all 4 languages
- ✅ Test on multiple screen sizes

**MUST NOT DO:**
- ❌ NEVER skip manual testing
- ❌ NEVER assume "it works on my machine"
- ❌ NEVER deploy without testing proxy solution

---

## 13. Deployment Guardrails

### 🚀 Vercel Deployment

**MUST DO:**
- ✅ Set environment variables in Vercel dashboard
- ✅ Test build locally before deploying
- ✅ Run database migrations before deployment
- ✅ Monitor deployment logs
- ✅ Test production URLs after deployment

**MUST NOT DO:**
- ❌ NEVER deploy with `.env` file
- ❌ NEVER skip production testing
- ❌ NEVER ignore build warnings

---

## Summary: Critical Guardrails

### ⚠️ TOP 10 RULES - NEVER VIOLATE

1. **NEVER commit API keys or `.env` files to git**
2. **NEVER modify `.devcontainer/` configuration**
3. **NEVER use npm or yarn (pnpm ONLY)**
4. **NEVER use Next.js Pages Router (App Router ONLY)**
5. **NEVER use proxy in production (Codespaces ONLY)**
6. **NEVER expose server-side API keys to client**
7. **NEVER skip input validation on API routes**
8. **NEVER use `any` type without validation**
9. **NEVER hardcode strings (use i18n)**
10. **NEVER deploy without testing in Codespaces first**

---

## Enforcement Checklist

Before marking any task complete, verify:

- [ ] No hardcoded API keys or secrets
- [ ] All environment variables properly prefixed
- [ ] TypeScript strict mode passes
- [ ] ESLint shows no errors
- [ ] All strings use i18n (no hardcoded text)
- [ ] Error handling implemented
- [ ] Logging includes environment context
- [ ] No commits of `.env`, `node_modules/`, or `*.db`
- [ ] Tested in Codespaces environment
- [ ] Performance targets met

---

**Version:** 1.0
**Last Updated:** 2025-11-05
**Framework:** D.E.C.I.D.E

---

**End of Guardrails Document**
