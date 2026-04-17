# Be Framework Skills

Agent skills for building applications with the [Be Framework](https://github.com/be-framework/be-framework).

## Skills

- **[be](be/SKILL.md)** — Development workflow: from user stories to ALPS profiles to implementation
- **[be-semantic](be-semantic/SKILL.md)** — Story → ALPS → Fake → Agreement → Schema → Be workflow
- **[semantic-ex](semantic-ex/SKILL.md)** — Semantic Exercise: AI-driven data generation → constraint discovery → JSON Schema

## Demo

Try this with your agent:

```
Build a Be app from this story using the be-semantic skill.

As a user, I want to log my weight every day.
Because I want to see if I'm getting closer to my goal weight.

ユーザーとして、毎日の体重を記録したい。
なぜなら、目標体重に近づいているか確認したいから。
```

Now change just the "Because" / 「なぜなら」 line and run it again:

```
... Because I want to share the data with my doctor at checkups.
... Because I want it to become a lasting habit.

... なぜなら、医師に診察時に共有したいから。
... なぜなら、三日坊主にならず習慣として続けたいから。
```

Same noun, three different apps:

| Because | Resulting design |
|---|---|
| I want to see if I'm approaching my goal weight | Goal entity + reached/not-reached Branching |
| I want to share the data with my doctor | Date-range export + normal-range Branching |
| I want it to become a lasting habit | Streak counter + continued/broken Branching |

The "Because" clause reshapes the domain — that's why `be-semantic` starts from a story, not a schema.

Behind the prompt, the agent walks Story → ALPS → **Fake** → Schema → Be. The Fake step is the pivot: the agent generates 50 realistic records from your story, you skim them together, and constraints (`maxLength: 80`, optional fields, value ranges) emerge from observation — not from defaults. Those 50 records become a shared image of the domain, a common language between you and the agent. Artifacts land under `design/`.

## Installation

### Claude Code Plugin

```bash
claude plugin marketplace add be-framework/be-skills
claude plugin install be-framework-skills
```

All skills are automatically available after installation.

### Manual

Point your agent at the relevant `SKILL.md` to teach it how to build Be applications.

## Using with Claude Opus 4.7

Recommended settings follow Anthropic's [Best practices for using Claude Opus 4.7 with Claude Code](https://claude.com/blog/best-practices-for-using-claude-opus-4-7-with-claude-code).

### Runtime settings

- **effort**: `xhigh` — the new default in Claude Code for 4.7. This is where the skills work best for coding and agentic tasks.
- **`max_tokens`**: aim for 64k or higher. 4.7 uses 0–35% more tokens for the same text than 4.6, so leave headroom for compaction and final reporting.
- **thinking**: `adaptive` only. Manual `budget_tokens` returns an error on 4.7. If you want more thinking, steer with a prompt like *"Think carefully and step-by-step before responding; this problem is harder than it looks."*
- **Don't pin effort**: you can toggle mid-task with `/effort`. Use `xhigh` for schema design; drop to `medium` for a typo fix.

### How to work with Claude Code on 4.7

4.7 works best when you treat it as **a capable engineer you're delegating to**, not a pair programmer you guide line by line. The skills in this repo were designed that way from the start:

- **Front-load the first turn.** `be-semantic` Step 1 requires "story + because + entity enumeration" precisely so intent, constraints, and acceptance criteria land in turn 1. Vague prompts drip-fed across many turns hurt both token efficiency and quality on 4.7.
- **Use auto mode (`Y`) when you can.** The `Y/n` gate at the start of `be-semantic` runs ALPS → Fake → Schema → Be implementation end-to-end. In Claude Code Max you can also toggle auto mode with `Shift+Tab`.
- **Minimize user interrupts.** Take agreement only at the steps where the domain can bend — Step 2 (ALPS HTML review) and Step 3 (Fake 50-item preview) — not at every turn.

### Why the skills were tuned for 4.7

Compared to 4.6, Claude Opus 4.7:

- Follows instructions **more literally** (vague adjectives like "appropriate" are taken at face value).
- **Spawns fewer subagents by default** (state the fan-out trigger if you want parallelism).
- **Calls tools less often and reasons more** (if a tool execution is the completion gate, say so).
- **Reports progress on its own** (scaffolding like "summarize every N steps" is unnecessary).

The edits in this PR concretize vague phrasing and make subagent triggers explicit to match these behaviors. The skills still work as-is on 4.6.

## Philosophy

Be's development flow is a process of **raising resolution for AI-driven development**:

```
User Story        ← domain language (ambiguous)
  → ALPS Profile  ← formalized state transitions & semantics
  → Semantic-Ex   ← data generation → observation → constraint discovery
  → JSON Schema   ← unambiguous goal (no room for interpretation)
  → Be code       ← implementation converges to schema
```

Natural language specs produce 100 different implementations. Schemas produce one.
