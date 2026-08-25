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

- **Per page × width** (320, 375, 390, 412, 768, 1024, 1440 — all four pages): no horizontal
  overflow (`scrollWidth <= clientWidth`); no interactive element under 24×24; no unnamed
  link or button; no console or network errors. **Apply the WCAG 2.5.8 inline exemption** —
  a link inside a sentence is size-constrained by line-height and is not a failure.
- **Nav**: 4 links on one line at ≥768px, each ≥44×44, and report the slack.
- **Menu behaviour** (375px): starts closed; click opens; panel sits flush under the bar;
  Escape closes and returns focus to the button; Tab from the button enters the menu; a link
  click closes it *and* navigates; outside tap closes; crossing 768px closes it; the closed
  nav exposes **2** links to the a11y tree and the open one **6** (both figures unverified —
  written from the current markup, not a rerun; `docs/claude/design-and-invariants.md` explains why).
- **Structure**: 10 bento cards in 5 rows; 22 sprite symbols with none orphaned; 7 command
  groups / 16 rows, each group either carrying a permission pill or a `.cmdgroup__note` (or
  both) — `الترحيب`, `التحقق بالكابتشا`, `تذاكر الدعم` have both; every `.cmd__name` computes
  `direction: ltr` and starts with `/`, with no exceptions as of 2026-08-11 (the one
  Arabic-named row, `/اختصار`, is gone — `docs/claude/copy-accuracy.md`,
  `docs/claude/design-and-invariants.md`); 1 `#dashboard-features` section with exactly
  3 `<li>`s and exactly 1 outbound link; 3 guarantee panels.

**154 assertions, all passing — rerun 2026-08-17** in headless Chrome 152 against
`python -m http.server`, covering all 5 pages *(now 4 — `updates.html` was removed
2026-08-25, `docs/claude/page-notes.md`; this count is stale by one page and hasn't been
rerun since)* × 9 widths
(320, 375, 390, 412, 768, 860, 900, 1024, 1440) plus the structure and bento-row checks
above. 860 and 900 are new and were added
for a reason: 860px is where `.bento` becomes a 12-column grid, so it is the width at which
the narrowest card is narrowest, and nothing was measuring it. **This run did not include the
17 menu assertions** — the 375px open/close/Escape/focus behaviour is still unverified since
2026-08-05, so treat that 17 as stale even though the 128 is now superseded.

**The bento grid has one row of three since 2026-08-17, and it is load-bearing.** Adding the
auto-responses card (`docs/claude/copy-accuracy.md`) made it 10 cards — 9 regular plus the full-width `card--full`
durations card — and 9 is odd, so the four-rows-of-two arrangement could not absorb it. Rows
are now 7+5, 5+7, 6+6, **4+4+4**, 12. The triple is cards 7–9 (`اختصارات نصية`,
`الردود التلقائية`, `أوامر واضحة`), which is coherent rather than arbitrary: those are the
three chat-behaviour cards. Measured narrowest card at the 860px breakpoint: **263px**, with
all three at equal height. If a card is ever added or removed here, the parity problem comes
straight back — re-measure, do not assume the spans still tile.

Use `page.accessibility.snapshot({ root })` for accessible names — `innerText` returns `""`
for `visibility: hidden` elements and will produce false "unnamed control" failures. Root the
snapshot at the element under test; a whole-document name search cannot distinguish a nav
link from a footer link with the same label.

## Traps that have already cost time

- **Reveal stagger vs. screenshots.** `main.js` staggers `--d` by 80ms per sibling, so the
  7th bento card waits 480ms before its 700ms fade *starts*. A short fixed wait catches late
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
