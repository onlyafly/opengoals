---
name: goals-cleanup
description: Perform housekeeping on all goal notes in the Goals/ folder — validate required fields, fix filename prefixes, sync title/H1 headings, check timescale/folder alignment, repair broken parent/child links, and move archived goals. Invoke when asked to clean up, audit, or tidy the goals system.
disable-model-invocation: true
context: fork
effort: medium
model: sonnet
allowed-tools: Bash, Read, Edit, Glob
---

Perform housekeeping on all goal notes in the Goals/ folder. Work through the steps below in order, making fixes as you go and keeping a running log of every change made and every issue flagged.

Refer to the `/goals-schema` skill for the full frontmatter schema, valid field values, filename prefix rules, folder mapping, and archiving rules.

The vault root is derived from the skill directory. `mdquery` is installed globally — call it directly.

```bash
VAULT="$(cd "${CLAUDE_SKILL_DIR}/../../../" && pwd)"
```

---

## Step 0 — Fresh index

First, rebuild the mdquery index from scratch to ensure you're working from current data:

```bash
VAULT="$(cd "${CLAUDE_SKILL_DIR}/../../../" && pwd)"
rm "$VAULT/.mdquery/index.db" && mdquery index "$VAULT" --recursive --full
```

Then load all goal frontmatter:

```bash
VAULT="$(cd "${CLAUDE_SKILL_DIR}/../../../" && pwd)"
MDQUERY=mdquery
"$MDQUERY" query 'SELECT f.path, f.filename, fm.key, fm.value FROM files f JOIN frontmatter fm ON f.id = fm.file_id WHERE f.directory LIKE "%Goals%" ORDER BY f.path, fm.key'
```

Build a working model of every goal (path, filename, and all frontmatter fields) to use throughout the steps below.

---

## Step 1 — Validate required fields

For each goal, check that all required fields are present and non-empty:
`title`, `type`, `timescale`, `status`, `owner`, `period`, `archived`

Flag any goal missing a required field. Do not auto-fix missing fields — list them for manual attention.

Valid values:
- `type`: `personal` or `company`
- `timescale`: `annual`, `quarterly`, `monthly`, `weekly`
- `status`: `Not Started`, `In Progress`, `Done`, `Cancelled`
- `kind` (if set): `objective`, `key result`, `goal`, `subgoal`
- `archived`: `true` or `false`

Flag any field with an invalid value.

---

## Step 2 — Fix filename prefix

The filename prefix must match the goal's `kind` per the `/goals-schema` skill.

For each goal where the prefix doesn't match (or is missing), rename the file. After renaming, update all wikilinks across the Goals/ folder that reference the old filename — check `parent_goal` and `child_goals` fields in other goal notes, and any inline wikilinks in note bodies.

---

## Step 3 — Fix title frontmatter and H1 heading

The `title` frontmatter field and the H1 heading (`# ...`) in each goal note should match the filename's title — that is, the filename without the prefix and without the `.md` extension.

For example, `KR— Weekly Active Users Exceed 500.md` should have:
- `title: Weekly Active Users Exceed 500`
- `# Weekly Active Users Exceed 500`

Fix any mismatches. If the file was just renamed in Step 2, re-derive the expected title from the new filename.

---

## Step 4 — Check timescale vs folder

Each goal's `timescale` field should match its parent folder per the `/goals-schema` skill.

Flag any mismatch. Do not auto-move — list for manual attention, since a mismatch may be intentional.

---

## Step 5 — Check parent/child link consistency

For every goal with `parent_goal` set: verify the named parent file exists and that the parent's `child_goals` list includes a link back to this goal.

For every goal with `child_goals` set: verify each named child file exists and that each child's `parent_goal` points back to this goal.

**Auto-fix one-sided links:** if the target file exists but the reciprocal link is missing, add it. Specifically:
- If a child has `parent_goal: "[[X]]"` but X's `child_goals` doesn't include this child, add the child to X's `child_goals`.
- If a parent has `"[[Y]]"` in `child_goals` but Y's `parent_goal` is not set or doesn't match, set Y's `parent_goal` to point back to this parent.

**Flag broken links:** if the target file doesn't exist at all, list it for manual attention — do not auto-fix.

---

## Step 6 — Move archived goals

For each goal where `archived: true` that is **not** already in an `Archive/` subfolder, move the file to the `Archive/` subfolder within its current timescale folder (e.g. `Goals/Annual/Archive/`). Create the `Archive/` folder if it doesn't exist.

After moving, update any wikilinks in other goal notes that referenced the moved file.

---

## Step 7 — Reindex

Rebuild the index to reflect all renames and moves made during cleanup:

```bash
VAULT="$(cd "${CLAUDE_SKILL_DIR}/../../../" && pwd)"
rm "$VAULT/.mdquery/index.db" && mdquery index "$VAULT" --recursive --full
```

---

## Output

Produce a clean report:

**Changes made** (auto-fixed):
- List each rename, title/H1 fix, added bidirectional link, and archive move

**Issues flagged** (require manual attention):
- Missing or invalid required fields
- Timescale/folder mismatches
- Broken links to non-existent goal files

If everything is clean, say so.
