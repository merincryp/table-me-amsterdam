# Table me, Amsterdam

Vind live beschikbare restaurants in Amsterdam. Zoek, filter en reserveer direct.

---

## Wat is dit?

Table me, Amsterdam is een restaurant-platform dat live beschikbaarheid toont van restaurants in Amsterdam. Gebruikers kunnen zoeken op datum en aantal personen, filteren op buurt/keuken/prijs, en direct reserveren via Formitable of de website van het restaurant.

Bovenop de brede aggregator zit een **curated laag** — onze persoonlijke aanbevelingen ("Table me picks").

## Status

Het project wordt gebouwd in 4 fases:

| Fase | Wat | Status |
|------|-----|--------|
| **Fase 1** — Fundament | Next.js + Supabase + Formitable beschikbaarheid, 10 restaurants, deploy | Nog niet gestart |
| **Fase 2** — Data & Filters | Google Places bulk import, honderden restaurants, foto's, filters | Gepland |
| **Fase 3** — Kaart & UX | Mapbox kaartweergave, "vanavond vrij" button, favorieten | Gepland |
| **Fase 4** — Polish & Curated | Curated picks, animaties, SEO, performance, mobile PWA | Gepland |

## Tech Stack

| Component | Keuze |
|-----------|-------|
| Framework | Next.js 14 (App Router) |
| Styling | Tailwind CSS |
| Database | Supabase (Postgres + Storage) |
| Beschikbaarheid | Formitable (widget embed) |
| Restaurant data | Google Places API |
| Foto's | Google Places + handmatige override |
| Kaart | Mapbox GL JS |
| Hosting | Vercel |

## Project structuur

```
table-me-amsterdam/
├── app/                        # Next.js pagina's en API routes
│   ├── layout.tsx              # Root layout (fonts, header, footer)
│   ├── page.tsx                # Homepage (zoek + resultaten)
│   ├── kaart/page.tsx          # Kaartweergave (fase 3)
│   ├── [slug]/page.tsx         # Restaurant detail (fase 2)
│   └── api/                    # Server-side API routes
│       ├── restaurants/        # Restaurant data uit Supabase
│       └── availability/       # Formitable beschikbaarheid proxy
├── components/                 # React componenten
├── lib/                        # Supabase client, types, helpers
├── supabase/                   # Database migraties en seed data
├── docs/                       # Design specs en implementatie plannen
│   └── superpowers/
│       ├── specs/              # Design documenten
│       └── plans/              # Stap-voor-stap implementatie plannen
├── index.html.original         # Merlijne's originele prototype (referentie)
├── CLAUDE.md                   # AI instructies (workflow, principes)
└── .claude/                    # AI configuratie (commands, rules, agents)
```

## Lokaal draaien

```bash
# Dependencies installeren
npm install

# Environment variables instellen (kopieer en vul in)
cp .env.example .env.local

# Development server starten
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

## Environment variables

Je hebt accounts nodig bij:
- **Supabase** (gratis) — database + opslag
- **Mapbox** (gratis tot 50k loads) — kaartweergave (fase 3)
- **Google Cloud** (gratis $200 credit) — Places API (fase 2)

```env
NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbG...
SUPABASE_SERVICE_ROLE_KEY=eyJhbG...
```

## Design

Het design is gebaseerd op Merlijne's originele prototype met een elegante, minimalistische stijl:

- **Kleuren:** cream (#F2EDE3), ivory (#FAF8F3), charcoal (#1C1916)
- **Fonts:** Cormorant Garamond (headings), Jost (body)
- **Stijl:** Minimalistisch met moderne touch — subtiele animaties, glasmorfisme

## Documentatie

- [Design Spec](docs/superpowers/specs/2026-04-08-table-me-amsterdam-design.md) — Volledige technische specificatie
- [Fase 1 Plan](docs/superpowers/plans/2026-04-08-fase-1-fundament.md) — Stap-voor-stap implementatieplan

## Bijdragen

Dit project wordt gebouwd door Lennard en Merlijne. We werken via de `main` branch met duidelijke commits per feature.

---

*Table me, Amsterdam — omdat een goede tafel vinden niet moeilijk hoeft te zijn.*
