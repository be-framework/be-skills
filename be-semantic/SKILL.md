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

### 作業ディレクトリ

このワークフローを使うときのみ、プロジェクト直下に **`design/`** を作り、その下に
全ての成果物を置く：

```
design/
├── story/    ストーリー（Step 1）
├── alps/     ALPS プロファイル（Step 2）
├── fake/     50 件フェイクデータ（Step 3）
└── schema/   JSON Schema（Step 5）
```

`design/` は **設計の authoritative source of truth**（手で編集、git にコミット）。
be-skeleton の `var/`（runtime tmp / cache）とは別レイヤー。

これらは be-skeleton には**同梱しない**。be-semantic を使わない Be アプリでは
不要なため。be-skeleton 同梱の `src/`、`tests/`、`bin/` とは別レイヤー。

### バージョン管理

各ステップのアウトプットを生成したら、`git add design/<path>` の候補として
ユーザーに提示する（コミット実行は必ずユーザー確認を取ってから）。これを怠ると
stop hook が「未追跡ファイルあり」で割り込み、ワークフローが中断する。

- `git add -A` / `git add .` は使わない。生成したファイル名を明示して add
- **インタラクティブモード**: 各ステップのアウトプット後に候補を提示
- **自走モード**: Step 1〜5 のアウトプットをまとめ、Step 6 直前に一括で候補を提示（ステップ境界で止まらないが、実装コードを書く前に design/ をコミットする機会を作る）

---

## 開始時の確認（必須・最初のアクション）

**このスキルが起動したら、Step 1 に進む前にこの質問を投げる。**

**スキップ条件**: ユーザーが「インタラクティブで」「合意を取りながら」等、
**インタラクティブモードを明示**している場合は質問をスキップしてよい（既に合意
フローの意思表示になっているため重複になる）。この場合はそのまま Step 1 へ進む。

**スキップしない条件**: 「自走で」「お任せで」「auto モードで」等を明示された
場合は、**むしろ必ず聞く**。ALPS / Fake の合意を飛ばすリスクがある自走モード
こそ、モード選択自体の確認を最後の安全弁として残す。

モード明示がない場合は、以下の質問を出す:

```
ストーリーから最後まで自走しますか？（Y/n）

- Y = 自走モード → 全ステップを連続実行し、完了後に design/ の成果物を一覧提示
- n = インタラクティブモード → ALPS / Fake の節目でファイルを開いて合意を取る
```

ユーザーの返答（Y / n / それに準ずる回答）を**待ってから** Step 1 に進む。
返答なしに先に進むのは禁止。

**なぜ必ず質問するか**: 意味例外（Semantic Exception）や Semantic 変数は後から変更できるが、
**ALPS と Fake は合意なしに進むと「思ったものと違うアプリ」になる**。
モード選択をユーザーに渡すこと自体が、ドメインに対する責任を人間に残す行為。

**インタラクティブモード（n）のチェックポイント**:

- **Step 2 完了時** — `asd design/alps/alps.xml` を実行して HTML を開き、
  「この ALPS で進めていいですか？」を確認
- **Step 3 完了時** — `design/fake/data-50.json` を提示し、
  「これで進めていいですか？」を確認

**自走モード（Y）でも省略不可**: Step 1（ストーリー）の「なぜなら」節
（これがなければドメインが定まらない）。

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

**アウトプット**: `design/story/main.md`

---

## Step 2：ALPSプロファイル

**AIの役割**: ストーリーからALPSプロファイルを **XML** で作成する。APIの状態遷移と内部の変容チェーンを1つのファイルにタグで区別して記述する。

**ファイル名は `alps.xml` 固定**。プロジェクト名や機能名を含めない（`weightlog.xml` や `todo.xml` ではなく `alps.xml`）。JSON 形式は使わない（XML はコメントで構造を整理できるため）。

**ツール**: `asd` コマンド

```bash
brew install alps-asd/asd/asd                    # macOS
npm install -g @alps-asd/app-state-diagram       # cross-platform

asd design/alps/alps.xml         # バリデーション + HTML生成
```

**ヘッドレス / GUI なし環境のフォールバック**:

`asd` の HTML 表示が使えない場合（CI、SSH、GUI 無効のサンドボックス等）は、
以下の MCP ツールで代替する（本リポの `be-framework-skills` プラグイン同梱の
`alps` スキル、または `mcp__alps__*` 系ツールが使える前提）:

1. `mcp__alps__validate_alps` — XML 構文とセマンティクスをバリデーション
2. `mcp__alps__alps2mermaid` — 状態遷移をテキスト Mermaid 図で出力（ターミナルでもレビュー可能）
3. `mcp__alps__alps2svg` — SVG として書き出し（画像ビューアがあれば可）

これらも使えない最終フォールバック: Claude が XML を直接読み、
**descriptor 一覧と遷移関係を要約してユーザーに提示**し、口頭で合意を取る。
ブラウザで HTML を開けなかったからといって Step 2 を黙って Step 3 へ
流さない（合意ゲートは必須）。

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

**アウトプット**: `design/alps/alps.xml` + `design/alps/alps.html`（asd生成）

**インタラクティブモードの確認**（Y モードでは省略）:

`asd` で HTML を生成・自動で開いた上で、ユーザーに **必ず確認を取る**。勝手に Step 3 へ進まない。

```
ALPS を生成しました（design/alps/alps.html を開いています）。

確認ポイント:
- API 遷移（tag="api"）はストーリーをカバーしていますか？
- 内部変容（tag="be" / become〜）の分岐は意図どおりですか？
- ディスクリプタ（属性）の漏れ・命名の違和感はありませんか？

このまま Step 3（Fake 50 件生成）へ進めていいですか？
```

---

## Step 3：フェイクデータ生成

**並列化のヒント**: ALPS 確定後、Step 3(Fake)と Step 5(Schema 草案)は **並列実行できる**。両方揃ってから Step 4(観察と合意)に進む。sub-agent を使うなら同時起動が有効。

**AIの役割**: ALPSプロファイルを読んで、50件のリアルなフェイクデータを生成する。

**重要**: ランダムなテストデータではなく、「実際のユーザーが日常的に作るようなデータ」を生成する。

**カウント単位**: 「50件」は主タクソノミー（ストーリーの主語にあたるエンティティ）の**レコード 50 件**。フィールド単位ではない。複数タクソノミーがある場合、参照される副タクソノミー（例: User、Book、Member 等）は 10〜20 件で可。

**50 件が観察で拾うべき軸**（すべてのフィールドで意識する。Step 4 の合意は
ここで取れた実例が元になる）:

- テキストの **最短と最長**（1 文字だけの投稿、長文の投稿、両方含める）
- **日本語と英語の混在**（i18n のあるドメインなら両方入れる）
- **optional フィールドの null / 空文字**（ユーザーが省略しそうなフィールドは
  実際に省略した例を混ぜる）
- **エッジケース**（境界値、絵文字、改行、極端に大きい/小さい数値）
- **通常のケース**（上記 4 つに寄せすぎない。50 件の大半は日常的な値）

**生成プロンプト**:

```
以下のALPSプロファイルに基づいて、[アプリ名]のリアルなフェイクデータを50件生成してください。
実際のユーザーが日常的に使うような内容にしてください。

[ALPSプロファイルの内容]

出力形式: JSON配列
```

**アウトプット**: `design/fake/data-50.json`

**インタラクティブモードの確認**（Y モードでは省略）:

50 件を生成したら `design/fake/data-50.json` を提示し、ユーザーに **必ず確認を取る**。勝手に Step 4 へ進まない。

```
50 件のフェイクデータを生成しました（design/fake/data-50.json）。
最初の 3 件をプレビューします:

[抜粋を 3 件表示]

確認ポイント:
- 「実際のユーザーが日常的に作るデータ」になっていますか？
- 「Test 1」「Sample A」のような無味なデータが混じっていませんか？
- カバーすべきエッジケース（境界値・null パターン）は含まれていますか？

このまま Step 4（観察と制約合意）へ進めていいですか？
```

---

## Step 4：観察と合意

**AIの役割**: フェイクデータを分析し、制約の候補をユーザーに提示する。確認を取る。

**分析項目**:

- テキストフィールドの長さ分布（最短・最長・平均）
- Optional フィールドの null 率
- 値のパターンと範囲
- エッジケース

**提示形式 — 合意チェックリスト**:

観察結果は単なるリストではなく **チェックボックス形式** で提示する。auto モードでも「何を確認すべきか」が視覚的に明確になり、ユーザーが返答漏れに気づきやすい。

**観察で上がった軸は省略せず全部チェックボックスに載せる**。「自明そう」「軽微」と AI が判断して落とすと、ユーザーが同意する機会がなくなる。フィールドごとに minLength / maxLength / required / optional / 値域 / i18n の 6 軸を走査し、観察値が特筆に値するものは必ず項目化する。

```
観察結果と合意事項:

- [ ] **maxLength の値**: todoTitle は最長 41 文字 → 80 で良いか?
- [ ] **optional フィールドの扱い**: todoMemo は 50 件中 35 件 null → optional で良いか?
- [ ] **数値の精度**: weightKg は小数 1 桁 → minimum/maximum の範囲は?
- [ ] **boolean / enum の値域**: isCompleted は true/false 混在 → そのまま boolean で良いか?
- [ ] **このまま Step 5(JSON Schema)へ進んで良いか?**
```

**必ず確認を取る**。勝手に決めない。インタラクティブモードでは **すべての項目に返答を得てから** Step 5 へ進む。

**モード別の挙動**:

- **インタラクティブモード**: 観察結果を提示したら **ユーザーの返答を待つ**（ブロッキング）。返答なしに Step 5 へ進まない
- **自走モード**: 提示はするが返答待ちはしない。観察根拠を `$comment` に残して Step 5 へ進む。ユーザーは事後に `design/schema/` を見て指摘できる

**アウトプット**: 観察メモ（次のステップで使う）

---

## Step 5：JSONスキーマ

**AIの役割**: 合意した制約をJSONスキーマとして記述する。

**原則**:

- `maxLength` は観察値ベースで設定（255のようなデフォルト値は使わない）
- `required` は「フェイクデータで常に存在するフィールド」のみ
- ALPS の `descriptor` id をプロパティ名として使う（一貫性）
- スキーマには観察根拠をコメントで記述する（`$comment`）
- **小数の粒度は `multipleOf` ではなく入力/表示レイヤで丸める** — `multipleOf: 0.1` は IEEE754 浮動小数の既知の罠で、実データの大半が validation 失敗する。スキーマは「型と範囲」までに留め、粒度はアプリ層の責務

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

**アウトプット**: `design/schema/[Taxonomy].json`（ALPS の `type="semantic"` タクソノミーごとに1ファイル、PascalCase 推奨。例: `TodoList.json`、`WeightRecord.json`）

---

## Step 6：Be実装

**Step 6 を始める前に、必ず `be/SKILL.md` を読む**。be-semantic と be は同じプラグイン (`be-framework-skills`) に同梱されており、両方ロードされている前提。Be 固有の以下の事実は be-semantic 側には書いていない:

- バリデーション例外は `DomainException` 継承必須
- セマンティック変数は **変数名 1 つにつき 1 クラス**、未登録は NOTICE で素通り
- `#[Message]` は多言語メッセージで必須

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

**完了前のセルフチェック — ALPS 整合性レビュー**:

実装が動いた後（テスト緑 + `bin/run.php` 確認の後）、`design/alps/alps.xml` を開いて Final クラスと突き合わせる:

1. Final の各 public プロパティが ALPS の descriptor に存在するか
2. ALPS にない属性（計算フィールド、派生プロパティ等）を追加していないか
3. ALPS の表現（`type="semantic"`）と Final クラス名が対応しているか

不整合があれば、以下のどちらかを選ぶ:

- ALPS に descriptor を追加（合意のうえで Step 2 を更新し、`asd` で再生成）
- Final から余計な属性を削除

**この突き合わせは、実装が動いた後の最後のゲート**。スキーマ検証では拾えないドメイン整合性（ALPS は意味の合意、スキーマは制約の合意）を、ここで目視確認する。Rule 7（ALPS にない属性を勝手に追加しない）を事後にも強制する仕組み。

---

## AIが守るべきルール

1. **ストーリーより先に技術を話させない** — 「エンドポイントは？」「DBは？」はStep 6まで出さない
2. **「なぜなら」節を省略させない** — ユーザーが省略したら「なぜそうしたいですか？」と聞く
3. **フェイクデータは現実的に** — 「Test Todo 1」「Sample Item 2」のようなデータは却下、作り直す
4. **制約は観察から** — ユーザーが「maxLength 255で」と言っても、「観察すると最大40文字なのでは？」と確認する
5. **インタラクティブモードでは ALPS / Fake で必ず合意を取る** — 詳細は冒頭「開始時の確認」を参照。自走モードでは最後にまとめて提示
6. **ストーリーが許す可能性を勝手に狭めない** — ドメインに分岐がある場合（減量/増量/維持、個人/法人、無料/有料 など）は ALPS 設計**前**にユーザーに確認する。例: 「体重を記録したい / 目標に近づいているか確認したい」というストーリーから `targetWeight < startWeight` を仮定して減量専用にしてはいけない。**「観察値の承認」より「ドメインの広がりの承認」が人間の本来の役割**。AI が暗黙に絞り込むのが最も危険な失敗モード
7. **ALPS に定義されていない属性を Final に勝手に追加しない** — `remainingKg` のような計算フィールドや派生プロパティを Step 6 の段階で発明しない。必要なら ALPS の descriptor 追加を Step 2 に戻して合意する。Final は ALPS 表現の写像であって、実装者の発想で属性を増やす場所ではない

---

## 参考ファイル（クローンして使える）

```bash
git clone https://github.com/koriym/be-semantic.git
```

- `example/story/main.md` — ストーリーのサンプル
- `example/alps/todo.xml` — ALPSプロファイルのサンプル
- `example/fake/data-50.json` — 50件フェイクデータのサンプル
- `example/schema/` — スキーマのサンプル
