# WeStep 認証システムセットアップガイド

## 🔐 認証システム実装完了

認証システムの実装が完了しました！以下の機能が実装されています：

### ✅ 実装済み機能

1. **Supabase認証サービス** (`src/lib/auth.ts`)
   - サインアップ（メール認証）
   - ログイン/ログアウト
   - ユーザー情報取得・更新
   - パスワードリセット

2. **認証フォームコンポーネント** (`src/components/auth/AuthForm.tsx`)
   - ログイン/サインアップフォーム
   - バリデーション機能
   - エラーハンドリング
   - パスワード表示切替

3. **認証ガード** (`src/components/auth/AuthGuard.tsx`)
   - 保護されたページの認証チェック
   - 未認証ユーザーのリダイレクト
   - ローディング状態の管理

4. **認証ページ** (`src/app/auth/page.tsx`)
   - 美しいログイン/サインアップUI
   - モード切り替え機能

5. **状態管理統合**
   - Zustandストアでの認証状態管理
   - 型安全な実装

## 🚀 テスト手順

### 1. 開発サーバー起動
```bash
npm run dev
```

### 2. アクセス確認
- ブラウザで `http://localhost:3000` にアクセス
- 認証されていない場合は `/auth` に自動リダイレクト

### 3. アカウント作成テスト
1. 「アカウント作成」を選択
2. 有効なメールアドレス、パスワード、名前を入力
3. 「アカウント作成」ボタンをクリック
4. 確認メールが送信される旨のメッセージ表示

### 4. メール確認
1. 登録したメールアドレスの受信箱を確認
2. Supabaseからの確認メールを開く
3. 「Confirm your mail」リンクをクリック

### 5. ログインテスト
1. メール確認後、ログインページに戻る
2. 「ログイン」を選択
3. メールアドレスとパスワードを入力
4. 成功するとメインダッシュボードにリダイレクト

## ⚙️ Supabase設定確認

### Authentication設定

Supabaseダッシュボードで以下を確認してください：

1. **プロジェクト > Authentication > Settings**
   - Enable email confirmations: ✅ ON
   - Enable secure email change: ✅ ON (推奨)
   - Enable custom SMTP: 📧 任意（本番環境では推奨）

2. **URL Configuration**
   - Site URL: `http://localhost:3000` (開発環境)
   - Redirect URLs: `http://localhost:3000/auth/callback` (必要に応じて)

3. **Email Templates**
   - Confirm signup: デフォルトまたはカスタマイズ済み
   - Reset password: デフォルトまたはカスタマイズ済み

## 🔧 トラブルシューティング

### よくある問題と解決方法

#### 1. メール認証ができない
```
❌ 問題: 確認メールが届かない
✅ 解決: 
   - スパムフォルダを確認
   - Supabaseダッシュボードでメール設定確認
   - 開発環境では時間がかかる場合があります
```

#### 2. 認証エラー
```
❌ 問題: "Invalid login credentials"
✅ 解決:
   - メールアドレスが確認済みか確認
   - パスワードが正しいか確認
   - Supabaseダッシュボードでユーザー状況確認
```

#### 3. データベース接続エラー
```
❌ 問題: ユーザー情報がDBに保存されない
✅ 解決:
   - .env.localファイルのDATABASE_URL確認
   - Prismaスキーマとの整合性確認
   - `npm run db:push` 再実行
```

#### 4. 型エラー
```
❌ 問題: TypeScriptエラー
✅ 解決:
   - `npm run build` でビルドエラー確認
   - 型定義の整合性確認
```

#### 5. ⚠️ Google認証ユーザーが目標設定できない
```
❌ 問題: 「保存できませんでした！」エラー
✅ 解決:
   1. ユーザーデータ整合性チェック:
      node scripts/debug-user.js
   
   2. 不整合が見つかった場合、修復実行:
      node scripts/fix-missing-users.js
   
   3. 問題の原因:
      - Google認証ユーザーがPrismaデータベースに未登録
      - OAuth認証の完了時にユーザー作成が失敗
      
   4. 手動でユーザー確認:
      - Supabaseダッシュボード > Authentication > Users
      - アプリデータベースでユーザーテーブル確認
```

## 🎯 次のステップ

認証システムが正常に動作することを確認したら、次の機能を実装できます：

1. **目標設定機能** (Phase 2)
2. **達成機能** (Phase 2) 
3. **統計表示** (Phase 2)
4. **コミュニティ機能** (Phase 2)

## 📝 実装ファイル一覧

```
src/
├── lib/
│   └── auth.ts                 # 認証サービス
├── components/
│   ├── auth/
│   │   ├── AuthForm.tsx       # 認証フォーム
│   │   └── AuthGuard.tsx      # 認証ガード
│   └── ui/                    # UIコンポーネント
├── app/
│   ├── auth/
│   │   └── page.tsx           # 認証ページ
│   ├── layout.tsx             # ルートレイアウト
│   └── page.tsx               # ホームページ（保護済み）
└── store/
    └── useStore.ts            # 状態管理
```

## ✨ 実装詳細

### セキュリティ機能
- メール認証必須
- パスワード強度チェック（6文字以上）
- CSRF保護（Supabase提供）
- セッション管理（Supabase提供）

### UX機能
- パスワード表示切替
- リアルタイムバリデーション
- ローディング状態表示
- エラーメッセージ表示
- 自動リダイレクト

認証システムの実装が完了しました！🎉 

# 認証システム設定ガイド

WeStepでは、メールアドレス認証とGoogle認証を提供しています。

## 📧 メールアドレス認証

メールアドレス認証は標準で有効になっています。

### 基本設定
- Supabaseで自動的にメール確認機能が有効
- パスワードリセット機能も利用可能
- ユーザー登録時に確認メールが送信される

## 🔗 Google認証設定

### Google認証の設定

1. **Google Cloud Consoleでプロジェクトを作成**
   - [Google Cloud Console](https://console.cloud.google.com/)にアクセス
   - 新しいプロジェクトを作成または既存のプロジェクトを選択

2. **Google OAuth 2.0 クライアントIDを設定**
   ```
   # 認証情報 > 認証情報を作成 > OAuth 2.0 クライアントID
   アプリケーションの種類: ウェブアプリケーション
   名前: WeStep
   承認済みのリダイレクトURI: 
   - https://your-project-ref.supabase.co/auth/v1/callback
   ```

3. **Supabaseでプロバイダーを有効化**
   ```
   Supabase Dashboard > Authentication > Providers > Google
   - Enable Google provider
   - Client ID: Google Consoleで取得したクライアントID
   - Client Secret: Google Consoleで取得したクライアントシークレット
   ```

## ⚙️ 環境変数設定

`.env.local`ファイルに以下を追加：

```env
# App URL (OAuth認証のリダイレクト先)
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Supabase (既存)
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
```

## 🔧 開発環境での確認

1. **認証フローのテスト**
   ```bash
   pnpm dev
   ```

2. **認証ページにアクセス**
   ```
   http://localhost:3000/auth
   ```

3. **各認証方式をテスト**
   - メールアドレス登録・ログイン
   - Googleログイン

## 🚨 トラブルシューティング

### よくある問題

#### Google認証エラー
```
Error: redirect_uri_mismatch
```
**解決方法**: Google Cloud ConsoleでリダイレクトURIを正しく設定

#### Supabaseセッションエラー
```
Error: Invalid session
```
**解決方法**: ブラウザのローカルストレージをクリアして再試行

#### Google認証ユーザーの目標設定エラー
```
Error: 保存できませんでした！
```
**原因**: Google認証で作成されたユーザーがPrismaデータベースに未登録

**解決手順**:
1. **問題の確認**
   ```bash
   node scripts/debug-user.js
   ```

2. **データ修復**
   ```bash
   node scripts/fix-missing-users.js
   ```

3. **修復完了後の確認**
   - アプリに再ログイン
   - 目標設定を再試行

### デバッグ方法

1. **ブラウザ開発者ツール**
   - Consoleでエラーログを確認
   - Networkタブで認証リクエストを確認

2. **Supabaseダッシュボード**
   - Authentication > Users でユーザー作成状況を確認
   - Authentication > Logs で認証ログを確認

3. **データベース確認**
   ```bash
   # ユーザー状況の詳細確認
   node scripts/debug-user.js
   
   # 不整合データの自動修復
   node scripts/fix-missing-users.js
   ```

## 📝 実装のカスタマイズ

### 新しいソーシャルプロバイダーの追加

1. **AuthStoreに新しいメソッドを追加**
   ```typescript
   signInWithNewProvider: async () => {
     const { data, error } = await supabase.auth.signInWithOAuth({
       provider: 'new_provider',
       options: {
         redirectTo: `${process.env.NEXT_PUBLIC_APP_URL}/auth/callback`,
       }
     })
     // エラーハンドリング
   }
   ```

2. **SocialAuthButtonsコンポーネントに新しいボタンを追加**

3. **Supabaseでプロバイダーを有効化**

### 認証後のリダイレクト先をカスタマイズ

`app/auth/callback/page.tsx`でリダイレクト先を変更できます：

```typescript
// 成功時のリダイレクト
router.push('/dashboard') // または任意のパス
```

## 🔒 セキュリティ考慮事項

1. **環境変数の管理**
   - `.env.local`は`.gitignore`に含める
   - 本番環境では適切なシークレット管理を使用

2. **CORS設定**
   - Supabaseで適切なCORS設定を行う
   - 本番ドメインのみを許可

3. **リダイレクトURI検証**
   - OAuth プロバイダーで正確なリダイレクトURIを設定
   - 予期しないリダイレクトを防ぐ

4. **データ整合性**
   - 定期的にユーザーデータの整合性をチェック
   - OAuth認証時のユーザー作成プロセスを監視

---

## 🛠️ メンテナンス用スクリプト

### ユーザーデータ状況確認
```bash
node scripts/debug-user.js
```

### 不整合データ修復
```bash
node scripts/fix-missing-users.js
```

認証システムの設定について不明点がある場合は、Supabaseのドキュメントも併せてご確認ください。 