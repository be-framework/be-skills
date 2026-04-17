# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## このリポジトリの性質

このリポジトリは **Claude Code プラグイン** であり、Be Framework（PHP）アプリ開発のための **3つの Agent Skill** を配布する。ビルド・テスト・リント対象のコードは含まない。編集対象は実質 `*/SKILL.md` とプラグイン設定のみ。

- 配布単位: `.claude-plugin/marketplace.json` が定義する `be-framework-skills` プラグイン
- インストール: `claude plugin marketplace add be-framework/be-skills` → `claude plugin install be-framework-skills`
- 動作確認は「スキルに従う PHP プロジェクトを別ディレクトリで生成して回す」ことで行う（本リポ内では完結しない）

## 3スキルの階層関係

スキル同士は疎結合だが、意味上のレイヤーがある。SKILL.md を編集するときはこの階層を壊さない：

```
be-semantic (ワークフロー全体)
  └─ Step 3〜5 で semantic-ex の手法を使う（データ生成 → 観察 → 制約）
  └─ Step 6 で be の実装ルールを使う（Input/Semantic/Final、CQRS、Been）
```

| スキル | 責務 | 典型トリガー |
|---|---|---|
| [be-semantic/SKILL.md](be-semantic/SKILL.md) | Story → ALPS → Fake → Schema → Be の end-to-end ワークフロー | 新規アプリを一気通貫で設計したい |
| [semantic-ex/SKILL.md](semantic-ex/SKILL.md) | ALPS から50件fakeデータを生成し観察から JSON Schema を導く手法 | ALPS だけあってスキーマを作りたい |
| [be/SKILL.md](be/SKILL.md) | Be Framework の実装ルール（属性、ディレクトリ構造、CQRS、Been、テスト哲学） | PHP 実装フェーズ |

一つのスキルに書いた内容を他スキルに二重化しない。`be-semantic` は Step 6 で明示的に `be/SKILL.md` を読ませる方針（DomainException 継承、`#[Message]` 必須、Semantic 変数 1:1 など be 固有事実は `be/` 側に集約）。

## SKILL.md の規約

各 SKILL.md は先頭に YAML フロントマターを持つ：

```yaml
---
name: <skill-id>         # ディレクトリ名と一致
description: "..."        # "Use when ..." を含める。起動トリガーになる
---
```

`description` は Claude がスキルを **いつ起動するか** を決める。キーワードや日本語での起動フレーズ（「〜のとき」「〜するには」）を含めるかどうかでヒット率が変わる。description を変えたら動作が変わる前提で編集する。

## 設計哲学（スキル間で共有される中核思想）

「解像度を上げる」プロセス。スキル本文の根拠になっているので編集時は踏襲する：

```
User Story (曖昧) → ALPS (状態遷移を形式化) → Semantic-Ex (観察から制約発見)
  → JSON Schema (一意のゴール) → Be code (スキーマに収束)
```

自然言語仕様は100通りの実装を生むが、スキーマは1通り。AI駆動開発でぶれない成果物を作るための足場。

生成物は **`design/`** 以下に置く（`design/story/`、`design/alps/`、`design/fake/`、`design/schema/`）。これは "authoritative source of truth" として git にコミットする層で、be-skeleton の `var/`（runtime tmp）とは別レイヤー。`src/`・`tests/`・`bin/` も別レイヤー。

## 編集時の注意

- `README.md` のデモ（体重記録ストーリー + 3種類の「なぜなら」→ 3種類のアプリ）は be-semantic の価値提案を示す中核例。変更するなら be-semantic/SKILL.md の「なぜなら節は必須」ルールと整合させる
- `be-semantic` の Step 2 で ALPS ファイル名は **`alps.xml` 固定**（プロジェクト名を含めない）。直近のコミット `535991f` でここを強制する意図あり
- `be/SKILL.md` の「実装前に相談すべき設計判断」「実装中の姿勢」「sub-agent 規約」セクションは、AIに対する **モードプロンプト** として機能している。命令形・断定形を維持する
- `multipleOf` の罠（IEEE754 浮動小数）、ドメイン狭小化の禁止、テスト実行の必須化など、過去に踏んだ落とし穴は意図的にスキルに書いてある。単なる冗長に見えても削らない

## Git 運用（ユーザーのグローバル規約を反映）

- メインブランチは `1.x`。**`1.x` に直接コミットしない**。必ずフィーチャーブランチを切る
- `git add -A` / `git add .` 禁止。ファイル名を明示指定
- PR の作成・マージはユーザーの明示許可が必要。`gh pr merge` 禁止
- コミットメッセージに `Co-Authored-By: Claude` や `Generated with Claude Code` を付けない
