# Aevareth: Monster Realm — Canonical Game Design Specification

> **Nine Elements. One Realm. Infinite Legends.**

**Status:** Canonical game-design baseline v1  
**Genre:** 3D monster-collecting RPG  
**Primary interaction:** Touch/mouse pointer; no joystick, WASD, or manual exploration-camera rotation  
**Exploration:** Node/stepping-stone movement  
**Combat:** Turn-based monster battles  
**Primary scope:** Single-player story campaign with future-map/event extensibility and future PvP preparation only

---

## 1. Product vision

Aevareth is a controlled 3D monster adventure in which the player explores handcrafted node graphs, encounters creatures and characters, captures monsters using Prism Orbs, builds a party, completes quests, advances through nine elemental worlds, and uncovers the purpose of the ancient Prism system.

The core design favors clarity and production practicality over free-roam complexity. The player chooses meaningful destinations and actions while the game controls movement paths, camera framing, encounter staging, and cinematic transitions.

### Core pillars

1. **Explore** visually distinct 3D elemental worlds through connected nodes.
2. **Collect** monsters through battle and Prism Orb capture.
3. **Build** a flexible party and learn elemental skills.
4. **Battle** through a readable, data-driven turn system.
5. **Progress** through quests, trainers, bosses, story flags, and world unlocks.
6. **Discover** one connected story about elemental balance and the Prism of Null.
7. **Expand** through new maps and events without rewriting core systems.

---

## 2. Player experience

### Exploration loop

```text
Choose visible connected node
 -> avatar rotates and follows authored path
 -> arrive at node
 -> resolve node state + conditions
 -> trigger interaction, encounter, reward, or nothing
 -> apply result
 -> reveal/enable next choices
```

The player never needs precision movement. Node placement and camera composition must make valid destinations easy to read.

### Complete gameplay loop

```text
Explore
 -> encounter / interact
 -> battle or story action
 -> capture / reward / quest progress
 -> improve party
 -> unlock routes and maps
 -> defeat trainer/boss
 -> complete world objectives
 -> unlock next world
 -> continue story
```

### Failure philosophy

Normal defeat should cost time and positioning, not destroy long-term progress.

Default defeat behavior:

- Battle ends as defeat.
- Consumables already committed remain consumed only if the battle result was finalized.
- Player returns to the last safe checkpoint/home or map entry defined by data.
- Story-required victories remain incomplete.
- No permanent monster loss in the base game.

Special failure rules must be explicit in content data.

---

## 3. Elements

Aevareth has exactly nine base elements:

| Element | Theme | Core relationship |
|---|---|---|
| Common | Balance | Neutral baseline |
| Water | Change | Strong vs Fire; weak vs Electric |
| Land | Strength | Strong vs Electric; weak vs Water |
| Electric | Energy | Strong vs Water; weak vs Land |
| Fire | Creation | Strong vs Ice; weak vs Water |
| Ice | Preservation | Strong vs Air; weak vs Fire |
| Air | Freedom | Strong vs Land; weak vs Electric |
| Light | Hope | Strong vs Dark; also vulnerable to Dark |
| Dark | Fear | Strong vs Light; also vulnerable to Light |

**Terminology rule:** use **Land**, never `Earth`, for this element.

The effectiveness table is configuration data, not hard-coded conditionals. Common defaults to neutral unless a specific skill/effect says otherwise.

---

## 4. World progression

The main story progresses through nine worlds:

```text
1. Common — Meadow of Beginnings
2. Water — Azure Tide
3. Land — Ancient Terra
4. Electric — Stormspire
5. Fire — Emberfall
6. Ice — Frostveil
7. Air — Skyreach
8. Light — Solara
9. Dark — Nocturne Abyss
   -> Prism of Null finale
```

Each world contains multiple maps/areas. A world is not one giant Unity scene.

Each map defines:

- node graph
- map art/environment
- NPCs and trainers
- encounters
- quests/story sequences
- collectibles/items
- exits/portals
- map music/lighting/weather profile
- one primary Battle World reference

World progression is story-gated but completed maps remain revisitable unless a specific story state temporarily prevents access.

---

## 5. Node exploration

### Node definition

A node may contain:

- empty traversal
- NPC interaction
- trainer interaction
- wild encounter
- rare/legendary encounter
- boss
- item/treasure
- quest interaction
- story sequence
- shop
- portal/exit
- puzzle
- secret path

A node itself does not hard-code “talk” or “battle.” It resolves an `InteractionDefinition` or `EncounterDefinition`, which decides what happens.

### Dynamic node states

Node presentation/content may change based on reusable conditions such as:

- quest active/completed
- story flag
- item owned
- monster captured
- boss defeated
- world unlocked
- player/monster level
- time/weather profile
- event active

Use AND/OR/NOT condition composition rather than custom scripts for normal content gating.

---

## 6. Interaction and story orchestration

Interactions are reusable action sequences.

Supported actions include:

- dialogue
- camera focus
- character movement
- animation
- VFX/audio
- spawn/despawn
- give/remove item
- set story flag
- start/advance quest
- start battle
- wait for battle result
- reward
- choice
- unlock node/map/world

Examples:

```text
NPC talk only:
Dialogue -> QuestUpdate -> End

Trainer:
Dialogue -> Battle -> Result -> Dialogue -> Reward

Boss:
StoryIntro -> Battle -> Result -> EndingSequence -> Unlock

Legendary:
Cinematic -> Battle -> Capture/DefeatResult -> StoryFlag
```

Battle is one reusable action inside a sequence, not a special NPC type.

---

## 7. Player home

The home is a hub for:

- party management
- monster collection/storage
- basic monster display
- item storage
- quest board
- map/world selection where appropriate
- shop access where appropriate
- player/settings screens

### Scope control

For the first production version, home monsters need only:

- idle
- walk/hover/swim locomotion as appropriate
- simple wander
- simple player reaction

Complex social simulation, eating, personality-to-personality interactions, advanced schedules, and large autonomous populations are **should-have polish**, not core architecture blockers.

---

## 8. Monster model

### Monster definition data

Every species defines:

- stable ID
- display name
- element
- species/family
- rarity
- base stats
- level-growth profile
- learnable skills
- traits/abilities if used
- capture rate
- evolution paths
- model/prefab reference
- animation/VFX/audio references
- habitat tags

### Monster instance data

Each captured monster owns persistent state:

- unique instance ID
- species definition ID
- level
- XP
- calculated stats
- current HP outside battle as required
- learned/equipped skills
- evolution state
- friendship/personality values only where gameplay uses them
- capture provenance

Presentation objects are never the authority for monster progression.

### Rarity

Recommended content rarity labels:

- Common
- Uncommon
- Rare
- Epic
- Legendary
- Mythic

Rarity affects acquisition/content expectations; it must not automatically determine battle power.

---

## 9. Party and starters

At the beginning of the story, the player selects **three starter monsters**.

Persistent party limit: **6 monsters**.

Default battle active slots: **1 vs 1**.

Battle rules may later support special 2v2 or boss formats through data, but the initial vertical slice and normal campaign battles use 1 active monster per side.

When the party is full, newly captured monsters go to storage unless content explicitly overrides this behavior.

---

## 10. Battle rules

### Player actions

- Skill
- Switch
- Item
- Prism Orb / Capture
- Defend
- Escape when allowed

### Turn flow

```text
Turn start
 -> process start-of-turn statuses
 -> collect player/AI actions
 -> validate actions/targets
 -> determine action order
 -> execute actions
 -> resolve effects
 -> resolve defeats/forced switches
 -> process end-of-turn statuses
 -> check victory/defeat/escape/capture
 -> next turn
```

### Action order

Order is based on:

1. explicit action priority
2. effective Speed
3. deterministic tie-breaker supplied by the battle RNG seed

### Skill resource decision

The original documents listed Mana Potions but did not define a mana stat. To avoid adding an unnecessary resource subsystem, **base v1 has no universal mana/MP resource**.

Skills are constrained by:

- availability/learn rules
- target rules
- accuracy
- priority
- battle/state restrictions
- optional per-skill usage limit only if a specific design later requires it

`Mana Potion`, `Super Mana Potion`, and `Full Mana Potion` are removed from the base item list unless a future approved resource system is introduced.

### Damage baseline

Keep the exact formula centralized and configurable. Initial balancing formula:

```text
Base = SkillPower * (EffectiveAttack / max(1, EffectiveDefense))
LevelScale = 0.75 + (Level * 0.025)
Final = round(Base * LevelScale * ElementMultiplier * CriticalMultiplier * OtherModifiers * Variance)
Minimum damaging hit = 1
Variance = 0.90 .. 1.00
```

Initial multipliers:

- strong: 1.5x
- weak/resisted: 0.67x
- neutral: 1.0x
- critical: configurable, initial 1.5x

These are tunable data values, not immutable design law.

### Buff/debuff stages

Stat-stage range: **-3 to +3** by default.

The stat-stage multiplier table must be centralized and content-validated.

---

## 11. Effects and status

Skills and items use reusable effects rather than custom battle-controller branches.

Core effect families:

- damage
- heal
- buff
- debuff
- status apply/remove
- shield
- drain
- cleanse
- stat modification
- forced switch
- capture modifier

Status definitions may include:

- Burn
- Freeze
- Paralysis
- Poison
- Sleep
- Confusion
- Fear
- Blind
- Curse
- Corruption
- Bleed
- Slow
- Unbalanced

Default status rule: one application does not stack unless its definition explicitly allows stacking. Duration, stack cap, periodic timing, immunity tags, and cleanse category are data.

---

## 12. Items and inventory

### Item categories

- healing
- status recovery
- battle buff
- revive
- Prism Orb
- evolution
- key/story
- quest
- crafting/material only if later justified

### Base healing/recovery set

Keep the first release small:

- Health Potion
- Super Health Potion
- Full Health Potion
- Revival Potion
- Full Revival
- Antidote
- Burn Cure
- Freeze Cure
- Shock Cure
- Sleep Cure
- Confusion Cure
- Fear Cure
- Full Cure

Avoid multiple nearly identical consumables until balancing proves they add value.

### Economy

Use one primary soft currency for shops/rewards: **Prism Marks**.

The economy must define in data:

- buy price
- sell price if sellable
- stack limit
- reward amount
- availability/unlock conditions

No premium currency or real-money economy is part of the current scope.

---

## 13. Prism Orb capture

Base orb families:

- Basic Prism Orb
- Greater Prism Orb
- Element Prism Orb
- Ancient Prism Orb
- Legendary Prism Orb
- Ultimate Prism

Element Prism Orbs exist for all nine elements.

### Capture eligibility

- Wild: allowed by default
- Trainer-owned: not allowed
- Boss: disabled unless explicitly enabled
- Legendary/story/event: explicit data rule

### Capture calculation

Capture is centralized and tunable. Inputs:

- species base capture rate
- current HP ratio
- status modifier
- orb modifier
- element match modifier if applicable
- battle/story restrictions

Recommended probability pipeline:

```text
Chance = BaseCaptureRate
       * HPFactor
       * StatusModifier
       * OrbModifier
       * ContextModifier
```

Clamp ordinary capturable encounters to a configurable minimum/maximum probability. Do not guarantee capture unless the content explicitly says so.

Capture success creates a real persistent `MonsterInstance`, then adds it to party/storage and publishes a capture event.

---

## 14. Progression, XP, skills, and evolution

### XP

Battle rewards grant XP according to defeated opponent level, encounter/boss modifiers, and participation rules.

Level-up flow:

```text
Gain XP
 -> level threshold check
 -> level increase
 -> recalculate stats
 -> check learnable skills
 -> check evolution conditions
 -> publish progression events
```

### Evolution

Evolution is definition-driven and can use reusable conditions:

- level
- item
- quest completion
- story flag
- friendship
- special location
- event condition

Evolution presentation must not contain the gameplay rule itself.

---

## 15. Quest system

Quest states:

```text
Locked -> Available -> Active -> Completed
                     -> Failed only when explicitly designed
```

Core objective types:

- talk to NPC
- reach node/location
- defeat monster/trainer/boss
- capture monster
- collect item
- use item
- win battle
- enter map
- complete quest
- trigger story sequence

Quest definitions include:

- stable ID
- prerequisites
- objective list
- rewards
- failure rules if any
- completion rules
- next quest/unlocks
- repeatability flag

Main-story quests are not repeatable. Optional repeatable content must be deliberately authored rather than inferred.

---

## 16. Main story

### Premise

Aevareth’s nine elemental forces were once stabilized by an ancient Prism network. During the Prism Festival in the Common world, a corrupted Dark Prism activates and destabilizes nearby monsters. The player discovers that someone is gathering elemental energy to create the **Prism of Null**, an artifact capable of absorbing and dominating all nine forces.

The player’s unusual Prism Orb is eventually revealed as the **Ninth Prism**, aligned to Common/Balance and capable of harmonizing the other elements rather than ruling them.

### Central theme

Power is sustainable through balance, not domination.

### Main antagonist

The antagonist believes elemental conflict is proof that freedom creates chaos. Their goal is to collect the elemental cores and force all elements into a single controllable Prism of Null.

Motivation must remain ideological rather than purely destructive: they believe imposed unity will prevent future disasters.

### Player motivation

The player begins by protecting home and investigating the corrupted Prism. Their goal expands into protecting captured partners, helping each region, understanding the ancient system, and ultimately choosing balance over control.

### Rival

The Rival begins as a competitive peer, repeatedly challenges the player, and gradually shifts from proving personal superiority to understanding cooperation and elemental balance.

### Recurring allies

- **Professor Arin** — researcher who gives context but does not solve the mystery for the player.
- **Prism Keeper** — guardian of historical knowledge and safe Prism practices.
- **Marina** — Water trainer; adaptation and responsibility.
- **Terra** — Land trainer; resilience and tradition.
- **Volt** — Electric trainer; ambition, invention, risk.
- **Kairo** — Fire trainer; passion and consequence.
- **Serena** — Ice trainer; discipline and preservation.
- **Aero** — Air trainer; independence and trust.
- **Luna** — Light trainer; hope without denial.
- **Raven** — Dark trainer; fear as a warning rather than evil.

---

## 17. Nine-act story structure

### Act 1 — Meadow of Beginnings / Common

- Prism Festival begins.
- Corrupted Dark Prism causes monster aggression.
- Player investigates Ancient Prism Shrine.
- Message: “When the nine elements awaken, the Prism Gate shall open.”
- Water-signature clue points outward.
- Rival establishes recurring competitive thread.

**Act goal:** teach movement, battle, capture, quests, home, save, and world unlock.

### Act 2 — Azure Tide / Water

- Abnormal tides threaten settlements.
- Prism Shards are being stolen.
- Marina helps investigate.
- Evidence points toward Land-marked transport routes.

**Reveal:** corruption is coordinated, not natural.

### Act 3 — Ancient Terra / Land

- Ruins explain the original balance system.
- Player learns about elemental cores.
- Antagonist faction is linked to excavation activity.

**Reveal:** nine cores can power a larger Prism device.

### Act 4 — Stormspire / Electric

- Prism reactor destabilizes.
- Corrupted Electric monsters appear.
- Antagonist organization becomes visible.
- Next target: Fire Core.

**Reveal:** the enemy has sufficient technology to manipulate Prism energy.

### Act 5 — Emberfall / Fire

- Enemy attempts to awaken/control the Fire Guardian.
- Player interrupts ritual.
- Ancient mural shows nine Guardians around a black Prism.

**Reveal:** Prism of Null concept is foreshadowed.

### Act 6 — Frostveil / Ice

- Artificial amplification freezes the region.
- Player defeats/recovers the affected Guardian.
- Ninth Prism reacts and reveals Skyreach.

**Reveal:** player’s Prism is not ordinary capture technology.

### Act 7 — Skyreach / Air

- Ancient Prism Library is discovered.
- Records define the Prism of Null as an absorber of elemental forces.
- Stopping it requires understanding the Light archive.

**Reveal:** Null can be completed only if Balance/Common is also subverted.

### Act 8 — Solara / Light

- Light Guardian identifies the player’s orb as the Ninth Prism.
- Common is revealed as Balance, not “no element.”
- Enemy’s final route leads to Nocturne Abyss.

**Reveal:** the player can harmonize the cores; this is why the enemy needs them.

### Act 9 — Nocturne Abyss / Dark

- Elemental collapse affects prior regions.
- Recurring trainers/allies contribute support sequences.
- Raven frames fear as information rather than corruption.
- Player confronts antagonist and completed/near-complete Prism of Null.

### Finale — Prism of Null

The final boss uses phase profiles representing the nine elements. The fight reuses the normal battle runtime plus boss rules; it does not require a separate combat engine.

After victory, the player uses the Ninth Prism to restore balance instead of absorbing the cores.

---

## 18. Ending and post-game

After the finale:

- elemental instability stops
- Guardians recover
- major story worlds remain explorable
- NPC dialogue updates to post-game states
- unfinished side quests remain available where logically valid
- trainers can offer rematches through explicit post-game interactions
- rare/legendary encounter chains can remain available
- collection/completion tracking stays active
- selected bosses may have challenge rematches as optional content

The player is recognized as a **Prism Guardian**.

A mysterious Prism fragment may foreshadow future content, but the base story must feel complete without requiring an expansion.

### Post-game goals

- complete monster collection
- finish side quests
- find optional secrets/treasures
- trainer/guardian rematches
- complete world/map completion targets
- optional legendary encounters

Do not add infinite scaling, prestige systems, or procedural endgame unless later testing proves they are needed.

---

## 19. Mission/content structure

Each production map should contain a deliberately small content package:

- one clear main objective path
- optional branch nodes
- a limited set of meaningful NPCs
- encounter table(s)
- collectibles/rewards
- one or more side interactions only where they add value
- map completion conditions

### Typical map flow

```text
Enter map
 -> establish local problem
 -> interact/investigate
 -> optional branch content
 -> trainer or story challenge
 -> dungeon/special area if needed
 -> boss/final objective
 -> resolution
 -> unlock next map/world progression
```

Rewards can include:

- Prism Marks
- consumables
- Prism Orbs
- evolution/key items
- monster/skill access
- story/world unlocks

---

## 20. Events

Events are future content using the same maps/nodes/interactions/quests/battle systems.

Supported future event concepts:

- seasonal map
- limited story
- special trainer chain
- legendary encounter
- anniversary content

### Important scope rule

The current game does **not** require a live-service backend.

Offline/bundled events may be activated by build/version/configuration. If future events have trusted start/end times, competitive rewards, or server-controlled availability, a backend time/entitlement authority must be added then.

Do not rely on the device clock for security-sensitive event eligibility.

---

## 21. Future PvP preparation

PvP is not implemented now.

Foundational choices made now so PvP can be added later:

- battle rules do not depend on scene objects
- battle actions are explicit data/commands
- battle calculations are centralized
- stable content IDs are used
- seeded RNG can be supplied through BattleContext
- presentation is separate from battle authority
- battle results are explicit objects

Do **not** build now:

- networking transport
- lobby
- matchmaking
- ranking
- anti-cheat
- replication
- server authority
- PvP UI

---

## 22. Content scope tiers

### Must-have

- node exploration
- Common vertical slice
- battle/capture
- monster party/storage
- items
- quest/story sequencing
- save/load
- one complete world-quality pipeline proven

### Should-have

- polished home presentation
- advanced trainer AI profiles
- optional boss phases
- richer side quests
- refined collectibles/completion tracking
- content authoring tools justified by production pain

### Optional

- complex home social simulation
- large puzzle catalog
- advanced weather-driven encounters
- extensive cosmetic systems

### Future only

- new maps/regions
- limited events
- PvP

---

## 23. Design consistency rules

1. Core systems are reusable; worlds/maps are content.
2. One map has one primary Battle World reference; boss/night/event differences are presentation variants unless a truly distinct scene is necessary.
3. Land is the canonical element name.
4. No base mana/MP system unless intentionally introduced later.
5. Story sequences orchestrate dialogue/battle/quest actions; NPC scripts do not own the whole flow.
6. Persistent state uses stable IDs, never display names or scene references.
7. Large content production begins only after the vertical slice proves save/load and map-to-battle return.
8. Future expansion is limited to maps, events, and PvP preparation unless a separate approved design expands scope.
9. A feature that cannot be tested independently should be decomposed before production.
10. New content should normally require data and assets, not modifications to core gameplay code.

---

## 24. Game-design acceptance criteria

The design is ready for implementation when the team can answer yes to all of these:

- Can a designer add a monster without editing battle-controller code?
- Can a map author add nodes/conditions/interactions without adding map-specific gameplay logic?
- Can story dialogue launch a battle and resume afterward?
- Can a battle return the player to the exact originating map/node?
- Can quests react to battle/capture/story events without battle knowing about quests?
- Can one map swap battle presentation without changing combat rules?
- Can save/load restore world, map, node, party, inventory, quests, story flags, and settings?
- Can new maps/events reuse the same core systems?
- Can a future PvP layer use battle commands/calculations without depending on Unity presentation objects?

This document is the canonical gameplay/story specification. Technical implementation details live in `archectural_plan.md`.