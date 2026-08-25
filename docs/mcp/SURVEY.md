# dxe-server — MCP 化調査（Phase 1）

- **調査日**: 2026-08-22
- **判定**: `defer`
- **repo**: [opaopa6969/dxe-server](https://github.com/opaopa6969/dxe-server)

---

## 概要

dxe-server は DxE ツールキットシリーズ（DGE / DDE / DRE / DVE）のオプション可視化アドオンとして計画されている Node.js サーバ。DVE（Decision Visualization Engine）がカバーしない **DDE ドキュメント補完状態の可視化**と**週次レポート生成**を唯一の防衛可能スコープとする。

2026-08-22 時点で **v0.1.0 scaffold のみ**。`package.json` に `bin/dxe-server.js` が宣言されているがファイルは未作成、実行可能コードは1行も存在しない。`dependencies` / `devDependencies` も未定義。SPEC（1500+ 行）と設計ドキュメントは充実しているが、すべて「計画」状態。

2つの Critical Gap が実装をブロックしている:
- **Gap #2**: API サーバーの本質価値（複数プロジェクト横断集約）が未証明。単一プロジェクトなら DDE CLI のファイル出力で十分。
- **Gap #3**: 利用者プロファイルと使用頻度が未定義。毎日→常駐デーモン、週次→オンデマンド npx、単発→CLI 出力で代替可能。

ADR-002（Option A: DVE 吸収 / B: DDE 専用サーバ / C: レポートジェネレータ）が **未決定**で、これが出ないと API 形態すら確定しない。

---

## 判定と理由

**`defer`** — 以下の前提が揃うまで MCP 化を見送る。

1. **実行可能コードがない**: MCP 化は既存 API/CLI を薄く包むか、新規サーバを立てる必要があるが、包む対象もサーバも存在しない。
2. **API 形態が未確定**: ADR-002 で Option B（常駐サーバ）か Option C（CLI ワンショット）か、あるいは Option A（リポジトリ archive）かが決まらない限り、tool の粒子も job 型かどうかも決められない。
3. **前提データが未実装**: DDE-toolkit が `completion.json` を出力しないと dxe-server は空状態しか返せない。これは DDE 側の変更要件で、dxe-server 単独では解決できない。
4. **再調査のトリガー**: ADR-002 決定 + `bin/dxe-server.js` 実装 + DDE 側で `completion.json` 出力が実装された時点で再調査。

---

## 公開候補

> すべて SPEC に「計画」として記載されているが、実装は未存在。

| kind | name | io | 副作用 | 長時間 | 対応元 |
|---|---|---|---|---|---|
| tool | `dde_status` | `projectId? → {projects: [{id, completionRate, totalPages, completedPages, terms, ddeState}]}` | read | no | SPEC §6.3 `GET /api/dde/status`（未実装） |
| tool | `weekly_report` | `projectId?, format → Markdown\|HTML text` | read | no | SPEC §5.3 / §6 `GET /api/dde/weekly-report`（未実装） |
| tool | `trend` | `projectId, days? → [{date, completionRate}]` | read | no | SPEC §5.4 / §6 `GET /api/dde/trend`（未実装） |
| tool | `register_project` | `path, label?, id? → {id, path, label, registeredAt}` | write | no | SPEC §6.4 `POST /api/projects/register`（未実装） |
| resource | `spec` | `dxe://spec` — DDE 補完状態の機械可読仕様 | — | — | 新規作成 |
| resource | `guide` | `dxe://guide` — dxe-server の使い方 | — | — | 新規作成 |
| skill | `dve-coexistence` | DVE との責務分割と連携手順（locality: repo） | — | — | 新規作成 |

- 壊す系 tool は `register_project`（プロジェクト登録・解除）のみ。`confirm` と dry-run が必要。
- 30 秒超の処理は想定しない（ファイル読み取り + テンプレート生成のみ）。
- `proposed_namespace`: `dxe`

---

## 組み合わせ例

1. `dxe__dde_status` → `design__search_design_resources`（補完率が低いページのデザイン改善候補を探す）
2. `dxe__weekly_report` → `kamishibai__render_start`（週次レポートを台本にして動画サマリーを生成）
3. `dxe__dde_status` → `index__agent_fork`（補完率が低いプロジェクトでドキュメント修正エージェントを起動）

---

## 依存と協調

| 相手 repo | 向き | 能力 | 現存 | 備考 |
|---|---|---|---|---|
| dxe-suite (DDE-toolkit) | depends_on | `completion.json` 出力 | ❌ | DDE 側で新規実装が必要。なければ dxe-server は空状態しか返せない。SPEC 付録 G 参照。 |
| dxe-suite (DVE) | depends_on | `POST /api/scan` プロジェクト発見 | ✅ | 将来候補: DVE の scan 結果から自動インポート。現在は未連携。SPEC 付録 H.3。 |
| dxe-suite (DVE) | provides_to | DDE 補完状態の可視化・週次レポート | ❌ | DVE の空白領域を dxe-server が担う設計だが、dxe-server 自身が未実装。 |

**協調の要否**: DDE 側の `completion.json` 出力実装が必須前提。これは Phase 2（issue-hub）で調整すべき依存関係。

---

## ライブラリのサーバ化

該当しない。dxe-server 自体がサーバ（または CLI）を想定しているが未実装のため、library_serve の枠組みは使わない。

---

## リスク

- **ADR-002 で Option A が選択された場合**: リポジトリ自体が archive され、MCP 化の前提が消える。
- **completion.json 未実装**: dxe-server を実装しても入力データが存在しない。データ契約の合意が先行必須。
- **Critical Gap #2 / #3 未解決**: サーバ化（Option B）が不要になる可能性がある。単発利用なら CLI 出力で十分。
- **localhost バインド想定**: SPEC §10.3 で `localhost` のみを想定。volta 参加には `0.0.0.0` bind + `healthz` + `PORT` 環境変数対応の追加が必要。

---

## 持ち主への質問

1. ADR-002 の決定はいつ出るか？Option A（archive）/ B（常駐サーバ）/ C（CLI レポート）のどれになるか？
2. DDE-toolkit に `completion.json` 出力を追加する計画はあるか？なければ dxe-server は動作不能。
3. Option B が選択された場合、localhost ツールを volta に参加させる価値があるか？それともローカル `npx` で十分か？
4. Option C（CLI レポートジェネレータ）になった場合、常駐サーバでないものを MCP 化するか、CLI を薄く包む tool を作るか？
