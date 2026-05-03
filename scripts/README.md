# スクリプト概要

Google Apps Script（`.gas`）を想定した **移行・検証** 用スクリプト置き場です。実装は各ファイル内コメントを参照してください。

| パス | 用途 |
|------|------|
| `migration/backup-v1.gas` | v1 バックアップ |
| `migration/v1-to-v2-migration.gas` | v1→v2 移行 |
| `validation/quality-check.gas` | 品質チェック |
| `validation/hallucination-detector.gas` | ハルシネーション検出（ヒューリスティック） |

## 実行環境

- Google Sheets / Notion API 連携など、プロジェクト標準の GAS コンテナにデプロイする。
