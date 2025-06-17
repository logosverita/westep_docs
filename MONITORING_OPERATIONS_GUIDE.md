# WeStepモニタリング・運用ガイド

## 概要

pgbouncer設定修正後の予防策として実装された包括的な監視・自動復旧システムの運用ガイドです。

## 🛠️ 実装済みシステム

### 1. ヘルスチェックシステム
- **ファイル**: `scripts/health-check.ts`
- **機能**: システム全体の健全性を包括的にチェック
- **監視項目**:
  - データベース基本接続
  - 接続プール状態（pgbouncer対応）
  - 重要テーブルアクセス
  - データ整合性
  - リアルタイム機能
  - 認証システム
  - APIエンドポイント

### 2. 自動診断システム
- **ファイル**: `scripts/auto-diagnose.ts`
- **機能**: 詳細な診断とレポート生成
- **診断項目**:
  - データベース接続パフォーマンス
  - テーブル権限状態
  - データ整合性問題
  - リアルタイム機能状態

### 3. 自動復旧システム
- **ファイル**: `scripts/auto-repair.ts`, `scripts/auto-recovery.ts`
- **機能**: 問題の自動検出と修復
- **復旧アクション**:
  - データベース接続復旧
  - 接続プール最適化
  - テーブル権限修復
  - 認証システム修復
  - データクリーニング
  - パフォーマンス最適化

### 4. 継続的モニタリング
- **ファイル**: `scripts/continuous-monitoring.ts`
- **機能**: 24/7リアルタイム監視
- **監視機能**:
  - 定期ヘルスチェック
  - パフォーマンス監視
  - 自動アラート
  - 自動復旧トリガー

### 5. パフォーマンス監視
- **ファイル**: `lib/performance-monitor.ts`
- **機能**: フロントエンド・バックエンドパフォーマンス監視
- **メトリクス**:
  - 応答時間
  - エラー率
  - スループット
  - メモリ使用量

## 🚀 運用開始手順

### 1. 基本ヘルスチェック実行
```bash
# システム全体の健康状態確認
npx tsx scripts/health-check.ts

# 詳細診断実行
npx tsx scripts/auto-diagnose.ts
```

### 2. 継続的モニタリング開始
```bash
# 標準設定で開始（5分間隔）
npx tsx scripts/continuous-monitoring.ts

# 高頻度監視（1分間隔）
npx tsx scripts/continuous-monitoring.ts --fast

# 自動復旧無効で開始
npx tsx scripts/continuous-monitoring.ts --no-recovery
```

### 3. 自動復旧テスト
```bash
# ヘルスチェック結果に基づく自動復旧
npx tsx scripts/auto-recovery.ts --with-health-check

# 手動復旧実行
npx tsx scripts/auto-repair.ts
```

## 📊 日常運用タスク

### 毎日のチェック項目

#### 朝のヘルスチェック（9:00）
```bash
# 夜間の状態確認
npx tsx scripts/health-check.ts
tail -n 50 logs/monitoring-alerts-$(date +%Y-%m-%d).log
```

#### 昼のパフォーマンスチェック（12:00）
```bash
# パフォーマンス状況確認
npx tsx scripts/auto-diagnose.ts
cat logs/diagnosis-report-$(date +%Y-%m-%d).md
```

#### 夕方の状態確認（18:00）
```bash
# 一日の総合状況確認
tail -n 100 logs/monitoring-events-*.json | jq '.[] | select(.severity == "error" or .severity == "critical")'
```

### 週次メンテナンス（日曜日）

#### データベース最適化
```bash
# パフォーマンス最適化実行
npx tsx scripts/auto-repair.ts optimize_performance

# 統計情報更新
npx tsx scripts/optimize-database-schema.ts
```

#### ログローテーション
```bash
# 古いログファイルの圧縮・保管
cd logs
tar -czf logs-$(date +%Y-%m-%d).tar.gz *.log *.json *.md
rm -f *.log *.json *.md
```

### 月次レビュー

#### トレンド分析
```bash
# 月間パフォーマンストレンド分析
python scripts/analyze-monthly-trends.py  # 要実装
```

#### 設定見直し
- 監視間隔の最適化
- アラート閾値の調整
- 自動復旧設定の見直し

## 🚨 アラート対応手順

### Critical（重要）アラート

#### データベース接続失敗
```bash
# 即座実行
npx tsx scripts/auto-recovery.ts
npx tsx scripts/check-database-status.ts

# Supabaseダッシュボード確認
# プロジェクト再開が必要な場合があります
```

#### 接続プール枯渇
```bash
# pgbouncer設定確認
npx tsx scripts/auto-repair.ts restore_connection_pool

# 必要に応じて接続プール設定調整
```

### Warning（警告）アラート

#### パフォーマンス低下
```bash
# パフォーマンス最適化実行
npx tsx scripts/auto-repair.ts optimize_performance

# 詳細分析
npx tsx scripts/auto-diagnose.ts
```

#### データ整合性問題
```bash
# データクリーニング実行
npx tsx scripts/auto-repair.ts fix_data_integrity

# 問題の詳細確認
npx tsx scripts/auto-diagnose.ts
```

## 📈 パフォーマンス監視

### リアルタイム監視

```typescript
// フロントエンドでの監視開始
import { performanceMonitor } from '@/lib/performance-monitor'

// 監視開始（5分間隔）
performanceMonitor.startMonitoring(300000)

// 最新メトリクス取得
const metrics = performanceMonitor.getLatestMetrics()
console.log(performanceMonitor.generateSummary())
```

### API ヘルスチェックエンドポイント

```bash
# アプリケーション経由でのヘルスチェック
curl http://localhost:3000/api/health

# システム外部からの監視
curl https://your-app.vercel.app/api/health
```

## 🔧 トラブルシューティング

### よくある問題と対処法

#### 1. 継続的モニタリングが停止する
```bash
# プロセス確認
ps aux | grep continuous-monitoring

# 再起動
nohup npx tsx scripts/continuous-monitoring.ts > monitoring.log 2>&1 &
```

#### 2. 自動復旧が機能しない
```bash
# 復旧履歴確認
ls -la logs/recovery-*.md

# 手動復旧実行
npx tsx scripts/auto-repair.ts
```

#### 3. ヘルスチェックが失敗する
```bash
# 環境変数確認
echo $NEXT_PUBLIC_SUPABASE_URL
echo $SUPABASE_SERVICE_ROLE_KEY

# 基本接続テスト
npx tsx scripts/test-connection.ts
```

### デバッグコマンド

```bash
# 詳細ログ出力でヘルスチェック
DEBUG=true npx tsx scripts/health-check.ts

# 診断結果の詳細表示
npx tsx scripts/auto-diagnose.ts | jq '.'

# モニタリングイベント確認
tail -f logs/monitoring-events-*.json | jq '.'
```

## 📋 運用チェックリスト

### 毎日
- [ ] 朝のヘルスチェック実行
- [ ] アラートログ確認
- [ ] パフォーマンスメトリクス確認
- [ ] 継続的モニタリング動作確認

### 毎週
- [ ] データベース最適化実行
- [ ] ログファイル整理
- [ ] 週間レポート確認
- [ ] 設定パラメータ見直し

### 毎月
- [ ] パフォーマンストレンド分析
- [ ] アラート閾値調整
- [ ] 自動復旧履歴レビュー
- [ ] システム改善点検討

## 📞 エスカレーション

### Level 1: 自動対応
- 自動復旧システムによる対応
- 標準的な問題パターンの解決

### Level 2: 手動対応
- 複雑な問題の手動診断
- カスタム修復スクリプト実行
- 設定調整

### Level 3: 外部支援
- Supabaseサポートへの連絡
- 専門技術チームへのエスカレーション
- 根本的システム見直し

## 🎯 成功指標（SLA）

### 可用性目標
- **システム稼働率**: 99.9%
- **応答時間**: 平均 < 2秒
- **エラー率**: < 1%

### 復旧目標
- **検出時間**: < 5分
- **復旧時間**: < 15分
- **自動復旧成功率**: > 80%

### 監視目標
- **アラート精度**: > 95%
- **誤検知率**: < 5%
- **監視カバレッジ**: 100%

---

**最終更新**: 2025-06-16  
**責任者**: pane4 - データベース復旧専門チーム  
**次回見直し**: 月次または重大インシデント発生時

## 付録

### A. コマンドリファレンス
```bash
# 全システム診断
npx tsx scripts/auto-diagnose.ts

# ヘルスチェック
npx tsx scripts/health-check.ts

# 自動復旧
npx tsx scripts/auto-recovery.ts

# 継続監視
npx tsx scripts/continuous-monitoring.ts

# パフォーマンス最適化
npx tsx scripts/auto-repair.ts optimize_performance
```

### B. 設定ファイル例
```json
{
  "monitoring": {
    "intervals": {
      "health": 300000,
      "performance": 60000
    },
    "thresholds": {
      "responseTime": 2000,
      "errorRate": 0.05
    },
    "autoRecovery": {
      "enabled": true,
      "maxAttemptsPerHour": 3
    }
  }
}
```

### C. アラート通知設定
```bash
# Webhook設定例（Slack通知）
export WEBHOOK_URL="https://hooks.slack.com/services/..."
export ALERT_CHANNEL="#westep-alerts"
```