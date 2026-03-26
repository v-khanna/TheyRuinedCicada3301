# The Impossibl Puzzle: A Complete Breakdown

**Date of investigation:** March 25, 2026
**Location:** San Francisco, CA
**Investigators:** Manav and I

---

## What Happened

On the afternoon of March 25, 2026, a white paper poster was spotted on a wooden utility pole near 310 Townsend Street in San Francisco, right next to the 4th & King Caltrain station in the SoMa district. The poster featured a hand-drawn cicada illustration above a QR code - an unmistakable callback to the infamous Cicada 3301 internet puzzles that captivated the world starting in 2012.

Scanning the QR code led to `https://impossibl.com/0`.

What followed was several hours of digital forensics, cryptographic analysis, source code reverse-engineering, and ultimately a LinkedIn message to the person behind it all.

---

## The Poster

The physical poster deliberately mimics the original Cicada 3301 style: a monochrome insect illustration on white paper, paired with a QR code, posted on public infrastructure near a transit hub. The original 2012 Cicada posters appeared on utility poles in 14 cities worldwide (London, Warsaw, Sydney, Seoul, and others) and were one of the first physical-world components of what became the internet's most famous unsolved puzzle.

This version appeared near San Francisco's main commuter rail station, maximizing foot traffic from the tech-heavy Peninsula workforce.

**Photo:** `photos/poster.jpg`

---

## The Website

### Layer 1: The Landing Page (`impossibl.com/0`)

The QR code leads to a black page with an animated ASCII art cicada rendered on an HTML canvas. Below it, a single line of text in faint white:

> *"We are looking for those who see what others don't."*

At first glance, that's all there is. But opening Chrome DevTools and inspecting the DOM reveals four HTML comments that aren't visible on the page:

```html
<!--I have stopped your motor.-->
<!--Do not attempt to find us. We do not choose to be found.-->
<!--The world you desired can be won, it exists, it is real, it is possible, it's yours.-->
<!--impossibl.com/0/0x7A5-->
```

The first three are quotes from Ayn Rand's *Atlas Shrugged*. "I have stopped your motor" is John Galt's famous declaration. The fourth appears to be a URL pointing to the next step.

**Critical detail:** These comments are NOT in the page's static HTML. They are injected into the DOM by client-side JavaScript after the page loads, using `document.createComment()`. This means you can only find them by inspecting the live DOM in DevTools, not by viewing the page source. This distinction becomes important later.

### Layer 2: The Honeypot (`impossibl.com/0/0x7A5`)

Following the obvious clue from the HTML comment leads to a page that says:

> *"You're not who we're looking for.* **Give up.**"

The hex value `0x7A5` equals 1957 in decimal - the publication year of *Atlas Shrugged*. A thematic nod, but the page is a dead end.

Or is it? Opening the browser console reveals three messages that were silently logged:

```
Nice try.
You took the bait.
Is that really the deepest you can look?
```

This is a **honeypot** - a deliberately planted false trail. The HTML comments were bait designed to catch people who inspect elements but don't dig further. The console messages taunt you and hint that the real path requires looking deeper than the DOM.

### Layer 3: The Hidden API Call

This is where most investigators would get stuck. The real clue is not in what the page shows, but in what it does silently in the background.

To find it, you have to read the actual JavaScript source code that renders the `/0` page. The site is built with Next.js (a React framework), and each page is a separate JavaScript bundle. Using the React Server Components protocol (sending `RSC: 1` as an HTTP header), you can identify which JS file renders which page.

The `/0` page is rendered by a chunk at `/_next/static/chunks/7180e2d3b3f57119.js`. Downloading and reading this file reveals three things:

1. The ASCII cicada art definition and canvas rendering code
2. The function that injects the fake HTML comments into the DOM
3. **A silent `fetch()` call:**

```javascript
fetch("/api/0x6578706c616e6174696f6e73").catch(()=>{})
```

The page makes an HTTP request to this API endpoint every time it loads, then throws away the result. The endpoint name `0x6578706c616e6174696f6e73` is hexadecimal. Decoded, it spells: **"explanations"**.

This is the real door. Finding it requires reading the bundled JavaScript source - not just inspecting the rendered page.

### Layer 4: The Encoded Message (`GET /api/0x6578706c616e6174696f6e73`)

Hitting the hidden API endpoint with a GET request returns JSON:

```json
{
  "next": "513239755a334a...4c673d3d"
}
```

The `next` value is a long hexadecimal string. Decoding requires two steps:

1. **Hex decode** the string to get Base64 text
2. **Base64 decode** that to get the plaintext message:

> *"Congratulations. Devotion to the truth is the hallmark of morality; there is no greater, nobler, more heroic form of devotion than the act of a man who assumes the responsibility of thinking. Now POST this same endpoint."*

Another Ayn Rand quote, followed by an instruction: send a POST request to the same URL.

### Layer 5: The Payload (`POST /api/0x6578706c616e6174696f6e73`)

Sending a POST request to the same endpoint returns a much larger JSON response (~4.9KB) containing three fields:

```json
{
  "next": "r/696D706F737369626C",
  "invite_token": "[REDACTED]",
  "decrypt_key": "data:application/pgp-keys;base64,LS0tLS1CRUdJTi..."
}
```

**Decoding each field:**

`**next`** - The prefix `r/` followed by hex-encoded text. `696D706F737369626C` decodes to `impossibl`. The next step is the Reddit subreddit **r/impossibl**.

`**invite_token`** - A UUID. This is the key to the next step: pasting it into `/portal` to register for the hackathon.

`**decrypt_key`** - A PGP private key encoded as a Base64 data URI. Turns out to be decorative - the source code confirms it's just an environment variable served in the response. No encrypted messages exist to decrypt with it.

The `r/impossibl` subreddit pointer is a red herring or secondary channel - the actual puzzle flow goes directly from the API to `/portal`.

### Layer 6: The Portal (`/portal`)

Navigating to `impossibl.com/portal` shows a blank black page with a blinking cursor. Pasting the invite_token validates it against a Supabase database, assigns a builder number, generates a SHA-256 hash (first 16 chars) of the token, and plays a typewriter animation:

```
you're in.
congratulations, you're impossibl[0][22]

this is your hash: [REDACTED]
save it. you will never see it again.

time to meet the others.
the sharpest minds in the world. united.
one day. one house. one goal.
to ship the impossible.

impossibl[0]
hackathon. san francisco. march 24.
claim your spot below. details will follow.
```

Then a registration form appears: name, email, phone, telegram, github. That's it. That's what the entire puzzle was building to. A sign-up form.

We completed registration as **impossibl[0][22]** - the 22nd person to solve the puzzle chain.

---

## The Person Behind It

The root domain credits Ultracontext and Firecrawl as sponsors. We also found the entire source code on GitHub at `github.com/itsfabioroma/impossibl`.

The root domain `impossibl.com` (without `/0`) reveals it's a hackathon: "Impossibl @ SF, March 24". The page credits two sponsors: **Ultracontext** (ultracontext.ai) and **Firecrawl** (firecrawl.dev).

Manav searched LinkedIn for the Ultracontext founder and found **Fabio Roma** (Founder @ UltraContext). He messaged him directly:

> **Manav:** "Hi, did you put the cicada billboard outside the Caltrain station?"
> **Fabio:** "Hey. How did you find me?"
> **Manav:** "I went to the website and found you. I was just curious haha"
> **Fabio:** "Still there?"
> **Manav:** "I'm close by"
> **Manav:** "What is this for?"
> **Fabio:** "It was a hackaton yesterday"

Fabio confirmed the poster was related to the Impossibl hackathon that took place on March 24, 2026. His surprised "How did you find me?" suggests the puzzle was designed to be anonymous. His "Still there?" hints he may have wanted to meet in person - perhaps the puzzle's true endpoint was a face-to-face conversation.

**Screenshots:** `photos/linkedin-chat-1.jpg`, `photos/linkedin-chat-2.jpg`

---

## The Puzzle Architecture

Here's a diagram of the full puzzle structure and how each layer filters investigators:

```
PHYSICAL WORLD
    |
    v
[Cicada poster on utility pole near Caltrain]
    |
    v (scan QR code)

LAYER 1 - SURFACE
    |
    v
[impossibl.com/0 - ASCII cicada + "We are looking for those who see what others don't"]
    |
    |---> Most people stop here. Nothing to click, no obvious next step.
    |
    v (open DevTools, inspect DOM)

LAYER 2 - BAIT
    |
    v
[HTML comments with Atlas Shrugged quotes + link to /0/0x7A5]
    |
    |---> 0x7A5 = 1957 = publication year of Atlas Shrugged
    |
    v (follow the link)

LAYER 3 - HONEYPOT
    |
    v
[/0/0x7A5 - "You're not who we're looking for. Give up."]
[Console: "Nice try. You took the bait. Is that really the deepest you can look?"]
    |
    |---> Many investigators stop here, thinking it's a dead end.
    |
    v (go back, read the JavaScript source code)

LAYER 4 - SOURCE CODE
    |
    v
[JS reveals silent fetch() to /api/0x6578706c616e6174696f6e73]
[0x6578706c616e6174696f6e73 = "explanations" in hex]
    |
    v (GET the API endpoint)

LAYER 5 - ENCODED MESSAGE
    |
    v
[Response: hex-encoded Base64 string]
[Decoded: Ayn Rand quote + "Now POST this same endpoint"]
    |
    v (POST to the same endpoint)

LAYER 6 - THE PAYLOAD
    |
    v
[Response: PGP private key (decoration) + r/impossibl (red herring) + invite_token]
    |
    v (navigate to /portal, paste the token)

LAYER 7 - THE PORTAL
    |
    v
[/portal - blank cursor, paste invite_token]
[Validates against Supabase, assigns builder number + SHA-256 hash]
["you're in. congratulations, you're impossibl[0][22]"]
    |
    v (fill out registration form)

LAYER 8 - REGISTRATION (THE REAL ENDPOINT)
    |
    v
[Name, email, phone, telegram, github → hackathon sign-up]
["impossibl[0][22] confirmed. check your inbox."]

```

---

## How This Compares to Real Cicada 3301


| Aspect             | Real Cicada 3301 (2012-2014)                                         | Impossibl (2026)                                            |
| ------------------ | -------------------------------------------------------------------- | ----------------------------------------------------------- |
| **Poster style**   | White paper, cicada illustration, QR code on utility poles           | Identical style - deliberate homage                         |
| **Cities**         | 14 cities worldwide                                                  | San Francisco only (near Caltrain)                          |
| **Authentication** | All messages PGP-signed with known key (`7A35 090F...`)              | No signed messages; provides a private key instead          |
| **Hosting**        | Tor hidden services (.onion)                                         | Vercel (commercial cloud hosting)                           |
| **Technology**     | Raw HTML, steganography in images, book ciphers                      | Next.js React app, hex/base64 encoding, JS source hiding    |
| **Philosophy**     | Mix of many traditions (Buddhist, Hermetic, etc.)                    | Exclusively Ayn Rand / Objectivism                          |
| **Purpose**        | Unknown (recruitment for unknown organization)                       | Hackathon qualifier/marketing for Impossibl event           |
| **Difficulty**     | Extreme (RSA, prime numbers, Tor, steganography, physical locations) | Moderate (hex encoding, source code reading, HTTP methods)  |
| **Organizer**      | Still unknown to this day                                            | Fabio Roma, Founder @ UltraContext (confirmed via LinkedIn) |


---

## Technical Inventory

### Files in This Repository


| File | Description |
|------|-------------|
| `REPORT.md` | This file - clean narrative report |
| `INVESTIGATION.md` | Raw investigation log with methodology |
| `private_key.asc` | PGP private key extracted from the puzzle |
| `photos/cicada3301poster.jpeg` | Physical poster near Caltrain station |
| `photos/devtoolssourcecicada3301.jpeg` | Chrome DevTools showing injected HTML comments |
| `photos/honeypot-page.jpeg` | Screenshot of the honeypot page |
| `photos/Cicada3301posterchatwithfounderultracontext1.jpeg` | LinkedIn DM with Fabio Roma (part 1) |
| `photos/Cicada3301posterchatwithfounderultracontext2.jpeg` | LinkedIn DM with Fabio Roma (part 2) |
| `photos/ScreenshotCensored.png` | Portal registration completed (personal info redacted) |


### Extracted Secrets


| Item                | Value                                                      |
| ------------------- | ---------------------------------------------------------- |
| PGP Key Fingerprint | `750169CFCF2E0C8572789A60FAD6BE338F5A7FEB`                 |
| PGP Key ID          | `FAD6BE338F5A7FEB`                                         |
| PGP Key User        | `impossibl[0] <0@impossibl.com>`                           |
| PGP Key Notation    | `manu=2,2.5+1.12,0,3`                                      |
| invite_token        | `[REDACTED]`                     |
| Hidden API endpoint | `/api/0x6578706c616e6174696f6e73` (hex for "explanations") |
| Next destination    | `r/impossibl` (Reddit, currently banned)                   |


### Infrastructure


| Component | Detail                                                    |
| --------- | --------------------------------------------------------- |
| Domain    | impossibl.com (GoDaddy, WHOIS privacy)                    |
| Hosting   | Vercel (SFO region, IP 216.150.1.1)                       |
| Framework | Next.js with React Server Components                      |
| SSL       | Let's Encrypt (issued 2026-02-28)                         |
| DNS       | GoDaddy nameservers, Google Workspace MX                  |
| Organizer | Fabio Roma, Founder @ UltraContext                        |
| Sponsors  | UltraContext (ultracontext.ai), Firecrawl (firecrawl.dev) |


---

## Timeline


| Date               | Event                                                        |
| ------------------ | ------------------------------------------------------------ |
| 2026-02-28         | SSL certificate issued for impossibl.com                     |
| 2026-03-16         | PGP key `impossibl[0]` created                               |
| 2026-03-24         | Impossibl hackathon takes place in San Francisco             |
| 2026-03-25         | Cicada-style poster spotted near 4th & King Caltrain station |
| 2026-03-25 ~4:58pm | Manav contacts Fabio Roma on LinkedIn                 |
| 2026-03-25 ~5:04pm | Fabio confirms: "It was a hackaton yesterday"                |
| 2026-03-25 evening | Full digital investigation conducted                         |
| 2026-03-25 evening | Source code found on GitHub (github.com/itsfabioroma/impossibl) |
| 2026-03-25 evening | Portal completed - registered as impossibl[0][22]             |


Note: The poster may have been placed on March 24 (day of hackathon) and only noticed on March 25.

---

## Remaining Unknowns

1. **PGP key notation `manu=2,2.5+1.12,0,3`** - Embedded in the key signature, not referenced anywhere in the source code.
2. **r/impossibl content** - The subreddit was banned. The puzzle flow doesn't require it (goes straight to `/portal`), so it may have been a secondary channel.

---

## Conclusion

The Impossibl puzzle is a marketing ARG disguised as something deeper, built to promote a hackathon run by UltraContext in San Francisco. It copies the Cicada 3301 playbook wholesale - physical posters, layered encoding, philosophical theming, progressive filtering - without any of the substance that made the original meaningful.

The puzzle has a honeypot and some multi-layer encoding, but the technical difficulty is shallow. Hex and base64 are not cryptography. Reading bundled JavaScript is not reverse engineering. The combination of a poster near a train station, a hidden API endpoint, and a PGP key creates the appearance of mystery, but it dissolves the moment you realize it's all in service of hackathon sign-ups.

It's a Cicada 3301 costume worn by a marketing campaign.