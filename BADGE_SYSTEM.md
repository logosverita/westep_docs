# バッジシステム設計書

## 概要

WeStepのバッジシステムは、ユーザーの目標達成を促進し、継続的なモチベーション向上を目的とした機能です。ユーザーの活動に応じて自動的にバッジを付与し、コミュニティ内での成長と貢献を視覚化します。

## テーブル構造分析

### 1. badge_templates テーブル
```sql
model badge_templates {
  id          String   @id
  badge_type  String   @unique @db.VarChar(50)     -- バッジの種類識別子
  name        String   @db.VarChar(100)            -- バッジ表示名
  description String                               -- バッジ説明文
  icon        String   @db.VarChar(10)            -- 絵文字アイコン
  color       String   @default("#3B82F6") @db.VarChar(7) -- カラーコード
  rarity      String   @default("common") @db.VarChar(20) -- レア度
  criteria    Json                                 -- 取得条件（JSON形式）
  sort_order  Int      @default(0)                -- 表示順序
  is_active   Boolean  @default(true)             -- アクティブ状態
  created_at  DateTime @default(now())
  updated_at  DateTime

  @@index([is_active, sort_order])
  @@index([rarity, sort_order])
}
```

### 2. user_badges テーブル
```sql
model user_badges {
  id          String   @id
  user_id     String                              -- ユーザーID
  badge_type  String   @db.VarChar(50)            -- バッジタイプ
  badge_level Int      @default(1)               -- バッジレベル（1=ブロンズ、2=シルバー、3=ゴールド）
  earned_at   DateTime @default(now())           -- 取得日時
  is_active   Boolean  @default(true)            -- 表示状態
  metadata    Json     @default("{}")            -- 追加メタデータ
  users       users    @relation(fields: [user_id], references: [id], onDelete: Cascade)

  @@unique([user_id, badge_type])               -- 同一ユーザー・同一バッジタイプは1つのみ
  @@index([badge_type, badge_level])
  @@index([user_id, is_active])
}
```

## バッジレベルシステム設計

### レベル体系
- **レベル1 (ブロンズ)**: 基本達成条件をクリア
- **レベル2 (シルバー)**: 中級達成条件をクリア  
- **レベル3 (ゴールド)**: 上級達成条件をクリア

### レベル表示仕様
- UIでは枠の色でレベルを表現
  - ブロンズ: `#CD7F32` (銅色)
  - シルバー: `#C0C0C0` (銀色)
  - ゴールド: `#FFD700` (金色)
- 同一バッジタイプ内では最高レベルのみ表示
- レベルアップ時は既存レコードのbadge_levelを更新

## バッジ取得条件詳細設計

### 1. 初心者系バッジ
| バッジタイプ | 名前 | ブロンズ条件 | シルバー条件 | ゴールド条件 |
|-------------|------|-------------|-------------|-------------|
| `newcomer` | 新参者 | アカウント作成から7日以内 | - | - |
| `first_goal` | 第一歩 | 目標を1つ設定 | - | - |
| `first_achievement` | 初達成 | 目標を1つ達成 | - | - |

### 2. 継続・ストリーク系バッジ
| バッジタイプ | 名前 | ブロンズ条件 | シルバー条件 | ゴールド条件 |
|-------------|------|-------------|-------------|-------------|
| `fire_streak` | 継続の炎 | 3日連続達成 | 7日連続達成 | 30日連続達成 |
| `long_streak` | 不屈の意志 | 50日連続達成 | 100日連続達成 | 365日連続達成 |

### 3. 累積達成系バッジ
| バッジタイプ | 名前 | ブロンズ条件 | シルバー条件 | ゴールド条件 |
|-------------|------|-------------|-------------|-------------|
| `achiever` | 達成者 | 10回達成 | 50回達成 | 100回達成 |
| `legend` | 伝説の達成者 | 200回達成 | 500回達成 | 1000回達成 |

### 4. コミュニティ貢献系バッジ
| バッジタイプ | 名前 | ブロンズ条件 | シルバー条件 | ゴールド条件 |
|-------------|------|-------------|-------------|-------------|
| `community_helper` | コミュニティヘルパー | 50回リアクション送信 | 200回リアクション送信 | 500回リアクション送信 |
| `popular` | 人気者 | 25回リアクション受信 | 100回リアクション受信 | 500回リアクション受信 |

### 5. 月間チャレンジ系バッジ
| バッジタイプ | 名前 | ブロンズ条件 | シルバー条件 | ゴールド条件 |
|-------------|------|-------------|-------------|-------------|
| `monthly_champion` | 月間チャンピオン | 月20回達成 | 月25回達成 | 月30回達成 |

### 6. 継続利用系バッジ
| バッジタイプ | 名前 | ブロンズ条件 | シルバー条件 | ゴールド条件 |
|-------------|------|-------------|-------------|-------------|
| `loyal_user` | 忠実なユーザー | 30日継続利用 | 90日継続利用 | 365日継続利用 |

### 7. 特別バッジ（レベルなし）
| バッジタイプ | 名前 | 取得条件 |
|-------------|------|----------|
| `early_adopter` | アーリーアダプター | ベータ版から参加 |
| `comeback_hero` | カムバックヒーロー | 30日以上の休止後に復帰 |

## 条件判定ロジック

### criteriaフィールドの構造
```json
{
  "type": "streak|total_achievements|monthly_achievements|reactions_given|reactions_received|account_age|special",
  "levels": {
    "1": { "minCount": 3 },     // ブロンズ条件
    "2": { "minCount": 7 },     // シルバー条件  
    "3": { "minCount": 30 }     // ゴールド条件
  }
}
```

### 特別条件の例
```json
{
  "type": "special",
  "condition": "beta_user",
  "checkDate": "2025-06-01"
}
```

## バッジ付与システム

### 1. 自動判定タイミング
- 目標達成時
- 日次バッチ処理（深夜実行）
- 月次バッチ処理（月初実行）
- リアクション送信/受信時

### 2. 判定フロー
1. ユーザーの現在のステータス取得
2. 全バッジテンプレートに対して条件チェック
3. 新規取得可能バッジの検出
4. レベルアップ可能バッジの検出
5. user_badgesテーブルへの登録/更新
6. 通知の送信（オプション）

### 3. レベルアップ処理
```sql
-- 既存バッジのレベルアップ
UPDATE user_badges 
SET badge_level = :newLevel, 
    earned_at = NOW(), 
    metadata = jsonb_set(metadata, '{previousLevel}', badge_level::text::jsonb)
WHERE user_id = :userId AND badge_type = :badgeType;
```

## API設計

### 1. バッジ取得API
```typescript
GET /api/badges/user/:userId
Response: {
  badges: {
    badgeType: string,
    name: string,
    description: string,
    icon: string,
    color: string,
    rarity: string,
    level: number,
    earnedAt: string
  }[]
}
```

### 2. バッジ進捗API
```typescript
GET /api/badges/progress/:userId
Response: {
  progress: {
    badgeType: string,
    name: string,
    currentLevel: number,
    nextLevel: number | null,
    currentValue: number,
    requiredValue: number,
    progressPercentage: number
  }[]
}
```

### 3. バッジ付与処理API (内部用)
```typescript
POST /api/badges/check/:userId
Body: {
  triggerType: 'goal_achieved' | 'reaction_given' | 'daily_batch' | 'monthly_batch'
}
```

## フロントエンド表示設計

### 1. バッジ表示コンポーネント
```tsx
<UserBadge 
  badge={badge}
  size="sm" | "md" | "lg"
  showLevel={true}
  showTooltip={true}
/>
```

### 2. バッジコレクション画面
- 取得済みバッジ一覧
- 未取得バッジの進捗表示
- レア度別フィルタ
- レベル別表示

### 3. バッジ取得通知
```tsx
<BadgeEarnedNotification 
  badge={badge}
  isLevelUp={boolean}
  previousLevel={number | null}
/>
```

## パフォーマンス考慮事項

### 1. インデックス戦略
- `user_badges(user_id, is_active)`: ユーザーバッジ取得用
- `badge_templates(is_active, sort_order)`: アクティブバッジ一覧用
- `user_badges(badge_type, badge_level)`: バッジタイプ別統計用

### 2. キャッシュ戦略
- バッジテンプレート: Redis 1時間キャッシュ
- ユーザーバッジ: localStorage 日次更新
- バッジ進捗: リアルタイム更新（WebSocket）

### 3. バッチ処理最適化
- 日次処理: 増分更新のみ
- 月次処理: 並列処理で高速化
- 条件判定: SQLクエリで一括処理

## セキュリティ考慮事項

### 1. 不正取得防止
- サーバーサイドでの条件検証必須
- フロントエンドからの直接バッジ付与禁止
- 監査ログの記録

### 2. データ整合性
- トランザクション内でのバッジ付与
- 重複付与防止ロジック
- ロールバック対応

## 運用・保守

### 1. バッジ管理
- 管理画面でのバッジテンプレート編集
- 一括バッジ付与/剥奪機能
- バッジ統計ダッシュボード

### 2. モニタリング
- バッジ付与エラーの監視
- 異常な付与パターンの検出
- パフォーマンス監視

### 3. 将来拡張予定
- シーズナルバッジ（期間限定）
- チームバッジ（グループ達成）
- カスタムバッジ（管理者付与）
- NFTバッジ連携（将来）

## 実装優先度

### Phase 1 (最優先)
- 基本バッジテンプレートの実装
- 自動付与システムの構築
- UI表示コンポーネント

### Phase 2 (中優先)
- レベルシステムの実装
- 進捗表示機能
- 通知システム

### Phase 3 (低優先)
- 管理画面の構築
- 詳細統計機能
- 高度なフィルタリング

この設計に基づいて、段階的にバッジシステムを実装していきます。