---
type: context
topic: "QA Workflow Prompts"
version: "7"
guidelines_version: "7"
created: 2026-03-06
last_updated: 2026-04-21
---

Two-step QA workflow for verifying context files: a QA prompt (audit, fix, and report) and a re-check prompt (verify fixes, fresh re-read from a different angle). Both run in the same chat that produced the files. The re-check is repeated until diminishing returns.

# QA Workflow Prompts

Two prompts run in sequence in the same chat that produced the context files. The QA prompt audits the output, applies fixes directly, escalates uncertain items, and reports findings in a structured summary. The re-check prompt verifies fixes and runs a fresh audit from a different angle. Re-check is repeated as needed until the user sees diminishing returns in the run history.

**When to use:** After an AI session has produced context files that you want to verify before uploading to the project. Most useful for large files, new files, or any output that felt rushed or complex. Even for simple outputs, at least one QA pass plus one re-check is recommended.

---

## Design Rationale

### Why Completeness Leads

The QA prompt is structured with completeness as the primary task and structural checks as the secondary task. This reflects the risk asymmetry between the two:

- **Structural errors** (wrong frontmatter, missing changelog entries, naming issues) are low-stakes and easy to catch. The AI almost always spots them, and even if it doesn't, the fix is trivial and non-destructive.
- **Content omissions** (missed decisions, lost rationale, misrepresented constraints) are high-stakes and hard to detect. The AI has to re-read the full conversation and compare it against the files, which is cognitively expensive. A missed decision is invisible — no future session will know it was supposed to be there.

Completeness gets the AI's best attention up front, before fatigue or anchoring sets in. Structural checks follow as a mechanical checklist.

### Why Audit and Fix Are Combined

Earlier versions separated diagnosis (check prompt) and treatment (fix prompt) into two steps, requiring the user to review findings and paste a second prompt before any changes were made. In practice, the separation added friction without proportionate value:

- Most findings were clear-cut improvements that didn't require user judgment — phrasing accuracy, missing nuance, minor structural issues. Forcing the user to review and approve each one before the AI could act added a round-trip for no decision-making benefit.
- For a typical 3-round QA, the two-step approach required 6 prompt pastes (check-fix-check-fix-check-fix). The combined approach requires 3 (QA + re-check + re-check).
- The reference-file workflows (`reference-file-workflows_v2.md`) already use the combined pattern on editorially heavier material (longer files, more placement decisions) without quality loss.

The combined prompt preserves the user's decision authority where it matters: the AI escalates uncertain items under "Needs your input" and waits, while applying clear fixes directly. The structured summary gives the user full visibility into what was changed and why, with the ability to call out any fix they disagree with after the fact.

### Why Re-Check Is Explicit

A single QA pass is rarely sufficient. In practice, three rounds typically reach diminishing returns. Making the re-check an expected step rather than an awkward re-paste of the same prompt serves two purposes:

- It gives the AI a different cognitive frame. The re-check explicitly asks for fix verification (did the changes land cleanly?) and a fresh re-read (approach from a different angle). Re-pasting the same QA prompt primes the same mental model, which fights against the fresh-eyes framing.
- The run-history table in the QA Summary provides trend data across rounds. The user sees issue counts drop and severity shift from major to minor, and decides when to stop based on data rather than gut feel.

### Why No AI Verdicts

The AI consistently claims files are "clean and ready for upload" and then subsequent runs find new issues. The AI cannot reliably assess its own blind spots — if it could see the remaining issues, it would have already fixed them. All verdict and confidence language is replaced with factual reporting: what this run found, what needs user input, what was checked. The run history gives the user the trend data to make their own readiness judgment.

---

## Step 1 — QA

Paste this prompt in the same chat, after the AI has produced the files.

### Prompt

```
Before I upload these files, I need you to QA your own output. Switch roles — you are now a reviewer, not the author.

**Primary task — Completeness review**

Re-read our entire conversation from the beginning. As you read, identify everything worth persisting per the guidelines' persistence criteria (§11) — decisions, rationale, constraints, open questions, architectural context. Build a mental inventory of what should be in the files.

Then compare that inventory against the files you produced. For each file, check:
- Is anything from the conversation missing that should be there?
- Is anything misrepresented — present but inaccurate, watered down, or missing important nuance?
- Is anything placed in the wrong file given topic scope boundaries?
- Is anything included that shouldn't be — transient content, debugging noise, abandoned approaches, or meta-discussion that doesn't belong per the persistence criteria (§11)?

This is the hardest and most important part of QA. Structural errors are easy to fix later; missing content is invisible until a future session needs it and it's not there.

**Secondary task — Structural and quality checks**

After the completeness review, run through these checks for each file:

1. **Frontmatter correctness** — all required fields present, version matches filename, guidelines_version is current, dates are valid.
2. **Summary block** — present immediately after frontmatter, uses domain-specific terms, first sentence works as standalone scope label.
3. **Structural compliance** — one theme per file (§6), changelog present, concise, and in reverse chronological order (§8), cross-references use versioned filenames (§6).
4. **Content quality** — information is clear, well-organized, and optimized for retrieval by a future AI assistant. Flag anything vague, redundant, contradictory, or confusing to a reader with no prior context.
5. **Cross-file consistency** — if multiple files were produced, check for contradictions, duplicated content, or scope overlaps.

**Apply fixes and report**

For clear improvements — phrasing accuracy, missing nuance, structural issues, completeness gaps — apply targeted fixes directly to the existing artifacts. Do not rewrite from scratch. Do not reorganize, rephrase, or touch anything outside the scope of the fix.

For ambiguous cases — judgment calls about scope, placement, or interpretation where reasonable people could disagree — do not fix. Flag these under "Needs your input" and wait.

After making fixes, verify that only the intended changes were made and no other content was altered.

End your response with a clearly separated **QA Summary for this run:**
- **New issues found and fixed:** numbered list with what was wrong and what was done. If none, say "no new fixes applied this run"
- **Needs your input:** ambiguities or judgment calls that can't be resolved without the user. If none, say "none"
- **What was checked:** what was actually reviewed this pass
```

---

## Step 2 — Re-Check

Paste after reviewing the QA results. Repeat until diminishing returns — the run history table shows the trend.

### Prompt

```
Verify the fixes you applied, then run a full re-check from scratch. These are two separate tasks — do both.

**Fix verification:**
For each fix you applied in the previous round, confirm the change is correct and that nothing else in the file was altered.

**Full re-check:**
Re-read the conversation and the files with fresh eyes. Do not assume your previous QA caught everything. Approach the re-read from a different angle than your previous pass — look for things your earlier check was not positioned to catch.

Run through the same completeness and structural checks. If you find new issues, apply targeted fixes directly. For ambiguous items, flag under "Needs your input."

End your response with a clearly separated **QA Summary for this run:**
- **New issues found and fixed:** numbered list with what was wrong and what was done. If none, say "no new fixes applied this run"
- **Needs your input:** ambiguities or judgment calls that can't be resolved without the user. If none, say "none"
- **What was checked:** what was actually reviewed this pass
- **Run history:** table with columns for run number, issue count, and severity (major/minor). Reconstruct from prior QA turns in this chat.
```

---

## Changelog

### v7 — 2026-04-21
- Collapsed check and fix into a single QA prompt. AI audits, applies clear fixes directly, and escalates uncertain items.
- Replaced verdict system (Pass / Pass with notes / Fail) with the structured QA Summary format per `system-design-decisions_v8.md` Problem 43.
- Added explicit re-check prompt (Step 2) with fix verification, fresh re-read from a different angle, and run history from run 2 onward. Repeatable until diminishing returns.
- Restructured design rationale to cover new design decisions. Preserved completeness-leads structure.
- Added changelog conciseness to structural compliance check (§8).
- Upgraded to `guidelines_v7.md` conformance.

### v6 — 2026-04-20
- Renamed file from `prompt-qa-workflow_v5.md` to `qa-workflow_v6.md` (removed `prompt-` prefix).

### v5 — 2026-04-12
- Upgraded to `guidelines_v5.md` conformance. Updated all section references to match v5 numbering: §5→§6, §6→§8, §9→§11.
- Removed `summary` frontmatter field; added body summary block.
- Updated structural checks: replaced "summary is descriptive" with summary block check (present, domain-specific terms, standalone first sentence).

### v4 — 2026-03-22
- Added over-inclusion check to QA Check completeness review: reviewer now checks for content that should have been excluded per persistence criteria (§9).

### v3 — 2026-03-22
- Restructured QA Check prompt: completeness review is now the primary task, structural and quality checks are secondary. Added explicit instructions to re-read the conversation and build an inventory before comparing against files.
- Added design rationale section explaining the risk asymmetry between structural errors and content omissions.

### v2 — 2026-03-07
- Upgraded to `guidelines_v4.md` conformance. Updated filename to new naming convention. Updated `guidelines_version` from `"2"` to `"4"`.

### v1 — 2026-03-06
- File created (merged from `prompt-qa-check_v1.md` and `prompt-qa-fix_v1.md`)
