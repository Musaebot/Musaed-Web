# CLAUDE.md — working on Musaed-Web

Orientation for an AI agent picking this repo up — read this before you touch anything.
`README.md` is a short plain-language page for human visitors (what the project is, how to
run it) and is not a technical reference — don't look there for history, reasoning, or
implementation detail.

**The landing page was rebuilt on 2026-08-25** from a supplied design mockup: it is now a
sidebar + tab panels rather than one long scrolling page. Same content, same URLs — the
tabs are real anchors and every old `#features`/`#commands`/`#trust`/`#about`/`#about-us`
deep link still resolves. See `docs/claude/implementation-reference.md` for the panel map.
**Eight panels as of 2026-09-05.** `#pricing` joined the original six on 2026-09-04 (Free/Pro
numbers from `core/plans.py`, still **no numeric Pro price**; the Pro CTA points at the
dashboard's `/pricing/get-pro` upgrade route as of 2026-09-05, replacing an earlier
"contact the team" Discord link). `#why-musaed` ("ليش مساعد؟") joined 2026-09-05 — a
competitor-comparison panel sitting **after `#faq`** (late-funnel), with a `.filters`-style
switcher (MEE6 / Dyno), driven by `initVersus()` in `main.js` — the switch replays a CSS
cascade-in (`vs-in` keyframe in `styles.css`, added 2026-09-05; opacity/transform only,
flattened by the reduced-motion block). It states claims about **other** bots, so it
carries the same accuracy burden as `#commands` — see `docs/claude/copy-accuracy.md`.
Several counts in `docs/claude/testing-and-traps.md` were pinned at six and are now stale
until the measurement suite is rerun — read that file's own note before trusting a number
in it.

**This file was split on 2026-08-25 into this slim root plus `docs/claude/*.md` files.**
Root keeps only identity, the hard rules, and the "Where to find things" table below.
Everything else — the detailed reference material, the change history, the per-feature
reasoning — moved into `docs/claude/`. Use the table to find the right file before you go
looking for something that used to live in this file directly.

**This file is committed to a public GitHub repo.** Do not put deployment internals in it —
no infrastructure identifiers, no environment variable names, no bot-side table or module
names, no server IDs. That rule applies to everything you write into this repo.

---

## 1. What this is

A **static public marketing site** for **مساعد (Musaed)**, an Arabic (Saudi dialect)
moderation/automod Discord bot. Three pages, one of them an eight-panel tabbed page. It
explains the bot to server owners before they add it.

It is **not** a dashboard, and nothing on it manages anything or shows real guild data —
this repo itself has no login state, no account, no backend. The two «لوحة التحكم» buttons
(sidebar + topbar) are plain outbound links to the dashboard's own OAuth2 flow, a **separate
application on a separate origin** (see `docs/claude/dashboard.md`) — same category of link
as the Discord bot-invite buttons, not auth scaffolded in this repo. They pointed at a local
`connect.html` interstitial until 2026-08-25; that page was removed (`docs/claude/page-notes.md`)
and they now link straight to the dashboard, which serves its own pre-OAuth disclosure.

```text
index.html                 landing page: 8 tab panels + inline icon sprite (24 symbols) + a JSON-LD @graph in the head
404.html                   custom error page, served by Caddy - see docs/claude/testing-and-traps.md
privacy.html               privacy policy   } same layout, one shared
terms.html                 terms of use     } stylesheet, NO JavaScript
google82b70d7af988f7a9.html  Google Search Console site-verification file - see docs/claude/page-notes.md
sitemap.xml                lists all 3 real pages (not 404.html) - see docs/claude/placeholders-and-domain.md
robots.txt                 Allow: / for everyone, points at sitemap.xml - see docs/claude/placeholders-and-domain.md
llms.txt                   plain-text overview for AI systems (llmstxt.org format) - added 2026-09-05
pricing.txt                machine-readable pricing for AI agents; MIRRORS the #pricing panel's numbers - added 2026-09-05
assets/css/styles.css      tokens, reset, shared components
assets/css/legal.css       BOTH legal pages, loaded after styles.css
assets/js/main.js          tab router, command filter, versus switch, scroll reveals, stats data + count-up, link guard
assets/fonts/              self-hosted woff2 (IBM Plex Sans Arabic + Plex Mono)
assets/Pics/               brand marks. Capital P — Linux hosts are case-sensitive
```

Three pages, two stylesheets, one script. **Zero dependencies, zero build step.** There used
to be more: `developer.html` was removed along with its CSS and JS, and do not resurrect it.
`updates.html` (the changelog) was removed on 2026-08-25 along with `assets/css/updates.css`
— see `docs/claude/page-notes.md` for the full removal record, and the notes on the Google
verification file, `#stats` being hidden, and the legal pages' no-JS rule.

**`#pricing` shipped 2026-09-04** — a seventh tab, not a new page (see the two paragraphs
above). It states the real Free/Pro figures (pulled from `core/plans.py`'s `PLANS` table in
the bot repo — re-read that fresh before ever touching a number on this panel, a retuned
figure there moves every guild on that plan and a stale copy here would misstate what a
customer gets) and still shows **no numeric Pro price** — `.plan__price--pending` renders
"يُعلن قريبًا" and the panel carries no figure. The Pro CTA (`.btn--accent`, "ترقّى لبرو")
points at `https://dashboard.musaed.dev/pricing/get-pro` as of 2026-09-05 — a first-party
outbound link to the dashboard's own upgrade flow, same category as the «لوحة التحكم» links,
not billing scaffolded here. It replaced an earlier Discord "contact the team" link from when
no upgrade route existed. Whoever sets a real price fills in `.plan__price--pending` then.
Early Access is deliberately not listed as a Pro perk — it grants no higher limits and its
own copy says its features may vanish or turn paid.

---

## 2. Hard rules — do not violate

These came from the project owner. They are not style preferences.

**Scope: they govern this repo — the public marketing site.** The planned dashboard
(`docs/claude/dashboard.md`) is a separate application with its own security model; it is not an exception to anything below,
because nothing below is about it. Do not weaken a rule here to make dashboard work fit.

1. **This site has zero connection to the bot's production database, and must never gain
   one.** The bot enforces strict per-server tenant isolation; guild data belongs to the
   servers that generated it. A public marketing page has no business touching it.
2. **Never read environment variables.** No `DATABASE_URL`, no bot token, no secrets, and no
   code path that could consume one.
3. **Never import a database, Discord, or auth library.** No SQLAlchemy, asyncpg, discord.py,
   session handling, or OAuth flow.
4. **Never scaffold login, auth, or protected routes *in this repo*.** There is no logged-in
   state on the marketing site at all. Auth belongs to the dashboard
   (`docs/claude/dashboard.md`), which lives elsewhere — not behind a flag here.
5. **Keep it dependency-free and build-free.** Fonts and icons are vendored. If you need
   tooling (a headless browser to test with), install it *outside* this repo.
6. **All user-facing copy is Saudi-dialect Arabic. All code comments and placeholder markers
   are English.**
7. **Every invented number is marked in-code** so it is trivial to find and replace.
8. **Keep `CLAUDE.md` current with every commit.** Any change that adds or removes a panel,
   moves a number, changes a rule, or opens or closes a goal updates this file (and any
   affected `docs/claude/*.md`) as part of the same batch of commits — same commit or an
   adjacent one, never left for "later". Someone reading only this file should never be
   behind the code.

If live numbers are ever needed, they must come from a **separate, purpose-built public
aggregate endpoint** — never per-server rows, never member identities. The seam already
exists; see `docs/claude/placeholders-and-domain.md`.

---

## Where to find things

| About to… | Read |
| --- | --- |
| touch CSS/JS/markup and need anchor names, phone-menu behavior, motion effects, brand assets, or the stats-fetch implementation | `docs/claude/implementation-reference.md` |
| work on anything dashboard-adjacent (auth, settings writes, per-guild data) | `docs/claude/dashboard.md` |
| edit `#commands` or `#features`, or check the bot's command surface hasn't drifted | `docs/claude/copy-accuracy.md` |
| touch CSS/design tokens, or need the "don't accidentally break this" checklist | `docs/claude/design-and-invariants.md` |
| check the stats placeholder, the invite URL, or do a domain move | `docs/claude/placeholders-and-domain.md` |
| test/verify a change, or want the list of known testing/deploy gotchas | `docs/claude/testing-and-traps.md` |
| commit and push (and the commit checklist / CLAUDE.md-first rule), or verify a deploy actually went live | `docs/claude/git-and-deploy.md` |
| pick up open work, plan growth/distribution, or check what's still owed | `## 4. Next goals` below + `.agents/*` (gitignored, local-only) |
| need the history behind the Google-verification file, the `updates.html` removal, `#stats` being hidden, or why the legal pages load no JS | `docs/claude/page-notes.md` |

---

## 3. Working style expected here

Purely technical and direct. Explain non-obvious decisions briefly; skip the obvious ones.

**Report honestly.** If a check was not run, say so rather than implying it passed. If a
claim was measured, say what was measured. If you were wrong, correct it in a sentence and
move on — do not narrate the mistake.

**Do not invent numbers.** Every figure on this site is either real or explicitly marked as a
placeholder. If you need a count, measure it; if you cannot, mark it.

Never truncate code with placeholders or `// ... rest unchanged`.

---

## 4. Next goals — what to pick up

Ordered roughly by priority. Open as of 2026-09-05. The growth work is tracked in `.agents/`
(gitignored, **local-only — a fresh clone won't have these files**; ask the owner for them
if missing): `directory-tracker.csv` (18 listing targets; **top.gg is live as of ~2026-08-29**,
the rest not submitted), `directory-listings-copy.md` (Arabic-first listing copy,
paste-ready), `product-marketing.md` (positioning, voice, goals).

**Primary objective (owner's, from `product-marketing.md`): adoption, not revenue.**
Target is 50 servers using Musaed; the current count lives in the gitignored docs, not here.
The site's job in that funnel is the "ضيف البوت" conversion and the directory long tail.

### Open — the agent can do these

1. **Brand voice — deeper pass still open.** `product-marketing.md` calls for "friendly,
   warm, made for your people". `#why-musaed` (2026-09-05) and a first light pass on the
   hero / `#about` / `#features` / `#trust` ledes + two FAQ answers (merged 2026-09-05) are
   done. Still deadpan: the `#features`/`#trust` **card bodies**, the `#commands` and
   `#dashboard-features` prose, most FAQ answers. Warm the framing, leave the spec concrete
   (the `#why-musaed` principle); legal pages stay neutral.
2. ~~Arabic directory research.~~ **Done 2026-09-05.** Findings in `directory-tracker.csv`
   (now 27 rows) and `product-marketing.md` v4: no standalone Arabic bot-directory sites
   exist; Arabic discovery = the `arabic`/`arabic-language` tags on the big lists, Arabic
   tech-blog roundups (مجنون كمبيوتر, سماعة تك, Khamsat), Arabic YouTubers, and Disboard
   bot-list servers. "Saudi bot" ≈ صقر بوت / SaqrBot. Submitting the listings is still the
   owner's job.

### Leave alone unless asked

- **`#why-musaed` MEE6 rows** (`docs/claude/copy-accuracy.md`): `التحقق عند الدخول`
  ("تحقق أساسي") and `السعر` ("مجاني محدود؛ ميزات كثيرة مدفوعة") arguably understate how thin
  MEE6's free tier is after its paywall creep. The owner's 2026-09-05 pass did **not** touch
  these — they are claims about another product and the current wording is defensible.

### Shipped 2026-09-05

- **AI-SEO pass.** `index.html`'s JSON-LD is now a single `@graph`: `Organization`, `WebSite`,
  `SoftwareApplication`, and a `FAQPage` mirroring the six `#faq` Q&As **word-for-word** —
  edit a FAQ answer and you edit the schema too (`docs/claude/copy-accuracy.md`). Added
  `/llms.txt` (llmstxt.org overview) and `/pricing.txt` (machine-readable pricing that
  mirrors the `#pricing` panel's numbers). A visible "آخر تحديث `<time>`" line in the footer
  — **bump its date on any real content change to the landing page.** `robots.txt` already
  allows every AI crawler (`* / Allow: /`), no change needed. Still open (Tier 3): a real
  `<table>` for `#why-musaed` (blocked by the no-`<table>` design lock), and a fuller
  `/pricing.txt` once a numeric Pro price exists.
- JSON-LD `SoftwareApplication` on `index.html` — Arabic name/description,
  `isAccessibleForFree`, five-item `featureList`, no ratings or install counts. Adds three
  `musaed.dev` absolute URLs (`docs/claude/placeholders-and-domain.md`).
- `assets/Pics/musaed-avatar.svg` — self-contained outlined «م» (IBM Plex Sans Arabic 700,
  pulled from the vendored woff2, no external font `@import`). Matches the PNG mark; not
  wired into any page. Regenerate with `fonttools` if the glyph needs adjusting.
- `sitemap.xml` `lastmod` dates refreshed; community invite link updated on every page.
- First warm-voice pass merged: hero sub, `#about` heading, `#features`/`#trust` ledes,
  two FAQ answers. Ledes and framing only; card bodies left concrete.
- Western numerals site-wide; Dyno verdict de-contradicted against its own table; the
  `#why-musaed` switch got its `vs-in` fade.
- The `#pricing` and `#why-musaed` panels were committed (they had been sitting uncommitted
  in the working tree since early September).

### Settled — do not reopen

- **No standalone comparison pages.** The owner rejected `/alternatives/*.html` on 2026-09-05.
  The MEE6/Dyno comparison stays as the single `#why-musaed` tab. A draft `alternatives/mee6.html`
  was built and deleted; do not resurrect it. The SEO tradeoff (a hash tab won't rank for
  "بديل MEE6") is known and accepted.
- **Numerals are Western (0-9) everywhere.** Converted site-wide 2026-09-05 — hero facts,
  duration chips, uptime label, `privacy.html`. This is final; do not reintroduce
  Arabic-Indic digits. (Copy may still *describe* the bot accepting both as duration input —
  that is a real bot capability, not a site-chrome choice.)
- **The systems count is 8**, stated as `8 أنظمة` (hero) and `ثمانية أنظمة` (`#features` lede).
  `#features` has 9 cards on purpose: 8 systems + the "أوامر واضحة" meta-card. The bot's 9th
  system (honeypot trap channel, `قناة الفخ`) **stays off this site** — see `docs/claude/copy-accuracy.md`.

### Not the agent's to do

Directory submissions themselves (discordbotlist, disboard, …) need the bot's Discord login
and manual form work — owner tasks. **top.gg is already done** (live ~2026-08-29).
`discord.bots.gg` is blocked until Discord verification (~100 servers). Product Hunt is
deferred. Arabic directories need a name-research pass first. See `directory-tracker.csv`
for the per-site state.
