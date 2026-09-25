# Implementation reference — the 2026-09-24 rebuild

**Replaced outright on 2026-09-24.** The previous version of this file documented the
sidebar-plus-tab-panels `index.html` that existed before that date. This version documents
the current build: a long-scrolling, bilingual landing page ("the gatekeeper" redesign),
brought into this repo from a separate working folder in the same batch that self-hosted its
fonts, ported its JSON-LD, and fixed its asset paths to this repo's `assets/` convention. If
you're looking for how the *old* tabbed page worked, that's gone — don't reach for git
history expecting to restore it without checking with the owner first (see root `CLAUDE.md`
"What reversed").

## Files

| File | Purpose |
| --- | --- |
| `index.html` | The landing page. All visible text is written in English and tagged with `data-i18n` keys; the Arabic dictionary in `i18n.js` overrides it at runtime. |
| `assets/js/i18n.js` | `window.MUSAED_I18N`: the Arabic dictionary (`ar`), runtime strings for both languages (`dyn`), join-gate demo accounts (`joiners`), and the slash-command list (`commands`). |
| `assets/js/main.js` | Landing behaviour: language switch, live stats, nav, join-gate simulation, reveals, stacked layers, member-DM counter, dashboard demo, command search, scroll effects. |
| `assets/css/styles.css` | `@font-face` rules, design tokens, layout, animations, RTL rules, reduced-motion fallbacks, and the 404 styles. Also used by the legal pages. |
| `privacy.html`, `terms.html` | Legal pages, Arabic only. |
| `assets/css/legal.css`, `assets/js/legal.js` | Legal pages only: reading-progress bar, contents highlighting, phone chip bar, back-to-top. |
| `404.html` | "هذي القناة مو موجودة" page, Arabic first, no JS. |
| `assets/Pics/musaed-favicon.svg` | The mint "م" glyph on a dark rounded square. `assets/Pics/musaed-favicon.png` and `musaed-avatar.png` are the `alternate icon`/`apple-touch-icon` fallbacks. |

**Cache-busting:** `styles.css` is `?v=7`, `main.js` is `?v=3`, `i18n.js` is `?v=2`, `legal.js` is
`?v=1`, `legal.css` is `?v=2`. Bump the query string on any further edit to that file — see
`docs/claude/git-and-deploy.md` for why (that doc's specifics predate this rebuild but the
cache-busting mechanism itself is unchanged).

## Language (Arabic is primary, English is a toggle)

- **Default is Arabic.** The inline script in `<head>` of `index.html` picks the language in
  this order: `?lang=ar|en` in the URL, then `localStorage['musaed-lang']` (only `'en'`
  overrides), then `'ar'`. It sets `lang` and `dir` on `<html>`.
- While Arabic loads, `<html>` carries the `i18n-pending` class, which hides `<body>` until
  `main.js` has swapped the text in. This prevents a flash of English.
- **How the swap works:** on load, `main.js` records each `[data-i18n]` element's English
  `innerHTML`, then writes `I.ar[key]` into it for Arabic. Attributes are handled the same
  way through `data-i18n-attr="attr:key,attr2:key2"`.
- **Adding or changing text:** write the English in `index.html` with a `data-i18n="section.name"`
  key, then add the same key to `ar` in `i18n.js`. Dictionary values may contain simple markup
  such as `<em>`, `<mark>` or `<a>`, because they are static trusted strings.
- **Runtime text** lives in `I.dyn.en` / `I.dyn.ar`: page title, language-button label and menu
  labels; gate rejection reasons and dashboard preview text; command categories; live-stats
  line and bot status.
- **Plural helpers:** `dyn.*.days(n)` and `dyn.*.attempts(n)` apply Arabic plural rules
  (يوم / يومين / 3 أيام / 11 يوم). `arCount()` does the same for server and member counts.
- **Adding language-dependent behaviour:** register it with `onLang(fn)` in `main.js`; it runs
  after every switch. The join-gate demo, dashboard, commands, live stats and mobile-menu
  label all do this.
- **RTL rules:** use logical CSS properties (`inset-inline-*`, `margin-inline-*`,
  `padding-inline-*`, `border-inline-*`) for anything that should mirror. Direction-specific
  overrides use `[dir="rtl"]`: arrow icons, marquee direction (`marqueeRtl`), progress-bar
  origin, gate-token positioning.
- `html[lang="ar"] body * { letter-spacing: normal !important; }`: tracking breaks Arabic
  letter joins, so never letter-space Arabic.
- Numbers use Western digits in both languages. Command names and `7 + 5 = ?` are forced LTR
  with `direction: ltr; unicode-bidi: isolate`.
- **Voice:** Gulf-flavoured, friendly Arabic matching the bot's own strings (e.g. "وش", "تبي",
  "على طول", "الحين").
- **Legal pages are Arabic-only, on purpose** (Hard Rule 6 in root `CLAUDE.md`). Don't add an
  English translation without the owner's sign-off.

## Brand

| Token | Value | Use |
| --- | --- | --- |
| `--bg` / `--bg-1..4` | `#0b0d0c` → `#232b28` | Green-tinted near-black surfaces |
| `--accent` | `#0fe37d` | Musaed mint: buttons, highlights, logo |
| `--accent-hi` / `--accent-deep` | `#6cf5b0` / `#0a9e57` | Hover state, gradients |
| `--on-accent` | `#07130c` | Text on mint |
| `--text` / `--dim` / `--mute` | `#f3f5f3` / `#a3aba4` / `#8a928c` | Text levels |
| `--danger` | `#ff6b6b` | Rejections, kicks, the honeypot |

- **Fonts, self-hosted as of this integration** (they loaded from Google Fonts in the
  original working-folder build — see root `CLAUDE.md`'s "Fonts are now self-hosted" note):
  - **Alexandria** (`assets/fonts/alexandria-arabic.woff2` / `alexandria-latin.woff2`) is the
    display face; variable weight 500–900, covers Latin and Arabic.
  - **Readex Pro** (`readexpro-arabic.woff2` / `readexpro-latin.woff2`) is the body face;
    variable weight 300–700, also Latin and Arabic.
  - **IBM Plex Mono** (`plex-mono-400.woff2` / `plex-mono-500.woff2`) is for labels and code,
    Latin only. Readex Pro is second in `--f-mono` so Arabic text in mono labels falls back
    to it.
- **Logo:** the same Arabic letterform path used by the old site's `assets/Pics/musaed-avatar.svg`.
  Each HTML file defines it inline as the SVG symbol `#glyph`.
- **Icons:** hand-inlined Lucide paths as `<symbol id="i-*">`, drawn with `<svg class="i"><use href="#i-..."/></svg>`. No emoji icons.
- **Visual theme:** "the gatekeeper". A grid background with a sweeping scan line, a mint grid
  that lights up under the cursor, a mint ticker band, and the big Arabic word مساعد in the
  finale.

## Landing page sections (in order)

1. **Nav:** a floating bar that gains a blurred background after scrolling. It holds the
   language toggle, "دخول اللوحة" / "Log in" (≥1180px), and "Add to Discord". A mobile menu
   takes over below 1080px.
2. **Hero:** headline "المشاكل توقف عند الباب" / "Trouble stops at the door." with an animated
   underline. The logo glyph draws its outline and then fills, with rotating rings and three
   floating status chips (hidden below 1024px). The live-stats line appears under the buttons
   once the API answers.
3. **Join-gate demo (`#gate`):** the signature animation. Accounts travel New joins → Age gate
   → Captcha → Honeypot → Verified. Accounts that fail turn red, shake, and show the rejection
   reason under their gate. Joined, Verified and Stopped counters update as it runs. Below
   820px the pipeline turns vertical. The demo pauses when off-screen or when the tab is
   hidden, and restarts on resize or language change.
4. **Ticker:** a mint marquee of feature names.
5. **Protection (`#protection`):** four layers (Age gate, Captcha, Honeypot, AutoMod) as
   sticky cards that stack from 900px up. `main.js` shrinks and dims each card as the next one
   slides over it (`scale: var(--s)` and `filter: brightness(var(--b))`). Each card has a
   one-shot illustration that plays when it scrolls into view.
6. **Toolkit (`#toolkit`):** a bento grid of Tickets, Shortcuts, Auto-responses, Welcome, Mod
   logs with `/lookup`, and Member DMs. The DM card runs a looping "Sending x / 1,204" counter.
7. **Dashboard (`#dashboard`):** an interactive browser mock, followed by an "افتح لوحة
   التحكم" button that goes to the real dashboard login. Controls: toggles for Age gate,
   Captcha and Honeypot; a range slider for minimum age (1–30 days); challenge type (button or
   math); a stepper for max attempts (1–10). A live "Discord preview" of the verify panel
   updates as you change them, and a "Panel synced" chip flashes.
8. **Commands (`#commands`):** a searchable, filterable list rendered from `I.commands`. 23
   real slash commands whose Arabic descriptions are copied from the bot.
9. **Your data (`#data`):** retention explained in three steps.
10. **Plans (`#plans`):** Free, and Pro marked **Coming soon** (see root `CLAUDE.md` "What
    reversed" — there is no numeric Free/Pro comparison table on the page anymore). The Pro
    button is a non-clickable dashed pill with `aria-disabled`.
11. **FAQ (`#faq`):** six questions in native `<details>`, animated with `::details-content`.
    Answers 2 and 5 link to the dashboard and the support server. Mirrored word-for-word in
    `index.html`'s JSON-LD `FAQPage`.
12. **Finale:** a giant مساعد that fills with mint from right to left as you scroll
    (`--fill`), then the closing CTA.
13. **Footer:** product, resource and legal links, the language toggle, and the live bot
    status.

## Content rules and sources

- **Every feature claim must match the bot**, whose source is at `../Musaed` (sibling
  repo, `cogs/agegate.py`, `cogs/captcha.py`, `cogs/honeypot.py`, `cogs/automod.py`,
  `cogs/tickets.py`, `welcome.py`, `shortcuts.py`, `autoresponse.py`, `memberdm.py`,
  `panelsync.py`, `maintenance.py`). Slash command names and their Arabic descriptions come
  from `core/strings.py` (`CMD_*_DESC`); group names come from the cogs.
- If the bot adds, renames or removes commands, update `commands` in `assets/js/i18n.js` —
  and the FAQ / featureList in `index.html`'s JSON-LD if the change is significant enough to
  affect those.

## Real links and live data

All links are hard-coded in the HTML, so they work without JS. External links open in a new
tab (`target="_blank" rel="noopener"`).

| What | URL | Where it's used |
| --- | --- | --- |
| Bot invite | `https://discord.com/oauth2/authorize?client_id=1341863717247258655&permissions=8&integration_type=0&scope=bot+applications.commands` | Every "Add" button (landing and legal pages), the Free plan, footer "Invite" |
| Dashboard login | `https://dashboard.musaed.dev/auth/login` | Nav, mobile menu, dashboard section, footers, FAQ answer 2 |
| Support server | `https://discord.gg/CcwRT6K5qv` | Footers, FAQ answer 5 |
| Privacy / Terms | local `privacy.html`, `terms.html` (canonical `https://musaed.dev/privacy.html`, `/terms.html`) | Landing footer, legal-page nav |
| Canonical / OG image | `https://musaed.dev/`, `https://musaed.dev/assets/Pics/musaed-banner.png` | `<head>` on every page |
| Contact email | Obfuscated with HTML entities inside the legal pages | Legal pages only. Keep the obfuscation. |

**Live stats:** `GET https://dashboard.musaed.dev/api/public/stats` returns
`{ guild_count, member_count, status, updated_at }`.

- `main.js` (`liveStats`) fetches it at most once every 5 minutes. It caches the response in
  `sessionStorage['musaed:stats']`, and after a failure or 429 it backs off for 15 minutes (it
  honours `Retry-After`).
- On success it shows the hero line "يحمي N سيرفر وM عضو الحين" / "Protecting N servers and M
  members right now", plus the footer status "البوت شغال" / "Bot is online" (a red dot if the
  status isn't `up`).
- Both stay `hidden` until real data arrives. Numbers are never faked.
- **CORS only allows `https://musaed.dev`.** Opened from `file://`, `localhost`, or another
  host, the fetch is blocked (a CORS error in the console) and the stats simply don't show —
  this is expected during local testing, not a bug.
- **Testing locally:** seed the cache, then reload:
  `sessionStorage.setItem('musaed:stats', JSON.stringify({at: Date.now(), data: {guild_count: 21, member_count: 2307, status: 'up'}}))`

## Legal pages

- **The wording is word for word identical** to the pages that existed before this rebuild —
  diffed by hand during integration (headings, contents list, every paragraph, the obfuscated
  `mailto:` entities, `<bdi>` and `<code dir="ltr">`). Only the markup and layout around it
  changed.
- **To update a policy,** change the text and also the `آخر تحديث` date in the hero chip. If
  you change any section IDs, update the contents list to match.
- **Layout:**
  - Sticky top bar with brand, Home, Privacy, Terms and the invite button; the current page is
    marked with `aria-current="page"`.
  - Mint reading-progress bar across the top.
  - Hero with the grid background, title, last-updated chip, "This page is available in
    Arabic." chip, and the opening paragraph.
  - The contents list is a sticky numbered sidebar from 1024px up (on the right, because the
    page is right-to-left). Below that it becomes a sticky chip bar that scrolls sideways.
  - `legal.js` highlights the current section in the contents list.
  - The body is limited to 72ch with 2.0 line height, list items are shown as cards, and the
    end of each page has a "اقرأ بعدها" card linking to the other page. A back-to-top button
    appears after 700px of scrolling.

## Accessibility and motion

- The pages have skip links, visible `:focus-visible` rings, 44px touch targets, and switches
  built as `role="switch"` + `aria-checked`. The stepper and radio group are labelled. Every
  decorative animation carries `aria-hidden`, and the join-gate demo has an `.sr-only` text
  description.
- **Reduced motion:** `prefers-reduced-motion` turns animations off and shows a meaningful
  static end state: the logo is filled and the underline drawn; the gate demo shows fixed
  counters and one reason per gate; the ticker wraps instead of scrolling; the finale word is
  fully filled; the DM counter shows 642.
- Without JS, all content stays visible (in English on the landing page — a `<noscript>` fix
  for this is still open, see root `CLAUDE.md` "Next goals").

## Gotchas found while building (carried from the original working-folder notes)

- **Never name a state class `scan`.** `.scan` is the hero's sweeping scan-line. The gate demo
  uses `.scanning` and `.deny` on stations; reusing `.scan` once threw the Age gate station
  out of the panel.
- **Don't use `-webkit-text-stroke` or semi-transparent fills on big Alexandria text.** It is a
  variable font with overlapping contours, which show as inner lines or darker overlaps. Use
  solid colours (as `.layer-n` and `.nf-code` do), or fade a solid copy with `opacity`, which
  flattens the glyphs first (as `.finale-ar::before` does).
- **iOS WebKit: no masked or hard-stop-gradient grid, no animated `background-clip: text`.**
  Found on a real iPhone (2026-09-25), invisible on desktop and in Playwright's Windows
  WebKit, so only a real iPhone can confirm a fix here. The `.grid` painted as nothing — the
  page was flat black — both with its `mask-image` and, after that was removed, still with
  its `linear-gradient(… 1px, transparent 1px)` line tiles. The finale word, filled by
  animating a `background-size` under `background-clip: text` (inside a composited layer
  from its `filter` and `.reveal`), painted once and never repainted as `--fill` changed.
  The fixes: the grid is a 56px inline SVG tile that fades under a `::after` overlay in
  `--bg` (each use sets `--fade`; the SVG's stroke is `--line` spelled out), and the finale
  fill is a solid `::after` copy revealed by `clip-path`. The hero's pointer glow keeps its
  gradient + mask, but only inside `(hover: hover) and (pointer: fine)`.
- **Don't use `<fieldset>` as a flex row.** Legends don't lay out as flex items. The dashboard
  uses `div[role="radiogroup"]` with `aria-labelledby`.
- **`.card` sets `display: flex` after the bento media queries.** Layout overrides for a
  specific card need higher specificity, e.g. `.bento .card.c-dms`.
- **Positioning the gate tokens:** tokens are absolutely positioned inside `.track`, and each
  move target is computed from the station `.node`'s bounding box. That makes horizontal,
  vertical, LTR and RTL layouts work without separate code.
- **Where the ready classes live:** `body.ready` (added two frames after boot) triggers the
  hero entrances. `.reveal` elements get `.in` from an IntersectionObserver.
- **Grid items and wide children:** a single-column grid holding a horizontally scrolling
  child needs `grid-template-columns: minmax(0, 1fr)` and `min-width: 0` on its children.
  Without them, the legal pages' chip bar could stretch the page sideways on phones.
- **Don't use `scrollIntoView` to centre a chip in a sticky bar.** It can scroll the whole
  page. `legal.js` uses `list.scrollBy({ left: … })` instead.
- **Mono font and Arabic:** IBM Plex Mono has no Arabic glyphs. Readex Pro must stay second in
  `--f-mono`, or Arabic mono labels fall back to a stretched system font.

## Verifying changes

There is no test suite. Check in a real browser at 1440px and 390px wide, in both languages
(`?lang=ar` / `?lang=en`), plus `privacy.html`, `terms.html` and `404.html`. Check that:

- the console has no errors (the stats CORS error is expected off `musaed.dev`);
- nothing scrolls sideways (`scrollWidth === clientWidth`);
- the gate demo runs;
- the dashboard preview reacts to the controls;
- the command search filters;
- the legal contents list highlights while scrolling.

**Headless check (Windows, Node 22+ for global `fetch`/`WebSocket`, Chrome DevTools Protocol
over `--remote-debugging-port`).** Edge is typically at
`C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe` on a Windows dev machine;
headless `--window-size` can't go narrow enough for phone widths, use
`Emulation.setDeviceMetricsOverride` instead. This is how the 2026-09-24 integration was
smoke-tested (zero console errors, no sideways scroll, correct titles, both languages, both
widths, all four pages) — see root `CLAUDE.md` "Next goals" for what that check did **not**
cover (clicking through the interactive pieces by hand).
