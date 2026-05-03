# 全体アーキテクチャ

## 構成要素

- **Insight DB**: JTBDインサイトの正本。
- **連携DB**: Job / VPC / BMC / TagDictionary（詳細は `docs/integration/`）。

## 境界

- 人間が確定する範囲と、AIが下書きする範囲を分離する（`docs/governance/human-in-the-point.md`）。

## 参照

- `relation-diagram.md`
- `data-flow.md`
