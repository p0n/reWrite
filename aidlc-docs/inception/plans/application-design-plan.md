# Application Design Plan

## 概要
このプランはアプリケーションの高レベルコンポーネント設計とサービスレイヤー設計を行います。
以下の質問に回答してください。各 `[Answer]:` タグの後に選択肢の文字を記入してください。

---

## チェックリスト
- [x] 質問への回答収集
- [x] components.md 生成
- [x] component-methods.md 生成
- [x] services.md 生成
- [x] component-dependency.md 生成
- [x] application-design.md（統合ドキュメント）生成

---

## Question 1: フロントエンドとバックエンドの構成
フロントエンドとバックエンドをどのように構成しますか？

A) フルスタック Next.js — API Routes を使ってフロントエンドとバックエンドを1つのプロジェクトにまとめる（シンプル、PoC向き）
B) 分離構成 — Next.js フロントエンド + 独立したバックエンドAPI（Node.js または Python FastAPI）
C) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Question 2: バックエンド言語（Question 1 で B を選んだ場合）
バックエンドAPIの言語・フレームワークを選んでください。

A) Node.js + TypeScript（Express または Fastify）— フロントエンドと言語統一
B) Python + FastAPI — AI/ML系ライブラリとの親和性が高い
C) Question 1 で A を選んだため不要
D) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Question 3: AI画像生成サービス
どのAI画像生成サービスを使用しますか？

A) Amazon Bedrock（Stable Diffusion / Titan Image Generator）— AWSに統一できる
B) OpenAI DALL-E 3 — 高品質、シンプルなAPI
C) Stability AI API（直接）
D) Other (please describe after [Answer]: tag below)

[Answer]: A 

---

## Question 4: ロケーション解釈の方法
ユーザーが入力したロケーション名（例：「京都 金閣寺」）をどのように画像生成プロンプトに変換しますか？

A) シンプルな文字列結合 — 入力テキストをそのままプロンプトに組み込む（例：「A lively scene at 京都 金閣寺, enjoying sightseeing」）
B) LLMによるプロンプト強化 — LLM（例：Claude/GPT）がロケーション+活動カテゴリから詳細な英語プロンプトを生成する
C) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Question 5: 認証セッション管理
Google OAuth後のセッション管理をどのように行いますか？

A) JWT トークン（ステートレス）— サーバーサイドセッション不要
B) サーバーサイドセッション（DynamoDB or Redis）
C) Next.js Auth.js（NextAuth）— Google OAuth対応の標準ライブラリ
D) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 6: 画像の一時保存
AI生成画像をGoogle Photoに保存する前に、一時的にどこに保存しますか？

A) S3 に一時保存してからGoogle Photoへアップロード
B) メモリ上（バッファ）で直接Google Photo APIへストリーミング
C) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 7: ロケーション履歴の保存
ユーザーが入力したロケーション履歴をどのように管理しますか？

A) DynamoDB に保存（ユーザーIDをパーティションキー）
B) ブラウザのローカルストレージのみ（サーバー保存なし）
C) Other (please describe after [Answer]: tag below)

[Answer]: A

---

すべての質問に回答したら、「完了しました」とお知らせください。
