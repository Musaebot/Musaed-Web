# How to verify — do not reason about CSS, measure it

Reading the stylesheet has produced wrong conclusions repeatedly in this project. Every
layout, contrast, and sizing claim below was wrong at least once before being measured.

```bash
python -m http.server 8000     # serve; file:// blocks the self-hosted fonts
```

**Headless Edge does not work.** Edge 150 turned `Application/msedge.exe` into a stub that
spawns the real binary under `EdgeCore/<version>/` and exits, so a launcher sees the process
die instantly (`Code: 0`, empty stder r). Pointing at the `EdgeCore` binary directly still
produces no headless output in any mode. Use a standalone shell instead, installed **outside
this repo**:

```bash
npx @puppeteer/browsers install chrome-headless-shell@stable
```

The suite that currently passes lives outside the repo. Recreate it to assert:

- **Per page × width** (320, 375, 390, 412, 768, 860, 900, 1024, 1440 — all three pages): no
  horizontal overflow (`scrollWidth <= clientWidth`); no interactive element under 24×24; no
  unnamed link or button; no console or network errors. **Apply the WCAG 2.5.8 inline
  exemption** — a link inside a sentence is size-constrained by line-height and is not a
  failure.
- **Structure** (`index.html`): 22 sprite symbols with none orphaned; 6 tabs and 6 panels;
  exactly one panel visible on load, and it is `#top`; 16 command rows; 4 filter chips; 6 FAQ
  items; 9 cards in `#features`; 3 in `#trust`; `#dashboard-features` with exactly 3 `<li>`s
  and exactly 1 outbound link; every `.cmd__name` computes `direction: ltr` and starts with
  `/`, with no exceptions.
- **Tab routing**: a tab click swaps the visible panel, pushes the hash, moves
  `aria-current="page"`, and updates the topbar crumb; browser Back returns to the previous
  panel; an in-page CTA (`.hero__actions a[href="#features"]`) routes through the same
  handler so the tabs can never disagree with what is on screen.
- **Deep links, at 390px and 1440px** — `#top`, `#features`, `#commands`, `#trust`, `#faq`,
  `#about-us`, plus the two nested anchors `#about` and `#dashboard-features`: each opens the
  right panel, and **a panel deep link lands at `scrollY === 0`** while a nested one scrolls
  past zero to its element. That scroll assertion is the one that caught a real bug — see the
  `scroll-behavior` trap below.
- **Filtering**: all=16, mod=5, setup=8, info=3, the three summing to 16, and exactly one
  chip carrying `aria-pressed="true"`.
- **No-JS** (`page.setJavaScriptEnabled(false)`): all 6 panels rendered, all 16 command rows
  rendered, filter chips `display: none`, the menu toggle `display: none`, `.reveal` elements
  at `opacity: 1`, and `.foot__nav` carrying all six tab hrefs — that last one is the
  assertion that stops a new tab from silently stranding no-JS phones.
- **connect.html is gone**: `/connect.html` returns 404; no `a[href*="connect.html"]` remains
  on the site; exactly 2 links point at `dashboard.musaed.dev/auth/login` and 5 at the
  dashboard host in total.
- **Menu behaviour** (320/375/412): `.side` computes `position: sticky`; `.topbar` is
  `display: none`; the toggle is shown; the menu starts closed with `aria-expanded="false"`
  and `visibility: hidden`; the panel's invite is `display: none` so the invite is not
  duplicated on one screen. Then: the closed menu exposes **0** links and the open one **10**
  (6 tabs + لوحة التحكم + 3 legal); a click opens it and the panel sits flush under the bar
  and inside the viewport; Escape closes it and returns focus to the toggle; a link click
  closes it *and* navigates; an outside tap closes it; crossing 900px with it open clears
  both `aria-expanded` and `is-open`.
- **Desktop chrome** (1440): `.side` sticky and exactly 256px; `.topbar` shown;
  `.side__actions` hidden; `.side__menu` `visibility: visible`; every tab ≥44px tall.
- **FAQ**: starts fully closed; opening a second item closes the first (native
  `<details name>` exclusive accordion).
- **Contrast**: for every unique colour+size+background combination on the page, composite
  each translucent layer over what is behind it up to the page background, then require AA
  (4.5:1, or 3:1 for large text). **Compositing is the whole point** — judging a label on
  `--accent-soft` (9% alpha) against the panel colour alone reports 1:1 and judging it
  against `--bg` alone reports a value it never actually has.

**231 assertions, all passing — rerun 2026-08-25** in headless Chrome 152 against
`python -m http.server`, covering the three pages × nine widths plus everything above. This
supersedes the old 154-assertion suite entirely: that one asserted on `.bento` and
`.cmdgroup`, neither of which exists after the rebuild. Its 17 menu assertions were stale
since 2026-08-05; the phone menu is asserted again here, and this time the run is real.

**Worst measured contrast on the page is 4.99:1** (white on `--discord`). Every grey clears
comfortably: `--text-dim` 7.4:1, `--text-mute` 6.1:1, `--text-faint` 5.3:1 on `--bg`. The
mockup's faintest grey (`#69726C`, used for footer and sidebar legal links at 12px) measured
**3.92:1** and was replaced rather than shipped — that was the one accessibility defect
carried in from the design file.

## Traps found by measuring, not reading

**`scroll-behavior: smooth` on `html` breaks deep links into a tabbed page.** On a cold load
of `/#faq` the browser performs its own scroll to the fragment target *after* the deferred
script has already run and reset the scroll position — and with a global smooth scroll that
browser scroll is a ~150ms animation, so it starts late and wins. The panel lands with its
heading under the sticky bar (measured: `scrollY` 116 at 390px, 58 at 1440px). Removing the
global smooth scroll makes the browser's jump instant; the router still asks for smooth
explicitly on nested anchors. Even then one reset is not enough — see `resetScroll(persist)`
in `docs/claude/implementation-reference.md`.

**A grid item's automatic minimum is its min-content width, and a scrolling child does not
save you.** `.app { grid-template-columns: 1fr }` sized the column to the six-tab nav's 468px
min-content width and pushed every phone width sideways — even though `.tabs` carried its own
`overflow-x: auto` at the time. `minmax(0, 1fr)` on the track plus `min-width: 0` on `.side`
is the fix, and both are still there because any wide child would do the same thing again.
This is invisible in the stylesheet and only shows up as `scrollWidth 468 > clientWidth 375`.

**An `IntersectionObserver` on a node inside a `hidden` ancestor never fires**, which is why
`refreshReveals()` re-observes a panel's nodes when it opens rather than trusting the
original `observe()` call. The same property is what makes `#stats` harmless while hidden.

**The bento grid and its hand-placed 12-column spans are gone as of 2026-08-25.** It used to
tile 10 cards into rows of 7+5, 5+7, 6+6, 4+4+4, 12, and because 9 regular cards is an odd
number the arrangement had a parity problem that came back every time a card was added or
removed — the note that used to sit here existed to warn about exactly that. The rebuild's
`.grid` is `repeat(auto-fit, minmax(min(100%, 270px), 1fr))`, so cards tile themselves at
every width and there is nothing left to re-measure when one is added. Card counts still
matter for accuracy (`docs/claude/copy-accuracy.md`); card *placement* no longer does.

Use `page.accessibility.snapshot({ root })` for accessible names — `innerText` returns `""`
for `visibility: hidden` elements and will produce false "unnamed control" failures. Root the
snapshot at the element under test; a whole-document name search cannot distinguish a nav
link from a footer link with the same label.

## Traps that have already cost time

- **Reveal stagger vs. screenshots.** `main.js` staggers `--d` by 70ms per sibling inside a
  `data-stagger` container, capped at the 9th, so the last card in `#features` waits 560ms
  before its 620ms fade *starts*. A short fixed wait catches late
  elements mid-transition and makes a correct layout look like missing content. Inject
  `.reveal{opacity:1!important;transform:none!important;transition:none!important}` instead
  of waiting it out.
- **`fullPage` screenshots used to clip** on pages with a `position: fixed` backdrop —
  `updates.html` had one (`.upbg`), the only page that ever did. It was removed 2026-08-25
  (`docs/claude/page-notes.md`) and no current page uses `position: fixed` for a backdrop, so this trap no longer
  applies anywhere on the site. Kept here in case a future page reintroduces the pattern: the
  geometry was fine even when it clipped — measure `getBoundingClientRect()` before believing
  a `fullPage` screenshot, and prefer viewport or element screenshots on a fixed-backdrop page.
- **Element screenshots include the sticky nav** painted across the middle. That is an
  artifact, not a defect.
- **A section taller than the viewport will not fully reveal** from a single `scrollIntoView`.
  Scroll through it in steps; reveals are one-shot and accumulate.
- **The Windows console is cp1252.** Printing Arabic raises `UnicodeEncodeError` — run
  Python with `PYTHONIOENCODING=utf-8`.
- **`grep` counts are inflated by your own explanatory comments.** Strip comments before
  counting markup, or you will "find" elements that only exist in a code comment.
- **The user edits files live in the IDE while you work.** A fragment that looks broken may
  be a half-typed sentence. Check `git diff` before deleting anything you did not write.
- **Git reports CRLF warnings on every write.** Harmless; not a failure.
- **`404.html` only works through Railway's real Caddy server, not a local static server.**
  `python -m http.server` (the tool this file's own "How to verify" section above tells you to test with) has no concept
  of a custom error page and will show its own plain 404 for a missing path — that is not a
  regression, it just isn't testing the mechanism that matters. The actual wiring is Railpack's
  staticfile provider's *default* Caddyfile (nothing in this repo — confirmed from
  `github.com/railwayapp/railpack`'s own template, not guessed): `handle_errors { rewrite *
  /{err.status_code}.html }`. That single line is why a file named exactly `404.html` at the
  root is the entire fix — no Caddyfile override needed, and the only way to actually verify
  it is against the live Railway URL.
- **`musaed.dev` sits behind Cloudflare, and Cloudflare edge-caches CSS/JS but not HTML —
  confirmed 2026-08-11 via response headers, not guessed.** `index.html` always comes back
  `cf-cache-status: DYNAMIC` (fresh every request); `assets/css/styles.css` and
  `assets/js/main.js` come back `cf-cache-status: HIT` with `Cache-Control: max-age=14400` (4
  hours), and a Railway redeploy does **not** purge that cache. Caught in the wild twice: a
  CSS-only change (new `.dashfeatures`/`.cmdgroup__note` rules) deployed successfully while
  `musaed.dev` kept serving CSS from over a day earlier, so the same page rendered with full
  styling on one host and as a plain unstyled list on the other; and again 2026-08-17, when a
  10th bento card shipped against cached CSS that only styled 9, leaving the full-width
  durations card auto-placed into a single 1/12 column on the live domain until the entry
  expired.

  **To read past the cache, add a cache-busting query string — do not go looking for a second
  host.** Verified 2026-08-18: the plain URL returned a week-old file while this returned the
  copy Railway had just built.

  ```bash
  curl -sS -H "Cache-Control: no-cache" "https://musaed.dev/assets/css/styles.css?probe=$RANDOM"
  curl -sSI "https://musaed.dev/assets/css/styles.css" | grep -iE "cf-cache-status|age:|last-modified"
  ```

  `last-modified` is the reliable tell: compare it against the deploy time. `cf-cache-status:
  HIT` with a stale `last-modified` is the failure; `MISS` means the edge just refetched.
  A distinct `?probe=` value each time matters — reusing one caches *that* URL too.

  **This used to say "check `musaed.up.railway.app`, no CDN in front". Don't.** The generated
  Railway host is no longer part of how this site is served or verified (`docs/claude/placeholders-and-domain.md`), and the query
  string above needs no second host anyway. Purging properly still has to happen from
  Cloudflare's dashboard (Caching → Configuration → Purge Cache), which no git push or Railway
  action can reach — but a stale entry does expire on its own inside the 4 hours, which is how
  the 2026-08-17 case resolved. HTML-only changes (like the `#stats` `hidden` attribute,
  `docs/claude/page-notes.md`) are unaffected, which is why that change worked immediately
  and these didn't.
