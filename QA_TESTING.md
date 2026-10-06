# Aevareth QA & Testing Strategy

**Goal:** Prevent progression, save, battle, content, and performance defects before they scale across nine worlds.

Testing should favor fast deterministic checks for rules and focused PlayMode/integration tests for Unity-specific flows.

---

## 1. Quality priorities

Highest-risk areas, in order:

1. Save corruption or lost progression
2. Battle state/result duplication or soft-lock
3. Map -> Battle World -> map return failures
4. Quest/story progression becoming impossible or duplicating rewards
5. Invalid content references as the content library grows
6. Capture/party/inventory inconsistencies
7. Node graph soft-locks and unreachable progression
8. Performance degradation from production assets
9. UI/input failures on target aspect ratios/devices
10. Event/future-content compatibility with existing saves

---

## 2. Test layers

### EditMode / unit tests

Use for logic that should not require a loaded scene:

- stable ID validation
- condition evaluation
- battle action validation
- turn order
- damage calculation
- effect processing
- status duration/stacking
- capture chance inputs
- party/storage rules
- inventory transactions
- quest objective evaluators
- progression unlock rules
- save serialization DTOs
- save migrations
- content validators

### PlayMode tests

Use for Unity integration:

- node selection/movement
- camera/input mode transitions
- interaction sequence playback
- map loading
- Battle World loading/unloading
- battle UI/runtime integration
- return to exact node
- save/reload scene checkpoint
- home presentation lifecycle

### Manual gameplay tests

Use for qualities automation does not judge well:

- movement readability
- camera framing
- battle pacing
- VFX readability
- audio feedback
- tutorial clarity
- difficulty/balance
- map navigation comprehension
- story pacing
- accessibility/usability

---

## 3. Determinism and reproducibility

Battle tests must be able to inject a fixed RNG seed.

A bug report for battle should capture where possible:

- build version
- battle seed
- player party IDs/levels
- opponent party
- BattleRules ID
- source map/node
- selected actions
- resulting exception/log

This allows difficult ordering/capture/critical-hit defects to be reproduced.

---

## 4. Foundation tests

### Bootstrap

Verify:

- services initialize once
- required service failure prevents unsafe continuation
- new game state is valid
- loading state transitions to expected mode

### Game modes

Verify legal/illegal transitions.

Examples:

- Exploration -> BattleLoading: allowed
- Battle -> Exploration directly before result finalization: rejected
- Moving -> second movement request: rejected or ignored by defined policy
- StoryDialogue -> BattleLoading -> Battle -> StoryDialogue: supported

---

## 5. Stable ID and content validation

Release validation must fail for:

- duplicate IDs
- empty required IDs
- references to missing IDs
- map without BattleWorldID
- Battle World without required spawn markers/profile data
- node connection to missing node
- quest objective referencing missing content
- monster referencing missing skill/evolution target
- item with negative stack limit/price
- invalid status duration/stack rule
- invalid element ID

Warnings may be used for non-blocking quality issues, but broken references are release blockers.

---

## 6. Node/exploration tests

Automate:

- valid adjacent node can be selected
- non-adjacent node cannot be selected
- locked node cannot be selected
- unlocking condition updates availability
- movement resolves arrival once
- repeated taps do not duplicate movement
- disabling input during cutscene/battle works
- returning from battle restores correct current node
- node interaction does not replay unintentionally when returning unless configured to do so

Manual:

- valid nodes are visually understandable
- camera does not hide the next legal choice
- paths do not clip through obvious geometry
- movement duration feels responsive

---

## 7. Interaction/story tests

Automate sequence runner behavior:

- actions execute in exact order
- dialogue waits for advance when required
- auto-advance respects configuration
- battle action pauses the sequence
- battle result resumes at the correct action
- one-time reward action cannot fire twice
- set-flag action persists through save/reload
- missing action target fails safely with useful diagnostics

Manual:

- speaker identity is clear
- text does not overflow
- skip/advance behavior is predictable
- camera focus supports readability rather than causing motion discomfort

---

## 8. Battle runtime tests

### Action validation

Test:

- actor is active/alive
- target is valid/alive unless action allows defeated target
- skill is legal in current state
- switch target is in party and not already active/defeated
- item exists and is usable
- capture is allowed for battle/target
- escape follows BattleRules

Invalid actions must not partially mutate state.

### Turn order

Test:

- higher priority resolves first
- same priority uses effective Speed
- ties are deterministic under fixed seed
- status/stat modifiers affect order correctly

### Damage

Test:

- neutral multiplier
- strong multiplier
- resisted multiplier
- critical multiplier
- minimum damage
- defense never causes divide-by-zero
- extreme valid values remain bounded and do not overflow

### Effects/status

Test:

- multi-effect skill resolves in defined order
- cleanse removes eligible status only
- non-stackable status does not duplicate
- stackable status respects cap
- duration decrements on correct phase
- periodic damage/heal fires once per intended hook
- defeat caused by periodic damage enters defeat resolution correctly

### Battle end

Test:

- victory result created once
- defeat result created once
- capture conclusion does not also issue duplicate victory rewards
- forced switch happens before next choice phase
- no action accepted after conclusion

---

## 9. Battle World transition tests

This integration path is a release-critical regression suite.

Test from:

- wild encounter
- trainer interaction
- story sequence
- boss interaction

For each:

1. create source map/node state
2. trigger battle
3. verify correct map BattleWorldID/profile
4. complete battle with deterministic result
5. unload Battle World
6. restore exact map/node
7. apply result once
8. resume story/interaction if applicable

Failure-path tests:

- Battle World missing
- load operation fails
- battle initialization fails
- return map reload fails

Expected behavior: explicit error, no duplicate rewards/state mutation, and safe recovery where possible.

---

## 10. Capture tests

Test:

- trainer monster cannot be captured
- disabled boss/legendary capture is rejected
- valid wild capture consumes one orb per committed attempt
- failed attempt leaves monster uncaptured
- successful capture creates exactly one unique MonsterInstance
- full party routes capture to storage
- instance survives save/reload
- capture event progresses only matching quests
- fixed seed reproduces capture result where calculation uses seeded RNG

Boundary cases:

- target at full HP
- target at 1 HP
- status present/absent
- minimum/maximum configured catch probability
- inventory contains exactly one orb

---

## 11. Inventory/economy tests

Test:

- item quantity cannot become negative
- stack cap enforced
- transaction validates before mutation
- failed purchase does not remove currency
- successful purchase changes currency/item once
- sell rules obey sellable flag/value
- key/quest items cannot be consumed/sold unless explicitly configured
- save/reload preserves all quantities and Prism Marks

---

## 12. Party/monster progression tests

Test:

- starter selection creates intended instances
- party limit is 6
- duplicate instance IDs rejected
- captured monster routes correctly when party full
- XP threshold exact boundary
- one reward can trigger multiple level-ups safely
- stats recalculate deterministically
- skill-learning eligibility triggers once
- evolution condition evaluates from persistent state
- canceled/deferred evolution does not corrupt species/state

---

## 13. Quest/progression tests

Every objective type requires at least one positive and one negative test.

Test:

- only matching event progresses objective
- count objectives stop at required amount
- completion fires once
- rewards grant once
- prerequisite gating works
- completed main quest cannot reactivate
- failed quest behavior exists only where defined
- reload preserves active quest counters
- world unlock occurs only after required conditions
- sequence/battle interruption does not skip objective update

Regression test each main-story critical path before release.

---

## 14. Save/load tests

This is the highest-priority automated suite.

### Round-trip

Save and load:

- player state
- party/collection
- inventory/currency
- quests/objectives
- story flags
- world/map unlocks
- current safe map/node checkpoint

### Integrity

Test:

- checksum mismatch detected
- truncated file rejected
- malformed JSON/data rejected
- invalid enum/ID handled by migration/fallback policy
- prior known-good backup remains usable

### Atomic-write behavior

Simulate interruption/failure where tooling allows.

Expected: previous valid save remains available.

### Migration

For every public schema version supported by an update:

```text
old save -> migration chain -> current valid save -> gameplay load
```

No public release should ship a schema change without a migration test or an explicit documented incompatibility policy.

### Mid-battle

Base v1 does not support saving mid-battle. Verify autosave never serializes an unrecoverable active battle state.

---

## 15. Content progression validation

Before a world is declared complete, verify:

- all required maps reachable
- required story nodes reachable in intended order
- no required quest depends on impossible condition
- exits point to valid destinations
- bosses/trainers reference valid parties
- required items can be obtained before they are required
- required capture objective has at least one valid encounter source
- final world completion reaches ending/post-game state

For large node graphs, add automated graph reachability checks where practical.

---

## 16. UI/UX testing

Test major screens on supported aspect ratios and safe areas:

- HUD
- node selection prompts
- dialogue
- battle menu
- skill selection
- item selection
- switch/party
- capture feedback
- quest log
- inventory
- home screens
- settings/save/load feedback

Check:

- no clipped text
- no inaccessible controls
- correct disabled states
- touch targets are usable
- loading indicators appear immediately for async transitions
- repeated taps cannot duplicate actions
- navigation back/cancel behavior is consistent

---

## 17. Performance testing

Track separately:

- CPU frame time
- GPU frame time
- memory
- allocations/GC
- scene transition time
- Addressables load time once used
- texture/audio memory
- home monster simulation cost
- battle VFX spikes

### Representative performance scenes

Maintain at least:

1. dense exploration map
2. typical Battle World
3. worst-case boss VFX battle
4. home with representative visible monster count
5. UI-heavy inventory/collection screen

### Targets

- 60 FPS target where practical on baseline production device
- 30 FPS minimum on lower-tier supported device class
- no sustained unbounded memory growth across repeated map/battle transitions
- no recurring large GC spikes during normal turn selection or node movement

Optimization changes must cite a measured bottleneck.

---

## 18. Soak and lifecycle testing

Run long sessions covering repeated:

```text
map -> battle -> map -> home -> map -> battle -> save -> reload
```

Check for:

- leaked scenes/assets
- duplicated event subscriptions
- increasing memory
- duplicated UI
- stale input locks
- story sequence resume failures
- save-size growth unrelated to progression

Also test application pause/resume and OS interruption on supported mobile platforms.

---

## 19. Compatibility testing

For each supported platform/device tier:

- fresh install
- update from previous release
- save migration
- different aspect ratios
- low-memory behavior
- app background/foreground
- audio interruption
- storage write failure/full-storage handling where practical

Do not claim device support that has not been tested.

---

## 20. Regression strategy

Every fixed blocker/critical bug should gain one of:

- automated unit test
- automated PlayMode/integration test
- content validator rule
- documented manual regression case

Prefer automation for deterministic defects.

Maintain a small smoke suite that runs on every meaningful change:

1. boot new game
2. node movement
3. dialogue
4. battle
5. capture
6. quest progress
7. save
8. reload
9. exact node restore

---

## 21. Severity

### Blocker

- cannot boot/progress
- widespread crash
- save corruption/data loss
- release build unusable

### Critical

- main story soft-lock
- battle cannot complete
- duplicate major rewards/progression exploit
- reliable severe crash in common flow

### Major

- important feature broken with workaround
- incorrect quest/monster/item behavior
- serious UI/input failure

### Minor

- cosmetic issue
- low-impact audio/VFX/layout issue
- typo without gameplay impact

No known blocker or critical defect may ship.

---

## 22. Release test gate

Before release candidate approval:

- automated EditMode suite passes
- PlayMode critical-path suite passes
- content validation has no errors
- full main-story critical path completed on RC build
- new-game and migrated-save runs completed
- clean install/update tested
- target-device performance meets supported minimum
- soak test shows no serious leak/state accumulation
- no blocker/critical defects open
- known major defects have explicit release decision and user impact understood

---

## 23. QA principle

Aevareth becomes harder to test as content multiplies. Therefore, invest early in **rules tests, state-transition tests, and content validation**, not in a huge test bureaucracy.

The objective is simple: a new map, monster, quest, skill, or event should fail fast during validation/testing if it breaks a core contract, long before a player discovers the defect.