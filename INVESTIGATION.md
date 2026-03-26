# Impossibl Puzzle Investigation

**Date:** 2026-03-25 / 2026-03-26
**Investigators:** Vir, Manav
**Entry Point:** Physical poster with QR code near 4th & King Caltrain station, SF
**Status:** Active — book cipher unsolved

---

## Summary

A Cicada 3301-style poster near the Caltrain station in San Francisco leads to `impossibl.com/0`, a multi-layered puzzle site. We solved the full puzzle chain, extracted a PGP key and invite token, registered at `/portal`, and identified the organizer as Fabio Roma (Founder @ UltraContext) via LinkedIn. The source code is public at `github.com/itsfabioroma/impossibl`. Phase 2 analysis of the source revealed additional layers, and Phase 3 analysis of git history recovered a removed book cipher layer — the deepest and only unsolved component of the puzzle.

---

## Phase 1: Puzzle Chain (Layers 1–7)

### Phase 1.1: Initial Recon

Fetched `impossibl.com/0` and `impossibl.com/0/0x7A5`.

- `/0` — minimal landing page: "We are looking for those who see what others don't."
- `/0/0x7A5` — rejection page: "You're not who we're looking for. Give up."
- Chrome DevTools showed HTML comments with Atlas Shrugged quotes and the path to `/0/0x7A5`

The HTML comments are injected by client-side JavaScript via `document.createComment()`, not present in the static HTML. This meant the puzzle logic lives in the JS bundles.

### Phase 1.2: Site Identification

The root domain `impossibl.com` reveals this is a hackathon: "Impossibl @ SF, March 24." Sponsors listed: Ultracontext (ultracontext.ai) and Firecrawl (firecrawl.dev).

### Phase 1.3: Path Enumeration

Tried ~20 URL paths including standard paths (`/robots.txt`, `/sitemap.xml`, `/pgp`, `/key`), numeric paths (`/1`, `/0/1`, `/0/3301`), hex paths (`/0/0x1`, `/0/0xCE5`), and thematic paths (`/0/puzzle`, `/0/motor`). All returned 404 except the two known pages.

### Phase 1.4: Infrastructure Analysis

| Component | Detail |
|-----------|--------|
| Hosting | Vercel, SFO region, IP `216.150.1.1` |
| SSL | Let's Encrypt, issued 2026-02-28 |
| DNS MX | Google Workspace |
| DNS TXT | Google site verification + SPF record |
| Framework | Next.js with React Server Components |

### Phase 1.5: Honeypot Identification

The `/0/0x7A5` page logs to console:
```
Nice try.
You took the bait.
Is that really the deepest you can look?
```

`0x7A5` = 1957 in decimal (publication year of Atlas Shrugged). This page is a deliberately planted false trail.

### Phase 1.6: React Server Components Payload

Used `curl -s -H "RSC: 1" https://impossibl.com/0` to identify JS chunk files:
- `/0` page: `7180e2d3b3f57119.js`
- `/0/0x7A5` page: `03937afc6f801e3a.js`

### Phase 1.7: Source Code Analysis

Downloaded and analyzed the JS bundles. The `/0` page source revealed:

1. ASCII cicada art rendered on canvas with animated effects
2. DOM comment injection code (the planted bait)
3. A silent fetch call: `fetch("/api/0x6578706c616e6174696f6e73").catch(()=>{})`

The hex endpoint name decodes to "explanations." This is the real path forward.

### Phase 1.8: Hidden API — GET

`GET /api/0x6578706c616e6174696f6e73` returned a hex-encoded Base64 string. Decoded:

> "Congratulations. Devotion to the truth is the hallmark of morality; there is no greater, nobler, more heroic form of devotion than the act of a man who assumes the responsibility of thinking. Now POST this same endpoint."

### Phase 1.9: Hidden API — POST

`POST /api/0x6578706c616e6174696f6e73` returned ~4.9KB JSON:

```json
{
  "next": "r/696D706F737369626C",
  "invite_token": "[REDACTED]",
  "decrypt_key": "data:application/pgp-keys;base64,LS0tLS1CRUdJTi..."
}
```

- `next` → hex decode → `r/impossibl` (Reddit subreddit, currently banned)
- `invite_token` → UUID for portal registration
- `decrypt_key` → PGP private key (decorative; source code confirms it's an env var with no encrypted content to decrypt)

### Phase 1.10: PGP Key Analysis

| Field | Value |
|-------|-------|
| User ID | `impossibl[0] <0@impossibl.com>` |
| Fingerprint | `750169CFCF2E0C8572789A60FAD6BE338F5A7FEB` |
| Key ID | `FAD6BE338F5A7FEB` |
| Encryption Subkey | `E3C689C1BBACEBDA` |
| Algorithm | RSA 2048-bit |
| Created | 2026-03-16 |
| Hidden Notation | `manu=2,2.5+1.12,0,3` |

The notation `manu=2,2.5+1.12,0,3` is embedded in the key's self-signature. Not referenced anywhere in the source code. Remains undecoded.

### Phase 1.11: Subreddit Investigation

`r/impossibl` returns `{"reason": "banned"}`. No Wayback Machine archive exists. The puzzle flow goes directly from the API to `/portal` without requiring Reddit access.

### Phase 1.12: Portal Registration

`impossibl.com/portal` — blank page with blinking cursor. Pasting the invite_token validates it against Supabase, assigns a builder number, generates `sha256(token).slice(0,16)` as a hash, and displays a registration form (name, email, phone, telegram, github).

The event was full at time of registration. Response: "impossibl[0] is full. you're on the list. impossibl[1] awaits."

### Phase 1.13: Organizer Identification

Manav found `github.com/itsfabioroma/impossibl` (the complete source code) and messaged Fabio Roma on LinkedIn. Fabio confirmed: "It was a hackaton yesterday."

---

## Phase 2: Source Code Analysis (Layers 8–11)

The GitHub repo at `github.com/itsfabioroma/impossibl` reveals four additional systems beyond the main puzzle chain.

### Layer 8: Hash Identity System (`/h` and `/h/[hash]`)

After portal registration, each builder receives a SHA-256 hash (first 16 chars of their invite_token). The `/h` page provides a verification system.

**Flow:**
1. `impossibl.com/h` — terminal-style page: "enter your hash."
2. Enter hash, press Enter
3. POSTs to `/api/hash/verify` — iterates all tokens in Supabase, computes `sha256(invite_token).slice(0,16)` for each, finds the match
4. If valid, redirects to `/h/[hash]` showing builder identity

**Our result:**
- Hash: `53f5f72a0f171443` (computed from `sha256("[REDACTED]")`)
- API response: `{"status":"valid","hash":"53f5f72a0f171443","builderNumber":null,"name":null}`
- Hash is valid but builder number is null (waitlisted, not a full builder)
- The `/h/53f5f72a0f171443` page shows: "verified.", "impossibl[0][]", hash, "verified at impossibl[0]. san francisco, 24 mar 2025."

### Layer 9: Project Submission (`/0/submit`)

A hackathon project submission form.

- Terminal-style page: "impossibl[0]" / "submit your project."
- Two fields: project description (280 char max) + GitHub repo URL
- POSTs to `/api/0/submit`, inserts into `impossibl_submissions` table
- On success: "submitted. the judges will see it. good luck, builder."

### Layer 10: Poster Location Map (`/map`)

A password-protected Google Maps interface for tracking physical poster and sticker locations across SF. Appears to be an admin/internal tool.

**5 poster/sticker variants:**

| ID | Label | Color | Description |
|----|-------|-------|-------------|
| 1 | Cicada | Red (#FF4444) | Main puzzle poster |
| 2 | Foguete | Blue (#44AAFF) | Portuguese for "rocket" |
| 3 | High Line | Green (#44DD44) | — |
| 4 | Find your way | Orange (#FFaa00) | — |
| 5 | Sticker | Purple (#CC44FF) | — |

**8 GPS coordinates from `data/pins.json`:**

| # | Coordinates | Variant | Area |
|---|-------------|---------|------|
| 1 | 37.75626, -122.38701 | Cicada | SoMa, near Caltrain |
| 2 | 37.75557, -122.38726 | Sticker | SoMa |
| 3 | 37.77135, -122.43554 | Cicada | Haight-Ashbury / Cole Valley |
| 4 | 37.77654, -122.42520 | High Line | Duboce Triangle |
| 5 | 37.77655, -122.42468 | Cicada | Duboce Triangle |
| 6 | 37.77663, -122.42436 | Sticker | Duboce Triangle |
| 7 | 37.77642, -122.42437 | Cicada | Duboce Triangle |
| 8 | 37.77623, -122.42479 | Cicada | Duboce Triangle |

All pins created 2026-03-17. Map is password-protected via `MAP_PASSWORD` env var. Password not derivable from puzzle content.

### Layer 11: Dasha's Second Brain (`/dasha` + `/api/chat`)

A two-part system unrelated to the puzzle chain, built as a personal project for Dasha Shunina (host of "Talks With Dasha" podcast, founder of Women Tech Meetup, Forbes contributor).

**Part A: The Dasha page (`/dasha`)**

An animated scroll experience addressed to Dasha Shunina (Дарья). Sequence includes:
- "Dasha." → "Or should I say Дарья?" → "dear Москва friend,"
- Rolling text acrostic spelling "YOU WANTED A SURPRISE"
- Sections about people who built the impossible
- Guest cards/photos from the podcast
- Reveal: "I built something while watching your podcast. It knows everything you've ever said."
- Links to the chatbot

**Part B: The Chat API (`/api/chat`)**

A Claude Opus-powered RAG chatbot serving as a searchable interface for the podcast.

- Uses `@ai-sdk/anthropic` with `claude-opus-4-6`
- Embeddings via OpenAI `text-embedding-3-small`
- 11 podcast episodes transcribed and chunked in `data/transcripts/`
- Retrieval: embeds user query, computes cosine similarity, returns relevant chunks
- System prompt: "You are Dasha's second brain — the AI that knows everything about the 'Talks With Dasha' podcast."

**Podcast episodes indexed:**
1. From Living in a Closet to YC's Fastest Growing AI Startup
2. Google Gemini AI — Peter Danenberg
3. How Corgi's Founders Raised $108M to Reinvent Insurance
4. Jeremiah Owyang — AI Influencer, Investor, Llama Lounge Host
5. Max Marchione — Founder at Superpower
6. Peter Walker — Head of Insights at Carta
7. Raising $30M in 7 Days
8. San Francisco is Back — But Only for AI Startups
9. Sasha Orloff — Co-Founder/CEO of Puzzle
10. This Founder Wants You to Stop Hiring Humans — Meet Jaspar, CEO of Artisan
11. Why Ideas Don't Matter — According to a 2x YC Founder Turned Investor

Built March 4-5, 2026. Co-authored with Claude (per commit messages).

---

## Phase 3: Git History Recovery (The Book Cipher)

Analysis of the git commit history revealed an entire puzzle layer that was built and then removed on March 16, 2026 — the same day the puzzle system was finalized. This is the deepest and only unsolved component.

### The Removed Manifesto (`manifesto.txt`)

Recovered from commit `b0c6c5e` (parent of `1fc9589` "remove manifesto"). Full text:

```
Explanations. Great explanations.

That's the engine of the future. The DNA of human progress.

Unfortunately, before they're accepted by common sense, they get dismissed as impossible. Delusional.

Why?

Because they break the foundations of power.

What happens when you manage to cheaply produce gold in your garage?

Who owns the power now?

That's why incumbents hate progress. When you subvert structures of control, it's irreversible. And they know it. That's the start of their downfall. And the beginning of infinity.

That's why they suppress you.

Until now.

There are roughly 150 people that control the world today. They know the foundation of their power is fragile. They fear better explanations. Because better explanations force new power structures. And they are terrified of losing theirs.

They are afraid of the impossible. We are not.

We ship it.

And we usually get ridiculed, punished and ostracized for it.

There are others. Not many. If you're reading this, it's probably too late.

Follow the inner light.

Here is the genesis. Count as an engineer. Everything counts.

https://ia601208.us.archive.org/24/items/TheFabricOfReality/The_Fabric_of_Reality.pdf

(0, 0, 8)
(12, 0, 42)
(25, 1, 3)
(38, 0, 10)
(51, 0, 25)
(63, 0, 7)
(76, 0, 1)
(89, 1, 0)
(102, 0, 61)
(114, 2, 17)
(127, 0, 60)
(140, 0, 25)
(153, 1, 19)
(151, 26, 9)
(178, 0, 49)
(191, 0, 9)
(204, 0, 19)
(216, 0, 6)
(229, 0, 23)
(242, 1, 23)
```

**Analysis:**
- References David Deutsch's *The Fabric of Reality* (1997) and *The Beginning of Infinity* (2011) — both about epistemology and the nature of explanations
- "Count as an engineer. Everything counts." = use 0-indexed counting
- The 20 tuples are a **book cipher** — coordinates into the PDF
- Format: **(page, paragraph, word)** — all 0-indexed
- Decoding the 20 words from the PDF should produce a passphrase

**Status: UNSOLVED.** The PDF at the archive.org URL needs to be downloaded and each of the 20 coordinates looked up manually.

### The Removed `/$` Page (`app/$/page.tsx`)

Recovered from commit `d166fd8` (parent of `d19f4b7` "remove /$ page").

- Black screen with text input: "enter passphrase"
- Submits to `POST /api/check-pass` with `{"pass": "input"}`
- If correct → reveals: "The world you desired can be won. It exists, it is real, it is possible — it is yours."
- If wrong → "access denied"

The `/$` page is the lock. The book cipher is the key.

### The Removed `/api/check-pass` Route

Removed in commit `7543110` ("remove check-pass API route"). Source code not recovered — would require checking the parent commit. Likely compared input against an environment variable or hardcoded passphrase.

### The Removed `/1` Page

Removed in commit `2a29185` ("remove skiper28.bak and unused /1 page"). Contents not recovered.

### Original vs. Simplified Puzzle Flow

The original puzzle had an additional layer between the API and the portal:

```
SIMPLIFIED FLOW (current, after March 16 removals):
POST /api/explanations → invite_token → /portal

ORIGINAL FLOW (before March 16):
POST /api/explanations → manifesto.txt (book cipher)
→ decode passphrase from The Fabric of Reality PDF
→ /$ (enter passphrase) → ??? → /portal
```

All removals occurred on March 16 in rapid succession:

| Commit | Message | Removed |
|--------|---------|---------|
| `d19f4b7` | "remove /$ page" | `app/$/page.tsx` (passphrase entry) |
| `7543110` | "remove check-pass API route" | `/api/check-pass` (passphrase verification) |
| `2a29185` | "remove skiper28.bak and unused /1 page" | backup + `/1` page |
| `1fc9589` | "remove manifesto" | `manifesto.txt` (book cipher) |
| `d166fd8` | "remove John Galt from GET hint" | Modified API hint text |

### Possible Connection to Map Password

The passphrase derived from the book cipher could potentially be the map password (stored as `MAP_PASSWORD` env var). We tried ~40 guesses without success. The book cipher passphrase remains the most likely candidate.

---

## Complete URL Map

### Active Pages

| URL | Content |
|-----|---------|
| `impossibl.com` | Hackathon landing page |
| `impossibl.com/0` | Puzzle entry: ASCII cicada + hidden API fetch |
| `impossibl.com/0/0x7A5` | Honeypot page |
| `impossibl.com/portal` | Token validation + registration |
| `impossibl.com/0/submit` | Project submission form |
| `impossibl.com/h` | Hash entry page |
| `impossibl.com/h/{hash}` | Builder verification badge |
| `impossibl.com/map` | Password-protected poster location map |
| `impossibl.com/dasha` | Podcast AI chatbot landing |

### Removed Pages (recovered from git history)

| URL | Content | Removed in commit |
|-----|---------|-------------------|
| `impossibl.com/$` | Passphrase entry (book cipher lock) | `d19f4b7` |
| `impossibl.com/1` | Unknown | `2a29185` |

### API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/0x6578706c616e6174696f6e73` | GET | Returns hex(base64(instruction)) |
| `/api/0x6578706c616e6174696f6e73` | POST | Generates invite_token, stores in Supabase |
| `/api/portal/validate` | POST | Validates invite_token |
| `/api/portal/claim` | POST | Claims hackathon spot; sends email with Telegram link |
| `/api/portal/waitlist` | POST | Waitlist when event is full |
| `/api/0/submit` | POST | Submits hackathon project |
| `/api/hash/verify` | POST | Verifies builder hash |
| `/api/map/auth` | POST | Map page authentication |
| `/api/map` | GET/POST | Pin CRUD for poster locations |
| `/api/chat` | POST | Dasha podcast chatbot |
| `/api/check-pass` | POST | Passphrase verification (REMOVED) |

---

## Artifacts

### Files in This Repository

| File | Description |
|------|-------------|
| `INVESTIGATION.md` | This file — full investigation log |
| `REPORT.md` | Narrative report |
| `OpenLetter.md` | Reflections on Cicada 3301 and the puzzle |
| `WhatIfItWas.md` | Comparison with real Cicada 3301 criteria |
| `REDDIT_POST.md` | Summary post |
| `private_key.asc` | PGP private key extracted from the puzzle |
| `photos/` | Physical poster, DevTools screenshots, LinkedIn chat |

### Extracted Secrets

| Item | Value |
|------|-------|
| invite_token | `[REDACTED]` |
| Builder hash | `53f5f72a0f171443` |
| PGP Key Fingerprint | `750169CFCF2E0C8572789A60FAD6BE338F5A7FEB` |
| PGP Key ID | `FAD6BE338F5A7FEB` |
| PGP Key Notation | `manu=2,2.5+1.12,0,3` |
| Hidden API | `/api/0x6578706c616e6174696f6e73` (hex for "explanations") |

### Infrastructure

| Component | Detail |
|-----------|--------|
| Domain | impossibl.com (GoDaddy, WHOIS privacy) |
| Hosting | Vercel (SFO region) |
| Framework | Next.js with React Server Components |
| Database | Supabase |
| SSL | Let's Encrypt (issued 2026-02-28) |
| DNS | GoDaddy nameservers, Google Workspace MX |

---

## Timeline

| Date | Event |
|------|-------|
| 2026-02-28 | SSL certificate issued for impossibl.com |
| 2026-03-04–05 | Dasha page + chat API built (per commit history) |
| 2026-03-13 | `/0` puzzle page created |
| 2026-03-16 | PGP key created; cicada puzzle system built; portal created; book cipher layer (manifesto, `/$` page, check-pass API) built then removed same day |
| 2026-03-17 | `/map` with Google Maps + poster pins created |
| 2026-03-22 | Original home page restored |
| 2026-03-23 | Waitlist system added |
| 2026-03-24 | Impossibl hackathon takes place in San Francisco; `/h` hash verification pages created |
| 2026-03-25 | `/0/submit` project submission form added |
| 2026-03-25 ~4:58pm | Poster spotted near 4th & King Caltrain station |
| 2026-03-25 ~4:58pm | Manav contacts Fabio Roma on LinkedIn |
| 2026-03-25 ~5:04pm | Fabio confirms: "It was a hackaton yesterday" |
| 2026-03-25 evening | Full puzzle chain solved; source code found on GitHub |
| 2026-03-26 | Hash verification completed; Phase 2 source analysis |

---

## Remaining Unknowns

1. **Book cipher passphrase (UNSOLVED)** — 20 tuples from `manifesto.txt` need to be decoded against *The Fabric of Reality* PDF at `https://ia601208.us.archive.org/24/items/TheFabricOfReality/The_Fabric_of_Reality.pdf`. Format: (page, paragraph, word), 0-indexed. This is the deepest unsolved layer.
2. **Map password** — stored as `MAP_PASSWORD` env var; possibly the decoded book cipher passphrase
3. **PGP key notation `manu=2,2.5+1.12,0,3`** — not referenced anywhere in source code
4. **r/impossibl content** — subreddit banned, no archive exists
5. **`/api/check-pass` source** — removed in commit `7543110`; source not recovered
6. **`/1` page contents** — removed in commit `2a29185`; not recovered
7. **impossibl[1]** — second event referenced in waitlist message
8. **Telegram group** — `https://t.me/+yxI9zUYC8CMwN2Nh` (from portal claim confirmation email); not explored
