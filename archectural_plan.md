# Aevareth — Canonical Technical Architecture & Production Foundation

**Status:** Architecture baseline v1  
**Engine direction:** Unity, C#  
**Architecture style:** Modular, data-driven, event-driven where decoupling provides real value  
**Primary rule:** Gameplay authority must not depend on scene presentation objects.

> **Definitions define what exists. Systems define what happens. Runtime state defines what is happening now. Presentation shows the result. Persistent GameState owns progression.**

---

## 1. Architecture goals

The architecture must be:

- simple enough for a small team to understand
- modular enough to test systems independently
- data-driven enough to create content without rewriting gameplay logic
- efficient on mobile-class hardware
- safe for save/load and future content migrations
- prepared for new maps/events
- prepared for a future PvP layer without building networking now

Avoid framework-building for hypothetical requirements. Add abstraction when it creates an actual boundary, test seam, or content-authoring benefit.

---

## 2. High-level architecture

```text
                         AEVARETH
                            |
                     GameBootstrap
                            |
        +-------------------+-------------------+
        |                   |                   |
     GameState           Services            Content
        |                   |                   |
        +-------------------+-------------------+
                            |
                     Gameplay Systems
        +-------------------+-------------------+
        |                   |                   |
    Exploration           Battle            Progression
        |                   |                   |
        +-------------------+-------------------+
                            |
                       Presentation
                            |
                        Save / Load
```

### Dependency direction

```text
Content Definitions
      ↓
Domain / Gameplay Rules
      ↓
Runtime Orchestration
      ↓
Unity Presentation
```

Presentation may observe domain state and play visuals. Domain logic must not require a `GameObject`, animation state, camera, or scene object to calculate gameplay results.

---

## 3. Project layers

### A. Definitions

Immutable authoring/configuration data:

- `ElementDefinition`
- `MonsterDefinition`
- `SkillDefinition`
- `EffectDefinition`
- `StatusDefinition`
- `ItemDefinition`
- `QuestDefinition`
- `NPCDefinition`
- `TrainerDefinition`
- `WorldDefinition`
- `MapDefinition`
- `NodeDefinition`
- `EncounterDefinition`
- `InteractionDefinition`
- `StorySequenceDefinition`
- `BattleWorldDefinition`
- `BattleRulesDefinition`
- `BossDefinition`
- `EventDefinition`

ScriptableObjects are appropriate authoring assets for these definitions.

**Never store mutable player-specific state in ScriptableObjects.**

### B. Persistent runtime state

```text
GameState
├── SaveVersion
├── PlayerState
├── MonsterCollectionState
├── PartyState
├── InventoryState
├── QuestState
├── StoryState
├── WorldProgressState
├── NPCState
├── EventState
└── SettingsState
```

### C. Session/runtime state

Short-lived state that may not be persisted directly:

- current game mode
- loaded map runtime
- movement state
- active interaction sequence
- active battle runtime
- transient UI state
- cached content references

### D. Presentation

- 3D views
- animation
- VFX
- audio
- camera
- UI
- dialogue panels
- scene/environment objects

---

## 4. Stable IDs

Every persistent or cross-system content object uses an immutable stable ID.

Examples:

```text
element.common
element.water
monster.fire.001
skill.water.001
item.potion.health.basic
quest.common.001
world.water
map.water.coral_cave
node.water.coral_cave.001
battleworld.water.coral_cave
npc.marina
trainer.water.marina
boss.water.guardian
```

Rules:

- display names are never persistent keys
- IDs are never recycled after release
- content validation rejects duplicate IDs
- save migrations map removed/replaced IDs explicitly

---

## 5. Core services

Keep services small and responsibility-focused.

```text
GameBootstrap
├── GameStateService
├── ContentService
├── SceneService
├── SaveService
├── AudioService
├── EventBus
└── ClockService
```

Add `LocalizationService` only when localization work begins.

Avoid turning every manager into a global singleton. Bootstrap owns long-lived services and passes dependencies to system roots.

---

## 6. Initialization order

```text
Application start
 -> create bootstrap/services
 -> load configuration/content catalog
 -> validate critical content IDs
 -> initialize save system
 -> load or create GameState
 -> apply save migrations
 -> initialize audio/settings
 -> enter Home or saved exploration checkpoint
```

A failed content/save validation must produce an explicit diagnostic and safe fallback rather than silently starting with corrupted progression.

---

## 7. Game modes

Use one explicit mode/state machine instead of scattered booleans.

```text
Boot
Home
Exploration
StoryDialogue
Cutscene
BattleLoading
Battle
BattleResult
Loading
Paused
Disabled
```

Mode transitions own input gating and high-level UI/camera behavior.

---

## 8. World and map architecture

```text
WorldDefinition
├── ID
├── ElementID
├── MapIDs
├── presentation defaults
└── unlock conditions

MapDefinition
├── ID
├── WorldID
├── Scene/Addressable reference
├── NodeGraphID
├── BattleWorldID
├── encounter references
├── NPC references
├── quest/story references
├── music/lighting/weather profile
└── entry/exit metadata
```

Do not build an entire elemental world as one giant scene. Maps are independently loadable content units.

---

## 9. Locked Battle World rule

> **Every exploration Map has exactly one primary `BattleWorldID`.**

That Battle World may expose presentation variants such as:

- normal
- trainer
- boss
- night
- story
- event

This resolves the contradiction in the old plan where one section required one Battle World per map while later examples created separate battle worlds for wild/trainer/boss encounters in the same map.

### Correct model

```text
Coral Cave Map
   -> BattleWorldID: battleworld.water.coral_cave

BattleWorld water.coral_cave
   ├── Normal profile
   ├── Trainer profile
   └── Boss profile
```

A truly different battle scene is allowed only when content has a real presentation requirement, not merely because the encounter type changed.

---

## 10. Battle World responsibility

Battle World owns presentation/environment only:

- arena geometry
- spawn markers
- battle camera profile
- lighting
- environment/weather presentation
- ambient VFX
- music/ambient audio
- terrain/background

Battle World does **not** own:

- damage rules
- turn order
- capture rules
- inventory
- XP
- quest progression
- story flags
- persistent monster state

---

## 11. Exploration/node architecture

```text
World
 -> Map
 -> NodeGraph
 -> Node
 -> NodeConnection
 -> Movement
 -> NodeResolver
 -> InteractionResolver
```

### Node definition

```text
NodeDefinition
├── ID
├── position/anchor reference
├── connection IDs
├── availability conditions
├── presentation state rules
└── interaction/encounter reference
```

### Connection definition

```text
NodeConnection
├── source node
├── destination node
├── authored path/spline reference
├── movement type
├── duration/speed profile
├── rotation behavior
└── camera profile
```

The graph is authoritative. The avatar cannot navigate arbitrary world positions outside defined movement rules.

---

## 12. Movement state machine

```text
Idle
 -> SelectingNode
 -> Moving
 -> Arrived
 -> ResolvingNode
 -> Interaction
 -> Idle
```

Movement code controls path traversal and arrival only. It does not decide quests, dialogue, encounters, or battle rules.

---

## 13. Condition system

One reusable condition engine is shared by:

- nodes
- interactions
- quests
- story
- encounters
- evolutions
- world unlocks
- events

Base conditions:

- quest active/completed
- story flag
- item owned
- monster captured
- monster/player level
- world/map unlocked
- boss defeated
- time/weather profile
- event active

Support AND/OR/NOT composition.

Avoid an unrestricted “CustomCondition” escape hatch in normal authoring. If a new repeated rule appears, add a named reusable condition type with tests.

---

## 14. Interaction architecture

A node resolves to an `InteractionDefinition`.

```text
InteractionDefinition
├── ID
├── trigger
├── conditions
├── sequence ID
├── completion rules
└── repeat policy
```

Do not encode combined types such as `DialogueThenBattle` as an ever-growing enum. Compose actions in a story/interaction sequence instead.

---

## 15. Story sequence architecture

```text
StorySequence
├── DialogueAction
├── CameraAction
├── CharacterMoveAction
├── AnimationAction
├── VFXAction
├── AudioAction
├── SpawnAction
├── DespawnAction
├── ItemAction
├── SetFlagAction
├── QuestAction
├── StartBattleAction
├── WaitForBattleAction
├── ChoiceAction
├── UnlockAction
└── End
```

This system orchestrates presentation and calls reusable gameplay services. It does not duplicate battle/quest/inventory rules.

---

## 16. Battle entry/return flow

```text
Exploration
 -> Encounter/Story requests battle
 -> build BattleContext
 -> snapshot ExplorationReturnContext
 -> read current Map.BattleWorldID
 -> load Battle World
 -> create BattleRuntime
 -> battle
 -> create BattleResult
 -> unload/release Battle World
 -> restore originating map/node context
 -> apply BattleResult to GameState
 -> resume interaction/story/quest
```

### Exploration return context

```text
ExplorationReturnContext
├── WorldID
├── MapID
├── NodeID
├── node/interaction state token if required
└── camera resume profile if required
```

Do not serialize raw scene object references.

---

## 17. Battle context

```text
BattleContext
├── BattleID / seed
├── BattleType
├── SourceMapID
├── BattleWorldID
├── PresentationVariant
├── PlayerParty snapshot/reference
├── OpponentParty
├── BattleRules
├── CaptureAllowed
├── EscapeAllowed
├── Story metadata
└── Event metadata
```

The random seed allows reproducible debugging and creates a foundation for future deterministic validation/PvP work.

---

## 18. Battle runtime

One authoritative battle runtime handles all battle sources.

```text
BattleRuntime
├── PhaseMachine
├── ActionQueue
├── TurnOrderSystem
├── TargetingSystem
├── DamageSystem
├── EffectProcessor
├── StatusSystem
├── SwitchSystem
├── ItemSystem bridge
├── CaptureSystem
├── BattleAI bridge
└── ResultBuilder
```

Battle sources include wild, trainer, boss, story, legendary, and event encounters.

---

## 19. Battle phases

```text
Initialization
Introduction
TurnStart
PlayerChoice
AIChoice
ActionValidation
ActionOrdering
ActionResolution
EffectResolution
DefeatResolution
ForcedSwitch
TurnEnd
Victory / Defeat / Escape / CaptureConclusion
Conclusion
```

The state machine must make interrupted transitions impossible. A battle cannot resolve rewards twice or accept new actions while a prior action is still resolving.

---

## 20. Battle action model

```text
BattleAction
├── ActionType
├── ActorID
├── TargetIDs
├── Priority
├── Parameters
└── SourceDefinitionID
```

Action types:

- skill
- switch
- item
- capture
- defend
- escape
- special scripted action only when necessary

AI chooses an action; `BattleRuntime` validates and executes it.

---

## 21. Monster architecture

```text
MonsterDefinition
      +
MonsterInstance
      ↓
BattleMonsterState (temporary)
      ↓
MonsterView (presentation)
```

`BattleMonsterState` holds temporary combat values such as:

- current HP
- effective stats
- buffs/debuffs
- statuses
- temporary shields
- turn flags

At battle completion, only intended persistent changes are applied back to `MonsterInstance`/GameState.

---

## 22. Skill/effect architecture

```text
SkillDefinition
├── ID
├── ElementID
├── Power
├── Accuracy
├── Priority
├── TargetRule
├── Effect list
├── restrictions
└── presentation references
```

Base v1 has **no global mana/MP resource**.

### Effect processor

Reusable effects:

- damage
- heal
- buff/debuff
- status apply/remove
- shield
- drain
- cleanse
- forced switch
- capture modifier

Do not put individual skill names into battle logic.

---

## 23. Status architecture

```text
StatusDefinition
├── ID
├── duration rules
├── stacking rules
├── max stacks
├── timing hooks
├── stat modifiers
├── periodic effects
├── immunity/cleanse tags
└── presentation references
```

Status processing occurs at defined battle hooks only, not arbitrary Unity `Update()` calls.

---

## 24. Item/inventory architecture

```text
ItemDefinition
├── ID
├── Category
├── StackLimit
├── BuyPrice
├── SellPrice
├── TargetRule
├── Effects
├── restrictions
└── presentation
```

Inventory state stores stable item IDs and quantities.

Mutating inventory is transactional from the gameplay perspective: validate first, apply once, publish result once.

---

## 25. Capture architecture

```text
Capture request
 -> verify BattleRules/CaptureAllowed
 -> verify target ownership/type
 -> consume orb when attempt commits
 -> calculate probability
 -> seeded RNG roll
 -> success/failure presentation event
```

On success:

```text
Create MonsterInstance
 -> assign unique instance ID
 -> initialize persistent growth state
 -> add to party or storage
 -> create BattleResult capture entry
 -> publish MonsterCaptured after GameState commit
```

Trainer monsters are not capturable.

---

## 26. Quest architecture

```text
QuestDefinition
      +
QuestRuntimeState
      ↓
Objective evaluators
      ↓
Completion
      ↓
Rewards / Unlocks
```

Objectives subscribe to domain events such as:

- battle completed
- monster captured
- item acquired
- node reached
- NPC interaction completed
- boss defeated
- story flag changed

The battle system never checks quest IDs directly.

---

## 27. Event/message architecture

Use an in-process event bus for cross-system notifications where direct dependencies would be wrong.

Examples:

```text
BattleCompleted
MonsterCaptured
ItemAdded
QuestCompleted
StoryFlagChanged
MapUnlocked
BossDefeated
```

Rules:

- publish immutable event payloads
- no hidden critical ordering between unrelated subscribers
- do not use the event bus when a direct function return is clearer
- persistent state changes complete before publishing “completed” events

---

## 28. Save/load architecture

Save **state**, never scenes.

### Save envelope

```text
SaveEnvelope
├── SchemaVersion
├── BuildVersion
├── SaveID
├── TimestampUTC
├── Checksum
└── GameState DTO
```

### Required saved state

- player
- monster collection
- party
- inventory/currency
- quests/objectives
- story flags
- world/map unlocks
- current safe exploration checkpoint (WorldID/MapID/NodeID)
- NPC state only when persistent
- event state only when persistent
- settings stored separately if appropriate

### Save rules

- version every schema
- validate before replacing the previous good save
- write to a temporary file then atomically replace where platform APIs allow
- keep one automatic backup of the previous valid save
- never save references to Unity scene objects
- migrations are explicit from version N to N+1
- unknown removed content IDs use migration/fallback rules; never silently delete valuable player state

### Battle saving

Base v1 does **not** save mid-battle. Autosave at safe points before/after battles and story transitions.

This avoids a large amount of state-restoration complexity while preserving player progress.

---

## 29. Scene/content loading

### Prototype

Use normal scenes/prefabs first. Do not block the vertical slice on a complete Addressables pipeline.

### Production

Use Unity Addressables for content where on-demand loading and cataloging provide real benefit:

- maps
- Battle Worlds
- monster prefabs
- major VFX/audio groups
- event content

SceneService owns async loading/unloading and exposes clear completion/failure results.

Loading UI must handle:

- requested content missing
- dependency load failure
- canceled/invalid transition
- fallback to safe state

---

## 30. Content validation

Before play/build, validators should detect:

- duplicate stable IDs
- missing referenced IDs
- map without BattleWorldID
- node connection to missing node
- unreachable required node where graph analysis can detect it
- quest objective with missing target
- monster skill/evolution reference missing
- battle presentation profile missing required spawn markers
- circular progression dependency where forbidden
- invalid item prices/stack limits
- invalid status duration/stack configuration

Content errors should fail CI/build validation for release branches.

---

## 31. Editor-tool policy

The original architecture proposed a large editor suite too early.

Build custom editor tools only after the vertical slice identifies repeated authoring pain.

Likely high-value order:

1. content database/ID validator
2. node graph visualization/editor
3. map validation inspector
4. quest/story sequence editor if raw ScriptableObject authoring becomes slow
5. encounter/battle-world helpers as needed

Do not build ten custom editors before content production proves the need.

---

## 32. Player home runtime

Home monster simulation uses distance/visibility budgets:

- visible/near: animation + simple movement
- far: reduced update frequency
- off-screen/unloaded: no live simulation

Do not simulate every stored monster as an active GameObject.

---

## 33. AI architecture

Start with simple heuristic profiles:

- Random
- Basic
- Aggressive
- Defensive
- Tactical
- Boss

AI receives observable battle state and returns a proposed `BattleAction`.

It does not mutate battle state directly.

Avoid behavior trees/ML unless normal heuristics become demonstrably insufficient.

---

## 34. Performance strategy

Optimize measured bottlenecks, but design obvious hot paths responsibly.

### Baseline targets

For the production target device class:

- 60 FPS target where practical
- 30 FPS minimum supported gameplay target on lower-tier supported devices
- no unbounded per-frame allocations in battle/exploration loops
- battle/map transitions should show responsive loading feedback immediately
- scenes must release unused large content after transition when safe

### Techniques when justified

- object pooling for frequently spawned VFX/projectiles/UI popups
- LOD for large environments/monsters
- animation culling/LOD
- GPU instancing where compatible
- texture compression/atlasing based on profiling
- baked lighting where appropriate
- limited real-time lights/shadows on mobile
- async scene/addressable loading
- reduced AI/update frequency for off-screen home entities

### Do not do prematurely

- ECS/DOTS conversion
- complex custom memory allocators
- multithreaded job systems without a measured hot path

---

## 35. Update/tick policy

Most gameplay systems should react to actions/events/state transitions, not run every frame.

Frame updates are justified for:

- active movement interpolation
- camera presentation
- animation/presentation timing
- active VFX

Turn systems, quests, inventory, save state, and progression should not poll every frame.

---

## 36. Future event preparation

`EventDefinition` may reference:

- maps
- quests
- encounters
- monsters
- items/rewards
- presentation
- activation policy

Base game supports configuration/build-driven activation.

If future events require trusted schedules or secure rewards, add backend authority later. `ClockService` exists so device time is not permanently coupled to event rules.

---

## 37. Future PvP preparation

Build now only the seams that also improve single-player quality:

- battle commands/actions are explicit data
- battle calculations are separated from presentation
- stable IDs
- seeded RNG source
- BattleContext/BattleResult are serializable domain objects
- validation happens before action execution

Do not implement networking now.

Future PvP will still require a separate design covering server authority, transport, matchmaking, anti-cheat, reconciliation, version compatibility, and disconnect behavior.

---

## 38. Blender -> Unity asset pipeline

Blender owns creation of authored 3D assets and animations. Unity owns runtime state and decides when assets/animations play.

### Character/monster workflow

```text
Concept/reference
 -> model
 -> UV/material prep
 -> rig
 -> authored animation clips
 -> export FBX
 -> import Unity
 -> rig/avatar setup where applicable
 -> Animator/presentation controller
 -> gameplay state drives animation parameters/triggers
```

Typical clips:

- idle
- walk/run/hover/swim as appropriate
- battle idle
- attack/skill-specific motions
- hit reaction
- defeat
- capture reaction where needed
- interaction/emote/cutscene clips only when content requires them

### Asset rules

- consistent scale/orientation/export settings
- documented naming convention
- animation clips are separated/named consistently
- gameplay logic never depends on a specific animation clip length unless an explicit presentation event reports completion
- skill VFX/monster animation/map art remain presentation references in definitions

The same principle applies to environment/map models, props, battle arenas, skills/VFX, NPCs, and other game assets.

---

## 39. Recommended Unity project structure

```text
Assets/
└── _Aevareth/
    ├── Runtime/
    │   ├── Core/
    │   ├── Content/
    │   ├── Exploration/
    │   ├── Interaction/
    │   ├── Battle/
    │   ├── Monsters/
    │   ├── Skills/
    │   ├── Items/
    │   ├── Quests/
    │   ├── Story/
    │   ├── World/
    │   ├── Progression/
    │   ├── Save/
    │   └── UI/
    ├── Editor/
    ├── Tests/
    │   ├── EditMode/
    │   └── PlayMode/
    └── Content/
        ├── Definitions/
        ├── Prefabs/
        ├── Models/
        ├── Animations/
        ├── Materials/
        ├── VFX/
        ├── Audio/
        ├── UI/
        └── Scenes/
```

Create folders when implementation reaches them; do not scaffold hundreds of empty folders.

---

## 40. Error handling

Expected failures must be explicit results, not null-reference cascades.

High-risk transitions:

- content ID missing
- map/Battle World load failure
- invalid battle request
- no valid battle target/action
- save parse/checksum/migration failure
- interaction references removed content
- quest progression references invalid target

For player-facing failures:

1. log structured diagnostic context
2. prevent duplicate state mutation
3. return to the last safe mode/checkpoint where possible
4. never overwrite a known-good save with invalid state

---

## 41. Debugging support

Development builds should provide lightweight tools for:

- show current world/map/node
- show active game mode
- inspect story flags/quests
- force-load a map/Battle World
- start a battle from a known context
- set deterministic RNG seed
- grant test monster/item
- validate all content
- print save schema/version

Debug tools must be excluded or locked out of production-facing flows.

---

## 42. Testing architecture

### EditMode/unit-friendly code

Keep pure calculations and state transitions testable without loading scenes:

- conditions
- damage
- turn order
- effects/status
- capture calculation
- quest objectives
- save migrations
- ID validation

### PlayMode/integration

Test Unity-specific flows:

- tap node -> move -> resolve
- map -> battle world -> battle -> map return
- dialogue -> battle -> dialogue resume
- save/reload checkpoints
- scene/addressable failure recovery

Detailed coverage lives in `QA_TESTING.md`.

---

## 43. Architecture acceptance tests

The foundation is ready for content production when all are true:

- a new monster requires data/assets, not battle-controller edits
- a new skill composes existing effects
- a new map can define nodes and one BattleWorldID without core-code changes
- a new quest uses generic objective types
- story can start and resume around battle
- battle returns to the exact source map/node
- save/load survives application restart and schema migration test
- invalid content is caught before release
- Battle World presentation can change without battle-rule changes
- Common and Water worlds can coexist without world-specific code branches

---

## 44. Production sequence

Use `DEVELOPMENT_ROADMAP.md` as the execution order.

The architectural proof milestone is intentionally tiny:

```text
1 small map
5–10 nodes
1 Battle World
1 NPC/story interaction
1 trainer or wild encounter
2–3 monsters
a few skills/effects
1 item
1 quest
capture
save/load
```

It must prove:

```text
Node -> Interaction -> Battle -> Result -> exact Node resume -> Progress -> Save -> Load
```

Only after this passes should large-scale content production begin.

---

## 45. Final authority rule

> **Core systems are stable infrastructure. Worlds, maps, monsters, skills, quests, story beats, Battle World presentation, and events are replaceable content.**

The architecture should make normal content expansion boring and predictable. If adding a new map, monster, skill, quest, or event routinely requires editing unrelated systems, the boundary is wrong and should be corrected before production scales.