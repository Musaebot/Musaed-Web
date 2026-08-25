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
`hidden` attribute is what actually does that. The markup, its
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
