---
name: goals-index
description: Delete the mdquery index and rebuild it from scratch. Use after adding, renaming, or moving any vault notes to ensure queries return current data.
allowed-tools: Bash
---

```bash
VAULT="$(cd "${CLAUDE_SKILL_DIR}/../../../" && pwd)"
rm "$VAULT/.mdquery/index.db" && mdquery index "$VAULT" --recursive --full --force
```

Report the indexing results (files processed, updated, skipped).
