# /timebox — A Cognitive Closure Skill for Claude Code

A Claude Code skill that helps you land your work sessions cleanly. Not just a timer — a closure engine that helps you walk away feeling *done enough*, without a trailing thread pulling you back.

## What it does

Type `/timebox` to start a session. The skill asks three questions:

1. **How long are you working?** Accepts a duration ("1h"), clock time ("until 3pm"), scope ("until the design doc is done"), or "I'm done now."
2. **What would feel complete?** Your intention for the session — what "done" looks like.
3. **What are you looking forward to after?** The "pull" — grounded in approach motivation research. Stopping because *"I want to go do X"* is more sustainable than stopping because *"time's up."*

Then it gets out of your way and lets you work.

Behind the scenes, a lightweight loop checks in periodically and shifts the skill's posture as time runs short — from open collaboration to gentle narrowing to clean closure. You'll barely notice it working until it matters.

### The phases

| Phase | What happens |
|---|---|
| **Working** | Normal collaboration. No time commentary. The skill is invisible. |
| **Wrap-up** (~20 min left) | Tone shifts from "what's next?" to "let's land this." New ideas get parked for later. The skill helps find the smallest clean stopping point. References the pull to make narrowing feel natural. |
| **At the boundary** | Hard stop? The skill closes out assertively — you said you have somewhere to be. Soft stop? It asks if you're ready to land, and adapts if you're not done yet. |

### Features

- **Break nudge** — After 45 minutes of continuous work, a gentle one-time suggestion to stretch. It works because it's *in the flow of your work*, not a separate notification you can dismiss.
- **The parking lot** — Ideas that surface mid-session get captured somewhere you'll find them (journal, plan file, Jira — you choose), so they stop living in working memory.
- **Closing artifact** — A brief summary of what happened, what's parked, and where to pick up next. The cognitive equivalent of putting your tools away.
- **Dynamic social beat** — The closing line is never templated. It's generated fresh from the session: what you accomplished, what's ahead, how you moved. The best closings name the arc — *"You started scattered and ended with a spec."*
- **Hard vs. soft boundaries** — "I have a meeting at 3" gets different treatment than "about an hour." Hard boundaries close assertively. Soft boundaries ask first and adapt.
- **Scope sensing** — When you start something new near the end of a session, the skill notices whether it's a quick one-pass task or something that needs iteration, and helps you decide whether to start it or park it.
- **Rhythm memory** — Over time, the skill learns your daily anchors (lunch, school pickup, standup) and can anticipate rather than always asking. It gets more useful the more you use it.

## Design philosophy

**This is a closure engine, not a timer.**

What makes something feel "closed"?

1. **The work is captured, not just done.** "I did stuff" vs. "I can see what I did." The wrap-up produces an artifact — not for process, but for your brain.
2. **Open loops are named and parked.** The anxiety of stopping isn't about the work — it's about the loose threads. Writing them down is a closure move.
3. **The next pickup point is clear.** You can stop mid-chapter if there's a bookmark.
4. **The emotional beat matters.** AI sessions just *end* — or don't. The skill provides the goodbye that makes a session feel shaped.
5. **There's something pulling you forward.** Flow states are hard to exit because the pull of the work is strong. You need an equally concrete pull *out* of the work.

## Install

Clone the repo and symlink the skill into your Claude Code skills directory:

```bash
git clone https://github.com/LisaMarieMay/claude-skill-timebox.git
ln -s "$(pwd)/claude-skill-timebox/timebox" ~/.claude/skills/timebox
```

Then type `/timebox` in any Claude Code session.

## Status and roadmap

Active development. The core works — setup, phases, break nudge, closing artifact, pull question. Tested across 5 sessions with real work (not just test runs).

**Working well:**
- Break nudge at 45 min (validated — warm, not naggy)
- Pull question as motivational anchor (validated — feels natural in setup, not like a third burden)
- Lightweight loop that survives heavy context (fixed after a session 3 failure)
- Natural narrowing in wrap-up phase (validated — feels like good collaboration, not time management)

**Being refined:**
- Opening question phrasing — needs more variety, getting stale after repeated use
- Short sessions (<15 min) — different animal, needs its own design pass with minimal setup
- "What would feel complete?" — needs nuance for longer sessions where "good focus time" doesn't give enough to restate at closure
- Body-state awareness — "have you eaten?" for sessions crossing mealtimes
- Context window awareness — suggest `/compact` when context is getting heavy (defense against long-session failures)
- Cowork adaptation (scheduled tasks instead of cron)

See [DESIGN.md](DESIGN.md) for the full design doc, session test logs, and ideas backlog.

## Contributing

This is a personal skill, but feedback and ideas are very welcome. If you try it and something feels off — or feels great — open an issue or reach out. The skill gets better from real usage patterns, not theory.

If you want to contribute code: fork, branch, PR. I'll review.
