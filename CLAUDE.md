# Rooted — project brief for Claude Code

## Vision
Rooted is a self-guided Bible study app that anchors a Christian's weekly/daily scripture reading in their **career/vocation**. Goal: encourage a consistent rhythm of reading ("as the Spirit leads", not a box-ticking chore) and show how scripture speaks to the user's actual work.
Owner: Raphael Apata (footer on every page, exactly: "Created by Raphael Apata").

## Hard product requirement
**Rooted must be usable by everyone, with no Claude account and no Claude-specific limits.** The current prototype only runs its AI features inside Claude (see below). The real app must not. This is the first thing to design.

## Current state
A working prototype: one static file `index.html` (inline CSS + vanilla JS, no build step) plus:
- `kjv-compact.json` — full public-domain KJV, `{Book: [[verse,...], ...chapters]}` (~4MB, lazily fetched)
- `logo.png` — owner-supplied oak-tree logo
It is published as a Claude artifact for early testers. Run locally with `python3 -m http.server` (fetch() fails on file://). Sections re-render via innerHTML; each render function includes its own footer.

## Features in the prototype
- **Foundation**: name + choose one of 13 vocations (`VOCATIONS` array), each with curated Foundation Passages: Sales; Finance · Banking; Builder · Engineer; Technology (Prov 8:12); Creative; Marketing · Social Media (Mark 1:35-37, Song 1:3); Management · Leadership (adds Esther 4:14); Healthcare; Education; Sports · Athlete · Combat; Food & Service; Legal; Other · Still deciding (Prov 18:18, 3:5-6). Vocation ids are stable keys in saved data — don't rename ids.
- **Reading Log**: rolling 7-day cycle labelled Day 1–7 (no calendar dates); Day 1 = the day of first log. Free-text passage entry (multiple per day); clicking a passage shows KJV text via `parseReference` + `BOOK_ALIASES`. Foundation Passage chips open verse text and turn green once read; "✓ Read" logs them to today.
- **Growth tree**: SVG tree beside the log (sticky right column desktop, below log on mobile), label "Grow your tree with the Word", Jeremiah 17:8 beneath; 8 growth stages as days are logged (`renderGrowthTreeSVG`).
- **Theme / Reflect / Growth arc**: weekly theme, practical work challenge, journal, monthly growth narrative. Generated with Claude when available; otherwise deterministic rotating templates (`seededPick`, `THEME_FALLBACK_TEMPLATES`, `GROWTH_FALLBACK_TEMPLATES`, `CHALLENGE_FALLBACKS`).
- **Companion** (built in a separate Claude chat, merged into this file): a weekly check-in agent. Claude is given tools (get_profile, get_reading_log, get_journal, get_theme_history, lookup_passage, suggest_passage — see `companionTools`), decides what to read, looks up a passage in the bundled KJV, saves ONE suggested passage for the week ahead, and returns {title, message, noticed}. Notes are stored in `state.companion.notes`; runs automatically once a week (`companionDue`), or manually; shows its steps ("How your companion decided"). Journal is treated as private. Today it needs `window.claude.use("sample")` with tool support, so it only works inside Claude; otherwise the page says "Your companion needs Claude".
- **Resources**: books + mentors per vocation (`RESOURCES`). Original 10 vocations were web-verified. Technology/Marketing/Other reuse already-verified titles only and could use dedicated, researched lists. Never invent titles or links.
- **Community**: placeholder.
- **Data**: localStorage `rooted_state_v1`; JSON backup/restore, text export, "Clear my reading log" (two-click confirm). No accounts.
- **Responsive**: hamburger drawer under 760px; vocation grid 1/2/3 columns.

## Decisions and gotchas learned
- Use LOCAL date parts in `fmtDate` — `toISOString()` caused a BST off-by-one bug (wrong Day N).
- Cycle logic (`ensureCycle`): an open cycle with nothing logged slides forward to today, so Day 1 = first real log.
- Avoid `window.prompt/confirm` (broken in sandboxed iframes); use inline UI.
- Vocation switch must re-render the Reading Log (`rerenderLog`).
- Tone: warm, non-preachy, Spirit-led. Scripture and sources must be accurate.

## Decisions for the owner (raise these before building)
1. **AI for everyone (top priority).** In the standalone app, the AI features (theme, challenge, growth arc, Companion) should run through a backend that holds an Anthropic API key and calls Claude on behalf of all users — readers never need a Claude account. This means the owner pays per call. Decide a model for cost: free tier with a weekly allowance + paid tier/donations, church/organisation sponsorship, cheaper model for light features, generate-once-and-cache per week. Keep the template fallbacks for outages/limits. The Companion's tool loop must be re-implemented server-side (Anthropic API tool use) with the same tools reading the user's data from the database instead of localStorage.
2. **Accounts + backend + database** for cross-device continuity (a stated owner need). Pick the stack; consider a PWA first, app stores later.
3. **Privacy of journals.** Journal and log text will be sent to the Anthropic API: needs explicit consent, a privacy notice, and a check of Anthropic's current API data-handling terms. Minimise what is sent; let users opt out of the Companion.
4. **Bible translations.** Only KJV (public domain) is bundled. Readers using NLT, AMP, etc.: those are copyrighted (NLT via Tyndale, AMP via Lockman Foundation) and need licences. Suggested path: (a) preferred-translation setting that only changes "Read the full passage" links (Bible Gateway/YouVersion), (b) add public-domain WEB/ASV as a modern-English option, (c) later a licensed Bible API (API.Bible, ESV API) via the backend — check each translation's licence/display/caching terms. Owner has paused this decision; ask before building.
5. Onboarding/first-run copy, in-app feedback, reminders/notifications, real Community features, analytics, accessibility pass, tests, split the monolithic `index.html` into modules.

## Working agreement
Ask the owner before big architectural choices (stack, hosting, AI cost model). Propose a plan first, then build in small verified steps.
