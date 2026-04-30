---
description: "Token and context efficiency rules for agentic tool use — file reads, search strategy, shell output, and sub-agent delegation"
applyTo: "**"
---

# Token Efficiency

## File Read Discipline

- Use `view_range` on any file you expect to exceed 100 lines — never read a large file whole
- Use `grep` or `glob` to confirm a file is relevant before opening it
- When a file is truncated, read the specific section with `view_range` — do not re-read from the top

```
# CORRECT: locate the symbol first, then read only that section
grep "def calculate_tax" src/         # find which file and line
view src/billing/tax.py [42, 85]      # read only that section

# WRONG: open the whole file to find one function
view src/billing/tax.py               # 600-line file, most irrelevant
```

## Search Efficiency

- Set `head_limit` on grep to avoid flooding context when many matches are expected
- Use specific glob patterns (`src/**/*.py`) over broad ones (`**/*` or `.`)
- When looking for a symbol definition, search first — open only the files that match

```
# CORRECT: scoped search with a result cap
grep "class OrderService" glob="**/*.py" head_limit=5

# WRONG: unscoped search returns hundreds of matches
grep "class OrderService"
```

## Shell Output Suppression

Suppress verbose output to keep context clean:

- Pass `--quiet` / `-q` where available (`pip install -q`, `npm install --silent`)
- Use `git --no-pager` for all git commands to prevent pager blocking
- Pipe to `| tail -20` to capture only the end of long build or install logs
- Pipe to `| grep -E "(error|warning|FAIL)"` to extract signal from noisy output

```bash
# CORRECT
git --no-pager diff --stat
pip install -q -r requirements.txt 2>&1 | tail -10
npm run build 2>&1 | grep -iE "(error|warn|failed|passed)"

# WRONG
git diff                     # opens pager, blocks
npm install                  # floods context with download progress
```

## Sub-agent Context Isolation

Use sub-agents (task tool) for broad exploration so the main context stays clean:

- Delegate file searches, multi-file reads, and test runs to sub-agents
- After a sub-agent completes, extract only its **conclusions** — never copy its full output into main context
- Use fast/lightweight models for worker sub-agents doing file reads or searches

```
# CORRECT
Sub-agent reads 15 files → returns: "Auth flow is in src/auth/; token
refresh logic is in refresh.py lines 42–88"
Main context receives that one summary line.

# WRONG
Sub-agent output (raw file contents, grep results, full reasoning trace)
pasted back into main context
```

## Strategic Compaction

Compact at logical breakpoints — not mid-task.

**Compact after:**
- Completing exploration, before starting implementation
- Finishing a milestone or resolving a debugging session
- A major context shift (switching from one feature to another)

**Do not compact:**
- Mid-implementation when changes span multiple related files
- While an active error is unresolved and still being investigated
- During multi-file refactoring that needs prior context to stay coherent
