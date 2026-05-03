# v1 → v2 移行手順

## 前提

- v1 のバックアップを取得済みであること。

## 手順（概要）

1. `scripts/migration/backup-v1.gas` で v1 をエクスポート／コピーする。
2. v2 スキーマ（`schemas/` と `SPECIFICATION.md`）に合わせて列をマッピングする。
3. `scripts/migration/v1-to-v2-migration.gas` をテスト環境で実行する。
4. サンプル件数で検証後、本番バッチを実行する。
5. `scripts/validation/quality-check.gas` で検証する。

## ロールバック

- バックアップから v1 を復元できる手順を別紙で保持する。
