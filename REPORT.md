# The Impossibl Puzzle: A Complete Breakdown

**Date of investigation:** March 25–26, 2026
**Location:** San Francisco, CA
**Investigators:** Vir, Manav

---

## What Happened

On the afternoon of March 25, 2026, a white paper poster was spotted on a wooden utility pole near 310 Townsend Street in San Francisco, adjacent to the 4th & King Caltrain station in the SoMa district. The poster featured a hand-drawn cicada illustration above a QR code — a clear reference to the Cicada 3301 internet puzzles that began in 2012.

Scanning the QR code led to `https://impossibl.com/0`.

What followed was several hours of digital forensics, source code analysis, and OSINT, ultimately tracing the puzzle to its creator.

---

## The Poster

The physical poster references the original Cicada 3301 style: a monochrome insect illustration on white paper, paired with a QR code, posted on public infrastructure near a transit hub. The original 2012 Cicada posters appeared on utility poles in 14 cities worldwide.

This version appeared near San Francisco's main commuter rail station, in a high-traffic area popular with the tech workforce.

**Photo:** `photos/poster.jpg`

---

## The Puzzle Chain

### Layer 1: The Landing Page (`impossibl.com/0`)

Black page with an animated ASCII art cicada rendered on an HTML canvas. One line of text: "We are looking for those who see what others don't."

Opening Chrome DevTools reveals four HTML comments injected into the DOM by client-side JavaScript:

```html
<!--I have stopped your motor.-->
<!--Do not attempt to find us. We do not choose to be found.-->
<!--The world you desired can be won, it exists, it is real, it is possible, it's yours.-->
<!--impossibl.com/0/0x7A5-->
```

The first three are quotes from Ayn Rand's *Atlas Shrugged*. The fourth points to the next URL. These comments are injected via `document.createComment()` — only visible in the live DOM, not in page source.

### Layer 2: The Honeypot (`impossibl.com/0/0x7A5`)

Following the URL leads to: "You're not who we're looking for. Give up."

`0x7A5` = 1957 (publication year of *Atlas Shrugged*). The browser console logs:

```
Nice try.
You took the bait.
Is that really the deepest you can look?
```

This is a deliberately planted false trail, designed to catch people who inspect elements but don't go further.

### Layer 3: The Hidden API Call

The real clue is in the JavaScript source code. The `/0` page's JS bundle contains a silent fetch call:

```javascript
fetch("/api/0x6578706c616e6174696f6e73").catch(()=>{})
```

The page makes this request on every load and discards the result. The hex endpoint name decodes to "explanations." Finding this requires reading the bundled JS, not just inspecting the rendered page.

### Layer 4: The Encoded Message (GET)

`GET /api/0x6578706c616e6174696f6e73` returns a hex-encoded Base64 string. Decoded (hex → base64 → plaintext):

> "Congratulations. Devotion to the truth is the hallmark of morality... Now POST this same endpoint."

### Layer 5: The Payload (POST)

`POST /api/0x6578706c616e6174696f6e73` returns:

- **`next`** → `r/impossibl` (hex-encoded; subreddit is banned)
- **`invite_token`** → `[REDACTED]`
- **`decrypt_key`** → PGP private key (decorative — no encrypted content exists to decrypt)

### Layer 6: The Portal (`/portal`)

Navigating to `impossibl.com/portal` shows a blank page with a blinking cursor. Pasting the invite_token validates it against Supabase, generates a SHA-256 hash, and displays a typewriter animation followed by a registration form (name, email, phone, telegram, github).

At time of our registration, the event was full: "impossibl[0] is full. you're on the list. impossibl[1] awaits."

### Layer 7: Hash Verification (`/h`)

After registration, builders can verify their identity at `impossibl.com/h` by entering their hash. The system checks all tokens in the database, finds the match, and displays a verification badge.

Our hash `53f5f72a0f171443` is valid. The verification page shows: "verified.", "impossibl[0][]" (no builder number — waitlisted), and the event date.

---

## Beyond the Puzzle Chain

Analysis of the public GitHub repository (`github.com/itsfabioroma/impossibl`) revealed additional systems.

### Project Submission (`/0/submit`)

A hackathon project submission form with two fields: project description (280 char max) and GitHub repo URL. Stores submissions in Supabase.

### Poster Location Map (`/map`)

A password-protected Google Maps interface tracking physical poster and sticker locations across SF. Five variants exist: Cicada (red), Foguete (blue), High Line (green), Find your way (orange), and Sticker (purple). Eight GPS pins are logged, concentrated in SoMa and Duboce Triangle. All pins created March 17, 2026.

### Dasha's Second Brain (`/dasha` + `/api/chat`)

A separate project built for Dasha Shunina, host of the "Talks With Dasha" podcast. The `/dasha` page is an animated scroll experience addressed to her, ending with a reveal of a Claude Opus-powered RAG chatbot trained on 11 podcast episode transcripts. The chatbot uses OpenAI embeddings for retrieval and Claude for generation. Built March 4–5, 2026, co-authored with Claude per commit messages.

This appears to serve a dual purpose: a portfolio piece and a personal pitch to get Dasha involved with the Impossibl event.

### The Book Cipher (Removed Layer — Deepest Found)

Analysis of the git commit history recovered an entire puzzle layer that was built and removed on March 16, 2026 — the same day the puzzle was finalized. This is the deepest and only unsolved component.

**The Manifesto:** A 54-line file (`manifesto.txt`) recovered from commit `b0c6c5e`. It references David Deutsch's *The Fabric of Reality* (1997) and *The Beginning of Infinity* (2011), contains a philosophical statement about explanations and progress, and ends with:

> "Here is the genesis. Count as an engineer. Everything counts."

Followed by a link to the PDF of *The Fabric of Reality* on archive.org, and 20 tuples:

```
(0, 0, 8)    (12, 0, 42)   (25, 1, 3)    (38, 0, 10)
(51, 0, 25)  (63, 0, 7)    (76, 0, 1)    (89, 1, 0)
(102, 0, 61) (114, 2, 17)  (127, 0, 60)  (140, 0, 25)
(153, 1, 19) (151, 26, 9)  (178, 0, 49)  (191, 0, 9)
(204, 0, 19) (216, 0, 6)   (229, 0, 23)  (242, 1, 23)
```

These are a **book cipher** — coordinates in the format **(page, paragraph, word)**, 0-indexed. Decoding the 20 words from the PDF should produce a passphrase.

**The Lock:** A removed page at `impossibl.com/$` (recovered from commit `d166fd8`) showed a text input: "enter passphrase." It submitted to `POST /api/check-pass`. Correct passphrase → success message. Wrong → "access denied."

**Original puzzle flow (before March 16 removals):**

```
POST /api/explanations → manifesto (book cipher) → decode passphrase
→ /$ (enter passphrase) → ??? → /portal
```

**Simplified flow (current):**

```
POST /api/explanations → invite_token → /portal
```

The book cipher layer was cut, and the API was changed to return the invite_token directly. The decoded passphrase may also be the map password (`MAP_PASSWORD` env var), which we were unable to guess.

**Status: UNSOLVED.** Requires downloading the PDF and looking up each coordinate.

---

## The Person Behind It

**Fabio Roma** — Founder @ UltraContext (ultracontext.ai). Source code is public at `github.com/itsfabioroma/impossibl`.

The root domain credits two sponsors: **UltraContext** and **Firecrawl** (firecrawl.dev).

Manav messaged Fabio on LinkedIn after finding him through the UltraContext sponsor link:

> **Manav:** "Hi, did you put the cicada billboard outside the Caltrain station?"
> **Fabio:** "Hey. How did you find me?"
> **Manav:** "I went to the website and found you."
> **Fabio:** "It was a hackaton yesterday"

**Screenshots:** `photos/linkedin-chat-1.jpg`, `photos/linkedin-chat-2.jpg`

---

## Puzzle Architecture

```
PHYSICAL WORLD
    |
    v
[Cicada poster on utility pole near Caltrain]
    |  (scan QR code)
    v

LAYER 1 — SURFACE
[impossibl.com/0 — ASCII cicada + cryptic text]
    |  (open DevTools, inspect DOM)
    v

LAYER 2 — BAIT
[HTML comments: Atlas Shrugged quotes + link to /0/0x7A5]
    |  (follow the link)
    v

LAYER 3 — HONEYPOT
[/0/0x7A5 — "Give up." + console taunts]
    |  (go back, read JS source code)
    v

LAYER 4 — SOURCE CODE
[JS reveals silent fetch() to /api/0x6578706c616e6174696f6e73]
    |  (GET the endpoint)
    v

LAYER 5 — ENCODED MESSAGE
[hex(base64(Ayn Rand quote + "Now POST this endpoint"))]
    |  (POST the endpoint)
    v

LAYER 6 — THE PAYLOAD
[PGP key + r/impossibl + invite_token]
    |
    |  CURRENT FLOW: navigate to /portal, paste token
    |
    |  ORIGINAL FLOW (removed March 16):
    |  → manifesto.txt with book cipher
    |  → decode passphrase from The Fabric of Reality PDF
    |  → /$ enter passphrase → ??? → /portal
    |
    v

LAYER 7 — THE PORTAL
[Validates token, assigns builder number + hash, registration form]
    |
    v

LAYER 8 — HASH VERIFICATION (/h)
[Enter hash to verify builder identity]
    |
    v

LAYER 9 — PROJECT SUBMISSION (/0/submit)
[Submit hackathon project: description + GitHub URL]

---

REMOVED LAYER (recovered from git history):

BOOK CIPHER (manifesto.txt → /$ → /api/check-pass)
[20 tuples referencing The Fabric of Reality PDF]
[Format: (page, paragraph, word) — 0-indexed]
[STATUS: UNSOLVED]

---

INDEPENDENT SYSTEMS (not part of puzzle chain):

POSTER MAP (/map)
[Password-protected Google Maps with 8 poster/sticker GPS pins]

DASHA'S SECOND BRAIN (/dasha + /api/chat)
[Animated page + Claude-powered RAG chatbot for podcast]
```

---

## How This Compares to Cicada 3301

| Aspect | Cicada 3301 (2012–2014) | Impossibl (2026) |
|--------|-------------------------|------------------|
| **Poster style** | Cicada illustration + QR on utility poles | Same style — deliberate reference |
| **Cities** | 14 cities across 5 continents | San Francisco only |
| **Authentication** | All messages PGP-signed with known key | No signed messages |
| **Hosting** | Tor hidden services | Vercel (commercial cloud) |
| **Cryptography** | RSA, steganography, book ciphers | Hex/base64 encoding |
| **Philosophy** | Buddhist, Hermetic, Anglo-Saxon, Blake, Crowley | Exclusively Ayn Rand |
| **Purpose** | Unknown (never revealed) | Hackathon qualifier |
| **Organizer** | Unknown to this day | Fabio Roma, identified via LinkedIn |

---

## Technical Inventory

### Files in This Repository

| File | Description |
|------|-------------|
| `REPORT.md` | This file — narrative report |
| `INVESTIGATION.md` | Detailed investigation log with methodology |
| `OpenLetter.md` | Reflections on Cicada 3301 and this puzzle |
| `WhatIfItWas.md` | Criteria for authenticating real Cicada 3301 |
| `REDDIT_POST.md` | Summary post |
| `private_key.asc` | PGP private key extracted from the puzzle |
| `photos/` | Poster, DevTools screenshots, LinkedIn chat |

### Extracted Secrets

| Item | Value |
|------|-------|
| PGP Key Fingerprint | `750169CFCF2E0C8572789A60FAD6BE338F5A7FEB` |
| PGP Key ID | `FAD6BE338F5A7FEB` |
| PGP Key Notation | `manu=2,2.5+1.12,0,3` |
| invite_token | `[REDACTED]` |
| Builder hash | `53f5f72a0f171443` |
| Hidden API | `/api/0x6578706c616e6174696f6e73` |

### Infrastructure

| Component | Detail |
|-----------|--------|
| Domain | impossibl.com (GoDaddy, WHOIS privacy) |
| Hosting | Vercel (SFO region) |
| Framework | Next.js with React Server Components |
| Database | Supabase |
| SSL | Let's Encrypt (issued 2026-02-28) |
| DNS | GoDaddy nameservers, Google Workspace MX |
| Organizer | Fabio Roma, Founder @ UltraContext |
| Sponsors | UltraContext, Firecrawl |

---

## Timeline

| Date | Event |
|------|-------|
| 2026-02-28 | SSL certificate issued |
| 2026-03-04–05 | Dasha page + chat API built |
| 2026-03-13 | `/0` puzzle page created |
| 2026-03-16 | Cicada puzzle system, portal, PGP key created |
| 2026-03-17 | `/map` with poster location pins |
| 2026-03-24 | Impossibl hackathon; hash verification pages added |
| 2026-03-25 | Project submission form added; poster spotted near Caltrain |
| 2026-03-25 evening | Full investigation conducted |
| 2026-03-26 | Phase 2 source analysis; hash verification completed |

---

## Remaining Unknowns

1. **Book cipher passphrase (UNSOLVED)** — 20 tuples from `manifesto.txt` need to be decoded against *The Fabric of Reality* PDF. Format: (page, paragraph, word), 0-indexed. Deepest unsolved layer.
2. **Map password** — stored as `MAP_PASSWORD` env var; possibly the decoded book cipher passphrase
3. **PGP key notation `manu=2,2.5+1.12,0,3`** — not referenced in source code
4. **r/impossibl** — subreddit banned, no archive
5. **`/api/check-pass` source** — removed; not recovered from git
6. **`/1` page contents** — removed; not recovered
7. **impossibl[1]** — second event referenced in waitlist message
8. **Telegram group** — invite link exists (from portal claim email); not explored
