# WeStep

一日にたった一つの必ず１００％達成できる目標を決めて実行する習慣をつけ、日数を積み重ねる経験を通じて、利用者の自己重要感、目標達成意欲、自己効力感の目覚ましい発育を促し、その人個人の変容を支援する Web アプリケーション。

**WeStep** - みんなで一歩ずつ、世界を変えよう。

## コンセプト

**WeStep**は「みんなで一歩ずつ」をテーマに、個人の成長とコミュニティの力を融合させたプラットフォームです。小さな目標の達成を通じて、世界中の人々が繋がり、共に成長していく体験を提供します。

### 🌟 WeStep の価値

-   🤝 **コミュニティの力**: 一人ではない、みんなで支え合う
-   📈 **確実な成長**: 100%達成可能な目標で自信を積み重ね
-   🌍 **グローバルな繋がり**: 世界中の人々との連帯感
-   💪 **継続の習慣**: 小さな一歩の積み重ねで大きな変化

## 技術スタック

-   Next.js 14 (App Router)
-   TypeScript
-   Prisma (ORM)
-   Supabase (Database & Auth)
-   Vercel (Deployment)
-   Tailwind CSS (Styling)

## DB 要件定義

### テーブル構成

#### 1. users (ユーザー)

-   id: String (UUID, Primary Key)
-   email: String (Unique)
-   name: String
-   created_at: DateTime
-   updated_at: DateTime

#### 2. daily_goals (日次目標)

-   id: String (UUID, Primary Key)
-   user_id: String (Foreign Key → users.id)
-   title: String (目標のタイトル)
-   description: String? (詳細説明、オプション)
-   date: Date (目標設定日)
-   is_completed: Boolean (達成フラグ)
-   completed_at: DateTime? (達成日時)
-   created_at: DateTime
-   updated_at: DateTime

#### 3. achievements (実績・統計)

-   id: String (UUID, Primary Key)
-   user_id: String (Foreign Key → users.id)
-   total_days: Int (総日数)
-   consecutive_days: Int (連続達成日数)
-   max_consecutive_days: Int (最大連続達成日数)
-   completion_rate: Float (達成率)
-   last_achievement_date: Date? (最後の達成日)
-   updated_at: DateTime

#### 4. motivational_messages (モチベーションメッセージ)

-   id: String (UUID, Primary Key)
-   message: String (メッセージ内容)
-   category: String (カテゴリー: start, success, encouragement)
-   is_active: Boolean (有効フラグ)
-   created_at: DateTime

### 主要な機能要件

1. **認証機能**

    - メールアドレスでのサインアップ/ログイン
    - Supabase Auth を使用

2. **目標設定機能**

    - 1 日 1 つの目標を設定
    - 同日に複数目標は設定不可
    - 目標の編集・削除（未完了の場合のみ）

3. **達成記録機能**

    - 目標の完了/未完了を記録
    - 達成時刻の記録

4. **統計・可視化機能**

    - 連続達成日数の表示
    - 月間/年間の達成率
    - 達成カレンダー表示

5. **モチベーション機能**
    - 達成時の称賛メッセージ
    - 連続記録の表示
    - 励ましメッセージ

### ビジネスルール

1. **1 日 1 目標の原則**

    - 1 日につき 1 つの目標のみ設定可能
    - 過去の日付への目標設定は不可

2. **達成の定義**

    - ユーザーが「完了」ボタンを押した時点で達成
    - 達成は取り消し不可

3. **連続記録の計算**

    - 毎日連続で目標を達成した日数をカウント
    - 1 日でも未達成があると連続記録はリセット

4. **達成率の計算**
    - (達成した日数 / 目標を設定した総日数) × 100

---

## 🚀 クイックスタート

### 📋 必要な環境

-   Node.js 18.0 以上
-   npm または yarn
-   Git

### ⚡ 5 分で開始

```bash
# 1. リポジトリのクローン
git clone <repository-url>
cd WeStep

# 2. 依存関係のインストール
npm install

# 3. 環境変数のセットアップ
cp env.example .env.local
# .env.local を編集してSupabaseの設定を追加

# 4. データベースのセットアップ
npx prisma db push
npx prisma generate

# 5. 開発サーバーの起動
npm run dev
```

ブラウザで `http://localhost:3000` を開いて**WeStep**を体験できます！

### 📚 詳細なドキュメント

-   **[環境構築ガイド](./SETUP.md)** - 詳細なセットアップ手順
-   **[開発ガイドライン](./DEVELOPMENT.md)** - コーディング規約と実装指針
-   **[デプロイメントガイド](./DEPLOYMENT.md)** - 本番環境へのデプロイ手順
-   **[アプリ名選定](./APP_NAMING.md)** - WeStep 命名の経緯と分析

## 🌟 主要機能

### 🎯 個人機能

-   **📅 日次目標設定**: 1 日 1 つの達成可能な目標
-   **✅ 進捗トラッキング**: 連続達成日数・達成率の可視化
-   **🏆 達成記録**: カレンダー表示と統計情報
-   **💬 モチベーション**: 励ましメッセージと継続のコツ

### 🌍 コミュニティ機能

-   **📊 グローバル統計**: 全ユーザーの達成状況をリアルタイム表示
-   **🔥 ライブフィード**: 他のユーザーの達成をリアルタイムで確認
-   **🏅 ランキング**: 連続達成記録によるコミュニティランキング
-   **🌐 地域別活動**: 国・地域ごとの活動状況

### 🔧 技術的特徴

-   **⚡ リアルタイム**: WebSocket による即座の更新
-   **📱 レスポンシブ**: モバイル・デスクトップ対応
-   **🔒 セキュア**: Row Level Security による安全なデータ管理
-   **🌍 スケーラブル**: Supabase + Vercel による自動スケーリング

## 🎨 UI/UX 設計

### デザインコンセプト

**WeStep**の UI は「温かさ」と「信頼性」を表現：

-   **🔵 プライマリーカラー**: 温かみのある青（信頼と成長）
-   **🟢 セカンダリーカラー**: 優しい緑（希望と調和）
-   **🟠 アクセントカラー**: 明るいオレンジ（エネルギーと行動）

### ユーザー体験

1. **簡潔性**: 必要最小限のインターフェース
2. **即座の反応**: リアルタイム更新による満足感
3. **成長の可視化**: 進捗が一目で分かるデザイン
4. **コミュニティ感**: 他者との繋がりを感じる設計

## 🔧 技術アーキテクチャ

### フロントエンド

-   **Next.js 14**: App Router、Server Components 対応
-   **TypeScript**: 型安全な開発環境
-   **Tailwind CSS**: ユーティリティファーストのスタイリング
-   **Zustand**: 軽量な状態管理

### バックエンド

-   **Supabase**: PostgreSQL + リアルタイム機能
-   **Prisma**: 型安全な ORM
-   **NextAuth.js**: セキュアな認証システム
-   **WebSocket**: リアルタイム通信

### インフラ

-   **Vercel**: 自動デプロイと最適化
-   **CDN**: グローバルな高速配信
-   **監視**: Sentry + Vercel Analytics

## 📊 プロジェクトの進捗

### ✅ 完了済み

-   [x] **プロジェクト設計**: アーキテクチャ・DB 設計完了
-   [x] **基本実装**: Next.js + TypeScript + Tailwind CSS
-   [x] **データベース**: Prisma スキーマ定義完了
-   [x] **UI 設計**: メインダッシュボードのデザイン
-   [x] **ドキュメント**: 包括的な開発・デプロイガイド

### 🚧 実装中

-   [ ] **認証システム**: Supabase Auth 統合
-   [ ] **データベース操作**: CRUD 機能実装
-   [ ] **リアルタイム機能**: WebSocket 通信
-   [ ] **コミュニティ機能**: グローバル統計表示

### 📅 今後の計画

-   [ ] **テスト実装**: ユニット・統合・E2E テスト
-   [ ] **パフォーマンス最適化**: Core Web Vitals 対応
-   [ ] **アクセシビリティ**: WCAG 2.1 準拠
-   [ ] **国際化**: 多言語対応
-   [ ] **本番デプロイ**: Vercel + Supabase
-   [ ] **ユーザーフィードバック**: ベータ版リリース

## 🌍 ビジョン・ミッション

### 🎯 ミッション

「小さな一歩の積み重ねを通じて、個人の成長とコミュニティの力を融合させ、世界中の人々の自己効力感向上に貢献する」

### 🌟 ビジョン

「WeStep を通じて、世界中の人々が日々の小さな目標達成を共有し、地球規模の意識上昇と個人の成長を同時に実現する未来」

### 💡 コア価値

-   **🤝 コミュニティの力**: 一人ではない、みんなで支え合う
-   **📈 確実な成長**: 100%達成可能な目標で自信を積み重ね
-   **🌍 グローバルな繋がり**: 世界中の人々との連帯感
-   **💪 継続の習慣**: 小さな一歩の積み重ねで大きな変化

## 🤝 コントリビューション

**WeStep**はオープンソースプロジェクトとして、世界中の開発者からの貢献を歓迎します：

### 貢献方法

1. **Issues**: バグ報告・機能提案
2. **Pull Requests**: コード改善・新機能実装
3. **ドキュメント**: 翻訳・内容改善
4. **テスト**: テストケース追加・品質向上

### 開発参加

```bash
# フォークしてクローン
git clone https://github.com/your-username/westep.git

# ブランチ作成
git checkout -b feature/amazing-feature

# 変更をコミット
git commit -m "Add amazing feature"

# プルリクエスト作成
git push origin feature/amazing-feature
```

## 📄 ライセンス

このプロジェクトは MIT ライセンスのもとで公開されています。詳細は[LICENSE](./LICENSE)ファイルをご確認ください。

## 📞 サポート・コミュニティ

### 🔗 リンク

-   **ウェブサイト**: https://westep.app（近日公開）
-   **ドキュメント**: [開発者ガイド](./DEVELOPMENT.md)
-   **Issues**: GitHub リポジトリの Issues タブ

### 💬 コミュニティ

-   **Discord**: WeStep 開発者コミュニティ（近日開設）
-   **Twitter**: @WeStepApp（近日開設）

---

## 🌟 一緒に世界を変えませんか？

**WeStep** - みんなで一歩ずつ、世界を変えよう！

あなたも**WeStep**コミュニティの一員として、世界中の人々の成長を支援し、地球規模の意識上昇に貢献しませんか？小さな一歩から始まる大きな変化を、一緒に創り上げていきましょう！

🚀 **今日から始める →** [クイックスタート](#🚀-クイックスタート)  
🤝 **貢献する →** [コントリビューション](#🤝-コントリビューション)  
📚 **学ぶ →** [開発ガイドライン](./DEVELOPMENT.md)

_「The journey of a thousand miles begins with one step. WeStep.」_
