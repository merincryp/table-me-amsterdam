# Fase 1 — Fundament: Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Transform the static HTML prototype into a working Next.js app with Supabase database and real Formitable availability for 10 restaurants, deployed on Vercel.

**Architecture:** Next.js 14 App Router with server components for the restaurant list, client components for interactivity (search, modal). API routes proxy Formitable requests server-side to keep widget IDs secure. Supabase Postgres stores restaurant data; the frontend queries it via the API layer.

**Tech Stack:** Next.js 14, React 18, Tailwind CSS, Supabase (Postgres), Formitable widget embeds, Vercel hosting, TypeScript

**Spec:** `docs/superpowers/specs/2026-04-08-table-me-amsterdam-design.md`

---

## File Structure

```
table-me-amsterdam/
├── app/
│   ├── layout.tsx                    # Root layout: fonts, metadata, Header/Footer
│   ├── page.tsx                      # Homepage: SearchBar + RestaurantList
│   ├── globals.css                   # Tailwind imports + design tokens
│   └── api/
│       ├── restaurants/route.ts      # GET: restaurant list from Supabase
│       └── availability/route.ts     # GET: Formitable slots for a restaurant
├── components/
│   ├── Header.tsx                    # Logo + tagline
│   ├── Footer.tsx                    # Credits
│   ├── SearchBar.tsx                 # Date + covers picker + search button
│   ├── RestaurantCard.tsx            # Single restaurant with slots
│   ├── RestaurantList.tsx            # List of RestaurantCards with loading state
│   ├── SlotButton.tsx                # Individual time slot button
│   ├── BookingModal.tsx              # Modal: Formitable widget / link / manual
│   └── StatusLine.tsx                # Loading/done status indicator
├── lib/
│   ├── supabase.ts                   # Supabase client singleton
│   ├── formitable.ts                 # Formitable availability fetcher
│   └── types.ts                      # Shared TypeScript types
├── supabase/
│   ├── migrations/
│   │   └── 001_create_restaurants.sql # Initial schema
│   └── seed.sql                      # 10 restaurants from current prototype
├── __tests__/
│   ├── api/
│   │   ├── restaurants.test.ts       # API route tests
│   │   └── availability.test.ts      # Availability API tests
│   └── components/
│       ├── SearchBar.test.tsx         # SearchBar tests
│       ├── RestaurantCard.test.tsx    # RestaurantCard tests
│       └── BookingModal.test.tsx      # BookingModal tests
├── tailwind.config.ts                # Design tokens configuration
├── next.config.ts                    # Next.js config
├── .env.local                        # Supabase keys (gitignored)
├── .env.example                      # Template for env vars
└── package.json
```

---

## Task 1: Next.js Project Setup

**Files:**
- Create: `package.json`, `tsconfig.json`, `next.config.ts`, `tailwind.config.ts`, `postcss.config.js`, `app/globals.css`, `app/layout.tsx`, `app/page.tsx`, `.env.example`, `.env.local`, `.gitignore`
- Remove: `index.html` (after migration complete, kept for reference during build)

- [ ] **Step 1: Initialize Next.js project**

```bash
cd /Users/lennardvanmierlo/Desktop/table-me-amsterdam
npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir=false --import-alias="@/*" --use-npm --yes
```

Note: This will create the project in the current directory. If it complains about existing files, move `index.html` and `vercel.json` aside first:
```bash
mv index.html index.html.bak
mv vercel.json vercel.json.bak
```

Then restore after setup:
```bash
mv index.html.bak index.html.original
```

- [ ] **Step 2: Install dependencies**

```bash
npm install @supabase/supabase-js
npm install -D vitest @testing-library/react @testing-library/jest-dom jsdom @vitejs/plugin-react
```

- [ ] **Step 3: Create vitest config**

Create `vitest.config.ts`:
```typescript
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./__tests__/setup.ts'],
    globals: true,
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, '.'),
    },
  },
})
```

Create `__tests__/setup.ts`:
```typescript
import '@testing-library/jest-dom/vitest'
```

- [ ] **Step 4: Add test script to package.json**

Add to `scripts` in `package.json`:
```json
"test": "vitest run",
"test:watch": "vitest"
```

- [ ] **Step 5: Create env files**

Create `.env.example`:
```
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key
```

Create `.env.local` (actual values — user fills in):
```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
```

- [ ] **Step 6: Update .gitignore**

Append to `.gitignore`:
```
.env.local
.superpowers/
```

- [ ] **Step 7: Verify project runs**

```bash
npm run dev
```

Expected: Next.js dev server starts on http://localhost:3000 with default page.

- [ ] **Step 8: Commit**

```bash
git add -A
git commit -m "feat: initialize Next.js 14 project with TypeScript and Tailwind"
```

---

## Task 2: Design Tokens & Global Styles

**Files:**
- Modify: `tailwind.config.ts`
- Modify: `app/globals.css`
- Modify: `app/layout.tsx`

- [ ] **Step 1: Configure Tailwind with Merlijne's design tokens**

Replace `tailwind.config.ts`:
```typescript
import type { Config } from "tailwindcss";

const config: Config = {
  content: [
    "./app/**/*.{js,ts,jsx,tsx,mdx}",
    "./components/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: {
    extend: {
      colors: {
        cream: "#F2EDE3",
        ivory: "#FAF8F3",
        card: "#FDFCF9",
        border: "#DDD8CE",
        "border-lt": "#EAE6DF",
        silver: "#B8B3A8",
        charcoal: "#1C1916",
        mid: "#6E6860",
        light: "#A8A39A",
        accent: "#8C8378",
        available: "#5A7A5C",
        full: "#A06060",
      },
      fontFamily: {
        heading: ['"Cormorant Garamond"', "serif"],
        body: ['"Jost"', "system-ui", "sans-serif"],
      },
    },
  },
  plugins: [],
};

export default config;
```

- [ ] **Step 2: Set up global styles**

Replace `app/globals.css`:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  body {
    @apply font-body font-light bg-cream text-charcoal antialiased;
  }
}

@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.2; }
}

@keyframes shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

.animate-blink {
  animation: blink 1.2s ease-in-out infinite;
}

.animate-shimmer {
  background: linear-gradient(90deg, #E8E4DC 25%, #DDD9D2 50%, #E8E4DC 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}
```

- [ ] **Step 3: Set up root layout with fonts**

Replace `app/layout.tsx`:
```tsx
import type { Metadata } from "next";
import { Cormorant_Garamond, Jost } from "next/font/google";
import "./globals.css";

const cormorant = Cormorant_Garamond({
  subsets: ["latin"],
  weight: ["300", "400"],
  style: ["normal", "italic"],
  variable: "--font-heading",
  display: "swap",
});

const jost = Jost({
  subsets: ["latin"],
  weight: ["300", "400", "500"],
  variable: "--font-body",
  display: "swap",
});

export const metadata: Metadata = {
  title: "Table me, Amsterdam",
  description:
    "Vind live beschikbare restaurants in Amsterdam. Zoek, filter en reserveer direct.",
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="nl" className={`${cormorant.variable} ${jost.variable}`}>
      <body>{children}</body>
    </html>
  );
}
```

- [ ] **Step 4: Verify fonts and colors load**

```bash
npm run dev
```

Open http://localhost:3000 — verify cream background, correct fonts in devtools.

- [ ] **Step 5: Commit**

```bash
git add tailwind.config.ts app/globals.css app/layout.tsx
git commit -m "feat: add design tokens from Merlijne's prototype (colors, fonts, animations)"
```

---

## Task 3: TypeScript Types

**Files:**
- Create: `lib/types.ts`

- [ ] **Step 1: Define shared types**

Create `lib/types.ts`:
```typescript
export type BookingType = "formitable" | "link" | "manual";

export interface Restaurant {
  id: string;
  name: string;
  slug: string;
  neighborhood_id: string | null;
  cuisine_id: string | null;
  price_range: number | null;
  address: string | null;
  lat: number | null;
  lng: number | null;
  phone: string | null;
  email: string | null;
  website: string | null;
  google_place_id: string | null;
  google_rating: number | null;
  photo_url: string | null;
  photo_override: string | null;
  booking_type: BookingType;
  formitable_id: string | null;
  booking_url: string | null;
  is_curated: boolean;
  curated_note: string | null;
  is_active: boolean;
  // Joined data
  neighborhood?: { name: string; slug: string } | null;
  cuisine?: { name: string; slug: string } | null;
}

export interface TimeSlot {
  time: string;
  fewLeft: boolean;
}

export interface Availability {
  status: "available" | "full" | "manual" | "loading";
  slots: TimeSlot[];
}

export interface RestaurantWithAvailability extends Restaurant {
  availability: Availability | null;
}
```

- [ ] **Step 2: Commit**

```bash
git add lib/types.ts
git commit -m "feat: add shared TypeScript types for restaurants and availability"
```

---

## Task 4: Supabase Setup & Schema

**Files:**
- Create: `lib/supabase.ts`
- Create: `supabase/migrations/001_create_restaurants.sql`
- Create: `supabase/seed.sql`

- [ ] **Step 1: Create Supabase client**

Create `lib/supabase.ts`:
```typescript
import { createClient } from "@supabase/supabase-js";

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!;
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(supabaseUrl, supabaseAnonKey);

// Server-side client with service role for admin operations
export function createServiceClient() {
  return createClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.SUPABASE_SERVICE_ROLE_KEY!
  );
}
```

- [ ] **Step 2: Create migration SQL**

Create `supabase/migrations/001_create_restaurants.sql`:
```sql
-- Lookup tables
CREATE TABLE IF NOT EXISTS cuisines (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name text UNIQUE NOT NULL,
  slug text UNIQUE NOT NULL
);

CREATE TABLE IF NOT EXISTS neighborhoods (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name text UNIQUE NOT NULL,
  slug text UNIQUE NOT NULL,
  lat float8,
  lng float8
);

-- Booking type enum
CREATE TYPE booking_type AS ENUM ('formitable', 'link', 'manual');

-- Main restaurants table
CREATE TABLE IF NOT EXISTS restaurants (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name text NOT NULL,
  slug text UNIQUE NOT NULL,
  neighborhood_id uuid REFERENCES neighborhoods(id),
  cuisine_id uuid REFERENCES cuisines(id),
  price_range int CHECK (price_range BETWEEN 1 AND 3),
  address text,
  lat float8,
  lng float8,
  phone text,
  email text,
  website text,
  google_place_id text,
  google_rating float4,
  photo_url text,
  photo_override text,
  booking_type booking_type NOT NULL DEFAULT 'link',
  formitable_id text,
  booking_url text,
  is_curated boolean DEFAULT false,
  curated_note text,
  is_active boolean DEFAULT true,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
);

-- Indexes for common queries
CREATE INDEX idx_restaurants_neighborhood ON restaurants(neighborhood_id);
CREATE INDEX idx_restaurants_cuisine ON restaurants(cuisine_id);
CREATE INDEX idx_restaurants_booking_type ON restaurants(booking_type);
CREATE INDEX idx_restaurants_is_active ON restaurants(is_active);

-- Enable Row Level Security
ALTER TABLE restaurants ENABLE ROW LEVEL SECURITY;
ALTER TABLE cuisines ENABLE ROW LEVEL SECURITY;
ALTER TABLE neighborhoods ENABLE ROW LEVEL SECURITY;

-- Public read access
CREATE POLICY "Public read restaurants" ON restaurants FOR SELECT USING (is_active = true);
CREATE POLICY "Public read cuisines" ON cuisines FOR SELECT USING (true);
CREATE POLICY "Public read neighborhoods" ON neighborhoods FOR SELECT USING (true);
```

- [ ] **Step 3: Create seed data from Merlijne's prototype**

Create `supabase/seed.sql`:
```sql
-- Neighborhoods
INSERT INTO neighborhoods (name, slug, lat, lng) VALUES
  ('Vondelpark', 'vondelpark', 52.3579, 4.8686),
  ('Centrum', 'centrum', 52.3731, 4.8922),
  ('Jordaan', 'jordaan', 52.3748, 4.8808),
  ('De Pijp', 'de-pijp', 52.3531, 4.8932),
  ('Noord', 'noord', 52.3906, 4.9222),
  ('Oost', 'oost', 52.3614, 4.9392),
  ('Zuid', 'zuid', 52.3468, 4.8780)
ON CONFLICT (slug) DO NOTHING;

-- Cuisines
INSERT INTO cuisines (name, slug) VALUES
  ('Internationaal', 'internationaal'),
  ('Japans', 'japans'),
  ('Frans-Europees', 'frans-europees'),
  ('Nederlands', 'nederlands'),
  ('Modern', 'modern'),
  ('Bistro', 'bistro'),
  ('Fine dining', 'fine-dining'),
  ('Frans', 'frans'),
  ('Eetcafé', 'eetcafe')
ON CONFLICT (slug) DO NOTHING;

-- Restaurants (matching Merlijne's 10 restaurants)
INSERT INTO restaurants (name, slug, neighborhood_id, cuisine_id, booking_type, formitable_id, website, phone, email) VALUES
  (
    'Trees Amsterdam', 'trees-amsterdam',
    (SELECT id FROM neighborhoods WHERE slug = 'vondelpark'),
    (SELECT id FROM cuisines WHERE slug = 'internationaal'),
    'link', NULL, 'https://www.treesamsterdam.nl', NULL, NULL
  ),
  (
    'Ken Sushi', 'ken-sushi',
    (SELECT id FROM neighborhoods WHERE slug = 'centrum'),
    (SELECT id FROM cuisines WHERE slug = 'japans'),
    'link', NULL, 'https://kensushi.nl', NULL, NULL
  ),
  (
    'Restaurant De Belhamel', 'de-belhamel',
    (SELECT id FROM neighborhoods WHERE slug = 'jordaan'),
    (SELECT id FROM cuisines WHERE slug = 'frans-europees'),
    'formitable', '86e443e6', 'https://www.belhamel.nl', NULL, NULL
  ),
  (
    'Café de Klepel', 'cafe-de-klepel',
    (SELECT id FROM neighborhoods WHERE slug = 'jordaan'),
    (SELECT id FROM cuisines WHERE slug = 'nederlands'),
    'formitable', NULL, 'https://www.cafedeklepel.nl', NULL, NULL
  ),
  (
    'Restaurant de Badcuyp', 'de-badcuyp',
    (SELECT id FROM neighborhoods WHERE slug = 'de-pijp'),
    (SELECT id FROM cuisines WHERE slug = 'modern'),
    'formitable', NULL, 'https://badcuypamsterdam.nl', NULL, NULL
  ),
  (
    'TROS', 'tros',
    (SELECT id FROM neighborhoods WHERE slug = 'noord'),
    (SELECT id FROM cuisines WHERE slug = 'internationaal'),
    'manual', NULL, 'https://www.detrosamsterdam.nl', '+31201234567', 'info@detrosamsterdam.nl'
  ),
  (
    'Troef Amsterdam', 'troef-amsterdam',
    (SELECT id FROM neighborhoods WHERE slug = 'oost'),
    (SELECT id FROM cuisines WHERE slug = 'bistro'),
    'link', NULL, 'https://troefamsterdam.nl', NULL, NULL
  ),
  (
    'Restaurant Franzen', 'restaurant-franzen',
    (SELECT id FROM neighborhoods WHERE slug = 'centrum'),
    (SELECT id FROM cuisines WHERE slug = 'fine-dining'),
    'link', NULL, 'https://www.franzen.amsterdam', NULL, NULL
  ),
  (
    'Pastis Amsterdam', 'pastis-amsterdam',
    (SELECT id FROM neighborhoods WHERE slug = 'jordaan'),
    (SELECT id FROM cuisines WHERE slug = 'frans'),
    'link', NULL, 'https://www.pastisamsterdam.nl', NULL, NULL
  ),
  (
    'Eetcafé Schotsheuvel', 'eetcafe-schotsheuvel',
    (SELECT id FROM neighborhoods WHERE slug = 'zuid'),
    (SELECT id FROM cuisines WHERE slug = 'eetcafe'),
    'manual', NULL, 'https://www.schotsheuvel.nl', '+31207654321', 'info@schotsheuvel.nl'
  )
ON CONFLICT (slug) DO NOTHING;
```

- [ ] **Step 4: Run migration and seed in Supabase**

Go to Supabase dashboard → SQL Editor:
1. Paste and run `001_create_restaurants.sql`
2. Paste and run `seed.sql`
3. Verify in Table Editor that 10 restaurants, 7 neighborhoods, and 9 cuisines exist

- [ ] **Step 5: Fill in .env.local with Supabase credentials**

Get from Supabase dashboard → Settings → API:
```
NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbG...
SUPABASE_SERVICE_ROLE_KEY=eyJhbG...
```

- [ ] **Step 6: Commit**

```bash
git add lib/supabase.ts supabase/
git commit -m "feat: add Supabase schema, seed data for 10 restaurants from prototype"
```

---

## Task 5: Header & Footer Components

**Files:**
- Create: `components/Header.tsx`
- Create: `components/Footer.tsx`
- Modify: `app/layout.tsx`

- [ ] **Step 1: Create Header component**

Create `components/Header.tsx`:
```tsx
export default function Header() {
  return (
    <header className="bg-ivory border-b border-border px-6 py-10 md:px-12 md:py-13">
      <h1 className="font-heading italic font-light text-charcoal text-4xl md:text-5xl leading-none tracking-tight">
        Table me,&nbsp;
      </h1>
      <p className="font-heading font-light text-xs md:text-sm tracking-[0.38em] uppercase text-mid mt-2.5">
        Amsterdam
      </p>
      <div className="w-12 h-px bg-silver mt-5" />
    </header>
  );
}
```

- [ ] **Step 2: Create Footer component**

Create `components/Footer.tsx`:
```tsx
export default function Footer() {
  return (
    <footer className="text-center py-6 font-body font-light text-[0.7rem] tracking-[0.12em] uppercase text-silver border-t border-border-lt">
      Table me, Amsterdam &middot; Live beschikbaarheid via Formitable
    </footer>
  );
}
```

- [ ] **Step 3: Add Header and Footer to layout**

Update `app/layout.tsx` — replace the `<body>` contents:
```tsx
import Header from "@/components/Header";
import Footer from "@/components/Footer";

// ... (keep existing imports and metadata)

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="nl" className={`${cormorant.variable} ${jost.variable}`}>
      <body className="min-h-screen flex flex-col">
        <Header />
        <main className="flex-1">{children}</main>
        <Footer />
      </body>
    </html>
  );
}
```

- [ ] **Step 4: Verify in browser**

```bash
npm run dev
```

Expected: Cream background, "Table me," in italic Cormorant Garamond, "AMSTERDAM" in small caps below, divider line, footer at bottom.

- [ ] **Step 5: Commit**

```bash
git add components/Header.tsx components/Footer.tsx app/layout.tsx
git commit -m "feat: add Header and Footer components with Merlijne's design"
```

---

## Task 6: SearchBar Component

**Files:**
- Create: `components/SearchBar.tsx`
- Create: `__tests__/components/SearchBar.test.tsx`

- [ ] **Step 1: Write failing test for SearchBar**

Create `__tests__/components/SearchBar.test.tsx`:
```tsx
import { render, screen, fireEvent } from "@testing-library/react";
import { describe, it, expect, vi } from "vitest";
import SearchBar from "@/components/SearchBar";

describe("SearchBar", () => {
  it("renders date picker and covers selector", () => {
    render(<SearchBar onSearch={vi.fn()} isLoading={false} />);
    expect(screen.getByLabelText("Datum")).toBeInTheDocument();
    expect(screen.getByLabelText("Personen")).toBeInTheDocument();
    expect(screen.getByRole("button", { name: /zoek een tafel/i })).toBeInTheDocument();
  });

  it("defaults date to today", () => {
    render(<SearchBar onSearch={vi.fn()} isLoading={false} />);
    const today = new Date().toISOString().split("T")[0];
    expect(screen.getByLabelText("Datum")).toHaveValue(today);
  });

  it("defaults covers to 2", () => {
    render(<SearchBar onSearch={vi.fn()} isLoading={false} />);
    expect(screen.getByLabelText("Personen")).toHaveValue("2");
  });

  it("calls onSearch with date and covers when submitted", () => {
    const onSearch = vi.fn();
    render(<SearchBar onSearch={onSearch} isLoading={false} />);
    fireEvent.click(screen.getByRole("button", { name: /zoek een tafel/i }));
    const today = new Date().toISOString().split("T")[0];
    expect(onSearch).toHaveBeenCalledWith(today, 2);
  });

  it("disables button when loading", () => {
    render(<SearchBar onSearch={vi.fn()} isLoading={true} />);
    expect(screen.getByRole("button")).toBeDisabled();
    expect(screen.getByRole("button")).toHaveTextContent("Zoeken…");
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

```bash
npm test -- __tests__/components/SearchBar.test.tsx
```

Expected: FAIL — module `@/components/SearchBar` not found.

- [ ] **Step 3: Implement SearchBar**

Create `components/SearchBar.tsx`:
```tsx
"use client";

import { useState } from "react";

interface SearchBarProps {
  onSearch: (date: string, covers: number) => void;
  isLoading: boolean;
}

function getToday(): string {
  const n = new Date();
  const y = n.getFullYear();
  const m = String(n.getMonth() + 1).padStart(2, "0");
  const d = String(n.getDate()).padStart(2, "0");
  return `${y}-${m}-${d}`;
}

export default function SearchBar({ onSearch, isLoading }: SearchBarProps) {
  const today = getToday();
  const [date, setDate] = useState(today);
  const [covers, setCovers] = useState(2);

  return (
    <div className="bg-ivory px-6 pb-10 md:px-12">
      <div className="flex items-end gap-10 flex-wrap">
        <div className="flex flex-col gap-2">
          <label
            htmlFor="datePicker"
            className="font-body font-normal text-[0.7rem] tracking-[0.2em] uppercase text-light"
          >
            Datum
          </label>
          <input
            type="date"
            id="datePicker"
            aria-label="Datum"
            value={date}
            min={today}
            onChange={(e) => setDate(e.target.value)}
            className="font-body font-light text-base text-charcoal bg-transparent border-0 border-b border-border py-1.5 outline-none min-w-[160px] focus:border-charcoal transition-colors"
          />
        </div>

        <div className="flex flex-col gap-2">
          <label
            htmlFor="coversPicker"
            className="font-body font-normal text-[0.7rem] tracking-[0.2em] uppercase text-light"
          >
            Personen
          </label>
          <select
            id="coversPicker"
            aria-label="Personen"
            value={String(covers)}
            onChange={(e) => setCovers(parseInt(e.target.value))}
            className="font-body font-light text-base text-charcoal bg-transparent border-0 border-b border-border py-1.5 outline-none min-w-[160px] focus:border-charcoal transition-colors appearance-none cursor-pointer"
          >
            <option value="1">1 persoon</option>
            <option value="2">2 personen</option>
            <option value="3">3 personen</option>
            <option value="4">4 personen</option>
            <option value="5">5 personen</option>
            <option value="6">6 personen</option>
            <option value="7">7 personen</option>
            <option value="8">8 personen</option>
          </select>
        </div>

        <button
          type="button"
          onClick={() => onSearch(date, covers)}
          disabled={isLoading}
          className="font-body font-normal text-[0.72rem] tracking-[0.2em] uppercase text-ivory bg-charcoal border border-charcoal px-7 py-3 cursor-pointer transition-colors hover:bg-accent hover:border-accent disabled:bg-silver disabled:border-silver disabled:cursor-not-allowed whitespace-nowrap"
        >
          {isLoading ? "Zoeken…" : "Zoek een tafel"}
        </button>
      </div>
    </div>
  );
}
```

- [ ] **Step 4: Run tests to verify they pass**

```bash
npm test -- __tests__/components/SearchBar.test.tsx
```

Expected: 5 tests PASS.

- [ ] **Step 5: Commit**

```bash
git add components/SearchBar.tsx __tests__/components/SearchBar.test.tsx
git commit -m "feat: add SearchBar component with date and covers picker"
```

---

## Task 7: Restaurants API Route

**Files:**
- Create: `app/api/restaurants/route.ts`
- Create: `__tests__/api/restaurants.test.ts`

- [ ] **Step 1: Write failing test for restaurants API**

Create `__tests__/api/restaurants.test.ts`:
```typescript
import { describe, it, expect, vi, beforeEach } from "vitest";

// Mock Supabase
const mockSelect = vi.fn();
const mockEq = vi.fn();
const mockFrom = vi.fn(() => ({
  select: mockSelect,
}));

vi.mock("@/lib/supabase", () => ({
  supabase: {
    from: mockFrom,
  },
}));

import { GET } from "@/app/api/restaurants/route";

describe("GET /api/restaurants", () => {
  beforeEach(() => {
    vi.clearAllMocks();
    mockSelect.mockReturnValue({
      eq: mockEq,
      then: vi.fn(),
      data: [],
      error: null,
    });
  });

  it("returns restaurants from Supabase", async () => {
    const mockRestaurants = [
      { id: "1", name: "Test Restaurant", slug: "test-restaurant" },
    ];
    mockSelect.mockResolvedValue({ data: mockRestaurants, error: null });

    const request = new Request("http://localhost:3000/api/restaurants");
    const response = await GET(request);
    const data = await response.json();

    expect(response.status).toBe(200);
    expect(data).toEqual(mockRestaurants);
    expect(mockFrom).toHaveBeenCalledWith("restaurants");
  });

  it("returns 500 on Supabase error", async () => {
    mockSelect.mockResolvedValue({
      data: null,
      error: { message: "Connection failed" },
    });

    const request = new Request("http://localhost:3000/api/restaurants");
    const response = await GET(request);

    expect(response.status).toBe(500);
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

```bash
npm test -- __tests__/api/restaurants.test.ts
```

Expected: FAIL — module `@/app/api/restaurants/route` not found.

- [ ] **Step 3: Implement restaurants API route**

Create `app/api/restaurants/route.ts`:
```typescript
import { NextResponse } from "next/server";
import { supabase } from "@/lib/supabase";

export async function GET() {
  const { data, error } = await supabase
    .from("restaurants")
    .select(`
      *,
      neighborhood:neighborhoods(name, slug),
      cuisine:cuisines(name, slug)
    `)
    .eq("is_active", true)
    .order("name");

  if (error) {
    return NextResponse.json(
      { error: "Failed to fetch restaurants" },
      { status: 500 }
    );
  }

  return NextResponse.json(data);
}
```

- [ ] **Step 4: Run tests to verify they pass**

```bash
npm test -- __tests__/api/restaurants.test.ts
```

Expected: PASS.

- [ ] **Step 5: Commit**

```bash
git add app/api/restaurants/route.ts __tests__/api/restaurants.test.ts
git commit -m "feat: add restaurants API route with Supabase query"
```

---

## Task 8: Formitable Availability

**Files:**
- Create: `lib/formitable.ts`
- Create: `app/api/availability/route.ts`
- Create: `__tests__/api/availability.test.ts`

- [ ] **Step 1: Write failing test for availability API**

Create `__tests__/api/availability.test.ts`:
```typescript
import { describe, it, expect, vi } from "vitest";

vi.mock("@/lib/formitable", () => ({
  fetchFormitableAvailability: vi.fn(),
}));

import { GET } from "@/app/api/availability/route";
import { fetchFormitableAvailability } from "@/lib/formitable";

describe("GET /api/availability", () => {
  it("returns slots for a valid formitable_id and date", async () => {
    const mockSlots = {
      status: "available" as const,
      slots: [
        { time: "18:00", fewLeft: false },
        { time: "19:30", fewLeft: true },
      ],
    };
    vi.mocked(fetchFormitableAvailability).mockResolvedValue(mockSlots);

    const request = new Request(
      "http://localhost:3000/api/availability?formitable_id=86e443e6&date=2026-04-10&covers=2"
    );
    const response = await GET(request);
    const data = await response.json();

    expect(response.status).toBe(200);
    expect(data.status).toBe("available");
    expect(data.slots).toHaveLength(2);
  });

  it("returns 400 if formitable_id is missing", async () => {
    const request = new Request(
      "http://localhost:3000/api/availability?date=2026-04-10&covers=2"
    );
    const response = await GET(request);

    expect(response.status).toBe(400);
  });

  it("returns 400 if date is missing", async () => {
    const request = new Request(
      "http://localhost:3000/api/availability?formitable_id=86e443e6&covers=2"
    );
    const response = await GET(request);

    expect(response.status).toBe(400);
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

```bash
npm test -- __tests__/api/availability.test.ts
```

Expected: FAIL — modules not found.

- [ ] **Step 3: Implement Formitable availability fetcher**

Create `lib/formitable.ts`:
```typescript
import type { Availability, TimeSlot } from "./types";

const FORMITABLE_BASE = "https://formitable.com/api/widget";

export async function fetchFormitableAvailability(
  widgetId: string,
  date: string,
  covers: number
): Promise<Availability> {
  try {
    const url = `${FORMITABLE_BASE}/${widgetId}/availability?date=${date}&covers=${covers}`;
    const response = await fetch(url, {
      headers: {
        Accept: "application/json",
      },
      next: { revalidate: 300 }, // Cache for 5 minutes
    });

    if (!response.ok) {
      return { status: "full", slots: [] };
    }

    const data = await response.json();

    // Parse Formitable response into our format
    // Note: The exact response format depends on Formitable's API.
    // This may need adjustment once we test against the real API.
    const slots: TimeSlot[] = Array.isArray(data.timeslots)
      ? data.timeslots.map((slot: { time: string; available: number }) => ({
          time: slot.time,
          fewLeft: slot.available <= 2,
        }))
      : [];

    return {
      status: slots.length > 0 ? "available" : "full",
      slots,
    };
  } catch {
    return { status: "full", slots: [] };
  }
}
```

- [ ] **Step 4: Implement availability API route**

Create `app/api/availability/route.ts`:
```typescript
import { NextResponse } from "next/server";
import { fetchFormitableAvailability } from "@/lib/formitable";

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const formitableId = searchParams.get("formitable_id");
  const date = searchParams.get("date");
  const covers = parseInt(searchParams.get("covers") || "2");

  if (!formitableId) {
    return NextResponse.json(
      { error: "formitable_id is required" },
      { status: 400 }
    );
  }

  if (!date) {
    return NextResponse.json(
      { error: "date is required" },
      { status: 400 }
    );
  }

  const availability = await fetchFormitableAvailability(
    formitableId,
    date,
    covers
  );

  return NextResponse.json(availability);
}
```

- [ ] **Step 5: Run tests to verify they pass**

```bash
npm test -- __tests__/api/availability.test.ts
```

Expected: PASS.

- [ ] **Step 6: Commit**

```bash
git add lib/formitable.ts app/api/availability/route.ts __tests__/api/availability.test.ts
git commit -m "feat: add Formitable availability API with 5-min cache"
```

---

## Task 9: StatusLine Component

**Files:**
- Create: `components/StatusLine.tsx`

- [ ] **Step 1: Create StatusLine component**

Create `components/StatusLine.tsx`:
```tsx
interface StatusLineProps {
  status: "idle" | "loading" | "done";
  text: string;
}

export default function StatusLine({ status, text }: StatusLineProps) {
  if (status === "idle") return null;

  return (
    <div className="flex items-center gap-2.5 mb-9 pb-5 border-b border-border-lt">
      <div
        className={`w-[5px] h-[5px] rounded-full shrink-0 ${
          status === "loading"
            ? "bg-accent animate-blink"
            : "bg-available"
        }`}
      />
      <span className="font-body font-light text-[0.78rem] tracking-[0.05em] text-mid">
        {text}
      </span>
    </div>
  );
}
```

- [ ] **Step 2: Commit**

```bash
git add components/StatusLine.tsx
git commit -m "feat: add StatusLine loading/done indicator component"
```

---

## Task 10: SlotButton Component

**Files:**
- Create: `components/SlotButton.tsx`

- [ ] **Step 1: Create SlotButton component**

Create `components/SlotButton.tsx`:
```tsx
interface SlotButtonProps {
  time: string;
  fewLeft: boolean;
  onClick: () => void;
}

export default function SlotButton({ time, fewLeft, onClick }: SlotButtonProps) {
  return (
    <button
      type="button"
      onClick={(e) => {
        e.stopPropagation();
        onClick();
      }}
      className={`font-body font-normal text-[0.78rem] tracking-[0.06em] border px-3 py-1.5 cursor-pointer transition-colors ${
        fewLeft
          ? "border-accent text-accent hover:bg-accent hover:text-ivory"
          : "border-border text-charcoal hover:bg-charcoal hover:text-ivory"
      }`}
    >
      {time}
    </button>
  );
}
```

- [ ] **Step 2: Commit**

```bash
git add components/SlotButton.tsx
git commit -m "feat: add SlotButton component with few-left state"
```

---

## Task 11: RestaurantCard Component

**Files:**
- Create: `components/RestaurantCard.tsx`
- Create: `__tests__/components/RestaurantCard.test.tsx`

- [ ] **Step 1: Write failing test**

Create `__tests__/components/RestaurantCard.test.tsx`:
```tsx
import { render, screen } from "@testing-library/react";
import { describe, it, expect, vi } from "vitest";
import RestaurantCard from "@/components/RestaurantCard";
import type { RestaurantWithAvailability } from "@/lib/types";

const baseRestaurant: RestaurantWithAvailability = {
  id: "1",
  name: "Test Restaurant",
  slug: "test-restaurant",
  neighborhood_id: null,
  cuisine_id: null,
  price_range: 2,
  address: null,
  lat: null,
  lng: null,
  phone: null,
  email: null,
  website: "https://test.nl",
  google_place_id: null,
  google_rating: null,
  photo_url: null,
  photo_override: null,
  booking_type: "formitable",
  formitable_id: "abc123",
  booking_url: null,
  is_curated: false,
  curated_note: null,
  is_active: true,
  neighborhood: { name: "Jordaan", slug: "jordaan" },
  cuisine: { name: "Frans", slug: "frans" },
  availability: {
    status: "available",
    slots: [
      { time: "18:00", fewLeft: false },
      { time: "19:30", fewLeft: true },
    ],
  },
};

describe("RestaurantCard", () => {
  it("renders restaurant name and meta", () => {
    render(<RestaurantCard restaurant={baseRestaurant} onSelect={vi.fn()} />);
    expect(screen.getByText("Test Restaurant")).toBeInTheDocument();
    expect(screen.getByText(/Jordaan/)).toBeInTheDocument();
    expect(screen.getByText(/Frans/)).toBeInTheDocument();
  });

  it("renders available time slots", () => {
    render(<RestaurantCard restaurant={baseRestaurant} onSelect={vi.fn()} />);
    expect(screen.getByText("18:00")).toBeInTheDocument();
    expect(screen.getByText("19:30")).toBeInTheDocument();
  });

  it("shows slot count when available", () => {
    render(<RestaurantCard restaurant={baseRestaurant} onSelect={vi.fn()} />);
    expect(screen.getByText("2 tijdslots")).toBeInTheDocument();
  });

  it("shows volzet when full", () => {
    const fullRestaurant = {
      ...baseRestaurant,
      availability: { status: "full" as const, slots: [] },
    };
    render(<RestaurantCard restaurant={fullRestaurant} onSelect={vi.fn()} />);
    expect(screen.getByText("Volzet")).toBeInTheDocument();
  });

  it("shows bel/mail for manual restaurants", () => {
    const manualRestaurant = {
      ...baseRestaurant,
      booking_type: "manual" as const,
      phone: "+31201234567",
      email: "info@test.nl",
      availability: { status: "manual" as const, slots: [] },
    };
    render(<RestaurantCard restaurant={manualRestaurant} onSelect={vi.fn()} />);
    expect(screen.getByText("Bel / mail")).toBeInTheDocument();
  });

  it("shows loading skeleton when availability is null", () => {
    const loadingRestaurant = { ...baseRestaurant, availability: null };
    render(<RestaurantCard restaurant={loadingRestaurant} onSelect={vi.fn()} />);
    expect(screen.getByText("Laden")).toBeInTheDocument();
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

```bash
npm test -- __tests__/components/RestaurantCard.test.tsx
```

Expected: FAIL.

- [ ] **Step 3: Implement RestaurantCard**

Create `components/RestaurantCard.tsx`:
```tsx
"use client";

import type { RestaurantWithAvailability } from "@/lib/types";
import SlotButton from "./SlotButton";

interface RestaurantCardProps {
  restaurant: RestaurantWithAvailability;
  onSelect: (restaurantId: string) => void;
}

export default function RestaurantCard({
  restaurant: r,
  onSelect,
}: RestaurantCardProps) {
  const avail = r.availability;
  const photoSrc = r.photo_override || r.photo_url;
  const meta = [r.neighborhood?.name, r.cuisine?.name]
    .filter(Boolean)
    .join(" · ");

  let statusEl: React.ReactNode = null;
  if (!avail) {
    statusEl = (
      <span className="font-body font-light text-[0.7rem] tracking-[0.15em] uppercase text-silver animate-blink">
        Laden
      </span>
    );
  } else if (r.booking_type === "manual") {
    statusEl = (
      <span className="font-body font-light text-[0.7rem] tracking-[0.15em] uppercase text-light">
        Bel / mail
      </span>
    );
  } else if (avail.status === "full") {
    statusEl = (
      <span className="font-body font-light text-[0.7rem] tracking-[0.15em] uppercase text-full">
        Volzet
      </span>
    );
  } else if (avail.status === "available") {
    const count = avail.slots.length;
    statusEl = (
      <span className="font-body font-light text-[0.7rem] tracking-[0.15em] uppercase text-available">
        {count} tijdslot{count !== 1 ? "s" : ""}
      </span>
    );
  }

  let bottomEl: React.ReactNode = null;
  if (!avail) {
    bottomEl = (
      <div className="flex gap-1.5">
        {[1, 2, 3].map((i) => (
          <div key={i} className="h-[30px] w-[58px] animate-shimmer rounded-sm" />
        ))}
      </div>
    );
  } else if (r.booking_type === "manual") {
    bottomEl = (
      <div className="flex items-center gap-4 flex-wrap">
        <span className="font-body font-light text-[0.75rem] tracking-[0.05em] text-light">
          Geen online reservering
        </span>
        {r.phone && (
          <a
            href={`tel:${r.phone}`}
            onClick={(e) => e.stopPropagation()}
            className="font-body font-normal text-[0.7rem] tracking-[0.15em] uppercase text-mid no-underline border-b border-border pb-px hover:text-charcoal hover:border-charcoal transition-colors"
          >
            Bellen
          </a>
        )}
        {r.email && (
          <a
            href={`mailto:${r.email}`}
            onClick={(e) => e.stopPropagation()}
            className="font-body font-normal text-[0.7rem] tracking-[0.15em] uppercase text-mid no-underline border-b border-border pb-px hover:text-charcoal hover:border-charcoal transition-colors"
          >
            Mailen
          </a>
        )}
      </div>
    );
  } else if (avail.status === "full") {
    bottomEl = (
      <p className="font-heading italic font-light text-[0.95rem] text-silver">
        Geen beschikbaarheid op deze datum
      </p>
    );
  } else if (avail.status === "available") {
    bottomEl = (
      <div className="flex flex-wrap gap-1.5">
        {avail.slots.map((slot) => (
          <SlotButton
            key={slot.time}
            time={slot.time}
            fewLeft={slot.fewLeft}
            onClick={() => onSelect(r.id)}
          />
        ))}
      </div>
    );
  }

  return (
    <div
      className="flex border-b border-border-lt cursor-pointer transition-colors hover:bg-card first:border-t"
      onClick={() => onSelect(r.id)}
    >
      <div
        className={`w-[140px] shrink-0 overflow-hidden bg-cream max-md:w-[90px] ${
          !photoSrc ? "border-r border-border-lt min-h-[140px]" : ""
        }`}
      >
        {!avail ? (
          <div className="w-full h-full min-h-[140px] animate-shimmer" />
        ) : photoSrc ? (
          <img
            src={photoSrc}
            alt={r.name}
            className="w-full h-full object-cover grayscale contrast-[1.05] opacity-[0.88] hover:opacity-[0.96] transition-opacity"
          />
        ) : null}
      </div>

      <div className="flex-1 p-6 flex flex-col justify-between min-h-[140px] max-md:p-4">
        <div className="flex justify-between items-start gap-4">
          <div>
            <div className="font-heading font-normal text-xl tracking-[0.01em] leading-tight text-charcoal">
              {r.name}
            </div>
            <div className="font-body font-light text-[0.72rem] tracking-[0.12em] uppercase text-light mt-1">
              {meta}
            </div>
          </div>
          {statusEl}
        </div>
        <div className="mt-4">{bottomEl}</div>
      </div>
    </div>
  );
}
```

- [ ] **Step 4: Run tests to verify they pass**

```bash
npm test -- __tests__/components/RestaurantCard.test.tsx
```

Expected: 6 tests PASS.

- [ ] **Step 5: Commit**

```bash
git add components/RestaurantCard.tsx __tests__/components/RestaurantCard.test.tsx
git commit -m "feat: add RestaurantCard with availability states (slots, full, manual, loading)"
```

---

## Task 12: BookingModal Component

**Files:**
- Create: `components/BookingModal.tsx`
- Create: `__tests__/components/BookingModal.test.tsx`

- [ ] **Step 1: Write failing test**

Create `__tests__/components/BookingModal.test.tsx`:
```tsx
import { render, screen, fireEvent } from "@testing-library/react";
import { describe, it, expect, vi } from "vitest";
import BookingModal from "@/components/BookingModal";
import type { Restaurant } from "@/lib/types";

const formitableRestaurant: Restaurant = {
  id: "1",
  name: "Test Formitable",
  slug: "test-formitable",
  neighborhood_id: null,
  cuisine_id: null,
  price_range: null,
  address: null,
  lat: null,
  lng: null,
  phone: null,
  email: null,
  website: "https://test.nl",
  google_place_id: null,
  google_rating: null,
  photo_url: null,
  photo_override: null,
  booking_type: "formitable",
  formitable_id: "abc123",
  booking_url: null,
  is_curated: false,
  curated_note: null,
  is_active: true,
};

describe("BookingModal", () => {
  it("renders nothing when no restaurant is selected", () => {
    const { container } = render(
      <BookingModal restaurant={null} onClose={vi.fn()} />
    );
    expect(container.innerHTML).toBe("");
  });

  it("renders restaurant name in header", () => {
    render(
      <BookingModal restaurant={formitableRestaurant} onClose={vi.fn()} />
    );
    expect(screen.getByText("Test Formitable")).toBeInTheDocument();
  });

  it("renders Formitable iframe for formitable restaurants", () => {
    render(
      <BookingModal restaurant={formitableRestaurant} onClose={vi.fn()} />
    );
    const iframe = document.querySelector("iframe");
    expect(iframe).toBeInTheDocument();
    expect(iframe?.src).toContain("abc123");
  });

  it("renders external link for link-type restaurants", () => {
    const linkRestaurant = {
      ...formitableRestaurant,
      booking_type: "link" as const,
      formitable_id: null,
    };
    render(<BookingModal restaurant={linkRestaurant} onClose={vi.fn()} />);
    expect(
      screen.getByText(/reserveer via website/i)
    ).toBeInTheDocument();
  });

  it("renders phone and email for manual restaurants", () => {
    const manualRestaurant = {
      ...formitableRestaurant,
      booking_type: "manual" as const,
      formitable_id: null,
      phone: "+31201234567",
      email: "info@test.nl",
    };
    render(<BookingModal restaurant={manualRestaurant} onClose={vi.fn()} />);
    expect(screen.getByText("Bellen")).toBeInTheDocument();
    expect(screen.getByText("Mailen")).toBeInTheDocument();
  });

  it("calls onClose when close button is clicked", () => {
    const onClose = vi.fn();
    render(
      <BookingModal restaurant={formitableRestaurant} onClose={onClose} />
    );
    fireEvent.click(screen.getByRole("button", { name: /sluiten/i }));
    expect(onClose).toHaveBeenCalled();
  });

  it("calls onClose when overlay is clicked", () => {
    const onClose = vi.fn();
    render(
      <BookingModal restaurant={formitableRestaurant} onClose={onClose} />
    );
    fireEvent.click(screen.getByTestId("modal-overlay"));
    expect(onClose).toHaveBeenCalled();
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

```bash
npm test -- __tests__/components/BookingModal.test.tsx
```

Expected: FAIL.

- [ ] **Step 3: Implement BookingModal**

Create `components/BookingModal.tsx`:
```tsx
"use client";

import { useEffect } from "react";
import type { Restaurant } from "@/lib/types";

interface BookingModalProps {
  restaurant: Restaurant | null;
  onClose: () => void;
}

export default function BookingModal({
  restaurant,
  onClose,
}: BookingModalProps) {
  useEffect(() => {
    if (!restaurant) return;

    document.body.style.overflow = "hidden";
    const handleEsc = (e: KeyboardEvent) => {
      if (e.key === "Escape") onClose();
    };
    document.addEventListener("keydown", handleEsc);

    return () => {
      document.body.style.overflow = "";
      document.removeEventListener("keydown", handleEsc);
    };
  }, [restaurant, onClose]);

  if (!restaurant) return null;

  let bodyContent: React.ReactNode;

  if (restaurant.booking_type === "formitable" && restaurant.formitable_id) {
    const widgetUrl = `https://widget.formitable.com/formitable/nl/${restaurant.formitable_id}/when`;
    bodyContent = (
      <>
        <div className="flex-1 relative min-h-[500px]">
          <iframe
            src={widgetUrl}
            title={`Reserveer ${restaurant.name}`}
            className="absolute inset-0 w-full h-full border-0"
          />
        </div>
        <div className="px-6 py-3.5 border-t border-border-lt flex items-center justify-between shrink-0">
          {restaurant.website && (
            <a
              href={restaurant.website}
              target="_blank"
              rel="noopener noreferrer"
              className="font-body font-light text-[0.72rem] tracking-[0.1em] uppercase text-light hover:text-charcoal transition-colors"
            >
              Website ↗
            </a>
          )}
          <span className="font-body text-[0.68rem] tracking-[0.08em] uppercase text-light">
            Via Formitable
          </span>
        </div>
      </>
    );
  } else if (restaurant.booking_type === "manual") {
    bodyContent = (
      <div className="px-6 py-10 text-center">
        <p className="font-heading italic text-base text-mid mb-5">
          Dit restaurant heeft geen online reserveringen.
          <br />
          Bel of mail voor een tafel.
        </p>
        <div className="flex gap-3 justify-center">
          {restaurant.phone && (
            <a
              href={`tel:${restaurant.phone}`}
              className="inline-block font-body font-normal text-[0.72rem] tracking-[0.2em] uppercase text-ivory bg-charcoal border border-charcoal px-6 py-2.5 hover:bg-accent hover:border-accent transition-colors"
            >
              Bellen
            </a>
          )}
          {restaurant.email && (
            <a
              href={`mailto:${restaurant.email}`}
              className="inline-block font-body font-normal text-[0.72rem] tracking-[0.2em] uppercase text-charcoal bg-transparent border border-charcoal px-6 py-2.5 hover:bg-charcoal hover:text-ivory transition-colors"
            >
              Mailen
            </a>
          )}
        </div>
      </div>
    );
  } else {
    bodyContent = (
      <div className="px-6 py-10 text-center">
        <p className="font-heading italic text-base text-mid mb-5">
          Reserveer direct via de website van {restaurant.name}.
        </p>
        {restaurant.website && (
          <a
            href={restaurant.website}
            target="_blank"
            rel="noopener noreferrer"
            className="inline-block font-body font-normal text-[0.72rem] tracking-[0.2em] uppercase text-ivory bg-charcoal border border-charcoal px-6 py-2.5 hover:bg-accent hover:border-accent transition-colors"
          >
            Reserveer via website ↗
          </a>
        )}
      </div>
    );
  }

  return (
    <div
      data-testid="modal-overlay"
      className="fixed inset-0 bg-charcoal/55 backdrop-blur-sm z-50 flex items-center justify-center p-6 max-md:items-end max-md:p-0 animate-[fadeIn_0.2s_ease]"
      onClick={onClose}
    >
      <div
        className="bg-ivory w-full max-w-[420px] max-h-[90vh] flex flex-col overflow-hidden max-md:max-w-full max-md:max-h-[92vh]"
        onClick={(e) => e.stopPropagation()}
      >
        <div className="flex items-center justify-between px-6 py-5 border-b border-border-lt shrink-0">
          <span className="font-heading font-normal text-lg tracking-[0.01em]">
            {restaurant.name}
          </span>
          <button
            onClick={onClose}
            aria-label="Sluiten"
            className="bg-transparent border-0 cursor-pointer text-xl text-light hover:text-charcoal transition-colors px-2 py-1 leading-none"
          >
            &#x2715;
          </button>
        </div>
        <div className="flex-1 overflow-hidden flex flex-col">
          {bodyContent}
        </div>
      </div>
    </div>
  );
}
```

- [ ] **Step 4: Run tests to verify they pass**

```bash
npm test -- __tests__/components/BookingModal.test.tsx
```

Expected: 7 tests PASS.

- [ ] **Step 5: Commit**

```bash
git add components/BookingModal.tsx __tests__/components/BookingModal.test.tsx
git commit -m "feat: add BookingModal with Formitable widget, link, and manual modes"
```

---

## Task 13: RestaurantList Component

**Files:**
- Create: `components/RestaurantList.tsx`

- [ ] **Step 1: Create RestaurantList**

Create `components/RestaurantList.tsx`:
```tsx
"use client";

import type { RestaurantWithAvailability } from "@/lib/types";
import RestaurantCard from "./RestaurantCard";

interface RestaurantListProps {
  restaurants: RestaurantWithAvailability[];
  onSelectRestaurant: (restaurantId: string) => void;
}

export default function RestaurantList({
  restaurants,
  onSelectRestaurant,
}: RestaurantListProps) {
  if (restaurants.length === 0) {
    return (
      <div className="py-16 pb-10">
        <p className="font-heading italic font-light text-xl text-silver">
          Kies een datum en wij zoeken waar nog een tafel vrij is.
        </p>
      </div>
    );
  }

  return (
    <div className="flex flex-col">
      {restaurants.map((restaurant) => (
        <RestaurantCard
          key={restaurant.id}
          restaurant={restaurant}
          onSelect={onSelectRestaurant}
        />
      ))}
    </div>
  );
}
```

- [ ] **Step 2: Commit**

```bash
git add components/RestaurantList.tsx
git commit -m "feat: add RestaurantList component"
```

---

## Task 14: Homepage Assembly

**Files:**
- Modify: `app/page.tsx`

- [ ] **Step 1: Implement homepage with search flow**

Replace `app/page.tsx`:
```tsx
"use client";

import { useState, useCallback } from "react";
import SearchBar from "@/components/SearchBar";
import RestaurantList from "@/components/RestaurantList";
import BookingModal from "@/components/BookingModal";
import StatusLine from "@/components/StatusLine";
import type {
  Restaurant,
  RestaurantWithAvailability,
  Availability,
} from "@/lib/types";

export default function Home() {
  const [restaurants, setRestaurants] = useState<RestaurantWithAvailability[]>(
    []
  );
  const [isLoading, setIsLoading] = useState(false);
  const [statusText, setStatusText] = useState("");
  const [statusState, setStatusState] = useState<"idle" | "loading" | "done">(
    "idle"
  );
  const [selectedRestaurant, setSelectedRestaurant] =
    useState<Restaurant | null>(null);
  const [hasSearched, setHasSearched] = useState(false);

  const handleSearch = useCallback(async (date: string, covers: number) => {
    setIsLoading(true);
    setStatusState("loading");
    setStatusText("Beschikbaarheid ophalen…");
    setHasSearched(true);

    // Fetch restaurant list
    const res = await fetch("/api/restaurants");
    const data: Restaurant[] = await res.json();

    // Initialize all restaurants with null availability (loading state)
    const withLoading: RestaurantWithAvailability[] = data.map((r) => ({
      ...r,
      availability: r.booking_type === "manual"
        ? { status: "manual" as const, slots: [] }
        : null,
    }));
    setRestaurants(withLoading);

    // Fetch availability for each restaurant progressively
    for (const r of data) {
      if (r.booking_type === "manual") continue;

      let availability: Availability;

      if (r.booking_type === "formitable" && r.formitable_id) {
        const availRes = await fetch(
          `/api/availability?formitable_id=${r.formitable_id}&date=${date}&covers=${covers}`
        );
        availability = await availRes.json();
      } else {
        // link-type: no availability data, just show as available for booking
        availability = { status: "available", slots: [] };
      }

      setRestaurants((prev) =>
        prev.map((item) =>
          item.id === r.id ? { ...item, availability } : item
        )
      );

      // Small delay for progressive loading effect (like Merlijne's prototype)
      await new Promise((resolve) =>
        setTimeout(resolve, 100 + Math.random() * 200)
      );
    }

    // Format done status
    const label = new Date(date + "T12:00:00").toLocaleDateString("nl-NL", {
      weekday: "long",
      day: "numeric",
      month: "long",
    });
    setStatusState("done");
    setStatusText(
      `${label}  ·  ${covers} ${covers === 1 ? "persoon" : "personen"}  ·  Klik een restaurant om te reserveren`
    );
    setIsLoading(false);
  }, []);

  const handleSelectRestaurant = useCallback(
    (restaurantId: string) => {
      const r = restaurants.find((x) => x.id === restaurantId);
      if (r) setSelectedRestaurant(r);
    },
    [restaurants]
  );

  return (
    <>
      <SearchBar onSearch={handleSearch} isLoading={isLoading} />

      <div className="max-w-[900px] mx-auto px-8 py-13 pb-20 max-md:px-5 max-md:py-10 max-md:pb-15">
        <StatusLine status={statusState} text={statusText} />

        {!hasSearched ? (
          <div className="py-16 pb-10">
            <p className="font-heading italic font-light text-xl text-silver">
              Kies een datum en wij zoeken waar nog een tafel vrij is.
            </p>
          </div>
        ) : (
          <RestaurantList
            restaurants={restaurants}
            onSelectRestaurant={handleSelectRestaurant}
          />
        )}
      </div>

      <BookingModal
        restaurant={selectedRestaurant}
        onClose={() => setSelectedRestaurant(null)}
      />
    </>
  );
}
```

- [ ] **Step 2: Verify in browser**

```bash
npm run dev
```

Open http://localhost:3000:
1. Verify header, search bar, empty state text, footer all render
2. Click "Zoek een tafel" — should fetch restaurants from Supabase
3. Cards should appear with progressive loading
4. Click a restaurant — modal should open

- [ ] **Step 3: Commit**

```bash
git add app/page.tsx
git commit -m "feat: assemble homepage with search flow, progressive loading, and booking modal"
```

---

## Task 15: Run All Tests & Verify Build

**Files:** None (verification only)

- [ ] **Step 1: Run full test suite**

```bash
npm test
```

Expected: All tests pass.

- [ ] **Step 2: Run production build**

```bash
npm run build
```

Expected: Build succeeds with no errors.

- [ ] **Step 3: Test production build locally**

```bash
npm start
```

Open http://localhost:3000 — verify everything works as in dev mode.

- [ ] **Step 4: Commit any fixes if needed**

---

## Task 16: Deploy to Vercel

**Files:**
- Delete or rename: `vercel.json` (Next.js doesn't need SPA rewrites)

- [ ] **Step 1: Remove old vercel.json**

The SPA rewrite rule was for the static `index.html`. Next.js handles routing itself. Delete `vercel.json`:
```bash
rm vercel.json
```

- [ ] **Step 2: Add Supabase env vars to Vercel**

In Vercel dashboard → Settings → Environment Variables, add:
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `SUPABASE_SERVICE_ROLE_KEY`

- [ ] **Step 3: Deploy**

```bash
npx vercel --prod
```

Or push to main and let Vercel auto-deploy.

- [ ] **Step 4: Verify production deployment**

Open the Vercel URL and test:
1. Homepage loads with header, search bar, footer
2. Search returns restaurants from Supabase
3. Formitable availability loads (may need real widget IDs)
4. Booking modal opens correctly
5. Mobile responsive layout works

- [ ] **Step 5: Final commit**

```bash
git add -A
git commit -m "feat: Fase 1 complete — Next.js + Supabase + Formitable, deployed to Vercel"
```

---

## Post-Fase 1 Notes

**What's ready:**
- Working Next.js app with real Supabase data
- 10 restaurants from Merlijne's prototype
- Formitable availability (for restaurants with widget IDs)
- Booking modal with three modes
- Progressive loading UX
- Mobile-responsive design
- Deployed on Vercel

**What's next (Fase 2):**
- Google Places bulk import to add hundreds of restaurants
- FilterBar (neighborhood, cuisine, price)
- Photo system (Google Places + override)
- Restaurant detail pages (/[slug])

**Known limitations to address:**
- Formitable API response format needs validation against real API (lib/formitable.ts parsing may need adjustment)
- Not all 10 seed restaurants have real Formitable widget IDs — need to look these up
- Photos are not yet loaded (no Google Places integration yet)
