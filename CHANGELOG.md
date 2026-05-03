# Changelog

このファイルは **insight DB（Notion）設計リポジトリ** の版履歴を記録する。運用上の正本は [`SPECIFICATION.md`](./SPECIFICATION.md)。

形式は [Keep a Changelog](https://keepachangelog.com/ja/1.0.0/) に近づけ、**Semantic Versioning（MAJOR.MINOR）** に従う（[`SPECIFICATION.md`](./SPECIFICATION.md) §10.1 参照）。

## [Unreleased]

### Documentation

- ドキュメント間の入力フロー・`job` 必須性・`Forces` 条件付き必須を整合
- `CHANGELOG.md` / `LICENSE` を追加し、README・SPEC からの参照を明確化

## [2.0] - 2026-05-03

### Added

- Forces（Multi-select）によるラベル化
- `レビュー状態` / `レビュー者` / `レビュー日`
- AI autofill 用プロンプト群（`docs/ai-prompts/`）
- スキーマ分割 MD（`schemas/`）

### Changed

- テキストフィールド統合（要約の構造化フォーマット）
- VPC / BMC 接続方針を Forces ベースに整理

## [1.3] - 2026-05-03

### Added

- `顧客属性` の仮分類拡張

## [1.0] - 初版

- insight DB の初版スコープ
