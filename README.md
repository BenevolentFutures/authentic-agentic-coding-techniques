# Authentic's Agentic Coding Techniques

**Production-tested workflows for AI-native software development.**

> **[Authentic](https://authentic.app)** is a social app focused on genuine connection - no algorithms, no discovery feeds, just the people you actually care about. We're a small team building something different, and we ship fast using the agentic techniques in this repo. **[We're hiring!](https://authentic.tech/careers)**
>
> **Interested in context engineering and agentic development?** Our founder [Atin](https://github.com/BenevolentFutures) leads the **[Context Engineering Guild of NYC](https://www.contextengineeringguild.nyc/)** - a community dedicated to advancing these practices. Join us if you want to go deeper.

This repository contains the commands, configurations, and patterns we use at Authentic to build software with agentic AI assistants. In an era of countless influencers sharing Claude Code tips, we're sharing something different: the actual workflows we use to ship production code, every day.

---

## Why We're Sharing This

Authentic has been at the frontier of agentic coding since the earliest days of Claude Code. We recognized early that the shift from "AI as autocomplete" to "AI as collaborative agent" represented a fundamental change in how software gets built - and we've invested heavily in developing the workflows to capitalize on it.

This repository represents our contribution to the broader community: the actual techniques we use, not a sanitized version.

## Things to Be Aware Of

### 1. These Are Our Actual Production Commands

We're not publishing a sanitized, generic version of our workflows. These commands ship production code at Authentic daily - they're the real thing. The global CLAUDE.md example is reasonably portable; the workspace example is quite specific to our codebase and conventions.

**Customization is expected.** Only you know your team's needs, tech stack, and workflow preferences. Treat these as a starting point and reference implementation, not a drop-in solution. Read the files, understand the patterns, and adapt accordingly.

### 2. Language Overloading is a Key Insight

If you take one thing from this repository, let it be this: **defining semantic trigger words that invoke specific agent behaviors is an extremely effective technique that remains deeply under-discussed** in the agentic development community.

When we say "loopy," the agent knows to complete full validation loops. When we say "dialogue," it shifts into requirements-gathering mode. When we say "triple force," it orchestrates a multi-model review. These aren't magic words - they're documented conventions in our CLAUDE.md that fundamentally change how we interact with agents.

This extends to code reviews. By looping on reviews - running them multiple times, with multiple agents, ideally across multiple models - we achieve far higher signal than single-pass review. We're currently seeing excellent results from GPT-5 for code reviews, in some cases surpassing our much-beloved Claude Opus 4.5. The models have different strengths; triangulating across them catches issues any single model would miss.

### 3. Empower the Agent to Close the Loop

The term "loopy" in our vocabulary means: don't stop at "I made the changes." Implement, validate, iterate, and report back when it's actually done.

For us as a mobile development team, this means empowering agents to use the iOS Simulator and Android Emulator directly. An agent fixing a UI bug doesn't just edit the code - it rebuilds, launches the app, navigates to the relevant screen, and visually confirms the fix. This level of autonomy, properly constrained, dramatically reduces the back-and-forth that slows down development.

The guardrails matter: destructive operations require confirmation, uncertainty triggers questions rather than assumptions, and production-affecting commands demand explicit approval. But within those boundaries, we want agents completing full loops, not waiting for permission at every step.

### 4. Dialogue-Driven Development

Before implementation begins, engage in structured dialogue to surface ambiguity. The model should be empowered - indeed, expected - to ask as many questions as necessary to fully understand requirements, constraints, and edge cases.

This front-loaded investment in understanding prevents the far more expensive cost of building the wrong thing. We've found that explicitly invoking "dialogue mode" (another overloaded term) shifts the agent's behavior from eager implementation to thorough requirements gathering. The resulting clarity pays for itself many times over.

---

## Repository Contents

### Commands

| Command | Purpose |
|---------|---------|
| **[ship.md](commands/ship.md)** | End-to-end implementation workflow: task understanding → implementation → dual-agent review → consolidation → parallel fixes → PR creation. Twelve phases with built-in self-correction. |
| **[code-review.md](commands/code-review.md)** | Comprehensive single-agent review covering architectural assessment, tactical implementation analysis, and validated issue categorization. |
| **[code-review-critical.md](commands/code-review-critical.md)** | Adversarial review focused on failure modes, security vulnerabilities, state management issues, and timing problems. |
| **[team-three-review.md](commands/team-three-review.md)** | Six-agent parallel review: Claude, Codex, and Gemini each run both standard and critical reviews, followed by intelligent synthesis. |

### CLAUDE.md Examples

Claude Code reads `CLAUDE.md` files at multiple levels, each adding context as the agent descends into your project structure. Understanding this hierarchy is key to effective configuration:

| Level | Location | Purpose |
|-------|----------|---------|
| **Global** | `~/.claude/CLAUDE.md` | User-wide settings: language overloading, multi-model patterns, personal conventions |
| **Workspace** | `/path/to/workspace/CLAUDE.md` | Shared conventions across multiple related repositories |
| **Project** | `/path/to/repo/CLAUDE.md` | Repository-specific context: tech stack, commands, gotchas |

The `claude.md/` folder contains examples at each level:

| File | Level | Description |
|------|-------|-------------|
| **[global-claude-md.md](claude.md/global-claude-md.md)** | Global | Language overloading, multi-model orchestration, cross-project conventions |
| **[workspace-claude-md.md](claude.md/workspace-claude-md.md)** | Workspace | Multi-repo coordination, shared team patterns, safety rules |

**Note:** We've included our global and workspace configurations. Project-level CLAUDE.md files tend to be highly specific to a tech stack and codebase - you and your Claude need to figure out what belongs in yours. Study the patterns in these examples, understand the reasoning, then build your own.

**Codex / AGENTS.md compatibility:** The emerging industry standard for agent configuration is `AGENTS.md`. If you use Codex or other tools that read `AGENTS.md`, simply symlink it:

```bash
ln -s CLAUDE.md AGENTS.md
```

This keeps a single source of truth while maintaining compatibility across tools.

---

## Key Techniques

### Language Overloading

We define semantic triggers that invoke specific agent behaviors:

| Trigger | Behavior |
|---------|----------|
| **"clear"** | Spawn a fresh agent with isolated context - useful for parallel work or escaping accumulated confusion |
| **"loopy"** | Execute complete validation loops: implement, test, iterate until actually done |
| **"full force"** | Parallel Claude + Codex analysis with merged synthesis |
| **"triple force"** | Parallel Claude + Codex + Gemini analysis with comprehensive synthesis |
| **"dialogue"** | Enter dialogue-driven development mode; exhaust all clarifying questions before proceeding |

### The Ship Workflow

Our `/ship` command orchestrates a complete development cycle:

```
Task → Understand → Implement → Commit
                                  ↓
            ┌─────────────────────┴─────────────────────┐
            ▼                                           ▼
      Claude Review                               Codex Review
            │                                           │
            └─────────────┬─────────────────────────────┘
                          ▼
                   Consolidation
                          ↓
                   Validation (main thread confirms/rejects)
                          ↓
                   Parallel Fixes (bucketed by file)
                          ↓
                   PR Creation
                          ↓
                   Second Round Review (higher threshold)
                          ↓
                   Summary + Cleanup
```

### Multi-Model Review Architecture

```
Orchestrator (Opus)
├── Executes tests once
├── Generates shared context
└── Launches 6 parallel agents:
         │
    ┌────┴────┬─────────────┬─────────────┬─────────────┬─────────────┐
    ▼         ▼             ▼             ▼             ▼             ▼
  Claude    Claude        Codex        Codex        Gemini       Gemini
(standard) (critical)  (standard)  (critical)   (standard)   (critical)
    │         │             │             │             │             │
    └─────────┴─────────────┴─────────────┴─────────────┴─────────────┘
                                    │
                                    ▼
                             Synthesis Layer
                    ├── High-confidence issues (3+ agents)
                    ├── Single-agent findings (investigate)
                    └── Contradictions (often the most valuable signal)
```

---

## Quick Start

### Installing Commands

```bash
# Clone the repository
git clone https://github.com/AuthenticTechnology/authentic-agentic-coding-techniques.git

# Copy commands to your Claude Code directory
cp authentic-agentic-coding-techniques/commands/*.md ~/.claude/commands/
```

### Usage

```bash
# Full implementation workflow
/ship implement the user authentication flow from notes/auth-spec.md

# Single-agent code review
/code-review

# Six-agent multi-model review
/team-three-review
```

### Configuring CLAUDE.md

The examples in `claude.md/` demonstrate each level of the hierarchy. Use them as starting points:

```bash
# Global configuration (applies to all your projects)
cp claude.md/global-claude-md.md ~/.claude/CLAUDE.md

# Workspace configuration (for multi-repo setups)
cp claude.md/workspace-claude-md.md /path/to/workspace/CLAUDE.md

# Project-specific: create your own based on your tech stack
# Study the patterns in global and workspace examples
touch /path/to/your/repo/CLAUDE.md
```

For Codex compatibility, symlink after creating:
```bash
cd /path/to/your/repo
ln -s CLAUDE.md AGENTS.md
```

---

## Requirements

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)
- Git and GitHub CLI (`gh`)
- For multi-model reviews: [Codex CLI](https://openai.com/codex), [Gemini CLI](https://ai.google.dev/gemini-cli)
- Node.js/npm (for validation steps in the workflows)

---

## About Authentic

[Authentic](https://authentic.app) is building a social platform focused on genuine human connection. As a small, fast-moving team, we've made agentic AI development a core part of our engineering practice - not as an experiment, but as our primary mode of building software.

## Contributing

We welcome contributions. If you've developed patterns that improve on these workflows, or discovered techniques we haven't considered, please open a PR or start a discussion.

## License

MIT License. Use these techniques however they serve you.

---

*Built by the Authentic engineering team. For more on context engineering and agentic development practices, visit the [Context Engineering Guild of NYC](https://www.contextengineeringguild.nyc/).*
