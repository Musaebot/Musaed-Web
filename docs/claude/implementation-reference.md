# Implementation reference

Folded in from `README.md` when that file was rewritten into a short human-facing page
(2026-08-25) — this is the detail an AI session working on the CSS/JS/markup needs that
didn't already live above. No history or reasoning trimmed here beyond what README itself
had already gone stale on (checked against the live markup, not copied blind).

## Running it locally

No build step, nothing to install. `file://` blocks the self-hosted fonts, so serve over
HTTP: `python -m http.server 8000` or `npx serve .`, then open `localhost:8000`. Deploys to
any static host (Cloudflare Pages, Netlify, Vercel, Railway's staticfile provider) with zero
config — no build command, output directory is the repo root.

## `index.html` section anchors

| Anchor | Heading | Purpose |
| --- | --- | --- |
| (hero) | سيرفرك مرتب، وانت مرتاح | value prop, primary CTA |
| `#features` | كل اللي يحتاجه سيرفرك | the eight systems, bento grid of 10 cards |
| `#commands` | كل الأوامر | curated command list, 7 groups / 16 rows |
| `#dashboard-features` | ميزات تُدار من الداشبورد | the 3 systems with no slash commands |
| `#stats` | مساعد بالأرقام | public counters (mock) — **`hidden`**, `docs/claude/page-notes.md` |
| `#trust` | بيانات سيرفرك تبقى لسيرفرك | the three product guarantees |
| `#about` | مبني لمجتمعات عربية | about the **bot**: Arabic-first |
| `#about-us` | من نحن | about the **project and people**, plus the community server |

`#about` and `#trust` used to overlap (`#about` asserted tenant isolation in one clause);
that claim now lives only in `#trust`, where it has room to say *how*. Don't put it back in
both. `#about` and `#about-us` are intentionally separate — product vs. people — and
`#about-us` is written in neutral project voice with no names, dates, or team size on
purpose; that's the section to edit if it should read as a solo maintainer.

## The phone menu

Below 768px the nav links become a panel under the bar, opened by `.nav__toggle`. It's a
plain disclosure, not a modal — no focus trap, no scroll lock, no overlay.

- **DOM order is brand, button, panel, CTA, login icon.** The toggle button sits immediately
  before the panel it controls, so tabbing out of it lands in the menu with no focus
  management needed, and DOM/visual/focus order agree at both breakpoints. No `order`
  property anywhere.
- **The panel is hidden with `visibility`, not `opacity` alone** — that's what keeps its
  links out of the tab order and accessibility tree while closed.
- **Mobile is the base, desktop is the override.** `.nav__links` defaults to the dropped
  panel; the `min-width: 768px` block turns it back into a row.
- **Opening the menu pins the bar solid.** `nav-solidify` (§ below) leaves the bar at 35%
  opacity until 140px of scroll, and a menu hanging off a see-through bar looks broken.
  `.nav.is-open { animation: none }` releases it — a plain declaration beating a running
  animation without `!important`.
- **Without JS the toggle button is hidden**, since it couldn't do anything. The footer is
  the fallback and must carry every nav destination (`docs/claude/design-and-invariants.md`)
  — add a link to both or no-JS phones lose it.

## Motion: scroll-driven effects

Layer 1 (baseline) is the `IntersectionObserver`-driven `.reveal` staggering already covered
in `docs/claude/design-and-invariants.md`. Layer 2 is progressive enhancement: native CSS scroll-driven timelines in one
`@supports (animation-timeline: view())` block at the bottom of `styles.css`, skipped
entirely outside Chromium 115+ / Safari 26+ (layer 1 carries the page fine on its own there).

| Effect | Timeline | What it does |
| --- | --- | --- |
| `progress-grow` | `scroll(root)` | read-progress line under the nav, grows from the right |
| `nav-solidify` | `scroll(root)` | nav fades from near-transparent to solid over 140px |
| `hero-recede` | `view()` | hero copy recedes as the hero exits |
| `hero-drift` | `view()` | wordmark drifts slower than the copy, separating the layers |
| `glow-drift` | `scroll(root)` | hero glow parallaxes over the first viewport height |

Two constraints: don't scrub body copy or cards (scrubbed text fades back out as you scroll
up, which is distracting to read against — scrubbing is for effects where being tied to
scroll position is the actual point), and this is also why `hero-drift` targets `.wordmark`
specifically rather than its `.reveal` parent `.hero__visual` (`docs/claude/design-and-invariants.md`'s reveal/scroll-link
conflict rule).

## Brand assets and the Discord mark

| File | Size | Used as |
| --- | --- | --- |
| `musaed-avatar.png` | 1024×1024 | brand mark in nav/footer (30px) and legal-page bars (25px); also `apple-touch-icon` |
| `musaed-favicon.png` | 512×512 | `rel="icon"` on every page |
| `musaed-banner.png` | 960×540 | the `og:image`/Twitter card image on every page. `1200×630` is the size every platform actually optimises for — worth regenerating at that size, but not currently a placeholder |

Both marks are fully transparent outside the glyph. Don't scale past ~50px, where upscaling
starts to show.

`assets/Pics/Discord-Icon.png` (100×100, transparent) renders in the 5 places
`docs/claude/design-and-invariants.md` counts, all on `index.html`:

| Where | Class | Size |
| --- | --- | --- |
| `ضيف البوت` CTA, nav | `.btn__logo` in `.btn--sm` | 16px |
| `ضيف البوت` CTA, hero and about | `.btn__logo` | 18px |
| `ادخل سيرفرنا` CTA, community panel | `.btn__logo` | 18px |
| `سيرفر مساعد` community panel icon | `.community__logo` | 44px |

White via `filter: brightness(0) invert(1)` on the blurple CTAs (Discord's own treatment on
a blurple field); left at natural colour on the dark community panel, which clears 7.2:1
there unaided.

**Why two accent colours at all.** The primary CTAs are Discord blurple because every one of
them leads to Discord (three `ضيف البوت` + `ادخل سيرفرنا`) — colouring by destination reads
as "this goes to Discord," the same convention as a "Sign in with Google" button carrying
Google's colours. `#5a63d8` is deliberately deeper/less saturated than Discord's own
`#5865f2` (not an exact brand match), white label at 4.99:1, rising to 6.05:1 on the
`#4e56c6` hover fill. Everything else stays green — see `docs/claude/design-and-invariants.md`
for the scoping rule (`.btn--primary` only) and the current `var(--accent)` usage count.

## The stats placeholder, in implementation terms

The three values live in one `STATS` object at the top of `assets/js/main.js`:

| Key | Kind | Rendered as | Section label |
| --- | --- | --- | --- |
| `servers` | `int` | grouped, e.g. `1,284` | سيرفر يشغّل مساعد |
| `members` | `int` | grouped, e.g. `418,930` | عضو داخل هالسيرفرات |
| `uptime` | `percent` | one decimal, e.g. `99.4%` | نسبة التشغيل خلال 30 يوم |

To wire a real endpoint: set `var STATS_ENDPOINT = "https://<your-public-api>/v1/stats";`
(currently `null`, `docs/claude/placeholders-and-domain.md`) and adjust `shapeStats()` to match the payload's field names — nothing
else needs editing. The rate-limiting path already written around it:

| | Behaviour | Constant |
| --- | --- | --- |
| Throttle | at most one request per tab per window; repeats/reloads inside it serve from cache | `STATS_TTL_MS`, 5 min |
| Backoff | a 429 or failure parks further requests, honouring `Retry-After` when sent | `STATS_BACKOFF_MS`, 15 min |
| Fallback | any failure keeps the last good values, or the placeholders if there was never a good response | |

`readStats()` always resolves, never rejects. The cache is `sessionStorage` under
`musaed:stats` — aggregate counts only, no identifiers, cleared when the tab closes, every
access wrapped in `try/catch` since private-mode browsers can throw (throttle degrades to
per-pageload if storage is unavailable rather than breaking).

## Vendored fonts

IBM Plex Sans Arabic and IBM Plex Mono, SIL Open Font License 1.1, subset `.woff2` files
taken from Fontsource. Icon sourcing is covered in `docs/claude/design-and-invariants.md` —
same file, don't duplicate the process description here.
