# Copy accuracy — the site describes a real bot

The statistics are invented. **Everything else must stay true to the bot.** A page listing a
command the bot does not answer to is worse than one listing nothing.

Two places go stale when the bot's command surface changes, and only these two:

1. the `.chip` lists in `#features`
2. `#commands` — the full reference: **7 groups, 16 subcommand rows** (down from 9/45 as of
   2026-08-11 — see below)

Both are plain HTML in `index.html`. No generator; sync by hand.

**`#commands` is deliberately no longer a full reference — as of 2026-08-11 it is a curated
subset plus dashboard pointers.** The project owner supplied an exact allow-list of 16 real
commands (`/about`, `/ban`, `/captcha lockdown`, `/captcha unlock`, `/kick`, `/modlogs`,
`/musaed`, `/prefix reset`, `/prefix set`, `/prefix show`, `/serverinfo`, `/tickets panel`,
`/tickets setup`, `/timeout`, `/warn`, `/welcome preview`) and every other subcommand
previously listed was removed — not because the underlying bot features stopped existing, but
because most per-feature configuration moved to the dashboard (`docs/claude/dashboard.md`) and the page no longer
tries to enumerate every slash subcommand that still technically exists. **This inverts the
historical rule below it in this section**: `#commands` used to be a complete, generated-by-
hand mirror of the bot's registered command tree (hence "the full reference"); it is now a
short list of commands still worth calling out directly, plus short notes pointing everything
else at the dashboard. If you're asked to "sync `#commands` with the bot" in the old sense —
listing every registered subcommand — confirm with the owner first; the 2026-08-11 change was
a deliberate scope-narrowing, not the site falling behind. The three "moved to dashboard"
groups (`الترحيب`, `التحقق بالكابتشا`, `تذاكر الدعم`) each end with an identical one-line
`.cmdgroup__note`: **"باقي الإعدادات من dashboard.musaed.dev."** — same wording every time, no
restated feature detail. Two groups (`الحماية التلقائية`, `بوابة عمر الحساب`) lost every
subcommand and were removed as standalone groups entirely; they now live in one line each
inside a new small section right after `#commands`, `#dashboard-features`
(`ميزات تُدار من الداشبورد`), which carries a single shared dashboard link for all of them
rather than one per item. **It holds three `<li>`s, not two, as of 2026-08-17** —
auto-responses was added alongside automod and agegate for the same reason (a real system
with no slash commands); the single shared link did not change. `اختصارات نصية` (shortcuts, `/اختصار`) was removed from `#commands`
outright with no dashboard note at all — it simply is not in the owner's 16-command allow-list
and was not one of the five sections named for the "rest is on the dashboard" treatment. Don't
assume that means the feature is gone from the bot; it means this page stopped listing it and
no reason was given. Verify against the bot's actual command tree (the advice further down
this section) before writing anything more specific than that.

**This has already gone wrong.** The page originally advertised prefix commands
(`!طرد`, `!حظر`, `!اسكات`, `!تحذير`), a per-server prefix feature, and a rate-limit card with
an invented `!تهدئة` command. The bot registered **slash commands only** at the time — none
of that existed. It is all gone.

**`/prefix` used to be deliberately excluded, and no longer is — this is a reversal, not
drift.** The original reasoning: the command group existed and wrote a prefix, but no
prefix-based text commands were registered, so setting one had no user-visible effect, and
listing it would have advertised a no-op. As of 2026-08-11 the project owner explicitly added
`/prefix set`, `/prefix show`, `/prefix reset` to `#commands`'s allow-list (its own group,
`بادئة الأوامر`, icon `i-terminal`, pill `إدارة السيرفر`) regardless of that old reasoning. If
you're asked to "clean up" and remove `/prefix` again citing the no-op argument, confirm with
the owner first — the same way the Administrator-permission reversal below asks you to.

**In-chat moderation triggers exist but are deliberately not enumerated.** The bot now
recognizes a few plain Arabic words typed directly in chat as shortcuts for ban/timeout/
untimeout/warn — a deliberate, owner-approved exception on the bot side to "slash commands
only," each trigger gated by the same permission its slash equivalent uses and silent for
anyone lacking it. The "أوامر واضحة" card in `#features` mentions this in one generic
sentence — no trigger words, no mention that they're per-deployment configurable — on
purpose: naming them here would be this site publishing exactly what the bot's own design
keeps unlisted. **Never add a `#commands` row for these** — they are not slash commands, and
`#commands`'s own lede claims everything listed there is one.

**Shortcuts used to be listed as their own `#commands` group, and no longer are.**
`/اختصار` (Administrator) is still a real registered slash command that lets a server admin
define their own trigger word mapped to ban/kick/timeout — a peer of agegate/captcha/tickets
in the bot's settings layer, not a sub-feature of moderation. It had its own group
(`اختصارات نصية`, icon `i-lightning`, pill `مدير السيرفر`) until 2026-08-11, when it was
removed from `#commands` along with everything else not on the owner's 16-command allow-list
(see above). **Its bento card in `#features` was not touched** — that section wasn't in scope
for the 2026-08-11 change — so the feature is still described there, just no longer given its
own command-reference row. `/اختصار` was also the one command name on the page written in
Arabic rather than Latin, styled with `.cmd__name--ar` (`text-align: start`, `--sans` instead
of `--mono`) instead of the `dir="ltr"` isolation every Latin command name gets. With that row
gone, **`.cmd__name--ar` in `styles.css` is now unused dead code** — every remaining
`.cmd__name` on the page is Latin and gets plain `dir="ltr"`. Left in place rather than
deleted, in case an Arabic-named command gets added back later; flag it for cleanup if one
never does.

**`#commands` grew from 7 groups to 9, then back down to 7 — do not conflate the two
episodes.** The 7→9 growth (historical, pre-2026-08-11): `/about` split off from `معلومات`
into its own single-command group (`دليل الأوامر`, icon `i-book`, pill `إدارة السيرفر`) when
it stopped sharing a permission with `/serverinfo`, and the shortcuts group above shipped
separately — two unrelated reasons, same net effect (7 → 8 → 9). The 9→7 drop is the
2026-08-11 rewrite described earlier in this section: automod and agegate lost their groups
entirely (their commands moved to `#dashboard-features` instead), shortcuts was removed with
no replacement, and a new `بادئة الأوامر` (`/prefix`) group was added — net 9 → 7, not a
coincidence that it isn't 9 → 6; the prefix reversal happened in the same pass.

**When the bot-side project guide gets pasted into a session, diff it against this site
before touching anything else.** Three times now the bot's own `CLAUDE.md`/guide has been
shared mid-conversation and each time the site had silently fallen behind: round one, a
whole feature (`/tickets`) plus a subcommand (`/captcha status`) were missing; round two, a
permission had changed underneath an already-published command (`/about`); round three, a
second whole feature (`/اختصار`/shortcuts) was missing *and* an already-listed group had
grown four subcommands the site never got (`/tickets add/remove/list/edit`, ticket-type
management) — two gaps at once, in the same group and in a brand-new one, which is exactly
why a partial glance at the diff isn't enough. None of these show up by reading this site's
own files — they only show up by comparing against the bot's current command surface. If
you're handed that guide, read §4, §7 (or wherever the
bot repo lists its commands/permissions) and cross-check every group and row here before
doing anything else asked of you.

The eight real systems: moderation, automod (banned words + spam + raid detection), welcome,
account-age gate, captcha, support tickets, text shortcuts (admin-defined trigger words for
ban/kick/timeout), auto-responses (admin-defined trigger word → canned reply). Plus Arabic
duration parsing, which is the most distinctive feature and is worth keeping prominent.

**It was seven until 2026-08-17.** Auto-responses (`الردود التلقائية`) shipped on the bot
side 2026-08-16 and the site had never mentioned it — found by diffing the bot/dashboard
product inventory against this repo, which is exactly the check the paragraph below this
section's history describes. It got a `#features` bento card (icon `i-chat-text`) and a
`#dashboard-features` row, and the site's seven-count moved to eight: `#features`'s lede
(`سبعة أنظمة` → `ثمانية أنظمة`). (`updates.html` also carried a matching `أنظمة` figure at
the time; it no longer exists — removed 2026-08-25, above.) It has no slash commands at all,
so it gets **no `#commands` row** — same shape as automod and agegate.

**A ninth system exists on the bot side and must stay off this site.** The honeypot trap
channel (`قناة الفخ`) is gated to a single pilot guild and is not released; it is not one of
the eight above. Do not add it to `#features`, `#dashboard-features`, or the counts, however
complete the bot-side documentation looks — publishing an unreleased feature is worse than
omitting a released one.

**Being a "real system" and having a `#commands` presence are no longer the same claim.**
Automod, account-age gate and auto-responses are all still real, all still count toward the
eight above, and none of them has a single row in `#commands` — they're dashboard-only from
this page's point of view (see `#dashboard-features`, above). Don't use "it's not in
`#commands`" as evidence a system stopped existing; check the dashboard-managed groups and
`#dashboard-features` before concluding that.

**`#trust` quotes no numbers** for retention windows or automod thresholds — those are
server-configurable defaults, not promises. Do not add figures there.
