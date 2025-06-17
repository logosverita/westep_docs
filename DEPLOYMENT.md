# WeStep デプロイメントガイド

**WeStep** を本番環境にデプロイするための手順書です。

## 🚀 デプロイメント概要

### 推奨構成

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Vercel        │    │   Supabase      │    │   Domain        │
│   (Frontend)    │◄──►│   (Backend/DB)  │◄──►│   (Custom)      │
│                 │    │                 │    │                 │
│ - Next.js App   │    │ - PostgreSQL    │    │ - westep.app    │
│ - Edge Functions│    │ - Auth          │    │ - SSL/TLS       │
│ - Global CDN    │    │ - Real-time     │    │ - DNS管理       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## 📋 デプロイ前チェックリスト

### 🔧 技術的準備

-   [ ] **コードレビュー完了**
-   [ ] **全てのテストが通過**
-   [ ] **型エラーが解消済み**
-   [ ] **ESLint エラーが解消済み**
-   [ ] **本番用環境変数の準備完了**
-   [ ] **データベースマイグレーション準備完了**

### 🌐 インフラ準備

-   [ ] **Supabase プロジェクト作成済み**
-   [ ] **Vercel アカウント準備済み**
-   [ ] **ドメイン取得済み** (westep.app 推奨)
-   [ ] **SSL 証明書設定準備**
-   [ ] **監視・ログツール設定**

## 🗄️ Step 1: Supabase セットアップ

### 1.1 プロジェクト作成

```bash
# Supabase CLI のインストール
npm install -g supabase

# Supabase にログイン
supabase login

# 新しいプロジェクトを作成
supabase projects create westep-production --org-id your-org-id --db-password your-secure-password --region ap-northeast-1
```

### 1.2 データベースセットアップ

```bash
# Prismaスキーマを本番データベースに適用
npx prisma db push --preview-feature

# 初期データの投入（必要に応じて）
npx prisma db seed
```

### 1.3 認証設定

Supabase ダッシュボードで以下を設定：

1. **Authentication > Settings**

    - Site URL: `https://westep.app`
    - Redirect URLs: `https://westep.app/auth/callback`

2. **Authentication > Providers**
    - Email 認証を有効化
    - 必要に応じて Google/GitHub 認証も設定

### 1.4 リアルタイム機能有効化

```sql
-- Supabase SQL Editor で実行
-- daily_goals テーブルのリアルタイム機能を有効化
ALTER PUBLICATION supabase_realtime ADD TABLE daily_goals;
ALTER PUBLICATION supabase_realtime ADD TABLE achievements;
```

### 1.5 Row Level Security (RLS) 設定

```sql
-- ユーザーは自分のデータのみアクセス可能
ALTER TABLE daily_goals ENABLE ROW LEVEL SECURITY;
ALTER TABLE achievements ENABLE ROW LEVEL SECURITY;

-- ポリシー作成
CREATE POLICY "Users can view their own goals" ON daily_goals
  FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "Users can insert their own goals" ON daily_goals
  FOR INSERT WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update their own goals" ON daily_goals
  FOR UPDATE USING (auth.uid() = user_id);
```

## 🌐 Step 2: Vercel デプロイメント

### 2.1 Vercel プロジェクト作成

```bash
# Vercel CLI のインストール
npm install -g vercel

# Vercelにログイン
vercel login

# プロジェクトをデプロイ
vercel --prod
```

### 2.2 環境変数設定

Vercel ダッシュボードで以下の環境変数を設定：

```bash
# 必須環境変数
DATABASE_URL=postgresql://postgres.xxx:password@xxx.supabase.co:5432/postgres
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_supabase_service_role_key
NEXTAUTH_SECRET=your-production-secret
NEXTAUTH_URL=https://westep.app

# アプリケーション設定
NEXT_PUBLIC_APP_NAME=WeStep
NEXT_PUBLIC_APP_URL=https://westep.app
NODE_ENV=production

# WebSocket設定
NEXT_PUBLIC_WS_URL=wss://westep.app

# 監視設定
NEXT_PUBLIC_SENTRY_DSN=https://xxx@xxx.ingest.sentry.io/xxx
NEXT_PUBLIC_GA_MEASUREMENT_ID=G-XXXXXXXXXX
```

### 2.3 ビルド設定

`vercel.json` を作成：

```json
{
	"buildCommand": "npm run build",
	"devCommand": "npm run dev",
	"framework": "nextjs",
	"regions": ["nrt1"],
	"functions": {
		"app/api/websocket/route.ts": {
			"maxDuration": 300
		}
	},
	"headers": [
		{
			"source": "/(.*)",
			"headers": [
				{
					"key": "X-Content-Type-Options",
					"value": "nosniff"
				},
				{
					"key": "X-Frame-Options",
					"value": "DENY"
				},
				{
					"key": "X-XSS-Protection",
					"value": "1; mode=block"
				}
			]
		}
	]
}
```

## 🌍 Step 3: ドメイン設定

### 3.1 DNS 設定

ドメインレジストラーで以下のレコードを設定：

```
Type: CNAME
Name: @
Value: cname.vercel-dns.com

Type: CNAME
Name: www
Value: cname.vercel-dns.com
```

### 3.2 Vercel でのドメイン設定

```bash
# Vercel CLIでドメインを追加
vercel domains add westep.app
vercel domains add www.westep.app

# SSL証明書の自動設定を確認
vercel certs ls
```

## 📊 Step 4: 監視・ログ設定

### 4.1 Sentry（エラートラッキング）

```bash
# Sentry CLI のインストール
npm install @sentry/nextjs

# Sentry設定ファイル作成
npx @sentry/wizard -i nextjs
```

`sentry.client.config.ts`:

```typescript
import * as Sentry from "@sentry/nextjs";

Sentry.init({
	dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
	environment: process.env.NODE_ENV,
	tracesSampleRate: 1.0,
});
```

### 4.2 Vercel Analytics

```bash
# Vercel Analytics の有効化
npm install @vercel/analytics

# _app.tsx に追加
import { Analytics } from '@vercel/analytics/react';

export default function App({ Component, pageProps }) {
  return (
    <>
      <Component {...pageProps} />
      <Analytics />
    </>
  );
}
```

### 4.3 ヘルスチェック

`app/api/health/route.ts` を作成：

```typescript
import { NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";

export async function GET() {
	try {
		// データベース接続チェック
		await prisma.$queryRaw`SELECT 1`;

		return NextResponse.json({
			status: "healthy",
			timestamp: new Date().toISOString(),
			version: process.env.npm_package_version,
		});
	} catch (error) {
		return NextResponse.json(
			{ status: "unhealthy", error: error.message },
			{ status: 500 }
		);
	}
}
```

## 🚦 Step 5: CI/CD パイプライン

### 5.1 GitHub Actions

`.github/workflows/deploy.yml`:

```yaml
name: Deploy to Production

on:
    push:
        branches: [main]
    pull_request:
        branches: [main]

jobs:
    test:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            - uses: actions/setup-node@v4
              with:
                  node-version: "18"
                  cache: "npm"

            - run: npm ci
            - run: npm run lint
            - run: npm run type-check
            - run: npm run test
            - run: npm run build

    deploy:
        needs: test
        runs-on: ubuntu-latest
        if: github.ref == 'refs/heads/main'
        steps:
            - uses: actions/checkout@v4
            - uses: amondnet/vercel-action@v25
              with:
                  vercel-token: ${{ secrets.VERCEL_TOKEN }}
                  vercel-org-id: ${{ secrets.ORG_ID }}
                  vercel-project-id: ${{ secrets.PROJECT_ID }}
                  vercel-args: "--prod"
```

### 5.2 ブランチ保護

GitHub で以下のブランチ保護を設定：

-   `main` ブランチの直接プッシュを禁止
-   プルリクエストレビューを必須に
-   ステータスチェック必須（テスト・ビルド成功）

## 🔒 Step 6: セキュリティ設定

### 6.1 セキュリティヘッダー

`next.config.js` に追加：

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
	experimental: {
		appDir: true,
	},
	async headers() {
		return [
			{
				source: "/(.*)",
				headers: [
					{
						key: "Strict-Transport-Security",
						value: "max-age=31536000; includeSubDomains",
					},
					{
						key: "Content-Security-Policy",
						value: "default-src 'self'; script-src 'self' 'unsafe-eval' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; connect-src 'self' https://*.supabase.co wss://*.supabase.co;",
					},
				],
			},
		];
	},
};

module.exports = nextConfig;
```

### 6.2 環境変数のローテーション

本番環境では定期的に以下をローテーション：

-   `NEXTAUTH_SECRET`
-   `SUPABASE_SERVICE_ROLE_KEY`
-   データベースパスワード

## 📈 Step 7: パフォーマンス最適化

### 7.1 Next.js 最適化

```javascript
// next.config.js
module.exports = {
	images: {
		domains: ["westep.app"],
		formats: ["image/webp", "image/avif"],
	},
	compress: true,
	poweredByHeader: false,
	generateEtags: true,
	experimental: {
		optimizeCss: true,
		scrollRestoration: true,
	},
};
```

### 7.2 データベース最適化

```sql
-- インデックス最適化
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_daily_goals_user_date
ON daily_goals(user_id, date DESC);

CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_achievements_user_updated
ON achievements(user_id, updated_at DESC);

-- 統計情報更新
ANALYZE daily_goals;
ANALYZE achievements;
```

## 🔄 Step 8: デプロイ後検証

### 8.1 機能テスト

```bash
# ヘルスチェック
curl https://westep.app/api/health

# 主要機能の動作確認
- ユーザー登録・ログイン
- 目標作成・完了
- リアルタイム機能
- レスポンシブデザイン
```

### 8.2 パフォーマンステスト

```bash
# Lighthouse CLI でのテスト
npm install -g lighthouse
lighthouse https://westep.app --output html --output-path ./lighthouse-report.html

# Core Web Vitals の確認
# - LCP (Largest Contentful Paint): < 2.5s
# - FID (First Input Delay): < 100ms
# - CLS (Cumulative Layout Shift): < 0.1
```

## 🚨 トラブルシューティング

### よくある問題と解決方法

#### 1. ビルドエラー

```bash
# 依存関係の問題
rm -rf node_modules package-lock.json
npm install

# 型エラー
npm run type-check
```

#### 2. データベース接続エラー

```bash
# 接続文字列の確認
npx prisma db pull

# SSL設定の確認
DATABASE_URL="postgresql://...?sslmode=require"
```

#### 3. 認証エラー

```bash
# Supabase認証設定の確認
- Site URL が正しく設定されているか
- Redirect URLs が本番URLに設定されているか
```

## 📚 運用・保守

### 定期メンテナンス

-   **毎週**: 依存関係の更新確認
-   **毎月**: セキュリティアップデート適用
-   **四半期**: パフォーマンス監査・最適化
-   **半年**: 全体的なアーキテクチャレビュー

### バックアップ戦略

-   **データベース**: Supabase の自動バックアップ（7 日間保持）
-   **コード**: GitHub でのバージョン管理
-   **設定**: 環境変数のセキュアな保管

---

## 🎉 デプロイ完了！

**WeStep** が本番環境で稼働開始です！

世界中の人々が「みんなで一歩ずつ」成長していくプラットフォームとして、多くのユーザーに価値を提供していきましょう！ 🌍✨

## 📞 サポート・ヘルプ

デプロイに関する問題や質問があれば：

-   [Vercel ドキュメント](https://vercel.com/docs)
-   [Supabase ドキュメント](https://supabase.com/docs)
-   [Next.js デプロイガイド](https://nextjs.org/docs/deployment)
