# dxe-server — 仕様書 (SPEC)

- **バージョン**: 0.1.0-spec
- **日付**: 2026-04-19
- **ステータス**: 設計中。ADR-002 判断待ち。実装コードなし。
- **対象リポジトリ**: `@unlaxer/dxe-server` (opaopa6969/dxe-server)

---

## 目次

1. [概要](#1-概要)
2. [機能仕様](#2-機能仕様)
3. [データ永続化層](#3-データ永続化層)
4. [ステートマシン](#4-ステートマシン)
5. [ビジネスロジック](#5-ビジネスロジック)
6. [API / 外部境界](#6-api--外部境界)
7. [UI](#7-ui)
8. [設定](#8-設定)
9. [依存関係](#9-依存関係)
10. [非機能要件](#10-非機能要件)
11. [テスト戦略](#11-テスト戦略)
12. [デプロイ / 運用](#12-デプロイ--運用)

---

## 1. 概要

### 1.1 パッケージ識別

| 項目 | 値 |
|---|---|
| パッケージ名 | `@unlaxer/dxe-server` |
| 現バージョン | `0.1.0` (scaffold) |
| ライセンス | MIT |
| CLI エントリ | `bin/dxe-server.js` (未作成) |
| 起動方法 | `npx dxe-server start` |

### 1.2 目的

dxe-server は DxE ツールキットシリーズのオプション可視化アドオンである。DxE シリーズ（DGE / DDE / DRE / DVE / DxE-suite）の中で、dxe-server が担う唯一の防衛可能なスコープは **DDE ドキュメント補完状態の可視化と週次レポート生成**である。

```
DGE-toolkit   → 設計 Gap 抽出（会話劇）
DDE-toolkit   → ドキュメント補完
DRE-toolkit   → rules/skills 配布・管理
DVE           → 決定グラフ可視化（Session→Gap→DD→Spec）
DxE-suite     → 上4本のインストール管理
──────────────────────────────────────────
dxe-server    → DDE 可視化 + 週次レポート（任意、スタンドアロン）
```

### 1.3 現在の開発ステータス

**v0.1.0 は scaffold のみ。実行可能コードは存在しない。**

- `bin/dxe-server.js` は `package.json` に宣言されているが未作成
- `npx dxe-server start` を実行するとファイル不存在エラーが発生する
- 全機能は「計画」または「設計中」の状態

### 1.4 DVE との責務重複課題

DVE（DxE-suite の一部、`@unlaxer/dve-toolkit` v4.2.0）がすでに以下を実装済みである:

| DVE 実装済み機能 | dxe-server での扱い |
|---|---|
| `state-detector.ts` — DRE FRESH/INSTALLED/OUTDATED/CUSTOMIZED 検出 | **スコープ外**（DVE に委譲） |
| `POST /api/scan` — マルチプロジェクト発見 | **スコープ外**（DVE に委譲） |
| `POST /api/register` — プロジェクト登録 | **スコープ外**（DVE に委譲） |
| Preact + Cytoscape Web UI（Session→Gap→DD→Spec グラフ） | **スコープ外**（DVE に委譲） |
| `GET /api/graph` — 決定グラフ全体取得 | **スコープ外**（DVE に委譲） |

DVE が**カバーしない**領域が dxe-server の存在理由である:

| DVE の空白領域 | dxe-server のターゲット |
|---|---|
| DDE 内部補完状態（どのページが完成か、どの用語がリンク済みか） | **対象スコープ** |
| DDE 時系列進捗（補完率の推移） | **対象スコープ** |
| 週次ダイジェストレポート（Markdown / HTML） | **対象スコープ** |
| DDE 中心のビュー（DVE は DGE 中心） | **対象スコープ** |

### 1.5 未解決の重要問題（ADR-002 判断待ち）

以下の2つの Critical Gap が解決されるまで実装は開始できない:

- **Critical Gap #2**: API サーバーの本質価値は複数プロジェクト横断集約にある。単一プロジェクトであれば DDE CLI のファイル出力で十分。サーバーのビジネスケースが未証明。
- **Critical Gap #3**: 利用者プロファイルと使用頻度が未定義。毎日使用するなら常駐デーモン、週次確認ならオンデマンド `npx`、単発ならば `dde status` CLI 出力がサーバーを代替できる。

**ADR-002 (Option A / B / C) の判断が出るまで、本仕様書はすべての実装詳細を「計画」として記述する。**

---

## 2. 機能仕様

> **ステータス**: scaffold only。Critical Gap #2 / #3 未解決のため機能確定不可。

### 2.1 確定スコープ（ADR-002 Option B / C 共通）

ADR-002 の選択肢によらず dxe-server が担うことが合意済みの機能:

| 機能 ID | 機能名 | 説明 |
|---|---|---|
| F-01 | DDE 補完状態ビュー | プロジェクトごとの DDE 補完率・未完成ページ一覧・未リンク用語数の表示 |
| F-02 | 週次ダイジェストレポート | DDE 進捗を Markdown または HTML で出力する定期レポート |
| F-03 | プロジェクト登録 | dxe-server が監視するプロジェクトの登録・解除 |

### 2.2 スコープ外（DVE に委譲）

| 機能 | 委譲先 |
|---|---|
| DRE ステート可視化（FRESH / INSTALLED / OUTDATED / CUSTOMIZED） | DVE |
| DGE Gap ライフサイクル（Active / Void / Archived） | DVE |
| マルチプロジェクトスキャン（ディスク探索） | DVE `POST /api/scan` |
| Session→Gap→DD→Spec 決定グラフ表示 | DVE Preact + Cytoscape UI |

### 2.3 ADR-002 オプション別の機能差分

#### Option A — DVE への吸収（現状最小オプション）

dxe-server リポジトリを廃止または archive。DDE 可視化を DVE の Preact アプリに新ビューとして追加。dxe-server での実装不要。

#### Option B — DDE 専用 dxe-server（推奨出発点）

| 機能 ID | 機能名 | 実装形態 |
|---|---|---|
| F-01 | DDE 補完状態ビュー | Node.js サーバー + ブラウザ UI（ミニマル） |
| F-02 | 週次ダイジェストレポート | `GET /api/dde/weekly-report` → Markdown / HTML レスポンス |
| F-03 | プロジェクト登録 | 設定ファイルベースの登録管理 |

#### Option C — レポートジェネレーター（サーバーなし）

| 機能 ID | 機能名 | 実装形態 |
|---|---|---|
| F-02 | 週次ダイジェストレポート | `npx dxe-server report` → 静的 Markdown / HTML ファイル生成 |
| F-03 | プロジェクト登録 | 設定ファイルベースの登録管理（サーバーなし） |
| F-01 | DDE 補完状態ビュー | CLI 出力（ブラウザ不要） |

Option C では持続的サーバーは不要。Gap #3（使用頻度問題）を回避できる。

### 2.4 機能の優先順位（Option B 前提）

| 優先度 | 機能 | 理由 |
|---|---|---|
| P0 | `bin/dxe-server.js` CLI エントリ作成 | 現在の最大ブロッカー |
| P0 | プロジェクト登録機能（F-03） | 他のすべての機能の前提 |
| P1 | DDE 補完状態 API（F-01） | 差別化コア機能 |
| P1 | 週次レポート生成（F-02） | DVE が持たない具体的成果物 |
| P2 | ブラウザ UI | F-01 の視覚化（Option C では不要） |

---

## 3. データ永続化層

> **ステータス**: 計画のみ。ADR-002 判断待ち。

### 3.1 データソース

dxe-server が読み取る外部データソース:

| データソース | パス（例） | 内容 |
|---|---|---|
| DDE 用語抽出結果 | `{project}/dde/terms/` | 抽出済み用語一覧 |
| DDE 生成記事 | `{project}/dde/articles/` | 生成ドキュメント |
| DDE 補完サマリー | `{project}/dde/completion.json` | **未実装**（DDE 側で新設が必要） |
| プロジェクトレジストリ | `~/.config/dxe-server/projects.json` | dxe-server が管理するプロジェクト一覧 |
| DRE 設定（参照のみ） | `{project}/.claude/` | DRE ステート（DVE に委譲するが参照は可） |

### 3.2 DDE 補完サマリーフォーマット（計画）

現時点で DDE は機械可読な補完サマリーを出力しない。dxe-server の動作には DDE 側での以下の新機能が必要:

```jsonc
// {project}/dde/completion.json (計画・フォーマット未確定)
{
  "generatedAt": "2026-04-19T00:00:00Z",
  "ddeVersion": "x.y.z",
  "project": "/path/to/project",
  "completionRate": 0.73,
  "totalPages": 42,
  "completedPages": 30,
  "pendingPages": 12,
  "terms": {
    "total": 156,
    "linked": 120,
    "unlinked": 36
  },
  "gaps": [
    {
      "pageId": "architecture.md",
      "missingTerms": ["DGE", "DVE"],
      "completionRate": 0.5
    }
  ]
}
```

このファイルが存在しない場合、dxe-server は「DDE 補完データ未生成」として扱い、エラーではなく空状態を返す。

### 3.3 プロジェクトレジストリ（計画）

```jsonc
// ~/.config/dxe-server/projects.json (計画・フォーマット未確定)
{
  "version": 1,
  "registeredAt": "2026-04-19T00:00:00Z",
  "projects": [
    {
      "id": "my-project",
      "path": "/home/user/work/my-project",
      "label": "My Project",
      "registeredAt": "2026-04-19T00:00:00Z",
      "enabled": true
    }
  ]
}
```

### 3.4 永続化戦略

| 要件 | 選択肢 | 現状 |
|---|---|---|
| プロジェクトレジストリ | JSON ファイル（`~/.config/dxe-server/`） | 計画 |
| DDE 状態キャッシュ | メモリキャッシュ（TTL: 5分）またはなし | 未決定 |
| 週次レポート履歴 | JSON ファイル（プロジェクトディレクトリ内） | 未決定 |
| データベース（SQLite 等） | Option B の将来拡張として検討 | 保留 |

Option C（レポートジェネレーター）であれば永続化層は不要。実行時に DDE 出力ファイルを直接読むだけで十分。

---

## 4. ステートマシン

> **ステータス**: 計画のみ。ADR-002 判断待ち。

### 4.1 dxe-server プロセスステートマシン

Option B（常駐サーバー）の場合:

```
         起動
          │
          ▼
      ┌─────────┐
      │ STARTING │  設定ファイル読み込み、ポートバインド
      └────┬────┘
           │ 成功
           ▼
      ┌─────────┐
      │ RUNNING  │  リクエスト受付中
      └────┬────┘
           │ Ctrl+C / SIGTERM
           ▼
      ┌──────────┐
      │ STOPPING │  接続クローズ、ファイルフラッシュ
      └────┬─────┘
           │
           ▼
        停止完了
```

Option C（ワンショット）の場合: `RUNNING` ステートなし。`STARTING → GENERATING → DONE` の3ステートのみ。

### 4.2 DDE 補完状態ステートマシン（プロジェクト単位）

各登録プロジェクトの DDE 補完状態:

```
          登録
           │
           ▼
    ┌──────────────┐
    │  UNINITIALIZED │  completion.json が存在しない
    └───────┬────────┘
            │ DDE が completion.json を生成
            ▼
    ┌──────────────┐
    │   TRACKED    │  補完データあり、正常トラッキング中
    └───────┬────────┘
            │ completion.json が古い（TTL 超過）
            ▼
    ┌──────────────┐
    │    STALE     │  データ陳腐化（再実行を促す）
    └───────┬────────┘
            │ DDE 再実行 → completion.json 更新
            ▼
    ┌──────────────┐
    │   TRACKED    │  （更新完了で TRACKED に戻る）
    └──────────────┘
```

### 4.3 週次レポートジョブステートマシン

```
IDLE → SCHEDULED → GENERATING → COMPLETED
                              └→ FAILED → IDLE (再試行)
```

このステートマシンは Option B（常駐サーバー）でのみ意味を持つ。Option C では `GENERATING → COMPLETED / FAILED` の2ステートのみ。

---

## 5. ビジネスロジック

> **ステータス**: DVE 既実装との差別化候補3種を定義。実装なし。

### 5.1 DVE 実装済み vs dxe-server 差別化候補

| 機能領域 | DVE 実装状況 | dxe-server の差別化候補 |
|---|---|---|
| DRE ステート検出 | 完全実装（`state-detector.ts`） | **なし**（完全委譲） |
| DGE Gap ライフサイクル | 完全実装（グラフビュー） | **なし**（完全委譲） |
| DDE 補完率 | `hasDDE: boolean` のみ | **差別化候補 1: DDE ドキュメント補完率ダッシュボード** |
| 週次レポート | なし | **差別化候補 2: 週次 Markdown ダイジェスト** |
| 時系列トレンド | なし（現在状態のみ） | **差別化候補 3: DDE 補完率の時系列推移** |

### 5.2 差別化候補 1 — DDE ドキュメント補完率ダッシュボード

**入力**: `{project}/dde/completion.json`（DDE 側で新設が必要）

**処理ロジック**:
1. 登録済みプロジェクトの `completion.json` を読み込む
2. プロジェクト別に補完率（`completedPages / totalPages`）を計算
3. 未リンク用語数（`terms.unlinked`）でソート（優先度付け）
4. 複数プロジェクトを集約してダッシュボードデータを構築

**出力**:
- `GET /api/dde/status` → JSON レスポンス（プロジェクト別補完状態）
- ブラウザ UI に補完率バーとして表示

**DVE との差異**: DVE は `hasDDE: boolean` しか持たない。補完率・ページ別状態・未リンク用語は dxe-server が初めて可視化する。

### 5.3 差別化候補 2 — 週次 Markdown ダイジェスト

**入力**: 登録プロジェクト全件の `completion.json` + 前回レポートのスナップショット（差分計算用）

**処理ロジック**:
1. 全登録プロジェクトの現在補完率を取得
2. 前回スナップショットと比較して変化量（`delta`）を計算
3. Markdown テンプレートに補完率・デルタ・未完成ページ上位5件を埋め込む
4. HTML オプション: Markdown を HTML に変換して添付メール・Slack 投稿に対応

**出力フォーマット例（Markdown）**:

```markdown
# DDE 週次ダイジェスト — 2026-04-19

## サマリー

| プロジェクト | 補完率 | 先週比 |
|---|---|---|
| my-project | 73% | +8% |
| another-project | 45% | +2% |

## 要対応: 未完成ページ上位5件（my-project）

1. `architecture.md` — 50% (未リンク用語: DGE, DVE)
2. `api-reference.md` — 60% (未リンク用語: endpoint, schema)
...
```

**DVE との差異**: DVE はインタラクティブ UI のみ。静的レポート出力は完全に異なる UX モデル。

### 5.4 差別化候補 3 — DDE 補完率の時系列推移

**入力**: スナップショットの蓄積（週次または日次で `completion.json` をアーカイブ）

**処理ロジック**:
1. `{project}/dde/history/YYYY-MM-DD.json` に補完スナップショットを保存
2. 任意期間の補完率変化を集計（30日間デフォルト）
3. JSON でトレンドデータを返す（折れ線グラフ用）

**出力**: `GET /api/dde/trend?project=my-project&days=30` → 日付×補完率の配列

**DVE との差異**: DVE グラフは現在状態のみ。時系列データモデルは DVE が持たない。

### 5.5 ビジネスロジックの前提条件

上記3候補はすべて以下を前提とする（現時点で未実装）:

- DDE 側で `completion.json` を出力する機能の追加（dxe-server ではなく DDE-toolkit 側の変更）
- プロジェクト登録機能（F-03）の実装
- Critical Gap #2（単一プロジェクトユーザーへの価値証明）の解決

---

## 6. API / 外部境界

> **ステータス**: 計画のみ。DVE と重複するエンドポイントは除外済み。

### 6.1 エンドポイント一覧（Option B 計画）

| メソッド | パス | 機能 | DVE 重複 |
|---|---|---|---|
| `GET` | `/api/health` | ヘルスチェック | なし（独自実装） |
| `GET` | `/api/dde/status` | 登録プロジェクト全件の DDE 補完状態 | **なし**（DVE にない） |
| `GET` | `/api/dde/status/:projectId` | 特定プロジェクトの DDE 詳細補完状態 | **なし**（DVE にない） |
| `GET` | `/api/dde/weekly-report` | 週次ダイジェストレポート（Markdown / HTML） | **なし**（DVE にない） |
| `GET` | `/api/dde/trend` | DDE 補完率の時系列トレンド | **なし**（DVE にない） |
| `POST` | `/api/projects/register` | プロジェクト登録 | DVE の `POST /api/register` と名前重複 — **別機能**（DDE 専用） |
| `DELETE` | `/api/projects/:projectId` | プロジェクト登録解除 | なし |
| `GET` | `/api/projects` | 登録プロジェクト一覧 | なし |
| `GET` | `/` | ブラウザ UI（SPA） | なし（DVE とは別 UI） |

### 6.2 DVE と重複するエンドポイントの扱い

以下は DVE が実装済みのため dxe-server に実装**しない**:

| エンドポイント | DVE 実装 | dxe-server の扱い |
|---|---|---|
| `POST /api/scan` | DVE `localhost:4174` | 実装しない |
| `GET /api/graph` | DVE `localhost:4174` | 実装しない |
| DRE ステート API | DVE `state-detector.ts` | 実装しない |

### 6.3 `GET /api/dde/status` レスポンス仕様（計画）

```jsonc
// HTTP 200
{
  "generatedAt": "2026-04-19T12:00:00Z",
  "projects": [
    {
      "id": "my-project",
      "path": "/home/user/work/my-project",
      "label": "My Project",
      "ddeState": "TRACKED",   // UNINITIALIZED | TRACKED | STALE
      "completionRate": 0.73,
      "totalPages": 42,
      "completedPages": 30,
      "pendingPages": 12,
      "terms": {
        "total": 156,
        "linked": 120,
        "unlinked": 36
      },
      "lastUpdated": "2026-04-18T10:00:00Z"
    }
  ]
}
```

エラーケース:

```jsonc
// HTTP 404 — プロジェクト未登録
{ "error": "PROJECT_NOT_FOUND", "projectId": "unknown" }

// HTTP 503 — completion.json 未生成（DDE 未実行）
{ "error": "DDE_COMPLETION_NOT_AVAILABLE", "projectId": "my-project",
  "hint": "Run `dde generate` in the project directory first." }
```

### 6.4 `POST /api/projects/register` リクエスト仕様（計画）

```jsonc
// リクエストボディ
{
  "path": "/home/user/work/my-project",  // 必須
  "label": "My Project",                 // 任意
  "id": "my-project"                     // 任意（省略時は path から自動生成）
}

// レスポンス HTTP 201
{
  "id": "my-project",
  "path": "/home/user/work/my-project",
  "label": "My Project",
  "registeredAt": "2026-04-19T12:00:00Z"
}
```

### 6.5 Option C のコマンドライン境界

Option C（レポートジェネレーター）ではサーバーを起動せず、CLI コマンドとして動作する:

```bash
# 標準出力に Markdown を出力
npx dxe-server report

# HTML ファイルとして出力
npx dxe-server report --format html --output ./dde-report.html

# 特定プロジェクトのみ
npx dxe-server report --project /path/to/project

# プロジェクト登録（設定ファイル更新のみ）
npx dxe-server register /path/to/project
```

---

## 7. UI

> **ステータス**: 計画のみ。ADR-002 Option B が選択された場合のみ実装。

### 7.1 UI の位置付け

DVE は Preact + Cytoscape による高機能な決定グラフ UI を持つ。dxe-server の UI はそれと競合せず、**DDE 補完状態に特化したミニマル UI**を目指す。

Option C（レポートジェネレーター）では UI 不要。静的 Markdown / HTML ファイルが最終成果物となる。

### 7.2 ブラウザ UI 構成（Option B 計画）

| ページ | パス | 内容 |
|---|---|---|
| ダッシュボード | `/` | 全登録プロジェクトの DDE 補完率一覧（バーグラフ） |
| プロジェクト詳細 | `/project/:id` | 特定プロジェクトのページ別補完状態、未リンク用語一覧 |
| 週次レポート | `/report` | 最新週次レポートのブラウザ表示 |
| トレンド | `/trend` | 補完率の時系列折れ線グラフ |
| 設定 | `/settings` | プロジェクト登録・解除 |

### 7.3 技術スタック（計画・未確定）

| 要素 | 候補 | 選定理由 |
|---|---|---|
| フレームワーク | Preact または Vanilla JS | DVE が Preact を使用 — 統一性あり |
| グラフ描画 | Chart.js または D3.js | 折れ線グラフ・バーグラフに特化（Cytoscape は不要） |
| スタイル | CSS Modules または TailwindCSS | 未決定 |
| バンドラー | Vite | DxE シリーズとの統一 |

DVE は Cytoscape（ノード/エッジグラフ）を使用するが、dxe-server には補完率バーや折れ線グラフのほうが適している。

### 7.4 UI の最小実装（P0 相当）

実装を開始する場合の最小要件:

1. 全登録プロジェクトの補完率を数値とバーグラフで表示
2. DVE へのリンク（「決定グラフ表示は DVE を使用」の明示）
3. レスポンシブ不要（デスクトップ専用でよい）
4. ブラウザ自動起動（`npx dxe-server start` で `localhost:PORT` を開く）

---

## 8. 設定

> **ステータス**: DRE の `.claude/` ディレクトリ構造を参照して設計。

### 8.1 設定ファイルの場所

dxe-server は DRE の状態管理パターンを参考に、ユーザーグローバル設定とプロジェクト固有設定を分離する:

| 設定ファイル | パス | 内容 |
|---|---|---|
| グローバル設定 | `~/.config/dxe-server/config.json` | サーバー設定（ポート、ホスト、ログレベル等） |
| プロジェクトレジストリ | `~/.config/dxe-server/projects.json` | 登録プロジェクト一覧 |
| レポート履歴 | `~/.config/dxe-server/reports/` | 過去の週次レポートスナップショット |

### 8.2 グローバル設定スキーマ（計画）

```jsonc
// ~/.config/dxe-server/config.json
{
  "version": 1,
  "server": {
    "port": 4175,          // DVE が 4174 を使用するため 4175 を計画
    "host": "localhost",
    "autoOpenBrowser": true
  },
  "report": {
    "format": "markdown",  // "markdown" | "html"
    "outputDir": "./",     // レポートファイルの出力先（Option C）
    "weeklySchedule": null // cron 式（Option B 常駐時）。null = スケジュールなし
  },
  "dde": {
    "completionJsonPath": "dde/completion.json",  // プロジェクトルートからの相対パス
    "cacheMaxAgeMinutes": 5
  }
}
```

### 8.3 DRE 状態との関係

dxe-server は DRE の `.claude/` ディレクトリを**直接書き込まない**。DRE ステート（FRESH / INSTALLED / OUTDATED / CUSTOMIZED）の読み取りは DVE に委譲する。

dxe-server が `.claude/` を参照するユースケースは現時点で想定しない（DVE が同データを既に提供するため）。

### 8.4 環境変数（計画）

| 変数名 | デフォルト | 説明 |
|---|---|---|
| `DXE_SERVER_PORT` | `4175` | サーバーポート番号 |
| `DXE_SERVER_CONFIG` | `~/.config/dxe-server/config.json` | 設定ファイルのパスを上書き |
| `DXE_SERVER_LOG_LEVEL` | `info` | `debug` / `info` / `warn` / `error` |

---

## 9. 依存関係

> **ステータス**: DVE の存在により検討中。ADR-002 判断待ち。

### 9.1 現在の依存（`package.json` 確定済み）

```json
{
  "name": "@unlaxer/dxe-server",
  "version": "0.1.0"
}
```

現時点では `dependencies` / `devDependencies` が未定義。

### 9.2 計画依存（Option B 前提）

| パッケージ | 種別 | 用途 |
|---|---|---|
| Node.js `>=18` | ランタイム | ES Modules、`fs/promises`、`http` |
| `marked` または `remark` | `dependencies` | Markdown → HTML 変換（週次レポート） |
| `Preact` | `devDependencies` | ブラウザ UI（Option B のみ） |
| `Vite` | `devDependencies` | SPA バンドル（Option B のみ） |
| `@unlaxer/dxe-suite` 型定義 | `devDependencies` | DDE データ構造の型参照（実装時に要確認） |

### 9.3 DVE との依存関係

**dxe-server は DVE を依存として追加しない。**

理由:
- DVE は DxE-suite の内部パッケージ（`@unlaxer/dve-toolkit`）であり、単体 npm インストールが想定されていない
- dxe-server はオプション add-on — DVE インストール済みか否かに関わらず動作する必要がある
- DVE のデータを利用するユースケースがあれば、DVE の API（`localhost:4174`）を HTTP 経由で呼び出す（パッケージ依存ではなくランタイム連携）

### 9.4 DDE への依存

dxe-server は DDE の出力ファイル（`completion.json`）を読み取るが、DDE パッケージに直接依存しない。ファイルベースの連携（ゼロ依存）を維持する。

ただし、DDE-toolkit が `completion.json` を出力する機能を追加しない限り、dxe-server は動作できない。これは **DDE 側の変更要件**であり、本仕様の範囲外である（DDE-toolkit の仕様書に記載すべき項目）。

### 9.5 Node.js バージョン要件

| 要件 | 理由 |
|---|---|
| Node.js `>=18.0.0` | ES Modules、`fetch` ネイティブ対応、`fs/promises` の安定性 |
| npm `>=9.0.0` | `package.json` の `exports` フィールド対応 |

---

## 10. 非機能要件

### 10.1 パフォーマンス

| 要件 | 目標値 | 備考 |
|---|---|---|
| `/api/dde/status` レスポンス時間 | < 200ms（キャッシュヒット時） | キャッシュ TTL: 5分 |
| `/api/dde/status` レスポンス時間 | < 2s（キャッシュミス、ファイル再読み込み） | プロジェクト数 ≤ 20 |
| 週次レポート生成時間 | < 5s | プロジェクト数 ≤ 20 |
| メモリ使用量 | < 100MB | 常駐サーバー（Option B） |
| 起動時間 | < 3s | `npx dxe-server start` からブラウザ表示まで |

### 10.2 信頼性

| 要件 | 仕様 |
|---|---|
| `completion.json` 不存在時 | エラーではなく `UNINITIALIZED` 状態を返す |
| 登録プロジェクトのパスが存在しない場合 | 警告ログを出し、当該プロジェクトをスキップ |
| DVE 非起動時 | dxe-server は DVE に依存しないため影響なし |
| サーバークラッシュ | プロセス再起動後に設定ファイルから状態を復元 |

### 10.3 セキュリティ

| 要件 | 仕様 |
|---|---|
| バインドアドレス | `localhost` のみ（外部公開なし） |
| 認証 | v0.1.x では不要（ローカルツール） |
| パストラバーサル | プロジェクトパスの正規化・バリデーションを実施 |
| プロジェクトディレクトリ外アクセス | 禁止（登録済みパス内のみ読み取り可） |

### 10.4 互換性

| 要件 | 仕様 |
|---|---|
| OS | Linux / macOS / Windows（WSL2） |
| Node.js | `>=18.0.0` |
| DVE の有無 | 独立して動作（DVE インストールは不要） |
| DxE-suite の有無 | 独立して動作（DxE-suite インストールは不要） |

---

## 11. テスト戦略

> **ステータス**: 未実装。以下は計画のみ。

### 11.1 テスト方針

| 方針 | 詳細 |
|---|---|
| ユニットテスト優先 | ビジネスロジック（補完率計算、レポート生成）は単体でテスト可能 |
| ファイル I/O はモック化 | `completion.json` の読み込みをモック化してテスト環境を独立させる |
| E2E は最小限 | サーバー起動 → API レスポンス → シャットダウンのスモークテストのみ |
| DVE 不要 | dxe-server のテストスイートに DVE は不要 |

### 11.2 テストカテゴリと対象（計画）

#### ユニットテスト

| テスト対象 | 主要なテストケース |
|---|---|
| `completionRateCalculator` | `completedPages / totalPages` の正確な計算、ゼロ除算対応 |
| `projectRegistry` | 登録・解除・一覧取得、重複登録の防止、存在しないパスの検出 |
| `reportGenerator` | Markdown テンプレート正確出力、差分（delta）計算、空プロジェクト時の graceful 出力 |
| `ddeStateResolver` | `UNINITIALIZED / TRACKED / STALE` の正確な判定 |
| `configLoader` | 設定ファイル未存在時のデフォルト値適用 |

#### インテグレーションテスト

| テスト対象 | 主要なテストケース |
|---|---|
| `GET /api/dde/status` | completion.json あり/なしで正しいレスポンスを返す |
| `POST /api/projects/register` | プロジェクトが実際に登録され `projects.json` に書き込まれる |
| `GET /api/dde/weekly-report` | Markdown レスポンスが正しい構造を持つ |
| エラーハンドリング | 不正パス・不正 JSON・TTL 超過でも 5xx を返さない |

#### E2E テスト（スモーク）

```bash
# スモークテストシナリオ（計画）
npx dxe-server start --port 4999 &
curl http://localhost:4999/api/health  # → 200 OK
curl http://localhost:4999/api/dde/status  # → 200 {"projects": []}
kill %1
```

### 11.3 テストフレームワーク（計画・未確定）

| 用途 | 候補 | 理由 |
|---|---|---|
| ユニット + インテグレーション | Vitest | DxE シリーズの統一（Vite ベース） |
| E2E | Supertest | Node.js サーバーテストの標準 |
| カバレッジ | Vitest (`--coverage`) | v8 カバレッジプロバイダ |

### 11.4 CI 統合（計画）

```yaml
# .github/workflows/ci.yml (計画)
# jobs:
#   test:
#     - npm install
#     - npm run test:unit
#     - npm run test:integration
#     - npm run test:smoke
```

ADR-002 判断後に CI ワークフローを実装する。現時点では scaffold のため CI は未設定。

---

## 12. デプロイ / 運用

> **ステータス**: scaffold only。ADR-002 判断待ち。

### 12.1 配布方式

| 方式 | 説明 | ステータス |
|---|---|---|
| `npx dxe-server` | インストール不要でワンショット実行 | 計画（`bin/dxe-server.js` 未作成） |
| `npm install -g @unlaxer/dxe-server` | グローバルインストール | 計画 |
| `npm install --save-dev @unlaxer/dxe-server` | プロジェクトローカルインストール | 計画 |

### 12.2 npm 公開設定（計画）

```jsonc
// package.json 拡張計画
{
  "main": "bin/dxe-server.js",
  "files": [
    "bin/",
    "dist/",        // ブラウザ UI の静的ファイル（Option B のみ）
    "README.md",
    "CHANGELOG.md"
  ],
  "engines": {
    "node": ">=18.0.0"
  }
}
```

### 12.3 リリースフロー（計画）

```
feature ブランチ → PR → main マージ → CHANGELOG.md 更新 → npm publish
```

DVE を含む DxE-suite のリリースサイクルとは**独立**している（ADR-001 の決定）。

### 12.4 運用モデル

| シナリオ | 運用形態 |
|---|---|
| 個人開発者・週次確認 | `npx dxe-server start`（オンデマンド） |
| チームリード・日次確認 | グローバルインストール + 常駐デーモン（Option B） |
| CI 組み込み | `npx dxe-server report > dde-report.md`（Option C） |
| Slack 週次通知 | Option B cron スケジュール + Webhook 送信（将来計画） |

### 12.5 ポート割り当て

DxE シリーズ内のポート競合を避けるため:

| サービス | ポート |
|---|---|
| DVE（preview + API） | `4174` |
| DVE（dev） | `4173` |
| **dxe-server（計画）** | **`4175`** |

`DXE_SERVER_PORT` 環境変数で上書き可能。

### 12.6 ログ出力

| レベル | 用途 |
|---|---|
| `info` | サーバー起動・停止、プロジェクト登録・解除 |
| `debug` | ファイル読み込み詳細、キャッシュヒット/ミス |
| `warn` | 登録プロジェクトのパスが存在しない、`completion.json` が古い |
| `error` | サーバー起動失敗、予期しない例外 |

---

## 付録 A — 用語集

| 用語 | 説明 |
|---|---|
| DDE | Document Debugging Engine — ドキュメント補完ツール（DxE シリーズの一員） |
| DVE | Decision Visualization Engine — 決定グラフ可視化ツール（DxE-suite 内） |
| DGE | Design Gap Extraction — 設計 Gap 抽出ツール（DxE シリーズの一員） |
| DRE | DxE Rules Engine — rules/skills 配布・管理ツール（DxE シリーズの一員） |
| DxE-suite | DGE + DDE + DRE + DVE のインストール管理パッケージ |
| Gap | 設計上の不足・未解決問題（DGE が抽出する） |
| completion.json | DDE が出力予定の機械可読補完サマリー（現時点で未実装） |
| ADR | Architecture Decision Record — アーキテクチャ決定の記録ドキュメント |
| Critical Gap #2 | API サーバーの価値命題（複数プロジェクト横断集約）が未証明の問題 |
| Critical Gap #3 | 利用者プロファイルと使用頻度が未定義の問題 |

---

## 付録 B — 関連ドキュメント

| ドキュメント | パス | 説明 |
|---|---|---|
| アーキテクチャ | `docs/architecture.md` | コンポーネント責務、DVE との境界 |
| DVE との関係 | `docs/relationship-with-dve.md` | DVE 重複分析、差別化候補 |
| ADR-001 | `docs/decisions/ADR-001-separate-package.md` | 別パッケージ化の決定 |
| ADR-002 | `docs/decisions/ADR-002-scope-options.md` | スコープ選択肢（OPEN） |
| 初期設計 | `design-materials/intake/initial-design.md` | DGE セッション出力 |

---

*本 SPEC は ADR-002 が解決された時点で更新される。Option A が選択された場合、本リポジトリは archive され SPEC は廃止となる。Option B または C が選択された場合、各セクションの「計画のみ」表記を「実装済み」に置き換えていく。*

---

## 付録 C — ADR-002 選択肢の詳細比較マトリクス

ADR-002 の判断を支援するため、3つの選択肢を多角的に比較する。

### C.1 機能カバレッジ比較

| 機能 | Option A（DVE 吸収） | Option B（DDE 専用サーバー） | Option C（レポートジェネレーター） |
|---|---|---|---|
| DDE 補完率表示 | DVE の新ビューとして実装 | `GET /api/dde/status` + ブラウザ UI | `npx dxe-server report` CLI 出力 |
| 週次 Markdown レポート | DVE の「エクスポート」機能として | `GET /api/dde/weekly-report` | `npx dxe-server report --format markdown` |
| 時系列トレンド | DVE のグラフ拡張として | `GET /api/dde/trend` + 折れ線グラフ | CLI テキスト表（限定的） |
| マルチプロジェクト集約 | DVE `POST /api/scan` を継続利用 | 独自プロジェクトレジストリ | 設定ファイルから静的に読む |
| ブラウザ UI | DVE の既存 UI を拡張 | 独自ミニマル SPA | なし（静的ファイル出力のみ） |
| CI 組み込み | 困難（DVE はブラウザ必須） | `curl http://localhost:4175/api/dde/weekly-report` | `npx dxe-server report > report.md`（容易） |
| インストール不要 | DVE インストール必須 | `npx dxe-server start` | `npx dxe-server report` |

### C.2 リスク比較

| リスク | Option A | Option B | Option C |
|---|---|---|---|
| DVE との重複 | なし（吸収するため） | 低（DDE 専用スコープ） | なし（非サーバー） |
| DxE-suite 変更影響 | **高**（DVE 改修が必要） | 低 | 低 |
| 実装コスト | **高**（DVE 内部変更） | 中（新規サーバー） | **低**（CLI ツール） |
| Gap #2 解決 | 不要（DVE が継続） | 要解決 | 回避可能 |
| Gap #3 解決 | 不要（DVE が継続） | 要解決 | **回避**（常駐不要） |
| メンテナンス | dxe-server 廃止 → 低 | 中（独立パッケージ） | **低**（シンプル） |
| 採用ハードル | DVE 必須 → 中 | 独立 → **低** | `npx` 一発 → **最低** |

### C.3 ユーザーペルソナ別適合性

| ペルソナ | Option A | Option B | Option C |
|---|---|---|---|
| 個人開発者（1プロジェクト、週1確認） | △ DVE も必要 | ◯ オンデマンド起動 | ◎ `npx` 一発 |
| チームリード（5プロジェクト、日次確認） | ◯ DVE に統合 | ◎ ダッシュボード常駐 | △ CLI 出力では不便 |
| マネージャー（週次レポートメール） | △ DVE はブラウザ必須 | ◯ API からレポート取得 | ◎ CI から自動生成 |
| CI エンジニア（PR ごとのレポート） | ✗ DVE はブラウザ必須 | △ サーバー起動が必要 | ◎ `npx dxe-server report` |

### C.4 ADR-002 判断フローチャート

```
Q1: DDE 可視化を DVE の一機能として吸収できるか?
    ├── Yes → Option A を検討（DxE-suite への PR が必要）
    └── No（DVE の UX モデルに合わない）→ Q2

Q2: 主なユースケースは「リアルタイムダッシュボード」か「非同期レポート」か?
    ├── リアルタイムダッシュボード → Option B（DDE 専用サーバー）
    └── 非同期レポート（CI / メール / Slack）→ Option C（レポートジェネレーター）

Q3（Option B / C 共通）: マルチプロジェクト横断集約は必要か?
    ├── Yes → API サーバー（Option B）が適切
    └── No（単一プロジェクトで十分）→ Option C または `dde status` CLI で代替可能
```

---

## 付録 D — 実装ロードマップ（ADR-002 判断後）

ADR-002 が解決した後の実装順序を示す。これは計画であり確定スケジュールではない。

### D.1 Option B 選択時のフェーズ計画

#### フェーズ 0 — 基盤（ADR-002 決定直後）

目標: `npx dxe-server start` が起動エラーを出さずに `"Hello, dxe-server"` を返すまで。

| タスク | 説明 | 完了条件 |
|---|---|---|
| `bin/dxe-server.js` 作成 | CLI エントリポイント。`--help`, `start`, `report` サブコマンドの骨格 | `node bin/dxe-server.js --help` が usage を表示する |
| `GET /api/health` 実装 | `{"status":"ok","version":"x.y.z"}` を返すだけ | `curl localhost:4175/api/health` → `200 OK` |
| 設定ファイルローダー | `~/.config/dxe-server/config.json` の読み書き | デフォルト値で初期化ファイルを生成できる |
| プロジェクトレジストリ | `projects.json` の CRUD | `POST /api/projects/register` → `projects.json` に書き込み |

#### フェーズ 1 — DDE ステータス API

目標: `GET /api/dde/status` が `completion.json` を読み取って返す。

前提: DDE-toolkit が `completion.json` を出力するよう更新済み（別チケット）。

| タスク | 説明 | 完了条件 |
|---|---|---|
| `completion.json` パーサー | DDE 出力ファイルを読んで内部オブジェクトに変換 | ユニットテスト通過 |
| `ddeStateResolver` | UNINITIALIZED / TRACKED / STALE の判定ロジック | ユニットテスト通過 |
| `GET /api/dde/status` | 全登録プロジェクトの補完状態 JSON | インテグレーションテスト通過 |
| `GET /api/dde/status/:id` | 特定プロジェクトの詳細 | インテグレーションテスト通過 |
| メモリキャッシュ | TTL: 5分。`completion.json` の毎回読み込みを回避 | ベンチマーク: < 200ms (キャッシュヒット時) |

#### フェーズ 2 — 週次レポート

目標: `GET /api/dde/weekly-report` が Markdown レポートを返す。

| タスク | 説明 | 完了条件 |
|---|---|---|
| `reportGenerator` | Markdown テンプレートエンジン（mustache or 手書き） | ユニットテスト通過 |
| `snapshotStore` | 過去レポートのスナップショット保存 | `~/.config/dxe-server/reports/` へのファイル書き込み |
| デルタ計算 | 先週との補完率差分 (`+8%` 等) の算出 | ユニットテスト通過（境界値テスト含む） |
| `GET /api/dde/weekly-report` | `Content-Type: text/markdown` レスポンス | インテグレーションテスト通過 |
| `GET /api/dde/weekly-report?format=html` | Markdown → HTML 変換 | インテグレーションテスト通過 |

#### フェーズ 3 — ブラウザ UI

目標: `npx dxe-server start` でブラウザが開き、補完率ダッシュボードが表示される。

| タスク | 説明 | 完了条件 |
|---|---|---|
| Vite + Preact セットアップ | `src/ui/` 以下に SPA 骨格 | `vite build` でビルドが通る |
| ダッシュボードコンポーネント | プロジェクト別補完率バー一覧 | 手動動作確認 |
| プロジェクト詳細ページ | ページ別補完状態、未リンク用語一覧 | 手動動作確認 |
| DVE へのリンク配置 | 「決定グラフは DVE で確認」のリンク | UI に DVE リンクが表示される |
| 静的ファイルサーブ | `GET /` で `dist/` の index.html を返す | ブラウザからアクセス可 |

#### フェーズ 4 — 時系列トレンド

目標: `GET /api/dde/trend` が 30日間の補完率推移を返す。

| タスク | 説明 | 完了条件 |
|---|---|---|
| `historyStore` | 日次スナップショットのファイル保存 | `{project}/dde/history/YYYY-MM-DD.json` が生成される |
| トレンド集計 API | `GET /api/dde/trend?project=id&days=30` | インテグレーションテスト通過 |
| UI: 折れ線グラフ | `/trend` ページに Chart.js 折れ線グラフ | 手動動作確認 |

### D.2 Option C 選択時のフェーズ計画

Option C はサーバーなしのため、実装がシンプルになる。

#### フェーズ 0 — CLI 基盤

| タスク | 説明 | 完了条件 |
|---|---|---|
| `bin/dxe-server.js` 作成 | `report`, `register`, `list` サブコマンドの骨格 | `node bin/dxe-server.js --help` が表示される |
| 設定ファイルローダー | `~/.config/dxe-server/config.json` の読み書き | 初期化ファイル生成 |
| `register` コマンド | `npx dxe-server register /path` → `projects.json` 更新 | 手動動作確認 |

#### フェーズ 1 — レポート生成

| タスク | 説明 | 完了条件 |
|---|---|---|
| `completion.json` パーサー | DDE 出力ファイルを読んで内部オブジェクトに変換 | ユニットテスト通過 |
| `reportGenerator` | Markdown テンプレート | ユニットテスト通過 |
| `report` コマンド | `npx dxe-server report` → stdout に Markdown | 手動動作確認 |
| `--format html` オプション | Markdown → HTML 変換 | 手動動作確認 |
| `--output FILE` オプション | ファイルへの書き込み | 手動動作確認 |

---

## 付録 E — `bin/dxe-server.js` 設計（Option B / C 共通スケルトン）

`bin/dxe-server.js` は現在存在しない。以下は実装開始時のスケルトン設計である。

### E.1 CLIサブコマンド体系

```
dxe-server [command] [options]

Commands:
  start          Start the local server (Option B only)
  report         Generate a DDE weekly report to stdout (Option C)
  register PATH  Register a project for tracking
  unregister ID  Remove a project from tracking
  list           List registered projects
  status         Show DDE completion status for all registered projects
  help           Show help

Options (global):
  --config PATH  Path to config file (default: ~/.config/dxe-server/config.json)
  --version      Show version
  --help         Show help

Options (start):
  --port NUMBER  Server port (default: 4175)
  --no-open      Do not open browser automatically

Options (report):
  --format       Output format: markdown | html (default: markdown)
  --output PATH  Write output to file instead of stdout
  --project ID   Limit report to a specific project
```

### E.2 エントリポイントの構造（擬似コード）

```javascript
#!/usr/bin/env node
// bin/dxe-server.js
// NOTE: このファイルは未作成。以下は設計スケルトンのみ。

import { parseArgs } from 'node:util';
import { loadConfig } from '../src/config/loader.js';
import { startServer } from '../src/server/index.js';   // Option B のみ
import { generateReport } from '../src/report/generator.js';
import { registerProject, listProjects } from '../src/registry/projects.js';

const { positionals, values } = parseArgs({
  allowPositionals: true,
  options: {
    port:    { type: 'string', default: '4175' },
    format:  { type: 'string', default: 'markdown' },
    output:  { type: 'string' },
    project: { type: 'string' },
    config:  { type: 'string' },
    version: { type: 'boolean' },
    help:    { type: 'boolean', short: 'h' },
  }
});

const [command, ...args] = positionals;

const config = await loadConfig(values.config);

switch (command) {
  case 'start':
    await startServer({ ...config.server, port: parseInt(values.port) });
    break;
  case 'report':
    const report = await generateReport({ config, projectId: values.project });
    if (values.output) {
      await writeFile(values.output, report);
    } else {
      process.stdout.write(report);
    }
    break;
  case 'register':
    await registerProject({ path: args[0], config });
    break;
  case 'list':
    const projects = await listProjects(config);
    console.table(projects);
    break;
  case 'status':
    // ... DDE ステータス表示
    break;
  default:
    console.log(USAGE_TEXT);
    process.exit(command ? 1 : 0);
}
```

### E.3 モジュール構成（計画）

```
bin/
  dxe-server.js         CLI エントリポイント（未作成）

src/
  config/
    loader.js           設定ファイル読み込み・デフォルト値適用
    schema.js           設定スキーマのバリデーション

  registry/
    projects.js         プロジェクト登録・解除・一覧
    validator.js        パスの正規化・存在確認

  dde/
    completionReader.js completion.json の読み込みとパース
    stateResolver.js    UNINITIALIZED / TRACKED / STALE 判定
    cache.js            メモリキャッシュ（TTL 管理）

  report/
    generator.js        Markdown / HTML レポート生成
    templates/
      weekly.md.js      週次レポートのテンプレート
    snapshot.js         スナップショット保存・読み込み
    delta.js            補完率差分計算

  server/               (Option B のみ)
    index.js            Node.js サーバー起動・ポートバインド
    router.js           ルート定義
    handlers/
      health.js
      dde.js
      projects.js

  ui/                   (Option B のみ)
    src/
      App.tsx
      components/
        CompletionBar.tsx
        ProjectCard.tsx
        TrendChart.tsx
    index.html
    vite.config.ts
```

---

## 付録 F — エラーコード一覧（計画）

### F.1 API エラーコード（Option B）

| エラーコード | HTTP ステータス | 意味 | 対処法 |
|---|---|---|---|
| `PROJECT_NOT_FOUND` | 404 | 指定 ID のプロジェクトが未登録 | `POST /api/projects/register` でまず登録する |
| `DDE_COMPLETION_NOT_AVAILABLE` | 503 | `completion.json` が存在しない | DDE を実行して `completion.json` を生成する |
| `DDE_COMPLETION_STALE` | 200 (警告付き) | `completion.json` が TTL を超過 | DDE を再実行して `completion.json` を更新する |
| `PROJECT_PATH_NOT_FOUND` | 422 | 登録パスがファイルシステムに存在しない | パスを確認して再登録する |
| `INVALID_PROJECT_PATH` | 400 | パストラバーサルまたは不正なパス | 絶対パスで再試行する |
| `CONFIG_LOAD_ERROR` | 500 | 設定ファイルが壊れている | `~/.config/dxe-server/config.json` を削除して再起動 |
| `REPORT_GENERATION_ERROR` | 500 | レポート生成中の予期しないエラー | ログを確認する |

### F.2 CLI エラーコード（Option C）

| 終了コード | 意味 |
|---|---|
| `0` | 正常終了 |
| `1` | 一般的なエラー（使用方法の誤り等） |
| `2` | プロジェクトパスが存在しない |
| `3` | `completion.json` が存在しない（DDE 未実行） |
| `4` | 設定ファイルの読み込みエラー |

---

## 付録 G — DDE との連携仕様詳細

dxe-server の動作は DDE-toolkit の出力に依存する。本付録では連携の詳細を定義する。

### G.1 DDE 側で必要な変更（要件）

現時点で DDE-toolkit は `completion.json` を出力しない。dxe-server を機能させるには DDE 側の変更が必要である。

| 変更項目 | DDE 側の対応 | dxe-server への影響 |
|---|---|---|
| `completion.json` 出力 | `dde generate` または `dde status --json` コマンドで生成 | **必須**（なければ dxe-server は動作不能） |
| フォーマット合意 | 付録 G.2 のスキーマを DDE-toolkit が遵守 | スキーマ変更時は両パッケージの更新が必要 |
| 出力パスの規約 | `{project}/dde/completion.json` に固定 | 設定可能にする（`dde.completionJsonPath`） |

### G.2 `completion.json` スキーマ詳細（提案）

```jsonc
{
  "$schema": "https://unlaxer.dev/schemas/dde-completion/v1.json",  // 将来的に
  "schemaVersion": 1,
  "generatedAt": "ISO 8601 datetime",    // 必須
  "ddeVersion": "semver string",         // 必須（バージョン確認用）
  "project": {
    "path": "absolute path string",      // 必須
    "name": "string"                     // 任意
  },
  "summary": {
    "completionRate": "0.0 - 1.0",       // 必須（totalPages > 0 の場合）
    "totalPages": "integer >= 0",        // 必須
    "completedPages": "integer >= 0",    // 必須
    "pendingPages": "integer >= 0"       // 必須（totalPages - completedPages）
  },
  "terms": {
    "total": "integer >= 0",             // 必須
    "linked": "integer >= 0",            // 必須
    "unlinked": "integer >= 0"           // 必須（total - linked）
  },
  "pages": [                             // 任意（詳細ビュー用）
    {
      "id": "string (relative path)",    // 必須
      "title": "string",                 // 任意
      "completionRate": "0.0 - 1.0",    // 必須
      "missingTerms": ["string"],        // 必須（空配列可）
      "lastModified": "ISO 8601"         // 任意
    }
  ]
}
```

バリデーションルール:
- `completedPages + pendingPages === totalPages`
- `linked + unlinked === total`
- `completionRate === completedPages / totalPages`（`totalPages === 0` の場合は `0.0`）

### G.3 バージョン互換性

| dxe-server バージョン | 対応 `completion.json` schemaVersion |
|---|---|
| `0.1.x` | `v1` |
| （将来） `0.2.x` | `v1`, `v2` |

スキーマバージョンが未対応の場合: `DDE_COMPLETION_SCHEMA_UNSUPPORTED` エラーとしてログ出力し、当該プロジェクトを `UNINITIALIZED` 扱いにする（クラッシュしない）。

---

## 付録 H — DVE との共存ガイドライン

dxe-server と DVE が同じ開発環境で並行して稼働するシナリオを想定した共存ガイドライン。

### H.1 推奨ワークフロー

```
1. DVE を起動（DGE/DRE/決定グラフ確認用）
   → npx dxe ui  （またはDxE-suiteのコマンド）
   → localhost:4174 でグラフビュー

2. dxe-server を起動（DDE 補完状態確認用）
   → npx dxe-server start
   → localhost:4175 でDDE補完ダッシュボード

3. 必要に応じて使い分ける
   - 「どの Gap が Open か」→ DVE
   - 「どのページが未完成か」→ dxe-server
   - 「DRE のインストール状態」→ DVE
   - 「今週 DDE 補完率は改善したか」→ dxe-server
```

### H.2 ユーザー向けの案内文（UI に表示予定）

```
dxe-server は DDE ドキュメント補完の可視化ツールです。

決定グラフ（Session → Gap → DD → Spec）の確認には
DVE (localhost:4174) をご利用ください。

DVE の起動: npx dxe ui
```

### H.3 データの二重管理を避けるルール

| データ種別 | 正規ソース | dxe-server の扱い |
|---|---|---|
| DRE インストール状態 | DVE `state-detector.ts` | 読み取らない（DVE に委譲） |
| Gap 一覧 | DGE `sessions/` ディレクトリ | 読み取らない（DVE に委譲） |
| DDE 補完状態 | DDE `completion.json` | **dxe-server が正規の可視化担当** |
| プロジェクト一覧 | DVE 内部 / dxe-server `projects.json` | **二重管理**（Gap #2 が解決するまで許容） |

プロジェクト一覧の二重管理は Gap #2 解決後に統合を検討する。DVE の `/api/scan` 結果を dxe-server が参照する方式（HTTP 経由）が有力候補。

---

## 付録 I — 既知の制約と将来計画

### I.1 v0.1.x の既知制約

| 制約 | 理由 | 将来の解決策 |
|---|---|---|
| `bin/dxe-server.js` が存在しない | ADR-002 判断待ちで実装未着手 | ADR-002 決定後に実装（フェーズ 0） |
| DDE の `completion.json` が存在しない | DDE-toolkit 側の変更が未実施 | DDE-toolkit への PR が必要 |
| CI なし | scaffold のみ | ADR-002 決定後に設定 |
| テストなし | scaffold のみ | フェーズ 0 と並行して設定 |
| npm 未公開 | 実装コードがないため | フェーズ 0 完了後に公開検討 |

### I.2 将来機能候補（バックログ）

以下は v0.1.x のスコープ外。将来バージョンでの検討候補:

| 機能 | 優先度 | 前提条件 |
|---|---|---|
| Slack Webhook 統合（週次レポート自動送信） | 低 | フェーズ 2 完了後 |
| メール送信（週次レポート） | 低 | フェーズ 2 完了後 |
| GitHub Actions アーティファクトとしてレポート添付 | 中 | Option C フェーズ 1 完了後 |
| DVE の `/api/scan` 結果からプロジェクトを自動インポート | 中 | Gap #2 解決後 |
| DDE completion.json の変化をウォッチ（inotify） | 低 | フェーズ 1 完了後（常駐デーモンのみ） |
| 認証（Basic Auth / Token）| 低 | チーム共有サーバー用途が出てきた場合 |
| Docker イメージ | 低 | チーム共有サーバー用途が出てきた場合 |

### I.3 廃止予定機能

現時点では廃止予定機能なし（scaffold のため）。

ADR-002 で Option A が選択された場合、リポジトリ全体が archive 対象となる。

---

## 付録 J — バージョン戦略

### J.1 セマンティックバージョニング適用方針

| バージョン変更 | 条件 |
|---|---|
| `MAJOR` (`1.0.0`) | API の後方互換性を破壊する変更（エンドポイントの削除・レスポンス構造の変更） |
| `MINOR` (`0.x.0`) | 後方互換性のある機能追加（新エンドポイント、新 CLI サブコマンド） |
| `PATCH` (`0.0.x`) | バグ修正、ドキュメント更新、内部リファクタリング |

### J.2 `0.x.x` 期間中の互換性保証

`v1.0.0` に達するまでの `0.x.x` 期間は**後方互換性を保証しない**。

- `completion.json` スキーマは `v1.0.0` まで変更される可能性がある
- API レスポンス構造は `v1.0.0` まで変更される可能性がある
- CLI フラグ名は `v1.0.0` まで変更される可能性がある

### J.3 CHANGELOG 管理

```
CHANGELOG.md を Keep a Changelog 形式で維持する。
ADR-002 決定・フェーズ完了・スキーマ変更を必ず記録する。
```

---

## 付録 K — セキュリティ考慮事項詳細

### K.1 脅威モデル（ローカルツールとして）

dxe-server v0.1.x は `localhost` にバインドされたローカルツールである。ネットワーク越しのアクセスは想定しない。

| 脅威 | リスク | 対策 |
|---|---|---|
| パストラバーサル | プロジェクト登録パスに `../../etc/passwd` 等が指定される | `path.resolve()` + 登録済みパスのプレフィックス検証 |
| 任意ファイル読み込み | `GET /api/dde/status` がプロジェクト外のファイルを読む | 登録済みプロジェクトパス内のファイルのみ読み取り許可 |
| JSON インジェクション | `completion.json` に悪意あるデータ | `JSON.parse()` の例外捕捉、スキーマバリデーション |
| ポート衝突 | DVE が 4174 を使用中に dxe-server が 4175 でバインドできない | 起動時に `EADDRINUSE` を適切にエラーメッセージとして表示 |

### K.2 `localhost` バインド以外の用途について

チームサーバー（LAN 内共有）用途は v0.1.x のスコープ外。将来サポートする場合は認証機能が必須。

---

## 付録 L — プロジェクト名と識別子の整理

dxe-server / DxE シリーズには似た名前の識別子が多い。混乱を防ぐため整理する。

| 識別子 | 種別 | 説明 |
|---|---|---|
| `dxe-server` | リポジトリ名 / npm パッケージ名の一部 | 本パッケージ |
| `@unlaxer/dxe-server` | npm パッケージ名（完全形） | 本パッケージの公開名 |
| `DxE-suite` | 製品名 | DGE + DDE + DRE + DVE のバンドル |
| `DVE` | コンポーネント名 | Decision Visualization Engine（DxE-suite 内） |
| `DDE` | コンポーネント名 | Document Debugging Engine（DxE-suite 内） |
| `DGE` | コンポーネント名 | Design Gap Extraction（DxE-suite 内） |
| `DRE` | コンポーネント名 | DxE Rules Engine（DxE-suite 内） |
| `dxe-server` | CLI コマンド名 | `npx dxe-server start` で使用するコマンド |
| `npx dxe ui` | 廃止候補 | DxE-suite サブコマンドとして検討されたが ADR-001 で却下 |

---

## 付録 M — 設計上の未決定事項（Open Questions）

ADR-002 以外に本 SPEC 作成時点で未決定の事項を列挙する。ADR-002 判断後に順次解決する。

### M.1 `completion.json` のトリガー方式

| 問い | 選択肢 | 現状 |
|---|---|---|
| `completion.json` はいつ生成されるか | (a) `dde generate` 実行時に自動生成 / (b) `dde status --json` のオンデマンド / (c) DDE のすべてのコマンドで更新 | 未決定（DDE-toolkit の設計に依存） |
| dxe-server は DDE の実行をトリガーできるか | (a) できない（読み取り専用） / (b) `dde status --json` を子プロセスとして実行 | 読み取り専用を原則とする（v0.1.x） |

### M.2 マルチプロジェクト集約のデータモデル

| 問い | 選択肢 | 現状 |
|---|---|---|
| プロジェクト間の補完率を集計するか | (a) 単純平均 / (b) ページ数加重平均 / (c) 集計しない（個別表示のみ） | 未決定 |
| プロジェクトのグループ化 | (a) タグ付け / (b) ディレクトリ構造に従う / (c) グループ化なし | 未決定 |

### M.3 スナップショット保存の場所

| 問い | 選択肢 | 現状 |
|---|---|---|
| 過去のスナップショットはどこに保存するか | (a) `~/.config/dxe-server/snapshots/` / (b) プロジェクトディレクトリ内 `dde/history/` / (c) 保存しない（週次レポート時に生成のみ） | 未決定 |
| スナップショットの保持期間 | (a) 30日 / (b) 90日 / (c) 無限 | 未決定 |

### M.4 週次レポートのスケジューリング

| 問い | 選択肢 | 現状 |
|---|---|---|
| レポートのスケジュール方法 | (a) OS の cron を使用（設定案内のみ） / (b) dxe-server 内蔵スケジューラー（Option B 常駐のみ） / (c) スケジュールなし（手動実行のみ） | 未決定 |
| 「週次」の基準日 | (a) 月曜 / (b) 設定ファイルで指定 / (c) 実行日から7日前 | 未決定 |

### M.5 ブラウザ自動起動の挙動

| 問い | 選択肢 | 現状 |
|---|---|---|
| `npx dxe-server start` でブラウザを開くか | (a) デフォルトで開く（`--no-open` で抑制）/ (b) デフォルトで開かない（`--open` で有効化） | 未決定 |
| CI 環境での挙動 | (a) `CI=true` 環境変数を検出して自動抑制 / (b) 明示的なフラグのみ | `CI=true` 検出を原則とする（計画） |

---

## 付録 N — 類似ツールとの位置付け

dxe-server の設計判断の文脈を明確にするため、類似ツールとの比較を示す。

### N.1 内部（DxE シリーズ）との比較

| ツール | 起動形態 | UI | 主なデータソース | dxe-server との差異 |
|---|---|---|---|---|
| DVE | 常駐サーバー (`localhost:4174`) | Preact + Cytoscape ブラウザ UI | DGE `sessions/`、DRE `.claude/` | DDE を扱わない。dxe-server は DDE 専用 |
| DDE-toolkit | CLI（ワンショット） | なし（テキスト出力） | `docs/` ディレクトリ | dxe-server は DDE の出力を可視化する上位レイヤー |
| DRE-toolkit | CLI（ワンショット） | なし（テキスト出力） | `.claude/` ディレクトリ | dxe-server は DRE データを扱わない（DVE に委譲） |
| DxE-suite | CLI（インストール管理） | なし | npm registry | dxe-server はDxE-suite に依存しない独立パッケージ |

### N.2 外部ツールとの比較

| ツール | 類似点 | dxe-server との差異 |
|---|---|---|
| Storybook | コンポーネントの状態を可視化するローカルサーバー | Storybook は UI コンポーネント専用。dxe-server は DDE ドキュメント補完専用 |
| Docusaurus | ドキュメントサイトの生成 | 静的サイト生成。dxe-server は「補完率」という動的メトリクスに特化 |
| Grafana | メトリクスの時系列ダッシュボード | 汎用。dxe-server は DDE 専用の軽量ツール |
| GitHub Insights | リポジトリのコントリビューション可視化 | GitHub 専用。dxe-server はローカルファイルベース |

dxe-server の最大の特徴は「DDE ドキュメント補完率という特定メトリクスに特化し、`npx` でゼロインストール起動できること」である。

---

## 付録 O — SPEC 更新プロセス

本 SPEC は生きたドキュメントである。更新のガイドラインを定める。

### O.1 更新トリガー

| イベント | 更新箇所 | 担当 |
|---|---|---|
| ADR-002 決定 | 全セクションの「計画」表記を更新。選択されなかった Option の詳細を削除または折りたたみ | TBD |
| DDE-toolkit に `completion.json` 出力が追加 | §3.2、§付録 G を「実装済み」に更新 | TBD |
| フェーズ 0 完了 | §12.1 の scaffold 表記を削除、CHANGELOG.md 更新 | TBD |
| API エンドポイント変更 | §6 全体を更新 | TBD |
| `completion.json` スキーマ変更 | §3.2、付録 G.2 を更新 | TBD |

### O.2 SPEC バージョンとパッケージバージョンの対応

| SPEC バージョン | 対応パッケージバージョン | 状態 |
|---|---|---|
| `0.1.0-spec` | `0.1.0` | 現在（scaffold / 設計中） |
| `0.2.0-spec` | `0.2.0`（予定） | ADR-002 決定後・フェーズ 0 完了時 |
| `1.0.0-spec` | `1.0.0`（予定） | フェーズ 2 完了・安定版 |

### O.3 この SPEC のメタデータ

| 項目 | 値 |
|---|---|
| ファイルパス | `spec/SPEC.md` |
| 作成日 | 2026-04-19 |
| 最終更新 | 2026-04-19 |
| 行数目安 | 1500–2500 行 |
| 関連 ADR | ADR-001（確定）、ADR-002（OPEN） |
| 次のレビュートリガー | ADR-002 の判断が出た時点 |

---

## 付録 P — 受け入れ基準チェックリスト（フェーズ 0 完了判定）

ADR-002 決定後、フェーズ 0 の実装が完了したと見なすための受け入れ基準。

### P.1 `bin/dxe-server.js` 基本動作

- [ ] `node bin/dxe-server.js --help` を実行すると usage が stdout に出力される
- [ ] `node bin/dxe-server.js --version` を実行すると `@unlaxer/dxe-server@0.2.0` 相当が表示される
- [ ] `node bin/dxe-server.js start` を実行するとポート 4175 でサーバーが起動する
- [ ] `Ctrl+C` または `SIGTERM` でサーバーが graceful shutdown する
- [ ] `curl http://localhost:4175/api/health` が `{"status":"ok"}` を返す
- [ ] `DXE_SERVER_PORT=4200 node bin/dxe-server.js start` で指定ポートで起動する

### P.2 設定ファイル

- [ ] `~/.config/dxe-server/` ディレクトリが自動作成される
- [ ] `~/.config/dxe-server/config.json` がデフォルト値で初期化される
- [ ] 既存の `config.json` があれば上書きしない

### P.3 プロジェクト登録

- [ ] `POST /api/projects/register` で正しいパスを指定するとプロジェクトが登録される
- [ ] `GET /api/projects` で登録済みプロジェクト一覧が返る
- [ ] `DELETE /api/projects/:id` で登録が解除される
- [ ] 存在しないパスを登録しようとすると `PROJECT_PATH_NOT_FOUND` エラーが返る
- [ ] パストラバーサルが含まれるパスは `INVALID_PROJECT_PATH` として拒否される

### P.4 テスト

- [ ] ユニットテストが存在し `npm test` で通過する
- [ ] カバレッジが主要ビジネスロジックで 80% 以上
- [ ] `npm run test:smoke` でスモークテストが通過する

### P.5 npm 公開準備

- [ ] `npm pack` でパッケージが正しく生成される（`bin/` と必要ファイルが含まれる）
- [ ] `package.json` に `engines.node: ">=18"` が設定されている
- [ ] `README.md` にインストール方法と使い方が記載されている
- [ ] `CHANGELOG.md` が更新されている
