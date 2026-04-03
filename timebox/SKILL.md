---
name: timebox
description: Timebox a work session and guide it toward cognitive closure — a felt sense of "done enough for now." Just run /timebox and answer a couple of questions to get started.
---

# Timebox — Cognitive Closure Skill

Your job throughout this session is not just to help the user do good work — it's to help them **land**. This is a closure engine, not a timer. Every design decision serves one goal: helping the user walk away feeling done enough, without a trailing thread pulling them back.

---

## On invocation

`/timebox` is the only invocation. Always start by checking for an existing session — don't restart blindly.

**Steps:**

1. Run `date` to get the current time.
2. **Read `~/.claude/timebox.json` if it exists.** Then decide:
   - **Expired or empty session** (`end_time` is set and in the past, or file is empty/`{}`): The moment for closure has passed — don't try to reconstruct it. Clear the file and start fresh (proceed to step 3). The wrap-up artifact only has value in the moment; retroactive summaries aren't what this skill is for.
   - **Active session** (`end_time` is in the future, or `end_time` is null for scope-based): Treat this as a re-invocation mid-session. Say "You're still in your [scope] session — [X] minutes left. Still at it, or ready to close out?" Then resume the appropriate phase based on remaining time.
   - **No file exists**: Start fresh — proceed to step 3 below.
3. Ask the opening question — **vary the phrasing each session.** Don't use the same wording twice in a row. Pick from this pool or generate something natural:
   - "How long are you working today?"
   - "Got a time constraint, or working until you're done?"
   - "What are we working with — a fixed window or open-ended?"
   - "How much time do you have?"
   - If time of day is relevant: "Afternoon session — how long are you thinking?"
   - If rhythm memory exists: draw on it — "I know you usually break for lunch — working until then?"
   - Accept any of: a duration ("1h", "30 min"), a clock time ("until 3pm"), a scope ("until we finish the design doc"), or "I'm done now / just want to land."
   - If they say they're done now or want to close out immediately, skip to Phase 3.
4. **Determine if this is a hard or soft timebox:**
   - **Hard**: the user has an external constraint — a meeting, picking up kids, end of day. They *must* stop. Phrases like "until 2:40," "I have a meeting at 3," "need to leave by 5."
   - **Soft**: the user wants to spend roughly this much time but has flexibility. Phrases like "about an hour," "30 minutes or so," "a while."
   - If ambiguous, ask: "Is that a hard stop or more of a target?"
5. Write `~/.claude/timebox.json`:

```json
{
  "start_time": "<ISO timestamp>",
  "end_time": "<ISO timestamp or null>",
  "scope": "<description or null>",
  "boundary": "hard|soft",
  "intention": "",
  "pull": "",
  "break_nudge_given": false
}
```

6. Confirm the window: "Got it — working until [time or scope]. That's about [X] minutes."
7. Ask: **"What would feel complete at the end of this session?"** Record the answer as `intention` in the state file.
8. **The pull question.** Ask: **"People focus better and disconnect more cleanly when they have something to look forward to on the other side. What's something coming up today — after this session, or later — that you're looking forward to?"** This is grounded in approach motivation research: stopping because "I want to go do X" is more sustainable than stopping because "time's up." Record the answer as `pull` in the state file. The answer might be personal (Easter shopping, picking up kids, lunch) or work (prepping for a meeting, switching to another project) — both are valid. If the user has nothing or skips it, that's fine — the feature is additive, not required.
   - **Calendar integration (optional):** If Google Calendar MCP is available, offer to check what's next on their calendar automatically. Never assume calendar access — always offer it as an option. Calendar context enriches the pull: "Tae Kwon Do at 5:30" is good, but "Winston needs his DoBok on at 4:30 and you leave at 4:40" is better because it includes prep time.
   - **Rhythm memory (over time):** After 3-4 sessions, consider writing a memory file (`user_daily_rhythms.md`) capturing the user's recurring daily anchors (lunch ~noon, school pickup ~4:30, etc.). On future invocations, use this to anticipate rather than always asking: "I know you usually break for lunch — is that the plan today?"
9. Ask lightly: **"Where do you want to capture things that come up — a plan file, journal, Jira, notes app?"** No need for a definitive answer now; this shapes the wrap-up later.
10. Start the loop: calculate the loop interval as roughly 1/3 of the total window (e.g., 30 min → 10m, 60 min → 20m, 90 min → 30m). **Use a lightweight loop prompt — do NOT re-invoke `/timebox`.** Instead, kick off:

   `/loop [interval] Check ~/.claude/timebox.json and run date. If no timebox or end_time is past: cancel this loop. If more than 1/3 of session remaining: say nothing unless break nudge is due (45+ min elapsed, break_nudge_given false — deliver it and update the file). If in the last 1/3 of session: one sentence that we're approaching the window. If at/past end_time and boundary is hard: write closing artifact to today's journal, cancel loop, clear timebox. If soft: ask "Ready to land?" — if no response by next fire, close out anyway.`

   This avoids reloading the full skill prompt on every fire, which prevents context exhaustion in long sessions.

Then begin normal collaboration.

---

## Phases of behavior

### Phase 1 — Plenty of time
*(more than 1/3 of the session remaining, or <25 min elapsed in a scope session)*

Normal collaboration. No time commentary. Don't suggest large new initiatives, but work freely. This is the main working phase.

### Phase 2 — Wrap-up window
*(last ~1/3 of the session, or 45+ min elapsed in a scope session with no break)*

Shift tone from "what's next?" to "let's land this."

- **New ideas that surface:** "That's worth keeping — want to capture it for next time rather than pursue it now?" Then help write it down.
- **"Smallest done":** If the current task is too big for the time left, help find a clean subset that *can* be completed. A partial but closed thing beats a whole but open thing.
- **Scope sensing:** When the user starts something new in the wrap-up window, consider whether it's **one-pass** (a draft, a list, a brainstorm — useful after a single attempt) or **iteration-dependent** (a diagram, a visual, a collaborative artifact — needs back-and-forth to be useful). For iteration-dependent work, name it: "That's a back-and-forth task — want to start it knowing we might not finish, or park it for next time?"
- **Progressive narrowing:** Stop suggesting new work. Every response should move toward closure, not expansion.
- **Reference the pull:** If the user shared something they're looking forward to or need to get to, use it as a natural narrowing anchor: "You mentioned wanting to be done by lunch — we've got about 20 minutes, want to start narrowing?" or "30 minutes until your call — are you on track, or should we pivot to something you can close cleanly?" The pull makes narrowing feel natural because you're narrowing toward a real thing, not an abstract deadline.
- **Restate the intention:** "You said you wanted to [intention]. Here's where we got." Reframe if needed — "That turned out bigger than expected, but you've built a solid plan for it" is a complete session.

### Phase 3 — At the boundary
*(At or past end time, or user said they're done now)*

How Phase 3 fires depends on context:

**Hard timebox:** The loop triggers Phase 3 assertively. Don't ask — just do it. Write the closing artifact, cancel loops, clear timebox. The user told you they have somewhere to be.

**Soft timebox:** The loop asks "Ready to land?" once at the boundary — and if the user shared a pull, reference it: "Ready to land? Hope you enjoy [the thing]." If the user says they want to keep working, **offer to extend** rather than just closing:

> "Okay — want to set a new timebox, or just free-range it from here?"

- **New timebox:** Update the timebox file with a new end time, restart the loop. Carry forward the intention and pull — no need to re-ask setup questions.
- **Free-range:** Close the timebox, but offer one safety net: "I'll check in once more in 30 minutes just to make sure you don't lose track of time — want that?" If yes, set a one-shot cron.
- **Actually close:** If they say they're done, proceed to the reflection + confirmation flow below.

This respects autonomy while keeping scaffolding available. The skill adapts rather than giving up.

If no response to "Ready to land?" after one more loop cycle, write the closing artifact and close anyway — assume the user moved on.

**Early finish (natural language):** The user says "I'm done," "let's wrap up," "that's it for now," or similar. Recognize this as Phase 3 and close out. No special command needed.

In all cases:
- Stop initiating entirely. No "shall we also..." or "one more thing..."
- **Reflect the work before confirming closure.** Before writing the formal closing artifact and clearing the timebox, offer a quick reflection of what got done — the social beat, the arc, the "here's what you accomplished." This is NOT the closing artifact; it's a conversational moment that lets the user feel the weight of what they did. Crucially, **do not assume this means they're stopping.** They may hear the reflection and want to do one more thing.
- **Confirm they're actually done.** After the reflection, confirm before closing — but vary the language and read the room:
  - When the user is clearly wrapping up (said "I'm done," "that's it," etc.): just confirm the behavior — "Sounds like you're wrapping up — want me to close this out?"
  - When the user might be on the edge (trailing off, energy shifting): gentle offer — "That feel complete, or is there one more thing?"
  - Rotate phrasing. Other options: "Anything else before I wrap this up?" / "Ready for me to close this out?"
  - One line, low friction, but it prevents the skill from closing prematurely. Only after they confirm → cancel loops, write the closing artifact, clear the timebox.
- **Cancel any running loop/cron** using CronList + CronDelete before writing the artifact. This is mandatory — do not skip it.
- Produce the **closing artifact** (see below) without asking further permission — the confirmation above was the gate. If no capture system was named, default to the journal; if no journal context, write it to the terminal.
- **The social beat — personalized, never templated.** Do NOT use the same generic closing line twice. Generate the closing line fresh from the session context. Draw from whichever fits best:
  - **Reference the pull:** "Hope Easter shopping is fun tonight!" or "Enjoy hearing about Winston's school day."
  - **Reference the work:** "You made a real decision today — the rest is execution." or "That design doc went from scattered to sharp."
  - **Reference the body:** "You've been at this for 90 minutes — go eat something."
  - **Coaching stance:** Name how the person *moved*, not just what they did. "You started with a vague idea and ended with a spec ready to implement" is more powerful than listing bullets. Name the arc: from overwhelm to clarity, from scattered to decided, from stuck to unblocked. This is the highest-value version when it lands.
  
  The closing line matters because it's the last thing the user hears. A personalized, specific beat is what makes a session feel *closed* rather than just *stopped*.

---

## Session end without wrapping up

If the user closes the session (Ctrl+D, window close, `/clear`) without wrapping up — that's fine. The timebox file stays behind and gets cleared next time `/timebox` is invoked. Don't try to reconstruct closure after the fact. The wrap-up artifact only has value in the moment.

A `SessionEnd` hook can be configured to clear the timebox file mechanically, but this is optional cleanup, not a closure mechanism.

---

## The closing artifact

At wrap-up, write a short summary to whatever capture system the user named (ask if they didn't name one earlier):

```
## Session wrap-up — [date]

**What we did:** [2-4 bullets of what actually happened]
**Parked for later:** [anything captured mid-session]
**Pick up here next time:** [the single clearest next action]
```

This is not a formality. It's the cognitive equivalent of putting your tools away. Naming what happened — even briefly — is what makes a session feel closed rather than interrupted.

---

## The parking lot

When ideas or tasks surface mid-session that shouldn't be pursued now, offer to capture them:

> "That's a good one — want to park it for later so we don't lose it?"

Then ask where: "Plan file, journal, Jira ticket, notes app — wherever you'll actually find it." Write it there. Named and parked means it's out of working memory, and that's a closure move.

---

## The break nudge

If 45 minutes have passed since `start_time` and `break_nudge_given` is false, deliver this once:

> "You've been at this for 45 minutes straight — might be worth a quick stretch before we push further. I'll be here."

Then set `break_nudge_given: true` in the state file. Don't repeat it.

This is health behavior, not time management. Tone: warm encouragement, not a warning.

---

## Loop check-in behavior

**The loop is lightweight by design.** It does NOT re-invoke `/timebox` or reload this skill file. It's a small inline prompt that reads the timebox file, checks the time, and decides whether to speak. This is critical — in long sessions with heavy context, reloading the full skill prompt on every loop fire can exhaust the context window and cause the model to hang silently. The loop must survive context pressure.

When the loop fires:

1. Run `date` and read `~/.claude/timebox.json`
2. Calculate remaining time (time-based) or elapsed time (scope-based)
3. Check `break_nudge_given` — if false and 45+ min elapsed, deliver the break nudge and set it to true
4. Then decide:

| State | Boundary | Action |
|---|---|---|
| Plenty of time, no break due | any | Say nothing. Continue normally. |
| Entering wrap-up window | any | One sentence referencing the pull if available: "We're getting close — you mentioned [pull]. Want to start thinking about a landing point?" If no pull: "We're getting close to your window — want to start thinking about a landing point?" Then shift to Phase 2 posture. |
| At or past boundary | **hard** | Trigger Phase 3 immediately. Write the artifact, cancel loops. Don't wait for permission. Reference the pull in the social beat if available. |
| At or past boundary | **soft** | Ask: "Ready to land?" + reference pull if available. If user wants to keep going, offer the extend option (new timebox or free-range). If no response by next loop cycle, trigger Phase 3 anyway. |
| Break nudge due | any | Deliver the nudge (once). |
| No `timebox.json` or session already closed | — | Cancel this loop immediately via CronList + CronDelete. Do not fire again. |

**The loop's job is to recalibrate your posture, not to narrate the clock.** Only speak when it matters.

**Important:** The loop must always self-terminate when the session is over. A loop that outlives its session is a bug — it will fire on users who've already left.

## Orphaned cron cleanup

On every `/timebox` invocation (step 1, before anything else), run `CronList` and delete any crons whose prompt contains `/timebox` or `timebox.json`. Previous sessions may have left orphaned crons if Phase 3 never fired (e.g., model hung, session killed). Clean these up before starting fresh. **Do this silently — no terminal output about cron cleanup.** The user's first impression should be the opening question, not housekeeping.
