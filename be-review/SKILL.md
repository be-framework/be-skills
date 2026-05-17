---
name: be-review
description: "Independent code review for Be Framework domain code. Audits both structural compliance (final readonly, Be/Input/Inject placement, layer locations) and judgment-level anti-patterns (lying-Being, Diamond mis-classification, Service-named Beings, hidden branching, Reason logic leaks). Use when the user says 'be-review', 'Beレビュー', 'Be原則チェック', or asks to audit Be Framework code against the principles in the be skill."
---

# Be Framework Review Skill

Be で書かれたコードを独立した視点で監査するためのスキル。**実装者ではなくレビュアー**として動く。コードを「動くか」ではなく「Be の原則と判断ガイドに従っているか」で評価する。

## 前提知識

- `be-framework-skills:be` SKILL.md（特に「実装ルール」と「設計判断ガイド」セクション）
- `be-patterns` の 8 デモ（正解パターン）: `https://github.com/be-framework/be-patterns`
- Be Framework 概念: `https://be-framework.github.io/llms-full.txt`

このスキルは be SKILL.md の **判断ガイド** を 1:1 で **検出ルール**に翻訳したもの。設計判断ガイドが「どう書くか」、本スキルが「書いたものに何が紛れているか」を担う。

---

## 視点の分離

レビュアーは実装者を疑ってかかる。「実装者は anti-pattern を踏んでいるかもしれない」を出発点にする。**疑わしきは指摘**、ただし指摘は **根拠 (該当コード)** を伴う。

「動作するか」「テストが通るか」は別レイヤー（テストランナーの責務）。本スキルは **構造と判断** だけを見る。

---

## レビューフロー

1. **対象を特定** — 引数で渡されたパス、または直近のコミットで変更された `src/Input/`、`src/Being/`, `src/Final/`, `src/Semantic/`, `src/Exception/`, `src/Reason/` 配下のファイル
2. **構造チェック (Section A)** を機械的に通す
3. **判断チェック (Section B)** で anti-pattern を探す（最重要）
4. **テスト存在チェック (Section C)** をかける
5. JSON で結果を返却（後述）

---

## A. 構造チェック

機械的に判定可能なルール。違反 = blocking。

### A1. クラス宣言

- Final / Being / Input / Semantic / Exception / Reason Entity は `final readonly class` か
- mutable property（`public string $x` で readonly でない、setter がある）を持たないか

### A2. メソッド命名

- `do*` / `execute*` / `process*` / `perform*` 等の動詞メソッドが無いか（「Objects don't DO things — they BECOME things」違反）
- public メソッドは constructor とアクセサ的なもののみが望ましい

### A3. 属性配置

- Input は `#[Be([TargetClass::class])]` を持つ
- Being / Final のコンストラクタ引数は `#[Input]` または `#[Inject]` のいずれかを持つ
- Semantic クラスは `#[Validate]` メソッドを持つ
- Exception クラスは `DomainException` 継承 + `#[Message(['en' => ..., 'ja' => ...])]` を持つ
- `#[Message]` の言語 key が少なくとも 1 つ（通常 'en' と 'ja'）含まれている

### A4. 層の配置

| クラスの種類 | 配置 |
|---|---|
| Input | `src/Input/` |
| Final | `src/Final/` |
| Being | `src/Being/` |
| Semantic | `src/Semantic/` |
| Exception | `src/Exception/` |
| Entity | `src/Reason/Entity/`（Media の**外**） |
| Query / Command | `src/Reason/Media/Query/` または `src/Reason/Media/Command/` |

- Entity が `Reason/Media/` の中にある場合は **blocking**（FakeQueryModule のスキャン対象になり Phase 1 のフィクスチャが壊れる）

### A5. PHP / 汎用 lint

- `else` 句を使っていないか（early return で書き換え可）
- 汎用例外（`InvalidArgumentException`, `RuntimeException`, generic `\Exception`, `LogicException`）を `throw` していないか
- Semantic クラスの validate 例外が `DomainException` を継承していないか（継承していなければ `SemanticVariableException` の収集対象にならず無言クラッシュする）
- `var_dump` / `print_r` / `die` / `exit` の debug コードが残っていないか
- `@codeCoverageIgnore` を安易に使っていないか

### A6. Semantic 変数

- クラス名（UpperCamelCase）= コンストラクタ引数名（lowerCamelCase）の対応
- **変数名 1 つ = 1 クラス**。`$weightKg` と `$targetWeightKg` を共通の `Weight` で済ませていないか
- nullable パラメータ（`string|null` 等）の場合、validate 側も `string|null` を受けて null は早期 return しているか

---

## B. 判断チェック — Anti-pattern 検出（最重要）

be SKILL.md「設計判断ガイド」で定義された anti-pattern を発見する。これが本スキルの中核。

### B1. Lying-Being — Being が orchestrator になっていないか

**症状**: Being の constructor が `#[Inject]` した service への委譲だけで、自身では計算していない。

```php
// ❌ 嘘の Being
#[Be([CartItemAdded::class])]
final readonly class CartMerged {
    public CartEntity $mergedCart;
    public function __construct(
        #[Input] ...,
        #[Inject] CartMergerInterface $merger,
    ) {
        $this->mergedCart = $merger->merge(...);  // 委譲だけ
    }
}
```

**検出方法**: Being / Final の constructor を読み、以下のパターンを探す:

- `$this->X = $injectedService->Y(...)` の代入だけで終わっている
- 自身の constructor 内に foreach / array_map / 算術 / 条件分岐などの **state を構築するロジック** が一切ない
- `#[Input]` プロパティのコピー + `#[Inject]` service への委譲のみ

**指摘文例**: 「`CartMerged` の constructor は `$merger->merge()` への委譲だけで、自身では何も計算していない。Being が嘘になっている。Reason に出している純粋計算を constructor に戻すべき」

**対処**: 純粋計算を Being の constructor に戻し、Reason は外界対話（DB / API / 時刻 / 副作用）のみに留める。

### B2. Service 名の Being / Final

**症状**: クラス名が動詞 / 役割名（Service 語彙）で、状態名（state 語彙）ではない。

| ❌ Service 名 | ✅ State 名 |
|---|---|
| `CartMerger` | `CartMerged` |
| `OrderProcessor` | `OrderProcessed` |
| `PaymentExecutor` | `PaymentExecuted` |
| `UserValidator` | `UserValidated` |
| `EmailSender` | `EmailSent` |

**検出方法**: `src/Being/*.php` と `src/Final/*.php` のクラス名末尾を見る。`-er` / `-or` / `-Service` / `-Manager` / `-Handler` / `-Processor` で終わるものは要疑問。

**例外**: `Reason/` 配下のサービス・判断ロジック (例: `JTASProtocol`, `UlidGenerator`) は service 語彙で良い。あくまで Being / Final のクラス名が対象。

### B3. 段数だけで Diamond を称してないか — 独立性テスト

**症状**: Cascade chain (sequential dependency) を Cascade Diamond と分類している。

**判定基準**: 上流 Being の **順序を入れ替えても等価か**?

- 順序入れ替え可能 → Cascade Diamond
- 順序が意味を持つ（B が A の `#[Input]` を受ける） → Cascade chain（Diamond ではない）

**検出方法**:

1. docblock / コメント / `HANDOVER.md` / `README.md` で "Diamond" / "Cascade Diamond" を主張しているクラスを抽出
2. その Final の `#[Input]` の出所を辿る（どの Being から来ているか）
3. 上流の Being 間に `#[Input]` 依存があれば → 順序固定 → Diamond ではない

**指摘文例**: 「`CartItemAdded` の docblock は Cascade Diamond と主張しているが、`QuantityAdjusted` → `CartMerged` → `CartItemAdded` は sequential cascade chain（CartMerged は QuantityAdjusted の出力を `#[Input]` で受けている）。順序を入れ替えると動かない。Linear/Cascade が正しい分類」

### B4. Reason に純粋ロジックが漏れていないか

**症状**: Reason 実装（Query / Command / Service interface の実装クラス）のメソッド内で、外界対話以外の純粋計算が行われている。

**検出対象**: 以下のロジックが Reason の実装メソッド内にあれば疑い:

- `foreach` でループしながら累積する計算（合計、merge、cap 適用）
- `array_map` / `array_sum` / `array_filter` / `array_reduce` での集計
- 条件分岐 (`if` / match) で値を変える純粋計算
- ドメインルールに基づく値の組み立て

これらは Being / Final の constructor にあるべきロジック。Reason に置くと Being が空になる anti-pattern（B1）と表裏一体。

**例外**: SQL を発行する Query / Command の interface 自体は実装が空（`#[DbQuery]` 付きの abstract）なので対象外。Fake 実装で foreach が出てきても、それは fixture を返すための glue なので OK。

**指摘文例**: 「`CartMerger::merge()` の中で foreach で既存カートをマージし、array_sum で totalPrice を計算している。これは Being の存在理由そのもの。`CartMerged` Being の constructor に戻すべき」

### B5. 隠れた分岐 — `??` / `if-else` で構造的に異なる state を吸収していないか

**症状**: Being / Final の constructor 内で、根本的に異なる state を `??` や `if-else` で分岐している。

```php
// ⚠️ 隠れた分岐の例
$existingCart = $cartQuery->byCartKey($cartKey)
    ?? new CartEntity(...);  // 「存在しない」case を constructor 内で吸収

// 後続のロジックが 2 ケースを両方扱おうとして肥大化していく
```

**検出方法**: constructor 内に以下があれば候補:

- `?? new SomethingElse(...)` で fallback object を作っている
- `if ($x === null) { ... } else { ... }` / `match (true) { ... }` で構造的に異なる state を構築している
- 複数の業務ルールが case 別に走る

**対処判定**: 各分岐が独立した state 名を持てるなら `#[Be([A::class, B::class])]` で割る。例:

- `existingCart === null` → `FreshCartCreated`
- `existingCart !== null` → `CartMerged`

下流（Final）で同じ振る舞いに合流するだけ、かつ各分岐の constructor が短いなら現状維持で良い（早すぎる Branching は意味の凝集を落とす）。

**指摘の重み**: 必ずしも blocking ではない。「Branching に出す価値があるか」は domain 判断。findings として記録し、必要に応じて blocking に昇格。

### B6. pre-persistence ロジックが Final に集中していないか

**症状**: Final の constructor が persistence（`$repo->save(...)`、`$command->persist(...)`）**以外** の計算で肥大化している。

**検出方法**: Final の constructor を読み、以下を確認:

- constructor の行数（コメント / 引数定義除いて、実際のロジック）
- persistence 呼び出し以外の計算（merge loop, total 計算, 集計）が占める割合

ロジックが大きく占めているなら、新しい Being を 1 段挟んで pre-persistence の純粋計算をそちらに移すべき。Final は **persistence の証拠** のみに留めるのが Be Framework の哲学。

**指摘文例**: 「`CartItemAdded` の constructor は 50 行あり、`$cartCommand->save()` 以外に merge loop と totalPrice 計算と deliveryFee 集計を含む。これらを新規 `CartMerged` Being に分離して、Final は persistence のみに薄くするべき」

### B7. その他の Be 原則違反

- `do*` プレフィックスのメソッドを持つ（B2 と被るが、メソッド単位で検出）
- 状態変更（setter, public 非 readonly プロパティ）を持つ
- Input が `#[Be([...])]` を持たない（中継できない）
- Input 内で副作用（DB 書き込み等）を行っている（Final の責務）

---

## C. テストチェック

- 存在の生成がテストされているか（`assertInstanceOf(SomeFinal::class, $final)`）
- DB を読み戻して確認する CRUD 的テストになっていないか（存在 = テスト完了の Be 哲学）
- Semantic バリデーション失敗が `SemanticVariableException` でテストされているか
- `composer test` が緑か（**実行確認**、推論ではない）

---

## D. 機械的検出のヒント (grep / ast)

判断系（Section B）は文脈読みが必要だが、最初のスクリーニングには grep が使える:

```bash
# B2 候補: Service 名の Being / Final
grep -rEn 'final readonly class [A-Z][a-zA-Z]+(er|or|Service|Manager|Handler|Processor)\b' src/Being src/Final

# B5 候補: 隠れた分岐
grep -rn '?? new ' src/Being src/Final

# A2: do* メソッド
grep -rEn 'public function (do|execute|process|perform)[A-Z]' src/Final src/Being src/Input

# A5: 汎用例外
grep -rEn 'throw new (InvalidArgumentException|RuntimeException|LogicException|\\\\Exception)\b' src/

# B4 候補: Reason に foreach (Fake は除外)
find src/Reason -name '*.php' -not -path '*Fake*' | xargs grep -l 'foreach\|array_sum\|array_map'
```

これらは候補出しで、最終判断は文脈読みが必須。

---

## レビュー結果の出力

以下の JSON 形式で返答する。これ以外の形式は認められない。

```json
{
  "verdict": "pass | fail",
  "findings": [
    "セクション番号. 問題の要約（blocking でないものも含む全ての気づき）"
  ],
  "blocking": [
    "差し戻し必須の重大問題のみ"
  ]
}
```

### 判定基準

- `blocking` が空 → `verdict: "pass"`
- `blocking` に 1 件以上 → `verdict: "fail"`

### blocking に入れるべき問題

- **B 系 anti-pattern のうち確定したもの** — Lying-Being / Service 名 Being / Diamond 誤分類 / Reason ロジック漏れ / Final 肥大
- **A 系構造違反** — `final readonly` でない、do* メソッドがある、汎用例外を throw、Entity が Media 内
- **テストが失敗している** or **存在しない**

### findings に入れる（blocking にしない）問題

- 命名の微妙な揺れ
- コメント / phpdoc の不足
- 最適化の余地（動作に影響しない）
- 将来の検討事項
- 判断が分かれる anti-pattern（例: B5「Branching に割るべきかも」だが現状で明確に壊れていない）
- B5 の隠れた分岐で、現状コードが短く済んでいるケース

### 指摘の書式

各 finding / blocking は以下の構造で書く:

```
{セクション番号}. {クラス名}::{メソッド名 or プロパティ}: {anti-pattern 名} — {根拠 (ファイル:行)} — {対処方針}
```

例:

```
B1. CartMerged::__construct: Lying-Being — be/src/Being/CartMerged.php:34 で `$this->mergedCart = $merger->merge(...)` だけ。constructor 内に計算が無い — merge ロジックを Reason から constructor に戻す
```

---

## 使い方

### 主セッションから sub-agent に委譲する場合

このスキルを sub-agent prompt に明示し、対象パスを引数として渡す:

```text
be-review skill に従って be/src/ 配下を監査してください。出力は JSON のみ。
```

sub-agent は本 SKILL.md の Section A → B → C の順にチェックを通し、JSON で返却する。

### main セッションが直接使う場合

ユーザーが「Be レビューして」と言った場合、本 SKILL.md の構造に沿って対象コードを読み、JSON ではなく **markdown の人間向けレポート** で返しても良い。判定基準と anti-pattern の名前は同じものを使う。
