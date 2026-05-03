# VPC DB 接続

> **Version**: 2.0
> **Last Updated**: 2026-05-03
> **スキーマ**: [schemas/vpc-db-schema.md](../../schemas/vpc-db-schema.md)
> **Relation 設計**: [schemas/relations.md](../../schemas/relations.md)

---

## 概要

Value Proposition Canvas（VPC）と Job DB を双方向 Relation で接続し、Customer Profile（Jobs/Pains/Gains）と Value Map（Products/Pain Relievers/Gain Creators）を Job 単位で管理する。Insight DB とは Job を介して間接接続される。

## DB 識別

| 項目 | 値 |
|---|---|
| DB名（Notion表示名） | `VPC` |
| 論理名（コード参照用） | `vpc` |
| Title プロパティ | `VPC名`（`[セグメント名]向け [job要約]` 形式） |
| Unique ID | `VPC ID`（プレフィックス: `VPC-`） |

## Relation 仕様

### VPC → Job（多対1）

| 項目 | 仕様 |
|---|---|
| VPC 側プロパティ | `対象job`（Relation） |
| Job 側プロパティ | `関連VPC`（Relation） |
| 多重度 | 多対1（複数 VPC → 1 job） |
| 必須 | ✅（VPC 作成時に Job 接続必須） |
| 双方向 | ✅ |

### VPC ↔ BMC（多対1）

| 項目 | 仕様 |
|---|---|
| VPC 側プロパティ | `関連BMC`（Relation） |
| BMC 側プロパティ | `関連VPC`（Relation） |
| 多重度 | 多対1（複数 VPC → 1 BMC） |
| 必須 | ❌ |
| 双方向 | ✅ |

### VPC → TagDictionary

| 項目 | 仕様 |
|---|---|
| VPC 側プロパティ | `タグ`（Relation） |
| 多重度 | 多対多 |
| 必須 | ❌ |

## Insight → VPC 間接接続

Insight DB と VPC DB の間に直接 Relation はない。Job を介して接続される。

```
Insight DB ──(job)──▶ Job DB ──(関連VPC)──▶ VPC DB
```

### Forces → VPC マッピング

Insight の Forces ラベルから VPC の Customer Profile へ機械的に変換する。

| Insight.Forces | VPC 接続先 | 変換ロジック |
|---|---|---|
| `Push` | VPC.Pains | 不満・問題を Pain として抽出 |
| `Pull` | VPC.Gains | 期待・望む結果を Gain として抽出 |
| `Anxiety` | VPC.Pains（心理的痛み） | 不安を Pain として抽出 |
| `Habit` | Job.現状代替手段 | 現状維持要因を代替手段として記録 |

## VPC 作成の単位

- 1 job × 1 顧客セグメント = 1 VPC
- 同じ Job でも顧客セグメントが異なれば別 VPC を作成
- 同じ顧客セグメントでも複数 Job があれば複数 VPC を作成

## Notion 設定手順

1. VPC DB に `対象job` Relation プロパティを作成 → Job DB を指定 → 双方向 ON
2. Job DB 側に自動生成される Relation プロパティ名を `関連VPC` に変更
3. VPC DB に `関連BMC` Relation プロパティを作成 → BMC DB を指定 → 双方向 ON
4. VPC DB に `タグ` Relation プロパティを作成 → TagDictionary を指定

## 関連ドキュメント

- [schemas/vpc-db-schema.md](../../schemas/vpc-db-schema.md) — プロパティ全量・Select 候補値
- [schemas/job-db-schema.md](../../schemas/job-db-schema.md) — 接続先 Job DB スキーマ
- [schemas/relations.md §3](../../schemas/relations.md#3-job関連vpc--vpc-db) — Job ↔ VPC Relation 詳細
- [schemas/relations.md §4](../../schemas/relations.md#4-vpc関連bmc--bmc-db) — VPC ↔ BMC Relation 詳細
