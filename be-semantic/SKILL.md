---
name: be-semantic
description: "End-to-end Be application design workflow: Story → ALPS → Fake data → Agreement → JSON Schema → Be implementation. Use when designing a new Be application from scratch, converting user stories into ALPS profiles, or when the user wants the full semantic-driven development flow."
---

# be-semantic Skill

**be-semantic** は、ユーザーの意図から始まり、意味論を経由してBeアプリを設計・実装するワークフロー。

**リポジトリ**: https://github.com/koriym/be-semantic  
**llms.txt**: https://raw.githubusercontent.com/koriym/be-semantic/main/llms.txt  
**ALPS docs**: https://www.app-state-diagram.com/llms-full.txt  
**Be Framework docs**: https://be-framework.github.io/llms-full.txt

---

## ワークフロー概要

```
ストーリー → ALPS → Fake（50件）→ 合意 → スキーマ → Be実装
```

各ステップは前のステップの出力を入力にする。スキップ不可。

---

## Step 1：ストーリー

**AIの役割**: ユーザーがアプリの機能を「ユーザーストーリー」として書くのを支援する。

**形式**（必須）:

```
ユーザーとして、
[アクション]したい。
なぜなら、[理由]だから。
```

**「なぜなら」節は必須**。省略を許さない。理由がドメインの真の意図を明かす。

**エンティティ列挙**: ストーリーの最後に、ドメインに存在するエンティティと属性を列挙させる。

**アウトプット**: `story/main.md`

---

## Step 2：ALPSプロファイル

**AIの役割**: ストーリーからALPSプロファイルをXMLで作成する。APIの状態遷移と内部の変容チェーンを1つのファイルにタグで区別して記述する。

**ツール**: `asd` コマンド

```bash
brew install alps-asd/asd/asd                    # macOS
npm install -g @alps-asd/app-state-diagram       # cross-platform

asd alps/todo.xml               # バリデーション + HTML生成
```

**ALPSの構造**（XML推奨 — コメントで構造を整理できる。コメントは簡潔に）:

```xml
<alps version="1.0">
  <!-- Ontology -->
  <descriptor id="todoId" title="Todo ID" def="https://schema.org/identifier" />
  <descriptor id="todoTitle" title="タイトル" />
  <descriptor id="isCompleted" title="完了状態" />

  <!-- Taxonomy -->
  <descriptor id="TodoList" type="semantic">
    <descriptor href="#todoId" />
    <descriptor href="#todoTitle" />
    <descriptor href="#isCompleted" />
  </descriptor>

  <!-- Choreography: API -->
  <descriptor id="goTodoList" type="safe" rt="#TodoList" tag="api" />
  <descriptor id="doCreateTodo" type="unsafe" rt="#TodoList" tag="api">
    <descriptor href="#todoTitle" />
  </descriptor>

  <!-- Choreography: Be -->
  <descriptor id="TodoValidation" type="semantic" tag="be">
    <doc>Semantic変数による入力検証の中間状態</doc>
    <descriptor href="#todoTitle" />
  </descriptor>
  <descriptor id="becomeValidated" type="unsafe" rt="#TodoValidation" tag="be" />
</alps>
```

**タグの使い分け**:

- `tag="api"` — API遷移（ユーザーに見える状態遷移）
- `tag="be"` — 内部変容（Being、中間状態、変容遷移）
- タグなし — オントロジー、タクソノミー（共有）

**命名規則**:

- `go` プレフィックス → safe（読み取り）
- `do` プレフィックス → unsafe/idempotent（書き込み）
- `become` プレフィックス → 内部変容遷移（Be固有）

**アウトプット**: `alps/[name].xml` + `alps/[name].html`（asd生成）

---

## Step 3：フェイクデータ生成

**AIの役割**: ALPSプロファイルを読んで、50件のリアルなフェイクデータを生成する。

**重要**: ランダムなテストデータではなく、「実際のユーザーが日常的に作るようなデータ」を生成する。

**生成プロンプト**:

```
以下のALPSプロファイルに基づいて、[アプリ名]のリアルなフェイクデータを50件生成してください。
実際のユーザーが日常的に使うような内容にしてください。

[ALPSプロファイルの内容]

出力形式: JSON配列
```

**アウトプット**: `fake/data-50.json`

---

## Step 4：観察と合意

**AIの役割**: フェイクデータを分析し、制約の候補をユーザーに提示する。確認を取る。

**分析項目**:

- テキストフィールドの長さ分布（最短・最長・平均）
- Optional フィールドの null 率
- 値のパターンと範囲
- エッジケース

**提示形式**:

```
観察結果:
- todoTitle: 最長41文字 → maxLength: 80 でどうでしょうか？
- todoMemo: 50件中35件がnull → optional にします
- isCompleted: true/false混在 → boolean
```

**必ず確認を取る**。勝手に決めない。

**アウトプット**: 観察メモ（次のステップで使う）

---

## Step 5：JSONスキーマ

**AIの役割**: 合意した制約をJSONスキーマとして記述する。

**原則**:

- `maxLength` は観察値ベースで設定（255のようなデフォルト値は使わない）
- `required` は「フェイクデータで常に存在するフィールド」のみ
- ALPS の `descriptor` id をプロパティ名として使う（一貫性）
- スキーマには観察根拠をコメントで記述する（`$comment`）

**例**:

```json
{
  "todoTitle": {
    "type": "string",
    "minLength": 1,
    "maxLength": 80,
    "$comment": "観察された最大長は41文字。余裕を持って80を上限とした。"
  }
}
```

**アウトプット**: `schema/[representation].json`（表現ごとに1ファイル）

---

## Step 6：Be実装

Be実装の詳細は `skills/be/SKILL.md` を参照。

**Beの変換モデル**: Input（起点）→ Final（終点）が基本。Beingは分岐が必要な場合のみ。

**ALPS → Be のマッピング**:

| ALPS                 | Be パターン                  | 例                                                                           |
| -------------------- | ---------------------------- | ---------------------------------------------------------------------------- |
| `safe`（go〜）       | Input → Final（直接変換）    | `GetTodoListInput` → `TodoList`                                              |
| `unsafe`（do〜）     | Input → Final（直接変換）    | `CreateTodoInput` → `TodoCreated`                                            |
| `idempotent`（do〜） | Input → Final（直接変換）    | `CompleteTodoInput` → `TodoCompleted`                                        |
| 分岐が必要な操作     | Input → Being → Final A or B | `PatientArrival` → `TriageAssessment` → `EmergencyCase` or `ObservationCase` |

**スキーマとBeの接続**:

- JSONスキーマの `required` → Input のコンストラクタ引数
- JSONスキーマの `properties` → 型とSemantic変数
- ALPS セマンティックディスクリプタ id → Input/Final のプロパティ名
- ALPS 表現（TodoList等）→ Final クラス名の候補

**セットアップ**: `be/SKILL.md` の「プロジェクト開始」セクションに従い、app templateからプロジェクトを作成する。

---

## AIが守るべきルール

1. **ストーリーより先に技術を話させない** — 「エンドポイントは？」「DBは？」はStep 6まで出さない
2. **「なぜなら」節を省略させない** — ユーザーが省略したら「なぜそうしたいですか？」と聞く
3. **フェイクデータは現実的に** — 「Test Todo 1」「Sample Item 2」のようなデータは却下、作り直す
4. **制約は観察から** — ユーザーが「maxLength 255で」と言っても、「観察すると最大40文字なのでは？」と確認する
5. **各ステップで合意を取る** — 次のステップに進む前に「このALPSで進めていいですか？」と確認

---

## 参考ファイル（クローンして使える）

```bash
git clone https://github.com/koriym/be-semantic.git
```

- `example/story/main.md` — ストーリーのサンプル
- `example/alps/todo.xml` — ALPSプロファイルのサンプル
- `example/fake/data-50.json` — 50件フェイクデータのサンプル
- `example/schema/` — スキーマのサンプル
