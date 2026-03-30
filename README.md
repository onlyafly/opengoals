# OpenGoals

A Claude Code plugin for managing goals in an Obsidian vault. Tracks personal and company goals (OKRs) using structured markdown notes with YAML frontmatter, queried via [mdquery](https://github.com/eristoddle/mdquery).

## Requirements

- [mdquery](https://github.com/eristoddle/mdquery) installed globally via pipx
- An Obsidian vault with a `Goals/` folder (run `/og:init` to set one up)

## Usage

Load the plugin when launching Claude Code from your vault:

```sh
claude --plugin-dir ./opengoals
```

## Skills

| Skill | Description |
|---|---|
| `/og:init` | First-time setup — creates `Goals/` subfolders, installs mdquery, builds the index |
| `/og:add` | Create a new goal note with correct frontmatter and folder placement |
| `/og:update` | Record progress, change status, update fields, or archive a goal |
| `/og:summary` | Show all active goals grouped by type and timescale, with metrics and due date highlights |
| `/og:cleanup` | Audit the Goals/ folder — validate fields, fix prefixes, repair links, archive goals |
| `/og:index` | Rebuild the mdquery index after adding, renaming, or moving notes |

`/og:schema` is an internal reference skill used by the others — not intended to be invoked directly.

## Goal Structure

Each goal is a markdown file in `Goals/{timescale}/` with frontmatter fields including `title`, `type`, `kind`, `timescale`, `status`, `owner`, `period`, and optional metric tracking fields. See `skills/schema/SKILL.md` for the full schema.

Goals are organized into four timescale folders:

```
Goals/
  Annual/
  Quarterly/
  Monthly/
  Weekly/
```

Each folder has an `Archive/` subfolder for completed or cancelled goals.
