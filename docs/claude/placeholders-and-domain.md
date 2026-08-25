# Placeholders — everything still open

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

**The dashboard host appears 5 times as of 2026-08-11: once in `connect.html`, four times in
`index.html`.** Until 2026-08-11 it was referenced exactly once, in `connect.html` — the two
login buttons in `index.html` (nav + hero) point at `connect.html`, the pre-auth disclosure
screen, whose "فهمت، كمّل" button is the single outbound link to the dashboard, and that was
the *only* place the host string appeared. The `#commands` allow-list rewrite
(`docs/claude/copy-accuracy.md`) added four
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
root `CLAUDE.md`'s Hard Rule 4 is intact. The dashboard serves its own copy of the same disclosure at `/auth/connect`,
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
sixteen URLs are updated (it was twenty before `updates.html` was removed — see
`docs/claude/page-notes.md`). This has now
bitten twice, in both directions: once when an old host
404'd and half the site still pointed at it, so social cards referenced a dead image — and
again on 2026-08-08, when the old host kept working and nothing looked wrong at all. The first
kind announces itself. The second does not, so grep the host after every move rather than
trusting the site to tell you.

## sitemap.xml and robots.txt

`sitemap.xml` lists exactly the three real, indexable pages — `/`, `/privacy.html`,
`/terms.html` — each with a `<lastmod>` taken from that file's last commit date, not
invented. **Deliberately excludes two pages that otherwise look like they belong:**
`connect.html` (a functional pre-auth step, not content someone should land on from a search
result) and `404.html` (an error page has no canonical address — same reasoning as the
`canonical`/`og:url` exclusion above). `updates.html` used to be a third deliberate
inclusion (unlinked from nav/footer but still live and real — `docs/claude/page-notes.md`) until it was removed
outright on 2026-08-25 — its sitemap entry went with it.

`robots.txt` allows everything and points at the sitemap. Both files hold absolute
`musaed.dev` URLs but are **not** `.html`, so they are invisible to the `grep -ohE
'https://[a-z0-9.-]+' *.html` check above — a future domain move has to update these two
files by hand as well, or the sitemap will keep advertising the old host to crawlers after
every page's own `canonical` has already moved on.

`dashboard.musaed.dev` gets no robots.txt of its own from this repo — that app is a separate
deployable (`docs/claude/dashboard.md`) and staying unindexed is tracked as a separate,
smaller task there, not here.
