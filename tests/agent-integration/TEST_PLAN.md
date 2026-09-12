# GodotPrompter Agent Integration Test Plan

Run these tests in a **fresh Claude Code session** with GodotPrompter installed.
Record results in `RESULTS.md` after each test.

---

## Category 1: Cold Start (Installation & Discovery)

### Test 1.1: Plugin loads

**Setup:** Fresh Claude Code session with GodotPrompter installed.

**Prompt:** "What Godot skills are available from GodotPrompter?"

**Expected:**
- Agent loads `using-godot-prompter` skill (or reads it)
- Lists skill categories: Core/Process, Architecture & Patterns, Gameplay & Systems, Physics & 2D/3D, UI, Scripting & Math
- Mentions at least 10 specific skill names

**Pass criteria:** Agent shows awareness of the skill catalog, not generic Godot advice.

---

### Test 1.2: Skill content access

**Prompt:** "What does the state-machine skill cover? Show me the approaches."

**Expected:**
- Agent reads `skills/state-machine/SKILL.md`
- Describes 3 approaches: enum-based, node-based, resource-based
- Shows the comparison table from the skill

**Pass criteria:** Response matches skill content, not generic FSM knowledge.

---

### Test 1.3: Cross-reference navigation

**Prompt:** "The state-machine skill mentions related skills. What are they?"

**Expected:**
- Agent finds the Related Skills line: resource-pattern, animation-system, event-bus, component-system
- Can describe what each related skill covers

**Pass criteria:** Agent navigates cross-references correctly.

---

## Category 2: Skill Discovery (Open-Ended Prompts)

### Test 2.1: State machine request

**Prompt:** "I need to add a state machine to my player character in Godot 4."

**Expected skill:** `state-machine`

**Expected behavior:**
- Loads the skill (not generic advice)
- Asks about complexity to recommend enum vs node vs resource approach
- Shows GDScript example from the skill
- Mentions C# equivalent

**Pass criteria:** Uses skill content, not generic FSM tutorial.

---

### Test 2.2: Project setup request

**Prompt:** "I'm starting a new Godot 4.3 project. How should I organize it?"

**Expected skill:** `godot-project-setup`

**Expected behavior:**
- Shows the split layout directory structure from the skill
- Recommends autoloads (GameManager, EventBus)
- Shows .gitignore template

**Pass criteria:** Directory structure matches skill exactly.

---

### Test 2.3: Input request

**Prompt:** "I want keyboard and gamepad support with rebindable actions in my Godot 4 game."

**Expected skill:** `input-handling`

**Expected behavior:**
- Shows Input Map action setup
- Uses `Input.get_vector` / action strings rather than raw key codes
- Shows the action-rebinding flow from the skill
- Mentions device/gamepad handling

**Pass criteria:** Uses Input Map actions (not hardcoded keys), shows rebinding code from skill.

---

### Test 2.4: Save/load request

**Prompt:** "Help me set up a save/load system for my Godot game."

**Expected skill:** `save-load`

**Expected behavior:**
- Shows strategy comparison table (ConfigFile, JSON, Resource)
- Recommends JSON for game saves
- Shows SaveManager autoload pattern
- Mentions version migration

**Pass criteria:** Shows the comparison table, recommends JSON with reasoning.

---

### Test 2.5: Code review request

**Prompt:** "Review this GDScript for common Godot issues."

**Sample script to paste with the prompt:**

```gdscript
extends CharacterBody2D

var health = 100
var speed = 200

func _process(delta):
    var player = get_node("/root/Main/Player")
    if player:
        var dir = (player.position - position).normalized()
        position += dir * speed * delta

func take_damage(amount):
    health -= amount
    if health <= 0:
        get_parent().remove_child(self)
        queue_free()
```

**Expected:** `godot-code-reviewer` agent

**Expected behavior:**
- Flags: untyped variables, using `_process` instead of `_physics_process` for movement
- Flags: hardcoded node path `/root/Main/Player` (use groups instead)
- Flags: `position +=` instead of `move_and_slide()` on CharacterBody2D
- Flags: `remove_child` before `queue_free` (unnecessary)
- Uses the agent's checklist structure

**Pass criteria:** Finds at least 3 of the 4 issues, uses the checklist format.

---

## Category 3: Full Workflow (End-to-End Build)

### Test 3.1: Project + Player

**Setup:** Empty directory, no existing Godot project.

**Prompt:** "Create a new Godot 4.3 project with a player that can move with WASD and attack with Space."

**Expected skills:** `godot-project-setup`, `input-handling`, `state-machine`

**Expected behavior:**
- Scaffolds project with directory structure from godot-project-setup
- Sets up Input Map actions and reads them via `input-handling`
- Moves a `CharacterBody2D` with `move_and_slide` per `physics-system`
- Adds FSM (idle/move/attack) from state-machine

**Pass criteria:** All 3 skills used, project structure matches skill patterns.

---

### Test 3.2: Add Enemy

**Prompt:** "Add an enemy that switches between idle, chase, and attack states, and takes damage from the player."

**Expected skills:** `state-machine`, `component-system`, `physics-system`

**Expected behavior:**
- Adds FSM (idle/chase/attack) from state-machine
- Uses Hitbox/Hurtbox/Health components from component-system
- Uses `Area2D` overlap and collision layers per physics-system

**Pass criteria:** Explicit state machine, component-based damage.

---

### Test 3.3: Add HUD

**Prompt:** "Add a health bar HUD that shows the player's health."

**Expected skills:** `godot-ui`, `event-bus`

**Expected behavior:**
- Creates a `CanvasLayer` + `Control` HUD from godot-ui
- Uses EventBus pattern from event-bus for health updates
- Anchors/containers keep the bar positioned across resolutions

**Pass criteria:** CanvasLayer HUD, EventBus-driven updates.

---

### Test 3.4: Code Review

**Prompt:** "Review all the code we just wrote for Godot best practices."

**Expected:** `godot-code-reviewer` agent

**Expected behavior:**
- Works through the agent's checklist sections
- Checks node architecture, style, performance, input, signals, resources
- Produces structured review output

**Pass criteria:** Uses the checklist format, not ad-hoc review.

---

### Test 3.5: Save/Load

**Prompt:** "Set up save/load for player position and health. F5 to save, F9 to load."

**Expected skill:** `save-load`

**Expected behavior:**
- Creates SaveManager autoload with JSON serialization
- Implements save_game/load_game functions
- Wires to input actions
- Includes version migration pattern

**Pass criteria:** JSON save with version field, matches skill's SaveManager pattern.

---

## How to Run

1. Start a fresh Claude Code session
2. Install GodotPrompter: `claude plugins add ./GodotPrompter`
3. Navigate to an empty test directory
4. Run each test sequentially, recording results in RESULTS.md
5. For Category 3, keep the same session (tests build on each other)

---

## Category 5: Mentor Mode & Presence (v1.13.0)

### Test 5.1: Mentor mode wraps, never replaces

**Setup:** Godot project with mentor mode activated (state in `~/.godot-prompter/state/`).

**Prompt:** "add an attack state to my player"

**Expected:**
- Agent invokes `godot-prompter:state-machine` (visible in the tool call)
- All five beats in order: Concept, Editor, Code, Verify, Next
- Exactly one suggestion in Beat 5

**Pass criteria:** The domain skill is loaded. A five-beat answer with no `state-machine`
invocation is a FAIL — that is the primary anti-pattern.

---

### Test 5.2: Editor beat boundary holds

**Prompt:** "where do I click to add an autoload?"

**Expected:** names the Project Settings → Autoload area at panel level and the fields to fill
in; does **not** invent toolbar positions, dock coordinates, or version-specific UI chrome.

**Pass criteria:** no fabricated click-path. Must keep passing after v1.14.0 relaxes the
constraint — answers get fuller, never fabricated.

---

### Test 5.3: Off-ramp

**Prompt (after 5.1):** "just give me the code"

**Expected:** plain code, no beats; the state file's `mode` becomes `"normal"`.

---

### Test 5.4: Coexistence regression

**Setup:** fresh session in a Godot project with Superpowers also installed.

**Prompt:** "implement a save system for my game"

**Expected:** `godot-prompter:save-load` is invoked during implementation, whether or not
Superpowers drives the workflow.

**Pass criteria:** this is the regression the release exists to fix. Generic serialization advice
with no `save-load` invocation is a FAIL.

---

### Test 5.5: Hook stays silent outside Godot

**Setup:** fresh session in a non-Godot repository.

**Expected:** no GodotPrompter routing card in context; no unprompted mention of Godot skills.

---

### Test 5.6: Subagent reach via the agent instructions file

**Setup:** Godot project whose instructions files have no `## GodotPrompter` section.

**Prompt:** "build me an event bus for my game" — then let the agent dispatch subagents.

**Expected:**
- The agent offers once to add the `## GodotPrompter` section, and waits for agreement
- It does **not** add it silently
- After the section exists, a dispatched subagent implementing a Godot system invokes the
  matching skill

**Pass criteria:** this is the only test covering subagents. The SessionStart hook does not reach
them; the instructions file is the mechanism.

---

### Test 5.6b: The offer respects an agent-agnostic repo (#15)

**Setup:** Godot project with an `AGENTS.md` and no `CLAUDE.md`.

**Variant A** — `AGENTS.md` already contains a `## GodotPrompter` section.

**Expected:** no offer at all, on this session or any later one.

**Variant B** — `AGENTS.md` has no such section. Decline the offer, then restart the session.

**Expected:**
- The offer names `AGENTS.md`, not a `CLAUDE.md` the repo does not keep
- On Claude Code it may mention that a `CLAUDE.md` containing `@AGENTS.md` would load it here too,
  as a suggestion only
- After the refusal, `~/.godot-prompter/state/<hash>.json` carries `"section_offer": "declined"`,
  any mentor keys in it survive, and the restarted session does not ask again

---

### Test 5.7: No state written into the game repo

**Setup:** activate mentor mode in a Godot project, then `git status` in that project.

**Expected:** clean. No `.godot-prompter/` directory, no new untracked files.

**Pass criteria:** state belongs in `~/.godot-prompter/state/`.

---

### Test 5.8: C# project leads with C#

**Setup:** Godot project whose `project.godot` has `config/features=PackedStringArray("4.5", "C#", "Forward Plus")`.

**Prompt:** "add a health component"

**Expected:** the C# example leads (the hook detects the `C#` feature tag); the renderer is
reported as Forward Plus, **not** "C#".

**Pass criteria:** guards the token-position parsing bug — see `tests/hooks/`.
