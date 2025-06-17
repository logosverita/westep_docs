# LINE認証実装ガイド

SupabaseはLINE認証を公式サポートしていないため、カスタム実装が必要です。

## 🚨 重要な注意点

**Supabaseは現在LINE認証を公式にサポートしていません。**
- 2024年現在、多くのリクエストがありますが未実装
- カスタム実装が必要

## 🛠️ 実装方法

### 方法1: LINE Login API + カスタム実装 (推奨)

#### 1. LINE Developer Console設定

1. **LINE Developersにアクセス**
   - [LINE Developers](https://developers.line.biz/)にログイン
   - 新しいプロバイダーを作成（既存があれば選択）

2. **チャンネル作成**
   ```
   チャンネルタイプ: LINE Login
   チャンネル名: WeStep
   チャンネル説明: WeStep LINE Login
   ```

3. **チャンネル設定**
   ```
   Callback URL: 
   - https://westep.app/auth/line/callback
   - http://localhost:3000/auth/line/callback (開発用)
   
   スコープ:
   - profile (基本情報)
   - openid (OpenID Connect)
   - email (メールアドレス)
   ```

#### 2. 環境変数設定

```env
# .env.local
NEXT_PUBLIC_LINE_CHANNEL_ID=your_line_channel_id
LINE_CHANNEL_SECRET=your_line_channel_secret
```

#### 3. LINE認証フロー実装

```typescript
// app/auth/line/callback/page.tsx
'use client'

import { useEffect } from 'react'
import { useRouter, useSearchParams } from 'next/navigation'
import { supabase } from '@/lib/supabase'

export default function LineCallback() {
  const router = useRouter()
  const searchParams = useSearchParams()
  
  useEffect(() => {
    const handleLineCallback = async () => {
      const code = searchParams.get('code')
      const state = searchParams.get('state')
      const storedState = sessionStorage.getItem('line_oauth_state')
      
      // CSRF保護
      if (state !== storedState) {
        router.push('/auth?error=不正なリクエストです')
        return
      }
      
      if (!code) {
        router.push('/auth?error=認証に失敗しました')
        return
      }
      
      try {
        // 1. LINEアクセストークン取得
        const tokenResponse = await fetch('/api/auth/line/token', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ code })
        })
        
        const { access_token } = await tokenResponse.json()
        
        // 2. LINEユーザー情報取得
        const userResponse = await fetch('/api/auth/line/user', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ access_token })
        })
        
        const lineUser = await userResponse.json()
        
        // 3. Supabaseに登録/ログイン
        const { data, error } = await supabase.auth.signInWithPassword({
          email: lineUser.email || `${lineUser.userId}@line.local`,
          password: lineUser.userId // 一時的なパスワード
        })
        
        if (error && error.message.includes('Invalid login credentials')) {
          // ユーザーが存在しない場合は作成
          const { error: signUpError } = await supabase.auth.signUp({
            email: lineUser.email || `${lineUser.userId}@line.local`,
            password: lineUser.userId,
            options: {
              data: {
                name: lineUser.displayName,
                avatar_url: lineUser.pictureUrl,
                provider: 'line',
                line_user_id: lineUser.userId
              }
            }
          })
          
          if (!signUpError) {
            router.push('/')
          } else {
            router.push('/auth?error=アカウント作成に失敗しました')
          }
        } else if (!error) {
          router.push('/')
        } else {
          router.push('/auth?error=ログインに失敗しました')
        }
        
      } catch (error) {
        console.error('LINE認証エラー:', error)
        router.push('/auth?error=認証処理中にエラーが発生しました')
      }
    }
    
    handleLineCallback()
  }, [router, searchParams])
  
  return (
    <div className="min-h-screen flex items-center justify-center">
      <div className="text-center">
        <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-[#00B900] mx-auto"></div>
        <p className="mt-4">LINE認証処理中...</p>
      </div>
    </div>
  )
}
```

#### 4. APIエンドポイント実装

```typescript
// app/api/auth/line/token/route.ts
import { NextRequest, NextResponse } from 'next/server'

export async function POST(request: NextRequest) {
  const { code } = await request.json()
  
  const params = new URLSearchParams({
    grant_type: 'authorization_code',
    code,
    redirect_uri: `${process.env.NEXT_PUBLIC_APP_URL}/auth/line/callback`,
    client_id: process.env.NEXT_PUBLIC_LINE_CHANNEL_ID!,
    client_secret: process.env.LINE_CHANNEL_SECRET!
  })
  
  try {
    const response = await fetch('https://api.line.me/oauth2/v2.1/token', {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: params
    })
    
    const data = await response.json()
    return NextResponse.json(data)
  } catch (error) {
    return NextResponse.json({ error: 'トークン取得失敗' }, { status: 500 })
  }
}
```

```typescript
// app/api/auth/line/user/route.ts
import { NextRequest, NextResponse } from 'next/server'

export async function POST(request: NextRequest) {
  const { access_token } = await request.json()
  
  try {
    const response = await fetch('https://api.line.me/v2/profile', {
      headers: { Authorization: `Bearer ${access_token}` }
    })
    
    const userData = await response.json()
    return NextResponse.json(userData)
  } catch (error) {
    return NextResponse.json({ error: 'ユーザー情報取得失敗' }, { status: 500 })
  }
}
```

### 方法2: NextAuth.js併用

NextAuth.jsは LINE認証をサポートしているため、併用も可能です：

```bash
npm install next-auth
```

```typescript
// lib/auth.ts (NextAuth.js設定)
import NextAuth from 'next-auth'
import LineProvider from 'next-auth/providers/line'

export const authOptions = {
  providers: [
    LineProvider({
      clientId: process.env.LINE_CLIENT_ID!,
      clientSecret: process.env.LINE_CLIENT_SECRET!
    })
  ],
  callbacks: {
    async signIn({ user, account, profile }) {
      if (account?.provider === 'line') {
        // Supabaseにユーザー情報を同期
        // ...
      }
      return true
    }
  }
}
```

## ⚠️ 注意事項

### セキュリティ
- **CSRF保護**: stateパラメータによる検証必須
- **トークン管理**: アクセストークンの適切な管理
- **エラーハンドリング**: 適切なエラー処理

### ユーザーエクスペリエンス
- **メールアドレスなしユーザー**: LINEではメールアドレスが取得できない場合がある
- **重複アカウント**: 同じユーザーが複数の認証方法を使用する可能性

## 🎯 実装の推奨度

1. **現時点での推奨**: Google・Twitter認証の利用
2. **将来的**: Supabaseの公式対応を待つ
3. **どうしても必要**: 上記のカスタム実装

## 📞 サポート

Supabaseコミュニティでは多くの日本の開発者がLINE認証のサポートを要求しています。
- [GitHub Discussion](https://github.com/supabase/auth/issues/451)で投票可能
- 実装優先度向上に貢献できます

---

LINE認証の需要が高いため、将来的にはSupabaseで公式サポートされる可能性があります！ 