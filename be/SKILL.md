# Be Framework Skill

Beは「AIに仕事をさせるための解像度を上げるプロセス」を内包したフレームワーク。
Objects don't DO things—they BECOME things.

**リポジトリ**: https://github.com/be-framework/be-framework
**llms.txt**: https://be-framework.github.io/llms-full.txt
**アプリ雛形**: https://github.com/be-framework/app (skeletonブランチ)

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
- **自己評価にバイアスがない** — 自分が生成したデータを別の目でフラットに評価できる。人間は自分の作品にバイアスがかかる
- **パターン抽出が得意** — 50件から最長文字数、null率、境界値を一瞬で抽出

人間の役割は最後の **判断（承認）** だけ：
「80文字制限でいい？」→ いい。「nullableにする？」→ する。

詳細は `skills/semantic-ex/SKILL.md` を参照。

---

## プロジェクト構造

```
src/
├── Input/      起点。#[Be([TargetClass::class])] で変換先を宣言
├── Semantic/   セマンティック変数（バリデーター）。クラス名=パラメータ名(camelCase)
├── Final/      終点。#[Input]でデータ受け取り、#[Inject]でDI注入
├── Reason/     「その存在を可能にするもの」すべて
│   └── Media/  CQRS: Command（書き込み）/ Query（読み取り）/ Entity
└── Module/     Ray.Di設定
```

### Reason/ — 存在を可能にするもの
Reasonは「なぜReasonか？」→ 「その存在を可能にするもの」の場所。

- **判断ロジック**（例: JTASProtocol — どの存在になるかを決める）
- **Guard**（例: 権限チェック、前提条件の判断）
- **Media/**（例: Command/Query/Entity — CQRS by Ray.MediaQuery）
  - `Command/` — 書き込み。存在を永続化する
  - `Query/` — 読み取り。既存の存在を認識する（次の変換を可能にする）
  - `Entity/` — 存在の実体定義。MediaQueryのhydration先
- **UlidGeneratorInterface** — テスト可能なID生成

`App/` は不要。Beの標準構造で全て収まる。

### Finalでの副作用：doing for being
FinalのコンストラクタでDB保存・通知などの副作用は正当。
「永続化した存在になる」ための行動（doing for being）。
自動車運転者になるために自動車学校で学ぶ（doing）のと同じ。

**ただし常に問え：「この doing は本当にこの being に不可欠か？」**

### オーケストレーション：BecomingInterface
複数ステップの処理もBeの中で自己完結する。FinalがBecomingInterfaceをInjectして別の変換を起動：

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
- **Direct**: Input → Final
- **Multi-stage**: Input → Being → Final
- **Diamond Pattern**: 複数の並行パイプライン → 1つのFinalに収束
- **Moment**: 遅延実行（`be()` で実現）

---

## CQRS + Ray.MediaQuery / Ray.FakeQuery

リポジトリパターンではなくCQRSを採用：

```php
// Reason/Media/Command/TodoCommandInterface.php
interface TodoCommandInterface
{
    public function add(string $todoId, string $todoTitle, ...): void;
    public function complete(string $todoId): void;
    public function delete(string $todoId): void;
}

// Reason/Media/Query/TodoQueryInterface.php
interface TodoQueryInterface
{
    public function item(string $todoId): TodoEntity;
    /** @return array<TodoEntity> */
    public function list(string $filterStatus = 'all'): array;
}
```

**開発フェーズ：**
```
Phase 1: Be + FakeQuery / InMemory  ← ドメインロジック、DBなし
Phase 2: Be + Ray.MediaQuery + SQL  ← var/sql/ にSQLファイル追加
Phase 3: BEAR.Sunday + Be           ← HTTP層でラップ
```

`src/` にはインターフェース（契約）だけ。SQLは `var/sql/` に。
FakeQueryは `var/fake/` のJSONファイルを返す（テスト/フロントエンド開発用）。

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
- `_links` のrel名はALPSのdescriptor idをそのまま使う
- 条件付きリンクはFinalのコンストラクタで制御

### ULIDの生成
- Reason/UlidGeneratorInterface + Reason/UlidGenerator として分離（テスタビリティ）
- Crockford's Base32: `0123456789ABCDEFGHJKMNPQRSTVWXYZ`
- `base_convert` は使わない（I, L, O, U を生成する）

---

## ⚠️ 実装前に相談すべき設計判断

1. **Being（中間変換）が必要かどうか** — Direct か Branching かを事前に決める
2. **エラーハンドリングの方針** — Semantic例外と業務例外をどう分けるか

---

## 実装中の姿勢

- **「設計方向」の不安** → 中断して相談（方向が間違ってると全部やり直し）
- **「実装方法」の不安** → 書ききってから相談
