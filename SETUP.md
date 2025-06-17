# WeStep 開発環境構築ガイド

**WeStep** - みんなで一歩ずつ、世界を変えよう

## 🚀 環境構築手順

### 前提条件

以下がインストールされていることを確認してください：

-   **Node.js** 18.0 以上
-   **npm** または **yarn**
-   **Git**
-   **VSCode**（推奨）

### 🔧 1. プロジェクトのクローン

```bash
git clone <repository-url>
cd WeStep
```

### 📦 2. 依存関係のインストール

```bash
# npmを使用する場合
npm install

# yarnを使用する場合
yarn install
```

### 🌐 3. 環境変数の設定

1. `.env.example` を `.env.local` にコピー：

```bash
cp .env.example .env.local
```

2. `.env.local` ファイルを編集し、必要な環境変数を設定：

```env
# Supabase設定
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key

# データベース設定
DATABASE_URL="postgresql://username:password@localhost:5432/westep_db"

# NextAuth設定（認証）
NEXTAUTH_SECRET=your_nextauth_secret
NEXTAUTH_URL=http://localhost:3000
```

### 🗄️ 4. データベースセットアップ

#### Supabase の場合：

1. [Supabase](https://supabase.com) でプロジェクト作成
2. データベース URL と API キーを `.env.local` に設定
3. Prisma スキーマの適用：

```bash
npx prisma db push
npx prisma generate
```

#### ローカル PostgreSQL の場合：

1. PostgreSQL をインストール
2. データベース作成：

```sql
CREATE DATABASE westep_db;
```

3. Prisma マイグレーション実行：

```bash
npx prisma migrate dev --name init
npx prisma generate
```

### 🎨 5. Tailwind CSS の設定確認

Tailwind CSS の設定は既に完了していますが、カスタムスタイルを確認：

```bash
# Tailwind CSS のビルド確認
npm run build
```

### 🏃‍♂️ 6. 開発サーバーの起動

```bash
npm run dev
```

ブラウザで `http://localhost:3000` を開いて WeStep アプリケーションを確認できます。

## 📋 利用可能なスクリプト

| コマンド             | 説明                             |
| -------------------- | -------------------------------- |
| `npm run dev`        | 開発サーバーを起動               |
| `npm run build`      | 本番用ビルドを作成               |
| `npm run start`      | 本番サーバーを起動               |
| `npm run lint`       | ESLint でコードチェック          |
| `npx prisma studio`  | Prisma Studio でデータベース管理 |
| `npx prisma db push` | データベーススキーマを同期       |

## 🔍 開発ツール

### VSCode 推奨拡張機能

以下の拡張機能をインストールすることを推奨します：

-   **ES7+ React/Redux/React-Native snippets**
-   **Prettier - Code formatter**
-   **ESLint**
-   **Tailwind CSS IntelliSense**
-   **Prisma**
-   **TypeScript Importer**

### 設定ファイル

`.vscode/settings.json` を作成して以下を設定：

```json
{
	"editor.formatOnSave": true,
	"editor.defaultFormatter": "esbenp.prettier-vscode",
	"typescript.preferences.importModuleSpecifier": "relative",
	"tailwindCSS.experimental.classRegex": [
		["clsx\\(([^)]*)\\)", "(?:'|\"|`)([^']*)(?:'|\"|`)"]
	]
}
```

## 🚨 トラブルシューティング

### よくある問題と解決方法

#### 1. Node.js バージョンエラー

```bash
# Node.js のバージョン確認
node --version

# 18.0 以上が必要です
# nvmを使用してバージョンを切り替え
nvm use 18
```

#### 2. 依存関係インストールエラー

```bash
# node_modules を削除して再インストール
rm -rf node_modules package-lock.json
npm install
```

#### 3. Prisma 接続エラー

```bash
# Prismaクライアントを再生成
npx prisma generate

# データベース接続を確認
npx prisma db pull
```

#### 4. Tailwind CSS スタイルが適用されない

```bash
# Tailwind設定を確認
npx tailwindcss -i ./src/app/globals.css -o ./dist/output.css --watch
```

## 📁 プロジェクト構造

```
WeStep/
├── src/
│   ├── app/                 # Next.js App Router
│   │   ├── layout.tsx       # ルートレイアウト
│   │   ├── page.tsx         # ホームページ
│   │   └── globals.css      # グローバルスタイル
│   ├── components/          # Reactコンポーネント
│   ├── lib/                 # ユーティリティ関数
│   └── types/               # TypeScript型定義
├── prisma/
│   └── schema.prisma        # データベーススキーマ
├── public/                  # 静的ファイル
├── docs/                    # ドキュメント
├── package.json             # 依存関係とスクリプト
├── tailwind.config.js       # Tailwind CSS設定
├── tsconfig.json            # TypeScript設定
└── README.md                # プロジェクト概要
```

## 🎯 次のステップ

環境構築が完了したら：

1. **デザインシステムの実装** - コンポーネントライブラリの構築
2. **認証システムの実装** - Supabase Auth の統合
3. **データベース操作の実装** - Prisma を使用した CRUD 操作
4. **リアルタイム機能の実装** - WebSocket によるコミュニティ機能
5. **デプロイメント** - Vercel へのデプロイ

詳細な実装ガイドは `DEVELOPMENT.md` を参照してください。

## 📞 サポート

問題が発生した場合は、以下を確認してください：

-   [Next.js ドキュメント](https://nextjs.org/docs)
-   [Prisma ドキュメント](https://www.prisma.io/docs)
-   [Supabase ドキュメント](https://supabase.com/docs)
-   [Tailwind CSS ドキュメント](https://tailwindcss.com/docs)

---

**WeStep** で世界を変える一歩を踏み出しましょう！ 🌍✨
