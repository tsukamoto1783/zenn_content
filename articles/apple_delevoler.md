---
title: "【Flutter/iOS】ややこしい証明書周りの関係をざっくり整理して理解してみた"
emoji: "🍎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, ios, apple, 証明書]
published: true
publication_name: ncdc
---
何度も証明書周りの理解がこんがらがり、つまづいていたので、改めて**ざっくり噛み砕いて**整理してみました。
ここでは全体の流れや、それぞれの項目の役割のイメージが掴めるように記載しています。

以下、基本的にはApple Developerの各種項目を基に順に見ていきます。
![](https://storage.googleapis.com/zenn-user-upload/15e115b55446-20230930.png)

<br>

## 全体の大まかな関係性
![](https://storage.googleapis.com/zenn-user-upload/f65e29cc8e4e-20230930.png)

<br>

## 1.CSR（Cert Signing Request：証明書署名要求）
**「Appleからアプリ開発許可書（証明書）を発行してもらうための申込書」**
![](https://storage.googleapis.com/zenn-user-upload/b211bade75c1-20230930.png =x200)

ローカルPCのキーチェーンにて作成できる。
ローカルPC情報（個人特定情報）を申込書に記載するイメージ。

作成時に内部的に以下作成してくれる。
- 公開鍵と秘密鍵
- 識別情報
- 署名

拡張子は`.CertSigningRequest`
![](https://storage.googleapis.com/zenn-user-upload/fdc523c8fa80-20230930.png)

<br>

## 2.Certificate（証明書）
**「Appleからアプリ開発許可されたという証明書」**
![](https://storage.googleapis.com/zenn-user-upload/c55be2463823-20230930.png =x200)

CSRを用いて、Apple Developerに申請することで発行される。
この発行された証明書をDLすることでキーチェーンに保存され、先ほどの※**CSRと紐づき開発（実機build）が可能に。**

**ローカルPC情報が記載されたCSRと証明書はセットで初めて開発（実機build）が可能になるイメージ。**

※正しくは、`CSRを作成時に作成された秘密鍵と証明書を紐づけることで開発が可能に。`以下、分かりやすいように`CSR（秘密鍵）`と記載します。
※シミュレーターなら証明書不要でbuildできる。

<br>

Certificate（証明書）には、主に以下2種類ある。
- Development :開発用
- Distribution :配信用

どちらもAppleからの開発許可証的な役割なので、仕組み基本は同じだが、**どちらの証明書もチームに最大数が決まっていることに要注意。**

運用としては、
- 組織の開発者各々がDevelopmentの証明書を作成して、Distributionの証明書は組織で管理共有。
- fastlaneとかを使用する場合は、Developmentの証明書も両方とも組織で管理共有。
とかがよくあるパターンかと。

拡張子は`.cer`

<br>

## 3. .p12ファイル
**「PC間で共有が可能なCertificate（証明書）≒Certificate（証明書）の完成版」**
![](https://storage.googleapis.com/zenn-user-upload/ebfd64921228-20230930.png =x200)

Distributionの証明書を組織で共有する例を挙げると、
Certificate（証明書）だけを共有しても、共有した先のPCにCSR（秘密鍵）が無いと情報が一致せず開発ができない。
そこで、Certificate（証明書）とCSR（秘密鍵）を一つのファイルにまとめたものが.p12ファイル。
この.p12ファイルを共有することで、どのPCでも開発が可能となる。

`どのPCでも開発が可能となる。`ということは、逆に言うと誰でも開発が可能となることになるので、**取り扱い、保管には十分注意が必要**。

また、.p12ファイルを作成する際は、CSR（秘密鍵）とCertificate（証明書）とが紐づいている大元のPCが存在することになるので、大元のPCは組織内のどのPCなのかは把握しておいた方が吉。
新しいPCへ移行時や、退職者が出た時などに困る可能性があるため。

拡張子は`.p12`

<br>

## 4.Identifier（識別子）
**「「このアプリは誰が作って、こんな機能を使ってますよ。」という名札」**
![](https://storage.googleapis.com/zenn-user-upload/05aa2787357b-20230930.png =x200)

**Identifier ≒ App ID**
他の記事ではApp IDと記載されていることが多い。

実際にApple DeveloperのIdentifiersで作成ボタンを押すと、すぐにApp IDの作成をすることになるので、実質は同じことかと。

Apple DeveloperのIdentifiersでは主に以下を指定して、App IDを作成する。
- Bundle ID：アプリを識別する文字列
- App ID Prefix：Team IDが自動で割り振られる
- Capabilities or AppServices: 使用する機能を選択する（push通知とか）

**Identifier（≒App ID）には開発用や配信用などの種類はなく、統一で1つでOK。**
![](https://storage.googleapis.com/zenn-user-upload/e0fbfd178d30-20230930.png)

<br>

## 5. Device
**「開発に使用する端末情報」**
![](https://storage.googleapis.com/zenn-user-upload/2c4c6fdf4882-20230930.png =x200)

開発用に実機ビルドをする端末や、AdHoc配信でアプリインストールする端末は、端末情報の登録が必要。

<br>

## 6. Provisioning Profile
**「アプリを実機で動かすためや、App Storeに公開するための許可証」**
![](https://storage.googleapis.com/zenn-user-upload/be545b0556f4-20230930.png =x200)

Apple Developerの項目では**Profile**と記載されている。

上記で作成した項目を一つにまとめる、最終結合するイメージ。
- Certificate（証明書）
- Identifier（識別子）≒ App ID
- Device（端末情報）

<br>

ややこしいポイントとしては、
**Provisioning Profileの作成は、以下の用途それぞれで作成する必要があること。**
- Development（開発用）
- AdHoc（配信用）
- App Store（配信用）

そして、
**Certificate（証明書）は、Development（開発用）とDistribution（配信用）の2種類であること。**
ここの違いは注意。

拡張子は`.mobileprovision`

<br>

## 7. Keys（.p8ファイル）
**「特定の機能を使用するための鍵」**
![](https://storage.googleapis.com/zenn-user-upload/c2616c51abe4-20230930.png =x200)

このKeysは開発に必須では無いので、Provisioning Profileに組み込むとかは無い。

主には他のミドルウェアにKeyを渡して、iOSの機能を使用するイメージ。
よく使用されるのKeyの例として、APNs（push通知）とかだと、KeyをFirebaseに渡してFirebaseからAPNsを使用するイメージ。

拡張子は`.p8`

<br>

-----
### おわり
ざっくりと簡単な言葉に置き換えて記載しているので、認識が違っている箇所があればそっとコメント頂けると幸いです。

<br>

::::details 参考記事
https://qiita.com/bl-lia/items/c6ec88020d526cdb454c
https://qiita.com/fujisan3/items/d037e3c40a0acc46f618
::::