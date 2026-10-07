# Aevareth: Monster Realm — Unity Development Workflow

## Purpose

This document defines how Aevareth should be implemented as a real Unity game.

The project must use a **hybrid development workflow**:

- **Codex is the primary implementation agent.**
- **Computer Use is the Unity Editor execution and validation agent.**
- GitHub is the source of truth for all project changes.

The goal is not to produce theoretical Unity code only. Every milestone must move the repository toward a playable, testable Unity project.

---

## 1. Core Rule

Use **Codex for code-first development** and **Computer Use for Unity Editor work**.

Recommended split:

- **Codex: ~80%**
- **Computer Use: ~20%**

Do not use Computer Use for tasks that are faster, safer, or more reproducible through source files.

Do not use Codex alone to claim that visual Unity behavior is verified when the Unity Editor has not actually been opened and tested.

---

## 2. Codex Responsibilities

Codex owns the repeatable, source-controlled implementation work.

Use Codex to:

- Create and maintain the Unity project structure.
- Pin and respect the selected Unity editor version.
- Create C# gameplay systems.
- Create reusable components and interfaces.
- Implement monster data and progression.
- Implement starter selection.
- Implement elemental/type rules.
- Implement battle logic.
- Implement skills and skill-use counters such as 3/3 or 12/12.
- Implement melee, ranged, healing, shielding, buff, debuff, status, and utility skills.
- Implement capture mechanics and elemental capture-orb modifiers.
- Implement inventory and consumables.
- Implement monster party/team management.
- Implement encounters.
- Implement quests and objectives.
- Implement NPC interaction logic.
- Implement map/node progression logic.
- Implement save/load.
- Implement settings and configuration.
- Implement UI logic.
- Implement ScriptableObject definitions and validation.
- Implement editor tooling where useful.
- Configure packages and source-controlled Unity settings.
- Configure Input System.
- Configure Addressables when the project reaches the stage where they provide real value.
- Write automated tests.
- Run command-line/static checks that are available.
- Fix compiler errors visible from repository or build output.
- Refactor duplicated or tightly coupled code.
- Maintain documentation when systems change.
- Commit completed milestones with clear messages.

Codex should prefer:

- Data-driven systems.
- Small focused classes.
- Clear ownership of state.
- Deterministic gameplay logic where practical.
- Low coupling between presentation and gameplay logic.
- Simple solutions before advanced frameworks.
- Testable pure C# logic for core battle/calculation systems.

---

## 3. Computer Use Responsibilities

Computer Use owns tasks that genuinely require interaction with the Unity Editor or desktop UI.

Use Computer Use to:

- Open Unity Hub.
- Open the Aevareth Unity project.
- Allow package and asset imports to finish.
- Confirm that Unity recognizes the project.
- Inspect the Unity Console.
- Verify compiler errors and warnings.
- Enter Play Mode.
- Test scenes visually.
- Test player movement.
- Test node/map navigation.
- Test starter selection.
- Test NPC and object interaction.
- Test encounter transitions.
- Test battles.
- Test menus and HUD behavior.
- Test animations.
- Test VFX and particles.
- Test cameras.
- Test lighting.
- Test UI scaling and layout.
- Inspect GameObjects and components.
- Wire scene references in the Inspector only when source-controlled or scripted setup is impractical.
- Configure Animator state machines when visual editing is materially easier.
- Validate prefabs.
- Validate scenes.
- Validate audio behavior.
- Validate Android/Windows build settings.
- Create a local test build when the environment supports it.
- Reproduce bugs that require real runtime interaction.

Whenever possible, convert repetitive Inspector work into prefabs, ScriptableObjects, editor scripts, or source-controlled configuration after the correct setup is understood.

---

## 4. Required Development Loop

Use this loop for every gameplay milestone:

```text
Requirement / milestone
        ↓
Codex implements source changes
        ↓
Static/code validation
        ↓
Unity opened through Computer Use
        ↓
Compile / import
        ↓
Unity Console checked
        ↓
Play Mode test
        ↓
Visual + gameplay validation
        ↓
Issue found?
   ↙             ↘
 Yes              No
  ↓                ↓
Codex fixes      Commit milestone
  ↓                ↓
Retest          Next milestone
```

Do not skip the Unity validation step for systems whose correctness depends on scene behavior, prefabs, animation, rendering, input, physics, UI, audio, or runtime integration.

---

## 5. Unity Version Rule

The project must pin **one exact Unity editor version** in:

`ProjectSettings/ProjectVersion.txt`

Do not casually change the Unity version after implementation begins.

If an upgrade is necessary:

1. Create a dedicated upgrade branch.
2. Record the old and new editor versions.
3. Open the project with the new version.
4. Allow Unity to migrate files.
5. Check Console errors.
6. Run automated tests.
7. Run core Play Mode smoke tests.
8. Inspect package compatibility.
9. Check scenes/prefabs for unintended changes.
10. Merge only after the project is stable.

The selected Unity version must be treated as part of the project's technical contract.

---

## 6. Source-Control Rule

Git is the source of truth.

All meaningful implementation changes must be reproducible from the repository.

Commit:

- Assets that belong in source control.
- C# scripts.
- Scenes.
- Prefabs.
- ScriptableObjects.
- Project settings.
- Package manifests/locks.
- Editor tooling.
- Tests.
- Documentation.

Do not commit generated Unity folders such as:

- `Library/`
- `Temp/`
- `Logs/`
- `Obj/`
- local build cache directories

Use an appropriate Unity `.gitignore`.

Prefer one logical milestone per commit or small related commit group.

Example:

```text
feat(starters): implement three-monster starter selection
feat(battle): add deterministic turn resolution
feat(skills): add usage counters and elemental effects
fix(capture): correct elemental orb catch modifiers
test(battle): cover damage and status resolution
```

---

## 7. Scene and Inspector Policy

Avoid making core gameplay depend on fragile manual Inspector setup.

Prefer:

- ScriptableObjects for content definitions.
- Prefabs for reusable visual/runtime objects.
- Serialized references only when appropriate.
- Runtime/bootstrap installers for deterministic initialization.
- Editor validation scripts for required references.
- Automated scene setup for repeatable structures where practical.

Manual Inspector setup is acceptable when it is clearly the simplest solution, but it must be documented and validated.

Never hide essential gameplay rules only inside a scene or Inspector field without a clear data definition.

---

## 8. Architecture Boundary

Keep **gameplay logic** separate from **presentation**.

Examples:

- Battle calculations should not require a Unity GameObject.
- Damage formulas should be testable without loading a scene.
- Capture probability should be pure/testable logic.
- Element matchup calculations should be independent of VFX.
- Skill definitions should not hard-code a specific monster prefab.
- Monsters should reference data and presentation assets rather than embedding rules in animation code.

Presentation systems can consume gameplay results to play:

- Animations
- VFX
- SFX
- Camera movement
- Hit reactions
- UI updates

This makes the game easier to test, balance, debug, and expand.

---

## 9. AI Asset Production Standard

Aevareth uses an **AI-first asset pipeline, not an AI-only rule**.

AI should be used aggressively to create original production assets where it provides a real quality or speed benefit, while Codex and Unity remain responsible for implementation, integration, optimization, validation, and runtime behavior.

The core principle is:

**AI generates creative source assets → Codex builds the systems and integration → Unity turns those assets into a polished playable game.**

Gameplay code must never depend on unfinished final art. Use placeholders first when necessary.

### 9.1 Assets That Should Be AI-Generated When Practical

Prioritize AI generation for original:

- Monster artwork.
- Every monster evolution stage.
- Alternate monster presentation art where needed.
- NPC and character artwork.
- Character portraits.
- Dialogue portraits.
- Map/environment concept art.
- Environment textures and decorative art.
- Battle backgrounds.
- Parallax layers.
- Loading screens.
- Splash art.
- Menu/key art.
- Item artwork.
- Consumable icons.
- Equipment icons if equipment is introduced.
- Capture-orb designs.
- Element icons.
- Status-effect icons.
- Skill icons.
- Quest icons.
- UI decorative artwork.
- VFX source textures.
- Spell/skill effect textures.
- Particles/sprite sheets when appropriate.
- Music when AI audio quality is production-ready.
- Sound effects when AI audio quality is production-ready.
- Ambient audio when appropriate.

AI-generated assets must remain **original to Aevareth** and must not intentionally imitate or copy protected characters, monsters, logos, or distinctive assets from another game.

### 9.2 Highest-Priority Asset Quality

The highest visual-quality priority is:

1. Monsters.
2. Monster evolution forms.
3. Skills and battle effects.
4. Battle presentation.
5. Maps and environments.
6. Important characters/NPCs.
7. UI and secondary assets.

Monster design is the game's primary visual identity and must receive the most iteration and quality control.

### 9.3 Monster and Evolution Rules

Every monster family must have a recognizable identity while each evolution must show meaningful progression.

Do not create evolution stages that are merely:

- recolors,
- enlarged versions,
- trivial accessory swaps,
- copies of another monster,
- or the same silhouette with minor edits.

Each evolution should communicate increased power, maturity, elemental mastery, specialization, or transformation while preserving enough visual DNA to remain recognizable as the same evolutionary family.

For every monster family, define:

- Core concept.
- Element.
- Body/silhouette language.
- Signature features.
- Personality or visual attitude.
- Evolution progression.
- Final-form visual payoff.
- Essence/final evolution concept where applicable.
- Signature skill visual motifs.

### 9.4 Aevareth Style Bible Requirement

Before large-scale final asset production, maintain an **Aevareth visual style bible**.

It should define:

- Art direction.
- Rendering style.
- Shape language.
- Silhouette rules.
- Monster proportions.
- Character proportions.
- Line treatment if applicable.
- Surface/material treatment.
- Lighting direction.
- Element visual language.
- Rarity visual language.
- UI visual language.
- VFX visual language.
- Environment mood.
- Background depth/parallax rules.
- Animation tone.
- Allowed and disallowed visual traits.

All newly generated assets must be reviewed against the style bible.

Do not accept individually attractive assets that look like they belong to different games.

### 9.5 Element Visual Language

Each element should have a recognizable but flexible visual identity.

Element identity may influence:

- Shape language.
- Materials.
- Energy effects.
- Particle behavior.
- Skill effects.
- UI indicators.
- Evolution details.
- Capture-orb styling.
- Battle feedback.

Element identity must not force every monster of the same element to look alike.

Creativity and monster individuality have higher priority than repeating a single elemental template.

### 9.6 Placeholder-First Production Rule

Use this order:

1. Build gameplay with placeholders.
2. Prove the mechanic works.
3. Define the exact production-asset requirement.
4. Generate candidate AI assets.
5. Review against the Aevareth style bible.
6. Select or iterate the best candidate.
7. Clean/prepare the asset as needed.
8. Import into Unity.
9. Configure import settings.
10. Connect it to data/prefabs.
11. Add animation/VFX/SFX bindings.
12. Validate in Play Mode.
13. Optimize only when needed.

Do not block battle, progression, capture, save/load, map traversal, or other core mechanics because final artwork is unfinished.

### 9.7 Unity-Generated / Unity-Configured Presentation

The following should generally be created or configured inside Unity rather than treated as static AI assets:

- Particle systems.
- Shaders.
- Materials.
- Lighting.
- Post-processing.
- Parallax behavior.
- Camera movement.
- Camera shake.
- Hit stop.
- Screen flashes.
- Damage feedback.
- Runtime transitions.
- UI animation.
- Animation Controllers.
- Timeline sequences when useful.
- Runtime skill composition.
- Layered battle effects.

AI can generate source textures, sprites, references, sound, or concepts for these systems, but Unity owns their final runtime behavior.

### 9.8 Codex Responsibilities for Assets

Codex should:

- Create data models linking gameplay content to presentation assets.
- Create ScriptableObjects.
- Create asset validators.
- Create naming/organization rules.
- Create editor tooling when it removes repetitive work.
- Create reusable VFX/SFX hooks.
- Create reusable skill-presentation interfaces.
- Create import/configuration helpers when justified.
- Detect missing references.
- Detect duplicate IDs.
- Detect invalid evolution chains.
- Detect missing monster/skill icons.
- Detect missing required presentation bindings.
- Keep gameplay logic independent of any specific image/audio file.

### 9.9 File Organization

Keep assets organized by feature and purpose.

Example:

```text
Assets/
├── Aevareth/
│   ├── Art/
│   │   ├── Monsters/
│   │   │   └── <MonsterFamily>/
│   │   │       ├── Stage01/
│   │   │       ├── Stage02/
│   │   │       ├── Stage03/
│   │   │       ├── Stage04/
│   │   │       └── FinalEssence/
│   │   ├── Characters/
│   │   ├── NPCs/
│   │   ├── Environments/
│   │   ├── BattleBackgrounds/
│   │   ├── Items/
│   │   ├── UI/
│   │   ├── Icons/
│   │   └── VFX/
│   ├── Audio/
│   │   ├── Music/
│   │   ├── SFX/
│   │   ├── Skills/
│   │   └── Ambience/
│   ├── Animations/
│   ├── Data/
│   │   ├── Monsters/
│   │   ├── Skills/
│   │   ├── Items/
│   │   ├── Encounters/
│   │   └── Maps/
│   ├── Prefabs/
│   ├── Scenes/
│   ├── Scripts/
│   ├── Tests/
│   └── Editor/
```

### 9.10 Naming and Asset Metadata

Every production asset should have:

- A clear file name.
- Associated monster/skill/item/map ID when relevant.
- Version or revision information when useful.
- Intended use.
- Expected resolution.
- Unity import rule if special handling is required.

Avoid ambiguous names such as:

- `monster1.png`
- `new_final2.png`
- `effect_test.png`

Prefer names such as:

- `MON_Flarecub_S01_Battle.png`
- `MON_Flarecub_S04_Portrait.png`
- `SKL_Fireburst_Impact_01.png`
- `ITEM_FireCaptureOrb_Icon.png`

### 9.11 Generation Provenance and Rights Tracking

Keep a lightweight production record for final AI-generated assets.

Record at minimum:

- Asset ID/path.
- Generation tool/model or source.
- Generation date.
- Whether the asset was manually edited afterward.
- License/usage status when the tool or source has relevant terms.
- Notes for important revisions.

Do not store private API keys or sensitive prompt-service credentials in the repository.

The goal is traceability, not bureaucracy.

### 9.12 Quality Gate

An AI-generated asset is not production-ready merely because it was generated successfully.

Before approval, check:

- Does it match the Aevareth style?
- Is the silhouette readable?
- Does it fit the intended element without becoming generic?
- Is it distinct from existing monsters/assets?
- Does it contain obvious AI artifacts?
- Is anatomy acceptable for the chosen style?
- Are repeated details coherent?
- Does it read correctly at in-game size?
- Does it work against the intended background?
- Does it crop correctly?
- Does transparency work correctly when needed?
- Is the resolution appropriate?
- Does the asset create any obvious copyright/trademark concern?
- Does it animate or layer correctly if animation is required?

If it fails, regenerate, edit, or reject it.

### 9.13 No Random Asset Dumping

Do not generate large numbers of assets without a production requirement.

Every generated production asset must correspond to a defined:

- Monster.
- Evolution.
- Skill.
- Item.
- Character.
- NPC.
- Map.
- Environment.
- UI component.
- VFX requirement.
- Audio requirement.

Generate deliberately and integrate continuously.

### 9.14 Final Asset Principle

Aevareth should be **AI-assisted at production scale but human/Unity validated at quality scale**.

AI generation increases production speed; it does not replace:

- Art direction.
- Consistency checking.
- Gameplay testing.
- Unity integration.
- Performance validation.
- Asset optimization.
- Final quality control.

---

## 10. Testing Requirements

A milestone is not complete because the code exists.

### Codex-side tests

Where practical, add tests for:

- Damage formulas.
- Element modifiers.
- Skill-use consumption.
- Status durations.
- Turn ordering.
- Capture probability.
- Experience/level calculations.
- Evolution conditions.
- Inventory transactions.
- Quest state transitions.
- Save serialization.
- Data validation.

### Unity runtime tests

Use Computer Use/Unity Play Mode for:

- Scene loading.
- Input.
- Cameras.
- Physics.
- Animation.
- VFX.
- Audio.
- UI behavior.
- Prefab integration.
- NPC interaction.
- Overworld progression.
- Battle presentation.
- Save/load integration.
- Build smoke testing.

---

## 11. Milestone Definition of Done

A gameplay milestone is complete only when all applicable items pass:

- Code is implemented.
- Project compiles.
- No new blocking Unity Console errors.
- Relevant automated tests pass.
- Required scene/prefab setup exists.
- Feature works in Play Mode.
- Basic edge cases are checked.
- Data is not unnecessarily hard-coded.
- No obvious duplicate architecture was introduced.
- Documentation is updated when the system contract changed.
- Changes are committed to Git.

If a milestone cannot be validated because Unity/Computer Use is unavailable, label it explicitly as:

`IMPLEMENTED — UNITY VALIDATION PENDING`

Do not call it fully verified.

---

## 12. Recommended Initial Build Order

Use the hybrid workflow in this order:

### Phase 0 — Bootstrap

Codex:

- Create the real Unity project.
- Pin the Unity version.
- Configure Git ignore rules.
- Establish folders/namespaces.
- Add core packages only.
- Create bootstrap scene.
- Add initial automated test assembly.

Computer Use:

- Open project.
- Verify import.
- Verify packages.
- Verify clean compile.
- Enter Play Mode.

### Phase 1 — Core Data

Codex:

- Elements.
- Monster definitions.
- Stats.
- Skills.
- Items.
- Evolution definitions.
- Encounter definitions.

Computer Use:

- Inspect sample ScriptableObjects.
- Confirm assets serialize correctly.

### Phase 2 — Starter Flow

Codex:

- Three starter definitions.
- Fire starter.
- Water starter.
- Land starter.
- Player chooses exactly **one of the three**.
- Persist selection.

Computer Use:

- Test starter UI.
- Test selecting each starter.
- Confirm only one can be chosen.
- Confirm save persistence.

### Phase 3 — Overworld / Node Map

Codex:

- Player/avatar traversal.
- Nodes.
- Node interaction types.
- NPC/dialogue.
- Item pickup.
- Quest step.
- Service.
- Encounter.
- Map exit.

Computer Use:

- Test movement/camera.
- Confirm content can visibly exist beside/on nodes.
- Test interactions and transitions.

### Phase 4 — Battle Vertical Slice

Codex:

- Teams.
- Turns.
- Stats.
- Damage.
- Skills.
- Usage counters.
- Elements.
- Win/loss.
- Rewards.

Computer Use:

- Run complete battle.
- Verify UI.
- Verify animations/VFX hooks.
- Verify transitions.

### Phase 5 — Capture

Codex:

- Capture calculations.
- Element-based orbs.
- Catch modifiers.
- Rare guaranteed-capture orb.
- Inventory consumption.

Computer Use:

- Test failed and successful capture.
- Test elemental modifiers.
- Test guaranteed orb.

### Phase 6 — Progression

Codex:

- XP.
- Levels.
- Evolution stages.
- Party management.
- Items.
- Save/load.

Computer Use:

- Test progression loop end to end.

### Phase 7 — Content and Polish

Codex:

- More monsters.
- Skills.
- encounters.
- maps.
- quests.
- balance data.
- validation tooling.

Computer Use:

- Visual QA.
- camera polish.
- animation polish.
- VFX/SFX validation.
- UI polish.
- performance checks.

### Phase 8 — Release Readiness

Codex:

- Fix outstanding defects.
- strengthen tests.
- remove debug-only behavior.
- prepare build configuration.

Computer Use:

- Full Play Mode regression.
- Build platform validation.
- device/build smoke test when available.

---

## 13. What Not to Do

Do not:

- Generate hundreds of unvalidated Unity files at once.
- Build every future system before the playable core exists.
- Couple game rules directly to scene objects.
- Depend on final art before gameplay works.
- Claim runtime behavior is verified without running Unity.
- Introduce networking/PvP now only because PvP may exist later.
- Add packages because they are popular rather than necessary.
- Overuse singleton managers.
- Put unrelated responsibilities into one manager.
- Hard-code monsters, skills, maps, or encounters into UI scripts.
- Manually repeat Inspector work when a reusable data/prefab/editor solution is simpler.
- commit generated Unity cache folders.

---

## 14. Decision Rule: Codex or Computer Use?

Ask:

**Can this task be expressed safely and reproducibly as source-controlled files?**

If **yes**, use Codex.

If **no**, and it requires seeing/interacting with the actual Unity Editor, use Computer Use.

Examples:

| Task | Tool |
|---|---|
| Write battle logic | Codex |
| Add monster data model | Codex |
| Implement save system | Codex |
| Refactor architecture | Codex |
| Create editor validator | Codex |
| Open Unity project | Computer Use |
| Check Console | Computer Use |
| Play-test battle | Computer Use |
| Inspect animation visually | Computer Use |
| Tune camera visually | Computer Use |
| Confirm UI layout | Computer Use |
| Verify build settings | Computer Use |

---

## 15. Final Execution Principle

Aevareth should be developed as a continuous verified loop:

**Plan → Codex implementation → Unity validation with Computer Use → fix → retest → commit → next milestone.**

The project should always prefer a **small playable and verified increment** over a large amount of untested generated code.
