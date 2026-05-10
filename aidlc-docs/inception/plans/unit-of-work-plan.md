# Unit of Work Plan

## 概要
このプランはシステムを開発ユニットに分解します。
以下の質問に回答してください。各 `[Answer]:` タグの後に選択肢の文字を記入してください。

---

## チェックリスト
- [x] 質問への回答収集
- [x] unit-of-work.md 生成
- [x] unit-of-work-dependency.md 生成
- [x] unit-of-work-story-map.md 生成

---

## Question 1: ユニット分解の方針
システムをどのように開発ユニットに分解しますか？

A) 2ユニット — フロントエンド（Next.js）とバックエンド（FastAPI）を独立したユニットとして開発する
B) 1ユニット — フロントエンドとバックエンドをまとめて1つのユニットとして開発する
C) Other (please describe after [Answer]: tag below)

[Answer]: 2

---

## Question 2: インフラ（AWS）の扱い
AWSインフラ（DynamoDB, S3, Lambda/ECS等）はどのユニットで管理しますか？

A) バックエンドユニットに含める — バックエンドと一緒にIaCを管理する
B) 独立したインフラユニットとして分離する — CDK/Terraformを別ユニットで管理する
C) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Question 3: 開発の優先順位
どちらのユニットを先に開発しますか？

A) バックエンドから先に開発する — APIを先に固めてからフロントエンドを開発する
B) フロントエンドから先に開発する — UIを先に作ってからAPIを繋ぎ込む
C) 並行開発する — フロントエンドとバックエンドを同時に進める
D) Other (please describe after [Answer]: tag below)

[Answer]: C

---

すべての質問に回答したら、「完了しました」とお知らせください。
