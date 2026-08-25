# Per-page notes

**`google82b70d7af988f7a9.html` is not a page.** It is the file Google Search Console issues
for the HTML-file verification method: single line of plain text, no doctype, no `<html>`.
It has to be reachable at exactly `https://musaed.dev/google82b70d7af988f7a9.html` — root of
the domain, filename unchanged — for Google to accept the verification, so it deploys as-is
alongside the real pages. Nothing links to it and nothing should; it is not part of site
navigation, the same way `404.html` isn't.

**`updates.html` (the changelog) was removed entirely on 2026-08-25.** It had been unlinked
since 2026-08-06 — the nav link (`التحديثات`) and its footer counterpart in the `النظام`
column were deleted from `index.html`, along with the `#i-sparkle` sprite symbol they used
(22 → 21 symbols at the time; back to 22 once `#i-chat-text` was added 2026-08-17,
`docs/claude/copy-accuracy.md`) — but
the page itself, `assets/css/updates.css`, and its own 1-symbol sprite kept deploying and
stayed reachable at `/updates.html` for anyone with the direct URL. The owner had it deleted
outright rather than left as a dead-but-live page: nothing linked to it, and an unlinked page
that stays crawlable indefinitely is more a liability than a convenience. Removed alongside
it: its `sitemap.xml` entry and its four `canonical`/`og:url`/`og:image`/`twitter:image`
URLs (`docs/claude/placeholders-and-domain.md`'s hardcoded-domain count dropped from 20 to 16
accordingly). There is nothing left
to relink — a changelog page would have to be rebuilt from scratch, including the `.upbar` /
`.uppanel` chrome and the `--up-*` local tokens `updates.css` used to carry.

**`#stats` (`مساعد بالأرقام`) is hidden, not removed.** As of 2026-08-10 the section carries
a `hidden` attribute, and its nav link (`الأرقام`) and footer counterpart in the `الموقع`
column were both deleted from `index.html`. `#stats` lives inline on `index.html` itself —
so unlinking the nav anchor alone would not have stopped it from rendering as you scroll; the
`hidden` attribute is what actually does that. **Since the 2026-08-25 rebuild it sits inside
the `#top` panel**, below `#about`, and is doubly hidden: the panel is only rendered on the
البداية tab, and the section still carries `hidden` on top of that. To bring it back, remove
the attribute — there is no nav `<a>` or footer `<li>` to restore any more, and it would show
on the landing tab rather than needing its own. The markup, its
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

**`Musaed Site.html` in the repo root is a design source, not a page.** It is the exported
design-canvas bundle the 2026-08-25 landing-page rebuild was implemented from: a 754KB
single-file artifact carrying React 18 and 30 Google-hosted woff2 files base64'd into a
manifest, which a runtime unpacks into blob URLs at load. **It must not deploy.** A static
host serves the repo root verbatim, so leaving it there publishes a second, broken,
React-dependent copy of the site at `/Musaed%20Site.html` — and it drags
`unpkg.com`/`fonts.googleapis.com`/`fonts.gstatic.com` into the
`grep -ohE 'https://[a-z0-9.-]+' *.html` host check in
`docs/claude/placeholders-and-domain.md`, which is supposed to print exactly four hosts and
currently prints seven. Nothing in the shipped site ever referenced it, and the design it encodes is
fully implemented in `index.html` in vanilla HTML/CSS/JS. **It was deleted the same day,
before it was ever committed**, so it exists in no history and the host check is back to
four. If another design export lands in the repo root, treat it the same way: it is a source
artifact, not a page, and a static host cannot tell the difference.

**`connect.html` (the pre-auth disclosure) was removed on 2026-08-25**, the same day as the
landing-page rebuild and on the owner's instruction. It sat between the site's two login
buttons and the dashboard's OAuth flow, explaining what Discord login would read (name,
avatar, id; the guild list scoped to what you can manage; role in the official server;
role/channel names; the account email, for Pro-lapse notice) and what it would not.

**Removing it did not remove the disclosure from the journey.** The dashboard serves its own
copy at `/auth/connect` and always has, because `/auth/login` is a public URL reachable by
bookmark without passing through this site — see
`docs/claude/placeholders-and-domain.md`, which also records that the
"keep the two wordings in sync" obligation dies with this page. The sidebar «لوحة التحكم»
button and the topbar icon link now point straight at
`https://dashboard.musaed.dev/auth/login`.

Removed alongside it:

- its four `canonical`/`og:url`/`og:image`/`twitter:image` URLs, dropping the hardcoded-domain
  count from 16 to **12** across **3** pages;
- `.doccta` and `.notice` in `assets/css/legal.css` — `connect.html` was the only page that
  ever used either, so both were dead the moment it went. `legal.css`'s `var(--accent)` count
  fell 8 → **7** with `.notice`;
- its own 1-symbol inline sprite (`#i-back`), which was local to the file. It did **not**
  orphan anything in `index.html`'s 22-symbol sprite — the two are separate, and the legal
  pages carry their own copy of `#i-back`, so `privacy.html` and `terms.html` are unaffected.

**No `sitemap.xml` edit was needed** — `connect.html` was deliberately never listed (a
functional pre-auth step is not a search-result landing page), which is the one way this
removal was cheaper than `updates.html`'s. `legal.css` still loads on both legal pages and
is otherwise untouched.

If a pre-auth screen is ever wanted here again, read the placeholders doc first: the reason
this one existed was that `/auth/login` is publicly reachable, and the dashboard already
solves that on its own side.
