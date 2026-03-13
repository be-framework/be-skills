---
name: be
description: "Be Framework development skill. Use when building PHP applications with the Be Framework, implementing Input/Semantic/Final patterns, CQRS with Ray.MediaQuery, or when the user mentions Be Framework, becoming pattern, or resource-oriented architecture."
---

# Be Framework Skill

Objects don't DO things—they BECOME things.

## 前提知識

Be Frameworkの概念（変換公式、意味変数、存在理由層、Potential/Momentなど）は以下を参照：

- **Be Framework概念リファレンス**: https://be-framework.github.io/llms-full.txt
- **Ray.Di（DI/AOP）**: https://ray-di.github.io/llms-full.txt
- **リポジトリ**: https://github.com/be-framework/be-framework
- **スケルトン**: https://github.com/be-framework/skeleton

このスキルは「どう作るか」に集中する。「何であるか」は上記リファレンスに委ねる。

---

## プロジェクト開始

`composer create-project` でプロジェクトを作成する。**コード生成の前に必ず**ユーザーに以下を確認すること：

1. **プロジェクト名** — ディレクトリ名に使用（例: `my-todo-app`）
2. **namespace** — vendor/package形式（例: `MyCompany\TodoApp`）

確認なしにコード生成を始めてはならない。

```bash
composer create-project be-framework/skeleton my-todo-app
```

回答に基づいて `composer.json` の `name`、`autoload.psr-4` のnamespace、および `src/` 内の全PHPファイルのnamespaceを書き換える。composer.jsonの依存関係はskeletonに含まれている。自作しない。

### skeletonのコードを正解パターンとして読む

コード生成の前に、skeletonに含まれるHello World実装を**必ず**全て読む：

- `src/Input/HelloInput.php` — `#[Be]` の書き方
- `src/Final/Hello.php` — `#[Input]` + `#[Inject]` の書き方
- `src/Semantic/Name.php` — Semantic変数の書き方
- `src/Exception/EmptyNameException.php` — ドメイン例外の書き方
- `src/Reason/Greeting.php` — Reasonクラスの書き方
- `src/Module/AppModule.php` — Ray.Di設定の書き方
- `tests/HelloTest.php` — テストの書き方
- `composer.json` — 依存関係とautoload

**llms-full.txtは概念リファレンス。skeletonは動作する正解コード。** 新しいコードを書く時は、skeletonの`use`文、FQCN、パターンをそのまま踏襲する。推測で別のnamespaceを使わない。

### 複雑なパターンはbe-patternsを参照

skeletonは最小限のHello Worldのみ。分岐、Diamond、Moment/Potential、CQRS等の実装パターンが必要なら **be-patterns** を参照する。

初回利用時にcloneしておく（キャッシュ的に使う）：

```bash
[ -d /tmp/be-patterns ] || git clone --depth 1 -b 1.x https://github.com/be-framework/be-patterns /tmp/be-patterns
```

- **パターン索引（README）**: https://raw.githubusercontent.com/be-framework/be-patterns/refs/heads/1.x/README.md

READMEにユースケース別のパターン索引がある。該当するパターンのデモを参照：

- **Minimal**: hello-world
- **Linear**: contact-form
- **Sequential Chain**: user-registration
- **Diamond**: order-processing
- **Multi-Reason Being**: blog-publishing
- **Branching**: medical-triage
- **Cascade Diamond**: loan-application
- **Complex Convergence**: insurance-claim

複雑なアプリを作る時は、該当するデモを先に読んで、そのコードを正解パターンとして踏襲する。skeletonと同じく、`use`文・FQCN・構造をそのまま参考にする。

---

## 哲学：解像度を上げるプロセス

自然言語の仕様書は解釈が揺れる。「適切な長さ」からは100通りの実装が生まれる。
`"maxLength": 80, "minLength": 1` からは1通りしか生まれない。

Beの開発フローは、各ステップで解像度を上げていく：

```
ユーザーストーリー        ← ドメインの言葉（曖昧）
  → ALPS プロファイル     ← 状態遷移とセマンティクスを形式化
  → Semantic-Ex          ← データ生成 → 観察 → 制約発見
  → JSON Schema          ← 揺るぎないゴール（解釈の余地なし）
  → Be 実装              ← スキーマに収束する実装
  → テスト
```

最終的にALPS（状態遷移のゴール）とSchema（制約のゴール）が揃えば、
実装は誰が（人間でもAIでも）やっても同じものに収束する。

### Semantic-Ex：AI時代の設計手法

従来の仕様決め：人間が想像して制約を「決める」（トップダウン）
Semantic-Ex：AIがデータを生成し、AIが観察して制約を「発見」する（ボトムアップ）

AIの特性がこのプロセスを可能にしている：

- **大量生成が得意** — 人間が50件作ると品質が落ちる。500件は無理。AIは一瞬
- **自己評価にバイアスがない** — 自分が生成したデータを別の目でフラットに評価できる
- **パターン抽出が得意** — 50件から最長文字数、null率、境界値を一瞬で抽出

人間の役割は最後の **判断（承認）** だけ。

詳細は `skills/semantic-ex/SKILL.md` を参照。

---

## 属性のimportパス

```php
use Be\Framework\Attribute\Be;        // #[Be([TargetClass::class])]
use Be\Framework\Attribute\Validate;   // #[Validate] — Semanticクラスのバリデーションメソッド
use Be\Framework\Attribute\Message;    // #[Message(['en' => '...', 'ja' => '...'])] — 例外の多言語メッセージ
use Ray\InputQuery\Attribute\Input;    // #[Input] — 前段からのデータ受け取り
use Ray\Di\Di\Inject;                  // #[Inject] — DI注入
```

---

## プロジェクト構造

```
src/
├── Input/      起点。#[Be([TargetClass::class])] で変換先を宣言
├── Semantic/   セマンティック変数（バリデーター）。クラス名=パラメータ名(camelCase)
├── Exception/  ドメイン例外。#[Message]で多言語メッセージ
├── Final/      終点。#[Input]でデータ受け取り、#[Inject]でDI注入
├── Reason/     「その存在を可能にするもの」すべて
│   ├── Entity/ Queryの戻り値型（MediaQueryのhydration先）
│   └── Media/  CQRS: Command（書き込み）/ Query（読み取り）
└── Module/     Ray.Di設定
```

### Reason/ — 存在を可能にするもの

- **Entity/** — Queryの戻り値型。Media/の外に置く（FakeQueryModuleがMedia/をスキャンするため）

```php
// Reason/Entity/TodoEntity.php
final readonly class TodoEntity
{
    public function __construct(
        public string $todoId,    // SQL: todo_id → camelCase自動変換
        public string $todoTitle,
        public bool $isCompleted,
    ) {}
}
```
- **Media/Command/** — 書き込みインターフェース。`#[DbQuery]` 属性付き
- **Media/Query/** — 読み取りインターフェース。`#[DbQuery]` 属性付き
- **判断ロジック**（例: JTASProtocol — どの存在になるかを決める）
- **Guard**（例: 権限チェック、前提条件の判断）
- **UlidGeneratorInterface** — テスト可能なID生成

`App/` は不要。Beの標準構造で全て収まる。

### Finalでの副作用：doing for being

FinalのコンストラクタでDB保存・通知などの副作用は正当。
「永続化した存在になる」ための行動（doing for being）。

ただし常に問え：「この doing は本当にこの being に不可欠か？」

### Potential（@link phpdoc）

Finalの `_links`（ランタイムハイパーメディア）はBEAR.Sundayの `#[Link]` 属性が担う。
Beでは、次のbecomingへの潜在性をphpdocで記述する：

```php
/**
 * A todo that has become completed. This state is irreversible.
 *
 * @link GetTodoListInput(goTodoList) Return to list
 * @link DeleteTodoInput(doDeleteTodo) Delete this todo
 */
final readonly class TodoCompleted { ... }
```

フォーマット: `@link InputClassName(alpsId) 説明`

BEAR.Sundayはこのphpdocを見て `#[Link(rel: 'goTodoList', href: '/todos')]` に変換する。
Beが設計図、BEARが配線。

### オーケストレーション：BecomingInterface

複数ステップの処理もBeの中で自己完結する：

```php
interface BecomingInterface
{
    /** @return object The final form after all transformations */
    public function __invoke(object $input): object;
}
```

```php
final readonly class OrderCompleted
{
    public function __construct(
        #[Input] string $orderId,
        #[Inject] BecomingInterface $becoming,
    ) {
        ($this->becoming)(new AllocateStockInput($orderId));
        ($this->becoming)(new SendReceiptInput($orderId));
    }
}
```

BEAR.Sundayは入口（HTTP → Input）だけを担う。

### 変換パターン

| パターン | フロー | 使い分け |
|----------|--------|----------|
| Direct | Input → Final | 単純な変換 |
| Multi-stage | Input → Being → Final | 中間判断が必要 |
| Diamond | 複数並行 → 1つのFinalに収束 | 独立処理の合流 |
| Moment | Potential → `be()` で確定 | 複数外部操作のアトミック実行 |

各パターンの概念と詳細なコード例は [llms-full.txt](https://be-framework.github.io/llms-full.txt) を参照。

### Beingクラス：分岐が必要なとき

Beingは中間変換。`$being` プロパティ（Union型）で次の変身先を決定する。
分岐が不要ならBeingは使わない（Direct: Input → Final）。

```php
// Being/TriageAssessment.php
#[Be([EmergencyCase::class, ObservationCase::class])]
final readonly class TriageAssessment
{
    public Emergency|Observation $being;

    public function __construct(
        #[Input] public float $bodyTemperature,
        #[Input] public int $heartRate,
        #[Inject] JTASProtocol $protocol,
    ) {
        $urgency = $protocol->assess($bodyTemperature, $heartRate);
        $this->being = ($urgency === 'emergency')
            ? new Emergency()
            : new Observation();
    }
}
```

Frameworkが `$being` の型を見て、対応するFinalクラスを自動選択する。

### Module/AppModule と実行

```php
// Module/AppModule.php — 基本パターン（Reasonクラスをバインド）
use Ray\Di\AbstractModule;

final class AppModule extends AbstractModule
{
    protected function configure(): void
    {
        $this->bind(Greeting::class);  // 具象クラスのバインド
        $this->bind(PaymentGatewayInterface::class)->to(PaymentGateway::class);  // インターフェースバインド
    }
}
```

CQRS使用時は `FakeQueryModule` を install する（後述の「CQRS」セクション参照）。

```php
// bin/app.php（エントリポイント）
$injector = new Injector(new AppModule());
$becoming = new Becoming($injector, __NAMESPACE__ . '\\Semantic');
// 第2引数はSemanticクラスの名前空間。Input のパラメータ名と Semantic クラス名を自動マッチングする。

$result = $becoming(new CreateTodoInput(todoTitle: '牛乳を買う'));
echo $result->todoId;
```

テストでは DI 経由で取得：

```php
$injector = new Injector(new AppModule());
$becoming = $injector->getInstance(Becoming::class);
$final = ($becoming)(new HelloInput(name: 'World'));
```

### エラーハンドリング

Semantic変数のバリデーション失敗は `SemanticVariableException` に自動収集される：

```php
try {
    $result = $becoming(new CreateTodoInput(todoTitle: ''));
} catch (SemanticVariableException $e) {
    // 全バリデーションエラーが一括で収集される
    $messages = $e->getErrors()->getMessages('ja');
    // ["タイトルは空にできません"]
}
```

例外クラスに `#[Message]` で多言語メッセージを定義：

```php
#[Message([
    'en' => 'Title cannot be empty.',
    'ja' => 'タイトルは空にできません。',
])]
final class EmptyTodoTitleException extends DomainException {}
```

---

## DB操作：CQRS + Ray.MediaQuery

リポジトリパターンではなくCQRSを採用。DB操作は `Reason/Media/` のインターフェースで行う。

- **Ray.MediaQuery**: https://github.com/ray-di/Ray.MediaQuery
- **Ray.FakeQuery**: https://github.com/ray-di/Ray.FakeQuery （Phase 1用）
- **Ray.WebQuery**: https://github.com/ray-di/Ray.WebQuery （外部API用）

### Beプロジェクトでの配置

| 種類 | 配置 | 役割 |
|------|------|------|
| Command | `Reason/Media/Command/` | 書き込みインターフェース |
| Query | `Reason/Media/Query/` | 読み取りインターフェース |
| Entity | `Reason/Entity/` | Queryの戻り値型（hydration先） |
| SQL | `var/sql/` | SQLファイル（Phase 2で追加） |
| Fake | `var/fake/` | FakeQueryフィクスチャ（Phase 1用） |

Entity/はMedia/の外に置く（FakeQueryModuleがMedia/をスキャンするため）。

### FakeQueryフィクスチャ形式

- `var/fake/{query_id}.json` — 単一エンティティ
- `var/fake/{query_id}.jsonl` — コレクション（1行1オブジェクト）
- voidメソッド（Command）はファイル不要（no-opとして動作）
- snake_case → camelCase は自動変換

### 開発フェーズ

**Phase 1: Be + FakeQuery** — ドメインロジック、DBなし

- `FakeQueryModule` でMedia/インターフェースをフェイク実装に束縛
- `var/fake/` のJSONフィクスチャがQueryの戻り値になる
- Commandはno-op。ドメインロジックとテストに集中

**Phase 2: Be + MediaQuery** — DB接続

- ローカルDB環境のセットアップ（例: [Malt](https://koriym.github.io/homebrew-malt/llms-full.txt)、Docker等）
- `FakeQueryModule` → `MediaQueryModule` に切り替え
- `var/sql/{query_id}.sql` を追加（ファイル名はDbQueryのidと一致）
- Entityのプロパティ名とSQLカラム名を合わせる（snake_case → camelCase自動変換）
- src/側のコードは変更不要

**Phase 3: BEAR.Sunday + Be** — HTTP層

- ResourceクラスがBecomingを呼び出す入口になる
- `@link` phpdoc → `#[Link]` 属性に変換
- BEAR.Sundayの詳細: https://bearsunday.github.io/

---

## テストの哲学

**存在の生成 = テスト完了。**

`TodoCompleted` が例外なく生成された = 完了した存在が生まれた。
その後にDBを読み戻して確認するのは「存在を疑う」行為 — CRUDテストの発想。

Beのテストは：

- Finalが正しいクラスとして生成されること
- プロパティが期待値を持つこと
- Semanticバリデーションが不正入力を拒否すること

```php
class TodoCompletedTest extends TestCase
{
    private BecomingInterface $becoming;

    protected function setUp(): void
    {
        $injector = new Injector(new AppModule());
        $this->becoming = $injector->getInstance(BecomingInterface::class);
    }

    public function testComplete(): void
    {
        $final = ($this->becoming)(new CompleteTodoInput('todo-123'));

        $this->assertInstanceOf(TodoCompleted::class, $final);
        $this->assertSame('todo-123', $final->todoId);
    }

    public function testRejectsEmptyId(): void
    {
        $this->expectException(SemanticVariableException::class);
        ($this->becoming)(new CompleteTodoInput(''));
    }
}
```

テストは作成したら必ず実行して全てパスすることを確認する。

Command/Queryの結合テストはMedia層の責任として分離する。

---

## 実装ルール

### Semantic変数

- クラス名がコンストラクタのパラメータ名と対応（`TodoTitle` → `$todoTitle`）
- `#[Validate]` メソッドの引数はパラメータの実際の型に合わせる
- nullable（`string|null`）の場合、バリデーターも `string|null` にしてnullを早期リターン

### Finalクラス

- `final readonly class`
- `#[Input]` で前段のデータを受け取る
- `#[Inject]` でDI（Command/Queryなど）を受け取る
- `@link` phpdocでPotential（次のbecomingへの潜在性）を記述

### ULIDの生成

- Reason/UlidGeneratorInterface + Reason/UlidGenerator として分離（テスタビリティ）
- Crockford's Base32: `0123456789ABCDEFGHJKMNPQRSTVWXYZ`
- `base_convert` は使わない（I, L, O, U を生成する）

---

## 実装前に相談すべき設計判断

1. **Being（中間変換）が必要かどうか** — Direct か Branching かを事前に決める
2. **エラーハンドリングの方針** — Semantic例外と業務例外をどう分けるか

---

## 実装中の姿勢

- **「設計方向」の不安** → 中断して相談（方向が間違ってると全部やり直し）
- **「実装方法」の不安** → 書ききってから相談
