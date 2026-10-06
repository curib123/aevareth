# Aevareth: Monster Realm

> **Nine Elements. One Realm. Infinite Legends.**

Aevareth is a 3D, node-based monster-collecting RPG built around controlled exploration, turn-based monster battles, Prism Orb capture, story-driven world progression, and reusable data-driven content.

## Project status

**Documentation foundation: Production-ready v1**  
**Unity implementation: Not started in this repository**

The original planning documents have been audited as one connected specification. Contradictions, duplicated concepts, missing rules, technical risks, and unnecessary documentation sprawl were consolidated into the canonical documents below.

## Canonical documentation

Read these in order:

1. [`all_plan.md`](all_plan.md) — canonical game design, gameplay rules, story, missions, post-game, and future scope.
2. [`archectural_plan.md`](archectural_plan.md) — canonical technical architecture, runtime flows, data model, save/load, Unity structure, extensibility, performance, and asset pipeline.
3. [`DEVELOPMENT_ROADMAP.md`](DEVELOPMENT_ROADMAP.md) — dependency-aware implementation phases from prototype to release.
4. [`QA_TESTING.md`](QA_TESTING.md) — test strategy, validation rules, regression coverage, performance targets, and release gates.
5. [`DOCUMENTATION_AUDIT.md`](DOCUMENTATION_AUDIT.md) — issues found in the original plans and the decisions used to resolve them.
6. [`plan-structure.md`](plan-structure.md) — final intentionally small documentation structure and rules for when a future split is justified.

## Locked product pillars

- **3D presentation** with controlled third-person/top-back camera.
- **Node/stepping-stone exploration**: tap/click a valid destination; the avatar moves automatically.
- **Nine elements**: Common, Water, Land, Electric, Fire, Ice, Air, Light, Dark.
- **Monster collection** through Prism Orbs.
- **Turn-based battles** using one universal battle runtime.
- **One primary Battle World reference per exploration map**; battle variants are presentation profiles, not separate battle engines.
- **Data-driven content** for monsters, skills, items, quests, encounters, maps, NPCs, bosses, and events.
- **Persistent GameState** is authoritative for player progression; scenes and presentation objects are not save data.
- **Story, dialogue, and battles are composable actions** orchestrated through reusable interaction/story sequences.
- **Future maps and events** must be addable without rewriting core systems.
- **Future PvP is architecture-prepared only**; networking, matchmaking, replication, and live PvP are not part of the current build scope.

## Core gameplay loop

```text
Explore node graph
    -> resolve node interaction
    -> dialogue / encounter / quest / item / battle
    -> battle or interaction result
    -> progression + rewards
    -> unlock content
    -> save
    -> continue exploration
```

## Production rule

Do not start by producing large amounts of final content.

The first playable proof must validate the complete loop with placeholders:

```text
Map
 -> Node
 -> Interaction / Encounter
 -> Battle Request
 -> Map Battle World
 -> Battle
 -> Battle Result
 -> Return to exact Map + Node
 -> Apply progression
 -> Save
 -> Load
```

Only after that loop is stable should production scale into full monsters, skills, maps, VFX, animation, audio, quests, and story content.

## Scope discipline

### Build now

- Core state/services
- Node exploration
- Interaction/story orchestration
- Battle runtime
- Monster/skill/effect/item foundations
- Capture
- Quest/progression
- Save/load
- One small vertical slice
- Content validation

### Build after the vertical slice proves the architecture

- Production editor tooling where manual authoring is demonstrably slow or error-prone
- Remaining worlds/content
- Home-system polish
- Advanced boss mechanics
- Event framework
- Optimization driven by profiling

### Do not build now

- PvP networking
- Matchmaking
- Live-service backend
- Large event catalog
- Hundreds of monsters before the content pipeline is proven
- ECS/DOTS without measured need
- Separate battle engines per world or encounter type
- Dozens of custom editor windows before the vertical slice

## Naming rule

Use **Land** consistently as the element name. `Earth` is not a separate element.

## Architecture rule of authority

```text
Definitions define what exists.
Systems define what happens.
Runtime state defines what is happening now.
Presentation shows the result.
Persistent GameState owns progression.
```

When documents conflict, this README and the canonical documents listed above take precedence over older commit history.