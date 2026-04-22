# AI Project Context Management

Every AI conversation starts from zero. You explain your project, your constraints, your preferences — and the next day, you do it again. The longer you work with AI on something complex, the more time you spend re-establishing context and the more prior decisions get forgotten or contradicted.

This system solves that by turning AI chat history into structured, persistent project knowledge. Decisions, rationale, constraints, and open questions are captured in markdown files that live inside an AI project (like Claude Projects). Instead of every conversation starting cold, each one picks up where the last left off — with access to the full history of what was decided and why.

## What's in this repo

A set of markdown files that form a complete context management system:

- **`guidelines_v7.md`** — The master rules file. Governs how all other files are created, named, versioned, and updated. Start here.
- **`context-workflow_v8.md`** — Prompts for capturing context at the end of each session (three cases: routine updates, transitional states, bootstrapping a new project).
- **`qa-workflow_v7.md`** — Two-step QA workflow for verifying files before uploading them to a project.
- **`parallel-workflow_v7.md`** — Prompts for managing multiple concurrent chat sessions without version collisions.
- **`handoff-workflow_v4.md`** — Prompts for transitioning between sequential chats during a work session.
- **`reference-file-workflows_v2.md`** — Six graduated workflows for managing research and reference material, from lightweight optimization to heavy restructuring.
- **`system-design-decisions_v8.md`** — The full decision record: every design choice, what was considered, what was rejected, and why. Useful for understanding the reasoning behind the system's structure.
- **`documentation-principles_v1.md`** — Writing guidance for project files: how to write rationale that stays useful over time.
- **`changelog-cleanup-prompt_v1.md`** — A transitional prompt for trimming verbose changelog entries.

## How it works

1. You create a project in Claude (or another AI platform that supports project-level context).
2. Upload `guidelines_v7.md` to the project.
3. At the end of a working session, paste one of the context workflow prompts. The AI reviews your conversation and produces structured context files.
4. Upload the files to the project. Delete the previous versions.
5. Next session, the AI reads the project files and picks up where you left off.

Context accumulates over sessions. Decisions reference their rationale. New sessions don't re-propose ideas that were already considered and rejected. The AI works *with* your project's history rather than guessing from scratch.

## Who this is for

Anyone who uses AI assistants for ongoing, complex work — not just one-off questions. If you find yourself repeatedly re-explaining your project setup, correcting the same mistakes, or losing decisions made three sessions ago, this system addresses that directly.

The system was designed for Claude Projects but the principles apply to any AI platform that supports persistent project context.

## Getting started

You don't need to adopt the whole system at once. Start with this:

1. Upload `guidelines_v7.md` to your AI project.
2. At the end of your first working session, paste the Case 3 (bootstrap) prompt from `context-workflow_v8.md`.
3. For every session after that, use Case 1 prompt (steady state).

That's it. The AI will review your conversation and produce structured context files that persist your decisions and rationale. Next session, it picks up where you left off.

Add the other workflows as you hit the problems they solve — parallel chats creating conflicts, handoffs losing context mid-day, research reports piling up without structure. Each workflow exists because a specific pain point earned it.

## Background

This system was developed through extensive iteration starting in early 2026. The design decisions file documents 54 problems identified and resolved through critical review, adversarial testing, real-world usage analysis, and external cross-checks. It has been tested across workflows including research integration, multi-chat parallel sessions, and long-running project management.

## License

This work is dedicated to the public domain under [CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/).

Created and maintained by [notewind](https://github.com/notewind).
