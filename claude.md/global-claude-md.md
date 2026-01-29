# Global Claude Settings

## Language Overloading

These terms have overloaded meanings—when you see them, they trigger specific behavior:

| Term | Meaning |
|------|---------|
| **clear** | Spawn a fresh, headless agent with isolated context |
| **loopy** | Emphasis on enabling a full loop for the agent when possible |
| **full force** | Parallel Claude + Codex analysis, then merge into one document |
| **triple force** | Parallel Claude + Codex + Gemini analysis, then merge |
| **pro perspective** | Launch ChatGPT 5.2 Pro with extended thinking via Claude-in-Chrome, save output to markdown |
| **deep pro perspective** | Launch ChatGPT 5.2 Pro Deep Research via Claude-in-Chrome, save output to markdown |
| **dialogue** | Enter dialogue-driven development mode—ask all questions needed to fully understand requirements before proceeding |
| **ultrathink** | Take extra time to reason through edge cases, side effects, and potential issues before acting |
| **deeply** | Same as ultrathink - thorough analysis before implementation |

See sections below for details.


## 'Clear' Agents (Headless Background Instances)

**"Clear"** means: spawn a fresh, headless agent with isolated context. The word is intentional—context is "clear" (fresh), and it's unusual enough to avoid accidental triggers.

| Trigger                 | Tool   | Command                                          |
| ----------------------- | ------ | ------------------------------------------------ |
| "Launch a clear claude" | Claude | `claude -p "prompt" --dangerously-skip-permissions` |
| "Launch a clear codex"  | Codex  | `codex exec --full-auto --skip-git-repo-check "prompt"` |
| "Launch a clear gemini" | Gemini | `gemini -m gemini-3-pro-preview --yolo "prompt"` |

All variants run in background (`run_in_background: true`) and are monitored via `TaskOutput`.

**Tip:** Use "clear codex" or "clear gemini" for code reviews to get perspectives from different models.

**Examples:**
- "Launch a clear agent to fix all lint errors"
- "Launch a clear codex to review this PR"
- "Launch a clear gemini to analyze the architecture"

### Invoking Codex Reliably

**Codex ignores stdin.** Do not pipe content to Codex — it won't receive it.

**Use the file reference approach:**
```bash
# 1. Write prompt/content to a file
cat <<'EOF' > /tmp/codex-prompt.md
[Your instructions here, can include complex markdown, backticks, etc.]
EOF

# 2. Tell Codex to read the file
codex exec --full-auto --skip-git-repo-check "Read /tmp/codex-prompt.md and follow the instructions. Write output to /tmp/codex-output.md"

# 3. Read the result
cat /tmp/codex-output.md
```

**For simple prompts** (no special characters), inline works fine:
```bash
codex exec --full-auto --skip-git-repo-check "Review /path/to/file.ts for bugs. Write findings to /tmp/review.md"
```

**Key flags:**
- `--full-auto` — No approval prompts
- `--skip-git-repo-check` — Works outside git repos

### Shell Escaping (Claude)

For Claude, prompts with backticks/quotes/markdown cause shell interpretation errors. Options:
- **File reference (recommended):** Write prompt to file, use `claude -p "Read /tmp/prompt.md and follow instructions"`
- **Piped stdin:** `cat file | claude -p -` (works for Claude, NOT Codex)

Avoid heredoc with `$(cat <<'EOF' ... EOF)` in nested bash contexts — can cause hangs.

## 'Loopy' Tasks (End-to-End Autonomy)

**"Loopy"** means: attempting the current item of discussion as close to a full loop yourself — implement, validate, iterate, report. A loopy task doesn't stop at "I've made the changes,' it instead attempts to fully validate as best possible. However, there are times where human involvement is required: do not steamroll this! Not every task should be made loopy.

In practice being asked to make the task loopy tends to mean to use curl, ios simulator mcp, or claude in chrome, to validate the work or item in discussion. It means to unblock and unleash the agent!

**Validation tools:** Use whatever makes sense—curl, test suites, log inspection, manual checks, etc. You're empowered to choose. That said, three tools are particularly powerful:
- **iOS Simulator MCP** (`mcp__ios-simulator-mcp__*`) — screenshots, tap flows, verify UI. Highly effective for iOS mobile validation.
- **Mobile MCP** (`mcp__mobile-mcp__*`) — Android emulator automation via UI Automator. Use for Android mobile validation. Note: Do NOT use for iOS—it misses interactive elements.
- **Claude-in-Chrome** (`mcp__claude-in-chrome__*`) — interact with live pages, verify behavior. Ideal for web-based validation.

**Be proactive.** Your job is to save the user time. If you can validate something yourself, do it. If you can figure something out, figure it out. Ask questions when genuinely blocked or uncertain about requirements — not when you're capable of completing the loop independently.

**Stay in the loop.** If a fix doesn't work, try another approach. If tests fail, investigate why. Iterate until the task is actually complete or you hit a genuine blocker requiring human input.

## 'Dialogue' (Dialogue-Driven Development)

**"Dialogue"** means: pause execution and enter a collaborative conversation to fully understand requirements before building. The model is empowered to ask as many questions as necessary—ignore normal "be concise / move fast" defaults.

**Why this matters:** Models have broad knowledge of patterns, tradeoffs, edge cases, and potential pitfalls—but this intelligence is wasted if the model charges ahead with assumptions instead of surfacing what it knows. Dialogue-driven development creates space for the model's knowledge to combine with the operator's context and decision-making authority. The result is a shared understanding that produces more accurate, valid, and well-considered solutions than either could achieve alone.

**Core principle:** Use the `AskUserQuestion` tool liberally. Ask every question needed to eliminate ambiguity. There is no limit—5 questions, 10 questions, 20 questions are all acceptable if that's what's needed to fully understand the task. Do not proceed with assumptions when you could ask instead.

**When to enter dialogue mode:**
- User explicitly invokes it ("dialogue", "let's discuss", "let's talk through this")
- Story or task lacks detail
- Model detects ambiguity or thinks the task deserves more discussion
- Scope is large enough that assumptions could lead to significant rework

**What good dialogue looks like:**
- Use `AskUserQuestion` repeatedly until you fully understand:
  - The user's actual needs and goals
  - The existing system's constraints and patterns
  - The story's acceptance criteria (stated and implied)
- Surface tradeoffs, edge cases, and options the user may not have considered
- Validate assumptions explicitly rather than guessing
- Ask follow-up questions when answers reveal new ambiguity

**How dialogue relates to planning:**
- **Dialogue** = "Do we fully understand what we're building?"
- **Planning** = "How do we build this thing we understand?"

Dialogue can come before planning, after planning (to refine), or standalone. It's a phase, not a replacement.

**Exit conditions:**
- Model feels confident it fully understands the requirements
- User signals completion ("good to go", "let's proceed", "do it")

**Anti-pattern this prevents:** Charging ahead with a plan based on incomplete information, forcing the model to guess at requirements and potentially building the wrong thing.

## 'Full Force' and 'Triple Force' (Multi-Model Analysis)

**"Full force"** means: spawn parallel clear agents for diverse model perspectives, then merge into one document with Claude Opus. Use for reviews, plans, and analysis—**not for implementation**.

| Term | Models |
|------|--------|
| **full force** | Claude (Opus) + Codex |
| **triple force** | Claude (Opus) + Codex + Gemini |

Run agents in parallel with identical prompts. Each writes to `notes/`. After all complete, spawn Claude Opus to synthesize a merged document. If one agent fails, proceed with the others and note the gap.

**Example:** "Full force review of this PR" → Claude + Codex review in parallel → merged feedback document.

## 'Pro Perspective' and 'Deep Pro Perspective' (ChatGPT 5.2 Pro)

Two workflows for getting external model perspectives via Claude-in-Chrome browser automation:

| Trigger | Model | Mode | Runtime |
|---------|-------|------|---------|
| **pro perspective** | GPT 5.2 Pro | Extended thinking | Minutes |
| **deep pro perspective** | GPT 5.2 Pro | Deep Research | 5-40 minutes |

The naming is compositional: **pro** (GPT 5.2 Pro model) + **perspective** (external analysis). Adding **deep** enables the Deep Research feature (web browsing, longer runtime, clarifying questions).

### Pro Perspective (Extended Thinking)

**"Pro perspective"** = GPT 5.2 Pro with extended thinking. Faster, no web research.

1. **Open ChatGPT:** Create new tab, navigate to chatgpt.com
2. **Configure:** Select Pro mode (no Deep Research)
3. **Submit prompt:** Use clipboard method (pbcopy + Cmd+V) for reliable text input
4. **Wait for completion:** Usually completes in minutes
5. **Extract output:** Use `get_page_text` (NOT the copy button—it doesn't work)
6. **Save to markdown:** Write output to appropriate location
7. **Continue:** Only after saving, proceed with whatever comes next

**Examples:**
- "Get a pro perspective on this architecture decision"
- "Pro perspective on whether this refactor is worth it"

### Deep Pro Perspective (Deep Research)

**"Deep pro perspective"** = GPT 5.2 Pro with Deep Research enabled. Web browsing, longer runtime, asks clarifying questions first.

1. **Reference the skill:** Use `/deep-research` skill for detailed UI navigation steps
2. **Open ChatGPT:** Create new tab, navigate to chatgpt.com
3. **Configure:** Select Pro mode → Enable Deep Research (via "+" menu)
4. **Submit prompt:** Use clipboard method (pbcopy + Cmd+V) for reliable text input
5. **Handle clarifying questions:** ChatGPT asks 3-5 questions before starting (see guidance below)
6. **Wait for completion:** Monitor every 5 minutes; can take up to 40 minutes. Use `sleep 300` (bash) between checks—the Claude-in-Chrome wait action is limited to 30 seconds
7. **Extract output:** Use `get_page_text` (NOT the copy button—it doesn't work)
8. **Save to markdown:** Write output to appropriate location
9. **Continue:** Only after saving, proceed with whatever comes next

**Clarifying questions guidance:** When ChatGPT asks its 3-5 clarifying questions before starting Deep Research:
- **Default behavior:** Use your judgment. Read the questions carefully, think deeply about them in context of the original prompt, then provide thoughtful answers that will steer the research effectively.
- **Stock questions mode:** If the prompt explicitly says to use "stock questions," give generic/neutral answers that don't narrow the research scope. This is useful when you want broad exploration rather than targeted research.
- **When genuinely uncertain:** If a question requires user-specific context you don't have, ask the user before answering.

**Examples:**
- "Get a deep pro perspective on this thesis" → full workflow, save output
- "Deep pro perspective on IT services disruption" → full workflow, save output
- "Deep pro perspective on EPAM, GLOB, and INFY" → parallel tabs, full workflow for each

### Shared Technical Details

**Text input method:** ChatGPT's input field breaks with direct typing of newlines. Always use:
```bash
echo "Your prompt here" | pbcopy
```
Then: click input → type single char → Cmd+A Cmd+V → Enter

**Output extraction:** The ChatGPT "Copy" button does NOT copy to system clipboard. Always use `get_page_text` to extract research output.

**Parallel sessions:** ChatGPT supports multiple simultaneous tabs. For batch research, open multiple tabs and submit prompts to each.

**Error handling:** You're used precisely because you can handle UI changes and unexpected states resiliently. Adapt and proceed. However, if you encounter **significant** UI changes (new layout, renamed features, broken workflows), flag them to the user and suggest updating this CLAUDE.md section or the `/deep-research` skill.

**Key point:** Both workflows are NOT just "ask ChatGPT." They are complete automated workflows including saving the output. Do not consider the task complete until markdown is written.

## Emphasis on Parallelizing Work

You can spawn up to 10 agents simultaneously. When a task is parallelizable, try divide it into up to 10 approximately equal **buckets of work** (by complexity/effort, not file count). Ensure that each bucket can be independent to avoid agents messing each other up: rather have too few buckets than too many.

**Proactively suggest parallelization** when you recognize a task that could fit the pattern. Some non exclusive examples:
- Fixing multiple failing tests
- Adding tests for components/modules
- Updating multiple independent files with similar changes
- Any batch operation on independent items

If the user hasn't explicitly requested parallel execution, suggest it: "This looks like a good candidate for parallel execution—want me to split this across multiple agents?"

**Example:** "Fix all 15 failing tests" → spawn 10 agents, each handling ~1-2 tests based on complexity

**When NOT to parallelize:** Tasks with shared mutable state, order-dependent operations, or where agents would edit the same files. When in doubt, fewer buckets is safer than too many.

## Known Issue: Background Agents (run_in_background: true)

**Status:** Bug in Claude Code as of Jan 2025

**Problem:** When using `run_in_background: true` with the Task tool, completion notifications are decoupled from `TaskOutput`. This causes:
1. Session gets stuck in "Channelling..." state after agents complete
2. `/tasks` shows nothing running, but session won't return to interactive mode
3. Pressing Esc sometimes dismisses notifications, sometimes does nothing
4. Calling `TaskOutput` retrieves results but does NOT acknowledge/consume notifications

**Workaround:** Do NOT use `run_in_background: true`. Instead, spawn multiple agents in a single message without the flag:

```
# BAD - causes stuck sessions
Task(prompt="...", run_in_background=true)
Task(prompt="...", run_in_background=true)

# GOOD - agents run in parallel, session handles completion correctly
Task(prompt="...")  # No background flag
Task(prompt="...")  # Called in same message = parallel execution
```

Both patterns run agents in parallel, but only the non-background pattern terminates cleanly.

**Impact on "clear" agents:** The "clear claude/codex/gemini" pattern uses `run_in_background: true` via Bash, which is different from the Task tool's background flag. Those still work because they're shell processes, not Task tool agents.

## Keeping Claude & Codex Commands in Sync

Claude Code and Codex CLI use identical command formats, so commands are shared via **hardlinks**:

| Location | Purpose |
|----------|---------|
| `~/.claude/commands/*.md` | Claude Code slash commands |
| `~/.codex/prompts/*.md` | Codex CLI prompts (hardlinked) |

**How it works:** Both paths point to the same files on disk. Edit one, both update.

**Sync script** (run after adding new commands):
```bash
~/.claude/scripts/sync-commands-to-codex.sh
```

Creates hardlinks for any new `.md` files in `~/.claude/commands/`. Runs instantly.

## Tmux

Custom prefix is `Ctrl-f`. See `~/.claude/references/tmux.md` for full cheat sheet.

## References Folder

`~/.claude/references/` holds detailed reference material (cheat sheets, configs, etc.) for global settings—keeps this file lean.

## Environment Confirmation

Before running any command that affects data or server state, confirm the target environment:
- "This will run against [dev/prod/local]. Is that correct?"
- Never assume - explicit confirmation prevents costly mistakes

## Branch Safety

**At session start:** Verify the current branch matches the intended work. If unclear, ask.

**When creating a new branch:**
1. Ensure you're on `dev` first: `git checkout dev && git pull`
2. If not on `dev`, confirm with user before proceeding
3. Use proper naming: `aut-XXX-type-description`

## Work Completion

When it appears all work on a task is done:
1. Ask if blueprints or CLAUDE.md need updating based on what was learned
2. Offer to commit, push, and open a PR (`/compushar`)

## Recognizing Thrashing

If you notice repeated failed attempts at the same bug (3+ tries), circular debugging, or user frustration:

1. **Pause and acknowledge:** "This isn't going as intended. We may be thrashing."
2. **Generate a fresh handoff prompt** for a new agentic session containing:
   - **Actions taken**: What we tried and why it didn't work
   - **Current state**: Relevant file states, error messages, test output
   - **Observed behavior**: What's actually happening vs. expected
   - **Possible directions** (non-exhaustive): A few hypotheses to explore, framed openly—not as "the answer"

Fresh context without accumulated assumptions often breaks through stuck situations.

## Self-Improvement

If you notice a pattern, convention, or piece of knowledge that would help future sessions, suggest adding it to the relevant CLAUDE.md file. Every future session (yours and other agents) reads these files first. A 30-second addition here can save hours across the team of agents and humans who would otherwise rediscover the same thing.
