# Luu Pix - AI搭載写真アルバムアプリ

Luu Pixは、AIによる自然言語検索機能を持つ次世代の写真アルバムアプリケーションです。写真を整理・管理する通常のアルバム機能に加え、Google Vertex AI のMultimodal Embeddingsを使用して写真とテキストを同一のベクトル空間にマッピングすることで、「夏休みの海での写真」「笑顔の家族写真」といった自然な言葉で写真を検索できます。

## 主な機能

- 📸 **写真管理**: アルバムの作成、写真のアップロード、整理
- 🔍 **AI検索**: 自然言語による直感的な写真検索
- 🤖 **自動ベクトル化**: アップロード時に自動的に検索用ベクトルを生成
- 👥 **共有機能**: 家族や友人とのアルバム共有
- 📱 **レスポンシブデザイン**: モバイル・デスクトップ両対応

## 技術スタック

- **フロントエンド**: Next.js 14 (App Router)
- **UI コンポーネント**: shadcn/ui
- **スタイリング**: Tailwind CSS
- **バックエンド**: Supabase
  - 認証: Supabase Auth
  - データベース: PostgreSQL + pgvector拡張
  - API: Edge Functions
  - ストレージ: Supabase Storage
  - リアルタイム: Supabase Realtime
  - キュー処理: pg_cron + Edge Functions
- **AI/ML**: Google Cloud Vertex AI (Multimodal Embeddings)
- **モノレポ管理**: Turborepo

## プロジェクト構成

このプロジェクトはTurborepoを使用したモノレポ構成です。

```
luu-pix/
├── apps/
│   ├── web/          # Next.js Webアプリケーション
│   └── mobile/       # 将来的なモバイルアプリ用
├── packages/
│   ├── ui/           # 共通UIコンポーネント
│   ├── database/     # データベーススキーマとマイグレーション
│   └── ai/           # AI関連のロジック
└── turbo.json        # Turborepo設定
```

## セットアップ

### 必要な環境

- Node.js 18.0以上
- pnpm 8.0以上

### インストール手順

```bash
# リポジトリのクローン
git clone https://github.com/your-username/luu-pix.git
cd luu-pix

# 依存関係のインストール
pnpm install

# 環境変数の設定
cp .env.example .env.local
# .env.localを編集してSupabaseの認証情報を設定

# 開発サーバーの起動
pnpm dev
```

## 開発コマンド

```bash
# 開発サーバーの起動
pnpm dev

# ビルド
pnpm build

# リント
pnpm lint

# 型チェック
pnpm typecheck

# テスト
pnpm test
```

## 環境変数

以下の環境変数を`.env.local`に設定してください：

```
# Supabase
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_supabase_service_key

# Google Cloud / Vertex AI
GOOGLE_APPLICATION_CREDENTIALS=path/to/service-account-key.json
GCP_PROJECT_ID=your-gcp-project-id
GCP_LOCATION=asia-northeast1
```

## ライセンス

このプロジェクトはMITライセンスで公開されています。