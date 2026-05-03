# Job DB Schema

> **File**: `schemas/job-db-schema.md`
> **Version**: 1.0
> **Last Updated**: 2026-05-03
> **DB名**: `Job` / **論理名**: `job`

## 概要
JTBD理論における Job（顧客が片付けたい用事）を管理する DB。Insight DB から根拠データを受け取り、VPC DB へ接続する中間層。

## 一意識別子
- Title: `job_summary`（人間可読な要約）
- Unique ID: `Job ID`（自動採番、プレフィックス: `JOB-`）

## プロパティ一覧

| # | プロパティ名 | 型 | 必須 | 説明 |
|---|---|---|---|---|
| 1 | job_summary | Title | ✅ | jobの簡潔な要約（25文字以内） |
| 2 | Job ID | Unique ID | ✅ | 自動採番（`JOB-`） |
| 3 | job_statement | Text | ✅ | "When [situation], I want to [motivation], so I can [outcome]" |
| 4 | situation | Text | ✅ | 状況（When 部分） |
| 5 | motivation | Text | ✅ | 動機（want to 部分） |
| 6 | outcome | Text | ✅ | 望む結果（so I can 部分） |
| 7 | job_type | Select | ❌ | Functional / Emotional / Social |
| 8 | 現状代替手段 | Text | ❌ | Habit Forces から導出 |
| 9 | 重要度 | Number | ✅ | 1-5 |
| 10 | 満足度 | Number | ❌ | 1-5（ODI参考値） |
| 11 | 機会スコア | Formula | ❌ | `重要度 + max(0, 重要度 - 満足度)` |
| 12 | 関連insight | Relation | ❌ | Insight DB（双方向） |
| 13 | 関連VPC | Relation | ❌ | VPC DB（双方向） |
| 14 | タグ | Relation | ❌ | TagDictionary |
| 15 | ステータス | Select | ✅ | 仮説 / 検証中 / 確定 / 棄却 |
| 16 | 作成日 | Date | ✅ | |
| 17 | 更新日 | Date | ❌ | |

## Select 候補値

### job_type
- Functional: 機能的なJob（タスク完遂）
- Emotional: 感情的なJob（気分・自己認識）
- Social: 社会的なJob（他者からの認識）

### ステータス
- 仮説: insight 不足、未検証
- 検証中: insight 収集中
- 確定: 戦略立案に使用可
- 棄却: 検証の結果、不採用

## 機会スコアの算出
ODI（Outcome-Driven Innovation）の機会アルゴリズムに準拠:
`opportunity_score = importance + max(0, importance - satisfaction)`

- 重要度高 + 満足度低 = 機会スコア最大（最優先）
- 重要度高 + 満足度高 = 機会スコア中
- 重要度低 = 機会スコア低（優先度下）

## Relation 仕様

### Job ↔ Insight（双方向）
- 多重度: 1対多（1つのjobに複数insight）
- 必須: ❌（Job作成時はinsight未接続でも可）

### Job ↔ VPC（双方向）
- 多重度: 1対多（1つのjobに複数VPC）
- 必須: ❌

### Job → TagDictionary
- 多重度: 多対多
- 必須: ❌

## 推奨ビュー

### Tier 1
- All Jobs (Table)
- 確定済み (Table, フィルタ: ステータス=確定)
- 機会スコア順 (Table, ソート: 機会スコア降順)

### Tier 2
- ステータス別 (Board, グループ: ステータス)
- job_type別 (Board, グループ: job_type)

## 初期エントリ
- `job_summary`: 未分類
- `ステータス`: 仮説
- 用途: 未分類insight の一時接続先

## 関連ドキュメント
- [SPECIFICATION.md](../SPECIFICATION.md)
- [property-definitions.md](./property-definitions.md)
- [relations.md](./relations.md)
- [docs/integration/job-db-connection.md](../docs/integration/job-db-connection.md)
