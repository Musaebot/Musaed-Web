# Where this is going — the dashboard

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

**Read this before you read root `CLAUDE.md`'s Hard Rules, because it looks like a
contradiction and is not.**

> The dashboard is a **separate application**. It does not relax a single Hard Rule.
> Those rules govern *this repo* — a public marketing page served to anonymous visitors.

## What that means concretely

- **Do not add auth, a backend, a database client, or a build step to this repo** in order to
  get there. The marketing site stays static. A dashboard is a different deployable with its
  own service, its own origin or subdomain, and its own threat model. Mixing them means the
  marketing page inherits a login surface it has no reason to have.
- **The marketing site keeps showing zero real guild data even after the dashboard ships.**
  A dashboard shows one server's data to people authorized for *that* server. A marketing
  page shows nobody's. That distinction does not soften later.

## Design constraints the bot already imposes

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

## Sequencing

The public stats seam (`docs/claude/placeholders-and-domain.md`) is the smaller, safer version of the same problem and is a
reasonable first step: a read-only public aggregate endpoint, no auth, no per-server data.
Getting that right establishes the deployment shape before anything touches per-guild
configuration.
