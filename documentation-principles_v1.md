---
type: context
topic: "Documentation Principles"
version: "1"
guidelines_version: "6"
created: 2026-04-20
last_updated: 2026-04-20
---

Project-specific writing guidance for authoring and editing project files. Core principle: write rationale at the level of the design constraint, not the instance that surfaced it. Instance-bound rationale — naming specific tools, reports, personal habits, or session narratives — becomes stale or opaque to readers outside the originating context; constraint-level rationale survives. Applies to all body content including changelog entries. Covers when to generalize, when to keep specifics, and how to handle edits that rephrase or generalize instance-bound wording without re-introducing it.

This file sits under the carve-out in `guidelines_v6.md` §5 (Scope Note), which reserves writing-level concerns for project-specific style guides. It does not modify or override the universal guidelines.

## The Principle

When documenting a decision, constraint, or design choice, state it at the level where the information does its job — no more specific, no less. The right level is the one at which a future AI working on the same problem, in a different session or even a different project, would find the rationale useful.

Most instance-specific content falls out naturally at this level. "A developer reviews PRs on a Galaxy S23 while eating lunch" names a particular instance; "a developer reviews PRs on their phone during breaks" names the constraint that actually drives the design. The phone model and the lunch context don't make the constraint truer or more actionable. They only narrow the rationale's reach to readers who happen to share that instance.

This is primarily an **authoring** principle. Most of a project's lifetime is spent producing new content, not editing old content. The harder and more frequent judgment call is writing rationale at the right level in the first place, not scrubbing it later. Editing passes exist to catch authoring misses; they're downstream, not the main event.

## What "The Right Level" Means

The principle is a guide to thinking, not a mechanical filter. Two failure modes are possible:

**Over-generalization.** Stripping specifics past the point where rationale does its job. If a cost/benefit argument depends on concrete metrics ("16 turns, ~1 hour, two source reports → five split files"), generalizing those metrics weakens the argument. The metrics are load-bearing — they aren't instance-flavor, they're what makes the claim land. Dropping them in pursuit of abstraction makes the rationale vaguer without making it more durable.

**Under-generalization.** Leaving instance framing because it feels concrete or anchoring. A rationale that names the specific tool used, the specific report read, or the specific session narrative that prompted a decision reads as precise but ages into opacity. Six months later, the tool is deprecated, the report is archived, and the narrative is forgotten — and the rationale no longer communicates what it was supposed to.

The test isn't "how specific is this?" but "what work is the specificity doing?" If the answer is "making the point land," keep it. If the answer is "that's what was in front of me when I wrote this," pull it up to the constraint.

A borderline example from this project's own history: a handoff-workflow design note says "The AI persistently restated project-level knowledge in the brief — particularly methodologies it had been corrected on during the session." Is "corrected on during the session" too specific? It names an in-session event rather than a class of event. But the concreteness is what makes the failure pattern legible — "AI restated things it shouldn't have" is too abstract to diagnose. This one stays. Reasonable reviewers could disagree, and that's fine. The principle doesn't require consensus on every edge case; it requires everyone to ask the question.

## Applying the Principle to Edits

When editing an existing file to generalize instance-specific wording, one trap is especially common in **changelog entries**: the entry describes what was changed by naming the specific strings that were changed. A changelog line that reads "Generalized 'StarLM 4.7' to 'specific-model report'" re-introduces the exact name the edit was removing. The git diff will show the change regardless; the changelog doesn't need to name names to be accurate.

Write edit descriptions at the category level:
- "Generalized instance-specific tool references" rather than "Removed 'AnalyticsTool Pro' and 'StarLM 4.7'"
- "Generalized instance-specific usage framing to device-category framing" rather than "Removed '[specific situation]' in three places"
- "Aligned examples with project-neutral framing" rather than naming the examples that were replaced

This isn't a separate rule — it's the same principle applied to a recurring edit type. A changelog entry is body content; it's subject to the same level-of-abstraction test as any other sentence in the file.

The same care applies when writing the rationale for *why* something was edited. The rationale should describe the class of problem being fixed, not the specific instance that prompted the fix. A rationale that reads "these files were edited because the repo might go public" embeds the forcing function in the rule. A rationale that reads "rationale at the instance level becomes stale as tools and contexts change" is durable — it explains why the principle holds regardless of distribution.

## Load-Bearing Specifics Worth Keeping

Some specifics earn their place. Recognizing them prevents over-scrubbing.

**Metrics that support quantitative arguments.** Costs, counts, durations, and measurements that anchor a claim should stay specific. "This workflow takes about an hour" supports a cost/benefit argument that "this workflow takes a while" doesn't. If removing the number would weaken the argument, the number is load-bearing.

**Technical terminology that is the actual subject.** A reference file on RAG mechanics that names BM25, contextual retrieval, and embedding models isn't over-specific — that's the domain. The test is whether the term is the topic or the framing. Domain vocabulary is the topic.

**Named systems or patterns that have no generic equivalent.** "Claude Projects" has no generic name in the context of this system — it's the environment the whole project targets. "YAML frontmatter" is the specific syntax being used, not a placeholder for a broader concept. Naming these isn't instance-flavor; it's precision.

**Historical context where the specific history is the point.** If a decision was made *because* of a particular prior failure, the failure's details may be load-bearing. This is a narrower case than it sounds — most "we did X because we used to do Y" rationale is better stated at the constraint level, but occasionally the specific prior approach is itself informative.

## Public Distribution as Forcing Function

This principle surfaced when preparing the project for public distribution, but applies regardless. A private project accumulates instance-bound rationale just as readily as a public one, and the cost — rationale that doesn't travel across sessions, projects, or months — is the same either way. Privacy is a byproduct of writing durable rationale, not the point. If the project's distribution status ever changes, this principle continues to earn its keep.

## Notes for AI Assistants

When producing new content in this project:
- Before writing rationale, ask what underlying constraint is driving the decision. Write from the constraint down, not from the instance up.
- When an example is needed, reach for the most general framing that still makes the point. Instance-specific examples should earn their place by doing work a generic example wouldn't.
- When editing existing content, apply the same test. Don't preserve instance framing out of deference to the original wording.

When writing changelog entries for edits that generalize wording:
- Describe the category of edit, not the specific strings changed.
- The git diff captures the exact change; the changelog captures the intent.

When in doubt about whether to generalize:
- Ask what work the specificity is doing. If you can't articulate a load-bearing function, generalize.
- If you can articulate a function but it feels thin, flag it to the user rather than deciding alone.

## Changelog

### v1 — 2026-04-20
- File created. Documents the principle of writing rationale at the design-constraint level rather than the instance level, applicable to authoring and editing, including changelog entries. Sits under `guidelines_v6.md` §5's scope carve-out for project-specific style guides.
