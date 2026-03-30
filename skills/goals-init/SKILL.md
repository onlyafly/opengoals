---
name: goals-init
description: "Set up the Goals system in this vault for the first time — creates any missing Goals/ subfolders, installs mdquery if needed, and builds the index. Safe to re-run: never deletes or modifies existing files or folders."
disable-model-invocation: true
effort: high
allowed-tools: Bash
---

This skill is fully idempotent. It only creates things that are missing — it never deletes, overwrites, or modifies any existing file or folder. Re-running it on an already-configured vault is safe.

## Current state (pre-loaded)

### Vault root
!`cd "${CLAUDE_SKILL_DIR}/../../../" && pwd`

### Existing goal files
!`VAULT="$(cd "${CLAUDE_SKILL_DIR}/../../../" && pwd)"; find "$VAULT/Goals" -name "*.md" 2>/dev/null | sort || echo "(none)"`

### Goals folder structure
!`VAULT="$(cd "${CLAUDE_SKILL_DIR}/../../../" && pwd)"; find "$VAULT/Goals" -type d 2>/dev/null | sort || echo "(Goals/ folder does not exist yet)"`

### mdquery status
!`which mdquery 2>/dev/null && echo "installed at $(which mdquery)" || echo "NOT installed"`

### mdquery index status
!`VAULT="$(cd "${CLAUDE_SKILL_DIR}/../../../" && pwd)"; test -f "$VAULT/.mdquery/index.db" && echo "index.db exists" || echo "index.db does not exist"`

---

Review the pre-loaded state above before proceeding. For each step, explicitly note what already exists and will be skipped, and what is missing and needs to be created.

**Important:** If existing goal files are shown above, confirm before proceeding that this skill will not touch them. Only folders and index infrastructure are ever created — never modified or removed.

---

## Step 1 — Create any missing Goals/ subfolders

The required folder structure is:

```
Goals/
  1-Weekly/
  2-Monthly/
  3-Quarterly/
  4-Annual/
  8-Archived/
  9-Someday/
```

`mkdir -p` is safe to run unconditionally — it creates only what is missing and leaves existing directories and their contents untouched:

```bash
VAULT="$(cd "${CLAUDE_SKILL_DIR}/../../../" && pwd)"
mkdir -p \
  "$VAULT/Goals/1-Weekly" \
  "$VAULT/Goals/2-Monthly" \
  "$VAULT/Goals/3-Quarterly" \
  "$VAULT/Goals/4-Annual" \
  "$VAULT/Goals/8-Archived" \
  "$VAULT/Goals/9-Someday"
```

After running, list which folders were already present and which were newly created (compare against the pre-loaded state above).

---

## Step 2 — Install mdquery (if not installed)

If mdquery is already installed (see pre-loaded state), skip this step entirely.

Otherwise, install pipx and then mdquery from GitHub:

```bash
brew install pipx
pipx ensurepath
pipx install git+https://github.com/eristoddle/mdquery.git
```

Verify it's on PATH:

```bash
which mdquery
```

---

## Step 3 — Build the index

If `index.db` already exists (see pre-loaded state), ask whether to rebuild it or skip. A rebuild is safe — it only updates the search index, it does not read from or write to any vault `.md` files.

To build or rebuild:

```bash
VAULT="$(cd "${CLAUDE_SKILL_DIR}/../../../" && pwd)"
mdquery index "$VAULT" --recursive --full
```

Report how many files were indexed.

---

## Step 4 — Smoke test

Confirm the index can query goal files:

```bash
mdquery query "SELECT filename, directory FROM files WHERE path LIKE '%/Goals/%' ORDER BY filename"
```

If Goals/ contains existing files, they should appear here. If the folders are empty, an empty result is expected and fine.

---

## Step 5 — Report

Summarise what happened:
- Folders: list which were created and which already existed
- mdquery: installed, already present, or skipped
- Index: built, rebuilt, or skipped
- Existing goal files found (if any) — confirm they were not touched

Suggest next steps:
- `/add-goal` — create a goal
- `/goals-summary` — view goals once some exist
- `/goals-cleanup` — audit and tidy goals at any time
