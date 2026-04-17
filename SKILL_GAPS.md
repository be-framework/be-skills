# SKILL_GAPS.md

be-semantic / semantic-ex スキルをインタラクティブセッションで使用した際に観察した課題と改善記録。

---

## 1. Git コミットタイミング未定義

### 観察
stop hook（`~/.claude/stop-hook-git-check.sh`）が「未追跡ファイルあり」と毎ステップ後に割り込み、ワークフローが中断された。`be-semantic` の SKILL.md はどのタイミングで `git add / commit` すべきか全く触れていなかった。

### 改善
各ステップの「アウトプット」欄に `git` コマンドを明記した（`be-semantic/SKILL.md`）。

```
**git**: `git add design/story/ && git commit -m "Add user story"`
```

---

## 2. `semantic-ex` と `be-semantic` のファイルパス不整合

### 観察
`semantic-ex/SKILL.md` は成果物を `story/fake-data-50.json` / `story/fake-data.md` / `schema/` に保存すると指示していた。
一方 `be-semantic/SKILL.md` は `design/fake/data-50.json` / `design/schema/` を使う。
be-semantic の中で semantic-ex を使うと、パスが衝突して混乱する。

### 改善
`semantic-ex/SKILL.md` の各パス指定に「スタンドアロン使用時」と「be-semantic ワークフロー内で使用時」を明記した。

---

## 3. 開始時の Y/n 質問がユーザー指定と重複する

### 観察
スキル起動時に「例外なく必ず Y/n を質問する」と書かれていたが、ユーザーがすでに「インタラクティブモードで」と指定している場合、確認が冗長になった。

### 改善
「ユーザーがすでにモードを明示している場合はその指示に従う。明示がない場合のみ質問する」に変更した（`be-semantic/SKILL.md`）。

---

## 4. `asd` コマンドのヘッドレス環境対応

### 観察
`asd design/alps/alps.xml` はブラウザで HTML を開く前提で書かれていたが、リモートサーバーや Web IDE ではブラウザが使えない。SKILL.md にフォールバック手順がなかった。

### 改善
`be-semantic/SKILL.md` に「ヘッドレス環境では `asd` でバリデーションと HTML 生成だけ行い、ALPS の構造をテキストで要約して確認を取る」フォールバックを追記した。
