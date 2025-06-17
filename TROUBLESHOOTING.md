# トラブルシューティングガイド

WeStepアプリケーションの開発・運用時によくある問題と解決方法をまとめています。

## データベース関連の問題

### Prismaスキーマ関連エラー

#### 問題: テーブル名・フィールド名の不一致エラー
```
P2021: The table `Achievement` does not exist in the current database.
```

**原因**: データベースのテーブル名はsnake_case（`achievements`）だが、Prismaスキーマでは大文字・小文字を混在していた

**解決方法**:
1. Prismaスキーマ（`prisma/schema.prisma`）で正しいテーブル名を指定
```prisma
model Achievement {
  @@map("achievements")  // データベース上のテーブル名を明示
  // ...
}
```

2. フィールド名もsnake_caseに統一
```prisma
model Achievement {
  user_id String
  created_at DateTime @default(now())
  @@map("achievements")
}
```

#### 問題: フィールド名の不一致
```
P2021: Unknown field `userId` for model Achievement
```

**解決方法**: 
- データベースではsnake_case（`user_id`）
- TypeScriptインターフェースではcamelCase（`userId`）
- Prismaクライアントではsnake_caseを使用

```typescript
// 正しい書き方
const achievements = await prisma.achievements.findMany({
  where: { user_id: userId }  // snake_case
})

// フロントエンド向けにcamelCaseに変換
const formatted = achievements.map(item => ({
  userId: item.user_id,  // snake_case → camelCase
  createdAt: item.created_at
}))
```

### データベース接続エラー

#### 問題: P1001 - データベース接続失敗
```
P1001: Can't reach database server
```

**原因と解決方法**:

1. **Supabaseプロジェクトの一時停止**
   - Supabaseの無料プランでは一定期間アクセスがないとプロジェクトが一時停止
   - 解決: [Supabaseダッシュボード](https://supabase.com/dashboard/projects)でプロジェクトを手動で再開

2. **環境変数の設定ミス**
   ```bash
   # .env.localを確認
   DATABASE_URL="postgresql://..."
   NEXT_PUBLIC_SUPABASE_URL="https://..."
   NEXT_PUBLIC_SUPABASE_ANON_KEY="..."
   ```

3. **ネットワーク接続問題**
   ```bash
   # 接続テスト
   pnpm test:connection
   ```

#### 問題: タイムアウトエラー
```
P1001: Connect timeout
```

**解決方法**:
1. 接続プールの設定を確認
```typescript
// lib/prisma.ts
const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_URL,
    },
  },
})
```

2. リトライ機能を実装済み（`/api/goals/route.ts`参照）

### マイグレーション関連

#### 問題: マイグレーション失敗
```bash
pnpm db:push
# Error: Migration failed
```

**解決手順**:
1. 現在のスキーマ状態を確認
```bash
pnpm db:studio  # Prisma Studioで確認
```

2. 段階的にマイグレーション
```bash
# Prismaクライアント再生成
pnpm db:generate

# スキーマを段階的にプッシュ
pnpm db:push --force-reset  # 注意: データが削除されます
```

3. 権限問題の場合
```sql
-- scripts/fix-supabase-permissions.sql を実行
-- SupabaseダッシュボードのSQL Editorで実行
```

## Edge Runtime関連の問題

### Prisma動的インポートエラー

#### 問題: Edge Runtime環境でPrismaが動作しない
```
Error: Dynamic require of "..." is not supported
```

**解決方法**: 動的インポートパターンを使用
```typescript
// ❌ 直接インポート（Edge Runtimeで問題）
import { prisma } from '@/lib/prisma'

// ✅ 動的インポート（推奨）
async function getPrismaClient() {
  const { prisma } = await import('@/lib/prisma')
  return prisma
}

export async function GET(request: NextRequest) {
  const prisma = await getPrismaClient()
  const result = await prisma.daily_goals.findMany(...)
}
```

### 認証関連の問題

#### 問題: 認証状態が不安定
```
Error: User not authenticated
```

**解決方法**:
1. `getCurrentUserFallback`を使用（フォールバック認証付き）
```typescript
import { getCurrentUserFallback } from '@/lib/auth-fallback'

const user = await getCurrentUserFallback()
if (!user) {
  return NextResponse.json({ error: '認証が必要です' }, { status: 401 })
}
```

2. クライアントサイドでの認証状態確認
```typescript
// フロントエンド
const { user, isLoading } = useAuthStore()

if (isLoading) return <Loading />
if (!user) return <AuthForm />
```

### API応答の問題

#### 問題: APIが503エラーを返す
```json
{
  "error": "データベースに接続できません",
  "details": "Supabaseプロジェクトが一時停止されている可能性があります"
}
```

**解決手順**:
1. Supabaseプロジェクトの状態を確認
2. プロジェクトを再開
3. 数分待ってから再試行

#### 問題: データが返されない
**チェックポイント**:
1. ユーザー認証は正常か？
2. データベースにデータは存在するか？
3. プライバシー設定で非表示になっていないか？

## フロントエンド関連の問題

### hooks使用時のエラー

#### 問題: 無限ループ・過度なAPI呼び出し
**原因**: 認証状態の変化による再レンダリング

**解決方法**:
```typescript
// ❌ 問題のあるパターン
const { user } = useAuthStore()

// ✅ 個別セレクターを使用
const user = useAuthStore(state => state.user)
const isLoading = useAuthStore(state => state.isLoading)
```

### データフェッチングの問題

#### 問題: キャッシュが効かない
**解決方法**: 適切なキャッシュキーを使用
```typescript
const cacheKey = `analytics_${userId}_${getTodayKey()}`
const cachedData = localStorage.getItem(cacheKey)
```

## デプロイ・本番環境の問題

### ビルドエラー

#### 問題: TypeScriptエラーでビルドが失敗
```bash
pnpm build
# Type error: Property 'user_id' does not exist
```

**解決方法**:
1. 型定義を確認・修正
2. 一時的な回避（開発時のみ）
```typescript
// tsconfig.json
{
  "compilerOptions": {
    "noEmit": true,
    "skipLibCheck": true  // 一時的な回避
  }
}
```

### 環境変数の問題

#### 問題: 本番環境で環境変数が読み込まれない
**チェックリスト**:
- [ ] `NEXT_PUBLIC_`プレフィックス（クライアントサイド変数）
- [ ] Vercel/Netlify等でのデプロイ先設定
- [ ] `.env.local`ファイルがgitignoreされているか

## 診断・デバッグツール

### 接続テスト
```bash
# データベース接続テスト
pnpm test:connection

# 認証テスト
pnpm test:auth
```

### ログ確認
```bash
# 開発サーバーのログ
tail -f dev.log

# 特定のAPIエンドポイントのログ
# コンソールで以下を確認:
# - 📡 API呼び出し開始
# - ✅ 成功ログ
# - ❌ エラーログ
# - ⚠️ 警告ログ
```

### データベース直接確認
```bash
# Prisma Studio起動
pnpm db:studio

# または診断スクリプト実行
node scripts/diagnose-permissions.ts
```

## よくある質問

### Q: 「目標が作成できない」
**A**: 
1. 同じ日の目標が既に存在していないか確認
2. 日付形式が正しいか確認（YYYY-MM-DD）
3. ユーザー認証が有効か確認

### Q: 「ランキングが表示されない」
**A**:
1. 完了済み目標が存在するか確認
2. `current_streak > 0`の条件を満たすユーザーがいるか確認
3. プライバシー設定で非表示になっていないか確認

### Q: 「リアルタイムフィードが更新されない」
**A**:
1. Supabaseプロジェクトのリアルタイム機能が有効か確認
2. 目標完了時にフィードエントリが作成されているか確認
3. プライバシー設定（`showInFeed`）を確認

## 緊急時の対応

### データベース完全復旧
```bash
# 緊急時のみ使用（データが削除されます）
pnpm db:push --force-reset

# テストデータ作成
node scripts/create-test-user-data.js
```

### 権限問題の緊急修復
```sql
-- scripts/fix-supabase-permissions-enhanced.sql
-- Supabaseダッシュボードで実行
```

## サポートとヘルプ

開発中に問題が発生した場合：

1. **まず確認**: このトラブルシューティングガイド
2. **ログ確認**: コンソールログとdev.log
3. **診断実行**: `pnpm test:connection`、`pnpm test:auth`
4. **データベース確認**: `pnpm db:studio`

問題が解決しない場合は、エラーメッセージとコンテキストを含めて報告してください。