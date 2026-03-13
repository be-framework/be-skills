# Be Framework Skills

Agent skills for building applications with the [Be Framework](https://github.com/be-framework/be-framework).

## Skills

- **[be](be/SKILL.md)** — Development workflow: from user stories to ALPS profiles to implementation
- **[be-semantic](be-semantic/SKILL.md)** — Story → ALPS → Fake → Agreement → Schema → Be workflow
- **[semantic-ex](semantic-ex/SKILL.md)** — Semantic Exercise: AI-driven data generation → constraint discovery → JSON Schema

## Installation

### Claude Code Plugin

```bash
claude plugin marketplace add be-framework/skills
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
