# Contributing to GodotPrompter

Thanks for your interest in contributing! GodotPrompter is an open-source skills framework for Godot 4.x. Please review our [Code of Conduct](CODE_OF_CONDUCT.md) before participating. Here's how to add skills and improve existing ones.

## Adding a New Skill

### 1. Create the skill folder

```
skills/<skill-name>/
  SKILL.md          # Required — main skill document
  *.md              # Optional — supporting references
```

Use **kebab-case** for folder names (e.g., `my-new-skill`).

### 2. Write SKILL.md with frontmatter

Every `SKILL.md` must start with YAML frontmatter:

```yaml
---
name: my-new-skill
description: Use when [specific trigger] — [brief scope]
---
```

- `name` must match the folder name
- `description` should start with "Use when" to help agents decide when to load it

### 3. Structure your content

Follow this general structure:

1. **Title and intro** — What this skill covers, when to use it
2. **Related skills** — `> **Related skills:** **skill-a** for X, **skill-b** for Y.`
3. **Numbered sections** — Each major concept or pattern
4. **Code examples** — GDScript first, then C# equivalent
5. **Checklist** — Implementation checklist at the end

### 4. Code examples

- Include **both GDScript and C#** where applicable
- GDScript comes first, C# follows
- Use ` ```gdscript ` and ` ```csharp ` language tags (never `gd` or `cs`)
- Target **Godot 4.3+** APIs only — no deprecated methods
- Follow Godot style: snake_case for GDScript, PascalCase for C#

### 5. Cross-references

Add a related skills line after the intro paragraph:

```markdown
> **Related skills:** **event-bus** for decoupled communication, **component-system** for composition patterns.
```

Keep to 3-5 references max. Only link genuinely related skills.

## Scope

This repo ships a deliberately lean set of broadly-applicable engine and architecture skills. Game-genre-specific skills (inventory, dialogue, abilities), platform-specific skills (mobile, XR, dedicated server), and community-addon skills (LimboAI, Beehave, etc.) are intentionally **out of scope** — they carry opinions that do not fit every project. New skills should clear the same bar: useful in almost any Godot 4.x project, and not a thin wrapper over one library or genre.

## Improving Existing Skills

- Fix incorrect API references or deprecated methods
- Add missing C# examples where GDScript-only
- Add cross-references to related skills
- Expand checklist items
- Fix typos or unclear wording

## Token Budget (enforced)

Every `SKILL.md` must stay **under 16 KB** (16,384 bytes). Since v1.12.0 this is a **hard rule**: the
validator raises a `token-budget-exceeded` **error** and CI fails the release. An advisory
`token-budget-approaching` **warning** fires from 15.5 KB (15,872 bytes) so you get a signal before the
wall.

Bytes are measured **LF-normalized**, so a Windows (CRLF) checkout reports the same numbers as CI —
trust `node scripts/validate-skills.mjs` over a raw `wc -c`.

If a skill would exceed the budget, don't cut teaching content — restructure with **Pattern X**: keep the
canonical recipe, key decisions, and anti-patterns in `SKILL.md`, and move deep dives into
`skills/<name>/references/<topic>.md` (unlimited size, loaded only when an agent opens them). Link every
reference file from `SKILL.md` or the validator will flag it as orphaned.

## Testing Skills

Before submitting:

1. **Read through** — Does the skill make sense for someone new to Godot?
2. **Try the code** — Open Godot 4.3+ and verify examples compile and run
3. **Check C# parity** — Every GDScript example should have a C# equivalent (unless language-specific)
4. **Verify cross-refs** — Referenced skills must exist
5. **Run the validator** — `node scripts/validate-skills.mjs` must report **0 errors** (it checks
   frontmatter, cross-references, the token budget, and orphaned reference files)

## Adding Agents

Agent definitions go in `agents/<agent-name>.md` with YAML frontmatter:

```yaml
---
name: my-agent
description: |
  When to use this agent, with examples.
model: inherit
---

Agent system prompt goes here.
```

## Releasing a New Version

When publishing a new version (e.g., v1.8.1):

1. **Make changes** in the GodotPrompter repo.
2. **Regenerate the token-budget docs page** (added in v1.7.0):
   ```bash
   npm install                                                # one-time, installs optional tokenizer deps
   node scripts/count-tokens.mjs --tokenizer --markdown
   ```
   Replace the contents between the `<!-- BEGIN-TOKEN-TABLE -->` / `<!-- END-TOKEN-TABLE -->` markers in `docs/token-budget.md` with the new output. Commit alongside the version bump.
3. **Bump version across all manifests** using the helper script:
   ```bash
   node scripts/bump-version.mjs 1.8.1
   ```
   This updates five in-repo manifests in one shot:
   - `package.json`
   - `.claude-plugin/plugin.json`
   - `.claude-plugin/marketplace.json` (the `godot-prompter` plugin entry)
   - `.cursor-plugin/plugin.json`
   - `plugin.json` (at root, for Antigravity CLI)

   It also attempts to bump sibling marketplaces when present at known relative paths:
   - `../skillsmith/.claude-plugin/marketplace.json` (or `../../AI/skillsmith/.claude-plugin/marketplace.json`)
   - `../godot-prompter-marketplace/.claude-plugin/marketplace.json`
4. **Update `CHANGELOG.md`** by adding a `## [1.8.1]` section.
5. **Validate skills and hooks** — both run in CI on the release tag, so failing here fails the release:
   ```bash
   node scripts/validate-skills.mjs   # must report 0 errors
   npm test                           # hooks + validator, must be all-pass
   ```
   If you touched `hooks/`, also confirm the scripts are still tracked executable and LF-pinned —
   `chmod +x` alone is a no-op in this repo because `core.filemode=false`:
   ```bash
   git ls-files -s hooks/             # session-start and run-hook.cmd must be 100755
   git check-attr text eol -- hooks/session-start hooks/run-hook.cmd
   ```
6. **Commit and tag:**
   ```bash
   git add -A
   git commit -m "chore: bump version to 1.8.1"
   git tag -a v1.8.1 -m "v1.8.1 — description of changes"
   git push origin master --tags
   ```
7. **Let GitHub Actions publish the release** (`.github/workflows/release.yml`):
   - Verifies tag and manifest versions are consistent
   - Runs `scripts/validate-skills.mjs`
   - Creates the GitHub Release from the matching `CHANGELOG.md` section
   - Opens marketplace bump PRs when `MARKETPLACE_TOKEN` is configured
8. **If marketplace PRs are skipped** (missing `MARKETPLACE_TOKEN`), manually bump:
   - `skillsmith/.claude-plugin/marketplace.json` (primary distribution)
   - `godot-prompter-marketplace/.claude-plugin/marketplace.json` (legacy)

Users update with:
```bash
claude plugins update godot-prompter          # Claude Code
copilot plugin update godot-prompter          # Copilot CLI
agy plugin update godot-prompter              # Antigravity CLI
```

## Conventions

- Skills must be self-contained and independently useful
- One skill per folder under `skills/`
- GDScript follows [Godot style guide](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_styleguide.html)
- C# follows [Godot C# conventions](https://docs.godotengine.org/en/stable/tutorials/scripting/c_sharp/c_sharp_style_guide.html)
- Target Godot 4.3+ minimum
- YAML frontmatter is required on every SKILL.md

## Questions?

Open an issue on GitHub or check existing skills for examples of the expected format.

## Editing the session card

The SessionStart hook injects the region between `<!-- SESSION-CARD-START -->` and
`<!-- SESSION-CARD-END -->` in `skills/using-godot-prompter/SKILL.md`, and the `MENTOR-CARD`
region in `skills/godot-mentor/SKILL.md`. Both are validated: markers must be present, appear
exactly once, be correctly ordered, be non-empty, and the region must stay under 3 KB.

The cards are injected on every session start and every compaction, so keep them lean — route by
category and defer detail to the skill. **Do not reproduce the marker strings in documentation
examples**; `card-marker-duplicate` will fail CI, because the hook extracts the first region only
and a documented example above the real card would silently become the injected payload.

After any change under `hooks/`, run `npm run test:hooks`; after any change to
`scripts/validate-skills.mjs`, run `npm run test:validator`. `npm test` runs both. Note that
`node --test tests/hooks/` does not work on Node 24 — a directory argument is imported as a
module — so use the npm scripts.

### The two hook directories

| Path | Runs where | Purpose |
|---|---|---|
| `hooks/` | the plugin **user's** machine | SessionStart routing card — shipped |
| `scripts/hooks/` | this repo, during development | `validate-skill-on-edit.mjs`, wired via `.claude/settings.json` |

Never merge them. Files under `hooks/` are pinned to LF in `.gitattributes` and tracked with the
exec bit — `bash` fails on `$'\r'`, and `chmod +x` alone is a no-op here because
`core.filemode=false`. Use `git update-index --chmod=+x` if you add another hook script.
