# Job DB 接続

> **Version**: 2.0
> **Last Updated**: 2026-05-03
> **スキーマ**: [schemas/job-db-schema.md](../../schemas/job-db-schema.md)
> **Relation 設計**: [schemas/relations.md](../../schemas/relations.md)

---

## 概要

Insight DB と Job DB を双方向 Relation で接続し、顧客の「片付けたい用事（Job）」単位で insight を集約する。

## DB 識別

| 項目 | 値 |
|---|---|
| DB名（Notion表示名） | `Job` |
| 論理名（コード参照用） | `job` |
| Title プロパティ | `job_summary` |
| Unique ID | `Job ID`（プレフィックス: `JOB-`） |

## Relation 仕様

### Insight ↔ Job

| 項目 | 仕様 |
|---|---|
| Insight 側プロパティ | `job`（Relation） |
| Job 側プロパティ | `関連insight`（Relation） |
| 多重度 | 多対1（複数 insight → 1 job） |
| 必須 | ✅（段階的: `レビュー状態 = 下書き` 前までに接続） |
| 双方向 | ✅ |

### Job ↔ VPC

| 項目 | 仕様 |
|---|---|
| Job 側プロパティ | `関連VPC`（Relation） |
| VPC 側プロパティ | `対象job`（Relation） |
| 多重度 | 1対多（1 job → 複数 VPC） |
| 必須 | ❌ |
| 双方向 | ✅ |

### Job → TagDictionary

| 項目 | 仕様 |
|---|---|
| Job 側プロパティ | `タグ`（Relation） |
| 多重度 | 多対多 |
| 必須 | ❌ |

## Notion 設定手順

1. Job DB に `関連insight` Relation プロパティを作成 → Insight DB を指定 → 双方向 ON
2. Insight DB 側に自動生成される Relation プロパティ名を `job` に変更
3. Job DB に `関連VPC` Relation プロパティを作成 → VPC DB を指定 → 双方向 ON
4. Job DB に `タグ` Relation プロパティを作成 → TagDictionary を指定

## データフロー

```
Insight DB                    Job DB                     VPC DB
┌──────────┐   job (多対1)   ┌──────────┐  関連VPC (1対多)  ┌──────────┐
│ 生の声   │ ─────────────▶ │job_summary│ ──────────────▶ │ VPC名    │
│ Forces   │                │job_statement│               │ Pains    │
│ 要約     │                │ 機会スコア │               │ Gains    │
└──────────┘                └──────────┘                └──────────┘
```

## 運用ルール

- Job 側がマスタのフィールド（`job_statement`, `situation`, `motivation`, `outcome`）は Job を正とし、Insight 側にコピーを持たない
- `unclassified_job`（job_summary = 未分類）を 1 件用意し、分類先が未定の insight の仮接続先とする
- 仮接続は月次レビューで整理する

## 集計・分析の活用

- 1 つの Job に紐づく insight 数 → Job の確度評価
- Job ごとに Forces 分布を集計 → Pain/Gain の傾向把握
- 機会スコア（ODI）で Job の優先順位を決定

## 関連ドキュメント

- [schemas/job-db-schema.md](../../schemas/job-db-schema.md) — プロパティ全量・Select 候補値
- [schemas/relations.md §1](../../schemas/relations.md#1-insightjob--job-db) — insight.job Relation 詳細
- [schemas/relations.md §3](../../schemas/relations.md#3-job関連vpc--vpc-db) — Job ↔ VPC Relation 詳細
