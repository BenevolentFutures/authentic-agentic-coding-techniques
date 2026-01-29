# Authentic Workspace

| Repo | Purpose |
|------|---------|
| `Authentic_React` | React Native client |
| `Authentic_Backend` | Current backend (active) |
| `authentic_v2` | Deprecated Next.js backend - reference only |

Each repo has its own CLAUDE.md with detailed context. When you run from a child repo, both that repo's CLAUDE.md and this parent file are loaded—child instructions take precedence.

## About Authentic

Authentic is a social media app focused on ephemeral photos (videos coming soon). Key concepts:

- **Glimpse**: Prompted photo (BeReal-style) - core content primitive
- **Home screen**: Face bubbles of closest friends; tap to open full-screen content viewer
- **Philosophy**: Connect users with people they care about—no discovery, no promotion

## Context

Early-stage startup with small team. Prioritize simplicity—avoid over-engineering. Ship quality core software efficiently.

**Push back like a senior engineer.** If a request could cause bugs, side effects, technical debt, or architectural problems—say so directly. Don't just execute questionable instructions; flag concerns and propose better alternatives.

## Project-Specific Conventions

### Field Naming (Critical)

- **`followerCount`** = people who follow this user
- **`followingCount`** = people this user follows
- **NEVER USE**: `followersCount`, `followsCount`, `follows`

### Expo Port Convention

MainRepos and SecondaryRepos may run simultaneously. To avoid port conflicts:
- **MainRepos** → use default port 8081 (`npx expo run:ios`)
- **SecondaryRepos** → use port 8082 (`npx expo run:ios --port 8082`)

## Documentation

**Blueprints submodule (primary source of truth):** Each active repo includes a `blueprints/` git submodule that contains the authoritative, shared system specification for Authentic. Start there when you need to understand product intent, architecture, types, APIs, workflows, or cross‑project conventions. Use `blueprints/README.md` as the index for navigation. Keep the submodule initialized and up to date; changes to core behavior or shared interfaces should be reflected in Blueprints so all projects stay aligned.

**Read `blueprints/README.md` first** — use Quick Navigation to find relevant docs.

**CLAUDE.md and AGENTS.md are synonymous:** In this workspace, `CLAUDE.md` is a symlink to `AGENTS.md`. Treat guidance in either file as identical and interchangeable.

### Three-Tier Documentation Hierarchy

Authentic uses three tiers of documentation:

1. **Blueprints** (`blueprints/`): Human‑written, gold‑standard architectural and system documentation. This is the canonical reference and should be updated when system‑level truths change.
2. **Docs** (`docs/`): Persistent supporting documentation that should remain accurate over time, but is not at the same criticality as Blueprints (for example, user guides, runbooks, or stable component details).
3. **Notes** (`notes/`): Temporary, work‑in‑progress material. By default, new decisions, discoveries, and conversation summaries land here first. Promote content upward to Docs or Blueprints once it is stable and broadly useful.

**Notes naming**: `aut-[storyID]-[username]-[name].md`
**Username codes**: ATN (Atin), POL (Polly), ANB (Anubhav)

## Full-Stack Awareness

When working on features, consider both sides. Client changes often need corresponding API work, and API changes may affect the client. Check the other repo when relevant—you have visibility into both.

- The backend repo `Authentic_Backend` is available for researching API endpoints, database schema, or validating integration points.
- The client repo `Authentic_React` is available for understanding how the API is consumed.

## Communication Style

When providing feedback with multiple items (suggestions, changes, issues), use a **numbered list**. This allows quick responses like "do 1, 3, and 4" or "skip 2".

### Recognizing Thrashing

If you notice repeated failed attempts at the same bug, circular debugging, or user frustration, pause and acknowledge it directly:

> "It seems like this isn't going the way we intended. We might be thrashing here."

Then suggest starting a fresh agent with a **handoff prompt** containing:

1. **Actions taken**: What we tried and why it didn't work
2. **Current state**: Relevant file states, error messages, test output
3. **Observed behavior**: What's actually happening vs. expected
4. **Possible directions** (non-exhaustive): A few hypotheses to explore, framed openly—not as "the answer"

Encourage the new agent to approach the problem from first principles rather than continuing down the same paths. Fresh context without accumulated assumptions often breaks through stuck situations.

## Parallelizing Work

You can spawn up to 10 agents simultaneously. When a task is parallelizable, divide it into up to 10 approximately equal **buckets of work** (by complexity/effort, not file count).

**Proactively suggest parallelization** when you recognize a task that fits these patterns:

- Fixing multiple failing tests
- Adding tests for components/modules
- Updating multiple independent files with similar changes
- Any batch operation on independent items

If the user hasn't explicitly requested parallel execution, suggest it: "This looks like a good candidate for parallel execution—want me to split this across multiple agents?"

**Example:** "Fix all 15 failing tests" → spawn 10 agents, each handling ~1-2 tests based on complexity, not just distributing 15/10 files.

### Managing Background Tasks

When using `run_in_background: true` with Bash or Task tools, background tasks send completion notifications that must be acknowledged. **Failure to acknowledge leaves dangling notifications that require manual dismissal (Esc key).**

**Rules for background tasks:**

1. **Always acknowledge completions** — When a background task completes, call `TaskOutput` with the task ID to formally close it out:
   ```
   TaskOutput(task_id: "abc123", block: false, timeout: 1000)
   ```

2. **Acknowledge even if you got results another way** — If a background agent writes to a file and you read that file directly, you still need to call `TaskOutput` to clear the notification.

3. **Track task IDs** — When launching background tasks, note the task IDs from the tool results so you can acknowledge them later.

4. **Batch acknowledgment** — If multiple background tasks complete, acknowledge all of them before proceeding:
   ```
   TaskOutput(task_id: "task1", block: false, timeout: 1000)
   TaskOutput(task_id: "task2", block: false, timeout: 1000)
   ```

**Common mistake:** Reading output files from background agents without calling `TaskOutput`. The files contain the results, but the task notification remains pending until explicitly acknowledged.

## Git Workflow

- **All PRs target `dev` branch** (never `prod` directly, except hotfixes)
- Branch naming: `aut-[storyID]-description` or `feature/description`
- **Merging to `prod` requires explicit user approval** — this triggers immediate builds and risks breaking production clients

## Linear Ticket Defaults

When creating Linear issues, use these defaults unless specified otherwise:

| Field        | Default       |
| ------------ | ------------- |
| **Cycle**    | Current cycle |
| **Estimate** | Small (S)     |
| **Priority** | Medium (3)    |
| **Status**   | To Do         |

**Workflow:**

- Always show the full ticket text and get confirmation before creating
- Keep tickets concise—no walls of text
- If a prompt is involved, save it as a separate markdown file in `notes/` and reference it in the ticket (don't embed long prompts in the ticket body)

**Signing comments:** Always sign Linear comments with `— Claude Atin` at the end. This identifies agent-generated comments while associating them with Atin's account.

## Testing

**Unit test fixes:** When asked to fix failing unit tests, first understand why they failed. Treat failures as strong signals of incorrect logic, not just brittle tests. If you conclude the root cause is production/business logic rather than the test itself, stop and bring it to the operator's attention before changing tests.

**Test runs:** Whenever you run tests, always report the number of failing tests in your final output; this is high‑value signal for the operator.

## Mobile Testing & Validation

**Be proactive.** When you make UI changes, API changes that affect the client, or fix bugs—validate them yourself using the simulator/emulator. Don't just say "I made the changes." Actually verify they work. Screenshot the result.

**For detailed guidance**, run `/authentic-simulator-guide` - covers tool selection, app navigation, visual indicators, test accounts, notification types, common test flows, and gotchas.

**Quick reference:**
- **iOS Simulator** → use `ios-simulator-mcp` (complete element tree)
- **Android Emulator** → use `mobile-mcp` (do NOT use for iOS)
- See `Authentic_React/docs/mobile-mcp-comparison.md` for technical details on why

**Cross-user testing:** For bugs involving synchronization between users (e.g., User A's screen updating when User B takes an action), use **both simulators simultaneously**:
1. iOS Simulator: Log in as User A
2. Android Emulator: Log in as User B
3. Perform the cross-user interaction and validate both sides

This is the proper way to validate multi-user scenarios—don't skip this just because it requires coordinating two devices.

## Safety Rules

**NEVER execute these commands without explicit user approval:**

```bash
# File deletion
rm -rf, rm -f, find . -delete

# Git destructive operations
git push --force, git reset --hard

# Production deployments (triggers immediate builds, risks breaking clients)
git push origin prod, git merge ... into prod

# Package management
npm ci --force, npm cache clean --force

# Prisma destructive commands
npx prisma db push --force-reset
npx prisma migrate reset
npx prisma db seed
npx prisma migrate deploy --force
```

**How to get approval:** Before running any destructive command, STOP and ask the user explicitly: "This command will [describe impact]. Do you want me to proceed?" Wait for a clear "yes" or approval before executing. Do not assume approval from general task instructions.

## Common Gotchas

- **Linear**: Story IDs (AUT-123) come from Linear. Branch names should match: `aut-123-description`.
- **Linear Status "Ready for Review" vs "In Review"**: When a PR is complete and waiting for human review, set status to **"Ready for Review"** (NOT "In Review"). "In Review" means a human is actively reviewing right now—agents should NEVER set this status. This is a common mistake.
- **Media URLs**: Cloudflare Images uses image IDs (not full URLs). The CDN URL is constructed at runtime. See `blueprints/32_Media_URL_Architecture.md`.

## Self-Improvement

If you encounter confusion, unexpected complexity, or need extensive searching to understand something, suggest adding a note to this CLAUDE.md file. This saves time across the entire project: every future session (yours and other agents) reads this file first. A 30-second addition here can save time across the team of many agents and humans repeatedly looking into the same thing.
