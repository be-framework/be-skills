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

ユーザーとして、毎日の体重を記録したい。
なぜなら、目標体重に近づいているか確認したいから。
```

Now change just the "なぜなら" line and run it again:

```
... なぜなら、医師に診察時に共有したいから。
... なぜなら、三日坊主にならず習慣として続けたいから。
```

Same noun, three different apps:

| なぜなら | Resulting design |
|---|---|
| 目標体重に近づいているか確認したい | Goal entity + 達成/未達 Branching |
| 医師に共有したい | Date-range export + 正常範囲 Branching |
| 三日坊主にならず続けたい | Streak counter + 継続/途切れ Branching |

The "なぜなら" clause reshapes the domain — that's why `be-semantic` starts from a story, not a schema.

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
