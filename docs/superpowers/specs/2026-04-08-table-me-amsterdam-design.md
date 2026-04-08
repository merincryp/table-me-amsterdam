# Table me, Amsterdam — Design Spec

**Datum:** 2026-04-08
**Auteurs:** Lennard & Merlijne
**Status:** Goedgekeurd

---

## 1. Visie

Table me, Amsterdam is een **restaurant-aggregator met curated laag** voor Amsterdam. Het platform toont live beschikbaarheid van zoveel mogelijk Amsterdamse restaurants, met daaroverheen een persoonlijke selectie ("Table me picks") van door Lennard & Merlijne aanbevolen restaurants.

**Core value proposition:** Eén plek om te zien waar vanavond nog een tafel vrij is in Amsterdam.

---

## 2. Tech Stack

| Component          | Keuze                                      |
|--------------------|---------------------------------------------|
| Framework          | Next.js 14 (App Router)                     |
| Styling            | Tailwind CSS + Merlijne's design tokens     |
| Database           | Supabase (Postgres + Storage)               |
| Beschikbaarheid    | Formitable (widget embed + API/scraping)    |
| Restaurant data    | Google Places API → Supabase                |
| Foto's             | Google Places (default) + Supabase Storage (override) |
| Kaart              | Mapbox GL JS (gratis tier, 50k loads/maand) |
| Hosting            | Vercel (gratis tier)                        |
| Caching            | Vercel KV (5 min TTL op beschikbaarheid)    |

---

## 3. Database Schema

### restaurants (kern tabel)

```
id                uuid (PK)
name              text NOT NULL
slug              text UNIQUE
neighborhood_id   uuid (FK → neighborhoods.id)
cuisine_id        uuid (FK → cuisines.id)
price_range       int (1=€, 2=€€, 3=€€€)
address           text
lat               float8
lng               float8
phone             text
email             text
website           text
google_place_id   text
google_rating     float4
photo_url         text (Google Places default)
photo_override    text (Supabase Storage URL)
booking_type      enum (formitable, link, manual)
formitable_id     text (widget UID)
booking_url       text (fallback externe link)
is_curated        boolean DEFAULT false
curated_note      text ("Beste pasta in de Pijp")
is_active         boolean DEFAULT true
created_at        timestamptz
updated_at        timestamptz
```

### cuisines (lookup)

```
id    uuid (PK)
name  text UNIQUE (Frans, Japans, ...)
slug  text UNIQUE
```

### neighborhoods (lookup)

```
id    uuid (PK)
name  text UNIQUE (Jordaan, De Pijp, ...)
slug  text UNIQUE
lat   float8 (center point)
lng   float8
```

### curated_collections (fase 4)

```
id              uuid (PK)
title           text ("Beste date spots")
description     text
restaurant_ids  uuid[]
is_featured     boolean
```

---

## 4. Pagina's & Route Structuur

```
app/
├── layout.tsx              // Fonts, metadata, header/footer
├── page.tsx                // Homepage: zoek + resultaten
├── kaart/page.tsx          // Mapbox kaartweergave
├── [slug]/page.tsx         // Restaurant detail (SEO)
│
├── api/
│   ├── availability/route.ts  // Formitable beschikbaarheid proxy
│   ├── restaurants/route.ts   // Restaurant lijst + filters
│   └── import/route.ts        // Google Places import (admin)
```

### Componenten

```
components/
├── SearchBar.tsx           // Datum + personen + zoekknop
├── RestaurantCard.tsx      // Restaurant kaart met slots
├── RestaurantList.tsx      // Lijst van kaarten
├── BookingModal.tsx        // Formitable widget / link / bel
├── MapView.tsx             // Mapbox kaart met pins
├── FilterBar.tsx           // Buurt, keuken, prijs filters
├── CuratedBadge.tsx        // "Table me pick" badge
├── TonightButton.tsx       // "Vanavond vrij" quick action
├── FavoriteButton.tsx      // Hart-icoon, localStorage
├── SlotButton.tsx          // Individueel tijdslot
├── Header.tsx              // Logo + navigatie (Lijst / Kaart)
└── Footer.tsx              // Credits + links
```

---

## 5. Data Flows

### Flow 1: Gebruiker zoekt beschikbaarheid

1. Gebruiker kiest datum + personen (+ optionele filters)
2. Frontend fetcht `GET /api/restaurants?date=X&covers=Y&neighborhood=Z&cuisine=W`
3. API route query't Supabase met filters
4. Per restaurant met `booking_type=formitable`: fetch Formitable beschikbaarheid
5. Beschikbaarheid wordt 5 minuten gecached in Vercel KV
6. Response: array van restaurants met beschikbare tijdslots

### Flow 2: Restaurant import (eenmalig/periodiek)

1. Admin triggert `POST /api/import` (beveiligd)
2. Google Places API: zoek "restaurants in Amsterdam" (paginated)
3. Per resultaat: extraheer naam, adres, lat/lng, foto, rating, type keuken
4. Match tegen Formitable: check of restaurant een widget heeft
5. Upsert in Supabase op `google_place_id` (bestaande data niet overschrijven)
6. Frequentie: eerste keer bulk, daarna wekelijks/maandelijks

### Flow 3: Reservering maken

1. Gebruiker klikt tijdslot of restaurant
2. BookingModal opent op basis van `booking_type`:
   - **formitable**: embedded Formitable widget iframe
   - **link**: redirect naar externe reserveer-URL
   - **manual**: toon telefoon + email
3. Wij verwerken geen reserveringen zelf — zero liability

---

## 6. Design Richting

- **Basis**: Merlijne's huidige design behouden (elegantie, typografie)
- **Moderniseren**: subtiele animaties, glasmorfisme, modernere transitions
- **Design tokens** (migreren uit huidige CSS):
  - `--cream: #F2EDE3`
  - `--ivory: #FAF8F3`
  - `--charcoal: #1C1916`
  - `--mid: #6E6860`
  - `--accent: #8C8378`
  - Fonts: Cormorant Garamond (headings), Jost (body)
- **Mobile-first**: primair ontworpen voor telefoon

---

## 7. MVP Features (alle fases)

1. Zoek op datum + personen (core flow)
2. Filters (buurt, keuken, prijsklasse)
3. Curated "Table me picks" badge + sectie
4. "Vanavond beschikbaar" quick-button
5. Direct boeken via Formitable widget embed
6. Kaartweergave (Mapbox)
7. Favorieten (localStorage)
8. Mobile-first responsive design

---

## 8. Fases

### Fase 1 — Fundament (eerste deploy)

- Next.js 14 project opzetten met Tailwind CSS
- Merlijne's design tokens migreren (kleuren, fonts, spacing)
- Supabase opzetten + restaurant tabel + seed met huidige 10 restaurants
- SearchBar component (datum + personen)
- RestaurantCard + RestaurantList componenten
- Echte Formitable beschikbaarheid ophalen via API route
- BookingModal met Formitable widget embed
- Responsive design (mobile-first)
- Deploy naar Vercel

**Resultaat:** Werkende site met 10 restaurants en echte beschikbaarheid.

### Fase 2 — Data & Filters

- Google Places API integratie (bulk import Amsterdam restaurants)
- Automatische Formitable widget-ID matching
- Google Places foto's als default + upload override systeem
- FilterBar component (buurt, keuken, prijsklasse)
- Neighborhood + cuisine lookup tabellen vullen
- Zoekresultaten filteren server-side
- Restaurant detailpagina (/[slug])

**Resultaat:** Honderden restaurants met foto's en filters.

### Fase 3 — Kaart & Quick Actions

- Mapbox kaartweergave (/kaart)
- Restaurant pins met kleur-codering (beschikbaar/vol)
- Popup met restaurant info + direct reserveren
- "Vanavond beschikbaar" quick-button
- Favorieten systeem (localStorage)
- "Mijn favorieten" filter optie

**Resultaat:** Twee browse-modi (lijst + kaart), snelle acties, persoonlijke touch.

### Fase 4 — Polish & Curated

- "Table me picks" curated badge + sectie
- Curated collections ("Beste terrassen", "Date night")
- Design modernisering (animaties, glasmorfisme, transitions)
- Loading skeletons en micro-interacties
- Mobile optimalisatie (touch targets, swipe, PWA)
- SEO optimalisatie (meta tags, structured data, sitemap)
- Performance (caching, ISR, image optimization)

**Resultaat:** Premium, gepolijst platform klaar om te delen en te groeien.

---

## 9. Externe Services & Kosten

| Service        | Gratis tier                        | Wanneer betalen?              |
|----------------|------------------------------------|-------------------------------|
| Vercel         | 100GB bandwidth, serverless        | Zeer hoge traffic             |
| Supabase       | 500MB DB, 1GB storage, 50k MAU     | 200+ restaurants met veel foto's |
| Mapbox         | 50.000 map loads/maand             | >50k kaart-views/maand        |
| Google Places  | $200/maand credit                  | Na ~5000 requests/maand       |
| Formitable     | Publieke widgets, gratis           | Als ze API-toegang beperken   |

**Totale kosten v1:** €0 (alles binnen gratis tiers)

---

## 10. Risico's & Mitigatie

| Risico | Impact | Mitigatie |
|--------|--------|-----------|
| Formitable blokkeert scraping | Hoog — geen beschikbaarheid | Widget embed als fallback, contact opnemen voor API toegang |
| Google Places API kosten | Middel | Cache agressief, import alleen periodiek, monitor usage |
| Te veel restaurants zonder booking | Laag | Markeer als "manual", toon bel/mail als fallback |
| Performance bij veel restaurants | Middel | Server-side filtering, pagination, ISR caching |
| Mapbox gratis tier overschreden | Laag | Fallback naar Leaflet/OSM indien nodig |
