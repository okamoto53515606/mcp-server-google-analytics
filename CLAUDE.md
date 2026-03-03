# CLAUDE.md

このファイルは Claude Code (claude.ai/code) がこのリポジトリで作業する際のガイダンスを提供します。

## 基本コマンド

### ビルド・開発
- `pnpm build` - TypeScript を JavaScript にコンパイル（出力先: `dist/`）
- `pnpm dev` - ts-node で開発モード起動
- `pnpm start` - コンパイル済みの `dist/index.js` を起動

### テスト
- `pnpm test` - `tests/` ディレクトリの Vitest テストを実行
- Vitest で構成、Node.js 環境をターゲット

### コード品質
- `pnpm lint` - `src/` ディレクトリに Biome リンターを実行
- `pnpm format` - Biome でコードフォーマット（書き込みあり）
- `pnpm check` - Biome チェックを実行し自動修正

## プロジェクトアーキテクチャ

### コア構造
Google Analytics 4 データにアクセスする Model Context Protocol (MCP) サーバーです。

**単一エントリポイント**: `src/index.ts` にサーバー実装全体を含む:
- `@modelcontextprotocol/sdk` — MCP サーバー機能
- `@google-analytics/data` — GA4 API アクセス

**認証方式**: 2 つの認証方式をサポート:
- **ADC (Application Default Credentials)** — `GA_PROPERTY_ID` のみで動作（推奨）
- **サービスアカウント秘密鍵** — `GOOGLE_CLIENT_EMAIL` + `GOOGLE_PRIVATE_KEY` + `GA_PROPERTY_ID`

**サーバーパターン**:
- `StdioServerTransport` で通信
- `ListToolsRequestSchema` と `CallToolRequestSchema` のリクエストハンドラ
- 起動時の環境変数バリデーション

### ツール実装
MCP プロトコル経由で 5 つのツールを公開:

1. **runReport** — カスタムディメンション・メトリクスによる汎用レポート
2. **getPageViews** — ページビュー指標（オプションのディメンション付き）
3. **getActiveUsers** — アクティブユーザー・新規ユーザー数の推移
4. **getEvents** — イベント追跡（イベント名フィルタ対応）
5. **getUserBehavior** — セッション時間、直帰率、ユーザーあたりセッション数

各ツールのパターン:
- 日付範囲バリデーション（YYYY-MM-DD 形式）
- `analyticsDataClient.runReport()` による GA API 呼び出し
- JSON レスポンスフォーマット
- `McpError` による包括的エラーハンドリング

### 環境変数
- `GA_PROPERTY_ID`（必須） — Google Analytics 4 プロパティ ID
- `GOOGLE_APPLICATION_CREDENTIALS`（任意） — ADC クレデンシャルファイルまたはサービスアカウント JSON キーファイルのパス
- `GOOGLE_CLIENT_EMAIL`（任意） — サービスアカウントメール
- `GOOGLE_PRIVATE_KEY`（任意） — サービスアカウント秘密鍵

### エラーハンドリング
- 起動時に環境変数を検証
- 全ツールで日付形式・範囲をバリデーション
- Google Analytics API エラーは `McpError` でラップ
- プロセスレベルの未キャッチ例外・未処理 Promise 拒否ハンドラ

## 開発メモ

### TypeScript 設定
- strict モード有効（包括的な型チェック）
- ES2020 ターゲット、CommonJS モジュール
- ソースマップと宣言ファイルを生成
- テストはコンパイル対象外

### コードスタイル
- Biome によるリント・フォーマット
- ダブルクォート
- スペースインデント
- インポート整理有効

### テスト設定
- Vitest + TypeScript サポート
- `tests/` ディレクトリにテストを配置
- `src/` ファイルのカバレッジレポート（v8 プロバイダ）
- Node.js テスト環境