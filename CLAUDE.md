# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

The Library is a pure agent application - a meta-skill for distributing skills, agents, and prompts across devices and teams. There is no code to build, compile, or test. The entire runtime is encoded in `SKILL.md` and `cookbook/` markdown files that teach the agent what to do.

This repo is designed to be cloned to `~/.claude/skills/library/`, which makes `/library` available as a global slash command in every Claude Code session.

## No Build/Test Commands

This project has no build system, no tests, no linter, and no dependencies. All "logic" lives in markdown. Modifications are made by editing markdown files.

## Architecture

```
SKILL.md          # Agent brain - commands table, source format rules, GitHub workflow,
                  # typed dependency resolution, target directory logic
library.yaml      # The catalog - references to skills/agents/prompts (pointers, not copies)
cookbook/         # Step-by-step execution guides - one file per command
justfile          # Terminal shortcuts that invoke claude with /library commands
```

**Execution flow:** When a user runs `/library <command>`, the agent reads `SKILL.md` to understand the command, then reads the matching `cookbook/<command>.md` for the exact steps to execute.

**The catalog (`library.yaml`) stores references, not copies.** Items are pulled on demand via `/library use`. The `default_dirs` section maps types (skills/agents/prompts) to default and global install paths.

## Key Conventions

**library.yaml format:**
- 2-space indentation
- Entries kept alphabetically sorted by name within each section
- `requires` field uses typed references: `skill:name`, `agent:name`, `prompt:name`
- Omit `requires` if there are no dependencies

**Source formats supported:**
- Local absolute path: `/path/to/SKILL.md`
- GitHub browser URL: `https://github.com/org/repo/blob/main/path/to/SKILL.md`
- GitHub raw URL: `https://raw.githubusercontent.com/org/repo/main/path/to/SKILL.md`

**Source always points to a specific file, not a directory.** The agent pulls the entire parent directory of that file.

**On `add`:** always `git pull` before editing `library.yaml`, then commit and push after - this keeps the catalog in sync across devices.

## Justfile Commands

```bash
just list                  # List catalog with install status
just use <name>            # Pull a skill into .claude/skills/
just push <name>           # Push local changes back to source
just add "<details>"       # Register a new entry
just sync                  # Re-pull all installed items
just search "<keyword>"    # Find by name or description
just install               # First-time setup
```

All justfile recipes use `--dangerously-skip-permissions` because the agent needs filesystem and git access.

## SKILL.md Variables

After forking and cloning, update the `## Variables` section in `SKILL.md`:
- `LIBRARY_REPO_URL` - your fork's git URL
- `LIBRARY_YAML_PATH` - defaults to `~/.claude/skills/library/library.yaml`
- `LIBRARY_SKILL_DIR` - defaults to `~/.claude/skills/library/`
