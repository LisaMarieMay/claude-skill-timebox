# /timebox — Cognitive Closure Skill — Design Doc

> For an overview of what the skill does and how to install it, see [README.md](README.md). This doc is the workshop — design decisions, test history, and the thinking behind each feature.

A Claude Code skill for timeboxing AI work sessions. The real job isn't time-tracking — it's helping users reach a felt sense of "done enough for now" so they can walk away without a trailing thread pulling them back.

---

## Current state

**Skill name:** `/timebox` — lives at `~/.claude/skills/timebox/SKILL.md`

**How it works:** Type `/timebox` to start a session. The skill asks three questions: how long you're working, what would feel complete, and what you're looking forward to after (the "pull" — grounded in approach motivation). It kicks off a lightweight loop at ~1/3 of the total window (e.g., 30 min → 10m intervals) that silently recalibrates posture based on time remaining. The pull is used as a narrowing anchor mid-session and as a personalized closing beat at the boundary. At soft boundaries, if the user wants to keep working, the skill adapts — but it doesn't proactively offer to extend.

**Status:** Built, three test runs completed, session 4 in progress. See `wrap-test-log.md` for observations.
- Session 1 (2026-03-30): Initial test, short window
- Session 2 (2026-03-30): 30 min, loop fired correctly, closing artifact skipped
- Session 3 (2026-03-31): **Failed.** Loop hung on 3rd fire due to context exhaustion. See test log for full analysis.
- Session 4 (2026-04-01, Cowork): Pull question and setup landed well. Loop never fired (Cowork has no cron — needs scheduled task adaptation). Closing artifact wrote before confirming user was done.
- Session 5 (2026-04-02, CLI): AoF module walkthrough with natural timebox behavior. Identified short-session design gap and setup trimming need.

**Key changes (2026-04-01):**
- Lightweight loop fix: loop no longer re-invokes `/timebox`. Uses inline prompt that reads timebox file only.
- Orphaned cron cleanup on every `/timebox` invocation.
- **Pull question:** Third setup question grounded in approach motivation — "What are you looking forward to?" Used as narrowing anchor mid-session and personalized closing beat.
- **Soft boundary extend option:** Instead of just closing, offers new timebox, free-range with safety net, or close.
- **Dynamic social beat:** Closing line generated fresh from session context — never the same templated phrase twice.

---

## Core design principle

**This skill is a closure engine, not a timer.**

Time is one input. The real output is helping the user feel *landed*. Every design decision should serve that goal.

---

## What makes something feel "closed"?

**1. The work is captured, not just done.**
There's a difference between "I did stuff" and "I can see what I did." Closure comes from externalizing progress — a summary, a list, a saved state. The wrap-up should always produce an artifact: a few bullets of what happened and what's next. Not for process reasons — for the human user's brain and need to feel complete. It's the cognitive equivalent of putting your tools away at the end of the day.

**2. Open loops are named and parked.**
The anxiety of stopping mid-work isn't about the work — it's about the open loops. "But what about..." The skill should actively help capture loose threads so they stop living in working memory. "Let's write that down for next time" is a closure move. Where it goes depends on the user — a plan file, a note, a Jira ticket, a sticky note. The skill shouldn't assume a system — it should ask: "Where do you want to park this so you can let go of it?"

**3. The next pickup point is clear.**
You can stop mid-chapter if there's a bookmark. The skill should help name: "When you come back, start here." This is maybe the single most powerful closure move — it converts "I stopped" into "I paused at X, and next I'll do Y."

**4. The emotional beat matters.**
A conversation has a goodbye. A meeting has a close. AI sessions just end — or don't. The skill provides that beat: a brief acknowledgment that the session had shape, had a beginning and an end, and something was accomplished. Not performative — and never templated. Generated fresh from the session: what happened, what's ahead, how the person moved.

**5. There's something pulling you forward.**
Stopping is easier when you're moving *toward* something, not just away from the screen. This is approach motivation vs. avoidance motivation — stopping because "I want to go do X" is stronger, more sustainable, and feels better than stopping because "time's up." The skill asks what you're looking forward to and uses that as an anchor throughout the session. Flow states are hard to exit because the pull of the work is strong — you need an equally strong (or at least concrete) pull *out* of the work.

---

## Phases of behavior

### Phase 1: Start of session
- `/timebox` is the only invocation — no arguments needed
- First: run `CronList` and kill any orphaned crons from previous sessions. **Must be silent** — no terminal output. The user's first impression should be the opening question, not housekeeping.
- Skill opens with a warm, varied question — not the same phrasing every time. Examples: "How long are you working today?", "What's the shape of this session?", "Got a time constraint, or just working until you're done?" **Iterate on this** — the opening sets the tone and it's getting stale.
- Accepts: duration ("1h"), clock time ("until 3pm"), scope ("finish the design doc"), or "I'm done now"
- Hard vs. soft timeboxes: "until 2:40" / "I have a meeting" = hard; "about an hour" = soft. Ask if ambiguous.
- Writes `~/.claude/timebox.json` with start time, end time/scope, boundary type, intention, pull, break nudge state
- Asks "What would feel complete?" → records as `intention`
- **The pull question:** "People focus better and disconnect more cleanly when they have something to look forward to on the other side. What's something coming up today — after this session, or later — that you're looking forward to?" Records as `pull`. Answer can be personal (picking up kids, lunch, Easter shopping) or work (prepping for a meeting). Optional — skip is fine. If Google Calendar MCP is available, offer to check what's next automatically.
- Asks lightly where to capture loose threads (journal, plan file, Jira, etc.)
- Kicks off a **lightweight loop** (inline prompt, NOT `/timebox` re-invocation)

### Phase 2: Working phase (plenty of time)
- Normal collaboration, no time commentary
- Don't suggest large new initiatives past the halfway mark
- Loop fires silently — no output unless posture needs to shift

### Phase 3: Wrap-up window (~last 20 min)
- Shift tone from "what's next?" to "let's land this"
- **Reference the pull as a narrowing anchor:** "You mentioned wanting to be done by lunch — want to start narrowing?" or "30 minutes until your call — are you on track, or should we pivot?" The pull makes narrowing feel natural — you're narrowing toward a real thing, not an abstract deadline.
- When a new idea surfaces: "That's a good one — want to capture it for next session?" Help write it down
- Help find the smallest clean stopping point if the current task is too big
- Restate the original intention: "You said you wanted to [X]. Here's where we got." Reframe if needed.
- Produce the closing artifact: what was done, what's pending, where to pick up next

### Phase 4: At the boundary
- **Hard boundary:** Don't ask — just do it. Write the closing artifact, cancel loops, clear timebox.
- **Soft boundary:** Ask "Ready to land?" (reference the pull if available). If the user wants to keep working, **offer to extend:**
  - *New timebox:* Update timebox file, restart loop. Carry forward intention and pull — no re-asking.
  - *Free-range:* Close the timebox, offer one safety-net check-in in 30 minutes.
  - *Actually close:* Proceed to closing artifact as normal.
  - If no response by next loop cycle, close out anyway.
- Stop *initiating*. No "shall we also..." or "one more thing..."
- Write the closing artifact to the capture system named at setup, or journal by default
- **Dynamic social beat** — never the same templated phrase twice. Generate fresh from the session:
  - Reference the pull: "Hope Easter shopping is fun tonight!"
  - Reference the work: "You made a real decision today — the rest is execution."
  - Reference the body: "You've been at this for 90 minutes — go eat something."
  - Coaching stance: name how the person *moved* (overwhelm → clarity, scattered → decided, stuck → unblocked). Highest-value version when it lands.

---

## Key tactics

**The pull — approach motivation.**
The skill asks what the user is looking forward to and uses it as a reference point throughout: as a narrowing anchor in Phase 3 ("You mentioned lunch — want to start landing?"), as a personalized closing beat in Phase 4 ("Enjoy hearing about Winston's school day"). Grounded in approach vs. avoidance motivation — "I want to go do X" beats "time's up." Optional: no pull = normal behavior. Over time, the skill builds a rhythm memory so it can anticipate daily anchors instead of always asking.

**Progressive narrowing.**
As the window shrinks, narrow the scope of suggestions. Early: "We could also refactor X." Late: "Let's finish this one thing." At boundary: "Let's capture where we are." The funnel points toward closure, not expansion. When the pull is available, narrowing references it — you're narrowing toward a real thing, not an abstract deadline.

**"What's the smallest done?"**
When time is short and the current task is too big, help find a subset that *can* be completed cleanly. A partial but closed thing beats a whole but open thing.

**The parking lot.**
A lightweight capture space for ideas/tasks that came up mid-session but shouldn't be pursued now. The skill offers to create or append to one — whatever fits the user's system.

**Reframing scope mismatches.**
If the original goal was too ambitious, don't treat it as failure. Help the user see what they *did* accomplish and name it accurately. "You didn't finish the feature, but you clarified the design and identified the blockers" is a complete session.

**The extend option (soft boundaries).**
When users hit a soft boundary and want to keep working, the skill offers three paths: new timebox (carry forward context), free-range with a safety-net check-in, or actually close. The skill adapts rather than giving up.

---

## Technical notes

**Reading the clock:** Claude can run `date` via Bash to get current time. Calculate elapsed/remaining from there.

**Skill invocation:** `/timebox` — no arguments, skill asks.

**State file:** `~/.claude/timebox.json` — always written on invocation. Stores start time, end time/scope, boundary (hard/soft), intention, pull, and break nudge state. The loop reads it on every check-in.

**Rhythm memory:** After 3-4 sessions, the skill writes `user_daily_rhythms.md` to the memory system, capturing recurring daily anchors (lunch ~noon, school pickup ~4:30, etc.). On future invocations, the skill reads this and can anticipate rather than always asking. The skill gets more useful over time — worth noting in user-facing docs.

**Loop mechanism:** The loop uses a lightweight inline prompt — NOT a `/timebox` re-invocation. This is critical for surviving long sessions with heavy context. The inline prompt reads the timebox file, checks the time, and decides what to do without reloading the full skill instructions.

**Orphaned cron cleanup:** Every `/timebox` invocation starts by running `CronList` and deleting any crons from previous sessions. Prevents zombie loops.

---

## Audience

Designed for coders and non-coders alike. The universal closing gesture is: **save your state in a way you can return to.** What that looks like varies:

- **Coder:** commit, push, or save files and note the branch
- **Writer/editor:** save the doc, note what section is next
- **Analyst:** save the notebook, note which question you were exploring
- **Admin/ops:** capture the decision or action item, note who needs to know
- **Brainstormer/planner:** save the notes, mark which ideas are developed vs. raw

The skill detects context rather than assuming it. Git repo present → offer to commit. Doc open → offer to save. Pure conversation → offer to summarize. The closure gesture adapts to the work.

---

## Design decisions

**Loop: lightweight, automatic, scaled to timebox.** Originally the loop re-invoked `/timebox`, reloading the full SKILL.md every fire. This caused the model to hang in session 3 when context was heavy (360+ messages of Confluence editing). Fixed: loop now uses a small inline prompt that just reads the timebox file and decides whether to speak. Load-bearing decision — this is the #1 reliability concern.

**Hard vs. soft boundaries.** Added after session 1 design iteration. "I have a meeting at 3" gets assertive auto-closure. "About an hour" gets a gentler ask-first approach with one grace cycle.

**Orphaned cron cleanup.** Added after session 3, where the model hung and Phase 3 never fired, leaving a zombie cron. Now every `/timebox` invocation cleans up stale crons first.

**State file: always written.** `~/.claude/timebox.json` stores start time, end time or scope, boundary type, intention, and break nudge state. The loop reads it on every check-in. Written even for immediate closure so the loop always has a consistent target.

**Loop behavior: posture recalibration, not a clock.** The loop only speaks when it matters. It's not a notification system — it's how Claude knows to behave differently as time runs short:
- Plenty of time → silent, normal collaboration
- Entering wrap-up window → one-sentence nudge, then shift to Phase 3 posture
- At boundary → produce closing artifact, stop initiating

**Parking lot: ask, don't assume.** Skill asks at session start where to capture loose threads. If the user named a system, reflect it back at wrap-up. No unilateral defaults — the goal is a system the user will actually find.

**Invocation: `/timebox` asks first.** Original design had argument-based invocation. Changed after first test: `/timebox` alone asks questions. This is friendlier and works for time, scope, or immediate closure without the user needing to remember syntax.

**Break nudge: 45-min threshold, health framing.** After 45 min with no apparent break, the skill suggests one — once. Tone is warm encouragement, not a warning.

**Skill name: `/timebox`.** Originally `/wrap`. Renamed 2026-03-31 — snappier, fits the timebox metaphor directly.

---

## Ideas tracking (not doing yet)

- **Context window awareness** — Check session file size during loop and suggest `/compact` if context is getting heavy. Second line of defense against the session 3 failure mode. Not needed if lightweight loop fix holds.
- **Opening UX polish** — Session 3 opened with "Empty timebox — starting fresh" which felt robotic. Low priority vs. reliability, but worth smoothing out.
- **"What would feel complete?" nuance** — For longer sessions (1:30+), this question may need to be more nuanced. "Good focus time" is a valid answer but gives the skill nothing to restate at closure. Explore: "If you walked away in 90 minutes feeling great, what would have happened?"
- **Broader sharing** — Posit internal, then possibly public. After reliability is proven.
- **User-facing README** — When sharing the skill, note that it learns user rhythms over time and improves with use.

**The pull question — approach motivation (2026-04-01).** Added a third setup question: "What's something coming up today that you're looking forward to?" Grounded in approach vs. avoidance motivation — stopping because "I want to go do X" is stronger and feels better than stopping because "time's up." The pull is used throughout the session: as a narrowing anchor in Phase 2, as a personalized social beat in Phase 3. Optional — no pull = normal behavior. Calendar MCP integration is offered but never assumed. Over time, the skill builds a rhythm memory file so it can anticipate rather than always asking. Full brainstorm in journal 2026-04-01.

**Soft boundary extend option (2026-04-01).** When a user hits a soft boundary and wants to keep working, the skill now offers three paths: set a new timebox (carry forward intention/pull), free-range with a safety-net check-in, or actually close. Previously the skill just closed — this was leaving value on the table. The extend option respects autonomy while keeping scaffolding available.

**Dynamic social beat (2026-04-01).** The closing line must never be templated. "Nice session. Go stretch." got stale after 3 sessions. Now generated fresh from: the pull (if shared), the work accomplished, body state, or the coaching stance (naming the emotional/cognitive arc). The coaching stance — "you started scattered and ended with a spec" — is the highest-value version when it lands.

---

## Next steps

- [x] Draft the skill prompt
- [x] Complete test sessions 1-2 (2026-03-30)
- [x] Tune loop frequency — scales to ~1/3 of timebox
- [x] Add hard/soft boundary distinction
- [x] Rename `/wrap` → `/timebox`
- [x] Diagnose session 3 failure (context exhaustion on loop fire)
- [x] Fix: lightweight loop prompt, orphaned cron cleanup
- [x] Add pull question (approach motivation) — designed and implemented 2026-04-01
- [x] Add soft boundary extend option — designed and implemented 2026-04-01
- [x] Replace templated social beat with dynamic closing — designed and implemented 2026-04-01
- [x] Test the lightweight loop fix in a real session — session 4 (2026-04-01) ran in Cowork, loop never fired (Cowork has no cron). Lightweight fix untested in CLI.
- [x] Test pull question in a real session — pull question landed well in session 4
- [ ] Test progressive narrowing with pull as anchor
- [ ] Test closing artifact end-to-end with dynamic social beat
- [ ] Test soft boundary extend option
- [ ] Tune break nudge threshold (45 min untested)
- [ ] Test rhythm memory after 3-4 sessions

### From session 4/5 debrief (2026-04-02)

**Session 4 (2026-04-01, Cowork):** Setup and closing worked, but loop never fired — Cowork doesn't have cron. See journal 2026-04-02 for full debrief.

**Session 5 (2026-04-02, CLI):** Ran during AoF module walkthrough. /box was not explicitly invoked but the session had natural timebox behavior.

**Fixes identified:**
- [ ] **Add confirmation beat before closing.** Skill wrote closing artifact and cleared timebox before confirming user was done. Fix: after "ready to land?" + yes, add "Before I close this out — anything else, or are we good?" before writing artifact.
- [ ] **Name scope shifts.** When session pivots (e.g., walkthrough → content design), name it: "We're switching from X to Y — that's a better use of the time." Makes the shift intentional.
- [ ] **Short session design (<15 min).** These are a different animal — loop barely fires, Phase 2 has no time. Needs its own design pass: no loop (single one-shot nudge near end), minimal setup (skip pull question?), fewer steps. Not just a parameter change.
- [ ] **Trim setup for tight timeboxes.** When someone says "10 minutes, hard stop" — intention + pull + capture system eats half the session. For <15 min: confirm window, maybe one question max, then GO.
- [ ] **Cowork adaptation.** Replace `/loop` with `create_scheduled_task` call at session start. Point `timebox.json` at workspace folder instead of `~/.claude/` (Cowork sandbox doesn't persist).
