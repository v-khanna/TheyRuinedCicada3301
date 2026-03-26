# Impossibl ARG Investigation

**Date:** 2026-03-25
**Investigators:** vir + Claude
**Entry Point:** Physical barcode -> `https://impossibl.com/0`
**Status:** In Progress - blocked at subreddit (banned) and rate-limited API

---

## TL;DR

A physical barcode (appeared March 25) leads to `impossibl.com/0`, a Cicada 3301-styled puzzle site. It is NOT real Cicada 3301 - it's an ARG tied to a hackathon called "Impossibl @ SF, March 24" sponsored by Ultracontext and Firecrawl. We cracked through 5 layers of the puzzle and extracted a PGP private key, a Reddit subreddit name, and an invite token. The subreddit is banned and the API is now rate-limited.

---

## Methodology: Exactly What We Did

### Phase 1: Initial Recon (Web Fetching)

**What we did:** Fetched `impossibl.com/0` and `impossibl.com/0/0x7A5` using the WebFetch tool, which downloads a URL and extracts content.

**Why:** To see what was on the pages the user found via barcode and in the HTML source screenshot they shared.

**What we found:**
- `/0` shows "We are looking for those who see what others don't." - minimal, mysterious landing page
- `/0/0x7A5` shows "You're not who we're looking for. Give up." - appeared to be a rejection page
- The user's screenshot of Chrome DevTools showed HTML comments with Atlas Shrugged quotes and the path `impossibl.com/0/0x7A5`

**What we missed initially:** WebFetch strips JS-rendered content. The HTML comments visible in the user's DevTools screenshot were NOT in the static HTML we fetched - they were injected by client-side JavaScript. This became important later.

### Phase 2: Identifying the Site's True Nature

**What we did:** Fetched the root domain `impossibl.com` (without `/0`).

**Why:** To understand what the site actually is, beyond the puzzle page.

**What we found:** The root page reveals this is **"Impossibl @ SF, March 24"** - a hackathon in San Francisco. The meta description says "A hackathon in San Francisco, March 24." It links to two sponsors: Ultracontext (ultracontext.ai) and Firecrawl (firecrawl.dev).

**Conclusion:** This is NOT real Cicada 3301. It's a hackathon ARG using Cicada aesthetics.

### Phase 3: Broad Path Enumeration

**What we did:** Tried ~20 different URL paths on the site to find hidden pages.

**Why:** Puzzle sites often hide content at non-obvious paths. We tried:
- Standard paths: `/robots.txt`, `/sitemap.xml`, `/pgp`, `/key`, `/api`, `/apply`, `/whoami`
- Numeric paths: `/1`, `/0/1`, `/0/3301`, `/0/1957`
- Hex-themed paths: `/0/0x1`, `/0/0xCE5` (3301 in hex), `/0/0xCE4`
- Thematic paths: `/0/puzzle`, `/0/motor` (from the quote "I have stopped your motor")
- `/3301` (direct Cicada reference)

**What we found:** ALL returned 404 except the two known pages (`/0` and `/0/0x7A5`). The site has a very limited surface area.

### Phase 4: Infrastructure Analysis

**What we did:** Checked HTTP headers, DNS records, and SSL certificate.

**Why:** Infrastructure details can reveal the operators, hidden services, or additional clues.

**Commands/tools used:**
- `curl -sI https://impossibl.com/0` - HTTP response headers
- `dig impossibl.com TXT +short` - DNS TXT records
- `dig impossibl.com MX +short` - DNS mail records

**What we found:**
- **Hosting:** Vercel, SFO region, IP `216.150.1.1`
- **SSL:** Let's Encrypt cert, issued 2026-02-28, expires 2026-05-29
- **DNS MX:** Google Workspace (they use Gmail for `@impossibl.com`)
- **DNS TXT:** Google site verification + SPF record
- **Next.js headers:** `x-matched-path: /0`, confirming route exists as a real page
- **No hidden DNS TXT clues** (some ARGs hide messages in DNS TXT records)

### Phase 5: Static HTML vs Client-Rendered Content

**What we did:** Used `curl` to fetch raw HTML and extract all HTML comments with `grep -o '<!--[^>]*-->'`.

**Why:** The user's screenshot showed Atlas Shrugged quotes as HTML comments, but our WebFetch didn't find them. We needed to understand why.

**What we found:** The raw HTML only contains React markers (`<!--$-->`, `<!--/$-->`) and a build ID (`<!--gaWqbX9CM0xHB6_CNYiQ8-->`). The Atlas Shrugged quotes are NOT in the static HTML at all. This means they're injected by JavaScript after the page loads.

**Why this matters:** This was the key insight. If the comments are injected by JS, then the JS itself contains the real puzzle logic. We needed to read the actual JavaScript source code.

### Phase 6: React Server Components Payload

**What we did:** Fetched the RSC (React Server Components) payload by sending `curl -s -H "RSC: 1" https://impossibl.com/0`.

**Why:** Next.js apps serve their component data as a special RSC stream when you send the `RSC: 1` header. This reveals which JavaScript chunks render each page.

**What we found:** The RSC payload maps pages to their JS chunk files:
- `/0` page loads from chunk `7180e2d3b3f57119.js`
- `/0/0x7A5` page loads from chunk `03937afc6f801e3a.js`
- Shared framework code in `2f236954d6a65e12.js` and `1574975e990fbf56.js`

### Phase 7: Reading the Source Code (THE BREAKTHROUGH)

**What we did:** Downloaded the actual JavaScript files for both pages using `curl`.

**Why:** Since the visible content is rendered by client-side JS, the source code would reveal the actual puzzle logic.

**What we found in `7180e2d3b3f57119.js` (the `/0` page):**

1. **ASCII Art:** A large cicada/moth drawn with `@` symbols, rendered on a `<canvas>` element with animated glowing "firefly" effects on random characters.

2. **Comment Injection Code:** A React component that, after mounting, programmatically creates DOM comment nodes and inserts them into the page:
   ```js
   let o = [
     "I have stopped your motor.",
     "Do not attempt to find us. We do not choose to be found.",
     "The world you desired can be won, it exists, it is real, it is possible, it's yours.",
     "impossibl.com/0/0x7A5"
   ];
   ```
   These are injected via `document.createComment()` - designed to be found only by people who open DevTools and inspect the DOM.

3. **THE HIDDEN API CALL:** The most important find. Buried in the component code:
   ```js
   fetch("/api/0x6578706c616e6174696f6e73").catch(()=>{})
   ```
   The page silently makes a `fetch()` call to `/api/0x6578706c616e6174696f6e73` on load, then discards the result. This is a clue hidden in the network activity. `0x6578706c616e6174696f6e73` decoded from hex = **"explanations"**.

**What we found in `03937afc6f801e3a.js` (the `/0/0x7A5` page):**

Console logging that taunts anyone who followed the comment clue:
```js
console.log("Nice try.")
console.log("You took the bait.")
console.log("Is that really the deepest you can look?")
```

**This confirmed `/0/0x7A5` is a honeypot.** The real path forward was the hidden API endpoint.

### Phase 8: Hitting the Hidden API (GET)

**What we did:** `curl -s https://impossibl.com/api/0x6578706c616e6174696f6e73`

**What we got back:**
```json
{"next":"513239755a334a...4c673d3d"}
```

A JSON object with a `next` field containing a long hex string.

**Decoding (two layers):**
1. **Hex decode** the string -> produces Base64 text: `Q29uZ3JhdHVsYXRpb25z...ZHBvaW50Lg==`
2. **Base64 decode** that -> produces plaintext:

> "Congratulations. Devotion to the truth is the hallmark of morality; there is no greater, nobler, more heroic form of devotion than the act of a man who assumes the responsibility of thinking. Now POST this same endpoint."

**The instruction:** Send a POST request to the same URL.

### Phase 9: Hitting the Hidden API (POST) - THE PAYLOAD

**What we did:** `curl -s -X POST https://impossibl.com/api/0x6578706c616e6174696f6e73`

**What we got back (JSON, ~4.9KB):**
```json
{
  "next": "r/696D706F737369626C",
  "invite_token": "[REDACTED]",
  "decrypt_key": "data:application/pgp-keys;base64,LS0tLS1CRUdJTi..."
}
```

**Decoding each field:**

1. **`next`:** The prefix `r/` followed by hex `696D706F737369626C`. Hex decode = `impossibl`. So the next step is **`r/impossibl`** - a Reddit subreddit.

2. **`invite_token`:** A UUID `[REDACTED]`. Purpose unknown - possibly for gated access to something.

3. **`decrypt_key`:** A data URI containing a base64-encoded PGP private key. We decoded it and got a full `-----BEGIN PGP PRIVATE KEY BLOCK-----` key for `impossibl[0] <0@impossibl.com>`.

### Phase 10: PGP Key Analysis

**What we did:**
1. Saved the key to `private_key.asc`
2. Installed GnuPG via `brew install gnupg`
3. Imported the key: `gpg --import private_key.asc`
4. Inspected key packets: `gpg --list-packets private_key.asc`

**Key details:**
- **User ID:** `impossibl[0] <0@impossibl.com>`
- **Fingerprint:** `750169CFCF2E0C8572789A60FAD6BE338F5A7FEB`
- **Key ID:** `FAD6BE338F5A7FEB`
- **Encryption Subkey:** `E3C689C1BBACEBDA`
- **Algorithm:** RSA 2048-bit
- **Created:** 2026-03-16

**Hidden notation found in key signature:**
```
notation: manu=2,2.5+1.12,0,3
```
This is an embedded metadata field in the PGP key's self-signature. The name `manu` and value `2,2.5+1.12,0,3` are unusual. Possible interpretations:
- "Manu" = Sanskrit for the first human/lawgiver
- Could be coordinates, mathematical parameters, or encoded references
- The `+` in `2.5+1.12` is odd - could mean 3.62, or could be two separate values
- Remains undecoded

### Phase 11: Subreddit Investigation

**What we did:** Tried to access `r/impossibl` via:
- WebFetch (`www.reddit.com/r/impossibl`) - blocked by tool
- Reddit JSON API (`reddit.com/r/impossibl/.json`) - returned `{"reason": "banned", "message": "Not Found", "error": 404}`
- Wayback Machine API - no archived snapshots exist
- Web search for `r/impossibl` - no results

**Result:** The subreddit is **banned**. No cached content exists anywhere. Whatever was posted there is gone.

### Phase 12: invite_token Testing

**What we did:** Tried using the token on various endpoints:
- `GET /invite/{token}` - 404
- `POST /api/invite` with token in body - 404
- `GET /api/verify?token={token}` - 404
- `GET /0/{token}` - 404
- `Authorization: Bearer {token}` header on the API - returns same content as without it

**Result:** No endpoint accepts the token. It may be for a different platform (Discord, Slack, etc.) or for a part of the puzzle we haven't found yet.

### Phase 13: Rate Limiting Discovery

**What we did:** Tried to POST to the API again with the token in the body.

**What we got:** `{"error": "rate limit exceeded"}`

**Implication:** The API tracks requests and limits how many times you can hit the POST endpoint. This is likely to prevent scraping or brute-forcing, or it could be part of the puzzle design (one-time-use access).

### Phase 14: Additional Source Code Analysis

**What we did:** Fetched and analyzed additional JS chunks from the root page (`/`) and the OG image.

**Root page chunks:**
- `3bb4040fec1a7de3.js` - Next.js framework internals (136KB, just boilerplate)
- `d91ca5048c766b32.js` - Root page component with fancy particle/wave animations
- No hidden API calls or puzzle content on the root page

**OG image** (`impossibl.com/opengraph-image?6e2a084d8b3fd391`):
- Simple PNG: white text "IMPOSSIBL." on black background
- No steganographic content visible

**CSS** (`32707f0cad99cf4d.css`, 50KB):
- Standard Tailwind CSS output
- No hidden content declarations

---

## Inventory of Artifacts

### Files in This Directory

| File | Description |
|------|-------------|
| `INVESTIGATION.md` | This file - full investigation writeup |
| `private_key.asc` | PGP private key extracted from the puzzle |

### PGP Private Key Details

| Field | Value |
|-------|-------|
| User ID | `impossibl[0] <0@impossibl.com>` |
| Fingerprint | `750169CFCF2E0C8572789A60FAD6BE338F5A7FEB` |
| Key ID | `FAD6BE338F5A7FEB` |
| Encryption Subkey | `E3C689C1BBACEBDA` |
| Algorithm | RSA 2048-bit |
| Created | 2026-03-16 |
| Hidden Notation | `manu=2,2.5+1.12,0,3` |
| GPG Status | Imported to local keyring |

### Tokens and Secrets

| Item | Value |
|------|-------|
| invite_token | `[REDACTED]` |
| Vercel Deployment ID | `dpl_3ariMhLmSNgBracZ3oF4juUdDJ4Z` |
| Next.js Build ID | `gaWqbX9CM0xHB6-CNYiQ8` |

### URL Map

| URL | HTTP | Content |
|-----|------|---------|
| `impossibl.com` | 200 | Hackathon landing: "Impossibl @ SF, March 24" |
| `impossibl.com/0` | 200 | Puzzle entry: ASCII cicada + hidden JS comments + hidden API fetch |
| `impossibl.com/0/0x7A5` | 200 | Honeypot: "Give up" + console taunts |
| `impossibl.com/api/0x6578706c616e6174696f6e73` | 200 | Hidden API (GET=hex/b64 clue, POST=PGP key+subreddit+token) |
| `impossibl.com/robots.txt` | 404 | - |
| `impossibl.com/sitemap.xml` | 404 | - |
| `impossibl.com/pgp` | 404 | - |
| `impossibl.com/key` | 404 | - |
| `impossibl.com/api` | 404 | - |
| `impossibl.com/0/1` | 404 | - |
| `impossibl.com/0/3301` | 404 | - |
| `impossibl.com/0/0x1` | 404 | - |
| `impossibl.com/0/0xCE5` | 404 | (0xCE5 = 3301 in hex) |
| `impossibl.com/0/puzzle` | 404 | - |
| `impossibl.com/0/motor` | 404 | - |
| `impossibl.com/whoami` | 404 | - |
| `impossibl.com/3301` | 404 | - |
| `impossibl.com/1` | 404 | - |
| `impossibl.com/apply` | 404 | - |
| `impossibl.com/invite/{token}` | 404 | - |
| `impossibl.com/api/invite` | 404 | - |
| `impossibl.com/0/{token}` | 404 | - |

### DNS Records

| Type | Value |
|------|-------|
| A | 216.150.1.1 (Vercel) |
| MX | Google Workspace (aspmx.l.google.com + alternates) |
| TXT | `v=spf1 include:dc-aa8e722993._spfm.impossibl.com ~all` |
| TXT | `google-site-verification=xpnfDog2ESWqkHp4wo4FtHmS5uWeReSamSPvigh1GtI` |

### JS Source Files Analyzed

| Chunk Hash | Page | Content Summary |
|------------|------|-----------------|
| `7180e2d3b3f57119` | `/0` | ASCII cicada art, DOM comment injector, hidden `fetch("/api/0x6578706c616e6174696f6e73")` call |
| `03937afc6f801e3a` | `/0/0x7A5` | Honeypot page, console.log taunts |
| `1574975e990fbf56` | shared | Next.js runtime, error handling, React 19.3.0-canary |
| `7fb3a912a0ab3b90` | shared | Google Analytics, Vercel analytics |
| `2f236954d6a65e12` | shared | Layout, routing, metadata boundaries |
| `3bb4040fec1a7de3` | `/` | Next.js framework internals (136KB boilerplate) |
| `d91ca5048c766b32` | `/` | Root page: particle/wave animations, sponsor links |

### Technical Stack

| Component | Detail |
|-----------|--------|
| Framework | Next.js with React Server Components |
| Bundler | Turbopack |
| Hosting | Vercel (SFO region) |
| SSL | Let's Encrypt (issued 2026-02-28) |
| Fonts | Adelle Sans (18 weights), EB Garamond, JetBrains Mono |
| Analytics | Google Analytics + Vercel Analytics |
| Favicon | SVG, letter "i" in black rounded square |

### Encoding Methods Encountered

| Layer | Method | Example |
|-------|--------|---------|
| 1 | Hex encoding | `0x6578706c616e6174696f6e73` = "explanations" |
| 2 | Hex -> Base64 double encoding | API GET response |
| 3 | Base64 encoding | PGP key in data URI |
| 4 | Client-side DOM injection | Comments inserted via `document.createComment()` |
| 5 | PGP encryption | Private key provided for future decryption |
| 6 | Hex in JSON values | `696D706F737369626C` = "impossibl" in `next` field |

### Sponsors/Organizers

| Entity | URL | Role |
|--------|-----|------|
| Impossibl | impossibl.com | Hackathon organizer |
| Ultracontext | ultracontext.ai | Sponsor / primary organizer (AI company) |
| Firecrawl | firecrawl.dev | Sponsor (web scraping company) |

### Confirmed People

| Name | Role | Source |
|------|------|--------|
| **Fabio Roma** | Founder @ UltraContext | LinkedIn conversation (confirmed via DM) |

### Physical Poster Details

- **Format:** White paper poster on a wooden utility pole
- **Content:** Hand-drawn cicada/moth illustration (top) + QR code (bottom)
- **QR code links to:** `https://impossibl.com/0`
- **Location:** Near **310 Townsend St, San Francisco, CA** - adjacent to the **4th & King Caltrain station** in the SoMa district
- **Spotted by:** Friend of investigator (Manav)
- **Timing:** Poster was NOT present in the morning of March 25, appeared later that day (day AFTER the hackathon on March 24)
- **Style:** Deliberately mimics the original Cicada 3301 posters from 2012 (hand-drawn cicada + QR code on white paper, posted on public infrastructure)

### LinkedIn Confirmation (Phase 25)

Manav messaged Fabio Roma (Founder @ UltraContext) directly on LinkedIn:

- **Manav:** "Hi, did you put the cicada billboard outside the Caltrain station?"
- **Fabio:** "Hey. How did you find me?"
- **Manav:** "I went to the website and found you. I was just curious haha"
- **Fabio:** "Still there?"
- **Manav:** "I'm close by"
- **Manav:** "What is this for?"
- **Fabio:** "It was a hackaton yesterday"
- **Manav:** "I see. That's unfortunate I just saw this"

**Key observations from the conversation:**
1. Fabio was surprised someone traced it back to him ("How did you find me?")
2. He asked "Still there?" - suggesting the poster may be time-limited or he might want to show something in person
3. He confirmed it was for the hackathon yesterday (March 24)
4. He didn't deny or explain further - kept it brief and mysterious

---

## Timeline

| Date | Event |
|------|-------|
| 2026-02-28 | SSL certificate issued for impossibl.com |
| 2026-03-16 | PGP key created (`impossibl[0]`) |
| 2026-03-24 | Hackathon event: "Impossibl @ SF, March 24" |
| 2026-03-25 morning | Poster NOT present at Caltrain station (per witness) |
| 2026-03-25 ~4:58pm | Poster discovered near 310 Townsend / 4th & King Caltrain, SF |
| 2026-03-25 ~4:58pm | Manav messages Fabio Roma (Founder @ UltraContext) on LinkedIn |
| 2026-03-25 ~5:04pm | Fabio confirms: "It was a hackaton yesterday" |
| 2026-03-25 evening | Digital investigation begins; API cracked, PGP key extracted |
| 2026-03-25 | r/impossibl found to be banned on Reddit |
| 2026-03-25 | API rate-limited after initial POST |

### Phase 15: HTTP Method Enumeration

**What we did:** Tried PUT, DELETE, PATCH, OPTIONS on the hidden API.

**Why:** In CTFs and ARGs, different HTTP methods can return different content. We wanted to see if the server had hidden responses.

**What we found:**
- `OPTIONS` response: `Allow: GET, HEAD, OPTIONS, POST` - server explicitly declares supported methods
- `PUT`, `DELETE`, `PATCH` - empty responses (not implemented)
- `HEAD` - returns same headers as GET, no hidden custom headers

### Phase 16: Referrer and User-Agent Testing

**What we did:** Sent GET requests with different HTTP headers:
- `Referer: https://www.reddit.com/r/impossibl/` (pretending to come from the subreddit)
- `Referer: https://impossibl.com/0` (pretending to come from the puzzle page)
- `User-Agent: 3301` (Cicada reference)
- POST with the decoded Ayn Rand quote as the request body

**Why:** Some ARGs serve different content based on WHERE you came from or WHO you claim to be. This is a common CTF technique.

**Result:** ALL returned the same response. The API doesn't check referrer, user-agent, or request body content on GET. POST is still rate-limited.

### Phase 17: ASCII Art Deep Analysis

**What we did:** Extracted every character from the ASCII cicada art and analyzed the non-`@` characters.

**Why:** In Cicada 3301's original puzzles, they hid data in what appeared to be decorative elements. The art uses 13 unique characters (`@`, `%`, `#`, `{`, `[`, `}`, `<`, `]`, `(`, `*`, `-`, `:`, `=`), which is more than needed for simple ASCII art.

**What we found:**
- 495 `@` characters, 44 `%`, 18 `#`, 13 `{`, plus rare chars
- Treating `%` as 0 and `#` as 1 in binary produces fragments including "if" and "a" but no coherent message
- The rare characters (`{[(*<-:=]}`) in order don't form a recognizable pattern
- **Verdict:** Likely decorative/artistic rather than encoded. The char variety adds visual texture to the cicada shape.

### Phase 18: Subdomain Enumeration

**What we did:** Checked 8 common subdomains: www, api, mail, dev, staging, admin, puzzle, ctf.

**Why:** Hidden subdomains are a classic way to hide the next puzzle stage.

**Result:** Only `www.impossibl.com` exists (301 redirect to main site). No secret subdomains.

### Phase 19: WHOIS Domain Lookup

**What we did:** `whois impossibl.com`

**What we found:**
- **Registrar:** GoDaddy
- **DNS:** GoDaddy nameservers (ns07/ns08.domaincontrol.com)
- **Expires:** 2027-06-15 (registered for over a year out)
- **WHOIS Privacy:** Enabled - no registrant name, email, or org visible
- **Domain locks:** All 4 client status flags set (maximum protection)
- **Registrant:** Hidden behind privacy service

### Phase 20: Additional API Endpoint Enumeration

**What we did:** Tried hex-encoded words as API paths, following the pattern of the known endpoint:
- `/api/0x6d616e75` ("manu") - 404
- `/api/0x6e657874` ("next") - 404
- `/api/0x696e76697465` ("invite") - 404
- `/api/0x6b6579` ("key") - 404
- `/api/0x70757a7a6c65` ("puzzle") - 404

Also tried: `/api/auth`, `/api/puzzle`, `/api/next`, `/api/decrypt`, `/api/verify` - all 404

**Result:** Only ONE API endpoint exists on the entire site. The puzzle is tightly scoped.

### Phase 21: Infrastructure and .well-known Checks

**What we did:** Checked standard web infrastructure files:
- `/favicon.ico` - 404 (uses SVG icon instead)
- `/manifest.json` - 404
- `/.well-known/security.txt` - 404
- `/.well-known/assetlinks.json` - 404

**Result:** No hidden files in standard locations.

### Phase 22: OG Image Hash Analysis

**What we did:** Analyzed the query parameter `6e2a084d8b3fd391` from the OG image URL.

**Why:** Could be an encoded value rather than a cache key.

**Result:** Hex decodes to garbage bytes (`n*\x08M\x8b?\xd3\x91`). It's a Next.js content hash for cache-busting. **Dead end.**

### Phase 23: Sponsor and Organizer Research

**What we did:** Web searched for Ultracontext and Firecrawl in connection with the hackathon.

**What we found:**
- **Firecrawl** is a well-known web scraping API company. They sponsor many hackathons. No direct mention of "Impossibl" anywhere.
- **Ultracontext** has very little public presence. Not the same as "Contextual AI" (different company).
- Neither sponsor has publicly mentioned the Impossibl hackathon.

### Phase 24: Site Freshness Check

**What we did:** Compared the build ID and ETag to our earlier values.

**Result:** Build ID `gaWqbX9CM0xHB6-CNYiQ8` and ETag `14becd09a56d2c41422c5b0225a70892` are unchanged. The site has NOT been redeployed since we started investigating.

---

## Extended URL Map (updated)

| URL | HTTP | Content |
|-----|------|---------|
| `impossibl.com` | 200 | Hackathon landing: "Impossibl @ SF, March 24" |
| `impossibl.com/0` | 200 | Puzzle entry: ASCII cicada + hidden JS comments + hidden API fetch |
| `impossibl.com/0/0x7A5` | 200 | Honeypot: "Give up" + console taunts |
| `impossibl.com/api/0x6578706c616e6174696f6e73` | 200 | Hidden API (GET=hex/b64 clue, POST=PGP key+subreddit+token) |
| `www.impossibl.com` | 301 | Redirect to impossibl.com |
| `impossibl.com/robots.txt` | 404 | - |
| `impossibl.com/sitemap.xml` | 404 | - |
| `impossibl.com/pgp` | 404 | - |
| `impossibl.com/key` | 404 | - |
| `impossibl.com/api` | 404 | - |
| `impossibl.com/api/auth` | 404 | - |
| `impossibl.com/api/puzzle` | 404 | - |
| `impossibl.com/api/next` | 404 | - |
| `impossibl.com/api/decrypt` | 404 | - |
| `impossibl.com/api/verify` | 404 | - |
| `impossibl.com/api/0x6d616e75` | 404 | ("manu" in hex) |
| `impossibl.com/api/0x6e657874` | 404 | ("next" in hex) |
| `impossibl.com/api/0x696e76697465` | 404 | ("invite" in hex) |
| `impossibl.com/api/0x6b6579` | 404 | ("key" in hex) |
| `impossibl.com/api/0x70757a7a6c65` | 404 | ("puzzle" in hex) |
| `impossibl.com/0/1` | 404 | - |
| `impossibl.com/0/3301` | 404 | - |
| `impossibl.com/0/0x1` | 404 | - |
| `impossibl.com/0/0xCE5` | 404 | (0xCE5 = 3301 in hex) |
| `impossibl.com/0/puzzle` | 404 | - |
| `impossibl.com/0/motor` | 404 | - |
| `impossibl.com/whoami` | 404 | - |
| `impossibl.com/3301` | 404 | - |
| `impossibl.com/1` | 404 | - |
| `impossibl.com/apply` | 404 | - |
| `impossibl.com/invite/{token}` | 404 | - |
| `impossibl.com/api/invite` | 404 | - |
| `impossibl.com/0/{token}` | 404 | - |
| `impossibl.com/favicon.ico` | 404 | - |
| `impossibl.com/manifest.json` | 404 | - |
| `impossibl.com/.well-known/security.txt` | 404 | - |
| `impossibl.com/.well-known/assetlinks.json` | 404 | - |

---

## Open Questions / Next Steps

1. **Barcode appeared today (March 25), hackathon was yesterday (March 24).** Is this a post-event puzzle? A new phase? Or cleanup marketing?
2. **r/impossibl is banned.** Whatever content was there is lost. Was it PGP-encrypted messages that needed the key we found?
3. **invite_token has no known endpoint.** Could it be for Discord, Slack, Telegram, or a different service entirely?
4. **PGP key notation `manu=2,2.5+1.12,0,3`** is undecoded. This is metadata deliberately embedded in the key.
5. **The physical barcode itself** could contain steganographic data beyond the URL. A photo of it would be useful to analyze.
6. **API rate limit may reset** - worth retrying the POST with different payloads once it resets.
7. **Who runs Ultracontext?** Very low public presence - may be a new/stealth startup.
8. **The `[0]` in the PGP user ID** (`impossibl[0]`) - suggests there may be keys `[1]`, `[2]`, etc. for different puzzle stages.

---

## Authenticity Assessment

**NOT real Cicada 3301.** Evidence:
- No PGP-signed messages using the known Cicada 3301 key (fingerprint `7A35 090F 4BEB 2740 3EB9 1A26 3009 B10A E141 8876`)
- The key we found (`FAD6BE338F5A7FEB`) is completely different from Cicada's
- Hosted on Vercel (Cicada used Tor hidden services and raw HTML)
- Directly connected to a commercial hackathon event with named sponsors
- Domain registered recently (SSL from Feb 2026)
- Original Cicada 3301 went dark in ~2014
- Uses modern web tech (Next.js/React/Turbopack) vs Cicada's minimalist approach
- Cicada always signed their messages cryptographically - this site has zero signed content

**What it actually is:** A hackathon marketing stunt that rips off Cicada 3301's aesthetic and wraps it around Ayn Rand quotes and beginner-level encoding. The honeypot at `/0/0x7A5` is the only mildly interesting design choice - the rest is hex encoding and base64, which is not cryptography by any serious standard.
