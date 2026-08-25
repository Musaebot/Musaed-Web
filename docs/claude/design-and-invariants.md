# Design locks and invariants

## Design locks

- **Dark theme only. One accent** (`--accent`, terminal green). Never introduce a second hue.
- **Exactly two radii, no exceptions**: `--r-sm` (interactive/small), `--r-md` (panels).
  `updates.html` used to override both to zero for its terminal look; it was removed
  2026-08-25 (`docs/claude/page-notes.md`), so this lock is no longer qualified by anything.
- **RTL throughout.** Use logical properties (`padding-inline`, `inset-inline-start`,
  `margin-inline-start`), never physical left/right.
- **Never apply negative letter-spacing to Arabic.** It breaks the connected script. Tracking
  is for Latin/mono only. Arabic also needs a taller line-height than Latin.
- **Discord blurple is scoped to `.btn--primary` only** — buttons whose destination is
  Discord. Spread it further and it stops reading as "this goes to Discord" and starts
  reading as a second brand colour. Everything else is green: **29 `var(--accent)` usages**
  (21 in `styles.css`, 8 in `legal.css`). Measured, not counted by hand —
  `grep -c 'var(--accent)' assets/css/*.css`. Note `var(--accent-soft)` does not match that
  pattern and is not part of the count. `updates.css` used to contribute 12 more (total 40);
  it was removed 2026-08-25 (`docs/claude/page-notes.md`). The `styles.css` count grew from 16 to 20 on 2026-08-11
  (two new link styles, `.cmdgroup__note a` and `.dashfeatures__note a`, each using
  `var(--accent)` twice — `color`, `:hover` `border-color`) and to 21 when the nav login icon
  (§ nav, "Show the nav login icon at every width") got a default `color: var(--accent)`
  instead of only on `:hover`.
- The Discord mark renders in **5 places**, all on `index.html`. White via
  `filter: brightness(0) invert(1)` on blurple; natural colour on dark panels.
- **New sprite icons must come from the compiled Phosphor source, never hand-drawn and
  never fetched through `WebFetch`.** Pull the exact path from
  `https://raw.githubusercontent.com/phosphor-icons/core/main/assets/regular/<name>.svg`
  via `curl`/Bash — that's the compiled single-`<path>` fill version matching every existing
  symbol. `WebFetch` converts pages to markdown through a summarizing model and will mangle
  a long `d="..."` attribute. The repo's `raw/regular/<name>.svg` path (note: `raw/`, not
  `assets/`) is a *different*, stroke-based editable source (`<line>`/`<path stroke=...>`)
  and will not match the sprite's style if copied in directly — it has to be the `assets/`
  one.
- **Motion is opacity and transform only**, fully disabled under `prefers-reduced-motion`.
  There are no scroll event listeners anywhere — scroll-linked effects use native CSS
  scroll-driven timelines behind `@supports`.

## Invariants that are easy to break by accident

Each of these was found by measuring, and each looks harmless to "clean up".

- **44px is the floor for tap targets.** Several elements reach it via `padding-block`
  cancelled by a negative margin, so the target grew without the layout moving. Tightening
  padding for looks shrinks the target.
- **Never `display: none` a label that is a control's accessible name.** `.nav__toggle-text`
  (the hamburger button's "القائمة" label) hides its text by clipping it
  (`clip-path: inset(50%)`), so it stays in the accessibility tree. `display: none` would
  leave screen readers announcing an unlabelled button, since the burger icon is
  `aria-hidden`. The updates back link used to be the other example of this pattern; it was
  removed with `updates.html` on 2026-08-25.
- **Contrast has to be measured against the worst case, not the average, on any
  half-transparent panel.** The updates panel used to be the concrete example here —
  `--up-faint`/`--up-dim` were picked to clear 4.5:1 over the *accent glow* behind the
  panel, not the plain background, and looked darker than necessary in isolation because
  that was never where they had to survive. That panel is gone (`updates.html`, removed
  2026-08-25) and no current page has a half-transparent text panel, so there's no live
  example to point at right now — but the sticky nav (`background: rgb(8 10 9 / 0.82)`,
  `backdrop-filter: blur(14px)`) is exactly this shape, and the principle applies the moment
  anything adds text contrast requirements to a translucent surface again.
- **Latin code tokens in RTL text need two things, not one.** `dir="ltr"` isolates them so
  the leading slash stays left — but `dir` also flips the block's own alignment, so
  `.cmd__name` sets `text-align: end`, which resolves against the element's *own* `ltr`
  direction and therefore means right. Together they put names on the page's reading edge
  with characters running left to right. **27 `dir="ltr"` attributes: 16 command names + 11
  Latin chips**, out of 16 total command rows — every remaining command name is Latin, so
  the ratio is 1:1 as of 2026-08-11 (down from 44 names / 45 rows; `docs/claude/copy-accuracy.md` has the history). Drop
  either half and you get a zigzag, or a slash on the wrong side.
- **`.cmd__name--ar` is currently unused, not deleted.** It existed for `/اختصار`, the one
  Arabic command name the page ever had — `text-align: start` (right-aligns without a `dir`
  override, since the element is already RTL) plus dropping `--mono` (no Arabic glyphs, same
  reason `.chip--ar` exists). That row was removed from `#commands` on 2026-08-11 (`docs/claude/copy-accuracy.md`), so
  nothing on the page uses this class right now. If a future Arabic-named command shows up,
  reuse it rather than reinventing it; if none ever does, it's a safe deletion candidate.
- **A chip containing Arabic needs `.chip--ar`.** Plex Mono has no Arabic glyphs, so `--mono`
  falls back part-way through the string and opens a wide gap.
- **`.reveal` grids must be added to `GROUPS` in `main.js`** or their children all land at
  once instead of sequencing. Currently:
  `.bento, .cmds, .stats, .guards, .team, .about`.
- **`nth-child` rules are scoped with `.bento >`** so they cannot leak into a future grid
  that happens to contain a `.card`.
- **Do not put a scroll-linked animation on an element that also has `.reveal`** — they fight
  over `transform`.
- **The footer must carry every nav destination.** Without JS the phone menu button is hidden
  (a control that cannot work is worse than none), so the footer is the only navigation.

### Measured width limits

- **320px** is where the nav bar breaks first — about **6px of slack**. Widening the brand,
  the menu button, or the CTA overflows there before anywhere else. This is why
  `.nav__login` (the icon-only login link beside the CTA) is hidden below 768px
  instead of squeezed in here — there is no slack left to give it.
- **768px** is where the nav links first share one line with the brand and CTA. There are
  **4** now, not 6 — `التحديثات`/`updates.html` (2026-08-06) and `الأرقام`/`#stats`
  (2026-08-10) were both removed from `.nav__links` (see `docs/claude/page-notes.md`). **Slack is 191.5px, measured
  2026-08-17** in headless Chrome at exactly 768px, with every nav link clearing 44×44. That
  supersedes the old **69.8px** figure, which had been taken against the 6-link version and
  was stale in three directions at once. There is a lot of room now; the constraint on adding
  a link back is 320px (above), not this breakpoint.
