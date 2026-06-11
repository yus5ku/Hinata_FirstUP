# Hinata_FirstApp

> AIがTodoからスケジュールを自動生成する、フルスタックタスク管理アプリ

![Next.js](https://img.shields.io/badge/Next.js-16-black?logo=next.js)
![TypeScript](https://img.shields.io/badge/TypeScript-5-blue?logo=typescript)
![Supabase](https://img.shields.io/badge/Supabase-green?logo=supabase)
![Prisma](https://img.shields.io/badge/Prisma-ORM-2D3748?logo=prisma)
![Tailwind CSS](https://img.shields.io/badge/TailwindCSS-4-38B2AC?logo=tailwind-css)

---

## 概要

**hinata_firstUP** は、Todoの管理とAIによるスケジュール自動生成を組み合わせたWebアプリケーション。  
起床・就寝・食事・学校・アルバイトなどの生活リズムをあらかじめ設定しておくと、登録済みのTodoをその空き時間に自動で割り当てた **時間軸のスケジュール表（タイムライン）**をボタン1つで生成
**フルスタック実装力**を示すことを目的に開発。

---

## 主な機能

| 機能               | 説明                                                                      |
| ------------------ | ------------------------------------------------------------------------- |
| 認証               | Supabase Authによるメール/パスワード認証                                  |
| Todo管理           | タスクの作成・編集・削除・ステータス管理                                  |
| 生活リズム設定     | 起床・就寝・食事・学校・アルバイトの時間帯をユーザーが自由に設定          |
| AIタイムライン生成 | ボタンを押すとTodoと生活リズムをもとにClaude APIが1日のタイムラインを生成 |
| タイムライン表示   | `9:00 勉強` `10:30 休憩` のような時間軸形式でスケジュールを表示           |

---

## AIスケジュール生成の仕組み

```
① ユーザーが設定画面で生活リズムを登録
   （例）起床 7:00 / 学校 9:00〜17:00 / アルバイト 18:00〜21:00 / 就寝 24:00

② Todoを登録（タイトル・優先度など）

③「スケジュール生成」ボタンを押す

④ AI APIが空き時間にTodoを割り当てた
   タイムラインを生成して表示

   例）
   07:00  起床・朝食
   09:00  学校
   17:00  移動
   17:30  英単語の復習（Todo）
   18:00  アルバイト
   21:00  帰宅・夕食
   22:00  レポート作成（Todo）
   24:00  就寝
```

---

## 技術スタック

### フロントエンド

| 技術                                          | バージョン | 用途                         |
| --------------------------------------------- | ---------- | ---------------------------- |
| [Next.js](https://nextjs.org/)                | 16         | フレームワーク（App Router） |
| [TypeScript](https://www.typescriptlang.org/) | 5          | 型安全な開発                 |
| [Tailwind CSS](https://tailwindcss.com/)      | 4          | スタイリング                 |
| [shadcn/ui](https://ui.shadcn.com/)           | latest     | UIコンポーネント             |

### バックエンド / インフラ

| 技術                                     | バージョン | 用途                             |
| ---------------------------------------- | ---------- | -------------------------------- |
| [Supabase](https://supabase.com/)        | -          | 認証・データベース（PostgreSQL） |
| [Prisma](https://www.prisma.io/)         | 7          | ORM                              |
| [Claude API](https://www.anthropic.com/) | -          | AIスケジュール生成               |

---

## ディレクトリ構成

```
hinata_firstUP/
├── src/
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/
│   │   │   │   └── page.tsx          # ログインページ
│   │   │   └── signup/
│   │   │       └── page.tsx          # サインアップページ
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx            # 共通レイアウト（サイドバー・ヘッダー）
│   │   │   ├── todos/
│   │   │   │   └── page.tsx          # Todo一覧ページ
│   │   │   └── schedule/
│   │   │       └── page.tsx          # AIスケジュールページ
│   │   ├── api/
│   │   │   ├── todos/
│   │   │   │   └── route.ts          # Todo CRUD API
│   │   │   └── schedule/
│   │   │       └── generate/
│   │   │           └── route.ts      # AIスケジュール生成API
│   │   ├── layout.tsx                # ルートレイアウト
│   │   └── globals.css
│   ├── components/
│   │   ├── todos/                    # Todoコンポーネント群
│   │   └── schedule/                 # スケジュールコンポーネント群
│   └── lib/
│       ├── prisma.ts                 # Prismaクライアント
│       └── supabase.ts               # Supabaseクライアント
├── prisma/
│   └── schema.prisma                 # DBスキーマ定義
├── middleware.ts                     # 認証ルート保護
├── .env.local                        # 環境変数（Git管理外）
└── README.md
```

---

## データベーススキーマ

```prisma
model User {
  id        String      @id @default(uuid())
  email     String      @unique
  todos     Todo[]
  settings  UserSetting?
}

model Todo {
  id          String    @id @default(uuid())
  title       String
  description String?
  priority    Priority  @default(MEDIUM)
  status      Status    @default(PENDING)
  userId      String
  user        User      @relation(fields: [userId], references: [id])
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
}

// 生活リズム設定
model UserSetting {
  id          String   @id @default(uuid())
  userId      String   @unique
  user        User     @relation(fields: [userId], references: [id])
  wakeUp      String   // 例: "07:00"
  bedTime     String   // 例: "24:00"
  meals       Json     // 例: ["07:30", "12:00", "19:00"]
  school      Json?    // 例: { start: "09:00", end: "17:00" }
  partTime    Json?    // 例: { start: "18:00", end: "21:00" }
  updatedAt   DateTime @updatedAt
}

enum Priority { LOW  MEDIUM  HIGH }
enum Status   { PENDING  IN_PROGRESS  DONE }
```

---

## セットアップ

### 必要な環境

- Node.js 18以上
- pnpm
- Supabaseアカウント

### 1. リポジトリのクローン

```bash
git clone https://github.com/<your-username>/hinata_firstUP.git
cd hinata_firstUP
```

### 2. 依存パッケージのインストール

```bash
pnpm install
```

### 3. 環境変数の設定

`.env.local` を作成し、以下を記入してください。

```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key

# Prisma (Supabase DB接続URL)
DATABASE_URL=your_supabase_database_url

# AI API
AI_API_KEY=your_ai_api_key
```

### 4. DBマイグレーション

```bash
pnpm dlx prisma migrate dev --name init
pnpm dlx prisma generate
```

### 5. 開発サーバーの起動

```bash
 pnpm dev

```

`http://localhost:3000` でアプリが起動します。

---

## API エンドポイント

| メソッド | エンドポイント           | 説明                                 |
| -------- | ------------------------ | ------------------------------------ |
| `GET`    | `/api/todos`             | Todo一覧取得                         |
| `POST`   | `/api/todos`             | Todo新規作成                         |
| `PATCH`  | `/api/todos/[id]`        | Todo更新                             |
| `DELETE` | `/api/todos/[id]`        | Todo削除                             |
| `GET`    | `/api/settings`          | 生活リズム設定取得                   |
| `POST`   | `/api/settings`          | 生活リズム設定保存                   |
| `POST`   | `/api/schedule/generate` | AIタイムライン生成（ボタントリガー） |

---

## 認証フロー

```
ユーザーアクセス
    ↓
middleware.ts でセッション確認
    ↓
未認証 → /login へリダイレクト
認証済 → ダッシュボードへ
```
