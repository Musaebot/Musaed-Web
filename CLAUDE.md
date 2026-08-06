# CLAUDE.md — working on Musaed-Web

Orientation for an AI agent picking this repo up. `README.md` is the reference manual and
is longer; this file is what you need before you touch anything.

**This file is committed to a public GitHub repo.** Do not put deployment internals in it —
no infrastructure identifiers, no environment variable names, no bot-side table or module
names, no server IDs. That rule applies to everything you write into this repo.

---

## 1. What this is

A **static public marketing site** for **مساعد (Musaed)**, an Arabic (Saudi dialect)
moderation/automod Discord bot. Four pages. It explains the bot to server owners before they
add it.

It is **not** a dashboard, and nothing on it manages anything or shows real guild data —
this repo itself has no login state, no account, no backend. The two login buttons (nav +
hero) are outbound links to the dashboard's own OAuth2 flow, a **separate application on a
separate origin** (see §2) — same category of link as the Discord bot-invite buttons, not
auth scaffolded in this repo.

```text
index.html                 landing page + its inline icon sprite (22 symbols)
404.html                   custom error page, served by Caddy - see §9
updates.html               changelog + its own sprite (1 symbol)
privacy.html               privacy policy   } same layout, one shared
terms.html                 terms of use     } stylesheet, NO JavaScript
assets/css/styles.css      tokens, reset, shared components
assets/css/updates.css     updates page only, loaded after styles.css
assets/css/legal.css       BOTH legal pages, loaded after styles.css
assets/js/main.js          stats data, count-up, scroll reveals, nav, link guard
assets/fonts/              self-hosted woff2 (IBM Plex Sans Arabic + Plex Mono)
assets/Pics/               brand marks. Capital P — Linux hosts are case-sensitive
```

Four pages, three stylesheets, one script. **Zero dependencies, zero build step.** There was
a fifth page (`developer.html`); it was removed along with its CSS and JS. Do not resurrect
it.

**`privacy.html` and `terms.html` load no JavaScript at all** — no `<script>`, no
`class="no-js"`. A legal document must be readable with scripting off and by a crawler that
does not run it, and both URLs go in Discord's Developer Portal (Privacy Policy URL / Terms
of Service URL), so they must resolve as plain documents. Do not add `main.js` to them "for
consistency": the scroll reveals would fade a privacy policy in.

---

## 2. Where this is going — the dashboard

**The dashboard exists and is deployed**: a server owner signs in with Discord, picks one
of their servers, and configures Musaed from the browser instead of typing slash commands.
It lives in its own repo (`musaed-dashboard`, private) and its own Railway service, at
`musaed-dashboard-production.up.railway.app` — linked from this site's two login buttons,
nowhere else. Every setting reachable through `/agegate`, `/captcha`, `/tickets` and
`/اختصار` is reachable through a form there today; `/automod` and `/welcome` aren't yet
(they predate the bot's settings-registry pattern — a bot-side change, not a dashboard one).
This section's rules are still the ones that govern extending it, not just history.

**Read this before you read §3, because it looks like a contradiction and is not.**

> The dashboard is a **separate application**. It does not relax a single rule in §3.
> Those rules govern *this repo* — a public marketing page served to anonymous visitors.

### What that means concretely

- **Do not add auth, a backend, a database client, or a build step to this repo** in order to
  get there. The marketing site stays static. A dashboard is a different deployable with its
  own service, its own origin or subdomain, and its own threat model. Mixing them means the
  marketing page inherits a login surface it has no reason to have.
- **The marketing site keeps showing zero real guild data even after the dashboard ships.**
  A dashboard shows one server's data to people authorized for *that* server. A marketing
  page shows nobody's. That distinction does not soften later.

### Design constraints the bot already imposes

The bot was built anticipating this, so most of the hard decisions are already made. Respect
them rather than re-inventing:

- **The settings layer is the single write path.** The bot has a generic per-guild settings
  service that the slash commands themselves go through, explicitly intended to also serve a
  dashboard or API. It owns types, bounds, defaults, dependency rules between settings, and
  cache invalidation. **A dashboard must call that service, not write settings rows
  directly.** Building a second write path is the specific failure this design exists to
  prevent — it would bypass validation and dependency checks that the slash commands honour.
- **Dependencies are declared, and queryable.** Some settings cannot be enabled until others
  are set, and a setting cannot be cleared while something active depends on it. The service
  exposes these so an interface can disable a toggle *before* the user tries it, instead of
  failing the write. Use that — a dashboard that only reports errors after submitting is
  strictly worse than the slash commands.
- **Being logged in is not authorization.** Discord OAuth2 establishes *who* someone is. For
  each server and each feature you must still verify they hold the Discord permission that
  feature requires — the age gate needs Administrator, the rest need Manage Server. Verify
  per request, server-side, against Discord. **Never trust a server id supplied by the
  client**, and never infer permission from the fact that a guild appeared in someone's list.
- **Tenant isolation is enforced at runtime, not by convention.** Every read and write is
  scoped to a single server, and violations raise rather than returning data. A dashboard
  gets no exemption from this; if a query needs an escape hatch, that is a design smell.
- **Changes made outside the bot process are not automatically picked up.** The bot caches
  per-guild settings. A write from a dashboard has to notify the bot, or it keeps serving the
  old values until something in-process invalidates them. Ordering matters: notify *after* the
  transaction commits, or the cache re-fills with the pre-write value.

### Sequencing

The public stats seam (§7) is the smaller, safer version of the same problem and is a
reasonable first step: a read-only public aggregate endpoint, no auth, no per-server data.
Getting that right establishes the deployment shape before anything touches per-guild
configuration.

---

## 3. Hard rules — do not violate

These came from the project owner. They are not style preferences.

**Scope: they govern this repo — the public marketing site.** The planned dashboard (§2) is a
separate application with its own security model; it is not an exception to anything below,
because nothing below is about it. Do not weaken a rule here to make dashboard work fit.

1. **This site has zero connection to the bot's production database, and must never gain
   one.** The bot enforces strict per-server tenant isolation; guild data belongs to the
   servers that generated it. A public marketing page has no business touching it.
2. **Never read environment variables.** No `DATABASE_URL`, no bot token, no secrets, and no
   code path that could consume one.
3. **Never import a database, Discord, or auth library.** No SQLAlchemy, asyncpg, discord.py,
   session handling, or OAuth flow.
4. **Never scaffold login, auth, or protected routes *in this repo*.** There is no logged-in
   state on the marketing site at all. Auth belongs to the dashboard (§2), which lives
   elsewhere — not behind a flag here.
5. **Keep it dependency-free and build-free.** Fonts and icons are vendored. If you need
   tooling (a headless browser to test with), install it *outside* this repo.
6. **All user-facing copy is Saudi-dialect Arabic. All code comments and placeholder markers
   are English.**
7. **Every invented number is marked in-code** so it is trivial to find and replace.

If live numbers are ever needed, they must come from a **separate, purpose-built public
aggregate endpoint** — never per-server rows, never member identities. The seam already
exists; see §7.

---

## 4. Copy accuracy — the site describes a real bot

The statistics are invented. **Everything else must stay true to the bot.** A page listing a
command the bot does not answer to is worse than one listing nothing.

Two places go stale when the bot's command surface changes, and only these two:

1. the `.chip` lists in `#features`
2. `#commands` — the full reference: **9 groups, 45 subcommand rows**

Both are plain HTML in `index.html`. No generator; sync by hand.

**This has already gone wrong.** The page originally advertised prefix commands
(`!طرد`, `!حظر`, `!اسكات`, `!تحذير`), a per-server prefix feature, and a rate-limit card with
an invented `!تهدئة` command. The bot registered **slash commands only** at the time — none
of that existed. It is all gone.

**`/prefix` is deliberately not listed.** The command group exists and writes a prefix, but
no prefix commands are registered, so setting one has no user-visible effect. Listing it
would advertise a no-op.

**In-chat moderation triggers exist but are deliberately not enumerated.** The bot now
recognizes a few plain Arabic words typed directly in chat as shortcuts for ban/timeout/
untimeout/warn — a deliberate, owner-approved exception on the bot side to "slash commands
only," each trigger gated by the same permission its slash equivalent uses and silent for
anyone lacking it. The "أوامر واضحة" card in `#features` mentions this in one generic
sentence — no trigger words, no mention that they're per-deployment configurable — on
purpose: naming them here would be this site publishing exactly what the bot's own design
keeps unlisted. **Never add a `#commands` row for these** — they are not slash commands, and
`#commands`'s own lede claims everything listed there is one.

**Shortcuts are a separate feature from the in-chat triggers above, and are listed.**
`/اختصار` (Administrator) is a real registered slash command that lets a server admin
define their *own* trigger word — any word they pick, stored per guild — that maps to
ban, kick, or timeout. It is a fourth feature in the bot's settings layer, a peer of
agegate/captcha/tickets, not a sub-feature of moderation, so it gets the same treatment
those three got: its own bento card (`اختصارات نصية`, icon `i-lightning`) and its own
`#commands` group. Publishing that it exists reveals nothing guild-specific — the word is
the admin's own choice, not a fixed set the bot ships with — and an admin has to know
`/اختصار` exists to configure it, so unlike the fixed triggers, hiding it would only make
the feature unusable. Its pill reads **مدير السيرفر**, matching `بوابة عمر الحساب` — both
are strictly Administrator-only, not the Administrator-or-Manage-Server tier `/about`
uses, so they get the stricter wording, not `إدارة السيرفر`.

**`/اختصار` is the one Arabic command name on the page**, so its `.cmd__name` carries no
`dir="ltr"` — the surrounding rows use that attribute to isolate a *Latin* name from RTL
reordering, and applying it to real Arabic text would reverse it, which is a different
failure than the zigzag it prevents for Latin names. It gets `.cmd__name--ar` instead:
`text-align: start` right-aligns it to the same edge as every other row (its direction is
left at the ambient RTL, where `start` is the right edge — no `dir` override needed to get
there), and the font swaps off `--mono` (no Arabic glyphs) the same way `.chip--ar` does
for a chip. See §5 for the icon-sourcing rule this icon was fetched under.

**`#commands` grew from 7 groups to 9 across two separate reasons — do not conflate them.**
First, `/about` and `/serverinfo` stopped sharing one permission: `/about` became an
admin-only command manual, auto-generated from the bot's live command tree (gated to
Administrator-or-Manage-Server on the bot side); `/serverinfo` stayed public. One group can
only carry one permission pill, so `/about` got its own single-command group
(`دليل الأوامر`, icon `i-book`) instead of forcing a false shared pill onto `معلومات`
(7 → 8). Its pill reads **إدارة السيرفر** — the same wording already used for
automod/welcome/captcha/tickets, not new wording — because Discord's Administrator
permission bypasses every lesser check anyway, so "Administrator or Manage Server" and plain
"Manage Server" gate the exact same set of members in practice. `/musaed` (a public,
DM-capable "what is this bot" command with an invite-link button) took `/about`'s old seat
in `معلومات` next to `/serverinfo`. Second, unrelated to any of that, the shortcuts feature
above shipped its own group (8 → 9).

**When the bot-side project guide gets pasted into a session, diff it against this site
before touching anything else.** Three times now the bot's own `CLAUDE.md`/guide has been
shared mid-conversation and each time the site had silently fallen behind: round one, a
whole feature (`/tickets`) plus a subcommand (`/captcha status`) were missing; round two, a
permission had changed underneath an already-published command (`/about`); round three, a
second whole feature (`/اختصار`/shortcuts) was missing *and* an already-listed group had
grown four subcommands the site never got (`/tickets add/remove/list/edit`, ticket-type
management) — two gaps at once, in the same group and in a brand-new one, which is exactly
why a partial glance at the diff isn't enough. None of these show up by reading this site's
own files — they only show up by comparing against the bot's current command surface. If
you're handed that guide, read §4, §7 (or wherever the
bot repo lists its commands/permissions) and cross-check every group and row here before
doing anything else asked of you.

The seven real systems: moderation, automod (banned words + spam + raid detection), welcome,
account-age gate, captcha, support tickets, text shortcuts (admin-defined trigger words for
ban/kick/timeout). Plus Arabic duration parsing, which is the most distinctive feature and
is worth keeping prominent.

**`#trust` quotes no numbers** for retention windows or automod thresholds — those are
server-configurable defaults, not promises. Do not add figures there.

---

## 5. Design locks

- **Dark theme only. One accent** (`--accent`, terminal green). Never introduce a second hue.
- **Exactly two radii**: `--r-sm` (interactive/small), `--r-md` (panels). `updates.html`
  deliberately overrides both to zero for its terminal look — that is the one exception.
- **RTL throughout.** Use logical properties (`padding-inline`, `inset-inline-start`,
  `margin-inline-start`), never physical left/right.
- **Never apply negative letter-spacing to Arabic.** It breaks the connected script. Tracking
  is for Latin/mono only. Arabic also needs a taller line-height than Latin.
- **Discord blurple is scoped to `.btn--primary` only** — buttons whose destination is
  Discord. Spread it further and it stops reading as "this goes to Discord" and starts
  reading as a second brand colour. Everything else is green: **36 `var(--accent)` usages**
  (17 in `styles.css`, 12 in `updates.css`, 7 in `legal.css`).
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

---

## 6. Invariants that are easy to break by accident

Each of these was found by measuring, and each looks harmless to "clean up".

- **44px is the floor for tap targets.** Several elements reach it via `padding-block`
  cancelled by a negative margin, so the target grew without the layout moving. Tightening
  padding for looks shrinks the target.
- **Never `display: none` a label that is a control's accessible name.** The updates back
  link hides its text under 560px by clipping it (`clip-path: inset(50%)`), so it stays in
  the accessibility tree. `display: none` would leave screen readers announcing an
  unlabelled link, since the icon is `aria-hidden`.
- **Contrast is measured against the worst case, not the average.** The updates panel is
  half-transparent, so small text must clear 4.5:1 over the *accent glow* behind it, not over
  the plain background. `--up-faint` and `--up-dim` look darker than necessary on plain
  background because that is not where they have to survive.
- **Latin code tokens in RTL text need two things, not one.** `dir="ltr"` isolates them so
  the leading slash stays left — but `dir` also flips the block's own alignment, so
  `.cmd__name` sets `text-align: end`, which resolves against the element's *own* `ltr`
  direction and therefore means right. Together they put names on the page's reading edge
  with characters running left to right. **55 `dir="ltr"` attributes: 44 command names + 11
  Latin chips**, out of 45 total command rows. Drop either half and you get a zigzag, or a
  slash on the wrong side.
- **`/اختصار` is the one Arabic command name, and it must NOT get `dir="ltr"`.** That
  attribute isolates *Latin* text from RTL reordering; applying it to real Arabic text
  reverses it, which is a worse failure than the zigzag it prevents elsewhere. Use
  `.cmd__name--ar` instead — `text-align: start`, which right-aligns it to the same edge
  as every `dir="ltr"` row, because the element is left at its natural (inherited) RTL
  direction, where `start` already means right. No `dir` override needed or wanted.
- **A chip containing Arabic needs `.chip--ar`.** Plex Mono has no Arabic glyphs, so `--mono`
  falls back part-way through the string and opens a wide gap. `.cmd__name--ar` swaps off
  `--mono` for the same reason.
- **`.reveal` grids must be added to `GROUPS` in `main.js`** or their children all land at
  once instead of sequencing. Currently:
  `.bento, .cmds, .stats, .guards, .team, .about`.
- **Consecutive `.rel__desc` paragraphs are one description.** `.rel__desc + .rel__desc`
  zeroes the border and top padding so only the first draws a hairline.
- **`nth-child` rules are scoped with `.bento >`** so they cannot leak into a future grid
  that happens to contain a `.card`.
- **Do not put `.reveal` on `.rel__meta`.** It animates `transform`, and a transformed
  ancestor traps the sticky version block inside it.
- **Do not put a scroll-linked animation on an element that also has `.reveal`** — they fight
  over `transform`.
- **The footer must carry every nav destination.** Without JS the phone menu button is hidden
  (a control that cannot work is worse than none), so the footer is the only navigation.

### Measured width limits

- **320px** is where the nav bar breaks first — about **6px of slack**. Widening the brand,
  the menu button, or the CTA overflows there before anywhere else. This is why
  `.nav__login` (the icon-only login link beside the CTA) is hidden below 768px
  instead of squeezed in here — there is no slack left to give it.
- **768px** is where the **6** nav links first share one line with the brand and CTA.
  Measured slack: **69.8px**. A seventh link costs roughly 75px, so it would not fit —
  re-measure before adding one, and prefer a footer-only link.

  **This figure is now stale for what actually renders here.** `.nav__login` was added
  beside the CTA at this exact breakpoint (44px + the 18px `nav__inner` gap ≈ 62px) on an
  estimate that it fits under the measured 69.8px, not on a re-measurement — no headless
  browser was available when it was added. Re-measure the real slack at 768px before
  trusting either number, and before adding anything else here.

---

## 7. Placeholders — everything still open

| What | Where | Marker |
| --- | --- | --- |
| 3 statistics (servers, members, uptime) | `STATS` in `main.js` | `// placeholder`, `data-mock="true"` |
| _(none — every text placeholder is filled)_ | | |

```bash
grep -rn "data-mock" index.html               # 3 — all in #stats
grep -rn "data-placeholder-link" index.html   # 0 — every link is real now
grep -c  "oauth2/authorize" index.html        # 3 — the live invite, must be identical
grep -n  "STATS_ENDPOINT" assets/js/main.js   # must stay null
```

**Every link on the site points somewhere real.** The footer's قانوني column was the last
holdout and is now wired to `privacy.html`, `terms.html` and a live `mailto:`.

Both legal documents are dated `2026/08/01`; the terms date arrived as `[التاريخ]` in the
source and was filled in by the project owner. The only invented values left anywhere on the
site are the three `#stats` numbers.

**The bot invite is live and appears three times** (nav, hero, about), each opening in a new
tab with `rel="noopener noreferrer"`. There is no single source for the URL, so a change to
one is a change to all three. Its `permissions` integer is what Discord pre-ticks on the
authorize screen — editing it changes what the bot is granted on join, so do not "tidy" it.
It requests 23 permissions and, correctly, **not** Administrator: a moderation bot that asks
for Administrator is asking a server to stop reasoning about what it can do. That is least
privilege, which is a different claim from the one `#trust` makes on the page
("صلاحياتك هي القرار" — the bot checks *the user's* permissions before acting). Both matter;
do not conflate them. Ampersands are `&amp;` because it is an HTML attribute; the browser
sends plain `&`.

**`updates.html` has zero `data-mock`** — its single release (`1.0.0`, `أغسطس 2026`, `البداية`)
is real, owner-supplied. Its two header figures are kept in step by hand: `إصدارات` counts
the `.rel` blocks (**1**), `أنظمة` mirrors the seven systems `#features` advertises (**7**).
**Do not add a release you cannot date.**

**Live network surface: exactly one `fetch`, and it is unreachable.** `readStats()` returns
the local `STATS` object while `STATS_ENDPOINT` is `null`. The rate-limiting path (TTL
throttle, 429 backoff honouring `Retry-After`, last-good fallback) is already written; wiring
a real endpoint is a one-constant change. The declaration is space-aligned, so
`grep "STATS_ENDPOINT ="` with one space matches nothing and looks like the guard is gone.

### The hardcoded domain

`https://musaed.up.railway.app` — **16 absolute URLs, 4 per page across 4 pages**:
`canonical`, `og:url`, `og:image`, `twitter:image`. Crawlers reject relative URLs, so it is
written literally.

```bash
grep -ohE 'https://[a-z0-9.-]+' *.html | sort -u
```

That must print exactly four hosts: the site's, `discord.gg` for the community invite
(`https://discord.gg/QvNXvDDFtz` — **live, not a placeholder**), `discord.com` for the
bot-invite URL and the link to Discord's own terms, and
`musaed-dashboard-production.up.railway.app` — the dashboard (a separate deployable, §2),
linked from the two login buttons in `index.html` (nav + hero). Only those two links point
there; the 16 `canonical`/`og:url`/`og:image`/`twitter:image` URLs below are a separate set
and stay pointed at the site's own host exclusively — unaffected by this.

A domain move is **not** done when the site loads at the new address; it is done when all
sixteen URLs are updated. This has already bitten once: the old host 404s, and half the site
still pointed at it, so social cards referenced a dead image.

---

## 8. How to verify — do not reason about CSS, measure it

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
- **Nav**: 6 links on one line at ≥768px, each ≥44×44, and report the slack.
- **Menu behaviour** (375px): starts closed; click opens; panel sits flush under the bar;
  Escape closes and returns focus to the button; Tab from the button enters the menu; a link
  click closes it *and* navigates; outside tap closes; crossing 768px closes it; the closed
  nav exposes **2** links to the a11y tree and the open one **8**.
- **Structure**: 9 bento cards in 5 rows; 9 command groups / 45 rows each with a permission
  pill; every `.cmd__name` computes `direction: ltr` and starts with `/`, **except**
  `/اختصار`, which is genuinely Arabic and computes `direction: rtl` by design (§6); 3
  guarantee panels.

Prior totals were **128 assertions + 17 menu assertions, all passing** — measured before the
`/tickets` system, `/musaed`, the `/about` permission/grouping change, the shortcuts system
(`/اختصار`), and the `/tickets` type-management subcommands were added to the site
(2026-08-05). The counts above (9 cards, 45 rows) are grep-verified; the full
browser-measured assertion count has not been rerun since, so treat 128/17 as stale until the
suite is recreated and rerun.

Use `page.accessibility.snapshot({ root })` for accessible names — `innerText` returns `""`
for `visibility: hidden` elements and will produce false "unnamed control" failures. Root the
snapshot at the element under test; a whole-document name search cannot distinguish a nav
link from a footer link with the same label.

---

## 9. Traps that have already cost time

- **Reveal stagger vs. screenshots.** `main.js` staggers `--d` by 80ms per sibling, so the
  7th bento card waits 480ms before its 700ms fade *starts*. A short fixed wait catches late
  elements mid-transition and makes a correct layout look like missing content. Inject
  `.reveal{opacity:1!important;transform:none!important;transition:none!important}` instead
  of waiting it out.
- **`fullPage` screenshots clip** on pages with a `position: fixed` backdrop (`updates.html`).
  The geometry is fine — measure `getBoundingClientRect()` before believing the image. Use
  viewport or element screenshots there.
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
  `python -m http.server` (the tool this file's own §8 tells you to test with) has no concept
  of a custom error page and will show its own plain 404 for a missing path — that is not a
  regression, it just isn't testing the mechanism that matters. The actual wiring is Railpack's
  staticfile provider's *default* Caddyfile (nothing in this repo — confirmed from
  `github.com/railwayapp/railpack`'s own template, not guessed): `handle_errors { rewrite *
  /{err.status_code}.html }`. That single line is why a file named exactly `404.html` at the
  root is the entire fix — no Caddyfile override needed, and the only way to actually verify
  it is against the live Railway URL.

---

## 10. Git and deploy

Work happens on **`main`**, which is what Railway deploys. Commit and push only when asked.

Before pushing, scan the tree — not just the diff:

```bash
grep -rniIE "DATABASE_URL|BOT_TOKEN|api[_-]?key|secret" --include="*.html" --include="*.css" --include="*.js" .
grep -rnIE "console\.(log|debug)|debugger|localhost|TODO|FIXME" --include="*.html" --include="*.css" --include="*.js" .
```

Both should come back empty. `README.md` mentions those names only to assert their absence,
which is fine — the scan above deliberately excludes `.md`.

After a push, verify against the **remote**, not just locally:

```bash
git ls-remote origin refs/heads/main    # must equal git rev-parse HEAD
```

---

## 11. Working style expected here

Purely technical and direct. Explain non-obvious decisions briefly; skip the obvious ones.

**Report honestly.** If a check was not run, say so rather than implying it passed. If a
claim was measured, say what was measured. If you were wrong, correct it in a sentence and
move on — do not narrate the mistake.

**Do not invent numbers.** Every figure on this site is either real or explicitly marked as a
placeholder. If you need a count, measure it; if you cannot, mark it.

Never truncate code with placeholders or `// ... rest unchanged`.
