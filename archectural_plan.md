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