# 要件確認の質問

AI-DLCワークフローを開始するにあたり、以下の質問にお答えください。
各質問の `[Answer]:` タグの後に、選択肢の文字（A、B、C など）を記入してください。
選択肢に該当するものがない場合は、最後の「Other」を選び、詳細を記述してください。

---

## Question 1
どのようなアプリケーションを開発したいですか？

A) Webアプリケーション（フロントエンド + バックエンド）
B) バックエンドAPI / マイクロサービス
C) モバイルアプリ（iOS / Android）
D) CLIツール / スクリプト
E) データパイプライン / バッチ処理
F) Other (please describe after [Answer]: tag below)

[Answer]: F
自分の情報を管理するWebアプリとバックグラウドで自動的に動作するAIエージェント

---

## Question 2
アプリケーションの主な目的・解決したい課題は何ですか？（自由記述）

[Answer]: 
XやInstagramで知り合いがリア充な投稿をしているのに、引きこもってる自分が情けなくなるので、自分のプロファイルからありがちなフェイクポストをして、フォロワーはもちろん自分までそれが本物の楽しい思い出として勘違いしてしまうほど完璧な記録を捏造してくれるサービス。

---

## Question 3
主なターゲットユーザーは誰ですか？

A) 一般消費者（B2C）
B) 企業・ビジネスユーザー（B2B）
C) 社内ユーザー（社内ツール）
D) 開発者・エンジニア向けツール
E) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 4
使用したいプログラミング言語・フレームワークはありますか？

A) JavaScript / TypeScript（React, Next.js, Node.js など）
B) Python（FastAPI, Django, Flask など）
C) Java / Kotlin（Spring Boot など）
D) Go
E) 特にこだわりなし（AI-DLCに任せる）
F) Other (please describe after [Answer]: tag below)

[Answer]: AかB

---

## Question 5
デプロイ先・インフラはどこを想定していますか？

A) AWS（クラウド）
B) GCP（クラウド）
C) Azure（クラウド）
D) オンプレミス / ローカル環境
E) まだ決めていない
F) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 6
データベースは必要ですか？必要な場合、どのタイプを想定していますか？

A) リレーショナルDB（PostgreSQL, MySQL など）
B) NoSQL ドキュメントDB（MongoDB, DynamoDB など）
C) キャッシュ / KVS（Redis など）
D) データベース不要
E) まだ決めていない
F) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Question 7
ユーザー認証・認可は必要ですか？

A) はい（ユーザーログイン機能が必要）
B) いいえ（認証不要）
C) まだ決めていない
D) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 8
プロジェクトの規模感はどのくらいですか？

A) 小規模（個人プロジェクト / PoC / プロトタイプ）
B) 中規模（チーム開発 / 本番運用を想定）
C) 大規模（エンタープライズ / 高可用性が必要）
D) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 9: Security Extensions
このプロジェクトにセキュリティ拡張ルールを適用しますか？

A) Yes — すべてのSECURITYルールをブロッキング制約として適用する（本番グレードのアプリケーションに推奨）
B) No — すべてのSECURITYルールをスキップする（PoC、プロトタイプ、実験的プロジェクトに適切）
C) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Question 10: Property-Based Testing Extension
このプロジェクトにプロパティベーステスト（PBT）ルールを適用しますか？

A) Yes — すべてのPBTルールをブロッキング制約として適用する（ビジネスロジック、データ変換、シリアライゼーション、ステートフルコンポーネントを持つプロジェクトに推奨）
B) Partial — 純粋関数とシリアライゼーションのラウンドトリップにのみPBTルールを適用する（アルゴリズムの複雑さが限られたプロジェクトに適切）
C) No — すべてのPBTルールをスキップする（シンプルなCRUDアプリ、UIのみのプロジェクト、ビジネスロジックのない薄い統合レイヤーに適切）
D) Other (please describe after [Answer]: tag below)

[Answer]: C

---

すべての質問に回答したら、「完了しました」とお知らせください。
