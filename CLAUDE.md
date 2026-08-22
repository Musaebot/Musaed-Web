# CLAUDE.md — working on Musaed-Web

Orientation for an AI agent picking this repo up. `README.md` is the reference manual and
is longer; this file is what you need before you touch anything.

**This file is committed to a public GitHub repo.** Do not put deployment internals in it —
no infrastructure identifiers, no environment variable names, no bot-side table or module
names, no server IDs. That rule applies to everything you write into this repo.

---

## 1. What this is

A **static public marketing site** for **مساعد (Musaed)**, an Arabic (Saudi dialect)
moderation/automod Discord bot. Five pages. It explains the bot to server owners before they
add it.

It is **not** a dashboard, and nothing on it manages anything or shows real guild data —
this repo itself has no login state, no account, no backend. The two login buttons (nav +
hero) are outbound links to the dashboard's own OAuth2 flow, a **separate application on a
separate origin** (see §2) — same category of link as the Discord bot-invite buttons, not
auth scaffolded in this repo.

```text
index.html                 landing page + its inline icon sprite (22 symbols)
404.html                   custom error page, served by Caddy - see §9
updates.html               changelog + its own sprite (1 symbol) - unlinked, see below
connect.html               pre-auth disclosure screen before the dashboard login - see §7
privacy.html               privacy policy   } same layout, one shared
terms.html                 terms of use     } stylesheet, NO JavaScript
google82b70d7af988f7a9.html  Google Search Console site-verification file - see below
sitemap.xml                lists the 4 real pages (not connect.html, not 404.html) - see §7
robots.txt                 Allow: / for everyone, points at sitemap.xml - see §7
assets/css/styles.css      tokens, reset, shared components
assets/css/updates.css     updates page only, loaded after styles.css
assets/css/legal.css       connect.html + BOTH legal pages, loaded after styles.css
assets/js/main.js          stats data, count-up, scroll reveals, nav, link guard
assets/fonts/              self-hosted woff2 (IBM Plex Sans Arabic + Plex Mono)
assets/Pics/               brand marks. Capital P — Linux hosts are case-sensitive
```

Five pages, three stylesheets, one script. **Zero dependencies, zero build step.** There was
a sixth page (`developer.html`); it was removed along with its CSS and JS. Do not resurrect
it.

**`google82b70d7af988f7a9.html` is not a page.** It is the file Google Search Console issues
for the HTML-file verification method: single line of plain text, no doctype, no `<html>`.
It has to be reachable at exactly `https://musaed.dev/google82b70d7af988f7a9.html` — root of
the domain, filename unchanged — for Google to accept the verification, so it deploys as-is
alongside the real pages. Nothing links to it and nothing should; it is not part of site
navigation, the same way `404.html` isn't.

**`updates.html` is unlinked, not removed.** As of 2026-08-06 the nav link (`التحديثات`)
and its footer counterpart in the `النظام` column were both deleted from `index.html`, along
with the now-orphaned `#i-sparkle` sprite symbol they used (22 → 21 symbols at the time; the
sprite is back at 22 since `#i-chat-text` was added 2026-08-17, §4). The page file,
`assets/css/updates.css`, its own sprite, and its content are all untouched and still deploy
normally — it is simply unreachable from site navigation, only by someone who already has
the direct `/updates.html` URL. If it ever gets relinked, restore both the nav `<a>` and the
footer `<li>` (§6's nav-link and footer-a11y counts below assume it stays absent) and re-add
`#i-sparkle` to the sprite.

**`#stats` (`مساعد بالأرقام`) is hidden, not removed — same pattern as `updates.html`, one
level deeper.** As of 2026-08-10 the section carries a `hidden` attribute, and its nav link
(`الأرقام`) and footer counterpart in the `الموقع` column were both deleted from
`index.html`. Unlike `updates.html` this isn't a separate page — `#stats` lives inline on
`index.html` itself — so unlinking the nav anchor alone would not have stopped it from
rendering as you scroll; the `hidden` attribute is what actually does that. The markup, its
three `data-mock` stats, and `main.js`'s `initStats()` are all untouched: the count-up code
still runs against the hidden nodes (harmless — an `IntersectionObserver` on a `display:none`
subtree just never fires), and `#i-users` stays referenced from inside the section itself, so
removing the footer row did not orphan that sprite symbol the way removing `التحديثات` did
for `#i-sparkle`. If this ever gets shown again, remove `hidden` and restore both the nav
`<a>` and the footer `<li>`.

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
**`dashboard.musaed.dev`** — linked from this site's two login buttons, nowhere else. Every
setting reachable through `/agegate`, `/captcha`, `/tickets`, `/اختصار`, `/welcome` and
`/automod` is reachable through a form there today — all six of the bot's registered
features. `/welcome` and `/automod` were the two holdouts and no longer are; the dashboard's
pages are generated from the bot's settings registry rather than written per feature, so a
newly registered feature appears there without dashboard work. This section's rules are still
the ones that govern extending it, not just history.

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
2. `#commands` — the full reference: **7 groups, 16 subcommand rows** (down from 9/45 as of
   2026-08-11 — see below)

Both are plain HTML in `index.html`. No generator; sync by hand.

**`#commands` is deliberately no longer a full reference — as of 2026-08-11 it is a curated
subset plus dashboard pointers.** The project owner supplied an exact allow-list of 16 real
commands (`/about`, `/ban`, `/captcha lockdown`, `/captcha unlock`, `/kick`, `/modlogs`,
`/musaed`, `/prefix reset`, `/prefix set`, `/prefix show`, `/serverinfo`, `/tickets panel`,
`/tickets setup`, `/timeout`, `/warn`, `/welcome preview`) and every other subcommand
previously listed was removed — not because the underlying bot features stopped existing, but
because most per-feature configuration moved to the dashboard (§2) and the page no longer
tries to enumerate every slash subcommand that still technically exists. **This inverts the
historical rule below it in this section**: `#commands` used to be a complete, generated-by-
hand mirror of the bot's registered command tree (hence "the full reference"); it is now a
short list of commands still worth calling out directly, plus short notes pointing everything
else at the dashboard. If you're asked to "sync `#commands` with the bot" in the old sense —
listing every registered subcommand — confirm with the owner first; the 2026-08-11 change was
a deliberate scope-narrowing, not the site falling behind. The three "moved to dashboard"
groups (`الترحيب`, `التحقق بالكابتشا`, `تذاكر الدعم`) each end with an identical one-line
`.cmdgroup__note`: **"باقي الإعدادات من dashboard.musaed.dev."** — same wording every time, no
restated feature detail. Two groups (`الحماية التلقائية`, `بوابة عمر الحساب`) lost every
subcommand and were removed as standalone groups entirely; they now live in one line each
inside a new small section right after `#commands`, `#dashboard-features`
(`ميزات تُدار من الداشبورد`), which carries a single shared dashboard link for all of them
rather than one per item. **It holds three `<li>`s, not two, as of 2026-08-17** —
auto-responses was added alongside automod and agegate for the same reason (a real system
with no slash commands); the single shared link did not change. `اختصارات نصية` (shortcuts, `/اختصار`) was removed from `#commands`
outright with no dashboard note at all — it simply is not in the owner's 16-command allow-list
and was not one of the five sections named for the "rest is on the dashboard" treatment. Don't
assume that means the feature is gone from the bot; it means this page stopped listing it and
no reason was given. Verify against the bot's actual command tree (the advice further down
this section) before writing anything more specific than that.

**This has already gone wrong.** The page originally advertised prefix commands
(`!طرد`, `!حظر`, `!اسكات`, `!تحذير`), a per-server prefix feature, and a rate-limit card with
an invented `!تهدئة` command. The bot registered **slash commands only** at the time — none
of that existed. It is all gone.

**`/prefix` used to be deliberately excluded, and no longer is — this is a reversal, not
drift.** The original reasoning: the command group existed and wrote a prefix, but no
prefix-based text commands were registered, so setting one had no user-visible effect, and
listing it would have advertised a no-op. As of 2026-08-11 the project owner explicitly added
`/prefix set`, `/prefix show`, `/prefix reset` to `#commands`'s allow-list (its own group,
`بادئة الأوامر`, icon `i-terminal`, pill `إدارة السيرفر`) regardless of that old reasoning. If
you're asked to "clean up" and remove `/prefix` again citing the no-op argument, confirm with
the owner first — the same way the Administrator-permission reversal below asks you to.

**In-chat moderation triggers exist but are deliberately not enumerated.** The bot now
recognizes a few plain Arabic words typed directly in chat as shortcuts for ban/timeout/
untimeout/warn — a deliberate, owner-approved exception on the bot side to "slash commands
only," each trigger gated by the same permission its slash equivalent uses and silent for
anyone lacking it. The "أوامر واضحة" card in `#features` mentions this in one generic
sentence — no trigger words, no mention that they're per-deployment configurable — on
purpose: naming them here would be this site publishing exactly what the bot's own design
keeps unlisted. **Never add a `#commands` row for these** — they are not slash commands, and
`#commands`'s own lede claims everything listed there is one.

**Shortcuts used to be listed as their own `#commands` group, and no longer are.**
`/اختصار` (Administrator) is still a real registered slash command that lets a server admin
define their own trigger word mapped to ban/kick/timeout — a peer of agegate/captcha/tickets
in the bot's settings layer, not a sub-feature of moderation. It had its own group
(`اختصارات نصية`, icon `i-lightning`, pill `مدير السيرفر`) until 2026-08-11, when it was
removed from `#commands` along with everything else not on the owner's 16-command allow-list
(see above). **Its bento card in `#features` was not touched** — that section wasn't in scope
for the 2026-08-11 change — so the feature is still described there, just no longer given its
own command-reference row. `/اختصار` was also the one command name on the page written in
Arabic rather than Latin, styled with `.cmd__name--ar` (`text-align: start`, `--sans` instead
of `--mono`) instead of the `dir="ltr"` isolation every Latin command name gets. With that row
gone, **`.cmd__name--ar` in `styles.css` is now unused dead code** — every remaining
`.cmd__name` on the page is Latin and gets plain `dir="ltr"`. Left in place rather than
deleted, in case an Arabic-named command gets added back later; flag it for cleanup if one
never does.

**`#commands` grew from 7 groups to 9, then back down to 7 — do not conflate the two
episodes.** The 7→9 growth (historical, pre-2026-08-11): `/about` split off from `معلومات`
into its own single-command group (`دليل الأوامر`, icon `i-book`, pill `إدارة السيرفر`) when
it stopped sharing a permission with `/serverinfo`, and the shortcuts group above shipped
separately — two unrelated reasons, same net effect (7 → 8 → 9). The 9→7 drop is the
2026-08-11 rewrite described earlier in this section: automod and agegate lost their groups
entirely (their commands moved to `#dashboard-features` instead), shortcuts was removed with
no replacement, and a new `بادئة الأوامر` (`/prefix`) group was added — net 9 → 7, not a
coincidence that it isn't 9 → 6; the prefix reversal happened in the same pass.

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

The eight real systems: moderation, automod (banned words + spam + raid detection), welcome,
account-age gate, captcha, support tickets, text shortcuts (admin-defined trigger words for
ban/kick/timeout), auto-responses (admin-defined trigger word → canned reply). Plus Arabic
duration parsing, which is the most distinctive feature and is worth keeping prominent.

**It was seven until 2026-08-17.** Auto-responses (`الردود التلقائية`) shipped on the bot
side 2026-08-16 and the site had never mentioned it — found by diffing the bot/dashboard
product inventory against this repo, which is exactly the check the paragraph below this
section's history describes. It got a `#features` bento card (icon `i-chat-text`) and a
`#dashboard-features` row, and both seven-counts on the site moved to eight: `#features`'s
lede (`سبعة أنظمة` → `ثمانية أنظمة`) and `updates.html`'s `أنظمة` figure (7 → 8). It has no
slash commands at all, so it gets **no `#commands` row** — same shape as automod and agegate.

**A ninth system exists on the bot side and must stay off this site.** The honeypot trap
channel (`قناة الفخ`) is gated to a single pilot guild and is not released; it is not one of
the eight above. Do not add it to `#features`, `#dashboard-features`, or the counts, however
complete the bot-side documentation looks — publishing an unreleased feature is worse than
omitting a released one.

**Being a "real system" and having a `#commands` presence are no longer the same claim.**
Automod, account-age gate and auto-responses are all still real, all still count toward the
eight above, and none of them has a single row in `#commands` — they're dashboard-only from
this page's point of view (see `#dashboard-features`, above). Don't use "it's not in
`#commands`" as evidence a system stopped existing; check the dashboard-managed groups and
`#dashboard-features` before concluding that.

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
  reading as a second brand colour. Everything else is green: **40 `var(--accent)` usages**
  (20 in `styles.css`, 12 in `updates.css`, 8 in `legal.css`). Measured, not counted by
  hand — `grep -c 'var(--accent)' assets/css/*.css`. Note `var(--accent-soft)` does not
  match that pattern and is not part of the count. The `styles.css` count grew from 16 to 20
  on 2026-08-11: two new link styles (`.cmdgroup__note a`, `.dashfeatures__note a`), each
  using `var(--accent)` twice (`color`, `:hover` `border-color`).
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
  with characters running left to right. **27 `dir="ltr"` attributes: 16 command names + 11
  Latin chips**, out of 16 total command rows — every remaining command name is Latin, so
  the ratio is 1:1 as of 2026-08-11 (down from 44 names / 45 rows; §4 has the history). Drop
  either half and you get a zigzag, or a slash on the wrong side.
- **`.cmd__name--ar` is currently unused, not deleted.** It existed for `/اختصار`, the one
  Arabic command name the page ever had — `text-align: start` (right-aligns without a `dir`
  override, since the element is already RTL) plus dropping `--mono` (no Arabic glyphs, same
  reason `.chip--ar` exists). That row was removed from `#commands` on 2026-08-11 (§4), so
  nothing on the page uses this class right now. If a future Arabic-named command shows up,
  reuse it rather than reinventing it; if none ever does, it's a safe deletion candidate.
- **A chip containing Arabic needs `.chip--ar`.** Plex Mono has no Arabic glyphs, so `--mono`
  falls back part-way through the string and opens a wide gap.
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
- **768px** is where the nav links first share one line with the brand and CTA. There are
  **4** now, not 6 — `التحديثات`/`updates.html` (2026-08-06) and `الأرقام`/`#stats`
  (2026-08-10) were both removed from `.nav__links` (see §1). **Slack is 191.5px, measured
  2026-08-17** in headless Chrome at exactly 768px, with every nav link clearing 44×44. That
  supersedes the old **69.8px** figure, which had been taken against the 6-link version and
  was stale in three directions at once. There is a lot of room now; the constraint on adding
  a link back is 320px (above), not this breakpoint.

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

`terms.html` is dated `2026/08/08`; `privacy.html` moved to `2026/08/18` when the email
scope and the Pro-lapse retention paragraph were added to it (`terms.html` was not in scope
for that pass, so the two dates are deliberately different now). Both were originally the
dashboard-aware rewrite of `2026/08/08`,
covering the dashboard's login (name/avatar/ID collected, guild list scoped to what you can
manage, session expiry) and the shortcuts feature's user-authored text. Staged first in
`txt.txt` (now gitignored — it is scratch, not the copy of record; delete it freely once its
content is absorbed) before being split into `privacy.html` / `terms.html` by hand. The only
invented values left anywhere on the site are the three `#stats` numbers.

**The bot invite is live and appears three times** (nav, hero, about), each opening in a new
tab with `rel="noopener noreferrer"`. There is no single source for the URL, so a change to
one is a change to all three. Its `permissions` integer is what Discord pre-ticks on the
authorize screen — editing it changes what the bot is granted on join, so do not "tidy" it.

**It requests `permissions=8` — Administrator, and nothing else — as of 2026-08-10.** This
is a deliberate reversal of the previous design: the invite used to request an explicit
23-permission set specifically to avoid Administrator, on the reasoning that a moderation bot
asking for Administrator is asking a server to stop reasoning about what it can do. The
project owner chose to switch to Administrator anyway; if you're asked to "clean up" the
permissions integer or restore least-privilege, confirm with the owner first — this was a
conscious tradeoff, not drift. `#trust`'s claim ("صلاحياتك هي القرار" — the bot checks *the
user's* permissions before acting, not what the bot's own OAuth grant allows) is unaffected
by this and still holds: it's about per-command authorization checks against the invoking
member, orthogonal to what scope the bot's own application token carries. Ampersands are
`&amp;` because it is an HTML attribute; the browser sends plain `&`.

**`updates.html` has zero `data-mock`** — its single release (`1.0.0`, `أغسطس 2026`, `البداية`)
is real, owner-supplied. Its two header figures are kept in step by hand: `إصدارات` counts
the `.rel` blocks (**1**), `أنظمة` mirrors the systems `#features` advertises (**8** since
2026-08-17, was 7 — §4). Nothing enforces that second one; it is a number in a second file
that silently disagrees with the landing page the moment a system is added, which is exactly
how it spent a day wrong. **Do not add a release you cannot date.**

**Live network surface: exactly one `fetch`, and it is unreachable.** `readStats()` returns
the local `STATS` object while `STATS_ENDPOINT` is `null`. The rate-limiting path (TTL
throttle, 429 backoff honouring `Retry-After`, last-good fallback) is already written; wiring
a real endpoint is a one-constant change. The declaration is space-aligned, so
`grep "STATS_ENDPOINT ="` with one space matches nothing and looks like the guard is gone.

### The hardcoded domain

`https://musaed.dev` — **20 absolute URLs, 4 per page across 5 pages**: `canonical`, `og:url`,
`og:image`, `twitter:image`. Crawlers reject relative URLs, so it is written literally.
(`404.html` carries none of the four and is not part of the twenty — an error page has no
canonical address.)

**These moved off `musaed.up.railway.app` on 2026-08-08.** That host served the site too and
always had, so nothing was broken — which is exactly why it went unnoticed: two live hosts
serving identical content, with `canonical` pointing every crawler and social card at the
unbranded one, so `musaed.dev` was published as the duplicate of its own Railway URL. A
domain move that leaves the site *working* at both names is the easy one to leave half-done.

**`musaed.up.railway.app` is retired as of 2026-08-18 and must not be referenced again.**
The generated Railway service domain may still answer while it remains attached to the
service, but it is not part of how this site is served, linked, or verified. It used to have
one real job — reading past Cloudflare to check a CSS/JS deploy — and §9 now does that with a
cache-busting query string instead, which needs no second host. Treat any surviving mention
as stale.

```bash
grep -ohE 'https://[a-z0-9.-]+' *.html | sort -u
```

That must print exactly four hosts: the site's, `discord.gg` for the community invite
(`https://discord.gg/QvNXvDDFtz` — **live, not a placeholder**), `discord.com` for the
bot-invite URL and the link to Discord's own terms, and
`dashboard.musaed.dev` — the dashboard (a separate deployable, §2).

**The dashboard host appears 5 times as of 2026-08-11: once in `connect.html`, four times in
`index.html`.** Until 2026-08-11 it was referenced exactly once, in `connect.html` — the two
login buttons in `index.html` (nav + hero) point at `connect.html`, the pre-auth disclosure
screen, whose "فهمت، كمّل" button is the single outbound link to the dashboard, and that was
the *only* place the host string appeared. The `#commands` allow-list rewrite (§4) added four
more, direct `<a href="https://dashboard.musaed.dev">` links: three identical
`.cmdgroup__note` lines (`الترحيب`, `التحقق بالكابتشا`, `تذاكر الدعم`) and one in
`#dashboard-features`. These are deliberately **not** routed through `connect.html` — they're
informational "read more about this feature" pointers, not the prominent "log in" CTAs
`connect.html` exists to intercept, and the dashboard has its own `/auth/connect` disclosure
that fires before OAuth regardless of how someone arrives. If the dashboard host ever moves,
grep will now surface 5 lines across 2 files, not 1 line in 1 file — update all of them. The
20 `canonical`/`og:url`/`og:image`/`twitter:image` URLs below are a separate set and stay
pointed at the site's own host exclusively — unaffected by this.

`connect.html` is a **content page, not auth** — no session, no OAuth, no protected state, so
§3.4 is intact. The dashboard serves its own copy of the same disclosure at `/auth/connect`,
because `/auth/login` is a public URL that a bookmark or a stale third-party link reaches
without passing through this site. **The two wordings must stay in sync**; the other one is
`app/templates/connect.html` in the Musaed-Dashboard repo.

**The dashboard's host changed on 2026-08-08** (it was a generated `*.up.railway.app` name)
and the old one was **deleted, not left redirecting** — so both login buttons pointed at a
dead host until they were updated. That is the same failure this section warns about below,
one host over: nothing in this repo breaks visibly when a *different* deployable moves, because
the links still render fine and only fail on click. If the dashboard host ever moves again,
grep for the host, not for `auth/login` — the `href` carrying it now lives in `connect.html`.

A domain move is **not** done when the site loads at the new address; it is done when all
twenty URLs are updated. This has now bitten twice, in both directions: once when an old host
404'd and half the site still pointed at it, so social cards referenced a dead image — and
again on 2026-08-08, when the old host kept working and nothing looked wrong at all. The first
kind announces itself. The second does not, so grep the host after every move rather than
trusting the site to tell you.

### sitemap.xml and robots.txt

`sitemap.xml` lists exactly the four real, indexable pages — `/`, `/privacy.html`,
`/terms.html`, `/updates.html` — each with a `<lastmod>` taken from that file's last commit
date, not invented. **Deliberately excludes two pages that otherwise look like they belong:**
`connect.html` (a functional pre-auth step, not content someone should land on from a search
result) and `404.html` (an error page has no canonical address — same reasoning as the
`canonical`/`og:url` exclusion above). `updates.html` **is** included even though it's
unlinked from nav/footer (§1) — unlinked from navigation and hidden from search are different
claims, and the page is still live and real.

`robots.txt` allows everything and points at the sitemap. Both files hold absolute
`musaed.dev` URLs but are **not** `.html`, so they are invisible to the `grep -ohE
'https://[a-z0-9.-]+' *.html` check above — a future domain move has to update these two
files by hand as well, or the sitemap will keep advertising the old host to crawlers after
every page's own `canonical` has already moved on.

`dashboard.musaed.dev` gets no robots.txt of its own from this repo — that app is a separate
deployable (§2) and staying unindexed is tracked as a separate, smaller task there, not here.

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
- **Nav**: 4 links on one line at ≥768px, each ≥44×44, and report the slack.
- **Menu behaviour** (375px): starts closed; click opens; panel sits flush under the bar;
  Escape closes and returns focus to the button; Tab from the button enters the menu; a link
  click closes it *and* navigates; outside tap closes; crossing 768px closes it; the closed
  nav exposes **2** links to the a11y tree and the open one **6** (both figures unverified —
  written from the current markup, not a rerun; §6 explains why).
- **Structure**: 10 bento cards in 5 rows; 22 sprite symbols with none orphaned; 7 command
  groups / 16 rows, each group either carrying a permission pill or a `.cmdgroup__note` (or
  both) — `الترحيب`, `التحقق بالكابتشا`, `تذاكر الدعم` have both; every `.cmd__name` computes
  `direction: ltr` and starts with `/`, with no exceptions as of 2026-08-11 (the one
  Arabic-named row, `/اختصار`, is gone — §4, §6); 1 `#dashboard-features` section with exactly
  3 `<li>`s and exactly 1 outbound link; 3 guarantee panels.

**Current: 154 assertions, all passing — rerun 2026-08-17** in headless Chrome 152 against
`python -m http.server`, covering all 5 pages × 9 widths (320, 375, 390, 412, 768, 860, 900,
1024, 1440) plus the structure and bento-row checks above. 860 and 900 are new and were added
for a reason: 860px is where `.bento` becomes a 12-column grid, so it is the width at which
the narrowest card is narrowest, and nothing was measuring it. **This run did not include the
17 menu assertions** — the 375px open/close/Escape/focus behaviour is still unverified since
2026-08-05, so treat that 17 as stale even though the 128 is now superseded.

**The bento grid has one row of three since 2026-08-17, and it is load-bearing.** Adding the
auto-responses card (§4) made it 10 cards — 9 regular plus the full-width `card--full`
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
  Railway host is no longer part of how this site is served or verified (§7), and the query
  string above needs no second host anyway. Purging properly still has to happen from
  Cloudflare's dashboard (Caching → Configuration → Purge Cache), which no git push or Railway
  action can reach — but a stale entry does expire on its own inside the 4 hours, which is how
  the 2026-08-17 case resolved. HTML-only changes (like the `#stats` `hidden` attribute, §1)
  are unaffected, which is why that change worked immediately and these didn't.

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

**Then verify the live deploy on `musaed.dev` with a cache-busting query string if the change
touched CSS or JS** — §9 has the exact commands. Cloudflare caches those for up to 4 hours on
the branded domain and a redeploy doesn't purge it, so the plain URL can look unchanged (or
half-changed) right after a push that fully succeeded. Check `last-modified` against the
deploy time, not just whether the page renders. Plain `musaed.dev` is fine for HTML-only
changes, which are never cached.

Also confirm the push actually triggered a Railway deploy — it doesn't always. `list-deployments`
should show a fresh entry with `reason: "deploy"` and the new commit hash within a few
minutes. If it doesn't, `redeploy` will **not** fix it (it re-runs the last known build, not
the latest commit — confirmed 2026-08-11); push an empty commit (`git commit --allow-empty`)
to re-trigger the GitHub App event instead.

---

## 11. Working style expected here

Purely technical and direct. Explain non-obvious decisions briefly; skip the obvious ones.

**Report honestly.** If a check was not run, say so rather than implying it passed. If a
claim was measured, say what was measured. If you were wrong, correct it in a sentence and
move on — do not narrate the mistake.

**Do not invent numbers.** Every figure on this site is either real or explicitly marked as a
placeholder. If you need a count, measure it; if you cannot, mark it.

Never truncate code with placeholders or `// ... rest unchanged`.
