---
name: update-goal
description: Update an existing goal note — record metric progress, change status, add a progress note, edit frontmatter fields, or archive a goal. Invoke when asked to update, record progress on, complete, cancel, or archive a goal.
argument-hint: "[goal title or keyword]"
allowed-tools: Bash, Read, Edit
---

You are helping update a goal note in the Goals/ folder.

## Today's date

!`date +%Y-%m-%d`

## Goal search results for "$ARGUMENTS"

!`if [ -n "$ARGUMENTS" ]; then mdquery query "SELECT f.path, f.filename FROM files f WHERE f.path LIKE '%/Goals/%' AND f.filename LIKE '%$ARGUMENTS%'"; else echo "(No goal specified in arguments)"; fi`

---

## Step 1 — Identify the goal

If the search results above found exactly one match, use that file. If multiple matches, ask which one. If no arguments were provided or no match found, ask the user to specify the goal.

Read the matched file to see its current state before making any changes.

---

## Step 2 — Determine what to update

Handle any combination of the following. If the user hasn't specified, ask what they want to change.

### Update metric progress

Update `metric_current` in the frontmatter to the new value.

Optionally add a dated progress note to the body (below the H1 heading):

```markdown
**{YYYY-MM-DD}** — {brief description of progress}
```

### Update status

Change the `status` frontmatter field to one of: `Not Started`, `In Progress`, `Done`, `Cancelled`.

If setting to `Done` or `Cancelled`, ask if the goal should also be archived (see below).

### Add a progress note

Append a dated entry to the body of the note:

```markdown
**{YYYY-MM-DD}** — {description of what happened}
```

Keep it concise — one or two sentences.

### Update other fields

Any frontmatter field can be updated: `due_date`, `owner`, `metrics`, `metric_target`, `metric_unit`, `period`, etc. Apply the change directly.

### Archive a goal

To archive:

1. Set `archived: true` in frontmatter.
2. Move the file to `Goals/8-Archived/`.
3. Update any wikilinks in other goal notes (`parent_goal` and `child_goals` fields) that referenced the old path — check with:

```bash
mdquery query "SELECT f.path FROM files f JOIN frontmatter fm ON f.id = fm.file_id WHERE f.path LIKE '%/Goals/%' AND fm.key IN ('parent_goal','child_goals') AND fm.value LIKE '%$ARGUMENTS%'"
```

---

## Step 3 — Apply changes

Make all edits. For file moves (archiving), rename/move the file and update referencing notes.

---

## Step 4 — Reindex

Run `/goals-index` to reflect any renames or moves.

---

## Step 5 — Confirm

Show a brief summary of every change made.
