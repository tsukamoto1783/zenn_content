---
title: "【RevenueCat】iOS,AndroidのStore設定まとめ"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [RevenueCat, flutter, ios, inAppPurchase,android]
published: true
publication_name: ncdc
---
RevenueCatを使用してアプリ内課金を実装する際に、各Storeでのサブスクリプション設定が必要です。
この設定が似たような項目がいっぱいありややこしかったので、設定がどこに反映されるかをまとめてみました。

### 前提
- 動作確認にはRevenueCatが提供しているサンプルアプリを使用しています。
  [Magic Weather Flutter - RevenueCat Sample](https://github.com/RevenueCat/purchases-flutter/tree/main/revenuecat_examples/MagicWeather)
- パッケージバージョン：`purchases_flutter: ^6.2.0`
- 動作環境配下。
    ```txt: flutter doctor
    [✓] Flutter (Channel stable, 3.13.2, on macOS 14.0 23A344 darwin-arm64, locale ja-JP)
    [✓] Android toolchain - develop for Android devices (Android SDK version 33.0.2)
    [✓] Xcode - develop for iOS and macOS (Xcode 15.0)
    [✓] Chrome - develop for the web
    [✓] Android Studio (version 2022.1)
    [✓] VS Code (version 1.84.2)
    [✓] Connected device (3 available)
    [✓] Network resources
    ```

<br>
<br>

# iOS
## 機能 > サブスクリプション > サブスクリプショングループ
![](https://storage.googleapis.com/zenn-user-upload/703c984bb23d-20231116.png =6700x)

- **グループ名は変更できません。**
- グループ名がユーザに表示されることはない。
- RevenueCatのコンソールにもグループ名を設定する箇所はない。

<br>

**【備考】**
> すべてのサブスクリプションは、グループに属している必要があります。
> **ユーザが**1つのグループ内で一度に登録できるサブスクリプションは1つだけですが、同じグループ内の別のサブスクリプションに変更することはできます。

例えば、サブスクに登録することでアプリのPro版の機能が使えるようになるとします。
Pro版のサブスクには、`weekly, monthly, yearly`の3つのプランがあるとします。
ユーザーはどれか1つしか登録できないが、他のプランへの変更は可能。

→ アプリ側で1グループに1つのサブスクプランしか登録できない**という意味ではない**こと注意。

<br>
<br>

# ... > サブスクリプショングループ > サブスクリプション
| -                                                                              | -                                                                              |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| ![](https://storage.googleapis.com/zenn-user-upload/dd4250a0af63-20231116.png) | ![](https://storage.googleapis.com/zenn-user-upload/c0a1caac15b9-20231116.png) |

### 参照名
> 参照名はApp Store Connectおよび「売上とトレンド」のレポートで使用されます。App Storeに表示されることはありません。名前は最大64文字です。

- App Store Connectのコンソール上でのみで表示される。
- 参照名は変更可能。

### 製品ID
> レポートに使用される固有な英数字のIDです。ある製品に対して使用した製品IDは、その製品が削除されても再度使用することはできません。

- **製品IDは変更できません。**
  - >App Store Connect で1度製品IDを使用すると、製品が削除された場合でも、**どのアプリでも再度使用することはできません。**
  - 複数のサブスクプランを出す場合は、製品IDの命名規則を考えておかないと統一性無くしそう。
    - IDの命名は、`<固有のprefix>_****_subscription`とか？   
    - RevenueCatが考えるID例は下記参照。
    [「Docs/iOS 製品のセットアップ/アプリ内購入を作成する」](https://www.revenuecat.com/docs/ios-products#create-an-in-app-purchase)
- このサブスク商品のID。RevenueCatと連携するためにも使用する。
![](https://storage.googleapis.com/zenn-user-upload/38f3cc650e72-20231116.png)

<br>
<br>

## ... > サブスクリプショングループ > App Storeのローカリゼーション
| -                                                                              | -                                                                              |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| ![](https://storage.googleapis.com/zenn-user-upload/9d6ad9c98c25-20231116.png) | ![](https://storage.googleapis.com/zenn-user-upload/6554d3874cb7-20231116.png) |

<br>

### サブスクリプショングループ表示名
> これらの名前は、ユーザがサブスクリプション内容を管理する際、デバイス上に表示されます。

> 同じコンテンツのロックを解除する複数のサブスクリプション グループを作成する予定がある場合は、同じサブスクリプション グループの表示名を使用してください。通常、これらのタイプの戦略は、価格テストと割引の提供に使用されます。
> https://www.revenuecat.com/docs/ios-products#subscription-groups



**【WIP】**
上記のように記載あるが、SANDBOX環境でサブスク管理画面を見ても反映されてない。
本番環境になると反映される？この画面じゃなくて→こっち？「設定→ユーザー名→サブスクリプション」
→ SANDBOX環境ではない時に確認する。

![](https://storage.googleapis.com/zenn-user-upload/7ee0e327c911-20231116.png =300x)
*設定 → AppStore → SANDBOXアカウント → 管理 → サブスクリプション*

<br>

### アプリ表示オプション
- カスタムもできるが、基本はアプリ名そのままで良さそう。

<br>
<br>

## ... > サブスクリプショングループ > サブスクリプション > [サブスク製品名] > App Storeのローカリゼーション
![](https://storage.googleapis.com/zenn-user-upload/b8e5b52a622f-20231116.png)

![](https://storage.googleapis.com/zenn-user-upload/1fede073d9ac-20231116.png =450x)
<br>

**※App Store Connectのローカリゼーションという設定項目は2種類あるので注意。**
**「デバイスの設定画面の表示設定」と「AppStore & 決済画面の表示設定」があり、混同注意。**

<br>

> サブスクリプションのローカライズされた表示名および説明がApp Storeに表示されます。

> サブスクリプション表示名と説明は、App Storeとサブスクリプション管理設定でユーザーに表示されます。
> また、**同じアクセスレベルをアンロックするすべての製品に同じサブスクリプション表示名を使用することをお勧めします。**
> 同じ名前を使用することで、App Storeのリストがすっきりし、製品群の増加に伴うユーザーの混乱が少なくなります。
> https://www.revenuecat.com/docs/ios-products#adding-localization

**【WIP】**
決済モーダルには表示されるが、AppStoreにも表示される？←見つけ次第記載する。

![](https://storage.googleapis.com/zenn-user-upload/87b2f932d982-20231116.jpeg =250x)

### 表示名
- 決済モーダル画面やiphoneのサブスク管理画面にも表示される。
  | アプリ内でのモーダル                                                                  | iPhoneデバイス設定の管理画面                                                         |
  | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
  | ![](https://storage.googleapis.com/zenn-user-upload/87b2f932d982-20231116.jpeg =250x) | ![](https://storage.googleapis.com/zenn-user-upload/7ee0e327c911-20231116.png =250x) |

  <br>

### 説明文
RevenueCatだとOfferingsから取得できる。
以下、サンプルアプリ参照。
![](https://storage.googleapis.com/zenn-user-upload/038b4d2de08e-20231116.png =250x)

<br>

## ios設定まとめ
設定時に命名検討が必要な項目
- サブスクリプショングループ名
  - **変更不可。**
  - ユーザーには表示されない。
- サブスクリプション名（製品名）
  - ユーザーには表示されない。
- サブスクリプションID（製品ID）
  - **変更不可。**
  - RevenueCatで使用する。
- ローカリゼーションの表示名（デバイス管理画面表示）
- ローカリゼーションの表示名（Storeと決済画面表示）
- ローカリゼーションの説明文（Storeと決済画面表示）
- etc.

※未検証部分もちらほらあるので、適宜修正予定。

<br>
<br>

# Android
## 定期購読
![](https://storage.googleapis.com/zenn-user-upload/20ec59ca1b30-20231205.png =600x)

  | アプリ内のモーダル                                                                   | Google Play 定期購読管理画面                                                         |
  | ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
  | ![](https://storage.googleapis.com/zenn-user-upload/ee45e84b6c2e-20231205.png =250x) | ![](https://storage.googleapis.com/zenn-user-upload/c918ce55d044-20231205.png =250x) |

### アイテムID
- 変更不可。
- 一度削除したとしても同じIDは今後使用できない。
→ IDの命名規則を考えて作成する。
- ユーザーには表示されない。
- iOSのサブスクリプショングループIDと統一したら混同せずに良さそう。

### 名前
- ユーザー向けのメールや定期購入センターに表示される。
- 変更可能。
- ex.) プレミアムプラン



## 基本プラン（製品）
![](https://storage.googleapis.com/zenn-user-upload/65ebb32de6b8-20231205.png =500x)

### プランID（製品ID）
- 変更不可。
- 一度削除したとしても同じIDは今後使用できない。
→ IDの命名規則を考えて作成する。
- ユーザーには表示されない。
- 購読期間と自動購読の設定も変更不可。
→ ミスするとプラン作り直しになる（**同一IDの再指定はできない**）ので要注意。
- iOSのサブスクリプション製品IDと統一したら混同せずに良さそう。

<br>

### 参考
RevenueCatの`Entitlement, Offerings, Products`については別途以下参照。
https://zenn.dev/tsukatsuka1783/articles/revenuecat_product_composition