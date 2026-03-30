---
name: goals-schema
description: Reference schema for the Goals/ folder — frontmatter fields, folder structure, filename naming conventions, period formats, archiving rules, and mdquery queries for goals. Use whenever creating, updating, querying, cleaning up, or summarizing goal notes.
user-invocable: false
---

# Goals Schema Reference

`Goals/` is the single source of truth for all work goals — both personal goals and company-level goals (OKRs).

## Vault paths

The vault root is the parent of the plugin directory. Derive it in any bash command with:

```bash
VAULT="$(cd "${CLAUDE_SKILL_DIR}/../../../" && pwd)"
```

`mdquery` is installed globally — call it directly without a path.

## Folder Structure

```
Goals/
  1-Weekly/         — Weekly goals (e.g. 2026-W13)
    Archive/        — Archived weekly goals
  2-Monthly/        — Monthly goals (e.g. 2026-03)
    Archive/        — Archived monthly goals
  3-Quarterly/      — Quarterly goals (e.g. Q1 2026)
    Archive/        — Archived quarterly goals
  4-Annual/         — Yearly goals
    Archive/        — Archived annual goals
  9-Someday/        — Someday/maybe goals with no committed time horizon
    Archive/        — Archived someday goals
```

## Filename Naming Conventions

Each goal file must begin with a prefix that reflects its `kind`:

| `kind` | Filename prefix |
|---|---|
| `objective` | `Objective— ` |
| `key result` | `KR— ` |
| `goal` | `Goal— ` |
| `subgoal` | `Subgoal— ` |
| *(unset)* | `Goal— ` |

Examples:
- `Objective— Improve Developer Productivity.md`
- `KR— Weekly Active Users Exceed 500.md`
- `Goal— Launch Self-Service Analytics Platform.md`
- `Subgoal— Define Data Access Policy for Analytics.md`

The `title` frontmatter field is the clean name **without** the prefix.

## Frontmatter Schema

```yaml
---
title: <goal title>
type: personal | company
kind: objective | key result | goal | subgoal  # optional
timescale: annual | quarterly | monthly | weekly | someday
status: Not Started | In Progress | Done | Cancelled
owner: <name>
period: <e.g. 2026, 2026-Q1, 2026-03, 2026-W13>
theme: <grouping label, e.g. "Compound Knowledge">  # optional
parent_goal: "[[Link to parent goal]]"  # optional
child_goals: []                          # optional, list of wikilinks
metrics: <target or description>         # optional
metric_unit: <%|flows|users|...>         # optional
metric_initial: <starting value>         # optional
metric_current: <latest value>           # optional
metric_target: <target value>            # optional
due_date: YYYY-MM-DD                     # optional
tracker_link: <URL>                      # optional
archived: false
---
```

### Field Reference

| Field | Required | Valid values / notes |
|---|---|---|
| `title` | Yes | Clean goal title, no filename prefix |
| `type` | Yes | `personal` or `company` |
| `kind` | No | `objective`, `key result`, `goal`, `subgoal` |
| `timescale` | Yes | `annual`, `quarterly`, `monthly`, `weekly`, `someday` |
| `status` | Yes | `Not Started`, `In Progress`, `Done`, `Cancelled` |
| `owner` | Yes | Person accountable |
| `period` | No | ISO-ish format — see Period Formats below. Omit for `someday` goals, or set to a target period like `2027-Q4` |
| `theme` | No | Grouping label / strategic theme (e.g. `Compound Knowledge`) |
| `parent_goal` | No | Wikilink to parent goal note |
| `child_goals` | No | List of wikilinks to child goal notes |
| `metrics` | No | Measurable target or key result description |
| `metric_unit` | No | Unit of measurement (e.g. `%`, `flows`, `users`) |
| `metric_initial` | No | Starting value at goal creation |
| `metric_current` | No | Latest known value |
| `metric_target` | No | Success threshold / target value |
| `due_date` | No | `YYYY-MM-DD` |
| `tracker_link` | No | URL to an external tracking tool (e.g. Garmin, Strava, Notion, a dashboard) |
| `archived` | Yes | `true` or `false` |

### Period Formats

| Timescale | Example |
|---|---|
| Annual | `2026` |
| Quarterly | `2026-Q1` |
| Monthly | `2026-03` |
| Weekly | `2026-W13` |

### Timescale → Folder Mapping

| `timescale` | Folder |
|---|---|
| `weekly` | `Goals/1-Weekly/` |
| `monthly` | `Goals/2-Monthly/` |
| `quarterly` | `Goals/3-Quarterly/` |
| `annual` | `Goals/4-Annual/` |
| `someday` | `Goals/9-Someday/` |

## Archiving a Goal

1. Set `archived: true` in frontmatter
2. Move the file to the `Archive/` subfolder within its timescale folder (e.g. `Goals/Annual/Archive/`)
3. Update any wikilinks in other goal notes that referenced the old path

## mdquery Queries

Use single quotes around SQL queries and `<>` (not `!=`) for inequality.

```bash
VAULT="$(cd "${CLAUDE_SKILL_DIR}/../../../" && pwd)"

# All active (non-archived) goals
mdquery query "SELECT files.filename FROM files JOIN frontmatter ON files.id = frontmatter.file_id WHERE frontmatter.key = 'archived' AND frontmatter.value = 'false' AND files.path LIKE '%/Goals/%'"

# All goal frontmatter (for cleanup/summary work)
mdquery query 'SELECT f.path, f.filename, fm.key, fm.value FROM files f JOIN frontmatter fm ON f.id = fm.file_id WHERE f.directory LIKE "%Goals%" ORDER BY f.path, fm.key'

# Goals in a specific period
mdquery query "
SELECT f.filename
FROM files f
JOIN frontmatter s ON f.id = s.file_id AND s.key = 'status'
JOIN frontmatter p ON f.id = p.file_id AND p.key = 'period'
WHERE s.value = 'In Progress' AND p.value = '2026-Q1'
AND f.path LIKE '%/Goals/%'"

# All company goals by status
mdquery query "
SELECT s.value AS status, COUNT(*) AS count
FROM files f
JOIN frontmatter t ON f.id = t.file_id AND t.key = 'type'
JOIN frontmatter s ON f.id = s.file_id AND s.key = 'status'
WHERE t.value = 'company' AND f.path LIKE '%/Goals/%'
GROUP BY s.value"

# Goals linked to a specific parent
mdquery query "
SELECT files.filename
FROM files JOIN frontmatter ON files.id = frontmatter.file_id
WHERE frontmatter.key = 'parent_goal'
AND frontmatter.value LIKE '%Parent Goal Title%'
AND files.path LIKE '%/Goals/%'"

# Completion rate by timescale
mdquery query "
SELECT l.value AS timescale,
       SUM(CASE WHEN s.value = 'Done' THEN 1 ELSE 0 END) AS done,
       COUNT(*) AS total
FROM files f
JOIN frontmatter l ON f.id = l.file_id AND l.key = 'timescale'
JOIN frontmatter s ON f.id = s.file_id AND s.key = 'status'
WHERE f.path LIKE '%/Goals/%' AND f.path NOT LIKE '%/Archive/%'
GROUP BY l.value"
```
