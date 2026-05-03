# Property Definitions

> **File**: `schemas/property-definitions.md`
> **Version**: 2.0
> **Last Updated**: 2026-05-03
> **Related**: [SPECIFICATION.md](../SPECIFICATION.md) §3 / [select-options.md](./select-options.md) / [relations.md](./relations.md)

---

## 概要

本リポジトリで管理する全 DB のプロパティ詳細定義。型・必須性・バリデーション・AI autofill 仕様・運用ルールを記載する。

| DB | 論理名 | プロパティ数 | スキーマ |
|---|---|---|---|
| Insight | `insight` | 17 | [insight-db-schema.md](./insight-db-schema.md) |
| Job | `job` | 17 | [job-db-schema.md](./job-db-schema.md) |
| VPC | `vpc` | 18 | [vpc-db-schema.md](./vpc-db-schema.md) |
| BMC | `bmc` | 18 | [bmc-db-schema.md](./bmc-db-schema.md) |

---

---

# Part 1: Insight DB プロパティ詳細

## プロパティ一覧

| # | プロパティ名 | 型 | 必須 | AI |
|---|---|---|---|---|
| 1 | [insight名](#1-insight名) | Title | ✅ | ✅ |
| 2 | [生の声](#2-生の声) | Text | ✅ | ❌ |
| 3 | [情報源タイプ](#3-情報源タイプ) | Select | ✅ | ❌ |
| 4 | [取得日](#4-取得日) | Date | ✅ | ❌ |
| 5 | [顧客属性](#5-顧客属性) | Select | ✅ | ❌ |
| 6 | [役職・立場](#6-役職立場) | Text | ❌ | ❌ |
| 7 | [所属部門](#7-所属部門) | Text | ❌ | ❌ |
| 8 | [Forces](#8-forces) | Multi-select | ✅（条件付き） | ✅ |
| 9 | [ジャーニー段階](#9-ジャーニー段階) | Select | ✅ | ✅ |
| 10 | [重要度](#10-重要度) | Number | ✅ | ❌ |
| 11 | [確からしさ](#11-確からしさ) | Number | ✅ | ❌ |
| 12 | [要約](#12-要約) | Text | ✅ | ✅ |
| 13 | [job](#13-job) | Relation | ✅（段階的） | ❌ |
| 14 | [タグ](#14-タグ) | Relation | ❌ | ❌ |
| 15 | [レビュー状態](#15-レビュー状態) | Select | ✅ | ❌ |
| 16 | [レビュー者](#16-レビュー者) | Person | ❌ | ❌ |
| 17 | [レビュー日](#17-レビュー日) | Date | ❌ | ❌ |

---

## 1. insight名

| 項目 | 値 |
|---|---|
| **型** | Title |
| **必須** | ✅ |
| **AI autofill** | ✅（Opus 4.6） |
| **デフォルト値** | なし |

### 用途
insightの見出しとして、一覧ビューでの識別に使用。

### バリデーション
- 最大25文字
- 体言止めを推奨
- 顧客の主訴または核心を一文で表現

### AI autofill 仕様
- 入力: `生の声` `顧客属性`
- プロンプト: [docs/ai-prompts/insight-name.md](../docs/ai-prompts/insight-name.md)
- 出力: 見出しテキストのみ（説明・装飾語不要）

### 運用ルール
- AI autofill 後、必ず人手レビュー
- 25文字を超える場合は短縮
- 同一人物・同一案件の場合、識別可能な要素を含める

### NG例
- ❌ 「顧客の声について」（抽象的）
- ❌ 「とても重要な顧客からの貴重なフィードバック」（装飾語過剰）

### OK例
- ✅ 「Excel管理の限界とクラウド化検討」
- ✅ 「価格よりサポート体制を重視」

---

## 2. 生の声

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |
| **AI autofill** | ❌ |
| **デフォルト値** | なし |

### 用途
顧客の発言・観察・メモの**原文**を保持する一次情報。本DBにおける唯一のSSOT（Single Source of Truth）。

### バリデーション
- 最低10文字
- 上限なし（実用上は500文字程度を推奨）

### 入力ルール
- **編集禁止**: 一次情報は改変しない
- 補足が必要な場合はNotionコメント機能を使用
- 録音からの書き起こしの場合、表記揺れを保持
- 個人特定情報はマスキング（A社、B氏）

### NG例
- ❌ 解釈を加えた要約（例: 「顧客は満足している」）
- ❌ 複数発言の合成（例: 「Aさんも言っていたが...」）
- ❌ 推測の混入（例: 「おそらく...と思っているはず」）

### OK例
- ✅ 「今のExcel管理が限界。クラウド化したいが、ITに弱い高齢スタッフがついてこれるか心配」
- ✅ 「（営業同行時の発言）うちは月3万までなら出せる、それ以上は無理」

### 注意事項
- 本プロパティの内容が後段の全AI autofillの入力となるため、品質が DB 全体の精度を決定する
- 改変・解釈の混入はハルシネーションの温床となる

---

## 3. 情報源タイプ

| 項目 | 値 |
|---|---|
| **型** | Select |
| **必須** | ✅ |
| **AI autofill** | ❌ |
| **デフォルト値** | なし |

### 用途
情報の取得元を分類し、信頼性評価および集計軸として使用。

### 候補値
詳細は [select-options.md](./select-options.md#情報源タイプ) を参照。

| 値 | 主な用途 |
|---|---|
| `interview` | 構造化インタビュー |
| `survey` | アンケート |
| `sales_call` | 営業電話・商談 |
| `support` | サポート問い合わせ |
| `sns` | SNS投稿 |
| `review` | レビュー記事・口コミ |
| `analytics` | 行動データ |

### 運用ルール
- 必ず1つ選択
- 複数情報源の合成は禁止（別エントリとして登録）

---

## 4. 取得日

| 項目 | 値 |
|---|---|
| **型** | Date |
| **必須** | ✅ |
| **AI autofill** | ❌ |
| **デフォルト値** | 入力日（手動設定） |

### 用途
情報の鮮度評価および時系列分析の軸。

### 入力ルール
- 情報を**取得した日**を記入（DB登録日ではない）
- インタビューの場合は実施日
- SNS投稿の場合は投稿日

### バリデーション
- 未来日付は不可
- 5年以上前の情報は `確からしさ` を低く評価

---

## 5. 顧客属性

| 項目 | 値 |
|---|---|
| **型** | Select |
| **必須** | ✅ |
| **AI autofill** | ❌ |
| **デフォルト値** | なし |

### 用途
購買・利用プロセス上の役割を分類。**役職ではなく役割**で切る。

### 候補値
詳細は [select-options.md](./select-options.md#顧客属性) を参照。

### 設計思想
- JTBD分析では肩書きより「どの立場でそのjobを抱えているか」が重要
- 役職を管理する場合は `役職・立場` `所属部門` プロパティを併用

### 判定が難しい場合
- `不明` を選択して問題なし
- 後で情報が増えたら更新

---

## 6. 役職・立場

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ❌ |
| **AI autofill** | ❌ |
| **デフォルト値** | なし |

### 用途
組織上の肩書きを記録。`顧客属性` とは別軸として運用。

### 入力例
- 代表取締役
- 営業部長
- 経理担当
- フリーランス

### 運用ルール
- 表記揺れを許容（厳密な統一は不要）
- 後で必要なら Select 化を検討

---

## 7. 所属部門

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ❌ |
| **AI autofill** | ❌ |
| **デフォルト値** | なし |

### 用途
所属部門を記録。組織横断の集計に使用。

### 入力例
- マーケティング
- 営業
- 人事
- 経理
- 情報システム

### 運用ルール
- 略称・正式名称どちらでも可
- 中小企業の場合「部門なし」も許容

---

## 8. Forces

| 項目 | 値 |
|---|---|
| **型** | Multi-select |
| **必須** | ✅（条件付き） |
| **AI autofill** | ✅（Opus 4.6） |
| **デフォルト値** | なし |

### 用途
JTBD理論の **Forces of Progress**（4つの力）を分類。本DBの戦略立案接続点の中核。

### 必須性の定義（重要）

Notion の Multi-select は UI 上「必須」にしにくいため、本プロパティの **必須** は運用ルールとして次を満たすことと定義する（Human-in-the-Point）。

**`レビュー状態 = 確定` までに、次のいずれかを満たすこと:**

- **(A)** 最低 1 つの Forces を選択する
- **(B)** 選択なし（空欄）の場合、`要約` の【主訴】に **空欄理由** を明記する

### 必須性の時間軸

| タイミング | 状態 | 許容範囲 |
|---|---|---|
| 作成時（Step 2） | 空欄 | 許容 |
| AI autofill 直後（Step 5） | 空欄も可 | 生の声に Forces が明示されていない場合、空欄が正しい |
| レビュー時 | 妥当性を人が判定 | 推測で埋めない |
| `レビュー状態 = 確定` 時 | (A) または (B) | **必須** |

### 空欄が妥当になりうる例

- 「弊社の従業員数は150名です」（事実報告のみ）
- 「年商は3億円規模」（属性情報のみ）

### 候補値
詳細は [select-options.md](./select-options.md#forces) を参照。

| 値 | 定義 |
|---|---|
| `Push` | 現状から押し出す力 |
| `Pull` | 新しい解決策へ引き寄せる力 |
| `Anxiety` | 変化への不安・懸念 |
| `Habit` | 現状維持を促す力 |

### AI autofill 仕様
- 入力: `生の声` `情報源タイプ` `顧客属性`
- プロンプト: [docs/ai-prompts/forces.md](../docs/ai-prompts/forces.md)
- 出力: カンマ区切りの英語ラベル

### 判定ルール
- 「生の声」に明示的に該当する内容がある場合のみ選択
- **推測で選ばない**
- 該当なしの場合は空欄可（確定前に、(A) または (B) を満たす）
- 複数該当する場合は複数選択

### VPC連携
- `Push` `Anxiety` → VPC.Pains
- `Pull` → VPC.Gains
- `Habit` → job.現状代替手段

---

## 9. ジャーニー段階

| 項目 | 値 |
|---|---|
| **型** | Select |
| **必須** | ✅ |
| **AI autofill** | ✅（Opus 4.6） |
| **デフォルト値** | `不明` |

### 用途
顧客の購買・意思決定プロセス上の段階を分類。BMC.Customer Segments への接続軸。

### 候補値
詳細は [select-options.md](./select-options.md#ジャーニー段階) を参照。

### AI autofill 仕様
- 入力: `生の声` `情報源タイプ` `顧客属性`
- プロンプト: [docs/ai-prompts/journey-stage.md](../docs/ai-prompts/journey-stage.md)
- 出力: 候補のうち1語のみ

### 判定優先順位
1. 生の声に明示された段階情報を優先
2. 明示されない場合のみ Forces やジャーニー文脈から推定
3. 推定根拠が薄い場合は `不明`

---

## 10. 重要度

| 項目 | 値 |
|---|---|
| **型** | Number |
| **必須** | ✅ |
| **AI autofill** | ❌（人手必須） |
| **範囲** | 1〜5（整数） |
| **デフォルト値** | なし |

### 用途
insight の戦略的重要度を評価。`確からしさ` と組み合わせて戦略立案使用条件を判定。

### 評価基準

| 値 | 定義 |
|---|---|
| 5 | 戦略の根幹に関わる |
| 4 | 重要な意思決定材料 |
| 3 | 参考情報として有用 |
| 2 | 補助的情報 |
| 1 | ノイズに近い |

### AI autofill 禁止理由
- 重要度判断は戦略コンテキスト依存
- AIに判定させると客観的根拠なく中央値（3）に寄る傾向
- Human-in-the-Point 設計原則に基づき必ず人手評価

### 戦略立案使用条件
- `重要度 ≥ 4` かつ `確からしさ ≥ 4` の組み合わせで採用

---

## 11. 確からしさ

| 項目 | 値 |
|---|---|
| **型** | Number |
| **必須** | ✅ |
| **AI autofill** | ❌（人手必須） |
| **範囲** | 1〜5(整数) |
| **デフォルト値** | なし |

### 用途
情報の信頼度を評価。

### 評価基準

| 値 | 定義 |
|---|---|
| 5 | 一次情報(録音・記録あり) |
| 4 | 信頼できる情報源からの伝聞 |
| 3 | 複数の傍証あり |
| 2 | 単一情報源・確認困難 |
| 1 | 推測・解釈に依存 |

### 評価ガイドライン
- インタビュー録音 → 5
- 営業同行メモ → 4
- 営業から聞いた話 → 3
- SNSの匿名投稿 → 2
- 業界の噂 → 1

---

## 12. 要約

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |
| **AI autofill** | ✅（Opus 4.6） |
| **デフォルト値** | なし |

### 用途
Forces of Progress フレームに沿った構造化要約。VPC/BMC への変換時の主たる入力データ。

### 出力フォーマット（厳守）

```
【主訴】（1文・25文字以内）

【Push】生の声から読み取れる不満・問題（明示なしの場合は「該当なし」）
【Pull】生の声から読み取れる望む結果・期待（明示なしの場合は「該当なし」）
【Anxiety】生の声から読み取れる不安・懸念（明示なしの場合は「該当なし」）
【Habit】生の声から読み取れる現状維持要因（明示なしの場合は「該当なし」）

【選定基準・見送り理由】生の声に明示されている場合のみ記載（なければ「該当なし」）
```

### AI autofill 仕様
- 入力: `生の声` `顧客属性` `ジャーニー段階`
- プロンプト: [docs/ai-prompts/summary.md](../docs/ai-prompts/summary.md)

### 判定ルール
- 「生の声」に明示されていない情報は記載しない
- 推測・補完・拡張は禁止
- 各項目は1〜2文以内
- 「該当なし」を恐れず使う

---

## 13. job

| 項目 | 値 |
|---|---|
| **型** | Relation |
| **必須** | ✅（`レビュー状態 = 下書き` へ至るまで） |
| **AI autofill** | ❌ |
| **接続先** | `job` DB |

### 必須性の時間軸

- 作成時（Step 2）: 任意（未接続でもよい）
- AI autofill 完了時（Step 5）: 任意（AI 出力を材料に job を選ぶことを推奨）
- **`レビュー状態 = 下書き` にする前（Step 7〜8）: 必須**
- `レビュー状態 = 確定` 時: 必須（下書き時点で満たしている前提）

詳細: [relations.md](./relations.md#1-insightjob--job-db)

### 用途
insightが紐づく顧客のJobを示す。本DBにおいて最重要のRelation。

### 多重度
多対1（1つのinsightは1つのjobに紐づく）

### 双方向Relation
- job 側に `関連insight` プロパティを設置
- 1つのjob は複数のinsightを持つ

### 詳細
[relations.md](./relations.md#1-insightjob--job-db) 参照

---

## 14. タグ

| 項目 | 値 |
|---|---|
| **型** | Relation |
| **必須** | ❌ |
| **AI autofill** | ❌ |
| **接続先** | `TagDictionary` DB |

### 用途
横断的な分類軸。テーマ・論点・選定基準・見送り理由・業界などを管理。

### 多重度
多対多

### 主な使用例
- 業界タグ（建設、医療、IT）
- テーマタグ（価格、機能、サポート）
- 選定基準（v1.3 から移管）
- 見送り理由（v1.3 から移管）

### 詳細
[relations.md](./relations.md#2-insightタグ--tagdictionary) 参照

---

## 15. レビュー状態

| 項目 | 値 |
|---|---|
| **型** | Select |
| **必須** | ✅ |
| **AI autofill** | ❌ |
| **デフォルト値** | `下書き` |

### 用途
Human-in-the-Point 原則の実装。AI autofill出力の人手レビュー進捗を管理。

### 候補値

| 値 | 定義 |
|---|---|
| `下書き` | AI autofill完了、レビュー未着手 |
| `レビュー中` | レビュー実施中 |
| `確定` | レビュー完了、戦略立案に使用可 |

### 状態遷移

```
[新規作成] → 下書き → レビュー中 → 確定
                ↑          │
                └──────────┘
              （差し戻し可）
```

### 戦略立案使用条件
- `レビュー状態 = 確定` のもののみ使用可
- `下書き` `レビュー中` は集計・戦略立案から除外

---

## 16. レビュー者

| 項目 | 値 |
|---|---|
| **型** | Person |
| **必須** | ❌（`確定` 時は必須） |
| **AI autofill** | ❌ |

### 用途
レビュー責任者を記録。

### 運用ルール
- `レビュー状態 = 確定` 時に必ず記入
- 通常はAK1RA本人
- クライアント案件の場合、クライアント側担当者を記入することも可

---

## 17. レビュー日

| 項目 | 値 |
|---|---|
| **型** | Date |
| **必須** | ❌（`確定` 時は必須） |
| **AI autofill** | ❌ |

### 用途
レビュー完了日を記録。データ鮮度の指標。

### 運用ルール
- `レビュー状態 = 確定` 時に必ず記入
- 入力からレビュー完了までの日数（`レビュー日 - 取得日`）が品質メトリクス

---

## 全体的な運用ルール

### 入力順序（`SPECIFICATION.md` §7.1 準拠）

Human-in-the-Point の観点では、**AI 提案（Step 5）の前に人が付けるべきもの**と、**AI のあとに人が付けるべきもの**を分離する。

1. `生の声` を貼り付ける
2. `情報源タイプ` `取得日` `顧客属性` を設定する
3. AI autofill を実行する: `insight名` → `Forces` → `ジャーニー段階` → `要約`
4. `重要度` `確からしさ` を人手評価する（AI に任せない）
5. `job` `タグ` を Relation 接続する（`job` はこの時点で必須。未分類なら `unclassified_job` 可）
6. `レビュー状態 = 下書き` にする（初期入力の完了地点）

### レビュー時の確認順序
1. `生の声` に改変・解釈の混入がないか
2. AI autofill 4項目のハルシネーション有無
3. `Forces` の判定が `生の声` の内容と整合
4. `要約` の各項目が「該当なし」の使い方が適切
5. `job` `タグ` の接続が論理的

---

---
---

# Part 2: Job DB プロパティ詳細

> スキーマ全量: [job-db-schema.md](./job-db-schema.md)

## プロパティ一覧（Job）

| # | プロパティ名 | 型 | 必須 |
|---|---|---|---|
| 1 | [job_summary](#j1-job_summary) | Title | ✅ |
| 2 | [Job ID](#j2-job-id) | Unique ID | ✅ |
| 3 | [job_statement](#j3-job_statement) | Text | ✅ |
| 4 | [situation](#j4-situation) | Text | ✅ |
| 5 | [motivation](#j5-motivation) | Text | ✅ |
| 6 | [outcome](#j6-outcome) | Text | ✅ |
| 7 | [job_type](#j7-job_type) | Select | ❌ |
| 8 | [現状代替手段](#j8-現状代替手段) | Text | ❌ |
| 9 | [重要度](#j9-重要度) | Number | ✅ |
| 10 | [満足度](#j10-満足度) | Number | ❌ |
| 11 | [機会スコア](#j11-機会スコア) | Formula | ❌ |
| 12 | [関連insight](#j12-関連insight) | Relation | ❌ |
| 13 | [関連VPC](#j13-関連vpc) | Relation | ❌ |
| 14 | [タグ](#j14-タグ) | Relation | ❌ |
| 15 | [ステータス](#j15-ステータス) | Select | ✅ |
| 16 | [作成日](#j16-作成日) | Date | ✅ |
| 17 | [更新日](#j17-更新日) | Date | ❌ |

---

## J1. job_summary

| 項目 | 値 |
|---|---|
| **型** | Title |
| **必須** | ✅ |
| **AI autofill** | ❌ |

### 用途
Job の簡潔な要約。一覧ビューでの識別に使用。

### バリデーション
- 最大 25 文字
- 顧客の Job を端的に表現

### OK例
- ✅ 「月次レポート作成の自動化」
- ✅ 「社内承認フローの短縮」

### NG例
- ❌ 「顧客が困っている問題について」（抽象的）
- ❌ 「レポート」（粒度が粗すぎる）

---

## J2. Job ID

| 項目 | 値 |
|---|---|
| **型** | Unique ID |
| **必須** | ✅（自動） |
| **プレフィックス** | `JOB-` |

自動採番。手動入力禁止。

---

## J3. job_statement

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

### フォーマット（厳守）
```
When [situation], I want to [motivation], so I can [outcome]
```

### バリデーション
- 3 パート（situation / motivation / outcome）がすべて含まれていること
- `situation`, `motivation`, `outcome` プロパティの内容と一致すること

### OK例
- ✅ "When preparing monthly reports manually, I want to automate data aggregation, so I can focus on analysis"

### NG例
- ❌ "I want better reports"（situation / outcome が欠落）

---

## J4. situation

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

job_statement の "When [situation]" 部分を独立プロパティとして保持。集計・検索用。

---

## J5. motivation

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

job_statement の "I want to [motivation]" 部分。

---

## J6. outcome

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

job_statement の "so I can [outcome]" 部分。

---

## J7. job_type

| 項目 | 値 |
|---|---|
| **型** | Select |
| **必須** | ❌ |

### 候補値
詳細は [select-options.md](./select-options.md#job_type) を参照。

| 値 | 定義 |
|---|---|
| `Functional` | 機能的な Job（タスク完遂） |
| `Emotional` | 感情的な Job（気分・自己認識） |
| `Social` | 社会的な Job（他者からの認識） |

### 判定ルール
- 1 つの Job は 1 つの type を持つ
- 複合的な場合は主要な側面を選択
- 判定が難しい場合は空欄可

---

## J8. 現状代替手段

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ❌ |

### 用途
顧客が Job を片付けるために現在使っている手段。Insight の `Habit` Forces から導出される。

### 入力例
- 「Excel で手動集計」
- 「紙の申請書 + 社内便」

---

## J9. 重要度

| 項目 | 値 |
|---|---|
| **型** | Number |
| **必須** | ✅ |
| **範囲** | 1〜5（整数） |

ODI 機会スコアの入力値。顧客にとっての Job の重要度を評価。

---

## J10. 満足度

| 項目 | 値 |
|---|---|
| **型** | Number |
| **必須** | ❌ |
| **範囲** | 1〜5（整数） |

ODI 機会スコアの入力値。現状の解決策に対する満足度。

---

## J11. 機会スコア

| 項目 | 値 |
|---|---|
| **型** | Formula |
| **必須** | ❌（自動算出） |

### 算出式
```
opportunity_score = 重要度 + max(0, 重要度 - 満足度)
```

ODI（Outcome-Driven Innovation）に準拠。スコアが高いほど市場機会が大きい。

---

## J12. 関連insight

| 項目 | 値 |
|---|---|
| **型** | Relation |
| **接続先** | Insight DB |
| **多重度** | 1対多 |
| **双方向** | ✅ |

詳細: [relations.md §1](./relations.md#1-insightjob--job-db)

---

## J13. 関連VPC

| 項目 | 値 |
|---|---|
| **型** | Relation |
| **接続先** | VPC DB |
| **多重度** | 1対多 |
| **双方向** | ✅ |

詳細: [relations.md §3](./relations.md#3-job関連vpc--vpc-db)

---

## J14. タグ

| 項目 | 値 |
|---|---|
| **型** | Relation |
| **接続先** | TagDictionary |
| **多重度** | 多対多 |

---

## J15. ステータス

| 項目 | 値 |
|---|---|
| **型** | Select |
| **必須** | ✅ |
| **デフォルト値** | `仮説` |

### 候補値
詳細は [select-options.md](./select-options.md#job-ステータス) を参照。

| 値 | 定義 |
|---|---|
| `仮説` | insight 不足、未検証 |
| `検証中` | insight 収集中 |
| `確定` | 戦略立案に使用可 |
| `棄却` | 検証の結果、不採用 |

### 状態遷移
```
[新規作成] → 仮説 → 検証中 → 確定
                               → 棄却
```

---

## J16. 作成日

| 項目 | 値 |
|---|---|
| **型** | Date |
| **必須** | ✅ |

---

## J17. 更新日

| 項目 | 値 |
|---|---|
| **型** | Date |
| **必須** | ❌ |

---
---

# Part 3: VPC DB プロパティ詳細

> スキーマ全量: [vpc-db-schema.md](./vpc-db-schema.md)

## プロパティ一覧（VPC）

| # | プロパティ名 | 型 | 必須 |
|---|---|---|---|
| 1 | [VPC名](#v1-vpc名) | Title | ✅ |
| 2 | [VPC ID](#v2-vpc-id) | Unique ID | ✅ |
| 3 | [対象job](#v3-対象job) | Relation | ✅ |
| 4 | [顧客セグメント](#v4-顧客セグメント) | Text | ✅ |
| 5 | [Customer Jobs](#v5-customer-jobs) | Text | ✅ |
| 6 | [Pains](#v6-pains) | Text | ✅ |
| 7 | [Gains](#v7-gains) | Text | ✅ |
| 8 | [Products & Services](#v8-products--services) | Text | ✅ |
| 9 | [Pain Relievers](#v9-pain-relievers) | Text | ✅ |
| 10 | [Gain Creators](#v10-gain-creators) | Text | ✅ |
| 11 | [Pain重要度](#v11-pain重要度) | Select | ❌ |
| 12 | [Gain重要度](#v12-gain重要度) | Select | ❌ |
| 13 | [Fit評価](#v13-fit評価) | Select | ❌ |
| 14 | [関連BMC](#v14-関連bmc) | Relation | ❌ |
| 15 | [タグ](#v15-タグ) | Relation | ❌ |
| 16 | [ステータス](#v16-ステータス) | Select | ✅ |
| 17 | [作成日](#v17-作成日) | Date | ✅ |
| 18 | [更新日](#v18-更新日) | Date | ❌ |

---

## V1. VPC名

| 項目 | 値 |
|---|---|
| **型** | Title |
| **必須** | ✅ |

### 命名規則
`[セグメント名]向け [job要約]` 形式。

### OK例
- ✅ 「中小企業経理向け 月次レポート自動化」
- ✅ 「SaaS導入推進者向け 承認フロー短縮」

---

## V2. VPC ID

| 項目 | 値 |
|---|---|
| **型** | Unique ID |
| **必須** | ✅（自動） |
| **プレフィックス** | `VPC-` |

---

## V3. 対象job

| 項目 | 値 |
|---|---|
| **型** | Relation |
| **接続先** | Job DB |
| **必須** | ✅ |
| **多重度** | 多対1 |
| **双方向** | ✅ |

VPC 作成時に必ず 1 つの Job を接続する。

---

## V4. 顧客セグメント

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

対象とする顧客像を記述。VPC 名の `[セグメント名]` 部分の詳細版。

---

## V5. Customer Jobs

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

接続先 Job の要約。`対象job` Relation で接続した Job の `job_statement` を要約・転記。

---

## V6. Pains

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

顧客の痛み。Insight の `Push` / `Anxiety` Forces から集約。

---

## V7. Gains

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

顧客の利得。Insight の `Pull` Forces から集約。

---

## V8. Products & Services

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

Pains/Gains に対して提供する製品・サービスの記述。

---

## V9. Pain Relievers

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

Pains を緩和する手段。Products & Services と対応づける。

---

## V10. Gain Creators

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

Gains を生み出す手段。Products & Services と対応づける。

---

## V11. Pain重要度

| 項目 | 値 |
|---|---|
| **型** | Select |
| **必須** | ❌ |

候補値: `High` / `Medium` / `Low`。詳細は [select-options.md](./select-options.md#vpc-pain重要度--gain重要度)。

---

## V12. Gain重要度

| 項目 | 値 |
|---|---|
| **型** | Select |
| **必須** | ❌ |

候補値: `High` / `Medium` / `Low`。詳細は [select-options.md](./select-options.md#vpc-pain重要度--gain重要度)。

---

## V13. Fit評価

| 項目 | 値 |
|---|---|
| **型** | Select |
| **必須** | ❌ |

候補値: `Problem-Solution Fit` / `Product-Market Fit` / `検証中`。詳細は [select-options.md](./select-options.md#vpc-fit評価)。

---

## V14. 関連BMC

| 項目 | 値 |
|---|---|
| **型** | Relation |
| **接続先** | BMC DB |
| **必須** | ❌ |
| **多重度** | 多対1 |
| **双方向** | ✅ |

詳細: [relations.md §4](./relations.md#4-vpc関連bmc--bmc-db)

---

## V15. タグ

| 項目 | 値 |
|---|---|
| **型** | Relation |
| **接続先** | TagDictionary |
| **多重度** | 多対多 |

---

## V16. ステータス

| 項目 | 値 |
|---|---|
| **型** | Select |
| **必須** | ✅ |
| **デフォルト値** | `ドラフト` |

候補値: `ドラフト` / `レビュー中` / `確定`。詳細は [select-options.md](./select-options.md#vpc-ステータス)。

---

## V17. 作成日

| 項目 | 値 |
|---|---|
| **型** | Date |
| **必須** | ✅ |

---

## V18. 更新日

| 項目 | 値 |
|---|---|
| **型** | Date |
| **必須** | ❌ |

---
---

# Part 4: BMC DB プロパティ詳細

> スキーマ全量: [bmc-db-schema.md](./bmc-db-schema.md)

## プロパティ一覧（BMC）

| # | プロパティ名 | 型 | 必須 |
|---|---|---|---|
| 1 | [事業名](#b1-事業名) | Title | ✅ |
| 2 | [BMC ID](#b2-bmc-id) | Unique ID | ✅ |
| 3 | [Customer Segments](#b3-customer-segments) | Text | ✅ |
| 4 | [Value Propositions](#b4-value-propositions) | Text | ✅ |
| 5 | [Channels](#b5-channels) | Text | ✅ |
| 6 | [Customer Relationships](#b6-customer-relationships) | Text | ✅ |
| 7 | [Revenue Streams](#b7-revenue-streams) | Text | ✅ |
| 8 | [Key Resources](#b8-key-resources) | Text | ✅ |
| 9 | [Key Activities](#b9-key-activities) | Text | ✅ |
| 10 | [Key Partnerships](#b10-key-partnerships) | Text | ✅ |
| 11 | [Cost Structure](#b11-cost-structure) | Text | ✅ |
| 12 | [関連VPC](#b12-関連vpc) | Relation | ❌ |
| 13 | [事業ステージ](#b13-事業ステージ) | Select | ❌ |
| 14 | [バージョン](#b14-バージョン) | Text | ✅ |
| 15 | [タグ](#b15-タグ) | Relation | ❌ |
| 16 | [ステータス](#b16-ステータス) | Select | ✅ |
| 17 | [作成日](#b17-作成日) | Date | ✅ |
| 18 | [更新日](#b18-更新日) | Date | ❌ |

---

## B1. 事業名

| 項目 | 値 |
|---|---|
| **型** | Title |
| **必須** | ✅ |

対象事業の名称。1 事業 = 1 BMC が基本。

---

## B2. BMC ID

| 項目 | 値 |
|---|---|
| **型** | Unique ID |
| **必須** | ✅（自動） |
| **プレフィックス** | `BMC-` |

---

## B3. Customer Segments

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

BMC 9ブロック: 誰のために価値を創造するか。VPC の `顧客セグメント` から集約。

---

## B4. Value Propositions

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

BMC 9ブロック: 何の価値を提供するか。VPC の Customer Jobs / Pains / Gains / Pain Relievers / Gain Creators から集約。

---

## B5. Channels

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

BMC 9ブロック: どう届けるか。

---

## B6. Customer Relationships

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

BMC 9ブロック: どう関係を築くか。

---

## B7. Revenue Streams

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

BMC 9ブロック: どう収益を得るか。

---

## B8. Key Resources

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

BMC 9ブロック: 何が必要か。

---

## B9. Key Activities

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

BMC 9ブロック: 何をするか。

---

## B10. Key Partnerships

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

BMC 9ブロック: 誰と組むか。

---

## B11. Cost Structure

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

BMC 9ブロック: 何にコストがかかるか。

---

## B12. 関連VPC

| 項目 | 値 |
|---|---|
| **型** | Relation |
| **接続先** | VPC DB |
| **必須** | ❌ |
| **多重度** | 1対多 |
| **双方向** | ✅ |

詳細: [relations.md §4](./relations.md#4-vpc関連bmc--bmc-db)

---

## B13. 事業ステージ

| 項目 | 値 |
|---|---|
| **型** | Select |
| **必須** | ❌ |

候補値: `アイデア` / `検証` / `立ち上げ` / `成長` / `成熟`。詳細は [select-options.md](./select-options.md#bmc-事業ステージ)。

---

## B14. バージョン

| 項目 | 値 |
|---|---|
| **型** | Text |
| **必須** | ✅ |

`v1.0` / `v2.0` 形式。確定後の大幅修正は新バージョン作成（旧版は `アーカイブ`）。

---

## B15. タグ

| 項目 | 値 |
|---|---|
| **型** | Relation |
| **接続先** | TagDictionary |
| **多重度** | 多対多 |

---

## B16. ステータス

| 項目 | 値 |
|---|---|
| **型** | Select |
| **必須** | ✅ |
| **デフォルト値** | `ドラフト` |

候補値: `ドラフト` / `レビュー中` / `確定` / `アーカイブ`。詳細は [select-options.md](./select-options.md#bmc-ステータス)。

---

## B17. 作成日

| 項目 | 値 |
|---|---|
| **型** | Date |
| **必須** | ✅ |

---

## B18. 更新日

| 項目 | 値 |
|---|---|
| **型** | Date |
| **必須** | ❌ |

---
---

## 関連ドキュメント

- [SPECIFICATION.md](../SPECIFICATION.md) - 全体仕様
- [insight-db-schema.md](./insight-db-schema.md) - Insight DB スキーマ
- [job-db-schema.md](./job-db-schema.md) - Job DB スキーマ
- [vpc-db-schema.md](./vpc-db-schema.md) - VPC DB スキーマ
- [bmc-db-schema.md](./bmc-db-schema.md) - BMC DB スキーマ
- [select-options.md](./select-options.md) - Select選択肢の詳細定義
- [relations.md](./relations.md) - Relation設計の詳細
- [docs/ai-prompts/](../docs/ai-prompts/) - AI autofill プロンプト集
- [docs/governance/human-in-the-point.md](../docs/governance/human-in-the-point.md) - 人手レビュー原則