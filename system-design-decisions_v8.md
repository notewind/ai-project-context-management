---
type: context
topic: "System Design Decisions"
version: "8"
guidelines_version: "7"
created: 2026-03-06
last_updated: 2026-04-21
---

Record of every design decision made during the critical review of the context management system, including the v5 retrieval optimization overhaul, the adversarial review that refined it, the plan cross-check and optimization prompt v3 refinements, the v4 workflow restructure that replaced the monolithic optimization workflow with six graduated workflows based on real-world cost/benefit analysis, the documentation-principles decision that established project-specific writing guidance, and the workflow modernization decisions that collapsed QA, reordered context cases, aligned proposal QA, and added changelog conciseness guidance. Includes cross-check refinements from external review. A future AI should treat this as the authoritative record of what was considered, decided, and rejected.

# System Design Decisions

This file documents every design decision made during the critical review of the project context management system. A future AI working in this project should treat this as the authoritative record of what was considered, what was decided, and — critically — what was intentionally rejected or deferred so it doesn't re-propose it.

---

## Origin

The system was designed to preserve AI chat context across many future conversations using a Claude project. The mechanism is a set of structured `.md` files uploaded to the project. A `guidelines` file governs how all other `context` files are created, named, versioned, and updated.

A draft guidelines file was submitted for critical review. The review identified 13 problems. The user then made decisions on each. The resulting `guidelines_v2.md` established the baseline rules. The current authoritative rules file is `guidelines_v7.md`.

---

## Decisions by Problem

### Problem 1 — Frontmatter Delimiter: ACCEPTED
The draft used `***` as YAML frontmatter delimiters. Standard is `---`. Changed to `---` to ensure correct parsing by AI models and any tooling.

### Problem 2 — Guidelines File Exempting Itself Inconsistently: ACCEPTED
The guidelines file was missing a `topic` field and had no explicit statement about its own exemptions. Fixed by adding `topic` to the guidelines frontmatter and adding §14 (Self-Reference) to make the exemptions explicit: `guidelines_version` is omitted, and only one `type: guidelines` file may exist.

### Problem 3 — Fragile Version Mismatch Detection: ACCEPTED (all suggestions)
Three issues were identified and all fixed:
- If multiple guidelines files exist, the highest version is authoritative (AI must flag the conflict).
- If multiple versions of the same context file exist, highest version is current (AI must flag).
- Mismatch detection compares against the `version` field in the guidelines file's frontmatter, not the filename. The frontmatter is the contract; the filename is a convenience label.

### Problem 4 — No Handling for Duplicate Context Files: ACCEPTED
Same-topic duplicate detection added to §9 (Version Mismatch Handling). AI treats highest version as current, flags the conflict, recommends deleting the outdated version.

### Problem 5 — Major/Minor Versioning Distinction: SIMPLIFIED
The major/minor distinction added complexity with no functional consequence — nothing in the system behaved differently based on whether a bump was major or minor. Replaced with single-integer versioning (1, 2, 3, ...). Every update increments by one. This changed the filename format from `{slug}_V_{major}.{minor}.md` to `{slug}_V_{version}.md` (later simplified further to `{slug}_v{version}.md` in guidelines v4).

### Problem 6 — Cross-Reference Integrity: INTENTIONALLY NOT FIXED
Stale cross-references are a feature, not a bug. If a file references an older version of another file, the stale reference signals that the referencing file predates changes in the referenced file and may need review.

**Important guardrail added:** The AI must *not* update a cross-reference's version number unless the referencing file's content is also being actively reconciled with the referenced file's current state. Without this rule, the AI would "helpfully" update references and destroy the staleness signal.

### Problem 7 — Changelog Bloat: DEFERRED
Automatic pruning was proposed (keep last 5 entries, or collapse old entries into summaries). Not implemented — deferred until it becomes an actual problem. If changelogs grow too large, they will be trimmed through a dedicated housekeeping session.

### Problem 8 — No Token Budget Awareness: DEFERRED
Proposed adding max file length guidance, max file count, and a priority frontmatter field. Not implemented — context files are not expected to grow too large. Will revisit if it becomes a problem.

### Problem 9 — Case 2 (Transitional State) Has No Rules: ACCEPTED (with user-driven resolution)
Non-conforming files are handled by §10 (Non-Conforming Files): the AI reports what's wrong, presents options, and the user decides. Detection criteria: missing/malformed frontmatter, wrong naming convention, no `guidelines_version` reference, structural issues, or the AI cannot find expected files in project storage. The AI must never assume how to fix non-conforming files.

### Problem 10 — No Guidance on What Not to Persist: ACCEPTED
Added §11 (What to Persist). The key framing: context files exist for future AI assistants, not for the user. Persist decisions, rationale, constraints, open questions. Do not persist transient debugging, emotional tone, abandoned approaches (unless the reason for abandonment is informative). When uncertain, ask the user.

### Problem 11 — Accidental Content Mutation on Full-File Rewrites: ACCEPTED
Added §12.3 (Preserve unchanged content): "when outputting an updated file, take care to preserve all existing content that is not being intentionally changed. Do not reorganize, rephrase, or condense sections that are outside the scope of the current update."

### Problem 12 — No Mechanism for User to Correct AI Mistakes: DEFERRED
Proposed a `user_notes` frontmatter field or correction section. Not implemented — no evidence this is a problem.

### Problem 13 — Minor Issues: ALL FIXED
- **Date format:** Explicit instruction added to use actual session date in ISO 8601 format.
- **Topic slug rules:** Defined in §1 (File Naming) — lowercase, numbers, hyphens only; 2–4 words preferred; once established, don't rename without user instruction.
- **Frontmatter quoting:** Explicit note that version must be quoted (`"3"` not `3`) to prevent YAML type coercion.
- **Guidelines type uniqueness:** Explicit rule that only one `type: guidelines` file may exist.

---

## V5 Decisions — Retrieval Optimization and File Categories

These decisions were made after a deep research investigation into Claude Projects' RAG mechanics, prompted by retrieval difficulties at scale (25–45+ files, 70k–200k+ words in active projects).

### Research Findings

A deep research investigation was conducted covering official Anthropic documentation, community practices, and practitioner experience. Key findings:

- Claude Projects use a RAG system that retrieves text chunks (not whole files) based on query similarity when project knowledge exceeds context window thresholds.
- Filenames and in-document text are retrieval signals. Anthropic explicitly recommends descriptive filenames.
- There is no evidence of multi-pass retrieval within a single turn (read index → then read referenced file). The RAG system performs a single retrieval step per turn.
- Retrieval degrades at scale as more chunks compete for limited retrieval slots.
- Anthropic's Contextual Retrieval blog describes a hybrid semantic/BM25 pipeline, but it is not confirmed whether Projects use that exact stack.

Full research results: see the deep research report uploaded to the project.

### Problem 14 — Index Files Don't Work as Intended: PROHIBITED

An earlier version of this system used a hierarchical index pattern: a master `project-index` file referenced by custom instructions (read at conversation start), with themed sub-indexes for research domains. The theory was that Claude reads the index, learns what files exist, then retrieves the right one.

**Analysis:** This pattern fails for multiple reasons discovered through systematic examination:

1. **No multi-pass retrieval.** The RAG system retrieves chunks in a single pass per turn. An index file is just another document whose chunks may or may not be retrieved. There is no guaranteed "read index first, then fetch referenced file" pipeline.
2. **Context-loading via custom instructions is a workaround, not a solution.** Forcing the index into conversation context via custom instructions does make it available — but custom instructions are injected into every message, and the index consumes context window space proportional to project size, exactly when context space is most scarce.
3. **The paradox of scale.** Small projects don't need indexes (few files, RAG handles it). Large projects need orientation, but the index itself becomes large enough to be counterproductive. Indexes are either unnecessary or expensive.
4. **Self-describing files solve the problem at the source.** If every file has a descriptive name, a retrieval-optimized summary, and clear headings, RAG can find files directly without a routing layer.

**Decision:** Index files are prohibited (§4, File Categories). No custom instructions for index reading. Each file is self-describing.

### Problem 15 — No File Categorization: ACCEPTED (then refined by adversarial review)

The system had only two file types: `guidelines` and `context`. Research reports and other backing material were treated identically to core decision files, despite having very different retrieval characteristics (longer, denser, different vocabulary).

**Initial decision:** Added three new types — `context`, `knowledge`, and `reference`. The `knowledge` type was intended for distilled domain expertise not tied to project-specific decisions.

**Adversarial review outcome:** The `knowledge` type was removed. It fully overlapped both context and reference files: domain findings that inform a decision belong in the context file alongside that decision; findings not yet tied to a decision sit in the reference file waiting for a future session. The boundary test ("would this be useful in a different project?") sounded clean but was unenforceable in practice — different sessions would categorize the same information differently, creating duplicated content that diverges over time. See Problem 19 for the full rationale.

**Final decision:** Two non-guidelines types remain:
- `context` — project-specific decisions, rationale, constraints, current state.
- `reference` — backing material (research reports, deep research results). Longer, denser, lower retrieval priority by nature.

When sessions draw on reference files to reach decisions, those decisions are recorded in context files with a cross-reference to the source reference file.

### Problem 16 — No Retrieval Optimization Guidance: ACCEPTED

The guidelines had no guidance on making files findable by RAG. Filenames, summaries, and headings were treated as human-readable conveniences, not retrieval signals.

**Decision:** Added retrieval optimization guidance throughout:
- §1 (File Naming): naming for retrieval — domain-specific terms, not generic labels
- §7 (Retrieval Optimization): new section covering summary blocks, section headings, vocabulary alignment, and query phrasing. Opens with a caveat that exact RAG mechanics are undocumented.

The frontmatter `summary` field was initially strengthened for retrieval, then removed entirely after adversarial review (see Problem 20).

### Problem 17 — No Guidance on File or Project Scaling: ACCEPTED

The guidelines had no guidance on what to do when files grow too large or projects accumulate too many files. The one-theme-per-file rule (§6, File and Project Scope) prevented thematic scope creep but not size bloat — a single-topic file could grow to 63,000 words.

**Decision:** Added size-based file splitting guidance and project splitting guidance to §6. Includes degradation signals (generic responses, wrong files cited, content missed) and remediation steps (prune, tighten metadata, split).

### Problem 18 — Research Files Not Optimized Before Upload: ADDRESSED WITH PROMPT

Raw deep research output lacks frontmatter, descriptive filenames, and retrieval-friendly structure. Uploading it directly to a project makes it hard for RAG to find.

**Decision:** Created `reference-file-workflows_v2.md` — six graduated workflows for managing reference files, from lightweight in-session optimization to rare heavy splitting with cross-checks. See Workflow Restructure Decisions (Problems 36–48) for how this evolved from the original monolithic approach.

---

## Adversarial Review Decisions

An adversarial review of the v5 session output was conducted in a separate chat. The following problems were identified and resolved.

### Problem 19 — `knowledge` File Type Is Redundant: REMOVED

The `knowledge` type (distilled domain expertise, not project-specific) was introduced in the v5 session and removed after adversarial review.

**Rationale for removal:**
1. It fully overlaps both context and reference files. Domain findings that inform a decision belong in the context file alongside that decision. Domain findings not yet tied to a decision sit in the reference file waiting for a future session. There is no content that belongs in a knowledge file but not in one of those two.
2. The research optimization prompt already produces retrieval-optimized reference files with summary blocks, descriptive headings, and domain-specific vocabulary. A knowledge file would be a worse copy of that same content in a separate container.
3. The context/knowledge boundary is unenforceable in practice. The boundary test ("would this be useful in a different project?") sounds clean but falls apart on real content. Different sessions will categorize the same information differently, creating duplicated content across knowledge and context files that diverges over time.
4. Removing it eliminates cascading problems. The distillation mandate in §11 (What to Persist) becomes simpler: when you make decisions based on reference files, record those decisions in context files and cite the reference file.

### Problem 20 — Frontmatter `summary` Field Is Redundant: REMOVED

The `summary` field and the body summary block (§7, Retrieval Optimization) served the same purpose, but the frontmatter version was artificially constrained to one sentence by YAML format conventions. RAG doesn't parse YAML as structured data — it's just text that gets chunked with everything else. The `topic` field already provides a one-line scope label. The body summary block (2–4 sentences, domain-specific terms, key findings) does everything the frontmatter summary did, better.

**Decision:** Removed `summary` from mandatory frontmatter. The body summary block in §7 is now the sole summary mechanism.

### Problem 21 — Reference Files Need Visual Distinction: ACCEPTED (`r-` prefix)

Reference files use the prefix `r-` before the topic slug to help the user visually distinguish them from context files when scanning a file list. The prefix is short enough (2 characters) that it doesn't meaningfully reduce the descriptive portion of the filename or hurt retrieval.

**Decision:** Added `r-` prefix rule to §1 (File Naming). Pattern: `r-{topic-slug}_v{version}.md`.

### Problem 22 — Reference Files Need Staleness Tracking: ACCEPTED (`shelf_life` field)

The AI performing research optimization is well-positioned to estimate how quickly findings will go stale. A `shelf_life` field captures this as a plain-language estimate, which is immediately actionable when compared against the `last_updated` date. Also helps with project scaling (§6, File and Project Scope) — reference files with short shelf life that are well past their estimate are obvious candidates for removal or re-research.

**Decision:** Added `shelf_life` as a required frontmatter field for reference files. Omit for other types.

### Problem 23 — Research Optimization Prompts Should Not Hardcode `guidelines_version`: ACCEPTED

Both optimization prompts run outside the project context. The AI has no access to the guidelines file and no way to verify the current version. Hardcoding a version creates silent conformance drift when guidelines are updated.

**Decision:** Removed `guidelines_version` from the frontmatter template in both optimization prompts. The `created` and `last_updated` dates allow inferring the approximate era if needed.

### Problem 24 — Research Optimization Prompts Lack Splitting Guidance: ACCEPTED (then superseded)

Deep research reports can be very long. The optimization prompts added frontmatter and better headings but didn't address file length — a core retrieval problem. An optimization prompt that ignores file length fails at its primary job.

**Decision:** Added splitting instructions to both prompts — when to split, how to split, how to name split files. Added a confirmation gate and a two-step QA workflow.

**Superseded by Problem 37:** Splitting was later removed from the default optimization path after real-world cost/benefit analysis showed it generated disproportionate editorial errors. Splitting now exists only in Workflow 6 (heavy optimization).

### Problem 25 — §7 Overstates RAG Certainty: ACCEPTED

§7 (Retrieval Optimization) originally presented RAG mechanics as settled fact. The research found that exact retrieval mechanisms are not publicly documented.

**Decision:** Added caveat to §7's opening: guidance is based on observed behavior and published research, optimizing for both keyword and semantic matching.

### Problem 26 — Section Number References Go Stale: ACCEPTED

Problems in this file referenced bare section numbers (e.g., "§10") that need updating with every guidelines restructure. Growing maintenance burden.

**Decision:** Use section names alongside numbers throughout (e.g., "§10 (Non-Conforming Files)") to make references survivable across renumbering.

---

## Research Optimization Prompt Refinements

These decisions were made during testing and iterative refinement of the research optimization prompts in `prompt-research-optimization_v2.md` (before the v3 cross-check work).

### Problem 27 — `shelf_life` Reasoning Not Surfaced Before File Production: ACCEPTED

The original prompts included `shelf_life` as a frontmatter field but did not ask the AI to propose its estimate during the confirmation gate. The user only saw the estimate after the optimized files were produced — too late to challenge a bad judgment call without rework.

**Decision (two touch points, different forms):**
1. **Confirmation gate:** Both prompts now ask the AI to propose its `shelf_life` for each file with the specific factor that would make findings go stale. The user reviews and can challenge the estimate before files are produced.
2. **Summary block:** Both prompts now instruct the AI to include a natural-prose sentence explaining the `shelf_life` estimate in the body summary block — what would cause staleness and how quickly. This gives a future AI context to interpret the estimate intelligently (e.g., distinguishing "5 months old but staleness driver is annual regulatory cycles" from "5 months old but staleness driver is monthly tooling changes").

Frontmatter stays a clean machine-parseable value. Reasoning lives in the body where it serves both retrieval and human/AI interpretation.

### Problem 28 — Specific `shelf_life` Examples Caused Anchoring: REMOVED

The AI was defaulting to "3–6 months" for every file regardless of content. The cause was triple reinforcement: the frontmatter template listed "3-6 months" as the first example, the confirmation gate example used "3–6 months," and the summary block example used "3–6 months." The AI pattern-matched to the nearest example instead of evaluating the actual content.

**Decision:** Removed all specific time-range examples from all three `shelf_life` touch points in both prompts. Replaced with format descriptions that explain what's needed (a time range tied to the specific staleness driver) without supplying a value to copy. Testing confirmed the fix: the AI subsequently produced five distinct time ranges (3–6, 4–8, 6–12, 12–18 months) with content-specific reasoning across a six-file split.

**Rejected alternative:** Varying the examples across the three touch points (e.g., "3–6 months" in one, "6–12 months" in another) was considered as a lighter fix. Dismissed as a weak solution — it would reduce anchoring to a single value but still supply defaults to copy rather than forcing the AI to evaluate the content.

---

## Plan Cross-Check and Optimization Prompt v3 Refinements

These decisions were made during design of the plan cross-check prompt and refinement of the optimization prompts, based on a review of 11 deep research reports and their AI-generated optimization plans.

### Problem 29 — Optimization Plans Need Independent Review: ACCEPTED (separate-chat cross-check)

The AI that produced optimization plans consistently failed to catch its own reasoning contradictions — identifying different staleness drivers per file then applying uniform estimates, acknowledging content overlap then not addressing it. Same-chat role switching (as used in existing QA prompts) worked for completeness checks but was insufficient for evaluating judgment calls like split quality and shelf life honesty.

**Decision:** Added a cross-check step (Prompt 3 in `prompt-research-optimization_v3.md`) between the AI's proposed plan and user approval. Runs in a separate chat with two inputs: raw research and the AI's proposed plan. The reviewer reads research first (to build an independent mental model before seeing the plan), then reviews the plan through open-ended reading.

### Problem 30 — Cross-Check: Checklist vs. Open-Ended Review: OPEN-ENDED

Initial design prescribed 7 evaluation criteria derived from the review findings (shelf life differentiation, grab-bag detection, content boundary clarity, cross-reference strategy, filename quality, thin file justification, confirmation gate completeness). Rejected — the reviewer reliably caught these issues without guidance during the original review session, and a checklist risks narrowing attention to known failure modes at the expense of novel problems.

**Decision:** The prompt describes the task and output format, then gets out of the way. The review findings stay as the user's reference for monitoring cross-check quality over time, not as the reviewer's script.

### Problem 31 — Cross-Check Output Format: THREE-SECTION FENCED CODE BLOCK

**Decision:** Output is a single fenced code block ready to copy-paste into the optimization chat. Three sections ordered for the human reader: Issues (act on these — numbered, ordered by importance, each with a concrete suggested fix), What's Good (don't touch these during revision), Comments (not problems, but worth the optimization AI considering — may be empty).

The Comments section was added to handle observations that aren't wrong enough to be issues but are worth surfacing. Without it, borderline observations either get forced into Issues (overstating severity) or dropped entirely.

### Problem 32 — Cross-Check Sequential Use: INDEPENDENT PAIRS WITH BATCH GUIDANCE

When multiple research+plan reviews are run sequentially, two degradation risks apply: attention budget depletion as context accumulates, and cross-contamination where the reviewer pattern-matches against previous plans rather than reading each one fresh.

**Decision:** The prompt is pasted once and frames the session as processing one or more pairs. An instruction tells the reviewer to treat each pair independently. Usage guidance recommends a fresh chat every 3–4 reviews.

### Problem 33 — Which Optimization Prompt Improvements to Accept: THREE ACCEPTED, TWO REJECTED

Five improvements were proposed based on review of 11 optimization plans. Three accepted, two rejected:

**Accepted:**
- **Cross-reference strategy in the confirmation gate** — standalone value; when splits produce interdependent files, the plan must state how they reference each other. Cross-references are useful not as a RAG routing mechanism but as retrieval hygiene: when one file is retrieved and its response mentions a sibling file by name, those terms enter the conversation and improve subsequent query matching.
- **Per-file shelf life assessment** — flattened estimates were always wrong when they occurred; one sentence prevents a revision round-trip the cross-check would otherwise catch.
- **Filename durability guidance** — bad filenames get caught in review but fixing them costs a revision round that cheap prevention avoids.

**Rejected:**
- **Grab-bag warning** — the cross-check catches it through open-ended reading; adding a warning to the optimization prompt may bias the AI to avoid a problem it wouldn't have committed.
- **Content boundary instruction** — same reasoning; better caught by the cross-check than prevented by instruction weight.

**Principle:** The cross-check handles judgment-level issues; the optimization prompt handles mechanical instructions.

### Problem 34 — Filename Guidance: Negative to Positive Framing: POSITIVE

Initial approach listed specific things to avoid (dates, version numbers, tool/product names). These examples were domain-specific and wouldn't generalize to other research areas.

Evolved through several iterations:
1. "Avoid dates, version numbers, or tool/product names that may be superseded" — domain-specific examples
2. "Avoid dates, version numbers, or anything that may be superseded" — too broad; the AI would play safe with overly generic slugs since most terms could theoretically be superseded
3. Positive framing: "use terms a person would naturally use when asking about the topic, choosing words that will stay accurate over time" — domain-agnostic, no examples to anchor on

**Decision:** Positive framing (option 3). The cross-check catches edge cases the optimization AI misjudges.

### Problem 35 — QA Self-Check Missing Consistency Verification: ACCEPTED

The QA workflow in the optimization prompts checked completeness, summary/frontmatter, headings, and split quality — but not consistency of claims across files. When the same fact appeared in multiple files after a split, qualifications and caveats sometimes failed to travel with it.

Initial proposal included detailed sub-checks (table headers matching columns, formulas matching stated numbers) — these were domain-specific anchoring on catches from one review session.

**Decision:** Added a Consistency section to QA Step 1 with one general line: "Are claims, facts, and qualifications consistent — within the file and across sibling files if split?" Header simplified from "Internal consistency" to "Consistency" ("internal" added no meaning). Step 2 re-check inherits automatically.

---

## Workflow Restructure Decisions

These decisions were made after analyzing an optimization transcript (16 turns, ~1 hour, two source reports → five split files) and comparing it against a lightweight in-session optimization case. The analysis revealed a fundamental cost/benefit imbalance in the monolithic optimization workflow.

### Problem 36 — Optimization Workflow Cost/Benefit Imbalance: RESTRUCTURED

The transcript showed: 16 turns, multiple separate chats for cross-checks, two external review rounds, two QA re-checks, ~1 hour of focused effort, significant usage limits consumed. The output: five well-optimized reference files. Scaling this to kickstarting a new project (5–10 reports) would consume an entire day on knowledge base preparation alone.

Meanwhile, another deep research report consumed during a working session was optimized as part of the normal Case 3 end-of-session update — no standalone prompts, no confirmation gates, no cross-checks. The result was a reference file of comparable quality: good filename, strong summary block, correct frontmatter, thoughtful shelf life estimate with per-component breakdown, session-specific empirical additions clearly marked, cross-references to existing project files.

**Core principle established:** Spending hours building a knowledge base that goes stale in 3 months is not worth the effort. The workflow shouldn't get in the way. Retrieval may be slightly worse without exhaustive optimization, but it will be adequate. Additionally, Anthropic will likely continue improving RAG.

**Decision:** Replaced the monolithic optimization workflow with six graduated workflows ordered by effort. See Problem 38.

### Problem 37 — File Splitting Generates Disproportionate Errors: REMOVED FROM DEFAULT PATH

Tracing every error in the transcript to its origin revealed that virtually all of them were consequences of merging two overlapping reports and then splitting the result across five files. The AI had to make editorial judgment calls about where every paragraph, benchmark, and launch command belonged — and got many wrong. The QA rounds existed because the split created the errors.

Error categories traced to splitting: slug scope too narrow, content stranded in wrong file, architecture details claimed by wrong file, quantization guidance split ambiguously, launch commands in wrong files, decision guide re-deriving content from another file, scope violations.

Without splitting, these errors wouldn't exist. The mechanical optimization steps (frontmatter, summary, headings) generate almost no errors and are easily caught by a single QA pass.

**Decision:** Splitting removed from all workflows except Workflow 6 (heavy optimization), which is reserved for files that meet two conditions: the knowledge won't go stale soon *and* it will be used extensively for an extended period.

### Problem 38 — Six-Workflow Graduated Model: ACCEPTED

Replaced the monolithic optimization approach with six workflows ordered from simplest/most common to rarest/most complex:

1. **In-session optimization** — research consumed during a session, Case 3 produces reference files, supplementary QA
2. **Standalone single-report** — mechanical optimization, no editorial decisions, three steps
3. **Multi-report merge** — inventory-first approach, cross-check on inventory, one merged file out
4. **Integrating new reports into existing project knowledge** — runs inside the project, merge with existing file
5. **Knowledge base housekeeping** — periodic review for staleness, overlap, irrelevance, overgrowth
6. **Heavy optimization** — the only workflow that splits files, full cross-check sequence

All six workflows live in `reference-file-workflows_v2.md`.

### Problem 39 — In-Session Optimization as Default Path: ACCEPTED

The comparison case (Problem 36) demonstrated that the session AI, working within the project with full context, produces reference files that are well-integrated with project vocabulary, cross-referenced to existing files, and enriched with session-specific empirical findings that no standalone optimization workflow would capture.

**Decision:** Workflow 1 is the default path. For simple cases (one small file), a supplementary self-QA check suffices. For complex cases (substantial file or 2+ reference files), a post-production cross-check in a fresh chat verifies completeness. The cross-check reviewer receives the produced r- files and original source docs, checks completeness, and flags uncertain additions — understanding that the producing AI may have had good reasons from session context the reviewer can't see.

### Problem 40 — No Directional Bias Between Sources: ACCEPTED

Initial discussion proposed treating existing project files as the "baseline" that new information is integrated into, implying the older file has authority. The user rejected this: age doesn't confer authority. In fast-moving domains, the newer report may be more accurate. Running five parallel deep researches with the same prompt can produce five different answers due to seed variance — none is inherently more trustworthy.

**Decision:** No source ranking in any merge workflow. The AI reads all sources, surfaces contradictions explicitly, and escalates to the user when it can't determine the right call. When multiple findings coexist without a clear winner, all are documented with their context. This applies uniformly across Workflows 3, 4, and 5.

The one thing worth protecting in existing project files is structural: corrections from previous QA rounds, cross-references from context files, and project-specific annotations. These are caught by QA, not by source-ranking rules.

### Problem 41 — Contradiction Handling Rules: SURFACE, ESCALATE, OR DOCUMENT

Three acceptable responses to contradictions between sources:

1. **Surface:** Document both positions with their context in the output file
2. **Escalate:** When the AI can't determine which is correct, ask the user
3. **Document:** When multiple findings coexist without a clear winner (e.g., five parallel researches giving five different recommendations), preserve all with attribution

Never: resolve silently, average numbers, pick one source over another without stating the reasoning, or "reconcile" by dropping the minority position.

### Problem 42 — Inventory-First Merge Approach: ACCEPTED

Based on research into seed variance across parallel deep research runs. The core insight: LLMs default toward compression, and the whole point of running multiple research passes is to capture the long tail. An inventory-first approach prevents anchoring on the first report read.

**Mechanical steps:** Before writing anything, the AI builds a complete visible list of every distinct item across all inputs (models, claims, numbers, techniques, recommendations). For each item, notes which source(s) it appears in. Flags contradictions with proposed handling. This inventory becomes the confirmation gate — the user (or cross-check AI) reviews the list, which is cheaper than reviewing a finished document.

**Consensus tiering available but not mandated.** For same-prompt parallel runs, items can be categorized by how many sources mention them. For different-prompt reports, tiering may not be meaningful. The underlying principle — preserve everything, attribute single-source findings — holds regardless.

### Problem 43 — QA Output Restructure: NO AI VERDICTS

The AI consistently claimed files were "clean and ready for upload" and then subsequent QA runs found new issues. The AI can't reliably assess its own blindspots — if it could see the remaining issues, it would have already fixed them.

**Decision:** Removed all verdict and confidence language from QA output. Replaced with factual reporting:
- **New issues found and fixed** (this run only — not cumulative across runs)
- **Needs your input** (ambiguities the AI can't resolve)
- **What was checked** (in the AI's own words)
- **Run history** (from run 2 onward) — table showing issue count and severity per run

The run history gives the user trend data: "run 1: 3 issues (2 major, 1 minor), run 2: 2 issues (1 major, 1 minor), run 3: 1 issue (minor)" — the downward trend and shift from major to minor is more reliable decision data than the AI's opinion of itself. In practice, three QA rounds typically reach diminishing returns.

The AI may say "no new issues found this run" but must never say "the file is clean." The first is a factual claim about what this pass caught; the second is a claim about the file's state that the AI is not qualified to make.

### Problem 44 — Workflow Simplifications: MULTIPLE

Several workflow steps were removed after analysis showed they added complexity without proportionate value:

- **Confirmation gate removed from Workflow 2.** Standalone single-report optimization is purely mechanical — no editorial decisions about splitting, merging, or content placement. The filename, shelf life, and summary are visible in the output. If wrong, the self-QA or user catches it. Removing the gate reduces the workflow from 5 steps to 3.
- **Pre-optimization removed from Workflow 4.** The integration AI doesn't need frontmatter and improved headings to understand the research content. The output already has proper structure because the existing project file has proper structure. Pre-optimizing is polishing something about to be disassembled.
- **Two-report ceiling for Workflow 4.** Context window pressure with the existing project file, two reports, conversation history, and project RAG results increases risk of silent content loss. One or two reports per integration session; three or more should be split across sessions.
- **Housekeeping merges execute in-project (Workflow 5).** The AI can see both files through project context, knows the project vocabulary, and can check cross-references. No reason to route to a separate Workflow 3 chat.

### Problem 45 — Cross-Check Chat Design: DUAL PURPOSE

In Workflows 4 (2+ reports) and 6, the cross-check chat reviews the plan first, then reviews the produced files. Same reviewer, same source context, two passes, one extra chat. This avoids redundant context setup — the reviewer already has the source material and the plan, so post-production review is a natural continuation.

In Workflow 1 (complex cases), the cross-check is post-production only — the Case 3 prompt doesn't produce a reviewable plan. In Workflow 3, the cross-check reviews the inventory (which serves as the plan), with optional post-production review for 3+ source reports.

### Problem 46 — Cross-Check Refinements from External Review: SIXTEEN FIXES APPLIED

The initial v4 draft was cross-checked by an external AI reviewer, which identified 17 concerns (4 major, 7 medium, 6 minor). Sixteen were accepted and applied to the final `reference-file-workflows_v1.md`:

**Major fixes:**
- **Workflow 5 can't sweep all files via RAG** — replaced "review every reference file" with `r-` prefix scanning: AI reads summary blocks of every file starting with `r-`, lists what it found, user confirms completeness before review proceeds.
- **Deletion impact assessed too late** — moved cross-reference breakage check from post-approval (step 5) into the recommendation list (Prompt 5A) so user approves with full cost information.
- **Workflow 4 assumed single target file** — added handling for three scenarios: one matching file (proceed), multiple sibling files (map content, flag ambiguous placement), no match (redirect to Workflow 2).
- **Merge base rule was arbitrary** — removed "higher-versioned file as base" proxy rule. The AI reads both files and decides based on comprehensiveness and scope, states its reasoning. No proxy rules that encode false assumptions.

**Medium fixes:**
- Deduplicated shared principles — one canonical version in `reference-file-workflows_v1.md` (the two-file split was also eliminated, see Problem 47).
- Consolidated Workflow 2's two near-identical prompts into one with natural phrasing covering both scenarios (reviewer suggested bracketed placeholder; user preferred natural phrasing — different implementation, same outcome).
- Reframed Prompt 3A's contradiction categories as inventory item handling (four labeled dispositions) rather than mixing agreement and contradiction types.
- Aligned Workflow 6 cross-reference instructions — symmetric references required in both 6A (production) and 6D (review) to prevent valid authoring choices being flagged as issues.
- Replaced "beefy file or multiple files" threshold in Workflow 1 with nature-of-work heuristic: cross-check when the AI made significant additions beyond the source research or reconciled against existing project knowledge.
- Added explicit note in Workflow 2 design rationale acknowledging the Problem 27 tradeoff (shelf_life gate removal).
- Added "including any corrections, annotations, or adjustments applied in previous sessions" to Workflow 4's preserved-content checks (Prompts 4A and 4C) — addresses the reviewer's structural-elements concern with explicit wording rather than a separate check.

**Minor fixes:**
- Removed "Brief" from Prompt 3A changelog template (contradicted no-compression principle).
- Added clearer file-location note to overview table.
- Added "reconstruct run history from prior QA turns" note to QA output format.
- Added explanatory note to Prompt 1B about why it differs from fenced-block cross-check pattern.
- Added bootstrap guidance under overview ("Run Workflow 2 for each distinct-topic report; Workflow 3 for overlapping ones").

**Left as-is (1 concern):**
- Plan + execution in single prompt (Concern 10) — "Wait for my approval" language has worked reliably; strengthened to "Do not continue past this point until I reply" in Workflow 6 as a precaution but no evidence of failure in other workflows.

**Second-round refinements.** The same reviewer conducted a follow-up pass on the revised files and identified additional items. Two had design content worth preserving:

- **"Fits no sibling" gap in Prompt 4A** — the multi-sibling integration scenario originally said "some content may not fit any" sibling file without instructing what to do with such content. Added explicit handling: the AI flags unplaceable content separately so the user can decide whether to run Workflow 2 for that portion. Silent placement into the closest sibling would cause scope drift.
- **Bidirectional → symmetric cross-references.** First-round fix required "bidirectional" cross-references between sibling files in Workflow 6 (applied to both 6A production and 6D review to prevent flagging valid authoring choices as issues). The reviewer correctly noted this swung past what's useful: with 4+ split files, requiring every file to reference every sibling in a 2–4 sentence summary block crowds out findings. Changed to "symmetric" — if file A references file B, file B must reference file A, but not every file needs to reference every sibling. This preserves the consistency check (6D can still verify) without mandating exhaustive cross-reference lists.

Three additional second-round items were purely editorial (summary block problem number range, counting correction, changelog updates) and are captured in the changelog without body documentation.

### Problem 47 — Two-File Split Was Unnecessary: MERGED INTO ONE FILE

The initial draft split workflows into `prompt-research-optimization_v4.md` (Workflows 1–3, 6) and `prompt-knowledge-base-upkeep_v1.md` (Workflows 4–5). The split was motivated by "optimization vs. maintenance" but in practice all six workflows are steps in the same lifecycle: research enters the project, gets integrated, gets maintained, occasionally gets restructured. Workflow 4 straddled both files. The split also caused shared principles to be duplicated with immediate wording drift (Concern 5).

**Decision:** Merged into `reference-file-workflows_v1.md`. One file, one lifecycle, shared principles defined once.

### Problem 48 — `prompt-` Prefix No Longer Serves a Purpose: STRIPPED

The `prompt-` prefix originally distinguished prompt-containing files from other context files. As the project grew, nearly every context file became a prompt file, making the prefix noise that tells you nothing you don't already know. The `r-` prefix earns its keep because reference files are categorically different. The `prompt-` prefix doesn't serve that function.

**Decision:** Stripped from the new merged file. Other existing files to be renamed in a separate session to avoid bloating this one's scope. Cross-references updated accordingly.

---

## Documentation Principles Decisions

### Problem 49 — Rationale Abstraction Level: ACCEPTED

Design rationale in project files was being written at the instance level (naming specific tools, reports, personal habits, and session narratives) rather than at the level of the underlying design constraint. Instance-bound rationale ages poorly: tools change, reports get archived, session narratives lose context, and rationale that depended on naming them becomes stale or opaque to readers outside the originating context. The same rationale written at the constraint level ("mobile device usage" rather than a specific personal situation, "a specific-model report" rather than naming the model) remains useful indefinitely.

**Forcing function:** Preparing the project for a public GitHub repository surfaced the issue, but review revealed the deeper principle holds regardless of distribution — instance-bound wording was already working poorly for anyone outside the originating session. Privacy is a byproduct of writing durable rationale, not the point.

**Decision:** Created `documentation-principles_v1.md` as a project-specific writing guide under the §5 scope carve-out. Core principle: write rationale at the level of the design constraint, not the instance that surfaced it. The principle applies to authoring (primary use) and editing (secondary use), including changelog entries — an edit description that names what was changed re-introduces it.

**Load-bearing subtlety:** Generalize to the level where information does its job, not past it. Specifics that support quantitative arguments, name the actual subject matter, or anchor a point that would fail without them should stay specific. The test is "what work is the specificity doing?" not "how specific is this?"

**Scope:** The principle is project-specific, not universal. Adding it to guidelines would contradict §5's explicit carve-out for writing-level concerns and would propagate writing-level rules to other projects using the system. The two illustrative examples in guidelines that were previously instance-specific (§1 naming examples, §6 size-based splitting example) were generalized in `guidelines_v6.md` — but that's an improvement to guidelines content, not a structural change that ties guidelines to this project. An earlier draft included GitHub distribution operational content (push checklists, commit message guidance); this was dropped because no recurring prep step exists to document.

---

## Workflow Modernization Decisions

These decisions were made during a review of the core QA and context update workflows, prompted by accumulated friction and misalignment with patterns established in the newer reference-file workflows.

### Problem 50 — Project Setup Prompt Retired: REMOVED

`project-setup_v4.md` provided a prompt for generating a project's name, description, and custom instructions from conversation content or attached files. Removed from the project. Name and description work as casual asks; no formal prompt needed. The custom instructions section was structurally broken: the instructions worth formalizing — behavioral corrections from repeated usage friction — don't exist in context files or conversation history, so the AI can't derive them. The prompt's guardrails correctly defaulted to "no instructions needed," making it a well-engineered no-op.

Surviving design principle: only add custom instructions when the rule is (a) not covered by any project file, (b) relevant to every conversation in the project, and (c) a directive, not context. Context belongs in context files.

### Problem 51 — QA Workflow Collapse and Re-Check: RESTRUCTURED

The QA workflow was restructured from a two-prompt separation (check then fix) to a single combined prompt, and a dedicated re-check prompt was added as the expected second step.

**Collapse rationale:** The two-prompt separation (diagnose, then wait for user approval, then fix) added friction without proportionate value. Most QA findings were clear-cut improvements — phrasing accuracy, missing nuance, minor structural issues — that didn't require user judgment before acting. For a typical 3-round QA, the two-step approach required 6 prompt pastes; the combined approach requires 3. The reference-file workflows (`reference-file-workflows_v2.md`) already used the combined pattern on editorially heavier material without quality loss.

**User decision authority preserved through escalation.** The combined prompt applies clear fixes directly and escalates ambiguous items under "Needs your input." The structured QA Summary gives full visibility into what was changed. The user can call out any disagreement after the fact — reverting a targeted fix is expected to be straightforward, though this is worth monitoring as the pattern sees more use.

**Re-check as explicit step.** A single QA pass is rarely sufficient — three rounds typically reach diminishing returns. Re-pasting the same QA prompt was the previous workaround, but it primes the same mental model and fights against fresh-eyes framing. The dedicated re-check prompt has two tasks: verify fixes landed cleanly (catching fix-introduced regressions), then re-read from a different angle. The "different angle" nudge pushes against the AI re-confirming prior findings without prescribing what to look at — prescribed per-round categories would create a ceiling by stopping the AI from looking outside them.

**No-verdicts alignment.** The previous Pass / Pass with notes / Fail verdict system was the exact pattern Problem 43 rejected. Replaced with the structured QA Summary format (issues found and fixed, needs your input, what was checked, run history from run 2 onward). The run history table provides trend data for the user to decide when to stop.

### Problem 52 — Context Workflow Case Reorder and Portability: REORDERED

Cases in `context-workflow_v8.md` were reordered by usage frequency rather than project chronology. Steady state (daily use) is now Case 1; transitional remains Case 2; bootstrap (one-time) is now Case 3. The previous ordering followed the lifecycle sequence (bootstrap → transitional → steady state), which matched how a project is set up but not how a user reaches for the prompts — steady state is used in virtually every session.

**Artifact language.** All three case prompts now specify "output each file as a separate downloadable `.md` artifact" rather than relying on implicit compliance with the output contract. This was prompted by cross-platform portability: some AI platforms default to fenced code blocks when the format expectation isn't explicit. The positive framing ("downloadable `.md` artifact") is sufficient without adding a negative instruction ("not as fenced code blocks") — if the positive instruction proves insufficient, the negative can be added later.

### Problem 53 — Proposal QA Alignment: UPDATED

The parallel workflow's Proposal QA (Prompt 2 in `parallel-workflow_v7.md`) was updated to align with the modernized QA patterns:

- Replaced verdict language ("If the proposal is accurate and complete, confirm that explicitly") with the structured QA Summary format.
- Added clear/ambiguous fix distinction: clear issues are fixed directly to the artifact, ambiguous items escalated under "Needs your input."
- Added a dedicated Proposal QA Re-Check prompt (Prompt 3) with the same two-task structure (verify fixes, then fresh re-read from different angle) and run history from run 2 onward.

The stakes argument is particularly strong here — the Proposal QA is the last moment the full conversation is visible, and the design rationale explicitly calls this out. An extra re-check round is almost always justified.

### Problem 54 — Changelog Verbosity: FIXED IN GUIDELINES

Changelog entries were consistently verbose — re-explaining context, re-describing what was replaced, treating entries as mini design rationales rather than change records. The pattern appeared independently across separate sessions, suggesting a systemic gap rather than a one-off authoring miss. The guidelines (§8) specified format but said nothing about density or conciseness, leaving no authoring signal to prevent the problem.

**Decision:** Added conciseness guidance to §8 in `guidelines_v7.md`. Initial wording biased toward minimality ("as lean as possible," "not re-explain context") and produced over-trimming when tested as a review criterion. Revised to event-level framing: entries should tell a reader what happened to the file, enough to follow its history without reading the body.

**Rejected alternative:** Framing entries as "what changed and why" was considered — it's the most natural-sounding option. Rejected because "why" invites design rationale, which was the content being over-included. Event-level framing naturally accommodates change-motivation ("per QA findings," "to align with guidelines v7") without inviting rationale.

Added changelog conciseness to the QA structural check. Existing verbose entries in the backlog compound the problem — the AI pattern-matches old entries when writing new ones, undermining the §8 rule until the backlog is cleaned. A transitional cleanup-as-you-go approach (trimming old entries when files are touched for other reasons) addresses this without a dedicated cleanup session.

---

## Deferred Items Summary

These were deliberately set aside. Do not re-propose them unless the user raises the topic:

- **Changelog pruning** — no automatic limit on changelog length
- **Token budget management** — no max file size, max file count, or priority field
- **User correction mechanism** — no `user_notes` field or correction section

---

## Dropped Review Findings

These were raised during the adversarial review and explicitly dismissed. Included here so future sessions don't re-raise them:

- **Index prohibition too absolute** — Dropped. No viable middle ground between a cheap map (just filenames, which RAG already has) and a useful map (thousands of words injected into every message).
- **"Preserve all content" vs "trim filler" contradiction in optimization prompt** — Dropped. Trimming filler requires judgment calls the optimization chat isn't equipped to make. If the file is too long, the splitting mechanism handles it.
- **Non-research files attached to batch prompt** — Dropped. Edge case too rare to justify instruction weight.
- **Prompt 1 could be shorter than Prompt 2** — Dropped. Both need the full template.
- **Parallel workflow Prompt 1 says "context files" not "all files"** — Dropped. Reference files don't need updating through proposals, so "context files" is correct.

---

## Changelog

### v8 — 2026-04-21
- Added Workflow Modernization Decisions section with Problems 50–54: project setup prompt retirement (Problem 50), QA workflow collapse and re-check restructure (Problem 51), context workflow case reorder and artifact portability language (Problem 52), proposal QA alignment (Problem 53), changelog verbosity fix in guidelines (Problem 54).
- Upgraded to `guidelines_v7.md` conformance.

### v7 — 2026-04-20
- Added Problem 49 to the Decisions by Problem set, establishing the documentation principles decision (create `documentation-principles_v1.md` as a project-specific writing guide under guidelines §5's scope carve-out). Records the forcing function, the load-bearing subtlety (generalize to where info does its job, not past it), and the scope rationale for not modifying guidelines' universal rules.
- Generalized instance-specific tool references and user-attribution framing throughout per `documentation-principles_v1.md`. No design decisions changed; only wording was pulled up to the constraint level.
- Upgraded to `guidelines_v6.md` conformance.
- Updated cross-references in Problems 18 and 38 to `reference-file-workflows_v2.md`. Problem 46 references to prior filenames preserved — those reference the filename as it existed at the time of each decision, and §6 (Cross-References Between Files) treats a stale version reference as a useful signal that the referencing content predates subsequent changes.

### v6 — 2026-04-19
- Added Workflow Restructure Decisions section with Problems 36–48: cost/benefit imbalance analysis from real-world transcript (Problem 36), splitting removed from default path (Problem 37), six-workflow graduated model (Problem 38), in-session optimization as default (Problem 39), no-bias merge principle (Problem 40), contradiction handling rules (Problem 41), inventory-first merge approach (Problem 42), QA output restructure removing AI verdicts (Problem 43), workflow simplifications including confirmation gate removal, pre-optimization removal, two-report ceiling, in-project merges (Problem 44), dual-purpose cross-check chat design (Problem 45), sixteen cross-check refinements from external review (Problem 46), two-file split eliminated (Problem 47), `prompt-` prefix stripped (Problem 48).
- Updated Problem 18 cross-reference to `reference-file-workflows_v1.md`.
- Updated Problem 24 to note it was superseded by Problem 37.
- Updated summary block to reference v4 workflow restructure and cross-check refinements.
- Second-round cross-check fixes: corrected Problem 46 fix count (14→16) and restructured to clarify which concerns were resolved with different implementations vs. left as-is; changed Workflow 6 cross-reference requirement from "bidirectional" to "symmetric" to prevent summary block overcrowding in larger splits.

### v5 — 2026-04-15
- Added Plan Cross-Check and Optimization Prompt v3 Refinements section with Problems 29–35: separate-chat cross-check design (Problem 29), open-ended vs. checklist review (Problem 30), three-section output format (Problem 31), sequential use with batch guidance (Problem 32), which prompt improvements to accept/reject (Problem 33), filename guidance negative-to-positive framing (Problem 34), QA consistency section (Problem 35).
- Updated Problem 18 cross-reference from `prompt-research-optimization_v2.md` to `prompt-research-optimization_v3.md`.

### v4 — 2026-04-14
- Added Research Optimization Prompt Refinements section with Problems 27–28: `shelf_life` reasoning surfaced at confirmation gate and summary block (Problem 27), specific time-range examples removed to prevent anchoring (Problem 28).
- Updated Problem 18 cross-reference from `prompt-research-optimization_v1.md` to `prompt-research-optimization_v2.md`.

### v3 — 2026-04-12
- Upgraded to `guidelines_v5.md` conformance. Updated `guidelines_version` from `"4"` to `"5"`. Updated Origin section to reference `guidelines_v5.md`.
- Added V5 Decisions section documenting Problems 14–18: index file prohibition, file categorization, retrieval optimization guidance, file/project scaling, and research optimization prompt.
- Added Adversarial Review Decisions section documenting Problems 19–26: `knowledge` type removal, `summary` field removal, `r-` prefix, `shelf_life` field, `guidelines_version` removal from optimization prompts, splitting guidance, RAG certainty caveat, and section name references.
- Added Dropped Review Findings section.
- Updated section number references throughout to include section names alongside numbers for resilience across renumbering.
- Removed `summary` frontmatter field; file now uses body summary block per §7 (Retrieval Optimization).

### v2 — 2026-03-07
- Upgraded to `guidelines_v4.md` conformance. Updated filename to new naming convention. Updated Origin section to reflect current authoritative guidelines file (`guidelines_v4.md`). Added parenthetical to Problem 5 noting the later naming convention change. Removed old-convention filenames from Problem 6 example (replaced with convention-neutral phrasing). Updated `guidelines_version` from `"2"` to `"4"`.

### v1 — 2026-03-06
- File created
