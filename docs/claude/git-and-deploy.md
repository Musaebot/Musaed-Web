# Git and deploy

Work happens on **`main`**, which is what Railway deploys. Commit and push only when asked.

**Before you commit, `CLAUDE.md` has to be current** (hard rule 8). If the change adds or
removes a panel, moves a number, changes a rule, or opens/closes a goal, update `CLAUDE.md`
and any affected `docs/claude/*.md` first — same commit, or the one right before.

Keep commit messages plain: one line saying what changed, in terms anyone could follow, no
jargon and no `Co-Authored-By` trailer. Split unrelated changes into separate commits.

**If you touched `assets/css/*` or `assets/js/*`, bump the `?v=N` on every `<link>` / `<script>`
that points at it** — `styles.css?v=N` in all four HTML pages, `legal.css?v=N` in the two
legal pages, `main.js?v=N` in `index.html`. The HTML is never CDN-cached, so a new `?v=`
makes Cloudflare fetch the asset fresh on the next load; without it, users get the old
CSS/JS for up to 4 hours (`max-age=14400`) after a deploy and a redeploy does not purge it.
One counter, bumped together. (Fonts are not versioned — they don't change without a rename.)

Before pushing, scan the tree — not just the diff:

```bash
grep -rniIE "DATABASE_URL|BOT_TOKEN|api[_-]?key|secret" --include="*.html" --include="*.css" --include="*.js" .
grep -rnIE "console\.(log|debug)|debugger|localhost|TODO|FIXME" --include="*.html" --include="*.css" --include="*.js" .
```

Both should come back empty. The scan above deliberately excludes `.md` files, since prose
discussing what a token or secret *isn't* present would otherwise false-positive.

After a push, verify against the **remote**, not just locally:

```bash
git ls-remote origin refs/heads/main    # must equal git rev-parse HEAD
```

**Then verify the live deploy on `musaed.dev`.** For HTML-only changes the plain URL is fine
(HTML is never cached). For CSS/JS, check the versioned URL you just bumped
(`musaed.dev/assets/css/styles.css?v=N`) returns the new file — that is the URL the new HTML
actually requests. The *un*-versioned `styles.css` will stay stale in Cloudflare for up to 4
hours and that is now expected and harmless. `docs/claude/testing-and-traps.md` has the
commands. If you forgot to bump `?v=` and need it live now, the only fix is a Cloudflare
dashboard purge (Caching → Purge) — a redeploy will not do it.

Also confirm the push actually triggered a Railway deploy — it doesn't always. `list-deployments`
should show a fresh entry with `reason: "deploy"` and the new commit hash within a few
minutes. If it doesn't, `redeploy` will **not** fix it (it re-runs the last known build, not
the latest commit — confirmed 2026-08-11); push an empty commit (`git commit --allow-empty`)
to re-trigger the GitHub App event instead.
