# Copy accuracy — the site describes a real bot

The statistics are invented. **Everything else must stay true to the bot.** A page listing a
command the bot does not answer to is worse than one listing nothing.

Two places go stale when the bot's command surface changes, and only these two:

1. the `.chip` list in the durations call-out in `#features`
2. `#commands` — **16 command rows in one flat list**, filtered client-side by three
   categories (`mod` 5, `setup` 8, `info` 3)

Both are plain HTML in `index.html`. No generator; sync by hand.

**The 7 groups became 1 flat list on 2026-08-25**, in the design rebuild — the same 16 rows
from the 2026-08-11 allow-list, none added, none removed, but no longer boxed into
`.cmdgroup` articles with a heading and a permission pill each. Each row now carries its own
scope in a third column (`صلاحيات ديسكورد` / `إدارة السيرفر` / `متاح للكل`), which is strictly
more precise than the old per-group pill: `/about` sits in the `info` filter but needs
`إدارة السيرفر`, and the group layout could not express that. **Consequences for anything
that referenced the old structure:** there is no `.cmdgroup`, no `.cmdgroup__title`, no
`.cmdgroup__perm` and no `.cmdgroup__note`; the three identical
"باقي الإعدادات من dashboard.musaed.dev." lines collapsed into **one** `.cmds__note` under the
whole list. `#dashboard-features` survived unchanged (3 `<li>`s, one shared dashboard link)
and now sits inside the `#commands` panel.

**The mockup's command list was wrong in two ways and both were corrected on the way in.**
It listed 14 rows, folding `/prefix reset` into `/prefix set`'s description and omitting
`/prefix show` — all three are on the owner's allow-list and all three are now their own row.
And it gave `/about` the scope `متاح للكل`; the real permission is `إدارة السيرفر`, which is
exactly the drift this file's "round two" note below records. Don't reintroduce either from
the mockup file.

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
(see above). **Its card in `#features` was not touched** — that section wasn't in scope
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
section's history describes. It got a `#features` card (icon `i-chat-text`) and a
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

## `#why-musaed` — the comparison panel, new 2026-09-05

`#why-musaed` ("ليش مساعد؟") sits after `#faq` and is a third place claims can go stale, and
the first that makes claims about **other people's bots**. A switcher (`.filters` style,
`initVersus()`) toggles one `.vs` block at a time: MEE6, Dyno.

- **Every "مساعد" cell is bound by the same rule as the rest of the site** — true to the
  bot, no invented numbers. The panel deliberately states "8 أنظمة" and "كل الأنظمة مجانية"
  and the same Free/Pro framing as `#pricing`, nothing sharper.
- **Every competitor cell must be fair and defensible.** MEE6 and Dyno rows stick to
  well-known, stable facts (English-first, MEE6's levelling, Dyno's configurable automod,
  both paywall a Premium tier). A `vs__foot` line dates the whole thing ("سبتمبر 2026")
  and points readers to each bot's own site. Do not tighten a hedge into a claim without
  checking the competitor's current feature set — comparison pages that overreach get
  cross-referenced and discounted by AI answer engines, and it's just bad faith.
- **Fact-check pass 2026-09-05 (web-verified).** The support-tickets row originally said
  "غير مدمجة" for **both** competitors — wrong both ways. Dyno shipped a **free** Tickets
  module in May 2026 (panels, private channels, intake forms, transcripts; Premium only adds
  deeper customization) — now "نظام مدمج ومجاني (من 2026)؛ برو يوسّع التخصيص". MEE6's
  Ticketing plugin exists but its free tier is stripped bare (0 embeds on panel/intro, no
  transcripts of deleted tickets per MEE6's own free-vs-premium page) — now "الإضافة موجودة،
  بس مميزاتها مقفولة لبرو". Also: MEE6's language cell was "إنجليزي؛ ترجمة جزئية" — no Arabic
  locale could be confirmed (MEE6 documents EN/FR/ES/DE, LTR dashboard), changed to "إنجليزي
  (بدون عربي)" and the Dyno cell matched. Dyno/MEE6 duration input confirmed English-only
  (`1w2d3h4m` / `10m`); Dyno automod confirmed rule-based and free.
- **MEE6 paywall creep, confirmed by owner screenshots 2026-09-05.** The MEE6 dashboard shows
  **Levels**, **Moderator**, **Automatic moderation**, **Reaction Roles**, **Welcome Channel**
  and **Automations** all behind the "To use this feature, upgrade your plan" wall. Four MEE6
  cells were corrected as a result:
  - اللغة الأساسية: "إنجليزي؛ ترجمة جزئية" → "إنجليزي (بدون عربي)" (no Arabic locale found)
  - الإشراف والسجل: "نعم" → "ضمن باقة برو" (Moderator plugin gated)
  - الحماية التلقائية: "نعم" → "ضمن باقة برو" (Automatic moderation gated — screenshot)
  - مستويات وXP، رتب تفاعل، إشعارات: "نعم — من أبرز مميزاته" → "موجودة، لكن كلها صارت لبرو"
  - تذاكر الدعم: "غير مدمجة" → "الإضافة موجودة، بس مميزاتها مقفولة لبرو"

  Still not tightened (owner's 2026-09-05 decision pass explicitly left these alone):
  **التحقق عند الدخول** still says "تحقق أساسي" — but Reaction Roles (the free
  verification-gate path) is now Premium too and captcha is limited-rollout, so this may
  also belong behind "ضمن باقة برو". And **السعر** ("مجاني محدود…") arguably understates how
  thin MEE6's free tier is (MEE6 Premium is ~$12/server/mo). Current wording is defensible;
  leave both until the owner asks.
- **Dyno verdict paragraph fixed 2026-09-05.** It used to imply مساعد was distinctive for
  having built-in tickets ("معاها التذاكر … جاهزة من غير إضافات"), which contradicts the
  tickets row saying Dyno *also* ships a free built-in system. Now: "ونظام تذاكر مدمج ومجاني
  مثل Dyno بالضبط. الفرق مو في التذاكر، الفرق إنه بلهجتك…" — the differentiator is language
  and Arabic customization, not tickets. Same dialect and warm tone as the rest of the panel.
- **Full-panel voice/consistency review 2026-09-05.** After the fact-check and verdict fix,
  every cell was re-checked against both verdicts: no other leftover sentence contradicts a
  table, and the lede / both verdicts / `vs__foot` are one consistent warm Saudi-dialect
  register.
- **Numerals here are Western**, in step with the site-wide 2026-09-05 conversion — the
  duration cells read "«30 يوم»", not "«٣٠ يوم»".
- **A Saudi bot block was drafted then pulled 2026-09-05** — no reliable feature data for it,
  and a comparison column full of "راجع موقعه" helps nobody. If an Arabic rival is added
  back, it needs a real, checkable feature set first, same bar as the MEE6/Dyno rows.
- **The tone is deliberately warm**, not a dry spec sheet — the verdict paragraphs open by
  granting the other bot its strengths ("بوت كبير وشامل", "شاطر في الأوتومود") before drawing
  the line. The tables stay neutral and factual; the warmth lives in the prose around them.
- **No competitor names in `#commands`, `#features`, or the counts** — the comparison lives
  in its own panel and nowhere else.

## The FAQ — new on 2026-08-25, and the two claims that were checked

`#faq` (أسئلة شائعة) is a six-item `<details>` accordion added in the design rebuild. It is
the first section on this site that answers questions rather than describing features, so it
is also the easiest place to publish something untrue. Two of the six came from the mockup
carrying claims this repo could not support, and both were changed with the owner's
decision:

- **"أحتاج أعطيه صلاحية أدمن؟"** The mockup answered **«لا»**. That is false as written: the
  live invite requests `permissions=8` — Administrator — and does so deliberately
  (`docs/claude/placeholders-and-domain.md`). Publishing "no admin needed" beside an invite
  screen that pre-ticks Administrator would have been the page contradicting its own button.
  The shipped answer says the invite does ask for it, that the age gate genuinely needs it
  and the rest need Manage Server (per `docs/claude/dashboard.md`), and then draws the
  distinction that actually matters: **the bot's grant is not the user's grant** — every
  moderation command still checks the invoking member's permissions and role position first.
  That is the same claim `#trust`'s صلاحياتك هي القرار makes, and it stays true regardless of
  what the OAuth scope carries.
- **"مساعد مجاني؟"** The mockup promised a Pro plan with **«حدود أعلى وميزات إضافية»**. A Pro
  tier is real — `privacy.html` references باقة برو and a 21-day post-lapse window — but
  nothing in this repo establishes *what* Pro adds, and the marketing site had never made a
  pricing claim at all. Shipped as "core systems free for every server, an optional Pro tier
  above it", with no description of what Pro includes. **If you are asked to describe Pro,
  get the details from the owner rather than inferring them from the dashboard or the bot.**

The other four (Arabic-first, where settings live, settings surviving a re-add, support)
restate claims already made elsewhere on the page and were left as written. The
settings-survival answer deliberately quotes **no retention figure** — same rule as `#trust`
below: those are configurable defaults, not promises.

**The FAQ is duplicated into JSON-LD (2026-09-05).** `index.html`'s head carries a `FAQPage`
node inside the `@graph`, and each `Question` / `acceptedAnswer` is the visible `#faq` text
**word-for-word** (Google requires the schema and the visible copy to match; AI answer
engines lift the `Answer.text` directly). So every FAQ edit is now two edits: the `<details>`
block **and** the matching `acceptedAnswer.text`. The validator in
`docs/claude/testing-and-traps.md` should check the two stay in sync.
