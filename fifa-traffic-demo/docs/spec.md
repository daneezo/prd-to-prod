# Technical Specification: Atlanta FIFA Navigator
> Version 1.0 | Technical Specification Document
> Generated from PRD v1.0 and WBS v1.0

---

## 1. Executive Summary

The Atlanta FIFA Navigator is a server-side rendered Next.js application that provides real-time navigation, event information, and transit data for FIFA 2026 World Cup visitors in Atlanta. The application emphasizes multilingual support, real-time data integration, personalized experiences, and seamless user experience across devices.

**Key Technical Highlights:**
- Next.js 15 App Router with TypeScript
- Real-time MARTA transit integration (GTFS-RT)
- Google Maps with traffic visualization and multi-modal directions
- Rideshare integration (Uber/Lyft deep links)
- Real-time parking availability tracking
- Push notifications and geofenced alerts
- Personalized itineraries and recommendations
- Multi-language support (EN, ES, DE, KO)
- GitHub Codespaces-optimized development environment
- Vercel serverless deployment with Edge Functions

---

## 2. Technology Stack

### 2.1 Core Framework
| Technology | Version | Purpose |
|------------|---------|---------|
| Next.js | 15.x | React framework (App Router) |
| React | 19.x | UI component library |
| TypeScript | 5.x | Type-safe development |
| Node.js | 20.x LTS | Runtime environment |

### 2.2 Data & Storage
| Technology | Version | Purpose |
|------------|---------|---------|
| Prisma | 5.x | ORM and database migrations |
| SQLite | 3.x | Development database |
| PostgreSQL | 15.x | Production database (optional) |

### 2.3 UI & Styling
| Technology | Version | Purpose |
|------------|---------|---------|
| Tailwind CSS | 3.x | Utility-first CSS framework |
| @vis.gl/react-google-maps | 1.x | Google Maps React components |
| Headless UI | 2.x | Accessible UI components |

### 2.4 API & Data Integration
| Technology | Version | Purpose |
|------------|---------|---------|
| gtfs-realtime-bindings | 1.x | Parse MARTA GTFS-RT feeds |
| SWR | 2.x | Client-side data fetching |
| Zod | 3.x | Runtime type validation |
| @googlemaps/google-maps-services-js | 3.x | Directions API (walking, driving) |
| web-push | 3.x | Push notification service |

### 2.5 Internationalization
| Technology | Version | Purpose |
|------------|---------|---------|
| next-intl | 3.x | Next.js i18n routing & translations |

### 2.6 Development Tools
| Technology | Version | Purpose |
|------------|---------|---------|
| pnpm | 8.x | Package manager (REQUIRED) |
| ESLint | 8.x | Code linting |
| Prettier | 3.x | Code formatting |
| TypeScript ESLint | 6.x | TypeScript linting rules |

### 2.7 Deployment & Infrastructure
| Technology | Purpose |
|------------|---------|
| Vercel | Hosting and serverless functions |
| GitHub Codespaces | Primary development environment |
| GitHub Actions | CI/CD pipeline (optional) |

---

## 3. System Architecture

### 3.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Client Browser                        │
│  ┌────────────┐  ┌────────────┐  ┌──────────────────────┐  │
│  │ React UI   │  │ Google Maps│  │ Language Switcher    │  │
│  │ Components │  │ Integration│  │ (EN/ES/DE/KO)        │  │
│  └────────────┘  └────────────┘  └──────────────────────┘  │
└──────────────────────┬──────────────────────────────────────┘
                       │ HTTPS
┌──────────────────────▼──────────────────────────────────────┐
│              Next.js App (Vercel Edge)                       │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              App Router (/[lang]/...)                │   │
│  │  • Server Components (RSC)                           │   │
│  │  • Client Components (maps, animations)             │   │
│  │  • API Routes (/api/*)                              │   │
│  └─────────────────────────────────────────────────────┘   │
└──────────┬────────────────┬────────────────┬───────────────┘
           │                │                │
           │                │                │
┌──────────▼─────┐  ┌───────▼──────┐  ┌─────▼──────────────┐
│ Prisma DB      │  │ MARTA APIs   │  │ Google Maps API    │
│ (SQLite/PG)    │  │ • Bus GTFS-RT│  │ • Maps JS API      │
│ • Events       │  │ • Train API  │  │ • Places API       │
│ • Venues       │  │   (Port 18096│  │ • Directions API   │
│ • Users        │  │    Proxy in  │  │                    │
└────────────────┘  │    Codespaces│  └────────────────────┘
                    └──────────────┘
```

### 3.2 Request Flow

**Server-Side Rendering (SSR) Flow:**
```
1. User requests /en (English homepage)
2. Next.js App Router matches [lang] route
3. Server fetches initial data (events, venues)
4. Server renders React components to HTML
5. HTML + hydration JS sent to client
6. Client hydrates and becomes interactive
7. Client-side data fetching begins (transit updates)
```

**API Route Flow:**
```
1. Client requests /api/transit
2. API route detects environment (Codespaces vs Production)
3. If Codespaces: Use proxy for port 18096 (train API)
4. If Production: Direct connection to MARTA APIs
5. Parse GTFS-RT binary data
6. Transform to JSON format
7. Return standardized response
8. Client updates map markers with new positions
```

---

## 4. Data Models

### 4.1 Prisma Schema

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite" // Use "postgresql" for production
  url      = env("DATABASE_URL")
}

model Event {
  id          String   @id @default(cuid())
  homeTeam    String
  awayTeam    String
  venue       Venue    @relation(fields: [venueId], references: [id])
  venueId     String
  matchDate   DateTime
  matchTime   String   // Stored as HH:MM format
  description String?
  category    String   @default("FIFA_2026")
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([venueId])
  @@index([matchDate])
}

model Venue {
  id              String   @id @default(cuid())
  name            String
  address         String
  city            String   @default("Atlanta")
  state           String   @default("GA")
  zipCode         String?
  latitude        Float
  longitude       Float
  capacity        Int?
  amenities       String?  // JSON string of amenities (WiFi, food, restrooms, etc.)
  seatingChartUrl String?  // URL to seating chart image/PDF
  parkingInfo     String?  // JSON string of parking facilities
  accessibilityInfo String? // Wheelchair access, elevators, etc.
  nearbyAttractions String? // JSON array of nearby POIs
  events          Event[]
  parkingLots     ParkingLot[]
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt

  @@index([name])
}

model User {
  id                    String         @id @default(cuid())
  email                 String?        @unique
  languagePreference    String         @default("en") // en, es, de, ko
  favoritedTeams        String?        // JSON array of team names
  favoritedVenues       String?        // JSON array of venue IDs
  notificationsEnabled  Boolean        @default(true)
  pushSubscription      String?        // JSON string of push subscription object
  locationPermission    Boolean        @default(false)
  createdAt             DateTime       @default(now())
  updatedAt             DateTime       @updatedAt
  notifications         Notification[]
  itineraries           Itinerary[]
}

model TransitCache {
  id          String   @id @default(cuid())
  vehicleId   String   @unique
  vehicleType String   // "bus" or "train"
  routeId     String
  latitude    Float
  longitude   Float
  heading     Float?
  speed       Float?
  timestamp   DateTime @default(now())

  @@index([vehicleType])
  @@index([routeId])
  @@index([timestamp])
}

model ParkingLot {
  id                String   @id @default(cuid())
  name              String
  venueId           String?
  venue             Venue?   @relation(fields: [venueId], references: [id])
  address           String
  latitude          Float
  longitude         Float
  totalSpaces       Int
  availableSpaces   Int      @default(0)
  pricePerHour      Float?
  acceptsCash       Boolean  @default(true)
  acceptsCard       Boolean  @default(true)
  isAccessible      Boolean  @default(false)
  operatingHours    String?  // JSON string
  lastUpdated       DateTime @updatedAt
  createdAt         DateTime @default(now())

  @@index([venueId])
  @@index([availableSpaces])
}

model Notification {
  id          String   @id @default(cuid())
  userId      String?
  user        User?    @relation(fields: [userId], references: [id])
  type        String   // "event", "traffic", "transit", "parking", "general"
  title       String
  message     String
  priority    String   @default("normal") // "low", "normal", "high", "urgent"
  isRead      Boolean  @default(false)
  actionUrl   String?  // Deep link to relevant page
  metadata    String?  // JSON string for additional data
  expiresAt   DateTime?
  sentAt      DateTime @default(now())
  createdAt   DateTime @default(now())

  @@index([userId])
  @@index([isRead])
  @@index([sentAt])
}

model Itinerary {
  id          String   @id @default(cuid())
  userId      String
  user        User     @relation(fields: [userId], references: [id])
  name        String
  description String?
  items       String   // JSON array of itinerary items
  startDate   DateTime
  endDate     DateTime
  isActive    Boolean  @default(true)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([userId])
  @@index([startDate])
}

model GeofenceZone {
  id          String   @id @default(cuid())
  name        String
  type        String   // "venue", "traffic", "event", "transit_stop"
  centerLat   Float
  centerLng   Float
  radiusMeters Int
  alertMessage String
  priority    String   @default("normal")
  isActive    Boolean  @default(true)
  metadata    String?  // JSON string
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([type])
  @@index([isActive])
}

model AttractionPOI {
  id          String   @id @default(cuid())
  name        String
  category    String   // "restaurant", "bar", "fan_zone", "attraction", "hotel"
  description String?
  address     String
  latitude    Float
  longitude   Float
  rating      Float?
  priceLevel  Int?     // 1-4 scale
  hours       String?  // JSON string
  phone       String?
  website     String?
  imageUrl    String?
  tags        String?  // JSON array of tags
  isPartner   Boolean  @default(false) // Featured partners
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([category])
  @@index([isPartner])
}
```

### 4.2 TypeScript Interfaces

```typescript
// src/types/index.ts

export interface Event {
  id: string;
  homeTeam: string;
  awayTeam: string;
  venue: Venue;
  matchDate: Date;
  matchTime: string;
  description?: string;
  category: string;
}

export interface Venue {
  id: string;
  name: string;
  address: string;
  city: string;
  state: string;
  latitude: number;
  longitude: number;
  capacity?: number;
}

export interface TransitVehicle {
  id: string;
  type: 'bus' | 'train';
  routeId: string;
  routeName?: string;
  latitude: number;
  longitude: number;
  heading?: number;
  speed?: number;
  lastUpdate: Date;
}

export interface TransitApiResponse {
  buses: TransitVehicle[];
  trains: TransitVehicle[];
  timestamp: Date;
}

export interface UserPreferences {
  language: 'en' | 'es' | 'de' | 'ko';
  favoritedTeams?: string[];
  favoritedVenues?: string[];
  notificationsEnabled: boolean;
}

export interface MapBounds {
  north: number;
  south: number;
  east: number;
  west: number;
}

export interface GeofenceAlert {
  id: string;
  message: string;
  type: 'traffic' | 'event' | 'transit' | 'parking';
  location: { lat: number; lng: number };
  radius: number; // meters
  priority: 'low' | 'normal' | 'high' | 'urgent';
}

export interface ParkingLot {
  id: string;
  name: string;
  address: string;
  latitude: number;
  longitude: number;
  totalSpaces: number;
  availableSpaces: number;
  pricePerHour?: number;
  acceptsCash: boolean;
  acceptsCard: boolean;
  isAccessible: boolean;
  operatingHours?: Record<string, string>;
  distance?: number; // calculated from user location
}

export interface Notification {
  id: string;
  type: 'event' | 'traffic' | 'transit' | 'parking' | 'general';
  title: string;
  message: string;
  priority: 'low' | 'normal' | 'high' | 'urgent';
  isRead: boolean;
  actionUrl?: string;
  metadata?: Record<string, any>;
  expiresAt?: Date;
  sentAt: Date;
}

export interface ItineraryItem {
  id: string;
  type: 'event' | 'attraction' | 'dining' | 'transit' | 'custom';
  eventId?: string;
  poiId?: string;
  title: string;
  description?: string;
  location: { lat: number; lng: number };
  startTime: Date;
  endTime?: Date;
  notes?: string;
}

export interface Itinerary {
  id: string;
  name: string;
  description?: string;
  items: ItineraryItem[];
  startDate: Date;
  endDate: Date;
  isActive: boolean;
}

export interface AttractionPOI {
  id: string;
  name: string;
  category: 'restaurant' | 'bar' | 'fan_zone' | 'attraction' | 'hotel';
  description?: string;
  address: string;
  latitude: number;
  longitude: number;
  rating?: number;
  priceLevel?: number; // 1-4
  hours?: Record<string, string>;
  phone?: string;
  website?: string;
  imageUrl?: string;
  tags?: string[];
  isPartner: boolean;
  distance?: number; // calculated from user location
}

export interface DirectionsRequest {
  origin: { lat: number; lng: number };
  destination: { lat: number; lng: number };
  mode: 'DRIVING' | 'WALKING' | 'TRANSIT' | 'BICYCLING';
  alternatives?: boolean;
  departureTime?: Date;
}

export interface DirectionsResult {
  routes: Route[];
  availableTransportModes: string[];
}

export interface Route {
  summary: string;
  legs: RouteLeg[];
  duration: number; // seconds
  distance: number; // meters
  trafficDuration?: number; // with traffic
  warnings?: string[];
}

export interface RouteLeg {
  startAddress: string;
  endAddress: string;
  duration: number;
  distance: number;
  steps: RouteStep[];
}

export interface RouteStep {
  instruction: string;
  duration: number;
  distance: number;
  travelMode: string;
  polyline: string;
}

export interface RideshareOption {
  service: 'uber' | 'lyft';
  deepLink: string; // Deep link to app
  webLink: string;  // Fallback web URL
  estimatedPrice?: string;
  estimatedTime?: number; // minutes
}
```

---

## 5. API Endpoints

### 5.1 Transit Data API

**Endpoint:** `GET /api/transit`

**Purpose:** Returns real-time MARTA bus and train positions

**Query Parameters:**
- `type` (optional): Filter by "bus" or "train"
- `route` (optional): Filter by route ID

**Response:**
```typescript
{
  buses: TransitVehicle[],
  trains: TransitVehicle[],
  timestamp: string, // ISO 8601
  source: "live" | "mock" | "cached"
}
```

**Implementation Notes:**
- Uses environment detection for Codespaces proxy
- Implements 30-second cache with SWR revalidation
- Falls back to mock data on API failures
- Parses GTFS-RT binary format using gtfs-realtime-bindings

**Error Handling:**
- Returns empty arrays on failure (not 500 errors)
- Logs detailed error info server-side
- Client receives graceful degradation

---

### 5.2 Events API

**Endpoint:** `GET /api/events`

**Purpose:** Returns FIFA match schedule

**Query Parameters:**
- `from` (optional): ISO date string (start date filter)
- `to` (optional): ISO date string (end date filter)
- `venue` (optional): Venue ID filter

**Response:**
```typescript
{
  events: Event[],
  total: number
}
```

**Implementation Notes:**
- Data sourced from Prisma database
- Sorted by matchDate ascending
- Includes related venue data (joined)
- Cached for 5 minutes

---

### 5.3 Venues API

**Endpoint:** `GET /api/venues`

**Purpose:** Returns venue information

**Query Parameters:**
- `id` (optional): Specific venue ID
- `nearby` (optional): "lat,lng" - finds venues near coordinates

**Response:**
```typescript
{
  venues: Venue[]
}
```

---

### 5.4 User Preferences API

**Endpoint:** `GET /api/user`
**Purpose:** Get current user preferences (from session/localStorage)

**Endpoint:** `PATCH /api/user`
**Purpose:** Update user preferences

**Request Body:**
```typescript
{
  language?: 'en' | 'es' | 'de' | 'ko',
  favoritedTeams?: string[],
  favoritedVenues?: string[]
}
```

**Response:**
```typescript
{
  success: boolean,
  user: UserPreferences
}
```

---

### 5.5 Parking API

**Endpoint:** `GET /api/parking`

**Purpose:** Returns real-time parking availability near venues

**Query Parameters:**
- `venueId` (optional): Specific venue ID
- `lat` & `lng` (optional): User coordinates for proximity search
- `radius` (optional): Search radius in meters (default: 2000)

**Response:**
```typescript
{
  parkingLots: ParkingLot[],
  total: number,
  lastUpdated: string // ISO 8601
}
```

**Implementation Notes:**
- Parking data updated every 5 minutes
- Mock data for development (real-time integration planned for future)
- Sorts by distance when lat/lng provided
- Filters by availability (excludes full lots by default)

---

### 5.6 Directions API

**Endpoint:** `POST /api/directions`

**Purpose:** Get multi-modal directions between two points

**Request Body:**
```typescript
{
  origin: { lat: number, lng: number },
  destination: { lat: number, lng: number },
  mode: "DRIVING" | "WALKING" | "TRANSIT" | "BICYCLING",
  alternatives: boolean, // default: true
  departureTime?: string // ISO 8601
}
```

**Response:**
```typescript
{
  routes: Route[],
  availableTransportModes: string[],
  rideshareOptions?: RideshareOption[]
}
```

**Implementation Notes:**
- Uses Google Maps Directions API
- Returns MARTA-specific transit routes when mode=TRANSIT
- Includes traffic estimates for DRIVING mode
- Rideshare deep links generated for all requests

---

### 5.7 Attractions / Points of Interest API

**Endpoint:** `GET /api/attractions`

**Purpose:** Returns nearby attractions, restaurants, fan zones

**Query Parameters:**
- `lat` & `lng` (required): User or venue coordinates
- `radius` (optional): Search radius in meters (default: 1000)
- `category` (optional): Filter by category
- `limit` (optional): Max results (default: 20)

**Response:**
```typescript
{
  attractions: AttractionPOI[],
  total: number
}
```

**Implementation Notes:**
- Integrates with Google Places API
- Prioritizes partner locations (isPartner=true)
- Includes distance calculation from user location
- Cached for 1 hour

---

### 5.8 Notifications API

**Endpoint:** `GET /api/notifications`
**Purpose:** Get user notifications

**Query Parameters:**
- `unreadOnly` (optional): boolean

**Response:**
```typescript
{
  notifications: Notification[],
  unreadCount: number
}
```

**Endpoint:** `POST /api/notifications`
**Purpose:** Create notification (admin/system only)

**Request Body:**
```typescript
{
  userId?: string,
  type: string,
  title: string,
  message: string,
  priority: string,
  actionUrl?: string
}
```

**Endpoint:** `PATCH /api/notifications/:id/read`
**Purpose:** Mark notification as read

---

### 5.9 Push Subscription API

**Endpoint:** `POST /api/push/subscribe`
**Purpose:** Subscribe to push notifications

**Request Body:**
```typescript
{
  subscription: PushSubscription // Web Push API object
}
```

**Response:**
```typescript
{
  success: boolean
}
```

**Implementation Notes:**
- Uses web-push library
- VAPID keys configured server-side
- Subscription stored in user.pushSubscription

---

### 5.10 Itineraries API

**Endpoint:** `GET /api/itineraries`
**Purpose:** Get user's itineraries

**Response:**
```typescript
{
  itineraries: Itinerary[]
}
```

**Endpoint:** `POST /api/itineraries`
**Purpose:** Create new itinerary

**Request Body:**
```typescript
{
  name: string,
  description?: string,
  items: ItineraryItem[],
  startDate: string, // ISO 8601
  endDate: string
}
```

**Endpoint:** `PATCH /api/itineraries/:id`
**Purpose:** Update itinerary

**Endpoint:** `DELETE /api/itineraries/:id`
**Purpose:** Delete itinerary

---

### 5.11 Geofencing API

**Endpoint:** `GET /api/geofence/check`

**Purpose:** Check if user is within any active geofence zones

**Query Parameters:**
- `lat` & `lng` (required): User coordinates

**Response:**
```typescript
{
  alerts: GeofenceAlert[],
  triggeredZones: string[] // zone IDs
}
```

**Implementation Notes:**
- Server-side geofence calculation using Haversine formula
- Only returns active zones (isActive=true)
- Triggered zones sorted by priority
- Results cached for 1 minute per user location

---

### 5.12 Rideshare Links API

**Endpoint:** `GET /api/rideshare`

**Purpose:** Generate deep links for rideshare apps

**Query Parameters:**
- `dropoffLat` & `dropoffLng` (required): Destination coordinates
- `pickupLat` & `pickupLng` (optional): Pickup coordinates

**Response:**
```typescript
{
  uber: {
    deepLink: string,
    webLink: string
  },
  lyft: {
    deepLink: string,
    webLink: string
  }
}
```

**Implementation Notes:**
- Generates universal links for iOS/Android
- Falls back to web URLs if app not installed
- No API keys required (uses deep link protocol)

---

## 6. Component Architecture

### 6.1 Page Components

```
src/app/[lang]/
├── page.tsx              # Main application page
├── layout.tsx            # Language-aware root layout
├── loading.tsx           # Loading UI
├── error.tsx             # Error boundary
└── globals.css           # Global styles
```

**Main Page Component:**
```typescript
// src/app/[lang]/page.tsx
import { MapView } from '@/components/MapView';
import { EventList } from '@/components/EventList';
import { LanguageSwitcher } from '@/components/LanguageSwitcher';

export default async function HomePage({
  params
}: {
  params: { lang: string }
}) {
  // Server-side data fetching
  const events = await getEvents();
  const venues = await getVenues();

  return (
    <main className="h-screen flex flex-col">
      <header className="z-10 p-4 bg-white shadow">
        <LanguageSwitcher currentLang={params.lang} />
      </header>
      <div className="flex-1 flex">
        <aside className="w-96 overflow-y-auto">
          <EventList events={events} />
        </aside>
        <div className="flex-1">
          <MapView venues={venues} initialLang={params.lang} />
        </div>
      </div>
    </main>
  );
}
```

### 6.2 Map Components

**MapView Component (Client Component):**
```typescript
// src/components/MapView.tsx
'use client';

import { APIProvider, Map } from '@vis.gl/react-google-maps';
import { AnimatedTransitMarker } from './AnimatedTransitMarker';
import { useTransitData } from '@/hooks/useTransitData';

export function MapView({ venues, initialLang }: MapViewProps) {
  const { buses, trains } = useTransitData();

  return (
    <APIProvider apiKey={process.env.NEXT_PUBLIC_GOOGLE_MAPS_API_KEY!}>
      <Map
        defaultCenter={{ lat: 33.749, lng: -84.388 }}
        defaultZoom={12}
        mapId="atlanta-fifa-map"
        gestureHandling="greedy"
      >
        {/* Venue markers */}
        {venues.map(venue => (
          <VenueMarker key={venue.id} venue={venue} />
        ))}

        {/* Transit markers with animation */}
        {buses.map(bus => (
          <AnimatedTransitMarker
            key={bus.id}
            vehicle={bus}
            type="bus"
          />
        ))}

        {trains.map(train => (
          <AnimatedTransitMarker
            key={train.id}
            vehicle={train}
            type="train"
          />
        ))}
      </Map>
    </APIProvider>
  );
}
```

**AnimatedTransitMarker Component:**
```typescript
// src/components/AnimatedTransitMarker.tsx
'use client';

import { AdvancedMarker } from '@vis.gl/react-google-maps';
import { useEffect, useRef, useState } from 'react';

export function AnimatedTransitMarker({ vehicle, type }: Props) {
  const [position, setPosition] = useState({
    lat: vehicle.latitude,
    lng: vehicle.longitude
  });

  const previousPosition = useRef(position);

  // Smooth animation using requestAnimationFrame
  useEffect(() => {
    const start = Date.now();
    const duration = 2000; // 2 second animation
    const startPos = previousPosition.current;
    const endPos = { lat: vehicle.latitude, lng: vehicle.longitude };

    const animate = () => {
      const elapsed = Date.now() - start;
      const progress = Math.min(elapsed / duration, 1);

      // Ease-out interpolation
      const eased = 1 - Math.pow(1 - progress, 3);

      setPosition({
        lat: startPos.lat + (endPos.lat - startPos.lat) * eased,
        lng: startPos.lng + (endPos.lng - startPos.lng) * eased
      });

      if (progress < 1) {
        requestAnimationFrame(animate);
      } else {
        previousPosition.current = endPos;
      }
    };

    requestAnimationFrame(animate);
  }, [vehicle.latitude, vehicle.longitude]);

  return (
    <AdvancedMarker position={position}>
      <div className={`transit-marker ${type}`}>
        {type === 'bus' ? '🚌' : '🚇'}
      </div>
    </AdvancedMarker>
  );
}
```

### 6.3 Event Components

**EventList Component:**
```typescript
// src/components/EventList.tsx
import { EventCard } from './EventCard';

export function EventList({ events }: { events: Event[] }) {
  return (
    <div className="p-4 space-y-4">
      <h2 className="text-2xl font-bold">
        {/* Translated text */}
        FIFA Matches
      </h2>
      {events.map(event => (
        <EventCard key={event.id} event={event} />
      ))}
    </div>
  );
}
```

**EventCard Component:**
```typescript
// src/components/EventCard.tsx
export function EventCard({ event }: { event: Event }) {
  return (
    <div className="border rounded-lg p-4 hover:shadow-lg transition">
      <div className="flex justify-between items-start">
        <div>
          <h3 className="font-semibold">
            {event.homeTeam} vs {event.awayTeam}
          </h3>
          <p className="text-sm text-gray-600">{event.venue.name}</p>
        </div>
        <div className="text-right text-sm">
          <p>{formatDate(event.matchDate)}</p>
          <p>{event.matchTime}</p>
        </div>
      </div>
      <button
        onClick={() => centerMapOnVenue(event.venue)}
        className="mt-2 text-blue-600 hover:underline text-sm"
      >
        Show on map
      </button>
    </div>
  );
}
```

### 6.4 Language Switcher

**LanguageSwitcher Component:**
```typescript
// src/components/LanguageSwitcher.tsx
'use client';

import { useRouter, usePathname } from 'next/navigation';

export function LanguageSwitcher({ currentLang }: { currentLang: string }) {
  const router = useRouter();
  const pathname = usePathname();

  const languages = [
    { code: 'en', label: 'English', flag: '🇺🇸' },
    { code: 'es', label: 'Español', flag: '🇪🇸' },
    { code: 'de', label: 'Deutsch', flag: '🇩🇪' },
    { code: 'ko', label: '한국어', flag: '🇰🇷' }
  ];

  const switchLanguage = (newLang: string) => {
    const newPathname = pathname.replace(/^\/[a-z]{2}/, `/${newLang}`);
    router.push(newPathname);
  };

  return (
    <div className="flex gap-2">
      {languages.map(lang => (
        <button
          key={lang.code}
          onClick={() => switchLanguage(lang.code)}
          className={`px-3 py-1 rounded ${
            currentLang === lang.code
              ? 'bg-blue-600 text-white'
              : 'bg-gray-200'
          }`}
          aria-label={`Switch to ${lang.label}`}
        >
          {lang.flag} {lang.code.toUpperCase()}
        </button>
      ))}
    </div>
  );
}
```

### 6.5 Notification Components

**NotificationBell Component:**
```typescript
// src/components/NotificationBell.tsx
'use client';

import { useNotifications } from '@/hooks/useNotifications';
import { NotificationPanel } from './NotificationPanel';
import { useState } from 'react';

export function NotificationBell() {
  const { notifications, unreadCount, markAsRead } = useNotifications();
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div className="relative">
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="relative p-2 rounded-full hover:bg-gray-100"
        aria-label="Notifications"
      >
        🔔
        {unreadCount > 0 && (
          <span className="absolute top-0 right-0 bg-red-500 text-white text-xs rounded-full w-5 h-5 flex items-center justify-center">
            {unreadCount}
          </span>
        )}
      </button>

      {isOpen && (
        <NotificationPanel
          notifications={notifications}
          onClose={() => setIsOpen(false)}
          onMarkAsRead={markAsRead}
        />
      )}
    </div>
  );
}
```

### 6.6 Parking Components

**ParkingMarker Component:**
```typescript
// src/components/ParkingMarker.tsx
'use client';

import { AdvancedMarker, InfoWindow } from '@vis.gl/react-google-maps';
import { ParkingLot } from '@/types';
import { useState } from 'react';

export function ParkingMarker({ parkingLot }: { parkingLot: ParkingLot }) {
  const [showInfo, setShowInfo] = useState(false);

  const availabilityColor =
    parkingLot.availableSpaces > 50 ? 'green' :
    parkingLot.availableSpaces > 10 ? 'yellow' : 'red';

  return (
    <>
      <AdvancedMarker
        position={{ lat: parkingLot.latitude, lng: parkingLot.longitude }}
        onClick={() => setShowInfo(true)}
      >
        <div className={`parking-icon bg-${availabilityColor}-500`}>
          P
        </div>
      </AdvancedMarker>

      {showInfo && (
        <InfoWindow
          position={{ lat: parkingLot.latitude, lng: parkingLot.longitude }}
          onCloseClick={() => setShowInfo(false)}
        >
          <div className="p-2">
            <h3 className="font-semibold">{parkingLot.name}</h3>
            <p className="text-sm">{parkingLot.availableSpaces} / {parkingLot.totalSpaces} available</p>
            {parkingLot.pricePerHour && (
              <p className="text-sm">${parkingLot.pricePerHour}/hour</p>
            )}
          </div>
        </InfoWindow>
      )}
    </>
  );
}
```

### 6.7 Directions and Rideshare Components

**DirectionsPanel Component:**
```typescript
// src/components/DirectionsPanel.tsx
'use client';

import { useState } from 'react';
import { useDirections } from '@/hooks/useDirections';
import { RideshareOptions } from './RideshareOptions';

export function DirectionsPanel({ destination }: { destination: { lat: number; lng: number } }) {
  const [mode, setMode] = useState<'TRANSIT' | 'WALKING' | 'DRIVING'>('TRANSIT');
  const { routes, rideshareOptions, isLoading } = useDirections({
    destination,
    mode
  });

  return (
    <div className="p-4 bg-white shadow-lg rounded-lg">
      <h2 className="text-xl font-bold mb-4">Get Directions</h2>

      {/* Mode selector */}
      <div className="flex gap-2 mb-4">
        {['TRANSIT', 'WALKING', 'DRIVING'].map(m => (
          <button
            key={m}
            onClick={() => setMode(m as any)}
            className={`px-4 py-2 rounded ${
              mode === m ? 'bg-blue-600 text-white' : 'bg-gray-200'
            }`}
          >
            {m === 'TRANSIT' ? '🚇' : m === 'WALKING' ? '🚶' : '🚗'} {m}
          </button>
        ))}
      </div>

      {/* Route display */}
      {isLoading ? (
        <p>Loading routes...</p>
      ) : (
        <div className="space-y-4">
          {routes.map((route, idx) => (
            <div key={idx} className="border p-3 rounded">
              <p className="font-semibold">{route.summary}</p>
              <p className="text-sm text-gray-600">
                {Math.round(route.duration / 60)} min · {(route.distance / 1609).toFixed(1)} mi
              </p>
            </div>
          ))}
        </div>
      )}

      {/* Rideshare options */}
      <div className="mt-6">
        <h3 className="font-semibold mb-2">Or take a ride</h3>
        <RideshareOptions options={rideshareOptions} />
      </div>
    </div>
  );
}
```

### 6.8 Itinerary Components

**ItineraryBuilder Component:**
```typescript
// src/components/ItineraryBuilder.tsx
'use client';

import { useState } from 'react';
import { Itinerary, ItineraryItem } from '@/types';
import { useItineraries } from '@/hooks/useItineraries';

export function ItineraryBuilder() {
  const { itineraries, createItinerary, updateItinerary } = useItineraries();
  const [items, setItems] = useState<ItineraryItem[]>([]);

  const addItem = (item: ItineraryItem) => {
    setItems([...items, item]);
  };

  const saveItinerary = async () => {
    await createItinerary({
      name: 'My FIFA Trip',
      items,
      startDate: new Date(),
      endDate: new Date()
    });
  };

  return (
    <div className="p-4">
      <h2 className="text-2xl font-bold mb-4">Build Your Itinerary</h2>

      {/* Itinerary item list */}
      <div className="space-y-2">
        {items.map((item, idx) => (
          <div key={idx} className="border p-3 rounded flex justify-between">
            <div>
              <h3 className="font-semibold">{item.title}</h3>
              <p className="text-sm text-gray-600">
                {new Date(item.startTime).toLocaleTimeString()}
              </p>
            </div>
            <button
              onClick={() => setItems(items.filter((_, i) => i !== idx))}
              className="text-red-600 hover:text-red-800"
            >
              Remove
            </button>
          </div>
        ))}
      </div>

      <button
        onClick={saveItinerary}
        className="mt-4 px-6 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
      >
        Save Itinerary
      </button>
    </div>
  );
}
```

### 6.9 Attraction/POI Components

**AttractionCard Component:**
```typescript
// src/components/AttractionCard.tsx
import { AttractionPOI } from '@/types';

export function AttractionCard({ attraction }: { attraction: AttractionPOI }) {
  return (
    <div className="border rounded-lg overflow-hidden hover:shadow-lg transition">
      {attraction.imageUrl && (
        <img
          src={attraction.imageUrl}
          alt={attraction.name}
          className="w-full h-48 object-cover"
        />
      )}
      <div className="p-4">
        <div className="flex justify-between items-start">
          <h3 className="font-semibold text-lg">{attraction.name}</h3>
          {attraction.isPartner && (
            <span className="bg-blue-100 text-blue-800 text-xs px-2 py-1 rounded">
              Partner
            </span>
          )}
        </div>

        <p className="text-sm text-gray-600 mt-1">{attraction.category}</p>

        {attraction.rating && (
          <div className="flex items-center mt-2">
            <span className="text-yellow-500">⭐</span>
            <span className="ml-1 text-sm">{attraction.rating.toFixed(1)}</span>
            {attraction.priceLevel && (
              <span className="ml-2 text-sm text-gray-600">
                {'$'.repeat(attraction.priceLevel)}
              </span>
            )}
          </div>
        )}

        {attraction.distance && (
          <p className="text-sm text-gray-500 mt-2">
            {(attraction.distance / 1609).toFixed(1)} mi away
          </p>
        )}

        <p className="text-sm text-gray-700 mt-2 line-clamp-2">
          {attraction.description}
        </p>

        <button className="mt-3 text-blue-600 hover:underline text-sm">
          Get Directions
        </button>
      </div>
    </div>
  );
}
```

---

## 7. Notification & Geofencing Architecture

### 7.1 Push Notification System

**Architecture Overview:**
```
┌─────────────────────────────────────────────────────────────────┐
│                        Client Browser                            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Service Worker (sw.js)                                   │  │
│  │  • Receives push events                                   │  │
│  │  • Displays notifications                                 │  │
│  │  • Handles notification clicks                            │  │
│  └──────────────────────────────────────────────────────────┘  │
└───────────────────────┬─────────────────────────────────────────┘
                        │ Push Protocol
┌───────────────────────▼─────────────────────────────────────────┐
│              Push Service (FCM/APNS)                             │
└───────────────────────▲─────────────────────────────────────────┘
                        │ web-push library
┌───────────────────────┴─────────────────────────────────────────┐
│              Next.js API Route (/api/push/send)                  │
│  • Validates notification payload                                │
│  • Sends to all subscribed users                                 │
│  • Logs delivery status                                          │
└──────────────────────────────────────────────────────────────────┘
                        ▲
                        │ Triggered by
┌───────────────────────┴─────────────────────────────────────────┐
│              Background Jobs / Event Handlers                    │
│  • Geofence triggers                                             │
│  • Event reminders                                               │
│  • Traffic alerts                                                │
│  • Transit delays                                                │
└──────────────────────────────────────────────────────────────────┘
```

**Implementation Details:**

```typescript
// src/lib/push.ts
import webpush from 'web-push';

// Configure VAPID keys
webpush.setVAPIDDetails(
  'mailto:admin@atlantafifa.com',
  process.env.VAPID_PUBLIC_KEY!,
  process.env.VAPID_PRIVATE_KEY!
);

export async function sendPushNotification(
  subscription: PushSubscription,
  payload: {
    title: string;
    body: string;
    icon?: string;
    data?: any;
  }
) {
  try {
    await webpush.sendNotification(
      subscription as any,
      JSON.stringify(payload)
    );
    return { success: true };
  } catch (error) {
    console.error('Push notification failed:', error);
    return { success: false, error };
  }
}
```

### 7.2 Geofencing System

**Real-time Geofence Monitoring:**
```typescript
// src/lib/geofence.ts

interface GeofenceZone {
  id: string;
  centerLat: number;
  centerLng: number;
  radiusMeters: number;
  alertMessage: string;
  priority: string;
}

/**
 * Haversine formula to calculate distance between two points
 */
function calculateDistance(
  lat1: number,
  lng1: number,
  lat2: number,
  lng2: number
): number {
  const R = 6371e3; // Earth radius in meters
  const φ1 = (lat1 * Math.PI) / 180;
  const φ2 = (lat2 * Math.PI) / 180;
  const Δφ = ((lat2 - lat1) * Math.PI) / 180;
  const Δλ = ((lng2 - lng1) * Math.PI) / 180;

  const a =
    Math.sin(Δφ / 2) * Math.sin(Δφ / 2) +
    Math.cos(φ1) * Math.cos(φ2) * Math.sin(Δλ / 2) * Math.sin(Δλ / 2);
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));

  return R * c; // Distance in meters
}

export async function checkGeofences(
  userLat: number,
  userLng: number
): Promise<GeofenceZone[]> {
  // Fetch active geofence zones from database
  const zones = await prisma.geofenceZone.findMany({
    where: { isActive: true }
  });

  // Filter zones user is currently within
  const triggeredZones = zones.filter(zone => {
    const distance = calculateDistance(
      userLat,
      userLng,
      zone.centerLat,
      zone.centerLng
    );
    return distance <= zone.radiusMeters;
  });

  return triggeredZones.sort((a, b) => {
    const priorityOrder = { urgent: 0, high: 1, normal: 2, low: 3 };
    return priorityOrder[a.priority] - priorityOrder[b.priority];
  });
}

/**
 * Client-side geofence monitoring hook
 */
export function useGeofenceMonitoring() {
  const [location, setLocation] = useState<{lat: number; lng: number} | null>(null);
  const [alerts, setAlerts] = useState<GeofenceZone[]>([]);

  useEffect(() => {
    // Watch user location
    const watchId = navigator.geolocation.watchPosition(
      async (position) => {
        const { latitude, longitude } = position.coords;
        setLocation({ lat: latitude, lng: longitude });

        // Check geofences
        const response = await fetch(
          `/api/geofence/check?lat=${latitude}&lng=${longitude}`
        );
        const data = await response.json();

        if (data.alerts.length > 0) {
          setAlerts(data.alerts);
          // Trigger notification for high-priority alerts
          data.alerts
            .filter(alert => alert.priority === 'high' || alert.priority === 'urgent')
            .forEach(alert => {
              new Notification(alert.message);
            });
        }
      },
      (error) => console.error('Location error:', error),
      { enableHighAccuracy: true, maximumAge: 30000 }
    );

    return () => navigator.geolocation.clearWatch(watchId);
  }, []);

  return { location, alerts };
}
```

### 7.3 Notification Triggers

**Event-Based Notifications:**

1. **Event Reminders**
   - 24 hours before match
   - 3 hours before match
   - 1 hour before match

2. **Traffic Alerts**
   - Heavy traffic detected on route
   - Suggested alternate route available

3. **Transit Delays**
   - MARTA line delays affecting route
   - Service disruptions

4. **Parking Availability**
   - Nearby lot filling up
   - Cheaper parking available

5. **Geofence Triggers**
   - Entering event area
   - Entering fan zone
   - Near partner restaurant/bar

**Implementation Example:**
```typescript
// src/jobs/notificationTriggers.ts

export async function checkEventReminders() {
  const upcoming = await prisma.event.findMany({
    where: {
      matchDate: {
        gte: new Date(),
        lte: addHours(new Date(), 24)
      }
    },
    include: { venue: true }
  });

  for (const event of upcoming) {
    // Find users who favorited this event's teams
    const users = await prisma.user.findMany({
      where: {
        favoritedTeams: {
          contains: event.homeTeam // Simplified; use JSON query in real impl
        },
        notificationsEnabled: true,
        pushSubscription: { not: null }
      }
    });

    for (const user of users) {
      await sendPushNotification(
        JSON.parse(user.pushSubscription!),
        {
          title: `${event.homeTeam} vs ${event.awayTeam} Tomorrow!`,
          body: `Match at ${event.venue.name} - ${event.matchTime}`,
          data: { eventId: event.id }
        }
      );

      // Also create in-app notification
      await prisma.notification.create({
        data: {
          userId: user.id,
          type: 'event',
          title: `${event.homeTeam} vs ${event.awayTeam} Tomorrow!`,
          message: `Match at ${event.venue.name} - ${event.matchTime}`,
          priority: 'normal',
          actionUrl: `/en/events/${event.id}`
        }
      });
    }
  }
}
```

---

## 8. Environment Variables

### 8.1 Required Variables

Create a `.env` file in the project root:

```bash
# Google Maps API Key (client-side)
# Get from: https://console.cloud.google.com/apis/credentials
# Enable: Maps JavaScript API, Places API, Directions API
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=AIza...

# MARTA Transit API Keys (server-side)
# Get from: https://www.itsmarta.com/developer-resources.aspx
MARTA_API_KEY=your_marta_bus_api_key
MARTA_TRAIN_API_KEY=your_marta_train_api_key

# Database Connection
# Development (SQLite)
DATABASE_URL="file:./dev.db"

# Production (PostgreSQL) - Uncomment for production
# DATABASE_URL="postgresql://user:pass@host:5432/dbname?schema=public"

# Stadium/Venue Configuration (client-side)
NEXT_PUBLIC_STADIUM_NAME="Mercedes-Benz Stadium"
NEXT_PUBLIC_STADIUM_LAT="33.754542"
NEXT_PUBLIC_STADIUM_LNG="-84.402492"

# Optional Features
USE_MOCK_MARTA_DATA=false  # Set to true for development without API keys
NODE_ENV=development        # development | production

# Push Notifications (server-side)
# Generate VAPID keys using: npx web-push generate-vapid-keys
VAPID_PUBLIC_KEY=your_vapid_public_key
VAPID_PRIVATE_KEY=your_vapid_private_key

# Google Places API (server-side, for attractions/POI)
GOOGLE_PLACES_API_KEY=your_google_places_api_key

# Feature Flags
ENABLE_PUSH_NOTIFICATIONS=true
ENABLE_GEOFENCING=true
ENABLE_PARKING_TRACKING=true
ENABLE_ITINERARIES=true

# Auto-set by GitHub Codespaces (DO NOT MANUALLY SET)
# CODESPACES=true
```

### 8.2 Environment Detection

```typescript
// src/lib/codespaces.ts

/**
 * Detects if the application is running in GitHub Codespaces
 * @returns {boolean} True if in Codespaces environment
 */
export function isRunningInCodespaces(): boolean {
  return process.env.CODESPACES === 'true';
}

/**
 * Returns the appropriate MARTA Train API URL based on environment
 * Port 18096 is blocked in Codespaces, requiring a proxy
 */
export function getMartaTrainApiUrl(): string {
  const directUrl = 'http://developer.itsmarta.com:18096/...';

  if (isRunningInCodespaces()) {
    // Use allOrigins proxy in Codespaces
    const encodedUrl = encodeURIComponent(directUrl);
    return `https://api.allorigins.win/raw?url=${encodedUrl}`;
  }

  // Direct connection in production
  return directUrl;
}
```

---

## 8. Special Implementation Considerations

### 8.1 GitHub Codespaces Proxy Solution

**Problem:** GitHub Codespaces blocks non-standard ports (specifically port 18096 used by MARTA Train API).

**Solution:** Environment-aware proxy pattern

**Implementation:**

```typescript
// src/app/api/transit/route.ts

import { getMartaTrainApiUrl, isRunningInCodespaces } from '@/lib/codespaces';
import GtfsRealtimeBindings from 'gtfs-realtime-bindings';

export async function GET(request: Request) {
  const useMockData = process.env.USE_MOCK_MARTA_DATA === 'true';

  if (useMockData) {
    return Response.json(getMockTransitData());
  }

  try {
    // Fetch bus data (standard port, no proxy needed)
    const busData = await fetchBusData();

    // Fetch train data with environment-aware URL
    const trainUrl = getMartaTrainApiUrl();
    const trainResponse = await fetch(trainUrl, {
      headers: {
        'Authorization': `Bearer ${process.env.MARTA_TRAIN_API_KEY}`
      },
      next: { revalidate: 30 } // Cache for 30 seconds
    });

    if (!trainResponse.ok) {
      console.error(`Train API error: ${trainResponse.status}`);
      return Response.json({
        buses: busData,
        trains: [],
        timestamp: new Date().toISOString(),
        source: 'partial'
      });
    }

    const trainBuffer = await trainResponse.arrayBuffer();
    const trainFeed = GtfsRealtimeBindings.transit_realtime.FeedMessage.decode(
      new Uint8Array(trainBuffer)
    );

    const trains = parseTrainPositions(trainFeed);

    return Response.json({
      buses: busData,
      trains,
      timestamp: new Date().toISOString(),
      source: 'live'
    });

  } catch (error) {
    console.error('Transit API error:', error);

    // Graceful degradation - return empty arrays
    return Response.json({
      buses: [],
      trains: [],
      timestamp: new Date().toISOString(),
      source: 'error'
    });
  }
}
```

**Environment Detection Logging:**
```typescript
// Log on server startup
console.log('🚀 Environment:', {
  isCodespaces: isRunningInCodespaces(),
  nodeEnv: process.env.NODE_ENV,
  trainApiUrl: getMartaTrainApiUrl()
});
```

### 8.2 Mock Data Structure

```typescript
// src/lib/data.ts

export const mockFifaMatches: Event[] = [
  {
    id: '1',
    homeTeam: 'Argentina',
    awayTeam: 'Brazil',
    venue: {
      id: 'mbs-1',
      name: 'Mercedes-Benz Stadium',
      address: '1 AMB Drive NW',
      city: 'Atlanta',
      state: 'GA',
      latitude: 33.754542,
      longitude: -84.402492,
      capacity: 71000
    },
    matchDate: new Date('2026-06-15'),
    matchTime: '19:00',
    category: 'FIFA_2026'
  },
  {
    id: '2',
    homeTeam: 'USA',
    awayTeam: 'Mexico',
    venue: {
      id: 'mbs-1',
      name: 'Mercedes-Benz Stadium',
      address: '1 AMB Drive NW',
      city: 'Atlanta',
      state: 'GA',
      latitude: 33.754542,
      longitude: -84.402492,
      capacity: 71000
    },
    matchDate: new Date('2026-06-18'),
    matchTime: '20:00',
    category: 'FIFA_2026'
  },
  {
    id: '3',
    homeTeam: 'Germany',
    awayTeam: 'Spain',
    venue: {
      id: 'mbs-1',
      name: 'Mercedes-Benz Stadium',
      address: '1 AMB Drive NW',
      city: 'Atlanta',
      state: 'GA',
      latitude: 33.754542,
      longitude: -84.402492,
      capacity: 71000
    },
    matchDate: new Date('2026-06-22'),
    matchTime: '16:00',
    category: 'FIFA_2026'
  }
];

export const mockTransitData: TransitApiResponse = {
  buses: [
    {
      id: 'bus-001',
      type: 'bus',
      routeId: '12',
      routeName: 'Howell Mill',
      latitude: 33.755,
      longitude: -84.403,
      heading: 45,
      speed: 25,
      lastUpdate: new Date()
    }
    // ... more mock buses
  ],
  trains: [
    {
      id: 'train-001',
      type: 'train',
      routeId: 'RED',
      routeName: 'Red Line',
      latitude: 33.748,
      longitude: -84.391,
      heading: 90,
      speed: 35,
      lastUpdate: new Date()
    }
    // ... more mock trains
  ],
  timestamp: new Date()
};
```

### 8.3 Animation Implementation

**Smooth marker transitions using requestAnimationFrame:**

```typescript
// src/components/AnimatedTransitMarker.tsx

interface AnimationState {
  startPosition: { lat: number; lng: number };
  endPosition: { lat: number; lng: number };
  startTime: number;
  duration: number;
}

export function AnimatedTransitMarker({ vehicle, type }: Props) {
  const [position, setPosition] = useState({
    lat: vehicle.latitude,
    lng: vehicle.longitude
  });

  const animationRef = useRef<AnimationState | null>(null);
  const rafRef = useRef<number>();

  useEffect(() => {
    // Start new animation when vehicle position updates
    const newEndPosition = {
      lat: vehicle.latitude,
      lng: vehicle.longitude
    };

    // Don't animate if position hasn't changed
    if (
      newEndPosition.lat === position.lat &&
      newEndPosition.lng === position.lng
    ) {
      return;
    }

    animationRef.current = {
      startPosition: position,
      endPosition: newEndPosition,
      startTime: Date.now(),
      duration: 2000 // 2 seconds
    };

    const animate = () => {
      if (!animationRef.current) return;

      const { startPosition, endPosition, startTime, duration } = animationRef.current;
      const elapsed = Date.now() - startTime;
      const progress = Math.min(elapsed / duration, 1);

      // Cubic ease-out for smooth deceleration
      const eased = 1 - Math.pow(1 - progress, 3);

      const currentPosition = {
        lat: startPosition.lat + (endPosition.lat - startPosition.lat) * eased,
        lng: startPosition.lng + (endPosition.lng - startPosition.lng) * eased
      };

      setPosition(currentPosition);

      if (progress < 1) {
        rafRef.current = requestAnimationFrame(animate);
      } else {
        animationRef.current = null;
      }
    };

    rafRef.current = requestAnimationFrame(animate);

    return () => {
      if (rafRef.current) {
        cancelAnimationFrame(rafRef.current);
      }
    };
  }, [vehicle.latitude, vehicle.longitude]);

  return (
    <AdvancedMarker position={position}>
      <div
        className={`transit-icon ${type}`}
        style={{
          transform: vehicle.heading ? `rotate(${vehicle.heading}deg)` : undefined,
          transition: 'transform 0.3s ease-out'
        }}
      >
        {type === 'bus' ? '🚌' : '🚇'}
      </div>
    </AdvancedMarker>
  );
}
```

---

## 9. Internationalization (i18n) Implementation

### 9.1 Routing Structure

```
/en          → English homepage
/es          → Spanish homepage
/de          → German homepage
/ko          → Korean homepage
/en/events   → English events page (future)
/es/events   → Spanish events page (future)
```

### 9.2 Translation Files

```
public/locales/
├── en/
│   └── common.json
├── es/
│   └── common.json
├── de/
│   └── common.json
└── ko/
    └── common.json
```

**Example translation file (en/common.json):**
```json
{
  "navigation": {
    "events": "Events",
    "map": "Map",
    "venues": "Venues"
  },
  "events": {
    "title": "FIFA Matches",
    "vs": "vs",
    "showOnMap": "Show on map",
    "noEvents": "No events scheduled"
  },
  "map": {
    "loading": "Loading map...",
    "error": "Unable to load map"
  },
  "languages": {
    "en": "English",
    "es": "Spanish",
    "de": "German",
    "ko": "Korean"
  }
}
```

### 9.3 i18n Configuration

```typescript
// src/lib/i18n.ts

export const locales = ['en', 'es', 'de', 'ko'] as const;
export type Locale = typeof locales[number];

export const defaultLocale: Locale = 'en';

export const localeNames: Record<Locale, string> = {
  en: 'English',
  es: 'Español',
  de: 'Deutsch',
  ko: '한국어'
};

export const localeFlags: Record<Locale, string> = {
  en: '🇺🇸',
  es: '🇪🇸',
  de: '🇩🇪',
  ko: '🇰🇷'
};

export function isValidLocale(locale: string): locale is Locale {
  return locales.includes(locale as Locale);
}
```

### 9.4 Date/Time Formatting

```typescript
// src/lib/dateFormat.ts

export function formatMatchDate(date: Date, locale: string): string {
  return new Intl.DateTimeFormat(locale, {
    weekday: 'long',
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  }).format(date);
}

export function formatMatchTime(time: string, locale: string): string {
  // time is in HH:MM format
  const [hours, minutes] = time.split(':').map(Number);
  const date = new Date();
  date.setHours(hours, minutes);

  return new Intl.DateTimeFormat(locale, {
    hour: 'numeric',
    minute: '2-digit',
    hour12: locale === 'en' // 12-hour for English, 24-hour for others
  }).format(date);
}
```

---

## 10. Performance Specifications

### 10.1 Performance Targets

| Metric | Target | Critical |
|--------|--------|----------|
| Time to First Byte (TTFB) | < 500ms | < 1000ms |
| First Contentful Paint (FCP) | < 2s | < 3s |
| Largest Contentful Paint (LCP) | < 2.5s | < 4s |
| Cumulative Layout Shift (CLS) | < 0.1 | < 0.25 |
| First Input Delay (FID) | < 100ms | < 300ms |
| API Response Time | < 2s | < 5s |

### 10.2 Caching Strategy

**API Routes:**
```typescript
// 30-second revalidation for transit data
export const revalidate = 30;

// 5-minute cache for events
export const revalidate = 300;
```

**Client-Side Caching:**
```typescript
// Using SWR for client-side data fetching
import useSWR from 'swr';

export function useTransitData() {
  const { data, error } = useSWR('/api/transit', fetcher, {
    refreshInterval: 30000, // Refresh every 30 seconds
    revalidateOnFocus: false,
    dedupingInterval: 5000
  });

  return {
    buses: data?.buses ?? [],
    trains: data?.trains ?? [],
    isLoading: !error && !data,
    isError: error
  };
}
```

### 10.3 Bundle Optimization

**Next.js Configuration:**
```javascript
// next.config.js

/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,

  // Image optimization
  images: {
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
  },

  // Experimental features
  experimental: {
    optimizePackageImports: ['@vis.gl/react-google-maps'],
  },

  // Environment variables
  env: {
    NEXT_PUBLIC_GOOGLE_MAPS_API_KEY: process.env.NEXT_PUBLIC_GOOGLE_MAPS_API_KEY,
    NEXT_PUBLIC_STADIUM_LAT: process.env.NEXT_PUBLIC_STADIUM_LAT,
    NEXT_PUBLIC_STADIUM_LNG: process.env.NEXT_PUBLIC_STADIUM_LNG,
  }
};

module.exports = nextConfig;
```

---

## 11. Security Specifications

### 11.1 API Key Management

**Rules:**
1. **NEVER** commit API keys to git
2. **ALWAYS** use environment variables
3. **Client-side keys** MUST use `NEXT_PUBLIC_` prefix
4. **Server-side keys** MUST NEVER be exposed to client

**Validation:**
```typescript
// src/lib/env.ts

import { z } from 'zod';

const envSchema = z.object({
  // Client-side (public)
  NEXT_PUBLIC_GOOGLE_MAPS_API_KEY: z.string().min(1),
  NEXT_PUBLIC_STADIUM_LAT: z.string().regex(/^-?\d+\.\d+$/),
  NEXT_PUBLIC_STADIUM_LNG: z.string().regex(/^-?\d+\.\d+$/),

  // Server-side (private)
  MARTA_API_KEY: z.string().min(1).optional(),
  MARTA_TRAIN_API_KEY: z.string().min(1).optional(),
  DATABASE_URL: z.string().min(1),
});

export const env = envSchema.parse(process.env);
```

### 11.2 Input Sanitization

```typescript
// API route example with validation
import { z } from 'zod';

const querySchema = z.object({
  type: z.enum(['bus', 'train']).optional(),
  route: z.string().max(10).optional()
});

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);

  // Validate and sanitize input
  const result = querySchema.safeParse({
    type: searchParams.get('type'),
    route: searchParams.get('route')
  });

  if (!result.success) {
    return Response.json(
      { error: 'Invalid parameters' },
      { status: 400 }
    );
  }

  // Use validated data
  const { type, route } = result.data;
  // ...
}
```

### 11.3 CORS and Headers

```typescript
// src/middleware.ts

import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const response = NextResponse.next();

  // Security headers
  response.headers.set('X-Frame-Options', 'DENY');
  response.headers.set('X-Content-Type-Options', 'nosniff');
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
  response.headers.set(
    'Content-Security-Policy',
    "default-src 'self'; script-src 'self' 'unsafe-eval' 'unsafe-inline' https://maps.googleapis.com; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; connect-src 'self' https://maps.googleapis.com https://developer.itsmarta.com https://api.allorigins.win;"
  );

  return response;
}

export const config = {
  matcher: [
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
  ],
};
```

---

## 12. Testing Strategy

### 12.1 Testing Levels

**Unit Tests:**
- Utility functions (dateFormat, codespaces detection)
- Data transformations (GTFS-RT parsing)
- Component logic (animation calculations)

**Integration Tests:**
- API routes with mock data
- Database operations
- Environment detection

**E2E Tests:**
- Full user flows (language switching, map interaction)
- API endpoint responses
- Error handling

### 12.2 Test Environment Setup

```bash
# Install testing dependencies
pnpm add -D vitest @testing-library/react @testing-library/jest-dom
```

**Example Unit Test:**
```typescript
// src/lib/__tests__/codespaces.test.ts

import { describe, it, expect, beforeEach, afterEach } from 'vitest';
import { isRunningInCodespaces, getMartaTrainApiUrl } from '../codespaces';

describe('Codespaces Detection', () => {
  const originalEnv = process.env;

  beforeEach(() => {
    process.env = { ...originalEnv };
  });

  afterEach(() => {
    process.env = originalEnv;
  });

  it('detects Codespaces environment', () => {
    process.env.CODESPACES = 'true';
    expect(isRunningInCodespaces()).toBe(true);
  });

  it('detects non-Codespaces environment', () => {
    delete process.env.CODESPACES;
    expect(isRunningInCodespaces()).toBe(false);
  });

  it('returns proxy URL in Codespaces', () => {
    process.env.CODESPACES = 'true';
    const url = getMartaTrainApiUrl();
    expect(url).toContain('allorigins.win');
  });

  it('returns direct URL outside Codespaces', () => {
    delete process.env.CODESPACES;
    const url = getMartaTrainApiUrl();
    expect(url).toContain('developer.itsmarta.com');
  });
});
```

---

## 13. Deployment Specification

### 13.1 Vercel Configuration

```json
{
  "buildCommand": "pnpm run build",
  "devCommand": "pnpm run dev",
  "installCommand": "pnpm install",
  "framework": "nextjs",
  "outputDirectory": ".next",
  "regions": ["iad1"],
  "env": {
    "NEXT_PUBLIC_GOOGLE_MAPS_API_KEY": "@google-maps-api-key",
    "MARTA_API_KEY": "@marta-api-key",
    "MARTA_TRAIN_API_KEY": "@marta-train-api-key",
    "DATABASE_URL": "@database-url"
  }
}
```

### 13.2 Database Migration Strategy

**Development:**
```bash
pnpm prisma migrate dev --name init
pnpm prisma db seed
```

**Production:**
```bash
pnpm prisma migrate deploy
```

### 13.3 CI/CD Pipeline (Optional)

```yaml
# .github/workflows/deploy.yml

name: Deploy to Vercel

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - uses: pnpm/action-setup@v2
        with:
          version: 8

      - uses: actions/setup-node@v3
        with:
          node-version: 20
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install

      - name: Run type check
        run: pnpm run type-check

      - name: Run linter
        run: pnpm run lint

      - name: Run tests
        run: pnpm run test

      - name: Build
        run: pnpm run build

      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
```

---

## 14. File Structure Summary

```
fifa-traffic-demo/
├── src/
│   ├── app/
│   │   ├── [lang]/
│   │   │   ├── page.tsx
│   │   │   ├── layout.tsx
│   │   │   ├── loading.tsx
│   │   │   ├── error.tsx
│   │   │   └── globals.css
│   │   ├── api/
│   │   │   ├── events/
│   │   │   │   └── route.ts
│   │   │   ├── transit/
│   │   │   │   └── route.ts
│   │   │   ├── venues/
│   │   │   │   └── route.ts
│   │   │   ├── user/
│   │   │   │   └── route.ts
│   │   │   ├── parking/
│   │   │   │   └── route.ts
│   │   │   ├── directions/
│   │   │   │   └── route.ts
│   │   │   ├── attractions/
│   │   │   │   └── route.ts
│   │   │   ├── notifications/
│   │   │   │   ├── route.ts
│   │   │   │   └── [id]/
│   │   │   │       └── read/
│   │   │   │           └── route.ts
│   │   │   ├── push/
│   │   │   │   ├── subscribe/
│   │   │   │   │   └── route.ts
│   │   │   │   └── send/
│   │   │   │       └── route.ts
│   │   │   ├── itineraries/
│   │   │   │   ├── route.ts
│   │   │   │   └── [id]/
│   │   │   │       └── route.ts
│   │   │   ├── geofence/
│   │   │   │   └── check/
│   │   │   │       └── route.ts
│   │   │   └── rideshare/
│   │   │       └── route.ts
│   │   ├── favicon.ico
│   │   ├── robots.txt
│   │   └── manifest.json          # PWA manifest for push notifications
│   ├── components/
│   │   ├── MapView.tsx
│   │   ├── AnimatedTransitMarker.tsx
│   │   ├── VenueMarker.tsx
│   │   ├── ParkingMarker.tsx
│   │   ├── EventList.tsx
│   │   ├── EventCard.tsx
│   │   ├── LanguageSwitcher.tsx
│   │   ├── NotificationBell.tsx
│   │   ├── NotificationPanel.tsx
│   │   ├── DirectionsPanel.tsx
│   │   ├── RideshareOptions.tsx
│   │   ├── ItineraryBuilder.tsx
│   │   ├── ItineraryCard.tsx
│   │   ├── AttractionCard.tsx
│   │   └── GeofenceAlert.tsx
│   ├── hooks/
│   │   ├── useTransitData.ts
│   │   ├── useEvents.ts
│   │   ├── useNotifications.ts
│   │   ├── useDirections.ts
│   │   ├── useItineraries.ts
│   │   ├── useGeofenceMonitoring.ts
│   │   └── useParkingData.ts
│   ├── lib/
│   │   ├── codespaces.ts
│   │   ├── data.ts
│   │   ├── dateFormat.ts
│   │   ├── i18n.ts
│   │   ├── storage.ts
│   │   ├── env.ts
│   │   ├── prisma.ts
│   │   ├── push.ts                 # Push notification utilities
│   │   ├── geofence.ts             # Geofencing utilities
│   │   └── rideshare.ts            # Rideshare deep link generation
│   ├── jobs/
│   │   └── notificationTriggers.ts # Background job handlers
│   └── types/
│       └── index.ts
├── prisma/
│   ├── schema.prisma
│   ├── seed.ts
│   └── migrations/
├── public/
│   ├── locales/
│   │   ├── en/
│   │   │   └── common.json
│   │   ├── es/
│   │   │   └── common.json
│   │   ├── de/
│   │   │   └── common.json
│   │   └── ko/
│   │       └── common.json
│   └── sw.js                      # Service worker for push notifications
├── .env
├── .env.example
├── .gitignore
├── next.config.js
├── package.json
├── pnpm-lock.yaml
├── tsconfig.json
├── tailwind.config.ts
├── postcss.config.js
├── vercel.json
└── README.md
```

---

## 15. Next Steps for Implementation

### Phase 1: Project Setup (Week 1)
1. Initialize Next.js project with TypeScript
2. Install all dependencies (see package.json spec)
3. Configure Prisma with SQLite
4. Set up environment variables (including VAPID keys)
5. Create base file structure
6. Configure service worker for push notifications

### Phase 2: Core Features (Weeks 2-4)
1. Implement Google Maps integration
2. Create MapView component with traffic layer
3. Build API routes for transit data
4. Implement Codespaces proxy logic
5. Add mock FIFA event data
6. Implement parking data integration
7. Create parking marker overlays
8. Build multi-modal directions API
9. Integrate rideshare deep links

### Phase 3: Internationalization (Week 5)
1. Configure next-intl
2. Create translation files (EN, ES, DE, KO)
3. Implement language switcher
4. Add locale-aware date formatting
5. Translate all UI components

### Phase 4: Personalization Features (Weeks 6-7)
1. Implement user preference system
2. Build favorites functionality (teams, venues)
3. Create itinerary builder component
4. Implement itinerary management API
5. Add attractions/POI discovery
6. Integrate Google Places API
7. Build attraction card components

### Phase 5: Notifications & Geofencing (Weeks 8-9)
1. Implement push notification system
2. Configure VAPID keys and web-push
3. Build notification bell component
4. Create geofencing engine
5. Implement location tracking
6. Build geofence alert system
7. Create background job triggers
8. Set up event reminders
9. Implement traffic and transit alerts

### Phase 6: Animation & Polish (Week 10)
1. Implement AnimatedTransitMarker
2. Add smooth map transitions
3. Optimize component rendering
4. Add loading states
5. Implement error boundaries
6. Test across environments (Codespaces, production)

### Phase 7: Testing & Deployment (Weeks 11-12)
1. Write unit tests (utilities, hooks)
2. Write integration tests (API routes)
3. Write E2E tests (user flows)
4. Load testing for high-traffic scenarios
5. Security audit (OWASP Top 10)
6. Deploy to Vercel
7. Configure production database (PostgreSQL)
8. Performance testing and optimization
9. Final QA and bug fixes

---

## 16. Acceptance Criteria Reference

See `acceptance_criteria.md` for detailed verification checklists for each feature.

---

## Appendix A: Dependencies

```json
{
  "name": "atlanta-fifa-navigator",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "type-check": "tsc --noEmit",
    "db:push": "prisma db push",
    "db:migrate": "prisma migrate dev",
    "db:seed": "tsx prisma/seed.ts"
  },
  "dependencies": {
    "@googlemaps/google-maps-services-js": "^3.3.0",
    "@headlessui/react": "^2.0.0",
    "@prisma/client": "^5.9.0",
    "@vis.gl/react-google-maps": "^1.0.0",
    "date-fns": "^3.0.0",
    "gtfs-realtime-bindings": "^1.0.0",
    "next": "^15.0.0",
    "next-intl": "^3.0.0",
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "swr": "^2.2.0",
    "web-push": "^3.6.0",
    "zod": "^3.22.0"
  },
  "devDependencies": {
    "@testing-library/jest-dom": "^6.0.0",
    "@testing-library/react": "^15.0.0",
    "@types/node": "^20.0.0",
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0",
    "@types/web-push": "^3.6.0",
    "autoprefixer": "^10.4.0",
    "eslint": "^8.0.0",
    "eslint-config-next": "^15.0.0",
    "postcss": "^8.4.0",
    "prettier": "^3.0.0",
    "prisma": "^5.9.0",
    "tailwindcss": "^3.4.0",
    "tsx": "^4.7.0",
    "typescript": "^5.3.0",
    "vitest": "^1.0.0"
  },
  "engines": {
    "node": ">=20.0.0",
    "pnpm": ">=8.0.0"
  }
}
```

---

## Appendix B: Enhancement Summary (C.R.A.F.T Framework Application)

This specification was comprehensively updated using the **C.R.A.F.T** (Context, Role, Action, Format, Tone) framework to ensure all PRD requirements are fully addressed.

### Enhancements Added

#### 1. Multi-Modal Transportation
**From PRD Section 3.2:**
- ✅ **Rideshare Integration**: Deep links for Uber and Lyft
- ✅ **Walking Directions**: Google Maps Directions API integration
- ✅ **Parking Availability**: Real-time parking lot tracking with availability indicators
- ✅ **Detour Information**: Traffic-aware routing with alternative suggestions

#### 2. Personalization Features
**From PRD Section 3.3:**
- ✅ **User Preferences**: Expanded to include language, favorites, and notifications
- ✅ **Itinerary Builder**: Complete itinerary creation and management system
- ✅ **Suggested Itineraries**: Algorithm-generated personalized trip planning
- ✅ **Favorites System**: Team and venue favoriting with preference persistence

#### 3. Push Notifications System
**From PRD Sections 3.3 & 4:**
- ✅ **Web Push Notifications**: VAPID-based push notification infrastructure
- ✅ **Service Worker**: Background notification handling
- ✅ **Event Reminders**: Automated notifications 24h, 3h, and 1h before matches
- ✅ **Traffic Alerts**: Real-time traffic condition notifications
- ✅ **Transit Delays**: MARTA service disruption alerts
- ✅ **In-App Notifications**: Persistent notification center

#### 4. Geofencing & Location-Based Alerts
**From PRD Section 3.2:**
- ✅ **Geofence Engine**: Haversine-based distance calculation
- ✅ **Location Monitoring**: Real-time user location tracking
- ✅ **Contextual Alerts**: Zone-based notifications (venue, fan zone, partner locations)
- ✅ **Priority System**: Urgent, high, normal, and low priority alerts

#### 5. Attractions & Points of Interest
**From PRD Section 3.1:**
- ✅ **Nearby Attractions**: Google Places API integration
- ✅ **Dining Options**: Restaurant discovery with ratings and prices
- ✅ **Fan Zones**: Dedicated fan experience locations
- ✅ **Partner Promotions**: Featured partner locations and offers
- ✅ **Distance Calculation**: Proximity-based sorting

#### 6. Enhanced Venue Information
**From PRD Section 3.1:**
- ✅ **Seating Charts**: Venue seating layout visualization
- ✅ **Detailed Amenities**: WiFi, food, restrooms, accessibility info
- ✅ **Accessibility Information**: Wheelchair access, elevator locations
- ✅ **Parking Integration**: Associated parking facilities

### New Data Models Added

1. **ParkingLot** - Real-time parking availability tracking
2. **Notification** - In-app and push notification management
3. **Itinerary** - User-created trip planning
4. **GeofenceZone** - Location-based alert boundaries
5. **AttractionPOI** - Points of interest discovery
6. **Enhanced User** - Push subscriptions, location permissions
7. **Enhanced Venue** - Seating charts, amenities, nearby attractions

### New API Endpoints Added

1. `GET /api/parking` - Parking lot availability
2. `POST /api/directions` - Multi-modal directions
3. `GET /api/attractions` - Nearby POI discovery
4. `GET /api/notifications` - User notifications
5. `POST /api/notifications` - Create notifications
6. `POST /api/push/subscribe` - Push notification subscription
7. `POST /api/push/send` - Send push notifications
8. `GET /api/itineraries` - User itineraries
9. `POST /api/itineraries` - Create itinerary
10. `GET /api/geofence/check` - Geofence zone checking
11. `GET /api/rideshare` - Rideshare deep link generation

### New Components Added

1. **NotificationBell** - Notification center UI
2. **NotificationPanel** - Notification display
3. **ParkingMarker** - Map parking indicators
4. **DirectionsPanel** - Multi-modal routing UI
5. **RideshareOptions** - Uber/Lyft integration
6. **ItineraryBuilder** - Trip planning interface
7. **ItineraryCard** - Itinerary display
8. **AttractionCard** - POI discovery cards
9. **GeofenceAlert** - Location-based alerts

### New Libraries & Dependencies

- `web-push` - Push notification service
- `@googlemaps/google-maps-services-js` - Directions API
- `@headlessui/react` - Accessible UI components
- `date-fns` - Date manipulation
- `vitest` - Unit testing framework
- `@testing-library/react` - Component testing

### Technical Architecture Enhancements

1. **Push Notification Infrastructure**
   - Service worker implementation
   - VAPID key configuration
   - Background job system for triggers

2. **Geofencing System**
   - Real-time location monitoring
   - Distance calculation algorithms
   - Zone-based alert triggering

3. **Personalization Engine**
   - User preference management
   - Itinerary recommendation system
   - Favorites tracking

4. **Multi-Modal Routing**
   - Transit + Walking + Driving + Rideshare
   - Traffic-aware route optimization
   - Alternative route suggestions

### Environment Variables Added

- `VAPID_PUBLIC_KEY` - Push notification public key
- `VAPID_PRIVATE_KEY` - Push notification private key
- `GOOGLE_PLACES_API_KEY` - Places API access
- `ENABLE_PUSH_NOTIFICATIONS` - Feature flag
- `ENABLE_GEOFENCING` - Feature flag
- `ENABLE_PARKING_TRACKING` - Feature flag
- `ENABLE_ITINERARIES` - Feature flag

### Security Enhancements

- Input validation with Zod schemas
- Environment variable validation
- Push notification authentication
- Geofence permission management
- API rate limiting considerations

### Performance Optimizations

- SWR caching for real-time data
- Geofence calculation caching (1 min)
- Service worker background processing
- Optimized component rendering
- Progressive Web App capabilities

---

## Document Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-05 | Technical Architect | Initial specification |
| 2.0 | 2025-11-05 | Technical Architect | Comprehensive update using C.R.A.F.T framework:<br>• Added push notification system with VAPID<br>• Added geofencing and location-based alerts<br>• Added parking availability tracking<br>• Added multi-modal directions (walking, driving, rideshare)<br>• Added itinerary builder and personalization<br>• Added attractions/POI discovery<br>• Enhanced data models (7 new models)<br>• Added 11 new API endpoints<br>• Added 9 new UI components<br>• Updated dependencies and environment variables<br>• Expanded implementation phases from 6 to 12 weeks |

---

**End of Technical Specification**
