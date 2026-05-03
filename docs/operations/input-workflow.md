# 入力フロー手順（v2.0）

この手順は Notion の **insight DB（正式名の目安: `Insight`）** における新規入力の標準フローです。仕様の正本は [`SPECIFICATION.md`](../../SPECIFICATION.md)（特に §7.1）です。

## Human-in-the-Point（読み方）

- **AI（Step 5）** は候補を出すだけです。**確定の判断は人間**が行います。
- **重要度・確からしさ（Step 6）** は戦略文脈に依存するため、**必ず人間**が入力します。
- **job の紐づけ（Step 7）** は、一次情報と AI 出力を見たうえで、**人間が**選びます（作成直後から Step 6 までの間、`job` が空でも構いません）。

## 事前準備

- [`templates/insight-entry-template.md`](../../templates/insight-entry-template.md) を複製してもよい（任意）。
- 関連する Job / VPC / BMC のページがあればリンクを控える（任意）。

## 手順（SPECIFICATION.md §7.1 と同一）

### Step 1: 一次情報を入手する

- インタビューメモ、商談メモ、サポートログなど、**原文**を用意する。

### Step 2: insight DB に新規ページを作る

- Notion で新規行（ページ）を作成する。

### Step 3: `生の声` に貼り付ける

- **一次情報は改変しない**（補足は Notion コメント推奨）。
- 個人を特定しうる情報はマスキングする（例: A社、B氏）。

### Step 4: メタ情報を入れる

次を設定する（AI autofill の前提になる）。

- `情報源タイプ`
- `取得日`
- `顧客属性`
- （任意）`役職・立場` `所属部門`

### Step 5: AI autofill を実行する（4 項目）

次の順で実行する（[`SPECIFICATION.md` §4.1](../../SPECIFICATION.md#41-実行順序)）。

1. `insight名`
2. `Forces`
3. `ジャーニー段階`
4. `要約`

プロンプト本文は [`docs/ai-prompts/`](../ai-prompts/) を参照する。

### Step 6: `重要度` と `確からしさ` を人手で入れる

- **AI に判定させない**。
- 定義は [`SPECIFICATION.md`](../../SPECIFICATION.md) §3.3 を参照する。

### Step 7: `job` と `タグ` を Relation 接続する

- `job` は **このステップまでに接続**する（`レビュー状態 = 下書き` の前提）。
- 該当 job が無い場合は、先に job DB に作成してから接続する。
- 迷う場合は `unclassified_job` に仮接続し、月次レビューで整理する（放置しない）。

詳細ルール: [`schemas/relations.md`](../../schemas/relations.md#1-insightjob--job-db)

### Step 8: `レビュー状態 = 下書き` にする

- ここまでが「入力セッションの完了地点」。
- 以降はレビュー担当が [`review-workflow.md`](./review-workflow.md) に従う。

## 品質ゲート（どこまでできたかの確認）

`レビュー状態 = 下書き` にする前に、少なくとも次を満たすこと（詳細は [`SPECIFICATION.md`](../../SPECIFICATION.md) の「6.1 必須性のレベル定義」）。

- `生の声` が埋まっている
- `情報源タイプ` `取得日` `顧客属性` が埋まっている
- AI autofill 4 項目が実行済み（中身の承認はレビューで行う）
- `重要度` `確からしさ` が人手で埋まっている
- `job` が接続されている

## 参照（禁止事項・品質）

- [`docs/governance/forbidden-actions.md`](../governance/forbidden-actions.md)
- [`docs/governance/quality-criteria.md`](../governance/quality-criteria.md)
