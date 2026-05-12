# ChoreBoard Landing

Marketing site for **ChoreBoard** — a family-dashboard SaaS where members claim
and complete household chores in a shared real-time view.

The landing page is itself a Kanban board. Visitors drag feature cards from
`Available` into persona swimlanes (**Parents · Kids · Family TV**) to read the
pitch from each angle, earn XP, and unlock the Sunday-payout signup
celebration. Tapping a card auto-claims it. Everything also has a passive
scroll path so a reader who never interacts still gets the full pitch.

## What's in this repo

```
index.html       The entire landing page. CSS and JS inlined.
README.md        This file.
.gitignore       OS / editor noise.
```

There is no build step. `index.html` is the deliverable.

## Run locally

Open it directly in a browser, or serve it statically:

```bash
# Pick one
python -m http.server 8000
npx serve .
npx http-server -p 8000
```

Then visit `http://localhost:8000/`.

## Configure

Override these by setting globals on `window` **before** the inline script runs
(e.g. via a tiny `<script>` injected at the top of the file by your CDN):

| Global                          | Effect                                                                                  |
| ------------------------------- | --------------------------------------------------------------------------------------- |
| `window.CHOREBOARD_APP_URL`     | URL the "Sign in" links target. Defaults to `https://app.choreboard.io`. The native iOS / Android apps (when published) deep-link from this URL via Universal Links / App Links. |
| `window.CHOREBOARD_WAITLIST_URL`| If set, the waitlist form `POST`s `{ email, family }` to this URL on submit (in addition to writing to `localStorage`). |

## Sections

1. **Pitch** — headline, two CTAs, and a small live demo card that cycles
   `available → claimed → pending → approved` every ~2s while a kid's tally
   ticks up. The metaphor explains itself before anyone scrolls.
2. **Board** — the interactive Kanban. `Available` (dark) on the left holding
   10 feature cards, three persona lanes in the middle (`Parents · Kids ·
   Family TV` with blue / orange / green accents), `Approved` on the right.
   Drag, tap, or hit **Auto-play tour**. A dark stat tile tracks XP / level
   using the same curve as the in-app gamification (spec §7).
3. **Family TV** — the kitchen-wall ambient screen. Money jar SVG that fills
   based on the visitor's demo XP, sample leaderboard with crown bob, badge
   tile, streak tile. Same visual language as the in-app Family Dashboard.
4. **Champion** — Sunday-payout reveal. Confetti fires when the visitor has
   filed ≥3 cards. Waitlist email + family name capture, plus a "have an
   invite, sign in" link.

## Why a single Kanban as the marketing page

A normal SaaS landing page hides its product behind a screenshot. ChoreBoard's
whole thesis — "everyone in the family sees the same board, anyone can grab a
chore, the dollar lands when a parent approves" — is a Kanban interaction. The
landing makes you do that interaction once before you sign up. Reading the
pitch from the Parents lane is structurally different from reading it from the
Kids lane, and the page rewards switching with XP, the same way the product
rewards completing a real chore.

Non-annoyance affordances baked in:

- **Tap** a card to auto-claim it into the best lane. No drag required.
- **Auto-play tour** walks the entire board on a ~2-second tick.
- **Replay** resets the demo if a returning visitor wants to start over.
- **Keyboard**: Tab to a card, Enter to auto-claim, `R` to reset.
- **Pointer Events** unify mouse + pen + touch in a single drag handler.
- **Persisted state** so a refresh / return visit picks up where you left.
- **Confetti only once per session** so the celebration never spams.

## Deploying

Production hostname is **`https://choreboard.io`**, fronted by Cloudflare and
served from Cloudflare Pages. The web app lives at `https://app.choreboard.io`
and the API at `https://api.choreboard.io` (both on the same Render service);
this site links out to `app.` for sign-in and embeds links to the App Store
and Play Store once those listings go live.

Drop `index.html` on any static host: Cloudflare Pages (preferred), Netlify,
GitHub Pages, Render Static Site, Vercel, an S3 bucket, or behind Fastify's
`static` plugin sitting next to `ChoreBoard-Api`.

For GitHub Pages with a custom domain, drop a `CNAME` file in next to
`index.html`. For Cloudflare Pages, set the custom domain in the Pages project
settings — no `CNAME` file needed.

### App Store / Play Store deep-link files

Once the native apps are submitted, the following files need to be served from
**`https://app.choreboard.io`** (not from this site) so iOS Universal Links
and Android App Links open the app instead of the browser:

- `https://app.choreboard.io/.well-known/apple-app-site-association`
- `https://app.choreboard.io/.well-known/assetlinks.json`

Those are served by the API process (`ChoreBoard-Api`), not by this static
site. The marketing pages here just link to `https://app.choreboard.io` and
let the OS resolve.
