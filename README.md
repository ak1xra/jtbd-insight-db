# insight DB

JTBD（Jobs to be Done）と Forces of Progress に沿って顧客インサイトを Notion 上で管理し、VPC / BMC へ接続しやすい形にするための **設計ドキュメント一式** です。

- **仕様の正本**: [SPECIFICATION.md](./SPECIFICATION.md)（プロパティ定義・運用ルールの一次情報）
- **入力の手順**: [docs/operations/input-workflow.md](./docs/operations/input-workflow.md)（v2.0 手順。README のクイックスタートの次に読むとよい）
- **変更履歴**: [CHANGELOG.md](./CHANGELOG.md)

[![Version](https://img.shields.io/badge/version-2.0-blue.svg)](./CHANGELOG.md)
[![Notion](https://img.shields.io/badge/platform-Notion-black.svg)](https://www.notion.so/)
[![Framework](https://img.shields.io/badge/framework-JTBD-orange.svg)](./docs/architecture/jtbd-forces-framework.md)
[![AI](https://img.shields.io/badge/AI-Opus_4.6_%2F_Sonnet_4.6-purple.svg)](./docs/ai-prompts/README.md)

## 概要

`Insight`（論理名: insight DB）は、顧客の **生の声** を一次情報として保持し、Forces ラベルと構造化された **要約** で分析可能にします。AI は下書きを提案し、**確定の判断は人間** が行います（Human-in-the-Point）。

### この設計が向いている状況

- インサイトが散在し、戦略の議論で再利用しづらい
- 細かい自由記述だけだと入力負荷とハルシネーションリスクが高い
- VPC / BMC への写像を、ラベルと要約から再現したい

### 提供する価値

- **Single Source of Truth**: 一次情報は `生の声` のみ（改変しない）
- **AI は補助**: autofill のあと必ず人手レビュー
- **下流との接続**: job / VPC / BMC / TagDictionary との Relation 方針を文書化

## クイックスタート

### 前提

- Notion ワークスペース（Plus 以上を推奨）
- Notion AI（Opus 4.6 が autofill に使えるプラン）
- 接続先 DB: `job` と `TagDictionary` は必須、`VPC` と `BMC` は推奨

### 最短の流れ

1. リポジトリを取得する。
2. [SPECIFICATION.md](./SPECIFICATION.md) に従い、Notion 上に DB とプロパティを作成する。
3. [docs/ai-prompts/](./docs/ai-prompts/) の各プロンプトを、該当プロパティの AI autofill に設定する。
4. [docs/operations/input-workflow.md](./docs/operations/input-workflow.md) に従い、最初の 1 件を通して運用を確認する。

## アーキテクチャ（要約）

データの流れの詳細は [docs/architecture/overview.md](./docs/architecture/overview.md) を参照してください。

```
┌─────────────────────────────────────────────────────┐
│                   insight DB                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────┐ │
│  │   生の声     │→ │  AI autofill │→ │  要約    │ │
│  │ (一次情報)   │  │  (Opus 4.6)  │  │ (構造化) │ │
│  └──────────────┘  └──────────────┘  └──────────┘ │
│         │                  │                │       │
│         ▼                  ▼                ▼       │
│  ┌──────────────────────────────────────────────┐  │
│  │  Forces / ジャーニー段階 / タグ              │  │
│  └──────────────────────────────────────────────┘  │
└────────────────┬────────────────────────────────────┘
                 │ Relation
                 ▼
        ┌────────────────┐
        │   job DB       │ ──> VPC ──> BMC
        └────────────────┘
```

## プロパティ一覧（v2.0 完全版）

次の表は [SPECIFICATION.md §3.1](./SPECIFICATION.md#31-完全なプロパティ一覧) と同一の定義です。列「必須」「AI」は運用上の意味（Notion の UI 上の必須とは一致しない場合があります）。

| # | プロパティ名 | 型 | 必須 | AI autofill | 説明 |
|---|-------------|-----|:----:|:-------------:|------|
| 1 | `insight名` | Title | はい | はい | 見出し（25 文字以内） |
| 2 | `生の声` | Text | はい | いいえ | 発言・観察・メモの原文（一次情報） |
| 3 | `情報源タイプ` | Select | はい | いいえ | 情報の取得元 |
| 4 | `取得日` | Date | はい | いいえ | 情報取得日 |
| 5 | `顧客属性` | Select | はい | いいえ | 購買・利用プロセス上の役割 |
| 6 | `役職・立場` | Text | いいえ | いいえ | 組織上の肩書き |
| 7 | `所属部門` | Text | いいえ | いいえ | 例: マーケ、営業、人事 |
| 8 | `Forces` | Multi-select | 条件付き※ | はい | JTBD Forces of Progress |
| 9 | `ジャーニー段階` | Select | はい | はい | 購買・意思決定プロセスの段階 |
| 10 | `重要度` | Number | はい | いいえ | 1〜5（人手のみ） |
| 11 | `確からしさ` | Number | はい | いいえ | 1〜5（人手のみ） |
| 12 | `要約` | Text | はい | はい | Forces 構造化要約 |
| 13 | `job` | Relation | 段階的※※ | いいえ | `job` DB への接続 |
| 14 | `タグ` | Relation | いいえ | いいえ | `TagDictionary` DB への接続 |
| 15 | `レビュー状態` | Select | はい | いいえ | `下書き` / `レビュー中` / `確定` |
| 16 | `レビュー者` | Person | いいえ | いいえ | レビュー担当（確定時に必須） |
| 17 | `レビュー日` | Date | いいえ | いいえ | レビュー完了日（確定時に必須） |

- **※ `Forces` の「条件付き必須」**: 確定までに「最低 1 つ選択」**または**「空欄の理由を `要約` の【主訴】に明記」のどちらかが必要です。詳細は [schemas/property-definitions.md §8](./schemas/property-definitions.md#8-forces) を参照してください。
- **※※ `job` の「段階的」**: 作成直後は未接続でもよい一方、`レビュー状態 = 下書き` に進む前に接続が必要です。詳細は [schemas/relations.md §1](./schemas/relations.md#1-insightjob--job-db) を参照してください。

## AI autofill（Notion）

| 対象プロパティ | モデル（目安） | プロンプト |
|---|---|---|
| `insight名` | Opus 4.6 | [insight-name.md](./docs/ai-prompts/insight-name.md) |
| `Forces` | Opus 4.6 | [forces.md](./docs/ai-prompts/forces.md) |
| `ジャーニー段階` | Opus 4.6 | [journey-stage.md](./docs/ai-prompts/journey-stage.md) |
| `要約` | Opus 4.6 | [summary.md](./docs/ai-prompts/summary.md) |

運用の原則（短縮版）:

- AI の出力は **下書き** として扱う
- **確定** は人が行う（`レビュー状態` を参照）
- レビュー前の行を戦略集計の母集団に混ぜない

詳細は [docs/governance/human-in-the-point.md](./docs/governance/human-in-the-point.md) を参照してください。

## ディレクトリ構造

```
jtbd-insight-db/
├── SPECIFICATION.md          # 仕様の正本
├── CHANGELOG.md
├── README.md
├── LICENSE
├── schemas/
│   ├── insight-db-schema.md  # Insight DB スキーマ
│   ├── job-db-schema.md      # Job DB スキーマ
│   ├── vpc-db-schema.md      # VPC DB スキーマ
│   ├── bmc-db-schema.md      # BMC DB スキーマ
│   ├── property-definitions.md
│   ├── select-options.md
│   └── relations.md          # 全DB間の Relation 設計
├── docs/
│   ├── ai-prompts/           # AI autofill プロンプト
│   ├── architecture/         # アーキテクチャ設計
│   ├── governance/           # ガバナンス・運用ポリシー
│   ├── integration/          # 他DB接続ドキュメント
│   └── operations/           # 運用手順
└── .specstory/
```

## 連携（他 DB）

| DB名 | 論理名 | スキーマ | 接続詳細 |
|---|---|---|---|
| Job | `job` | [schemas/job-db-schema.md](./schemas/job-db-schema.md) | [接続仕様](./docs/integration/job-db-connection.md) |
| VPC | `vpc` | [schemas/vpc-db-schema.md](./schemas/vpc-db-schema.md) | [接続仕様](./docs/integration/vpc-db-connection.md) |
| BMC | `bmc` | [schemas/bmc-db-schema.md](./schemas/bmc-db-schema.md) | [接続仕様](./docs/integration/bmc-db-connection.md) |
| TagDictionary | `tag_dictionary` | — | [接続仕様](./docs/integration/tag-dictionary-connection.md) |

## 運用ドキュメント

- [入力フロー](./docs/operations/input-workflow.md)
- [レビュー手順](./docs/operations/review-workflow.md)
- [v1.3 から v2.0 への移行](./docs/operations/migration-v1-to-v2.md)
- [定期メンテナンス](./docs/operations/maintenance.md)

## 禁止事項（短縮版）

完全版は [docs/governance/forbidden-actions.md](./docs/governance/forbidden-actions.md) を参照してください。

- AI autofill の結果を、人のレビューなしで **`レビュー状態 = 確定`** にしない
- `生の声` にない事実を AI に推測させない
- `重要度` と `確からしさ` を AI に任せない
- 移行作業中に本番運用を続けない

## 設計原則

1. **Human-in-the-Point**: AI は提案、確定の判断は人間
2. **Single Source of Truth**: 一次情報は `生の声` のみ
3. **構造化優先**: ラベルとテンプレに寄せる
4. **JTBD 整合**: Forces of Progress に合わせる
5. **VPC / BMC 接続**: 下流の戦略設計に渡せる情報設計にする

## バージョン履歴

| Version | Date | Summary |
|---|---|---|
| 2.0 | 2026-05-03 | Forces ラベル化、テキストフィールド統合 |
| 1.3 | 2026-05-03 | 顧客属性の仮分類追加 |
| 1.0 | （未記録） | 初版 |

詳細な差分は [CHANGELOG.md](./CHANGELOG.md) を参照してください。

## メンテナー

**AK1RA (Hayakawa Akira)** — AI/DX Consultant, Tokyo

## ライセンス

MIT License。全文は [LICENSE](./LICENSE) を参照してください。

## 関連ドキュメント

- [JTBD Forces of Progress フレーム](./docs/architecture/jtbd-forces-framework.md)
- [Notion AI Autofill プロンプト集](./docs/ai-prompts/README.md)
- [品質基準](./docs/governance/quality-criteria.md)
