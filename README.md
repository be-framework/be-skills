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

## Claude Opus 4.7 で使う

これらのスキルはモデル非依存で書かれているが、Claude Opus 4.7 に合わせた推奨設定がある:

- **effort**: `xhigh` を推奨（コーディング / エージェント用途のデフォルト。4.7 はこのレベルで tool 使用とサブエージェント起動が期待通りに出る）
- **`max_tokens`**: 64k 以上を目安に。4.7 は同じ文面でもトークン消費が +0〜35% 増えるため、compaction と完了報告の余地を残す
- **thinking**: `adaptive` のみ。4.7 では `budget_tokens` による手動指定はエラーになる
- **Claude Code**: `/model` で `claude-opus-4-7` に切り替え、`/effort xhigh` を設定する

4.6 以前でもスキルはそのまま動く。上の設定は 4.7 の振る舞い（より忠実な指示追従、サブエージェント起動の抑制、進捗報告の内蔵化）に対して最大効果を出すためのもの。

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
