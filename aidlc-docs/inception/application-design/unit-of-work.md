# Unit of Work

## ユニット分解概要

**サービス名**: re:Write

本システムは **3ユニット** に分解する。フロントエンドとバックエンドは並行開発し、インフラは独立ユニットとして管理する。

```
+------------------+     +------------------+     +------------------+
|                  |     |                  |     |                  |
|  Unit 1          |     |  Unit 2          |     |  Unit 3          |
|  Frontend        |     |  Backend         |     |  Infrastructure  |
|  (Next.js)       |     |  (FastAPI)       |     |  (AWS CDK)       |
|                  |     |                  |     |                  |
+------------------+     +------------------+     +------------------+
  並行開発可能              並行開発可能              先行作成推奨
```

**MVP対象外メモ**: Google Photo位置情報からの自動画像生成（ユーザー承認なしのバッチ処理）はPhase 2（FR-EXT-02）。
---

## Unit 1: Frontend

**名前**: `frontend`  
**ディレクトリ**: `frontend/`  
**技術スタック**: Next.js 14+ (TypeScript), Tailwind CSS  
**デプロイ先**: AWS Amplify または Vercel（PoC）

**責務**:
- ユーザーインターフェース全体
- Google OAuth 2.0 ログインフロー
- ロケーション入力・活動カテゴリ選択UI
- AI生成画像のプレビュー表示
- Google Photoへの保存操作
- バックエンドAPIとのHTTPS通信
- JWTトークン管理

**主要機能**:
- ログインページ（`/`）
- ダッシュボード（`/dashboard`）— ロケーション入力・画像生成・保存
- OAuthコールバック処理（`/auth/callback`）
- Google Photoからのロケーション提案表示（`/dashboard` 内）

**コード構成**:
```
frontend/
  src/
    app/                    # Next.js App Router
      page.tsx              # ランディング/ログインページ
      dashboard/
        page.tsx            # メインダッシュボード
      auth/
        callback/
          page.tsx          # OAuthコールバック
    components/             # UIコンポーネント
      LoginButton.tsx
      LocationForm.tsx
      LocationSuggestions.tsx  # Google Photoからの提案リスト（新規）
      ActivitySelector.tsx
      ImagePreview.tsx
      SaveButton.tsx
    services/               # APIクライアント
      auth.ts
      location.ts
      image.ts
      suggestions.ts        # ロケーション提案API（新規）
    lib/
      api-client.ts         # HTTPクライアント設定
  package.json
  tsconfig.json
  tailwind.config.ts
```

---

## Unit 2: Backend

**名前**: `backend`  
**ディレクトリ**: `backend/`  
**技術スタック**: Python 3.11+, FastAPI, Pydantic, boto3  
**デプロイ先**: AWS Lambda + API Gateway（または ECS Fargate）

**責務**:
- REST API エンドポイント提供
- Google OAuth 2.0 認証フロー処理
- JWT発行・検証
- ロケーション管理（DynamoDB）
- プロンプト生成（Claude on Bedrock）
- 画像生成（SDXL on Bedrock）
- 画像一時保存（S3）
- Google Photos API連携

**主要エンドポイント**:
- `POST /auth/google` — Google OAuth認証・JWT発行
- `GET /auth/me` — 現在ユーザー情報
- `POST /locations` — ロケーション登録
- `GET /locations` — ロケーション履歴
- `GET /locations/suggestions` — Google Photoからのロケーション提案（新規）
- `POST /generate` — 画像生成（非同期）
- `GET /generate/{job_id}` — 生成ステータス確認
- `POST /photos/save` — Google Photoへの保存

**コード構成**:
```
backend/
  app/
    main.py
    routers/
      auth.py
      locations.py          # /locations/suggestions エンドポイント追加
      generation.py
      photos.py
    services/
      auth_service.py
      location_service.py
      location_suggestion_service.py  # Google Photo位置情報取得（新規）
      image_generation_service.py
      photo_save_service.py
      prompt_builder_service.py
      image_generator_service.py
      image_storage_service.py
      google_photos_service.py        # list_photos_with_location() 追加
      database_service.py
    models/
      user.py
      location.py
      generation.py
      suggestion.py         # ロケーション提案モデル（新規）
    config.py
  requirements.txt
  Dockerfile
```

---

## Unit 3: Infrastructure

**名前**: `infrastructure`  
**ディレクトリ**: `infrastructure/`  
**技術スタック**: AWS CDK (Python) または Terraform  
**管理リソース**:
- Amazon DynamoDB テーブル（users, locations）
- AWS S3 バケット（画像一時保存）
- AWS Lambda 関数（バックエンドデプロイ先）
- Amazon API Gateway（REST API）
- AWS IAM ロール・ポリシー（Bedrock, DynamoDB, S3アクセス）
- AWS Systems Manager Parameter Store（シークレット管理）

**コード構成**:
```
infrastructure/
  app.py                    # CDKアプリエントリポイント
  stacks/
    database_stack.py       # DynamoDBテーブル定義
    storage_stack.py        # S3バケット定義
    api_stack.py            # Lambda + API Gateway定義
    iam_stack.py            # IAMロール・ポリシー定義
  requirements.txt
  cdk.json
```

**開発順序**: インフラユニットは他の2ユニットより先に基本リソース（DynamoDB, S3）を作成することを推奨。Lambda/API Gatewayはバックエンド完成後にデプロイ。

---

## 開発戦略

| ユニット | 開発順序 | 並行可否 | 備考 |
|---------|---------|---------|------|
| Unit 3: Infrastructure | 先行（基本リソース） | — | DynamoDB/S3を先に作成 |
| Unit 1: Frontend | 並行 | Unit 2と並行可 | モックAPIで開発可能 |
| Unit 2: Backend | 並行 | Unit 1と並行可 | ローカル環境で開発可能 |
| Unit 3: Infrastructure | 後続（デプロイ） | Unit 2完成後 | Lambda/API Gatewayデプロイ |
