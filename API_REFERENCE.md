# API Reference

WeStepアプリケーションのAPI仕様書です。

## 認証要件

すべてのAPIエンドポイント（`/api/auth/*`を除く）は認証が必要です。

- **認証方式**: Supabase JWT Token
- **ヘッダー**: `Authorization: Bearer <token>`
- **認証失敗**: `401 Unauthorized`

## 共通レスポンス形式

### 成功レスポンス
```json
{
  "success": true,
  "data": {},
  "message": "Optional success message"
}
```

### エラーレスポンス
```json
{
  "error": "Error message",
  "details": "Optional detailed error information",
  "code": "ERROR_CODE"
}
```

## エンドポイント一覧

### 認証関連

#### `POST /api/auth/callback`
Supabase認証コールバック処理

**パラメータ**: URL searchParams経由
- `code`: 認証コード
- `next`: リダイレクト先URL（オプション）

**レスポンス**: リダイレクト

---

### 目標管理

#### `GET /api/goals`
ユーザーの目標を取得

**クエリパラメータ**:
- `date`: 特定日の目標 (YYYY-MM-DD形式)
- `month`: 特定月の目標 (YYYY-MM形式)

**レスポンス**:
```json
{
  "goals": [
    {
      "id": "string",
      "title": "string",
      "description": "string",
      "date": "2023-12-01T00:00:00.000Z",
      "is_completed": false,
      "completed_at": null,
      "created_at": "2023-12-01T10:00:00.000Z",
      "updated_at": "2023-12-01T10:00:00.000Z"
    }
  ]
}
```

#### `POST /api/goals`
新しい目標を作成

**リクエストボディ**:
```json
{
  "title": "string (1-100文字)",
  "description": "string (オプション)",
  "date": "YYYY-MM-DD"
}
```

**レスポンス**: 201 Created
```json
{
  "goal": {
    "id": "string",
    "title": "string",
    "description": "string",
    "date": "2023-12-01T00:00:00.000Z",
    "is_completed": false,
    "completed_at": null,
    "created_at": "2023-12-01T10:00:00.000Z",
    "updated_at": "2023-12-01T10:00:00.000Z"
  }
}
```

#### `PUT /api/goals?id={goalId}`
目標を更新

**クエリパラメータ**:
- `id`: 目標ID (必須)

**リクエストボディ**:
```json
{
  "title": "string (オプション)",
  "description": "string (オプション)", 
  "is_completed": "boolean (オプション)"
}
```

**レスポンス**:
```json
{
  "goal": {
    "id": "string",
    "title": "string",
    "description": "string",
    "date": "2023-12-01T00:00:00.000Z",
    "is_completed": true,
    "completed_at": "2023-12-01T15:30:00.000Z",
    "created_at": "2023-12-01T10:00:00.000Z",
    "updated_at": "2023-12-01T15:30:00.000Z"
  }
}
```

#### `GET /api/goals/history`
目標履歴を取得（ページネーション対応）

**クエリパラメータ**:
- `page`: ページ番号 (デフォルト: 1)
- `limit`: 1ページあたりの件数 (デフォルト: 20)

**レスポンス**:
```json
{
  "goals": [],
  "totalCount": 100,
  "currentPage": 1,
  "totalPages": 5,
  "hasNextPage": true,
  "hasPreviousPage": false
}
```

---

### 分析・統計

#### `GET /api/analytics`
ユーザーの分析データを取得

**レスポンス**:
```json
{
  "analytics": {
    "totalGoals": 50,
    "completedGoals": 35,
    "completionRate": 70,
    "currentStreak": 5,
    "maxStreak": 12,
    "monthlyData": [
      {
        "month": "2023-12",
        "goals": 10,
        "completed": 8
      }
    ]
  }
}
```

#### `GET /api/stats`
グローバル統計を取得

**レスポンス**:
```json
{
  "globalStats": {
    "totalUsers": 1000,
    "totalGoals": 25000,
    "totalCompletedGoals": 18000,
    "averageCompletionRate": 72
  }
}
```

---

### ランキング

#### `GET /api/rankings/streak`
連続達成ランキングを取得

**クエリパラメータ**:
- `limit`: 取得件数 (デフォルト: 20)

**レスポンス**:
```json
{
  "rankings": [
    {
      "rank": 1,
      "medal": "🥇",
      "user": "ユーザー名",
      "userId": "string",
      "streak": 25,
      "maxStreak": 30,
      "completionRate": 85.5,
      "totalGoals": 100,
      "completedGoals": 85,
      "lastCompletedDate": "2023-12-01T00:00:00.000Z"
    }
  ],
  "stats": {
    "totalUsers": 50,
    "topStreak": 25,
    "averageStreak": 8
  }
}
```

#### `GET /api/rankings/monthly`
月間ランキングを取得

**クエリパラメータ**:
- `month`: 対象月 (YYYY-MM形式、デフォルト: 当月)
- `limit`: 取得件数 (デフォルト: 20)

**レスポンス**: `/api/rankings/streak`と同様の形式

---

### フィード・コミュニティ

#### `GET /api/achievements-feed`
達成フィードを取得

**クエリパラメータ**:
- `limit`: 取得件数 (デフォルト: 50)

**レスポンス**:
```json
{
  "success": true,
  "data": [
    {
      "id": "string",
      "user_id": "string",
      "goal_title": "今日の目標",
      "goal_category": "健康",
      "achievement_type": "completed",
      "streak_count": 5,
      "message": "目標達成しました！",
      "created_at": "2023-12-01T15:30:00.000Z",
      "reaction_count": 3,
      "is_pinned": false,
      "user": {
        "id": "string",
        "name": "ユーザー名",
        "avatar_url": "https://example.com/avatar.jpg"
      },
      "user_badges": [],
      "reactions": [
        {
          "id": "string",
          "user_id": "string",
          "reaction_type": "👏"
        }
      ]
    }
  ]
}
```

#### `POST /api/feed-reactions`
フィードにリアクションを追加

**リクエストボディ**:
```json
{
  "feed_id": "string",
  "reaction_type": "👏"
}
```

#### `GET /api/community-messages`
コミュニティメッセージを取得

**クエリパラメータ**:
- `limit`: 取得件数 (デフォルト: 50)

---

### プロフィール

#### `GET /api/profile`
ユーザープロフィールを取得

**レスポンス**:
```json
{
  "profile": {
    "id": "string",
    "email": "user@example.com",
    "name": "ユーザー名",
    "avatar_url": "https://example.com/avatar.jpg",
    "privacy_settings": {
      "showInFeed": true,
      "publicProfile": false
    },
    "created_at": "2023-01-01T00:00:00.000Z"
  }
}
```

#### `PUT /api/profile`
プロフィールを更新

**リクエストボディ**:
```json
{
  "name": "新しい名前",
  "privacy_settings": {
    "showInFeed": true,
    "publicProfile": false
  }
}
```

#### `GET /api/public/profile/{user_id}`
公開プロフィールを取得

**パラメータ**:
- `user_id`: ユーザーID

---

### カレンダー

#### `GET /api/calendar/annual`
年間カレンダーデータを取得

**クエリパラメータ**:
- `year`: 取得年 (デフォルト: 当年)

**レスポンス**:
```json
{
  "calendar": {
    "2023": {
      "1": {
        "1": {
          "hasGoal": true,
          "isCompleted": true,
          "goalTitle": "新年の目標"
        }
      }
    }
  }
}
```

---

### サブスクリプション

#### `GET /api/subscription`
サブスクリプション状態を取得

#### `POST /api/subscription`
サブスクリプションを管理

---

## エラーハンドリング

### HTTPステータスコード

- `200 OK`: 成功
- `201 Created`: リソース作成成功
- `400 Bad Request`: バリデーションエラー
- `401 Unauthorized`: 認証エラー
- `404 Not Found`: リソースが見つからない
- `500 Internal Server Error`: サーバーエラー
- `503 Service Unavailable`: データベース接続エラー

### 一般的なエラーコード

- `AUTH_REQUIRED`: 認証が必要
- `VALIDATION_ERROR`: バリデーションエラー
- `DB_CONNECTION_ERROR`: データベース接続エラー
- `GOALS_CREATE_ERROR`: 目標作成エラー
- `GOALS_UPDATE_ERROR`: 目標更新エラー
- `GOALS_FETCH_ERROR`: 目標取得エラー

## レート制限

現在、レート制限は実装されていませんが、将来的に追加予定です。

## 開発者向けヒント

1. **リトライ機能**: データベース接続エラー（P1001）に対して、APIは自動的にリトライを行います
2. **キャッシュ**: 分析データなどは適切にキャッシュされます
3. **リアルタイム**: 目標完了時、自動的にフィードエントリが作成されます
4. **プライバシー**: ユーザーのプライバシー設定に基づいてフィード表示が制御されます