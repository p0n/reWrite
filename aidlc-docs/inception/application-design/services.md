# Services

## サービスレイヤー概要

バックエンド（FastAPI）内部のサービスレイヤーは、ルーター（HTTPハンドラー）とインフラ（外部API・DB）の間を仲介するオーケストレーション層として機能する。

---

## Service 1: AuthService

**責務**: Google OAuth 2.0 認証フローの管理とJWT発行

**オーケストレーションフロー**:
```
1. Google OAuth認可コードを受け取る
2. Google Token Endpoint に認可コードを送信してアクセストークンを取得
3. Google UserInfo API でユーザー情報を取得
4. DatabaseService.upsert_user() でユーザーをDynamoDBに保存/更新
5. JWTを生成して返却
```

**依存サービス**: DatabaseService, Google OAuth API

---

## Service 2: LocationService

**責務**: ロケーション情報の管理

**オーケストレーションフロー（登録）**:
```
1. JWTからユーザーIDを検証
2. ロケーション名・活動カテゴリを受け取る
3. DatabaseService.save_location() でDynamoDBに保存
4. 保存したLocationオブジェクトを返却
```

**依存サービス**: DatabaseService

---

## Service 3: ImageGenerationService

**責務**: 画像生成の全フローをオーケストレーション（非同期）

**オーケストレーションフロー**:
```
1. JWTからユーザーIDを検証
2. DatabaseService からロケーション情報を取得
3. PromptBuilderService.build_prompt() でプロンプトを生成（Claude呼び出し）
4. ImageGeneratorService.generate() で画像を生成（Bedrock呼び出し）
5. ImageStorageService.upload_temp() でS3に一時保存
6. ImageStorageService.get_presigned_url() でプレビューURL生成
7. ジョブ完了ステータスと画像URLを返却
```

**依存サービス**: PromptBuilderService, ImageGeneratorService, ImageStorageService, DatabaseService

---

## Service 4: PhotoSaveService

**責務**: 生成画像をGoogle Photoに保存するフローをオーケストレーション

**オーケストレーションフロー**:
```
1. JWTからユーザーIDを検証
2. DatabaseService からユーザーのGoogleアクセストークンを取得
3. ImageStorageService.download() でS3から画像を取得
4. GooglePhotosService.get_or_create_album() でアルバムを確認/作成
5. GooglePhotosService.upload_image() でGoogle Photoにアップロード
6. 保存完了URLを返却
```

**依存サービス**: ImageStorageService, GooglePhotosService, DatabaseService

---

## Service 5: PromptBuilderService

**責務**: Claude（Amazon Bedrock）を使って画像生成プロンプトを構築

**オーケストレーションフロー**:
```
1. ロケーション名と活動カテゴリを受け取る
2. Claudeへのシステムプロンプトを構築
   - 「リア充っぽい」シーンの詳細な英語プロンプトを生成するよう指示
   - ロケーションの特徴・雰囲気を反映するよう指示
3. Amazon Bedrock (Claude) を呼び出す
4. 生成されたプロンプトとネガティブプロンプトを返却
```

**依存**: Amazon Bedrock (Claude 3 Sonnet/Haiku)

---

## Service 6: ImageGeneratorService

**責務**: Amazon Bedrock（Stable Diffusion XL）で画像を生成

**オーケストレーションフロー**:
```
1. プロンプト・ネガティブプロンプト・枚数を受け取る
2. Bedrock InvokeModel API を呼び出す（Stable Diffusion XL）
3. 生成された画像バイナリ（base64デコード済み）を返却
```

**依存**: Amazon Bedrock (Stable Diffusion XL / Titan Image Generator)

---

## Service 7: ImageStorageService

**責務**: S3への画像一時保存と署名付きURL管理

**依存**: AWS S3

---

## Service 8: GooglePhotosService

**責務**: Google Photos Library API との連携

**依存**: Google Photos Library API v1

---

## Service 9: DatabaseService

**責務**: DynamoDBへのデータアクセス抽象化

**依存**: Amazon DynamoDB

---

## Service 10: LocationSuggestionService（新規追加）

**責務**: Google Photoの写真から位置情報を取得してロケーション候補を提案

**オーケストレーションフロー**:
```
1. JWTからユーザーIDを検証
2. DatabaseService からユーザーのGoogleアクセストークンを取得
3. GooglePhotosService.list_photos_with_location() でEXIF位置情報付き写真を取得
4. 緯度・経度を逆ジオコーディングでロケーション名に変換
5. 重複排除・ソートしてロケーション候補リストを返却
```

**依存サービス**: GooglePhotosService, DatabaseService, Geocoding API

**注意**: 自動画像生成はMVP対象外。ユーザーが候補から選択して手動で画像生成を開始する。
