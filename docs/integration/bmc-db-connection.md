# BMC DB 接続

> **Version**: 2.0
> **Last Updated**: 2026-05-03
> **スキーマ**: [schemas/bmc-db-schema.md](../../schemas/bmc-db-schema.md)
> **Relation 設計**: [schemas/relations.md](../../schemas/relations.md)

---

## 概要

Business Model Canvas（BMC）と VPC DB を双方向 Relation で接続する。BMC は事業単位の最上位戦略層であり、VPC の Customer Segments / Value Propositions を集約する。Insight DB からは Job → VPC を介して間接接続される。

## DB 識別

| 項目 | 値 |
|---|---|
| DB名（Notion表示名） | `BMC` |
| 論理名（コード参照用） | `bmc` |
| Title プロパティ | `事業名` |
| Unique ID | `BMC ID`（プレフィックス: `BMC-`） |

## Relation 仕様

### BMC ↔ VPC（1対多）

| 項目 | 仕様 |
|---|---|
| BMC 側プロパティ | `関連VPC`（Relation） |
| VPC 側プロパティ | `関連BMC`（Relation） |
| 多重度 | 1対多（1 BMC → 複数 VPC） |
| 必須 | ❌ |
| 双方向 | ✅ |

### BMC → TagDictionary

| 項目 | 仕様 |
|---|---|
| BMC 側プロパティ | `タグ`（Relation） |
| 多重度 | 多対多 |
| 必須 | ❌ |

## Insight → BMC 間接接続

Insight DB から BMC DB への直接 Relation はない。Job → VPC を介して接続される。

```
Insight DB ──(job)──▶ Job DB ──(関連VPC)──▶ VPC DB ──(関連BMC)──▶ BMC DB
```

### Insight → BMC 接続フロー

1. Insight 確定（重要度 4 以上 + 確からしさ 4 以上）
2. 顧客属性 × ジャーニー段階で集計
3. BMC.Customer Segments へ反映
4. Forces 集計から BMC.Value Propositions の根拠データ化

### VPC → BMC マッピング

| VPC プロパティ | BMC 9ブロック | 備考 |
|---|---|---|
| 顧客セグメント | Customer Segments | VPC の対象顧客を BMC に集約 |
| Customer Jobs + Pains + Gains | Value Propositions | 顧客理解が価値提案の根拠 |
| Products & Services | Value Propositions | 提供物の具体化 |
| Pain Relievers / Gain Creators | Value Propositions | 価値提案の詳細 |

## BMC 作成の単位

- 1 事業 = 1 BMC が基本
- 複数サービスラインがある場合のみサービス単位に分割
- 事業ピボット時はバージョン番号を更新（旧版は `アーカイブ`）

## Notion 設定手順

1. BMC DB に `関連VPC` Relation プロパティを作成 → VPC DB を指定 → 双方向 ON
2. VPC DB 側に自動生成される Relation プロパティ名を `関連BMC` に変更
3. BMC DB に `タグ` Relation プロパティを作成 → TagDictionary を指定

## 関連ドキュメント

- [schemas/bmc-db-schema.md](../../schemas/bmc-db-schema.md) — プロパティ全量・Select 候補値
- [schemas/vpc-db-schema.md](../../schemas/vpc-db-schema.md) — 接続先 VPC DB スキーマ
- [schemas/relations.md §4](../../schemas/relations.md#4-vpc関連bmc--bmc-db) — VPC ↔ BMC Relation 詳細
