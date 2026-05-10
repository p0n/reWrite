# Component Dependencies

## 依存関係マトリクス

| コンポーネント | 依存先 | 通信方式 |
|--------------|--------|---------|
| Frontend (Next.js) | Backend API (FastAPI) | HTTPS REST API |
| Frontend (Next.js) | Google OAuth | ブラウザリダイレクト |
| Backend API | PromptBuilderService | 内部関数呼び出し |
| Backend API | ImageGeneratorService | 内部関数呼び出し |
| Backend API | ImageStorageService | 内部関数呼び出し |
| Backend API | GooglePhotosService | 内部関数呼び出し |
| Backend API | DatabaseService | 内部関数呼び出し |
| PromptBuilderService | Amazon Bedrock (Claude) | AWS SDK |
| ImageGeneratorService | Amazon Bedrock (SDXL) | AWS SDK |
| ImageStorageService | AWS S3 | AWS SDK |
| GooglePhotosService | Google Photos API | HTTPS REST API |
| DatabaseService | Amazon DynamoDB | AWS SDK |

---

## データフロー図

### フロー1: ログイン
```
User
 |
 | (1) Googleログインボタンクリック
 v
Frontend
 |
 | (2) Google OAuth リダイレクト
 v
Google OAuth
 |
 | (3) 認可コード返却
 v
Frontend (/auth/callback)
 |
 | (4) POST /auth/google { code }
 v
Backend API (AuthService)
 |
 | (5) Googleトークン交換 + ユーザー情報取得
 v
Google OAuth API
 |
 | (6) upsert_user()
 v
DynamoDB
 |
 | (7) JWT返却
 v
Frontend (JWTをCookieに保存)
```

### フロー2: 画像生成
```
User
 |
 | (1) ロケーション入力 + 活動カテゴリ選択 + 「生成」クリック
 v
Frontend
 |
 | (2) POST /generate { locationId, count }
 v
Backend API (ImageGenerationService)
 |
 | (3) build_prompt(location, activity)
 v
PromptBuilderService
 |
 | (4) Bedrock InvokeModel (Claude)
 v
Amazon Bedrock (Claude)
 |
 | (5) { prompt, negative_prompt } 返却
 v
PromptBuilderService → ImageGenerationService
 |
 | (6) generate(prompt, negative_prompt, count)
 v
ImageGeneratorService
 |
 | (7) Bedrock InvokeModel (SDXL)
 v
Amazon Bedrock (SDXL)
 |
 | (8) 画像バイナリ返却
 v
ImageGeneratorService → ImageGenerationService
 |
 | (9) upload_temp(image_bytes)
 v
ImageStorageService → S3
 |
 | (10) get_presigned_url()
 v
S3 → Frontend (プレビュー表示)
```

### フロー3: Google Photoへの保存
```
User
 |
 | (1) 「Google Photoに保存」クリック
 v
Frontend
 |
 | (2) POST /photos/save { imageId }
 v
Backend API (PhotoSaveService)
 |
 | (3) download(s3_key)
 v
ImageStorageService → S3
 |
 | (4) get_or_create_album()
 v
GooglePhotosService → Google Photos API
 |
 | (5) upload_image(image_bytes, description)
 v
GooglePhotosService → Google Photos API
 |
 | (6) { photoUrl } 返却
 v
Frontend (保存完了表示)
```

---

## 外部依存サービス一覧

| サービス | 用途 | 認証方式 | 注意事項 |
|---------|------|---------|---------|
| Google OAuth 2.0 | ユーザー認証 | OAuth 2.0 認可コードフロー | GCPプロジェクト設定が必要 |
| Google Photos Library API | 画像保存 | OAuth 2.0 アクセストークン | スコープ: `photoslibrary.appendonly` |
| Amazon Bedrock (Claude) | プロンプト生成 | AWS IAM | リージョン: us-east-1 推奨 |
| Amazon Bedrock (SDXL) | 画像生成 | AWS IAM | モデルアクセス申請が必要 |
| AWS DynamoDB | データ永続化 | AWS IAM | オンデマンドキャパシティ |
| AWS S3 | 画像一時保存 | AWS IAM | TTL設定でコスト最適化 |
