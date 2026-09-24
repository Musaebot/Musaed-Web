# CLAUDE.md — working on Musaed-Web

Orientation for an AI agent picking this repo up — read this before you touch anything.
`README.md` is a short plain-language page for human visitors (what the project is, how to
run it) and is not a technical reference — don't look there for history, reasoning, or
implementation detail.

**The site was completely rebuilt on 2026-09-24.** This was not an incremental edit — it
replaces the previous sidebar-plus-tab-panels design (eight/nine-panel `index.html`, separate
`assets/css/styles.css` written for that layout) with a different landing page built
independently by the owner in a separate working folder ("the gatekeeper" redesign) and then
brought into this repo, in this same batch of changes, to keep it under version control. Read
`docs/claude/implementation-reference.md` for the full technical reference — sections, brand
tokens, the i18n system, gotchas, content rules, real links/live data, legal-page structure,
accessibility and how to verify changes. It is detailed and accurate as of this rebuild;
treat it as the primary implementation doc going forward.

**Bilingual now — this is a deliberate reversal of the old Arabic-only rule.** Arabic is
still the default and primary language (loads first, matches the bot's own voice), but
English is a genuine one-click toggle (`main.js`/`i18n.js`), not a translation afterthought.
See Hard Rule 6 below — it changed to reflect this.

**What reversed from the previous design, so you don't assume continuity that isn't there:**
- The `#why-musaed` MEE6/Dyno comparison panel is **gone entirely** — not just off the nav,
  removed from the page. The old CLAUDE.md called keeping it as a single tab "Settled — do
  not reopen"; the rebuild reopened it by replacing the whole page. If comparison content is
  wanted again, that's a new decision, not a restore.
- The `#pricing` panel's numeric Free/Pro breakdown (the six-figure table pulled from
  `core/plans.py`) is gone from the page itself. The new `#plans` section shows Free's
  features as plain bullets and Pro as a non-clickable **"Coming soon"** pill — no numbers,
  no dashed-border CTA, no link to `dashboard.musaed.dev/pricing/get-pro` (that route was
  already turned into a "not available yet" modal on 2026-09-17; the new design goes further
  and doesn't link to it at all). `pricing.txt` still carries the granular numeric limits
  from the bot's real plans table as a machine-readable detail beyond what the page itself
  shows now — see "SEO/AI-facing files" below.
- The systems count ("9 أنظمة") is no longer stated anywhere on the page. The new `#protection`
  section has four layers (age gate, captcha, honeypot, automod) and `#toolkit` has six cards
  (tickets, shortcuts, auto-responses, welcome, mod logs/`/lookup`, member DMs). Don't
  reintroduce a "9 systems" claim without checking it still matches the bot's actual feature
  set (`docs/claude/copy-accuracy.md`'s underlying facts may still be right — its specific
  panel/count claims are not, see the staleness note there).

**What carried over unchanged:** brand colors (`--accent: #0fe37d`, dark-only background
scale), the invite link and its `permissions=8` scope, the dashboard login URL, the support
server link, and — word for word, diffed against the previous versions — the full text of
`privacy.html` and `terms.html`. Only their layout changed (sticky table-of-contents sidebar
on desktop / horizontal chip bar on phones, a reading-progress bar, `legal.css`/`legal.js`).

**Fonts are now self-hosted, not loaded from Google Fonts.** The owner's original build (in
the separate working folder) loaded Alexandria, Readex Pro and IBM Plex Mono from
`fonts.googleapis.com`/`fonts.gstatic.com` — that both violates Hard Rule 5 below and would
have contradicted `privacy.html`'s "only two outside services" claim. As part of bringing it
into this repo, all three families were vendored as `.woff2` under `assets/fonts/`
(`alexandria-arabic/latin.woff2`, `readexpro-arabic/latin.woff2`, `plex-mono-400/500.woff2`),
with matching `@font-face` rules at the top of `assets/css/styles.css`, and every Google
Fonts `<link>` removed from all four HTML pages. The previous design's font files
(`plex-arabic-*.woff2`, `plex-latin-*.woff2` — IBM Plex Sans Arabic, unrelated to Alexandria/
Readex Pro) were deleted since nothing references that family anymore.

**JSON-LD was ported forward, not carried over verbatim.** The previous `index.html` had an
`Organization`/`WebSite`/`SoftwareApplication`/`FAQPage` `@graph`. The new `index.html` has
the same four nodes, rebuilt to match this page: `SoftwareApplication.description` and
`featureList` reflect the new copy, and `FAQPage.mainEntity` mirrors the new page's six
`#faq` questions word-for-word (same discipline as before — **edit a FAQ answer in `i18n.js`
and `index.html`, and you edit the JSON-LD too**).

**This file is committed to a public GitHub repo.** Do not put deployment internals in it —
no infrastructure identifiers, no environment variable names, no bot-side table or module
names, no server IDs. That rule applies to everything you write into this repo.

---

## 1. What this is

A **static public marketing site** for **مساعد (Musaed)**, an Arabic (Saudi dialect)
moderation/automod Discord bot, bilingual with English as a toggle. Three pages: a
long-scrolling `index.html` (thirteen sections, real anchors — nav links, footer links and
the FAQ all point at `#protection`, `#toolkit`, `#dashboard`, `#commands`, `#plans`, `#faq`
etc.), plus `privacy.html` and `terms.html`. It explains the bot to server owners before
they add it.

It is **not** a dashboard, and nothing on it manages anything or shows real guild data —
this repo itself has no login state, no account, no backend. The nav's "دخول اللوحة"/"Log in"
link and the dashboard-preview section's CTA are plain outbound links to the dashboard's own
OAuth2 flow, a **separate application on a separate origin** (see `docs/claude/dashboard.md`)
— same category of link as the Discord bot-invite buttons, not auth scaffolded in this repo.

```text
index.html                 landing page: 13 sections, inline icon sprite (SVG symbols), a JSON-LD @graph, bilingual (ar default, en toggle)
404.html                   custom error page, served by Caddy, no JS, Arabic-first with an English line
privacy.html               privacy policy   } same layout, one shared
terms.html                 terms of use     } stylesheet + legal.css, legal.js drives the reading-progress bar and TOC highlighting
google82b70d7af988f7a9.html  Google Search Console site-verification file
sitemap.xml                 lists all 3 real pages (not 404.html)
robots.txt                  Allow: / for everyone, points at sitemap.xml
llms.txt                    plain-text overview for AI systems (llmstxt.org format) — updated 2026-09-24 for the rebuild
pricing.txt                 machine-readable pricing for AI agents; more detailed than the on-page #plans card now — updated 2026-09-24
assets/css/styles.css       @font-face rules + tokens, reset, every component. Linked as ?v=5 — bump on change
assets/css/legal.css        privacy.html/terms.html only, loaded after styles.css. ?v=2
assets/js/main.js           language switch, live stats, nav, join-gate demo, reveals, stacked protection cards, dashboard preview, command search, scroll-linked effects. ?v=3
assets/js/i18n.js           the Arabic dictionary, bilingual runtime strings (dyn), join-gate demo accounts, the slash-command list. ?v=1
assets/js/legal.js          privacy.html/terms.html only: reading-progress bar, TOC highlighting, back-to-top. ?v=1
assets/fonts/                self-hosted woff2: Alexandria + Readex Pro (arabic/latin subsets), IBM Plex Mono (400/500)
assets/Pics/                 brand marks, including the new musaed-favicon.svg. Capital P — Linux hosts are case-sensitive
```

Three pages, two stylesheets, three scripts. **Zero dependencies, zero build step.**

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
5. **Keep it dependency-free and build-free.** Fonts and icons are vendored — no CDN font
   links (Google Fonts included), even for a quick prototype. If you need tooling (a headless
   browser to test with), install it *outside* this repo.
6. **Arabic is the default and primary language; English is a genuine toggle, not scope
   creep.** This reverses the old "Arabic-only" rule — the 2026-09-24 rebuild made the site
   bilingual on the owner's design, and that stands. What still holds: all *Arabic* copy is
   Saudi-dialect ("وش", "تبي", "على طول", "الحين" — see `docs/claude/implementation-reference.md`
   §Language), all code comments and placeholder markers are English, and the legal pages
   (`privacy.html`/`terms.html`) stay **Arabic-only** — don't add an English translation of
   them without the owner's explicit sign-off (see that doc's "Open items").
7. **Every invented number is marked in-code** so it is trivial to find and replace. There
   are none left as of this rebuild — the only figures on the page are the bot's real invite
   permissions, and the live server/member count fetched from the dashboard's public stats
   endpoint (hidden until real data arrives, never faked; see
   `docs/claude/implementation-reference.md` §"Real links and live data").
8. **Keep `CLAUDE.md` current with every commit.** Any change that adds or removes a
   section, moves a number, changes a rule, or opens or closes a goal updates this file (and
   any affected `docs/claude/*.md`) as part of the same batch of commits — same commit or an
   adjacent one, never left for "later". Someone reading only this file should never be
   behind the code.

If live numbers are ever needed beyond what's already wired, they must come from a
**separate, purpose-built public aggregate endpoint** — never per-server rows, never member
identities. That seam now exists and is live (`STATS_ENDPOINT` in `assets/js/main.js`); see
`docs/claude/implementation-reference.md`.

---

## Where to find things

| About to… | Read |
| --- | --- |
| touch any part of the current implementation — section anchors, the i18n system, brand tokens, motion, the join-gate demo, real links/live stats, legal-page structure, accessibility, or how to verify a change | `docs/claude/implementation-reference.md` — accurate as of the 2026-09-24 rebuild |
| work on anything dashboard-adjacent (auth, settings writes, per-guild data) | `docs/claude/dashboard.md` — unaffected by the rebuild, still accurate |
| pick up open work, or check what's still owed from the rebuild | `## 4. Next goals` below |
| check something in `docs/claude/copy-accuracy.md`, `design-and-invariants.md`, `git-and-deploy.md`, `page-notes.md`, `placeholders-and-domain.md`, or `testing-and-traps.md` | **Read the banner at the top of that file first.** They describe the pre-2026-09-24 tabbed-panel implementation in detail — section IDs, CSS custom-property counts, `grep` counts for the old invite/link occurrences — almost all of which no longer match. They're kept for history, not as a current reference. A proper rewrite of each is still open work (see below). |

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

Open as of 2026-09-24, right after the rebuild landed in this repo.

### Open — carried from the rebuild's own handoff notes

1. **Not yet deployed.** This swap put the new site in the working tree and (once committed)
   git history, but nobody has pushed it live at `musaed.dev` yet. That's the owner's call —
   don't deploy without being asked.
2. **English legal pages** — only if the owner wants them and will review the translation.
   `privacy.html`/`terms.html` stay Arabic-only until then (Hard Rule 6).
3. **When Pro actually launches:** replace the `#plans` "Coming soon" ribbon/pill with a real
   CTA, and update `plans.ribbon`, `plans.proCta` and FAQ 5 in both languages (English inline
   in `index.html`, Arabic in `assets/js/i18n.js`) — and the matching FAQ entry in
   `index.html`'s JSON-LD `@graph`.
4. **A `<noscript>` fallback is still missing.** Without JS, `index.html` shows its English
   source text inside a right-to-left layout — a real rough edge, not fixed by this
   integration.
5. **Invite permissions** are still `permissions=8` (Administrator) — unchanged from before
   the rebuild. If asked to "clean this up" to a least-privilege set, confirm with the owner
   first; a previous session recorded this as a deliberate choice, not drift.

### Open — from bringing the rebuild into this repo

6. **The six stale `docs/claude/*.md` files need a real rewrite pass**, not just the
   staleness banners they got in this batch. Each one currently describes the pre-rebuild
   implementation in detail (old section IDs, old CSS class names, `grep` counts that no
   longer hold). `docs/claude/implementation-reference.md` is the one exception — it was
   replaced outright with accurate reference material for the current build.
7. **`README.md` and `docs/claude/dashboard.md`'s "sequencing" note weren't re-checked in
   depth.** `README.md` looked generic enough not to need edits on a skim; `dashboard.md`
   still frames the live-stats endpoint as a "first step yet to be wired" — it's already
   wired in `assets/js/main.js`, so that framing needs a line fixing next time someone is in
   that file, even though its core architecture content is unaffected by the rebuild.
8. **Verification so far: automated only.** A headless-Edge check (both languages, 1440px
   and 390px, all four pages) showed zero console errors, no sideways scroll, correct
   per-page/per-language `<title>`, and fonts rendering from the new self-hosted files —
   screenshots were reviewed and matched expectations. What was **not** done: clicking
   through the interactive pieces by hand (the join-gate demo's full run, the dashboard
   preview's controls, the command search, the language toggle's live behavior, the legal
   pages' TOC-highlight-while-scrolling). Do that before calling this fully verified.

### Settled — do not reopen without the owner

- **The redesign's content and structure are the owner's decisions**, made in a separate
  working session before this integration. Don't second-guess the removal of `#why-musaed`,
  the simplified `#plans` card, or the bilingual toggle — those were deliberate, not
  something this integration pass introduced.
- **Numerals stay Western (0-9).** Unchanged from before the rebuild.
