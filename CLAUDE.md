# Luu Pix プロジェクトガイド

このドキュメントは、Luu Pixプロジェクトの開発に関する技術的な詳細と、Claude AIアシスタントが理解すべき重要な情報をまとめています。

## プロジェクト概要

Luu PixはAI搭載の写真アルバムアプリケーションです。主な特徴：
- 自然言語による写真検索（例：「夏の海の写真」「笑顔の家族写真」）
- Multimodal Embeddingsによる画像とテキストの統一的なベクトル化
- 通常のアルバム管理機能

## 技術スタック

### フロントエンド
- **Next.js 14** (App Router使用)
- **shadcn/ui** - UIコンポーネントライブラリ
- **Tailwind CSS** - スタイリング
- **TypeScript** - 型安全性

### バックエンド (Supabase完結型アーキテクチャ)
- **Supabase Auth** - 認証・認可
- **PostgreSQL** - メインデータベース
- **pgvector** - ベクトル類似検索拡張機能
- **Edge Functions** - サーバーレスAPI
- **Supabase Storage** - 画像・動画ストレージ
- **Supabase Realtime** - リアルタイム同期
- **pg_cron** - 定期実行ジョブ
- **Queue (PostgreSQL)** - ジョブキュー管理

### AI/ML
- **Google Cloud Vertex AI** - Multimodal Embeddings API
- **ベクトル次元数**: 1408次元（画像・テキスト共通）

### インフラ・ツール
- **Turborepo** - モノレポ管理
- **pnpm** - パッケージマネージャー

## ディレクトリ構造

```
luu-pix/
├── apps/
│   ├── web/                    # Next.js Webアプリケーション
│   │   ├── app/                # App Router
│   │   ├── components/         # アプリ固有コンポーネント
│   │   ├── lib/               # ユーティリティ関数
│   │   └── public/            # 静的ファイル
│   │
│   └── mobile/                # 将来のモバイルアプリ用（予定）
│
├── packages/
│   ├── ui/                    # 共通UIコンポーネント
│   │   ├── components/        # 再利用可能なコンポーネント
│   │   └── styles/           # 共通スタイル
│   │
│   ├── database/             # データベース関連
│   │   ├── schema/          # Supabaseスキーマ
│   │   └── migrations/      # マイグレーション
│   │
│   └── supabase/            # Supabase関連
│       ├── functions/       # Edge Functions
│       ├── types/          # 型定義
│       └── lib/            # Supabaseクライアント
│
├── turbo.json              # Turborepo設定
├── package.json            # ルートパッケージ
└── pnpm-workspace.yaml     # pnpmワークスペース設定
```

## 主要機能の実装方針

### 1. 写真アップロード
- Supabase Storageを使用
- アップロード時にベクトル化処理をトリガー
- サムネイル自動生成

### 2. AI画像ベクトル化
- Vertex AI Multimodal Embeddingsで画像を1408次元のベクトルに変換
- アップロード時に非同期で実行
- ベクトルはpgvectorに保存

### 3. 自然言語検索
- テキストクエリもMultimodal Embeddingsでベクトル化
- 画像とテキストが同一ベクトル空間に存在するため高精度な検索が可能
- pgvectorでコサイン類似度による検索
- 日本語対応

### 4. アルバム管理
- ユーザー別のプライベートアルバム
- 共有アルバム機能
- 権限管理（閲覧/編集）

### 5. 認証・認可
- Supabase Auth による認証
- メール/パスワード、OAuth対応
- Row Level Security (RLS) による細かいアクセス制御
- マルチテナンシー対応

### 環境設定

#### Google Cloud 認証
1. サービスアカウントキーを作成
2. Vertex AI API を有効化
3. 必要な権限:
   - `aiplatform.endpoints.predict`
   - `aiplatform.models.predict`

#### Supabase pgvector セットアップ
```sql
-- Supabase ダッシュボードの SQL エディタで実行
CREATE EXTENSION IF NOT EXISTS vector;
```

## 開発時の注意事項

### Supabase開発フロー
1. ローカル開発は `supabase start` でローカル環境を起動
2. マイグレーションは `supabase migration new` で作成
3. Edge Functionsは `supabase functions serve` でローカルテスト
4. 型定義は `supabase gen types typescript` で自動生成

### コーディング規約
- TypeScriptの厳格モード使用
- ESLint/Prettierの設定に従う
- コンポーネントは関数コンポーネント使用
- カスタムフックでロジックを分離

### パフォーマンス
- 画像の遅延読み込み実装
- 無限スクロール対応
- WebP形式への自動変換

### セキュリティ
- 画像URLは署名付きURL使用
- RLSポリシーで適切なアクセス制御
- APIキーは環境変数で管理

## よく使うコマンド

```bash
# 開発サーバー起動
pnpm dev

# 特定のアプリのみ起動
pnpm dev --filter=web

# ビルド
pnpm build

# リント実行
pnpm lint

# リント自動修正
pnpm lint:fix

# 型チェック
pnpm typecheck

# テスト実行
pnpm test

# 依存関係の更新
pnpm update -r

# 新しいUIコンポーネント追加（shadcn/ui）
pnpm dlx shadcn-ui@latest add [component-name]

# Supabase pgvector拡張の確認
npx supabase db push

# Supabaseローカル開発環境
supabase start

# Edge Functions開発
supabase functions serve

# 型定義生成
supabase gen types typescript --local > packages/supabase/types/database.types.ts

# マイグレーション作成
supabase migration new add_photo_tables

# Edge Function作成
supabase functions new upload-handler
```

## AI機能の実装戦略

### Vertex AI Multimodal Embeddings
Google Cloud の Vertex AI Multimodal Embeddings API を使用して、画像とテキストを同一のベクトル空間に埋め込みます。

#### 主な特徴
- **統一ベクトル空間**: 画像とテキストを同じ1408次元のベクトル空間に埋め込み
- **高精度な検索**: 「海で遊ぶ子供」などの自然言語でピンポイントに画像を検索可能
- **動画対応**: 将来的に動画の検索にも対応可能
- **多言語対応**: 日本語を含む多言語でのクエリに対応

### pgvector によるベクトル保存と検索
Supabase の pgvector 拡張機能を使用して、効率的なベクトル類似検索を実現します。

#### ベクトル化フロー
1. **画像アップロード時**:
   - Supabase Storage に画像を保存
   - Database Webhook で Edge Function をトリガー
   - ジョブキューにベクトル化タスクを登録
   - Edge Function から Vertex AI API を呼び出し
   - 画像をベクトル化（1408次元）
   - pgvector に保存

2. **検索時**:
   - Edge Function でテキストクエリを受信
   - Vertex AI でベクトル化
   - pgvector でコサイン類似度による近傍検索
   - 類似度スコアでランキングして結果を返却

### Supabase Edge Functions アーキテクチャ

#### 主要なEdge Functions
1. **upload-handler**: 画像アップロード処理
2. **embedding-processor**: ベクトル化処理（キューから実行）
3. **search-api**: 検索API
4. **album-api**: アルバム管理API
5. **auth-webhook**: 認証関連の処理

#### 重い処理のキュー管理
```sql
-- ジョブキューテーブル
CREATE TABLE job_queue (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  job_type TEXT NOT NULL,
  payload JSONB NOT NULL,
  status TEXT DEFAULT 'pending',
  attempts INT DEFAULT 0,
  max_attempts INT DEFAULT 3,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  started_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  error_message TEXT
);

-- pg_cronでキュー処理を定期実行
SELECT cron.schedule(
  'process-embedding-queue',
  '*/1 * * * *', -- 1分ごと
  $$
    SELECT http.post(
      'https://your-project.supabase.co/functions/v1/embedding-processor',
      headers => jsonb_build_object(
        'Authorization', 'Bearer YOUR_SERVICE_ROLE_KEY',
        'Content-Type', 'application/json'
      )
    )
  $$
);
```

### データベース設計

```sql
-- pgvector 拡張を有効化
CREATE EXTENSION IF NOT EXISTS vector;

-- 写真テーブル
CREATE TABLE photos (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users(id) NOT NULL,
  file_path TEXT NOT NULL,
  file_size BIGINT,
  mime_type TEXT,
  width INTEGER,
  height INTEGER,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Row Level Security
ALTER TABLE photos ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own photos" ON photos
  FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own photos" ON photos
  FOR INSERT WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own photos" ON photos
  FOR UPDATE USING (auth.uid() = user_id);

CREATE POLICY "Users can delete own photos" ON photos
  FOR DELETE USING (auth.uid() = user_id);

-- ベクトル埋め込みテーブル
CREATE TABLE photo_embeddings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  photo_id UUID REFERENCES photos(id) ON DELETE CASCADE,
  embedding vector(1408) NOT NULL,
  model_version TEXT DEFAULT 'multimodalembedding@001',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ベクトル検索用インデックス
CREATE INDEX photo_embeddings_embedding_idx ON photo_embeddings 
USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 100);
```

## トラブルシューティング

### よくある問題
1. **Supabase接続エラー**: 環境変数を確認
2. **型エラー**: `pnpm typecheck`で詳細確認
3. **ビルドエラー**: `pnpm clean`後に再ビルド
4. **Vertex AI認証エラー**: サービスアカウントキーのパスと権限を確認
5. **pgvectorエラー**: 拡張機能が有効化されているか確認
6. **ベクトル次元エラー**: embedding vector(1408)の次元数を確認

## 今後の拡張予定

- モバイルアプリ対応
- 動画検索対応（Vertex AI Multimodal Embeddingsは動画にも対応）
- 顔認識によるグループ化
- 自動アルバム生成（イベント検出）
- バックアップ・エクスポート機能
- リアルタイム類似画像提案
- 多言語検索の強化