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
             | (Claude   |  | (Nova    |  |  保存)   |  |         |  | API      |
             | Sonnet4.6)|  | Canvas)  |  |          |  |         |  |          |
             +-----------+  +----------+  +----------+  +---------+  +----------+
```

---

## コンポーネント一覧

詳細は `components.md` を参照。

| コンポーネント | 技術 | 役割 |
|--------------|------|------|
| Frontend | Next.js 14+ (TypeScript) | UI、OAuth開始、画像プレビュー |
| Backend API | Python FastAPI | REST API、オーケストレーション |
| Prompt Builder | Amazon Bedrock (Claude claude-sonnet-4-6) | プロンプト生成 |
| Image Generator | Amazon Bedrock (Nova Canvas) ※レガシー、将来Nova 2 Omniへ移行検討 | AI画像生成 |
| Image Storage | AWS S3 | 画像一時保存 |
| Database | Amazon DynamoDB | ユーザー・ロケーション管理 |
| Google Photos | Google Photos API | 画像保存先 |
| Location Suggestion | Geocoding API (AWS Location Service / Google Maps) | Google Photo位置情報からロケーション提案 |

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
| ImageGeneratorService | Bedrock Nova Canvas呼び出し（将来Nova 2 Omniへ移行検討） |
| ImageStorageService | S3一時保存・署名付きURL |
| GooglePhotosService | Google Photos API連携（保存 + 位置情報読み取り） |
| DatabaseService | DynamoDBアクセス抽象化 |
| LocationSuggestionService | Google Photo EXIF位置情報取得・逆ジオコーディング・ロケーション提案（FR-05） |

---

## 主要設計決定

| 決定事項 | 選択 | 理由 |
|---------|------|------|
| フロントエンド/バックエンド構成 | 分離構成 | 将来の拡張性、バックエンドの独立スケール |
| バックエンド言語 | Python + FastAPI | AI/MLライブラリとの親和性、Bedrock SDK |
| AI画像生成 | Amazon Bedrock (Nova Canvas) | AWSに統一、IAM認証でセキュア。Nova Canvasはレガシーモデルのため将来Nova 2 Omniへ移行検討 |
| プロンプト生成 | Claude claude-sonnet-4-6 (Bedrock) | LLMによる高品質プロンプト自動生成 |
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

FR-EXT-02（Google Photo位置情報からの自動画像生成）の追加を見据えた設計：
- FR-05（ロケーション提案）はMVPで実装済みのため、自動生成トリガーの追加のみで対応可能
- `ImageGenerationService` にバッチ実行モードを追加するだけで対応可能
- `GooglePhotosService` の読み取りスコープ（`photoslibrary.readonly`）はFR-05で既に取得済み
- 既存のコンポーネント境界を変更する必要なし
