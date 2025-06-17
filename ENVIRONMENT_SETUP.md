# WeStep 環境設定ガイド

## 🚀 **環境変数の設定**

### 1. .env.local ファイルの作成

プロジェクトルートに `.env.local` ファイルを作成してください：

```bash
# WeStep アプリケーション環境変数
# ===================================

# Next.js 設定
NODE_ENV=development
NEXT_PUBLIC_APP_URL=http://localhost:3001

# Supabase 設定 (必須)
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key

# データベース接続 (必須)
DATABASE_URL=postgresql://postgres:[YOUR_PASSWORD]@[YOUR_HOST]:[YOUR_PORT]/postgres

# セキュリティ設定
NEXTAUTH_SECRET=your_secret_key_here_make_it_strong
NEXTAUTH_URL=http://localhost:3001

# API 設定
API_SECRET_KEY=your_api_secret_key_here

# 開発設定
NEXT_PUBLIC_DEBUG=true
```

## 🏗️ **Supabase プロジェクトセットアップ**

### 1. Supabase プロジェクト作成
1. [Supabase](https://supabase.com)にアクセス
2. 「New Project」をクリック
3. プロジェクト名: `WeStep`
4. データベースパスワードを設定（強力なものを選択）
5. リージョン: `Asia Northeast (Tokyo)` を推奨

### 2. 環境変数の取得
Supabaseダッシュボードから以下を取得：

1. **Settings** → **API** から：
   - `Project URL` → `NEXT_PUBLIC_SUPABASE_URL`
   - `anon public` キー → `NEXT_PUBLIC_SUPABASE_ANON_KEY`

2. **Settings** → **Database** から：
   - `Connection string` → `DATABASE_URL`

### 3. 認証設定
1. **Authentication** → **Settings** → **Site URL**:
   - `http://localhost:3001` を追加
   - 本番環境のドメインも追加

2. **Authentication** → **Providers**:
   - Emailプロバイダーを有効化
   - 必要に応じて他のプロバイダー（Google, GitHub等）も設定

## 🗄️ **データベースセットアップ**

### 1. Prisma クライアント生成
```bash
npm run db:generate
```

### 2. データベーススキーマ適用
```bash
npm run db:push
```

### 3. 初期データ投入（オプション）
```bash
npm run db:seed
```

## ✅ **接続テスト**

### 1. 開発サーバー起動
```bash
npm run dev
```

### 2. テスト手順
1. ブラウザで `http://localhost:3001` にアクセス
2. `/auth` ページで新規ユーザー登録
3. 認証が成功すればルートページにリダイレクト
4. コンソールでエラーがないことを確認

## 🐛 **トラブルシューティング**

### よくある問題

#### 1. Supabase接続エラー
```
Error: Invalid Supabase URL or Key
```
**解決策**:
- `.env.local` の設定値を再確認
- Supabaseダッシュボードから正しい値をコピー

#### 2. データベース接続エラー
```
Error: Can't reach database server
```
**解決策**:
- `DATABASE_URL` の接続文字列を確認
- パスワードに特殊文字が含まれる場合はURLエンコード
- ファイアウォール設定を確認

#### 3. 認証エラー
```
Error: Authentication failed
```
**解決策**:
- Supabase認証設定でSite URLが正しく設定されているか確認
- 認証プロバイダーが有効化されているか確認

## 🔧 **package.json スクリプト**

```json
{
  "scripts": {
    "dev": "next dev",
    "db:generate": "prisma generate",
    "db:push": "prisma db push",
    "db:pull": "prisma db pull",
    "db:seed": "prisma db seed",
    "db:studio": "prisma studio",
    "db:migrate": "prisma migrate dev",
    "db:reset": "prisma migrate reset"
  }
}
```

## 🎯 **次のステップ**

環境設定完了後：
1. 認証フローのテスト
2. 目標作成機能の実装
3. リアルタイム統計の実装
4. コミュニティ機能の追加

---

## 🆘 **サポート**

問題が発生した場合：
1. コンソールエラーを確認
2. `.env.local` の設定を再確認
3. Supabaseダッシュボードで接続状況を確認
4. 必要に応じてプロジェクトを再作成 