---
title: "【curl】シンプルなLINEリッチメニュの作成"
emoji: "🌽"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [curl, LINE, LineRichMenu]
published: true
---
<!-- 公開に変更する -->
<!-- 参考記事のURLを更新する -->

公式のサンプル通りに、超シンプルなリッチメニュを作成。
[LINE Developers＞Home＞ドキュメント＞Messaging API＞リッチメニューを使う](https://developers.line.biz/ja/docs/messaging-api/using-rich-menus/#creating-a-rich-menu-using-the-messaging-api)

# 0.LINE Developersの登録
※登録方法の詳細は割愛(検索するとたくさん出てくるので、そちらを各自参照)

LINE Developers に記載があるアクセストークンを主に使用
![](https://storage.googleapis.com/zenn-user-upload/9a9926a8c059-20230221.png)
![](https://storage.googleapis.com/zenn-user-upload/7ab46269d8b0-20230221.png)

※以下サンプルコードの{access_token}箇所には、上記のアクセストークンが入るものとする。

# 1.リッチメニュ作成
```
curl -v https://api.line.me/v2/bot/richmenu \
-H 'Authorization: Bearer {access_token}' \
-H 'Content-Type: application/json' \
-d \
'{
  "size":{
      "width":2500,
      "height":1686
  },
  "selected": false,
  "name": "LINE Developers Info",
  "chatBarText": "Tap to open",
  "areas": [
      {
          "bounds": {
              "x": 34,
              "y": 24,
              "width": 169,
              "height": 193
          },
          "action": {
              "type": "uri",
              "uri": "https://developers.line.biz/en/news/"
          }
      },
      {
          "bounds": {
              "x": 229,
              "y": 24,
              "width": 207,
              "height": 193
          },
          "action": {
              "type": "uri",
              "uri": "https://www.line-community.me/ja/"
          }
      },
      {
          "bounds": {
              "x": 461,
              "y": 24,
              "width": 173,
              "height": 193
          },
          "action": {
              "type": "uri",
              "uri": "https://engineering.linecorp.com/en/blog/"
          }
      }
  ]
}'
```

# 2.リッチメニュの画像をアップロード
```
// 上記のリッチメニュ作成時に生成されたリッチメニュID{richmenu_id}をurlに指定
curl -v -X POST https://api-data.line.me/v2/bot/richmenu/{richmenu_id}/content \
-H "Authorization: Bearer {access_token}" \
-H "Content-Type: image/png" \
-T {画像パス}
```

# 3.デフォルトのリッチメニューを設定
```
curl -v -X POST https://api.line.me/v2/bot/user/all/richmenu/{richmenu_id} \
-H "Authorization: Bearer {access_token}"
```

# 応用
curlで作成したリッチメニュを、AWS Lambdaで作成してみる。

↓coming soon...
[【Flutter / Lambda】LINEリッチメニュ作成]()

# 備考
- リクエストの確認には以下の記事が参考になった。
  [APIクライアント開発時のモックに使えるhttpbinの紹介](https://qiita.com/sameyasu/items/adacceb8a1bee893599b)
- headersのjsonデータの中身を変更すると、もっと多機能なリッチメニュを作成できるので色々試してみてください。