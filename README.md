# Impossibl Puzzle Investigation

On March 25, 2026, a poster referencing Cicada 3301 appeared near the 4th & King Caltrain station in San Francisco. Cicada illustration. QR code. White paper.

We traced the QR code to `impossibl.com/0`, solved the full multi-layered puzzle chain, and identified the organizer — Fabio Roma, founder of UltraContext — along with the complete source code on GitHub.

The puzzle is a hackathon qualifier for an event called "Impossibl," using Cicada 3301 aesthetics as a filter for technically curious participants.

This repo documents the full investigation.

---

## Files

| File | Description |
|------|-------------|
| `REPORT.md` | Complete narrative report |
| `INVESTIGATION.md` | Detailed investigation log with methodology |
| `OpenLetter.md` | Reflections on Cicada 3301 and this puzzle |
| `WhatIfItWas.md` | Criteria for authenticating real Cicada 3301 |
| `REDDIT_POST.md` | Summary post |
| `private_key.asc` | PGP private key extracted from the puzzle |

## Photos

| File | Description |
|------|-------------|
| `photos/cicada3301poster.jpeg` | The physical poster near Caltrain |
| `photos/devtoolssourcecicada3301.jpeg` | Chrome DevTools showing injected HTML comments |
| `photos/honeypot-page.jpeg` | The honeypot page |
| `photos/Cicada3301posterchatwithfounderultracontext1.jpeg` | LinkedIn chat with Fabio Roma (part 1) |
| `photos/Cicada3301posterchatwithfounderultracontext2.jpeg` | LinkedIn chat with Fabio Roma (part 2) |

---

## Puzzle Chain (Summary)

1. **Poster** on utility pole → QR code → `impossibl.com/0`
2. **Landing page** with ASCII cicada art and hidden JS-injected HTML comments
3. **Honeypot** at `/0/0x7A5` — a false trail designed to filter investigators
4. **Hidden API** found by reading the page's JavaScript source code
5. **GET the API** → hex-encoded base64 message: "Now POST this endpoint"
6. **POST the API** → PGP private key + Reddit pointer + invite token
7. **Portal** at `/portal` — paste token to register for the hackathon
8. **Hash verification** at `/h` — verify builder identity with SHA-256 hash
9. **Project submission** at `/0/submit` — submit hackathon projects
10. **Poster map** at `/map` — password-protected map of physical poster locations
11. **Dasha chatbot** at `/dasha` — AI-powered podcast search (separate project)
12. **Book cipher** (removed layer) — 20-tuple cipher referencing *The Fabric of Reality* PDF, recovered from git history. **Unsolved.**

---

## Key Findings

- **Organizer:** Fabio Roma, Founder @ UltraContext
- **Source code:** Public at `github.com/itsfabioroma/impossibl`
- **Event:** Impossibl hackathon, San Francisco, March 24, 2026
- **Sponsors:** UltraContext, Firecrawl
- **Our status:** Hash `53f5f72a0f171443` verified; waitlisted (event was full)
- **Unsolved:** Book cipher passphrase from removed `manifesto.txt` — requires decoding 20 word coordinates from David Deutsch's *The Fabric of Reality*
