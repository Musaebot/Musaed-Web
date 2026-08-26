# Placeholders — everything still open

**Servers and members went live** when `/api/public/stats` shipped on the
dashboard (`../Musaed-Dashboard/app/routers/public.py`) — `STATS_ENDPOINT` in
`main.js` now points at it and `#stats` is no longer `hidden`. Uptime is the
one still open: the endpoint has no `uptime_30d` field yet (it would need a
real history of heartbeats to compute honestly, not a snapshot), so that one
`.stat` in `index.html` stays individually `hidden` rather than showing
against an undefined value.

| What | Where | Marker |
| --- | --- | --- |
| 1 statistic (uptime) | `STATS.uptime` in `main.js`, its `.stat` in `index.html` | `// placeholder`, `data-mock="true"` |
| _(none — every text placeholder is filled)_ | | |

```bash
grep -rn "data-mock" index.html               # 1 — the uptime .stat only
grep -rn "data-placeholder-link" index.html   # 0 — every link is real now
grep -c  "oauth2/authorize" index.html        # 4 — the live invite, must be identical
grep -n  "STATS_ENDPOINT" assets/js/main.js   # now the real dashboard URL
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

**The bot invite is live and appears four times in the DOM** — sidebar footer, phone header,
desktop topbar, hero — each opening in a new tab with `rel="noopener noreferrer"`. It was
three (nav, hero, about) until the 2026-08-25 rebuild. Two of the four are mutually exclusive
by media query, so a visitor sees three on desktop and two on a phone; see
`docs/claude/implementation-reference.md` for which is which. There is no single source for the URL, so a change to
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

**Live network surface: exactly one `fetch`, and it is unreachable.** `readStats()` returns
the local `STATS` object while `STATS_ENDPOINT` is `null`. The rate-limiting path (TTL
throttle, 429 backoff honouring `Retry-After`, last-good fallback) is already written; wiring
a real endpoint is a one-constant change. The declaration is space-aligned, so
`grep "STATS_ENDPOINT ="` with one space matches nothing and looks like the guard is gone.

## The hardcoded domain

`https://musaed.dev` — **16 absolute URLs, 4 per page across 4 pages**: `canonical`, `og:url`,
`og:image`, `twitter:image`. Crawlers reject relative URLs, so it is written literally.
(`404.html` carries none of the four and is not part of the sixteen — an error page has no
canonical address. This was 20 URLs across 5 pages before `updates.html` was removed
2026-08-25 — see `docs/claude/page-notes.md`.)

**These moved off `musaed.up.railway.app` on 2026-08-08.** That host served the site too and
always had, so nothing was broken — which is exactly why it went unnoticed: two live hosts
serving identical content, with `canonical` pointing every crawler and social card at the
unbranded one, so `musaed.dev` was published as the duplicate of its own Railway URL. A
domain move that leaves the site *working* at both names is the easy one to leave half-done.

**`musaed.up.railway.app` is retired as of 2026-08-18 and must not be referenced again.**
The generated Railway service domain may still answer while it remains attached to the
service, but it is not part of how this site is served, linked, or verified. It used to have
one real job — reading past Cloudflare to check a CSS/JS deploy — and
`docs/claude/testing-and-traps.md` now does that with a cache-busting query string instead,
which needs no second host. Treat any surviving mention as stale.

```bash
grep -ohE 'https://[a-z0-9.-]+' *.html | sort -u
```

That must print exactly four hosts: the site's, `discord.gg` for the community invite
(`https://discord.gg/QvNXvDDFtz` — **live, not a placeholder**), `discord.com` for the
bot-invite URL and the link to Discord's own terms, and
`dashboard.musaed.dev` — the dashboard (a separate deployable, `docs/claude/dashboard.md`).

**The dashboard host appears 5 times as of 2026-08-25, all in `index.html`.** Its history is
worth knowing, because the shape keeps changing under it:

| When | Count | Where |
| --- | --- | --- |
| until 2026-08-11 | 1 | `connect.html` only — the login buttons pointed at *that page*, not at the host |
| 2026-08-11 | 5 | plus 3 `.cmdgroup__note` lines and 1 in `#dashboard-features` |
| 2026-08-25 (rebuild) | 4 | the flat command list collapsed the 3 notes into 1; `#faq` added 1 |
| 2026-08-25 (connect removed) | **5** | `connect.html` deleted, its 1 gone — but both login buttons now carry the host directly |

Today that is **2 login hrefs** (`https://dashboard.musaed.dev/auth/login` — the sidebar
«لوحة التحكم» button and the topbar icon link) plus **3 informational links** (`.cmds__note`,
`#dashboard-features`, and the "وين أضبط الأنظمة؟" FAQ answer). The informational three stay
bare `https://dashboard.musaed.dev` on purpose: they are "read more about this feature"
pointers, not "log in" CTAs. If the dashboard host ever moves, **grep for the host, not for
`auth/login`** — all five live in one file now, but they are two different kinds of link and
only the two login ones carry a path. The 12 `canonical`/`og:url`/`og:image`/`twitter:image`
URLs are a separate set pointed at this site own host exclusively, unaffected by any of it.

**`connect.html` was removed on 2026-08-25** — see `docs/claude/page-notes.md` for the full
record. It was the pre-auth disclosure screen the two login buttons used to pass through: a
**content page, not auth** — no session, no OAuth, no protected state — so root
`CLAUDE.md` Hard Rule 4 was intact before and is intact now. There is simply one fewer page,
and the buttons link straight out.

**The disclosure itself did not disappear with it.** The dashboard serves its own copy at
`/auth/connect` and always has, because `/auth/login` is a public URL that a bookmark or a
stale third-party link reaches without passing through this site at all. That copy
(`app/templates/connect.html` in the Musaed-Dashboard repo) is now the only one — so the
**"keep the two wordings in sync" obligation this section used to carry no longer applies
here**, because there is nothing left in this repo to sync it against. If you are ever asked
to restore a pre-auth screen on the marketing site, check first whether the dashboard own
screen is considered insufficient: duplicating it is exactly what created the sync burden.

**The dashboard's host changed on 2026-08-08** (it was a generated `*.up.railway.app` name)
and the old one was **deleted, not left redirecting** — so both login buttons pointed at a
dead host until they were updated. That is the same failure this section warns about below,
one host over: nothing in this repo breaks visibly when a *different* deployable moves, because
the links still render fine and only fail on click. If the dashboard host ever moves again,
grep for the host, not for `auth/login` — the two `href`s carrying it now live in `index.html`.

A domain move is **not** done when the site loads at the new address; it is done when all
twelve URLs are updated — twenty before `updates.html` was removed, sixteen until
`connect.html` went the same way on 2026-08-25 (see `docs/claude/page-notes.md`). This has now
bitten twice, in both directions: once when an old host
404'd and half the site still pointed at it, so social cards referenced a dead image — and
again on 2026-08-08, when the old host kept working and nothing looked wrong at all. The first
kind announces itself. The second does not, so grep the host after every move rather than
trusting the site to tell you.

## sitemap.xml and robots.txt

`sitemap.xml` lists exactly the three real, indexable pages — `/`, `/privacy.html`,
`/terms.html` — each with a `<lastmod>` taken from that file's last commit date, not
invented. **It now lists every page the site has except `404.html`**, excluded because an error page
has no canonical address — same reasoning as the `canonical`/`og:url` exclusion above. It
used to deliberately exclude a second one, `connect.html` (a functional pre-auth step, not
content someone should land on from a search result); that page was removed outright on
2026-08-25, so the exclusion is moot. `updates.html` was a third (unlinked from nav/footer
but still live and real — `docs/claude/page-notes.md`) until it too was removed the same day,
taking its sitemap entry with it. **Neither removal needed a sitemap edit** — `connect.html`
was never listed, and `updates.html` was the only one that had been.

`robots.txt` allows everything and points at the sitemap. Both files hold absolute
`musaed.dev` URLs but are **not** `.html`, so they are invisible to the `grep -ohE
'https://[a-z0-9.-]+' *.html` check above — a future domain move has to update these two
files by hand as well, or the sitemap will keep advertising the old host to crawlers after
every page's own `canonical` has already moved on.

`dashboard.musaed.dev` gets no robots.txt of its own from this repo — that app is a separate
deployable (`docs/claude/dashboard.md`) and staying unindexed is tracked as a separate,
smaller task there, not here.
