# Shuttly - Smart Shuttle Tracking and Fare Enforcement System

## Project Overview

A digital campus shuttle system with QR-based boarding, GPS tracking, automated fare calculation, and penalty enforcement for incomplete rides.

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Framework | Next.js 16 (App Router, RSC) |
| UI Components | shadcn/ui (new-york style, neutral theme) |
| Styling | Tailwind CSS v4 + CSS variables |
| Database | Neon (Serverless PostgreSQL) |
| ORM | Drizzle ORM with @neondatabase/serverless |
| Authentication | Better Auth |
| Real-time | Server-Sent Events (SSE) |
| Maps | Google Maps API |
| QR Scanning | html5-qrcode |
| Forms | react-hook-form + zod |

---

## Development Rules

1. **shadcn Components**: Always use CLI to add (`npx shadcn@latest add <component>`)
2. **Never edit**: `components/ui/*` files directly
3. **Never edit**: Generated CSS files from shadcn
4. **Styling**: Use CSS variables (`bg-primary`, `text-muted-foreground`, etc.)
5. **Dependencies**: Use `npm add <package>` (not manual package.json edits)
6. **Config changes**: Use CLI commands where possible
7. **Documentation**: Always read relevant docs before implementing features

---

## Project Structure

```
Documents/shuttly/
├── app/
│   ├── (auth)/                    # Auth route group (public)
│   │   ├── login/page.tsx
│   │   └── register/page.tsx
│   ├── (dashboard)/               # Protected routes
│   │   ├── layout.tsx             # Dashboard layout with sidebar
│   │   ├── student/
│   │   │   ├── page.tsx           # Student dashboard
│   │   │   ├── scan/page.tsx      # QR scanner
│   │   │   ├── wallet/page.tsx    # Wallet & transactions
│   │   │   └── history/page.tsx   # Ride history
│   │   ├── driver/
│   │   │   ├── page.tsx           # Driver dashboard
│   │   │   └── route/page.tsx     # Active route with GPS broadcast
│   │   └── admin/
│   │       ├── page.tsx           # Admin overview dashboard
│   │       ├── routes/page.tsx    # Route management (CRUD)
│   │       ├── stops/page.tsx     # Stop management (CRUD + map)
│   │       ├── fares/page.tsx     # Fare matrix configuration
│   │       ├── shuttles/page.tsx  # Shuttle management (CRUD)
│   │       ├── users/page.tsx     # User management
│   │       └── logs/page.tsx      # Ride logs & analytics
│   ├── api/
│   │   ├── auth/[...all]/route.ts # Better Auth handler
│   │   ├── shuttles/
│   │   │   ├── route.ts           # GET all, POST create
│   │   │   └── [id]/route.ts      # GET, PUT, DELETE by id
│   │   ├── routes/
│   │   │   ├── route.ts
│   │   │   └── [id]/route.ts
│   │   ├── stops/
│   │   │   ├── route.ts
│   │   │   └── [id]/route.ts
│   │   ├── fares/
│   │   │   └── route.ts           # Fare matrix CRUD
│   │   ├── rides/
│   │   │   ├── route.ts           # GET rides
│   │   │   ├── tap-in/route.ts    # POST tap-in
│   │   │   └── tap-out/route.ts   # POST tap-out
│   │   ├── wallet/
│   │   │   ├── route.ts           # GET balance
│   │   │   └── topup/route.ts     # POST top-up (admin)
│   │   ├── qr/
│   │   │   └── generate/route.ts  # Generate QR for stops
│   │   └── gps/
│   │       ├── stream/route.ts    # SSE endpoint for GPS
│   │       └── update/route.ts    # POST GPS position update
│   ├── globals.css                # Tailwind + shadcn theme (exists)
│   ├── layout.tsx                 # Root layout (update with providers)
│   └── page.tsx                   # Landing page
│
├── components/
│   ├── ui/                        # shadcn components (DO NOT EDIT)
│   ├── auth/
│   │   ├── login-form.tsx
│   │   └── register-form.tsx
│   ├── dashboard/
│   │   ├── app-sidebar.tsx        # Main sidebar navigation
│   │   ├── header.tsx             # Dashboard header
│   │   ├── stats-card.tsx         # Statistics display card
│   │   └── data-table.tsx         # Reusable data table wrapper
│   ├── maps/
│   │   ├── shuttle-map.tsx        # Live shuttle tracking map
│   │   ├── route-editor.tsx       # Route/stop editor on map
│   │   └── stop-marker.tsx        # Custom stop marker
│   ├── scanner/
│   │   └── qr-scanner.tsx         # QR code scanner component
│   ├── wallet/
│   │   ├── balance-card.tsx       # Wallet balance display
│   │   └── transaction-list.tsx   # Transaction history
│   └── providers/
│       ├── auth-provider.tsx      # Better Auth context
│       └── theme-provider.tsx     # Dark/light mode provider
│
├── lib/
│   ├── auth.ts                    # Better Auth server config
│   ├── auth-client.ts             # Better Auth client
│   ├── db/
│   │   ├── index.ts               # Drizzle + Neon connection
│   │   └── schema.ts              # Database schema (all tables)
│   ├── utils.ts                   # cn() utility (exists)
│   └── services/
│       ├── fare.ts                # Fare calculation logic
│       ├── penalty.ts             # Incomplete ride detection
│       └── qr.ts                  # QR generation/validation
│
├── hooks/
│   ├── use-gps-stream.ts          # SSE GPS subscription hook
│   ├── use-session.ts             # Auth session hook wrapper
│   └── use-geolocation.ts         # Browser geolocation hook
│
├── types/
│   └── index.ts                   # TypeScript type definitions
│
├── drizzle/
│   └── migrations/                # Auto-generated migrations
│
├── public/
│   └── ... (existing files)
│
├── .env.local                     # Environment variables
├── drizzle.config.ts              # Drizzle CLI configuration
├── components.json                # shadcn config (exists)
├── next.config.ts                 # Next.js config
├── tsconfig.json                  # TypeScript config (exists)
└── package.json                   # Dependencies
```

---

## Database Schema

### Tables

#### Better Auth Managed Tables
- `user` - Extended with `role` field (student, driver, admin)
- `session` - Session management
- `account` - OAuth accounts (if added later)
- `verification` - Email verification tokens

#### Application Tables

```sql
-- Shuttles
CREATE TABLE shuttle (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL,
  plate_number VARCHAR(20) NOT NULL UNIQUE,
  capacity INTEGER NOT NULL,
  status VARCHAR(20) DEFAULT 'inactive', -- active, inactive, maintenance
  current_route_id UUID REFERENCES route(id),
  current_driver_id TEXT REFERENCES user(id),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Routes (circular)
CREATE TABLE route (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL,
  description TEXT,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Stops
CREATE TABLE stop (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  route_id UUID NOT NULL REFERENCES route(id) ON DELETE CASCADE,
  name VARCHAR(100) NOT NULL,
  latitude DOUBLE PRECISION NOT NULL,
  longitude DOUBLE PRECISION NOT NULL,
  sequence_order INTEGER NOT NULL, -- Order in circular route (1, 2, 3...)
  qr_code VARCHAR(255) UNIQUE, -- Unique QR identifier
  created_at TIMESTAMP DEFAULT NOW()
);

-- Fare Matrix (from stop A to stop B)
CREATE TABLE fare_matrix (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  route_id UUID NOT NULL REFERENCES route(id) ON DELETE CASCADE,
  from_stop_id UUID NOT NULL REFERENCES stop(id) ON DELETE CASCADE,
  to_stop_id UUID NOT NULL REFERENCES stop(id) ON DELETE CASCADE,
  fare DECIMAL(10, 2) NOT NULL,
  UNIQUE(route_id, from_stop_id, to_stop_id)
);

-- Rides (tap-in/tap-out records)
CREATE TABLE ride (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id TEXT NOT NULL REFERENCES user(id),
  shuttle_id UUID NOT NULL REFERENCES shuttle(id),
  route_id UUID NOT NULL REFERENCES route(id),
  entry_stop_id UUID NOT NULL REFERENCES stop(id),
  exit_stop_id UUID REFERENCES stop(id), -- NULL until tap-out
  entry_time TIMESTAMP DEFAULT NOW() NOT NULL,
  exit_time TIMESTAMP, -- NULL until tap-out
  fare DECIMAL(10, 2), -- Calculated on tap-out
  status VARCHAR(20) DEFAULT 'active', -- active, completed, penalty
  penalty_applied BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Wallets
CREATE TABLE wallet (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id TEXT NOT NULL REFERENCES user(id) UNIQUE,
  balance DECIMAL(10, 2) DEFAULT 0.00,
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Transactions
CREATE TABLE transaction (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  wallet_id UUID NOT NULL REFERENCES wallet(id),
  type VARCHAR(20) NOT NULL, -- credit, debit, penalty
  amount DECIMAL(10, 2) NOT NULL,
  description TEXT,
  ride_id UUID REFERENCES ride(id),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Shuttle Locations (real-time GPS - optional persistence)
CREATE TABLE shuttle_location (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  shuttle_id UUID NOT NULL REFERENCES shuttle(id),
  latitude DOUBLE PRECISION NOT NULL,
  longitude DOUBLE PRECISION NOT NULL,
  heading DOUBLE PRECISION, -- Direction in degrees
  speed DOUBLE PRECISION, -- km/h
  recorded_at TIMESTAMP DEFAULT NOW()
);
```

---

## Dependencies

### To Install

```bash
# Database & ORM
npm add @neondatabase/serverless drizzle-orm
npm add -D drizzle-kit

# Authentication
npm add better-auth

# QR Code
npm add qrcode html5-qrcode
npm add -D @types/qrcode

# Google Maps
npm add @react-google-maps/api

# Form handling
npm add react-hook-form @hookform/resolvers zod

# Date handling
npm add date-fns
```

### shadcn Components to Add

```bash
# Core UI
npx shadcn@latest add button input label

# Forms
npx shadcn@latest add form select checkbox switch textarea

# Layout
npx shadcn@latest add card sidebar sheet separator scroll-area

# Data Display
npx shadcn@latest add table badge avatar skeleton

# Feedback
npx shadcn@latest add dialog alert-dialog dropdown-menu sonner tooltip

# Navigation
npx shadcn@latest add tabs pagination
```

---

## Environment Variables

```env
# .env.local

# Database (Neon)
DATABASE_URL="postgresql://user:password@ep-xxx.region.aws.neon.tech/shuttly?sslmode=require"

# Better Auth
BETTER_AUTH_SECRET="generate-a-32-char-minimum-secret-key-here"
BETTER_AUTH_URL="http://localhost:3000"

# Google Maps
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY="your-google-maps-api-key"

# App Config
NEXT_PUBLIC_APP_URL="http://localhost:3000"
DEFAULT_PENALTY_AMOUNT="50.00"
```

---

## Implementation Plan (4 Weeks)

### Week 1: Foundation & Authentication

#### Day 1-2: Database Setup
- [ ] Create Neon project and get connection string
- [ ] Install database dependencies (`@neondatabase/serverless`, `drizzle-orm`, `drizzle-kit`)
- [ ] Create `lib/db/index.ts` - Drizzle + Neon connection
- [ ] Create `lib/db/schema.ts` - Complete database schema
- [ ] Create `drizzle.config.ts`
- [ ] Run `npx drizzle-kit generate` to create migrations
- [ ] Run `npx drizzle-kit migrate` to apply migrations

#### Day 3-4: Authentication Setup
- [ ] Install Better Auth (`npm add better-auth`)
- [ ] Create `lib/auth.ts` - Better Auth server configuration
- [ ] Create `lib/auth-client.ts` - Better Auth React client
- [ ] Create `app/api/auth/[...all]/route.ts` - Auth API handler
- [ ] Create `components/providers/auth-provider.tsx`
- [ ] Update `app/layout.tsx` with auth provider

#### Day 5-6: Auth UI & Protection
- [ ] Add shadcn components: `button`, `input`, `label`, `form`, `card`
- [ ] Create `app/(auth)/login/page.tsx`
- [ ] Create `app/(auth)/register/page.tsx`
- [ ] Create `components/auth/login-form.tsx`
- [ ] Create `components/auth/register-form.tsx`
- [ ] Create `proxy.ts` for route protection
- [ ] Test authentication flow

#### Day 7: Basic Layout
- [ ] Add shadcn `sidebar`, `sheet`, `separator`, `avatar`, `dropdown-menu`
- [ ] Create `app/(dashboard)/layout.tsx`
- [ ] Create `components/dashboard/app-sidebar.tsx`
- [ ] Create `components/dashboard/header.tsx`
- [ ] Create role-based navigation (student/driver/admin views)

---

### Week 2: Core CRUD & QR System

#### Day 8-9: Admin - Route & Stop Management
- [ ] Add shadcn `table`, `dialog`, `alert-dialog`, `select`
- [ ] Create `app/api/routes/route.ts` - GET, POST
- [ ] Create `app/api/routes/[id]/route.ts` - GET, PUT, DELETE
- [ ] Create `app/api/stops/route.ts` - GET, POST
- [ ] Create `app/api/stops/[id]/route.ts` - GET, PUT, DELETE
- [ ] Create `app/(dashboard)/admin/routes/page.tsx`
- [ ] Create `app/(dashboard)/admin/stops/page.tsx`
- [ ] Add Google Maps for stop location picking

#### Day 10-11: Admin - Fare Matrix
- [ ] Create `app/api/fares/route.ts` - CRUD
- [ ] Create `app/(dashboard)/admin/fares/page.tsx`
- [ ] Build fare matrix UI (from stop -> to stop -> price)
- [ ] Implement auto-calculation for circular routes

#### Day 12-13: Admin - Shuttle Management
- [ ] Create `app/api/shuttles/route.ts` - CRUD
- [ ] Create `app/api/shuttles/[id]/route.ts`
- [ ] Create `app/(dashboard)/admin/shuttles/page.tsx`
- [ ] Assign drivers to shuttles
- [ ] Assign routes to shuttles

#### Day 14: QR Code System
- [ ] Install QR packages (`npm add qrcode html5-qrcode`)
- [ ] Create `lib/services/qr.ts` - QR generation & validation
- [ ] Create `app/api/qr/generate/route.ts`
- [ ] Generate unique QR codes for each stop
- [ ] Create `components/scanner/qr-scanner.tsx`
- [ ] Create `app/(dashboard)/student/scan/page.tsx`

---

### Week 3: Rides, GPS & Wallet

#### Day 15-16: Tap-In/Tap-Out System
- [ ] Create `app/api/rides/tap-in/route.ts`
- [ ] Create `app/api/rides/tap-out/route.ts`
- [ ] Create `lib/services/fare.ts` - Fare calculation logic
- [ ] Implement circular route distance calculation
- [ ] Test tap-in/tap-out flow

#### Day 17-18: GPS & Real-Time Tracking
- [ ] Create `app/api/gps/update/route.ts` - Receive GPS from driver
- [ ] Create `app/api/gps/stream/route.ts` - SSE endpoint
- [ ] Create `hooks/use-gps-stream.ts` - Client-side SSE hook
- [ ] Create `hooks/use-geolocation.ts` - Browser geolocation
- [ ] Create `app/(dashboard)/driver/route/page.tsx` - GPS broadcast UI
- [ ] Implement simulation mode for testing

#### Day 19-20: Google Maps Integration
- [ ] Install `@react-google-maps/api`
- [ ] Create `components/maps/shuttle-map.tsx`
- [ ] Display live shuttle positions
- [ ] Show route with stops
- [ ] Update `app/(dashboard)/student/page.tsx` with map

#### Day 21: Wallet System
- [ ] Create `app/api/wallet/route.ts` - GET balance
- [ ] Create `app/api/wallet/topup/route.ts` - Admin top-up
- [ ] Create `components/wallet/balance-card.tsx`
- [ ] Create `components/wallet/transaction-list.tsx`
- [ ] Create `app/(dashboard)/student/wallet/page.tsx`
- [ ] Auto-deduct fare on tap-out

---

### Week 4: Penalties, Polish & PWA

#### Day 22-23: Incomplete Ride Detection
- [ ] Create `lib/services/penalty.ts`
- [ ] Implement cycle completion detection logic
- [ ] Create background job/cron for checking incomplete rides
- [ ] Auto-apply penalty to wallet
- [ ] Add penalty transaction records
- [ ] Add sonner notifications for penalties

#### Day 24-25: Admin Dashboard & Logs
- [ ] Create `app/(dashboard)/admin/page.tsx` - Overview with stats
- [ ] Create `components/dashboard/stats-card.tsx`
- [ ] Create `app/(dashboard)/admin/logs/page.tsx` - Ride logs
- [ ] Create `app/(dashboard)/admin/users/page.tsx` - User management
- [ ] Add filtering and search to logs

#### Day 26-27: Student Features
- [ ] Create `app/(dashboard)/student/page.tsx` - Dashboard with active ride
- [ ] Create `app/(dashboard)/student/history/page.tsx` - Ride history
- [ ] Show current ride status
- [ ] Show wallet balance prominently

#### Day 28: PWA & Final Polish
- [ ] Create `public/manifest.json`
- [ ] Add PWA icons
- [ ] Configure service worker (optional)
- [ ] Add `components/providers/theme-provider.tsx`
- [ ] Implement dark mode toggle
- [ ] Final testing of all flows
- [ ] Bug fixes and code cleanup

---

## Key Implementation Details

### 1. Better Auth Configuration

```typescript
// lib/auth.ts
import { betterAuth } from "better-auth";
import { drizzleAdapter } from "better-auth/adapters/drizzle";
import { nextCookies } from "better-auth/next-js";
import { db } from "./db";

export const auth = betterAuth({
  database: drizzleAdapter(db, { provider: "pg" }),
  emailAndPassword: { enabled: true },
  user: {
    additionalFields: {
      role: { 
        type: "string", 
        defaultValue: "student",
        input: false // Don't allow users to set their own role
      },
    },
  },
  plugins: [nextCookies()], // Must be last plugin
});

export type Session = typeof auth.$Infer.Session;
```

### 2. Drizzle + Neon Connection

```typescript
// lib/db/index.ts
import { neon } from '@neondatabase/serverless';
import { drizzle } from 'drizzle-orm/neon-http';
import * as schema from './schema';

const sql = neon(process.env.DATABASE_URL!);
export const db = drizzle(sql, { schema });
```

### 3. SSE for GPS Streaming

```typescript
// app/api/gps/stream/route.ts
export const runtime = 'nodejs';
export const dynamic = 'force-dynamic';

export async function GET(request: Request) {
  const encoder = new TextEncoder();
  
  const stream = new ReadableStream({
    start(controller) {
      const sendUpdate = async () => {
        // Fetch current shuttle positions from DB or cache
        const positions = await getShuttlePositions();
        const data = `data: ${JSON.stringify(positions)}\n\n`;
        controller.enqueue(encoder.encode(data));
      };

      // Send initial data
      sendUpdate();
      
      // Send updates every 3 seconds
      const interval = setInterval(sendUpdate, 3000);

      request.signal.addEventListener('abort', () => {
        clearInterval(interval);
        controller.close();
      });
    },
  });

  return new Response(stream, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive',
    },
  });
}
```

### 4. Fare Calculation Logic

```typescript
// lib/services/fare.ts
import { db } from "@/lib/db";
import { fareMatrix, stop } from "@/lib/db/schema";
import { and, asc, eq } from "drizzle-orm";

export async function calculateFare(
  routeId: string,
  entryStopId: string,
  exitStopId: string
): Promise<number> {
  // Get fare from matrix
  const fare = await db.query.fareMatrix.findFirst({
    where: and(
      eq(fareMatrix.routeId, routeId),
      eq(fareMatrix.fromStopId, entryStopId),
      eq(fareMatrix.toStopId, exitStopId)
    ),
  });

  if (fare) return Number(fare.fare);

  // Fallback: Calculate based on stop distance
  const stops = await db.query.stop.findMany({
    where: eq(stop.routeId, routeId),
    orderBy: [asc(stop.sequenceOrder)],
  });

  const entryIndex = stops.findIndex(s => s.id === entryStopId);
  const exitIndex = stops.findIndex(s => s.id === exitStopId);
  
  // Handle circular route (if exit < entry, went around)
  let distance = exitIndex - entryIndex;
  if (distance < 0) {
    distance = stops.length + distance;
  }

  const BASE_FARE = 10;
  const PER_STOP_FARE = 5;
  return BASE_FARE + (distance * PER_STOP_FARE);
}
```

### 5. Incomplete Ride Detection

```typescript
// lib/services/penalty.ts
import { db } from "@/lib/db";
import { ride, wallet, transaction } from "@/lib/db/schema";
import { and, eq, lt, sql } from "drizzle-orm";

export async function detectIncompleteRides() {
  const CYCLE_TIMEOUT_MINUTES = 90; // Max time for one cycle
  const PENALTY_AMOUNT = Number(process.env.DEFAULT_PENALTY_AMOUNT || 50);

  // Find rides that are still "active" but exceeded timeout
  const incompleteRides = await db.query.ride.findMany({
    where: and(
      eq(ride.status, 'active'),
      lt(ride.entryTime, new Date(Date.now() - CYCLE_TIMEOUT_MINUTES * 60 * 1000))
    ),
  });

  for (const incompleteRide of incompleteRides) {
    await db.transaction(async (tx) => {
      // Mark ride as penalty
      await tx.update(ride)
        .set({ status: 'penalty', penaltyApplied: true })
        .where(eq(ride.id, incompleteRide.id));

      // Deduct from wallet
      const userWallet = await tx.query.wallet.findFirst({
        where: eq(wallet.userId, incompleteRide.userId),
      });

      if (userWallet) {
        await tx.update(wallet)
          .set({ balance: sql`${wallet.balance} - ${PENALTY_AMOUNT}` })
          .where(eq(wallet.id, userWallet.id));

        // Record transaction
        await tx.insert(transaction).values({
          walletId: userWallet.id,
          type: 'penalty',
          amount: PENALTY_AMOUNT.toString(),
          description: 'Incomplete ride penalty - no tap-out detected',
          rideId: incompleteRide.id,
        });
      }
    });
  }
}
```

---

## User Flows

### Student Flow
1. Login → Dashboard (see active shuttles on map, wallet balance)
2. Board shuttle → Scan QR at entry stop
3. System records entry (stop, time, shuttle)
4. Exit shuttle → Scan QR at exit stop
5. System calculates fare, deducts from wallet
6. View ride history and transactions

### Driver Flow
1. Login → Driver Dashboard
2. Start route → Select assigned shuttle/route
3. System starts GPS broadcasting
4. Drive route → GPS updates sent every 3 seconds
5. End route → Stop GPS broadcasting

### Admin Flow
1. Login → Admin Dashboard (overview stats)
2. Manage routes → Create/edit circular routes
3. Manage stops → Add stops with map coordinates, generate QR
4. Configure fares → Set fare matrix
5. Manage shuttles → Assign drivers, routes
6. View logs → See all rides, filter by date/status
7. Manage users → View/edit user roles, top-up wallets

---

## API Endpoints Summary

| Method | Endpoint | Description |
|--------|----------|-------------|
| * | `/api/auth/*` | Better Auth handler |
| GET | `/api/shuttles` | List all shuttles |
| POST | `/api/shuttles` | Create shuttle |
| GET | `/api/shuttles/[id]` | Get shuttle by ID |
| PUT | `/api/shuttles/[id]` | Update shuttle |
| DELETE | `/api/shuttles/[id]` | Delete shuttle |
| GET | `/api/routes` | List all routes |
| POST | `/api/routes` | Create route |
| GET | `/api/routes/[id]` | Get route with stops |
| PUT | `/api/routes/[id]` | Update route |
| DELETE | `/api/routes/[id]` | Delete route |
| GET | `/api/stops` | List stops (filter by route) |
| POST | `/api/stops` | Create stop |
| PUT | `/api/stops/[id]` | Update stop |
| DELETE | `/api/stops/[id]` | Delete stop |
| GET | `/api/fares` | Get fare matrix for route |
| POST | `/api/fares` | Create/update fare entry |
| POST | `/api/rides/tap-in` | Record tap-in |
| POST | `/api/rides/tap-out` | Record tap-out, calculate fare |
| GET | `/api/rides` | List rides (with filters) |
| GET | `/api/wallet` | Get user wallet balance |
| POST | `/api/wallet/topup` | Admin: top-up wallet |
| POST | `/api/qr/generate` | Generate QR for stop |
| GET | `/api/gps/stream` | SSE: Real-time GPS positions |
| POST | `/api/gps/update` | Driver: Update GPS position |

---

## Testing Checklist

### Authentication
- [ ] User can register with email/password
- [ ] User can login
- [ ] User can logout
- [ ] Protected routes redirect to login
- [ ] Role-based access works (student/driver/admin)

### Admin Functions
- [ ] Can create/edit/delete routes
- [ ] Can create/edit/delete stops with map
- [ ] Can configure fare matrix
- [ ] Can create/edit/delete shuttles
- [ ] Can assign drivers to shuttles
- [ ] Can view all ride logs
- [ ] Can top-up user wallets

### Student Functions
- [ ] Can view dashboard with map
- [ ] Can scan QR code (tap-in)
- [ ] Can scan QR code (tap-out)
- [ ] Fare is calculated correctly
- [ ] Wallet is debited on tap-out
- [ ] Can view ride history
- [ ] Can view wallet transactions

### Driver Functions
- [ ] Can start route
- [ ] GPS is broadcast while route active
- [ ] Can end route
- [ ] Position appears on student maps

### Penalties
- [ ] Incomplete rides detected after timeout
- [ ] Penalty applied to wallet
- [ ] Transaction recorded
- [ ] Ride marked as penalty

---

## Notes

- All times should be stored in UTC
- GPS coordinates: latitude/longitude as DOUBLE PRECISION
- Fare amounts: DECIMAL(10, 2) for currency precision
- Use Zod for all API input validation
- Use React Hook Form for all forms
- Use Sonner for toast notifications
- Implement proper error handling and loading states
