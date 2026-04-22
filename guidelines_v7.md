---
type: guidelines
topic: "project context management"
version: "7"
created: 2026-03-02
last_updated: 2026-04-21
---

Master rules for creating, naming, versioning, and updating all project context files. Covers file categorization (context and reference), retrieval optimization for RAG, and project scaling guidance.

Every AI assistant working within this project must read and follow this file before producing or modifying any context file. This file is itself subject to its own rules.

---

## 1. File Naming

All files must embed their version number in the filename:

`{topic-slug}_v{version}.md`

Examples: `guidelines_v1.md`, `auth-system_v3.md`, `api-design_v12.md`

The version in the filename must always match the `version` field in the file's frontmatter. When a file is updated, both must change together.

### Naming Rules

- **Word separators:** Use hyphens between words in the topic slug. No underscores, spaces, or special characters within the slug.
- **Version separator:** Use a single underscore before the version marker (`_v`). This is the only underscore in the filename.
- **Version marker:** Lowercase `v` followed by the integer version number, no space or separator between them.
- **Topic slugs:** Use lowercase letters, numbers, and hyphens only. Be concise but descriptive (2–4 words preferred, e.g., `auth-system`, `api-design`, `onboarding-flow`). Once a topic slug is established, do not rename it without user instruction.
- **Reference file prefix:** Reference files (`type: reference`) use the prefix `r-` before the topic slug: `r-{topic-slug}_v{version}.md`. Example: `r-kubernetes-networking_v1.md`. This visually distinguishes reference files from context files when scanning a file list.

The underscore before `v` acts as a visual parsing seam between the topic slug and the version number — both for scanning a file list and for any tooling that may parse filenames later.

### Naming for Retrieval

File names are a retrieval signal — they influence which files RAG surfaces for a given query. Name files using the domain-specific terms a person would naturally use when asking about the topic:

- **Do:** `r-postgres-to-dynamodb-migration_v1.md`, `vendor-contract-review-process_v2.md`
- **Don't:** `r-research-results-7_v1.md`, `notes-last-session_v1.md`

Reference files especially benefit from descriptive naming, since they are longer and denser than context files and rely more heavily on filename matching to be retrieved.

---

## 2. Version Numbering

Versions are single integers: 1, 2, 3, and so on. Every update increments the version by one.

Progression example: `_v1` → `_v2` → `_v3` → `_v4`

When a new version is delivered, the user uploads it to the project and deletes the previous version.

---

## 3. Mandatory Frontmatter

Every file must open with a YAML frontmatter block delimited by `---`. All fields are required unless noted otherwise.

```yaml
---
type: [guidelines | context | reference]
topic: "<short topic label>"
version: "<integer as quoted string>"
guidelines_version: "<version of guidelines active when this file was last written or updated>"
created: YYYY-MM-DD
last_updated: YYYY-MM-DD
shelf_life: "<plain-language estimate, reference files only>"
---
```

Field notes:
- **type:** Use `guidelines` for this file only. Use `context` for project-specific working files (decisions, rationale, constraints, current state). Use `reference` for backing material (research reports, deep research results, comprehensive guides). See §4 for full definitions.
- **topic:** A human-readable label for the file's subject (e.g., "Authentication System", "API Design").
- **version:** Always quote this value (e.g., `"3"`, not `3`) to prevent YAML type coercion.
- **guidelines_version:** The version of the guidelines file that was active when this file was last written or updated. Omit this field in the guidelines file itself.
- **shelf_life:** Required for reference files, omit for other types. A plain-language estimate of how long the findings are expected to remain current (e.g., `"3-6 months"`, `"1-2 years"`, `"evergreen"`). Useful for project scaling (§6) — when pruning a large project, reference files with short shelf life that are well past their estimate are obvious candidates for removal or re-research.
- **Dates:** Use the actual date of the session, in ISO 8601 format (YYYY-MM-DD). If any date is unknown, use `unknown` — never approximate or fabricate dates.

The body summary block (§7) serves as the file's primary description for both humans and RAG. See §7 for guidance.

---

## 4. File Categories

Every file in the project falls into one of three categories, indicated by the `type` frontmatter field.

### Guidelines (`type: guidelines`)

This file. Governs how all other files are created, named, versioned, and updated. Only one guidelines file may exist in the project at any time.

### Context (`type: context`)

Project-specific working files containing decisions, rationale, constraints, open questions, current state, and architectural context. Context files record *what the user chose and why* — they are tied to this project's specific situation, goals, and history.

Context files are primary retrieval targets — they should be concise, high-signal, and written in the vocabulary the user naturally uses when asking questions.

### Reference (`type: reference`)

Backing material: research reports, deep research results, comprehensive guides, and other source documents. Reference files are typically longer, denser, and use the researcher's vocabulary rather than the user's. They exist for depth when context files need supporting detail.

Reference files have lower retrieval priority by nature — their length means they get chunked into many pieces, and their vocabulary may not match user queries. This is expected and acceptable. To compensate, reference files should have strong filenames (§1), retrieval-optimized summary blocks (§7), and clear section headings (§7).

Context files should cross-reference their source reference files so the user or AI can drill into the evidence when needed.

### Index Files Are Prohibited

Do not create routing index files — files whose primary purpose is listing other files and their scopes to help the AI navigate the project.

**Rationale:** Routing indexes rely on an assumption that doesn't hold: that Claude will read the index first, then use it to find and read the right file. In practice, Claude's project RAG retrieves text chunks based on query similarity, not by following a navigation hierarchy. An index file is just another document competing for retrieval budget. It either surfaces content RAG would have found anyway (unnecessary), or it gets retrieved instead of the actual file you need (harmful — Claude learns a file exists but doesn't get its content). At project scale, where indexes would be most useful, they consume context window space proportional to the number of files — exactly when context space is most scarce.

Instead, each file should be self-describing through its filename, summary block, and section headings. This distributes the routing signal across the files themselves rather than concentrating it in a single point of failure.

**Note on custom instructions:** Do not use project custom instructions to tell Claude to read an index file at the start of every conversation. This forces the index into the context window of every conversation regardless of whether it's needed.

---

## 5. File Body Structure

The body is free-form markdown. There is no mandatory section template. Structure content however best serves retrieval for the specific topic.

Guiding principle: **optimize for retrieval, not conformity.**

### Scope Note

These guidelines govern file structure and lifecycle. How content is *written* within that structure — tone, clarity, inclusion criteria, rationale placement — is a separate concern. Projects with significant documentation may benefit from a project-specific style guide as an additional context file.

### Non-Binding Recommended Sections

Use when applicable:
- Key decisions and their rationale
- Open questions and next steps
- Constraints
- Caveats and nuances that would be lost without context
- References to related context files (see §6 for cross-reference rules)

Additional recommended sections (lower priority — include when the project would benefit, omit when it wouldn't):
- Rejected alternatives and their rationale — what was considered and ruled out, and why. Prevents future AI assistants from re-proposing dead ends. Distinct from "key decisions" because decisions naturally get documented while rejections often don't.
- Notes for AI assistants — behavioral instructions scoped to the document's topic, describing how the AI should use or interpret this file's content. Useful when files live in AI-facing project contexts. Not for general personality or interaction preferences, which belong in a separate file.
- Current state summary — a brief snapshot of what's true right now. Useful for files that evolve heavily over time, where the current situation might otherwise need to be reconstructed from a sequence of past decisions. Not needed for files already structured around current state.

---

## 6. File and Project Scope

### One Theme Per File

Each context file must cover a single coherent topic or workstream. If a session produces content that doesn't fit the scope of any existing file, create a new file — do not expand an existing file's scope to absorb it.

Heuristic: if someone would not naturally look for both pieces of information in the same place, they belong in separate files.

### Cross-References Between Files

When referencing another context file, include the version number as it exists at the time of writing (e.g., "See: `api-design_v2.md`").

Do **not** update a cross-reference's version number in later sessions unless the referencing file's content is also being actively reconciled with the referenced file's current state. A stale version in a cross-reference is a useful signal: it indicates that the referencing file predates changes in the referenced file and may need review.

### Size-Based Splitting

Even when a file covers a single coherent theme, it can grow too large for effective retrieval. RAG systems chunk large documents, and individual chunks lose the broader context of the document they came from.

If a file has grown large enough that relevant content is being missed in retrieval, or the AI struggles to locate specific information within it, consider splitting it into sub-topics that each stand alone. A long notes file covering a project with distinct phases or regions could split along those boundaries. A long design decisions file could split by subsystem.

When splitting, each resulting file must have its own frontmatter, its own retrieval-optimized summary block, and enough context to be useful without the sibling files.

### Project Splitting

As projects grow, retrieval quality degrades. More files means more candidate chunks competing for limited retrieval slots per query.

**Signs of degradation:** Claude answering from general training instead of project knowledge, citing wrong or outdated files, giving generic responses that ignore project-specific context, or mentioning that files exist without incorporating their content.

**When to split:** If a project has clearly separable knowledge domains that rarely interact — for instance, a project that has organically grown to be 90% about one subtopic — the dominant domain likely deserves its own project. Split when the cross-domain retrieval noise outweighs the convenience of having everything in one place.

**Remediation short of splitting:** Prune obsolete files, tighten filenames and summary blocks, ensure reference files have strong retrieval metadata, and consolidate scattered content into focused context files.

---

## 7. Retrieval Optimization

The exact retrieval mechanism for Claude Projects is not publicly documented. The following guidance is based on observed behavior and Anthropic's published retrieval research, and optimizes for both keyword and semantic matching.

### Summary Blocks

Every file should open with 2–3 sentences immediately after the frontmatter that describe what the file contains and its key conclusions or decisions. The first sentence should work as a standalone scope label for quick scanning. Use domain-specific terms that match how the user naturally asks about the topic. This is the file's single summary mechanism — there is no separate frontmatter summary field.

This content is the highest-value text for retrieval because it appears at the start of the file and is dense with topical signals.

For reference files, the summary should capture key findings, not just the topic. "Covers database migration options" is less useful than "Comparison of PostgreSQL-to-DynamoDB migration paths: CDC-based replication preferred over batch ETL for tables above 10M rows, estimated 3-week timeline for full cutover."

### Section Headings

Use specific, descriptive headings that mirror how the user would ask about the content. "Configuration" is less retrievable than "Redis Cache Configuration and Eviction Policies." Headings are chunking boundaries — clear headings help RAG extract meaningful, self-contained chunks.

### Vocabulary Alignment

Write using the terms the user would use when asking about the topic, not just the terms the source material uses. If the user says "auth flow" but the documentation says "OAuth 2.0 authorization code grant with PKCE," the file should include both terms — especially in the summary block and headings.

### Note on Query Phrasing

Retrieval is a two-sided match: files need to use the right vocabulary, but queries also need to use the right vocabulary. Anthropic explicitly recommends referencing specific document names, headings, or distinctive terminology in prompts when asking about project knowledge. Well-named files make this easy — the user can say "according to the database migration plan" and RAG matches it to the file.

---

## 8. Changelog

Every file must include a `## Changelog` section as its final section, with entries in reverse chronological order (newest first).

Keep entries concise. Each entry should tell a reader what happened to the file — enough to follow its history without reading the body.

```markdown
## Changelog

### v3 — 2026-03-15
- Added constraint on X
- Clarified rationale for decision Y

### v2 — 2026-03-10
- Expanded scope to cover Z

### v1 — 2026-03-02
- File created
```

---

## 9. Version Mismatch Handling

When working with an existing context file, the AI must:

1. Read the `guidelines_version` field in the context file's frontmatter.
2. Compare it against the `version` field inside the current guidelines file's frontmatter (not the filename — the frontmatter is the authoritative source).
3. If they differ, identify what changed in the guidelines.
4. Update the context file to conform to the current guidelines before applying any session-based updates.
5. Record the guidelines upgrade as a separate entry in the context file's changelog.

### Multiple Guidelines Files Detected

If more than one guidelines file is present in the project, the highest version number is authoritative. The AI must flag this conflict to the user and recommend deleting the outdated guidelines file.

### Multiple Versions of the Same Context File

If more than one version of the same context file is detected (same topic slug, different versions), treat only the highest version as current. Flag the conflict to the user and recommend deleting the outdated version.

---

## 10. Non-Conforming Files

If the AI encounters project files that do not follow these guidelines — missing or malformed frontmatter, wrong naming convention, no `guidelines_version` reference, or structural issues — the AI must:

1. **Not assume how to fix them.** Instead, report the specific issues found to the user.
2. **Present options** for how to resolve each non-conforming file (e.g., reconstruct frontmatter from content, create a new conforming file from scratch, discard the file).
3. **Let the user decide.** The user makes the call on how each non-conforming file is handled.

This applies especially during transitional states where the project is being set up or migrated.

---

## 11. What to Persist

Context files exist to manage the project's context window and make AI assistants within the project more effective. When deciding what to include in a context file, apply this filter:

**Persist:** Decisions, rationale, constraints, open questions, architectural context, and anything a future AI assistant would need to reconstruct the mental model of the topic.

**Do not persist:** Transient debugging steps, the user's emotional tone or personal preferences (unless directly relevant to the topic domain), abandoned approaches (unless the reason for abandonment is itself informative), or meta-discussion about the context system itself.

**Distill from reference files:** When a session draws on reference files to reach decisions, those decisions must be recorded in context files with a cross-reference to the source reference file. Domain findings that informed the decision should be summarized at the point of use in the context file, not left only in the reference material.

**When uncertain:** Ask the user whether a piece of information is worth preserving.

---

## 12. AI Output Contract

At the end of every session, the AI must:

1. **Summarize in chat first** — list each file being created or updated and briefly describe what changed and why, before outputting any file content.

2. **Output each file as a separate artifact** — each file must be its own markdown artifact, titled with the full filename (e.g., `auth-system_v3.md`). This gives the user a download button and copy option for easy upload to the project. Never combine multiple files into a single artifact.

3. **Preserve unchanged content** — when outputting an updated file, take care to preserve all existing content that is not being intentionally changed. Do not reorganize, rephrase, or condense sections that are outside the scope of the current update.

4. **Confirm explicitly when nothing needs updating** — if the session produced nothing worth persisting, state this clearly. Silence is not acceptable.

5. **Justify new file creation** — when creating a new file, briefly explain why no existing file was the right home for the content.

---

## 13. User Responsibilities

- Upload new file versions to the project after receiving them.
- Delete the previous version of any updated file from the project.
- Do not manually rename files without updating the version frontmatter field to match.

---

## 14. Self-Reference

This guidelines file is itself subject to its own rules, with two exceptions:
- The `guidelines_version` frontmatter field is omitted (this file *is* the guidelines).
- Only one file with `type: guidelines` may exist in the project at any time.

All other rules — naming, versioning, changelog, output format — apply to this file.

---

## Changelog

### v7 — 2026-04-21
- Added conciseness guidance to §8 (Changelog).

### v6 — 2026-04-20
- Generalized instance-specific illustrative examples in §1 (Naming for Retrieval) and §6 (Size-Based Splitting) to non-anchoring alternatives. Examples now illustrate the rules without tying them to any one domain or personal context.

### v5 — 2026-04-12
- Added §4 (File Categories): defined three file types — `guidelines`, `context`, and `reference`. Context files hold project-specific decisions and state. Reference files hold backing material. A `knowledge` type (distilled domain expertise) was drafted and removed after adversarial review — see `system-design-decisions_v3.md` Problem 19 for rationale. Added `reference` as a valid frontmatter `type` value. Prohibited routing index files with rationale based on RAG retrieval mechanics research.
- Added `r-` prefix for reference file naming (§1) to visually distinguish them from context files.
- Added `shelf_life` as a required frontmatter field for reference files (§3) — a plain-language estimate of how long findings remain current, useful for project pruning.
- Removed `summary` frontmatter field. The body summary block (§7) is now the sole summary mechanism — it serves the same purpose with more space and better retrieval characteristics.
- Added retrieval optimization guidance: §1 now includes naming for retrieval; new §7 (Retrieval Optimization) covers summary blocks, section headings, vocabulary alignment, and query phrasing. §7 opens with a caveat that exact RAG mechanics are undocumented.
- Inserted new §4 (File Categories) and §7 (Retrieval Optimization). Former §4–§12 renumbered to §5–§14 to accommodate.
- Expanded §6 (was §5): renamed to "File and Project Scope." Added size-based file splitting guidance and project splitting guidance with degradation signals and remediation steps.
- Updated §11 (was §9, What to Persist): added distillation rule — decisions drawn from reference files must be recorded in context files with cross-references.

### v4 — 2026-03-06
- Changed naming convention from `{slug}_V_{version}.md` to `{slug}_v{version}.md` — lowercase v, no underscore between v and version number. Updated all examples throughout.
- Standardized word separators: hyphens between words in topic slugs, underscore only before the version marker. Added explicit naming rules subsection to §1 with rationale for the parsing seam.
- Added content quality scope note to §4: guidelines govern structure and lifecycle; content quality (tone, clarity, writing style) is a separate concern best handled by a project-specific style guide when needed.
- Added unknown dates rule to §3: if any date is unknown, use `unknown` rather than approximating.

### v3 — 2026-03-06
- Added three lower-priority recommended sections to §4: rejected alternatives and their rationale, notes for AI assistants, current state summary. These are non-binding and should be omitted when the project wouldn't benefit.

### v2 — 2026-03-03
- Changed output format from fenced code blocks to separate artifacts for easier download and upload to projects (§10)

### v1 — 2026-03-02
- File created
