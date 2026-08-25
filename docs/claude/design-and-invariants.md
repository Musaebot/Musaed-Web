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
  reading as a second brand colour. Everything else is green: **35 `var(--accent)` usages**
  (28 in `styles.css`, 7 in `legal.css`). Measured, not counted by hand —
  `grep -c 'var(--accent)' assets/css/*.css`. Note `var(--accent-soft)`,
  `var(--accent-line)` and `var(--on-accent)` do not match that pattern and are not part of
  the count. The `styles.css` figure was 21 before the 2026-08-25 rebuild; the page grew a
  tab list, filter chips, a FAQ accordion and several new link styles, all green.
  `legal.css` lost one on 2026-08-25 when `.notice` went with `connect.html`
  (`docs/claude/page-notes.md`); `updates.css` used to contribute 12 more and was removed
  the same day.
- **The blurple token is `#5a63d8`, not Discord's own `#5865f2`, and that is deliberate.**
  The 2026-08-25 design mockup used `#5865f2`; the project owner chose to keep `#5a63d8`.
  White label measures **4.99:1** on it versus 4.60:1 on the brand colour — both pass AA,
  ours by more. If you are asked to "match Discord's brand colour", confirm first.
- The Discord mark renders in **6 places**, all on `index.html` (5 before the
  2026-08-25 rebuild — `docs/claude/implementation-reference.md` has the table). White via
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

- **44px is the floor for tap targets**, and since 2026-08-25 it is reached with
  `min-height`, not with padding a later tidy-up could shave off. The old trick — padding
  cancelled by a negative margin, so the target grew without the layout moving — is gone with
  the elements that used it. Asserted at all nine widths.
- **Never `display: none` a label that is a control's accessible name.**
  `.side__toggle-text` (the menu button's "القائمة" label) hides its text by clipping it
  (`clip-path: inset(50%)`), so it stays in the accessibility tree. `display: none` would
  leave screen readers announcing an unlabelled button, since the burger icon is
  `aria-hidden`. (This was `.nav__toggle-text` before the 2026-08-25 rebuild — same rule,
  renamed control.) The `.vh` helper in `styles.css` is the reusable version of the same
  clip. The one icon-only control left, the topbar's `.iconlink`, is named with `aria-label`
  instead, which is the other correct answer. The updates back link used to be a third
  example of this pattern; it was removed with `updates.html` on 2026-08-25.
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
  with characters running left to right. **21 `dir="ltr"` attributes: 16 command names, 4
  Latin chips, 1 inline `.code`**, out of 16 total command rows — every command name is
  Latin, so that ratio is 1:1. (27 before the 2026-08-25 rebuild, when there were 11 chips.)
  Drop either half and you get a zigzag, or a slash on the wrong side.
- **`.cmd__name` means "a registered slash command" and things assert on it.** A Latin token
  inside a sentence — the `1h30m` in the "مدد بالعربي" card — uses `.code` instead. They look
  nearly identical; the split exists because the test suite asserts that every `.cmd__name`
  computes `direction: ltr` *and* starts with `/`, and reusing the class for prose broke
  that. Don't merge them back.
- **`.cmd__name--ar` is currently unused, not deleted.** It existed for `/اختصار`, the one
  Arabic command name the page ever had — `text-align: start` (right-aligns without a `dir`
  override, since the element is already RTL) plus dropping `--mono` (no Arabic glyphs, same
  reason `.chip--ar` exists). That row was removed from `#commands` on 2026-08-11 (`docs/claude/copy-accuracy.md`), so
  nothing on the page uses this class right now. If a future Arabic-named command shows up,
  reuse it rather than reinventing it; if none ever does, it's a safe deletion candidate.
- **A chip containing Arabic needs `.chip--ar`.** Plex Mono has no Arabic glyphs, so `--mono`
  falls back part-way through the string and opens a wide gap.
- **A `.reveal` grid needs `data-stagger` on its container** or its children all land at
  once instead of sequencing. `main.js` reads the attribute rather than a hardcoded selector
  list — the old `GROUPS` constant (`.bento, .cmds, .stats, .guards, .team, .about`) is gone,
  so adding a grid no longer means editing JS. There are **4** `data-stagger` containers.
- **Reveals inside a hidden panel need re-observing when the panel opens.** A panel starts
  under `hidden`, where an observed element never intersects. `refreshReveals()` unobserves
  and re-observes that panel's not-yet-revealed nodes on activation, forcing a fresh
  intersection check instead of waiting on the next frame budget.
- **There are no `nth-child` span rules any more.** The old `.bento` grid hand-placed cards
  into 12 columns and needed `.bento >` scoping so its rules could not leak into another
  grid containing a `.card`. The rebuild's `.grid` is a plain
  `repeat(auto-fit, minmax(min(100%, 270px), 1fr))` — cards tile themselves, and the parity
  problem that made the old grid fragile (see `docs/claude/testing-and-traps.md`) is gone
  with it. If you reintroduce hand-placed spans, reintroduce the scoping too.
- **Do not put a scroll-linked animation on an element that also has `.reveal`** — they fight
  over `transform`.
- **The tabs are `<a href="#panel-id">`, not `<button>`, and that is load-bearing.** With
  scripting off every panel is rendered and the tabs behave as ordinary in-page links. Never
  convert them to buttons: a button with no JS is dead, and these are the whole navigation.
- **The footer must carry every tab destination.** Without JS the phone menu button is hidden
  (a control that cannot work is worse than none), so `.foot__nav` is the only navigation a
  no-JS phone gets. It holds all six. Add a tab and you add a footer link, or don't add it.

### Measured width limits

**All of the figures that used to live here described the old nav bar and are gone with it.**
The 320px/6px-of-slack finding and the 768px four-links-on-one-line finding were both about
`.nav`, which no longer exists. Do not carry them forward.

What replaces them, measured 2026-08-25 in headless Chrome 152 at 320/375/390/412/768/860/
900/1024/1440px across all four pages:

- **900px is the layout breakpoint.** Below it the sidebar is a sticky bar plus a dropped
  menu panel; at and above it, a 256px sticky sidebar plus a 58px topbar.
- **700px is where command rows gain their three columns.** Below that they stack.
- **No page scrolls horizontally at any of the nine widths.** This was not free — see the
  `minmax(0, 1fr)` note in `docs/claude/implementation-reference.md`; a plain `1fr` sized the
  grid column to the nav's 468px min-content width and broke every phone width at once.
- **The filter chip row is the only thing on the page allowed to overflow.** It carries its
  own `overflow-x: auto`; nothing else may.
- **320px is still the tightest width**, and the phone bar is what's tight in it: brand +
  invite + menu toggle. `.side__invite` sheds padding below 900px and again below 360px to
  make it fit. Widening the brand or the toggle overflows there before anywhere else.
- **Every tab, filter chip, button, `summary` and the menu toggle clear 44px**, asserted at
  all nine widths. They get there with `min-height` / fixed `44px`, not with padding that a
  later tidy-up could shave off.
