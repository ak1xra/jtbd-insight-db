# プロンプト: Forces autofill

## 役割

与えられた **観察メモ** から Forces of Progress の各カテゴリに分類できる文を抽出・整形する。

## 入力

- インタビューメモ / 調査メモ（原文）

## 出力形式

- Push / Pull / Anxiety / Habit を見出しに、各 **箇条書き最大5件**。
- 各項目の末尾に `(事実)` または `(解釈)` を付ける。

## 制約

- 原文にない内容を埋め合わせない。不足は「（情報不足）」と明記。

---

## Notion プロパティ autofill（`Forces` / Multi-select）向けメモ

Notion の `Forces` プロパティにそのまま貼る場合は、次の方針を守る（詳細定義は [`../../schemas/property-definitions.md`](../../schemas/property-definitions.md#8-forces)）。

### 判定ルール（autofill）

- 「生の声」に明示的に該当する内容がある場合のみ選択する
- 推測で選ばない
- 該当なしの場合は何も選ばない（空欄）
- 空欄は許容される（レビュー時に、空欄が妥当かを人間が判定する）

### Human-in-the-Point（運用）

- 「該当なし」を恐れず空欄にすること。**推測で無理に選ぶより空欄の方が品質が高い**場合がある。
- `レビュー状態 = 確定` までに、次のどちらかを満たす（[`SPECIFICATION.md`](../../SPECIFICATION.md) §6.1 / §3.1 脚注）:
  - 最低 1 つの Forces を選択する
  - または、空欄の理由を `要約` の【主訴】に明記する
