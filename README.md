# Impossibl: A Cicada 3301 Ripoff for a Hackathon

On March 25, 2026, a poster mimicking the iconic Cicada 3301 style appeared on a telephone pole near the 4th & King Caltrain station in San Francisco. Cicada illustration. QR code. White paper. The whole act.

It wasn't Cicada 3301. It was a marketing stunt for a hackathon called "Impossibl" run by **Fabio Roma**, founder of **UltraContext**, with **Firecrawl** as a sponsor. We cracked the entire puzzle in one evening and traced it back to him in 6 minutes via LinkedIn.

This repo documents the full investigation.

---

## Files

| File | Description |
|------|-------------|
| `REPORT.md` | Complete narrative report of the investigation |
| `INVESTIGATION.md` | Raw 25-phase investigation log with exact commands and methodology |
| `WhatIfItWas.md` | What would have had to be true for this to be real Cicada 3301 |
| `fuckthebayarea.md` | An open letter to the people who did this |
| `private_key.asc` | PGP private key extracted from the puzzle |

## Photos

| File | Description |
|------|-------------|
| `photos/cicada3301poster.jpeg` | The physical poster near Caltrain station |
| `photos/devtoolssourcecicada3301.jpeg` | Chrome DevTools showing injected HTML comments |
| `photos/honeypot-page.jpeg` | The "Give up" honeypot page |
| `photos/Cicada3301posterchatwithfounderultracontext1.jpeg` | LinkedIn DM with Fabio Roma (part 1) |
| `photos/Cicada3301posterchatwithfounderultracontext2.jpeg` | LinkedIn DM with Fabio Roma (part 2) |

---

## The Puzzle Chain (Summary)

1. **Poster** on telephone pole -> QR code -> `impossibl.com/0`
2. **Landing page** with ASCII cicada art and hidden JS-injected HTML comments
3. **Honeypot** at `/0/0x7A5` - designed to waste your time ("Give up")
4. **Hidden API** found by reading the page's JavaScript source code
5. **GET the API** -> hex-encoded base64 message saying "Now POST this endpoint"
6. **POST the API** -> PGP private key + Reddit subreddit (banned) + invite token
7. **Reddit subreddit** `r/impossibl` is banned. Trail goes cold.
8. **LinkedIn** -> Fabio Roma, Founder @ UltraContext, confirms: "It was a hackaton yesterday"

---

## Verdict

No PGP-signed messages. No Tor. No real cryptography. Vercel hosting with Google Analytics. Founder found on LinkedIn in 6 minutes. Hex encoding is not encryption.

These people pollute mother Earth with their presence