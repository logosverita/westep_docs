# WeStep リアルタイム達成フィード実装計画

**🎯 目標**: Twitchチャット風のリアルタイム達成フィード実装  
**🚀 優先度**: 高（コミュニティ体験の核心機能）  
**⏰ 実装予定**: Phase 2メイン機能

---

## 🌟 機能概要

### 🎮 Twitchチャット風UI/UX
- **リアルタイムフィード**: 達成がリアルタイムで流れる
- **ユーザーバッジ**: コミュニティ貢献度を可視化
- **美しいアニメーション**: 達成時のお祝い効果
- **インタラクション**: いいね、リアクション機能

### 🏆 コミュニティバッジシステム
- **🌱 新参者**: 登録1週間未満
- **🔥 熱血**: 連続達成7日以上
- **⭐ 貢献者**: 月間達成30回以上
- **👑 伝説**: 総達成100回以上
- **💎 サブスク1ヶ月**: アプリ利用1ヶ月
- **🎖️ サブスク6ヶ月**: アプリ利用6ヶ月
- **🏅 サブスク1年**: アプリ利用1年

---

## 🗄️ データベース設計

### 1. achievements_feed テーブル
```sql
CREATE TABLE achievements_feed (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  goal_title TEXT NOT NULL,
  goal_category TEXT,
  achievement_type TEXT NOT NULL, -- 'completed', 'streak', 'milestone'
  streak_count INTEGER DEFAULT 0,
  message TEXT, -- ユーザーのコメント（オプション）
  
  -- メタデータ
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  -- リアルタイム用
  is_pinned BOOLEAN DEFAULT FALSE,
  reaction_count INTEGER DEFAULT 0,
  
  -- インデックス用
  CONSTRAINT valid_achievement_type CHECK (achievement_type IN ('completed', 'streak', 'milestone', 'first_goal', 'comeback'))
);

-- インデックス作成
CREATE INDEX idx_achievements_feed_created_at ON achievements_feed(created_at DESC);
CREATE INDEX idx_achievements_feed_user_id ON achievements_feed(user_id);
CREATE INDEX idx_achievements_feed_type ON achievements_feed(achievement_type);
```

### 2. user_badges テーブル
```sql
CREATE TABLE user_badges (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  badge_type TEXT NOT NULL,
  badge_level INTEGER DEFAULT 1,
  earned_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  is_active BOOLEAN DEFAULT TRUE,
  
  -- バッジメタデータ
  metadata JSONB DEFAULT '{}',
  
  UNIQUE(user_id, badge_type)
);

-- バッジタイプ定義
INSERT INTO user_badges (user_id, badge_type, badge_level, metadata) VALUES
-- システムバッジの例
('system', 'newcomer', 1, '{"icon": "🌱", "title": "新参者", "description": "WeStepへようこそ！"}'),
('system', 'fire_streak', 1, '{"icon": "🔥", "title": "熱血", "description": "7日連続達成"}'),
('system', 'contributor', 1, '{"icon": "⭐", "title": "貢献者", "description": "月間30回達成"}'),
('system', 'legend', 1, '{"icon": "👑", "title": "伝説", "description": "総達成100回"}'),
('system', 'subscriber_1m', 1, '{"icon": "💎", "title": "サブスク1ヶ月", "description": "1ヶ月継続利用"}'),
('system', 'subscriber_6m', 1, '{"icon": "🎖️", "title": "サブスク6ヶ月", "description": "6ヶ月継続利用"}'),
('system', 'subscriber_1y', 1, '{"icon": "🏅", "title": "サブスク1年", "description": "1年継続利用"});
```

### 3. user_community_stats テーブル
```sql
CREATE TABLE user_community_stats (
  user_id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  
  -- 基本統計
  total_achievements INTEGER DEFAULT 0,
  current_streak INTEGER DEFAULT 0,
  max_streak INTEGER DEFAULT 0,
  
  -- 月間統計
  monthly_achievements INTEGER DEFAULT 0,
  monthly_reset_date DATE DEFAULT DATE_TRUNC('month', NOW()),
  
  -- コミュニティ活動
  reactions_given INTEGER DEFAULT 0,
  reactions_received INTEGER DEFAULT 0,
  
  -- アカウント情報
  account_created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  last_active_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  -- 更新日時
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### 4. feed_reactions テーブル
```sql
CREATE TABLE feed_reactions (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  feed_id UUID NOT NULL REFERENCES achievements_feed(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  reaction_type TEXT NOT NULL, -- 'like', 'fire', 'clap', 'heart'
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  UNIQUE(feed_id, user_id, reaction_type)
);
```

---

## ⚡ Supabaseリアルタイム設定

### 1. リアルタイム有効化
```sql
-- テーブルのリアルタイム機能を有効化
ALTER PUBLICATION supabase_realtime ADD TABLE achievements_feed;
ALTER PUBLICATION supabase_realtime ADD TABLE feed_reactions;
ALTER PUBLICATION supabase_realtime ADD TABLE user_community_stats;
```

### 2. Row Level Security (RLS)
```sql
-- achievements_feed のRLS
ALTER TABLE achievements_feed ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Anyone can read feed" ON achievements_feed
  FOR SELECT USING (true);

CREATE POLICY "Users can insert their own achievements" ON achievements_feed
  FOR INSERT WITH CHECK (auth.uid() = user_id);

-- feed_reactions のRLS
ALTER TABLE feed_reactions ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Anyone can read reactions" ON feed_reactions
  FOR SELECT USING (true);

CREATE POLICY "Users can manage their own reactions" ON feed_reactions
  FOR ALL USING (auth.uid() = user_id);
```

---

## 🎨 UI/UXコンポーネント設計

### 1. RealtimeFeed コンポーネント
```typescript
interface FeedItem {
  id: string;
  user: {
    id: string;
    name: string;
    badges: Badge[];
  };
  achievement: {
    title: string;
    type: 'completed' | 'streak' | 'milestone';
    streakCount?: number;
  };
  message?: string;
  timestamp: Date;
  reactions: Reaction[];
}

interface Badge {
  type: string;
  level: number;
  icon: string;
  title: string;
}
```

### 2. フィードアイテムレイアウト
```
┌─────────────────────────────────────────────┐
│ 💎🔥⭐ ユーザー名    [2分前]    ❤️ 12 🔥 5   │
│ 目標「毎日ランニング」を達成しました！        │
│ 「今日も頑張った！明日も続けるぞ💪」         │
└─────────────────────────────────────────────┘
```

### 3. バッジ表示システム
- **優先順位**: サブスク > 特別 > 達成バッジ
- **最大表示**: 3個まで
- **ツールチップ**: ホバーで詳細表示

---

## 🚀 実装フェーズ

### Phase 2A: 基盤実装 (1週間)
1. **データベースセットアップ**
   - テーブル作成・マイグレーション
   - RLS設定
   - リアルタイム有効化

2. **基本コンポーネント**
   - `RealtimeFeed` コンポーネント
   - `FeedItem` コンポーネント
   - `UserBadge` コンポーネント

### Phase 2B: リアルタイム機能 (1週間)
1. **Supabaseリアルタイム統合**
   - リアルタイム購読機能
   - 新着フィード自動表示
   - 接続状態管理

2. **バッジシステム**
   - バッジ判定ロジック
   - 自動バッジ付与
   - バッジ表示優先順位

### Phase 2C: インタラクション (1週間)
1. **リアクション機能**
   - いいね・絵文字リアクション
   - リアルタイム更新
   - アニメーション効果

2. **UX改善**
   - スムーズスクロール
   - 無限ローディング
   - パフォーマンス最適化

---

## 📊 パフォーマンス考慮事項

### 1. リアルタイム最適化
- **接続プール管理**: 同時接続数制限
- **フィード制限**: 表示件数50件まで
- **バッチ処理**: 統計更新の効率化

### 2. データベース最適化
- **インデックス**: created_at, user_id
- **パーティション**: 月別でデータ分割
- **アーカイブ**: 古いデータの自動削除

### 3. フロントエンド最適化
- **仮想スクロール**: 大量データ対応
- **メモ化**: 不要な再レンダリング防止
- **遅延ローディング**: 画像・アニメーション

---

## 🎮 Twitchライクな体験

### 1. リアルタイム感
- **即座の更新**: 達成と同時にフィード表示
- **ライブ感**: 「現在◯人がアクティブ」表示
- **ストリーミング**: 途切れない更新フロー

### 2. コミュニティ感
- **バッジの意味**: 貢献度が一目でわかる
- **達成の共有**: みんなで喜びを分かち合う
- **継続の励み**: 他の人の頑張りに刺激される

### 3. エンゲージメント
- **リアクション**: 簡単な応援機能
- **ストリーク表示**: 連続達成の可視化
- **マイルストーン**: 特別な達成の強調

---

## 🔧 技術スタック

### バックエンド
- **Supabase Realtime**: WebSocket接続
- **PostgreSQL**: 高性能データベース
- **Row Level Security**: セキュリティ

### フロントエンド
- **React Query**: リアルタイムデータ管理
- **Framer Motion**: スムーズアニメーション
- **React Virtualized**: 高性能スクロール

### リアルタイム
- **Supabase Channels**: WebSocket通信
- **Presence**: ユーザーオンライン状態
- **Broadcast**: リアルタイムイベント

---

## 🎯 期待される効果

### 1. ユーザー体験向上
- **モチベーション**: 他の人の達成に刺激される
- **コミュニティ感**: 一人じゃない感覚
- **継続性**: 楽しみながら続けられる

### 2. エンゲージメント向上
- **利用時間**: フィード閲覧で滞在時間延長
- **継続率**: コミュニティの力で継続促進
- **口コミ**: 楽しい体験をシェア

### 3. データ価値
- **ユーザー行動**: リアルタイム分析
- **コミュニティ健康度**: 活発さの測定
- **改善指標**: エンゲージメント追跡

---

**🎉 WeStepコミュニティの心臓部となる機能です！**

この実装により、単なる目標管理アプリから、**真のコミュニティプラットフォーム**へと進化します。Twitchの成功体験を参考に、達成の喜びを共有する温かいコミュニティを築きましょう！ 