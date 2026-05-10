# Application Design（統合ドキュメント）

## プロジェクト概要

**サービス名**: re:Write  
**構成**: 分離構成（Next.js フロントエンド + Python FastAPI バックエンド）  
**インフラ**: AWS (Lambda, API Gateway, DynamoDB, S3) + Amazon Bedrock

---

## アーキテクチャ概要

```
+------------------+     HTTPS      +----------------------+
|                  |  REST API      |                      |
|  Next.js         +--------------->+  FastAPI Backend     |
|  Frontend        |                |  (Python 3.11+)      |
|  (TypeScript)    |                |                      |
+------------------+                +----------+-----------+
        |                                      |
        | Google OAuth                         |
        | (ブラウザリダイレクト)                  |
        v                           +----------v-----------+
+------------------+                |  Internal Services   |
|  Google OAuth    |                |                      |
|  2.0             |                | - AuthService        |
+------------------+                | - LocationService    |
                                    | - ImageGenerationSvc |
                                    | - PhotoSaveService   |
                                    | - PromptBuilderSvc   |
                                    | - ImageGeneratorSvc  |
                                    | - ImageStorageSvc    |
                                    | - GooglePhotosSvc    |
                                    | - DatabaseService    |
                                    +----------+-----------+
                                               |
                    +--------------------------+---------------------------+
                    |              |            |              |           |
                    v              v            v              v           v
             +------+----+  +-----+----+  +----+-----+  +----+----+  +---+------+
             | Amazon    |  | Amazon   |  | AWS S3   |  | AWS     |  | Google   |
             | Bedrock   |  | Bedrock  |  | (一時    |  | DynamoDB|  | Photos   |
             | (Claude)  |  | (SDXL)   |  |  保存)   |  |         |  | API      |
             +-----------+  +----------+  +----------+  +---------+  +----------+
```

---

## コンポーネント一覧

詳細は `components.md` を参照。

| コンポーネント | 技術 | 役割 |
|--------------|------|------|
| Frontend | Next.js 14+ (TypeScript) | UI、OAuth開始、画像プレビュー |
| Backend API | Python FastAPI | REST API、オーケストレーション |
| Prompt Builder | Amazon Bedrock (Claude 3) | プロンプト生成 |
| Image Generator | Amazon Bedrock (SDXL) | AI画像生成 |
| Image Storage | AWS S3 | 画像一時保存 |
| Database | Amazon DynamoDB | ユーザー・ロケーション管理 |
| Google Photos | Google Photos API | 画像保存先 |

---

## サービスレイヤー一覧

詳細は `services.md` を参照。

| サービス | 責務 |
|---------|------|
| AuthService | Google OAuth + JWT発行 |
| LocationService | ロケーション登録・取得 |
| ImageGenerationService | 画像生成フロー全体のオーケストレーション |
| PhotoSaveService | Google Photo保存フローのオーケストレーション |
| PromptBuilderService | Claude経由でプロンプト構築 |
| ImageGeneratorService | Bedrock SDXL呼び出し |
| ImageStorageService | S3一時保存・署名付きURL |
| GooglePhotosService | Google Photos API連携 |
| DatabaseService | DynamoDBアクセス抽象化 |

---

## 主要設計決定

| 決定事項 | 選択 | 理由 |
|---------|------|------|
| フロントエンド/バックエンド構成 | 分離構成 | 将来の拡張性、バックエンドの独立スケール |
| バックエンド言語 | Python + FastAPI | AI/MLライブラリとの親和性、Bedrock SDK |
| AI画像生成 | Amazon Bedrock (SDXL) | AWSに統一、IAM認証でセキュア |
| プロンプト生成 | Claude (Bedrock) | LLMによる高品質プロンプト自動生成 |
| 認証セッション | JWT (ステートレス) | サーバーサイドセッション不要、シンプル |
| 画像一時保存 | S3 | 信頼性、プレビューURL発行、TTL管理 |
| ロケーション履歴 | DynamoDB | AWSに統一、スケーラブル |

---

## 依存関係

詳細は `component-dependency.md` を参照。

---

## メソッド一覧

詳細は `component-methods.md` を参照。

---

## 拡張性への考慮（Phase 2）

FR-EXT-02（Google Photo位置情報取得）の追加を見据えた設計：
- `LocationService` に `import_from_google_photos()` メソッドを追加するだけで対応可能
- `GooglePhotosService` に読み取りスコープ（`photoslibrary.readonly`）を追加
- 既存のコンポーネント境界を変更する必要なし
