# Unit of Work Story Map

## 機能要件とユニットのマッピング

| 機能要件 | Unit 1: Frontend | Unit 2: Backend | Unit 3: Infrastructure |
|---------|:---:|:---:|:---:|
| FR-01: ユーザー認証 | ✓ ログインUI, OAuthリダイレクト, JWT保存 | ✓ OAuth処理, JWT発行, ユーザー保存 | ✓ DynamoDB usersテーブル, IAM |
| FR-02: ロケーション入力 | ✓ 入力フォーム, 履歴表示UI | ✓ ロケーションAPI, DynamoDB保存 | ✓ DynamoDB locationsテーブル |
| FR-03: AI画像生成 | ✓ 生成ボタン, プレビュー表示, ポーリング | ✓ 生成API, Claude呼び出し, Nova Canvas呼び出し, S3保存 | ✓ S3バケット, Bedrock IAM権限 |
| FR-04: Google Photos連携 | ✓ 保存ボタン, 完了表示 | ✓ Google Photos API呼び出し | ✓ Google OAuth設定（GCP側） |
| FR-05: ロケーション提案 | ✓ 提案リスト表示UI, 選択操作 | ✓ 位置情報取得API, 逆ジオコーディング | ✓ Geocoding API権限 |
| FR-EXT-02: 自動画像生成（MVP外） | — | — | — |

## ユニット別タスクサマリー

### Unit 1: Frontend タスク

**認証（FR-01）**:
- [ ] ログインページ実装（Googleログインボタン）
- [ ] Google OAuthリダイレクト処理
- [ ] `/auth/callback` コールバックページ実装
- [ ] JWTトークンのCookie保存・管理
- [ ] 認証状態管理（ログイン/ログアウト）

**ロケーション入力（FR-02）**:
- [ ] ロケーション入力フォームコンポーネント
- [ ] 活動カテゴリ選択コンポーネント
- [ ] ロケーション履歴一覧表示

**AI画像生成（FR-03）**:
- [ ] 画像生成リクエスト送信
- [ ] 生成状況ポーリング（ローディング表示）
- [ ] 生成画像プレビュー表示（複数枚対応）
- [ ] 再生成ボタン

**Google Photos連携（FR-04）**:
- [ ] 「Google Photoに保存」ボタン
- [ ] 保存完了通知表示

**ロケーション提案（FR-05）**:
- [ ] 「Google Photoから場所を取得」ボタン
- [ ] ロケーション提案リストコンポーネント（LocationSuggestions.tsx）
- [ ] 提案から選択してロケーション入力フォームに反映

**共通**:
- [ ] APIクライアント設定（JWT自動付与）
- [ ] エラーハンドリング・トースト通知
- [ ] レスポンシブデザイン（Tailwind CSS）

---

### Unit 2: Backend タスク

**認証（FR-01）**:
- [ ] `POST /auth/google` — Google OAuth認可コード交換
- [ ] Google UserInfo API呼び出し
- [ ] JWT発行・検証ミドルウェア
- [ ] `GET /auth/me` — ユーザー情報取得
- [ ] DynamoDB ユーザー保存（upsert）

**ロケーション入力（FR-02）**:
- [ ] `POST /locations` — ロケーション登録
- [ ] `GET /locations` — ロケーション履歴取得
- [ ] DynamoDB ロケーション保存・取得

**AI画像生成（FR-03）**:
- [ ] `POST /generate` — 画像生成ジョブ開始
- [ ] `GET /generate/{job_id}` — 生成ステータス確認
- [ ] PromptBuilderService（Claude呼び出し）
- [ ] ImageGeneratorService（Nova Canvas呼び出し）
- [ ] ImageStorageService（S3一時保存・署名付きURL）

**Google Photos連携（FR-04）**:
- [ ] `POST /photos/save` — Google Photo保存
- [ ] GooglePhotosService（アルバム作成・画像アップロード）

**ロケーション提案（FR-05）**:
- [ ] `GET /locations/suggestions` — Google Photo位置情報取得・提案
- [ ] GooglePhotosService.list_photos_with_location()（EXIF位置情報読み取り）
- [ ] LocationSuggestionService（逆ジオコーディング・重複排除）

---

### Unit 3: Infrastructure タスク

**Phase 1（先行作成）**:
- [ ] DynamoDB `users` テーブル（PK: userId）
- [ ] DynamoDB `locations` テーブル（PK: userId, SK: createdAt）
- [ ] S3 バケット（画像一時保存、TTL設定）
- [ ] IAM ロール（Bedrock, DynamoDB, S3, Google Photos読み取りアクセス）
- [ ] Geocoding API 設定（AWS Location Service または Google Maps API）

**Phase 2（バックエンド完成後）**:
- [ ] Lambda 関数（バックエンドコードデプロイ）
- [ ] API Gateway（REST API設定、CORSヘッダー）
- [ ] 環境変数設定（Parameter Store）

## 並行開発スケジュール（概要）

```
Week 1-2:
  Unit 3 (Phase 1): DynamoDB, S3, IAM, Geocoding API設定
  Unit 2: 認証API + ロケーションAPI + ロケーション提案API開発
  Unit 1: ログインUI + ロケーション入力UI + 提案UI開発（モックAPI使用）

Week 3-4:
  Unit 2: 画像生成API + Google Photos API開発
  Unit 1: 画像生成UI + 保存UI開発（モックAPI使用）

Week 5:
  Unit 3 (Phase 2): Lambda + API Gateway デプロイ
  統合テスト: Frontend → Backend → AWS サービス
```
