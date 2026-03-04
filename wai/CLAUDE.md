# BLT
Primary project context loaded at the start of every session. Add coding standards, build commands, architectural decisions, and domain knowledge here.

## About This Project
BLT is an acronym of the company name Beyond Limited Thinking, LLC (bltwai.com).  BLT Core is an AI-native, modular point-of-sale (POS) infrastructure built for restaurants and retail. BLTPos A vibe coded  AI-native restaurant point-of-sale system. Our focus is on composability, open standards, and AI-first workflows—supporting rapid, flexible integration with banks, independent sales prganization (ISOs), and strategic partners. BLT provides a payments-agnostic alternative to legacy POS software, enabling full control of the merchant relationship and economics.

BLT POS is an AI-native restaurant point-of-sale system built by Beyond Limited Thinking, LLC. It is a modern, composable, payments-agnostic alternative to legacy POS software — designed for tablets and touch interfaces.

## About the Company
- **Industry:** Tech / Software — Enterprise Software / Fintech / Payments
- **Company Name:** Beyond Limited Thinking, LLC (a.k.a BLT)
- **Product:** BLT Core — AI-native restaurant point-of-sale infrastructure
- **Value proposition:** "Payments freedom" — gives banks, ISOs, and strategic partners full ownership of the merchant relationship and payments economics
- **Differentiation:** Modern, scalable, payments-agnostic alternative to legacy POS incumbents. No forced processor lock-in, no proprietary hardware requirements, no hidden fees. Competitive parity with legacy systems but built AI-native.


## Tech Stack
- **Runtime:** Bun >= 1.0.0
- **Framework:** React 18, TypeScript (ESNext), Vite 5
- **UI Components:** Radix UI (headless) + shadcn/ui (styled wrappers) + CVA (variants)
- **Styling:** Tailwind CSS 3 with HSL CSS custom properties
- **State:** React Context (7+ providers) + TanStack React Query 5
- **Forms:** React Hook Form + Zod validation
- **Routing:** React Router v6 with lazy-loaded pages
- **Backend:** Supabase (auth, PostgreSQL, realtime subscriptions, storage)
- **Icons:** Lucide React
- **Charts:** Recharts
- **Font:** Inter (Latin, weights 300-900 via @fontsource)
- **Path alias:** `@/*` maps to `./src/*`

### BLT-Specific
- **Payments:** Stripe (`@stripe/stripe-js`) + Stripe Terminal for card readers
- **Device ID:** FingerprintJS for device fingerprinting
- **PDF:** jsPDF + html2canvas for receipt/report generation

## Build & Run
```bash
bun install                # install dependencies
bun run dev                # vite dev server (port 8080)
bun run build:dev          # build for development
bun run build:demo         # build for demo environment
bun run build:test         # build for test environment
bun run lint               # eslint
bun run typecheck          # tsc --noEmit
bun run test               # vitest
bun run test:run           # vitest single run
bun run test:coverage      # vitest with coverage
```

## Project Structure
```
src/
  pages/           # Route-based page components (lazy-loaded)
  components/
    ui/            # shadcn/ui primitives (button, card, dialog, input, etc.)
    layout/        # DashboardLayout, DashboardHeader
    <domain>/      # Business components grouped by domain
  contexts/        # React Context providers (Auth, Theme, Role, Settings, Module, Shift, etc.)
  hooks/           # Custom React hooks (40+)
  api/             # API client functions (*.Api.ts)
  services/        # Business logic services
  lib/             # Utilities (utils.ts with cn(), formatCurrency, etc.)
  types/           # TypeScript type definitions
  integrations/    # Supabase client setup
  config/          # App configuration
```

## Design System

### Color System
Colors use HSL values as CSS custom properties in `src/index.css`, consumed via Tailwind as `hsl(var(--token))`.

**Semantic tokens:**
- `--primary` (orange 26 100% 50%), `--secondary`, `--accent`, `--muted`, `--destructive`
- `--success`, `--warning` (restaurant-specific)
- `--background`, `--foreground`, `--card`, `--popover`, `--border`, `--input`, `--ring`
- Each token has a matching `-foreground` variant

**Static palette** (in `tailwind.config.ts`):
- `core.black`, `core.yellow`, `core.gray.100-900`

#### BLT-Specific: Category & Status Colors
- 12 category colors: `--category-orange`, `--category-amber`, `--category-yellow`, `--category-emerald`, `--category-cyan`, `--category-purple`, `--category-red`, `--category-green`, `--category-blue`, `--category-orange-alt`, `--category-violet`, `--category-teal`
- 9 order status colors (each with `-fg` and `-border` variants): `--status-open`, `--status-suspended`, `--status-sent`, `--status-in-progress`, `--status-ready`, `--status-served`, `--status-closed`, `--status-voided`, `--status-refunded`

### Typography
- Font: `Inter, sans-serif` (set in `tailwind.config.ts`)
- Weights: 300 (light) through 900 (black)
- Standard Tailwind type scale (`text-sm`, `text-base`, `text-lg`, etc.)

### Spacing & Layout
- Border radius: `--radius: 0.5rem` with Tailwind `lg/md/sm` derived from it
- Container: centered, `2rem` padding, max `1400px`
- Touch targets: minimum `44px` via `.tablet-optimized` class

### Shadows & Transitions
Design tokens defined as CSS variables:
- `--shadow-soft` / `--shadow-medium` / `--shadow-strong`
- `--transition-smooth`: `all 0.3s cubic-bezier(0.4, 0, 0.2, 1)`
- `--transition-bounce`: `all 0.3s cubic-bezier(0.68, -0.55, 0.265, 1.55)`

### Dark Mode
- Class-based switching (`.dark` on `<html>`)
- All semantic tokens have dark-mode variants in `src/index.css`
- Theme persisted in localStorage (key: `bltpos-theme`)
- Flash prevention: theme class applied synchronously in `main.tsx` before React mounts
- Color scheme caching in localStorage for instant application

### Custom Utility Classes (`src/index.css`)
```
.core-card         — bg-card rounded-xl shadow-sm border
.core-card-hover   — hover:shadow-md hover:border-accent
.core-button       — rounded-lg px-4 py-2 font-medium
.core-input        — px-3 py-2 bg-background border rounded-lg focus:ring-2
.tablet-optimized  — touch-manipulation min-h-[44px] min-w-[44px]
.bg-gradient-primary / .bg-gradient-card
.shadow-soft / .shadow-medium / .shadow-strong
.transition-smooth / .transition-bounce
```

## Component Patterns

### UI Primitives (`src/components/ui/`)
shadcn/ui components built on Radix UI headless primitives:
- Always use `React.forwardRef` and set `displayName`
- Variants defined with CVA (`class-variance-authority`)
- Class merging: `cn()` = `twMerge(clsx())` from `src/lib/utils.ts`
- Polymorphism: `asChild` prop via `@radix-ui/react-slot`
- Compound components: export named subcomponents (e.g., `Card`, `CardHeader`, `CardTitle`, `CardContent`, `CardFooter`)

### Business Components (`src/components/<domain>/`)
- Import and compose UI primitives
- Props defined as TypeScript interfaces
- Early return pattern for null/loading guards
- Icons from `lucide-react`
- Domain-specific styling via Tailwind utilities

### Pages (`src/pages/`)
- Lazy-loaded with `React.lazy()` (Login and Dashboard are static imports)
- Wrapped in `<Suspense>` with a loading fallback
- Use `DashboardLayout` for consistent page chrome
- Compose multiple context hooks, combine loading states into a single `isLoading`

## State Management

### React Context (`src/contexts/`)
- Providers: Auth, Theme, Role, Settings, Module, Animation, Shift (+ Orders, Quick, Location)
- Nested in `App.tsx` in dependency order
- Each provider exports a custom `use*` hook that throws if used outside the provider
- Context values wrapped in `useMemo` to avoid unnecessary re-renders
- Stable callbacks via `useCallback` with correct dependency arrays
- Cache pattern: `CacheEntry<T>` with `{ data: T; timestamp: number }` for TTL-based caching

### TanStack React Query
- `useQuery` with `queryKey` arrays for automatic caching/deduplication
- `useMutation` with `onSuccess` that calls `queryClient.invalidateQueries` + shows toast
- Conditional fetching with `enabled` flag
- `staleTime` for cache freshness control

## Data Layer

### API Functions (`src/api/`)
- Organized as object namespaces: `orderApi.createOrder()`, `orderApi.getOrder()`
- Supabase RPC calls with `p_`-prefixed parameters
- Custom error classes: `class OrderApiError extends Error { code, details }`
- Always check Supabase errors: `if (error) throw new ApiError(...)`

### Data Hooks (`src/hooks/`)
- CRUD hooks per entity: `useMenuItems()`, `useCreateMenuItem()`, `useUpdateMenuItem()`
- Queries return `{ data, isLoading, error }`
- Mutations invalidate related query keys on success
- Toast notifications for user feedback on success/error

### Supabase Realtime
- Channel subscriptions in contexts via `supabase.channel().on('postgres_changes', ...).subscribe()`
- Cleanup in `useEffect` return with `supabase.removeChannel(channel)`

## Forms
- `FormProvider` wraps entire form (from React Hook Form)
- Composition: `FormField` > `FormItem` > `FormLabel` + `FormControl` + `FormMessage`
- Validation: Zod schemas resolved via `@hookform/resolvers/zod`
- shadcn form components in `src/components/ui/form.tsx`

## Error Handling
- `ErrorBoundary` class component (`src/components/ui/ErrorBoundary.tsx`) with retry/refresh buttons
- Custom error classes with `code` and `details` fields for API errors
- Try-catch in all async handlers
- Toast notifications (via sonner) for user-facing error messages
- Supabase pattern: `const { data, error } = await supabase.rpc(...); if (error) throw ...`

## Coding Conventions

### Do
- Use `cn()` for all Tailwind class merging
- Use `React.forwardRef` + `displayName` on UI components
- Use CVA for component variants
- Use Tailwind utilities; reference design tokens via `hsl(var(--token))`
- Use `useCallback`/`useMemo` to stabilize context values
- Use Supabase RPC for database operations
- Use `@/*` path alias for imports
- Use `formatCurrency()` from `@/lib/utils` for money formatting
- Use `getInitials()` from `@/lib/utils` for avatar fallbacks
- Ensure minimum 44px touch targets for tablet interfaces
- Lazy-load pages with `React.lazy()`
- Invalidate React Query cache after mutations

### Don't
- Don't use `console.log` in production code — use toast for user feedback
- Don't hardcode colors — use CSS variables and Tailwind semantic tokens
- Don't skip loading states — combine and handle `isLoading` from all hooks
- Don't create inline styles when a Tailwind utility or CSS variable exists
- Don't add dependencies without checking if Bun or existing libs cover the need

## Testing
- **Framework:** Vitest with jsdom environment
- **Component testing:** React Testing Library (`@testing-library/react`)
- **User interaction:** `@testing-library/user-event`
- **Assertions:** `@testing-library/jest-dom` matchers
- **Coverage:** `@vitest/coverage-v8`
- Run: `bun run test` (watch) / `bun run test:run` (single) / `bun run test:coverage`

## TypeScript Configuration
- `strict: false`, `noImplicitAny: false`, `strictNullChecks: false`
- Module: ESNext, Resolution: bundler, Target: ESNext
- JSX: react-jsx (new transform)
- `skipLibCheck: true` for faster compilation
