---
type: context
topic: "Reference File Workflows"
version: "2"
guidelines_version: "6"
created: 2026-04-19
last_updated: 2026-04-20
---

Six graduated workflows for managing reference files in a Claude project, from lightweight in-session optimization to rare heavy splitting. Workflows are ordered from most common and simplest to rarest and most complex. Use the simplest workflow that fits the situation. For the design rationale and decision history behind this structure, see `system-design-decisions_v7.md`, Problems 36–48.

# Reference File Workflows

## Workflow Overview

All six workflows are in this file. They cover the full lifecycle of reference files: getting research into the project (Workflows 1–3), maintaining what's there (Workflows 4–5), and occasionally restructuring for better retrieval (Workflow 6).

| # | Workflow | When | Effort | Splitting | Cross-check |
|---|---|---|---|---|---|
| 1 | In-session optimization | Research consumed during a working session | Lowest | No | Optional (complex cases) |
| 2 | Standalone single-report | Raw report exists outside a session | Low | No | No |
| 3 | Multi-report merge | 2+ reports with significant overlap | Medium | No | Yes (inventory) |
| 4 | Integrating new reports | New report + existing project file(s) | Medium | No | If 2+ reports |
| 5 | Knowledge base housekeeping | Periodic review, no new reports | Variable | No | Optional (merges) |
| 6 | Heavy optimization | Long-lived, heavily-used reference file | High | Yes (only here) | Yes (plan + production) |

**Kickstarting a new project with multiple reports:** Run Workflow 2 for each report that covers a distinct topic. For reports with significant overlap on the same topic, use Workflow 3 instead.

---

## Shared Principles

These apply to every workflow that produces or updates reference files.

### Merge Principles

When combining content from multiple sources (applies to Workflows 3, 4, 5, and 6):

- **Deduplicate overlapping coverage.** When multiple sources say the same thing, write it once.
- **Never silently drop a unique finding.** If a claim, model, technique, or number appears in any source — even just one — it survives into the output.
- **Preserve competing numbers with attribution.** When sources give different values for the same measurable thing, keep both with context. Do not average, resolve, or pick one.
- **Surface contradictions explicitly.** When sources disagree on a qualitative claim, document both positions with their context. Escalate to the user when the right call isn't obvious.
- **No source ranking.** Do not bias toward any source based on age, recency, or assumed authority. A newer report is not automatically more trustworthy than an older one, and vice versa. Let the content speak.
- **Length reduction comes only from deduplication, never from compression.** The merged output may be longer than any individual input. That is expected and correct.

### Content Removal Rules

Content is removed from reference files only when it is genuinely stale or factually superseded — never because the file is getting long. File length is a structural problem flagged by Workflow 5 and solved by Workflow 6 (heavy optimization with splitting).

When the AI proposes removing content, it must state the specific reason (superseded by newer finding, confirmed incorrect, no longer relevant to project scope) and wait for user confirmation. When in doubt, keep the content.

### QA Output Format

All self-QA and re-check prompts in this file use the same output format. The AI's working notes (full comparison, line-by-line checking) can be as thorough as needed. But the response must end with a clearly separated summary:

**QA Summary for this run:**
- **New issues found and fixed:** numbered list with what was wrong and what was done. If none, say "none found this run"
- **Needs your input:** ambiguities or judgment calls that can't be resolved without the user. If none, say "none"
- **What was checked:** what was actually reviewed this pass
- **Run history** (include from run 2 onward): table with columns for run number, issue count, and severity (major/minor). Reconstruct run history from prior QA turns in this chat.

The AI must not state whether the file is ready for upload or make confidence claims about quality. Report what this run found, not an assessment of the file's state.

### Frontmatter Template

All workflows producing reference files use this frontmatter:

```yaml
---
type: reference
topic: "<short topic label>"
version: "1"
created: <today's date, YYYY-MM-DD>
last_updated: <today's date, YYYY-MM-DD>
shelf_life: "<plain-language time range reflecting how quickly the subject matter evolves>"
---
```

`guidelines_version` is omitted because optimization workflows typically run outside the project context. The AI has no access to the guidelines file and no way to verify the current version. The `created` and `last_updated` dates allow inferring the approximate era if needed.

### Summary Block

Immediately after frontmatter, write 2–4 sentences summarizing the most important findings and conclusions. The first sentence should work as a standalone scope label. Use terms a person would naturally use when asking about this topic. Include a sentence explaining the `shelf_life` estimate — what specifically would cause these findings to go stale. This block is the single highest-value text for RAG retrieval.

---

## Workflow 1 — In-Session Optimization

### When to Use

Research was consumed during a working session — a deep research run, a report pasted in, a paper discussed. The session AI already understands the content in context. At end of session, the Case 3 prompt (from `context-workflow_v6.md`) fires and the AI produces or updates reference files alongside context files as part of its normal duties.

This is the most common path. No standalone optimization prompts needed — the session AI does the work as part of its regular update duties.

### Steps

1. Research consumed during the working session
2. End-of-session Case 3 prompt fires as normal
3. AI produces or updates reference files alongside context files
4. Regular QA check (from `qa-workflow_v6.md`)
5. Supplementary reference-file self-QA (Prompt 1A below)
6. **If the session AI made significant additions beyond the source research, or reconciled the research against existing project knowledge:** post-production cross-check in a fresh chat (Prompt 1B below)

Simple case (straightforward transcription of one report into a reference file): stop at step 5. Complex case (substantial AI additions, reconciliation with project knowledge, or two or more reference files produced): continue to step 6.

### Prompt 1A — Reference-File Self-QA

Paste after the regular QA check, in the same session chat.

```
This session produced reference files from research consumed during the conversation. Verify the reference files you produced or updated this session.

For each reference file you produced or updated this session:

1. Compare it against the source research. Are all substantive findings, evidence, and conclusions preserved? Was anything dropped, compressed, or diluted?

2. Identify what you added beyond the source research — session context, empirical findings, connections to other project knowledge. For each addition, briefly note its source (e.g., "from session testing," "from project file X," "domain knowledge").

3. Check that contradictions between the new research and existing project knowledge are surfaced explicitly, not silently resolved.

4. Verify frontmatter is correct, summary block leads with a standalone scope label using domain-specific terms, and section headings are specific and query-matching.

If you find issues, apply targeted fixes. Do not rewrite from scratch.

End your response with a clearly separated **QA Summary for this run:**
- **New issues found and fixed:** numbered list with what was wrong and what was done. If none, say "none found this run"
- **Needs your input:** ambiguities or judgment calls you can't resolve alone. If none, say "none"
- **What was checked:** what you actually reviewed this pass
- **Run history** (include from run 2 onward): table with columns for run number, issue count, and severity (major/minor)

Do not state whether the file is ready for upload. Report what this run found.
```

### Prompt 1B — Post-Production Cross-Check

Open a fresh chat. Drop in the produced r- file(s) and the original source research doc(s). This cross-check differs from others (3B, 4B, 6B) in that the reviewer doesn't produce a fenced code block for copy-pasting back — the session chat may or may not still be open, so the user reads the flags and decides how to act on each.

```
I'm sharing one or more reference files that were produced during a working session, alongside the original source research they were based on. The AI that produced these files also had access to session context and project files that you don't have — so it may have added relevant information beyond what's in the source research.

For each reference file:

1. Compare it against the source research for completeness. Are all substantive findings preserved? Flag anything that appears to be missing.

2. Identify content in the reference file that doesn't appear in the source research. These are likely session additions. Flag any that seem questionable or unsupported by the source material — but understand that the producing AI may have had good reasons from session context you can't see.

3. Check that the summary block accurately represents the file's key findings and that section headings are specific and query-matching.

Present your findings as a list of flags. For each flag, state what you found and why you're flagging it. The producing AI or the user will make the final call on each flag.
```

### Design Rationale

- No standalone optimization prompt exists for this workflow. The Case 3 prompt already tells the AI to produce files per the guidelines. The supplementary QA (Prompt 1A) is the only addition — it focuses specifically on research content survival, which the regular QA doesn't check for.
- The cross-check (Prompt 1B) explicitly acknowledges that the reviewer lacks session context. This prevents the reviewer from confidently rejecting valid additions that came from empirical testing or project knowledge during the session. The reviewer flags; the user or session AI decides.

---

## Workflow 2 — Standalone Single-Report Optimization

### When to Use

A raw research report exists outside a working session — an export from another research tool, a downloaded paper, a report from a chat that's already closed. You just need it project-ready without doing a working session around it.

### Steps

1. Open the chat where the research was produced, or a separate chat with the report attached
2. Paste optimization prompt (Prompt 2A). AI produces the optimized file directly — no plan, no confirmation gate
3. Self-QA check (Prompt 2B)

Three steps. If the user spots something wrong in the output, they say so and the AI fixes it — that's normal conversation, not a workflow step.

### Prompt 2A — Optimization

Paste at the end of the chat that produced the research, or in a new chat with the report attached.

````
I need the research you just produced (or the attached research, if this is a new chat) optimized for RAG retrieval before I upload it to a Claude project.

**Frontmatter:**
Add YAML frontmatter with these exact fields:
```yaml
---
type: reference
topic: "<short topic label>"
version: "1"
created: <today's date, YYYY-MM-DD>
last_updated: <today's date, YYYY-MM-DD>
shelf_life: "<plain-language time range reflecting how quickly the subject matter evolves>"
---
```

**Summary block (immediately after frontmatter):**
Write 2–4 sentences summarizing the most important findings and conclusions. The first sentence should work as a standalone scope label. Use terms a person would naturally use when asking about this topic — not just the terminology the source material uses. Include a sentence explaining the `shelf_life` estimate — what would cause these findings to go stale and how quickly.

**Filename:**
Propose a filename using the pattern `r-{topic-slug}_v1.md`. Use domain-specific terms a person would naturally use when asking about this topic, choosing words that will stay accurate over time.

**Section headings:**
Replace generic headings with specific, descriptive ones that mirror how a person would ask about the content. Headings are chunk boundaries for RAG — clear, specific headings produce meaningful chunks.

**Content:**
Preserve all substantive content. Do not summarize, condense, or remove findings. Preserve any data tables, charts, or structured data. Ensure tables render correctly in markdown. Fix formatting issues. If the research contains a references/sources/citations section, preserve it at the end.

**Changelog:**
Add as the final section:
```
## Changelog
### v1 — <today's date>
- Optimized for project RAG retrieval from raw deep research output
```

Output the optimized file as an artifact titled with the proposed filename.
````

### Prompt 2B — Self-QA

Paste in the same chat after the optimized file is produced.

```
Before I upload this file, verify your optimization. Switch roles — you are now a reviewer, not the author.

Re-read the original research. Compare it against the optimized file:

1. **Completeness** — are all findings, evidence, and conclusions preserved? Was anything dropped, compressed, or diluted?
2. **Summary and frontmatter** — does the summary block accurately represent the key findings, not just the topic? Is the shelf_life estimate reasonable?
3. **Headings** — are all section headings specific and descriptive? Did any generic headings survive?
4. **Content integrity** — are data tables, structured data, and references preserved correctly?

If you find issues, apply targeted fixes to the existing artifact. Do not rewrite from scratch.

End your response with a clearly separated **QA Summary for this run:**
- **New issues found and fixed:** numbered list with what was wrong and what was done. If none, say "none found this run"
- **Needs your input:** ambiguities or judgment calls you can't resolve alone. If none, say "none"
- **What was checked:** what you actually reviewed this pass
- **Run history** (include from run 2 onward): table with columns for run number, issue count, and severity (major/minor)

Do not state whether the file is ready for upload. Report what this run found.
```

### Design Rationale

- No confirmation gate. The optimization is purely mechanical — no editorial decisions about splitting, merging, or content placement. The filename, shelf life, and summary are all visible in the output. If any are wrong, the self-QA or the user catches it.
- The gate's removal is a deliberate tradeoff against Problem 27 (shelf life reasoning should be surfaced before production). In Workflow 2, the shelf life is visible in the output and the self-QA checks it. A bad estimate triggers a targeted fix in the existing artifact — a cheap revision, not a full rewrite. The confirmation gate's other functions (splitting plan, contradiction handling) are absent in this purely mechanical workflow, so the gate's remaining value doesn't justify its cost.

---

## Workflow 3 — Multi-Report Merge

### When to Use

Two or more reports with significant overlap on the same topic. The overlap is why you're merging — if the reports cover different sub-topics with minimal overlap, they're separate Workflow 2 runs.

Common scenario: multiple deep researches run with the same or similar prompts. Seed variance produces different answers, different emphasis, different specific findings. The merge preserves all unique findings while deduplicating overlapping coverage.

### Steps

1. Open a separate chat. Attach all reports
2. Paste merge prompt (Prompt 3A). AI builds visible inventory, flags contradictions, proposes filename and shelf life
3. User takes inventory + source reports to a **cross-check chat** (Prompt 3B). Fresh AI reviews inventory for missing items and contradiction-handling quality
4. User returns to merge chat with cross-check feedback or approval
5. AI revises inventory if needed, then produces merged file
6. Self-QA check (Prompt 3C)
7. **Optional: post-production cross-review** (Prompt 3D) — recommended for 3+ source reports or particularly complex merges. Run in the same cross-check chat from step 3

### Prompt 3A — Merge

Paste in a separate chat with all reports attached.

````
I have multiple deep research reports on the same topic that need to be merged into a single reference file optimized for RAG retrieval. The reports were generated from the same or similar queries — they differ because research runs can produce varying results from the same query.

**Step 1 — Inventory (before writing anything):**

Build a complete visible inventory of every distinct item across all inputs:
- Every model, tool, or technique mentioned
- Every specific numerical claim (benchmarks, sizes, speeds, requirements)
- Every recommendation or conclusion
- Every unique analytical insight

For each item, note which report(s) it appears in. Categorize how you will handle each item:
- **Same claim across sources:** Deduplicate — write it once
- **Different specifics about the same subject:** Preserve all details with attribution
- **Disagreement between sources:** Surface both positions with context — do not resolve silently
- **Cannot determine handling:** Escalate to user for decision

Also propose:
- A filename using the pattern `r-{topic-slug}_v1.md` — use terms a person would naturally use when asking about this topic, choosing words that will stay accurate over time
- A `shelf_life` estimate with the specific factor that would make findings go stale

**Wait for my approval before proceeding to the merge.**

**Step 2 — Merge (after approval):**

Produce the merged file following these rules:

- Consolidate overlapping coverage into single entries. When multiple reports say the same thing, write it once.
- Never delete a claim, model, technique, or number that appears in any input — even if it appears in only one. Attribute single-source findings.
- When reports give different numbers for the same measurable thing, preserve both numbers and note the discrepancy. Do not average, resolve, or pick one.
- Do not shorten, compress, or omit any substantive content. The only acceptable source of length reduction is deduplication. The merged document may be longer than any individual input — that is expected.
- Structure the output for chunk-level retrieval: use specific, query-matching headings. Each section should be understandable without reading the rest of the document.
- Do not evaluate which source is "better." Preserve everything and let the content speak.

**Frontmatter:**
```yaml
---
type: reference
topic: "<short topic label>"
version: "1"
created: <today's date, YYYY-MM-DD>
last_updated: <today's date, YYYY-MM-DD>
shelf_life: "<approved estimate>"
---
```

**Summary block:** 2–4 sentences summarizing the most important findings. First sentence as standalone scope label. Include shelf_life reasoning.

**Changelog:**
```
## Changelog
### v1 — <today's date>
- Merged from N source reports. <Note on key merge decisions — contradictions preserved, items deduplicated, etc.>
```

Output the merged file as an artifact titled with the approved filename.
````

### Prompt 3B — Inventory Cross-Check

Open a separate chat. Paste this prompt. Then send the inventory from the merge chat along with all source reports.

```
I'm sending you an inventory that an AI produced as the first step of merging multiple research reports into one reference file. The inventory lists every distinct item across the reports and flags contradictions with proposed handling.

Read the source reports first. Build your own understanding of what they cover — every model, technique, number, recommendation, and insight. Then read the inventory.

Review the inventory against the reports:
1. **Missing items** — is anything in the reports absent from the inventory?
2. **Contradiction handling** — are the proposed handling decisions sound? Flag any where the AI silently favored one source, averaged numbers, or downplayed a disagreement.
3. **Misattribution** — are items attributed to the correct source reports?
4. **Filename and shelf life** — does the proposed filename use natural query terms? Is the shelf life estimate reasonable for the subject matter?

Format your entire response as a single fenced code block that I can copy-paste into the merge chat. Use this structure:

## Issues
Numbered, ordered by importance. Each must identify a specific problem and include a concrete suggested fix. Address the merge AI directly (use "you/your").

## What's Good
What holds up and should not change during revision. Keep it brief.

## Comments
Observations worth considering but not requiring a fix. If nothing, write "None."
```

### Prompt 3C — Self-QA

Paste in the merge chat after the merged file is produced.

```
Verify your merge. Switch roles — you are now a reviewer, not the author.

Go back to the approved inventory. For each item in it, confirm it survived into the merged file. Then re-read the source reports and check for anything the inventory itself missed.

Check specifically:
1. **Inventory coverage** — did every item from the inventory make it into the merged file?
2. **Single-source findings** — are findings that appeared in only one report preserved with attribution?
3. **Contradictions** — are they surfaced with both positions and context, not silently resolved?
4. **Deduplication quality** — when overlapping content was consolidated, was any unique detail from one source lost in the process?
5. **Summary and frontmatter** — accurate representation of key findings, reasonable shelf_life?
6. **Headings** — are all headings specific and query-matching?
7. **Consistency** — are claims, facts, and qualifications consistent within the file?

If you find issues, apply targeted fixes to the existing artifact. Do not rewrite from scratch.

End your response with a clearly separated **QA Summary for this run:**
- **New issues found and fixed:** numbered list with what was wrong and what was done. If none, say "none found this run"
- **Needs your input:** ambiguities or judgment calls you can't resolve alone. If none, say "none"
- **What was checked:** what you actually reviewed this pass
- **Run history** (include from run 2 onward): table with columns for run number, issue count, and severity (major/minor)

Do not state whether the file is ready for upload. Report what this run found.
```

### Prompt 3D — Post-Production Cross-Review (Optional)

Return to the same cross-check chat from step 3. Send the produced merged file.

```
Here is the merged file that was produced after your inventory review. Compare it against the source reports:

1. **Completeness** — did all substantive content from every source report survive? Pay special attention to single-source findings — items that appeared in only one report.
2. **Contradiction handling** — are contradictions surfaced with both positions and context, or were any silently resolved?
3. **Additions** — is there anything in the merged file that doesn't appear in any source report? Flag it.
4. **Deduplication** — where overlapping content was consolidated, was any unique detail from one version lost?

Present your findings as a list of flags, ordered by importance. For each flag, state what you found and why you're flagging it.
```

### Design Rationale

- **Inventory-first approach** prevents anchoring on the first report read. By listing every item before writing, the AI can't treat Report 1 as the base and subsequent reports as supplements.
- **The cross-check reviews the inventory, not the finished file.** Catching a missing item in a bullet list is cheaper than catching it missing from a finished document. The post-production cross-review (Prompt 3D) is optional because the inventory cross-check already validated completeness — the post-production review catches execution errors, not planning errors.
- **One file out, not many.** Splitting was removed from the merge workflow. The merge output may be long — that's acceptable. If it's too long for effective retrieval later, Workflow 6 handles splitting as a separate, deliberate step.
- **No "be concise" language anywhere.** Length reduction comes only from deduplication. The instinct to compress is exactly what drops the long-tail findings that running multiple research passes was supposed to capture.

---

## Workflow 4 — Integrating New Reports into Existing Project Knowledge

### When to Use

A new research report covers a topic already represented by one or more reference files in the project. You want to update the existing reference file(s) with the new findings rather than having separate files on the same topic.

### Steps

1. Open a chat inside the project. Paste or attach the raw report(s) — no pre-optimization needed
2. Paste integration prompt (Prompt 4A). AI reads the new report(s) and identifies the matching project file(s)
3. AI proposes: what to integrate, what's now stale (with reasoning for each proposed removal), contradictions to surface or escalate
4. **If two reports are being integrated:** open a cross-check chat inside the project (Prompt 4B). Share the plan, source reports, and current project file(s). Reviewer evaluates the plan. User returns with feedback or approval
5. User confirms (directly if one report, after cross-check if two)
6. AI produces updated file(s) (version bump, changelog entry)
7. Self-QA check (Prompt 4C)
8. **If two reports were integrated:** return to the cross-check chat. Share the produced file(s). Reviewer compares against source reports and previous version (Prompt 4D)

One report: steps 1–3, 5–7. Two reports: all steps.

**Practical ceiling:** Integrate at most two reports per session. Beyond that, context window pressure increases the risk of silent content loss. If you have three reports to integrate, run two sessions.

### Prompt 4A — Integration

Paste inside a project chat. Attach or paste the raw report(s).

```
I have new research that covers a topic already represented in the project's reference files. I need you to integrate the new findings into the existing reference file(s).

1. Read the attached new report(s) and identify the existing project reference file(s) on this topic. There may be one file or multiple sibling files if the topic was previously split.

   - If you find **one matching file**: proceed with this prompt.
   - If you find **multiple sibling files** on this topic: list them and describe how the new findings map across them. Some new content may belong in one sibling, some in another. For any new content that doesn't fit any sibling, flag it separately — I may want to run Workflow 2 for that portion instead.
   - If you find **no matching file**: this is a new topic — flag it and I'll use Workflow 2 instead.

2. Build a comparison:
   - **New findings to integrate:** information in the new report(s) that doesn't exist in the current reference file(s). List each item. If placing into sibling files, note which file each item belongs in.
   - **Stale or superseded content:** information in the current reference file(s) that the new research shows is outdated, incorrect, or no longer applicable. For each item, state specifically why it's stale and what supersedes it. Do not propose removing content simply because the file is long.
   - **Contradictions:** cases where the new report(s) and the existing file(s) disagree. For each, state both positions with context. Propose handling: surface both in the updated file, or escalate to me if you can't determine the right approach.
   - **Content to preserve unchanged:** confirm what in the existing file(s) remains valid and should be kept as-is, including any corrections, annotations, or adjustments applied in previous sessions.

3. Propose any changes to the summary block, headings, or shelf_life that the new findings warrant.

Wait for my confirmation before producing the updated file(s). I may adjust which removals to approve or how contradictions should be handled.

After approval: produce the updated file(s) as artifact(s). Bump the version number. Add a changelog entry describing what was integrated, what was removed (with reasoning), and which report(s) were the source.
```

### Prompt 4B — Cross-Check (Plan Review, 2+ Reports)

Open a fresh chat inside the project. Paste this prompt. Then send the integration plan from the main chat, the source reports, and the current project reference file(s).

```
I'm sharing an integration plan, source research reports, and the current project reference file(s) that would be updated. An AI in another chat proposed this plan for integrating the new reports into the existing reference file(s).

Read the existing reference file(s) and the source reports first. Build your own understanding of what's new, what's stale, and where contradictions exist. Then read the proposed plan.

Review the plan:
1. **Missing new findings** — is anything in the new reports absent from the plan's integration list?
2. **Unjustified removals** — does the plan propose removing content that isn't actually stale or superseded? Is the reasoning for each removal sound?
3. **Contradiction handling** — are contradictions surfaced fairly, or does the plan silently favor one source?
4. **Scope** — does the plan try to change content that the new reports don't actually address?

Format your entire response as a single fenced code block that I can copy-paste into the integration chat. Use this structure:

## Issues
Numbered, ordered by importance. Each with a specific problem and concrete suggested fix.

## What's Good
What holds up and should not change. Keep brief.

## Comments
Observations worth considering. If nothing, write "None."
```

### Prompt 4C — Self-QA

Paste in the integration chat after the updated file(s) are produced.

```
Verify your integration. Switch roles — you are now a reviewer, not the author.

Compare the updated file(s) against three sources: the new report(s), the previous version of the reference file(s), and your approved integration plan.

1. **New content integration** — did every approved new finding make it into the file? Is it placed in a logical location?
2. **Removals** — was only the approved stale content removed? Was anything else silently dropped?
3. **Contradictions** — are they surfaced with both positions and context, as approved?
4. **Preserved content** — is everything that should have been kept unchanged actually unchanged, including any corrections, annotations, or adjustments from previous sessions?
5. **Summary and frontmatter** — does the summary block reflect the updated content? Is shelf_life still appropriate? Is the version bumped?
6. **Changelog** — does the entry accurately describe what was integrated and removed?
7. **Consistency** — are claims, facts, and qualifications consistent within the file?

If you find issues, apply targeted fixes. Do not rewrite from scratch.

End your response with a clearly separated **QA Summary for this run:**
- **New issues found and fixed:** numbered list with what was wrong and what was done. If none, say "none found this run"
- **Needs your input:** ambiguities or judgment calls you can't resolve alone. If none, say "none"
- **What was checked:** what you actually reviewed this pass
- **Run history** (include from run 2 onward): table with columns for run number, issue count, and severity (major/minor)

Do not state whether the file is ready for upload. Report what this run found.
```

### Prompt 4D — Cross-Check (Post-Production, 2+ Reports)

Return to the same cross-check chat from step 4. Send the produced updated file(s).

```
Here is the updated reference file (or files) produced after your plan review. Compare against the source reports and the previous version of the reference file(s):

1. **New content** — did all new findings from the source reports survive into the updated file(s)?
2. **Removals** — was only stale/superseded content removed? Was anything valid silently dropped?
3. **Contradictions** — are they surfaced with both positions, or were any silently resolved?
4. **Preserved content** — is content from the previous version that should be unchanged actually unchanged?
5. **Consistency** — do claims and qualifications hold up internally?

Present your findings as a list of flags, ordered by importance.
```

### Design Rationale

- **No pre-optimization of the new report.** The integration AI doesn't need frontmatter and improved headings to understand the research content. The output already has proper structure because the existing project file has proper structure. Pre-optimizing the input is polishing something that's about to be disassembled.
- **The cross-check chat serves dual purpose** (plan review then post-production review) — same pattern as Workflow 6. One extra chat, same context, two passes.
- **Cross-check only for 2+ reports.** A single report integration has a constrained enough scope that the proposal step plus self-QA is sufficient. Two reports multiply the judgment calls: two sets of new findings, two contradiction sources against the existing file, and the reports may contradict each other.
- **Two-report ceiling per session.** Context window pressure with the existing project file, two reports, conversation history, and project RAG results increases the risk of exactly the kind of silent content loss these workflows exist to prevent.
- **Sibling file handling** addresses the post-Workflow-6 scenario where a topic has been split across multiple reference files. The AI identifies all matching files, maps new content to the right sibling, and flags ambiguous placements for the user.

---

## Workflow 5 — Knowledge Base Housekeeping

### When to Use

Periodically — when the project has accumulated reference files over time and you want to check what's still current. No new research involved. Especially useful after heavy optimization (Workflow 6) has broken reports into many smaller files, creating potential for overlap and fragmentation.

### Steps

1. Open a chat inside the project
2. Paste housekeeping prompt (Prompt 5A). AI reads the summary block of every `r-` file and reviews for staleness, overlap, irrelevance, and overgrowth
3. AI produces a recommendation list — not files
4. User reviews each recommendation (including breakage costs for deletions) and decides what to act on
5. For approved merges: AI executes in the same chat (Prompt 5B)
6. Self-QA on merged files (Prompt 5C)
7. **Optional: post-production cross-check in a fresh in-project chat** (Prompt 5D) for merged files
8. For oversized files flagged for splitting: take them into Workflow 6

### Prompt 5A — Housekeeping Review

Paste inside a project chat.

```
I'd like to do a housekeeping review of the project's reference files. Start by reading the summary block of every file in the project whose filename begins with `r-`. List every r- file you found before proceeding — I need to confirm you haven't missed any.

After I confirm the file list is complete, review each file's summary block and frontmatter and produce a recommendation list covering:

1. **Stale files** — reference files where `last_updated` is well past the `shelf_life` estimate, or where you can identify that the findings are likely outdated based on the subject matter. For each, state what specifically is likely stale and whether the file should be re-researched, updated, or deleted.

2. **Overlapping files** — reference files whose summary blocks suggest they cover substantially the same ground. For each pair, describe the overlap and recommend whether to merge them or keep them separate with clarified scope boundaries.

3. **Irrelevant files** — reference files that no longer seem relevant to the project's current direction, based on what context files reference and discuss. For each, explain why you think it's no longer relevant.

4. **Oversized files** — reference files that have grown large enough that they likely produce many decontextualized chunks during RAG retrieval. For each, note the approximate scope covered and whether splitting into sub-topics would improve retrieval.

5. **Cross-reference health** — context files that reference specific versions of reference files where the reference file has since been updated. List the stale cross-references.

6. **Deletion impact** — for each file you recommend deleting, check which context files cross-reference it. List the cross-references that would break so I can weigh the breakage cost against the deletion benefit.

For each recommendation, provide your reasoning. Present this as a list organized by the categories above. Do not produce any files — I will decide which recommendations to act on.
```

### Prompt 5B — Execute Approved Merges

Paste in the same chat after deciding which merges to approve.

```
I'd like you to execute the following approved merges: [list which files to merge and any specific instructions].

For each merge:
1. Read both files completely.
2. Decide which file to use as the base — choose the one that is more comprehensive or has broader scope. State your choice and reasoning. If the two files are genuinely equivalent in scope and coverage, ask me which to use before proceeding. Keep the base file's filename, bump its version, and note the merge in the changelog. The other file will be deleted after upload.
3. Merge following these principles:
   - Deduplicate overlapping coverage — write it once
   - Preserve every unique finding from both files, even if it appears in only one
   - When the files give different numbers or conclusions for the same thing, preserve both with context
   - Do not shorten or compress — length reduction comes only from deduplication
   - Structure for RAG retrieval: specific, query-matching headings
4. Update the summary block and shelf_life to reflect the merged content.

Output each merged file as a separate artifact.
```

### Prompt 5C — Self-QA (Merged Files)

Paste after merged files are produced.

```
Verify the merge(s) you just performed. Switch roles — reviewer, not author.

For each merged file, compare it against the two source files:

1. **Completeness** — did all substantive content from both source files survive? Pay attention to unique findings that appeared in only one source.
2. **Deduplication quality** — when overlapping content was consolidated, was any unique detail lost?
3. **Contradictions** — if the source files disagreed on anything, is it surfaced with both positions?
4. **Summary and frontmatter** — does the summary reflect the merged content? Is shelf_life reasonable? Is the version bumped correctly?
5. **Consistency** — are claims and qualifications consistent within the merged file?

If you find issues, apply targeted fixes. Do not rewrite from scratch.

End your response with a clearly separated **QA Summary for this run:**
- **New issues found and fixed:** numbered list with what was wrong and what was done. If none, say "none found this run"
- **Needs your input:** ambiguities or judgment calls you can't resolve alone. If none, say "none"
- **What was checked:** what you actually reviewed this pass
- **Run history** (include from run 2 onward): table with columns for run number, issue count, and severity (major/minor)

Do not state whether the file is ready for upload. Report what this run found.
```

### Prompt 5D — Post-Production Cross-Check (Optional)

Open a fresh chat inside the project. Send the merged file(s) and identify the source files that were merged.

```
I'm sharing one or more merged reference files. Each was produced by merging two existing project reference files. I need you to verify the merges.

For each merged file, find the original source files in the project (they should still be present — they haven't been deleted yet). Compare the merged file against both sources:

1. **Completeness** — did all substantive content from both sources survive?
2. **Unique findings** — are findings that appeared in only one source preserved?
3. **Contradictions** — where sources disagreed, are both positions documented?
4. **Deduplication** — where content was consolidated, was any unique detail lost?

Present your findings as a list of flags, ordered by importance.
```

### Design Rationale

- **Recommendations, not files.** The AI produces a triage list. The user decides what to act on. This prevents the AI from making irreversible decisions about what's stale or irrelevant.
- **File discovery uses `r-` prefix scanning.** The AI reads the summary block of every file starting with `r-` and lists what it found. The user confirms completeness before the review proceeds. This avoids relying on RAG to surface all files for a generic housekeeping query — a query pattern that matches poorly against any individual file's content.
- **Deletion impact assessed upfront.** Cross-reference breakage is part of each deletion recommendation, not a post-approval discovery. The user approves with full cost information.
- **Merges execute in-project.** The AI can see both files through project context, knows the project vocabulary, and can check cross-references. No reason to leave the project.
- **No merge base rule.** The AI reads both files and decides which is the better base based on comprehensiveness and scope, states its reasoning. No proxy rules (version number, recency) that would produce bad outcomes.
- **No QA on the recommendations themselves.** The user is the QA — they review each recommendation and decide. QA happens on the outputs of executed decisions (merged files).
- **The post-production cross-check (Prompt 5D) is optional** because housekeeping merges involve files that are already optimized project reference files. The merge is typically cleaner than merging raw research reports. Reserve the cross-check for cases where the merged files were large or the overlap was complex.

---

## Workflow 6 — Heavy Optimization

### When to Use

A reference file whose knowledge won't go stale soon **and** will be used extensively for an extended period. Both conditions must be true. This is a deliberate, resource-intensive choice — not a default escalation from other workflows.

**This is the only workflow that splits files into smaller topic-scoped pieces.**

### Steps

1. Take the reference file to a separate chat (outside the project). Attach it
2. Paste heavy optimization prompt (Prompt 6A). AI proposes splitting plan
3. **Open a cross-check chat.** Share the plan and source file. Reviewer evaluates the plan (Prompt 6B)
4. Return to optimization chat with feedback or approval
5. AI revises plan if needed, then produces split files
6. Self-QA check (Prompt 6C)
7. **Return to the cross-check chat.** Share produced files. Reviewer compares against source and approved plan (Prompt 6D)
8. If cross-review produced fixes: return to optimization chat, apply fixes, re-check (Prompt 6E) to verify fixes landed cleanly

### Prompt 6A — Heavy Optimization (Splitting Plan)

Paste in a separate chat with the reference file attached.

````
I need this reference file split into smaller, topic-scoped files optimized for RAG retrieval. Apply thorough analysis — careful evaluation of topic boundaries will produce better splits than a quick first pass.

**Assessment — before producing any files:**

Evaluate how the file should be split:
- Identify clearly separable sub-topics within the file
- Each resulting file must make sense on its own — complete frontmatter, summary block, and enough context to stand alone without sibling files
- Use related but distinct topic slugs (e.g., `r-kubernetes-networking_v1.md` and `r-kubernetes-storage_v1.md`, not `r-kubernetes-part-1_v1.md`)

Propose your plan:
- The filename for each resulting file, using the pattern `r-{topic-slug}_v1.md` — use terms a person would naturally use when asking about the topic, choosing words that will stay accurate over time
- How many files, what each covers, and the proposed topic boundaries
- How the files will cross-reference each other in their summary blocks — references should be symmetric (if file A references file B, file B should reference file A)
- Your proposed `shelf_life` for each file, assessed independently

**Do not continue past this point until I reply.**

**After approval, for each file:**

**Frontmatter:**
```yaml
---
type: reference
topic: "<short topic label>"
version: "1"
created: <today's date, YYYY-MM-DD>
last_updated: <today's date, YYYY-MM-DD>
shelf_life: "<approved estimate>"
---
```

**Summary block:** 2–4 sentences summarizing the most important findings. First sentence as standalone scope label. Include shelf_life reasoning. Cross-reference sibling files symmetrically — if file A references file B, file B should also reference file A.

**Section headings:** Specific, descriptive, mirroring how a person would ask about the content. Headings are chunk boundaries for RAG.

**Content:** Preserve all substantive content. Do not summarize, condense, or remove findings. Preserve data tables and structured data. Fix formatting issues. Preserve references/citations at the end of each file where applicable.

**Changelog:**
```
## Changelog
### v1 — <today's date>
- Split from <source filename> for RAG optimization
```

Output each file as a separate artifact titled with the approved filename.
````

### Prompt 6B — Plan Cross-Check

Open a separate chat. Paste this prompt. Then send the source file and the AI's proposed splitting plan.

````
I'm sending you a reference file and an AI-generated plan for splitting it into smaller, topic-scoped files for RAG retrieval. Read the reference file first — build your own understanding of where the natural topic boundaries are and what would make different parts go stale. Then read the proposed plan.

Review the plan against the source file. Look for anything that doesn't hold up — inconsistencies between the plan's reasoning and its conclusions, bad judgment calls on scope or structure, missing information. Check whether the plan accounts for all substantive content — flag any sections, tables, or findings that would be lost or orphaned under the proposed split.

Format your entire response as a single fenced code block that I can copy-paste into the optimization chat. Use this structure:

## Issues
Numbered, ordered by importance. Each must identify a specific problem and include a concrete suggested fix. Address the AI that wrote the plan directly (use "you/your").

## What's Good
What holds up and should not change during revision. Keep it brief.

## Comments
Observations worth considering but not requiring a fix. If nothing, write "None."
````

### Prompt 6C — Self-QA

Paste in the optimization chat after the split files are produced.

```
Verify your optimization. Switch roles — you are now a reviewer, not the author.

Re-read the source file from the beginning. Compare it against the set of files you produced:

1. **Completeness** — does the set of resulting files fully cover the original? Is anything lost in the gaps between them?
2. **Split quality** — does each file make sense on its own without requiring sibling files? Does each have its own complete summary block? Are topic boundaries clean?
3. **Placement** — is each piece of content in the file a user would retrieve when asking about it? Would a query about topic X land on the file that contains topic X's content?
4. **Summary and frontmatter** — accurate key findings, reasonable per-file shelf_life, symmetric cross-references to siblings?
5. **Headings** — are all headings specific and query-matching?
6. **Consistency** — are claims, facts, and qualifications consistent within each file and across sibling files?

If you find issues, apply targeted fixes to the existing artifacts. Do not rewrite from scratch.

End your response with a clearly separated **QA Summary for this run:**
- **New issues found and fixed:** numbered list with what was wrong and what was done. If none, say "none found this run"
- **Needs your input:** ambiguities or judgment calls you can't resolve alone. If none, say "none"
- **What was checked:** what you actually reviewed this pass
- **Run history** (include from run 2 onward): table with columns for run number, issue count, and severity (major/minor)

Do not state whether the file is ready for upload. Report what this run found.
```

### Prompt 6D — Post-Production Cross-Review

Return to the same cross-check chat from step 3. Send the produced files.

```
Here are the split files produced from the plan you reviewed earlier. Compare them against the source file and the approved plan:

1. **Completeness** — did all substantive content from the source file survive across the split files? Flag anything missing.
2. **Placement** — is each piece of content in the file a user would retrieve when asking about it? Flag content that seems stranded in the wrong file.
3. **Consistency** — do claims, facts, and qualifications agree across sibling files? When the same fact appears in multiple files, did caveats travel with it?
4. **Cross-references** — are sibling file references accurate and symmetric (if A references B, B references A)?
5. **Plan adherence** — does the execution match the approved plan? Flag any deviations.

Present your findings as a list of flags, ordered by importance. For each flag, state what you found and why you're flagging it.
```

### Prompt 6E — Re-Check

Paste in the optimization chat after fixes from the cross-review have been applied.

```
Verify the fixes you just applied, then run a full re-check from scratch. These are two separate tasks — do both.

**Fix verification:**
For each fix you applied, confirm the change is correct and that nothing else in the file was altered.

**Full re-check:**
Re-read the source file from the beginning. Compare it against the current state of the split files with fresh eyes — do not assume your previous QA caught everything. Run through the same checks: completeness, split quality, placement, consistency, cross-references.

If you find new issues, apply targeted fixes. List what you changed.

End your response with a clearly separated **QA Summary for this run:**
- **New issues found and fixed:** numbered list with what was wrong and what was done. If none, say "none found this run"
- **Needs your input:** ambiguities or judgment calls you can't resolve alone. If none, say "none"
- **What was checked:** what you actually reviewed this pass
- **Run history** (include from run 2 onward): table with columns for run number, issue count, and severity (major/minor)

Do not state whether the file is ready for upload. Report what this run found.
```

### Design Rationale

- **This workflow exists separately from Workflow 3** because splitting is a fundamentally different operation from merging. Merging combines sources into one file. Splitting decomposes one file into many. The editorial error profile is different — splitting creates placement errors (content in wrong file), while merging creates omission errors (content dropped during deduplication).
- **The cross-check chat serves dual purpose** — reviewing the plan first, then reviewing the produced files. Same reviewer, same source context, two passes. This avoids redundant context setup.
- **Symmetric cross-references are required** in both 6A (production) and 6D (review). If file A references file B, file B must reference file A — but not every file needs to reference every sibling. This ensures consistency without crowding summary blocks with exhaustive cross-reference lists in larger splits.
- **Re-check (Prompt 6E) only appears here.** Lighter workflows use self-QA with additional runs at user discretion. Heavy optimization justifies a structured re-check because the transcript analysis showed that post-production fixes can themselves introduce new problems when files cross-reference each other.

---

## After Uploading

Reference files sit in the project as backing material. When future sessions make decisions based on reference file content, those decisions are recorded in context files through the normal update workflow (Case 3 or parallel proposals). You do not need to manually distill reference files into other file types.

---

## Changelog

### v2 — 2026-04-20
- Generalized instance-specific tool reference in Workflow 2 "When to Use" per `documentation-principles_v1.md`. No workflow logic changed; only wording was pulled up to the constraint level.
- Upgraded to `guidelines_v6.md` conformance.
- Updated cross-reference to `system-design-decisions_v7.md`.

### v1 — 2026-04-19
- File created. Merges and replaces `prompt-research-optimization_v3.md` and the planned `prompt-knowledge-base-upkeep_v1.md`.
- Complete restructure around six graduated workflows, replacing the monolithic optimization approach.
- Splitting removed from all workflows except Workflow 6 (heavy optimization).
- Shared merge principles, content removal rules, and QA output format defined once.
