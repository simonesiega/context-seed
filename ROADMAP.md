# Roadmap

ContextSeed will be developed in small, independently testable layers. Plans may change as repository evidence and benchmarks improve.

## 1. Repository foundation

- Establish the public repository layout and contribution workflow.
- Define Python 3.11+ development checks.
- Keep the future runtime standard-library-only and located in `skills/contextseed/`.

## 2. Skill contract

- Define the Agent Skill entry point and supported workflow.
- Document local state, configuration, and output contracts.

## 3. Deterministic indexing

- Inventory repository files with stable identities and content hashes.
- Parse supported source structures without storing unnecessary source copies.
- Persist a reproducible local index under `.contextseed/`.

## 4. Bounded retrieval

- Rank task-relevant files, symbols, and relationships deterministically.
- Return evidence-backed context within explicit limits.

## 5. Incremental maintenance

- Update changed files and affected relationships safely.
- Fall back to rebuilding when existing state cannot be trusted.

## 6. Validation and release readiness

- Compare assisted and baseline workflows with reproducible benchmarks.
- Add compatibility validation, packaging, CI, and release automation only after the supported behavior is established.
