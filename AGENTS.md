# AGENTS.md - Shuttly Development Guide

## Project Overview

Shuttly is a smart shuttle tracking and fare enforcement system built with Next.js 16, featuring QR-based boarding, GPS tracking, automated fare calculation, and penalty enforcement.

---

## Build/Lint/Test Commands

```bash
# Development
npm run dev          # Start development server (next dev)

# Production
npm run build        # Build for production (next build)
npm run start        # Start production server (next start)

# Linting
npm run lint         # Run ESLint

# Database (Drizzle ORM)
npx drizzle-kit generate    # Generate migrations
npx drizzle-kit migrate     # Apply migrations
npx drizzle-kit studio      # Open Drizzle Studio GUI

# shadcn/ui Components
npx shadcn@latest add <component>   # Add a component (e.g., button, card, form)
```

### Running Single Tests

Testing framework is not yet configured. When added, use:
```bash
# Vitest (recommended for Next.js)
npx vitest run path/to/test.test.ts     # Run single test file
npx vitest run -t "test name"           # Run test by name
```

---

## Tech Stack

| Layer          | Technology                              |
|----------------|----------------------------------------|
| Framework      | Next.js 16 (App Router, RSC)           |
| UI Components  | shadcn/ui (new-york style, neutral)    |
| Styling        | Tailwind CSS v4 + CSS variables        |
| Database       | Neon (Serverless PostgreSQL)           |
| ORM            | Drizzle ORM + @neondatabase/serverless |
| Authentication | Better Auth                            |
| Real-time      | Server-Sent Events (SSE)               |
| Maps           | Google Maps API                        |
| QR Scanning    | html5-qrcode                           |
| Forms          | react-hook-form + zod                  |

---

## Code Style Guidelines

### TypeScript

- **Strict mode enabled** - No implicit any, strict null checks
- **Target**: ES2017 with ESNext modules
- Always define explicit types for function parameters and return values
- Use `interface` for object shapes, `type` for unions/intersections

### Imports

Use path aliases configured in tsconfig.json:
```typescript
import { cn } from "@/lib/utils"
import { Button } from "@/components/ui/button"
import { useSession } from "@/hooks/use-session"
import { db } from "@/lib/db"
import type { User } from "@/types"
```

**Import order:**
1. React/Next.js imports
2. Third-party libraries
3. Path alias imports (`@/`)
4. Relative imports
5. Type imports (use `import type` for type-only imports)

### Formatting

- **No Prettier configured** - Follow ESLint rules
- Use 2-space indentation
- Use double quotes for JSX attributes
- Use template literals for string interpolation
- Trailing commas in multi-line arrays/objects

### Naming Conventions

| Type              | Convention        | Example                    |
|-------------------|-------------------|----------------------------|
| Components        | PascalCase        | `ShuttleMap.tsx`           |
| Hooks             | camelCase, use-   | `use-gps-stream.ts`        |
| Utilities         | camelCase         | `calculateFare.ts`         |
| Constants         | SCREAMING_SNAKE   | `DEFAULT_PENALTY_AMOUNT`   |
| Types/Interfaces  | PascalCase        | `interface User {}`        |
| Database tables   | snake_case        | `fare_matrix`, `shuttle`   |
| API routes        | kebab-case        | `tap-in/route.ts`          |
| Files             | kebab-case        | `balance-card.tsx`         |

### React/Next.js Patterns

```typescript
// Server Components (default in app/)
export default async function Page() {
  const data = await fetchData()
  return <Component data={data} />
}

// Client Components
"use client"
export function InteractiveComponent() {
  const [state, setState] = useState()
  return <div onClick={() => setState(...)}>...</div>
}

// API Routes
export async function GET(request: Request) {
  return Response.json({ data })
}

export async function POST(request: Request) {
  const body = await request.json()
  // Validate with zod
  return Response.json({ success: true })
}
```

### Error Handling

- Use try-catch for async operations
- Return proper HTTP status codes in API routes
- Use Zod for input validation on all API endpoints
- Display user-friendly errors with Sonner toast notifications

```typescript
// API route error handling
try {
  const validated = schema.parse(await request.json())
  // ... logic
  return Response.json({ data })
} catch (error) {
  if (error instanceof z.ZodError) {
    return Response.json({ error: error.errors }, { status: 400 })
  }
  return Response.json({ error: "Internal server error" }, { status: 500 })
}
```

---

## Development Rules (CRITICAL)

1. **shadcn Components**: Always use CLI to add (`npx shadcn@latest add <component>`)
2. **Never edit**: `components/ui/*` files directly - these are generated
3. **Never edit**: Generated CSS files from shadcn
4. **Styling**: Use CSS variables (`bg-primary`, `text-muted-foreground`, etc.)
5. **Dependencies**: Use `npm add <package>` (not manual package.json edits)
6. **Config changes**: Use CLI commands where possible
7. **Documentation**: Always read relevant docs before implementing features

---

## Project Structure

```
app/
├── (auth)/              # Public auth routes (login, register)
├── (dashboard)/         # Protected routes (student, driver, admin)
├── api/                 # API routes
├── globals.css          # Tailwind + shadcn theme
├── layout.tsx           # Root layout
└── page.tsx             # Landing page

components/
├── ui/                  # shadcn components (DO NOT EDIT)
├── auth/                # Auth-related components
├── dashboard/           # Dashboard components
├── maps/                # Google Maps components
├── scanner/             # QR scanner components
├── wallet/              # Wallet components
└── providers/           # Context providers

lib/
├── auth.ts              # Better Auth server config
├── auth-client.ts       # Better Auth client
├── db/
│   ├── index.ts         # Drizzle + Neon connection
│   └── schema.ts        # Database schema
├── utils.ts             # cn() utility
└── services/            # Business logic (fare, penalty, qr)

hooks/                   # Custom React hooks
types/                   # TypeScript type definitions
drizzle/                 # Database migrations
```

---

## Database Conventions

- Use UUID for primary keys (`gen_random_uuid()`)
- Store timestamps in UTC
- Use DECIMAL(10, 2) for currency/fare amounts
- Use snake_case for table and column names
- Always include `created_at` timestamp
- Use proper foreign key constraints with ON DELETE behavior

---

## API Design

- RESTful endpoints under `app/api/`
- Use Zod schemas for request validation
- Return consistent JSON responses: `{ data }` or `{ error }`
- Proper HTTP status codes (200, 201, 400, 401, 403, 404, 500)
- SSE for real-time GPS streaming (`text/event-stream`)

---

## Forms

- Use `react-hook-form` with `@hookform/resolvers/zod`
- Define Zod schemas for all form inputs
- Use shadcn Form components for consistent styling
- Show validation errors inline

---

## Environment Variables

Required in `.env.local`:
```
DATABASE_URL              # Neon PostgreSQL connection string
BETTER_AUTH_SECRET        # 32+ char secret key
BETTER_AUTH_URL           # http://localhost:3000
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY
NEXT_PUBLIC_APP_URL
DEFAULT_PENALTY_AMOUNT    # e.g., 50.00
```

---

## Common Utilities

```typescript
// Class name merging (lib/utils.ts)
import { cn } from "@/lib/utils"
cn("base-class", condition && "conditional-class", className)
```

---

## User Roles

Three roles with different access levels:
- **student**: Dashboard, QR scanning, wallet, ride history
- **driver**: Route management, GPS broadcasting
- **admin**: Full CRUD access, user management, logs
