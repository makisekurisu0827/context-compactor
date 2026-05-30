---
name: context-compactor
description: Keep long Claude Code sessions coherent by rolling old conversation into a single, constantly-updated summary so the context window stops overflowing and the assistant stops giving degraded or off-topic answers. Use this skill whenever a coding or development session has been running long, whenever the conversation has grown past roughly 20+ substantive exchanges, whenever the user notices Claude "starting to forget", repeating itself, contradicting earlier decisions, or answering questions that weren't asked, or whenever the user explicitly asks to "compact", "summarize", "trim", "hide old stuff", or "free up context". Inspired by SillyTavern's /hide message-management workflow, adapted for Claude Code's continuous-history context model.
---

# Context Compactor

## What this is for

Long sessions degrade. As a conversation grows, the early turns push the model toward its context limit, and the symptoms are familiar: the assistant starts forgetting decisions made earlier, repeating suggestions, contradicting itself, or drifting off the actual question. This is the Claude Code analogue of a problem SillyTavern users solve with `/hide` — pulling old messages out of what gets sent to the model.

Claude Code has no message "floors" and no per-message hide command, so the mechanism is different: instead of hiding individual turns, **maintain one rolling summary** that absorbs old content, and let the recent turns stay verbatim. Done right, context usage **converges to a stable band** instead of growing without bound.

This skill defines that workflow. It uses only ordinary abilities — reading, writing, and summarizing — so it changes nothing about Claude Code itself. It is a behavior convention, not a patch.

## The core principle: roll, don't accumulate

The trap to avoid is **stacking summaries**. If you summarize turns 1–20 into Summary A, then later make a separate Summary B for turns 21–40, then C, then D, you have just moved the overflow problem downstream — the pile of summaries grows forever.

Instead, keep exactly **one** summary and rewrite it in place:

**Wrong (accumulating):**
`Summary A` + `Summary B` + `Summary C` + recent turns → still grows unbounded.

**Right (rolling):**
At each compaction, feed `current summary + the new turns since last compaction` back through summarization, and produce a **single replacement summary**. Drop the old one. The summary's length stays roughly constant because each rewrite keeps only what still matters and lets stale detail fall away — the same way a person tracks "where the project is now" without memorizing every past day.

Result: `dropped old turns` + `one constant-size summary` + `last N verbatim turns`. Context converges.

Be honest about the limits: this is a buffer, not infinity. It extends a session substantially but does not make it eternal. For a genuinely fresh start, the right move is still a new session.

## When to compact

Trigger compaction when **any** of these hold:

- The conversation has run past roughly **20+ substantive exchanges** since the last compaction.
- The user reports degradation: forgetting, repetition, self-contradiction, or answering the wrong question.
- The user explicitly asks to compact / summarize / trim / free up context.
- You can tell context is getting tight and quality is slipping.

When you decide to compact mid-session, briefly tell the user you're doing it and why — don't do it silently, since they may want certain details preserved.

## What goes in the summary, what gets dropped

The value of the summary is in **selection**, not completeness. Keep what future turns will need; drop what's already resolved or superseded.

**Keep:**
- Current state of the work — what's done, what's in progress, what's next.
- Decisions that still bind future work, and the *reason* behind them (so they aren't relitigated).
- Constraints, requirements, and user preferences stated earlier.
- Open problems, unresolved bugs, and known gotchas.
- File/module names and their current role or status.

**Drop:**
- Intermediate back-and-forth that led to a decision already captured.
- Abandoned approaches (unless *why* they were abandoned is still useful — then keep one line).
- Verbatim code that already lives in the actual files on disk.
- Resolved questions and dead ends.
- Pleasantries and meta-chatter.

Rule of thumb: if the information is recoverable by reading the project files, it does not belong in the summary.

## Summary format

Use **two parts** — a state table for fast lookup, and a decisions narrative for the things a table would flatten.

```markdown
# Session Summary (updated: <turn count or timestamp>)

## State
| Item | Status | Notes |
|------|--------|-------|
| <module / task / file> | done / in progress / blocked / todo | <one line> |

## Decisions & context
<Short prose. Each entry: what was decided, and why. Causal chains and
trade-offs go here because a table loses the reasoning. Keep it tight —
a few sentences per decision, not a transcript.>

## Open items
- <unresolved bug, pending question, known gotcha>
```

The table is for structured, scannable facts (task status, file roles, todos) — quick to retrieve on re-read. The narrative is for anything with a *because* in it (why a design path was chosen or dropped), which a table would strip of meaning.

## Where the summary lives

Default to writing the summary to a file in the project — `CONTEXT_SUMMARY.md` is a good name — rather than only emitting it into the chat. A file persists, survives a new session, and can be re-read deliberately. Overwrite the same file on each compaction (that *is* the rolling behavior).

If the user prefers it stay in-conversation only, that's fine too — the rolling principle is identical; you just regenerate and replace the summary block rather than rewriting a file.

Before overwriting, if there's any risk of losing something the user wanted, confirm with them first.

## Workflow

1. **Detect** a trigger condition (turn count, degradation, or explicit request).
2. **Announce** briefly: tell the user you're compacting and why.
3. **Gather** the current summary (if one exists) plus the turns since the last compaction.
4. **Re-summarize** the two together into a *single* replacement summary, applying the keep/drop rules.
5. **Write** the replacement — overwriting `CONTEXT_SUMMARY.md` (or the in-chat summary block). Never append a second summary alongside the old one.
6. **Continue** the session working from `summary + recent verbatim turns`.
7. **Repeat** at the next trigger. The summary size should stay roughly flat across cycles — if it's creeping up every time, you're accumulating instead of rolling; tighten the keep/drop pass.

> **IMPORTANT:** Before generating a replacement summary, ALWAYS load and merge the existing `CONTEXT_SUMMARY.md` first. Do not generate a new summary from recent turns alone — doing so silently drops everything captured in earlier cycles, which defeats the whole point.

## Example

**Situation:** A refactoring session has run ~25 turns. The user says answers are getting sloppy and Claude keeps re-suggesting a logging approach already rejected.

**Action:** Announce a compaction. Produce one summary:

```markdown
# Session Summary (updated: turn 25)

## State
| Item | Status | Notes |
|------|--------|-------|
| auth module refactor | done | split into auth/ and session/ |
| logging | done | using structlog; stdlib logging was rejected |
| DB migration | in progress | step 2 of 4 |
| test coverage | todo | auth/ untested |

## Decisions & context
Chose structlog over stdlib logging because the team wanted structured
JSON output for the log aggregator; stdlib was rejected for that reason,
so don't revisit it. DB migration is being done in 4 incremental steps to
keep each PR reviewable rather than one big cutover.

## Open items
- auth/ has no tests yet
- migration step 3 may need a backfill script — undecided
```

Overwrite `CONTEXT_SUMMARY.md` with this, drop the 25 turns of back-and-forth, and carry on. The rejected-logging loop stops because the rejection (and its reason) is now pinned in the summary.
