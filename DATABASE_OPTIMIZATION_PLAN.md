# データベーススキーマ最適化計画

## 調査結果サマリー

WeStepアプリケーションのコードベース全体を調査した結果、以下の不要なフィールドとテーブルを特定しました。

## 🔴 削除対象項目

### 1. usersテーブルの不要フィールド

```sql
-- 削除対象フィールド
ALTER TABLE users DROP COLUMN IF EXISTS user_name;               -- 完全に未使用
ALTER TABLE users DROP COLUMN IF EXISTS subscription_end_date;   -- subscriptionsテーブルと重複  
ALTER TABLE users DROP COLUMN IF EXISTS subscription_start_date; -- subscriptionsテーブルと重複
ALTER TABLE users DROP COLUMN IF EXISTS subscription_status;     -- subscriptionsテーブルと重複
```

**削除理由:**
- `user_name`: コードベース全体で使用されていない
- `subscription_*`: `subscriptions`テーブルで管理されており完全に重複

### 2. global_statsテーブルの未使用フィールド

```sql
-- 削除対象フィールド
ALTER TABLE global_stats DROP COLUMN IF EXISTS active_realtime_users; -- 未実装
ALTER TABLE global_stats DROP COLUMN IF EXISTS total_feed_items;       -- 未使用
ALTER TABLE global_stats DROP COLUMN IF EXISTS total_reactions;        -- 未使用
```

**削除理由:**
- 実際の統計計算では動的に計算されており、これらのフィールドは更新・参照されていない

### 3. 完全に未使用のテーブル

```sql
-- 削除対象テーブル
DROP TABLE IF EXISTS community_stats CASCADE;      -- achievementsテーブルと重複、未使用
DROP TABLE IF EXISTS motivational_messages CASCADE; -- 機能未実装
```

**削除理由:**
- `community_stats`: `achievements`テーブルで統計情報を管理しており、完全に重複
- `motivational_messages`: アプリケーション機能として実装されていない

## 📊 期待される効果

### ストレージ削減
- **usersテーブル**: 約25-30%のストレージ削減
- **global_statsテーブル**: 約30%のストレージ削減  
- **全体**: 20-40%のストレージ使用量削減

### パフォーマンス向上
- クエリの高速化（不要なフィールドの除外）
- インデックスサイズの削減
- メモリ使用量の最適化

### メンテナンス性向上
- データ重複の解消
- スキーマの簡素化
- コード理解の向上

## 🛠️ 実行手順

### フェーズ1: 安全な削除（影響度：低）
1. `users`テーブルの重複フィールド削除
2. `global_stats`テーブルの未使用フィールド削除

### フェーズ2: テーブル削除（影響度：中）
1. `motivational_messages`テーブル削除
2. `community_stats`テーブル削除

### フェーズ3: 検証・最適化
1. アプリケーション動作確認
2. パフォーマンステスト
3. Prismaスキーマ更新

## ⚠️ 注意事項

### 実行前の確認
- [ ] データベースのフルバックアップ作成
- [ ] 開発環境での事前テスト
- [ ] 関連するアプリケーションコードの確認

### リスク軽減策
- 段階的な実行（フェーズ分け）
- 各ステップでの動作確認
- ロールバック計画の準備

## 🚀 実行スクリプト

最適化スクリプトを以下に用意：
- `scripts/optimize-database-schema.ts` - Prisma経由での実行
- `scripts/optimize-schema-supabase.ts` - Supabase経由での実行

## 📋 実行状況

- [ ] フェーズ1: usersテーブル最適化
- [ ] フェーズ1: global_statsテーブル最適化  
- [ ] フェーズ2: 未使用テーブル削除
- [ ] フェーズ3: 検証・最適化完了

## 🔍 検証項目

### 機能テスト
- [ ] ユーザー登録・ログイン
- [ ] プロフィール編集・保存
- [ ] 目標作成・完了
- [ ] 統計表示
- [ ] バッジシステム
- [ ] リアクション機能

### パフォーマンステスト
- [ ] API応答時間計測
- [ ] データベースクエリ最適化確認
- [ ] ストレージ使用量比較

---

**最終更新日:** 2025-06-16  
**作成者:** Database Optimization Team