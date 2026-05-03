# Changelog

このファイルは **insight DB（Notion）設計リポジトリ** の版履歴を記録する。運用上の正本は [`SPECIFICATION.md`](./SPECIFICATION.md)。

形式は [Keep a Changelog](https://keepachangelog.com/ja/1.0.0/) に近づけ、**Semantic Versioning（MAJOR.MINOR）** に従う（[`SPECIFICATION.md`](./SPECIFICATION.md) §10.1 参照）。

## [Unreleased]

### Added

- `schemas/job-db-schema.md` — Job DB スキーマ（v1.0）: JTBD Job 管理、ODI 機会スコア
- `schemas/vpc-db-schema.md` — VPC DB スキーマ（v1.0）: Value Proposition Canvas、Forces→VPC マッピング
- `schemas/bmc-db-schema.md` — BMC DB スキーマ（v1.0）: Business Model Canvas 9ブロック
- `schemas/relations.md` に Job ↔ VPC / VPC ↔ BMC の Relation 仕様を追加
- `SPECIFICATION.md` §2.2 に論理名・スキーマリンク列を追加
- `README.md` にディレクトリ構造セクション、連携DB表を追加
- `docs/integration/` の job/vpc/bmc 接続ドキュメントを新スキーマに整合（プレースホルダ解消）
- `schemas/property-definitions.md` に Job/VPC/BMC プロパティ詳細（Part 2〜4）を追加
- `schemas/select-options.md` に Job/VPC/BMC Select 候補値を統合管理

### Documentation

- ドキュメント間の入力フロー・`job` 必須性・`Forces` 条件付き必須を整合
- `CHANGELOG.md` / `LICENSE` を追加し、README・SPEC からの参照を明確化
- `SPECIFICATION.md` §7.4 に **推奨ビュー仕様**（Tier 1〜3、計 12 ビュー）を追加。デフォルトビューは `戦略候補` に統一

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
