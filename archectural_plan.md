> **AUTHORITY NOTICE:** This document is governed by [SOURCE_OF_TRUTH.md](./SOURCE_OF_TRUTH.md). It is the authoritative technical-architecture document for Aevareth unless a higher-authority document defines the same topic differently.

Yes. I would update the finalized architecture to make the **Battle World system explicit and locked**.

 The key rule will be:

 > **Each exploration Map has exactly one designated Battle World. Every battle triggered from that Map transitions into that Map’s Battle World, while the battle itself remains controlled by the same universal Battle System.**

 This keeps the visual identity tied to the map without creating a separate battle system or dozens of unnecessary battle scenes.

 # Aevareth — Finalized Technical Architecture & Production Plan

 ## 1\. Final architectural principle

 Aevareth is built as:

 > **A data-driven, modular, event-driven 3D RPG framework where gameplay systems are independent from world content and presentation.**

 The fundamental separation is:

```
DATA
  ↓
SYSTEMS
  ↓
RUNTIME STATE
  ↓
PRESENTATION
```

 - **Data** defines what exists.
- **Systems** define what things do.
- **Runtime State** defines what is currently happening.
- **Presentation** displays the result through 3D, animation, UI, VFX, audio, camera and environments.

 The architecture is considered **locked for production**. Implementation details may still be optimized if profiling or testing proves a change is necessary, but the major system boundaries should not be casually redesigned.

---

 # 2\. Final architecture

```
                         AEVARETH
                            │
                    ┌───────▼────────┐
                    │   GAME BOOT     │
                    └───────┬────────┘
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
      GAME STATE         SERVICES         CONTENT
          │                 │                 │
          │                 │                 ├── Monsters
          │                 │                 ├── Skills
          │                 │                 ├── Effects
          │                 │                 ├── Items
          │                 │                 ├── Quests
          │                 │                 ├── NPCs
          │                 │                 ├── Worlds
          │                 │                 ├── Maps
          │                 │                 ├── Nodes
          │                 │                 ├── Encounters
          │                 │                 ├── Battle Worlds
          │                 │                 ├── Bosses
          │                 │                 └── Events
          │                 │
          └─────────────────┼─────────────────┐
                            │                 │
                            ▼                 ▼
                     GAMEPLAY SYSTEMS    PRESENTATION
                            │                 │
       ┌────────────────────┼─────────────┐   │
       │                    │             │   │
       ▼                    ▼             ▼   ▼
 Exploration             Battle       Progression
       │                    │             │
       ├── Node             ├── Skills    ├── Quest
       ├── Movement         ├── Effects   ├── Story
       ├── Encounter        ├── Capture   ├── Unlock
       ├── NPC              ├── Status    └── Events
       └── Camera           └── Boss
                            │
                            ▼
                       SAVE / LOAD
```

---

 # 3\. Architecture layers

 ## Layer A — Definitions

 Static content:

```
MonsterDefinition
SkillDefinition
EffectDefinition
ItemDefinition
QuestDefinition
NPCDefinition
WorldDefinition
MapDefinition
NodeDefinition
EncounterDefinition
BossDefinition
BattleWorldDefinition
EventDefinition
ElementDefinition
```

 Definitions contain configuration, not mutable player state.

---

 # 4\. Runtime state

```
GameState
├── PlayerState
├── MonsterCollection
├── PartyState
├── InventoryState
├── QuestState
├── StoryState
├── WorldProgressState
├── NPCState
├── EventState
└── SettingsState
```

 Example:

```
MonsterDefinition
        +
MonsterInstance
```

 Definition:

```
Emberling
Fire
Base HP: 50
Base Attack: 20
Prefab
Skills
Evolution
```

 Instance:

```
Instance ID: 8F92...
Level: 17
XP: 2380
Current HP: 82
Skills: [...]
Friendship: 42
Personality: Brave
```

 This separation is locked.

---

 # 5\. Stable IDs

 Every important content object receives an immutable ID.

```
element.fire
element.water

monster.fire.001
monster.water.001

skill.fire.001
skill.water.001

item.potion.001

quest.common.001
quest.water.001

world.common
world.water

map.water.coral_cave

node.water.coral_cave.001

battleworld.water.coral_cave

npc.marina

boss.water.guardian
```

 Display names are never persistent identifiers.

---

 # 6\. ScriptableObjects

 Use ScriptableObjects for static authoring data:

```
MonsterDefinition
SkillDefinition
EffectDefinition
ItemDefinition
QuestDefinition
WorldDefinition
MapDefinition
NodeDefinition
NPCDefinition
EncounterDefinition
BattleWorldDefinition
```

 Never store mutable player-specific state inside these assets.

---

 # 7\. Exploration architecture

 The Aevareth exploration structure is:

```
World
  ↓
Map
  ↓
Node Graph
  ↓
Node
  ↓
Connection
  ↓
Player Movement
  ↓
Node Resolver
```

 The node graph is authoritative.

 The player does not freely roam outside the rules defined by the node graph.

---

 # 8\. World → Map → Battle World relationship

 This is now a **locked architectural rule**.

```
WORLD
  │
  ├── MAP 01
  │     └── BATTLE WORLD 01
  │
  ├── MAP 02
  │     └── BATTLE WORLD 02
  │
  ├── MAP 03
  │     └── BATTLE WORLD 03
  │
  └── MAP 04
        └── BATTLE WORLD 04
```

 Therefore:

 > **One Map = One designated Battle World.**

 A MapDefinition contains a reference to its BattleWorldDefinition:

```
MapDefinition
├── ID
├── WorldID
├── Nodes
├── Connections
├── NPCs
├── Encounters
├── Music
├── Environment
├── Lighting
└── BattleWorldID
```

 Example:

```
map.water.coral_cave
        ↓
battleworld.water.coral_cave
```

---

 # 9\. What happens when a battle starts

 The complete flow is:

```
PLAYER
  ↓
Exploration Map
  ↓
Node
  ↓
Encounter Trigger
  ↓
Battle Request
  ↓
Current Map identified
  ↓
Map.BattleWorldID retrieved
  ↓
Battle Context created
  ↓
Battle World loaded
  ↓
Battle presentation initialized
  ↓
Battle starts
```

 Example:

```
Player is exploring:
Water World
    ↓
Coral Cave Map
    ↓
Wild encounter
    ↓
Battle requested
    ↓
Coral Cave Battle World
    ↓
Battle begins
```

 After the battle:

```
Battle Complete
      ↓
Battle Result
      ↓
Unload Battle World
      ↓
Return to original Map
      ↓
Restore exploration state
      ↓
Resolve result
      ↓
Continue exploration
```

---

 # 10\. Battle World is presentation, not battle logic

 This distinction is extremely important.

 The Battle World does **not** contain the battle rules.

 The Battle System controls:

```
Turn
Action
Damage
Effects
Status
Capture
Victory
Defeat
Rewards
```

 The Battle World controls:

```
Arena
Environment
Lighting
Camera
Spawn Positions
Background
Terrain Presentation
VFX Environment
Music
Battle Atmosphere
```

 Therefore:

```
BATTLE SYSTEM
        +
BATTLE CONTEXT
        +
BATTLE WORLD
        ↓
COMPLETE BATTLE
```

 This prevents the battle engine from becoming dependent on a specific environment.

---

 # 11\. Battle World definition

```
BattleWorldDefinition
├── ID
├── MapID
├── ArenaPrefab
├── PlayerSpawnPoints
├── OpponentSpawnPoints
├── CameraProfile
├── LightingProfile
├── EnvironmentProfile
├── Music
├── AmbientAudio
├── VFXProfile
├── TerrainProfile
├── WeatherProfile
└── SpecialRules
```

 Most maps can have a unique Battle World.

 However, the system allows multiple maps to intentionally reference the same Battle World when appropriate.

 For example:

```
Map: Forest Entrance
Map: Forest Path
Map: Forest Clearing
       ↓
Shared Forest Battle World
```

 But the architectural rule remains:

 > **Every Map has one BattleWorld reference.**

 It does not mean every Battle World must be unique.

---

 # 12\. Battle World variants

 A Map can optionally choose presentation variants through data without changing the battle engine.

 For example:

```
Normal
Boss
Night
Event
Story
```

 The Map still has one primary Battle World definition.

 That Battle World can resolve an appropriate presentation profile based on:

```
BattleType
Time
Weather
StoryState
EventState
BossState
```

 Example:

```
Coral Cave Battle World
        │
        ├── Normal Battle Profile
        ├── Boss Battle Profile
        └── Event Battle Profile
```

 This avoids creating unnecessary battle scenes.

---

 # 13\. Battle architecture

 Battle remains its own game mode:

```
Exploration
    ↓
BattleRequest
    ↓
BattleContext
    ↓
BattleWorldResolver
    ↓
BattleWorld
    ↓
BattleController
    ↓
Battle
    ↓
BattleResult
    ↓
Exploration
```

 Battle doesn't care whether it came from:

 - Wild monster
- Trainer
- Boss
- Quest
- Legendary
- Event
- Story sequence

 The BattleContext defines the rules.

---

 # 14\. Battle Context

```
BattleContext
├── BattleType
├── SourceMapID
├── BattleWorldID
├── PlayerParty
├── OpponentParty
├── Rules
├── CaptureAllowed
├── EscapeAllowed
├── BossRules
├── EventRules
└── StoryRules
```

 The important addition is:

```
SourceMapID
BattleWorldID
```

 This makes the origin of every battle explicit.

---

 # 15\. Battle loading flow

```
BattleRequest
      ↓
Validate Request
      ↓
Read Current Map
      ↓
Get Map.BattleWorldID
      ↓
Load Battle World
      ↓
Spawn Player
      ↓
Spawn Opponent
      ↓
Apply Battle Camera
      ↓
Apply Environment
      ↓
Initialize Battle Controller
      ↓
Start Battle
```

 This means there is no hard-coded logic such as:

```
if WaterWorld
    load WaterBattleScene
```

 Instead:

```
CurrentMap
    ↓
BattleWorldID
    ↓
ContentService
    ↓
Load Battle World
```

---

 # 16\. Battle World return

 The original exploration state must be preserved.

```
ExplorationState
├── WorldID
├── MapID
├── CurrentNodeID
├── PlayerPosition
├── CameraState
├── NodeState
└── EncounterState
```

 When battle ends:

```
BattleResult
      ↓
Restore ExplorationState
      ↓
Return to Map
      ↓
Resolve Node
      ↓
Apply Rewards
      ↓
Continue
```

 The player should return to the correct map/node rather than restarting the map.

---

 # 17\. Node architecture

```
Node
├── ID
├── Position
├── Connections
├── Visual Style
├── Conditions
├── State Rules
└── Content
```

 Connections:

```
NodeConnection
├── Start
├── Destination
├── Path
├── Movement Type
├── Duration
├── Rotation
└── Camera Profile
```

 Supports:

 - Forward
- Backward
- Left
- Right
- Diagonal
- Branches
- Junctions
- Special movement

 without changing the movement engine.

---

 # 18\. Node content

```
NodeContent
├── Empty
├── Encounter
├── NPC
├── Trainer
├── Boss
├── Item
├── Quest
├── Story
├── Shop
├── Portal
├── Exit
├── Puzzle
└── Treasure
```

 Node content remains independent of Battle World loading.

---

 # 19\. Dynamic node states

 A node can change based on game state.

```
Node 042

Quest.Water.005 incomplete
        ↓
Locked
```

 Then:

```
Quest.Water.005 complete
        ↓
NPC Marina
```

 Then:

```
WaterGuardian defeated
        ↓
Portal
```

 Same node.

 Different state.

---

 # 20\. Condition system

 Reusable condition framework:

```
Condition
├── QuestCompleted
├── QuestActive
├── StoryFlag
├── ItemOwned
├── MonsterCaptured
├── MonsterLevel
├── WorldUnlocked
├── BossDefeated
├── PlayerLevel
├── TimeOfDay
├── Weather
├── EventActive
└── Custom
```

 Combination:

```
AND
OR
NOT
```

 The same condition system is used by:

 - Nodes
- Quests
- Encounters
- NPCs
- Story
- World unlocks
- Evolution
- Events
- Battle availability

---

 # 21\. Player movement

```
Idle
 ↓
SelectingNode
 ↓
Moving
 ↓
Arrived
 ↓
ResolvingNode
 ↓
Interaction
 ↓
Idle
```

 Additional game modes:

```
Battle
Cutscene
Disabled
Loading
```

 No uncontrolled collection of booleans.

---

 # 22\. Camera

 Camera remains independent from movement.

```
CameraSystem
├── Exploration
├── NodeSelection
├── Movement
├── Interaction
├── Battle
├── Boss
├── Cutscene
├── Home
└── WorldUnlock
```

 Battle camera profiles are supplied by the Battle World.

---

 # 23\. Skills

```
SkillDefinition
├── ID
├── Element
├── Power
├── Accuracy
├── Cost
├── Target
├── Priority
├── Effects
├── Animation
├── VFX
└── Audio
```

 The Battle System does not contain skill-specific hard-coded logic.

```
Skill
 ↓
Effects
 ↓
Effect Processor
```

---

 # 24\. Effect system

```
Effect
├── Damage
├── Heal
├── Buff
├── Debuff
├── Status
├── Shield
├── Drain
├── Cleanse
├── ModifyStat
├── Switch
├── Capture
└── Custom
```

 Skills, items, abilities, bosses and story events can reuse the same Effect System.

---

 # 25\. Status system

```
StatusDefinition
├── ID
├── Duration
├── Stack Rules
├── Max Stacks
├── Stat Changes
├── Periodic Effects
├── Icon
├── VFX
└── Audio
```

 Supports:

```
Burn
Freeze
Poison
Paralysis
Sleep
Fear
Curse
Corruption
Bleed
Slow
Unbalanced
```

---

 # 26\. Monster architecture

```
MonsterDefinition
        ↓
MonsterFactory
        ↓
MonsterInstance
        ↓
MonsterController
        ↓
MonsterView
```

 Gameplay data never depends on a Unity GameObject.

---

 # 27\. Monster home

 Simulation levels:

```
Full
Reduced
Low
Inactive
```

 Nearby:

```
Full AI
Full animation
Full interaction
```

 Far:

```
Reduced simulation
```

 Off-screen:

```
No active simulation
```

---

 # 28\. Quest architecture

```
QuestDefinition
      ↓
QuestInstance
      ↓
Objectives
      ↓
Conditions
      ↓
Rewards
```

 Generic objectives:

```
TalkToNPC
ReachNode
DefeatMonster
CaptureMonster
CollectItem
UseItem
WinBattle
DefeatBoss
EnterLocation
CompleteQuest
TriggerStory
```

---

 # 29\. Story architecture

```
StorySequence
├── Dialogue
├── Camera
├── Animation
├── MoveNPC
├── Spawn
├── VFX
├── Audio
├── Battle
├── GiveItem
├── SetFlag
├── StartQuest
└── Unlock
```

 A StorySequence can launch a battle using the same BattleRequest system as normal exploration.

---

 # 30\. NPC architecture

```
NPCDefinition
        +
NPCState
        ↓
NPCController
        ↓
NPCView
```

 State:

```
Location
DialogueState
QuestState
StoryState
```

---

 # 31\. World architecture

```
WorldDefinition
├── ID
├── Element
├── Maps
├── Music
├── Lighting
├── Weather
├── NodeStyle
├── EncounterTables
├── NPCs
├── Quests
└── Boss
```

 Then:

```
World
 ├── Map
 │    └── Battle World
 ├── Map
 │    └── Battle World
 ├── Map
 │    └── Battle World
 └── Dungeon
      └── Battle World
```

 Do not make every world one gigantic scene.

---

 # 32\. Scene architecture

 Persistent systems:

```
Bootstrap
    ↓
Persistent Systems
```

 Content:

```
World/Map Scene
Battle World Scene
```

 Large content can be streamed and unloaded as required.

---

 # 33\. Addressables

 Use Addressables for:

```
Monster Prefabs
World Assets
Maps
Battle Worlds
VFX
Audio
Animations
NPCs
Event Content
```

 Battle Worlds are especially suitable for on-demand loading.

---

 # 34\. Event architecture

 Events are first-class content.

```
EventDefinition
├── ID
├── StartTime
├── EndTime
├── Maps
├── BattleWorlds
├── Monsters
├── NPCs
├── Quests
├── Items
├── Rewards
├── Rules
├── Bosses
└── Presentation
```

 Therefore:

```
Main Game
Christmas
Halloween
Summer
Anniversary
Collaboration
```

 all use the same framework.

---

 # 35\. Save architecture

 Save state, never scenes.

```
SaveData
├── Player
├── Monsters
├── Party
├── Inventory
├── Quests
├── StoryFlags
├── WorldProgress
├── NPCStates
├── EventStates
└── Settings
```

 If a player saves before a battle:

```
WorldID
MapID
NodeID
```

 remain the authoritative exploration location.

 Battle runtime state should only be persisted if the design explicitly supports saving during battle.

---

 # 36. Event/message architecture

 Example:

```
BattleCompleted
       ↓
 ┌─────┼─────┬────────┐
 ↓     ↓     ↓        ↓
Quest Story Achievement World
```

 The Battle System does not need to know which quests or achievements exist.

 It reports events.

 Other systems react.

---

 # 37. Services

```
GameBootstrap
├── GameStateService
├── SaveService
├── SceneService
├── ContentService
├── AudioService
├── LocalizationService
├── EventBus
└── Time/Event Service
```

 Avoid turning every system into a global Singleton.

---

 # 38\. Editor tools

 Final editor suite:

```
Aevareth Editor
├── Node Graph Editor
├── World Editor
├── Map Editor
├── Battle World Editor
├── Quest Editor
├── Encounter Editor
├── Monster Editor
├── Skill Editor
├── Item Editor
├── Story Sequence Editor
└── Database Browser
```

 The goal:

 > Designers should be able to create content without modifying gameplay code.

 The Map Editor must expose:

```
Map
 ├── Nodes
 ├── Connections
 ├── Encounters
 ├── NPCs
 ├── Quests
 ├── Environment
 └── Battle World
```

 This makes the Map → Battle World relationship easy to author and verify.

---

 # 39\. Performance architecture

 Use:

```
Addressables
Object Pooling
LOD
Animation LOD
AI LOD
Scene Streaming
GPU Instancing
Texture Compression
Baked Lighting where appropriate
Efficient VFX
Limited Real-Time Lights
Profiling
```

 Battle Worlds should be loaded only when a battle requires them and unloaded afterward when appropriate.

---

 # 40\. ECS/DOTS decision

 Do not prematurely use ECS.

 Initial foundation:

```
C#
+
MonoBehaviour
+
ScriptableObject
+
Services
+
Events
```

 DOTS/ECS is introduced only if real profiling proves that a specific subsystem requires it.

---

 # 41\. Final Unity project structure

```
Assets/
│
├── _Aevareth/
│
│   ├── Runtime/
│   │   ├── Core/
│   │   │   ├── GameBootstrap
│   │   │   ├── GameState
│   │   │   ├── EventBus
│   │   │   ├── GameMode
│   │   │   └── Services
│   │   │
│   │   ├── Exploration/
│   │   │   ├── Nodes
│   │   │   ├── Movement
│   │   │   ├── Encounters
│   │   │   └── Camera
│   │   │
│   │   ├── Battle/
│   │   │   ├── BattleController
│   │   │   ├── BattleContext
│   │   │   ├── BattleWorld
│   │   │   ├── Actions
│   │   │   ├── Effects
│   │   │   ├── Status
│   │   │   └── Capture
│   │   │
│   │   ├── Monsters/
│   │   ├── Skills/
│   │   ├── Items/
│   │   ├── Quests/
│   │   ├── NPC/
│   │   ├── Story/
│   │   ├── World/
│   │   ├── Events/
│   │   ├── Prism/
│   │   ├── Progression/
│   │   ├── Save/
│   │   └── UI/
│   │
│   ├── Editor/
│   │   ├── NodeGraph/
│   │   ├── QuestEditor/
│   │   ├── WorldEditor/
│   │   ├── BattleWorldEditor/
│   │   ├── Database/
│   │   └── Tools/
│   │
│   └── Tests/
│
├── Content/
│   ├── Definitions/
│   ├── Prefabs/
│   ├── Models/
│   ├── Animations/
│   ├── Materials/
│   ├── VFX/
│   ├── Audio/
│   ├── UI/
│   └── Scenes/
│
└── Addressables/
```

---

 # 42\. Final production sequence

 ## Stage 1 — Foundation

```
GameBootstrap
GameState
EventBus
Services
Scene Management
Save Architecture
Game Modes
Stable IDs
Data Architecture
```

 ## Stage 2 — Exploration

```
World
Map
Node Graph
Connections
Player Movement
Camera
Node States
Conditions
Node Resolver
```

 ## Stage 3 — Battle foundation

 Before producing the full monster/skill/VFX library, build:

```
BattleContext
BattleController
BattleWorldResolver
Battle World loading
Battle World return flow
Turn system
Action system
Effect system
Status system
Victory/Defeat
Capture
Battle result
```

 Use placeholder assets.

 ## Stage 4 — Tiny vertical slice

 Build only:

```
1 World
1 Map
5–10 Nodes
1 Battle World
1 NPC
1 Monster
1 Skill
1 Effect
1 Item
1 Quest
1 Battle
1 Capture
1 Save
```

 The battle test must prove:

```
Map
 ↓
Node
 ↓
Encounter
 ↓
Battle Request
 ↓
Map's Battle World
 ↓
Battle
 ↓
Battle Result
 ↓
Return to Same Map/Node
```

 ## Stage 5 — Validate architecture

 Test:

```
Save/load
Scene transitions
Map → Battle World
Battle World → Map
Node states
Quest progression
Battle
Capture
Game state
Performance
Mobile input
```

 ## Stage 6 — Data framework

 Convert the prototype into fully data-driven content.

 ## Stage 7 — Editor tools

 Build:

```
Node Editor
Map Editor
Battle World Editor
Quest Editor
Encounter Editor
Database Tools
```

 ## Stage 8 — First production world

 Build Common World properly.

 ## Stage 9 — Second production world

 Build Water World.

 This validates that multiple Maps and multiple Battle Worlds work correctly.

 ## Stage 10 — Remaining worlds

```
Land
Electric
Fire
Ice
Air
Light
Dark
```

 Each Map receives its appropriate Battle World reference.

 ## Stage 11 — Event framework

 Build one complete event.

 Future events then become primarily content production.

 ## Stage 12 — Optimization

 Profile the real game and optimize measured bottlenecks.

 ## Stage 13 — Final content production

 Now produce the large-scale:

```
Monsters
Skills
Effects
Items
VFX
Animations
Audio
NPCs
Maps
Battle Worlds
Bosses
Quests
Story
Events
```

 ## Stage 14 — Polish

```
VFX
Animation
Audio
Lighting
Cinematics
UI
Camera
Performance
Battle presentation
World presentation
```

---

 # 43\. What should NOT be produced first

 Do not begin with:

```
❌ 500 monsters
❌ 1,000 skills
❌ Complete VFX library
❌ Complete item database
❌ Nine finished worlds
❌ Full story cinematics
❌ Every NPC
❌ Every boss
❌ Final UI art
❌ Final music library
```

 First prove the framework with placeholders.

 The first complete loop is:

```
Tap
 ↓
Move
 ↓
Resolve Node
 ↓
Trigger Encounter
 ↓
Resolve Map's Battle World
 ↓
Load Battle World
 ↓
Battle
 ↓
Battle Result
 ↓
Return to Original Map
 ↓
Progress
 ↓
Save
 ↓
Load
```

---

 # 44\. Final acceptance test

 The architecture is ready for full production when all of these are true:

```
Can I create a new map without changing core code?
                 ↓
Can I assign that map one Battle World without changing battle code?
                 ↓
Can I create a new node without changing core code?
                 ↓
Can I create a new quest without changing core code?
                 ↓
Can I create a new monster without changing battle code?
                 ↓
Can I create a new skill without changing battle code?
                 ↓
Can I create a new item without changing inventory code?
                 ↓
Can I create a new NPC without changing NPC code?
                 ↓
Can I create a new Battle World without changing Battle logic?
                 ↓
Can I create a new world without changing exploration code?
                 ↓
Can I create an event map without changing the main game?
```

 If the answer is yes, the architecture is doing its job.

---

 # 45\. Final locked architecture

```
                         AEVARETH
                            │
                            ▼
                     GAME FOUNDATION
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
         GAME STATE      SERVICES       CONTENT
             │              │              │
             └──────────────┼──────────────┘
                            ▼
                       GAME SYSTEMS
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
     EXPLORATION          BATTLE         PROGRESSION
          │                 │                 │
          ▼                 ▼                 ▼
        WORLD            CONTEXT            QUEST
          │                 │                STORY
        MAP                 │               UNLOCK
          │                 │               EVENTS
        NODE                │
          │                 │
     ENCOUNTER              │
          │                 │
          ▼                 ▼
     BATTLE REQUEST ──→ BATTLE WORLD
                            │
                            ▼
                         BATTLE
                            │
               ┌────────────┼────────────┐
               ▼            ▼            ▼
             SKILL        EFFECT       STATUS
               │            │            │
               └────────────┼────────────┘
                            ▼
                       BATTLE RESULT
                            │
                            ▼
                    RETURN TO MAP/NODE
                            │
                            ▼
                       SAVE / LOAD
```

 # 46\. The definitive Battle World rule

 This is now part of the **final architecture**:

 > **One Map has one primary Battle World reference.**

 When a battle occurs:

```
Current Map
    ↓
Map.BattleWorldID
    ↓
BattleWorldResolver
    ↓
Load Battle World
    ↓
Battle
    ↓
Unload / release Battle World
    ↓
Return to Current Map + Node
```

 The Battle World is therefore **map-specific presentation**, while the Battle System is **global and universal**.

 For example:

```
Common World
│
├── Meadow Map
│    └── Meadow Battle World
│
├── Forest Map
│    └── Forest Battle World
│
└── Ruins Map
     └── Ruins Battle World

Water World
│
├── Coral Cave Map
│    └── Coral Cave Battle World
│
├── Ocean Map
│    └── Ocean Battle World
│
└── Water Temple Map
     └── Water Temple Battle World
```

 A battle in the **Coral Cave Map** therefore never needs to ask, "Which battle environment should I use?" The map already defines it.

 That makes the system deterministic, easy to author, easy to debug, and scalable.

---

 # 47\. Final architectural rule

 The most important rule remains:

 > **Core systems are permanent. Content is replaceable.**

```
Node System        ← permanent
Map System         ← permanent
Battle System      ← permanent
Battle World System← permanent
Monster System     ← permanent
Skill System       ← permanent
Effect System      ← permanent
Quest System       ← permanent
Event System       ← permanent
Save System        ← permanent

Common World       ← content
Water World        ← content
Forest Map         ← content
Coral Cave         ← content
Coral Battle World ← content
Christmas Map      ← content
Emberling          ← content
Fireball           ← content
Marina             ← content
```

 The architecture is now designed so that **Monster, Skill, Effect, VFX, animation, audio, maps, battle environments, bosses, quests and events can all be produced later without rebuilding the foundation**.

 The first production milestone is therefore **not "make the monsters."** It is to prove the complete **Map → Battle World → Battle → Map** pipeline with placeholder content. Once that passes, large-scale content production can safely begin.


 add ths A node does not decide “talk” or “battle” by itself. The node triggers an encounter/story interaction, and the story/encounter definition determines what happens.

So a Stepping Stone node could contain an NPC who is part of the story, and the interaction can be:

Talk only

Talk → Battle

Battle → Talk

Talk → Choice → Battle

Talk → Story sequence → Leave

Wild encounter → Battle immediately

Trainer encounter → Talk → Battle → Victory dialogue

Boss → Story introduction → Battle → Ending sequence

Final interaction flow
PLAYER ARRIVES AT NODE
        │
        ▼
   NODE RESOLVER
        │
        ▼
  What content is here?
        │
 ┌──────┼─────────┬───────────┐
 ▼      ▼         ▼           ▼
NPC   WILD      TRAINER      STORY
 │    MONSTER      │           │
 │       │         │           │
 ▼       ▼         ▼           ▼
Story   Battle   Dialogue    Story
Check   Start    │           Sequence
 │                ▼
 │             Battle?
 │             /    \
 │           YES     NO
 │            │       │
 ▼            ▼       ▼
Dialogue    Battle   Continue
 │            │
 ▼            ▼
Next Story  Result

Story dialogue should be a real system
For example, the player reaches:

Map: Water World
Node: Stepping Stone 07
Content: NPC_Marina
Story Requirement: WaterQuest_03

The node resolver checks the current story state.

Scenario A — NPC is part of the story
Player arrives
      ↓
Marina appears
      ↓
Story Sequence starts
      ↓
MC dialogue
      ↓
Marina dialogue
      ↓
Camera focuses on characters
      ↓
Dialogue continues
      ↓
Marina leaves
      ↓
Quest objective updated

No battle occurs.

Scenario B — Story requires a battle
Player arrives
      ↓
Marina dialogue
      ↓
"Then prove yourself!"
      ↓
Battle starts
      ↓
Battle
      ↓
Victory
      ↓
Marina dialogue
      ↓
Story continues
      ↓
Quest updated

The battle is therefore a step inside the story sequence.

Scenario C — Trainer encounter
This should be separate from a normal NPC conversation.

Player reaches trainer
       ↓
Trainer detects player
       ↓
Trainer introduction
       ↓
"Let's battle!"
       ↓
Battle
       ↓
Victory / Defeat
       ↓
Trainer reaction dialogue
       ↓
Reward

So the trainer's encounter can be:

Dialogue
   ↓
Battle
   ↓
Dialogue

Scenario D — Wild monster
Wild monsters should normally skip dialogue.

Player enters encounter node
        ↓
Wild Encounter Resolver
        ↓
Battle
        ↓
Victory / Capture / Escape
        ↓
Return to exploration

But the system should still allow special wild encounters:

Approach legendary monster
        ↓
Story sequence
        ↓
Legendary appears
        ↓
Dialogue / cinematic
        ↓
Battle

So even a wild monster can become story-controlled when necessary.

The key architecture change
I would update the previous architecture to explicitly introduce:

InteractionDefinition

and:

StorySequence

InteractionDefinition
InteractionDefinition
├── ID
├── Trigger
├── Conditions
├── InteractionType
├── StorySequence
├── BattleContext
├── CompletionRules
└── NextAction

Interaction types could include:

Dialogue
Story
WildEncounter
TrainerEncounter
Battle
DialogueThenBattle
BattleThenDialogue
StoryThenBattle
StoryThenDialogue
Choice
QuestInteraction
Shop
Cutscene

But don't hard-code these as giant if/else chains.

Instead, they should resolve into reusable actions.

Story Sequence
The story system becomes an orchestration layer.

StorySequence
│
├── Dialogue
├── CharacterMove
├── CameraFocus
├── Animation
├── VFX
├── Audio
├── Spawn
├── Despawn
├── GiveItem
├── SetFlag
├── StartQuest
├── CompleteObjective
├── StartBattle
├── Wait
├── Choice
└── End

For example:

SteppingStone_07_Story
│
├── CameraFocus(Marina)
├── Dialogue(MC)
├── Dialogue(Marina)
├── Dialogue(MC)
├── Dialogue(Marina)
├── StartBattle(WaterTrainer)
├── WaitForBattle
├── Dialogue(Marina)
├── SetFlag(WaterTrialStarted)
└── End

That is much cleaner than making the NPC itself contain all the logic.

The screen presentation
Yes, the dialogue should be presented as an actual story dialogue scene on the screen, while the world remains the underlying environment.

For example:

┌─────────────────────────────────────────┐
│                                         │
│              3D GAME WORLD              │
│                                         │
│        MC                    NPC        │
│                                         │
│                                         │
├─────────────────────────────────────────┤
│ MARINA                                  │
│                                         │
│ "You finally made it to the stones."    │
│                                         │
│                         [Tap to continue]│
└─────────────────────────────────────────┘

The dialogue system controls:

Speaker

Portrait, if used

Name

Text

Typing speed

Auto advance

Player input

Camera

Character focus

Animation

Facial expression

Voice/audio

Choices

Story flags

The world doesn't stop existing. The game simply enters a StoryDialogue game mode.

Final game-mode relationship
I would lock this into the architecture:

EXPLORATION
     │
     ▼
NODE RESOLVER
     │
     ▼
INTERACTION
     │
 ┌───┴───────────────────────────┐
 │                               │
 ▼                               ▼
STORY / DIALOGUE             ENCOUNTER
 │                               │
 │                       ┌───────┼────────┐
 │                       │       │        │
 │                     WILD    TRAINER   BOSS
 │                       │       │        │
 │                       ▼       ▼        ▼
 │                    BATTLE  DIALOGUE  STORY
 │                               │        │
 │                               ▼        ▼
 │                             BATTLE   BATTLE
 │
 ▼
STORY ACTIONS
 │
 ├── Dialogue
 ├── Battle
 ├── Choice
 ├── Quest
 ├── Reward
 ├── Flag
 ├── Cutscene
 └── Unlock
 │
 ▼
RETURN TO EXPLORATION

Most important rule
Battle should be a reusable system, not a special type of NPC.

And dialogue should be a reusable story system, not something hard-coded into NPCs.

Therefore the same battle can be launched by:

Wild monster
Trainer
NPC
Boss
Quest
Story
Legendary
Event

And the same dialogue system can be launched by:

NPC
Trainer
Boss
Wild monster
Quest
Story
Event
World entrance
Node

That gives you the flexibility you're describing without creating separate systems for every scenario.

So your Stepping Stone example becomes completely valid:
Stepping Stone Node
        ↓
NPC is present
        ↓
Node Interaction
        ↓
Story condition checked
        ↓
Story Sequence
        ↓
MC ↔ NPC dialogue
        ↓
Does story require battle?
        │
       YES
        ↓
Battle Context created
        ↓
Battle
        ↓
Battle Result
        ↓
Story continues
        ↓
More dialogue
        ↓
Quest/Story state updated
        ↓
Return to node


add this 

Not completely. The previous architecture established the **battle framework**, but it did not fully specify the **battle-world rules and mechanics** you just listed.

 For the plan to truly be finalized, I would add a dedicated **Battle Rules & Battle Runtime Architecture** section. That becomes part of the locked foundation.

 # Aevareth — Final Battle World Architecture

 The battle system should be treated as a complete game mode:

```
EXPLORATION
    ↓
Battle Trigger
    ↓
Battle Context
    ↓
BATTLE WORLD
    ↓
Battle Runtime
    ↓
Battle Result
    ↓
Exploration / Story
```

 The **Battle World** is the actual combat environment, while the **Battle Runtime** controls the rules.

---

 # 1\. Battle World

 Every battle loads into a battle environment appropriate to the battle context.

```
BattleWorld
├── Arena
├── BattleCamera
├── PlayerSide
├── OpponentSide
├── SpawnPoints
├── Lighting
├── Environment
├── BattleUI
├── VFX
├── Audio
└── BattleController
```

 The important distinction is:

 > **The map/world determines where the battle comes from. The Battle World determines where and how the battle is fought.**

 For example:

```
Water World
   ↓
Coral Cave Map
   ↓
Stepping Stone Node
   ↓
Wild Monster Encounter
   ↓
Water Cave Battle World
```

 A different node could produce:

```
Water World
   ↓
Trainer Encounter
   ↓
Water Trainer Battle World
```

 And a boss:

```
Water World
   ↓
Guardian Node
   ↓
Boss Battle
   ↓
Water Guardian Battle World
```

 So you can have **battle-world themes/templates**, without creating a completely separate battle engine for every location.

---

 # 2\. One Battle Engine

 There should be **one authoritative BattleController**.

```
BattleController
├── Battle Initialization
├── Turn/Action Flow
├── Target Selection
├── Damage Calculation
├── Status Processing
├── Switching
├── Items
├── Capture
├── Experience
├── Defeat
├── Victory
└── Battle Completion
```

 The battle type changes through `BattleContext`, not by creating separate battle systems.

---

 # 3\. How many monsters can fight?

 This must be an explicit battle rule.

```
BattleRules
├── PlayerActiveSlots
├── OpponentActiveSlots
├── PlayerPartyLimit
├── OpponentPartyLimit
├── ReserveLimit
└── SwitchingRules
```

 For example, the architecture supports:

```
1 vs 1
2 vs 2
3 vs 3
1 vs 2
2 vs 1
Boss vs Party
```

 without rewriting the battle engine.

 The **default Aevareth battle configuration** can be:

```
Player Party: up to 6
Opponent Party: up to 6
Active Monsters: 1
```

 Then special battles can override the rules.

 For example:

```
Normal Wild
1 active vs 1 active

Trainer
1 active vs 1 active

Double Battle
2 active vs 2 active

Boss
1 boss vs 1–3 active player monsters
```

 The exact numbers are therefore **data-driven BattleRules**, not hard-coded into the battle controller.

---

 # 4\. Party system

```
PartyState
├── Slot 1
├── Slot 2
├── Slot 3
├── Slot 4
├── Slot 5
└── Slot 6
```

 Each slot references a `MonsterInstance`.

 The battle creates a runtime copy/reference of the relevant combat state.

```
MonsterInstance
       ↓
BattleMonsterState
```

 This prevents the battle system from directly modifying unrelated presentation objects.

---

 # 5\. Switching

 Switching becomes a proper battle action.

```
SwitchAction
├── SourceMonster
├── DestinationMonster
├── SwitchReason
└── Priority
```

 Possible reasons:

```
PlayerSwitch
ForcedSwitch
MonsterDefeated
AbilityEffect
StoryEffect
BossMechanic
```

 Flow:

```
Player chooses Switch
        ↓
Validate switch
        ↓
Current monster leaves
        ↓
Switch effects
        ↓
New monster enters
        ↓
Entry effects
        ↓
Battle continues
```

 The rules determine whether switching is allowed.

---

 # 6\. Battle actions

 Every combatant operates through an action system.

```
BattleAction
├── Skill
├── Switch
├── Item
├── Capture
├── Escape
├── Defend
└── Special
```

 Each action has:

```
Actor
Target
Priority
Speed
ActionType
Parameters
```

 Then:

```
Choose Actions
      ↓
Validate
      ↓
Determine Order
      ↓
Execute
      ↓
Resolve Effects
      ↓
Check Defeat
      ↓
Check Battle End
      ↓
Next Turn
```

---

 # 7\. Turn order

 Turn order should be its own system.

```
TurnOrderSystem
```

 It considers things such as:

```
Priority
Speed
Move modifiers
Status effects
Battle rules
Forced actions
```

 Example:

```
Monster A
Skill priority: +1
Speed: 80

Monster B
Skill priority: 0
Speed: 150
```

 A's priority may allow it to act first.

 The exact formula belongs to `BattleRules`.

---

 # 8\. Skills during battle

 The skill system previously defined becomes the actual combat pipeline:

```
SkillSelection
      ↓
SkillValidation
      ↓
TargetValidation
      ↓
ActionOrder
      ↓
SkillExecution
      ↓
EffectProcessor
      ↓
Damage/Status/etc.
      ↓
BattleEvents
```

 A skill can produce multiple effects:

```
Fireball
├── Damage
└── Burn
```

 Or:

```
Guardian Blessing
├── Heal
├── Shield
└── Defense Buff
```

---

 # 9\. Damage calculation

 Create a dedicated:

```
DamageSystem
```

 Conceptually:

```
Base Power
      ↓
Attacker Stats
      ↓
Defender Stats
      ↓
Element
      ↓
Type Effectiveness
      ↓
Critical
      ↓
Modifiers
      ↓
Randomization
      ↓
Final Damage
```

 Important:

 **Do not bury the damage formula inside individual skills.**

 The formula belongs to the battle calculation layer.

---

 # 10\. Element/type system

```
ElementSystem
```

 handles:

```
Fire
Water
Earth
Electric
Ice
Air
Light
Dark
...
```

 It can resolve:

```
Weak
Resistant
Immune
Neutral
Super Effective
```

 And special mechanics can be added through data rather than modifying every skill.

---

 # 11\. HP and defeat

 Each battle monster has runtime combat state:

```
BattleMonsterState
├── CurrentHP
├── MaxHP
├── CurrentStats
├── Statuses
├── Buffs
├── Debuffs
├── Cooldowns
├── TemporaryEffects
└── BattleFlags
```

 When:

```
CurrentHP <= 0
```

 the battle engine creates:

```
MonsterDefeated
```

 Then:

```
Forced Switch?
       ↓
YES → Choose next monster
NO  → Continue
```

---

 # 12\. Items

 Items should use the same effect system.

```
ItemDefinition
├── ID
├── ItemType
├── Target
├── Effects
├── Restrictions
└── BattleRules
```

 Battle items could include:

```
Potion
Super Potion
Status Cure
Battle Buff
Revive
Capture Item
Special Quest Item
```

 For example:

```
Potion
   ↓
Target Monster
   ↓
Heal Effect
   ↓
HP Updated
```

 No special potion logic should be hard-coded into BattleController.

---

 # 13\. Prism Orbs

 Yes — **Prism Orbs should be explicitly part of the battle architecture** if they are Aevareth's capture mechanic.

 Create:

```
PrismOrbSystem
```

 Flow:

```
Player chooses Prism Orb
        ↓
Capture validation
        ↓
Calculate capture chance
        ↓
Capture attempt
        ↓
Success?
     /      \
   YES       NO
    │         │
Capture      Monster remains
    │
Battle ends / continues
```

 The capture calculation can consider:

```
Monster species
Monster level
Current HP
Status
Orb type
Battle type
Boss rules
Story rules
Capture modifiers
```

 And importantly:

```
CaptureAllowed
```

 comes from `BattleRules`.

 Therefore:

```
Wild Monster → YES
Trainer Monster → NO
Story Boss → configurable
Legendary → configurable
Event Monster → configurable
```

---

 # 14\. Capture result

 The capture system should not simply spawn a monster.

 It creates a proper `MonsterInstance`.

```
CaptureSuccess
      ↓
Create MonsterInstance
      ↓
Assign InstanceID
      ↓
Generate/retain stats
      ↓
Assign Level
      ↓
Assign XP
      ↓
Assign Skills
      ↓
Assign Personality/etc.
      ↓
Add to Party / Storage
```

 Then:

```
MonsterCaptured
```

 is broadcast through the event system.

 That lets quests, achievements, story and progression react without the capture system knowing about them.

---

 # 15\. Experience

 Experience needs its own system:

```
ExperienceSystem
```

 After battle:

```
BattleResult
      ↓
XP Calculation
      ↓
XP Distribution
      ↓
Monster XP Updated
      ↓
Level-Up Check
```

---

 # 16\. Monster leveling

 Each monster instance contains persistent progression:

```
MonsterProgression
├── Level
├── CurrentXP
├── XPToNextLevel
├── Stats
├── LearnedSkills
├── EvolutionState
└── Other Growth Data
```

 When enough XP is earned:

```
XP >= XP Required
       ↓
Level Up
       ↓
Increase Level
       ↓
Recalculate Stats
       ↓
Check New Skills
       ↓
Check Evolution
       ↓
Story/Quest Events
```

---

 # 17\. Level-up presentation

 The battle system should report the result.

 Then presentation handles:

```
Battle Victory
   ↓
XP Screen
   ↓
Monster gains XP
   ↓
Level Up
   ↓
Stat Increase
   ↓
New Skill?
   ↓
Evolution?
```

 This keeps gameplay and presentation separate.

---

 # 18\. Evolution

 Evolution should also be data-driven.

```
EvolutionDefinition
├── FromMonster
├── ToMonster
├── Conditions
└── Presentation
```

 Conditions can be:

```
Level
Item
Quest
Story Flag
Element
Friendship
Time
Battle
Special Event
```

 The same condition system used by nodes and quests can be reused here.

---

 # 19\. Status effects

 During battle:

```
Burn
Poison
Freeze
Paralysis
Sleep
Fear
Curse
Bleed
Slow
Shield
Buff
Debuff
```

 are all handled by:

```
StatusSystem
```

 At appropriate battle phases:

```
Turn Start
   ↓
Status Processing
   ↓
Actions
   ↓
Turn End
   ↓
Periodic Effects
```

---

 # 20\. Battle phases

 This is important and should be explicitly locked.

```
BattlePhase
│
├── Initialization
├── Introduction
├── PlayerChoice
├── ActionResolution
├── EffectResolution
├── DefeatResolution
├── CaptureResolution
├── Victory
├── Defeat
├── Escape
└── Conclusion
```

 A more detailed turn can be:

```
TURN START
    ↓
Status Start
    ↓
Choose Actions
    ↓
Validate Actions
    ↓
Determine Order
    ↓
Execute Actions
    ↓
Resolve Effects
    ↓
Check KO
    ↓
Forced Switch
    ↓
Status End
    ↓
Check Victory/Defeat
    ↓
NEXT TURN
```

 This gives the battle system a predictable state machine.

---

 # 21\. Escape

 Wild battles can use:

```
EscapeAction
```

 Battle rules decide whether it is allowed.

```
Wild Battle
→ Escape allowed

Trainer Battle
→ Escape disabled

Boss Battle
→ Configurable

Story Battle
→ Usually disabled
```

---

 # 22\. Battle rewards

 After victory:

```
BattleResult
├── Winner
├── Loser
├── XP
├── Items
├── Currency
├── CaptureResult
├── QuestProgress
├── StoryProgress
└── Unlocks
```

 Rewards are processed outside the core action execution.

---

 # 23\. Trainer battles

 Trainer definitions:

```
TrainerDefinition
├── ID
├── Party
├── AIProfile
├── BattleRules
├── IntroDialogue
├── VictoryDialogue
├── DefeatDialogue
├── Rewards
└── StoryIntegration
```

 Example:

```
Player approaches trainer
        ↓
Intro dialogue
        ↓
Battle
        ↓
Victory
        ↓
Victory dialogue
        ↓
Reward
        ↓
Story/Quest update
```

---

 # 24\. AI

 Opponent AI should also be independent.

```
BattleAI
├── Random
├── Basic
├── Defensive
├── Aggressive
├── Tactical
├── Boss
└── Story
```

 AI chooses a `BattleAction`.

 It does **not** execute the action itself.

```
AI
 ↓
Choose Action
 ↓
BattleController
 ↓
Validate
 ↓
Execute
```

 That separation is important.

---

 # 25\. Boss battles

 Bosses get additional rules:

```
BossBattleRules
├── Phases
├── HP Thresholds
├── Special Skills
├── Summons
├── Immunities
├── CaptureAllowed
├── Forced Events
├── Enrage
└── StoryTriggers
```

 Example:

```
100% HP
→ Phase 1

70% HP
→ Phase 2

40% HP
→ Enrage

10% HP
→ Final mechanic
```

 The normal BattleController still handles the combat.

 The boss definition supplies the special rules.

---

 # 26\. Battle + story integration

 This is exactly what you were asking about earlier.

 A story sequence can contain:

```
Dialogue
Dialogue
Camera
Battle
WaitForBattle
Dialogue
Reward
QuestUpdate
```

 Example:

```
Stepping Stone
      ↓
NPC appears
      ↓
MC dialogue
      ↓
NPC dialogue
      ↓
NPC challenges MC
      ↓
Battle starts
      ↓
Battle World loads
      ↓
Battle
      ↓
Victory
      ↓
Battle World closes
      ↓
Return to Story
      ↓
NPC dialogue
      ↓
Quest progresses
```

 So the battle does **not break the story**.

 It is simply another action inside the story sequence.

---

 # 27\. Battle World loading

 The flow should be:

```
Exploration World
       ↓
Battle Request
       ↓
Create BattleContext
       ↓
Save Exploration State
       ↓
Load Battle World
       ↓
Spawn Combatants
       ↓
Initialize Battle
       ↓
Battle
       ↓
Create BattleResult
       ↓
Unload Battle World
       ↓
Restore Exploration
       ↓
Apply BattleResult
       ↓
Continue Story / Quest
```

 This also means the player can return to the **exact exploration node/state** after battle.

---

 # 28\. Battle World should not own progression

 The Battle World should **not** permanently own:

```
Quest progress
Monster collection
Player inventory
Story flags
World unlocks
```

 Instead:

```
Battle World
     ↓
Battle Runtime
     ↓
Battle Result
     ↓
GameState
```

 The persistent game state remains authoritative.

---

 # 29\. Final Battle architecture

 I would now lock this into Aevareth:

```
                     BATTLE REQUEST
                           │
                           ▼
                    BATTLE CONTEXT
                           │
                           ▼
                     BATTLE WORLD
                           │
                    ┌──────┴──────┐
                    │             │
                    ▼             ▼
               PRESENTATION    RUNTIME
                    │             │
                    │       BattleController
                    │             │
                    │       ┌─────┼─────┐
                    │       ▼     ▼     ▼
                    │    Actions Effects Rules
                    │       │     │     │
                    │       └─────┼─────┘
                    │             │
                    │      ┌──────┼────────┐
                    │      ▼      ▼        ▼
                    │   Skills  Items   Switching
                    │      │      │        │
                    │      └──────┼────────┘
                    │             ▼
                    │        Capture
                    │             │
                    │             ▼
                    │       XP / Level
                    │             │
                    └─────────────┼─────────────
                                  ▼
                            BATTLE RESULT
                                  │
                    ┌─────────────┼─────────────┐
                    ▼             ▼             ▼
                 QUEST          STORY       GAME STATE
                    │             │             │
                    └─────────────┼─────────────┘
                                  ▼
                           RETURN TO MAP
```

 ## 30\. What is now covered

 With this addition, the finalized architecture explicitly covers:

 - Battle World loading/unloading
- Battle types
- Party size
- Active monster count
- Switching
- Forced switching
- Turn order
- Skills
- Targets
- Damage
- Elements
- Critical hits/modifiers
- Status effects
- Buffs/debuffs
- Items
- Potions
- Revives
- Prism Orbs
- Capture rules
- Capture calculations
- Monster instances
- XP
- Leveling
- Stat growth
- Skill learning
- Evolution
- Wild battles
- Trainer battles
- Boss battles
- Story battles
- Escape
- AI
- Battle rewards
- Battle phases
- Battle victory/defeat
- Battle → story continuation
- Story → battle → story
- Quest integration
- Event integration
- Save-state integration
- Battle presentation
- Battle camera
- Battle VFX/audio
- Battle UI

 And the key architectural rule remains:

 > **The Battle World is presentation/environment. The Battle Runtime is the authority for combat rules. The persistent GameState is the authority for the player's actual progression.**


Simple workflow
In Blender:

Create/model your character.

Create the character's armature/bones.

Rig the character.

Animate a complete walk cycle.

Make it loop smoothly.

Export the character + armature + walk animation as FBX.

In Unity:

Import the FBX.

Set the character's rig to Humanoid if it's a human character.

Unity detects the walk animation.

Create an Animator Controller.

Add your Blender walk animation.

Connect it to your movement system.

For example:

Player not moving
       ↓
    Idle
       ↓
Player moves
       ↓
    Walk animation
       ↓
Player stops
       ↓
    Idle

The important part is that Unity does not need to create the walking animation. Blender already contains the animation. Unity simply plays the animation according to the player's movement.

You can do this for basically all character animations
Blender:

Idle

Walk

Run

Sprint

Jump

Fall

Attack

Block

Dodge

Hit reaction

Death

Interactions

Emotes

Cutscene animations

Unity:

Decides when to play them.

For example, your Unity movement code can say:

Player speed = 0 → play Idle
Player speed > 0 → play Walk
Player speed > walking threshold → play Run

So Blender handles how the character moves, while Unity handles when and why the animation happens.

For your project, this is a perfectly normal and practical pipeline, and you don't need a complicated animation system just to use Blender-made walking animations.

NOT ONLY IN PERSON ,SKILLS AND MONSTER ANIMATION,MAP DESIGN ,EACH MAP WORDL DESIGN AND OTHER GAME ASSETS