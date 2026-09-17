# AGENTS.md

## Purpose

This repository builds **ContextSeed**, a local-first Agent Skill for giving AI coding agents persistent, token-efficient understanding of a codebase.

The core idea is simple: an agent should not have to rediscover the same repository every time a new chat or coding session starts. ContextSeed analyzes a repository locally, stores a compact structural index, and returns only the files, symbols, and relationships relevant to the current task. The real source files remain the source of truth.

The project should stay easy to try:

```text
open repository
→ understand the value immediately
→ install the skill in seconds
→ run it in a real codebase
→ get useful context without a hosted service
```

### Core priorities

1. Keep installation and first use extremely fast.
2. Keep repository analysis local, deterministic, and reproducible.
3. Return bounded, evidence-backed context instead of broad repository dumps.
4. Reuse persistent context across sessions and update it incrementally.
5. Keep the source code authoritative.
6. Prove token-efficiency claims with reproducible benchmarks.

## Agent Context and Skills

This repository uses `.context/` for project context and `.agents/skills/` for reusable agent procedures. Treat both as **lazy-loaded context**.

### `.context/`

- `.context/` contains development notes, decisions, roadmap context, and task-specific background for agents working on this repository.
- Inspect filenames when a task may depend on a prior decision or planned architecture.
- Read only the smallest relevant set; do not load the entire directory by default.
- If current code, tests, or canonical documentation conflict with an older context file, prefer current repository evidence and note the mismatch.
- Do not use `.context/` for generated user indexes.

ContextSeed-generated data inside a user's target repository belongs under `.contextseed/`, not this repository's `.context/`.

### `.agents/skills/`

- Development skills live under `.agents/skills/<skill-name>/`.
- Each development skill should use `SKILL.md` as its entry point.
- If the user names a skill or the task clearly matches one, read that `SKILL.md` before acting.
- Follow referenced files only when needed.

Before editing a nested area, check whether it contains another `AGENTS.md`. If one exists, follow it in addition to this root file.

## Repository Structure

The expected top-level responsibilities are:

```text
skills/contextseed/          → Distributable ContextSeed Agent Skill and its runtime scripts/resources
.agents/skills/              → Development-only skills, such as commit/review helpers
.context/                    → Lazy-loaded project decisions, plans, and development context
benchmarks/                  → Reproducible ContextSeed-vs-baseline experiments
tests/                       → Unit and integration coverage for skill behavior and indexing
scripts/                     → Repository, validation, packaging, and release tooling only
docs/                        → Deeper architecture, format, benchmark, and roadmap documentation
.github/workflows/           → CI and release automation
README.md                    → Public product pitch, installation, quick start, and documentation routing
CHANGELOG.md                 → Released and unreleased user-facing changes
AGENTS.md                    → Repository instructions for coding agents
```

Keep the public skill portable. If code is required at runtime after a user installs the skill, it should normally ship inside the ContextSeed skill directory.

## Product Model

The intended flow is:

```text
new coding session
→ agent loads ContextSeed skill
→ ContextSeed checks or builds the local index
→ task is matched to relevant repository structure
→ a bounded context packet is returned
→ agent reads the referenced source files
→ agent performs the task
→ structural changes refresh only affected context where possible
```

A generated ContextSeed index may contain repository metadata, file identities, symbols, imports, relationships, hashes, and other compact navigation data. Avoid storing unnecessary full source copies.

Context returned to an agent should prefer concrete evidence such as repository-relative paths, symbol names, source ranges, and relationships. Do not present guessed architecture as fact.

## Architectural Invariants

1. **Source is authoritative.** ContextSeed guides discovery; source files decide behavior.
2. **Local by default.** Indexing must not upload repository contents or require a remote service.
3. **Deterministic core.** The same repository state and configuration should produce equivalent structural output.
4. **Bounded context.** Retrieval must respect explicit limits instead of dumping everything relevant-looking.
5. **Persistent context.** Useful repository understanding survives new chats and agent sessions.
6. **Incremental updates.** Prefer updating changed files and affected relationships over rebuilding the whole index.
7. **Evidence over inference.** Paths, symbols, imports, and dependency relationships should support important claims.
8. **Portable skill.** The distributable skill must not silently depend on tools that are absent after installation.
9. **Measured claims.** Token, speed, and tool-call savings must come from reproducible benchmarks.

Do not intentionally weaken these contracts without explicit user direction.

### Retrieval

Retrieval should answer the current task with the smallest useful evidence set.

Prefer:

```text
task
→ likely subsystem
→ relevant files
→ relevant symbols
→ important relationships
→ source inspection
```

over broad summaries or repository-wide dumps.

A token budget is a product constraint, not a cosmetic option. Ranking and truncation must remain deterministic enough to benchmark and debug.

### Incremental indexing

Use stable file identity and content hashing where appropriate. When files change:

- reparse changed or added files;
- remove deleted-file data;
- refresh relationships affected by those changes;
- avoid a full rescan when the same result can be reached safely with bounded incremental work.

Correctness is more important than avoiding a rebuild. Fall back to a safe rebuild when incremental state cannot be trusted.

## README and Installation UX

The README is part of the product.

The first screen should make the value and installation path obvious without requiring readers to study the architecture. Preserve the intended flow:

```text
strong one-sentence value proposition
→ one fast install command
→ one tiny example
→ immediate result
```

Do not bury installation behind long explanations. Keep advanced architecture, formats, benchmarks, and roadmap detail below the quick-start path or in dedicated docs.

Never publish benchmark percentages, performance claims, compatibility claims, or supported-agent claims that have not been verified.

## Benchmarks

Benchmarks exist to determine whether ContextSeed actually reduces repository-discovery cost.

For comparative experiments:

- keep the repository revision, task, model, and relevant agent configuration equivalent;
- compare a baseline without ContextSeed against the ContextSeed-assisted workflow;
- measure input tokens, output tokens where available, tool calls, files inspected, elapsed time, and task success;
- record enough metadata for the experiment to be reproduced;
- do not optimize the benchmark task specifically for ContextSeed;
- do not treat lower token usage as a win if task quality or correctness regresses.

Raw results should remain available behind any summarized README claim.

## Testing and Review

Run the smallest focused tests first, then every repository check affected by the change.

Before handing off a code change:

```bash
git status --short
git diff --check
git diff
```

Inspect untracked files as well as tracked changes. Confirm that:

- behavior is covered by relevant tests;
- installation remains portable;
- no secrets or machine-specific paths were introduced;
- generated runtime data is not accidentally committed;
- documentation matches observable behavior;
- benchmark claims, when changed, are backed by actual results.

Do not describe a check as passed if it was not run.

## Changelog

`CHANGELOG.md` is the canonical history of user-facing changes and follows the same structure used by `codex-limits`.

Under `## [Unreleased]`, use these sections:

```text
### Breaking Changes
### Added
### Changed
### Fixed
### Removed
### Security
```

Rules:

- Add an entry for every user-facing, release-relevant change.
- Put new entries under `## [Unreleased]`; do not edit released version sections.
- Append to an existing subsection rather than creating duplicate headings.
- Do not add entries for purely internal refactors, tests, formatting, or documentation changes unless they materially affect released behavior, release operations, safety, or maintainability.
- Write concise, product-facing release notes describing observable behavior.
- Use past-tense phrasing such as `Added ...`, `Changed ...`, `Fixed ...`, or `Removed ...`.
- Mention implementation details only when they are part of the public surface.
- Link issues or pull requests when useful, and credit external contributors.

## Commit and Change Rules

- Do not commit unless the user explicitly asks.
- Keep each commit scoped to one coherent change.
- Do not use `git add .`; stage only reviewed files.
- Preserve unrelated or pre-existing user changes.
- Use the commit skill under `.agents/skills/` when one exists and the user asks to commit.
- Derive commit subjects from the actual final diff unless the current roadmap/task specifies an exact subject.
- Update `CHANGELOG.md` in the same change when released user behavior changes.
- Do not combine roadmap features early merely because they are adjacent. Build the smallest complete layer required by the current milestone.

## User Override

If user instructions conflict with these repository conventions, follow explicit user direction when the override is intentional and safe.

Do not silently weaken privacy, local-first behavior, evidence requirements, benchmark integrity, or other architectural invariants. Explain the conflict when a requested change would do so.
