---
type: context
topic: "Changelog Cleanup Prompt"
version: "1"
guidelines_version: "7"
created: 2026-04-21
last_updated: 2026-04-21
---

Transitional cleanup prompt for trimming verbose older changelog entries when files are touched for other reasons. This file can be deleted once existing project backlogs are clean — §8's conciseness guidance and the QA structural check sustain the standard going forward for new entries.

# Changelog Cleanup Prompt

## When to Use

When a file is being updated for other reasons and you want to trim its older changelog entries in the same session. Paste before QA. The AI reports proposed trims and waits for confirmation before making changes.

This is a cleanup-as-you-go approach, not a bulk cleanup tool. Files get cleaner as they're naturally touched.

## The Prompt

```
Before QA, review the changelog in each file you just produced. Check all entries except the one from this session against §8's conciseness guidance, using the documentation-principles test: is the specificity doing work?

**The bar:** Changelog entries let a reader understand what happened without digging into the file. Over-trimmed entries force them to go hunting. When in doubt, keep the entry.

**Flag when wording isn't earning its place:**
- Restatement of rules or content already in the body, in the same language
- "What wasn't changed" tails (e.g., "No design decisions changed; only wording was updated")
- Cross-reference-recoverable context — but only when the cross-reference obviously addresses the point. If it has a counterintuitive rule a reader might not think to check, a short cite preempts a legitimate question and should stay

**Don't trim — even if it looks descriptive:**
- Design intent recorded at the change point (e.g., "§7 opens with a caveat that X is undocumented" flags a deliberate choice, not a restatement of §7's content)
- Preemptive explanations that answer a likely reader question
- Rationale for a change, where the rationale isn't also in the body

**Process:**
1. Name recurring patterns you see across files — it makes the review consistent.
2. For each flagged entry, mark it **clear-cut** or **borderline**.
3. At the end, explicitly state what you considered but chose not to flag, and why.

Report findings and wait for confirmation before changing anything.
```

## Why This Exists

§8 added conciseness guidance in `guidelines_v7.md`, and the QA structural check enforces it for new entries. But older entries were written before this guidance existed, and they serve as examples the AI pattern-matches when writing new entries — undermining §8 until they're cleaned up.

This prompt was iterated through three rounds of testing against the project's actual changelogs. Key findings that shaped it:

- A terseness-first prompt over-trimmed, stripping design intent and useful context alongside genuine redundancy.
- The documentation-principles test ("is the specificity doing work?") proved to be the right evaluation framework.
- Three durable redundancy patterns were identified: "what wasn't changed" tails, rule restatements that duplicate the body, and cross-reference-recoverable context. These are useful vocabulary beyond this prompt.
- An explicit "don't trim" list was needed to protect design intent at the change point, preemptive explanations, and rationale not duplicated in the body.
- Clear-cut vs. borderline marking and a "what I didn't flag" section improved review quality.

See `system-design-decisions_v8.md` Problem 54 for the full decision record.

## When to Delete This File

When you're satisfied that the changelog backlogs across your projects are clean. The prompt has no value after that — §8 and QA prevent the problem from recurring.

## Changelog

### v1 — 2026-04-21
- File created.
