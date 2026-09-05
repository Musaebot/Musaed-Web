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

## `index.html` panels and anchors

**Rebuilt 2026-08-25 from a design mockup.** The page was one long scroll with anchor
sections; it is now a sidebar (a sticky header on phones) plus tab panels, only one of which
is rendered at a time. Nothing was dropped — two of the old sections now live *inside* a
panel rather than owning one. **Eight panels as of 2026-09-05** (`#pricing` joined the
original six on 2026-09-04, `#why-musaed` on 2026-09-05).

Every panel is present in the HTML and nothing is hidden until `main.js` runs, so a crawler
and a no-JS browser get the whole document. The tabs are `<a href="#panel-id">`, not buttons,
which is what keeps the old URLs working in both modes.

| Tab | Panel id | Contains |
| --- | --- | --- |
| البداية | `#top` | hero + 3 facts, 4 quick cards, `#about`, `#stats` (`hidden`) |
| الأنظمة | `#features` | the eight systems, 9 cards + the durations call-out |
| الأوامر | `#commands` | filter chips, 16 command rows, `#dashboard-features` |
| الأسعار | `#pricing` | 2 plan cards + a compare table — **new 2026-09-04**, numbers-only, no self-serve billing |
| الأمان | `#trust` | the three product guarantees |
| أسئلة شائعة | `#faq` | 6 `<details>` items — **new in this rebuild** |
| ليش مساعد؟ | `#why-musaed` | competitor comparison — **new 2026-09-05**, a `.filters`-style switcher (MEE6 / Dyno) driven by `initVersus()`; 2 `.vs` blocks |
| من نحن | `#about-us` | the project and people, plus the community server |

`#about` (مبني لمجتمعات عربية) and `#dashboard-features` are **nested anchors**: they have no
tab of their own, and hitting `/#about` opens the panel that contains it and then scrolls to
the element. `#about` and `#trust` used to overlap (`#about` asserted tenant isolation in one
clause); that claim now lives only in `#trust`, where it has room to say *how*. Don't put it
back in both. `#about` and `#about-us` are intentionally separate — product vs. people — and
`#about-us` is written in neutral project voice with no names, dates, or team size on
purpose; that's the section to edit if it should read as a solo maintainer.

## The tab router

`initTabs()` in `main.js`. Roughly 120 lines and worth reading before changing any of it.

- **A tab click is intercepted, not followed.** The handler `preventDefault`s, `pushState`s
  the hash, and swaps which panel carries `hidden`. `popstate` and `hashchange` both route
  back through `syncFromHash()`, so back/forward and a pasted URL behave identically.
- **A delegated document-level handler catches every other `a[href^="#"]`** — the hero CTA,
  the brand mark, footer links. Without it an in-page link could leave the page showing a
  panel the tabs disagree with.
- **`scroll-behavior: smooth` is deliberately NOT set on `html`.** A global smooth scroll
  turns the browser's own fragment jump on a cold `/#faq` load into a ~150ms animation that
  *starts after* the router has already reset the scroll position — and wins, landing the
  panel with its heading tucked under the sticky bar. Measured, not theorised. The router
  asks for smooth explicitly where it wants it (nested anchors).
- **The initial route re-asserts `scrollTo(0, 0)` for six frames and again at `load`**
  (`resetScroll(persist)`). Even with the instant fragment jump the browser can scroll after
  the script runs; one reset is not enough. Later resets, from tab clicks, have no such
  competition and use the single call.
- **Panels get `tabindex="-1"` and focus on click**, so the next Tab press resumes inside
  what was just opened rather than at a control that scrolled away.

## The phone menu

Below 900px the tabs live in a panel under the bar, opened by `.side__toggle`. It's a plain
disclosure, not a modal — no focus trap, no scroll lock, no overlay. `initMenu()` in
`main.js`.

**The first cut of the rebuild used a horizontally scrolling tab strip instead**, and it was
replaced with the hamburger on the same day, at the owner's request. If you find a
`keepTabVisible()` reference anywhere, it belonged to that strip and is gone.

- **DOM order is brand, invite, toggle, panel.** The toggle sits immediately before the panel
  it controls, so tabbing out of it lands in the menu with no focus management needed, and
  DOM/visual/focus order agree at both breakpoints. No `order` property anywhere.
- **The panel is hidden with `visibility`, not `opacity` alone** — that's what keeps its
  links out of the tab order and accessibility tree while closed. The `visibility` transition
  is delayed by the fade duration on close and zero on open, so it fades out rather than
  vanishing.
- **Mobile is the base, desktop is the override.** `.side__menu` defaults to the dropped
  panel; the `min-width: 900px` block turns it back into the sidebar's body, with the toggle
  and the header invite hidden.
- **The invite is not duplicated on one screen.** `.side__invite` (header) shows only below
  900px; `.side__foot-invite` (in the panel) is `display: none` there and only appears in the
  desktop sidebar. Five `oauth2/authorize` links in the DOM (the fifth is the `#pricing`
  bottom CTA), never more than four visible at once.
- **Without JS the toggle is hidden**, since it couldn't do anything, and the footer is the
  fallback — `.foot__nav` carries all eight tab destinations (six before `#pricing` 2026-09-04,
  seven before `#why-musaed` 2026-09-05) for exactly this case. Add a tab and you must add a
  footer link, or no-JS phones lose it.
- **Escape closes and returns focus to the toggle; an outside tap closes; crossing 900px with
  it open clears `is-open`** — otherwise a rotation leaves the class set on a sidebar that no
  longer has a panel.
- **The panel caps at `min(70dvh, 560px)` and scrolls internally**, so a short phone in
  landscape cannot end up with a menu taller than the screen.

## The command filter and the versus switch

Two near-identical toggles, both reusing `.filters` / `.filter` and both hidden by
`.no-js .filters` because everything they toggle is rendered anyway.

- **`initFilters()`** — the `#commands` category chips. `[data-filter]` buttons show/hide
  `[data-cat]` rows inside `[data-cmds]`; `[data-cmds-empty]` appears if a category is empty.
- **`initVersus()`** — the `#why-musaed` bot switcher (MEE6, Dyno). `[data-vs]` buttons toggle
  `hidden` on `[data-vs-panel]` blocks (`.vs`), one visible at a time; the first button's
  target is applied on load. No `.reveal` inside the `.vs` blocks — they're shown by a click
  after the reveal observer has already passed them, so an animated-in block would stay at
  `opacity: 0`. `refreshReveals()` only runs on *panel* activation, not on a versus switch.
  Adding a third comparison (e.g. an Arabic rival) is one more button + one more `.vs` block,
  no JS change.

Neither function touches the tab router. Adding a third toggle of this shape means a third
`initX()` and a matching `.no-js .filters` group — do not fold them into one generic helper
unless all three genuinely share behaviour.

## Phone layout, generally

- **`.app` uses `minmax(0, 1fr)`, never `1fr`.** A grid item's automatic minimum is its
  min-content width, so the nav sized the whole column to 468px and pushed the page sideways
  at every phone width. `.side` also carries `min-width: 0` for the same reason. Measured at
  320/375/390/412px, not guessed. This bit with the scrolling tab strip and would bite again
  with any wide child.
- **The desktop `.topbar` is `display: none` on phones**; the header carries the invite and
  the panel carries the dashboard link.
- **Command rows stack below 700px.** The three-column grid
  (`minmax(140px, 175px) 1fr auto`) only earns its columns when the description still has
  room beside a fixed-width name; `.cmd`'s base state is a single column.
- **Without JS the filter chips are hidden** (`.no-js .filters`), since all sixteen rows are
  rendered anyway and the controls could not do anything. The same rule hides the
  `#why-musaed` bot switcher (also `.filters`); its three `.vs` blocks then stack.
- **320px still fits brand + invite + toggle on one row**, but only because `.side__invite`
  sheds padding below 900px and again below 360px. That's the tightest thing on the page.

## Motion

Layer 1 (baseline) is the `IntersectionObserver`-driven `.reveal` staggering already covered
in `docs/claude/design-and-invariants.md`. Layer 2 is progressive enhancement: native CSS
scroll-driven timelines in one `@supports (animation-timeline: view())` block at the bottom
of `styles.css`, skipped entirely outside Chromium 115+ / Safari 26+ (layer 1 carries the
page fine on its own there).

| Effect | Timeline | What it does |
| --- | --- | --- |
| `hero-glow` | (time) | the hero's radial glow breathes — not scroll-linked |
| `glow-drift` | `scroll(root)` | hero glow parallaxes over the first viewport height |
| `caret-blink` | (time) | terminal caret beside the sidebar wordmark |
| `status-pulse` | (time) | the "البوت شغّال" dot in the desktop topbar |
| `faq-in` | (time) | FAQ answer fades in when its `<details>` opens |

**This list shrank from five scroll-driven effects to one on 2026-08-25**, because four of
them targeted chrome the rebuild removed: `progress-grow` and `nav-solidify` belonged to the
old sticky nav bar (there is no read-progress line and no translucent-until-scrolled bar any
more), and `hero-recede`/`hero-drift` scrubbed the hero copy and the `.wordmark` block as the
hero exited — but panels are now roughly one screen and the hero no longer exits under a
following section. `glow-drift` survives because it still has a viewport-height of scroll to
work against. Note it composes with `hero-glow` on the same element via a two-value
`animation-timeline: auto, scroll(root block)`.

Two constraints still hold: don't scrub body copy or cards (scrubbed text fades back out as
you scroll up, which is distracting to read against — scrubbing is for effects where being
tied to scroll position is the actual point), and never put a scroll-linked animation on an
element that also carries `.reveal` — they fight over `transform`
(`docs/claude/design-and-invariants.md`).

## Brand assets and the Discord mark

| File | Size | Used as |
| --- | --- | --- |
| `musaed-avatar.png` | 1024×1024 | brand mark in nav/footer (30px) and legal-page bars (25px); also `apple-touch-icon` |
| `musaed-favicon.png` | 512×512 | `rel="icon"` on every page |
| `musaed-banner.png` | 960×540 | the `og:image`/Twitter card image on every page. `1200×630` is the size every platform actually optimises for — worth regenerating at that size, but not currently a placeholder |

Both marks are fully transparent outside the glyph. Don't scale past ~50px, where upscaling
starts to show.

`assets/Pics/Discord-Icon.png` (100×100, transparent) renders in the **6** places
`docs/claude/design-and-invariants.md` counts, all on `index.html`:

| Where | Class | Size |
| --- | --- | --- |
| `ضيف البوت` CTA, sidebar footer (`.side__foot-invite`, desktop only) | `.btn__logo` in `.btn--sm` | 16px |
| `ضيف البوت` CTA, phone header (`.side__invite`, phone only) | `.btn__logo` in `.btn--sm` | 16px |
| `ضيف البوت` CTA, desktop topbar | `.btn__logo` in `.btn--sm` | 16px |
| `ضيف البوت` CTA, hero | `.btn__logo` | 18px |
| `ادخل سيرفرنا` CTA, community panel | `.btn__logo` in `.btn--sm` | 16px |
| `سيرفر مساعد` community panel icon | `.community__logo` | 44px |

**It was 5 before the 2026-08-25 rebuild**; the extra one is the phone-header invite.
`.side__foot-invite` and `.side__invite` are mutually exclusive by media query, as are
`.side__invite` and the topbar's, so a visitor sees **3 invite buttons on desktop** (sidebar,
topbar, hero) and **2 on a phone** (header, hero) — six marks in the DOM, never six on
screen. If you add a fourth invite, check it does not become a second one visible at the
same width.

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
