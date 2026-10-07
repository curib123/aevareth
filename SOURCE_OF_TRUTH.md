# Aevareth: Monster Realm — Source of Truth

## 1. Purpose

This file is the **highest-authority repository document** for deciding which Aevareth instruction wins when documentation overlaps, conflicts, becomes outdated, or is ambiguous.

Codex and Computer Use must read and obey this file before implementation work.

This file defines **authority and conflict resolution**. It does not replace the detailed gameplay, architecture, structure, or execution documents.

---

## 2. Authority Order

When two repository documents conflict, use this order from highest to lowest authority:

1. **SOURCE_OF_TRUTH.md**
   - Documentation authority hierarchy.
   - Conflict-resolution rules.
   - Mandatory execution/verification rules.
   - Definition of authoritative state.

2. **DEVELOPMENT_WORKFLOW.md**
   - How the game is implemented.
   - Codex responsibilities.
   - Computer Use responsibilities.
   - Unity validation.
   - AI asset production.
   - Testing and Definition of Done.
   - Git/commit workflow.

3. **all_plan.md**
   - Product/gameplay requirements.
   - Core game rules.
   - Intended player experience.
   - Content and feature requirements.
   - Progression, battle, capture, maps, monsters, skills, items, story/content requirements where specified.

4. **archectural_plan.md**
   - Technical architecture.
   - Runtime systems.
   - Data boundaries.
   - Dependencies.
   - Unity implementation architecture.
   - Performance/scalability structure.

5. **plan-structure.md**
   - Repository organization.
   - Folder/file layout.
   - Documentation/project structure.

6. **Other project documentation**
   - Supporting notes, implementation records, milestone notes, QA reports, asset records, and future documents.

7. **Code comments, TODOs, temporary notes, generated suggestions**
   - Lowest authority unless promoted into a higher-authority document.

The README is the **entrypoint**, not an independent competing specification. It must direct agents to this hierarchy.

---

## 3. Direct User Instructions

A new explicit instruction from the project owner/user takes precedence over existing repository documentation for the requested change.

When that happens, Codex must update the appropriate authoritative Markdown document **in the same change or before implementation**, so the repository returns to a consistent source-of-truth state.

Do not leave an important new rule only in chat history.

If the user changes:

- gameplay behavior → update `all_plan.md`
- technical architecture → update `archectural_plan.md`
- repository/project layout → update `plan-structure.md`
- development/validation/asset workflow → update `DEVELOPMENT_WORKFLOW.md`
- authority or conflict rules → update `SOURCE_OF_TRUTH.md`

---

## 4. Mandatory Preflight for Codex

Before changing Aevareth, Codex must:

1. Read `SOURCE_OF_TRUTH.md`.
2. Read `DEVELOPMENT_WORKFLOW.md`.
3. Read the relevant requirement sections in `all_plan.md`.
4. Read the relevant technical sections in `archectural_plan.md`.
5. Read `plan-structure.md` when files/folders/project organization are affected.
6. Inspect the current repository state before editing.
7. Identify the smallest coherent milestone or fix.
8. Check whether the requested work conflicts with an existing higher-authority rule.
9. Keep documentation synchronized with implementation when requirements change.
10. Implement only after the authoritative intent is clear enough to proceed safely.

Do not treat a single document in isolation when another higher-authority document defines the same topic.

---

## 5. Mandatory Preflight for Computer Use

Before Unity Editor validation, Computer Use must use the current repository implementation and follow the relevant validation requirements from `DEVELOPMENT_WORKFLOW.md`.

Computer Use must:

1. Open the correct Aevareth Unity project.
2. Use the pinned Unity version.
3. Allow import/compilation to finish.
4. Check the Unity Console.
5. Validate the exact milestone being tested.
6. Report observed behavior accurately.
7. Never mark a feature verified solely because files exist.
8. Never silently alter gameplay rules to make a test pass.
9. Prefer reporting a defect back to Codex for a source-controlled fix.
10. Ensure required scene/prefab/Inspector changes are reproducible and source-controlled when applicable.

Computer Use verifies implementation; it does not redefine the game specification.

---

## 6. Conflict Resolution Rules

When documentation conflicts:

### Rule A — Higher authority wins

The higher document in Section 2 wins.

### Rule B — Specific beats general within the same authority level

If two instructions exist in the same document and do not directly contradict, the more specific instruction for the affected system wins.

### Rule C — Newer explicit requirement beats older wording only when authority is equal

Do not use recency to override a higher-authority document.

### Rule D — Do not silently merge incompatible requirements

If two requirements cannot both be true:

- follow the higher-authority rule;
- update/remove the stale lower-authority wording when safe;
- record the correction in the same commit when practical.

### Rule E — Preserve established behavior when ambiguity is non-blocking

If wording is incomplete but current behavior is valid and does not violate a higher-authority rule, preserve that behavior rather than inventing a new system.

### Rule F — Choose the simplest compliant implementation

When multiple implementations satisfy all authoritative requirements, choose the solution that is:

- simpler,
- easier to test,
- easier to debug,
- easier to maintain,
- performant enough,
- and less tightly coupled.

Do not overengineer.

---

## 7. No Silent Improvisation Rule

Codex and Computer Use must not silently invent major:

- gameplay systems,
- progression rules,
- monster rules,
- battle rules,
- capture rules,
- currencies,
- monetization,
- networking,
- PvP,
- lore,
- architecture layers,
- dependencies,
- art-direction changes,
- or release requirements

that are not supported by the authoritative documents or a direct user instruction.

Small implementation details may be decided autonomously when they are necessary to complete an authorized feature and do not change product intent.

Examples of acceptable implementation decisions:

- naming a private helper class,
- choosing a simple internal collection type,
- selecting a safe serialization helper,
- creating a small editor validation utility,
- choosing a reasonable prefab hierarchy.

Examples requiring specification alignment first:

- adding stamina,
- changing turn rules,
- adding gacha,
- changing evolution stages,
- changing starter count,
- making the game online-only,
- adding PvP now,
- replacing the node-map design,
- changing core elements.

---

## 8. Documentation Synchronization Rule

Code and documentation must not intentionally drift.

When implementation reveals that a documented rule is impossible, unsafe, contradictory, or obsolete:

1. Identify the authoritative document for that rule.
2. Correct that document.
3. Keep the original product intent when possible.
4. Update affected lower-authority documents if they contain stale conflicting wording.
5. Implement the corrected rule.
6. Validate it.
7. Commit the documentation and implementation together when practical.

Do not keep knowingly obsolete instructions because they were written earlier.

---

## 9. Unity Verification Truth States

Use only these statuses for implemented milestones:

### PLANNED
Specified but not implemented.

### IN PROGRESS
Implementation has started but is incomplete.

### IMPLEMENTED — UNITY VALIDATION PENDING
Source code/data/scenes are implemented, but required Unity Editor/Play Mode validation has not completed.

### VERIFIED IN UNITY
Applicable Unity import/compile, Console checks, and Play Mode/runtime validation have passed.

### BLOCKED
A concrete dependency or defect prevents completion.

A feature must not be called **done**, **complete**, **working**, or **verified** when it is only source-generated and still requires Unity validation.

---

## 10. AI Asset Truth Rules

AI-generated assets follow `DEVELOPMENT_WORKFLOW.md`.

Important authority rules:

- AI generation does not override game design.
- An attractive generated asset is not automatically approved.
- Monster/evolution designs must follow Aevareth's defined rules and style bible.
- Final production assets must be integrated and validated in Unity.
- Generated assets with obvious artifacts, inconsistent style, or unclear rights must not be treated as final production assets.
- Placeholder assets are allowed and preferred when they unblock gameplay implementation.

---

## 11. GitHub Is the Persistent Project Record

The repository is the persistent project record.

Important decisions must be represented in:

- authoritative Markdown,
- source code,
- data,
- scenes/prefabs,
- project settings,
- tests,
- or committed production records.

Chat history alone is not the long-term source of truth.

Every completed milestone should leave the repository in a state another developer or agent can inspect and understand without relying on hidden context.

---

## 12. Required Execution Loop

The authoritative implementation loop is:

```text
Read SOURCE_OF_TRUTH.md
        ↓
Read relevant authoritative requirements
        ↓
Inspect current repository state
        ↓
Choose smallest coherent milestone
        ↓
Codex implements
        ↓
Static / automated validation
        ↓
Computer Use opens Unity
        ↓
Import + compile + Console check
        ↓
Play Mode / visual validation where applicable
        ↓
Defect?
  ↙              ↘
Yes               No
 ↓                 ↓
Codex fixes      Mark VERIFIED IN UNITY
 ↓                 ↓
Retest           Commit / continue
```

If Unity validation is unavailable, use:

`IMPLEMENTED — UNITY VALIDATION PENDING`

and continue only where doing so does not create unsafe dependency assumptions.

---

## 13. Definition of Repository Consistency

The repository is consistent only when:

- higher-authority documents do not contradict lower-authority documents on active requirements;
- implementation does not intentionally contradict authoritative requirements;
- the pinned Unity version matches the project;
- gameplay/content data follows documented rules;
- milestones use truthful verification states;
- AI assets follow the production standard;
- tests/validation match the implemented behavior;
- stale superseded requirements are corrected rather than left ambiguous.

---

## 14. Final Rule

When uncertain, do not optimize for producing the most files or the most code.

Optimize for:

**correct requirement → smallest coherent implementation → reproducible source control → real Unity validation → truthful status → next milestone.**

This authority model must be followed by Codex and Computer Use throughout Aevareth development.
