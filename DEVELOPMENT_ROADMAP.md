# Aevareth Development Roadmap

**Purpose:** Build the smallest reliable playable game first, prove the architecture, then scale content.

This roadmap is dependency-aware. A phase is complete only when its acceptance gate passes.

---

## 1. Scope priority

### Must-have for first shippable game

- bootstrap/services/game modes
- persistent GameState
- stable content IDs
- node exploration
- map loading and return context
- interaction/story sequences
- monster definitions/instances
- party/storage
- battle runtime
- skill/effect/status foundations
- items/inventory/currency
- Prism Orb capture
- quests/progression/world unlocks
- save/load + migrations
- Common through Dark main story content
- player home hub at practical scope
- UI needed for all core actions
- audio/VFX/animation required for readability and polish
- content validation
- QA/release pipeline

### Should-have

- richer home behaviors
- advanced trainer AI profiles
- optional boss phases
- completion tracking
- useful authoring tools proven necessary by production
- optional legendary/post-game quest chains

### Optional

- complex home social simulation
- advanced weather encounters
- large puzzle catalog
- extensive cosmetic systems

### Future only

- new maps/regions
- limited-time/special events
- PvP networking/matchmaking/ranking

---

# Phase 0 — Project setup and production rules

## Build

- Create Unity project using a supported Unity LTS version selected at implementation time.
- Establish `_Aevareth` runtime/content/test folders only as needed.
- Configure source control ignores and text serialization suitable for Unity collaboration.
- Add formatting/analyzer rules that do not block iteration unnecessarily.
- Establish development, test, and release build configurations.
- Add a simple CI job for compilation/tests/content validation once tests exist.

## Why now

Every later system depends on a stable project foundation and predictable asset serialization.

## Exit gate

- clean project opens without console errors
- empty development build succeeds
- one automated EditMode test runs in CI/local runner
- repository has no generated Library/Temp artifacts committed

---

# Phase 1 — Foundation and state

## Build

- `GameBootstrap`
- dependency ownership for long-lived services
- `GameMode` state machine
- `GameState`
- stable ID conventions
- `ContentService`
- `EventBus`
- `SaveService` interface and empty versioned save envelope
- basic debug overlay/logging hooks

## Key tests

- bootstrap initializes services once
- invalid mode transition is rejected
- duplicate content IDs fail validation
- new game creates deterministic valid initial state
- empty save round-trip succeeds

## Exit gate

A new game can boot into a placeholder Home/Exploration mode, create state, save it, restart, and reload the same state.

---

# Phase 2 — Node exploration prototype

## Build

- one placeholder map
- 5–10 nodes
- node connections
- tap/click selection
- authored movement path
- automatic avatar rotation/movement
- controlled exploration camera
- `NodeResolver`
- reusable condition evaluation
- one empty node, one blocked node, one simple interaction node

## Why now

Node movement is the defining exploration mechanic. It must feel clear before battle/content production grows.

## Key tests

- only connected available nodes can be selected
- movement input is locked while moving
- arrival resolves exactly once
- blocked node becomes available when its condition changes
- rapid repeated taps cannot queue invalid moves

## Exit gate

The player can navigate the small graph repeatedly without getting stuck, double-resolving nodes, or bypassing conditions.

---

# Phase 3 — Interaction and story sequence prototype

## Build

- `InteractionDefinition`
- sequence runner
- dialogue UI
- speaker/name/text flow
- camera focus action
- set-flag action
- quest placeholder action
- start-battle placeholder action
- wait/resume capability

## Key tests

- sequence actions run in order
- user can advance dialogue safely
- repeated input does not skip state mutations
- sequence pauses for battle request and resumes from correct action
- canceled/failed action returns a clear result

## Exit gate

A node can run:

```text
Dialogue -> SetFlag -> Dialogue -> End
```

and can also pause at a placeholder battle action and resume correctly.

---

# Phase 4 — Battle foundation with placeholder content

## Build

- `BattleContext`
- seeded battle RNG provider
- `BattleRuntime` state machine
- one active slot per side
- skill action
- defend
- switch
- escape rule
- target validation
- turn order
- damage calculation
- reusable effect processor
- basic status hook
- victory/defeat
- battle result
- minimal battle UI
- placeholder Battle World

## Content

- 2–3 placeholder monsters
- 3–5 skills
- damage effect
- heal/buff/status example

## Key tests

- action priority beats speed where intended
- seeded tie-breaks are reproducible
- invalid target/action is rejected without mutating state
- defeat triggers required switch or battle end
- battle result is produced once
- no quest/story state is mutated directly by battle runtime

## Exit gate

A deterministic 1v1 battle can be played to victory/defeat using placeholder assets and repeated test seeds.

---

# Phase 5 — Map -> Battle World -> Map vertical slice

## Build

- `MapDefinition.BattleWorldID`
- `BattleWorldResolver`
- Battle World async load/unload
- `ExplorationReturnContext`
- map/node restoration
- battle presentation variant selection
- integration with story sequence pause/resume

## Required proof flow

```text
Exploration map
 -> arrive at encounter node
 -> dialogue or encounter
 -> create BattleContext
 -> load that map's Battle World
 -> battle
 -> create BattleResult
 -> unload Battle World
 -> return to exact map/node
 -> resume sequence
```

## High-risk tests

- battle win returns correctly
- battle defeat returns according to checkpoint rule
- escape returns correctly
- load failure does not corrupt exploration state
- story resumes after battle exactly once
- player cannot trigger two battle transitions simultaneously

## Exit gate

This entire flow is stable over repeated runs. **Do not scale content before this passes.**

---

# Phase 6 — Capture, inventory, party, progression

## Build

- item definitions/inventory
- Prism Marks currency
- Prism Orb items
- capture validation/calculation
- `MonsterInstance` creation
- party limit 6
- storage overflow behavior
- XP
- level-up
- skill learning hooks
- evolution condition hooks
- healing/revive/status-cure effects

## Tests

- trainer monster capture rejected
- successful capture creates unique instance once
- full party sends capture to storage
- orb consumption happens once per committed attempt
- level-up correctly handles multiple levels from one reward
- evolution eligibility is deterministic from persistent state
- inventory never drops below zero or exceeds stack limits

## Exit gate

Wild battle -> capture -> party/storage -> save -> reload preserves the captured monster and inventory accurately.

---

# Phase 7 — Quest and world progression vertical slice

## Build

- quest definitions/runtime state
- generic objective evaluators
- rewards
- world/map unlock conditions
- story flags
- NPC state only where needed
- one complete Common-world mini quest chain

## Example chain

```text
Talk to Professor Arin
 -> reach shrine node
 -> wild battle
 -> capture or defeat objective
 -> return dialogue
 -> trainer battle
 -> quest completion
 -> unlock next map
```

## Tests

- objective events count once
- unrelated events do not progress quest
- reload preserves partial objectives
- reward applies once
- completed quest cannot accidentally restart
- world unlock is derived from valid progression state

## Exit gate

The vertical slice has a beginning, objective progression, battle/capture, story resolution, reward, and map/world unlock.

---

# Phase 8 — Save/load hardening

## Build

- full `SaveEnvelope`
- schema version
- checksum/integrity verification
- temporary-write + replace strategy where supported
- one previous-good backup
- autosave points
- explicit migration pipeline
- corrupted-save fallback UX

## Safe autosave points

- entering/leaving home
- map entry or stable checkpoint
- after finalized battle result
- after major quest/story completion
- manual save if product design includes it

Do not save mid-battle in v1.

## Tests

- forced interruption during write leaves prior valid save usable
- corrupted primary falls back to backup
- old schema migrates to current
- removed/renamed content ID follows migration rule
- save/reload restores exact safe map/node

## Exit gate

Save corruption, migration, and restart tests pass before content production accelerates.

---

# Phase 9 — Content production pipeline

## Build only the tooling production now proves necessary

Priority:

1. ID/content validator
2. node graph visualization/editor if Inspector authoring is too slow
3. map validation tools
4. quest/story editor if sequence authoring becomes error-prone
5. encounter/Battle World helpers as needed

## Asset pipeline

Standardize Blender -> Unity export settings for:

- player/NPCs
- monsters
- animations
- map/environment assets
- battle arenas
- props

Establish naming, scale, pivot, rig, animation-clip, material, and texture rules before mass asset creation.

## Exit gate

A second small map and second Battle World can be authored by following documented data/asset workflows without adding map-specific core code.

---

# Phase 10 — Common world production

Build the first world to near-release quality.

## Include

- final-ish environment direction
- multiple maps
- NPC/trainer content
- starter selection
- core tutorials integrated into story
- first boss/guardian sequence
- polished map unlock
- home loop
- production UI baseline
- representative VFX/audio

## Why Common first

It establishes the reusable production standard for every later world.

## Exit gate

A new player can complete Common from new game through the Water unlock with no developer intervention.

---

# Phase 11 — Water world production and architecture validation

Build Water using the same systems without special-case core logic.

This phase is the most important scalability proof after the vertical slice.

## Validate

- multiple maps under one world
- distinct Battle Worlds per map
- map presentation variants
- different encounter tables
- trainer/story integrations
- world-specific assets/audio
- save/load across world transition

## Exit gate

Water ships internally without adding hard-coded `if WaterWorld` behavior to reusable systems.

---

# Phase 12 — Remaining worlds

Produce sequentially:

```text
Land
Electric
Fire
Ice
Air
Light
Dark
Finale
```

For each world:

1. content brief
2. maps/node graphs
3. Battle Worlds
4. encounters/monsters
5. quests/story sequences
6. trainers/bosses
7. assets/audio/VFX
8. QA content validation
9. performance pass
10. regression pass

Do not finish all art before gameplay/content validation.

---

# Phase 13 — Post-game

Implement only the post-game already defined in the design:

- revisit worlds
- updated NPC states
- unfinished side quests
- optional trainer/guardian rematches
- optional legendary chains
- collection/completion tracking

Avoid adding an unrelated endless-game system late in production.

---

# Phase 14 — Event framework proof

After the base campaign is stable, build **one** small event using existing systems.

Prove:

- event activation policy
- event map/content references
- event quest/reward separation
- expiration/deactivation behavior where applicable
- save compatibility when event content is unavailable

If trusted online schedules/rewards are required, stop and design the backend separately instead of hiding networking inside the offline event framework.

---

# Phase 15 — Optimization

Profile real production content.

Optimize only measured bottlenecks in:

- CPU/frame time
- GPU/frame time
- memory
- scene loads
- shader compilation/stutter
- animation
- VFX
- audio memory
- home simulation
- UI overdraw/layout

Potential tools:

- pooling
- LOD
- animation culling
- Addressables
- GPU instancing
- baked lighting
- texture compression
- reduced update rates

No ECS/DOTS migration without profiling evidence and a contained subsystem benefit.

---

# Phase 16 — Release preparation

## Required

- full regression suite
- save migration test from every supported public schema
- clean install/update tests
- long-session soak tests
- target-device performance checks
- input/UI scaling checks
- localization-safe layouts if localization ships
- crash/error logging policy
- legal/credits/licenses review for third-party assets/packages
- release build validation

## Release gate

No known blocker/critical issue and no known save-corruption/data-loss defect.

---

# Implementation principles

1. Build one complete loop before building large content libraries.
2. Placeholder assets are correct during architecture proof.
3. New systems need a real current use case.
4. Prefer data/configuration over world-specific code branches.
5. Prefer explicit state machines over many booleans.
6. Keep battle rules independent from battle presentation.
7. Never let scenes become the save-data authority.
8. Tests and validators scale with content production.
9. Fix architecture pain after the second real content example proves it, not because a hypothetical future might need it.
10. Future PvP remains a separate future project even though the battle core is designed to be reusable.