# Acceptance Criteria: Atlanta FIFA Navigator
> Feature Verification & Success Metrics
> Version 1.0 | Generated using MAP Framework

---

## Framework: M.A.P (Metrics, Actions, Proof)

This document defines **how we measure success** for every feature of the Atlanta FIFA Navigator application.

---

## Overall Success Metrics

### Quantitative Metrics

| Metric | Target | Critical Threshold |
|--------|--------|-------------------|
| Active Users (during FIFA events) | 100,000+ | 50,000 minimum |
| User Satisfaction Score | ≥ 90% | ≥ 80% |
| Application Crash Rate | < 2% | < 5% |
| Bilingual Feature Usage | ≥ 50% | ≥ 30% |
| Page Load Time (LCP) | < 2.5s | < 4s |
| API Uptime | 99.9% | 99.5% |
| Time to First Byte | < 500ms | < 1000ms |

### Qualitative Metrics

- User can successfully navigate to FIFA events
- Map is interactive and displays real-time data
- Language switching is seamless
- Transit information is accurate and timely
- Error messages are clear and actionable

---

## Organism MS-01: Core Application Setup

### Molecule MS-01.01: Project Initialization

**Metrics:**
- Build time: < 2 minutes
- Installation time: < 5 minutes
- Zero errors on fresh install

**Actions:**
1. Clone repository
2. Run `pnpm install`
3. Set up environment variables
4. Run `pnpm dev`
5. Navigate to `localhost:3000`

**Proof - Acceptance Criteria:**

**Given** a new GitHub Codespace is launched
**When** the environment completes setup
**Then:**
- [ ] All dependencies install successfully without errors
- [ ] `node_modules/` folder is created with all packages
- [ ] `.next/` folder is created after first build
- [ ] No TypeScript compilation errors
- [ ] No ESLint errors
- [ ] `pnpm dev` starts successfully on port 3000
- [ ] Next.js development server shows "Ready in Xms"
- [ ] Browser can connect to forwarded port 3000
- [ ] Homepage loads within 3 seconds
- [ ] No console errors in browser developer tools

**Verification Commands:**
```bash
# Verify installation
pnpm --version  # Should show 8.x
node --version  # Should show v20.x

# Verify build
pnpm run build  # Should complete without errors
pnpm run lint   # Should pass with no errors

# Verify development server
pnpm dev        # Should start on port 3000
```

---

### Molecule MS-01.02: Database Setup

**Metrics:**
- Database creation time: < 30 seconds
- Migration success rate: 100%
- Seed data insertion: < 10 seconds

**Actions:**
1. Initialize Prisma
2. Create database schema
3. Run migrations
4. Seed sample data

**Proof - Acceptance Criteria:**

**Given** Prisma is configured in the project
**When** database setup commands are run
**Then:**
- [ ] `prisma/schema.prisma` file exists with all models
- [ ] `prisma migrate dev` completes successfully
- [ ] SQLite database file `dev.db` is created
- [ ] All 4 tables are created (Event, Venue, User, TransitCache)
- [ ] `prisma db seed` populates sample data
- [ ] At least 3 FIFA matches are seeded
- [ ] At least 1 venue (Mercedes-Benz Stadium) is seeded
- [ ] Prisma Client is generated without errors
- [ ] Can query data using Prisma Studio: `pnpm prisma studio`

**Verification Commands:**
```bash
# Verify Prisma setup
pnpm prisma generate
pnpm prisma migrate dev --name init
pnpm prisma db seed

# Verify data exists
pnpm prisma studio  # Open GUI and check data
```

**Database Schema Validation:**
- [ ] Events table has foreign key to Venues
- [ ] All required fields are NOT NULL
- [ ] Indexes are created on venueId and matchDate
- [ ] Default values are set correctly

---

### Molecule MS-01.03: Environment Variables Configuration

**Metrics:**
- All required variables defined
- No plaintext secrets in code
- Zero security violations

**Actions:**
1. Create `.env` file from `.env.example`
2. Add all required API keys
3. Verify no keys are committed to git

**Proof - Acceptance Criteria:**

**Given** the project requires external API keys
**When** environment variables are configured
**Then:**
- [ ] `.env.example` file exists with all variable names
- [ ] `.env.example` contains placeholder values (not real keys)
- [ ] `.env` file is created locally
- [ ] `.env` is listed in `.gitignore`
- [ ] All required variables are documented in README
- [ ] `NEXT_PUBLIC_*` prefix used for client-side variables
- [ ] Server-side keys do NOT use `NEXT_PUBLIC_` prefix
- [ ] Running `git status` does NOT show `.env` file
- [ ] No API keys appear in `git log` history

**Required Variables Checklist:**
- [ ] `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY` (client-side)
- [ ] `MARTA_API_KEY` (server-side)
- [ ] `MARTA_TRAIN_API_KEY` (server-side)
- [ ] `DATABASE_URL` (server-side)
- [ ] `NEXT_PUBLIC_STADIUM_NAME` (client-side)
- [ ] `NEXT_PUBLIC_STADIUM_LAT` (client-side)
- [ ] `NEXT_PUBLIC_STADIUM_LNG` (client-side)
- [ ] `USE_MOCK_MARTA_DATA` (optional flag)

**Security Validation:**
```bash
# Verify .env is ignored
git status  # Should NOT show .env

# Verify no keys in code
grep -r "AIza" src/  # Should return no results
grep -r "sk-" src/   # Should return no results
```

---

## Organism MS-02: Google Maps Integration

### Molecule MS-02.01: Basic Map Display

**Metrics:**
- Map load time: < 3 seconds
- Map tile rendering: 100% complete
- Zero JavaScript errors

**Actions:**
1. User opens homepage
2. Map component renders
3. Map tiles load
4. Map becomes interactive

**Proof - Acceptance Criteria:**

**Given** the application is running
**When** a user navigates to the homepage
**Then:**
- [ ] A full-screen Google Map is visible
- [ ] Map is centered on Atlanta (lat: 33.749, lng: -84.388)
- [ ] Default zoom level is 12
- [ ] All map tiles load successfully (no gray squares)
- [ ] Map is interactive (user can pan with mouse)
- [ ] Map is interactive (user can zoom with scroll wheel)
- [ ] Map controls are visible (zoom buttons, fullscreen)
- [ ] No errors in browser console
- [ ] No "Invalid API key" warnings
- [ ] Map loads in under 3 seconds

**Visual Verification:**
- [ ] Atlanta downtown area is clearly visible
- [ ] Mercedes-Benz Stadium is visible on the map
- [ ] Street names are legible at default zoom
- [ ] Map styling is consistent with Google Maps standard theme

**Accessibility:**
- [ ] Map has proper ARIA labels
- [ ] Keyboard navigation works (tab to controls)
- [ ] Screen reader announces "Map region"

---

### Molecule MS-02.02: Traffic Layer

**Metrics:**
- Traffic data loads within 2 seconds
- Traffic colors are accurate
- Updates every 5 minutes

**Actions:**
1. Map loads successfully
2. Traffic layer activates
3. Real-time traffic displayed

**Proof - Acceptance Criteria:**

**Given** the map is loaded
**When** the traffic layer is enabled
**Then:**
- [ ] Traffic overlay appears on roads
- [ ] Green lines indicate light traffic
- [ ] Yellow/orange lines indicate moderate traffic
- [ ] Red lines indicate heavy traffic
- [ ] Traffic data updates automatically
- [ ] Traffic layer can be toggled off
- [ ] No performance degradation with traffic enabled
- [ ] Traffic data is current (within 5 minutes)

---

### Molecule MS-02.03: Venue Markers

**Metrics:**
- All venues displayed correctly
- Click interaction works 100%
- Info windows load < 1 second

**Actions:**
1. Map loads with venue data
2. User sees venue markers
3. User clicks on a marker
4. Info window appears

**Proof - Acceptance Criteria:**

**Given** venues exist in the database
**When** the map is displayed
**Then:**
- [ ] Mercedes-Benz Stadium marker is visible
- [ ] Marker is positioned at correct coordinates
- [ ] Marker icon is distinct and recognizable
- [ ] Clicking marker shows venue information
- [ ] Info window displays: venue name, address, capacity
- [ ] Clicking "Get Directions" button works (future feature)
- [ ] Multiple markers don't overlap
- [ ] Markers remain visible when panning/zooming

**Data Validation:**
- [ ] Marker lat/lng matches database values
- [ ] All seeded venues have markers
- [ ] No duplicate markers

---

## Organism MS-03: MARTA Transit Integration

### Molecule MS-03.01: Bus API Integration

**Metrics:**
- API response time: < 5 seconds
- Data refresh rate: every 30 seconds
- Error rate: < 1%

**Actions:**
1. Application requests bus data from `/api/transit?type=bus`
2. API fetches from MARTA Bus API
3. Response is parsed and returned
4. Bus markers appear on map

**Proof - Acceptance Criteria:**

**Given** the MARTA Bus API is accessible
**When** the `/api/transit` endpoint is called
**Then:**
- [ ] API returns HTTP 200 status
- [ ] Response is valid JSON
- [ ] Response contains `buses` array
- [ ] Each bus has: `id`, `type`, `routeId`, `latitude`, `longitude`
- [ ] Response includes `timestamp` field
- [ ] Response includes `source` field ("live" or "mock")
- [ ] Response time is under 5 seconds
- [ ] Bus data updates automatically every 30 seconds
- [ ] No console errors during fetch

**Data Validation:**
- [ ] Bus IDs are unique
- [ ] Latitude is between 33.6 and 33.9 (Atlanta area)
- [ ] Longitude is between -84.6 and -84.2 (Atlanta area)
- [ ] Route IDs match MARTA route format

**Error Handling:**
- [ ] Invalid API key returns graceful error (not 500)
- [ ] Timeout returns empty array (not crash)
- [ ] Network error returns empty array
- [ ] Source field indicates "error" on failure

**Manual Test:**
```bash
curl http://localhost:3000/api/transit?type=bus

# Expected response:
{
  "buses": [
    {
      "id": "bus-001",
      "type": "bus",
      "routeId": "12",
      "latitude": 33.755,
      "longitude": -84.403,
      "heading": 45,
      "lastUpdate": "2025-11-05T19:00:00Z"
    }
  ],
  "trains": [],
  "timestamp": "2025-11-05T19:00:00Z",
  "source": "live"
}
```

---

### Molecule MS-03.02: Train API Integration with Codespaces Proxy

**Metrics:**
- Proxy detection: 100% accurate
- Proxy response time: < 8 seconds
- Direct connection time: < 3 seconds

**Actions:**
1. Application detects environment (Codespaces vs Production)
2. Uses appropriate API URL (proxy vs direct)
3. Fetches train data
4. Parses GTFS-RT binary format

**Proof - Acceptance Criteria:**

**Given** the application is running in GitHub Codespaces
**When** the `/api/transit` endpoint fetches train data
**Then:**
- [ ] Environment detection identifies Codespaces correctly
- [ ] Console logs show "Using proxy for MARTA Train API"
- [ ] Request goes to `https://api.allorigins.win/raw?url=...`
- [ ] Proxy URL contains encoded MARTA Train API endpoint
- [ ] Response is successfully parsed from binary GTFS-RT
- [ ] Train data is returned in JSON format
- [ ] Response time is under 8 seconds

**Given** the application is running in production (Vercel)
**When** the `/api/transit` endpoint fetches train data
**Then:**
- [ ] Environment detection identifies production correctly
- [ ] Console logs show "Direct connection to MARTA Train API"
- [ ] Request goes directly to `http://developer.itsmarta.com:18096`
- [ ] No proxy is used
- [ ] Response time is under 3 seconds

**Environment Detection Validation:**
```bash
# In Codespaces
echo $CODESPACES  # Should output "true"

# In production
echo $CODESPACES  # Should be empty or undefined
```

**Code Verification:**
- [ ] `isRunningInCodespaces()` function exists
- [ ] Function checks `process.env.CODESPACES === 'true'`
- [ ] `getMartaTrainApiUrl()` returns correct URL for environment
- [ ] URL encoding is applied: `encodeURIComponent(directUrl)`
- [ ] Fallback logic handles proxy failures

**GTFS-RT Parsing:**
- [ ] Binary protobuf is decoded successfully
- [ ] FeedMessage structure is valid
- [ ] Vehicle positions are extracted
- [ ] Latitude/longitude values are correct
- [ ] Route IDs match MARTA train lines (RED, GOLD, BLUE, GREEN)

---

### Molecule MS-03.03: Transit Marker Display

**Metrics:**
- Marker update latency: < 500ms
- Animation smoothness: 60 FPS
- All vehicles visible: 100%

**Actions:**
1. Transit data is fetched
2. Markers are rendered on map
3. Positions update every 30 seconds
4. Markers animate smoothly

**Proof - Acceptance Criteria:**

**Given** transit data is available
**When** the map displays transit vehicles
**Then:**
- [ ] Bus markers appear on the map
- [ ] Train markers appear on the map
- [ ] Each marker has a distinct icon (🚌 for bus, 🚇 for train)
- [ ] Markers are color-coded by route
- [ ] Clicking a marker shows vehicle details
- [ ] Details include: route ID, heading, speed, last update time
- [ ] Markers update position every 30 seconds
- [ ] Markers animate smoothly (no "jumping")
- [ ] Animation duration is ~2 seconds
- [ ] Animation uses easing (not linear)
- [ ] No performance lag during animation
- [ ] Markers rotate to show heading direction (if available)

**Animation Quality:**
- [ ] Markers move smoothly between positions
- [ ] No frame drops during animation
- [ ] Easing function is cubic ease-out
- [ ] `requestAnimationFrame` is used (not `setInterval`)
- [ ] Previous positions are cached for interpolation

**Edge Cases:**
- [ ] Marker doesn't animate if position unchanged
- [ ] Animation cancels if new update arrives
- [ ] Markers don't overlap excessively
- [ ] Markers are removed when vehicle goes offline

---

## Organism MS-04: Event Schedule Display

### Molecule MS-04.01: Event List Component

**Metrics:**
- All events displayed: 100%
- Sort order correct: 100%
- Load time: < 1 second

**Actions:**
1. User views homepage
2. Event list sidebar loads
3. Events are displayed in order

**Proof - Acceptance Criteria:**

**Given** FIFA matches exist in the database
**When** the event list component renders
**Then:**
- [ ] All events are displayed
- [ ] Events are sorted by date ascending (earliest first)
- [ ] Each event shows: home team, away team, venue, date, time
- [ ] Event cards are visually distinct
- [ ] Date is formatted correctly for current locale
- [ ] Time is formatted correctly (12-hour for EN, 24-hour for others)
- [ ] Venue name is clickable
- [ ] No duplicate events appear
- [ ] Empty state message appears if no events

**Data Display:**
- [ ] Team names are spelled correctly
- [ ] Venue matches database entry
- [ ] Dates are in the future (or recent past for testing)
- [ ] Times are in local timezone (Atlanta ET)

**Interaction:**
- [ ] Clicking event highlights venue on map
- [ ] Clicking event centers map on venue
- [ ] Clicking event zooms map appropriately
- [ ] Hover effect on event cards
- [ ] Selected event has visual indicator

---

### Molecule MS-04.02: Event Filtering (Future Feature)

**Metrics:**
- Filter response time: < 100ms
- Filter accuracy: 100%

**Proof - Acceptance Criteria:**

**Given** multiple events exist
**When** a user applies filters
**Then:**
- [ ] Events can be filtered by date range
- [ ] Events can be filtered by venue
- [ ] Events can be filtered by team
- [ ] Filter results are instant (< 100ms)
- [ ] "Clear filters" button resets view
- [ ] Filter state is preserved on language change

---

## Organism MS-05: Internationalization (i18n)

### Molecule MS-05.01: Multi-language Routing

**Metrics:**
- Language switch time: < 500ms
- Translation coverage: 100%
- URL structure correct: 100%

**Actions:**
1. User opens `/en` (English)
2. User clicks language switcher
3. URL changes to `/es` (Spanish)
4. Page content updates

**Proof - Acceptance Criteria:**

**Given** the application supports 4 languages
**When** a user accesses different language routes
**Then:**
- [ ] `/en` route loads English version
- [ ] `/es` route loads Spanish version
- [ ] `/de` route loads German version
- [ ] `/ko` route loads Korean version
- [ ] Root `/` redirects to `/en` (default)
- [ ] Invalid locale `/xx` redirects to `/en`
- [ ] Language persists across page navigation
- [ ] URL structure is `/[lang]/[page]` format

**Language Switcher:**
- [ ] Switcher displays all 4 language options
- [ ] Current language is visually highlighted
- [ ] Each option shows: flag emoji, language code, native name
- [ ] Clicking language option changes URL immediately
- [ ] Page content updates without full reload
- [ ] Switcher is visible on all pages
- [ ] Switcher is accessible via keyboard (tab, enter)

**URL Behavior:**
- [ ] `/en` → English homepage
- [ ] `/es` → Spanish homepage
- [ ] `/de` → German homepage
- [ ] `/ko` → Korean homepage
- [ ] Switching from `/en` to `/es` preserves page path
- [ ] Query parameters are preserved on language change

---

### Molecule MS-05.02: Translation Coverage

**Metrics:**
- Translation files exist for all 4 languages
- All strings are translated: 100%
- No missing translation keys

**Actions:**
1. Audit all UI text
2. Verify translation files
3. Test each language

**Proof - Acceptance Criteria:**

**Given** translation files exist in `/public/locales/`
**When** each language is activated
**Then:**
- [ ] All UI strings are translated (no English fallbacks)
- [ ] Navigation labels are translated
- [ ] Button text is translated
- [ ] Error messages are translated
- [ ] Loading states are translated
- [ ] Empty state messages are translated
- [ ] Accessibility labels are translated

**Translation Files:**
- [ ] `/public/locales/en/common.json` exists
- [ ] `/public/locales/es/common.json` exists
- [ ] `/public/locales/de/common.json` exists
- [ ] `/public/locales/ko/common.json` exists
- [ ] All files have identical keys (structure matches)
- [ ] No missing keys across languages
- [ ] No extra keys in any language

**Content Validation:**
- [ ] Spanish translations are grammatically correct
- [ ] German translations use proper capitalization
- [ ] Korean translations use appropriate formality level
- [ ] No machine translation artifacts (e.g., literal translations)

---

### Molecule MS-05.03: Locale-aware Formatting

**Metrics:**
- Date formatting correct for all locales
- Number formatting correct for all locales

**Actions:**
1. Display dates in different languages
2. Display times in different languages
3. Display numbers in different languages

**Proof - Acceptance Criteria:**

**Given** a FIFA match on June 15, 2026 at 19:00
**When** displayed in each language
**Then:**

**English (en):**
- [ ] Date: "Monday, June 15, 2026"
- [ ] Time: "7:00 PM"
- [ ] Numbers: "71,000" (comma separator)

**Spanish (es):**
- [ ] Date: "lunes, 15 de junio de 2026"
- [ ] Time: "19:00"
- [ ] Numbers: "71.000" (period separator)

**German (de):**
- [ ] Date: "Montag, 15. Juni 2026"
- [ ] Time: "19:00"
- [ ] Numbers: "71.000" (period separator)

**Korean (ko):**
- [ ] Date: "2026년 6월 15일 월요일"
- [ ] Time: "오후 7:00" or "19:00"
- [ ] Numbers: "71,000"

**Implementation Validation:**
- [ ] Uses `Intl.DateTimeFormat` for dates
- [ ] Uses `Intl.DateTimeFormat` for times
- [ ] Uses `Intl.NumberFormat` for numbers
- [ ] Locale passed correctly to formatters

---

## Organism MS-06: Performance & Optimization

### Molecule MS-06.01: Core Web Vitals

**Metrics:**
- LCP (Largest Contentful Paint): < 2.5s
- FID (First Input Delay): < 100ms
- CLS (Cumulative Layout Shift): < 0.1

**Actions:**
1. Load homepage
2. Measure Core Web Vitals
3. Verify targets are met

**Proof - Acceptance Criteria:**

**Given** the application is deployed
**When** measured with Lighthouse or PageSpeed Insights
**Then:**
- [ ] LCP is under 2.5 seconds (target: < 2s)
- [ ] FID is under 100ms
- [ ] CLS is under 0.1
- [ ] Performance score is ≥ 90
- [ ] Accessibility score is ≥ 90
- [ ] Best Practices score is ≥ 90
- [ ] SEO score is ≥ 90

**Load Time Breakdown:**
- [ ] TTFB (Time to First Byte): < 500ms
- [ ] FCP (First Contentful Paint): < 2s
- [ ] Speed Index: < 3s
- [ ] Time to Interactive: < 3.5s

**Measurement Tools:**
```bash
# Using Lighthouse CLI
pnpm lighthouse http://localhost:3000 --view

# Expected scores:
# Performance: ≥ 90
# Accessibility: ≥ 90
# Best Practices: ≥ 90
# SEO: ≥ 90
```

---

### Molecule MS-06.02: Caching Strategy

**Metrics:**
- Cache hit rate: > 80%
- Stale data tolerance: < 30 seconds
- Cache invalidation: correct 100%

**Actions:**
1. Initial data fetch (cache miss)
2. Subsequent fetch (cache hit)
3. Wait for revalidation period
4. Data refreshes automatically

**Proof - Acceptance Criteria:**

**Given** the application uses caching
**When** data is fetched multiple times
**Then:**

**Transit API Caching:**
- [ ] First request fetches from MARTA API
- [ ] Response is cached for 30 seconds
- [ ] Subsequent requests use cached data
- [ ] Cache expires after 30 seconds
- [ ] Next request triggers new fetch
- [ ] Stale data is served while revalidating (SWR)

**Events API Caching:**
- [ ] Events are cached for 5 minutes
- [ ] Cached data is served instantly
- [ ] Cache invalidates after 5 minutes
- [ ] Cache updates on manual refresh

**Client-side Caching (SWR):**
- [ ] Data is cached in memory
- [ ] Refreshes every 30 seconds
- [ ] Deduplicates simultaneous requests
- [ ] Revalidates on focus (configurable)

**Cache Headers Validation:**
```bash
curl -I http://localhost:3000/api/transit

# Expected headers:
Cache-Control: s-maxage=30, stale-while-revalidate=60
```

---

## Organism MS-07: Error Handling & Resilience

### Molecule MS-07.01: API Error Handling

**Metrics:**
- Error detection: 100%
- Graceful degradation: 100%
- User-facing errors: clear and actionable

**Actions:**
1. Simulate API failure
2. Verify error handling
3. Check user experience

**Proof - Acceptance Criteria:**

**Given** the MARTA API returns an error
**When** the application attempts to fetch data
**Then:**
- [ ] Error is caught and logged (server-side)
- [ ] Empty array is returned (not 500 error)
- [ ] User sees friendly message "Unable to load transit data"
- [ ] Map continues to function
- [ ] Other features are not affected
- [ ] Automatic retry after 30 seconds
- [ ] Source field indicates "error"

**Error Scenarios:**

**Invalid API Key:**
- [ ] Detects 401 Unauthorized
- [ ] Logs error with context
- [ ] Returns empty transit data
- [ ] Shows user-friendly message

**API Timeout:**
- [ ] Request times out after 10 seconds
- [ ] Does not crash application
- [ ] Returns empty array
- [ ] Logs timeout event

**Network Error:**
- [ ] Handles fetch failures
- [ ] Does not expose error stack to user
- [ ] Provides retry mechanism
- [ ] Falls back to mock data if configured

**Invalid Response Format:**
- [ ] Validates response schema with Zod
- [ ] Rejects invalid data
- [ ] Logs validation errors
- [ ] Returns empty array

---

### Molecule MS-07.02: User-Facing Error Messages

**Metrics:**
- Error clarity: user understands issue
- Error actionability: user knows what to do
- Error localization: translated to all languages

**Proof - Acceptance Criteria:**

**Given** an error occurs
**When** displayed to the user
**Then:**
- [ ] Error message is in current language
- [ ] Message is clear and non-technical
- [ ] Message suggests action (e.g., "Try again later")
- [ ] No stack traces are shown
- [ ] No internal error codes are exposed
- [ ] Contact information provided (if applicable)

**Example Error Messages:**

**Map Load Failure:**
- EN: "Unable to load map. Please check your internet connection."
- ES: "No se puede cargar el mapa. Verifique su conexión a Internet."
- DE: "Karte kann nicht geladen werden. Bitte überprüfen Sie Ihre Internetverbindung."
- KO: "지도를 로드할 수 없습니다. 인터넷 연결을 확인하세요."

**Transit Data Failure:**
- [ ] Message explains transit data is temporarily unavailable
- [ ] Indicates automatic retry is happening
- [ ] Does not blame user

---

## Organism MS-08: Security & Compliance

### Molecule MS-08.01: API Key Security

**Metrics:**
- No keys exposed in client code: 100%
- No keys in git history: 100%
- Proper key prefixing: 100%

**Proof - Acceptance Criteria:**

**Given** the application uses external APIs
**When** security is audited
**Then:**
- [ ] No API keys visible in browser DevTools
- [ ] No API keys in client-side JavaScript bundle
- [ ] `.env` file is in `.gitignore`
- [ ] `.env` is not present in git history
- [ ] Server-side keys never sent to client
- [ ] Client-side keys use `NEXT_PUBLIC_` prefix
- [ ] No keys in source code (all in env vars)

**Security Audit Commands:**
```bash
# Check git history for keys
git log --all -p | grep -i "api.key"  # Should return nothing
git log --all -p | grep "AIza"        # Should return nothing

# Check client bundle
pnpm build
grep -r "MARTA_API_KEY" .next/        # Should return nothing

# Check source code
grep -r "AIza" src/                    # Should return nothing
grep -r "sk-" src/                     # Should return nothing
```

---

### Molecule MS-08.02: Input Validation

**Metrics:**
- All inputs validated: 100%
- SQL injection attempts blocked: 100%
- XSS attempts blocked: 100%

**Proof - Acceptance Criteria:**

**Given** the application accepts user input
**When** input is processed
**Then:**
- [ ] All query parameters are validated with Zod
- [ ] Invalid inputs return 400 Bad Request
- [ ] SQL injection attempts are rejected
- [ ] XSS payloads are sanitized
- [ ] Input length limits are enforced
- [ ] Special characters are handled safely

**Test Cases:**

**Query Parameter Validation:**
```bash
# Valid request
curl "http://localhost:3000/api/transit?type=bus"
# Returns 200

# Invalid type
curl "http://localhost:3000/api/transit?type=invalid"
# Returns 400 with error message

# SQL injection attempt
curl "http://localhost:3000/api/transit?type=bus';DROP TABLE events;--"
# Returns 400, no SQL executed

# XSS attempt
curl "http://localhost:3000/api/transit?type=<script>alert(1)</script>"
# Returns 400 or sanitized
```

---

### Molecule MS-08.03: WCAG 2.1 AA Compliance

**Metrics:**
- Accessibility score: ≥ 90
- Keyboard navigation: 100% functional
- Screen reader compatibility: 100%

**Proof - Acceptance Criteria:**

**Given** the application should be accessible
**When** tested with accessibility tools
**Then:**
- [ ] All interactive elements are keyboard accessible
- [ ] Tab order is logical
- [ ] Focus indicators are visible
- [ ] Color contrast ratio ≥ 4.5:1 for text
- [ ] All images have alt text
- [ ] All form inputs have labels
- [ ] ARIA labels are present on custom controls
- [ ] No keyboard traps
- [ ] Skip navigation link exists
- [ ] Screen reader announces page changes
- [ ] Error messages are announced

**Keyboard Navigation:**
- [ ] Tab key cycles through all interactive elements
- [ ] Enter key activates buttons
- [ ] Escape key closes modals/popups
- [ ] Arrow keys work in dropdown menus
- [ ] Map can be navigated with keyboard

**Screen Reader Testing:**
- [ ] VoiceOver (macOS): announces all content
- [ ] NVDA (Windows): announces all content
- [ ] Map region is announced
- [ ] Button purposes are announced
- [ ] Form labels are associated correctly

---

## Cross-Functional Acceptance Criteria

### Browser Compatibility

**Supported Browsers:**
- [ ] Chrome 90+ (100% functionality)
- [ ] Firefox 88+ (100% functionality)
- [ ] Safari 14+ (100% functionality)
- [ ] Edge 90+ (100% functionality)

**Mobile Browsers:**
- [ ] iOS Safari 14+ (100% functionality)
- [ ] Chrome Mobile (100% functionality)

**Unsupported:**
- [ ] IE11 (graceful degradation or unsupported message)

---

### Responsive Design

**Breakpoints:**
- [ ] Mobile (320px - 767px): Single column layout
- [ ] Tablet (768px - 1023px): Adjusted sidebar
- [ ] Desktop (1024px+): Full sidebar + map

**Mobile Experience:**
- [ ] Map is full-screen on mobile
- [ ] Event list is collapsible drawer
- [ ] Language switcher is accessible
- [ ] Touch gestures work (pinch to zoom)
- [ ] No horizontal scrolling

---

### Device Testing

**Tested on:**
- [ ] iPhone 12+ (iOS 15+)
- [ ] Samsung Galaxy S21+ (Android 11+)
- [ ] iPad Pro (iPadOS 15+)
- [ ] MacBook Pro (Chrome, Safari, Firefox)
- [ ] Windows PC (Chrome, Edge, Firefox)

---

## Deployment Acceptance Criteria

### Pre-Deployment Checklist

Before deploying to production:
- [ ] All environment variables set in Vercel
- [ ] Database migrations run successfully
- [ ] Build completes without errors
- [ ] All linting passes
- [ ] TypeScript compilation successful
- [ ] No console errors in production build
- [ ] All API keys are valid
- [ ] `.env` file is NOT deployed

### Post-Deployment Validation

After deploying to Vercel:
- [ ] Production URL is accessible
- [ ] Homepage loads successfully
- [ ] Map displays correctly
- [ ] Transit data loads (direct connection, not proxy)
- [ ] All 4 languages work
- [ ] Events display correctly
- [ ] No console errors in production
- [ ] Performance metrics meet targets
- [ ] SSL certificate is valid
- [ ] Custom domain configured (if applicable)

---

## Final Acceptance Sign-off

### Definition of Done

The Atlanta FIFA Navigator project is **COMPLETE** when:

1. **All Organisms are Complete:**
   - [ ] MS-01: Core Application Setup ✅
   - [ ] MS-02: Google Maps Integration ✅
   - [ ] MS-03: MARTA Transit Integration ✅
   - [ ] MS-04: Event Schedule Display ✅
   - [ ] MS-05: Internationalization ✅
   - [ ] MS-06: Performance & Optimization ✅
   - [ ] MS-07: Error Handling ✅
   - [ ] MS-08: Security & Compliance ✅

2. **All Success Metrics are Met:**
   - [ ] Performance targets achieved
   - [ ] Security audit passed
   - [ ] Accessibility compliance verified
   - [ ] All features functional in all languages

3. **Quality Gates Passed:**
   - [ ] Code review completed
   - [ ] Manual testing completed
   - [ ] Production deployment successful
   - [ ] Post-deployment validation passed

4. **Documentation Complete:**
   - [ ] README updated
   - [ ] API documentation written
   - [ ] Deployment guide created
   - [ ] User guide created (if applicable)

---

**Version:** 1.0
**Last Updated:** 2025-11-05
**Framework:** M.A.P (Metrics, Actions, Proof)

---

**End of Acceptance Criteria Document**
