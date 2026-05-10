# Unit of Work Dependencies

## ユニット間依存関係マトリクス

| ユニット | 依存先 | 依存種別 | 説明 |
|---------|--------|---------|------|
| Unit 1: Frontend | Unit 2: Backend | ランタイム依存 | バックエンドAPIを呼び出す（開発時はモック可） |
| Unit 2: Backend | Unit 3: Infrastructure | ランタイム依存 | DynamoDB, S3, Bedrockを使用 |
| Unit 3: Infrastructure | なし | — | 独立ユニット |

## 依存関係図

```
Unit 3: Infrastructure
  (DynamoDB, S3, Lambda, API GW)
         ^
         | ランタイム依存
         |
Unit 2: Backend (FastAPI)
         ^
         | ランタイム依存
         | (開発時はモック可)
         |
Unit 1: Frontend (Next.js)
```

## 開発時の依存解消戦略

### Unit 1 (Frontend) の独立開発
- バックエンドAPIのモックサーバー（MSW または json-server）を使用
- API契約（OpenAPI仕様）を先に定義してフロントエンド開発を進める
- 環境変数 `NEXT_PUBLIC_API_URL` でAPIエンドポイントを切り替え

### Unit 2 (Backend) の独立開発
- ローカル環境でDynamoDB Local、LocalStack（S3）を使用
- Amazon Bedrock はAWSアカウントへの接続が必要（ローカルモック困難）
- 環境変数で本番/ローカル環境を切り替え

### Unit 3 (Infrastructure) の先行作成
- Phase 1: DynamoDB テーブルとS3バケットを先に作成（Unit 2開発に必要）
- Phase 2: Lambda + API Gateway はUnit 2完成後にデプロイ

## ビルド・デプロイ順序

```
Step 1: Unit 3 (Infrastructure Phase 1)
  - DynamoDB テーブル作成
  - S3 バケット作成
  - IAM ロール作成

Step 2: Unit 1 + Unit 2 (並行開発)
  - Frontend: モックAPIで開発
  - Backend: ローカル環境で開発・テスト

Step 3: Unit 3 (Infrastructure Phase 2)
  - Lambda 関数デプロイ（Unit 2のコードを使用）
  - API Gateway 設定

Step 4: 統合テスト
  - Frontend → Backend → AWS サービス の結合確認
```

## API契約（フロントエンド/バックエンド間）

フロントエンドとバックエンドの並行開発を可能にするため、以下のAPI契約を先に定義する：

| エンドポイント | メソッド | リクエスト | レスポンス |
|--------------|---------|-----------|-----------|
| `/auth/google` | POST | `{ code: string }` | `{ access_token: string, user: User }` |
| `/auth/me` | GET | — (JWT Header) | `User` |
| `/locations` | POST | `{ name: string, activity: string }` | `Location` |
| `/locations` | GET | — (JWT Header) | `Location[]` |
| `/generate` | POST | `{ location_id: string, count: number }` | `{ job_id: string }` |
| `/generate/{job_id}` | GET | — | `{ status: string, images: ImageResult[] }` |
| `/photos/save` | POST | `{ image_id: string }` | `{ photo_url: string }` |
