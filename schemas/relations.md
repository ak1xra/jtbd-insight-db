# Relations

> **File**: `schemas/relations.md`
> **Version**: 2.0
> **Last Updated**: 2026-05-03
> **Related**: [SPECIFICATION.md](../SPECIFICATION.md) §5 / [property-definitions.md](./property-definitions.md)

---

## 概要

`insight` DB と他DBとの Relation 設計を定義する。

---

## Relation 全体図

```
┌──────────────────────────────────┐
│         insight DB               │
│  (一次情報・Forces・ジャーニー段階) │
└────────┬───────────────┬─────────┘
         │               │
         │ job           │ タグ
         │ (多対1)       │ (多対多)
         ▼               ▼
   ┌──────────┐   ┌─────────────────┐
   │  job DB  │   │ TagDictionary DB │
   └────┬─────┘   └─────────────────┘
        │
        │ (多対1)
        ▼
   ┌──────────┐
   │  VPC DB  │
   └────┬─────┘
        │
        │ (多対1)
        ▼
   ┌──────────┐
   │  BMC DB  │
   └──────────┘
```

---

## Relation 一覧

| # | Relation名 | 接続元 | 接続先 | 多重度 | 必須 | 双方向 |
|---|---|---|---|---|---|---|
| 1 | [insight.job](#1-insightjob--job-db) | insight | job | 多対1 | ✅（段階的） | ✅ |
| 2 | [insight.タグ](#2-insightタグ--tagdictionary) | insight | TagDictionary | 多対多 | ❌ | ✅ |

---

## 1. insight.job → job DB

### 概要
insightが紐づく顧客のJobを示す。本DBにおいて最重要のRelation。

### 仕様

| 項目 | 仕様 |
|---|---|
| **接続元プロパティ** | `insight.job` |
| **接続先DB** | `job` |
| **接続先プロパティ** | `job.id`（または job のTitle） |
| **多重度** | 多対1（1つのinsightは1つのjobに紐づく） |
| **必須** | ✅（段階的。`レビュー状態 = 下書き` 前までに接続。作成直後は空可） |
| **双方向** | ✅（job側に `関連insight` プロパティ） |
| **削除時の挙動** | jobが削除されたら、紐づくinsightの本Relationを空にする |

### 設計理由

- JTBD理論では、insightは特定のJobに紐づく一次情報
- 1つのinsightが複数のJobに同時に紐づくことは原則ない
- 複数Jobに関連する場合は、別エントリとして登録するか、`タグ` で補足

### job DB側の対応プロパティ

```
job DB:
  - id (Title)
  - job_statement (Text)  ※ "When [situation], I want to [motivation], so I can [outcome]" 形式
  - 関連insight (Relation → insight DB) ★双方向
  - VPC (Relation → VPC DB)
  - 重要度 (Number)
  ...
```

### 必須性のレベル定義

`job` Relation の必須性は、**時間軸（= 入力ステップ）** で変化する（[`SPECIFICATION.md`](../SPECIFICATION.md) §7.1 と整合）。

| タイミング | 必須性 | 備考 |
|---|---|---|
| insight 作成直後（Step 2） | 任意 | 未接続でもよい |
| AI autofill 完了時（Step 5） | 任意 | `Forces` / `要約` を見てから選ぶことを推奨 |
| **`レビュー状態 = 下書き` にする前（Step 7〜8）** | **必須** | 接続済みであること |
| `レビュー状態 = 確定` 時 | 必須 | 下書き時点で満たしている前提 |
| 戦略立案・集計の実利用 | 必須 | 未接続のまま採用しない |

### 入力ルール

1. insight 作成時点では、`job` 未接続でもよい（[`SPECIFICATION.md`](../SPECIFICATION.md) §7.1 の Step 7 までに接続）
2. `レビュー状態 = 下書き` にする前に、`job` を **必ず** 接続する
3. 該当 job が既存 DB にない場合：
   - **選択肢A**: 先に job DB で新規エントリを作成して接続する
   - **選択肢B**: 仮接続として `unclassified_job` を選び、後日のレビューで整理する
4. ワークスペースには、未分類用の `unclassified_job` を 1 件用意しておく

### 推奨フロー（Step 対応）

```
[Step 2: insight 作成]
    ↓ job 未接続でOK
[Step 3〜6: 生の声・属性・AI autofill・重要度/確からしさ]
    ↓ job 未接続でOK（AI 結果を材料に job を選ぶ）
[Step 7: job・タグ Relation 接続]
    ↓ job 接続必須
[Step 8: レビュー状態 = 下書き]
    ↓ job 接続済みであること
[レビュー → 確定]
```

### 集計・分析の活用

- 1つのjobに紐づくinsight数で job の確度を評価
- jobごとに `Forces` 分布を集計し、Pain/Gain の傾向把握
- jobごとに `ジャーニー段階` 分布を集計し、ファネル分析

### NG例

- ❌ 1つの insight に複数 job を紐づける
- ❌ `レビュー状態 = 下書き` にする時点で job が未接続
- ❌ 関連性の低い job に無理に紐づける
- ❌ `unclassified_job` の仮接続のまま放置する（月次レビューで整理しない）

### OK例

- ✅ Step 2〜6 の間は job 未接続のまま作業する
- ✅ AI autofill の `Forces` / `要約` を見てから適切な job を選ぶ
- ✅ 該当 job がなければ先に job を作成する
- ✅ 判定が難しければ `unclassified_job` に一時接続し、後日整理する

---

## 2. insight.タグ → TagDictionary

### 概要
横断的な分類軸。テーマ・論点・選定基準・見送り理由・業界などを管理。

### 仕様

| 項目 | 仕様 |
|---|---|
| **接続元プロパティ** | `insight.タグ` |
| **接続先DB** | `TagDictionary` |
| **接続先プロパティ** | `TagDictionary.tag_name`（Title） |
| **多重度** | 多対多 |
| **必須** | ❌ |
| **双方向** | ✅ |
| **削除時の挙動** | タグが削除されたら、紐づくinsightの本Relationから自動削除 |

### TagDictionary DB の構造（推奨）

```
TagDictionary DB:
  - tag_name (Title)
  - category (Select)  ※ 業界 / テーマ / 選定基準 / 見送り理由 / その他
  - description (Text)
  - 関連insight (Relation → insight DB) ★双方向
  - 関連job (Relation → job DB) ※必要に応じて
  - 使用頻度 (Number)  ※ 自動カウント
```

### category 候補値

| 値 | 用途 | 例 |
|---|---|---|
| `業界` | 業界分類 | 建設、医療、IT、製造 |
| `テーマ` | 議論テーマ | 価格、機能、サポート |
| `選定基準` | v1.3から移管 | 月額予算、契約期間、SLA |
| `見送り理由` | v1.3から移管 | 予算オーバー、機能不足、社内反発 |
| `規模` | 企業規模 | 小規模、中規模、大規模 |
| `その他` | カテゴリ未定 | 自由 |

### 設計理由

- v1.3 で独立プロパティだった `選定基準` `見送り理由` をタグに統合
- 横断的に集計・フィルタリング可能
- 新しい論点が出現したら動的に追加可能

### 入力ルール

1. insight 内容に応じて該当タグを複数選択
2. 該当タグが TagDictionary にない場合：
   - TagDictionary に新規エントリ作成
   - category を設定
   - その後 insight に接続
3. 1つのinsight に5個以下のタグを推奨

### タグ作成のガイドライン

#### 良いタグ
- ✅ 具体的（「月額予算」）
- ✅ 検索しやすい（「ITリテラシー不足」）
- ✅ 一意（「クラウド化」）

#### 悪いタグ
- ❌ 抽象的すぎる（「重要」）
- ❌ 揺れがある（「クラウド」「Cloud」「クラウドサービス」）
- ❌ 主観的（「めちゃくちゃ困っている」）

### タグ運用ルール

1. **新規タグ作成は慎重に**: 既存タグで代替できないか確認
2. **タグの統合**: 類似タグが複数できたら統合
3. **使用頻度の低いタグの整理**: 6ヶ月使用されないタグは削除候補
4. **categoryの一貫性**: 同じcategoryのタグは粒度を揃える

### 集計・分析の活用

- タグ別のinsight件数 → 顧客の関心領域マップ
- タグ × `Forces` のクロス集計 → 痛み・期待のパターン分析
- タグ × `ジャーニー段階` のクロス集計 → ファネル別論点分析
- 業界タグ × 重要度 → ターゲット業界の優先順位

---

## 間接的な関連DB

直接Relationはないが、insight DBから派生・接続される下流DB。

### VPC DB（Value Proposition Canvas）

| 項目 | 内容 |
|---|---|
| **接続経路** | insight → job → VPC |
| **データ流** | insight.Forces → job.related_insight → VPC.Pains/Gains |
| **接続方法** | jobを介して間接接続 |

#### Forces → VPC マッピング

| insight.Forces | VPC接続先 | 変換ロジック |
|---|---|---|
| `Push` | VPC.Pains | 不満・問題をPainとして抽出 |
| `Pull` | VPC.Gains | 期待・望む結果をGainとして抽出 |
| `Anxiety` | VPC.Pains（心理的痛み） | 不安をPainとして抽出 |
| `Habit` | job.現状代替手段 | 現状維持要因を代替手段として記録 |

### BMC DB（Business Model Canvas）

| 項目 | 内容 |
|---|---|
| **接続経路** | insight → job → VPC → BMC |
| **データ流** | 顧客属性 × ジャーニー段階 → BMC.Customer Segments |
| **接続方法** | VPCを介して間接接続 |

#### insight → BMC 接続フロー

```
1. insight 確定（重要度4以上 + 確からしさ4以上）
   ↓
2. 顧客属性 × ジャーニー段階 で集計
   ↓
3. BMC.Customer Segments へ反映
   ↓
4. Forces 集計から BMC.Value Propositions の根拠データ化
```

---

## Relation 運用ルール

### 必須Relationの確認タイミング

- insight 入力時：`job` Relation の必須チェック
- `レビュー状態 = 確定` 時：全必須Relation の確認

### Relation の双方向性

すべてのRelation は双方向に設定：
- データの整合性確保
- 逆引き集計の容易化
- 削除時の整合性自動維持

### Relation 削除時の影響

| 削除対象 | 影響 |
|---|---|
| insight 削除 | 紐づく job/タグからの参照が自動削除 |
| job 削除 | 紐づく insight の `job` Relation が空に |
| タグ 削除 | 紐づく insight の `タグ` から自動削除 |

### Relation の見直しタイミング

- 月次：未接続のinsightをチェック
- 四半期：使用頻度の低いタグを整理
- 半年：job DB と insight DB の関係性を見直し

---

## 接続先DB の前提条件

### 必須DB

#### `job` DB
- ✅ 存在必須
- 推奨スキーマ:
```
  - id (Title)
  - job_statement (Text)
  - 関連insight (Relation → insight DB)
  - VPC (Relation → VPC DB)
  - 重要度 (Number)
```
- 初期エントリとして `unclassified_job` を1件用意

#### `TagDictionary` DB
- ✅ 存在必須
- 推奨スキーマ:
```
  - tag_name (Title)
  - category (Select)
  - description (Text)
  - 関連insight (Relation → insight DB)
  - 使用頻度 (Number)
```

### 推奨DB

#### `VPC` DB
- 推奨（戦略立案で使用）
- 詳細: [docs/integration/vpc-db-connection.md](../docs/integration/vpc-db-connection.md)

#### `BMC` DB
- 推奨（戦略立案で使用）
- 詳細: [docs/integration/bmc-db-connection.md](../docs/integration/bmc-db-connection.md)

---

## マイグレーション時の注意

### v1.3 → v2.0 移行

#### Relation変更点
- `タグ` Relation の用途拡大（v1.3の `選定基準` `見送り理由` を統合）
- `job` Relation は変更なし

#### 移行手順
1. v1.3 の `選定基準` `見送り理由` の値を抽出
2. TagDictionary に対応タグを作成（category = `選定基準` または `見送り理由`）
3. 既存insight の `タグ` Relation に該当タグを接続
4. 動作確認後、v1.3 の `選定基準` `見送り理由` プロパティを削除

詳細: [docs/operations/migration-v1-to-v2.md](../docs/operations/migration-v1-to-v2.md)

---

## トラブルシューティング

### Relation が空のinsightが多い

**症状**: `job` Relation 未接続のinsightが多数

**原因と対策**:
- jobの粒度が荒すぎる → job を細分化
- jobの粒度が細かすぎる → 抽象度を上げて統合
- `unclassified_job` への一時接続が放置 → 月次レビューで整理

### タグが乱立している

**症状**: 似たようなタグが多数できている

**原因と対策**:
- タグ作成基準が曖昧 → ガイドライン徹底
- 既存タグの確認不足 → 入力時に検索を徹底
- カテゴリ別の粒度不一致 → category別に粒度を統一

### Relationの双方向が機能しない

**症状**: insight側で接続したのに、相手側に表示されない

**原因と対策**:
- Notion設定で双方向Relation有効化を忘れている
- 一度Relationを削除し、双方向で再作成

---

## 関連ドキュメント

- [SPECIFICATION.md](../SPECIFICATION.md) - 全体仕様
- [property-definitions.md](./property-definitions.md) - プロパティ詳細
- [select-options.md](./select-options.md) - Select候補値定義
- [docs/integration/job-db-connection.md](../docs/integration/job-db-connection.md) - job DB接続詳細
- [docs/integration/vpc-db-connection.md](../docs/integration/vpc-db-connection.md) - VPC DB接続詳細
- [docs/integration/bmc-db-connection.md](../docs/integration/bmc-db-connection.md) - BMC DB接続詳細
- [docs/integration/tag-dictionary-connection.md](../docs/integration/tag-dictionary-connection.md) - TagDictionary接続詳細
- [docs/operations/migration-v1-to-v2.md](../docs/operations/migration-v1-to-v2.md) - 移行手順