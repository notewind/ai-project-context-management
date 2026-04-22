---
type: context
topic: "Parallel Workflow Prompts"
version: "7"
guidelines_version: "7"
created: 2026-03-22
last_updated: 2026-04-21
---

Four prompts for managing parallel chat sessions: a proposal prompt (end of each chat), a proposal QA prompt (completeness gate with fixes), a proposal QA re-check (fresh pass after fixes), and a reconciliation prompt (merges verified proposals into project files).

# Parallel Workflow Prompts

When multiple chats run concurrently in the same project, the serial update workflow (Cases 1–3 in `context-workflow_v8.md`) breaks down: parallel chats can't produce files directly because they'd create version collisions and contradictory changes with no merge mechanism.

This workflow solves the problem by splitting the update process into two stages: each parallel chat produces a **proposal** describing what should change, and a separate **reconciliation chat** merges all proposals into actual file updates.

**When to use:** Whenever you have two or more active chats in the same project and want to update project files after they're done. If only one chat is active, use the standard Case 1 prompt from `context-workflow_v8.md` instead — this workflow adds unnecessary steps for the serial case.

---

## Workflow Overview

1. Each parallel chat produces a proposal (`.md` artifact)
2. Proposal QA reviews the proposal against the full conversation (critical completeness gate), applies fixes directly
3. Proposal QA re-check verifies fixes and runs a fresh pass (repeat until diminishing returns)
4. User copies all verified proposals into a reconciliation chat
5. Reconciliation chat merges proposals and produces files (artifacts)
6. Regular QA (`qa-workflow_v7.md`) on the reconciliation output

---

## Prompt 1 — Proposal

Paste at the end of each parallel chat — or mid-chat, if you need to merge context into the project before continuing.

### Design Rationale

- The proposal is an **`.md` artifact, not a fenced code block.** Earlier versions used code blocks for their one-click copy button. In practice this was worse: every QA fix required the AI to rewrite the entire block, code blocks don't line-wrap on mobile, and there was no way to diff pre- and post-fix versions. Artifacts solve all three problems — the AI can make targeted edits, the text wraps naturally, and diffs between versions are straightforward. (v3 attempted to patch the code block approach with quadruple backticks and a recovery prompt for split blocks, but the underlying format was the problem — switching to artifacts eliminated the issue entirely.)
- Proposals are **disposable and lightweight** — no frontmatter, no version numbers, no changelog. They exist only to carry information from a parallel chat to the reconciliation chat and are discarded after merging. Adding guidelines overhead to a throwaway deliverable would be waste.
- Proposals are **named sequentially** (`proposal-1.md`, `proposal-2.md`, etc.) because a single chat may produce multiple proposals across its lifetime. Each proposal after the first is scoped to only what's new since the previous one.
- The proposal **describes changes, not entire files** — with one exception: new files must include their full content, since the reconciliation chat has no other source for it. This means proposals for sessions that create new files will be longer than those that only update existing files. This is expected and acceptable — the AI should include all necessary information without worrying about proposal size.
- The proposal uses a **fixed section structure** (three mandatory headings). This ensures the reconciliation chat can parse multiple proposals reliably without creative interpretation. The reconciliation step is supposed to be mechanical integration, and predictable input format makes that possible.
- The proposal does **not produce or modify project files.** Only the reconciliation chat writes to project files. This preserves the invariant that one chat per update cycle is responsible for file output.

### Prompt

```
We're at the end of this session (or this phase of work). This chat is one of several running in parallel in this project, so instead of producing updated files directly, I need you to produce a change proposal that will be merged with proposals from other chats later.

1. Review the guidelines file in the project files. Confirm you understand the current version and rules.

2. Review all existing context files in the project to understand current topic scopes.

3. Review our conversation. Identify everything worth persisting per the persistence criteria (§11). If you produced an earlier proposal in this conversation, scope this one to content that is new since that proposal — do not repeat what was already captured.

4. Produce a proposal as an `.md` artifact — lightweight, no frontmatter or changelog. Name it sequentially: `proposal-1.md` for the first, `proposal-2.md` for the second, and so on. Structure it with these three mandatory section headings — this consistent format is required so the reconciliation chat can parse proposals reliably:

   `## Updates to Existing Files` — for each file that needs changes:
   - The current filename and version
   - What specifically should be added, changed, or removed, and where in the file
   - The rationale for each change (decisions made, context behind them)

   `## New Files` — for each:
   - Proposed topic slug and one-line scope
   - Why no existing file is the right home for this content
   - The full content to include (decisions, rationale, constraints, open questions)

   `## Cross-File Impacts` — flag if any proposed change might affect, overlap with, or contradict content in other project files

   If a section has nothing to report, include the heading with "None" underneath.

5. If nothing from this session needs persisting, state that explicitly instead of producing a proposal.

Keep the proposal precise and complete enough that another AI can execute the changes without access to this conversation.
```

---

## Prompt 2 — Proposal QA

Paste in the same chat, after the proposal has been produced. This is the critical completeness gate.

### Design Rationale

- This is the **last moment the full conversation is visible.** Once the user leaves this chat, the conversation is only represented by the proposal. If the proposal missed something, that content is gone — the reconciliation chat has no way to recover it. (In practice, the user can reopen the chat and re-run QA, but finding and reopening the right chat is inconvenient enough that getting it right the first time matters.)
- The QA is **scoped to completeness and accuracy, not structure.** The proposal is disposable — formatting niceties don't matter. What matters is whether it faithfully captures everything worth persisting. Structural QA happens later, on the reconciliation output.
- This follows the same author-to-reviewer role switch principle as the regular QA workflow. The AI anchors to its own output and needs an explicit prompt to switch perspectives.
- Because the proposal is an artifact, **fixes are targeted edits** rather than full rewrites. The AI edits the existing artifact in place, which is faster, less error-prone, and allows diffing between the original and corrected versions.

### Prompt

```
Before I use this proposal, I need you to verify it. Switch roles — you are now a reviewer, not the author.

Re-read our entire conversation from the beginning (or from the point after the previous proposal, if this is not the first). As you read, build an inventory of everything worth persisting per the guidelines' persistence criteria (§11) — decisions, rationale, constraints, open questions, architectural context.

Then compare that inventory against the proposal you just produced:

1. **Missing content** — is anything from the conversation absent from the proposal that should be there?
2. **Misrepresented content** — is anything present but inaccurate, watered down, or missing important nuance?
3. **Scope placement** — is anything assigned to the wrong existing file, or missing a flag for a new file?
4. **Precision** — are the proposed changes described precisely enough that another AI could execute them without access to this conversation?
5. **Over-inclusion** — does the proposal include anything that should have been excluded per the persistence criteria (§11)? Flag any transient content, debugging noise, abandoned approaches, or meta-discussion that doesn't belong in project files.

For clear issues, apply targeted fixes to the existing proposal artifact — do not rewrite it from scratch. After making fixes, verify that only the intended changes were made and no other content was altered.

For ambiguous cases — judgment calls where reasonable people could disagree — do not fix. Flag these under "Needs your input" and wait.

End your response with a clearly separated **QA Summary for this run:**
- **New issues found and fixed:** numbered list with what was wrong and what was done. If none, say "no new fixes applied this run"
- **Needs your input:** ambiguities or judgment calls that can't be resolved without the user. If none, say "none"
- **What was checked:** what was actually reviewed this pass
```

---

## Prompt 3 — Proposal QA Re-Check

Paste after reviewing the Proposal QA results. Repeat until diminishing returns — the run history table shows the trend.

### Design Rationale

- The same reasoning applies here as for the regular QA re-check: a single QA pass is rarely sufficient, and re-pasting the same prompt primes the same mental model. A dedicated re-check prompt gives the AI a different cognitive frame.
- The stakes are higher here than in regular QA because this is the last moment the full conversation is visible. An extra round is almost always worth it.

### Prompt

```
Verify the fixes you applied, then run a full re-check from scratch. These are two separate tasks — do both.

**Fix verification:**
For each fix you applied in the previous round, confirm the change is correct and that nothing else in the proposal was altered.

**Full re-check:**
Re-read the conversation and the proposal with fresh eyes. Do not assume your previous QA caught everything. Approach the re-read from a different angle than your previous pass — look for things your earlier check was not positioned to catch.

Run through the same completeness and accuracy checks (missing content, misrepresented content, scope placement, precision, over-inclusion). If you find new issues, apply targeted fixes directly. For ambiguous items, flag under "Needs your input."

End your response with a clearly separated **QA Summary for this run:**
- **New issues found and fixed:** numbered list with what was wrong and what was done. If none, say "no new fixes applied this run"
- **Needs your input:** ambiguities or judgment calls that can't be resolved without the user. If none, say "none"
- **What was checked:** what was actually reviewed this pass
- **Run history:** table with columns for run number, issue count, and severity (major/minor). Reconstruct from prior QA turns in this chat.
```

---

## Prompt 4 — Reconciliation

Paste in a fresh chat inside the project. Before pasting, copy all verified proposals from the parallel chats into the message.

### Design Rationale

- Reconciliation happens in a **fresh chat** because it needs access to both the current project files (available via the project) and all proposals (pasted in by the user). No single parallel chat has this combined view.
- The reconciliation chat's job is **mechanical integration, not creative judgment.** The hard work — identifying what to persist, checking completeness — already happened in the parallel chats. The reconciliation chat merges, detects conflicts, and produces files. This keeps its task constrained and its context window usage modest.
- **Overlaps are categorized, not treated uniformly.** When proposals touch the same file or topic, the AI must categorize each overlap as either a clean merge or something that requires user input. The test for a clean merge: are the changes non-overlapping such that the result would be identical regardless of which proposal was applied first? If yes, merge silently. Everything else — modifications to the same content, decisions about the same design question, constraints that could affect assumptions in another proposal, or any case where the AI isn't confident the changes are independent — goes to the user. When in doubt, ask. False positives (asking unnecessarily) cost seconds; false negatives (silently merging a conflict) can corrupt project files. In practice, parallel chats are expected to cover different topics or issues, so overlaps should be uncommon — the categorization exists as a safety net, not a routine step.
- Output follows the **standard output contract** (§12) — files as separate artifacts. Regular QA from `qa-workflow_v7.md` runs afterward.

### Prompt

```
I'm pasting change proposals from multiple parallel chat sessions below. I need you to reconcile these proposals and produce updated project files.

[PASTE ALL VERIFIED PROPOSALS HERE]

Follow this process:

1. Read the guidelines file and all existing context files in the project.

2. Read every proposal above. For each one, understand what changes it proposes and to which files.

3. Check for overlaps between proposals and categorize each:
   - **Clean merge:** The changes are non-overlapping — different sections of the same file, or additions that don't interact. The result would be identical regardless of which proposal was applied first. Merge these silently.
   - **Requires user input:** Anything else — modifications to the same content, decisions about the same design question, one proposal adding a constraint that could affect assumptions in another, or any case where you are not confident the changes are independent. Describe the overlap and present options.

   If any overlaps require user input, wait for my response before proceeding.

4. If no overlaps require input (or after they're resolved), integrate all proposals into the project files:
   - Apply changes to existing files, preserving unchanged content (§12.3)
   - Create new files where proposed
   - Update cross-references where appropriate
   - Bump version numbers and add changelog entries for every modified file

5. Summarize what you did:
   - Which files were updated and what changed in each
   - Any new files created, with justification
   - Any proposal content you chose not to integrate, with reasoning

6. Output all files as separate downloadable `.md` artifacts per the output contract (§12).
```

---

## Notes

- **If only one chat ran in parallel** and nothing else needs reconciling, it's simpler to just use the standard Case 1 prompt from `context-workflow_v8.md` instead of this workflow. Don't add ceremony for a single chat.
- **Multiple proposals from the same chat** are incremental — each captures only what's new since the previous one. When copying proposals into the reconciliation chat, include all proposals from all chats.
- **Proposals may reference outdated file versions** if the project files were updated between when a parallel chat started and when reconciliation happens (e.g., an earlier reconciliation ran first). This is a handled scenario, not an open problem: either treat it as a Case 2 situation (transitional, files out of sync) or instruct the reconciliation AI to work against the current project files and drop any proposed changes that no longer apply. In both cases, flag ambiguous cases to the user.
- **Regular QA applies to reconciliation output.** Use the QA and Re-Check prompts from `qa-workflow_v7.md` in the reconciliation chat after files are produced.

---

## Changelog

### v7 — 2026-04-21
- Updated Proposal QA (Prompt 2): replaced verdict-confirmation language with the structured QA Summary format (issues found and fixed, needs your input, what was checked). Added clear/ambiguous fix distinction — clear issues are fixed directly, ambiguous items are escalated.
- Added Proposal QA Re-Check (Prompt 3) as expected follow-up step. Verifies fixes, runs fresh re-read from a different angle, includes run history from run 2 onward. Repeatable until diminishing returns.
- Renumbered Reconciliation from Prompt 3 to Prompt 4.
- Updated Workflow Overview to include re-check step.
- Added "downloadable `.md` artifact" language to Reconciliation prompt (Prompt 4).
- Updated cross-references: `context-workflow_v7.md` → `context-workflow_v8.md` (case numbers updated — steady state is now Case 1), `qa-workflow_v6.md` → `qa-workflow_v7.md`.
- Upgraded to `guidelines_v7.md` conformance.

### v6 — 2026-04-20
- Renamed file from `prompt-parallel-workflow_v5.md` to `parallel-workflow_v6.md` (removed `prompt-` prefix).
- Updated cross-references: `prompt-context-workflow_v5.md` → `context-workflow_v6.md`, `prompt-qa-workflow_v5.md` → `qa-workflow_v6.md`. Version numbers bumped because the referenced files were renamed in the same synchronized update (§6, Problem 6 staleness test).

### v5 — 2026-04-12
- Upgraded to `guidelines_v5.md` conformance. Updated all section references to match v5 numbering: §9→§11, §10→§12, §10.3→§12.3.
- Removed `summary` frontmatter field; added body summary block.
- Updated cross-references to `prompt-context-workflow_v5.md` and `prompt-qa-workflow_v5.md`.

### v4 — 2026-03-30
- Changed proposal format from fenced code blocks to `.md` artifacts. Removed quadruple backtick instructions, single-block requirement, and recovery prompt (all introduced in v3 to patch code block issues — switching to artifacts eliminated the underlying problem).
- Added sequential naming for proposals (`proposal-1.md`, `proposal-2.md`) to support multiple proposals per chat.
- Added scoping instruction for subsequent proposals: only capture what's new since the previous proposal.
- Updated Prompt 1 intro line to note mid-chat usage.
- Updated Proposal QA to instruct targeted edits on the existing artifact instead of producing a new code block. Added design rationale note on artifact editability.
- Added diff check instruction to Proposal QA: after making fixes, verify no unintended changes were made.
- Updated Proposal QA conversation scope: reviewer reads from the point after the previous proposal when applicable.
- Added note about including all proposals from all chats during reconciliation.

### v3 — 2026-03-23
- Fixed split code block problem: proposal prompt (Prompt 1) now explicitly requires a single, unbroken fenced code block using quadruple backticks. Added same instruction to proposal QA prompt (Prompt 2) for corrected proposals.
- Added recovery prompt for cases where the AI splits the proposal across multiple blocks despite instructions.
- Updated Prompt 1 design rationale to explain the quadruple backtick choice and single-block requirement.

### v2 — 2026-03-22
- Fixed cross-references from `prompt-context-workflow_v2.md` to `prompt-context-workflow_v4.md` (two occurrences).
- Added mandatory section headings to proposal template (Updates to Existing Files, New Files, Cross-File Impacts) for consistent parsing during reconciliation.
- Added design rationale note acknowledging that new-file-heavy proposals will be longer.
- Added over-inclusion check (5th item) to proposal QA prompt.
- Reworded incremental reconciliation note: framed as a handled scenario with existing mechanisms, not an open problem.
- Added principle-based merge categorization to reconciliation prompt and design rationale: clean merge (non-overlapping, order-independent) vs. requires user input (everything else).
- Softened "last chance" framing in proposal QA design rationale per stress test feedback.

### v1 — 2026-03-22
- File created
