# Aevareth Documentation Structure

The original proposal split the project into well over one hundred Markdown files. That creates duplication, stale references, and unnecessary maintenance before production has even started.

Aevareth will use a deliberately small canonical documentation set until the project becomes large enough to justify a split.

## Current canonical structure

```text
aevareth/
├── README.md
├── all_plan.md
├── archectural_plan.md
├── DEVELOPMENT_ROADMAP.md
├── QA_TESTING.md
├── DOCUMENTATION_AUDIT.md
└── plan-structure.md
```

## Responsibilities

### `README.md`

Project entry point and authority order.

Contains:

- project identity
- implementation status
- locked product pillars
- canonical document links
- scope boundaries
- production rule

### `all_plan.md`

Canonical game-design specification.

Contains:

- product vision
- gameplay loop
- nine elements
- world progression
- node exploration
- interactions/story orchestration
- player home
- monster/party systems
- battle rules
- items/capture
- quests/progression
- complete main-story structure
- post-game
- events
- future PvP preparation

### `archectural_plan.md`

Canonical technical architecture.

Contains:

- architecture layers
- stable IDs
- runtime state
- services/bootstrap
- node/map architecture
- Battle World rules
- battle runtime
- story/interaction architecture
- quest/event integration
- save/load
- content validation
- performance
- Unity project structure
- Blender/Unity asset pipeline
- debugging/testing seams

### `DEVELOPMENT_ROADMAP.md`

Dependency-aware production plan.

Contains:

- must/should/optional/future scope
- prototype and vertical-slice sequence
- production phases
- acceptance gates
- content-production order
- release preparation

### `QA_TESTING.md`

Quality strategy.

Contains:

- unit/EditMode testing
- PlayMode/integration testing
- gameplay/regression coverage
- save/load and migration tests
- content validation
- performance testing
- compatibility/release gates
- high-risk system matrix

### `DOCUMENTATION_AUDIT.md`

Record of the README audit.

Contains:

- contradictions found
- missing requirements
- technical risks
- simplifications
- decisions/resolutions
- deferred features

### `plan-structure.md`

This file. It defines documentation ownership and prevents uncontrolled documentation sprawl.

---

## When a document may be split

Split a canonical document only when **at least one** of these is true:

1. It has multiple owners who need to change separate areas independently.
2. A section becomes large enough that readers consistently struggle to find or review it.
3. A production workflow needs a dedicated artifact with its own lifecycle/versioning.
4. A system has enough implementation detail and tests that mixing it into a broader document creates ambiguity.

Do not split simply because a new system exists.

---

## Future splits that may become justified

These are **not required now**.

Possible later documents:

```text
docs/
├── STORY_BIBLE.md
├── CONTENT_DATABASE_GUIDE.md
├── ART_DIRECTION.md
├── AUDIO_DIRECTION.md
├── UI_UX_SPEC.md
├── SAVE_SCHEMA.md
└── LIVE_EVENT_BACKEND.md
```

Create them only when the related work actually begins and the canonical document can link to a single authoritative source.

---

## Unity project documentation

When the Unity project is created, keep implementation-adjacent instructions near the code when possible.

Recommended eventual structure:

```text
Assets/_Aevareth/
├── Runtime/
├── Editor/
├── Tests/
└── Content/
```

Do not create empty directory trees just to match a plan. Create folders alongside the first real implementation that needs them.

---

## Documentation rules

1. Every fact should have one authoritative home.
2. Other files link to that authority instead of duplicating long sections.
3. Stable terminology is mandatory: use **Land**, not Earth.
4. Use stable system/content names consistently with `archectural_plan.md`.
5. Mark assumptions and tuning values as such.
6. Do not describe unimplemented future systems as if they already exist.
7. Keep future PvP architecture preparation separate from actual networking implementation.
8. Update `DOCUMENTATION_AUDIT.md` when a major locked decision intentionally changes.
9. Update `DEVELOPMENT_ROADMAP.md` when phase gates or scope change.
10. A document marked canonical must not contain raw conversational notes such as “add this”, duplicate proposals, or unresolved alternatives.

---

## Authority order

If documents conflict, use this order:

1. `README.md` — project-wide scope and authority
2. `all_plan.md` — gameplay/story intent
3. `archectural_plan.md` — technical implementation boundaries
4. `DEVELOPMENT_ROADMAP.md` — build order
5. `QA_TESTING.md` — verification expectations
6. `DOCUMENTATION_AUDIT.md` — rationale/history, not runtime authority

This structure is intentionally small, practical, and maintainable.