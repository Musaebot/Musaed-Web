# CLAUDE.md — working on Musaed-Web

Orientation for an AI agent picking this repo up — read this before you touch anything.
`README.md` is a short plain-language page for human visitors (what the project is, how to
run it) and is not a technical reference — don't look there for history, reasoning, or
implementation detail.

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
moderation/automod Discord bot. Four pages. It explains the bot to server owners before they
add it.

It is **not** a dashboard, and nothing on it manages anything or shows real guild data —
this repo itself has no login state, no account, no backend. The two login buttons (nav +
hero) are outbound links to the dashboard's own OAuth2 flow, a **separate application on a
separate origin** (see `docs/claude/dashboard.md`) — same category of link as the Discord bot-invite buttons, not
auth scaffolded in this repo.

```text
index.html                 landing page + its inline icon sprite (22 symbols)
404.html                   custom error page, served by Caddy - see docs/claude/testing-and-traps.md
connect.html               pre-auth disclosure screen before the dashboard login - see docs/claude/placeholders-and-domain.md
privacy.html               privacy policy   } same layout, one shared
terms.html                 terms of use     } stylesheet, NO JavaScript
google82b70d7af988f7a9.html  Google Search Console site-verification file - see docs/claude/page-notes.md
sitemap.xml                lists the 3 real pages (not connect.html, not 404.html) - see docs/claude/placeholders-and-domain.md
robots.txt                 Allow: / for everyone, points at sitemap.xml - see docs/claude/placeholders-and-domain.md
assets/css/styles.css      tokens, reset, shared components
assets/css/legal.css       connect.html + BOTH legal pages, loaded after styles.css
assets/js/main.js          stats data, count-up, scroll reveals, nav, link guard
assets/fonts/              self-hosted woff2 (IBM Plex Sans Arabic + Plex Mono)
assets/Pics/               brand marks. Capital P — Linux hosts are case-sensitive
```

Four pages, two stylesheets, one script. **Zero dependencies, zero build step.** There used
to be more: `developer.html` was removed along with its CSS and JS, and do not resurrect it.
`updates.html` (the changelog) was removed on 2026-08-25 along with `assets/css/updates.css`
— see `docs/claude/page-notes.md` for the full removal record, and the notes on the Google
verification file, `#stats` being hidden, and the legal pages' no-JS rule.

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
| commit and push, or verify a deploy actually went live | `docs/claude/git-and-deploy.md` |
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
