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
├── Being/      中間変換（分岐がある場合のみ）。$beingプロパティで次の変身先を決定
├── Context/    AbstractContextのサブクラス。Beenに記録するイベント定義
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

#### Reason の Case クラスは「型 + データ + 振る舞い」を持って良い

`EmergencyCase` のような分岐先 Reason クラスは、単なる空の判別子ではなく、
そのケース固有のデータや振る舞いを持って構わない。Final はそれに **委譲**する。

```php
// Reason/UrgentCase.php
final readonly class UrgentCase
{
    public string $triageCode;

    public function __construct()
    {
        $this->triageCode = 'YELLOW';  // ケース固有のデータ
    }

    /**
     * ケース固有の振る舞い。Final から委譲される。
     *
     * @return array{queueId: string, queuePosition: int, status: string}
     */
    public function queue(string $patientId): array
    {
        $position = abs(crc32($patientId)) % 5 + 1;

        return [
            'queueId' => sprintf('QUE-%s-%04d', date('Ymd'), $position),
            'queuePosition' => $position,
            'status' => 'queued',
        ];
    }
}

// Final/UrgentQueued.php
final readonly class UrgentQueued
{
    public string $queueId;
    public int $queuePosition;
    public string $status;
    public string $triageCode;

    public function __construct(
        #[Input] public UrgentCase $being,        // 判別子として注入
        #[Input] public string $patientId,
    ) {
        $result = $being->queue($patientId);     // 振る舞いを委譲
        $this->queueId = $result['queueId'];
        $this->queuePosition = $result['queuePosition'];
        $this->status = $result['status'];
        $this->triageCode = $being->triageCode;
    }
}
```

Case クラスを「ただの空マーカー」にする必要はない。**そのケースを成立させる
ためのデータ・ロジックは Case クラスが持つのが自然**で、Final は判定後の状態を
表現することに集中できる。判定ロジックは Being が、ケース固有のロジックは
Case が、結果の存在は Final が持つ、という三層分業。

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

### 完成条件は「テスト緑 + Been 目視確認」

PHPUnit が緑になっただけでは完成ではない。Beの核心は **Been の事実連鎖が人間に読めること**。
`bin/run.php` を作って実行し、`$final->been` の JSON 出力を目視で確認する（書き方は Been セクション参照）。

これを完成条件に入れないと、型は通るがログとして無意味な Context を量産してしまう。

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

### Been — 存在証明

Finalオブジェクトに `Been` をインジェクトすると、変容の来歴を保持する自己証明になる。

```php
final readonly class RegisteredUser
{
    public string $userId;
    public Been $been;

    public function __construct(
        #[Input] string $email,
        #[Inject] UserIdGenerator $idGen,
        #[Inject] Been $been,
    ) {
        $this->userId = $idGen->generate();
        $event = new UserCreatedContext(
            userId: $this->userId,
            email: $email,
        );
        $this->been = $been->with($event);

        // 自己証明：このユーザーはこのメールで作られた
        assert($event->email === $email);
    }
}
```

- `Been` は `#[Inject]` で受け取る。変容元・変容先の情報がすでに含まれている
- `with()` でプロパティに現れないコンテキスト（権限、判断理由など）を追記
- `assert()` で因果を自己検証 — 証明がテストではなくプロダクションコードにある
- イベントは `AbstractContext` のサブクラスとして定義する（TYPE, SCHEMA_URL, プロパティ）
- プロパティのエコーは無意味。Beenには「なぜそうなったか」の文脈だけを記録する

#### Context に何を含めるか — Why / What の判断基準

ログの目的は **未来の人が読んで「なぜこうなったか」を理解できる**こと。判断基準：

| 判断 | 含める？ | 例 |
|------|----------|-----|
| **識別子（ID）** | ✅ 必ず | `userId`, `orderId` — どの存在の出来事か |
| **判断の根拠** | ✅ 必ず | `rating: 5`（なぜ Enjoyed に分岐したか）、`reason: 'manual_override'` |
| **再現に必要な値** | ✅ 必ず | `finishedAt`（時刻）、`amount`（金額確定値） |
| **本文・長文データ** | ❌ 含めない | `noteText`、`articleBody` — Final がプロパティで持つ。Beenには ID だけ書いて参照可能にする |
| **入力プロパティの単純コピー** | ❌ 含めない | Final の `$bookTitle` をそのまま入れる等 |

迷ったら自問：「3ヶ月後にこのログを読んで、なぜこの存在になったかが分かるか？」

#### assert() による自己証明のパターン

`assert()` は production コードに置く**自己証明**。テストとは目的が違う（テストは外部からの確認、assert は内部の不変条件）。

```php
// パターン1: ID が正しくコピーされた
assert($event->userId === $this->userId);

// パターン2: Branching の条件に従っていることの自己証明
assert($event->rating >= 4); // BookEnjoyed なら必ず

// パターン3: 状態遷移の整合性
assert($event->status === 'confirmed');

// パターン4: 時系列の整合性
assert($event->finishedAt >= $event->startedAt);
```

ポイント：assert は「このコードがこの存在を作る限り、必ず真であるべき」ことだけを書く。
入力検証は Semantic でやる。assert は **存在が成立する論理的前提**の表明。

> **注意**: PHP の production 設定（`zend.assertions=-1`）では assert は **コード生成自体されず実行されない**。
> assert は development / staging で **論理破綻を早期に捕捉する**ためのもの。
> 入力検証や production で必須のチェックは Semantic Validator や明示的な例外で行うこと（assert に頼らない）。

#### Been の連鎖 — `#[Inject]` vs `#[Input]`

`#[Inject] Been $been` は **per-injection scope** — 毎回 **空の Been** が注入される。
A → B → C と多段で metamorphose するとき、B の `$been` には A の event は入っていない。

ただし `SemanticLogger` 自体は singleton なので、**外部のログストリームには A も B も C も全部記録される**。
in-memory の `$been` プロパティと semantic log は別物として扱う：

| 用途 | 使うもの |
|------|----------|
| 外部ログ（観測・監査） | SemanticLogger が自動で全 step 集約 |
| Final オブジェクトに **来歴を持たせたい** | `#[Input] Been $been` で前段からプロパティ経由で受け取る |

```php
// Step A: 空の Been を作って渡す
final readonly class A {
    public function __construct(
        #[Inject] public Been $been,  // 空 Been を取得
    ) {
        $this->been = $been->with(new AContext(...));
    }
}

// Step B: 前段の Been を Input で受け取って継ぐ
final readonly class B {
    public function __construct(
        #[Input] Been $been,  // ← Inject ではなく Input
    ) {
        $this->been = $been->with(new BContext(...));
    }
}
```

通常の Direct / Branching パターンでは `#[Inject]` で十分（各 Final が独立した「自分の存在証明」を持つ）。
**多段で来歴連鎖が必要な場合のみ `#[Input]` 方式に切り替える**。

#### Context の SCHEMA_URL

`AbstractContext::SCHEMA_URL` は、その Context イベントの構造を定義する **JSON Schema ファイルへの相対パス** を指す。
be-semantic ワークフローで `design/schema/{Name}.json` を作っているなら、それを直接指す：

```php
final class BookEnjoyedContext extends AbstractContext
{
    public const string TYPE = 'book_enjoyed';
    public const string SCHEMA_URL = 'design/schema/BookEnjoyedContext.json';
    // ...
}
```

スキーマがまだ無い場合のみ空文字 `''` を許容するが、Phase 1 終了までに対応する schema ファイルを置くこと。

#### bin/ で目視確認する

テスト green は機械的な保証にすぎない。Be の本質である **Been の事実連鎖が意味を持って読めるか** は、人間が一度見るまで分からない。`bin/run.php` を作って実行し、`$final->been` を JSON 出力して目視する：

```php
// bin/run.php
$injector = new Injector(new AppModule());
$becoming = $injector->getInstance(Becoming::class);
$final = ($becoming)(new FinishReadingInput(readingId: 'r-1', rating: 5));

echo json_encode($final->been, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE), "\n";
```

**完成条件**（すべてを実行して確認したうえで完了報告する）:

1. `composer install` が成功する
2. `vendor/bin/phpunit`（または `composer test`）が **緑** になる — 「テストを書いた」だけでは不十分、**実行して通ったこと**を確認する
3. `php bin/run.php` が走り、出力された `$final->been` が **意味を持って読める**（property の機械的なエコーになっていない）

namespace やクラス名を skeleton と照合せずに推測で書いて完了報告するのは禁止。実行確認のないまま「実装完了」と報告された結果は信用できない。

詳細: [意味的ログ](https://be-framework.github.io/manuals/1.0/ja/10-semantic-logging.html)

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

---

## sub-agent に Be 実装を委譲するときの規約

main セッションが sub-agent (auto モード) に Be アプリ実装を任せるとき、
prompt に以下を明示すると無駄な action と認可プロンプトが減る。

### Negative instructions（やらないことリスト）

- skeleton 同梱 dir に `mkdir` しない（`src/Final/`、`src/Input/`、`src/Module/`、
  `src/Reason/`、`src/Semantic/`、`src/Exception/`、`src/Context/`、`src/Being/`、
  `bin/`、`tests/` は同梱されている）
- README / CHANGELOG / LICENSE を勝手に作らない
- task に含まれていないリファクタを副次的に行わない
- 既存の skeleton ファイル（`HelloInput.php` 等）を消さない、必要なら namespace だけ書き換える
- `git add -A` / `git add .` を使わない、ファイル名を明示する

### SKILL_GAPS.md の必須化

**sub-agent はタスクの最後に必ず `SKILL_GAPS.md` を作成して報告する。**

実装中に SKILL.md が言及していない判断・落とし穴・曖昧さに遭遇したら、その場で
`SKILL_GAPS.md` に追記する。これがないと知見が失われ、次回 sub-agent が同じ
落とし穴に落ちる。

`SKILL_GAPS.md` のフォーマット：

```markdown
## N. <短いタイトル>

### 観察
<何が起きたか、何に迷ったか>

### 改善候補
<SKILL.md / skeleton / framework のどこに何を書けば次回防げるか>
```

main セッションは sub-agent 完了後、`SKILL_GAPS.md` を必ずレビューし、
妥当な項目を SKILL.md / skeleton にフィードバックする。これがスキル改善ループ。
