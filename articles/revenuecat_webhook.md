---
title: "【RevenueCat】Webhookを使ってサーバー側で購入情報を取得する"
emoji: "🐾"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [RevenueCat, Flutter, ios, android,inaAppPurchase]
published: true
publication_name: ncdc
---

RevenueCatには便利なWebhook機能が存在します。
Webhookを使うことで、アプリ内課金の購入情報をサーバー（バックエンド）側でも管理することができます。

記載内容の詳細は、以下の公式ドキュメントを参照してください。

https://www.revenuecat.com/docs/webhooks

<br>

**【参考記事】**
https://codewithandrea.com/articles/webhooks-flutter-backend/

<br>

## 前提
サーバー（バックエンド）は、FireStore以外のDBを使用していることを前提として記載。
FireStoreを使用している場合は、Webhookを使わずにFirebase Extensionを使うとより便利に同期できる。
https://www.revenuecat.com/docs/firebase-integration

https://extensions.dev/extensions/revenuecat/firestore-revenuecat-purchases

<br>

## Webhookの使用背景
- サーバー側で購入情報を管理・同期しておきたい。
  - ex.) 管理画面など、スマホアプリとは別の画面で購入情報を表示したい。
- etc.

<br>

## Webhookを使用する際の考慮点
主には公式ドキュメント参照。
ドキュメントを読んでよく分からなかった部分を以下抜粋。

> **Syncing Subscription Status**
Webhooks are commonly used to sync a customer's subscription status across multiple systems. Because different webhook events contain unique information, we recommend calling the `GET /subscribers` [REST](https://docs.revenuecat.com/reference#subscribers) API endpoint after receiving any webhook. That way, the customer's information is always in the same format and is easily synced to your database. This approach is simpler than writing custom logic to handle each webhook event, and has the added benefit of making your system more robust and scalable.

----------

Webhookを受信した後、REST APIを呼び出すことが推奨されているが、最新のREST API v2 のドキュメントを参照すると、`GET/subscribers` というエントリは存在しない。
→ **REST APIを使用する場合は v1 のREST APIを使用する必要があるとのこと。**

https://community.revenuecat.com/sdks-51/it-is-recommended-to-call-the-rest-api-after-receiving-the-webhook-but-there-is-no-mention-of-get-subscribers-in-the-resa-api-v2-documentation-3735

<br>

カスタムロジックなどを作成する手間を省くために、Webhookを受信した後にREST APIを呼び出すことを推奨されている。
しかし、Webhookの共通フィールドだけを使用したい場合でもREST APIでデータを取得した方がいいのか？
→ **共通フィールドだけを使用する場合は、REST APIを使用する必要はないとのこと。**

https://community.revenuecat.com/sdks-51/why-is-it-recommended-to-go-through-the-rest-api-even-if-we-only-want-to-synchronize-common-fields-in-the-webhook-3734

<br>

## 設定
Webhookの設定は、コンソールの`Integrations`からWebhookを選択してエンドポイントを貼り付けるだけ。
また、設定画面からWebhookへの手動のテスト送信も可能。
|                              RevenueCatコンソール                              |                              RevenueCatコンソール                              |
| :----------------------------------------------------------------------------: | :----------------------------------------------------------------------------: |
| ![](https://storage.googleapis.com/zenn-user-upload/4081305ef09b-20231229.png) | ![](https://storage.googleapis.com/zenn-user-upload/17e23f690976-20231229.png) |


<br>

Webhookの動作確認には、[Webhook.site](https://webhook.site/)を使うと便利。
Webhook.siteにアクセスするとURLが自動発行されるので、それを設定画面のエンドポイント項目に貼り付けるだけでWebhookの確認ができる。
|                            Webhook.site起動時の画面                            |                              Webhook受信時の画面                               |
| :----------------------------------------------------------------------------: | :----------------------------------------------------------------------------: |
| ![](https://storage.googleapis.com/zenn-user-upload/b9a9c3814323-20231229.png) | ![](https://storage.googleapis.com/zenn-user-upload/368d0e10d941-20231229.png) |


<br>

RevenueCatが送信するWebhookの中身や、Webhook発火イベントの種類についてはドキュメント参照。
https://www.revenuecat.com/docs/event-types-and-fields
https://www.revenuecat.com/docs/sample-events

<br>

## 終わり
Webhook.siteで動作確認ができたら、実際のサーバー側にエンドポイントを作成して、購入情報を保存する処理を実装すれば自前のサーバー側でも購入情報を管理・同期ができるようになります。