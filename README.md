# mcp-server-ga4

Google Analytics 4 (GA4) の MCP サーバー実装です。  
[ruchernchong/mcp-server-google-analytics](https://github.com/ruchernchong/mcp-server-google-analytics)（アーカイブ済み）をフォークし、**Application Default Credentials (ADC)** に対応しました。

> **元リポジトリとの違い**: サービスアカウントの秘密鍵（`GOOGLE_CLIENT_EMAIL` / `GOOGLE_PRIVATE_KEY`）が必須ではなくなりました。ADC が利用できる環境（Google Cloud / Firebase App Hosting など）では `GA_PROPERTY_ID` のみで動作します。

## 機能

- ページビュー指標の取得（カスタムディメンション対応）
- アクティブユーザー・新規ユーザー数の推移取得
- イベント指標の取得（イベント名フィルタ対応）
- ユーザー行動指標の取得（セッション時間、直帰率など）
- 柔軟な日付範囲指定
- 汎用レポート実行（カスタムディメンション・メトリクス・フィルタ）

## 認証方式

本サーバーは 3 つの認証方式をサポートしています。環境に合わせて選択してください。

### 方式 1: Application Default Credentials（ADC）— Google Cloud 環境向け

Cloud Run、App Hosting、GCE など Google Cloud のマネージド環境では、`GA_PROPERTY_ID` だけで動作します。サービスアカウントの IAM 認証が自動的に使われます。

```bash
export GA_PROPERTY_ID="123456789"
```

### 方式 2: `GOOGLE_APPLICATION_CREDENTIALS` — ローカル・非 Google Cloud 環境向け（推奨）

Google Cloud 外の環境（ローカル PC、AWS、Azure など）で最も手軽な方式です。  
`gcloud auth application-default login` で取得したクレデンシャルファイル、またはサービスアカウントの JSON キーファイルのパスを環境変数で指定します。

**ローカル開発（ユーザー認証）の場合:**

```bash
# 初回のみ: ブラウザで認証し、クレデンシャルファイルを生成
gcloud auth application-default login

# 生成されたファイルのパスを指定
export GOOGLE_APPLICATION_CREDENTIALS="$HOME/.config/gcloud/application_default_credentials.json"
export GA_PROPERTY_ID="123456789"
```

**サービスアカウント JSON キーファイルの場合:**

```bash
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/service-account-key.json"
export GA_PROPERTY_ID="123456789"
```

### 方式 3: サービスアカウント秘密鍵（環境変数直接指定）

JSON キーファイルを配置できない環境では、`client_email` と `private_key` を環境変数で直接指定できます。

```bash
export GOOGLE_CLIENT_EMAIL="your-sa@project.iam.gserviceaccount.com"
export GOOGLE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
export GA_PROPERTY_ID="123456789"
```

### 認証の優先順位

サーバーは以下の順序で認証情報を解決します:

1. `GOOGLE_CLIENT_EMAIL` + `GOOGLE_PRIVATE_KEY` が両方設定されている → サービスアカウント認証（方式 3）
2. 上記が未設定 → Google Cloud クライアントライブラリの標準フォールバック:
   - `GOOGLE_APPLICATION_CREDENTIALS` 環境変数が指すファイル（方式 2）
   - Google Cloud メタデータサーバー（方式 1）

## 前提条件

- Node.js 20 以上
- Google Analytics 4 プロパティ
- Google Cloud プロジェクトで **Analytics Data API** が有効化されていること
- 以下のいずれかの認証:
  - Google Cloud マネージド環境の ADC（Cloud Run、App Hosting など）
  - `GOOGLE_APPLICATION_CREDENTIALS` 環境変数（`gcloud auth application-default login` のクレデンシャルまたはサービスアカウント JSON キー）
  - `GOOGLE_CLIENT_EMAIL` + `GOOGLE_PRIVATE_KEY` 環境変数

## セットアップ

### 1. Google Analytics Data API の有効化

1. [Google Cloud Console](https://console.cloud.google.com/) を開く
2. プロジェクトを選択
3. **API とサービス > ライブラリ** から **「Google Analytics Data API」** を検索して有効化

### 2. アクセス権の設定

ADC を使う場合は、ADC のアカウント（またはサービスアカウント）に GA4 プロパティの **「閲覧者」** ロールを付与してください。

1. [Google Analytics](https://analytics.google.com/) を開く
2. **管理 > プロパティのアクセス管理** に移動
3. **「+」** をクリックし、アカウントのメールアドレスを追加
4. **「閲覧者」** ロールを割り当て

## インストール

### npx で直接実行（推奨）

```bash
npx -y mcp-server-ga4
```

### グローバルインストール

```bash
npm install -g mcp-server-ga4
```

## MCP クライアント設定例

### Google Cloud 環境（ADC 自動認証）

```json
{
  "mcpServers": {
    "ga4": {
      "command": "npx",
      "args": ["-y", "mcp-server-ga4"],
      "env": {
        "GA_PROPERTY_ID": "123456789"
      }
    }
  }
}
```

### ローカル・非 Google Cloud 環境（`GOOGLE_APPLICATION_CREDENTIALS` 使用）— おすすめ

```json
{
  "mcpServers": {
    "ga4": {
      "command": "npx",
      "args": ["-y", "mcp-server-ga4"],
      "env": {
        "GA_PROPERTY_ID": "123456789",
        "GOOGLE_APPLICATION_CREDENTIALS": "/path/to/application_default_credentials.json"
      }
    }
  }
}
```

> **ヒント**: `gcloud auth application-default login` を実行すると、通常 `~/.config/gcloud/application_default_credentials.json` にクレデンシャルファイルが生成されます。

### サービスアカウント秘密鍵（環境変数直接指定）

```json
{
  "mcpServers": {
    "ga4": {
      "command": "npx",
      "args": ["-y", "mcp-server-ga4"],
      "env": {
        "GOOGLE_CLIENT_EMAIL": "your-sa@project.iam.gserviceaccount.com",
        "GOOGLE_PRIVATE_KEY": "-----BEGIN PRIVATE KEY-----\nYOUR_KEY\n-----END PRIVATE KEY-----",
        "GA_PROPERTY_ID": "123456789"
      }
    }
  }
}
```

## 利用可能なツール

### runReport

汎用レポートを実行します。ディメンション・メトリクス・フィルタを自由に指定できます。

```json
{
  "startDate": "2024-01-01",
  "endDate": "2024-01-31",
  "dimensions": [{ "name": "country" }, { "name": "city" }],
  "metrics": [{ "name": "activeUsers" }, { "name": "newUsers" }],
  "dimensionFilter": {
    "filter": {
      "fieldName": "country",
      "stringFilter": { "value": "Japan" }
    }
  }
}
```

### getPageViews

ページビュー指標を取得します。

```json
{
  "startDate": "2024-01-01",
  "endDate": "2024-01-31",
  "dimensions": ["page", "country"]
}
```

### getActiveUsers

アクティブユーザー・新規ユーザー数を取得します。

```json
{
  "startDate": "2024-01-01",
  "endDate": "2024-01-31"
}
```

### getEvents

イベント指標を取得します。`eventName` でフィルタ可能です。

```json
{
  "startDate": "2024-01-01",
  "endDate": "2024-01-31",
  "eventName": "purchase"
}
```

### getUserBehavior

ユーザー行動指標（セッション時間、直帰率、ユーザーあたりセッション数）を取得します。

```json
{
  "startDate": "2024-01-01",
  "endDate": "2024-01-31"
}
```

## セキュリティに関する注意事項

- **最小権限の原則**: GA4 では「閲覧者」ロールのみを付与してください
- **鍵の管理**: サービスアカウントの秘密鍵は安全に保管し、クライアントサイドに露出させないでください
- **環境変数**: 機密情報は環境変数で管理してください

## 開発

```bash
pnpm install
pnpm build     # TypeScript コンパイル
pnpm dev       # 開発モードで起動
pnpm test      # テスト実行
pnpm lint      # リント
pnpm format    # フォーマット
```

## 元リポジトリ

このプロジェクトは [ruchernchong/mcp-server-google-analytics](https://github.com/ruchernchong/mcp-server-google-analytics) のフォークです。  
元リポジトリは 2025年10月5日 にアーカイブされました。

## ライセンス

MIT License - [LICENSE](LICENSE) を参照してください。
