# Component Methods

## Frontend (Next.js)

### AuthService
| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `initiateGoogleLogin()` | なし | `void` | Google OAuth認証フローを開始（リダイレクト） |
| `handleOAuthCallback(code: string)` | 認可コード | `{ token: string }` | バックエンドに認可コードを送信しJWTを取得 |
| `logout()` | なし | `void` | JWTを削除してログアウト |
| `getStoredToken()` | なし | `string \| null` | 保存済みJWTを取得 |

### LocationService
| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `submitLocation(location: string, activity: string)` | ロケーション名、活動カテゴリ | `{ locationId: string }` | バックエンドにロケーションを送信 |
| `fetchLocationHistory()` | なし | `Location[]` | ロケーション履歴を取得 |

### ImageService
| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `requestGeneration(locationId: string, count: number)` | ロケーションID、生成枚数 | `{ jobId: string }` | 画像生成をリクエスト |
| `pollGenerationStatus(jobId: string)` | ジョブID | `GenerationStatus` | 生成状況をポーリング |
| `saveToGooglePhotos(imageId: string)` | 画像ID | `{ success: boolean }` | Google Photoへの保存をリクエスト |

---

## Backend API (FastAPI)

### AuthRouter
| メソッド | エンドポイント | 入力 | 出力 | 概要 |
|---------|--------------|------|------|------|
| `google_auth(code)` | `POST /auth/google` | 認可コード | `{ access_token: string }` | Google OAuthトークン交換、JWT発行 |
| `get_current_user(token)` | `GET /auth/me` | JWT | `UserProfile` | 現在のユーザー情報を返す |

### LocationRouter
| メソッド | エンドポイント | 入力 | 出力 | 概要 |
|---------|--------------|------|------|------|
| `create_location(data, user)` | `POST /locations` | `{ name, activity }` | `Location` | ロケーションをDynamoDBに保存 |
| `list_locations(user)` | `GET /locations` | JWT | `Location[]` | ユーザーのロケーション履歴を返す |

### GenerationRouter
| メソッド | エンドポイント | 入力 | 出力 | 概要 |
|---------|--------------|------|------|------|
| `generate_images(data, user)` | `POST /generate` | `{ locationId, count }` | `{ jobId }` | 非同期で画像生成ジョブを開始 |
| `get_generation_status(jobId, user)` | `GET /generate/{jobId}` | jobId | `GenerationStatus` | 生成ジョブの状態を返す |

### PhotosRouter
| メソッド | エンドポイント | 入力 | 出力 | 概要 |
|---------|--------------|------|------|------|
| `save_to_google_photos(data, user)` | `POST /photos/save` | `{ imageId }` | `{ photoUrl }` | S3から画像を取得しGoogle Photoに保存 |

---

## PromptBuilderService (Backend内部サービス)

| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `build_prompt(location, activity)` | ロケーション名、活動カテゴリ | `{ prompt, negative_prompt }` | Claude経由で画像生成プロンプトを構築 |

---

## ImageGeneratorService (Backend内部サービス)

| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `generate(prompt, negative_prompt, count)` | プロンプト、ネガティブプロンプト、枚数 | `bytes[]` | Bedrockで画像を生成し画像バイナリを返す |

---

## ImageStorageService (Backend内部サービス)

| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `upload_temp(image_bytes, image_id)` | 画像バイナリ、ID | `{ s3_key }` | S3に一時保存 |
| `get_presigned_url(s3_key)` | S3キー | `{ url }` | プレビュー用署名付きURLを発行 |
| `download(s3_key)` | S3キー | `bytes` | S3から画像を取得 |

---

## GooglePhotosService (Backend内部サービス)

| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `get_or_create_album(access_token)` | Googleアクセストークン | `{ albumId }` | 「My Moments」アルバムを取得または作成 |
| `upload_image(access_token, image_bytes, description)` | トークン、画像バイナリ、説明文 | `{ photoUrl }` | Google Photoに画像をアップロード |

---

## DatabaseService (Backend内部サービス)

| メソッド | 入力 | 出力 | 概要 |
|---------|------|------|------|
| `get_user(user_id)` | ユーザーID | `User \| None` | ユーザー情報を取得 |
| `upsert_user(user_data)` | ユーザーデータ | `User` | ユーザー情報を作成または更新 |
| `save_location(user_id, location_data)` | ユーザーID、ロケーションデータ | `Location` | ロケーションを保存 |
| `list_locations(user_id)` | ユーザーID | `Location[]` | ロケーション履歴を取得 |
