# 品質基準

## 必須

- Title・要約・Forces のいずれかが空のまま確定にしない。
- 少なくとも **1つ** の観察根拠（リンク・引用・メモ）を付ける。

## 推奨

- Job / VPC / BMC のいずれかと接続できる場合は接続する。
- 要約は `templates/summary-format.md` に準拠する。

## 自動検証

- `scripts/validation/quality-check.gas`（ルールはスクリプト内コメントと同期させる）。
