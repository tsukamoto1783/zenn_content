---
title: "AWS SNSとFirebase Cloud Messagingを使ってモバイルデバイスにプッシュ通知を送信する"
emoji: "📮"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, fcm, awsSNS, aws, firebase]
published: true
publication_name: ncdc
---

AWS SNS を使って端末にプッシュ通知を送信してみる。

# AWS SNS の簡単な説明

https://docs.aws.amazon.com/sns/latest/dg/welcome.html
ドキュメントよりざっくり引用・要約。

> Amazon Simple Notification Service (Amazon SNS) は、
> 配信者から購読者 へのメッセージ配信を提供するマネージドサービスです。
> 配信者は、トピックにメッセージを送信することで、購読者と非同期的に通信します。
>
> クライアントは、Amazon Data Firehose、Amazon SQS、AWS Lambda、HTTP、E メール、モバイルプッシュ通知、モバイルテキストメッセージ (SMS) などのサポートされたエンドポイントタイプを使用して発行されたメッセージを受信できます。

<br>

# 前提条件

- モバイルアプリ側は Flutter で作成されたアプリで動作検証しています。（記事の内容的にはあまり関係ないですが。）
- モバイルアプリと FCM（Firebase Cloud Messaging） との接続や設定は既に完了しているものとします。この記事のメインは AWS SNS の設定の記載となります。
- 検証用の簡単な設定となっているため、AWS SNS のオプション設定についてはあまり触れません。
- AWS SNS の設定については、ここではコンソールからポチポチと設定しております。
- コンソール画面の UI はころころ変わるので、閲覧時には変更されてしまってる可能性があることご了承ください。

<br>

# 設定方法

## トピック作成

AWS SNS を選択して、トピックを作成。

オプションが沢山あるが、とりあえずは必須項目の`タイプ`と`トピック名`だけ設定。
上記項目は、作成後は変更できないこと注意。

![](https://storage.googleapis.com/zenn-user-upload/88e180a1197e-20240531.png)

![](https://storage.googleapis.com/zenn-user-upload/888c120a8da7-20240531.png)

### トピックとは？

https://docs.aws.amazon.com/ja_jp/sns/latest/dg/sns-create-subscribe-endpoint-to-topic.html

https://docs.aws.amazon.com/ja_jp/sns/latest/dg/mobile-push-send-topicmobile.html

上記のドキュメントからざっくり要約すると、

Amazon SNS において、メッセージを公開するための「アクセスポイント、チャネル」のようなものです。
メッセージの配信者がこのトピックにメッセージを送信すると、そのトピックをサブスクライブ（購読）しているエンドポイント（受信者）にメッセージが配信されます。

エンドポイントには、今回のプッシュ通知以外にも、SMS、HTTP/S エンドポイント、Amazon SQS、AWS Lambda などがエンドポイントに設定できます。

AWS SNS でのプッシュ通知で言うと、
**AWS SNS で設定した「トピック」に紐づいているデバイス（デバイストークン）に対して、プッシュ通知が配信されます。**
**「トピック」≒「グループ」**

<br>

## プッシュ通知サービスのプラットフォーム

アプリで使用するプッシュ通知サービスのプラットフォームを設定。
![](https://storage.googleapis.com/zenn-user-upload/f1227ffd9550-20240531.png)

作成ボタンを押下後、「アプリ名」と「プッシュ通知サービスのプラットフォーム」を選択する画面が出るので、各種設定していく。
今回は FCM を使用するので、FCM を選択。

![](https://storage.googleapis.com/zenn-user-upload/05239595a8e9-20240531.png)
![](https://storage.googleapis.com/zenn-user-upload/ad3520d591b6-20240531.png =350x )
![](https://storage.googleapis.com/zenn-user-upload/7e0d416b1a14-20240531.png)
FCM を選択すると、Firebase の json ファイルをアップロードする必要があるため、Firebase のコンソールから json ファイル取得する。

![](https://storage.googleapis.com/zenn-user-upload/ce35f91bb12d-20240531.png =400x)
![](https://storage.googleapis.com/zenn-user-upload/b0ebbacb62e7-20240531.png)
![](https://storage.googleapis.com/zenn-user-upload/46af38af6828-20240531.png =300x)

json ファイルを作成して、その json ファイルをアップロードして登録すると、以下のように FCM と紐づいたアプリケーションが作成される。

![](https://storage.googleapis.com/zenn-user-upload/2cacf4b8a886-20240531.png)

<br>

## プッシュ通知を送信する端末のトークン登録

先ほど作成したアプリケーションを選択し、通知を送信する端末のトークンを登録する。
Flutter アプリ側では、以下のようなコードで取れるトークンのこと。

```dart: 例
final messaging = FirebaseMessaging.instance;
final token = await messaging.getToken();
```

![](https://storage.googleapis.com/zenn-user-upload/b1f47d1c7ebb-20240531.png)
![](https://storage.googleapis.com/zenn-user-upload/ac5e7d0a139f-20240531.png)

登録後、以下のように表示される。
ARN は後ほど使用するのでコピーしておく。

![](https://storage.googleapis.com/zenn-user-upload/8514484ba48f-20240531.png)

<br>

## サブスクリプション設定

次にサブスクリプションの設定を行う。
![](https://storage.googleapis.com/zenn-user-upload/af6ecee68b10-20240531.png =500x)
![](https://storage.googleapis.com/zenn-user-upload/a26df7147440-20240531.png =500x)
![](https://storage.googleapis.com/zenn-user-upload/30578e34eedb-20240531.png =300x)

先ほど作成した、「トピック」と端末設定後にコピーした「ARN」を設定して登録する。

![](https://storage.googleapis.com/zenn-user-upload/de7b321631ec-20240531.png)

<br>

## プッシュ通知の送信テスト

作成したトピックを選択して、先ほど登録したサブスクリプション（エンドポイント、通知対象端末）が紐づいていることを確認。

![](https://storage.googleapis.com/zenn-user-upload/89e5fd2038f4-20240531.png =500x)
![](https://storage.googleapis.com/zenn-user-upload/7e4ba199a7cf-20240531.png =500x)

「メッセージを発行」ボタンを押下し、端末に送るメッセージを設定する。
![](https://storage.googleapis.com/zenn-user-upload/60274daf2cfa-20240531.png)

ここのメッセージの中身は、アプリ側の実装によるかと思うので、適宜適切なパラメータや body の中身にしていただければと思います。
https://docs.aws.amazon.com/sns/latest/dg/sns-send-custom-platform-specific-payloads-mobile-devices.html

送信を押下すると、アプリ側に送信が飛ぶことを確認。

<br>

## 備考(メモ)

FCM 単体でのプッシュ通知と、AWS SNS とでトピックの役割？設定の認識？が若干違うこと注意。

どちらも「指定したトピックを購読しているユーザー」に対して、プッシュ通知を送信することができますが、

FCM 単体での push 通知設定では、
「指定したトピックを購読しているユーザー」は、**FCM のコンソールからの設定ではなく、アプリ側でトピック名を設定する実装が必要。**

```dart
final messaging = FirebaseMessaging.instance;
await messaging.subscribeToTopic("notice_topic")
```

![](https://storage.googleapis.com/zenn-user-upload/42a2c9035b71-20240601.png =300x)
_FCM のコンソール_

SNS の push 通知設定では、
「指定したトピックを購読しているユーザー」は、**AWS SNS のコンソール上で設定する。アプリ側にはトピック名を設定する実装は不要。**
![](https://storage.googleapis.com/zenn-user-upload/7e4ba199a7cf-20240531.png =500x)

<br>

## 参考

:::details 参考リンク
https://dev.classmethod.jp/articles/tsnote-amazon-sns-raw-message-delivery/
https://qiita.com/Rei_2020/items/ab1768c1a0191b73492e#%E9%80%9A%E7%9F%A5%E3%83%A1%E3%83%83%E3%82%BB%E3%83%BC%E3%82%B8
https://qiita.com/Dai_Kentaro/items/5b17272207c66adcdbe4
https://zenn.dev/issy/articles/zenn-sns-overview#%E3%83%88%E3%83%94%E3%83%83%E3%82%AF

:::
