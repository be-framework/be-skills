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

詳細は `semantic-ex/SKILL.md`（同プラグイン同階層）を参照。

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
├── Moment/     Diamond パターンの Moment クラス（PotentialからFinalで実現）
├── LogContext/ AbstractContextのサブクラス。Beenに記録する意味的ログイベント定義
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
// Module/AppModule.php — 基本パターン
use Be\Framework\Module\BeModule;
use Ray\Di\AbstractModule;

final class AppModule extends AbstractModule
{
    protected function configure(): void
    {
        // Semantic 変数の名前空間を BeModule で install する（必須）
        $this->install(new BeModule('MyVendor\\MyApp\\Semantic'));

        // Reason クラスや Interface バインドは必要に応じて追加する
        $this->bind(Greeting::class);
        $this->bind(PaymentGatewayInterface::class)->to(PaymentGateway::class);
    }
}
```

**重要**: `BeModule` の install は Be アプリで必須。Semantic 変数の自動マッチング（Input のパラメータ名 → Semantic クラス）はこれがないと機能しない。

CQRS使用時は `FakeQueryModule` を install する（後述の「CQRS」セクション参照）。

```php
// bin/be.php（エントリポイント）
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

### 開発ループ：composer dev / stree

skeleton のエントリは `bin/be.php` 一本。CLI 引数を BEAR.Sunday 風の URI として解釈する: `<input>?<key>=<value>&...`。Module は `MODULE` env var で選ぶ（既定 `dev`）。`MODULE=foo` → `Be\Skeleton\Module\FooModule`。

| コマンド | 等価呼び出し | 用途 |
|----------|--------------|------|
| `composer dev` | `MODULE=dev php bin/be.php 'hello?name=World'` | 既定モード。`var/log/<timestamp>.json` を書きつつ greeting を出す |
| `composer app` | `MODULE=app php bin/be.php 'hello?name=World'` | 本番モード。ログなし |
| `composer stree` | `@dev` → `vendor/bin/stree <最新log>` | 直近の log をツリー描画 |
| `composer stree:full` | 同上、verbose | 全 context key を leaf として展開 |

直接呼ぶ場合:

```bash
php bin/be.php                                          # 既定 → "Hello World" + log
php bin/be.php 'hello?name=Alice'                       # 引数を上書き
MODULE=app php bin/be.php 'hello?name=Alice'            # production-style
php bin/be.php 'order?customerId=42&items[]=P1001'      # 別 Input + 多引数
```

`parse_url` + `parse_str` で構文解析。HTTP query と同じ意味論（配列・URL エンコード対応）。将来 `be://order?...` のような URI scheme で BEAR.Sunday から呼び出しても、同じ syntax → 同じ意味、で繋がる設計。

`DevModule` は `AppModule` を install した上で `BecomingInterface` を `DevBecoming` に差し替える。`DevBecoming` は `Becoming` をラップして、呼び出しごとに `Koriym\SemanticLogger` のログを `var/log/` に吐く。

実装中は `composer stree` を繰り返すのが基本ループ：

```
$ composer stree
Hello, World!
becoming from=HelloInput be=Hello input=[name] inject=[greeting]
⎿ final=Hello prop=[greeting]
```

**記号の読み方**:

- `becoming from=X be=Y input=[...] inject=[...]` — 開側。どのInputがどのクラスに変容したか、その時の入力と注入
- `├──` — イベント（ネストした becoming や途中のログ）
- `⎿` — 閉側の継続行。ひとつの変容の締めくくり
  - `being=X` — 中間Being（まだ変容が続く）
  - `final=X` — 終端Final（ここで完了）
  - `error=<Class> message=<...>` — 失敗
- `prop=[a, b, c]` — 生成されたプロパティ名。多いと `[a, b, c +N items]` に省略

ツリーが設計どおりの変容チェーンになっているかを、コード読解ではなくログで確認する。分岐・Diamond・Momentも素直にネストで現れる。

### デバッグ：どのビューを見るか

問題に応じて使い分ける。全部 `var/log/<timestamp>.json` が単一の情報源で、`composer stree` はその **圧縮ビュー**。

| 目的 | 見る先 |
|------|--------|
| 変容チェーン全体像 | `composer stree` |
| Context の生値、Property全キー | `composer stree:full` または JSON を直接 |
| 実行時間・ボトルネック | JSON の各open下 `profile.wallTime`（ms）と `profile.xhprof[]` |
| 関数レベルの呼び出し追跡 | JSON の `profile.xdebug[].source`（file path） → xdebug trace viewer |

**`composer stree` の記号で不明瞭なとき → JSON 直読み**：

- `⎿ final=Hello prop=[greeting]` で `prop` が `[N items]` 省略されていたら、JSON の `close.context.prop` に全キーと値が載っている
- event の `[event]` マーカーだけ見えて中身が分からないときも、JSON の `events[]` の対応する context を見る

**性能劣化を追うとき → profile セクション**：

- 各 `open` ノードに `profile.wallTime`（この変容にかかった秒数）
- `profile.xhprof` にxhprofが書き出したファイルの参照（`[extension=xhprof.so](指定)` で動いてれば）
- `profile.xdebug[].source` に xdebug trace のファイルパス

xhprof/xdebug 拡張が無い環境では `profile` は空で残る。警告は無視でよい（ログ自体は正常に書かれる）。

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

**ルール**:

- 例外クラスは **`DomainException` を継承必須**（`RuntimeException` や汎用例外では `SemanticVariableException` の収集対象にならず、無言クラッシュする）
- `#[Message]` は `getErrors()->getMessages('ja')` で多言語メッセージを返したいときに必須。属性が無いと例外クラス名がそのまま返る

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

### 完成条件

1. `composer install` が成功する
2. `vendor/bin/phpunit`（または `composer test`）が **緑**になる — 「テストを書いた」だけでは不十分、**実行して通ったこと**を確認する

上記 2 つは **実際にコマンドを実行して確認する**。推論で「通るはず」と代替しない。

**スモーク確認（推奨、必須ではない）**:

テスト緑は機械的な保証にすぎない。アプリの挙動を人間の目で一度見ると、テストでは気付けない意味の崩れ（`$final->been` の Context 連鎖がログとして無意味、Final のプロパティが想定と違う等）が検出できる。

- 単純なアプリ: `bin/smoke.php` 1 本に代表シナリオ 1 つ
- 多段・多分岐アプリ: `smoke/` ディレクトリにシナリオごとのファイル（関数化しておけばテストからも呼べる）

Been を持つアプリは `$final->been` を JSON 出力して**事実連鎖が意味を持って読めるか**を目視するのが有効。Been を持たない単純な Be アプリ（hello 的）では、Final のプロパティを確認すれば足りる。

---

## 実装ルール

### Semantic変数

- クラス名がコンストラクタのパラメータ名と対応（`TodoTitle` → `$todoTitle`）
- **変数名 1 つにつき 1 クラスが必要**。`$weightKg` には `WeightKg`、`$targetWeightKg` には `TargetWeightKg`、`$startDate` には `StartDate` ── 共通の "Weight" や "Date" にまとめない
- **未登録のセマンティック変数は NOTICE を出して素通りする**（例外にはならない）。Semantic ディレクトリにクラスがあれば検証され、無ければ素通り。「バリデーションが効いていない」原因のほとんどはクラス名と変数名の不一致
- `#[Validate]` メソッドの引数はパラメータの実際の型に合わせる
- nullable（`string|null`）の場合、バリデーターも `string|null` にしてnullを早期リターン
- バリデーション例外は **`DomainException` を継承**（`RuntimeException` や汎用 `\Exception` ではない）。`SemanticVariableException` は `DomainException` のみ catch して errors に集約する仕様

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

`AbstractContext::SCHEMA_URL` は、その Context イベントの構造を定義する **JSON Schema の公開 URL**（絶対 URL）を指す。Context は production で事実ログとして流通するため、スキーマは安定した URL でホストされている前提（GitHub Pages、S3、社内の静的ホスティング等）。

```php
final class BookEnjoyedContext extends AbstractContext
{
    public const string TYPE = 'book_enjoyed';
    public const string SCHEMA_URL = 'https://example.com/schemas/book/book-enjoyed.json';
    // ...
}
```

be-semantic ワークフローで `design/schema/{Name}.json` を作ったなら、それを公開 URL に配置して（GitHub Pages 等）、その URL を `SCHEMA_URL` に書く。スキーマがまだ公開されていない場合のみ空文字 `''` を許容するが、公開前提を忘れない。

#### bin/smoke.php で目視確認する（Been を持つアプリの場合）

Been を使うアプリでは、テスト緑だけでは **事実連鎖が意味を持って読めるか** が分からない。`bin/smoke.php` を作って代表シナリオを 1 つ実行し、`$final->been` を JSON 出力して目視する：

```php
// bin/smoke.php
$injector = new Injector(new AppModule());
$becoming = $injector->getInstance(Becoming::class);
$final = ($becoming)(new FinishReadingInput(readingId: 'r-1', rating: 5));

echo json_encode($final->been, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE), "\n";
```

**Input が多いアプリ**では `bin/smoke.php` 1 本では収まらない。`smoke/` ディレクトリに複数のシナリオスクリプトを分割するのが良い（シナリオを関数化しておけばテストからも呼べる）:

```
smoke/
├── create_user.php
├── publish_article.php
└── reject_and_resubmit.php
```

namespace やクラス名を skeleton と照合せずに推測で書いて完了報告するのは禁止。実行確認のないまま「実装完了」と報告された結果は信用できない。

#### Been をデバッグツールとして使う

「なぜこの Final になったのか」が分からないとき、**`$final->been` の JSON を読む**。
変換チェーン上の各 Context が時系列で並んでいるので、どの Being で何が判断されたか、どの Context で何の事実が記録されたかが追える。

例: Branching の判定が想定と違うとき

```php
$final = ($becoming)(new PatientArrivalInput(bodyTemperature: 38.5, heartRate: 95));
echo json_encode($final->been, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE), "\n";

// 出力例:
// [
//   { "type": "triage_assessed", "urgency": "emergency", "reason": "高体温" },
//   { "type": "urgent_queued", "queueId": "QUE-...", "queuePosition": 3 }
// ]
```

`triage_assessed` の `reason` を見れば「なぜ emergency に分岐したか」が即座に分かる。
これが **デバッグ用の `var_dump` ではなく、production の事実ログ** として常時記録される。

Been を活用するコツ:

- 分岐先 Final が予想外なら、**Being の Context で判断根拠（temperature, score, threshold）を記録**しておく
- 「property を Context にエコーするだけ」を避け、**判断・選択・理由** を残す（Why / What の判断基準は前述の表を参照）
- `bin/smoke.php` や `smoke/*.php` で複数の Input パターンを試して `$been` を読み比べると、ドメイン全体の動きが見える

**変換チェーン自体をステップ実行したいとき** — `xstep` で `Becoming` の中をブレークポイントで追える:

```bash
xstep --break="vendor/be-framework/be/src/Becoming.php:53" --steps=10 -- php bin/smoke.php
```

Being から次の Final が選ばれる瞬間や、`#[Input]` パラメータが何で埋まるかが見える。
詳細は xdebug skill の `xstep` セクションを参照。

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
  - 例: Being を挟むべきか直接 Final か、Branching の判定軸をどう取るか、Final クラス名が ALPS の表現と対応しないとき
- **「実装方法」の不安** → 書ききってから相談
  - 例: コンストラクタ引数の並び、phpdoc の書き方、テストの命名、import 文の整理

どちらか迷うときは「これを選んだ結果、書いたコードを全部捨てる可能性があるか」で判定する。**ある → 設計方向、ない → 実装方法**。

---

## sub-agent に Be 実装を委譲するときの規約

main セッションが sub-agent (auto モード) に Be アプリ実装を任せるとき、
prompt に以下を明示すると無駄な action と認可プロンプトが減る。

### sub-agent を起動する条件（明示的トリガー）

単一のファイルや単一のクラスで完結する作業は sub-agent に出さず main
セッションで直接書く。以下のように **独立した N 個のアイテムに fan out
できる** ときに初めて sub-agent を使う:

- Semantic 変数クラス **複数**（変数名 → クラス名が機械的に決まるもの）
- 同じ Input チェーンに属さない **独立した Final / Being クラス複数**
- Reason/Media の **複数** インターフェース実装 + Fake フィクスチャ
- ALPS 確定後の Fake データ生成と JSON Schema 草案（2 エージェント並列）

逆に、以下は sub-agent を **使わない**:

- ひとつの Final クラスを refactor する
- テストを 1 本追加する
- 既に見えているコードの typo を直す

これを明示するのは、モデルの既定が「迷ったら sub-agent を起動」ではなく
「迷ったら自分で書く」寄りになったため。fan-out が欲しいときは上記の
条件に当てはまるかを確認してから起動する。

### Negative instructions（やらないことリスト）

- skeleton 同梱 dir に `mkdir` しない（`src/Final/`、`src/Input/`、`src/Module/`、
  `src/Reason/`、`src/Semantic/`、`src/Exception/`、`src/LogContext/`、`src/Being/`、
  `src/Moment/`、`bin/`、`tests/` は同梱されている）
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

### 並列化できる/できない

**並列化できる**(複数 sub-agent に分割可能):

- Semantic 変数クラス群(変数名 → クラス名が機械的に決まる)
- 独立した Final / Being クラス群(同じ Input チェーンに属さないもの)
- Reason/Media のインターフェース実装と Fake フィクスチャ
- ALPS 確定後の Fake データ生成と JSON Schema 草案

**逐次が自然**(main セッションが担当):

- Being チェーンの設計(「何が次の Being になるか」はドメイン判断)
- 分岐ロジックの設計(`$being` の Union 型と判定基準)
- ストーリー → ALPS の意味設計
- 観察 → 合意ステップ(ユーザー返答が同期点)

**原則**: **設計はシングルスレッド、実装はマルチエージェント**。
クラスの役割が明確という Be の特性が、そのまま「sub-agent への指示が明確」に転換できる。
