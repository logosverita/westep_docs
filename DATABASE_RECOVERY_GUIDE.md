# Supabaseデータベース復旧手順書

## 概要

WeStepアプリケーションのSupabaseデータベースに障害が発生した際の包括的な復旧手順書です。段階的な診断から完全復旧まで、システマティックなアプローチで問題を解決します。

## 🚨 緊急時対応フロー

### 1. 初期診断（5分以内）

```bash
# 基本接続確認
pnpm test:connection

# データベース状態詳細確認
npx tsx scripts/check-database-status.ts

# ヘルスチェック実行
npx tsx scripts/health-check.ts
```

### 2. 問題分類と対応優先度

#### 🔴 Critical（即座対応）
- データベース完全停止
- 認証システム障害
- 全APIエンドポイント500エラー

#### 🟡 High（1時間以内）
- 特定テーブルアクセス不可
- パフォーマンス大幅劣化
- リアルタイム機能停止

#### 🟢 Medium（24時間以内）
- 部分的データ不整合
- キャッシュ問題
- 非重要機能の軽微な不具合

## 📋 段階的復旧手順

### Phase 1: 緊急停止・安全確保

```bash
# 1. 現在の状態をスナップショット
git add -A
git commit -m "緊急時スナップショット - $(date)"

# 2. エラーログの収集
mkdir -p logs/emergency/$(date +%Y%m%d_%H%M%S)
cp dev.log dev-health-check.log goal-creation-debug.log logs/emergency/$(date +%Y%m%d_%H%M%S)/

# 3. データベースバックアップ確認（Supabaseダッシュボード）
```

### Phase 2: 問題特定・診断

```typescript
// 実行: npx tsx scripts/diagnose-system.ts
// 以下の診断を自動実行：
// - データベース接続状態
// - 各テーブルのアクセス権限
// - RLSポリシーの状態
// - インデックスの整合性
// - レプリケーション状態
```

### Phase 3: 段階的修復

#### 3.1 基本接続復旧
```sql
-- scripts/emergency-connection-fix.sql
-- Step 1: 最小限の権限設定
GRANT USAGE ON SCHEMA public TO anon, authenticated;
GRANT ALL ON ALL TABLES IN SCHEMA public TO service_role;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO authenticated;

-- Step 2: RLS一時無効化（緊急時のみ）
ALTER TABLE users DISABLE ROW LEVEL SECURITY;
ALTER TABLE daily_goals DISABLE ROW LEVEL SECURITY;
ALTER TABLE achievements DISABLE ROW LEVEL SECURITY;
```

#### 3.2 認証システム復旧
```typescript
// 実行: npx tsx scripts/fix-auth-system.ts
// - Supabase Auth設定確認
// - JWT検証設定
// - 認証フォールバック有効化
```

#### 3.3 データ整合性確認
```sql
-- scripts/data-integrity-check.sql
-- 孤立レコードの検出
SELECT u.id, u.email FROM users u 
LEFT JOIN daily_goals dg ON u.id = dg.user_id 
WHERE dg.user_id IS NULL AND u.created_at < now() - interval '7 days';

-- 重複データの検出
SELECT user_id, date, COUNT(*) as count 
FROM daily_goals 
GROUP BY user_id, date 
HAVING COUNT(*) > 1;
```

#### 3.4 パフォーマンス最適化
```sql
-- scripts/performance-optimization.sql
-- インデックス再構築
REINDEX INDEX CONCURRENTLY idx_daily_goals_user_date;
REINDEX INDEX CONCURRENTLY idx_achievements_user_created;

-- 統計情報更新
ANALYZE users;
ANALYZE daily_goals;
ANALYZE achievements;
```

### Phase 4: 機能別復旧確認

#### 4.1 認証機能
```bash
npx tsx scripts/test-auth-comprehensive.ts
```

#### 4.2 目標管理機能
```bash
npx tsx scripts/test-goals-system.ts
```

#### 4.3 リアルタイム機能
```bash
npx tsx scripts/test-realtime-features.ts
```

#### 4.4 ランキング・統計
```bash
npx tsx scripts/test-analytics-system.ts
```

## 🔧 自動化スクリプト群

### 自動診断スクリプト

```typescript
// scripts/auto-diagnose.ts
export async function runComprehensiveDiagnosis() {
  const results = {
    database: await checkDatabaseHealth(),
    permissions: await checkPermissions(),
    performance: await checkPerformance(),
    dataIntegrity: await checkDataIntegrity(),
    realtime: await checkRealtimeFeatures()
  }
  
  return generateDiagnosisReport(results)
}
```

### 自動修復スクリプト

```typescript
// scripts/auto-repair.ts
export async function runAutoRepair(issues: DiagnosisResult[]) {
  for (const issue of issues) {
    switch (issue.type) {
      case 'connection':
        await repairDatabaseConnection()
        break
      case 'permissions':
        await repairPermissions()
        break
      case 'performance':
        await optimizePerformance()
        break
    }
  }
}
```

### モニタリング・アラート

```typescript
// scripts/monitoring.ts
export async function continuousMonitoring() {
  setInterval(async () => {
    const health = await checkSystemHealth()
    
    if (health.status === 'critical') {
      await sendAlert(health)
      await runEmergencyResponse()
    }
  }, 300000) // 5分間隔
}
```

## 📊 パフォーマンス監視

### リアルタイムメトリクス

```typescript
// lib/performance-monitor.ts
export class PerformanceMonitor {
  async getMetrics() {
    return {
      responseTime: await this.getAverageResponseTime(),
      connectionCount: await this.getActiveConnections(),
      errorRate: await this.getErrorRate(),
      throughput: await this.getThroughput()
    }
  }
  
  async checkThresholds() {
    const metrics = await this.getMetrics()
    
    const alerts = []
    if (metrics.responseTime > 2000) alerts.push('slow_response')
    if (metrics.errorRate > 0.05) alerts.push('high_error_rate')
    
    return alerts
  }
}
```

### ヘルスチェックAPI

```typescript
// app/api/health/route.ts
export async function GET() {
  const health = await runHealthCheck()
  
  return NextResponse.json({
    status: health.overall,
    timestamp: new Date().toISOString(),
    services: {
      database: health.database,
      auth: health.auth,
      realtime: health.realtime
    },
    metrics: await getPerformanceMetrics()
  }, { 
    status: health.overall === 'healthy' ? 200 : 503 
  })
}
```

## 🔍 トラブルシューティング決定木

```
データベース問題発生
├── 接続できない
│   ├── Supabaseプロジェクト停止 → 手動再開
│   ├── 環境変数問題 → 設定確認・修正
│   └── ネットワーク問題 → ISP・DNS確認
├── 接続はできるがクエリ失敗
│   ├── 権限問題 → fix-permissions.sql実行
│   ├── スキーマ不一致 → マイグレーション実行
│   └── RLS問題 → ポリシー見直し
├── パフォーマンス問題
│   ├── 遅いクエリ → インデックス最適化
│   ├── 接続プール枯渇 → 接続設定調整
│   └── リソース不足 → Supabaseプラン確認
└── データ不整合
    ├── 孤立レコード → データクリーニング
    ├── 重複データ → 重複削除スクリプト
    └── 制約違反 → データ修復スクリプト
```

## 📞 エスカレーション手順

### Level 1: 自動対応（5分以内）
- 自動診断・修復スクリプト実行
- 既知の問題パターンに基づく自動復旧

### Level 2: 手動対応（30分以内）
- 手動診断・原因特定
- カスタム修復スクリプト実行
- 一時的な回避策実装

### Level 3: 外部支援（2時間以内）
- Supabaseサポートへの連絡
- 専門チームのエスカレーション
- 完全復旧計画の策定

## 🎯 復旧完了確認リスト

### 機能確認
- [ ] ユーザー認証（ログイン・ログアウト）
- [ ] 目標作成・更新・削除
- [ ] 統計・ランキング表示
- [ ] リアルタイムフィード更新
- [ ] プロフィール管理
- [ ] バッジシステム

### パフォーマンス確認
- [ ] API応答時間 < 2秒
- [ ] エラー率 < 1%
- [ ] 同時接続 > 100ユーザー
- [ ] データ一貫性100%

### 監視体制確認
- [ ] ヘルスチェックAPI正常
- [ ] アラート機能動作
- [ ] ログ収集正常
- [ ] メトリクス取得正常

## 📚 参考資料

- [Supabase公式ドキュメント](https://supabase.com/docs)
- [Prisma トラブルシューティング](https://www.prisma.io/docs/guides/other/troubleshooting)
- [WeStep TROUBLESHOOTING.md](./TROUBLESHOOTING.md)
- [緊急修復計画書](../scripts/emergency-recovery-plan.md)

---

**最終更新**: 2025-06-16
**責任者**: pane4 - データベース復旧専門チーム
**次回見直し**: 月次または重大インシデント発生時