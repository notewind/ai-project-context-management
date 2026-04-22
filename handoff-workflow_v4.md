---
type: context
topic: "Handoff Workflow Prompt"
version: "4"
guidelines_version: "6"
created: 2026-04-04
last_updated: 2026-04-20
---

Two handoff prompts (time-sensitive and general) plus a cleanup prompt, for transitioning between sequential chats. The time-sensitive variant includes a timestamp lookup for workflows where the next chat needs the current time. Both use a two-pass approach: the first pass captures context, the cleanup pass trims over-inclusion.

# Handoff Workflow Prompt

## Problem

Two failure modes emerged when using Claude for extended work sessions:

1. **Single long chat:** The context window fills up over a full day of activity. Attention dilution degrades quality, and end-of-day context updates require injecting even more content (files to update) into an already bloated window.

2. **Many topic-based parallel chats:** Splitting by topic (as the parallel workflow suggests) produced too many chats — sometimes 9 in a day. Each needed a proposal, multiple QA rounds, and end-of-day reconciliation. The reconciliation cost in time, effort, and usage limits was disproportionate to the value.

## Solution

Use one chat at a time, sequentially. When finishing an activity or stretch of work, close the current chat and open a new one. Before closing, the AI produces a short **handoff brief** — not a proposal, not documentation, just enough context for the next chat to pick up without retreading ground.

This typically produces 3–4 chats per day rather than 9, significantly reducing reconciliation overhead.

## Relationship to Other Workflows

- **Not a replacement for the parallel workflow.** Each chat still produces a proposal at end-of-day via `parallel-workflow_v6.md`. The handoff brief is for inter-chat continuity; the proposal is for project documentation.
- **Not a replacement for serial updates.** If only one chat runs for an entire work session, the standard Case 3 prompt from `context-workflow_v6.md` is simpler.
- **This workflow changes how parallel chats are created** (sequentially through the day rather than by topic), but doesn't change how they're reconciled.

---

## Prompt 1A — Time-Sensitive Handoff

**When to use:** Workflows where the next chat needs to know the current time — e.g., how long until something closes, what's still feasible today.

```
Thank you for all the help this session! I want to wrap up this chat and create a new one to get a fresh context window for the next stretch of topics/activities. I'll paste the handoff at the start of my next chat.

Can you write me a short handoff brief covering:

* What we did — topics covered, questions answered, things resolved
* What's next — anything we planned or discussed for later that hasn't happened yet
* Watch-outs — session-specific constraints or preferences the next chat should respect
* Check the current local time before writing the handoff (use whatever tool is available — time tool, bash, etc.) and use the result for the timestamp.

The next AI chat will have access to the same project files, because of that there are hard rules:

* No repeating of anything that's already in the project files
* No lecturing future AI on how and when to use the project files
* No mentioning of project files, period.

Important: keep the handoff very short and aim for a few bullet points total across all sections. Prioritize what needs to be included.
```

## Prompt 1B — General Handoff

**When to use:** Research sessions, planning work, or any workflow where time-of-day doesn't matter to the next chat.

```
Thank you for all the help this session! I want to wrap up this chat and create a new one to get a fresh context window for the next stretch of topics/activities. I'll paste the handoff at the start of my next chat.

Can you write me a short handoff brief covering:

* What we did — topics covered, questions answered, things resolved
* What's next — anything we planned or discussed for later that hasn't happened yet
* Watch-outs — session-specific constraints or preferences the next chat should respect

The next AI chat will have access to the same project files, because of that there are hard rules:

* No repeating of anything that's already in the project files
* No lecturing future AI on how and when to use the project files
* No mentioning of project files, period.

Important: keep the handoff very short and aim for a few bullet points total across all sections. Prioritize what needs to be included.
```

---

## Prompt 2 — Handoff Cleanup

**When to use:** After the AI produces a handoff brief (from either 1A or 1B). Review the brief, then paste this prompt to trim it down. This is expected to be the normal flow — the first pass over-includes, the cleanup pass fixes it.

```
The next AI will have access to the same project documents, so please drop from the handoff any information it will already have. Also drop anything that was only relevant to this session and won't help the next one.
```

---

## Design Decisions

### Two-pass approach: capture then trim

The AI almost always over-includes on the first pass due to anchoring — it has just spent an entire conversation on these topics and everything feels important. Fighting this with tighter first-pass constraints didn't work reliably; the AI either still over-included or started making bad relevance judgments about what to cut. Accepting the over-inclusion and adding a cleanup pass proved more reliable in practice: the user scans the brief, pastes the cleanup prompt, and gets a clean result on the second pass. The cleanup prompt is short and stable enough to be worth formalizing for easy copy-paste.

### The brief is for continuity, not documentation

The handoff brief exists so the next chat doesn't re-suggest things already done, knows what's planned next, and respects constraints that came up. It is not the record of the session — proposals handle that. This distinction matters because it determines what gets included: only things the next chat would make a worse decision without.

### Under-inclusion is cheaper than over-inclusion

The Claude mobile UI makes it easier to add a line to a pasted brief than to edit or remove text. When the brief is consumed on a mobile device, this asymmetry means the cost of the AI missing something (user types a line) is much lower than the cost of over-including (user has to fight the UI to delete, or the next chat acts on stale/irrelevant context). The cleanup prompt exists as the pressure valve for over-inclusion.

### "A few bullet points" over a fixed range

Early versions used "3–5 bullet points" as a hard budget. This failed for short conversations (not enough to fill 3 bullets) and long ones (5 was too tight). With the two-pass approach, the first pass no longer needs to be perfect — the cleanup prompt handles over-inclusion — so a slightly looser budget is acceptable. "A few bullet points" lets the AI scale naturally to conversation length while still signaling that brevity is expected. A fixed range of "5–8 bullets" was considered as the loosened budget but rejected because short conversations may not have enough content to fill even 5 bullets; a vague quantifier adapts better.

### "No mentioning project files" as a hard rule

The AI persistently restated project-level knowledge in the brief — particularly methodologies it had been corrected on during the session. It treated "I made a mistake and was corrected" as session-specific context worth passing on, which is technically true but useless: the next chat has the project files and will follow the methodology regardless.

Softer instructions ("don't repeat project file content," "the next chat can read those directly") didn't work — the AI found ways to rephrase the same content without technically naming the files. The nuclear option — "no mentioning of project files, period" — finally killed the pattern. Minor leaks persist (e.g., "per established methodology" without naming the file) but are small enough to not be worth further prompt engineering.

### Two variants, not a parameterized prompt

The time-sensitive variant differs only in the timestamp instruction. A single prompt with "if applicable, check the time" would work logically but invites the AI to skip the time check when it judges it unnecessary — and it judges wrong often enough to matter in time-sensitive workflows. Separate prompts make the time check mandatory when it matters and absent when it doesn't.

### Timestamp via tool, not approximation

The handoff timestamp matters because the next chat may use it for time-sensitive decisions (e.g., how long until a venue closes or a deadline hits). Early versions asked the AI to use a specifically-named time-lookup tool which didn't always exist. The instruction was changed to be tool-agnostic: "use whatever tool is available — time tool, bash, etc." and requires using the tool result for the timestamp, which implicitly prevents approximation.

---

## Changelog

### v4 — 2026-04-20
- Generalized remaining instance-specific framing throughout summary block, prompts, design rationale, and prior changelog entries per `documentation-principles_v1.md`. No design decisions changed; only wording was pulled up to the constraint level.
- Further generalized user-attribution framing to constraint-level descriptions in the Problem section and design rationale.
- Upgraded to `guidelines_v6.md` conformance.

### v3 — 2026-04-20
- Renamed file from `prompt-handoff-workflow_v2.md` to `handoff-workflow_v3.md` (removed `prompt-` prefix).
- Updated cross-references: `prompt-parallel-workflow_v5.md` → `parallel-workflow_v6.md`, `prompt-context-workflow_v5.md` → `context-workflow_v6.md`. Version numbers bumped because the referenced files were renamed in the same synchronized update (§6, Problem 6 staleness test).

### v2 — 2026-04-12
- Split into two variants: time-sensitive (1A) and general (1B). The general variant drops timestamp lookup and time-of-day references.
- Added cleanup prompt (Prompt 2) formalizing the two-pass approach that proved more reliable than fighting for a perfect first pass.
- Changed bullet budget from "3–5 bullet points" to "a few bullet points" — the two-pass approach allows a looser first-pass budget since cleanup handles over-inclusion; a fixed range of "5–8" was considered but rejected because short conversations may not fill it.
- Upgraded to `guidelines_v5.md` conformance: removed `summary` frontmatter field, added body summary block, updated cross-references to v5 files.
- Generalized problem and solution framing to apply to any sequential-chat workflow.

### v1 — 2026-04-04
- File created
