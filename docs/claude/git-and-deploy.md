# Git and deploy

Work happens on **`main`**, which is what Railway deploys. Commit and push only when asked.

**Before you commit, `CLAUDE.md` has to be current** (hard rule 8). If the change adds or
removes a panel, moves a number, changes a rule, or opens/closes a goal, update `CLAUDE.md`
and any affected `docs/claude/*.md` first — same commit, or the one right before.

Keep commit messages plain: one line saying what changed, in terms anyone could follow, no
jargon and no `Co-Authored-By` trailer. Split unrelated changes into separate commits.

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

**Then verify the live deploy on `musaed.dev` with a cache-busting query string if the change
touched CSS or JS** — `docs/claude/testing-and-traps.md` has the exact commands. Cloudflare caches those for up to 4 hours on
the branded domain and a redeploy doesn't purge it, so the plain URL can look unchanged (or
half-changed) right after a push that fully succeeded. Check `last-modified` against the
deploy time, not just whether the page renders. Plain `musaed.dev` is fine for HTML-only
changes, which are never cached.

Also confirm the push actually triggered a Railway deploy — it doesn't always. `list-deployments`
should show a fresh entry with `reason: "deploy"` and the new commit hash within a few
minutes. If it doesn't, `redeploy` will **not** fix it (it re-runs the last known build, not
the latest commit — confirmed 2026-08-11); push an empty commit (`git commit --allow-empty`)
to re-trigger the GitHub App event instead.
