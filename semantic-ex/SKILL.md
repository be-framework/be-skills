---
name: semantic-ex
description: "Generate JSON Schema from ALPS profiles using the Semantic-Ex method: generate realistic fake data, observe patterns, and derive constraints from data rather than assumptions. Use when creating JSON Schema from ALPS, generating fake data for constraint discovery, or when the user mentions Semantic-Ex."
---

# Semantic-Ex Skill

ALPSプロファイルを入力に受け取り、Semantic-Ex Methodに従ってJSONスキーマを生成するスキル。

**参考:** https://koriym.github.io/blog/2025/08/10/semantic-method-en

## 概要

「抽象から制約を決める」のではなく「現実のデータから制約を導く」アプローチ。

```
ALPS → (Experience) 大量fakeデータ生成 → (Examples) パターン観察 → (Constraints) JSONスキーマ
```

## 使い方

ユーザーが以下のようなリクエストをしたら、このスキルに従う：

- 「ALPSからJSONスキーマを作って」
- 「Semantic-Exでスキーマを生成して」
- 「fakeデータからスキーマを作って」

## 実行手順

### Phase 1: Experience（意味を体験する）

ALPSプロファイルのontology（セマンティックフィールド）を読む。各フィールドについて、**最低50件**のリアルなfakeデータを生成する。

生成の観点：

- 最短・最長の例
- 日本語と英語の混在
- 日常的なケースとエッジケース
- 実際のユーザーが入力しそうな内容

生成したfakeデータは `story/fake-data-50.json` に保存する。

### Phase 2: Examples（制約を観察する）

生成したfakeデータを分析して制約の仮説を立てる。

各フィールドについて記録：

- 最短の値（文字数）
- 最長の値（文字数）
- 典型的な値の長さ
- 使われている文字の種類（記号、数字、多言語）
- null/空文字の可能性

制約仮説は `story/fake-data.md` に保存する。

### Phase 3: Constraints（スキーマを生成する）

観察から導いた制約でJSONスキーマを生成する。

**重要なルール：**

- `maxLength` は「最長の観察値 × 1.5〜2倍」を目安にする（余裕を持たせる）
- `minLength` は現実の最短値から決める
- 「データベースの制限が255だから」「慣例的に50だから」という理由はNG
- すべての制約に根拠を持たせる

**リンク（\_links）の構造：**

- ALPSの遷移idをそのままrel名として使用（Beのrel）
- 例: `"goTodoList"`, `"doCreateTodo"`, `"doCompleteTodo"`
- 条件付きリンク（完了済みのときは `doCompleteTodo` が存在しない等）はスキーマにdescriptionで明記

生成するファイル：

- `schema/[state-name].json` — 各状態のJSONスキーマ（ALPSのtaxonomy単位）

### Phase 4: 確認

生成したスキーマをユーザーに確認してもらう：

- maxLength の値は適切か
- 省略可能なフィールドの扱いは正しいか
- linksの構造は想定通りか

## 出力形式

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/[app]/[state]",
  "title": "[StateName]",
  "description": "ALPSプロファイルの[StateName]状態に対応する",
  "type": "object",
  "required": [...],
  "properties": {
    "[fieldId]": {
      "type": "string",
      "description": "[ALPSのtitleやdocから]",
      "minLength": [観察から導いた値],
      "maxLength": [観察から導いた値]
    },
    "_links": {
      "type": "object",
      "properties": {
        "[alpsDscriptorId]": {"$ref": "#/$defs/link"}
      }
    }
  },
  "$defs": {
    "link": {
      "type": "object",
      "required": ["href"],
      "properties": {
        "href": {"type": "string", "format": "uri-reference"}
      }
    }
  }
}
```
