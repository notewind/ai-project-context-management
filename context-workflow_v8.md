---
type: context
topic: "Context Workflow Prompts"
version: "8"
guidelines_version: "7"
created: 2026-03-06
last_updated: 2026-04-21
---

Three context update prompts (steady state, transitional, bootstrap) with their design rationale, plus rationale for the output format and QA workflow. Ordered by usage frequency: Case 1 is the daily routine, Case 3 happens once.

# Context Workflow Prompts

This file contains the prompts that enforce the guidelines file at session end, along with the design rationale behind each one. A future AI working in this project should understand both *what* to do (the prompts) and *why* it's structured that way (the rationale) so it doesn't accidentally undo deliberate choices.

All prompts are designed to be pasted at the end of a conversation.

**Parallel chats:** If you are running multiple chats concurrently in the same project, do not use these prompts. Use the parallel workflow in `parallel-workflow_v7.md` instead — it produces change proposals rather than files, then merges them in a separate reconciliation step. The prompts below assume a single active chat with exclusive write access to the project files.

---

## Case 1 — Steady State: Established Project, Routine Update

**When to use:** This chat was created inside a well-established project. The guidelines file and context files are already in the project's files section. The AI has had access to them throughout the conversation. You're now at the end of the session and need the AI to do its update duties.

**How to use:** Paste this prompt at the end of the conversation.

### Design Rationale

- No confirmation gate. In steady state, files are already clean, the AI has been working within the guidelines all session, and the risk of a bad call is low. The friction of a mandatory approval step isn't worth it for routine updates. A confirmation gate can be added by inserting "Wait for my confirmation before producing files" after the summary step.
- The AI is still required to summarize its plan before outputting files — this provides a lightweight check without blocking execution.

### Prompt

```
We're at the end of this session. Perform your context file update duties per the project guidelines.

Specifically:

1. Review the guidelines file in the project files. Confirm you understand the current version and rules.

2. Review all existing context files in the project. Check for version mismatches against the current guidelines and flag any if found (§9).

3. Review our conversation. Identify anything worth persisting per the persistence criteria (§11).

4. Match persistent content to existing files by topic scope. If something doesn't fit any existing file, flag it as a candidate for a new file.

5. Summarize your plan in chat before producing any files:
   - Which existing files are being updated, and what's changing
   - Any new files being created, with justification (§12.5)
   - Any version mismatches or issues found
   - If nothing needs updating, say so explicitly (§12.4)

6. After your summary, output each file as a separate downloadable `.md` artifact per the output contract (§12).
```

---

## Case 2 — Transitional: Project Exists But Needs Integration

**When to use:** You're moving a chat into an existing project, or the project's files have gotten out of sync since this chat started (e.g., you worked with a different AI, updated guidelines elsewhere, or the project evolved outside this conversation). Files may be non-conforming, outdated, or conflicting.

**How to use:** Attach `guidelines_v{x}.md` and any existing context files (whether conforming or not) to the chat, then paste this prompt.

### Design Rationale

- Originally had three phases (Audit → Plan → Execute) with two pauses. Consolidated to two phases (Audit & Plan → Execute) with one pause after testing revealed the three-phase structure caused the AI to list every problem twice — once when auditing, then again when proposing fixes. Merging audit and plan into a single pass eliminates the redundancy while preserving the user's decision point before any files are produced.
- Non-conforming files are treated as raw material. The AI extracts their content and rewrites to guidelines spec — it does not try to patch non-conforming files in place.
- The AI must not assume how to fix non-conforming files. It reports issues and presents options; the user makes the call. This is critical because the transitional state has the most ambiguity and the highest risk of the AI making a bad judgment call.

### Prompt

```
I'm attaching a guidelines file and one or more existing project files. Some of these files may already conform to the guidelines. Others may be outdated, partially conforming, or in a completely different format. I need you to integrate the context from our conversation into this existing file set.

Do this in two phases. Do not skip ahead — wait for my input after Phase 1 before proceeding.

**Phase 1 — Audit & Plan**

Read the guidelines file completely, then examine every other file I've attached (and any files visible in the project, if this chat is inside a project). For each file, report its status and your recommended action in a single pass:

- **Conforming files:** Note the file is conforming. If its `guidelines_version` is behind the current guidelines, note what needs upgrading. If content from our conversation belongs in this file, describe what would be added or changed.
- **Non-conforming files:** Describe what specifically is wrong (missing/malformed frontmatter, wrong naming, structural issues). Recommend whether to rewrite it to spec (treating its content as raw material), merge it with another file, or discard it. Explain your reasoning.
- **Duplicate files:** If multiple versions of the same file exist, identify which is latest and recommend deleting the others.
- **New files needed:** If content from our conversation doesn't belong in any existing file, propose new files with topic slugs and one-line scope descriptions. Explain why no existing file was the right home.
- **Conflicts:** For any overlapping scope or ambiguous cases, present options and let me choose.

Wait for my approval or adjustments before proceeding.

**Phase 2 — Execute**

After I've confirmed the plan, output each file as a separate downloadable `.md` artifact per the output contract (§12). For updated files, preserve unchanged content (§12.3). For files rewritten from non-conforming originals, start at v1 with a changelog note indicating the source material.
```

---

## Case 3 — Bootstrap: No Project Exists

**When to use:** You had a conversation in a standalone chat (no project). You now want to extract context from this conversation into properly formatted context files that will seed a new project.

**How to use:** Attach `guidelines_v{x}.md` to the chat, then paste this prompt.

### Design Rationale

- The prompt is pasted *after* the conversation, not before. The AI reviews the conversation retroactively.
- There is a mandatory confirmation gate: the AI proposes file splits (topic slugs, scopes, what's included/excluded) and waits for approval before producing files. This matters because the AI is making judgment calls about topic boundaries with no prior project structure to guide it.
- All files start at v1. This is a one-time bootstrap — after uploading the output, the user is in Case 1 territory.

### Prompt

```
I'm attaching a guidelines file for a project context management system. I need you to do the following:

1. Read the attached guidelines file completely. Every rule in it applies to what you produce.

2. Review our entire conversation above. Identify all information worth persisting into project context files. Apply the persistence criteria from the guidelines (§11) — keep decisions, rationale, constraints, open questions, and anything a future AI assistant would need to reconstruct the mental model. Discard transient back-and-forth, debugging noise, and anything that wouldn't help a future AI working on this topic.

3. Organize what you found into context files, one coherent topic per file (§6). If everything fits in one file, that's fine. If the conversation spanned multiple distinct topics, create multiple files.

4. Before producing any files, summarize in chat:
   - How many files you plan to create
   - The proposed topic slug and one-line scope for each
   - A brief note on what you're including and what you're leaving out

   Wait for my confirmation before producing the files. I may want to adjust scope, rename topics, or exclude something.

5. After I confirm, output each file as a separate downloadable `.md` artifact per the output contract (§12) — conforming to all naming, frontmatter, versioning, and changelog rules.

These will be the first files in a new project, so everything starts at v1.
```

---

## Notes on Case Prompts

- **Case 1 does not ask the AI to wait for confirmation** before producing files. This is intentional — in steady state, the overhead of a confirmation step isn't worth it for routine updates. If you want the confirmation step, add "Wait for my confirmation before producing files" after step 5.
- **Case 2 uses a single pause between audit/plan and execution.** The audit and plan are merged into one phase to avoid the AI restating problems twice. You still review everything before any files are produced.
- **All three prompts reference guideline section numbers.** If the guidelines file is updated and section numbers shift, these prompts should be reviewed.

---

## Output Format: Why Artifacts, Not Fenced Code Blocks

The original design used labeled fenced code blocks in chat. Testing revealed this was painful in practice: fenced blocks require manually copying text, creating `.md` files, naming them correctly, and uploading them. Downloadable artifacts provide a download button and copy option, which maps directly to the upload-to-project workflow.

The tradeoff: artifacts are platform-specific, while fenced blocks work everywhere. Since this system targets AI platforms that support artifact output, this tradeoff is acceptable. The "downloadable `.md` artifact" language in each prompt makes the format expectation explicit across platforms.

---

## QA Workflow

QA is handled by two prompts in `qa-workflow_v7.md`, run in sequence in the same chat that produced the files.

### Why QA Is a Separate Prompt (Not Built Into the Output Contract)

Two reasons:
- **Self-checking in the same pass is unreliable.** The AI is anchored to its own output and sees what it intended, not what it produced. A separate prompt forces a role switch from author to reviewer.
- **Built-in QA taxes every session.** Most sessions produce small updates. A mandatory QA pass adds cost and latency to the common case to protect against the uncommon case (large or complex files). A separate prompt lets the user run QA selectively.

---

## Changelog

### v8 — 2026-04-21
- Reordered cases by usage frequency: Case 1 is now Steady State (daily use), Case 2 remains Transitional, Case 3 is now Bootstrap (one-time use). Previous ordering followed project chronology rather than usage frequency.
- Added "downloadable `.md` artifact" language to all three case prompts for explicit format expectation across AI platforms.
- Updated Output Format rationale to reflect platform-neutral framing.
- Updated QA workflow cross-reference to `qa-workflow_v7.md`.
- Updated parallel workflow cross-reference to `parallel-workflow_v7.md`.
- Updated Notes section to reflect new case numbering.
- Upgraded to `guidelines_v7.md` conformance.

### v7 — 2026-04-20
- Generalized user-attribution framing to constraint-level descriptions in the Output Format rationale section per `documentation-principles_v1.md`.
- Upgraded to `guidelines_v6.md` conformance.

### v6 — 2026-04-20
- Renamed file from `prompt-context-workflow_v5.md` to `context-workflow_v6.md` (removed `prompt-` prefix).
- Updated cross-references: `prompt-parallel-workflow_v5.md` → `parallel-workflow_v6.md`, `prompt-qa-workflow_v5.md` → `qa-workflow_v6.md`. Version numbers bumped because the referenced files were renamed in the same synchronized update (§6, Problem 6 staleness test).

### v5 — 2026-04-12
- Upgraded to `guidelines_v5.md` conformance. Updated all section references to match v5 numbering: §5→§6, §7→§9, §9→§11, §10→§12, §10.3→§12.3, §10.4→§12.4, §10.5→§12.5.
- Removed `summary` frontmatter field; added body summary block.
- Updated cross-references to `prompt-qa-workflow_v5.md` and `prompt-parallel-workflow_v5.md`.

### v4 — 2026-03-22
- Updated parallel workflow cross-reference from `prompt-parallel-workflow_v1.md` to `prompt-parallel-workflow_v2.md`.

### v3 — 2026-03-22
- Added parallel chats notice at the top of the file, directing users to `prompt-parallel-workflow_v1.md` when running concurrent sessions.
- Updated QA workflow cross-reference from `prompt-qa-workflow_v2.md` to `prompt-qa-workflow_v3.md`.

### v2 — 2026-03-07
- Upgraded to `guidelines_v4.md` conformance. Updated all references to use new naming convention (`_v{N}` instead of `_V_{N}`). Updated cross-reference to `prompt-qa-workflow_v2.md`. Updated `guidelines_version` from `"2"` to `"4"`.

### v1 — 2026-03-06
- File created (merged from `prompt-context-updates_v1.md` and `prompt-and-workflow-design_v1.md` to eliminate content overlap)
