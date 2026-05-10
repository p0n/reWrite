# Components

## システム構成概要

本システムは **分離構成**（Next.js フロントエンド + Python FastAPI バックエンド）を採用する。

```
+------------------+        +----------------------+        +-------------------+
|                  |  HTTPS |                      |  SDK   |                   |
|  Next.js         +------->+  FastAPI Backend      +------->+  Amazon Bedrock   |
|  Frontend        |        |  (Python)            |        |  (Image Gen)      |
|                  |        |                      |        +-------------------+
+------------------+        |                      |
                             |                      |  SDK   +-------------------+
                             |                      +------->+  Amazon Bedrock   |
                             |                      |        |  (Claude LLM)     |
                             |                      |        +-------------------+
                             |                      |
                             |                      |  API   +-------------------+
                             |                      +------->+  Google Photos    |
                             |                      |        |  API              |
                             |                      |        +-------------------+
                             |                      |
                             |                      |  SDK   +-------------------+
                             |                      +------->+  AWS DynamoDB     |
                             |                      |        +-------------------+
                             |                      |
                             |                      |  SDK   +-------------------+
                             |                      +------->+  AWS S3           |
                             +----------------------+        +-------------------+
```

---

## Component 1: Frontend (Next.js)

**名前**: `frontend`  
**技術**: Next.js 14+ (TypeScript), Tailwind CSS  
**責務**:
- ユーザーインターフェースの提供
- Google OAuth 2.0 ログインフロー（リダイレクト）
- ロケーション入力フォームの表示・送信
- 活動カテゴリ選択UI
- AI生成画像のプレビュー表示
- Google Photoへの保存操作のトリガー
- JWTトークンの管理（ローカルストレージ or Cookie）
- バックエンドAPIとのHTTPS通信

**主要画面**:
- `/` — ランディング / ログインページ
- `/dashboard` — メインダッシュボード（ロケーション入力・画像生成）
- `/auth/callback` — Google OAuth コールバック処理

---

## Component 2: Backend API (FastAPI)

**名前**: `backend`  
**技術**: Python 3.11+, FastAPI, Pydantic  
**責務**:
- REST API エンドポイントの提供
- Google OAuth 2.0 認証フローの処理（認可コード交換、トークン管理）
- JWTトークンの発行・検証
- ロケーション情報の受け取りとDynamoDBへの保存
- Claude（Amazon Bedrock）を使ったプロンプト生成の呼び出し
- Amazon Bedrock（画像生成）の呼び出し
- 生成画像のS3一時保存
- Google Photos APIへの画像アップロード
- ユーザーデータのアクセス制御

**主要エンドポイント**:
- `POST /auth/google` — Google OAuth認証
- `GET /auth/me` — 現在のユーザー情報取得
- `POST /locations` — ロケーション登録
- `GET /locations` — ロケーション履歴取得
- `POST /generate` — 画像生成リクエスト
- `POST /photos/save` — Google Photoへの保存

---

## Component 3: Prompt Builder (Claude on Bedrock)

**名前**: `prompt_builder`  
**技術**: Amazon Bedrock (Claude 3 Sonnet/Haiku)  
**責務**:
- ユーザー入力（ロケーション名 + 活動カテゴリ）を受け取る
- そのロケーションでリア充っぽい活動をしているシーンの詳細な英語プロンプトを生成
- 画像生成に最適化されたプロンプト形式で出力
- ネガティブプロンプトも生成（低品質要素の除外）

**入力例**: `{ "location": "京都 金閣寺", "activity": "観光" }`  
**出力例**: `{ "prompt": "A cheerful group of friends taking photos in front of Kinkaku-ji temple in Kyoto, golden pavilion reflecting in the pond, sunny day, vibrant colors, candid lifestyle photography", "negative_prompt": "blurry, low quality, text, watermark" }`

---

## Component 4: Image Generator (Bedrock)

**名前**: `image_generator`  
**技術**: Amazon Bedrock (Stable Diffusion XL / Titan Image Generator G1)  
**責務**:
- Prompt Builderが生成したプロンプトを受け取る
- 高品質な画像を生成（1〜5枚）
- 生成画像をbase64またはバイナリで返却

---

## Component 5: Storage (S3)

**名前**: `image_storage`  
**技術**: AWS S3  
**責務**:
- 生成画像の一時保存（Google Photo保存前のバッファ）
- 一時ファイルの自動削除（TTL設定）
- フロントエンドへのプレビュー用署名付きURL発行

---

## Component 6: Database (DynamoDB)

**名前**: `database`  
**技術**: Amazon DynamoDB  
**責務**:
- ユーザープロフィールの保存（Google ユーザーID、メール、アクセストークン）
- ロケーション履歴の保存（ユーザーIDをパーティションキー）

**テーブル設計（概要）**:
- `users` テーブル: PK=`userId`
- `locations` テーブル: PK=`userId`, SK=`createdAt`

---

## Component 7: Google Photos Integration

**名前**: `google_photos`  
**技術**: Google Photos Library API v1  
**責務**:
- S3から画像を取得してGoogle Photos APIへアップロード
- 専用アルバム（「My Moments」）の作成・管理
- アップロード時のメタデータ設定（ロケーション情報を説明文に含める）
- ユーザーの既存写真からEXIF位置情報を読み取る（`photoslibrary.readonly`スコープ）

---

## Component 8: Location Suggestion (Geocoding)

**名前**: `location_suggestion`  
**技術**: Google Maps Geocoding API または AWS Location Service（逆ジオコーディング）  
**責務**:
- Google Photoから取得した緯度・経度を人間が読めるロケーション名に変換（逆ジオコーディング）
- ロケーション候補リストの生成・重複排除
- ユーザーへの提案リスト返却
