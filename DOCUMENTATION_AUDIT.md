# Aevareth Documentation Audit

**Audit scope:** `all_plan.md`, `archectural_plan.md`, and `plan-structure.md` reviewed as one connected project specification.

**Outcome:** The original design intent is viable, but the documents contained contradictions, duplicated architecture, undefined rules, premature tooling, and excessive documentation fragmentation. The canonical documents in this branch resolve those issues while keeping the core game concept intact.

---

## 1. Executive assessment

### What was strong

- Clear identity: 3D monster-collecting RPG.
- Distinctive controlled node/stepping-stone exploration.
- Strong separation between gameplay systems and content.
- Good instinct to separate immutable definitions from mutable runtime state.
- Good use of stable IDs.
- Correct direction for data-driven monsters, skills, quests, encounters, worlds, and events.
- Correct distinction between Battle World presentation and universal battle logic.
- Good decision to avoid premature ECS/DOTS.
- Good emphasis on proving a small vertical slice before mass content production.
- Strong future extensibility potential for maps/events and later PvP.

### What needed correction

- Core battle/world rules contradicted themselves.
- Several gameplay systems were named but not sufficiently defined.
- Save/load lacked migration, integrity, atomic-write, and failure behavior.
- Quest/content failure and repeatability rules were incomplete.
- Economy/currency was referenced indirectly but undefined.
- Mana potions existed despite no mana stat/resource design.
- Future event timing implied online/live-service authority that did not exist.
- PvP preparation was mixed with systems that should not be implemented yet.
- The proposed documentation tree was far too large for the current project stage.
- Raw conversational notes and repeated “add this” sections made the architecture non-canonical.
- The technical document repeatedly called itself “locked” while still adding major systems afterward.

---

## 2. Major contradiction: Battle World ownership

### Original issue

One section correctly established:

> One Map = one designated Battle World.

Later sections gave examples where the same map could route wild, trainer, and boss encounters into separate Battle Worlds.

### Why this is a problem

This creates uncertainty for:

- authoring
- loading
- debugging
- scene count
- save/return behavior
- map identity

It also risks turning encounter types into scene-selection code.

### Resolution

Locked rule:

> **Every exploration map has exactly one primary Battle World reference.**

That Battle World can expose presentation variants:

- normal
- trainer
- boss
- story
- event
- night/weather where useful

A separate battle scene is allowed only for a genuine presentation requirement.

### Benefit

- deterministic loading
- fewer scenes
- simpler authoring
- shared battle rules
- easier testing

---

## 3. Element naming inconsistency

### Original issue

The game design uses **Land**, while a technical battle section listed **Earth**.

### Risk

Stable IDs, UI labels, content filters, elemental tables, and save data could diverge.

### Resolution

Canonical element is **Land**.

`Earth` is not a separate element and should not appear in production content IDs or UI terminology.

---

## 4. Undefined skill resource vs mana items

### Original issue

The item list included:

- Mana Potion
- Super Mana Potion
- Full Mana Potion

but monsters did not define Mana/MP, and the battle system did not define mana recovery, regeneration, or costs.

### Why this is a problem

Adding mana just to justify three items introduces:

- another stat
- more balancing
- UI requirements
- AI logic
- item logic
- save fields

### Resolution

Base v1 has **no global mana/MP system**.

Mana potions are removed from the base content plan.

Skills use power, accuracy, priority, targets, restrictions, and reusable effects. A skill-resource system can be designed later only if playtesting proves one is needed.

### Benefit

Simpler combat, fewer dependencies, less balancing work.

---

## 5. Party-size ambiguity

### Original issue

The player selects three starting monsters, while the technical plan later suggested a six-monster party and variable active counts.

### Resolution

- Starter selection: **3 monsters**.
- Persistent party limit: **6 monsters**.
- Default active battle format: **1 vs 1**.
- Special multi-active battles may be supported later through `BattleRules`, but they are not required for the first vertical slice.

This is not a contradiction once the three concepts are separated.

---

## 6. Battle runtime incompleteness

### Original issue

The early battle description listed actions but did not fully define:

- turn phases
- validation
- action ordering
- target validation
- forced switching
- duplicate result prevention
- deterministic debugging
- battle result authority

Later appended sections improved this but duplicated and conflicted with earlier material.

### Resolution

The canonical architecture now defines:

- explicit battle state machine
- battle actions
- validation before mutation
- priority + Speed ordering
- seeded RNG
- centralized damage
- effect/status pipeline
- defeat/forced-switch phase
- one final `BattleResult`

### Benefit

Battle can be tested without relying on animations/scenes.

---

## 7. Damage and elemental balancing were underdefined

### Original issue

The documents gave an elemental advantage table but no centralized damage pipeline or multiplier ownership.

### Resolution

- Damage formula is centralized in `DamageSystem`.
- Element multipliers are configuration.
- Initial values are explicitly marked as tuning defaults, not immutable rules.
- Skills do not contain bespoke damage formulas.

### Benefit

Balancing changes do not require editing every skill.

---

## 8. Status/buff/debuff rules were incomplete

### Original issue

Statuses and “maximum three stages” were listed, but duration, stacking, timing, immunity, cleanse behavior, and update phase were not defined.

### Resolution

`StatusDefinition` now owns:

- duration rules
- stack rules
- max stacks
- timing hooks
- modifiers/effects
- immunity/cleanse tags

Default non-stackable unless explicitly configured.

Buff/debuff stat stages remain bounded by a centralized default of -3 to +3.

---

## 9. Capture rules lacked a full authority model

### Original issue

Prism Orb types existed, but the documents did not consistently specify:

- trainer capture prohibition
- boss/story exceptions
- inventory consumption timing
- instance creation
- full-party behavior
- deterministic testing

### Resolution

Canonical flow:

```text
Validate capture
 -> consume orb when attempt commits
 -> calculate chance
 -> seeded roll
 -> success/failure
 -> create exactly one MonsterInstance on success
 -> party or storage
 -> publish result after state commit
```

Trainer monsters are never capturable.

---

## 10. Economy was missing

### Original issue

The plans referenced shops and rewards but no primary currency.

### Why this matters

Without a currency model, shop prices, rewards, selling, progression pacing, and save state cannot be implemented consistently.

### Resolution

Add one base soft currency: **Prism Marks**.

No premium currency or real-money economy is in current scope.

### Benefit

Single-currency economy is the simplest sufficient solution.

---

## 11. Quest state model was incomplete

### Original issue

Quest categories and objective examples existed, but the following were unclear:

- lifecycle states
- repeatability
- failure
- reward idempotency
- save/reload of partial progress

### Resolution

Default lifecycle:

```text
Locked -> Available -> Active -> Completed
```

`Failed` exists only for quests explicitly designed to fail.

Main story quests are non-repeatable.

Rewards must be applied once.

---

## 12. Story system was incorrectly drifting toward NPC-owned logic

### Original issue

Some examples implied an NPC might own dialogue, battle, quest progression, and story behavior directly.

### Why this is risky

NPC scripts would become tightly coupled and hard to reuse.

### Resolution

Introduce explicit reusable orchestration:

- `InteractionDefinition`
- `StorySequence`

NPCs provide identity/presentation/state; the sequence orchestrates dialogue, camera, battle, rewards, flags, and quests.

### Benefit

The same battle/dialogue systems work for NPCs, trainers, bosses, wild legendaries, and story nodes.

---

## 13. Save system was not production-safe

### Original issue

The plan correctly said “save state, never scenes,” but did not define:

- schema version
- migrations
- integrity checks
- interrupted writes
- backup behavior
- removed content IDs
- mid-battle save policy

### Resolution

Add a versioned `SaveEnvelope` with:

- schema version
- build version
- save ID
- timestamp
- checksum/integrity marker
- persistent DTO state

Use temporary-write + safe replace where supported and retain one previous valid backup.

Base v1 does not save mid-battle.

### Benefit

Greatly reduces player-data-loss risk and state-restoration complexity.

---

## 14. Event system implied live-service behavior too early

### Original issue

`EventDefinition` included start/end times and the design described online event maps, but there was no backend, account model, trusted clock, or entitlement authority.

### Risk

Using local device time for limited rewards is trivial to manipulate and creates inconsistent behavior.

### Resolution

Current framework supports:

- bundled/config-driven events
- event maps/quests/content using existing systems

If future events require trusted schedules or secure rewards, add a dedicated backend design then.

`ClockService` prevents permanent coupling to local device time.

---

## 15. Future PvP was at risk of overengineering the current game

### Original issue

The architecture repeatedly referenced future PvP but did not clearly separate useful current foundations from future network systems.

### Resolution

Build now only what is also good single-player architecture:

- stable IDs
- explicit battle commands
- deterministic/seeded RNG source
- calculation/presentation separation
- serializable BattleContext/BattleResult

Do not build:

- networking
- lobby
- matchmaking
- ranking
- anti-cheat
- replication
- server authority

### Benefit

Future PvP remains possible without making the current project a network game.

---

## 16. Custom editor scope was excessive

### Original issue

The original technical plan proposed a large suite of custom editors before proving the game:

- node editor
- world editor
- map editor
- Battle World editor
- quest editor
- encounter editor
- monster editor
- skill editor
- item editor
- story editor
- database browser

### Why this is risky

Editor tooling can consume months before the team knows which authoring workflow is actually painful.

### Resolution

Tool priority after the vertical slice:

1. ID/content validator
2. node graph visualization/editor if necessary
3. map validator
4. quest/story editor only if Inspector authoring becomes inefficient
5. additional helpers only from demonstrated production need

---

## 17. Documentation structure was dramatically over-split

### Original issue

`plan-structure.md` proposed well over 100 Markdown files across design, development, Unity, gameplay, items, worlds, story, monsters, skills, characters, trainers, quests, bosses, encounters, events, technical, and production folders.

### Why this is a problem

At the current project stage this would create:

- duplication
- stale contradictions
- excessive navigation
- unclear authority
- documentation maintenance overhead

### Resolution

Current canonical set is intentionally small:

```text
README.md
all_plan.md
archectural_plan.md
DEVELOPMENT_ROADMAP.md
QA_TESTING.md
DOCUMENTATION_AUDIT.md
plan-structure.md
```

Documents split only when ownership/size/workflow genuinely requires it.

---

## 18. “Locked architecture” language was premature

### Original issue

The architecture called itself locked, then later appended major new systems and conflicting examples.

### Resolution

The current documents use “canonical baseline” and reserve locked language for specific rules that now have explicit rationale, such as:

- Land terminology
- map -> one primary Battle World
- GameState authority
- battle/presentation separation
- no current PvP networking

Architecture may still evolve when profiling/testing proves a real problem.

---

## 19. Raw conversational content reduced maintainability

### Original issue

The architecture contained phrases such as:

- “add this”
- “Not completely”
- repeated proposal blocks
- user/assistant-style explanatory fragments

### Risk

Readers cannot distinguish final decisions from discussion history.

### Resolution

Canonical documents contain only final design/architecture decisions and rationale needed to implement them.

Git history remains the place for earlier discussion evolution.

---

## 20. Asset pipeline needed normalization

### Original issue

The final architecture appended a Blender/Unity explanation focused first on walking animation, then broadly stated that it also applies to skills, monsters, maps, and assets.

### Resolution

The canonical technical plan now defines a concise general Blender -> Unity pipeline:

```text
concept -> model -> rig if needed -> animation if needed -> export -> Unity import -> presentation controller -> gameplay state drives presentation
```

It explicitly covers characters, monsters, maps, battle arenas, props, skills/VFX, and other assets.

Gameplay must not depend on animation clip internals.

---

## 21. Player-home simulation was oversized for the core loop

### Original issue

The home plan included walking, running, sleeping, playing, eating, interacting, following, object reactions, monster reactions, personality, and elemental animations.

### Risk

This can become a simulation project separate from the RPG.

### Resolution

Core home scope:

- display monsters
- simple idle/movement
- basic reaction
- party/storage management

Advanced social simulation is should-have/optional polish.

---

## 22. Performance advice was broad but lacked test ownership

### Original issue

The plan listed many optimization techniques but did not clearly distinguish design safeguards from measured optimization.

### Resolution

Performance rules now say:

- event/action driven systems should not poll every frame
- use async loading for larger content
- profile representative scenes
- use pooling/LOD/culling/compression only where useful
- no ECS/DOTS without evidence

`QA_TESTING.md` defines representative performance scenes and release checks.

---

## 23. Missing error-handling behavior

### Original issue

There was little definition for failure cases such as:

- Battle World load failure
- missing content ID
- corrupted save
- invalid interaction reference
- duplicate result application

### Resolution

The architecture now requires:

1. validate before mutation
2. structured diagnostic logging
3. apply state changes once
4. preserve previous known-good saves
5. return to a safe mode/checkpoint where possible

---

## 24. Missing content validation

### Original issue

A data-driven project was proposed, but no concrete validator contract was specified.

### Resolution

Release validation checks include:

- duplicate/missing IDs
- missing map BattleWorldID
- invalid node connections
- missing quest targets
- invalid monster skill/evolution refs
- invalid item/status configuration
- missing battle spawn/presentation requirements

This becomes increasingly important as content scales.

---

## 25. Story and post-game were incomplete as production specs

### Original issue

The story had a strong world-by-world outline but recurring characters lacked functional motivations/arcs, and post-game was mostly a sequel hook.

### Resolution

The canonical game plan now defines:

- antagonist ideology
- player motivation
- Rival arc
- thematic role for each elemental trainer
- nine-act reveal structure
- post-game world state
- optional rematches
- remaining side quests
- collection/completion goals
- optional legendary chains

The base ending remains complete without requiring future DLC.

---

## 26. Future-scope expansion was too broad

### Original issue

One section suggested future new elements, monsters, regions, skills, items, NPCs, quests, bosses, events, and more, while the README specifically asked to prepare only for future maps, events, and PvP.

### Resolution

Current future-design work is constrained to:

- new maps/regions using existing architecture
- future events
- future PvP preparation

New elements or major new system families require a separate approved design later.

---

## 27. Recommended production assumptions

These are now explicit:

- touch/mouse pointer abstraction
- no free-roam movement
- no manual exploration-camera rotation
- one map -> one primary Battle World
- default 1v1 active battle
- party size 6
- starter selection 3
- no universal mana system in v1
- one soft currency: Prism Marks
- no mid-battle save in v1
- future PvP is not implemented
- online trusted-event timing requires future backend authority

---

## 28. High-risk implementation areas

The following require extra engineering and QA attention:

1. save migrations and interrupted writes
2. battle result idempotency
3. exact exploration-state restoration after battle
4. sequence pause/resume around battle
5. content ID migrations
6. quest reward idempotency
7. capture -> party/storage -> save flow
8. async scene/load failures
9. duplicate event subscriptions after scene cycles
10. node graph progression reachability

These are covered in `QA_TESTING.md` and phase gates in `DEVELOPMENT_ROADMAP.md`.

---

## 29. Final consistency check

The canonical documentation now agrees on these relationships:

```text
World
 -> Map
 -> NodeGraph
 -> Node
 -> Interaction / Encounter
 -> optional StorySequence
 -> optional BattleRequest
 -> Map.BattleWorldID
 -> universal BattleRuntime
 -> BattleResult
 -> persistent GameState
 -> Quest/Story/Progression reactions
 -> exact Map/Node return
```

And:

```text
Definitions = immutable content configuration
Runtime state = current session/combat state
GameState = persistent player authority
Presentation = Unity scenes, views, animation, VFX, audio, UI
```

No system requires a separate battle engine per map/world.

No future networking system is required for the current single-player game.

No giant documentation tree or editor suite is required before the vertical slice.

---

## 30. Final recommendation

Proceed to Unity implementation in the order defined by `DEVELOPMENT_ROADMAP.md`.

The first success milestone is **not** “finish Common World” and not “create many monsters.”

It is:

```text
Small Map
 -> Nodes
 -> Interaction
 -> Battle World
 -> Battle
 -> Result
 -> Exact Node Resume
 -> Quest/Progression
 -> Save
 -> Restart
 -> Correct Load
```

Once that loop is stable and tested, Aevareth has a sound foundation for scaling into the full nine-world game.