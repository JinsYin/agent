---
name: x-save-skill
description: Save a skill (SKILL.md and its resources) to the correct location. Use this skill whenever the user says "save skill", "install skill", "保存技能", or after creating/editing a skill and wanting to persist it. Also use it when the user wants to move a skill between project-level and user-level, or set up symlinks for a skill across Claude Code and Cursor.
---

# Save Skill

Persist a skill to project-level or user-level storage, with automatic symlink setup for cross-tool access.

## How It Works

There are two storage levels:

- **Project-level** (default): Saves to `<project-root>/.claude/skills/<skill-name>/`. Only available in that project.
- **User-level**: Saves to `~/.agents/skills/<skill-name>/` and creates symlinks in `~/.claude/skills/` and `~/.cursor/skills/` so both Claude Code and Cursor can use it.

## Workflow

### 1. Identify the Skill

Figure out where the skill content is coming from:

- **A file path** the user provides (e.g., `./my-skill/SKILL.md`)
- **A directory** containing SKILL.md and optional bundled resources
- **Conversation context** — a skill was just created or output in the current session

Read the SKILL.md frontmatter to get the `name` field. This becomes the directory name.

### 2. Determine Save Level

If the user hasn't specified, ask them. Use this as a guide:

- Skills useful across many projects → user-level
- Skills specific to one codebase → project-level

### 3. Execute the Save

#### Project-level

```bash
mkdir -p <project-root>/.claude/skills/<skill-name>
# Copy SKILL.md and any bundled resources (scripts/, references/, assets/) into it
```

#### User-level

```bash
# Step 1: Save to central repository
mkdir -p ~/.agents/skills/<skill-name>
# Copy SKILL.md and any bundled resources into it

# Step 2: Ensure target directories exist
mkdir -p ~/.claude/skills
mkdir -p ~/.cursor/skills

# Step 3: Create symlinks (relative paths for portability)
ln -sfn ../../.agents/skills/<skill-name> ~/.claude/skills/<skill-name>
ln -sfn ../../.agents/skills/<skill-name> ~/.cursor/skills/<skill-name>
```

The relative symlink `../../.agents/skills/<name>` works because both `~/.claude/skills/` and `~/.cursor/skills/` are two levels deep from `~`.

### 4. Verify and Report

```bash
# Confirm file exists
head -5 ~/.agents/skills/<skill-name>/SKILL.md

# Confirm symlinks
ls -la ~/.claude/skills/<skill-name>
ls -la ~/.cursor/skills/<skill-name>
```

Report to the user:
- Where the skill was saved (full path)
- Which symlinks were created (for user-level)
- The skill name and description extracted from frontmatter

## Edge Cases

- **Skill already exists at target**: Ask the user whether to overwrite before proceeding.
- **Bundled resources**: If the source is a directory with `scripts/`, `references/`, or `assets/` subdirectories, copy the entire directory tree — not just SKILL.md.
- **Missing target dirs**: Always `mkdir -p` before writing or linking. `~/.cursor/skills/` may not exist yet.
- **Broken symlinks**: If a symlink already exists but points to a missing target, `ln -sfn` will replace it — this is fine and expected.
