# WeStep サブスクリプション機能実装計画

**🎯 目標**: ピン留め機能をサブスクライバー向け特典として実装  
**🚀 ステータス**: 基本実装完了（マイグレーション保留中）  
**📅 実装日**: 2024年12月

---

## 🌟 実装概要

### 📌 ピン留め機能の制限
- **無料ユーザー**: ピン留め機能は使用不可
- **プレミアムユーザー**: 特別な達成（初回達成、ストリーク、マイルストーン、カムバック）をピン留め可能
- **表示**: フィードの上部にピン留めされた達成を表示

### 🔒 現在の実装状態

#### ✅ 実装済み機能
1. **達成フィード作成時のピン留め制限**
   - `app/api/goals/route.ts` の `createAchievementFeedEntry` 関数を修正
   - 通常ユーザーはピン留め機能を使用不可（`isPinned: false`）

2. **フロントエンド表示の更新**
   - `components/realtime/FeedItem.tsx`: ピン留めアイコンに「プレミアム特典」表示を追加
   - `components/realtime/RealtimeFeed.tsx`: ヘッダーにプレミアム特典の説明を追加

3. **サブスクリプションAPIの基本構造**
   - `app/api/subscription/route.ts`: GET/POST/DELETE エンドポイントの基本実装（モック）
   - `lib/subscription.ts`: サブスクリプション状態チェック用ユーティリティ関数

#### ⏳ 実装保留中
1. **データベースマイグレーション**
   ```sql
   -- Userテーブルに追加予定フィールド
   subscription_status VARCHAR(20) DEFAULT 'free'
   subscription_start_date TIMESTAMP
   subscription_end_date TIMESTAMP
   ```

2. **実際のサブスクリプション判定ロジック**
   - 現在は全ユーザーがピン留め無効
   - マイグレーション後にサブスクリプション状態による制御を実装予定

---

## 🛠️ 技術詳細

### データベース設計

```sql
-- Usersテーブル更新（実装予定）
ALTER TABLE users ADD COLUMN subscription_status VARCHAR(20) DEFAULT 'free';
ALTER TABLE users ADD COLUMN subscription_start_date TIMESTAMP;
ALTER TABLE users ADD COLUMN subscription_end_date TIMESTAMP;

-- サブスクリプション状態
-- 'free': 無料プラン
-- 'active': アクティブなサブスクリプション
-- 'expired': 期限切れ
-- 'cancelled': キャンセル済み
```

### ピン留め判定ロジック

```typescript
// 将来の実装予定
const isSubscriber = user?.subscriptionStatus === 'active' && 
  (!user.subscriptionEndDate || new Date() <= user.subscriptionEndDate)

// 特別な達成タイプでのみピン留め適用
const shouldPin = isSubscriber && (
  achievementType === 'first_goal' ||
  achievementType === 'milestone' ||
  achievementType === 'streak' ||
  achievementType === 'comeback'
)
```

---

## 🎨 UI/UX 設計

### フィード表示
- **ピン留めアイテム**: 上部に黄色背景で表示
- **プレミアム表示**: 「📌 ピン留め（プレミアム特典）」バッジ
- **説明文**: ヘッダーに「💎 プレミアム特典: 特別な達成はピン留めされ、フィードの上部に表示されます」

### 制限メッセージ
- 無料ユーザーには特別なピン留め促進メッセージは現在なし
- 将来的にはプレミアムアップグレードの案内を追加予定

---

## 🚀 今後の実装計画

### Phase 1: データベース移行
1. 本番環境でのマイグレーション実行
2. Prismaクライアントの型更新
3. サブスクリプション判定ロジックの有効化

### Phase 2: 決済統合
1. Stripe等の決済サービス統合
2. サブスクリプション自動更新機能
3. プラン変更機能

### Phase 3: 追加特典
1. 高度な統計機能
2. カスタムバッジ
3. 追加のフィード機能

---

## 📊 期待される効果

### ユーザー体験
- **差別化**: 無料ユーザーとプレミアムユーザーの明確な違い
- **価値提供**: ピン留めによる達成の目立ちやすさ
- **コミュニティ感**: プレミアムユーザーの達成がより注目される

### ビジネス価値
- **収益化**: サブスクリプション収入の基盤
- **ユーザー継続**: プレミアム特典による継続率向上
- **コミュニティ活性化**: 特別な達成の可視化

---

**✨ WeStepの持続可能な成長と、ユーザーにとって価値のあるプレミアム体験を提供します！** 