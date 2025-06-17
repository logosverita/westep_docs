# WeStep 開発ガイドライン

**WeStep** の開発における実装指針、コーディングスタンダード、アーキテクチャ設計について説明します。

## 🏗️ アーキテクチャ概要

### システム構成

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Backend       │    │   Database      │
│   (Next.js)     │◄──►│   (API Routes)  │◄──►│   (Supabase)    │
│                 │    │                 │    │                 │
│ - React/TS      │    │ - Server Actions│    │ - PostgreSQL    │
│ - Tailwind CSS  │    │ - Prisma ORM    │    │ - Real-time     │
│ - Zustand       │    │ - WebSocket     │    │ - Auth          │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### 主要技術選定理由

| 技術             | 理由                                             |
| ---------------- | ------------------------------------------------ |
| **Next.js 14**   | App Router、Server Components、最新の React 機能 |
| **TypeScript**   | 型安全性、開発体験向上、大規模開発対応           |
| **Prisma**       | 型安全な ORM、優れた DX、スキーマ駆動開発        |
| **Supabase**     | リアルタイム機能、認証、スケーラビリティ         |
| **Tailwind CSS** | 高速開発、一貫性のあるデザイン                   |

## 📁 ディレクトリ構造詳細

```
src/
├── app/                     # Next.js App Router
│   ├── (auth)/             # 認証関連ページ群
│   │   ├── login/          # ログインページ
│   │   └── register/       # 登録ページ
│   ├── dashboard/          # ダッシュボード
│   │   ├── personal/       # 個人ダッシュボード
│   │   ├── community/      # コミュニティ表示
│   │   └── settings/       # 設定ページ
│   ├── api/               # API Routes
│   │   ├── goals/         # 目標関連API
│   │   ├── achievements/  # 実績関連API
│   │   └── websocket/     # WebSocket処理
│   ├── layout.tsx         # ルートレイアウト
│   ├── page.tsx          # ホームページ
│   ├── loading.tsx       # ローディングUI
│   ├── error.tsx         # エラーUI
│   └── globals.css       # グローバルスタイル
├── components/            # Reactコンポーネント
│   ├── ui/               # 基本UIコンポーネント
│   │   ├── Button.tsx    # ボタンコンポーネント
│   │   ├── Input.tsx     # 入力フィールド
│   │   ├── Modal.tsx     # モーダル
│   │   └── index.ts      # エクスポート定義
│   ├── layout/           # レイアウトコンポーネント
│   │   ├── Header.tsx    # ヘッダー
│   │   ├── Sidebar.tsx   # サイドバー
│   │   └── Footer.tsx    # フッター
│   ├── features/         # 機能別コンポーネント
│   │   ├── goals/        # 目標関連
│   │   ├── achievements/ # 実績関連
│   │   └── community/    # コミュニティ関連
│   └── providers/        # プロバイダー
├── lib/                  # ユーティリティ・設定
│   ├── supabase.ts      # Supabase クライアント
│   ├── prisma.ts        # Prisma クライアント
│   ├── websocket.ts     # WebSocket クライアント
│   ├── utils.ts         # 汎用ユーティリティ
│   └── validations.ts   # バリデーション
├── hooks/               # カスタムフック
│   ├── useAuth.ts       # 認証フック
│   ├── useGoals.ts      # 目標管理フック
│   └── useRealtime.ts   # リアルタイム機能フック
├── stores/              # 状態管理 (Zustand)
│   ├── authStore.ts     # 認証状態
│   ├── goalStore.ts     # 目標状態
│   └── communityStore.ts # コミュニティ状態
├── types/               # TypeScript型定義
│   ├── database.ts      # データベース型
│   ├── api.ts          # API レスポンス型
│   └── global.ts       # グローバル型
└── styles/              # スタイル関連
    ├── components.css   # コンポーネント固有スタイル
    └── animations.css   # アニメーション定義
```

## 🎨 デザインシステム

### カラーパレット

```css
/* WeStep カラーパレット */
:root {
	/* プライマリーカラー - 温かみのある青（信頼と成長） */
	--primary-50: #eff6ff;
	--primary-100: #dbeafe;
	--primary-500: #3b82f6;
	--primary-600: #2563eb;
	--primary-900: #1e3a8a;

	/* セカンダリーカラー - 優しい緑（希望と調和） */
	--secondary-50: #f0fdf4;
	--secondary-100: #dcfce7;
	--secondary-500: #22c55e;
	--secondary-600: #16a34a;

	/* アクセントカラー - 明るいオレンジ（エネルギーと行動） */
	--accent-50: #fff7ed;
	--accent-100: #ffedd5;
	--accent-500: #f97316;
	--accent-600: #ea580c;
}
```

### タイポグラフィ

```css
/* フォントサイズ階層 */
.text-hero {
	@apply text-4xl md:text-6xl font-bold;
}
.text-h1 {
	@apply text-3xl md:text-4xl font-bold;
}
.text-h2 {
	@apply text-2xl md:text-3xl font-semibold;
}
.text-h3 {
	@apply text-xl md:text-2xl font-medium;
}
.text-body {
	@apply text-base leading-relaxed;
}
.text-small {
	@apply text-sm text-gray-600;
}
```

### コンポーネントクラス

```css
/* 基本コンポーネント */
.btn-primary {
	@apply bg-primary-600 hover:bg-primary-700 text-white px-6 py-3 rounded-lg font-medium transition-colors duration-200;
}

.btn-secondary {
	@apply bg-gray-100 hover:bg-gray-200 text-gray-900 px-6 py-3 rounded-lg font-medium transition-colors duration-200;
}

.card {
	@apply bg-white rounded-xl shadow-sm border border-gray-200 p-6;
}

.input-field {
	@apply w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-primary-500 focus:border-transparent;
}
```

## 🔧 コーディング規約

### TypeScript

```typescript
// ✅ 良い例
interface DailyGoal {
	id: string;
	title: string;
	description?: string;
	date: Date;
	isCompleted: boolean;
	completedAt?: Date;
	userId: string;
}

// 型ガードの使用
function isDailyGoal(obj: unknown): obj is DailyGoal {
	return typeof obj === "object" && obj !== null && "id" in obj;
}

// ❌ 悪い例
const goal: any = {};
```

### React コンポーネント

```typescript
// ✅ 良い例 - 関数コンポーネント + TypeScript
interface ButtonProps {
	children: React.ReactNode;
	onClick?: () => void;
	variant?: "primary" | "secondary";
	disabled?: boolean;
}

export function Button({
	children,
	onClick,
	variant = "primary",
	disabled = false,
}: ButtonProps) {
	return (
		<button
			onClick={onClick}
			disabled={disabled}
			className={`btn-${variant} ${disabled ? "opacity-50" : ""}`}
		>
			{children}
		</button>
	);
}

// ❌ 悪い例
const Button = (props: any) => <button {...props} />;
```

### 状態管理パターン

```typescript
// ✅ Zustand ストアの例
interface GoalStore {
	goals: DailyGoal[];
	currentGoal: DailyGoal | null;
	isLoading: boolean;

	// アクション
	setGoals: (goals: DailyGoal[]) => void;
	addGoal: (goal: Omit<DailyGoal, "id">) => void;
	completeGoal: (goalId: string) => void;
}

export const useGoalStore = create<GoalStore>((set, get) => ({
	goals: [],
	currentGoal: null,
	isLoading: false,

	setGoals: (goals) => set({ goals }),

	addGoal: async (goalData) => {
		set({ isLoading: true });
		try {
			const newGoal = await createGoal(goalData);
			set((state) => ({
				goals: [...state.goals, newGoal],
				isLoading: false,
			}));
		} catch (error) {
			set({ isLoading: false });
			throw error;
		}
	},

	completeGoal: (goalId) =>
		set((state) => ({
			goals: state.goals.map((goal) =>
				goal.id === goalId
					? { ...goal, isCompleted: true, completedAt: new Date() }
					: goal
			),
		})),
}));
```

## 🗄️ データベース設計

### Prisma スキーマパターン

```prisma
// ✅ 良い例 - 関係性とインデックスの適切な定義
model DailyGoal {
  id          String   @id @default(cuid())
  title       String   @db.VarChar(255)
  description String?  @db.Text
  date        DateTime @db.Date
  isCompleted Boolean  @default(false)
  completedAt DateTime?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  // 関係性
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  userId      String

  // インデックス
  @@unique([userId, date], name: "user_daily_goal")
  @@index([userId, createdAt])
  @@index([date, isCompleted])
}
```

### データアクセスパターン

```typescript
// ✅ 良い例 - Prisma を使用したデータアクセス
export async function getTodayGoal(userId: string): Promise<DailyGoal | null> {
	const today = new Date();
	today.setHours(0, 0, 0, 0);

	return await prisma.dailyGoal.findUnique({
		where: {
			user_daily_goal: {
				userId,
				date: today,
			},
		},
	});
}

export async function createDailyGoal(
	data: Omit<DailyGoal, "id" | "createdAt" | "updatedAt">
): Promise<DailyGoal> {
	return await prisma.dailyGoal.create({
		data,
		include: {
			user: {
				select: {
					id: true,
					name: true,
				},
			},
		},
	});
}
```

## 🌐 API 設計

### Server Actions パターン

```typescript
"use server";

import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";

export async function createGoalAction(prevState: any, formData: FormData) {
	// バリデーション
	const title = formData.get("title") as string;
	if (!title || title.length < 1) {
		return { error: "タイトルは必須です" };
	}

	try {
		// データベース操作
		const goal = await createDailyGoal({
			title,
			description: formData.get("description") as string,
			date: new Date(),
			userId: "current-user-id", // 認証から取得
		});

		// キャッシュの更新
		revalidatePath("/");

		return { success: true, goal };
	} catch (error) {
		return { error: "目標の作成に失敗しました" };
	}
}
```

### WebSocket ハンドラー

```typescript
// リアルタイム機能の実装
export function setupWebSocketHandler() {
	const wss = new WebSocketServer({ port: 8080 });

	wss.on("connection", (ws) => {
		ws.on("message", async (data) => {
			const message = JSON.parse(data.toString());

			switch (message.type) {
				case "GOAL_COMPLETED":
					// 他のユーザーに達成通知をブロードキャスト
					broadcastAchievement(message.payload);
					break;

				case "JOIN_COMMUNITY":
					// コミュニティチャンネルに参加
					joinCommunityChannel(ws, message.payload.userId);
					break;
			}
		});
	});
}
```

## 🧪 テスト戦略

### ユニットテスト

```typescript
// ✅ コンポーネントテストの例
import { render, screen, fireEvent } from "@testing-library/react";
import { Button } from "./Button";

describe("Button Component", () => {
	it("renders with correct text", () => {
		render(<Button>Click me</Button>);
		expect(screen.getByRole("button")).toHaveTextContent("Click me");
	});

	it("calls onClick when clicked", () => {
		const mockClick = jest.fn();
		render(<Button onClick={mockClick}>Click me</Button>);

		fireEvent.click(screen.getByRole("button"));
		expect(mockClick).toHaveBeenCalledTimes(1);
	});
});
```

### E2E テスト

```typescript
// ✅ Playwright を使用したE2Eテスト
import { test, expect } from "@playwright/test";

test("user can create and complete a daily goal", async ({ page }) => {
	await page.goto("/");

	// 目標作成
	await page.fill('[data-testid="goal-title"]', "本を10ページ読む");

	// 目標が表示されることを確認
	await expect(page.locator('[data-testid="current-goal"]')).toContainText(
		"本を10ページ読む"
	);

	// 目標完了
	await page.click('[data-testid="complete-goal-button"]');

	// 完了状態が表示されることを確認
	await expect(page.locator('[data-testid="goal-status"]')).toContainText(
		"達成"
	);
});
```

## 🚀 パフォーマンス最適化

### 画像最適化

```typescript
// ✅ Next.js Image コンポーネントの使用
import Image from "next/image";

export function AvatarImage({ src, alt }: { src: string; alt: string }) {
	return (
		<Image
			src={src}
			alt={alt}
			width={64}
			height={64}
			className="rounded-full"
			priority={false}
			placeholder="blur"
			blurDataURL="data:image/svg+xml;base64,..."
		/>
	);
}
```

### コード分割

```typescript
// ✅ 動的インポートの使用
import dynamic from "next/dynamic";

const CommunityDashboard = dynamic(() => import("./CommunityDashboard"), {
	loading: () => <div>Loading community...</div>,
	ssr: false,
});
```

## 📊 モニタリング・ログ

### エラーハンドリング

```typescript
// ✅ エラー境界の実装
export class ErrorBoundary extends React.Component {
	constructor(props: any) {
		super(props);
		this.state = { hasError: false };
	}

	static getDerivedStateFromError(error: Error) {
		return { hasError: true };
	}

	componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
		// エラーログ送信
		logErrorToService(error, errorInfo);
	}

	render() {
		if (this.state.hasError) {
			return <ErrorFallback />;
		}

		return this.props.children;
	}
}
```

## 🔐 セキュリティ

### 認証・認可

```typescript
// ✅ サーバーサイドでの認証チェック
export async function requireAuth() {
	const session = await getServerSession();

	if (!session) {
		redirect("/auth");
	}

	return session;
}

// ✅ API ルートでの認証
export async function POST(request: Request) {
	const session = await requireAuth();

	// 処理を続行
}
```

## 📚 コードレビューガイドライン

### チェックポイント

1. **型安全性** - TypeScript の型は適切に定義されているか
2. **パフォーマンス** - 不要な再レンダリングがないか
3. **アクセシビリティ** - ARIA 属性、キーボード操作対応
4. **セキュリティ** - 入力値検証、XSS 対策
5. **テスタビリティ** - テストしやすい構造になっているか

---

このガイドラインに従って、**WeStep** の高品質な開発を進めていきましょう！ 🚀
