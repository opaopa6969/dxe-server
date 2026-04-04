# dxe-server — DxE Visualization Server

> DxE toolkit の状態（State）と決定（Decision）を可視化する API サーバー + ブラウザUI。

```
npx dxe-server start
```

ローカルサーバーが起動してブラウザが開く。

---

## DxEシリーズにおける位置づけ

```
DGE-toolkit   → 設計Gap抽出（会話劇）
DDE-toolkit   → ドキュメント補完
DRE-toolkit   → rules/skills 配布・管理
DxE-suite     → 3本のインストール管理
─────────────────────────────────────
dxe-server    → 状態・決定の可視化（任意）
```

DxE-suite を使わなくても単独で動く。インストールは任意。

---

## 可視化対象

### State View（現在の断面）

| ソース | 表示内容 |
|--------|---------|
| DRE | FRESH / INSTALLED / OUTDATED / CUSTOMIZED |
| DGE | Gap一覧。Active / Void / Archived の分布 |
| DDE | ドキュメント補完状態 |

### Decision View（時系列の決定履歴）

| ソース | 表示内容 |
|--------|---------|
| DGE | どのGapをいつ・なぜ解決したか |
| DRE | install/update 履歴 |

---

## ステータス

設計中（v0.1.0）。`design-materials/intake/` に初期設計を格納。
