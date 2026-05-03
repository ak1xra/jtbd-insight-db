# BMC DB Schema

> **File**: `schemas/bmc-db-schema.md`
> **Version**: 1.0
> **Last Updated**: 2026-05-03
> **DB名**: `BMC` / **論理名**: `bmc`

## 概要
Osterwalder の Business Model Canvas 9ブロックに準拠。事業単位で1BMC が基本。VPC（Customer Segments / Value Propositions の根拠）から接続される最上位の戦略層。

## 一意識別子
- Title: `事業名`
- Unique ID: `BMC ID`（自動採番、プレフィックス: `BMC-`）

## プロパティ一覧

| # | プロパティ名 | 型 | 必須 | 説明 |
|---|---|---|---|---|
| 1 | 事業名 | Title | ✅ | 対象事業の名称 |
| 2 | BMC ID | Unique ID | ✅ | 自動採番（`BMC-`） |
| 3 | Customer Segments | Text | ✅ | 顧客セグメント |
| 4 | Value Propositions | Text | ✅ | 価値提案 |
| 5 | Channels | Text | ✅ | 顧客接点・チャネル |
| 6 | Customer Relationships | Text | ✅ | 顧客関係 |
| 7 | Revenue Streams | Text | ✅ | 収益の流れ |
| 8 | Key Resources | Text | ✅ | 主要リソース |
| 9 | Key Activities | Text | ✅ | 主要活動 |
| 10 | Key Partnerships | Text | ✅ | 主要パートナー |
| 11 | Cost Structure | Text | ✅ | コスト構造 |
| 12 | 関連VPC | Relation | ❌ | VPC DB（1対多） |
| 13 | 事業ステージ | Select | ❌ | アイデア / 検証 / 立ち上げ / 成長 / 成熟 |
| 14 | バージョン | Text | ✅ | v1.0 / v2.0 形式 |
| 15 | タグ | Relation | ❌ | TagDictionary |
| 16 | ステータス | Select | ✅ | ドラフト / レビュー中 / 確定 / アーカイブ |
| 17 | 作成日 | Date | ✅ | |
| 18 | 更新日 | Date | ❌ | |

## BMC 9ブロックの対応

| BMC 9ブロック | プロパティ | 説明 |
|---|---|---|
| Customer Segments | Customer Segments | 誰のために |
| Value Propositions | Value Propositions | 何の価値を |
| Channels | Channels | どう届けるか |
| Customer Relationships | Customer Relationships | どう関係を築くか |
| Revenue Streams | Revenue Streams | どう収益を得るか |
| Key Resources | Key Resources | 何が必要か |
| Key Activities | Key Activities | 何をするか |
| Key Partnerships | Key Partnerships | 誰と組むか |
| Cost Structure | Cost Structure | 何にコストがかかるか |

## Select 候補値

### 事業ステージ
- アイデア: 構想段階
- 検証: PoC / MVP実施中
- 立ち上げ: ローンチ後
- 成長: 拡大期
- 成熟: 安定期

### ステータス
- ドラフト: 作成中
- レビュー中: レビュー実施中
- 確定: 戦略実行に使用
- アーカイブ: 旧版保管

## Insight → BMC 接続フロー
1. Insight 確定（重要度4以上 + 確からしさ4以上）
2. 顧客属性 × ジャーニー段階 で集計
3. BMC.Customer Segments へ反映
4. Forces 集計から BMC.Value Propositions の根拠データ化

## Relation 仕様

### BMC ↔ VPC（双方向）
- 多重度: 1対多（1BMCに複数VPC）
- 必須: ❌

### BMC → TagDictionary
- 多重度: 多対多
- 必須: ❌

## BMC作成の単位
- 1事業 = 1 BMC が基本
- 複数サービスラインがある場合のみサービス単位に分割
- 事業のピボット時はバージョン番号を更新

## バージョン管理
- `バージョン` プロパティで世代管理
- 確定後の大幅修正は新バージョン作成（旧版は `アーカイブ`）

## 推奨ビュー

### Tier 1
- All BMC (Table)
- 確定済み (Table, フィルタ: ステータス=確定)

### Tier 2
- 事業ステージ別 (Board, グループ: 事業ステージ)
- バージョン履歴 (Table, グループ: 事業名, ソート: バージョン降順)

## 関連ドキュメント
- [SPECIFICATION.md](../SPECIFICATION.md)
- [vpc-db-schema.md](./vpc-db-schema.md)
- [docs/integration/bmc-db-connection.md](../docs/integration/bmc-db-connection.md)
