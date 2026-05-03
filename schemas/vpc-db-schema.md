# VPC DB Schema

> **File**: `schemas/vpc-db-schema.md`
> **Version**: 1.0
> **Last Updated**: 2026-05-03
> **DB名**: `VPC` / **論理名**: `vpc`

## 概要
Strategyzer社の Value Proposition Canvas に準拠。Customer Profile（Jobs/Pains/Gains）と Value Map（Products/Pain Relievers/Gain Creators）を管理。job単位で1VPCを基本とする。

## 一意識別子
- Title: `VPC名`（命名規則: `[セグメント名]向け [job要約]`）
- Unique ID: `VPC ID`（自動採番、プレフィックス: `VPC-`）

## プロパティ一覧

| # | プロパティ名 | 型 | 必須 | 説明 |
|---|---|---|---|---|
| 1 | VPC名 | Title | ✅ | 「[セグメント名]向け [job要約]」形式 |
| 2 | VPC ID | Unique ID | ✅ | 自動採番（`VPC-`） |
| 3 | 対象job | Relation | ✅ | Job DB（多対1） |
| 4 | 顧客セグメント | Text | ✅ | 対象とする顧客像 |
| 5 | Customer Jobs | Text | ✅ | jobの要約 |
| 6 | Pains | Text | ✅ | 顧客の痛み |
| 7 | Gains | Text | ✅ | 顧客の利得 |
| 8 | Products & Services | Text | ✅ | 提供する製品・サービス |
| 9 | Pain Relievers | Text | ✅ | Pains を緩和する手段 |
| 10 | Gain Creators | Text | ✅ | Gains を生み出す手段 |
| 11 | Pain重要度 | Select | ❌ | High / Medium / Low |
| 12 | Gain重要度 | Select | ❌ | High / Medium / Low |
| 13 | Fit評価 | Select | ❌ | Problem-Solution Fit / Product-Market Fit / 検証中 |
| 14 | 関連BMC | Relation | ❌ | BMC DB（多対1） |
| 15 | タグ | Relation | ❌ | TagDictionary |
| 16 | ステータス | Select | ✅ | ドラフト / レビュー中 / 確定 |
| 17 | 作成日 | Date | ✅ | |
| 18 | 更新日 | Date | ❌ | |

## Select 候補値

### Pain重要度 / Gain重要度
- High / Medium / Low

### Fit評価
- Problem-Solution Fit: Pains/Gains に対し Pain Relievers/Gain Creators が適合
- Product-Market Fit: 市場検証済み
- 検証中: 評価未完了

### ステータス
- ドラフト: 作成中
- レビュー中: レビュー実施中
- 確定: 戦略立案に使用可

## Forces → VPC マッピング
Insight DB の Forces から VPC へ機械的に接続:

| Insight.Forces | VPC接続先 |
|---|---|
| Push | VPC.Pains |
| Pull | VPC.Gains |
| Anxiety | VPC.Pains（心理的痛み） |
| Habit | Job.現状代替手段（VPC直接ではない） |

## Relation 仕様

### VPC → Job（多対1）
- 多重度: 多対1（複数VPCが1つのjobに紐付く可能性）
- 必須: ✅

### VPC ↔ BMC（双方向）
- 多重度: 多対1（1BMCに複数VPC）
- 必須: ❌

### VPC → TagDictionary
- 多重度: 多対多
- 必須: ❌

## VPC作成の単位
- 1 job × 1 顧客セグメント = 1 VPC
- 同じjobでも顧客セグメントが異なれば別VPC
- 同じ顧客セグメントでも複数jobあれば複数VPC

## 推奨ビュー

### Tier 1
- All VPC (Table)
- 確定済み (Table, フィルタ: ステータス=確定)

### Tier 2
- Fit評価別 (Board, グループ: Fit評価)
- job別 (Table, グループ: 対象job)

## 関連ドキュメント
- [SPECIFICATION.md](../SPECIFICATION.md)
- [job-db-schema.md](./job-db-schema.md)
- [bmc-db-schema.md](./bmc-db-schema.md)
- [docs/integration/vpc-db-connection.md](../docs/integration/vpc-db-connection.md)
