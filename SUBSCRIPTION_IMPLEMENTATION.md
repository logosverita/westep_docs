# WeStep サブスクリプション機能実装計画

## 🎯 実装目標

コミュニティメッセージ機能をプレミアム会員限定機能として実装し、サブスクリプション管理システムを構築する。

## 📋 実装計画

### Phase 1: 基盤整備 ✅ 完了
1. **データベーススキーマ設計**
   - ✅ Subscriptionモデルの追加
   - ✅ Userモデルにサブスクリプション関連フィールド追加
   - ✅ AchievementsFeedの`goalTitle`を任意に変更
   - ✅ `achievementType`を30文字に拡張

2. **API実装**
   - ✅ `/api/subscription` - サブスクリプション管理API
   - ✅ `/api/community-messages` - メッセージ投稿・取得API
   - ✅ サブスクリプション状態チェック機能

3. **フロントエンド基盤**
   - ✅ `useSubscription` フック
   - ✅ `SubscriptionManager` コンポーネント
   - ✅ `CommunityMessageForm` の権限制御

### Phase 2: 認証・権限管理 🔄 進行中
1. **認証システム統合**
   - ⚠️ ユーザーID取得の問題解決が必要
   - 🔄 Supabase認証とPrismaデータベースの連携
   - 🔄 セッション管理の改善

2. **権限チェック機能**
   - ✅ サブスクリプション状態による機能制限
   - ✅ 開発環境でのバイパス機能
   - 🔄 実際のユーザー認証との統合

### Phase 3: UI/UX改善 📋 計画中
1. **サブスクリプション管理画面**
   - 📋 プラン比較表示
   - 📋 アップグレード・ダウングレード機能
   - 📋 請求履歴表示

2. **プレミアム機能の視覚的表示**
   - 📋 プレミアム会員バッジ
   - 📋 機能制限の分かりやすい表示
   - 📋 アップグレード促進UI

### Phase 4: 決済システム統合 📋 将来計画
1. **Stripe統合**
   - 📋 決済処理
   - 📋 Webhook処理
   - 📋 自動サブスクリプション更新

## 🔧 現在の実装状況

### ✅ 完了した機能

#### データベース設計
```sql
-- Subscriptionテーブル
CREATE TABLE subscriptions (
  id TEXT PRIMARY KEY,
  user_id TEXT UNIQUE NOT NULL,
  plan_type VARCHAR(20) DEFAULT 'free',
  status VARCHAR(20) DEFAULT 'inactive',
  start_date TIMESTAMP,
  end_date TIMESTAMP,
  features JSON DEFAULT '{"pinningEnabled": false, "messagePostingEnabled": false, "maxGoals": 10, "advancedStats": false}',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

#### API エンドポイント
- `GET /api/subscription` - サブスクリプション状態取得
- `POST /api/subscription` - サブスクリプション開始/更新
- `DELETE /api/subscription` - サブスクリプションキャンセル
- `POST /api/community-messages` - メッセージ投稿（権限チェック付き）
- `GET /api/community-messages` - メッセージ一覧取得

#### フロントエンド機能
- サブスクリプション状態管理
- プレミアム機能の制限表示
- アップグレード・キャンセル機能

### ⚠️ 現在の課題

#### 1. 認証問題
**症状**: "ログインが必要です"エラー
**原因**: ユーザーID取得の不整合
**対策**: 
- Supabase認証状態とPrismaユーザーデータの同期
- 認証ヘッダーの統一

#### 2. 型定義エラー
**症状**: Prismaクライアントの型エラー
**原因**: スキーマ変更後のクライアント再生成不完全
**対策**: 
- Prismaクライアント完全再生成
- TypeScript型定義の更新

## 🚀 次のステップ

### 即座に対応が必要
1. **認証問題の解決**
   ```typescript
   // ユーザーID取得の統一
   const getUserId = (request: NextRequest): string | null => {
     // 複数の方法でユーザーIDを取得
     const authHeader = request.headers.get('authorization')
     const userIdHeader = request.headers.get('user-id')
     // Supabaseセッションからの取得も追加
   }
   ```

2. **開発環境でのテスト機能**
   ```typescript
   // 開発環境でのサブスクリプション状態切り替え
   const isDev = process.env.NODE_ENV === 'development'
   if (isDev) {
     // テスト用のサブスクリプション状態切り替え
   }
   ```

### 中期的な改善
1. **ユーザー体験の向上**
   - プレミアム機能の価値を明確に表示
   - アップグレード促進のUI改善
   - 機能制限の分かりやすい説明

2. **管理機能の追加**
   - 管理者用のサブスクリプション管理画面
   - 使用状況の分析機能
   - 収益レポート

## 📊 機能仕様

### サブスクリプションプラン

#### フリープラン
- 目標設定: 最大10個
- コミュニティメッセージ: ❌
- ピン留め機能: ❌
- 詳細統計: ❌

#### プレミアムプラン
- 目標設定: 最大100個
- コミュニティメッセージ: ✅
- ピン留め機能: ✅
- 詳細統計: ✅

#### プロプラン（将来実装）
- 目標設定: 最大500個
- 全プレミアム機能: ✅
- 優先サポート: ✅
- カスタム統計: ✅

### 権限チェックフロー
```
1. ユーザー認証確認
2. サブスクリプション状態取得
3. 機能権限チェック
4. 期限切れチェック
5. アクセス許可/拒否
```

## 🔍 テスト計画

### 単体テスト
- [ ] サブスクリプションAPI
- [ ] 権限チェック機能
- [ ] メッセージ投稿機能

### 統合テスト
- [ ] 認証からメッセージ投稿まで
- [ ] サブスクリプション変更フロー
- [ ] 期限切れ処理

### E2Eテスト
- [ ] ユーザー登録からプレミアム機能利用まで
- [ ] アップグレード・ダウングレードフロー

## 📝 実装メモ

### 開発環境での確認方法
1. ブラウザで `http://localhost:3002` にアクセス
2. ログイン後、コミュニティメッセージ機能を確認
3. 開発者ツールでAPIリクエストを監視
4. サブスクリプション状態切り替えボタンでテスト

### デバッグ情報
- コンソールログでサブスクリプション状態を確認
- ネットワークタブでAPIレスポンスを確認
- データベースで実際のサブスクリプションレコードを確認

---

**最終更新**: 2024年6月3日
**実装者**: WeStep開発チーム
**ステータス**: Phase 2 進行中 