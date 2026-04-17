# ユーザーストーリー：読書記録アプリ

## ストーリー

ユーザーとして、
読んだ本・読んでいる本・読みたい本を記録して、
感想と評価を残したい。

なぜなら、積ん読が増えて「この本読んだっけ？」と迷うことが多く、
読書の歴史を振り返って次に読む本を選びやすくしたいから。

## エンティティと属性

### Book（本）
- bookId: 本の識別子
- bookTitle: タイトル
- authorName: 著者名
- genre: ジャンル（小説、ビジネス、技術書、自己啓発 等）
- isbn: ISBN番号（オプション）

### ReadingRecord（読書記録）
- recordId: 記録の識別子
- bookId: 紐づく本のID
- status: 読書状態（unread=読みたい / reading=読んでいる / done=読了）
- startDate: 読み始めた日（reading/doneのとき）
- finishedDate: 読み終わった日（doneのとき）
- rating: 評価（1〜5の整数、doneのとき）
- reviewText: 感想・メモ（オプション）
