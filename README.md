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

Anthropic の [Best practices for using Claude Opus 4.7 with Claude Code](https://claude.com/blog/best-practices-for-using-claude-opus-4-7-with-claude-code) に沿った推奨設定。

### ランタイム設定

- **effort**: `xhigh` (Claude Code では 4.7 から新規デフォルト。コーディング / エージェント用途でこのスキルが最も期待通りに動く)
- **`max_tokens`**: 64k 以上を目安に。4.7 は同じ文面でもトークン消費が +0〜35% 増えるため、compaction と完了報告の余地を残す
- **thinking**: `adaptive` のみ。4.7 では `budget_tokens` による手動指定はエラーになる。思考量を増やしたいときは "この問題は見た目より難しい。step-by-step で慎重に考えてから答えて" のようにプロンプトで直接指示する
- **effort は固定しない**: タスクの途中でも `/effort` で切り替えられる。schema 設計 → xhigh、typo 修正 → medium のように使い分ける

### Claude Code での流儀

4.7 はペアプログラマではなく、**コンテキストを渡して任せる有能なエンジニア**として扱うのが相性がいい。このスキルのワークフローは最初からそれを前提に設計されている:

- **最初のターンで front-load する** — be-semantic の Step 1 が「ストーリー + なぜなら + エンティティ列挙」を必須にしているのは、まさに intent / constraints / acceptance criteria を 1 ターン目に集約させるため。ユーザーの指示が曖昧なまま複数ターンで断片的に出されると、4.7 は token もパフォーマンスも落ちる
- **auto モード (Y) を遠慮なく使う** — be-semantic 冒頭の `Y/n` 選択で Y を選ぶと、ALPS / Fake / Schema / Be 実装まで連続実行する。Claude Code Max では `Shift+Tab` で auto モードに入れる
- **ユーザー介入を減らす** — 毎ターンで確認を取るより、Step 2 (ALPS HTML) と Step 3 (Fake 50 件) のようにドメインが曲がるポイントだけで合意を取る方が結果が良い

### なぜ 4.7 でスキルを調整したか

4.7 は 4.6 に比べて:

- 指示をより**忠実に**解釈する (曖昧な "適切な" を字義どおり受ける)
- サブエージェント起動を**控える** (fan-out が欲しいときは明示が必要)
- ツール呼び出しを**控え、推論を増やす** (ツール実行が完了条件ならその旨を明示)
- 進捗報告を**内蔵**している (「N ステップごとに要約せよ」のような足場は不要)

スキル本体の vague な表現を具体化し、サブエージェント起動トリガーを明示化したのはこれに対応するため。4.6 以前でもスキルはそのまま動く。

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
