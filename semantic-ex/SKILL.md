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

生成したfakeデータは `design/fake/data-50.json` に保存する（be-semantic ワークフロー内から呼ばれる場合も同じパス。単体使用でもこのパスに統一）。

### Phase 2: Examples（制約を観察する）

生成したfakeデータを分析して制約の仮説を立てる。

各フィールドについて記録：

- 最短の値（文字数）
- 最長の値（文字数）
- 典型的な値の長さ
- 使われている文字の種類（記号、数字、多言語）
- null/空文字の可能性

制約仮説は `design/fake/observations.md` に保存する。

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

- `design/schema/[Taxonomy].json` — ALPS の `type="semantic"` タクソノミーごとに1ファイル（PascalCase 推奨。例: `TodoList.json`、`WeightRecord.json`）

### Phase 4: 確認

生成したスキーマをユーザーに確認してもらう。「適切か」のような曖昧な問いでは答えようがないので、**観察値と比較できる形で** 一項目ずつ提示する:

- **maxLength**: 観察された最長値 N 文字 → スキーマは M 文字（M / N の倍率 と `$comment` の根拠）
- **minLength**: 観察された最短値は 1 文字か、それ以上か
- **required**: 50 件中で **常に存在** したフィールドだけが載っているか。時々 null だったフィールドが required に混ざっていないか
- **optional の挙動**: null 率が高かったフィールドは optional として定義されているか
- **値域（数値 / enum）**: 観察された min / max / 取りうる値がそのまま制約になっているか
- **i18n**: 日本語と英語が混在する場合、`pattern` で ASCII のみに絞っていないか
- **\_links 構造**: ALPS の遷移 id がそのまま rel 名になっているか。条件付きリンクに description が書かれているか

ユーザーが軸ごとに OK / 修正指示を返せるよう、**チェックボックス形式で提示する**。

## 出力形式

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/[app]/[taxonomy]",
  "title": "[Taxonomy]",
  "description": "ALPSプロファイルの[Taxonomy]タクソノミーに対応する",
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
