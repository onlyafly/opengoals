---
name: add-goal
description: Create a new goal note in the Goals/ folder with correct frontmatter, filename prefix, and folder placement. Optionally wire up parent/child links. Invoke when asked to add, create, or track a new goal, subgoal, objective, or key result.
argument-hint: "[goal title]"
allowed-tools: Bash, Read, Write, Edit
---

You are helping add a new goal note to the Goals/ folder.

Refer to the `/goals-schema` skill for the full frontmatter schema, filename prefix rules, folder mapping, and period formats.

## Today's date

!`date +%Y-%m-%d`

## Existing goals (for parent_goal selection)

!`mdquery query 'SELECT f.filename, fm.value AS title FROM files f JOIN frontmatter fm ON f.id = fm.file_id WHERE f.path LIKE "%/Goals/%" AND fm.key = "title" ORDER BY f.filename'`

---

## Step 1 — Gather information

Use `$ARGUMENTS` as the goal title if provided. Otherwise ask.

If the user hasn't provided the details inline, ask for:

- **Title** — the clean goal name (no prefix) — use `$ARGUMENTS` if given
- **Kind** — `objective`, `key result`, `goal`, or `subgoal` (if unsure, default to `goal`)
- **Timescale** — `annual`, `quarterly`, `monthly`, `weekly`, or `someday` (no committed time horizon)
- **Type** — `personal` or `company`
- **Period** — e.g. `2026`, `2026-Q1`, `2026-03`, `2026-W13` (today's date above can help infer this). Omit for `someday` goals, or set to a target period like `2027-Q4` if known.
- **Status** — `Not Started`, `In Progress`, `Done`, or `Cancelled` (default: `Not Started`)
- **Owner** — person accountable for this goal
- **Theme** — optional grouping label (e.g. "Compound Knowledge", "Keep Centered Body & Soul")
- **Parent goal** — wikilink to a parent goal file, if this is a child goal (choose from the existing goals list above if applicable)
- **Metrics** — target description, e.g. "Retention rate exceeds 90%" (optional)
- **Metric fields** — `metric_unit`, `metric_initial`, `metric_current`, `metric_target` (optional)
- **Due date** — `YYYY-MM-DD` (optional)

---

## Step 2 — Determine filename and folder

Use the prefix and folder mapping from the `/goals-schema` skill.

Full filename = `{prefix}{title}.md`
Full path = `Goals/{Folder}/{filename}`

---

## Step 3 — Create the note

Write the file at `Goals/{Folder}/{filename}` with this structure:

```markdown
---
title: {title}
type: {type}
kind: {kind}
timescale: {timescale}
status: {status}
owner: {owner}
period: {period}
theme: {theme}                    # omit if not set
parent_goal: "[[Parent Goal filename without .md]]"  # omit if not set
child_goals: []
metrics: {metrics}                # omit if not set
metric_unit: {metric_unit}        # omit if not set
metric_initial: {metric_initial}  # omit if not set
metric_current: {metric_current}  # omit if not set
metric_target: {metric_target}    # omit if not set
due_date: {due_date}              # omit if not set
archived: false
---

# {title}
```

Omit any optional fields that were not provided — do not write empty or null values.

---

## Step 4 — Update parent goal (if applicable)

If a `parent_goal` was set:

1. Read the parent goal file.
2. Check its `child_goals` list.
3. If this new goal is not already listed, add a wikilink to it (using the filename without `.md`).

---

## Step 5 — Reindex

Run `/goals-index` to pick up the new file.

---

## Step 6 — Confirm

Show:
- The full path of the file created
- The frontmatter as written
- Any parent goal update made
