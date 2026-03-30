---
name: goals-summary
description: Summarize current goals and their status. Shows all active goals grouped by type and timescale, with progress metrics and due date highlighting. Invoke when asked for a goal overview, status check, or progress summary.
allowed-tools: Bash
context: fork
agent: Explore
model: haiku
---

## Today's date

!`date +%Y-%m-%d`

## Goal data (pre-loaded)

!`cd "${CLAUDE_SKILL_DIR}/../../../" && mdquery query 'SELECT f.filename, fm.key, fm.value FROM files f JOIN frontmatter fm ON f.id = fm.file_id WHERE f.directory LIKE "%Goals%" AND fm.key IN ("title","type","kind","timescale","status","period","due_date","parent_goal","archived","metrics","metric_target","metric_current","metric_initial","metric_unit","theme") ORDER BY f.filename, fm.key'`

---

From the data above, reconstruct each goal's fields (title, type, kind, timescale, status, period, due_date, parent_goal, theme, metrics, metric_unit, metric_initial, metric_current, metric_target) and present them grouped by **type** (Company / Personal) and sorted by **timescale** (Weekly → Monthly → Quarterly → Annual). Within each group, sort by status: Not Started first (needs attention), then In Progress, then Done/Cancelled. Exclude archived goals (where `archived = true`) and exclude someday goals (where `timescale = someday`).

**Parent/child handling:** A goal with a `parent_goal` value is a child — render it as an indented row directly beneath its parent rather than as a standalone row. A child goal should only appear indented under its parent — never as a standalone row. Goals with no `parent_goal` are top-level and render as normal rows.

Output format:

---

## Goal Summary

### Company Goals

**[Level] — [Period]**

| Goal | Theme | Kind | Status | Progress & Metrics |
|---|---|---|---|---|
| [title] | [theme if set] | [kind if set] | [status] | [progress string — see below] |
| ↳ [child title] | [theme if set] | [kind if set] | [status] | [progress string] |

### Personal Goals

**[Level] — [Period]**

| Goal | Theme | Kind | Status | Progress & Metrics |
|---|---|---|---|---|
| [title] | [theme if set] | [kind if set] | [status] | [progress string — see below] |

---

**Progress & Metrics string rules:**
- Always show the `metrics` text if set.
- If `metric_target` is set and numeric, append the numbers on a new line separated by a divider (use `<br> ———— <br>` in the table cell): `[initial] → **[current]** → [target][unit]`, replacing any blank values with `?`. Bold the current value. Example: `? → **28** → 35%`
- If neither is set: leave blank

**Due date highlighting** (compare against today's date injected above):
- If a goal is overdue (due_date < today) and not Done/Cancelled: prefix the title cell with 🔴
- If a goal is due within 14 days and not Done/Cancelled: prefix the title cell with 🟡
- Apply highlighting to both parent rows and child rows independently.

End with a one-line summary: total goals, how many are Not Started, In Progress, Done, Cancelled.

The `title` frontmatter field is the clean name without the filename prefix — use that for display in the summary table.

Do not read individual goal files unless the user asks for more detail on a specific goal.
