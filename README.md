# cc-codex-dev

CC orchestrate, Codex implement, CC verify — A Claude Code skill for cross-model AI collaborative development.

## What is this?

A Claude Code slash command (`/codex-dev`) that delegates coding tasks to OpenAI Codex while Claude Code handles scoping, review, and verification.

**The core idea**: Split work by context needs, not by role. Cross-model review catches blindspots that same-model review misses.

## Design Principles

This skill was designed based on guidance from Anthropic and Karpathy:

1. **Split by context, not role** — Agents don't need "personas". They need the right files in their context window. (Anthropic: [How we built our multi-agent research system](https://www.anthropic.com/engineering/built-multi-agent-research-system))

2. **Start simple, evolve with data** — Single Codex call is the default. No parallel execution in V1. Add complexity only when real usage data proves it's needed. (Anthropic: [Building Effective Agents](https://www.anthropic.com/research/building-effective-agents))

3. **Verification > Orchestration** — Hard gates (build/test) run before LLM review. Deterministic checks are more reliable than LLM judgment. (Karpathy: March of Nines — a 10-step workflow at 90% per step = 35% overall reliability)

4. **Fewer steps, harder checks** — Every handoff compounds failure. Keep the chain short: scope → delegate → verify → report.

5. **Abort on scope creep** — If Codex needs files outside the allowed list, new dependencies, or schema changes, it stops immediately.

## Workflow

```
/codex-dev "implement feature X"

Step 1: CC scopes (reads code, identifies files, defines contract)
Step 2: CC delegates to Codex (with task + files + constraints + stop conditions)
Step 3: Codex implements
Step 4: CC verifies (build/test first, then LLM review of diff)
Step 5: Report (PASS/FAIL + next steps)
```

## Installation

### Option A: Copy to your commands directory

```bash
mkdir -p ~/.claude/commands
cp codex-dev/SKILL.md ~/.claude/commands/codex-dev.md
```

### Option B: Clone and symlink

```bash
git clone https://github.com/Lntanohuang/cc-codex-dev.git ~/.claude/skills/cc-codex-dev
ln -s ~/.claude/skills/cc-codex-dev/codex-dev/SKILL.md ~/.claude/commands/codex-dev.md
```

## Prerequisites

- [Claude Code](https://claude.ai/code) installed
- [Codex CLI](https://github.com/openai/codex) installed and authenticated (`codex --version`)

## Usage

In Claude Code, type:

```
/codex-dev "add a statistics page showing student accuracy and weak points"
```

Or just say: "let codex write this" — CC will auto-trigger the skill.

## When to use

- Well-scoped implementation tasks with clear file boundaries
- Tasks that benefit from a second model's perspective
- When you want CC to focus on architecture decisions while Codex handles implementation

## When NOT to use

- Trivial changes (rename, typo) — just do it directly
- Vague/unscoped tasks — scope first, delegate after
- Pure exploration — use CC's built-in Explore subagent

## References

- Anthropic — [Building Effective AI Agents (PDF)](https://resources.anthropic.com/hubfs/Building%20Effective%20AI%20Agents-%20Architecture%20Patterns%20and%20Implementation%20Frameworks.pdf)
- Anthropic — [How we built our multi-agent research system](https://www.anthropic.com/engineering/built-multi-agent-research-system)
- Anthropic — [Claude Code Subagent Docs](https://code.claude.com/docs/en/sub-agents)
- Karpathy — March of Nines / Agentic Engineering

## License

MIT
