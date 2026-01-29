---
name: ship
description: >
  Full implementation-to-PR workflow with dual-agent code review, consolidation, and self-correction.
  Use to implement features/fixes end-to-end, OR reference specific phases (e.g., Phase 3 dual-agent
  review, Phase 7 PR format) from other workflows. Phases are designed to be composable.
---

# Ship

Implement a task from prompt through PR with dual-agent code review, consolidation, and self-correction.

## When to Use

- **End-to-end workflow**: Implement a feature from ticket to open PR (implement → review → fix → PR)
- **Reference**: Other agents can reference specific phases as examples for their own workflows

## Usage

```
/ship [--worktree] <implementation prompt or task description>
```

**Flags:**
- `--worktree` — Run in an isolated worktree. Main working directory is untouched. Recommended when you have WIP or want guaranteed isolation.

**Examples:**
- `/ship implement the permission flow update from notes/permission-flow-prompt.md`
- `/ship --worktree add user authentication to the API`

---

## Tracking Progress

### Phase Tracking with TodoWrite

Use **TodoWrite** to track phase progress. At the start of the ship process, create todos for all phases:

```
TodoWrite([
  { content: "Phase 0: Understand & Clarify", status: "in_progress", activeForm: "Understanding task" },
  { content: "Phase 1: Branch Setup & Implementation", status: "pending", activeForm: "Implementing" },
  { content: "Phase 2: Commit", status: "pending", activeForm: "Committing changes" },
  { content: "Phase 3: First Round Review", status: "pending", activeForm: "Running first round review" },
  { content: "Phase 4: Consolidate First Round", status: "pending", activeForm: "Consolidating review feedback" },
  { content: "Phase 5: Validate Issues", status: "pending", activeForm: "Validating issues" },
  { content: "Phase 6: Parallel Fixes", status: "pending", activeForm: "Fixing issues" },
  { content: "Phase 7: Open PR", status: "pending", activeForm: "Opening PR" },
  { content: "Phase 8: Post-PR Summary", status: "pending", activeForm: "Summarizing PR status" },
  { content: "Phase 9: Second Round Review", status: "pending", activeForm: "Running second round review" },
  { content: "Phase 10: Second Round Summary", status: "pending", activeForm: "Summarizing second round" },
  { content: "Phase 11: Cleanup", status: "pending", activeForm: "Cleaning up" }
])
```

**Update TodoWrite as you progress:**
- Mark current phase `in_progress` when starting
- Mark phase `completed` when done
- Only one phase should be `in_progress` at a time

### Metadata File

Create a simple metadata file for artifacts at `notes/ship-{identifier}.md`:

```markdown
# Ship: {identifier}

Branch: {branch-name}
PR: #{number} {url}

## Artifacts
- Initial Commit: {hash}
- Claude Review: {file path}
- Codex Review: {file path}
- Fix Commit: {hash}
- Post-Review: {file path}

## Issues Found
- [IMPORTANT] {description} - Fixed in {hash}
- [POTENTIAL] {description} - Deferred
```

This file is for reference only — TodoWrite is the primary progress tracker.

### After Context Compaction

If context is compacted:
1. Check TodoWrite status — the incomplete phases show where you are
2. Read the metadata file if you need artifact references
3. Continue from the first non-completed phase

---

## Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              /ship Flow                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Phase 0: Understand & Clarify                                              │
│     ├─► Update ticket status (In Progress)                                  │
│     ├─► Switch to dev (CRITICAL - before reading any code)                  │
│     └─► Read task, ask clarifying questions if needed                       │
│                                                                             │
│  Phase 1: Branch Setup & Implementation                                     │
│     ├─► Standard: create feature branch (already on dev)                    │
│     └─► Worktree (--worktree): create isolated worktree from dev            │
│            └─► Multi-repo: parallel worktree creation + npm install         │
│                                                                             │
│  Phase 2: Commit                                                            │
│     └─► Stage and commit with conventional format                           │
│                                                                             │
│  Phase 3: First Round Review (async)                                        │
│     ├─► Launch Claude review agent ──┐                                      │
│     └─► Launch Codex review agent ───┴─► Both write to notes/               │
│                                                                             │
│  Phase 4: Consolidate First Round                                           │
│     └─► Merge + deduplicate issues into single list                         │
│                                                                             │
│  Phase 5: Validate Issues                                                   │
│     └─► Main thread marks each: ✅ Confirmed / ❌ False Positive / ❓ Uncertain│
│                                                                             │
│  Phase 6: Parallel Fixes                                                    │
│     └─► Spawn fix agents (bucketed by file to avoid conflicts)              │
│            └─► Revalidate: type-check, lint, test                           │
│                                                                             │
│  Phase 7: Open PR                                                           │
│     └─► Sync with dev, push, create PR via gh                               │
│                                                                             │
│  Phase 8: Post-PR Summary                                                   │
│     └─► Verify no conflicts, all checks pass, output PR status              │
│                                                                             │
│  Phase 9: Second Round Review (async)                                       │
│     ├─► Claude + Codex sanity check on full diff ──┐                        │
│     ├─► Consolidate & validate                     │                        │
│     └─► Fix issues (higher threshold) ─────────────┘                        │
│                                                                             │
│  Phase 10: Second Round Summary                                             │
│     ├─► Output review results                                               │
│     ├─► List all deferred items (from both rounds)                          │
│     └─► Offer to generate follow-up prompt for deferred work                │
│                                                                             │
│  Phase 11: Cleanup (if --worktree)                                          │
│     └─► Ask user: remove worktree / keep                                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Phase 0: Understand & Clarify

### 0.0 Initialize Progress Tracking

1. **Determine identifier:**
   - If `$ARGUMENTS` contains a ticket reference: use that ID
   - Otherwise, derive from the task (e.g., `add-user-auth`)

2. **Create TodoWrite phases** (see Tracking Progress section above)

3. **Create metadata file** at `notes/ship-{identifier}.md`

### 0.1 Update Ticket Status (Optional)

If your project uses a ticket system (Linear, Jira, etc.), mark the ticket as "In Progress".

### 0.2 Switch to Dev (CRITICAL — Standard Mode Only)

**Skip this step entirely if `--worktree` flag is present.** In worktree mode:
- The main working directory must remain untouched (another agent may be working there)
- The worktree will be created directly from `origin/dev` in Phase 1
- This ensures the new feature stands alone, not based on whatever branch happens to be checked out

**For Standard Mode (no --worktree):**

Before reading any code, switch to dev to ensure you're looking at the correct codebase:

```bash
git fetch origin dev
git checkout dev
git pull origin dev
```

If working with multiple repos, do this for ALL repos involved.

**Why this matters:** The session may be on a random branch from previous work. Reading code from the wrong branch leads to incorrect understanding and wasted effort.

### 0.3 Understand the Task

Before writing any code:

1. **Read the task** — read `$ARGUMENTS` and any referenced files thoroughly
2. **Identify unknowns** — what's ambiguous, underspecified, or could go multiple ways?
3. **Ask clarifying questions** — use `AskUserQuestion` tool to resolve unknowns

**Question categories to consider:**
- Scope: "Should this also handle X, or just Y?"
- Approach: "There are two ways to do this: A or B. Which do you prefer?"
- Edge cases: "What should happen when X occurs?"
- Dependencies: "This will require changing Z. Is that acceptable?"
- Testing: "Should I add new tests for this, or just ensure existing tests pass?"
- Priority: "Which of these concerns matters most?"

**Rules:**
- Ask as many questions as necessary — no limit
- Use multiple `AskUserQuestion` calls if needed (each supports up to 4 questions)
- Don't ask obvious questions — use judgment for straightforward decisions
- If the prompt is crystal clear with no ambiguity, skip to Phase 1

---

## Phase 1: Branch Setup & Implementation

### 1.1 Ensure Clean Starting Point

Check if `--worktree` flag is present in `$ARGUMENTS`. If yes, follow **Worktree Mode**. Otherwise, follow **Standard Mode**.

---

#### Standard Mode (no --worktree flag)

Already on dev from Phase 0.2. Create a new branch for this work:

```bash
git checkout -b <branch-name>
```

**Multi-repo work:** If the task spans multiple repositories, create matching branch names in each repo.

---

#### Worktree Mode (--worktree flag present)

Create an isolated worktree to guarantee a clean starting point. **The main working directory is NEVER modified.**

**Why this matters:**
- **Other agents may be working** — The main directory might have another ship operation in progress. Touching it would corrupt their work.
- **You might be on a random branch** — This command could be invoked while the main directory is on a different feature branch.
- **Features must stand alone** — Each feature branch should be based on `dev`, not on other in-progress work.

**Key principle:** Worktrees are ALWAYS created from `origin/dev`, regardless of what branch the main directory is on.

**Step 1: Determine context and repos involved**

```bash
# Check if we're inside a git repo
if git rev-parse --show-toplevel >/dev/null 2>&1; then
    REPOS_PARENT="$(git rev-parse --show-toplevel)/.."
else
    REPOS_PARENT="$(pwd)"
fi

WORKTREE_BASE="$REPOS_PARENT/.worktrees"
```

**Step 2: Check for collisions**

```bash
BRANCH_NAME="<branch-name>"
REPO_PATH="<path-to-repo>"
REPO_NAME="$(basename "$REPO_PATH")"
WORKTREE_PATH="$WORKTREE_BASE/$BRANCH_NAME/$REPO_NAME"

# Check if worktree path exists
if [ -d "$WORKTREE_PATH" ]; then
    echo "WORKTREE_EXISTS"
fi

# Check if branch exists
if git show-ref --verify --quiet "refs/heads/$BRANCH_NAME"; then
    echo "BRANCH_EXISTS"
fi
```

Ask user how to proceed if collision detected.

**Step 3: Create worktrees from origin/dev**

```bash
git -C "$REPO_PATH" fetch origin dev
mkdir -p "$WORKTREE_BASE/$BRANCH_NAME"
git -C "$REPO_PATH" worktree add "$WORKTREE_PATH" -b "$BRANCH_NAME" origin/dev
```

**Step 4: Initialize worktrees**

```bash
cd "$WORKTREE_PATH"
git submodule update --init --recursive
npm install
```

---

### 1.2 Execute the Task

Execute the task described in `$ARGUMENTS`:

1. **Implement** — make the changes, following existing codebase patterns
2. **Validate** — run checks after implementation:
   ```bash
   npm run type-check
   npm run lint
   npm test
   ```
3. Fix any issues before proceeding

**Constraints:**
- No over-engineering — do what's asked, nothing more
- Match existing patterns — consistency over preference
- Keep commits atomic and reviewable

---

## Phase 2: Commit

1. Stage changes: `git add -A`
2. Auto-generate commit message in conventional format:
   ```
   fix|feat|refactor(scope): concise description

   - Key changes as bullet points

   Co-Authored-By: Claude <noreply@anthropic.com>
   ```
3. Commit the changes
4. Update metadata file with commit hash

---

## Phase 3: First Round Review (async)

Launch TWO code reviews in parallel using clear agents. Both agents use the same review criteria from `/code-review` to ensure consistency.

### 3.1 Generate Review Prompt Files

```bash
REVIEW_TS=$(date +%Y%m%d-%H%M)
COMMIT_HASH=$(git rev-parse --short HEAD)
DIFF_FILE="/tmp/ship-review-diff-${REVIEW_TS}.txt"
CLAUDE_PROMPT="/tmp/ship-review-prompt-claude-${REVIEW_TS}.md"
CODEX_PROMPT="/tmp/ship-review-prompt-codex-${REVIEW_TS}.md"
CLAUDE_OUTPUT="notes/ship-review-claude-${REVIEW_TS}.md"
CODEX_OUTPUT="notes/ship-review-codex-${REVIEW_TS}.md"

git diff HEAD~1 > "$DIFF_FILE"
```

Create prompt files for both agents pointing to the diff and specifying output locations.

### 3.2 Launch Reviews in Parallel

**Claude Review:**
```bash
claude -p "Read $CLAUDE_PROMPT and follow the instructions." --dangerously-skip-permissions
```

**Codex Review:**
```bash
codex exec --full-auto --skip-git-repo-check "Read $CODEX_PROMPT and follow the instructions."
```

Use `run_in_background: true` for both. Launch in a single message for parallel execution.

---

## Phase 4: Consolidate First Round

Spawn a clear agent to merge findings into a single deduplicated list:
- Extract all issues from both reviews
- Deduplicate (note which agent(s) reported each)
- Sort by priority: BLOCKER → IMPORTANT → POTENTIAL
- Write to `notes/ship-review-consolidated-{timestamp}.md`

---

## Phase 5: Validate Issues

The main thread validates each issue from the consolidated list:

1. Read the code at the specified location
2. Consider the implementation intent
3. Mark each item:
   - ✅ **Confirmed** — issue is real and should be fixed
   - ❌ **False Positive** — not actually an issue (explain why)
   - ❓ **Uncertain** — needs user input

Update the consolidated file in place with validation status.

---

## Phase 6: Parallel Fixes

### 6.1 Bucket Issues by File

Group confirmed issues to avoid conflicts:
- Same file → same bucket (one agent)
- Different files → can be parallel (separate agents)

### 6.2 Spawn Fix Agents

For each bucket, spawn a clear agent with the list of issues to fix.

### 6.3 Fix Thresholds

- **BLOCKER (✅):** Always fix
- **IMPORTANT (✅):** Fix unless clearly wrong
- **POTENTIAL (✅):** Fix only if trivial

### 6.4 Revalidate

```bash
npm run type-check && npm run lint && npm test
```

### 6.5 Commit Fixes

```
fix: address code review feedback

- [List each fix applied]

Co-Authored-By: Claude <noreply@anthropic.com>
```

---

## Phase 7: Open PR

1. Sync with target branch:
   ```bash
   git fetch origin dev
   git merge origin/dev
   ```

2. Push: `git push -u origin HEAD`

3. Create PR:
   ```bash
   gh pr create --title "..." --body "..."
   ```

   PR format:
   ```
   ## Summary
   {2-3 bullets: what this PR does}

   ## Test Plan
   - [x] Type check passes
   - [x] Lint passes
   - [x] Unit tests pass
   - [ ] {Manual verification steps if applicable}

   ## Review Notes
   {Any deferred items or known limitations}

   🤖 Generated with Claude Code
   ```

---

## Phase 8: Post-PR Summary

Verify PR is ready and output status report including:
- All phases completed
- Test results
- Issues found and fixed
- Deferred items

---

## Phase 9: Second Round Review (async)

Run one more sanity check reviewing the complete PR diff against dev. Same process as Phase 3-6 but with higher fix threshold:

- **BLOCKER (✅):** Always fix
- **IMPORTANT (✅):** Fix only if clearly correct and low-risk
- **POTENTIAL:** Skip unless trivially obvious

---

## Phase 10: Second Round Summary

1. Output review results
2. List all deferred items from both rounds
3. Offer to generate follow-up prompt for deferred work
4. Offer to update ticket status to "Ready for Review"

---

## Phase 11: Cleanup

If `--worktree` was used, ask user whether to remove or keep the worktree.

---

## Failure Handling

- **Implementation unclear:** Ask for clarification before proceeding
- **Tests fail repeatedly:** Summarize attempts and ask for help
- **Single-agent issue:** Still valid — validate it yourself and fix if confirmed
- **Uncertain items (❓):** Ask user for guidance before spawning fix agents
- **Fix agent fails:** Main thread picks up the issue directly
- **Merge conflicts:** Resolve them, or ask for help if complex
- **Context compaction:** Check TodoWrite FIRST, then continue from first incomplete phase

---

Now begin. Ship it.
