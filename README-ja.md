[English version](README.md)

# dxe-server

DxE ツールキットスイートの可視化サーバー — **DDE 状態の可視化 + 週次レポート**。

> **ステータス: v0.1.0 scaffold — 設計中。**
> `bin/dxe-server.js` はまだ存在しません。`package.json` で宣言されている CLI エントリーポイントはプレースホルダーです。

---

## 解決する問題

DxE-suite（DGE / DDE / DVE / DRE）はプロジェクト横断で状態を蓄積します：

- DRE はインストール状態（FRESH / INSTALLED / OUTDATED / CUSTOMIZED）を追跡しますが、ターミナル表示のみ
- DGE は Gap セッションを `.md` ファイルとして蓄積 — 人間が読む前提で構造集計できない
- **DDE ドキュメント補完状態の可視化がない**

DVE（DxE-suite 内の Decision Visualization Engine）は DGE と DRE の可視化をすでにカバーしています。詳細な重複分析は [docs/relationship-with-dve-ja.md](docs/relationship-with-dve-ja.md) を参照してください。

**Critical Gap #2 と #3 は未解決** — dxe-server の正確なスコープはまだ未決定です。[docs/architecture-ja.md](docs/architecture-ja.md) を参照してください。

---

## 想定スコープ（検討中）

| 機能 | ステータス |
|---|---|
| DDE ドキュメント補完の可視化 | 対象スコープ |
| 週次ダイジェストレポート（DDE 進捗） | 対象スコープ |
| DRE 状態の可視化 | DVE と重複 — Gap 分析参照 |
| DGE Gap ライフサイクルの可視化 | DVE 実装済み — スコープ外の可能性大 |
| マルチプロジェクトスキャン（`/api/scan`） | DVE 実装済み — スコープ外の可能性大 |

---

## DxEシリーズにおける位置づけ

```
DGE-toolkit   → 設計Gap抽出（会話劇）
DDE-toolkit   → ドキュメントGap発見・補完
DVE           → 決定グラフの可視化（Session→Gap→DD→Spec）
DRE-toolkit   → rules/skills の配布・適用
DxE-suite     → 4本を1つのスイートとしてバンドル
─────────────────────────────────────────────────────
dxe-server    → DDE 可視化 + 週次レポート（任意・スタンドアロン）
```

dxe-server は DxE-suite なしで動作します。必須コンポーネントではなく、**任意のアドオン**です。

---

## 想定する使い方（実装後）

```
npx dxe-server start
```

ローカルサーバーを起動してブラウザを開きます。常駐デーモンなし — オンデマンド起動のみ。

---

## 現在の状態

このリポジトリは **scaffold** です。実行可能なコードはまだありません。

実装をブロックしている既知の Gap：

- **Critical Gap #2** — API サーバーの本質的な価値は「複数プロジェクト横断集約」にあります。単一プロジェクトなら DDE CLI のファイル出力で十分です。サーバーのビジネスケースを先に確立する必要があります。
- **Critical Gap #3** — 誰がいつどのくらいの頻度で使うかが未定義です。使用頻度がアーキテクチャを決定します: 毎日使うダッシュボード → 常駐デーモン; 週次確認 → オンデマンド `npx`; 単発 → `dxe status` CLI 出力で十分（サーバー不要）。

これらの Gap を生んだ DGE セッションの詳細は [design-materials/intake/initial-design.md](design-materials/intake/initial-design.md) を参照してください。

---

## ドキュメント

| ドキュメント | 説明 |
|---|---|
| [docs/architecture-ja.md](docs/architecture-ja.md) | コンポーネント責務、DVE との境界、将来の DDE スコープ |
| [docs/relationship-with-dve-ja.md](docs/relationship-with-dve-ja.md) | DVE 重複分析 — DVE が実装済みのもの、差別化候補 |
| [docs/decisions/ADR-001-separate-package.md](docs/decisions/ADR-001-separate-package.md) | 決定: スタンドアロンパッケージ（DxE-suite への統合なし） |
| [docs/decisions/ADR-002-scope-options.md](docs/decisions/ADR-002-scope-options.md) | 未決定: Option A（DVE に吸収）/ B（DDE 専念）/ C（併存） |
| [CHANGELOG.md](CHANGELOG.md) | バージョン履歴 |

---

## コントリビュート

このプロジェクトは初期設計フェーズです。PR を送る前に Issue でディスカッションしてください。

**ライセンス**: MIT
